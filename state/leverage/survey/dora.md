# DORA — Four Keys + DORA AI Reports 2023–2026

Sub-investigator survey file. Mission: per-engineer "AI-assisted engineering leverage" score.
All quotes verified by direct fetch on 2026-07-08 unless marked otherwise. My own reasoning is labeled **[inference]**.

---

## 1. What it measures & theory

### The keys (current canonical definitions, dora.dev)

DORA's software-delivery metrics are now **five**, reorganized in the 2025 cycle [primary: dora.dev four-keys guide]:

**Throughput:**
1. **Change lead time** — "The amount of time it takes for a change to go from committed to version control to deployed in production."
2. **Deployment frequency** — "The number of deployments over a given period or the time between deployments."
3. **Failed deployment recovery time** — "The time it takes to recover from a deployment that fails and requires immediate intervention." (Moved from stability to throughput in the 2025 restructuring; formerly "time to restore service"/MTTR.)

**Instability** (category renamed from "stability" because high values = bad):
4. **Change fail rate** — "The ratio of deployments that require immediate intervention following a deployment."
5. **Rework rate** (new, 2025) — "The ratio of deployments that are unplanned but happen as a result of an incident in production." [primary: dora.dev; independent confirmation: CD Foundation, Oct 2025]

### Theory
- Delivery performance is a property of a **sociotechnical system** (team + pipeline + architecture + culture), measured via annual surveys (Accelerate research program, Forsgren/Humble/Kim, running since 2014; now Google Cloud).
- Core empirical claim across a decade: speed and stability are **not** a trade-off — top performers achieve both — and delivery performance predicts organizational performance. **[inference from the report series; consistent across 2023–2025 reports]**
- Explicit anti-target stance from DORA itself: avoid "Setting metrics as a goal," which "encourages gaming the system"; avoid "Competing" against other teams; avoid "Making disparate comparisons" between unlike applications [primary: dora.dev four-keys guide].

### The AI arc, 2023 → 2026

| Year | Report | Key AI finding |
|---|---|---|
| 2023 | Accelerate State of DevOps 2023 (n > 36,000) | AI adoption measured but **no performance link claimed**: "the impact of AI development tools on teams is still in its infancy." [primary: Google Cloud 2023 announcement] |
| 2024 (mid) | *Impact of Generative AI in Software Development* | Individual-level gains: flow, satisfaction, productivity, less burnout; ~67% say AI improves their code; "developers who trust generative AI accept more suggestions, submit more change lists." [primary: dora.dev/ai/gen-ai-report; per-claim details from search-result summary — spot-check PDF before quoting numbers downstream] |
| 2024 | Accelerate State of DevOps 2024 | >75% "rely on AI for at least one daily professional responsibility"; 39% report "little to no trust in AI-generated code." A **25% increase in AI adoption** is associated with: **+7.5% documentation quality, +3.4% code quality, +3.1% code review speed — but −1.5% delivery throughput and −7.2% delivery stability**: "As AI adoption increased, it was accompanied by an estimated decrease in delivery throughput by 1.5%, and an estimated reduction in delivery stability by 7.2%." Explanation offered: neglect of "small batch sizes and robust testing mechanisms." [primary: Google Cloud 2024 announcement] |
| 2025 | **State of AI-assisted Software Development 2025** (~5,000 respondents + "over 100 hours of qualitative data") | 90% use AI at work; >80% believe it increased their productivity; 30% still report little/no trust in AI-generated code. Sign flip on throughput: "Unlike last year, we observe a positive relationship between AI adoption on both software delivery throughput and product performance." But: "AI adoption does continue to have a negative relationship with software delivery stability." Central thesis: **"AI's primary role is as an amplifier, magnifying an organization's existing strengths and weaknesses."** Mechanism: "AI accelerates software development, but that acceleration can expose weaknesses downstream. Without robust control systems… an increase in change volume leads to instability." [primary: dora.dev/dora-report-2025 + Google Cloud 2025 announcement] |
| 2025 (companion) | **DORA AI Capabilities Model** — 7 capabilities that amplify AI's benefit: **clear and communicated AI stance; healthy data ecosystems; AI-accessible internal data; strong version control practices; working in small batches; user-centric focus; quality internal platforms.** [primary: Google Cloud "Introducing DORA's inaugural AI Capabilities Model"; dora.dev/ai/capabilities-model/report/] Also 7 team archetypes, from "Foundational challenges" to "Harmonious high achievers." |
| 2026 | **ROI of AI-assisted Software Development (2026.01)**, published ~May 11, 2026 | ROI framed at the org level, explicitly NOT per-developer-output: "Return on investment is no longer a measure of how many developers an organization can replace"; "We don't measure AI by the code it writes but by the bottlenecks it clears"; without strong foundations "AI creates localized pockets of productivity that are often lost in downstream chaos." Worked model: 500-eng org, ~39% first-year ROI, 8-month payback. [independent: InfoQ, May 2026; primary landing: dora.dev/ai/roi/report/] |
| 2026 | **No 2026 "State of…" report exists yet** as of 2026-07-08 — the dora.dev publications page lists nothing newer than the 2025 pair plus the ROI report. [primary: dora.dev/research/publications/] |

### DORA on INDIVIDUAL-level measurement — and why it's team-level

- Unit of analysis, verbatim: **"These metrics are meant to be applied at the application or service level"** [primary: dora.dev four-keys guide]. Also: "In October 2023, the DORA team cautioned against using these metrics to compare teams… Creating league tables leads to unhealthy comparisons and counterproductive competition." [vendor: Octopus Deploy guide, quoting DORA guidance]
- Nathen Harvey (DORA lead, Google Cloud), verbatim: **"These truly are team metrics. And rolling them up organization wide, I think there is a lot of peril to be found there."** And: "Really, it's the cross-functional team that owns that application or service." On targets: "When that becomes the goal to hit that metric, that really encourages people to game that metric." [primary-interview: DX podcast "Mastering DORA metrics with Google's Nathen Harvey"]
- Bryan Finster (IT Revolution whitepaper): "measure processes, not individuals"; treat metrics as "team-level improvement tools, not performance ratings." [practitioner]
- **Why they cannot be individualized [inference, grounded in the definitions]:** every key is a property of shared machinery. A deployment bundles many engineers' changes (deployment frequency and rework rate are pipeline/service properties); lead time is dominated by review latency, CI capacity, and release cadence outside one person's control; change-fail attribution is confounded by reviewers, test infra, and on-call; recovery time belongs to whoever is on call, not whoever wrote the change. Individual attribution either double-counts, mis-assigns blame, or measures the environment rather than the person.
- **The most load-bearing DORA lesson for a per-engineer score [inference]:** DORA 2024 documented that **individual-level gains and system-level outcomes diverge under AI** — individuals report productivity/flow gains while delivery stability drops 7.2% per 25% adoption increase. So a per-engineer score built purely on individual output (PR count, lines) can go UP while the engineer's actual system contribution goes DOWN (bigger batches, review burden exported to teammates, instability shipped downstream). Any honest "leverage" score must not reward that divergence.

---

## 2. Evidence quality

- **Survey self-report, cross-sectional, correlational.** Throughput/stability are respondents' perceptions, not telemetry. DORA's own language is "associated with" / "estimated"; no causal identification. [inference from primary wording]
- **Sample composition swings hard**: >36,000 (2023, after a "3.6x" organic increase) → ~5,000 (2025). Year-over-year comparability is weak. [primary announcements]
- **The headline AI effect flipped sign in one year** (throughput −1.5% in 2024 → positive in 2025). Could be real (better tools/agentic workflows), or sample/model change. DORA itself does not adjudicate. Treat any single-year coefficient as unstable. [inference]
- **Sponsor conflict**: research funded and published by Google Cloud, which sells AI dev tooling — findings are still the field's best longitudinal series, but headline framings ("amplifier") serve a narrative. Tag primary-with-vendor-interest.
- **Strengths**: decade of replication, large N, mixed methods in 2025 (qualitative hours), published capability models with mechanisms (small batches, version control) rather than bare correlations, and independent triangulation (InfoQ Sep 2025 & May 2026; RedMonk — the latter cited in this repo's SOURCES.md [1],[2]).
- The 4 keys themselves have well-known construct-validity critiques from practitioners: vendor dashboards mis-measure them ("change fail almost never measured correctly in vendor solutions" — Finster [practitioner, LinkedIn, via search snippet — unverified verbatim]), and skewed distributions make averages misleading for recovery time [vendor: Octopus].

---

## 3. Known failure modes (per key)

1. **Deployment frequency** — counts events, not value; suppressed by manual pipeline gates regardless of engineer skill ("If you have manual tasks in your deployment pipeline, teams tend to deploy less often" [vendor: Octopus]); meaningless across release models (mobile store review, firmware, regulated).
2. **Change lead time** — start-point ambiguity (first commit? PR open? ticket?); dominated by wait states (review queues, CI queues) not work; punishes deliberate validation.
3. **Change fail rate** — definitional swamp: what counts as "failure" and what counts as "deployment" both drift ("A successful build is not a deployment. A push to staging is not a deployment in DORA terms" [practitioner: cicd.watch]); severity-blind ("a cosmetic CSS regression and a checkout outage both count as one failed deployment"); denominators differ per vendor tool.
4. **Failed deployment recovery time** — heavy-tailed distribution, "averages can be misleading" [vendor: Octopus]; measures on-call/incident process more than code quality; sensitive to incident-severity classification policy.
5. **Rework rate** — newest, least standardized; depends entirely on "planned vs unplanned" labeling discipline. [inference from definition]
6. **Cross-cutting** — metrics-as-targets failure (DORA's own warning); "vanity radiators" dashboards with no improvement loop; speed measured without quality guardrails "will result in poor outcomes" [practitioner: Finster via Abi Noda's summary].

---

## 4. Gameability & who each key penalizes

| Key | How an engineer/team games it | Who it unfairly penalizes |
|---|---|---|
| Deployment frequency | Split work into many trivial/no-op deploys; redefine "deployment" (count staging pushes, flag flips); ship rushed changes to pump the count | Mobile/desktop/firmware/regulated teams on release trains; monorepo teams with batched deploys; anyone behind a manual change-approval board |
| Change lead time | **Hide work locally, commit only when done** — Harvey verbatim: "I'm going to do a whole bunch of work over here on the side that's invisible. And when I know it's good to go, then I'll put it into the repository and commit it, and then it'll flow through very fast. No, don't do that."; cherry-pick the clock-start definition; skip validation steps | Engineers doing complex/risky/cross-team work; timezone-split teams (review wait); heavy-CI repos |
| Change fail rate | **Definitional drift** — "a hotfix within 24 hours… gets reclassified as a planned follow-up"; "an incident… gets recorded as not deploy-related"; "the production environment gets redefined so a class of deploys no longer counts" [cicd.watch]; also risk-aversion: stop shipping anything risky | Teams with honest incident-reporting cultures; high-traffic services where failures are visible; teams owning inherently risky surfaces (data migrations, payments) |
| Recovery time | Close incidents fast and reopen as "new" ones; exclude outliers; downgrade severity so the clock never starts | Owners of stateful systems (restores are slow by physics); on-call engineers inheriting others' failures |
| Rework rate | Relabel unplanned deploys as planned follow-ups (same drift lever as CFR) | Teams that document reality; platform teams whose "unplanned" deploys respond to others' incidents |

**Meta-point [inference]:** the four keys' gaming levers are almost all *classification* levers (what counts as a deploy/failure/incident), not *effort* levers — cicd.watch: "The only way it does get gamed" is definitional drift. This is exactly why DORA keeps them team-level and target-free: at individual level, every engineer becomes their own classifier, and Goodhart wins immediately.

---

## 5. Applicability to our constraints (HAVE / CANNOT mapping)

**Direct reuse of the four keys per engineer: NOT POSSIBLE and NOT ADVISABLE.**
- We lack deployment events, production incident data, and per-author post-merge defect linkage (CANNOT list). The keys need all three. Our per-PR CI signal is *pre-merge*, which is upstream of everything DORA measures.
- Even with the data, DORA's own guidance (unit = application/service; Harvey's "truly team metrics") says a "personal DORA score" is a category error. **Log this as a dead end for the design doc.**

**What DORA DOES give our per-engineer design (all mappable to HAVE):**
1. **Counterbalance principle** → never ship a throughput-ish subscore (merged PRs, lines) without a stability-ish counterweight. Our only counterweight today, CI-clean-rate, is saturated (brief problems #1/#7) — it behaves like a CFR that trends to zero via iteration-to-green, the exact "definition stops discriminating" failure above. [inference]
2. **"Working in small batches" is one of DORA's 7 AI capabilities** and its proposed mechanism for AI-driven instability (batch-size inflation). We HAVE lines-changed per PR → a **PR batch-size discipline signal** (e.g., median PR size, share of PRs under a threshold) is a DORA-endorsed, individual-controllable behavior. Note it *inverts* the current metric, which rewards MORE lines per dollar. Gaming: artificial PR-splitting (stacked trivial PRs) — bound it with a per-PR floor or diminishing returns; penalizes: engineers doing legitimate large migrations/codegen — winsorize the reward, don't punish size, just stop rewarding it. [inference]
3. **Amplifier thesis → context conditioning.** AI leverage is conditional on team/system environment; therefore cohort by **function + repo environment**, not job-level-only (fixes brief problem #6). A per-engineer score computed against a cross-function cohort imports team-level confounds into an individual rank — the "disparate comparisons" DORA forbids.
4. **Individual vs system divergence (2024 finding) → score behaviors, show outcomes.** Per-engineer scoring should target individual-controllable *inputs/behaviors* (adoption breadth/depth across the tool portfolio incl. the agent platform — all in HAVE; batch-size discipline; active-day consistency), while delivery-ish outcomes are displayed as team-level *context*, never as personal credit. This also rescues agent-platform engineers (spend/sessions/run counts exist in HAVE even though outputs don't).
5. **2026 ROI report framing** — "We don't measure AI by the code it writes but by the bottlenecks it clears" — is a primary-source endorsement of dropping code-volume numerators, which supports retiring signal #2 of the current metric (lines/$).
6. **Percentile-ranking within cohort ≈ league table.** Self-view-only softens but does not remove the harm: the rank is still built by comparing individuals whose environments differ (DORA's core objection). Prefer self-vs-self trend + anonymous cohort distribution bands. [inference]

**For every metric touched here, gaming + penalty analysis is in §4; for the derived signals proposed in §5, gaming/penalty is stated inline.**

---

## 6. Sources (annotated)

1. [primary] DORA — "DORA's software delivery performance metrics" (four/five keys guide). https://dora.dev/guides/dora-metrics-four-keys/ — definitions of all 5 metrics; "applied at the application or service level"; anti-gaming warnings. Fetched 2026-07-08.
2. [primary] DORA — State of AI-assisted Software Development 2025 landing. https://dora.dev/dora-report-2025/ — amplifier thesis verbatim. Fetched.
3. [primary] Google Cloud — "Announcing the 2025 DORA report." https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report — 90% adoption, 80% productivity, 30% distrust, throughput sign-flip, stability still negative, ~5,000 respondents. Fetched.
4. [primary] Google Cloud — "Announcing the 2024 DORA report." https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report — 25%-adoption-increase effects: +7.5% docs, +3.4% code quality, +3.1% review speed, −1.5% throughput, −7.2% stability; 39% distrust. Fetched.
5. [primary] Google Cloud — "Announcing the 2023 State of DevOps Report." https://cloud.google.com/blog/products/devops-sre/announcing-the-2023-state-of-devops-report — n>36,000; AI impact "still in its infancy," no performance link claimed. Fetched.
6. [primary] DORA — Impact of Generative AI in Software Development (2024). https://dora.dev/ai/gen-ai-report/ (PDF: https://services.google.com/fh/files/misc/dora-impact-of-generative-ai-in-software-development.pdf) — individual flow/satisfaction gains; trust→acceptance link. Landing verified; detailed stats from search summaries — spot-check PDF before reusing numbers.
7. [primary] Google Cloud — "Introducing DORA's inaugural AI Capabilities Model." https://cloud.google.com/blog/products/ai-machine-learning/introducing-doras-inaugural-ai-capabilities-model (report: https://dora.dev/ai/capabilities-model/report/) — the 7 capabilities incl. "working in small batches." Names confirmed via search across multiple pages.
8. [primary] DORA — ROI of AI-assisted Software Development report. https://dora.dev/ai/roi/report/ — landing only. [independent] InfoQ, May 2026: https://www.infoq.com/news/2026/05/dora-roi-ai-assisted-dev-report/ — "bottlenecks it clears" quote, 39% ROI model, May 11 2026 date. Fetched.
9. [primary] DORA publications index. https://dora.dev/research/publications/ — confirms no 2026 State-of report as of 2026-07-08. Fetched.
10. [primary-interview] DX podcast — "Mastering DORA metrics with Google's Nathen Harvey." https://getdx.com/podcast/masterclass-on-dora-metrics/ — "truly team metrics"; hidden-work lead-time gaming quote. Fetched.
11. [practitioner] Bryan Finster — "How to Misuse & Abuse DORA Metrics" (IT Revolution whitepaper). https://bryanfinster.com/whitepapers/dora-metrics + summary https://newsletter.getdx.com/p/misuse-dora — "measure processes, not individuals"; speed-without-quality-guardrails warning. Both fetched. DOES2021 deck: https://github.com/devopsenterprise/2021-virtual-us/blob/main/Bryan%20Finster%20-%20DOES%202021%20-%20Misuse%20and%20Abuse%20DORA%20Metrics.pdf (not fetched).
12. [practitioner] CI/CD Watch — "Change Failure Rate: The Stability Metric You Can't Fake." https://cicd.watch/blog/change-failure-rate — definitional-drift gaming taxonomy, deployment-rule discipline. Fetched.
13. [vendor] Octopus Deploy — DORA metrics guide. https://octopus.com/devops/metrics/dora-metrics/ — per-key pitfalls; "cautioned against using these metrics to compare teams"; league-table warning; skewed recovery-time distributions. Fetched.
14. [independent] CD Foundation — "The DORA 4 key metrics become 5" (Oct 16, 2025). https://cd.foundation/blog/2025/10/16/dora-5-metrics/ — rework rate; recovery time moved to throughput; stability→instability rename. Fetched.
15. [independent] InfoQ — "DORA Report Finds AI Is an Amplifier…, But Trust Remains Low" (Sep 2025). https://www.infoq.com/news/2025/09/dora-state-of-ai-in-dev-2025/ — triangulation of 2025 findings. Title-level only, not fetched.
16. [independent] Jellyfish — "AI as Amplifier: … with Lead Author Nathen Harvey." https://jellyfish.co/blog/2025-dora-report/ — not fetched; candidate for deeper Harvey quotes if needed.
