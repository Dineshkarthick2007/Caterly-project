# Product Requirements Document (PRD): ShiftServe Platform

**Document Version:** 1.0.0  
**Status:** Approved / In Production  
**Primary Platform Targets:** Web Application (Desktop) & Mobile Application (Responsive Web / PWA / iOS & Android)  
**Design System:** ShiftServe Utility System (Tokens: Light Mode, Solid Marigold `#E8871E`, Clean Surface `#F8F9FF` / `#FAFAF8`, 8px Border Radius, Crisp Hairline Borders)

---

## 1. Executive Summary & Vision

### 1.1 Product Overview
**ShiftServe** is an on-demand, transparent event staffing and escrow settlement platform connecting verified hospitality and catering workers (servers, bartenders, kitchen stewards, ushers) with local banquet halls, caterers, and event organizers. 

The platform eliminates predatory middleman/contractor cuts, delayed paychecks, and wage disputes by pairing **pre-funded RBI-compliant escrow pools** with automated **instant UPI disbursements** and **geofenced biometric/QR attendance tracking**.

### 1.2 Core Value Propositions
* **For Workers (Hospitality Crew):**
  * **0% Commission / Deductions:** Workers keep 100% of the listed shift wage.
  * **Guaranteed Pay via Escrow:** Shift funds are pre-deposited into escrow before the shift begins.
  * **Instant UPI Settlement:** Payouts clear automatically within 15–30 minutes of shift completion.
  * **Verifiable Work History:** Digital trust score and verified shift log replacing fragmented word-of-mouth references.
* **For Companies (Organizers & Caterers):**
  * **Fast Staffing Requisition:** Publish a multi-role event with precise headcounts and wages in under 2 minutes.
  * **Vetted & Punctual Crew:** Geofenced check-in, Aadhaar/Govt photo verification, and reliability ratings.
  * **Dispute & Fraud Protection:** Quarantined escrow holds and automated telemetry logs in case of overtime or no-show disputes.
* **For Platform Administrators:**
  * **Transparent Arbitration:** Real-time access to GPS geofence timestamps, supervisor check-ins, and dual-party dispute claims.
  * **Ledger Auditing:** Real-time tracking of escrow inflows, worker UPI releases, platform fees (8%), and GST compliance.

---

## 2. User Personas & Target Roles

| Persona | Primary Needs & Frustrations | Core Platform Touchpoints |
|---|---|---|
| **Amit Sharma**<br>*(Banquet Server / Bartender)* | • Wants transparent hourly pay without agency cuts.<br>• Frustrated by 30-day payout delays and unpaid teardown overtime.<br>• Relies on mobile for immediate shift alerts and direct UPI payouts. | • Shift Discovery (`/worker/browse`)<br>• ShiftPass QR Check-in (`/worker/shifts/[id]/checkin`)<br>• Earnings & UPI Ledger (`/worker/earnings`) |
| **Rajesh Kumar**<br>*(Owner, Rajesh Catering Co.)* | • Needs 10–50 reliable staff for high-profile weddings and banquets.<br>• Frustrated by last-minute crew no-shows and cash ledger discrepancies.<br>• Demands quick event creation and live gate check-in consoles. | • Event Creator Wizard (`/company/events/new`)<br>• Escrow Funding (`/company/events/[id]/fund`)<br>• Day-of Check-in Console (`/company/events/[id]/checkin`) |
| **Pooja V.**<br>*(ShiftServe Platform Arbitrator)* | • Requires impartial, data-backed resolution for attendance & overtime conflicts.<br>• Monitors platform escrow health, KYC mismatches, and nodal bank sync. | • Admin Dashboard (`/admin/dashboard`)<br>• Disputes Queue & Arbitration (`/admin/disputes/[id]`)<br>• Audit & Transaction Ledger (`/admin/transactions`) |

---

## 3. End-to-End System Architecture & Information Architecture

The system encompasses **37 core screens** designed across both **Desktop** and **Mobile** form factors:

```
[Public & Auth Hub]
  ├── / (Landing Hub)
  ├── /signup (Phone Entry & Role Selection: Worker vs. Organizer)
  └── /signup/verify (6-Digit OTP Verification)
         │
         ├───► [Company Track]
         │      ├── /onboarding/company (Business Details & GSTIN)
         │      ├── /company/dashboard (Active Shifts, Staffing KPIs, Live Roster)
         │      ├── /company/events/new (4-Step Wizard: Details ➔ Roles ➔ Requirements ➔ Review)
         │      ├── /company/events/[id]/fund (Pre-Event Escrow Funding: UPI / NetBanking / Cards)
         │      ├── /company/events/[id] (Event Roster & Live Management)
         │      ├── /company/events/[id]/checkin (QR Scanner & Geofence Console)
         │      ├── /company/events/[id]/settle (Shift Conclusion, Overtime & Escrow Authorization)
         │      ├── /company/events/history (Past Invoices & Rehire Flow)
         │      ├── /company/settings (Company Profile & Payment Methods)
         │      └── /company/settings/ratings (Worker Reviews & Feedback)
         │
         ├───► [Worker Track]
         │      ├── /onboarding/worker/profile (Identity, Experience, Preferred Roles)
         │      ├── /onboarding/worker/payout (Direct UPI VPA Verification & KYC)
         │      ├── /worker/dashboard (Upcoming Shifts, Earnings Summary, Quick Apply)
         │      ├── /worker/browse (Shift Directory, Radius & Role Filtering)
         │      ├── /worker/shifts/[id] (Guaranteed Pay, Perks, Venue Details, Gate Instructions)
         │      ├── /worker/shifts/[id]/confirm (Commitment Bottom-Sheet & Uniform SLA)
         │      ├── /worker/bookings (Upcoming, Completed, Cancelled Shifts)
         │      ├── /worker/shifts/[id]/checkin (Live GPS Geofence, ShiftPass QR, Offline OTP)
         │      ├── /worker/earnings (UPI Transaction Ledger, ICICI UTR Records)
         │      ├── /worker/ratings (Ratings Breakdown & Organizer Feedback)
         │      └── /worker/profile (Personal Dossier, Preferences, App Settings)
         │
         ├───► [Universal Modals & Trays]
         │      ├── /reviews/new (Dual-sided Rate & Review Modal)
         │      ├── /notifications (Slide-over / Bottom Drawer Notifications)
         │      ├── /disputes/new (Raise a Dispute: Overtime, No-show, Deduction)
         │      └── /disputes/[id] (Arbitration Trajectory & Telemetry Status)
         │
         └───► [Platform Admin & Governance Console]
                ├── /admin/dashboard (GMV, Active Gigs, Quarantined Escrow, Heartbeat)
                ├── /admin/disputes (SLA Triage Queue & Overtime Claims)
                ├── /admin/disputes/[id]/arbitrate (Telemetry Evidence, Testimony, Binding Settlement)
                ├── /admin/users (Directory, Aadhaar Match Status, Dossier Inspection)
                ├── /admin/events (Live Shift Operations Matrix & Roster Radar)
                └── /admin/transactions (Settlement Ledger, Revenue, Quarantined Escrow)
```

---

## 4. Key Functional Modules & Technical Specifications

### 4.1 Escrow & Payment Engine
* **Pre-Funding Requirement:** Events cannot transition to `ACTIVE / MATCHING` until 100% of base wages + platform fee (8%) + GST (18% on platform fee) are authorized and deposited into an RBI-compliant nodal account.
* **Escrow Quarantine Protocol:** When either party files a dispute prior to payout execution, the contested funds are automatically locked into Escrow Quarantine. Neither party can withdraw until administrative adjudication or mutual consent.
* **Payout Automation:** Once a host completes shift sign-off or the 2-hour auto-release grace window expires without objection, automated webhook triggers execute direct UPI payouts via ICICI Bank / Razorpay Route with generated UTR tracking.

### 4.2 Attendance, Geofencing & Biometrics
* **Dual-Check-in Verification:**
  * **Geofence Matching:** Native GPS perimeter verification (15–50m radius around venue coordinates).
  * **Dynamic QR ShiftPass:** Time-bound, rotatable QR codes scanned via the organizer's camera console.
  * **Offline Fallback:** 4-digit supervisor-verified OTP generated on the worker's device for underground banquet halls with weak network connectivity.
* **Forensic Overtime Logging:** Worker exit telemetry records actual departure times. Overtime claims compare supervisor sign-off against GPS sensor logs.

### 4.3 Trust & Reputation System
* **Two-Way Ratings:** Organizers and workers review each other on punctuality, grooming, briefing clarity, and prompt payout.
* **Reliability Metric:** Calculated based on verified show-up rate, cancellations within 4 hours, and completed shifts.
* **Identity Verification (KYC):** Aadhaar match verification and optional PAN validation prior to unlocking payout thresholds > ₹10,000/mo.

---

## 5. Design System Standards & UI Tokens

ShiftServe utilizes the **ShiftServe Utility System** optimized for both high-density desktop management and touch-first mobile ergonomics:

* **Color Palette:**
  * **Primary Brand Accent:** `#E8871E` (Solid Marigold — energetic, distinct, high-contrast action color).
  * **Backgrounds:** `#FAFAF8` (Off-white / Warm Gray baseline canvas) and `#F8F9FF` (Light tint surface container).
  * **Status Tints:** 
    * Positive / Success: Emerald `#2E7D32` (Escrow Funded, Payout Cleared, Geofence Match).
    * Alert / Priority: Coral / Amber `#D97706` / `#C53030` (Quarantined Escrow, SLA Expiry, ID Mismatch).
  * **Borders & Dividers:** 1px hairline solid `#E2E8F0` / `#D8DADF`. No heavy drop shadows.
* **Typography:** Manrope (Clean, modern sans-serif with tabular numerics for financial transparency).
* **Border Radii:** Consistent 8px (`rounded-lg`) on buttons, cards, input fields, and status badges.
* **Touch Targets:** Minimum 44x44px touch targets on mobile viewports with fixed bottom CTA anchors.

---

## 6. Release & Verification Metrics

* **Conversion Rate:** >75% of new organizers complete first event posting within 3 minutes of signup.
* **Escrow Reliability:** 100% pre-funded escrow compliance; zero unbacked shifts.
* **Payout Latency:** 95% of verified shift payouts cleared to worker UPI within 20 minutes of sign-off.
* **Dispute SLA:** Average resolution time < 4 hours utilizing geofence telemetry evidence.
