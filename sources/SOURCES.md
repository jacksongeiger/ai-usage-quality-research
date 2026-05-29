# SOURCES — Annotated Bibliography

Credibility tags: **Primary** (original source / first-party data), **Vendor** (commercial party with an interest), **Independent** (third-party/academic with no stake), **Practitioner** (authoritative working-engineer commentary). Numbers and quotes were extracted from fetched primary sources where possible; items not fetched from primary text are flagged **[snippet/secondary]** or **[unverified]**.

---

## A. Developer-productivity & DevEx frameworks

**[1] DORA — 2024 Accelerate State of DevOps Report.** DORA / Google Cloud, 2024. https://dora.dev/research/2024/dora-report/ · https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report — *Primary.* ~3,000 respondents. ~75.9% use AI for part of their job; 75% report productivity gains; **39% report little/no trust in AI-generated code.** Headline: a 25% increase in AI adoption associated with ~**−1.5% delivery throughput** and ~**−7.2% delivery stability**. Hypothesis: AI inflates batch sizes; the "productivity paradox" — individuals feel more productive while system throughput drops. Independent triangulation: RedMonk https://redmonk.com/rstephens/2024/11/26/dora2024/.

**[2] DORA — 2025 State of AI-assisted Software Development Report.** DORA / Google Cloud, 2025. https://dora.dev/dora-report-2025/ · https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report — *Primary, current.* ~90% use AI at work; 80%+ report productivity increases; ~30% still distrust AI code. AI now **positively** related to throughput AND product performance, but **still negatively related to delivery stability.** Central thesis: **"AI is an amplifier — it doesn't fix a team; it amplifies what's already there."** Introduces the **DORA AI Capabilities Model** (7 capabilities incl. clear AI policy, connect AI to internal context, foundational practices, strong safety nets, internal platform investment, user focus, healthy data ecosystems) and 7 team archetypes. Triangulation: RedMonk https://redmonk.com/rstephens/2025/12/18/dora2025/; Faros https://www.faros.ai/blog/key-takeaways-from-the-dora-report-2025.

**[3] The SPACE of Developer Productivity.** Forsgren, Storey, Maddila, Zimmermann, Houck, Butler. ACM Queue, 2021. https://queue.acm.org/detail.cfm?id=3454124 · MSR landing https://www.microsoft.com/en-us/research/publication/the-space-of-developer-productivity-theres-more-to-it-than-you-think/ — *Primary, foundational* (ACM full text 403'd to automated fetch; corroborated via MSR landing + multiple secondaries). Five dimensions: **S**atisfaction & well-being, **P**erformance, **A**ctivity, **C**ommunication & collaboration, **E**fficiency & flow. Core tenets: productivity "cannot be measured by a single metric or dimension"; measure a constellation of metrics *in tension*; draw from ≥2–3 dimensions; activity is "the most visible and most misused dimension"; always include a perceptual/survey dimension. **[some exact wordings snippet-level]**

**[4] DevEx: What Actually Drives Productivity.** Noda, Storey, Forsgren, Greiler. ACM Queue, 2023. https://queue.acm.org/detail.cfm?id=3595878 · https://getdx.com/research/devex-what-actually-drives-productivity/ · InfoQ https://www.infoq.com/articles/devex-metrics-framework/ — *Primary.* Three dimensions: **Feedback Loops, Cognitive Load, Flow State** — distilled from 25 sociotechnical factors. Mandates combining **perceptual** (developer surveys) + **workflow/system** measures; "break down results by team and persona" (role/tenure). Illustration: fast review turnaround can still *feel* disruptive — a good objective metric can hide a bad experience.

**[5] DevEx in Action.** Noda, Storey, Forsgren, Greiler. ACM Queue, 2024. https://queue.acm.org/detail.cfm?id=3639443 — *Primary follow-up* **[secondary mirrors only — effect sizes not directly verified].** Links improved DevEx to developer/team/org outcomes; reinforces survey-based perceptual measurement segmented by persona.

**[6] DX Core 4.** DX (Abi Noda et al.), 2024. https://getdx.com/dx-core-4/ · https://getdx.com/research/measuring-developer-productivity-with-the-dx-core-4/ — *Vendor (co-developed with SPACE/DevEx/DORA authors), tested across 300+ orgs.* Unifies DORA+SPACE+DevEx into **Speed, Effectiveness (the Developer Experience Index / DXI), Quality (change failure rate, failed-deployment recovery), Impact/Business**. Design principle: counterbalance speed/output with DXI to prevent gaming ("speed metrics used in isolation incite fear and counterproductive behaviors"). On individuals: "diffs per FTE is a useful signal when utilized carefully" with 3 preconditions — counterbalance with DXI, **no targets/rewards tied to it**, communicate to prevent misuse.

**[7] Measuring AI code assistants and agents / DX AI Measurement Framework.** DX, 2024–2025. https://getdx.com/research/measuring-ai-code-assistants-and-agents/ · https://getdx.com/blog/ai-measurement-hub/ — *Vendor; most explicit published AI-specific taxonomy.* Three dimensions: **Utilization** (DAU/WAU of AI tools, **% AI-assisted PRs**, **% AI-generated code committed**, adoption cohorts none/light/moderate/heavy), **Impact** (AI-driven **time savings/week**, per-tool CSAT survey, regression of AI cohorts on Core-4 outcomes, human-equivalent agent hours), **Cost** (spend/ROI). Flags **code-generation volume as "particularly susceptible to gaming."** Notes ~82% of devs use 3+ AI tools; even leading orgs ~60% active usage; AI-generated merged-code share ~24%→30.8% for daily users. Claims 3–12% efficiency gains, ~14% more R&D feature-dev time **[vendor-reported].**

**[8] Quantifying GitHub Copilot's Impact on Developer Productivity and Happiness.** Kalliamvakou et al., GitHub, 2022. https://github.blog/news-insights/research/research-quantifying-github-copilots-impact-on-developer-productivity-and-happiness/ — *Vendor RCT (conflict of interest).* n=95, JS HTTP-server task. Copilot group **55% faster** (1h11m vs 2h41m), **P=.0017, 95% CI [21%, 89%]** (wide). Survey n=2,000+: **73% better able to stay in flow; 87% preserved mental effort on repetitive tasks; >90% perceived faster completion.** Framed via SPACE.

**[9] Productivity Assessment of Neural Code Completion.** Ziegler et al., GitHub, arXiv 2022. https://arxiv.org/pdf/2205.06537 — *Vendor-affiliated, telemetry study.* Acceptance rate ~mid-20s% (21.2–23.5% reported; ~30% commonly cited). Key result: **perceived productivity correlates most strongly with acceptance rate** among telemetry metrics — i.e., acceptance is the best telemetry proxy for how productive devs *feel* (not necessarily are).

**[10] Copilot code retention/"survival".** GitHub docs + engineering artifacts. https://docs.github.com/en/copilot/concepts/copilot-usage-metrics/copilot-metrics · custom-models post on github.blog — *Vendor.* "Survival rate" (accepted code still present after a window) is a defined telemetry metric. Commonly cited: ~30% acceptance, **~88% of accepted code retained** **[vendor self-reported, circulates via secondary summaries — approximate].**

**[11] GitClear — Coding on Copilot: 2023 Data Shows Downward Pressure on Code Quality.** Bill Harding / GitClear, Jan 2024. https://www.gitclear.com/coding_on_copilot_data_shows_ais_downward_pressure_on_code_quality — *Vendor research, transparent diff-metadata methodology.* ~153M changed lines. Operation taxonomy (Added/Updated/Deleted/Moved/Copy-pasted) computed **from diff metadata, not semantic reading.** **Churn = lines reverted/updated within 2 weeks of authorship**, projected to ~double in 2024 vs 2021 baseline. "Moved" (refactoring proxy) declining.

**[12] GitClear — AI Copilot Code Quality: 2025 Research (2024 data).** GitClear, Feb 2025. https://www.gitclear.com/ai_assistant_code_quality_2025_research · PDF https://gitclear-public.s3.us-west-2.amazonaws.com/GitClear-AI-Copilot-Code-Quality-2025.pdf — *Vendor.* ~211M lines, 2020–2024. **Refactoring (% changed lines): ~25% (2021) → <10% (2024)**; **copy/pasted: 8.3% → 12.3%**; **churn: 5.5% → 7.9%** revised within 2 weeks; code clones up ~4x; first year copy/paste > moved code. *The canonical proof that meaningful quality signals derive from metadata alone.* **[magnitudes vendor-reported; direction credible].**

**[13] GitClear — 17 popular software-engineering metrics and how to game them.** GitClear, updated Jan 2025. https://www.gitclear.com/popular_software_engineering_metrics_and_how_they_are_gamed — *Vendor, concrete.* Gaming mechanics for LOC (padding), commits (micro-commits), story points (sandbag/cherry-pick), PR cycle time (force-push to fake authorship time), test coverage (assertion-free tests), change-failure-rate (redefine "critical").

**[14] McKinsey — Yes, you can measure software developer productivity.** McKinsey & Co., Aug 2023. — *Primary (the framework being critiqued).* ~20 companies; DORA/SPACE-derived plus output-leaning custom metrics (inner/outer-loop time, contribution analysis).

**[15] Measuring developer productivity? A response to McKinsey (Parts 1 & 2).** Gergely Orosz (Pragmatic Engineer) + Kent Beck, 2023. https://newsletter.pragmaticengineer.com/p/measuring-developer-productivity · part 2 …-part-2 · https://tidyfirst.substack.com/p/measuring-developer-productivity-440 — *Practitioner, authoritative.* 4 of 5 McKinsey metrics measure effort/output not outcomes; easily gamed; risk of leaking into reviews/layoffs ("a popular reason CTOs want to measure productivity is to identify who to fire"). Verdict: "wrong-headed and certain to backfire." Measure like sales — track **outcomes**, not output.

**[16] Thoughtworks Technology Radar — Complacency with AI-generated code.** Thoughtworks, 2024–2025. https://www.thoughtworks.com/en-us/radar/techniques/complacency-with-ai-generated-code — *Practitioner radar.* Flags "complacency with AI-generated code" and "AI-accelerated shadow IT" as anti-patterns; cites GitClear duplicate/churn rise + refactoring drop; tracks shift to "context engineering" and agentic systems.

**[17] METR — Measuring the Impact of Early-2025 AI on Experienced OSS Developer Productivity.** METR, Jul 2025. https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ · arXiv 2507.09089 · analysis https://simonwillison.net/2025/Jul/12/ai-open-source-productivity/ — *Independent RCT — strongest counter-evidence to vendor claims.* n=16 experienced devs, 246 real tasks in large mature repos (22k+ stars). AI (Cursor Pro + Claude 3.5/3.7) **increased completion time by 19%** (devs slower). **Perception gap: forecast +24% faster; after the study still believed +20% faster.** Authors stress it does NOT prove AI fails to help most devs; specific to this cohort/setting. Update: https://metr.org/blog/2026-02-24-uplift-update/.

**[18] Vendor delivery-analytics platforms — Faros AI, LinearB, Jellyfish, Swarmia.** https://www.faros.ai/blog/key-takeaways-from-the-dora-report-2025 · https://linearb.io/blog/is-github-copilot-worth-it · https://www.swarmia.com/blog/space-framework/ — *Vendor/commercial.* Track PR cycle time, commit volume, review latency, DORA metrics from metadata. Recurring admission: they **cannot distinguish AI-authored vs human-authored lines** — the central gap AI-measurement tools chase. ROI claims = marketing.

---

## B. AI-coding-tool vendor telemetry (usage-quality signals)

**[19] GitHub — Copilot usage metrics (data reference).** GitHub docs, current. https://docs.github.com/en/copilot/reference/copilot-usage-metrics/copilot-usage-metrics — *Vendor, canonical.* Exposes acceptance rate, code completions suggested/accepted, DAU/WAU/total active, **agent adoption & contribution (% lines by agents/28d)**, avg chat requests/user, **requests per chat mode (ask/edit/plan/agent)**, language/model usage, feature flags `used_cli/used_agent/used_chat/used_copilot_code_review`, breakdowns by IDE/feature/language/model, recency (`last_known_*_version`). LOC fields `loc_suggested_to_add/delete_sum`, `loc_added/deleted_sum`, `agent_edit`. **Teams with <5 seats are excluded** (small-team blind spot).

**[20] GitHub — Copilot Metrics REST API.** GitHub docs. https://docs.github.com/en/rest/copilot/copilot-metrics — *Vendor.* `total_active_users`, `total_engaged_users`, `copilot_ide_code_completions` (per language/editor/model, `is_custom_model`), `copilot_ide_chat` (`total_chats`, `total_chat_insertion_events`, `total_chat_copy_events`), `copilot_dotcom_chat`, `copilot_dotcom_pull_requests`. NOTE: legacy line-level `total_code_lines_suggested/accepted` came from the **deprecated** `copilot_usage` API and are **not** in the new top-level schema **[unverified for new API].**

**[21] GitHub — Interpreting usage/adoption metrics + changelog.** GitHub docs/blog, 2026. https://docs.github.com/en/copilot/reference/copilot-usage-metrics/interpret-copilot-metrics — *Vendor.* **Active user** = any activity; **Engaged user** = intentional action (accepted suggestion / used chat / generated PR summary). "Engaged > active" is the quality cut. Self-reported: ~30% acceptance, ~88% retention.

**[22] Cursor — Analytics API & Admin API.** Cursor/Anysphere docs. https://cursor.com/docs/account/teams/analytics-api · …/admin-api — *Vendor.* Agent edits (`total_suggested/accepted/rejected_diffs`, green/red line accepts), Tab autocomplete (`total_suggestions/accepts/rejects`), DAU family (`dau`, `cli_dau`, `cloud_agent_dau`, `bugbot_dau`), model usage, file-type breakdown, feature adoption (`intent` Write Code/Ask/Plan, MCP `tool_name`, `skill_name`), leaderboard ratios (`line_acceptance_ratio`, `accept_ratio`), per-call tokens/cost/model via `filtered-usage-events`. 90-day max window.

**[23] Anthropic — Claude Code Monitoring (OpenTelemetry).** Anthropic docs. https://code.claude.com/docs/en/monitoring-usage — *Vendor.* 8 metrics incl. `session.count`, `lines_of_code.count` (added/removed), `pull_request.count`, `commit.count`, `cost.usage` (attrs model/effort/speed/agent/skill), `token.usage` (input/output/cacheRead/cacheCreation), **`code_edit_tool.decision` (accept/reject by tool/language)**, `active_time.total` (excludes idle). Events: `user_prompt` (`prompt_length`), `tool_result` (`success`, `duration_ms`), `api_request` (`ttft_ms`), `tool_decision` (accept/reject + source), `skill_activated`, `mcp_server_connection`. **CRITICAL: telemetry OFF by default** — needs `CLAUDE_CODE_ENABLE_TELEMETRY=1` + an OTLP collector; user attribution needs authenticated OAuth.

**[24] AWS — Amazon Q Developer dashboard metrics + user-activity report.** AWS docs. https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/dashboard-metrics-descriptions.html — *Vendor.* `Active users`, `AI code lines` (accepted), `Acceptance rate` (inline), per-feature acceptance for `/dev` `/doc` `/test` `/review`, reference tracking. Hourly except active users (daily UTC).

**[25] Sourcegraph — Analytics (formerly Cody Analytics) + handbook.** Sourcegraph docs + handbook. https://sourcegraph.com/docs/analytics · handbook cody_analytics.md — *Vendor; best quality signal of any vendor.* **CAR** (Completion Acceptance Rate) = distinct accepted ÷ distinct suggested; **wCAR** (weighted) = accepted suggested chars ÷ total suggested chars. **Persistence/survival**: % of accepted completions unchanged/mostly-unchanged at **30 / 120 / 300 / 600 seconds**. Exports split by user/day/editor/language.

**[26] Tabnine — Usage Reports / Usage API.** Tabnine docs. https://docs.tabnine.com/main/administering-tabnine/managing-your-team/reporting — *Vendor* **[field-level JSON keys unverified from public docs].** Tracks lines in accepted completions, "useful chat interactions," per-tool metrics (agent/chat/completions/instruct), CSV one-row-per-user-per-day.

**[27] Windsurf (Codeium) — Analytics.** Windsurf docs. https://docs.windsurf.com/windsurf/accounts/analytics — *Vendor* **[exact API keys partly unverified].** Exposes **PCW (Percent of Code Written by AI)**, total LOC written, total tool calls, credit consumption, per-user patterns.

**[28] LeadDev — The rise and looming fall of acceptance rate.** LeadDev (editorial), current. https://leaddev.com — *Practitioner.* Argues acceptance rate is a vanity / tool-quality proxy, not productivity; pushes retention/revision-depth instead.

**[29] GitHub Blog / Accenture Copilot study.** Ya Gao & GitHub Customer Research, May 2024. https://github.blog — *Vendor research (favorable bias).* 30% acceptance, **88% character retention**, +8.69% PRs/dev, +15% merge rate, +84% successful builds, satisfaction 90–95% **[vendor-reported].**

---

## C. Measurement validity & ethics

**[30] Goodhart's Law.** Goodhart (1975); Strathern (1997). https://en.wikipedia.org/wiki/Goodhart%27s_law — *Reference* (Cambridge primary PDF TLS-failed; encyclopedic corroboration). Goodhart: "Any observed statistical regularity will tend to collapse once pressure is placed upon it for control purposes." Strathern: **"When a measure becomes a target, it ceases to be a good measure."** Documented gaming: h-index, hospital length-of-stay, test-target distortions.

**[31] Campbell's Law.** Donald T. Campbell, "Assessing the Impact of Planned Social Change," *Evaluation and Program Planning* 2(1), 1979. https://en.wikipedia.org/wiki/Campbell%27s_law — *Reference.* "The more any quantitative social indicator is used for social decision-making, the more subject it will be to corruption pressures and the more apt it will be to distort and corrupt the social processes it is intended to monitor." Education analog: when test scores become the goal they lose value as indicators.

**[32] Construct validity (measurement theory).** Cronbach & Meehl (1955); Campbell & Fiske (1959); Messick (1989). https://en.wikipedia.org/wiki/Construct_validity — *Reference.* Construct validity = how well indicators reflect a not-directly-measurable concept; **nomological network**; **convergent** validity (measures that should relate, do) and **discriminant** validity (measures that shouldn't relate, don't); multitrait-multimethod matrix; content & criterion validity subsumed under construct validity.

**[33] Measurement to Meaning: A Validity-Centered Framework for AI Evaluation.** Salaudeen, Reuel, Ahmed, et al. (MIT/Stanford/Cornell Tech), arXiv 2025. https://arxiv.org/html/2505.10573v3 — *Independent preprint.* "A construct is an abstract concept not directly measurable"; the inferential gap widens for constructs; benchmarks can "conflate genuine reasoning with domain knowledge or memorization" (analog: conflating usage *quality* with *volume*). "A claim about a construct… gains meaning and validity through its relationships with other constructs and observable measures."

**[34] Article 29 Working Party — Opinion 2/2017 on data processing at work.** WP29 (now EDPB), 2017. https://collab.dpa.gr/wp-content/uploads/2023/07/WP29_Opinion-2-2017-on-data-processing-at-work.pdf — *Primary regulatory* **[snippet-level summary].** Monitoring must satisfy **necessity, fairness, proportionality, subsidiarity**; **data minimization** (GDPR Art. 5(1)(c)); systematic monitoring triggers a **DPIA**; employees must get clear prior information. The legal floor for any individual telemetry.

**[35] Observer effect / Hawthorne effect.** General reference **[snippet-level]** (fs.blog/observer-effect; Hawthorne Works studies, 1920s). Measuring changes the measured behavior — introducing an AI-usage metric will itself shift AI-usage behavior.

**[36] Psychological safety + metrics.** Code Climate, "Using Data to Encourage Risk-Taking and Foster Psychological Safety" https://codeclimate.com/blog/using-data-psychological-safety **[snippet-level]**; Agile Analytics, "Psychological Safety Is an Engineering Metric." — *Vendor/practitioner.* "Be transparent, put data in context, and avoid using it to penalize"; invokes Campbell's Law directly.

**[37] Individual-variance / confounds in dev productivity.** arXiv 2104.13713 (referenced) — *Independent* **[snippet-level].** Individual variance accounts for most of the variance in developer output; tenure/seniority/role/repo/task type are heavy confounders. Reinforces DevEx "break down by persona."

---

## D. Metadata-signal derivation (cycle-time, churn, CI, review, normalization)

**[38] LinearB — Cycle Time & the 4 PR metrics.** LinearB docs/blog. https://linearb.helpdocs.io/article/0vif1ihmgc · https://linearb.io/blog/pull-request-pickup-time — *Vendor, de-facto standard.* 4-segment decomposition with timestamp boundaries: **Coding** (first commit → PR open), **Pickup** (PR open → first non-author review action), **Review** (first review action → merge), **Deploy** (merge → release).

**[39] Code Climate Velocity — Code Review metrics.** docs.velocity.codeclimate.com — *Vendor.* Independent definitions of review cycles, time-to-first-review, review coverage, "Comments Addressed / Influence," comment-size buckets. Cross-validates [38].

**[40] Swarmia — Cycle time / Investment balance.** help.swarmia.com · https://www.swarmia.com/blog/space-framework/ — *Vendor.* Median/p95 framing (long-tailed distributions); work-log contribution model (PRs, reviews, commits, comments).

**[41] GitClear — Diff Delta methodology & "Count LOC at your peril".** GitClear. https://www.gitclear.com — *Vendor, metadata-only.* Diff Delta `DD = Σ φ·⊖·⧉·β·τ·σ` — durability-weighted change measure; **τ time scalar** rewards code that is NOT churned. Churn = lines reverted/revised within 2 weeks; moved vs copy-paste from diff structure. Numbers in [11][12].

**[42] git-scm — git-revert.** https://git-scm.com/docs/git-revert — *Primary.* Revert detection from metadata/text-lite: subject "Revert …" / body "This reverts commit \<sha\>". Hotfix detection from branch/commit naming + short commit→deploy latency.

**[43] CI metrics — Harness / Gitar / AWS Well-Architected.** harness.io/blog · gitar.ai · AWS Well-Architected DevOps "Metrics for code review" https://docs.aws.amazon.com — *Vendor/Primary.* First-pass build success (>90% on default branch = healthy), build-failure-to-fix latency, review-friction definitions.

**[44] Flaky-build research.** arXiv "Understanding and Detecting Flaky Builds in GitHub Actions." — *Independent preprint* **[not peer-reviewed].** ~13% of build failures attributable to flakiness; 67.7% of rerun builds show flaky behavior; affects 51.3% of projects. Flaky build = passes on rerun with identical SHA.

**[45] SonarSource — Quality Gates.** docs.sonarsource.com — *Vendor/Primary.* Gate Pass/Fail status + issue/severity counts via API; "Sonar way for AI Code" gate. (Gate status only — no code.)

**[46] Jira/PM metrics.** LinearB "12 Jira metrics" · OBSS Jira analytics · reworkcost.com — *Vendor/practitioner.* Estimate accuracy (points vs actual), bug-to-feature ratio, rework = ticket moving backward in workflow, ticket cycle time.

**[47] PagerDuty — MTTR / incident linkage.** PagerDuty + DX PagerDuty connector docs. https://www.pagerduty.com — *Vendor.* Linking incident timestamps to deploy timestamps for change-failure-rate / MTTR.

**[48] Worklytics — Productivity Score Benchmarks (2025).** worklytics.co — *Vendor* **[single-vendor scheme; individual scoring ethically contested].** Concrete weighted composite (Output 40% / Collaboration 35% / Sustainability 25%) on 0–100; metadata-only; needs large N. Used only as an example of composite construction, not endorsed.

**[49] AI-code-survival research.** arXiv "Will It Survive? Deciphering the Fate of AI-Generated Code in Open Source." — *Independent preprint* **[not peer-reviewed].** Methodology for measuring survival of AI-generated code via line-survival over time.

---

## Coverage check
- Frameworks (DORA/SPACE/DevEx/DX): [1–7], [14–16] ✅
- Vendor telemetry: [8–10], [19–29] ✅
- Churn/rework/diff-metadata: [11–13], [41–42], [49] ✅
- Validity & ethics: [15], [30–37], [48] ✅
- Independent counter-evidence (triangulation against vendor optimism): [17] METR, [33] validity preprint, [37] confounds ✅
- Metadata-signal derivation: [38–48] ✅

**~34 substantive sources** spanning all required domains; vendor claims triangulated against independent/academic sources. Threshold (>25) met.
