# 12 · Release Plan

> Cowos requirements pack · v0.3 draft · Durations assume the team in 00 Q12 (2 BE, 2 FE, 1 full-stack/DevOps, 1 QA, part-time designer, PM). Treat them as planning estimates, to be re-estimated after sprint 0.

## 1. Phase overview

| Phase | Code | Theme | Indicative timing | Exit criteria |
|-------|------|-------|-------------------|---------------|
| Sprint 0 | P1 | Foundations and long-lead approvals | Weeks 1–3 | Domains bought; Razorpay Partner, DLT, WhatsApp, SES production applications submitted; CloudFront multi-tenant routing spike passed or Caddy fallback chosen (R5); repo, CI, Terraform skeleton, environments up; DESIGN.md and the 2 launch theme designs approved (22A) |
| Phase 1 (MVP launch) | P1 | "Sign up, publish a site, take paid bookings, pay for usage" | Weeks 4–24 (~5 months) | Production entry gate passed (§4.1); 10 tenants live at the week-24 launch (Pricing Rule #1); launch checklist (§4.2) passed |
| Themes 3–4 | P1 | Remaining 2 theme directions (24A) | Launch + about 6 weeks [estimated] | Directions validated with 3 early tenant owners; each theme passes Lighthouse, axe and the screen-reader pass before release (CEO-S3) |
| Phase 2 | P2 | "Run the space on Cowos" | Months 7–10 | Memberships live for tenants; widget used by ≥10 tenants; PWA installs measurable |
| Phase 3 | P3 | "Scale and integrate" | Months 11–18 | Dedicated cell for first large tenant; e-invoicing; accounting exports; WordPress plugin |

There is no Billing release. Every tenant pays from the end of its trial, so metering, the rate card, rating, server invoices, the slab-priced wallet, collection, dunning, the 14-day trial, self-serve sign-up and the onboarding wizard ship at launch (Pricing Rules #3–#9). This reverses the CEO review's deferral (CEO-S1, CEO-S2) and adds back roughly 3 weeks of 1 BE + 1 FE and about 1.5 weeks of FE before launch [estimated, from the CEO review]. Launch stays at week 24. All tenant states (BRD §7.3) are in use from launch.

## 2. Feature split

| Area | Phase 1 (MVP launch, P1) | Phase 2 | Phase 3 |
|------|--------------------------|---------|---------|
| Marketing site | Product pages, pricing page with the 0% commission promise and the current rate card (slab tables per meter) (28A, Pricing Rule #5), savings calculator, demo sites (one per launched theme), "Book a demo" form with spam protection, privacy notice and storage in Super Admin as Demo requests (CEO-A2), public sign-up (FR-MW-005), referral capture (FR-MW-006), slug availability check (FR-MW-012), help centre | Blog, city landing pages, case studies | Partner/reseller portal |
| Onboarding | 10-step wizard in the 4A order (FR-TA-001), GSTIN format/checksum, PAN derivation and GSTIN legal-name lookup, Places autocomplete (FR-TA-004), booking-only path (FR-TA-006), demo inventory with the "Sample" label and its publish check (FR-TA-008); settings screens for every setup step (business details, location, hours, theme, branding, inventory, policies, Razorpay connect with MFA); dashboard checklist with the owner-only steps (FR-TA-002) (Pricing Rule #9) | AI copy assistant, assisted-onboarding tooling | Import from existing website (scrape content) |
| Site builder | 2 themes at launch (Loft and Metro, or the 2 that early validation favours), each with its booking shell (16A); themes 3–4 about 6 weeks after launch (CEO-S3); branding tokens with OKLCH colour roles and contrast check (23A); 12 section types, custom pages, SEO fields, analytics IDs, device preview, share preview | 2–4 more themes, blog, draft vs live compare, scheduled publish | Multi-language content, HTML embed, white-label admin |
| Publish & domains | Versions + rollback, first-publish review, platform subdomain, custom domain with DNS guides + auto SSL, 301 canonicalisation, health alerts | Badge removal (if decided, 13 OQ4), domain change history | Domain purchase via reseller, static IPs for apex |
| Inventory | Locations (1 in UI, many in data), 6 product types, units + pools, hourly/daily/weekly/monthly pricing, member price field, add-ons, buffers, lead time, holidays, maintenance, tour slots, deposits | Multi-location UI, peak/off-peak pricing, floor-plan pins | Full floor-plan editor, dynamic pricing |
| Booking app | Phone/email OTP, browse, availability as duration then a list of start times (1A), unit list selection, price breakdown with GST, coupons, checkout in the 2A order (summary, OTP, hold, pay), Razorpay Checkout, confirmation + QR + `.ics`, desk pass, Request to Book, Enquire/Tour, my bookings, cancel, reschedule, invoices, bill-to-company | PWA + push, member credits at checkout, membership purchase, virtual office application with KYC upload, company admin, self check-in, floor-plan selection, in-app notifications | Waitlist, gift cards / passes, single "Cowos Member" native app (only if data supports it) |
| Front desk & bookings | Calendar views, today view, staff bookings, approvals, cancel/override, reschedule, QR check-in, no-show, timeline | Recurring bookings, visitor management | Access control integrations |
| CRM | Customers, companies with GSTIN, leads pipeline, tours, import/export, DPDP queue | Segments + broadcasts (consent-checked), lead source analytics | Automations (follow-up sequences) |
| Payments & invoices | Razorpay OAuth + key fallback, test/live, payments list, offline payments, GST invoices, credit notes, bills of supply, deposits | Second gateway, monthly consolidated company invoices, GSTR-1 export, TDS recording | E-invoicing IRN, Tally/Zoho Books |
| Memberships | Data model only | Plans, credits ledger, renewals, pause/upgrade/cancel | UPI Autopay for member renewals |
| Notifications | Email (SES), SMS (DLT), WhatsApp from Cowos number; channel toggles; delivery log; prepaid wallet with per-channel slab pricing, month-to-date counts, email fallback at zero balance and the −₹500 floor for OTPs and emails (Pricing Rule #6, eng R4) | Tenant-owned WhatsApp numbers, tenant sending domain | Two-way WhatsApp inbox |
| Integration with existing sites | `book.<domain>` CNAME, button snippets, deep links, QR poster | Script widget (floating + inline), widget events, allowed origins | WordPress plugin |
| Platform billing | Full tenant state model (BRD §7.3); 14-day trial with server usage not billed (Pricing Rule #7); metering of request, storage and notification units (Rule #3); rate card with versioned slab tables (Rule #4); rating job on the 1st and server invoices with GST (Rule #5); billing page with units per meter, slab breakdown, daily month-to-date estimate and daily usage detail; payment link and UPI Autopay / e-mandate collection; dunning with automatic Restricted and Suspended moves (Rule #8) | Reseller billing. Removed in v0.2: annual plan incentives and coupons for plans (no plans, D1) | Dedicated cell charging (open under D1, 13 OQ20) |
| Super Admin | Tenant list/detail, review queue, suspend/reinstate with the suspended-page switch (CEO-A1), Demo requests (CEO-A2), tenant scorecard with CSV export (CEO-A4), support grants, rate card and versions, rating runs and server invoices, wallet adjustments, dunning dashboard (FR-SA-023), business dashboard with sign-up, trial and conversion metrics (FR-SA-051), SSPL cost view from CUR 2.0 (Pricing Rule #10), theme catalog, settings, health dashboard, deletion workflow | Announcements, cohort analytics | Cell provisioning, partner management |
| Reports | Dashboard KPIs, CSV exports | Revenue, occupancy heatmap, coupon, lead conversion, churn reports | Benchmarks across tenants (opt-in, anonymised) |
| Platform/infra | Shared multi-tenant on one app server (EC2) running the API, admin portals, booking app, worker and renderer, with automatic instance recovery, an infrastructure-as-code rebuild path and health alarms; managed PostgreSQL (RDS) with backups and PITR; RLS; static sites on S3 + CloudFront; WAF; DR copy; usage metering pipeline (CloudFront standard logs, API request counts, daily storage snapshots); observability, CI/CD, pen test (Pricing Rule #11, D2) | Read replica for reporting, SQS for high-volume queues if needed | Dedicated cells, second pool, multi-region active-passive |
| Calendar | `.ics` + add-to-calendar links | Staff iCal feeds | Google/Outlook resource calendar push |

## 3. MVP milestone plan (indicative)

| Weeks | Milestone | Content |
|-------|-----------|---------|
| 1–3 | M0 Foundations | Sprint 0 items above; architecture spikes (CloudFront SaaS Manager, Razorpay OAuth, renderer), the app server and managed PostgreSQL defined as infrastructure as code (Rule #11), RLS + tenant context skeleton, CI with isolation test harness |
| 4–7 | M1 Tenant core | Auth + MFA, self-serve sign-up with referral capture and slug availability check (FR-MW-005, 006, 012), tenant state model with the 14-day trial (BRD §7.3), onboarding wizard in the 4A order with GSTIN legal-name lookup and Places autocomplete (FR-TA-001, 004), settings screens for business/location/hours, branding, media pipeline |
| 8–11 | M2 Builder + publish | 2 themes, section editors, preview, site renderer, versions/rollback, platform subdomain publishing, review queue, booking-only path (FR-TA-006) |
| 12–15 | M3 Inventory + booking | Products, units, pools, pricing, rules, availability engine, holds with constraints, booking app browse → OTP → hold (1A, 2A), demo inventory with its sample publish check (FR-TA-008) |
| 16–18 | M4 Money | Razorpay OAuth + Checkout, webhooks, reconciliation, invoices + credit notes, cancellations/refunds, offline payments; metering of request and storage units, rate card with versioned slab tables, rating job on the 1st and server invoices with GST, payment link and UPI Autopay / e-mandate collection (Pricing Rules #3–#5) |
| 19–21 | M5 Operations | Front desk views, requests, leads/tours, notifications (email/SMS/WhatsApp) with the slab-priced prepaid wallet (Pricing Rule #6, eng R4), billing and wallet pages, dunning with automatic Restricted and Suspended moves (Rule #8), custom domains + SSL, Super Admin essentials (Demo requests, tenant scorecard, dunning dashboard, business dashboard, SSPL cost view, suspend/reinstate with the suspended page, support grants) |
| 22–24 | M6 Hardening + launch | Accessibility and performance passes on the 2 launch themes, pen test and fixes, DR drill, app server recovery and rebuild test, production entry gate (§4.1). 10 tenants live, launch (Pricing Rule #1). No themes 3–4: they follow after launch (CEO-S3) |
| Launch + ~6 weeks | Themes 3–4 | Themes 3–4 after validation with 3 early tenant owners, each passing Lighthouse, axe and the screen-reader pass (CEO-S3) |

M7 and M8 of v0.2 are removed in v0.3: there is no Billing release, and the pilot programme is removed. Billing, metering, the rate card, sign-up and the wizard are back in M1–M5. That adds roughly 3 weeks of 1 BE + 1 FE and about 1.5 weeks of FE before launch [estimated, from the CEO review], with launch still at week 24 (13 R22).

Recruiting the first tenants in the beachhead city runs in parallel with the build. Feedback from early tenant owners on setup and booking modes is worth more than any extra feature.

## 4. MVP launch checklist

### 4.1 Production entry gate (Pricing Rule #1)

Tenants go live on production (publishing and live payments) only after every item below passes. That is no earlier than week 22, and likely week 24.

- [ ] Cross-tenant isolation suite green for every endpoint; RLS enabled and forced on every tenant table
- [ ] Third-party penetration test done; all critical/high findings fixed
- [ ] Razorpay Partner approval live
- [ ] DLT templates approved; WhatsApp templates approved; SES out of sandbox
- [ ] Verified backups and point-in-time recovery in place, with a documented restore drill
- [ ] Counsel has reviewed the terms of service, the DPA, and the privacy and refund policy templates Cowos generates for tenants (DPDP + consumer protection)

### 4.2 Launch checklist

- [ ] Production entry gate (§4.1) passed
- [ ] Core Web Vitals and Lighthouse budgets met on every launched theme (NFR-PERF-01–03, CEO-S3)
- [ ] axe-core clean and manual screen reader pass on booking flow and every launched theme
- [ ] End-to-end live payment and refund on a real tenant account
- [ ] GST invoice and credit note formats reviewed by a CA
- [ ] Rate card configured by the product owner, with slab tables for every meter (Pricing Rule #4, 13 OQ21)
- [ ] App server automatic instance recovery, infrastructure-as-code rebuild and health alarms in place (Rule #11)
- [ ] On-call rota, runbooks (payments down, DNS/SSL issues, data breach, tenant suspension), status page live
- [ ] Support playbook and help-centre articles for top 20 expected questions (DNS per registrar, Razorpay connection)
- [ ] Super Admin Demo requests and tenant scorecard live, and manual suspend/reinstate tested with the suspended page (CEO-A1, CEO-A2, CEO-A4)
- [ ] 10 tenants live; at least 5 with paid online bookings (Pricing Rule #1)

Each later theme (themes 3–4) passes the same performance and accessibility checks before its release (CEO-S3).

## 5. What we deliberately cut from MVP (and why)

| Cut | Why it's safe to wait |
|-----|----------------------|
| Memberships with credits | Tenants can manage monthly members with monthly bookings + offline payments; build it with real usage data |
| Script widget | `book.<domain>` link covers existing-website tenants with less risk |
| PWA | Responsive web works; add install + push once booking volume shows repeat use |
| Floor plans | List selection is enough for most spaces under 100 seats |
| Peak pricing, recurring bookings, broadcasts | Nice to have; low early demand expected |
| Second gateway | Razorpay covers the market; add when a tenant can't use Razorpay |
| Themes 3–4 → launch + about 6 weeks (CEO-S3) | Keeps design work out of the launch-gate weeks, and the directions are validated with 3 early tenant owners first (24A). Frees about 2 weeks of FE + designer time in weeks 22–24 [estimated] |
| Plans, plan limits, upgrades/downgrades, annual plans, plan coupons → Removed in v0.2 (D1) | Tenants pay only metered server usage and notifications; every tenant gets every feature |

Removed in v0.3: the v0.2 rows that deferred platform billing and the onboarding wizard to the Billing release (CEO-S1, CEO-S2). Both ship at launch (§1).
