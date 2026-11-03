# 02 · Personas, Roles & Permissions

> Cowos requirements pack · v0.2 draft

## 1. Personas

### P1. Ravi, Tenant Owner (independent operator)
- 38, runs "Workloft", one 60-seat space in Madhapur, Hyderabad. Former IT manager.
- Today: Instagram page, Google Business Profile, WhatsApp group with members, Excel for invoices, Google Form for enquiries.
- Lists on two aggregators and dislikes that they show competitors next to him.
- **Wants:** a site that looks as good as the big chains, meeting rooms bookable without him on the phone, GST invoices that don't need his accountant.
- **Fears:** hidden costs, getting locked in, spending weekends on setup.
- **Cowos success:** site live on `workloft.in` in an afternoon, first online meeting-room booking the same week.

### P2. Sneha, Tenant Staff (community manager / front desk)
- 26, works the front desk 9–6. Handles walk-ins, visitor entries, room conflicts, payment follow-ups.
- **Wants:** one screen showing today's bookings and who is checked in; record a cash payment in 10 seconds; resend an invoice.
- **Must not:** change pricing, see payouts, edit the website.

### P3. Arjun, Customer (freelancer / day-pass user)
- 29, UX freelancer, works from a coworking space 2–3 days a week, books on his phone on the way.
- **Wants:** see if seats are free today, pay by UPI, a QR code to show at the desk.

### P4. Meera, Customer (startup admin booking for a team)
- 34, office manager at a 12-person startup on a private cabin plan.
- **Wants:** book the boardroom for client meetings using the plan's included hours, invoices with the company GSTIN, add/remove team members.

### P5. Kiran, SSPL Super Admin
- Platform operations lead. Creates pilot tenants and sends Owner invites, triages pilot requests, watches the pilot scorecard, approves first publishes, handles escalations, watches platform health (CEO-A2, CEO-A4). From the Billing release, also runs the monthly hosting billing with Finance (D1, P1-B).

### P6. Divya, SSPL Support Executive
- Answers tenant tickets ("my domain isn't working", "Razorpay not connecting"). Needs to see a tenant's configuration and, with the tenant's permission, view their admin as they see it.
- **Must not:** see customers' personal data or payment details without an approved, time-limited support grant.

### P7 (recommended addition). Anil, SSPL Finance
- Manages hosting invoices, held billing runs, adjustment lines, failed hosting payments, GST filing data for SSPL's hosting invoices, and refunds of hosting charges (D1, Pilot Rules #10, P1-B).
- Recommended as a separate role so Support can't issue hosting credits and Finance can't impersonate tenants.

## 2. Role definitions

| Role | Scope | Assigned by | Authentication |
|------|-------|-------------|----------------|
| Super Admin | Whole platform | SSPL (max 3 people) | SSO (Google Workspace / Entra ID) + hardware key or TOTP MFA, IP allowlist |
| SSPL Support | Whole platform, read-mostly | Super Admin | SSO + MFA |
| SSPL Finance (recommended) | Hosting billing only (P1-B) | Super Admin | SSO + MFA |
| Tenant Owner | One tenant (all locations) | Pilots (launch): Super Admin Owner invite on Create pilot (CEO-A2). Non-pilots: self sign-up from the Billing release (CEO-S1, P1-B). Exactly one per tenant; transferable | Email + password + mandatory TOTP or email OTP MFA; Google sign-in optional |
| Tenant Manager (recommended addition) | One tenant, chosen locations | Tenant Owner | Email + password + MFA |
| Tenant Staff / Front desk | One tenant, chosen locations | Tenant Owner or Manager | Email or phone OTP; MFA optional |
| Customer | One tenant's booking app | Self (sign-up) or staff-created | Phone OTP (primary) or email OTP / magic link |
| Customer Company Admin | Their company within one tenant | Tenant staff or self when creating a company | Same as Customer |

Tenant Manager is recommended because operators with 2+ locations need a centre manager who can run operations and pricing for their centre without owning billing or the website.

There are no paid tiers. Every tenant gets every feature, so no permission depends on a plan (D1, Pilot Rules #2).

## 3. Permissions matrix

Legend: ✅ full · 👁 view only · 🔒 only with an active support grant (time-limited, audited): tenant-approved after handover, or the pre-handover setup grant before a pilot has an Owner (R19, §3.3 rule 6) · 📍 limited to assigned locations · ⚙ own records only · — no access

### 3.1 Platform-level capabilities

| Capability | Super Admin | SSPL Support | SSPL Finance | Tenant Owner | Tenant Manager | Tenant Staff | Customer |
|------------|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| View list of all tenants, status, pilot flag | ✅ | 👁 | 👁 | — | — | — | — |
| Create pilot tenant (Active, pilot flag) and send the Owner invite (CEO-A2) | ✅ | — | — | — | — | — | — |
| View and triage pilot requests from the Book-a-demo form (CEO-A2) | ✅ | — | — | — | — | — | — |
| View pilot scorecard and export CSV (CEO-A4) | ✅ | — | — | — | — | — | — |
| Set up a pilot before handover (pre-handover setup grant, R19) | 🔒 | 🔒 | — | — | — | — | — |
| Approve / reject first publish (KYC) | ✅ | ✅ | — | — | — | — | — |
| Unpublish / suspend a tenant site | ✅ | — (can request) | — | — | — | — | — |
| Reactivate a suspended tenant | ✅ | — | ✅ (after payment, P1-B) | — | — | — | — |
| Delete a tenant (after retention period) | ✅ (two-person approval) | — | — | — | — | — | — |
| Manage platform plans and prices | Removed in v0.2 (no plans; hosting cost only, D1) | | | | | | |
| View cost pools, billing runs and hosting invoices (28A, P1-B) | ✅ | — | ✅ | — | — | — | — |
| Issue hosting invoice credits / refunds (P1-B) | ✅ | — | ✅ | — | — | — | — |
| Manage theme catalog | ✅ | 👁 | — | — | — | — | — |
| Platform settings (messaging providers, feature flags) | ✅ | — | — | — | — | — | — |
| View platform health dashboards | ✅ | ✅ | — | — | — | — | — |
| Request support access to a tenant | ✅ | ✅ | — | — | — | — | — |
| Approve / revoke support access | — | — | — | ✅ | — | — | — |
| View tenant configuration (theme, inventory, domains) | ✅ | ✅ | — | ✅ | 👁 | — | — |
| View tenant's customer personal data | 🔒 | 🔒 | — | ✅ | 📍 | 📍 | ⚙ |
| View global audit log | ✅ | 👁 | — | — | — | — | — |

### 3.2 Tenant-level capabilities

| Capability | Tenant Owner | Tenant Manager | Tenant Staff | Customer | Super Admin / Support |
|------------|:-:|:-:|:-:|:-:|:-:|
| Edit business profile, GSTIN, legal details | ✅ | 👁 | — | — | 🔒 |
| Choose theme, edit branding and website content | ✅ | ✅ (if granted "Website editor") | — | — | 🔒 |
| Preview site | ✅ | ✅ | — | — | 🔒 |
| Publish / roll back website | ✅ | ✅ (if granted) | — | — | Super Admin can unpublish only |
| Connect domains | ✅ | — | — | — | 🔒 (assist) |
| Connect / disconnect payment gateway | ✅ | — | — | — | — (never sees keys) |
| Manage hosting billing: payment method (payment link details or UPI Autopay / e-mandate), messaging wallet top-up (D1, Pilot Rules #10, P1-B) | ✅ | — | — | — | Finance: 👁 |
| Invite / remove staff, assign roles | ✅ | 📍 (Staff only) | — | — | — |
| Create / edit locations | ✅ | 📍 (edit) | — | — | 🔒 |
| Create / edit spaces, resources, photos | ✅ | 📍 | — | — | 🔒 |
| Set prices, add-ons, availability rules | ✅ | 📍 | — | — | 🔒 |
| Set cancellation and refund policies | ✅ | 📍 | — | — | — |
| Create / edit / pause promo codes | ✅ | 📍 | Apply only (on staff bookings) | ⚙ apply at checkout | — |
| Membership plans (create, price) | ✅ | 📍 | — | — | — |
| View bookings calendar | ✅ | 📍 | 📍 | ⚙ | 🔒 |
| Create booking on behalf of customer | ✅ | 📍 | 📍 | — | — |
| Approve / decline booking requests | ✅ | 📍 | 📍 | — | — |
| Cancel booking (with policy) | ✅ | 📍 | 📍 (policy refund only) | ⚙ (within policy) | — |
| Override policy (full refund, waive fee) | ✅ | 📍 | — | — | — |
| Reschedule booking | ✅ | 📍 | 📍 | ⚙ (within policy) | — |
| Check in / mark no-show | ✅ | 📍 | 📍 | ⚙ (self check-in via QR, Phase 2) | — |
| Record offline payment | ✅ | 📍 | 📍 | — | — |
| Issue refund via gateway | ✅ | 📍 (up to limit set by Owner) | — | — | — |
| View payments and payouts summary | ✅ | 📍 | — | ⚙ (own payments) | 🔒 |
| Issue / void invoice, issue credit note | ✅ | 📍 | 👁 / resend only | ⚙ download own | 🔒 |
| View / edit customers and companies | ✅ | 📍 | 📍 (view, edit contact) | ⚙ own profile, company admin manages members | 🔒 |
| Export customer data | ✅ | — | — | ⚙ (own data, DPDP request) | — |
| Delete customer (DPDP erasure) | ✅ | — | — | ⚙ request deletion | — |
| Manage leads and tours | ✅ | 📍 | 📍 | ⚙ (submit enquiry) | — |
| Notification templates and channels | ✅ | 👁 | — | ⚙ (own preferences) | — |
| Reports and analytics | ✅ | 📍 | Today view only | — | 🔒 |
| Tenant audit log | ✅ | 👁 📍 | — | — | 👁 |

### 3.3 Permission rules
1. Permissions are evaluated server-side on every request. The UI hides actions, but the API is the source of truth.
2. Tenant roles are a fixed set in MVP. Phase 2 adds per-permission toggles on top of roles (e.g. "Manager without refund rights").
3. Support access ("🔒") once the tenant has an Owner: Support requests access with a ticket reference and reason → Tenant Owner approves in-app or by email link → grant lasts up to 24 hours, read-only by default, write access needs a separate approval → every action is logged with both identities ("Divya (SSPL Support) acting for Workloft"). The tenant sees active grants and can revoke them. Tenant-approved grants apply after handover (R19, Pilot Rules #9).
4. Super Admin can suspend or unpublish without a grant (platform safety), but cannot read customer personal data without one, except where legally required, which is logged as a "break-glass" action and reviewed weekly.
5. No SSPL role can see a tenant's payment gateway secrets. They are encrypted with a KMS key that only the payment service's IAM role can use.
6. Pre-handover setup grant (R19): Create pilot also creates a setup grant with write access, so SSPL can set up inventory, theme and content before the owner's first login (Pilot Rules #9). The grant is valid only while the tenant has no Owner, is unpublished and has no live payments. It ends automatically when the Owner accepts the invite. Every write is audited, and the first-login handoff screen lists those changes ("Review what we set up", 11A). After handover, rule 3 applies.
7. Owner invite for pilots (CEO-A2): the invite reuses the staff invite flow (FR-TA-172) with the Owner role. The link is single use and expires after 72 hours; a resend revokes the old link. Accepting verifies the email by the link and the phone by OTP; the owner sets the password and MFA and accepts the ToS and DPA. Creating the pilot and sending the invite are audited like FR-SA-011.
8. In pilot onboarding the owner does Razorpay KYC and OAuth, GSTIN/PAN, DNS changes and ToS/DPA acceptance personally; SSPL's setup covers inventory, theme and content (Pilot Rules #9).
