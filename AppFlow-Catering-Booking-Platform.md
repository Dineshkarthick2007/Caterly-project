# App Flow Document
## ShiftServe — Catering & Banquet Workforce Booking Platform (MVP)

**Version:** 1.0 (Draft)
**Companion docs:** PRD-Catering-Booking-Platform.md, TRD-Catering-Booking-Platform.md
**Date:** August 21, 2026

---

## 1. Navigation Philosophy (decision)

Company and Worker have different usage contexts, so navigation differs by role rather than forcing one pattern everywhere:

- **Company & Admin (desktop-first, data-dense work):** collapsible **left sidebar** — these users are managing multiple events, applicant lists, and dashboards, which benefits from persistent, scannable navigation.
- **Worker (mobile-first, on-the-go browsing):** **top navbar on desktop, bottom tab bar on mobile** — workers are checking shifts between other activities, often on a phone, so a thumb-reachable tab bar beats a sidebar.
- **Public/marketing pages:** simple **top navbar** with Login/Signup CTA, no sidebar.

All authenticated areas use a persistent **back button / breadcrumb** on detail pages (e.g., inside an event detail, "← Back to Dashboard"), since deep-linking (via SEO pages, notifications) means users won't always arrive via in-app navigation.

## 2. Screen Inventory

### 2.1 Public / Unauthenticated

| Screen | Route | Description |
|---|---|---|
| Landing Page | `/` | Hero, value prop for both sides, "Post an Event" and "Find Shifts" CTAs, how-it-works, trust/transparency messaging |
| Browse Events (public) | `/events` | SSR list of open events, filterable — visible without login, but booking requires signup (SEO entry point) |
| Event Detail (public) | `/events/[city]/[eventId]` | Full event card details, "Book this shift" CTA prompts login if not authenticated |
| City/Role Landing Pages | `/jobs/[city]`, `/jobs/[role]` | SEO landing pages, e.g. "Catering Staff Jobs in Chennai" |
| Login | `/login` | Phone number + OTP |
| Signup | `/signup` | Phone number → OTP → role toggle (Company / Worker) → routes into onboarding |
| Forgot/Resend OTP | inline on `/login` | Not a separate screen; resend link + timer |
| About / How it Works | `/how-it-works` | Explains transparent pay model, no-broker promise |
| Terms / Privacy | `/terms`, `/privacy` | Standard legal pages |

### 2.2 Auth & Onboarding (shared shell, role-branches after signup)

| Screen | Route | Description |
|---|---|---|
| OTP Verification | `/signup/verify` | 6-digit OTP input, auto-focus, resend timer |
| Role Selection | `/signup/role` | "I'm hiring staff" vs "I want to work shifts" toggle |
| Company Onboarding — Business Details | `/onboarding/company` | Business name, type, address, city |
| Worker Onboarding — Profile | `/onboarding/worker/profile` | Name, DOB, city, role preferences (multi-select: server, kitchen helper, cleanup) |
| Worker Onboarding — Payout Setup | `/onboarding/worker/payout` | UPI ID entry (Razorpay Linked Account creation) — can be marked "Skip for now," but booking is blocked until completed |
| Onboarding Complete | (transient, auto-redirects) | Brief success state → redirects into role-specific dashboard |

### 2.3 Company (Authenticated)

| Screen | Route | Description |
|---|---|---|
| Company Dashboard | `/company/dashboard` | Overview: active events, upcoming events needing attention (no-shows, unfilled vacancies), recent activity feed |
| Create Event | `/company/events/new` | Multi-step form: event details → role & pay → vacancies → review & fund |
| Event Payment (Fund Escrow) | `/company/events/[eventId]/fund` | Razorpay checkout, shows fee breakdown transparently |
| Event Detail / Manage | `/company/events/[eventId]` | Applicant list, live filled-count, worker profiles (reliability score, rating), backup queue |
| Event Day — Check-in Console | `/company/events/[eventId]/checkin` | Live checklist to mark workers checked-in, trigger backup queue notifications for no-shows |
| Event Complete / Confirm | `/company/events/[eventId]/complete` | "Confirm event completed" action → triggers payout release |
| Company Profile / Settings | `/company/settings` | Business details, saved payment methods, notification preferences |
| Company Ratings Received | `/company/settings/ratings` | Reviews left by workers |
| Past Events | `/company/events/history` | List of completed/cancelled events, downloadable payout summary |

### 2.4 Worker (Authenticated)

| Screen | Route | Description |
|---|---|---|
| Worker Dashboard | `/worker/dashboard` | Upcoming booked shifts, reliability score, quick stats (shifts completed, total earned) |
| Browse Shifts | `/worker/browse` | Filterable list (date, location, pay, role) — same data as public `/events` but with one-tap booking |
| Shift Detail | `/worker/shifts/[eventId]` | Full details + company rating + "Book this shift" button |
| Booking Confirmation | (modal, not a route) | Confirms booking, shows commitment terms if applicable |
| My Bookings | `/worker/bookings` | Tabs: Upcoming / Past / Cancelled |
| Shift Check-in | `/worker/shifts/[eventId]/checkin` | Geo/QR check-in screen, only active near shift start time and venue location |
| Worker Profile / Settings | `/worker/profile` | Personal details, role preferences, UPI/payout settings, optional ID upload |
| Earnings / Payout History | `/worker/earnings` | List of completed shifts with payout status and amount |
| Ratings Given/Received | `/worker/profile/ratings` | Reviews about this worker + reviews they've left |

### 2.5 Shared (Both Roles)

| Screen | Route | Description |
|---|---|---|
| Notifications | `/notifications` (or slide-over drawer) | Booking confirmations, reminders, payout alerts, dispute updates |
| Rate & Review | (modal, post-completion) | Triggered after event completion, prompts both sides to rate each other |
| Dispute Raise Form | `/disputes/new?bookingId=` | Simple form: reason + description, attaches to a specific booking |
| Dispute Status | `/disputes/[disputeId]` | Read-only status view for the person who raised it |

### 2.6 Admin

| Screen | Route | Description |
|---|---|---|
| Admin Dashboard | `/admin` | Platform-wide metrics: active events, GMV, open disputes, flagged users |
| Disputes Queue | `/admin/disputes` | List of open disputes, sortable by age/severity |
| Dispute Resolution | `/admin/disputes/[disputeId]` | Full context (booking, chat history if any, both parties' info), resolve/reject actions |
| User Management | `/admin/users` | Search/filter companies & workers, view strikes, manually verify/suspend accounts |
| Event Oversight | `/admin/events` | All events across the platform, manually intervene if needed (e.g., force-cancel, manual escrow release) |
| Transaction Log | `/admin/transactions` | Full ledger view for reconciliation with Razorpay |

## 3. Navigation Structure Detail

### Company & Admin — Sidebar
```
[Logo]
 Dashboard
 Events
   ├─ Create Event
   └─ Past Events
 Settings
 ─────────────
 [Notification bell icon — top right]
 [Profile menu — top right: Settings / Logout]
```

### Worker — Bottom Tab Bar (mobile) / Top Navbar (desktop)
```
[ Dashboard ] [ Browse ] [ My Bookings ] [ Earnings ] [ Profile ]
```

### Back Button / Breadcrumb Convention
- Any "detail" screen reached from a list (event detail, dispute detail, user detail) shows a `← Back to [Parent]` link at the top-left, not just browser back — this matters because many entry points are deep links from notifications or SEO pages, not always in-app navigation.
- Multi-step flows (Create Event, Onboarding) show a step indicator (e.g., "Step 2 of 4") with a `← Previous` action, not a generic back button, to avoid losing form state.

## 4. Entry Points — Where Users Land First

| Entry Source | Lands On |
|---|---|
| Direct visit / typed URL | `/` (Landing Page) |
| Google search (SEO — job listing) | `/events/[city]/[eventId]` or `/jobs/[city]` |
| "Post an Event" CTA (landing) | `/signup` (if unauthenticated) → onboarding → `/company/events/new` |
| "Find Shifts" CTA (landing) | `/signup` (if unauthenticated) → onboarding → `/worker/browse` |
| Returning user, has session | Role-based dashboard (`/company/dashboard` or `/worker/dashboard`) directly, skipping landing page |
| SMS/Email notification link (e.g., shift reminder) | Deep link directly to the relevant screen (e.g., `/worker/shifts/[eventId]`), with login gate if session expired |
| Admin | `/admin/login` (separate, not shown on public nav) → `/admin` |

## 5. Auth Flow — Signup → Onboarding → Dashboard

```
Landing Page
     │
     ▼
 /signup ──► Enter phone number ──► OTP sent (MSG91)
     │
     ▼
 /signup/verify ──► Enter OTP ──► [invalid] → inline error, retry (max 5 attempts, then 30s lockout)
     │ [valid]
     ▼
 /signup/role ──► Toggle: "I'm hiring staff" / "I want to work shifts"
     │
     ├── Company ──► /onboarding/company (business details form)
     │                     │
     │                     ▼
     │              /company/dashboard  (empty state: "Post your first event")
     │
     └── Worker ──► /onboarding/worker/profile (personal + role prefs)
                           │
                           ▼
                    /onboarding/worker/payout (UPI setup — "Skip for now" allowed)
                           │
                           ▼
                    /worker/dashboard  (empty state: "Browse shifts near you")
                           │
                           [if payout skipped] banner persists: "Add UPI to get paid — complete now"
```

**Returning user login:** `/login` → phone → OTP → session created → redirected straight to their role's dashboard (role stored in `profiles.role`, no re-selection needed).

**Session expiry mid-action:** if a user's session expires while trying to book/create/pay, the action's state is preserved (e.g., via URL params or a short-lived local draft) and they're bounced to `/login`, then redirected back to complete the original action after re-auth.

## 6. Key User Journeys

### Journey 1 — Company Posts an Event and Fills It
```
/company/dashboard → "Create Event" button
  → /company/events/new (Step 1: Event details — title, type, date/time, venue via Mapbox picker)
  → (Step 2: Role & pay — role needed, pay per worker, vacancies)
  → (Step 3: Contact & requirements — reporting time, contact person, uniform notes)
  → (Step 4: Review — shows fee breakdown: "Workers receive ₹X · Platform fee ₹Y · You pay ₹Z")
  → "Fund & Publish" → /company/events/[eventId]/fund (Razorpay checkout)
  → [payment success] → event status becomes "open", visible on /events
  → Company returns to /company/events/[eventId] → sees live applicant list fill in real time
  → [vacancies full] → status auto-updates to "full"
  → Event day → /company/events/[eventId]/checkin → marks arrivals, backup queue auto-notifies replacements for no-shows
  → Post-event → /company/events/[eventId]/complete → "Confirm Completed" → triggers payout release
  → Rate & Review modal appears → rates each worker
```

### Journey 2 — Worker Discovers and Completes a Shift
```
/worker/dashboard (or arrives via SEO page /events/chennai/[eventId])
  → /worker/browse → applies filters (date, city, pay range, role)
  → taps an EventCard → /worker/shifts/[eventId]
  → "Book this Shift" → confirmation modal (shows exact pay, commitment terms) → confirms
  → booking appears in /worker/bookings (Upcoming tab)
  → reminder notification 24h and 2h before shift
  → on shift day, near venue + near start time → /worker/shifts/[eventId]/checkin becomes active
  → geo/QR check-in → status becomes "checked_in"
  → post-event, company confirms completion → payout auto-releases to worker's UPI
  → notification: "You've been paid ₹X for [Event]" → appears in /worker/earnings
  → Rate & Review modal appears → rates the company
```

### Journey 3 — No-Show Handling (Operational Edge Case, Core to the Product)
```
Event day, reporting time passes → worker hasn't checked in
  → Company's check-in console flags them: "Not checked in" with a countdown/grace period (e.g., 15 min)
  → Grace period expires → company taps "Mark as No-Show" → booking.status = 'no_show'
  → System auto-notifies next available workers from a ranked backup queue (based on reliability score + proximity)
  → Backup worker accepts via push/SMS notification → new booking created, vacancy refilled
  → No-show worker's reliability_score recalculated (Edge Function), strike added if unexcused
  → That portion of escrow is refunded to company if not replaced in time, or reallocated to the replacement worker if filled
```

## 7. Edge Cases, Empty States, Loading States

| Scenario | Handling |
|---|---|
| Worker has no bookings yet | Empty state on `/worker/dashboard` and `/worker/bookings`: illustration + "Browse open shifts near you" CTA |
| Company has no events yet | Empty state on `/company/dashboard`: "Post your first event" CTA, brief 3-step explainer |
| No events match filters | `/worker/browse` and `/events`: "No shifts match your filters — try widening your search" + a "Clear filters" button |
| Event fills up while worker is viewing it | Real-time subscription updates the button from "Book this Shift" to a disabled "Shift Full" state without needing a refresh |
| Payment fails during escrow funding | Redirect back to `/company/events/[eventId]/fund` with an inline error and "Retry Payment" — event stays in `draft`, not visible publicly, until funded |
| Worker tries to check in too early / wrong location | Check-in button disabled with helper text: "Check-in opens 30 min before shift start, at the venue" |
| Session expired mid-form (e.g., creating event) | Form data preserved in local state/draft; user re-authenticates and returns to the same step |
| Network/API error (generic) | Toast notification: "Something went wrong — please try again," with a retry action where applicable, never a silent failure |
| Loading states | Skeleton loaders for list views (EventCard skeletons on `/events`, `/worker/browse`), spinner + disabled button state during form submissions (e.g., "Publishing…", "Booking…") to prevent double-submits |
| Worker's payout setup incomplete | Persistent (dismissible-but-recurring) banner on `/worker/dashboard`: "Add your UPI ID to receive payouts" — booking still allowed, but a blocking modal appears if they try to book without it, since payout must be guaranteed before commitment |
| Company cancels a funded event | Confirmation modal warning about compensation-to-committed-workers policy before allowing cancellation; refund + partial compensation logic runs automatically |
| Dispute raised, blocks auto-release | Escrow release for that specific booking is held; rest of the event's payouts proceed normally; admin notified via `/admin/disputes` |
| Worker strike cooldown active | `/worker/browse` shows a banner: "You're temporarily restricted from booking until [date] due to missed shifts" — booking buttons disabled during cooldown |

## 8. Modal / Drawer / Overlay Interactions

| Interaction | Type | Trigger |
|---|---|---|
| Booking confirmation | Modal | Tapping "Book this Shift" |
| Rate & Review | Modal | Auto-triggered after event/shift completion, on next dashboard visit if not immediately available |
| Notifications | Slide-over drawer (right side) | Bell icon tap, any screen |
| Cancel Event confirmation | Modal | "Cancel Event" action on `/company/events/[eventId]` |
| Mark No-Show confirmation | Modal | "Mark as No-Show" action on check-in console |
| Raise Dispute | Modal (short form) or dedicated page for longer description | Triggered from a booking/event detail's "Report an Issue" link |
| Image/ID upload preview | Modal | On worker profile, previewing uploaded ID doc before submit |
| Logout confirmation | Small popover (not full modal) | Profile menu → Logout |

## 9. Redirect Logic Summary

| Action | Redirects To |
|---|---|
| Successful signup + role selection (Company) | `/onboarding/company` |
| Successful signup + role selection (Worker) | `/onboarding/worker/profile` |
| Onboarding complete (Company) | `/company/dashboard` |
| Onboarding complete (Worker) | `/worker/dashboard` |
| Login (existing user) | Role-based dashboard |
| Logout | `/` (landing page) |
| Event successfully funded & published | `/company/events/[eventId]` (management view) |
| Booking confirmed | `/worker/bookings` (Upcoming tab), toast: "Shift booked!" |
| Check-in successful | Stays on `/worker/shifts/[eventId]/checkin`, updates state to show "You're checked in" |
| Event marked complete by company | `/company/events/[eventId]/complete` → after processing → `/company/events/history` |
| Dispute submitted | `/disputes/[disputeId]` (status view) |
| Admin resolves dispute | `/admin/disputes` (back to queue) |
| Session expired anywhere | `/login?redirect=[original-path]` → after auth → original path |
| Unauthorized role access (e.g., worker hits a `/company/*` URL) | Redirected to their own dashboard with a toast: "You don't have access to that page" |

## 10. Notes for Implementation Priority

Given MVP scope, the highest-priority flows to build first (per PRD roadmap) are: **Auth/Onboarding → Company Create Event + Fund → Worker Browse + Book → Check-in → Payout Release**. Admin panel, disputes, and ratings can lag slightly behind since they support the core loop rather than gate it — but the check-in and no-show backup queue should not be deprioritized, since that's the operational core the PRD is solving for.
