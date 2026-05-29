# 04 — Additional Data Sources (ranked exploration)

Realistic, no-code data the company could acquire to materially improve (a) measuring AI usage quality or (b) generating better recommendations. Every candidate is something a normal software org **already has or can turn on** (vendor dashboards, CI/CD, ticketing, surveys) — no fantasy data, nothing requiring source-code reading.

## Scoring method
**Score = (Signal Value × Realism) ÷ Privacy Cost**, each rated 1–5.
- *Signal Value*: how much it improves measurement/recommendations.
- *Realism*: how easily a normal org can obtain/ingest it (5 = toggle-on / already emitted).
- *Privacy Cost*: individual-privacy / surveillance-optics burden (1 = artifact-level, 5 = person-level intrusive).
- *No-code verdict*: Clean / Borderline / Out-of-bounds.

> Note: rank order below blends the ratio with strategic importance. The ratio rewards low-privacy artifact data; but the two highest-*value* sources (IDE telemetry, surveys) are listed first because they are the *only* sources that directly measure AI-usage **quality** and its perceived value — everything else is a downstream proxy. Both interpretations are shown so the trade-off is explicit.

## The ranked table

| # | Source | Signal it adds | SV | Real | Priv | Ratio | No-code |
|---|---|---|---|---|---|---|---|
| 1 | **IDE/assistant vendor telemetry** (acceptance, accept-latency, **retention/survival of accepted AI code**, chat-vs-completion-vs-agent mix, session depth, per-feature use) [19][22][23][25] | The *only direct* measure of AI-usage quality & depth; survival/revision-depth > acceptance | 5 | 5 | 2 | 12.5 | **Clean** — counts/telemetry, no code |
| 2 | **Developer self-report / surveys** (SPACE "S"; DX perceived-productivity & AI-helpfulness) [3][4][7] | The validating perceptual dimension; *why* + felt helpfulness; ties subjective↔objective | 5 | 5 | 2 | 12.5 | **Clean** — opinions, no code |
| 3 | **DORA-grade deployment data + revert/rollback/hotfix freq** (CD tooling) [1][2][42][47] | Deploy freq + **change failure rate**; rollback/hotfix = direct landing-quality outcome | 5 | 4 | 1 | 20 | **Clean** — deploy events/timestamps |
| 4 | **Test metadata from CI** (coverage % deltas, pass/fail counts, flaky-test rate) [44] | Did AI-assisted work keep tests green / coverage stable? Flaky-rate isolates noise | 4 | 4 | 1 | 16 | **Clean** — test *results*, not test code |
| 5 | **Static-analysis / quality-gate status** (SonarQube, Code Climate) [45] | Gate Pass/Fail + issue/severity *counts*; "Sonar way for AI Code" gate exists | 4 | 4 | 1 | 16 | **Clean** — gate status/counts only |
| 6 | **Security-scan results** (Snyk, Dependabot, Semgrep) | New-finding counts by severity per change — AI-introduced-vuln signal | 4 | 4 | 1 | 16 | **Clean** — counts/severity, not code |
| 7 | **Ticketing/PM data** (Jira/Linear): throughput, **estimate accuracy**, bug:feature ratio, **rework tickets**, ticket cycle time [46] | Business-outcome context; task-type normalization; rework signal | 4 | 4 | 2 | 8 | **Clean** — workflow metadata |
| 8 | **Incident/on-call data** (PagerDuty/Opsgenie) linked to recent changes [47] | Ground-truth CFR/MTTR; ties AI-assisted deploys to real prod failures | 4 | 3 | 2 | 6 | **Clean** — incident timestamps/links |
| 9 | **Comms/collaboration metadata** (PR comment *counts*, response latency; calendar/focus-time) [4] | Cognitive-load/flow + collaboration burden | 3 | 4 | 4 | 3 | **Clean if counts-only**; calendar raises privacy sharply |
| 10 | **Richer PR review feedback TEXT** (theme/sentiment of reviewer comments) | *Why* AI code gets pushback ("over-engineered", "untested") | 4 | 3 | 4 | 2.25 | **Borderline** — touches *feedback text*, NOT source code; needs aggregation + consent |

## Rationale & notes per tier

**Top by ratio (3–6: deploy / test / gate / security, scores 16–20).** Outcome-anchored, almost always already on in a normal org, and near-zero personal-privacy cost because they describe *artifacts*, not people. These are the cheapest high-integrity additions and should be ingested first. They strengthen Pillar 3 (Durability/Landing-quality) — the most trustworthy quality pillar — and provide the DORA "amplifier" context lens [2].

**Highest value (1–2: IDE telemetry + surveys, ratio 12.5).** Top signal value (5) and realism (5) but each carries privacy weight 2 (individual-attributable usage / personal opinions), pulling the ratio below the artifact sources. They rank first on *importance* because:
- **IDE telemetry** is the only source that directly observes *how* AI is used (mode mix, survival, revision depth). Critically, **retention/survival is the cleanest quality signal and only Sourcegraph exposes it natively** [25]; for Copilot/Cursor/Claude Code it must be reconstructed from git history. Much of this requires *turning on* telemetry not yet in Snowflake: **Claude Code OTel is OFF by default** [23] (needs `CLAUDE_CODE_ENABLE_TELEMETRY=1` + a collector + authenticated user attribution); Copilot **excludes <5-seat teams** [19]; Cursor has a 90-day window cap [22].
- **Surveys are the standout, most-overlooked, highest-leverage addition.** SPACE [3] and DX Core 4 [6] both insist a perceptual dimension is *mandatory* and that objective signals are uninterpretable without it. Surveys are cheap, fast, and supply the convergent-validity partner for every objective metric. Best practice (from [4][7]): short pulse cadence, anonymized/aggregated reporting, per-persona segmentation, and explicit linking of subjective responses to objective signals (the perception-gap calibration in deliverable 03 D1).

**Bottom by ratio (9–10).** Comms/calendar (privacy 4 — surveillance optics) and review-text sentiment (privacy 4 + the *only* item touching human-written text) score lowest. Both are usable but must be **aggregated, opt-in, and never individually punitive**. Review-feedback text is explicitly **Borderline**: permitted under the constraint (it is feedback text, not source code) but it is the highest-privacy item and should be the *last* to adopt, behind clear consent.

## Recommended acquisition sequence (inference)
1. **First (free, clean, high-integrity):** deploy/CD + revert (#3), test metadata (#4), gate status (#5), security findings (#6). Strengthens the durability pillar immediately at near-zero privacy cost.
2. **Second (highest value, modest privacy):** turn on full IDE/assistant telemetry (#1) — especially survival/retention (Sourcegraph native, or git-reconstruct) and feature-mix. This is what converts "we know token counts" into "we know usage *quality*."
3. **Third (the multiplier):** stand up a lightweight, anonymized **developer survey** program (#2) to validate every objective signal and power perception-gap coaching.
4. **Later / cautious:** ticketing (#7) and incident (#8) for context; comms-counts (#9) only if cognitive-load work is prioritized; review-text sentiment (#10) last, with consent.

## What we deliberately excluded (no-code / ethics guardrail)
- Anything reading diff/source contents (complexity, AST, semantic diff review) — violates constraint #1.
- Keystroke logging, screen capture, always-on screen-time — fails GDPR data-minimization [34] and the enablement framing [35][36].
- Prompt-text content analysis — code-adjacent engineer-authored content, out of scope (only prompt length/count metadata retained, cautiously).
