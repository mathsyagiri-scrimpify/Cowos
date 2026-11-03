# 12 · Release Plan

> Cowos requirements pack · v0.2 draft · Durations assume the team in 00 Q12 (2 BE, 2 FE, 1 full-stack/DevOps, 1 QA, part-time designer, PM). Treat them as planning estimates, to be re-estimated after sprint 0.

## 1. Phase overview

| Phase | Code | Theme | Indicative timing | Exit criteria |
|-------|------|-------|-------------------|---------------|
| Sprint 0 | P1 | Foundations and long-lead approvals | Weeks 1–3 | Domains bought; Razorpay Partner, DLT, WhatsApp, SES production applications submitted; CloudFront multi-tenant routing spike passed or Caddy fallback chosen (R5); repo, CI, Terraform skeleton, environments up; DESIGN.md and the 2 launch theme designs approved (22A) |
| Phase 1 (MVP launch) | P1 | "Publish a site and take paid bookings" | Weeks 4–24 (~5 months) | Production entry gate passed (§4); 10 pilot tenants live with real bookings at the week-24 launch, the other 10 within 6 weeks (Pilot Rules #1); launch checklist (§4) passed |
| Themes 3–4 | P1 | Remaining 2 theme directions (24A) | Launch + about 6 weeks [estimated] | Directions validated with 3 pilot owners; each theme passes Lighthouse, axe and the screen-reader pass before release (CEO-S3) |
| Billing release | P1-B | "Charge the real hosting cost and open sign-up" | Launch + about 4 months [estimated] | Billing release scope in §2 live (CEO-S1, CEO-S2); SSPL's paid test tenant on real hosting charges. Before the first pilot converts: at least one real monthly hosting charge paid and one dunning cycle run end to end on production (Pilot Rules #7) |
| Phase 2 | P2 | "Run the space on Cowos" | Months 7–10 | Memberships live for pilot tenants; widget used by ≥10 tenants; PWA installs measurable |
| Phase 3 | P3 | "Scale and integrate" | Months 11–18 | Dedicated cell for first large tenant; e-invoicing; accounting exports; WordPress plugin |

**P1-B (Billing release)** is the phase code for requirements that ship about 4 months after the week-24 launch. Nobody pays during the pilot free year, so platform billing, self-serve sign-up and the onboarding wizard wait for this release; it still ships about 8 months before the first pilot is invoiced (CEO-S1, CEO-S2). Until then, tenants are pilots created by SSPL, and the only tenant states in use are Active (pilot) and Suspended (BRD §7.3).

## 2. Feature split

| Area | Phase 1 (MVP launch, P1) | Billing release (P1-B) | Phase 2 | Phase 3 |
|------|--------------------------|------------------------|---------|---------|
| Marketing site | Product pages, pricing page with the 0% commission promise and the pilot offer (28A), savings calculator, demo sites (one per launched theme), "Book a demo" pilot request form with spam protection, privacy notice and storage in Super Admin (CEO-A2), help centre | Public sign-up (FR-MW-005), referral capture (FR-MW-006), slug availability check (FR-MW-012), "last month's actual" per-space share from the monthly pricing JSON (R12) (CEO-S1) | Blog, city landing pages, case studies | Partner/reseller portal |
| Onboarding | Settings screens for every setup step (business details with GSTIN format/checksum and PAN derivation, location, hours, theme, branding, inventory, policies, Razorpay connect with MFA); slug set in settings; dashboard checklist with the owner-only steps (FR-TA-002); first-login handoff (11A) | 10-step wizard in the 4A order (FR-TA-001), GSTIN legal-name lookup, Places autocomplete (FR-TA-004), booking-only path (FR-TA-006), demo inventory with the "Sample" label and its publish check (FR-TA-008) (CEO-S2) | AI copy assistant, assisted-onboarding tooling | Import from existing website (scrape content) |
| Site builder | 2 themes at launch (Loft and Metro, or the 2 that pilot validation favours), each with its booking shell (16A); themes 3–4 about 6 weeks after launch (CEO-S3); branding tokens with OKLCH colour roles and contrast check (23A); 12 section types, custom pages, SEO fields, analytics IDs, device preview, share preview | None | 2–4 more themes, blog, draft vs live compare, scheduled publish | Multi-language content, HTML embed, white-label admin |
| Publish & domains | Versions + rollback, first-publish review, platform subdomain, custom domain with DNS guides + auto SSL, 301 canonicalisation, health alerts | None | Badge removal (if decided, 13 OQ4), domain change history | Domain purchase via reseller, static IPs for apex |
| Inventory | Locations (1 in UI, many in data), 6 product types, units + pools, hourly/daily/weekly/monthly pricing, member price field, add-ons, buffers, lead time, holidays, maintenance, tour slots, deposits | None | Multi-location UI, peak/off-peak pricing, floor-plan pins | Full floor-plan editor, dynamic pricing |
| Booking app | Phone/email OTP, browse, availability as duration then a list of start times (1A), unit list selection, price breakdown with GST, coupons, checkout in the 2A order (summary, OTP, hold, pay), Razorpay Checkout, confirmation + QR + `.ics`, desk pass, Request to Book, Enquire/Tour, my bookings, cancel, reschedule, invoices, bill-to-company | None | PWA + push, member credits at checkout, membership purchase, virtual office application with KYC upload, company admin, self check-in, floor-plan selection, in-app notifications | Waitlist, gift cards / passes, single "Cowos Member" native app (only if data supports it) |
| Front desk & bookings | Calendar views, today view, staff bookings, approvals, cancel/override, reschedule, QR check-in, no-show, timeline | None | Recurring bookings, visitor management | Access control integrations |
| CRM | Customers, companies with GSTIN, leads pipeline, tours, import/export, DPDP queue | None | Segments + broadcasts (consent-checked), lead source analytics | Automations (follow-up sequences) |
| Payments & invoices | Razorpay OAuth + key fallback, test/live, payments list, offline payments, GST invoices, credit notes, bills of supply, deposits | None | Second gateway, monthly consolidated company invoices, GSTR-1 export, TDS recording | E-invoicing IRN, Tally/Zoho Books |
| Memberships | Data model only | None | Plans, credits ledger, renewals, pause/upgrade/cancel | UPI Autopay for member renewals |
| Notifications | Email (SES), SMS (DLT), WhatsApp from Cowos number; channel toggles; delivery log; SSPL-funded pilot wallet with fair-use caps (Pilot Rules #5) | Non-pilot wallet top-up at provider cost (Pilot Rules #10) | Tenant-owned WhatsApp numbers, tenant sending domain | Two-way WhatsApp inbox |
| Integration with existing sites | `book.<domain>` CNAME, button snippets, deep links, QR poster | None | Script widget (floating + inline), widget events, allowed origins | WordPress plugin |
| Platform billing | Tenant state model with Active (pilot) and Suspended only; moves into Trial, PastDue or Restricted rejected with 409 (Q1); pilot flag and free-period end date | Hosting cost billing from CUR 2.0 (cost pool, tenant cost shares, hosting invoices with GST, adjustment lines), payment link and UPI Autopay / e-mandate collection, dunning with automatic Restricted and Suspended moves, 14-day trial, daily charge estimate (CEO-S1, D1) | Reseller billing. Removed in v0.2: annual plan incentives and coupons for plans (no plans, D1) | Dedicated cell charging (open under D1, 13 OQ20) |
| Super Admin | Tenant list/detail, review queue, suspend/reinstate with the suspended-page switch (CEO-A1), Pilots page with three tabs: Create pilot with Owner invite (CEO A2), Pilot requests (CEO-A2) and Scorecard with CSV export (CEO-A4), support grants including the pre-handover setup grant (R19), theme catalog, settings, health dashboard, deletion workflow | Cost pool, invoice and billing-run screens (28A), dunning dashboard (FR-SA-023), business dashboard with sign-up, trial and conversion metrics (FR-SA-051) (CEO-S1, CEO-A4) | Announcements, cohort analytics | Cell provisioning, partner management |
| Reports | Dashboard KPIs, CSV exports | None | Revenue, occupancy heatmap, coupon, lead conversion, churn reports | Benchmarks across tenants (opt-in, anonymised) |
| Platform/infra | Shared multi-tenant on ECS + RDS, RLS, static sites on CloudFront, WAF, backups + DR copy, observability, CI/CD, pen test | None | Read replica for reporting, SQS for high-volume queues if needed | Dedicated cells, second pool, multi-region active-passive |
| Calendar | `.ics` + add-to-calendar links | None | Staff iCal feeds | Google/Outlook resource calendar push |

## 3. MVP milestone plan (indicative)

| Weeks | Milestone | Content |
|-------|-----------|---------|
| 1–3 | M0 Foundations | Sprint 0 items above; architecture spikes (CloudFront SaaS Manager, Razorpay OAuth, renderer), RLS + tenant context skeleton, CI with isolation test harness |
| 4–7 | M1 Tenant core | Auth + MFA, Super Admin create pilot tenant with Owner invite (CEO A2), tenant state with the launch guard (Q1), settings screens for business/location/hours, branding, media pipeline |
| 8–11 | M2 Builder + publish | 2 themes, section editors, preview, site renderer, versions/rollback, platform subdomain publishing, review queue |
| 12–15 | M3 Inventory + booking | Products, units, pools, pricing, rules, availability engine, holds with constraints, booking app browse → OTP → hold (1A, 2A) |
| 16–18 | M4 Money | Razorpay OAuth + Checkout, webhooks, reconciliation, invoices + credit notes, cancellations/refunds, offline payments |
| 19–21 | M5 Operations | Front desk views, requests, leads/tours, notifications (email/SMS/WhatsApp) with the SSPL-funded pilot wallet and caps, custom domains + SSL, Super Admin essentials (Pilots page, suspend/reinstate with the suspended page, support grants). No platform billing: it moves to the Billing release (CEO-S1) |
| 22–24 | M6 Hardening + pilot | Accessibility and performance passes on the 2 launch themes, pen test and fixes, DR drill, production entry gate (§4). From week 22, pilot tenants are set up on production before the gate passes (Pilot Rules #1). 10 pilots live, launch. No themes 3–4: they follow after launch (CEO-S3) |
| Launch + ~6 weeks | M7 Themes 3–4 and remaining pilots | Themes 3–4 after validation with 3 pilot owners, each passing Lighthouse, axe and the screen-reader pass (CEO-S3). The other 10 pilots live (Pilot Rules #1) |
| Launch + ~4 months | M8 Billing release (P1-B) | Billing release scope in §2 (CEO-S1, CEO-S2). Exit criterion before the first pilot converts in §1 (Pilot Rules #7) |

Pilot timing (Pilot Rules #1, #9):
- **Weeks 18–21:** pilots see a guided staging preview in Razorpay test mode only, with no real customers.
- **From week 22:** pilot tenants are set up on production before the production entry gate passes: created and configured through the pre-handover setup grant (R19), then tenant-approved support grants, unpublished, with no live payments and no customer data. The gate blocks only publishing and live payments.
- **Launch:** 10 pilots live at the week-24 launch; the other 10 within 6 weeks after it. If the gate passes after week 24, the "10 pilots live" target moves to 2 weeks after the gate passes.
- **Onboarding owner:** one named SSPL owner (PM or support lead) runs pilot onboarding from the staging preview to production, about 15 person-days across weeks 18–30, outside the core dev team. Each pilot is set up fresh on production, about 2 hours more per pilot.

Pilot feedback on setup and booking modes is worth more than any extra feature.

## 4. MVP launch checklist

### 4.1 Production entry gate (Pilot Rules #1)

Pilots move to production publishing and live payments only after every item below passes. That is no earlier than week 22, and likely week 24. Before the gate passes, pilot tenants can be set up on production (unpublished, no live payments, no customer data); the gate blocks only publishing and live payments.

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
- [ ] On-call rota, runbooks (payments down, DNS/SSL issues, data breach, tenant suspension), status page live
- [ ] Support playbook and help-centre articles for top 20 expected questions (DNS per registrar, Razorpay connection)
- [ ] Super Admin Pilots page live (Create pilot, Pilot requests, Scorecard) and manual suspend/reinstate tested with the suspended page (CEO A2, CEO-A1, CEO-A2, CEO-A4)
- [ ] 10 pilot tenants live; at least 5 with paid online bookings

Each later theme (themes 3–4) passes the same performance and accessibility checks before its release (CEO-S3).

## 5. What we deliberately cut from MVP (and why)

| Cut | Why it's safe to wait |
|-----|----------------------|
| Memberships with credits | Pilot tenants can manage monthly members with monthly bookings + offline payments; build it with real usage data |
| Script widget | `book.<domain>` link covers existing-website tenants with less risk |
| PWA | Responsive web works; add install + push once booking volume shows repeat use |
| Floor plans | List selection is enough for most spaces under 100 seats |
| Peak pricing, recurring bookings, broadcasts | Nice to have; low pilot demand expected |
| Second gateway | Razorpay covers the market; add when a tenant can't use Razorpay |
| Platform billing automation (CUR billing job, invoices, collection, dunning with automatic Restricted/Suspended moves, pricing JSON, daily charge estimate), 14-day trial, non-pilot wallet top-up, public sign-up, referral capture, slug availability check, sign-up and trial metrics, dunning dashboard → Billing release (CEO-S1) | Nobody pays during the pilot free year. Frees about 3 weeks of 1 BE + 1 FE before launch [estimated]. Ships about 8 months before the first pilot is invoiced, and the billing validation exit criterion still applies (Pilot Rules #7) |
| Onboarding wizard (4A order), GSTIN legal-name lookup, Places autocomplete, booking-only path, demo inventory with its sample publish check → Billing release (CEO-S2) | Only self-serve sign-ups use the wizard, and they start at the Billing release. SSPL sets up pilots through the settings screens. Frees about 1.5 weeks of FE before launch [estimated] |
| Themes 3–4 → launch + about 6 weeks (CEO-S3) | Keeps design work out of the launch-gate weeks, and the directions are validated with 3 pilot owners first (24A). Frees about 2 weeks of FE + designer time in weeks 22–24 [estimated] |
| Plans, plan limits, upgrades/downgrades, annual plans, plan coupons → Removed in v0.2 (D1) | Tenants pay only their hosting cost share and messaging at cost; every tenant gets every feature |
