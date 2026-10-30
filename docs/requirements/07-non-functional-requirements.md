# 07 · Non-Functional Requirements

> Cowos requirements pack · v0.1 draft · IDs `NFR-<area>-<nn>`; all are MVP (Must) unless marked.

## 1. Multi-tenancy and data isolation

| ID | Requirement |
|----|-------------|
| NFR-MT-01 | Every tenant-owned row carries a non-null `tenant_id`. Tables without it are platform-global and contain no tenant personal data. |
| NFR-MT-02 | Isolation is enforced in two independent layers: (1) EF Core global query filters bound to the request's tenant context, and (2) PostgreSQL Row-Level Security policies using a session variable (`SET LOCAL app.tenant_id`) set per transaction. The application DB role cannot bypass RLS (`NOBYPASSRLS`); migrations use a separate owner role. |
| NFR-MT-03 | Tenant context is derived from the hostname (public) or a signed token claim (admin), never from a client-supplied header or body field. |
| NFR-MT-04 | Background jobs carry `tenant_id` explicitly in their payload and set the RLS session variable before touching data; jobs that iterate tenants process one tenant per transaction. |
| NFR-MT-05 | Cache keys, S3 object keys (`tenants/{tenantId}/...`), search indexes, log entries and metrics include tenant ID. |
| NFR-MT-06 | An automated cross-tenant test suite runs in CI: for every API endpoint, a request authenticated as tenant A attempting to read/write tenant B's resource IDs must return 404. Adding an endpoint without coverage fails the build. |
| NFR-MT-07 | Per-tenant rate limits (API requests, OTP sends, holds, contact form submissions) protect against noisy neighbours and abuse. |
| NFR-MT-08 | The architecture supports moving a tenant to a dedicated cell (own database, optional own compute) by updating the tenant catalog and running a data migration job, with no application code change (Phase 3 capability, designed in P1). |

## 2. Security

### 2.1 Application security (OWASP ASVS Level 2 as the target, OWASP Top 10 coverage)

| ID | Requirement |
|----|-------------|
| NFR-SEC-01 | Broken access control: server-side authorization on every endpoint via policy-based authorization (role + tenant + location scope). Deny by default. |
| NFR-SEC-02 | Authentication: tenant users with ASP.NET Core Identity (PBKDF2 or Argon2 hashing), MFA (TOTP) mandatory for Owners; lockout after 5 failed attempts with backoff; breached-password check (k-anonymity API) on set/reset. Super Admin via SSO + MFA only. |
| NFR-SEC-03 | Customer OTP: 6 digits, 5-minute expiry, max 5 attempts per code, max 5 sends per number per hour and per IP limits; OTP stored hashed. |
| NFR-SEC-04 | Sessions: short-lived access tokens (15 min) + rotating refresh tokens (httpOnly, Secure, SameSite=Lax, host-only cookies). Admin portals use the BFF pattern (tokens never in `localStorage`). |
| NFR-SEC-05 | Injection: parameterised queries only (EF Core / Dapper with parameters); no dynamic SQL from user input. Rich text sanitised server-side with an allowlist (HtmlSanitizer). |
| NFR-SEC-06 | XSS: React escaping by default; `dangerouslySetInnerHTML` banned by lint rule except in the sanitised rich-text renderer. Strict Content Security Policy on all surfaces (nonces for inline scripts; tenant analytics IDs mapped to known script hosts only). |
| NFR-SEC-07 | Tenant sites served from a separate registrable domain (`cowos.site`) from the admin portal (`cowos.in`) to prevent cookie tossing and session attacks (see 00 §C5). |
| NFR-SEC-08 | SSRF: no server-side fetch of tenant-supplied URLs except through an allowlisted egress proxy (e.g. for importing images by URL, if added). |
| NFR-SEC-09 | File uploads: type sniffing (not extension trust), size limits, malware scan (ClamAV or a managed scanner) before publish, images re-encoded, SVG sanitised or rasterised. |
| NFR-SEC-10 | Secrets: AWS Secrets Manager; gateway credentials encrypted with a dedicated KMS key usable only by the payment service role; no secrets in code, config files or logs. |
| NFR-SEC-11 | Webhooks: signature verification (HMAC), replay protection by event ID, raw payload stored for audit. |
| NFR-SEC-12 | Security headers: HSTS (with preload on `cowos.in`), `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, `frame-ancestors` per surface. |
| NFR-SEC-13 | Dependency and image scanning in CI (Dependabot/Renovate, `dotnet list package --vulnerable`, npm audit, container scan); critical findings block release. SAST (CodeQL or Semgrep) on every PR. |
| NFR-SEC-14 | AWS WAF on CloudFront with managed rule sets, bot control on OTP and contact-form endpoints, geo rules optional. |
| NFR-SEC-15 | Annual third-party penetration test; one before public launch. |
| NFR-SEC-16 | Least-privilege IAM per service; no long-lived IAM user keys; CI deploys via OIDC federation. |
| NFR-SEC-17 | Support impersonation only through tenant-approved grants (02 §3.3), with dual-identity audit records. |

### 2.2 PCI-DSS scope minimisation

| ID | Requirement |
|----|-------------|
| NFR-PCI-01 | Card data is entered only into Razorpay Checkout (Razorpay-hosted iframe/modal). Cowos never receives, stores, processes or logs PAN, CVV or expiry. |
| NFR-PCI-02 | Cowos targets SAQ-A eligibility: payment pages load the gateway script from the gateway domain only; CSP restricts script sources; integrity of payment page scripts monitored (PCI DSS v4.0 req. 6.4.3 and 11.6.1). |
| NFR-PCI-03 | Stored payment data limited to gateway IDs (order, payment, refund), method type, last 4 digits/UPI handle masked as returned by the gateway, amounts and statuses. |
| NFR-PCI-04 | Recurring platform billing uses gateway-side tokenisation/mandates (RBI tokenisation rules), no card-on-file at Cowos. |

## 3. Performance

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-PERF-01 | Tenant website Core Web Vitals (field data, p75, mobile) | LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1 |
| NFR-PERF-02 | Tenant website Lighthouse mobile performance score on a published demo site | ≥ 90 |
| NFR-PERF-03 | Tenant website page weight (home, excluding hero media) | ≤ 300 KB transferred; JS ≤ 50 KB gzip |
| NFR-PERF-04 | Booking app initial JS bundle | ≤ 200 KB gzip; route-level code splitting |
| NFR-PERF-05 | Booking app time to interactive on a mid-range Android on 4G (Moto G Power class, simulated) | ≤ 3.5 s |
| NFR-PERF-06 | Public API latency (availability, holds) | p95 ≤ 300 ms, p99 ≤ 800 ms at design load |
| NFR-PERF-07 | Admin API latency | p95 ≤ 500 ms |
| NFR-PERF-08 | Publish time (click → live) for a typical site | ≤ 2 minutes p95 |
| NFR-PERF-09 | Builder preview refresh after an edit | ≤ 1 s |
| NFR-PERF-10 | Notification dispatch after booking confirmation | ≤ 60 s p95 |
| NFR-PERF-11 | Design load (year 3) | 1,500 tenants, 50 req/s sustained public API, 300 req/s peak, 5,000 bookings/hour peak |

## 4. Availability and resilience

| ID | Requirement |
|----|-------------|
| NFR-AV-01 | Tenant websites: 99.95% monthly availability (static on S3 + CloudFront; independent of API health). |
| NFR-AV-02 | Booking app and public API: 99.9% monthly (≈43 min downtime/month). |
| NFR-AV-03 | Admin portals: 99.5% monthly, maintenance windows announced 48h ahead, Sunday 02:00–05:00 IST. |
| NFR-AV-04 | Multi-AZ for database (RDS Multi-AZ) and compute (tasks across ≥2 AZs). |
| NFR-AV-05 | Zero-downtime deploys (blue/green or rolling with health checks); database migrations backward-compatible (expand/contract). |
| NFR-AV-06 | Graceful degradation: if messaging providers fail, bookings still confirm and notifications retry; if the gateway is down, booking app shows a clear message and offers Request-to-Book where enabled. |
| NFR-AV-07 | Public status page (`status.cowos.in`) for tenants. |

## 5. Scalability

| ID | Requirement |
|----|-------------|
| NFR-SC-01 | Stateless API and worker containers that scale horizontally on CPU/requests/queue depth. |
| NFR-SC-02 | Database sized for year-1 load on a single primary; read replica added for reporting when CPU > 60% sustained. PgBouncer / RDS Proxy for connection pooling. |
| NFR-SC-03 | Large tables (bookings, notification logs, audit logs) designed for time-based partitioning once they exceed ~50 M rows. |
| NFR-SC-04 | Static websites scale with the CDN; no per-tenant compute is needed to serve them. |
| NFR-SC-05 | Cell-based scale-out path: when `pool-1` reaches agreed limits, new tenants go to `pool-2` (separate DB), keeping blast radius bounded. |

## 6. Backups and disaster recovery

| ID | Requirement |
|----|-------------|
| NFR-BK-01 | RDS automated backups with point-in-time recovery, 35-day retention. |
| NFR-BK-02 | Daily snapshot copied to a second region (ap-south-2 Hyderabad) and to a separate AWS backup account (protects against account compromise / ransomware). |
| NFR-BK-03 | S3 versioning on media and site buckets; cross-region replication of media. |
| NFR-BK-04 | RPO ≤ 15 minutes; RTO ≤ 4 hours for a full region loss (static sites keep serving from CDN cache during DR). |
| NFR-BK-05 | Quarterly restore drill to a scratch environment with a documented result. |
| NFR-BK-06 | Per-tenant restore: ability to restore one tenant's data to a point in time via a restore-to-staging + selective copy runbook (needed for "I deleted my products" tickets). |

## 7. Logging, monitoring and alerting

| ID | Requirement |
|----|-------------|
| NFR-OBS-01 | Structured JSON logs (Serilog) with correlation ID, tenant ID, user ID (pseudonymous), route; no OTPs, tokens, secrets, or full phone/email in logs (masked). |
| NFR-OBS-02 | OpenTelemetry traces and metrics from API and workers, exported to CloudWatch / X-Ray or a managed Grafana stack. |
| NFR-OBS-03 | Frontend error monitoring (e.g. Sentry) for all React apps, with tenant tag and release version. |
| NFR-OBS-04 | Business metrics: bookings/min, hold-to-confirm conversion, payment failure rate, webhook lag, notification delivery rate per channel. |
| NFR-OBS-05 | Alerts (PagerDuty/Opsgenie or SNS → on-call): API 5xx > 1% for 5 min, p95 latency breach, queue age > 5 min, payment webhook failures, certificate expiry < 14 days, DB CPU > 80%, disk, replication lag, failed backups. |
| NFR-OBS-06 | Synthetic checks every 5 minutes: demo tenant website, booking availability, hold creation (test tenant), admin login. |
| NFR-OBS-07 | Audit logs retained 3 years (append-only, separate table/stream, tamper-evident hashing optional). Application logs retained 30 days hot, 1 year archive. |
| NFR-OBS-08 | Per-tenant cost visibility via AWS cost allocation tags on shared resources plus usage metering (requests, storage, messages) for the cost-to-serve KPI. |

## 8. Accessibility (WCAG 2.1 AA)

| ID | Requirement |
|----|-------------|
| NFR-A11Y-01 | All themes, the booking app and admin portals meet WCAG 2.1 AA. |
| NFR-A11Y-02 | Colour choices validated for 4.5:1 text contrast and 3:1 UI component contrast at input time (FR-TA-022). |
| NFR-A11Y-03 | Keyboard operable everywhere: date pickers, slot grids, modals (focus trap and return), Razorpay Checkout launch. |
| NFR-A11Y-04 | Semantic HTML, one `h1` per page, landmark regions, labelled form controls, error messages linked to fields (`aria-describedby`). |
| NFR-A11Y-05 | Alt text required for content images (FR-TA-024); decorative images marked as such. |
| NFR-A11Y-06 | Slot availability conveyed by text and pattern, not colour alone. |
| NFR-A11Y-07 | Respects `prefers-reduced-motion`; no auto-playing video with sound; hero video has pause control. |
| NFR-A11Y-08 | Automated axe-core checks in CI for every theme and booking flow; manual screen reader pass (NVDA + TalkBack) before each theme release. |
| NFR-A11Y-09 | Touch targets ≥ 44×44 px in booking app. |

## 9. SEO for tenant websites

| ID | Requirement |
|----|-------------|
| NFR-SEO-01 | Pre-rendered HTML with all content present without JavaScript. |
| NFR-SEO-02 | One canonical host per tenant (custom domain when live; 301 from platform subdomain and from apex). |
| NFR-SEO-03 | JSON-LD `LocalBusiness` data (name, address, geo, hours, phone, images, price range, sameAs social links) and `FAQPage`; `Product`/`Offer` for spaces where valid. |
| NFR-SEO-04 | Auto-generated `sitemap.xml` (with `lastmod`) and `robots.txt`; trial and preview hosts `noindex`. |
| NFR-SEO-05 | Clean, stable URLs (`/spaces/meeting-rooms`, `/book/p/boardroom`), no hash routing on the website. |
| NFR-SEO-06 | Responsive images with `srcset`, width/height attributes, lazy loading below the fold, WebP/AVIF. |
| NFR-SEO-07 | Title and meta description defaults generated from business data ("Coworking space in Madhapur, Hyderabad · Acme Hub") and editable. |
| NFR-SEO-08 | Guidance in admin to link the website from the tenant's Google Business Profile (the biggest local SEO lever) and to verify in Google Search Console (verification token field). |
| NFR-SEO-09 | Booking app pages that should rank (product pages) server-render meta tags at the edge; checkout and account pages `noindex`. |

## 10. Data privacy: DPDP Act 2023 and DPDP Rules 2025

Roles: the **tenant** is the Data Fiduciary for its customers' personal data; **SSPL** acts as Data Processor for that data, and as Data Fiduciary for tenant users' and marketing-site visitors' data. Confirm exact obligations and the phased commencement dates of the Rules with counsel.

| ID | Requirement |
|----|-------------|
| NFR-PRIV-01 | Data Processing Agreement between SSPL and each tenant, accepted at sign-up (versioned, timestamped). |
| NFR-PRIV-02 | Notice at collection: customer sign-up and contact/tour forms show a clear notice (what data, purpose, how to withdraw consent, how to exercise rights, grievance contact, how to complain to the Data Protection Board) in plain English (Indian languages in Phase 3). |
| NFR-PRIV-03 | Consent: separate, unticked, granular consent for marketing (WhatsApp/SMS/email). Booking-related messages rely on the performance-of-contract/legitimate use basis as advised by counsel. Consent records store who, what notice version, when, channel, and withdrawal. |
| NFR-PRIV-04 | Withdrawal of consent as easy as giving it: profile toggle, "STOP" handling on WhatsApp/SMS, unsubscribe link in emails. |
| NFR-PRIV-05 | Rights: access/summary, correction, erasure, grievance redressal, nomination; tenant DPDP queue with due dates (FR-TA-116); SSPL assists tenants as processor. |
| NFR-PRIV-06 | Data minimisation: collect only what booking, invoicing and legal duties need; KYC documents (virtual office) stored encrypted with restricted access and purged per retention rule. |
| NFR-PRIV-07 | Retention: customer personal data erased or anonymised when the purpose ends (e.g. 3 years after last activity, tenant-configurable within limits) except data required by law (GST invoices). Tenant data handled per the billing lifecycle (BRD §7.3). |
| NFR-PRIV-08 | Security safeguards: encryption in transit (TLS 1.2+) and at rest (RDS, S3 with KMS); access logging; role-based access; masking of personal data in support views. |
| NFR-PRIV-09 | Breach response: documented runbook; notification to affected tenants without delay so they can meet their obligations to the Data Protection Board and data principals; SSPL notifies the Board for its own fiduciary data. Target: tenant notified within 24 hours of confirmation. |
| NFR-PRIV-10 | Children: the booking app is not directed at children; age confirmation (18+) at sign-up to avoid verifiable parental consent requirements. |
| NFR-PRIV-11 | Data residency: all personal data stored and processed in AWS India regions; sub-processors (email, SMS, WhatsApp, error monitoring) listed publicly with their data locations; prefer India-region options. |
| NFR-PRIV-12 | Personal data never placed in URLs, query strings or analytics events. |

## 11. Other quality attributes

| ID | Area | Requirement |
|----|------|-------------|
| NFR-OT-01 | Browser support | Last 2 versions of Chrome, Edge, Safari, Firefox; Chrome on Android 10+; Safari on iOS 16+. |
| NFR-OT-02 | Localisation readiness | All UI strings externalised (i18n keys); dates in IST with `Asia/Kolkata`, stored UTC; currency formatting `en-IN` (₹1,23,456.00). |
| NFR-OT-03 | Maintainability | Modular monolith with enforced module boundaries (architecture tests, e.g. NetArchTest); ≥70% unit test coverage on domain modules (pricing, availability, policies, invoicing); contract tests for gateway adapters. |
| NFR-OT-04 | API | Versioned (`/api/v1`), OpenAPI documented, consistent error format (RFC 9457 problem details), idempotency keys on POSTs that create money-affecting records. |
| NFR-OT-05 | Time correctness | Availability, buffers and policies computed in the location's timezone; DST-safe (future-proof for non-IST tenants). |
| NFR-OT-06 | Environments | Dev, Staging (with Razorpay test mode and real DNS on `cowos-staging.site`), Production; production data never copied to lower environments without anonymisation. |
