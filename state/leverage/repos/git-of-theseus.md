# git-of-theseus — code survival/decay analysis (repo read)

## Repo & commit
- Repo: https://github.com/erikbern/git-of-theseus [primary] — clone succeeded, not moved/renamed.
- HEAD analyzed: `961bda027ffa9fcd8bbe99d5b8809cc0eaa86464` (2023-11-25, master). Small, stable Python package (~5 source files).
- Clone location: `/private/tmp/claude-501/-Users-jacksongeiger/044a2943-cb9f-4a0f-82cf-62b1372890ad/scratchpad/repos/git-of-theseus`
- Companion methodology post: "The half-life of code & the ping-pong of blame" (2016-12-05), https://erikbern.com/2016/12/05/the-half-life-of-code.html [primary — tool author]. Reported half-lives via exponential fit: Linux 6.60y, Rails 2.43y, aggregate across sampled OSS projects ≈3.33y (verified via fetch 2026-07-08).
- Files read: `git_of_theseus/analyze.py`, `git_of_theseus/survival_plot.py`, `git_of_theseus/stack_plot.py`, `git_of_theseus/line_plot.py`, `git_of_theseus/utils.py`, `README.md`.

## What it computes — exact algorithm

### Stage 1: `analyze.py` (`analyze()`, lines 238–544)
1. **Cohort assignment** (lines 298–314): iterate ALL commits on the branch; each commit gets `cohort = strftime(cohortfm)` of its commit date (default `%Y` → year cohorts). Curve keys registered per commit: `("cohort", year)`, `("author", name)`, `("domain", email.split("@")[-1])`. If `.mailmap` exists at repo root, author name/email are canonicalized via `git check-mailmap` (`get_mailmap_author_name_email`, lines 547–552).
2. **Snapshot sampling** (lines 316–328): walk **first-parent only** (`commit.parents[0]`) from HEAD backwards, keeping snapshots ≥ `interval` apart (default `7*24*60*60` = weekly). These snapshots are the x-axis.
3. **File filter** (lines 33–53, 338–354): only files whose extension matches a Pygments lexer filetype, MINUS an explicit ignore list of non-code types: `*.json, *.md, *.txt, *.xml, *.yaml, *.yml, *.ps, *.eps, ...` (`IGNORE_PYGMENTS_FILETYPES`). Overridable with `--all-filetypes`, `--only`, `--ignore` globs.
4. **Incremental blame** (lines 464–493): between consecutive snapshots, compare per-file blob SHAs; only re-`git blame` files whose blob changed; subtract the previous per-file contributions for modified/deleted files. Optional `-w` (ignore whitespace) via `--ignore-whitespace`. No `-C`/`-M` copy/move detection is passed (whole-file renames are still followed by blame's default behavior; cross-file chunk moves re-attribute lines to the mover).
5. **Attribution** (`get_file_histogram`, lines 88–118): for every blamed line at a snapshot, credit 1 line to each of `(cohort, ext, author, dir, domain)` and — for commits seen in step 1 — `("sha", hexsha)`. Blame failures are swallowed: `except: pass` (line 116–117) → file silently contributes nothing.
6. **Outputs**: `cohorts.json`, `authors.json`, `exts.json`, `dirs.json`, `domains.json` — each `{y: [[counts per snapshot]...], ts: [...], labels: [...]}` = surviving-line time series per key; and `survival.json` = `commit_history[sha] = [(snapshot_ts, lines_still_blamed_to_sha), ...]`.

### Stage 2: `survival_plot.py` (`survival_plot()`, lines 32–120)
Aggregate survival curve over ALL commits, in *relative* time (age since a commit's first appearance):

```python
for commit, history in commit_history.items():
    t0, orig_count = history[0]
    total_n += orig_count
    last_count = orig_count
    for t, count in history[1:]:
        deltas[t - t0] += (count - last_count, 0)      # line deaths at age t-t0
        last_count = count
    deltas[history[-1][0] - t0] += (-last_count, -orig_count)  # end of observation
...
P = 1.0
for t in sorted(deltas.keys()):
    delta_k, delta_n = deltas[t]
    P *= 1 + delta_k / total_n     # hazard step
    total_n += delta_n             # shrink risk set
    if P < 0.05: break
```

- README calls this Kaplan-Meier. It is K-M-like: per-age hazard applied multiplicatively, risk set (`total_n`) shrunk as commits exit observation. **Inference/caveat:** at a commit's last observation, its still-surviving lines (`-last_count`) are folded into `delta_k` (the death term) at the same tick where the risk set shrinks, so censored survivors leak into the hazard — an approximation, not textbook K-M right-censoring. Aggregated over thousands of commits the curve is dominated by real deaths, but a per-person version with few commits would inherit this distortion.
- **Half-life** (`--exp-fit`, lines 79–109): least-squares fit of `total_n * exp(-k * t/YEAR)` to the aggregate curve via `scipy.optimize.fmin`; label printed as `half-life = ln(2)/k` years. One interpretable number summarizing decay.

### Plot helpers
- `stack_plot.py` / `line_plot.py`: top-`max_n` series by peak, rest rolled into "other"; `--normalize` → share-of-total %. No metric logic beyond that.

## Data it assumes
- Full (non-shallow) git clone of one repo; a single trackable branch; ability to run `git blame` on every changed file at ~weekly snapshots (expensive: mitigated by blob-hash diffing and multiprocessing).
- git author name/email as identity; `.mailmap` as the only dedup mechanism. Email domain as a crude org-affiliation signal.
- **No** PR data, **no** CI data, **no** bot filtering, **no** force-push/rebase handling (history rewrite silently changes everything), **no** merge-commit semantics beyond first-parent sampling, **no** notion of task/PR/person cohorts — cohorting is by *calendar time of authorship* only.
- Line survival == "still returned by `git blame` for that commit at a later snapshot". Any edit, chunk move (without `-C`), or reformat (without `-w`) kills a line and births a new one owned by the toucher.

## Design choices relevant to OUR problem
- **Quality adjustment:** none. Survival IS the implicit quality claim ("code that lasts was worth writing"). The tool never asserts direction; the per-engineer proxy would have to, and that assertion is the weak point (see failure analysis).
- **Churn:** measured as line death at snapshot resolution. Weekly first-parent snapshots mean code added and deleted *within the same week* never appears in any snapshot → it is invisible, not counted as fast death. Survival is thus overestimated, and — critically for us — **the sub-2-week churn that GitClear-style AI-churn metrics target is exactly what this design cannot see.** Per-commit diff streams, not snapshot blame, are needed for short-window churn.
- **Normalization/cohorting:** two good moves. (1) Age-relative alignment: every commit's curve is shifted to its own t0, so 2015 code and 2023 code are compared at the same *age*, not the same date. (2) Right-censoring: commits too recent to have died are removed from the risk set rather than counted as survivors — the exact discipline the current leverage metric lacks (it treats <3 CI PRs as a perfect 1.0 instead of "not yet observable").
- **Identity:** `.mailmap` canonicalization + email-domain grouping. Minimal but the right shape: identity resolution happens once, upstream of all metrics.
- **Gaming defenses:** essentially none — it was built as descriptive repo archaeology, not a people metric. The only accidental defense is the Pygments code-only filetype filter (JSON/YAML/MD/lock-ish files excluded), which happens to block the cheapest line-count inflation vector.
- **Per-person vs per-team:** `authors.json` gives per-author surviving-line series, but the survival estimator is only ever run on the whole-repo aggregate. Nothing in the tool endorses per-person survival scoring.

## "Half-life as a per-engineer quality proxy" — why it fails
Hypothetical metric: K-M survival (or fitted half-life) of lines authored by engineer E; higher = "more durable, higher-quality output".

**(a) How would an engineer game it?**
1. Write code nobody touches: dead code, one-off scripts, generated files, verbose boilerplate in cold paths. Untouched == immortal. AI makes producing plausible cold code nearly free.
2. Blame capture: run a mass reformat/restyle or chunk-move commit — `git blame` (no `-C`, `-w` off by default) re-attributes every touched line to you, resetting their clocks under your name.
3. Ownership defense: resist teammates' refactors of "your" files; never delete your own obsolete code; steer review comments away from rewrites of your lines.
4. Verbosity: line-weighted aggregates reward writing 300 lines where 30 would do, provided they sit still.

**(b) Who does it unfairly penalize?**
1. Engineers on fast-iterating products (prototypes, growth experiments, frontend): their code *should* die quickly; deletion of a killed feature is a business decision, not an authorship-quality signal. (Blog data agrees: Angular-era half-life 0.32y vs Linux 6.60y — repo choice dwarfs individual skill [primary].)
2. Refactorers and hot-path owners: high-traffic code gets edited by everyone; survival confounds *exposure to change* with *quality*.
3. New hires / recent work: heavily right-censored → unstable or missing scores (K-M handles it statistically but per-person sample sizes are tiny; the estimator's censoring quirk above bites hardest here).
4. Anyone downstream of a repo-wide reformat, lint migration, or file reorganization — their half-life collapses through no action of their own ("ping-pong of blame").
5. **Fatal for our use case regardless:** feedback latency is measured in *years* (aggregate half-life ≈3.33y [primary]) against a 4–12-week scoring window; also requires reading/blaming source trees, which the sibling mission's constraint forbids and which our warehouse-only data (PR counts, lines, CI bits) cannot reproduce. Direction is ambiguous in both directions: surviving ≠ good (fossilized fear-to-touch code survives), deleted ≠ bad.

## What we should steal
1. **Right-censoring discipline (the big one).** Never score an outcome that hasn't had time to occur. Applies directly to: cold-start floors (report "insufficient observation," never a neutral 1.0), and any durability-flavored signal (e.g., "% of E's merged PRs *not followed by* a revert/fix-PR within 21 days" must be computed only over PRs ≥21 days old, K-M-style).
2. **Age-relative cohort alignment.** Compare work at the same *age since merge*, and cohort people by comparable environments — the analog for us is cohorting by function/repo-cluster, since decay/churn norms are repo properties, not person properties.
3. **Non-code filetype exclusion** (Pygments allowlist minus JSON/YAML/MD/lock files) before any line-based metric. The current metric's 2000-line winsorization caps a lock-file PR but still credits it; an exclusion filter is the correct upstream fix.
4. **Whitespace-insensitive diffing** (`-w`) so formatting churn doesn't pollute line metrics.
5. **Identity canonicalization upstream** (mailmap → employee-ID mapping in the warehouse) before any per-person aggregation.
6. **Cheap incrementality pattern**: hash-diff to recompute only what changed — the right shape for a lagging warehouse pipeline.
7. **Half-life as a presentation device**: one number in time units is far more interpretable than a percentile — IF the underlying signal is fast enough (e.g., "median time-to-first-fixup of your PRs"), not multi-year line survival.

## What we should avoid
1. Per-engineer line-survival/half-life as a leverage or quality score — gameable (blame capture, cold dead code), confounded (product churn, exposure), too slow (years vs weeks), and identity-unstable (reformat ping-pong).
2. Snapshot sampling for churn measurement — misses all sub-interval churn, i.e., exactly the AI fast-rework signal; use per-PR/per-commit event streams instead.
3. Silent data loss (`except: pass` around blame; unhandled force-pushes/history rewrites) — in a people-facing metric, dropped observations must surface as "unmeasured," never as an implicit zero or an implicit pass.
4. Treating no-signal as best-signal — the estimator's cousin in the current metric is CI-clean defaulting to 1.0; the survival framing shows the correct alternative (censor, don't impute perfection).
5. Any metric requiring `git blame` over source trees at org scale — computationally heavy and outside our warehouse data contract.
