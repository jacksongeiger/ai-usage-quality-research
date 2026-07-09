# GitClear AI Code Quality Research — churn, copy/paste, survival, Diff Delta (+ Code Time / per-engineer alternatives)

Sub-investigator report. Date: 2026-07-08. All external claims cited; my own reasoning labeled *(inference)*.

---

## 1. What it measures & theory

### 1.1 The operation taxonomy (verbatim from the 2025 PDF, Appendix A0)

GitClear classifies every changed line in a git repo into seven **code operations** [primary-within-vendor PDF, v2025.2.5]:

1. **Added code** — "Newly committed lines of code that are distinct, excluding lines that incrementally change an existing line (labeled 'Updates'). 'Added code' also does not include lines that are added, removed, and then re-added (these lines are labeled as 'Updated' and 'Churned')."
2. **Deleted code** — "Lines of code that are removed, committed, and not subsequently re-added for at least the next two weeks."
3. **Moved code** — "A line of code that is cut and pasted to a new file, or a new function within the same file. By definition, the content of a 'Moved' operation doesn't change within a commit, except for (potentially) the white space that precedes the content."
4. **Updated code** — "A committed line of code based off an existing line of code, that modifies the existing line of code by approximately three words or less."
5. **Find/Replaced code** — "A pattern of code change where the same string is removed from 3+ locations and substituted with consistent replacement content."
6. **Copy/Pasted code** — "Identical line contents, excluding programming language keywords (e.g., `end`, `})`, `[`), that are committed to multiple files or functions **within a commit**."
7. **No-op code** — whitespace/line-number-only changes; excluded from the research.

**Churn** is not an operation but an overlay: "For a line to qualify as 'churned,' it must have been authored, pushed to the git repo, and then **reverted or substantially revised within the subsequent two weeks**. Churn is best understood as 'changes that were either incomplete or erroneous when the author initially wrote, committed, and pushed them.'" (2025 PDF, A0.)

### 1.2 The theory

- **Moved code = the signature of refactoring/reuse.** Declining moved % ⇒ declining consolidation into shared modules.
- **Copy/paste = duplication debt.** Anchored in the (real, independent) code-clone literature: clones that don't co-evolve breed bugs — e.g., "57.1% of all co-changed clones are involved in bugs" (Mo, Zhang et al. 2023, cited in the 2025 PDF); Mondal 2019; Wagner 2016; Kamiya CCFinder 2002.
- **Churn = incomplete/erroneous work** — a proxy for "wasted motion" / quality. Academic anchor for churn→defects: Nagappan & Ball (ICSE 2005) showed **relative** code churn measures are highly predictive of defect density (89% discrimination accuracy on Windows Server 2003) while **absolute churn is a poor predictor** [independent].
- **Code provenance / age of revised code**: what fraction of edits touch code <2 weeks, <1 month, >1 year old. "Long-term update percent" (share of changes touching code >12 months old) is their maintenance-health signal (2026 report).

### 1.3 Diff Delta — the per-engineer durability-weighted output metric

Verbatim from GitClear's "mathematical proof" page [vendor]:

> Δ(ℓ) = φ(ℓ) · ⊖(ℓ) · ⧉(ℓ) · β(o) · τ(a) · σ(x)  and  E(d, T) = Σ Δ(ℓ) over all lines authored by developer d in interval T

- φ file/branch filter (drops auto-generated files, unmerged/release branches, compiled files); ⊖ context filter (negates keywords, whitespace, ad-hoc comments, delimiters); ⧉ duplication filter (conserves credit across forks/rebases/cherry-picks); β base score **by operation: Delete 25 pts max, Update 20, Add 10, Find/Replace 3, Move/Copy 0**; τ time scalar — "code that isn't churned → higher durability premium"; σ context scalar (language weight, proximity, greenfield adjustment).
- It **is** computed per-developer (E(d,T)) — this is the closest existing thing to a "quality-adjusted output" score, and it explicitly rewards durability (low churn) and penalizes duplication. *(inference: the Delete>Update>Add weighting is GitClear's answer to LOC-gaming — deleting code can't be inflated by generation.)*

### 1.4 The reports and headline numbers

**2024 report ("Coding on Copilot", Jan 2024; data 2020–2023, ~153M changed lines)** [vendor]: churn "projected to double in 2024 vs its 2021, pre-AI baseline"; added+copy/pasted rising relative to updated/deleted/moved.

**2025 report (Feb 2025, v2025.2.5; 211M meaningful changed lines 2020–2024, ~10,000 repos)** [vendor; PDF appendix has raw tables]:
- Operations by year (% of changed lines): Moved 24.8% (2021) → **9.5% (2024)**; Copy/pasted 8.3–8.4% (2020–21) → **12.3% (2024)**; Added 39% → 46%; all-line churn 3.3% (2021) → **5.7% (2024)** (2025 projected 6.9%).
- New-code churn (A7, 45M new lines): % of newly added lines revised within 2 weeks: **4.78% (2020) → 7.04% (2024)**; within a month: 5.49% → 7.91%.
- Age of revised code: share of revised lines <1 month old: 70.0% (2020) → **79.2% (2024)**; >1-year-old code getting touched collapsed.
- 2024 = first year copy/pasted lines **exceeded** moved lines; ~8x rise in frequency of 5+-line duplicated blocks during 2024.
- Dataset split: "about two-thirds private corporations that have opted in to anonymized data sharing, and one-third open source projects (mostly those run by Google, Facebook, and Microsoft)" — OSS list includes Chromium, React, VS Code, Kubernetes, Postgres, CPython, etc. Fewer than half of raw changed lines survive their exclusion filters as "meaningful."

**2026 report ("The Maintainability Gap", ~mid-2026; 623M analyzed changes 2023–2026, with GitKraken)** [vendor]:
- Block duplication (5+ consecutive repeated meaningful lines): 40.3 per M changed lines (2023) → **73.0 YTD 2026 (+81%)**; copy/paste 9.4% (2022) → **15.7% (H1 2026)**; moved code down to **3.8%**; "long-term update percent" 1.7% (2023) → **0.46% (−74%)**; cross-file function calls −35%; "error-masking constructs" +47%; 2-week churn +15% vs 2023. Claims AI-assisted commits are now ~25% of all commits (attribution mechanism **undisclosed**; unverified — *inference: likely GitKraken client telemetry*).
- Framing (their words, via the report page): "The headline is not 'AI writes bad code.' It is that today's default AI workflow is incentivized to deliver atomic code — a happy-path, a passing test, a closed ticket — while quietly taxing the invisible and the deferred."

**GitKraken × GitClear "AI Multiplier" research (Mar 2026)** [vendor] — the only *per-engineer, within-subject* application: 2,172 developer-weeks across Copilot/Cursor/Claude Code, comparing "developers against their own prior performance": **+25% output (Diff Delta)**, heavy AI users 4–14x more activity, test code 4x, **"code churn increases up to 9x."** Also: "AI amplifies existing skill levels… rather than equalizing them."

---

## 2. Evidence quality

- **Vendor research end to end.** GitClear sells the product that computes these metrics; every report ends with a sales appendix (2025 PDF, A3). Magnitudes should be treated as vendor-reported; directions have independent corroboration.
- **Dataset is not public and not longitudinally stable.** Self-admitted in the 2025 PDF: "This data evolves by percentage points from year-to-year, **as customers evolve**, so our line counts this year were not identical to the numbers that our database produced at the beginning of 2024." I.e., the panel composition drifts — a serious confound for year-over-year trend claims. No per-language/per-repo-size breakdowns, no confidence intervals, no significance tests.
- **No causal identification in the 2024/2025 reports.** They measure secular time trends over the AI-adoption era, not AI-vs-non-AI code (no attribution). Alternative explanations (junior influx, layoffs/reorgs, changing repo mix, post-COVID practices) are mostly unaddressed; the 2025 PDF does run one robustness check (new-code churn, to separate "changing team focus" from "changing author tendencies").
- **Backfill bias, acknowledged**: duplicate-block backfill for 2020–2023 sampled the 1,000 largest commits per repo, which they argue biases *against* their conclusion (2025 PDF, A8) — a point in their favor.
- **Prediction track record**: directionally good, magnitudes off. 2024 projected churn 7.1% vs actual 5.7%; projected moved 13.4% vs actual 9.5% ("we got the general direction right, but underestimated the magnitude"). They publicly predicted the DORA 2024 stability decline before it published; DORA 2024 then estimated ~7.2% decrease in delivery stability per 25% increase in AI adoption [independent corroboration, cited in the PDF].
- **Independent critique is thin but pointed.** The HN threads on the 2024 report contain the sharpest published objections [practitioner]:
  - datadrivenangel: "If the cost of updating code has dropped due to AI, is code churn still a good proxy for low code quality?"
  - nzach: "It implies that… churn is a bad thing. And I don't think [that is] true."
  - naasking: population-mix confound — "maybe it's partly because more people are getting involved in programming."
  No formal academic replication or rebuttal of GitClear's numbers found (searched multiple formulations) — **unverified whether any exists**.
- **Definitions are genuinely published** (A0 appendix, raw yearly tables, and even the ActiveRecord queries used) — unusually transparent for vendor research, even though the underlying classifier is proprietary.

---

## 3. Known failure modes (of the metrics themselves)

1. **Churn conflates bad and good revision.** GitClear itself concedes: "Performed in moderation, revising recent code is often a *good* thing. It suggests a developer is conscientious about submitting code that has been tested & polished." Churn bundles: review-driven improvements, spec/product changes, healthy fast-follow polish, deliberate iterative delivery, and genuine defects — with no way to tell them apart from git data.
2. **Workflow dependence / measurement inconsistency.** Churn counts lines *pushed then revised*. Squash-merge repos hide intra-PR churn; trunk-based commit-often teams and auto-committing AI agents expose it. Two engineers with identical behavior in different repos get different churn. *(inference, mechanically certain from the definition.)*
3. **The 2-week window is arbitrary** and misses slow rework; Pluralsight Flow uses 3 weeks, others 30 days — benchmark numbers are not comparable across tools.
4. **Moved-code detection requires near-exact content match** (unchanged content modulo leading whitespace). Real refactors that move-and-edit are classified as Delete+Add — understating refactoring, especially in the AI era where "move" happens via regenerate. *(inference from definition.)*
5. **Copy/paste is within-commit only** — duplication spread across commits is invisible; conversely test fixtures, migrations, table-driven tests, and verbose-by-convention languages (Go error handling) legitimately duplicate lines.
6. **Absolute churn is a poor defect predictor** (Nagappan & Ball 2005) — only *relative/normalized* churn measures predict well. Any raw per-engineer churn number is noise until normalized to code size and cohort.
7. **Survival/half-life metrics have years-long feedback loops.** Erik Bernhardsson's git-of-theseus analysis: aggregate code half-life ≈ 3.33 years (Git repo itself ~6 years) via Kaplan–Meier on line survival. Useless for a 4–12-week score window; and survival is dominated by *which subsystem you touch*, not personal quality.
8. **High survival can be pathology, not quality.** GitClear's own 2026 finding — long-term update % collapsing means old code is "frozen strata" nobody maintains. A survival-maximizing engineer is indistinguishable from an engineer whose code nobody dares touch.

---

## 4. Gameability & who it penalizes (per metric)

**Churn % (lower = better):**
- *Gamed by:* pushing only "finished" work (batch locally, hide WIP — defeats CI early-signal); waiting until day 15 to fix known issues; fixing via new adjacent lines/files instead of editing recent lines; doing risky exploration on branches that never merge (excluded by φ); lobbying for squash-merge.
- *Penalizes:* honest iterators and prototypers; engineers on volatile product surfaces; people who diligently apply review feedback post-merge; trunk-based/small-commit teams; anyone downstream of shifting requirements. Also charges the *original author* when a teammate legitimately revises their code (attribution choice varies by tool — Flow splits "rework" (own recent code) vs "help others" (someone else's)).

**Copy/paste % & block duplication (lower = better):**
- *Gamed by:* trivial renames/reordering to break Type-1 exact-line matching; splitting the duplication across multiple commits (detector is within-commit); moving boilerplate into generated files (excluded by φ).
- *Penalizes:* test authors, data/migration/config engineers, teams in verbose languages or codegen-adjacent stacks.

**Moved % ("refactoring share", higher = better):**
- *Gamed by:* performative file reorganization — cut/paste shuffles are cheap and match the detector exactly.
- *Penalizes:* greenfield engineers (nothing to move); anyone whose refactors involve editing (mis-classified, see §3.4).

**Diff Delta (higher = better):**
- *Gamed by:* farming Delete points (25 pts/line) on dead code; making plausible small "Updates" (20 pts) across old files; avoiding zero-point moves. τ (durability) and churn-relabeling partially defend delete/re-add cycles, but the weights are proprietary → an engineer cannot fully audit their own score. Opaqueness itself is the interpretability failure.
- *Penalizes:* maintainers of stable systems (little to change); non-PR/agent-platform engineers (invisible, same as any git metric); work in unweighted-favorably languages/configs (σ language weight undisclosed).

**Survival / half-life (higher = better):**
- *Gamed by:* writing code nobody touches (ownership gatekeeping, discouraging refactors of "your" files); verbose stable boilerplate; working only in slow-moving subsystems.
- *Penalizes:* engineers in fast-evolving areas; prototype/experiment teams; anyone whose correct code is deleted for business reasons.

**Code Time (active editor minutes):**
- *Gamed by:* keeping the editor warm (typing/no-op edits — the 10-minute idle rule is trivially defeated).
- *Penalizes:* engineers who think/design/review/pair/operate rather than type — exactly the seniors and agent-orchestrators.

---

## 5. Applicability to our constraints (HAVE / CANNOT mapping)

**Can we compute any of this from the data we HAVE (PR count, lines changed per PR, per-PR CI signal, AI usage/spend, HR cohorts)? No.** Every GitClear-style metric needs, at minimum, line-level diff history:
- Churn/survival require tracking *line identity across commits* (blame-like traversal) — needs full git history + diff content.
- Moved/copy-paste require *line content matching* (even if non-semantic).
- `git log --numstat` (true metadata, no contents) supports only a **file-level approximation**: per-author re-touch rate — same author modifying the same file again within N days across separate PRs (code-maat-style, à la Tornhill). Crude and noisy, but the only churn-flavored signal that stays strictly content-free. *(inference)* — and it still needs per-PR **file lists**, which our HAVE list does not include (PR count + lines only). File lists are cheap to acquire from the GitHub API → belongs in "new signals worth acquiring," near the top.

**As a NEW signal (recommended framing):** run a repo-side analyzer (GitClear itself, Appfire Flow, or an OSS pipeline: git-of-theseus / Hercules / code-maat) that exports **only aggregate per-engineer metadata** (churn %, dup %, re-touch rate) into the warehouse, never code. This respects the privacy/self-view constraint and the no-code-in-warehouse posture; commercial precedent proves per-engineer computation works at scale (Flow's per-engineer rework <3 weeks; GitClear's E(d,T); GitKraken's 2,172 developer-week within-subject study).

**Why it matters for our metric despite the cost:** GitClear's population-level result is that AI's failure mode *is* churn + duplication — i.e., a churn/duplication signal is the most direct available counterweight to the volume signals (PRs, lines) in the current metric. Today's metric rewards lines changed (winsorized), and GitClear's data shows copy/paste is the easiest way AI inflates lines — **our current output numerator is maximally exposed to exactly the pathology GitClear documents.** Also note GitClear's finding that <50% of raw changed lines are "meaningful" — raw line counts double-count no-ops and duplicates.

**Design rules if adopted (all grounded above):**
1. Normalize: relative churn within function/repo cohort, never absolute (Nagappan & Ball).
2. Use it as a *quality dampener* on volume, not a standalone score; keep the sign honest (some churn is healthy — target "not an outlier," not zero).
3. Attribute self-churn (own code revised) separately from being-revised-by-others; never charge authors for teammates' follow-ups without care (Flow's rework vs help-others split).
4. Window: churn finalizes 2 weeks after the observation window ends — compatible with the brief's ~2-week data lag but pushes toward the 8–12-week rolling window. Survival/half-life is disqualified for scoring (feedback loop measured in years).
5. Per-engineer variance is high at 4-week volumes *(inference — a typical engineer changes O(10²–10³) meaningful lines/month; churn events are rare)*; require a minimum changed-line floor, degrade to "no score" gracefully.
6. It still does nothing for agent-platform/non-PR engineers (same blindness as all git metrics).

**Code Time / effort-telemetry alternatives:** Software.com's Code Time (VS Code/JetBrains/Eclipse plugins, ~700k devs; "active code time" = typing with <10-min gaps) is an *effort* signal, not quality, requires new invasive editor telemetry, and is trivially gameable — credible only as opt-in self-reflection garnish, not for the score. Pluralsight Flow (GitPrime lineage; acquired by Appfire, Feb 2025) is the proof-of-existence for commercial per-engineer churn but its own positioning warns against individual performance use.

---

## 6. Sources (annotated)

1. **GitClear — Coding on Copilot (Jan 2024 report)** [vendor] — https://www.gitclear.com/coding_on_copilot_data_shows_ais_downward_pressure_on_code_quality — 153M lines 2020–2023; churn-to-double projection; churn defined as reverted/updated <2 weeks.
2. **GitClear — AI Copilot Code Quality 2025 (report page)** [vendor] — https://www.gitclear.com/ai_assistant_code_quality_2025_research — 211M lines 2020–2024; 4x clones headline.
3. **GitClear 2025 full PDF (v2025.2.5)** [vendor; primary data + verbatim definitions] — https://gitclear-public.s3.us-west-2.amazonaws.com/GitClear-AI-Copilot-Code-Quality-2025.pdf — A0 operation definitions; A1 raw yearly tables; A2 dataset composition (2/3 private opted-in, 1/3 OSS; repo list); A7 new-code churn table; A8 backfill-bias admission; DORA prediction record. *Read in full via local text extraction.*
4. **GitClear — The Maintainability Gap (2026 report)** [vendor] — https://www.gitclear.com/the_ai_code_quality_maintainability_gap — 623M changes 2023–2026; block dup +81%; long-term update % −74%; error-masking +47%; "default AI workflow" framing; AI-attribution mechanism undisclosed.
5. **LeadDev — "Code maintainability plummets in the AI coding era" (2026-07-07)** [independent] — https://leaddev.com/ai/code-maintainability-plummets-in-the-ai-coding-era — independent coverage of #4; Harding quotes; notes plateauing churn/duplication.
6. **GitClear — Mathematical proof of Diff Delta factors** [vendor] — https://www.gitclear.com/mathematical_proof_of_diff_delta_factors — Δ(ℓ) formula, six factors, per-operation base scores, per-developer E(d,T).
7. **GitKraken × GitClear — "AI Multiplier" research (PR, 2026-03-26)** [vendor] — https://www.prlog.org/13135413-gitkraken-research-ai-boosts-developer-output-25but-also-increases-code-churn.html — 2,172 developer-weeks, within-developer comparison; +25% Diff Delta; churn up to 9x for heavy AI users.
8. **Hacker News thread on the 2024 report** [practitioner] — https://news.ycombinator.com/item?id=39177008 — sharpest critiques: churn-as-quality-proxy questioned (datadrivenangel), churn-is-bad assumption contested (nzach), population-mix confound (naasking). Companion thread (mostly anecdote, no methodology critique): https://news.ycombinator.com/item?id=39168105.
9. **Nagappan & Ball — "Use of Relative Code Churn Measures to Predict System Defect Density" (ICSE 2005)** [independent/academic] — https://dl.acm.org/doi/10.1145/1062455.1062514 (also https://www.microsoft.com/en-us/research/publication/use-of-relative-code-churn-measures-to-predict-system-defect-density/) — absolute churn poor predictor; relative churn highly predictive (89% discrimination, Windows Server 2003).
10. **Erik Bernhardsson — "The half-life of code & the ship of Theseus" (2016)** [practitioner] — https://erikbern.com/2016/12/05/the-half-life-of-code.html — exponential-decay fits; aggregate half-life ≈3.33 years.
11. **git-of-theseus (OSS)** [practitioner] — https://github.com/erikbern/git-of-theseus — Kaplan–Meier line-survival from git history; per-author cohorts (`authors.json`, `.mailmap`); slow on large repos; Hercules cited as faster alternative.
12. **Appfire acquires Flow (2025-02-05)** [vendor] — https://appfire.com/newsroom/appfire-acquires-flow — Pluralsight Flow (ex-GitPrime) ownership; Flow = canonical commercial per-engineer churn/rework ("code <3 weeks old rewritten or deleted"; per-engineer). Exact help-center definitions were 403-blocked — corroborated via secondary: https://www.em-tools.io/engineering-metrics/code-churn [practitioner]. Flow help pages: https://help.pluralsight.com/help/metrics (unverified directly).
13. **Software.com Code Time docs** [vendor] — https://docs.software.com/plugins/code-time and https://www.software.com/engineering-metrics/code-time — "active code time" = typing with <10-min idle cutoff; editor-plugin telemetry; ~700k devs; docs updated Dec 2025 (still maintained).
14. **GitClear — 17 popular software engineering metrics and how they're gamed** [vendor] — https://www.gitclear.com/popular_software_engineering_metrics_and_how_they_are_gamed — GitClear's own gaming taxonomy for LOC/commits/coverage etc.
15. **devclass — "AI assistance is leading to lower code quality, claim researchers" (2024-01-24)** [independent] — https://devclass.com/2024/01/24/ai-assistance-is-leading-to-lower-code-quality-claim-researchers/ — early independent coverage of #1.
16. **The Register — "GitHub Copilot code quality claims challenged" (2024-12-03)** [independent] — https://www.theregister.com/2024/12/03/github_copilot_code_quality_claims/ — GitClear (Harding) critiquing GitHub's positive Copilot-quality study; context for the vendor-vs-vendor evidence war.
17. **jonas.rs — Report summary of GitClear 2025** [practitioner] — https://www.jonas.rs/2025/02/09/report-summary-gitclear-ai-code-quality-research-2025.html — broadly accepting summary; flags causation gap only implicitly.
