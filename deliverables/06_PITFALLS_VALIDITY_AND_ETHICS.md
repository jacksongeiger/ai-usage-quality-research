# 06 — Pitfalls, Validity, and Ethics

This deliverable is the framework's conscience. It catalogs the ways a measure of "AI usage quality" can be wrong, gameable, or harmful — and the concrete design choices that keep it valid, fair, and genuinely useful *for the engineer receiving it*. It is also the record of what the red-team passes found and fixed.

## 1. The four validity traps

### Trap 1 — Goodhart / Campbell (gaming)
[30][31]: "When a measure becomes a target, it ceases to be a good measure"; the more a quantitative indicator drives decisions, the more it corrupts the process it monitors. Documented engineering gaming [13]: LOC padding, micro-commits, force-push to fake cycle-time authorship, assertion-free coverage tests, redefining "critical" to dodge change-failure triggers.

**Where our framework is exposed, and the mitigation:**
| Metric | How it'd be gamed | Mitigation in our design |
|---|---|---|
| % AI-assisted / acceptance | Route trivial edits through AI; accept-then-rewrite | Volume gate (never alone); pair with survival |
| Token usage / consistency | Generate daily filler tokens | Demoted to context; volume gate |
| First-pass CI | Run locally first (actually *good*) | Treat as personal trend, low stakes |
| Review friction | Pick lenient reviewers / under-review | Multi-reviewer aggregation; trend-vs-baseline |
| Tool breadth | Adopt tools for show | Anchor to durability, not tool count |

**The structural defense**: no single score is ever exposed as a target; recommendations require **signal agreement** across method types; everything is **engineer-only** (no external stakes → far less corruption pressure, per Campbell [31]).

### Trap 2 — Construct validity (activity ≠ quality)
We want the latent construct *usage quality* but telemetry only yields *activity* [32][33]. High volume is consistent with *both* skillful leverage and thrashing — a **discriminant-validity failure** if we let volume stand in for quality.

**Mitigation**: (a) explicit nomological network (three pillars, deliverable 01 §2); (b) **convergent validity** — require agreement across telemetry + diff-metadata + self-report; (c) **discriminant validity** — structurally down-weight all volume metrics; (d) **content validity** — include the hard-to-see parts of the construct (durability, knowing when *not* to use AI) so we don't measure a sliver and call it the whole.

### Trap 3 — Correlation ≠ causation & confounds
Individual variance dominates developer output; tenure, seniority, role, repo, and task type are heavy confounders [37]. "More AI → better outcomes" is correlational; reverse causation (productive devs adopt more) and common causes (easy tasks) are unaddressed.

**Mitigation**: (a) **trend-vs-personal-baseline**, never cross-person — neutralizes static confounders; (b) **confound gate** — drop a tier and name the confound when a pattern is plausibly explained by task-type/tenure/repo shift; (c) **causal humility** — all findings framed as association, never "AI caused X."

### Trap 4 — Self-report unreliability (the perception gap)
METR [17]: the same devs were **19% slower** while believing they were **20% faster**. Self-report cannot be a *truth* signal for speed — yet it is indispensable for *experience* (DevEx [4]).

**Mitigation**: self-report measures experience/friction/satisfaction only; never treated as ground-truth speed; the perception gap is surfaced *to the engineer* as a calibration insight (deliverable 03 D1), not a correction.

## 2. The surveillance-vs-enablement problem

Individual developer metrics are widely and rightly warned against [3][4][15] because they slide into surveillance, ranking, and performance management. Our entire output goes to individuals — so this is the central ethical risk, not a footnote.

**Design choices that keep it enablement, not surveillance:**
1. **Audience-lock (architectural, not a promise).** Data and recommendations are visible to the engineer only. No manager, HR, review, or compensation linkage — ever. (constraint #2; [15] "a popular reason CTOs want to measure productivity is to identify who to fire.")
2. **Opt-in + agency.** The engineer can enable/disable each signal, see exactly what's collected and why, and delete their data. GDPR/Article-29 floor: necessity, proportionality, data-minimization, prior clear information, DPIA for systematic monitoring [34].
3. **Trend-vs-baseline, never peer ranking.** No leaderboards, no percentiles against colleagues [3][37].
4. **Aggregation/k-anonymity for any org rollup.** No individual re-identifiable in team reports [36].
5. **Coaching tone, not grading.** Recommendations read as "you might try…" with a concrete next action and the *why* — never a score/grade. Punitive framing both harms and *invalidates* (people game and mis-report under stakes [31][36]).
6. **Design for the observer effect [35], deliberately.** Measurement will change behavior — so aim it at *good* behavior (reflection, verification, calibration) and tell the engineer the metric exists to help them think, not to catch them. Psychological safety is a precondition for the honest self-report the framework depends on [36].
7. **Affirm, don't only flag.** Healthy usage (deliverable 03 pattern C) gets recognition. Enablement includes telling people what's working.

## 3. The no-code guarantee (constraint #1)
Audited every cycle. The one casualty: a "complexity-trend of changed code" metric was proposed and **killed** in red-team #1 because it requires parsing diff contents. Also out-of-bounds: prompt-text content analysis (code-adjacent, not in our data) and keystroke/screen capture (ethics). Everything retained runs on metadata, status, counts, timestamps, categorical labels, and self-report. Revert/hotfix detection uses commit *message* metadata (subject/body) [42], not source — flagged as text-lite but acceptable; review-feedback sentiment touches *feedback text*, not source code, and is gated as Borderline (deliverable 04 #10).

## 4. Honest limitations (what this framework cannot do)

> **The biggest one, stated first — this framework is unvalidated.** It is a synthesis of the literature, not a tested model. The convergent-validity argument (deliverable 01 §2) is *theoretical*: we have **no empirical evidence that our specific proxies actually correlate with any ground-truth measure of "good AI usage,"** because no such validation study has been run here. Everything downstream — the pillars, the tiers, the recommendations — is a *well-reasoned hypothesis*, not a proven instrument. Before any real deployment, it needs a validation step: e.g., correlate the proxy signals against an independent criterion (expert/peer assessment of a sample of engineers' AI usage, or self-rated + manager-blind expert review of anonymized workflow recordings — *not* code), and prune metrics that don't converge. Treat the current artifact as the design to be validated, not the answer to be trusted. (inference)

- **The AI-attribution gap (the most important technical limitation).** The strongest pillar — durability of *AI-assisted* code — usually can't isolate AI-authored lines. Vendor delivery-analytics tools admit they can't distinguish AI vs human lines [18]; only Sourcegraph ties survival to accepted suggestions natively [25]. For most orgs, Pillar-3 signals measure *overall* durability that AI merely contributes to (precondition P4). Real but coarser; must be labeled "overall" vs "AI-attributed" honestly.
- **Durability can reward timidity.** High survival / low churn can mean *unambitious, trivial* work, not skillful AI use. Durability is a quality signal only alongside non-trivial work — pair it with PR-size/ticket context, never read it alone.
- **Recommendation efficacy is unproven.** We recommend "verification-first," "smaller diffs," etc. on *plausibility*, with no evidence these actually improve the measured signals for a given engineer. Recommendations should be A/B-reflected by the engineer ("did this help?"), not assumed effective.
- **It cannot judge whether the code is good.** It infers *usage* quality (durable leverage), not *output* correctness. (inference)
- **A simpler alternative may capture most of the value.** A skeptic's fair challenge: a good anonymized survey + structured self-reflection might deliver ~70% of the benefit at ~10% of the effort. The objective-metadata edifice earns its keep only if validation (above) shows it adds signal beyond self-report. Until then, surveys are the safer first investment (deliverable 04 #2). (inference)
- **"Knowing when NOT to use AI" is only weakly proxied** — a genuine content-validity gap (M3). We can hint but not measure abstention reliably without code/task context. (inference)
- **Durability windows are judgment calls.** "Churned within 2 weeks" [11][12] and survival windows are heuristics; legitimate iteration can look like low durability.
- **Cold-start engineers** (new hires, light users) produce sparse, unreliable signals — suppressed to Exploratory or withheld (X6).
- **Vendor telemetry is uneven** — survival native only in Sourcegraph [25]; Claude Code telemetry off by default [23]; Copilot drops small teams [19]; some fields unverified [26][27].
- **The amplifier confound** [2]: a healthy engineer in a weak environment will show poor downstream signals not of their making — team context is a *lens*, never blended into the individual signal.

### The realistic MVP (don't build the whole catalog)
The 20+ metric catalog is comprehensive by design; it is **not** a build list. The minimum viable, defensible version is ~4 signals: **(1) code survival/churn** (overall, git-reconstructed), **(2) feature/mode mix**, **(3) review-friction trend**, **(4) one short anonymized survey**. Start there, validate, then expand only if validation justifies it.

## 5. Red-team record (what the adversarial passes changed)
- **RT#1 (cycle 3):** killed the code-access metric; killed peer-ranking and per-individual DORA; demoted all volume metrics behind a volume gate; rewrote recommendation tone from grading to coaching; added perception-gap handling.
- **RT#2 (cycle 4):** no material weakness; refinements — cold-start gating, multi-tool normalization wording, explicit "what this is NOT" boxes.
- **RT#3 (cycle 5):** no material weakness. Two consecutive clean passes → durability criterion met.
- **RT#4 (skeptic pass, post-delivery):** a fresh adversarial review (jg-skeptic) surfaced four real *honesty* gaps the prior passes under-weighted: (a) the framework is **empirically unvalidated** (theoretical convergent validity only); (b) the **AI-attribution gap** means durability is usually *overall*, not AI-specific [18]; (c) **durability can reward timidity**; (d) **recommendation efficacy is unproven**. None is a fundamental flaw, but all materially affect honest interpretation. Fixed: added precondition P4 (01), strengthened failure modes on 3.1/3.2 (02), added the unvalidated-model + attribution + timidity + efficacy + simpler-alternative limitations and the MVP note (06), updated caveats in 00. This pass found real problems → it counts as a non-clean red-team; saturation re-evaluated below.

## 6. The one-sentence ethic
Treat every metric as an **indicator to be validated, never a target to be hit**; deliver it to the engineer as a mirror for self-improvement, never to anyone else as a measure of their worth — because the instant it becomes a target or a manager's tool, Goodhart and Campbell guarantee it stops measuring quality and starts causing harm.
