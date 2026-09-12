# Technical Requirement Document (TRD)
## ShiftServe — Catering & Banquet Workforce Booking Platform (MVP)

**Version:** 1.0 (Draft)
**Companion doc:** PRD-Catering-Booking-Platform.md
**Date:** August 21, 2026
**Audience:** Solo builder, non-technical → this doc includes step-by-step setup, not just architecture.

---

## 1. Scope of This Document

This TRD translates the PRD into a concrete, buildable system: final tech stack, database schema (with actual SQL), authentication design, payment/escrow architecture, real-time behavior, API structure, project folder layout, environment variables, and a step-by-step setup guide from zero to a deployed skeleton app.

## 2. Final Tech Stack

| Layer | Choice | Notes |
|---|---|---|
| Frontend framework | **Next.js 15** (App Router, TypeScript) | SSR for SEO, deploys natively on Vercel |
| UI/Styling | **Tailwind CSS** + **shadcn/ui** | Fast, clean, aesthetic components without a heavy design system |
| Backend/DB | **Supabase** (Postgres 15) | Auth, DB, Row-Level Security, Storage, Realtime, Edge Functions |
| Auth | **Supabase Auth** (Phone OTP via MSG91/Twilio + Email) | Native fit with DB, no separate auth service needed |
| Payments/Escrow | **Razorpay Route** | India-first, native UPI support, built for marketplace split-payments (holds funds, releases to linked accounts) — chosen over Stripe Connect since your users and payouts are India-based and UPI is the expected payout rail |
| Realtime | **Supabase Realtime** (Postgres change subscriptions) | Live vacancy counts, live booking confirmations |
| Maps/Geolocation | **Mapbox** (GL JS + Geocoding API) | Venue pin, distance filtering, geo check-in; generous free tier |
| SMS/OTP delivery | **MSG91** (India-focused, cheaper than Twilio for Indian numbers) | Used via Supabase's custom SMS provider hook |
| Transactional email | **Resend** | Booking confirmations, payout receipts |
| Hosting | **Vercel** | As planned |
| Analytics | **PostHog** (free self-serve tier) | Funnel: view → book → check-in → payout |
| Error monitoring | **Sentry** (free tier) | Catch production errors early, solo builder needs this |
| Version control/CI | **GitHub** + Vercel's built-in CI (auto-deploy on push) | No separate CI tool needed for MVP |

## 3. System Architecture Overview

```
┌─────────────────┐        ┌──────────────────────┐
│   Browser (Web)  │◄──────►│  Next.js App (Vercel) │
│  Company / Worker│        │  - SSR pages (SEO)     │
└─────────────────┘        │  - API routes          │
                            │  - Server components    │
                            └──────────┬──────────────┘
                                       │
                 ┌─────────────────────┼─────────────────────┐
                 ▼                     ▼                     ▼
        ┌────────────────┐   ┌─────────────────┐   ┌──────────────────┐
        │ Supabase        │   │ Razorpay Route   │   │ Mapbox / MSG91 /  │
        │ - Postgres DB   │   │ - Escrow hold    │   │ Resend / PostHog  │
        │ - Auth (OTP)    │   │ - Split payout   │   │ (third-party APIs)│
        │ - Storage       │   │ - Webhooks       │   └──────────────────┘
        │ - Realtime      │   └─────────────────┘
        │ - Edge Functions│
        └────────────────┘
```

**Flow in words:** The Next.js app (hosted on Vercel) is the only thing the browser talks to directly. It talks to Supabase for everything data/auth/realtime-related, and to Razorpay for payment collection and payout. Supabase Edge Functions handle server-side logic that must stay off the client (e.g., verifying Razorpay webhook signatures, releasing escrow, computing reliability scores).

## 4. Database Schema

Run this in the Supabase SQL Editor after project creation (see §9 for step-by-step). This is the MVP schema — extend later, don't over-engineer now.

```sql
-- ============================
-- ENUMS
-- ============================
create type user_role as enum ('company', 'worker', 'admin');
create type event_status as enum ('draft', 'open', 'full', 'in_progress', 'completed', 'cancelled');
create type booking_status as enum ('booked', 'checked_in', 'completed', 'no_show', 'cancelled');
create type escrow_status as enum ('unfunded', 'funded', 'partially_released', 'released', 'refunded');
create type txn_type as enum ('escrow_fund', 'payout', 'refund', 'platform_fee', 'commitment_hold', 'commitment_release', 'commitment_forfeit');
create type txn_status as enum ('pending', 'success', 'failed');

-- ============================
-- USERS (extends Supabase auth.users)
-- ============================
create table public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  role user_role not null,
  phone text unique not null,
  email text unique,
  full_name text,
  phone_verified boolean default false,
  created_at timestamptz default now()
);

create table public.company_profiles (
  user_id uuid primary key references public.profiles(id) on delete cascade,
  business_name text not null,
  business_type text, -- e.g. 'catering', 'event_management'
  address text,
  city text,
  rating_avg numeric(3,2) default 0,
  rating_count int default 0
);

create table public.worker_profiles (
  user_id uuid primary key references public.profiles(id) on delete cascade,
  dob date,
  city text,
  lat double precision,
  lng double precision,
  role_preferences text[] default '{}', -- e.g. {'server','kitchen_helper'}
  upi_id text,
  bank_account_number text,
  bank_ifsc text,
  id_doc_url text, -- Supabase Storage path, optional self-upload
  is_verified_badge boolean default false,
  reliability_score numeric(5,2) default 100.00, -- % of committed shifts completed
  rating_avg numeric(3,2) default 0,
  rating_count int default 0,
  strikes_count int default 0,
  strike_cooldown_until timestamptz
);

-- ============================
-- EVENTS ("Workers Needed" cards)
-- ============================
create table public.events (
  id uuid primary key default gen_random_uuid(),
  company_id uuid not null references public.company_profiles(user_id),
  title text not null,
  event_type text not null, -- wedding, birthday, corporate, etc.
  role_needed text not null, -- server, kitchen_helper, cleanup
  description text,
  venue_address text not null,
  city text not null,
  lat double precision,
  lng double precision,
  event_date date not null,
  start_time timestamptz not null,
  end_time timestamptz not null,
  pay_per_worker numeric(10,2) not null,
  vacancies int not null check (vacancies > 0),
  filled_count int default 0,
  reporting_time timestamptz,
  contact_person text,
  contact_phone text,
  status event_status default 'draft',
  escrow_status escrow_status default 'unfunded',
  platform_fee_amount numeric(10,2) default 0,
  total_escrow_amount numeric(10,2) default 0, -- (pay_per_worker * vacancies) + platform_fee_amount
  created_at timestamptz default now()
);

create index idx_events_city_date on public.events(city, event_date);
create index idx_events_status on public.events(status);

-- ============================
-- BOOKINGS
-- ============================
create table public.bookings (
  id uuid primary key default gen_random_uuid(),
  event_id uuid not null references public.events(id) on delete cascade,
  worker_id uuid not null references public.worker_profiles(user_id),
  status booking_status default 'booked',
  commitment_hold_amount numeric(10,2) default 0,
  commitment_hold_status text default 'none', -- none/held/refunded/forfeited
  checked_in_at timestamptz,
  checked_in_lat double precision,
  checked_in_lng double precision,
  completed_at timestamptz,
  no_show_marked_at timestamptz,
  cancelled_at timestamptz,
  cancellation_reason text,
  created_at timestamptz default now(),
  unique(event_id, worker_id)
);

create index idx_bookings_event on public.bookings(event_id);
create index idx_bookings_worker on public.bookings(worker_id);

-- ============================
-- TRANSACTIONS (payment ledger)
-- ============================
create table public.transactions (
  id uuid primary key default gen_random_uuid(),
  event_id uuid references public.events(id),
  booking_id uuid references public.bookings(id),
  user_id uuid references public.profiles(id), -- who it's from/to
  type txn_type not null,
  amount numeric(10,2) not null,
  razorpay_ref text,
  status txn_status default 'pending',
  created_at timestamptz default now()
);

-- ============================
-- RATINGS
-- ============================
create table public.ratings (
  id uuid primary key default gen_random_uuid(),
  event_id uuid not null references public.events(id),
  from_user uuid not null references public.profiles(id),
  to_user uuid not null references public.profiles(id),
  score int not null check (score between 1 and 5),
  comment text,
  created_at timestamptz default now(),
  unique(event_id, from_user, to_user)
);

-- ============================
-- NOTIFICATIONS
-- ============================
create table public.notifications (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references public.profiles(id),
  type text not null, -- booking_confirmed, shift_reminder, payout_sent, etc.
  title text not null,
  body text,
  payload jsonb,
  read_at timestamptz,
  created_at timestamptz default now()
);

-- ============================
-- DISPUTES (lightweight, MVP)
-- ============================
create table public.disputes (
  id uuid primary key default gen_random_uuid(),
  booking_id uuid not null references public.bookings(id),
  raised_by uuid not null references public.profiles(id),
  reason text not null,
  status text default 'open', -- open/resolved/rejected
  admin_notes text,
  created_at timestamptz default now(),
  resolved_at timestamptz
);
```

### 4.1 Row-Level Security (RLS) — critical, do not skip

```sql
alter table public.profiles enable row level security;
alter table public.company_profiles enable row level security;
alter table public.worker_profiles enable row level security;
alter table public.events enable row level security;
alter table public.bookings enable row level security;
alter table public.transactions enable row level security;
alter table public.ratings enable row level security;
alter table public.notifications enable row level security;
alter table public.disputes enable row level security;

-- Profiles: users can read/update only their own row; public read of minimal fields via a view (see §4.2)
create policy "own profile read" on public.profiles for select using (auth.uid() = id);
create policy "own profile update" on public.profiles for update using (auth.uid() = id);

-- Company profiles: owner can update; anyone authenticated can read (needed for worker to see company rating)
create policy "company read" on public.company_profiles for select using (true);
create policy "company update own" on public.company_profiles for update using (auth.uid() = user_id);

-- Worker profiles: owner can update; company can read only workers who applied to their events (enforced via join logic in app layer + a limited public view for reliability_score/rating)
create policy "worker read own" on public.worker_profiles for select using (auth.uid() = user_id);
create policy "worker update own" on public.worker_profiles for update using (auth.uid() = user_id);

-- Events: anyone authenticated can read open events; only owning company can insert/update their events
create policy "events public read" on public.events for select using (status in ('open','full','in_progress','completed'));
create policy "events owner read" on public.events for select using (auth.uid() = company_id);
create policy "events owner write" on public.events for insert with check (auth.uid() = company_id);
create policy "events owner update" on public.events for update using (auth.uid() = company_id);

-- Bookings: worker sees own bookings; company sees bookings for their events
create policy "bookings worker read" on public.bookings for select using (auth.uid() = worker_id);
create policy "bookings company read" on public.bookings for select using (
  auth.uid() in (select company_id from public.events where events.id = bookings.event_id)
);
create policy "bookings worker insert" on public.bookings for insert with check (auth.uid() = worker_id);
create policy "bookings worker update own" on public.bookings for update using (auth.uid() = worker_id);
create policy "bookings company update" on public.bookings for update using (
  auth.uid() in (select company_id from public.events where events.id = bookings.event_id)
);

-- Transactions: only visible to the involved user; writes only via Edge Functions (service role), not client
create policy "transactions own read" on public.transactions for select using (auth.uid() = user_id);

-- Ratings: readable by anyone (public trust signal); insertable only by participants of a completed event
create policy "ratings public read" on public.ratings for select using (true);
create policy "ratings insert own" on public.ratings for insert with check (auth.uid() = from_user);

-- Notifications: only the owner
create policy "notifications own" on public.notifications for select using (auth.uid() = user_id);
create policy "notifications update own" on public.notifications for update using (auth.uid() = user_id);

-- Disputes: raised_by or the counterparty on the booking can read
create policy "disputes involved read" on public.disputes for select using (auth.uid() = raised_by);
create policy "disputes insert own" on public.disputes for insert with check (auth.uid() = raised_by);
```

**Important:** Anything that touches money (funding escrow, releasing payouts, marking commitment holds forfeited) must happen through a **Supabase Edge Function using the service role key**, never directly from the client — this is non-negotiable, since RLS alone cannot safely gate financial writes.

### 4.2 Public-safe views

To let a company see a worker's reliability score without exposing their bank details or phone number, create a restricted view:

```sql
create view public.worker_public_profile as
select user_id, city, role_preferences, reliability_score, rating_avg, rating_count, is_verified_badge
from public.worker_profiles;
```

## 5. Authentication Design

- **Sign-up:** phone number → Supabase Auth sends OTP via SMS (configure MSG91 as custom SMS provider in Supabase Auth settings, since default Twilio pricing is higher for Indian numbers) → on verification, insert a row into `profiles` with role = 'company' or 'worker' (chosen at signup) → redirect to role-specific onboarding (company: business details; worker: profile + role preference + bank/UPI).
- **Session handling:** Supabase Auth's JWT, stored in an httpOnly cookie via `@supabase/ssr` package (Next.js App Router pattern) — do NOT use localStorage for the session token.
- **Route protection:** Next.js middleware checks session + role on every request to `/company/*` and `/worker/*` route groups, redirecting unauthenticated users to `/login`.

## 6. Payment & Escrow Architecture (Razorpay Route)

### 6.1 Why Razorpay Route
It's built specifically for marketplaces: the platform collects one payment from the company, and Razorpay can programmatically split/route parts of it to multiple linked accounts (each worker) later, or the platform can hold funds and trigger transfers on-demand — which matches your "fund now, release after event" model. UPI-native, which matters since workers will expect UPI payouts, not bank wire.

### 6.2 Flow
1. **Company posts event** → app calculates `total_escrow_amount = (pay_per_worker × vacancies) + platform_fee_amount`.
2. **Company pays via Razorpay Checkout** (standard order + payment flow) for `total_escrow_amount`. This money sits in the platform's Razorpay account balance — it is not yet linked to any specific worker.
3. Razorpay sends a **webhook** (`payment.captured`) to a Supabase Edge Function (`/functions/v1/razorpay-webhook`), which verifies the signature (using Razorpay's webhook secret) and updates `events.escrow_status = 'funded'` and inserts a `transactions` row (`type = 'escrow_fund'`).
4. **Workers book** shifts against that event (no money moves yet at booking, aside from the optional small commitment hold — see §6.4).
5. **Event day:** workers check in (§7). Company can mark no-shows and pull from the backup queue.
6. **Post-event release:** company clicks "Confirm event completed," or a scheduled Edge Function (cron via `pg_cron` in Supabase) auto-releases 24–48h after `end_time` if no dispute is open.
7. Release logic: for each `booking.status = 'checked_in'` or `'completed'`, an Edge Function calls **Razorpay Route's transfer API** to move that worker's `pay_per_worker` to their linked account (linked accounts created for each worker's UPI ID at profile setup via Razorpay's Linked Account API — this needs each worker to complete Razorpay's lightweight KYC for payouts, which is a Razorpay requirement, not a platform choice). For no-show bookings, that portion is refunded to the company instead.
8. Every transfer/refund writes a `transactions` row for audit trail.

### 6.3 Worker payout account setup
Razorpay Route requires each payee to have a "Linked Account" with minimal KYC (name, bank/UPI, PAN optional for small amounts). Build a simple in-app step: "Add your UPI ID to get paid" → calls Razorpay's Linked Account creation API server-side (Edge Function) → store the returned `linked_account_id` against `worker_profiles`.

### 6.4 Commitment hold (optional, recommended)
If implemented: at booking, a small amount (e.g., ₹20–50) is charged to the worker via a saved payment method or UPI autopay mandate, held as `commitment_hold`, and auto-refunded on check-in. This adds real payment complexity (recurring/pre-auth mandates) — **recommend deferring this to post-MVP** and instead using the reliability score + strike system alone for v1, since it avoids adding UPI mandate integration complexity for a first launch. Flagging this as a scope trade-off for you to confirm.

## 7. Real-Time Behavior (Supabase Realtime)

Since you want live updates, use Supabase's Postgres change subscriptions:

- **Company dashboard:** subscribes to `bookings` table filtered by `event_id`, so `filled_count` and the applicant list update live as workers book, without a page refresh.
- **Worker browsing view:** subscribes to `events` table so a card's remaining vacancy count updates live, and cards move from "open" to "full" in real time, preventing workers from trying to book a just-filled shift.
- **Check-in updates:** company dashboard subscribes to `bookings.status` changes on event day to show live check-in progress ("12/15 checked in") during the shift.

Implementation pattern (client-side, in a React Server/Client boundary component):
```javascript
const channel = supabase
  .channel(`event-${eventId}`)
  .on('postgres_changes',
    { event: '*', schema: 'public', table: 'bookings', filter: `event_id=eq.${eventId}` },
    (payload) => { /* update local state */ }
  )
  .subscribe();
```

## 8. Application Structure (Next.js App Router)

```
/app
  /(public)
    /page.tsx                     -- landing page
    /events/page.tsx               -- browse all events (SEO: SSR, filters via query params)
    /events/[city]/[eventId]/page.tsx  -- individual event detail (SSR, JobPosting schema markup)
    /login/page.tsx
    /signup/page.tsx
  /company
    /dashboard/page.tsx
    /events/new/page.tsx
    /events/[eventId]/page.tsx     -- manage applicants, check-ins, backup queue
    /events/[eventId]/payment/page.tsx  -- Razorpay checkout
  /worker
    /dashboard/page.tsx            -- my bookings, upcoming shifts
    /browse/page.tsx               -- filterable event list
    /profile/page.tsx              -- profile + UPI setup
  /admin
    /disputes/page.tsx
    /users/page.tsx
  /api
    /webhooks/razorpay/route.ts    -- (or Supabase Edge Function, see note below)
/components
  /ui                              -- shadcn components
  /events (EventCard, EventFilterBar, VacancyBadge, etc.)
  /bookings
/lib
  /supabase (client.ts, server.ts, middleware.ts)
  /razorpay (client.ts, webhook-verify.ts)
/supabase
  /functions
    /razorpay-webhook
    /release-escrow
    /compute-reliability-score
  /migrations (schema.sql, rls.sql)
```

**Note on webhooks:** Razorpay webhooks can be received either by a Next.js API route (`/app/api/webhooks/razorpay/route.ts`) or a Supabase Edge Function. Recommend the **Supabase Edge Function** so it has direct, low-latency access to the service-role DB client and keeps financial logic out of the general app deployment.

## 9. Step-by-Step Setup Guide (from zero)

### 9.1 Accounts to create
1. GitHub account (if you don't have one) — for code + connecting to Vercel.
2. Supabase account → supabase.com → "New Project" → choose region closest to India (Singapore) → note down the **Project URL** and **anon public key** and **service_role key** (Settings → API).
3. Vercel account → vercel.com → sign in with GitHub.
4. Razorpay account → razorpay.com → complete business KYC (needed to activate Route/marketplace features — this can take a few days, start this early).
5. Mapbox account → mapbox.com → get a public access token (free tier: 50,000 loads/month).
6. MSG91 account → msg91.com → for SMS OTP delivery.
7. Resend account → resend.com → for transactional emails.
8. PostHog account (optional at first) → posthog.com.
9. Sentry account (optional at first) → sentry.io.

### 9.2 Local project scaffold
```bash
npx create-next-app@latest shiftserve --typescript --tailwind --app
cd shiftserve
npx shadcn@latest init
npm install @supabase/supabase-js @supabase/ssr
npm install razorpay
npm install mapbox-gl
```

### 9.3 Environment variables (`.env.local`)
```
NEXT_PUBLIC_SUPABASE_URL=your-supabase-project-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key   # server-only, never expose to client
RAZORPAY_KEY_ID=your-razorpay-key-id
RAZORPAY_KEY_SECRET=your-razorpay-key-secret       # server-only
RAZORPAY_WEBHOOK_SECRET=your-webhook-secret         # server-only
NEXT_PUBLIC_MAPBOX_TOKEN=your-mapbox-public-token
MSG91_AUTH_KEY=your-msg91-key                       # configured in Supabase Auth settings, not app code
RESEND_API_KEY=your-resend-key
```

### 9.4 Supabase setup
1. In Supabase dashboard → SQL Editor → paste and run the schema from §4, then the RLS policies from §4.1, then the view from §4.2.
2. Authentication → Providers → enable Phone, configure MSG91 as the custom SMS provider (Supabase supports custom SMS hooks via Auth Hooks / a small Edge Function that calls MSG91's API).
3. Storage → create a bucket `id-documents` (private) for optional worker ID uploads.
4. Edge Functions → deploy `razorpay-webhook`, `release-escrow`, `compute-reliability-score` (scaffolded via Supabase CLI: `supabase functions new razorpay-webhook`).
5. Database → Extensions → enable `pg_cron` for the scheduled auto-release job.

### 9.5 Vercel setup
1. Push your Next.js project to a GitHub repo.
2. In Vercel → "Add New Project" → import the GitHub repo.
3. Add all `NEXT_PUBLIC_*` and server-only env vars in Vercel's Project Settings → Environment Variables (mark service-role/secret keys as "Sensitive").
4. Deploy — Vercel auto-builds on every push to `main`.
5. Add your custom domain under Project Settings → Domains once you have one (matters for SEO — launch on the real domain, not a `.vercel.app` subdomain, if possible).

### 9.6 Razorpay Route setup
1. Once KYC is approved, enable "Route" under Razorpay Dashboard → Route settings.
2. Generate API keys (test mode first) → add to env vars.
3. Set up the webhook URL pointing to your deployed Supabase Edge Function endpoint, subscribe to `payment.captured`, `payment.failed`, `transfer.processed` events.
4. Test the full flow in Razorpay's test mode before going live (test UPI IDs are provided in their docs).

## 10. SEO Technical Implementation

- Use Next.js `generateMetadata` per page for dynamic titles/descriptions (e.g., "Catering Staff Jobs in Chennai — Book Verified Servers | ShiftServe").
- Add **JobPosting structured data** (schema.org) on each event detail page — this is directly applicable since these are real gig postings, and can get listings surfaced in Google's job search features.
- Generate `sitemap.xml` dynamically via Next.js's `app/sitemap.ts`, including all open event pages and city/role landing pages.
- Server-render (not client-render) the public event listing and detail pages so content is crawlable.
- `robots.txt` allowing crawl of public pages, disallowing `/company/*`, `/worker/*`, `/admin/*`.

## 11. Non-Functional Requirements

| Category | Requirement |
|---|---|
| Performance | Public pages should target <2.5s LCP (Core Web Vitals) — use Next.js Image optimization, SSR caching where possible |
| Scalability | MVP target: a few thousand monthly active users, few hundred concurrent — Supabase's default tier handles this comfortably |
| Availability | Vercel + Supabase managed infra gives you reasonable default uptime for MVP; no custom HA setup needed yet |
| Security | RLS on all tables, service-role key never shipped to client, webhook signature verification mandatory, HTTPS everywhere (default on Vercel) |
| Data privacy | ID documents stored in a private Supabase Storage bucket, signed URLs only, never public |
| Accessibility | Use shadcn's accessible components as a baseline; ensure form labels, contrast ratios meet WCAG AA |

## 12. Monitoring & Logging

- **Sentry** wired into the Next.js app for client + server error capture.
- **Supabase's built-in logs** (Postgres logs, Auth logs, Edge Function logs) for backend debugging.
- **PostHog** for product funnel tracking (post event → view → book → check-in → payout completed) — critical for a solo builder to know where users drop off.

## 13. Testing Strategy (lean, for solo MVP)

- Manual test scripts for the two critical money-paths: (1) company funds escrow → worker books → check-in → payout release; (2) no-show → refund logic — run these end-to-end in Razorpay test mode before every deploy that touches payment code.
- Basic unit tests (Vitest) for reliability score calculation and payout-splitting logic, since bugs there cost real money.
- No need for full E2E automation (e.g., Playwright) at MVP stage given solo bandwidth — revisit once you have a co-founder/engineer or real transaction volume.

## 14. Key Technical Risks

- **Razorpay Route KYC/activation delay** — start this the same day you start building, since approval can take days and blocks the entire payment flow.
- **Webhook reliability** — payment status must never depend solely on the client redirect after checkout; always reconcile via webhook + a periodic Edge Function that checks pending transactions against Razorpay's API as a safety net.
- **RLS misconfiguration** — a single wrong policy can either leak data or lock out legitimate users; test every policy with multiple role accounts before launch.
- **Realtime at scale** — Supabase Realtime free tier has connection limits; monitor usage as user count grows and plan for a paid tier upgrade if needed.
