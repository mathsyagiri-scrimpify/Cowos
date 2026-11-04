# TODOS

## Pricing

### Review slab prices against real usage and cost

**What:** After 3 months of live billing, compare each meter's billed revenue with SSPL's actual AWS and messaging-provider cost (Super Admin cost view) and with how tenants spread across the slabs, then adjust slab boundaries and prices in a new rate card version.

**Why:** Slab prices are set before any real traffic exists; prices that are too low leave SSPL paying for the server, and prices that are too high push small spaces away.

**Context:** Metered slab pricing replaced the AWS-cost equal split on 2026-11-04 (design doc `docs/designs/cowos-platform-v1.md`, Pricing and Billing Rules #3–#5 and #10). Rate card versions are effective-dated, so a change applies from the 1st of a month. Start from the cost view and the slab distribution per meter.

**Effort:** S
**Priority:** P2
**Depends on:** 3 months of live billing; Open Question 5 (notice period for rate changes)

## Themes

### Build themes 3-4 after validation with early tenants

**What:** Validate the four 24A directions with 3 early tenant owners, then build the 2 not shipped at launch (Courtyard and Campus unless validation says otherwise), with their RT7 booking-shell chunks, Lighthouse budgets, axe checks and screen-reader pass. Release about 6 weeks after launch.

**Why:** Keeps theme design and build out of weeks 22-24, when the launch gate is decided, and follows 24A's own validation rule.

**Context:** Deferred by /plan-ceo-review CEO-S3 (D4, 2026-11-03); kept by the 2026-11-04 revision. Launch ships 2 themes with all 24A rules and 23A tokens. The launch checklist item reads "every launched theme".

**Effort:** M
**Priority:** P2
**Depends on:** launch; validation with 3 early tenant owners

## Completed

- **Cancelled 2026-11-04: weight the hosting-cost split.** Tenants are billed by metered units, not a share of the AWS bill.
- **Cancelled 2026-11-04: Billing release at launch + 4 months.** Billing, metering, the rate card and self-serve sign-up are launch scope again, because there are no free pilots.
- **Cancelled 2026-11-04: ship the wizard with the Billing release.** The onboarding wizard (4A) is launch scope again with self-serve sign-up.
