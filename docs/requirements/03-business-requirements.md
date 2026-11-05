# 03 · Business Requirements Document (BRD)

> Cowos requirements pack · v0.4 draft

## 1. Purpose

This BRD states what the business needs from Cowos, the rules the product must enforce, and how SSPL makes money. The SRS ([04](04-software-requirements-specification.md)) turns these into testable system requirements. Business requirements are numbered `BR-xx`.

## 2. Business objectives

| ID | Objective | Linked goal |
|----|-----------|-------------|
| BO-1 | Launch a self-serve product that lets a coworking space publish a branded website and booking app without SSPL staff doing the setup | G1 |
| BO-2 | Cover SSPL's production AWS and messaging-provider cost from Fixed plan fees and from metered Usage plan and wallet charges at prices SSPL sets, with every feature on both plans and 0% commission on bookings (D1, CEO-A5, Pricing Rules #10, #12) | G3 |
| BO-3 | Keep the cost to serve one small tenant under ₹400/month (AWS + messaging overhead excluding the tenant's own notification usage) | G3 |
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
| BR-01 | A coworking operator can sign up on cowos.in, verify email and phone, and start a 14-day free trial without talking to SSPL. Self-serve sign-up is available from launch (Pricing Rules #7, #9). |
| BR-02 | The onboarding wizard (4A) collects everything needed to publish a site in one sitting, and saves progress at every step so the tenant can resume. The same data can be edited later in the Tenant Admin settings screens, and the dashboard checklist lists what remains, including the owner-only steps (Razorpay KYC/OAuth, GSTIN/PAN, DNS, ToS and DPA) (Pricing Rule #9). |
| BR-03 | Onboarding offers a choice: "Build my website and booking app" or "I already have a website, I only need the booking app" (4A). |
| BR-04 | Sign-ups referred by a tenant's badge or a partner are attributed (referral code, UTM) and visible to SSPL. |
| BR-05 | SSPL can offer assisted onboarding ("we set it up for you") as a paid one-time service, using the same tools with a tenant-approved support grant (FR-SA-030). Whether the paid service stays under D1 is open (13 OQ12, Pricing Rule #9). |
| BR-27 | An operator can ask for a demo through the "Book a demo" form on the marketing site. The form has spam protection and a privacy notice, and each request is saved to a "Demo requests" list in Super Admin before sales is notified (CEO-A2, Pricing Rule #9). |
| BR-28 | Removed in v0.3 (the pilot programme and Super Admin "Create pilot tenant" with Owner invite and the pre-handover setup grant are removed; tenants sign up self-serve, Pricing Rule #9). |

### 4.2 Information captured during onboarding

| Group | Fields | Required to publish? |
|-------|--------|---------------------|
| Business details | Brand name, legal entity name, entity type, GSTIN (optional, validated format + checksum at launch; legal-name lookup from the GST public search API where available), PAN (if no GSTIN), primary contact name, phone, email, support phone/WhatsApp | Brand name, legal name, contact, phone, email: yes. GSTIN: no (required to issue tax invoices) |
| Location(s) | Address lines, locality, city, state (drives GST place of supply), PIN code, map pin (lat/long; Google Places autocomplete), directions text, landmarks, parking, nearest metro | Yes (one location) |
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
| BR-13a | Tenants draw each floor or zone of their space as a grid seat map, from a template or a blank grid, without help from SSPL, and customers can pick their seat on it like a movie ticket, or take any available seat in one tap. Ships at launch (Pricing Rule #13, 00 §C13). |

### 4.4 Money

| ID | Requirement |
|----|-------------|
| BR-14 | SSPL takes **0% commission** on tenant bookings: Cowos never takes a percentage or per-booking fee from the tenant's booking revenue. This statement appears on the marketing site, in onboarding, on the billing page, and in the terms. |
| BR-25 | Customer-facing pricing is the **tenant's decision**. The tenant sets prices, member rates, promo codes, discounts, optional convenience fees, deposits, and whether gateway fees are passed on. Cowos provides the tools but never sets, funds, or adds anything to the customer's bill. |
| BR-26 | Every tenant can create and manage its own promo codes in the Tenant Admin, without SSPL involvement. |
| BR-15 | Customer payments settle directly to the tenant's own payment gateway account. SSPL never holds or routes tenant funds. |
| BR-16 | Each tenant chooses a plan for paying SSPL. **Fixed plan:** one flat monthly fee, unlimited server usage that is not billed, and messages included up to a monthly fair-use allowance per channel, with messages above it debited from a prepaid wallet. **Usage plan:** no fee; server usage (request units and storage units) is invoiced monthly in arrears and notifications (OTP SMS, other SMS, WhatsApp, email) are debited from a prepaid wallet, each meter priced in slabs that SSPL configures. The Owner can switch plans with effect from the next 1st. There are no feature tiers; every tenant gets every feature (D1, Pricing Rules #2–#6, #12, 5A). Rules in §7. |
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
- Hot desk products are capacity pools (e.g. 40 seats, no fixed seat). Day bookings decrement capacity per day. When the tenant has drawn a seat map, each hot desk product has a "Seat choice" setting: Off (plain pool), Optional (default: the customer picks a seat on the map or takes "Any available seat") or Required. With a map, the pool's capacity is the number of active seats assigned to the product, and every booking holds a specific seat (Rule #13).
- Seat maps are grids per floor or zone. Desk cells are seats; cabin and meeting-room blocks are units. Bookings refer to seats and units, never to grid cells, so redrawing a map never breaks a booking. A seat or block with future bookings can't be deleted or reassigned until those bookings are moved.
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

**Decision:** v1 has four named theme directions (24A): Loft, Metro, Courtyard and Campus. Two ship at launch (Loft and Metro, or the 2 that early validation favours); themes 3–4 ship about 6 weeks after launch, after validation with 3 early tenant owners (CEO-S3). Each theme is a fixed set of section types (Hero, About, Spaces, Amenities, Gallery, Pricing, Testimonials, FAQ, Location, Contact, CTA band, Footer). Tenants switch sections on/off, reorder them within allowed slots, edit content through forms, and set branding tokens (colours, fonts, button shape, light/dark).

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

Decided by D1 (2026-11-02, revised 2026-11-04, revision 2 the same day) and Pricing and Billing Rules #1–#13 in `docs/designs/cowos-platform-v1.md`. Everything in this section ships at launch (P1).

### 7.1 What tenants pay (D1, Pricing Rules #2–#7, #12)
- **No plan tier, plan limit or commission.** Every tenant gets every feature, including Phase 2 features as they ship (Rule #2). There are no upgrades, downgrades, annual plans or plan coupons.
- **Two plans (Rule #12).** The Owner chooses when the tenant becomes Active and can switch at any time; the new plan applies from the 1st of the next month, so each calendar month has one plan.
  - **Fixed plan.** One flat monthly fee, the same for every Fixed tenant, set by SSPL on each rate card version (it changes only with a new version from a future 1st). Server usage is unlimited and not billed; the platform still counts it for SSPL's cost view only (Rule #10). Messages are included up to a monthly fair-use allowance per channel (OTP SMS, other SMS, WhatsApp, email) set on the rate card; above the allowance, each message is debited from the wallet at the Usage plan slab price for the month-to-date count, with the same wallet rules as below. The fee is billed in arrears on the monthly invoice, prorated by day in the month the tenant becomes Active and the month it cancels, plus GST. Days in Suspended, PendingDeletion or after cancellation are not charged. The product owner sets the fee and the allowances before launch (13 OQ35).
  - **Usage plan.** No fee. The meters, slabs and wallet below apply from the first unit.
- **Usage plan meters.** On the Usage plan a tenant pays for server usage and notifications from the first unit, measured on these meters (Rule #3). On the Fixed plan the same meters are counted for the allowances and SSPL's cost view, but only messages above the allowances are charged:

| Meter | Unit | What counts | What does not count |
|-------|------|-------------|---------------------|
| Server: requests | 1 unit = 1,000 page views and API calls | Page views (HTML document requests) on the tenant's website and booking-app hosts, static files not counted (C2-A1); API calls (`/api/*`) on the same hosts, from the same logs (R22). Only 2xx and 304 responses count (R21). Each is counted once | Requests blocked by WAF; SSPL's own health checks and monitoring; redirects, 4xx and 5xx responses; Tenant Admin, Super Admin and support traffic (R21, R22) |
| Server: storage | 1 unit = 1 GB-month | The tenant's stored media and files, averaged from daily snapshots | |
| Notifications: OTP SMS | 1 unit = 1 message | Each sign-in OTP sent by SMS | |
| Notifications: other SMS | 1 unit = 1 message | Each other SMS sent for the tenant | |
| Notifications: WhatsApp | 1 unit = 1 message | Each WhatsApp message sent for the tenant | |
| Notifications: email | 1 unit = 1 message | Each email sent for the tenant | |

- **Fallback and bots.** A WhatsApp message followed by its SMS fallback counts once on each meter (Rule #3). Whether requests from known bots and crawlers that pass WAF are billed is open (13 OQ22).
- **Slabs (Rule #4).** Every meter has its own slab table, applied telescopically like an electricity bill: units in slab 1 at the slab-1 price, the next units at the slab-2 price, and so on. A slab price can be ₹0, which gives a free allowance. SSPL sets slab boundaries and prices in the Super Admin rate card. The product owner configures them before launch (13 OQ21).
- **Rate card versions (Rule #4).** Each version takes effect on the 1st of a month. Invoices and wallet debits always use the version in force when the usage happened. The notice a tenant gets before a new version takes effect is open (13 OQ24).
- **Monthly counts.** Slab counts reset at the start of each calendar month in IST (Rule #4).
- **Server usage** is invoiced monthly in arrears (§7.4, Rule #5).
- **Notifications** are paid from a prepaid wallet (Rule #6). Minimum top-up ₹500. Each message is debited at its channel's slab price for the tenant's month-to-date count on that meter. The count and the debit happen in one atomic step: a conditional update that succeeds only if the balance stays at or above the floor (eng R4). At zero balance, SMS and WhatsApp notifications fall back to email. OTPs and emails continue down to a −₹500 floor. What stops once the floor is reached is open (13 OQ23).
- **Trial (Rule #7).** No plan fee, and server usage during the 14-day trial is not billed. Notifications need wallet balance from day one.
- **Tax.** GST is added on SSPL's invoice. Prices are shown excluding GST, with GST shown on the invoice.
- **Usage detail.** Daily usage records are kept, so a tenant can see the daily detail behind each server invoice.

### 7.2 Transparency requirements
- **Billing page (5A, 12A, 15A, Rule #5, #12).** Shows, in order: (1) next server invoice: issue date (the 1st), due date, and last invoice amount and status; (2) this month so far, starting with one plan line ("Fixed plan · Change", or a scheduled switch "Usage plan from 1 Jan · Cancel", DR3-3A); (3) "0% commission: ₹0 on ₹<bookings> of bookings"; (4) Usage plan: this month's units per server meter with the slab breakdown and "This month so far: about ₹<amount> · estimate, final on the 1st", updated daily; Fixed plan: the fee for the month and messages used against each allowance; (5) Usage plan: daily usage per server meter behind the current month and each past invoice, with "Last month on the other plan: about ₹<amount>" under the detail; (6) during the trial, "No charges during your trial" and the plan choice. Each billing state shows a one-line reason and one action (12A).
- **Wallet page (Rule #6).** Shows the balance, top-up, and per channel (OTP SMS, other SMS, WhatsApp, email) the month-to-date count, the current slab and its price. A warning shows when customers are on email-only notifications.
- **Marketing pricing page (28A, Rule #5, #12; DR3-1A, DR3-9A).** Opens with the "0% commission" headline, one supporting line and a "Start free trial" button. Then one comparison table: sample space profiles as rows, Fixed and Usage as columns, each cell the monthly total + GST, the cheaper cell marked "Lower". The Fixed plan reads "₹<fee>/month · unlimited website and booking-app traffic · includes <n> WhatsApp, <n> SMS, <n> emails a month". Below: what's included on both plans (website, booking app, seat maps, hosting, SSL, backups, updates), the folded rate card with its effective date, and the savings calculator ("You do 300 bookings a month at ₹500. An aggregator charging 15% would take ₹22,500. Cowos takes ₹0 commission."), with the commission rate editable by the visitor. No plan cards. The page never shows SSPL's AWS bill, the tenant count or any tenant's data.

### 7.3 Billing lifecycle

Tenant states: Trial, TrialEnded, Active, PastDue, Restricted, Suspended, PendingDeletion, Deleted, with a Cancelled path (FR-PL-062). All states are in use from launch (Rule #8).

| Stage | Trigger | System behaviour |
|-------|---------|------------------|
| Trial | Self-serve sign-up | 14 days. Publish allowed to platform subdomain only (noindex). Booking app works fully. No plan fee and server usage is not billed; notifications need wallet balance. Daily reminder from day 10 (Rule #7) |
| Trial ended | Day 15 with no plan chosen | **TrialEnded**: site stays live on the platform subdomain and bookings keep working; nothing is billed; "Your trial has ended · Choose a plan to keep your site live · Your site pauses on <date>"; reminders by email and WhatsApp on days 15, 22, 29 and 40. Day 45: **Suspended** (row below), data kept; choosing a plan restores at once (eng re-review, D2) |
| Activation | Tenant adds a payment method and chooses Fixed or Usage | **Active**; the chosen plan starts that day (Rule #12); custom domain and indexing enabled without waiting for the first arrears invoice; first-publish approval queued if not done (Rule #7) |
| Monthly server invoice | The 1st of the month (IST) | Server invoice for the previous month (Fixed plan line, or Usage plan meter lines), due in 10 days (§7.4). Email + WhatsApp to the Owner. With UPI Autopay or e-mandate, the RBI pre-debit notice is sent, then the mandate is charged (Rule #5) |
| Payment failed / not paid on due date | Due date passes | Day 0: status **PastDue**. Retries day 1, 3, 5 (for mandates). Banner in admin. Everything keeps working |
| Grace period | Days 0–7 past due | Full service. Daily reminders |
| Restricted | Day 8–21 past due | Website stays live; every Book CTA switches to "Call / WhatsApp to book", with no billing mention to visitors (10A, R11). Booking app stops accepting **new** online bookings and shows the tenant's phone/WhatsApp. Existing bookings, customer logins, cancellations, refunds and invoices keep working. Admin is read-only except billing, exports and booking management |
| Suspended | Day 22+ past due, or Super Admin suspends the tenant with a reason (FR-SA-011) | Website replaced by a "temporarily unavailable" page in the tenant's theme with name, address, phone and WhatsApp (no SSPL branding beyond the badge) (10A). Owner publishes are blocked with a clear message while Suspended; a render that finishes after the state changed does not go live (CEO-A1). Customer portal read-only (view bookings, download invoices). Owner can log in to pay, export data, or cancel |
| Data retention | Suspended for 90 days, or tenant cancels | Data kept, read-only. Reminder emails at day 30, 60, 80 with export link |
| Deletion | Day 91 after suspension/cancellation | Personal data and content deleted or anonymised; invoices and tax records kept for the statutory period (8 years from end of the relevant financial year under GST law; confirm with advisor) in a restricted archive; audit log retained. Domain mapping removed, SSL certificates released |
| Reactivation | Payment at any stage before deletion, or Super Admin reinstates a suspended tenant (FR-SA-011) | Immediate restore, no data loss |

Rationale for not taking the site down at day 1: customers of the tenant are not party to the dispute; breaking their paid bookings hurts the tenant's reputation and creates refund disputes. Restricting new bookings creates enough pressure.

### 7.4 Server invoices, metering and rating
- Issued by SSPL with SSPL's GSTIN; tenant GSTIN included when provided; IGST for inter-state tenants, CGST+SGST for tenants in SSPL's state.
- Invoice numbering: unique, sequential per financial year, max 16 characters (e.g. `CW/26-27/000123`).
- **Rating (Rule #5, #12).** Billing is monthly in arrears, by calendar month in IST. A rating job runs on the 1st. For a Usage plan tenant it totals last month's billable units per server meter, prices them through the slabs of the rate card version in force, and writes one line per server meter (units and slab breakdown). For a Fixed plan tenant it writes one Fixed plan line: the fee of the rate card version in force, prorated by billable days only in the activation and cancellation months. GST is added. Usage is known by then, so the invoice does not wait for any other data. Usage during the trial is not billed (Rule #7).
- Due 10 days after issue. Paid by Razorpay payment link, or by optional UPI Autopay or e-mandate with a ₹5,000 per-tenant maximum and the RBI pre-debit notice.
- **Metering (Rule #3).**
  - Request units: CloudFront standard logs are written to S3; a daily job maps each host to its tenant and counts page views and `/api/*` calls with a 2xx or 304 status (R21, R22). WAF-blocked requests and SSPL monitoring are excluded.
  - Storage units: a daily snapshot of each tenant's stored media and files; the month's units are the average of the daily snapshots.
  - Notification units: counted at send time, together with the wallet debit (§7.1, eng R4).
  - Usage is stored per tenant, meter and day, so the billing page can show the daily detail behind each invoice (§7.2).
- **Rate card (Rule #4, #12).** Super Admin creates rate card versions. Each version has an effective date (the 1st of a month), the person who published it, the Fixed plan fee and the Fixed plan's monthly message allowance per channel, and a slab table per meter: from unit, to unit (open for the last slab) and price per unit, which can be ₹0.
- **Data (indicative names; the model is in 09):** Meter, UsageDaily (tenant, meter, date, units), MonthlyMeterCount (tenant, meter, month, count; used by wallet debits), RateCardVersion, RateSlab, ServerInvoice and ServerInvoiceLine, PlatformCostMonth.
- Proration: there is no mid-month plan change (a switch applies from the next 1st, Rule #12), and metered units need no proration. The Fixed plan fee is prorated by day only in the month a tenant becomes Active and the month it cancels.
- Refunds of SSPL charges: decision D8 (Proposed).
- Super Admin billing screens show the rate card and its versions, rating runs, server invoices, wallet balances, the dunning dashboard (FR-SA-023) and the SSPL cost view.
- **SSPL cost view (Rule #10).** SSPL loads AWS Data Exports (CUR 2.0) from the production account into Super Admin only to show its actual AWS and messaging-provider cost next to billed revenue, per month, so it can set slab prices. CUR data is never used to bill tenants.

### 7.5 Removed in v0.3 (pilot programme)
The free pilot programme (first 20 owners free for 12 months, SSPL-funded wallet and fair-use caps, end-of-year notices, unconverted-pilot rule) is removed. Pricing and Billing Rules #1–#11 replace the Pilot Rules.

### 7.6 Removed in v0.3 (Billing release)
There is no Billing release. Billing, collection, dunning, metering, the rate card, the trial, self-serve sign-up and the onboarding wizard all ship at launch (12 §1).

## 8. Compliance requirements (business view)

| Area | Requirement |
|------|-------------|
| GST | Correct tax invoices, credit notes, HSN/SAC, place of supply, invoice series per tenant per FY, exports for GSTR-1 filing (CSV in GSTR-1 format, Phase 2) |
| DPDP Act 2023 + DPDP Rules 2025 | Tenant is the Data Fiduciary for its customers; SSPL is Data Processor for tenant customer data and Data Fiduciary for tenant users' data. Data Processing Agreement in tenant terms. Notice and consent at customer sign-up, consent records, rights requests (access, correction, erasure, grievance), breach notification process, retention limits. Phased compliance dates per the Rules: confirm with counsel |
| TRAI DLT | All SMS templates and headers registered; promotional SMS only with consent and outside DND rules |
| WhatsApp Business Policy | Template approval, opt-in captured, opt-out honoured |
| Consumer protection (E-Commerce Rules) | Clear display of price breakdown, cancellation/refund policy before payment, grievance officer contact on tenant site |
| Payments | No storage of card data; Razorpay Checkout; RBI e-mandate rules (pre-debit notice, ₹5,000 per-tenant maximum) for optional UPI Autopay / e-mandate on server invoices (Pricing Rule #5) |
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
| Tenant scorecard: paid online bookings per live tenant per month (payment captured, not fully refunded, counted in the Asia/Kolkata capture month; staff-created and offline excluded), tenants at 10+ this month, 3-month streaks, expansion gate met: 10 tenants at 10+ paid online bookings for 3 consecutive months (G2, CEO-A4, R20) | Booking + payment events |
| Cost recovery: billed server and notification revenue against SSPL's production AWS and messaging-provider cost, per month (G3, CEO-A5, Pricing Rule #10) | SSPL cost view (CUR 2.0) + server invoices and wallet debits |
| Usage per meter: request, storage and notification units per tenant per month (Pricing Rule #3) | Usage records |
| Time to first publish (median, p90) | Platform events |
| Online bookings per tenant per month; GMV processed through tenant gateways (for marketing claims, not billing) | Booking + payment events |
| Lead → tour → conversion rate per tenant | CRM events |
| Badge click-throughs and attributed sign-ups | Referral codes |
| Support tickets per tenant per month | Help desk |
| Cost to serve per tenant: SSPL's monthly AWS and messaging-provider cost divided by live tenants (BO-3) | SSPL cost view (AWS Data Exports, CUR 2.0) |
| Logo churn, net revenue retention | Billing |
