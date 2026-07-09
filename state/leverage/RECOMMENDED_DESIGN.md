# RECOMMENDED DESIGN — "AI Leverage Profile" (ALP, Anchored Hybrid) — REVISED post red-team

Design synthesizer, 2026-07-08; **revised same day after three red-team passes** (`redteam/gaming.md` A1–A10, `redteam/fairness.md` F1–F20, `redteam/statistics.md` S1–S14). Inputs: five design candidates (`designs/`), four critiques (`crit-*.md`), three-judge panel, three red teams. All external claims re-cited from Phase-A-verified sources with [tags]; my own reasoning labeled **[inference]**. This file is the definitive spec for the final deliverable's §6. Every red-team attack's disposition is recorded in §4.3 (Red-team log); no attack was left un-adjudicated.

---

## 0. The decision and its lineage

**Committed design: the AI Leverage Profile (ALP) chassis from `design-composite.md`, with four grafts and one deletion**, exactly where the three judge hybrids independently converged — then **re-machined after red-team** where the assembly, not the philosophy, was wrong. The red teams' unanimous verdict was "patch, don't redesign": the chassis (no composite number, no spend denominator, no live percentiles, no fake-neutral defaults, explicit unmeasured states, self-vs-self headline, unscored cost strip) survived all three passes; the failures were in specific mechanisms that violated the design's own stated principles (frozen anchors not applied to detrending; evidence-ordering not applied to the quality flag; full-time PR-authoring calendar hard-coded into "absolute" anchors).

| Element | Taken from | What changed vs. the donor |
|---|---|---|
| Chassis: 5 non-compensatory facets (Practice · Flow · Quality-floor · Momentum · Experience), work-mode profiles, volume-gated categorical trajectory headline, no composite number, unscored cost strip, explicit unmeasured states | design-composite (ALP) | Facet display no longer uses live posterior cohort quantiles (graft 1); headline state machine rebuilt post red-team (§1.4) |
| Graft 1 — **published absolute rule anchors** for the Practice facet (0–4 ladder, calendar-verifiable counts) and **frozen cutpoints** everywhere a cohort quantile survives | design-adoption (PML) + design-goodput (NDG) | Post red-team: anchors are **FTE-scaled** (F13), graded not cliff-shaped (S6/A8), and cutpoint freezes are **quarterly blended**, not yearly hard (S5) |
| Graft 2 — **quarterly empirical validation loop with a committed demotion path** | design-adoption (PML) | Post red-team: demotion fires only on **evidence of absence** (equivalence logic), never on failure-to-reject (S4); outcome-coverage gate protects non-PR cohorts (F9); transition population canary-screened (A10) |
| Graft 3 — **durability netting** (path exclusion, both-legs revert zeroing, excess self-rework ≤21d) as the substance of the Quality floor once three cheap metadata acquisitions land — **flag/cap only, never a numerator** | design-goodput (NDG) | Post red-team: flag reference changed from median to a true outlier boundary; per-repo only; min-evidence rule (S3/F10) |
| Graft 4 — **regime-break resets** for the Momentum baseline; display raw AND context trend | design-trend (Trajectory) | Post red-team: detrending term FROZEN and demoted to context — the headline gates on the **raw self-vs-self slope** (F2/A6/S2); reset triggers extended (F6/F16/F17) |
| Deletion — **all within-person spend-elasticity / uplift estimators** removed from the product | judge consensus (3/3) | unchanged |
| Kept from design-cost (LM-E) | FinOps cost-showback strip vs own baseline (never scored) + "investment phase" framing | unchanged; agent-platform users get their own cost cohort context (F8) |

**Where I overruled a judge:** judge 3 (shipping lens) picked PML as sole winner. PML's raw total is highest, but its entire scored construct is counterfeitable by a scripted session habit and it carries zero outcome anchoring — disqualifying as the *sole* design under the mission's adversarial weighting (judges 1–2 concur; red-team A2/S7 confirmed the counterfeit is even cheaper than the judges assumed). I answer the shipping objection structurally: **no MCMC, no live posteriors, no hierarchical Bayes in the score path.** Everything below is closed-form (rule ladders, frozen blended quantile snapshots, Beta-binomial EB with method-of-moments priors + degenerate-case fallback, Theil–Sen + block bootstrap) and implementable as SQL + a small batch job. Phased rollout (§5.5) ships the PML-simple core first.

**Construct (named honestly):** "leverage" is NOT presented as a measured amplification factor — the per-person counterfactual is unmeasurable under the hard limits (METR RCT: experienced devs −19% actual vs +20% believed, https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ [independent]; 2026 update calls its own reversal "very weak evidence", https://metr.org/blog/2026-02-24-uplift-update/ [independent]). ALP operationalizes leverage as **practice + trajectory**: (i) how closely the engineer's observable usage pattern matches practices evidence associates with durable, system-friendly output (DORA capability framing, https://dora.dev/dora-report-2025/ [primary]); (ii) the direction of their own practice and output trend **vs their own baseline** (raw, self-vs-self), with cohort drift shown as unscored context. Quality can only cap, never boost. The headline is a sentence, not a number — and post red-team the design no longer pretends the sentence is automatically more reliable than a number: the pilot must demonstrate headline test-retest concordance ≥0.8 or the state vocabulary is coarsened until it does (S11; §5.5).

---

## 1. Full formulas and every parameter

### 1.0 Global machinery

| Parameter | Value | One-line justification |
|---|---|---|
| Facet window W | rolling **12 weeks** (84d), weekly buckets, zero-filled calendar spine | 4-week reliability ≈0.3 (crit-stats Flaw 7); 12w is brief-sanctioned and matches DX Core 4 lookback [vendor] |
| Momentum history | trailing **26 weeks, biweekly buckets (13 points)** — estimator consumes more history than the display window | 12 weekly points have near-zero trend power (S1: 90% CI excludes zero only at ≥4.2× growth/quarter); brief sanctions window redefinition |
| AS_OF | **per-source completeness watermarks**, not a constant: for each source, latest day with row count ≥ 99% of its trailing-4-week average; AS_OF = min across sources; sub-watermark weeks are **"data pending"** — excluded from recency numerators AND denominators, never treated as zero-activity | "~2 weeks" lag is variable, not 14.000 days; a constant silently under-fills recent buckets and demotes stages via pipeline latency (S10) |
| Refresh | weekly recompute; **UI states the loop latency plainly** ("changes you make today reach facets in ~3 weeks and the trajectory in ~2 months") | honesty about the feedback loop (S10); rungs/bands move on multi-week patterns |
| Cohort c(e) | **function × level** (≈6×6); small cells use a **precision-weighted blend**: constants := w·cell + (1−w)·function, w = n/(n+20) — no hard n<20 switch | function/repo dominate between-person variance (Cui [independent-adjacent]; Jellyfish ~300x cost/PR spread [vendor]); the hard fallback was a relocated cliff — one hire flipped every constant (S13) |
| All cohort-derived constants (τ_c, repo size bands, band cutpoints) | computed on trailing history, then **frozen per period and refreshed as a blend** (new := 0.75·old + 0.25·latest, quarterly; τ_c monthly — see §1.1), published with changelog and **pre-announced migration notes** ("~15% of backend L4s shift down one band at the January refresh — org-wide improvement, not your behavior") | yearly hard freezes lock small-cell sampling noise in for a year, then fire a synchronized mass-demotion cliff at refresh (S5); blending keeps anchors slow-moving and non-zero-sum within any quarter |
| FTE scaling | every absolute day-count threshold scales by HR FTE fraction: x → ⌈x·FTE⌉, published on the engineer's own dashboard as *their* anchor | absolute full-time day counts bind harder per hour worked part-time — a structural penalty on accommodations and 4-day weeks (F13); FTE is an HR fact the engineer cannot set |
| Work-mode profile p(e) | routed on **output-thinness first**: `pr_weeks ≤ 1 → non-PR profile` (sub-labeled agent-platform if agent_share ≥ 0.5, else chat/mixed); `hybrid` if pr_weeks ≥ 2 AND agent_share ≥ 0.3; else `pr-shaped`. **agent_share = share of meaningful DAYS** (defined basis — not dollars/tokens, which misroute on the 30x token-cost variance [independent]). Profiles change **display emphasis only: every facet computes whenever its own data floor is met**, regardless of profile. Profile changes persist ≥8 weeks; departing facets grey out as "paused (profile change)", and a profile switch can never retire an active attention flag | the old router had a hole (low-PR chat-heavy engineers fell into full pr-shaped scoring, F3), an undefined basis (F3/S14), and a dodge (batching merges into one week un-scored Flow and Quality, A1); facets-always-compute kills the dodge |
| Display hysteresis | a stage/band change is confirmed only when **the 4 newest weeks alone support the new state**, or after it persists 4 weekly recomputes; boundary-distance cue always displayed ("you are on the 2/3 boundary") | consecutive weekly recomputes share 11/12 weeks (~0.92-correlated), so "2 consecutive recomputes" confirmed noise, not signal (S6) |
| Never | facets averaged; spend as divisor; missing data as neutral/zero; live peer statistics **anywhere in a gate (including detrending)** | the beyond-repair components per the critiques — extended after red-team found one live peer statistic surviving in Momentum (F2/A6) |

### 1.1 Facet P — Practice (rule-anchored ladder, 0–4) — scored for ALL profiles

Primitives:

```
class(t) ∈ {CHAT, CODING_AGENT, AGENT_PLATFORM}   -- static config table, owner named, quarterly review,
                                                  -- PUBLISHED classification criteria + rationale log (A9.3)
τ_c  = P10 of the nonzero per-DAY token distribution per class, org-wide trailing 26w,
       refreshed MONTHLY as a published blend      -- grain corrected: the old spec tested a DAILY total
                                                  -- against a per-SESSION P10, making the floor a no-op (S7);
                                                  -- monthly refresh tracks model-efficiency drift (F20)
meaningful(u,d,c) = sessions ≥ 1 AND tokens(u,d,c) ≥ τ_c
       -- AGENT_PLATFORM: the runs≥1 requirement is DROPPED in v0 — run attribution (owner vs caller)
       -- is unresolved until acquisition #1, and either resolution is unfair or gameable (F7);
       -- runs feed only canary #5
       -- HONESTY NOTE (S7): τ_c is a data-hygiene floor, NOT an anti-theater defense. Anti-theater
       -- burden sits on: bounded payoffs, timing-entropy canaries, and Stage 4's external-verification
       -- requirement. A ≥2-sessions/day requirement was considered and REJECTED — it would penalize
       -- honest single-long-session deep workers, the population τ_c's P10 placement exists to protect.
active_week(u,w) = any AUTHORED PR event OR any AI session       -- defined: review activity does NOT
       -- make a week active for C1's denominator (F5). v0: review-only weeks are invisible; the
       -- low-AW state below is worded accordingly. Post-acquisition #4: review-only weeks become a
       -- third state, "present — work not measurable here", counting toward AW but excluded from C1.
AW = Σ active_week over W        -- "data pending" (sub-watermark) weeks excluded from both sides
```

Components (each bounded [0,1]; tokens above τ_c and ALL dollars contribute nothing). **All day-count thresholds below are FTE-scaled (⌈x·FTE⌉).**

```
C1 CONSISTENCY = mean over active weeks of  max( min(days_w, 3)/3 ,  min(days_trailing4w, 6)/6 )
   -- GRADED, replacing the old binary ≥2-day week test: the cliff form (a) inverted the construct —
   --   a perfectly consistent 1-day/week user scored Stage 0 below a lumpy 2-day user at Stage 2 (S6);
   --   (b) made one light day strictly worse than total abstention (A8). Partial credit fixes both.
   -- The trailing-4w term is the binge-worker repair (fortnightly deep-work patterns), kept in graded form.
C2 BREADTH   = Σ_c min(1, meaningful_days(c)/⌈5·FTE⌉) / n_classes_available(u)
   -- n_classes_available counts a class only from the user's FIRST SEAT-ASSIGNMENT date (F19), from
   -- per-unit licensing config; stale-config canary watches rows that never change (F1)
C3 AGENTIC   = min(1, meaningful_days(CODING_AGENT ∪ AGENT_PLATFORM) / ⌈10·FTE⌉)
C4 RECENCY   = share of the last 4 ACTIVE weeks (not calendar weeks) with ≥1 meaningful day
   -- redefined on active weeks: the calendar form demoted Stage 3 after an ordinary 2–3 week vacation,
   -- below the ≥4-week reset trigger (F14); data-pending weeks excluded from numerator and denominator (S10)
```

Stage rules (breakpoint ladder — explicitly NOT a weighted composite; Fleming & Wallace forbid ratio/percentile averaging, https://dl.acm.org/doi/10.1145/5666.5673 [primary]):

```
AW < 6            → "LOW OBSERVABLE ACTIVITY" — renamed from "insufficient observation"; copy:
                    "much of your work may not be visible to us." The provisional stage computed over
                    whatever exists is ALWAYS shown with a wide-uncertainty badge (A9.1) — never a blank.
Stage 0 EXPLORING :  C1 < 0.25
Stage 1 OCCASIONAL:  C1 ≥ 0.25
Stage 2 REGULAR   :  C1 ≥ 0.50 AND ≥1 established class
Stage 3 INTEGRATED:  C1 ≥ 0.75 AND established_classes ≥ min(2, n_classes_available(u)) AND C4 ≥ 0.5
   -- min(2, n) replaces the ratio test C2 ≥ 2/n, which made the published target stage
   -- mathematically UNREACHABLE for single-class-licensed teams and hardest for exactly the
   -- license-restricted population the denominator was built to protect (F1)
Stage 4 FRONTIER  :  Stage 3 AND C3 = 1 AND ≥1 externally-verified event in the window:
                     an AUTHORED MERGED PR in a week with agentic-class activity, or
                     (post-acquisition #1) ≥1 distinct NON-SELF caller of an agent they own.
   -- the old "no integrity-canary flag" condition is DELETED: it contradicted §1.8's "canaries are
   -- never per-person" (unimplementable as written — A2/F20), and would have profiled the honest
   -- terse prompters τ_c protects. The external-event requirement replaces it: the top rung can no
   -- longer be reached purely from self-generated telemetry (A2 patch iii).
   -- Pure non-PR agent builders pre-acquisition-#1 see: "Stage 4 not yet assessable for your work
   -- mode — this is our data gap, not your ceiling."
```

Thresholds (0.25/0.50/0.75, 5d, 10d, τ_c=P10) are judgment calls dressed as constants: published, with a quarterly ±30% sensitivity analysis reporting % of engineers whose stage would change (OECD/JRC composite-indicator practice [primary/institutional]). **Stage 3 remains the published TARGET for all roles; Stage 4 is explicitly optional** — the evidence has not settled agentic>chat for all roles (METR seniors-slower tension [independent]) — and the validation loop (§1.6) can demote the rung per cohort.

### 1.2 Facet F — Flow (batch-size discipline) — computed whenever the PR data floor is met

```
lines_changed := additions + deletions as reported by the hosting platform   -- DEFINED (was undefined, S8);
   -- deletions count equally (pro-refactorer). Churn-cycle inflation of lines is a residual until path
   -- exclusion (acq #2) — blast radius is Flow only, where inflated lines push PRs OUT of band (self-defeating).
in_band(pr) := 10 ≤ lines_changed(pr) ≤ P75(lines per PR | band_grain(pr), trailing 26w, frozen-blended qtrly)
   band_grain: repo, IF repo has ≥30 PRs in the trailing 26w (min-evidence rule — a solo author can no
   longer self-set their band by landing huge PRs pre-freeze, A7); else function-level band.
   Shared monorepos above a weekly-PR-volume threshold band by repo × author-function-cell (the dominant
   tenant's P75 read platform teams as undisciplined, F11); repo × top-level directory once acq #2 lands.
   -- 10-line floor: micro-PR confetti earns nothing
F_raw = in_band / PRs ;  posterior = Beta(in_band + α_f, PRs − in_band + β_f), (α_f,β_f) per function by
   method-of-moments; DEGENERATE-CASE RULE: if observed variance ≤ binomial expectation, fall back to a
   weakly-informative prior of strength α+β = 10 at the pooled mean, and log the cell (S12)
Banding: assign band b iff P(true share > cutpoint_b | data) > 0.6  — POSTERIOR-PROBABILITY banding,
   not point-estimate banding: EB point estimates made band assignment volume-confounded (a 4-PR engineer
   with 100% in-band could never reach the top band; a 40-PR one coasted there — S9).
   Raw share and n always displayed beside the band ("9 of 15 in band").
Cutpoints: frozen-blended prior-period function×level quintiles (§1.0); PR cadence sparkline unscored.
```

Small batches are DORA's causal story for AI-era instability [primary]; F **inverts** the old lines numerator — size stops being rewarded, discipline is. F is a share: more PRs never raise it. Slice-at-merge-time cosmetic splitting remains F's declared residual — indistinguishable from genuine small-batch practice without diff content (A7); it no longer pays anywhere else (Momentum now counts PR-days, §1.4).

### 1.3 Facet Q — Quality floor (state, never a score; can only CAP the headline)

```
Scope: STRICTLY PER REPO — clean/checked never pool across repos (pooling reintroduces the repo-mix
   bias the per-repo priors exist to remove, F10). Computable in a repo only at ≥10 checked PRs in
   the trailing window (below that: "not computable" — never "clear", never neutral).
v0 (HAVE): flag_CI in repo r  iff  Wilson95_upper(clean_r, checked_r) < max( P10(rate_EB | repo tier),
                                                                             median(rate_EB | repo tier) − 0.05 )
   -- REFERENCE CHANGED from the median: a below-median test is not an anomaly floor — it flagged a
   -- true-0.9 engineer 27% of windows at n=3 and converges to flagging ~half of all high-volume
   -- engineers (S3). The new reference is a genuine outlier boundary with a declared minimum
   -- practically-significant deficit (non-inferiority margin), and the min-n rule kills the small-n lottery.
   rate_EB = (clean + α_r)/(checked + α_r + β_r), (α_r,β_r) per repo (blend to function), with the same
   degenerate-MoM fallback as §1.2 (S12). Repo tier = CI-job-count decile once acquired.
   Evaluation cadence: computed at MONTHLY FROZEN boundaries, not on every weekly slide (overlapping
   weekly tests made "persistence" free and multiplied false flags, S3); entering `attention` requires
   the test to fire at 2 consecutive INDEPENDENT monthly evaluations; EXIT is immediate when the test
   stops firing (no exit hysteresis — a false flag should not cost a quarter, F12); flags auto-expire;
   a free-text self-annotation ("incident remediation") can be attached and displays beside the flag.
v1 (3 cheap acquisitions): NDG netting as FLAGS —
   flag_revert : both-legs revert-linked rate confidently above repo norm; linkage KEYED TO THE
                 REVERTED PR'S AUTHOR, explicitly (else reciprocal-revert pairs launder both parties, A5)
   flag_churn  : own-file re-touch ≤21d beyond repo-median allowance (per-PR file lists)
                 -- only EXCESS churn: relative not absolute (Nagappan & Ball [independent])
   flag_review : merged-without-review share confidently above repo norm — EXEMPTING merges carrying
                 incident/deploy-tooling labels (break-glass emergency merges are legitimately
                 review-less; flagging them punished incident responders twice, F15)
Q_state ∈ {clear (n=…), attention:[flags], not computable}
   -- evidence is ALWAYS displayed with the state: "Clear (4 checked PRs)" ≠ "Clear (34 checked PRs)" (A5)
```

No default value exists below the floor — "not computable", never clean (kills the fake-1.0 pathology class AND its epistemic twin, the low-evidence "Clear", A5).

### 1.4 Facet M — Momentum (within-person, RAW self-vs-self; cohort drift as context) — the headline driver

```
Series: biweekly buckets b over trailing 26w ending AS_OF (13 buckets); require ≥9 usable buckets.
   Buckets marked "data pending" (sub-watermark) or self-marked on-call/incident (≤3 weeks/quarter,
   rate-limited, logged, displayed; prefer rotation-tool data where it exists — F15) are excluded.
y(e,b)   = log1p( pr_days(e,b) )          -- pr_days = distinct days in the bucket with ≥1 PR
   -- THE LOC TERM IS DELETED at launch (S8): y was LOC-dominated (σ≈1.0 vs 0.48), i.e., the headline's
   -- output leg was a lines trend — the most inflatable object in the literature — and the
   -- additions-only reading would have fired "Drifting" at the org's best refactorers. LOC returns
   -- only after acq #2 (path-filtered, additions+deletions defined) at 0.5 weight, if the validation
   -- loop shows it adds signal.
   -- pr_days (not PR count): same-day salami-slicing adds nothing (A7); bucketing v0 by MERGED date,
   -- switching to OPENED date when acq (opened_at timestamps) lands — merge-timing banking then moves
   -- nothing (A4).
   -- agent-platform / non-PR profile: M_out := NOT COMPUTABLE. The old rule substituted the practice
   -- index for y, which made the entire headline synthesizable from self-generated telemetry — the
   -- ghost-profile attack (A1). No substitution survives.
M_out_raw (e) = TheilSen_b(y(e,b));  SE via moving-block bootstrap (block = 2 buckets, B = 1000)
M_prac(e)     = same machinery on biweekly meaningful-day counts
ZERO-OUT (applied BEFORE the state machine — fixes the order bug where +0.0001 noise slopes read
   "Improving", S1):   slope := 0  iff  |slope| < max( 10%/quarter , 1 × SE(slope) )
SPARSE RULE (S2): if >50% of an engineer's pairwise Theil–Sen slopes are exactly 0 (tie-degenerate),
   M_out := "too sparse to trend" — an unmeasured state, never fed to any gate. (At λ=0.3 PRs/wk,
   62% of engineers had a deterministic zero slope; detrending then manufactured "Drifting" for them.)
COHORT DRIFT (context only, NEVER a gate): D(c) = Theil–Sen slope of the cohort's MEDIAN bucket-series
   (dense even when individuals are sparse), computed on the FROZEN PRIOR QUARTER, published with
   changelog. Displayed beside the raw slope ("output is rising org-wide this quarter"). The old live
   median-of-individual-slopes was (a) a live peer statistic in the headline — the victim/cartel channel
   the design claimed to have deleted (F2/A6); (b) pinned to exactly 0 in sparse cohorts by the same tie
   arithmetic it ignored (S2). Gating on the raw slope accepts mild drift-inflated "Improving" as the
   lesser harm at self-view stakes: a drift-deflated "Drifting" harms the person displayed (F2a).
CEILING WAIVER (S11/A3): if mean meaningful days per active week ≥ ⌈5·FTE⌉ (practice at ceiling),
   the M_prac>0 requirement is treated as satisfied — state "Sustained". The old machine reserved
   "Improving" for people with room below the ceiling, structurally denying its top state to its best users.
Regime resets — baseline truncates on: HR team/function/level change; FTE-fraction change (phase-backs
   and accommodations are HR events too, F16); ≥4 consecutive away weeks; OR a SELF-SERVICE "my role
   changed" reset (≤2/year, logged — the most common senior transition, IC→de-facto-lead, changes no
   HR field, F6) → "RECALIBRATING (k/9 buckets)"; old chart greyed "before your switch".
   "Improving" is SUPPRESSED for the first full window after any reset ("ramping — baseline forming") —
   resets were a scheduled sandbag opportunity (A4).
   New hires: "Drifting" is SUPPRESSED for 24 weeks post-hire (floored at "Steady — post-onboarding
   settling", stated in copy) — the onboarding ramp otherwise guarantees a flattering "Improving"
   followed by a near-certain "Drifting" at months 4–6 (F17). Interns (program shorter than the
   measurement window): descriptive profile only — Practice stage, cost strip, unmeasured panels;
   no Momentum, no headline; declared upfront (F18).

HEADLINE (sentence + glyph, never a number) — evaluation order is normative:
  1  usable buckets < 9                          → "Building baseline (k of 9)"
  2  post-reset first window                     → "Recalibrating" (Improving suppressed)
  3  Q_state = attention (any repo)              → "Mixed — check quality flags"
  4  M_out too sparse / not computable:
       M_prac > 0                                → "Practice trend up — output unmeasured"
       else                                      → "Steady (output unmeasured)"
  5  ceiling waiver AND M_out ≥ 0                → "Sustained — high practice, output steady"
  6  M_prac > 0 AND M_out ≥ 0 AND CORROBORATION  → "Improving"
       CORROBORATION (A3/A5 — "Improving" must not be reachable from self-generated telemetry alone):
       ≥5 checked PRs in-window (Q evidence exists) OR Flow band ≥ prior quarter's band.
       If corroboration fails: "Improving (unverified — quality evidence too thin)".
  7  M_out > 0 AND M_prac ≤ 0                    → "Steady (output up)"     ← volume gate: output volume
                                                                              alone never yields "Improving"
  8  M_out confidently negative (CI excludes 0    → "Drifting — output trending down"
     AND beyond the zero-out floor; suppressed        -- REDEFINED (F4): practice-down/output-steady no
     per the post-hire rule)                          -- longer lands here. A senior who reduces AI usage
                                                      -- while output holds — METR's central finding for
                                                      -- experienced devs [independent] — now reads
                                                      -- "Steady" with a neutral, non-valenced note
                                                      -- ("practice depth declined; output held — if
                                                      -- intentional, no action needed").
  9  else                                         → "Steady"
  Bootstrap P(slope>0) is displayed as graded language ("more likely rising than not — 68%") rather
  than pretending 3-state certainty the data cannot support (S1).
```

The Δ heavy-vs-light-week uplift contrast from the donor design is **not computed** (unchanged).

### 1.5 Facet E — Experience pulse (quarterly, ~9 items) — all profiles, phase 2

Friction/experience **facts only, never self-estimated time savings** (METR 40pp miscalibration [independent]): share of AI-assisted changes needing major rework, friction sources, tool fit; agent builders additionally report who consumes their agents (CHAOSS structured self-report pattern [primary]) — rendered on their profile as a clearly-labeled self-reported consumers list (voice, not score; F8). Displayed vs anonymous function-cohort bands (DXI pattern [vendor]). Satisfies SPACE's ≥1-perceptual-dimension rule (https://queue.acm.org/detail.cfm?id=3454124 [primary]). Non-response = "no data", never scored.

### 1.6 The validation loop (what stops this being a vanity adoption score)

Quarterly, per function×level (hierarchically pooled — this is the one place partial pooling costs nothing, it is quarterly and offline), for engineers with a **sustained** transition (≥8 weeks at a new Practice stage or Flow band): within-person 12w-before vs 12w-after deltas on median weekly merged-PR trend, median PR size + share > cell p90, CI-anomaly rate, and (v1) retouch/revert rates. Effect + CI shown on the dashboard ("evidence: moderate, n=214 transitions").

**Demotion rule, rebuilt (S4/F9/A10):**
- **Fires only on evidence of absence**: the pooled CI must *exclude* the pre-registered minimum effect of interest for 2 consecutive quarters (equivalence/non-inferiority logic). Failure-to-reject NEVER demotes — the old "null-or-negative" rule was an underpowered test whose default outcome was demotion (at n=10 transitions/quarter, a genuinely working rung showed "2 consecutive nulls" with ~65% probability), i.e., a machine for dismantling true thresholds in small cohorts (S4).
- **Joint success criteria are pre-registered per rung** — e.g., "PR-size delta negative AND throughput-delta CI excludes worse than −10%." The old rule's logical bug (its own advertised success mode, "smaller PRs at unchanged throughput", satisfied the demotion condition on the throughput leg) is thereby closed (S4).
- **The loop publishes its own power** beside every verdict ("this cohort's loop can only detect effects > 38%/quarter — verdicts here are uninformative") and displays "insufficient evidence" rather than borrowing significance.
- **Outcome-coverage gate (F9):** the demotion path is live only in cohorts where the measured outcome set covers the cohort's work mode (proxy: median pr_weeks above a published threshold). Elsewhere the loop reports "cannot validate — outcomes unmeasured for this cohort" instead of demoting the only scored facet non-PR cohorts have.
- **Integrity screen (A10):** transition cohorts are screened through the aggregate canaries before inclusion; every verdict reports sensitivity to excluding canary-flagged weeks; cohort analyses pre-registered. Otherwise theater-driven transitions (which mechanically contribute null deltas) become the evidence that demotes honest cohorts' rungs.

Outcomes NEVER enter any individual's score (DORA 2024: individual gains and system outcomes diverge, https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report [primary]). The loop additionally reports the empirical P–M_prac correlation (they share a data series — level and slope of meaningful-days; the design does not claim facet orthogonality, S11) and the headline's empirical week-over-week concordance (§5.5 gate).

### 1.7 Cost Awareness strip — displayed, NEVER scored

Own 13-week spend sparkline vs own trailing-26w baseline + tool-category mix + anonymous cohort IQR; **agent-platform-profile engineers are contextualized against agent-platform users' IQR, not their function cohort's** (agentic workflows are structural token outliers; the function IQR framed them as extreme, F8); high-spend periods framed "investment phase", never "wasteful" (spend ≠ difficulty ≠ waste: 30x same-task token variance, https://arxiv.org/abs/2604.22750 [independent]; per-dollar ranks invert productivity, $0.28→$89.32/PR, https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/ [vendor]; FinOps showback stops above the individual, https://www.finops.org/wg/finops-for-ai-overview/ [primary]).

### 1.8 Integrity canaries — org-side aggregate monitoring, never per-person alerts, never a gate

(1) PR-size pile-up just under band edges; (2) falling share of CI-checked PRs / falling pr_weeks; (3) near-floor session-token distributions / near-zero daily-token variance; (4) retouch-lag bunching past day 21 (v1); (5) agent runs with ~zero tokens; **added post red-team:** (6) **timing-entropy** — near-zero entropy in time-of-day and inter-session intervals (cron signatures; humans are irregular — A2); (7) same-day multi-merge-to-same-repo bursts (salami slicing, A7); (8) band-edge / cutpoint pile-up in F shares (A9.2); (9) reciprocal-revert pairs once revert linkage lands (A5); (10) self-marked on-call week rates (F15); (11) `class_availability` rows that never change (stale licensing config, F1). Canaries inform threshold retuning and org-side investigation only — **no canary gates any individual's stage, band, or headline** (the one clause that violated this, Stage 4's, is deleted; §1.1).

---

## 2. Why this design wins

### 2.1 Vs the other four candidates

- **vs PML alone**: PML's scored construct is counterfeitable by a scripted session habit (red-team S7 showed the spoof is one crontab), and it contains zero output, zero quality, zero trajectory. ALP keeps PML's best organs (rule anchors, validation loop, ladder interpretability) but couples them to an outcome-side headline whose top states now require **externally-anchored evidence** (merged-PR events, checked-PR corroboration) that self-generated telemetry cannot supply.
- **vs NDG alone**: NDG's currency is lines — the most inflatable object in the literature (GitClear [vendor]) — its own text concedes v0 "should lose to a practice/trend design". ALP takes only its netting (as flags) and its frozen-cutpoint discipline; post red-team ALP also deleted its own LOC exposure from the headline (S8).
- **vs Trajectory alone**: the most causally careful machinery, but underpowered — a defect ALP initially inherited (S1/S2) and has now engineered around (26w biweekly series, tie-degeneracy states, graded probability language) rather than denied. ALP embeds trend under a headline that also credits *practice*, so a null output trend does not read as a null person.
- **vs LM-E alone**: still promotes the weakest-validity quantity (within-person spend-output covariation) to the headline. ALP keeps only its showback strip and protective framing; spend never touches a scored quantity.
- **Under the mandated adversarial weighting**, ALP was the only candidate ≥8 on BOTH gaming-resistance and fairness (2 of 3 judges' winner); this revision removes the scored weaknesses three red teams found: the ghost profile (A1), the live detrending term (F2/A6), the below-median quality lottery (S3), the self-dismantling validation rule (S4), and the full-time-calendar bias in the anchors (F13/F14/F1).

### 2.2 Vs the current metric's 7 known problems, point by point

1. **CI-clean weak/unfair quality proxy** → CI demoted from multiplier to a per-repo, minimum-evidence, outlier-boundary flag that can only cap, never boost; discrimination reassigned to acquirable churn/revert/review counterweights. Saturation becomes harmless instead of score-defining — and the flag's reference is a true anomaly boundary, not the median (a below-median test would have rebuilt the old lottery one layer up, S3).
2. **Narrow spend denominator; agent work invisible** → no denominator anywhere; ALL tools (incl. agent platform) feed Practice; agent-platform engineers get a scored P/M(practice)/E profile plus explicit "unmeasured" (not zero) output facets — and, post red-team, an honest headline vocabulary ("Practice trend up — output unmeasured") instead of a synthesizable substitute (A1); the #1 acquisition gives them a true output signal and is now a GA precondition for their profile (F8).
3. **Right-skewed per-dollar percentiles; top ranks = CI noise** → no ratios, no continuous percentile score; log-scale series, rule ladders, frozen-blended coarse bands with non-overlapping-evidence hysteresis and intervals; the headline is self-vs-self on the RAW slope, immune to cohort skew, rank churn, and (post-F2) to other people's adoption speed.
4. **Brittle cold-start floors** → all AND-gates deleted; explicit LOW-OBSERVABLE-ACTIVITY / Building-baseline / Recalibrating states with always-visible provisional stages; EB shrinkage with displayed intervals and degenerate-case fallbacks; no on/off flicker.
5. **Per-dollar penalizes harder, more expensive work** → spend never divides anything; unscored self-showback with "investment phase" framing (the Jellyfish inversion is impossible by construction).
6. **Level-only cross-function cohort** → function×level cohorts with precision-weighted blending (no n=20 cliff, S13) + repo-grain normalization with min-evidence and monorepo splits (F11/A7) + score-neutral work-mode profiles whose facets always compute when their data floor is met.
7. **CI clustering at 1.0 / fake-neutral default** → the <3-PR ⇒ 1.0 default is abolished; low evidence yields "not computable" and evidence-labeled states ("Clear (n=34)"), never the best displayable state (A5); evidence ordering is restored — and, post-S3, more evidence no longer *increases* false-flag exposure.

Structural wins beyond the list: the old metric's two "signals" were ~0.9-correlated pseudo-triangulation (crit-stats Flaw 5) — ALP's facets are deliberately in tension, though the design **does not claim orthogonality**: P and M_prac share the meaningful-days series (level vs slope) and their empirical correlation is reported by the validation loop (S11). The old optimum was abandoning AI (extremal Goodhart) — ALP's best profile requires sustained, consistent, small-batch, flag-free, improving AI-assisted work with externally-verified output events, which is what the org actually wants.

---

## 3. How it handles each hard limit in the brief

1. **No AI-vs-human attribution** → no facet claims to isolate the AI's contribution; the construct is displayed as practice + trajectory; the headline compares the engineer only to themselves (raw slope). Nothing breaks if attribution never arrives.
2. **No task difficulty / business value** → no cross-person volume verdict exists: Flow is a discipline *share*, Momentum is self-relative and volume-gated, bands are frozen context not ranks, and spend is unscored. A hard, slow, expensive quarter reads as "Steady" with an investment-phase cost strip, not as failure — and post-F4, a quarter of *deliberately reduced* AI use with held output also reads "Steady", not "Drifting".
3. **No review depth / no defect linkage** → quality is an anomaly floor (flags on confident outliers vs repo norms with minimum evidence), never a discriminating score; the real counterweights (retouch/revert/review flags) are metadata-only acquisitions; org-level truth-testing lives in the cohort validation loop under equivalence logic, not in per-person defect claims.
4. **Agent-platform engineers (no output signal)** → first-class on the practice side (sessions/active days are score inputs; AGENT_PLATFORM is a class; C3 values their mode; the unattributable runs requirement is dropped v0, F7); routed to their own profile where output facets display **"unmeasured — we don't score what we can't see"** and the headline says "Practice trend up — output unmeasured" (never a synthesized "Improving", A1); upgraded to a true output facet (distinct repeat non-self consumers, EB-shrunk) when run→caller identity lands — now a **GA precondition** for this profile rather than merely "top acquisition" (F8).
5. **~2-week data lag** → per-source completeness watermarks replace the constant AS_OF; sub-watermark weeks are "data pending", excluded from recency numerators and denominators (no pipeline-latency demotions, S10); UI prints the actual watermark per facet and states the feedback-loop latency plainly.
6. **Privacy / strictly self-view** → no live peer statistic exists anywhere — including Momentum's context drift, now a frozen prior-quarter constant (F2/A6); cohort context is frozen, anonymous, aggregate; canaries are org-side aggregates that never gate an individual (the Stage-4 contradiction is resolved by deletion, F20); survey results self-view with anonymous bands.

---

## 4. Red team

### 4.1 How an engineer inflates it — attack table (post-revision state of every scored/displayed signal)

| Signal | (a) Gamed by | Design's answer (post-revision) | (b) Unfairly penalizes | Design's answer |
|---|---|---|---|---|
| P: consistency (C1) | scripted daily τ_c-clearing sessions (jittered to defeat distribution canaries) | τ_c grain-corrected to per-day P10 (S7) but **honestly repriced as data hygiene, not defense**; timing-entropy canary (org-side); payoff saturates at a rung; Stage 4 now needs an externally-verified event; graded C1 means theater buys stages, never the headline alone (corroboration gate) | fortnightly-binge deep workers; part-timers; light-but-daily users | graded credit with trailing-4w term; FTE-scaled thresholds (F13); 1 day/wk ≈ 0.67 credit, no longer Stage 0 (S6); PTO drops out via active weeks + C4-on-active-weeks (F14) |
| P: breadth (C2) | one checkbox day per class | ⌈5·FTE⌉ meaningful days to establish a class; capped at n_classes_available from seat dates | single-class-licensed teams; new hires awaiting seats | Stage 3 needs min(2, n_available) — reachable at every licensing level (F1); classes count from first seat assignment (F19) |
| P: agentic (C3) | no-op agent runs (Meta/Amazon precedent, https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/ [independent]) | runs no longer feed meaningful() v0 (F7); zero-token-run canary; Stage 4 external-event gate; caller-identity acquisition is the real fix | chat-appropriate roles; builders of scheduled agents | Stage 3 is the published target; Stage 4 labeled optional + "not yet assessable" where honest (§1.1); validation loop can demote the rung per cohort |
| F: in-band share | splitting large changes into in-band chunks; band poisoning in solo repos; repo shopping | posterior-probability banding (no volume ceiling/floor, S9); repo_band min-evidence rule (≥30 PRs) kills self-set bands (A7); pile-up canaries; splitting no longer pays in Momentum (pr_days); F-side cosmetic splitting = declared residual | legitimate large-unit work; monorepo platform teams; low-volume disciplined engineers | per-repo/function-cell bands (F11); posterior banding lets a 4-PR perfect record reach the top band with honest uncertainty (S9); F is one facet, never the headline |
| Q: CI floor | keep PRs off CI paths; keep volume low | "not computable" ≠ clear, evidence always displayed (A5); checked-share canary; min-n + outlier boundary mean low volume no longer strictly dominates (S3: high volume no longer converges to 50% flag rate, so the incentive to starve evidence is gone) | flaky/heavy-CI repo residents; prolific engineers | per-repo scope, never pooled (F10); P10/margin reference fires only on confident true outliers; monthly independent evaluations; immediate exit + annotation (F12) |
| Q: netting flags (v1) | delay fixes past day 21; fix-in-new-files; teammate-landed reverts | retouch-lag bunching canary; revert linkage keyed to reverted PR's author + reciprocal-pair canary (A5); declared undetectable residuals; flags cap, never subtract | prototypers/iterators; incident responders | only EXCESS churn vs repo median; incident-labeled merges exempt from flag_review (F15) |
| M: trends | sandbag baseline; bank merges; ride org drift; ramp practice telemetry | opened-date bucketing kills merge banking (A4, post-acq); pr_days kills salami slicing (A7); post-reset Improving suppression kills reset seeding (A4); raw-slope gating + frozen context D(c) kills the victim/cartel channel (F2/A6); zero-out + sparse rules kill noise-"Improving" and fabricated "Drifting" (S1/S2); corroboration gate means a practice ramp alone cannot reach "Improving" (A3) | task-mix shifts; lumpy-cadence engineers; refactorers; low-volume engineers; new hires; seniors de-adopting | wide-interval graded states; sparse = unmeasured, never negative (S2); LOC deleted from y (S8); Drifting only on confident raw decline (F4); 24-week new-hire floor (F17); self-service role-change resets (F6); on-call week exclusions (F15) |
| E: survey | inflate/sandbag; phantom agent consumers | facts-not-estimates wording; self-view; never scored; consumers list labeled self-report (F8) | non-respondents, honest pessimists | "no data" state, never a penalty |
| Cost strip | nothing to gain — unscored | that is the point | monorepo/large-context and agent-platform engineers *look* heavy | own-baseline framing; agent-platform IQR context (F8); no valence |
| Headline | the composed play: jittered practice ramp + salami splitting + merge banking + evidence starvation (gaming.md §B) | each leg individually broken post-revision: ramp alone → "Improving (unverified)" at best without ≥5 checked PRs or a Flow-band rise; splitting adds nothing to pr_days; banking dies at opened-date; starvation forfeits the corroboration gate it needs. The remaining spoof requires real merged, CI-checked PRs — i.e., real work | at-ceiling expert users; stable engineers | ceiling waiver → "Sustained" (S11); "Steady" framed as the honest majority state with facet levers carrying engagement (§7.6) |

**Residual attack honestly stated (revised):** sustained practice theater still lifts the Practice *stage* (bounded, private label, canaried — and now honestly labeled as such rather than claimed to be defended by τ_c, S7). Reaching **"Improving"** now additionally requires non-declining, externally-verified output plus checked-PR or Flow corroboration — the fronts no longer share one data source (A1/A2/A3 closed). An alternate-quarter ramp/coast sawtooth on practice remains possible and is accepted: its payoff is one private label, its "Improving" still needs the corroboration gate each cycle, and coast quarters no longer read "Drifting" (F4) so the cycle's incentive is weak. Priced in as if the score will leak (Amazon KiroRank disbelief precedent [independent]).

### 4.2 Who it unfairly penalizes — the residual list and the design's answer

1. **Highly productive minimal-AI engineers** read "Stage 0–1" on Practice. Contained: the headline is self-vs-self, and post-F4 a *decline* in AI use with held output reads "Steady" with a neutral note, not "Drifting" — skeptics are genuinely not headline-penalized now, including ones whose skepticism grew mid-window. The METR seniors-slower tension is the validation loop's job to adjudicate.
2. **Reviewers, mentors, designers, incident responders** — v0: invisible to P/F/M; the low-AW state is worded as a data admission ("much of your work may not be visible to us"), never as a deficiency, with the provisional stage always shown (F5/A9.1). Self-service role-change resets and (post-acq #4) "present — work not measurable here" weeks and review-share Drifting suppression close most of the gap (F5/F6). Honesty ≠ measurement; declared.
3. **Agent-platform engineers** — scored Practice/M_prac/E, honest "output unmeasured" states, their own cost-context cohort, and a **GA gate**: the profile ships beyond labeled beta only when caller identity lands (F8). Their residual thin profile is now a stated data-acquisition debt with a deadline, not a permanent condition.
4. **ML engineers with unmerged experimentation** — output facets under-read them; sparse-trend rule shows "too sparse to trend" (never "Drifting", S2); the unmeasured panel names unmerged work.
5. **Small/rare cohorts (e.g., L8 SRE)** — precision-weighted blending replaces cliff fallbacks (S13); the validation loop refuses to issue verdicts beyond its power there (S4); band whiskers + "small cohort" note disclose it.
6. **Part-timers, leave-takers, phased returns** — FTE-scaled anchors (F13), C4 on active weeks (F14), FTE-change regime resets (F16), on-call exclusions (F15).
7. **New hires / role changers / interns** — Building-baseline, Recalibrating, the 24-week Drifting floor (F17), self-service resets (F6), seat-date breadth denominators (F19), intern descriptive profiles (F18).

### 4.3 Red-team log — every attack → disposition

**Gaming report (redteam/gaming.md):**

| ID | Attack (one line) | Disposition |
|---|---|---|
| A1 | Agent-platform "ghost profile": substituted output series made the whole headline synthesizable; profile-dodge laundering | **Patched** — M_out substitution deleted; new honest headline states; facets compute whenever their data floor is met regardless of profile; agent_share defined on meaningful days (§1.0, §1.4) |
| A2 | Practice theater w/ distribution-shaped jitter; Stage-4 canary gate unimplementable (contradicts §1.8) | **Patched** — canary clause deleted from Stage 4 and replaced with an external-verification event; timing-entropy canary added; τ_c honestly reframed as hygiene, not defense (§1.1, §1.8). Weak-form ambient theater **accepted-as-residual** (§7.5) |
| A3 | Practice ramp + flat output → "Improving" (no mirror gate); ceiling inversion denies top state to best users | **Patched** — cross-source corroboration gate on "Improving"; ceiling waiver → "Sustained" state (§1.4) |
| A4 | Merge-timing banking sawtooth; post-reset baseline seeding | **Patched** — pr-day bucketing switches to opened_at (new acquisition); "Improving" suppressed first window post-reset (§1.4, §6) |
| A5 | Q floor never binds: evidence starvation renders best state; revert laundering; author-linkage ambiguity | **Patched** — evidence-labeled states; corroboration gate ties the headline to checked-PR evidence; revert linkage keyed to reverted PR's author; reciprocal-pair canary (§1.3, §1.4) |
| A6 | D(c) live cohort median = restored victim/cartel channel | **Patched** — headline gates on raw slope; D(c) frozen prior-quarter, context-only (§1.4) |
| A7 | PR splitting triple-dip; P75 reconstructable/poisonable; repo shopping | **Patched** in Momentum (pr_days) and banding (min-evidence repo_band, canaries); F-side cosmetic splitting **accepted-as-residual** — unfixable without diff content (§1.2, §7.5) |
| A8 | C1 active-week cliff: 1 light day strictly worse than abstention | **Patched** — graded C1 (§1.1) |
| A9 | (1) observation suppression, (2) cutpoint camping, (3) tool_class lobbying | **Patched** (cheap): always-visible provisional stage; band-edge pile-up canary; published classification criteria + rationale log (§1.1, §1.8). Camping pressure itself **accepted-as-residual** (any published boundary invites it, §7.8) |
| A10 | Validation-loop poisoning by theater-driven or strategically-timed transitions | **Patched** — canary-screened transition cohorts, sensitivity reporting, pre-registration (§1.6) |

**Fairness report (redteam/fairness.md):**

| ID | Attack | Disposition |
|---|---|---|
| F1 | Stage 3 unreachable for n_classes<3 (C2 ratio inversion) | **Patched** — min(2, n_available) rule + stale-config canary (§1.1) |
| F2 | Live detrending = zero-sum victim channel in the headline | **Patched** — raw-slope gating; frozen context D(c); org-drift copy note. Mild drift-inflated raw "Improving" **accepted** as the lesser harm, per the red team's own trade analysis (§1.4) |
| F3 | Routing hole: low-PR chat-heavy engineers → full pr-shaped scoring; agent_share undefined | **Patched** — output-thinness-first routing; agent_share = meaningful days (§1.0) |
| F4 | Practice-down/output-steady ⇒ "Drifting" punishes evidence-based de-adoption | **Patched** — "Drifting" reserved for confidently-negative raw output; neutral practice note (§1.4) |
| F5 | active_week ambiguity: reviewers get insulting state or Practice demotion | **Patched (phased)** — authored-only definition committed; state renamed with honest copy + provisional stage v0; "present" weeks at acquisition #4 (§1.1) |
| F6 | Role drift (IC→lead) has no reset; contradicts unmeasured panel | **Patched** — self-service rate-limited resets; review-share Drifting suppression at acq #4 (§1.4) |
| F7 | Agent-run attribution ambiguity: builders undercounted or vacation-theater minted | **Patched** — runs dropped from meaningful() v0; canary-only (§1.1) |
| F8 | Agent-platform profile permanently "Steady"; no honest action improves any display | **Patched** — acquisition #1 elevated to GA precondition; self-reported consumers list; agent-platform cost IQR (§1.5, §1.7, §6). Interim thinness **accepted-as-residual** with a stated deadline (§7.4) |
| F9 | PR-shaped validation outcomes will demote non-PR cohorts' only scored facet | **Patched** — outcome-coverage gate on the demotion path (§1.6) |
| F10 | Q cross-repo pooling undefined ⇒ repo-mix bias | **Patched** — strictly per-repo, min-n per repo, never pooled (§1.3) |
| F11 | Monorepo Flow bands set by dominant tenant | **Patched** — repo×function-cell split above volume threshold; repo×top-dir at acq #2 (§1.2) |
| F12 | Q attention: exit hysteresis delays exoneration; no expiry/annotation | **Patched** — entry-only persistence, immediate exit, auto-expiry, self-annotation (§1.3) |
| F13 | Absolute day counts don't pro-rate FTE | **Patched** — global FTE scaling of all day thresholds (§1.0) |
| F14 | C4 demotes Stage 3 after a 2–3 week vacation | **Patched** — C4 on last 4 ACTIVE weeks (§1.1) |
| F15 | On-call months drag Momentum; break-glass merges trip flag_review | **Patched** — self-marked (rate-limited) or rotation-data week exclusions; incident-label exemption (§1.3, §1.4) |
| F16 | FTE changes / phased returns not regime events | **Patched** — FTE change added to reset triggers (§1.4) |
| F17 | Onboarding ramp ⇒ predictable "Drifting" at months 4–6 | **Patched** — 24-week post-hire Drifting floor with stated copy (§1.4) |
| F18 | Interns unspecified end-to-end | **Patched** — descriptive profile only, declared upfront (§1.4) |
| F19 | License-provisioning lag undercounts new-hire breadth | **Patched** — classes count from first seat-assignment date (§1.1) |
| F20 | Stage-4 canary gate contradiction; stale τ_c penalizes efficient-model users | **Patched** — canary clause deleted (with A2); τ_c monthly blended refresh (§1.1) |

**Statistics report (redteam/statistics.md):**

| ID | Attack | Disposition |
|---|---|---|
| S1 | Momentum powerless at n=12; decorative threshold (0.11·SE); state-machine order bug; degenerate bootstrap | **Patched** — 26w biweekly series; zero-out at max(10%/qtr, 1·SE) BEFORE the state machine; graded P(slope>0) language; sparse rule handles degenerate bootstrap (§1.4). Low power for detecting modest real declines **accepted-as-residual** — by design, "Drifting" fires only on confident evidence (§7.6) |
| S2 | Theil–Sen zero-tie + detrending ⇒ fabricated "Drifting"; detrending self-defeats in sparse cohorts | **Patched** — >50%-tie ⇒ "too sparse to trend" unmeasured state; D(c) from the dense cohort median series, frozen (§1.4) |
| S3 | Q flag = below-median test: 27%/window false "Mixed" at n=3; ~50% flag creep at high n; multiplicity | **Patched** — outlier-boundary reference max(P10, median−0.05); min n=10/repo; monthly independent evaluations; persistence across independent windows (§1.3) |
| S4 | Validation demotion fires on power failure; success criterion (null throughput) triggers demotion | **Patched** — equivalence-logic demotion; pre-registered joint criteria; pooled min-n; published power (§1.6) |
| S5 | Frozen small-cell quantiles: year-long noise lock-in; drift ceiling; synchronized refresh cliff | **Patched** — quarterly blended refresh; precision-weighted small-cell shrinkage; distance-to-boundary display; pre-announced migrations (§1.0) |
| S6 | C1 construct inversion; boundary coin flips; hysteresis powerless under 11/12 overlap | **Patched** — graded C1; non-overlapping-evidence hysteresis; boundary-distance cue (§1.0, §1.1). Boundary noise for true-borderline engineers **accepted-as-residual** — disclosed via the cue (§7.8) |
| S7 | τ_c grain error (session P10 vs daily total): anti-theater floor is a no-op; crontab "Improving" | **Patched** — per-day grain; honesty reframe (τ_c = hygiene); corroboration gate closes the crontab-headline path. The red team's ≥2-sessions/day sub-patch **rejected-with-reason**: it would penalize honest single-long-session deep workers, contradicting the floor's own protection goal; theater defense relocated to bounded payoffs + external-event gates instead (§1.1) |
| S8 | y LOC-dominated, level-dependent; lines_changed undefined; refactorers read "Drifting" | **Patched** — LOC deleted from y at launch (reinstated only path-filtered at 0.5 weight post-acq #2, validation-gated); lines_changed defined (additions+deletions); level dependence largely resolved by bounded pr_days + sparse state; remaining log-scale compression at very low volume **accepted-as-residual**, mitigated by the sparse state (§1.2, §1.4, §7.8) |
| S9 | EB point-estimate banding = volume ceiling/floor in Flow | **Patched** — posterior-probability banding; raw share + n displayed (§1.2) |
| S10 | Constant AS_OF vs variable lag ⇒ pipeline demotions; ~quarter feedback loop unstated | **Patched** — per-source watermarks; data-pending weeks excluded both sides; per-facet watermark display; loop latency stated in UI. The ~2-month trajectory latency itself **accepted-as-residual** — inherent to trend measurement under the lag limit (§7.6) |
| S11 | Headline test-retest ≈0.4–0.6; orthogonality claim false (P vs M_prac); ceiling excludes "Improving" | **Patched** — pilot concordance gate ≥0.8 with committed coarsening path (§5.5); orthogonality claim withdrawn + correlation reported (§1.6, §2.2); ceiling waiver (§1.4) |
| S12 | MoM Beta-binomial fits degenerate under saturation | **Patched** — weakly-informative fallback prior, logged (§1.2, §1.3) |
| S13 | n<20 cohort fallback is a relocated cliff | **Patched** — precision-weighted blend w=n/(n+20) (§1.0) |
| S14 | Profile routing noisy; agent_share undefined; "unmeasured" beats a bottom band | **Patched** — defined basis; 8-week persistence; greyed "paused" facets; switches never retire flags; facets-always-compute removes the unmeasured-beats-bottom-band payoff (a computable bottom band stays displayed) (§1.0) |

---

## 5. Implementation sketch — HAVE data only

### 5.1 Tables (warehouse)

- `usage_daily(user_id, day, tool_id, dollars, tokens, sessions, active_day, runs)` — HAVE
- `prs(user_id, pr_id, repo_id*, merged_at*, lines_changed)` — HAVE (*verify repo_id + merge timestamps; current 4-week windowing implies merge dates exist [inference — verify]; `opened_at` is a new near-zero-cost acquisition, §6)
- `ci(pr_id, checked, clean)` — HAVE
- `hr(user_id, function, level, team, fte_fraction, effective_from)` — HAVE (fte_fraction exists for payroll; verify exposure)
- NEW-trivial config: `tool_class(tool_id, class, criteria_ref)`; `class_availability(org_unit, class, licensed_from)`; `seat_assignment(user_id, class, assigned_from)` (exists wherever per-user spend exists)
- Derived snapshots (frozen-blended, versioned): `tau(class, month, tau_c)`; `repo_band(band_grain_id, quarter, p75_lines, n_prs)`; `f_cutpoints(cell, quarter, q1..q4)`; `cohort_drift(cell, quarter, d_c)`; `watermark(source, as_of_day)`
- `self_events(user_id, kind∈{role_change_reset, oncall_week, q_annotation}, week, note, logged_at)` — new, tiny

### 5.2 Weekly batch (per engineer)

1. Resolve per-source watermarks → AS_OF; build zero-filled week spine over trailing 26w; mark active weeks, data-pending weeks; compute meaningful days per class (join `tau`, day-grain).
2. Compute graded C1, C2 (seat-dated availability), C3, C4 (active-week window), all FTE-scaled → stage; hysteresis on the 4 newest weeks / 4-recompute persistence; Stage-4 external-event check.
3. Join PRs to `repo_band` (min-evidence grain) → Beta posterior → posterior-probability band vs `f_cutpoints`; degenerate-MoM fallback.
4. Q: per-repo, monthly-boundary evaluation, min n=10, outlier-boundary Wilson test → flags; annotations joined.
5. Biweekly pr_days series (on-call/data-pending buckets excluded) → Theil–Sen + block bootstrap; zero-out; sparse check; frozen `cohort_drift` context; regime-reset check vs `hr` + `self_events`; headline state machine in the §1.4 normative order.
6. Emit one row per engineer per facet: {value, band/stage/state, interval, because-facts, watermark_date}.

### 5.3 Pseudo-SQL (core Practice + Momentum inputs)

```sql
WITH meaningful AS (
  SELECT u.user_id, u.day, tc.class,
         (u.sessions >= 1 AND u.tokens >= t.tau_c) AS meaningful     -- tau_c is DAY-grain P10 (S7)
  FROM usage_daily u
  JOIN tool_class tc USING (tool_id)
  JOIN tau t ON t.class = tc.class AND t.month = month_of(watermark(u.tool_id))
  WHERE u.day <= (SELECT MIN(as_of_day) FROM watermark)              -- per-source watermarks (S10)
),
weeks AS (                                                -- zero-filled spine × engineer; data-pending
  SELECT e.user_id, s.week, s.data_pending,               -- weeks flagged, excluded downstream
         COUNT(DISTINCT m.day) FILTER (WHERE m.meaningful)           AS meaningful_days,
         MAX(CASE WHEN p.pr_id IS NOT NULL OR m.day IS NOT NULL
                  THEN 1 ELSE 0 END)                                 AS active_week,   -- authored PRs only (F5)
         COUNT(DISTINCT day_of(p.merged_at))                         AS pr_days        -- pr_days, not PRs/LOC
  FROM engineers e CROSS JOIN week_spine s
  LEFT JOIN meaningful m ON m.user_id = e.user_id AND week_of(m.day) = s.week
  LEFT JOIN prs p        ON p.user_id = e.user_id AND week_of(p.merged_at) = s.week
  GROUP BY 1, 2, 3
)
SELECT user_id, week,
       -- graded C1 credit per active week (S6/A8), FTE scaling applied downstream
       CASE WHEN active_week = 1 THEN
         GREATEST( LEAST(meaningful_days, 3) / 3.0,
                   LEAST(SUM(meaningful_days) OVER (PARTITION BY user_id ORDER BY week
                         ROWS BETWEEN 3 PRECEDING AND CURRENT ROW), 6) / 6.0 )
       END                                                            AS c1_credit,
       LN(1 + pr_days)                                                AS y_week   -- LOC term deleted (S8)
FROM weeks WHERE NOT data_pending;
-- Biweekly aggregation, Theil–Sen, block bootstrap, zero-out, sparse rule: batch job (Python), not SQL.
```

### 5.4 Dashboard copy (the sentences the engineer sees)

> **Trajectory: Improving** — over the last ~6 months your practice depth rose and your merged output trended up vs your own baseline (more likely rising than not — 82%). Corroborated by 11 CI-checked PRs, no quality flags. Note: output is rising org-wide this quarter. Data through Jun 22 (usage) / Jun 27 (PRs). You are only ever compared to yourself; peers appear as anonymous context. Changes you make today reach these facets in ~3 weeks and this trajectory in ~2 months.
>
> **Practice — Stage 3, Integrated**: you used AI meaningfully in 10 of your 12 active weeks, across 2 of the 3 tool classes available to you, including recently. You are near the Stage 3/4 boundary. Stage 4 is an optional frontier — 10+ agentic days this quarter plus a merged PR in an agentic week (you have 4 days). Cohort context: 38% of backend L5s are Stage 3+. Evidence note: backend engineers who sustained Stage 4 shipped ~15% smaller PRs at unchanged throughput (moderate evidence, n=214 transitions, loop power: detects effects >8%, updated Apr 2026).
>
> **Flow — Established (9 of 15 PRs in band)**: 4 were >3x your repos' norms. Lever: slice large changes — small batches are the strongest evidence-backed AI-stability practice.
>
> **Quality floor — Clear (11 checked PRs)**: no anomalies vs your repos' norms. This floor only ever flags confident outliers; it never boosts.
>
> **Cost awareness** (never scored): your spend is 12% above your 6-month baseline, typical for your cohort — high-spend stretches often mean harder work or exploration.
>
> **Not visible to us**: code review you give, mentoring, design, incident work, unmerged experiments, your agents' outputs. Absence of signal here is not absence of value. If your role has shifted toward these, you can reset your baseline (twice a year).

Agent-platform variant: *"≈70% of your recorded AI work happens on the agent platform, which has no output signal yet — your Practice and practice-trend are scored; output is unmeasured, not zero, and your headline reads 'Practice trend up — output unmeasured.' Making agent outputs measurable (who runs your agents) is the gate for this profile's full launch — our data debt, not your ceiling."*

Practice-declined variant (F4): *"Your practice depth declined this quarter while output held — headline: Steady. If the reduction was deliberate, no action is needed; this dashboard describes, it does not prescribe."*

### 5.5 Phased rollout (answers the shipping-lens dissent) + pilot gates

- **Phase 1 (weeks 1–4, SQL-only):** Practice ladder (graded, FTE-scaled) + config tables + watermarks + cost strip + profiles + unmeasured panels + canaries.
- **Phase 2 (quarter 1):** Flow posterior bands, Q outlier floor, Momentum + headline, hysteresis, validation-loop harness (equivalence rules pre-registered), sensitivity analyses.
- **Phase 3 (as acquired):** opened_at bucketing; durability netting into Q; Experience survey; PELT changepoints; reviewer-credit and agent-consumer facets; agent-platform profile GA (gated on acquisition #1).
- **Pilot gates (S11, committed):** (i) measure headline week-over-week concordance on identical-behavior engineers; **if <0.8, coarsen the state vocabulary (e.g., to "on track"/"check flags") until it clears** — better honest-coarse than fake-fine; (ii) report the empirical P–M_prac correlation; (iii) run the §1.1 threshold sensitivity analysis before org-wide launch.

---

## 6. New signals worth acquiring — prioritized

| # | Signal | Cost | Lift | Why |
|---|---|---|---|---|
| 1 | Agent run → caller identity (agent_id, owner_id, caller) | Medium (platform telemetry; OTel GenAI/Langfuse already model it [primary/vendor]) | **High — now a GA precondition for the agent-platform profile (F8)** | The only path to a real output facet for agent-platform engineers; distinct repeat non-self consumers is the one gaming-resistant value proxy for agent work; also completes Stage 4's external-verification gate for builders (A2) |
| 2 | Per-PR file path lists (GitHub API, metadata only) | Low | **High** | Path exclusion (lockfiles/vendored/generated) + ≤21d self-rework netting; precondition for reinstating any LOC term in Momentum (S8) and for monorepo directory-grain Flow bands (F11) |
| 3 | **PR opened_at timestamps** *(new, from gaming red-team A4)* | Near-zero | **Med-High** | Switches Momentum bucketing to opened week — merge-timing banking then moves nothing |
| 4 | Revert linkage (branch `revert-<n>` / title regex), keyed to the reverted PR's author | Near-zero | **Med-High** | Both-legs revert zeroing; post-merge signal AI cannot saturate by iterating to green; reciprocal-pair canary (A5) |
| 5 | Review metadata (approvals, merged-without-review, turnaround) | Low-Med | **Med** | flag_review + reviewer "present" weeks (F5) + review-share Drifting suppression (F6) + future reviewer-credit facet; counters the documented cost-shift to reviewers (Faros +91% review time [vendor]); metadata counts only, flag for privacy review |
| 6 | **Incident/deploy-tooling merge labels + on-call rotation data** *(new, F15)* | Low | **Med** | Exempts break-glass merges from flag_review; replaces self-marked on-call weeks with system-of-record data |
| 7 | Additions/deletions split per PR | Low | **Med** | Required before any LOC term returns to Momentum (deletions at full weight — the only pro-refactorer numerator available, Atkinson [practitioner]) |
| 8 | Repo CI-job-count tier | Low | **Med-Low** | Cleaner Q priors than per-repo empirical fits |
| 9 | Quarterly pulse survey instrument | Medium | **Med** | Activates facet E (SPACE perceptual dimension); only voice for non-PR leverage until #1/#5 land |
| 10 | License/allowlist coverage + seat-assignment dates per user | Low | **Low-Med** | Fixes C2's denominator (F1) and new-hire breadth lag (F19) — partially HAVE via procurement |
| 11 | PTO/leave calendar + FTE fraction exposure | Low | **Low** | Sharpens active-week pro-rating and FTE scaling (F13/F16) — largely HAVE in HR |

---

## 7. Honest residual weaknesses

1. **Construct retreat, stated plainly:** this measures practice + trajectory, not amplification. Nobody can measure per-person amplification under these limits; any design claiming to is overclaiming (crit-construct §1.1). The dashboard must keep saying so.
2. **The direction-of-practice bet is unproven.** Practice→outcomes evidence is associational (DORA capabilities) and METR cuts against it for seniors. The validation loop tests the bet under equivalence logic, but is observational, self-selected, and — now honestly — publishes its own power instead of borrowing significance; in small cohorts it will say "cannot validate" for years. If rungs keep failing validation, the committed demotion path degrades Practice to a labeled descriptive profile.
3. **Quality floor is mostly promissory at launch** (a deliberately conservative CI outlier flag only, until acquisitions #2/#4 land). Shown in-product as "quality coverage: partial." Post-S3 it will fire rarely — by design; a floor that rarely binds is the honest price of not running a false-flag lottery.
4. **Invisible work stays invisible.** Review, mentoring, design, incident response, unmerged experimentation, and (until #1) agent outputs are acknowledged, not measured. Agent-platform engineers get a thinner profile in the interim — now with a GA gate attached so the interim has an owner and an end (F8).
5. **Practice theater residual** — bounded to the Practice stage (a private label), canaried (including timing entropy), excluded from the headline by the corroboration gate, and no longer misrepresented as defended by τ_c (S7). The ambient weak form ("open the tool twice a week") is indistinguishable from habitual adoption and is accepted at self-view stakes. An alternate-quarter ramp/coast sawtooth on the practice trend survives; its payoff is one private label per cycle (A3).
6. **Motivation risk & feedback latency:** for many engineers most quarters the headline will read "Steady", and real behavior change takes ~3 weeks to reach facets and ~2 months to reach the trajectory (stated in-UI, S10). "Drifting" fires only on confident evidence, so modest real declines will often read "Steady" (S1) — deliberate: the alternative is showing noise. Engagement load falls on facet levers and evidence notes, not score movement.
7. **Ceiling effect:** in a mandate-driven org most engineers may reach Stage 3 within ~a year; the refresh path is validation-driven rung revision plus new facets (reviewer credit, agent adoption-by-others). The "Sustained" state (S11) gives at-ceiling engineers a headline to keep, but a saturated ladder with a stalled refresh becomes a hygiene checkbox.
8. **Boundary and edge residuals (minor red-team findings folded here):** true-borderline engineers still experience some stage/band noise — disclosed via the boundary-distance cue rather than hidden (S6); published cutpoints invite edge camping — watched by canaries #1/#8, accepted as the price of interpretability (A9); log-scale compression still under-shows improvement at very low volumes — mitigated by the sparse state, not eliminated (S8); F-side cosmetic PR splitting is unfixable without diff content (A7); tool_class classification remains a lobbyable human decision — bounded by published criteria and rationale log (A9.3); self-marked on-call weeks are self-report — rate-limited, displayed, canaried, and replaceable by rotation data (#6) (F15).
9. **Cognitive compression risk:** five facets invite mental re-averaging into one number; framing and the absence of any numeric composite mitigate, not prevent.
10. **No deployed precedent for the assembly.** Every component has an evidence-backed track record; the assembly now also carries three adversarial passes and 44 logged dispositions — but ALP as a whole is unvalidated. Pilot with the §1.6 loop armed and the §5.5 concordance gate enforced from day one is mandatory.

---

## 8. Source key (all previously verified in Phase-A survey/repo files; tags repeated — no new external sources were introduced by the revision; all red-team exploit mechanics are [inference] per their own labeling)

METR RCT + 2026 update [independent] https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ · https://metr.org/blog/2026-02-24-uplift-update/ ; DORA 2024/2025 + ROI [primary] https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report · https://dora.dev/dora-report-2025/ ; SPACE [primary] https://queue.acm.org/detail.cfm?id=3454124 ; Fleming & Wallace [primary] https://dl.acm.org/doi/10.1145/5666.5673 ; Nagappan & Ball [independent] https://dl.acm.org/doi/10.1145/1062455.1062514 ; Brown/Cai/DasGupta [primary] (Wilson intervals); Cui et al. Mgmt Sci 2026 [independent-adjacent] https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 ; Jellyfish tokenmaxxing [vendor] https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/ ; Faros [vendor] https://www.faros.ai/blog/ai-software-engineering ; GitClear 2025 [vendor] https://gitclear-public.s3.us-west-2.amazonaws.com/GitClear-AI-Copilot-Code-Quality-2025.pdf ; Fortune Meta/Amazon leaderboards [independent] https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/ ; Bai et al. token variance [independent] https://arxiv.org/abs/2604.22750 ; Manheim & Garrabrant [primary] https://arxiv.org/abs/1803.04585 ; FinOps for AI [primary] https://www.finops.org/wg/finops-for-ai-overview/ ; DX Core 4 [vendor] https://getdx.com/research/measuring-developer-productivity-with-the-dx-core-4/ ; Stan partial pooling [primary] https://mc-stan.org/rstanarm/articles/pooling.html ; Robinson EB [practitioner] http://varianceexplained.org/r/empirical_bayes_baseball/ ; Beck [practitioner] https://newsletter.kentbeck.com/p/measuring-developer-productivity-440 ; repo evidence [primary code] repos/{devlake,fourkeys,middleware,faros,git-of-theseus,chaoss,discovered}.md ; internal red-team inputs: redteam/{gaming,fairness,statistics}.md. Verify-before-ship assumptions flagged inline: repo_id and merge timestamps in the PR warehouse; HR fte_fraction exposure; agent-platform run counts per the brief's stated assumption.
