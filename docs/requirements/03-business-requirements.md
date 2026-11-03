# 03 · Business Requirements Document (BRD)

> Cowos requirements pack · v0.2 draft

## 1. Purpose

This BRD states what the business needs from Cowos, the rules the product must enforce, and how SSPL makes money. The SRS ([04](04-software-requirements-specification.md)) turns these into testable system requirements. Business requirements are numbered `BR-xx`.

## 2. Business objectives

| ID | Objective | Linked goal |
|----|-----------|-------------|
| BO-1 | Launch a self-serve product that lets a coworking space publish a branded website and booking app without SSPL staff doing the setup | G1 |
| BO-2 | Recover the production AWS bill and messaging cost from tenants' hosting cost shares, with no subscription, plan fee or margin, and 0% commission on bookings (D1, CEO-A5) | G3 |
| BO-3 | Keep the cost to serve one small tenant under ₹400/month (AWS + messaging overhead excluding pass-through usage) | G3 |
| BO-4 | Make the "Powered by Cowos" badge a measurable acquisition channel | G5 |
| BO-5 | Be compliant from day 1 with GST invoicing rules and the DPDP Act 2023 / DPDP Rules 2025 | G6 |

## 3. Stakeholders

| Stakeholder | Interest |
|-------------|----------|
| SSPL leadership | Cost recovery, brand |
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
| BR-01 | From the Billing release (P1-B), a coworking operator can sign up, verify email and phone, and start a 14-day free trial without talking to SSPL. At launch, new tenants are pilots created by SSPL (BR-28) (CEO-S1). |
| BR-02 | From the Billing release (P1-B), the onboarding wizard collects everything needed to publish a site in one sitting, and saves progress at every step so the tenant can resume. At launch, the same data is entered through the Tenant Admin settings screens, and the dashboard checklist lists what remains, including the owner-only steps (Razorpay KYC/OAuth, GSTIN/PAN, DNS, ToS and DPA) (CEO-S2). |
| BR-03 | From the Billing release (P1-B), onboarding offers a choice: "Build my website and booking app" or "I already have a website, I only need the booking app" (CEO-S2). |
| BR-04 | From the Billing release (P1-B), sign-ups referred by a tenant's badge or a partner are attributed (referral code, UTM) and visible to SSPL (CEO-S1). |
| BR-05 | SSPL can offer assisted onboarding ("we set it up for you") as a paid one-time service, using the same tools with a support grant. Pilots get this setup free, as an exception (Pilot Rules #9). Whether the paid service stays under D1 is open (13 OQ12). |
| BR-27 | At launch, an operator can ask to join the pilot through the "Book a demo" form on the marketing site. The form has spam protection and a privacy notice, and each request is saved to a "Pilot requests" list in Super Admin before sales is notified (CEO-A2). |
| BR-28 | SSPL creates pilot tenants from Super Admin directly in the Active state with a pilot flag, and invites the Owner (single-use invite, 72-hour expiry, phone OTP, owner sets password and MFA). SSPL sets up the tenant through a pre-handover setup grant that ends when the Owner accepts; after that, tenant-approved support grants apply (CEO A2, R19, Pilot Rules #4, #9). |

### 4.2 Information captured during onboarding

| Group | Fields | Required to publish? |
|-------|--------|---------------------|
| Business details | Brand name, legal entity name, entity type, GSTIN (optional, validated format + checksum at launch; legal-name lookup from the GST public search API where available from P1-B, CEO-S2), PAN (if no GSTIN), primary contact name, phone, email, support phone/WhatsApp | Brand name, legal name, contact, phone, email: yes. GSTIN: no (required to issue tax invoices) |
| Location(s) | Address lines, locality, city, state (drives GST place of supply), PIN code, map pin (lat/long; Google Places autocomplete from P1-B, CEO-S2), directions text, landmarks, parking, nearest metro | Yes (one location) |
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
| BR-07 | Every tenant website and booking app shows a "Powered by Cowos" badge in the footer, linked to the Cowos marketing site with the tenant's referral code. Tenants cannot remove or hide it in v1. |
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
| BR-26 | Every tenant can create and manage its own promo codes in the Tenant Admin, without SSPL involvement. |
| BR-15 | Customer payments settle directly to the tenant's own payment gateway account. SSPL never holds or routes tenant funds. |
| BR-16 | Tenants pay SSPL only their monthly share of the production AWS bill plus messaging at provider cost from a prepaid wallet. There is no subscription, plan fee, plan tier or plan limit; every tenant gets every feature. The billing page shows the AWS bill breakdown behind the charge and what each messaging line cost (D1, Pilot Rules #2, #10, 5A). Rules in §7. |
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

**Decision:** v1 has four named theme directions (24A): Loft, Metro, Courtyard and Campus. Two ship at launch (Loft and Metro, or the 2 that pilot validation favours); themes 3–4 ship about 6 weeks after launch, after validation with 3 pilot owners (CEO-S3). Each theme is a fixed set of section types (Hero, About, Spaces, Amenities, Gallery, Pricing, Testimonials, FAQ, Location, Contact, CTA band, Footer). Tenants switch sections on/off, reorder them within allowed slots, edit content through forms, and set branding tokens (colours, fonts, button shape, light/dark).

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
- "Custom code" slot for analytics scripts only (Google Analytics 4, Meta Pixel, Google Tag Manager IDs as fields, not free HTML) in v1. Free HTML embed blocks are a later feature with a content security policy review; there are no paid tiers (D1).
- AI-assisted copy (Phase 2): generate hero text, about section and FAQ from business details.

## 7. Platform charges and tenant lifecycle

Decided by D1 (2026-11-02) and Pilot Rules #1–#10. Automated billing ships in the Billing release (P1-B, launch + about 4 months; CEO-S1). §7.6 lists what exists at launch.

### 7.1 What tenants pay (D1, Pilot Rules #10)
- **No subscription, plan fee, plan tier, plan limit or commission.** Every tenant gets every feature, including Phase 2 features as they ship (Pilot Rules #2). There are no upgrades, downgrades, annual plans or plan coupons.
- **Hosting cost share.** The cost pool is the production AWS account's monthly bill, read from AWS Data Exports (CUR 2.0), net of AWS credits. SSPL pays for staging, dev, CI, the marketing site and Super Admin outside the pool.
- **Counted tenants.** Tenants in Active, PastDue or Restricted for any part of the month, including pilots in their free period, prorated by days live. The pool is split equally by tenant-days. SSPL's test and demo tenants are counted, and SSPL pays their share. SSPL also pays pilots' shares during the free year.
- **Unpaid shares.** SSPL absorbs them. They are never passed to other tenants.
- **Storage.** No separate charge. Fair use is 10 GB of media per tenant, with no extra charge in v1.
- **Tax.** GST is added on SSPL's invoice. Prices are shown excluding GST, with GST shown on the invoice.
- **Messaging wallet.** Prepaid at provider cost, with no free allowance. Minimum top-up ₹500. Per-message prices are displayed (SMS; WhatsApp utility, authentication and marketing template categories). Each send debits the wallet in one conditional update that succeeds only if the balance stays at or above the floor (R4). At zero balance, notifications fall back to email only. Sign-in OTPs still go out with the wallet allowed down to −₹500; below that, customers sign in by email OTP. Pilots use an SSPL-funded wallet (§7.5).
- **Indicative amount.** About ₹1,250–2,250 per tenant per month at 20 tenants, falling to about ₹100–400 at 200+ tenants [estimated from 00 §C2].
- **Split method.** The equal split ships in v1. A weighted split (for example by bookable resources or traffic) is reviewed after 3 months of real bills from the Billing release (13 OQ19).

### 7.2 Transparency requirements
- **Billing page (5A, 12A, 15A).** Shows, in order: (1) next invoice: issue date (the 10th), last month's amount, due date, any adjustment line; pilots see "Free until <date> · SSPL covered ₹X this month"; (2) "0% commission: ₹0 on ₹<bookings> of bookings"; (3) messaging wallet balance and usage by channel and template category, with a warning when customers are on email-only notifications; (4) the AWS bill breakdown by service category plus the counted-tenant number, never other tenants' usage; (5) "This month so far: about ₹<low>–<high> · estimate, final on the 10th". The estimate is computed once a day after the CUR refresh: month-to-date pool plus a straight-line projection to month end, divided by today's counted tenants; the high value adds a 15% buffer (R15). From the 60-day notice, pilots see "After your free year: about ₹X/month at today's numbers". Each billing state shows a one-line reason and one action (12A). There is no plan, limit or upgrade UI. Items that need billing data (invoice, AWS breakdown, estimate, "SSPL covered ₹X") arrive with the Billing release.
- **Marketing pricing page (28A, R12).** One headline: "0% commission. You pay only your share of our real AWS bill." Below it: "about ₹X per space per month", which is last month's actual average share, plus the two months before it; what's included (website, booking app, hosting, SSL, backups, updates); messaging at cost with per-message prices; and the savings calculator ("You do 300 bookings a month at ₹500. An aggregator charging 15% would take ₹22,500. Cowos takes ₹0 commission."), with the commission rate editable by the visitor. No plan cards or plan picker. The per-space number comes from a JSON written after each monthly billing run; it never contains the total AWS bill, the tenant count or any tenant's data, and the page keeps the last published number if a run fails. At launch the page shows the promise and the pilot offer; "last month's actual" is added after the first billing run (P1-B).

### 7.3 Billing lifecycle

Tenant states: Trial, Active, PastDue, Restricted, Suspended, PendingDeletion, Deleted, with a Cancelled path (FR-PL-062). Pilots carry a pilot flag and a free-period end date.

| Stage | Trigger | System behaviour |
|-------|---------|------------------|
| Pilot free period (P1) | Super Admin creates a pilot tenant (BR-28) | Created on production directly in **Active** with a pilot flag. The free-period end date is set at the first live publish, 12 months later. Publishing, custom domains and search indexing work as for a paying tenant; SSPL's first-publish review still applies. No payment method or mandate is needed; trial expiry, trial publish restrictions and dunning never apply. No ₹0 invoices (Pilot Rules #3, #4) |
| Trial (P1-B) | Self-serve sign-up | 14 days. Publish allowed to platform subdomain only (noindex). Booking app works fully. Daily reminder from day 10. Never applies to pilots |
| Activation (P1-B) | Non-pilot tenant adds a payment method at the end of its trial | **Active**; custom domain and indexing enabled without waiting for the first arrears invoice; first-publish approval queued if not done (Pilot Rules #7) |
| Monthly invoice (P1-B) | The 10th, once AWS marks the previous month's bill final | Hosting invoice for the previous month, due in 10 days (§7.4). Email + WhatsApp to the Owner. With UPI Autopay or e-mandate, the RBI pre-debit notice is sent, then the mandate is charged |
| Payment failed / not paid on due date (P1-B) | Due date passes | Day 0: status **PastDue**. Retries day 1, 3, 5 (for mandates). Banner in admin. Everything keeps working |
| Grace period (P1-B) | Days 0–7 past due | Full service. Daily reminders |
| Restricted (P1-B) | Day 8–21 past due | Website stays live; every Book CTA switches to "Call / WhatsApp to book", with no billing mention to visitors (10A, R11). Booking app stops accepting **new** online bookings and shows the tenant's phone/WhatsApp. Existing bookings, customer logins, cancellations, refunds and invoices keep working. Admin is read-only except billing, exports and booking management |
| Suspended | Day 22+ past due (P1-B), or Super Admin suspends the tenant with a reason (FR-SA-011, P1) | Website replaced by a "temporarily unavailable" page in the tenant's theme with name, address, phone and WhatsApp (no SSPL branding beyond the badge) (10A). Owner publishes are blocked with a clear message while Suspended; a render that finishes after the state changed does not go live (CEO-A1). Customer portal read-only (view bookings, download invoices). Owner can log in to pay, export data, or cancel |
| Unconverted pilot (P1-B) | First post-free-year invoice not paid by its due date plus 14 days | **Restricted** as above, for 90 days instead of moving to Suspended at day 22. After the 90 days, the normal Suspended → retention → deletion lifecycle applies, with its notices (Pilot Rules #6) |
| Data retention | Suspended for 90 days, or tenant cancels | Data kept, read-only. Reminder emails at day 30, 60, 80 with export link |
| Deletion | Day 91 after suspension/cancellation | Personal data and content deleted or anonymised; invoices and tax records kept for the statutory period (8 years from end of the relevant financial year under GST law; confirm with advisor) in a restricted archive; audit log retained. Domain mapping removed, SSL certificates released |
| Reactivation | Payment at any stage before deletion, or Super Admin reinstates a suspended tenant (FR-SA-011) | Immediate restore, no data loss |

**At launch (Q1):** no billing events exist, so the only states in use are Active (pilot) and Suspended. Super Admin suspends and reinstates manually (FR-SA-011). The API rejects any move into Trial, PastDue or Restricted with 409 until the Billing release enables dunning.

A former pilot that pays its first invoice and later misses one follows the standard stages above.

Rationale for not taking the site down at day 1: customers of the tenant are not party to the dispute; breaking their paid bookings hurts the tenant's reputation and creates refund disputes. Restricting new bookings creates enough pressure.

### 7.4 Platform invoices (P1-B)
- Issued by SSPL with SSPL's GSTIN; tenant GSTIN included when provided; IGST for inter-state tenants, CGST+SGST for tenants in SSPL's state.
- Invoice numbering: unique, sequential per financial year, max 16 characters (e.g. `CW/26-27/000123`).
- Billing is monthly in arrears. A monthly job runs on the 10th and checks that AWS has marked the bill final (CUR 2.0 column `bill_invoice_id` populated). It builds the cost pool (net of credits, with the service breakdown), splits it by tenant-days into each tenant's share, and writes the hosting invoice with GST. If the data is missing, invoices are held, Finance is alerted and the job retries daily (R2).
- The job re-reads the previous month. Credits, refunds or support fees AWS applied after the invoice appear as an adjustment line on the next invoice (R1).
- Due 10 days after issue. Paid by Razorpay payment link, or by optional UPI Autopay or e-mandate with a ₹5,000 per-tenant maximum and the RBI pre-debit notice.
- Proration on mid-cycle upgrades: removed in v0.2 (no plans, D1). Shares are prorated by days live instead.
- Refunds of SSPL charges: decision D8 (Proposed). Later AWS corrections reach tenants through the adjustment line.
- Super Admin billing screens show the cost pool, invoices and billing runs (28A), and the dunning dashboard (FR-SA-023).

### 7.5 Pilot programme (Pilot Rules #1–#10)
- **Who and what.** The first 20 owners in the beachhead city get 12 months fully free, with the full feature set (Pilot Rules #2). Summary in 00 §4.
- **Free-year start.** Per tenant, from the tenant's first live publish on production, ending 12 months later. There is no shared cohort end date (Pilot Rules #3).
- **Representation.** As in the "Pilot free period" row of §7.3 (Pilot Rules #4).
- **Messaging fair use (provisional).** 1,000 SMS + WhatsApp notification messages per pilot per month. At the cap, notifications fall back to email only and SSPL is alerted. Sign-in OTPs are not counted in this cap: SSPL is alerted at 1,000 OTPs per pilot per month, and at 2,000 phone OTP pauses for that tenant for the rest of the month, so customers sign in by email OTP. Platform per-number OTP rate limits (NFR-SEC-03) apply as for any tenant. Every message sent counts, including a fallback SMS: a WhatsApp message followed by its SMS fallback counts as 2 (Pilot Rules #5).
- **End of the free year.** Notices at 60, 30 and 7 days before the end date. The tenant adds a payment method (payment link details or UPI Autopay/e-mandate) with the 60-day notice. From month 13 the tenant pays its monthly hosting cost share; month 13's share is invoiced on the 10th of month 14. If that first invoice isn't paid: 14-day grace, then Restricted for 90 days (§7.3). The website stays live, existing customer bookings, cancellations and invoices keep working, data export stays available, and new online bookings pause until payment (Pilot Rules #6).
- **Non-pilot tenants and billing validation.** Non-pilot spaces can sign up in any city from the Billing release, paying their hosting cost share after the 14-day trial. SSPL's own test tenant pays real hosting charges from the Billing release. **Exit criterion before the first pilot converts:** at least one real monthly hosting charge has been paid, and one dunning cycle has run end to end on production (Pilot Rules #7, CEO-S1).
- **Cost to SSPL.** SSPL pays pilots' hosting shares and the SSPL-funded wallet for the pilot year (13 R25).

### 7.6 What exists at launch and what waits for the Billing release (CEO-S1, CEO-S2)

| At launch (P1) | Billing release (P1-B) |
|----------------|------------------------|
| Tenant state model with manual Super Admin suspend/reinstate and the 409 guard (Q1); suspended page switch (CEO-A1) | CUR billing job and billing tables; payment link and UPI Autopay / e-mandate collection |
| Pilot flag and free-period end date; Super Admin create pilot with Owner invite (BR-28) | Dunning with automatic Restricted and Suspended moves; dunning dashboard (FR-SA-023) |
| SSPL-funded pilot wallet with fair-use caps | Non-pilot wallet top-up |
| Pilot request form (BR-27); pilot scorecard (CEO-A4) | Public self-serve sign-up, referral capture, slug availability check; 14-day trial; sign-up, trial and conversion metrics (FR-SA-051) |
| Pricing page with the promise and the pilot offer | Pricing JSON and "last month's actual"; daily charge estimate |
| Settings screens and dashboard checklist; GSTIN format/checksum and PAN derivation | Onboarding wizard, GSTIN legal-name lookup, Places autocomplete, booking-only path, demo inventory with its sample publish check |

## 8. Compliance requirements (business view)

| Area | Requirement |
|------|-------------|
| GST | Correct tax invoices, credit notes, HSN/SAC, place of supply, invoice series per tenant per FY, exports for GSTR-1 filing (CSV in GSTR-1 format, Phase 2) |
| DPDP Act 2023 + DPDP Rules 2025 | Tenant is the Data Fiduciary for its customers; SSPL is Data Processor for tenant customer data and Data Fiduciary for tenant users' data. Data Processing Agreement in tenant terms. Notice and consent at customer sign-up, consent records, rights requests (access, correction, erasure, grievance), breach notification process, retention limits. Phased compliance dates per the Rules: confirm with counsel |
| TRAI DLT | All SMS templates and headers registered; promotional SMS only with consent and outside DND rules |
| WhatsApp Business Policy | Template approval, opt-in captured, opt-out honoured |
| Consumer protection (E-Commerce Rules) | Clear display of price breakdown, cancellation/refund policy before payment, grievance officer contact on tenant site |
| Payments | No storage of card data; Razorpay Checkout; RBI e-mandate rules (pre-debit notice, ₹5,000 per-tenant maximum) for optional UPI Autopay / e-mandate on hosting invoices (Pilot Rules #10) |
| Accessibility | WCAG 2.1 AA for booking app and themes |

## 9. Constraints

- Tech stack fixed: ASP.NET Core (.NET 10 LTS), React + TypeScript, PostgreSQL, AWS.
- India data residency (Mumbai primary).
- Team size per 00 Q12; MVP scope must fit ~5–6 months.
- Razorpay Partner onboarding and DLT/WhatsApp approvals have lead times of weeks; start in sprint 0.

## 10. Business KPIs to instrument from day 1

| KPI | Source |
|-----|--------|
| Sign-up → trial → published → paid conversion (from P1-B) | Platform events |
| Pilot scorecard: paid online bookings per pilot per month (payment captured, not fully refunded, counted in the Asia/Kolkata capture month; staff-created and offline excluded), pilots at 10+ this month, 3-month streaks, expansion gate met (CEO-A4, R20) | Booking + payment events |
| Cost recovery: tenant hosting payments as a share of the production AWS bill and messaging cost, with SSPL's unrecovered cost alongside (G3, CEO-A5; from P1-B) | Cost pool + platform invoices |
| Time to first publish (median, p90) | Platform events |
| Online bookings per tenant per month; GMV processed through tenant gateways (for marketing claims, not billing) | Booking + payment events |
| Lead → tour → conversion rate per tenant | CRM events |
| Badge click-throughs and attributed sign-ups | Referral codes |
| Support tickets per tenant per month | Help desk |
| Cost to serve per tenant (AWS tags + messaging) | AWS Data Exports (CUR 2.0) |
| Logo churn, net revenue retention | Billing |
