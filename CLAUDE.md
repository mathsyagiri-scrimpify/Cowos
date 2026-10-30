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
- Decisions D1–D16 in `docs/requirements/13-risks-open-questions-decisions.md` are still Proposed. Don't treat them as approved.

## Fixed stack (when code starts)
ASP.NET Core (.NET LTS) API, React + TypeScript, PostgreSQL, AWS (ap-south-1).
