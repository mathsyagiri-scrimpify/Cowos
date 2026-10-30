# 10 · API Module List

> Cowos requirements pack · v0.1 draft · Modules and key endpoints only; full OpenAPI specs come during design.

## 1. Conventions

| Area | Prefix | Tenant from | Auth |
|------|--------|-------------|------|
| Public (website + booking app) | `/api/v1/public/...` | Host (`X-Cowos-Host` set by CloudFront) | Anonymous or customer token |
| Customer account | `/api/v1/me/...` | Host | Customer token |
| Tenant admin | `/api/v1/admin/...` | Token claim (`tid`) | Tenant user session (BFF cookie) |
| Platform (Super Admin) | `/api/v1/platform/...` | Explicit `{tenantId}` in path | SSPL SSO session |
| Webhooks | `/webhooks/{provider}/...` | Resolved from provider account | Provider signature |
| Internal (renderer, edge sync) | `/internal/...` | n/a | IAM / private network only |

- JSON, camelCase; errors as RFC 9457 problem details; cursor pagination (`?cursor=&limit=`).
- `Idempotency-Key` header required on POSTs that create holds, bookings, payments, refunds, invoices.
- Optimistic concurrency with `If-Match` / ETag on drafts and configuration resources.

## 2. Modules and key endpoints

### 2.1 Identity & Auth

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/v1/auth/tenant/signup` | Create tenant + Owner (marketing sign-up) |
| POST | `/api/v1/auth/otp/send` · `/verify` | Email/phone OTP (sign-up, customer login) |
| POST | `/api/v1/auth/tenant/login` · `/logout` · `/refresh` | Tenant user session (BFF) |
| POST | `/api/v1/auth/tenant/mfa/setup` · `/mfa/verify` | TOTP |
| POST | `/api/v1/auth/password/forgot` · `/reset` | |
| POST | `/api/v1/public/auth/otp/send` · `/verify` | Customer login on tenant host |
| GET | `/api/v1/auth/platform/login` | SSO redirect for SSPL staff |

### 2.2 Tenancy & Onboarding

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/v1/public/slugs/{slug}/availability` | Slug check |
| GET/PUT | `/api/v1/admin/tenant` | Business details |
| POST | `/api/v1/admin/tenant/gstin/verify` | GSTIN lookup |
| GET | `/api/v1/admin/onboarding` | Checklist status |
| PUT | `/api/v1/admin/onboarding/steps/{step}` | Save step |
| GET | `/api/v1/admin/tenant/limits` | Plan limits and usage |

### 2.3 Sites (builder, preview, publish)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/v1/admin/themes` | Available themes (by plan) |
| GET/PUT | `/api/v1/admin/site/branding` | Branding tokens |
| PUT | `/api/v1/admin/site/theme` | Switch theme |
| GET/PUT | `/api/v1/admin/site/draft` | Draft content (ETag) |
| PATCH | `/api/v1/admin/site/draft/sections/{sectionId}` | Edit one section |
| CRUD | `/api/v1/admin/site/pages` | Custom pages |
| POST | `/api/v1/admin/site/preview-links` | Shareable preview token |
| POST | `/api/v1/admin/site/publish/checks` | Run pre-publish checks |
| POST | `/api/v1/admin/site/publish` | Publish (first → review) |
| GET | `/api/v1/admin/site/versions` | History |
| POST | `/api/v1/admin/site/versions/{n}/restore` | Roll back |
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
| GET | `/api/v1/public/products/{id}/availability?from=&to=&quantity=` | Slots / day capacity / monthly availability |
| POST | `/api/v1/public/quotes` | Price breakdown for a selection (coupon, add-ons, GST) |
| GET | `/api/v1/public/tours/availability?locationId=&date=` | Tour slots |
| POST | `/api/v1/public/leads` | Contact form / enquiry / tour request |

### 2.7 Booking

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/v1/public/bookings/holds` | Create hold (Instant) + payment order |
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
| POST | `/api/v1/admin/check-ins/scan` | Validate QR payload |
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
| POST | `/webhooks/razorpay/platform` | SSPL account events (subscriptions) |

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
| GET | `/api/v1/admin/billing` | Plan, next invoice, usage, fee allocation |
| GET | `/api/v1/public/platform-plans` | Plans for marketing + admin |
| POST | `/api/v1/admin/billing/subscription` · PATCH · DELETE | Start, change, cancel |
| GET | `/api/v1/admin/billing/invoices` · `/{id}/pdf` | SSPL invoices |
| POST | `/api/v1/admin/billing/wallet/top-ups` | Messaging wallet top-up |

### 2.15 Platform (Super Admin)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/v1/platform/tenants?state=&plan=&q=` | Tenant list |
| GET | `/api/v1/platform/tenants/{id}` | Tenant detail |
| GET | `/api/v1/platform/reviews` · POST `/reviews/{id}/approve` · `/reject` · `/request-changes` | First-publish queue |
| POST | `/api/v1/platform/tenants/{id}/suspend` · `/reinstate` · `/unpublish` | Enforcement |
| POST | `/api/v1/platform/tenants/{id}/trial-extension` · `/credits` | Commercial actions |
| POST | `/api/v1/platform/tenants/{id}/support-grants` | Request access |
| POST | `/api/v1/platform/tenants/{id}/deletion` · `/deletion/{id}/approve` | Two-person deletion |
| CRUD | `/api/v1/platform/plans` | Plans and versions |
| GET | `/api/v1/platform/invoices` · `/dunning` | Finance |
| CRUD | `/api/v1/platform/themes` | Theme catalog |
| GET/PUT | `/api/v1/platform/settings` · `/feature-flags` · `/messaging-rates` | Settings |
| GET | `/api/v1/platform/metrics/business` · `/metrics/health` | Dashboards |
| GET | `/api/v1/platform/audit-events` | Global audit |

### 2.16 Internal / edge

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/internal/edge/routes/sync` | Push host → tenant/version/state into CloudFront KeyValueStore |
| POST | `/internal/renderer/jobs/{id}/complete` | Renderer callback |
| GET | `/healthz` · `/readyz` | Health checks |
