# Product Requirement Document
## ShiftServe — Catering & Banquet Workforce Booking Platform (MVP)

**Version:** 1.0 (Draft)
**Owner:** [Your Name]
**Status:** Draft for review
**Date:** August 21, 2026

---

## 1. Problem Statement

In the event catering and banquet industry, an informal, multi-tier chain of verbal middlemen takes 50–75% in undisclosed commissions, leaving temporary service workers underpaid, while subjecting catering businesses to unpredictable attendance and last-minute labor shortages.

## 2. Problem Description

Catering businesses source temporary staff (students, unemployed youth, daily-wage gig workers) through informal local broker networks. Because the process is verbal and unrecorded, requirements pass through a multi-tier chain:

```
Catering Company → Broker A → Broker B → Broker C → Food Service Worker
```

Each broker skims an undisclosed commission simply for forwarding word-of-mouth requirements. A ₹1,000/shift budget can shrink to ₹250–300 by the time it reaches the worker, who has no visibility into the original rate, faces delayed cash payouts, and has no recourse.

For catering companies, verbal commitments carry no accountability — no-shows are common, there's no real-time visibility into confirmed staff, and no structured backup pipeline, causing floor mismanagement during peak service hours.

## 3. Vision

A transparent, two-sided marketplace where catering companies post verified event staffing needs with clear pay and role details, and workers directly discover, commit to, and get paid for shifts — with the platform replacing the broker chain, not adding another layer to it.

## 4. Goals & Non-Goals (MVP)

### Goals
- Let catering companies post an event staffing requirement in under 3 minutes.
- Let workers browse, filter, and book shifts near them with full pay transparency.
- Guarantee workers get paid the exact rate shown — no hidden cuts.
- Reduce no-show risk via commitment mechanics and a live backup queue.
- Handle payment end-to-end via escrow so trust doesn't depend on either party.

### Non-Goals (out of scope for MVP)
- Full HR/payroll compliance (PF, ESI, TDS filing) — flag as a future consideration, not MVP.
- Complex shift-swapping marketplace between workers.
- Native mobile apps (MVP is a responsive, SEO-friendly web app).
- Multi-language support beyond English (add regional languages post-MVP).
- In-app chat between company and worker (use platform-mediated notifications only, to keep trust/dispute handling simple).

## 5. Users & Personas

### Persona 1 — Rajesh, Catering Company Owner/Manager
Runs a mid-size catering business handling 8–15 events/month. Currently calls 2–3 brokers every time he needs 20 servers for a wedding, never knows who's actually confirmed until people show up (or don't). Wants: reliable headcount, transparent cost, zero-surprise labor bill.

### Persona 2 — Priya, College Student / Gig Worker
Looking for flexible weekend/evening shifts to earn extra income. Currently depends on a local contact who calls her last-minute and pays cash, often less than promised, sometimes days late. Wants: to see the actual pay before committing, get paid on time, build a track record.

## 6. Core User Flows

### 6.1 Catering Company — Post a Requirement ("Workers Needed" Card)
1. Sign up / log in (business).
2. Create event: title, event type (wedding/birthday/corporate/etc.), date & time (start–end), venue/location (with map pin), number of vacancies, role(s) needed (server, kitchen helper, cleanup, etc.), pay per worker per shift, any requirements (uniform, experience level), reporting time & contact person.
3. Publish → escrow the total payout amount (vacancies × pay) via Razorpay/Stripe at time of posting, or at a confirmation deadline (see §8.2 for MVP decision).
4. Track applicants/bookings in real time on a dashboard: confirmed count vs. vacancies, worker profiles & ratings.
5. Manage the shift: mark attendance (check-in) on event day, replace no-shows from an auto-suggested backup queue.
6. Post-event: confirm completion → escrow releases payout to each checked-in worker automatically.

### 6.2 Worker — Discover & Book a Shift
1. Sign up / log in (worker), complete profile (basic info, phone OTP verified, skills/role preference, location).
2. Browse/filter open "Event" cards by date, location, pay range, role.
3. View full details: exact pay, timing, venue, company rating.
4. Book/commit to a slot (may involve a small refundable commitment action — see §8.3).
5. Get shift reminders (24h, 2h before).
6. Check in at venue (QR/geo-tagged check-in) on event day.
7. Get paid automatically to linked account/UPI once company confirms shift completion (or automatically after a set window if no dispute is raised).
8. Rate the company; build a completed-shifts track record shown on their profile.

## 7. Feature List — MVP Scope

| # | Feature | Priority |
|---|---|---|
| 1 | Company & Worker auth (phone OTP + email) | P0 |
| 2 | Company profile (business name, type, location) | P0 |
| 3 | Worker profile (name, phone, location, role preference, bank/UPI details) | P0 |
| 4 | "Post Event / Workers Needed" card creation | P0 |
| 5 | Public event listing with filters (date, location, pay, role) | P0 |
| 6 | Booking/commit flow for workers | P0 |
| 7 | Company dashboard — applicants, confirmed count, vacancy tracker | P0 |
| 8 | Escrow payment collection at posting/confirmation | P0 |
| 9 | Automated payout release to workers post-event | P0 |
| 10 | Attendance / check-in (geo or QR-based) | P0 |
| 11 | No-show backup queue (auto-notify next available workers) | P1 |
| 12 | Ratings & reviews (both directions) | P1 |
| 13 | Notifications (SMS/email/push) for booking, reminders, payout | P0 |
| 14 | Basic dispute flag (worker/company can flag an issue before payout release) | P1 |
| 15 | SEO-friendly public pages (landing, city/role event listings) | P0 |
| 16 | Admin panel (manual dispute resolution, user moderation) | P0 |

## 8. Key Product Decisions (from your inputs)

### 8.1 Monetization — Transparent Commission
Platform charges a small, clearly disclosed commission (e.g., X%) on top of or within the posted pay, shown separately in the company's checkout summary. The worker always sees and receives the full "pay per worker" amount advertised on the card — the commission is the company's cost of using the platform, not a cut of the worker's earnings. This is the central trust promise that differentiates ShiftServe from the broker chain, so it should be reflected explicitly in the UI (e.g., "Worker receives: ₹1,000 · Platform fee: ₹80 · You pay: ₹1,080").

### 8.2 Payments — In-App Escrow
- Company funds the full shift payout (vacancies × pay + platform fee) into escrow at the time of posting, or once vacancies are filled — recommend **at posting**, since it also acts as a soft commitment filter (companies who won't actually pay are less likely to post fake listings).
- Funds are held by the platform (via Razorpay Route / Stripe Connect for split payments) until the event's completion is confirmed.
- Release trigger: company marks the event "completed" OR an auto-release timer (e.g., 24–48 hours after event end) fires if no dispute is raised — this prevents companies from indefinitely withholding payment.
- Refund logic: if a worker no-shows and is not replaced, that worker's earmarked portion is refunded to the company (not paid out); if the company cancels the event, workers who already committed get a small compensation (TBD %) and the rest is refunded.

### 8.3 Trust & Verification — Lean MVP Approach
Since full ID/KYC upload adds friction and slows worker onboarding (a key growth lever), recommend a graduated trust model:

**At signup (both sides):**
- Phone number OTP verification (mandatory) — this alone kills a large share of fake accounts and is nearly frictionless.
- Email verification (mandatory, secondary contact channel).

**Lightweight identity signal (no manual review needed for MVP):**
- Optional self-upload of a government ID photo (Aadhaar/PAN) and a selfie — stored but not manually verified at MVP stage; simply having it on file increases accountability and can be a prerequisite for withdrawing payouts above a threshold. Frame it as "Verified" badge (blue check) vs. unverified profile — verified profiles get priority visibility/booking preference.

**Behavioral trust (this is what actually solves your no-show problem):**
- A visible **reliability score** per worker: % of committed shifts actually completed, shown to companies before they see applicants.
- A visible **rating score** per company: how often they pay on time / event conditions match what was posted, shown to workers before booking.
- A **strike system**: 2 no-shows (unexcused, without 12h+ cancellation notice) in a rolling 30-day window temporarily restricts a worker from booking new shifts for a cooldown period. Same logic in reverse for companies who cancel late or under-pay vs. posted rate.
- Small refundable commitment hold (e.g., ₹20–50) taken from the worker at booking time, refunded automatically on check-in — forfeited (and credited toward the replacement worker's incentive) if they no-show without notice. This is optional for MVP v1 but strongly recommended as your single highest-leverage anti-no-show lever, since it mirrors what a real deposit does without needing manual enforcement.

This avoids building a KYC verification pipeline for MVP (expensive, slow, needs manual review team) while still directly attacking the two things that matter: fake postings (solved by escrow-at-posting) and no-shows (solved by reliability scores + commitment hold + backup queue).

## 9. Information Architecture / Data Model (high-level)

- **users** (id, role: company/worker, phone, email, verified_status, created_at)
- **company_profiles** (user_id, business_name, business_type, address, rating_avg)
- **worker_profiles** (user_id, name, dob, location, role_preferences[], bank/upi_id, reliability_score, rating_avg, id_doc_url, strikes_count)
- **events** (id, company_id, title, event_type, venue_address, lat/lng, date, start_time, end_time, role_needed, pay_per_worker, vacancies, filled_count, status: draft/open/full/in_progress/completed/cancelled, escrow_status)
- **bookings** (id, event_id, worker_id, status: booked/checked_in/completed/no_show/cancelled, commitment_hold_status, checked_in_at)
- **transactions** (id, event_id, worker_id/company_id, type: escrow_fund/payout/refund/fee, amount, gateway_ref, status)
- **ratings** (id, event_id, from_user, to_user, score, comment)
- **notifications** (id, user_id, type, payload, sent_at, read_at)

## 10. Tech Stack Recommendation

Given your Supabase + Vercel starting point, here's a coherent MVP stack:

| Layer | Choice | Why |
|---|---|---|
| Frontend framework | Next.js (App Router) | SSR/SSG for SEO-friendly public event pages; deploys natively on Vercel |
| Styling/UI | Tailwind CSS + shadcn/ui | Fast to build a clean, aesthetic UI without a heavy design system |
| Backend/DB | Supabase (Postgres) | Auth, DB, Row-Level Security (RLS) for company/worker data isolation, storage for ID docs |
| Auth | Supabase Auth (phone OTP + email) | Native fit, avoids building custom auth |
| Payments/Escrow | Razorpay Route (India-first) or Stripe Connect | Supports split payments/marketplace payouts, needed for escrow-then-release model |
| Geolocation/Maps | Mapbox or Google Maps API | Venue pinning, distance-based filtering, geo check-in |
| Notifications | Supabase Edge Functions + MSG91/Twilio (SMS), Resend (email), Web Push | Shift reminders, booking confirmations |
| Hosting | Vercel | Matches your existing plan |
| Analytics | PostHog or Plausible | Funnel tracking (post → view → book → complete) |

**SEO approach:** Next.js SSR for public event listing pages (`/events/[city]/[eventId]`), structured data (JobPosting schema markup — genuinely applicable here since these are gig listings), sitemap generation, and city/role landing pages (e.g., "Catering staff jobs in Chennai") for organic discovery — this also directly supports worker-side growth without paid acquisition.

## 11. Success Metrics (MVP)

- **Time-to-fill**: median time from event posting to all vacancies booked.
- **No-show rate**: % of confirmed bookings that don't check in — target meaningful reduction vs. informal-broker baseline (establish baseline via early user interviews).
- **Payout accuracy/speed**: % of workers paid within X hours of event completion, with zero discrepancy vs. posted rate.
- **Repeat usage**: % of companies posting a 2nd event within 30 days; % of workers completing a 2nd shift within 30 days.
- **GMV & take rate**: total payout volume processed, platform fee revenue.

## 12. Risks & Open Questions

- **Cold start problem**: need supply (workers) and demand (companies) simultaneously in a launch city — recommend concentrating on one city/region first.
- **Cash-in-hand competition**: informal brokers may still appeal to workers wanting instant cash rather than post-event bank transfer — consider instant/near-instant payout via UPI as a differentiator, not next-day settlement.
- **Regulatory**: gig worker payment platforms may eventually need to account for labor classification — flag for legal review, not a blocker for MVP.
- **Dispute resolution**: MVP relies on manual admin review for flagged disputes — needs a lightweight internal tool from day one, not deferred.
- **Commitment hold friction**: charging even a small deposit to low-income gig workers could depress signups — worth A/B testing before hard-coding it into the flow.


