# 11 · Integrations

> Cowos requirements pack · v0.2 draft · Every external provider sits behind an internal interface (`IPaymentGateway`, `ISmsSender`, `IWhatsAppSender`, `IEmailSender`, `IGstVerifier`, `IGeoService`) so it can be swapped without touching domain code.

## 1. Summary

| Integration | Purpose | Recommended provider (v1) | Alternatives | Phase |
|-------------|---------|---------------------------|--------------|:---:|
| Payment gateway (tenant bookings) | Customers pay tenants directly | Razorpay (Partner OAuth on the tenant's own account) | Cashfree, PhonePe PG (P2) | P1 |
| Payment gateway (hosting invoices) | Tenants pay SSPL their monthly hosting cost share | Razorpay (SSPL account): Payment Links, optional UPI Autopay / e-mandate (D1, Pilot Rules #10) | Cashfree payment links / mandates | P1-B |
| AWS cost data | Source of the monthly hosting charge and daily estimate | AWS Data Exports (CUR 2.0) from the production account to S3 (R2) | — | P1-B |
| Edge routing table | Host → tenant, live version, state at the edge | CloudFront KeyValueStore, written only by the EdgeSyncWorker (R17) | Caddy on-demand TLS edge if the spike fails (R5) | P1 |
| SMS | OTP, booking confirmations, reminders | DLT-compliant CPaaS (e.g. MSG91, Gupshup, Kaleyra) | Twilio (higher India cost) | P1 |
| WhatsApp | Confirmations, reminders, payment links, OTP | Meta WhatsApp Cloud API directly, or a BSP (Gupshup, Interakt, AiSensy) | — | P1 (platform number), P2 (tenant numbers) |
| Email | Transactional email | Amazon SES (ap-south-1) | Postmark, SendGrid | P1 |
| Maps | Address autocomplete, map pin, embed, directions | Google Maps Platform | Mappls (MapmyIndia), OpenStreetMap + MapLibre | P1 |
| GSTIN verification | Validate tenant/customer GSTIN | Local format + checksum (P1); online legal-name lookup via a GST Suvidha Provider (GSP) or KYC API vendor (P1-B for the onboarding lookup, CEO-S2) | Manual check by reviewer | P1 / P1-B |
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
- **Hosting invoices (P1-B):** SSPL's own Razorpay account. SSPL charges no subscription, plan fee or commission, so Razorpay Subscriptions are not used for platform billing (D1). Each monthly HostingInvoice (issued on the 10th, due in 10 days) is paid by a Razorpay payment link, or by an optional UPI Autopay or e-mandate (eNACH) the tenant sets up, with a ₹5,000 per-tenant maximum and the RBI pre-debit notice before each debit (Pilot Rules #10). Pilots add a payment method with the 60-day end-of-free-year notice (Pilot Rules #6).

### 2.2 Flows used
| Flow | Razorpay feature |
|------|------------------|
| Instant booking | Orders API + Standard Checkout (modal) |
| Request-to-Book payment, staff "send payment link", membership renewals | Payment Links API (on tenant account) |
| Refunds | Refunds API (normal speed; instant refunds optional, at tenant's cost) |
| Reconciliation | Payments/Settlements fetch APIs |
| Hosting invoice payment (SSPL account, P1-B) | Payment Links API; UPI Autopay / e-mandate debits against a mandate the tenant set up (not Subscriptions) |
| Events | Webhooks: `payment.captured`, `payment.failed`, `order.paid`, `refund.processed`, `refund.failed`, `payment_link.paid`, plus payment link and mandate events on the SSPL account (P1-B) |

### 2.3 Requirements and risks
- Start the Razorpay Partner application in sprint 0; OAuth access and webhook setup per connected account need partner approval and can take weeks. Razorpay Partner live is part of the production entry gate for pilots (Pilot Rules #1).
- Staging (weeks 18-21) runs Razorpay in test mode only for the guided pilot preview; the E2E suite runs on staging in test mode (R6).
- For hosting invoices, confirm at build time which Razorpay API carries UPI Autopay and e-mandate debits on the SSPL account, and how the pre-debit notice is sent (P1-B).
- Webhooks for connected accounts: confirm in the spike whether partner-level webhooks cover connected accounts or each tenant account needs a webhook registered via API. Reconciliation jobs must cover gaps either way.
- Tenant KYC and activation happen at Razorpay. Tenants without an activated account can publish with Enquire / Request / offline-payment modes only.
- Test mode used for preview and demo sites; live mode required for Instant Book in production.
- Abstract behind `IPaymentGateway` with capabilities (`SupportsOAuth`, `SupportsPaymentLinks`, `SupportsInstantRefund`) so Cashfree/PhonePe can be added in Phase 2.

## 3. SMS (India)

- **TRAI DLT** registration is mandatory: SSPL registers as the principal entity, registers headers (sender IDs, e.g. `COWOSB`) and every template with variables (`{#var#}`). Unregistered content is blocked by operators.
- Templates (v1): OTP, booking confirmation, booking reminder, cancellation, refund, payment link, request approved/declined, tour confirmation. Tenant name is a variable.
- Use transactional/service-implicit category for booking messages; promotional SMS only with consent (P2 broadcasts).
- Delivery reports via webhook; cost per message recorded. Every SMS, OTPs included, is debited from the tenant's wallet at provider cost before sending, with no free allowance: notifications need the balance to stay at or above 0, sign-in OTPs may take it down to −₹500, then customers sign in by email OTP (Pilot Rules #10, R4). Pilots use an SSPL-funded wallet within the fair-use caps (Pilot Rules #5).
- DLT entity, header and template approvals are part of the production entry gate for pilots (Pilot Rules #1).
- Lead time: DLT entity + header + template approval typically takes days to a few weeks; do it in sprint 0.

## 4. WhatsApp

- **MVP:** one Cowos WhatsApp Business number (Meta-verified business: SSPL) sending utility and authentication templates with the tenant's name ("Your booking at *Acme Hub* is confirmed..."). Customer replies are routed to the tenant via email/admin notification with a "Reply on your WhatsApp" link to the tenant's own number.
- **Phase 2:** tenant-owned WhatsApp numbers via Embedded Signup (Meta Cloud API) or a BSP, so messages come from the tenant's number and the tenant can chat with customers.
- Pricing: Meta charges per delivered template message by category (marketing, utility, authentication) for India; utility templates sent inside an open customer service window may be free. Confirm current rates at build time and show them in the messaging price table.
- Opt-in captured at booking ("Get updates on WhatsApp", default on for transactional updates where counsel agrees it's covered by the booking; marketing opt-in separate and unticked). Honour STOP.
- Fallback: WhatsApp failed/undelivered within 2 minutes → send SMS for critical events (confirmation, OTP).
- Each WhatsApp message is debited from the tenant's wallet at provider cost under the same floors as SMS (Pilot Rules #10, R4). For pilot fair-use caps, a WhatsApp message followed by its SMS fallback counts as 2 (Pilot Rules #5).
- WhatsApp approvals are part of the production entry gate for pilots (Pilot Rules #1).

## 5. Email: Amazon SES

- Sending domain `notify.cowos.site` (tenant-facing) and `cowos.in` (platform mail), SPF, DKIM, DMARC (`p=quarantine` then `reject`), custom MAIL FROM.
- Configuration sets for bounce/complaint events → suppress addresses, show "email bouncing" on customer profiles.
- Request SES production access early (sandbox by default).
- Templates rendered by the Notifications module (MJML or React Email compiled to HTML) with tenant branding; plain-text alternative always included.
- Phase 2: tenant custom sending domain (DKIM CNAMEs added during domain setup).

## 6. Google Maps Platform

| Use | API | Cost note |
|-----|-----|-----------|
| Address autocomplete in onboarding (P1-B, FR-TA-004, CEO-S2) | Places API (Autocomplete + Place Details), session tokens | Per session; admin-only, low volume |
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
- Online verification (legal name, status Active/Cancelled, state, registration type) through a GSP or KYC API vendor, called during onboarding (legal-name lookup from P1-B, CEO-S2) and before first-publish approval; customer GSTINs verified on entry for B2B invoices (cached 30 days).
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

## 11. AWS Data Exports (CUR 2.0) (P1-B)

- **Purpose:** the source of each tenant's hosting charge. The charge is the production AWS account's monthly bill, net of AWS credits, split equally by tenant-days across counted tenants, with GST added (Pilot Rules #10, R2).
- **Setup:** a CUR 2.0 export from the production account only to an S3 bucket, refreshed daily. Staging, dev, CI, the marketing site and Super Admin costs are not in the pool; SSPL pays them.
- **Bill final check:** on the 10th the MonthlyHostingBilling job reads the export and proceeds only when the CUR 2.0 column `bill_invoice_id` is populated for the month (R1). If it is empty or the export is missing, invoices are held, SSPL Finance is alerted and the job retries daily.
- **Late changes:** the job re-reads the prior month each run. Credits, refunds or support fees AWS applied after invoicing go on the next invoice as an AdjustmentLine (R1).
- **Service breakdown:** the export's cost by service category is stored on the CostPool and shown on the tenant billing page with the tenant count, never other tenants' usage (5A).
- **Daily estimate:** ChargeEstimateService reads the month-to-date data after each daily refresh (R15).
- **Tests:** billing runs from fixture exports cover a non-final bill (held), proration by tenant-days, an adjustment line, pilot absorption, and invoice plus payment link.

## 12. CloudFront KeyValueStore (P1)

- **Purpose:** the edge routing table (host → tenant, live version, state) read by the viewer-request CloudFront Function (08 §6). Depends on the sprint-0 spike passing (R5); otherwise the Caddy edge replaces it.
- **Single writer (R17):** only the EdgeSyncWorker writes KVS. Publish, rollback, data refresh, domain, state and pilot-creation jobs add an `edge_sync` outbox row instead of calling KVS.
- **UpdateKeys with IfMatch:** `UpdateKeys` needs the whole store's current ETag in `IfMatch`, so concurrent writers get 409. The worker re-reads the ETag (`DescribeKeyValueStore`) and retries on 409, and batches many tenants into one `UpdateKeys` call within KVS batch limits.
- **Value written:** the tenant's current DB pointer and state, read at send time, so a retry can never write a stale value.
- **Tests:** KVS writes forced into the reverse of the DB commit order end with KVS equal to DB; a worker restart mid-batch resends current values; 409 handling retries with a fresh ETag.
