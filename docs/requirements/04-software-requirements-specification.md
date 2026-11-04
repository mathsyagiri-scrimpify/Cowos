# 04 · Software Requirements Specification (SRS): Functional Requirements

> Cowos requirements pack · v0.3 draft · Non-functional requirements are in [07](07-non-functional-requirements.md)

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

**Phase:** P1 (MVP launch), P2, P3 (defined in [12](12-release-plan.md)).

**Revenue model:** Cowos has no plans, paid tiers, subscriptions or commission. Every tenant gets every feature. Tenants pay only for metered consumption: server request units and server storage units, invoiced monthly in arrears, and notification units (OTP SMS, other SMS, WhatsApp, email), debited from a prepaid wallet. Each meter is priced in telescopic slabs that SSPL sets in the rate card (D1, Pricing Rules #3 to #6).

---

## 2. FR-MW · Cowos Marketing Web App

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-MW-001 | The site shall present product pages: home, features (website builder, booking app, payments, invoices, CRM), pricing, demo sites, about SSPL, contact, legal (terms, privacy, DPA, refund policy). | M | P1 |
| FR-MW-002 | The pricing page shall lead with the 0% commission promise and state that tenants pay only for the server usage and notifications they use, priced in slabs. It lists what is included (website, booking app, hosting, SSL, backups, updates) and shows the current rate card (FR-MW-003). There are no plan cards, plan prices or limits (D1, 28A, Pricing Rule #2). | M | P1 |
| FR-MW-003 | The pricing page shall show the rate card in force: one slab table per meter (server request units, server storage units, OTP SMS, other SMS, WhatsApp, email), each slab with its unit range and price per unit (a slab may be ₹0), the unit definitions (1 request unit = 1,000 page views and API calls; 1 storage unit = 1 GB-month; 1 notification unit = 1 message), a note that slabs apply telescopically and reset each calendar month (IST), prices excluding GST, and the version's effective date (Pricing Rules #3, #4, 28A). Above the folded tables the page leads with 2-3 sample monthly bills for typical space profiles set by SSPL, computed by the SlabCalculator (DR2-2A). | M | P1 |
| FR-MW-004 | The pricing page shall include a commission savings calculator with editable inputs (bookings/month, average booking value, aggregator commission %). It serves tenant recruitment at launch (CEO review). | S | P1 |
| FR-MW-005 | The site shall offer sign-up with name, work email, mobile number, business name, city, slug (FR-MW-012), and a password or Google sign-in; email and mobile are verified by OTP before the 14-day trial starts. Sign-up creates the tenant in Trial (FR-PL-062), adds an edge_sync row (FR-PL-010) and opens the onboarding wizard (FR-TA-001) (Pricing Rules #7, #9). | M | P1 |
| FR-MW-006 | The site shall capture referral codes (`?ref=`) and UTM parameters, persist them for 30 days in a first-party cookie, and attach them to the tenant at sign-up (Pricing Rule #9). | M | P1 |
| FR-MW-007 | The site shall link to one live demo tenant site per launched theme (2 at launch), with sample inventory and test-mode payments (CEO-S3). | S | P1 |
| FR-MW-008 | The site shall provide a "Book a demo" request form (name, phone, email, space, city, message). Protection as FR-TW-006: honeypot, 5 requests per IP per hour, invisible challenge. Validation: required fields, phone in +91 E.164, message up to 2,000 characters. Each request is saved to the Super Admin "Demo requests" list (FR-SA-057) before sales is notified through the email outbox with retry; a failed email leaves the request visible in Super Admin. A repeat from the same phone within 24 hours updates the existing request and sends one notification. The form shows a privacy notice with purpose, 12-month retention and grievance contact. Input is stored as plain text and rendered escaped. Success message: "We'll contact you within 2 working days" (CEO-A2). | M | P1 |
| FR-MW-009 | The site shall be pre-rendered for SEO, with structured data (Organization, SoftwareApplication, FAQPage) and a sitemap. | M | P1 |
| FR-MW-010 | The site shall host a help centre (searchable articles: DNS guides per registrar, Razorpay connection, GST setup). | S | P1 |
| FR-MW-011 | The site shall host a blog / SEO landing pages ("coworking software India", city pages). | C | P2 |
| FR-MW-012 | The site shall show a domain availability check during sign-up for the platform subdomain slug (Pricing Rule #9). | M | P1 |
| FR-MW-013 | The pricing page shall be rebuilt from the published rate card (FR-SA-020) when SSPL publishes a version and when a version takes effect on the 1st. The page never shows any tenant's usage, the tenant count or SSPL's AWS bill. If a rebuild fails, the page keeps the last published rate card (Pricing Rule #4, 28A). | M | P1 |

---

## 3. FR-TA · Tenant Admin Portal

### 3.1 Account, onboarding wizard

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-001 | For a self-serve tenant, the first login shall start an onboarding wizard in this order: (1) Goal: full website + booking app, or booking app only; (2) Business details and GSTIN; (3) Location and hours; (4) Theme, previewed with the owner's logo, uploaded photos and demo spaces; (5) Branding; (6) Inventory (at least one product); (7) Policies; (8) Payments, with MFA set up inline; (9) Preview; (10) Publish (4A, Pricing Rule #9). Every step's data stays editable afterwards in the matching settings screen. | M | P1 |
| FR-TA-002 | The Tenant Admin home shall show a setup checklist of what remains before publishing, with links to Razorpay KYC and OAuth, GSTIN/PAN, DNS, ToS and DPA. Each item is re-checked on load: a Razorpay disconnect flips the item back to not done, and a timeout shows "Couldn't check". An Owner lands on the checklist before first publish and on the dashboard after it, where any leftovers become one card (31A). Each wizard step (FR-TA-001) auto-saves, shows completion status and allows skipping non-required steps. | M | P1 |
| FR-TA-003 | The business details wizard step and settings shall validate GSTIN format and checksum and derive state and PAN from the GSTIN; the tenant can override the trade name. The wizard also fetches the legal name from the GST public search provider; this lookup is optional and the tenant can confirm or edit the result. | M | P1 |
| FR-TA-004 | The location step and location settings (FR-TA-060) shall use Google Places autocomplete to fill address and coordinates, show a draggable map pin, and accept directions/landmark text. | M | P1 |
| FR-TA-005 | The platform subdomain slug (3–30 chars, lowercase letters, digits, hyphens; reserved words blocked) shall be chosen by the Owner at sign-up (FR-MW-005, FR-MW-012) and shown in settings (Pricing Rule #9). | M | P1 |
| FR-TA-006 | The portal shall show a "Booking app only" path that skips theme/website content steps and leads to the integration step (FR-EM) (4A). | M | P1 |
| FR-TA-007 | The Owner shall be able to enable MFA (TOTP) and shall be required to enable it before connecting a payment gateway or a custom domain. | M | P1 |
| FR-TA-008 | The portal shall offer demo inventory (sample hot desk, meeting room, cabin) labelled "Sample", each removable in one click, so preview isn't empty. Products carry an `is_sample` flag; pre-publish checks block publishing while any sample item remains (4A). | M | P1 |
| FR-TA-009 | Removed in v0.3 (pilot programme removed: no SSPL-created tenants, pre-handover setup grant or first-login handoff; Owners sign up self-serve, Pricing Rule #9). | – | – |

### 3.2 Site builder (themes, branding, content)

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-020 | The portal shall offer four named theme directions, each with its own type pair, hero style (FR-TA-034) and density: Loft (warehouse/industrial, bold condensed display), Metro (tech-park corporate, crisp sans, dense), Courtyard (calm, plant-filled, generous spacing, warm humanist type) and Campus (startup/community, energetic, photo-led). Cream-serif, black-neon and broadsheet-hairline looks are ruled out (24A). Launch ships 2 themes (Loft and Metro, or the 2 that validation with early tenant owners favours). The other 2 are built after validating the directions with 3 early tenant owners and released about 6 weeks after launch; each passes Lighthouse, axe and the screen-reader pass before it appears in the catalog (CEO-S3). Each theme has a live thumbnail, a full preview using the tenant's own content, and a booking shell (FR-BA-045). | M | P1 (themes 3–4 about launch + 6 weeks) |
| FR-TA-021 | Switching themes shall keep all content (sections and fields are theme-independent); only layout and styling change. | M | P1 |
| FR-TA-022 | Branding editor: logo upload (SVG/PNG/JPG ≤5 MB), optional dark-mode logo, favicon (auto-generated if not provided), primary/secondary/accent colours, font pair, button shape (rounded/pill/square), theme mode (light/dark/auto). **Colour roles (23A):** from each owner colour the system derives roles in OKLCH for light and dark mode: primary, primary-hover, primary-pressed, tint (backgrounds), border, link, focus ring, text-on-primary. Every role is checked in both modes (4.5:1 for text, 3:1 for UI components); a failing role has its lightness adjusted automatically and the owner sees a before/after. Selection colour, caret, scrollbars and focus rings use the derived palette. **Fonts (25A):** the owner picks one of the theme's 2 named font pairs (no Inter, Roboto, Arial or system font as the display face). Every pair has a matching Noto or Anek family for Telugu and Devanagari, loaded only when the page contains that script. Fonts are self-hosted WOFF2, Latin subset by default, `font-display: swap`, and count against the 300 KB page budget. | M | P1 |
| FR-TA-023 | Section editor: enable/disable, reorder within the theme's allowed order, and edit each section through a form: Header (logo, nav auto-generated, CTA label + target: Book / Tour / WhatsApp / Call), Hero (headline, sub-headline, background image or short video, primary and secondary CTA), About, Amenities (from amenity list), Spaces & Pricing (auto-built from inventory, choose which products to show and whether to show prices), Gallery (up to 40 images, captions, alt text required; the site shows the owner's first 6–9 photos in a grid in the owner's order, with "See all <n> photos" opening a keyboard- and swipe-friendly lightbox with alt text), Testimonials (name, role, company, quote, photo, rating; up to 3 shown as static quotes), FAQ (Q&A pairs; default coworking FAQ set offered), Location (map, address, directions, nearby transit), Contact form (fields configurable: name, phone, email, company, interest, message), CTA band, Footer (text, links, social links, contact). No theme uses autoplay carousels (20A). A section whose content is empty hides itself on the website (13A). Section rules: FR-TA-033 to FR-TA-035. Each section row also has Move up / Move down buttons and Alt+Arrow keyboard moves, announced in a live region (DR2-10A). | M | P1 |
| FR-TA-024 | Every image field shall require alt text (prefilled with a suggestion the tenant can edit) and run through the media pipeline (FR-PL-030). | M | P1 |
| FR-TA-025 | The tenant shall be able to create additional simple pages (title, slug, rich text, images) and add them to the nav. There is no plan limit (D1). | S | P1 |
| FR-TA-026 | SEO settings per page: title, meta description, social share image; site-level: Google Analytics 4 ID, Google Tag Manager ID, Meta Pixel ID, Google Search Console verification token. | M | P1 |
| FR-TA-027 | The builder shall show a live preview pane updated within 1 second of an edit, with device toggles (desktop 1440, tablet 768, mobile 390) and a "open preview in new tab" shareable link (token-protected, expires in 7 days). | M | P1 |
| FR-TA-028 | The preview shall include the booking app in the same branding with real inventory and pricing, and payment in test mode. | M | P1 |
| FR-TA-029 | AI copy assistant: generate hero, about, FAQ and meta descriptions from business details; tenant reviews before saving. | C | P2 |
| FR-TA-030 | Free HTML embed section, sandboxed and reviewed against a content security policy. Available to every tenant; there are no paid tiers (D1). | C | P3 |
| FR-TA-031 | Blog/news section with posts. | C | P2 |
| FR-TA-032 | Multi-language content (Telugu, Hindi). | W | P3 |
| FR-TA-033 | First-screen contract (all themes): the first viewport shows the logo, an H1 with brand and area ("Coworking in Madhapur"), one primary CTA (Book or Tour; required by the pre-publish checks, FR-TA-041) and one proof line (price from, or rating). On mobile, one floating Book bar with a WhatsApp icon, with no stacked floating buttons. The offer banner (FR-TA-156) appears only after the cookie choice (3A). | M | P1 |
| FR-TA-034 | Hero over owner photos: each theme declares its hero as either text over image with an enforced solid scrim (opacity computed so headline text reaches 4.5:1 on the image's sampled luminance; no gradients) or a split layout with text beside the photo. Owners set a focal point for cropping. Video heroes get a pause control and a poster image (18A). | M | P1 |
| FR-TA-035 | Section jobs and word limits: each section form shows its one job and a live word counter. Hero H1 ≤12 words (where + for whom), sub-line ≤20; About ≤60 words (what's different here); Amenities as short labels; CTA band ≤8 words with one action; FAQ answers ≤40 words. The AI copy assistant (FR-TA-029) follows the same limits and a banned-phrase list ("vibrant", "unlock", "all-in-one", "welcome to") (21A). | M | P1 |
| FR-TA-036 | Builder layout (desktop): left section list (show/hide, drag to reorder, locked positions shown with a lock icon), centre live preview with device toggles (FR-TA-027), right form for the selected section; clicking a section in the preview selects it. A persistent "Draft saved · Not live" chip sits beside Publish. On tablet the form becomes a slide-over (7A). | M | P1 |

### 3.3 Publishing and versions

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TA-040 | The portal shall hold one working draft per tenant site; edits never affect the live site until published. | M | P1 |
| FR-TA-041 | Publish shall run pre-publish checks, shown as a pass/fail list with Fix links: required fields present, one primary CTA on the first screen (FR-TA-033), badge present, all images have alt text, policies present, at least one bookable or enquirable product, and no sample items (FR-TA-008). Instant Book products without a live Razorpay connection show to customers as "Request" and do not block publish (33A). Tenant state: an Active tenant publishes normally; a Trial tenant publishes to the platform subdomain only (Pricing Rule #7); while the tenant is Suspended, publish and rollback are rejected with "Your site is suspended. Contact SSPL support." and the reason (CEO-A1). There is no plan or subscription check (D1). During the build the owner sees "Building… usually under 2 min"; if the render fails, the owner can retry and the previous version stays live (11A). | M | P1 |
| FR-TA-042 | The first publish shall go to SSPL approval (FR-SA-010); the tenant sees "In review · typical under 4 working hours · you can keep editing" with the share-preview link, and gets notified on decision. Changes requested shows the reviewer's notes with Fix links. Live shows the link and the share kit. Later publishes go live immediately (11A). | M | P1 |
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
| FR-TA-091 | Today view for front desk: first a "Now / next" timeline of today's arrivals with a Check-in button per row; always visible, Scan QR and "Walk-in day pass + cash" (one screen, about 10 seconds); below, an exceptions strip (requests with SLA countdown, unpaid bookings). Designed for 390px phones first; two columns on desktop. Visitor count is on the dashboard (FR-TA-173) (6A). | M | P1 |
| FR-TA-092 | Staff shall create a booking for a customer (search or quick-create customer), choose payment: send payment link, record offline payment, use member credits, or mark complimentary (with reason, Owner/Manager only). | M | P1 |
| FR-TA-093 | Approve / decline Request-to-Book requests with optional message; approval sends payment link; auto-decline after the configured SLA. | M | P1 |
| FR-TA-094 | Cancel a booking: system calculates the refund per policy; Owner/Manager may override amount with a reason; refund executes via gateway or as credit; cancellation notifications sent. When a gateway refund fails, the Today view pins the failure (DR2-12A). "Refunded another way" (UPI/bank reference, audited) is enabled only when the gateway refund is in a final failed state with no retry pending, and recording it disables Retry; refund states (Failed, Retrying, Refunded, RefundedManually) change only through one guarded transition (R24). | M | P1 |
| FR-TA-095 | Reschedule a booking to an available slot; price difference handled (charge link or credit). | S | P1 |
| FR-TA-096 | Check-in by scanning the QR on the customer's desk pass (FR-BA-037) with the device camera, or by tapping check-in; the QR holds a short-lived check-in code validated against the tenant at scan (R10). Mark no-show after configurable grace (default 30 min). | S | P1 |
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
| FR-TA-130 | Connect Razorpay via Partner OAuth; show connected account name/ID, mode (test/live), and connection health; disconnect with warning. While KYC is pending, the payments screen shows "Razorpay: waiting for approval (usually 1–5 days)" with a daily status check; the owner can continue and publish with Book a tour, Request to Book and offline payment. On activation the owner gets a one-tap "Turn on instant booking" (33A). | M | P1 |
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
| FR-TA-150 | Promo codes: the tenant (Owner; Manager for assigned locations) shall create, edit, pause and archive promo codes with code, type (% / flat), value, max discount, min order, validity dates, usage limits (total/per customer), applicable products and locations, first-booking-only. Available to every tenant (D1). | M | P1 |
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
| FR-TA-172 | Staff management: invite by email/phone, assign role (Manager, Staff) and locations, deactivate; Owner transfer with re-verification. The Owner creates the account at sign-up (FR-MW-005); SSPL does not invite Owners (Pricing Rule #9). | M | P1 |
| FR-TA-173 | Dashboard: today's bookings, revenue this month (online + offline), occupancy % by product, new leads, pending requests, unpaid invoices, visitor count (6A). On the tenant's first paid online booking, a one-time dismissible card (and WhatsApp message) reads "Your first online booking: ₹<amount> straight to your bank. Cowos took ₹0." with a one-tap share of the booking link; `Tenant.first_online_booking_at` is set atomically with the booking confirmation so it sends once (14A). | M | P1 |
| FR-TA-174 | Reports: revenue by product/location/period, occupancy by product/unit and hour heatmap, booking source, coupon usage, lead conversion, member churn; export CSV. | S | P2 |
| FR-TA-175 | Billing page (5A): (1) latest server invoice: month, issue date (the 1st), due date (10 days later), amount with GST, status and Pay action, with one line per server meter (request units, storage units) showing units used and the slab breakdown (units and price per slab); (2) "This month so far": units per server meter with the slab breakdown and "about ₹<amount> + GST · estimate, final on the 1st", updated daily from the stored estimate (FR-PL-064, 15A); (3) "0% commission: ₹0 on ₹<bookings> of bookings"; (4) notification wallet balance with a link to the wallet page (FR-TA-180); (5) server invoices from SSPL (download PDF), each with the daily units per meter behind it (FR-PL-068). It never shows another tenant's usage. Each state has a one-line reason and one action (12A): Trial ("Trial ends <date>. Server usage isn't billed during the trial · Add a payment method", Pricing Rule #7), invoice due, PastDue, wallet below zero ("Customers now get email-only notifications · Top up"), Restricted ("Online booking paused until <invoice> is paid · Pay now"). An admin-wide banner appears only when the owner must act. There is no plan, limit or upgrade UI (D1, Pricing Rule #5). Page order: status line ("Nothing due" or "₹X due by <date> · Pay"), this month's total (server estimate + wallet spend, + GST), per-meter detail, wallet, commission line, invoice history (DR2-1A). Request meters show raw counts split into page views and booking-app activity, then units; Indian digit grouping; money to paise (DR2-6A). States per Design Review 2 (DR2-3A); below 600px slab lines become stacked rows (DR2-9A). | M | P1 |
| FR-TA-176 | Payment method: the Owner adds or changes how server invoices are paid: Razorpay payment link, or optional UPI Autopay / e-mandate with a ₹5,000 per-tenant maximum and the RBI pre-debit notice. Adding a payment method makes a Trial tenant Active (Pricing Rule #7). Close account with confirmation and export reminder. There is no plan upgrade, downgrade or monthly/annual switch (D1, 5A, Pricing Rule #5). | M | P1 |
| FR-TA-177 | Support: in-app help widget, create ticket, view tickets, approve/revoke SSPL support access grants. Every SSPL support grant needs the Owner's approval (FR-SA-030, Pricing Rule #9). | M | P1 |
| FR-TA-178 | Tenant audit log: who changed what (pricing, policy, publish, refunds, staff) with before/after values. | M | P1 |
| FR-TA-179 | Webhooks and public API keys for tenants. Available to every tenant; there are no paid tiers (D1). | C | P3 |
| FR-TA-180 | Notification wallet page: balance; top-up (minimum ₹500); per channel (OTP SMS, other SMS, WhatsApp, email) the month-to-date count, the current slab and the price per message in that slab, from the rate card in force; the wallet transaction list (date, channel, count, slab, price); and a warning when the balance is at or below zero ("Customers now get email-only notifications · Top up"). From day one of the trial, notifications need wallet balance. Counts reset at the start of each calendar month (IST) (Pricing Rules #4, #6, #7, R4). Shows "about N days left at this month's rate" and each channel's next price step; alerts the Owner (platform message, not charged) when runway falls below 5 days, once per top-up cycle (DR2-4A). Top-up sheet: "You pay ₹<amount + GST> · You get ₹<amount> of messages", GST invoice per top-up, subject to CA confirmation (DR2-11A). | M | P1 |
| FR-TA-181 | Usage alert: the Owner can set a monthly server amount; when the month-to-date server estimate (FR-PL-064) passes 80% and then 100% of it, the Owner gets one email and one WhatsApp message each (platform messages, not charged to the wallet) and a banner in Tenant Admin. With no amount set, one alert is sent when the estimate passes last month's server invoice by 50%. Alerts never block the website or bookings (C2-A3). The amount is server usage only, before GST; the setting suggests this month's estimate rounded up to ₹100; month 1 shows "We'll start alerting you after your first bill"; the banner links to "This month so far" and can be dismissed for the month (DR2-5A). | M | P1 |

---

## 4. FR-TW · Tenant Website (public)

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-TW-001 | The website shall be served as pre-rendered static HTML per published version, with the theme's CSS and minimal JavaScript (contact form, gallery lightbox, map lazy-load). | M | P1 |
| FR-TW-002 | The website shall resolve the tenant by hostname (custom domain or platform subdomain) at the CDN edge. | M | P1 |
| FR-TW-003 | The site shall render all enabled sections in the tenant's branding, and the "Powered by Cowos" badge as the last line of the footer on every page, linking to `https://cowos.in/?ref=<slug>` with `rel="noopener"`. The badge is a text link with a 12px mark in the theme's footer text colour at ≥4.5:1 (light and dark versions), with a minimum 44px tap height, rendered by the platform so themes and custom CSS can't hide it (30A). | M | P1 |
| FR-TW-004 | Spaces & pricing sections shall show current products and "from" prices; prices update without republishing (fetched at build and refreshed by a lightweight client call, or the site is rebuilt automatically when prices change). A small caption "Prices confirmed at checkout" sits under the "from" prices (DR2-14A). | M | P1 |
| FR-TW-005 | CTAs shall deep-link into the booking app with product pre-selected (`/book/p/<product-slug>`) or into the tour flow. | M | P1 |
| FR-TW-006 | The contact form shall submit to the API with spam protection (honeypot + rate limit + invisible challenge), create a lead, and notify the tenant. | M | P1 |
| FR-TW-007 | Each page shall output title, meta description, canonical URL, Open Graph/Twitter tags, and JSON-LD `LocalBusiness` (CoworkingSpace subtype where supported) with address, geo, opening hours, price range, images; FAQ section outputs `FAQPage`. | M | P1 |
| FR-TW-008 | The site shall generate `sitemap.xml` and `robots.txt`; platform subdomains of Trial tenants are `noindex` until the tenant becomes Active (Pricing Rule #7). | M | P1 |
| FR-TW-009 | Click-to-call and click-to-WhatsApp buttons (configurable). On mobile, one floating Book bar with a WhatsApp icon, with no stacked floating buttons (3A). | M | P1 |
| FR-TW-010 | The site shall load the map only on interaction or when scrolled into view (static map image first). | S | P1 |
| FR-TW-011 | Cookie/consent banner shown only when the tenant enables analytics/marketing pixels; scripts load after consent. | M | P1 |
| FR-TW-012 | Suspended tenant: the site is replaced by a page in the tenant's theme with name, address, phone, WhatsApp, "Online booking is temporarily unavailable" and the badge. Restricted tenant: every Book CTA switches to "Call / WhatsApp to book", with no billing mention to visitors. Both switches go through the data-refresh render in FR-PL-012 (10A, CEO-A1, R11). | M | P1 |
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
| FR-BA-010 | Home: tenant branding; location name (a selector if more than one location); then "Today: <n> seats left · Day pass ₹<price> · Book" (or the tenant's top product when there is no day pass); then meeting rooms, each with its next free time; then other categories as a plain list. No category tiles or card grid. When every product is Enquire, "Book a tour" is the first row (19A). | M | P1 |
| FR-BA-011 | Product page: swipeable photo strip; title, capacity and "from ₹600/hour + GST" on one line; description; amenities as one row of labelled icons; add-ons; one-line policy summary; sticky "Check times" bar (or the booking mode CTA for Request and Enquire products). No card stacks (32A). | M | P1 |
| FR-BA-012 | Availability for hourly products: date strip, then duration chips (1h / 2h / half day / full day), then the list of available start times as 44px rows ("10:00 – 12:00 · ₹1,416"). Availability is shown as text ("2 left", "Sold out"), never by colour alone. Start times are a labelled radio group (each option reads "10:00 to 12:00, ₹1,416, available"), with arrow keys to move and Enter or Space to select (1A, 27A). Loading shows skeleton rows; a date with no times shows "Fully booked on <day> · next free: <day time>" with one tap; a closed day says why ("Closed for Diwali"); a load failure shows "<Tenant> couldn't load times · Try again" (9A, 13A, 29A). For day products, a calendar showing remaining capacity; for monthly products, a start date picker with availability check. | M | P1 |
| FR-BA-013 | Unit selection: "any room" / "any available" is the default, and start times merge across units; choosing a specific unit from a list is an optional step (1A). Floor-plan selection when configured (FR-TA-068). | M (list), C (floor plan) | P1 / P2 |
| FR-BA-014 | Duration and quantity: hours, days (date range), months; number of seats for day passes. | M | P1 |
| FR-BA-015 | Checkout step 1, summary: slot, total, "Pay to <tenant legal name>", GST collapsed to one tappable line ("incl. GST ₹216 ▸") that expands to the full breakdown (unit price × quantity, add-ons, promo code discount, member discount, tenant convenience fee if enabled, taxable value, CGST/SGST or IGST), and a one-line cancellation policy. Step 2 is phone OTP (FR-BA-001/002) with the privacy notice, unticked marketing consent (FR-BA-003) and bill-to-company (FR-BA-023). Step 3 is the hold and payment (FR-BA-017). On mobile a sticky bottom bar shows the total and "Pay ₹…" (2A). | M | P1 |
| FR-BA-016 | Promo code entry with immediate validation and clear error messages; public offers from the tenant shown on the product page. | M | P1 |
| FR-BA-017 | Holding: the 10-minute hold starts only after OTP verification (checkout step 3), never before (2A). Hold creation re-quotes the price and shows any difference before Razorpay opens. If the slot was taken during OTP, the next free times show inline. The countdown is visible every second; an `aria-live` region announces only at 5 minutes, 1 minute and expiry (27A). The hold is released on expiry or payment failure. If the hold expired and the slot is still free, the customer gets a one-tap retry (9A). | M | P1 |
| FR-BA-018 | Payment via Razorpay Checkout on the tenant's connected account (UPI, cards, netbanking, wallets); success confirmed server-side by signature verification and webhook, never by the client alone. After payment the customer sees "Confirming your payment…" with progress, polled for up to 2 minutes, then "We've got your payment, confirmation is on its way by WhatsApp"; never an error while the webhook is pending. If the slot was lost after payment: "Full refund of ₹X started, 5–7 working days" plus the next free times. If the gateway is down, the app offers Request to Book. Test mode and preview carry an unmistakable ribbon (9A). | M | P1 |
| FR-BA-019 | Confirmation screen: booking reference, desk pass link and QR (FR-BA-037), details, add-to-calendar (Google link + `.ics`), directions link, invoice download (an "Invoice being prepared" placeholder until the PDF is ready, 9A). | M | P1 |
| FR-BA-020 | Request-to-Book: customer submits request with details; sees "Awaiting confirmation"; receives payment link on approval. | M | P1 |
| FR-BA-021 | Enquire / Book a tour: customer picks a tour slot (if configured) or preferred date/time, team size, budget, move-in date; creates a lead; confirmation sent. | M | P1 |
| FR-BA-022 | Pay with member credits for eligible products; show remaining credits; mixed payment (credits + online) when credits are insufficient. | S | P2 |
| FR-BA-023 | Bill to company: customer chooses personal or company billing at checkout when they belong to a company. | M | P1 |
| FR-BA-024 | Virtual office application: choose plan, upload KYC documents (PAN, ID, address proof, company documents), submit for approval; pay after approval. | S | P2 |
| FR-BA-025 | Membership purchase online (choose plan, start date, pay first cycle + deposit). | S | P2 |
| FR-BA-026 | Resumable booking URL: `/book/checkout/{bookingRef}` and its status endpoint land a UPI return or a reload on the right state (9A). Both require the customer's session; without one, the page asks for OTP and then shows the state; another customer's session gets 404 (R16). | M | P1 |

### 5.3 Manage bookings

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-BA-030 | My bookings: a "Next up" row on top (date, time, room, Show pass), then Upcoming, Past and Cancelled as plain rows with status text, no card stacks. Booking detail shows the desk pass, status, invoice and policy; cancel and reschedule live in the booking detail (32A). | M | P1 |
| FR-BA-031 | Cancel within policy: show refund amount before confirming; refund to original method or as credit (if offered). | M | P1 |
| FR-BA-032 | Reschedule within policy to an available slot; pay or receive credit for price difference. | S | P1 |
| FR-BA-033 | Invoices and credit notes list with PDF download. | M | P1 |
| FR-BA-034 | Membership page: plan, assigned unit, credits balance and history, renewal date, pay renewal. | S | P2 |
| FR-BA-035 | Company admin: invite/remove team members, view company bookings and invoices. | S | P2 |
| FR-BA-036 | Self check-in by scanning a QR at the reception (location QR). | C | P2 |
| FR-BA-037 | Desk pass: confirmation messages link to a full-screen pass with a large QR, first name and last initial, date/time, room or seat, and tenant logo. The link works without login: its token is opaque with no personal data in the URL, HMAC-bound to the booking id and the purpose "pass", expires at booking end, and is revoked on cancel or reschedule. The QR holds a separate short-lived check-in code validated against the tenant at scan. Pass responses send `noindex`, `Referrer-Policy: no-referrer` and `Cache-Control: private`. A service worker scoped to the pass route only caches the opened pass page and QR image for offline use and clears them when the booking ends; Add to Home Screen stays with the PWA (FR-BA-042) (17A, R10, R14). | M | P1 |

### 5.4 Notifications and app shell

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-BA-040 | Notifications: booking confirmed, request received/approved/declined, payment link, reminder (24h and 1h before for meeting rooms; evening before for day passes), cancelled, refund initiated/processed, invoice issued, membership renewal due. Channels: email always; SMS and/or WhatsApp per tenant settings and customer preference. | M | P1 |
| FR-BA-041 | In-app notification list. | S | P2 |
| FR-BA-042 | PWA: installable, offline shell, web push notifications. | S | P2 |
| FR-BA-043 | "Powered by Cowos" badge under the booking app's pay bar and on the confirmation page, with the same rules as FR-TW-003 (30A). | M | P1 |
| FR-BA-044 | Booking app shall show the tenant's support phone/WhatsApp/email on every page footer and on error screens. Every customer-facing screen, error, email, SMS and WhatsApp speaks as the tenant ("Workloft couldn't load times · Try again"), with the tenant's phone/WhatsApp as the help route; "Cowos" appears only in the badge and "Powered by" lines (29A). | M | P1 |
| FR-BA-045 | Theme booking shell: each theme package exports a booking shell (header and footer matching the website, the theme's type scale and fonts, and a product hero-image treatment). Booking screens (slot picker, checkout, account) stay one shared component set. The app dynamic-imports only the tenant's shell chunk named in site-config (theme key + published version), and the edge inlines the shell header/footer HTML with site-config so there is no unbranded flash. CI checks the main bundle ≤200 KB and each shell chunk ≤40 KB gzip (fonts excluded); a tenant on theme X never downloads theme Y. Theme visual-regression tests cover the shell (16A, R13). | M | P1 |

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
| FR-SA-002 | Tenant list: search, filters (status: trial, active, past due, restricted, suspended, cancelled; city; created date), key metrics per tenant (bookings 30d, last login, publish status). There is no plan filter (D1). | M | P1 |
| FR-SA-003 | Tenant detail: profile, KYC status, trial end date, month-to-date units per meter, server invoices and billing history, wallet balance and transactions, domains and SSL status, published versions, staff users, support grants, notes, audit log. | M | P1 |
| FR-SA-010 | First-publish review queue: automated check results (GSTIN/PAN verification, phone/email verified, gateway connected, content scan flags), preview link, approve / reject with reason / request changes; SLA timer. | M | P1 |
| FR-SA-011 | Unpublish, suspend or reinstate a tenant with reason (content violation, fraud, legal request); tenant notified; action audited. Suspend and reinstate raise one TenantStateChanged event that triggers the data-refresh render in FR-PL-012 at launch: the site shows the suspended page (FR-TW-012) within about 2 minutes, and reinstate restores the normal site. Owner publish is blocked while Suspended. A second suspend is a no-op. Suspending during first-publish review pauses the review; reinstate resumes it (CEO-A1). Suspend and reinstate are the only manual state changes in Super Admin; moves into Trial, Active, PastDue and Restricted come from sign-up, payment-method and dunning events (FR-PL-062, FR-PL-063). | M | P1 |
| FR-SA-020 | Rate card editor: SSPL creates a rate card version (RateCardVersion) with an effective date on the 1st of a month and, for each meter (server request units, server storage units, OTP SMS, other SMS, WhatsApp, email), a slab table (RateSlab: from unit, to unit, empty on the last slab, price per unit in ₹, excluding GST). A slab price can be ₹0. Validation: every meter has a table; slabs in a table are contiguous with no gaps or overlaps; only the last slab is open-ended. Publishing records who published it and when, is audited, and rebuilds the pricing page (FR-MW-013). A version that is in force or past can't be changed; a price change needs a new version from a later 1st. The editor lists all versions with their effective dates. Production launch needs a version in force (Pricing Rules #3, #4, D1). A version can start only on a future 1st of a month, with one version per effective month; versions already used for billing are read-only (CEO review 2, Section 2). Drafts (save, duplicate from the version in force, delete), side-by-side diff, impact preview re-rating the last complete month (tenants paying more/less, median and largest change, total), and publish confirmed by typing the effective month (DR2-7A). | M | P1 |
| FR-SA-021 | Server invoices: per tenant and across tenants, list server invoices (month, units and slab breakdown per server meter, GST, amount, due date, payment status), download, resend, issue credit notes, and mark an invoice paid (offline); view the payment method or mandate status; view the daily usage behind each invoice (FR-PL-068); extend a trial. List each monthly rating run (FR-PL-060) with month, rate card version, invoices issued and failures; retry a failed run. GST summary export for SSPL's returns. There are no plans, subscriptions or cost shares (D1, Pricing Rule #5). | M | P1 |
| FR-SA-022 | SSPL cost view: per calendar month, the actual production AWS cost by service (loaded from AWS Data Exports, CUR 2.0, FR-PL-069) and the messaging-provider cost per channel, next to billed revenue (server invoices and wallet debits) and the difference. Read-only and visible only to SSPL. It is never used to compute a tenant's bill (Pricing Rule #10). Leads with a 12-month row of billed revenue, actual cost and margin (₹ and %); revenue per meter is shown beside the closest cost lines, labelled "related cost, not exact" (DR2-8A). | M | P1 |
| FR-SA-023 | Dunning dashboard: past due tenants, stage, last reminder, next action (Pricing Rule #8). | M | P1 |
| FR-SA-024 | Tenant wallet adjustments: credit or debit a tenant's notification wallet with a reason; every adjustment is audited and appears in the tenant's wallet transactions (FR-TA-180). Notification prices are set in the rate card (FR-SA-020) (Pricing Rule #6). | S | P1 |
| FR-SA-030 | Support tools: request support grant, view tenant admin as tenant (read-only by default) within a grant, with a persistent banner and full audit. | M | P1 |
| FR-SA-031 | Ticketing: integrate with a help desk (e.g. Freshdesk/Zoho Desk) or a built-in simple ticket list. | S | P1 |
| FR-SA-040 | Theme catalog: register theme versions, mark deprecated; preview with sample data. A new theme stays hidden until its Lighthouse, axe and screen-reader checks pass (CEO-S3). Every theme is available to every tenant (D1). | M | P1 |
| FR-SA-041 | Platform settings: notification providers and credentials, SMS DLT templates and IDs, WhatsApp templates, reserved slugs, blocked words list, feature flags per tenant. | M | P1 |
| FR-SA-050 | Monitoring: platform health (API error rate, latency, queue depth, failed jobs, webhook failures, DNS/SSL issues by tenant), links to CloudWatch/Grafana dashboards. | M | P1 |
| FR-SA-051 | Business dashboard: sign-ups, trials, conversions to Active, monthly billed revenue (server invoices and wallet debits), churn, referrals by source/tenant, and cost recovery: billed revenue against production AWS and messaging-provider cost per month from the cost view (FR-SA-022), with SSPL's other costs (staging, dev, team) alongside (G3, CEO-A5). The tenant scorecard (FR-SA-058) measures booking success (CEO-A4). | S | P1 |
| FR-SA-052 | Tenant deletion workflow with two-person approval and retention-aware purge. | M | P1 |
| FR-SA-053 | Global announcements to tenants (admin banner + email). | S | P2 |
| FR-SA-054 | Dedicated-cell provisioning for hybrid tenants. | C | P3 |
| FR-SA-055 | Tenants area: the tenant list (FR-SA-002) plus two tabs, Demo requests (FR-SA-057) and Tenant scorecard (FR-SA-058). Built with the DESIGN.md tokens and the state copy in FR-SA-057 and 058 (U1). | M | P1 |
| FR-SA-056 | Removed in v0.3 (pilot programme removed: no Create pilot tenant, Owner invite by SSPL or pre-handover setup grant; Owners sign up self-serve and SSPL helps only through Owner-approved grants, FR-SA-030, Pricing Rule #9). | – | – |
| FR-SA-057 | Demo requests: list of requests from the cowos.in "Book a demo" form (FR-MW-008) with status, a badge when the email to sales failed, and "No demo requests yet" when empty. Requests are kept for 12 months (CEO-A2). | M | P1 |
| FR-SA-058 | Tenant scorecard: per live tenant per month, paid online bookings (customer-made in the booking app, payment captured, not fully refunded; staff-created and offline bookings excluded), cancellations and first publish date. Each paid online booking counts in the month its payment was captured, in Asia/Kolkata time; a later full refund removes it from that month and past months recompute; the page shows "as of <time>" and labels the current month "in progress" (R20). Roll-up: tenants live, tenants at ≥10 this month and their share of active tenants (G2), tenants with 3 consecutive months at ≥10, and expansion gate met yes/no (10 tenants in the beachhead city at ≥10 for 3 consecutive months). CSV export (CEO-A4). | M | P1 |

---

## 8. FR-PL · Platform services (cross-cutting)

| ID | Requirement | Pri | Phase |
|----|-------------|:---:|:---:|
| FR-PL-001 | Tenant resolution: every request resolves a tenant from (a) hostname for public sites/booking app/public API, or (b) the authenticated user's tenant claim for admin APIs; requests with no resolvable tenant are rejected. | M | P1 |
| FR-PL-002 | Data isolation: every tenant-owned table has `tenant_id`; application queries filtered by tenant; PostgreSQL RLS policies enforce the same at the database. | M | P1 |
| FR-PL-010 | Site build service: on publish or rollback, render the site version to static files, upload to S3 under a versioned prefix, atomically switch the tenant's live pointer, invalidate CDN paths. **State check (CEO-A1):** the pointer switch re-reads tenant state in the same transaction; a normal-site render aborts if the tenant is Suspended, and a suspended-page render aborts if it is not. Owner publish and rollback are rejected while Suspended. **Edge sync (R17):** every pointer or state change adds a tenant row to an `edge_sync` outbox in the same transaction as the DB switch. One EdgeSyncWorker drains the outbox, reads each tenant's current DB pointer and state at send time, and writes them to CloudFront KeyValueStore with IfMatch, retrying on 409 and batching tenants. Publish, rollback, domain, state and sign-up jobs never write KVS directly. | M | P1 |
| FR-PL-011 | Data-refresh render when data shown on the website changes (inventory names, prices, hours), without creating a new content version; refreshes run at lower priority than owner publishes. `tenant.data_seq` is incremented in the same transaction as every website-visible data change and every tenant state change (so RT4 suspend and reinstate renders pass the rule); every render records the data_seq it read, and the pointer stores (version_id, data_seq). A data refresh switches the pointer only if version_id is unchanged and its data_seq is higher; otherwise it discards itself and logs (CEO-A3, R18). | S | P1 |
| FR-PL-012 | TenantStateCtaService: on each TenantStateChanged event, enqueue a data-refresh render of the tenant's live site version (rules as FR-PL-011) so pages are correct within about 2 minutes p95. Suspended → suspended page (FR-TW-012); leaving Suspended → normal site; entering Restricted → Book CTAs become "Call / WhatsApp to book"; leaving Restricted → Book restored. Triggers are FR-SA-011 suspend/reinstate (CEO-A1) and the dunning-driven Restricted and Suspended moves (FR-PL-063) (R11, Pricing Rule #8). | M | P1 |
| FR-PL-020 | Domain service: add domain, ownership verification via TXT, DNS polling, certificate issuance and renewal, health checks, removal. | M | P1 |
| FR-PL-030 | Media pipeline: virus scan, strip EXIF location data, generate responsive variants (WebP/AVIF + JPEG fallback; widths 320/640/1024/1600/2400), store in S3, serve via CDN. Stored media and files count toward the tenant's server storage units (FR-PL-067, Pricing Rule #3). | M | P1 |
| FR-PL-040 | Notification service: templated email (SES), SMS (DLT-compliant CPaaS), WhatsApp (BSP/Cloud API); per-message status tracking, retries, fallback channel (WhatsApp fails → SMS); every message sent is counted on its channel's meter and debited from the wallet at its slab price (FR-PL-042). A WhatsApp message followed by its SMS fallback counts once on each meter (Pricing Rules #3, #6). | M | P1 |
| FR-PL-041 | Quiet hours for non-urgent messages (no promotional/reminder messages 9 PM–8 AM IST except booking within that window). | S | P1 |
| FR-PL-042 | Notification wallet (prepaid): for each message, one transaction increments the tenant's MonthlyMeterCount for the channel (OTP SMS, other SMS, WhatsApp, email), takes the slab price for the new count from the rate card version in force, debits the wallet with one conditional UPDATE that succeeds only if the balance minus the price stays at or above the floor, and writes the WalletTransaction (channel, count, slab, price). Count and debit commit together or not at all (R4). Floors: ₹0 for SMS and WhatsApp notifications; −₹500 for OTPs and emails. A failed SMS or WhatsApp debit falls back to email. Which messages stop when OTPs and emails reach −₹500 is open (design OQ4). Counts reset at the start of each calendar month (IST). Minimum top-up ₹500 (BRD §7.1). Trial tenants need wallet balance from day one (Pricing Rules #4, #6, #7). When a delivery report shows the provider did not charge SSPL (a WhatsApp message not delivered within 24 hours, or an SMS rejected by DLT scrubbing before submission), the wallet is credited, the month's count for the channel goes down by one and a linked credit WalletTransaction records the reason; duplicate reports credit nothing more; emails are never credited (R23). Sign-in OTP emails are always sent: past the −₹500 floor their cost is absorbed by SSPL and logged, not debited, and the wallet page tells the Owner "Sign-in codes are still being sent; top up to restore SMS and WhatsApp" (R25). | M | P1 |
| FR-PL-050 | Payment service: gateway adapter interface; order creation on tenant account; signature verification; webhook ingestion (idempotent, signature-verified, stored raw); reconciliation job comparing gateway payments to bookings daily. | M | P1 |
| FR-PL-051 | Booking hold expiry job: release holds after 10 minutes; late payment success after expiry triggers auto-refund if the slot was taken, or confirmation if still free. | M | P1 |
| FR-PL-052 | Invoice service: numbering with gap-free sequences per series per FY (transactional), PDF generation, storage, regeneration of the same PDF on demand. | M | P1 |
| FR-PL-060 | Server rating and invoicing (MonthlyRating job): on the 1st (IST), for the previous calendar month, sum each tenant's UsageDaily units per server meter (request units; storage units as the average of the daily snapshots), leaving out days the tenant was in Trial. Apply the telescopic slabs of the rate card version in force for that month: units in slab 1 at the slab-1 price, the next units at the slab-2 price, and so on. Write one ServerInvoice per tenant with one ServerInvoiceLine per server meter (units, slab breakdown with units and price per slab, amount), GST added (BRD §7.4), due in 10 days, and send it to the Owner by email and WhatsApp. The job is idempotent per tenant and month; a failed run alerts SSPL and can be retried from Super Admin (FR-SA-021). Tables: Meter, UsageDaily, RateCardVersion, RateSlab, ServerInvoice, ServerInvoiceLine. No subscriptions, plans, cost shares or proration (D1, Pricing Rules #4, #5, #7). Before rating, every IST day of the month must have a completed run for each metering source (log job, storage snapshot); otherwise invoices are held, SSPL Finance is alerted and rating retries daily; invoices issued after the 1st say why; reruns are idempotent (C2-A2). | M | P1 |
| FR-PL-061 | Feature flags (FR-SA-041) evaluated server-side. There are no plan limits or paid tiers: every tenant gets every feature (D1, Pricing Rule #2). | M | P1 |
| FR-PL-062 | Tenant state machine (Trial → Active → PastDue → Restricted → Suspended → PendingDeletion → Deleted; Cancelled path) controls site serving, booking acceptance and admin capabilities. Sign-up creates the tenant in Trial for 14 days (FR-MW-005): it publishes to the platform subdomain only, its server usage is not billed, and its notifications need wallet balance. Adding a payment method moves it from Trial to Active (FR-TA-176). Unpaid server invoices move a tenant through PastDue, Restricted and Suspended on the BRD §7.3 schedule (FR-PL-063); payment returns it to Active. Super Admin suspends and reinstates manually (FR-SA-011). Every state change raises one TenantStateChanged event (FR-PL-012) (Pricing Rules #7, #8). | M | P1 |
| FR-PL-063 | Server invoice collection and dunning: each ServerInvoice is paid by Razorpay payment link, or by optional UPI Autopay or e-mandate with a ₹5,000 per-tenant maximum and the RBI pre-debit notice. Billing is monthly in arrears. Unpaid invoices follow the dunning schedule with automatic PastDue, Restricted and Suspended moves (BRD §7.3); the dunning dashboard shows each tenant's stage (FR-SA-023) (Pricing Rules #5, #8). | M | P1 |
| FR-PL-064 | ChargeEstimateService: once a day after the usage jobs (FR-PL-066, FR-PL-067), rate each tenant's month-to-date server units with the rate card version in force and store the estimate per meter with its slab breakdown. Days in Trial are left out. The billing page (FR-TA-175) reads the stored value (Pricing Rule #5, 15A). | M | P1 |
| FR-PL-065 | Removed in v0.3 (pricing JSON with "last month's actual" share replaced: the pricing page shows the rate card, FR-MW-013, Pricing Rule #4). | – | – |
| FR-PL-066 | Request metering: CloudFront standard logs for the tenant website and booking-app hosts are delivered to S3; a daily job maps each request's host (platform subdomain or verified custom domain) to its tenant and adds the count for that IST day to UsageDaily (request meter). API calls are counted from the same logs: `/api/*` requests on those hosts (R22); Tenant Admin, Super Admin and support traffic is not metered. Only responses with status 2xx or 304 count; redirects, client errors and server errors are not billed (R21). Requests blocked by WAF and SSPL's own health checks and monitoring are not counted. Requests whose host maps to no tenant are not counted. Each request is counted once (Pricing Rule #3). Only HTML document requests (page views) count from the logs; images, CSS, JS, fonts and other static files are not counted, so 1 unit = 1,000 page views and API calls (CEO review 2, C2-A1). There is no in-process API counter (R22). | M | P1 |
| FR-PL-067 | Storage metering: once a day, a job records each tenant's stored media and files in GB to UsageDaily (storage meter). Monthly storage units are the average of the month's daily snapshots; 1 storage unit = 1 GB-month (Pricing Rule #3). | M | P1 |
| FR-PL-068 | Usage detail: UsageDaily (tenant, meter, date, units) is kept for every meter, including notification counts rolled up from the wallet transactions, so the Owner can see the daily usage behind each server invoice and wallet debit (FR-TA-175, FR-TA-180) and SSPL can answer billing questions (FR-SA-021) (Pricing Rule #5). | M | P1 |
| FR-PL-069 | SSPL cost import: each month, load AWS Data Exports (CUR 2.0) from the production account and the messaging providers' charges into PlatformCostMonth for the SSPL cost view (FR-SA-022). This data is never used to bill tenants (Pricing Rule #10). | M | P1 |
| FR-PL-070 | Audit log service: append-only, tenant-scoped and platform-scoped events with actor, impersonator, IP, before/after. | M | P1 |
| FR-PL-071 | Data export service: async job builds a zip (CSV/JSON + media) and emails a time-limited link to the Owner. | M | P1 |
| FR-PL-080 | Calendar: generate `.ics` per booking; Google Calendar add link. | M | P1 |
| FR-PL-081 | One-way push of meeting-room bookings to a tenant's Google/Outlook resource calendar. | C | P3 |
| FR-PL-090 | Search-engine-friendly redirects and canonical handling across platform and custom domains. | M | P1 |
