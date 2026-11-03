# 09 · Data Model

> Cowos requirements pack · v0.2 draft · Logical model for PostgreSQL. All tenant-owned tables carry `tenant_id` (RLS-enforced). IDs are UUIDv7. Timestamps `timestamptz` in UTC. Money as `numeric(12,2)` in INR with `currency char(3)` for future markets.

## 1. Entity overview by module

| Module | Entities |
|--------|----------|
| Tenancy | Tenant, TenantSettings, TenantStateHistory, Cell (tenant catalog) |
| Identity | TenantUser, TenantUserRole, TenantUserLocation, TenantUserInvitation, Customer, CustomerIdentity (phone/email), OtpChallenge, PlatformUser |
| Sites | Theme, ThemeVersion, Site, SiteDraft, SiteVersion, Page, MediaAsset, MediaVariant, EdgeSync (`edge_sync` outbox) |
| Domains | Domain, DomainVerification, CertificateStatus |
| Inventory | Location, OperatingHours, Holiday, Amenity, LocationAmenity, Product, ProductUnit (Resource), PriceRule, AddOn, AvailabilityRule, MaintenanceBlock, TourSlotRule |
| Booking | Booking, BookingAllocation, BookingAddOn, BookingEvent, DayCapacityLedger, CheckIn, BookingPass, CheckInCode |
| Policies | CancellationPolicy, CancellationTier, PolicyDocument |
| Payments | PaymentGatewayAccount, PaymentOrder, Payment, Refund, OfflinePayment, WebhookEvent |
| Invoicing | InvoiceSeries, Invoice, InvoiceLine, CreditNote, CreditNoteLine, TaxProfile |
| CRM | Company, CompanyMember, Lead, LeadActivity, Tour, Note, ConsentRecord, DataRightsRequest, SecurityDeposit |
| Memberships (P2) | MembershipPlan, Membership, CreditLedgerEntry |
| Promotions | Coupon, CouponRedemption |
| Notifications | NotificationTemplate, NotificationMessage, MessagingWallet, WalletTransaction, PilotMessagingUsage |
| Platform billing (P1-B) | BillingProfile, CostPool, TenantCostShare, HostingInvoice, AdjustmentLine, PlatformPayment, UsageRecord. PlatformPlan, PlatformPlanVersion, PlatformSubscription and PlatformInvoice: Removed in v0.2 (D1, R2) |
| Platform admin | PilotRequest (platform schema, outside tenant RLS) |
| Audit & support | AuditEvent, SupportGrant, PreHandoverGrant, SupportTicketRef |

## 2. Core relationships

```mermaid
erDiagram
    TENANT ||--o{ DOMAIN : has
    TENANT ||--|| SITE : owns
    TENANT ||--o{ LOCATION : operates
    TENANT ||--o{ TENANT_USER : employs
    TENANT ||--o{ CUSTOMER : serves
    TENANT ||--o{ COMPANY : bills
    TENANT ||--o{ PAYMENT_GATEWAY_ACCOUNT : connects
    TENANT ||--o| BILLING_PROFILE : "pays via (P1-B)"
    TENANT ||--o{ HOSTING_INVOICE : "billed (P1-B)"
    TENANT ||--o{ EDGE_SYNC : "queues edge updates"
    TENANT }o--|| CELL : "placed in"

    SITE }o--|| THEME_VERSION : uses
    SITE ||--|| SITE_DRAFT : edits
    SITE ||--o{ SITE_VERSION : publishes
    SITE ||--o{ MEDIA_ASSET : stores

    LOCATION ||--o{ PRODUCT : offers
    LOCATION ||--o{ HOLIDAY : closes
    LOCATION ||--o{ OPERATING_HOURS : opens
    PRODUCT ||--o{ PRODUCT_UNIT : contains
    PRODUCT ||--o{ PRICE_RULE : priced_by
    PRODUCT ||--o{ ADD_ON : extends
    PRODUCT }o--|| CANCELLATION_POLICY : governed_by
    PRODUCT ||--o{ AVAILABILITY_RULE : constrained_by

    CUSTOMER ||--o{ BOOKING : makes
    COMPANY ||--o{ COMPANY_MEMBER : includes
    CUSTOMER ||--o{ COMPANY_MEMBER : "is"
    BOOKING }o--|| PRODUCT : for
    BOOKING ||--o{ BOOKING_ALLOCATION : reserves
    BOOKING_ALLOCATION }o--o| PRODUCT_UNIT : on
    BOOKING ||--o{ BOOKING_ADD_ON : includes
    BOOKING ||--o{ PAYMENT_ORDER : paid_via
    PAYMENT_ORDER ||--o{ PAYMENT : results_in
    PAYMENT ||--o{ REFUND : refunded_by
    BOOKING ||--o{ INVOICE : invoiced_by
    INVOICE ||--o{ INVOICE_LINE : has
    INVOICE ||--o{ CREDIT_NOTE : adjusted_by
    INVOICE }o--|| INVOICE_SERIES : numbered_in
    BOOKING }o--o| COUPON : applies
    BOOKING }o--o| MEMBERSHIP : "uses credits of"

    CUSTOMER ||--o{ MEMBERSHIP : holds
    MEMBERSHIP }o--|| MEMBERSHIP_PLAN : on
    MEMBERSHIP ||--o{ CREDIT_LEDGER_ENTRY : tracks

    TENANT ||--o{ LEAD : receives
    LEAD ||--o{ TOUR : schedules
    LEAD }o--o| CUSTOMER : converts_to

    COST_POOL ||--o{ TENANT_COST_SHARE : "split into"
    TENANT_COST_SHARE |o--o| HOSTING_INVOICE : "invoiced on"
    HOSTING_INVOICE ||--o{ ADJUSTMENT_LINE : carries
    BOOKING ||--o| BOOKING_PASS : "shown as"
    TENANT ||--|| MESSAGING_WALLET : funds
```

## 3. Key entities and attributes

### 3.1 Tenancy and identity

**Tenant**
| Field | Type | Notes |
|-------|------|-------|
| id | uuid | |
| slug | citext unique | Platform subdomain; reserved list enforced |
| brand_name, legal_name | text | |
| entity_type | enum | Proprietorship, Partnership, LLP, PvtLtd, Other |
| gstin | text null | Validated format + checksum; unique among active tenants |
| pan | text null | |
| state_code | char(2) | From GSTIN or address; drives tax |
| state | enum | Trial, Active, PastDue, Restricted, Suspended, SuspendedByAdmin, Expired, Cancelled, PendingDeletion, Deleted. At launch the state machine rejects any move into Trial, PastDue or Restricted with 409; the Billing release enables them (Q1, P1-B) |
| trial_ends_at, state_changed_at | timestamptz | trial_ends_at used from P1-B (14-day trial for non-pilots) |
| is_pilot | bool | Pilot flag. Pilots are created by Super Admin directly in Active; trial expiry and trial publish restrictions never apply (Pilot Rules #4, F-C2) |
| free_period_ends_at | timestamptz null | Set at the tenant's first live publish on production: 12 months later (Pilot Rules #3, F-C2) |
| unconverted_restricted_until | timestamptz null | End of the 90-day Restricted hold for an unconverted pilot; after it the normal Suspended lifecycle applies (Pilot Rules #6, F-C2) |
| data_seq | bigint | Increments in the same transaction as every website-visible data change (inventory names, prices, hours) and every tenant state change; compared by data-refresh renders (R18) |
| first_online_booking_at | timestamptz null | Set atomically with the first paid online booking confirmation, so the first-booking message sends once (14A, F-RR-C2) |
| goal | enum | FullSite, BookingOnly (booking-only path from P1-B, CEO-S2) |
| cell_id | uuid | Tenant catalog (pool-1 by default) |
| kyc_status | enum | NotStarted, Pending, Verified, Rejected |
| first_publish_approved_at | timestamptz null | |
| referral_code, referred_by_code, utm | text / jsonb | Captured from P1-B with public sign-up (FR-MW-006, CEO-S1) |
| timezone | text | Default `Asia/Kolkata` |
| currency | char(3) | Default INR |

Pilot state transitions (F-C2): created by Super Admin → Active with `is_pilot`; first live publish sets `free_period_ends_at`; free-period end → stays Active and pays its hosting share from month 13 (P1-B); first post-free-year invoice unpaid by due date + 14 days → Restricted with `unconverted_restricted_until` = 90 days later. Each transition has a test.

**TenantUser**: id, tenant_id, email (citext), phone, name, password_hash, mfa_secret (encrypted), mfa_enabled, role (Owner, Manager, Staff), status, last_login_at. **TenantUserLocation**: tenant_user_id, location_id (scope).

**TenantUserInvitation** (FR-TA-172; also the pilot Owner invite, CEO-A2): id, tenant_id, email, phone, role (Owner, Manager, Staff), location_ids, token_hash, invited_by (tenant user or platform user), expires_at (72 hours after send for the Owner invite), accepted_at null, revoked_at null (a resend revokes the previous invitation). Single use. At most one open Owner invitation per tenant.

**Customer** (per tenant): id, tenant_id, name, phone_e164, email, gstin, billing_address (jsonb), marketing_consent (derived from latest ConsentRecord), status (Active, Blocked, Anonymised), created_via (Online, Staff, Import), last_activity_at. Unique (tenant_id, phone_e164) and (tenant_id, email) where not null.

**Company**: id, tenant_id, name, gstin, billing_address, admin_customer_id, credit_terms_days, notes. **CompanyMember**: company_id, customer_id, role (Admin, Member), can_bill_to_company.

**PlatformUser** (SSPL staff): id, idp_subject, email, role (SuperAdmin, Support, Finance), status.

### 3.2 Sites and domains

**Theme / ThemeVersion**: theme key, name, version (semver), package reference, content JSON schema, supported sections, booking shell chunk reference (16A, R13), status (Hidden, Active, Deprecated). Every theme is available to every tenant; there are no plans (D1). Hidden keeps a new theme out of the catalog until its Lighthouse, axe and screen reader checks pass (CEO-S3).

**Site**: id, tenant_id, theme_version_id, live pointer (`live_version_id` null, `live_data_seq` null), booking_placement (Subdomain, Path), branding (jsonb: logo asset ids, colours, font pair, button shape, mode). The pointer changes only at the render switch step: owner publish and rollback set it (blocked while Suspended); a data refresh sets it only if `live_version_id` is unchanged and its `data_seq` is higher than `live_data_seq` (CEO-A1, CEO-A3, R18). Every pointer change adds an EdgeSync row in the same transaction (R17).

**EdgeSync** (`edge_sync` outbox, R17): id, tenant_id, reason (Publish, Rollback, DataRefresh, StateChange, Domain, PilotCreated), enqueued_at, sent_at null. Added in the same transaction as every change to a tenant's live pointer, state or hostnames. The single EdgeSyncWorker reads unsent rows, then reads the tenant's current pointer and state from the DB and writes them to KVS; it marks the rows sent. Partial index on `enqueued_at` WHERE `sent_at IS NULL`; alert when the oldest unsent row is older than 2 minutes.

**SiteDraft**: site_id, content (jsonb, theme-independent sections), updated_at, updated_by, revision (optimistic concurrency).

**SiteVersion** (immutable): id, site_id, number (int, increments), content snapshot (jsonb), branding snapshot, theme_version_id, status (Rendering, Live, Superseded, Failed), s3_prefix, published_by, published_at, note, rolled_back_from_version_id null.

**MediaAsset**: id, tenant_id, original_key, mime, width, height, bytes, alt_text, blurhash, scan_status, variants (jsonb), usage_count.

**Domain**: id, tenant_id, hostname (citext unique globally), purpose (Website, Booking, WebsiteAndBooking, Redirect), is_platform (bool), status (PendingDns, Verified, CertPending, Active, Degraded, Removed), verification_token, last_checked_at, dns_provider_detected, cert_expires_at, edge_tenant_ref.

### 3.3 Inventory

**Location**: id, tenant_id, name, slug, address (jsonb), lat, lng, directions, gst_registration_id (for multi-state), timezone, contact phone/email, is_24x7, status.

**OperatingHours**: location_id, day_of_week, opens_at, closes_at (multiple rows for split shifts). **Holiday**: location_id, date, name, closed_all_day or partial hours.

**Product** (what is sold)
| Field | Notes |
|-------|-------|
| id, tenant_id, location_id | |
| type | HotDesk, DedicatedDesk, PrivateCabin, MeetingRoom, EventSpace, VirtualOffice, Custom |
| inventory_model | Pool (capacity count) or Unit (specific resources) |
| name, slug, description, photos (media ids), amenities | |
| booking_mode | Instant, Request, Enquire |
| visibility | Public, MembersOnly, Hidden |
| capacity_per_unit | Seats (rooms/cabins) |
| pool_capacity | For Pool model (seats per day) |
| slot_granularity_min, min_duration_min, max_duration_min | |
| buffer_before_min, buffer_after_min, lead_time_min, advance_window_days | |
| tax: sac_code (default 997212), gst_rate (default 18.00), price_includes_tax | |
| cancellation_policy_id, request_sla_hours | |
| security_deposit_amount | Monthly products |
| convenience_fee_type, convenience_fee_value | Optional tenant fee (None, Flat, Percent); goes to the tenant |
| is_sample | Demo inventory labelled "Sample"; pre-publish checks block publishing while any sample item remains (4A, F-RR-C2; P1-B with demo inventory, CEO-S2) |

**ProductUnit** (Resource): id, tenant_id, product_id, label ("Cabin 3", "D-12"), capacity, floor, zone, floor_plan_x/y (P2), status (Active, Maintenance, Retired).

**PriceRule**: id, product_id, unit (Hour, HalfDay, Day, Week, Month), amount, member_amount null, applies_to_unit_id null (unit-specific price), valid_from, valid_to, peak rules (P2: days_of_week, time window, multiplier).

**AddOn**: id, product_id, name, amount, basis (PerBooking, PerHour, PerPerson), max_qty, stock null.

**AvailabilityRule**: product_id or unit_id, days_of_week, start_time, end_time (overrides location hours). **MaintenanceBlock**: unit_id, during (tstzrange), reason.

**TourSlotRule**: location_id, days_of_week, start, end, slot_minutes, parallel_tours.

### 3.4 Booking

**Booking**
| Field | Notes |
|-------|-------|
| id, tenant_id, reference (human-readable, e.g. `WL-7K3D9`) | |
| customer_id, company_id null, bill_to (Customer, Company) | |
| product_id, location_id | |
| status | Requested, Held, AwaitingPayment, Confirmed, CheckedIn, Completed, Cancelled, NoShow, Expired, Declined |
| source | Online, Staff, Widget, Import |
| starts_at, ends_at | Customer-facing period |
| quantity | Seats (pool) |
| price_snapshot | jsonb: unit prices, add-ons, discounts, tax split, totals (frozen at booking) |
| subtotal, discount, taxable_value, tax_amount, total | |
| coupon_id, membership_id (credits) | |
| hold_expires_at | For Held |
| payment_link_expires_at | For AwaitingPayment |
| cancellation_policy_snapshot | jsonb (policy at time of booking applies, not current policy) |
| cancelled_at, cancelled_by, cancel_reason, refund_amount_policy, refund_amount_final | |
| notes_customer, notes_internal | |
| idempotency_key | Unique per tenant |

**BookingAllocation** (what is actually blocked)
| Field | Notes |
|-------|-------|
| id, tenant_id, booking_id | |
| unit_id | For Unit-model products |
| blocked_during | tstzrange including buffers |
| active | Boolean, true while booking status blocks inventory |

Double-booking prevention for unit products:

```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;
ALTER TABLE booking_allocation
  ADD CONSTRAINT booking_allocation_no_overlap
  EXCLUDE USING gist (tenant_id WITH =, unit_id WITH =, blocked_during WITH &&)
  WHERE (active);
```

**DayCapacityLedger** (pool products): tenant_id, product_id, date, capacity, booked (int), with `CHECK (booked <= capacity)`. Holds increment `booked` in the same transaction (`UPDATE ... SET booked = booked + :qty WHERE booked + :qty <= capacity`), so overselling is impossible under concurrency.

**BookingEvent**: booking_id, type (Created, Held, PaymentStarted, Paid, Confirmed, Rescheduled, Cancelled, RefundIssued, CheckedIn, NoShow, NotificationSent), actor (customer/staff/system/support), data (jsonb), at.

**CheckIn**: booking_id, checked_in_at, method (QR, Manual, Self), by_user_id.

**BookingPass** (desk pass link, 17A, R10): id, tenant_id, booking_id, token_hash, purpose (`pass`), expires_at (booking end), revoked_at null (set on cancel or reschedule). The link token is opaque, carries no personal data and is HMAC-bound to booking id + purpose. The pass page shows only first name + last initial, date/time, room and tenant logo.

**CheckInCode** (QR on the pass, R10): id, tenant_id, booking_id, code_hash, expires_at (short-lived). Separate from the pass token. The front desk scan validates it against the scanning tenant only.

### 3.5 Policies

**CancellationPolicy**: id, tenant_id, name, product_type default, no_show_refund_pct, deduct_gateway_fee (bool), allow_credit_instead (bool), credit_bonus_pct, reschedule_allowed_until_hours, free_text. **CancellationTier**: policy_id, min_hours_before, refund_pct (ordered).

**PolicyDocument**: tenant_id, kind (Privacy, Terms, HouseRules, Refund), version, content, published_at.

### 3.6 Payments

**PaymentGatewayAccount**: id, tenant_id, provider (Razorpay, Cashfree, PhonePe), connection_type (OAuth, ApiKey), account_ref, display_name, mode (Test, Live), credentials_ciphertext (KMS envelope), token_expires_at, status (Connected, Error, Disconnected), last_health_check_at.

**PaymentOrder**: id, tenant_id, booking_id or invoice_id, gateway_account_id, provider_order_id, amount, status (Created, Attempted, Paid, Expired), idempotency_key.

**Payment**: id, tenant_id, order_id, provider_payment_id (unique), method (UPI, Card, Netbanking, Wallet, EMI), method_detail_masked, amount, fee, tax_on_fee, status (Authorized, Captured, Failed, Refunded, PartiallyRefunded), captured_at, raw (jsonb, no card data).

**OfflinePayment**: id, tenant_id, booking_id/invoice_id, method (Cash, UpiDirect, BankTransfer, Cheque), reference, amount, received_at, recorded_by.

**Refund**: id, tenant_id, payment_id null, booking_id, amount, destination (Original, Credit), provider_refund_id, status (Pending, Processed, Failed), reason, initiated_by, override (bool).

**WebhookEvent**: id, provider, provider_event_id (unique), tenant_id (resolved from account), type, signature_valid, payload (jsonb), received_at, processed_at, error.

### 3.7 Invoicing

**TaxProfile** (per tenant/location GST registration): gstin, legal name, address, state_code.

**InvoiceSeries**: id, tenant_id, location_id, kind (Invoice, CreditNote, BillOfSupply), prefix, financial_year ('2026-27'), next_number. Number allocation via `UPDATE ... RETURNING` in the invoice transaction (gap-free; max 16 characters total).

**Invoice**: id, tenant_id, series_id, number, issue_date, supplier snapshot (jsonb), buyer snapshot (name, address, GSTIN, state), place_of_supply_state, invoice_type (B2B, B2C, BillOfSupply), booking_id/membership_id, subtotal, discount, taxable_value, cgst, sgst, igst, total, amount_paid, status (Issued, PartiallyPaid, Paid, Void), pdf_key. Immutable after issue (only status and paid amounts change).

**InvoiceLine**: invoice_id, description, sac_code, quantity, unit, rate, discount, taxable_value, gst_rate, cgst, sgst, igst.

**CreditNote / CreditNoteLine**: same shape, references invoice_id, reason.

### 3.8 CRM and compliance

**Lead**: id, tenant_id, source (WebsiteForm, Enquiry, Tour, Walkin, Phone, Import, Widget), name, phone, email, company_name, interest_product_id, team_size, budget, move_in_date, stage (New, Contacted, TourScheduled, TourDone, Proposal, Won, Lost), lost_reason, owner_user_id, next_follow_up_at, customer_id (when converted). **LeadActivity**: lead_id, type (Call, WhatsApp, Email, Note, StageChange), data, by, at.

**Tour**: id, tenant_id, lead_id, location_id, scheduled_at, status (Scheduled, Done, NoShow, Cancelled), host_user_id.

**ConsentRecord**: id, tenant_id, customer_id or lead_id, purpose (Marketing_WhatsApp, Marketing_SMS, Marketing_Email, Analytics), granted (bool), notice_version, channel (Web, Staff, WhatsApp reply), captured_at, ip_hash.

**DataRightsRequest**: id, tenant_id, customer_id, type (Access, Correction, Erasure, Grievance, Nomination), status, due_at, handled_by, completed_at, notes.

**SecurityDeposit**: id, tenant_id, customer/company, booking/membership, amount, collected_via, status (Held, PartiallyRefunded, Refunded, Forfeited), deductions (jsonb).

### 3.9 Memberships (Phase 2) and promotions

**MembershipPlan**: id, tenant_id, location_id, name, price, cycle (Monthly, Quarterly, Annual), included_product_id, included_credit_minutes, credit_products (list), member_discount_pct, access_hours, deposit_amount, notice_days, rollover_cap_minutes, visibility.

**Membership**: id, tenant_id, plan_id, customer_id or company_id, assigned_unit_id, starts_on, current_cycle_start/end, status (Active, Paused, NoticeGiven, Ended), next_invoice_on.

**CreditLedgerEntry**: id, tenant_id, membership_id, change_minutes (+/−), reason (CycleGrant, BookingUse, Expiry, Adjustment, Refund), booking_id null, at. Balance = sum.

**Coupon** (tenant promo code): id, tenant_id, code (citext, unique per tenant), type (Percent, Flat), value, max_discount, min_order, valid_from/to, total_limit, per_customer_limit, product_ids, location_ids, first_booking_only, visibility (Public, Private), banner_text, status (Active, Paused, Expired, Archived), created_by. **CouponRedemption**: coupon_id, booking_id, customer_id, discount_amount.

### 3.10 Notifications

**NotificationTemplate**: id, tenant_id null (platform default), event, channel (Email, Sms, WhatsApp), locale, subject, body, provider_template_id (DLT/WhatsApp), category (Utility, Authentication, Marketing).

**NotificationMessage**: id, tenant_id, recipient (customer/user id + masked address), event, channel, template_id, status (Queued, Sent, Delivered, Read, Failed), provider_message_id, cost, error, related_entity, created_at.

**MessagingWallet**: tenant_id, balance, funded_by (Tenant, SsplPilot). Messaging is charged at provider cost with no free allowance; pilots use an SSPL-funded wallet (Pilot Rules #4, #10). Floors (R4): a notification debit needs `balance - cost >= 0`; a sign-in OTP debit needs `balance - cost >= -500` (₹). Each send runs one conditional `UPDATE messaging_wallet SET balance = balance - :cost WHERE tenant_id = :t AND balance - :cost >= :floor RETURNING balance`; if no row returns, the notification goes by email only, or the OTP falls back to email OTP. `CHECK (balance >= -500)`.

**WalletTransaction**: tenant_id, amount (+top-up / −usage), kind (TopUp, Notification, Otp, Adjustment), message_id null, top_up_payment_id null, at. Written in the same transaction as the wallet debit (R4). Non-pilot top-up from P1-B (CEO-S1).

**PilotMessagingUsage** (Pilot Rules #5): tenant_id, month (IST), notifications_sent, otps_sent, notification_cap_reached_at null, otp_alert_sent_at null, phone_otp_paused_at null. Caps: 1,000 SMS + WhatsApp notifications per month, then email only and an SSPL alert; OTP alert at 1,000, phone OTP paused at 2,000 for the rest of the month. Every message sent counts, including a fallback SMS.

### 3.11 Platform billing (P1-B)

There are no plans or subscriptions. Each tenant pays its share of the production AWS bill plus messaging at cost (D1, Pilot Rules #10). Launch carries none of these tables; the Billing release adds them as additive migrations and backfills a BillingProfile for every existing tenant (pilots with their free-period end date, SSPL test and demo tenants) (R2).

**PlatformPlan / PlatformPlanVersion**: Removed in v0.2 (no plans, prices, limits or feature tiers; every tenant gets every feature; D1, R2).

**PlatformSubscription**: Removed in v0.2 (no subscriptions; replaced by BillingProfile and HostingInvoice; D1, R2).

**PlatformInvoice**: Removed in v0.2 (replaced by HostingInvoice; R2).

**BillingProfile**: id, tenant_id (unique), billing contact email, GSTIN snapshot for the invoice, payment_method (PaymentLink, UpiAutopay, EMandate), mandate_ref null, mandate_max_amount null (at most ₹5,000; RBI pre-debit notice applies), paid_by_sspl (bool: SSPL test and demo tenants), status. A pilot adds a payment method with the 60-day notice (Pilot Rules #6); a non-pilot becomes Active when it adds one at the end of its trial (Pilot Rules #7).

**CostPool**: id, month, aws_bill_invoice_id null (CUR 2.0 `bill_invoice_id`; null until AWS marks the bill final), gross_cost, credits, net_cost, service_breakdown (jsonb: cost by service category), counted_tenant_days, status (Pending, Held, Final), estimate_low, estimate_high, estimate_as_of (R15: month-to-date pool plus a straight-line projection to month end, divided by today's counted tenants; high = projection + 15% buffer), computed_at. Unique (month). Net of AWS credits; production account only (R1, R2).

**TenantCostShare**: id, cost_pool_id, tenant_id, tenant_days (days in Active, PastDue or Restricted in the month), share_amount (= net_cost × tenant_days / counted_tenant_days), payer (Tenant, SsplTestDemo, SsplPilot), hosting_invoice_id null. Pilot shares in the free period and SSPL test/demo shares are recorded and absorbed by SSPL; no ₹0 invoice is issued. Unpaid shares are absorbed by SSPL, never passed to other tenants. Unique (cost_pool_id, tenant_id).

**HostingInvoice**: id, tenant_id, number (`CW/26-27/000123`), month, share_amount, adjustment_total, taxable_value, cgst, sgst, igst (SSPL state vs tenant state), total, status (Held, Issued, Paid, Overdue, Void), issued_at (10th of the next month), due_at (issued_at + 10 days), paid_at, payment_link_ref, pdf_key. Immutable after issue like tenant invoices.

**AdjustmentLine**: id, hosting_invoice_id, for_month, cost_pool_id, amount (+/−), reason (AwsCredit, AwsRefund, AwsSupportFee, AwsCorrection), created_at. Written when the billing job re-reads the prior month and finds AWS changed it after invoicing (R1).

**PlatformPayment**: hosting_invoice_id, provider ids, method (PaymentLink, UpiAutopay, EMandate), amount, status.

**UsageRecord**: tenant_id, metric (Messages, StorageGB, Bookings, ApiCalls), period, quantity (for cost-to-serve and the later review of a weighted split, R9; nothing here is billed; bookings never billed). Media fair use is 10 GB per tenant with no extra charge in v1 (Pilot Rules #10).

**Pricing JSON** (file, not a table; R12): written after each billing run with the per-space average share for last month and the two before it. It never contains the total AWS bill, the tenant count or any tenant's data.

### 3.11A Platform admin

**PilotRequest** (Book-a-demo form, CEO-A2): id, name, phone_e164, email, space_name, city, seats, aggregators_used, message (plain text, ≤ 2,000 characters), status, notify_status (Pending, Sent, Failed), ip_hash, created_at, updated_at. Platform schema, outside tenant RLS, readable only by the Super Admin role. Saved before sales is notified through the email outbox. A second request from the same phone within 24 hours updates the existing row and sends one notification. Retained 12 months.

Pilot scorecard (CEO-A4, R20): computed on demand, not stored. A paid online booking is customer-made in the booking app (`source = Online`), has a captured payment and is not fully refunded; staff-created and offline bookings are excluded. It counts in the month its payment was captured, in Asia/Kolkata time; a later full refund removes it from that month and past months recompute. The page shows "as of <time>".

### 3.12 Audit and support

**AuditEvent** (append-only): id, tenant_id null (platform events), actor_type, actor_id, impersonated_by null, action (e.g. `pricing.updated`, `refund.override`), entity_type, entity_id, before (jsonb), after (jsonb), ip, user_agent, at. Partitioned by month.

**SupportGrant**: id, tenant_id, requested_by (platform user), ticket_ref, reason, access (ReadOnly, ReadWrite), status (Requested, Approved, Denied, Revoked, Expired), approved_by (tenant user), starts_at, expires_at. Tenant-approved; applies after handover (R19).

**PreHandoverGrant** (R19): id, tenant_id (unique), created_by (platform user), created_at (with Create pilot), access (ReadWrite), status (Active, Ended), ended_at, end_reason (OwnerAccepted, Published, LivePayment). Valid only while the tenant has no Owner, is unpublished and has no live payments; every request re-checks these conditions, and writes are rejected once any fails. It ends automatically when the Owner accepts the invite. Every write under it is an AuditEvent with `impersonated_by` set, and the first-login handoff screen lists those events (11A).

## 4. Data rules and constraints

| Rule | Mechanism |
|------|-----------|
| No overlapping allocations on a unit | GiST exclusion constraint on `booking_allocation` |
| No overselling of pool capacity | Conditional update on `day_capacity_ledger` + CHECK |
| Invoices immutable after issue | No UPDATE of financial columns (trigger) except status/paid; corrections by credit note |
| Gap-free invoice numbers per series per FY | Row lock on `invoice_series` inside the invoice transaction |
| Webhook idempotency | Unique `(provider, provider_event_id)`; unique `provider_payment_id` |
| Booking creation idempotency | Unique `(tenant_id, idempotency_key)` |
| Policy at time of booking applies | `cancellation_policy_snapshot` on booking |
| Price at time of booking applies | `price_snapshot` on booking |
| Hostnames unique across platform | Unique index on `domain.hostname` where status ≠ Removed |
| Soft delete for configuration | `archived_at` on products, units, coupons, membership plans (referenced by history) |
| Hard delete / anonymise for personal data | Erasure job replaces personal fields with tokens; keeps financial records with masked buyer details unless law needs them |
| Tenant isolation fails closed | RLS policies use `current_setting('app.tenant_id', true)`; `FORCE ROW LEVEL SECURITY` on every tenant table; the tenant is set as the first statement of each explicit transaction (R3) |
| Sweeps find work across tenants without reading data | `cowos_sweeper` role with read-only policies exposing only `(id, tenant_id)` of matching rows (R8), served by partial indexes: `booking(hold_expires_at) WHERE status = 'Held'`; `booking(starts_at) WHERE status IN ('Confirmed','AwaitingPayment')`; `booking(ends_at) WHERE status = 'Confirmed'`. An integration test asserts each sweep's query plan uses its index (R7) |
| Wallet never passes its floor | Conditional `UPDATE` with the floor in the `WHERE` clause and the WalletTransaction in the same transaction; `CHECK (balance >= -500)` (R4) |
| Newest website data stays live | `tenant.data_seq` bumped in the same transaction as every website-visible data change and every tenant state change; pointer `(live_version_id, live_data_seq)` compared at the switch (R18) |
| Edge equals DB | `edge_sync` row in the same transaction as every pointer, state or hostname change; partial index on unsent rows (R17) |
| Pass tokens are purpose-bound | Unique `booking_pass.token_hash`; lookup requires purpose `pass`, not expired, not revoked, same tenant (R10) |
| One hosting share per tenant per month | Unique `(cost_pool_id, tenant_id)` on `tenant_cost_share`; unique `month` on `cost_pool` (P1-B) |
| Scorecard query | Index on payments `(tenant_id, captured_at)` filtered to captured payments of online bookings (R20) |
