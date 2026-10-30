# 11 · Integrations

> Cowos requirements pack · v0.1 draft · Every external provider sits behind an internal interface (`IPaymentGateway`, `ISmsSender`, `IWhatsAppSender`, `IEmailSender`, `IGstVerifier`, `IGeoService`) so it can be swapped without touching domain code.

## 1. Summary

| Integration | Purpose | Recommended provider (v1) | Alternatives | Phase |
|-------------|---------|---------------------------|--------------|:---:|
| Payment gateway (tenant bookings) | Customers pay tenants directly | Razorpay (Partner OAuth on the tenant's own account) | Cashfree, PhonePe PG (P2) | P1 |
| Payment gateway (platform billing) | Tenants pay SSPL | Razorpay (SSPL account) with Subscriptions | Cashfree Subscriptions | P1 |
| SMS | OTP, booking confirmations, reminders | DLT-compliant CPaaS (e.g. MSG91, Gupshup, Kaleyra) | Twilio (higher India cost) | P1 |
| WhatsApp | Confirmations, reminders, payment links, OTP | Meta WhatsApp Cloud API directly, or a BSP (Gupshup, Interakt, AiSensy) | — | P1 (platform number), P2 (tenant numbers) |
| Email | Transactional email | Amazon SES (ap-south-1) | Postmark, SendGrid | P1 |
| Maps | Address autocomplete, map pin, embed, directions | Google Maps Platform | Mappls (MapmyIndia), OpenStreetMap + MapLibre | P1 |
| GSTIN verification | Validate tenant/customer GSTIN | A GST Suvidha Provider (GSP) or KYC API vendor | Manual check by reviewer | P1 |
| Calendar | Add booking to customer calendar | `.ics` + Google Calendar link | Google/Outlook API push (P3) | P1 / P3 |
| Analytics (tenant) | Tenant's own tracking | GA4, GTM, Meta Pixel by ID | — | P1 |
| Error monitoring | Frontend + backend errors | Sentry | CloudWatch RUM | P1 |
| Help desk | Tenant support tickets | Freshdesk or Zoho Desk | Built-in simple tickets | P1 |
| Malware scan | Uploaded files | ClamAV in worker, or a managed S3 malware scanning service | — | P1 |
| E-invoicing (IRN) | Large tenants' B2B invoices | GSP integration | — | P3 |
| Accounting | Export / sync | Tally XML export, Zoho Books API | — | P3 |
| Google Business Profile | Local SEO | Guidance + link (no API in v1) | GBP API for hours/posts sync | P1 / P3 |
| Access control / Wi-Fi | Door access, captive portal | Partner integrations (vendor APIs) | — | P3 |

## 2. Payment gateway: Razorpay

### 2.1 Model (see 00 §C4)
- **Tenant bookings:** Cowos registers as a Razorpay Technology Partner. Tenants connect their own Razorpay account through OAuth. Cowos creates orders and refunds on the tenant's account with a scoped access token. Settlement goes to the tenant's bank. SSPL never holds funds and takes no commission.
- **Fallback:** tenant enters key ID + secret; encrypted with a dedicated KMS key.
- **Platform billing:** SSPL's own Razorpay account; Razorpay Subscriptions with UPI Autopay, e-mandate (eNACH) and card mandates (RBI tokenisation; additional factor authentication applies above the e-mandate threshold).

### 2.2 Flows used
| Flow | Razorpay feature |
|------|------------------|
| Instant booking | Orders API + Standard Checkout (modal) |
| Request-to-Book payment, staff "send payment link", membership renewals | Payment Links API (on tenant account) |
| Refunds | Refunds API (normal speed; instant refunds optional, at tenant's cost) |
| Reconciliation | Payments/Settlements fetch APIs |
| Events | Webhooks: `payment.captured`, `payment.failed`, `order.paid`, `refund.processed`, `refund.failed`, `payment_link.paid`, plus subscription events on SSPL account |

### 2.3 Requirements and risks
- Start the Razorpay Partner application in sprint 0; OAuth access and webhook setup per connected account need partner approval and can take weeks.
- Webhooks for connected accounts: confirm in the spike whether partner-level webhooks cover connected accounts or each tenant account needs a webhook registered via API. Reconciliation jobs must cover gaps either way.
- Tenant KYC and activation happen at Razorpay. Tenants without an activated account can publish with Enquire / Request / offline-payment modes only.
- Test mode used for preview and demo sites; live mode required for Instant Book in production.
- Abstract behind `IPaymentGateway` with capabilities (`SupportsOAuth`, `SupportsPaymentLinks`, `SupportsInstantRefund`) so Cashfree/PhonePe can be added in Phase 2.

## 3. SMS (India)

- **TRAI DLT** registration is mandatory: SSPL registers as the principal entity, registers headers (sender IDs, e.g. `COWOSB`) and every template with variables (`{#var#}`). Unregistered content is blocked by operators.
- Templates (v1): OTP, booking confirmation, booking reminder, cancellation, refund, payment link, request approved/declined, tour confirmation. Tenant name is a variable.
- Use transactional/service-implicit category for booking messages; promotional SMS only with consent (P2 broadcasts).
- Delivery reports via webhook; cost per message recorded and debited from the tenant's wallet (OTP cost absorbed by the platform plan, decision D7).
- Lead time: DLT entity + header + template approval typically takes days to a few weeks; do it in sprint 0.

## 4. WhatsApp

- **MVP:** one Cowos WhatsApp Business number (Meta-verified business: SSPL) sending utility and authentication templates with the tenant's name ("Your booking at *Acme Hub* is confirmed..."). Customer replies are routed to the tenant via email/admin notification with a "Reply on your WhatsApp" link to the tenant's own number.
- **Phase 2:** tenant-owned WhatsApp numbers via Embedded Signup (Meta Cloud API) or a BSP, so messages come from the tenant's number and the tenant can chat with customers.
- Pricing: Meta charges per delivered template message by category (marketing, utility, authentication) for India; utility templates sent inside an open customer service window may be free. Confirm current rates at build time and show them in the messaging price table.
- Opt-in captured at booking ("Get updates on WhatsApp", default on for transactional updates where counsel agrees it's covered by the booking; marketing opt-in separate and unticked). Honour STOP.
- Fallback: WhatsApp failed/undelivered within 2 minutes → send SMS for critical events (confirmation, OTP).

## 5. Email: Amazon SES

- Sending domain `notify.cowos.site` (tenant-facing) and `cowos.in` (platform mail), SPF, DKIM, DMARC (`p=quarantine` then `reject`), custom MAIL FROM.
- Configuration sets for bounce/complaint events → suppress addresses, show "email bouncing" on customer profiles.
- Request SES production access early (sandbox by default).
- Templates rendered by the Notifications module (MJML or React Email compiled to HTML) with tenant branding; plain-text alternative always included.
- Phase 2: tenant custom sending domain (DKIM CNAMEs added during domain setup).

## 6. Google Maps Platform

| Use | API | Cost note |
|-----|-----|-----------|
| Address autocomplete in onboarding | Places API (Autocomplete + Place Details), session tokens | Per session; admin-only, low volume |
| Map pin placement in admin | Maps JavaScript API | Admin-only |
| Map on tenant website | **Maps Embed API** (iframe, no per-load charge) or a static map image that links to Google Maps | Use Embed/static to avoid per-load JS map costs across thousands of visitors |
| "Get directions" | Plain `https://www.google.com/maps/dir/?api=1&destination=lat,lng` link | Free |

- Restrict API keys by HTTP referrer and API; separate keys for admin and public.
- Load the map lazily (FR-TW-010).
- Alternative: Mappls for India-specific address accuracy if Google costs become material.

## 7. Calendar

| Phase | Capability |
|-------|------------|
| P1 | `.ics` attachment on confirmation and reschedule emails (with `METHOD:REQUEST`, stable `UID`, `SEQUENCE` increments on change, `METHOD:CANCEL` on cancellation); "Add to Google Calendar" and "Add to Outlook" links on confirmation page |
| P2 | Tenant staff calendar feed (read-only iCal URL per location/product) so staff can subscribe in Google/Outlook |
| P3 | One-way push of meeting-room bookings to the tenant's Google Workspace / Microsoft 365 resource calendars (OAuth per tenant, service account or Graph app). Two-way sync (external events block Cowos availability) only if tenants ask for it; requires watch channels/subscriptions and conflict rules |

## 8. GSTIN verification

- Format + checksum validation locally (free, instant).
- Online verification (legal name, status Active/Cancelled, state, registration type) through a GSP or KYC API vendor, called during onboarding and before first-publish approval; customer GSTINs verified on entry for B2B invoices (cached 30 days).
- Fallback if the API is down: accept with "Unverified" flag; reviewer checks manually on the public GST portal.

## 9. Analytics and marketing tags (tenant-owned)

- Tenants enter IDs only (GA4 measurement ID, GTM container ID, Meta Pixel ID). Cowos injects the official snippets after consent (FR-TW-011) and fires standard events from the booking app: `view_item`, `begin_checkout`, `purchase` (value, currency, booking reference; no personal data).
- Cowos's own product analytics (admin portal usage) via a privacy-respecting tool (e.g. PostHog self-hosted or Plausible), never on tenant public sites without disclosure.

## 10. Other integrations

| Integration | Notes |
|-------------|-------|
| Sentry | Separate projects per app; tenant tag; PII scrubbing on |
| Help desk | Tenant identity passed via signed widget token; ticket links to tenant ID in Super Admin |
| Malware scanning | Scan before media becomes public; quarantine bucket for flagged files |
| E-invoicing (P3) | For tenants above the e-invoicing turnover threshold: generate IRN + signed QR via GSP before invoice issue; store IRN, ack number |
| Tally / Zoho Books (P3) | Tally: XML voucher export per period. Zoho Books: OAuth, push invoices and payments |
| Google Business Profile | P1: guidance and deep link to add the website and booking URL to the GBP listing ("Book" button link). P3: GBP API to sync hours and holidays |
| Access control (P3) | Webhook-based integration with door access vendors (generate PIN/QR at check-in), Wi-Fi vouchers |
