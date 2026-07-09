# DESIGN CANDIDATE — "Trajectory" (Self-Relative Trend + Within-Person Uplift)

Sub-investigator design, 2026-07-08. Angle assigned: self-relative trajectory — each engineer scored against their OWN pre/low-AI baseline and recent trend; peers only as anonymous aggregate context. All external claims cite Phase-A-verified sources with tags; my own reasoning is labeled **[inference]**. Design constraints inherited from all four critiques (crit-construct, crit-fairness, crit-gaming, crit-stats): no spend denominator, no CI-clean multiplier, no percentile-averaged composite, no fake-neutral defaults, no hard eligibility cliffs, no level-only cross-function ranking.

---

## 1. Name & construct

**Name: "AI Trajectory" — leverage as within-person direction of travel, drift-corrected.** Under this design, "AI-assisted engineering leverage" is NOT a measured amplification factor (unmeasurable per person under the hard limits — crit-construct §1.1) and NOT a cross-person rank. It is defined as: *the direction and consistency of an engineer's own output, quality, and practice trajectory as their AI usage deepens, measured against their own rolling baseline, net of their cohort's secular trend.* Three subscores held in deliberate tension (SPACE requirement, https://queue.acm.org/detail.cfm?id=3454124 [primary]): (T) **Trajectory** — is my own output trend rising faster or slower than my function×level cohort's trend? (U) **Uplift contrast** — do my own AI-heavy weeks outperform my own AI-light weeks? (P) **Practice depth** — does my usage pattern match evidence-backed good practice? — plus a non-compensatory **Quality guardrail** (flags only) and a never-scored **Cost showback**. This is the only causal-ish design HAVE data supports without an RCT (survey/independent-evidence.md §5 [internal, evidence-anchored]; within-person period contrast is a strictly stronger version of Faros CE's person-period tool windows — repos/faros.md [primary code]). The critical addition over a naive self-trend: **double-differencing against the cohort trend**, because a raw self-vs-self trend is inflated by ambient workflow drift — "the metric cannot distinguish a better engineer from a later calendar date" (crit-gaming §3). Subtracting the cohort's median trend cancels shared secular inflation (difference-in-differences logic **[inference — standard econometric identification, applied not proven causal here]**).

---

## 2. Formula / signals

**Grain**: engineer-week on a zero-filled calendar spine (fourkeys/DevLake pattern — repos/fourkeys.md, repos/devlake.md [primary code]). All data through `T_ref = today − 14 days` (the usage-lag boundary; PR/CI data truncated to the same boundary so week classification never mixes lagged and unlagged sources — see §7).

```
-- 0. WEEK MASKING (away-detection, labeled heuristic)
away(i,t)   = (active_days_alltools(i,t) = 0 AND merged_prs(i,t) = 0)
usable(i,t) = NOT away(i,t) AND usage_data_present(i,t)   -- weeks missing usage rows are
                                                          -- excluded, never treated as AI-light

-- 1. OUTPUT SERIES (log scale; cohort-percentile winsorization, not fixed 2000)
cap_f       = P95 of per-PR lines-changed within function f, trailing 52 wks (sensitivity-checked ±)
wloc(i,t)   = Σ_pr min(lines_changed_pr, cap_f)
y_pr(i,t)   = log1p(merged_prs(i,t))          -- PRIMARY output series
y_loc(i,t)  = log1p(wloc(i,t))                -- SECONDARY (divergence check only, never scored alone)

-- 2. AI-INTENSITY (used ONLY relative to the person's own history — never cross-person)
A(i,t)      = log1p(dollars_alltools(i,t))    -- all tools: chat + CLI + agent platform

-- 3. BASELINE STATE
B(i)        = trailing 26 usable weeks ending at T_ref, truncated at last regime break (see §7)
if |B(i)| < 8:  state = BUILDING_BASELINE (show "k/8 weeks"; no T or U; P and guardrail still shown)

-- 4. SUBSCORE T — DRIFT-CORRECTED TRAJECTORY
T_raw(i)    = TheilSen slope of y_pr(i,t) over trailing 12 usable weeks (require ≥8, else INSUFFICIENT)
              -- Theil–Sen: median of pairwise slopes; robust to single spike weeks [reference method]
D(c)        = median over cohort c = function×level of T_raw(j), partial-pooled toward the
              function-level median when n_cell < 20 (hierarchical pooling per crit-stats Flaw 2 fix)
T_net(i)    = T_raw(i) − D(c(i))
CI_T        = moving-block bootstrap over weeks (blocks of 3, preserves autocorrelation) [inference]
DISPLAY     = both T_raw and T_net, as %/quarter ± CI, bucketed coarsely:
              {Declining < −10%/q | Stable −10..+10 | Improving > +10%/q}, bucket shown WITH interval;
              if CI spans two buckets → "no detectable trend yet"

-- 5. SUBSCORE U — WITHIN-PERSON UPLIFT CONTRAST ("association, not causation" label mandatory)
terciles    = own-history terciles of A(i,t) over B(i)          -- classification is self-relative
heavy(i)    = usable weeks with A in own top tercile;  light(i) = own bottom tercile
U(i)        = median(y_pr | heavy) − median(y_pr | light)       -- log points; display as ×factor
eligibility = |heavy| ≥ 5 AND |light| ≥ 5 AND IQR(A)/max(median(A),ε) ≥ 0.5
              else state = INSUFFICIENT_CONTRAST ("your usage is too uniform to estimate — that's
              fine; see Trajectory instead")
CI_U        = permutation test shuffling week labels within B(i); show p-band, never a bare point

-- 6. SUBSCORE P — PRACTICE DEPTH (fixed absolute rubric anchors; NEVER percentiles)
over trailing 8 usable weeks:
  coverage      = AI-active days / non-away days (cap 1.0)
  breadth       = # tool categories used ≥3 days (chat | coding CLI | agent platform | other)
  agentic_share = (CLI + platform sessions) / all sessions
  consistency   = fraction of weeks with ≥2 AI-active days
class = published rubric → {Emerging | Regular | Deep | Frontier}
        e.g. Deep = coverage ≥0.6 AND breadth ≥2 AND consistency ≥0.75  (anchors fixed & documented,
        revised at most annually with change-log — CHAOSS declared-parameter pattern, repos/chaoss.md)

-- 7. QUALITY GUARDRAIL (non-compensatory FLAGS, never a multiplier, never rewards)
ci_EB(i)    = (clean + α_f)/(checked + α_f + β_f), (α,β) fit per function (per repo-tier if repo id
              lands in warehouse — assumed-cheap, see §3); NO default when checked=0 →
              state = "insufficient CI data (n=k)" (DevLake N/A pattern, repos/devlake.md)
flag_ci     iff Wilson 95% upper bound of ci_EB < cohort prior mean   -- anomaly floor ONLY
flag_batch  iff TheilSen slope of own median-PR-size > +50%/quarter vs own baseline
              (DORA batch-inflation mechanism — survey/dora.md #4 [primary])
flag_diverge iff slope(y_loc) − slope(y_pr) > threshold   -- LOC rising with flat PRs = padding/churn
flag_integrity iff PR-size mass piles up just under cap_f, or share of CI-checked PRs falls
              (the two telltales inherited from crit-gaming §6 — kept as canaries)

-- 8. COST SHOWBACK (context, never scored, never in any subscore)
sparkline of dollars/active-day vs own trailing-26wk median, labeled "cost awareness" (FinOps
showback pattern — survey/roi-cost.md #4)

-- ASSEMBLY: NO composite number. Dashboard = one sentence (§6) + three tiles (T, U, P) + flags.
-- If the org later mandates a single number: combine z(T_net) and ordinal P on the z scale with
-- OECD/JRC robustness checks (crit-stats Flaw 5 fix) — recommended AGAINST; recorded for honesty.
```

**Cohort role (and only role)**: `function×level` cells, hierarchically pooled — used for (a) drift term D(c), (b) anonymous context band (cohort IQR of T_net shaded behind the personal trend line), (c) EB priors and winsorization caps. Never a rank, never a percentile shown as "you vs them."

**Work-profile routing**: if `merged_prs(trailing 12wk)=0 AND platform_sessions>0` → profile = PLATFORM: suppress T/U on PR output entirely (no zero, no bottom rank), show P + a self-trend of platform engagement (sessions, active days, run counts) labeled *engagement, not output*, plus the explicit state: **"Output for agent-platform work is not measurable with current signals"** (honest dead end per survey/agent-native.md #5). Hybrids see both lanes wherever each lane independently has data.

---

## 3. Data mapping

| Signal | Source dataset | HAVE / NEW |
|---|---|---|
| merged_prs(i,t), lines per PR | Warehouse PR table | HAVE (weekly bucketing needs merge timestamps — assumed present in a warehouse that "joins usage/PR/CI per user"; **stated assumption**, verify) |
| Per-PR CI clean flag, checked count | Warehouse CI signal | HAVE |
| Repo id per PR (for repo-tier EB priors, per-repo cap option) | PR table | Likely HAVE; if absent → cheap NEW (metadata only); design degrades gracefully to function-level priors |
| dollars, tokens, sessions, active days — per user/day/tool, ALL tools incl. agent platform | AI usage tables | HAVE (~2wk lag — handled by T_ref) |
| Tool-category mapping (chat/CLI/platform/other) | Static config over usage table | HAVE (trivial config table to maintain) |
| Agent-platform run counts | Platform logs | HAVE per brief's stated assumptions |
| function, level, team (cohorts + regime detection) | HR join | HAVE |
| Regime breaks (HR change events) | HR join | HAVE; changepoint fallback computed from HAVE series |
| Away/PTO ground truth | HR/calendar | NEW (optional; heuristic in §2 step 0 works without it, labeled heuristic) |
| Post-merge churn/rework ≤4wk, revert detection | Per-PR file lists / branch names via GitHub API | NEW — top acquisition; would replace flag_diverge with a real durability counterweight (survey/gitclear.md #1) |
| Agent run → caller identity / artifact linkage (adoption-by-others) | Platform telemetry (OTel GenAI/Langfuse-shaped) | NEW — the only path to a real PLATFORM-profile output lane (survey/agent-native.md #2) |
| Quarterly experience/friction survey (DevEx battery; facts, never self-estimated time savings) | New instrument | NEW — calibration layer for U's confounds (METR 40pp miscalibration — survey/space-devex.md #3) |

Everything in §2 runs on HAVE alone; the three NEW rows are upgrades, not dependencies.

---

## 4. Gaming resistance — top 5 attacks

1. **Sandbag the baseline** (go slow for 26 weeks so any later normalcy reads "Improving"). Response: baseline is *rolling*, so the sandbag decays and yesterday's inflation becomes tomorrow's higher bar — sandbagging buys one transient window, not a persistent score; T_net additionally requires beating the *cohort's* trend, and coarse buckets cap the payoff at one bucket step. Self-view-only removes most motive (crit-gaming §0). Residual: a determined new hire can still stage a ramp — accepted, low-stakes. **Penalizes if over-defended**: nobody; the defense is structural, not a threshold.
2. **Ambient-drift masquerade** (org-wide AI defaults inflate everyone's PR counts; raw self-trend reads "I improved" — crit-gaming §3, the documented killer of naive self-trend). Response: this is the design's core move — D(c) subtraction cancels drift common to the cohort; flag_diverge and flag_batch catch the LOC/batch flavors. Residual: drift that is function-specific but not org-wide is cancelled too (correct), drift unique to one person is not (indistinguishable from improvement — honest limit, §8).
3. **PR splitting to steepen own trend** (also the classic N1 quadruple-dip). Response: (i) splitting adopted org-wide cancels via D(c); (ii) individual splitting lowers own median PR size → flag_batch, and piles mass under cap_f → flag_integrity; (iii) Theil–Sen + median-of-weekly-buckets blunt single spike weeks; (iv) coarse trend buckets cap the reward. Residual: patient, moderate splitting sustained for 12 weeks still moves one bucket — telltales fire but can't prove intent. **Unfairly penalized by the defense**: legitimately small-batch (DORA-virtuous) engineers may trip flag_batch's inverse — flags are therefore *prompts to reflect*, never deductions.
4. **Uplift self-selection** (concentrate AI spend into weeks with naturally big output — or schedule easy work into heavy weeks — so U inflates). Response: U is quarantined — labeled "association, not causation," permutation-banded, feeds no composite, and the METR selection-effect caveat is displayed verbatim in-product; classification uses own terciles over 26 weeks, so staging requires sustained choreography of both spend and output. Residual: U is the least trustworthy subscore and is presented as such (§8). **Penalized**: uniformly-heavy adopters get INSUFFICIENT_CONTRAST — stated on-screen as expected, not as failure.
5. **Practice-depth theater** (one trivial chat session per day for coverage/consistency; touch a third tool category for breadth). Response: absolute rubric anchors mean theater buys at most one class step and cannot beat anyone (no ranking); breadth requires ≥3 days/category; sessions are capped per-day in coverage (binary AI-active day, so spamming sessions adds nothing); agentic_share resists chat-ping theater. Residual: a committed pretender can reach "Deep" — but a self-view class label with no comparative payoff makes this Austin's informational mode, where theater has no audience (survey/goodhart-design.md #5). **Penalized by the defense**: specialists whose one tool is genuinely right for their work sit at "Regular" breadth — rubric text explicitly says breadth is not a target for all roles.

Cross-cutting: the strongest single control is inherited — strictly self-view (crit-gaming §6). The two integrity canaries (cap pile-up, falling CI-checked share) are monitored org-side as *aggregate* health checks, never per-person alerts.

---

## 5. Fairness

**Cohort = function×level (6 functions × ~6 levels), hierarchically pooled, context-only.** Why: function and repo environment dominate cross-person variance (crit-fairness §2.1–2.2; Cui +27–39% junior vs +8–13% senior, https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 [independent-adjacent]; DORA amplifier thesis, https://dora.dev/dora-report-2025/ [primary]) — so cross-person comparison is structurally removed and cohorts survive only where they are valid: estimating *shared trends and priors*, which need cohort homogeneity of drift, not of level.

- **Function**: self-relative core means an ML engineer's few expensive iterations are compared to *their own* few expensive iterations. Function-specific PR norms affect only cap_f and D(c), both function-fitted. ML/experimentation unmerged work remains invisible to y_pr — mitigated only by P and honest labeling (§8).
- **Repo environment**: CI enters only as an EB anomaly floor with function/repo-tier priors — a heavy-CI repo lowers the *prior*, not the person (fixes the (1−f)^J arithmetic unfairness, crit-stats Flaw 3.3). Repo switches that shift output norms trip regime detection → baseline resets rather than reading as decline.
- **Seniority**: seniors are never ranked against juniors' steeper achievable gains; "Stable" is a first-class good state, displayed without red coloring. Review/mentoring/design work stays invisible (no signal exists — hard limit) but is no longer *punished*: flat output + Deep practice reads as a healthy senior profile. Honest text: "outputs measured cover only PR-shaped work."
- **Agent-platform engineers**: routed to PLATFORM profile — full Practice lane, engagement self-trend, explicit "output unmeasurable" state instead of the current near-zero score (fixes the categorical error, crit-fairness §2.4). No fake number is invented for them.
- **Part-time/leave/on-call/interns**: away-masking + usable-week logic + BUILDING/RECALIBRATING states replace on/off floors; per-active-day normalization inside P. Interns mostly live in BUILDING_BASELINE — honest, since 4-week internships genuinely cannot support trend inference (crit-stats Flaw 7).
- **Who this design still treats worst**: engineers with intrinsically lumpy cadence (release trains, big migrations) — 12-week Theil–Sen on lumpy series yields wide CIs and perpetual "no detectable trend." That is honest (noise displayed as noise) but unsatisfying (§8).

---

## 6. Interpretability

Dashboard headline sentence (real values interpolated):

> **"Compared with your own typical week over the last 6 months, your merged output is trending +8%/quarter — about +3%/quarter faster than the typical backend-L5 trend (±6%, so read this as 'roughly stable-to-improving'). Your AI-heavy weeks ship ~1.3× your AI-light weeks (association, not causation). Your practice depth is 'Deep' (AI on 78% of active days, 3 tool categories). No quality flags. Data through June 24; you are always compared to yourself — peers appear only as the shaded band."**

Every number traces to a visible chart: the personal weekly series with baseline median line, cohort band, and flag annotations. States are sentences, not zeros: "Building your baseline: 5/8 weeks," "Baseline resetting after your team change (3/8 weeks)," "Your usage is too uniform for an uplift estimate," "Output for agent-platform work isn't measurable yet — here's your practice and engagement view."

---

## 7. Cold start & churn

- **New hires**: BUILDING_BASELINE until 8 usable weeks (T needs 8 of trailing 12; U needs 26-week history to fill terciles, so U typically arrives last). From week ~2 they already see P and the cost showback; the cohort context band is shown labeled "context, not comparison." No fake neutral, no zero — right-censoring made explicit (git-of-theseus/K-M pattern, repos/git-of-theseus.md).
- **Regime changes** (the design's named hard case): a break is declared when (a) HR team/function/level change lands, OR (b) a changepoint detector (PELT, min segment 8 weeks) fires on the joint weekly (y_pr, A) series, OR (c) ≥4 consecutive away weeks end. On break: B(i) truncates at the break, T/U enter RECALIBRATING with a visible "n/8 weeks" progress state and the *old* trend chart preserved but greyed ("before your switch"); P and the guardrail continue uninterrupted (they are regime-robust). This converts the classic self-trend failure ("moved to a harder team, dashboard says I'm declining") into an explicit reset. **[inference — PELT is a standard changepoint method; threshold tuning needs a pilot]**
- **Low-activity months**: away weeks are masked from medians and slopes (not counted as zero output); partial weeks count as usable if any activity exists. Usable-week counting replaces every AND-ed floor of the current design — score *existence* is continuous-ish (states), not a step function (fixes crit-stats Flaw 6's 32% flicker / +27% self-trend selection bias — no month can "vanish" because low months are shown with wide intervals instead of suppressed).
- **2-week data lag**: everything is computed through T_ref = today−14d, PR/CI truncated to the same boundary; both baseline and current window lag equally, so contrasts are unbiased, just delayed **[inference]**. Weeks with missing usage rows are excluded from U's classification (never default to "AI-light" — the fake-neutral lesson, generalized). UI always prints "data through <date>."

---

## 8. Failure modes (honest)

1. **U is confounded and quarantined but still displayed**: engineers assign AI to AI-suited weeks (METR: 30–50% withheld unsuited tasks, https://metr.org/blog/2026-02-24-uplift-update/ [independent]) — U systematically flatters. Labeling mitigates misreading, not the bias.
2. **The deepest adopters get no U at all** (uniform heavy usage → INSUFFICIENT_CONTRAST). The design's most AI-native users lose one of three tiles permanently. Fallback is T only.
3. **Trend ≠ altitude**: a plateaued expert using AI superbly reads "Stable" — same word as a stagnating novice. P partially separates them; nothing in HAVE fully does.
4. **Statistical power is thin even at 12 weeks**: per-person weekly PR series are zero-inflated with SD>mean (Cui [independent-adjacent]); many engineers will see "no detectable trend" most quarters. Honest, but a dashboard that mostly says "nothing detectable" risks disengagement.
5. **D(c) noise propagates to everyone**: small function×level cells make the drift term jumpy; partial pooling helps but blurs genuinely different sub-cohort drifts.
6. **Person-specific gaming that mimics improvement is invisible** if it doesn't move batch size, LOC/PR divergence, or cap pile-up — e.g., slowly shifting effort from unmeasured work (review, mentoring) into PR-shaped work (Austin's reallocation, crit-gaming W3). The design reduces the *reward* (coarse buckets, no rank) but cannot detect it.
7. **Quality lane is weak until NEW signals land**: the guardrail is an anomaly floor plus proxies; durable-output measurement (churn/revert/review-burden) requires the §3 acquisitions. Meanwhile 25–45% of CI-passing AI code carries flaws the design cannot see (survey/independent-evidence.md #4).
8. **PLATFORM profile has no output lane** — engagement self-trend is participation, not value, and is labeled so. This design makes the gap honest, not closed.
9. **Slow feedback**: 26-week baselines and 12-week slopes mean behavior changes surface in ~1–2 quarters — appropriate statistically, frustrating motivationally.
10. **Task-mix drift within a regime** (same team, different work) is not detected; it will read as trend. Regime detection catches only large breaks.

---

## 9. Why this beats the current metric — vs the 7 known problems

1. **CI-clean weak proxy (P1)** → CI removed as multiplier entirely; survives only as an EB-shrunk, prior-anchored anomaly *flag* with Wilson intervals and an explicit insufficient-data state. No default 1.0 exists anywhere; a saturated signal can no longer distort anything because it no longer scales anything (crit-stats Flaw 3 fix, both layers).
2. **Narrow spend denominator / invisible agent work (P2)** → spend is never a denominator; ALL-tool usage including the agent platform feeds intensity A and practice P; agent-platform engineers get a dedicated profile with honest states instead of structural near-zero.
3. **Right-skewed, poorly discriminating per-dollar distribution (P3)** → no ratios at all; log-scale robust slopes with displayed CIs and coarse buckets; "top scorers decided by marginal CI noise" is impossible because there are no top scorers — there is no ranking.
4. **Brittle cold-start floors (P4)** → all AND-ed cliffs deleted; replaced by usable-week counting, BUILDING/RECALIBRATING/INSUFFICIENT states, and intervals — score presence degrades gracefully instead of flickering (2-vs-3-PR months change the CI width, not existence).
5. **Per-dollar penalizes expensive hard work (P5)** → spend appears only as self-relative intensity classification and a never-scored showback sparkline; spending more can never lower any subscore (it can only change which of your own weeks count as "heavy").
6. **Level-only cross-function cohort (P6)** → function×level cohorts, hierarchically pooled, demoted from ranking device to drift-correction + context band + priors; cross-person comparison is removed from the construct itself, so cohort misspecification degrades a *correction term*, not a person's rank.
7. **CI barely discriminates (P7)** → nothing asks CI to discriminate anymore; discrimination load moves to within-person contrasts where the engineer is their own control — and where the one documented killer of self-trend designs (ambient drift) is explicitly countered by cohort-trend subtraction, which no version of the current metric attempts.

Versus the critiques' cross-cutting demands: ≥3 dimensions in tension with a perceptual layer slot (SPACE), no percentile averaging or ratio means (Fleming–Wallace), EB shrinkage + censoring states (goodhart-design fixes #1), self-vs-self with anonymous bands (DORA anti-league-table), and the two integrity canaries carried forward as monitoring, not scoring.
