# 14 · Build Plan: order, sprints and user stories

> Cowos requirements pack · v0.4 draft · 2026-11-04 · For a small team using Claude Code. Week numbers are estimates: re-plan after sprint 3, when the real pace is known.

The product owner wants every app finished before Cowos is sold to tenants, and as soon as possible. This plan builds all six apps with **4 developers who use Claude Code to write most of the code**, a part-time designer, and the product owner accepting work every week. **Launch target: week 20** [estimated]; launch tenants are told "launching week 20–24". **Checkpoint at the end of week 4:** the first paid booking must be demoed end to end; if not, re-plan from the measured pace and the owner chooses between moving the date and cutting from §6 (CEO review 3, R3-P1). With 3 developers, expect about week 24. The full-team plan in 12 §3 (week 28) stays as the comparison.

## 1. How we go fast

1. **Thin slices, end to end.** Each sprint finishes something a person can use, through every layer: database, API, screen and test. The first paid booking works in week 4, not month 4.
2. **Claude Code writes, people decide.** For each story the developer gives Claude Code the story, its FRs and the repo's `CLAUDE.md`. Claude Code writes the tests from the acceptance criteria first, then the code. The developer reviews every diff, and a second developer reviews the pull request.
3. **One-week sprints, Friday demo.** The product owner sees working software every Friday and accepts or rejects each story within a day. A rejected story goes back to the top of next week.
4. **Decide in week 1, not later.** D3–D16 are still Proposed (13 §3). The pack's recommended defaults are used unless the owner changes them by the end of week 1. D3, D4 and D6 block day-one work.
5. **Start slow approvals on day 1.** Razorpay Partner, DLT, WhatsApp templates, SES production access, domains and the pen-test booking take weeks no matter how fast we code (§5).
6. **No new scope until launch.** New ideas go in `TODOS.md`. Only bugs and the cut list in §6 change the plan.

## 2. Team

| Who | Owns | Works mostly in |
|-----|------|-----------------|
| Dev A, backend lead | Tenancy, auth, RLS, availability engine, bookings, payments | `api/` |
| Dev B, admin frontend | Tenant Admin: wizard, inventory, site builder, seat map builder, front desk | `web/apps/tenant-admin` |
| Dev C, customer frontend | Booking app, tenant website renderer and 2 themes, seat picker | `web/apps/booking-app`, `web/apps/renderer`, `web/packages/themes` |
| Dev D, platform and billing | Infrastructure, CI/CD, notifications, wallet, both plans and rating, Super Admin, marketing site | `infra/`, `api/` billing modules, `web/apps/super-admin`, `web/apps/marketing` |
| Designer (part-time) | DESIGN.md tokens, 2 launch themes, seat map builder and picker screens, ahead of the sprint that builds them | Figma |
| Product owner | Accepts stories, sets prices (OQ35, OQ21), recruits the first tenants, chases approvals | |

Every developer uses Claude Code. QA is part of every story (tests written first, end-to-end flows in Playwright); the owner and the first tenants do the hands-on testing.

## 3. Build order of the apps

| Order | App | Why this position | Weeks |
|:-:|-----|-------------------|-------|
| 1 | **Platform core** (API, database, tenancy, auth, RLS) | Every app depends on it; tenant isolation must be right from the first table | 1–2 |
| 2 | **Tenant Admin** | The owner must set up a space before anyone can book; it then grows every sprint | 2 onward |
| 3 | **Booking app** | It is the money path. Getting the first real booking and payment working early removes the biggest risk | 3–5 |
| 4 | **Tenant website + 2 themes** | Shows the inventory that already exists; publishing and domains come right after | 6–7 |
| 5 | **Seat maps** (builder in Tenant Admin, picker in the booking app) | Builds on products, units and holds from weeks 3–5 | 8–9 |
| 6 | **SSPL billing** (Fixed and Usage plans) and **Super Admin** | No one pays SSPL until the trial ends, so billing can come later, but it must be finished before the first real tenants in week 17. The first-publish review queue is built earlier, in week 7 | 11–13 |
| 7 | **Marketing site (cowos.in)** | Sign-up is built in week 2 as a simple page; the full site with the two-plan pricing page needs the rate card, so it comes last | 2, 13 |

## 4. Sprint plan

Stories are in `05-user-stories.md`; the FRs they cite are in `04`. "New" marks stories still to write (§7) before their sprint starts.

| Week | Sprint goal (Friday demo) | User stories | Who does what |
|------|---------------------------|--------------|---------------|
| 1 | **Sprint 0: foundations.** Empty apps deploy to staging on every merge | none (setup, §8) | A: .NET solution, modules, auth skeleton, tenant context + RLS. B and C: React app shells, shared UI package from DESIGN.md tokens, generated API client. D: Terraform (app server, RDS, S3 + CloudFront), CI/CD, isolation test harness, CloudFront multi-tenant spike |
| 2 | **An owner can sign up and set up the business** | US-1.1, US-1.2, US-1.3, US-1.4; new: staff invite | A: sign-up, OTP, Trial state. B: wizard steps 1–4, home checklist. C: sign-up page on cowos.in, booking app shell. D: SES email, media pipeline, audit log |
| 3 | **The owner can list what they sell** | US-4.1, US-4.2, US-4.3, US-4.4 | A: products, units, pools, prices, availability engine. B: inventory and policy screens. C: booking app home, product page, start-times list. D: holds with the exclusion constraint, demo inventory |
| 4 | **First paid booking end to end** (Razorpay test mode) | US-6.2, US-6.1, US-5.1; new: desk pass | A: OTP login, hold, Razorpay order, webhook. B: Razorpay connect screen. C: checkout, confirmation, desk pass. D: confirmation email, `.ics`, Playwright booking flow |
| 5 | **The money is right** | US-5.2, US-5.3, US-5.4, US-8.1, US-8.3, US-8.4 | A: GST invoices, credit notes, refunds, reconciliation. B: payments and invoices screens, offline payment. C: My bookings, cancel and reschedule. D: invoice PDFs, idempotency tests |
| 6 | **The owner's website** | US-2.1, US-2.2, US-2.3, US-2.4 | C: renderer, themes Loft and Metro, booking shells. B: section editors, branding with contrast check, device preview. A: site versions API. D: S3 publishing, CloudFront |
| 7 | **Go live on the owner's own domain** | US-3.1, US-10.1, US-3.2, US-3.3, US-11.1 | B: publish checks, versions, rollback, domain screens. D: custom domains + SSL, review queue in Super Admin. A: publish pipeline, `data_seq`, edge sync. C: `book.<domain>` booking app |
| 8–9 | **Seat maps** | US-4.6, US-4.7, US-4.9, US-6.4 | B: seat map builder (templates, paint, labels, assign, publish), "Change seat" on the booking. C: seat picker, list view. A: seat units in availability and holds, publish rules, auto-assign for older bookings (R3-A1), seat locks (R3-D1). Owner tests the builder with 3 real owners (R31) |
| 8–9 (parallel) | **Notifications and enquiries** | US-9.7, US-7.1, US-7.2, US-7.3; new: notification settings | D: SMS (DLT), WhatsApp, wallet with atomic debit, delivery log. A (week 9): Request to Book, tours, leads pipeline |
| 10 | **Front desk, customers and offers** | US-8.2, US-4.5, US-6.3, US-12.1, US-12.2; new: dashboard | B: today view, QR check-in, customers and companies, promo codes, dashboard. C: coupon at checkout, privacy requests. A: exports, data requests. D: reminders and notification templates |
| 11–12 | **SSPL billing with both plans** | US-9.1, US-9.8, US-9.9, US-9.6, US-9.2, US-9.3, US-9.4, US-10.7 | D: rate card with Fixed fee, allowances and slabs (plain editor with typed publish; drafts, diff and impact preview after launch, R3-C3), hold-and-alert for a missing plan period (R3-E1), metering jobs, rating on the 1st, server invoices. A: tenant states, dunning, Restricted and Suspended moves. B: plan choice at activation, billing page, plan switch, wallet page. C: Restricted and suspended pages in the themes |
| 13 | **Super Admin and the marketing site** | US-10.2, US-10.3, US-10.5, US-10.6 | D: Super Admin tenant list, support grants, scorecard (cost view and business dashboard follow launch, R3-C2, R3-C5). C: cowos.in pages, two-plan pricing page, sample bills, demo form, help centre. A and B: gaps from earlier demos |
| 14 | **Feature freeze: finish and polish** | Rejected or carried-over stories | Everyone: fix what the owner rejected; accessibility pass (axe + screen reader); performance budgets (NFR-PERF) |
| 15–16 | **Hardening** | none (launch gate, 12 §4.1) | Isolation suite on every endpoint, external pen test and fixes, backup restore drill, app server recovery and rebuild test, CA review of invoices, counsel review of terms and policies |
| 17–18 | **First 3–5 real tenants on production** | none | Owners sign up themselves, draw seat maps, take real bookings; the team fixes what they hit within a day |
| 19–20 | **Launch: 10 tenants live** | none | Launch checklist (12 §4.2) |

## 5. Things outside the code that set the date

| Item | Start | Needed by | Owner |
|------|-------|-----------|-------|
| Decisions D3–D16 confirmed (defaults otherwise) | Week 1 | End of week 1 | Product owner |
| Buy `cowos.in` and `cowos.site` (D4) | Week 1 | Week 2 | Product owner |
| Razorpay Partner application (D3) | Week 1 | Week 15 (live payments) | Product owner |
| TRAI DLT registration and SMS templates | Week 1 | Week 9 (test), week 17 (live) | Dev D |
| WhatsApp Business templates | Week 2 | Week 9 (test), week 17 (live) | Dev D |
| SES production access | Week 2 | Week 4 | Dev D |
| Designs: DESIGN.md, 2 themes, seat map screens | Week 1 | Weeks 2, 6 and 8 | Designer |
| Pen test booked | Week 6 | Weeks 15–16 | Product owner |
| Counsel: terms, DPA, privacy and refund templates | Week 6 | Week 16 | Product owner |
| CA review of invoice and credit note formats | Week 10 | Week 16 | Product owner |
| Fixed plan fee, allowances and slab prices (OQ35, OQ21) | Week 8 | Week 12 | Product owner |
| 10 launch tenants recruited | Week 4 | Week 17 | Product owner |

## 6. If we fall behind

Check progress every Friday against this plan, and hold the week-4 checkpoint (§1). If a sprint slips, cut in this order rather than moving the launch:
1. "Last month on the other plan" comparison on the billing page → after launch.
2. Reports beyond the dashboard and CSV exports.
3. The third and later seat map templates (keep "Rows of desks" and "Blank").

Already moved after launch by CEO review 3 (2026-11-05), see TODOS.md: SSPL cost view (R3-C2), rate card drafts, diff and impact preview (R3-C3), Today on the map and "Seat us together" (R3-C4), business dashboard (R3-C5).

Nothing on the production entry gate (12 §4.1) is ever cut.

## 7. Stories to write before their sprint

The pack has no stories yet for these P1 FRs. Write them, with Given/When/Then, the week before they are built:

| Sprint | Story to write | FRs |
|--------|----------------|-----|
| 2 | Invite my staff and give them a role | FR-TA-172 |
| 4 | Show my desk pass at the door | FR-BA-037, FR-BA-019 |
| 8 | Choose which messages customers get | FR-TA-170, FR-TA-171 |
| 10 | See my day on the dashboard | FR-TA-173 |

## 8. Week 1 checklist (Sprint 0)

- Repository layout (one monorepo):
  - `api/`: ASP.NET Core (.NET 10 LTS) modular monolith with one module per area in 08 (Tenancy, Sites, Inventory, Booking, Payments, Invoicing, Crm, Notifications, PlatformBilling, PlatformAdmin) and a Hangfire worker host.
  - `web/` (pnpm workspace): `apps/tenant-admin`, `apps/booking-app`, `apps/super-admin`, `apps/marketing`, `apps/renderer`, `packages/ui`, `packages/api-client` (generated from OpenAPI), `packages/themes`.
  - `infra/` (Terraform) and `e2e/` (Playwright).
- `CLAUDE.md` in the code repo: stack, module rules, "tests first from acceptance criteria", RLS rules (R3), naming, how to run tests. Every Claude Code session reads it.
- CI on every pull request: build, unit tests, integration tests on real PostgreSQL (Testcontainers), the tenant isolation suite, lint and type-check. Merge to `main` deploys to staging.
- Definition of done for a story: every acceptance criterion has a test that failed before the code existed; CI green; reviewed by a second developer; deployed to staging; accepted by the product owner at the Friday demo.

## 9. Stories by app (where each app's work is)

| App | Stories |
|-----|---------|
| Platform core (API) | All stories touch it; isolation and state rules in US-1.1, US-9.3, US-9.4 |
| Marketing site | US-1.1 (sign-up), US-9.8 (pricing page), US-10.5 (demo form) |
| Tenant Admin | US-1.2–1.4, US-2.1–2.4, US-3.1–3.3, US-4.1–4.7, US-4.9 (US-4.8 after launch), US-5.1–5.4, US-7.3, US-8.2–8.4, US-9.1, US-9.2, US-9.7–9.9, US-12.2 |
| Tenant website | US-2.1–2.4, US-3.1–3.3, US-9.3 (Restricted CTAs) |
| Booking app | US-6.1–6.4, US-7.1, US-7.2, US-8.1, US-8.4, US-11.1, US-12.1 |
| Super Admin | US-10.1–10.3, US-10.5–10.7, US-9.4 (US-10.8 after launch) |
