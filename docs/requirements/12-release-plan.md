# 12 · Release Plan

> Cowos requirements pack · v0.1 draft · Durations assume the team in 00 Q12 (2 BE, 2 FE, 1 full-stack/DevOps, 1 QA, part-time designer, PM). Treat them as planning estimates, to be re-estimated after sprint 0.

## 1. Phase overview

| Phase | Theme | Indicative timing | Exit criteria |
|-------|-------|-------------------|---------------|
| Sprint 0 | Foundations and long-lead approvals | Weeks 1–3 | Domains bought; Razorpay Partner, DLT, WhatsApp, SES production applications submitted; CloudFront SaaS Manager spike done; repo, CI, Terraform skeleton, environments up; design system and 2 theme designs approved |
| Phase 1 (MVP) | "Publish a site and take paid bookings" | Weeks 4–24 (~5 months) | 10 pilot tenants live with real bookings; launch checklist (§4) passed |
| Phase 2 | "Run the space on Cowos" | Months 7–10 | Memberships live for pilot tenants; widget used by ≥10 tenants; PWA installs measurable |
| Phase 3 | "Scale and integrate" | Months 11–18 | Dedicated cell for first large tenant; e-invoicing; accounting exports; WordPress plugin |

## 2. Feature split

| Area | Phase 1 (MVP) | Phase 2 | Phase 3 |
|------|---------------|---------|---------|
| Marketing site | Product pages, pricing with 0% commission + fee allocation, savings calculator, sign-up, referral capture, demo sites, help centre | Blog, city landing pages, case studies | Partner/reseller portal |
| Onboarding | 10-step wizard, GSTIN validation + lookup, Places autocomplete, demo inventory, booking-only path | AI copy assistant, assisted-onboarding tooling | Import from existing website (scrape content) |
| Site builder | 4 themes, branding tokens with contrast check, 12 section types, custom pages, SEO fields, analytics IDs, device preview, share preview | 2–4 more themes, blog, draft vs live compare, scheduled publish | Multi-language content, HTML embed (Pro), white-label admin |
| Publish & domains | Versions + rollback, first-publish review, platform subdomain, custom domain with DNS guides + auto SSL, 301 canonicalisation, health alerts | Badge removal add-on (if decided), domain change history | Domain purchase via reseller, static IPs for apex (Pro) |
| Inventory | Locations (1 in UI, many in data), 6 product types, units + pools, hourly/daily/weekly/monthly pricing, member price field, add-ons, buffers, lead time, holidays, maintenance, tour slots, deposits | Multi-location UI, peak/off-peak pricing, floor-plan pins | Full floor-plan editor, dynamic pricing |
| Booking app | Phone/email OTP, browse, availability, unit list selection, price breakdown with GST, coupons, holds, Razorpay Checkout, confirmation + QR + `.ics`, Request to Book, Enquire/Tour, my bookings, cancel, reschedule, invoices, bill-to-company | PWA + push, member credits at checkout, membership purchase, virtual office application with KYC upload, company admin, self check-in, floor-plan selection, in-app notifications | Waitlist, gift cards / passes, single "Cowos Member" native app (only if data supports it) |
| Front desk & bookings | Calendar views, today view, staff bookings, approvals, cancel/override, reschedule, QR check-in, no-show, timeline | Recurring bookings, visitor management | Access control integrations |
| CRM | Customers, companies with GSTIN, leads pipeline, tours, import/export, DPDP queue | Segments + broadcasts (consent-checked), lead source analytics | Automations (follow-up sequences) |
| Payments & invoices | Razorpay OAuth + key fallback, test/live, payments list, offline payments, GST invoices, credit notes, bills of supply, deposits | Second gateway, monthly consolidated company invoices, GSTR-1 export, TDS recording | E-invoicing IRN, Tally/Zoho Books |
| Memberships | Data model only | Plans, credits ledger, renewals, pause/upgrade/cancel | UPI Autopay for member renewals |
| Notifications | Email (SES), SMS (DLT), WhatsApp from Cowos number; channel toggles; delivery log; wallet | Tenant-owned WhatsApp numbers, tenant sending domain | Two-way WhatsApp inbox |
| Integration with existing sites | `book.<domain>` CNAME, button snippets, deep links, QR poster | Script widget (floating + inline), widget events, allowed origins | WordPress plugin |
| Platform billing | Plans, trial, Razorpay Subscriptions, SSPL GST invoices, proration, dunning, state machine (restrict/suspend/retain/delete), wallet top-up | Annual plan incentives, coupons for plans, reseller billing | Dedicated tier billing |
| Super Admin | Tenant list/detail, review queue, suspend/reinstate, plans, subscriptions, invoices, dunning, support grants, theme catalog, settings, health and business dashboards, deletion workflow | Announcements, cohort analytics | Cell provisioning, partner management |
| Reports | Dashboard KPIs, CSV exports | Revenue, occupancy heatmap, coupon, lead conversion, churn reports | Benchmarks across tenants (opt-in, anonymised) |
| Platform/infra | Shared multi-tenant on ECS + RDS, RLS, static sites on CloudFront, WAF, backups + DR copy, observability, CI/CD, pen test | Read replica for reporting, SQS for high-volume queues if needed | Dedicated cells, second pool, multi-region active-passive |
| Calendar | `.ics` + add-to-calendar links | Staff iCal feeds | Google/Outlook resource calendar push |

## 3. MVP milestone plan (indicative)

| Weeks | Milestone | Content |
|-------|-----------|---------|
| 1–3 | M0 Foundations | Sprint 0 items above; architecture spikes (CloudFront SaaS Manager, Razorpay OAuth, renderer), RLS + tenant context skeleton, CI with isolation test harness |
| 4–7 | M1 Tenant core | Sign-up, auth + MFA, tenant state, onboarding wizard shell, business/location/hours, branding, media pipeline |
| 8–11 | M2 Builder + publish | 2 themes, section editors, preview, site renderer, versions/rollback, platform subdomain publishing, review queue |
| 12–15 | M3 Inventory + booking | Products, units, pools, pricing, rules, availability engine, holds with constraints, booking app browse → hold |
| 16–18 | M4 Money | Razorpay OAuth + Checkout, webhooks, reconciliation, invoices + credit notes, cancellations/refunds, offline payments |
| 19–21 | M5 Operations | Front desk views, requests, leads/tours, notifications (email/SMS/WhatsApp), custom domains + SSL, platform billing + dunning, Super Admin essentials |
| 22–24 | M6 Hardening + pilot | Themes 3–4, accessibility and performance passes, pen test and fixes, DR drill, pilot with 10 tenants, launch |

Run the pilot with real tenants from week 18 (on staging with live Razorpay test mode first, then production) rather than waiting for week 22. Pilot feedback on onboarding and booking modes is worth more than any extra feature.

## 4. MVP launch checklist

- [ ] Cross-tenant isolation suite green for every endpoint; RLS enabled and forced on every tenant table
- [ ] Third-party penetration test done; all critical/high findings fixed
- [ ] Core Web Vitals and Lighthouse budgets met on all 4 themes (NFR-PERF-01–03)
- [ ] axe-core clean and manual screen reader pass on booking flow and themes
- [ ] Razorpay Partner approval live; end-to-end live payment and refund on a real tenant account
- [ ] DLT templates approved; WhatsApp templates approved; SES out of sandbox
- [ ] GST invoice and credit note formats reviewed by a CA
- [ ] Terms of service, DPA, privacy policy, refund policy reviewed by counsel (DPDP + consumer protection)
- [ ] Backups, PITR and a full restore drill documented
- [ ] On-call rota, runbooks (payments down, DNS/SSL issues, data breach, tenant suspension), status page live
- [ ] Support playbook and help-centre articles for top 20 expected questions (DNS per registrar, Razorpay connection)
- [ ] 10 pilot tenants live; at least 5 with paid online bookings

## 5. What we deliberately cut from MVP (and why)

| Cut | Why it's safe to wait |
|-----|----------------------|
| Memberships with credits | Pilot tenants can manage monthly members with monthly bookings + offline payments; build it with real usage data |
| Script widget | `book.<domain>` link covers existing-website tenants with less risk |
| PWA | Responsive web works; add install + push once booking volume shows repeat use |
| Floor plans | List selection is enough for most spaces under 100 seats |
| Peak pricing, recurring bookings, broadcasts | Nice to have; low pilot demand expected |
| Second gateway | Razorpay covers the market; add when a tenant can't use Razorpay |
