# OPEN QUESTIONS

Priority: HIGH · MED · LOW. Resolved items kept for the record with resolution.

## HIGH (must be resolved before "done")
- [RESOLVED] H1 Can "AI usage quality" be validly measured at all without code, or is it irreducibly a code-quality question? → **Yes, partially.** It is measurable as *durable, low-friction leverage* via triangulated proxies (usage-depth telemetry + diff-metadata durability + self-report), but NOT as code correctness. Construct is honestly bounded in deliverable 01. (cycle 2)
- [RESOLVED] H2 What is the single most defensible signal of *quality* (not volume)? → **Retention/survival of accepted/AI-attributed code** (Sourcegraph exposes natively at 30–600s; reconstructable elsewhere from git line-survival). Volume and raw acceptance rate are explicitly down-weighted. (cycle 1–2)
- [RESOLVED] H3 How do signals become *tiered* recommendations without overclaiming? → Signal-agreement gating: High = ≥2–3 independent agreeing signals over ≥4 weeks; Medium = one strong or two weak; Exploratory = single pattern-match. Encoded in deliverable 03. (cycle 2–3)
- [RESOLVED] H4 How to keep this ethical/enablement given individual metrics are widely warned against? → Audience-lock to the engineer, opt-in, trend-vs-baseline (no peer rank), no comp/review linkage, perception-gap as coaching feature, GDPR-minimization floor. Deliverable 06. (cycle 1–3)

## MED
- [RESOLVED] M1 Is acceptance rate usable at all? → Only paired with a survival/retention signal; never alone (vendor-optimized, gameable). (cycle 1)
- [RESOLVED] M2 How to handle the ≥82%-use-3+-tools reality? → Normalize across each engineer's actual tool portfolio; report tool-fit, not single-tool loyalty. (cycle 2)
- [RESOLVED] M3 Can we proxy "knew when NOT to use AI"? → Only weakly (e.g., high-judgment tasks done with low AI + good outcomes), flagged as an Exploratory-tier inference and a known content-validity gap. (cycle 3)
- [OPEN-LOW-CARRY] M4 Exact Snowflake schema of the "some other" datasets is unknown. → Reasoned about likely additions (deploy/CD, tickets, security/gate status) and ranked them in deliverable 04; cannot fully resolve without schema access. Downgraded to LOW (does not block the analysis answer).

## LOW
- [RESOLVED] L1 Are Tabnine/Windsurf field names confirmed? → No, field-level unverified; flagged in SOURCES + metrics catalog. Does not affect framework logic. (cycle 1)
- [RESOLVED] L2 Is the Copilot ~88% retention figure primary? → Vendor self-reported via secondary summaries; cited with caveat; framework does not depend on the exact number. (cycle 1)
- [RESOLVED] L3 Are DevEx/SPACE "never rank individuals" exact prohibitions directly quotable? → Core tenets verified; some exact wordings snippet-level due to ACM 403; argued from corroborated tenets, not invented quotes. (cycle 1)

No HIGH-priority items remain open.
