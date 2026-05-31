# Design System — Bencium

> Source of truth for every visual and UI decision on this site. All tokens, rules, and rationale live here. When `globals.css` and `DESIGN.md` diverge, one of them is wrong — reconcile before making new changes.

## The memorable thing

**Warm human expertise, not SaaS.** Every decision in this document serves that single line. When in doubt, ask: does this make Bencium feel like a person who thinks, or a product that deploys?

## Product Context

- **What this is:** Marketing + service site for Bencium's agentic AI training and advisory practice. Bilingual EN/HU.
- **Who it's for:** Founders, product leaders, and enterprise teams evaluating agentic AI. People who book consultations, not people who download free tools.
- **Space/industry:** AI training/advisory, adjacent to executive coaching and technical consultancy.
- **Project type:** Marketing site with authenticated studio (Neon Auth guards `/studio` only).

## Aesthetic Direction

- **Direction:** Warm editorial-minimal. Paper + ink + one electric accent.
- **Decoration level:** Minimal — typography and whitespace carry the work; lime accent used sparingly as a signal, not a pattern.
- **Mood:** "A person who thinks" rather than "a product that deploys." Warm human expertise, not SaaS. Calm confidence. Craft-first.
- **Reference posture:** Editorial independent publication, not a Webflow SaaS template.

## Typography

- **Display / Headings (h1-h6) + Body + UI:** `Telegraf` (self-hosted variable font in `public/fonts/Telegraf-*.{ttf,woff2,woff}`, weights 100-900, declared via `@font-face` in `globals.css`). Headings are bold (700/600); body is line-height 1.5.
  - *Why:* One typeface across the whole site. Telegraf is a geometric sans with slight humanist warmth. Self-hosted = no third-party Google Fonts call for the primary face, faster LCP, privacy-friendlier.
- **Heading ramp** (set in `globals.css` `@layer base`): h1 `clamp(48px, 8vw, 120px)` / 700 / -0.035em / lh 0.95 · h2 `clamp(32px, 5vw, 64px)` / 700 / -0.03em / lh 1.02 · h3 `22px` / 600 / -0.01em / lh 1.2. Base h1-h6 default to weight 700, lh 1.1, -0.02em.
- **Mono / Terminal accents:** `JetBrains Mono` (Google Fonts `@import` — **must stay at the very top of `globals.css`**, before any style rule, or the browser drops it). Used for `.font-terminal` and `.terminal-accent` highlights only.
  - *Why:* Builder-signal. The lime-tinted `.terminal-accent` is one of the site's fingerprints.
- **Fallbacks (sans):** `system-ui, -apple-system, 'Segoe UI', 'Roboto', 'Helvetica', 'Arial', sans-serif`. **Do not add `'Inter'` back** — it was removed because it was a dead fallback string that signaled "generic SaaS."
- **Scale:** Headings lean on size + tight letter-spacing for hierarchy.

> **Note:** `Newsreader` / `--font-serif` are **not** used in the live app. They exist only in the standalone `Bencium Design System/` kit, never wired into `src/`. The site renders everything in Telegraf — do not reintroduce a serif heading face without a deliberate design decision.

## Color

- **Approach:** Restrained. Two core colors (warm paper + deep brown ink), one electric accent (neon lime), plus neutrals. Color is rare and meaningful.
- **Palette (light — shipped, defined in `globals.css:49-102`):**

  | Role | Token | Hex |
  |---|---|---|
  | Page background | `--bg` | `#F0E9EC` |
  | Elevated surface / cards | `--surface`, `--surface-elevated` | `#FAF6F2` |
  | Tinted section bg | `.section-tinted-*` | `#F5EDE4` |
  | Primary text + primary brand | `--ink-1`, `--brand` | `#2B180A` |
  | Muted body text | `--ink-2` | `#4A3E32` |
  | Subtlest UI labels / kickers | `--ink-3` | `#6B5B4C` |
  | Borders / dividers | `--line` | `#E3DCD5` |
  | Accent (secondary brand) | `--brand-2` | `#bdff1b` |
  | Success | `--ok` | `#059669` |
  | Warning | `--warn` | `#d97706` |
  | Danger | `--danger` | `#dc2626` |

- **Rationale:** Warm beige backgrounds instead of pure white. Deep brown ink instead of pure black. This is the "human expertise, not SaaS" fingerprint — every AI consultancy converges on white + blue/purple. Bencium is paper.
- **Muted text contrast (2026-04-23):** `--ink-2` and `--ink-3` are both warm browns stepped down from `--ink-1` — `#4A3E32` (body-muted, ~8:1 AAA) and `#6B5B4C` (subtlest, ~5:1 AA) on `--bg`. Prior values were both `#94877C` (taupe, ~3.3:1) which failed WCAG AA for body text. The two tokens are now visually distinct again to preserve the three-step hierarchy (primary → body-muted → subtlest).
- **Accent rules:** `--brand-2` (neon lime) is used for: `.tag` pills, `.terminal-accent` inline highlights, "New" ribbons, and select hover states. **Never** as a primary CTA background — primary CTAs use `--ink-1` (deep brown) with white text.
- **Dark mode:** **Not supported.** The `.dark {}` tokens in `globals.css` are unmodified shadcn defaults (pure grayscale oklch) and do not cohere with the warm palette. Do not ship a dark-mode toggle until those tokens are redesigned against the warm identity.

### Section tints — current state

`.section-tinted-purple`, `.section-tinted-lime`, and `.section-tinted-sage` are all currently visually identical (`#F5EDE4`). The class names are kept because they're already used across 20+ section components (`about`, `solutions`, `method`, `doctrine`, `movement`, `resources`, `evidence`, `tools-showcase`, `ai-readiness`, `workshops`, `design`). They serve as **semantic anchors** for potential future differentiation. If the design intent changes to actually tint sections differently, all three hex values get assigned at once — don't partially tint.

## Spacing & Layout

- **Content max width:** `--maxw: 1200px` applied via `.wrap`.
- **Container padding:** `24px` desktop, `16px` mobile (under 600px).
- **Border radius scale:**
  - Cards: `--radius: 24px` (also `--radius-lg` via `@theme inline`)
  - `--radius-md`: 22px, `--radius-sm`: 20px, `--radius-xl`: 28px
  - Buttons + `.tag`: `100px` (pills)
  - `.chip`: `999px` (fully pill)
- **Shadow:** `none`. Flat design is a deliberate choice — no drop shadows, no elevation, no depth. Hover states use `transform: translateY(-2px)` and color changes, never shadow.
- **Grid:** Inherited from Tailwind v4. Responsive breakpoints at `600px` and `900px` (see `globals.css` media queries).

## Motion

- **Approach:** Intentional, not expressive. Motion aids comprehension (reveal on scroll, FAQ accordion, card hover lift). Never decorative.
- **Reveal on scroll:** `.reveal` class (`opacity: 0 → 1` + `translateY(50px) → 0`), 1.2s cubic-bezier(0.25, 0.46, 0.45, 0.94). Stagger via `.reveal-stagger` children.
- **Card hover:** `transform: translateY(-5px)` + optional `.card-reveal` (scaling brand border). 0.3s ease-out.
- **Button hover:** `transform: translateY(-2px)`. Active state `scale(0.98)`.
- **FAQ:** max-height + opacity + padding transitions, 0.3-0.5s cubic-bezier(0.4, 0, 0.2, 1).
- **Marquee:** 30s linear infinite for logo rows. Pauses on hover.
- **Accessibility:** All motion respects `prefers-reduced-motion: reduce` — reveal becomes instant, card transforms disabled.

## Components (shipped conventions)

- **`.btn`** — base pill button, `padding: 14px 28px`, transparent border.
- **`.btn.primary`** — deep brown (`--ink-1`) background, white text. Primary CTA.
- **`.btn.ghost`** — outlined, transparent background.
- **`.btn.small`** — reduced padding.
- **`.card`** — 24px radius, `--surface` background, 32px padding, no border, no shadow.
- **`.card-reveal`** — card variant with scaling brand border on hover/focus. Lime variant: add `.card-reveal-lime`.
- **`.tag`** — lime pill, 6px/16px padding, weight 600, for category chips.
- **`.tag-outline`** — subtle filled chip, no border, `--bg` background. (Border removed 2026-04-24 — see Decisions Log.)
- **`.chip`** + `.chips`** — smaller pill chip, for metadata rows (1px border retained — used on dense tech-stack lists in `/about`, not on home).
- **Latest Activity cards (`status-board.tsx`)** — borderless white-fill cards (no `--line` 1px). Title is `<h4>`; renders in Telegraf via the global `h1-h6` rule (all headings are sans). Existing inline `var(--font-sans)` pins are now redundant but harmless.
- **External links** — default: text inherits, underline 2px `--brand-2` (lime off-site signal). Hover: text and underline both `--ink-1` (no lime text fill on hover).
- **`.kicker`** — uppercase brown eyebrow, 12px, weight 700, letter-spacing 0.06em.
- **`.ribbon`** — "New" lime badge, absolute-positioned on cards.
- **`.font-terminal`** — JetBrains Mono for code/terminal visuals.
- **`.terminal-accent`** — lime-tinted mono inline highlight.
- **`.sticky-mobile-cta`** — fixed bottom bar, uses shadow (the only shadow in the system — justified by depth cue for fixed elements).

## Content Rules (hard rules)

- **Never alter user-provided copy.** Preserve exact Hungarian accents (á, é, í, ó, ö, ő, ú, ü, ű).
- **No emojis in UI/copy** unless explicitly requested.
- **External links preserved:**
  - Booking: `https://calendar.app.google/CxQaQwjEYpW1NgVJ8`
  - Newsletter: `https://bencium.substack.com`
  - Email: `ben@bencium.io`
- **Bilingual pattern:** Hungarian pages at `/hu/` with `_hu.tsx` component twins. No i18n library.

## Anti-patterns (automatically flag in review)

- Purple or violet gradients. The system has no purple. Any `rgba(49, 8, 227, ...)` or `#310ae3` is a leftover from a prior theme.
- Generic Inter/Roboto/system-ui as the primary display or body font.
- Drop shadows on cards or buttons. Flat design is deliberate.
- Dark mode assumptions. Dark mode is not supported.
- Pure white (`#FFFFFF`) backgrounds on content surfaces. Use `--bg` or `--surface`.
- 3-column icon grids with colored circles (the SaaS feature-grid trope).
- Centered-everything marketing layouts. Bencium leans editorial-asymmetric where possible.
- 1px outline borders on pill/chip components on home sections. `.tag-outline` is now borderless; pill separation comes from fill + radius, not stroke.
- External-link hover turning text to `--brand-2` (lime). Hover state is `--ink-1` for both text and underline — the lime is the off-site *signal*, not a hover effect.

## Decisions Log

| Date | Decision | Rationale |
|---|---|---|
| 2026-04-20 | Initial DESIGN.md created from `globals.css` as-shipped | Stops design drift across Claude sessions. Formalizes the Claura warm-beige system iterated on since at least Mar 2026. |
| 2026-04-20 | Removed `'Inter'` from sans fallback stack | Dead fallback string — never actually loaded. Signaled "generic SaaS" vs the intended "human expertise" posture. |
| 2026-04-20 | Deleted `.btn-glow` / `.btn-glow-lime` and their keyframes | Both classes had zero usages across `src/`. Removed ~50 lines of dead CSS including purple `rgba(49, 8, 227, ...)` leftovers. Restore from git if pulsing CTAs come back. |
| 2026-04-20 | Re-tinted `.prose code` background from purple to brown | `rgba(49, 8, 227, 0.08)` → `rgba(43, 24, 10, 0.08)`. Matched `--ink-1`. |
| 2026-04-20 | Dark mode documented as **not supported** | `.dark {}` tokens are shadcn defaults, don't cohere with warm palette. Honest documentation beats shipping broken. |
| 2026-04-20 | `section-tinted-*` kept as three distinct class names despite identical hex | Preserves semantic anchoring across 20+ section components. Differentiation is a future decision; collapsing now would force a bigger rewrite later. |
| 2026-04-23 | Darkened `--ink-2` and `--ink-3` from `#94877C` to `#4A3E32` and `#6B5B4C` | Prior taupe muted tokens sat at ~3.3:1 on `--bg`, failing WCAG AA for body text (4.5:1 minimum). New values pass AA/AAA while preserving the warm-brown identity. Primary `--ink-1` unchanged. Also normalized the last hardcoded `text-black` / `text-gray-*` occurrences in `client-logos.tsx`, `comparison.tsx`, `header.tsx`, and `brand-analysis-widget.tsx` to token references. |
| 2026-04-23 | FAQ accordion on shadcn `<Accordion>` (Radix) | Replaced the `useState`-driven custom `<summary>` implementation. Shadcn/Radix primitive gives proper ARIA/keyboard semantics, data-state-driven motion, and a clean card-pill visual. Trigger pins Telegraf via inline `style={{ fontFamily: 'var(--font-sans)' }}` to override the Radix `AccordionPrimitive.Header` which renders as `<h3>` (globally styled serif in `globals.css`). `text-lg md:text-xl` for prominence; `faqData` and the two-column layout on `/faq` preserved. |
| 2026-04-23 | Header auth/language group +30%, footer tagline clamped to 50% width | Prior `text-[10px]` auth row and `text-[9px]` language switcher were illegible; bumped to `text-[13px]` / `text-[12px]`. Footer tagline given `max-w-[50%]` so it reads as a short claim rather than filling the full two-column brand slot. |
| 2026-04-24 | External-link hover text color changed from `--brand-2` to `--ink-1` | Lime on hover was doing double duty — off-site signal AND hover state. Splitting them: default underline stays lime (signal), hover darkens underline + text to `--ink-1` (state change). Cleaner semantics, less flicker between "is this external?" and "am I hovering?" |
| 2026-04-24 | Latest Activity cards (home `status-board.tsx`): removed 1px `--line` border, pinned title font to Telegraf sans | Borderless white fill reads as a cleaner list inside the hero-side column; the 1px stroke was competing visually with the category pill. Title `<h4>` forced to `var(--font-sans)` inline — Radix/global `h1-h6` rule renders serif by default, but dense UI labels in a card read better sans. Same override pattern used on shadcn `AccordionTrigger` (2026-04-23). |
| 2026-04-24 | `.tag-outline` border removed (1px `--line` → none) | Pill separation now comes from `--bg` fill against its container, not a stroke. Affects `project-card` tags on home Evidence section, plus `resources`, `tools-showcase`, and `evidence_hu` where the class is reused. Consistent with the "flat, no depth" system posture. |
| 2026-04-24 | New "Anthropic Early Access Programme" card pinned to top of home status board; old "Agentic AI for Non-Tech Users" card removed | That BUILDING entry had been stale ("see demo" pointing at `/design` — the point is better made by the affiliation itself). Mirrored in `status-updates-hu.ts`. |
