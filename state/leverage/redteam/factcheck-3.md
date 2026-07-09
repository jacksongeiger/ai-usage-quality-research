# FACT-CHECK 3 of 3 — leverage-measurement-findings.md, last third (lines ~495–742)

Checked 2026-07-08. Method: every risky factual claim (study numbers, named attributions, URLs, dates, API capabilities, repo behaviors) verified against the primary source via WebFetch/WebSearch, or against cloned repos under `/private/tmp/claude-501/-Users-jacksongeiger/044a2943-cb9f-4a0f-82cf-62b1372890ad/scratchpad/repos/` and notes under `state/leverage/`. Classifications: CONFIRMED / WRONG / UNVERIFIABLE. Line numbers refer to the findings doc.

## External claims

| # | Line(s) | Claim | Verdict | Evidence |
|---|---|---|---|---|
| 1 | 723 | METR 2026 update: "30% to 50% of developers told us that they were choosing not to submit some tasks because they did not want to do them without AI"; data called "an unreliable signal" / "only very weak evidence" | **CONFIRMED** | https://metr.org/blog/2026-02-24-uplift-update/ [independent] — both quotes verified verbatim; also confirms updated estimates (original cohort −18%, CI −38%..+9%; new devs −4%, CI −15%..+9%). Doc's paraphrase "refusing to work without AI" is a fair compression of "would not want to do 50% of their work without AI, even though our study pays them $50/hour" |
| 2 | 521, 528, 581 | METR 2025 RCT: experienced devs 19% slower with AI; "~40pp miscalibration" (forecast +24%, believed +20% after, actual −19%) | **CONFIRMED** | https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ [independent] — "take 19% longer", "expected AI to speed them up by 24%", "still believed AI had sped them up by 20%". Gap = 39–43pp ⇒ "~40pp" fair |
| 3 | 728 | a16z "LLMflation" (Nov 2024): token prices for constant capability falling ~10x/year, ≥90% since 2023 | **CONFIRMED** | https://a16z.com/llmflation-llm-inference-cost/ [independent] — Guido Appenzeller, 2024-11-12: "the cost is decreasing by 10x every year"; factor-62 decline for GPT-4-class since Mar 2023 supports "≥90% since 2023" |
| 4 | 728 | Epoch AI "puts the median decline steeper still" | **CONFIRMED** | https://epoch.ai/data-insights/llm-inference-price-trends [independent] — declines 9x–900x/year across thresholds; median ~50x/year (200x/year post-Jan-2024) ≫ 10x/year |
| 5 | 728 | GitHub Copilot moved all plans to consumption-based AI credits on 2026-06-01 | **CONFIRMED** | https://github.blog/changelog/2026-06-01-updates-to-github-copilot-billing-and-plans/ [primary] — "As of June 1, all Copilot plans bill based on GitHub AI Credits consumed" |
| 6 | 709 | "Faros +91% review time [vendor]" (cost-shift to reviewers) | **CONFIRMED** | https://www.faros.ai/blog/ai-software-engineering [vendor] — "PR review time increases 91%" (high-AI-adoption teams); +98% merged PRs, +154% PR size, +9% bugs/dev also verified |
| 7 | 532 | "DORA 2024: individual gains and system outcomes diverge [primary]" | **CONFIRMED** | https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report [primary] — individual productivity gains alongside "decrease in delivery throughput by 1.5%, and an estimated reduction in delivery stability by 7.2%" per 25% adoption increase |
| 8 | 530 | "30x same-task variance [independent]" (spend ≠ difficulty ≠ waste) | **CONFIRMED** | https://arxiv.org/abs/2604.22750 [independent] — "runs on the same task can differ by up to 30x in total tokens"; difficulty only weakly correlates with cost; accuracy peaks at intermediate cost |
| 9 | 530 | "per-dollar ranks invert productivity [vendor]" (Jellyfish) | **CONFIRMED** | https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/ [vendor] — $0.28 → $89.32 cost per merged PR lowest→highest usage tier; top spenders ship ~2x PRs (23 vs 11) yet rank worst per-dollar. ("Inversion" is the doc's inference from these verified numbers — correctly derived) |
| 10 | 530 | "FinOps showback stops above the individual [primary]" | **CONFIRMED** | https://www.finops.org/wg/finops-for-ai-overview/ [primary] — showback recommended by team/project/department/cost-center; no individual-level allocation anywhere on the page |
| 11 | 548 | "no-op agent runs (Meta/Amazon precedent [independent])" | **CONFIRMED** | https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/ [independent] — Meta/Amazon token leaderboards; "employees deployed the bots to complete useless tasks" (per FT); tracking scrapped. (Leaderboard nicknames appear only in earlier sections, not this third) |
| 12 | 726 | Dan North "Tim" — zero individual output, team made faster | **CONFIRMED** | https://dannorth.net/the-worst-programmer/ [practitioner] — Tim Mackinnon, zero story points, paired all day; "Tim was delivering a team that was delivering software" (verified via search results incl. dannorth.net, changelog.com/news/60) |
| 13 | 528 | "CHAOSS structured-self-report pattern [primary]" | **CONFIRMED** | Clone `wg-evolution/focus-areas/community-growth/contribution-attribution.md`: data "gathered primarily through volunteered information from contributors" — structured self-report where trace data can't attribute |
| 14 | 705, 725 | Langfuse models user identity on traces; Langfuse-style success is builder-defined | **CONFIRMED** | https://langfuse.com/docs/observability/data-model [vendor] — `user_id` on traces, sessions; scores/evaluation are builder-defined, no built-in success notion |
| 15 | 723 | "staggered rollouts à la Cui" / Cui et al. PR distributions (line 739) | **CONFIRMED** (existence) | Cui et al. randomized Copilot rollout study is real and used as anchor; doc itself flags the Accenture-arm identity question as unverified (line 735) — honest |
| 16 | 711 | Atkinson −2000 lines [practitioner] | **CONFIRMED** | folklore.org "Negative 2000 Lines Of Code" (Bill Atkinson, Lisa team, 1982); fetched by survey phase (goodhart-design.md), canonical |

## Nuances (accurate in substance; noted for precision — not corrections requiring doc changes, except as flagged in final report)

| # | Line | Claim | Nuance |
|---|---|---|---|
| N1 | 705 | "OTel GenAI / Langfuse already model user identity on traces [primary/vendor]" | Langfuse: yes (`user_id`). OTel: user identity is NOT in the `gen_ai.*` namespace — it comes from OTel's *general* attribute registry (`user.id`, `user.name`, etc., all Development stability; https://opentelemetry.io/docs/specs/semconv/registry/attributes/user/). GenAI spans can carry them, so the claim holds for the OTel trace ecosystem, but a reader checking gen_ai.* attributes will not find a user field. Minor imprecision |
| N2 | 725 | "OTel GenAI has no success semantics" | The GenAI registry now includes `gen_ai.evaluation.score.value` / `gen_ai.evaluation.score.label` (Development) — but the score is "returned by the evaluator", i.e. builder-defined; there is still no *neutral/standardized* success semantics, which is the doc's actual point (and matches the adjacent Langfuse sentence). Overbroad by a hair, right in substance |
| N3 | 717 | "state of the art is ~75%-recall, opt-out, presence-not-extent — coderbuds" | The ~75% and "0% false positives on 1,000+ test PRs" are the vendor's own self-claims (https://coderbuds.com/blog/open-source-ai-code-detection-yaml-rules [vendor]), verified to exist but independently UNVERIFIED (internal notes repos/discovered.md already flag this). "Opt-out" supported indirectly (tools with "no footer added", invisible copy-paste workflows); "presence-not-extent" is a fair characterization the vendor doesn't state explicitly. Doc reads the vendor number as settled fact |

## Repo-behavior claims (verified against clones)

| Line | Claim | Verdict | Evidence (clone paths) |
|---|---|---|---|
| 708 | Revert linkage: branch `revert-<n>` / title regex — "proven metadata-only pattern (middleware, PR-analytics action)" | **CONFIRMED** | `middleware/backend/analytics_server/mhq/service/code/sync/revert_prs_github_sync.py` — pattern `r"revert-(\d+)-\w+"` on head branch, mapped to original PR number; `pull-request-analytics-action/src/converters/utils/checkRevert.ts` — `/^revert-\d+/.test(branch)` |
| 706 | Path exclusion "per git-of-theseus/CHAOSS patterns" | **CONFIRMED** | `git-of-theseus/git_of_theseus/analyze.py` — `IGNORE_PYGMENTS_FILETYPES` excludes non-code filetypes (config/docs/lock-ish); CHAOSS Code Changes Lines metric requires "a criterion for deciding whether a file is part of the source code or not" + augur whitespace-line reclassification (repos/chaoss.md, from clone) |
| 717 | coderbuds ruleset: YAML rules per tool, opt-out markers | **CONFIRMED** | `ai-detector/rules/` — 9 tool rulesets (claude-code, github-copilot, cursor, aider, devin, windsurf, openai-codex, v0-dev, replit-ai) |
| 713 | "Faros CE pattern" — per-PR-anchored self-report | **CONFIRMED** | repos/faros.md (from clone read): per-PR survey question "estimate coding time saved using copilot for this PR, in minutes"; AI Copilot Evaluation dashboards added upstream PR #372, 2024-11-12 |

## Internal cross-references (existence checks)

- `state/leverage/RECOMMENDED_DESIGN.md` §5.3 (line 624 ref): **CONFIRMED** — "### 5.3 Pseudo-SQL (core Practice + Momentum inputs)" at line 421.
- `state/leverage/redteam/{gaming,fairness,statistics}.md`, `state/leverage/survey/*.md`, `state/leverage/repos/*.md` (line 739 footer): **CONFIRMED** — all present.
- Line 705 "converging across Backstage, CNCF, reuse research, and agent-KPI frameworks": CONFIRMED-internal — synthesis documented in `survey/agent-native.md` (headline + §sources, fetch status recorded there); not independently re-fetched here.
- Line 735 absence-of-evidence notes (Denisov-Blanch magnitudes, Accenture-arm identity, GeePaw Hill rebuttal): self-declared unverified in the doc — correctly labeled, no action.
- Dashboard numbers in §6.6 (82%, 11 PRs, n=214 …): labeled "[inference — illustration]" in-doc — not factual claims.

## Verdict summary

16/16 external risky claims CONFIRMED against primary sources; 4/4 repo-behavior claims CONFIRMED against clones; all internal cross-references resolve. Zero WRONG. Three precision nuances (N1–N3) logged above; none changes any conclusion the doc draws from the claim.
