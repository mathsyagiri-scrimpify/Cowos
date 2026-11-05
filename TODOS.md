# TODOS

## Pricing

### Review slab prices against real usage and cost

**What:** After 3 months of live billing, compare each meter's billed revenue with SSPL's actual AWS and messaging-provider cost (Super Admin cost view) and with how tenants spread across the slabs, then adjust slab boundaries and prices in a new rate card version.

**Why:** Slab prices are set before any real traffic exists; prices that are too low leave SSPL paying for the server, and prices that are too high push small spaces away.

**Context:** Metered slab pricing replaced the AWS-cost equal split on 2026-11-04 (design doc `docs/designs/cowos-platform-v1.md`, Pricing and Billing Rules #3–#5 and #10). Rate card versions are effective-dated, so a change applies from the 1st of a month. Start from the cost view and the slab distribution per meter.

**Effort:** S
**Priority:** P2
**Depends on:** 3 months of live billing; Open Question 5 (notice period for rate changes)

### Set the Fixed plan fee, allowances and slab prices together

**What:** Choose the Fixed plan fee and its monthly message allowances (Pricing Rule #12) together with the Usage plan slab prices, so that a typical small space pays about the same on either plan and a busy space is better off on Fixed, then enter them in the first rate card version.

**Why:** Owners compare the two plans side by side (pricing page, billing page); if one is always much cheaper, the other is pointless. The allowances also cap SSPL's biggest variable cost, messages.

**Context:** Revision 2 (2026-11-04). Pack OQ35 and OQ21. Needed by week 12 of the build plan (`docs/requirements/14-build-plan.md` §5). The sample bills on the pricing page show both plans, so check them before publishing.

**Effort:** S
**Priority:** P1
**Depends on:** before launch (rate card in force is a launch checklist item)

## Seat maps

### Test the seat map builder with 3 early tenant owners in M3b

**What:** Have 3 owners draw their real space with the builder, unaided, then book a picked seat as a customer. Note where they get stuck, and adjust templates and tools before launch.

**Why:** The builder is the riskiest new piece of revision 2 (pack R31). If owners can't finish a map, the seat picker has nothing to show.

**Context:** Design doc "Seat maps (revision 2)", pack FR-TA-068 to 068e, US-4.6 to US-4.8, US-6.4. Also confirm the 10 maps / 40 × 60 limits (OQ36).

**Effort:** S
**Priority:** P1
**Depends on:** M3b (weeks 16-19)

### Floor photo behind the seat map grid (only if owners ask)

**What:** Let an owner upload a floor plan image and show it faintly behind the grid so desks line up with the real room.

**Why:** Some owners may want the map to look like the real floor. Not needed for seat picking.

**Context:** Offered as option "Both" in revision 2 and not chosen; about 1-2 weeks [estimated].

**Effort:** S
**Priority:** P3
**Depends on:** feedback after launch

## Deferred by CEO review 3 (2026-11-05, scope reduction for the fastest launch)

### SSPL cost view and CUR import (R3-C2)

**What:** Build the Super Admin cost view (FR-SA-022) and the monthly AWS CUR 2.0 import (FR-PL-069) as designed.

**Why:** Shows SSPL's real AWS and messaging cost next to billed revenue, including whether each Fixed plan tenant costs more than its fee.

**Context:** Deferred from launch by CEO review 3 (D3). Until it ships, SSPL reads the AWS bill and the per-tenant usage counts by hand.

**Effort:** M
**Priority:** P1
**Depends on:** launch + about 4 weeks

### Rate card drafts, diff and impact preview (R3-C3)

**What:** Add drafts (save, duplicate, delete), the side-by-side diff and the impact preview to the rate card editor (DR2-7A, FR-SA-020).

**Why:** Needed before SSPL changes prices for tenants who already have a billed month.

**Context:** Deferred by CEO review 3 (D4). The launch editor keeps versions, the Fixed fee and allowances, slab validation and the typed publish confirmation.

**Effort:** M
**Priority:** P1
**Depends on:** before the first price change after launch

### Today on the map and "Seat us together" (R3-C4)

**What:** Build the front-desk "Today on the map" view with drag-to-move (FR-TA-068e, US-4.8) and the adjacent-seat suggestion for group bookings (FR-BA-013).

**Why:** Makes daily front-desk work and group bookings faster.

**Context:** Deferred by CEO review 3 (D5). At launch staff move a booking with a "Change seat" list on the booking detail, and groups pick seats one by one.

**Effort:** M
**Priority:** P2
**Depends on:** launch

### Super Admin business dashboard (R3-C5)

**What:** Build the business dashboard (FR-SA-051): sign-ups, trials, conversions, revenue, churn, referrals.

**Why:** One view of the business once there is more than a month of history.

**Context:** Deferred by CEO review 3 (D6). Until then SSPL uses the tenant list filters and the scorecard CSV.

**Effort:** S
**Priority:** P2
**Depends on:** launch + about 4 weeks

## Themes

### Build themes 3-4 after validation with early tenants

**What:** Validate the four 24A directions with 3 early tenant owners, then build the 2 not shipped at launch (Courtyard and Campus unless validation says otherwise), with their RT7 booking-shell chunks, Lighthouse budgets, axe checks and screen-reader pass. Release about 6 weeks after launch.

**Why:** Keeps theme design and build out of weeks 26-28 (weeks 22-24 before revision 2), when the launch gate is decided, and follows 24A's own validation rule.

**Context:** Deferred by /plan-ceo-review CEO-S3 (D4, 2026-11-03); kept by the 2026-11-04 revision. Launch ships 2 themes with all 24A rules and 23A tokens. The launch checklist item reads "every launched theme".

**Effort:** M
**Priority:** P2
**Depends on:** launch; validation with 3 early tenant owners

## Completed

- **Cancelled 2026-11-04: weight the hosting-cost split.** Tenants are billed by metered units, not a share of the AWS bill.
- **Cancelled 2026-11-04: Billing release at launch + 4 months.** Billing, metering, the rate card and self-serve sign-up are launch scope again, because there are no free pilots.
- **Cancelled 2026-11-04: ship the wizard with the Billing release.** The onboarding wizard (4A) is launch scope again with self-serve sign-up.
