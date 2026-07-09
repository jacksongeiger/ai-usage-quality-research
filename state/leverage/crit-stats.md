# Statistical Validity Critique — the Current "AI Leverage" Metric

Sub-investigator report. Angle: statistical construction of the current score only (ratio pathologies, skew + percentile ranking, CI-clean ceiling, winsorization, correlated-percentile averaging, cold-start discontinuities, window power). Date: 2026-07-08.

Conventions: every worked example uses explicitly stated parameters; where parameters are assumed rather than measured from client data, the example is labeled **[inference — illustration with stated assumptions]**. External claims carry URL + tag. Each flaw ends with (a) gaming, (b) unfair penalty, (c) the standard statistical fix.

The metric under critique (from BRIEF.md):

```
score = mean( pctile_within_level( PRs_merged × CI_clean / spend$ ),
              pctile_within_level( Σ min(lines_changed_per_PR, 2000) × CI_clean / spend$ ) )
window = 4 weeks; gates: ≥3 merged PRs AND ≥$5 spend AND ≥3 active days AND ≥14-day window
CI_clean = clean_CI_PRs / CI_checked_PRs, defaulting to 1.0 if CI_checked_PRs < 3
```

---

## FLAW 1 — "Output per AI dollar" is a ratio metric with a denominator-dominated distribution

### 1.1 The ranking inversion is empirically real, not hypothetical

Jellyfish (12k devs / 200 companies, Q1 2026): bottom-20% token spenders merged 11 PRs/quarter at ~$3 (**$0.28/PR ⇒ 3.67 PRs/$**); top-20% spenders merged 23 PRs at ~$1,822 (**$89.32/PR ⇒ 0.0126 PRs/$**). ([Jellyfish, 2026-04-15](https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/) [vendor])

Worked example on those numbers: the top-quintile engineer produces **2.1× the merged PRs** but scores **~290× lower** on PRs-per-dollar. Percentile-ranked, the empirically most productive engineers land in the bottom percentiles of a "leverage" score. The metric is the exact mirror image of the spend-maximizing leaderboards Meta and Amazon deployed and scrapped after employees gamed them with fabricated agent tasks ([Fortune, 2026-07-07](https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/) [independent]) — same Goodhart pressure, opposite sign.

### 1.2 The variance of the ratio is dominated by the denominator

Standard result (delta method; industrial practice at Microsoft's experimentation platform — [Deng, Knoblich & Lu, KDD 2018](https://alexdeng.github.io/public/files/kdd2018-dm.pdf) [primary]):

```
CV²(X/Y) ≈ CV²(X) + CV²(Y) − 2ρ·CV(X)·CV(Y)
```

Cui et al. (n=4,867, three RCTs) report pre-period weekly merged-PR means of 0.13–0.87 with **SDs larger than means** and "a high fraction of zeros" ([Mgmt Sci 2026](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566) [primary]). **[inference — illustration with stated assumptions]**: take a 4-week PR count with mean 2, CV = 0.7 (consistent with Cui), spend CV = 1.0 (conservative for a spend distribution whose quintile means span $3–$1,822), ρ = 0.3. Then CV²(ratio) ≈ 0.49 + 1.00 − 0.42 = 1.07 → **CV ≈ 103%**. The 4-week per-dollar value carries noise on the order of its own magnitude; a single heavy agent session or one bunched PR moves the ratio by tens of percent with zero change in underlying behavior.

### 1.3 Extremal pathology at small spend

The ratio is maximized as spend→0 (extremal Goodhart, [Manheim & Garrabrant 2018](https://arxiv.org/abs/1803.04585) [primary]). The $5 floor is an ad-hoc patch that *relocates* the singularity, not removes it: 3 PRs at $5.01 spend = **0.599 PRs/$**; 30 PRs at $500 = **0.06 PRs/$**. The near-floor engineer outranks a 10×-output colleague by 10×. Also note mean-of-ratios ≠ ratio-of-means: per-person 4-week ratios do not aggregate to any meaningful cohort quantity.

### 1.4 Gaming / penalty / fix

- **(a) Gamed by**: shrinking the denominator — route usage to unmetered/personal accounts or tools off the allowlist (the allowlist is narrow by the brief's own admission); time spend so it lands just after a window boundary; choose easy work; hover just above the $5 floor.
- **(b) Penalizes**: legitimate heavy spenders doing harder work (brief problem #5); agent-platform engineers (all spend, no PR output → score → 0 by construction); anyone exploring/learning new tools (spend precedes output).
- **(c) Standard fix**: never use spend as a divisor. Options in ascending ambition: (i) spend as a *context/adoption axis* displayed beside output; (ii) elasticity-style `log(output+1) − log(spend+s₀)` — bounded, symmetric in errors, does not explode at small spend; (iii) output *conditional on* spend band (residual from a cohort regression). If any ratio survives, compute its uncertainty via the delta method and display it.

---

## FLAW 2 — Percentile-ranking a right-skewed ratio: maximal noise amplification exactly where most people sit

The ratio of a right-skewed count over a right-skewed spend is heavy-right-tailed (brief problem #3 concedes this). Percentile transformation of a heavy-tailed variable has a known property: rank is hypersensitive to small absolute differences in the dense bulk and insensitive to huge differences in the sparse tail.

Worked example **[inference — illustration]**: model the per-dollar value as log-normal, σ_log = 1. Percentile = Φ(z).

- Near the median, d(percentile)/dz = φ(0) ≈ 0.40: a **1-noise-SD fluctuation on the log scale moves the score Φ(1)−Φ(0) = 34 percentile points**. With Flaw 1's CV ≈ 100% (σ_log-noise ≈ √ln2 ≈ 0.83), a mid-cohort engineer's month-to-month score swings ±25–30 points from pure sampling noise.
- At the top (z = 2), the same 1-SD fluctuation moves Φ(3)−Φ(2) = **2.2 points**.
- Meanwhile a genuine 28% improvement near the median (0.88→1.13 on the raw scale) moves only ~10 points — indistinguishable from one month of noise.

So the display amplifies noise for the middle of the cohort (most engineers) and compresses real differences in the tail — backwards on both ends. This also explains brief problem #3's observation that "top scorers often just have marginally better CI rate": selecting the top of a noisy proxy selects the noise (regressional Goodhart).

Two further percentile defects (both violated design axioms, not judgment calls):

1. **Zero-sum + non-stationary**: within-cohort percentiles guarantee half the org is "below average" forever; during an adoption wave (exactly this setting) the cohort distribution shifts monthly, so an engineer's score moves with no change in their behavior. For a self-reflection dashboard that is uninterpretable and demotivating; DORA explicitly warns against exactly this league-table use (see survey/dora.md; [Harvey, DX podcast](https://getdx.com/podcast/masterclass-on-dora-metrics/) [primary-interview]).
2. **Granularity**: percentile resolution is 100/n. The proposed fix for brief problem #6 (function×level cohorts, ~36 cells over 500–5,000 engineers) yields cells of ~15–140 people; a 15-person cell moves in ~7-point jumps and reshuffles whenever one person joins or leaves.

- **(a) Gamed by**: not directly manipulable in self-view, but it rewards matching cohort norms rather than doing good work; a cohort can collectively sandbag.
- **(b) Penalizes**: mid-distribution engineers (their score is mostly noise); minority functions inside a mixed cohort (ML vs frontend at the same level — brief problem #6); everyone during adoption waves.
- **(c) Standard fix**: work on an interval scale (log1p spend/LOC/counts; EB posterior for rates), z-score *inside a model with cohort effects* (hierarchical partial pooling — [Stan case study](https://mc-stan.org/rstanarm/articles/pooling.html) [primary]), convert to percentile or coarse bucket only for final display; prefer self-vs-self trend + fixed research-anchored buckets (DORA/fourkeys pattern: medians + coarse buckets — repos/fourkeys.md) over live peer ranks.

---

## FLAW 3 — CI-clean-rate: a ceiling-effect variable with a manufactured point mass at 1.0

### 3.1 The ceiling

Two independent mechanisms pile mass at exactly 1.0:

1. **Saturation**: AI-assisted authors iterate until CI is green before merge (causal Goodhart — intervening on the proxy without moving quality). Cui et al. found build *volume* +38.4% while build success *rate* was flat-to-negative (−17.4% and significant at Accenture) — compile-looping, not quality ([Cui et al.](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566) [primary]). CHAOSS's acceptance-ratio metric saturates identically (repos/chaoss.md).
2. **The fake-neutral default**: < 3 CI-checked PRs ⇒ CI_clean := 1.0. The brief itself says "a large share of people sit at exactly 1.0."

A variable that is 1.0 for most of the population carries near-zero information, so the "quality adjustment" is a no-op for the majority — for them the two signals reduce to pure output/spend — while acting as a **lottery penalty** on the minority who are off the ceiling.

### 3.2 Worked example: the lottery penalty and the fake-neutral inversion

Binomial, true per-PR clean probability p = 0.9, n = 3 checked PRs:

| Outcome | Probability | CI_clean | Multiplicative score effect |
|---|---|---|---|
| 3/3 clean | 0.729 | 1.00 | none |
| 2/3 | 0.243 | 0.667 | −33% on BOTH signals |
| ≤1/3 | 0.028 | ≤0.333 | −67% or worse |

A genuinely good engineer (p = 0.9) has a **27.1% chance each window of a ≥33% score haircut from a single Bernoulli event** (often a flaky test). Meanwhile an engineer with 2 checked PRs is pinned at 1.0 — *above any honest estimate of anyone's rate*. Fewer data ⇒ better score: the default inverts the evidence ordering. Because CI_clean multiplies **both** signals, averaging the two percentiles does not dilute this shock — it passes through at full strength (see Flaw 5).

### 3.3 Worked example: the repo-unfairness is arithmetic, not anecdotal

A PR is "clean" only if every CI job passes. With per-job flake-free pass probability 0.995 **[inference — illustration]**:

- 5-job repo: 0.995⁵ = **0.975** expected clean rate
- 50-job repo: 0.995⁵⁰ = **0.778** expected clean rate

Identical engineer quality, ~20-point rate gap driven entirely by repo CI configuration (brief problems #1/#7). The off-ceiling minority that CI_clean actually discriminates among is therefore disproportionately composed of engineers in strict/flaky-CI repos — the adjustment penalizes the repo, not the person.

### 3.4 Grain mismatch

CI_clean is computed over *CI-checked* PRs only, but multiplies a numerator counting *all* merged PRs — non-CI'd PRs inherit the quality credit of the checked ones (same mismatched-grain bug class as fourkeys' CFR, whose per-commit/per-deploy mismatch can exceed 1.0 — repos/fourkeys.md).

- **(a) Gamed by**: iterate-to-green before pushing (undetectable, universal); route risky changes through repos/paths with no or light CI (exploits 3.4); submit only trivial PRs; stay under 3 checked PRs to bank the free 1.0.
- **(b) Penalizes**: engineers in heavy/flaky-CI repos; risk-takers on hard changes; the 3–5-checked-PR crowd relative to the <3 crowd (evidence is punished, absence of evidence is rewarded).
- **(c) Standard fix, in two layers**:
  1. *Statistical*: replace the fake 1.0 with an empirical-Bayes Beta-binomial posterior mean, `rate_EB = (clean + α)/(checked + α + β)`, with (α,β) fit per repo-tier or function cohort ([Robinson, Variance Explained](http://varianceexplained.org/r/empirical_bayes_baseball/), mirror https://www.r-bloggers.com/2015/10/understanding-empirical-bayes-estimation-using-baseball-statistics/ [practitioner]); display Wilson intervals, never Wald ([Brown, Cai & DasGupta 2001](http://www-stat.wharton.upenn.edu/~lbrown/Papers/2001a%20Interval%20estimation%20for%20a%20binomial%20proportion%20(with%20T.%20T.%20Cai%20and%20A.%20DasGupta).pdf) [primary]). Under EB, the 2/3-flaky engineer shrinks toward the cohort prior instead of eating a 33% cliff, and nobody gets a free 1.0.
  2. *Honest limit*: shrinkage cannot restore discrimination to a saturated signal — if everyone iterates to green, all posteriors converge on the prior. **CI-clean-rate is statistically dead as a quality multiplier under any transform**; demote it to an anomaly floor and acquire real quality counterweights (churn/revert/review-burden — survey/gitclear.md, survey/independent-evidence.md).

---

## FLAW 4 — Winsorization at a fixed 2000-line constant: an anti-gaming cap that is itself the gaming instruction

### 4.1 Worked example: the cap makes splitting strictly dominant

A 6,000-line generated migration PR:

| Strategy | PR count (signal 1 numerator) | Capped lines (signal 2 numerator) |
|---|---|---|
| 1 PR of 6,000 lines | 1 | min(6000, 2000) = 2,000 |
| 3 PRs of 2,000 lines | **3 (×3)** | **6,000 (×3)** |

Splitting under the published cap **triples both numerators simultaneously**. The cap doesn't just fail to prevent one huge generated PR from dominating — it converts it into three scoring PRs and tells the engineer the exactly optimal PR size (2,000 lines). Because the cap is per-PR but the metric sums over PRs, the effective winsorization depends on PR granularity: the estimator is inconsistent across work styles, and fine-slicers are structurally favored. AI makes padding to just-under-cap free (GitClear: only ~5% of changed lines correspond to meaningful work — [vendor claim, unaudited](https://www.gitclear.com/count_lines_of_code_at_your_own_peril)).

### 4.2 The constant is arbitrary across functions

Standard winsorization practice caps at distribution percentiles (1/99 or 5/95) chosen by sensitivity analysis, not at a fixed absolute value (accounting/finance norm — Leone; SAS caution that cutoff choice materially changes results: https://blogs.sas.com/content/iml/2017/02/08/winsorization-good-bad-and-ugly.html [practitioner]). A 2,000-line cap means something entirely different for a data-eng migration PR than a frontend PR; it also counts deletions the same as additions, scoring Atkinson's famous −2000-line improvement (https://www.folklore.org/Negative_2000_Lines_Of_Code.html [practitioner/primary anecdote]) identically to 2,000 lines of generated padding — and code-simplification work worse than padding once capped.

- **(a) Gamed by**: PR splitting (4.1); verbose/duplicated generation up to the cap; lockfile/codegen/vendored churn (no path exclusion exists in the current design).
- **(b) Penalizes**: functions with legitimately large units of work (migrations, codegen, data); code deleters and simplifiers; senior engineers who solve with less code.
- **(c) Standard fix**: (i) exclude vendored/generated/non-code paths *before* counting (git-of-theseus filetype allowlist, code-maat path exclusion, CHAOSS whitespace/merge-commit zeroing — repos/*.md); (ii) discount deletions asymmetrically only if rewarding them (e.g., additions + 0.2×deletions bucketed, repos/discovered.md) or drop >30-file change-sets outright (code-maat); (iii) cap at **cohort percentiles** (95th/99th of per-PR lines within function) with an OECD/JRC-style sensitivity analysis on the cutoff ([Handbook on Composite Indicators](https://www.oecd.org/content/dam/oecd/en/publications/reports/2008/08/handbook-on-constructing-composite-indicators-methodology-and-user-guide_g1gh9301/9789264043466-en.pdf) [primary/institutional]); (iv) accept that winsorization treats the symptom — LOC is a bad numerator regardless.

---

## FLAW 5 — Averaging two correlated percentiles: pseudo-triangulation that collapses into an inverse-spend leaderboard

### 5.1 Ordinal-scale violation

Percentile ranks are ordinal, not equal-interval (bunched at the median, stretched in the tails); psychometrics forbids averaging them — combination happens on z/interval scales, percentiles are display-only (https://en.wikipedia.org/wiki/Percentile_rank [reference]). And Fleming & Wallace (CACM 1986) proved the arithmetic mean of normalized ratios produces normalization-dependent, inconsistent rankings; the geometric mean is the invariant summary ([ACM DL](https://dl.acm.org/doi/10.1145/5666.5673) [primary]). The current score commits both anti-patterns in one line.

### 5.2 Worked example: the two signals are one signal — and that signal is mostly spend

Write both signals in logs:

```
log s₁ = log PRs + log CI_clean − log spend
log s₂ = log capped_LOC + log CI_clean − log spend
```

They share the CI term and the spend term by construction; only the numerator count differs (and PR count vs LOC are themselves positively correlated). Estimate the spend dispersion from Jellyfish's quintile means ($3 → $1,822): under log-normality, the gap between bottom- and top-quintile means is ≈2.8σ, so σ_log-spend ≈ ln(1822/3)/2.8 ≈ **2.3** **[inference — derivation from vendor-published quintile means, assumes log-normality]**.

With plausible σ_log-PRs = 0.5, σ_log-LOC = 0.9, σ_log-CI ≈ 0 (saturated, Flaw 3), independence of components **[inference — illustration]**:

- Share of Var(log s₁) contributed by the spend term: 5.29/(5.29+0.25) ≈ **95%**
- corr(log s₁, log s₂) = 5.29/√(5.54 × 6.10) ≈ **0.91**

Consequences: (i) averaging two ~0.9-correlated signals adds essentially no information over either alone — it is one signal counted twice, wearing the costume of triangulation; (ii) since ~95% of each signal's variance is −log spend, the composite is, to first order, **a percentile rank of not spending** — the untested mirror image of the failed Meta/Amazon token leaderboards (survey/roi-cost.md), inheriting their Goodhart dynamics with the sign flipped; (iii) because CI_clean multiplies both signals, its noise (Flaw 3.2) passes through the average undiluted — the composite provides zero diversification against exactly its noisiest component.

This is also why the metric fails SPACE's core design rule that combined metrics be deliberately *in tension* ([SPACE, ACM Queue](https://queue.acm.org/detail.cfm?id=3454124) [primary]): two per-dollar activity ratios sharing denominator and multiplier are maximally in agreement, not in tension.

- **(a) Gamed by**: any denominator attack (Flaw 1) games both components at once — the average is exactly as gameable as its most gameable part; compensability lets an engineer max the cheap component (PR count via micro-PRs) and coast.
- **(b) Penalizes**: engineers strong on whichever axis the composite underweights after normalization (which is normalization-dependent, per Fleming & Wallace, i.e., arbitrary); everyone, via uninterpretability — "why is my score 61?" has no stable answer when the weights are implicit in two cohort distributions.
- **(c) Standard fix**: don't combine — show non-compensatory subscores side by side (Austin/Harris-Tayler/SPACE convergence; survey/goodhart-design.md §5.7). If a single number is mandated: combine on log/z scale inside a cohort model, geometric mean for any ratio components, run the OECD/JRC robustness checklist (vary weights/normalization, check rank stability), and display sub-parts.

---

## FLAW 6 — Hard cold-start floors: step-function eligibility on a zero-inflated count

### 6.1 Worked example: the score flickers for a typical engineer

Four AND-ed gates (≥3 merged PRs, ≥$5, ≥3 active days, ≥14 days) make score *existence* a step function of noisy counts. Cui et al.'s distributions (weekly PR means 0.13–0.87, SD > mean, zero-inflated [primary]) put a typical engineer's 4-week count right at the threshold. Model the 4-week count as Poisson(λ = 3.5) **[inference — illustration]**:

- P(N ≤ 2) = e^−3.5(1 + 3.5 + 6.125) ≈ **0.32** — an engineer whose true rate is 3.5 PRs/month has a ~1-in-3 chance of simply having **no score** in any given month (brief problem #4, quantified).
- **Selection bias in their own trend line**: E[N | N ≥ 3] = (3.5 − 0.476)/0.679 ≈ **4.45** — the months that display a score average 4.45 PRs against a true mean of 3.5, a **+27% upward bias**. The engineer's visible history systematically shows their lucky months, then "regresses" whenever they cross the gate — the self-trend the dashboard exists to show is structurally distorted.

### 6.2 The floors interact perversely with the ratio

The $5 spend gate places scored engineers *adjacent to the ratio's singularity* (Flaw 1.3): the score distribution's right tail is populated by near-floor spenders, so the gate designed to prevent degenerate ratios instead concentrates scoring mass right where the ratio is most degenerate.

- **(a) Gamed by**: timing — hold merges to bunch PRs into windows where they clear the gate; keep spend just above $5; conversely, sandbag below gates in bad months so no score is recorded (missing > bad, since missing months vanish from the trend).
- **(b) Penalizes**: part-timers, on-call rotations, leave-takers, big-feature engineers with lumpy cadence; agent-platform engineers permanently (never pass the PR gate — brief hard limit #4); new joiners (14-day + 3-active-day gates).
- **(c) Standard fix**: delete all hard floors. (i) EB shrinkage/partial pooling gives everyone a score always — low-evidence people sit near their cohort prior with a wide displayed interval ("58 ± 21, low confidence — 2 PRs, 9 active days") ([Robinson](http://varianceexplained.org/r/empirical_bayes_baseball/) [practitioner]; [Stan partial pooling](https://mc-stan.org/rstanarm/articles/pooling.html) [primary]); (ii) explicit "insufficient observation" states instead of fake neutrals or silent absence (DevLake's `_is_collected_data` N/A pattern, Kaplan-Meier right-censoring — repos/devlake.md, repos/git-of-theseus.md); (iii) rolling 8–12-week windows on a zero-filled calendar spine (fourkeys) — the cheapest reliability gain available and explicitly permitted by the brief. Note EB's own gaming/penalty: below-average performers can hide at the prior by keeping n low; low-volume *excellent* performers get pulled toward the mean — mitigate by showing raw + shrunk + interval.

---

## FLAW 7 (cross-cutting) — The 4-week window cannot support per-person inference at all

Cui et al. needed n ≈ 5,000 engineers over months for PR effects to be *barely* significant (+26.08%, SE 10.3 — the 95% CI spans roughly +6% to +46%) [primary]. The current design asks a 4-week window to rank *one* engineer. Reliability sketch **[inference — illustration]**: test-retest reliability = σ²_between/(σ²_between + σ²_noise). With within-person sampling noise from Flaw 1 (CV ≈ 100% ⇒ σ_log ≈ 0.83) and a generous true between-person σ_log of 0.5–0.7, reliability ≈ **0.27–0.42** — the majority of month-over-month score movement is sampling noise, before any gaming. A score that mostly measures its own noise fails the brief's "motivating and honest" and "interpretable" requirements on statistical grounds alone: engineers will attribute noise to their behavior (the METR RCT shows exactly how badly self-perception calibrates — devs believed +20%, measured −19%: https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ [independent]).

- **(a) Gamed by**: nothing needed — noise games itself; but window-boundary timing attacks (Flaws 1, 6) are amplified by short windows.
- **(b) Penalizes**: low-volume engineers most (their windows are noisiest); rewards high-volume micro-PR styles with tighter sampling error.
- **(c) Standard fix**: rolling 8–12 weeks (brief-sanctioned), median-of-weekly-buckets on a calendar spine (DevLake pattern), shrinkage toward cohort priors, and displayed uncertainty so the engineer can see when a change is real.

---

## Consolidated gaming / penalty table (metrics and fixes discussed above)

| Element | (a) How gamed | (b) Who unfairly penalized |
|---|---|---|
| PRs per $ | Shrink denominator (unmetered tools, timing); micro-PRs | Heavy spenders on hard work; agent-platform engineers; explorers |
| Capped-LOC per $ | Split at 2,000; pad verbose code; lockfile churn | Deleters/simplifiers; large-unit functions; seniors |
| CI-clean multiplier | Iterate-to-green; avoid CI'd paths; stay <3 checked PRs | Strict/flaky-CI repos; risk-takers; 3–5-PR engineers vs <3 |
| Level-only percentile cohort | Match norms, not value; collective sandbagging | Minority functions; everyone during adoption waves |
| Arithmetic mean of 2 percentiles | Optimize cheapest component; any shared-term attack hits both | Whoever the (arbitrary) implicit weights disfavor |
| Fixed 2,000 winsorization | Sit exactly under the published cap | Migration/codegen/data work |
| Cold-start AND-gates | Bunch merges across boundaries; sandbag bad months into "missing" | Part-timers, on-call, new joiners, lumpy-cadence, non-PR engineers |
| **Fix: EB shrinkage** | Keep n low to hide at prior; slow collective prior poisoning | Low-volume excellent performers pulled to mean (show raw+interval) |
| **Fix: Wilson lower bound** | Volume raises the bound → quantity inflation helps | ALL low-n users by construction — use for ranking, not self-scores |
| **Fix: log-elasticity (log out − log spend)** | Still a per-dollar shape: unmetered-tool shifting persists | Softer but nonzero penalty on legitimate high spend — keep as context, not headline |
| **Fix: geometric-mean composite** | Must move all components (harder) | A legitimate zero in one axis (no PRs) zeroes the person — fatal for agent-platform roles unless axis redefined |
| **Fix: coarse buckets/medians** | Camp just above a bucket boundary | Within-bucket differences invisible (acceptable cost for self-view) |

---

## Verdict

The current metric fails as *statistics* before it fails as *incentive design*: it percentile-ranks (ordinal violation) an arithmetic average (Fleming–Wallace violation) of two ~0.9-correlated (pseudo-triangulation) heavy-tailed ratios whose variance is ~95% denominator (a de facto inverse-spend leaderboard), multiplied by a ceiling-saturated quality proxy with a fake-neutral point mass at 1.0 (evidence inversion), capped at a published gaming boundary, gated by step functions on zero-inflated counts (32% score-flicker, +27% self-trend bias), over a window with an estimated test-retest reliability of ~0.3. Every one of these has a decades-old textbook fix (delta method, log/z-scale combination, geometric means, EB shrinkage, Wilson intervals, cohort-percentile winsorization, rolling windows, hierarchical cohorts) — but two components are beyond statistical repair and must be replaced, not transformed: **spend-as-denominator** (economically and causally wrong per the entire ROI literature) and **CI-clean-rate-as-quality** (saturated; no transform restores discrimination to a dead signal).

## Sources cited in this file

All previously verified by Phase-A agents in survey/*.md and repos/*.md; tags repeated here: Jellyfish tokenmaxxing post [vendor]; Fortune 2026-07-07 [independent]; Cui et al. Mgmt Sci 2026 [primary]; Deng/Knoblich/Lu KDD 2018 [primary]; Manheim & Garrabrant 2018 [primary]; Fleming & Wallace CACM 1986 [primary]; Brown/Cai/DasGupta 2001 [primary]; Robinson EB baseball [practitioner]; Stan partial-pooling case study [primary]; OECD/JRC composite-indicator handbook [primary/institutional]; SPACE ACM Queue [primary]; METR RCT [independent]; GitClear LOC pages [vendor, unaudited]; folklore.org −2000 lines [practitioner/primary anecdote]; SAS winsorization caution [practitioner]; percentile-rank ordinality [reference]; Harvey/DX DORA masterclass [primary-interview]. Internal cross-references: survey/goodhart-design.md, survey/roi-cost.md, survey/copilot-studies.md, survey/gitclear.md, survey/independent-evidence.md, survey/space-devex.md, survey/dora.md; repos/fourkeys.md, repos/devlake.md, repos/git-of-theseus.md, repos/code-maat.md, repos/chaoss.md, repos/discovered.md. All numeric worked examples with assumed parameters are labeled [inference — illustration]; the Jellyfish-derived σ_log-spend ≈ 2.3 assumes log-normality of spend [inference].
