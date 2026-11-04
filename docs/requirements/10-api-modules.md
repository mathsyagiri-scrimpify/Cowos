# 10 · API Module List

> Cowos requirements pack · v0.3 draft · Modules and key endpoints only; full OpenAPI specs come during design. Endpoints are P1 unless marked.

## 1. Conventions

| Area | Prefix | Tenant from | Auth |
|------|--------|-------------|------|
| Public (website + booking app) | `/api/v1/public/...` | Host (`X-Cowos-Host` set by CloudFront) | Anonymous or customer token |
| Customer account | `/api/v1/me/...` | Host | Customer token |
| Tenant admin | `/api/v1/admin/...` | Token claim (`tid`) | Tenant user session (BFF cookie) |
| Platform (Super Admin) | `/api/v1/platform/...` | Explicit `{tenantId}` in path | SSPL SSO session |
| Marketing site (`cowos.in`) | `/api/v1/marketing/...` | n/a (platform data, no tenant) | Anonymous, rate-limited |
| Webhooks | `/webhooks/{provider}/...` | Resolved from provider account | Provider signature |
| Internal (renderer) | `/internal/...` | n/a | IAM / private network only |

- JSON, camelCase; errors as RFC 9457 problem details; cursor pagination (`?cursor=&limit=`).
- `Idempotency-Key` header required on POSTs that create holds, bookings, payments, refunds, invoices, sign-ups and wallet top-ups.
- Optimistic concurrency with `If-Match` / ETag on drafts and configuration resources, including draft rate card versions.
- Tenant state guard: Removed in v0.3. Trial, PastDue, Restricted and Suspended are all in use from launch (Pricing Rules #7, #8).
- Publishing while Suspended returns 409 with "Your site is suspended. Contact SSPL support." and the reason (CEO-A1).
- Every request runs in one explicit transaction that sets the tenant first (08 §5.2, R3).

## 2. Modules and key endpoints

### 2.1 Identity & Auth

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/v1/auth/tenant/signup` | Create tenant in Trial + Owner (marketing sign-up); the Owner accepts the ToS and DPA (FR-MW-005, Pricing Rules #7, #9) |
| GET | `/api/v1/auth/invitations/{token}` | Check a staff invitation; expired or used → 410 (FR-TA-172) |
| POST | `/api/v1/auth/invitations/{token}/accept` | Accept: phone OTP, set password and MFA. Single use |
| POST | `/api/v1/auth/otp/send` · `/verify` | Email/phone OTP (invite acceptance, sign-up, customer login) |
| POST | `/api/v1/auth/tenant/login` · `/logout` · `/refresh` | Tenant user session (BFF) |
| POST | `/api/v1/auth/tenant/mfa/setup` · `/mfa/verify` | TOTP |
| POST | `/api/v1/auth/password/forgot` · `/reset` | |
| POST | `/api/v1/public/auth/otp/send` · `/verify` | Customer login on tenant host |
| GET | `/api/v1/auth/platform/login` | SSO redirect for SSPL staff |

### 2.2 Tenancy & Onboarding

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/v1/public/slugs/{slug}/availability` | Slug check (FR-MW-012) |
| GET/PUT | `/api/v1/admin/tenant` | Business details (GSTIN format/checksum and PAN derivation) |
| POST | `/api/v1/admin/tenant/gstin/verify` | GSTIN legal-name lookup (CEO-S2) |
| GET | `/api/v1/admin/onboarding` | Owner "left to do" checklist status (FR-TA-002): Razorpay KYC/OAuth, GSTIN/PAN, DNS, ToS, DPA; each item re-checked on load |
| GET | `/api/v1/admin/onboarding/handoff` | Removed in v0.3 (first-login handoff for pilots removed with the pre-handover setup grant, R19) |
| PUT | `/api/v1/admin/onboarding/steps/{step}` | Save wizard step (4A, CEO-S2) |
| GET | `/api/v1/admin/tenant/limits` | Removed in v0.2 (no plan limits, D1) |

### 2.3 Sites (builder, preview, publish)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/v1/admin/themes` | Available themes: every released theme, for every tenant (D1, CEO-S3) |
| GET/PUT | `/api/v1/admin/site/branding` | Branding tokens |
| PUT | `/api/v1/admin/site/theme` | Switch theme |
| GET/PUT | `/api/v1/admin/site/draft` | Draft content (ETag) |
| PATCH | `/api/v1/admin/site/draft/sections/{sectionId}` | Edit one section |
| CRUD | `/api/v1/admin/site/pages` | Custom pages |
| POST | `/api/v1/admin/site/preview-links` | Shareable preview token |
| POST | `/api/v1/admin/site/publish/checks` | Run pre-publish checks |
| POST | `/api/v1/admin/site/publish` | Publish (first → review); 409 while Suspended (CEO-A1) |
| GET | `/api/v1/admin/site/versions` | History |
| POST | `/api/v1/admin/site/versions/{n}/restore` | Roll back; 409 while Suspended (CEO-A1) |
| POST | `/api/v1/admin/media/uploads` | Presigned upload |
| GET/PATCH/DELETE | `/api/v1/admin/media/{id}` | Alt text, delete |
| GET | `/internal/sites/{tenantId}/versions/{n}/render-input` | Renderer input |

### 2.4 Domains

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/v1/admin/domains` | List incl. platform domain |
| POST | `/api/v1/admin/domains` | Add custom domain (+ booking placement) |
| GET | `/api/v1/admin/domains/{id}/instructions` | Provider-specific DNS records |
| POST | `/api/v1/admin/domains/{id}/check` | Check DNS now |
| DELETE | `/api/v1/admin/domains/{id}` | Remove |

### 2.5 Inventory

| Method | Endpoint | Purpose |
|--------|----------|---------|
| CRUD | `/api/v1/admin/locations` | Locations |
| PUT | `/api/v1/admin/locations/{id}/hours` | Operating hours |
| CRUD | `/api/v1/admin/locations/{id}/holidays` | Holidays |
| CRUD | `/api/v1/admin/products` | Products (type, mode, rules) |
| CRUD | `/api/v1/admin/products/{id}/units` · POST `.../units:bulk` | Units |
| PUT | `/api/v1/admin/products/{id}/prices` | Price rules |
| CRUD | `/api/v1/admin/products/{id}/addons` | Add-ons |
| CRUD | `/api/v1/admin/products/{id}/availability-rules` · `/maintenance-blocks` | Availability |
| CRUD | `/api/v1/admin/tour-slot-rules` | Tour availability |
| CRUD | `/api/v1/admin/policies/cancellation` | Cancellation policies |
| CRUD | `/api/v1/admin/policies/documents` | Privacy/terms/house rules |

### 2.6 Public catalog & availability (booking app, website)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/v1/public/site-config` | Branding tokens, locations, products summary, policies, contacts |
| GET | `/api/v1/public/products?locationId=` | Catalog |
| GET | `/api/v1/public/products/{slug}` | Product detail |
| GET | `/api/v1/public/products/{id}/availability?from=&to=&quantity=&duration=` | Start times for a chosen duration, merged across units unless a unit is chosen (1A) / day capacity / monthly availability |
| POST | `/api/v1/public/quotes` | Price breakdown for a selection (coupon, add-ons, GST) |
| GET | `/api/v1/public/tours/availability?locationId=&date=` | Tour slots |
| POST | `/api/v1/public/leads` | Contact form / enquiry / tour request |

### 2.7 Booking

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/v1/public/bookings/holds` | Create hold (Instant) + payment order, after phone OTP (customer token required); re-quotes the price and returns any difference before Razorpay opens; if the slot was taken, returns the next free times (2A) |
| GET | `/api/v1/me/checkout/{bookingRef}/status` | Resumable checkout state for UPI return or reload; requires the customer's session, another customer gets 404 (9A, R16) |
| GET | `/api/v1/public/passes/{token}` | Desk pass: opaque purpose-bound token; returns first name + last initial, date/time, room, logo and a short-lived check-in code; expired, revoked, wrong-tenant or wrong-purpose → not found (17A, R10) |
| DELETE | `/api/v1/public/bookings/holds/{id}` | Release hold |
| POST | `/api/v1/public/bookings/requests` | Request to Book |
| GET | `/api/v1/me/bookings` · `/me/bookings/{id}` | Customer bookings |
| POST | `/api/v1/me/bookings/{id}/cancel-preview` · `/cancel` | Refund preview, cancel |
| POST | `/api/v1/me/bookings/{id}/reschedule-preview` · `/reschedule` | Reschedule |
| GET | `/api/v1/admin/bookings?from=&to=&status=&productId=` | Calendar/list |
| GET | `/api/v1/admin/bookings/today` | Front desk view |
| POST | `/api/v1/admin/bookings` | Staff-created booking |
| POST | `/api/v1/admin/bookings/{id}/approve` · `/decline` | Requests |
| POST | `/api/v1/admin/bookings/{id}/cancel` | Cancel (with optional override) |
| POST | `/api/v1/admin/bookings/{id}/reschedule` | |
| POST | `/api/v1/admin/bookings/{id}/check-in` · `/no-show` | Front desk |
| POST | `/api/v1/admin/check-ins/scan` | Validate the pass QR check-in code against the scanning tenant only (R10) |
| GET | `/api/v1/admin/bookings/{id}/timeline` | Event history |

### 2.8 Payments

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/v1/admin/payment-gateways` | Connections |
| GET | `/api/v1/admin/payment-gateways/razorpay/oauth/start` · `/callback` | Partner OAuth |
| POST | `/api/v1/admin/payment-gateways/razorpay/keys` | Fallback key entry |
| DELETE | `/api/v1/admin/payment-gateways/{id}` | Disconnect |
| POST | `/api/v1/public/payments/verify` | Client-side success verification |
| POST | `/api/v1/public/payment-links/{token}/order` | Pay an approved request / invoice |
| GET | `/api/v1/admin/payments` | Payments list |
| POST | `/api/v1/admin/payments/offline` | Record offline payment |
| POST | `/api/v1/admin/refunds` · GET `/admin/refunds` | Refunds |
| POST | `/webhooks/razorpay/tenant` | Tenant-account events (payment, refund, order) |
| POST | `/webhooks/razorpay/platform` | SSPL account events: server invoice payment links, UPI Autopay / e-mandate debits and wallet top-ups (Pricing Rules #5, #6) |

### 2.9 Invoicing

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET/PUT | `/api/v1/admin/invoice-settings` | Series, prefixes, footer, bank details |
| GET | `/api/v1/admin/invoices` · `/admin/invoices/{id}` | List/detail |
| GET | `/api/v1/admin/invoices/{id}/pdf` | Presigned PDF |
| POST | `/api/v1/admin/invoices/{id}/resend` | Email/WhatsApp again |
| POST | `/api/v1/admin/invoices/{id}/credit-notes` | Manual credit note |
| GET | `/api/v1/admin/exports/gstr1?period=` (P2) | GST export |
| GET | `/api/v1/me/invoices` · `/me/invoices/{id}/pdf` | Customer invoices |

### 2.10 CRM (customers, companies, leads, privacy)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| CRUD | `/api/v1/admin/customers` | Customers |
| POST | `/api/v1/admin/customers/import` | CSV import (async) |
| CRUD | `/api/v1/admin/companies` · `/companies/{id}/members` | Companies |
| CRUD | `/api/v1/admin/leads` · POST `/leads/{id}/activities` · `/leads/{id}/convert` | Leads |
| CRUD | `/api/v1/admin/tours` | Tours |
| GET/POST | `/api/v1/admin/data-requests` · `/data-requests/{id}/complete` | DPDP queue |
| GET/PUT | `/api/v1/me/profile` · `/me/consents` | Customer profile, consent |
| POST | `/api/v1/me/data-requests` | Access / erasure requests |
| CRUD | `/api/v1/me/company/members` (P2) | Company admin |

### 2.11 Memberships & Promotions

| Method | Endpoint | Purpose |
|--------|----------|---------|
| CRUD | `/api/v1/admin/coupons` | Coupons |
| POST | `/api/v1/public/coupons/validate` | Validate coupon in quote |
| CRUD | `/api/v1/admin/membership-plans` (P2) | Plans |
| CRUD | `/api/v1/admin/memberships` · POST `/memberships/{id}/credits/adjust` (P2) | Memberships, credits |
| GET | `/api/v1/me/memberships` (P2) | Customer membership + credits |

### 2.12 Notifications

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET/PUT | `/api/v1/admin/notification-settings` | Channel toggles per event |
| GET/PUT | `/api/v1/admin/notification-templates/{event}/{channel}` | Email customisation |
| GET | `/api/v1/admin/messages?bookingId=` | Delivery log |
| POST | `/webhooks/sms/{provider}` · `/webhooks/whatsapp/{provider}` · `/webhooks/ses` | Delivery receipts, inbound STOP |

### 2.13 Staff, audit, support, reports

| Method | Endpoint | Purpose |
|--------|----------|---------|
| CRUD | `/api/v1/admin/staff` · POST `/staff/invitations` | Staff |
| POST | `/api/v1/admin/owner-transfer` | Ownership transfer |
| GET | `/api/v1/admin/audit-events` | Tenant audit log |
| GET/POST | `/api/v1/admin/support-grants` · `/support-grants/{id}/approve` · `/revoke` | Support access |
| GET | `/api/v1/admin/dashboard` | Dashboard KPIs |
| GET | `/api/v1/admin/reports/{report}?from=&to=` | Reports (P2 for most) |
| POST | `/api/v1/admin/exports` · GET `/admin/exports/{id}` | Full data export |

### 2.14 Platform billing (tenant side)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/v1/admin/billing` | Billing page (5A): month-to-date units per server meter with slab breakdown and estimate (updated daily, "as of"), last invoice and its status, next issue date (the 1st), payment method, "₹0 commission" line, wallet summary; trial end date during Trial. Never other tenants' usage (Pricing Rules #5, #7) |
| GET | `/api/v1/public/platform-plans` | Removed in v0.2 (no plans, D1) |
| POST | `/api/v1/admin/billing/subscription` · PATCH · DELETE | Removed in v0.2 (no subscriptions, D1) |
| PUT | `/api/v1/admin/billing/payment-method` | Payment link details or UPI Autopay / e-mandate (₹5,000 maximum, RBI pre-debit notice); adding one moves a Trial tenant to Active (Pricing Rules #5, #7) |
| GET | `/api/v1/admin/billing/invoices` · `/{id}` · `/{id}/pdf` | SSPL server invoices: one line per server meter with units and slab breakdown, GST, due date, status (Pricing Rule #5) |
| GET | `/api/v1/admin/billing/wallet` | Wallet balance and transactions; month-to-date count and current slab per notification channel (Pricing Rule #6) |
| POST | `/api/v1/admin/billing/wallet/top-ups` | Wallet top-up through Razorpay on SSPL's account (Pricing Rule #6) |
| GET | `/api/v1/admin/billing/usage?month=` | Daily usage per meter for a month: the detail behind an invoice or the month to date (NFR-MTR-01, Pricing Rules #3, #5) |
| GET | `/api/v1/admin/billing/rate-card` | The rate card version in force: slab table per meter (Pricing Rule #4) |

### 2.15 Platform (Super Admin)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/v1/platform/tenants?state=&q=` | Tenant list |
| GET | `/api/v1/platform/tenants/{id}` | Tenant detail |
| GET | `/api/v1/platform/reviews` · POST `/reviews/{id}/approve` · `/reject` · `/request-changes` | First-publish queue |
| POST | `/api/v1/platform/tenants/{id}/suspend` · `/reinstate` · `/unpublish` | Enforcement (FR-SA-011, with reason, audited). Suspend and reinstate raise `TenantStateChanged`, which triggers the TenantStateCtaService render (CEO-A1). A repeat suspend is a no-op |
| POST | `/api/v1/platform/tenants/{id}/trial-extension` · `/credits` | Commercial actions: trial extension, server invoice credits (Pricing Rules #5, #7) |
| POST | `/api/v1/platform/tenants/{id}/support-grants` | Request access |
| POST | `/api/v1/platform/tenants/{id}/deletion` · `/deletion/{id}/approve` | Two-person deletion |
| POST | `/api/v1/platform/pilots` | Removed in v0.3 (Create pilot removed with the pilot programme; tenants sign up self-serve, Pricing Rule #9) |
| POST | `/api/v1/platform/pilots/{tenantId}/owner-invite` | Removed in v0.3 (no Owner invite by SSPL, Pricing Rule #9) |
| GET | `/api/v1/platform/demo-requests` · PATCH `/demo-requests/{id}` | Demo requests list and status (CEO-A2) |
| GET | `/api/v1/platform/scorecard?month=` · `/scorecard.csv` | Tenant scorecard: all live tenants per month and roll-up with the expansion gate (10 tenants at 10+ paid online bookings for 3 consecutive months), "as of" time; CSV export (CEO-A4, R20) |
| CRUD | `/api/v1/platform/plans` | Removed in v0.2 (no plans, D1) |
| GET | `/api/v1/platform/billing/cost-pools` · `/cost-pools/{month}` | Removed in v0.3 (no AWS-cost split; see `/platform/cost-view`, Pricing Rule #10) |
| GET | `/api/v1/platform/billing/server-invoices` · `/dunning` | Finance: server invoices with lines and slab breakdown, payment status, dunning (Pricing Rules #5, #8; FR-SA-023) |
| CRUD | `/api/v1/platform/themes` | Theme catalog |
| GET/PUT | `/api/v1/platform/settings` · `/feature-flags` | Settings |
| GET | `/api/v1/platform/metrics/health` | Health dashboard |
| GET | `/api/v1/platform/metrics/business` | Sign-ups, trials, conversions, churn, referrals (FR-SA-051, CEO-A4) |
| GET | `/api/v1/platform/metrics/cost-recovery` | Removed in v0.3 (replaced by `/platform/cost-view`, Pricing Rule #10) |
| GET | `/api/v1/platform/audit-events` | Global audit |
| GET/POST | `/api/v1/platform/rate-cards` · GET/PUT `/rate-cards/{id}` · POST `/rate-cards/{id}/publish` | Rate card versions (Super Admin edits; Finance views): slab table per meter, `effective_from` the 1st of a month. Only a Draft can be changed; PUT on a Published version → 409. Publish checks every meter has a complete, contiguous slab table and rebuilds the pricing page (Pricing Rule #4, NFR-MTR-02) |
| GET | `/api/v1/platform/tenants/{id}/usage?month=` | A tenant's daily usage per meter (Super Admin, Finance) (NFR-MTR-01) |
| GET | `/api/v1/platform/billing/rating-runs` · POST `/rating-runs/{month}/retry` | Rating runs per month: status, tenants rated, invoices issued, failures; retry a failed run, which is idempotent (Pricing Rule #5, NFR-MTR-05) |
| GET | `/api/v1/platform/cost-view?month=` | SSPL cost view: production AWS cost (CUR 2.0) and messaging-provider cost next to billed server and notification revenue per month (Super Admin; Finance views). Not used for tenant bills (Pricing Rule #10) |

### 2.16 Internal / edge

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/internal/edge/routes/sync` | Removed in v0.2: no job or endpoint pushes to KeyValueStore; the EdgeSyncWorker job does (§2.18, R17) |
| POST | `/internal/renderer/jobs/{id}/complete` | Renderer callback; the API then runs the pointer switch with the state check and the `data_seq` rule (08 §8, CEO-A1, CEO-A3, R18) |
| GET | `/healthz` · `/readyz` | Health checks |

### 2.17 Marketing site (`cowos.in`)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/v1/marketing/demo-requests` | Book-a-demo form (FR-MW-008): honeypot, 5 per IP per hour, invisible challenge; validates name, phone (+91 E.164), email, space, city, message ≤ 2,000 characters (400 with inline errors); saves the request before notifying sales through the email outbox; same phone within 24 h updates the existing request; DB unavailable → 503 and no email (CEO-A2) |

The pricing page shows the rate card version in force (slab tables per meter) and the savings calculator. It is rebuilt when a version is published and when a version takes effect on the 1st (08 §12, Pricing Rule #4), not served by an API call.

### 2.18 Worker jobs with no HTTP endpoint

| Job | Purpose |
|-----|---------|
| EdgeSyncWorker | Single consumer of the `edge_sync` outbox: reads each tenant's current DB pointer and state and writes them to CloudFront KeyValueStore with `UpdateKeys` and `IfMatch`, retrying on 409, batching tenants (08 §6, R17) |
| Request log metering | Daily per IST day: CloudFront standard logs from S3 → host to tenant → request usage in UsageDaily, excluding WAF-blocked, SSPL monitoring and `/api/*` requests (08 §4.1, NFR-MTR-03 to 05) |
| Storage snapshot | Daily: each tenant's stored media and files for storage units (08 §4.1) |
| Month-to-date estimate | Daily after metering: units, slab breakdown and estimate for the billing page (Pricing Rule #5) |
| Rating | 1st of each month (IST), after the last day's metering: previous month's server usage → slabs of the version in force → ServerInvoice + lines + GST → payment link or mandate debit (08 §4.1, Pricing Rules #4, #5) |
| Pricing page rebuild | When a rate card version is published or takes effect: rebuilds the marketing pricing page from the rate card in force (Pricing Rule #4) |
| SSPL cost view load | Daily after the CUR 2.0 refresh: PlatformCostMonth for the cost view (Pricing Rule #10) |
| Server dunning | Daily 06:00 IST: reminders and automatic Restricted / Suspended moves (BRD §7.3, Pricing Rule #8) |
