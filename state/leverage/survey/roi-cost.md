# Per-Dollar / ROI Framing — FinOps for AI, Cost-per-Output Metrics, and the Spend-Denominator Question

Sub-investigator report. Window researched: 2025-04 → 2026-07. All external claims cited and tagged; my own reasoning labeled **inference**.

---

## 1. What it measures & theory

### 1.1 The two competing framings of AI spend (they point in opposite directions)

The industry currently holds **two mutually contradictory theories** of what an engineer's AI spend means:

**A. Spend-as-investment ("leverage"):** high spend = high adoption = good.
- Nvidia CEO Jensen Huang: "If that $500,000 engineer did not consume at least $250,000 worth of tokens, I am going to be deeply alarmed." Meta CTO Bosworth: "This is easy money. No limit," claiming his best engineer spending salary-equivalent in tokens is "5x to 10x more productive" (no data substantiates this). Databricks CEO had the engineering department applaud an engineer who spent $7,000 in tokens in two weeks. Sendbird ranks token spenders "Beginner" → "AI God" (100M+ tokens/day). ([Forbes, Mar 31 2026](https://www.forbes.com/sites/richardnieva/2026/03/31/the-ai-gods-spending-as-much-as-they-can-on-ai-tokens/) [independent])
- Practitioner steelman: ElevateX CTO Ralf Gehrer argues token spend per engineer is a legitimate leadership KPI *paired with output* — "token ROI" targets of 2.5x–6x, even in performance reviews. ([ElevateX, May 6 2026](https://elevatex.de/blog/ai/token-spend-engineering-kpi/) [practitioner]) SaaSRise argues spend is "the cheapest leverage you'll ever buy," a rounding error vs. salary. ([SaaSRise, Jul 7 2026](https://www.saasrise.com/blog/should-you-token-max-what-to-spend-per-software-engineer-on-ai-coding-tools) [practitioner])

**B. Spend-as-cost (unit economics):** dollars are a denominator; efficiency = output per dollar.
- **FinOps Foundation** "FinOps for AI" (updated Feb 17 2026): unit-cost metrics — cost per inference, cost per token, cost per API call, training-cost efficiency; **showback** (visibility without charging) to nudge cost-conscious behavior. Critically, the Foundation recommends allocation at **workload / project / team level via tagging — it does NOT recommend per-person efficiency metrics.** ([finops.org](https://www.finops.org/wg/finops-for-ai-overview/) [primary])
- **Vantage**: "Cost per PR = Monthly AI spend / PRs merged" per developer; alternative denominators tickets/story points/deploys; sells a "Unit Costs" automation feature. Their own illustrative table shows the *highest* spender ($1,200/mo) having near-lowest cost-per-PR — i.e., even the vendor selling the ratio argues high spenders are the efficient ones. Data is entirely hypothetical. ([Vantage, 2026](https://www.vantage.sh/blog/agentic-coding-efficiency) [vendor])
- **Olakai** (CFO framing): (1) **cost-per-outcome** ("fully-loaded dollar cost of each unit of value produced" — per merged PR, per deployed feature); (2) **spend run-rate forecast** (spend grows exponentially, not linearly); (3) **value leak rate** = share of AI spend disconnected from committed outputs — with an explicit caveat to "track patterns, not flag individual sessions, to avoid penalizing exploratory work." Recommends per-engineer and per-team tracking for baselines. ([Olakai, Jun 17 2026](https://olakai.ai/blog/token-cost-metrics-cfo/) [vendor])
- **Faros AI "Token Intelligence"** (launched Jun 17 2026): classifies each token as **productive / inefficient / wasteful** based on the *quality of the session that consumed it* (wasteful = agent loops on abandoned paths, redundant context, expensive models on simple tasks). Ranks **teams, not individuals**; each team sees own spend vs. baseline. ([Faros](https://www.faros.ai/blog/token-intelligence-for-ai-engineering) [vendor])
- **DX**: cost is one of three dimensions (Utilization / Impact / Cost), never a standalone per-person score; ROI = (business value − total cost)/total cost computed at org level. ([DX pricing guide, Jun 12 2026](https://getdx.com/blog/ai-coding-assistant-pricing/); [DX ROI, May 21 2026](https://getdx.com/blog/ai-roi-engineering/) [vendor])
- **Gartner** (press release, Jun 24 2026; body 403'd to fetch, details confirmed via The Register): predicts **AI coding costs surpass the average developer's salary by 2028**; ~25% of tech leaders already spend $200–$500/dev/month, ~6% >$2,000/dev/month; recommends governance (model routing, context engineering, token-usage reviews) — cost-to-value tracking at org level, not individual scoring. ([Gartner](https://www.gartner.com/en/newsroom/press-releases/2026-06-24-gartner-predicts-ai-coding-costs-will-surpass-average-developer-salary-by-2028-as-token-consumption-surges) [primary]; [The Register](https://www.theregister.com/ai-and-ml/2026/06/24/ai-coding-agents-could-soon-cost-more-than-the-developers-using-them/5260864) [independent])

**Inference:** the client's current metric (output ÷ AI dollars, percentile-ranked per person) sits in framing B pushed one step further than any published framework goes — every credible FinOps/vendor source stops at team-level unit costs or explicitly warns against individual session-level penalization.

### 1.2 Did any real org percentile-rank engineers on output-per-AI-dollar?

**No documented case found (searched extensively; unverified that none exists privately).** What exists in the wild is the *inverse* — individual **spend-maximization** rankings — and they all failed fast:

- **Meta "Claudeonomics"**: internal leaderboard ranking token consumption across 85,000+ staff; 60T tokens/30d; titles "Token Legend," "Cache Wizard." Employees left agents running idle solely to inflate rank. Shut down ~April 2026 per later coverage. ([The Decoder, reporting The Information](https://the-decoder.com/meta-employees-compete-for-token-consumption-on-an-internal-ai-leaderboard/) [independent press, secondhand]; shutdown date via [ElevateX](https://elevatex.de/blog/ai/token-spend-engineering-kpi/) [practitioner] — **shutdown date unverified against a primary source**)
- **Amazon "KiroRank"**: leaderboard of AI/token usage on Kiro + internal agent tooling. Employees fed trivial/fabricated tasks to climb rankings ("tokenmaxxing" was Amazon's internal term); infra costs rose; Amazon stated tokens would not inform performance reviews but employees didn't believe it. Scrapped; SVP Dave Treadwell: "Please don't use AI just for the sake of using AI." Amazon pivoted to tracking **"normalized deployments"** — output-shaped, not spend-shaped. ([Fortune, Jul 7 2026](https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/) [independent]; [MLQ aggregation](https://mlq.ai/news/amazon-scraps-internal-ai-leaderboard-after-employees-gamed-it-with-fake-tasks/) [independent press aggregation — original outlet unverified])
- **Sendbird**: ongoing spend leaderboard; CEO John Kim concedes only **8 of the top 10 spenders** show consistent mega-productivity and himself compares the exercise to 1990s lines-of-code metrics. ([Forbes](https://www.forbes.com/sites/richardnieva/2026/03/31/the-ai-gods-spending-as-much-as-they-can-on-ai-tokens/) [independent])
- **Uber**: burned its entire 2026 AI budget in ~4 months; capped employee token spend at $1,500/mo. **Microsoft** revoked Claude Code licenses after budget exhaustion. **Priceline** imposed token limits. ([Fortune] [independent]; [TechCrunch, Jun 5 2026](https://techcrunch.com/2026/06/05/the-token-bill-comes-due-inside-the-industry-scramble-to-manage-ais-runaway-costs/) [independent])
- Cognition CEO Scott Wu, on the whole trend: "People are like, 'We rank our engineers by how many tokens they're spending.' Well, let's try and rank people by how much output they're actually producing." ([Fortune] [independent])

**Inference (load-bearing):** the only at-scale natural experiments with individual spend-linked rankings ran in the spend-maximizing direction, were gamed within months by fabricated agent activity, and were scrapped. Goodhart pressure is symmetric: a spend-*minimizing* percentile rank invites the mirror gaming (spend leakage, underuse, cheap-model theater — §4). The client's metric is the untested mirror image of a design that just publicly failed.

---

## 2. Evidence quality

### 2.1 Does higher spend correlate with harder work or with waste? (the core empirical question)

**Best individual-level dataset — Jellyfish** (vendor telemetry; 12,000 developers, 200 companies, Q1 2026):
- Median user ~51M tokens/mo; 90th percentile ~380M (~7.5x median); 75th pct spend ≈ $227/mo, 90th ≈ $691/mo.
- Bottom-20% token spenders: 11 merged PRs/quarter at ~$3 → **$0.28/PR**. Top-20%: 23 merged PRs at ~$1,822 → **$89.32/PR**.
- Headline: heavy users are **~2x as productive on merged PRs but consume ~10x the tokens** — strongly diminishing marginal PRs per dollar; a ~300x spread in cost-per-PR.
- Jellyfish's own recommendation: "Broad, moderate adoption delivers far more value than narrow, extreme usage" — best ROI is moving the middle of the adoption curve up, **not** pushing heavy users to spend less. ([Jellyfish, Apr 15 2026](https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/) [vendor])
- **Inference:** this single dataset supports BOTH sides — spend does correlate with real output (heavy users produce 2x), AND a per-dollar ratio mechanically ranks the most productive engineers last. It also confirms the brief's problem #3 (right-skewed, poorly discriminating per-dollar distribution) at industry scale.

**Best independent/academic evidence — "How Do AI Agents Spend Your Money?"** (Bai, Mihalcea, Brynjolfsson et al.; UMich/Stanford/Microsoft; arXiv 2604.22750, submitted Apr 24 2026; 8 frontier LLMs on SWE-bench Verified):
- **Human-expert-rated task difficulty only weakly aligns with actual token cost.**
- Runs on the *identical task* differ by up to **30x** in total tokens.
- **Higher token usage does not yield higher accuracy** — accuracy peaks at intermediate cost and saturates.
- Agentic tasks are ~1000x more token-expensive than chat; **input tokens dominate cost** (context size, not output volume).
- Models can't predict their own costs (correlation ≤0.39, systematic underestimation). ([arXiv](https://arxiv.org/abs/2604.22750) [independent])
- **Inference (critical for the denominator debate):** the "higher spend = harder work" defense AND the "higher spend = waste" attack are both only weakly true at task level. Spend is a *noisy* signal of difficulty and of effectiveness; a large share of an individual's monthly cost is run-to-run stochasticity plus repo context size — neither is engineer skill.

**Org-level — DX** (vendor; 400+ orgs, 14 months): AI usage +65% while median PR throughput +7.76% (90th pct 43.9%); "no linear correlation between increased investment and proportional output gains"; coding is only ~14% of developer time so bottlenecks shift to review. ([DX](https://getdx.com/blog/ai-coding-assistant-pricing/), [DX ROI](https://getdx.com/blog/ai-roi-engineering/) [vendor])

**Team-level counterpoint — Weave data** (via Ojstersek's Engineering Leadership newsletter, Jun 15 2026; 10,000+ engineers): top-1% teams spend MORE on tokens yet have LOWER cost per unit of output (output measured by an ML "expert-hours" model). ([newsletter.eng-leadership.com](https://newsletter.eng-leadership.com/p/what-the-top-1-of-engineering-teams) [vendor data via practitioner])

**Waste-side — Entelligence** (vendor selling review agents — heavy conflict of interest; 1M+ PRs across 2,444 orgs, Feb 16–May 4 2026): of every $1 of AI token spend, $0.44 goes to reactive bug-fixing, $0.27 to rework/churn, $0.11 to review friction; only **$0.18 reaches shipped product**. 25% of AI lines discarded within the sprint; revert rate grew 3.7x vs 2.6x PR-volume growth. No pre-AI baseline; treat magnitudes as directional only. ([research.entelligence.ai](https://research.entelligence.ai/) [vendor]; secondary: [TechStartups, Jul 6 2026](https://techstartups.com/2026/07/06/for-every-1-spent-on-ai-companies-pay-0-44-fixing-bugs-0-27-rewriting-code-and-0-11-on-review-delays-study-finds/) [independent press])

**Faros AI** (vendor; 22,000 devs / 4,000 teams): throughput up but "bugs, incidents, and rework are rising faster"; CTO anecdote via Faros CEO: "One of my engineers spent $40,000 on tokens last month, and I don't know" whether to stop him or clone him. ([Faros](https://www.faros.ai/blog/token-intelligence-for-ai-engineering) [vendor]; [TechCrunch] [independent])

### 2.2 Overall quality assessment

- The per-dollar space is dominated by **vendor content marketing** (Vantage, Olakai, Jellyfish, DX, Faros, Entelligence all sell the dashboards that would compute these ratios). Only ONE peer-review-track independent study (arXiv 2604.22750) touches spend-vs-difficulty directly.
- The tokenmaxxing failures are well-triangulated (Fortune + Forbes + The Information via Decoder + multiple aggregators + named executives on record).
- **No study of any kind evaluates ranking individuals on output-per-AI-dollar** — the client's exact design is empirically untested everywhere. **unverified/absent evidence, stated plainly.**
- Cost figures are highly non-stationary: token prices fell ~90% since 2023 while spend rose ([Fortune]); Copilot moved to consumption "AI Credits" June 1 2026 with promotional credits masking baselines ([DX]); current pricing is arguably subsidized ([Guisinger] [practitioner]). Dollar denominators are not comparable across quarters or tools.

---

## 3. Known failure modes (of per-dollar / ROI metrics generally)

1. **Ranking inversion:** cost-per-PR ranks the demonstrably most productive engineers (2x output) at the bottom because their denominator is 10x (Jellyfish). The metric anti-correlates with the behavior (deep agentic adoption) most orgs say they want.
2. **Rewards non-adoption:** the cheapest way to a great ratio is barely using AI. Directly fights Shopify-style "reflexive AI usage is a baseline expectation" mandates (Lütke memo, Apr 2025 — [Forbes coverage](https://www.forbes.com/sites/douglaslaney/2025/04/09/selling-ai-strategy-to-employees-shopify-ceos-manifesto/) [independent]) and DORA's amplifier framing.
3. **Noise swamps signal:** 30x same-task token variance + weak difficulty↔cost alignment (arXiv) means one month's individual spend is substantially luck + repo context size. Percentiles computed on that are unstable. **inference from cited variance**
4. **Non-stationary denominator:** price cuts, credit promotions, model-mix shifts, and vendor subsidy withdrawal change $ values ~quarterly; scores aren't comparable over time (Fortune, DX, Guisinger).
5. **Wrong cost term:** downstream rework dwarfs the token bill (Entelligence: 82% of AI-dollar value lost downstream) — optimizing the token denominator optimizes the small number. [vendor, directional]
6. **Session-blind aggregation:** naive monthly $ totals conflate genuinely wasteful patterns (idle loops, model overkill) with legitimate exploration and big-context work; Faros needed session-level classification to separate them; Olakai explicitly warns against flagging individual sessions.
7. **Leaderboard dynamics (either direction) get gamed within months** — Meta, Amazon documented; both scrapped tracking (Fortune). Even non-review "informational" rankings weren't believed to be consequence-free by employees (Amazon).
8. **ROI theater at org level:** BCG survey via Fortune — 42% of workers report ~8h/week saved, 66% got no guidance on reinvesting it, 50% didn't spend it on anything strategic. Time "saved" ≠ value captured.

---

## 4. Gameability & who it penalizes (per metric)

### 4.1 Output ÷ AI-dollars, percentile-ranked per person (the client's design)
- **How gamed:** (a) **spend leakage** — route heavy work through personal/free accounts, unmetered chat tools, or a teammate's session, driving the measured denominator toward the $5 floor while output stays attributed; (b) always pick the cheapest model regardless of fit (quality cost invisible — brief has no defect linkage); (c) slice work into many trivial merged PRs (numerator inflation, already noted in brief); (d) prompt-frugality theater — minimize context to cut input tokens, degrading results invisibly; (e) simply underuse AI and hand-write code — ratio soars; (f) time spend and PRs across window boundaries around the 4-week window and cold-start floors.
- **Who it unfairly penalizes:** agent-platform/infra engineers (spend with no PR-shaped output — the brief's own validated problem); ML/data engineers with expensive experimentation loops; engineers in large-context monorepos (input tokens dominate cost — arXiv); anyone assigned genuinely hard or exploratory work (agent discovery cost ≠ human difficulty); conscientious users of expensive models for correctness-critical code; and the org's heaviest, demonstrably 2x-productive adopters (Jellyfish).

### 4.2 Spend-maximization leaderboards (Meta/Amazon/Sendbird pattern)
- **Gamed by:** idle agents, bot loops, trivial/fabricated tasks purely to burn tokens (documented at Meta and Amazon; Amazon's own term "tokenmaxxing").
- **Penalizes:** efficient engineers, selective senior users, cheap-model discipline; plus real budget damage (Uber, Microsoft).

### 4.3 Cost-per-outcome / value-leak-rate (Olakai/Vantage/Faros style)
- **Gamed by:** attaching trivial committed outputs to every session so no spend looks "disconnected"; redefining the outcome unit (tickets vs PRs vs deploys) to whichever flatters; batching.
- **Penalizes:** exploratory/research/prototyping work and learning time (Olakai itself flags this); non-PR roles again.

### 4.4 Token-ROI targets in reviews (ElevateX's 2.5–6x proposal)
- **Gamed by:** inflating self-estimated "value generated"; choosing tasks with legible value.
- **Penalizes:** glue-work, mentoring-heavy, platform, and incident-response roles whose value isn't unit-priced. **inference**

---

## 5. Applicability to our constraints (HAVE / CANNOT mapping)

**What maps onto data we HAVE:**
- Per-user/day/tool dollars, tokens, sessions, active days across **ALL** tools incl. the agent platform → spend is actually the org's *best-covered* signal. It can power: adoption/engagement features (active days, tool breadth, session regularity, agentic-vs-chat mix), a **self-view FinOps showback panel** (own spend trend, own cost-per-merged-PR vs *own* trailing baseline — never vs peers), and team/function-level unit-cost aggregates (the only level any credible framework endorses).
- Merged PRs + lines + CI signal → cost-per-merged-PR is computable today, but inherits brief problems #2/#3/#5 verbatim, now confirmed externally: 300x cost-per-PR spread (Jellyfish) guarantees a right-skewed, weakly discriminating ratio; agent-platform engineers have denominator-only records.
- Role/level/team → if any ratio survives, cohort by **function** (Jellyfish/Gartner spend norms differ hugely by work type); the current level-only cohort is indefensible against this evidence.

**What the CANNOT list kills:**
- No task difficulty/value → cannot difficulty-adjust spend, and the arXiv result says spend itself is NOT a usable difficulty proxy. Dead end for "per-dollar as fairness-adjusted efficiency," stated plainly.
- No agent-platform outputs → any spend-in-denominator score structurally zeros out agent builders; Faros/Olakai "waste classification" needs session-level output linkage we lack.
- No post-merge defect linkage → cheap-model quality degradation is invisible; the metric can't see the Entelligence-style 82% downstream cost it would incentivize.
- Privacy/self-view-only → percentile leaderboards on spend are disqualified BY DESIGN here — and §1.2 shows even internal, "no-consequences" spend rankings were disbelieved and gamed.
- ~2-week data lag → run-rate forecasts and rolling 8–12-week self-trends fine; anything reactive/real-time is not.

**Bottom-line recommendation for the orchestrator (inference, evidence-backed):** dollars should move OUT of the denominator of any individual percentile score. Defensible uses of the spend data: (1) adoption/engagement/breadth signals (numerator-side context, capped/log-scaled); (2) a private self-view unit-cost trend panel (FinOps showback pattern, own-baseline only); (3) team/function-level cost-per-outcome for org stewardship. If leadership insists on an individual efficiency ratio: log-dollars, winsorized, function-cohorted, own-trend-referenced, never percentile-ranked across people — and label it "cost awareness," not "leverage."

---

## 6. Sources (annotated, tagged)

| # | Source | Tag | Date | Why it matters | Status |
|---|--------|-----|------|----------------|--------|
| 1 | [FinOps Foundation — FinOps for AI Overview](https://www.finops.org/wg/finops-for-ai-overview/) | [primary] | upd. 2026-02-17 | Canonical AI unit-economics framework; team/workload allocation + showback; **no per-person efficiency metrics** | fetched |
| 2 | [Jellyfish — Is tokenmaxxing cost-effective?](https://jellyfish.co/blog/is-tokenmaxxing-cost-effective-new-data-from-jellyfish-explains/) | [vendor] | 2026-04-15 | 12k devs/200 cos: heavy users 2x PRs, 10x tokens; $0.28 vs $89.32 cost/PR; "move the middle" | fetched |
| 3 | [Bai, Mihalcea, Brynjolfsson et al. — How Do AI Agents Spend Your Money? (arXiv 2604.22750)](https://arxiv.org/abs/2604.22750) | [independent] | 2026-04-24 | Token cost weakly tracks expert-rated difficulty; 30x same-task variance; accuracy peaks at intermediate cost | fetched (abstract) |
| 4 | [Fortune — Cognition CEO: tech got 'carried away' with token leaderboards](https://fortune.com/2026/07/07/cognition-ceo-tokenmaxxing-big-tech-carried-away-ai-productivity-gains/) | [independent] | 2026-07-07 | Scott Wu "rank by output not tokens"; Meta/Amazon leaderboards gamed & scrapped; Uber $1,500/mo cap | fetched |
| 5 | [Forbes (Nieva) — The 'AI Gods' Spending As Much As They Can](https://www.forbes.com/sites/richardnieva/2026/03/31/the-ai-gods-spending-as-much-as-they-can-on-ai-tokens/) | [independent] | 2026-03-31 | Databricks/Sendbird/Nvidia/Meta spend-celebration; Huang $250k quote; Sendbird 8-of-10 admission | fetched |
| 6 | [The Decoder — Meta 'Claudeonomics' leaderboard](https://the-decoder.com/meta-employees-compete-for-token-consumption-on-an-internal-ai-leaderboard/) | [independent, secondhand of The Information] | 2026-04 | 85k staff, 60T tokens/30d; idle agents run to inflate rank | fetched |
| 7 | [MLQ News — Amazon scraps KiroRank](https://mlq.ai/news/amazon-scraps-internal-ai-leaderboard-after-employees-gamed-it-with-fake-tasks/) | [independent aggregation; original outlet unverified] | 2026-05 | Gaming via fabricated tasks; Treadwell quote; pivot to "normalized deployments" | search-corroborated (multiple outlets) |
| 8 | [Vantage — Your Most Expensive Developer Might Be Your Most Efficient](https://www.vantage.sh/blog/agentic-coding-efficiency) | [vendor] | 2026 | Cost-per-PR formula per developer; hypothetical data only; sells Unit Costs feature | fetched |
| 9 | [Olakai — 3 Token Cost Metrics Every CFO Should Watch](https://olakai.ai/blog/token-cost-metrics-cfo/) | [vendor] | 2026-06-17 | Cost-per-outcome, run-rate forecast, value-leak-rate + anti-penalization caveat | fetched |
| 10 | [Faros AI — Token Intelligence](https://www.faros.ai/blog/token-intelligence-for-ai-engineering) | [vendor] | 2026-06-17 | Productive/inefficient/wasteful token classification; **team-level** ranking; 22k-dev report: bugs rising faster than throughput | fetched |
| 11 | [DX — AI coding assistant pricing & ROI guide](https://getdx.com/blog/ai-coding-assistant-pricing/) | [vendor] | 2026-06-12 | $200–600/dev/mo; median +7.76% PR throughput; "no linear correlation" spend→output | fetched |
| 12 | [DX — Why AI ROI is lower than expected](https://getdx.com/blog/ai-roi-engineering/) | [vendor] | 2026-05-21 | ROI formula at org level; coding = 14% of dev time; hidden costs | fetched |
| 13 | [Entelligence Research — Token Maxxing Is Making Teams Slower](https://research.entelligence.ai/) | [vendor, strong COI] | 2026 (Feb–May data) | $1 → $0.18 shipped; $0.44 bugs / $0.27 rework / $0.11 review friction; 1M+ PRs, 2,444 orgs; no pre-AI baseline | fetched |
| 14 | [TechCrunch — The token bill comes due](https://techcrunch.com/2026/06/05/the-token-bill-comes-due-inside-the-industry-scramble-to-manage-ais-runaway-costs/) | [independent] | 2026-06-05 | Spend-management vendor landscape (Pay-i, Ramp, Datadog…); Uber/Microsoft/Priceline caps; $40k/mo engineer anecdote; 18.6x per-dev consumption growth in 9 months | fetched |
| 15 | [Gartner press release — AI coding costs surpass dev salary by 2028](https://www.gartner.com/en/newsroom/press-releases/2026-06-24-gartner-predicts-ai-coding-costs-will-surpass-average-developer-salary-by-2028-as-token-consumption-surges) + [The Register](https://www.theregister.com/ai-and-ml/2026/06/24/ai-coding-agents-could-soon-cost-more-than-the-developers-using-them/5260864) | [primary + independent] | 2026-06-24 | Spend trajectory; 25% of orgs at $200–500/dev/mo; governance-not-scoring guidance | primary 403'd; verified via secondaries |
| 16 | [Dan Guisinger — Tokenmaxxing: Why AI Token Leaderboards Are a Terrible Idea](https://danguisinger.com/blog/tokenmaxxing-why-ai-token-leaderboards-are-a-terrible-idea/) | [practitioner] | 2026-05-28 | Goodhart framing; explicitly rejects output-per-dollar too — argues for outcome metrics + budgets, not targets; subsidized-pricing warning | fetched |
| 17 | [Ralf Gehrer / ElevateX — Token Spend Is the New Engineering KPI](https://elevatex.de/blog/ai/token-spend-engineering-kpi/) | [practitioner, pro-spend steelman] | 2026-05-06 | Only found advocacy of per-person "token ROI" (2.5–6x) in reviews; concedes "token volume is an input metric, never proof of output quality" | fetched |
| 18 | [Addy Osmani — The reality of AI-assisted software engineering productivity](https://addyo.substack.com/p/the-reality-of-ai-assisted-software) | [practitioner] | 2025-08-16 | "AI leverage" as situational force multiplier; output≠outcome; seniority effects; review-bottleneck data | fetched |
| 19 | [Ojstersek / Weave — What the Top 1% of Engineering Teams Do Differently](https://newsletter.eng-leadership.com/p/what-the-top-1-of-engineering-teams) | [practitioner w/ vendor data] | 2026-06-15 | Top teams spend more AND get more output per dollar (team level); ML expert-hours output model | fetched |
| 20 | [Forbes (MSV) — Why Your Engineers' Favorite AI Tools Are Wrecking Your 2026 Budget](https://www.forbes.com/sites/janakirammsv/2026/05/26/why-your-engineers-favorite-ai-tools-are-wrecking-your-2026-budget/) | [independent] | 2026-05-26 | "Blunt per-engineer ceilings are a starting point but a poor finish"; layered team-level controls; cost-per-merged-change as board metric | fetched |
| 21 | [SaaSRise (Allis) — Should You Token Max?](https://www.saasrise.com/blog/should-you-token-max-what-to-spend-per-software-engineer-on-ai-coding-tools) | [practitioner, anecdotal] | 2026-07-07 | "Cheapest leverage" argument; $200–800/mo norms; explicitly does NOT measure output/dollar per engineer | fetched |
| 22 | [Shopify Lütke memo coverage (Forbes)](https://www.forbes.com/sites/douglaslaney/2025/04/09/selling-ai-strategy-to-employees-shopify-ceos-manifesto/) | [independent coverage of primary memo] | 2025-04-09 | AI usage as baseline expectation + in performance reviews — the adoption-mandate pole a spend-denominator fights against | search-verified |

Cross-checks performed: Jellyfish 2x/10x claim traced from secondary mentions to the primary Jellyfish post (numbers confirmed and extended); Nvidia/Huang quote confirmed in Forbes primary reporting; Meta leaderboard confirmed across The Decoder (citing The Information) + Fortune + ElevateX; Amazon KiroRank corroborated across ≥5 outlets but original reporting outlet not identified (flagged); Entelligence numbers confirmed against its own research site (COI flagged); Gartner via press-release title + two independent secondaries.
