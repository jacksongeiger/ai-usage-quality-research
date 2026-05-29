# RESEARCH ANGLES

Status legend: OPEN · EXPLORED · DISMISSED(reason)

## Cluster A — Established frameworks
- A1 DORA metrics + AI findings (2024 throughput/stability trade-off; 2025 amplifier thesis) — **EXPLORED** (cycle 1)
- A2 SPACE framework + why it warns against single/activity metrics — **EXPLORED** (cycle 1)
- A3 DevEx (flow, feedback loops, cognitive load) — **EXPLORED** (cycle 1)
- A4 DX / DX Core 4 + DX AI Measurement Framework (Utilization/Impact/Cost) — **EXPLORED** (cycle 1)
- A5 GitHub Copilot research (acceptance, survival, +55% RCT, perceived productivity) — **EXPLORED** (cycle 1)
- A6 GitClear churn/rework/refactoring decline (diff-metadata model) — **EXPLORED** (cycle 1)
- A7 McKinsey framework + Orosz/Beck critique — **EXPLORED** (cycle 1)
- A8 Thoughtworks Radar, METR independent RCT, vendor research (Faros/LinearB/Jellyfish/Swarmia) — **EXPLORED** (cycle 1)

## Cluster B — Vendor telemetry
- B1 GitHub Copilot Metrics API exact fields (engaged vs active, acceptance, survival, feature mix) — **EXPLORED** (cycle 1)
- B2 Cursor analytics/admin API fields — **EXPLORED** (cycle 1)
- B3 Claude Code OTel metrics/events (off by default) — **EXPLORED** (cycle 1)
- B4 Amazon Q Developer dashboard metrics — **EXPLORED** (cycle 1)
- B5 Sourcegraph Cody analytics (CAR/wCAR + time-bucketed persistence/survival) — **EXPLORED** (cycle 1)
- B6 Tabnine / Windsurf(Codeium) analytics — **EXPLORED** (cycle 1, field-level partly unverified)
- B7 Good-vs-poor AI usage patterns; why acceptance rate is a flawed proxy — **EXPLORED** (cycle 1)

## Cluster C — Validity & ethics
- C1 Goodhart's / Campbell's law + engineering gaming examples — **EXPLORED** (cycle 1)
- C2 Construct validity (convergent/discriminant/content/criterion, nomological net) — **EXPLORED** (cycle 1)
- C3 Correlation≠causation + confounds (tenure/role/repo/task) — **EXPLORED** (cycle 1)
- C4 Surveillance-vs-enablement ethics; GDPR/Article-29; observer/Hawthorne effect — **EXPLORED** (cycle 1)
- C5 Design principles for fair individual metrics (trend-vs-baseline, opt-in, triangulation) — **EXPLORED** (cycle 1)
- C6 AI perception gap (METR: felt +20% while −19%) implications for self-report — **EXPLORED** (cycle 1)

## Cluster D — Metadata signals + additional data
- D1 Cycle-time/PR-flow decomposition (coding/pickup/review/deploy) — **EXPLORED** (cycle 1)
- D2 Churn/rework proxies (line survival, 2-week churn, revert/hotfix, Diff Delta) — **EXPLORED** (cycle 1)
- D3 First-pass CI success + fix latency + flaky detection — **EXPLORED** (cycle 1)
- D4 Review friction (changes-requested ratio, round-trips, turnaround, abandonment) — **EXPLORED** (cycle 1)
- D5 Adoption depth/consistency (tool diversity, DAU/WAU, engaged-vs-active, recency) — **EXPLORED** (cycle 1)
- D6 Normalization/stats (PR-size norm, confound control, trend-vs-baseline, weak-signal combination) — **EXPLORED** (cycle 1)
- D7 Ranked additional-data table (value×realism÷privacy) — **EXPLORED** (cycle 1)

## Cross-cutting / emergent angles (generated during synthesis & red-team)
- X1 How to map signals → the four recommendation *types* (tools/skills/agents/workflows) specifically — **EXPLORED** (cycle 2, deliverable 03)
- X2 The "amplifier" framing as the spine connecting individual signals to team context — **EXPLORED** (cycle 2)
- X3 Multi-tool normalization (≥82% use 3+ tools) — how to score a person across their tool portfolio — **EXPLORED** (cycle 2, metric N-norm)
- X4 The "knowing when NOT to use AI" dimension (content validity gap) — how to proxy abstention without code — **EXPLORED** (cycle 3 red-team; partial proxy only, logged as limitation)
- X5 Distinguishing "shallow autocomplete" vs "deep agentic" usage as a quality axis — **EXPLORED** (cycle 2, feature-mix metric)
- X6 Cold-start / low-data engineers (new hires, light AI users) — how to avoid false signals — **EXPLORED** (cycle 3, confidence-gating)
- X7 Survival/retention reconstruction from git metadata when vendor doesn't expose it (only Sourcegraph does natively) — **EXPLORED** (cycle 2)
- X8 Gaming surface of each recommendation metric + mitigations — **EXPLORED** (cycle 3 red-team)
- X9 Worked-example construction (2 personas end-to-end) — **EXPLORED** (cycle 2, deliverable 07)
- X10 Whether any proposed metric secretly needs code access (audit pass) — **EXPLORED** (cycle 3 red-team; one candidate killed — see log)
- X11 Composite-index vs signal-agreement: which is safer against Goodhart — **EXPLORED** (cycle 3; chose signal-agreement, no single published score)
- X12 Feedback *tone*/delivery as part of validity (coaching vs grading) — **EXPLORED** (cycle 2, deliverable 06)
- X13 DORA stability-decline as an individual-level caution signal vs team-only — **DISMISSED** (DORA is team/system-level by design; using it per-individual violates construct validity — kept strictly as team context. Logged red-team cycle 3.)
- X14 Keystroke/screen monitoring for "real" AI interaction depth — **DISMISSED** (fails GDPR data-minimization + surveillance ethics + observer effect; out of bounds for enablement framing.)
- X15 Inferring prompt quality from prompt text content — **DISMISSED** (prompt text is arguably engineer-authored "code-adjacent" content and not in our data; treated as out-of-scope; only prompt *length/count* metadata used, cautiously.)
- X16 Peer/percentile ranking of engineers — **DISMISSED** (violates constraint #2 + confound-invalidity; replaced with trend-vs-personal-baseline.)
