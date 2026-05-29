# AI Usage Quality Research

**An exhaustively-researched, self-critiqued answer to one question:**

> How can we measure the *quality* of an individual software engineer's AI tool usage — **without ever seeing their code** — and turn that into tiered, actionable, per-engineer recommendations that help them use AI more effectively, delivered to the engineer as enablement?

This is a **research & analysis** project, not a product build. It produces a defensible measurement framework, a metrics catalog, signal→recommendation logic, an additional-data exploration, an industry/academic synthesis, and an honest account of the validity and ethics traps — all grounded in ~34 cited sources and stress-tested by multiple adversarial red-team passes.

## Start here
- **[`deliverables/00_EXECUTIVE_SUMMARY.md`](deliverables/00_EXECUTIVE_SUMMARY.md)** — the complete answer in ~5 minutes.

## Deliverables
| File | What it covers |
|---|---|
| [00_EXECUTIVE_SUMMARY](deliverables/00_EXECUTIVE_SUMMARY.md) | The crisp, complete answer + headline caveats |
| [01_FRAMEWORK](deliverables/01_FRAMEWORK.md) | The "AI usage quality" measurement model (3 no-code pillars, construct-validity argument) |
| [02_METRICS_CATALOG](deliverables/02_METRICS_CATALOG.md) | Every metric: data dependency, what it proxies, confidence tier, failure modes, gaming risk |
| [03_SIGNAL_TO_RECOMMENDATION_LOGIC](deliverables/03_SIGNAL_TO_RECOMMENDATION_LOGIC.md) | How signals become tiered tool/skill/agent/workflow recommendations |
| [04_ADDITIONAL_DATA_SOURCES](deliverables/04_ADDITIONAL_DATA_SOURCES.md) | Ranked, realism-checked exploration of data worth acquiring |
| [05_INDUSTRY_RESEARCH_SYNTHESIS](deliverables/05_INDUSTRY_RESEARCH_SYNTHESIS.md) | How the top of industry/academia measure this & where they disagree |
| [06_PITFALLS_VALIDITY_AND_ETHICS](deliverables/06_PITFALLS_VALIDITY_AND_ETHICS.md) | Goodhart/gaming, construct validity, confounds, surveillance-vs-enablement |
| [07_WORKED_EXAMPLE](deliverables/07_WORKED_EXAMPLE.md) | Two fictional engineers run end-to-end through the framework |

## The answer in one paragraph
Define AI usage quality as **durable, low-friction, well-matched leverage from AI** — never code quality, productivity, or activity. Measure it as a latent construct through three no-code proxy pillars (**Durability/Landing-quality**, **Adoption Depth**, **Leverage/Efficiency**), trust a finding only where independent signals agree, express every output in confidence tiers (High/Medium/Exploratory), compare each engineer only to their own trend baseline, read everything through DORA's "AI is an amplifier" lens, and deliver it to the engineer alone as coaching. Volume (tokens, raw acceptance, lines) is structurally demoted and can never, by itself, signal quality.

## Honest status
This is a **literature-grounded design, not a validated instrument.** The convergent-validity argument is theoretical; before production use it needs an empirical validation step (correlate the proxies against an independent, no-code criterion). The biggest technical caveat is the **AI-attribution gap**: outside Sourcegraph, durability signals usually measure *overall* code survival/churn, not AI-specific. See [06](deliverables/06_PITFALLS_VALIDITY_AND_ETHICS.md) for the full limitations and the realistic ~4-signal MVP.

## How this was produced
A fully autonomous research mission: workspace scaffolded under a mission contract ([`CLAUDE.md`](CLAUDE.md)), four parallel deep-research agents across the framework / vendor-telemetry / validity-ethics / metadata-signal clusters, synthesis into the deliverables, then multiple red-team passes (including a post-delivery skeptic pass). The full audit trail lives in [`state/`](state/) (angles, open questions, assumptions, research log, saturation tracker, skeptic review) and every source is annotated in [`sources/SOURCES.md`](sources/SOURCES.md).

## Repository layout
```
.
├── CLAUDE.md            # the mission contract
├── README.md           # this file
├── deliverables/       # 00–07, the answer
├── sources/SOURCES.md  # ~34 annotated, triangulated sources
└── state/              # full research audit trail
```

> Sources are tagged Primary / Vendor / Independent / Practitioner, and any claim not verified from a primary text is flagged. Vendor claims are triangulated against independent evidence (e.g., the METR RCT) throughout.
