# 03 · Business Requirements Document (BRD)

> Cowos requirements pack · v0.1 draft

## 1. Purpose

This BRD states what the business needs from Cowos, the rules the product must enforce, and how SSPL makes money. The SRS ([04](04-software-requirements-specification.md)) turns these into testable system requirements. Business requirements are numbered `BR-xx`.

## 2. Business objectives

| ID | Objective | Linked goal |
|----|-----------|-------------|
| BO-1 | Launch a self-serve product that lets a coworking space publish a branded website and booking app without SSPL staff doing the setup | G1 |
| BO-2 | Earn recurring revenue from flat subscriptions, with 0% commission on bookings | G3 |
| BO-3 | Keep the cost to serve one small tenant under ₹400/month (AWS + messaging overhead excluding pass-through usage) | G3 |
| BO-4 | Make the "Powered by Cowos" badge a measurable acquisition channel | G5 |
| BO-5 | Be compliant from day 1 with GST invoicing rules and the DPDP Act 2023 / DPDP Rules 2025 | G6 |

## 3. Stakeholders

| Stakeholder | Interest |
|-------------|----------|
| SSPL leadership | Revenue, margin, brand |
| SSPL product & engineering | Buildable scope, maintainable platform |
| SSPL support & operations | Low ticket volume, clear tools |
| SSPL finance | Clean platform invoices, GST returns, collections |
| Tenants (owners, staff) | Bookings, revenue, simplicity, trust |
| End customers | Fast booking, clear pricing, invoices |
| Partners (Razorpay, CPaaS providers, AWS) | Compliant usage, partner program terms |
| Legal / tax advisors | GST, DPDP, consumer protection, terms of service |

## 4. Business requirements

### 4.1 Tenant acquisition and onboarding

| ID | Requirement |
|----|-------------|
| BR-01 | A coworking operator can sign up, verify email and phone, and start a 14-day free trial without talking to SSPL. |
| BR-02 | The onboarding wizard collects everything needed to publish a site in one sitting, and saves progress at every step so the tenant can resume. |
| BR-03 | Onboarding offers a choice: "Build my website and booking app" or "I already have a website, I only need the booking app". |
| BR-04 | Sign-ups referred by a tenant's badge or a partner are attributed (referral code, UTM) and visible to SSPL. |
| BR-05 | SSPL can offer assisted onboarding ("we set it up for you") as a paid one-time service, using the same tools with a support grant. |

### 4.2 Information captured during onboarding

| Group | Fields | Required to publish? |
|-------|--------|---------------------|
| Business details | Brand name, legal entity name, entity type, GSTIN (optional, validated format + checksum, verified against the GST public search API where available), PAN (if no GSTIN), primary contact name, phone, email, support phone/WhatsApp | Brand name, legal name, contact, phone, email: yes. GSTIN: no (required to issue tax invoices) |
| Location(s) | Address lines, locality, city, state (drives GST place of supply), PIN code, map pin (lat/long via Google Places autocomplete), directions text, landmarks, parking, nearest metro | Yes (one location) |
| Operating hours | Per day of week, open/close, 24×7 flag, holiday calendar, member-only extended hours | Yes |
| Amenities | Pick list (Wi-Fi speed, power backup, AC, parking, cafeteria, printing, lockers, phone booths, mail handling, 24×7 access, CCTV, pet friendly, wheelchair access) + custom amenities with icon | No (recommended) |
| Policies | House rules, cancellation policy (template-based), refund policy, privacy policy (generated from template, tenant as data fiduciary), terms of booking | Cancellation, refund, privacy and terms: yes (defaults offered) |
| Branding | Logo (SVG/PNG, light + dark variant optional), favicon (auto-generated from logo if not supplied), primary / secondary / accent colours (with automatic contrast check), font pair from a curated list, button shape (rounded / pill / square), theme mode (light / dark / auto) | Logo and primary colour: yes |
| Website content | Header (nav items auto-built from enabled sections, CTA button text + target), hero (headline, sub-headline, image or video, CTA), about, amenities, spaces/plans (auto-built from inventory), gallery, testimonials, FAQ, contact form, location block, footer (text, links, social links: Instagram, LinkedIn, Facebook, X, YouTube, Google Business Profile), SEO title/description per page | Hero headline + 1 image: yes. Others optional with sensible defaults |
| Inventory | Workspace types (hot desk, dedicated desk, private cabin, meeting room, event space, virtual office, plus "custom"), name, description, capacity, count of units, seat/room labels, photos, amenities, floor/zone, booking mode (Instant / Request / Enquire), min and max duration, pricing per hour / half-day / day / week / month, member pricing, add-ons (projector, catering, extra chair, locker, parking), buffer before/after, lead time, advance booking window | At least one product with booking mode set: yes |
| Payments | Gateway connection (Razorpay OAuth or keys), accepted methods, tax settings (prices inclusive/exclusive of GST, SAC code default, rate), invoice prefix and numbering, invoice footer (bank details, signature image), security deposit rules | Gateway: required for Instant Book products. Tenants without a gateway can publish with Enquire/Request modes and offline payments only |

### 4.3 Website and booking app

| ID | Requirement |
|----|-------------|
| BR-06 | Tenants build their site by choosing one of several prebuilt themes and customising branding and content. There is no free-form drag-and-drop editor in v1 (justification in §6). |
| BR-07 | Every tenant website and booking app shows a "Powered by Cowos" badge in the footer, linked to the Cowos marketing site with the tenant's referral code. Tenants cannot remove or hide it on any v1 plan. |
| BR-08 | Tenants can preview the draft website and booking app on desktop, tablet and mobile sizes before publishing, including real inventory and prices. |
| BR-09 | Every publish creates an immutable version. Tenants can view the version history and roll back to any of the last 20 versions in one click. |
| BR-10 | Tenants can publish on a free platform subdomain or on their own domain. Custom domains get SSL automatically. |
| BR-11 | Tenants who already have a website can use only the booking app, integrated by subdomain link (MVP) or embeddable widget (Phase 2). |
| BR-12 | Tenant websites must be indexable and fast enough to rank in local search (see NFR SEO and performance budgets). |
| BR-13 | The booking app supports the booking modes in §5.1 and sells day passes, hourly meeting rooms, monthly desks and cabins, event spaces and virtual offices. |

### 4.4 Money

| ID | Requirement |
|----|-------------|
| BR-14 | SSPL takes **0% commission** on tenant bookings: Cowos never takes a percentage or per-booking fee from the tenant's booking revenue. This statement appears on the marketing site, in onboarding, on the billing page, and in the terms. |
| BR-25 | Customer-facing pricing is the **tenant's decision**. The tenant sets prices, member rates, promo codes, discounts, optional convenience fees, deposits, and whether gateway fees are passed on. Cowos provides the tools but never sets, funds, or adds anything to the customer's bill. |
| BR-26 | Tenants can create and manage their own promo codes in the Tenant Admin on every plan, without SSPL involvement. |
| BR-15 | Customer payments settle directly to the tenant's own payment gateway account. SSPL never holds or routes tenant funds. |
| BR-16 | Tenants pay SSPL a flat subscription (monthly or annual), plus pass-through messaging usage from a prepaid wallet. The billing page shows what is included and what each usage line cost. |
| BR-17 | All SSPL invoices to tenants are GST tax invoices from SSPL's GSTIN; tenants with a GSTIN get it printed for input tax credit. |
| BR-18 | All invoices from tenants to customers are generated by Cowos on the tenant's behalf, with the tenant's GSTIN, customer GSTIN (if provided), correct SAC and tax split, sequential numbering per financial year. |
| BR-19 | Refunds after an invoice is issued produce a credit note; issued invoices are never edited or deleted. |

### 4.5 Operations and trust

| ID | Requirement |
|----|-------------|
| BR-20 | SSPL reviews and approves a tenant's first publish after automated KYC checks (target: decision within 4 working hours). |
| BR-21 | SSPL support can only access a tenant's customer data through a tenant-approved, time-limited, audited support grant. |
| BR-22 | Tenants can export all their data (customers, bookings, invoices, payments, content, media) at any time, in CSV/JSON and a zip of media. |
| BR-23 | A tenant's domain remains the tenant's. On cancellation, Cowos provides DNS instructions to point it elsewhere and does not hold it hostage. |
| BR-24 | The platform keeps tenants' data in AWS India regions. |

## 5. Business rules

### 5.1 Booking modes

| Mode | Customer sees | What happens | Payment |
|------|---------------|--------------|---------|
| Instant Book | "Book now" with price | Slot held for 10 minutes during checkout → confirmed on successful payment | Online at booking (or member credits) |
| Request to Book | "Request booking" with indicative price | Request sent to tenant → tenant approves within SLA (default 24h) → customer gets payment link valid 24h → confirmed on payment. Auto-declines when SLA expires | Online after approval, or offline |
| Enquire / Book a Tour | "Schedule a visit" / "Get a quote" | Lead created with preferred dates; optional tour slot picked from tenant's tour calendar | None at this stage; tenant converts lead to a membership/booking later |

### 5.2 Pricing rules
- Prices are set per product per unit: hour, half-day (4h), day, week, month. A product may offer any subset.
- When a booking spans multiple units, the system offers the cheapest valid combination (e.g. 5 hours priced as half-day + 1 hour when cheaper) only if the tenant enables "smart pricing". Default: the customer picks the unit.
- Member pricing overrides public pricing when the customer has an active membership that grants it.
- Prices are entered as GST-exclusive or GST-inclusive per tenant setting; the invoice always shows taxable value + tax.
- Promo codes (coupons) are created by the tenant (Owner, or Manager for their locations) in the Tenant Admin: percentage or flat, min order value, max discount, validity dates, usage limits (total, per customer), applicable products and locations, first-booking-only flag, public (shown on the site) or private (shared by the tenant directly). The discount is the tenant's own; Cowos does not fund or approve it. One code per booking in v1.
- Optional convenience / booking fee: the tenant may add a flat or percentage fee per product, shown to the customer as a separate line before payment and taxed per the tenant's settings. Off by default. This fee goes to the tenant, not Cowos.
- Add-ons are priced per booking, per hour, or per person.

### 5.3 Availability rules
- A resource is bookable when: inside location operating hours (or 24×7), not on a holiday/blackout, not within buffer time of another booking, not under maintenance, within the advance window (default 90 days) and after lead time (default 30 minutes).
- Hot desk products are capacity pools (e.g. 40 seats, no fixed seat). Day bookings decrement capacity per day. Optional seat selection when the tenant labels seats.
- Dedicated desks and cabins are specific resources and, once on a monthly booking/membership, are unavailable for the period.
- Double booking of a specific resource for overlapping time is impossible (enforced by the database, not only by application code).

### 5.4 Cancellation and refund rules (defaults, tenant configurable)

| Product | Free cancellation until | Partial refund window | No refund |
|---------|------------------------|-----------------------|-----------|
| Meeting room | 24h before start: 100% | 24h–2h before: 50% | <2h or no-show |
| Day pass | Until 9 PM the day before: 100% | Morning of: 50% credit | After check-in |
| Event space | 7 days before: 100% (minus gateway fees, if tenant chooses) | 7–2 days: 50% | <48h |
| Monthly desk / cabin | Governed by agreement; default 30 days' notice, deposit refunded after deductions | n/a | n/a |

- Refund destination options per tenant: original payment method (default) or account credit (customer can choose credit if the tenant allows and offers a bonus).
- Gateway fees are not refundable by Razorpay; the tenant chooses whether to absorb them or deduct them from refunds. The policy text shown to customers must state this.
- Tenant-initiated cancellations always refund 100%.
- Reschedule: allowed when the new slot is available and the request is before the free-cancellation cutoff; price difference charged or credited.

### 5.5 Membership rules (Phase 2 for full feature; data model in MVP)
- A membership plan has: price per month, billing cycle, included products (e.g. 1 dedicated desk), included credits (meeting-room hours per month), member discounts, access hours, security deposit amount, notice period.
- Credits reset each billing cycle; unused credits do not roll over unless the tenant enables rollover (with a cap).
- Memberships auto-renew with an invoice generated N days before the cycle (default 5). Online collection by payment link or UPI Autopay (Phase 3); otherwise marked for offline collection.

## 6. Builder approach: theme-based with sections, not drag-and-drop

**Decision:** v1 ships 4–6 prebuilt themes. Each theme is a fixed set of section types (Hero, About, Spaces, Amenities, Gallery, Pricing, Testimonials, FAQ, Location, Contact, CTA band, Footer). Tenants switch sections on/off, reorder them within allowed slots, edit content through forms, and set branding tokens (colours, fonts, button shape, light/dark).

**Why:**

| Factor | Theme + sections | Drag-and-drop |
|--------|-----------------|---------------|
| Time to a good result for a non-designer | ~1 hour; looks professional by default | Hours, and often looks worse than a template |
| Mobile quality | Guaranteed by the theme | Tenants break mobile layouts without knowing |
| Performance and SEO | Controlled HTML, image sizes, semantic headings | Arbitrary nesting, oversized images, missing headings |
| Accessibility (WCAG 2.1 AA) | Built into themes; colour contrast checked on input | Tenants can create inaccessible combinations |
| Build cost | ~2–3 months for builder + 4 themes | 6–12 months for a competent editor |
| Support cost | Low; "how do I change X" has one answer | High; every site is unique |
| Badge integrity | Badge is part of the footer section | Tenants can cover or hide it |
| Upgrades | Theme improvements roll out to every tenant | Each custom site must be checked |

**Escape hatches** (so tenants don't feel boxed in):
- Custom pages from a simple rich-text + image page template (e.g. "Events", "Community").
- "Custom code" slot for analytics scripts only (Google Analytics 4, Meta Pixel, Google Tag Manager IDs as fields, not free HTML) in v1. Free HTML embed blocks are a Phase 2 Pro feature with a content security policy review.
- AI-assisted copy (Phase 2): generate hero text, about section and FAQ from business details.

## 7. Platform pricing and billing lifecycle

### 7.1 Plans (indicative; see 00 §C1)
- Starter, Growth, Pro; monthly or annual (annual = ~2 months free).
- Plan limits enforced: locations, resources, staff seats, custom domains, features.
- Messaging wallet: prepaid, minimum top-up ₹500, per-message prices displayed (SMS, WhatsApp utility/authentication/marketing templates priced at Meta/CPaaS cost + 0% markup in v1, or a small disclosed handling fee: **decision D7**).
- Prices shown excluding GST, with GST shown at checkout.

### 7.2 Transparency requirements
- Billing page shows: current plan and what it includes; next invoice date and amount; messaging usage this cycle by channel and template category; a "Where your fee goes" allocation chart (hosting and CDN, database and backups, security and SSL, monitoring, support, product development), updated quarterly from actual platform costs and labelled as an allocation.
- Marketing pricing page shows the same allocation plus a calculator ("You do 300 bookings a month at ₹500. An aggregator charging 15% would take ₹22,500. Cowos Starter costs ₹999.") with the commission rate input editable by the visitor.

### 7.3 Billing lifecycle

| Stage | Trigger | System behaviour |
|-------|---------|------------------|
| Trial | Sign-up | 14 days. Publish allowed to platform subdomain only. Booking app works fully. Daily reminder from day 10 |
| Activation | Tenant pays first invoice | Plan active; custom domain enabled; first-publish approval queued if not done |
| Renewal | 7 days and 1 day before renewal | Reminder email + WhatsApp to Owner. Auto-charge if mandate exists |
| Payment failed / not paid on due date | Due date passes | Day 0: status **Past due**. Retries day 1, 3, 5 (for mandates). Banner in admin. Everything keeps working |
| Grace period | Days 0–7 past due | Full service. Daily reminders |
| Restricted | Day 8–21 past due | Website stays live. Booking app stops accepting **new** online bookings (shows "Contact us" with tenant phone/WhatsApp). Existing bookings, customer logins, cancellations and refunds keep working. Admin is read-only except billing, exports and booking management |
| Suspended | Day 22+ past due | Website replaced by a neutral "temporarily unavailable" page with the tenant's contact details (no SSPL branding beyond the badge). Customer portal read-only (view bookings, download invoices). Owner can log in to pay, export data, or cancel |
| Data retention | Suspended for 90 days, or tenant cancels | Data kept, read-only. Reminder emails at day 30, 60, 80 with export link |
| Deletion | Day 91 after suspension/cancellation | Personal data and content deleted or anonymised; invoices and tax records kept for the statutory period (8 years from end of the relevant financial year under GST law; confirm with advisor) in a restricted archive; audit log retained. Domain mapping removed, SSL certificates released |
| Reactivation | Payment at any stage before deletion | Immediate restore, no data loss |

Rationale for not taking the site down at day 1: customers of the tenant are not party to the dispute; breaking their paid bookings hurts the tenant's reputation and creates refund disputes. Restricting new bookings creates enough pressure.

### 7.4 Platform invoices
- Issued by SSPL with SSPL's GSTIN; tenant GSTIN included when provided; IGST for inter-state tenants, CGST+SGST for tenants in SSPL's state.
- Invoice numbering: unique, sequential per financial year, max 16 characters (e.g. `CW/26-27/000123`).
- Proration on mid-cycle upgrades; downgrades take effect at next renewal.
- Refund policy for platform fees: annual plans refundable pro-rata within 30 days of purchase; monthly non-refundable (decision D8).

## 8. Compliance requirements (business view)

| Area | Requirement |
|------|-------------|
| GST | Correct tax invoices, credit notes, HSN/SAC, place of supply, invoice series per tenant per FY, exports for GSTR-1 filing (CSV in GSTR-1 format, Phase 2) |
| DPDP Act 2023 + DPDP Rules 2025 | Tenant is the Data Fiduciary for its customers; SSPL is Data Processor for tenant customer data and Data Fiduciary for tenant users' data. Data Processing Agreement in tenant terms. Notice and consent at customer sign-up, consent records, rights requests (access, correction, erasure, grievance), breach notification process, retention limits. Phased compliance dates per the Rules: confirm with counsel |
| TRAI DLT | All SMS templates and headers registered; promotional SMS only with consent and outside DND rules |
| WhatsApp Business Policy | Template approval, opt-in captured, opt-out honoured |
| Consumer protection (E-Commerce Rules) | Clear display of price breakdown, cancellation/refund policy before payment, grievance officer contact on tenant site |
| Payments | No storage of card data; Razorpay Checkout; RBI e-mandate rules for recurring platform billing |
| Accessibility | WCAG 2.1 AA for booking app and themes |

## 9. Constraints

- Tech stack fixed: ASP.NET Core (.NET 10 LTS), React + TypeScript, PostgreSQL, AWS.
- India data residency (Mumbai primary).
- Team size per 00 Q12; MVP scope must fit ~5–6 months.
- Razorpay Partner onboarding and DLT/WhatsApp approvals have lead times of weeks; start in sprint 0.

## 10. Business KPIs to instrument from day 1

| KPI | Source |
|-----|--------|
| Sign-up → trial → published → paid conversion | Platform events |
| Time to first publish (median, p90) | Platform events |
| Online bookings per tenant per month; GMV processed through tenant gateways (for marketing claims, not billing) | Booking + payment events |
| Lead → tour → conversion rate per tenant | CRM events |
| Badge click-throughs and attributed sign-ups | Referral codes |
| Support tickets per tenant per month | Help desk |
| Cost to serve per tenant (AWS tags + messaging) | Cost and Usage Report |
| Logo churn, net revenue retention | Billing |
