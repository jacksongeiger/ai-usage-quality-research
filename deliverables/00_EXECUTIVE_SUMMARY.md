# 00 — Executive Summary

**The question:** How can we measure the *quality* of an individual software engineer's AI tool usage — **without ever seeing their code** — and turn that into tiered, actionable, per-engineer recommendations that help them use AI more effectively, delivered to the engineer as enablement?

**The answer, in one paragraph.** Define AI usage quality not as code quality, productivity, or activity, but as **durable, low-friction, well-matched leverage from AI**. Measure it as a *latent construct* through a small nomological network of three proxy pillars — **Durability/Landing-quality**, **Adoption Depth**, and **Leverage/Efficiency** — each built entirely from metadata, telemetry, and self-report. Trust a finding only where **independent signals from different method types agree**, express every output in **confidence tiers** (High / Medium / Exploratory), compare each engineer **only to their own trend baseline** (never to peers), read everything through DORA's **"AI is an amplifier" lens** against team context, and deliver it **to the engineer alone** as coaching. Volume (tokens, raw acceptance, lines) is structurally demoted and can never, by itself, signal quality.

---

## The measurement model (deliverables 01–02)

**Three pillars, all no-code:**

1. **Durability / Landing-quality — the strongest pillar.** Does AI-assisted output survive contact with reality?
 - *Code/suggestion survival & retention* — the single most defensible quality signal (Sourcegraph exposes it natively at 30–600s [25]; reconstructable elsewhere from git line-survival [41]).
 - *Churn / rework* — lines revised within ~2 weeks; GitClear proves this is computable from diff metadata alone [11][12].
 - *First-pass CI trend, review-friction trend, revert/hotfix rate* — all from status/timestamp metadata.
 This pillar is *uniquely well-served* by the no-code constraint: durability is a metadata property.

2. **Adoption Depth — shallow autocomplete vs deep agentic.** Feature/mode mix (agent/plan/chat vs tab-only) [19][22][23], engaged-not-just-active [21], tool fit across the engineer's portfolio (~82% use 3+ tools [7]), model/effort calibration.

3. **Leverage / Efficiency — is AI reducing friction?** Cycle-time shape decomposed from PR/commit timestamps [38], plus self-report of flow/time-saved — used for *experience*, never as ground-truth speed.

**Why proxies are valid without code:** measurement theory [32] says a latent construct is validly measured by a network of indicators showing convergent validity (signals that should co-move, do) and discriminant validity (quality separated from quantity — enforced by demoting volume). We never claim to see "quality"; we infer it where proxies converge.

## From signals to tiered recommendations (deliverable 03)

Recommendations span four types — **Tools, Skills, Agents, Workflows** — gated by **signal agreement**:
- **High** = ≥2 independent signals from different method types agree, sustained, uncontradicted.
- **Medium** = one strong (durability/depth) signal, or two weak agreeing ones.
- **Exploratory** = a single pattern-match or self-report-only hint.

Four hard gates run first: **volume gate** (nothing triggers on tokens/acceptance/LOC alone), **confound gate** (drop a tier and name the confound), **cold-start gate** (suppress sparse-data engineers), **contradiction gate** (surface tensions, don't paper over them). Every recommendation ships with its tier, a plain-English "why" naming the triggering signals, the main caveat, and trend-vs-baseline framing.

*Example:* low survival + rising churn + heavy acceptance → **Skill: verification-first + smaller diffs @ High** — because two method types agree on GitClear's documented "high generation / low retention" anti-pattern, the highest-leverage, lowest-regret fix. (Full worked personas in deliverable 07.)

## Top additional data to pursue (deliverable 04)

Ranked by (signal value × realism ÷ privacy cost), all no-code:
1. **Free, clean, high-integrity first:** deploy/CD + revert data, CI test metadata, quality-gate status, security-scan counts — artifact-level, near-zero privacy cost, strengthen the durability pillar immediately.
2. **Highest value:** turn on full **IDE/assistant telemetry** — especially **survival/retention** and feature-mix. Much requires *turning on* telemetry not yet in Snowflake (Claude Code OTel is off by default [23]; Copilot drops <5-seat teams [19]).
3. **The multiplier:** a lightweight, anonymized **developer survey** program — the most overlooked, highest-leverage addition; SPACE and DX both make the perceptual dimension mandatory [3][7] and it validates every objective signal.

## The headline caveats (deliverables 05–06)

- **The evidence on AI's impact is genuinely contested.** Vendor RCT says +55% [8]; independent RCT (METR) says experienced devs were **19% slower while feeling 20% faster** [17]. DORA reconciles it: **AI amplifies what's already there** [2] — durable leverage where foundations are strong, churn where they're weak. Our framework exists to detect which way AI is amplifying for each engineer.
- **Never let a metric become a target or a manager's tool.** Goodhart/Campbell [30][31] guarantee that the instant it does, it stops measuring quality and starts causing harm. Hence: engineer-only, opt-in, deletable, trend-not-rank, coaching-not-grading, GDPR-minimal [34].
- **Self-report measures experience, not truth.** The perception gap [17] is used *constructively* (calibration insight), never as a gotcha.
- **This framework is unvalidated** — it is a literature-grounded *design*, not a tested instrument; the convergent-validity argument is theoretical and needs an empirical validation step (correlate proxies against an independent, no-code criterion) before it's trusted in production.
- **The AI-attribution gap:** outside Sourcegraph, durability signals usually measure *overall* code survival/churn, not AI-specific — vendor tools can't isolate AI-authored lines [18] (precondition P4). Real but coarser.
- **Other honest limits:** durability can reward *timid/trivial* work (pair with PR-size/ticket context); recommendation efficacy is unproven; a good survey alone may capture most of the value; "knowing when NOT to use AI" is only weakly proxied; cold-start engineers yield unreliable signals. **Don't build the 20+ metric catalog — the MVP is ~4 signals** (survival/churn, feature-mix, review-friction trend, one survey); validate, then expand.

## What this is NOT
Not a code-quality measure · not a productivity or performance score · not a peer ranking · not a manager/HR artifact · not a single number · not a real-time judgment · not a license to recommend "use AI *more*" (DORA shows more AI can *hurt* stability [1][2] — the goal is *better*, not *more*).

## Bottom line
A defensible, ethical answer exists. Build it on **diff-metadata durability signals** (the quality core, perfectly suited to no-code), **vendor depth telemetry**, and **survey-based experience**, triangulated and confidence-tiered, interpreted through the **amplifier lens**, and delivered to the engineer alone as enablement. Measure to help the person in front of the tool get durable leverage from it — never to score them.
