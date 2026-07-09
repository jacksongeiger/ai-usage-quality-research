# Repo notes: Code Maat (adamtornhill/code-maat)

**Investigated:** 2026-07-08, for the AI-leverage metric design mission.
**Repo:** https://github.com/adamtornhill/code-maat [primary] — cloned `--depth 1` to `/private/tmp/claude-501/-Users-jacksongeiger/044a2943-cb9f-4a0f-82cf-62b1372890ad/scratchpad/repos/code-maat`
**Commit:** `50537abb8dffa1b1ba4e91a605ee5c558c01b224` (2025-07-03). Clojure, GPL-3.0, ~984 LOC of analysis code. Actively the "book companion" tool; the README states the analyses "have evolved into CodeScene" (commercial successor) — repo not moved/renamed.

**Files read (all under the clone root):**
- `src/code_maat/analysis/churn.clj`, `effort.clj`, `authors.clj`, `communication.clj`, `logical_coupling.clj`, `coupling_algos.clj`, `sum_of_coupling.clj`, `entities.clj`, `code_age.clj`, `math.clj`
- `src/code_maat/app/app.clj` (analysis dispatch + pipeline), `team_mapper.clj`, `time_based_grouper.clj`, `grouper.clj`
- `src/code_maat/cmd_line.clj` (all thresholds/defaults), `src/code_maat/parsers/git2.clj`, `parsers/hiccup_based_parser.clj`
- `README.md` (analysis docs + Tornhill's warnings)

---

## 1. What it computes (concrete formulas)

Input schema after parsing: one row per **file-touch per commit**: `:entity :date :author :rev` plus `:loc-added :loc-deleted` (git/hg only). Everything below is computed on that flat table. Analysis registry: `app/app.clj` `supported-analysis` map.

### Per-author analyses (the ones that touch individuals)

| Analysis | Formula (from source) | File |
|---|---|---|
| `author-churn` | per author: Σ loc-added, Σ loc-deleted, `commits` = count distinct `:rev` | `churn.clj` `by-author` |
| `entity-ownership` | per (entity, author): Σ added, Σ deleted | `churn.clj` `as-ownership` |
| `main-dev` | per entity: main dev = argmax(added lines); `ownership = main_dev_added / max(total_added, 1)`, rounded to 2 decimals | `churn.clj` `pick-main-developer` |
| `refactoring-main-dev` | same but argmax(**deleted** lines) — comment: "when you remove code, you make a more active design choice… We speak refactoring here" | `churn.clj` |
| `entity-effort` | per (entity, author): `author-revs / total-revs` (commit counts, no LOC — works on all VCSs) | `effort.clj` `as-revisions-per-author` |
| `main-dev-by-revs` | main dev = argmax(author-revs); ownership = author-revs/total-revs | `effort.clj` |
| `fragmentation` | per entity: **fractal value** `FV = 1 − Σ_authors (revs_i / total_revs)²` — i.e., 1 − Herfindahl concentration; 0 = single author, →1 = many. Attributed in source comments to D'Ambros/Gall/Lanza/Pinzger research | `effort.clj` `as-entity-fragmentation` |
| `communication` | for each author pair: `shared` = # entities/revisions where both touched the same entity (frequency over per-entity author permutations); `strength = 100 × shared / ceil(mean(my_commits, peer_commits))`. Clever hack: self-pairs `[me me]` in the frequency map carry each author's total | `communication.clj` |
| `authors` | per entity: n distinct authors + n revs (source comment: author count per module correlates with quality problems) | `authors.clj` `by-count` |

### Entity-level analyses (no author)

- `entity-churn`: Σ added/deleted per file, sorted by **added** desc — docstring: lines added is "a better predictor [of post-release defects] than lines deleted". README concedes absolute (not relative) churn was chosen "because it's easier to calculate".
- `abs-churn`: added/deleted per date (trend; integration-bottleneck spotting).
- `coupling` (change coupling): `degree% = 100 × shared_revs / mean(revs_A, revs_B)`; included only if `revs ≥ min-revs (5)` AND `shared_revs ≥ min-shared-revs (5)` AND `30 ≤ degree ≤ 100`; change sets with > `max-changeset-size (30)` files are **dropped entirely** (`coupling_algos.clj` `within-threshold?`, `exceeds-max-changeset-size?`).
- `soc` (sum of coupling): per entity Σ over commits of (change-set size − 1), filtered > min-revs.
- `age`: months since last commit touching the entity.

## 2. Data it assumes

- A raw VCS log; preferred git2 format: `git log --all --numstat --date=short --pretty=format:'--%h--%ad--%aN' --no-renames --after=YYYY-MM-DD` (README). Note `%aN` = mailmap-normalized author **name**; that is the ONLY identity normalization. No emails, no dedup logic in the tool.
- Time-windowing is delegated to the user (`--after`); path exclusion delegated to git pathspecs (README example: `":(exclude)vendor/*"`).
- Binary-file numstat `-` coerced to 0 (`churn.clj` `as-int`). Merge commits contribute no rows (no numstat lines in this format; the grammar `entry = <prelude*> prelude changes` absorbs commits without file changes — `git2.clj` comment "covers pull requests").
- **No** bot filtering, **no** force-push/rebase awareness (analyzes whatever the rewritten history says), **no** CI data, **no** PR data at all — commits only. `--all` means unmerged branch work is counted.

## 3. Tornhill's own warnings about individual use [primary]

- README, "Churn by author" section, **verbatim**: "And, of course, you wouldn't use this data for any performance evaluation; it wouldn't serve well (in case anything should be rewarded it would be a net deletion of code - there's too much of it in the world)." (README.md line ~277)
- README, ownership section, verbatim: "Just note that the ownership metrics are sensitive to the same biases as the churn metrics; they're both heuristics and no absolute truths."
- His book *Software Design X-Rays* contains "Appendix 1, The Hazards of Productivity and Performance Metrics" — confirmed on the publisher's page: https://pragprog.com/titles/atevol/software-design-x-rays/ [primary]. (Appendix contents not readable online; existence verified, contents unverified.)
- Inference: the tool's own architecture backs this stance — per-author signals are framed as *navigation aids* ("who to ask when visiting a new module", `effort.clj` comment) and as inputs to *entity/team-level* risk analyses, never as scores.

## 4. Design choices relevant to OUR problem

1. **Team-first aggregation as a pipeline stage** (`team_mapper.clj`): an author→team CSV remaps `:author` **before** any analysis, so every individual algorithm runs unchanged at team granularity. Unmapped authors stay as themselves ("see them as a team in themselves") so mapping gaps surface immediately. Relevant to: privacy/fairness — same metric engine, coarser identity.
2. **Commit-style de-biasing** (`time_based_grouper.clj`): sliding-window regrouping treats all commits within N days as one logical change set, explicitly "to remove these biases" from workflows with "many small commits" vs multi-commit cross-team changes. Source comment warns the overlap double-counts commits, so it's valid for coupling but "not hotspots" — an honest scope-of-validity note. Relevant to: our PR-count and merged-PR metrics have exactly this style bias (squash vs stacked PRs).
3. **Concentration math instead of raw counts**: fractal value `1 − Σ(share_i)²` measures *distribution* of contribution, robust to many tiny contributions dominating. Relevant to: tool-mix/adoption-depth or work-spread subscores.
4. **Ratios normalized within the unit** (ownership = share of an entity's churn, coupling = share of mean revisions) rather than absolute volume across the org. Relevant to: cohort normalization.
5. **Noise floors as inclusion filters on entities** (min-revs 5, min-shared-revs 5, min-coupling 30%) — thresholds gate whether a *row* appears, they don't flip a person's score on/off.
6. **Mega-change-set rejection** (`max-changeset-size` 30 files): pathological commits are **dropped**, not capped — cheaper and more honest than winsorizing for coupling-type signals.
7. **Two competing "main developer" definitions** (added vs deleted lines) shipped side by side — an admission that any single ownership formula embeds a contestable value judgment.
8. **Churn as an entity-level quality proxy**: the tool's whole quality story is "high churn → more post-release defects" *per module*, from metadata only — never per person.

## 5. Gaming & unfair-penalty analysis (required per brief; per-author metrics)

| Metric | How an engineer games it | Who it unfairly penalizes |
|---|---|---|
| `author-churn` (LOC ±) | commit generated/vendored files, reformat-the-world commits, add-then-delete churn loops; AI makes this nearly free | config/infra/review-heavy engineers; people in repos that gitignore generated code; deleters (unless you invert to reward deletion, which is then gamed by delete-and-readd) |
| `entity-effort` / `main-dev-by-revs` (commit counts) | micro-commit splitting (tool's own temporal-period feature exists because commit counts are style artifacts) | squash-merge teams; engineers who batch work into large deliberate commits |
| `main-dev` / ownership ratio | pad lines into files you want to "own"; touch hot files superficially | maintainers who mostly delete/refactor; late arrivals to old files; pair/mob programmers whose partner commits |
| `fragmentation` (1−HHI) | funnel a team's commits through one committer to look "focused" | healthy pairing/mobbing teams (look fragmented); shared-ownership cultures |
| `communication` strength | co-commit to popular files to fake collaboration | remote/async collaborators whose coordination happens in review/docs, not co-edits; also breaks under squash (committer ≠ all contributors) |
| identity itself (`%aN` string) | commit under multiple names to split embarrassing history, or under a bot account | anyone with name variants across machines (their contribution fragments); non-Latin-name transliteration variants |

## 6. What we should steal

- **Run individual-grade algorithms at team/cohort granularity via an identity-mapping pre-stage**; keep person-level output self-view only. (Direct fit to our privacy constraint.)
- **Logical-change windowing** to neutralize PR/commit slicing style before counting output units.
- **Drop (don't just winsorize) pathological change sets** above a size threshold; cheaper to explain, kills the mega-PR vector outright for coupling-style signals.
- **Concentration indices (1−HHI)** for distributional subscores (e.g., AI-tool mix depth, spread of work) — resistant to padding with many tiny units.
- **Within-unit ratios** rather than cross-org absolute volume.
- **Exclude vendored/generated paths before any line counting** (client's current "lines changed" has no such filter — inference: cheap, high-value fix).
- **Scope-of-validity notes attached to each metric** (the codebase comments state where each heuristic breaks — do this in the dashboard UI).

## 7. What we should avoid

- LOC-added as contribution/ownership — the exact metric Tornhill ships and then tells you not to use on people; in an AI-assisted org it's the single most inflatable number.
- Commit/PR counts as effort without style normalization.
- Raw author-string identity; require org SSO/employee-ID joins (we have them), never VCS names.
- No-bot/no-merge/no-force-push naivety: any pipeline reading git logs must filter bot authors and be robust to history rewrites; code-maat simply isn't, because it was never meant to score people.
- Fixed inclusion thresholds create cliff effects (min-revs=5 here ≈ the client's brittle ≥3-PR floor); prefer confidence-weighted shrinkage toward a prior over hard gates (inference).

## Sources
- https://github.com/adamtornhill/code-maat [primary] — code + README quoted above, commit 50537ab.
- https://pragprog.com/titles/atevol/software-design-x-rays/ [primary] — confirms "Appendix 1, The Hazards of Productivity and Performance Metrics".
- https://pragprog.com/titles/atcrime2/your-code-as-a-crime-scene-second-edition/ [primary] — successor book; TOC has "Organizational Metrics" chapters, no individual-scoring appendix listed.
