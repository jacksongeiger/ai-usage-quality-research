# RESEARCH BRIEF — "AI-Assisted Engineering Leverage" Metric Design (July 2026)

**Read this whole file before doing anything.** You are one sub-investigator in a multi-agent, staff-level research mission. Today's date: **2026-07-08**. Prefer recent sources; note publication dates. The final deliverable is `leverage-measurement-findings.md` at the root of this repo (`/Users/jacksongeiger/ai-usage-quality-research/`).

---

## THE PROBLEM (verbatim from the client)

Find, from all available research and real-world implementations, the best possible way to measure **"AI-assisted engineering leverage" for an individual engineer**, given a fixed set of data constraints. Be exhaustive. No clarifying questions — make reasonable assumptions, state them explicitly.

### The product context (generic)
An internal, employee-facing dashboard that shows each software engineer a personal "AI Leverage" score: how effectively their AI-assisted work turns into valuable engineering output. Properties:
- **Self-reflection only** — never a performance-review input, never manager-visible, never a public ranking. Any peer comparison must be aggregate/anonymous.
- **Fair** across roles, teams, seniority, and repo environments.
- **Hard to game.**
- **Motivating and honest** — nudge genuinely better AI usage, not vanity behavior.
- **Interpretable** — an engineer should understand why their score is what it is.

### How it is measured TODAY (to be critiqued)
"Leverage" = quality-adjusted coding output per AI dollar. Concretely:
- Two per-dollar signals, each **percentile-ranked within a peer cohort of all engineers at the SAME job level** (regardless of function or team — cohort split only into full-time / contractor / intern buckets), then **averaged into a 0–100 score**:
  1. (merged PRs in window × CI-clean-rate) / AI-coding-tool spend $
  2. (lines changed, winsorized per PR at 2000 lines × CI-clean-rate) / AI-coding-tool spend $
- **CI-clean-rate** = fraction of an engineer's CI-checked PRs that passed cleanly (per-PR, at the head commit, before merge). Computed ONLY over PRs that actually ran CI; an engineer with fewer than 3 CI-checked PRs defaults to a neutral 1.0 — so a large share of people sit at exactly 1.0.
- **AI-coding-tool spend** = dollars spent on a small allowlist of AI coding-assistant CLIs (a handful of tools).
- **Window** ≈ 4 weeks.
- **Cold-start floors** (no score unless ALL pass): ≥3 merged PRs, ≥$5 coding spend, ≥3 active days, ≥14-day window.
- **Anti-gaming**: per-PR line cap (2000 lines) so one huge generated PR can't dominate.

### Data we HAVE
- Per-user, per-day, per-tool AI usage for **ALL** AI tools: dollars, tokens, session counts, active days. Spans chat assistants, AI coding-assistant CLIs, and an internal **agent-execution platform** (engineers build and run AI agents on it).
- **Merged PRs** per engineer: count and lines changed.
- A **per-PR CI pass/clean signal**.
- **Role, level, team/function** for peer cohorts.

### Hard limits (design AROUND these — do not wish them away)
- **No reliable AI-vs-human code attribution.** Cannot isolate "the AI's contribution" to any PR. (An "AI-made" tag existed once; dropped as unreliable.)
- **No task difficulty, complexity, or business value** of a PR.
- **No code-review depth**; no reliable per-author linkage to post-merge defects/incidents.
- **No per-user OUTPUT signal for agent-platform / non-PR work.** Engineers who build agents/infra/config/automation show spend but no visible output — today they score near-zero despite being heavy, effective AI engineers.
- **Data lag**: some usage tables lag ~2 weeks.
- **Privacy**: strictly self-view. Anything that only works as a cross-person leaderboard exposed to others is disqualified.

### Known problems with the current metric (validated by user feedback + analysis)
1. CI-clean-rate is a weak quality proxy: AI iterates PRs until CI passes → rate trends to ~100% and stops discriminating; also highly repo/team-dependent (some repos have 50+ CI jobs) → unfair.
2. Coding-spend denominator too narrow (few CLIs only); agent-platform work invisible; merged-PR count wrong output signal for non-PR engineers.
3. Per-dollar distribution right-skewed, poorly discriminating; top scorers often just have marginally better CI rate.
4. Cold-start floors brittle — 2 vs. 3 merged PRs flips the score on/off month to month.
5. "Output per dollar" can penalize people who legitimately spend more to do harder work.
6. Peer cohort is job-level-only across ALL functions (backend vs. ML vs. infra vs. frontend vs. platform compared head-to-head) — PR/line/spend norms vary hugely by function.
7. CI-clean-rate barely discriminates in practice (defaults to 1.0 under 3 checked PRs; AI iterates to green) — most cluster at ~1.0, little signal, still penalizes heavy-CI repos.

### The client's task list (the final doc must cover all of it)
1. **Survey the field**: DORA, SPACE, DevEx, flow/throughput, GitClear-style AI churn/rework, GitHub Copilot & enterprise AI-productivity studies, academic AI pair-programming work, "goodput"/quality-adjusted throughput, credible newer frameworks. For each: what it measures, theory, known failure modes, gameability.
2. **Read real implementations**: clone/inspect OSS repos, dashboards, metric libraries; report concretely how they compute things and what data they assume.
3. **Adversarially critique the current metric.**
4. **Reconsider framing from scratch**: not bound to "output per dollar" or PR-centric output; efficiency vs. impact vs. quality vs. adoption lenses; one score or several; fairly capture agent/infra engineers.
5. **Ranked menu of candidate approaches** — for EACH: formula/signal; data needed mapped to HAVE vs. NEW; gaming-resistance; fairness; interpretability; cold-start behavior.
6. **Commit to ONE recommended design** (or small composite): full formula; why it wins; how it handles each stated limitation; explicit red-team ("how would an engineer inflate this, and who does it unfairly penalize?"); implementation sketch using ONLY data we have; prioritized "new signals worth acquiring" with estimated lift.
7. **Be honest about dead ends** — where no good answer exists under constraints, say so plainly.

### Required deliverable structure (exact)
`leverage-measurement-findings.md`:
1. Executive summary — the recommendation in ~10 lines.
2. Landscape — framework-by-framework, failure modes + gameability.
3. Real-implementation notes — what actual repos/dashboards do, with links.
4. Critique of the current metric — point by point.
5. Candidate approaches — ranked comparison table + a paragraph each.
6. Recommended design — formula, rationale, red-team analysis, implementation sketch (data we have only).
7. New signals worth acquiring — prioritized, with expected lift.
8. Open problems — honestly flagged.

---

## STATED ASSUMPTIONS (adopt these; the writer will list them in the doc)
- Large tech org, O(500–5000) engineers, GitHub(-like) hosting, warehouse (Snowflake-like) already joins usage/PR/CI/HR data per user.
- "AI dollars" = metered per-user spend (API tokens and/or seat amortization) available uniformly across all tools including the agent platform.
- Agent platform logs per-user: spend, tokens, sessions, active days, and (assume) run counts — but NOT artifacts/outputs.
- Score window can be redefined (e.g., rolling 8–12 weeks) if justified; 4 weeks is not sacred.
- Engineering levels L3–L8-ish; functions: backend, frontend, ML, infra/platform, SRE, data. Cohort redesign is in scope.
- The dashboard can show multiple numbers/subscores; "one score" is not sacred either.

## PRIOR ART IN THIS REPO (reuse, but re-verify before citing)
A May 2026 mission answered a sibling question ("measure AI usage *quality* without seeing code"). Reuse its groundwork where valid:
- `deliverables/00_EXECUTIVE_SUMMARY.md` … `07_WORKED_EXAMPLE.md` — 3-pillar framework (Durability, Adoption Depth, Leverage/Efficiency), metrics catalog with gaming risk per metric, ethics/validity traps.
- `sources/SOURCES.md` — ~34 annotated sources (DORA/SPACE/DevEx/GitClear/Copilot studies/METR RCT etc.), tagged Primary/Vendor/Independent/Practitioner.
Key differences in THIS mission: code metadata IS available (PR counts, line counts, CI status); the question is a *leverage score* design incl. per-dollar critique; and repo-reading of real implementations is required.

## CONVENTIONS (all agents)
- Write intermediate findings to your assigned file under `/Users/jacksongeiger/ai-usage-quality-research/state/leverage/`. Return only a compact summary as your final message.
- Cite with URLs; tag each source [primary]/[vendor]/[independent]/[practitioner]. NEVER invent sources, studies, statistics, or quotes. Label your own reasoning as inference. If unverified, say "unverified".
- For every metric you touch, answer: how is it gamed? who does it unfairly penalize?
- Web tools: if WebSearch/WebFetch are not loaded, load them via ToolSearch query "select:WebSearch,WebFetch".
- Clones go under `/private/tmp/claude-501/-Users-jacksongeiger/044a2943-cb9f-4a0f-82cf-62b1372890ad/scratchpad/repos/` — never into the research repo. Use `git clone --depth 1`.
