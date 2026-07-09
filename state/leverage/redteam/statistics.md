# RED TEAM — Statistics Attack on RECOMMENDED_DESIGN.md (ALP, Anchored Hybrid)

Hostile-statistician pass, 2026-07-08. Target: the NEW design only (the old metric was already executed in `crit-stats.md`; nothing here re-litigates it). Angle of attack per mandate: distributional assumptions, small-n behavior, normalization artifacts, correlated components, ceiling/floor effects, the 2-week lag.

**Method note.** Every numeric counterexample below was computed (not eyeballed) with explicitly stated parameters. Where parameters are assumed rather than measured from client data, the example is labeled **[inference — illustration with stated assumptions]**. Assumed activity distributions are anchored to Cui et al.'s published pre-period moments (weekly merged-PR means 0.13–0.87, SD > mean, high zero fraction — https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 [independent-adjacent]). Wilson-interval mechanics per Brown/Cai/DasGupta [primary]. Tie-counting for Theil–Sen is pure arithmetic — derived, not cited. Each attack ends with (a) how an engineer games the attacked signal and (b) who it unfairly penalizes, then a patch if one exists.

Verdict up front: ALP genuinely kills the old metric's four worst components (spend denominator, live percentiles, fake-neutral CI default, hard AND-gates). But its **headline machinery (Momentum + Q-gate) has roughly the same test-retest reliability as the 0.3-reliability score it mocks**, its quality "anomaly floor" is a mislabeled below-median test, and its self-audit loop is statistically guaranteed to demote working thresholds in small cohorts. Four attacks are critical; all four have patches.

---

## ATTACK 1 — CRITICAL — Momentum has ≈zero power at n=12 weekly points; the headline is either sign-noise or never fires

**The dilemma.** The spec is ambiguous about whether headline transitions use point estimates (`M_prac > 0 AND M_out ≥ 0`) or the bootstrap CI ("CI spanning buckets → no detectable trend yet"). Both readings fail:

- **Point-estimate reading**: slope signs at this SNR are near coin flips (see below) → the headline is noise wearing a sentence.
- **CI reading**: over weeks x=1..12, S_x=√143≈11.96. With weekly y-noise σ=0.8 (conservative: log1p of a zero-inflated Poisson PR count alone has SD≈0.48; the LOC term adds ≈1.0; see Attack 8), SE(slope)=0.067/wk. A 90% interval spans ±1.43 log-units per quarter — output must grow **4.2× per quarter** (10.2× at σ=1.3) before the CI excludes zero. **[inference — illustration with stated assumptions]** No honest behavior change is detectable; ~everyone reads "no detectable trend yet" forever, and the design's admitted motivation risk (§7.6) becomes near-certainty.

**The threshold is decorative.** The "small slope" boundary of 10%/quarter = 0.0073/wk is **0.11 × SE(slope)** — eleven times smaller than the noise. It classifies nothing; noise does.

**State-machine order bug.** As written, `M_prac > 0 AND M_out ≥ 0 → Improving` is evaluated BEFORE `both small → Steady`. An engineer with M_prac=+0.0001 and M_out=+0.0001 (pure noise, both far inside the "small" band) reads **"Improving"**. If the intent was to map small slopes to 0 first, the spec must say so; the pseudocode does the opposite.

**Block bootstrap at n=12 is theater.** Moving-block bootstrap with block=3 on 12 points has 10 blocks and resamples 4; small-sample undercoverage of MBB intervals at n≈12 is a known property (textbook small-n behavior — stated as inference, no URL). Worse, for sparse engineers (Attack 2) the pairwise-slope distribution is a point mass at 0 → the bootstrap distribution degenerates to {0} and the "interval" is [0,0] — a maximally confident interval from minimally informative data.

- **(a) Gamed by**: under the point-estimate reading, nothing needed — noise hands out "Improving" ~25% of quarters; a monotone practice-ramp (Attack 7) makes it deterministic.
- **(b) Penalizes**: under the CI reading, everyone (no achievable improvement is detectable); lumpy-cadence and low-volume engineers most (their σ is larger).
- **Patch**: (i) resolve the ambiguity: slopes are set to 0 iff |slope| < max(10%/quarter, 1×SE) BEFORE the state machine runs; (ii) lengthen the Momentum series to 26 weeks (window is brief-sanctioned up to 12 for the score, but the trend estimator may consume more history than the display window); (iii) replace weekly buckets with biweekly (halves noise per bucket at n=13 over 26w); (iv) report P(slope>0) from the bootstrap as a graded statement ("more likely rising than not, 68%") instead of a 3-state classifier that needs power it doesn't have.

---

## ATTACK 2 — CRITICAL — Exact-zero Theil–Sen ties + cohort detrending = "Drifting" verdicts for unchanged low-output engineers; detrending self-defeats in sparse cohorts

**Tie arithmetic (exact, not simulated).** If an engineer has nonzero output in k of 12 weeks, the zero-week pairs contribute C(12−k,2) zero slopes of 66 pairs. For k≤3, that is ≥36/66 = 54.5% ≥ half → **the Theil–Sen slope is exactly 0 with probability 1**. For a Cui-calibrated engineer at λ=0.3 PRs/wk (P(zero week)=0.74), P(≥9 zero weeks of 12) = **62%** **[inference — illustration]**. So for a large slice of the org, M_out's raw slope is a deterministic 0 — not an estimate, a tie artifact.

**Now subtract D(c).** During an adoption wave (exactly when this ships), a dense cohort's median slope is positive — say D(c)=+0.015/wk (+21%/quarter drift; plausible under a mandate ramp **[inference]**). The sparse engineer's detrended M_out = 0 − 0.015 = −0.015, which exceeds the 0.0073 "small" boundary → not small, not ≥0 → falls through every branch to **"Drifting — practice or output trending down"** — for an engineer whose output is *structurally constant* and whose practice may be improving. The design's own fairness section never notices that its detrending term manufactures decline for the tied-at-zero majority.

**The self-defeat.** If instead >50% of the cohort is itself sparse, the cohort median slope is pinned to exactly 0 by the same tie arithmetic → D(c)=0 and the detrending **removes no drift at all**. So the mechanism fails in one of two ways depending on cohort density: dense cohort → fabricated "Drifting" for the sparse minority; sparse cohort → no detrending (the "cannot distinguish a better engineer from a later calendar date" hole it was built to close reopens).

- **(a) Gamed by**: a sparse engineer who understands the tie mechanics can guarantee M_out ≥ 0 by ensuring the second half-window has at least as many nonzero weeks as the first (trivial merge-timing).
- **(b) Penalizes**: juniors, part-timers, big-feature engineers, SREs — anyone with ≥9 zero-output weeks — precisely the populations the fairness section promises to protect.
- **Patch**: (i) never compare a tie-degenerate slope to a continuous D(c): if >50% of an engineer's pairwise slopes are 0, emit "output too sparse for trend" (an unmeasured state, consistent with the design's own philosophy) instead of detrending; (ii) detrend on the aggregated cohort *series* (median weekly y across the cohort, then slope) rather than the median of individual slopes — the cohort series is dense even when individuals are sparse, so D(c) is estimable without the pin-to-zero pathology; (iii) apply the small-slope threshold AFTER detrending symmetrically to both terms.

---

## ATTACK 3 — CRITICAL — The "anomaly floor" is a below-median test: it flags a p=0.9 engineer 27% of windows at n=3, and asymptotically flags half of all high-volume engineers

The v0 flag is `Wilson95_upper(clean, checked) < median(rate_EB | repo tier)`. **The median is not an anomaly boundary.** This is a one-sided test of H0: rate ≥ median — and half the cohort is truly below the median.

**Small-n lottery (the old Flaw 3.2, reincarnated).** Repo-tier median rate_EB = 0.95 (mild for a saturated distribution; the design itself says rates trend to ~1.0). Engineer with true clean rate p=0.9, n=3 checked PRs **[inference — illustration]**:

| Outcome | Prob | Wilson95 upper | Flag? |
|---|---|---|---|
| 3/3 | 0.729 | 1.000 | no |
| 2/3 | 0.243 | **0.9385** | **yes** |
| 1/3 | 0.027 | 0.7923 | yes |
| 0/3 | 0.001 | 0.5615 | yes |

P(flag) = **0.271 per window** — and because `Q_state = attention` short-circuits the headline, this is a 27% chance of **"Mixed — check quality flags"** for a genuinely good engineer, from a single flaky test. The design abolished the fake-1.0 default and then rebuilt the identical lottery one layer up. (It fires whenever the repo-tier median exceeds 0.9385 — i.e., in every saturated repo tier, which is the normal case by the design's own account.)

**Large-n creep (worse).** True rate 0.90 vs median 0.95: at n=40 (36/40), Wilson upper = 0.960 → no flag; at n=100 (90/100), Wilson upper = 0.945 → **flagged with certainty in expectation**. As n→∞ the test consistently detects "below median", so among high-volume engineers the flag rate converges to ~the fraction below the median ≈ **50%**. Two engineers with identical true rates, one shipping 3× the volume: only the prolific one gets flagged — evidence punishes, again, in the design that claimed to restore evidence ordering.

**Multiplicity.** v1 adds flag_revert and flag_churn. Three tests at ~5% each ⇒ ~14% family false-flag per window even with well-calibrated references; the 12-week window slides weekly, giving ~13 correlated tests/quarter (effective ~2–3 independent looks) ⇒ a clean engineer's chance of ≥1 "Mixed" headline week per quarter ≈ 1−(1−0.14)^2.5 ≈ **31%** **[inference — illustration]**.

- **(a) Gamed by**: keep PRs off CI paths (design has a canary, but the canary is org-side and doesn't unflag anyone); keep checked-PR volume low — n=3–5 flags only via the lottery, n=100 flags deterministically, so low volume strictly dominates.
- **(b) Penalizes**: high-volume engineers with slightly-below-median (i.e., normal) rates; small-n engineers via the flake lottery; heavy/flaky-CI repo residents whenever the repo-tier prior is mis-tiered (tiers don't exist until acquisition #6).
- **Patch**: (i) reference = a genuine outlier boundary — P10 of the repo-tier EB rate distribution, or median − δ with δ ≥ 0.05 declared as the minimum practically-significant deficit (a non-inferiority margin, not a point null); (ii) minimum n=10 checked PRs before the flag is computable ("quality: not yet computable" below that — consistent with the design's own no-fake-neutral rule); (iii) test once per frozen window boundary, not on every weekly slide, or control the family-wise rate across flags × recomputes; (iv) the headline gate should require the flag to persist 2 consecutive *independent* windows, not 2 overlapping ones (see Attack 6 for why overlap makes persistence free).

---

## ATTACK 4 — CRITICAL — The validation loop's demotion rule is an underpowered test whose default outcome triggers demotion; it will dismantle working rungs in small cohorts with near-certainty

Rule (§1.6): "any threshold whose sustained transitions show **null-or-negative** within-person deltas in a cohort for 2 consecutive quarters gets revised."

**Power arithmetic [inference — illustration with stated assumptions].** Weekly PR SD = 1.0 (Cui: SD > mean at mean ≈ 1). A 12-week person-period mean has SE = 0.289; a within-person before/after difference (independence assumed, conservative) has SE = 0.408/person. Cohort mean of n transitions:

| n transitions/quarter | SE of mean delta | Power vs a true +0.1 PR/wk (+10%) effect | P(2 consecutive nulls despite real effect) |
|---|---|---|---|
| 10 | 0.129 | **0.19** | **0.65** |
| 20 | 0.091 | 0.29 | 0.50 |
| 50 | 0.058 | 0.53 | 0.22 |

The design's own headline evidence example is "n=214 transitions" **org-wide**; per function×level per quarter, n=5–30 is the realistic regime. There, a *genuinely working* practice rung shows "null" ~70–80% of quarters, and the demotion rule fires on a working rung with ~50–65% probability per year per cohort. The "only self-auditing mechanism in any of the five designs" is a machine for demoting true thresholds in exactly the small cohorts (L8 SRE) where the design already admits weakness.

**Logical bug on top.** The design's advertised success mode is "~15% smaller PRs at **unchanged throughput**" — i.e., a **null** throughput delta is the *success* criterion, yet "null-or-negative deltas" is the *demotion* criterion. As written, a rung delivering exactly the promised benefit satisfies the demotion condition on the throughput outcome. The spec never says whether deltas are evaluated jointly or any-one-null-triggers.

- **(a) Gamed by**: not engineer-gameable, but org-gameable — whoever wants the program dead only needs to point at the (statistically guaranteed) null readouts.
- **(b) Penalizes**: small cohorts, and eventually everyone — a self-dismantling quality mechanism converges to no mechanism.
- **Patch**: (i) demote only on **evidence of absence**: the CI must *exclude* the pre-registered minimum effect of interest (equivalence/non-inferiority logic), never on failure-to-reject; (ii) minimum pooled n before the rule is live (pool cohorts hierarchically; the loop is the one place partial pooling costs nothing because it's quarterly and offline); (iii) define the joint success criterion per rung in advance (e.g., "PR-size delta negative AND throughput delta CI excludes < −10%"); (iv) publish the loop's own power alongside its verdicts ("this cohort's loop can only detect effects > 38%/quarter — verdicts here are uninformative").

---

## ATTACK 5 — MAJOR — Frozen small-cell quantiles lock sampling noise in for a year, then non-stationary drift converts bands into a ceiling followed by a synchronized mass demotion

**Noise lock-in.** f_cutpoints are prior-year quintiles per function×level cell with fallback at n<20 — so cells of n=20–40 exist by construction. SE of the 20th-percentile estimate at n=20 (Normal(0.5, 0.15) illustration **[inference]**): SE(q20) ≈ 0.048 while the adjacent-band width is 0.088 — **cutpoint noise is 54% of a band width**, and freezing publishes that noise as truth for a year. Two identical cells get materially different band boundaries by luck of the draw; "frozen" removes zero-sum coupling but not estimation error, and adds a 12-month correction latency.

**Drift ceiling + refresh cliff.** The whole premise is a fast-moving adoption wave, so this year's F distribution is not last year's. If cohort F drifts from mean 0.50 to 0.65 (SD 0.15) over the year **[inference — illustration]**: share above the frozen top-quintile cutpoint (0.626) reaches **56%** — a "quintile" band holding half the org discriminates nothing (ceiling effect). At the annual refresh the cutpoint jumps to 0.776 and **~36% of the org drops out of the top band overnight with zero behavior change** — the exact "your display moved because of other people" victim channel the design claims to have deleted (§0, graft 1), merely time-shifted and synchronized into one demoralizing event. Same mechanism hits the frozen P95 weekly-LOC caps (AI-era LOC growth means the cap binds progressively harder → Momentum's LOC term is increasingly right-censored → slopes attenuate toward 0 → "Steady" bias in year 2; direction consistent with GitClear's reported AI-era code-volume growth, https://gitclear-public.s3.us-west-2.amazonaws.com/GitClear-AI-Copilot-Code-Quality-2025.pdf [vendor]) and the repo P75 in_band bounds.

- **(a) Gamed by**: learn the frozen boundary (it's published) and camp on it all year — a static target is strictly easier to game than a moving one.
- **(b) Penalizes**: engineers in small cells (noisy boundaries); everyone in drift years (ceiling, then cliff); late-adopting cohorts get flattered, early cohorts get squeezed.
- **Patch**: (i) refresh cutpoints **quarterly with a blend** (e.g., 0.75·old + 0.25·new) — kills both the year-long noise lock-in and the single-day cliff while keeping movements slow and non-zero-sum within any quarter; (ii) for cells n<50, estimate cutpoints with hierarchical shrinkage toward the function level rather than the n<20 hard fallback; (iii) show band + distance-to-boundary so a refresh shift is legible; (iv) publish refresh dates and pre-announce expected band migration ("~15% of backend L4s will shift down one band at the January refresh — this reflects org-wide improvement, not your behavior").

---

## ATTACK 6 — MAJOR — The Practice ladder misclassifies at the boundaries, the hysteresis is powerless against 11/12-week window overlap, and C1's construct inverts on a clean counterexample

**Construct inversion.** Engineer A uses AI meaningfully exactly 1 day/week, all 12 weeks (perfect consistency). No week has ≥2 meaningful days; every trailing-4-week sum is 4 < 6 → C1 = 0 → **Stage 0 "EXPLORING"**. Engineer B uses AI 2 days/week in 6 of 12 active weeks and nothing otherwise → C1 = 0.5 → **Stage 2 "REGULAR"**. The component named CONSISTENCY ranks the perfectly consistent light user two rungs below the lumpier user. The binary ≥2-days cliff also means the difference between 1 and 2 sessions/week is the difference between Stage 0 and Stage 2 — a one-day cliff on a count variable.

**Boundary coin flip.** C1 at AW=12 is a Binomial share with SD up to √(0.25/12)=0.144, against stage boundaries spaced 0.25 apart. An engineer with true weekly-consistency propensity q=0.7 (a real Stage-2/3 borderline) has P(C1 ≥ 0.75) = **0.493** — a coin flip between "REGULAR" and "INTEGRATED (the published target)" every recompute **[inference — illustration]**.

**Hysteresis illusion.** Consecutive weekly recomputes share 11 of 12 weeks, so consecutive C1 values are ~0.92-correlated. If a boundary crossing appears this week, it survives next week's recompute unless the single exchanged week reverses it — so "2 consecutive recomputes" confirms noise the large majority of the time. It filters one-week data glitches, nothing else; the design's anti-flicker claim rests on it doing much more.

**Denominator manipulability.** C1 = consistent weeks / active weeks, and active_week is triggered by ANY session or PR. Concentrating identical total usage into fewer weeks raises C1 (consistency rewards lumping — backwards); conversely one stray phone-chat session in an off week adds a failing week to the denominator.

- **(a) Gamed by**: 2 scripted τ_c-clearing days/week (see Attack 7); avoid any single-session days in low weeks to keep the denominator clean.
- **(b) Penalizes**: daily-but-light users (Stage 0 despite ideal habit); vacationers with one stray session; anyone near a boundary (half their stage changes are noise).
- **Patch**: (i) score C1 on *meaningful days per active week* as a graded quantity (e.g., mean min(days,3)/3) instead of a binary ≥2-day week indicator — removes both the cliff and the inversion; (ii) hysteresis must demand persistence across **non-overlapping** evidence: require the new stage to hold on the 4 newest weeks alone, or delay confirmation 4 weeks, not 1; (iii) publish P(stage|data) or a boundary-distance cue so borderline engineers see "you are on the 2/3 boundary" rather than stochastic promotions/demotions.

---

## ATTACK 7 — MAJOR — τ_c has a grain error (per-SESSION P10 used as a per-DAY floor), making the named anti-theater defense a no-op and the "multi-front" headline spoof single-front

τ_c is defined as "P10 of nonzero per-**session** token distribution per class" but meaningful() tests `tokens(u,d,c) ≥ τ_c` — a **daily total** against a **session** quantile. By construction ~90% of single sessions clear P10 alone; any day with one median session clears it severalfold. With heavy-tailed session tokens (30× same-task variance per the design's own cite, https://arxiv.org/abs/2604.22750 [independent]), P10 of sessions is minuscule relative to daily totals. The floor filters essentially nothing **[inference from the spec's own definitions]**.

**Consequence — the headline spoof is cheaper than advertised.** §4.1 claims the headline needs "simultaneous practice-trend theater AND (output trend ≥ 0)... multi-front cost." But: (1) practice theater = a cron job clearing a near-zero τ_c on a ramp (2→3→4 days/wk over 12 weeks) → monotone meaningful-day series → M_prac > 0 with a tie-free positive Theil–Sen slope, the ONE configuration where n=12 significance is actually reachable; (2) M_out ≥ 0 is free — in a sparse cohort D(c)=0 and the engineer's own tied slope is 0 (Attack 2), so 0 ≥ 0 passes; (3) Q clear is the default. **"Improving" for the cost of a crontab.** The zero-variance canary (#3) is org-side and per-person-invisible by design, so it deters nothing at the individual margin.

- **(a) Gamed by**: as above; also splitting one real session into several tiny ones never hurts and padding never helps — the only behavior the floor blocks is the one nobody does (literally-zero-token sessions).
- **(b) Penalizes**: nobody directly (a no-op floor penalizes no one) — the harm is misplaced institutional confidence in a defense that isn't there.
- **Patch**: (i) fix the grain: τ_c := P10 of the nonzero per-**day** token distribution per class (same intent, correct units); (ii) require ≥2 sessions OR co-occurrence with any same-day PR/commit event for a meaningful day — raises scripted cost above zero without penalizing terse prompters; (iii) honesty option: keep the low floor but delete the claim that τ_c is an anti-theater defense; move the defense burden explicitly to the canaries and say the headline is spoofable at low cost (the design half-admits this in §4.1's residual, then contradicts it with "multi-front cost is the defense").

---

## ATTACK 8 — MAJOR — y = log1p(PRs) + log1p(LOC): LOC-dominated, level-dependent, and built on an undefined `lines_changed`

**Correlated double-count, dominated by the worse term.** PRs and LOC are strongly positively correlated; summing their logs is one volume signal counted twice, with the noisier, more inflatable term dominating: log1p(weekly LOC) ranges ~4–8 with week-to-week SD ≈ 1.0 vs log1p(PRs) SD ≈ 0.48 **[inference — illustration]**. Var(y) ≈ 0.23 + 1.0 + 2ρ(0.48)(1.0) ≈ 1.7 at ρ=0.5 → σ_y ≈ 1.3, which is the σ that pushes Momentum's detectable effect to 10× per quarter (Attack 1). M_out is, to first order, **a LOC trend** — the object the design's own §2 calls "the most inflatable in the literature" is smuggled back in as the headline's output leg.

**Level dependence (fairness).** log1p compresses proportional gains at low levels: doubling 0.2→0.4 PRs/wk moves y by 0.154 (slope 0.013/wk over 12w — undetectable); doubling 5→10 PRs/wk moves it by 0.61 (slope 0.05/wk). The SAME relative improvement is ~4× more visible for the high-volume engineer. Momentum structurally cannot show improvement for low-volume engineers — compounding Attack 2's tie problem for the same population.

**`lines_changed` is undefined, and both readings are exploitable.** If additions+deletions: delete-and-restore churn cycles inflate y on a ramp (revert zeroing arrives only in v1 and only as flags, which never touch M); if additions-only: the org's best refactorer — replacing 2000-line weeks with 200-line cleanup weeks — posts Δy ≈ log(201/2001) ≈ −2.3, a steep negative slope → **"Drifting"** for the person doing the most valuable work in the codebase (the design cites Atkinson's −2000 lines approvingly [practitioner], then builds a headline that would fire him).

- **(a) Gamed by**: ramped verbose generation / lockfile churn (no path exclusion until acquisition #2); churn cycles under the additions+deletions reading.
- **(b) Penalizes**: refactorers/simplifiers (additions-only reading); low-volume engineers (level dependence); anyone whose repo has legitimately spiky LOC.
- **Patch**: (i) define lines_changed now, in the spec; (ii) drop the LOC term from y at launch — `y = log1p(PRs)` alone is less informative but not adversarially owned; reintroduce LOC only after path exclusion (acquisition #2) lands, at reduced weight, e.g., y = log1p(PRs) + 0.5·log1p(LOC_filtered); (iii) for level dependence, compute the trend on log(1 + count/baseline_e) (self-normalized growth) so equal relative gains produce equal slopes.

---

## ATTACK 9 — MAJOR — EB point-estimate banding builds a volume ceiling/floor into Flow

F = (in_band + α_f)/(PRs + α_f + β_f) is shrunk toward the function prior, then banded against frozen quintile cutpoints of the **point estimates**. Shrinkage strength depends on n, so band assignment is confounded with volume. With prior mean 0.6, strength α+β=10 **[inference — illustration]**: an engineer with 4 PRs, **100% in-band**, gets F=0.714; an engineer with 40 PRs, 100% in-band, gets F=0.920. Identical (perfect) behavior, likely different bands; if the top-band cutpoint is ~0.8, the low-volume engineer **cannot reach the top band at any level of discipline** — a hard ceiling by n. Symmetrically, a low-volume sloppy engineer can't fall to the bottom band (floor). The band axis quietly becomes a PR-volume axis at both ends — in the facet whose whole point was that "more PRs never raise it." The mixture of n-dependent shrinkage also makes the frozen quintile cutpoints themselves volume-composition-dependent: a cell's cutpoints shift if its volume mix shifts, with no behavior change.

- **(a) Gamed by**: high-volume engineers bank enough n that a few out-of-band PRs barely move F; low-volume engineers can't influence their band at all in either direction — so the rational response at low n is indifference.
- **(b) Penalizes**: low-volume engineers with genuinely excellent batch discipline (pulled to the middle, boxed out of the top band); interns/part-timers structurally.
- **Patch**: band on **posterior probability**, not the point estimate: assign top band iff P(true in-band share > cutpoint | data) > 0.6 etc. (closed-form Beta posterior, no MCMC — same shipping cost); display the raw share and n beside the band ("9 of 15 in band" is already in the §5.4 copy — make it the banding input's visible justification).

---

## ATTACK 10 — MAJOR — AS_OF = today−14d treats a variable lag as a constant; silent recent-week undercounts demote stages, and the feedback loop is ~a quarter long

**The lag is "~2 weeks", not 14.000 days.** A constant AS_OF assumes a hard completeness guarantee the brief never gives. Any table that occasionally lags 15–20 days silently under-fills the newest 1–2 weekly buckets with no error raised. Consequences concentrate in recency-weighted components: C4 = weeks with ≥1 meaningful day in the last 4 observable weeks. Two lag-corrupted weeks → C4 = 0.5, sitting exactly on Stage 3's C4 ≥ 0.5 boundary; three corrupted → C4 = 0.25 → **stage demotion caused by pipeline latency, not behavior** — then the demotion has to be un-done a week later when data lands, and the 2-recompute hysteresis (which shares 11/12 weeks, Attack 6) will happily confirm the corrupt reading before the backfill arrives. M_prac takes a milder hit: a recent-week deficit lands in ~32% of Theil–Sen pairs (21/66 involve the last 2 weeks); the median usually survives (46/66 pairs are unaffected when 10 clean weeks tie at their level) but the bootstrap interval widens asymmetrically downward.

**Feedback latency.** Behavior change → visible at AS_OF after 14d → enters the weekly recompute up to +7d → hysteresis +7–14d: **~4–5 weeks before a stage can move**, and Momentum needs ≥8 usable weeks of the new pattern → **~2.5 months before "Improving" can acknowledge a real change**. For a self-improvement dashboard, the reinforcement loop is roughly one quarter — the design nowhere states this to the engineer, but §5.4's copy implies week-scale responsiveness ("including last week").

- **(a) Gamed by**: not gameable per se; but an engineer who learns backfill timing can attribute any bad week to "the lag" — plausible deniability that corrodes trust in every state.
- **(b) Penalizes**: engineers whose tools sit on the laggiest tables (tool choice becomes a data-quality lottery); everyone, via a feedback loop too slow to reinforce anything.
- **Patch**: (i) replace the constant with per-source **watermarks**: AS_OF = min over sources of (latest day with completeness ≥ 99% of trailing-4-week average row count); refuse to compute recency components over sub-watermark weeks (mark them "data pending", excluded from C4's numerator AND denominator); (ii) print the actual watermark date per facet, not a blanket "data through"; (iii) state the loop latency in the UI ("changes you make today reach this page in ~3 weeks and the trajectory in ~2 months").

---

## ATTACK 11 — MAJOR — The headline's test-retest reliability is in the same ~0.3–0.5 band as the score it replaced, and the "orthogonal facets" claim is false for P vs M_prac

The headline is a conjunctive classifier: Q-gate ∧ sign(M_prac) ∧ sign(M_out). Conjunctions inherit the noise of their *noisiest* component, not the average. Composing the attack numbers **[inference — illustration]**: for a true "steady, clean" engineer — P(no false Q flag) ≈ 0.73–0.86 per window (Attack 3), P(both slope signs land in the "Steady" cell) ≈ 0.5–0.7 (Attack 1) — headline repeat-concordance lands around **0.4–0.6**, i.e., a 3-state label that changes category for ~half of engineers on re-measurement of identical behavior. The design ridicules the old score's 0.3 reliability (§0, §1.0) but never computes its own; a sentence is not more reliable than a number for being a sentence.

**Correlated components.** "These five facets are orthogonal" (§2.2) is asserted, never shown, and false in at least one place by construction: **P is the level and M_prac is the slope of the same meaningful-day series** — for a series bounded below at 0, level and short-window slope correlate positively (rising series have higher means). C4 (recency) is again the same series' tail. So Practice stage promotions and "Improving" headlines will co-occur; the dashboard will read as one signal told three ways — the pseudo-triangulation pattern crit-stats Flaw 5 diagnosed in the old metric, at lower correlation but the same species. Meanwhile M_prac at the practice **ceiling** (already 5 meaningful days/wk) cannot be >0 — the "Improving" state is structurally reserved for engineers with room below the ceiling, i.e., it rewards a low baseline and invites sandbag-then-ramp cycling across quarters (rolling-baseline decay does not prevent an alternate-quarters cycle: one quarter of "Drifting" buys the next quarter's "Improving").

- **(a) Gamed by**: sandbag/ramp cycling of the practice series across quarters; nothing else needed — noise does the rest.
- **(b) Penalizes**: stable expert users (headline permanently "Steady", M_prac pinned by the ceiling); anyone who takes the label seriously enough to introspect on category changes that are re-measurement noise.
- **Patch**: (i) compute and publish the headline's empirical week-over-week concordance in the pilot; if < 0.8, lengthen windows / coarsen states until it clears (a 2-state headline — "on track" / "check flags" — may be all the data supports; better honest-coarse than fake-fine); (ii) report the P–M_prac correlation in the validation loop and stop claiming orthogonality; (iii) exempt at-ceiling engineers from the M_prac>0 requirement (ceiling ⇒ treat as M_prac>0 satisfied for headline purposes: "sustained at maximum practice depth").

---

## ATTACK 12 — MINOR — Method-of-moments Beta-binomial fits degenerate exactly where the design needs them

MoM for (α, β) requires observed between-engineer variance to exceed binomial sampling variance (overdispersion). In saturated CI regimes (the design's own stated norm: rates → ~1.0) and in small repos, observed variance is routinely ≤ binomial variance → MoM yields **negative or undefined α, β**. Same exposure for (α_f, β_f) per function in F. The spec ships pseudo-SQL but no degenerate-case rule, so the behavior is whatever the batch job's division happens to do.
- **(a) Gamed by**: not gameable; it's a crash/garbage-prior risk, and garbage priors quietly reshape everyone's flags and bands.
- **(b) Penalizes**: engineers in the repos/functions where the fit degenerates (arbitrary).
- **Patch**: floor the prior at a fixed weakly-informative fallback (e.g., strength α+β=10 at the pooled mean) whenever MoM variance ≤ binomial expectation; log which cells fell back; this is one `CASE WHEN`.

## ATTACK 13 — MINOR — The n<20 cohort fallback relocates the old discontinuity; it doesn't remove it

Cell n=19 uses function-level constants; one hire later, n=20 flips every constant (τ-adjacent bands, D(c), caps, cutpoints) to cell-level values estimated from 20 people — the noisiest possible estimates (Attack 5) adopted at the worst possible moment. Crit-stats Flaw 2 mocked 15-person-cell jumps; the hybrid moved the jump to n=20 and made it binary. Also D(c) inherits the same fallback: an engineer's Momentum detrending term can jump because a *colleague was hired*.
- **(a) Gamed by**: not individually gameable.
- **(b) Penalizes**: everyone in cells oscillating around n=20 (reorgs, attrition).
- **Patch**: blend rather than switch: constants := w·cell + (1−w)·function with w = n/(n+20) (precision weighting; still closed-form SQL). Removes the cliff, keeps the pooling intent, and is the "poor-man's partial pooling" the design already claims to be doing but isn't.

## ATTACK 14 — MINOR — Work-mode profile routing is itself a noisy classifier with undefined inputs

`agent-platform` requires pr_weeks ≤ 1 over 12 weeks. A PR-shaped engineer at λ=0.3 PRs/wk has P(pr_weeks ≤ 1) = **14%** per window **[inference — illustration]** — with agent_share ≥ 0.5 they get routed to a profile where their output facets vanish ("unmeasured"), then routed back later; the 2-recompute hysteresis is again ~powerless under 11/12 overlap. And `agent_share` is never defined (share of dollars? tokens? sessions? days?) — four different classifiers, four different populations. Profile flapping changes *which facets exist*, the most jarring possible display change.
- **(a) Gamed by**: an engineer wanting output facets hidden (e.g., a bad Flow band) can nudge agent_share past 0.5 and hold PRs for 2 recomputes — "unmeasured" is a better look than a bottom band; the design says profile shopping "earns unmeasured, never a better band," apparently not noticing unmeasured IS the better band for a bottom-band engineer.
- **(b) Penalizes**: genuinely hybrid engineers oscillating at the boundary — their dashboard reshapes itself quarterly.
- **Patch**: define agent_share (recommend: share of meaningful days, the design's own currency); require profile changes to persist 8 weeks; show the departing facets greyed with "paused (profile change)" rather than removing them; never let a profile switch retire an active "attention" flag.

---

## Summary table

| # | Attack | Severity | One-line counterexample |
|---|---|---|---|
| 1 | Momentum powerless at n=12; threshold = 0.11·SE; state-machine order bug; degenerate bootstrap | **critical** | CI detects only ≥4.2×/quarter output growth; noise +0.0001 slopes read "Improving" as written |
| 2 | Theil–Sen exact-zero ties + D(c) drift ⇒ fabricated "Drifting"; detrending self-defeats in sparse cohorts | **critical** | 62% of λ=0.3 engineers have slope ≡ 0; D(c)=+0.015 ⇒ all of them read "Drifting" |
| 3 | Q flag = below-median test, not anomaly floor; small-n lottery + large-n 50% flag creep + multiplicity | **critical** | p=0.9, n=3 ⇒ 27%/window "Mixed"; 90/100 clean flags, 36/40 doesn't |
| 4 | Validation demotion rule fires on power failure; its success criterion (null throughput) triggers demotion | **critical** | n=10 transitions: 65% chance a working rung shows 2 consecutive nulls |
| 5 | Frozen small-cell quantiles: noise locked a year; drift ceiling (56% in top band) then 36-pp refresh cliff | major | SE(q20)@n=20 = 54% of a band width |
| 6 | C1 construct inversion + boundary coin flips + hysteresis powerless under 11/12 overlap | major | 1-day/wk every week = Stage 0; 2-days/wk half the time = Stage 2; q=0.7 ⇒ P(Stage 3)=0.49 |
| 7 | τ_c grain error (session P10 vs daily total) ⇒ anti-theater floor is a no-op; cron-ramp "Improving" | major | ~90% of single sessions clear a day; spoof cost ≈ one crontab |
| 8 | y LOC-dominated + level-dependent + lines_changed undefined | major | refactorer's cleanup quarter: slope −2.3 ⇒ "Drifting"; low-volume doubling undetectable |
| 9 | EB point-estimate banding = volume ceiling/floor in Flow | major | 4 vs 40 PRs, both 100% in-band: F = 0.714 vs 0.920 |
| 10 | Constant AS_OF vs variable lag ⇒ C4 pipeline demotions; ~quarter-long feedback loop | major | 3 lag-corrupted weeks ⇒ C4 = 0.25 ⇒ Stage-3 loss with zero behavior change |
| 11 | Headline test-retest ≈ 0.4–0.6 (≈ the old score's); P/M_prac share a series; practice-ceiling excludes "Improving" | major | conjunction of a 27% false gate and coin-flip signs |
| 12 | MoM Beta fits degenerate under saturation | minor | all-1.0 repos ⇒ variance ≤ binomial ⇒ α,β undefined |
| 13 | n<20 fallback is a relocated cliff | minor | one hire flips every constant in the cell |
| 14 | Profile routing noisy + agent_share undefined; "unmeasured" beats a bottom band | minor | λ=0.3 engineer: 14%/window misroute to agent-platform profile |

**What survives the attack** (stated for fairness): no spend denominator, no live percentiles, no fake-neutral defaults, no numeric composite, explicit unmeasured states, frozen-anchor non-zero-sum intent, self-vs-self headline, and the unscored cost strip are all sound and strictly dominate the current metric. Every critical attack above has a closed-form, no-MCMC patch (Attacks 1–4 patches); none requires abandoning the chassis. The two patches that must land before pilot: Attack 3 (change the flag reference from median to P10/margin + min-n) and Attack 4 (equivalence-test demotion rule) — both are one-line spec changes with outsized blast radius if shipped as written.
