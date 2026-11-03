# TODOS

## Billing

### Decide whether to weight the hosting-cost split

**What:** After 3 months of real AWS bills, decide whether to weight each tenant's hosting-cost share (for example by bookable resources, page views or storage) instead of the equal split.

**Why:** An equal split makes small coworking spaces pay for large ones as the tenant mix diverges, which can feel unfair under the "real cost only" promise.

**Context:** v1 ships an equal split by tenant-days (design doc `docs/designs/cowos-platform-v1.md`, Pilot Rules #10; engineering review D3). CostPool and TenantCostShare already store per-tenant rows, so a weight column can be added later. Start by comparing per-tenant CloudFront and S3 usage from AWS cost-allocation tags. Raised as Open Question 4 in the design doc; tracked by /plan-eng-review D10 (2026-11-03).

**Effort:** M
**Priority:** P3
**Depends on:** 3 months of production AWS bills after the Billing release (CEO-S1); per-tenant cost-allocation tags

## Billing release (launch + 4 months)

### Ship platform billing automation and open non-pilot self-serve sign-up

**What:** Build and release the CUR 2.0 MonthlyHostingBilling job with its billing tables, payment-link and UPI Autopay/e-mandate collection, dunning with automatic Restricted/Suspended moves (the dunning triggers for RT4; RT4 itself ships at launch for FR-SA-011, CEO-A1), the pricing JSON (RT5), the daily charge estimate (RT6), the 14-day trial, non-pilot wallet top-up and public self-serve sign-up. Start SSPL's paid test tenant at this release.

**Why:** No pilot pays during its free year, so this machinery had no paying user at launch. Deferring it frees about 3 weeks of one backend and one frontend engineer before the launch gate [estimated].

**Context:** Deferred by /plan-ceo-review CEO-S1 (D2, 2026-11-03). The design stays as approved in the eng review (D2/D3) and re-review (D4-D6); see `docs/designs/cowos-platform-v1.md` "CEO Review". Pilot Rules #7 exit criterion is unchanged: one real hosting charge paid and one dunning cycle run on production before the first pilot converts (launch + 12 months). Until this ships, Super Admin creates pilots and suspends or reinstates tenants (FR-SA-011), and the marketing site's Book-a-demo form is the only intake (CEO-A2). The release adds the billing tables as additive migrations, backfills a BillingProfile for every existing tenant, bills only from the first full calendar month after release, rolls out in the order deploy dark → SSPL paid test tenant → open sign-up, moves FR-SA-051 metrics here, and starts the CEO-A5 cost-recovery tracking from CostPool.

**Effort:** L
**Priority:** P1
**Depends on:** launch; must ship before the first pilot's free year ends

### Ship the self-serve onboarding wizard with public sign-up

**What:** Build the 4A wizard flow (goal → business/GSTIN → location → theme → branding → inventory → policies → payments → preview → publish), GSTIN lookup, Places autocomplete, demo inventory with the Sample label and its publish check, and the booking-only path.

**Why:** Only self-serve sign-ups use the wizard; pilots are set up by SSPL through a support grant (Pilot Rules #9).

**Context:** Deferred by /plan-ceo-review CEO-S2 (D3, 2026-11-03). Launch keeps every settings screen and an owner "left to do" list on the Tenant Admin home. Design decision 4A is unchanged; collect pilot setup notes from the onboarding owner to refine it.

**Effort:** M
**Priority:** P1
**Depends on:** the Billing release (ships with public sign-up)

## Themes

### Build themes 3-4 after pilot validation

**What:** Validate the four 24A directions with 3 pilot owners, then build the 2 not shipped at launch (Courtyard and Campus unless validation says otherwise), with their RT7 booking-shell chunks, Lighthouse budgets, axe checks and screen-reader pass. Release about 6 weeks after launch.

**Why:** Keeps theme design and build out of weeks 22-24, when the launch gate is decided, and follows 24A's own validation rule.

**Context:** Deferred by /plan-ceo-review CEO-S3 (D4, 2026-11-03). Launch ships 2 themes with all 24A rules and 23A tokens. The launch checklist item now reads "every launched theme".

**Effort:** M
**Priority:** P2
**Depends on:** launch; validation with 3 pilot owners

## Completed
