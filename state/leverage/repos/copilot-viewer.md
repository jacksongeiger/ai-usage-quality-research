# Repo read: copilot-metrics-viewer (GitHub's official Copilot dashboard)

## Repo & commit
- URL: https://github.com/github-copilot-resources/copilot-metrics-viewer [vendor — GitHub-affiliated resources org, MIT]
- Clone: `git clone --depth 1` OK at canonical URL (no rename/successor). Local: `/private/tmp/claude-501/-Users-jacksongeiger/044a2943-cb9f-4a0f-82cf-62b1372890ad/scratchpad/repos/copilot-viewer`
- Commit read: `9f42443bb56764358e7a484127558369ffe2ffed` (2026-07-08, "fix(#410): compute rolling active-user windows for team metrics"). App version v3.2.0 (TRANSPARENCY_DISCLOSURES.md).
- Stack: Nuxt/Vue/TypeScript. v3.0+ uses the **new download-based "Copilot Usage Metrics API"**; the legacy `/copilot/metrics` REST API **shut down 2026-04-02** (README.md, `shared/utils/metrics-util-v2.ts` header comment).
- API docs cited in code (URLs from code comments; not independently fetched — unverified by direct fetch): https://docs.github.com/en/enterprise-cloud@latest/rest/copilot/copilot-usage-metrics [primary], https://docs.github.com/en/copilot/reference/copilot-usage-metrics/copilot-usage-metrics [primary].

Files read: `app/model/Copilot_Metrics.ts`, `app/model/MetricsToUsageConverter.ts`, `shared/utils/metrics-util.ts`, `shared/utils/metrics-util-v2.ts`, `shared/utils/feature-classification.ts`, `server/services/github-copilot-usage-api.ts`, `server/services/user-metrics-aggregator.ts`, `server/services/report-transformer.ts`, `server/api/user-metrics.ts`, `server/api/my-usage.get.ts`, `server/utils/billing-spend-aggregator.ts`, `server/utils/restrict-user-rows.ts`, `app/components/MetricsViewer.vue`, `app/components/UserMetricsViewer.vue`, `app/components/MyUsageViewer.vue`, `app/components/PullRequestViewer.vue`, `app/components/AgentModeViewer.vue`, `README.md`, `TRANSPARENCY_DISCLOSURES.md`.

## What the official API actually returns (as modeled by this repo)

### Granularity
- **Org/enterprise aggregate**: per-day `ReportDayTotals` via 1-day and 28-day report downloads (signed URLs, JSON). Rolling **28-day window max** from the API; anything longer requires your own DB ("historical mode": Postgres + daily sync service).
- **Per-user**: `users-1-day` / `users-28-day` endpoints return **NDJSON, one record per user per active day** (`UserDayRecord`). Large orgs are sharded across multiple files; the same user appears in each shard.
- Identity: `user_id` + `user_login` (GitHub login) only. Requires org "Copilot metrics: Read" permission.

### Per-user, per-day fields (`server/services/github-copilot-usage-api.ts` `UserDayRecord`, lines 160–197)
- `user_initiated_interaction_count`, `code_generation_activity_count`, `code_acceptance_activity_count`
- `loc_suggested_to_add_sum`, `loc_suggested_to_delete_sum`, `loc_added_sum`, `loc_deleted_sum` (adds and deletes tracked SEPARATELY, for suggested and accepted)
- `ai_credits_used` (per-user spend; added by GitHub **2026-06-19**; optional — `undefined` means "no data", explicitly distinguished from 0 in `aggregateUserDayRecords`)
- `used_agent` / `used_chat` / `used_cli` booleans
- Breakdowns: `totals_by_ide` (with last-known IDE/plugin versions), `totals_by_feature`, `totals_by_language_feature`, `totals_by_model_feature`
- `totals_by_cli`: session_count, request_count, prompt_count, `token_usage {output_tokens_sum, prompt_tokens_sum, avg_tokens_per_request}` — code comment calls this "the most authoritative per-user token signal GitHub exposes"
- `ai_adoption_phase {phase_number 0–3, phase, version}` — **GitHub's own per-user adoption cohort classifier**. Code comment: docs publish only labels ("No Cohort", "Phase 1–3") — "The docs do not publish the activity criteria GitHub uses to assign each phase."
- Feature taxonomy (`shared/utils/feature-classification.ts`): `code_completion`; chat = `chat_panel_{agent,ask,edit,custom,unknown}_mode`, `chat_inline`; agent = `chat_panel_agent_mode`, `agent_edit`.

### Aggregate-only fields (NEVER per-user) — `ReportDayTotals` lines 37–86
- `daily/weekly/monthly_active_users`, `monthly_active_chat_users`, `monthly_active_agent_users`
- `pull_requests` block: `total_created`, `total_reviewed`, `total_merged`, `median_minutes_to_merge`, `total_created_by_copilot`, `total_merged_created_by_copilot`, `median_minutes_to_merge_copilot_authored`, `total_suggestions`/`total_applied_suggestions` (review suggestions). **PR outcomes exist ONLY at org level — GitHub does not attribute PR outcomes to individual users.**

## What the dashboard computes (verbatim-level formulas)

1. **Acceptance rate by count** (org: `MetricsViewer.vue` ~477; per-user: `UserMetricsViewer.vue` 853–854):
   `acceptance_rate = code_acceptance_activity_count / code_generation_activity_count * 100` (0 if denominator 0 — never a "neutral default").
2. **Acceptance rate by lines**: `total_lines_accepted / total_lines_suggested * 100`.
3. **Trend**: window split in half; compare mean of first half vs second half (`computeTrend`, `MetricsViewer.vue` ~500). No regression, no seasonality.
4. **Engagement cohorts** (`UserMetricsViewer.vue` 673–676): All / **Active (≥7 active days)** / Occasional (1–6 days) / Inactive (0 days) within the window. Adoption % = active/total seats with a roster denominator (see stubs below).
5. **Team metrics** (`server/services/user-metrics-aggregator.ts`): no team endpoint exists → filter per-user day records by team-member logins (case-insensitive), sum per day; **rolling 7-day/28-day distinct-user sets recomputed per day at the team level** (fix #410 — naive per-day sums overstate WAU/MAU).
6. **Roster zero-fill** (`server/api/user-metrics.ts` `filterByTeamIfNeeded`): team members with no usage get zero-filled stub rows "to support accurate adoption calculations (e.g. '6 of 10 members active')" — denominators are the full roster, not just users who appear in telemetry.
7. **Shard dedup** (issue #398, `fetchLatestUserReport`): with >1 download file, per-user totals are re-derived from merged day records; naive concat would double-count users.
8. **Spend** (`server/utils/billing-spend-aggregator.ts`): sums both **net** (post-discount, what's billed) and **gross** (list-price consumption) per model, sorts by gross — because fully discounted plans return net=0 on every row while gross reflects real usage ("'0 credits used' despite thousands of daily calls").
9. **Language×model per-user cross-product is NOT in the API** → `buildLanguageModelTotals` distributes each model's completion activity proportionally across languages by each language's share — explicitly labeled "an approximation suited for visualization".
10. **Normalization knobs**: optional weekend + locale-aware holiday exclusion before charting/averaging (`app/utils/dateUtils.ts`); date-range filter to 100 days (historical mode only).
11. **Semantics warning in the transformer** (`server/services/report-transformer.ts` header): `code_generation_activity_count` maps to old `total_code_suggestions` but counts "activities, not individual suggestions"; `loc_added_sum` spans ALL features and must be feature-scoped ("code_completion") for accurate completion LOC. I.e., **the vendor changed metric semantics between API versions; acceptance rates are not comparable across the v1/v2 boundary** (inference from that comment).

## Data it assumes
- GitHub-hosted org/enterprise; Copilot-only telemetry (single vendor — no cross-tool view).
- 28-day rolling API window; longer history is the DEPLOYER's problem (Postgres + daily sync).
- Identity = GitHub login; SAML/Microsoft Graph services exist for enterprise display-name mapping (`server/services/github-saml-service.ts`, `microsoft-graph-service.ts`).
- Per-user spend only since 2026-06-19 (`ai_credits_used`) plus a billing endpoint per login; both optional.
- No repo/PR/CI join at user level; no seniority/role/team metadata beyond GitHub team membership.

## Design choices relevant to OUR problem
- **Quality adjustment: none.** The biggest vendor ships volume, acceptance funnel, mix, spend, active days, and an opaque adoption phase — no productivity, quality, or value claim per user. PR outcomes (incl. "created by Copilot", median-minutes-to-merge) surface ONLY as org aggregates. (Inference: GitHub does not consider per-user output/quality attribution defensible.)
- **Churn-adjacent signal exists**: `loc_suggested_to_delete_sum` / `loc_deleted_sum` are first-class, separate from adds — the funnel suggested→accepted→added is measurable per user per day, but it stops at the editor buffer (pre-review, pre-CI, pre-merge).
- **Identity**: case-insensitive login matching everywhere; rows with missing/empty login are NEVER returned to non-admins ("a legacy sentinel like '' cannot wildcard-match").
- **Privacy**: `restrict-user-rows.ts` (opt-in, `NUXT_USAGE_ADMINS`, "Austrian/EU compliance" issue #398): non-admins see only their own row; there is a dedicated self-view endpoint `/api/my-usage` (403 on `?login=other` for non-admins). Comment calls the filter "the LOAD-BEARING security primitive". Matches our brief's strictly-self-view constraint.
- **Gaming defenses: essentially none** — no winsorization, no caps, no outlier handling. Defense-by-abstention: the vendor avoids per-user claims that would be worth gaming.
- **Missing-data honesty**: `ai_credits_used` kept `undefined` (not 0) until at least one day reports it — "no data" ≠ "zero". Contrast with our current metric's CI-clean default of 1.0.

### Gameability / fairness of each metric this stack surfaces
- **Acceptance rate (count or lines)**: gamed by accepting suggestions then deleting them (acceptance is logged at accept-time; the add/delete LOC split only partially exposes this) or by working only where completions are easy. Penalizes careful reviewers who reject more, and engineers in languages/frameworks where Copilot is weak.
- **LOC suggested/accepted**: gamed via boilerplate-heavy files where completions are long. Penalizes terse languages, config/infra work, deletion-heavy refactoring (deletes shown but every headline chart uses adds).
- **Active days / engagement cohort (≥7 days)**: gamed by triggering one trivial completion daily. Penalizes part-timers, on-call rotations, meeting-heavy senior roles.
- **AI credits / tokens**: as a usage signal, gamed by spamming expensive models; as any denominator, penalizes people doing legitimately harder work (same flaw as our per-dollar metric).
- **ai_adoption_phase**: criteria unpublished → hard to game deliberately, but also uninterpretable and unappealable; penalizes engineers whose AI use happens outside GitHub tooling entirely.
- **Org-level Copilot-PR merge stats**: if ever made per-person, gamed by fast-merging trivial Copilot PRs; penalizes teams with rigorous review (long time-to-merge reads as bad).

## What we should steal
1. **Funnel framing**: suggested→accepted→retained-in-buffer LOC, with adds and deletes tracked separately — a per-user, code-free churn precursor. Extend the same shape to our data: usage → PR → CI-clean → merged.
2. **Adoption-depth profile from booleans + feature mix** (`used_chat/used_agent/used_cli`, feature/model breakdowns): describes HOW someone uses AI without claiming output value — fits our agent-platform engineers whose output is invisible.
3. **Roster zero-fill for denominators**: compute adoption/cohort stats over the full roster, not just telemetry-visible users.
4. **Undefined ≠ zero** for missing signals (vs. our CI-clean-rate defaulting to a fake-neutral 1.0).
5. **Gross vs net spend**: any per-dollar metric must use metered consumption (gross), not billed dollars — discounts/seat deals make net spend garbage as a denominator.
6. **Rolling distinct-user windows** (7/28-day sets per day) instead of snapshot counts; and re-aggregating from atomic day records when merging shards (dedup discipline).
7. **Privacy architecture**: self-view endpoint + tiny admin allowlist + never returning identity-less rows + banner explaining restricted view (works-council-compatible defaults).
8. **Normalization knobs users control**: weekend/holiday exclusion, date-range choice — interpretability through user-adjustable windows rather than hidden corrections.
9. **"Adoption phase" as a concept** (coarse, ordered cohort label instead of a continuous score) — but unlike GitHub, publish the criteria.

## What we should avoid
1. **Acceptance rate as a quality/leverage proxy** — it stops at the editor buffer; and GitHub itself redefined the counters ("activities, not individual suggestions") across API versions, silently changing every downstream rate.
2. **Opaque vendor cohorting as a score input** (`ai_adoption_phase` with unpublished criteria) — fails our interpretability requirement.
3. **Proportional-distribution approximations presented as data** (language×model backfill) — if a cross-cut isn't measured, show it as unmeasured.
4. **Treating accepted/added LOC as delivered output** — pre-review, pre-CI; would recreate our lines-changed problem one step earlier.
5. **Single-vendor telemetry as the spend/usage universe** — this stack can't see non-Copilot tools; our allowlist-denominator critique applies to the vendor's own dashboard.
6. **Trend = first-half vs second-half mean** — too coarse for a personal score that people will act on (fine for a wall dashboard).
