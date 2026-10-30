# 06 · Key User Flows

> Cowos requirements pack · v0.1 draft · Diagrams are Mermaid (render on GitHub and in most Markdown viewers).

## Flow 1 · Tenant registration → build → preview → pay → publish

```mermaid
flowchart TD
    A[Visitor on cowos.in<br/>ref code captured] --> B[Sign up: name, email, mobile,<br/>business, city, slug]
    B --> C{Email + mobile<br/>OTP verified?}
    C -- No --> B
    C -- Yes --> D[Tenant created: state = Trial<br/>14 days]
    D --> E{Goal?}
    E -- Full website + booking --> F[Business details + GSTIN]
    E -- Booking app only --> F2[Business details + GSTIN]
    F --> G[Location, hours, amenities]
    G --> H[Branding: logo, colours,<br/>fonts, button shape, mode]
    H --> I[Choose theme]
    I --> J[Edit sections<br/>hero, about, gallery, FAQ...]
    F2 --> K
    J --> K[Inventory: products, units,<br/>prices, availability, add-ons]
    K --> L[Policies: cancellation, refund,<br/>privacy, terms]
    L --> M{Any Instant Book<br/>products?}
    M -- Yes --> N[Connect Razorpay<br/>MFA required]
    M -- No --> O
    N --> O[Live preview<br/>desktop / tablet / mobile<br/>test-mode payments]
    O --> P[Pre-publish checks]
    P -- Fail --> Q[Fix list with links] --> O
    P -- Pass --> R{Domain choice}
    R -- Platform subdomain --> S{First publish?}
    R -- Own domain --> T{Plan paid?}
    T -- No --> U[Choose plan, pay via Razorpay<br/>SSPL invoice issued<br/>state = Active]
    T -- Yes --> V
    U --> V[DNS instructions per registrar<br/>TXT verify + CNAMEs]
    V --> W[DNS poll every 5 min<br/>up to 72h]
    W -- Verified --> X[Certificate issued automatically]
    X --> S
    S -- Yes --> Y[SSPL review queue<br/>auto KYC checks + manual review<br/>target under 4 working hours]
    Y -- Changes requested --> Q
    Y -- Approved --> Z
    S -- No --> Z[Render static site to S3<br/>switch live pointer<br/>CDN invalidation]
    Z --> AA[Live. Notify owner by email + WhatsApp<br/>share kit: links, QR poster, GBP tips]
```

Key rules:
- Trial tenants can publish to the platform subdomain (marked `noindex` until paid). Payment is required before a custom domain goes live, and before the trial ends.
- If you want strict "pay before any publish" (as in the original brief), set the platform flag `require_payment_before_first_publish = true`. Recommendation: keep it off, because a live site during the trial is the strongest conversion driver.
- Domain verification runs in the background. The tenant can publish on the platform subdomain while waiting.

## Flow 2 · Existing-website integration (booking app only)

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

    C->>BA: Open book.acmehub.in, pick Boardroom, tomorrow 10-12
    BA->>API: GET /public/availability?product&date (tenant from Host)
    API->>DB: free slots (rules, buffers, holidays, existing bookings)
    API-->>BA: slots
    C->>BA: Continue to pay
    BA->>C: Phone OTP login (if not logged in)
    BA->>API: POST /public/bookings/holds {unit, start, end, addOns, coupon}
    API->>DB: INSERT booking status=Held, expires_at=now+10m<br/>(exclusion constraint blocks overlaps)
    alt Slot taken
        DB-->>API: constraint violation
        API-->>BA: 409 + next available slots
    end
    API->>RZ: Create order (amount, receipt=booking id) using tenant credentials
    RZ-->>API: order_id
    API-->>BA: hold + order_id + price breakdown
    BA->>RZ: Open Checkout (UPI / card / netbanking)
    C->>RZ: Pays
    RZ-->>BA: payment_id + signature
    BA->>API: POST /public/payments/verify
    API->>API: Verify signature (HMAC)
    API->>DB: Payment=Captured, Booking=Confirmed (idempotent)
    RZ-->>API: Webhook payment.captured (backup path, idempotent)
    API->>W: BookingConfirmed event (outbox)
    W->>W: Generate invoice number + PDF
    W->>C: Email (invoice, .ics) + WhatsApp/SMS confirmation
    W->>W: Notify tenant staff
    API-->>BA: Confirmed + QR
```

Edge cases handled:
| Case | Behaviour |
|------|-----------|
| Customer closes the browser after paying | Webhook confirms the booking; customer gets notifications anyway |
| Hold expires during payment, slot still free | Late capture confirms the booking |
| Hold expires, slot taken by someone else, payment captured | Automatic full refund + apology notification with alternatives |
| Payment fails | Hold released immediately; customer can retry with a new hold |
| Webhook arrives before client verify | Same idempotency key (gateway payment id); second call is a no-op |
| Tenant in `Restricted` state | Hold creation refused with "Online booking paused, call/WhatsApp <tenant>" |

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

## Flow 5 · Tenant payment lapse → restriction → suspension → deletion

```mermaid
stateDiagram-v2
    [*] --> Trial
    Trial --> Active: first payment
    Trial --> Expired: trial ends unpaid
    Expired --> Active: payment (within 30 days)
    Expired --> PendingDeletion: 30 days
    Active --> PastDue: renewal unpaid on due date
    PastDue --> Active: paid
    PastDue --> Restricted: day 8
    Restricted --> Active: paid
    Restricted --> Suspended: day 22
    Suspended --> Active: paid
    Suspended --> PendingDeletion: 90 days suspended
    Active --> Cancelled: owner cancels (end of paid period)
    Cancelled --> Active: resubscribe
    Cancelled --> PendingDeletion: 90 days after period end
    PendingDeletion --> Deleted: purge job (tax records archived)
    Active --> SuspendedByAdmin: policy/fraud action
    SuspendedByAdmin --> Active: Super Admin reinstates
    Deleted --> [*]
```

What each state allows:

| State | Website | Booking app: new bookings | Booking app: existing customers | Tenant admin | Notifications |
|-------|---------|--------------------------|-------------------------------|--------------|---------------|
| Trial | Live on platform subdomain (`noindex`) | Yes | Yes | Full (custom domain locked) | Day 10–14 reminders |
| Active | Live | Yes | Yes | Full | Renewal reminders −7d, −1d |
| PastDue (days 0–7) | Live | Yes | Yes | Full + banner | Daily |
| Restricted (days 8–21) | Live | **No**: "Online booking paused, contact <tenant>" | View, cancel, invoices | Read-only + billing, exports, booking management | Every 2 days |
| Suspended (day 22+) | Neutral "temporarily unavailable" page with tenant contacts | No | View bookings, download invoices | Billing, export, cancel only | Day 30, 60, 80 |
| Expired (trial unpaid) | Offline | No | View | Billing + export | Day 1, 7, 25 |
| Cancelled | Live until period end, then as Suspended | Until period end | Yes | Until period end, then billing/export | Export reminders |
| SuspendedByAdmin | Offline (generic page) | No | No | Owner can see the reason and contact SSPL | Immediate |
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
