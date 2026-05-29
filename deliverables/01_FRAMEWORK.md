# 01 — The AI Usage Quality Framework

## 1. What we are measuring (and the trap we refuse)

The question is deceptively easy to corrupt. "Measure the quality of an engineer's AI usage" slides almost instantly into one of three things it must **not** become:

- **Code quality** — forbidden (no code access) and a different construct entirely.
- **Productivity / individual worth** — the McKinsey trap [15]; produces ranking, fear, and gaming.
- **Activity / volume** — the most common and most seductive error. SPACE [3] names activity "the most visible and most misused dimension." Tokens consumed, suggestions accepted, lines generated — all are *volume*, not *quality*.

So the first act of the framework is a definition that survives the no-code constraint and the enablement mandate:

> **AI usage quality = the degree to which an engineer is getting durable, low-friction, well-matched leverage from AI tools — i.e., AI is reliably reducing their effort on the right work without creating downstream rework, friction, or fragility.**

Three load-bearing words, each chosen against a failure mode:

- **Durable** — the leverage survives contact with reality (review, CI, time). This is what separates *quality* from *volume*. GitClear [11][12] shows AI inflates volume while churn rises; the durability lens is precisely what catches that. Operationalized as code/suggestion **survival/retention** and low **rework**.
- **Low-friction** — AI is smoothing the path, not adding round-trips. Grounded in DevEx [4] (feedback loops, cognitive load). Operationalized via cycle-time shape and review friction *trends*.
- **Well-matched** — the engineer reaches for the right mode/tool for the task (autocomplete for boilerplate, agent for refactors, chat for exploration) rather than one shallow habit. Grounded in the DX AI framework [7] and vendor feature-mix telemetry [19][22][23].

This is a **proxy construct**, and we say so plainly (construct-validity honesty, [32][33]): we never observe "quality" directly. We observe metadata and self-report, and infer quality only where multiple independent proxies agree.

### What this framework is NOT (keep this box visible)
| It is NOT… | Because… |
|---|---|
| A measure of code quality/correctness | No code access; different construct |
| A productivity or performance score | Enablement-only; output-scoring backfires [15] |
| A peer ranking / leaderboard | Confounds make cross-person comparison invalid [37]; ethics [3][4] |
| A manager- or HR-facing artifact | Audience-locked to the engineer (constraint #2) |
| A single number | SPACE [3]: no single metric; triangulation required [32] |
| A real-time judgment | Individual samples are tiny/noisy; ≥4-week windows |

## 2. Why proxies *can* validly stand in for quality (the construct-validity argument)

A skeptic's fair challenge: "you can't see the code, so you can't judge quality." Correct — for code quality. But "usage quality" is a different latent construct, and measurement theory [32] tells us how to validly measure latent constructs we can't see directly: build a **nomological network** — a small web of indicators that, *per theory*, should move together if the construct is real, and establish **convergent** and **discriminant** validity.

Our nomological network:

```
                 AI USAGE QUALITY (latent)
                          │
      ┌───────────────────┼───────────────────┐
   LEVERAGE/           ADOPTION             DURABILITY /
   EFFICIENCY           DEPTH               LANDING QUALITY
   (low friction)     (well-matched)        (durable)
      │                   │                     │
  cycle-time          feature-mix           code/suggestion
  shape, time-        (agent/chat vs        survival, churn,
  saved (self-        autocomplete),        first-pass CI,
  report)             tool fit, engaged     review friction
                      not just active        trend, revert rate
```

- **Convergent validity**: if AI usage is genuinely high-quality, an engineer should show *durable* output (high survival, low churn) AND *low friction* (healthy cycle-time shape, clean first-pass CI) AND *depth* (uses the right mode). When these co-move, the construct is real. When only volume moves, it is not.
- **Discriminant validity**: the framework must separate "quality" from "quantity." We enforce this structurally by **down-weighting all volume metrics** (tokens, raw acceptance, LOC) and never letting volume alone trigger a positive finding (see §5 and deliverable 02).
- **Triangulation across method types** [3][4][32]: objective telemetry + diff-metadata outcomes + perceptual self-report. Each method has different biases; agreement across methods is what earns confidence.

This is exactly why the framework is multi-signal and confidence-tiered rather than a score: a latent construct measured by noisy proxies *demands* agreement-based inference.

## 3. The three pillars in detail

### Pillar 1 — Leverage / Efficiency ("is AI reducing friction?")
*Theory base: DevEx feedback loops & cognitive load [4]; DX Impact [7]; DORA flow [1][2].*

The question: is AI making the path to a merged change smoother, or just busier? Signals (all no-code):
- **Cycle-time shape** decomposed into coding / pickup / review / deploy from PR+commit timestamps [38][39]. Quality looks like compressed *coding* time without ballooning *review/rework* time. The anti-pattern (DORA amplifier [2]): AI compresses authoring but inflates downstream instability — friction moved, not removed.
- **Self-reported time-saved & flow** [7][4] — the legitimate domain of self-report (perception of *experience*, not ground-truth speed).
- **Cognitive-load proxies** — context-switching, interruption patterns from collaboration metadata (additional data, deliverable 04).

**Honesty flag**: self-reported speed is *not* truth (METR [17]: devs felt +20% while −19%). Pillar 1 uses self-report for *experience*, and pairs it with objective cycle-time shape; it never treats "I feel faster" as proof of faster.

### Pillar 2 — Adoption Depth ("shallow autocomplete vs deep agentic?")
*Theory base: DX AI Utilization [7]; vendor feature-mix telemetry [19][22][23][25].*

The question: is the engineer using AI as a thinking partner across the right modes, or stuck in one shallow habit? Signals:
- **Feature/mode mix** — agent/plan/chat vs autocomplete-only [19][22][23]. A heavy-tab / zero-agent / zero-chat profile = shallow usage.
- **Tool fit & breadth** — given ~82% of devs use 3+ tools [7], quality is using the *right* tool per task, not maximal tools. Normalized across the engineer's actual portfolio.
- **Engaged-not-just-active & consistency** — GitHub's own "engaged user" cut [21] (took an intentional action) over mere "active"; recency/cadence.
- **Model/effort calibration** — right-sizing model choice to task difficulty [23][7].

**Honesty flag**: depth is a *means*, not an end. Deep agentic use that produces high churn is worse than shallow use that lands cleanly. Pillar 2 is only meaningful *in conjunction with* Pillar 3.

### Pillar 3 — Durability / Landing Quality ("does AI-assisted output clear review cleanly?")
*Theory base: GitClear churn/survival [11][12][41]; Copilot survival [10][25]; DORA stability [1][2]; CI/review metadata [43].*

The strongest pillar for *quality* (vs volume), and the one most uniquely enabled by the no-code constraint — because durability is computable entirely from diff metadata and event status. Signals:
- **Code / suggestion survival & retention** — Sourcegraph exposes time-bucketed persistence natively (30–600s) [25]; reconstructable elsewhere from git line-survival over N days [41][49]. *The single most defensible quality signal* (resolved H2).
- **Churn / rework** — lines reverted/revised within ~2 weeks [11][12][41]; revert/hotfix detection from commit metadata [42].
- **First-pass CI success & trend** [43] — does AI-assisted work pass on the first run?
- **Review friction trend** — changes-requested ratio, review round-trips, turnaround [38][39] — observed as *personal trend*, not cross-person.

**Honesty flag**: every Pillar-3 signal is heavily confounded (CI fails for a hundred reasons; review friction tracks task difficulty). These are usable only as *personal trends* with confound controls (§5), never as absolutes or rankings.

## 4. The "amplifier" spine (connecting individual → context)

DORA 2025's central finding — **"AI is an amplifier; it doesn't fix a team, it amplifies what's already there"** [2] — is the interpretive spine of the whole framework. It means an individual's AI-usage signals can only be read *against context*: the same churn rate means different things on a mature, well-tested platform vs a fragile legacy repo. Practically:
- Team/system context (DORA metrics [1][2], repo characteristics) is used as a **lens to interpret** individual signals — never blended into the individual score (that would violate construct validity, X13).
- Recommendations explicitly account for whether the engineer's environment lets AI amplify *up* (strong foundations) or *down* (weak foundations).

## 5. How the framework avoids needing code — and stays valid

| Design choice | What it defends against | Mechanism |
|---|---|---|
| Proxy construct, stated openly | Overclaiming; construct invalidity | Define quality as durable/low-friction/well-matched leverage; never as code quality |
| Multi-signal triangulation | Single-metric gaming (Goodhart [30]) | Positive findings require ≥2 independent agreeing signals across method types |
| Volume metrics structurally down-weighted | Activity-as-quality (SPACE [3]); discriminant-validity failure | Tokens/acceptance/LOC can never *alone* trigger a finding |
| Trend-vs-personal-baseline | Confounds (tenure/role/repo/task [37]); peer-ranking harm | Compare each engineer to their own rolling baseline, not peers |
| Confidence tiering | Inference dressed as evidence | Every output carries High/Medium/Exploratory + reasoning |
| Audience-lock + opt-in | Surveillance/observer effect [34][35] | Engineer-only; no manager/HR/comp linkage |
| Self-report for experience only | Perception gap (METR [17]) | Never treat felt-speed as ground-truth speed; surface the gap as coaching |
| Code-access audit each cycle | Constraint #1 drift | Any metric needing diff contents is killed (one was — see RESEARCH_LOG cycle 3) |

## 6. Preconditions (be honest about what must be true)
- **P0 — Identity join**: AI-tool usage identity must join to GitHub identity per engineer (assumption A2). Without it, per-engineer framing is impossible.
- **P1 — Time-bucketing**: signals bucketed to ≥4-week windows for stable individual inference (A7).
- **P2 — Tool-portfolio awareness**: signals normalized across each engineer's actual tools (A9).
- **P3 — Consent/transparency**: engineer can see, opt out of, and delete their data (deliverable 06).
- **P4 — AI attribution (the hardest precondition)**: the durability pillar's *ideal* form ("survival/churn of *AI-assisted* code") requires knowing which committed lines were AI-influenced. **Most vendor telemetry cannot do this** — delivery-analytics tools openly admit they can't distinguish AI- vs human-authored lines [18], and git line-survival measures *all* code, not AI-specific code. Only Sourcegraph natively ties survival to *accepted suggestions* [25]. **Honest consequence:** without AI attribution, Pillar-3 signals measure the engineer's *overall* durability (a coarser proxy that AI usage merely *contributes* to), not AI-specific durability. The framework still works at this coarser resolution — read longitudinally against the engineer's own pre-AI baseline, a *rise* in churn coinciding with *rising* AI adoption is suggestive — but every survival/churn claim must be labeled "AI-attributed" vs "overall" honestly, and "overall" is the realistic default for most orgs.

## 7. One-paragraph statement of the framework
We define AI usage quality as *durable, low-friction, well-matched leverage from AI* — never code quality, productivity, or activity. We measure it as a latent construct through a nomological network of three pillars (Leverage/Efficiency, Adoption Depth, Durability/Landing-quality), each built from no-code metadata and self-report, triangulated across method types, interpreted through DORA's "amplifier" lens against team context, disciplined by confound controls and confidence tiering, and delivered only to the engineer as enablement. Quality is inferred only where independent proxies agree; volume never counts as quality.
