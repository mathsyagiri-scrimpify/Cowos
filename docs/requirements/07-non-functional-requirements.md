# 07 · Non-Functional Requirements

> Cowos requirements pack · v0.3 draft · IDs `NFR-<area>-<nn>`; all are MVP (Must, P1) unless marked.

## 1. Multi-tenancy and data isolation

| ID | Requirement |
|----|-------------|
| NFR-MT-01 | Every tenant-owned row carries a non-null `tenant_id`. Tables without it are platform-global and contain no tenant personal data. |
| NFR-MT-02 | Isolation is enforced in two independent layers: (1) EF Core global query filters bound to the request's tenant context, and (2) PostgreSQL Row-Level Security with `ENABLE` and `FORCE ROW LEVEL SECURITY` on every tenant table. Every DB unit of work (API request, job step) runs in an explicit transaction opened by middleware or an interceptor, whose first statement is `set_config('app.tenant_id', id, true)` (transaction-local). Policies read `current_setting('app.tenant_id', true)`, so a missing setting returns zero rows (fail closed). Session-level `SET` is not allowed. The application DB role cannot bypass RLS (`NOBYPASSRLS`); migrations use a separate owner role (R3). |
| NFR-MT-03 | Tenant context is derived from the hostname (public) or a signed token claim (admin), never from a client-supplied header or body field. |
| NFR-MT-04 | Background jobs carry `tenant_id` explicitly in their payload and set the tenant as the first statement of each transaction (NFR-MT-02); jobs that iterate tenants process and commit one tenant per transaction (R3). Cross-tenant sweeps find work only through the `cowos_sweeper` role (NFR-MT-10). |
| NFR-MT-05 | Cache keys, S3 object keys (`tenants/{tenantId}/...`), search indexes, log entries and metrics include tenant ID. |
| NFR-MT-06 | An automated cross-tenant test suite runs in CI: for every API endpoint, a request authenticated as tenant A attempting to read/write tenant B's resource IDs must return 404. Adding an endpoint without coverage fails the build. |
| NFR-MT-07 | Per-tenant rate limits (API requests, OTP sends, holds, contact form submissions) protect against noisy neighbours and abuse. |
| NFR-MT-08 | The architecture supports moving a tenant to a dedicated cell (own database, optional own compute) by updating the tenant catalog and running a data migration job, with no application code change (Phase 3 capability, designed in P1). |
| NFR-MT-09 | Tenant-context guard tests run in CI: a pooled-connection leak test through RDS Proxy alternates tenants on reused connections and asserts no row of the previous tenant is visible; an architecture test fails the build on any session-level `SET` (R3). |
| NFR-MT-10 | Cross-tenant sweeps (hold expiry, reminders, no-show) find work through a `cowos_sweeper` DB role. Its read-only RLS policies expose only `(id, tenant_id)` of rows matching each sweep predicate, served by the sweep partial indexes (09 §4). The worker then handles each returned row in its own per-tenant transaction (NFR-MT-02). An isolation test proves the role can't read other columns or write; a sweep test finds rows across two tenants; a query-plan test asserts each sweep uses its index (R7, R8). |
| NFR-MT-11 | Platform data that is not tenant data, such as demo requests from the Book-a-demo form and the SSPL cost view data (Pricing Rule #10), lives in the platform schema outside tenant RLS and is readable only by the Super Admin role (NFR-SEC-01, CEO-A2). |

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
| NFR-SEC-17 | Support impersonation only through tenant-approved grants (02 §3.3), with dual-identity audit records. There is no exception (the pre-handover setup grant R19 was removed in v0.3 with the pilot programme; Pricing Rule #9). |
| NFR-SEC-18 | Removed in v0.3 (Owner invite links by SSPL removed with the pilot programme; Owners sign up self-serve, Pricing Rule #9). Staff invitation links keep their own rules (FR-TA-172). |
| NFR-SEC-19 | Public Book-a-demo form on `cowos.in`: honeypot, 5 submissions per IP per hour, invisible challenge (the FR-TW-006 protection); input stored as plain text and rendered escaped in Super Admin. Tests: honeypot and rate limit reject; a script in a message renders as text (CEO-A2). |
| NFR-SEC-20 | Resumable booking URL (`/book/checkout/{bookingRef}`) and its status endpoint require the customer's session; without one the page asks for OTP and then shows the state; another customer's session gets 404 (R16). |

### 2.2 PCI-DSS scope minimisation

| ID | Requirement |
|----|-------------|
| NFR-PCI-01 | Card data is entered only into Razorpay Checkout (Razorpay-hosted iframe/modal). Cowos never receives, stores, processes or logs PAN, CVV or expiry. |
| NFR-PCI-02 | Cowos targets SAQ-A eligibility: payment pages load the gateway script from the gateway domain only; CSP restricts script sources; integrity of payment page scripts monitored (PCI DSS v4.0 req. 6.4.3 and 11.6.1). |
| NFR-PCI-03 | Stored payment data limited to gateway IDs (order, payment, refund), method type, last 4 digits/UPI handle masked as returned by the gateway, amounts and statuses. |
| NFR-PCI-04 | Server invoices are paid by Razorpay payment link or, optionally, UPI Autopay / e-mandate held at Razorpay (₹5,000 per-tenant maximum, RBI pre-debit notice). Wallet top-ups are paid through Razorpay on SSPL's account. No card-on-file at Cowos (Pricing Rules #5, #6). |

## 3. Performance

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-PERF-01 | Tenant website Core Web Vitals (field data, p75, mobile) | LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1 |
| NFR-PERF-02 | Tenant website Lighthouse mobile performance score on a published demo site of every launched theme (2 at launch; themes 3-4 must pass before their release) (CEO-S3) | ≥ 90 |
| NFR-PERF-03 | Tenant website page weight (home, excluding hero media); self-hosted fonts count against it (25A) | ≤ 300 KB transferred; JS ≤ 50 KB gzip |
| NFR-PERF-04 | Booking app main JS bundle (excluding theme shell chunks), checked in CI (R13) | ≤ 200 KB gzip; route-level code splitting |
| NFR-PERF-05 | Booking app time to interactive on a mid-range Android on 4G (Moto G Power class, simulated) | ≤ 3.5 s |
| NFR-PERF-06 | Public API latency (availability, holds) | p95 ≤ 300 ms, p99 ≤ 800 ms at design load |
| NFR-PERF-07 | Admin API latency | p95 ≤ 500 ms |
| NFR-PERF-08 | Publish time (click → live) for a typical site | ≤ 2 minutes p95 |
| NFR-PERF-09 | Builder preview refresh after an edit | ≤ 1 s |
| NFR-PERF-10 | Notification dispatch after booking confirmation | ≤ 60 s p95 |
| NFR-PERF-11 | Design load (year 3) | 1,500 tenants, 50 req/s sustained public API, 300 req/s peak, 5,000 bookings/hour peak |
| NFR-PERF-12 | Booking app theme shell chunk, one per theme, loaded only for the tenant's theme (16A, R13); checked in CI; a test asserts a tenant on theme X never downloads theme Y | ≤ 40 KB gzip each (fonts excluded) |
| NFR-PERF-13 | Website pages correct after a tenant enters or leaves Suspended or Restricted (R11, CEO-A1, Pricing Rule #8) | ≤ ~2 min p95 |
| NFR-PERF-14 | Core Web Vitals (NFR-PERF-01) and Lighthouse budgets (NFR-PERF-02) met on every launched theme: a launch checklist item and a release gate for each later theme (CEO-S3) | Pass |

## 4. Availability and resilience

| ID | Requirement |
|----|-------------|
| NFR-AV-01 | Tenant websites: 99.95% monthly availability (static on S3 + CloudFront; independent of API health). |
| NFR-AV-02 | Booking app and public API: 99.9% monthly (≈43 min downtime/month). Open: this target is not confirmed for one app server (Open Question 6, Rule #11). |
| NFR-AV-03 | Admin portals: 99.5% monthly, maintenance windows announced 48h ahead, Sunday 02:00–05:00 IST. Open: this target is not confirmed for one app server (Open Question 6, Rule #11). |
| NFR-AV-04 | Database: managed PostgreSQL on RDS, Multi-AZ, with automated backups and point-in-time recovery (NFR-BK-01). Compute: one app server (EC2) runs the API, the admin portal and booking app back ends, the worker and the site renderer for every tenant (Rule #11). The app server is a single point of failure for admin and booking; tenant websites keep serving from S3 + CloudFront when it is down (NFR-AV-01). |
| NFR-AV-05 | Deploys to the app server may cause a short outage of the API, admin and booking. The acceptable downtime per deploy is open (Open Question 6). Each deploy health-checks the new release and rolls back to the previous release if the check fails. Database migrations stay backward-compatible (expand/contract) so a rollback needs no schema change. |
| NFR-AV-06 | Graceful degradation: if messaging providers fail, bookings still confirm and notifications retry; if the gateway is down, booking app shows a clear message and offers Request-to-Book where enabled. |
| NFR-AV-07 | Public status page (`status.cowos.in`) for tenants. |
| NFR-AV-08 | Edge consistency (R17): the CloudFront KeyValueStore always ends equal to the database's live pointer and tenant state. Only the EdgeSyncWorker writes KVS; it reads the current DB values at send time from the `edge_sync` outbox. No other job writes KVS directly. Tests: KVS writes forced into the reverse of the DB commit order end with KVS equal to DB; a worker restart resends the current values. If the worker is down the edge keeps the last value and an alert fires (NFR-OBS-05). |
| NFR-AV-09 | Live-pointer correctness: after every render job finishes, the live pointer shows the newest owner-chosen version (publish or rollback) in the page variant that matches the tenant's current state. A normal-site render never goes live while the tenant is Suspended (CEO-A1); a data refresh never replaces a newer publish or newer data (CEO-A3, R18). |
| NFR-AV-10 | Automatic instance recovery: when the app server fails its EC2 status checks or its health check, it is recovered or replaced automatically with no manual step, and the services start on boot (Rule #11). A recovery drill on staging has a documented result before launch. |
| NFR-AV-11 | Infrastructure-as-code rebuild: the app server, its network, load balancer and configuration are defined in Terraform, so a new server can be built from code and the current release in another AZ if the instance or its AZ is lost. Nothing needed to run the app is kept only on the server's disk (NFR-SC-01). A rebuild drill runs with the quarterly restore drill (NFR-BK-05) (Rule #11). |
| NFR-AV-12 | Health alarms on the app server: instance status checks, API health endpoint (`/healthz`, `/readyz`), CPU, memory, disk, and worker and renderer process health, routed to on-call (NFR-OBS-05) (Rule #11). |

## 5. Scalability

| ID | Requirement |
|----|-------------|
| NFR-SC-01 | API, worker and renderer processes are stateless: no session, file or queue state lives only on the app server (sessions, jobs and files are in PostgreSQL, Valkey and S3). At launch they run on one app server (Rule #11); the server can be resized or replaced, and more servers added later, with no application code change. Renders run with a concurrency limit so they don't starve the API on the shared server. |
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
| NFR-OBS-05 | Alerts (PagerDuty/Opsgenie or SNS → on-call): API 5xx > 1% for 5 min, p95 latency breach, queue age > 5 min, `edge_sync` outbox age > 2 min (R17), suspend render failed 3 times (CEO-A1), demo request notification failed (CEO-A2), payment webhook failures, certificate expiry < 14 days, DB CPU > 80%, disk, replication lag, failed backups, app server health alarms (NFR-AV-12), a daily metering job failed or not finished for its day, the rating job on the 1st failed (NFR-MTR-01 to 06, Pricing Rules #3, #5). |
| NFR-OBS-06 | Synthetic checks every 5 minutes: demo tenant website, booking availability, hold creation (test tenant), admin login. |
| NFR-OBS-07 | Audit logs retained 3 years (append-only, separate table/stream, tamper-evident hashing optional). Application logs retained 30 days hot, 1 year archive. |
| NFR-OBS-08 | SSPL cost view: the production AWS bill is read from AWS Data Exports (CUR 2.0) and messaging-provider cost from the recorded per-message cost, and shown in Super Admin next to billed revenue (server invoices and wallet debits) per month, so SSPL can set slab prices and track goal G3. CUR is never used to bill tenants (Pricing Rule #10, CEO-A5). AWS cost allocation tags are kept for cost-to-serve. |

## 8. Accessibility (WCAG 2.1 AA)

| ID | Requirement |
|----|-------------|
| NFR-A11Y-01 | All themes, the booking app and admin portals meet WCAG 2.1 AA. |
| NFR-A11Y-02 | Colour choices validated for 4.5:1 text contrast and 3:1 UI component contrast at input time (FR-TA-022). |
| NFR-A11Y-03 | Keyboard operable everywhere: date pickers, start-time lists as labelled radio groups (arrow keys move, Enter or Space selects, focus visible) (1A, 27A), modals (focus trap and return), Razorpay Checkout launch. |
| NFR-A11Y-04 | Semantic HTML, one `h1` per page, landmark regions, labelled form controls, error messages linked to fields (`aria-describedby`). |
| NFR-A11Y-05 | Alt text required for content images (FR-TA-024); decorative images marked as such. |
| NFR-A11Y-06 | Availability conveyed by text ("2 left", "Sold out"), not colour alone (1A). |
| NFR-A11Y-07 | Respects `prefers-reduced-motion`; no auto-playing video with sound; hero video has pause control. |
| NFR-A11Y-08 | Automated axe-core checks in CI for every launched theme (including its booking shell) and the booking flow; manual screen reader pass (NVDA + TalkBack) on the booking flow each release and before each theme release. Launch ships 2 themes; themes 3-4 pass Lighthouse, axe and the screen reader pass before release (CEO-S3, 27A). |
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
| NFR-PRIV-01 | Data Processing Agreement between SSPL and each tenant, accepted by the Owner (versioned, timestamped) at sign-up (FR-MW-005, Pricing Rule #9). |
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
| NFR-PRIV-13 | Desk pass link (no login, 17A): the token is opaque with no personal data, HMAC-bound to booking id and purpose `pass`, expires at booking end, and is revoked on cancel or reschedule. The pass shows first name + last initial, date/time, room and tenant logo. The QR holds a separate short-lived check-in code validated against the tenant at scan. Responses carry `noindex`, `Referrer-Policy: no-referrer` and `Cache-Control: private`. Tests reject expired, revoked, wrong-tenant and wrong-purpose tokens and confirm a forwarded link shows no full name (R10). |
| NFR-PRIV-14 | Offline desk pass in Phase 1: a service worker scoped to the pass route only caches the opened pass page and QR image, cleared when the booking ends. The full PWA stays in Phase 2 (R14). |
| NFR-PRIV-15 | Book-a-demo form on `cowos.in`: a privacy notice under the form shows purpose (demo contact), 12-month retention and the grievance contact; requests are retained for 12 months under NFR-PRIV-07 (CEO-A2). |

## 11. Other quality attributes

| ID | Area | Requirement |
|----|------|-------------|
| NFR-OT-01 | Browser support | Last 2 versions of Chrome, Edge, Safari, Firefox; Chrome on Android 10+; Safari on iOS 16+. |
| NFR-OT-02 | Localisation readiness | All UI strings externalised (i18n keys); dates in IST with `Asia/Kolkata`, stored UTC; currency formatting `en-IN` (₹1,23,456.00). |
| NFR-OT-03 | Maintainability | Modular monolith with enforced module boundaries (architecture tests with NetArchTest); ≥70% unit test coverage on domain modules (pricing, availability, policies, invoicing); contract tests for gateway adapters. |
| NFR-OT-04 | API | Versioned (`/api/v1`), OpenAPI documented, consistent error format (RFC 9457 problem details), idempotency keys on POSTs that create money-affecting records. |
| NFR-OT-05 | Time correctness | Availability, buffers and policies computed in the location's timezone; DST-safe (future-proof for non-IST tenants). |
| NFR-OT-06 | Environments | Dev, Staging (with Razorpay test mode and real DNS on `cowos-staging.site`), Production; production data never copied to lower environments without anonymisation. |
| NFR-OT-07 | Test stack | .NET: xUnit + FluentAssertions. Integration tests run on Testcontainers PostgreSQL 17 through WebApplicationFactory, so RLS, `btree_gist` exclusion constraints and transactions are real (no in-memory database). React apps: Vitest + React Testing Library. End to end: Playwright + axe on staging with Razorpay test mode. Module boundaries: NetArchTest. Docker is available in CI and locally (R6). |
| NFR-OT-08 | Concurrency proof | Money and ordering paths have concurrency tests on the real database: parallel wallet sends at a near-floor balance never overspend (R4); render pause/release tests for the live-pointer rules (CEO-A1, CEO-A3, R18); reversed-order KVS writes (R17). Pause/release tests use explicit hooks, not sleeps; month-boundary tests use a fixed clock. |

## 12. Metering and billing accuracy

| ID | Requirement |
|----|-------------|
| NFR-MTR-01 | Usage records are kept: each tenant's daily usage per meter (request units, storage units, and messages per notification channel) is stored and retained at least as long as the invoice or wallet debits it supports, so a tenant can see the daily detail behind any invoice (Pricing Rules #3, #5). |
| NFR-MTR-02 | Invoices are reproducible: re-rating any past month from the stored daily usage and the rate card version recorded on the invoice gives the same units, slab breakdown, taxable value and GST. A published rate card version is never changed; a price change is a new version effective from the 1st of a later month. A test re-rates fixture months and compares them with the issued invoices (Pricing Rule #4). |
| NFR-MTR-03 | Exclusions are applied before counting: requests blocked by WAF and SSPL's own health checks, synthetic checks (NFR-OBS-06) and monitoring are not counted as request units. Tests feed log fixtures with blocked and monitoring requests and assert they are excluded (Pricing Rule #3). |
| NFR-MTR-04 | Each request is counted once and from one durable source: CloudFront standard logs for the tenant's website and booking-app hosts (page views and `/api/*` calls, R22). Each request maps to exactly one tenant by its host; requests that map to no tenant, admin-host traffic and responses other than 2xx or 304 are not billed (Pricing Rule #3, R21). |
| NFR-MTR-05 | Metering jobs are idempotent: re-running the metering for a day replaces that day's records and never adds to them. The rating job on the 1st runs only after every daily metering job for the last day of the month has finished, and issues at most one server invoice per tenant per month (Pricing Rule #5). |
| NFR-MTR-06 | Notification counts and wallet debits are exact: each message sent increments its channel's month-to-date count and debits the wallet at that count's slab price in one atomic step (R4), so the count always equals the messages sent and parallel sends at a slab boundary each get the right slab price. Counts reset at 00:00 IST on the 1st. Tests use a fixed clock at the month boundary and parallel sends across a slab boundary (Pricing Rules #4, #6). |
| NFR-MTR-07 | Trial days are not billed: server usage while the tenant is in Trial is recorded but not rated (Pricing Rule #7). |
| NFR-MTR-08 | Invoices only from complete data: each metering source records a completed run per IST day, and no server invoice is issued for a month until every day is complete for every source; gaps hold invoices and alert SSPL Finance (C2-A2). |
