# 00 · Start Here: Questions, Assumptions, and What I'd Change

> Cowos requirements pack · Version 0.2 (draft) · Owner: SSPL Product · Date: 2026-11-03

This pack has 13 deliverables (files `01` to `13`). Read this file first. It holds five things:

1. Clarifying questions, each with the assumption the rest of the pack uses if you don't answer.
2. Changes I recommend to the original brief, with the reasoning. Several are significant.
3. The decisions you need to make before engineering starts.
4. The pilot programme (decided 2026-11-02).
5. The release phases, including the Billing release (P1-B).

v0.2 applies the approved design doc `docs/designs/cowos-platform-v1.md` (Premises, Pilot Rules #1–#10, design decisions 1A–33A, engineering and CEO review decisions).

---

## 1. Clarifying questions and default assumptions

| # | Question | Assumption used in this pack (if you don't answer) |
|---|----------|------------------------------------------------------|
| Q1 | Who is the beachhead customer? Metro operators (Bengaluru, Hyderabad, Pune) or Tier-2/3 spaces? | Small and mid-size independent spaces (1–3 locations, 20–300 seats) in Tier-1 and Tier-2 Indian cities. Price sensitive, run on WhatsApp + Excel today. Beachhead: independent 1–3 location spaces in one city with a meaningful share of bookings from aggregators. Hyderabad was suggested; the city is not confirmed (13 OQ16). The first 20 owners there are pilots (§4). |
| Q2 | Is the product India-only for v1? | Yes. INR, GST, Indian phone numbers, IST, Razorpay, DLT-registered SMS. Data designed so currency, tax and timezone are per tenant, so other markets can be added later. |
| Q3 | Is "Powered by SSPL" the final badge text? Customers will not know who SSPL is. | Badge reads **"Powered by Cowos"** (optionally "Cowos by SSPL") and links to the Cowos marketing site with a referral code. See Change C6. |
| Q4 | Does SSPL want to be in the money flow for customer bookings? | No. Customer money goes straight to the tenant's own payment gateway account. SSPL never holds customer funds. See Change C4. |
| Q5 | What does "charge only for server cost" mean commercially? Is SSPL willing to run at zero margin? | **Answered (D1, 2026-11-02): yes.** SSPL charges tenants only their share of the real AWS hosting cost, plus messaging at cost. There is no subscription, plan fee or commission, so SSPL earns no margin. Cowos takes **0% commission on bookings**: it never takes a share of the tenant's booking revenue. What the tenant charges its own customers (prices, discounts, promo codes, fees) is the tenant's decision, not Cowos's. See Change C1. |
| Q6 | How many tenants do you expect in year 1 and year 3? | 50–150 tenants in year 1, 500–1,500 by year 3. Average 15,000 customer-facing page views and 300 bookings per tenant per month. |
| Q7 | Do tenants have multiple locations (centres)? | Data model supports many locations per tenant from day 1. The MVP admin UI is tuned for 1 location; multi-location UI lands in Phase 2. |
| Q8 | Should private cabins and dedicated desks be instantly bookable online? | No, not by default. These are sales-led deals (tour, quote, agreement, deposit). Cowos supports per-product **booking modes**: Instant Book, Request to Book, and Enquire / Book a Tour. See Change C7. |
| Q9 | Are customers' accounts shared across tenants? (One login for all Cowos-powered spaces?) | No. Each customer account belongs to one tenant. This keeps the white-label promise and makes the tenant the clear data owner under the DPDP Act. |
| Q10 | Do you need native mobile apps in v1? | No. Responsive web in Phase 1, installable PWA in Phase 2. Per-tenant native apps are not recommended (see Change C9). |
| Q11 | Is the Super Admin "publishing approval" a manual review of every publish? | Only the tenant's **first** publish (plus any custom-domain change) needs SSPL approval, after a KYC check. Later republishes go live immediately and are audited. See Change C8. |
| Q12 | What team will build this, and what is the target MVP date? | 2 backend (.NET), 2 frontend (React), 1 full-stack/DevOps, 1 QA, 1 designer (part-time), 1 PM. MVP in ~5–6 months. |
| Q13 | Do you have the domains? | SSPL owns or will buy `cowos.in` (marketing + admin) and a **separate** domain for tenant sites, for example `cowos.site`. See Change C5. |
| Q14 | Language support? | English only in Phase 1. Content model stores text so Telugu/Hindi can be added in Phase 3. |
| Q15 | Is a free trial allowed? | Yes, for non-pilot tenants from the Billing release (P1-B, CEO-S1). 14-day trial, no card needed, publish allowed on a platform subdomain only, with a "Trial" ribbon in admin (not on the public site). Pilots never have a trial (Pilot Rules #4). |
| Q16 | Who handles customer support for end customers (members)? | The tenant. SSPL supports tenants only. The booking app shows the tenant's contact details, never SSPL's. |

---

## 2. Changes I recommend to the brief

Each change lists the original idea, the problem, and the better alternative. All later documents assume these changes. If you reject one, the affected sections are listed so you know what to rework.

### C1. Pricing: hosting cost share + messaging at cost + 0% commission (decided, D1)

**Original:** Charge tenants only for their EC2 cost, shown with a transparent cost breakdown.

**v0.1 recommendation (superseded):** flat monthly plans, because a pass-through charge of ₹100–₹400 per tenant per month cannot fund support, development or sales. One EC2 per tenant was rejected as too expensive (C2).

**Decision (D1, product owner, 2026-11-02; Premise 5):** SSPL charges tenants only their share of the real AWS hosting cost, plus messaging at cost. There is no subscription, plan fee, plan tier, plan limit or commission. Every tenant gets every feature (Pilot Rules #2). The headline (28A):

> **0% commission. You pay only your share of our real AWS bill.**

How the charge works (Pilot Rules #10; full rules in BRD §7):
- **Cost pool:** the production AWS account's monthly bill, read from AWS Data Exports (CUR 2.0), net of AWS credits. SSPL pays for staging, dev, CI, the marketing site and Super Admin.
- **Split:** equally across counted tenants (Active, PastDue or Restricted for any part of the month, pilots included), prorated by days live. SSPL pays the shares of its test and demo tenants and of pilots in their free year. SSPL absorbs unpaid shares; they are never passed to other tenants.
- **Invoice:** monthly in arrears on the 10th, once AWS marks the bill final; GST added; due in 10 days. Later AWS credits, refunds or support fees appear as an adjustment line on the next invoice. Paid by Razorpay payment link, or by optional UPI Autopay or e-mandate (₹5,000 per-tenant maximum, RBI pre-debit notice).
- **Messaging:** prepaid wallet at provider cost, with no free allowance.
- **Transparency:** the billing page shows the AWS bill breakdown behind the charge (5A). The pricing page shows last month's actual per-space share (28A).
- **Indicative charge:** about ₹1,250–2,250 per tenant per month at 20 tenants, falling to about ₹100–400 at 200+ tenants [estimated from the C2 table].
- **Timing:** automated billing ships in the Billing release (P1-B, §5). Pilots pay nothing for their first 12 months (§4).

**What "0% commission" means (and doesn't mean):**
- It is a promise from **Cowos to the tenant**: Cowos takes no percentage or per-booking fee from the tenant's booking revenue. The tenant's only charges from Cowos are its hosting cost share and messaging usage.
- It says **nothing about what the tenant charges its customers.** Prices, member rates, promo codes, discounts, convenience fees, deposits and whether to pass gateway fees on to customers are all set by the tenant in the Tenant Admin. Cowos never adds its own fee to a customer's bill.
- Payment gateway charges (Razorpay's MDR) are between the tenant and Razorpay. Cowos doesn't mark them up.

**Accepted risk:** with no margin on cost, tenant payments do not fund SSPL's support, onboarding or development (Premise 5; 13 R1). Goal G3 is cost recovery, not margin (CEO-A5).

Affects: 01 Vision, 03 BRD §7, 04 SRS (FR-MW-002/003/004, FR-PL-060/061/062, FR-TA-175/176, FR-SA-020/021), 08 Architecture, 09 Data model, 12 Release plan, 13 Decisions.

### C2. Hosting: one EC2 per tenant → shared multi-tenant platform (hybrid later)

**Original:** One EC2 instance per tenant.

**Problem:** Fine for 5 tenants, painful at 50, unworkable at 500. Every deploy, OS patch, .NET security update, certificate, log pipeline and backup is multiplied by the tenant count. Most tenants sit at <2% CPU, so you pay for idle servers.

Comparison (rough ap-south-1 Mumbai on-demand figures, Oct 2026; validate with the AWS Pricing Calculator):

| Factor | A. EC2 per tenant (silo) | B. Shared multi-tenant (pool) | C. Hybrid (pool default, silo for large) |
|--------|--------------------------|-------------------------------|------------------------------------------|
| Infra cost per small tenant / month | ~₹2,500–₹4,000 (t4g.small + EBS + public IPv4 + DB + backups + logs) | ~₹100–₹400 at 200+ tenants (shared ECS/Fargate + RDS Multi-AZ + CloudFront) | Pool tenants share the pool cost (D1); how a silo cell is charged is open (13 OQ20) |
| Fixed base cost | Low at start, grows linearly | ~₹25,000–₹45,000/month base regardless of tenant count | Same as B, plus each silo stack |
| Scalability | Per tenant, manual or per-tenant ASG | Horizontal autoscaling of shared services | Both |
| Isolation | Strong (separate VM and DB) | Logical: TenantId + PostgreSQL Row-Level Security + per-tenant encryption context | Strong for silo tenants, logical for the rest |
| Security | Smaller blast radius, but 500 servers to patch means patch lag, which is its own risk | One hardened fleet, patched once; needs strict tenant-isolation tests | Same as B plus silo |
| Deployment | N deploys per release; version drift across tenants | One deploy per release (blue/green) | One pool deploy + silo deploy wave |
| Operations | Heavy: N sets of monitoring, backups, certs | Light: one set, per-tenant dashboards by tag | Medium |
| Noisy neighbour risk | None | Exists; mitigated by rate limits per tenant, static sites on CDN | Large tenants move to silo |
| Time to onboard a tenant | Minutes to provision (Terraform per tenant) | Seconds (a database row) | Seconds; silo on request |

**Recommendation for v1: B (shared multi-tenant), architected so C is possible later.** Concretely:
- Tenant websites are **pre-rendered static files** on S3 + CloudFront at publish time. They cost almost nothing to serve and stay up even if the API is down.
- The booking app is **one React build** served to every tenant host, styled at runtime from the tenant's theme.
- One ASP.NET Core API (modular monolith) on ECS Fargate, one PostgreSQL (RDS Multi-AZ), shared schema with `tenant_id` and Row-Level Security.
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

SSPL's own platform billing (tenant pays its hosting cost share to SSPL) uses SSPL's own Razorpay account: a payment link per monthly invoice, or optional UPI Autopay / e-mandate with a ₹5,000 per-tenant maximum and the RBI pre-debit notice (Pilot Rules #10). It ships in the Billing release (P1-B).

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
| Interactive floor-plan seat picker | Custom canvas editor for tenants plus seat map renderer; months of work | Phase 2 as "upload floor image + drop pins"; full editor Phase 3 if needed. MVP: pick seat from a list or "any available" |
| Two-way Google/Outlook calendar sync | OAuth per user, webhook channels, conflict resolution | MVP: `.ics` attachment + "Add to Google Calendar" link. Phase 3: one-way push for meeting rooms |
| Free-form drag-and-drop builder | See section 4 of the BRD; large build and support cost | Not planned; section-based builder instead |
| Per-tenant WhatsApp Business numbers | Meta Business verification for each tenant | MVP: platform WhatsApp number with tenant name in approved templates. Phase 2: tenant's own number through embedded signup |
| GST e-invoicing (IRN) | GSP integration and per-tenant credentials | Phase 3 (needed by tenants over the e-invoice turnover threshold) |

---

## 3. Decisions you need to make

The full list with owners and deadlines is in [13-risks-open-questions-decisions.md](13-risks-open-questions-decisions.md). D1 is decided. D2–D6 are still Proposed and block engineering:

| ID | Decision | My recommendation | Needed by |
|----|----------|-------------------|-----------|
| D1 | Pricing model | **Decided 2026-11-02:** hosting cost share + messaging at cost, no subscription or plan fee, 0% commission (C1) | Done |
| D2 | Hosting model | Shared multi-tenant, hybrid-ready (C2) | Before architecture sprint 0 |
| D3 | Payment model | Tenant's own Razorpay via OAuth; no Route (C4) | Before payments sprint; start Razorpay Partner application now (takes weeks) |
| D4 | Domains | `cowos.in` + separate `cowos.site` (C5) | Before sprint 0 (buy domains) |
| D5 | Badge text | "Powered by Cowos" (C6) | Before theme design |
| D6 | Multi-tenancy DB strategy | Shared schema + `tenant_id` + RLS (see 08) | Before sprint 0 |

---

## 4. Pilot programme (decided 2026-11-02)

The binding rules are Pilot Rules #1–#10 in `docs/designs/cowos-platform-v1.md`. BRD §7 and the release plan (12) carry them as requirements. Summary:

| Topic | Rule |
|-------|------|
| Who | The first 20 owners in the beachhead city (§1 Q1). |
| Offer | 12 months fully free per tenant, from its first live publish on production. Full feature set, plus Phase 2 features as they ship. No other commitments (Pilot Rules #2, #3). |
| Set-up | Super Admin creates the pilot tenant on production directly in Active with a pilot flag, and invites the Owner through the staff invite flow with the Owner role (single use, 72-hour expiry, phone OTP, owner sets password and MFA) (CEO A2). A pre-handover setup grant lets SSPL configure the tenant while it has no Owner, is unpublished and has no live payments; it ends when the Owner accepts, and every write is audited. After handover, tenant-approved support grants apply (R19). The owner does Razorpay KYC and OAuth, GSTIN/PAN, DNS, and accepts the ToS and DPA. One named SSPL onboarding owner runs pilot onboarding, about 15 person-days across weeks 18–30 (Pilot Rules #4, #9). |
| Timing | Weeks 18–21: guided staging preview in Razorpay test mode, no real customers. From week 22: setup on production, unpublished, no live payments, until the production entry gate passes (12 §4). 10 pilots live at the week-24 launch, the other 10 within 6 weeks. If the gate passes after week 24, the 10-pilot target moves to 2 weeks after the gate (Pilot Rules #1). |
| Messaging | SSPL-funded wallet. Fair use: 1,000 SMS + WhatsApp notifications per pilot per month (provisional), then email only and SSPL is alerted. Sign-in OTPs: SSPL alerted at 1,000; at 2,000, phone OTP pauses for that tenant for the rest of the month and customers sign in by email OTP (Pilot Rules #5). |
| Billing | No payment method, trial or dunning during the free year, and no ₹0 invoices. Pilots are counted in the cost pool and SSPL pays their share (Pilot Rules #4, #10). |
| End of the free year | Notices at 60, 30 and 7 days; the tenant adds a payment method with the 60-day notice. Month 13's share is invoiced on the 10th of month 14. A pilot that hasn't paid that first invoice by its due date plus 14 days stays Restricted for 90 days, then follows the normal lifecycle (Pilot Rules #6). |
| Billing validation | Non-pilot sign-up and SSPL's own paid test tenant start at the Billing release. Before the first pilot converts, at least one real monthly hosting charge has been paid and one dunning cycle has run end to end on production (Pilot Rules #7, CEO-S1). |
| Tracking | Super Admin pilot scorecard: paid online bookings per pilot per month, counted in the Asia/Kolkata month the payment was captured; cap usage; expansion gate status (CEO-A4, R20). |
| Expansion gate | Open a second city when 10 pilots each take 10 or more paid online bookings a month for 3 consecutive months (Pilot Rules #8). |
| Cost to SSPL | About ₹4–7.5 lakh hosting for the pilot year including staging, plus up to about ₹1.8 lakh messaging [estimated] (13 R25). |

---

## 5. Release phases

The release plan (12 §1) has the detail. Requirement tables use these phase codes:

| Code | Phase | Indicative timing | What ships |
|------|-------|-------------------|------------|
| P1 | Phase 1 (MVP launch) | Weeks 1–24 | Everything needed for pilots to publish and take paid bookings: settings screens with the owner checklist, builder with 2 themes, booking app, payments and invoices, front desk, CRM, notifications with the SSPL-funded pilot wallet, custom domains, Super Admin with the Pilots page (create pilot, pilot requests, scorecard) and manual suspend/reinstate. Tenant states at launch are Active (pilot) and Suspended only. Themes 3–4 follow about 6 weeks after launch (CEO-S3). |
| P1-B | Billing release | Launch + about 4 months [estimated] | Platform billing and self-serve sign-up (CEO-S1, CEO-S2): the CUR billing job and billing tables, collection by payment link and UPI Autopay / e-mandate, dunning with automatic Restricted and Suspended moves, the pricing JSON and "last month's actual", the daily charge estimate, the 14-day trial, non-pilot wallet top-up, public sign-up (FR-MW-005), referral capture (FR-MW-006), slug availability check (FR-MW-012), sign-up and trial metrics (FR-SA-051), the dunning dashboard (FR-SA-023), and the onboarding wizard (FR-TA-001 flow, GSTIN legal-name lookup, Places autocomplete FR-TA-004, booking-only path FR-TA-006, demo inventory FR-TA-008 with its sample publish check). |
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
