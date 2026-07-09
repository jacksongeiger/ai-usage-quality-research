# DESIGN CANDIDATE — "Net Delivery Goodput" (NDG)

Sub-investigator design, 2026-07-08. Angle: quality-adjusted throughput ("goodput") done right — output-centric, spend removed from the denominator, CI-clean-rate replaced, normalization solved. Grounded in BRIEF.md, all four crit-*.md files, and Phase-A survey/repo notes; external claims cited with tags; my own reasoning labeled **[inference]**.

Design contract with the critiques (what this design is forbidden to do): no investment denominator (crit-construct §3; crit-stats Flaw 1); no CI-clean multiplier (crit-stats Flaw 3 — "statistically dead"); no averaging of correlated percentile ranks (crit-stats Flaw 5; Fleming & Wallace, https://dl.acm.org/doi/10.1145/5666.5673 [primary]); no fixed per-PR winsorization cap (crit-stats Flaw 4); no hard AND-gated floors (crit-stats Flaw 6); no level-only cross-function cohort (crit-fairness §3); no live zero-sum peer percentile as the display (crit-gaming W2, crit-stats Flaw 2); no fake-neutral defaults (DevLake N/A pattern, repos/devlake.md [primary code]).

---

## 1. Name & construct

**Net Delivery Goodput (NDG).** Borrowing the networking definition — goodput is useful application-level throughput after subtracting retransmissions and protocol overhead — NDG defines an engineer's measurable output as **merged change volume net of "retransmissions" (reverted work and near-term self-rewrites) and "overhead" (generated/vendored/non-code churn), per week of actual presence, read against function×level cohort norms and the engineer's own trailing baseline.** Under this design, "AI-assisted engineering leverage" is deliberately NOT a ratio and NOT a measured amplification factor (the counterfactual is unmeasurable per person — METR, https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ [independent]; crit-construct §1.1). Leverage is operationalized as a **joint read**: the trajectory of your durable output (NDG) displayed alongside the trajectory of your AI intensity (all-tool spend/tokens/sessions — context strip, never a divisor). The honest claim the dashboard makes: "your durable delivered output is at level X for your cohort and moving in direction Y, over a period in which your AI use did Z." Spend never divides anything; quality is netted out of the numerator (goodput semantics) rather than multiplied in as a saturated rate; and work the data cannot see (agent-platform builds, review, mentoring) is declared unmeasured, never scored zero. **[inference — construct synthesis; consistent with DORA 2026 ROI framing "we don't measure AI by the code it writes but by the bottlenecks it clears," https://dora.dev/ai/roi/report/ [primary], and SPACE's rule that activity metrics never stand alone, https://queue.acm.org/detail.cfm?id=3454124 [primary]]**

---

## 2. Formula / signals

### 2.0 Overview of assembly

ONE headline quantity (splitting-neutral net volume per exposure week), z-scored inside a hierarchical function×level cohort model, displayed as one of **5 fixed annually-anchored bands + a self-trend with uncertainty**. Three bounded companion tiles deliberately in tension with the headline (cadence, batch discipline, durability), one unscored context strip (AI intensity), and explicit non-score states. Nothing is averaged into a composite; the companion tiles are non-compensatory displays (SPACE metrics-in-tension; goodhart-design.md §5 non-compensatory subscores).

### 2.1 Windows and spine

- Scoring window: **rolling 12 weeks**, recomputed weekly on a zero-filled calendar spine (median-of-weekly-buckets machinery per DevLake, repos/devlake.md [primary code]; 12 weeks matches DX Core 4's lookback, https://getdx.com/research/measuring-developer-productivity-with-the-dx-core-4/ [vendor], and answers crit-stats Flaw 7's power complaint — brief-sanctioned).
- Trend baseline: engineer's own trailing **26-week** median of the same quantity.
- Data-lag rule: score is stamped "data through D" where D = min(freshness of all inputs) − safety margin. PR/CI warehouse data is assumed fresher than the ~2-week-lagged usage tables **[inference — verify]**; the headline (PR/CI-based) uses its own freshness date, and the AI-intensity strip carries its own, later-lagged "through" date rather than holding the whole dashboard back.

### 2.2 Per-week net delivered volume (the goodput numerator — the only scored volume quantity)

```
-- Grain: engineer e, ISO week w. All caps/priors computed per cohort CELL (function × level, §2.5).

eff_lines(pr)      = additions(pr) + deletions(pr)              -- deletions at FULL weight: simplification is delivery
                                                                 -- v0: total "lines changed" if add/del split unavailable
                     -- v1 filters (NEW data, §3): computed ONLY over non-excluded paths
                     --   exclude: lockfiles, vendored dirs, generated files, non-code filetypes
                     --   (git-of-theseus Pygments-allowlist pattern; CHAOSS analyzecommit zeroing — repos/*.md [primary code])

retransmit(pr)     = eff_lines(pr)  if pr is REVERTED within 28 days           (v1, NEW revert linkage)
                   = eff_lines of the revert PR itself (the revert leg also earns 0)
self_rework(e,w)   = lines merged by e that e re-changes in the same files within 21 days   (v1, NEW per-PR file lists)
rework_allowance   = cell_median(self_rework_share) * lines(e,w)
                     -- only EXCESS rework is netted: relative, not absolute, churn
                     -- (Nagappan & Ball ICSE 2005: relative churn predicts defects, absolute does not,
                     --  https://dl.acm.org/doi/10.1145/1062455.1062514 [independent]; some churn is healthy — gitclear.md)

net_lines(e,w)     = max(0,  Σ_pr eff_lines − Σ retransmit − max(0, self_rework − rework_allowance))
wk_lines(e,w)      = min(net_lines(e,w), P95_cell_weekly)
                     -- winsorize at the WEEK level at a cohort percentile:
                     -- splitting one PR into k PRs changes nothing (splitting-neutral);
                     -- no published per-PR cap to camp under (fixes crit-stats Flaw 4);
                     -- cutpoint refit yearly with sensitivity analysis (OECD/JRC checklist, goodhart-design.md)
```

Retro-adjustment: reverts/rework discovered later reduce already-counted weeks on recompute — the score can legitimately fall after the fact, and the UI says so ("durability discount"). This is the goodput semantics: delivered minus retransmitted. **[inference]**

### 2.3 Headline: Goodput level and trend

```
exposure_weeks(e)  = # of the 12 spine weeks with ANY activity signal (AI-active day, merged PR, or CI event),
                     floored at 6      -- leave/part-time weeks drop out; floor bounds the residual gaming (§4)
G(e)               = log1p( Σ_w wk_lines(e,w) / exposure_weeks(e) )

z(e)               = hierarchical z-score of G within cell(function × level), partial pooling across cells
                     (engineer ⊂ cell; small cells shrink toward the function and org grand means —
                      Stan/Efron-Morris partial pooling, https://mc-stan.org/rstanarm/articles/pooling.html [primary];
                      fixes the 36-tiny-cells problem, crit-fairness §3.3)

BAND(e)            = which of 5 fixed cutpoints z falls in; cutpoints = quintiles of the PRIOR CALENDAR YEAR's
                     cell distribution, frozen for 12 months
                     -- fixed research-anchored buckets, not live percentiles: not zero-sum, no adoption-wave drift
                     -- (fourkeys/DORA bucket pattern — repos/fourkeys.md [primary code]; crit-stats Flaw 2 fix)

TREND(e)           = (G_now − median(G, trailing 26w)) with a bootstrap/posterior interval;
                     shown as ↑ / → / ↓ only when the interval excludes zero, else "no reliable change"
```

### 2.4 Companion tiles (bounded, non-compensatory, deliberately in tension with volume)

```
CADENCE     = EB-shrunk share of exposure weeks with ≥1 merged PR
              Beta-binomial: (weeks_with_merge + α) / (exposure_weeks + α + β), (α,β) fit per cell
              (Robinson EB method, http://varianceexplained.org/r/empirical_bayes_baseball/ [practitioner])
              -- bounded ≤ 1: confetti cannot push it past "shipped every week"
BATCH       = EB-shrunk share of the engineer's PRs whose eff_lines fall inside the cell's central band [p10, p90]
              -- DORA "working in small batches" capability, inverted from rewarding raw size (dora.md #4 [primary]);
              -- tension: maximizing volume with monster PRs tanks BATCH; confetti (all-tiny PRs) also tanks it
DURABILITY  = 1 − EB-shrunk excess-rework share (reverts + excess self-rework, from §2.2), vs cell norm
              -- v1 only; in v0 the tile shows "pending — durability signals not yet collected", never a fake value
CI ANOMALY  = display-only flag if EB posterior of CI-clean rate < cell p5 (Beta-binomial prior per cell;
              Wilson intervals for display, Brown/Cai/DasGupta 2001 [primary])
              -- CI is demoted to an anomaly floor exactly as crit-stats Flaw 3 prescribes; NEVER a multiplier,
              -- no default for <3 checked PRs — the flag is simply not computable and says so
AI INTENSITY (context, never scored) = all-tool $ + tokens + sessions + active days (chat, CLIs, agent platform),
              log-scaled, shown vs OWN trailing baseline + anonymous cell band; separate "data through" date (lag)
```

### 2.5 Cohorts

Cell = **function × level** (≈6 functions × ~6 levels), estimated hierarchically (partial pooling), never hard-split percentiles. Interns are their own cell (largest structural AI gains — Cui et al., https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 [independent-adjacent]); contractors sit in their function×level cell with a data-coverage caveat until spend-metering coverage is audited (crit-fairness §2.5). A **work-mode profile** overlays the cell (not a cohort): share of the engineer's AI activity on the agent-execution platform + PR volume classify them PR-delivery / hybrid / platform-builder; platform-builder ⇒ headline suppressed in favor of the honest state in §2.6. Repo environment cannot be a person-cohort (one engineer, many repos — crit-fairness §2.2): v1 normalizes per-PR eff_lines against per-repo median PR size before cell aggregation where repo_id is available **[assumption: PR rows carry repo_id — verify]**; residual repo-mix confounding is declared in §8.

### 2.6 States (no floors, no fake neutrals)

- **Scored** — interval narrow enough to band.
- **Provisional** — banded but wide interval shown ("Band 3, low confidence — 2 merged PRs, 7 exposure weeks"); also the permanent v0 label while durability netting is unbuilt.
- **Insufficient observation** — <6 exposure weeks or zero merges in 12 weeks: no band, explicit reason (Kaplan-Meier right-censoring spirit — repos/git-of-theseus.md).
- **Unmeasured work profile** — platform-builder profile: "≈X% of your recorded AI work happens on the agent platform, which has no output signal yet — your goodput covers only the PR-visible remainder." Hybrid profiles get the headline plus a coverage percentage. (Fixes the categorical error of crit-fairness §2.4 by refusing to fake a number.)

---

## 3. Data mapping

| Signal | Status | Source / acquisition |
|---|---|---|
| Merged PR count, per-PR lines changed | **HAVE** | PR warehouse (per-PR grain with merge timestamp — **assumption: merge dates exist; the current 4-week metric already windows PRs** [inference — verify]) |
| Per-PR CI pass/clean flag | **HAVE** | CI warehouse (used only for the anomaly flag + exposure signal) |
| All-tool AI usage: $, tokens, sessions, active days (incl. chat + agent platform) | **HAVE** | usage tables (~2-week lag — context strip only) |
| Role, level, function | **HAVE** | HR join (cells) |
| Additions vs deletions split per PR | **NEW (cheap)** | GitHub API `additions`/`deletions` fields; v0 falls back to total lines changed |
| repo_id per PR | **HAVE-adjacent** | verify in warehouse; needed for per-repo size normalization (v1) |
| Per-PR file path list | **NEW (cheap, top priority)** | GitHub API files endpoint; enables path exclusion + 21-day self-rework netting (gitclear.md #1 acquisition) |
| Revert linkage | **NEW (cheap)** | PR title "Revert …" / branch `revert-<n>` regex (middleware pattern, repos/middleware.md [primary code]); metadata-only |
| Bot/service-account filter on PR authors | **NEW (cheap, hygiene)** | GitHub `user.type=="Bot"` + name regex (no reference impl we read does this — repos/fourkeys.md, repos/faros.md; must not inherit the gap) |
| Agent-platform run outcomes + caller identity (agent_id, owner_id, caller) | **NEW (larger)** | platform telemetry (OTel GenAI/Langfuse-shaped — agent-native.md); unlocks adoption-by-others scoring for platform-builders — out of NDG scope, named as successor signal |
| Review metadata (approvals, merged-without-review counts) | **NEW (optional)** | forge API; shallow counts only — respects "no code-review depth" limit **[inference — borderline; flag for privacy review]**; would counter cost-shifting (Faros: review time +91%, https://www.faros.ai/blog/ai-software-engineering [vendor]) |

**v0 = HAVE-only** (headline computable but labeled Provisional; durability tile "pending"). **v1 = + the three cheap NEW rows** (paths, reverts, add/del split) — that is the complete design.

---

## 4. Gaming resistance — top 5 attacks

Threat model per crit-gaming §0: perceived stakes exist even self-view; ambient (no-intent) exploits matter most.

1. **Spend/denominator attacks (crit-gaming D1–D5, the old #1 lever, ~40x).** Structurally dead: there is no denominator to shrink and no reward for abstention or cheap-model theater. Residual: exposure_weeks is a mild divisor — an engineer could avoid logging AI activity in zero-output weeks to shrink it. Bounded by the floor of 6 (max benefit 2x), by PR/CI events also counting as exposure, and it simultaneously tanks their own AI-intensity context — low value, flagged as a monitored canary. **[inference]**
2. **PR splitting / cap-camping (N1, L2 — old quadruple-dip).** Volume is summed at week level and winsorized at week level: splitting one PR into k changes the headline by exactly zero lines. No per-PR cap exists to camp under. Splitting can still lift CADENCE (bounded at 1) and can push BATCH down if the pieces leave the central band. Residual: spreading a big change across window weeks to dodge the weekly p95 cap — 12-week rolling recompute means the same lines count once either way; only the cap-clipping differs (small, monitored via pile-up-at-cap canary inherited from crit-gaming §6).
3. **Verbose line farming / lockfile-vendored churn (L1).** Three stacked defenses: v1 path exclusion removes lockfiles/vendored/generated/non-code lines before counting; log1p gives steep diminishing returns; weekly cell-p95 winsorize caps bursts. Residual (honest): hand-crafted verbose *code* on real paths survives v0 entirely and survives v1 until it is rewritten (then netted as rework) — this is the design's largest open surface (§8), partially self-limiting because padded code attracts future rework that the durability discount charges back to its author. **[inference; GitClear: <50% of raw changed lines meaningful, https://gitclear-public.s3.us-west-2.amazonaws.com/GitClear-AI-Copilot-Code-Quality-2025.pdf [vendor]]**
4. **Churn recycling / revert double-dip (L3, N4 — old metric paid 2–3x for failure).** Reverted PRs are netted (both legs: the reverted lines AND the revert PR earn zero), and same-file self-rework within 21 days beyond the cell-median allowance is subtracted. Attacks that remain: delay the fix past day 21 (leaves the bug live — costly to the gamer; retouch-lag distribution monitored as a canary); fix in *new* files (undetectable in metadata — declared); have a teammate land the revert (revert regex still links the reverted PR; the reverter loses nothing since revert PRs are merely zero, not negative, for the reverter). **[inference]**
5. **Effort reallocation toward PR-shaped work (W3 — the Austin attack).** Not solvable by any output metric; NDG mitigates rather than pretends: platform-builder/hybrid profiles remove the "abandon the agent platform" gradient (that work is 'unmeasured', not zero); coverage-% labeling tells hybrids the score sees only part of their work; review/mentoring remain invisible and the dashboard SAYS so (§6 sentence includes coverage framing). The residual nudge toward mergeable work is inherent to the design angle and is declared in §8. (Austin 1996 via goodhart-design.md [academic].)

Also inherited/kept: self-view-only privacy (strongest single control — Beck, https://newsletter.kentbeck.com/p/measuring-developer-productivity-440 [practitioner]); integrity canaries dashboarded for the metric owners: weekly-cap pile-ups, CI-checked-share drops, retouch-lag bunching at 22+ days, exposure-week shrinkage.

**Per-signal gaming/penalty table (every metric in this design):**

| Signal | (a) Gamed by | (b) Unfairly penalizes |
|---|---|---|
| Net lines headline | Verbose real-path code (v0 esp.); new-file rework evasion; monster weeks under p95 cap | Precision workers (small critical fixes), reviewers/mentors/designers, ML engineers whose experiments never merge; brownfield owners vs greenfield **[inference; Denisov-Blanch gradient unverified]** |
| Exposure weeks | Hide AI use in idle weeks (bounded 2x by floor) | None materially; part-timers are protected by it |
| CADENCE | One trivial merge per week (bounded at 1.0) | Big-batch release-train and long-cycle workflows — mitigated by EB prior per cell, tile-not-score |
| BATCH | Sizing PRs to sit inside [p10,p90] regardless of natural task shape | Legitimate large migrations/codegen and legitimate one-liner fixers; per-cell bands limit this |
| DURABILITY | Delay rework >21d; fix-in-new-files; refactor-avoidance/timidity | Prototypers/iterators and hot-surface owners — countered by the cell-median allowance (only EXCESS rework netted) and by tile-not-multiplier status |
| CI anomaly flag | Keep PRs out of CI paths (flag becomes uncomputable — shown as such, not as clean) | Flaky/heavy-CI repos — bounded: it's a p5 anomaly flag vs cell, not a rate score |
| AI-intensity context | Idle sessions/token burn to look engaged (Meta/Amazon failure mode, https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/ [independent]) | Nobody — it is unscored; displayed vs own baseline |
| Bands + self-trend | Sandbag the 26w baseline to farm ↑ later (costly: 26 weeks of real underdelivery); merge-timing arbitrage is dead (12w rolling + fixed annual cutpoints, not live percentiles) | New hires (no baseline — trend suppressed, shown as "building baseline", not zero) |

---

## 5. Fairness

- **Function:** cells are function×level, so frontend PR confetti never ranks against data-eng migrations (fixes brief problem #6). All caps, priors, batch bands, and rework allowances are per-cell. Partial pooling keeps 12-person cells stable (crit-stats Flaw 2 granularity fix).
- **Repo environment:** CI is out of the score, so the (1−f)^jobs unfairness (crit-stats §3.3) is gone; PR-size norms are per-repo-normalized in v1; residual repo-mix effects inside a cell are declared, not hidden. Cross-repo winsorization uses cell percentiles, not global constants.
- **Seniority:** deletions count at full weight (rewards senior simplification — Tornhill, code-maat README [primary]); BATCH and DURABILITY favor right-sized, durable senior work instead of volume; review/mentoring/design remain invisible — the dashboard states coverage honestly rather than scoring the visible part as if it were the whole (SPACE activity-metric rule [primary]). Juniors are not punished for volume, but churny volume is netted.
- **Agent-platform engineers:** never bottom-ranked, never zero — explicit "unmeasured work profile" state with coverage %, plus the named successor signal (per-run caller identity → adoption-by-others) as the real fix (agent-native.md). Escaping the score by shifting spend to the platform is possible but earns "unmeasured," not a good band — weak gaming value. **[inference]**
- **Part-time / leave / on-call / interns:** exposure-week pro-rating + no floors + EB shrinkage + intervals; interns get their own cell; months with low data degrade to Provisional/Insufficient states instead of flickering on/off (fixes brief problem #4).
- **Why this cohort:** function×level is the finest person-attribute grouping the HAVE data supports that removes the dominant structural variance (crit-fairness verdict matrix); repo and work-mode are handled as signal normalization and profile overlays because they are work-context attributes, not person attributes.

---

## 6. Interpretability — the dashboard sentence

> "Over the 12 weeks ending {date}: you shipped **{N} durable lines/week** — merged work, minus reverted and quickly-rewritten code, with generated/vendored files excluded — placing you in **Band {B} of 5** among {function} {level}s (fixed yearly benchmarks, not a live ranking). That's **{↑/→/no reliable change}** vs your own 6-month baseline. Your AI intensity (all tools, incl. the agent platform; data through {date−lag}) was {low/typical/high} for your cohort and {rising/flat}. This score covers your merged-PR work only — review, mentoring, and agent-platform building are not measured."

Every clause maps to one computed quantity; nothing in the sentence is a black-box weight. **[inference — interpretability by construction]**

---

## 7. Cold start & churn

- **New hires:** scored from week 1 as Provisional (cell-prior-shrunk, wide interval); band appears when the interval fits inside ~2 bands; TREND suppressed until 8+ weeks of own history ("building your baseline"). No cliff: 2 vs 3 merged PRs changes the interval width, not existence (fixes brief problem #4).
- **Low-activity months:** exposure-week pro-rating absorbs leave/on-call; a genuinely idle 12 weeks yields "Insufficient observation" with the stated reason — a labeled state, not a hidden zero or a vanished month (avoids the +27% lucky-month self-trend bias of gated displays, crit-stats §6.1).
- **2-week data lag:** headline runs on PR/CI freshness; the AI-intensity strip lags independently and is labeled with its own "data through" date; recomputes are idempotent on late-arriving usage rows. Retro-adjustments from late-discovered reverts/rework are expected behavior and labeled.
- **Metric churn (prices, tools, vendors):** no dollar term in the score → model-price deflation and the Copilot credits transition (roi-cost.md [vendor/independent]) cannot move the headline; band cutpoints refreshed annually with a published changelog.

---

## 8. Failure modes (honestly weak)

1. **v0 is inflatable.** Without path filters + revert/rework netting, the headline is log-tempered LOC — the exact commodity-gameable quantity the literature damns (goodhart-design.md §4). NDG is only defensible shipped as v1; v0 must carry a permanent "Provisional — durability netting pending" label. If the three cheap acquisitions are refused, this design should lose to a practice/trend-based design.
2. **Goodput ≠ value.** No difficulty/business-value signal exists (hard limit #2): a large, durable, trivial change outscores a brilliant one-line fix. Coarse 5-band display + self-trend-first framing blunt but do not fix this. Permanent, declared.
3. **The leverage read is correlational.** NDG × AI-intensity is a joint display, not a causal estimate; sign instability in the field (METR 2025 vs 2026 [independent]; DORA 2024 vs 2025 [primary]) means the dashboard must never phrase it as "AI made you X% faster."
4. **Invisible-work blind spot persists.** Review burden absorbed by teammates (Faros +91% review time [vendor]) stays unpriced without the optional review-metadata acquisition; mentors/reviewers/designers still under-read. Declared in the UI sentence.
5. **New-file rework evasion + >21d delayed rework** are undetectable in metadata; durability netting is a floor on churn pathology, not a full quality measure (25–45% of AI code passes CI with flaws — independent-evidence.md #4).
6. **Line-shaped residue.** Even netted and log-tempered, lines-per-week inherits LOC pathologies at reduced strength; terse-language and precision-work styles under-read within cells.
7. **Annual band anchoring drifts** during fast adoption waves (a Band-3 of last year ≠ this year); the alternative (live percentiles) is worse (zero-sum, non-stationary) — accepted trade-off with yearly recalibration.
8. **Ambient inflation of the baseline:** as AI defaults get more verbose/churny org-wide (GitClear trend [vendor]), self-trend ↑ can reflect workflow drift; mitigated (not solved) by netting churn and yearly re-anchoring — crit-gaming §3's warning applies at reduced amplitude.

---

## 9. Why this beats the current metric — vs the 7 known problems

1. **CI-clean-rate weak/saturated:** removed from the score entirely; demoted to a cell-relative anomaly flag with EB posterior + Wilson interval and no fake 1.0 default; quality lives in the numerator as revert/rework netting — a signal AI cannot saturate by iterating to green, because it fires on post-merge behavior. (Directly implements crit-stats Flaw 3's two-layer fix.)
2. **Narrow spend denominator / invisible agent work:** there is no spend denominator; AI intensity covers ALL tools including the agent platform as unscored context; platform-builders get an honest unmeasured state + a named acquisition path instead of near-zero scores.
3. **Right-skewed per-dollar, noise-decided top ranks:** no ratio exists; volume is log-scaled, week-winsorized at cell percentiles, z-scored in a hierarchical model, and displayed as 5 fixed bands — top-band membership requires beating a year-frozen cutpoint, not this month's noise (kills regressional-Goodhart rank churn; crit-stats Flaw 2).
4. **Brittle cold-start floors:** all AND-gates deleted; EB shrinkage + displayed intervals + three explicit non-score states; 12-week rolling window; score existence is continuous, not a step function.
5. **Per-dollar punishes spending more on harder work:** spend cannot lower any number on the dashboard; heavy legitimate spend shows only in the context strip vs one's own baseline — the Jellyfish inversion (top producers ranked last, https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/ [vendor]) is impossible by construction.
6. **Level-only cross-function cohort:** replaced by function×level cells with partial pooling, per-cell caps/priors/bands, plus per-repo size normalization — comparison bias removed at the layer it lives in (crit-fairness §3).
7. **CI-clean barely discriminates / penalizes heavy-CI repos:** see #1 — the signal no longer touches the score, so 50-job repos lose no points; the anomaly flag is cell-relative so heavy-CI cells set their own p5.

Plus the statistical anti-patterns eliminated: no averaged percentiles (single-construct headline), no correlated pseudo-triangulation (companion tiles are non-compensatory and in tension), no fixed absolute winsorization constant (cell-percentile, week-level), no fake neutrals, no live zero-sum display, splitting-neutral volume accounting.

---

### Sources (all previously verified in Phase-A files; tags repeated)
DORA small-batch + 2024/2025 reports [primary] https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report · https://dora.dev/ai/roi/report/ ; SPACE [primary] https://queue.acm.org/detail.cfm?id=3454124 ; Nagappan & Ball 2005 [independent] https://dl.acm.org/doi/10.1145/1062455.1062514 ; GitClear 2025 PDF [vendor] https://gitclear-public.s3.us-west-2.amazonaws.com/GitClear-AI-Copilot-Code-Quality-2025.pdf ; Jellyfish tokenmaxxing [vendor] https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/ ; Faros paradox [vendor] https://www.faros.ai/blog/ai-software-engineering ; Cui et al. Mgmt Sci 2026 [independent-adjacent] https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 ; METR RCT + update [independent] https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ · https://metr.org/blog/2026-02-24-uplift-update/ ; Fortune tokenmaxxing [independent] https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/ ; Fleming & Wallace [primary] https://dl.acm.org/doi/10.1145/5666.5673 ; Brown/Cai/DasGupta [primary] http://www-stat.wharton.upenn.edu/~lbrown/Papers/2001a%20Interval%20estimation%20for%20a%20binomial%20proportion%20(with%20T.%20T.%20Cai%20and%20A.%20DasGupta).pdf ; Robinson EB [practitioner] http://varianceexplained.org/r/empirical_bayes_baseball/ ; Stan pooling [primary] https://mc-stan.org/rstanarm/articles/pooling.html ; Beck [practitioner] https://newsletter.kentbeck.com/p/measuring-developer-productivity-440 ; code-maat README [primary] https://github.com/adamtornhill/code-maat ; repo evidence [primary code] repos/{devlake,fourkeys,middleware,faros,git-of-theseus,chaoss,discovered}.md. "Unverified" items flagged inline (merge-timestamp/repo_id availability; Denisov-Blanch magnitudes; review-metadata privacy boundary).
