# CRITIQUE — Construct Validity: Does "Quality-Adjusted Coding Output per AI Dollar" Measure "AI-Assisted Engineering Leverage" At All?

Sub-investigator critique, 2026-07-08. Angle: construct validity (Cronbach & Meehl lineage — content, convergent, discriminant, criterion validity, nomological network; https://en.wikipedia.org/wiki/Construct_validity [reference]).
Grounding: Phase-A survey files in `state/leverage/survey/` and `state/leverage/repos/` (read directly); all external claims cited with tags; my own reasoning labeled **[inference]**.

**Verdict up front: No. The current metric is not a noisy measure of leverage — it is a valid-looking measure of a different construct ("metered-spend frugality × PR-shaped volume"), and it is anti-correlated with the most defensible definitions of leverage. It fails all four classical validity tests, and its optimum is reached by minimizing AI use. This is not fixable by re-weighting; the construct itself must be replaced.**

---

## 1. What should "AI-assisted engineering leverage" mean?

Before testing the metric, the target construct needs a definition. Three candidate definitions exist in the evidence base; the current metric matches none of them.

### 1.1 Leverage as causal amplification (the literal meaning)
Leverage is a ratio of realized outcome to counterfactual outcome: how much more (durable, valuable) engineering output this engineer achieves *with* AI than they would *without*, holding the person and task constant. **[inference from the plain semantics of "leverage"]**

- This is inherently a **causal, within-person, counterfactual quantity**. The brief's hard limit #1 (no AI-vs-human attribution) plus no task-difficulty signal means the counterfactual is unobservable per-person. The best measurement designs in the world are struggling with exactly this: METR's RCT found experienced devs 19% *slower* with AI while believing +20% (https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ [independent]), and its Feb-2026 update flips the point estimate but calls its own data "only very weak evidence" due to selection effects (https://metr.org/blog/2026-02-24-uplift-update/ [independent]).
- Consequence: **any cross-sectional per-person number that claims to BE leverage-as-amplification is overclaiming by construction.** The honest options are within-person contrasts (own AI-heavy vs AI-light periods) or renaming the construct. **[inference]**

### 1.2 Leverage as conditional amplification (DORA's amplifier thesis)
DORA 2025: "AI's primary role is as an amplifier, magnifying an organization's existing strengths and weaknesses" (https://dora.dev/dora-report-2025/ [primary]). Under this thesis leverage is a property of **person × environment**, not of the person alone: the same practices yield 30–40% gains in greenfield work and ~0–10% in brownfield (Denisov-Blanch, Stanford telemetry, magnitudes unverified — [independent, not peer-reviewed]), +27–39% for juniors vs +8–13% for seniors (Cui et al., Management Science 2026, n=4,867, https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 [independent-adjacent]).

- Consequence: a leverage construct must either **condition on environment** (function × repo-environment cohorts, expectations differing per cohort) or **score only what the individual controls** (practices/behaviors) and display outcomes as context. The current metric does neither — it percentile-ranks raw ratios across all functions at a job level.

### 1.3 Leverage as usage quality (the prior mission's 3-pillar construct)
The May-2026 sibling mission defined the latent construct as "durable, low-friction, well-matched leverage from AI" — Durability (does output survive?), Adoption Depth (right tool, deep use), Leverage/Efficiency (friction reduction) — with a hard "volume gate": volume signals (tokens, LOC, PR count) may never alone produce a positive finding (prior-art.md §1 [internal prior art]).

- Consequence: the current metric — which is ~100% volume-per-dollar — violates the construct's own gate. Its "quality adjustment" (CI-clean-rate) was already rated the *weakest* member of the weakest-replaceable pillar, with saturation-by-iteration predicted before this mission confirmed it (brief problems #1/#7).

### 1.4 What the current metric's construct actually is
"Output per AI dollar" is the **marginal unit economics of a tool**, not the leverage of a person. FinOps — the discipline that owns cost-per-output metrics — recommends them at workload/team level with showback and explicitly ships **no per-person efficiency metric** (https://www.finops.org/wg/finops-for-ai-overview/ [primary]). DORA's own 2026 ROI report frames AI ROI at org level: "We don't measure AI by the code it writes but by the bottlenecks it clears" (https://dora.dev/ai/roi/report/ [primary]; https://www.infoq.com/news/2026/05/dora-roi-ai-assisted-dev-report/ [independent]). **Calling a tool's unit-cost ratio a person's "leverage" is a category error: engineering output is produced by engineer-time + AI assistance jointly; dividing output by the AI complement alone attributes the whole product to the divisor.** **[inference]**

---

## 2. What the current metric ACTUALLY measures (decomposition)

Formula (from the brief): average of two within-level percentile ranks of
(1) merged PRs × CI-clean-rate / coding-CLI spend, and (2) winsorized lines × CI-clean-rate / coding-CLI spend.

### 2.1 The quality term is a constant → "quality-adjusted" is nominal
CI-clean-rate saturates: AI agents iterate to green, sub-3-PR engineers default to a fake 1.0, so "most cluster at ~1.0" (brief problem #7; corroborated externally — Houck/DX 2026: "does a 95% merge rate mean the AI is flawless? Or does it mean your human developers are rubber-stamping machine-generated code?", https://newsletter.getdx.com/p/revisiting-the-dx-core-4 [vendor]; Accenture RCT: build *volume* +38% while success *rate* was flat-to-negative — copilot-studies.md finding #4). Multiplying by a signal that is ≈1 for nearly everyone changes nothing. **Content-validity failure: the construct's "quality" facet is present in the label and absent from the measurement.** The 25–45% security-flaw rates in CI-passing AI code (Pearce ~40%, https://arxiv.org/abs/2108.09293 [independent]; Fu 29.5%/24.2%, https://arxiv.org/abs/2310.02059 [independent]; Veracode 45%, https://www.veracode.com/resources/analyst-reports/2025-genai-code-security-report/ [vendor]) mean the omitted quality variance is large and AI-correlated.

### 2.2 The two "signals" are one signal measured twice
PR-count/$ and lines/$ share the same denominator and correlated numerators (both merged-PR volume). SPACE requires ≥3 dimensions, ≥1 perceptual, metrics deliberately **in tension** "by design" (https://queue.acm.org/detail.cfm?id=3454124 [primary]). Averaging two correlated activity-per-dollar ratios is one dimension wearing two hats — and averaging percentile ranks (ordinal, unequal-interval) then arithmetic-averaging normalized ratios commits two documented statistical anti-patterns (Fleming & Wallace, CACM 1986: arithmetic mean of normalized ratios gives normalization-dependent rankings, https://dl.acm.org/doi/10.1145/5666.5673 [primary/academic]; goodhart-design.md §3.1–3.2).

### 2.3 Variance decomposition: whose variance is the score reporting? [inference, each component evidence-anchored]
The between-person variance in this ratio is plausibly dominated, in order, by:
1. **Function/repo environment** — PR/line norms and achievable gains differ structurally 3–5x (Cui; Stanford gradient; brief problem #6). The level-only cohort guarantees this variance reads as "personal leverage."
2. **Spend-metering coverage** — the denominator counts only an allowlist of CLIs; identical work via chat tools, personal accounts, or the agent platform moves the measured denominator without any change in behavior value. Also non-stationary: token prices fell ~90% since 2023, Copilot moved to credits mid-2026 (roi-cost.md §2.2 [independent/vendor]) — the dollar is not a stable unit.
3. **Task-run stochasticity** — identical tasks vary up to 30x in token cost, and cost only weakly tracks expert-rated difficulty (Bai/Mihalcea/Brynjolfsson et al., https://arxiv.org/abs/2604.22750 [independent]). A month of individual spend is substantially luck plus repo context size.
4. **CI-rate noise in the tail** — with the ratio distribution right-skewed and compressed (brief problem #3: "top scorers often just have marginally better CI rate"), top ranks select on noise — textbook *regressional Goodhart* (Manheim & Garrabrant, https://arxiv.org/abs/1803.04585 [primary/academic]).
5. **Actual AI-usage skill** — whatever residual is left.

A score whose variance is mostly environment + metering + luck, presented to the engineer as a personal attribute ("your leverage"), is not merely imprecise — **it systematically misattributes the environment to the person**, the exact "disparate comparisons" failure DORA forbids and the confound the prior mission ruled invalidating for cross-person comparison (prior-art.md §4.3).

### 2.4 The nomological kill-shot: the score is maximized by NOT using AI
Within the scored population (≥$5 spend), the ratio is strictly improved by pushing the denominator toward the floor: hand-write the code, use unmetered chat tools, avoid the agent platform. An engineer who abandons AI entirely except $5.01 of token spend while shipping the same PRs is the theoretical optimum of a metric named **"AI leverage."** A measure whose maximum sits at near-zero levels of the phenomenon it is named for has no construct validity in the ordinary sense — it is a *frugality* score. **[inference from the formula; extremal-Goodhart mode per Manheim & Garrabrant]** External corroboration that the relationships run the wrong way:
- Jellyfish (12k devs, 200 cos, Q1'26): heavy AI users merge ~2x the PRs on ~10x the tokens; cost-per-PR spans $0.28→$89.32 across quintiles — **the per-dollar ratio mechanically ranks the org's demonstrably most productive adopters last** (https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/ [vendor]).
- The only deployed individual spend-linked rankings (Meta "Claudeonomics," Amazon "KiroRank") ran the mirror direction (spend-maximizing) and were gamed by fabricated/idle agent activity and scrapped within months (https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/ [independent]; roi-cost.md §1.2). Goodhart pressure is symmetric; the client's design is the untested mirror of a publicly failed one. **[inference, evidence-backed]**
- No documented org percentile-ranks individuals on output-per-AI-dollar; every credible cost framework stops above the individual (roi-cost.md finding #1/#4 — searched extensively; absence stated plainly, unverified that none exists privately).

### 2.5 Convergent/discriminant/criterion validity scorecard

| Validity test | What it requires | Result for the current metric |
|---|---|---|
| **Content** | Cover the construct's facets (amplification, quality/durability, non-PR output, practices) | FAIL — covers one facet (PR-shaped volume); quality term saturated ≈1; agent-platform output structurally zero (brief hard limit #4) |
| **Convergent** | Correlate with independent indicators of good AI use (deep effective adoption, durable output) | FAIL — anti-correlates with adoption depth (Jellyfish 2x-output users rank last); no durability signal exists in HAVE data |
| **Discriminant** | NOT be explained by confounds (function, seniority, repo, metering) | FAIL — §2.3: environment and metering plausibly dominate variance; level-only cross-function cohorts import all of it |
| **Criterion** | Correlate with an external criterion of leverage | UNTESTABLE/FAIL — no org has validated any per-dollar individual score against anything (roi-cost.md); the causal literature's sign is unstable year-to-year (METR 2025 vs 2026; DORA 2024 vs 2025 throughput flip) |
| **Nomological network** | Move in theoretically expected directions | FAIL — improves when AI use decreases; rises with PR-splitting; falls when an engineer moves to harder/browner work or the agent platform |

### 2.6 The divergence problem: the numerator can rise BECAUSE value is falling
DORA 2024: a 25% increase in AI adoption associated with +3.4% code quality/+3.1% review speed but **−1.5% delivery throughput and −7.2% delivery stability** (https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report [primary]). Faros telemetry (10k+ devs): AI users merge **+98% more PRs** yet PR size +154%, review time +91%, bugs/dev +9%, and **no company-level throughput/quality gain** — "downstream bottlenecks are absorbing the value" (https://www.faros.ai/blog/ai-software-engineering [vendor]). An individual-volume numerator credits exactly the observed failure mode: exporting review burden and instability to teammates. A construct called "leverage" that pays out for cost-shifting is measuring **local activity**, not engineering value — the third arrow of the assumption chain (output → value) demonstrably breaks at org level (independent-evidence.md §1). Dan North's "worst programmer" (zero individual output, made the whole team faster — https://dannorth.net/the-worst-programmer/ [practitioner]) marks the other tail: the highest-leverage people score zero.

---

## 3. Can ANY denominator-based score be motivating AND honest for a self-reflection tool?

A self-reflection score must satisfy three properties **[inference — design criteria distilled from Austin's informational-measurement mode and SPACE's opt-in self-analytics blessing]**:
(P1) **Controllability** — movements trace to the engineer's own choices; (P2) **Aligned improvability** — the action that raises the score is genuinely better practice; (P3) **Stability** — no movement without behavior change.

An AI-spend denominator violates all three:
- **P1**: denominator moves with task assignment, repo context size (input tokens dominate agent cost — arXiv 2604.22750 [independent]), model price changes, vendor credit promotions, and metering coverage — none engineer-controlled.
- **P2**: the score-raising actions are use-less-AI, use-cheaper-models-regardless-of-fit, avoid-the-agent-platform, split PRs. Every one is behavior the org does not want; several fight explicit adoption mandates (Shopify memo pole — roi-cost.md §3.2 [independent coverage]).
- **P3**: 30x same-task token variance plus a 4-week window means scores jump with no behavior change; percentile framing adds cohort non-stationarity during an adoption wave (your score falls when colleagues adopt faster — goodhart-design.md §3.1). A self-reflection number that moves for reasons the engineer cannot reconstruct is uninterpretable, hence demotivating and dishonest — it violates the brief's own "interpretable" and "motivating and honest" requirements simultaneously. **[inference]**

Two denominator families must be distinguished **[inference]**:
1. **Exposure normalizers** (per active day, per week, per engineer in roster) — legitimate; they correct for opportunity, do not price the input, and do not create an incentive to suppress the divisor (you can't easily fake fewer active days without hiding work).
2. **Investment denominators** (dollars, tokens) — structurally punish investment; the ratio's optimum is under-investment (extremal Goodhart); evidence says returns to spend are real but diminishing (Jellyfish), so the honest summary of spend-vs-output is a **curve position**, not a scalar ratio.

Surviving uses of spend (all consistent with roi-cost.md's bottom line): (a) **adoption/engagement context** (log-scaled, capped, numerator-side); (b) **private self-vs-self cost showback** (FinOps pattern; own trailing baseline; elasticity form `log(output+1) − log(spend+s0)` if a trend must be summarized) — labeled "cost awareness," never "leverage"; (c) team/function-level unit costs for org stewardship. **A per-dollar scalar as the headline personal score is unsalvageable.**

---

## 4. What construct SHOULD the replacement target?

Given the counterfactual is unmeasurable per-person (hard limits #1–#4), the honest, measurable construct is **[inference, synthesizing DORA's amplifier thesis + prior mission's 3 pillars + SPACE's dimensionality rules]**:

> **"AI-leverage practice and trajectory": the degree to which an engineer's AI-usage pattern matches practices evidence associates with durable, system-friendly output — and the direction of their own output/quality/experience trend as their usage deepens, read against function×level cohort context.**

Operational skeleton (subscores in deliberate tension, never one composite; each with explicit uncertainty and "insufficient data" states):
1. **Practice/adoption depth** (HAVE data): tool-category mix incl. agent platform, breadth, consistency, agentic-vs-chat depth — individual-controllable, rescues agent-platform engineers, aligned with DORA's capabilities framing (score behaviors, show outcomes).
2. **Batch-size discipline** (HAVE data): PR size distribution vs function norms — DORA's own AI-instability mechanism, and it *inverts* the current lines numerator (dora.md finding #4).
3. **Within-person outcome trend** (HAVE data): own merged-output and (until replaced) CI-anomaly floor vs own rolling 8–12-week baseline, AI-heavy vs AI-light weeks — the only causal-ish design available without an RCT (independent-evidence.md §5); Faros-CE precedent for period contrasts (repos/faros.md).
4. **Durability counterweight** (NEW signals, top acquisitions): post-merge churn/rework ≤2–4wk, reverts, review-burden — the quality facet CI cannot supply (gitclear.md, vendor-frameworks.md).
5. **Experience/calibration layer** (NEW, survey): friction/experience *facts* (never self-estimated time savings — METR's 40pp miscalibration), per DevEx persona segmentation; the only framework mechanism making non-PR leverage visible (space-devex.md §5).

Naming matters: if the dashboard keeps the word "leverage" it must display it as **practice + trend**, not as a measured amplification factor — the field cannot currently measure the latter for individuals, and saying so plainly is a brief requirement (task #7).

---

## 5. Gaming & unfair-penalty analysis (every metric discussed here)

| Metric | (a) How an engineer games it | (b) Who it unfairly penalizes |
|---|---|---|
| Current composite (avg. percentile of PRs·CI/$ and lines·CI/$) | Suppress denominator (personal/free accounts, unmetered chat, avoid agent platform, cheapest models); split PRs under the 2000-line cap (raises PR count too); time spend/PRs across window boundaries; do only easy work | Heavy legitimate spenders on hard problems; agent-platform builders (spend, no PRs); seniors in brownfield repos; ML/data engineers with expensive loops; minority functions inside level-only cohorts |
| CI-clean-rate (quality term) | Iterate to green locally/agentically before push (default AI behavior — saturates); avoid strict/flaky-CI repos; submit only trivial PRs | Heavy-CI (50+ jobs) and flaky-test repos; risk-takers; <3-PR people frozen at fake 1.0 |
| Merged-PR count (numerator) | PR confetti; AI mass-generated trivial diffs; dep/docs churn (commodity tooling exists — fake-git-history et al.) | Big-change engineers, mature-codebase owners, reviewers/mentors, non-PR roles |
| Winsorized lines (numerator) | Verbose generation, vendored/lockfile churn; split one 6k-line change into 3×2k (beats cap AND triples PR count) | Deleters/refactorers (Atkinson's −2000 lines), config/infra work, terse-language users |
| Spend/tokens as denominator | Denominator suppression (above); as *any* individual percentile: cheap-model theater with invisible quality cost | The org's most productive heavy adopters (Jellyfish); large-context monorepo engineers |
| Within-level cross-function percentile | Not directly gameable self-view, but rewards matching cohort norms over good work; moves without behavior change during adoption waves | Minority functions head-to-head vs different work shapes; everyone during cohort drift |
| Proposed: adoption-depth/practice subscore | Performative sessions on "deep" tools; trivial daily usage for consistency | Specialists whose one tool is right; restricted-allowlist teams; PTO/part-time (normalize by active days) |
| Proposed: batch-size discipline | Artificial PR-splitting (bound with floors/diminishing returns; stop rewarding size, don't punish it) | Legitimate large migrations/codegen; release-train workflows |
| Proposed: within-person trend | Sandbag early baseline to show improvement; batch work across period boundaries | New hires (no baseline — show "insufficient data," not zero); role-changers; task-mix shifts read as trend |
| Proposed: churn/revert durability counterweight | Delay fixes past the window; fix-in-new-files; avoid ambitious refactors (timidity) | Prototypers/iterators, hot-surface owners, trunk-based workflows — must be cohort/repo-normalized (Nagappan & Ball: relative not absolute churn, https://dl.acm.org/doi/10.1145/1062455.1062514 [independent]) |
| Proposed: survey/experience layer | Near-ungameable when self-view-only (nobody to impress); collectively gameable if aggregates drive budgets (both directions) | Non-respondents; cultural response-style differences; honest pessimists — ask experience facts, never estimated time saved |

Cross-cutting: self-view-only is the single strongest anti-gaming control (Austin's informational mode; Beck's sanctioned self-measurement — https://newsletter.kentbeck.com/p/measuring-developer-productivity-440 [practitioner]); gaming pressure scales with *perceived* stakes, so any leak to managers reactivates every mode above.

---

## 6. Summary of the construct-validity case

1. "Leverage" is a causal/counterfactual construct; the data cannot support measuring it cross-sectionally per person — the metric overclaims by construction.
2. The quality adjustment is nominal (CI ≈ 1 for nearly all), so the metric's real content is volume-per-metered-dollar: **PR volume × spend frugality**.
3. Its variance is dominated by environment, metering coverage, and stochastic token cost — it reports the situation and calls it the person.
4. Its optimum is near-zero AI use; it anti-ranks the org's demonstrably most productive adopters (Jellyfish) and zero-ranks its highest-leverage builders (agent platform; North's "worst programmer").
5. It credits the documented AI failure mode (individual PR inflation while org value stalls — Faros, DORA 2024).
6. No investment-denominator scalar can be simultaneously motivating, honest, and interpretable for self-reflection; spend survives only as context, self-showback, and team-level unit economics.
7. The replacement construct: **AI-leverage practice + within-person trajectory, with a durability counterweight and an experience layer** — subscores in tension, function×level context, uncertainty shown, "unmeasurable" stated where it is true.
