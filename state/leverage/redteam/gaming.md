# RED TEAM — Gaming the RECOMMENDED_DESIGN (ALP, Anchored Hybrid)

Red-team sub-investigator, 2026-07-08. Persona: rational engineer maximizing displayed profile (Stage, Flow band, Q state, headline) with minimal real improvement. Target: `RECOMMENDED_DESIGN.md` as specified, including its own §4 defenses — every attack below either bypasses a stated defense or exploits a gap §4 does not cover. All exploit mechanics are my own analysis **[inference]** unless cited; citations reuse Phase-A-verified sources (tags repeated). Prior art: `crit-gaming.md` (attacks on the OLD metric — I do not repeat attacks ALP genuinely kills: denominator games, per-dollar ratio games, default-1.0-as-max, 2000-line cap chunking, percentile zero-sum in bands).

**Threat-model delta vs the old metric.** ALP deletes the strongest old levers (spend denominator, line numerator, CI multiplier). What remains attackable is exactly what ALP promoted: (1) the Practice ladder, whose rules and thresholds are *deliberately published* — a published absolute anchor is also a published recipe; (2) the Momentum headline, whose "Improving" state is the only movement the product offers, making it the sole Goodhart target (Manheim & Garrabrant, https://arxiv.org/abs/1803.04585 [primary]); (3) the profile router, which decides *which facets are scored at all*. Perceived stakes remain nonzero despite self-view (Amazon KiroRank disbelief precedent, https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/ [independent]).

---

## A. Attack catalog, ranked by expected prevalence × severity

### A1. CRITICAL — The agent-platform profile is a fully synthetic scoring path ("ghost profile")

**Mechanics [inference].** For profile `agent-platform`, §1.4 substitutes `y := weekly practice index` for the output series. So M_out ≈ M_prac by construction, and the headline rule `M_prac > 0 AND M_out ≥ 0 → "Improving"` collapses to *practice slope alone*. Every input to that slope — sessions, tokens, runs, active days — is engineer-generated telemetry with zero external verification (the brief's hard limit #4 is precisely that no output exists). A cron job that (a) runs 1 agent run/week with jittered nonzero tokens, (b) clears τ_c on 2–6 days/week, ramping the day-count upward over the quarter, produces: Stage 3→4, M_prac > 0, M_out > 0, Q "not computable" (never `attention`, so the quality gate on the headline can never fire for this profile), headline **"Improving."** Total cost: one afternoon plus cents/week. The design's §4 defenses (τ_c floor, run+token floor, canary #3/#5) all check *levels*, which the script clears by construction.

**Bypass of stated defenses:** canary #3 (near-floor tokens / near-zero variance) is defeated by sampling payload sizes from a log-normal fitted to the published τ_c and plausible session sizes — distribution-shaped jitter is indistinguishable at the aggregate grain the canaries operate on [inference]. Canary #5 (zero-token runs) is defeated by any nonzero payload.

**Adjacent exploit — profile dodging into the ghost path.** p(e) rules are published: `agent-platform if pr_weeks ≤ 1 AND agent_share ≥ 0.5`. A pr-shaped engineer facing a "Drifting" quarter or a pending Q flag can (i) batch all merges into one calendar week (pr_weeks = 1), (ii) inflate agent_share with cheap platform sessions, and after 2 recomputes be re-routed: F and Q are no longer scored ("pr-shaped & hybrid only"), output facets read the design's own dignified "unmeasured — not zero," and the headline switches to the practice-substituted series. Bad quarter laundered into "Improving." §4's answer — "profile shopping earns 'unmeasured', never a better band" — is factually wrong for the headline: the substitution rule *does* hand the dodger a better headline. Note also `agent_share` is undefined (sessions? days? tokens? spend?) — if tokens or spend, inflation is free.

**Prevalence:** medium (agent-platform population + any pr-shaped engineer with a bad quarter). **Severity: CRITICAL** — it is the only path where the *entire* displayed profile is synthesizable, and it targets exactly the population the design claims to serve best.

**Unfairly penalized by the defense-as-designed:** honest agent-platform engineers, who share a scoring path with the easiest fake — if canaries or word-of-mouth taint the profile, their legitimate "Improving" reads become disbelieved (guilt-by-path) [inference].

**Patch:** until acquisition #1 (run→caller identity) lands, the agent-platform profile must not emit "Improving." Replace the M_out substitution with `M_out := not computable` and add a headline state: *"Practice trend up — output unmeasured."* Honest, still motivating, removes the synthetic headline entirely. Additionally: facets should compute whenever their own data floor is met, regardless of profile (profile changes emphasis, never suppresses a computable F/Q) — this kills the dodge, because batching merges into one week no longer un-scores Flow and Quality. Define `agent_share` on *meaningful days*, not tokens/spend.

---

### A2. CRITICAL — Practice theater with distribution-shaped jitter defeats every canary, and the Stage-4 canary gate is unimplementable as specified

**Mechanics [inference].** The ladder is fully deterministic on published, calendar-verifiable counts (that is its design goal). The complete Stage-4 recipe: 2 jittered sessions/week/class (C1), 5 days/quarter in a second class (C2), 10 days touching CODING_AGENT/AGENT_PLATFORM (C3), any recent day (C4). Every threshold is published, τ_c is published, and quarterly τ_c refreshes are published — the script updates its payload each quarter. The §4 defense rests on three legs, each of which fails:

1. *"Faking cost ≈ honest-use cost"* — false. Honest use costs attention and workflow change; the script costs nothing after setup and never has an off week. The equality holds only for dollar cost, which ALP correctly stopped caring about — but the relevant currency (engineer attention) is where the costs diverge by orders of magnitude.
2. *Canary #3* — detects near-floor/low-variance distributions. Defeated by jitter (A1). A stronger canary family exists and is absent: **timing regularity** (cron signatures have near-zero entropy in time-of-day and inter-session intervals; humans do not) [inference]. Even this is spoofable with randomized scheduling, but it raises scripting effort meaningfully.
3. *"No integrity-canary flag" as a Stage-4 gate* — **spec contradiction**: §1.8 states canaries are "org-side aggregate monitoring, never per-person alerts." An aggregate signal cannot gate an individual's stage. As written, the Stage-4 gate is either unenforceable (canaries stay aggregate → clause is dead text) or a stealth per-person accusation system (violating the design's own privacy stance and inviting false positives on honest terse prompters near P10). The design's single anti-theater enforcement point does not exist in implementable form.

**Prevalence:** high in weak form (nobody needs a script — "open the tool twice a week and paste anything" is ambient behavior once the ladder is published; cf. fabricated-task gaming at Amazon within months [independent, Fortune above]). Scripted strong form: low-medium. **Severity: CRITICAL** when combined with A3 (theater ramp drives the headline); MAJOR standing alone (bounded to a private stage label — the design prices this correctly *only* for the standalone case).

**Unfairly penalized:** honest low-frequency deep workers whose real usage pattern looks worse than theater; terse prompters near the τ_c floor if canaries are ever given per-person teeth without care.

**Patch:** (i) resolve the contradiction explicitly — either delete the canary clause from Stage 4 and accept bounded theater, or define a non-accusatory per-person state ("stage under data-quality review") with published trigger criteria; (ii) add timing-entropy canaries alongside token-level ones; (iii) make Stage 4 additionally require *any* externally-verifiable event (a merged PR in an agentic-tool-active week, or — post-acquisition #1 — a non-self agent caller), so the top rung alone cannot be reached purely from self-generated telemetry.

---

### A3. MAJOR — "Improving" via practice ramp + flat output: the volume gate doesn't gate the practice side

**Mechanics [inference].** §1.4's volume gate blocks output-only inflation from yielding "Improving" (`M_out > 0 AND M_prac ≤ 0 → Steady (output up)`). There is no mirror gate: `M_prac > 0 AND M_out ≥ 0 → Improving` means a scripted or half-hearted practice ramp **plus merely not-declining output** earns the top headline. The design admits this residual (§4.1 "residual attack") but underprices two aspects: (a) the ramp is *renewable* — the 12-week rolling window means a sawtooth (ramp 12w, coast/decay, re-ramp) re-earns "Improving" roughly every other quarter, at the cost of privately reading "Drifting" in coast phases; (b) it composes with A2's zero-cost theater, so the "multi-front sustained effort" §2.1 claims is really one front (practice telemetry) plus abstaining from output decline.

**Prevalence:** medium-high (the weak, unscripted form is just "use the tools a bit more visibly this quarter"). **Severity: MAJOR** — it devalues the headline, the product's centerpiece.

**Unfairly penalized (inversion):** the design's own best users. An engineer already at 6–7 meaningful days/week has M_prac ≈ 0 by ceiling and can *never* read "Improving" via practice; their best case is "Steady (output up)." The headline structurally rewards having been mediocre at baseline and denies its top state to sustained excellence — a known pathology of self-vs-self trend metrics, aggravated here because Practice has a hard ceiling [inference].

**Patch:** require corroboration across *data sources*, not facets sharing a source: "Improving" ⇐ M_prac > 0 AND M_out ≥ 0 AND (Q computable with n ≥ 5 checked PRs OR Flow band ≥ prior quarter). Add a ceiling state — "Sustained high practice" — that outranks "Steady" in display copy so ceiling-dwellers have a headline to keep, removing both the inversion and part of the sandbag incentive (A4).

---

### A4. MAJOR — Baseline sandbagging and merge-timing sawtooth (the cheap Momentum lever)

**Mechanics [inference].** §4 prices "quarter-long sandbag = high cost." That is true only for *work* sandbagging. Two cheap variants:

1. **Merge-timing banking.** Momentum's y-series buckets by *merge week* (`week_of(p.merged_at)`, §5.3). Keep working normally; delay pressing merge so completions land back-weighted in the window. Theil–Sen over 12 points reads a positive slope from a back-loaded step. Cost ≈ zero — merge delays of 1–3 weeks are indistinguishable from ordinary review latency. Rolling-window decay forces a sawtooth rather than a permanent gain, but each release phase is a fresh "Improving" screenshot.
2. **New-hire / post-reset baseline seeding.** After every regime reset (HR change, ≥4 away-weeks, new hire), the baseline restarts. Rational play: go visibly light on tooling and merges during "RECALIBRATING," then ramp — a guaranteed "Improving" first quarter. Regime resets convert the design's fairness feature into a scheduled sandbag opportunity, and role changers learn this once.

**Prevalence:** medium (variant 1 is near-ambient; variant 2 is folk knowledge within a year). **Severity: MAJOR.**

**Unfairly penalized:** engineers who merge when work is ready (their honest cadence yields flatter slopes than the banker's step); genuinely struggling new hires whose slow ramp reads identically to strategic sandbagging, eroding the state's meaning.

**Patch:** compute the output series on PR *opened* week, not merged week — opened_at is standard GitHub metadata and a near-zero-cost acquisition (add to §6 list); delaying merges then moves nothing. For resets: suppress "Improving" for the first full window post-reset, displaying "ramping — baseline forming" instead, and compare new-hire ramps only against a frozen cohort onboarding curve.

---

### A5. MAJOR — The Quality floor never binds on a rational engineer (evidence-starvation + iterate-to-green + revert laundering)

**Mechanics [inference].** Q can only cap, and it caps only on *confident* outliers: `Wilson95_upper(clean, checked) < median`. Three compounding evasions:

1. **Evidence starvation.** With few checked PRs the Wilson upper bound is wide and cannot fall below the median — an engineer with a genuinely bad clean rate but n ≤ ~8 checked PRs is mathematically unflaggable, yet displays **"Clear — no anomalies,"** the same state as a 40-PR spotless record. The old fake-1.0 default is abolished, but its epistemic twin survives: low evidence renders the *best displayable state*. Keeping PRs off CI paths (the old C1 attack) now serves this directly; the share-of-checked canary sees the aggregate, not the person, and has no defined consequence.
2. **Iterate-to-green saturation** still neutralizes flag_CI for everyone (conceded in-design; Cui et al.: build volume +38%, success rate flat-to-negative, https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 [independent]).
3. **v1 revert laundering.** flag_revert keys on branch/title regex. If linkage attributes the revert to the *reverting* author rather than the *reverted* PR's author, a reciprocal pair (I land your reverts, you land mine) hides both parties' failures; even solo, "fix-forward in new files" evades both revert regex and ≤21d same-file retouch (design concedes residuals). Delay-past-day-21 defeats flag_churn at zero cost; the bunching canary is aggregate-only.

Net effect: the only outcome-flavored gate on the headline (`Q_state = attention → "Mixed"`) is avoidable at near-zero cost, so the headline is in practice gated by nothing but self-generated telemetry and volume slopes. **Severity: MAJOR** (it silently converts ALP from "practice + trajectory with a quality floor" into "practice + trajectory"). **Prevalence:** high for 1–2 (ambient), low-medium for 3.

**Unfairly penalized:** engineers in heavy/flaky-CI repos remain the only people Q can confidently flag (their n is high and their repo prior is noisy); prolific engineers generally — flaggability rises with evidence, so doing more checked work strictly increases flag risk while doing less guarantees "Clear" [inference].

**Patch:** display evidence with the state — "Clear (low evidence: 4 checked PRs)" vs "Clear (34 checked PRs)" — and gate "Improving" on Q being *computable at n ≥ 5* (else headline reads "Improving (quality unmeasured)"); specify both-legs revert linkage as keyed to the **reverted PR's author**, explicitly; add reciprocal-revert-pair detection to the canary list (cheap once revert linkage exists).

---

### A6. MAJOR — D(c) is a live peer statistic: the victim channel the design claims to have deleted

**Mechanics [inference].** §1.0 declares "Never: live peer percentiles," and §2.2/§4 sell frozen anchors as "no engineer's display moves because a colleague gamed." But Momentum's headline subtracts a **live cohort median slope** D(c) computed each recompute from current peers (§1.4). Consequences: (a) *victim channel restored* — during a mandate-driven adoption wave, cohort median output slopes go positive, so an honest engineer's flat quarter reads "Drifting" with zero behavior change on their part; equally, widespread PR-splitting inflation by others (A7) raises D(c) and deflates everyone honest; (b) *cartel channel* — in a 20-person fallback cell, slopes cluster near zero, so a handful of coordinated sandbagger colleagues can drag the median down, gifting themselves and free-riders positive detrended slopes. Internal inconsistency, gaming vector, and fairness regression in one.

**Prevalence:** low as deliberate cartel; **high as ambient victim channel during adoption waves** (the exact condition the org is in). **Severity: MAJOR.**

**Unfairly penalized:** late-but-honest adopters in fast-moving cohorts; small/fallback cells where one team's seasonality *is* the median.

**Patch:** apply the design's own frozen-anchor principle to detrending — subtract a **frozen prior-quarter cohort drift constant** (published with changelog) instead of a live median; accept one quarter of staleness. Keep displaying raw alongside. In cells n < ~30, detrend by the function-level or org-level frozen drift only.

---

### A7. MAJOR — PR splitting survives into ALP as a triple-dip (Flow band + M_out + headline)

**Mechanics [inference].** Splitting one change into k chunks of 10–P75 lines: (i) raises F (each chunk in-band; F is a share, but splitting converts out-of-band PRs into in-band ones — the design's "share, so PR count never raises it" answers count-padding, not *conversion*); (ii) raises M_out — y = log1p(PRs) + log1p(LOC): splitting multiplies the PR term while LOC is unchanged, so y strictly increases; (iii) with any practice signal ≥ flat, feeds "Improving." §4's answer — "gamed behavior ≈ desired behavior" — holds for genuine small-batch *development* but not for cosmetic slice-at-merge-time splitting, which shifts cost to reviewers without any of the small-batch benefits (Faros: review time +91% under AI PR inflation, https://www.faros.ai/blog/ai-software-engineering [vendor]). Also the "unpublished band" defense is illusory: **any engineer can compute their own repo's trailing P75 from git log** — the band is secret only from people who can't run one command [inference].

**Sub-attack — P75 poisoning in small repos:** in a repo where one engineer authors most PRs, the trailing-26w P75 is self-set; land a few huge PRs pre-freeze, and next quarter everything you do is "in band." No minimum-PR-volume fallback is specified for repo_band. **Sub-attack — repo shopping:** land work in repos with generous frozen bands.

**Prevalence:** high (ambient; cloaked as DORA-blessed small batches, per crit-gaming N1). **Severity: MAJOR.**

**Unfairly penalized:** engineers whose legitimate unit of work is large (migrations, codegen, data eng) — now hit twice, in F and in the headline path via reviewers' slowed turnaround; solo maintainers of small repos who *don't* poison their band get compared against poisoned bands elsewhere in the same frozen quintile pool.

**Patch:** replace the PR-count term in y with **PR-days** (distinct days with ≥1 merge) — same-day salami slices then add nothing to Momentum; add a same-day-multi-merge-to-same-repo integrity canary; give repo_band a minimum-evidence rule (repo PRs < 30 in 26w → inherit function-level band) so solo repos can't self-set; note honestly that F-side splitting is unfixable without diff content (declared residual).

---

### A8. MINOR — Selective abstention beats light usage (the C1 active-week cliff)

**Mechanics [inference].** `active_week = any PR event OR any AI session`; C1 = share of active weeks with ≥2 meaningful days. A week with exactly 1 meaningful day *lowers* C1; a week with zero sessions and zero PR events vanishes from the denominator. So one light AI day is strictly worse than total abstention — a perverse cliff. Rational play: never open a tool unless you'll clear 2 days that week (or script it, per A2). Ambient effect: meetings-heavy or on-call weeks teach engineers to go fully dark.

**Prevalence:** medium (once noticed). **Severity: MINOR** (bounded to one component). **Unfairly penalized:** honestly-light-but-real users — on-call rotations, meeting-heavy leads — whose organic pattern is many 1-day weeks; they ladder-stall below engineers who use AI less but lumpier.

**Patch:** partial credit (1 meaningful day = 0.5 toward the week test), or define the denominator on employment-active weeks once PTO/leave data (acquisition #9) lands.

---

### A9. MINOR — Suppression and threshold-edge management

1. **INSUFFICIENT-OBSERVATION suppression** [inference]: a bad quarter can be traded for the provisional state by holding active weeks < 6 — expensive (requires suppressing PR events too) but available; W1's cliff logic survives in softened form. **Severity: minor.** *Patch:* show the provisional stage computed over whatever exists, labeled wide-uncertainty (the design half-does this — make the provisional stage always visible, never blank).
2. **Frozen-cutpoint edge camping** [inference]: published frozen quintile cutpoints for F invite managing the EB-shrunk share to sit just above a band edge; hysteresis slows but doesn't stop it. Canary #1 covers PR-size pile-up only, not share-at-cutpoint pile-up. **Severity: minor.** *Patch:* add band-edge pile-up to the canary list.
3. **tool_class lobbying** [inference]: C3 counts CODING_AGENT days; getting a cheap completion tool classified as CODING_AGENT via the quarterly config review is a social attack on the one named owner. **Severity: minor.** *Patch:* publish classification criteria + change log (the table is already owned and published; add rationale requirements).

**Unfairly penalized (this family):** engineers near boundaries by honest chance, who get canary-suspicion externalities from campers.

---

### A10. MINOR (low prevalence, high structural impact) — Validation-loop poisoning

**Mechanics [inference].** §1.6 demotes any threshold whose sustained transitions show null-or-negative within-person deltas for 2 consecutive quarters. The transition population is self-selected and unauthenticated: (a) theater-driven transitions (A2) mechanically contribute null output deltas, dragging honest cohorts toward demotion — the gamers the loop can't see become the evidence against the ladder; (b) inversely, engineers who time genuine stage transitions to coincide with independently-planned high-output quarters (promotion pushes) inflate the "evidence: moderate" notes; (c) an AI-skeptic cohort could deliberately transition-in-name-only to force rung demotion. The loop is the design's only self-audit and it has no integrity screen on its input population.

**Severity: MINOR today, MAJOR if the loop drives real threshold governance as intended.** **Unfairly penalized:** future engineers in cohorts whose thresholds get demoted by others' theater — a delayed, collective victim channel.

**Patch:** screen transition cohorts through the integrity canaries before inclusion (aggregate-level, so privacy-consistent); report sensitivity of every validation verdict to excluding canary-flagged weeks; pre-register cohort analyses.

---

## B. The composed play — "clean sweep, ALP edition" [inference]

Cron-scripted jittered practice ramp (A2) + slice-at-merge splitting into in-band chunks (A7) + merge-timing back-loading (A4) + checked-PR starvation for a permanent low-evidence "Clear" (A5). Result: **Stage 4, Flow top band, Quality Clear, headline "Improving"** — the design's maximal profile — with zero improvement in real work, at roughly one afternoon of setup plus mild merge-cadence discipline. Every component is policy-compliant and individually indistinguishable from virtue. The §2.1 claim that the headline "requires multi-front sustained effort to spoof" fails because the fronts share one data source (self-generated usage telemetry) and one free non-action (not declining in output); the genuinely independent front (Q) never binds (A5). What the composed play *cannot* fake at launch is nothing; post-acquisitions #1–#3 it cannot fake non-self agent consumers or clean revert/churn records — which is the correct priority order and the strongest argument for accelerating those acquisitions.

## C. Ranked summary

| # | Attack | Severity | Expected prevalence | Patch exists? |
|---|--------|----------|--------------------:|---------------|
| A1 | Agent-platform ghost profile + profile dodge | **Critical** | Medium | Yes — no "Improving" from substituted output; facets score whenever computable |
| A2 | Practice theater w/ jitter; Stage-4 canary gate unimplementable | **Critical** (w/ A3) | High (weak form) | Partial — resolve spec contradiction; external-event requirement for Stage 4 |
| A3 | Practice-ramp → "Improving" (no mirror volume gate); ceiling inversion | Major | Med-High | Yes — cross-source corroboration gate; ceiling state |
| A4 | Merge-timing sawtooth; post-reset sandbagging | Major | Medium | Yes — opened_at series; post-reset "Improving" suppression |
| A5 | Q floor never binds (evidence starvation; revert laundering) | Major | High (ambient) | Partial — evidence-labeled Clear; author-of-reverted-PR linkage |
| A6 | D(c) live cohort median = restored victim/cartel channel | Major | High (ambient) | Yes — frozen prior-quarter drift constant |
| A7 | PR splitting triple-dip; P75 reconstructable/poisonable | Major | High (ambient) | Partial — PR-days term; repo_band evidence floor; F residual declared |
| A8 | C1 abstention cliff (1 day < 0 days) | Minor | Medium | Yes — partial credit / HR-based denominator |
| A9 | Suppression, cutpoint camping, tool_class lobbying | Minor | Low-Med | Yes — always-visible provisional stage; edge canaries; published criteria |
| A10 | Validation-loop poisoning | Minor→Major | Low | Yes — canary-screened transition cohorts |

## D. What survives this red team (honest inventory)

1. **Spend is untouchable** — no attack above involves dollars; deleting the denominator genuinely killed the old metric's whole §1.1 attack class.
2. **Frozen band/anchor discipline works where it's actually applied** (F cutpoints, τ_c, LOC caps) — A6 exists precisely because Momentum's detrending is the one place the principle was not applied.
3. **The volume gate works for what it was built for** — pure output inflation without practice signal cannot reach "Improving."
4. **Self-view privacy remains the strongest control**; every severity above assumes leak-adjacent perceived stakes, per the design's own pricing.
5. **The acquisition priorities (#1–#3) are validated by this red team** — every critical/major attack is materially weakened by caller identity, file lists, or revert linkage. The red-team conclusion is not "redesign" but "the launch-state gap between what the headline claims to certify and what it can verify is wide; patch the six patchable holes above and say the residuals out loud."

**Overall verdict [inference]:** ALP is an order of magnitude harder to game than the current metric — the old metric's top five exploits are all dead or demoted. But its own §4 overstates the headline's spoof-resistance: as specified, "Improving" is reachable from a single self-generated data source (A1/A2/A3), and its two enforcement mechanisms (canary gate, quality floor) are respectively unimplementable-as-written and non-binding. All six load-bearing patches are cheap and none require new philosophy — they apply the design's own principles (frozen anchors, evidence-labeled states, score-whatever-is-computable) to the places the assembly missed.

## E. Sources (all previously Phase-A-verified; none new)

- Fortune, Amazon/Meta leaderboard gaming, 2026-07-07 [independent] — https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/
- Cui et al., Mgmt Sci 2026 [independent] — https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566
- Faros AI [vendor] — https://www.faros.ai/blog/ai-software-engineering
- Manheim & Garrabrant [primary] — https://arxiv.org/abs/1803.04585
- Jellyfish tokenmaxxing [vendor] — https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/ (context for stakes-perception only)
- Internal: crit-gaming.md, crit-stats.md, RECOMMENDED_DESIGN.md (attack surface). All exploit mechanics in §A–B are this red team's own analysis [inference].
