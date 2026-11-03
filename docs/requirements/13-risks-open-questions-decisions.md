# 13 · Risks, Open Questions & Decisions

> Cowos requirements pack · v0.2 draft

Note: other pack files cite engineering review decisions from `docs/designs/cowos-platform-v1.md` as "(R1)"–"(R20)". This file writes them "eng R1"–"eng R20" so they are not confused with the risks R1–R27 below.

## 1. Risk register

Likelihood (L) and Impact (I): H / M / L.

| ID | Risk | L | I | Mitigation | Owner |
|----|------|:-:|:-:|------------|-------|
| R1 | **The revenue model does not fund SSPL's support, onboarding or development.** Tenants pay only cost (hosting share + messaging at cost), with no margin | H | H | **Accepted risk** (Premise 5, D1, 2026-11-02). G3 tracks cost recovery monthly, with SSPL's unrecovered cost (staging, dev, pilot year, team) shown alongside (CEO-A5). The billing page shows the real AWS breakdown, so the charge holds up when tenants compare it to AWS prices (5A) | Leadership |
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
| R19 | **Messaging costs** (WhatsApp/SMS) exceed expectations: for tenants as wallet spend, and for SSPL on pilot wallets | M | M | Prepaid wallet at provider cost with a conditional debit (eng R4), per-event channel toggles, WhatsApp-first for OTP where cheaper, monitoring; pilot fair-use caps (Pilot Rules #5) | Finance |
| R20 | **Suspending sites harms tenants' customers** and creates disputes | M | M | Restricted state keeps existing bookings working; neutral suspended page with tenant contacts (BRD §7.3) | Product |
| R21 | **Node renderer** adds a second runtime to a .NET team | L | L | Small, stateless service; shared theme packages with admin preview; documented ownership | Architecture |
| R22 | **Team capacity**: 5-month MVP is tight for the listed scope | M | H | Cut list ready (12 §5), including the Billing release and themes 3–4 deferrals (CEO-S1, CEO-S2, CEO-S3); pilot staging preview from week 18; weekly scope review | PM |
| R23 | **"Powered by SSPL" badge** does little for acquisition | M | L | "Powered by Cowos" with referral tracking (00 §C6) | Marketing |
| R24 | **Image licensing / third-party content** on tenant sites | L | M | Terms make tenant responsible; takedown process; stock image library with licensed images in themes | Legal |
| R25 | **SSPL carries the pilot-year cost**: about ₹4–7.5 lakh hosting for 12 months including staging, plus up to about ₹1.8 lakh messaging for 20 pilots at both caps [estimated] | H | M | Fair-use caps on pilot notifications and OTPs (Pilot Rules #5); scorecard shows cap usage (CEO-A4); the fixed base cost is shared with paying tenants from the Billing release | Leadership / Finance |
| R26 | **Demand is unverified**: the complete suite is built whatever the demand evidence says | M | H | **Accepted risk** (Premise 3). Pilot request intake (CEO-A2), pilot scorecard (CEO-A4), and the expansion gate before a second city (Pilot Rules #8) | Leadership |
| R27 | **Billing is built about 4 months after launch** and runs against live tenants and real AWS bills from its first run | M | H | Exit criterion before the first pilot converts: one real monthly charge paid and one dunning cycle run end to end on production, with SSPL's paid test tenant from the Billing release (Pilot Rules #7); invoices held with a Finance alert when CUR data is missing (eng R2); billing states rejected with 409 until then (CEO review Q1); estimate tested with fixture CUR data (eng R15) | Engineering / Finance |

## 2. Open questions

| ID | Question | Why it matters | Default if unanswered |
|----|----------|----------------|----------------------|
| OQ1 | Final plan prices and limits | Marketing site, billing build | **Resolved by D1 (2026-11-02):** there are no plans or limits. Tenants pay their hosting cost share plus messaging at cost (00 §C1) |
| OQ2 | Will SSPL add a margin on messaging (handling fee), or pass through at cost? | Transparency message, wallet pricing | **Resolved by D1 (2026-11-02):** at provider cost from a prepaid wallet, with no free allowance; OTPs are debited like other messages, down to −₹500. Pilots use an SSPL-funded wallet within fair-use caps (Pilot Rules #5, #10) |
| OQ3 | Refund policy for SSPL charges | Terms, billing logic | Updated for D1: plan fees no longer exist, so the annual pro-rata option no longer applies. Monthly hosting invoices: non-refundable; later AWS credits or refunds reach tenants as an adjustment line (Pilot Rules #10) |
| OQ4 | Is badge removal ever sold? | Growth vs revenue | Not in v1; revisit with data |
| OQ5 | Is a trial publish to subdomain allowed, or must tenants pay before any publish (original brief)? | Conversion, fraud risk | Trial publish allowed on subdomain, noindex |
| OQ6 | Which SMS and WhatsApp providers? | Sprint 0 DLT/template registration | Shortlist 2, choose on price + delivery SLA in sprint 0 |
| OQ7 | Final domains (`cowos.in`, `cowos.site` or alternatives) | Everything domain-related | As per 00 §C5 |
| OQ8 | Support hours and SLA for tenants | Staffing, review SLA | Mon–Sat 9:00–19:00 IST; first-publish review within 4 working hours |
| OQ9 | Who owns theme design: in-house or agency? | Quality of the first impression | Contract one agency for 4 themes |
| OQ10 | Do pilot tenants already exist (warm leads)? | Pilot timing | Updated by the Pilot Rules (2026-11-02): the pilot is the first 20 owners in the beachhead city, 12 months free; 10 live at the week-24 launch, the other 10 within 6 weeks (Pilot Rules #1). Intake through the "Book a demo" pilot request form (CEO-A2). Whether warm leads exist is still unknown. Default: recruit 20 in the beachhead city (OQ16, OQ17), starting in sprint 0 |
| OQ11 | Should customers be able to pay "at the venue" for Instant Book products? | Booking modes, no-show risk | Off by default; tenant can enable per product |
| OQ12 | Assisted onboarding: free, paid, or both? Does a paid setup service still fit D1 ("no subscription, plan fee or commission")? | Revenue, support load | Pilots: free (Pilot Rules #9). Others: paid one-time setup (e.g. ₹2,999) as in BRD BR-05. The v0.1 "free for annual plans" option is removed (no plans, D1) |
| OQ13 | Do we need Telugu/Hindi in the booking app for Phase 1? | Localisation effort | No; English only, strings externalised |
| OQ14 | Data retention period for inactive customer data | DPDP compliance | 3 years after last activity, tenant-configurable within legal bounds |
| OQ15 | GST treatment of SSPL's own service (SAC, place of supply) | Platform invoices | Confirm with CA during sprint 0 |
| OQ16 | Which city is the beachhead? | Pilot recruitment, expansion gate | Hyderabad (suggested, not confirmed) |
| OQ17 | How are the 20 pilot spaces chosen (size, aggregator share, existing website or not)? | Pilot results, expansion gate | Independent 1–3 location spaces with a meaningful share of bookings from aggregators (beachhead definition) |
| OQ18 | Is the provisional messaging cap of 1,000 notifications per pilot per month right? | SSPL's pilot cost, pilot experience | Keep it; confirm after the first month of pilot volumes |
| OQ19 | Should the AWS bill be split equally across counted tenants, or weighted (for example by bookable resources or traffic)? | Fairness between small and large tenants | Equal split in v1; decide after 3 months of real bills from the Billing release (TODOS.md) |
| OQ20 | How is a Phase 3 dedicated cell charged under D1? | Large tenants, Phase 3 billing | Not decided. The v0.1 quote of "AWS cost + fixed ops fee" conflicts with D1 |

## 3. Decision log

Status: **Proposed** until you approve. Change the status column as you decide. D1 is Decided; D2–D16 are still Proposed.

| ID | Decision | Options | Recommendation | Status | Needed by |
|----|----------|---------|----------------|--------|-----------|
| D1 | Commercial model | (a) Pass-through server cost (b) Flat plans + 0% commission (c) Commission-based | v0.1 recommended (b). **Owner chose (a) as a hosting cost share:** tenants pay their share of the production AWS bill plus messaging at cost, with no subscription, plan fee or commission (00 §C1, Premise 5, Pilot Rules #10) | **Decided 2026-11-02** | Done |
| D2 | Hosting model | (a) EC2 per tenant (b) Shared pool (c) Hybrid | (b) now, (c) capability designed in | Proposed | Sprint 0 |
| D3 | Payment model | (a) Razorpay Route (b) Tenant's own account via OAuth (c) Tenant API keys only | (b) with (c) as fallback | Proposed | Sprint 0 (start partner application) |
| D4 | Domain structure | As brief (`booking.<tenant>.cowos.in`) vs separate tenant domain with `/book` path | `<slug>.cowos.site` + `/book`; custom: `www.` + `book.` or `/book` | Proposed | Sprint 0 |
| D5 | Badge text | "Powered by SSPL" vs "Powered by Cowos" | "Powered by Cowos" | Proposed | Before theme design |
| D6 | Multi-tenancy DB | Shared schema + RLS vs schema per tenant vs DB per tenant | Shared + RLS; dedicated DB cells for large tenants later | Proposed | Sprint 0 |
| D7 | Messaging pricing | At cost vs cost + handling fee; OTPs included or charged | At provider cost, no free allowance; OTPs charged from the wallet like other messages (follows from D1 and Pilot Rules #10, OQ2) | Proposed | Week 6 |
| D8 | Refunds of SSPL charges | Policy options in OQ3 | As in OQ3 default (updated for D1) | Proposed | Before the Billing release build |
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
