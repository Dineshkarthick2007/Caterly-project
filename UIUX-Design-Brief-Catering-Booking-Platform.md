# UI/UX Design Brief
## ShiftServe — Catering & Banquet Workforce Booking Platform (MVP)

**Version:** 1.0 (Draft)
**Prepared for:** UI/UX Designer
**Companion docs to send alongside this brief:** PRD-Catering-Booking-Platform.md, AppFlow-Catering-Booking-Platform.md (the TRD is optional background, not needed for design work)
**Date:** August 21, 2026

> **Note to designer:** This brief covers visual direction and interaction principles. The **App Flow document** is your source of truth for the full screen list, navigation structure, user journeys, edge cases, and modal/redirect behavior — read that alongside this one before starting. This brief tells you *how it should look and feel*; the App Flow doc tells you *what screens exist and how they connect*.

---

## 1. Product Context (why this matters to the design)

ShiftServe replaces an exploitative, informal broker chain in the catering industry with a transparent booking platform. The single most important feeling the design needs to earn is **trust** — a catering company is about to pay real money upfront into escrow, and a worker is trusting the platform (not a familiar local broker) with their income. The design should read as premium and dependable, closer to a fintech or professional booking platform than a casual gig-app. This is not a playful consumer app — treat it with the seriousness of a payments product, but keep it warm and human, since the end users on the worker side are ordinary people, not enterprise buyers.

## 2. Overall Aesthetic Direction

**Direction: Premium, minimalist, quietly confident.** Not corporate-cold, not playful/gig-app-casual. Think of the calm authority of a well-run banquet hall — precise, warm-lit, nothing fussy — rather than a bright, cheerful marketplace app.

Explicitly avoid:
- Blue/green color theming (per your instruction — also the two most overused colors in fintech/gig apps, so avoiding them helps this feel distinct).
- Bright, saturated "gig economy" playfulness (illustrated mascots, bouncy rounded shapes, candy colors).
- Generic SaaS-dashboard blandness (flat grey-on-white with a single default blue accent).
- Cliché "AI-generated startup" looks: cream background with terracotta accent, or pure black with a single neon accent — both have become defaults precisely because they're overused; this brand should be identifiable as itself.

Reach instead for the material world of catering and banquets for inspiration — brass, linen, porcelain, dark wood, candlelight — translated into a restrained digital palette. This gives "premium hospitality" a visual language without becoming literal/decorative about it.

## 3. Color Palette

A warm, neutral-forward palette with a single confident accent, avoiding blue/green entirely:

| Role | Color | Hex (starting point — designer has license to refine) | Usage |
|---|---|---|---|
| Background (primary) | Porcelain White | `#FAF8F5` | Main app background, light mode |
| Background (dark section) | Espresso Charcoal | `#211D1A` | Footer, hero sections, dark-mode base, high-contrast dashboard panels |
| Primary text | Near-Black Ink | `#211D1A` | Body copy, headings on light backgrounds |
| Secondary text | Warm Grey | `#6B645C` | Captions, metadata, timestamps, helper text |
| Primary accent | Deep Burgundy | `#7A2E33` | Primary buttons, links, key CTAs, active states — evokes fine dining/hospitality without being literal |
| Secondary accent | Aged Brass | `#B08D57` | Highlights, badges (e.g., "Verified"), icons, dividers — used sparingly, never as a large fill |
| Success | Muted Olive | `#5C6B47` | Confirmations, "Paid," "Checked in" states — intentionally muted, not a bright green, to stay off the blue/green-heavy palette while still reading as positive |
| Warning | Warm Amber | `#B8722E` | No-show alerts, pending states, cooldown warnings |
| Error | Brick Red | `#9C3B2E` | Failed payments, validation errors — distinct enough from the burgundy accent to not be confused with a CTA |
| Border/divider | Hairline Grey | `#E4DFD8` | Card borders, table dividers, input outlines |

**Guidance for the designer:** treat Burgundy as the one color that means "act here" — it should appear only on primary buttons and active/selected states, never as decorative fill. Brass is a finishing touch (badges, icons, subtle accents) — if brass starts appearing on more than ~10% of a screen, pull it back. This restraint is what will make the palette read premium rather than themed.

## 4. Typography

Pair a characterful serif for display/headings with a clean grotesk for body and UI — this contrast (editorial warmth + functional clarity) reinforces "premium but usable," and avoids the generic feel of using one sans-serif everywhere.

| Role | Typeface direction | Notes |
|---|---|---|
| Display / Headings | A refined, slightly high-contrast serif — e.g., **Fraunces**, **Canela**, or **Ivar Text** (any in this family) | Used for page titles, hero headlines, event card titles. Should feel editorial, not decorative — restrained weight, not overly stylized |
| Body / UI text | A clean, humanist grotesk sans-serif — e.g., **General Sans**, **Inter**, or **Söhne** | Used for all body copy, form labels, buttons, navigation |
| Data / Tabular (numbers, pay amounts, tables) | A monospaced or tabular-figure variant — e.g., **IBM Plex Mono** or the sans's tabular-figure setting | Ensures pay amounts, dates, and dashboard stats align cleanly in columns — important for a product where money figures need to feel precise and trustworthy |

**Type scale (starting point, designer to refine in the actual design system):**

| Level | Size (desktop) | Weight | Use |
|---|---|---|---|
| Display / H1 | 40–48px | Serif, Medium | Landing page hero, page titles |
| H2 | 28–32px | Serif, Medium | Section headers |
| H3 | 20–22px | Sans, Semibold | Card titles, modal titles |
| Body | 16px | Sans, Regular | Default body copy |
| Small / Caption | 13–14px | Sans, Regular/Medium | Metadata, timestamps, helper text |
| Button/Label | 14–15px | Sans, Medium/Semibold | All interactive labels |

Line height: 1.5 for body text, 1.2 for display/headings. Maintain generous letter-spacing on all-caps labels (e.g., "OPEN," "CHECKED IN" status pills) since tight all-caps type reads cheap.

## 5. Component Style

- **Corners:** Softly rounded, not sharp, not pill-shaped — a **6–10px radius** on cards, inputs, and buttons. This reads modern and approachable without tipping into the bouncy/playful register of fully rounded (pill) buttons, which would undercut the premium tone.
- **Elevation:** Mostly **flat**, using hairline borders (`#E4DFD8`) rather than drop shadows to separate content — closer to a well-typeset print layout than a "floating cards" app. Reserve **soft, low-opacity shadows** for genuinely elevated elements only — modals, dropdowns, the booking confirmation card — so elevation actually communicates hierarchy instead of being applied everywhere by default.
- **Buttons:** Solid burgundy fill for primary actions; outlined (1px hairline, burgundy text) for secondary actions; text-only (no border) for tertiary/low-emphasis actions. Avoid gradients entirely.
- **Cards (Event Card is the most important component in the product):** Generous padding, clear visual hierarchy (event title in serif, key facts — date, pay, location — in a clean data row, vacancy count as a small badge). This card appears everywhere (browse list, dashboard, SEO pages) so it deserves the most design iteration of any single component.
- **Status pills/badges:** Small, rounded-rectangle pills with a dot indicator + label (e.g., ● Open, ● Full, ● Completed), using the semantic colors from §3 at low-opacity backgrounds with full-opacity text/dot.
- **Icons:** A single consistent icon set throughout (recommend Phosphor or Lucide — clean, geometric, consistent stroke-width) — never mix icon families.

## 6. Dark Mode / Light Mode

**Light mode is primary and must be designed first and fully** — this is the mode most users (especially workers on mobile, in daylight/venue conditions) will actually use. **Dark mode is a nice-to-have for MVP, not a requirement** — if time allows, the designer can produce a dark variant using the Espresso Charcoal background (`#211D1A`) with the same Porcelain/Burgundy/Brass accents inverted appropriately, but this should not delay the light-mode design system. Flag dark mode as a Phase 2 deliverable if the timeline is tight.

## 7. Inspiration References

Rather than pointing at other gig-economy apps (Urban Company, Swiggy Genie, etc. — which tend toward the bright/casual register we're avoiding), look toward:

- **Premium hospitality/booking products:** Resy, SevenRooms, Tock — for how they handle reservation-like cards, confirmations, and a "this is a real commitment" tone without feeling corporate.
- **Fintech-adjacent trust design:** Mercury, Ramp — for how restrained color, confident typography, and generous whitespace communicate "you can trust us with money" without resorting to literal bank imagery.
- **Editorial/print-inspired digital design:** Cash App's early editorial work, or independent hospitality brand sites — for the serif+sans pairing and warm neutral palettes.

**Designer's discretion encouraged:** these are directional references for *tone*, not for literal layout copying — the brief wants a distinctive identity, not a lookalike of any one of these.

## 8. Key UI Patterns

| Pattern | Where used | Design notes |
|---|---|---|
| **Event Card** | Browse lists, dashboard, SEO pages | The most-repeated, most important component — see §5. Needs a compact variant (dashboard lists) and a full variant (browse grid) |
| **Multi-step form** | Create Event, Worker Onboarding | Clear step indicator (e.g., "Step 2 of 4"), persistent summary of prior steps if helpful, generous spacing — this is a moment to feel unhurried, not like a rushed checkout |
| **Dashboard summary cards** | Company/Worker/Admin dashboards | Small stat cards (e.g., "Active Events: 3," "Reliability Score: 96%") — data-forward, using the tabular numeral type from §4 |
| **Data table** | Applicant lists, transaction logs, admin panels | Hairline row dividers, generous row height (not cramped), sortable column headers, status pills inline |
| **Modal** | Booking confirmation, rate & review, cancel confirmations | Centered, max-width ~480–560px, soft shadow (one of the few places shadow is used), clear single primary action |
| **Slide-over drawer** | Notifications | Right-side slide-in, doesn't fully obscure the underlying screen, dismissible via overlay click or explicit close |
| **Sidebar (Company/Admin)** | Persistent left nav | Collapsible on smaller desktop widths, icon+label pattern, active state uses burgundy left-border or subtle fill, not a bright highlight |
| **Bottom tab bar (Worker, mobile)** | Persistent mobile nav | 5 items max, icon+label, active state in burgundy |
| **Empty states** | Any list/dashboard with no data yet | Should feel like an invitation, not a dead end — a short line of encouraging copy + a clear single CTA, small supporting illustration/icon (simple line-art, not cartoonish) |
| **Status/progress indicators** | Check-in console, event lifecycle | Visual progression (e.g., a simple stepped bar: Open → Full → In Progress → Completed) so users always know where an event stands at a glance |

## 9. Mobile Responsiveness Requirements

- **Worker-facing screens must be designed mobile-first** — browsing, booking, and check-in will overwhelmingly happen on a phone, often outdoors/at a venue with variable lighting, so contrast and tap-target size matter more than usual (see §10).
- **Company/Admin dashboards are desktop-first** but must degrade gracefully to tablet/mobile (a manager may check on the event day from a phone) — sidebar collapses to a bottom sheet or hamburger menu below a defined breakpoint.
- Design at three breakpoints minimum: **mobile (375px)**, **tablet (768px)**, **desktop (1280px+)**. Provide artboards for all three for every worker-facing screen; company/admin screens need mobile at least for the dashboard and check-in console (the two screens most likely to be used on-site).
- Tap targets minimum **44×44px** on all interactive elements in mobile layouts (buttons, checkboxes, filter chips).
- The public SEO pages (`/events`, `/jobs/[city]`) must be fully responsive and fast-loading on mobile, since a large share of organic search traffic will arrive on phones.

## 10. Accessibility Considerations

- **Contrast:** All text must meet **WCAG AA** (4.5:1 for body text, 3:1 for large text/headings) against its background. Double-check the Burgundy accent (`#7A2E33`) and Brass (`#B08D57`) against Porcelain White — brass in particular tends to fail contrast at small sizes, so reserve it for icons/borders/large elements rather than small text.
- **Color is never the only signal:** status pills use both a color *and* a text label (e.g., not just a colored dot) — this matters for colorblind users and also just makes the interface clearer generally.
- **Font size:** body text no smaller than 14px anywhere; never rely on sub-13px type for anything actionable.
- **Focus states:** every interactive element (buttons, links, form fields, cards that are clickable) needs a visible keyboard focus indicator — don't rely on default browser outlines, but don't remove them without replacing them either.
- **Form labels:** always visible, persistent labels on inputs (not placeholder-only text that disappears on focus/entry) — especially important for the payout/UPI setup screens where mistakes have real financial consequences.
- **Motion:** respect `prefers-reduced-motion` — any animated transitions (page loads, card reveals) should have a reduced/instant fallback.
- **Alt text / semantic structure:** while this is partly a build-phase concern, the designer should design with real heading hierarchy in mind (one H1 per page, logical H2/H3 nesting) rather than styling arbitrary text large for visual effect.

## 11. Motion & Micro-interactions

Keep motion purposeful and restrained — this is a trust-building product, not an entertainment app:

- Subtle hover states on cards and buttons (slight elevation or border color shift, not scale/bounce effects).
- A brief, satisfying confirmation moment on key actions — booking a shift, funding escrow, checking in — a small success animation (checkmark draw-in, brief color pulse) reinforces "this worked," which matters a lot for a first-time user handing over money or committing to a shift.
- Real-time updates (live vacancy count, live applicant list per the App Flow doc) should animate in gently (fade/slide, ~150–200ms) rather than jarringly popping in, so the interface feels alive without feeling twitchy.
- No decorative animation beyond this — no parallax, no auto-playing illustrations, nothing that could read as trying too hard.

## 12. What Else the Designer Needs (Beyond This Brief)

Since the designer is only handling UI/UX (not build), here's the complete package they need from you to work without gaps:

1. **This UI/UX Design Brief** — visual direction and interaction principles (this document).
2. **App Flow Document** — full screen inventory, navigation structure, user journeys, edge cases, modal behavior, redirect logic. The designer needs this to know *every* screen and state to design (including empty/loading/error states listed in §7 of that doc) — don't let them design only the "happy path" screens.
3. **PRD** — for product context, personas, and the "why" behind features, so design decisions (e.g., how prominently to show the fee breakdown) are made with the right priorities in mind.
4. **Content/copy direction:** decide whether you'll write UI copy (button labels, empty-state text, error messages) yourself or want the designer/a copywriter to draft it. If you want the designer to write it, explicitly ask them to follow the "Writing in Design" principles: plain language, active voice, no filler, described from the user's point of view (e.g., "Add your UPI ID to get paid" not "Configure payout method").
5. **Real (or realistic) sample content:** actual-feeling event titles, pay amounts, worker names/profiles, company names — designing with "Lorem Ipsum" or placeholder data ("Event 1," "₹XXX") tends to produce weaker layout decisions than designing with real-feeling content from day one.
6. **Deliverable format expectations:** clarify upfront whether you need a full Figma file with: (a) a documented design system/component library (colors, type, spacing tokens, button/input/card components with all states — default/hover/active/disabled/error), (b) every screen from the App Flow inventory at the three breakpoints specified in §9, and (c) a clickable prototype covering the key journeys from the App Flow doc, so you (and any future developer) can click through the experience before build starts.
7. **Logo/brand mark:** if you don't already have one, this needs to be scoped as part of the engagement — it isn't covered by this brief and affects the color/type choices above if a strong existing mark already exists.
8. **Icon and illustration needs:** confirm whether custom illustrations are wanted for empty states/onboarding (recommended: simple, warm line-art consistent with §7's tone) or whether a stock icon set (§5) is sufficient for MVP — custom illustration adds cost/time, so decide this consciously rather than by default.
9. **Review checkpoints:** recommend at least two structured review points — (a) after the design system + 2–3 key screens (Event Card, Company Dashboard, Worker Browse) are drafted, before the designer builds out every remaining screen, and (b) after the full clickable prototype, before handoff to development — so misalignment is caught early rather than after everything is built.
