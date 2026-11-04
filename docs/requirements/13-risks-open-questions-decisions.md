# 13 · Risks, Open Questions & Decisions

> Cowos requirements pack · v0.3 draft

Note: other pack files cite engineering review decisions from `docs/designs/cowos-platform-v1.md` as "(R1)"–"(R20)". This file writes them "eng R1"–"eng R20" so they are not confused with the risks R1–R30 below. "Pricing Rule #n" refers to the Pricing and Billing Rules #1–#11 in the same design doc.

## 1. Risk register

Likelihood (L) and Impact (I): H / M / L.

| ID | Risk | L | I | Mitigation | Owner |
|----|------|:-:|:-:|------------|-------|
| R1 | **Slab prices don't cover SSPL's cost.** SSPL's revenue depends on the slab prices it sets for server usage and notifications | M | H | No longer an accepted risk (revised 2026-11-04): SSPL sets the slab prices, which may include a margin (Premise 5, D1). The Super Admin cost view shows actual AWS and messaging-provider cost next to billed revenue each month, so SSPL can adjust the rate card (Pricing Rules #4, #10). G3 tracks cost recovery (CEO-A5) | Leadership |
| R2 | **One EC2 per tenant** makes cost per tenant too high and operations unmanageable beyond ~50 tenants | H | H | Shared multi-tenant pool, hybrid-ready cells (00 §C2) | Architecture |
| R3 | **Cross-tenant data leak** in a shared database | M | H | Two-layer isolation (EF filters + forced RLS), isolation test suite gating CI, tenant from host/token only, pen test | Engineering |
| R4 | **Razorpay Partner approval / OAuth access delayed** | M | H | Apply in sprint 0; API-key fallback; offline/Request modes allow launch without gateway | Product |
| R5 | **Being in the money flow** (if Route is chosen) brings refund, chargeback and compliance liability | M | H | Direct-to-tenant settlement; SSPL never holds funds (00 §C4) | Leadership |
| R6 | **Apex domains at Indian registrars** can't CNAME; tenants get stuck on DNS | H | M | `www` canonical + registrar forwarding, provider-specific guides, DNS check UI showing exactly what's wrong, assisted setup via support grant | Product / Support |
| R7 | **Custom domain SSL at scale** (quotas, issuance failures) | M | M | CloudFront SaaS Manager spike in sprint 0; Caddy on-demand TLS fallback; health monitoring + alerts | DevOps |
| R8 | **SEO underperforms** for tenant sites (thin content, duplicate platform subdomains) | M | M | Pre-rendering, structured data, canonical redirects, noindex trials, GBP guidance, content prompts in builder | Product |
| R9 | **DLT / WhatsApp template approvals** slow launch or messages get blocked | M | M | Start in sprint 0; email always available; template library reviewed by provider before submission | Product |
| R10 | **Double bookings or overselling** under concurrency | L | H | Database exclusion constraint and conditional capacity updates; hold expiry; late-payment auto-refund | Engineering |
| R11 | **Webhook gaps** leave paid bookings unconfirmed | M | H | Client verify + webhook (idempotent), hourly and nightly reconciliation, alerting on mismatch | Engineering |
| R12 | **GST invoicing errors** (wrong place of supply, numbering gaps, edited invoices) | M | H | Rules engine reviewed by a CA, immutable invoices, gap-free series, credit notes, test matrix | Product / Finance |
| R13 | **DPDP non-compliance** (consent, rights, breach notification) as the Rules phase in | M | H | Roles defined (tenant fiduciary, SSPL processor), DPA, consent records, DPDP queue, breach runbook, counsel review | Legal |
| R14 | **Fraudulent tenants** use Cowos to run fake booking sites | M | M | First-publish KYC + review, GSTIN/PAN verification, gateway KYC by Razorpay, takedown tooling, blocked words/links | Operations |
| R15 | **Low discovery**: tenants get a site but no new customers, then churn | H | M | Position as conversion + retention tool alongside aggregators; GBP and SEO guidance; badge referral program; share kit (QR posters, WhatsApp links) | Product / Marketing |
| R16 | **Template themes feel generic**; tenants want more control | M | M | High-quality themes from a real designer, strong photo handling, custom pages; collect requests before adding flexibility | Design |
| R17 | **Scope creep toward full coworking management** (access control, accounting, community) delays MVP | H | M | Phase plan with explicit cuts; prioritisation driven by early tenants | PM |
| R18 | **Third-party cookie / iframe restrictions** break the embedded widget | H | M | Subdomain link as default; widget opens login/payment top-level (00 §C10) | Engineering |
| R19 | **Messaging costs** (WhatsApp/SMS) exceed expectations for tenants as wallet spend | M | M | Prepaid wallet with a conditional debit (eng R4), per-channel slab prices and month-to-date counts on the wallet page (Pricing Rule #6), per-event channel toggles, WhatsApp-first for OTP where cheaper, monitoring | Finance |
| R20 | **Suspending sites harms tenants' customers** and creates disputes | M | M | Restricted state keeps existing bookings working; neutral suspended page with tenant contacts (BRD §7.3) | Product |
| R21 | **Node renderer** adds a second runtime to a .NET team | L | L | Small, stateless service; shared theme packages with admin preview; documented ownership | Architecture |
| R22 | **Team capacity**: 5-month MVP is tight for the listed scope. Billing, metering, the rate card, sign-up and the wizard are back in launch scope, adding roughly 3 weeks of 1 BE + 1 FE and about 1.5 weeks of FE [estimated, from the CEO review] | H | H | Cut list ready (12 §5), including the themes 3–4 deferral (CEO-S3); weekly scope review; launch date and the production entry gate stay fixed (Pricing Rule #1) | PM |
| R23 | **"Powered by SSPL" badge** does little for acquisition | M | L | "Powered by Cowos" with referral tracking (00 §C6) | Marketing |
| R24 | **Image licensing / third-party content** on tenant sites | L | M | Terms make tenant responsible; takedown process; stock image library with licensed images in themes | Legal |
| R25 | Removed in v0.3 (the pilot programme and its free year are removed, so SSPL carries no pilot-year cost) | | | | |
| R26 | **Demand is unverified**: the complete suite is built whatever the demand evidence says | M | H | **Accepted risk** (Premise 3). Demo request intake (CEO-A2), tenant scorecard (CEO-A4), and the expansion gate before a second city: 10 tenants at 10+ paid online bookings a month for 3 consecutive months | Leadership |
| R27 | Removed in v0.3 (billing ships at launch; there is no Billing release) | | | | |
| R28 | **Single app server outage** takes down the admin portals, booking app and API for all tenants. Deploys may also cause a few minutes of downtime | M | H | Tenant websites stay up on S3 + CloudFront; automatic instance recovery, an infrastructure-as-code rebuild path and health alarms; managed PostgreSQL with backups and point-in-time recovery (Pricing Rule #11). Availability target and deploy downtime are open (OQ25) | Engineering / DevOps |
| R29 | **Tenants dispute metered bills** they don't understand or don't expect | M | M | Billing page shows units per meter, the slab breakdown, a daily month-to-date estimate and the daily usage behind each invoice; WAF-blocked requests and SSPL monitoring are excluded (Pricing Rules #3, #5); rate card published on the pricing page | Product / Finance |
| R30 | **Bot traffic inflates request units**: crawlers and scrapers that pass WAF count as billable requests | M | M | WAF-blocked requests are excluded (Pricing Rule #3); whether known bots that pass WAF are billed is open (OQ22); daily usage detail lets a tenant see spikes | Engineering / Product |

## 2. Open questions

| ID | Question | Why it matters | Default if unanswered |
|----|----------|----------------|----------------------|
| OQ1 | Final plan prices and limits | Marketing site, billing build | **Resolved by D1 (revised 2026-11-04):** there are no plans or limits. Tenants pay metered server usage and notifications at slab prices SSPL sets (00 §C1). The slab prices themselves are OQ21 |
| OQ2 | Will SSPL add a margin on messaging (handling fee), or pass through at cost? | Transparency message, wallet pricing | **Resolved by D1 (revised 2026-11-04):** notifications are priced per channel in slabs that SSPL sets, which may include a margin; a ₹0 slab gives a free allowance. Paid from a prepaid wallet; OTPs and emails are debited like other messages, down to −₹500 (Pricing Rules #4, #6) |
| OQ3 | Refund policy for SSPL charges | Terms, billing logic | Updated for D1: plan fees no longer exist, so the annual pro-rata option no longer applies. Monthly server invoices: non-refundable |
| OQ4 | Is badge removal ever sold? | Growth vs revenue | Not in v1; revisit with data |
| OQ5 | Is a trial publish to subdomain allowed, or must tenants pay before any publish (original brief)? | Conversion, fraud risk | Trial publish allowed on subdomain, noindex |
| OQ6 | Which SMS and WhatsApp providers? | Sprint 0 DLT/template registration | Shortlist 2, choose on price + delivery SLA in sprint 0 |
| OQ7 | Final domains (`cowos.in`, `cowos.site` or alternatives) | Everything domain-related | As per 00 §C5 |
| OQ8 | Support hours and SLA for tenants | Staffing, review SLA | Mon–Sat 9:00–19:00 IST; first-publish review within 4 working hours |
| OQ9 | Who owns theme design: in-house or agency? | Quality of the first impression | Contract one agency for 4 themes |
| OQ10 | Do launch tenants already exist (warm leads)? | Launch milestone of 10 tenants live (Pricing Rule #1) | Unknown. Default: recruit in the beachhead city (OQ16) from sprint 0, through owner interviews and the "Book a demo" form (CEO-A2) |
| OQ11 | Should customers be able to pay "at the venue" for Instant Book products? | Booking modes, no-show risk | Off by default; tenant can enable per product |
| OQ12 | Assisted onboarding: free, paid, or both? Does a paid setup service still fit D1 ("no subscription, plan fee or commission")? | Revenue, support load | Paid one-time setup (e.g. ₹2,999) as in BRD BR-05, through a tenant-approved support grant (Pricing Rule #9). The v0.1 "free for annual plans" option is removed (no plans, D1) |
| OQ13 | Do we need Telugu/Hindi in the booking app for Phase 1? | Localisation effort | No; English only, strings externalised |
| OQ14 | Data retention period for inactive customer data | DPDP compliance | 3 years after last activity, tenant-configurable within legal bounds |
| OQ15 | GST treatment of SSPL's own service (SAC, place of supply) | Platform invoices | Confirm with CA during sprint 0 |
| OQ16 | Which city is the beachhead? | Tenant recruitment, expansion gate | Hyderabad (suggested, not confirmed) |
| OQ17 | Removed in v0.3 (pilot selection; the pilot programme is removed) | | |
| OQ18 | Removed in v0.3 (pilot messaging cap; the pilot programme is removed) | | |
| OQ19 | Removed in v0.3 (equal or weighted split of the AWS bill; tenants now pay metered slabs, Pricing Rules #3–#5) | | |
| OQ20 | How is a Phase 3 dedicated cell charged under D1? | Large tenants, Phase 3 billing | Not decided. The v0.1 quote of "AWS cost + fixed ops fee" conflicts with D1 |
| OQ21 | What are the slab boundaries and prices for each meter? | Pricing page, rate card, every invoice and wallet debit | The product owner configures them in the rate card before launch (Pricing Rule #4; 12 §4.2) |
| OQ22 | Should requests from known bots and crawlers that pass WAF be billed as request units? | Fair bills, disputes (R29, R30) | Not decided. WAF-blocked requests and SSPL monitoring are already excluded (Pricing Rule #3) |
| OQ23 | When a wallet reaches the −₹500 floor, which messages stop (OTP email, booking confirmations, invoices)? | Customer sign-in, tenant operations | Partly decided 2026-11-04 (R25): sign-in OTP emails always send, cost absorbed by SSPL past the floor; other messages past the floor still open (Pricing Rule #6) |
| OQ24 | How much notice does a tenant get before a new rate card version takes effect? | Trust, terms of service | Not decided. Versions take effect on the 1st of a month (Pricing Rule #4) |
| OQ25 | What availability target does the single app server need to meet, and is a few minutes of downtime per deploy acceptable? | Admin and booking uptime for all tenants (R28) | Not decided (Pricing Rule #11). The availability target in 07 stands until this is answered |
| OQ26 | How are partial units rounded (requests ÷ 1,000; GB-month), and to how many decimals can slab prices be set? | Every invoice and wallet debit | Not decided |
| OQ27 | Does Trial usage count toward a tenant's slab position, and do Trial days count in the storage average? | First invoice after a trial | Not decided. Trial server usage is not billed (Pricing Rule #7) |
| OQ28 | CloudFront logs for the month's last day can arrive late; when does rating on the 1st run, and where do late requests go? | Invoice accuracy and timing | Resolved 2026-11-04 (CEO review 2, C2-A2): rating waits until every day of the month has been counted, so late logs delay the invoice rather than falling into another month |
| OQ29 | Are ₹0 invoices issued, and how are SSPL's own test and demo tenants billed? | Billing runs, GST records | Not decided |
| OQ30 | What counts as stored media and files (rendered site versions, image variants, exports)? | Storage units | Not decided |
| OQ31 | What happens when a server invoice exceeds the ₹5,000 e-mandate maximum? | Collection, dunning | Not decided. A payment link always works (Pricing Rule #5) |
| OQ32 | Do wallet top-ups and debits carry GST, and how is it invoiced? | Wallet, GST returns | Resolved for the UI 2026-11-04 (DR2-11A): wallet gets the typed amount, GST added on top with a GST invoice per top-up; CA to confirm |
| OQ33 | How long are raw CloudFront logs (which hold visitor IP addresses) kept after metering? | DPDP compliance | Not decided; needs counsel |
| OQ34 | Do Super Admin and support-grant requests count as the tenant's request units? | Fair bills | Resolved 2026-11-04 (eng re-review 3, R22): only customer-facing tenant hosts are metered; admin and support traffic is not |

## 3. Decision log

Status: **Proposed** until you approve. Change the status column as you decide. D1 and D2 are Decided; D3–D16 are still Proposed.

| ID | Decision | Options | Recommendation | Status | Needed by |
|----|----------|---------|----------------|--------|-----------|
| D1 | Commercial model | (a) Pass-through server cost (b) Flat plans + 0% commission (c) Commission-based | v0.1 recommended (b). On 2026-11-02 the owner chose (a) as a share of the production AWS bill plus messaging at cost. **Revised 2026-11-04:** tenants pay only metered server usage (request and storage units) and notifications (OTP SMS, other SMS, WhatsApp, email), each priced in slabs that SSPL sets in a rate card; prices may include a margin. No subscription, plan fee or commission (00 §C1, Premise 5, Pricing Rules #3–#7, #10) | **Decided 2026-11-02, revised 2026-11-04** | Done |
| D2 | Hosting model | (a) EC2 per tenant (b) Shared pool (c) Hybrid | **Owner chose (b) on one app server:** one app server (EC2) runs every tenant's API, admin portals, booking app, worker and renderer; managed PostgreSQL (RDS) with backups and point-in-time recovery; tenant websites on S3 + CloudFront. (c) capability stays designed in (00 §C2, Pricing Rule #11) | **Decided 2026-11-04** | Done |
| D3 | Payment model | (a) Razorpay Route (b) Tenant's own account via OAuth (c) Tenant API keys only | (b) with (c) as fallback | Proposed | Sprint 0 (start partner application) |
| D4 | Domain structure | As brief (`booking.<tenant>.cowos.in`) vs separate tenant domain with `/book` path | `<slug>.cowos.site` + `/book`; custom: `www.` + `book.` or `/book` | Proposed | Sprint 0 |
| D5 | Badge text | "Powered by SSPL" vs "Powered by Cowos" | "Powered by Cowos" | Proposed | Before theme design |
| D6 | Multi-tenancy DB | Shared schema + RLS vs schema per tenant vs DB per tenant | Shared + RLS; dedicated DB cells for large tenants later | Proposed | Sprint 0 |
| D7 | Messaging pricing | At cost vs cost + handling fee; OTPs included or charged | Per-channel slab prices set by SSPL in the rate card (a ₹0 slab gives a free allowance); OTPs charged from the wallet like other messages (follows from D1 and Pricing Rules #4, #6, OQ2) | Proposed | Week 6 |
| D8 | Refunds of SSPL charges | Policy options in OQ3 | As in OQ3 default (updated for D1) | Proposed | Before the billing build (12 §3 M4) |
| D9 | Builder approach | Themes + sections vs drag-and-drop | Themes + sections (BRD §6) | Proposed | Sprint 0 |
| D10 | Website rendering | SPA vs SSR at request vs pre-render at publish | Pre-render at publish (static on CDN) | Proposed | Sprint 0 |
| D11 | Mobile strategy | Per-tenant native vs PWA vs single member app | PWA in Phase 2; single app only if data supports | Proposed | Phase 2 planning |
| D12 | First-publish approval | Every publish vs first publish + automated checks | First publish only (00 §C8) | Proposed | Week 8 |
| D13 | Background jobs | Hangfire vs SQS workers vs Quartz | Hangfire on PostgreSQL in v1 | Proposed | Sprint 0 |
| D14 | Auth server | ASP.NET Identity + OpenIddict vs Cognito vs Keycloak vs Duende | Identity + OpenIddict | Proposed | Sprint 0 |
| D15 | Custom-domain edge | CloudFront SaaS Manager vs distribution per tenant vs Caddy edge | SaaS Manager, subject to spike | Proposed | End of sprint 0 |
| D16 | Trial publish | Allowed on subdomain vs pay first | Allowed on subdomain (noindex) | Proposed | Week 8 |

## 4. Alternative product ideas worth considering

These are not in scope, but you asked whether there is a better idea. My view: the core idea is sound (Directoo's model transfers well because coworking has the same aggregator-commission pain), with the changes in 00. Three adjacent bets could make it stronger:

1. **"Bookings-first" wedge instead of "website-first".** Many spaces already have an Instagram page and a Google Business Profile but no booking system. Leading with "meeting-room and day-pass booking with UPI and WhatsApp confirmations, live in 15 minutes" (a booking page + link-in-bio + GBP "Book" button) may convert faster than "build your website". The website becomes the upsell. The architecture in this pack supports both; the choice changes marketing and onboarding order, not the build.
2. **Meeting rooms for non-coworking businesses.** Hotels, cafes with a private room, business centres, co-living spaces, and training institutes rent rooms by the hour and have the same problem. The same product with a "Rooms" template could double the addressable market without new engineering.
3. **An opt-in "Cowos Network" later (Phase 3+).** Once there are a few hundred tenants, an opt-in directory where members of one space can book day passes or rooms at another (with a flat network fee, not a commission; any such fee would need a change to D1) gives tenants discovery and gives Cowos a moat aggregators can't copy easily. Only pursue it if tenant density in a city justifies it, and keep it opt-in so it never contradicts the "you own your customers" promise.
