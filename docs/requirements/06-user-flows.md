# 06 · Key User Flows

> Cowos requirements pack · v0.2 draft · Diagrams are Mermaid (render on GitHub and in most Markdown viewers).

## Flow 1 · Owner onboarding → build → preview → publish

There are two paths. At launch (P1), every tenant is a pilot created by SSPL in Super Admin (Flow 1a). Public self-serve sign-up and the onboarding wizard arrive with the Billing release (P1-B, Flow 1b) (CEO-S1, CEO-S2). Both paths end in the same pre-publish checks, review and publish steps (Flow 1c). No path has a plan picker or a subscription checkout (D1, 28A).

### Flow 1a · Pilot path (P1)

```mermaid
flowchart TD
    A[Owner submits Book a demo /<br/>pilot request on cowos.in] --> B[Saved to Super Admin<br/>Pilots → Pilot requests<br/>sales notified via email outbox]
    B --> C[SSPL triages request]
    C --> D[Pilots → Create pilot:<br/>business name, city, slug, pilot flag]
    D --> E[Tenant created: state = Active, is_pilot<br/>pre-handover setup grant created<br/>edge_sync row added<br/>action audited]
    E --> F[SSPL sets up under the setup grant:<br/>inventory, theme, content<br/>every write audited]
    E --> G[Owner invite sent<br/>single use, 72h]
    G --> H{Owner opens link<br/>within 72h?}
    H -- No / resent --> I[This invite has expired.<br/>Ask SSPL to resend it.] --> G
    H -- Yes --> J[Email verified by link<br/>phone OTP, password, MFA<br/>owner accepts ToS + DPA]
    J --> K[Setup grant ends automatically]
    K --> L[First-login handoff:<br/>Review what we set up<br/>list of SSPL's audited changes]
    L --> M[Owner checklist on home:<br/>Razorpay KYC + OAuth, GSTIN/PAN,<br/>DNS, ToS/DPA]
    M --> N[Owner finishes settings screens<br/>SSPL helps only through<br/>Owner-approved support grants]
    N --> P[To Flow 1c: pre-publish checks]
```

Key rules (pilot path):
- Pilots are created directly in Active with a pilot flag. Trial expiry, trial publish rules and dunning never apply. The free year starts at the first live publish on production and ends 12 months later (Pilot Rules #3, #4).
- The pre-handover setup grant gives SSPL write access only while the tenant has no Owner, is unpublished and has no live payments. It ends when the Owner accepts the invite. After handover SSPL works only through tenant-approved grants (FR-SA-030) (R19, Pilot Rules #9).
- The owner does Razorpay KYC and OAuth, GSTIN/PAN, DNS changes, and accepts the ToS and DPA themselves (Pilot Rules #9).
- Until the production entry gate passes, the publish screen shows "Your site is ready. Publishing opens when Cowos launches". The gate blocks only publishing and live payments (Pilot Rules #1, 11A).
- Double submit of Create pilot creates one tenant (idempotency key); a taken slug returns 409. A resend revokes the old invite link (CEO A2).

### Flow 1b · Self-serve path (P1-B)

```mermaid
flowchart TD
    A[Visitor on cowos.in<br/>ref code captured] --> B[Sign up: name, email, mobile,<br/>business, city, slug]
    B --> C{Email + mobile<br/>OTP verified?}
    C -- No --> B
    C -- Yes --> D[Tenant created: state = Trial<br/>14 days]
    D --> E{Goal?}
    E -- Full website + booking --> F[Business details + GSTIN]
    E -- Booking app only --> F2[Business details + GSTIN]
    F --> G[Location and hours]
    G --> I[Choose theme: previewed with<br/>own logo, photos, Sample spaces]
    I --> H[Branding: logo, colours,<br/>font pair, button shape, mode]
    H --> K
    F2 --> K[Inventory: products, units,<br/>prices, availability, add-ons<br/>Sample items removable in one click]
    K --> L[Policies: cancellation, refund,<br/>privacy, terms]
    L --> N[Payments: connect Razorpay<br/>MFA set up inline]
    N --> P[To Flow 1c: pre-publish checks]
```

Key rules (self-serve path):
- The wizard order follows 4A. Sample inventory is labelled "Sample"; pre-publish checks block publishing while any sample item remains (FR-TA-008).
- Trial tenants can publish to the platform subdomain, marked `noindex`. A tenant becomes Active when it adds a payment method (Razorpay payment link details, or UPI Autopay / e-mandate up to ₹5,000) at the end of the trial; custom domains and indexing then unlock without waiting for the first arrears invoice (Pilot Rules #7, 28A).
- If Razorpay KYC is pending, the owner can still publish with Book a tour, Request to Book and offline payment; Instant Book products show as "Request" until Razorpay is live (33A).

### Flow 1c · Pre-publish checks → review → publish (both paths)

```mermaid
flowchart TD
    O[Live preview<br/>desktop / tablet / mobile<br/>test-mode payments] --> P[Pre-publish checks<br/>pass/fail list]
    P -- Fail --> Q[Fix links] --> O
    P -- Pass --> ST{Tenant state}
    ST -- Suspended --> SX[Your site is suspended.<br/>Contact SSPL support.]
    ST -- Pilot before production gate --> SG[Your site is ready.<br/>Publishing opens when Cowos launches]
    ST -- Trial, P1-B --> S
    ST -- Active --> R{Domain choice}
    R -- Platform subdomain --> S{First publish?}
    R -- Own domain --> V[DNS instructions per registrar<br/>TXT verify + CNAMEs<br/>MFA inline if not done]
    V --> W[DNS poll every 5 min<br/>up to 72h]
    W -- Verified --> X[Certificate issued automatically]
    X --> S
    S -- Yes --> Y[In review · typical under 4 working hours<br/>you can keep editing<br/>auto KYC checks + manual review]
    Y -- Changes requested --> Q
    Y -- Approved --> Z
    S -- No --> Z[Building… usually under 2 min<br/>render to S3, switch live pointer<br/>with state check, edge_sync outbox]
    Z -- Render failed --> ZF[Retry; previous version stays live]
    Z --> AA[Live. Notify owner by email + WhatsApp<br/>share kit: links, QR poster, GBP tips]
```

Key rules (publish):
- Domain verification runs in the background. The tenant can publish on the platform subdomain while waiting.
- The live-pointer switch re-reads tenant state in the same transaction, so a publish that finishes after a suspension never puts the normal site back (CEO-A1). The edge learns of the switch only through the edge_sync outbox (R17).
- A pilot's first live publish on production sets its free-period end date (Pilot Rules #3).

## Flow 2 · Existing-website integration (booking app only)

The "I already have a website" goal choice is a wizard step and arrives with the wizard at P1-B (FR-TA-006, CEO-S2).

```mermaid
flowchart TD
    A[Tenant chooses 'I already have a website'] --> B[Business, location, inventory,<br/>policies, payments]
    B --> C[Branding: logo + colours only<br/>to match existing site]
    C --> D[Preview booking app]
    D --> E{How to integrate?}
    E -- Subdomain link MVP --> F[Add CNAME book.theirdomain.com<br/>+ TXT verify]
    F --> G[Cert issued, book.theirdomain.com live]
    G --> H[Copy 'Book now' button HTML<br/>and per-product deep links<br/>into existing site]
    E -- Script widget Phase 2 --> I[Copy one script tag<br/>set allowed origins]
    I --> J[Floating button + inline buttons<br/>on existing site]
    J --> K[Browse in modal, login + payment<br/>open in full tab]
    E -- No domain change --> L[Use acme.cowos.site/book link]
    H --> M[Customer books on Cowos booking app<br/>'Back to website' link returns them]
    K --> M
    L --> M
```

## Flow 3 · Customer booking → payment → confirmation (Instant Book)

```mermaid
sequenceDiagram
    autonumber
    actor C as Customer
    participant BA as Booking App (SPA)
    participant API as Cowos API
    participant DB as PostgreSQL
    participant RZ as Razorpay (tenant account)
    participant W as Worker / Notifications

    C->>BA: Open book.acmehub.in, pick Boardroom
    C->>BA: Date strip: tomorrow · duration chip: 2h
    BA->>API: GET /public/availability?product&date&duration (tenant from Host)
    API->>DB: free start times (rules, buffers, holidays, existing bookings),<br/>merged across units for "any room"
    API-->>BA: start times with price and text availability
    BA->>C: Start-times list ("10:00 – 12:00 · ₹1,416")
    C->>BA: Choose 10:00 – 12:00, Continue
    BA->>C: Step 1 summary: slot, total, "Pay to (tenant legal name)",<br/>"incl. GST ₹216 ▸", one-line cancellation policy
    C->>BA: Continue
    BA->>C: Step 2 phone OTP (if not logged in)<br/>privacy notice, unticked marketing consent, bill-to-company
    C->>BA: Verify OTP
    BA->>API: POST /public/bookings/holds {unit or any, start, end, addOns, coupon}
    API->>DB: INSERT booking status=Held, expires_at=now+10m<br/>(exclusion constraint blocks overlaps)
    alt Slot taken during OTP
        DB-->>API: constraint violation
        API-->>BA: 409 + next free times
        BA->>C: Next free times inline
    end
    API->>API: Re-quote price
    API->>RZ: Create order (amount, receipt=booking id) using tenant credentials
    RZ-->>API: order_id
    API-->>BA: hold + order_id + price breakdown (+ any price difference)
    BA->>C: Step 3: 10-minute countdown, any price difference shown
    BA->>RZ: Open Checkout (UPI / card / netbanking)
    C->>RZ: Pays
    RZ-->>BA: payment_id + signature
    BA->>C: "Confirming your payment…" (poll up to 2 min)
    BA->>API: POST /public/payments/verify
    API->>API: Verify signature (HMAC)
    API->>DB: Payment=Captured, Booking=Confirmed (idempotent)
    RZ-->>API: Webhook payment.captured (backup path, idempotent)
    API->>W: BookingConfirmed event (outbox)
    W->>W: Generate invoice number + PDF
    W->>C: Email (invoice, .ics) + WhatsApp/SMS confirmation<br/>with desk pass link
    W->>W: Notify tenant staff
    API-->>BA: Confirmed
    BA->>C: Confirmation: desk pass + QR, .ics,<br/>invoice (or "Invoice being prepared")
```

The hold starts only after OTP, never before (2A). A UPI return or a reload opens `/book/checkout/{bookingRef}`, which needs the customer's session; without one the page asks for OTP and then shows the booking's state (R16).

Edge cases handled:
| Case | Behaviour |
|------|-----------|
| Customer closes the browser after paying | Webhook confirms the booking; customer gets notifications anyway |
| Webhook slower than the client | "Confirming your payment…" for up to 2 minutes, then "We've got your payment, confirmation is on its way by WhatsApp"; never an error (9A) |
| Hold expires during payment, slot still free | Late capture confirms the booking |
| Hold expired before payment, slot still free | One-tap retry with a new hold (9A) |
| Hold expires, slot taken by someone else, payment captured | Automatic full refund: "Full refund of ₹X started, 5–7 working days" plus the next free times (9A) |
| Price changed between summary and hold | Hold re-quotes the price; the difference shows before Razorpay opens (2A) |
| Payment fails | Hold released immediately; customer can retry with a new hold |
| Webhook arrives before client verify | Same idempotency key (gateway payment id); second call is a no-op |
| Gateway down | App offers Request to Book (9A) |
| Tenant in `Restricted` state (P1-B) | Hold creation refused; booking app shows "Call / WhatsApp to book" with the tenant's contacts (10A) |
| Tenant `Suspended` | Website shows the suspended page; no new holds (10A, CEO-A1) |

## Flow 4 · Cancellation and refund

```mermaid
flowchart TD
    A{Who cancels?} -- Customer --> B[Open booking, tap Cancel]
    A -- Tenant staff --> C[Open booking in admin, Cancel]
    B --> D[System computes refund from policy tier<br/>shows amount + destination]
    C --> E{Override?}
    E -- No --> D2[Policy amount<br/>tenant-initiated = 100%]
    E -- Yes, Owner/Manager --> F[Enter amount + reason<br/>audited]
    D --> G{Customer confirms?}
    G -- No --> Z[No change]
    G -- Yes --> H
    D2 --> H
    F --> H[Booking = Cancelled<br/>slot released immediately]
    H --> I{Refund amount > 0?}
    I -- No --> N
    I -- Yes --> J{Destination}
    J -- Original method --> K[Refund API on tenant Razorpay account<br/>Refund = Pending]
    J -- Account credit --> L[Credit ledger entry]
    K --> M{Webhook result}
    M -- processed --> M1[Refund = Processed]
    M -- failed --> M2[Refund = Failed<br/>alert tenant, retry action]
    L --> N
    M1 --> N[Credit note issued if invoice exists]
    N --> O[Notify customer + tenant<br/>email + WhatsApp/SMS]
```

Notes:
- Refunds come from the tenant's Razorpay balance. If the tenant's balance is insufficient, Razorpay may fail or queue the refund; the tenant gets a clear alert.
- Gateway fees are not returned by Razorpay. Whether the customer refund deducts them is a tenant policy setting and is shown in the policy text.
- Monthly bookings and memberships don't use this flow; they follow notice-period rules handled by staff (Phase 2 automates it).

## Flow 5 · Tenant states: hosting invoice lapse → restriction → suspension → deletion

```mermaid
stateDiagram-v2
    [*] --> Active: Create pilot (P1, pilot flag)
    [*] --> Trial: self-serve sign-up (P1-B)
    Trial --> Active: payment method added
    Trial --> Expired: trial ends with no payment method
    Expired --> Active: payment method added (within 30 days)
    Expired --> PendingDeletion: 30 days
    Active --> PastDue: hosting invoice unpaid on due date
    PastDue --> Active: paid
    PastDue --> Restricted: day 8 (unconverted pilot - day 14)
    Restricted --> Active: paid
    Restricted --> Suspended: day 22 (unconverted pilot - after 90 days Restricted)
    Suspended --> Active: paid
    Suspended --> PendingDeletion: 90 days suspended
    Active --> Cancelled: owner closes account
    Cancelled --> Active: owner reopens
    Cancelled --> PendingDeletion: 90 days after period end
    PendingDeletion --> Deleted: purge job (tax records archived)
    Active --> SuspendedByAdmin: FR-SA-011 suspend (reason, audited)
    SuspendedByAdmin --> Active: Super Admin reinstates
    Deleted --> [*]
```

Key rules:
- **At launch (P1)** the only transitions are Create pilot (into Active with the pilot flag), FR-SA-011 suspend and reinstate. Super Admin offers no move into Trial, PastDue or Restricted, and the API rejects one with 409 until the Billing release (P1-B) enables trials and dunning (Q1, CEO-S1).
- Pilots stay Active through the free year: no payment method, trial expiry or dunning applies. The free year runs 12 months from the first live publish on production (Pilot Rules #3, #4).
- From month 13 a former pilot pays its hosting cost share; month 13 is invoiced on the 10th of month 14. A pilot that hasn't paid that first invoice by its due date plus 14 days is unconverted: it moves to Restricted and stays there for 90 days before the normal Suspended lifecycle. A former pilot that pays and then misses a later invoice follows the standard schedule (Pilot Rules #6).
- Hosting invoices are issued on the 10th for the previous month and are due in 10 days (Pilot Rules #10, R1).
- Every state change raises one TenantStateChanged event; the website switch for Restricted and Suspended goes through the data-refresh render in FR-PL-012 (R11, CEO-A1).

What each state allows:

| State | Website | Booking app: new bookings | Booking app: existing customers | Tenant admin | Notifications |
|-------|---------|--------------------------|-------------------------------|--------------|---------------|
| Trial (P1-B) | Live on platform subdomain (`noindex`) | Yes | Yes | Full (custom domain locked) | Day 10–14 reminders |
| Active | Live | Yes | Yes | Full | Monthly hosting invoice on the 10th; pilots: free-year notices at 60, 30 and 7 days |
| PastDue (days 0–7, P1-B) | Live | Yes | Yes | Full + banner | Daily |
| Restricted (days 8–21, P1-B) | Live; every Book CTA becomes "Call / WhatsApp to book", with no billing mention (10A) | **No**: booking app home shows the tenant's call/WhatsApp options | View, cancel, invoices | Read-only + billing, exports, booking management | Every 2 days |
| Suspended (day 22+, P1-B) | Page in the tenant's theme with name, address, phone, WhatsApp, "Online booking is temporarily unavailable" and the badge (10A) | No | View bookings, download invoices | Billing, export, cancel only | Day 30, 60, 80 |
| Expired (trial ended with no payment method, P1-B) | Offline | No | View | Billing + export | Day 1, 7, 25 |
| Cancelled (P1-B) | Live until period end, then as Suspended | Until period end | Yes | Until period end, then billing/export | Export reminders |
| SuspendedByAdmin | Suspended page in the tenant's theme, as Suspended (10A, CEO-A1) | No | No | Owner can see the reason and contact SSPL; publish blocked | Immediate |
| PendingDeletion → Deleted | Offline, DNS mapping removed | No | No | No | Final notice 7 days before purge |

## Flow 6 · Request to Book (event space)

```mermaid
sequenceDiagram
    actor C as Customer
    participant API as Cowos API
    actor T as Tenant staff
    C->>API: Submit request (date, time, attendees, notes)
    API->>API: Booking = Requested, tentative hold (not blocking by default)
    API->>T: Staff notification (WhatsApp + admin)
    alt Approved within SLA
        T->>API: Approve (optional price adjustment)
        API->>API: Booking = AwaitingPayment, slot blocked for 24h
        API->>C: Payment link (24h)
        C->>API: Pays
        API->>API: Booking = Confirmed, invoice
    else Declined
        T->>API: Decline with message
        API->>C: Declined + alternatives
    else SLA expired
        API->>C: Auto-declined, tenant contact shown
    end
```

## Flow 7 · Lead → tour → membership (sales-led cabins and dedicated desks)

```mermaid
flowchart LR
    A[Website form / Enquire product /<br/>Tour booking / Walk-in / Phone] --> B[Lead: New]
    B --> C[Contacted<br/>WhatsApp/call from admin]
    C --> D[Tour scheduled<br/>reminder to lead + staff]
    D --> E[Tour done]
    E --> F[Proposal sent<br/>price, unit, start date, deposit]
    F --> G{Outcome}
    G -- Won --> H[Customer + company created<br/>monthly booking / membership<br/>deposit recorded<br/>first invoice + payment link]
    G -- Lost --> I[Lost reason captured<br/>price, location, size, competitor]
```
