# DESIGN CANDIDATE — "AI Leverage Profile" (ALP): five non-compensatory facets, trajectory headline, no composite number

Sub-investigator design, 2026-07-08. Angle assigned: a leverage PROFILE of 3–5 orthogonal subscores with an explicit interpretability story. Grounded in BRIEF.md, all four crit-*.md files, and Phase-A survey/repo evidence; external claims cited with URL + tag; my own reasoning labeled **[inference]**. Constraint honored throughout: this design contains **no spend denominator, no CI-clean multiplier, no percentile averaging, no hard cold-start floors, no level-only cohort** — the five components the critiques ruled beyond repair.

---

## 1. Name & construct

**AI Leverage Profile (ALP)** — five facets: **Practice · Flow · Quality Floor · Momentum · Experience**, plus a non-scored **Cost Awareness** context strip. Under this design, "leverage" is explicitly NOT a measured amplification factor (the counterfactual is unmeasurable per person — METR RCT/update, https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ · https://metr.org/blog/2026-02-24-uplift-update/ [independent]; crit-construct §1.1). Instead, leverage is operationalized as **practice + trajectory**: (i) the degree to which an engineer's observable AI-usage pattern matches practices the evidence associates with durable, system-friendly output (adoption depth/consistency, small-batch discipline — DORA's own AI capability, https://dora.dev/guides/ [primary]); and (ii) the **direction of the engineer's own outcome and practice trend against their own trailing baseline, cohort-detrended**, which is the only causal-ish per-person design available without an RCT (within-person contrasts; Faros-CE period-contrast precedent, repos/faros.md [primary code]). Quality appears only as a **floor/veto** (anomaly flags), never as a multiplier — because the available quality signal (CI-clean) is saturated and repo-confounded (BRIEF problems #1/#7; Cui et al.: build volume +38%, success rate flat-to-negative, https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 [independent]). The facets are deliberately in tension and never averaged (SPACE dimensionality + tension rules, https://queue.acm.org/detail.cfm?id=3454124 [primary]; Fleming & Wallace on ratio averaging, https://dl.acm.org/doi/10.1145/5666.5673 [primary]). The headline is a **categorical trajectory sentence**, not a number: a 0–100 headline cannot be made non-misleading at individual×window grain (test-retest reliability ≈0.3, percentile zero-sum/non-stationarity — crit-stats Flaws 2/7), so ALP does not ship one. **[inference, evidence-anchored]**

---

## 2. Formula / signals

### 2.0 Global machinery

```
WINDOW      : rolling 12 weeks (84d), weekly buckets on a zero-filled calendar spine
              (fourkeys/DevLake pattern — repos/fourkeys.md, repos/devlake.md [primary code])
AS_OF       : today − 14d  (absorbs the 2-week usage-table lag; UI shows "data through <AS_OF>")
REFRESH     : weekly
COHORT c(e) : function(e) × level(e), hierarchical partial pooling
              engineer ⊂ function×level ⊂ function ⊂ org  (Stan/Efron-Morris pattern,
              https://mc-stan.org/rstanarm/articles/pooling.html [primary]) — no hard cells,
              small cells shrink toward function then org priors
PROFILE p(e): data-driven work-mode profile, score-neutral, selects WHICH facets are scored:
  pr_weeks    = # weeks in window with ≥1 merged PR
  agent_share = agent-platform tokens / total AI tokens in window
  p = 'agent-platform'  if pr_weeks ≤ 1 AND agent_share ≥ 0.5
      'hybrid'          if pr_weeks ≥ 2 AND agent_share ≥ 0.3
      'pr-shaped'       otherwise
DISPLAY     : each scored facet → posterior cohort quantile → 5 named bands
              (Emerging / Developing / Established / Deep / Leading) + uncertainty whisker;
              band changes require crossing the edge by >5 quantile pts for 2 consecutive
              weeks (hysteresis — anti-flicker) [inference]
NEVER       : facets averaged into one number; spend used as divisor; missing data scored
              as neutral/zero — always explicit 'unmeasured' / 'building baseline' states
              (DevLake `_is_collected_data` N/A pattern; K-M right-censoring —
              repos/devlake.md, repos/git-of-theseus.md [primary code])
```

### 2.1 Facet P — Practice (adoption depth & consistency) — scored for ALL profiles

```sql
-- meaningful session: anti-theater floor
meaningful(s) := tokens(s) >= P10(session tokens | tool_category, org, trailing 26w)

-- weekly components (per engineer e, week w):
ai_days(e,w)      = COUNT(DISTINCT day WITH >=1 meaningful session)
active_days(e,w)  = COUNT(DISTINCT day WITH any usage OR PR event)   -- exposure normalizer,
                                                                     -- NOT an investment denominator
consistency(e,w)  = ai_days / GREATEST(active_days, 1)
breadth(e,w)      = COUNT(DISTINCT tool_category WITH meaningful use)   -- {chat, CLI, agent, other}, cap 4
agentic(e,w)      = log1p( LEAST(meaningful CLI+agent sessions, P95(cohort)) )  -- winsorize at cohort pctile

-- assembly: per-component 12-week medians → z inside hierarchical model → mean of z's
P_z(e) = mean_z( median_w consistency, median_w breadth, median_w agentic | partial pooling in c(e) )
P_band = 5-band posterior quantile of P_z within c(e), with interval
```
Consistency credit is **capped** (satomic capped-bonus pattern, repos/discovered.md [primary code]); components are individual-controllable behaviors per DORA's "score behaviors, show outcomes" implication (survey/dora.md [primary]).

### 2.2 Facet F — Flow (batch-size discipline) — scored for pr-shaped & hybrid only

```sql
-- per merged PR i: repo-normalized size band (repo norms from ALL engineers, trailing 26w)
in_band(i) := 10 <= lines_changed(i) <= P75(lines per PR | repo(i))
-- 10-line floor: micro-PR confetti earns nothing; upper bound = repo property, not a
-- published org-wide constant (fixes the 2000-line gaming boundary, crit-stats Flaw 4)

F_raw(e)  = EB-shrunk share: (in_band_PRs + α_f) / (PRs + α_f + β_f), (α_f,β_f) fit per function cohort
F_band    = 5-band posterior quantile within c(e); weekly PR cadence shown as context sparkline, NOT scored
```
Rationale: small batches are DORA's causal story for AI-era instability (survey/dora.md finding #4 [primary]); F **inverts** the current lines numerator — size stops being rewarded, discipline is. Volume gate: F is a share, so more PRs never raise it.

### 2.3 Facet Q — Quality Floor (state, not score) — scored where observable

```sql
-- CI anomaly floor (HAVE): EB Beta-binomial per repo-CI-tier prior
rate_EB(e) = (clean + α_r) / (checked + α_r + β_r)      -- (α_r,β_r) per repo CI-job-count decile
flag_CI    := Wilson95_upper(clean, checked) < median(rate_EB | repo tier)
             -- flags only "confidently below repo-tier norm"; saturation at ~1.0 is harmless
             -- because Q never boosts anything (Brown/Cai/DasGupta on Wilson intervals,
             -- http://www-stat.wharton.upenn.edu/~lbrown/Papers/2001a%20Interval%20estimation%20for%20a%20binomial%20proportion%20(with%20T.%20T.%20Cai%20and%20A.%20DasGupta).pdf [primary])

-- NEW signals, added as acquired (each: EB-shrunk, repo-cohort-RELATIVE per Nagappan & Ball,
-- https://dl.acm.org/doi/10.1145/1062455.1062514 [independent]):
flag_revert := revert-linked-PR rate confidently above repo norm      (branch/title regex — repos/middleware.md)
flag_churn  := own-file re-touch rate ≤21d confidently above repo norm (needs per-PR file lists)
flag_review := merged-without-review share confidently above repo norm (review metadata)

Q_state(e) ∈ { 'clear', 'attention: [list of flags]', 'unmeasured' }
-- Q can only CAP the headline; it never multiplies or boosts. Target is 'not an outlier',
-- never 'zero churn' (some churn is healthy — survey/gitclear.md [vendor]).
```

### 2.4 Facet M — Momentum (within-person, cohort-detrended) — the headline driver

```sql
-- own weekly outcome series (pr-shaped/hybrid):
y(e,w) = log1p(PRs(e,w)) + log1p( LEAST(LOC(e,w), P95(weekly LOC | c(e))) )
-- agent-platform profile: y := practice index p(e,w) until an output signal exists (honest substitution, labeled)

-- robust own-trend minus cohort trend (kills ambient workflow-drift inflation, crit-gaming §3):
M_out(e)  = TheilSen_w( y(e,w) ) − median_{e'∈c(e)} TheilSen_w( y(e',w) )
M_prac(e) = TheilSen_w( p(e,w) ) − median_{e'∈c(e)} TheilSen_w( p(e',w) )

-- descriptive within-person contrast (the "leverage-ish" readout; Faros-CE precedent):
heavy = own weeks in top tercile of meaningful AI use; light = bottom tercile
Δ(e)  = median( y | heavy ) − median( y | light ), bootstrap 90% CI
suppress Δ if observed_weeks < 9 OR var(own usage) < floor  -- else it's noise theater

-- HEADLINE (sentence + glyph, never a number):
if observed_weeks < 8                      → 'Building baseline (k of 8 weeks)'
elif Q_state = 'attention'                 → 'Mixed — check quality flags'
elif M_prac > 0 AND (M_out ≥ 0 OR Δ CI > 0) → 'Improving'
elif M_out > 0 AND M_prac ≤ 0              → 'Steady (output up)'   -- VOLUME GATE: output volume
                                                                    -- alone never yields 'Improving'
elif |M_prac| , |M_out| small              → 'Steady'
else                                        → 'Drifting — practice or output trending down'
```

### 2.5 Facet E — Experience pulse (perceptual layer; SPACE-mandated) — all profiles

Quarterly, ~9 items, self-view results vs anonymous function-cohort bands (DXI per-driver pattern, https://getdx.com/research/measuring-developer-productivity-with-the-dx-core-4/ [vendor]). **Experience/friction facts only — never self-estimated time savings** (METR: believed +20%, measured −19% — 40pp miscalibration [independent]). Items: share of AI-assisted changes needing major rework before merge; friction sources (context assembly, review wait, flaky CI); tool-fit; for agent-platform builders: who consumes your agents + artifact links (CHAOSS Contribution Attribution structured self-report pattern, repos/chaoss.md [primary]).

### 2.6 Cost Awareness strip — displayed, NEVER scored

Own 13-week spend sparkline + tool-category mix + anonymous cohort IQR band; labeled "cost awareness," FinOps showback pattern (https://www.finops.org/wg/finops-for-ai-overview/ [primary]). Optional descriptive elasticity `Δlog(output) vs Δlog(spend)` on own trend only. Spend is context because it is noise as an individual signal: 30x same-task token variance, cost only weakly tracks difficulty (https://arxiv.org/abs/2604.22750 [independent]); per-dollar ranks invert productivity (Jellyfish $0.28→$89.32/PR, https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/ [vendor]).

### 2.7 Integrity canaries (ops-only, never shown as scores)

Keep monitoring the two telltales the old metric emitted (crit-gaming §6): PR-size pile-up just under band edges, and falling share of CI-checked PRs; add session-token distributions near the meaningful-session floor. **[inference]**

---

## 3. Data mapping

| Signal | Feeds | HAVE today (dataset) | NEW to collect |
|---|---|---|---|
| Per-user/day/tool dollars, tokens, sessions, active days (ALL tools incl. agent platform) | P, M, Cost strip, profile p(e) | HAVE — usage warehouse | — |
| Agent-platform run counts | P (agentic), profile | HAVE (per BRIEF stated assumption) | — |
| Merged PR count + per-PR lines | F, M | HAVE — PR dataset (per-PR lines implied by current 2000-line winsorization) | — |
| Repo id per PR (for repo-normalized size bands & priors) | F, Q | Likely HAVE in PR metadata; **verify** | cheap backfill via GitHub API if absent |
| Per-PR CI pass/clean flag | Q (anomaly floor) | HAVE | — |
| Role / level / function / team | cohorts c(e) | HAVE — HR | — |
| Repo CI-job-count tier | Q priors | — | NEW-cheap: CI config/API metadata |
| Revert detection (branch `revert-<pr#>` / title regex) | Q | — | NEW-cheap: PR branch/title metadata (repos/middleware.md pattern) |
| Per-PR file lists → ≤21d self re-touch churn | Q (best quality counterweight) | — | NEW: GitHub API, metadata only (survey/gitclear.md top acquisition) |
| Review metadata (approvals, merged-without-review, review turnaround) | Q; future reviewer-credit facet | — | NEW: GitHub API |
| Agent run: agent_id + owner_id + caller identity | future agent-output facet (repeat adoption by non-self consumers) | — | NEW: agent-platform logs (OTel GenAI/Langfuse already model it — survey/agent-native.md) |
| Quarterly pulse survey | E | — | NEW: survey instrument |
| PTO/part-time calendar (optional exposure refinement) | active-day pro-rating | maybe HR | NEW-optional |

Launchable **today** on HAVE data: P, F, M, Q(CI-floor only), Cost strip, profiles, cohorts. E and Q's real counterweights are the first two acquisitions.

---

## 4. Gaming resistance — top 5 attacks and responses

Threat model: self-view-only is the strongest single control (Austin informational measurement; Beck self-measurement — https://newsletter.kentbeck.com/p/measuring-developer-productivity-440 [practitioner]) but stakes are *perceived*, not declared (Amazon KiroRank gamed despite "won't inform reviews" — https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/ [independent]), so ALP is designed as if it will leak.

1. **Denominator suppression (the old metric's #1 exploit, ~40x lever).** *Response: structurally eliminated* — no spend divisor exists anywhere; spend is an unscored context strip. Routing usage to chat/agent/personal tools changes tool-mix display only; the agent platform is inside P's universe, so shifting to it is not an escape hatch (profiles are score-neutral).
2. **Micro-PR splitting / trivia farming / cap-aware chunking.** *Response:* F is a **share** with a 10-line floor and a repo-relative (unpublished, moving) upper band — splitting a 6,000-line PR into 3×2,000 fails a repo whose P75 is 400; confetti PRs <10 lines lower the in-band share; M uses log-scaled **medians of weekly buckets** (bursts barely move medians); the volume gate means PR-count inflation can at best produce "Steady (output up)," never "Improving." Residual: splitting into moderately-sized in-band PRs remains partially rewarded — mitigated by it being… actual small-batch practice (the gamed behavior and the desired behavior converge — crit-gaming N1's own observation). Canary: size pile-up at band edges.
3. **Iterate-to-green CI saturation (highest-prevalence ambient exploit).** *Response:* CI is demoted to an anomaly floor that can only flag "confidently below repo-tier norm" — a saturated rate at ~1.0 flags no one and boosts no one; the signal's death is priced in, and quality discrimination is reassigned to churn/revert/review flags as they are acquired. Nothing to game because nothing pays out.
4. **Consistency/breadth theater (one trivial session per day; touch every tool once).** *Response:* meaningful-session token floor (cohort P10 per tool category); capped breadth (4 categories) and capped consistency credit; banded display (marginal theater moves a z-score, rarely a band); P is one facet among five with no composite to inflate. Residual: a determined engineer can run one real session daily — which is indistinguishable from, and behaviorally identical to, habitual adoption; acceptable at self-view stakes. **[inference]**
5. **Momentum manipulation (sandbag early baseline; bank work then release; ride org-wide drift).** *Response:* cohort-detrending subtracts the cohort's median trend, so calendar drift and org-wide workflow inflation cancel (fixes crit-gaming §3's "the metric cannot distinguish a better engineer from a later calendar date"); Theil–Sen slopes on 12 weekly medians are robust to a few manufactured weeks; suppression states remove the "hide a bad month below a floor" play because there are no floors — every week counts, low-evidence weeks just widen intervals. Residual: a patient gamer sandbagging for a full quarter beats it; cost is high, payoff is a private glyph. **[inference]**

Per-signal gaming/penalty micro-table (every metric this design touches):

| Signal | (a) Gamed by | (b) Unfairly penalizes |
|---|---|---|
| P: consistency | daily token-floor-clearing session | PTO/part-time (active-day pro-rating mitigates); tool-restricted teams |
| P: breadth | one meaningful session per category | specialists whose single tool is correct (band, don't preach; cap at 4) |
| P: agentic depth | idle agent runs (Meta/Amazon precedent) | chat-first workflows that are genuinely right for the role |
| F: in-band share | split into in-band chunks | legitimate large-unit work (migrations, codegen) — repo-relative band + F not scored for their PRs' repos' norms helps but does not vanish |
| Q: CI floor | keep PRs off CI paths (share-of-checked canary) | flaky-repo residents (repo-tier priors mitigate) |
| Q: churn/revert flags (NEW) | delay fixes past 21d; fix-in-new-files; revert-branch evasion | prototypers/iterators, hot-surface owners (repo-relative norms + 'outlier-only' flagging mitigate) |
| M: trends/Δ | sandbagging; window-boundary timing | task-mix shifts read as trend; new hires (explicit 'building baseline', never zero) |
| E: survey | inflate/sandbag if aggregates ever drive budgets | non-respondents; honest pessimists; cultural response styles (facts-not-estimates wording mitigates) |
| Cost strip | nothing to gain — unscored | none (that's the point) |

---

## 5. Fairness

**Cohort = function × level with hierarchical partial pooling** (backend/frontend/ML/infra/SRE/data × L3–L8). Why: function and repo environment dominate between-person variance in every output-shaped signal (3–5x structural gains spread — Cui [independent]; ~300x cost/PR spread — Jellyfish [vendor]); level-only cohorts read environment as skill (crit-fairness §3). Partial pooling avoids the ~36-hard-cell granularity trap (15-person cells jump in ~7-point steps — crit-stats Flaw 2); small cells shrink toward function-level priors. Repo-level effects are handled by **normalizing signals at the repo grain** (F's size bands, Q's CI-tier and churn priors) because repo is a work-context attribute no person-cohort can fix (crit-fairness §2.2).

- **Function**: F and M norms are cohort/repo-relative; ML's unmerged experimentation still under-observed → coverage panel states it (see §8).
- **Repo environment**: heavy-CI repos get their own Q priors; monorepo PR granularity handled by repo-relative bands; no cross-repo absolute volume anywhere.
- **Seniority**: no volume ranking exists, so juniors' higher acceptance/volume gains (Cui +27–39% vs +8–13% [independent]) stop reading as seniors "failing at AI"; seniors' review/mentoring remains invisible to P/F/M — ALP says so explicitly (coverage panel: "these facets observe roughly the coding share of your role") and E gives it partial voice; review-metadata acquisition later enables a reviewer-credit facet.
- **Agent-platform engineers**: own profile; scored on P, M(practice), E; F and output-M shown as **"unmeasured — your output isn't PR-shaped; we don't score what we can't see"** rather than zero (fixes the categorical error, crit-fairness §2.4). The prioritized new signal (agent_id/owner_id/caller → distinct repeat non-self consumers, EB-shrunk) upgrades them to a real output facet — the one gaming-resistant value proxy converging across Backstage/CNCF/reuse research (survey/agent-native.md [primary/vendor]). Hybrids get both facet sets with per-facet observation shares.
- **Part-time / leave / interns / contractors**: consistency is per-active-day; no AND-gate floors; 12-week medians + intervals; interns keep a separate cohort; contractor spend-coverage audited before display (crit-fairness §2.5).

---

## 6. Interpretability

Every facet renders as: **value → band → because-sentence → one action lever.** No hidden weights exist because nothing is averaged.

Headline example:
> **"Trajectory: Improving** — over your last 12 weeks your practice depth rose while your cohort's stayed flat, and your AI-heavy weeks shipped ~18% more merged work than your lighter weeks (±12%, descriptive — not a causal estimate). No quality flags. Data through Jun 24."

Facet examples:
> **Practice — Established** (backend · L5 band, whisker ±1 band): "You used AI meaningfully on 14 of 21 active days across 3 tool categories. Lever: you haven't tried the agent platform for your recurring refactor tasks."
> **Flow — Developing**: "9 of your 15 merged PRs were inside your repos' typical size band; 4 were >3x repo norm. Lever: slice large changes — small batches are the strongest evidence-backed AI-stability practice."
> **Quality floor — Clear**: "No anomalies vs your repos' norms. This floor only ever flags; it never boosts."
> **Unmeasured panel**: "Not visible to us: code review you give, mentoring, design, incident work, unmerged experiments[, your agents' outputs]. Absence of signal here is not absence of value."

Engineer-facing rules stated in-product: facets never combine; bands compare to an anonymous function×level cohort; the headline compares you only to yourself.

---

## 7. Cold start & churn

- **New hires**: all facets appear immediately with wide whiskers (EB shrinkage toward cohort prior — everyone always has a state); headline = "Building baseline (k of 8 weeks)" until 8 observed weeks; Δ contrast suppressed until 9 weeks with usage variance. No fake neutral values anywhere (kills the CI=1.0 pathology class).
- **Low-activity months**: zero-filled weekly spine keeps medians defined; bands widen rather than vanish; ≤2 active-day weeks are marked "reduced observation" and down-weighted in trends (right-censoring pattern, repos/git-of-theseus.md). The +27% lucky-month selection bias of floor-gated trends (crit-stats Flaw 6) disappears because no month is ever hidden.
- **2-week data lag**: the window ends at `today − 14d` uniformly across sources, so no facet mixes fresh PR data with stale usage data; UI shows "data through <date>." PR/CI data arriving earlier than usage data is truncated to the same AS_OF. **[inference — design choice]**
- **Role/team changes**: cohort switch re-anchors bands with a "cohort changed" annotation; Momentum baseline resets to "building" for 8 weeks (a trend across two different jobs is not a trend).
- **Score churn control**: band hysteresis (§2.0) + medians + 12-week window put month-over-month movement mostly under the engineer's control, addressing the reliability ≈0.3 finding for 4-week ratios (crit-stats Flaw 7).

---

## 8. Failure modes (honest)

1. **Construct retreat.** ALP measures practice + trajectory, not amplification. An engineer who is highly productive with minimal AI reads "Emerging" on Practice — accurate as description, potentially misread as deficiency. Mitigation is framing only; the tension with METR's seniors-slower finding [independent] is real: deep adoption is not proven universally good, and P leans on DORA-capability evidence that is associational.
2. **Quality floor is weak at launch.** Until churn/revert/review signals are acquired, Q = a saturated CI floor that will flag almost no one; ALP ships with its quality counterweight mostly promissory. Stated in-product ("quality coverage: partial").
3. **Momentum Δ is confounded by task mix** — heavy-AI weeks may simply be easier weeks; labeled descriptive, but users will over-read it (METR's 40pp perception gap says exactly this). Suppression thresholds trade coverage for honesty: many engineers will see no Δ at all.
4. **Invisible work stays invisible.** Review, mentoring, design, incident response, unmerged ML experimentation, and (until the consumer signal lands) agent outputs are acknowledged, not measured. Honesty ≠ measurement; agent-platform engineers still get a thinner profile than PR-shaped peers.
5. **Cognitive compression risk.** Five facets + bands + intervals invite users to mentally average them back into one number, resurrecting compensatory logic the design forbids. **[inference]**
6. **Small/rare cohorts** (e.g., L8 SRE) pool heavily toward function priors — their bands say more about the prior than about peers.
7. **Survey layer fragility**: response rates, quarterly latency, and instrument non-validation (SPACE/DevEx are doctrine, not validated instruments — survey/space-devex.md).
8. **No deployed precedent**: like the current metric, ALP as a whole is unvalidated; unlike it, every component has an evidence-backed track record and the assembly avoids every documented anti-pattern found in Phase A. Pilot-and-audit is still mandatory.

---

## 9. Why this beats the current metric — vs the 7 known problems

1. **CI-clean weak/unfair quality proxy** → CI demoted from multiplier to EB-shrunk, repo-tier-normalized anomaly floor that can only flag, never boost; discrimination reassigned to acquirable churn/revert/review counterweights. Saturation becomes harmless instead of score-defining.
2. **Narrow spend denominator; agent work invisible** → no denominator at all; ALL tools (incl. agent platform) feed Practice; work-mode profiles give agent-platform engineers a scored P/M(practice)/E profile plus explicit "unmeasured" (not zero) output facets, and the #1 prioritized acquisition (agent consumer identity) gives them a true output signal.
3. **Right-skewed per-dollar percentiles, top ranks = CI noise** → no ratios, no continuous percentile score; log-scale medians, hierarchical z, 5 coarse bands with hysteresis and intervals; headline is self-vs-self, immune to cohort skew.
4. **Brittle cold-start floors** → all AND-gates deleted; shrinkage + explicit censoring states; no on/off flicker, no lucky-month trend bias.
5. **Per-dollar penalizes harder/more-expensive work** → spend never divides anything; it appears only as unscored self-showback (FinOps pattern), so spending more on hard work costs zero score.
6. **Level-only cross-function cohort** → function×level hierarchical cohorts + repo-grain normalization + score-neutral work-mode profiles; environment stops being read as skill.
7. **CI clustering at 1.0 / fake-neutral default** → the <3-PR ⇒ 1.0 default is abolished; low evidence yields wide intervals and "insufficient observation," never the maximum score; evidence ordering is restored (more data can only sharpen, not worsen relative to absence).

Plus two structural wins over the old design: the old metric's two "signals" were ~0.9-correlated one-dimensional pseudo-triangulation (crit-stats Flaw 5) — ALP's five facets are genuinely orthogonal axes in deliberate tension; and the old metric's optimum was abandoning AI (extremal Goodhart, crit-construct §2.4) — ALP's best-possible profile requires sustained, consistent, small-batch, flag-free, improving AI-assisted work, which is the behavior the org actually wants.
