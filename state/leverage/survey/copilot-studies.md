# GitHub Copilot Evidence Base: RCTs, Telemetry Studies, Acceptance Rate, and the Metrics API (as of 2026-07)

Sub-investigator report for the AI-Leverage metric design mission. All primary sources were fetched and read in full text (Peng, Ziegler, Cui et al. read from extracted PDFs; API docs fetched live). Every external claim carries a tagged URL. My own reasoning is labeled **[inference]**.

---

## 1. What it measures & theory

The Copilot evidence base contains four distinct measurement theories, in rough historical order:

1. **Task-time RCTs** (Peng et al. 2023): productivity = time to complete a standardized task, with correctness gated by a visible test suite. Theory: hold the task constant, randomize tool access, measure speed.
2. **Telemetry ↔ perceived-productivity correlation** (Ziegler et al. 2022/CACM 2024): since real-world output can't be measured directly, use developers as "expert assessors of their own productivity" (SPACE-based survey) and find which telemetry signal best predicts their self-report. This is the paper that crowned **acceptance rate** (accepted/shown completions).
3. **In-situ workflow-output RCTs** (GitHub/Accenture 2024; Cui, Demirer, Jaffe, Musolff, Peng & Salz — Microsoft + Accenture + anonymous Fortune 100): productivity = counts of workflow artifacts (PRs "as a unit of work," commits, builds), quality = build success rate. Theory: PR scope is roughly stable *within* an org, so PR counts are usable as a task-completion proxy under randomization.
4. **Deployment/adoption telemetry** (ZoomInfo 2025; GitHub Copilot usage-metrics product 2024–2026): operational monitoring — acceptance rates by language, engaged users, feature adoption (chat/agent/CLI/code review), LoC suggested vs added, per-user credits consumed. Theory has shifted from "is the tool good?" (acceptance) to "who has adopted how deeply, and what flows downstream?" (feature-mix + PR outcomes).

Key definitional details verified from primary texts:
- **Acceptance rate** = `accepted_per_shown` (fraction of *shown* completions accepted into the file). Copilot ~26% mean / 24% median in Ziegler's sample (n=2,038 with data); IntelliCode Compose reported 10% "CTR" — cross-system comparison is meaningless because systems differ in what/when they choose to show [primary: Ziegler].
- **Persistence/retention** = accepted completion still (mostly) unchanged after 30/120/300/600s, "mostly" = Levenshtein distance < 33% [primary: Ziegler]. The famous "88% retention" in the Accenture blog is *character* retention of accepted code [vendor].
- **Cui et al. outcome hierarchy**: PRs = deliverable unit; commits & builds "do not directly correspond to a deliverable" but assumed monotone in work; **build success rate is their only code-quality measure** [primary].

---

## 2. Evidence quality

### Peng et al. 2023 (arXiv 2302.06590) — the "55.8% faster" RCT [vendor: Microsoft/GitHub/MIT authors]
- n=95 randomized (45 treated / 50 control) **Upwork freelancers**; "Thirty-five developers from both the treated and control groups completed the task and survey" — i.e., massive attrition from 95 to ~35 analyzed completers; 5 treated never got Copilot working.
- Population: mostly age 25–34, India/Pakistan, median income $10–19k/yr, ~6 yrs experience. **Not enterprise engineers.**
- Task: greenfield JavaScript HTTP server, 12 visible tests, pay scaled to speed. Treated 71.17 min vs control 160.89 min → 55.8% faster, p=0.0017, **95% CI [21%, 89%]** (very wide). Task success +7pp, **not significant** (CI [-0.11, 0.25]).
- Heterogeneity: bigger gains for less experienced, heavier daily coders, age 25–44.
- Authors' own limits: single standardized greenfield task; "this study does not examine the effects of AI on code quality."
- **Quality grade: internally clean randomization, externally weak** — greenfield solo task, speed-incentivized freelancers, no quality outcome. The most-quoted number in vendor marketing is the least generalizable. **[inference]**

### Ziegler et al. 2022, MAPS / CACM Mar 2024 (arXiv 2205.06537) — the acceptance-rate study [vendor: GitHub]
- 17,420 technical-preview users emailed → 2,047 matched survey responses (~12% response; self-selected enthusiasts; survey users had *higher* acceptance rates than the full population — stated in paper).
- Outcome = **perceived** productivity (11 SPACE-derived Likert items + 1 aggregate). Best telemetry correlate: acceptance rate, **ρ = 0.24, p<0.0001 — i.e., ~6% of variance explained**. Persistence metrics correlated *worse* (unchanged_X_per_accepted ρ = -0.00–0.07). Mean acceptance 26%; acceptance varies by language (JS/Py higher), and by time regime: weekends 23.5%, off-hours 23%, work hours 21.2% (p<0.001).
- The authors **pre-emptively debunked their own headline metric**: "blindly optimizing for a proxy (acceptance rate) … cutting code suggestions into half and suggesting both parts consecutively … would likely increase acceptance rate without substantially increasing, and maybe even while decreasing, user benefit. We can thus not recommend acceptance rate as singular and ultimate criterion of quality." Also the faulty-tab-key thought experiment.
- **Quality grade: honest, transparent (data released), but correlational, perceived-productivity-only, single tool, self-selected sample.** The industry then quoted the headline and ignored Section 8. **[inference]**

### GitHub/Accenture enterprise study, May 2024 (github.blog) [vendor]
- Blog-only RCT report, **no sample size, no CIs, no methods appendix published**. Headline numbers: +8.69% PRs, +15% PR merge rate, "+84% increase in successful builds" (a **volume** count, not a success-rate), ~30% acceptance, 88% character retention, 90–95% satisfaction items, 96% of installers accepted ≥1 suggestion, 67% used ≥5 days/wk.
- Independent press (DevClass) noted it "does not answer questions about potential increased code verbosity or long-term maintainability" and collected practitioner counter-experience [independent].
- Crucial cross-check: the academic analysis of an Accenture Copilot RCT (Cui et al., same data-help acknowledgee Ya Gao) found builds **volume** +92.4% (W-IV, sig.) but **build success RATE −17.4% (SE 7.12, significant; DiD −20.7%***)** at Accenture. The blog surfaced the volume number and omitted the rate decline. Whether it is the exact same experiment is **unverified**, but personnel overlap makes it plausibly the same/related program **[inference]**.
- **Quality grade: marketing summary of a real RCT; treat every number as favorable-selection until reconciled with Cui et al.**

### Cui et al. — "Effects of Generative AI on High-Skilled Work: Three Field Experiments" (SSRN 4945566; Management Science, published online Feb 27 2026) [primary; Microsoft-affiliated authors, independent-journal peer review]
- **The strongest causal evidence that exists**: 3 RCTs (Microsoft, Accenture, anonymous Fortune 100), **4,867 developers**, 2022–2024, post-registered AEARCTR-0014530.
- Pooled treatment-on-treated (IV for imperfect compliance): **weekly merged/completed PRs +26.08% (SE 10.3)**; commits +13.55% (SE 10.0, n.s.); builds +38.38% (SE 12.55); **build success rate −5.53 (SE 3.64, n.s. pooled; significantly negative at Accenture)**. Paper's own caveat: more builds may reflect "trial-and-error coding" — accept suggestion, compile to see if it works.
- Outcome noise is brutal: pre-period weekly PR means 0.13–0.87 with SDs larger than means; "high fraction of zeros … limits our power" — even with ~5,000 devs, effects are barely significant.
- Heterogeneity (Microsoft arm): short-tenure devs adopt more (81.6% vs 72.1%), persist longer, and gain more (+27–39% vs +8–13% for long tenure). **Acceptance-rate heterogeneity: low pre-productivity developers accept significantly more suggestions (p=0.01); higher-tenure devs accept ~4.3% less.** So acceptance rate correlates *negatively* with prior productivity across people.
- **Quality grade: best-in-class design, honest about noise; effects are on volume counts, quality signal is flat-to-negative.**

### ZoomInfo case study, Jan 2025 (arXiv 2501.13282) [practitioner]
- >400 developers, 4-phase rollout (5-engineer pilot → 126-engineer trial, 72 survey responses → org rollout). Acceptance: **33% of suggestions, 20% of suggested lines** (Nov–Dec 2024 window); ~6,500 suggestions/day; 2.8 lines/suggestion; top languages ~30% acceptance; **explicitly chose acceptance rate because of Ziegler et al.** and acknowledged its limit (equal credit for 1-line and multi-line suggestions).
- Self-report: 72% DevSat; "90% … stated that GitHub Copilot reduces the amount of time" with **median 20% time reduction**; 63% said more tasks/sprint; 77% said quality improved. Noted Copilot "struggles with domain-specific logic," inconsistent quality needing extra review scrutiny.
- **Quality grade: honest operational telemetry, no causal design, outcomes are perception + acceptance only.**

### Microsoft telemetry/behavioral papers
- **Mozannar et al., "Reading Between the Lines" (arXiv 2210.14306; CHI 2024)** [primary]: CUPS taxonomy, 21 programmers, retrospectively labeled sessions. **"Thinking/Verifying Suggestion" alone = 22.4% of session time** (largest single state), plus Editing Last Suggestion 11.9% and Prompt Crafting 11.6%. Explicitly: existing metrics like "suggestion acceptance rates, time to accept … tell only part" of the story — acceptance ignores the verification cost the AI itself creates.
- **METR RCT (arXiv 2507.09089, Jul 2025)** [independent] — the boundary condition for all of the above: 16 experienced OSS maintainers, 246 real tasks in mature repos they knew well; allowing early-2025 AI made them **19% slower**, while they *estimated* they were 20% faster. Perception-based and adoption-based measures can carry the wrong sign for experts on familiar, high-context code.

**Net reading of the evidence base [inference]:** vendor RCTs on greenfield/standardized tasks show large speedups; the best multi-firm RCT shows a real but noisy ~26% volume effect concentrated in juniors, with no quality gain and a quality *decline* in one firm; the one independent RCT on experts in familiar code shows a slowdown. Effect direction depends on task familiarity, seniority, and code maturity — which is precisely what a fair per-engineer leverage metric must not punish.

---

## 3. Known failure modes (per signal)

| Signal | Failure modes (evidence-backed) |
|---|---|
| **Acceptance rate** | ρ=0.24 with even *perceived* productivity [Ziegler]; varies with language, time-of-day, weekend/work-hour regime [Ziegler]; inversely related to developer pre-productivity and tenure [Cui]; equal credit for 1-line vs 50-line suggestions [ZoomInfo]; denominator ("shown") is controlled by the vendor's trigger heuristics, so tool updates move the metric with zero behavior change [Ziegler defines shown_per_opportunity; inference]; incomparable across tools (Copilot 26% vs IntelliCode 10% "CTR") [Ziegler]. |
| **Persistence/retention (30–600s)** | Correlates *worse* with perceived productivity than acceptance (ρ≤0.07) [Ziegler]; after enough time code becomes "simply part of the code" — attribution decays [Ziegler]; vendor-only signal. |
| **Task-time (RCT headline numbers)** | Only valid for standardized tasks; Peng's own success-rate effect was n.s.; wide CI; unrepresentative population; no quality outcome [Peng]. Not implementable as an ongoing per-engineer metric at all [inference]. |
| **PR / commit / build counts** | Zero-inflated, SD > mean; underpowered even at n≈5,000 over months [Cui] → a 4-week per-individual window is statistically hopeless [inference mapped to brief problem #3/#4]. PR scope stable only *within* org norms — cross-function comparison invalid [Cui's own theory]. |
| **Build/CI success rate as quality** | Declined significantly at Accenture while build volume rose; "trial-and-error coding" makes builds cheap and frequent [Cui]. Matches client's observed CI-clean-rate saturation (AI iterates to green) — the signal measures *iteration cost visibility*, not quality [inference]. |
| **Self-reported time savings** | METR: felt +20%, measured −19% [independent]; Peng: both arms guessed 35% vs measured 55.8% — perception is miscalibrated in both directions. |
| **Engaged/active users, feature flags** | Measures presence, not value; GitHub's own docs distinguish "active" (any activity) vs "engaged" (intentional action) because raw activity over-counts [vendor docs]. |
| **"% of code by agents" / LoC added** | Rewards verbose generation; GitClear line of evidence (covered by sibling agent) shows AI-era churn/duplication growth; vendor dashboards now expose `loc_added_sum` AND `loc_deleted_sum`, implicitly conceding addition-only counts mislead [vendor docs; inference]. |

### Why vendors moved away from acceptance rate (the arc, with receipts)
1. **2022 — the metric's own authors disclaim it** (Ziegler §8: suggestion-splitting gaming; "cannot recommend acceptance rate as singular and ultimate criterion").
2. **2023–2024 — evidence accumulates that it tracks the wrong thing**: Mozannar shows verification cost invisible to acceptance; Cui shows the *least* productive accept the most; CUPS/CACM framing shifts to "conversation," not completion.
3. **2024–2025 — tool-mix breaks the denominator**: chat, agent mode, CLI, and PR code review don't have "shown suggestions"; a completion-centric ratio covers a shrinking slice of usage. GitHub's replacement schema instruments `chat_panel_agent_mode`, `agent_edit`, `used_cli`, `used_copilot_coding_agent`, "Agent contribution (% LoC by agents)" instead [vendor docs].
4. **2025 — practitioner backlash**: LeadDev (Jul 10, 2025): leaders calling it a "vanity metric"; DX CTO Laura Tacho: "acceptance rate shouldn't be front and center anymore"; GitLab CTO Sabrina Farmer: it's only "the entry point into your code base"; JFrog's Yonatan Arbel: mandates risk "a game of compliance: 'accept more to show I'm productive'" [practitioner].
5. **2025–2026 — GitHub formally sunsets the acceptance-centric APIs**: beta `/copilot/usage` endpoints deprecated Feb 2025; Jan 29 2026 changelog closes down the Copilot Metrics API (sunset Apr 2 2026) and User-level Feature Engagement Metrics API (Mar 2 2026), replaced by "usage metrics" reports centered on engaged users, feature/agent adoption, LoC suggested-vs-added/deleted, PR outcomes, and `ai_credits_used` [vendor/primary changelog].

---

## 4. Gameability & who it penalizes (required red-team, per metric)

**Acceptance rate**
- *Gamed by:* accepting everything then deleting (persistence only partially catches this, and persistence is vendor-side); configuring aggressive completion triggers; working in boilerplate-heavy files/languages where acceptance is naturally high; shifting coding to off-hours/weekends (+2pp free lift per Ziegler); under a mandate, literal "accept to comply" [practitioner: LeadDev].
- *Penalizes:* senior/high-prior-productivity engineers (empirically accept less — Cui); engineers in domains where the model is weak (C/systems/graphics per DevClass anecdotes; HTML/CSS/SQL lower per ZoomInfo); chat/agent-first users who rarely see inline completions; careful reviewers of suggestions (their verification time is the *cost* acceptance ignores — Mozannar).

**Persistence/retention rate**
- *Gamed by:* accepting only trivially-obvious completions (high persistence, low value); not touching generated code again even when it should be revised.
- *Penalizes:* refactorers and people who iterate rapidly on drafts (their legitimate edits count as "non-persistence"); anyone whose work involves deleting AI output as part of quality control.

**PR count / merged PRs**
- *Gamed by:* PR-splitting (explicitly the same shape as Ziegler's suggestion-splitting warning — halve the unit, double the count); trivial dependency-bump PRs.
- *Penalizes:* engineers whose unit-of-work isn't a PR — agent-platform builders, infra/config, ML experimentation (maps directly to brief CANNOT #4); long-cycle work; org functions with big-PR norms. Cui's own identification argument (PR scope stable within org) *breaks* for cross-function individual comparison.

**Commits / builds / CI runs**
- *Gamed by:* commit spam; letting the AI compile-loop (build count rises "for free" — Cui found +38% builds).
- *Penalizes:* engineers in heavy-CI monorepos (more jobs → more chances to be non-clean — matches client's problem #1/#7); trunk-based devs who batch locally.

**Build/CI success or clean rate**
- *Gamed by:* iterating until green before pushing (AI makes this nearly free → metric saturates at ~1.0, exactly what the client observed); pushing only safe changes; avoiding repos with flaky CI.
- *Penalizes:* people doing risky/hard changes, people in repos with 50+ checks or flaky suites; discourages ambitious work (client problem #5 analog).

**Engaged days / active users / feature-usage flags**
- *Gamed by:* one trivial prompt per day (flag flips true); opening agent mode once a month ("Agent adoption" counts "tried an agent in the current calendar month" — a one-shot box-tick per GitHub docs).
- *Penalizes:* part-timers, on-call-heavy weeks, meeting-heavy senior roles, leave-takers — any presence metric is a proxy for hours present [inference].

**LoC suggested/accepted/added; "% code by AI"; `ai_credits_used`**
- *Gamed by:* generating verbose code, letting agents rewrite whole files (inflates agent-contribution %), burning credits on low-value runs (if credits are read as engagement) or hoarding them (if read as cost).
- *Penalizes:* code deleters/simplifiers; low-token high-judgment work (design review, incident response); anyone on cheaper models/tools. Note `ai_credits_used` is the same "AI dollars" object as the client's denominator — GitHub exposes it as a *consumption/adoption* fact, not an efficiency denominator [vendor docs; inference].

**Self-reported time saved**
- *Gamed by:* optimistic answering (costless); anchoring on vendor marketing numbers.
- *Penalizes:* honest skeptics and experts, whose true effect may be negative (METR) and who will report low savings while juniors report high — inverts fairness by seniority [inference].

---

## 5. Applicability to our constraints (mapped to brief HAVE / CANNOT)

**What we can reuse directly (HAVE-compatible):**
- **Adoption-depth over acceptance**: GitHub's own destination (engaged vs active, feature-mix flags, DAU/WAU/MAU, adoption phases) is computable from our HAVE data: per-user per-day per-tool sessions + active days + tool diversity (chat vs CLI vs agent platform). This is the industry-converged, least-gameable *family* of signals, and it covers agent-platform engineers who have spend/sessions but no PRs (brief CANNOT #4). Recommend: breadth (distinct tool classes used), consistency (active days over an 8–12-wk window), and mix-shift toward agentic tools — as *descriptive subscores*, not a ranked efficiency ratio. **[inference from vendor direction + brief]**
- **Window length**: Cui's zero-inflation/noise findings are direct quantitative evidence that 4-week per-individual PR-based scores are noise (client problems #3, #4). PR-count-based anything needs ≥8–12-wk rolling windows and shrinkage toward cohort means. **[inference from primary]**
- **Cohorts**: every credible study conditions on language/function/tenure (Ziegler: language & time regimes; Cui: tenure/level/pre-productivity). Job-level-only cohorts across all functions (client problem #6) contradict the entire evidence base. Function × level cohorts are the minimum.
- **CI-clean-rate**: the evidence actively indicts it — Cui's build-success-rate was the only quality measure and it was flat-to-negative while volume rose; AI compile-looping saturates it. Drop it as a quality *score*; keep per-PR CI outcome only as a data-quality/context annotation. **[inference]**

**What we cannot import (CANNOT-blocked):**
- Acceptance rate, persistence/retention, LoC-suggested-vs-accepted, verification-time — all require IDE/vendor-side suggestion telemetry we do not have (we have dollars/tokens/sessions/days only). Fortunately the evidence says these are the *weakest* signals; being unable to buy them costs little. Persistence/survival is the one vendor signal with a defensible quality theory (wasted-work detection), but even it under-correlated in Ziegler — put it low on the "new signals worth acquiring" list, behind cohort/HR joins and agent-platform run outcomes. **[inference]**
- Task-time RCT designs: not a per-engineer dashboard mechanism; only useful for org-level A/B validation of the score itself (e.g., staggered rollouts à la Cui) — worth recommending as a *validation* method, not a metric.
- "% of code by AI"/agent-contribution: requires AI-vs-human attribution (brief CANNOT #1 — explicitly dropped as unreliable).

**Two portable design lessons [inference]:**
1. Every ratio metric in this literature was gamed at the denominator (shown-suggestions, spend-$, CI-checked-PRs). Prefer counts with caps + cohort-percentile *bands* over ratios; if a ratio survives, its denominator must be something the engineer can't cheaply manipulate.
2. The single most replicated finding — juniors/low-productivity devs adopt more, accept more, gain more — means any adoption- or acceptance-flavored score will systematically read *higher for juniors*. A self-reflection dashboard must either cohort by tenure or present adoption as neutral descriptive facts, never as a "leverage" ranking, or it silently tells senior engineers they are bad at AI (METR suggests they may be *correctly* selective).

---

## 6. Sources (annotated, tagged)

1. **Peng, Kalliamvakou, Cihon, Demirer — "The Impact of AI on Developer Productivity: Evidence from GitHub Copilot"** (Feb 2023). https://arxiv.org/abs/2302.06590 — [vendor] (Microsoft/GitHub/MIT authors). RCT, n=95 Upwork freelancers, ~35 completers analyzed; 55.8% faster (p=0.0017, CI [21,89]); success +7pp n.s.; juniors gain more. Read in full from PDF.
2. **Ziegler et al. — "Productivity Assessment of Neural Code Completion"** (MAPS '22; arXiv 2205.06537). https://arxiv.org/abs/2205.06537 — [vendor] (GitHub). n=2,047 matched survey+telemetry; acceptance rate best correlate of *perceived* productivity, ρ=0.24; persistence worse; §8 gaming warning. Data released at https://github.com/wunderalbert/prod-neural-materials. Read in full from PDF. CACM 2024 version: https://dl.acm.org/doi/10.1145/3633453.
3. **Cui, Demirer, Jaffe, Musolff, Peng, Salz — "The Effects of Generative AI on High-Skilled Work: Three Field Experiments with Software Developers"** (SSRN 4945566; Management Science, online Feb 27 2026). https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 · https://pubsonline.informs.org/doi/10.1287/mnsc.2025.00535 · working PDF https://economics.mit.edu/sites/default/files/inline-files/draft_copilot_experiments.pdf — [primary] (peer-reviewed; Microsoft-affiliated authors). n=4,867; PRs +26.08% (SE 10.3) TOT; build success rate −17.4%** at Accenture; juniors adopt/gain more; low performers accept more. Read Feb 2025 draft in full from PDF.
4. **GitHub Blog — "Research: Quantifying GitHub Copilot's impact in the enterprise with Accenture"** (May 13, 2024). https://github.blog/news-insights/research/research-quantifying-github-copilots-impact-in-the-enterprise-with-accenture/ — [vendor]. +8.69% PRs, +15% merge rate, "+84% successful builds" (volume), 30% acceptance, 88% char retention; no methods/CI published.
5. **DevClass — "GitHub research reports high Copilot satisfaction from enterprise devs… but others doubt productivity gains"** (May 14, 2024). https://devclass.com/2024/05/14/github-research-reports-high-copilot-satisfaction-from-enterprise-devs-but-others-doubt-productivity-gains/ — [independent]. Maintainability/verbosity questions unanswered; practitioner counterexamples.
6. **ZoomInfo — "Experience with GitHub Copilot for Developer Productivity at Zoominfo"** (arXiv 2501.13282, Jan 2025). https://arxiv.org/abs/2501.13282 — [practitioner]. >400 devs; 33% suggestion / 20% line acceptance; 72% DevSat; median self-reported 20% time saving; chose acceptance rate citing Ziegler; states its single/multi-line credit flaw.
7. **Mozannar et al. — "Reading Between the Lines: Modeling User Behavior and Costs in AI-Assisted Programming"** (arXiv 2210.14306; CHI 2024). https://arxiv.org/abs/2210.14306 · https://dl.acm.org/doi/10.1145/3613904.3641936 — [primary] (Microsoft Research). CUPS taxonomy, 21 programmers; Thinking/Verifying Suggestion = 22.4% of session time; acceptance metrics "tell only part" of the story. Stats verified from arXiv PDF.
8. **GitHub Docs — Copilot usage metrics (data reference)** (current, fetched 2026-07-08). https://docs.github.com/en/copilot/reference/copilot-usage-metrics/copilot-usage-metrics — [vendor/primary]. Per-user fields: `code_acceptance_activity_count`, `loc_suggested_to_add_sum`/`loc_added_sum`/`loc_deleted_sum`, `used_agent/used_chat/used_cli/used_copilot_coding_agent/used_copilot_code_review_active`, `ai_credits_used`, `ai_adoption_phase`, per-IDE/language/model breakdowns; PR metrics incl. `total_created_by_copilot`, `median_minutes_to_merge_copilot_authored`; teams <5 seated users excluded from user-teams reports; dashboard "Agent contribution" (% LoC by agents/28d).
9. **GitHub Docs — Copilot Metrics/usage REST endpoints** (fetched 2026-07-08). https://docs.github.com/en/rest/copilot/copilot-metrics — [vendor/primary]. New report endpoints incl. `.../copilot/metrics/reports/users-1-day` and `users-28-day/latest` (per-user granularity), data from Oct 10 2025, 1-yr retention.
10. **GitHub Changelog — "Closing down notice of legacy Copilot metrics APIs"** (Jan 29, 2026). https://github.blog/changelog/2026-01-29-closing-down-notice-of-legacy-copilot-metrics-apis/ — [vendor/primary]. Copilot Metrics API sunset Apr 2 2026; User-level Feature Engagement Metrics API sunset Mar 2 2026; replaced by usage-metrics endpoints (agents, models, LoC suggested vs accepted, fine-grained permissions). Beta usage API deprecation: https://github.blog/changelog/2025-02-21-deprecation-of-beta-github-copilot-usage-api-endpoint/.
11. **LeadDev — "The rise – and looming fall – of acceptance rate"** (Jul 10, 2025). https://leaddev.com/reporting/the-rise-and-looming-fall-of-acceptance-rate — [practitioner]. Tacho (DX), Farmer (GitLab), Arbel (JFrog): vanity-metric framing, mandate-driven gaming, shift to utilization/impact/cost triads.
12. **METR — "Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity"** (Jul 2025, arXiv 2507.09089). https://arxiv.org/abs/2507.09089 · https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ — [independent]. 16 expert OSS devs, 246 tasks: 19% *slower* with AI while estimating 20% faster. Boundary condition on all perception- and adoption-based signals.

Unverified/flagged: exact identity between the GitHub-blog Accenture RCT and Cui et al.'s Accenture experiment(s) — plausible (shared personnel), **unverified**. Ziegler abstract-era "2,631 responses" (intro figure) vs 2,047 matched-to-telemetry used in analysis — both appear in the paper; analysis n is 2,047 (1,763–1,789 per-metric).
