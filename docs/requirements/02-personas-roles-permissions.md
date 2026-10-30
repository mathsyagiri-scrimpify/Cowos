# 02 · Personas, Roles & Permissions

> Cowos requirements pack · v0.1 draft

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
- Platform operations lead. Approves first publishes, manages plans and pricing, handles escalations, watches platform health.

### P6. Divya, SSPL Support Executive
- Answers tenant tickets ("my domain isn't working", "Razorpay not connecting"). Needs to see a tenant's configuration and, with the tenant's permission, view their admin as they see it.
- **Must not:** see customers' personal data or payment details without an approved, time-limited support grant.

### P7 (recommended addition). Anil, SSPL Finance
- Manages platform invoices, failed renewals, GST filing data for SSPL's own revenue, refunds of platform fees.
- Recommended as a separate role so Support can't issue platform credits and Finance can't impersonate tenants.

## 2. Role definitions

| Role | Scope | Assigned by | Authentication |
|------|-------|-------------|----------------|
| Super Admin | Whole platform | SSPL (max 3 people) | SSO (Google Workspace / Entra ID) + hardware key or TOTP MFA, IP allowlist |
| SSPL Support | Whole platform, read-mostly | Super Admin | SSO + MFA |
| SSPL Finance (recommended) | Platform billing only | Super Admin | SSO + MFA |
| Tenant Owner | One tenant (all locations) | Self (sign-up). Exactly one per tenant; transferable | Email + password + mandatory TOTP or email OTP MFA; Google sign-in optional |
| Tenant Manager (recommended addition) | One tenant, chosen locations | Tenant Owner | Email + password + MFA |
| Tenant Staff / Front desk | One tenant, chosen locations | Tenant Owner or Manager | Email or phone OTP; MFA optional |
| Customer | One tenant's booking app | Self (sign-up) or staff-created | Phone OTP (primary) or email OTP / magic link |
| Customer Company Admin | Their company within one tenant | Tenant staff or self when creating a company | Same as Customer |

Tenant Manager is recommended because operators with 2+ locations need a centre manager who can run operations and pricing for their centre without owning billing or the website.

## 3. Permissions matrix

Legend: ✅ full · 👁 view only · 🔒 only with an active, tenant-approved support grant (time-limited, audited) · 📍 limited to assigned locations · ⚙ own records only · — no access

### 3.1 Platform-level capabilities

| Capability | Super Admin | SSPL Support | SSPL Finance | Tenant Owner | Tenant Manager | Tenant Staff | Customer |
|------------|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| View list of all tenants, status, plan | ✅ | 👁 | 👁 | — | — | — | — |
| Approve / reject first publish (KYC) | ✅ | ✅ | — | — | — | — | — |
| Unpublish / suspend a tenant site | ✅ | — (can request) | — | — | — | — | — |
| Reactivate a suspended tenant | ✅ | — | ✅ (after payment) | — | — | — | — |
| Delete a tenant (after retention period) | ✅ (two-person approval) | — | — | — | — | — | — |
| Manage platform plans and prices | ✅ | — | 👁 | — | — | — | — |
| Issue platform invoice credits / refunds | ✅ | — | ✅ | — | — | — | — |
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
| Manage Cowos subscription, payment method | ✅ | — | — | — | Finance: 👁 |
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
3. Support access ("🔒"): Support requests access with a ticket reference and reason → Tenant Owner approves in-app or by email link → grant lasts up to 24 hours, read-only by default, write access needs a separate approval → every action is logged with both identities ("Divya (SSPL Support) acting for Workloft"). The tenant sees active grants and can revoke them.
4. Super Admin can suspend or unpublish without a grant (platform safety), but cannot read customer personal data without one, except where legally required, which is logged as a "break-glass" action and reviewed weekly.
5. No SSPL role can see a tenant's payment gateway secrets. They are encrypted with a KMS key that only the payment service's IAM role can use.
