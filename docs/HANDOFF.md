# Cowos · Handoff Notes

> Handed off on 2026-10-30 to **Bharath** (picking up in their own Claude Code with their own credentials).
> Read this file first, then `docs/requirements/00-start-here.md`.

## 1. Where everything is

| Item | Location |
|------|----------|
| Repository | https://github.com/mathsyagiri-scrimpify/Cowos (branch `main`) |
| Requirements pack (13 deliverables + start-here) | `docs/requirements/` (index: `docs/requirements/README.md`) |
| Original brief from the product owner (verbatim) + later clarifications | `docs/original-brief.md` |
| Requirements pack commit | `c030982` "Add initial documentation for Cowos requirements pack". Includes the promo-code and "0% commission" clarifications |
| This handoff note | `docs/HANDOFF.md` (committed together with `docs/original-brief.md` and `CLAUDE.md` in the handoff commit) |
| Context file Claude Code loads automatically | `CLAUDE.md` at the repo root (points here) |

**Check before starting:** `git log --oneline -3` should show a handoff commit on top of `c030982`. If `docs/HANDOFF.md` is missing from your clone, the handoff commit hasn't been pushed yet; ask the previous owner to push `main`.

Nothing else is needed from the previous machine. The `.claude/worktrees/...` folder there was a scratch copy and is not part of the repo.

**No secrets or credentials are in the repo, and none have been created yet** (no AWS, Razorpay, SMS, WhatsApp or domain accounts). Use your own accounts and keep keys out of git.

## 2. What is done

1. **Requirements pack v0.1** covering all 13 requested deliverables: vision, personas and permissions, BRD, SRS (~200 numbered FRs with MoSCoW), user stories (Given/When/Then), flows (Mermaid), NFRs, architecture, data model, API modules, integrations, release plan, risks and decisions.
2. **12 recommended changes to the original brief** are in `00-start-here.md` §2. The biggest:
   - flat plans instead of "server cost only" pricing
   - shared multi-tenant platform instead of one EC2 per tenant
   - pre-rendered tenant websites
   - each tenant's own Razorpay account instead of Razorpay Route
   - `cowos.site` for tenant sites, separate from `cowos.in`
   - "Powered by Cowos" badge
   - three booking modes (instant, request, enquire/tour)
3. **Product owner clarifications already applied:**
   - **0% commission means Cowos takes no share of the tenant's booking revenue.** What the tenant charges its own customers (prices, promo codes, discounts, optional convenience fee, deposits) is the tenant's decision. See `00` §C1, BR-14, BR-25.
   - **Tenants create their own promo codes** in the Tenant Admin on every plan, as an MVP must. See FR-TA-150/156/157/159, FR-BA-016, US-4.5, BR-26.
   - Added an optional tenant convenience fee (FR-TA-071, off by default). **Not explicitly approved yet:** confirm with the product owner or remove it.

## 3. What is NOT decided yet

All decisions in `docs/requirements/13-risks-open-questions-decisions.md` §3 (D1–D16) are still **Proposed**. None has been approved. The six that block engineering: D1 pricing model, D2 hosting model, D3 payment model, D4 domains, D5 badge text, D6 database tenancy strategy.

Open questions OQ1–OQ15 in the same file are also unanswered. The pack uses the default assumptions listed there.

## 4. Work in progress: strategy review (stopped mid-way)

The previous session started `/plan-ceo-review` (gstack). Since there was no design doc, it ran `/office-hours` inline first, in **startup mode** (goal: intrapreneurship at SSPL). It stopped at **Phase 3 (premise challenge)** when the work was handed over.

Exact position:

| Step | Status |
|------|--------|
| `/office-hours` Phase 1 (context), goal, stage | Done (see 4.1) |
| Phase 2A forcing questions (Q2 status quo, Q4 wedge; Q5 skipped as nothing to observe yet) | Done (see 4.1) |
| Phase 2.75 landscape search | Done (see 4.3) |
| Phase 3 premises | **Next.** The question call failed with a tool error, so the four premises in 4.4 have no answers |
| Phase 3.5 second opinion, Phase 4 alternatives, design doc | Not started |
| `/plan-ceo-review` Step 0 (premise challenge, mode selection) and the 11-section review | Not started. Nothing was written to `~/.gstack` design docs or review logs on the previous machine |

### 4.1 Answers already given by the product owner

| Question | Answer |
|----------|--------|
| Goal | Intrapreneurship (internal SSPL product) |
| Stage | Said "has interested users". On follow-up: **1–3 casual chats with coworking spaces, plus SSPL's own idea**. So effectively pre-product, demand unverified |
| Status quo (how spaces handle bookings today) | "Don't know". Likely all of: WhatsApp/calls/Excel, aggregators taking commission, a basic website with manual follow-up. Existing coworking software is less likely. Wants to "target all customers" with a strong USP |
| Narrowest wedge | Chose **full website + booking** (the complete MVP), not a smaller booking-only wedge |

### 4.2 Reviewer's position (for context, not a decision)

- Interest so far isn't demand. Nobody has yet shown pain or willingness to pay.
- "Target all customers" is risky: each segment's pain is different (admin hours vs aggregator commission vs no online presence).
- Full website + booking as the wedge is a 5–6 month bet before the first payment. **What would change this view:** 2+ owners saying they would only switch for a full branded website.

### 4.3 Landscape findings (web search, generic terms only)

- India already has India-first coworking tools: [Meshfloor](https://www.producthunt.com/posts/1237227), [Engage Apps](https://yourstory.com/companies/engage-apps) (claims 100+ brands), [CoWorkingOps](https://www.coworkingops.com.cropburning.in/), [Omnify](https://www.getomnify.com/case-study/10000-start-ups). "Cheap + GST" alone is not a unique position.
- Aggregator commission: a 2023 founder interview reports [myHQ takes 25–30% per booking](https://getlatka.com/companies/myhq/interviews/myhq-utkarsh-kawatra-2023). [Anarock bought 75% of myHQ](https://www.freepressjournal.in/amp/business/anarock-buys-75-stake-in-coworking-marketplace-myhq-for-rs-125-cr).
- [Spacebring argues](https://www.spacebring.com/blog/coworking-software-where-uk-operators-still-lose-time) operators mostly have a workflow problem, not a software problem.
- **Insight:** competitors sell operations software. The unclaimed angle is "stop paying 25–30% for customers you already earned", which is the Directoo model. That suggests leading with direct booking and commission recovery.

### 4.4 Premises waiting for the product owner's agree/disagree (not yet asked successfully)

1. The pain owners will pay to fix is losing 25–30% to aggregators on bookings they could take directly. Admin work is second-order and already targeted by existing tools.
2. "Cheap + GST" is not the USP. The USP is "your own branded website + booking app, customers pay you directly, 0% commission".
3. Demand is unverified. Before the 5–6 month build, SSPL talks properly to 8–10 named coworking owners (channels, monthly volume, aggregator fees, willingness to pay ₹999–2,499/month).
4. Replace "target all customers" with one beachhead: independent 1–3 location spaces in one city with a meaningful share of aggregator bookings. Expand after 20 pay.

## 5. How to continue (in your own Claude Code)

1. Clone the repo and open it in Claude Code:
   ```bash
   git clone https://github.com/mathsyagiri-scrimpify/Cowos.git
   ```
2. Claude Code loads `CLAUDE.md` automatically, which points it here. To be explicit, ask it to read `docs/HANDOFF.md`, `docs/original-brief.md` and `docs/requirements/00-start-here.md`.
3. Resume the strategy review: run `/office-hours` and say "resume from docs/HANDOFF.md §4: ask the four premises in §4.4, then continue to alternatives and write the design doc". After the design doc exists, run `/plan-ceo-review`, then `/plan-eng-review`.
4. Get the product owner's answers on decisions D1–D6 (`13-risks-open-questions-decisions.md`), then update the decision log status column.
5. Start the long-lead approvals as soon as D3/D4 are decided: Razorpay Partner application, TRAI DLT registration, WhatsApp templates, SES production access, buying domains.

### Suggested first assignment: 8–10 owner interviews

For each coworking space, record:
- Name, city, locations, seats
- How bookings come in today (WhatsApp, phone, walk-in, aggregators, website) and the rough monthly split
- Aggregator fees paid per month
- Hours per week spent on booking admin and invoices
- Whether they have a website/domain, and who built it
- What they'd pay per month for "own website + booking + payments, 0% commission"
- Would they pilot this quarter? (yes/no and why)

Add the results to a new `docs/research/owner-interviews.md` before changing the build scope.

## 6. Tooling notes

- gstack is installed per machine (`~/.claude/skills/gstack`). Your telemetry, learnings and decision logs live in your own `~/.gstack/`, and none of the previous session's local preferences carry over. Expect the first-run prompts again.
- The repo also contains a vendored copy at `.claude/skills/gstack/`, which gstack marks as deprecated. Decide whether to keep it, or move to team mode (`gstack-team-init`) and stop vendoring.
- On the previous Windows machine, `gstack-skill-start` took over 2 minutes. If a skill preamble times out, rerun it with a longer timeout.
- A gstack upgrade (v1.91.93 → v1.91.332) was available but not applied (blocked by permission settings). Run `/gstack-upgrade` if you want it.
