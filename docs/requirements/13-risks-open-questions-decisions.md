# 13 · Risks, Open Questions & Decisions

> Cowos requirements pack · v0.1 draft

## 1. Risk register

Likelihood (L) and Impact (I): H / M / L.

| ID | Risk | L | I | Mitigation | Owner |
|----|------|:-:|:-:|------------|-------|
| R1 | **Pricing as "server cost only" is unsustainable** and erodes trust once tenants compare it to AWS prices | H | H | Flat plans + 0% commission + messaging at cost; honest "where your fee goes" allocation (00 §C1) | Leadership |
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
| R17 | **Scope creep toward full coworking management** (access control, accounting, community) delays MVP | H | M | Phase plan with explicit cuts; pilot-driven prioritisation | PM |
| R18 | **Third-party cookie / iframe restrictions** break the embedded widget | H | M | Subdomain link as default; widget opens login/payment top-level (00 §C10) | Engineering |
| R19 | **Messaging costs** (WhatsApp/SMS) exceed expectations and squeeze margin | M | M | Wallet-based pass-through, per-event channel toggles, WhatsApp-first for OTP where cheaper, monitoring | Finance |
| R20 | **Suspending sites harms tenants' customers** and creates disputes | M | M | Restricted state keeps existing bookings working; neutral suspended page with tenant contacts (BRD §7.3) | Product |
| R21 | **Node renderer** adds a second runtime to a .NET team | L | L | Small, stateless service; shared theme packages with admin preview; documented ownership | Architecture |
| R22 | **Team capacity**: 5-month MVP is tight for the listed scope | M | H | Cut list ready (12 §5); pilot from week 18; weekly scope review | PM |
| R23 | **"Powered by SSPL" badge** does little for acquisition | M | L | "Powered by Cowos" with referral tracking (00 §C6) | Marketing |
| R24 | **Image licensing / third-party content** on tenant sites | L | M | Terms make tenant responsible; takedown process; stock image library with licensed images in themes | Legal |

## 2. Open questions

| ID | Question | Why it matters | Default if unanswered |
|----|----------|----------------|----------------------|
| OQ1 | Final plan prices and limits | Marketing site, billing build | Indicative plans in 00 §C1 |
| OQ2 | Will SSPL add a margin on messaging (handling fee), or pass through at cost? | Transparency message, wallet pricing | At cost in v1; OTPs included in plan up to a fair-use limit |
| OQ3 | Refund policy for SSPL plan fees | Terms, billing logic | Annual: pro-rata within 30 days; monthly: non-refundable |
| OQ4 | Is badge removal ever sold? | Growth vs revenue | Not in v1; revisit with data |
| OQ5 | Is a trial publish to subdomain allowed, or must tenants pay before any publish (original brief)? | Conversion, fraud risk | Trial publish allowed on subdomain, noindex |
| OQ6 | Which SMS and WhatsApp providers? | Sprint 0 DLT/template registration | Shortlist 2, choose on price + delivery SLA in sprint 0 |
| OQ7 | Final domains (`cowos.in`, `cowos.site` or alternatives) | Everything domain-related | As per 00 §C5 |
| OQ8 | Support hours and SLA for tenants | Staffing, review SLA | Mon–Sat 9:00–19:00 IST; first-publish review within 4 working hours |
| OQ9 | Who owns theme design: in-house or agency? | Quality of the first impression | Contract one agency for 4 themes |
| OQ10 | Do pilot tenants already exist (warm leads)? | Pilot timing | Recruit 10 in Hyderabad during sprint 0 |
| OQ11 | Should customers be able to pay "at the venue" for Instant Book products? | Booking modes, no-show risk | Off by default; tenant can enable per product |
| OQ12 | Assisted onboarding: free, paid, or both? | Revenue, support load | Paid one-time setup (e.g. ₹2,999), free for annual plans |
| OQ13 | Do we need Telugu/Hindi in the booking app for Phase 1? | Localisation effort | No; English only, strings externalised |
| OQ14 | Data retention period for inactive customer data | DPDP compliance | 3 years after last activity, tenant-configurable within legal bounds |
| OQ15 | GST treatment of SSPL's own service (SAC, place of supply) | Platform invoices | Confirm with CA during sprint 0 |

## 3. Decision log

Status: **Proposed** until you approve. Change the status column as you decide.

| ID | Decision | Options | Recommendation | Status | Needed by |
|----|----------|---------|----------------|--------|-----------|
| D1 | Commercial model | (a) Pass-through server cost (b) Flat plans + 0% commission (c) Commission-based | (b), see 00 §C1 | Proposed | Before marketing copy (week 2) |
| D2 | Hosting model | (a) EC2 per tenant (b) Shared pool (c) Hybrid | (b) now, (c) capability designed in | Proposed | Sprint 0 |
| D3 | Payment model | (a) Razorpay Route (b) Tenant's own account via OAuth (c) Tenant API keys only | (b) with (c) as fallback | Proposed | Sprint 0 (start partner application) |
| D4 | Domain structure | As brief (`booking.<tenant>.cowos.in`) vs separate tenant domain with `/book` path | `<slug>.cowos.site` + `/book`; custom: `www.` + `book.` or `/book` | Proposed | Sprint 0 |
| D5 | Badge text | "Powered by SSPL" vs "Powered by Cowos" | "Powered by Cowos" | Proposed | Before theme design |
| D6 | Multi-tenancy DB | Shared schema + RLS vs schema per tenant vs DB per tenant | Shared + RLS; dedicated DB cells for large tenants later | Proposed | Sprint 0 |
| D7 | Messaging pricing | At cost vs cost + handling fee; OTPs included or charged | At cost; OTPs included up to fair use | Proposed | Week 6 |
| D8 | Platform fee refunds | Policy options in OQ3 | As in OQ3 default | Proposed | Before billing build (week 19) |
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
3. **An opt-in "Cowos Network" later (Phase 3+).** Once there are a few hundred tenants, an opt-in directory where members of one space can book day passes or rooms at another (with a flat network fee, not a commission) gives tenants discovery and gives Cowos a moat aggregators can't copy easily. Only pursue it if tenant density in a city justifies it, and keep it opt-in so it never contradicts the "you own your customers" promise.
