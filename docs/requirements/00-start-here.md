# 00 · Start Here: Questions, Assumptions, and What I'd Change

> Cowos requirements pack · Version 0.5 (draft) · Owner: SSPL Product · Date: 2026-11-11

This pack has 13 deliverables (files `01` to `13`). Read this file first. It holds five things:

1. Clarifying questions, each with the assumption the rest of the pack uses if you don't answer.
2. Changes I recommend to the original brief, with the reasoning. Several are significant.
3. The decisions you need to make before engineering starts.
4. Pricing and billing (decided 2026-11-04).
5. The release phases.

v0.3 applies the product owner's revision of 2026-11-04 to the approved design doc `docs/designs/cowos-platform-v1.md`: metered slab pricing (Pricing and Billing Rules #1–#11), one app server for hosting (Rule #11), and no pilot programme. Design decisions 1A–33A and the engineering and CEO review decisions that the revision does not replace still stand.

v0.4 applies revision 2 of the same day: two pricing plans, Fixed and Usage (Pricing Rule #12), and grid seat maps that owners build and customers pick seats from, at launch (Rule #13, change C13). It adds a build plan for a small team using Claude Code (14), with a launch target of week 20 [estimated]. Revision 2 has not yet been through the CEO, design or engineering reviews.

v0.5 (2026-11-11) adds five requested changes, C15 to C19 in section 2: floors with an uploaded floor plan to trace over and prices by seat type; renewals and "booking ending" alerts; a tenant-managed amenity list; check-in by QR or code inside a configurable window; and SSPL offer codes that make Cowos free for the first 3 to 5 tenants. None of these has been reviewed, and none is scheduled in the build plan yet (14 §4, 13 OQ42). Decided 2026-11-11: Cowos launches only when all the features are complete, so all five are launch scope and the launch date follows the re-estimated build plan, not week 20.

---

## 1. Clarifying questions and default assumptions

| # | Question | Assumption used in this pack (if you don't answer) |
|---|----------|------------------------------------------------------|
| Q1 | Who is the beachhead customer? Metro operators (Bengaluru, Hyderabad, Pune) or Tier-2/3 spaces? | Small and mid-size independent spaces (1–3 locations, 20–300 seats) in Tier-1 and Tier-2 Indian cities. Price sensitive, run on WhatsApp + Excel today. Beachhead: independent 1–3 location spaces in one city with a meaningful share of bookings from aggregators. Hyderabad was suggested; the city is not confirmed (13 OQ16). |
| Q2 | Is the product India-only for v1? | Yes. INR, GST, Indian phone numbers, IST, Razorpay, DLT-registered SMS. Data designed so currency, tax and timezone are per tenant, so other markets can be added later. |
| Q3 | Is "Powered by SSPL" the final badge text? Customers will not know who SSPL is. | Badge reads **"Powered by Cowos"** (optionally "Cowos by SSPL") and links to the Cowos marketing site with a referral code. See Change C6. |
| Q4 | Does SSPL want to be in the money flow for customer bookings? | No. Customer money goes straight to the tenant's own payment gateway account. SSPL never holds customer funds. See Change C4. |
| Q5 | What does "charge only for server cost" mean commercially? Is SSPL willing to run at zero margin? | **Answered (D1, revised 2026-11-04, revision 2).** Each tenant chooses two pricing plans the owner chooses between: a **Fixed plan** (one flat monthly fee, unlimited server usage that is not metered for billing, and messages included up to a monthly fair-use allowance per channel) and a **Usage plan** (no monthly fee; server usage and messages metered and priced in slabs like an electricity bill). Both plans include every feature. SSPL sets the prices, so they may include a margin (Premise 5). Cowos takes **0% commission on bookings**: it never takes a share of the tenant's booking revenue. What the tenant charges its own customers (prices, discounts, promo codes, fees) is the tenant's decision, not Cowos's. See Change C1. |
| Q6 | How many tenants do you expect in year 1 and year 3? | 50–150 tenants in year 1, 500–1,500 by year 3. Average 15,000 customer-facing page views and 300 bookings per tenant per month. |
| Q7 | Do tenants have multiple locations (centres)? | Data model supports many locations per tenant from day 1. The MVP admin UI is tuned for 1 location; multi-location UI lands in Phase 2. |
| Q8 | Should private cabins and dedicated desks be instantly bookable online? | No, not by default. These are sales-led deals (tour, quote, agreement, deposit). Cowos supports per-product **booking modes**: Instant Book, Request to Book, and Enquire / Book a Tour. See Change C7. |
| Q9 | Are customers' accounts shared across tenants? (One login for all Cowos-powered spaces?) | No. Each customer account belongs to one tenant. This keeps the white-label promise and makes the tenant the clear data owner under the DPDP Act. |
| Q10 | Do you need native mobile apps in v1? | No. Responsive web in Phase 1, installable PWA in Phase 2. Per-tenant native apps are not recommended (see Change C9). |
| Q11 | Is the Super Admin "publishing approval" a manual review of every publish? | Only the tenant's **first** publish (plus any custom-domain change) needs SSPL approval, after a KYC check. Later republishes go live immediately and are audited. See Change C8. |
| Q12 | What team will build this, and what is the target MVP date? | 2 backend (.NET), 2 frontend (React), 1 full-stack/DevOps, 1 QA, 1 designer (part-time), 1 PM. MVP in ~5–6 months. |
| Q13 | Do you have the domains? | SSPL owns or will buy `cowos.in` (marketing + admin) and a **separate** domain for tenant sites, for example `cowos.site`. See Change C5. |
| Q14 | Language support? | English only in Phase 1. Content model stores text so Telugu/Hindi can be added in Phase 3. |
| Q15 | Is a free trial allowed? | Yes, for every new tenant from launch (Pricing Rule #7). 14-day trial, no card needed, publish allowed on a platform subdomain only, with a "Trial" ribbon in admin (not on the public site). Server usage is not billed during the trial; notifications need wallet balance from day one. |
| Q16 | Who handles customer support for end customers (members)? | The tenant. SSPL supports tenants only. The booking app shows the tenant's contact details, never SSPL's. |

---

## 2. Changes I recommend to the brief

Each change lists the original idea, the problem, and the better alternative. All later documents assume these changes. If you reject one, the affected sections are listed so you know what to rework.

### C1. Pricing: a Fixed plan or a Usage plan (metered server usage and notifications in slabs) + 0% commission (decided, D1)

**Original:** Charge tenants only for their EC2 cost, shown with a transparent cost breakdown.

**v0.1 recommendation (superseded):** flat monthly plans, because a pass-through charge of ₹100–₹400 per tenant per month cannot fund support, development or sales. One EC2 per tenant was rejected as too expensive (C2).

**Decision (D1, product owner, 2026-11-02, revised 2026-11-04 and again the same day in revision 2; Premise 5):** each tenant chooses two pricing plans the owner chooses between: a **Fixed plan** (one flat monthly fee, unlimited server usage that is not metered for billing, and messages included up to a monthly fair-use allowance per channel) and a **Usage plan** (no monthly fee; server usage and messages metered and priced in slabs like an electricity bill). There are no feature tiers, plan limits or commission: every tenant gets every feature on either plan (Pricing Rule #2). SSPL sets the prices, so they may include a margin. The 2026-11-02 version (an equal share of the AWS bill plus messaging at cost) is replaced.

How the charge works (Pricing Rules #3–#7, #10, #12; full rules in BRD §7):
- **Plans (Rule #12):** the Owner chooses Fixed or Usage when the tenant becomes Active, and can switch at any time with effect from the 1st of the next month. **Fixed:** one flat monthly fee set by SSPL on the rate card; server usage unlimited and not billed; messages included up to a monthly allowance per channel, then paid from the wallet; the fee is billed in arrears on the monthly invoice, prorated by day only in the first and last month. **Usage:** no fee; the meters and slabs below apply.
- **Meters (Rule #3):** server request units (1 unit = 1,000 page views and API calls on the tenant's website and booking-app hosts, read from CloudFront logs: page views are HTML document requests, API calls are `/api/*` requests; only 2xx and 304 responses count; static files, WAF-blocked requests, SSPL monitoring and Tenant Admin, Super Admin and support traffic are not counted; C2-A1, R21, R22); server storage units (1 unit = 1 GB-month of the tenant's media and files, averaged from daily snapshots); notification units (1 unit = 1 message, metered separately for OTP SMS, other SMS, WhatsApp and email).
- **Slabs (Rule #4):** every meter has its own slab table, applied telescopically. SSPL sets slab boundaries and prices in a Super Admin rate card. A slab price can be ₹0 (a free allowance). Each rate card version takes effect on the 1st of a month. Counts reset at the start of each calendar month in IST.
- **Server invoice (Rule #5):** monthly in arrears, issued on the 1st for the previous month: a Fixed plan line, or (Usage plan) one line per server meter with units and slab breakdown; plus GST; due in 10 days. Paid by Razorpay payment link, or by optional UPI Autopay or e-mandate (₹5,000 per-tenant maximum, RBI pre-debit notice).
- **Notifications (Rule #6):** prepaid wallet. Each message is debited at its channel's slab price for the month-to-date count, in one atomic step with the count (eng R4).
- **Trial (Rule #7):** 14 days; no plan fee and server usage is not billed; notifications need wallet balance.
- **Transparency:** the pricing page shows the current rate card (28A, Rule #5). The billing page shows units per meter, the slab breakdown and a daily month-to-date estimate. The wallet page shows the month-to-date count and current slab per channel.
- **SSPL cost view (Rule #10):** SSPL loads AWS Data Exports (CUR 2.0) from production only to compare its actual AWS and messaging-provider cost with billed revenue each month. CUR is not used to bill tenants.
- **Prices:** the product owner configures the Fixed plan fee and message allowances (13 OQ35) and the slab boundaries and prices (13 OQ21) in the rate card before launch.
- **Timing:** metering, the rate card, server invoices and the wallet ship at launch (P1). Every tenant pays from the end of its trial.

**What "0% commission" means (and doesn't mean):**
- It is a promise from **Cowos to the tenant**: Cowos takes no percentage or per-booking fee from the tenant's booking revenue. The tenant's only charges from Cowos are its plan: the Fixed plan fee (plus wallet messages above the allowance), or its metered server usage and notifications on the Usage plan.
- It says **nothing about what the tenant charges its customers.** Prices, member rates, promo codes, discounts, convenience fees, deposits and whether to pass gateway fees on to customers are all set by the tenant in the Tenant Admin. Cowos never adds its own fee to a customer's bill.
- Payment gateway charges (Razorpay's MDR) are between the tenant and Razorpay. Cowos doesn't mark them up.

**Revenue:** SSPL's revenue is the Fixed plan fees (predictable) plus metered charges from Usage plan tenants and wallet messages, at the prices it sets. The earlier accepted risk that cost-only charges can't fund SSPL (13 R1) no longer applies. Goal G3 compares billed revenue with SSPL's production AWS and messaging-provider cost (CEO-A5, Rule #10).

Affects: 01 Vision, 03 BRD §7, 04 SRS (FR-MW-002/003/004, FR-PL-060/061/062, FR-TA-175/176, FR-SA-020/021), 08 Architecture, 09 Data model, 12 Release plan, 13 Decisions.

### C2. Hosting: one EC2 per tenant → shared multi-tenant platform on one app server (hybrid later) (decided, D2)

**Original:** One EC2 instance per tenant.

**Problem:** Fine for 5 tenants, painful at 50, unworkable at 500. Every deploy, OS patch, .NET security update, certificate, log pipeline and backup is multiplied by the tenant count. Most tenants sit at <2% CPU, so you pay for idle servers.

Comparison (rough ap-south-1 Mumbai on-demand figures, Oct 2026; validate with the AWS Pricing Calculator):

| Factor | A. EC2 per tenant (silo) | B. Shared multi-tenant (pool) | C. Hybrid (pool default, silo for large) |
|--------|--------------------------|-------------------------------|------------------------------------------|
| Infra cost per small tenant / month | ~₹2,500–₹4,000 (t4g.small + EBS + public IPv4 + DB + backups + logs) | ~₹100–₹400 at 200+ tenants (shared app server + managed PostgreSQL + CloudFront) | Pool tenants pay metered usage (D1); how a silo cell is charged is open (13 OQ20) |
| Fixed base cost | Low at start, grows linearly | ~₹25,000–₹45,000/month base regardless of tenant count | Same as B, plus each silo stack |
| Scalability | Per tenant, manual or per-tenant ASG | One app server, resized as load grows; tenant websites scale on CloudFront (Rule #11) | Both |
| Isolation | Strong (separate VM and DB) | Logical: TenantId + PostgreSQL Row-Level Security + per-tenant encryption context | Strong for silo tenants, logical for the rest |
| Security | Smaller blast radius, but 500 servers to patch means patch lag, which is its own risk | One hardened server, patched once; needs strict tenant-isolation tests | Same as B plus silo |
| Availability | One tenant's server fails alone | The app server is a single point of failure for admin and booking; tenant websites stay up on CloudFront (Rule #11) | Same as B for pool tenants |
| Deployment | N deploys per release; version drift across tenants | One deploy per release on the app server; whether a few minutes of downtime per deploy is acceptable is open (13 OQ25) | One pool deploy + silo deploy wave |
| Operations | Heavy: N sets of monitoring, backups, certs | Light: one set, per-tenant dashboards by tag | Medium |
| Noisy neighbour risk | None | Exists; mitigated by rate limits per tenant, static sites on CDN | Large tenants move to silo |
| Time to onboard a tenant | Minutes to provision (Terraform per tenant) | Seconds (a database row) | Seconds; silo on request |

The cost figures in column B were estimated for the v0.2 container design. Re-check them with the AWS Pricing Calculator for one app server.

**Decision for v1 (D2, product owner, 2026-11-04; Rule #11): B (shared multi-tenant) on one app server, architected so C is possible later.** Concretely:
- Tenant websites are **pre-rendered static files** on S3 + CloudFront at publish time. They cost almost nothing to serve and stay up even if the app server is down.
- The booking app is **one React build** served to every tenant host, styled at runtime from the tenant's theme.
- One app server (EC2) runs every tenant's ASP.NET Core API (modular monolith), the admin portals, the booking app, the background worker and the site renderer. It has automatic instance recovery, an infrastructure-as-code rebuild path and health alarms.
- Managed PostgreSQL (RDS) with backups and point-in-time recovery, shared schema with `tenant_id` and Row-Level Security.
- The availability target for the app server is open (13 OQ25).
- A tenant catalog maps each tenant to a "cell" (database + compute). In v1 every tenant is in cell `pool-1`. In Phase 3, a Dedicated tenant gets its own cell with no code change.

Affects: 07 NFR, 08 Architecture, 12 Release plan.

### C3. Tenant websites must be server-rendered or pre-rendered, not a client-side SPA

**Original:** React frontend (implied single-page app for everything).

**Problem:** Tenant websites live or die on Google ranking for "coworking space in Madhapur". A client-rendered React SPA ships an empty HTML shell, ranks poorly, and has slow first paint on mid-range Android phones.

**Better:** Keep React and TypeScript, but render the tenant **website** to static HTML at publish time (React server-side rendering in a Node build worker, output uploaded to S3). The **booking app** stays an SPA because it is behind interactions and doesn't need ranking (its landing page gets pre-rendered meta tags). The admin portals stay SPAs.

Note: this adds a small Node.js renderer service next to the .NET backend. That's unavoidable with React on the server and is a normal pattern.

Affects: 07 NFR (SEO), 08 Architecture.

### C4. Payments: Razorpay Route (split settlement) → tenant's own Razorpay account via OAuth

**Original question:** Tenant's own keys, or Razorpay Route for split settlements?

**Problem with Route:** Route is built for marketplaces that collect money and split it. With 0% commission there is nothing to split. Using Route puts SSPL in the money flow: SSPL becomes responsible for refunds, chargebacks, disputes and settlement reconciliation for every tenant, and needs linked-account KYC for each tenant. That brings regulatory and operational load with no benefit.

**Problem with pasting API keys:** Works, but tenants paste secrets into a form (support burden, leaked keys, keys rotate and break silently).

**Better:**
1. **Primary: Razorpay Partner OAuth ("Connect with Razorpay").** Cowos registers as a Razorpay Technology Partner. The tenant clicks "Connect Razorpay", logs into their own Razorpay account, approves access. Cowos gets a scoped token to create orders and refunds on the tenant's account. Money settles directly to the tenant's bank. The tenant's Razorpay KYC covers the merchant.
2. **Fallback: API key + secret**, stored encrypted with AWS KMS, for tenants whose account can't use OAuth.
3. **Offline payments:** front desk can record cash, bank transfer or UPI received outside the gateway. Monthly members in India often pay this way. Without it, tenants keep a second spreadsheet and churn.
4. **Phase 2:** add a second gateway (Cashfree or PhonePe PG) behind the same `IPaymentGateway` interface.

Cowos uses Razorpay Checkout (hosted/modal). Card data never touches Cowos servers, so PCI-DSS scope stays at SAQ-A level.

SSPL's own platform billing (tenant pays its server invoice and tops up its notification wallet) uses SSPL's own Razorpay account: a payment link per monthly server invoice, or optional UPI Autopay / e-mandate with a ₹5,000 per-tenant maximum and the RBI pre-debit notice (Pricing Rule #5). It ships at launch (P1).

Affects: 04 SRS (FR-TA-PAY, FR-BA-PAY), 08 Architecture, 11 Integrations.

### C5. Domain structure: fix the wildcard certificate trap and isolate tenant sites from the admin domain

**Original:** `<tenant>.cowos.in` for website, `booking.<tenant>.cowos.in` for booking.

**Problems:**
- A wildcard certificate `*.cowos.in` covers one level only. `booking.acme.cowos.in` is two levels deep, so every tenant needs its own certificate. That is avoidable.
- If tenant sites (which display tenant-supplied content) share the parent domain with the admin portal (`app.cowos.in`), a compromised or malicious tenant page can set cookies for `.cowos.in` (cookie tossing) and attack admin sessions. GitHub serves user pages from `github.io` instead of `github.com` for this reason.

**Better:**

| Surface | Platform domain (no custom domain) | Tenant's own domain |
|---------|-------------------------------------|---------------------|
| Cowos marketing | `cowos.in` | n/a |
| Tenant Admin Portal | `app.cowos.in` | n/a (white-label admin is Phase 3) |
| Super Admin Portal | `ops.cowos.in` (SSO + IP allowlist) | n/a |
| Tenant website | `acme.cowos.site` | `www.acmecowork.com` (apex redirects to www) |
| Tenant booking app | `acme.cowos.site/book` | `book.acmecowork.com` (CNAME), or `www.acmecowork.com/book` |
| API | Same origin as the page: `/api/*` routed by CloudFront | Same origin: `/api/*` |

- One wildcard certificate `*.cowos.site` covers every tenant that has no domain.
- Path-based `/book` on the platform domain means one hostname, one cert, one cookie scope, and the booking app inherits the website's SEO authority.
- On a custom domain, the tenant chooses `book.<domain>` (matches the Directoo pattern and works best for tenants who keep their existing website) or `/book` on the same host.
- Apply to add `cowos.site` to the Public Suffix List so browsers treat each tenant subdomain as a separate site for cookies.

**Apex domain trap (India specific):** Most Indian tenants buy domains from GoDaddy, Hostinger or BigRock, whose DNS does not support CNAME/ALIAS at the apex (`acmecowork.com`). CloudFront has no fixed IP by default. Recommended handling: the site runs on `www`, and the tenant sets up the registrar's free "domain forwarding" from apex to `www` (we show step-by-step guides per registrar). Option for later: offer static anycast IPs for apex A records (CloudFront Anycast Static IPs or Global Accelerator) if demand justifies the cost. There are no paid tiers to reserve it for (D1).

Affects: 06 Flows, 08 Architecture, 09 Data model (Domain).

### C6. Badge: "Powered by SSPL" → "Powered by Cowos"

A badge is a growth channel (every tenant site advertises the product). Visitors searching "SSPL" will find nothing useful, and SSPL is a generic acronym shared by other companies. Use "Powered by Cowos", link to `cowos.in/?ref=<tenant-slug>` for attribution, and keep "a product of SSPL" on the Cowos marketing site. The badge is mandatory for every tenant in v1. Make it small, accessible, theme-aware, and not removable through custom CSS (rendered by the platform, checked at publish).

### C7. Booking modes: not everything should be "Book and pay now"

Generic booking software assumes every product is bought instantly. Coworking is different:

| Product | How it actually sells | Default Cowos booking mode |
|---------|----------------------|----------------------------|
| Day pass / hot desk (day) | Impulse, same day | Instant Book + pay |
| Meeting room (hourly) | Planned, often by members | Instant Book + pay, or use member credits |
| Event space | Needs approval, custom setup | Request to Book (tenant approves, then customer pays) |
| Dedicated desk (monthly) | Tour, then commit | Enquire / Book a Tour, or Request to Book |
| Private cabin (monthly) | Tour, quote, agreement, deposit | Enquire / Book a Tour (lead goes to CRM) |
| Virtual office | KYC documents, agreement | Apply online + upload KYC, tenant approves, then pay |

"Book a Tour" is the single highest-value call to action on a coworking website. The MVP includes lead capture and a simple lead pipeline because most of a tenant's revenue (cabins, dedicated desks) comes through it, even though it isn't instant booking.

### C8. Publishing approvals: approve the tenant once, not every publish

Manual review of every publish creates a queue that SSPL staff must clear 7 days a week, and tenants will hate waiting to fix a typo. Instead:
- **First publish:** KYC check (GSTIN or PAN validation, phone/email verified, payment gateway connected) plus a quick manual review by SSPL support (target under 4 working hours). This blocks fake "coworking" sites that collect payments.
- **Later publishes:** live immediately, with automated checks (badge present, no blocked words/links, images scanned) and a full audit trail. Super Admin can unpublish at any time.
- **Custom domain change:** re-verified automatically through DNS; no manual approval.

### C9. Mobile: PWA, not per-tenant native apps

Per-tenant native apps mean one Apple Developer and one Google Play listing per tenant, store review for each release, and Apple's guideline 4.2.6 rejects apps generated from a template. Recommended path: responsive web (Phase 1), installable PWA with push notifications (Phase 2; iOS supports web push for home-screen apps since iOS 16.4). Revisit a single "Cowos Member" native app (one app, tenant chosen at login) only if PWA engagement data shows the need.

### C10. Embedding in an existing website: subdomain link first, widget second, no raw iframes for checkout

| Option | How | Pros | Cons | Verdict |
|--------|-----|------|------|---------|
| 1. Subdomain link | Tenant points `book.theirdomain.com` (CNAME) to Cowos; adds "Book now" buttons linking to it | Works with any site builder (Wix, WordPress, static), full-page UX, payments and login work, no cookie issues | Visitor leaves the main site (same domain, so it feels native) | **MVP, recommended default** |
| 2. Script snippet (floating button + modal) | `<script src="https://cdn.cowos.in/widget.js" data-tenant="acme">` adds a "Book a space" button and opens a modal | Feels integrated, quick install, good conversion | Third-party iframe: Safari and Chrome partitioning block login cookies; UPI intent links can fail inside iframes on mobile | **Phase 2**, with checkout and login opening in a full page or popup, not inside the iframe |
| 3. Raw iframe embed | Tenant pastes an `<iframe>` | Simple | Height/responsiveness issues, same cookie and payment problems, poor accessibility | Not offered (the snippet in option 2 handles resizing) |
| 4. Web component in host DOM | Render booking UI inside their page | Seamless look | CSS conflicts with host site, larger security surface, hard to support | Not recommended |
| 5. WordPress plugin | Shortcode/block that renders the widget | Large share of existing Indian SMB sites are WordPress | Plugin maintenance and review | Phase 3 |

### C11. Things missing from the brief that coworking tenants will ask for in week one

These are added to the requirements:
- **Customer companies (B2B):** a company account with multiple team members, one GSTIN, consolidated monthly invoice. Corporates need the GSTIN on the invoice to claim input tax credit. Without it they won't pay through the app.
- **Memberships with credits:** e.g. "Dedicated Desk ₹7,000/month includes 5 meeting room hours". Credits are consumed by bookings.
- **Security deposits** for monthly plans (recorded, refundable).
- **Check-in** (QR code or front-desk tap) for day passes and meeting rooms, plus no-show marking.
- **Lead and tour pipeline** (see C7).
- **Holidays and blackout dates** per location.
- **Offline payment recording** (see C4).
- **Credit notes** for refunds after an invoice is issued (GST requires it; a deleted or edited invoice is not compliant).
- **Customer GSTIN validation** and correct place of supply. For services tied to immovable property, the place of supply is the property's location, so most invoices are CGST + SGST. Confirm with a tax advisor.
- **Exports** (bookings, customers, invoices to CSV/Excel) because accountants will ask.

### C12. Things in the brief that are expensive for the value, moved later

| Item | Why it's expensive | Where it goes |
|------|-------------------|---------------|
| Free-form floor-plan canvas (drag, rotate and resize desks anywhere) | Months of work, and hard for owners to use | Not planned. Replaced in v0.4 by the grid seat map at launch (C13, Rule #13) |
| Two-way Google/Outlook calendar sync | OAuth per user, webhook channels, conflict resolution | MVP: `.ics` attachment + "Add to Google Calendar" link. Phase 3: one-way push for meeting rooms |
| Free-form drag-and-drop builder | See section 4 of the BRD; large build and support cost | Not planned; section-based builder instead |
| Per-tenant WhatsApp Business numbers | Meta Business verification for each tenant | MVP: platform WhatsApp number with tenant name in approved templates. Phase 2: tenant's own number through embedded signup |
| GST e-invoicing (IRN) | GSP integration and per-tenant credentials | Phase 3 (needed by tenants over the e-invoice turnover threshold) |

### C13. Seat maps: owners draw their space, customers pick a seat like a movie ticket (decided, revision 2)

**Original:** "floor layout (optional)" in inventory and "select seat/room" in the booking app.

**v0.1–v0.3 position (superseded):** a list of units at launch, with "upload a floor image and drop pins" in Phase 2 (old FR-TA-068).

**Decision (product owner, 2026-11-04, revision 2; Pricing Rule #13):** every coworking space has a different shape and seating, and describing it is the hardest setup step for an owner. At launch, owners build a **grid seat map** per floor or zone in the Tenant Admin: start from a template (Rows of desks, Pods of 4, Cabins along a wall, Meeting room + desks, Blank) or a rows × columns grid, then tap or drag to paint Desk, Cabin, Meeting room, Aisle, Wall, Entrance and Label cells. Seats get labels automatically (A1, A2...). Painting creates the bookable units. Customers then choose a seat on the map in the booking app (free green, booked grey, out of service hatched, their pick gold), or keep the one-tap "Any available seat". A list view gives the same choice to keyboard and screen-reader users. The full design is "Seat maps (revision 2)" in the design doc.

Why a grid and not a free canvas or floor photo: a grid works on a phone, an owner can finish it in minutes, the same grid renders fast for customers, and seat-level availability is simple to compute. A floor photo behind the grid is a possible later option.

**Cost:** about 4 weeks of frontend and backend work in the full-team plan (12 §3); a 2-week slice for two developers in the small-team build plan (14, weeks 8–9).

Affects: 03 BRD §5, 04 SRS (FR-TA-068, FR-TA-068a–e, FR-TA-063, FR-TA-091, FR-BA-013, FR-BA-017, FR-BA-019), 05 stories (US-4.6–US-4.8), 06 flows, 07 NFR (accessibility of the map), 09 data model (SeatMap, seat fields on ProductUnit), 10 API, 12 release plan, 13 OQ36.

### C14. Conversational assistants: chat that fills the form, one per app (after launch; added 2026-11-11)

**Request (product owner, 2026-11-11):** every form should have two ways in. The form stays as it is. A chat assistant collects the same details in conversation, then opens the real form with them filled in; the person reviews, edits and submits there. Each app has its own assistant for its own forms (the marketing site for sign-up and onboarding, the booking app for bookings, and so on), not one shared assistant.

**Status:** recorded as a requirement to be introduced after launch, on a date not yet set (13 OQ37). It is not in the week-20 build plan (14) and has not been reviewed. Who pays for the AI usage (13 OQ38), the exact forms per app (OQ39) and the privacy questions (OQ40, OQ41) are open.

Affects: 04 SRS §9 (FR-AI-001 to FR-AI-019), 05 stories (E14), 12 release plan §2, 13 (R32, R33, OQ37–OQ41).

### C15. Layout: floors, an uploaded floor plan to trace over, and prices by seat type (added 2026-11-11, v0.5)

**Request (2026-11-11):** every coworking space has its own layout and floors. The tenant gives the number of floors and uploads a layout plan per floor, the layout grid in the booking app and Tenant Admin comes from it, and the tenant sets prices for seats and rooms the way cinemas price seat categories.

**What the pack now says:**
- **Floors and plans.** The tenant sets the number of floors per location, names each one and uploads one plan per floor (FR-TA-068g). A floor with no plan still starts from a template.
- **Trace over the plan.** The plan shows behind the grid in the builder. The tenant sizes the grid so one cell covers one desk, lines the plan up and paints desks, cabins and rooms over it (FR-TA-068h). The bookable map is still the grid (C13); the plan is a guide. Customers see the drawn map with the floor's real outline, not the uploaded picture.
- **Seat types.** The tenant defines seat types such as Open desk, Partition desk and Cubicle, each with a shape (no partition, 1 side, 2 sides, cubicle) that draws on the seat's cell (FR-TA-068i).
- **Prices.** Cinemas price a category of seats, not each seat. Cowos does the same: a price per seat type for each duration, and a price per cabin or meeting-room unit (FR-TA-068j). The booking app shows the prices in the legend and a tab per floor (FR-BA-013a).
- **Editing later.** The tenant can edit the map or replace the plan picture at any time; the live map changes only on publish, and seats with future bookings are protected as before (FR-TA-068k).
- **Automatic suggestion.** Cowos proposing seats from the picture is recorded but not scheduled, because its accuracy on real plans is untested (FR-TA-068l, 13 OQ43).

This replaces two earlier positions: the design doc's "not in this version: a floor-plan photo behind the grid", and the Phase 3 row "floor photo behind the seat map grid".

Affects: 04 SRS (FR-TA-068g to FR-TA-068l, FR-BA-013a), 05 stories (US-4.10, US-4.11, US-6.5), 09 data model, 12 release plan §2, 13 (R34, OQ43–OQ45), design doc "Seat maps".

### C16. Renewals and "booking ending" alerts (added 2026-11-11, v0.5)

**Request (2026-11-11):** add renewal, cancellation and reschedule to the booking app. A customer who booked a cabin for a month should be told the booking is ending on a date and asked to renew, and the same for hourly and daily bookings.

**What the pack now says:** cancellation and reschedule were already in the pack at launch (FR-BA-031, FR-BA-032, FR-TA-080, FR-TA-094, FR-TA-095) and are unchanged. New in v0.5:
- **Ending alerts** for every duration, on timings the tenant can change (defaults: monthly 7 and 2 days before the end, weekly 2 days, daily 3 hours, hourly 15 minutes) (FR-BA-039, FR-TA-084).
- **Renew or extend** from the alert or the booking detail: the same seat or unit when it is free, at the current price, paid through the normal checkout (FR-BA-038). For weekly and monthly bookings the seat or unit is held for the current customer until a deadline the tenant sets, default 2 days before the end (decided in CEO review 4). Staff can do the same for a customer (FR-TA-095a).

Not the same as memberships (Phase 2, FR-TA-151 to FR-TA-154), which bill a plan automatically each cycle. A renewal here is a new booking the customer chooses to make.

Affects: 04 SRS (FR-BA-038, FR-BA-039, FR-TA-084, FR-TA-095a), 05 stories (US-8.5, US-8.6), 09 data model, 13 (OQ46).

### C17. Amenities the tenant configures (added 2026-11-11, v0.5)

**Request (2026-11-11):** show features such as pantry, Wi-Fi, fridge, coffee and oven in the booking app; the tenant configures them in the Tenant Admin.

**What the pack now says:** the pack already had amenities on locations and products (FR-TA-060, FR-TA-061) and on the product page (FR-BA-011), but no screen to manage the list. v0.5 adds an amenity list per location with a starter set and the tenant's own entries (FR-TA-072), and shows it on the booking app home and product pages (FR-BA-047).

Affects: 04 SRS (FR-TA-072, FR-BA-047), 05 stories (US-4.12), 09 data model.

### C18. Validating a booking at the desk: QR or code, inside a check-in window (added 2026-11-11, v0.5)

**Request (2026-11-11):** the customer has a QR and an OTP in the booking app, and the tenant validates it in the Tenant Admin. The customer can validate only 5 or 10 minutes before the booking time; the time is configurable in the Tenant Admin.

**What the pack now says:** QR check-in was already in the pack (FR-TA-096, FR-BA-037). New in v0.5:
- **A 6-digit check-in code** on the pass, typed by staff when scanning isn't possible (FR-BA-046, FR-TA-096a).
- **A check-in window** the tenant sets per location: check-in opens N minutes before the start (default 10). Earlier attempts are refused with the time it opens (FR-TA-083).
- **A manual override** for an Owner or Manager, with a reason, for customers who arrive early (FR-TA-096a).

Affects: 04 SRS (FR-TA-083, FR-TA-096a, FR-BA-046), 05 stories (US-8.7), 09 data model, 13 (R36, OQ47).

### C19. Launch offer: Cowos gives the first 3 to 5 tenants a free period through offer codes (added 2026-11-11, v0.5)

**Request (2026-11-11):** for the first three or five tenants Cowos is free; Cowos gives them promo codes.

**What the pack now says:** SSPL creates offer codes in the Super Admin (discount %, months, number of tenants, last date to redeem) and a tenant enters one on the Billing page after signing in; there is no code field on the public sign-up form (FR-SA-059, FR-TA-183; eng review 6). A 100% code makes the server invoices ₹0 for the covered months. These tenants use the same sign-up, plans and tenant states as everyone else, so the pilot machinery removed in v0.3 does not come back.

This changes the v0.3 statement "no free pilot programme": there is still no separate pilot programme, but SSPL can now waive its charges for chosen tenants with a code. It does not change "0% commission" (C1) or the tenant's own promo codes for its customers (FR-TA-150). The free duration is set on each code (decided 2026-11-11). Decided in CEO review 4: a tenant on a 100% code needs no payment method until the free period ends, and a code can carry a monthly message credit so the offer is free for messages too. The number of tenants and the credit amount are values SSPL enters on the code (13 OQ48).

Affects: 04 SRS (FR-SA-059, FR-TA-183), 05 stories (US-10.9), 09 data model, 13 (OQ48), 14 build plan weeks 17–18.

---

## 3. Decisions you need to make

The full list with owners and deadlines is in [13-risks-open-questions-decisions.md](13-risks-open-questions-decisions.md). D1 and D2 are decided. D3–D6 are still Proposed and block engineering:

| ID | Decision | My recommendation | Needed by |
|----|----------|-------------------|-----------|
| D1 | Pricing model | **Decided 2026-11-02, revised 2026-11-04 (revision 2 the same day):** the tenant chooses a Fixed plan (flat monthly fee, unlimited server usage, messages within a fair-use allowance) or a Usage plan (metered server usage and notifications in slabs SSPL configures); every feature on both; 0% commission (C1) | Done |
| D2 | Hosting model | **Decided 2026-11-04:** shared multi-tenant on one app server + managed PostgreSQL + S3/CloudFront, hybrid-ready (C2, Rule #11) | Done |
| D3 | Payment model | Tenant's own Razorpay via OAuth; no Route (C4) | Before payments sprint; start Razorpay Partner application now (takes weeks) |
| D4 | Domains | `cowos.in` + separate `cowos.site` (C5) | Before sprint 0 (buy domains) |
| D5 | Badge text | "Powered by Cowos" (C6) | Before theme design |
| D6 | Multi-tenancy DB strategy | Shared schema + `tenant_id` + RLS (see 08) | Before sprint 0 |

---

## 4. Pricing and billing (decided 2026-11-04)

The binding rules are Pricing and Billing Rules #1–#13 in `docs/designs/cowos-platform-v1.md`. BRD §7 and the release plan (12) carry them as requirements. The free pilot programme of 2026-11-02 is removed. Summary of the charging rules:

| Topic | Rule |
|-------|------|
| Plans (Rule #12) | Fixed plan: one flat monthly fee set by SSPL on each rate card version; unlimited server usage, not billed; messages included up to a monthly allowance per channel, then paid from the wallet at Usage plan prices; fee billed in arrears, prorated by day only in the first and last month, not charged while Suspended or after cancellation. Usage plan: no fee; the meters, slabs and wallet below apply. The Owner chooses at activation and can switch at any time, effective the 1st of the next month. |
| Meters (Rule #3) | Server request units: 1 unit = 1,000 requests served for the tenant (CloudFront requests on its website and booking-app hosts, plus API requests resolved to it). WAF-blocked requests and SSPL's own health checks and monitoring are not counted. Server storage units: 1 unit = 1 GB-month of the tenant's media and files, averaged from daily snapshots. Notification units: 1 unit = 1 message, metered separately for OTP SMS, other SMS, WhatsApp and email. A WhatsApp message followed by its SMS fallback counts once on each meter. |
| Slabs (Rule #4) | Each meter has its own telescopic slab table in the Super Admin rate card. A slab price can be ₹0. Each rate card version takes effect on the 1st of a month; invoices and wallet debits use the version in force when the usage happened. Counts reset at the start of each calendar month in IST. |
| Server invoice (Rule #5) | Monthly in arrears, calendar month in IST, issued on the 1st for the previous month. Fixed plan: one plan line. Usage plan: one line per server meter with units and slab breakdown. Plus GST. Due in 10 days. Razorpay payment link, or optional UPI Autopay or e-mandate (₹5,000 per-tenant maximum, RBI pre-debit notice). Usage plan billing pages show units per meter, the slab breakdown and a month-to-date estimate, updated daily. |
| Notification wallet (Rule #6) | Prepaid. Each message is debited at its channel's slab price for the month-to-date count, in one atomic step with the count (eng R4). At zero balance, SMS and WhatsApp fall back to email. OTPs and emails continue down to a −₹500 floor. What stops at the floor is open (13 OQ23). The wallet page shows the month-to-date count and current slab per channel. |
| Trial (Rule #7) | 14 days. Publish on a platform subdomain only. No plan fee and server usage is not billed. Notifications need wallet balance from day one. The tenant becomes Active when it adds a payment method and chooses a plan. |
| SSPL cost view (Rule #10) | Super Admin shows SSPL's actual AWS and messaging-provider cost (AWS Data Exports, CUR 2.0, from production) next to billed revenue per month, so SSPL can set slab prices. CUR is not used to bill tenants. |

Other rules: tenants go live after the production entry gate passes, with 10 tenants live at launch (Rule #1, 12 §4); unpaid server invoices follow BRD §7.3 from launch (Rule #8); tenants sign up self-serve with the onboarding wizard from launch, and the "Book a demo" form stays as a lead channel (Rule #9); one app server hosts the platform (Rule #11, C2). The Fixed plan fee, its allowances and the slab prices are set by the product owner before launch (13 OQ35, OQ21). Seat maps (Rule #13) ship at launch (C13).

---

## 5. Release phases

The release plan (12 §1) has the detail. Requirement tables use these phase codes:

| Code | Phase | Indicative timing | What ships |
|------|-------|-------------------|------------|
| P1 | Phase 1 (MVP launch) | Weeks 1–20 (small-team build plan, 14) [estimated] | Everything needed for tenants to sign up, publish, take paid bookings and pay SSPL: self-serve sign-up with the 14-day trial and the onboarding wizard, settings screens with the owner checklist, builder with 2 themes, seat maps with seat picking (C13), booking app, payments and invoices, front desk, CRM, notifications with the slab-priced wallet, custom domains, the Fixed and Usage plans, metering, the rate card, server invoices, collection and dunning, and Super Admin with Demo requests, the tenant scorecard, the dunning dashboard, the business dashboard and the SSPL cost view. Themes 3–4 follow about 6 weeks after launch (CEO-S3). |
| P2 | Phase 2 | Months 7–10 | "Run the space on Cowos": memberships, widget, PWA. |
| P3 | Phase 3 | Months 11–18 | "Scale and integrate". |

---

## 6. Document index

| # | Document | What's in it |
|---|----------|--------------|
| 01 | [Product Vision & Scope](01-product-vision-and-scope.md) | Problem, goals, users, value proposition, scope, assumptions |
| 02 | [Personas, Roles & Permissions](02-personas-roles-permissions.md) | Personas, role definitions, permissions matrix |
| 03 | [Business Requirements (BRD)](03-business-requirements.md) | Business objectives, rules, pricing, billing lifecycle, builder approach |
| 04 | [Software Requirements (SRS)](04-software-requirements-specification.md) | Numbered functional requirements per application, MoSCoW |
| 05 | [User Stories](05-user-stories.md) | Epics and stories with Given/When/Then |
| 06 | [Key User Flows](06-user-flows.md) | Onboarding to publish, integration, booking, refund, suspension |
| 07 | [Non-Functional Requirements](07-non-functional-requirements.md) | Isolation, security, performance, availability, SEO, DPDP |
| 08 | [High-Level Architecture](08-architecture.md) | Components, tenancy, domains/SSL, storage, jobs, CI/CD |
| 09 | [Data Model](09-data-model.md) | Entities, relationships, key constraints |
| 10 | [API Modules](10-api-modules.md) | Modules and key endpoints |
| 11 | [Integrations](11-integrations.md) | Payments, SMS, WhatsApp, email, maps, calendar |
| 12 | [Release Plan](12-release-plan.md) | MVP, Phase 2, Phase 3 |
| 13 | [Risks, Open Questions & Decisions](13-risks-open-questions-decisions.md) | Risk register, open questions, decision log |
| 14 | [Build Plan](14-build-plan.md) | Build order of the apps, weekly sprints with their user stories, team, outside dependencies, cut list (small team using Claude Code) |
