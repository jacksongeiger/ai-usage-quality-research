# DESIGN CANDIDATE — "AI Leverage Map": Investment × Delivery Quadrant with a Within-Person Elasticity Headline

Design angle: **cost-aware done right** — the steelman of the client's original per-dollar framing. Keeps the efficiency lens; removes spend from every denominator. Sub-investigator, 2026-07-08. All external claims cited [primary]/[vendor]/[independent]/[practitioner]; my reasoning labeled **[inference]**. Grounded in BRIEF.md, all four critiques (crit-construct, crit-fairness, crit-gaming, crit-stats), and survey/roi-cost.md + goodhart-design.md + repos/*.md.

---

## 1. Name & construct

**"AI Leverage Map" (LM-E).** Under this design, an individual's *leverage* is not a scalar ratio but a **position and a personal trajectory on a two-axis map**: (X) how much the engineer invests in AI across ALL metered tools, and (Y) how much PR-shaped delivery they produce — each read against their own function×level cohort — plus the headline: a **within-person spend→output elasticity**, i.e., "in *your own* heavier-AI weeks, does *your own* delivery rise?" This replaces the cross-sectional counterfactual claim the data cannot support (crit-construct §1.1: leverage-as-amplification is unobservable per person; METR https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ [independent]) with the only causal-ish design HAVE data allows: within-person period contrasts (Faros CE precedent, repos/faros.md [primary code]; independent-evidence.md §5). Spend appears **only as an axis, a band, and a private showback** — never a divisor — because per-dollar scalars are maximized by not using AI (extremal Goodhart, Manheim & Garrabrant https://arxiv.org/abs/1803.04585 [primary]), mechanically rank the most productive adopters last (Jellyfish: heavy users 2x PRs on 10x tokens, $0.28→$89.32/PR https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/ [vendor]), and have zero deployed precedent while their mirror image (spend-maximizing leaderboards) was gamed and scrapped at Meta/Amazon (https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/ [independent]).

---

## 2. Formula / signals

**Window & spine.** Rolling **12 weeks** of ISO-week buckets on a zero-filled calendar spine (fourkeys/DevLake pattern, repos/fourkeys.md, repos/devlake.md [primary code]), computed through `T − 14 days` (respects the 2-week usage lag; every panel labeled "data through <date>"). Personal baseline = trailing 26 weeks. Weeks with `active_days = 0` are marked `away` and excluded from medians (not zero-filled) — vacation is not low output.

**Cohort.** `function × level` (≈6 functions × ≈6 levels), fit as **hierarchical empirical-Bayes priors** (engineer ⊂ function×level ⊂ function), never hard percentile splits — small cells borrow strength from the function level (goodhart-design.md finding #1; Stan partial pooling https://mc-stan.org/rstanarm/articles/pooling.html [primary]). Band edges are **frozen quarterly snapshots** of cohort quantiles, not live percentiles — no zero-sum monthly reshuffling (fixes crit-stats Flaw 2; DevLake fixed-benchmark bucketing steal).

**Work-mode profile** (assigned from data, 2-window hysteresis to block profile-shopping):
```
pr_rate_EB_i    = (Σ merged_PRs_12w + α_fl) / (active_weeks_12w + β_fl)   -- Gamma-Poisson EB, (α,β) per cohort
platform_share  = agent_platform_spend_12w / total_ai_spend_12w
profile = PR-builder        if pr_rate_EB ≥ cohort_P25
        = platform-builder  if pr_rate_EB < cohort_P25 AND platform_share ≥ 0.5
        = hybrid            otherwise        -- gets BOTH panels
```

### Panel 1 — the Map (position; context, not a grade)

**X axis — AI Investment band.**
```
I_iw = log1p( total_ai_spend_iw / active_days_iw )      -- ALL tools: CLIs + chat + agent platform
X_i  = median over non-away weeks of I_iw
band = {Light, Moderate, Heavy, Frontier} from FROZEN cohort quartiles of X
```
Per-active-day normalization is an *exposure* normalizer (legitimate), not an *investment* denominator (illegitimate) — crit-construct §3 distinction. Log scale because spend is heavy-tailed (Jellyfish 90th pct ≈ 7.5x median [vendor]). Tokens tracked as a shadow axis for price-deflation robustness (dollar is non-stationary: prices −90% since 2023, Copilot credits transition 2026 — roi-cost.md §2.2 [independent/vendor]).

**Y axis — Delivery band (PR-profile and hybrid only).**
```
O_iw = min( merged_PRs_iw, cohort_P95_weekly )           -- cohort-percentile winsorization, not fixed 2000-line cap
Y_i  = EB-shrunk median of O_iw over non-away weeks      -- Gamma-Poisson posterior; Wilson-style interval displayed
band = {Emerging, Typical, High} vs frozen cohort terciles
```
Lines changed are **diagnostic display only** (PR-size distribution vs cohort norms — batch-size context per DORA's small-batches capability, survey/dora.md finding #4), used in **zero** scored quantities (middleware precedent: LOC stored, unused in metrics — repos/middleware.md [primary code]). CI-clean-rate is **removed as a score input**; it survives only as a private **anomaly flag** (EB-shrunk rate ≪ repo-cohort prior → "worth a look"), because the signal is saturated beyond statistical repair (crit-stats Flaw 3; Cui et al.: build volume +38%, success rate flat-to-negative https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 [independent]).

**Quadrant labels (protective by design):** Low-X/Low-Y "Untapped", Low-X/High-Y "Efficient — or under-invested", High-X/Low-Y **"Investment phase"** (never "wasteful" — spend only weakly tracks difficulty AND waste, 30x same-task token variance, arXiv https://arxiv.org/abs/2604.22750 [independent]), High-X/High-Y "Compounding". Optional one-tap self-annotation on any window: {exploration | migration | new domain | incident-heavy | agent-building} — displayed beside the quadrant, never scored (self-report as context only; METR's 40pp miscalibration bars scoring it — independent-evidence.md §5).

### Panel 2 — Personal Elasticity (the headline)

```
weeks  = last 12 non-away ISO weeks with data (extend lookback to 16 to collect 12 if needed)
IF count(weeks) < 8 OR cv(spend_iw) < 0.25:  RETURN state = "insufficient variation to estimate"
pairs  = [ ( log1p(spend_iw / active_days_iw), log1p(O_iw) ) for each week ]
E_i    = TheilSen_slope(pairs)                 -- robust to ≤~25% manipulated/outlier weeks
CI_i   = bootstrap 95% CI (1000 resamples, block bootstrap for serial correlation)
display: "In your heavier-AI weeks (~2× spend) you merge about (2^E − 1)% more/fewer PRs [CI]."
         Direction glyph: ▲ (CI > 0), ▼ (CI < 0), ◆ flat/uncertain (CI straddles 0 — the honest majority case)
```
E is **descriptive, not causal** and is labeled so on-screen ("your pattern, not a verdict — hard weeks cost more and ship less"). A person with uniformly high spend gets "insufficient variation," **not** a penalty — this is the explicit protection for legitimately-expensive hard work: no level of spend ever lowers any number; only your own week-to-week covariation is described. **[inference — design choice]**

### Panel 3 — Cost showback (FinOps pattern; labeled "cost awareness," never "leverage")

Per-tool-category $ and token trend vs **own 26-week baseline** only; cost-per-merged-PR shown vs **own** trailing value, never vs peers (FinOps showback stops above the individual — https://www.finops.org/wg/finops-for-ai-overview/ [primary]; roi-cost.md bottom line). Platform-builders see agent-platform engagement here (runs, sessions, active-day consistency) explicitly labeled **"participation, not value"** (run counts are engagement — survey/agent-native.md finding #3).

### Score assembly

**There is no composite 0–100.** The dashboard = quadrant position (categorical) + elasticity direction with CI + trends vs own baseline. Non-compensatory, per SPACE's metrics-in-tension rule (https://queue.acm.org/detail.cfm?id=3454124 [primary]) and the OECD/JRC composite-robustness objections (crit-stats Flaw 5). If leadership demands one number, ship `E_i` itself in plain units — never a percentile.

**Integrity canaries (org-side monitoring, never displayed per person):** PR-size pile-up just under winsorization boundaries; falling share of CI-checked PRs; cohort-level spend-mix shifts toward unmetered categories (the three telltales from crit-gaming §6).

---

## 3. Data mapping

| Signal | HAVE today (dataset) | NEW to collect |
|---|---|---|
| Total AI spend $/tokens/sessions/active-days, per user/day/tool, ALL tools incl. agent platform | ✅ usage warehouse (brief HAVE #1) | — |
| Agent-platform run counts | ✅ assumed per brief stated assumptions | — |
| Merged PR count per engineer | ✅ PR warehouse | — |
| **Per-PR merge timestamp** (needed for weekly buckets) | ⚠️ assumed present (window counts computed today imply it) — **verify; stated assumption** | if absent: GitHub API backfill, trivial |
| Lines changed per PR (diagnostic only) | ✅ PR warehouse | — |
| Per-PR CI clean flag (anomaly flag only) | ✅ CI warehouse | — |
| Role / level / function (cohorts) | ✅ HR join | — |
| Frozen cohort quantiles, EB priors | derivable from HAVE | — |
| Period self-annotation (one-tap context tags) | — | NEW: dashboard UI field, self-report, unscored |
| Post-merge churn / same-file re-touch ≤14d (quality counterweight, v2) | — | NEW: per-PR file lists via GitHub API (top acquisition — survey/gitclear.md #1) |
| Revert detection (`revert-*` branch regex; quality counterweight, v2) | — | NEW: branch-name metadata (repos/middleware.md pattern) |
| Agent run caller identity (agent_id, owner_id, caller ≠ owner) → platform-builder Y axis, v2 | — | NEW: warehouse existing platform logs (OTel GenAI/Langfuse already model this — survey/agent-native.md #2) |

v1 ships on HAVE data alone (plus the UI tag). v2 acquisitions upgrade quality and platform-builder coverage.

---

## 4. Gaming resistance — top 5 attacks

| # | Attack (from crit-gaming catalog) | Design response | Residual risk |
|---|---|---|---|
| 1 | **Spend suppression / tool substitution (D1/D2)** — route work through unmetered tools or abstain, to look "efficient" | Dies structurally: spend is an axis, not a divisor. Low spend ⇒ "Light/Untapped," which is not a better outcome; nothing anywhere improves when spend falls. All-tool metering (incl. chat + agent platform) closes the allowlist hole. | Personal/free accounts (D3) remain invisible — but they now buy *nothing* (no score rises), so the incentive is gone. **[inference]** |
| 2 | **PR splitting / trivia farming (N1/N2)** — inflate Y and tilt E | Weekly counts winsorized at cohort P95; Y is a *median of weeks* (one burst week ≈ no movement); elasticity is within-person, so *sustained* inflation raises the engineer's own baseline and stops paying; size pile-up canary monitored org-side. | A disciplined always-micro-PR habit still lifts the Y band. Bounded by winsorization + zero external stakes (self-view). |
| 3 | **Spend-week shaping** — batch expensive sessions into the same weeks big merges land, to fake ▲ elasticity | Theil-Sen tolerates ~25% contaminated weeks; faking a slope requires coordinating spend AND merge timing across ≥8 of 12 weeks — sustained effort for a private glyph; block bootstrap widens CI under autocorrelation. | Real but expensive; E labeled descriptive so a faked ▲ misleads only its owner. |
| 4 | **Profile shopping** — shift spend onto the agent platform to escape the Y axis (crit-fairness §2.4 gaming note) | Profile keyed on *output presence* (pr_rate_EB ≥ P25), not spend share: if you merge PRs, you get the PR panel regardless of where spend goes; 2-window hysteresis stops flip-flopping; hybrids get both panels. | An engineer who truly stops merging changes profile — correctly. |
| 5 | **Token burning / idle agents** to reach "Frontier" band (mirror of Meta "Claudeonomics" gaming [independent]) | Top band is open-ended with zero attached reward — no leaderboard, no score credit, band capped at label level; showback frames high spend as cost awareness, not status. | Vanity band-climbing possible; pays nothing, costs real money, visible in org-side spend-mix canary. |

Cross-cutting: strict self-view is retained as the single strongest control (Austin informational measurement; Beck self-measurement — https://newsletter.kentbeck.com/p/measuring-developer-productivity-440 [practitioner]); Amazon showed perceived stakes matter more than declared ones [independent], so nothing here exports to managers, and quadrant labels avoid valence that would make screenshots status-worthy. **[inference]**

---

## 5. Fairness

**Cohort = function × level, hierarchical.** Function is the dominant structural axis (PR/line/spend norms and achievable gains differ 3–5x by function/environment — Cui https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 [independent]; DORA 2025 amplifier thesis https://dora.dev/dora-report-2025/ [primary]); level captures the tenure gains gradient (juniors +27–39% vs seniors +8–13%, Cui). Partial pooling avoids the 36-cell granularity trap (crit-stats Flaw 2). FT/contractor/intern kept only as *coverage* flags: contractors with unmetered spend get "spend coverage unknown," never a fake Light band (crit-fairness §2.5).

Per axis/metric — (a) gamed how / (b) unfairly penalizes:
- **X (investment band):** (a) token burning (attack 5); (b) monorepo/large-context engineers read Heavy for identical work (input tokens dominate cost — arXiv 2604.22750 [independent]) — mitigated by band coarseness + zero valence on high X; residual unfairness acknowledged.
- **Y (delivery band):** (a) micro-PR habit (attack 2); (b) reviewers/mentors/design-heavy seniors and ML experimenters whose work isn't merged-PR-shaped — Y is banded not ranked, and the panel states "measures merged-PR delivery only, a partial view of your work" (SPACE activity-metric rule [primary]). Not fully fixable with HAVE data; stated on-screen.
- **E (elasticity):** (a) spend-week shaping (attack 3); (b) engineers whose hard weeks are both expensive and slow (task-mix confound → spurious ▼) — mitigated by CI display, "descriptive" labeling, self-annotation tags, and the flat/uncertain default.
- **Profiles:** (a) profile shopping (attack 4); (b) hybrids straddling panels — they get both, with each panel's coverage caveat.
- **Agent-platform engineers:** no longer near-zero scored. They get X band + showback + engagement (labeled participation-not-value) + an explicit **"delivery unmeasured — no PR-shaped output signal exists for your work"** state, per the honest-unmeasured rule (repos/devlake.md N/A pattern; DORA J-curve shows delivery metrics mis-price platform work — survey/agent-native.md #5 [primary]). v2 caller-identity signal (distinct non-self repeat consumers) is the fairness upgrade path.
- **Part-timers / leave / on-call:** per-active-day normalization + away-week exclusion + EB intervals instead of floors; new hires see "building baseline (n weeks observed)," never a cliff.

---

## 6. Interpretability

Header sentence, PR-profile: **"Data through Jun 24. You sit in the Heavy-investment / High-delivery quadrant for backend L5. In your heavier-AI weeks (~2× your usual spend) you merged about 23% more PRs (95% CI +6% to +41%) ▲. Your spend band is context, not a grade — this panel measures merged-PR delivery only."**

Platform-builder variant: **"Data through Jun 24. Your AI investment is Frontier for infra L6, mostly on the agent platform. We can't yet measure the value of agent-building work — this shows participation, not output. Your cost trend is 12% below your own 26-week baseline."**

Every number traces to a visible input (weeks, PRs, $/active day); no hidden weights, no composite, no percentile.

---

## 7. Cold start & churn

- **New hires:** map position appears from week 1 with wide EB intervals and "building baseline — n of 8 weeks observed"; elasticity locked to "not yet estimable" until 8 qualifying weeks. No AND-gates, no on/off flicker (kills brief problem #4 and crit-stats Flaw 6's 32%-flicker / +27% self-trend bias).
- **Low-activity months:** away weeks excluded, active weeks shrink toward the cohort prior with the interval widening — a visible "less certain," never a worse grade; raw + shrunk + interval all shown (goodhart-design.md EB caveat: protect low-volume excellent performers).
- **2-week lag:** all panels computed to T−14d and dated; the 12-week window makes the lag ~17% of the window instead of 50% of a 4-week one. **[inference — arithmetic]**
- **Quarterly band re-freezing:** band edges move only at quarter boundaries with a changelog note ("cohort norms updated"), so within-quarter movement is always attributable to the engineer's own data.

---

## 8. Failure modes (honestly weak)

1. **E is correlational.** Task-mix confounding can flip its sign for individuals (hard weeks: more spend, less output). Mitigations (labeling, tags, CI) manage but do not remove this. No design under these constraints measures true amplification (crit-construct §1.1).
2. **Most engineers will read ◆ flat/uncertain.** At individual weekly grain, CIs will often straddle zero (Cui needed n≈5,000 for significance [independent]) — statistically honest, motivationally underwhelming. The map and trends carry the engagement load.
3. **No quality counterweight in v1.** CI is dead, churn/revert need new data — output inflation with declining quality is invisible until v2 acquisitions land (GitClear churn evidence — [vendor]). Y is labeled "delivery," not "quality."
4. **Y is still PR-shaped.** Review, mentoring, design, incident response remain unmeasured (Austin effort-reallocation risk persists, attenuated by no-composite/no-rank design). Same blind spot as every framework surveyed.
5. **Platform-builders get honesty, not measurement,** until caller-identity signals are warehoused — an explicit "too new to score" posture (survey/agent-native.md #5), which may frustrate them even though it beats a near-zero score.
6. **Dollar non-stationarity** (price cuts, credit promos) contaminates X and showback across quarters; token shadow axis + quarterly re-freezing only partially deflate it (roi-cost.md §3.4).
7. **Ambient workflow drift**: as AI defaults inflate PR counts org-wide (Faros +98% PRs [vendor]), within-person baselines inflate too; a rising own-trend can reflect the calendar, not the person (crit-gaming §3). Cohort-context bands partially anchor this.

---

## 9. Why this beats the current metric — vs the 7 known problems

1. **CI-clean weak proxy** → removed from scoring entirely; demoted to a private EB-shrunk anomaly flag with repo-cohort priors. No saturated signal multiplies anything (crit-stats Flaw 3 fix).
2. **Narrow spend denominator / invisible agent work** → denominator abolished; spend coverage widened to ALL tools including the agent platform; platform-builders get their own profile, showback, and an explicit unmeasured state instead of structural bottom rank.
3. **Right-skewed, non-discriminating per-dollar distribution** → no ratio exists to skew. Log-scale bands from frozen cohort quantiles + EB medians; the headline is a within-person slope, immune to cross-person skew (crit-stats Flaws 1–2 fixes).
4. **Brittle cold-start floors** → all AND-gates deleted; EB shrinkage + displayed intervals + explicit "building baseline"/"insufficient variation" states; 12-week rolling window (crit-stats Flaw 6 fix).
5. **Penalizes expensive hard work** → the design's centerpiece: no number decreases when spend increases, high-X/low-Y is labeled "Investment phase," elasticity uses only own variation and returns "insufficient variation" for steady heavy spenders, and self-annotation lets the engineer contextualize expensive periods.
6. **Level-only cross-function cohort** → function×level hierarchical cohorts with partial pooling; contractors handled as a coverage question, not a fairness bucket (crit-fairness §3 fix).
7. **CI-rate non-discrimination / 1.0 point mass** → same as #1; additionally no fake-neutral defaults anywhere — every missing signal renders as an explicit state (DevLake `_is_collected_data` steal), so absence of evidence is never scored as excellence.

Plus three inherited-anti-pattern removals the problem list didn't name: no averaged percentile ranks (ordinal violation), no arithmetic mean of normalized ratios (Fleming & Wallace https://dl.acm.org/doi/10.1145/5666.5673 [primary]), no published fixed gaming boundary (2000-line cap → cohort-P95 winsorization with sensitivity analysis).

---

### Key sources (all previously verified in Phase A; re-cited above)
Jellyfish tokenmaxxing [vendor] · Fortune 2026-07-07 Meta/Amazon leaderboards [independent] · arXiv 2604.22750 spend≠difficulty [independent] · Cui et al. Mgmt Sci 2026 [independent] · METR RCT + 2026 update [independent] · FinOps for AI [primary] · SPACE [primary] · DORA 2024/2025 [primary] · Manheim & Garrabrant [primary] · Fleming & Wallace [primary] · Stan partial pooling [primary] · Robinson EB [practitioner] · Beck [practitioner] · Faros productivity paradox [vendor] · repos: fourkeys, devlake, middleware, faros-CE [primary code].
