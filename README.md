# Meta Ad Library Intelligence Tool — Architecture Proposal

> **Status:** Test assignment response — architectural proposal. **No code in this iteration.**
> All documents in this repository are Markdown specifications intended to defend the design and the development plan before any implementation is committed.

## What this proposal covers

This proposal defines a research tool that takes a brand name or product URL and reconstructs the **full Meta advertising ecosystem** behind it. Instead of returning ads from a single page, it links related advertiser pages (brand-owned, agency-managed, persona-style, and parallel testing structures) and analyzes creative strategy with a grounded AI layer.

Target users: competitive researchers, growth leads, and agency analysts who need to understand *who* is advertising, *what* is being tested, *which creatives are scaling*, and *how* the acquisition strategy is structured.

## Why this is hard

At first glance, this looks like a simple API integration. In practice, three problems drive most of the complexity:

1. **Naive search by brand name returns 10–30% of the picture.** Competitors run from multiple pages — brand, agency, persona networks. The tool's value is in *reconstructing the ecosystem*, not in retrieving ads.
2. **Meta Ad Library API has practical limitations** — incomplete impression data, partial creative metadata, rate limits, inconsistent fields. The pipeline must tolerate partial data without falling back to generic results.
3. **AI analysis without grounding becomes marketing noise.** The hard part is forcing the model to cite specific elements of ad copy and surviving spend signals, not to wax poetic about "engaging hooks."

## Core value proposition

- **Discovery layer** that finds related advertiser pages by shared landing domains, recurring creative patterns, CTA fingerprints, and UTM signals — with explicit confidence scoring.
- **Grounded AI** that must quote concrete creative elements; generic answers are filtered by post-validation and re-prompted.
- **Mock-first architecture** so the prototype is buildable in ~2 weeks and graduates to production without rewrites — Meta and Anthropic clients are hidden behind interfaces from day one.

## What's inside

|Document|Read it for|
|---|---|
|[ARCHITECTURE.md](./ARCHITECTURE.md)|The complete technical design: stack, data model, API surface, the deep Discovery section (signals, weights, pseudocode, 5 cases), AI grounding, resilience, success metrics, assumptions.|
|[ROADMAP.md](./ROADMAP.md)|Two-phase development plan: v1 MVP (~2 weeks, day-by-day) and v2 production hardening (~3–4 weeks) with explicit gates and off-scope.|

## Suggested reading path (10–15 minutes)

- **5 minutes:** this README + `ARCHITECTURE.md` §6 (Discovery).
- **15 minutes:** `ARCHITECTURE.md` §6–§7 first (Discovery + AI grounding), then `ROADMAP.md`.
- **After design review:** `ROADMAP.md` answers execution sequencing and delivery gates.

## Glossary

Quick definitions for non-technical reviewers:

- **Page** — the account unit on Meta from which ads are published. A single brand typically operates several related pages.
- **Advertiser ecosystem** — all pages promoting one product or brand: brand-owned, agency-managed, persona-style, and parallel test pages.
- **Creative** — a single ad: body text + media + CTA + landing URL.
- **n-gram overlap** — a similarity measure between two texts based on shared word sequences of length k. Used to detect near-duplicate headlines and CTAs across pages.
- **Confidence (link)** — a score in [0, 1] estimating that two pages belong to the same ecosystem. Computed as a weighted sum of independent signals.
- **Grounding (AI)** — the requirement that an LLM answer cite verbatim elements of its input rather than rely on the model's general prior. Enforced here by prompt design and post-validation.
- **Mock-first** — a development mode where external services (Meta, Anthropic) are replaced by fixtures behind a stable interface. The real integration is enabled by a configuration switch without changing any consuming code.

## Engineering principles

- **No marketing language.** The same grounding rule that we enforce on the AI layer is enforced on these documents: no "engaging," "compelling," "powerful." Concrete claims with concrete evidence.
- **Trade-offs stated honestly.** Where SQLite breaks, where confidence weights are expert priors rather than learned, where Meta API returns partial data — all called out.
- **Numbers are concrete but labelled.** Weights, thresholds, prices, and limits are spelled out, but explicitly marked as "expert prior" or "estimate" when they are not derived from ground-truth data.
- **No business-metric promises.** This is an engineering proposal, not an agency pitch — no ROI or conversion-lift claims.
