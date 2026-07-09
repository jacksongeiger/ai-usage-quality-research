# Vendor Frameworks for Per-Engineer AI Impact Measurement (2025–2026)

Sub-investigator report — commercial engineering-intelligence vendors: DX (getdx), Jellyfish, LinearB, Swarmia, Uplevel, Faros AI, Athenian, Waydev.
Researched 2026-07-08 from vendor product docs/blogs plus independent corroboration. All quotes verified against fetched pages unless marked otherwise.

**Headline (inference):** No vendor in this set publishes a per-engineer "AI leverage" score, and none uses a per-dollar-per-engineer denominator. The universal industry pattern is a **Utilization → Impact → Cost funnel measured at team/cohort level**, with impact inferred from cohort deltas on standard delivery metrics (cycle time, throughput, review time, quality counterweights). The client's current metric (quality-adjusted output ÷ individual AI spend, percentile-ranked) has **no commercial precedent** — every vendor either aggregates cost at org level or warns explicitly against individual performance use.

---

## 1. What it measures & theory (per vendor)

### 1.1 DX (getdx) — AI Measurement Framework
Theory: three dimensions mirroring an adoption journey — **Utilization → Impact → Cost** ("organizations start by focusing on usage, then shift to measuring impact, and finally to understanding and optimizing ROI").

**Utilization signals** (telemetry):
- DAU/WAU/MAU of AI tools; adoption cohorts "no adoption, light, moderate, or heavy"; power-user identification (individual-level).
- **% AI-assisted PRs** and **% AI-generated code committed** — detected via proprietary "file system-level observability" analyzing "the rate and pattern of file changes in real-time," claiming to distinguish "human typing and AI-generated batch modifications," trackable "down to the commit, user, repo, and branch level," "across all IDEs, AI tools, and CLI agents."
- "Tasks assigned to agents" + "human-equivalent hours by agents" — announced, under development as of the implementation guide.

**Impact signals**:
- **AI-driven time savings** (hrs/dev/week) — from quarterly surveys / "experience sampling" (self-report; "developers report saving approximately 2 hours per week, with high-end users saving 6 hours or more").
- Per-tool **CSAT** (quarterly survey).
- **Core 4 correlation**: PR throughput, change failure rate, Developer Experience Index, % time on feature development — "compare AI users versus non-users, and heavy users versus moderate users," plus "before-and-after trend analysis," broken down "by team or custom attribute."
- Adjacent DX metrics: TrueThroughput (LLM-adjusted PR complexity), PR cycle time, **PR revert rate** (quality proxy from PR metadata).

**Cost/ROI**: ROI calculator = time saved (default **2.4 hrs/wk**) × engineers × hourly cost (e.g., $78/hr from $150K salary) ÷ tooling cost (e.g., Copilot $19/user/mo) → marketing examples of "~39x ROI." Spend imported into custom tables; "level of detail available depends on what vendors provide."

**Cohorting/normalization**: adoption-level cohorts, team/custom-attribute breakdowns; Core 4 doctrine says to counterbalance speed metrics with DXI, and individual "diffs per FTE" is allowed only with "no targets/rewards tied to it."
**Explicit warning**: "Metrics like code generation volume are particularly susceptible to gaming"; avoid "individual performance evaluation."

### 1.2 Jellyfish — AI Impact + GitHub Copilot Dashboard
Theory: connect "AI usage and spend to delivery metrics such as throughput, cycle time, code quality, and cost efficiency" — "defensible ROI data instead of vendor-reported activity metrics."

- **Utilization**: active users/adoption rate by team, segment, and usage level ("power users, casual users, unlicensed engineers"); suggestion **acceptance rate by language and editor**; "who's using AI, where, how, and which tool from automatically detected system signals" (mechanism not disclosed).
- **Cost**: "AI Token Cost Management — monitor spend and token usage by tool, team, or initiative." (Closest any vendor gets to per-dollar framing; still tool/team/initiative-level, not per-engineer output-per-dollar.)
- **Impact**: "compares issue cycle time, PR cycle time, and throughput for AI tool users against non-users"; work-type and roadmap-vs-KTLO allocation shifts.
- Marketing numbers: own team "up to 34% faster" cycle time for Copilot users; cross-customer claims from "more than 300 companies and 20,000 engineers": "25% speed up in coding time," "12% increase in PR throughput," "17% increase in roadmap work vs KTLO." No statistical methodology disclosed anywhere I could find [inference: raw user/non-user comparison, self-selection-confounded].
- Tools: Copilot, Cursor, Claude Code, Amazon Q, Gemini Code Assist, Windsurf, CodeRabbit, Devin, Copilot Agent, Jules — "one consistent, vendor-neutral measurement model."
- Granularity: team/initiative aggregates plus "individual power users."

### 1.3 LinearB — AI Measurement Framework (Adoption + Impact)
Theory: two-part — **Adoption** ("are teams actively using AI tools?") and **Impact** ("measurable difference in engineering performance and quality"), with a benefits-vs-risks dashboard design.

- **Adoption**: DAU by org/group/team; "lines of code suggested by GenAI tools"; "acceptance rate of AI-suggested code"; tracks 24+ AI tools.
- **Impact — core four**: Time to Merge; Merge Frequency; **PR Maturity** ("how review-ready is a PR on its first submission?"); **Rework Rate** (merged code rewritten within 21 days).
- Dashboard triad: Adoption (PRs opened/merged) / **Benefits** (merge frequency, coding time, completed stories, planning accuracy) / **Risks** (PR size, rework rate, **PRs merged without review**, review time, time to approve). The explicit "risks" panel is the clearest vendor implementation of a quality counterweight.
- **Attribution**: PR labels — (a) manual self-labeling; (b) **gitStream auto-labeling** from "list of users included in the GenAI experiment," "hint text in the PR title, commit messages, or PR comments," or a forced "Yes/No button as part of the PR flow." Also claims commit-level tagging into human / AI-assisted / fully-AI-generated, mechanics undisclosed.
- **Cohorting**: AI-labeled PRs vs all other PRs as baseline (dashboard shows labeled-vs-baseline lines).
- **ROI**: explicitly rejects "hours saved" metrics; models ROI as deltas in throughput, cycle time, rework/defects, engagement. No per-dollar formula published.

### 1.4 Swarmia — AI tools suite (most transparent docs)
Theory: "There isn't one metric that can tell the full story" — combine adoption, PR-level comparisons, agent metrics, quality signals, and surveys.

- **Detection (fully documented)**: a PR is AI-assisted if (1) "any of its commits were authored or co-authored by an AI tool"; (2) a commit "has the `Made-with: Cursor` Git trailer"; (3) "it has the `claude-code-assisted` label"; or (4) low-confidence heuristic — "any of its commits were made by an author who used an AI tool within the previous 24 hours" (because "some AI coding tool providers report user activity only on the daily level"). Admits: "This can result in overreporting AI involvement… if you simultaneously work on tasks A and B, but only use AI for task A, Swarmia inaccurately associates the tool usage also with task B." High-confidence-only filter available.
- **Impact metrics** on AI vs non-AI PRs: PRs merged (throughput), cycle time, time to first review, time in review, batch size; grouped by tool (Copilot/Claude Code/Cursor) and mode (local / cloud agent / review agent). Double-counting caveat: a PR touched by two tools counts once in each category.
- **Adoption**: licenses, active users, idle seats, per team.
- **Agent metrics** (template for agent-platform measurement): "Agent PRs," Merged/Closed, "Merge percentage" ("how effectively people are able to use agents"), "Agent commit percentage" (share of merged agent PRs needing human intervention vs "100% agent commits"), batch-size distribution, per-team tabs. Review agents: which agents review PRs "and how much code they cover."
- **Surveys**: developer experience surveys on "AI tool usage, perceived speed, quality, and friction."
- **ROI stance**: skeptical — "no single measure of productivity and, thus, no simple definition of ROI"; warns vendor productivity percentages rest on "narrow definitions, broad assumptions, or flawed statistics." Named caveats: no clear baseline; fragmented tool mix; "gains in one metric can unintentionally hurt others"; "early adopters tend to be high performers, skewing results" (self-selection); overreliance → tech debt.

### 1.5 Uplevel — SEE platform + Uplevel Data Labs Copilot study
Theory: system-level observability of how work moves; AI is one input among meetings/focus/flow.

- **Signals**: deep work ("blocks of two or more hours"; from calendar + chat interruption classification), cycle time, DORA benchmarks, "deep work availability, and AI adoption patterns" benchmarks; "AI Impact and ROI… AI investment comparison" feature (no formula published); FlowState™ monitors "PR complexity, review wait times, and handoff delays."
- **Privacy**: "Privacy protections keep individual developer data aggregated at the team level" — no per-engineer scores by design.
- **Copilot study (2024, Uplevel Data Labs)**: ~800 developers; Copilot-access vs pre-access baseline; measured "cycle time and PR throughput," "incidence of PRs with bugs," retained issues, out-of-hours work. Finding: **no cycle-time/throughput improvement and a 41% increase in bug rate** for Copilot users ("Developers with Copilot access saw a significantly higher bug rate while their issue throughput remained consistent"). Full report download-gated; operational bug definition not public [flag: unverifiable methodology detail; 41% figure corroborated across secondary coverage].

### 1.6 Faros AI — AI Copilot Evaluation module / AI Transformation platform
Theory: "establish baselines, quantify lift, and communicate ROI across speed, throughput, quality, and risk — by team, repo, and workflow"; strongest cohort/causal ambitions of the set.

- **Adoption**: daily/weekly/monthly adoption; "acceptance rates and lines of code generated, by language and editor"; "percentage of AI-generated code by repo"; "unused licenses and power users." Power user = "usage on 15 different days or in 30 different hours in a calendar month" (docs.faros.ai, retrieved via search snippet — docs page itself blocked; mark partially verified).
- **Impact**: velocity (PR merge rate, PR merge time, review time, PR cycle time, lead time, throughput) + quality (test coverage, code smells, bugs, incidents, change failure rate, MTTR, test flakiness) + sentiment surveys. Methods: before/after baselines ("baseline prior to using the AI coding tool"), cohort comparison, claimed "cause-and-effect analysis" (methodology not published).
- **Granularity**: "Data displayed in our dashboards is at the team level," with optional power-user identification.
- **ROI**: GAINS diagnostic; third-party review reports a cost-per-incremental-PR framing (e.g., "$37.50 cost per incremental PR… 4:1 ROI") [independent, low confidence — RockB blog].
- **AI Engineering Report 2026 ("The Acceleration Whiplash")** — telemetry, not surveys: 2 years, ~22,000 developers, 4,000+ teams: epics/developer **+66%**, task throughput/developer **+33.7%**, PR merge rate/developer **+16.2%**, **bugs per developer +54%**, incident-to-PR ratio more than tripled, median PR review time **5x**, **31% more PRs merging without any review**; "strong engineering foundations did not shield teams." This is the largest telemetry counterweight to rosy vendor deltas.

### 1.7 Athenian — (pre-AI-era; philosophy precedent)
- Team-level-only by explicit design: founder Eiso Kant said competitors "focus too much on individual performance" and engineers "hate them because they feel like surveillance software"; Athenian chose "to start over from scratch and focus on teams and events instead of individuals."
- Metrics: PR workflow stages (plan/design, review, release), release frequency, outstanding bugs by priority, CI build failure rates, bottleneck detection from a GitHub+Jira+CI/CD event graph.
- Status: acquired by The Linux Foundation, Nov 2023 (Crunchbase via search [independent]); GitHub org activity through ~mid-2024; **no AI-impact measurement product found** — relevant only as a privacy/aggregation design precedent, not as an AI framework. (Unverified whether the commercial product still operates.)

### 1.8 Waydev — AI Adoption dashboard (Copilot)
- Ingests GitHub Copilot metrics via GitHub API into an "AI Adoption" dashboard; docs note "the API does not provide per-user data for usage, and metrics only cover usage via supported IDEs (with telemetry enabled)."
- Impact = **naive before/after on Waydev git metrics**: coding time, cycle time ("when they come down after you've deployed Copilot, you've got quite a good indicator your productivity increased"), commit frequency ("an upward trend after adopting Copilot suggests that engineers are moving faster"), active coding days as "engagement," time-to-first-commit/first-merge for onboarding speed. "At both individual and team levels."
- No ROI formula; no cohort controls; weakest methodology in the set [inference].

---

## 2. Evidence quality

- **Best-documented computation**: Swarmia help docs (exact detection rules, confidence tiers, admitted false positives) and DX's implementation guide (named metrics + data sources). LinearB's gitStream labeling rules are concrete but attribution is fundamentally declarative.
- **Vendor marketing numbers** (Jellyfish 34%/25%/12%, DX "39x ROI," Faros "10x PR velocity") are unaudited, methodology-free, and self-selection-confounded — treat as [vendor] claims only.
- **Two vendor studies publish against interest**, which raises credibility: Uplevel 2024 (no speed gain, +41% bugs; gated methodology) and Faros 2026 Acceleration Whiplash (throughput up, bugs +54%, review time 5x; large-N telemetry; PDF public). Both remain [vendor]-produced, non-peer-reviewed.
- **Independent coverage** is thin: trade press (ADTmag, Visual Studio Magazine, The New Stack) restates vendor numbers; the RockB Faros review is detailed but a personal blog [independent, low confidence].
- Nothing here is an RCT; no vendor publishes confidence intervals, matching procedures, or regression specs. Faros' "causal analysis" claim is unverified.

## 3. Known failure modes (cross-vendor)

1. **Attribution is unsolved.** Three approaches, all leaky: declarative labels (LinearB — self-report/forced buttons), git trailers + 24-hour activity heuristic (Swarmia — documented over-reporting), proprietary file-change-pattern observability (DX — unverifiable black box). The client's abandoned "AI-made" tag mirrors industry experience.
2. **Self-selection confound** in every users-vs-non-users delta (Jellyfish, DX Core-4 correlation, Faros cohorts): "early adopters tend to be high performers, skewing results" (Swarmia's own words). Before/after designs (Waydev, Uplevel baseline) confound with time trends, team changes, model upgrades.
3. **Acceptance rate ≠ value**: Windsurf (quoted via Ferrari): "percent of code written is not actual productivity… many code assistant vendors muddle metrics such as acceptance rate with value propositions such as developer productivity." Accepted code can be immediately rewritten.
4. **Self-reported time savings anchor ROI** (DX 2.4 hrs/wk default → 39x): unverifiable, anchoring-prone, and the whole ROI number is linear in it. LinearB explicitly rejects hours-saved for this reason.
5. **Speed metrics without quality counterweights invert under AI**: Faros telemetry shows throughput up while bugs/dev +54% and unreviewed merges +31%; Uplevel +41% bug incidence. A leverage score built on throughput alone rewards the failure mode.
6. **Daily-granularity vendor APIs** force 24-hour attribution windows (Swarmia) and org-level-only usage (Waydev/Copilot API historically) — per-user precision is often vendor-limited.
7. **Double counting** across tools (Swarmia documents it); multi-tool users break single-tool ROI accounting.

## 4. Gameability & who it penalizes (per signal family)

| Signal (vendors) | How an engineer games it | Who it unfairly penalizes |
|---|---|---|
| DAU/WAU, active days, power-user badges (DX, Jellyfish, Faros 15-day/30-hour rule) | Open the tool daily, fire trivial prompts; leave sessions running | Engineers who batch AI use into deep-work bursts; part-time/on-call weeks; those whose tools under-report telemetry |
| Acceptance rate, LOC suggested/accepted (Jellyfish, LinearB, Faros) | Accept everything, then rewrite (accept-then-delete); prompt for verbose boilerplate | Careful suggestion reviewers; devs in languages/domains where AI suggests poorly; deleters/refactorers (negative LOC) |
| % AI-generated code / % AI-assisted PRs (DX, Faros, LinearB commit tags) | Ask AI to regenerate files it barely changed; route trivial edits through the agent; spoof git trailers or labels (a one-line `Made-with:` trailer is free text) | Architects/reviewers/mentors whose value isn't committed code; DX itself: "code generation volume [is] particularly susceptible to gaming" |
| AI-assisted PR labels incl. 24h heuristic (Swarmia, LinearB) | Touch an AI tool once per day → ALL your PRs count as AI-assisted (Swarmia low-confidence rule); or under-label to look "efficient without AI" | Honest non-labelers; teams where the forced Yes/No button becomes compliance noise |
| Self-reported time savings (DX) | Inflate hours; copy the vendor's 2.4h anchor | Honest reporters; skeptics score their org's ROI down |
| Cycle time / time-to-merge / merge frequency deltas (all) | Slice work into micro-PRs; pre-negotiate reviews offline; merge trivia | Complex-domain and heavy-CI teams; multi-timezone review chains; non-PR work (infra, agents, research/ML) |
| Rework rate, 21-day rewrite (LinearB) | Defer fixes past day 21; avoid touching own recent code | Fast iterators; legitimate refactoring; hotfix-heavy teams |
| PRs merged without review (LinearB risk, Faros +31%) | Solicit rubber-stamp approvals | Solo maintainers; emergency responders |
| Bugs/incidents per developer (Faros, Uplevel) | Under-file bugs; reclassify tickets | Teams with rigorous triage culture; people assigned to bug-dense legacy code |
| Org ROI = hrs×salary÷cost (DX) / cost-per-incremental-PR (Faros via RockB) | Inflate numerator surveys; suppress denominator by rationing licenses | High spenders doing legitimately harder work — the same flaw as the client's per-dollar score |

## 5. Applicability to our constraints (HAVE / CANNOT mapping)

**Directly importable with data we HAVE:**
- **DX-style utilization cohorts** (none/light/moderate/heavy; breadth across tool classes; active-day cadence) — we have per-user/per-day/per-tool dollars, tokens, sessions, active days across ALL tools including the agent platform. This is the best-supported vendor pattern and replaces per-dollar division with cohort placement.
- **Cohort-delta impact analysis** (Jellyfish/DX/Faros pattern): regress or compare our PR throughput/lines/CI outcomes across usage-intensity cohorts **within function × level** — fixes the brief's problem #6 (vendors normalize by team/custom attributes, never level-across-all-functions). Usable only as aggregate/eval evidence, not as an individual score (self-selection).
- **LinearB's benefits-AND-risks pairing**: every output signal shipped alongside a quality counterweight. With no defect linkage, our nearest counterweights from PR metadata: revert-rate (DX uses PR revert rate), merged-without-review flag, PR-size distribution — all computable from PR/CI metadata we plausibly have or can cheaply add.
- **Swarmia's agent metrics as the template for our agent-platform gap**: agent runs → PRs, merge %, human-intervention share. Requires linking agent-platform runs to PR artifacts — the single highest-value NEW signal for the invisible-agent-engineer problem (brief hard limit #4). Without that link, vendors have nothing per-user either.
- **Survey triangulation** (DX time savings + CSAT; Swarmia DevEx surveys): cheap, works for non-PR engineers, standard practice at every credible vendor — a strong candidate NEW signal (self-reported time-savings + per-tool satisfaction), with the known inflation caveat.

**Blocked by our CANNOTs:**
- All AI-vs-human attribution (LinearB labels, Swarmia trailers, DX file-system observability) — brief hard limit #1. Note: Swarmia's docs show tools like Cursor/Claude Code already emit trailers/co-author lines, so *partial, gameable* attribution exists in raw git data; the brief says this was tried and dropped — do not resurrect as a scoring input, possibly acceptable as non-scored context.
- DX TrueThroughput (LLM complexity-adjustment of PRs) would address our "no task difficulty" limit but requires reading diffs — likely out of bounds; list under new-signals-with-caveats only.
- Faros/Uplevel defect-and-incident linkage — we lack per-author post-merge defect data (hard limit #3).
- Faros/Jellyfish org-level ROI framing is fine for the org, but no vendor validates per-engineer per-dollar scoring — evidence *against* keeping the current denominator.

**Norm alignment:** Uplevel and Athenian aggregate individuals away entirely; Faros defaults to team-level; DX warns against individual performance evaluation. A private self-view dashboard is *more* individual than any vendor ships — our privacy constraint (self-view only, no rankings) is the minimum viable ethical posture, and vendor precedent supports multi-signal panels over one score.

## 6. Sources (annotated)

**DX (getdx)**
1. [vendor] DX — Measuring AI code assistants and agents (framework overview, 3 dimensions, gaming warning): https://getdx.com/research/measuring-ai-code-assistants-and-agents/
2. [vendor] DX — How to implement the AI Measurement Framework (every metric + data source; file-system observability detection; cohorts): https://getdx.com/blog/how-to-implement-ai-measurement-framework/
3. [vendor] DX — AI ROI calculator (2.4 hrs/wk default, hrs×salary÷cost, 39x example, experience-sampling basis): https://getdx.com/blog/ai-roi-calculator/
4. [vendor] DX — 5 metrics in DX to measure AI impact (TrueThroughput, PR revert rate): https://getdx.com/blog/5-metrics-in-dx-to-measure-ai-impact/
5. [vendor] DX — AI measurement framework guide (utilization/impact/cost narrative): https://getdx.com/blog/ai-measurement-framework-guide/

**Jellyfish**
6. [vendor] Jellyfish AI Impact (adoption/spend/impact signals, token cost management, tool list): https://jellyfish.co/platform/jellyfish-ai-impact/
7. [vendor] Jellyfish Copilot Dashboard (acceptance rate by language, users-vs-non-users cycle-time framing, 34% claim): https://jellyfish.co/platform/jellyfish-copilot/
8. [vendor] Jellyfish — Measure AI Impact: Copilot, Cursor, Gemini, Sourcegraph (25%/12%/17% claims; 300 companies/20k engineers): https://jellyfish.co/blog/measure-ai-impact-copilot-cursor-gemini-sourcegraph/
9. [practitioner] Adam Ferrari (Jellyfish SVP Eng) — Quantifying AI Coding Impact (acceptance-rate critique incl. Windsurf quote): https://adamferrari.substack.com/p/quantifying-ai-coding-impact

**LinearB**
10. [vendor] LinearB — AI Measurement Framework (adoption/impact split, 4 impact metrics, no-hours-saved ROI stance): https://linearb.io/blog/ai-measurement-framework
11. [vendor] LinearB — AI Metrics: How to Measure Gen AI Code (gitStream labeling mechanics, benefits/risks dashboard, baseline comparison): https://linearb.io/blog/AI-metrics-how-to-measure-gen-ai-code
12. [vendor] LinearB — Measure Generative AI Impact (GenAI PR labeling + org baseline comparison): https://linearb.io/blog/measure-generative-ai-impact

**Swarmia**
13. [vendor] Swarmia docs — AI tool detection & filters (exact detection rules, 24h heuristic, over-reporting admission, confidence tiers): https://help.swarmia.com/features/ai-tools/ai-tool-detection-and-filters.md
14. [vendor] Swarmia docs — AI impact (PR-metric comparison set, tool/mode grouping, double-counting caveat): https://help.swarmia.com/features/ai-tools/ai-impact.md
15. [vendor] Swarmia docs — Cloud agents (agent PRs, merge %, agent commit %, batch size): https://help.swarmia.com/features/ai-tools/cloud-agents.md
16. [vendor] Swarmia guide — Understand the impact of AI tools (six caveats incl. self-selection; anti-vendor-stat warning): https://help.swarmia.com/guides/understand-the-impact-of-ai-tools.md
17. [vendor] Swarmia — AI impact product page: https://www.swarmia.com/product/ai-impact/

**Uplevel**
18. [vendor] Uplevel — SEE product (deep work, FlowState, team-level privacy stance, AI Impact & ROI feature): https://uplevelteam.com/product/see
19. [vendor] Uplevel Data Labs — Gen AI for Coding report landing (800 devs; metrics incl. "incidence of PRs with bugs"; report gated): https://resources.uplevelteam.com/gen-ai-for-coding
20. [independent] Visual Studio Magazine coverage of Uplevel study (41% bug increase, no throughput gain) — fetch blocked (403); figure corroborated in multiple secondary writeups: https://visualstudiomagazine.com/articles/2024/09/17/another-report-weighs-in-on-github-copilot-dev-productivity.aspx (partially verified)

**Faros AI**
21. [vendor] Faros — Copilot module / AI impact analysis (adoption + velocity/quality metric lists, cohort/before-after/causal claims, team-level granularity): https://www.faros.ai/copilot-module
22. [vendor] Faros — AI Engineering Report 2026 "The Acceleration Whiplash" (22k devs, 4k teams telemetry; +66% epics/dev, +54% bugs/dev, 5x review time, +31% unreviewed merges): https://www.faros.ai/research/ai-acceleration-whiplash · PDF: https://pages.faros.ai/hubfs/AI_Engineering_Report_2026_The_Acceleration_Whiplash_Faros.pdf
23. [vendor] Faros docs — AI Copilot Evaluation Module (power user = 15 days/30 hours per month) — page 403'd; definition retrieved via search snippet (partially verified): https://docs.faros.ai/docs/ai-copilot-evaluation-module
24. [independent, low confidence] RockB — Faros AI Review 2026 (five-layer model, $37.50/incremental-PR ROI example): https://baeseokjae.github.io/posts/faros-ai-coding-analytics-guide-2026/
25. [independent] ADTmag — "More Code, More Bugs" on Faros report: https://adtmag.com/articles/2026/04/22/more-code-more-bugs.aspx

**Athenian**
26. [independent] TechCrunch (2022) — Athenian team-only philosophy, Eiso Kant quotes: https://techcrunch.com/2022/03/02/athenian-gives-you-metrics-about-your-engineering-team-without-focusing-on-individuals/
27. [independent] Crunchbase — Athenian acquired by The Linux Foundation, Nov 2023 (via search result; unverified page fetch): https://www.crunchbase.com/organization/athenian

**Waydev**
28. [vendor] Waydev — Measure GitHub Copilot Impact (before/after coding time, commit frequency, active coding days, onboarding timing): https://waydev.co/measure-github-copilot-impact-waydev/
29. [vendor] Waydev docs — GitHub Copilot integration (Copilot API limits: no per-user usage data; IDE-telemetry-only): https://docs.waydev.co/docs/github-copilot-integration
