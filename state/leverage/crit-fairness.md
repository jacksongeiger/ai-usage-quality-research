# Fairness Audit — Current "AI Leverage" Metric (Critique, Phase B)

Sub-investigator: fairness audit across function, repo environment, seniority, work mode, employment bucket; explicit critique of the job-level-only cohort. Evidence drawn from Phase-A files under `state/leverage/survey/` and `state/leverage/repos/` (all external claims re-cited with URLs and tags); my own reasoning is labeled **[inference]**. Date: 2026-07-08.

**The metric under audit** (BRIEF.md): score = mean of two percentile ranks within a job-level cohort (split only FT/contractor/intern):
- S1 = (merged PRs × CI-clean-rate) / AI-coding-CLI spend $
- S2 = (winsorized lines ≤2000/PR × CI-clean-rate) / AI-coding-CLI spend $
- CI-clean-rate defaults to 1.0 under 3 CI-checked PRs; 4-week window; floors: ≥3 merged PRs, ≥$5 spend, ≥3 active days, ≥14 days.

**Headline:** the score is not one unfair metric but a *stack* of biased components multiplied together, then percentile-ranked in a cohort that controls for none of the bias axes. Because S1 and S2 are strongly correlated (both volume-shaped, same denominator, same CI multiplier), averaging them diversifies nothing — the score is effectively "volume per measured dollar," which amplifies every axis bias below. **[inference]** Averaging two percentile ranks is itself statistically unjustified (ordinal scale; psychometrics combines on z-scale — see goodhart-design.md §3.1, https://en.wikipedia.org/wiki/Percentile_rank [reference]), and arithmetic-averaging normalized ratios makes rankings normalization-dependent (Fleming & Wallace, CACM 1986, via goodhart-design.md [primary]).

---

## 1. Component-level gaming & penalty table (required per-metric answers)

| Component | (a) How an engineer games it | (b) Who it unfairly penalizes |
|---|---|---|
| Merged-PR count / $ | PR-splitting (also defeats the 2000-line cap simultaneously); trivial dep-bump/formatting PRs; timing merges across the 4-week window boundary; **shrinking the denominator** — route heavy AI use through unmetered chat tools, personal accounts, or the (unmeasured) agent platform so measured spend → $5 floor (roi-cost.md §4.1 [inference grounded in Meta/Amazon leaderboard gaming, Fortune 2026-07-07 https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/ [independent]) | Mature/brownfield-codebase engineers (fewer, harder PRs — METR https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ [independent]); ML/experimentation roles whose branches legitimately never merge; agent-platform/infra/non-PR engineers (zeroed); long-cycle work; reviewers |
| Winsorized lines / $ | Verbose generated code; lockfiles/vendored/codegen output; splitting a 6k-line PR into 3×2k (beats the published cap AND triples PR count — goodhart-design.md §4) | Deleters/refactorers/simplifiers (Tornhill: "in case anything should be rewarded it would be a net deletion of code," code-maat README [primary]); terse-language and config-heavy roles; functions whose legitimate unit-of-work exceeds 2000 lines (data migrations, codegen) — the fixed cap binds differentially by function |
| CI-clean-rate multiplier | Iterate to green before pushing (near-free with AI — saturates at ~1.0, brief problems #1/#7; Cui et al.: build volume +38% while success rate flat-to-negative, Accenture −17.4% significant, https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 [primary]); avoid strict/flaky-CI repos; keep CI-checked PRs under 3 to bank the free 1.0 | Heavy-CI repos (50+ jobs — mechanically more chances to fail); early-feedback/draft-PR users; risk-takers; honest experimenters |
| The 1.0 default (<3 checked PRs) | Deliberately keep scored-PR count low; do work in no-CI repos | It's a **regressive subsidy**: engineers in the *least*-verified environments get a guaranteed-perfect quality multiplier while heavy-CI engineers get a real (lower) one **[inference]** |
| Coding-CLI-only spend denominator | Spend leakage (above); cheapest-model theater; prompt-frugality that degrades quality invisibly (no defect linkage) | Heavy *legitimate* spenders — Jellyfish: top-quintile users ~2x merged PRs on ~10x tokens, $89.32 vs $0.28 cost/PR, so the ratio ranks the demonstrably most productive engineers LAST (https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/ [vendor]); large-context monorepo workers (input tokens dominate cost — arXiv 2604.22750 https://arxiv.org/abs/2604.22750 [independent]); anyone doing harder work (brief problem #5) |
| Cold-start floors (≥3 PRs, ≥$5, ≥3 days, ≥14d) | Time PRs/spend across boundaries to flip the score on/off (brief problem #4) | Part-timers, leave-takers, on-call-heavy weeks, new hires, interns mid-ramp, agent-platform engineers (structurally under 3 PRs) — presence proxies penalize hours-not-present (copilot-studies.md §4) |
| Percentile-in-level-cohort, averaged | Not directly gameable in self-view, but rewards matching cohort norms over good work; collective drift poisons the norm | Minority functions inside a mixed cohort; everyone during adoption waves (zero-sum: your score falls when colleagues adopt faster, with no behavior change — goodhart-design.md §3.1 **[inference]**) |

---

## 2. Axis-by-axis audit

### 2.1 Function (backend / frontend / ML / infra-platform / SRE / data)

**Mechanisms.**
1. *Numerator norms differ structurally.* PR count and line counts are function-shaped: frontend ships many small PRs; data-eng ships rare huge migrations (the 2000-line cap truncates their legitimate output while never binding on frontend); ML work is experimentation-heavy — much of the AI-assisted effort never merges at all, so both numerators under-count it. Cui et al.'s own identification argument — PR scope usable as a task proxy only because it is "roughly stable *within* an org['s norms]" — explicitly breaks for cross-function individual comparison (copilot-studies.md §2, [primary]).
2. *Denominator norms differ structurally.* Agentic workflows are ~1000x more token-expensive than chat, and input tokens (context size) dominate cost (arXiv 2604.22750 [independent]) — so ML/infra engineers in large-context repos pay more per identical task. Function determines tool-mix, tool-mix determines measured spend.
3. *Achievable AI gain differs structurally.* Greenfield ~30–40% vs brownfield ~0–10% rework-adjusted gains (Denisov-Blanch, Stanford talks — magnitudes unverified, [independent, not peer-reviewed]); DORA 2025's amplifier thesis makes leverage conditional on environment (https://dora.dev/dora-report-2025/ [primary]).

**Magnitude.** Jellyfish shows a ~300x cost-per-PR spread across spend quintiles in a 12k-dev population [vendor]; same-task token cost varies up to 30x run-to-run [independent]. Against that, the largest *behavioral* effect in the causal literature is ~1.26x (Cui pooled +26%). **[inference]** Function/environment membership therefore plausibly explains more of an engineer's percentile than anything the engineer does — the score is substantially a function classifier wearing a leverage costume.

**Cohort-fixable?** Partially. Function×level cohorts (with hierarchical partial pooling, not hard splits — goodhart-design.md §5) remove the cross-function *comparison* injustice and are mandatory (every credible study conditions on function/language/tenure). But cohorting cannot repair the numerator for ML/experimentation work (unmerged output invisible) or the denominator anywhere (noise + non-stationarity, §2 of roi-cost.md). **Verdict: comparison layer fixable by cohort redesign; the per-dollar ratio itself remains broken.**

### 2.2 Repo environment (CI job count; monorepo vs many small repos; merge workflow)

**Mechanisms.**
1. *CI-clean-rate is a repo property, not a person property.* With per-job flake/failure probability f and J independent required jobs, P(clean at head commit) ≈ (1−f)^J: at f=1%, one job → 99%, 50 jobs → ~60%. **[inference, illustrative arithmetic]** A 40-point quality-multiplier gap can arise from repo choice alone, with identical engineer behavior. This is the brief's own problem #1 quantified.
2. *Two failure modes pushing in opposite directions make the signal pure noise.* AI iterate-to-green saturates the rate toward 1.0 (Cui: build volume +38%, success rate flat-to-negative [primary]) while heavy-CI repos mechanically depress it — so observed variation is dominated by environment and luck, not quality. **[inference]**
3. *PR granularity and line counting are workflow artifacts.* Monorepos batch changes into fewer, larger PRs (fails the PR-count numerator, hits the line cap); many-small-repo teams generate many small PRs (inflates it). Squash-merge vs trunk-based changes what is visible per PR — GitClear's file documents the identical mechanism for churn: "Two engineers with identical behavior in different repos get different [measurements]" (gitclear.md §3, [inference, mechanically certain from definitions]). Release-train/regulated environments (mobile, firmware) batch merges the way they batch deploys (dora.md gaming table [primary/practitioner]).
4. *Context size drives spend.* Monorepo engineers carry structurally larger input-token bills for the same task (arXiv 2604.22750 [independent]).

**Magnitude.** Repo effects dwarf person effects wherever both are measured: code half-life spans 0.32y (Angular) to 6.6y (Linux) — a ~20x repo spread (https://erikbern.com/2016/12/05/the-half-life-of-code.html [primary/practitioner]); the CI arithmetic above gives tens of percentile points. DevLake/fourkeys both treat deployment/CI semantics as *per-repo configuration*, conceding cross-repo comparability requires centralized definitions (repos/devlake.md, repos/fourkeys.md [primary code]).

**Cohort-fixable?** **No.** Cohorts are person-attribute groupings (function, level); repo environment is a work-context attribute, and one engineer works across several repos in a window. Fixing it needs per-repo normalization of the signal itself (e.g., repo-cohort EB priors per goodhart-design.md fix #1) — and even then CI-clean-rate stays saturated. **Verdict: signal broken. CI-clean-rate should be removed as a multiplier regardless of any cohort redesign; PR/line numerators need repo-normalization no cohort provides.**

### 2.3 Seniority (L3–L8; seniors review/design more, merge less)

**Mechanisms.**
1. *The gains gradient is real and inverted.* Juniors gain +27–39% from AI vs seniors +8–13% (Cui, n=4,867 [primary]); low-pre-productivity devs accept significantly more suggestions (p=0.01); METR's experts were 19% *slower* with AI while believing +20% (https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ [independent]). Any volume-per-dollar score reads higher for juniors doing greenfield work and tells correctly-selective seniors they are "bad at AI."
2. *Level cohorts control for title, not work-mode.* The current design's one defensible instinct — same-level comparison — fails within level: an L6 IC who codes 80% of the time and an L6 tech lead who reviews/designs/unblocks 60% of the time sit in the same cohort, and the second is invisible to both numerators. SPACE's hard rule — activity metrics "never used in isolation either to reward or to penalize developers" — names exactly this uncounted work (https://queue.acm.org/detail.cfm?id=3454124 [primary]). Faros telemetry shows where AI-era senior time actually goes: review time +91% on +154% bigger PRs (https://www.faros.ai/blog/ai-software-engineering [vendor]) — the metric charges reviewers' time to *authors'* scores. **[inference]**
3. *Harder work costs more.* Seniors hold the brownfield, high-blast-radius surfaces; per-dollar ratios punish spending more on harder work (brief problem #5; spend is NOT a usable difficulty adjuster — arXiv 2604.22750 [independent]).

**Magnitude.** 2–4x structural difference in achievable gains by tenure (Cui); review/design share of senior time is unmeasured but plausibly 30–60% at L6+ **[inference, unverified]** — that fraction of their contribution scores exactly zero.

**Cohort-fixable?** Partially. Level cohorts (kept) plus function cohorts help; but no cohort makes review/mentoring/design visible to a merged-PR numerator — the signal does not observe the work. **Verdict: cohort redesign necessary but insufficient; senior work-mix requires non-output signals (review-burden metadata, survey) or explicit "outputs measured cover X% of this role" honesty.**

### 2.4 Work mode (agent-platform builders; non-PR work)

**Mechanisms.** Agent-platform engineers have high measured spend (denominator) and ~zero merged PRs (numerator). Outcome is deterministic, not noisy: either the ≥3-PR floor excludes them (no score) or they occupy the bottom percentiles by construction. The brief itself validates this (problem #2). There is **no published methodology anywhere** that scores an individual by the agents/automation they build (agent-native.md headline, honest dead end); DORA 2024's platform J-curve (~5% productivity gain alongside ~8% throughput / ~14% stability decrease during build-out — verify exact figures in the PDF) is direct evidence that delivery metrics *mis-price* platform work in-flight (https://dora.dev/research/2024/dora-report/ [primary]). The invisible-labor literature (arXiv 2401.06889 [independent]; Tanya Reilly "Being Glue" https://www.noidea.dog/glue [practitioner]) shows this class of unfairness pushes people off technical tracks when scores leak into status — even a "self-view" score shapes self-assessment and morale. **[inference]**

**Gaming note (required):** even the defensive fix is gameable — if "agent-platform profile" users get a separate cohort or exemption, an engineer can shift spend onto the platform to escape the PR-denominated score (roi-cost.md spend-leakage family). **Penalizes:** conversely, hybrid engineers (half PRs, half platform) straddle profiles and fit neither. **[inference]**

**Magnitude.** Total. This is the worst axis: the metric isn't biased against them at the margin — it is *categorically wrong* about them.

**Cohort-fixable?** Only defensively. Splitting agent-platform-profile users into their own cohort and refusing to divide their spend by PR output stops the punishment (agent-native.md §5) — but produces no valid score, because HAVE data contains no output surface for them (engagement/consistency signals are participation proxies and must be labeled as such). **Verdict: signal missing, not broken. Requires NEW signals (per-run agent_id/owner_id/caller identity → adoption-by-others, repeat use) or an explicit, honest "unmeasured" state. Anything else is false precision.**

### 2.5 Employment bucket & exposure time (FT / contractor / intern; part-time, leave, on-call)

**Mechanisms.**
1. *The buckets chosen are the least informative axis.* FT/contractor/intern splits populations whose main measurable differences are procedural, while the variance drivers (function, repo, tenure, exposure time) go uncontrolled. **[inference]**
2. *Contractors:* commonly excluded from AI-tool procurement/licenses or restricted to client-approved tools **[inference, unverified as to this org]** → measured spend ≈ $0 → either no score (floor) or a divide-by-near-minimum ratio that inflates their percentile relative to metered FTEs. Either way the number reflects procurement, not leverage.
3. *Interns:* correctly separated (they'd otherwise dominate — juniors show the largest AI gains, Cui [primary]); but a 10–12-week internship against a ≥14-day window, ≥3-PR, ≥3-active-day floor set means much of the internship is spent in cold-start; the 4-week window makes their score mostly ramp-noise. **[inference]**
4. *Part-time / leave / on-call:* every numerator is an hours-present proxy (copilot-studies.md §4: "any presence metric is a proxy for hours present"); floors flip scores on/off month to month (brief problem #4); and the standard statistical fix (EB shrinkage) itself pulls low-volume *excellent* performers toward the cohort mean (goodhart-design.md §4) — they get penalized by the disease and then again by the cure unless intervals are displayed.

**Magnitude.** On/off score flicker at floor boundaries (documented, brief problem #4); for contractors, category-level distortion of unknown sign — worth an explicit data check before any redesign. **[inference]**

**Cohort-fixable?** Mostly yes — this is the one axis where design mechanics (not the signal concept) are the problem: pro-rate denominators by active days, lengthen to rolling 8–12 weeks, replace floors with shrinkage + displayed uncertainty and explicit "insufficient observation" states (fourkeys zero-filled windows; git-of-theseus right-censoring pattern; goodhart-design fixes #1–2). **Verdict: fixable by window/floor/normalization redesign; keep the intern separation, audit the contractor spend-coverage assumption.**

---

## 3. The job-level-only cohort — explicit critique

1. **It controls for the axis with modest explanatory power and ignores the dominant ones.** Level correlates with achievable AI gain (Cui tenure gradient) — but function, repo environment, and work-mode drive far larger structural spreads (§2.1–2.4). A level-only, cross-function percentile is a "disparate comparison" of exactly the kind DORA rejects — Nathen Harvey: "These truly are team metrics… rolling them up… there is a lot of peril" (https://getdx.com/podcast/masterclass-on-dora-metrics/ [primary-interview]); the amplifier thesis makes leverage conditional on environment, so ranking across environments attributes the environment to the person (dora.md §implications [primary + inference]).
2. **Every credible framework segments by persona/function.** DevEx: break results down "by team and persona — role, tenure, seniority"; comparing raw scores across roles is invalid (https://queue.acm.org/detail.cfm?id=3595878 [primary]). DX Core 4 stamps its throughput metric "Not at individual level" even *within* its cohorting (https://getdx.com/research/measuring-developer-productivity-with-the-dx-core-4/ [vendor]).
3. **The naive fix is also wrong.** Hard function×level splits give ~36 cells over 500–5000 engineers; percentile resolution is 100/n (a 12-person cell moves in ~8-point jumps; one joiner reshuffles everyone). The correct tool is a hierarchical model (engineer ⊂ function×level, partial pooling) with uncertainty display — goodhart-design.md §3.1/§5 [primary/technical].
4. **Percentiles are zero-sum and non-stationary.** Half of every cohort is below median forever; during an adoption wave the whole distribution shifts monthly, so an engineer's score moves with no behavior change — corrosive for a *motivational, self-view* product and uninterpretable (violates two of the brief's five stated properties). **[inference, grounded in Campbell/goodhart-design.md]**
5. **The FT/contractor/intern buckets are the design's only cohort nuance, and they answer a compliance question, not a fairness one.** The bucket boundary that matters — "PR-output profile vs agent-platform profile vs hybrid" — is absent entirely. **[inference]**

---

## 4. Cross-cutting statistical unfairness: noise presented as signal

Even inside a perfectly chosen cohort, the score is unreliable at the individual×4-week grain, and unreliability *is* unfairness when displayed as a personal skill measure **[inference]**:
- PR counts are zero-inflated with SD > mean; Cui was underpowered at n≈5,000 devs over months → one person over 4 weeks is statistical noise (copilot-studies.md §2/§3 [primary]).
- Spend is 30x run-to-run variable on identical tasks [independent] and non-stationary (prices −90% since 2023; Copilot credits transition 2026 — roi-cost.md [independent/vendor]): the denominator drifts quarter to quarter for reasons no engineer controls.
- CI-clean-rate is saturated near 1.0 (brief; Cui), so tiny denominators of checked PRs produce coarse jumps (1/3, 2/3, 1.0) that decide top-scorer rank — matching the brief's observation that "top scorers often just have marginally better CI rate" (problem #3).
- The two averaged signals are correlated by construction (shared denominator, shared multiplier, volume-shaped numerators), so the composite has ~one effective dimension while presenting as two — violating SPACE's metrics-in-tension requirement (space-devex.md §implications [primary + inference]).

---

## 5. Verdict matrix

| Axis | Mechanism (short) | Rough magnitude | Cohort redesign enough? |
|---|---|---|---|
| Function | PR/line/spend norms + achievable-gain differences are function properties | 3–5x structural gains spread; ~300x cost-per-PR spread vs ~1.26x behavioral effects | Partial — function×level pooling fixes comparison; per-dollar ratio + volume numerator stay broken |
| Repo environment | CI-clean = (1−f)^jobs; PR granularity & line counts are workflow artifacts; context size drives spend | up to ~40pt CI multiplier gap [inference calc]; ~20x repo half-life spread | **No** — needs per-repo signal normalization; CI-clean-rate should be dropped as multiplier outright |
| Seniority | Juniors gain/accept more; senior review/design/mentoring invisible to numerators; harder work costs more | 2–4x tenure gains gradient; unmeasured 30–60% of senior work-mix [inference] | Partial — no cohort makes unobserved work visible; needs non-output signals |
| Work mode (agent/non-PR) | Spend without PR-shaped output → deterministic bottom rank or exclusion | Total (categorical, not marginal, error) | **No** — cohort split only stops punishment; valid scoring needs NEW signals or honest "unmeasured" |
| Employment/exposure | Floors + presence-proxy numerators + unmetered contractor spend | On/off score flicker; category-level contractor distortion | **Mostly yes** — pro-rating, 8–12wk windows, shrinkage+intervals, censoring states |

**Bottom line for the orchestrator:** cohort redesign (function×level, hierarchical, plus work-mode profiles) is necessary and fixes roughly the comparison layer — but three of the metric's four components (CI-clean multiplier, dollar denominator, volume numerators for non-PR/senior/ML work) are unfair *as signals*, and no cohort geometry repairs a signal that is saturated, non-stationary, or blind to the work being valued.

## 6. Source key (external, as cited above)
- Cui et al., Mgmt Sci 2026 [primary] https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566
- METR RCT [independent] https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ · update https://metr.org/blog/2026-02-24-uplift-update/
- Jellyfish tokenmaxxing data [vendor] https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/
- Bai et al., arXiv 2604.22750 [independent] https://arxiv.org/abs/2604.22750
- Faros AI Productivity Paradox [vendor] https://www.faros.ai/blog/ai-software-engineering
- DORA 2024/2025 [primary] https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report · https://dora.dev/dora-report-2025/
- Harvey interview [primary-interview] https://getdx.com/podcast/masterclass-on-dora-metrics/
- SPACE [primary] https://queue.acm.org/detail.cfm?id=3454124 ; DevEx [primary] https://queue.acm.org/detail.cfm?id=3595878
- DX Core 4 [vendor] https://getdx.com/research/measuring-developer-productivity-with-the-dx-core-4/
- Fortune tokenmaxxing [independent] https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/
- Denisov-Blanch (Stanford, unverified magnitudes) [independent] https://getcoai.com/video/does-ai-actually-boost-developer-productivity-100k-devs-study-yegor-denisov-blanch-stanford/
- erikbern half-life [practitioner] https://erikbern.com/2016/12/05/the-half-life-of-code.html
- Tanya Reilly, Being Glue [practitioner] https://www.noidea.dog/glue ; Invisible labor [independent] https://arxiv.org/abs/2401.06889
- code-maat README (Tornhill warning) [primary] https://github.com/adamtornhill/code-maat
- Percentile-rank ordinality [reference] https://en.wikipedia.org/wiki/Percentile_rank ; Fleming & Wallace via goodhart-design.md [primary]
