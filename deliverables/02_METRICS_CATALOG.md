# 02 — Metrics Catalog

Every candidate metric for inferring AI usage quality. Each entry carries the five required fields:
**Data dependency** · **What it proxies** · **Confidence tier** · **Known failure modes / confounds** · **Gaming risk**.

**Data-source legend**
- `[HAVE]` — derivable from data the brief says is already in Snowflake (token usage, tool adoption, PR/commit/LOC metadata, CI build/review status, PR-feedback categorical records).
- `[TURN-ON]` — exists in a tool the org likely uses but must be ingested (vendor telemetry beyond tokens, CD/deploy, security/gate status, tickets, surveys). Ranked in deliverable 04.
- `[KILLED]` — proposed then removed for violating the no-code constraint.

**Confidence-tier meaning** (a property of how *trustworthy the signal is as a quality proxy*, before combination): **High** = strong, hard-to-game, well-validated proxy; **Medium** = decent proxy but confounded or partially gameable; **Exploratory** = weak/pattern-match only, use as a hint. (Recommendation confidence in deliverable 03 is a separate, agreement-based computation that *combines* these.)

> **Structural rule (discriminant validity):** every metric tagged **VOLUME** below can never, on its own, produce a positive quality finding. Volume metrics are context only, always paired with a durability or perceptual signal. This is enforced in deliverable 03.

---

## Pillar 1 — Leverage / Efficiency

### 1.1 Cycle-time shape (decomposed) `[HAVE]`
- **Data dependency**: PR + commit timestamps; review-action timestamps. Segments per LinearB/Code Climate [38][39]: Coding (first commit→PR open), Pickup (PR open→first non-author action), Review (first action→merge), Deploy (merge→release; Deploy needs `[TURN-ON]` CD data).
- **Proxies**: friction reduction — quality looks like compressed *coding* time without inflated *review/rework* time.
- **Confidence**: **Medium**.
- **Failure modes/confounds**: task size & type, repo, reviewer availability, WIP load, time zones. Coding-time compression can reflect smaller tasks, not AI leverage.
- **Gaming risk**: Medium — force-push to fake first-commit time [13]; split work into trivial PRs. Mitigation: pair with PR-size normalization; use personal trend.

### 1.2 Coding-time trend vs personal baseline `[HAVE]`
- **Data dependency**: as 1.1, longitudinal per engineer.
- **Proxies**: whether AI adoption is associated with the engineer's *own* authoring friction dropping over time.
- **Confidence**: **Medium**.
- **Failure modes/confounds**: confounded by changing task mix; correlation≠causation [37] — never claim AI *caused* the change.
- **Gaming risk**: Low-Medium.

### 1.3 Self-reported time-saved per week `[TURN-ON: survey]`
- **Data dependency**: periodic developer survey (DX-style) [7][4].
- **Proxies**: *perceived* leverage / experience.
- **Confidence**: **Exploratory** (as a truth signal) / **Medium** (as an experience signal).
- **Failure modes/confounds**: **the perception gap** — METR [17] found felt +20% while actually −19%. Must NOT be read as actual speed.
- **Gaming risk**: Low (self-directed), but socially-desirable-response bias.

### 1.4 Self-reported flow / cognitive load `[TURN-ON: survey]`
- **Data dependency**: survey (DevEx items) [4].
- **Proxies**: friction & focus quality of AI-assisted work.
- **Confidence**: **Medium** (perception is the legitimate domain of self-report for experience).
- **Failure modes/confounds**: mood, recency bias.
- **Gaming risk**: Low.

### 1.5 Context-switch / interruption rate `[TURN-ON: collaboration metadata]`
- **Data dependency**: PR-comment response latency counts, calendar/focus-time (privacy-heavy — deliverable 04 rank #9).
- **Proxies**: cognitive load / flow [4].
- **Confidence**: **Exploratory**.
- **Failure modes/confounds**: highly role-dependent; privacy cost high.
- **Gaming risk**: Low.

---

## Pillar 2 — Adoption Depth

### 2.1 Feature/mode mix (agent/plan/chat vs autocomplete) `[TURN-ON: vendor telemetry]`
- **Data dependency**: Copilot `requests per chat mode`, `used_agent/used_chat` [19]; Cursor `intent` Write/Ask/Plan, `agentRequests` [22]; Claude Code `skill_activated`, mode events [23].
- **Proxies**: depth of usage — using AI as a reasoning partner vs shallow tab-completion.
- **Confidence**: **Medium-High** (direct behavioral signal, hard to fake meaningfully).
- **Failure modes/confounds**: task type dictates appropriate mode; not all work warrants agentic use; tool availability differs.
- **Gaming risk**: Low-Medium — could invoke agent mode performatively; mitigated by pairing with durability (Pillar 3).

### 2.2 Tool-fit / portfolio breadth (normalized) `[HAVE: tool adoption] + [TURN-ON: per-tool telemetry]`
- **Data dependency**: which tools used (HAVE) + per-tool feature use (TURN-ON). Normalized across the engineer's actual set (A9; ~82% use 3+ [7]).
- **Proxies**: well-matched usage — right tool per task.
- **Confidence**: **Exploratory-Medium** (matching tool→task without code is inferential).
- **Failure modes/confounds**: more tools ≠ better; org tool availability; license constraints.
- **Gaming risk**: Medium — adopting tools for show. **VOLUME-adjacent**: breadth alone never a positive finding.

### 2.3 Engaged-not-just-active rate `[TURN-ON: vendor telemetry]`
- **Data dependency**: GitHub "engaged vs active" [21]; analogous DAU/engaged splits other vendors.
- **Proxies**: substantive vs nominal adoption (seat-waste detection).
- **Confidence**: **Medium**.
- **Failure modes/confounds**: vendor-specific "engaged" definitions differ; <5-seat teams excluded by Copilot [19].
- **Gaming risk**: Low-Medium.

### 2.4 Usage consistency & recency `[HAVE: token usage timestamps]`
- **Data dependency**: active-days / available-days; days since last use; per-tool token cadence (HAVE).
- **Proxies**: durable habit formation vs bursty/abandoned adoption.
- **Confidence**: **Medium** (for consistency) / **Exploratory** (consistency≠quality).
- **Failure modes/confounds**: PTO, role, project phase; consistency is a habit signal, not a quality signal on its own.
- **Gaming risk**: Medium — trivially generate daily tokens. **VOLUME**: never alone.

### 2.5 Model/effort calibration `[TURN-ON: vendor telemetry]`
- **Data dependency**: model usage by task [19][22]; Claude Code `cost.usage` by effort/model [23].
- **Proxies**: right-sizing capability to task difficulty (cost-per-outcome discipline).
- **Confidence**: **Exploratory**.
- **Failure modes/confounds**: "right" model is task-dependent and unknowable without task context; cost ≠ quality.
- **Gaming risk**: Low.

### 2.6 Raw token usage / volume `[HAVE]`
- **Data dependency**: token usage per tool (HAVE).
- **Proxies**: **VOLUME only** — adoption magnitude, NOT quality.
- **Confidence**: **Exploratory** (explicitly *not* a quality metric).
- **Failure modes/confounds**: agentic tools burn 10–100× tokens for the same outcome [7]; volume decoupled from value.
- **Gaming risk**: **High** — trivially inflated. Included only to be explicitly down-weighted and as a denominator for normalization.

---

## Pillar 3 — Durability / Landing Quality (strongest for *quality*)

### 3.1 Code/suggestion survival & retention `[TURN-ON: Sourcegraph native | reconstruct from HAVE git metadata]`
- **Data dependency**: Sourcegraph time-bucketed persistence 30–600s [25] (native); OR reconstruct line-survival over N days from commit/LOC metadata [41][49] (HAVE). Copilot ~88% retention figure [10][29] (vendor, approximate).
- **Proxies**: **durability — the single most defensible quality signal** (H2). High survival = AI output that withstands reality.
- **Confidence**: **High** (Sourcegraph native, AI-attributed) / **Medium** (git-reconstructed, *overall* not AI-specific — see attribution caveat).
- **Failure modes/confounds**: **(1) AI-attribution gap** — outside Sourcegraph, this measures *overall* code survival, not AI-specific; AI usage only contributes to it [18] (precondition P4). **(2) Durability can reward timidity** — a dev writing trivial, low-ambition, copy-stable code shows high survival without doing anything hard; survival is a quality signal only alongside non-trivial work (use ticket/PR-size context). Legitimate iteration looks like low survival; refactors churn old lines; survival-window choice matters.
- **Gaming risk**: Low-Medium — hard to fake durable code, but gameable by avoiding ambitious/refactor work to keep survival high (the timidity trap above).

### 3.2 Code churn / rework rate `[HAVE: LOC+commit metadata]`
- **Data dependency**: lines reverted/revised within ~2 weeks of authoring [11][12][41]; added/deleted LOC + timestamps (HAVE).
- **Proxies**: rework risk — the GitClear AI-quality red flag ("high generation + low retention").
- **Confidence**: **Medium-High** (as *overall* churn) / lower as AI-specific (attribution gap P4 — most orgs can't isolate AI-authored lines [18]).
- **Failure modes/confounds**: **AI-attribution gap** — measures overall rework, not AI-specific, outside Sourcegraph; **inverse-timidity confound** — very low churn can mean unambitious work, not skill (pair with PR-size/ticket context); exploratory/spike work churns legitimately; early-stage code; not all churn is bad. Magnitudes vendor-reported [12].
- **Gaming risk**: Low-Medium — avoid revising own code (a *bad* behavior, so somewhat self-correcting); or avoid hard work to suppress churn.

### 3.3 Revert / rollback / hotfix frequency `[HAVE: commit metadata] + [TURN-ON: deploy data]`
- **Data dependency**: revert detection from commit subject/body [42] (HAVE, text-lite — message metadata not source); hotfix branch naming; deploy linkage (TURN-ON).
- **Proxies**: landing quality / change failure (DORA stability [1][2]).
- **Confidence**: **Medium**.
- **Failure modes/confounds**: reverts also reflect process/upstream issues, not the individual; sparse events → noisy.
- **Gaming risk**: Low.

### 3.4 First-pass CI success rate & trend `[HAVE: CI build state]`
- **Data dependency**: build pass/fail/cancel + timestamps (HAVE); rerun detection for flakiness [44].
- **Proxies**: does AI-assisted work pass on first run (durability at the gate).
- **Confidence**: **Medium**.
- **Failure modes/confounds**: **huge** — CI fails for a hundred reasons (env, flaky tests ~13% [44], deps, infra); strongly task/repo dependent. Personal-trend only.
- **Gaming risk**: Medium — run CI locally first / push only when green (arguably *good* behavior).

### 3.5 Build-failure-to-fix latency `[HAVE: CI timestamps]`
- **Data dependency**: first-fail → next-pass on same branch [43] (HAVE).
- **Proxies**: feedback-loop efficiency / recovery (DevEx [4]).
- **Confidence**: **Exploratory-Medium**.
- **Failure modes/confounds**: context-switching, working hours, severity.
- **Gaming risk**: Low.

### 3.6 Review-friction trend (changes-requested ratio, round-trips, turnaround) `[HAVE: review-outcome metadata]`
- **Data dependency**: review outcomes approved/changes-requested/commented + timestamps + reviewer/author ids [38][39] (HAVE).
- **Proxies**: landing quality / how cleanly AI-assisted output clears human review — as a *personal trend*.
- **Confidence**: **Medium**.
- **Failure modes/confounds**: tracks task difficulty, reviewer strictness, team norms, relationship dynamics; rising friction may mean harder work, not worse AI use.
- **Gaming risk**: Medium — choose lenient reviewers; under-review. Mitigation: trend + multi-reviewer aggregation.

### 3.7 PR rejection / abandonment rate `[HAVE: PR metadata]`
- **Data dependency**: closed-unmerged / total (HAVE).
- **Proxies**: wasted AI-assisted effort.
- **Confidence**: **Exploratory**.
- **Failure modes/confounds**: legitimate spikes/experiments; reorg/priority changes.
- **Gaming risk**: Low.

### 3.8 PR-feedback theme/sentiment `[TURN-ON: review feedback TEXT]`
- **Data dependency**: theme/sentiment of reviewer feedback text [brief notes limited text may exist; deliverable 04 rank #10]. **Touches feedback TEXT, NOT source code** — permitted but highest privacy cost.
- **Proxies**: *why* AI-assisted output gets pushback (e.g., "over-engineered," "untested").
- **Confidence**: **Exploratory**.
- **Failure modes/confounds**: NLP sentiment noise; reviewer tone varies; consent required.
- **Gaming risk**: Low.

---

## Cross-cutting / outcome-context metrics (interpretation lens, not individual score)

### 4.1 Team DORA metrics (deploy freq, lead time, change-fail rate, restore time) `[TURN-ON: CD data]`
- **Data dependency**: CD/deploy events + incident data [1][2][47].
- **Proxies**: whether the *environment* lets AI amplify up or down ("amplifier" lens [2]).
- **Confidence**: **High (team-level)** / **N/A (individual)**.
- **Failure modes/confounds**: **must not be attributed to an individual** (X13 — construct-validity violation). Context only.
- **Gaming risk**: Medium at team level.

### 4.2 Static-analysis / quality-gate status `[TURN-ON]`
- **Data dependency**: SonarQube/Code Climate gate Pass/Fail + issue/severity counts [45] — **status only, no code**.
- **Proxies**: did AI-assisted change clear the quality gate.
- **Confidence**: **Medium**.
- **Failure modes/confounds**: gate config varies; counts ≠ severity-weighted risk.
- **Gaming risk**: Medium — loosen gate thresholds [13].

### 4.3 Security-scan findings `[TURN-ON]`
- **Data dependency**: Snyk/Dependabot/Semgrep finding counts by severity [deliverable 04] — **counts only, no code**.
- **Proxies**: did AI-assisted change introduce vulnerabilities.
- **Confidence**: **Medium**.
- **Failure modes/confounds**: scanner FP rates; dependency findings ≠ authored code.
- **Gaming risk**: Low-Medium.

### 4.4 Test metadata (coverage delta, pass/fail, flaky rate) `[TURN-ON: CI]`
- **Data dependency**: coverage % deltas, test pass/fail counts, flaky-test rate [44] — **results, not test code**.
- **Proxies**: did AI-assisted work maintain test health.
- **Confidence**: **Medium**.
- **Failure modes/confounds**: coverage % is itself gameable [13]; flaky noise.
- **Gaming risk**: Medium — assertion-free tests to lift coverage.

### 4.5 Ticketing/PM signals (estimate accuracy, bug:feature, rework tickets) `[TURN-ON: Jira/Linear]`
- **Data dependency**: ticket throughput, points-vs-actual, bug ratio, backward-status moves [46].
- **Proxies**: business-outcome context; task-type normalization input.
- **Confidence**: **Medium**.
- **Failure modes/confounds**: estimation culture varies; ticket hygiene.
- **Gaming risk**: Medium — sandbagging estimates [13].

### 4.6 Incident / on-call linkage `[TURN-ON: PagerDuty]`
- **Data dependency**: incident timestamps linked to recent deploys [47].
- **Proxies**: production landing quality (ground-truth change failure).
- **Confidence**: **Medium (team)** / **Exploratory (individual)**.
- **Failure modes/confounds**: attribution to an individual is fraught; sparse.
- **Gaming risk**: Low.

---

## KILLED metrics (constraint #1 audit, cycle 3)
- `[KILLED]` **Complexity-trend of changed code** (cyclomatic/cognitive complexity of AI-assisted diffs). *Reason*: requires parsing diff **contents** — violates no-code constraint. Removed in red-team #1.
- `[KILLED]` **Prompt-quality scoring from prompt text**. *Reason*: prompt text is engineer-authored content not in our data and code-adjacent; out of scope (X15). Only prompt *length/count* metadata retained, cautiously (1.x).
- `[KILLED]` **Keystroke/screen-capture "real interaction depth."** *Reason*: GDPR data-minimization + surveillance ethics + observer effect [34][35] (X14).

---

## Summary: the metric tiers at a glance
| Quality dimension | Strongest metric(s) | Tier | Down-weighted volume metric (never alone) |
|---|---|---|---|
| Durability | Code survival/retention (3.1), churn (3.2) | High/Med-High | LOC, lines accepted |
| Landing quality | First-pass CI trend (3.4), review-friction trend (3.6), revert rate (3.3) | Medium | — |
| Adoption depth | Feature/mode mix (2.1), engaged rate (2.3) | Med-High/Med | token usage (2.6), consistency (2.4) |
| Leverage/efficiency | Cycle-time shape (1.1), self-report experience (1.4) | Medium | — |
| Context (lens only) | Team DORA (4.1), gate/security/test status (4.2–4.4) | High(team) | — |

**The headline**: durability metrics (survival, churn) are the most trustworthy quality proxies and uniquely well-served by the no-code constraint; volume metrics (tokens, raw acceptance, LOC) are explicitly demoted and never sufficient alone.
