# Cowos

SSPL's product: a white-label website + booking app platform for independent coworking spaces (Directoo model applied to coworking). No application code yet; the repo holds the requirements pack.

## Start here
- `docs/HANDOFF.md`: current status, what is decided vs open, and where the strategy review stopped. Read first.
- `docs/original-brief.md`: the product owner's original brief and later clarifications.
- `docs/requirements/`: requirements pack v0.4 (start with `00-start-here.md`).

## Ground rules from the product owner
- "0% commission" means Cowos takes no share of a tenant's booking revenue. What tenants charge their customers (prices, promo codes, fees) is the tenant's decision.
- Tenants create their own promo codes in the Tenant Admin.
- Docs live at project level under `docs/`.
- Each tenant chooses how it pays SSPL (D1, revision 2 on 2026-11-04, Pricing Rule #12): a **Fixed plan** (one flat monthly fee set by SSPL; server usage unlimited and not metered for billing; messages included up to a monthly fair-use allowance per channel, then paid from the wallet) or a **Usage plan** (no fee; server usage and notifications metered in slabs whose prices SSPL configures, like an electricity bill). Switching applies from the next 1st. Every feature on both plans, no commission, no free pilot programme. Hosting is one app server plus managed PostgreSQL (D2, decided 2026-11-04).
- Owners build a grid seat map of their space (templates, paint desks/cabins/rooms) and customers pick a seat like a movie ticket, at launch (Pricing Rule #13 and "Seat maps (revision 2)" in the design doc).
- The owner wants every app finished before selling to tenants, as fast as possible: build with a small team using Claude Code, following `docs/requirements/14-build-plan.md` (launch target week 20, estimated).
- The pack is v0.4 (2026-11-04). Revision 2 has not been through the CEO, design or eng reviews yet.
- The approved design doc is `docs/designs/cowos-platform-v1.md` (premises, pricing and billing rules, chosen approach).
- Decisions D3–D16 in `docs/requirements/13-risks-open-questions-decisions.md` are still Proposed. Don't treat them as approved.

## Fixed stack (when code starts)
ASP.NET Core (.NET LTS) API, React + TypeScript, PostgreSQL, AWS (ap-south-1).

## Skill routing

When the user's request matches an available skill, invoke it via the Skill tool. Route only to skills in the session's available-skills list; answer directly for quick questions or small scoped edits.

Key routing rules:
- Product ideas/brainstorming → invoke /office-hours
- Strategy/scope → invoke /plan-ceo-review
- Architecture → invoke /plan-eng-review
- Design system/plan review → invoke /design-consultation or /plan-design-review
- Full review pipeline → invoke /autoplan
- Bugs/errors → invoke /investigate
- QA/testing site behavior → invoke /qa or /qa-only
- Code review/diff check → invoke /review
- Visual polish → invoke /design-review
- Ship/deploy/PR → invoke /ship or /land-and-deploy
- Save progress → invoke /context-save
- Resume context → invoke /context-restore
- Author a backlog-ready spec/issue → invoke /spec
