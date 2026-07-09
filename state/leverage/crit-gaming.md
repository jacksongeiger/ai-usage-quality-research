# Adversarial Critique of the Current Leverage Metric — The Rational Gamer's Playbook

Sub-investigator report, 2026-07-08. Persona: a rational engineer who wants a high "AI Leverage" score with the least real improvement in work. All external claims cited and tagged; my own reasoning labeled **[inference]**. Every attack answers (a) how it's executed and (b) who the gamer unfairly beats.

Target metric (BRIEF verbatim): score = mean of two within-level-cohort percentile ranks over a ~4-week window:
- S1 = (merged PRs × CI-clean-rate) / allowlisted-AI-CLI spend $
- S2 = (lines changed, winsorized 2000/PR, × CI-clean-rate) / allowlisted-AI-CLI spend $
- CI-clean-rate over CI-checked PRs only; <3 checked PRs → defaults to 1.0. Floors: ≥3 merged PRs, ≥$5 spend, ≥3 active days, ≥14-day window.

---

## 0. Threat model — why gaming analysis applies to a "self-view-only" score

Self-view-only is genuinely the strongest anti-gaming control available (Austin's "informational measurement"; Beck's sanctioned self-measurement use case — goodhart-design.md §1.1 [academic/practitioner]). Three reasons the red-team stands anyway:

1. **Stakes are perceived, not declared.** Amazon told employees KiroRank token stats "would not inform performance reviews"; employees didn't believe it and gamed it with fabricated tasks within months ([Fortune, 2026-07-07](https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/) [independent]). Goodhart pressure scales with perceived stakes [inference, grounded in Campbell].
2. **Most of the top exploits are *ambient*** — the default AI workflow executes them with zero intent (iterate-to-green, churn recycling, verbose generation, small-PR inflation). The score inflates without anyone "cheating," which is worse: it's undetectable and universal.
3. **The dashboard's stated purpose is to change behavior.** A broken incentive gradient nudges even honest users toward the gamed behaviors. And because the score is a within-cohort percentile (zero-sum), every gamer mechanically pushes every honest engineer's score down with no behavior change on the victim's part — corrosive and uninterpretable even in private view (goodhart-design.md §3.1).

**Structural observation [inference]:** the composite is effectively ONE lever. S1 and S2 share the denominator and the CI multiplier; only the numerator differs (counts vs capped lines, themselves correlated). Attack leverage is wildly asymmetric:
- Cut measured spend $200 → $5.01: both raw ratios ×~40.
- Split 5 PRs into 15: S1 ×3 (S2 roughly unchanged or better via the cap).
- Lift CI-clean 0.90 → 1.00: ×1.11.
A rational gamer attacks the denominator first — and the denominator is the least-defended input (allowlist boundary, personal accounts, $5 floor). Percentiling compresses magnitudes but a 40x raw shift tops any cohort.

---

## 1. Exploit catalog

Ratings: Effort / Detectability (using ONLY the HAVE data: all-tool usage, PR counts+lines, per-PR CI flag, role/level/function) / expected Prevalence.

### 1.1 Denominator attacks (allowlisted AI-CLI spend)

**D1. Tool-substitution ("spend leakage").** Do the AI-assisted work in non-allowlisted tools — chat assistants, IDE completions, and notably the org's OWN agent-execution platform (its spend is outside the CLI allowlist per the BRIEF) — then paste/apply the results. Output identical, measured spend → $5 floor.
- Effort: none-to-trivial (many already prefer chat). Detectability: Medium — the org HAS all-tool usage, so a CLI→chat/agent mix-shift is visible, but it is indistinguishable from legitimate tool preference and violates no policy. Prevalence: High.
- Beats: allowlisted-CLI power users; the demonstrably most productive heavy spenders — Jellyfish (12k devs): top-quintile spenders produce ~2x merged PRs at ~$89.32/PR vs $0.28/PR at the bottom, so the per-dollar rank already inverts productivity before any gaming ([Jellyfish, 2026-04-15](https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/) [vendor]).
- Evidence: roi-cost.md §4.1(a); copilot-viewer.md "single-vendor spend universe" anti-pattern [vendor]; no vendor framework meters a per-person all-tool denominator (vendor-frameworks.md).

**D2. Non-adoption / minimum-viable spend.** Hand-write code; spend $5.01 for the floor. Extremal Goodhart: the ratio is maximized at the spend singularity, not at great AI use (Manheim & Garrabrant taxonomy, https://arxiv.org/abs/1803.04585 [primary]).
- Effort: negative (abstention). Detectability: visible but unactionable — "efficient" and "non-adopter" are the same number. Prevalence: High among AI-skeptics, who get *rewarded* for skepticism by an AI-leverage score.
- Beats: genuine adopters and learners mid-exploration; directly fights adoption mandates (Shopify-style baselines — roi-cost.md §3.2).

**D3. Personal/free accounts.** Route heavy sessions through personal subscriptions or free tiers — invisible to the meter entirely.
- Effort: Low. Detectability: near-zero with HAVE data. Prevalence: Medium (policy/compliance risk deters some). Beats: compliant engineers. [inference; mechanism noted in roi-cost.md §4.1]

**D4. Spend-window shaping.** Batch expensive agentic exploration into one "sacrifice window" (or right after a scoring cutoff); keep other windows lean. The ~2-week usage-table lag blurs window edges further.
- Effort: Low-Med. Detectability: Low. Prevalence: Medium. Beats: steady spenders. [inference]

**D5. Cheap-model / context-starvation theater.** Always pick the cheapest model, minimize context. Quality damage is invisible: no defect/incident linkage exists (hard limit #3), and accuracy peaks at *intermediate* cost while identical tasks vary 30x in tokens — spend is noise, not diligence ([arXiv 2604.22750](https://arxiv.org/abs/2604.22750) [independent]).
- Effort: trivial. Detectability: near-zero. Prevalence: Med-High once engineers notice the denominator. Beats: engineers using expensive models for correctness-critical code.

### 1.2 Numerator attacks — merged PR count

**N1. PR splitting / stacked micro-PRs.** Split any change into k small PRs. This **quadruple-dips**: (i) PR count ×k; (ii) each stays under the 2000-line cap so total counted lines can rise too; (iii) small PRs pass CI more easily, lifting CI-clean-rate; (iv) it clears the ≥3-PR cold-start floor. Stacked-diff tooling automates it.
- Effort: Low. Detectability: Low-Med — and here's the trap: small batches are *literally DORA best practice* (dora.md finding #4), so moderate-intensity gaming is indistinguishable from virtue. Prevalence: High.
- Beats: big-feature/ML engineers with long-lived branches; and it cost-shifts to reviewers — Faros telemetry (10k+ devs): AI users +98% merged PRs but review time +91% and NO company-level gain ([Faros](https://www.faros.ai/blog/ai-software-engineering) [vendor]); goodhart-design.md §4 table.

**N2. Trivia farming.** AI-generated typo/docs/config/version-bump/format PRs; each merged PR counts 1.0 regardless of value because no difficulty/value signal exists (hard limit #2).
- Effort: near-zero. Detectability: Low (a trickle of small PRs looks like healthy hygiene). Prevalence: Med-High. Beats: engineers on hard, slow problems. Precedent: commit fabrication is commodity tooling (`fake-git-history`, `gitfiti` — goodhart-design.md §1.2 [primary artifacts]); PR-count analogues are easy.

**N3. Codemod / automation fan-out.** One AI-generated codemod applied across 30–50 repos as separate personally-authored PRs: near-fixed spend, massive PR+line harvest. NO reference implementation we read filters bots or automation from per-author PR counts (fourkeys: zero bot handling, grep-verified; Faros CE: `vcs_User.type` unused; middleware: bot-authored PRs still counted — repos/*.md [primary code]).
- Effort: Medium once, amortized forever. Detectability: Medium (burst pattern in counts) but unsanctionable — codemods are legitimate work; only diff content distinguishes value, which we cannot see. Prevalence: Medium (concentrated in infra/platform folks who know how). Beats: single-repo deep workers.

**N4. Revert/rework double-dip.** Ship broken → revert PR → fix PR = 3 merged PRs and ~3× lines from one unit of failed work. HAVE data has no revert detection (middleware detects `revert-<pr#>` branches; we don't ingest branch names). Entelligence: revert rate grew 3.7x vs 2.6x PR-volume growth in AI-heavy orgs ([research.entelligence.ai](https://research.entelligence.ai/) [vendor, COI]).
- Effort: none — the metric literally *pays extra for failure*. Detectability: none in HAVE. Prevalence: High as ambient inflation, Low as deliberate strategy. Beats: engineers who ship once, correctly (fewer PRs, fewer lines).

### 1.3 Numerator attacks — lines changed (winsorized 2000/PR)

**L1. Verbose-code / junk-line farming.** Generated boilerplate, redundant tests, lockfiles, vendored deps, generated files, long-winded AI style. HAVE gives only line *counts* — no path or content filters possible. GitClear: <50% of raw changed lines are "meaningful"; copy/paste share 8.3%→15.7% (2022→H1 2026) ([GitClear 2025 PDF](https://gitclear-public.s3.us-west-2.amazonaws.com/GitClear-AI-Copilot-Code-Quality-2025.pdf); [2026 Maintainability Gap](https://www.gitclear.com/the_ai_code_quality_maintainability_gap) [vendor]). AI makes LOC free.
- Effort: zero (it's the default output style). Detectability: near-zero. Prevalence: High (ambient). Beats: simplifiers and deleters (Atkinson's −2000 lines, folklore.org [practitioner]); terse-language engineers; seniors who solve with less code (Dan North [practitioner]).

**L2. Cap-aware chunking.** A 6,000-line change as 3×2,000-line PRs: all 6,000 lines count (vs 2,000 in one PR) *plus* 3 PR counts. A fixed, known cap is a published gaming boundary (goodhart-design.md §3.3).
- Effort: Low. Detectability: Low-Med (pile-up of PRs just under 2,000 lines is a detectable signature — the one honest telltale this metric emits) [inference]. Prevalence: Medium. Beats: functions whose legitimate unit-of-work is large (migrations, codegen, data eng).

**L3. Churn recycling.** Merge, then rewrite your own code next week — every pass counts as fresh lines. Heavy AI users show churn up to 9x within-subject (GitKraken×GitClear, 2,172 dev-weeks, Mar 2026 [vendor]); the metric pays 2–3x for doing work badly repeatedly.
- Effort: zero (documented default AI failure mode). Detectability: none in HAVE (needs per-PR file lists — top acquisition candidate per gitclear.md). Prevalence: High (ambient). Beats: right-first-time engineers, who change fewer total lines by construction.

**L4. Move/reformat sweeps.** Renames, file moves, repo-wide formatting: lines count on both sides, zero semantic content; also transfers blame/ownership wholesale (hercules.md, code-maat.md [primary code]).
- Effort: trivial. Detectability: Low (a 1,999-line "cleanup" PR looks fine as counts). Prevalence: Medium. Beats: everyone whose real lines compete with fake lines in the same percentile pool.

### 1.4 CI-clean-rate attacks (the quality multiplier)

**C1. The default-1.0 exploit.** Keep CI-*checked* PRs ≤2 → automatic 1.0, which is the *maximum possible value* of the multiplier. Meanwhile PRs that never run CI (docs-only paths, no-CI repos, skip-CI labels) still count fully in both numerators. Optimal play: many non-CI merged PRs + <3 CI-checked → perfect quality score on unlimited output. Defaulting missing data to the best score is a design gift; compare DevLake's explicit "N/A — missing signal" pattern (devlake.md #4 [primary code]).
- Effort: Low-Med (repo/path selection). Detectability: Medium (share of CI-checked PRs is computable from HAVE — a second honest telltale). Prevalence: Medium deliberate; High accidental (brief says "a large share of people sit at exactly 1.0"). Beats: engineers with 20 checked PRs and one flaky failure (0.95 < defaulted 1.0).

**C2. Iterate-to-green.** Let the AI loop until CI passes locally or in draft, push only green heads. The rate saturates at ~1.0 population-wide — already conceded in the brief (#1, #7). Best-RCT evidence: build *volume* +38% while success *rate* was flat-to-negative (Accenture arm −17.4%, significant) — volume looped, quality didn't rise (Cui et al., Mgmt Sci 2026, https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 [independent]). And 25–45% of AI code carries security flaws that pass CI (independent-evidence.md #4).
- Effort: zero — this IS the modern workflow. Detectability: n/a (nothing to detect; the signal is dead). Prevalence: ~universal, no intent required. Beats: fast-pushing trunk-based devs and risk-takers on hard changes who use CI as the iteration loop it was built to be; heavy-CI (50+ jobs) and flaky-test repos.

**C3. CI-environment arbitrage.** Prefer repos/paths with light, reliable CI; avoid the 50-job legacy monolith. Classification/venue-shopping is exactly how DORA keys get gamed (Harvey: hide work to shrink lead time — dora.md #5 [primary-interview]).
- Effort: Medium (requires latitude in work selection). Detectability: Low. Prevalence: Medium. Beats: engineers locked into legacy heavy-CI systems (brief problem #1).

### 1.5 Window, floor, and cohort attacks

**W1. Score suppression.** Heading for a bad month? Deliberately fail one floor (hold at 2 merged PRs, spend <$5, <3 active days) → "no score" instead of a low score. The cliff makes absence strictly better than honesty. Same hide-at-low-n logic the EB-shrinkage literature warns about (goodhart-design.md §4).
- Effort: trivial. Detectability: Low (floors legitimately exclude many people). Prevalence: Medium. Beats: steady contributors who always post a score, including bad months.

**W2. Merge-timing / seasonal arbitrage.** Percentiles are zero-sum within the window: hold ready PRs and land them when the cohort is quiet (holidays, release freezes) → rank spikes with zero behavior change. Non-stationary cohort distributions during an adoption wave amplify this (goodhart-design.md §3.1). [inference]
- Effort: Low. Detectability: Low. Prevalence: Medium. Beats: engineers who merge when work is ready; anyone on vacation-heavy teams' *off*-cycle.

**W3. Effort reallocation (the Austin attack).** Shift time from everything unmeasured — code review, mentoring, design docs, incident response, and *building agents on the internal platform* — into PR-shaped output. Austin 1996: under partial measurement, effort flows from unmeasured to measured dimensions until the real goal fails (goodhart-design.md §1.1 [academic]). This isn't cheating; it's obeying the dashboard.
- Effort: zero (it's prioritization). Detectability: very Low. Prevalence: High and unconscious. Beats: the org itself, reviewers, mentors, the Dan-North "Tims," and — with maximum irony — the agent-platform engineers the brief already flags as scoring near-zero: the score tells the org's most AI-native builders to stop building agents and go farm PRs.

**S1. Cohort free-riding / work-shape mimicry.** Level-only cohorts pool backend, ML, infra, frontend (brief problem #6). Individuals can't switch cohorts, but they can adopt the winning function's work *shape* — many small config/UI-style PRs with minimal CLI spend — regardless of what their role actually needs. [inference]
- Beats: minority functions in the pooled cohort (ML/data: expensive loops, few PRs; infra: non-PR output). Also contractors/interns are bucketed separately but functions are not — the metric is precise about employment status and blind to work type.

**S2. Noise-surfing.** Because most sit at CI≈1.0 and the per-dollar ratio is right-skewed and 4-week PR counts are statistically noise even at n≈5,000 (Cui pooled SE — copilot-studies.md #3), top ranks are decided by denominator luck + CI noise (regressional Goodhart; brief problem #3). The gamer doesn't have to beat the best engineer — only the noise floor. Any single lucky-window screenshot becomes a self-narrative ("I'm top-decile") that the next window can't falsify because the gamer can re-run W1/W2. [inference]

---

## 2. The combined play ("clean sweep") and its cost

A rational gamer composes: D1+D2 (route AI use through chat/agent platform, $5.01 CLI spend) + N1/N2 (stacked micro-PRs, trivia trickle) + C1 (favor non-CI paths, keep checked PRs ≤2 → default 1.0) + W2 (land merges in cohort lulls). Every component is Low/zero effort, individually plausible, and policy-compliant. Expected result: top-of-cohort score with *negative* org value (reviewer load up, churn up, adoption down). Nothing in the HAVE data distinguishes this engineer from the org's best — except the two honest telltales: PR-size pile-up just under 2,000 lines, and a low share of CI-checked PRs. [inference; every component grounded above]

The mirror case already ran publicly: Meta's "Claudeonomics" and Amazon's "KiroRank" spend-maximization leaderboards were gamed within months via idle agents and fabricated tasks, then scrapped ([Fortune](https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/) [independent]; [The Decoder](https://the-decoder.com/meta-employees-compete-for-token-consumption-on-an-internal-ai-leaderboard/) [independent, secondhand]). Goodhart pressure is symmetric; the client's spend-*minimizing* rank is the untested mirror of a design that just failed at scale (roi-cost.md §1.2).

---

## 3. Ambient gaming — the score inflates with no one cheating

Even with zero deliberate gamers, the metric's inputs drift in the gamer's direction as AI workflows mature:
- CI-clean-rate → 1.0 for everyone (iterate-to-green is the product design of coding agents; brief #7; Cui [independent]).
- Lines and PR counts inflate with AI defaults: PR size +154%, merged PRs +98% (Faros [vendor]); copy/paste 15.7% and 2-week churn rising (GitClear [vendor]); merge/throughput signals get noisier as review becomes "a rubber-stamping machine" (Houck, via space-devex.md #5 [practitioner]).
- Meanwhile system outcomes can move the *other way*: DORA 2024 — per +25% AI adoption, −1.5% delivery throughput, −7.2% stability ([DORA 2024](https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report) [primary]).

Consequence [inference]: month-over-month score gains will read as "my AI leverage improved" while measuring workflow drift. The metric cannot distinguish a better engineer from a later calendar date. That breaks the one causal design the data could support (self-vs-self trend), because the baseline itself inflates.

---

## 4. Top 5 exploits ranked by expected prevalence

| # | Exploit | Why it wins on prevalence | Effort | Detectability (HAVE data) | Primary victims |
|---|---------|---------------------------|--------|---------------------------|-----------------|
| 1 | **C2 Iterate-to-green CI saturation** | Zero intent needed — it IS the AI workflow; brief concedes population already at ~1.0; Cui shows volume-up/rate-flat [independent] | None | N/A — signal is dead, nothing to catch | Heavy-CI/flaky-repo engineers; risk-takers; honest CI-as-iteration users |
| 2 | **D1/D2 Spend minimization** (tool-substitution incl. agent-platform routing; thrift; personal accounts) | Strongest mathematical lever (~40x); zero effort; plausibly deniable; documented mirror failure at Meta/Amazon [independent] | None–Low | Medium (mix-shift visible, unactionable) | Heavy legitimate adopters (2x-productive per Jellyfish [vendor]); learners; compliant meterees |
| 3 | **N1 PR splitting / micro-PR inflation** | Cloaked as DORA best practice; tool-automated; quadruple-dips (count, cap, CI, floor) | Low | Low-Med (size pile-up under cap is the telltale) | Big-feature/ML engineers; reviewers (cost-shift: +91% review time, Faros [vendor]) |
| 4 | **W3 Effort reallocation to PR-shaped work** | Not perceived as gaming; the dashboard nudges it by design; Austin's law says it's inevitable under partial measurement [academic] | None | Very Low | Mentors, reviewers, incident responders, agent-platform builders — the org's highest-leverage people |
| 5 | **L1/L3 Line farming + churn recycling** | AI's default output is verbose and churny — the exploit executes itself; GitClear churn up to 9x for heavy users [vendor] | None | Near-zero (counts only, no content/file data) | Simplifiers/deleters, right-first-time seniors, terse-language stacks |

Deliberate-only ranking would promote C1 (default-1.0 farming) and W1 (score suppression) into the top 5; they are bounded here because they require noticing the rules. [inference]

---

## 5. The fairness inversion, summarized

Who the gamed metric systematically beats — independent of any individual gamer, compounded by every one of them:

1. **Agent-platform / non-PR engineers** — spend, no PR-shaped output; structurally near-zero (brief-validated). W3 then tells them to abandon that work.
2. **Heavy legitimate adopters** — per-dollar rank inverts productivity (Jellyfish 300x cost/PR spread; top spenders 2x output [vendor]).
3. **Engineers on hard/exploratory work** — spend is a noisy function of task+context, not skill (30x same-task variance [independent]); harder work → more spend → lower score (brief problem #5 confirmed empirically).
4. **Heavy-CI and flaky-repo residents** — every checked PR is a coin-flip against people whose repos default them to 1.0.
5. **Simplifiers, deleters, refactorers** — negative or small line deltas; the metric's numerator is anti-correlated with their value (Atkinson; North [practitioner]).
6. **Reviewers, mentors, incident responders** — unmeasured work absorbs the gamers' cost-shift (review time +91% [vendor]).
7. **Part-timers, on-call rotations, vacation-takers** — 4-week window + hard floors turn low-n months into no-score or noise-score; percentile zero-sum means others' gaming lowers their number with no behavior change.
8. **ML/data functions in the pooled level-cohort** — expensive iteration loops and few, large PRs against frontend/config PR cadences (brief problem #6).

---

## 6. What survives the red-team

Honest inventory — three things about the current design that resist gaming:
1. **Self-view-only privacy** — the single strongest control in the literature (Austin informational mode; Beck; goodhart-design.md §4 cross-cutting). Keep it absolute; every attack above gets an order of magnitude worse if the score leaks to managers.
2. **The cohort assignment itself** — an engineer cannot choose their level cohort. (The cohort *definition* is wrong, but it isn't individually manipulable.)
3. **Two emitted telltales** — PR-size pile-up just under the 2,000-line cap, and a falling share of CI-checked PRs. These are the only detection signatures the metric produces about its own gaming; any successor design should keep monitoring both as integrity canaries. [inference]

Everything else — both numerators, the denominator, the multiplier, the floors, the window, the averaging of percentiles — is attackable at Low-or-zero effort with Low-or-zero detectability, and the highest-prevalence attacks require no intent at all.

---

## 7. Sources cited in this file (all previously verified in survey/repos files; re-listed for traceability)

- Fortune, "Cognition CEO: tokenmaxxing…" 2026-07-07 [independent] — https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/ (Amazon/Meta leaderboards gamed & scrapped; disbelief in "no consequences")
- The Decoder (via The Information), Meta "Claudeonomics" [independent, secondhand] — https://the-decoder.com/meta-employees-compete-for-token-consumption-on-an-internal-ai-leaderboard/
- Jellyfish, tokenmaxxing cost data, 2026-04-15 [vendor] — https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/ ($0.28→$89.32/PR; heavy users 2x PRs on 10x tokens)
- Bai/Mihalcea/Brynjolfsson et al., arXiv 2604.22750 [independent] — https://arxiv.org/abs/2604.22750 (30x same-task token variance; accuracy peaks at intermediate cost)
- Cui et al., Mgmt Sci 2026, 3 RCTs n=4,867 [independent] — https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566 (build volume +38%, success rate flat-to-negative; 4-wk PR counts are noise)
- Faros AI productivity paradox [vendor] — https://www.faros.ai/blog/ai-software-engineering (+98% PRs, +154% PR size, +91% review time, no org-level gain)
- GitClear 2025 report PDF + 2026 Maintainability Gap [vendor] — https://gitclear-public.s3.us-west-2.amazonaws.com/GitClear-AI-Copilot-Code-Quality-2025.pdf ; https://www.gitclear.com/the_ai_code_quality_maintainability_gap (<50% lines meaningful; copy/paste 15.7%; churn 9x heavy users per GitKraken×GitClear Mar-2026)
- Entelligence Research [vendor, strong COI] — https://research.entelligence.ai/ (revert rate 3.7x vs 2.6x PR growth; directional only)
- DORA 2024 announcement [primary] — https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report (−1.5% throughput, −7.2% stability per +25% adoption)
- Manheim & Garrabrant, Goodhart taxonomy [primary] — https://arxiv.org/abs/1803.04585
- Austin 1996; Beck & Orosz 2023; Dan North; folklore.org "-2000 lines"; fake-git-history/gitfiti — full citations in survey/goodhart-design.md §6
- Harvey/DORA lead-time gaming interview [primary-interview] — https://getdx.com/podcast/masterclass-on-dora-metrics/
- Repo evidence (bot-filtering absent; default-vs-N/A patterns): state/leverage/repos/{fourkeys,faros,middleware,devlake,hercules,code-maat}.md [primary code, read in Phase A]

**Unverified/absent evidence, stated plainly:** no documented case of any org percentile-ranking individuals on output-per-AI-dollar exists (roi-cost.md §1.2) — the client's design has zero deployed precedent to learn from; this critique is therefore grounded in its components' individual track records plus the symmetric spend-leaderboard failures.
