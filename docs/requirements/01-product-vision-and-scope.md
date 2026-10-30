# 01 · Product Vision & Scope

> Cowos requirements pack · v0.1 draft · Depends on decisions in [00-start-here.md](00-start-here.md)

## 1. Vision statement

> **Every independent coworking space in India gets its own professional website and booking app in one afternoon, takes bookings and payments directly, and keeps 100% of the revenue and 100% of its customer data.**

## 2. Problem statement

Independent coworking operators (1–3 centres, 20–300 seats) run their business on WhatsApp, phone calls, Instagram DMs, and spreadsheets. Specifically:

| Problem | Effect on the operator |
|---------|------------------------|
| No website, or a stale one built by a freelancer years ago | Loses Google searches ("coworking near Hitech City") to aggregators and chains |
| No online booking for meeting rooms or day passes | Front desk spends hours on WhatsApp back-and-forth; walk-ins find the room already taken |
| Leads come through aggregators and listing sites | Pays commission or listing fees per lead or booking; the aggregator owns the customer relationship and can redirect them to a competitor |
| Manual invoicing in Excel or Tally | GST mistakes, missing customer GSTINs, slow month-end collections |
| Global coworking software (Nexudus, OfficeRnD, Cobot, Optix) | Priced in USD per member or per location, built for Western workflows (no UPI, GST, DLT SMS, WhatsApp), heavy to set up |
| Freelance website + separate booking tool | Two vendors, two logins, broken branding, no single customer record |

## 3. Goals and success measures

| Goal | Measure | Target (12 months after MVP launch) |
|------|---------|-------------------------------------|
| G1. Fast time to value | Median time from sign-up to published site | Under 2 hours of tenant effort; under 1 day elapsed (including first-publish approval) |
| G2. Tenants get real direct bookings | % of active tenants with ≥10 paid online bookings/month | ≥60% |
| G3. Sustainable business for SSPL | Paying tenants; gross margin | 150 paying tenants; gross margin ≥70% after AWS and messaging |
| G4. Retention | Monthly logo churn | ≤3% |
| G5. Badge-driven growth | % of new sign-ups attributed to "Powered by Cowos" referrals | ≥15% |
| G6. Trust | % of tenants who can correctly describe what they pay for (survey) | ≥90% |

## 4. Target users

| Segment | Description | Role in Cowos |
|---------|-------------|---------------|
| Tenant: independent operator | Owner-run space, 1 location, 20–100 seats, no website or a basic one | Primary buyer, Starter/Growth plan |
| Tenant: growing operator | 2–5 locations, 100–500 seats, small team, some existing tools | Growth/Pro plan |
| Tenant: hybrid business | Business centre, serviced office, cafe with a meeting room, incubator | Starter plan; uses meeting room + day pass products |
| Tenant staff | Community manager, front-desk executive | Daily user of Tenant Admin |
| Customer: freelancer / remote worker | Buys day passes, hot desk monthly | Booking app user |
| Customer: startup / SME | Team of 2–20, books cabins and meeting rooms, needs GST invoices | Booking app user with company account |
| Customer: corporate / enterprise team | Needs flexible seats, consolidated invoices, TDS deductions | Booking app user, invoiced monthly |
| Customer: visitor / one-time meeting | Books a meeting room for 2 hours | Booking app user (light account) |
| SSPL staff | Super Admin, Support, Finance | Super Admin Portal |

## 5. Value proposition

### 5.1 For tenants vs. aggregators and marketplaces

| | Aggregator / marketplace | Cowos |
|---|---|---|
| Who owns the customer | Aggregator | Tenant |
| Fee | Commission or listing fee per lead/booking | Flat monthly fee, **0% commission** |
| Branding | Aggregator's brand; your space is one of many listings | Your logo, colours and domain |
| Competitors shown next to you | Yes | Never |
| Payment settlement | Aggregator collects and pays out on its schedule | Straight to your bank via your own Razorpay account |
| Customer data export | Limited or none | Full export, any time |
| Discovery (new customers) | Strong (their traffic) | Weaker at first (your SEO + Google Business Profile). Cowos does not replace aggregators for discovery; it lets you convert and retain directly |

Positioning point: Cowos is **complementary** to aggregators in year 1. Tenants keep listings for discovery and move repeat customers and members to their own channel, where they pay no commission.

### 5.2 For tenants vs. global coworking software

| | Global coworking software | Cowos |
|---|---|---|
| Price | USD pricing per location/member | INR flat plans, from ₹999/month |
| India fit | Card-first, no GST/DLT/WhatsApp defaults | UPI, GST invoices with customer GSTIN, DLT SMS, WhatsApp |
| Website | Usually "bring your own" or a basic portal | Full SEO-ready website + booking app included |
| Setup | Days to weeks, consultant-led | Guided wizard, under 2 hours |
| Depth | Deep (access control, accounting integrations, community features) | Focused on website + bookings + payments + invoices first |

### 5.3 For end customers

- Book a room or day pass in under a minute from a phone, pay with UPI.
- Instant confirmation on WhatsApp/SMS/email, GST invoice with their company GSTIN.
- See, cancel, or reschedule bookings without calling.

## 6. Scope

### 6.1 In scope (across all phases)

| Application | Summary |
|-------------|---------|
| Cowos Marketing Web App | Product site, pricing with "0% commission" message and fee allocation, sign-up, demo tenant sites, blog/SEO pages, referral attribution |
| Tenant Admin Portal | Onboarding wizard, site builder (themes + sections), inventory and pricing, bookings calendar, leads, customers and companies, memberships, payments and invoices, domain and publishing, billing with Cowos, staff and roles, reports |
| Tenant Website | Pre-rendered, SEO-ready, themed website per tenant |
| Tenant Booking App | Responsive (later PWA) booking experience per tenant: browse, check availability, book, pay, manage bookings, invoices, memberships |
| Embeddable integration | Subdomain link (MVP), script widget (Phase 2), WordPress plugin (Phase 3) |
| Cowos Super Admin Portal | Tenant management, KYC and first-publish approval, plans and platform billing, support tools (audited support access), theme catalog, platform settings, monitoring dashboards |
| Platform services | Notifications (email, SMS, WhatsApp), payment gateway adapters, invoicing engine, background jobs, domain/SSL automation, media pipeline, audit logging |

### 6.2 Out of scope (v1 and the foreseeable roadmap unless noted)

| Item | Reason / alternative |
|------|----------------------|
| Free-form drag-and-drop website builder | Section-based themes cover the need; see BRD §6 |
| Per-tenant native iOS/Android apps | PWA instead (C9) |
| SSPL collecting or splitting customer payments | Direct-to-tenant settlement (C4) |
| Full accounting (ledger, P&L, payroll) | Export to Tally/Zoho Books (Phase 3 integrations) |
| Physical access control hardware | Phase 3 integration with existing vendors only (no hardware) |
| Marketplace / discovery listing of all Cowos tenants | Possible Phase 3 opt-in "Cowos Network"; would need a separate pricing decision, not commission |
| Lease management for the operator's own landlord | Not our customer's core problem for this product |
| Multi-currency, non-Indian tax regimes | Data model ready; not built until a non-India market is chosen |
| White-labelled Tenant Admin portal on the tenant's domain | Phase 3 at the earliest |

## 7. Assumptions

| # | Assumption | If wrong |
|---|-----------|----------|
| A1 | Tenants can obtain or already have a Razorpay account (or will create one during onboarding; Razorpay onboarding takes 1–5 days) | Offer offline-payment-only mode so tenants can publish while KYC is pending |
| A2 | Tenants accept a mandatory "Powered by Cowos" badge | Offer badge removal as a paid add-on on Pro (Phase 2) |
| A3 | Most tenants will use a platform subdomain first and add a custom domain later | Custom domain flow must still be in MVP |
| A4 | Tenants will provide their own photos; many photos are taken on phones | Media pipeline must compress and resize; themes must look good with imperfect photos |
| A5 | End customers mostly pay by UPI | Razorpay Checkout covers UPI, cards, netbanking, wallets |
| A6 | SSPL can obtain TRAI DLT registration and register SMS templates under its entity | Use a CPaaS provider's DLT-registered templates; MVP may launch with email + WhatsApp OTP only |
| A7 | Tenants' GSTIN is optional (small spaces below the GST threshold exist) | Invoices without GSTIN render as "Bill of Supply" / receipt, no tax lines |
| A8 | All data stays in AWS India regions (Mumbai primary, Hyderabad DR) | Matches DPDP expectations and tenant comfort |
| A9 | .NET 10 (LTS, released Nov 2025) is the "latest LTS" at build time | Update to the next LTS when available; no design change |

## 8. Product principles

1. **Tenant first, customer second, Cowos third.** The badge is the only Cowos brand element customers see.
2. **Honest money.** No hidden fees. Every rupee a tenant pays SSPL is explained on one screen.
3. **Opinionated over flexible.** Fewer, better themes and defaults that match how coworking sells.
4. **Works on a ₹12,000 Android phone on 4G.** Performance budgets are requirements, not nice-to-haves.
5. **The tenant can leave.** Full data export, domain stays theirs, no lock-in tricks. This builds trust and lowers sales friction.
