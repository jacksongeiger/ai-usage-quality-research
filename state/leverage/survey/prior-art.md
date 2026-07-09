# PRIOR ART — What the May-2026 "AI Usage Quality" Mission Gives the July-2026 "Leverage" Mission

Source material: `deliverables/00_EXECUTIVE_SUMMARY.md`–`07_WORKED_EXAMPLE.md`, `sources/SOURCES.md` (all read in full, 2026-07-08).
Status of everything here: **carried over — re-verify before citing in the final deliverable.** Old-mission citation numbers `[n]` refer to `sources/SOURCES.md` and are reproduced in §5.
Anything not traceable to the old deliverables is labeled **(inference)**.

---

## 1. The 3-pillar framework and how it maps onto "leverage"

Old construct: **AI usage quality = durable, low-friction, well-matched leverage from AI** — a *latent construct* measured via a nomological network (construct validity [32][33]), inferred only where independent proxies from different method types agree. Volume (tokens, acceptance, LOC) structurally demoted: can never alone produce a positive finding ("volume gate").

| Old pillar | What it asks | Mapping onto the new "leverage score" (inference) |
|---|---|---|
| **P1 Leverage/Efficiency** | Is AI reducing friction? (cycle-time shape, self-reported time-saved) | The closest ancestor of the new question — but the old mission had **no cost/dollar dimension at all**. "Per-dollar" efficiency is new ground; old P1 measured friction, not economic efficiency. |
| **P2 Adoption Depth** | Shallow autocomplete vs deep agentic; right tool per task; engaged-not-just-active | Maps to the "adoption lens" the new brief asks to reconsider. Directly rescues **agent-platform engineers**: depth/breadth signals exist even when no PR output exists. |
| **P3 Durability/Landing-quality** | Does output survive reality? (survival, churn, first-pass CI, review friction, reverts) | Maps to the "quality-adjusted" half of the new metric. Old mission's verdict: durability is **the strongest quality pillar** and CI-pass is one of its **weakest members** — the new metric built on its weakest brick. |

Key structural rules worth carrying whole:
- **Triangulation**: trust only where ≥2 independent signals from different method types agree; single signals cap at Medium/Exploratory.
- **Volume gate**: tokens/LOC/PR-count/consistency never alone → a direct indictment of the current leverage formula, which is ~100% volume-per-dollar.
- **Trend-vs-personal-baseline, never cross-person** — old mission held cross-person comparison invalid due to confounds [37]. The new mission's percentile-within-cohort design violates this; the new brief permits cohorts but demands redesign (function-aware). Tension to resolve, not inherit.
- **Amplifier lens** (DORA 2025 [2]): individual signals are readable only against team/repo context; never blend team context into the individual score.
- **Confidence tiers** (High/Medium/Exploratory) + four hard gates (volume, confound, cold-start, contradiction).
- Old mission refused any single score (Goodhart [30], SPACE [3]). The new brief requires a score but allows subscores — carry the counterbalancing principle from DX Core 4 [6]: speed/output metrics only ever paired against a quality/experience counterweight.

## 2. Catalog metrics computable from THIS mission's HAVE data

New HAVE: per-user/day/tool dollars+tokens+sessions+active-days (ALL tools incl. agent platform); merged-PR count + lines changed; per-PR CI pass/clean; role/level/team.
NOTE (inference): the OLD mission assumed richer GitHub metadata (commit/PR/review **timestamps**, review outcomes, closed-unmerged PRs, line-level history) than the new HAVE list states. Metrics needing only that are in §2b as near-miss acquisitions, not HAVE.

### 2a. Computable now

| Metric (old id) | Old confidence tier | How an engineer games it | Who it unfairly penalizes |
|---|---|---|---|
| First-pass/clean CI rate (3.4) | **Medium**, "huge" confounds, personal-trend only | Run CI locally / iterate until green — AI agents do this automatically, so the rate saturates ~1.0 and stops discriminating (old catalog predicted the new mission's problems #1/#7) | Heavy-CI repos (50+ jobs), flaky-test repos [44] (~13% of failures flaky), engineers doing risky/experimental work |
| Tool-category mix — chat vs coding-CLI vs agent-platform shares (coarse analog of 2.1 feature-mix) | Old 2.1 was Medium-High on vendor mode telemetry; per-tool-category version is **Exploratory-Medium** (inference — coarser) | Performative sessions on the "deep" platform | Engineers whose role legitimately needs one category; teams with restricted tool allowlists |
| Tool-fit / portfolio breadth (2.2) | Exploratory-Medium; VOLUME-adjacent | Adopt tools for show | License-constrained teams; specialists whose one tool is the right tool |
| Usage consistency & recency (2.4) | Medium (consistency) / Exploratory (as quality); **VOLUME** | Trivial daily token generation | PTO, part-time, on-call rotations, project-phase variation |
| Raw tokens / dollars / sessions (2.6) | Exploratory; **VOLUME, high gaming risk** | Trivially inflated — and when used as a *denominator* (current metric), inverted gaming: suppress spend to look efficient | As numerator: no one directly; as denominator: legitimate heavy spenders on hard work (new problem #5); agentic tools burn 10–100× tokens for the same outcome [7] |
| Merged-PR count | **VOLUME** per SPACE "activity is the most misused dimension" [3] | PR-splitting into trivial units [13] | Agent/infra/platform engineers with no PR output; functions with big-PR norms; reviewers/mentors |
| Lines changed (LOC) | **VOLUME**, explicitly demoted; "count LOC at your peril" [41] | LOC padding [13]; generated/vendored code; winsorizing at 2000 is evaded by splitting one 10k-line change into six PRs (inference) | Config/infra work (tiny diffs), ML engineers (experiments not lines), deleters/refactorers (negative-LOC value) |
| PR size distribution (lines/PR) — derivable | Not in old catalog as such; small-batch is a DORA foundation [1][2] (inference) | Artificial splitting | Generated-code workflows, migrations, lockfile churn |
| Spend-per-PR / tokens-per-merged-PR ratios | Not in old catalog; old mission only used volume as normalization denominator | Suppress spend or split PRs | Hard-problem engineers, agent-platform builders (infinite ratio: spend with zero PRs) |
| Tokens-per-session / sessions-per-active-day (intensity proxies; loose 2.3 "engaged-not-just-active" analog) | Exploratory (inference — old 2.3 was Medium on vendor-defined "engaged" [21]) | Session churn / idle sessions | Long-single-session deep workers; tools that meter sessions differently |

### 2b. Near-miss: old-catalog metrics needing only cheap git/GitHub-metadata ingestion (feed the new mission's "new signals worth acquiring" section)

| Metric (old id) | Old tier | Why it matters for leverage | Gaming / unfair-penalty notes (old catalog) |
|---|---|---|---|
| Code churn/rework ≤2 weeks (3.2) | **Medium-High** — the GitClear AI red flag [11][12], computable from diff metadata alone | The strongest available quality-adjuster; replaces saturated CI-clean-rate | Gamed by avoiding revising own code or avoiding hard work; penalizes legitimate iterators, spike/exploratory work; **AI-attribution gap** — measures overall churn, not AI's [18] |
| Code survival/retention (3.1) | **High** (AI-attributed, Sourcegraph-native [25]) / **Medium** (git-reconstructed, overall) | "The single most defensible quality signal" | Gamed by timidity — avoid ambitious/refactor work to keep survival high; penalizes refactorers and experimenters |
| Revert/hotfix frequency (3.3) | Medium | DORA-stability-grade landing quality from commit-message metadata [42] | Sparse/noisy; reverts often process-caused, not individual |
| Review-friction trend (3.6) | Medium | Human-judgment quality signal CI can't give | Gamed by picking lenient reviewers/under-reviewing; penalizes hard-task owners, strict-team members |
| Cycle-time shape (1.1/1.2) | Medium | Friction reduction — leverage as time, not dollars | Force-push fakes first-commit time [13]; PR-splitting; penalizes cross-timezone teams, reviewer-starved repos |
| PR rejection/abandonment (3.7) | Exploratory | Wasted-effort signal (needs all PRs, not just merged) | Penalizes legitimate experimentation |
| Build-failure-to-fix latency (3.5) | Exploratory-Medium | Recovery/feedback-loop signal (needs CI timestamps) | Penalizes context-switchers, off-hours pushes |

Also carried: survey/self-report metrics (1.3/1.4 — time-saved, flow) as TURN-ON; old mission ranked a lightweight anonymized survey as the single highest-leverage addition (04 #2) and the mandatory perceptual counterweight [3][4][7].

## 3. Gaming-risk and confidence-tier machinery worth carrying over

- The gaming table (06 §1): acceptance/%-AI → route trivial edits through AI; tokens/consistency → daily filler; first-pass CI → push-only-when-green (arguably good); review friction → lenient reviewers; tool breadth → adopt for show. Documented mechanics catalog: GitClear "17 metrics and how to game them" [13] (LOC padding, micro-commits, force-push cycle-time fakery, assertion-free coverage, redefining "critical").
- **Denominator inversion (inference, new to this mission):** old gaming analysis covers inflating numerators; a per-dollar score adds a new class — *deflating the denominator* (use personal/free accounts, hoard a cheap allowlisted tool, avoid the agent platform because its spend counts against you). The current metric actively disincentivizes agent-platform adoption.
- Confidence-tier semantics worth reusing verbatim: metric-level tier (trustworthiness as proxy) is distinct from recommendation/score-level confidence (computed from cross-method agreement). Four hard gates precede any output: volume, confound (drop a tier + name it), cold-start (suppress, don't flip to a default — cf. new problem #4's brittle floors and the CI default-to-1.0 pathology), contradiction (surface tensions).
- Structural anti-gaming beats per-metric anti-gaming: no single exposed target; agreement across method types required; engineer-only audience removes most corruption pressure (Campbell [31]).

## 4. Validity/ethics traps that constrain the new design

1. **Goodhart/Campbell [30][31]** — any exposed score becomes a target. The new mission *must* ship a score → mitigations: self-view-only (the new brief already locks this), multiple counterbalanced subscores (DX Core 4 [6]), no targets/rewards, trend framing.
2. **Construct validity / volume≠quality [32][33][3]** — the current metric is output-volume-per-dollar percentile-ranked; it fails discriminant validity exactly as the old mission warned. Any new design needs a genuine quality/durability counterweight, and CI-clean-rate is not one (saturates).
3. **Confounds invalidate cross-person comparison [37]** — tenure/role/repo/task dominate. Level-only cohorts across all functions (new problem #6) are indefensible; if percentiles survive at all, cohorts must be function×level and the primary reading should stay trend-vs-own-baseline.
4. **Perception gap [17]** — self-report measures experience, never speed-truth (felt +20%, measured −19%). Use surveys as counterweight/calibration, not ground truth.
5. **AI-attribution gap (old precondition P4) [18][25]** — identical to the new brief's hard limit #1. Consequence: all durability/output signals are *overall*, AI merely contributes; label honestly; never claim to isolate "the AI's share."
6. **Timidity trap** — durability/quality signals reward trivial, unambitious work; must pair with size/ambition context (which the new mission lacks → honest open problem).
7. **Surveillance→enablement discipline [34][35][36][15]** — audience-lock architectural not policy; opt-in/deletable; aggregation/k-anonymity for any org rollup; coaching tone; affirm healthy usage, not only flag gaps; observer effect means the metric itself will shift behavior — aim it at behaviors you want.
8. **Amplifier confound [2]** — a good engineer in a weak environment shows bad downstream signals not of their making; context is a lens, never a score input.
9. **Unvalidated-instrument caveat** — the old framework was a literature-grounded design, never empirically validated; convergent validity was theoretical. Any new score needs a stated validation step before being trusted.
10. **Cold-start handling** — suppress/withhold with an explicit "insufficient data" state rather than hard floors that flip on/off (old X6 vs new problem #4) or defaults that masquerade as signal (CI=1.0 under 3 PRs).
11. **MVP discipline** — old conclusion: don't build the 20-metric catalog; ~4 signals, validate, expand. Transfers directly.
12. **Causal humility** — association only; never "AI made you faster/slower."

## 5. Carried-over source list (ALL: carried over — re-verify before citing; tags normalized to this mission's scheme)

Frameworks:
- [1] DORA 2024 Accelerate State of DevOps — https://dora.dev/research/2024/dora-report/ [primary] — +25% AI adoption ≈ −1.5% throughput, −7.2% stability.
- [2] DORA 2025 State of AI-assisted Software Development — https://dora.dev/dora-report-2025/ [primary] — "AI is an amplifier"; AI+throughput positive, stability still negative; AI Capabilities Model.
- [3] SPACE — Forsgren et al., ACM Queue 2021 — https://queue.acm.org/detail.cfm?id=3454124 [primary] — no single metric; activity most misused; perceptual dimension mandatory.
- [4] DevEx: What Actually Drives Productivity — Noda et al., ACM Queue 2023 — https://queue.acm.org/detail.cfm?id=3595878 [primary] — feedback loops / cognitive load / flow.
- [5] DevEx in Action — ACM Queue 2024 — https://queue.acm.org/detail.cfm?id=3639443 [primary; effect sizes previously unverified].
- [6] DX Core 4 — https://getdx.com/dx-core-4/ [vendor] — Speed/Effectiveness(DXI)/Quality/Impact; counterbalancing; individual "diffs per FTE" only with no-targets guardrails.
- [7] DX AI Measurement Framework — https://getdx.com/research/measuring-ai-code-assistants-and-agents/ [vendor] — Utilization/Impact/Cost (the only prior framework with a Cost axis — closest ancestor of "per-dollar"); code-gen volume "particularly susceptible to gaming"; ~82% use 3+ tools.
- [14] McKinsey "Yes, you can measure developer productivity" 2023 [primary, the anti-pattern] (no URL captured — re-locate).
- [15] Orosz + Beck response — https://newsletter.pragmaticengineer.com/p/measuring-developer-productivity [practitioner] — output metrics gamed, leak into reviews/layoffs.
- [16] Thoughtworks Radar — complacency with AI-generated code — https://www.thoughtworks.com/en-us/radar/techniques/complacency-with-ai-generated-code [practitioner].

Empirical AI-impact evidence:
- [8] GitHub Copilot RCT (Kalliamvakou 2022) — https://github.blog/news-insights/research/research-quantifying-github-copilots-impact-on-developer-productivity-and-happiness/ [vendor] — +55%, wide CI [21%,89%], greenfield task.
- [9] Ziegler et al. 2022 — https://arxiv.org/pdf/2205.06537 [vendor] — acceptance rate predicts *perceived* productivity.
- [10] Copilot retention/survival docs — https://docs.github.com/en/copilot/concepts/copilot-usage-metrics/copilot-metrics [vendor] — ~88% retention figure approximate.
- [11] GitClear Coding on Copilot (Jan 2024) — https://www.gitclear.com/coding_on_copilot_data_shows_ais_downward_pressure_on_code_quality [vendor] — churn from diff metadata alone.
- [12] GitClear 2025 (2024 data) — https://www.gitclear.com/ai_assistant_code_quality_2025_research [vendor] — refactoring ~25%→<10%; copy/paste 8.3%→12.3%; churn 5.5%→7.9%.
- [13] GitClear 17 gameable metrics — https://www.gitclear.com/popular_software_engineering_metrics_and_how_they_are_gamed [vendor] — gaming mechanics catalog.
- [17] METR RCT (Jul 2025) — https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ + arXiv 2507.09089 [independent] — −19% actual vs +20% believed; update https://metr.org/blog/2026-02-24-uplift-update/ (re-verify — this mission should fetch the update).
- [29] GitHub/Accenture study — https://github.blog [vendor] — +8.69% PRs/dev, 88% char retention (favorable bias).

Vendor telemetry (feature-mix/survival — mostly TURN-ON for the new mission but defines what's acquirable):
- [19] Copilot usage metrics reference — https://docs.github.com/en/copilot/reference/copilot-usage-metrics/copilot-usage-metrics [vendor] — mode mix, agent %, <5-seat exclusion.
- [20] Copilot Metrics REST API — https://docs.github.com/en/rest/copilot/copilot-metrics [vendor].
- [21] Copilot interpreting metrics — https://docs.github.com/en/copilot/reference/copilot-usage-metrics/interpret-copilot-metrics [vendor] — engaged vs active.
- [22] Cursor Analytics/Admin API — https://cursor.com/docs/account/teams/analytics-api [vendor] — intent Write/Ask/Plan, per-call tokens/cost, 90-day cap.
- [23] Claude Code OTel monitoring — https://code.claude.com/docs/en/monitoring-usage [vendor] — OFF by default; cost.usage, tool_decision accept/reject.
- [24] Amazon Q Developer metrics — https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/dashboard-metrics-descriptions.html [vendor].
- [25] Sourcegraph analytics — https://sourcegraph.com/docs/analytics [vendor] — native suggestion persistence at 30/120/300/600s; the only AI-attributed survival.
- [26] Tabnine reports — https://docs.tabnine.com/main/administering-tabnine/managing-your-team/reporting [vendor; fields unverified].
- [27] Windsurf analytics — https://docs.windsurf.com/windsurf/accounts/analytics [vendor; partly unverified] — PCW (% code written by AI).
- [28] LeadDev on acceptance-rate fall — https://leaddev.com [practitioner; article URL not captured — re-locate].
- [18] Delivery-analytics vendors admit no AI-vs-human line attribution — https://linearb.io/blog/is-github-copilot-worth-it · https://www.faros.ai/blog/key-takeaways-from-the-dora-report-2025 · https://www.swarmia.com/blog/space-framework/ [vendor].

Validity & ethics:
- [30] Goodhart's law — https://en.wikipedia.org/wiki/Goodhart%27s_law [independent/reference].
- [31] Campbell's law — https://en.wikipedia.org/wiki/Campbell%27s_law [independent/reference].
- [32] Construct validity (Cronbach & Meehl 1955 etc.) — https://en.wikipedia.org/wiki/Construct_validity [independent/reference].
- [33] Salaudeen et al., validity-centered AI evaluation — https://arxiv.org/html/2505.10573v3 [independent].
- [34] Article 29 WP Opinion 2/2017 (workplace monitoring) — https://collab.dpa.gr/wp-content/uploads/2023/07/WP29_Opinion-2-2017-on-data-processing-at-work.pdf [primary regulatory].
- [35] Observer/Hawthorne effect — general reference [independent; snippet-level].
- [36] Psychological safety + metrics — https://codeclimate.com/blog/using-data-psychological-safety [vendor/practitioner; snippet-level].
- [37] Individual-variance confounds — arXiv 2104.13713 [independent; snippet-level — re-verify, was never fetched from primary].

Metadata-signal derivation:
- [38] LinearB cycle-time segments — https://linearb.helpdocs.io/article/0vif1ihmgc · https://linearb.io/blog/pull-request-pickup-time [vendor].
- [39] Code Climate Velocity review metrics — docs.velocity.codeclimate.com [vendor].
- [40] Swarmia cycle time / work log — help.swarmia.com [vendor].
- [41] GitClear Diff Delta methodology — https://www.gitclear.com [vendor] — durability-weighted change measure (τ scalar rewards un-churned code) — direct prior art for a quality-adjusted output unit.
- [42] git-revert metadata detection — https://git-scm.com/docs/git-revert [primary].
- [43] CI metrics (Harness/Gitar/AWS Well-Architected) — https://docs.aws.amazon.com [vendor/primary; exact URLs to re-locate].
- [44] Flaky-build research — arXiv "Understanding and Detecting Flaky Builds in GitHub Actions" [independent preprint] — ~13% of failures flaky.
- [45] SonarSource quality gates — docs.sonarsource.com [vendor].
- [46] Jira/PM metrics — LinearB/OBSS/reworkcost.com [vendor/practitioner].
- [47] PagerDuty MTTR/incident linkage — https://www.pagerduty.com [vendor].
- [48] Worklytics composite productivity score — worklytics.co [vendor; ethically contested; example of composite construction only].
- [49] "Will It Survive?" AI-code survival — arXiv [independent preprint] — line-survival methodology.

## 6. What the old mission got wrong / left open for THIS mission (inference unless cited)

1. **No cost dimension.** "Leverage/Efficiency" pillar never touched dollars; per-dollar efficiency, spend denominators, and their right-skew/denominator-gaming pathologies are unexplored. Only DX [7] has a Cost axis — thin.
2. **Refused a score.** Old answer was "recommendations, never a number." New mission must ship a self-view score; old work supplies guardrails (counterbalance, subscores, no targets) but zero score-construction design (percentiles vs z-scores vs bands, skew handling, winsorizing).
3. **Cross-person comparison declared invalid, full stop [37]** — but the new brief requires cohort-relative context. Cohort *design* (function×level, minimum-N, shrinkage) is untouched ground.
4. **Agent-platform/non-PR engineers never considered.** Old data model assumed IDE-assistant + GitHub activity; "spend with no visible output" is a wholly new fairness problem.
5. **CI-clean-rate**: old catalog rated first-pass CI Medium-confidence with saturation-by-iteration gaming already noted — it predicted, but did not solve, the new problem; churn/survival was its proposed better quality signal and needs git-history ingestion here.
6. **Real-implementation repo reading was shallow** (vendor docs only, no cloned OSS metric libraries) — new mission's task #2 is genuinely new work.
7. **Unvalidated**: no proxy in the old framework was ever empirically validated; carrying tiers over inherits hypotheses, not evidence.
8. **Old HAVE ≠ new HAVE**: old mission assumed review outcomes, commit/PR timestamps, and closed-PR data the new brief doesn't list; every §2b metric silently depends on that — must be bought, not assumed.
9. Several sources were snippet-level/unverified ([26][27][28][35][36][37][43], METR 2026 update) — re-verify before reuse; McKinsey [14] and LeadDev [28] URLs were never captured.
