# Faros Community Edition — repo-reading notes

## Repo & commit

- **Assigned repo `github.com/faros-ai/faros-community-edition` no longer exists** — GitHub API returns 404 and it is absent from the faros-ai org repo list (verified directly via `gh api`, 2026-07-08) [primary]. An independent research page states the repo "has since been removed from GitHub without a public deprecation notice" as Faros pivoted to closed-source enterprise (https://rywalker.com/research/faros-ai) [independent, claim seen in search snippet — the removal itself is verified first-hand by the 404]. The companion `faros-ai/faros-ai.github.io` repo was archived 2026-04-01 per the same search results [independent, unverified].
- **Successor used**: the most-recently-synced surviving fork, `github.com/JetsettingJames/faros-community-edition`, cloned at commit `35e3a3ab99cd` (2025-07-09). Verified by deepening history: the tip is exactly the final upstream commit `bc52033` ("Add faros-ai-devin and Copilot to cla.yml (#384)", 2025-07-09, by a faros-ai employee) **plus one trivial README commit by the fork owner**. All code analyzed below is upstream Faros code.
- The AI Copilot Evaluation dashboards were added upstream in PR #372 "[FAI-13039] - Add AI Copilot Evaluation dashboards and mock data", merged 2024-11-12 [primary, from git log].
- Local clone: `/private/tmp/claude-501/-Users-jacksongeiger/044a2943-cb9f-4a0f-82cf-62b1372890ad/scratchpad/repos/faros`

**Files read** (paths relative to clone root):
- `init/resources/metabase/dashboards/copilot_impact.json`, `copilot_usage.json`, `copilot_surveys.json`, `dora_deployments_incidents.json`, `dora_releases_bugs.json` (partially), `git.json`, `github.json`, `prs.json`, `builds.json` (card list)
- `canonical-schema/V001__create_tms_vcs_ims_cicd.sql`, `V013__create_vcs_email_copilot.sql`, `V014__create_survey.sql`, `V015__alter_vcs_UserToolUsage.sql`
- `dbt-transforms/models/custom_metrics/{deployment,incident,task_creators}.sql`, `dbt-transforms/models/custom_metrics/base/{base_build,base_pipeline,base_task}.sql`
- `mock-data/src/mockdata.ts` (semantics of the Copilot data model), `init/resources/airbyte/workspace/airbyte_config/STANDARD_SYNC.yaml` (`faros_copilot_seats`, `faros_copilot_usage` streams)

Note: metric logic lives in **Metabase dashboard JSON (MBQL + native SQL) and dbt models**, not in Go/Python application code. The Metabase JSON is Handlebars-templated (`{{ table "…" }}`); I normalized and parsed it programmatically.

## What it computes (concretely)

### 1. AI Copilot Impact dashboard (`copilot_impact.json`) — cohort comparison, NOT per-engineer scores
Every card joins `vcs_PullRequest.author = vcs_UserTool.user` and classifies each PR row with an MBQL case expression (verbatim semantics):

```
Tool = CASE WHEN pr.createdAt > userTool.startedAt
             AND (userTool.endedAt IS NULL OR pr.createdAt < userTool.endedAt)
       THEN coalesce(userTool.toolDetail, userTool.toolCategory)
       ELSE 'No Tool' END
```
i.e. **attribution is a person-period seat window** (Copilot license start/end dates), not per-PR AI detection. Filter on every card: `mergedAt IS NOT NULL` (merged PRs only). Metrics, broken out by Tool vs "No Tool" (and by week):
- **"PR Merge Rate"** = `distinct(pr.id) / distinct(pr.author)` — PRs merged **per user**, normalizing away cohort size. (Despite the name, it is not a merge percentage.)
- **Merge Time** = `avg(datetime-diff(createdAt, mergedAt, hour))`.
- **PR Size** = `median(linesAdded)`, `median(linesDeleted)`, Net = `median(linesAdded) − median(linesDeleted)` — medians, deliberately skew-resistant.

### 2. AI Copilot Adoption & Usage dashboard (`copilot_usage.json`)
Two data paths:
- **Per-user events**: `vcs_UserToolUsage` (usedAt, optional repository/branch/file, `charactersAdded`). DAU = `distinct(userTool.user)` per day.
- **Pre-aggregated Copilot API metrics** in a generic metric store: `faros_MetricValue` rows keyed by `faros_MetricDefinition` uid (`DailyGeneratedLineCount_Accept`, `DailyGeneratedLineCount_Discard`, `DailySuggestionReferenceCount_Accept/Discard`, `DailyChatTurnCount`, `DailyChatAcceptanceCount`, `DailyActiveUserTrend`, `DailyActiveChatUserTrend`) with dimension tags (`copilotEditor`, `copilotLanguage`, `copilotOrg`, `copilotTeam`).
- **Acceptance Rate** = `sum(value WHERE def='DailyGeneratedLineCount_Accept') / sum(value WHERE def IN (Accept, Discard))` — computed by team/editor/language, never per person.

### 3. AI Copilot Surveys dashboard (`copilot_surveys.json`) — self-report triangulation
- **Time saved (hours)** — verbatim SQL logic:
```sql
SUM(CASE
  WHEN question = 'Based on your experience, estimate how much total coding time you
                   saved using copilot for this PR? (in minutes)' THEN response::float / 60
  WHEN question = 'On average how many hours per day did you save in the past week?'
                   THEN response::float * 5
END)
```
  (per-PR anchored estimate + weekly estimate × 5 workdays; also shown cumulatively by week).
- **"Net Promoter Score" per task type** = `(count(Agree ∪ Strongly Agree) − count(Disagree ∪ Strongly Disagree)) / count(*)` over Likert responses, task type regex-extracted from the question text.
- Survey schema (`V014`) has `survey_User` (respondent) → responses are per-person joinable but only shown in aggregate.

### 4. DORA (`dora_deployments_incidents.json` + dbt)
- **Lead time** (large native SQL): per merged PR, latest pre-merge review defines `pr_review_time = review.submittedAt − pr.createdAt`, `pr_merge_time = mergedAt − submittedAt` (falls back to whole span if no review); merge commit → artifact → deployment; changesets are commits between consecutive deployments per (env, application); stage times dev→qa→staging→prod from first deployment timestamps per env; final row: `lead_time = prod_deployed_at − pr_created_at` **plus an explicit `unaccounted_time` residual** = lead_time − (review + merge + dev + qa + staging).
- **CFR** = `sum(type='Incident') / sum(type='Deployment')` over a bare UNION of incident and deployment rows — incidents are not linked to the deployments that caused them.
- **MTTR** = `avg(resolvedAt − createdAt)` over resolved incidents.
- dbt heuristics: a "deployment" is any `cicd_Build` whose pipeline `name LIKE '%deploy%'` (`deployment.sql`); an "incident" is any `tms_Task` with `typeDetail = 'Incident'` (`incident.sql`).

### 5. Git/PR/build dashboards
- PR Cycle Time = `avg(mergedAt − createdAt)` in days (native SQL).
- **Top 5 Contributors** = PR count grouped by `vcs_User.name` — the only per-person view in the product, a small leaderboard.
- Build Reliability = `avg(CASE WHEN statusCategory='Success' THEN 1 ELSE 0 END)` per build (latest status only; no flake/rerun handling).
- `task_creators.sql` (dbt): per-person monthly task-creation counts with `rank() OVER (PARTITION BY month, year ORDER BY num_tasks_created DESC)` — a per-person ranking table (apparently a demo/custom-metric example).

## Data it assumes

- A **canonical, source-agnostic warehouse schema**: `vcs_*` (PRs: created/merged timestamps, linesAdded/linesDeleted, commitCount, commentCount, author, mergeCommit), `cicd_*` (builds with statusCategory, pipelines, deployments, artifact↔commit↔deployment association tables), `tms_*` (tasks), `ims_*` (incidents), `survey_*`, populated by Airbyte connectors.
- **AI-tool data model** (the transferable part): `vcs_UserTool` = (user, org, tool{category,detail}, startedAt, endedAt, inactive) — seat/entitlement windows from the GitHub Copilot seats API (`faros_copilot_seats` stream); `vcs_UserToolUsage` = per-user usage events with optional repo/branch/file and `charactersAdded` (`faros_copilot_usage` stream); `faros_MetricDefinition/faros_MetricValue/faros_Tag` = generic pre-aggregated metric store for vendor API metrics at org/team granularity.
- **Identity**: separate per-source user tables (`vcs_User(source, uid, name, email, type)`, `tms_User`, `ims_User`, `survey_User`) — **no cross-source identity resolution in CE** (that is a paid-Faros feature); only `vcs_UserEmail` (email↔vcs user, V013) as a join aid. `vcs_User.type` (jsonb, could mark bots) exists but **is never filtered on anywhere**.

## Design choices relevant to OUR problem

| Ours | Faros CE's answer |
|---|---|
| No AI-vs-human attribution per PR | Doesn't attempt it. Attributes at **person-period** level: PR inherits "with tool" if author held a seat when the PR was created. Cohort A/B ("with Tool" vs "No Tool") instead of per-artifact attribution. |
| Quality adjustment | Essentially none on the AI dashboards. Quality lives in *separate* metrics (build reliability, CFR, MTTR) never composited with output. No churn/rework metric anywhere. |
| Normalization / cohorting | Per-author normalization (`PRs / distinct authors`); medians for size; breakouts by team/editor/language — but **no role/level/function cohorts** and no percentiles. |
| Per-person vs per-team | Deliberately team/cohort-level. No per-engineer score exists. Only per-person artifacts: Top-5-Contributors PR leaderboard, task-creator rank table. |
| Gaming defenses | **None** — no winsorization, no caps, no bot exclusion, no rerun handling. Defensible only because nothing is a personal score. |
| Missing CI / force-push / bots | Not handled. Build Reliability uses whatever builds exist; bots flow into PR counts and contributor leaderboards; only latest build status is kept. |
| One score vs several | Several dashboards, three lenses — Adoption/Usage, Impact (delivery metrics), Surveys (self-report) — never collapsed into one number. |
| Cost/leverage denominator | **No dollar or token denominator anywhere.** "Impact" is directional cohort deltas, not efficiency ratios. |

## Gaming & fairness analysis (required, per metric)

- **PRs merged per user (tool cohort)** — (a) Gamed by slicing work into many trivial PRs; at cohort level, self-selection bias does the "gaming" for you (eager adopters ≠ average engineers), and seat-holding ≠ actual use (idle seat-holders dilute the tool cohort). (b) Unfairly penalizes engineers doing large/complex/non-PR work (infra, agents, reviews) and inflates functions with naturally small PRs.
- **Acceptance rate (accept lines / generated lines)** — (a) Gamed by accepting suggestions then immediately deleting/rewriting (counts as accepted); also editor/telemetry configuration. (b) Penalizes people working in languages/domains where the assistant is weak and reveals nothing about outcome quality. [inference]
- **Median PR size / net lines** — (a) Medians resist single huge PRs but not a systematic habit of padded generated code; "net lines added" rewards additions over healthy deletions. (b) Penalizes refactoring/deletion-heavy work, config-file engineers.
- **Merge time** — (a) Gamed via rubber-stamp reviews, merging trivial PRs, pre-approving offline. (b) Penalizes teams with rigorous review culture or timezone-spread reviewers; avg (not median) is outlier-sensitive.
- **Self-reported time saved** — (a) Trivially inflated (no verification; the ×5 weekly multiplier amplifies exaggeration). (b) Penalizes honest/conservative estimators; per-PR anchoring ("for this PR, in minutes") partially mitigates by tying the estimate to a concrete artifact. [inference]
- **Build reliability avg(success)** — (a) Rerun-until-green (only latest status stored), batching risky changes into fewer builds. (b) Penalizes repos with flaky infra or many independent CI jobs — the same failure mode as our CI-clean-rate.
- **CFR = incidents/deployments** — (a) Deploy more trivial changes to inflate the denominator; reclassify incidents as bugs. (b) Penalizes teams that diligently record incidents.
- **MTTR** — (a) Close fast/reopen; timestamp hygiene. (b) Penalizes honest incident tracking, long-tail severity mixes.
- **Top-5-Contributors PR-count leaderboard** — (a) PR-splitting. (b) Penalizes reviewers/mentors/non-PR roles; the exact leaderboard pattern our brief forbids.

## What we should steal

1. **Person-period attribution windows as the core primitive.** `vcs_UserTool(startedAt, endedAt)` sidesteps per-PR AI attribution exactly the way our constraint demands. Our per-user/per-day usage data is a *stronger* version (actual activity, not seat entitlement): define engineer-weeks as "AI-heavy / AI-light" from spend/sessions and compare that person's own output metrics across periods (within-person contrast beats Faros's cross-person cohort, which suffers adoption self-selection). [inference]
2. **Normalize by exposed population and use medians** (`distinct PR / distinct author`; median lines added/deleted) — never sum raw output across unequal cohorts.
3. **Three-lens separation without a composite score**: Adoption/Usage vs Delivery Impact vs Self-report sentiment as sibling views. Directly supports our "one score is not sacred" framing.
4. **Per-PR-anchored self-report time-saved question** ("estimate coding time saved using copilot *for this PR*, in minutes") — the cheapest credible outcome signal for agent-platform/non-PR engineers our data can't see; a strong candidate for "new signals worth acquiring."
5. **Lead-time decomposition with an explicit `unaccounted_time` residual** — an interpretability pattern worth copying: always show the residual your model can't explain instead of silently normalizing it away.
6. **Tool-agnostic AI data model**: tool as `{category, detail}` jsonb spanning any assistant + a generic `MetricDefinition/MetricValue/Tag` store for vendor-API aggregates — matches our "all AI tools incl. agent platform" inventory and keeps the metric layer stable as tools churn.
7. **`vcs_UserToolUsage.charactersAdded` with repo/file context** — evidence that per-user, per-event AI-output volume telemetry (without code contents) is an established schema pattern, not exotic.

## What we should avoid

1. **Acceptance rate as a quality or leverage signal** — volume of accepted suggestions, blind to whether the code survived; Faros wisely keeps it in "usage", never "impact".
2. **No bot filtering / no identity resolution** — `vcs_User.type` exists but is never used; per-source user tables are never merged in CE. A per-engineer score must do both explicitly (bot exclusion; email↔SSO↔tool-account identity merge like `vcs_UserEmail` but enforced).
3. **The seat-window join's double-count hazard**: a user with multiple/overlapping `vcs_UserTool` rows yields one joined row per assignment, so the same PR can land in both the "Tool" and "No Tool" buckets in aggregates. Windowed attribution needs dedup rules per (entity, period). [inference from the MBQL join structure]
4. **Name-based heuristics for stage classification** (`pipeline name LIKE '%deploy%'`) — silently wrong coverage per repo; the same trap as inferring "AI work" from tool names.
5. **Ratio of unlinked counts** (CFR = incidents/deployments with no causal link) — the same structural flaw as "output per AI dollar" with no link between numerator and denominator.
6. **Latest-status-only CI signal** (rerun-until-green invisible) — reproduces our CI-clean-rate saturation problem; if CI is used at all, count first-attempt outcomes, not final.
7. **PR-count leaderboards** (Top 5 Contributors, task-creator ranks) — the exact anti-pattern our privacy constraint bans.
8. **Seat-assignment ≠ usage** as an exposure definition — dilutes cohorts with idle license holders; we have real usage data, so use it.

## Bottom line for our design

Faros CE — a real engineering-ops vendor's OSS product — **never attempts a per-engineer AI leverage score and never divides output by cost**. Its AI-impact story is: (1) adoption/usage telemetry, (2) team-level cohort deltas on delivery metrics attributed by person-period tool windows, (3) self-reported time savings and sentiment. That triangulated, cohort-level, multi-lens architecture is the strongest external validation yet that our redesign should move away from "quality-adjusted output per dollar" toward within-person period contrasts plus self-report anchoring — while adding the per-person cohorting, bot/identity hygiene, and gaming defenses Faros never needed because it never scored individuals.
