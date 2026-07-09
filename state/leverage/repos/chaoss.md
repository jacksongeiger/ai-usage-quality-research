# CHAOSS — Metric Definitions + Reference Implementations

**Investigator note on repo location:** `chaoss/knowledge-base` (the assigned repo) **does not exist** (GitHub: "Repository not found", checked 2026-07-08). The CHAOSS metric-definition estate is currently split:

| What | Where | Status |
|---|---|---|
| Released metric catalog ("knowledge base") | https://chaoss.community/kb-metrics-and-metrics-models/ (WordPress; 89 metrics, 17 metrics models) [primary] | live |
| Metric definition sources (markdown, GQM format) | `chaoss/wg-metrics-development` (Common Metrics WG) and `chaoss/wg-evolution` (+ other wg-* repos) [primary] | live |
| Old canonical repo | `chaoss/metrics` | **archived**, README redirects to chaoss.community/metrics |
| Archived releases (2019–2021 compiled metric md) | `chaoss/website` repo, `archive/release/` | live |
| Reference implementation #1 (Augur) | `chaoss/augur` → **redirects to `augurlabs/augur`** (left the org); CHAOSS-org continuation is **`chaoss/CollectOSS`** ("Python-based data collection and relational storage tool for … CHAOSS metrics") [primary] | live |
| Reference implementation #2 | GrimoireLab (`chaoss/grimoirelab-*`), identity layer = `grimoirelab-sortinghat` [primary] | live |

## Repos & commits inspected
- `chaoss/website` @ `2e74f562ebbf23b258672283267094c9cd49d5f8` (clone: `/private/tmp/claude-501/-Users-jacksongeiger/044a2943-cb9f-4a0f-82cf-62b1372890ad/scratchpad/repos/chaoss`)
- `chaoss/CollectOSS` @ `adfa6e89b95f9c43f5093e72a4f4ac95a884b03b` (clone: `.../repos/chaoss-collectoss`)
- `chaoss/wg-metrics-development` and `chaoss/wg-evolution` @ HEAD 2026-07-08 (clones: `.../repos/wg-metrics-development`, `.../repos/wg-evolution`)
- `chaoss/grimoirelab-sortinghat` via GitHub API (`sortinghat/core/models.py`)
- `chaoss/metrics` (archived) via GitHub API (`README.md`, `resources/metrics-template.md`)

Key files read:
- `wg-evolution/focus-areas/code-development-activity/code-changes-lines.md`, `code-changes-commits.md`, `change-request-commits.md`
- `wg-evolution/focus-areas/code-development-efficiency/change-request-acceptance-ratio.md`, `change-requests-duration.md`
- `wg-evolution/focus-areas/code-development-process-quality/change-request-reviews.md`
- `wg-evolution/focus-areas/community-growth/contribution-attribution.md`
- `wg-metrics-development/focus-areas/people/contributors.md`, `bot-activity.md`, `occasional-contributors.md`; `contributions/self-merge-rates.md`; `time/burstiness.md`, `activity-dates-and-times.md`; focus-area `README.md`s
- `chaoss/website: archive/release/metrics-head.md`, `archive/release/metrics-tables/metrics-table.md`
- `chaoss-collectoss: collectoss/api/metrics/contributor.py`, `pull_request.py`; `collectoss/tasks/git/util/facade_worker/facade_worker/analyzecommit.py`, `rebuildcache.py`; `collectoss/application/db/models/data.py`; `collectoss/tasks/data_analysis/contributor_breadth_worker/contributor_breadth_worker.py`

---

## 1. The Goal-Question-Metric (GQM) structure (worth stealing)

Verbatim from the released metrics table (`chaoss/website: archive/release/metrics-tables/metrics-table.md`) [primary]:

> "CHAOSS metrics are sorted into Focus Areas. CHAOSS uses a Goal-Question-Metric format to present metrics. Individual metrics are released based on identified goals and questions."

Mechanics as implemented in the repos:
- **Focus Area = Goal.** e.g. `wg-evolution/focus-areas/README.md`: "Code Development Efficiency | Learn how efficiently activities around code development get resolved." Four Common-Metrics focus areas are literally *what/who/where/when*: Contributions, People, Place, Time (`wg-metrics-development/focus-areas/README.md`).
- **Every metric file opens with a single `Question:`** — e.g. `code-changes-lines.md`: "What is the sum of the number of lines touched (lines added plus lines removed) in all changes to the source code during a certain period?"
- The template (`chaoss/metrics: resources/metrics-template.md`) mandates: Question → Description → **Objectives ("why someone wants to measure this")** → Implementation (Filters / Visualizations / Data Collection Strategies) → References → Contributors, plus a standing privacy/ethics warning box in every metric: "__The usage and dissemination of health metrics may lead to privacy violations… The usage of metrics must be examined for risk and potential data ethics problems.__"
- Metrics vs **Metrics Models**: "Metrics are meant to answer one single question…; metrics models are collections of metrics… to answer more complex questions" (https://chaoss.community/kb-metrics-and-metrics-models/) [primary]. A composite score is a *model*, and models are documented as compositions of released single-question metrics.
- Definitions are deliberately **implementation-agnostic** (repo description of archived `chaoss/metrics`), with per-platform "Specific description: Git/GitHub/GitLab/Gerrit" subsections and **named mandatory parameters with defaults** (e.g. `code-changes-commits.md`: "Date type: author date or committer date. Default: author date"; "Include merge commits. Boolean. Default: True"; "Include empty commits. Boolean. Default: True").

## 2. Explicit warnings about measuring individuals

- CHAOSS Practitioner Guide — Introduction (https://chaoss.community/practitioner-guide-introduction/, source in `chaoss/wg-data-science/practitioner-guides/`) [primary], verbatim:
  > "Be very careful with metrics that can be used to compare people against each other in ways that might result in the punishment of individuals."
  > "Be careful never to set yourself up for people to weaponize your metrics"
  > "One of the best places to start isn't actually with the metrics, but by spending some time understanding the overall goals for the project."
  > "Every open source project is a little different, and metrics should always be interpreted with the needs of that project and its context taken into account"
- Released metrics page header (`chaoss/website: archive/release/metrics-head.md`) [primary], verbatim:
  > "The CHAOSS project recognizes that there are ethical and legal challenges when using the metrics and software provided by the CHAOSS community. **Ethical challenges** exist around protecting community members and empowering them with their personal information. **Legal challenges** exist around GDPR and similar laws…"
- Anti-valence stance (`wg-evolution/README.md`) [primary], verbatim:
  > "We do not valence the stages but seek to build awareness… The decline can be ok when a project's utility is waning; or terrible if 20% of the entire internet relies on that project. Context matters, and we are not the arbiters of your context. The 'valence' of 'good/bad' emerges from use cases and YOUR context."
- Structural choice reinforcing this: **no released CHAOSS metric is a per-person score.** Contributor-level data appears only as *filters/dimensions* on project-level questions ("Filters: By actors (author, committer). Requires actor merging" — `code-changes-lines.md`). Every CollectOSS metric endpoint signature is `f(repo_group_id, repo_id, period, begin_date, end_date)` — the unit of analysis is the repo/project, never the person (all of `collectoss/api/metrics/*.py`). (Inference: this is a deliberate design posture, consistent with the quoted warnings, not an accident.)
- `activity-dates-and-times.md` openly notes timestamps "can offer transparency for employers about employee contributions" — the current text has lost the older privacy caveats (inference from file contents; the ethics warning now lives centrally in the template/practitioner guides rather than per-metric).

## 3. What the code actually computes (CollectOSS / ex-Augur)

### 3a. Commit-line accounting — `collectoss/tasks/git/util/facade_worker/facade_worker/analyzecommit.py` (Facade algorithm)
Parses `git log -p -M <commit> -n1` per commit, emitting **one record per file touched per commit** with `cmt_added`, `cmt_removed`, `cmt_whitespace`, both author and committer name/email/date/timestamp. Concrete edge-case handling:
- **Merge commits zeroed** (lines 229–233): `if len(parents) == 2: filename = '(Merge commit)'; added = removed = whitespace = 0` → the merger gets zero line credit; merged code isn't double-counted.
- **Whitespace-only changes split out** (lines 267–289): a `+` line that is blank after strip → `whitespace += 1` (not `added`). If an added line's stripped text equals a recently removed line's stripped text **and is >8 chars**, it is reclassified: `removed -= 1; whitespace += 1` → re-indents/reformat shuffles don't count as churn or new code.
- **Rename detection** via `-M` plus `rename to` handling → file moves aren't counted as full delete+add.
- Deleted files tagged `'(Deleted) <path>'` as a distinct filename category.
- Data-mart rollups per `(email, affiliation, week/month/year)` in `rebuildcache.py` (`dm_repo_group_weekly(… added, removed, whitespace, files, patches …)`) — normalization is by *time bucket*, and lines/commits/files/patches are kept as **separate columns, never collapsed into one score**.

### 3b. Per-contributor activity — `collectoss/api/metrics/contributor.py`
`contributors()` SQL (short form): UNION ALL of per-type event counts, then per-user sum:
```sql
SELECT id, SUM(commits), SUM(issues), SUM(commit_comments), SUM(issue_comments),
       SUM(pull_requests), SUM(pull_request_comments), SUM(...) AS total
FROM ( ...issues WHERE reporter_id IS NOT NULL AND pull_request IS NULL...
 UNION ALL ...commits WHERE cmt_ght_author_id IS NOT NULL... GROUP BY cmt_ght_author_id... )
```
Design facts: contribution = **typed event counts** (a "portfolio", not a single number); PR-typed issues excluded from issue counts (`pull_request IS NULL`) to avoid double counting; commits keyed on canonicalized `cmt_ght_author_id`, not raw email. `lines_changed_by_author()` returns weekly `additions, deletions, whitespace` per `(email, affiliation)` — whitespace stays a separate column all the way to the API.

### 3c. PR metrics — `collectoss/api/metrics/pull_request.py`
`pull_request_acceptance_rate` = `merged events / ready_for_review events` per week (event-sourced from `issue_events.action IN ('merged','ready_for_review')`), computed **per repo-group/date, never per author**. Other endpoints: `review_duration`, `pull_request_average_time_to_close`, `pull_request_average_time_between_responses`, `pull_requests_closed_no_merge` — all distributional/time-series at project level.

### 3d. Identity & affiliation
- `collectoss/application/db/models/data.py` `contributors_aliases` table (schema comment, verbatim): "Every open source user may have more than one email… CollectOSS selects the first email it encounters for a user as its 'canonical_email'… an email search will only need to join the alias table…" — canonical id + alias mapping, unique on `(cntrb_id, alias_email)`.
- Affiliation is **resolved data, not ground truth**: `rebuildcache.py` `nuke_affiliations()` / `fill_empty_affiliations()` — affiliations are nullable, recomputed wholesale from editable domain→org mappings whenever mappings change.
- GrimoireLab SortingHat (`sortinghat/core/models.py`): merged "individuals" carry a `Profile` with **`is_bot = BooleanField(default=False)`** — bot-ness is a *property of the resolved identity*, set once, honored by every downstream metric, rather than per-metric regex hacks.
- `contributor_breadth_worker.py`: for each known contributor, pulls their **platform-wide event history including repos the instance doesn't track** — an explicit "work outside the repos we watch exists" correction.

### 3e. Definitions with direct relevance to our metric (from the WG markdown)
- **Code Changes Lines**: counts *lines touched* (add+remove; an edit = 1 removal + 1 addition; a line touched 3× counts 3×); parameters include "Criteria for source code… If we are focused on source code, we need a criterion for deciding whether a file is part of the source code or not"; type split into Lines added / Lines removed / Whitespace.
- **Self Merge Rates** (`wg-metrics-development/focus-areas/contributions/self-merge-rates.md`): "number of change requests merged by the author… without going through a change request review process" vs total merges; explicitly framed as a *review-culture health* signal ("Self-merges should be rare and justified"); filters include lines changed and code-vs-docs file type. A process-quality signal computable from PR metadata only.
- **Change Request Reviews**: measures "the number of reviews, types of review feedback, and review outcomes (accepted / changes requested / declined)" — quality of *process*, not of code; filter "Contributor Type — differentiate between bot- and human-driven reviews".
- **Bot Activity** (`people/bot-activity.md`): bots are a first-class measured category ("Ratio of bot to human activity over time"), so automated work is *visible and attributed*, not discarded.
- **Occasional Contributors**: contributor classes defined by **cadence thresholds as tunable filters** ("Minimum number of contributions before someone is no longer an occasional contributor; maximum length of time between contributions…") instead of hard eligibility cliffs.
- **Burstiness** (`time/burstiness.md`): activity spikes treated as something to *explain* (release cycles, hackathons, events), "differentiate skewed activity versus normal activity", with outlier statistics (Bollinger Bands) — i.e., bursts are context, not merit.
- **Contribution Attribution** (`wg-evolution/community-growth/contribution-attribution.md`): attribution of who did what under what sponsorship is collected "primarily through volunteered information from contributors" — when trace data can't attribute (exactly our AI-vs-human attribution gap), CHAOSS's answer is **structured self-report + links to artifacts**, not inference.
- **Contributors** (`people/contributors.md`): "A contributor is … anyone who contributes to the project in **any way**"; explicitly instruments survey questions for contributors "often overlooked because their contributions are more 'behind the scenes'" — direct precedent for our invisible agent-platform engineers.

## 4. Data the implementations assume
- Full git history (bare clones) for line accounting; GitHub/GitLab REST/GraphQL event streams (issues, PR events, review events, comments) with event timestamps and actor ids.
- A Postgres warehouse (CollectOSS `data` schema) with commits-per-file, typed events, contributor + alias + affiliation tables; time-bucketed data marts.
- Human-maintained inputs: domain→org affiliation mappings, bot flags, identity merges (SortingHat is a whole Django app for this). No CI data assumed by the metrics above; no spend/token data anywhere (CHAOSS "Labor Investment" in the value WG is the closest cost-side concept — noted, not inspected).

## 5. Gaming & unfair-penalty analysis (required per metric)

| Metric | How an engineer games it | Whom it unfairly penalizes |
|---|---|---|
| Code Changes Lines | Churn loops (add→remove→re-add counts every touch), vendoring/codegen, mechanical refactors just above the whitespace detector (>8-char rule dodged by trivial edits) | Reviewers/architects/debuggers; config- and infra-heavy roles; people in squash-merge repos (fewer, denser commits) |
| Code Changes Commits | Commit splitting (defaults count empty and merge commits: "Include empty commits. Default: True") | Squash-merge teams; people who rebase/clean history |
| Contributors / typed event counts | Comment spam, drive-by issue filing, self-review chatter | Heads-down deep workers; non-platform contributors (the def'n says count them, trace data can't see them) |
| Self Merge Rate | Rubber-stamp reviews from a colleague make self-merge → 0 while review value stays 0 | Solo maintainers; on-call/emergency fixers; tiny repos with no second reviewer |
| CR Acceptance Ratio | Submit only safe PRs; pre-negotiate merges offline | Engineers doing exploratory/risky work (wg-evolution explicitly notes declined CRs aren't inherently bad) |
| Bot Activity ratio | Mislabel your automation as human (or your low-quality bulk work as a bot) to move either side of the ratio | Automation authors, if bot-attributed work is then excluded from "their" contribution (mirror of our agent-platform blind spot) |
| Occasional/new-contributor classes | Timing contributions to hop classes (one PR per window stays "active") | Part-timers, parental leave, seasonal contributors — any cadence-based class conflates availability with engagement |
| Activity Dates & Times | n/a to game upward — but trivially reveals off-hours patterns | Off-timezone / caregiving / moonlighting contributors; CHAOSS itself flags employer-transparency use (privacy hazard) |

## 6. What we should steal
1. **GQM discipline**: every number on our dashboard must be the answer to one written question, grouped under a named goal; composites are declared "metrics models" whose member metrics remain individually visible. This is the cheapest interpretability win available (BRIEF requirement: engineer understands *why* their score is what it is).
2. **Parameters with defaults, stated in the definition** (author-vs-committer date, include-merges, include-empty, source-file criterion). Our leverage metric should publish its knobs (window, winsorization cap, cohort rule, CI definition) the same way.
3. **Whitespace/merge/rename accounting** from `analyzecommit.py`: zero-credit merge commits, whitespace-only reclassification, rename detection — a concrete, cheap "quality-adjusted lines" floor that defuses the dumbest LOC gaming. (Our warehouse has only per-PR lines changed; acquiring add/remove/whitespace splits is a plausible "new signal worth acquiring".)
4. **Typed contribution portfolio instead of one output count**: commits / PRs / issues / comments / reviews as separate columns, summed only at display time. Maps to our need: PRs, reviews, agent-platform sessions, chat usage as parallel typed lanes, not one collapsed "output".
5. **Identity-layer bot/automation flag** (SortingHat `is_bot` on the merged identity): resolve identity once, tag automation once, have every metric inherit it — vs per-metric bot regexes. For us: tag agent-platform service identities at the identity layer and *attribute their activity back to the owning engineer as a visible lane* (Bot Activity metric precedent) rather than dropping it.
6. **Affiliation/attribution as refillable, human-correctable data** (`nuke_affiliations`/`fill_empty_affiliations`): assume attribution tables are wrong, make them cheap to rebuild; and where trace data can't attribute (AI-vs-human share), use **structured self-report** (Contribution Attribution metric) instead of pretending.
7. **Cadence classes as tunable filters, not eligibility cliffs** (Occasional Contributors) — replaces our brittle ≥3-PR cold-start floor with a declared "low-data" class that still gets a (wider-error-bar) readout.
8. **The three ethics guardrails as design tests**: "could this compare people in ways that punish individuals?", "could this be weaponized?", "is valence context-dependent?" — CHAOSS applies these *before* release; we should run each candidate subscore through them.
9. **Contributor breadth correction**: explicitly measure work happening outside the repos/systems you track before declaring someone low-output.

## 7. What we should avoid
- **Don't ship per-person project-health metrics as scores.** CHAOSS's own structure (person = filter, never unit of analysis) plus the practitioner-guide warnings say the field's most experienced measurers deliberately refused to build what our client's current metric is. Our self-view-only framing is the mitigation, but any residual comparison (percentile vs cohort) still imports the "punishment of individuals" risk if it ever leaks.
- **Don't adopt raw activity counts as merit.** CHAOSS metrics are descriptive health signals with explicit anti-valence language; every count in §5 is gameable the moment it becomes a score.
- **Don't copy the implementation-agnostic vagueness where we need decisions**: many definitions stop at "Filters: …" with no formula (e.g., Burstiness never defines a burstiness statistic). Good for a standards body; useless as a spec. We must fully specify.
- **Don't rely on acceptance/pass ratios as quality**: CHAOSS's CR Acceptance Ratio has the same failure mode as our CI-clean-rate (iterate/negotiate until green; ratio saturates; punishes risk-takers).
- **Avoid timestamp-pattern metrics** (Activity Dates & Times) for anything scored — privacy hazard CHAOSS itself flags.
- Note: parts of the CollectOSS metric SQL are stale/buggy inherited Augur code (e.g., `commit_comment_ref.cmt_id = commit_comment_ref.cmt_id` self-join tautology in `contributor.py` line 96; `contributors_code_development` mixes `dm_repo_annual.patches` with raw commit adds) — treat it as design precedent, not production-grade math. [inference from code reading]

## Sources
- https://github.com/chaoss/wg-metrics-development , https://github.com/chaoss/wg-evolution [primary]
- https://github.com/chaoss/CollectOSS [primary]; https://github.com/augurlabs/augur (successor of chaoss/augur) [primary]
- https://github.com/chaoss/website (`archive/release/`) [primary]; https://github.com/chaoss/metrics (archived) [primary]
- https://github.com/chaoss/grimoirelab-sortinghat [primary]
- https://chaoss.community/kb-metrics-and-metrics-models/ [primary]
- https://chaoss.community/practitioner-guide-introduction/ [primary]
- https://handbook.chaoss.community/community-handbook/community-initiatives/metrics/metrics-faq (release process only) [primary]
- Independent 2026 overview of the CHAOSS metrics estate (not relied on for claims): https://nesbitt.io/2026/05/27/chaoss-metrics-in-2026.html [independent, unverified]
