# Metric-Design & Gaming Literature — The Statistical Toolbox Chapter

Sub-investigator report. Area: Goodhart/Campbell, LOC/commit gaming history, McKinsey controversy, composite-score design, percentile/cohort normalization, ratio-metric pathologies, winsorization, log transforms, small-n reliability (empirical Bayes, hierarchical models, Wilson/Beta priors).
Date: 2026-07-08. Verification level noted per source in the Sources section. My own reasoning is labeled **[inference]** throughout.

---

## 1. What it measures & theory

### 1.1 The core laws
- **Goodhart (1975)**: "Any observed statistical regularity will tend to collapse once pressure is placed upon it for control purposes." Popularized by **Strathern (1997)**: "When a measure becomes a target, it ceases to be a good measure." ([Wikipedia/reference], corroborated by Rodamar 2018 [independent]).
- **Campbell (1976)**: "The more any quantitative social indicator is used for social decision-making, the more subject it will be to corruption pressures and the more apt it will be to distort and corrupt the social processes it is intended to monitor." Campbell's version adds the second clause — the *process itself* gets corrupted, not just the number. (Rodamar, Significance 2018.)
- **Manheim & Garrabrant (2018, arXiv:1803.04585)** taxonomize four mechanisms — the most useful analytical frame we found for red-teaming any leverage score:
  1. **Regressional**: proxy = goal + noise; selecting on the proxy selects on the noise (top scorers on the current metric are largely the lucky tail of CI-clean-rate noise — exactly the brief's problem #3).
  2. **Extremal**: proxy–goal relationship breaks in the tails (per-dollar ratio is maximized by near-zero spend, not by great AI use).
  3. **Causal**: intervening on the proxy doesn't move the goal (making CI pass by retry-until-green does not improve code quality).
  4. **Adversarial**: agents deliberately game (splitting PRs under the line cap).
- **Austin (1996), *Measuring and Managing Performance in Organizations*** (award-winning CMU thesis): with *partial* measurement, effort flows from unmeasured to measured dimensions until the org's real goal fails; dysfunction is avoidable only under (rare) "full supervision" where every valued dimension is measured. Austin's prescription when full measurement is impossible: informational (not motivational) measurement plus internal motivation. [academic, verified via publisher + two independent reviews]
- **Surrogation — Harris & Tayler, HBR Sept–Oct 2019**: people *mentally replace the strategy with the metric*. Wells Fargo tracked cross-sales as a proxy for "long-term relationships" → 3.5M fake accounts. Mitigations they give: use **multiple yardsticks** (no single metric = the strategy), **do not tie incentives to the metric**, involve the measured population in metric design. [independent/practitioner]
- **Beck & Orosz (2023)**, effort→output→outcome→impact ladder: measurable things (effort, output) are furthest from value; "You can measure activity, but not without directing that activity *away* from the ends you care about"; "If you choose to create incentives around measures, know that you will never again receive accurate data." Beck's guidelines include: **self-measurement for personal improvement is the legitimate use case**, and "Red Team any incentives." [practitioner, fetched]

**Key implication for us [inference]**: the product is self-view-only, which is precisely Austin's "informational" mode and Beck's sanctioned use. Goodhart pressure scales with *perceived* stakes, not stated ones — the moment engineers suspect the score leaks into reviews, all four Goodhart modes activate. Design must therefore assume adversarial pressure anyway (the brief's "hard to game" requirement is correct), and the org must never let the score become manager-visible.

### 1.2 LOC / commit-count gaming: the historical record
- **Bill Atkinson, Apple Lisa team, 1982** (folklore.org, fetched): management tracked weekly LOC; Atkinson rewrote QuickDraw's region engine to be ~6x faster and **2,000 lines smaller**, filed "-2000" on the form; management stopped asking. Canonical demonstration that LOC is anti-correlated with value at the top of the skill curve. [practitioner/primary anecdote]
- **Dan North, "The Worst Programmer I Know"** (2023, dannorth.net): consultant Tim Mackinnon scored **zero** story points every sprint — because he paired all day and made the whole team faster. "Tim wasn't delivering software; Tim was delivering a team that was delivering software." Individual-output metrics score the highest-leverage person at zero. [practitioner]
- **Commit-count gaming is commodity tooling**: open-source tools exist whose sole purpose is to fabricate commit histories/contribution graphs — `fake-git-history`, `gitfiti`, `fake_commit` (GitHub repos, verified to exist). Faking timestamps and empty commits is trivial; any count-based signal inherits this. [primary artifacts]
- **GitClear** (vendor): claims only ~5% of changed lines correspond to meaningful work; their **Diff Delta** metric discounts moved code, whitespace, duplication, and find/replace churn — i.e., a whole commercial product exists because raw LOC is so gameable. Notably relevant post-AI: generated code inflates LOC essentially free. [vendor]
- **McKinsey controversy (Aug 2023)**: McKinsey's "Yes, you can measure software developer productivity" proposed, i.a., **contribution analysis** (per-person code contribution) and inner/outer-loop time. Rebuttals:
  - **Beck & Orosz** (2-part, Pragmatic Engineer + tidyfirst): 4 of 5 metrics measure effort/output not outcomes; gaming inevitable; measurement of individuals impossible — "too many confounding factors, network effects, variance, observation effects, and feedback loops." [practitioner, fetched]
  - **Dan North** (dannorth.net/blog/mckinsey-review): contribution analysis treats development like "building a wall, where you can measure who lays the most bricks"; "those who contribute *less* code to solve the same problem… are often your highest-value developers"; "attempting to measure the individual contribution of a person is like trying to measure the individual contribution of a piston in an engine." [practitioner, fetched]
  - **GeePaw Hill**: a specific published McKinsey rebuttal by GeePaw Hill could **not be verified** in this session (searched geepawhill.org and web; his general anti-metric writing exists but no McKinsey-specific piece found). **Unverified — do not cite.**

---

## 2. Evidence quality

- Goodhart/Campbell/Austin/surrogation: strong, old, replicated across domains (education testing, healthcare targets, Soviet quotas, Wells Fargo). Mostly theory + case study, not RCT — but the mechanism is not seriously disputed.
- Manheim & Garrabrant: theory/taxonomy paper (AI-alignment community), widely cited; useful as a checklist, not as empirical evidence.
- LOC-gaming: anecdote + artifact evidence (tools exist) + one vendor's telemetry (GitClear, unaudited). Direction unambiguous; magnitudes vendor-claimed.
- Statistical techniques (EB shrinkage, Wilson, hierarchical models, delta method, winsorization): textbook-grade, peer-reviewed (Brown-Cai-DasGupta in Statistical Science; Efron-Morris lineage; KDD paper from Microsoft's experimentation platform). Highest-confidence material in this chapter.
- McKinsey rebuttals: practitioner opinion by extremely credible practitioners; not data. Their *predictions* about gaming are grounded in the Austin/Goodhart literature above.

---

## 3. Known failure modes (per technique in our toolbox)

### 3.1 Percentile / cohort normalization
- **Ordinal-scale violation**: percentile ranks are not equal-interval (cases bunch at the median, spread in tails); **averaging two percentile ranks — exactly what the current metric does — is statistically unjustified**. Psychometrics combines z/T (equal-interval) scores and converts to percentile only for final display. [reference: percentile-rank literature]
- **Zero-sum**: within-cohort percentiles guarantee half the population is "below average" forever, even if everyone improves. For a *motivational self-view* dashboard this is corrosive: your score drops when colleagues adopt AI faster, with no change in your behavior — uninterpretable and demotivating. [inference, grounded in Campbell's "corrupt the process" clause]
- **Small-cohort granularity**: percentile resolution is 100/n. A level×function cohort of 12 people moves in ~8-point jumps; one person joining/leaving reshuffles everyone. Hard cohort splits (the fix for the brief's problem #6) collide with small-n noise — this is a bias–variance tradeoff, and hard splitting is the wrong tool (see §5: partial pooling).
- **Non-stationarity**: during an adoption wave (exactly our setting), the cohort distribution shifts monthly; percentile time series confound self-change with cohort-change.

### 3.2 Ratio metrics ("output per AI dollar")
- **Denominator pathology**: ratio improves when the denominator shrinks — extremal Goodhart. The observed $5 floor is an ad-hoc patch for the singularity at spend→0.
- **Statistical instability**: ratios of noisy quantities are heavy-right-tailed; naive variance is wrong — Microsoft's experimentation platform uses the **delta method** for ratio-metric variance (Deng, Knoblich & Lu, KDD 2018). Mean-of-ratios ≠ ratio-of-means; per-person ratios over 4-week windows are dominated by denominator noise. [academic]
- **Aggregation trap**: **Fleming & Wallace (CACM 1986)**: taking the **arithmetic mean of normalized ratios yields normalization-dependent, inconsistent rankings; the geometric mean is invariant**. The current metric arithmetic-averages two normalized ratio signals — the precise anti-pattern this 40-year-old paper warns about. [academic/primary]
- **Economic wrongness**: efficiency ratios punish investment. Spending more to attempt harder work lowers the score (brief problem #5) — causal Goodhart: minimizing spend ≠ maximizing leverage.

### 3.3 Winsorization / caps
- Standard practice: cap at percentiles (1/99 or 5/95), chosen by **sensitivity analysis**, not a fixed absolute constant (finance/accounting norm; Leone; SAS "good, bad, ugly" post warns cutoff choice materially changes results and can hide genuine signal). The current **fixed 2000-line cap** is top-coding at an arbitrary absolute threshold: it doesn't adapt to function norms (data-eng migration PRs vs frontend PRs) and creates a *known, sharp* gaming boundary.
- Winsorization treats the symptom (outliers) not the disease (LOC is a bad numerator).

### 3.4 Log transforms
- log/log1p compresses right skew of spend/LOC/token counts and makes multiplicative differences additive — appropriate for *display and scoring scale*. Caution (**Feng et al. 2014**): log transform does not reliably produce normality, can *increase* skew, and inference on the log scale answers a different question (geometric-mean comparisons); for modeling, prefer GLMs (Gamma, negative binomial) or rank-based methods over "log then t-test." [academic]

### 3.5 Cold-start floors vs. shrinkage
- Hard eligibility gates (≥3 PRs, ≥$5, ≥3 days) create cliff effects: 2 vs 3 merged PRs flips the score on/off (brief problem #4). The statistics literature solved this class of problem decades ago:
  - **Wald intervals/raw proportions are untrustworthy at small n** (Brown, Cai & DasGupta 2001, Statistical Science: Wald coverage is "chaotic" even at large n; **Wilson** interval recommended). [academic/primary]
  - **Wilson lower bound** (Evan Miller, "How Not To Sort By Average Rating", fetched): rank items by the lower confidence bound of the proportion — "given the ratings I have, what proportion can I be 95% confident of?" Correctly handles 2/2=100% vs 100/101=99%. [practitioner]
  - **Empirical-Bayes / Beta-binomial shrinkage** (D. Robinson, Variance Explained, fetched via R-bloggers mirror): fit Beta(α,β) to the population, then estimate each individual as **(successes + α)/(trials + α + β)**. Worked example: career .400 hitter with 10 AB shrinks to .244; .300 hitter with 1000 AB stays ≈.299. Low-evidence individuals sit near the population mean and move smoothly as evidence accrues — **no floors, no cliffs, no fake neutral 1.0**. [practitioner, canonical]
  - **Hierarchical partial pooling** (Stan/rstanarm "Repeated Binary Trials" case study, Carpenter; Efron–Morris lineage): full multilevel version — per-engineer effects drawn from cohort-level distributions whose variance is *learned*; automatically tunes how much to trust the individual vs the cohort. Solves small-per-person-n AND small-cohort-n simultaneously, and yields per-person **uncertainty intervals** we can display. [primary/technical]

---

## 4. Gameability & who it penalizes — (a) how gamed / (b) who's hurt, per metric

| Metric / technique | (a) How an engineer games it | (b) Who it unfairly penalizes |
|---|---|---|
| **Lines changed (even winsorized)** | Generate verbose code, vendor deps, churn/rewrite, lockfiles; AI makes LOC free. Split a 6k-line PR into 3×2k PRs — beats the cap AND triples PR count | Code deleters/simplifiers (Atkinson), reviewers, mentors, config/infra work, senior engineers who solve with less code (North) |
| **Merged-PR count** | Micro-PRs, typo/README PRs, splitting; commit fabrication is commodity tooling (`fake-git-history`) — PR-count analogues are easy | Big-feature engineers, ML researchers, agent-platform builders (zero PRs by design), pairing "Tims" |
| **CI-clean-rate (head commit)** | Iterate locally/AI-iterate until green then push (causal Goodhart — signal saturates at ~1.0); avoid repos with flaky/heavy CI; only submit trivial PRs | Engineers in 50+-job CI repos, flaky-test repos, risk-takers on hard changes; under-3-PR people frozen at fake 1.0 |
| **Output per AI dollar** | Minimize denominator: use unmetered/personal accounts, shift spend to tools outside the meter, batch spend into a different window; do easy work | Legitimate heavy spenders on hard problems; agent-platform engineers (all spend, no visible output); explorers/learners of new tools |
| **Within-cohort percentile** | Not directly gameable in self-view (can't pick cohort), but incentivizes matching cohort norms rather than doing good work | Minority functions inside a mixed cohort (ML vs frontend at same level); everyone during adoption waves (score moves without behavior change) |
| **Arithmetic composite of subscores** | Optimize the cheapest component and coast (compensability); e.g., max PR count, ignore quality | People strong on the hard-to-move axis; makes the score uninterpretable ("why is mine 61?") |
| **Geometric-mean composite** | Must move all components — harder to game — but a legitimate zero in one axis (no PRs) zeroes the person | Non-PR engineers, unless the output axis is redefined per role |
| **Winsorized values (fixed cap)** | Split work to sit under the known cap; cap is a published gaming boundary | Functions whose legitimate unit-of-work is large (migrations, codegen, data) |
| **EB shrinkage / partial pooling** | If you're below cohort average, keep n low to hide at the prior mean (deliberately push fewer scored PRs); prior-fitting can be poisoned slowly by collective drift | Low-volume *excellent* performers get pulled toward the mean (part-timers, specialists, on-call-heavy folks); mitigations: show raw+shrunk and the interval |
| **Wilson lower bound** | Not very gameable (that's its point) — but volume raises the bound, so quantity inflation helps | ALL low-n users by construction — appropriate for ranking strangers' content, too punitive as a personal score; prefer EB posterior mean + interval for self-view |

**Cross-cutting [inference]**: gaming pressure ∝ perceived stakes. Self-view-only design is the single strongest anti-gaming control available; every rebuttal author (Beck, North, Austin) lands on the same point. Second strongest: multiple non-compensatory subscores (surrogation mitigation) so there is no single number to Goodhart.

---

## 5. Applicability to our constraints (mapped to the brief's HAVE / CANNOT)

Concrete lifts, in priority order:

1. **Replace CI-clean-rate's "neutral 1.0 under 3 PRs" with an EB posterior mean.** Formula: `rate_EB = (clean + α)/(checked + α + β)`, with (α,β) fit by MLE/method-of-moments on the population of engineers — ideally per repo-tier or function cohort, since CI harshness varies (brief problem #1). Uses only the per-PR CI signal we HAVE. Kills the 1.0 pileup and the small-n cliff in one move. **Honest limit**: shrinkage cannot restore discrimination to a saturated signal — if everyone iterates to green, the posterior means all converge near the prior; statistics can't fix a dead proxy (flag as a dead end; a "first-attempt CI pass" event would fix it but is on the CANNOT/NEW list).
2. **Delete all hard cold-start floors; replace with shrinkage + displayed uncertainty.** Everyone always has a score; low-evidence people sit near their cohort prior with a wide interval ("58 ± 21, low confidence — based on 2 PRs / 9 active days"). Optionally lengthen the window to rolling 8–12 weeks (allowed by brief) to raise n — this is the cheapest reliability gain available.
3. **Never average percentile ranks.** If combining signals: transform to roughly-interval scale first (log1p for spend/LOC/tokens, EB posterior for rates), z-score *within a model that includes cohort effects*, combine, and convert to a percentile/level label only for final display.
4. **Break the per-dollar ratio.** Spend is a *context/adoption* axis, not a denominator. If leadership insists on an efficiency read: (i) use `log(output+1) − log(spend+s0)` (elasticity-style, bounded, symmetric in errors) or (ii) show output *conditional on* spend band within cohort (residual from a cohort regression), and combine any remaining ratio components by **geometric mean** (Fleming & Wallace), never arithmetic. Either way, per-dollar must not punish spending more to do harder work.
5. **Cohorts via hierarchical model, not hard splits.** Level×function cells (L3–L8 × 6 functions ≈ 36 cells over 500–5000 engineers) are too small for stable percentiles. Fit a multilevel model (engineer ⊂ function×level, partially pooled) — we HAVE role/level/team. This answers brief problem #6 without the small-cell noise that naive re-cohorting creates.
6. **Winsorize at cohort percentiles (e.g., 95th/99th of per-PR lines within function), not a global 2000-line constant**, and run the OECD-style sensitivity analysis (vary cutoffs, weights, normalization; check rank stability) before shipping any composite. The OECD/JRC Handbook's checklist (normalization → weighting → correlation/compensability → robustness) is directly liftable as our composite-QA procedure.
7. **Multiple subscores, not one number** (Austin/Harris-Tayler/Beck all converge): e.g., adoption/depth, output-with-quality, efficiency-context — presented side by side. A single composite invites surrogation and compensability gaming; multiple yardsticks are the literature's only robust mitigation. If ONE number is demanded for display, make it explicitly a labeled blend with visible sub-parts (interpretability requirement).
8. **Red-team ritual** (Beck: "Red Team any incentives"): before shipping any formula change, enumerate Manheim–Garrabrant's four modes against it. This file's §4 table is the seed.

**Dead ends to state plainly**: (i) no statistical transform rescues CI-clean-rate's saturation; (ii) individual *productivity* measurement per se is judged infeasible-and-harmful by the strongest practitioner literature — the defensible product is a self-improvement instrument with uncertainty made visible, not a precise personal productivity number; (iii) count-based outputs (PRs, LOC) remain gameable at commodity-tool level no matter the normalization — quality/durability signals must carry weight the counts cannot.

---

## 6. Sources (annotated, tagged)

1. Goodhart's law (Goodhart 1975; Strathern 1997 phrasing) — https://en.wikipedia.org/wiki/Goodhart%27s_law — [reference/independent]. Verified quotes; also in repo SOURCES.md [30].
2. Rodamar, "There ought to be a law! Campbell versus Goodhart," *Significance* (2018) — https://rss.onlinelibrary.wiley.com/doi/full/10.1111/j.1740-9713.2018.01205.x — [independent/academic]. Campbell vs Goodhart formulations compared. (Read via search snippet + abstract; full text not fetched.)
3. Manheim & Garrabrant, "Categorizing Variants of Goodhart's Law" (2018) — https://arxiv.org/abs/1803.04585 — [primary/academic]. Four-mode taxonomy (regressional/extremal/causal/adversarial); verified via arXiv abstract + MIRI announcement.
4. McKinsey, "Yes, you can measure software developer productivity" (Aug 2023) — https://www.mckinsey.com/industries/technology-media-and-telecommunications/our-insights/yes-you-can-measure-software-developer-productivity — [vendor; the artifact being critiqued]. Also SOURCES.md [14].
5. Beck & Orosz, "Measuring developer productivity? A response to McKinsey," Parts 1–2 (2023) — https://newsletter.pragmaticengineer.com/p/measuring-developer-productivity ; https://newsletter.pragmaticengineer.com/p/measuring-developer-productivity-part-2 ; Part 2 canonical: https://newsletter.kentbeck.com/p/measuring-developer-productivity-440 — [practitioner]. **Fetched Part 2 in full**; quotes verified ("never again receive accurate data"; "not possible… too many confounding factors"; self-measurement guideline; Red-Team guideline).
6. Dan North, "McKinsey Developer Productivity Review" (2023) — https://dannorth.net/blog/mckinsey-review/ — [practitioner]. **Fetched**; piston/engine and bricks quotes verified.
7. Dan North, "The Worst Programmer I Know" (2023) — https://dannorth.net/the-worst-programmer/ — [practitioner]. Tim Mackinnon zero-story-points story; verified via search + multiple mirrors.
8. "-2000 Lines Of Code," folklore.org (Andy Hertzfeld's site, story ca. 1982) — https://www.folklore.org/Negative_2000_Lines_Of_Code.html — [practitioner/primary anecdote]. **Fetched.**
9. Austin, *Measuring and Managing Performance in Organizations* (Dorset House, 1996) — https://books.google.com/books/about/Measuring_and_Managing_Performance_in_Or.html?id=hVMUAAAAQBAJ — [academic/primary book]. Measurement dysfunction; partial vs full supervision. Verified via publisher metadata + independent reviews (codearcana.com, nehrlich.com); book text not fetched.
10. Harris & Tayler, "Don't Let Metrics Undermine Your Business," *HBR* Sept–Oct 2019 — https://hbr.org/2019/09/dont-let-metrics-undermine-your-business — [independent/practitioner]. Surrogation; Wells Fargo; multiple-yardsticks mitigation. (Search-snippet verified; paywalled.)
11. Fleming & Wallace, "How not to lie with statistics: the correct way to summarize benchmark results," *CACM* 29(3), 1986 — https://dl.acm.org/doi/10.1145/5666.5673 — [primary/academic]. Arithmetic mean of normalized ratios inconsistent; geometric mean normalization-invariant. Verified via ACM DL + secondary writeups.
12. OECD/EC-JRC, *Handbook on Constructing Composite Indicators: Methodology and User Guide* (2008) — https://www.oecd.org/content/dam/oecd/en/publications/reports/2008/08/handbook-on-constructing-composite-indicators-methodology-and-user-guide_g1gh9301/9789264043466-en.pdf — [primary/institutional]. Normalization/weighting/compensability/robustness checklist.
13. Evan Miller, "How Not To Sort By Average Rating" (2009) — https://www.evanmiller.org/how-not-to-sort-by-average-rating.html — [practitioner]. **Fetched.** Wilson lower bound; failure of diff-score and raw average.
14. Brown, Cai & DasGupta, "Interval Estimation for a Binomial Proportion," *Statistical Science* 16(2), 2001 — http://www-stat.wharton.upenn.edu/~lbrown/Papers/2001a%20Interval%20estimation%20for%20a%20binomial%20proportion%20(with%20T.%20T.%20Cai%20and%20A.%20DasGupta).pdf — [primary/academic]. Wald chaotic coverage; Wilson recommended.
15. D. Robinson, "Understanding empirical Bayes estimation (using baseball statistics)" (2015) — http://varianceexplained.org/r/empirical_bayes_baseball/ (site cert broken 2026-07-08; **fetched via mirror** https://www.r-bloggers.com/2015/10/understanding-empirical-bayes-estimation-using-baseball-statistics/) — [practitioner]. Beta prior fit (α≈78.7, β≈303.5 in example); posterior mean formula; worked shrinkage numbers.
16. Stan team / Bob Carpenter, "Hierarchical Partial Pooling for Repeated Binary Trials" — https://mc-stan.org/rstanarm/articles/pooling.html (also PyMC port https://www.pymc.io/projects/examples/en/latest/case_studies/hierarchical_partial_pooling.html) — [primary/technical]. Complete/no/partial pooling on Efron–Morris data.
17. Deng, Knoblich & Lu, "Applying the Delta Method in Metric Analytics: A Practical Guide with Novel Ideas," KDD 2018 — https://alexdeng.github.io/public/files/kdd2018-dm.pdf — [primary/academic]. Ratio-metric variance; industrial practice at Microsoft experimentation.
18. Feng et al., "Log-transformation and its implications for data analysis," *Shanghai Archives of Psychiatry* 26(2), 2014 — https://pmc.ncbi.nlm.nih.gov/articles/PMC4120293/ — [academic]. Log transform ≠ normality; can worsen skew; prefer distribution-robust methods for inference.
19. Winsorization practice: Wikipedia — https://en.wikipedia.org/wiki/Winsorizing [reference]; Leone, "Influential Observations and Inference in Accounting Research" (slides) — https://business.uoregon.edu/sites/default/files/media/andrew-leone-accounting-research-wkshop-2013-11.pdf [academic]; Wicherts/SAS caution: "Winsorization: the good, the bad, and the ugly" — https://blogs.sas.com/content/iml/2017/02/08/winsorization-good-bad-and-ugly.html [practitioner].
20. GitClear, "Count lines of code at your own peril" — https://www.gitclear.com/count_lines_of_code_at_your_own_peril ; Diff Delta calculation — https://www.gitclear.com/help/technical/diff_delta_calculation — [vendor]. ~5% of LOC = meaningful work (vendor claim, unaudited).
21. Commit-fabrication artifacts: https://github.com/artiebits/fake-git-history ; https://github.com/gelstudios/gitfiti — [primary artifacts/practitioner]. Existence verified.
22. Percentile ranks are ordinal / not averageable — https://en.wikipedia.org/wiki/Percentile_rank ; https://www.cogn-iq.org/learn/theory/percentiles/ — [reference]. Psychometric convention: combine on z/T scale, display as percentile.

**Explicit non-findings**: GeePaw Hill McKinsey-specific rebuttal — unverified, not found; do not cite. Microsoft stack-ranking harm reporting (Vanity Fair 2012) — relevant to percentile-cohort pitfalls but not verified this session; treat as unverified unless another agent confirms.
