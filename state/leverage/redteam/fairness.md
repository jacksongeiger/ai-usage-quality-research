# RED TEAM — FAIRNESS ATTACKS ON RECOMMENDED_DESIGN.md (ALP, Anchored Hybrid)

Sub-investigator: fairness advocate for agent-platform builders, heavy-CI-repo devs, review-heavy seniors, part-time/on-call-heavy months, new hires. Date: 2026-07-08. Target: `state/leverage/RECOMMENDED_DESIGN.md` (spec sections cited as §n). Everything below is my own adversarial reasoning against the spec text — labeled **[inference]** by default; external claims reuse Phase-A-verified sources with tags. Per the brief's mandate, each attack notes (a) how the *patch* could be gamed and (b) who the *attacked mechanism* unfairly penalizes.

**Attack IDs:** F1–F20, ordered worst-first within each section. Severity: **CRITICAL** = a population gets a wrong or unreachable verdict by construction; **MAJOR** = honest common behavior systematically reads worse than it is; **MINOR** = edge case, spec gap, or bounded harm.

---

## A. Design-wide criticals (hit several protected populations at once)

### F1 — CRITICAL — Stage 3 is mathematically unreachable for single-class-licensed engineers, and gets HARDER as licensing shrinks
**Mechanism.** §1.1: `C2 = Σ_c min(1, meaningful_days(c)/5) / n_classes_available(u)`, so max(C2) = 1 for every engineer. Stage 3 requires `C2 ≥ 2/n_classes`. For n_classes_available = 3 the threshold is 2/3 (need 2 of 3 classes). For n = 2 it is 1.0 — perfection required, both classes at 5+ days. For n = 1 it is 2.0 — **unreachable**. The published "TARGET stage for all roles" (§1.1) cannot be reached by exactly the population the licensed-classes denominator was introduced to protect (§4.1 row C2: "denominator = licensed classes" is the design's stated fix for "allowlist-restricted teams"). Security-restricted teams, contractor org units, and teams whose org disallows the agent platform are the plausible n<3 populations. The fairness fix inverts into the harshest penalty in the ladder. **[inference — arithmetic from the spec's own formulas]**
**(b) Penalizes:** tool-restricted teams; contractors (already commonly license-restricted per crit-fairness §2.5); anyone in an org unit with a stale/wrong `class_availability` row (default 3 makes it worse: a genuinely 1-class team with the default-3 row needs 2 classes they cannot access).
**Patch.** Replace the ratio threshold with an absolute count clipped to availability: Stage 3 requires `established_classes ≥ min(2, n_classes_available)`. One-line fix. Also alarm on `class_availability` rows that never change (stale-config canary).
**(a) Patch gamed by:** an org unit lobbying its `n_classes_available` down to 1 so members reach Stage 3 with one class — mitigate by making the config table owner-reviewed quarterly (already in §1.1) and publishing per-unit values.

### F2 — CRITICAL — Momentum's live cohort detrending re-opens the "victim channel" the design claims to have deleted — in the headline driver
**Mechanism.** §1.0 promises "frozen anchors are non-zero-sum: no engineer's display moves because a colleague gamed or adopted faster." But §1.4 defines `M_out(e) = TheilSen(y_e) − median_{e'∈c(e)} TheilSen(y_{e'})` — the cohort median slope is computed **live, every window**. During an adoption wave (the org's stated goal!), the cohort median slope rises, and an honest engineer with flat output sees detrended M_out go negative — headline falls from "Steady" to "Drifting — practice or output trending down" **with zero behavior change**. This is precisely the zero-sum/non-stationarity pathology crit-fairness §3.4 documented for percentiles, relocated into the single most prominent display. The headline gate ("M_out ≥ 0") reads the detrended value per the spec's own definition. The frozen-anchor guarantee is false for the facet that drives the headline. **[inference — contradiction internal to the spec]**
**(b) Penalizes:** everyone whose cohort adopts faster than they do — disproportionately the design's own protected classes: review-heavy seniors, on-call-heavy months, part-timers, and agent-platform hybrids, whose y-series are flat-to-thin for structural reasons (see F5–F11).
**Patch.** Headline gate uses the **raw** within-person slope (self-vs-self, as §2.2.3 advertises); show cohort-detrended slope only as unscored context, and compute D(c) from a **frozen prior-quarter** cohort slope if it must gate anything. Keep the "no detectable trend" CI state as the guard against drift-inflation instead of live subtraction.
**(a) Patch gamed by:** riding ambient org-wide drift to a raw "Improving" (the exact inflation D(c) was built to kill, crit-gaming §3). Accept the trade: at self-view stakes, a mildly drift-inflated "Improving" harms no colleague; a drift-deflated "Drifting" harms the person displayed. Canary the org-wide raw-slope median so the copy can say "note: output is rising org-wide this quarter."

### F3 — CRITICAL — Profile-routing hole: low-PR engineers with agent_share < 0.5 are routed to the FULL pr-shaped profile
**Mechanism.** §1.0: `agent-platform` requires pr_weeks ≤ 1 AND agent_share ≥ 0.5; `hybrid` requires pr_weeks ≥ 2 AND agent_share ≥ 0.3; **else pr-shaped**. An engineer with pr_weeks ∈ {0,1} and agent_share < 0.5 — e.g., a security engineer or API designer doing 60% chat / 40% agent work and merging one PR a quarter, or a pure-chat incident responder — falls through both gates into `pr-shaped`: scored on Flow (EB-shrunk to the function prior, i.e., displaying the prior, not them) and on M_out over a y-series that is zero in 11 of 12 weeks. Combined with F2, their headline oscillates Steady/Drifting on cohort noise. The population the profile system exists to protect is protected only above an arbitrary 50% share line. Additionally `agent_share` has **no defined basis** (dollars? tokens? sessions? days?) — a dollars basis misroutes anyone whose coding-CLI spend dollar-dominates despite platform-centric work, since agentic token costs vary ~30x on identical tasks (https://arxiv.org/abs/2604.22750 [independent]). **[inference]**
**(b) Penalizes:** chat-heavy non-PR roles (security review, design, incident command); agent builders below the share line; anyone whose tool dollar-mix misstates their work-mode.
**Patch.** Route on output-thinness first: `pr_weeks ≤ 1 → non-PR profile` (output facets "unmeasured") regardless of tool mix; define agent_share on **meaningful days** (the design's own least-spoofable unit), not dollars. Keep the 2-recompute hysteresis.
**(a) Patch gamed by:** withholding PRs (≤1 PR-week) to escape output scoring entirely — bounded because the non-PR profile can never display better than "practice-only" (profile shopping earns "unmeasured", never a better band, per §1.0), and the falling-PR-share canary (§1.8 #2) generalizes to falling pr_weeks.

### F4 — CRITICAL — "Practice down + output steady" ⇒ "Drifting": the headline punishes the evidence-aligned behavior of efficient seniors
**Mechanism.** §1.4 state machine: M_prac significantly negative with M_out ≈ 0 fails "both small" and lands in `else → "Drifting — practice or output trending down"`. So an engineer who **reduces** AI usage while output holds — a senior who correctly concluded AI slows them on their task mix, which is METR's central finding for experienced devs (−19% actual vs +20% believed, https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ [independent]), or an engineer who matured from many exploratory sessions to few efficient ones — gets the design's only negative label. Meanwhile the mirror case (output up, practice down) earns a neutral-positive "Steady (output up)". The asymmetry privileges usage-consistency over output in the negative direction — the exact adoption-vanity slope §1.6 exists to prevent. This is a headline-level penalty, not a Practice-facet description: §4.2.1's containment ("skeptics are not headline-penalized because the headline is self-vs-self") is **false** for any skeptic whose skepticism *grew* during the window. **[inference — state-machine trace]**
**(b) Penalizes:** seniors de-adopting for evidence-based reasons; engineers who got more efficient (fewer, better sessions); anyone whose Q3 project needed less AI than Q2's.
**Patch.** Reserve "Drifting" for `M_out` confidently negative. Practice-down/output-steady → "Steady" plus a neutral, non-valenced practice note ("your AI practice depth declined; output held — if intentional, no action needed"). This also fixes the asymmetry.
**(a) Patch gamed by:** letting practice theater lapse once a rung is banked while output coasts — acceptable: the Practice *stage* still decays via C1/C4, only the negative *headline* is withheld.

---

## B. Review-heavy seniors (mentors, tech leads, incident commanders)

### F5 — MAJOR — The `active_week`/"PR event" ambiguity makes reviewers lose either way
**Mechanism.** §1.1: `active_week(u,w) = any PR event OR any AI session`. "PR event" is undefined: authored-only, or including review activity? If **authored-only**: a senior spending most weeks reviewing, with light AI use, accrues AW < 6 → **"INSUFFICIENT OBSERVATION"** — a fully-present, heavily-contributing tech lead is labeled as barely observed. If **review-inclusive**: every review-heavy week counts as active, entering C1's denominator; weeks where AI was legitimately unnecessary (review, design, incident) drag C1 below stage thresholds → Practice demotion. Lose–lose; the spec never chooses. Faros telemetry shows AI-era senior time shifting exactly this way (review time +91% on +154% bigger PRs, https://www.faros.ai/blog/ai-software-engineering [vendor]); SPACE's rule against activity metrics used alone names this uncounted work (https://queue.acm.org/detail.cfm?id=3454124 [primary]). **[inference on the spec gap]**
**(b) Penalizes:** L6+ tech leads, mentors, incident commanders — 30–60% of whose work-mix is plausibly non-authoring **[inference, unverified]**.
**Patch.** Define: authored PR events OR AI sessions ⇒ active (C1 denominator); review events ⇒ a third state, "present — work not measurable here," which counts toward AW (killing the insulting INSUFFICIENT OBSERVATION) but is excluded from C1's denominator. Review-event *counts* are metadata-only and almost certainly in the warehouse (acquisition #4 formalizes them).
**(a) Patch gamed by:** rubber-stamp review events to farm "present" weeks — harmless, since the state neither scores nor gates anything; a review-spam canary can watch approval-latency distributions org-side.

### F6 — MAJOR — The headline contradicts the unmeasured panel: role drift into leadership reads as "Drifting" with no reset available
**Mechanism.** §1.4 regime resets fire only on **HR events** (team/function/level change) or ≥4 consecutive away weeks. The most common senior transition — becoming de-facto tech lead, taking incident command duty, ramping mentoring — changes *none* of those fields. Their y(e,w) declines across the window → M_out negative → "Drifting — practice or output trending down," while three panels below, §5.4's copy says "code review you give, mentoring… absence of signal here is not absence of value." The product asserts and denies the same fact on one screen. Under F2's live detrending the effect is amplified (IC peers' slopes rise). **[inference]**
**(b) Penalizes:** promotion-track seniors at exactly the moment the org asks more of them; also maintenance/bug-duty rotations (see F13).
**Patch.** (i) Add a self-service, rate-limited "my role changed" reset (2/year, logged, with the same RECALIBRATING display as HR resets); (ii) suppress "Drifting" whenever the engineer's review-event share of total GitHub events exceeds a published threshold, replacing it with "Output shifted toward review — not measured here."
**(a) Patch gamed by:** (i) resetting to bury a genuine decline — bounded by rate limit and by RECALIBRATING being a non-scored state (you erase a down-arrow but earn nothing); (ii) review-spam to trigger suppression — canary as in F5.

---

## C. Agent-platform builders

### F7 — MAJOR — Run-attribution ambiguity makes `meaningful()` wrong for builders in both directions
**Mechanism.** §1.1: for AGENT_PLATFORM, `meaningful` requires `runs(u,d) ≥ 1`. The spec's own #1 acquisition (§6: run → caller identity) concedes runs are not currently attributed among owner/caller. If runs attribute to the **triggering user**: a builder who spends full days writing/configuring agents without executing runs logs zero meaningful platform days → C3 = 0, Stage 4 unreachable, and for agent-platform-profile users the substituted practice index (their entire M) under-counts their build days. If runs attribute to the **owner**: scheduled/cron agents mint meaningful days while the owner is on vacation — a standing theater machine (gaming), and the vacation weeks now count "active," corrupting the pro-rating that §1.1 promises ("PTO drops out"). Either resolution is unfair or gameable; the spec doesn't resolve it. **[inference]**
**(b) Penalizes:** builders of scheduled/autonomous agents — the heaviest, most legitimate platform users (the population the brief says the current metric wrongs categorically).
**Patch.** Until acquisition #1 lands: `meaningful` for AGENT_PLATFORM = `sessions ≥ 1 AND tokens ≥ τ_c` alone (interactive presence), with runs used only in canary #5 (zero-token runs). Document that run counts are unattributed and unscored v0.
**(a) Patch gamed by:** token-burning platform sessions — same theater cost/bound as every other class (τ_c + canary #3); no worse.

### F8 — MAJOR — Practice saturation makes the agent-platform profile permanently "Steady": improvement becomes structurally undisplayable
**Mechanism.** For the agent-platform profile, M substitutes the weekly practice index for y (§1.4). A builder already using the platform daily has a practice series at ceiling → slope ≈ 0 forever → headline "Steady" forever; F and Q are unmeasured; E is phase 2. Their dashboard is one static stage, one permanent "Steady," and a cost strip on which they are a structural outlier vs the function-cohort IQR (§1.7) — agentic workflows are orders of magnitude more token-hungry [independent, arXiv 2604.22750]. The design admits a "thinner profile" (§4.2.3) but understates it: it is a profile in which **no honest action can ever improve any display**. For a motivational self-view product, that is a null product for this population — and per the ceiling-effect admission (§7.7) it eventually becomes everyone's problem, but it is the builders' problem on day one. **[inference]**
**(b) Penalizes:** exactly the "heavy, effective AI engineers" the brief names as the current metric's worst victims — again told, more politely this time, that their work doesn't register.
**Patch.** (i) Elevate acquisition #1 from "top priority" to a launch **precondition** for shipping the agent-platform profile beyond a labeled beta; (ii) meanwhile, let the E-survey's "who consumes your agents" item (§1.5) render as a self-reported, clearly-labeled consumers list on their profile — voice, not score; (iii) cost-strip cohort context for this profile compares against agent-platform users' IQR, not the function cohort's.
**(a) Patch gamed by:** self-reporting phantom consumers — labeled self-report, never scored, so inflation buys a private vanity line only.

### F9 — MAJOR — The validation loop's outcome set is PR-shaped, so its committed demotion path will demote the only scored facet non-PR cohorts have
**Mechanism.** §1.6 validates rungs against within-person deltas on merged-PR trend, PR size, CI anomalies, retouch/revert — all PR-side outcomes. For cohorts dominated by agent-platform/non-PR work, sustained Practice-stage transitions will show **null deltas on outcomes they structurally don't produce** → per the committed rule ("null-or-negative in a cohort for 2 consecutive quarters ⇒ revise the threshold for that cohort"), their Practice ladder gets demoted to a descriptive profile — stripping the one scored facet from the population with the fewest facets. The self-audit mechanism inherits the output bias it was built to catch. **[inference]**
**(b) Penalizes:** agent-platform and hybrid cohorts; long-cycle functions (ML research) whose PR outcomes lag beyond the 12w-after window.
**Patch.** Gate the demotion rule on **outcome coverage**: it fires only in cohorts where the measured outcome set covers ≥ (published %) of the cohort's work-mode (proxy: median pr_weeks). Elsewhere the loop reports "cannot validate — outcomes unmeasured for this cohort" instead of demoting.
**(a) Patch gamed by:** a cohort suppressing PRs to dodge validation — implausible as a coordinated act at self-view stakes; org-side canary on cohort pr_weeks trends suffices.

---

## D. Heavy-CI-repo and monorepo residents

### F10 — MAJOR — Q's cross-repo pooling is undefined; multi-repo engineers are compared against an ill-defined norm
**Mechanism.** §1.3 defines rate_EB with per-repo priors and a flag vs `median(rate_EB | repo tier)`, but an engineer's window typically spans several repos and the spec never says whether clean/checked pool across repos before the Wilson test, or which repo's median a pooled rate is tested against. Pooling reintroduces exactly the repo-mix bias the per-repo priors were built to remove: an engineer doing 70% of PRs in a 50-job repo blends a structurally lower rate, then gets tested against a norm dominated by their lighter-CI repo (or vice versa). P(clean) ≈ (1−f)^J arithmetic (crit-fairness §2.2: ~40pt spread from repo choice alone **[inference, illustrative]**) means the blend is repo-weights, not behavior. **[inference on spec gap]**
**(b) Penalizes:** platform/infra engineers, who touch the most repos and the heaviest CI; also anyone on a temporary tour through a flaky repo.
**Patch.** Compute the flag strictly per repo (≥5 checked PRs in that repo to be testable); Q_state = attention only if some single repo flags. Never pool across repos.
**(a) Patch gamed by:** spreading PRs thinly across repos so no repo reaches 5 checked PRs → Q "not computable" — already the design's honest state (uncomputable ≠ clean, §4.1), and canary #2 watches the checked-share.

### F11 — MAJOR — Shared-monorepo Flow bands are set by the dominant tenant; platform teams inside them read as undisciplined
**Mechanism.** §1.2's in_band ceiling is `P75(lines per PR | repo)`. In a shared monorepo the P75 is dominated by the majority tenant (product teams shipping small PRs); platform/codegen/migration teams in the same repo whose honest unit-of-work is larger sit systematically out-of-band — §5.4's copy would tell them "slice large changes" when the correct slicing already happened. Repo-grain normalization is the design's fix for repo effects (§2.2.6), but repo ≠ team in monorepos — the normalization is at the wrong grain for exactly the orgs (large, monorepo-heavy) the brief assumes. **[inference; mechanism parallel to GitClear's two-engineers-two-repos point, gitclear survey file]**
**(b) Penalizes:** monorepo platform/infra/data-migration teams; codegen owners (acknowledged in §4.1 but answered with the wrong-grain band).
**Patch.** Where path metadata exists (acquisition #2), band by repo × top-level directory; until then, band by repo × author's function-cell when the repo's weekly PR volume clears a threshold (enough data to split).
**(a) Patch gamed by:** filing PRs under a large-band directory — path-mix canary; bounded, F is never the headline.

### F12 — MINOR — Q "attention" needs 2 recomputes to clear and has no expiry or annotation path
**Mechanism.** §1.0 hysteresis applies to "any stage/band change," so a flag that fired on environment noise (flaky quarter, one bad incident week) also delays its own clearing by 2 weekly recomputes, and while present it overrides the headline to "Mixed" (§1.4) — for heavy-CI residents the false-positive cost is a quarter of suppressed "Improving." No appeal, context, or expiry exists in the spec. **[inference]**
**Patch.** Hysteresis on entering attention, none on exiting; flags auto-expire when the trailing test stops firing; allow a free-text self-annotation ("incident remediation") displayed beside the flag — self-view product, so annotation abuse costs nothing.
**(a) Patch gamed by:** annotating away real problems — the flag still displays; only the story changes.

---

## E. Part-time, on-call-heavy months, leave-takers

### F13 — MAJOR — The absolute day-count anchors assume a full-time calendar; every threshold binds harder per hour worked part-time
**Mechanism.** Graft 1's core virtue — "calendar-verifiable counts" (≥2 meaningful days/active week; ≥6 days/trailing-4-weeks; 5 days to establish a class; 10 agentic days; C4's 4-week recency) — is also its core fairness bug: absolute day counts don't pro-rate. A 60%-FTE engineer (3 days/week) must clear the same ≥2 days/week (67% of their working days vs 40% for full-timers) and the same 10 agentic days per quarter out of ~36 working days vs 60. The active-week denominator pro-rates *whole absent weeks* only; it does nothing for partial weeks. Frozen absolute anchors deleted the victim channel by hard-coding a full-time workweek. Presence-proxy penalties on hours-not-present are the documented failure class (crit-fairness §2.5; copilot-studies §4). **[inference]**
**(b) Penalizes:** part-timers — disproportionately parental/disability/return-to-work accommodations (protected-class adjacency); 4-day-week teams; chronic on-call rotations with fragmented days.
**Patch.** Where HR carries FTE fraction (it does, for payroll): scale day-count thresholds by FTE (≥2 → ≥⌈2·FTE⌉, 10 → ⌈10·FTE⌉), publish the scaled anchor on the engineer's own dashboard — still absolute, still calendar-verifiable, just *their* calendar. Where FTE is unavailable, fall back to observed active-days capacity.
**(a) Patch gamed by:** nothing new — FTE is an HR fact the engineer can't set; a full-timer can't claim 0.6.

### F14 — MAJOR — C4 recency demotes Stage 3 after an ordinary 2–3 week vacation; the away-week reset only fires at ≥4 weeks
**Mechanism.** §1.1: `C4 = weeks with ≥1 meaningful day in last 4 observable weeks / min(4, AW)`. The numerator window is *observable calendar* weeks; leave weeks are not "active" but are observable. A 3-week vacation ending at AS_OF leaves at most 1 qualifying week in the last 4 → C4 ≤ 0.25 < 0.5 → Stage 3 lost (hysteresis then delays re-promotion ~2+ more weeks after return). The §1.4 regime reset ignores it (< 4 consecutive away weeks), and it's a Practice-facet rule anyway. The design's claim "PTO/leave drop out via active-week denominator" (§4.1) is true for C1 and false for C4. European-length annual leave triggers a visible demotion on return. **[inference — direct rule trace]**
**(b) Penalizes:** anyone taking 2–3 weeks of contiguous leave; on-call SREs whose recovery weeks follow rotations.
**Patch.** Define C4's numerator window as the last 4 **active** weeks (consistent with C1's denominator philosophy), or exclude zero-activity weeks from the 4-week lookback.
**(a) Patch gamed by:** going dark for weeks so stale usage stays "recent" — bounded: dark weeks also shrink AW toward INSUFFICIENT OBSERVATION and freeze Momentum's usable weeks.

### F15 — MAJOR — An on-call/incident month drags the Momentum slope; incident work is named unmeasured but still moves the headline down
**Mechanism.** On-call weeks are *active* (PR events from hotfixes, AI debugging sessions) with structurally low y(e,w). Theil–Sen absorbs 1–2 such weeks, but a real rotation (1 week in 4 = 3 of 12 weeks) near the window's end biases the median pairwise slope negative → "Drifting," compounded by F2's detrending. Incident response appears in the "Not visible to us" panel (§5.4) while actively depressing the headline — same contradiction class as F6. flag_review (v1) doubles the hit: break-glass emergency merges are precisely "merged without review" → Q attention → "Mixed." **[inference]**
**(b) Penalizes:** SREs and on-call-heavy teams as a *function* (their cohort helps only if the whole cohort shares the rotation phase); incident responders in ordinary product teams (cohort doesn't help at all).
**Patch.** (i) Let engineers mark weeks as "on-call/incident" (self-service, rate-limited, displayed) — marked weeks are excluded from the y-series like away weeks; (ii) exempt merges labeled by the incident/deploy tooling from flag_review (metadata exists in any org with incident management).
**(a) Patch gamed by:** marking bad weeks "on-call" to launder a genuine decline — rate-limit (e.g., ≤3 weeks/quarter self-marked), display the marks, and canary org-side on self-mark rates; where PagerDuty-like rotation data exists, prefer it over self-marking.

### F16 — MINOR — FTE-fraction changes and phased returns are not regime events
**Mechanism.** §1.4 resets on team/function/level changes and ≥4 away weeks. Going from full-time to 80% (parental phase-back, accommodation) is an HR event that changes none of those fields → mechanical y-drop → "Drifting" for a legally protected schedule change; a phased return after leave (reset fired) builds the new baseline on reduced-hours weeks, then reads the return to full-time as an unexplained surge. **[inference]**
**Patch.** Add FTE-fraction change to the HR reset triggers (the table in §5.1 already ingests `hr` effective-dated rows; add the column).
**(a) Patch gamed by:** nothing — HR-recorded facts.

---

## F. New hires and interns

### F17 — MAJOR — The onboarding ramp contaminates the Momentum baseline: typical new hires get "Improving" then a near-guaranteed "Drifting" at months 4–6
**Mechanism.** After the 8-week Building-baseline state, the rolling 12-week window spans the steepest part of the ramp → early "Improving" (flattering, noise). At months 4–6 the window is ramp-inflected on the left and normal-productivity on the right → slope flattens/negative → "Steady" then plausibly "Drifting," exactly when the new hire has successfully stabilized. The design resets on hire (Building baseline) but never models that the first measurable baseline is *unrepresentative by construction*. A predictable negative arc delivered to every new hire in their first half-year is a motivational anti-pattern for a product whose stated purpose is motivation. F2's detrending softens this only if the cohort contains many simultaneous new hires — usually false. **[inference]**
**(b) Penalizes:** every new hire, on a schedule; worst for those hired into slow-ramp (brownfield, high-context) environments — METR's population [independent].
**Patch.** Extend RECALIBRATING semantics: for 24 weeks post-hire, suppress "Drifting" (floor the headline at "Steady — post-onboarding settling") and say so in copy. Descriptive facets (Practice stage, Flow band) display normally from week 1 as designed.
**(a) Patch gamed by:** nothing — hire date is an HR fact; a 24-week grace on a *negative label only* buys no positive display.

### F18 — MINOR — Interns are unspecified end-to-end
**Mechanism.** A 10–12 week internship ≈ the 12-week window minus the 14-day AS_OF lag minus the 8-week Momentum minimum: an intern's headline arrives in their final week or never; their function×level cohort cell is undefined (interns often carry no L-band) so F/M context bands fall back to org-grain; the brief's old design at least named the intern bucket. **[inference]**
**Patch.** Interns get a fixed descriptive profile (Practice stage + cost strip + unmeasured panels), no Momentum, no headline — declared upfront: "your program is shorter than our measurement window."
**(a) Patch gamed by:** n/a (nothing scored).

### F19 — MINOR — License-provisioning lag undercounts new-hire breadth
**Mechanism.** C2's denominator counts classes the org unit is *licensed* for; new hires typically get seats provisioned over weeks (chat day 1, CLI week 3, platform month 2). The unit is licensed, the person has no seat → breadth structurally undercounted through week ~8, delaying Stage 2/3 beyond what their behavior warrants. **[inference]**
**Patch.** Count a class in n_classes_available(u) only from the user's first seat-assignment date (seat data exists wherever spend-per-user exists).
**(a) Patch gamed by:** delaying one's own seat to shrink the denominator — backwards (fewer classes ⇒ harder Stage 3 post-F1-patch is false; with F1's patch it's min(2, n), so n=2 vs 3 is neutral-to-easier by one class at 5 days) — mitigated by F1's `min(2, n)` form making the marginal gain ≈ one 5-day class; acceptable.

---

## G. Cross-cutting spec contradictions with fairness weight

### F20 — MAJOR — Stage 4's "no integrity-canary flag" condition contradicts "canaries are never per-person" and re-penalizes the terse users τ_c was tuned to protect
**Mechanism.** §1.1 Stage 4 requires "no integrity-canary flag"; §1.8 says canaries are "org-side aggregate monitoring, never per-person alerts." Both cannot hold: either canaries silently gate an individual's rung (a per-person adverse decision with no notice, no appeal, invisible criteria — procedural-unfairness by construction), or the Stage 4 condition is uncomputable. And canary #3 (near-floor session tokens / low daily-token variance) profiles exactly the honest terse habitual prompter that §1.1 says τ_c's P10 placement protects — the design protects them at the meaningful() gate, then un-protects them at the top rung. Also unpatched staleness: τ_c is frozen quarterly on trailing token distributions while models/pricing drift fast (token efficiency improves), so honest sessions on newer efficient models sink toward the stale floor and *look* like theater. **[inference — internal contradiction + non-stationarity reasoning; model-efficiency drift direction unverified for this org]**
**(b) Penalizes:** disciplined terse users; early adopters of efficient models; and (per the contradiction) anyone flagged by an invisible criterion.
**Patch.** Delete the canary condition from Stage 4 (canaries inform threshold retuning and org-side investigation only, as §1.8 states); recompute τ_c per model-generation or shorten its freeze to monthly with a published changelog.
**(a) Patch gamed by:** theater users reaching Stage 4 unimpeded — accept it: §4.1 already prices in practice theater as "bounded, private label"; a hidden per-person tripwire is worse than the theater it catches at self-view stakes.

---

## Severity roll-up

| ID | Severity | One line |
|---|---|---|
| F1 | CRITICAL | Stage 3 unreachable/harder for license-restricted teams (C2 arithmetic inversion) |
| F2 | CRITICAL | Live cohort detrending in Momentum = zero-sum victim channel in the headline; contradicts frozen-anchor guarantee |
| F3 | CRITICAL | Routing hole sends low-PR chat-heavy engineers to full pr-shaped scoring; agent_share basis undefined |
| F4 | CRITICAL | Practice-down/output-steady ⇒ "Drifting" punishes evidence-aligned de-adoption (METR seniors) |
| F5 | MAJOR | active_week ambiguity: reviewers get either INSUFFICIENT OBSERVATION or Practice demotion |
| F6 | MAJOR | Role drift (IC→lead) has no regime reset ⇒ "Drifting" while the panel calls the work invisible |
| F7 | MAJOR | Agent-run attribution ambiguity: builders undercounted or vacation-theater minted |
| F8 | MAJOR | Agent-platform profile saturates ⇒ permanently "Steady"; no honest action improves any display |
| F9 | MAJOR | PR-shaped validation loop will demote non-PR cohorts' only scored facet |
| F10 | MAJOR | Q cross-repo pooling undefined ⇒ repo-mix bias returns for multi-repo/platform engineers |
| F11 | MAJOR | Shared-monorepo Flow band set by dominant tenant; platform teams read undisciplined |
| F13 | MAJOR | Absolute day-count anchors don't pro-rate ⇒ structural penalty on part-time/accommodations |
| F14 | MAJOR | C4 recency demotes Stage 3 after a 2–3 week vacation (reset needs ≥4 weeks) |
| F15 | MAJOR | On-call/incident months drag Momentum + trip flag_review; headline contradicts unmeasured panel |
| F17 | MAJOR | Onboarding-ramp baseline ⇒ predictable "Drifting" for new hires at months 4–6 |
| F20 | MAJOR | Stage 4 canary gate contradicts "never per-person"; re-penalizes terse users; stale τ_c |
| F12 | MINOR | Q attention: exit hysteresis delays exoneration; no expiry/annotation |
| F16 | MINOR | FTE-fraction changes/phased returns are not regime events |
| F18 | MINOR | Interns unspecified; window ≥ internship length |
| F19 | MINOR | License-provisioning lag undercounts new-hire breadth |

**Meta-observation [inference].** Four of the five populations I represent are harmed through the same two channels: (1) the headline state machine treats "no measurable output movement" as evidence of decline whenever anything else moves (F2/F4/F6/F15/F17), and (2) absolute calendar anchors — the design's flagship interpretability graft — hard-code a full-time, PR-authoring, fully-licensed engineer as the reference human (F1/F13/F14/F19). Both channels are patchable without abandoning the chassis: gate "Drifting" on confidently-negative *raw* output only, and scale/clip every absolute anchor to the individual's own licensed-and-scheduled capacity. None of the patches require data the org lacks; F1, F3, F4, F14 are one-line spec fixes.

## Source key (all previously Phase-A-verified; reused here)
METR RCT [independent] https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ · SPACE [primary] https://queue.acm.org/detail.cfm?id=3454124 · Faros review-load [vendor] https://www.faros.ai/blog/ai-software-engineering · Bai et al. token variance [independent] https://arxiv.org/abs/2604.22750 · DORA 2025 [primary] https://dora.dev/dora-report-2025/ · crit-fairness.md / crit-gaming.md / crit-stats.md (internal, this repo). All other content: **[inference]** against the spec text.
