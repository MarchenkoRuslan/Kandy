# PITCH.md

Outline for the 10–15 minute oral defense of the architectural proposal. Audience: a technical interviewer evaluating a take-home assignment. Tone: engineering, no marketing.

This document is the **narrative spine**. It does not duplicate technical content — it points into [ARCHITECTURE.md](./ARCHITECTURE.md) and [ROADMAP.md](./ROADMAP.md) and tells the speaker which slide is on screen at each moment.

---

## Time budget

| Time | Block |
|---|---|
| 0:00–0:30 | Hook |
| 0:30–2:00 | Problem |
| 2:00–3:30 | Solution shape |
| 3:30–6:30 | Concept walkthrough (no UI mockup — ASCII wireframes from §9) |
| 6:30–8:30 | Architecture |
| 8:30–10:30 | Discovery deep-dive |
| 10:30–12:00 | AI grounding |
| 12:00–13:00 | Roadmap |
| 13:00–15:00 | Q&A |

Total: 13 spoken minutes + 2 minutes of buffer that doubles as Q&A overflow.

---

## Slide-by-slide

### Slide 1 — Hook (30 sec)

**On screen:**
> "A competitor is running campaigns from 12 pages. A naive search returns 1.
> This is not a data problem. It is a reconstruction problem."

**Speaker notes.**
- Lead with the asymmetry: the data is *available*; what's missing is *reconstruction*.
- Set the frame for the next 13 minutes — the value is in the layer between Meta and the user, not in calling Meta.
- Avoid: superlatives, "we've cracked," "game-changing," etc.

---

### Slide 2 — Problem (1.5 min)

**On screen:** the central diagram from [ARCHITECTURE.md §2](./ARCHITECTURE.md#2-container-view), with `Discovery Aggregator` highlighted.

**Speaker notes.**
- State the structure observed empirically: brand + agency + persona pages + parallel test pages. Pick **one** pattern (persona networks) to make concrete.
- Quote the ~10–30% naive-search coverage figure as an order-of-magnitude observation, not a measured KPI. Be honest that it's an estimate.
- Connect each Meta API limitation to a concrete consequence: incomplete impressions ⇒ ranking falls back to longevity; partial creative metadata ⇒ aggregator must tolerate `None`.
- Avoid: implying naive search returns "nothing useful." It returns *something*, just an incomplete picture.

**Anchor in docs.** [ARCHITECTURE.md §1](./ARCHITECTURE.md#1-system-context), [§8](./ARCHITECTURE.md#8-meta-api-resilience).

---

### Slide 3 — Solution shape (1.5 min)

**On screen:** the central diagram again, full view.

**Speaker notes.**
- Three concrete components beyond a Meta wrapper: **Discovery Aggregator**, **Grounded AI**, **Markdown Exporter**.
- The mock-first principle is the headline architectural decision. Say plainly: *"both `MetaClient` and `AdAnalyzer` are Protocol-typed; mocks ship from day one; the real adapters are the only modules that change for v2."* This is the de-risking argument.
- Explain why Markdown is the export format: deterministic against the stored snapshot, no rendering toolchain, easy to extend. If asked "what does it actually look like?" — flip to [ARCHITECTURE.md §9.6](./ARCHITECTURE.md#96-markdown-export--sample-structure) for the literal export template.
- Avoid: getting drawn into stack-justification yet — that's slide 5.

---

### Slide 4 — Concept walkthrough (3 min)

**On screen:** ASCII wireframes from [ARCHITECTURE.md §9](./ARCHITECTURE.md#9-ui--ux), one screen at a time. **No Figma. No mockup.** The wireframes *are* the demo.

**Speaker script (rough).**
1. (Search screen) "User enters `acme-fitness.com` as a URL. The system normalizes to eTLD+1 and derives search terms."
2. (Dashboard) "We surface five pages — one is the seed, four are discovered. Stats first, creatives second."
3. (Expanded card) "Persona-style page `Sarah's Fitness Tips`. Confidence 0.86, marked strong. AI summary cites the recurring CTA pattern that survived across 7 of 8 ads."
4. While walking through: open [§6.5 case 2](./ARCHITECTURE.md#65-synthetic-cases) and read the confidence breakdown aloud — `0.40 + 0.175 + 0.15 + 0.08 + 0.10 ≈ 0.91`. **This is the demo without a product.**

**Speaker notes.**
- This slide is the test of whether the audience believes the system reconstructs ecosystems correctly. The persona-grid case is the most visceral example — pick it.
- It's fine to spend 90 seconds here; it's the most concrete slide.
- Avoid: pretending the wireframes are an implementation. State openly that this is the *intended* shape.

**Anchor in docs.** [ARCHITECTURE.md §9](./ARCHITECTURE.md#9-ui--ux), [§6.5 case 2](./ARCHITECTURE.md#65-synthetic-cases).

---

### Slide 5 — Architecture (2 min)

**On screen:** the container diagram, then the sequence diagram from [§2](./ARCHITECTURE.md#2-container-view).

**Speaker notes.**
- Three containers: React SPA, FastAPI service, SQLite file. Justify each in one sentence (see [§3](./ARCHITECTURE.md#3-stack-and-rationale)).
- Mock-first + interface-driven + partial-data tolerance — the three cross-cutting principles. Say the names; they recur in the FAQ.
- Long requests go via SSE with stage markers, not async-job polling. Briefly explain why: less infrastructure for v1, scales by replacing the in-process generator with Redis pub-sub in v2.
- Avoid: deep stack religion. Rejected alternatives are listed in [§3](./ARCHITECTURE.md#3-stack-and-rationale); refer the curious there.

---

### Slide 6 — Discovery deep-dive (2 min)

**On screen:** the signals table from [§6.2](./ARCHITECTURE.md#62-signals-of-relatedness-between-pages) and the confidence formula from [§6.3](./ARCHITECTURE.md#63-confidence-scoring).

**Speaker notes.**
- Walk through the five signals in order of weight. **State that the weights are expert priors, not learned.** Calibration on ground truth is in [v2 step 6](./ROADMAP.md#v2-scope-in-priority-order).
- Pick **case 5** from §6.5 (the affiliate-program false-link) as the second concrete example. It demonstrates two things: (a) the conservative threshold rejects single-signal evidence, (b) we explicitly designed for false positives, not just true positives.
- Explain the 2-hop traversal limit — predictability over completeness.
- Avoid: getting trapped in weight-tuning debates. Say plainly: "the v2 calibration task makes these weights a hypothesis, not a commitment."

**Anchor in docs.** [ARCHITECTURE.md §6](./ARCHITECTURE.md#6-discovery--the-core-differentiator).

---

### Slide 7 — AI grounding (1.5 min)

**On screen:** before/after comparison on a single ad. Construct on the fly:

> **Generic (rejected by post-validation):**
> "This is a compelling ad with an engaging hook and a powerful CTA that creates urgency."
>
> **Grounded (accepted):**
> "Hook: `I dropped 10 lbs without changing what I eat` — first-person specific outcome.
> Offer clarity: 4/5 — `free first week` is a concrete commitment.
> Why performing: 47-day longevity at 30K–50K imp range suggests survived spend allocation."

**Speaker notes.**
- Read both aloud. The first one *sounds* like analysis. The second one *is* analysis.
- Walk through the enforcement mechanism: prompt rule + forbidden-vocabulary regex + one-shot retry + `[weak]` flag if the second attempt also fails.
- Be explicit that the grounding pass-rate is a measured success metric ([§12.2](./ARCHITECTURE.md#122-quality-of-ai-analysis)), not an aspiration.
- Avoid: claiming AI is hallucination-proof. Say: "we limit the surface area where hallucination can hide."

---

### Slide 8 — Roadmap (1 min)

**On screen:** the phase mermaid from [ROADMAP.md](./ROADMAP.md#phases-at-a-glance).

**Speaker notes.**
- v1 in ~2 weeks, mock-first, 5 cases pass, all interfaces in place.
- v2 in ~3–4 weeks, real APIs, Postgres, queue, calibrated weights.
- Off-scope (multi-platform, OCR, RBAC, on-premise) listed explicitly — these are *not* "we'll get to it"; they're "deliberately not in this product right now."
- The gate between v1 and v2 is success-metric driven, not time driven.

**Anchor in docs.** [ROADMAP.md](./ROADMAP.md).

---

### Slide 9 — Q&A (2 min minimum)

Open the floor. Use the FAQ below as a backup if questions are slow.

---

## FAQ — likely interviewer questions

Phrased as concrete answers, not bullet points the speaker has to translate.

### "Why this stack — why not Next.js / Node?"

FastAPI gives async I/O for paginated fan-out to Meta and Pydantic for a strict, generated contract — `openapi-typescript` produces frontend types from the backend schema, so we don't maintain two sources of truth. Vite is the right answer when SSR isn't needed; Next.js's App Router and RSC are dev-loop overhead for a single-tenant research dashboard. SQLite is intentional for v1 — zero-ops, single-file snapshots make Markdown export deterministic, and the migration to Postgres is a [single explicit gate in v2](./ROADMAP.md#v2-scope-in-priority-order).

### "How is this different from Adbeat / BigSpy / native Meta search?"

Those tools surface *ads*. We surface the *advertiser ecosystem* by reasoning over signals across pages. Native Meta search is name-based and does not connect a brand to an unrelated-named persona page even when both link to the same domain. The product's argument is the discovery layer in [§6](./ARCHITECTURE.md#6-discovery--the-core-differentiator), not the ad data itself.

### "How do you validate the AI doesn't hallucinate?"

Three layers. (1) Prompt-level grounding rules require citations from the input. (2) Regex post-validation against a forbidden-vocabulary list — concrete words like `engaging`, `compelling`, `powerful`. (3) On a hit, one-shot retry with an explicit "quote from the body" instruction; on a second failure, the response is stored with a `[weak]` flag and surfaced in the UI rather than retried indefinitely. The first-try grounding pass rate is a measured metric — target 80% for v1, 95% with retry in v2. See [§7.3](./ARCHITECTURE.md#73-grounding-rules), [§12.2](./ARCHITECTURE.md#122-quality-of-ai-analysis).

### "How do you handle Meta API limitations?"

Cursor pagination to the empty `paging.next` (no first-page-only assumption); `tenacity` exponential backoff with jitter on `429`/`5xx`; partial-data tolerance throughout — all Pydantic ad fields except the two stable IDs are `Optional`; session limits (`MAX_PAGES`, `MAX_ADS`, `MAX_DURATION_SECONDS`) for predictable cost and time; 24-hour response cache keyed by `(page_id, fetched_at_day)`. See [§8](./ARCHITECTURE.md#8-meta-api-resilience).

### "Where do the confidence weights come from?"

Expert priors, not learned. The dominance of the domain signal (0.40) reflects an empirical observation that a shared landing domain is the strongest single indicator; n-gram overlap is next-most-reliable because reusing creative copy is operationally common across pages an advertiser owns. The 0.45 membership threshold means **two strong signals are sufficient, one supporting signal is not** — biasing toward precision over recall. v2 task 6 fits weights against ground truth — see [ROADMAP.md v2 scope](./ROADMAP.md#v2-scope-in-priority-order).

### "Why is a code-less prototype useful here?"

The hard part of this assignment is not implementation. It is: (a) formalizing what "ecosystem reconstruction" means, (b) choosing which signals to use and how to combine them, (c) ensuring AI output is grounded rather than decorative. All three are settled by these documents. Two weeks of implementation against this design starts with no remaining design decisions. A code prototype written before this design would either reproduce these decisions implicitly (and fragilely) or paper over them.

### "What would you do differently with more time?"

Three concrete things. (1) Add visual signals to discovery — perceptual hashes of creative images and OCR of in-image text are powerful but require a media-handling pipeline. (2) Calibrate confidence weights against a manually labelled ground-truth set rather than expert priors. (3) Add an LLM-as-a-judge layer to the AI grounding validation rather than regex-only — catches semantic generic phrasing the regex misses. All three are in [v2 or off-scope](./ROADMAP.md#v2-production-hardening-3-4-weeks).

### "What did you deliberately leave out?"

Multi-platform (TikTok, Google Ads), visual hashes, OCR, RBAC, multi-user workspaces, on-premise deployment, exports beyond Markdown, scheduled monitoring, conversion-funnel analytics. Each is named explicitly in [ROADMAP.md off-scope](./ROADMAP.md#off-scope-neither-v1-nor-v2). The discipline of naming them is part of the answer.

### "How would you measure success?"

[§12](./ARCHITECTURE.md#12-success-metrics) lists six measurable properties: ecosystem recall vs. ground truth, false-link rate, AI grounding pass rate, citation density, time-to-result P95, AI cost per run. v1 targets are looser than v2 by design — v1 is a structural gate, v2 is a quality gate.

---

## Self-checks for the speaker before the call

- Have I read the central diagram, §6, and §7 of `ARCHITECTURE.md` immediately before the call?
- Can I draw the confidence formula from memory?
- Do I have **two** synthetic cases ready to talk through (recommended: case 2 and case 5 from §6.5)?
- Am I prepared to answer "how would you implement this in two weeks" by walking through the day-by-day in [ROADMAP.md week 1 and week 2](./ROADMAP.md#week-1-backend-foundation)?
- Have I disabled marketing vocabulary in my own speech? The same words I forbid the AI to use are the ones I should not use either.
