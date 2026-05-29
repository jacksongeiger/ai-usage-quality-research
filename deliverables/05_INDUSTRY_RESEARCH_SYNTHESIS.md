# 05 — Industry & Academic Research Synthesis

How the top of the industry and academia actually measure developer productivity and AI's impact, what they've found, where they disagree, and what transfers to our constraints (no-code, individual enablement, metadata/self-report). Every claim traces to SOURCES.md; inferences are labeled **(inference)**.

## 1. The framework lineage (and what survives our constraints)

There is a clear intellectual lineage, largely from the same researchers (Forsgren, Storey, Noda, et al.):

**DORA → SPACE → DevEx → DX Core 4.**

- **DORA** [1][2] measures *team delivery*: deployment frequency, lead time, change failure rate, time to restore. It is the most empirically grounded engineering-measurement program. **Transfers as**: team-level *context lens* only — DORA is explicitly system-level and attributing it to an individual breaks construct validity (X13). Its AI findings are the spine of our framework (§3).
- **SPACE** [3] reacted to single-metric misuse: productivity is multi-dimensional (Satisfaction, Performance, Activity, Communication, Efficiency) and must be measured with metrics *in tension*, never one number, always including a perceptual dimension; activity is "the most misused dimension." **Transfers as**: the anti-single-metric discipline and the mandatory self-report pillar — the backbone of our triangulation.
- **DevEx** [4] reframed productivity as *experience*: Feedback Loops, Cognitive Load, Flow State, measured primarily by developer perception plus system data. **Transfers as**: the enablement framing itself (remove friction *for* the developer) and Pillar 1.
- **DX Core 4** [6] unified the three into Speed / Effectiveness (DXI) / Quality / Impact, deliberately counterbalancing speed against the DXI to prevent gaming, and stating that individual-level signals (diffs/FTE) are usable *only* with no targets/rewards and careful communication. **Transfers as**: the counterbalancing principle and the individual-use guardrails (our deliverable 06).
- **DX AI Measurement Framework** [7] is the most explicit AI-specific taxonomy: Utilization (DAU/WAU, % AI-assisted PRs, % AI-generated code, adoption cohorts), Impact (time-saved survey, CSAT, Core-4 regression), Cost. It flags code-generation *volume* as "particularly susceptible to gaming." **Transfers as**: most of Pillar 2 and the explicit volume-skepticism.

**The McKinsey counter-example** [14] and the **Orosz/Beck rebuttal** [15] define the boundary: McKinsey's output-leaning metrics (inner/outer-loop time, contribution analysis) are, per Orosz/Beck, effort/output proxies that ignore outcomes, are easily gamed, and dangerously leak into reviews/layoffs. **Transfers as**: the explicit anti-pattern our framework refuses (no output-scoring, no manager-facing use).

## 2. What the empirical record actually says about AI's impact

This is where the research is genuinely contested, and honesty requires holding the contradiction.

**The optimistic, mostly-vendor evidence:**
- GitHub's RCT [8]: Copilot users **55% faster** on a greenfield JS task (n=95) — but the 95% CI is **[21%, 89%]**, very wide, and the task is not representative of mature-codebase work. Surveys: 73% more flow, 87% less mental effort.
- Ziegler et al. [9]: **acceptance rate is the best telemetry predictor of *perceived* productivity** — a crucial nuance: it predicts how productive devs *feel*, not how productive they *are*.
- Accenture/GitHub [29]: +8.69% PRs/dev, +84% build success, 88% retention — **(vendor-reported, favorable bias).**

**The skeptical, independent evidence:**
- **METR [17]** — the most important counterweight: a randomized trial of 16 experienced devs on 246 real tasks in their *own large mature repos* found AI made them **19% slower**, while they believed they were **20% faster** (and forecast +24%). This is the empirical anchor for distrusting self-reported speed.
- **GitClear [11][12]** — from diff metadata alone: churn (lines revised within 2 weeks) rose **5.5%→7.9%**, refactoring fell **~25%→<10%** of changed lines, copy/paste rose **8.3%→12.3%**, clones up ~4×. AI inflates *volume* while *durability* degrades.
- **DORA 2024 [1]**: +25% AI adoption ≈ **−1.5% throughput, −7.2% stability** — AI was, at that point, net-negative on delivery stability.

**The reconciliation — DORA 2025's "amplifier" thesis [2]:** by 2025 AI's relationship to throughput turned positive, but it **remained negative for delivery stability**. The unifying explanation: *"AI is an amplifier — it doesn't fix a team; it amplifies what's already there."* Strong foundations (testing, small batches, fast feedback, good internal platform) convert AI into gains; weak foundations convert it into instability. This single finding resolves the vendor-vs-independent contradiction: GitHub's greenfield task and METR's mature-repo tasks are different "amplification environments," and both results can be true.

**What this means for us (inference):** measuring AI usage quality is precisely about detecting *which way AI is amplifying for this engineer* — durable leverage vs churn — which is why our strongest pillar is Durability, and why we read every individual signal against team context.

## 3. Vendor telemetry: what's actually exposed (and what to trust)

A practical map of usage-quality signals across the major tools [19–29]:

| Signal | Best-exposed by | Trust | Note |
|---|---|---|---|
| **Suggestion survival/retention** | **Sourcegraph** (30/120/300/600s persistence) [25] | **Highest** | The cleanest quality proxy; only Sourcegraph native — reconstruct elsewhere from git |
| Feature/mode mix (agent/chat/autocomplete) | Copilot [19], Cursor [22], Claude Code [23] | High | Direct depth signal, hard to fake |
| Engaged vs active user | GitHub [21] | High | "Engaged" (intentional action) is GitHub's own quality cut |
| Acceptance rate (CAR/wCAR) | All majors [19][22][24][25] | **Low alone** | Vendor-optimized, gameable [28]; use only paired with survival |
| Model/effort + cost-per-outcome | Claude Code [23], Cursor [22] | Medium | Calibration hint |
| Lines suggested/accepted, tokens | All | **Lowest** | Pure volume; agentic tools burn 10–100× tokens [7] |

Two operational truths: (1) **most of this requires turning on telemetry** not yet in Snowflake — Claude Code OTel is off by default [23], Copilot drops <5-seat teams [19], Cursor caps at 90 days [22]. (2) **~82% of devs use 3+ tools** [7], so any single-tool view undercounts an engineer — scores must normalize across the portfolio.

## 4. The validity & ethics literature (why naïve measurement fails)

- **Goodhart [30] / Campbell [31]**: any metric used as a target degrades and corrupts the process. Engineering-specific gaming is well documented [13] (LOC padding, micro-commits, force-push to fake cycle time, assertion-free coverage). **Transfers as**: never expose a single score as a target; signal-agreement gating; audience-lock.
- **Construct validity [32][33]**: a latent construct ("usage quality") is validly measured only via a nomological network with convergent + discriminant validity — exactly our multi-pillar triangulation. The AI-evaluation validity paper [33] warns benchmarks "conflate genuine reasoning with domain knowledge" — our analog risk is conflating *quality* with *volume*, which we defend against structurally.
- **Confounds [37]**: individual variance dominates; tenure/role/repo/task are heavy confounders — invalidating cross-person comparison and mandating trend-vs-baseline.
- **Surveillance ethics [34][35][36]**: GDPR/Article-29 necessity-proportionality-minimization; the observer/Hawthorne effect (measuring changes behavior); psychological safety as a precondition for honest self-report. **Transfers as**: opt-in, transparent, deletable, engineer-only, aggregated, minimal.

## 5. Where serious sources disagree (held honestly)
1. **Does AI help?** Vendor RCT (+55% [8]) vs independent RCT (−19% [17]). Reconciled by the amplifier thesis [2] + task/environment differences.
2. **Is self-report trustworthy?** DevEx/DX lean on it [4][7]; METR shows it's wrong about *speed* [17]. Resolution: self-report is valid for *experience*, invalid as a *speed truth*.
3. **Is acceptance rate meaningful?** Vendors surface it; Ziegler shows it predicts *perception* [9]; LeadDev calls it a vanity metric [28]. Resolution: usable only paired with survival.
4. **Can you measure individuals at all?** McKinsey yes [14]; SPACE/DevEx/Orosz-Beck strongly caution [3][4][15]. Resolution: only for self-improvement, engineer-only, never evaluation.

## 6. The transfer summary (what the literature licenses us to build)
The defensible recipe, fully grounded: **DevEx framing (enablement, survey-led) + DX AI utilization telemetry (depth) + GitClear-style diff-metadata durability signals (the quality core) + DORA team context as the amplifier lens**, disciplined by **SPACE's no-single-metric rule**, **construct-validity triangulation**, **METR's perception-gap caution**, and the **Goodhart/Orosz-Beck rule** that the instant a metric becomes a target or a manager's tool, it stops measuring quality and starts harming people. McKinsey is the documented anti-pattern to avoid.
