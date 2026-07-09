# Hercules (src-d/hercules) — git-history line-survival / burndown analysis

## Repo & commit
- URL: https://github.com/src-d/hercules [primary]
- Cloned (depth 1) to `/private/tmp/claude-501/-Users-jacksongeiger/044a2943-cb9f-4a0f-82cf-62b1372890ad/scratchpad/repos/hercules`
- HEAD: `68bb211faaedeffb53e799ab89e2aa48d8cb0ad3` (2022-11-29, v10; Go engine + Python `labours` plotting)
- Status: NOT archived, but dormant — GitHub API: `pushed_at: 2023-02-07`, 2,799 stars [primary]. The source{d} company behind it is defunct (widely reported; **unverified here** — I only verified repo dormancy). No successor repo found under the same lineage; the closest live analogue in spirit is git-of-theseus (https://github.com/erikbern/git-of-theseus [practitioner]), which hercules explicitly claims to reproduce "but much faster" (README.md).
- Reference the code itself cites: Erik Bernhardsson, "The half-life of code" https://erikbern.com/2016/12/05/the-half-life-of-code.html [practitioner] (comment at `leaves/burndown.go:33`).

## Files read
`leaves/burndown.go`, `leaves/devs.go`, `internal/burndown/file.go`, `internal/plumbing/line_stats.go`, `internal/plumbing/identity/identity.go`, `internal/plumbing/ticks.go`, `internal/plumbing/renames.go`, `internal/plumbing/diff.go` (grep), `internal/core/pipeline.go` (commit selection), `python/labours/modes/burndown.py`, `python/labours/modes/overwrites.py`, `python/labours/modes/ownership.py`, `README.md`.

## What it computes (concretely)

### 1. Line survival ("burndown") — the core
Each tracked file is a red-black interval tree mapping line ranges -> packed `(author, tick)` of the LAST edit of those lines (`internal/burndown/file.go`). It is an incremental blame: only last-modification time is kept, never full blame history.

- Packing (`leaves/burndown.go:1114`): `result = tick & TreeMergeMark; result |= person << TreeMaxBinPower` — 14 bits for tick (max ~16,382 ticks), 18 bits for author (max ~262,141 devs).
- Per commit: tree diff -> per-file diff (google diff-match-patch over lines mapped to runes; line count == rune count, `leaves/burndown.go:1358`) -> `File.Update(packedTime, pos, insLength, delLength)`. Every update fires callbacks that adjust:
  - `GlobalHistory[curTick][prevTick] += delta` — sparse matrix of "lines created at prevTick changed/deleted at curTick" (`updateGlobal`, :1143);
  - per-file history (optional, `--burndown-files`);
  - `PeopleHistories[prevAuthor][curTick][prevTick] += delta` — per-developer line survival (`updateAuthor`, :1170);
  - `matrix[oldAuthor][newAuthor] += delta` — the "overwrites matrix"; if `newAuthor == oldAuthor && delta > 0` it is recorded as `authorSelf` (`updateMatrix`, :1189–1198). So self-insertions vs. other-people-deleting-your-lines are separated.
- Output (`Finalize`, :514): `GlobalHistory`/`PeopleHistories` as `[samples][age-bands]` matrices (band = lines last edited in that time band that are still alive at sample time); `FileOwnership[file][dev] = alive lines` via last-writer-wins walk of the tree; `PeopleMatrix[dev] = [lines added, lines removed by unidentified, lines removed by dev_0, dev_1, ...]` (README "Overwrites matrix" section).
- Integrity is hard-enforced: after applying diffs, `file.Len()` must equal the diff's new line count or the run errors out (:1429).

### 2. Kaplan–Meier survival of lines (`python/labours/modes/burndown.py:125`, `fit_kaplan_meier`)
Bands are treated as populations of lines; a decrease of a band between consecutive samples = that many line "deaths" at that age; lines still alive at the last sample are **right-censored**; fits `lifelines.KaplanMeierFitter(T, E, weights=deaths)` and prints "Ratio of survived lines" vs. age. This is the statistically principled version of "code half-life" / GitClear-style churn-vs-durability.

### 3. Per-developer daily stats (`leaves/devs.go` + `internal/plumbing/line_stats.go`)
Per tick (default 24h) per developer: commit count + `LineStats{Added, Removed, Changed}`, additionally split per detected language. The Added/Removed/Changed classification (line_stats.go:121–147): within a modification hunk, a delete run followed by an insert run is paired — `min(del, ins)` counts as **Changed**, the excess as Added or Removed. So "changed" ~ replaced-in-place lines, distinct from net growth.
- Merge-commit diffs are entirely ignored for line stats ("we ignore merge commit diffs // TODO", devs.go:165–169; same in line_stats.go:83) — the merge only increments the commit count.

### 4. Time / "tick" handling (`internal/plumbing/ticks.go`)
- `tick := int(commit.Committer.When.Sub(tick0) / TickSize)` — **committer** time, not author time; tick 0 floored to tick boundary; warns on timestamps before 1990.
- Rebase defense (verbatim comment): "rebase works miracles, but we need the monotonous time" — `if tick < previousTick { tick = previousTick }` (clamps non-monotonic committer times).

### 5. Identity resolution (`internal/plumbing/identity/identity.go`)
- Default heuristic: lowercase email OR name matching with transitive union across the commit list (`GeneratePeopleDict`): known email+new name -> same person, new email+known name -> same person. Seeded from `.mailmap` at HEAD if present. Unmatched -> `AuthorMissing` ("<unmatched>").
- Optional external dict file: one person per line, `name|email|email2|...`, case-insensitive.
- Cross-run/cross-repo merging: identities are `|`-joined strings; `MergeReversedDictsIdentities` finds connected components over the shared name/email vertices (graph walk, :361–496).

### 6. Renames & branches
- Own rename detector (`internal/plumbing/renames.go`): default similarity threshold **80%** — "CGit's default is 50%. Ours is 80% because 50% can be too computationally expensive" (:41–44); min blob size 32; 60s per-commit timeout.
- Full-DAG analysis by default: pipeline **forks** the whole interval-tree state per branch and **merges** at merge commits by flattening files line-by-line; when the same line was introduced in different branches, "consider the oldest version as the ground truth" (`internal/burndown/file.go:300–305`); merge-conflict-resolution lines are attributed to the merge author at merge day. `--first-parent` linearizes history, "effectively decreasing the accuracy but increasing performance" (`internal/core/pipeline.go:449`).
- Acknowledged unsound edges (comments in `handleDeletion`, burndown.go:1281–1288): "Parallel independent file removals are incorrectly handled... happen *very* rarely, so we don't bother"; "Early removal in one branch with pre-merge changes in another is not handled correctly."
- Binary files skipped (line counting fails -> ignored); file becoming binary = deletion, becoming text = insertion.

## Data it assumes
- A **full clone with blobs** — it diffs actual file contents commit-by-commit. This is NOT a metadata-only analysis (relevant to our sibling mission's no-code-access constraint: hercules-style survival needs code bytes, though only mechanically, never semantically).
- Commit DAG + author name/email (+ optional `.mailmap`, optional identity dict). Nothing else.
- **No forge data at all**: no PRs, no CI, no reviews, no bot list, no force-push awareness (rewritten-away commits are simply invisible; rebases only mitigated by the monotonic-tick clamp).

## Cost/perf caveats (from README + code)
- torvalds/linux full burndown: **1h 40min** with `--burndown --first-parent --pb` (README, image caption) [primary].
- Burndown OOM is a documented failure mode with a dedicated README section; mitigations: disk-backed go-git, RBTree allocator "hibernation" to disk (`--burndown-hibernation-threshold/-disk`), `--first-parent`, coarser granularity/sampling ("5. --first-parent, you win.").
- Batch-only: every run recomputes the entire history; no incremental update store. For a 500–5000-engineer org-wide dashboard this is the wrong execution model — you'd want an incremental warehouse pipeline that ingests per-commit diffs once.
- 14-bit tick budget: with daily ticks ~44 years is fine, but hourly ticks overflow after ~682 days (inference from `TreeMaxBinPower = 14`).

## Design choices relevant to OUR problem

| Dimension | Hercules choice | Relevance |
|---|---|---|
| Quality adjustment | None built-in; quality = *survival* (lines still alive at age t), plus Added/Removed/Changed decomposition | Survival is a code-quality proxy computable WITHOUT CI, reviews, or AI attribution — orthogonal to our broken CI-clean-rate |
| Churn | K-M survival with right-censoring; self-overwrite vs. overwritten-by-others separated | Right-censoring honestly handles "code too new to judge" — the statistical fix for churn-window cold-start |
| Normalization | None in the engine (raw line counts); only plot-time row-normalization (overwrites matrix divided by lines written) | We must add cohorting ourselves; hercules shows the raw signal is per-repo, per-language decomposable |
| Identity | Heuristic email/name union + .mailmap + external dict; unmatched bucket kept explicit | In-org SSO/HR join mostly solves this, but the explicit `<unmatched>` bucket and bot-less design are gaps |
| Per-person vs per-team | Strictly per-repo, per-identity; cross-repo merging via identity connected components | Multi-repo aggregation pattern exists but everything is per-person raw counts |
| Gaming defenses | Essentially none: no bot filter, no vendored/generated-file exclusion, no size caps; only rename detection + oldest-line-wins merge de-duplication | Every defense must be added by us |
| Interpretability | Excellent: "your lines written in March, still alive today" is directly explainable; ownership/overwrite plots are human-readable | Matches our interpretability requirement |

### Gameability & unfair-penalty analysis (required per metric)
- **Line survival / half-life**: *Gamed by* writing code nobody touches — vendored deps, lockfiles, generated code, dead code, giant stable config blobs all "survive" 100%; also by discouraging or delaying refactors of one's own code, or by never deleting anything. *Unfairly penalizes* engineers in hot/experimental areas (ML experiments, early-stage product code), prototypers whose code is *supposed* to be replaced, and people doing healthy self-cleanup if self-deletion is counted as death. Mitigations we'd need: path/file-type exclusions, bot-author exclusion, separate self-churn vs. other-churn (hercules already separates these), and cohorting by repo volatility.
- **Lines Added/Removed/Changed per day**: *Gamed by* reformatting sweeps, vendoring, codegen, churny WIP commits (classic LOC gaming). *Penalizes* reviewers, architects, SREs, agent-platform engineers — anyone whose output isn't LOC. Use only as a decomposition (rework ratio), never as volume credit.
- **Ownership share (last-writer-wins)**: *Gamed by* mass-touching lines (format a whole file -> you now "own" it; the interval tree reassigns wholesale). *Penalizes* authors of dense, small, high-value diffs and anyone whose files get reformatted by others. Do not use as an individual credit metric.
- **Overwrites matrix**: *Gamed by* avoiding ever touching colleagues' code (looks "non-destructive" while blocking refactoring). *Penalizes* people assigned to cleanup/refactoring duty, who look like they "destroy" others' work. Use aggregate/anonymous only — as a leaderboard it is toxic by construction (inference).

## What we should steal
1. **Kaplan–Meier survival with right-censoring over line-age cohorts** (`fit_kaplan_meier`) — the single best idea here: a churn/durability quality signal with honest treatment of recent code; directly replaces brittle fixed-window churn and fixes the "merged yesterday" cold-start edge.
2. **Added/Removed/Changed decomposition** (paired delete-insert hunks) — cheap GitClear-lite rework taxonomy from pure git; "changed/(added+changed)" on one's own recent lines ≈ rework ratio.
3. **Self-overwrite vs. overwritten-by-others separation** (`authorSelf` in `updateMatrix`) — distinguishes iteration (fine, maybe even good AI usage) from rapid external replacement (durability problem).
4. **Identity connected-components merging + .mailmap seeding** — reusable pattern for cross-repo identity if HR join is incomplete.
5. **Monotonic-tick clamp on committer time** — one-line robustness against rebase timestamp chaos.
6. **Band/cohort framing for interpretability** — "of the lines you wrote in period X, Y% are alive" is self-explanatory to an engineer.

## What we should avoid
1. **Batch full-history recompute** — hours per large repo, documented OOM failure mode, no incrementality; an org dashboard needs an incremental pipeline keyed on new commits.
2. **Last-writer-wins ownership as individual credit** — reformat-gaming transfers ownership wholesale.
3. **No bot / vendored / generated-file filtering** — hercules counts everything; unusable raw at org scale.
4. **Dropping merge-commit line stats** — systematically undercounts integrators; conflict-line attribution to the merge author is arbitrary.
5. **Raw line counts without normalization/cohorting** — hercules never normalizes; all fairness machinery is on us.
6. **Requiring blob-level clones per analysis run** — expensive and (for the sibling no-code-reading constraint) a policy question even if analysis is purely mechanical.
7. Accuracy-vs-cost knobs silently change results: rename threshold 80% (vs git 50%), `--first-parent`, rename timeout — two runs with different flags produce different survival numbers; pin the config (inference from renames.go/pipeline.go).
