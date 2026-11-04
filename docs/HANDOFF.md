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
| Approved design doc + all review results (office-hours, eng x3, design, CEO) | `docs/designs/cowos-platform-v1.md`; deferred work in `TODOS.md` |
| This handoff note | `docs/HANDOFF.md` (committed together with `docs/original-brief.md` and `CLAUDE.md` in the handoff commit) |
| Context file Claude Code loads automatically | `CLAUDE.md` at the repo root (points here) |

**Check before starting:** `git log --oneline -3` should show a handoff commit on top of `c030982`. If `docs/HANDOFF.md` is missing from your clone, the handoff commit hasn't been pushed yet; ask the previous owner to push `main`.

Nothing else is needed from the previous machine. The `.claude/worktrees/...` folder there was a scratch copy and is not part of the repo.

**No secrets or credentials are in the repo, and none have been created yet** (no AWS, Razorpay, SMS, WhatsApp or domain accounts). Use your own accounts and keep keys out of git.

## 2. What is done

1. **Requirements pack v0.1** covering all 13 requested deliverables: vision, personas and permissions, BRD, SRS (~200 numbered FRs with MoSCoW), user stories (Given/When/Then), flows (Mermaid), NFRs, architecture, data model, API modules, integrations, release plan, risks and decisions.
2. **12 recommended changes to the original brief** are in `00-start-here.md` §2. The biggest:
   - flat plans instead of "server cost only" pricing (superseded: D1 is metered slab pricing, see §3)
   - shared multi-tenant platform instead of one EC2 per tenant
   - pre-rendered tenant websites
   - each tenant's own Razorpay account instead of Razorpay Route
   - `cowos.site` for tenant sites, separate from `cowos.in`
   - "Powered by Cowos" badge
   - three booking modes (instant, request, enquire/tour)
3. **Product owner clarifications already applied:**
   - **0% commission means Cowos takes no share of the tenant's booking revenue.** What the tenant charges its own customers (prices, promo codes, discounts, optional convenience fee, deposits) is the tenant's decision. See `00` §C1, BR-14, BR-25.
   - **Tenants create their own promo codes** in the Tenant Admin, as an MVP must (every tenant gets every feature; there are no plans). See FR-TA-150/156/157/159, FR-BA-016, US-4.5, BR-26.
   - Added an optional tenant convenience fee (FR-TA-071, off by default). **Not explicitly approved yet:** confirm with the product owner or remove it.

## 3. What is NOT decided yet

**D1 (pricing), revised on 2026-11-04:** SSPL charges each tenant only for metered consumption: server usage (request units and storage units) and notifications (OTP SMS, other SMS, WhatsApp, email), each priced in slabs SSPL configures in a Super Admin rate card, applied like an electricity bill (each slab at its own rate). No subscription, plan fee or commission. The free pilot programme (first 20 owners free for a year) is removed, so billing, metering, self-serve sign-up and the wizard are launch scope. **D2 (hosting), decided 2026-11-04:** one app server plus managed PostgreSQL, with tenant websites on S3 + CloudFront. See the design doc's Revision note and Pricing and Billing Rules. The pack was updated to v0.3 to match. The 2026-11-02 version (share of the real AWS cost, messaging at cost) and the pilot rules are kept in the review sections as history.

The other decisions in `docs/requirements/13-risks-open-questions-decisions.md` §3 (D2–D16) are still **Proposed**. The five that block engineering: D2 hosting model, D3 payment model, D4 domains, D5 badge text, D6 database tenancy strategy.

Open questions OQ1–OQ15 in the same file are also unanswered. The pack uses the default assumptions listed there.

## 4. Work in progress: strategy review (stopped mid-way)

The previous session started `/plan-ceo-review` (gstack). Since there was no design doc, it ran `/office-hours` inline first, in **startup mode** (goal: intrapreneurship at SSPL). It stopped at **Phase 3 (premise challenge)** when the work was handed over.

Exact position:

| Step | Status |
|------|--------|
| `/office-hours` Phase 1 (context), goal, stage | Done (see 4.1) |
| Phase 2A forcing questions (Q2 status quo, Q4 wedge; Q5 skipped as nothing to observe yet) | Done (see 4.1) |
| Phase 2.75 landscape search | Done (see 4.3) |
| Phase 3 premises | Done on 2026-11-02 (outcomes in 4.5) |
| Phase 3.5 second opinion | Skipped by the product owner |
| Phase 4 alternatives | Done. Owner chose **Approach A: full MVP as specified** |
| Design doc | **Approved 2026-11-02:** `docs/designs/cowos-platform-v1.md`. Spec review ran 3 rounds (quality 5 → 7 → 6). The last 9 blocking fixes are applied but no reviewer re-checked them, and 20 minor concerns are recorded in the doc |
| `/plan-eng-review` | Done 2026-11-03 (first review: 39 findings → tasks T1-T10) |
| `/plan-design-review` | Done 2026-11-03 (score 1 → 6/10, 33 decisions; **6 design gaps still open**) |
| `/plan-eng-review` re-review after design | Done 2026-11-03 (26 findings → RT1-RT9) |
| `/plan-ceo-review` (SCOPE REDUCTION) | Done 2026-11-03: billing automation, self-serve sign-up and the wizard move to a **Billing release at launch + 4 months**; 2 themes at launch, themes 3-4 six weeks later; added suspended page, render race guard, hardened demo form, pilot scorecard, cost-recovery goal (CT1-CT9) |
| `/plan-eng-review` re-review 2 after CEO | Done 2026-11-03: one PilotProgramService, single EdgeSyncWorker for CloudFront KVS, data_seq ordering, pre-handover setup grant, scorecard month rule (ET1-ET7) |
| Requirements pack update (T9, DT8, CT1, ET7) | Done 2026-11-03: pack v0.2 |
| Pricing revision by the product owner | 2026-11-04: pilot programme removed; metered slab pricing for server and notifications; one app server. Design doc revised, pack v0.3. Billing release cancelled. **Not yet reviewed by /plan-eng-review** |
| `/plan-ceo-review` 2 (SCOPE REDUCTION) | Done 2026-11-04: cost view kept at launch; a request unit counts page views and API calls only (not every file); invoices wait until every day is counted; owners get a usage alert (default 50% above last month). Tasks C2T1-C2T6 |
| `/plan-eng-review` re-review 3 | Done 2026-11-04: one SlabCalculator for bill, estimate and wallet; request units read only from CloudFront logs (page views + booking-app API calls on tenant hosts; admin traffic not billed); only successful responses billed; wallet credit for undelivered messages the provider didn't charge. Tasks E3T1-E3T6 |
| `/plan-design-review` 2 (billing UI) | Done 2026-11-04: billing page leads with a status line; pricing page leads with sample bills; full billing/wallet states; wallet runway alert; GST added on top of wallet top-ups (CA to confirm); rate card drafts, impact preview and typed publish; cost view leads with margin; phone layouts; the first design review's 6 gaps settled. Tasks DR2T1-DR2T7. Score 4 → 5/10 overall; it reaches 8+ once DESIGN.md exists (Sprint 0) |
| `/plan-eng-review` re-review 4 | Done 2026-11-04: no new services for the design additions; manual refunds only after a final gateway failure (no double refunds); sign-in code emails always sent even past the wallet limit (SSPL absorbs the tiny cost). Tasks E4T1-E4T3 |

### 4.5 Outcomes on 2026-11-02 (superseding 4.4; premise 4 and 5 and the pilot rules were revised on 2026-11-04, see §3)

- **Premise 1 (revised):** cover both pains, aggregator commission and admin work.
- **Premise 2 (agreed):** the USP is own brand + direct payment + 0% commission, not "cheap + GST".
- **Premise 3 (owner's decision):** build the complete suite regardless of demand evidence. Recorded as an accepted risk.
- **Premise 4 (agreed, changed):** one beachhead city; the first 20 owners pilot **fully free for one year**.
- **Premise 5 (owner's decision):** hosting cost only (D1, see §3).
- **Pilot rules:** 10 rules covering the production gate, free-year start, billing state, messaging caps, the end of the free year and how the hosting charge is calculated and collected. See the design doc's Pilot Rules.
- **Assignment:** sign the first 5 pilot spaces and record their booking channels, volumes and aggregator fees (§5 interview fields).

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

### 4.4 Premises as first drafted (superseded by 4.5; kept for history)

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
3. The pack (v0.3) and the design doc match the 2026-11-04 pricing revision. **Before launch the product owner sets the slab boundaries and prices** for each meter in the rate card. Open questions: bot traffic in request units, what stops when a wallet hits the −₹500 floor, notice period before a rate change, availability target and deploy downtime for the single app server, BR-05 paid assisted onboarding (OQ12), the OQ3 refund default, dedicated-cell charging, a custom-domain limit without plans, the metering details in pack OQ26–OQ34 (rounding, trial usage, late logs, ₹0 invoices, GST on wallet top-ups, log retention). The CEO and eng reviews of this revision are done (CEO review 2, eng re-review 3).
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
- Expected monthly website traffic and messages (OTP, SMS, WhatsApp, email), to test slab prices
- Would they sign up at launch? (yes/no and why)

Add the results to a new `docs/research/owner-interviews.md` before changing the build scope.

## 6. Tooling notes

- gstack is installed per machine (`~/.claude/skills/gstack`). Your telemetry, learnings and decision logs live in your own `~/.gstack/`, and none of the previous session's local preferences carry over. Expect the first-run prompts again.
- The repo also contains a vendored copy at `.claude/skills/gstack/`, which gstack marks as deprecated. Decide whether to keep it, or move to team mode (`gstack-team-init`) and stop vendoring.
- On the previous Windows machine, `gstack-skill-start` took over 2 minutes. If a skill preamble times out, rerun it with a longer timeout.
- A gstack upgrade (v1.91.93 → v1.94.8) was available but not applied (blocked by permission settings). Run `/gstack-upgrade` if you want it.
- On that machine Codex, jq and an OpenAI key were not set up, so outside-voice reviews, /autoplan task JSONL and design mockups were skipped.
