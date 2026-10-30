# 04 · Software Requirements Specification (SRS): Functional Requirements

> Cowos requirements pack · v0.1 draft · Non-functional requirements are in [07](07-non-functional-requirements.md)

## 1. Conventions

**IDs:** `FR-<APP>-<NNN>`

| Prefix | Application |
|--------|-------------|
| FR-MW | Cowos Marketing Web App |
| FR-TA | Tenant Admin Portal |
| FR-TW | Tenant Website (public, pre-rendered) |
| FR-BA | Tenant Booking App (customer-facing) |
| FR-EM | Embedding / existing-website integration |
| FR-SA | Cowos Super Admin Portal |
| FR-PL | Platform services (cross-cutting, no UI of their own) |

**Priority (MoSCoW, relative to the MVP release):**
- **M** Must: MVP cannot launch without it.
- **S** Should: planned for MVP; can slip to the first post-MVP release without blocking launch.
- **C** Could: Phase 2 or 3 candidate.
- **W** Won't (this release): explicitly deferred; kept for traceability.

**Phase:** P1 = MVP, P2, P3 (see [12](12-release-plan.md)).

---

## 2. FR-MW · Cowos Marketing Web App

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-MW-001 | The site shall present product pages: home, features (website builder, booking app, payments, invoices, CRM), pricing, demo sites, about SSPL, contact, legal (terms, privacy, DPA, refund policy). | M | P1 |
| FR-MW-002 | The pricing page shall state "0% commission on bookings" above the plan cards and show each plan's price (excl. GST), limits, and included features. | M | P1 |
| FR-MW-003 | The pricing page shall show the "Where your fee goes" allocation and a messaging price table (per SMS, per WhatsApp template category). | M | P1 |
| FR-MW-004 | The pricing page shall include a commission savings calculator with editable inputs (bookings/month, average booking value, aggregator commission %). | S | P1 |
| FR-MW-005 | The site shall offer sign-up with name, work email, mobile number, business name, city, and a password or Google sign-in; email and mobile are verified by OTP before the trial starts. | M | P1 |
| FR-MW-006 | The site shall capture referral codes (`?ref=`) and UTM parameters, persist them for 30 days in a first-party cookie, and attach them to the tenant at sign-up. | M | P1 |
| FR-MW-007 | The site shall link to 2–3 live demo tenant sites (one per flagship theme) with sample inventory and test-mode payments. | S | P1 |
| FR-MW-008 | The site shall provide a "Book a demo" form that creates a lead in SSPL's CRM / notifies sales by email. | S | P1 |
| FR-MW-009 | The site shall be pre-rendered for SEO, with structured data (Organization, SoftwareApplication, FAQPage) and a sitemap. | M | P1 |
| FR-MW-010 | The site shall host a help centre (searchable articles: DNS guides per registrar, Razorpay connection, GST setup). | S | P1 |
| FR-MW-011 | The site shall host a blog / SEO landing pages ("coworking software India", city pages). | C | P2 |
| FR-MW-012 | The site shall show a domain availability check during sign-up for the platform subdomain slug. | M | P1 |

---

## 3. FR-TA · Tenant Admin Portal

### 3.1 Account, onboarding wizard

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-001 | On first login the portal shall start an onboarding wizard with steps: (1) Goal choice: full website + booking app, or booking app only; (2) Business details; (3) Location and hours; (4) Branding; (5) Theme; (6) Inventory (at least one product); (7) Payments; (8) Policies; (9) Preview; (10) Publish. | M | P1 |
| FR-TA-002 | Each wizard step shall auto-save, show completion status, and allow skipping non-required steps; a checklist on the dashboard shows what remains before publishing. | M | P1 |
| FR-TA-003 | The business details step shall validate GSTIN format and checksum, derive state and PAN from the GSTIN, and optionally fetch legal name from the GST public search provider; the tenant can override the trade name. | M | P1 |
| FR-TA-004 | The location step shall use Google Places autocomplete to fill address and coordinates, show a draggable map pin, and accept directions/landmark text. | M | P1 |
| FR-TA-005 | The portal shall let the tenant choose a platform subdomain slug (3–30 chars, lowercase letters, digits, hyphens; reserved words blocked) during onboarding. | M | P1 |
| FR-TA-006 | The portal shall show a "Booking app only" path that skips theme/website content steps and leads to the integration step (FR-EM). | M | P1 |
| FR-TA-007 | The Owner shall be able to enable MFA (TOTP) and shall be required to enable it before connecting a payment gateway or a custom domain. | M | P1 |
| FR-TA-008 | The portal shall offer demo inventory (sample hot desk, meeting room, cabin) the tenant can edit or delete, so preview isn't empty. | S | P1 |

### 3.2 Site builder (themes, branding, content)

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-020 | The portal shall offer at least 4 themes at launch, each with a live thumbnail and a full preview using the tenant's own content. | M | P1 |
| FR-TA-021 | Switching themes shall keep all content (sections and fields are theme-independent); only layout and styling change. | M | P1 |
| FR-TA-022 | Branding editor: logo upload (SVG/PNG/JPG ≤5 MB), optional dark-mode logo, favicon (auto-generated if not provided), primary/secondary/accent colours with a WCAG contrast checker that blocks combinations failing AA for text on buttons, font pair from a curated list (≥8 pairs, including Indian-script-capable fonts), button shape (rounded/pill/square), theme mode (light/dark/auto). | M | P1 |
| FR-TA-023 | Section editor: enable/disable, reorder within the theme's allowed order, and edit each section through a form: Header (logo, nav auto-generated, CTA label + target: Book / Tour / WhatsApp / Call), Hero (headline, sub-headline, background image or short video, primary and secondary CTA), About, Amenities (from amenity list), Spaces & Pricing (auto-built from inventory, choose which products to show and whether to show prices), Gallery (up to 40 images, captions, alt text required), Testimonials (name, role, company, quote, photo, rating), FAQ (Q&A pairs; default coworking FAQ set offered), Location (map, address, directions, nearby transit), Contact form (fields configurable: name, phone, email, company, interest, message), CTA band, Footer (text, links, social links, contact). | M | P1 |
| FR-TA-024 | Every image field shall require alt text (prefilled with a suggestion the tenant can edit) and run through the media pipeline (FR-PL-030). | M | P1 |
| FR-TA-025 | The tenant shall be able to create additional simple pages (title, slug, rich text, images) up to the plan limit, and add them to the nav. | S | P1 |
| FR-TA-026 | SEO settings per page: title, meta description, social share image; site-level: Google Analytics 4 ID, Google Tag Manager ID, Meta Pixel ID, Google Search Console verification token. | M | P1 |
| FR-TA-027 | The builder shall show a live preview pane updated within 1 second of an edit, with device toggles (desktop 1440, tablet 768, mobile 390) and a "open preview in new tab" shareable link (token-protected, expires in 7 days). | M | P1 |
| FR-TA-028 | The preview shall include the booking app in the same branding with real inventory and pricing, and payment in test mode. | M | P1 |
| FR-TA-029 | AI copy assistant: generate hero, about, FAQ and meta descriptions from business details; tenant reviews before saving. | C | P2 |
| FR-TA-030 | Free HTML embed section (Pro only), sandboxed and reviewed against a content security policy. | C | P3 |
| FR-TA-031 | Blog/news section with posts. | C | P2 |
| FR-TA-032 | Multi-language content (Telugu, Hindi). | W | P3 |

### 3.3 Publishing and versions

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-040 | The portal shall hold one working draft per tenant site; edits never affect the live site until published. | M | P1 |
| FR-TA-041 | Publish shall run pre-publish checks: required fields present, badge present, all images have alt text, policies present, at least one bookable or enquirable product, payment gateway connected if any Instant Book product exists, platform subscription active or in trial (trial: platform subdomain only). Failures are listed with links to fix. | M | P1 |
| FR-TA-042 | The first publish shall go to SSPL approval (FR-SA-010); the tenant sees status "In review" and gets notified on decision. Later publishes go live immediately. | M | P1 |
| FR-TA-043 | Each publish shall create an immutable version with timestamp, publisher, and optional note; version history lists the last 20 versions. | M | P1 |
| FR-TA-044 | Rollback shall make a chosen previous version live within 60 seconds and create a new version entry ("Rolled back to v12"). Inventory and prices are live data and not part of the website version (stated in the UI). | M | P1 |
| FR-TA-045 | The tenant shall be able to compare draft vs live (list of changed sections). | C | P2 |
| FR-TA-046 | Scheduled publish (publish at date/time). | C | P2 |

### 3.4 Domains

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-050 | The portal shall show the platform URLs (`<slug>.cowos.site`, `<slug>.cowos.site/book`) as always available. | M | P1 |
| FR-TA-051 | The tenant shall add a custom domain by entering it; the system detects the DNS provider (via NS lookup) and shows provider-specific instructions: CNAME `www` → tenant edge target; CNAME `book` → booking target (if chosen); TXT `_cowos-verify` for ownership; apex forwarding instructions. | M | P1 |
| FR-TA-052 | The system shall check DNS every 5 minutes for 72 hours after adding a domain (and on "Check now"), show per-record status (found / wrong value / missing), and on success request an SSL certificate automatically. | M | P1 |
| FR-TA-053 | The tenant shall choose booking app placement on custom domain: `book.<domain>` (default) or `<domain>/book`. | M | P1 |
| FR-TA-054 | When a custom domain is live, the platform subdomain shall 301-redirect to it (canonical URL) for SEO. | M | P1 |
| FR-TA-055 | The system shall alert the tenant (email + admin banner) if DNS later stops pointing to Cowos or the certificate cannot renew. | M | P1 |
| FR-TA-056 | "Buy a domain" via a registrar reseller API. | C | P3 |

### 3.5 Inventory: locations, products, resources

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-060 | The tenant shall manage locations (name, address, map pin, hours, holidays, amenities, photos, contact, GST registration per state if different). MVP UI optimised for 1; data and API support many. | M | P1 |
| FR-TA-061 | The tenant shall create products of type: Hot Desk (capacity pool), Dedicated Desk, Private Cabin, Meeting Room, Event Space, Virtual Office, Custom; each with name, description, photos, amenities, capacity per unit, booking mode (Instant / Request / Enquire), visibility (public / members only / hidden). | M | P1 |
| FR-TA-062 | For resource-based products (dedicated desk, cabin, meeting room, event space) the tenant shall define units (e.g. "Cabin 3, 6 seats, 2nd floor"), each bookable individually; bulk create ("add 20 desks D-01 to D-20"). | M | P1 |
| FR-TA-063 | For pool products (hot desk), the tenant shall set capacity per day and optional per-time-slot capacity. | M | P1 |
| FR-TA-064 | Pricing per product: hour, half-day, day, week, month prices (any subset), minimum and maximum booking duration, time slot granularity (15/30/60 min), member price per unit, GST inclusive/exclusive, SAC code (default 997212, editable), GST rate (default 18%). | M | P1 |
| FR-TA-065 | Add-ons per product: name, price, pricing basis (per booking / per hour / per person), max quantity, optional stock. | S | P1 |
| FR-TA-066 | Availability rules per product or unit: buffer before/after (minutes), lead time, advance window, allowed days/hours (overrides location hours), maintenance blocks with reason. | M | P1 |
| FR-TA-067 | Peak/off-peak pricing (time-of-day and day-of-week multipliers). | C | P2 |
| FR-TA-068 | Floor plan: upload a floor image per floor and drop pins for units; customers pick from the plan. | C | P2 |
| FR-TA-069 | Tour slots: define tour availability (days, times, duration, staff capacity) for Enquire products. | S | P1 |
| FR-TA-070 | Security deposit amount per monthly product. | S | P1 |
| FR-TA-071 | Optional convenience / booking fee per product (flat or %), off by default, shown as a separate line to customers; collected by the tenant. Cowos adds no fee of its own to any customer payment. | S | P1 |

### 3.6 Policies

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-080 | Cancellation policy builder: tiers of (hours/days before start → refund %), no-show rule, gateway fee handling, reschedule rule; assign policies to products; default templates per product type (BRD §5.4). | M | P1 |
| FR-TA-081 | Policy text shown to customers is generated from the configured rules so text and behaviour always match; tenant can add free-text notes. | M | P1 |
| FR-TA-082 | Privacy policy and terms generated from templates with tenant details, grievance officer contact; tenant can edit (with a warning that edits are their responsibility). | M | P1 |

### 3.7 Bookings and front desk

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-090 | Bookings calendar: day/week/month views, filter by location/product/unit/status, colour by status; resource timeline view for meeting rooms. | M | P1 |
| FR-TA-091 | Today view for front desk: today's bookings by time, check-in status, pending requests, unpaid bookings, visitor count. | M | P1 |
| FR-TA-092 | Staff shall create a booking for a customer (search or quick-create customer), choose payment: send payment link, record offline payment, use member credits, or mark complimentary (with reason, Owner/Manager only). | M | P1 |
| FR-TA-093 | Approve / decline Request-to-Book requests with optional message; approval sends payment link; auto-decline after the configured SLA. | M | P1 |
| FR-TA-094 | Cancel a booking: system calculates the refund per policy; Owner/Manager may override amount with a reason; refund executes via gateway or as credit; cancellation notifications sent. | M | P1 |
| FR-TA-095 | Reschedule a booking to an available slot; price difference handled (charge link or credit). | S | P1 |
| FR-TA-096 | Check-in by scanning the customer's booking QR code with the device camera, or by tapping check-in; mark no-show after configurable grace (default 30 min). | S | P1 |
| FR-TA-097 | Booking detail shows full timeline: created, held, paid, invoiced, notifications sent (with delivery status), check-in, changes, refunds, with actor for each event. | M | P1 |
| FR-TA-098 | Recurring bookings for meeting rooms (weekly, N occurrences) created by staff. | C | P2 |
| FR-TA-099 | Waitlist for fully booked day passes/events. | C | P3 |

### 3.8 Customers, companies, leads

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-110 | Customer list with search (name, phone, email, company), filters (member, active booking, last visit), and profile showing bookings, payments, invoices, memberships, credits, notes, consent records. | M | P1 |
| FR-TA-111 | Companies: name, GSTIN (validated), billing address, admin contact, members; bookings by members can bill to the company. | M | P1 |
| FR-TA-112 | Leads pipeline: leads from website contact form, Enquire products, tour bookings, and manual entry; stages New → Contacted → Tour scheduled → Tour done → Proposal → Won / Lost (with reason); owner assignment; follow-up date; notes; one-click WhatsApp/call. | M | P1 |
| FR-TA-113 | Convert lead to customer and create a monthly booking/membership from the lead. | S | P1 |
| FR-TA-114 | Import customers from CSV (with mapping and validation report). | S | P1 |
| FR-TA-115 | Export customers, bookings, payments, invoices to CSV/Excel. | M | P1 |
| FR-TA-116 | DPDP request handling: view/download a customer's data, correct, erase (anonymise while retaining tax invoices), with a request log and due dates. | M | P1 |
| FR-TA-117 | Customer tags and segments; send a WhatsApp/email broadcast to a segment (marketing templates, consent-checked). | C | P2 |

### 3.9 Payments, invoices, credit notes

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-130 | Connect Razorpay via Partner OAuth; show connected account name/ID, mode (test/live), and connection health; disconnect with warning. | M | P1 |
| FR-TA-131 | Fallback: enter Razorpay key ID and secret; system validates by a test API call and stores the secret encrypted (KMS); secret never displayed again. | M | P1 |
| FR-TA-132 | Test mode toggle for previews and demos; live mode required for publish of Instant Book products. | M | P1 |
| FR-TA-133 | Payments list: date, customer, booking, method, amount, gateway fee (from gateway data), status, refund status; filter and export. | M | P1 |
| FR-TA-134 | Record offline payment: method (cash, UPI direct, bank transfer, cheque), reference, date, amount, against booking or invoice. | M | P1 |
| FR-TA-135 | Invoice settings: invoice prefix per location, numbering series per FY (auto-reset 1 April), logo, address, GSTIN, bank details, signature image, footer notes, terms. | M | P1 |
| FR-TA-136 | Invoices generated automatically on payment (or on booking confirmation for credit/offline flows, configurable); B2B invoice when customer/company GSTIN present; Bill of Supply / receipt when tenant has no GSTIN. | M | P1 |
| FR-TA-137 | Credit note on refund or price reduction after invoice, referencing the original invoice. | M | P1 |
| FR-TA-138 | Monthly consolidated invoice for a company (all member bookings in the month). | S | P2 |
| FR-TA-139 | GSTR-1 friendly export (B2B, B2C, credit notes, HSN/SAC summary). | S | P2 |
| FR-TA-140 | Security deposits: record collection, hold, refund with deductions and notes. | S | P1 |
| FR-TA-141 | Record TDS deducted by corporate customers against an invoice. | C | P2 |
| FR-TA-142 | Second gateway option (Cashfree / PhonePe PG). | C | P2 |
| FR-TA-143 | E-invoicing (IRN, QR) through a GSP for eligible tenants. | C | P3 |
| FR-TA-144 | Accounting export / sync to Tally and Zoho Books. | C | P3 |

### 3.10 Memberships, credits, coupons

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-150 | Promo codes: the tenant (Owner; Manager for assigned locations) shall create, edit, pause and archive promo codes with code, type (% / flat), value, max discount, min order, validity dates, usage limits (total/per customer), applicable products and locations, first-booking-only. Available on all plans. | M | P1 |
| FR-TA-156 | Promo code visibility: Public (optionally shown as an offer banner on the website and booking app) or Private (not shown; tenant shares the code directly). | M | P1 |
| FR-TA-157 | Promo code list shows status, redemptions used / limit, total discount given, and revenue from bookings that used the code; export CSV. | S | P1 |
| FR-TA-158 | Bulk single-use codes (e.g. 50 unique codes for a corporate partner or event). | C | P2 |
| FR-TA-159 | Staff can apply a tenant promo code when creating a booking on behalf of a customer. | S | P1 |
| FR-TA-151 | Membership plans: name, price, cycle (monthly/quarterly/annual), included resource (e.g. assigned desk), included credits (hours of meeting room per cycle), member discount %, access hours, deposit, notice period, visibility (public/private). | S | P2 (data model P1) |
| FR-TA-152 | Assign membership to a customer or company; start date, assigned unit, deposit; auto-generate renewal invoice N days before cycle; payment link sent. | S | P2 |
| FR-TA-153 | Credit ledger per customer/company: grants, consumption by bookings, expiry, manual adjustments with reason. | S | P2 |
| FR-TA-154 | Pause, upgrade, downgrade, cancel membership with notice-period handling. | C | P2 |
| FR-TA-155 | Gift cards / prepaid packs (e.g. 10 day passes). | C | P3 |

### 3.11 Notifications, staff, reports, billing with Cowos

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-170 | Notification settings: enable/disable channels per event (booking confirmed, reminder, cancelled, refund, invoice, request approved, payment link, lead received) for customers and for staff. | M | P1 |
| FR-TA-171 | Customise email templates (subject, intro text) within platform layouts; SMS/WhatsApp use pre-approved platform templates with tenant name variables. | S | P1 |
| FR-TA-172 | Staff management: invite by email/phone, assign role (Manager, Staff) and locations, deactivate; Owner transfer with re-verification. | M | P1 |
| FR-TA-173 | Dashboard: today's bookings, revenue this month (online + offline), occupancy % by product, new leads, pending requests, unpaid invoices. | M | P1 |
| FR-TA-174 | Reports: revenue by product/location/period, occupancy by product/unit and hour heatmap, booking source, coupon usage, lead conversion, member churn; export CSV. | S | P2 |
| FR-TA-175 | Billing page: plan, limits usage (locations/resources/staff), next invoice, payment method, invoices from SSPL (download PDF), messaging wallet balance and usage, "where your fee goes". | M | P1 |
| FR-TA-176 | Upgrade/downgrade plan; switch monthly ↔ annual; cancel subscription with confirmation and export reminder. | M | P1 |
| FR-TA-177 | Support: in-app help widget, create ticket, view tickets, approve/revoke SSPL support access grants. | M | P1 |
| FR-TA-178 | Tenant audit log: who changed what (pricing, policy, publish, refunds, staff) with before/after values. | M | P1 |
| FR-TA-179 | Webhooks and public API keys for tenants (Pro). | C | P3 |

---

## 4. FR-TW · Tenant Website (public)

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TW-001 | The website shall be served as pre-rendered static HTML per published version, with the theme's CSS and minimal JavaScript (contact form, gallery lightbox, map lazy-load). | M | P1 |
| FR-TW-002 | The website shall resolve the tenant by hostname (custom domain or platform subdomain) at the CDN edge. | M | P1 |
| FR-TW-003 | The site shall render all enabled sections in the tenant's branding, and the "Powered by Cowos" badge in the footer on every page, linking to `https://cowos.in/?ref=<slug>` with `rel="noopener"`. | M | P1 |
| FR-TW-004 | Spaces & pricing sections shall show current products and "from" prices; prices update without republishing (fetched at build and refreshed by a lightweight client call, or the site is rebuilt automatically when prices change). | M | P1 |
| FR-TW-005 | CTAs shall deep-link into the booking app with product pre-selected (`/book/p/<product-slug>`) or into the tour flow. | M | P1 |
| FR-TW-006 | The contact form shall submit to the API with spam protection (honeypot + rate limit + invisible challenge), create a lead, and notify the tenant. | M | P1 |
| FR-TW-007 | Each page shall output title, meta description, canonical URL, Open Graph/Twitter tags, and JSON-LD `LocalBusiness` (CoworkingSpace subtype where supported) with address, geo, opening hours, price range, images; FAQ section outputs `FAQPage`. | M | P1 |
| FR-TW-008 | The site shall generate `sitemap.xml` and `robots.txt`; platform subdomains of trial tenants are `noindex` until the first paid activation. | M | P1 |
| FR-TW-009 | Click-to-call and click-to-WhatsApp buttons (floating on mobile, configurable). | M | P1 |
| FR-TW-010 | The site shall load the map only on interaction or when scrolled into view (static map image first). | S | P1 |
| FR-TW-011 | Cookie/consent banner shown only when the tenant enables analytics/marketing pixels; scripts load after consent. | M | P1 |
| FR-TW-012 | Suspended tenant: site replaced by a neutral "temporarily unavailable" page with tenant contact details (FR-PL-062). | M | P1 |
| FR-TW-013 | Custom 404 page in theme. | S | P1 |

---

## 5. FR-BA · Tenant Booking App (customer-facing)

### 5.1 Account

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-BA-001 | Customers shall sign up / log in with mobile number + OTP (SMS, with WhatsApp OTP as alternative channel), or email + OTP / magic link. | M | P1 |
| FR-BA-002 | Browsing and availability shall not require login; login is asked at checkout (guest-to-account at checkout with OTP). | M | P1 |
| FR-BA-003 | Sign-up shall show a privacy notice (tenant as Data Fiduciary, purposes, rights, grievance contact) and capture separate, unticked consent for marketing messages; consent records stored with timestamp, notice version, channel. | M | P1 |
| FR-BA-004 | Profile: name, email, phone, company (join existing via invite or create), GSTIN for invoices, billing address, notification preferences. | M | P1 |
| FR-BA-005 | Customers shall request account deletion and data download from profile (DPDP rights); requests go to the tenant's DPDP queue. | M | P1 |
| FR-BA-006 | Sessions scoped to the tenant host (host-only cookies); a customer of two tenants has two separate accounts. | M | P1 |

### 5.2 Browse and book

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-BA-010 | Home: tenant branding, location selector (if >1), product categories (Day pass, Meeting rooms, Desks & cabins, Event space, Virtual office), featured products. | M | P1 |
| FR-BA-011 | Product page: photos, description, capacity, amenities, prices per unit, add-ons, policies summary, booking mode CTA. | M | P1 |
| FR-BA-012 | Availability: for hourly products, a date picker and time-slot grid showing free/busy per unit; for day products, a calendar showing remaining capacity; for monthly products, start date picker with availability check. | M | P1 |
| FR-BA-013 | Unit selection: "any available" (default) or a specific unit from a list; floor-plan selection when configured (FR-TA-068). | M (list), C (floor plan) | P1 / P2 |
| FR-BA-014 | Duration and quantity: hours, days (date range), months; number of seats for day passes. | M | P1 |
| FR-BA-015 | Price summary before payment: unit price × quantity, add-ons, promo code discount, member discount, tenant convenience fee (if enabled), taxable value, GST (CGST/SGST or IGST), total, cancellation policy summary, and "Pay to <tenant legal name>". | M | P1 |
| FR-BA-016 | Promo code entry with immediate validation and clear error messages; public offers from the tenant shown on the product page. | M | P1 |
| FR-BA-017 | Holding: when the customer proceeds to pay, the slot is held for 10 minutes (countdown shown); the hold is released on expiry or payment failure. | M | P1 |
| FR-BA-018 | Payment via Razorpay Checkout on the tenant's connected account (UPI, cards, netbanking, wallets); success confirmed server-side by signature verification and webhook, never by the client alone. | M | P1 |
| FR-BA-019 | Confirmation screen: booking reference, QR code, details, add-to-calendar (Google link + `.ics`), directions link, invoice download. | M | P1 |
| FR-BA-020 | Request-to-Book: customer submits request with details; sees "Awaiting confirmation"; receives payment link on approval. | M | P1 |
| FR-BA-021 | Enquire / Book a tour: customer picks a tour slot (if configured) or preferred date/time, team size, budget, move-in date; creates a lead; confirmation sent. | M | P1 |
| FR-BA-022 | Pay with member credits for eligible products; show remaining credits; mixed payment (credits + online) when credits are insufficient. | S | P2 |
| FR-BA-023 | Bill to company: customer chooses personal or company billing at checkout when they belong to a company. | M | P1 |
| FR-BA-024 | Virtual office application: choose plan, upload KYC documents (PAN, ID, address proof, company documents), submit for approval; pay after approval. | S | P2 |
| FR-BA-025 | Membership purchase online (choose plan, start date, pay first cycle + deposit). | S | P2 |

### 5.3 Manage bookings

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-BA-030 | My bookings: upcoming, past, cancelled; booking detail with QR, status, invoice, policy. | M | P1 |
| FR-BA-031 | Cancel within policy: show refund amount before confirming; refund to original method or as credit (if offered). | M | P1 |
| FR-BA-032 | Reschedule within policy to an available slot; pay or receive credit for price difference. | S | P1 |
| FR-BA-033 | Invoices and credit notes list with PDF download. | M | P1 |
| FR-BA-034 | Membership page: plan, assigned unit, credits balance and history, renewal date, pay renewal. | S | P2 |
| FR-BA-035 | Company admin: invite/remove team members, view company bookings and invoices. | S | P2 |
| FR-BA-036 | Self check-in by scanning a QR at the reception (location QR). | C | P2 |

### 5.4 Notifications and app shell

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-BA-040 | Notifications: booking confirmed, request received/approved/declined, payment link, reminder (24h and 1h before for meeting rooms; evening before for day passes), cancelled, refund initiated/processed, invoice issued, membership renewal due. Channels: email always; SMS and/or WhatsApp per tenant settings and customer preference. | M | P1 |
| FR-BA-041 | In-app notification list. | S | P2 |
| FR-BA-042 | PWA: installable, offline shell, web push notifications. | S | P2 |
| FR-BA-043 | "Powered by Cowos" badge in the booking app footer and on the confirmation page. | M | P1 |
| FR-BA-044 | Booking app shall show the tenant's support phone/WhatsApp/email on every page footer and on error screens. | M | P1 |

---

## 6. FR-EM · Existing-website integration

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-EM-001 | Booking-only tenants shall get a booking app on `<slug>.cowos.site/book` and can map `book.<theirdomain>` via CNAME with automatic SSL. | M | P1 |
| FR-EM-002 | The integration page shall generate copy-paste snippets: button HTML linking to the booking app (styled with tenant colours), direct links per product, and a QR code poster (PDF) for the reception. | M | P1 |
| FR-EM-003 | Booking app shall support a "return to website" link configured by the tenant. | M | P1 |
| FR-EM-004 | Script widget: one `<script>` tag that adds a floating "Book a space" button and/or inline buttons (`data-cowos-product`), opening the booking flow in a modal on desktop and full-page on mobile; login and payment steps open in a top-level window to avoid third-party cookie and UPI-intent issues. | S | P2 |
| FR-EM-005 | Widget events (`cowos:booking_completed`) exposed to the host page for their analytics. | C | P2 |
| FR-EM-006 | Allowed embedding origins configured per tenant (CSP `frame-ancestors`). | S | P2 |
| FR-EM-007 | WordPress plugin wrapping the widget (shortcode + block). | C | P3 |

---

## 7. FR-SA · Cowos Super Admin Portal

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-SA-001 | Login via SSO (Google Workspace or Entra ID) with MFA; access restricted to SSPL IP ranges / VPN. | M | P1 |
| FR-SA-002 | Tenant list: search, filters (status: trial, active, past due, restricted, suspended, cancelled; plan; city; created date), key metrics per tenant (bookings 30d, last login, publish status). | M | P1 |
| FR-SA-003 | Tenant detail: profile, KYC status, plan and billing history, domains and SSL status, published versions, staff users, support grants, notes, audit log. | M | P1 |
| FR-SA-010 | First-publish review queue: automated check results (GSTIN/PAN verification, phone/email verified, gateway connected, content scan flags), preview link, approve / reject with reason / request changes; SLA timer. | M | P1 |
| FR-SA-011 | Unpublish or suspend a tenant with reason (content violation, fraud, legal request); tenant notified; action audited. | M | P1 |
| FR-SA-020 | Plans management: create/edit plans, prices, limits, feature flags; versioned so existing subscribers keep their price until changed explicitly. | M | P1 |
| FR-SA-021 | Platform subscriptions: view, extend trial, apply discount/credit, change plan, mark paid (offline), pause, cancel. | M | P1 |
| FR-SA-022 | Platform invoices list, download, resend, credit notes; GST summary export for SSPL's returns. | M | P1 |
| FR-SA-023 | Dunning dashboard: past due tenants, stage, last reminder, next action. | M | P1 |
| FR-SA-024 | Messaging rates management (per channel/category) and tenant wallet adjustments. | S | P1 |
| FR-SA-030 | Support tools: request support grant, view tenant admin as tenant (read-only by default) within a grant, with a persistent banner and full audit. | M | P1 |
| FR-SA-031 | Ticketing: integrate with a help desk (e.g. Freshdesk/Zoho Desk) or a built-in simple ticket list. | S | P1 |
| FR-SA-040 | Theme catalog: register theme versions, set availability per plan, mark deprecated; preview with sample data. | M | P1 |
| FR-SA-041 | Platform settings: notification providers and credentials, SMS DLT templates and IDs, WhatsApp templates, reserved slugs, blocked words list, feature flags per tenant. | M | P1 |
| FR-SA-050 | Monitoring: platform health (API error rate, latency, queue depth, failed jobs, webhook failures, DNS/SSL issues by tenant), links to CloudWatch/Grafana dashboards. | M | P1 |
| FR-SA-051 | Business dashboard: sign-ups, trials, conversions, MRR, churn, referrals by source/tenant. | S | P1 |
| FR-SA-052 | Tenant deletion workflow with two-person approval and retention-aware purge. | M | P1 |
| FR-SA-053 | Global announcements to tenants (admin banner + email). | S | P2 |
| FR-SA-054 | Dedicated-cell provisioning for hybrid tenants. | C | P3 |

---

## 8. FR-PL · Platform services (cross-cutting)

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-PL-001 | Tenant resolution: every request resolves a tenant from (a) hostname for public sites/booking app/public API, or (b) the authenticated user's tenant claim for admin APIs; requests with no resolvable tenant are rejected. | M | P1 |
| FR-PL-002 | Data isolation: every tenant-owned table has `tenant_id`; application queries filtered by tenant; PostgreSQL RLS policies enforce the same at the database. | M | P1 |
| FR-PL-010 | Site build service: on publish, render the site version to static files, upload to S3 under a versioned prefix, atomically switch the tenant's live pointer, invalidate CDN paths. | M | P1 |
| FR-PL-011 | Automatic rebuild when data shown on the website changes (inventory names, prices, hours) without creating a new content version. | S | P1 |
| FR-PL-020 | Domain service: add domain, ownership verification via TXT, DNS polling, certificate issuance and renewal, health checks, removal. | M | P1 |
| FR-PL-030 | Media pipeline: virus scan, strip EXIF location data, generate responsive variants (WebP/AVIF + JPEG fallback; widths 320/640/1024/1600/2400), store in S3, serve via CDN; per-tenant storage quota. | M | P1 |
| FR-PL-040 | Notification service: templated email (SES), SMS (DLT-compliant CPaaS), WhatsApp (BSP/Cloud API); per-message status tracking, retries, fallback channel (WhatsApp fails → SMS), wallet debit for pass-through costs. | M | P1 |
| FR-PL-041 | Quiet hours for non-urgent messages (no promotional/reminder messages 9 PM–8 AM IST except booking within that window). | S | P1 |
| FR-PL-050 | Payment service: gateway adapter interface; order creation on tenant account; signature verification; webhook ingestion (idempotent, signature-verified, stored raw); reconciliation job comparing gateway payments to bookings daily. | M | P1 |
| FR-PL-051 | Booking hold expiry job: release holds after 10 minutes; late payment success after expiry triggers auto-refund if the slot was taken, or confirmation if still free. | M | P1 |
| FR-PL-052 | Invoice service: numbering with gap-free sequences per series per FY (transactional), PDF generation, storage, regeneration of the same PDF on demand. | M | P1 |
| FR-PL-060 | Platform billing engine: subscriptions, renewals, proration, invoices, dunning schedule, state transitions (BRD §7.3). | M | P1 |
| FR-PL-061 | Plan limit enforcement and feature flags evaluated server-side. | M | P1 |
| FR-PL-062 | Tenant state machine (Trial → Active → PastDue → Restricted → Suspended → PendingDeletion → Deleted; Cancelled path) controls site serving, booking acceptance and admin capabilities. | M | P1 |
| FR-PL-070 | Audit log service: append-only, tenant-scoped and platform-scoped events with actor, impersonator, IP, before/after. | M | P1 |
| FR-PL-071 | Data export service: async job builds a zip (CSV/JSON + media) and emails a time-limited link to the Owner. | M | P1 |
| FR-PL-080 | Calendar: generate `.ics` per booking; Google Calendar add link. | M | P1 |
| FR-PL-081 | One-way push of meeting-room bookings to a tenant's Google/Outlook resource calendar. | C | P3 |
| FR-PL-090 | Search-engine-friendly redirects and canonical handling across platform and custom domains. | M | P1 |
