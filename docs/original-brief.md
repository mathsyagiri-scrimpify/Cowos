# Cowos · Original Brief (as given by the product owner, 2026-10-30)

> This is the brief the requirements pack in `docs/requirements/` was written from. Kept verbatim (lightly formatted) so later readers can see what was asked versus what was recommended. Clarifications given afterwards are listed at the end.

## 1. Product context

Reference model: Directoo (https://www.directoo.in/) gives restaurants their own branded website (e.g. https://chantabbai.com/, https://pallevindu.in/) and a separate branded ordering app on a subdomain (e.g. https://order.chantabbai.com/order, https://order.pallevindu.in/order), so restaurants take direct orders without aggregator commissions and own their customer data.

Cowos applies this model to coworking spaces. Target customers (tenants) are coworking spaces that do not have their own website and/or booking system. Cowos lets them:
- Register on the Cowos platform
- Configure and generate a branded website and a branded booking app without writing code
- If they already have a website, embed/integrate only the Cowos booking app into it (via subdomain link, embeddable widget, or script snippet — evaluate the options)
- Preview everything before publishing
- Publish on their own domain or a Cowos-provided subdomain
- Let their own customers (members/visitors) book seats, desks, cabins, and meeting rooms and pay online

Every tenant website and booking app must show a "Powered by SSPL" footer badge linking to Cowos.

## 2. Applications in scope

1. Cowos Marketing Web App – public site for Cowos itself: product info, pricing, tenant sign-up/registration
2. Tenant Admin Portal – where a coworking space owner builds and manages their website, booking app, inventory, pricing, bookings, customers, and payments
3. Tenant Booking App + Website – the customer-facing, tenant-branded experience (responsive web; evaluate PWA vs native mobile for later phases)
4. Cowos Super Admin Portal – for SSPL staff to manage all tenants, subscriptions/server billing, publishing approvals, support, platform settings, and monitoring

## 3. Tech stack (fixed)
- Backend: .NET (latest LTS, ASP.NET Core Web API)
- Frontend: React (with TypeScript)
- Database: PostgreSQL
- Hosting: AWS

## 4. Tenant onboarding and site builder – information to capture
Design the onboarding/builder flow to collect, at minimum:
- Business details: name, legal name, GST number, contact, address, operating hours, amenities, policies
- Branding: logo, favicon, primary/secondary/accent colors, font family, button shape (rounded/pill/square), theme (light/dark)
- Website content: header (nav, CTA), hero section, about, amenities, gallery images, pricing/plans, testimonials, FAQ, contact form, footer content, social links
- Location: address, Google Maps embed/coordinates, directions
- Booking inventory: workspace types (hot desk, dedicated desk, private cabin, meeting room, event space, virtual office), number of seats/rooms, capacity, floor layout (optional), photos, per-hour/day/week/month pricing, add-ons, availability rules, buffer times, cancellation and refund policy
- Payments: tenant's payment gateway setup (recommend a model, e.g. Razorpay with tenant's own keys or Razorpay Route for split settlements), taxes/GST on invoices

Ask for a template-based approach (several prebuilt themes that the tenant customizes) rather than a free-form drag-and-drop builder for v1, and justify the choice.

## 5. Preview and publish flow
- Live preview of website and booking app (desktop and mobile views) before publishing
- Draft vs published versions, with the ability to republish after edits and roll back
- At publish time, ask for the domain:
  - If the tenant owns a domain: website on their domain, booking app on booking.<tenantdomain>; include DNS setup instructions (CNAME/A records), domain verification, and automatic SSL
  - If the tenant has no domain: provide a platform subdomain such as <tenant>.cowos.in and booking.<tenant>.cowos.in (or similar — recommend a structure)
- Before publishing, collect payment for the tenant's server hosting

## 6. Pricing and hosting transparency (important)
- We charge tenants only for server (AWS EC2) hosting cost — no commission on their bookings. This must be stated clearly and transparently in the UI, with a cost breakdown.
- Our initial idea is one EC2 instance per tenant. Compare this against (a) a shared multi-tenant deployment with per-tenant cost allocation and (b) a hybrid model (shared by default, dedicated for larger tenants). Cover cost, scalability, isolation, security, deployment complexity, and operations, and recommend one for v1.
- Define billing: monthly/annual plans, invoices with GST, payment reminders, grace period, and what happens to a tenant's site when payment lapses (suspension, data retention).

## 7. Customer (end-user) features in the booking app
Registration/login (phone OTP and email), browse spaces and availability calendar, book by hour/day/month, select seat/room, apply coupons, pay online, receive confirmation via email/SMS/WhatsApp, view/cancel/reschedule bookings, invoices download, membership plans, and notifications.

## 8. Deliverables requested
1. Product Vision & Scope – problem statement, goals, target users, value proposition vs. coworking aggregators, in-scope/out-of-scope, assumptions
2. User Personas and roles with a permissions matrix (Super Admin, SSPL support, Tenant Owner, Tenant Staff/Front-desk, Customer)
3. Business Requirements Document (BRD)
4. Software Requirements Specification (SRS) – functional requirements per application, numbered (e.g. FR-TA-001), each with priority (MoSCoW)
5. User stories with acceptance criteria (Given/When/Then), grouped into epics
6. Key user flows: tenant registration → build → preview → pay → publish; existing-website integration; customer booking → payment → confirmation; cancellation/refund; tenant suspension
7. Non-functional requirements: multi-tenancy and data isolation, security (OWASP, PCI-DSS scope minimization), performance, availability, scalability, backups, logging/monitoring, accessibility (WCAG 2.1 AA), SEO for tenant websites, data privacy (India's DPDP Act 2023)
8. High-level architecture: multi-tenancy strategy (schema-per-tenant vs shared schema with TenantId — recommend one), tenant resolution by hostname, custom domain and SSL automation, file/image storage (S3 + CDN), notifications, background jobs, CI/CD
9. Data model: core entities and relationships (Tenant, Domain, Theme, SiteContent, Space, Resource, Pricing, Booking, Payment, Customer, Invoice, Subscription, etc.)
10. API module list (no full specs, just modules and key endpoints)
11. Integrations: payment gateway, SMS/WhatsApp, email, Google Maps, calendar (Google/Outlook) sync
12. Release plan: MVP (Phase 1) vs Phase 2 vs Phase 3, with a rough feature split
13. Risks, open questions, and decisions to make

## 9. How to work
- Before writing, list clarifying questions for anything ambiguous and state the assumptions used if not answered.
- Be specific to coworking spaces, not generic booking software.
- Use tables where they help (permissions matrix, requirement lists, phase plan).
- Flag anything technically risky, expensive, or likely to cause problems, and suggest a better alternative.

Closing request: "Can you prepare better requirement document for above prompt. If my idea is not good then you can suggest better ideas."

## Clarifications given after the first draft

1. **Promo codes:** tenants can add their own promo codes in the Tenant Admin.
2. **0% commission:** means Cowos does not charge the tenant any commission. What the tenant charges its own customers is the tenant's decision, not Cowos's.
3. **Location of documents:** everything lives at project level under `docs/`, not inside a Claude worktree folder.
