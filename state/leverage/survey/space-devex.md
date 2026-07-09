# SPACE, DevEx, and DX Core 4 — how they operationalize measurement, and what that means for an individual "AI leverage" score

Sub-investigator findings, 2026-07-08. All ACM Queue full texts were read directly (Cloudflare bypassed via real browser); Table/Figure images pulled from the publisher CDN and read verbatim. Quotes are exact unless marked inference/unverified.

---

## 1. What it measures & theory

### 1.1 SPACE (Forsgren, Storey, Maddila, Zimmermann, Houck, Butler — ACM Queue, Mar 2021)

Theory: developer productivity is multi-dimensional and context-dependent; any single metric mismeasures it. The paper is organized as five myths → five dimensions:

- **S**atisfaction & well-being — "how fulfilled developers feel with their work, team, tools, or culture"; example metrics: employee satisfaction, developer efficacy ("whether developers have the tools and resources they need"), burnout. "These qualities are often best captured with surveys."
- **P**erformance — "the outcome of a system or process… often best evaluated as outcomes instead of output." Simplest view: "Did the code written by the developer reliably do what it was supposed to do?" Metrics: quality (reliability, absence of bugs, service health), impact (customer satisfaction/adoption, feature usage, cost reduction).
- **A**ctivity — "a count of actions or outputs." Metrics: PRs, commits, code reviews, builds/deploys, incident counts. Hard limit stated: activity metrics "should never be used in isolation either to reward or to penalize developers"; "it is almost impossible to comprehensively measure and quantify all the facets of developer activity."
- **C**ommunication & collaboration — discoverability of documentation, speed of work integration, review quality, network metrics, onboarding time.
- **E**fficiency & flow — "ability to complete work or make progress on it with minimal interruptions or delays"; handoffs, perceived ability to stay in flow, interruptions, total/value-added/wait time.

Figure 1 ("Example Metrics", read from publisher image) gives an individual/team/system grid, e.g. individual-level: developer satisfaction, code-review velocity, # commits, lines of code†, PR merge times, productivity perception. Footnote: "† Use these metrics with (even more) caution — they can proxy more things" (applies to retention, LOC, story points, quality of meetings).

**Operationalization rules (the load-bearing part):**
1. "capture several metrics across multiple dimensions of the framework — at least three are recommended." Adding another activity metric to an activity metric does NOT count.
2. "at least one of the metrics include perceptual measures such as survey data… perceptual data may provide more accurate and complete information than what can be observed from instrumenting system behavior alone."
3. Metrics deliberately **in tension**: "Including metrics from multiple dimensions and types of measurements often creates metrics in tension; this is by design, because a balanced view provides a truer picture."
4. Metrics signal values and shape behavior: "Metrics shape behavior, so by adding and valuing just two metrics, you've helped shape a change in your team and organization."
5. Not too many: "a good measure of productivity consists of a handful of metrics across at least three dimensions."
6. Privacy: "report only anonymized, aggregate results at the team or group level. (In some countries, reporting on individual productivity isn't legal.)" But individual-level analysis is endorsed **opt-in, for the developer's own use**: "Developers can opt in to these types of analyses, gaining valuable insights to optimize their days."
7. Bias/norm checks: gendered review-feedback effects; time normalization "would bias against those taking parental leave"; "Some cultures naturally report higher, while some report lower… measures from these different cultures… shouldn't be compared with each other."

### 1.2 DevEx (Noda, Storey, Forsgren, Greiler — ACM Queue, May 2023)

Theory: measure the *lived experience* (friction) of developers rather than output; 25 sociotechnical factors distilled into three dimensions: **feedback loops** (speed/quality of responses to actions), **cognitive load** (mental processing required), **flow state**. "Like developer productivity, developer experience cannot be boiled down to a single metric or dimension."

**Operationalization = a 3×2 grid + KPIs** (Table 1, read verbatim from publisher image):

| | Feedback loops | Cognitive load | Flow state |
|---|---|---|---|
| **Perceptions** (human attitudes & opinions) | satisfaction with automated test speed and output; satisfaction with time to validate a local change; satisfaction with time to deploy to production | perceived complexity of codebase; ease of debugging production systems; ease of understanding documentation | perceived ability to focus and avoid interruptions; satisfaction with clarity of task/project goals; perceived disruptiveness of being on-call |
| **Workflows** (system & process behaviors) | time to generate CI results; code review turnaround time; deployment lead time | time to get answers to technical questions; manual steps to deploy a change; frequency of documentation improvements | number of blocks of time without meetings/interruptions; frequency of unplanned tasks/requests; frequency of incidents requiring team attention |
| **KPIs** (North Star) | colspan: overall perceived ease of delivering software; employee engagement or satisfaction; perceived productivity | | |

Why both halves are mandatory: "A comparative analysis of perceptual measures and workflows is necessary because neither alone can tell the full picture. For example, seemingly fast code review turnaround times may still feel disruptive to developers if code reviews regularly interrupt their work progress." And the political function of surveys: "Measuring developers' perceptions can help leaders counterbalance metrics that are designed around a top-down view of developers' work."

Survey craft guidance: base items on "well-defined constructs… rigorously tested in interviews"; **break down results by team and persona** ("role, tenure, seniority… Focusing only on aggregate results can lead to overlooking problems that affect small but important populations"); benchmark against peers; mix in transactional surveys; quarterly-to-semi-annual cadence to avoid fatigue ("A lack of follow-up action commonly causes developers to feel that repeatedly responding to surveys is not a worthwhile exercise").

### 1.3 DevEx in Action (Forsgren, Kalliamvakou, Noda, Greiler, Houck, Storey — ACM Queue, Jan 2024)

The empirical validation attempt. Grounded in Work Design Theory; hypothesizes the 3 DevEx dimensions drive developer/team/org outcomes. Survey items: flow = deep-work satisfaction, interruption frequency, engaging tasks; feedback loops = time to get code approved, question-answer speed; cognitive load = ease of deploying, ease of understanding code, intuitiveness of processes/tools. Results (PLS, n=219): **H1 (flow) and H3 (cognitive load) fully supported; H2 (feedback loops) only partially — team outcomes only, not developer or organization outcomes.** Likelihood analysis (all self-reported "feel" outcomes): deep-work time → "feel 50 percent more productive"; engaging work → 30%; high code understanding → 42%; intuitive tools → "50 percent more innovative"; fast review turnaround → 20% more innovative; fast question answers → "50 percent less technical debt."

### 1.4 DX Core 4 (DX / Abi Noda et al., launched Dec 2024–Jan 2025) — what it ACTUALLY ships

Vendor unification of DORA+SPACE+DevEx. Canonical table (read verbatim from DX's published Table 1 image):

| | Speed | Effectiveness | Quality | Impact |
|---|---|---|---|---|
| **Key metric** | **Diffs per engineer\* (PRs or MRs)** — "\*Not at individual level" | **Developer Experience Index (DXI)** — "measure of key engineering performance drivers, developed by DX" | **Change failure rate** | **Percentage of time spent on new capabilities** |
| **Secondary** | Lead time; Deployment frequency; Perceived rate of delivery | Time to 10th PR; Ease of delivery; Regrettable attrition\* (\*org level only) | Failed deployment recovery time; Perceived software quality; Operational health & security metrics | Initiative progress and ROI; Revenue per engineer\*; R&D as % of revenue\* (\*org level only) |

Shipping mechanics (from DX product docs): PR throughput = "Average PRs merged/engineer/week" over a **12-week lookback**, **bot PRs omitted**; DXI = survey-only ("DX Snapshots", quarterly/semi-annual), an aggregate of **14 standardized Likert-scale items** ("actionable… such as deep work, local iteration speed, release process, and confidence in making changes" — full 14-item list not publicly enumerated in fetched sources; unverified beyond examples); Quality also ships a "defect ratio: % of PRs for bug fixes (**AI-classified**)"; Impact ships "% of PRs allocated to new capabilities (AI-classified)"; roadmap: "TrueThroughput® (complexity-adjusted PR count)". Measurement-methods Table 2: system metrics ("objective… real-time" / "cross-system visibility and data normalization"), self-reported ("rapid… experience metrics" / "question design, participation rates"), experience sampling ("targeted in-the-moment insights" / "complexity of setup").

Anti-gaming design (their words): "Speed and throughput metrics, when used in isolation, often incite fear and counterproductive behaviors from developers. The DX Core 4 equally weights speed and output metrics with the [DXI]…" Preconditions for diffs-per-engineer: (1) "counterbalancing with oppositional metrics like the Developer Experience Index," (2) "not setting targets or rewards tied to them," (3) communication/rollout that "does not result in abuse." Repeated: "It is critical that this metric is never used at the individual level or tied to performance evaluations."

DXI→dollars claim [vendor]: "Each one-point gain in DXI score translates to saving 13 minutes per week per developer, equivalent to 10 hours annually"; top-quartile DXI teams "4 to 5 times higher" speed/quality; validation "data from over 40,000 developers across 800 organizations."

AI-era update (Brian Houck, getdx newsletter, 2026-06-24, "Revisiting the DX Core 4 in the Age of AI"): "AI doesn't change what engineering organizations are trying to accomplish. What it does is make the signals noisier." AI adoption ≈ "7.8% increase in PR throughput" [vendor-reported]. On merge rate: "does a 95% merge rate mean the AI is flawless? Or does it mean your human developers are rubber-stamping machine-generated code?" On time-to-10th-PR: may now "just track how quickly someone learned to use AI tools." "Using PR throughput to assess individuals is the wrong application of the metric."

Related recent operationalization: Microsoft's **EngThrive** (Houck, Bozarth, Liu, Carignan; ACM Queue/arXiv 2026): speed/ease/quality dimensions + "thriving" well-being guardrail; "pairs outcome-oriented North Star metrics with diagnostic submetrics, combining system telemetry with developer surveys" — same survey+telemetry dual-track pattern, now positioned as "a general-purpose evaluation language applicable to tools, AI, and organizational policies."

---

## 2. Evidence quality

- **SPACE**: conceptual/expert framework, essay in ACM Queue — no empirical validation of the framework itself in the paper; authority derives from authors' prior research (DORA, Microsoft studies) and citations. Treat as design doctrine, not measured fact. [primary]
- **DevEx (2023)**: framework paper; cites Gartner ("78 percent of surveyed organizations have a formal DevEx initiative either established or planned") and a 2020 McKinsey developer-velocity study (4–5× revenue growth) — both secondhand, not independently verified here. [primary, framework]
- **DevEx in Action (2024)**: the only quantitative test in this family. Real but weak-to-moderate evidence: n=219 completions from 2,213 invited (**9.9% response rate**), all from DX-customer companies (selection bias the authors acknowledge: "could bias the results"), cross-sectional (no causality), and **all outcomes are self-reported feelings** ("feel 50 percent more productive"), not measured output. Psychometrics reported (CR, AVE, HTMT) are sound for what it is. Notable honest negative result: feedback loops did NOT significantly affect developer or org outcomes. [primary]
- **DX Core 4**: vendor framework co-authored with the SPACE/DORA/DevEx researchers. Validation claims — "implemented at over 300 tech, finance, retail, and pharmaceutical companies," "3%–12% overall increase in engineering efficiency," "14% increase in R&D time spent on feature development," 15% engagement gain, Booking.com "16% productivity lift" across 3,500 engineers — are all **[vendor], no independent replication found**. DXI's "13 minutes per DXI point" is a regression on self-reported time loss [vendor, method not public]. Will Larson [practitioner, disclosed DX angel investor]: metric choice "very reasonable," but "no one understands composite metrics" and "skeptical people just don't trust composite metrics particularly well."
- **Counter-evidence on perception measures in AI context**: METR RCT (Jul 2025, 16 experienced OSS devs, 246 tasks): devs forecast AI would make them 24% faster, *afterwards* believed 20% faster — actually **19% slower**. Directly undermines self-reported AI time savings (which is exactly what DX's AI Measurement Framework ships as its headline impact metric). [independent]
- Beller et al., "Mind the Gap" (arXiv 2012.07428, 81 Microsoft developers): self-reported and automatically measured productivity are studied as historically disjoint lines; paper bridges them — exact correlation coefficients not extracted (PDF unreadable to fetch tool); direction of finding (imperfect alignment) verified from abstract only. [independent, partially verified]

---

## 3. Known failure modes

1. **Single-metric collapse** — the exact failure SPACE names: "one metric that matters… isn't true." The client's current design (two highly correlated per-dollar activity ratios averaged into one 0–100 number) is a one-dimensional Activity measure in SPACE terms, violating "at least three dimensions" and "at least one perceptual." Its two components are not "metrics in tension" — both rise with more merged volume per dollar. (inference, grounded in SPACE's stated rules)
2. **Activity misuse** — SPACE: activity metrics "should never be used in isolation either to reward or to penalize developers"; invisible work (mentoring, unblocking, agent/infra work) is uncounted. This is the precise mechanism behind the brief's "agent-platform engineers score near-zero."
3. **Objective-only blindness** — DevEx: "neither alone can tell the full picture"; fast metrics can hide bad experience and vice versa. A leverage score computed purely from PR/CI/spend telemetry inherits this.
4. **Survey-only blindness** — METR: perception of AI speedup can be directionally wrong. DXI-style self-report as the *sole* AI-leverage signal would reward confident feelings, not leverage.
5. **Composite opacity** — Larson: composite indices (DXI, and by analogy any 0–100 blended leverage score) are distrusted precisely because members can't trace movement; violates the brief's interpretability requirement when weights are hidden.
6. **Survey fatigue & non-response** — DevEx paper itself: participation collapses without visible follow-up action; DX's own docs list "question design, participation rates" as the method's core challenge. DevEx in Action's 9.9% research-survey response rate is a live demonstration.
7. **Benchmark/persona mismatch** — DevEx: experience "can differ radically across teams or roles"; comparing raw scores across functions is invalid. Mirrors the brief's known problem #6 (job-level-only cohorts across all functions).
8. **AI-era signal noise** — Houck 2026: identical PR-throughput numbers can now come from very different human-AI mixes; merge/CI-pass rates inflate as agents iterate to green ("rubber-stamping machine-generated code") — independently corroborates the brief's observation that CI-clean-rate saturates at ~1.0 and stops discriminating.

---

## 4. Gameability & who it penalizes (per metric)

| Metric | (a) How an engineer games it | (b) Who it unfairly penalizes |
|---|---|---|
| Diffs/PRs per engineer (Core 4 Speed key) | Split work into confetti PRs; have AI mass-generate trivial diffs (LinearB: "extremely easy to game by generating purposeless PRs"); config/docs churn | Platform/infra/agent engineers whose output isn't PRs; reviewers/mentors; those on gnarly long-lived changes; heavy-monorepo teams with batched merges |
| Lead time / deployment frequency | Pre-bake branches, deploy no-op changes, reclassify work | Teams with compliance gates, mobile release trains, heavy-CI repos |
| CI-clean / change failure rate | Let AI iterate until green before opening PR (rate→100%, zero discrimination); shrink deploy units so failures dilute; reclassify incidents | Engineers in repos with 50+ flaky CI jobs; SREs whose work surfaces failures; anyone doing risky-but-valuable changes |
| % time on new capabilities (Impact) | Self-classify maintenance as "new capabilities"; AI-classified PR labels are promptable/gameable by title wording | Maintenance, security, refactoring, reliability specialists — their work is definitionally "not innovation" |
| DXI / survey items (Effectiveness) | If stakes attach: answer high to look good, or **sandbag low to lobby for tooling budget/headcount**; managers can campaign for scores before survey windows (inference; mechanism follows Goodhart). With no stakes (self-view only), gaming incentive largely disappears — the residual failure is honest miscalibration (METR), not manipulation | Non-respondents (coverage bias); cultural response-style differences (SPACE: "some cultures naturally report higher… shouldn't be compared"); low-psych-safety teams; senior engineers with higher standards report lower satisfaction for identical tooling (inference) |
| Self-reported AI time savings (DX AI framework Impact) | Overstate savings to justify tool spend; anchor on vendor marketing numbers | Honest reporters; skeptics; anyone whose AI value is quality/learning rather than speed — and METR shows even honest answers overstate |
| Perceived productivity / perceived rate of delivery | Same as DXI; also recency-biased (LinearB: "influenced by recent frustrating experiences") | Same as DXI; people mid-crunch or post-incident |
| Time to 10th PR (onboarding) | Assign trivial starter tasks; AI-generate first PRs (Houck: may just measure "how quickly someone learned to use AI tools") | New hires on complex/regulated codebases; career-switchers into new domains |
| Revenue per engineer / R&D % | Org-level only; individual can't game, but leadership games by headcount cuts (LinearB: "incentivizes headcount reduction over value creation") | Everyone during downturns; cost-center teams |

**Cross-cutting finding on survey gameability**: surveys are the *hardest* signal for an individual to game **for personal score inflation when the score is self-view-only** (there is nobody to impress), but the *easiest* to game **collectively when aggregate results drive budgets or comparisons** (both directions: inflation to please, deflation to lobby). SPACE/DevEx implicitly rely on the no-stakes condition; DX enforces it as precondition ("not setting targets or rewards"). The residual, non-gaming failure of surveys is honest miscalibration, which METR shows is severe specifically for AI-assisted work — so surveys should ask about *friction, satisfaction, and where AI is used* (experience facts), not *estimated time saved* (counterfactual guesses). (inference from cited sources)

---

## 5. Applicability to our constraints (HAVE / CANNOT mapping)

**What transfers directly onto data we HAVE:**
- Core 4 Speed key metric ≈ our merged-PR count (12-week lookback, bot-PR exclusion, per-week averaging are directly liftable mechanics). But DX's own table stamps it "**Not at individual level**" — the current leverage score's per-engineer PR-per-dollar core is, by the framework's own label, a misapplication. The mitigations DX names (no targets/rewards; counterbalance; careful comms) are partially satisfied by the brief's self-view-only rule — that is the strongest argument that a per-engineer activity signal is *survivable here at all*. (inference)
- DevEx "workflows" column: CI result time, deployment lead time, review turnaround are computable from PR/CI metadata we have (turnaround needs PR event timestamps — likely in warehouse).
- SPACE's ≥3-dimensions rule maps to a redesign as **subscores, not one number**: e.g., Adoption/Activity (usage telemetry — have), Output flow (PR metadata — have), Experience/perception (survey — NEW). SPACE explicitly blesses opt-in individual-level self-analytics — exactly the product's privacy posture.
- DevEx persona guidance ("break down results by team and persona — role, tenure, seniority") is the direct fix for the brief's known problem #6: cohorts must be function×level, not level-only.
- SPACE time-normalization warning (parental leave) → fix for cold-start brittleness: normalize by active days, use rolling 12-week windows (Core 4 uses 12 weeks), degrade gracefully instead of on/off floors. (inference)

**What hits our CANNOT list:**
- Core 4 Quality (change failure rate, failed-deployment recovery) needs per-author production-failure linkage — brief says we don't have it. Nearest available: CI-clean signal (saturating, weak) or NEW survey item "perceived software quality" (Core 4 secondary, survey-sourced — cheap to add).
- Core 4 Impact (% time on new capabilities, initiative ROI) needs work classification — we have "no task difficulty/complexity/business value." DX ships this via AI-classification of PR titles/Jira; that is acquirable but gameable via wording.
- DevEx cognitive-load workflow metrics (time to get answers, manual deploy steps) — not in warehouse; survey substitutes exist.
- DXI itself is proprietary; but its *pattern* — a small standardized Likert battery, scored per-driver and aggregated, quarterly — is replicable in-house. For agent-platform engineers (spend-but-no-PR population), a 3–5 item self-report of non-PR output categories + AI friction is the ONLY signal family in these frameworks that makes their leverage visible at all, since all system-side output metrics in SPACE/DevEx/Core 4 are PR/deploy-shaped. This is the frameworks' single most important contribution to the brief's hardest constraint. (inference)
- Data lag (~2 weeks) is compatible with quarterly survey cadence and 12-week windows.

**Verdict for metric design:** These frameworks converge on four commitments any credible leverage score must honor: (1) ≥3 dimensions with ≥1 perceptual; (2) metrics in deliberate tension rather than averaged look-alikes; (3) activity/throughput never standing alone at individual level; (4) persona-segmented baselines. The current metric violates all four. A survey is the highest-lift NEW signal (fills quality-perception, agent-work visibility, and experience dimensions at once) — but must ask experience facts, never self-estimated AI time savings (METR). (inference, grounded above)

---

## 6. Sources (annotated)

1. **[primary]** Forsgren et al., "The SPACE of Developer Productivity," ACM Queue 19(1), Mar 2021 — https://queue.acm.org/detail.cfm?id=3454124 — full text + Figure 1 read directly. Five dimensions; ≥3-dimensions + perceptual-measure rules; anti-gaming and bias guidance; opt-in individual analytics.
2. **[primary]** Noda, Storey, Forsgren, Greiler, "DevEx: What Actually Drives Productivity," ACM Queue 21(2), May 2023 — https://queue.acm.org/detail.cfm?id=3595878 — full text + Table 1 image read directly. 3 dimensions × (perceptions|workflows) + KPIs; survey craft; persona breakdown.
3. **[primary]** Forsgren, Kalliamvakou, Noda, Greiler, Houck, Storey, "DevEx in Action," ACM Queue 21(6), Jan 2024 — https://queue.acm.org/detail.cfm?id=3639443 — full text read directly. n=219, 9.9% RR, PLS; H2 only partially supported; 50/30/42/50/20/50% self-reported effects; limitations acknowledged in-paper.
4. **[vendor]** DX, "Measuring developer productivity with the DX Core 4" — https://getdx.com/research/measuring-developer-productivity-with-the-dx-core-4/ — canonical Table 1 (key/secondary metrics, "not at individual level" flags) and Table 2 (measurement methods) read from published images; 300+ companies, 3–12%/14%/15% claims [vendor-reported].
5. **[vendor]** DX docs, "Guide to the DX Core 4" — https://docs.getdx.com/dx-core-4/ — shipping mechanics: 12-week lookback, bot-PR exclusion, AI-classified defect/innovation ratios, TrueThroughput roadmap, DXI via survey snapshots.
6. **[vendor]** DX, "The one number you need to increase ROI per engineer" (DXI) — https://getdx.com/research/the-one-number-you-need-to-increase-roi-per-engineer/ — DXI = 14 Likert drivers; 13 min/week per point; 40k devs / 800 orgs validation claims.
7. **[vendor]** Houck, "Revisiting the DX Core 4 in the Age of AI," getdx newsletter, 2026-06-24 — https://newsletter.getdx.com/p/revisiting-the-dx-core-4 — "signals noisier"; +7.8% PR throughput; merge-rate rubber-stamping; time-to-10th-PR ambiguity; anti-individual-use warning.
8. **[practitioner]** Will Larson, "Measuring developer experience, benchmarks, and providing a theory of improvement" — https://lethain.com/measuring-developer-experience-benchmarks-theory-of-improvement/ — composite-metric distrust; benchmark verifiability; disclosed DX investor.
9. **[practitioner]** LinearB, "DX Core 4 deep dive" — https://linearb.io/blog/dx-core-4-deep-dive — competitor critique: PR-count gaming, DXI "proprietary black box," perception lag/recency, missing collaboration dimension. (Competitor bias noted.)
10. **[independent]** METR, "Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity," Jul 2025 — https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ · arXiv:2507.09089 — RCT: forecast −24%, post-hoc belief −20%, actual +19% time. Follow-up: https://metr.org/blog/2026-02-24-uplift-update/ (design revision, Feb 2026).
11. **[independent, partially verified]** Beller et al., "Mind the Gap: On the Relationship Between Automatically Measured and Self-Reported Productivity," IEEE Software / arXiv:2012.07428 — 81 Microsoft devs; self-report vs automated measures imperfectly aligned; exact coefficients unverified (PDF unparseable to fetch tool).
12. **[primary]** Houck, Bozarth, Liu, Carignan, "EngThrive: Make It Fast and Easy to Do Great Work," ACM Queue / arXiv:2605.04259 (2026) — Microsoft's deployed system: speed/ease/quality + thriving guardrail; North Star + diagnostic submetrics; telemetry + surveys. (Read via abstract/related-article blurb only.)
13. **[practitioner]** LeadDev, "How DX Core 4 aims to unify developer productivity frameworks" — https://leaddev.com/reporting/dx-core-4-aims-to-unify-developer-productivity-frameworks — independent reporting on launch and metric placement (corroboration).
