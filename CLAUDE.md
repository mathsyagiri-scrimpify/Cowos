# Cowos

SSPL's product: a white-label website + booking app platform for independent coworking spaces (Directoo model applied to coworking). No application code yet; the repo holds the requirements pack.

## Start here
- `docs/HANDOFF.md`: current status, what is decided vs open, and where the strategy review stopped. Read first.
- `docs/original-brief.md`: the product owner's original brief and later clarifications.
- `docs/requirements/`: requirements pack v0.1 (start with `00-start-here.md`).

## Ground rules from the product owner
- "0% commission" means Cowos takes no share of a tenant's booking revenue. What tenants charge their customers (prices, promo codes, fees) is the tenant's decision.
- Tenants create their own promo codes in the Tenant Admin.
- Docs live at project level under `docs/`.
- SSPL charges tenants only their share of the real AWS hosting cost, plus messaging at cost: no subscription, plan fee or commission (D1, decided 2026-11-02). The pack was updated to v0.2 on 2026-11-03 to match.
- The approved design doc is `docs/designs/cowos-platform-v1.md` (premises, pilot rules, chosen approach).
- Decisions D2–D16 in `docs/requirements/13-risks-open-questions-decisions.md` are still Proposed. Don't treat them as approved.

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
