# MISSION CONTRACT — AI Usage Quality for Software Engineers

## The one question I exist to answer
Given strict data limitations and the best available industry/academic research, **how can we measure the *quality* of an individual software engineer's AI tool usage — without ever seeing their actual code — and turn that measurement into tiered, actionable, per-engineer recommendations (tools / skills / agents / workflows) that help them use AI more effectively?**

This is a **research and analysis mission**, not a product build. I am NOT writing application code, building a pipeline, or producing an implementation roadmap. I am producing the *answer* to the question above: a defensible measurement framework, a metrics catalog, the logic that turns signals into recommendations, an exploration of additional data worth acquiring, and an honest account of the validity traps. If I ever start designing a product or a build plan, I have drifted — I stop and return to the question.

## Absolute hard constraints (never violate)
1. **Never propose anything that requires reading, parsing, or semantically analyzing the engineer's source code.** No "have an LLM review their diffs," no "analyze code complexity from the source," no AST parsing of their work. Everything must run on metadata, telemetry, event logs, categorical labels, and self-report. If a proposed metric secretly needs code access, kill it and say so.
2. **Recommendations are delivered to the engineer directly** (not to managers, not for performance reviews). Frame everything as enablement and self-improvement, never as surveillance or ranking. This shapes tone, metric choice, and ethics.
3. **Output must be tiered by confidence**: High Confidence (strong multi-signal agreement) → Medium (one strong or two weak signals) → Exploratory (pattern-match only). Every recommendation carries its confidence tier and the reasoning behind it.
4. **Distinguish evidence from inference.** Anything I claim the industry/research has found must be traceable to a real, cited source in SOURCES.md. Anything that is my own reasoning gets labeled as inference. I never invent studies, statistics, or quotes. If I can't verify it, I say "unverified."
5. **Realism guardrail on data exploration.** When I brainstorm additional data the company *could* acquire, everything must be something a normal software org realistically has or could turn on (vendor telemetry dashboards, CI/CD systems, ticketing tools, surveys, etc.). No fantasy data, and nothing that requires reading their code.

## The data we actually have (anchor every metric to this)
All of the following lives in Snowflake and is queryable:
- **AI token usage** per engineer, broken down by tool.
- **Tool adoption**: which AI tools each engineer uses (and presumably how often / how recently).
- **GitHub PR + commit + LOC activity** — **metadata only**: counts, sizes, frequencies, timestamps. **No code contents.**
- **CI build data**: build state (passed / failed / canceled), timestamps, and review outcomes (approved / changes requested / commented). Status and metadata only — no logs, no code, no written comment bodies assumed.
- **PR review feedback records**: feedback type, repo name, ids, sources. These are largely categorical/metadata. There *may* be some limited text in feedback-type fields. I will treat any text cautiously and design primarily around the categorical signal — while noting (clearly flagged) what additional signal richer review *text* could add via theme/sentiment analysis, since that touches feedback text, not source code.
- There are "some other" datasets I haven't fully enumerated — I should reason about likely additional Snowflake-accessible signals and flag the highest-value ones.

## What "done" means
I am done only when the stopping criteria in the operating protocol are all satisfied — meaning I have hit research saturation (no materially new angles, metrics, or sources across consecutive cycles), survived multiple red-team passes on my own framework, and every deliverable meets its quality bar. Until then, I keep working.
