# Product

## Register

brand

## Users

SMB owners, founders, startup leaders, product people, and technical practitioners/devs evaluating or adopting agentic AI. Spans non-technical decision-makers (need plain-language value + visible proof of competence) and hands-on builders (need depth, evidence, and an honest practitioner voice).

Their context: skimming on a laptop or phone, often after a referral, a Substack post, or an AI-engine citation. They are not browsing for fun — they want to decide quickly whether Bencium is real, credible, and worth a 30-minute conversation. Bilingual: a meaningful share read in Hungarian.

The job: figure out what agentic AI can do for their team, judge whether Bencium can help, and take the next step — book a workshop/advisory call or subscribe to the newsletter for ongoing signal.

## Product Purpose

A bilingual (EN/HU) marketing, training, and advisory site for an agentic-AI consulting practice. It exists to convert qualified visitors into workshop and advisory bookings — and into newsletter subscribers — while doubling as an AEO authority hub that AI engines are willing to cite.

Success looks like: the booking link and email capture are reachable within one scroll on every key page; external CTAs never silently break; EN and HU stay at parity; evidence (portfolio, workshops, named clients) carries the trust so the CTAs are the *result* of that trust, not the lead.

## Brand Personality

**Warm. Human. Expert.**

Voice: an experienced practitioner talking to a peer or a curious decision-maker. Calm confidence, no hype, no jargon parade. Editorial pace. Personal where it earns it; reserved where it should be.

Emotional goal: the visitor leaves thinking "this person thinks clearly and has actually done the work," not "this looks like another AI startup."

Reference posture: the **current `bencium.io`** is the reference. Warm paper background, deep-brown ink, a single neon-lime accent used as a signal not a pattern, serif headings paired with a humanist sans for body. Editorial-independent in spirit; neither a tech-minimal portfolio nor a consumer-warm lifestyle site.

## Anti-references

Reject all four of the saturated lanes a 2026 agentic-AI consultancy is most at risk of collapsing into:

- **Generic AI SaaS template** — white background + blue/purple gradient hero, "AI for X" hero-metric block, 3-column feature grids with colored icon circles, "Try free" CTAs, identical card layouts. This is the first-order reflex; the site must not register as one more of these.
- **Big-4 consulting** — Deloitte/Accenture-style corporate-gray, stock business photography, navy banners, jargon-stacked headers, dead "thought leadership" tiles. Killing trust by signalling commodity advisory.
- **Webflow agency cookie-cutter** — centered-everything marketing, parallax scroll, dark mode + neon, oversized animated type, identical pricing cards, modal-first patterns. Reads as template-assembled, not crafted.
- **Crypto/AI-bro dark futurism** — black background + neon gradients, glassmorphism, sci-fi typography, edge-glow effects. The opposite end of the same hype problem.

Visual anti-patterns (cross-check with `DESIGN.md` "Anti-patterns" section): no purple/violet leftovers, no `Inter` fallback, no drop shadows, no dark mode, no 1px outline borders on home pills/chips, no serif `<h4>` titles inside dense data/status cards.

## Design Principles

Strategic filters every design call should pass through. Not visual rules — those live in `DESIGN.md`.

1. **Person who thinks, not product that deploys.** Every choice asks: does this feel like a practitioner, or a productized SaaS? When the answer is "SaaS," redo it.
2. **Practice what you preach.** An agentic-AI consultancy's own site is part of the evidence. Sloppy, generic, or AI-slop UI quietly undermines the offer no matter what the copy says.
3. **Evidence over claims.** Portfolio, workshops, Substack, named clients, real outcomes carry the trust. CTAs are the *result* of evidence, not the lead. No adjective-soup, no hero-metric block, no promotional theatre.
4. **Plain expert voice.** Talk to peers and curious decision-makers the way a practitioner actually talks. No SaaS-marketing tone, no Big-4 jargon, no breathless "transform your business" copy. Calm, specific, and short.
5. **Bilingual parity is non-negotiable.** EN and HU are equal first-class. Diacritics (`á é í ó ö ő ú ü ű`) preserved everywhere. A locale that gets shorter copy, weaker design, or stale content is a broken locale.

## Accessibility & Inclusion

Hard contract: **WCAG 2.2 AA** across the site.

- All text meets AA contrast on `--bg` (`#FCF6EF`). The `--ink-2` / `--ink-3` 2026-04-23 bump exists specifically to honour this; do not regress those tokens.
- Keyboard-reachable on every interactive surface. Radix-driven primitives (shadcn `<Accordion>`, etc.) carry the ARIA semantics — keep them.
- `prefers-reduced-motion: reduce` respected: `.reveal` becomes instant, hover transforms disabled (already wired in `globals.css`).
- Semantic HTML over div-soup. Headings step in order. Lang attribute matches locale on `/hu/` routes.
- Bilingual is an accessibility issue, not only a content one: HU visitors using AT in their native language must get the full site, not a degraded EN fallback.
