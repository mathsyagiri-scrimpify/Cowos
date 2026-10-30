# 00 · Start Here: Questions, Assumptions, and What I'd Change

> Cowos requirements pack · Version 0.1 (draft) · Owner: SSPL Product · Date: 2026-10-30

This pack has 13 deliverables (files `01` to `13`). Read this file first. It holds three things:

1. Clarifying questions, each with the assumption the rest of the pack uses if you don't answer.
2. Changes I recommend to the original brief, with the reasoning. Several are significant.
3. The decisions you need to make before engineering starts.

---

## 1. Clarifying questions and default assumptions

| # | Question | Assumption used in this pack (if you don't answer) |
|---|----------|------------------------------------------------------|
| Q1 | Who is the beachhead customer? Metro operators (Bengaluru, Hyderabad, Pune) or Tier-2/3 spaces? | Small and mid-size independent spaces (1–3 locations, 20–300 seats) in Tier-1 and Tier-2 Indian cities, starting with Telangana and Andhra Pradesh. Price sensitive, run on WhatsApp + Excel today. |
| Q2 | Is the product India-only for v1? | Yes. INR, GST, Indian phone numbers, IST, Razorpay, DLT-registered SMS. Data designed so currency, tax and timezone are per tenant, so other markets can be added later. |
| Q3 | Is "Powered by SSPL" the final badge text? Customers will not know who SSPL is. | Badge reads **"Powered by Cowos"** (optionally "Cowos by SSPL") and links to the Cowos marketing site with a referral code. See Change C6. |
| Q4 | Does SSPL want to be in the money flow for customer bookings? | No. Customer money goes straight to the tenant's own payment gateway account. SSPL never holds customer funds. See Change C4. |
| Q5 | What does "charge only for server cost" mean commercially? Is SSPL willing to run at zero margin? | No. SSPL charges a flat monthly subscription that covers hosting plus platform costs, and takes **0% commission on bookings**: Cowos never takes a share of the tenant's booking revenue. What the tenant charges its own customers (prices, discounts, promo codes, fees) is the tenant's decision, not Cowos's. See Change C1. |
| Q6 | How many tenants do you expect in year 1 and year 3? | 50–150 tenants in year 1, 500–1,500 by year 3. Average 15,000 customer-facing page views and 300 bookings per tenant per month. |
| Q7 | Do tenants have multiple locations (centres)? | Data model supports many locations per tenant from day 1. The MVP admin UI is tuned for 1 location; multi-location UI lands in Phase 2. |
| Q8 | Should private cabins and dedicated desks be instantly bookable online? | No, not by default. These are sales-led deals (tour, quote, agreement, deposit). Cowos supports per-product **booking modes**: Instant Book, Request to Book, and Enquire / Book a Tour. See Change C7. |
| Q9 | Are customers' accounts shared across tenants? (One login for all Cowos-powered spaces?) | No. Each customer account belongs to one tenant. This keeps the white-label promise and makes the tenant the clear data owner under the DPDP Act. |
| Q10 | Do you need native mobile apps in v1? | No. Responsive web in Phase 1, installable PWA in Phase 2. Per-tenant native apps are not recommended (see Change C9). |
| Q11 | Is the Super Admin "publishing approval" a manual review of every publish? | Only the tenant's **first** publish (plus any custom-domain change) needs SSPL approval, after a KYC check. Later republishes go live immediately and are audited. See Change C8. |
| Q12 | What team will build this, and what is the target MVP date? | 2 backend (.NET), 2 frontend (React), 1 full-stack/DevOps, 1 QA, 1 designer (part-time), 1 PM. MVP in ~5–6 months. |
| Q13 | Do you have the domains? | SSPL owns or will buy `cowos.in` (marketing + admin) and a **separate** domain for tenant sites, for example `cowos.site`. See Change C5. |
| Q14 | Language support? | English only in Phase 1. Content model stores text so Telugu/Hindi can be added in Phase 3. |
| Q15 | Is a free trial allowed? | Yes. 14-day trial, no card needed, publish allowed on a platform subdomain only, with a "Trial" ribbon in admin (not on the public site). |
| Q16 | Who handles customer support for end customers (members)? | The tenant. SSPL supports tenants only. The booking app shows the tenant's contact details, never SSPL's. |

---

## 2. Changes I recommend to the brief

Each change lists the original idea, the problem, and the better alternative. All later documents assume these changes. If you reject one, the affected sections are listed so you know what to rework.

### C1. Pricing: "we charge only the server cost" → flat plans + 0% commission

**Original:** Charge tenants only for their EC2 cost, shown with a transparent cost breakdown.

**Problems:**
- With a shared platform (C2), the real AWS cost per small tenant is roughly ₹100–₹400/month. Pass-through pricing at that level cannot fund support, development, SMS/WhatsApp, security, or sales. The business would lose money on every tenant.
- With one EC2 per tenant, the real cost is ₹2,500–₹4,000/month per tenant, which is too expensive for the target customer.
- Calling a price "server cost" when it contains margin is misleading. Tenants will compare it to AWS's public price list. That damages the trust the whole product is built on.
- Variable costs (SMS, WhatsApp, email, storage beyond limits) are real and change per tenant. They need their own line.

**Better:** Keep the promise that matters to tenants, and make it the headline:

> **0% commission on every booking. Your customers pay you directly. You pay Cowos a flat monthly fee.**

**What "0% commission" means (and doesn't mean):**
- It is a promise from **Cowos to the tenant**: Cowos takes no percentage or per-booking fee from the tenant's booking revenue. The tenant's only charges from Cowos are the flat plan fee and messaging usage.
- It says **nothing about what the tenant charges its customers.** Prices, member rates, promo codes, discounts, convenience fees, deposits and whether to pass gateway fees on to customers are all set by the tenant in the Tenant Admin. Cowos never adds its own fee to a customer's bill.
- Payment gateway charges (Razorpay's MDR) are between the tenant and Razorpay. Cowos doesn't mark them up.

- 3 flat plans (Starter / Growth / Pro) priced by locations and bookable resources, not bookings or revenue.
- Messaging (SMS / WhatsApp) billed at cost from a prepaid wallet, shown line by line. This is where "transparent cost breakdown" belongs.
- A pricing page panel "Where your fee goes" showing an honest allocation (hosting and CDN, backups, SSL, messaging infrastructure, support, product updates). It is an allocation, labelled as such, not a fake per-server invoice.

Indicative plans (validate with 10 customer interviews before locking):

| Plan | Who | Indicative price (excl. GST) | Limits |
|------|-----|------------------|--------|
| Starter | Single small space | ₹999/month or ₹9,990/year | 1 location, 25 bookable resources, platform subdomain or 1 custom domain, tenant promo codes |
| Growth | Established single/dual centre | ₹2,499/month or ₹24,990/year | 3 locations, 100 resources, memberships, embeddable widget |
| Pro | Multi-centre operator | ₹5,999/month or ₹59,990/year | 10 locations, unlimited resources, staff roles, API, priority support |
| Dedicated (Phase 3) | Large operator needing isolation | Quote (AWS cost + fixed ops fee, shown separately) | Dedicated database and compute |

Affects: 01 Vision, 03 BRD (pricing), 04 SRS (FR-PB-*), 12 Release plan.

### C2. Hosting: one EC2 per tenant → shared multi-tenant platform (hybrid later)

**Original:** One EC2 instance per tenant.

**Problem:** Fine for 5 tenants, painful at 50, unworkable at 500. Every deploy, OS patch, .NET security update, certificate, log pipeline and backup is multiplied by the tenant count. Most tenants sit at <2% CPU, so you pay for idle servers.

Comparison (rough ap-south-1 Mumbai on-demand figures, Oct 2026; validate with the AWS Pricing Calculator):

| Factor | A. EC2 per tenant (silo) | B. Shared multi-tenant (pool) | C. Hybrid (pool default, silo for large) |
|--------|--------------------------|-------------------------------|------------------------------------------|
| Infra cost per small tenant / month | ~₹2,500–₹4,000 (t4g.small + EBS + public IPv4 + DB + backups + logs) | ~₹100–₹400 at 200+ tenants (shared ECS/Fargate + RDS Multi-AZ + CloudFront) | Pool tenants ₹100–₹400; silo tenants priced at real cost |
| Fixed base cost | Low at start, grows linearly | ~₹25,000–₹45,000/month base regardless of tenant count | Same as B, plus silo stacks as sold |
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

SSPL's own platform billing (tenant pays Cowos) uses SSPL's own Razorpay account with Subscriptions (UPI Autopay / e-mandate / card).

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

**Apex domain trap (India specific):** Most Indian tenants buy domains from GoDaddy, Hostinger or BigRock, whose DNS does not support CNAME/ALIAS at the apex (`acmecowork.com`). CloudFront has no fixed IP by default. Recommended handling: the site runs on `www`, and the tenant sets up the registrar's free "domain forwarding" from apex to `www` (we show step-by-step guides per registrar). Option for later: offer static anycast IPs for apex A records (CloudFront Anycast Static IPs or Global Accelerator) on Pro plans if demand justifies the cost.

Affects: 06 Flows, 08 Architecture, 09 Data model (Domain).

### C6. Badge: "Powered by SSPL" → "Powered by Cowos"

A badge is a growth channel (every tenant site advertises the product). Visitors searching "SSPL" will find nothing useful, and SSPL is a generic acronym shared by other companies. Use "Powered by Cowos", link to `cowos.in/?ref=<tenant-slug>` for attribution, and keep "a product of SSPL" on the Cowos marketing site. The badge is mandatory on all plans in v1. Make it small, accessible, theme-aware, and not removable through custom CSS (rendered by the platform, checked at publish).

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

The full list with owners and deadlines is in [13-risks-open-questions-decisions.md](13-risks-open-questions-decisions.md). The six that block engineering:

| ID | Decision | My recommendation | Needed by |
|----|----------|-------------------|-----------|
| D1 | Pricing model | Flat plans + 0% commission + messaging at cost (C1) | Before marketing site copy |
| D2 | Hosting model | Shared multi-tenant, hybrid-ready (C2) | Before architecture sprint 0 |
| D3 | Payment model | Tenant's own Razorpay via OAuth; no Route (C4) | Before payments sprint; start Razorpay Partner application now (takes weeks) |
| D4 | Domains | `cowos.in` + separate `cowos.site` (C5) | Before sprint 0 (buy domains) |
| D5 | Badge text | "Powered by Cowos" (C6) | Before theme design |
| D6 | Multi-tenancy DB strategy | Shared schema + `tenant_id` + RLS (see 08) | Before sprint 0 |

---

## 4. Document index

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
