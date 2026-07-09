# Repo notes: middlewarehq/middleware (OSS DORA dashboard)

**Repo & commit**: https://github.com/middlewarehq/middleware [primary] — cloned `--depth 1`, HEAD = `a31c4daee37340556ca9d4603940696e31672111` (2026-06-09, "Fix script injection"). Apache-2.0. Stack: Flask/SQLAlchemy analytics server (Python) + Next.js web server (TS) + Postgres. Clone at `/private/tmp/claude-501/-Users-jacksongeiger/044a2943-cb9f-4a0f-82cf-62b1372890ad/scratchpad/repos/middleware`.

All claims below are read directly from the code at this commit [primary]. My interpretations are marked *inference*.

## Files read (all under `backend/analytics_server/` unless noted)
- `mhq/service/code/sync/etl_code_analytics.py` — per-PR metric derivation (the core file)
- `mhq/service/code/lead_time.py`, `mhq/service/code/models/lead_time.py` — team lead-time aggregation
- `mhq/service/code/sync/etl_github_handler.py` — GitHub ETL, bot filter, PR model mapping
- `mhq/service/code/sync/revert_prs_github_sync.py` — revert-PR detection
- `mhq/service/code/pr_filter.py`, `mhq/store/models/code/filter.py` — PR filtering (authors, branches, exclusions)
- `mhq/store/repos/code.py` — SQL(Alchemy) queries: merged-in-interval filter
- `mhq/store/models/code/pull_requests.py` — PullRequest schema (incl. code_stats meta)
- `mhq/service/deployments/analytics.py`, `deployment_pr_mapper.py`, `models/adapter.py`, `deployments_factory_service.py` — deployment frequency + PR→deployment mapping
- `mhq/service/merge_to_deploy_broker/mtd_handler.py` — merge→deploy caching
- `mhq/service/incidents/incidents.py`, `mhq/exapi/git_incidents.py` — CFR + MTTR
- `mhq/store/models/core/teams.py`, `users.py` — Team/Users models
- `mhq/utils/string.py` — bot-name regex
- `web-server/src/utils/dora.ts` — composite DORA score + industry benchmark

## What it computes (concretely)

Team-level DORA four keys only. A "team" is a set of repos (`TeamRepos`); metrics aggregate over the team's repos' PRs/deployments/incidents, never over people. `Team.member_ids` exists but is not used in metric computation.

### 1. Lead time for changes — additive per-PR segments (`etl_code_analytics.py`)
Computed per merged PR at ETL time, stored on the PR row:
- `first_commit_to_open` = PR `created_at` − earliest PR-commit `created_at`; **clamped to 0 if negative** (rebases/force-pushes can reorder dates).
- `first_response_time` = first REVIEW event − first READY_FOR_REVIEW event (falls back to PR `created_at` if never draft). If **no review ever**: falls back to `state_changed_at` (merge/close time) — i.e., the whole PR duration counts as "waiting for first response".
- `rework_time` = if no approving review: `state_changed_at` − first-response end; if first review IS an approval: `0`; else: first approval − first review.
- `merge_time` = `merged_at` − first approval; `0` if merged with no approvals; **set to null if negative** (comment in code: "Prevent garbage state when PR is approved post merging").
- `merge_to_deploy` = deployment `conducted_at` − PR `state_changed_at`, backfilled by a broker (`mtd_handler.py`) that walks successful "deployment" workflow runs and assigns every not-yet-deployed merged PR reachable from the deployed branch (BFS over a base←head branch graph, `deployment_pr_mapper.py`).
- `lead_time` = sum of all 5 segments; `cycle_time` = same minus `first_commit_to_open` (`models/lead_time.py`).

Team aggregate = **PR-count-weighted arithmetic mean** of each segment over all merged PRs in the window (`lead_time.py::_get_avg_time`). Weekly trends bucket by `merged_at`, with `fill_missing_week_buckets` zero-filling empty weeks. Only MERGED PRs count (`_filter_prs_merged_in_interval`: `state == MERGED AND state_changed_at BETWEEN from AND to`). PRs deduped by id via `set()`.

- `rework_cycles` = number of review→commit alternations between the first blocking (non-approve) review by an assigned reviewer and the first approval; commits and blocking reviews in that interval are interleaved on a timeline and each block→commit transition counts as one cycle.

### 2. Deployment frequency (`deployments/analytics.py`)
Two per-repo modes (fallback hierarchy):
- **WORKFLOW mode**: user marks specific CI workflows as "deployments"; successful runs = deployments.
- **PR_MERGE mode** (default when no workflow configured): **every merged PR counts as one deployment** (`PullRequestToDeploymentsAdaptor`), `merge_to_deploy` forced to 0.
Frequency = mean deployments per day/week/month bucket over SUCCESS-status deployments; oddity: `_adjust_frequency_for_granularity` clamps weekly ≥ daily×7 and monthly ≥ daily×30.

### 3. Change failure rate (`incidents/incidents.py`)
Incident-window attribution: sort deployments and incidents by time; every incident created between deployment N and N+1 marks deployment N failed. `CFR = |failed deployments| / |all deployments| × 100`. Incident sources:
- Incident-tool integrations;
- **Revert-PR detection** (`revert_prs_github_sync.py`): a PR whose head branch matches `revert-<original-pr-number>` (GitHub's auto-revert branch convention) is an incident against the original PR — pure metadata, no code access;
- Configurable regex on PR title/head-branch (`INCIDENT_PRS_SETTING`) to treat e.g. hotfix PRs as incidents, with resolution PR linked by regex-extracted PR number.

### 4. MTTR
Arithmetic mean of `resolved_date − creation_date` over resolved incidents in window.

### 5. Composite "DORA score" (frontend, `web-server/src/utils/dora.ts`)
Each pillar → 0/2/4/6/8/10 via breakpoint ladders (LT & MTTR breakpoints: 6 months, 1 month, 1 week, 1 day, 1 hour, 0; weekly-DF breakpoints: once/6mo, monthly, weekly, daily, 2×daily, on-demand; CFR score = `(100−cfr)/10`). Missing pillars dropped; remaining averaged to one number, compared against a hardcoded per-industry benchmark table (all-industries = 6.3; the values are shipped as constants — source not cited in code, *unverified provenance*).

## Data it assumes
- GitHub/GitLab PR objects incl. timeline events (reviews, ready_for_review), per-PR commit list, requested reviewers; PAT-based sync with bookmark incremental fetch.
- CI workflow runs (status, `conducted_at`, actor, head branch) only for repos where a human configured "this workflow = deployment".
- Optional incident-tool integrations; else falls back to revert-PR convention.
- Stores `additions/deletions/changed_files/comments` per PR in `meta.code_stats` (`pull_requests.py`) — **displayed in PR tables but used in NO metric** (*inference: deliberate choice to keep LOC out of scoring*).
- Postgres via SQLAlchemy; no dbt/warehouse; all aggregation in Python, not SQL.

## Per-person handling
- **There is no per-person metric or score anywhere.** Metrics attach to repo-sets ("teams").
- `PRFilter.authors` exists (API can filter PR lists by author usernames) and every PR row carries `author` (GitHub login) and `reviewers`; deployments carry `actor`. So person-level *drill-down rows* exist (PR tables in UI show authors), but nothing is ranked/scored per person.
- Identity = raw provider username strings; a `Users` table exists (name/email) but there is no username↔employee identity resolution feeding metrics. Multi-provider identity is unhandled for metrics.

## Bot & noise handling
- Bot **events** filtered two ways: GitHub API `user.type == "Bot"` (`_github_bot_filter`, ETL) and name-regex `(?i)(\b[\w@-]*[-_\[\]@ ]+bot[-_\d\[\]]*\b|\[bot\]|_bot_|_bot$|^bot_)` (`utils/string.py::is_bot_name`, applied in `filter_non_bot_events` before computing first-response/rework). So a bot auto-approval doesn't count as "first response".
- **Gap**: bot-authored PRs (e.g., dependabot) are NOT excluded — they flow into lead time and, in PR_MERGE mode, into deployment frequency. Only manual per-team `EXCLUDED_PRS_SETTING` (explicit PR-id list) removes them.
- Reviewers list excludes the PR author (`e.actor_username != pr.author`), so self-reviews don't count as responses; but merge-without-review yields `merge_time = 0` and `rework_time` spanning to merge — unreviewed fast merges look "elite".

## Gaming & unfair-penalty analysis (per metric, our required lens)
- **Lead time**: gamed by squash/rebase (resets first-commit date; they clamp negatives to 0, which *helps* the gamer), by merging without review (no-review PRs get merge_time 0), by slicing work into trivial PRs, or by pre-arranged instant approvals. Unfairly penalizes: authors in review-rigorous or cross-timezone teams (first_response dominated by reviewer latency, not author behavior); big-change/complex work.
- **Deployment frequency**: in PR_MERGE mode literally = merged-PR count → gamed by PR splitting; in WORKFLOW mode gamed by pointing it at a chatty workflow. Penalizes release-train/mobile teams with batched deploys.
- **CFR (revert/incident based)**: gamed by reverting via a differently-named branch or fixing forward silently — failures vanish; honest incident logging RAISES your CFR ("honesty tax"). Penalizes teams with disciplined incident hygiene.
- **MTTR**: gamed by close-fast-reopen-new; penalizes teams that own genuinely severe surfaces.
- **Rework cycles**: gamed by resolving feedback in DMs then getting a clean approve; penalizes authors whose reviewers leave iterative feedback on-platform.
- **Composite 0–10 bucket score**: marginal gaming is muted inside a bucket (good), but breakpoint edges create cliff incentives (merge before the 1-day lead-time boundary).

## What we should steal
1. **Additive time decomposition** (commit→open→first-response→rework→approve→merge→deploy): the most interpretable pattern seen — an engineer can see exactly *where* time goes. Adapt for our leverage score: decompose into named, additive sub-signals rather than one opaque ratio.
2. **Revert-PR detection from branch-name convention** as a zero-code-access, per-author-attributable quality signal (`revert-<pr#>`). Maps to our "no post-merge defect linkage" constraint — a cheap NEW signal: % of an engineer's merged PRs later reverted. (Lagging + sparse, so use as slow-moving quality adjuster, not headline. *inference*)
3. **Dual bot filtering** (provider `type == Bot` AND name regex) applied to *events*, plus the lesson from their gap: also filter bot *authors*.
4. **Fallback definition hierarchy** (workflow deployments → PR-merge proxy, with the affected term zeroed): graceful degradation when a data source is missing — the right pattern for our agent-platform engineers (define "output" at best-available granularity per person instead of scoring them zero).
5. **Breakpoint-ladder scoring + external benchmark anchor** (`dora.ts`): coarse ordinal buckets are interpretable and dampen marginal gaming; average only over pillars that exist (missing signal ≠ zero) — directly fixes our brittle cold-start floors.
6. **Manual exclusion escape hatch** (`EXCLUDED_PRS_SETTING`) + explicit "garbage-state" guards (negative merge_time → null, negative first_commit_to_open → 0): metric code that names its edge cases.
7. **Zero-filled trend buckets** (`fill_missing_week_buckets`): never let empty weeks silently vanish from trends.
8. **LOC stored but never scored**: they keep additions/deletions purely as display context. Matches our concern that line counts are the most gameable signal.

## What we should avoid
1. **PR-count-weighted arithmetic means everywhere** — no medians, no percentiles, no winsorization; one 3-month-old PR wrecks the team average (their `max_cycle_time` outlier filter is opt-in). Use medians/p75 + winsorizing.
2. **"Every merged PR = a deployment"** fallback silently turns a throughput metric into raw PR count — exactly the failure mode our current metric has.
3. **No-review PRs scoring as instant/elite** (`merge_time=0`, no first-response penalty) — rewards skipping review; any leverage metric must not make *less scrutiny* look *better*.
4. **Unmapped provider usernames as identity** — fine for team aggregates, unusable for a per-person score (aliases, multiple accounts, bots-as-authors).
5. **Frequency clamping** (`weekly ≥ daily×7`) — silent "adjustments" that make numbers non-reproducible from raw data; every transform should be explainable on the dashboard.
6. **Hardcoded benchmark constants without provenance** (industry DORA scores in `dora.ts`) — anchor numbers must carry citations or engineers will (rightly) distrust the score.
7. **Incident-window CFR attribution** (all incidents blamed on the most recent deployment) — too coarse if repurposed per person; don't attribute team-level failures to individuals.
