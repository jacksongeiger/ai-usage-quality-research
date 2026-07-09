# Independent & Academic Evidence on AI Dev Productivity — What It Implies for a "Leverage" Metric

Sub-investigator report · 2026-07-08 · Area: METR RCT, Uplevel, academic replications/critiques, downstream quality/security studies, perceived-vs-actual gap. All numbers below were extracted from fetched primary pages unless marked otherwise; my own analysis is labeled **[inference]**.

---

## 1. What it measures & theory

This evidence base tests the assumption chain that every "leverage" metric silently relies on:

```
AI usage ($ / tokens / sessions)  →  faster task completion  →  more merged output  →  more engineering value
```

The current metric (output per AI dollar) assumes each arrow is positive, roughly linear, and homogeneous across people. The independent literature tests each arrow causally (RCTs), observationally (large-scale telemetry), and perceptually (surveys + forecast-vs-actual designs). Theoretical frame: SPACE/DevEx argue no single metric captures productivity; Goodhart/Campbell predict any single ratio under incentive pressure degrades. The empirical question here is narrower: **does AI usage reliably cause more output at all?**

**Answer from the evidence: no — the sign and size of the first arrow vary by task type, codebase maturity, and developer experience; the third arrow (output → value) demonstrably breaks at the org level.**

## 2. Evidence quality (study-by-study)

### 2a. The METR RCT — the strongest independent counter-evidence
- **Becker, Rush, Barnes, Rein (METR), July 2025** [independent] — RCT, 16 experienced OSS maintainers, 246 real tasks (~2h each) on large mature repos (avg 22k+ stars, 1M+ LOC), Cursor Pro + Claude 3.5/3.7 Sonnet. **AI-allowed condition took 19% LONGER** (paper CI roughly +2% to +39% per the 2026 update page's restatement). Perception gap: devs forecast **24% speedup**, still believed **20% speedup after** experiencing slowdown; economists forecast 39% speedup, ML experts 38%. Authors' own generalization caveats are explicit: does NOT show AI fails most developers, other settings, or future models. (https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/, https://arxiv.org/abs/2507.09089)
- **Design strengths**: real tasks, real repos, randomization at issue level, pre-registered-style forecasts, robustness checks. **Weaknesses (per critics and authors)**: n=16 elite maintainers, ~56% had never used Cursor; hourly pay may blunt urgency; deep repo familiarity is near worst-case for AI assistance. METR's rebuttal: no speedup trend across 30–50h of Cursor use, no difference by prior AI experience, excluding each dev's first 8 AI tasks doesn't flip the result (covered in Zvi Mowshowitz's analysis and Sean Goedecke's, both [practitioner]).
- **METR update, Feb 24 2026** [independent] — New data: 57 devs, 143 repos, 800+ tasks, late-2025 models. Original cohort point estimate flips to **−18% (≈18% faster), CI −38% to +9%**; newly recruited devs **−4%, CI −15% to +9%** — both cross zero. Critically, METR reports the RCT design is now breaking down from **selection effects**: devs refuse to work without AI even at $50/hr; **"30% to 50% of developers told us that they were choosing not to submit some tasks because they did not want to do them without AI"**; they call the new data "an unreliable signal" and "only very weak evidence," noting "the true speedup could be much higher among the developers and tasks which are selected out." (https://metr.org/blog/2026-02-24-uplift-update/)
- **[inference]** Net read as of mid-2026: the famous "19% slower" is real but setting-specific and time-specific; the honest state of causal evidence is "sign uncertain, heterogeneity large, and even the best measurement designs are collapsing under selection effects." Any dashboard metric that hard-codes "more AI usage should mean more output" is building on a claim the best available RCTs cannot support.

### 2b. Uplevel — observational, no throughput gain, more bugs
- **Uplevel Data Labs, 2024** [vendor — engineering-analytics vendor, no AI product stake; treat as semi-independent telemetry] — ~800 developers, Copilot access vs no access, real engineering data. **No improvement in PR cycle time or throughput; Copilot group introduced 41% more bugs**; "Sustained Always On" (burnout proxy) fell 28% in the non-Copilot group vs only 17% with Copilot. **Bug measurement methodology is not fully public** (bug-labeled issues; unverified detail) — the 41% should be treated as directional, not precise. (https://uplevelteam.com/blog/ai-for-developer-productivity; independent coverage: https://visualstudiomagazine.com/articles/2024/09/17/another-report-weighs-in-on-github-copilot-dev-productivity.aspx [fetch 403'd; cited from search snippet — secondary])

### 2c. The positive RCTs it contradicts (context: simple/greenfield tasks, vendor-run)
- **Peng, Kalliamvakou, Cihon, Demirer, 2023** [vendor — GitHub/Microsoft authors] — RCT, recruited devs implementing a JS HTTP server (greenfield toy task): Copilot group **55.8% faster**. (https://arxiv.org/abs/2302.06590)
- **Cui, Demirer, Jaffe, Musolff, Peng, Salz — Management Science 2026** [independent-adjacent: academic authors + Microsoft co-authors; field RCTs at Microsoft, Accenture, Fortune 100] — 4,867 developers; pooled **+26.08% completed tasks**. Heterogeneity is the headline for us: **juniors +27–39%, seniors +8–13%**; adoption only ~60% after a year; authors state they **could not evaluate work quality**. (https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566, https://mitsloan.mit.edu/ideas-made-to-matter/how-generative-ai-affects-highly-skilled-workers, https://pubsonline.informs.org/doi/10.1287/mnsc.2025.00535)
- **Paradis et al. (Google), 2024** [vendor — internal] — RCT, 96 Google engineers, one enterprise task: **~21% faster**, authors flag "our confidence interval is large" and warn against generalizing. (https://arxiv.org/abs/2410.12944)
- **[inference]** Reconciliation accepted by most careful readers: AI speed gains are largest for **junior devs on simple/greenfield/unfamiliar-context tasks** and shrink toward zero (or negative, pre-2026) for **experienced devs in large mature codebases**. The Stanford telemetry work (below) independently shows the same gradient.

### 2d. Large-scale telemetry (observational, confounded, but consistent)
- **Faros AI "AI Productivity Paradox," July 2025** [vendor] — 10,000+ developers, 1,255 teams, multi-source telemetry, Spearman correlations, p<0.05, ≥6 companies per reported result. AI users complete **+21% tasks** and merge **+98% more PRs**… but **PR size +154%, review time +91%, +9% bugs per developer, and no significant correlation between AI adoption and company-level throughput/DORA/quality improvement** — "downstream bottlenecks are absorbing the value." (https://www.faros.ai/blog/ai-software-engineering)
- **DORA 2024** [primary] (from prior mission SOURCES.md, re-checked) — 25% increase in AI adoption associated with **−1.5% delivery throughput, −7.2% delivery stability**, while individuals report feeling more productive; DORA 2025 flips throughput positive but stability stays negative. (https://dora.dev/research/2024/dora-report/, https://dora.dev/dora-report-2025/)
- **Denisov-Blanch (Stanford), 2025** [independent, NOT peer-reviewed — conference talks/slides] — telemetry claimed across ~100k engineers, 600+ companies; rework-adjusted productivity: **greenfield/low-complexity ~30–40% gains; brownfield/high-complexity ~0–10%, sometimes negative**; average Copilot-era gain ~7–9% net of rework. Magnitudes unverified against a paper; direction consistent with the RCT literature. (https://getcoai.com/video/does-ai-actually-boost-developer-productivity-100k-devs-study-yegor-denisov-blanch-stanford/, https://proxify.io/articles/stanford-study-of-100000-developers-on-engineering-productivity [secondary])
- **Agarwal, He, Vasilescu, Jan 2026** [independent] — staggered diff-in-diff on OSS repos adopting coding agents (AIDev dataset): velocity gains front-loaded and only when agents are the first AI tool ("diminishing returns to AI assistance"); **static-analysis warnings +18%, cognitive complexity +39%** persist across settings. (https://arxiv.org/abs/2601.13597)
- **Mohamed, Assi, Guizani — systematic review, rev. Mar 2026** [independent] — 39 peer-reviewed studies 2014–2024: benefits widely *perceived*; measured code-quality outcomes "contradictory... contingent on context"; 59% of studies exploratory; perceived-vs-measured gap called out explicitly. (https://arxiv.org/abs/2507.03156)

### 2e. Downstream quality & security of AI code
- **Perry, Srivastava, Kumar, Boneh (Stanford), CCS 2023** [independent] — user study, codex-davinci-002: AI-assisted participants **wrote significantly less secure code AND were more likely to believe they wrote secure code**. The confidence inversion is the key result. (https://arxiv.org/abs/2211.03622)
- **Pearce et al., "Asleep at the Keyboard?", IEEE S&P 2022** [independent] — 1,689 Copilot-generated programs across 89 CWE-relevant scenarios: **~40% vulnerable**. (https://arxiv.org/abs/2108.09293)
- **Fu et al., 2023 (rev. 2025)** [independent] — 733 Copilot/CodeWhisperer/Codeium snippets found in real GitHub projects: **29.5% of Python, 24.2% of JS snippets contain security weaknesses**, 43 CWE categories, 8 in the CWE Top-25. (https://arxiv.org/abs/2310.02059)
- **Veracode GenAI Code Security Report, 2025** [vendor] — 100+ LLMs, 4 languages: **45% of AI-generated code failed security checks**; Java worst (~72% failure); newer/larger models NOT more secure. (https://www.veracode.com/resources/analyst-reports/2025-genai-code-security-report/)
- **GitClear 2024/2025** [vendor] (prior SOURCES.md [11][12]) — 211M changed lines: churn (lines revised <2 weeks) 5.5%→7.9%, copy/paste 8.3%→12.3%, refactoring ~25%→<10% as AI adoption rose.
- **[inference]** Convergent finding across independent academic + two competing vendors + one 2026 causal OSS study: **AI-heavy code ships with more latent defects, duplication, and complexity — and essentially all of it passes CI.** This directly refutes CI-clean-rate as a sufficient quality adjuster.

### 2f. Perceived vs actual productivity gap
- METR: −19% actual vs +20% believed (the canonical number). [independent]
- **Ziegler et al. 2022** [vendor] (prior SOURCES [9]): acceptance rate predicts *perceived* productivity — i.e., the best telemetry proxies track feelings, not output.
- **Stack Overflow 2025 survey** [primary survey, n≈49k] — trust in AI accuracy fell to **29%** (46% actively distrust, 3% highly trust); **#1 frustration (66%): "AI solutions that are almost right, but not quite"; 45%: debugging AI-generated code is more time-consuming**. (https://survey.stackoverflow.co/2025/ai, https://stackoverflow.co/company/press/archive/stack-overflow-2025-developer-survey/)
- **Butler, Suh, Haniyur, Hadley, "Dear Diary" RCT, 2024** [vendor — Microsoft Research authors, workplace RCT + diary] — sustained use raises perceived usefulness/enjoyment; **trust in AI code quality remains unchanged**. (https://arxiv.org/abs/2410.18334)
- **[inference]** Self-report of AI time-savings is systematically inflated (METR gap; DORA individual-vs-system paradox), yet perception is not worthless — it is a valid *satisfaction/experience* signal (SPACE "S"), just not a valid *output* signal. Use surveys as a calibration/context layer, never as the leverage numerator.

## 3. Known failure modes (of metrics built on the "AI → output" assumption)

1. **Sign instability**: the same population can show −19% → −18%-to-+9% across 12 months as models change (METR 2025 → 2026). A leverage score assuming positive returns to spend will mislabel entire eras and cohorts.
2. **Heterogeneity swamps the mean**: junior/greenfield +27–55% vs senior/brownfield ~0±10%. Level-only cohorts (the current design) compare people whose *achievable* leverage differs by 3–5x for structural reasons.
3. **Output inflation without value**: individual PR/task counts rise (Faros +98% merged PRs) while org throughput doesn't move and PR size (+154%), review load (+91%), and bugs (+9%) rise — the metric's numerator can grow *because* quality is falling and cost is shifting to reviewers.
4. **CI-blindness**: 25–45% security-flaw rates and +18–39% complexity/warning growth all occur in code that compiles and passes tests; plus AI iterates until green (brief's own finding: CI-clean-rate ~1.0 for most). CI-clean is close to zero-information as a quality adjuster.
5. **Selection effects**: any observational comparison of AI users vs non-users (or high vs low spenders) is confounded by who chooses to use AI and for which tasks (METR 2026: 30–50% task withholding). Cross-person leverage rankings inherit this bias wholesale.
6. **Perception miscalibration**: self-estimates of AI benefit run ~40pp optimistic in the only forecast-vs-actual design we have (METR). Survey-anchored "time saved × salary" ROI math (common in vendor frameworks) inherits this inflation.

## 4. Gameability & who it penalizes (per metric touched)

| Metric | How an engineer games it | Who it unfairly penalizes |
|---|---|---|
| Merged-PR count (per $ or raw) | Split work into many small PRs; auto-generate trivial PRs (deps, formatting, docs); agent-bulk changes. Faros shows AI already organically inflates this +98% with no org-level gain | Engineers in mature/complex codebases (structurally fewer, harder PRs — the METR cohort); reviewers; agent-platform/infra engineers with no PR output at all (brief's stated blind spot) |
| Lines changed (winsorized) | Verbose generated code, vendored/boilerplate, churn rewrites; GitClear shows AI inflates added-LOC while refactoring collapses | Deleters/refactorers/simplifiers; frontend or config-heavy roles vs codegen-heavy ML roles; careful reviewers of their own output |
| CI-clean-rate | Let the agent iterate to green (default behavior); avoid repos with strict CI; push only when green locally; add fewer tests | Heavy-CI repos (50+ jobs), experimenters, people who open PRs early for feedback |
| Task/ticket completion count (Cui/Faros-style) | Slice tickets thinner; reclassify work as tickets | Ops/glue/mentoring work; long-cycle projects |
| Completion/cycle time | Start branch late, force-push to compress apparent time (GitClear's gaming catalog); do work before opening PR | Complex-task owners — and we have NO task-difficulty signal to normalize (hard limit) |
| Bug/defect rate (Uplevel-style) | Under-label bugs; fix silently without tickets; reclassify severity | Whoever inherits legacy hotspots; teams with honest defect-labeling culture. NOTE: we CANNOT compute this anyway (no per-author defect linkage) |
| Suggestion acceptance rate | Accept everything then rewrite (counts as accepted); measures tool quality more than user skill (Ziegler: correlates with *perceived* productivity) | Skeptical/careful users — exactly the ones Perry et al. show write more secure code. Not in our HAVE list anyway |
| Self-reported time savings | Inflate; or honestly miscalibrated upward (~40pp per METR) | Honest and pessimistic reporters; seniors whose true gains are genuinely small — they'd look like "bad AI users" while being the best-calibrated people in the building |
| Output per AI dollar (current metric) | **Minimize the denominator**: use cheapest models, avoid the agent platform, do AI work on personal accounts; keep output constant | Heavy legitimate spenders (hard problems, agent builders — brief problems #2/#5); seniors in brownfield code whose achievable per-dollar output is structurally lower |

**[inference]** The deepest gameability problem is not any one numerator — it is that the *ratio form* rewards suppressing AI experimentation (denominator) while the evidence says the org's actual goal (durable, high-quality output) is invisible to every numerator we have.

## 5. Applicability to our constraints (HAVE / CANNOT mapping)

What this evidence base licenses us to build with the data we HAVE, and what it forbids:

- **FORBIDDEN by evidence — cross-sectional "usage → output" scoring.** With no task difficulty (CANNOT), no AI-attribution (CANNOT), and selection effects dominating observational comparisons (METR 2026, Faros), percentile-ranking people on output-per-dollar has no causal foundation. It measures task mix + codebase maturity + seniority, then attributes the residual to "AI leverage."
- **SUPPORTED — within-person, over-time designs.** The only defensible causal-ish unit without an RCT is each engineer vs their own rolling baseline (pre/post adoption, quarter-over-quarter), which we CAN build from HAVE data (per-day usage across ALL tools incl. agent platform + PR counts/lines + CI). This also satisfies the privacy constraint (self-view only) natively. Caveat [inference]: within-person trends still confound task-mix drift; show as trends, not scores.
- **SUPPORTED — heterogeneity-aware cohorts.** Junior-vs-senior and greenfield-vs-brownfield gradients (Cui; Stanford; METR) mean cohorts must be at least function × level (we HAVE role/level/team), and expectations must differ by cohort; the current level-only, cross-function cohort is empirically indefensible.
- **WEAKLY SUPPORTED — CI as one of several guardrails.** Keep CI signal only as a *floor/anomaly detector* (sudden clean-rate drops), never as a quality multiplier; the security/complexity literature shows most AI-induced quality loss is CI-invisible, and the brief documents saturation at ~1.0.
- **UNAVAILABLE but high-value (new-signal candidates ranked by evidence):** (1) **post-merge churn/revert of one's own PRs within 2–4 weeks** (GitClear churn; the single best metadata-only durability proxy — needs diff/commit lineage, not code semantics); (2) **review-burden signals** (review time on one's PRs, PR size distribution — Faros +91%/+154% shows where AI cost hides); (3) **static-analysis warning deltas** (Agarwal 2026; gate-level counts, no code reading); (4) periodic **calibrated self-report** (forecast-then-check style, exploiting the METR gap as a *feature*: show engineers their own perception-vs-telemetry delta).
- **Agent-platform engineers**: nothing in this literature measures agent-builder output; the field has no answer yet (honest dead end). Closest analog is DX's "human-equivalent agent hours" [vendor, unvalidated]. Their invisibility in PR-based numerators is confirmed as structural, not fixable by reweighting.
- **Data lag (~2 weeks)** is compatible with within-person rolling windows of 8–12 weeks; incompatible with 4-week snapshot scoring [inference].

## 6. Sources (annotated, tagged)

1. **METR RCT (blog + paper)** — [independent] — RCT, 16 devs/246 tasks, 19% slowdown; forecast gap +24/+20 vs −19; economists +39/ML experts +38 wrong. https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ · https://arxiv.org/abs/2507.09089
2. **METR uplift update (Feb 24, 2026)** — [independent] — 57 devs/143 repos/800+ tasks; original cohort −18% [CI −38,+9], new devs −4% [CI −15,+9] (negative = faster); severe selection effects; "only very weak evidence." https://metr.org/blog/2026-02-24-uplift-update/
3. **Uplevel Data Labs 2024** — [vendor, no AI-product stake] — ~800 devs; no cycle-time/throughput gain; +41% bugs; burnout-reduction gap. Bug methodology not fully public (unverified detail). https://uplevelteam.com/blog/ai-for-developer-productivity
4. **Peng et al. 2023** — [vendor: GitHub/MSFT] — toy-task RCT, +55.8% speed. https://arxiv.org/abs/2302.06590
5. **Cui et al., Management Science 2026** — [independent-adjacent field RCTs, MSFT co-authors] — 4,867 devs, +26.08% tasks; juniors +27–39% vs seniors +8–13%; ~60% adoption; quality unmeasured. https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 · https://mitsloan.mit.edu/ideas-made-to-matter/how-generative-ai-affects-highly-skilled-workers
6. **Paradis et al. (Google) 2024** — [vendor-internal RCT] — 96 engineers, ~21% faster, wide CI, explicit non-generalization caveat. https://arxiv.org/abs/2410.12944
7. **Faros AI Productivity Paradox, Jul 2025** — [vendor telemetry] — 10k+ devs: +21% tasks/+98% merged PRs individually; PR size +154%, review time +91%, +9% bugs/dev; no company-level gain. https://www.faros.ai/blog/ai-software-engineering
8. **DORA 2024 / 2025** — [primary] — adoption ↔ −1.5% throughput/−7.2% stability (2024); stability still negative in 2025. https://dora.dev/research/2024/dora-report/ · https://dora.dev/dora-report-2025/
9. **Perry et al., CCS 2023 (Stanford)** — [independent] — AI-assisted users wrote less secure code and were MORE confident it was secure. https://arxiv.org/abs/2211.03622
10. **Pearce et al., IEEE S&P 2022** — [independent] — 1,689 Copilot programs, ~40% vulnerable in CWE scenarios. https://arxiv.org/abs/2108.09293
11. **Fu et al. 2023/2025** — [independent] — in-the-wild Copilot code: 29.5% (Py)/24.2% (JS) with CWE weaknesses. https://arxiv.org/abs/2310.02059
12. **Veracode GenAI Code Security 2025** — [vendor] — 100+ models: 45% of generations fail security checks; newer models not better. https://www.veracode.com/resources/analyst-reports/2025-genai-code-security-report/
13. **Agarwal, He, Vasilescu, Jan 2026** — [independent] — diff-in-diff, OSS agent adoption: front-loaded velocity only for AI-naive repos; static warnings +18%, cognitive complexity +39%. https://arxiv.org/abs/2601.13597
14. **Mohamed, Assi, Guizani systematic review (rev. Mar 2026)** — [independent] — 39 studies: perceived benefits broad, measured quality contradictory/context-contingent. https://arxiv.org/abs/2507.03156
15. **Stack Overflow Developer Survey 2025** — [primary survey, n≈49k] — trust 29% (46% distrust); 66% "almost right" frustration; 45% debugging-AI-code time sink. https://survey.stackoverflow.co/2025/ai
16. **Butler et al., "Dear Diary" 2024** — [vendor: MSFT Research workplace RCT] — perceived usefulness rises with use; trust in AI code unchanged. https://arxiv.org/abs/2410.18334
17. **Zvi Mowshowitz, "On METR's AI Coding RCT"** — [practitioner] — best consolidated critique/defense (Shear learning-curve objection "somewhat convincing" but no-improvement-over-50h "hard to explain"). https://thezvi.substack.com/p/on-metrs-ai-coding-rct
18. **Sean Goedecke / Pragmatic Engineer analyses** — [practitioner] — replication-quality discussion of METR's robustness checks. https://www.seangoedecke.com/impact-of-ai-study/ · https://newsletter.pragmaticengineer.com/p/cursor-makes-developers-less-effective
19. **Denisov-Blanch (Stanford), 2025 talks** — [independent, not peer-reviewed; magnitudes unverified] — ~100k devs telemetry: greenfield 30–40% vs brownfield 0–10% rework-adjusted gains. https://getcoai.com/video/does-ai-actually-boost-developer-productivity-100k-devs-study-yegor-denisov-blanch-stanford/
20. **GitClear 2025 (2024 data)** — [vendor] — churn 5.5%→7.9%, copy/paste 8.3%→12.3%, refactoring collapse (prior SOURCES [12], re-used). https://www.gitclear.com/ai_assistant_code_quality_2025_research
