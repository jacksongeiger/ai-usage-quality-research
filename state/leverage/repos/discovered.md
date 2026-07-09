# Discovered Implementations (not previously listed)

Sub-investigator report, 2026-07-08. Excludes already-covered: DevLake, fourkeys, middleware, Faros CE, CHAOSS, hercules, git-of-theseus, code-maat, copilot-metrics-viewer.
Clones under `/private/tmp/claude-501/-Users-jacksongeiger/044a2943-cb9f-4a0f-82cf-62b1372890ad/scratchpad/repos/`.

---

## DEEP DIVE 1 — satomic/copilot-usage-advanced-dashboard [practitioner, MIT]
https://github.com/satomic/copilot-usage-advanced-dashboard — Python + Elasticsearch + Grafana; feeds on GitHub Copilot per-user metrics API. **The only OSS project found that computes a composite per-engineer AI score.** Read from clone (`regenerate_adoption.py` L64–233; `src/cpuad-updater/main.py` ~L380–445):

- **Adoption Score** (leaderboard, 0–100): five signals, equal weight 0.2 each — (1) volume = user-initiated interactions, (2) interactions/active-day, (3) acceptance_rate = code_acceptance_activity_count / code_generation_activity_count, (4) avg LoC added per active day, (5) feature_breadth = count of days using agent + days using chat. Each signal **robust-scaled to the org's 5th–95th percentile** bounds. Then a multiplicative **consistency bonus**: ×(1 + min(0.1, 0.1 × active_days/max_active_days)). Finally **normalized so the org's top user = 100** (purely relative).
- **Activity Score A** (v1.9, per user, 0–100): `0.4×(active_days/range_days) + 0.35×norm(accepted_per_active_day) + 0.25×norm(chat_per_active_day)` (main.py L410–414).
- **Reliance Score B** (0–100 "proxy"): robust-scaled `loc_added / loc_suggested` — line-level acceptance rate of Copilot suggestions. Code comment explicitly concedes true reliance ("Copilot lines / total lines added") needs git data the API lacks (main.py L416–423).
- (a) **Gaming**: every input is usage-side — trigger many completions and accept indiscriminately (raises acceptance rate, LoC/day, volume simultaneously); open chat daily for the consistency bonus and the 0.4 active-days term. No output/quality term anywhere, so inflation is free.
- (b) **Unfairly penalizes**: non-Copilot-tool users (invisible); low-completion roles (reviewers, architects, SREs); agent-heavy/chat-light users under the A-score weights; everyone in an org with one hyperactive user (max-normalization deflates all other scores).
- **Takeaway for us**: it measures *adoption/reliance*, never *leverage* — no join to delivery output. But its mechanics are reusable: 5–95 pctile robust scaling, capped consistency bonus, and its honest "proxy" labeling.

## DEEP DIVE 2 — AlexSim93/pull-request-analytics-action [practitioner, MIT]
https://github.com/AlexSim93/pull-request-analytics-action — TypeScript GitHub Action; per-developer reports from **PR metadata only** (closest OSS thing to "PR-quality scoring from metadata"). Read from clone:

- **"Pull request quality" table** (`src/view/utils/createPullRequestQualityTable.ts`): per user — merged PR count; changes-requested received; discussions received split **agreed / disagreed / total**, where agreed = discussion-opening review comment got ≥1 👍 reaction and disagreed = ≥1 👎; review comments received. Raw counts side by side, deliberately NOT collapsed into one score.
- **PR size** (`calcPRsize.ts`, `getPullRequestSize.ts`): `additions + 0.2 × deletions`, bucketed xs(≤50)/s(≤200)/m(≤400)/l(≤700)/xl — deletions discounted 5:1 vs. additions.
- **Revert detection** (`checkRevert.ts`): branch-name regex `^revert-\d+` (GitHub UI's auto-generated revert branches) — a zero-cost metadata revert signal.
- Also: median/p75/p95 time-to-review/approve/merge with workday/weekend adjustment (`calcNonWorkingHours.ts`, `checkWeekend.ts`).
- (a) **Gaming**: farm 👍 reactions from teammates to inflate "agreed"; split work into many tiny PRs (more merged, smaller size buckets, fewer comments each); manual `git revert` from CLI evades the branch regex entirely.
- (b) **Unfairly penalizes**: engineers on high-scrutiny/central code (comments-received reads as low quality); discussion-heavy team cultures; conversely it *favors* deletion-heavy refactorers via the 0.2 coefficient.
- **Takeaway for us**: review-reaction (👍/👎) sentiment and revert-branch detection are metadata-only signals the client org likely already has but isn't using — candidate NEW signals. The deletion-discounted size formula is a better line proxy than raw lines-changed.

## DEEP DIVE 3 — coderbuds/ai-detector [vendor-open-sourced, MIT]
https://github.com/coderbuds/ai-detector — YAML rulesets detecting AI-assisted PRs/commits from metadata; rules for claude-code, github-copilot, cursor, aider, devin, windsurf, openai-codex, v0-dev, replit-ai. Read from clone (`rules/*.yml`):

- **Signals**: commit/PR-description footers (regex, e.g. `🤖 Generated with … Claude Code`), `Co-Authored-By:` trailers (e.g. `Claude … noreply@anthropic.com`), bot author emails/usernames, branch-name patterns (`cursor-*`), PR labels, HTML comments. Each pattern carries a confidence (100 for explicit markers; OpenAI Codex only 60 — markers ambiguous across tools).
- Vendor claims ~75% of AI-assisted PRs carry explicit markers and 0% false positives on 1,000+ test PRs (https://coderbuds.com/blog/open-source-ai-code-detection-yaml-rules [vendor], unverified independently).
- (a) **Gaming**: trivial in both directions — strip trailers/footers (tools allow disabling) if AI use is penalized, or add fake trailers if rewarded; detects *presence*, never *extent* of AI contribution.
- (b) **Unfairly penalizes/undercounts**: users of tools leaving no markers (Cursor defaults, ChatGPT paste) are invisible → any tag-based measure is an opt-out lower bound.
- **Takeaway for us**: this is the state of the art for open AI attribution and it validates BRIEF hard-limit #1 — do not rebuild the abandoned "AI-made" tag; attribution can at best be a soft, non-scored contextual annotation.

---

## Secondary finds (verified to exist; not deep-dived)
- **microsoft/copilot-metrics-dashboard** [vendor, MIT] — Azure "solution accelerator"; acceptance average, active users, seat utilization from Copilot Metrics + User Management APIs. https://github.com/microsoft/copilot-metrics-dashboard
- **satomic/copilot-metrics-4-every-user** [practitioner] — MITM-proxy capture of individual Copilot traffic for per-user data beyond the API; shows demand for per-user granularity, and a privacy anti-pattern for a self-view tool. https://github.com/satomic/copilot-metrics-4-every-user
- **Backstage community Copilot plugin** [practitioner] — Copilot adoption inside the dev portal: seats assigned/unused, inactivity at 7/14/28 days. https://github.com/backstage/community-plugins (plugin `copilot`)
- **DevoteamNL/opendora** [practitioner] — Backstage DORA plugin backed by Apache DevLake (derivative of already-covered DevLake). https://github.com/DevoteamNL/opendora
- **github-community-projects/issue-metrics** [vendor] — time-to-first-response / time-to-close per issue/PR, report as GitHub issue. https://github.com/github-community-projects/issue-metrics
- **vintasoftware/github-metrics** [practitioner] — script computing PR metrics incl. hotfix counts and merge rate (merged PRs / active developers). https://github.com/vintasoftware/github-metrics
- **athenianco/athenian-api** [practitioner; "Proprietary until further notice" license — readable, not reusable] — production-grade PR cycle-time/metrics engine from a defunct SaaS; deliberately team-level, refused individual scoring. https://github.com/athenianco/athenian-api
- **botcommits.dev** [practitioner; OSS status unverified] — live tracker of AI-attributed commits on public GitHub via trailers; reports growth to ~5.2M/mo Claude-attributed commits by Feb 2026 (site's own claim, unverified). https://botcommits.dev/
- **GitHub Copilot metrics GA (2026-02-27)** [vendor] — user-level API "analyze individual Copilot usage for a specific day" + code-generation dashboard (suggested/added/deleted lines). https://github.blog/changelog/2026-02-27-copilot-metrics-is-now-generally-available/

## Dead end (honest)
- **No OSS implementation of engineering "goodput" / quality-adjusted throughput exists** that I could find. The term is only implemented in LLM-serving benchmarks (https://github.com/vllm-project/vllm/issues/8782 [primary]) and Google's ML-training "ML Productivity Goodput" (https://cloud.google.com/blog/products/ai-machine-learning/goodput-metric-as-measure-of-ml-productivity [vendor]) — different domains. Anyone claiming an off-the-shelf engineering-goodput library is bluffing.

## Design-space implications
1. **No OSS project joins per-user AI usage with delivery output.** Adoption dashboards stop at usage; delivery dashboards stop at PRs. The client's "leverage" join has no reference implementation — we are designing something genuinely un-copied (inference).
2. **Reusable mechanics**: 5–95 pctile robust scaling + capped consistency bonus (satomic); deletion-discounted size (`adds + 0.2×dels`), 👍/👎 review-reaction sentiment, and revert-branch regex (AlexSim93) — the latter two are metadata-only candidate NEW signals for the client.
3. **Attribution is confirmed dead** as a scoring foundation: best open ruleset ≈75% recall, opt-out, presence-not-extent (coderbuds).
4. **Per-user suggested-vs-added LoC is now vendor-GA** for Copilot — a possible NEW signal, but tool-specific (Copilot only), so it can't be the backbone of a tool-agnostic leverage score (inference).
