# 03 — Signal-to-Recommendation Logic

This is the "turn measurement into actionable insight" half of the question. It specifies how measured signals (deliverable 02) become **tiered, per-engineer recommendations** across four recommendation *types*: **Tools, Skills, Agents, Workflows** — delivered to the engineer as enablement.

## 1. How confidence tiers are computed (signal-agreement gating)

Recommendation confidence is **not** the same as a single metric's tier. It is computed by **agreement across independent signals** over a ≥4-week window, because a latent construct measured by noisy proxies is only trustworthy where proxies converge (deliverable 01 §2; [32]).

| Recommendation tier | Trigger rule |
|---|---|
| **High Confidence** | ≥2 **independent** signals from **different method types** (telemetry / diff-metadata outcome / self-report) agree on the same pattern, sustained over the window, with no strong contradicting signal. |
| **Medium Confidence** | One **strong** signal (a Pillar-3 durability metric or feature-mix), OR two **weak** signals that agree but share a method type / are confounded. |
| **Exploratory** | A single pattern-match, a volume-adjacent hint, or a self-report-only signal. Framed as "you might explore," never as a finding. |

**Hard gates (applied before any tier is assigned):**
1. **Volume gate** — no recommendation may be triggered by a VOLUME metric (tokens, raw acceptance, LOC, consistency) *alone*. Volume can only *support* a finding already anchored by durability/depth/perceptual signals. (Discriminant validity; Goodhart [30][3].)
2. **Confound gate** — if the pattern is plausibly explained by a known confound (task-type shift, tenure, repo change, PTO), drop one tier and state the confound.
3. **Cold-start gate** — if the engineer has < the minimum events for a signal (new hire, light user), suppress to Exploratory or withhold (X6). Never infer from sparse data.
4. **Contradiction gate** — if a strong signal contradicts the pattern (e.g., depth is low but survival is excellent), do not recommend "use AI more"; surface the tension instead.

## 2. The signal→recommendation map

Notation: **Pattern** (observed signal combination) → **Recommendation** (type: Tool/Skill/Agent/Workflow) @ **Tier** — *because* (legible reasoning grounded in framework/sources). Signal IDs reference deliverable 02.

### Pattern group A — Shallow usage (depth gap)

**A1. Autocomplete-only profile.** Feature-mix (2.1) shows heavy tab/inline, ~zero chat/agent; survival (3.1) fine; cycle-time (1.1) unremarkable.
→ **Skill**: "Try chat-driven exploration for unfamiliar code; try agent/plan mode for multi-file refactors." @ **Medium** — *because* depth is a direct behavioral signal [19][22] and the engineer is leaving leverage on the table without any durability cost. Framed as opportunity, not deficiency.
→ **Workflow**: "Adopt a plan→implement→review loop with the assistant for larger tasks." @ **Exploratory** — *because* workflow fit depends on task context we can't fully see.

**A2. Bursty/abandoned adoption.** Consistency/recency (2.4) low; engaged rate (2.3) low; token usage (2.6) sporadic.
→ **Tool/Skill**: "Lightweight on-ramp — pick one daily task to pair with AI for two weeks." @ **Exploratory** — *because* this is volume/consistency-only (volume gate forbids higher) and consistency≠quality; offered as a gentle nudge.

**A3. Single-tool when portfolio would help.** Tool-fit (2.2): uses one tool for everything; team peers get leverage from a complementary tool for a task type this engineer struggles with (high churn 3.2 on, say, refactors).
→ **Tool**: "For refactors, try an agentic tool (your autocomplete tool is great for boilerplate, less so here)." @ **Medium** — *because* this is anchored by a durability signal (refactor churn), not tool-count vanity; the volume gate is satisfied.

### Pattern group B — Durability / landing-quality gap (the most important)

**B1. High generation, low retention (the canonical AI anti-pattern [11][12]).** Survival (3.1) low / churn (3.2) high, AND feature-mix (2.1) shows heavy acceptance of large agent/autocomplete output.
→ **Skill**: "Verification-first habit — review and test AI output before accepting; smaller prompts, smaller diffs." @ **High** — *because* two independent signals agree across method types (diff-metadata durability + telemetry acceptance behavior), this is GitClear's documented churn pattern [12], and it's the highest-leverage fix. Tone: coaching, with the *why* (churn costs you rework later).
→ **Workflow**: "Adopt a 'generate → test → refine' loop rather than 'generate → accept'." @ **Medium**.

**B2. Rising review friction on AI-assisted work.** Review-friction trend (3.6) rising (changes-requested ratio, round-trips up) vs personal baseline; churn (3.2) up; **confound check**: task difficulty not obviously higher (ticket data 4.5 if available).
→ **Skill**: "AI output may be clearing review less cleanly than before — try asking the assistant to self-review against your team's conventions before opening the PR." @ **Medium** — *because* review friction is confounded (reviewer strictness, task type) so even with two signals we cap at Medium per the confound gate; trend-vs-baseline avoids peer-ranking harm.
→ **Workflow**: "Pre-PR checklist / assistant-driven self-review step." @ **Medium**.

**B3. Falling first-pass CI after AI adoption.** First-pass CI (3.4) trend down vs baseline; flaky-rate (4.4) controlled out; churn (3.2) up.
→ **Skill**: "Run/generate tests locally with the assistant before pushing." @ **Medium** — *because* CI is heavily confounded [44] (so not High even with agreement), but two signals + flaky-control make it actionable as a personal trend.

**B4. Reverts/hotfixes clustering on AI-assisted changes.** Revert/hotfix (3.3) elevated; survival (3.1) low.
→ **Skill/Workflow**: "Add a verification gate for AI-generated changes; prefer incremental landing." @ **Medium-High** if deploy data confirms (4.1/4.6), else **Exploratory** — *because* event sparsity and attribution risk; never frame as blame.

### Pattern group C — Healthy usage (reinforce, don't fix)

**C1. Durable + low-friction + deep.** Survival (3.1) high, churn (3.2) low, feature-mix (2.1) rich, cycle-time shape (1.1) healthy, self-report (1.4) positive.
→ **Recognition + stretch**: "Your AI usage is landing cleanly and you're using it deeply — consider sharing your workflow with the team / trying agentic automation for repetitive multi-file tasks." @ **High** — *because* all three pillars agree across method types. Enablement includes *affirming* good practice, not only flagging gaps.
→ **Agent**: "You're a candidate for delegating well-specified, testable tasks to coding agents." @ **Medium**.

### Pattern group D — Calibration / efficiency

**D1. Perception–reality gap.** Self-report time-saved (1.3) high BUT durability/cycle-time objective signals flat or worse.
→ **Skill (calibration)**: "Here's where your felt speed and measured outcomes diverge — a chance to sanity-check where AI is actually helping vs feeling helpful." @ **Exploratory-Medium** — *because* this is METR's perception gap [17] used constructively; framed as honest self-calibration, never "gotcha." This is one of the most valuable insights self-report enables once we stop trusting it as truth.

**D2. Model/effort mis-calibration.** Calibration (2.5): max-effort model on trivial tasks (high cost-per-outcome) or never escalating on hard ones (high churn 3.2 on complex work).
→ **Skill**: "Match model/effort to task — cheaper/faster for boilerplate, stronger models for gnarly problems." @ **Exploratory** — *because* "right model" is task-dependent and unknowable without code; pure hint.

### Pattern group E — Environment-aware (amplifier lens [2])

**E1. Good individual usage, weak environment.** Individual signals healthy but team DORA (4.1) shows low deploy frequency / high change-fail; first-pass CI low team-wide.
→ **Workflow**: "AI is amplifying onto shaky foundations — the highest-leverage move may be team practices (testing, smaller batches), not your AI habits." @ **Medium** — *because* DORA 2025 [2]: AI amplifies what's there. Honest framing that the bottleneck isn't the engineer.

## 3. Worked rule examples (fully legible)

> **Example rule 1 (High):**
> IF `survival_rate(30–600s or N-day)` < personal_baseline − threshold (signal 3.1, diff-metadata)
> AND `agent/autocomplete acceptance volume` high (signal 2.1, telemetry)
> AND NOT (cold-start) AND NOT (task-type shift confound)
> THEN recommend **Skill: verification-first + smaller diffs** @ **High**
> BECAUSE two independent method types agree on the GitClear churn anti-pattern [11][12]; it's the highest-impact, lowest-regret fix; volume gate satisfied (anchored by durability, not by tokens).

> **Example rule 2 (Medium):**
> IF `feature_mix` = autocomplete-only (signal 2.1) AND `survival/churn` healthy (3.1/3.2)
> THEN recommend **Skill: try chat/agent modes** @ **Medium**
> BECAUSE depth is directly observed [19] and there's no durability cost — pure upside opportunity; capped at Medium because depth is a means, not a confirmed quality gain (single method type).

> **Example rule 3 (Exploratory):**
> IF `self_report_time_saved` high (1.3) AND objective leverage flat (1.1)
> THEN surface **calibration insight** @ **Exploratory**
> BECAUSE the perception gap [17] means felt speed isn't truth; offered as reflection, never as correction.

## 4. Output contract for each recommendation (so it's actionable + honest)
Every recommendation delivered to the engineer carries:
1. **The recommendation** (specific Tool/Skill/Agent/Workflow action).
2. **Confidence tier** (High/Medium/Exploratory).
3. **The "why"** — plain-English, naming the triggering signals and the reasoning.
4. **The caveat** — the main confound / what could make this wrong.
5. **Trend framing** — relative to the engineer's own baseline, never peers.
6. **Opt-out** — the engineer can dismiss/disable any signal (deliverable 06).

## 5. What this logic deliberately will NOT do
- Will not produce a single "AI quality score" or grade (Goodhart [30]; SPACE [3]).
- Will not rank engineers or expose data to managers (constraint #2; [4]).
- Will not recommend "use AI more" as a blanket goal — DORA shows more AI can *hurt* stability [1][2]; recommendations are about *better*, not *more*.
- Will not trigger anything from volume alone (volume gate).
- Will not assert causation ("AI made you slower"); only association, with confounds named [37].
