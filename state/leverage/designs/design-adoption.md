# DESIGN CANDIDATE — "Practice Maturity Ladder" (Adoption-Depth / Behavior-First)

Sub-investigator design, 2026-07-08. Angle: score the QUALITY of AI usage patterns (breadth, depth, consistency, agentic mix), validated against outcomes at cohort level — never rewarding raw consumption.
All external claims carry URL + tag, re-cited from Phase-A verified files; my own reasoning is labeled **[inference]**. Constraints honored: crit-construct.md (no spend denominator, no cross-sectional "amplification" claim), crit-stats.md (no percentile averaging, no hard floors, no saturated multipliers), crit-fairness.md (no level-only cohorts, agent-platform first-class), crit-gaming.md (bounded payoffs, integrity canaries, ambient-gaming awareness).

---

## 1. Name & construct

**Practice Maturity Ladder (PML).** Under this design, "AI-assisted engineering leverage" is *not* claimed to be a measured amplification factor (unmeasurable per person without attribution or a counterfactual — crit-construct.md §1.1; METR RCT −19% actual vs +20% believed, https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ [independent]). It is redefined as the only leverage-shaped thing the HAVE data can honestly measure: **the degree to which an engineer's observable AI-usage pattern matches a sustained, integrated, multi-mode practice — the behavior profile that leads outcome gains where gains occur** (DORA 2025's amplifier thesis: AI magnifies what's already there, so *practice* is the individual-controllable input, https://dora.dev/dora-report-2025/ [primary]; DX's Utilization→Impact→Cost journey scores adoption depth first, https://getdx.com/blog/how-to-implement-ai-measurement-framework/ [vendor]). The score is a **coarse, rule-based maturity stage (0–4)** built purely from usage-*shape* signals (consistency, breadth across tool classes, agentic-mode establishment, recency) — all bounded and saturating so raw tokens/dollars/sessions beyond low floors contribute nothing — plus a **quarterly cohort-level validation loop** that empirically tests whether higher rungs actually precede better delivery trends, and demotes/revises any rung that fails. Leverage here = *practice + validated trajectory*, exactly the replacement construct crit-construct.md §4 calls for.

---

## 2. Formula / signals

### 2.1 Primitives (per engineer u, per day d, per tool t)

```
usage(u,d,t) = {sessions, tokens, dollars, runs (agent platform only)}   -- HAVE
class(t) ∈ {CHAT, CODING_AGENT (agentic CLI), AGENT_PLATFORM}            -- config table (NEW, trivial)
window W = rolling 12 weeks (84 days), ending at T_max = today − 14d     -- absorbs the 2-week lag
weekly calendar spine, zero-filled (fourkeys pattern, repos/fourkeys.md)
```

**Meaningful-use day** (the anti-ping-farming unit; a *floor*, never a gradient):

```
meaningful(u,d,c) = 1  iff  sessions(u,d,c) ≥ 1
                     AND tokens(u,d,c) ≥ τ_c            -- τ_c = p10 of cohort nonzero token-days for class c,
                                                        --  recomputed quarterly, published on the dashboard
                     AND (c ≠ AGENT_PLATFORM OR runs(u,d) ≥ 1)
```
Tokens above τ_c add nothing. Dollars appear NOWHERE in scoring (crit-stats FLAW 1; Jellyfish: per-dollar ranks the most productive engineers last, https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/ [vendor]).

**Coding-active week** (pro-rating denominator; excludes PTO/leave without floors):

```
active_week(u,w) = 1 iff (any PR event by u in w) OR (any AI session by u in w)
AW = Σ_w active_week(u,w)   over W
```

### 2.2 Components (each bounded [0,1] by construction)

```
-- C1 CONSISTENCY: integrated into the working rhythm, not binge/abandon
C1 = ( Σ_w [ active_week(u,w) AND meaningful_days(u,w) ≥ 2 ] ) / AW

-- C2 BREADTH: established practice across tool classes (saturating per class)
est(c) = min(1, meaningful_days_in_class(u,c,W) / 5)        -- ≥5 days/quarter establishes a class
C2 = Σ_c est(c) / n_classes_available(u)                     -- denominator = classes the engineer's
                                                             --  org unit is licensed for (NEW small signal;
                                                             --  default 3 if unavailable)

-- C3 AGENTIC ESTABLISHMENT: sustained use of autonomous modes (not a chat penalty)
C3 = min(1, meaningful_days(u, CODING_AGENT ∪ AGENT_PLATFORM, W) / 10)

-- C4 RECENCY: practice is current
C4 = ( # of last 4 observable weeks with ≥1 meaningful day ) / min(4, AW)
```

### 2.3 Stage assembly — explicit rules, NOT a weighted composite, NOT a percentile

Combining correlated signals into one averaged number is the current metric's core statistical sin (Fleming & Wallace via crit-stats FLAW 5, https://dl.acm.org/doi/10.1145/5666.5673 [primary]); PML instead uses a breakpoint ladder (middleware/fourkeys coarse-bucket pattern, repos/middleware.md #6):

```
IF AW < 6:                      state = "INSUFFICIENT OBSERVATION" (show components + provisional
                                 stage with wide-uncertainty badge; never a zero, never absent)
Stage 0 EXPLORING:   C1 < 0.25
Stage 1 OCCASIONAL:  C1 ≥ 0.25
Stage 2 REGULAR:     C1 ≥ 0.50  AND max_c est(c) = 1            (one class established)
Stage 3 INTEGRATED:  C1 ≥ 0.75  AND C2 ≥ 2/n_classes_available  AND C4 ≥ 0.5
Stage 4 FRONTIER:    Stage 3    AND C3 = 1                       AND no integrity-canary flag
```
Everyone can be Stage 4 — non-zero-sum, non-stationary-cohort-proof (fixes crit-stats FLAW 2's percentile pathologies). Thresholds (0.25/0.5/0.75, 5 days, 10 days, τ_c) are published on the dashboard with a quarterly OECD/JRC-style sensitivity analysis (vary each ±30%, report % of engineers whose stage changes — goodhart-design.md §5).

### 2.4 Cohort layer (context + calibration only, never ranking)

Cohort = **function × level**, hierarchically pooled for sparse cells (partial pooling, https://mc-stan.org/rstanarm/articles/pooling.html [primary]). Used for exactly three things: (a) calibrating τ_c; (b) anonymous context ("41% of backend L5s are Stage 3+"); (c) the validation loop below. The stage itself is rule-based and cohort-independent, so no engineer's number moves because a colleague adopted faster.

### 2.5 The validation loop (what stops this from being a vanity adoption score)

The one OSS per-engineer AI score that exists (satomic/copilot-usage-advanced-dashboard) is pure usage-side with zero outcome term and is gameable by accept-everything spam (repos/discovered.md #1) — PML's differentiator is that rungs must keep earning their existence:

```
Quarterly, per function×level cohort (pooled):
  For engineers with a sustained stage transition (≥8 weeks at the new stage):
    Δoutcomes = within-person change, 12w-before vs 12w-after, on:
      - median weekly merged-PR count (throughput trend vs own baseline)
      - median PR size + share of PRs > cohort p90 lines (batch discipline — DORA's AI-instability
        mechanism, survey/dora.md #4)
      - CI-anomaly rate (floor signal only: share of PRs failing at head — NOT a multiplier)
      - [when acquired] same-file re-touch ≤14d, revert-branch rate
  Report effect + CI on the dashboard ("evidence: moderate, n=214 transitions").
  RULE: any rung whose transitions show null-or-negative Δoutcomes in a cohort for 2 consecutive
  quarters gets its criteria revised for that cohort, and the dashboard says so.
```
This is the Faros-CE period-contrast pattern (repos/faros.md, key takeaway) done within-person — the only causal-ish design available without an RCT (survey/independent-evidence.md #5). Outcomes NEVER enter the individual score (DORA 2024: individual gains and system outcomes diverge, −1.5% throughput/−7.2% stability per +25% adoption, https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report [primary]).

### 2.6 Companion panels (displayed beside the stage, never folded in)

1. **Self-trend sparklines** per component (own 12-week baseline).
2. **Cost showback** (private): own spend trend, log scale, labeled "cost awareness" — the FinOps-sanctioned use of spend (https://www.finops.org/wg/finops-for-ai-overview/ [primary]).
3. **Output context** (PR count/size vs own baseline) with the caption "context, not score."
4. **Experience pulse** (quarterly 4-item survey: friction facts, tool fit — never self-estimated time savings; METR's 40pp miscalibration, survey/space-devex.md #3). Satisfies SPACE's ≥1-perceptual-dimension rule (https://queue.acm.org/detail.cfm?id=3454124 [primary]).

---

## 3. Data mapping

| Signal | HAVE today (dataset) | NEW to collect |
|---|---|---|
| sessions/tokens/dollars/active-days per user·day·tool, ALL tools | HAVE — usage warehouse (brief HAVE #1) | — |
| Agent-platform run counts per user·day | HAVE — assumed per brief (stated assumption) | — |
| tool → class mapping (CHAT / CODING_AGENT / AGENT_PLATFORM) | — | NEW: static config table, ~hours of work, maintained quarterly |
| n_classes_available per org unit (license/allowlist coverage) | — | NEW: small join to procurement/seat table; default 3 if absent |
| Merged-PR count & lines (validation + context panels only) | HAVE — PR dataset | — |
| Per-PR CI pass flag (anomaly floor + validation outcome only) | HAVE — CI dataset | — |
| Function, level (cohorts) | HAVE — HR dataset | — |
| τ_c calibration inputs | HAVE — derived from usage warehouse | — |
| Same-file re-touch ≤14d churn (validation outcome upgrade) | — | NEW: per-PR file lists via GitHub API (cheap; top acquisition per survey/gitclear.md #1) |
| Revert detection (validation outcome upgrade) | — | NEW: PR head-branch names + `^revert-\d+` regex (zero-cost metadata; repos/discovered.md #2) |
| Experience pulse (4 Likert items, quarterly) | — | NEW: survey instrument; DevEx persona-segmented (https://queue.acm.org/detail.cfm?id=3595878 [primary]) |
| Agent run → caller identity (future Stage-4+ rung for agent builders: adoption-by-others) | — | NEW: agent_id + owner_id + caller per run; OTel GenAI/Langfuse already model this (survey/agent-native.md #2) |

Score is fully shippable on HAVE + the two trivial config tables; all other NEW items upgrade the *validation* layer, not the score.

---

## 4. Gaming resistance — top 5 attacks

Threat model per crit-gaming.md §0: self-view-only bounds stakes but perceived stakes still drive gaming (Amazon KiroRank disbelief, https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/ [independent]). PML's structural defense: **every scored signal saturates, so every attack has a hard payoff ceiling — a stage label, no rank, no tail to climb.**

1. **Streak farming** (one trivial ping per day to mint meaningful days — the attack that kills satomic's consistency bonus and DX active-day cohorts). Response: τ_c token floor per class; C1 needs ≥2 meaningful days/week sustained over ~9 of 12 weeks; payoff caps at the rung. Residual: scripted τ-clearing sessions — integrity canary #3 (near-zero variance in daily tokens/session counts) flags the pattern; and the honest cost of faking it (~2 real-ish sessions/wk) approaches the cost of just using the tool. **[inference]**
2. **Class-checkbox theater** (touch the agent platform 5 times for breadth). Response: est(c) needs 5 *meaningful* days incl. ≥1 actual run/day for AGENT_PLATFORM; C3 needs 10 days. Bounded payoff (one rung). Residual: 10 no-op agent runs is real residual risk — Meta/Amazon precedent says idle-agent fabrication happens [independent, above]; mitigation is the canary (runs with ~zero tokens) + future caller-identity signal.
3. **Token/spend maximization** (Claudeonomics-style). Structurally impossible to profit: tokens beyond τ_c and all dollars contribute zero. PML is invariant to consumption by construction — the design's central property.
4. **Sandbagging / window shaping** (hide bad months, bunch activity at boundaries — crit-stats FLAW 6). No AND-gate floors exist to duck under; denominators are observed coding-active weeks, so absence shrinks the denominator too; 12-week rolling windows with weekly spines make boundary timing nearly worthless (~1/12 of weight moves per week).
5. **Austin effort reallocation** (crit-gaming W3 — the current metric's worst ambient attack). PML demands zero PR-shaped output, so it exerts no pressure to abandon review, mentoring, incident response, or agent-building; satisfying Stage 3 honestly costs ~2 AI-use days/week — cheaper than gaming it. The reallocation gradient the current metric creates is simply absent. **[inference]**

Carried integrity canaries (crit-gaming §6): PR-size pile-up under caps and CI-checked-share remain monitored org-side (they now protect the *validation* outcomes), plus new canary #3 above.

**Per-metric gaming/penalty table (required):**

| Metric | (a) Gamed how | (b) Unfairly penalizes |
|---|---|---|
| C1 consistency | daily trivial sessions (τ floor + 2-day/wk rule blunt it) | batch deep-workers who binge AI fortnightly (flagged in §8) |
| C2 breadth | checkbox days per class (5-meaningful-day bar) | single-tool specialists whose one tool is right; allowlist-restricted teams (n_classes_available denominator repairs) |
| C3 agentic establishment | no-op agent runs / cron self-invocation (canary; run+token floor) | chat-appropriate roles (careful reviewers, security) — see §5/§8 |
| C4 recency | one ping in each of last 4 weeks (bounded, worth ≤1 rung) | leave-takers (min(4,AW) denominator repairs) |
| τ_c floors | scripted floor-clearing (variance canary) | terse prompters/efficient users near p10 (floor is low by design) |
| Validation outcomes (PR size, throughput trend, CI floor) | individual can't move cohort stats; collective drift possible | (aggregate-only, so no individual penalty) — cohort drift monitored vs org baseline |
| Experience pulse | inflate if believed stakes-bearing (keep self-view, facts-not-estimates) | non-respondents, honest pessimists — never scored |

---

## 5. Fairness

**Cohort = function × level, hierarchically pooled** — because achievable AI benefit and usage norms are function- and environment-conditional (juniors +27–39% vs seniors +8–13%, Cui et al., https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 [independent-adjacent]; DORA amplifier thesis [primary]), and because level-only cross-function cohorts were the current metric's problem #6. But the deeper fairness move is that **the cohort never ranks anyone**: stage rules are absolute, so cross-function unfairness can only enter through thresholds (τ_c is cohort-calibrated; day-count bars are function-agnostic by intent since they describe *rhythm*, not volume).

- **Function**: PR/line/spend norms are irrelevant to the score (not inputs). ML engineers with expensive loops and few merged PRs are scored on practice rhythm they fully control. **[inference]**
- **Repo environment**: CI config, monorepo granularity, squash-merge — all absent from scoring (CI is a floor/validation signal only), eliminating the (1−f)^J repo lottery (crit-stats FLAW 3.3).
- **Seniority**: seniors doing review/design/mentoring are no longer invisible — the score never demanded merged PRs. A correctly-selective senior who uses chat deliberately 3 days/week reaches Stage 3. Stage 4's agentic bar is the residual seniority/role risk (§8.2).
- **Agent-platform engineers**: the design's best case — they flip from categorically mis-scored (crit-fairness §2.4: "total" magnitude error) to **first-class**: their sessions/active-days/runs are score inputs, AGENT_PLATFORM is a class, C3 explicitly values their mode. Honesty requirement: their panel carries "outcome validation pending — no output surface exists for agent work yet" until run→caller signals land (survey/agent-native.md #5).
- **Part-time / leave / on-call / interns**: coding-active-week denominators pro-rate everything; no floors; INSUFFICIENT OBSERVATION is a labeled state, never a zero or an absence (DevLake `_is_collected_data` N/A pattern, repos/devlake.md #4; K-M right-censoring, repos/git-of-theseus.md #1).
- **Contractors**: if unmetered/unlicensed, n_classes_available reflects it; if usage is genuinely invisible, they get INSUFFICIENT OBSERVATION, not a bad score. Audit spend-coverage first (crit-fairness §2.5).

---

## 6. Interpretability

The dashboard sentence:

> **"You're at Stage 3 — Integrated: you used AI meaningfully in 10 of your 12 working weeks, across 2 of the 3 tool classes available to you, including last week. Stage 4 needs an established agentic practice — 10+ days this quarter on the coding agent or agent platform (you have 4). Cohort context: 38% of backend L5s are Stage 3+. Evidence note: backend engineers who sustained Stage 4 shipped ~15% smaller PRs at unchanged throughput (moderate evidence, n=214 transitions, updated Apr 2026)."**

Every clause is a count the engineer can verify against their own calendar; the rung gap is an explicit, achievable action; the evidence line is honest about strength. No engineer ever needs to ask "why is my score 61?" — there is no 61 (crit-stats FLAW 5's uninterpretability).

---

## 7. Cold start & churn

- **New hires**: components are computable from week 1 (fractions over observed weeks). Until AW ≥ 6, state = INSUFFICIENT OBSERVATION with provisional stage + wide-uncertainty badge — no cliff, no fake neutral, no absence. Contrast: current design's 4 AND-gates give a Poisson(3.5) engineer a 32% chance of no score monthly (crit-stats FLAW 6).
- **Low-activity months** (PTO, on-call, crunch elsewhere): non-active weeks drop out of every denominator; a 3-week vacation cannot demote anyone. Stage transitions require sustained (multi-week) pattern changes, so single-week noise moves nothing — coarse rungs are deliberately insensitive (fourkeys buckets rationale, repos/fourkeys.md #2).
- **2-week data lag**: window is pinned to T−14d and the dashboard prints "data through <date>"; because rungs move on ≥multi-week patterns, a 2-week lag delays stage changes without ever misstating them. No nowcasting, no backfill-presented-as-data (copilot-viewer anti-pattern, repos/copilot-viewer.md #4).
- **Score churn**: expected month-to-month stage volatility is near zero for stable behavior — by design the ladder trades resolution for reliability, directly answering the ~0.3 test-retest reliability of the current 4-week score (crit-stats FLAW 7). Component sparklines carry the fine-grained movement instead.

---

## 8. Failure modes (honest)

1. **The construct bet can be wrong.** Usage maturity may not be a leading indicator of value for everyone — METR showed experts *slowed down* while feeling faster [independent, above]; the 2026 update calls even its reversal "only very weak evidence" (https://metr.org/blog/2026-02-24-uplift-update/ [independent]). The validation loop tests the bet but is observational (self-selection into stage transitions; task-mix changes confound within-person contrasts). If rungs keep failing validation, PML must degrade gracefully into a labeled *descriptive adoption profile* — the design commits to that demotion path, but a demoted score is a weaker product.
2. **Stage 4 hard-codes a direction** (agentic modes > chat) that evidence has not settled; for some roles (security review, API design) chat-only may be optimal. Cohort-conditional rung revision mitigates; it cannot fully repair a role for which the whole ladder shape is wrong. Batch deep-workers (2 intense AI days/week, C1-hostile rhythm) are a second honest casualty.
3. **Behavior-side blindness**: PML says nothing about the *quality of output* of any individual. A Stage-4 engineer can ship churn and verbose AI code all quarter (GitClear: churn up to 9x for heavy AI users, GitKraken×GitClear Mar-2026 via survey/gitclear.md [vendor]) — the design's quality protection lives entirely in aggregate validation + acquired counterweights, not in the personal score. This is deliberate (no honest per-person quality signal exists in HAVE data) but must be stated on the dashboard.
4. **Ceiling effect**: in a mandate-driven org, most engineers may reach Stage 3–4 within ~a year, and PML saturates the way CI-clean-rate did. PML is a *transitional instrument* for the adoption era; the refresh path (validation-driven rung revision, adoption-by-others rungs for agent builders) is designed in, but a saturated ladder with a stalled refresh becomes a hygiene checkbox. **[inference]**
5. **Threshold arbitrariness**: 5 days, 10 days, 0.25/0.5/0.75, τ_c = p10 are judgment calls dressed as constants; the published sensitivity analysis exposes but does not remove this.
6. **Taxonomy drift**: tool classes are blurring (chat apps spawn agents); misclassification silently shifts C2/C3. The config table needs a named owner and quarterly review.
7. **Validation power**: within-person transition effects on PR outcomes need large n (Cui was underpowered near n≈5,000, survey/copilot-studies.md #3); small function×level cohorts will show wide CIs for years — the dashboard must show "insufficient evidence" rather than borrow significance, and partial pooling only partly helps.

---

## 9. Why this beats the current metric — vs the 7 known problems

1. **CI-clean-rate weak/unfair quality proxy** → removed from scoring entirely; survives only as an anomaly *floor* and a validation outcome. No transform was going to fix a saturated signal (crit-stats FLAW 3 verdict).
2. **Narrow spend denominator; agent-platform work invisible** → there is no denominator; ALL tools including the agent platform are first-class score inputs, and agent-platform engineers move from near-zero scores to the design's best-served cohort (score from behavior they demonstrably perform; honest "validation pending" label).
3. **Right-skewed per-dollar ratio, poor discrimination, top ranks = CI noise** → no ratio, no percentile, no tail: bounded components on saturating rules; the top of the scale is a rung anyone can reach, so nothing selects on noise (kills regressional Goodhart, crit-stats FLAW 2).
4. **Brittle cold-start floors** → zero AND-gates; fraction-based components over observed weeks, explicit INSUFFICIENT OBSERVATION state, provisional stages with uncertainty badges. 2-vs-3-PR flicker is structurally impossible (PRs aren't score inputs).
5. **Per-dollar penalizes spending more on harder work** → dollars/tokens cannot lower (or raise) the score; spend appears only as a private cost-awareness sparkline. The Jellyfish inversion (most productive engineers ranked last [vendor, above]) cannot occur.
6. **Level-only cross-function cohort unfairness** → cohorts are function×level with partial pooling AND demoted to calibration/context/validation duty; since the score isn't a rank, cohort composition can't move anyone's number (fixes the zero-sum, adoption-wave non-stationarity of percentiles).
7. **CI 1.0 clustering / fake-neutral defaults** → no defaults anywhere; missing data is a labeled state, never 1.0, never 0 (DevLake N/A pattern). The evidence-inversion (fewer data ⇒ better score) is eliminated.

Cross-cutting wins: score-raising actions are all genuinely desirable behaviors (use AI regularly, try the agentic tools, keep practice current) — restoring P1/P2/P3 controllability-improvability-stability (crit-construct §3); the two statistical anti-patterns (percentile averaging, ratio-mean) are gone; SPACE's perceptual dimension enters via the pulse survey; and the design is honest about what it is not measuring (individual output quality, true amplification), which the brief's task #7 demands.
