# 07 — Worked Examples

Two fictional engineers run end-to-end through the framework, producing tiered recommendations with legible reasoning. All data is invented for illustration; the *logic* is exactly that of deliverables 02–03. Both are read as **trends vs the engineer's own baseline**, never vs each other.

---

## Engineer A — "Priya" — the high-volume, low-durability profile

### Observed signals (4-week window)
| Signal (deliverable 02 id) | Observation | Method type |
|---|---|---|
| Token usage (2.6) | Very high; top decile of her own history | telemetry/volume |
| Feature mix (2.1) | ~90% autocomplete + large agent generations; almost no chat/plan | telemetry |
| Acceptance rate (2.1) | High (~45%), rising | telemetry |
| **Code survival / retention (3.1)** | **Low and falling** — ~55% of AI-attributed lines gone within 10 days (her baseline ~80%) | diff-metadata |
| **Churn (3.2)** | **Rising** — 12% of new lines revised within 2 weeks (baseline ~6%) | diff-metadata |
| Review friction (3.6) | Changes-requested ratio up vs baseline; +1.5 review round-trips/PR | review metadata |
| First-pass CI (3.4) | Down ~10 pts vs baseline; flaky-rate controlled out | CI metadata |
| Self-report (1.3/1.4) | "AI saves me tons of time, I feel way faster" | survey |
| Cold-start? | No — rich history | — |
| Confound check | Task mix unchanged (ticket data: same bug:feature ratio) | PM metadata |

### Framework reading
- **Pillar 2 (depth):** moderate — uses AI heavily but shallowly (autocomplete-dominant).
- **Pillar 3 (durability):** **weak and worsening** — low survival + rising churn + rising review friction + falling CI, four signals agreeing across diff-metadata, review, and CI method types.
- **Pillar 1 (leverage):** self-reported high, objectively contradicted → **perception gap**.
- **Amplifier lens:** team DORA healthy, so this is about her usage, not a broken environment.

### Recommendations (with confidence + why + caveat)
1. **Skill — verification-first + smaller diffs.** @ **High**
 *Triggering signals:* low survival (3.1) + rising churn (3.2) + heavy acceptance volume (2.1) — independent method types agreeing on GitClear's documented "high generation / low retention" anti-pattern [11][12].
 *Why:* the volume feels productive but is generating rework downstream; reviewing and testing AI output before accepting, and prompting for smaller changes, directly targets the churn.
 *Caveat:* some churn is legitimate iteration; this is a trend signal, not a verdict.
2. **Workflow — adopt "generate → test → refine" instead of "generate → accept."** @ **Medium**
 *Why:* restructures the habit driving the churn; pairs with #1.
3. **Calibration insight — your felt speed and measured outcomes diverge.** @ **Medium**
 *Triggering signals:* high self-report time-saved (1.3) vs flat/worse objective leverage.
 *Why:* the METR perception gap [17] in miniature — offered as honest self-calibration, *not* "you're wrong." A chance to notice where AI helps vs feels helpful.
 *Caveat:* perception still matters for experience; this isn't a reprimand.
4. **Skill (Exploratory) — try chat/plan mode for complex changes.** @ **Exploratory**
 *Why:* her autocomplete-dominant mix may be mismatched to the gnarlier tasks where churn concentrates; single-method hint only.

**What Priya is NOT told:** any score, any grade, any comparison to teammates, anything routed to her manager.

---

## Engineer B — "Marco" — the deep, durable, healthy profile (reinforce)

### Observed signals (4-week window)
| Signal | Observation | Method type |
|---|---|---|
| Token usage (2.6) | Moderate | telemetry/volume |
| Feature mix (2.1) | Rich — plan + agent for refactors, chat for exploration, autocomplete for boilerplate | telemetry |
| Tool fit (2.2) | Uses agentic tool for multi-file work, autocomplete tool for boilerplate | telemetry |
| **Code survival (3.1)** | **High** — ~85%, stable | diff-metadata |
| Churn (3.2) | Low (~5%), stable | diff-metadata |
| Review friction (3.6) | Low changes-requested ratio; stable | review metadata |
| First-pass CI (3.4) | High, stable | CI metadata |
| Cycle-time shape (1.1) | Compressed coding time, no review-time inflation | PR metadata |
| Self-report (1.4) | Positive flow, low frustration | survey |
| Engaged rate (2.3) | High | telemetry |

### Framework reading
All three pillars agree and are healthy, across all three method types. This is deliverable 03 **Pattern C1** (durable + low-friction + deep).

### Recommendations
1. **Recognition + share.** @ **High**
 *Triggering signals:* high survival (3.1) + low churn (3.2) + rich feature mix (2.1) + healthy cycle-time shape (1.1) — full tri-pillar agreement across method types.
 *Why:* his AI usage lands cleanly and is deep; enablement includes affirming what works and inviting him to share his workflow with peers (opt-in).
 *Caveat:* recognition is offered to him, not published as a ranking.
2. **Agent — candidate for delegating well-specified, testable tasks to coding agents.** @ **Medium**
 *Why:* his durability track record suggests he can safely push more into agentic automation; single-method inference so capped at Medium.
3. **Workflow (Exploratory) — template his plan→implement→review loop as a reusable team practice.** @ **Exploratory**
 *Why:* pattern-match on his healthy mode mix; offered as an idea, not a directive.

---

## What the two examples demonstrate
- **Volume never wins.** Priya's high tokens/acceptance do not produce a positive reading — the volume gate holds; her durability signals drive the recommendations.
- **Durability is the spine.** Both engineers' headline findings come from survival/churn (Pillar 3), the most defensible quality proxies.
- **Confidence is earned by agreement.** High-tier recommendations appear only where independent method types converge; single-method signals cap at Medium/Exploratory.
- **Trend, not rank.** Every reading is vs the engineer's own baseline; the two are never compared.
- **Enablement, not judgment.** Even the weak profile (Priya) is delivered as coaching with concrete next actions and explicit caveats — never a grade, never to a manager.
- **Honesty about perception.** The perception gap is surfaced as calibration, embodying the METR caution [17].
