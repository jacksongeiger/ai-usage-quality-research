# Apache DevLake — DORA + engineering-metrics + AI-usage implementation notes

## Repo & commit
- Repo: https://github.com/apache/incubator-devlake [primary] (still at this name; not renamed as of clone date)
- Cloned: `--depth 1`, HEAD = `15e34bb80b5b3e35c77a88122761d507b23363e6` (2026-07-02)
- Local: `/private/tmp/claude-501/-Users-jacksongeiger/044a2943-cb9f-4a0f-82cf-62b1372890ad/scratchpad/repos/devlake`
- Architecture: Go plugin ETL → "domain layer" SQL tables (pull_requests, commits, cicd_deployment_commits, incidents, project_pr_metrics, user_accounts/teams) → all metric math lives in (a) Go "enricher" subtasks and (b) rawSql inside Grafana dashboard JSON. There is no dbt; dashboards ARE the metric definitions.

## Files read (paths relative to repo root)
- `backend/plugins/dora/tasks/change_lead_time_calculator.go` (per-PR cycle-time pipeline)
- `backend/plugins/dora/tasks/incident_deploy_connector.go` (incident→deployment attribution)
- `backend/plugins/dora/tasks/deployment_generator.go` (what counts as a deployment)
- `backend/plugins/dora/tasks/incident_from_issue_generator.go` (issues typed INCIDENT become incidents)
- `backend/plugins/dora/tasks/cicd_task_env_enricher.go` (deprecated no-op; env classification moved to scope-config regexes)
- `backend/plugins/refdiff/tasks/deployment_commit_diff_calculator.go` (git-graph commit attribution between deployments)
- `backend/plugins/org/tasks/user_account.go` (identity resolution)
- `backend/plugins/claude_code/models/user_activity.go`, `backend/plugins/gh-copilot/models/{user_metrics,language_metrics,org_metrics}.go`, `backend/plugins/q_dev/models/user_report.go` (per-user AI telemetry schemas)
- `backend/core/models/domainlayer/code/{pull_request,commit}.go`, `backend/core/models/domainlayer/crossdomain/account.go`
- Grafana rawSql: `grafana/dashboards/postgresql/{DORA,dora-by-team,github-copilot-dora-correlation,ai-cost-efficiency,multi-ai-comparison,qdev_user_report,engineering-overview,engineering-throughput-and-cycle-time,steering-adoption-tracker,developer-productivity-hours,language-ai-heatmap,contributor-experience}.json`

## What it computes (concrete formulas)

### 1. Per-PR lead-time decomposition (Go: `change_lead_time_calculator.go`, table `project_pr_metrics`)
For every MERGED PR (`pr.merged_date IS NOT NULL`), minutes, ceil-rounded, negative spans → NULL:
- `PrCodingTime` = first_commit.authored_date → pr.created_date (first commit = MIN(commit_authored_date) in `pull_request_commits`)
- `PrPickupTime` = pr.created_date → first review comment **excluding the PR author's own comments** (`prc.account_id != pr.author_id`)
- `PrReviewTime` = first review → merged_date
- `PrDeployTime` = merged_date → finished_date of the **first successful PRODUCTION deployment whose commit-diff set contains the PR's merge_commit_sha**
- `PrCycleTime` = CodingTime + (created→merged) + DeployTime; each component nil-tolerant (missing component simply contributes 0 — a PR with no CI/deploy still gets a cycle time).

### 2. Deployment identification (Go: `deployment_generator.go`)
A `cicd_pipeline` becomes a deployment iff `pipeline.type = 'DEPLOYMENT'` OR any of its tasks is a DEPLOYMENT task; results restricted to `{SUCCESS, FAILURE}` (skipped runs excluded). Environment inferred if blank: production > staging > testing by presence of tasks. What is "DEPLOYMENT"/"PRODUCTION" is user-configured regex on pipeline/task names (scope config) — i.e., the semantic layer is config, not code.

### 3. Commit→deployment attribution (Go: `refdiff/deployment_commit_diff_calculator.go`)
Builds an in-memory git commit graph and computes the "lost sha" set between `prev_success_deployment_commit` and the current deployment commit → `commits_diffs` rows. So attribution of changes to a deployment is by real git ancestry, NOT by timestamps. Memoized in `_tool_refdiff_finished_commits_diffs`.

### 4. DORA dashboard SQL (`grafana/dashboards/postgresql/DORA.json`)
- **Deployment Frequency** = median over weeks of `COUNT(DISTINCT deployment days)` per week (calendar-spine LEFT JOIN so zero-weeks count), with GitLab/BitBucket multi-row dedup: `GROUP BY cicd_deployment_id, MAX(finished_date::DATE)`. Bucketed to 2023 or 2021 DORA benchmark labels via a `$dora_report` switch, e.g. 2023: `median_weekly >= 5 → elite; >=1 → high; monthly >=1 → medium; else low`.
- **Lead Time for Changes** = median `pr_cycle_time` over merged PRs in window; median implemented as `PERCENT_RANK() OVER (ORDER BY pr_cycle_time)`, then `MAX(...) WHERE ranks <= 0.5` (lower-median, no interpolation).
- **Change Failure Rate** = `SUM(has_incident) / COUNT(deployment_id)` over successful production deployments in window, where `has_incident` is 0/1 per deployment (multiple incidents on one deployment count once). 2023 buckets: `<=5% elite, <=10% high, <=15% medium, >15% low`.
- **Failed Deployment Recovery Time (2023) / MTTR (2021)** = median of `incident.resolution_date − causal_deployment.finished_date` (minutes), same PERCENT_RANK median trick.
- **Incident→deployment attribution** (Go: `incident_deploy_connector.go`): incident is blamed on the **most recent successful PRODUCTION deployment finished before incident.created_date** (`ORDER BY finished_date DESC LIMIT 1`). Purely temporal, explicitly a heuristic; incidents with no prior deployment are dropped.
- **Incidents** = issues whose type maps to `INCIDENT` in board scope config (`incident_from_issue_generator.go`) or native PagerDuty/Opsgenie/Rootly incidents.

### 5. Team-level DORA (`dora-by-team.json`)
Deployment counts for a team iff **any commit inside the deployment's diff set was authored by a team member**: chain `cicd_deployment_commits → commits (author_id) → user_accounts → users → team_users → teams`. Same pattern re-scopes CFR/MTTR/LT by team. Teams and users come from an uploaded org CSV (org plugin).

### 6. Identity resolution (Go: `org/tasks/user_account.go`)
`users` (directory) ↔ `accounts` (tool identities) linked by **exact email match, else exact full-name match**, only for accounts not already manually mapped. No fuzzy matching, no bot flagging at the domain level (GitHub `type` field exists in the raw `_tool_github_accounts` table but is not consumed by any metric).

### 7. AI-usage plugins (per-user, per-day telemetry — no code content)
- **gh-copilot** (`models/*.go`): per-user-per-day booleans `used_agent/used_chat/used_cli/used_code_review_active|passive`; per-language-per-model completion metrics `acceptances, lines_suggested, lines_accepted`; org/enterprise daily `total_active_users`, `seat_total`.
- **claude_code** (`models/user_activity.go`): per-user-per-day `cc_commit_count, cc_pull_request_count, cc_lines_added, cc_lines_removed, cc_session_count`, per-tool accept/reject counts (`edit_tool_accepted/rejected`, write, multi-edit, notebook), chat conversation/message/artifact counts.
- **q_dev** (Amazon Q; `models/user_report.go`): per-user-per-day `credits_used, overage_credits_used`, messages, conversations, subscription tier.
- Dashboards: `qdev_user_report.json` renders a **named per-user table** of credits/messages/first-last activity (no anonymization); `developer-productivity-hours.json` plots AI activity by hour-of-day/day-of-week; `language-ai-heatmap.json` completions by language; `steering-adoption-tracker.json` compares prompt/response length with vs. without "steering" files (crude before/after feature-efficacy readout).

### 8. AI ⇄ outcome joins (the part closest to OUR problem)
- **`github-copilot-dora-correlation.json`**: weekly org-level `adoption_pct = daily_active_users / total_seats`, joined to weekly avg `pr_cycle_time` (and DF, CFR, MTTR, review time, SonarQube bugs/smells/complexity/coverage); computes **Pearson r in pure SQL** (STDDEV_POP/AVG cross-products), returns NULL when n < 4 weeks; also buckets weeks into adoption tiers `<25 / 25-50 / 50-75 / >75%` and compares mean cycle time across tiers ("High vs Low adoption" delta panels).
- **`ai-cost-efficiency.json` / `multi-ai-comparison.json`**: **Credits per Merged PR** = `SUM(credits_used) / COUNT(merged PRs)` per ISO week — an org/team-level "output per AI dollar", never per person. Also credits per deployment and credits per resolved issue; Kiro-vs-Copilot acceptance-rate comparison.

### 9. Other productivity metrics (engineering-overview / throughput dashboards)
- **Coding Days %** = `100 * COUNT(author×day pairs with ≥1 commit) / (COUNT(DISTINCT authors) × COUNT(DISTINCT days))`, weekdays only (`ISODOW 1..5`).
- **PR Review Depth** = `COUNT(DISTINCT pr_comments) / COUNT(DISTINCT merged PRs)` per period (comments per PR).
- **PR Size** = `additions + deletions` from PR metadata (no winsorization anywhere in the repo).
- Bug counts per 1k LOC, Unlinked-PR % (PRs not linked to an issue), Mean Time to Merge.

## Data it assumes
- Merged-PR metadata incl. `merge_commit_sha`, additions/deletions, `is_draft`, parent PR; PR commit list with authored dates; PR comments with account ids.
- CI/CD pipeline+task rows with name-regex-classifiable environments and results; a git commit graph (via gitextractor) deep enough to diff consecutive deployments.
- An **org-supplied user↔account↔team CSV** — team-level anything depends entirely on this manual mapping.
- Incident-typed issues or an incident tool; incident created/resolved timestamps.
- Vendor AI telemetry exports (Copilot API, Q Dev S3 reports, Claude Code/Kiro exports) keyed by user id/email per day.

## Design choices relevant to OUR problem
1. **Level of aggregation**: DORA and all AI-correlation metrics are project/team-level; per-person data exists (PR author, commit author, per-user AI credits) but is only ever surfaced as raw activity tables, never as a per-person composite score. The Pearson-r panels deliberately correlate *weekly team aggregates*, sidestepping per-person attribution entirely.
2. **Quality adjustment**: none multiplicative. Quality is a *separate axis* (CFR, MTTR, SonarQube panels alongside adoption) rather than a discount factor on output. Their CFR analog of "quality" is deployment-scoped and binary (has_incident 0/1), medians everywhere instead of means.
3. **Churn**: no churn/rework metric anywhere (no GitClear-style re-edit tracking). Line counts are used only as PR-size descriptive stats. `commits_diffs` infra could support "code deployed then reverted" but is not used that way.
4. **Normalization/cohorting**: benchmark-anchored, not peer-percentile — raw values are bucketed against fixed published DORA thresholds (2021/2023 selectable). No within-org percentile ranking of individuals exists.
5. **Identity**: exact email/full-name matching plus manual CSV override; **no bot filtering in any metric SQL** (dependabot-style accounts silently inflate PR/commit/comment counts unless the org omits them from the user CSV).
6. **Gaming defenses**: almost none by design (it's a team observability tool, not an incentive system). The only robustness mechanisms are: medians via PERCENT_RANK (outlier-resistant), first-review excludes self-comments, deployment dedup (multi-commit pipelines = ONE deployment), calendar-spine joins so empty weeks count as zeros, min-n guard (r = NULL under 4 data points), and skipped-CI runs excluded from deployment results.
7. **Missing-data honesty**: every DORA panel has an `_is_collected_data` CTE that returns explicit "N/A. Please check if you have collected deployments/incidents." strings instead of silently rendering 0 — metric validity is surfaced to the viewer.
8. **Config-as-semantics**: what counts as "deployment", "production", "incident" is per-repo regex config. Great flexibility; also means cross-team comparability is only as good as config discipline.

## Gaming & unfair-penalty analysis (per metric, as required)
- **Deployment Frequency (median deploy-days/week)**: *Game*: split one release into many no-op pipeline runs matching the deployment regex; deploy trivial config bumps daily. *Penalizes*: teams on mobile/firmware/release-train cadences and repos where "deploy" isn't pipeline-shaped (data/ML jobs).
- **Lead Time (median PR cycle time incl. deploy)**: *Game*: open the PR only after work is done locally (kills CodingTime), self-merge fast, keep PRs cosmetic-small; rebase away early commits so first-commit date is late. *Penalizes*: engineers in heavy-review or slow-deploy repos (DeployTime dominates and is out of the author's control), and long-lived research branches.
- **Change Failure Rate (temporal incident→last-deploy blame)**: *Game*: deploy something trivial right after a risky deploy (the newer deploy absorbs the blame is FALSE — actually the *latest* pre-incident deploy takes blame, so ship a harmless deploy just before incidents get filed... equivalently, delay incident ticket creation until after someone else deploys); reclassify incident tickets as bugs. *Penalizes*: high-frequency deployers (more deployments near any incident) and whoever deploys last before a slow-burn defect surfaces; teams with disciplined incident logging look worse than teams that under-report.
- **MTTR/Recovery time (resolution − deploy finished)**: *Game*: close incident tickets fast and reopen new ones. *Penalizes*: teams whose incidents are detected late (clock starts at the deploy, not at detection), on-call teams inheriting others' defects.
- **Coding Days %**: *Game*: one trivial commit per weekday (a cron can do it). *Penalizes*: batch committers, reviewers/architects, part-timers; weekday-only rule penalizes flexible schedules.
- **PR Review Depth (comments/PR)**: *Game*: comment spam, bot comments (not filtered). *Penalizes*: teams doing synchronous/pair review with few written comments.
- **Credits per Merged PR (org-week)**: *Game*: at org level, hard for one person; if made per-person it inherits every flaw of the client's current metric (deflate spend, inflate PR count via PR-splitting). *Penalizes*: agent-platform/exploratory users whose credits map to non-PR output — exactly the client's known problem 2.
- **Copilot adoption % & tier correlations**: *Game*: open the IDE plugin daily to count as an active user (adoption is presence, not use). *Penalizes*: nobody directly (aggregate), but tier comparisons are confounded — high-adoption weeks may simply be weeks with different work mix; the dashboard itself ships a "How to Read Correlation ≠ causation" panel.
- **Acceptance rate (lines_accepted/lines_suggested)**: *Game*: accept everything then rewrite (acceptance logged, rewrite invisible). *Penalizes*: careful users who reject aggressively; language communities where suggestions are weaker.

## What we should steal
1. **Cycle-time decomposition per PR** (coding/pickup/review/deploy) — interpretable, per-person computable from data the client HAS (PR timestamps; CI events), and it reframes "leverage" as *where AI removes time*, not output-per-dollar.
2. **Median-of-weekly-buckets with a calendar spine** — robust to right-skew (client's known problem 3) and to zero-activity weeks; kills the need for winsorization.
3. **Benchmark/threshold bucketing instead of peer-percentile ranking** — fixed, published thresholds make scores stable month-to-month and non-competitive (fits the self-reflection privacy constraint; addresses cold-start brittleness, problem 4).
4. **Tier-comparison + in-SQL correlation with a min-n guard (r=NULL if n<4)** — honest "is AI usage even associated with better flow HERE?" readout before any per-person scoring; cheap to replicate on the client's warehouse.
5. **`_is_collected_data` pattern**: explicit "N/A — you lack signal X" states instead of a fake neutral score (directly fixes the client's CI-clean-rate defaulting-to-1.0 pathology).
6. **Quality as a separate co-displayed axis** (CFR/MTTR/static-analysis beside throughput), never a multiplier.
7. **Per-user AI telemetry schema**: day×user×tool×feature grain with accept/reject counts and feature booleans (used_agent/used_chat/used_cli) — an "adoption depth" vector, richer than dollars; the claude_code table shows vendors already export per-user sessions→commits/PR counts linkage.
8. **Deployment dedup + git-graph attribution** ideas generalize to "count units of shipped change once, attribute by ancestry not timestamps."

## What we should avoid
1. **Temporal incident→deployment blame** (last deploy before incident) — too coarse for anything person-adjacent; only usable at team/quarter aggregate.
2. **Exact-match identity with no bot filtering** — any per-person metric on top of DevLake-style tables silently counts dependabot/renovate/CI accounts; we must filter bot accounts and merge identities before anything else.
3. **Named per-user AI-spend leaderboards** (`qdev_user_report.json` renders exactly this) — violates the client's privacy constraint; a warning of how easily "observability" becomes surveillance.
4. **Acceptance rate as a quality/leverage signal** — measures suggestion plausibility, not value retained; invisible post-accept rewrites (inference: same failure family as the client's CI-clean-rate).
5. **Credits-per-PR at individual grain** — DevLake pointedly keeps it org/week; individualizing it reproduces the client's current broken metric.
6. **Presence-based adoption (seat-active %)** as anything more than a denominator sanity check.
7. **Regex-configured semantics without config audit** — cross-cohort fairness dies if each team defines "deployment"/"incident" differently; any client rollout needs centralized definitions.
8. **PERCENT_RANK "median" quirk**: their median is the lower median via `MAX(... WHERE ranks<=0.5)` — fine at scale, biased at small n; use proper percentile_cont for small per-person windows.

All claims above are from reading the cited repo files at commit 15e34bb [primary]; interpretive judgments (steal/avoid, gaming scenarios) are inference.
