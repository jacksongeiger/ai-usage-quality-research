# Repo notes: Google Four Keys (dora-team/fourkeys)

## Repo & commit
- URL: https://github.com/dora-team/fourkeys [primary]
- Status: **archived / unmaintained** — README line 1: "This repository is not currently maintained. We encourage you to explore it, fork it, or otherwise use it as inspiration for your own metrics instrumentation." No renamed successor repo; DORA points people to the [DORA Quick Check](https://www.devops-research.com/quickcheck.html) [primary] for survey-based assessment instead. Clone succeeded (no redirect).
- Cloned (depth 1) to: `/private/tmp/claude-501/-Users-jacksongeiger/044a2943-cb9f-4a0f-82cf-62b1372890ad/scratchpad/repos/fourkeys`
- HEAD commit: `6cc642e037a95a280ed8d3b1624dd003b647d797` (2024-01-23, "Update CONTRIBUTING.md")
- Files read (paths relative to clone root):
  - `queries/changes.sql`, `queries/deployments.sql`, `queries/incidents.sql`, `queries/events.sql`, `queries/function_multiFormatParseTimestamp.sql`
  - `dashboard/fourkeys_dashboard.json` (Grafana panels; the actual metric SQL lives in `rawSql` of each panel)
  - `METRICS.md` (documents the same SQL + rationale)
  - `bq-workers/github-parser/main.py`, `shared/shared.py`, `event-handler/event_handler.py`
  - `terraform/modules/fourkeys/queries/*.sql` — byte-identical to `queries/*.sql` (verified with diff)

## Architecture (how data flows)
Webhooks (GitHub/GitLab/CircleCI/Tekton/ArgoCD/Cloud Build/PagerDuty) → Cloud Run `event-handler` (HMAC signature check, `event_handler.py:43-54`) → Pub/Sub → per-source parser (`bq-workers/*/main.py`) → BigQuery `four_keys.events_raw` (raw JSON payload stored whole; dedup by SHA-1 signature, `shared/shared.py:is_unique`) → three derived tables built by `queries/*.sql` (`changes`, `deployments`, `incidents`) → Grafana dashboard computes the four metrics in SQL at query time.

## What it computes (verbatim where short)

### Derived table: changes (`queries/changes.sql`, complete)
```sql
SELECT
source,
event_type,
JSON_EXTRACT_SCALAR(commit, '$.id') change_id,
TIMESTAMP_TRUNC(TIMESTAMP(JSON_EXTRACT_SCALAR(commit, '$.timestamp')),second) as time_created,
FROM four_keys.events_raw e,
UNNEST(JSON_EXTRACT_ARRAY(e.metadata, '$.commits')) as commit
WHERE event_type = "push"
GROUP BY 1,2,3,4
```
A "change" = a commit SHA seen in a push event. GROUP BY dedups repeated SHAs (so a force-push replay of the same SHA doesn't double count, but a force-push that REWRITES SHAs creates brand-new "changes" — no force-push handling exists anywhere in the repo; grep for "force" returns nothing).

### Derived table: deployments (`queries/deployments.sql`)
A deployment = a *successful* deploy event, per source-specific filter, e.g.:
- Cloud Build: `JSON_EXTRACT_SCALAR(metadata,'$.status') = "SUCCESS"`
- GitHub: `event_type = "deployment_status" AND ...deployment_status.state = "success"`
- GitLab pipeline/deployment: status `= "success"`
- CircleCI: `workflow.name LIKE "%deploy%" AND workflow.status = "success"` ← string-match on workflow *name*
- ArgoCD / Tekton: status SUCCESS.
Each deployment is joined back to the commits it shipped: `main_commit` (the deployed SHA) plus GitHub's `additional_sha` array, then it unnests the push event's `$.commits` array to get the full list of `changes` per deploy. Failed deployments are absent from the table entirely.

### Derived table: incidents (`queries/incidents.sql`)
An incident = a GitHub/GitLab issue labeled exactly `Incident` (regex on labels JSON: `REGEXP_CONTAINS(...,'"name":"Incident"')`) or any PagerDuty event. Root-cause linkage is a **free-text convention**: `REGEXP_EXTRACT(metadata, r"root cause: ([[:alnum:]]*)")` — someone must type "root cause: <commit-sha>" in the issue body. `time_created = MIN(root-cause deploy time, issue open time)`; `time_resolved = MAX(closed_at)`. `HAVING max(bug) is True` keeps only incident-labeled issues.

### Metric 1 — Deployment Frequency (dashboard panel "Deployment Frequency")
Not a raw count: a **bucket over a 3-month window with zero-filled days** (`GENERATE_DATE_ARRAY` LEFT JOIN so no-deploy days exist as rows):
```sql
CASE WHEN daily THEN "Daily" WHEN weekly THEN "Weekly"
     WHEN PERCENTILE_CONT(monthly_deploys, 0.5) OVER () >= 1 THEN "Monthly"
     ELSE "Yearly" END
-- daily  := median days-with-a-deploy per week >= 3
-- weekly := median weeks-with-a-deploy >= 1
```
Thresholds map to 2019 State of DevOps buckets (METRICS.md).

### Metric 2 — Lead Time for Changes
Median (PERCENTILE_CONT 0.5) of `TIMESTAMP_DIFF(deploy.time_created, change.time_created, MINUTE)` over every (deployment, commit) pair, with one filter:
```sql
IF(TIMESTAMP_DIFF(d.time_created, c.time_created, MINUTE) > 0, ..., NULL) # Ignore automated pushes
```
i.e. **non-positive lead times are nulled out** — the repo's only "bot/automation" defense. METRICS.md explains: a PR merge creates a push on main that is "not its own distinct change"; deploys triggered off it would have ~0 lead time and "artificially skew the metrics." Bucketed at <1 day / <1 week / <1 month (730h) / <6 months / else.

### Metric 3 — Time to Restore Services
`PERCENTILE_CONT(TIMESTAMP_DIFF(time_resolved, time_created, HOUR), 0.5)` over incidents, 3-month window, bucketed same way.

### Metric 4 — Change Failure Rate
```sql
IF(COUNT(DISTINCT change_id) = 0, 0,
   SUM(IF(i.incident_id is NULL, 0, 1)) / COUNT(DISTINCT deploy_id)) as change_fail_rate
```
Deployments unnested to per-commit rows, LEFT JOIN to incidents on commit SHA; numerator = count of (deploy,commit) rows that match an incident, denominator = distinct deploys. Note numerator/denominator granularity mismatch: one deploy with many incident-linked commits can push the "rate" above 1.0 (my inference from the SQL; unhandled edge case). Buckets: `<=0.15 → "0-15%"`, `<0.46 → "16-45%"`, else `"46-60%"`.

## Data it assumes
- Webhook access to the CI/CD + VCS + incident systems, with success/failure states and commit SHAs in payloads.
- Deploy events carry the shipped commit SHA(s) (GitHub `deployment.sha` + `additional_sha`; CircleCI relies on a `%deploy%` workflow-name convention).
- Humans follow two labeling conventions: an `Incident` issue label, and "root cause: <sha>" free text. Without them, MTTR and CFR are empty/zero.
- Timestamps parse (a whole UDF, `multiFormatParseTimestamp`, exists just to handle GitLab's 3+ timestamp formats — real-world messiness acknowledged in code).
- Missing-CI case: if a repo never emits deploy events, it simply produces no rows — silently absent, not flagged.

## Design choices relevant to OUR problem
1. **Strictly team/pipeline-level. Zero per-person attribution.** No author, actor, or user field appears in any query (verified by grep over `queries/*.sql` and the dashboard JSON). Google's canonical DORA implementation deliberately never computes an individual metric — consistent with DORA's own guidance that these are team capabilities. Directly relevant precedent AGAINST per-engineer leverage scores built from delivery metrics.
2. **Medians everywhere, never means** (`PERCENTILE_CONT(...,0.5)`), plus coarse research-anchored buckets (Daily/Weekly/…, 0-15%/…) instead of continuous scores. Robust to outliers; kills false precision; interpretable ("you are 'Weekly'" beats "you are 62.3").
3. **Zero-filling absent periods** before aggregating frequency (`GENERATE_DATE_ARRAY` LEFT JOIN) — otherwise "median deploys/day" is computed only over active days and flatters bursty teams.
4. **Success-filtered denominators**: only *successful* deploys count as deployments. Analogy for us: only merged PRs count — same choice the client already makes.
5. **Quality = linked failures, not CI green-ness.** CFR ties quality to production incidents traced to specific changes, not to pre-merge CI status. That linkage is bought with human conventions (label + "root cause:" text) — i.e., even Google couldn't get change→failure linkage without asking humans to annotate. Matches our brief's hard limit (no per-author defect linkage) and shows the cheapest known workaround.
6. **Automation/bot handling is minimal and heuristic**: the single `> 0 minutes` lead-time filter. No bot-account filtering, no force-push handling, no backdated-commit handling.
7. **Idempotency/dedup by content hash** (SHA-1 signature per event, checked before insert) and webhook HMAC verification — integrity-of-pipeline defenses, not anti-gaming defenses.
8. **Windows**: rolling 3 months for buckets, daily series for trends. Longer than the client's 4 weeks; smoother and less cold-start-brittle.
9. **No normalization/cohorting at all** — no per-repo, per-team-size, or per-role adjustment. Fairness is achieved by *not comparing* (each team sees its own buckets), not by cohort math.

## Gameability & fairness (per metric, my analysis — inference)
- **Deployment Frequency** — Game: split releases into many tiny deploys; rename any CircleCI workflow to contain "deploy" (string match!) to mint deployments. Penalizes: teams shipping libraries/mobile apps with release trains (README itself says the tool "does not work well" for release-based projects); low-traffic-but-critical services.
- **Lead Time for Changes** — Game: rebase/squash just before deploy so commit timestamps are fresh (commit timestamps are author-controlled and forgeable); delete long-lived branches' history; deploy trivial changes frequently to pull the median down. Penalizes: teams doing careful long-running work (research, migrations); repos where commits sit in review — review time is invisible here, only commit→deploy.
- **Time to Restore** — Game: close the incident issue fast and open a "follow-up" issue; don't apply the `Incident` label at all (metric silently empties). Penalizes: teams that diligently label incidents vs. teams that don't (measurement = labeling discipline, not reliability).
- **Change Failure Rate** — Game: never write "root cause: <sha>" in incident text → CFR reads 0%; inflate deploy count (denominator) with no-op deploys. Penalizes: honest annotators; high-blast-radius teams where one bad deploy files many incidents (and the >1.0 rate bug above).
- Cross-cutting: everything is convention/webhook-fed with no adversarial validation — fine for a *self-measured team* dashboard (its stated use), dangerous the moment it's individual or comparative.

## What we should steal
- **Medians + research-anchored coarse buckets** instead of continuous percentile scores — robust, interpretable, less rank-anxiety. Map a leverage score to named tiers with published anchors rather than 0–100.
- **Zero-fill inactive periods** before computing any frequency/consistency metric (directly applicable to AI-usage active-days and PR cadence).
- **Rolling ~3-month window** — fixes the client's cold-start brittleness (problem #4 in BRIEF) better than a 4-week window with hard floors.
- **"Ignore non-positive / degenerate durations" nulling pattern** — cheap, explainable automation filter; analog: null out PRs with 0 review latency or usage days with degenerate token patterns rather than building a bot classifier.
- **Event-signature dedup + HMAC at ingestion** — integrity before analytics.
- **Honesty about convention-dependence**: METRICS.md states each metric's assumptions next to its SQL. Our design doc should co-locate formula + assumptions + known skew the same way.
- **The precedent itself**: cite that Google's reference DORA implementation is team-only, success-filtered, and bucket-based when arguing against per-engineer delivery-metric leaderboards.

## What we should avoid
- **String-convention detection** (CircleCI `LIKE "%deploy%"`, `"root cause: sha"` regex, `Incident` label) as a metric foundation — silently measures annotation discipline; trivially gamed by omission. Any "AI-assisted" tag we might reintroduce would have exactly this failure mode (the client already learned this — dropped unreliable AI-made tag).
- **Numerator/denominator granularity mismatches** like the CFR >1.0 bug — for us: PRs-per-dollar where the PR side is per-merge but spend is per-day invites the same class of error.
- **No identity model at all** — fine for fourkeys (team-level), fatal for a per-engineer score; we need explicit bot/service-account and multi-account handling, which fourkeys offers no pattern for.
- **Silence on missing data** — repos with no deploy events just vanish; our agent-platform engineers similarly "vanish" today (BRIEF problem #2). Absence must be surfaced as "unmeasurable," not scored as zero.
- **Assuming payload timestamps are trustworthy** — commit timestamps are user-controlled; prefer server-side event times (merge time, CI completion time) for any duration metric.
