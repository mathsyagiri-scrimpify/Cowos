# 05 · User Stories with Acceptance Criteria

> Cowos requirements pack · v0.2 draft · Story IDs `US-<epic>.<n>`; each story lists the FRs it covers. A story marked P1-B ships with the Billing release.

## Epic index

| Epic | Name | Phase |
|------|------|:---:|
| E1 | Tenant onboarding (pilot invite at launch, self-serve sign-up at P1-B) | P1 / P1-B |
| E2 | Site builder and preview | P1 |
| E3 | Publishing, versions and domains | P1 |
| E4 | Inventory, pricing and policies | P1 |
| E5 | Payment gateway and invoicing | P1 |
| E6 | Customer instant booking | P1 |
| E7 | Requests, enquiries and tours | P1 |
| E8 | Booking management and front desk | P1 |
| E9 | Hosting charge, dunning and suspension | P1 / P1-B |
| E10 | Super Admin and support | P1 |
| E11 | Existing-website integration | P1 / P2 |
| E12 | Privacy and data rights | P1 |
| E13 | Memberships and credits | P2 |

---

## E1 · Tenant sign-up and onboarding

**US-1.1 Sign up and start a trial** (FR-MW-005, FR-MW-006, FR-MW-012) · P1-B
As a coworking owner, I want to sign up in under 3 minutes so that I can try Cowos without a sales call.
- **Given** I am on the Cowos sign-up page with `?ref=workloft` in the URL, **when** I submit name, email, mobile, business name, city and slug `acme-hub` and verify both OTPs, **then** a tenant in state `Trial` is created with a 14-day trial end date, referral `workloft` attached, and I land on the onboarding wizard.
- **Given** the slug `acme-hub` is already taken or reserved, **when** I type it, **then** I see "Not available" within 500 ms and 3 suggested alternatives.
- **Given** I don't verify my mobile OTP, **when** I try to continue, **then** I can't proceed and can resend the OTP after 30 seconds (max 5 per hour).
- **Given** the Billing release is not live, **when** anyone opens `/signup` or calls the sign-up API, **then** it returns 404; the pilot request form (FR-MW-008) is the only intake (CEO-S1).

**US-1.2 Choose my goal** (FR-TA-001, FR-TA-006) · P1-B
As a new tenant, I want to say whether I need a full website or only a booking app so that the wizard only asks what I need.
- **Given** I'm on wizard step 1, **when** I choose "I already have a website", **then** theme and website-content steps are hidden, and an "Integrate with your website" step is added before publish.
- **Given** I chose booking-only, **when** I later go to Settings → Website, **then** I can switch to the full website at any time without losing data.

**US-1.3 Enter business details with GSTIN** (FR-TA-003)
As a tenant owner, I want my legal details filled from my GSTIN so that invoices are correct.
- **Given** I enter a GSTIN with a valid checksum, **when** I tab out of the field, **then** state and PAN are derived and shown for confirmation. From P1-B the legal name is also fetched from the GST public search provider (CEO-S2).
- **Given** I enter a GSTIN with an invalid checksum, **when** I tab out, **then** I see "This GSTIN is not valid" and cannot save it.
- **Given** I have no GSTIN, **when** I tick "I'm not GST registered", **then** I'm asked for PAN and my invoices will be issued as bills of supply without tax lines.

**US-1.4 Resume onboarding later** (FR-TA-001, FR-TA-002) · P1-B
As a busy owner, I want the wizard to save as I go so that I can finish later.
- **Given** I completed steps 1–4 and closed the browser, **when** I log in again, **then** I return to step 5 with all earlier data saved, and the dashboard checklist shows 4 of 10 steps done.

**US-1.5 Accept my pilot invite and review the setup** (FR-TA-172, FR-TA-009, FR-TA-002, FR-TA-007)
As a pilot owner, I want to accept SSPL's invite on my phone and see what was set up for me so that I can take over my space's account.
- **Given** SSPL created my pilot tenant, **when** I open the invite link within 72 hours, **then** my email is verified by the link, I verify my phone by OTP, set a password and MFA, and accept the ToS and DPA myself.
- **Given** I accepted the invite, **when** I first log in, **then** I see "Review what we set up" with every change SSPL made under the setup grant, then a checklist of my remaining steps (Razorpay, GSTIN/PAN, DNS, ToS/DPA), and SSPL's setup grant has ended.
- **Given** the invite is more than 72 hours old, or SSPL resent it, **when** I open the old link, **then** I see "This invite has expired. Ask SSPL to resend it." and it can't be used.
- **Given** my Razorpay connection was marked done, **when** it is disconnected and I open the home page, **then** the checklist item flips back to not done.

---

## E2 · Site builder and preview

**US-2.1 Pick and switch themes** (FR-TA-020, FR-TA-021)
As a tenant owner, I want to try different themes with my own content so that I pick the one that suits my brand.
- **Given** I've entered hero text and uploaded 6 gallery images, **when** I switch from theme "Loft" to theme "Metro", **then** the preview shows the same content in the new layout within 2 seconds and nothing is lost.
- **Given** theme "Metro" doesn't support video hero, **when** I switch to it with a video hero set, **then** I'm told the video will show as its poster image, and switching back restores the video.

**US-2.2 Set brand colours safely** (FR-TA-022)
As a tenant owner, I want to use my brand colours so that the site looks like my business, without making text unreadable.
- **Given** I set primary colour `#FFE600` (yellow), **when** the system derives the colour roles, **then** any role that fails contrast (4.5:1 text, 3:1 UI components) in light or dark mode has its lightness adjusted automatically, and I see a before/after of the change (23A).
- **Given** I set valid colours, **when** I open the preview, **then** buttons, links and headers use those colours on both website and booking app.

**US-2.3 Edit sections** (FR-TA-023, FR-TA-024)
As a tenant owner, I want to turn sections on and off and edit them through simple forms so that I don't need a designer.
- **Given** the Testimonials section is enabled with no testimonials, **when** I try to publish, **then** the pre-publish check suggests disabling it or adding at least one.
- **Given** I upload a gallery image without alt text, **when** I save, **then** I'm shown a suggested alt text that I must confirm or edit.
- **Given** I disable the FAQ section, **when** I preview, **then** FAQ is not in the page or the navigation.

**US-2.4 Preview on devices** (FR-TA-027, FR-TA-028)
As a tenant owner, I want to see my site on desktop and mobile before it goes live so that I'm confident it looks right.
- **Given** I'm in the builder, **when** I toggle "Mobile", **then** the preview renders at 390 px width with the mobile nav and one floating Book bar with a WhatsApp icon (3A).
- **Given** I click "Share preview", **when** my partner opens the link on their phone within 7 days, **then** they see the draft site and booking app, with a "Preview, not live" ribbon, and can't make real payments.

---

## E3 · Publishing, versions and domains

**US-3.1 First publish with approval** (FR-TA-041, FR-TA-042, FR-SA-010)
As a tenant owner, I want to publish my site so that customers can find and book me.
- **Given** all pre-publish checks pass, **when** I click Publish for the first time, **then** the site status becomes "In review · typical under 4 working hours · you can keep editing", and SSPL's queue receives the request (11A).
- **Given** I'm a pilot and the production entry gate hasn't passed, **when** I open Publish, **then** I see "Your site is ready. Publishing opens when Cowos launches" (Pilot Rules #1, 11A).
- **Given** my tenant is Suspended, **when** I click Publish, **then** it is rejected with "Your site is suspended. Contact SSPL support." and the reason (CEO-A1).
- **Given** SSPL approves, **when** the approval is recorded, **then** the site is live on `acme-hub.cowos.site` within 2 minutes and I get an email and WhatsApp with the live link.
- **Given** SSPL requests changes, **when** I open the admin, **then** I see the reviewer's notes and can republish after fixing.

**US-3.2 Republish and roll back** (FR-TA-043, FR-TA-044)
As a tenant owner, I want to republish edits and undo a bad change so that my live site is never stuck broken.
- **Given** my site is live at version 7, **when** I edit the hero and publish, **then** version 8 is live within 2 minutes, without approval, and version 7 is in history.
- **Given** version 8 has a mistake, **when** I click "Restore" on version 7, **then** version 7's content is live within 60 seconds and history shows "v9: rolled back to v7".

**US-3.3 Connect my own domain** (FR-TA-051, FR-TA-052, FR-TA-053, FR-TA-054)
As a tenant owner with a GoDaddy domain, I want step-by-step DNS instructions so that my site runs on `www.acmehub.in` with SSL.
- **Given** I enter `acmehub.in` and choose booking placement `book.acmehub.in`, **when** the system detects GoDaddy nameservers, **then** I see GoDaddy-specific steps with exact record names and values to copy, including apex forwarding to `www`.
- **Given** I've added the records, **when** DNS check runs, **then** each record shows "Found" or "Wrong value: expected X, found Y", and once all pass, SSL is issued and the domain shows "Live" within 30 minutes of verification.
- **Given** my custom domain is live, **when** someone visits `acme-hub.cowos.site`, **then** they get a 301 redirect to `https://www.acmehub.in` with the same path.
- **Given** DNS verification hasn't succeeded within 72 hours, **when** the window ends, **then** I get an email with troubleshooting tips and a "Check again" button, and the platform subdomain stays live.

---

## E4 · Inventory, pricing and policies

**US-4.1 Create a meeting room product** (FR-TA-061, FR-TA-062, FR-TA-064, FR-TA-066)
As a tenant owner, I want to add my two meeting rooms with hourly prices so that customers can book them online.
- **Given** I create product "Meeting Rooms" of type Meeting Room with units "Boardroom (10 seats)" and "Huddle (4 seats)", hourly price ₹600 and ₹300 GST exclusive, 15-minute buffer after, minimum 1 hour, **when** I save, **then** both units appear in the booking app with their own prices, and their free start times show in the start-times list (1A).
- **Given** a booking ends at 11:00 in the Boardroom, **when** a customer picks the Boardroom and a 1h duration, **then** no start time between 11:00 and 11:15 is listed, and 11:15 is the next start option.

**US-4.2 Create a hot desk day pass pool** (FR-TA-063)
As a tenant owner, I want 30 flexible seats sold as day passes so that I don't need to label each seat.
- **Given** a hot desk product with daily capacity 30, **when** 28 seats are booked for 12 Nov, **then** customers see "2 left" and can book at most 2 seats for that date.
- **Given** capacity is reached, **when** a customer selects 12 Nov, **then** the date shows "Sold out".

**US-4.3 Configure cancellation policy** (FR-TA-080, FR-TA-081)
As a tenant owner, I want to set refund tiers so that cancellations are handled without arguments.
- **Given** I set "100% if ≥24h before, 50% if ≥2h before, 0% otherwise" on Meeting Rooms, **when** a customer views checkout, **then** the policy summary shows exactly these tiers in plain language.
- **Given** a customer cancels 5 hours before a ₹1,180 booking, **when** the refund is calculated, **then** it is ₹590 and shown to the customer before they confirm.

**US-4.4 Holidays** (FR-TA-060)
As a tenant owner, I want to mark Diwali as closed so that nobody books that day.
- **Given** I add 1 Nov 2026 as a holiday for my location, **when** a customer opens the calendar, **then** 1 Nov is shown closed for all products, and existing bookings on that date are listed for me to handle.

**US-4.5 Create my own promo code** (FR-TA-150, FR-TA-156, FR-TA-157)
As a tenant owner, I want to create promo codes myself so that I can run my own offers without asking SSPL.
- **Given** I'm on Promotions in the Tenant Admin, **when** I create `MONSOON15` (15% off meeting rooms, max ₹300, valid 1–31 July, 100 uses, public), **then** it is active immediately, shown as an offer on the meeting rooms page, and customers can apply it at checkout.
- **Given** `MONSOON15` has reached 100 uses, **when** a customer applies it, **then** they see "This offer has ended", and my promo list shows 100/100 used with the total discount given.
- **Given** I'm a Manager assigned to the Madhapur location, **when** I create a code, **then** I can only apply it to Madhapur products.
- **Given** I pause a code, **when** a customer applies it, **then** it is rejected, and bookings that already used it are unaffected.

---

## E5 · Payment gateway and invoicing

**US-5.1 Connect Razorpay** (FR-TA-130, FR-TA-131, FR-TA-007)
As a tenant owner, I want to connect my Razorpay account so that payments come straight to my bank.
- **Given** I have MFA enabled, **when** I click "Connect Razorpay" and approve access on Razorpay, **then** I return to Cowos with status "Connected: Acme Hub LLP (live)" and Instant Book products can be published.
- **Given** I don't have MFA enabled, **when** I click "Connect Razorpay", **then** I'm asked to set up MFA first.
- **Given** my Razorpay KYC is pending, **when** I open Payments, **then** I see "Razorpay: waiting for approval (usually 1–5 days)", I can still publish with Book a tour, Request to Book and offline payment, and my Instant Book products show as "Request" until Razorpay is live; on activation I get a one-tap "Turn on instant booking" (33A).
- **Given** OAuth isn't available for my account, **when** I choose "Use API keys", **then** keys are validated with a test call, the secret is stored encrypted, and only the last 4 characters of the key ID are shown afterwards.

**US-5.2 GST invoice for a company customer** (FR-TA-136, FR-BA-023)
As a startup admin, I want invoices with my company GSTIN so that my company can claim input tax credit.
- **Given** I belong to company "Byteloom Pvt Ltd" with a valid GSTIN and choose "Bill to company", **when** my payment succeeds, **then** a tax invoice is generated with the tenant's GSTIN, Byteloom's GSTIN and address, SAC 997212, taxable value, CGST 9% + SGST 9% (property in the same state as the tenant registration), and a sequential number in the tenant's series for the FY.

**US-5.3 Credit note on refund** (FR-TA-137)
As a tenant owner, I want refunds after invoicing to create credit notes so that my GST filings are correct.
- **Given** invoice `WL/26-27/0042` for ₹1,180 exists, **when** a ₹590 refund is processed, **then** credit note `WL-CN/26-27/0007` for ₹590 (₹500 taxable + ₹90 GST) referencing `WL/26-27/0042` is generated, and the original invoice is unchanged.

**US-5.4 Record an offline payment** (FR-TA-134)
As a front-desk executive, I want to record a cash payment so that the booking is confirmed without online payment.
- **Given** a walk-in customer pays ₹350 cash for a day pass, **when** I create the booking and choose "Record offline payment: Cash", **then** the booking is confirmed, an invoice is generated, the payment appears with method "Cash" and my name as recorder.

---

## E6 · Customer instant booking

**US-6.1 Book a meeting room and pay by UPI** (FR-BA-012, FR-BA-013, FR-BA-015, FR-BA-017, FR-BA-018, FR-BA-019, FR-BA-026)
As a customer, I want to book the boardroom for tomorrow 10–12 and pay with UPI so that it's guaranteed.
- **Given** the Boardroom is free tomorrow 10:00–12:00, **when** I pick tomorrow in the date strip and the 2h duration chip, **then** I see a list of start times including "10:00 – 12:00 · ₹1,416", with availability as text (1A).
- **Given** I chose the 10:00 – 12:00 start time, **when** I tap Continue, **then** I see the summary: slot, total, "Pay to <tenant legal name>", "incl. GST ₹216 ▸" (tap to expand) and a one-line cancellation policy; nothing is held yet (2A).
- **Given** I verified my phone OTP on step 2, **when** step 3 starts, **then** the slot is held for 10 minutes with a visible countdown, the price is re-quoted (any difference shown before Razorpay opens), and Razorpay Checkout opens for the tenant's account (2A).
- **Given** I complete payment, **when** the app is waiting for the server, **then** I see "Confirming your payment…" for up to 2 minutes, never an error, and then a confirmation with my desk pass and QR; I receive email + WhatsApp confirmation within 1 minute with an invoice PDF (or "Invoice being prepared") and add-to-calendar link (9A).
- **Given** someone else took the slot while I was verifying my OTP, **when** the hold is attempted, **then** I'm told the time was just booked and the next free times show inline. A double booking is never created.
- **Given** my hold expired and the slot is still free, **when** I return to checkout, **then** I can retry with one tap (9A).
- **Given** my hold expired and my payment succeeds after the slot was taken by someone else, **when** the late payment is detected, **then** I see "Full refund of ₹1,416 started, 5–7 working days" plus the next free times (9A).
- **Given** I come back from my UPI app or reload, **when** `/book/checkout/{bookingRef}` opens in my session, **then** I land on the booking's current state; without my session I'm asked for OTP first, and another customer's session gets 404 (R16).

**US-6.2 Log in at checkout with phone OTP** (FR-BA-001, FR-BA-002, FR-BA-003)
As a first-time customer, I want to browse without an account and only verify my phone at checkout so that booking is quick.
- **Given** I'm not logged in and I'm on the checkout summary, **when** I tap Continue, **then** I'm asked for my mobile number, receive an OTP by SMS (or WhatsApp if I choose), and after verifying, the hold starts and I continue to payment without losing my selection (2A).
- **Given** I'm a new customer, **when** I verify my OTP, **then** I see the tenant's privacy notice and an unticked optional marketing consent checkbox, and my consent choice is stored with the notice version.

**US-6.3 Apply a coupon** (FR-BA-016, FR-TA-150)
As a customer, I want to apply a coupon so that I get the advertised discount.
- **Given** coupon `FIRST20` gives 20% off up to ₹200 on first bookings, **when** I apply it to my first ₹1,200 booking, **then** the discount shows as ₹200 and GST is calculated on the discounted taxable value.
- **Given** I already have a past booking, **when** I apply `FIRST20`, **then** I see "This coupon is valid on your first booking only".

---

## E7 · Requests, enquiries and tours

**US-7.1 Request an event space** (FR-BA-020, FR-TA-093)
As a customer, I want to request the event hall for a Saturday workshop so that the space can confirm my setup.
- **Given** the event hall uses Request to Book with a 24h SLA, **when** I submit a request for 60 people with notes, **then** I see "Awaiting confirmation" and the tenant gets a staff notification.
- **Given** the tenant approves, **when** approval is saved, **then** I get a payment link valid for 24 hours, and the slot is reserved for me until the link expires.
- **Given** the tenant doesn't respond in 24 hours, **when** the SLA expires, **then** the request is auto-declined and I'm notified with the tenant's phone number.

**US-7.2 Book a tour for a private cabin** (FR-BA-021, FR-TA-069, FR-TA-112)
As a startup founder, I want to schedule a visit to see a 6-seat cabin so that I can decide before committing.
- **Given** the tenant has tour slots Mon–Sat 11:00–17:00, **when** I pick Thursday 15:00, team size 6, move-in 1 Dec, **then** a lead is created at stage "Tour scheduled", I get a confirmation with directions, and the tenant's assigned staff gets a WhatsApp notification.

**US-7.3 Work the lead pipeline** (FR-TA-112, FR-TA-113)
As a tenant owner, I want to move leads through stages so that I don't lose deals.
- **Given** a lead at "Tour done", **when** I move it to "Won" and click "Create booking", **then** a customer record is created (or matched by phone) and a monthly booking form opens prefilled with the cabin and start date.
- **Given** a lead has a follow-up date of today, **when** I open the dashboard, **then** it appears under "Follow-ups due".

---

## E8 · Booking management and front desk

**US-8.1 Customer cancels within policy** (FR-BA-031, FR-TA-094, FR-PL-050)
As a customer, I want to cancel a booking and see my refund before confirming so that there are no surprises.
- **Given** a confirmed booking 30 hours away with 100% refund tier, **when** I tap Cancel, **then** I see "You'll get ₹1,416 back to your original payment method in 5–7 working days", and after confirming, the booking is `Cancelled`, a refund is created on the tenant's Razorpay account, and a credit note is issued.
- **Given** the refund fails at the gateway, **when** the failure webhook arrives, **then** the tenant sees a "Refund failed" alert with a retry action, and I'm told the tenant will contact me.

**US-8.2 Front desk check-in** (FR-TA-091, FR-TA-096)
As a front-desk executive, I want to scan a customer's QR code so that check-in takes seconds.
- **Given** a day-pass booking for today, **when** I scan its QR code with the admin app on my phone, **then** the booking shows "Checked in at 09:42 by Sneha".
- **Given** I scan a QR for a booking at another tenant or another date, **when** the scan completes, **then** I see "Not valid here/today" without revealing the other booking's details.

**US-8.3 Owner overrides a refund** (FR-TA-094)
As a tenant owner, I want to give a full refund outside the policy so that I can keep a good customer happy.
- **Given** the policy gives 0% refund, **when** I choose "Override" and enter ₹1,416 with reason "Power outage", **then** the refund is processed, the audit log records my name, the policy amount, the override amount and the reason.

**US-8.4 Reschedule** (FR-BA-032, FR-TA-095)
As a customer, I want to move my meeting from Tuesday to Wednesday so that I don't lose my money.
- **Given** I'm before the free-cancellation cutoff and Wednesday 10–12 is free, **when** I reschedule, **then** the booking moves, no extra payment is required for the same price, and both of us get updated notifications with a new `.ics`.

---

## E9 · Platform billing, dunning and suspension

**US-9.1 Add a payment method to go Active** (FR-TA-176, FR-PL-062, FR-PL-063) · P1-B
As a trial tenant, I want to add a payment method so that I can use my own domain and get indexed.
- **Given** I'm in Trial, **when** I add a custom domain, **then** I'm told custom domains need an Active tenant and shown "Add a payment method"; there is no plan picker (28A).
- **Given** I'm in Trial, **when** I add a payment method (Razorpay payment link details, or UPI Autopay / e-mandate with a ₹5,000 maximum), **then** my tenant becomes `Active` straight away, the domain step unlocks and my site is no longer `noindex`; my first hosting invoice arrives on the 10th of the next month for the days I was counted (Pilot Rules #7, #10).

**US-9.2 Transparent billing page** (FR-TA-175, FR-PL-060, FR-PL-064, FR-MW-003) · P1-B
As a tenant owner, I want to see exactly what I'm paying for so that I trust Cowos.
- **Given** I open Billing, **when** the page loads, **then** I see my next invoice (issue date 10th, last month's amount, due date, any adjustment line), "0% commission: ₹0 on ₹3,42,000 of bookings", my messaging wallet balance and usage with unit prices, and the AWS breakdown by service category with the tenant count, but never another tenant's usage (5A).
- **Given** it's mid-month, **when** I open Billing, **then** I see "This month so far: about ₹<low>–<high> · estimate, final on the 10th" from the stored daily estimate (15A, R15).
- **Given** AWS hasn't finalized last month's bill on the 10th, **when** I open Billing, **then** I see "Waiting for AWS to finalize <month>'s bill" with the expected date, and no invoice is issued until it is final (12A, R1).
- **Given** I'm a pilot in my free year, **when** I open Billing, **then** I see "Free until <date> · SSPL covered ₹X this month" and no invoice (5A, Pilot Rules #4).

**US-9.3 Restricted after non-payment** (FR-PL-062, FR-PL-012) · P1-B
As SSPL, I want to restrict, not break, a non-paying tenant so that their customers aren't harmed.
- **Given** a tenant's hosting invoice is 8 days past due, **when** the daily billing job runs, **then** the tenant moves to `Restricted`: the website stays live with every Book CTA switched to "Call / WhatsApp to book" within about 2 minutes and no billing mention to visitors, the booking app home shows the same contact options, existing customers can still view, cancel and download invoices, and the Owner gets email + WhatsApp (10A, R11).
- **Given** the tenant pays, **when** payment is confirmed, **then** state returns to `Active` within 1 minute, online booking resumes, and Book CTAs are restored.

**US-9.4 Suspension and data retention** (FR-PL-062, FR-PL-071, FR-SA-052) · P1-B
As a tenant owner who stopped paying, I want my data kept for a while so that I can come back or take it with me.
- **Given** I'm 22 days past due, **when** the billing job runs, **then** my website shows a page in my theme with my name, address, phone, WhatsApp and "Online booking is temporarily unavailable", and I can still log in to pay or export (10A).
- **Given** I've been suspended for 90 days, **when** the retention job runs after 3 reminder emails, **then** personal data and content are deleted, tax invoices are archived per retention rules, and SSPL keeps an audit record of the deletion.

**US-9.5 End of my pilot year** (FR-PL-062, FR-TA-175, FR-TA-176) · P1-B
As a pilot owner, I want clear notice before my free year ends so that I can decide to keep going without surprises.
- **Given** my free period ends on 30 Nov, **when** 60, 30 and 7 days are left, **then** I get a notice each time with the expected first charge, and with the 60-day notice I'm asked to add a payment method (Pilot Rules #6, 12A).
- **Given** my free year ended, **when** the 10th of month 14 arrives, **then** I receive my first hosting invoice, for month 13 (Pilot Rules #6, R1).
- **Given** I haven't paid that first invoice by its due date plus 14 days, **when** the billing job runs, **then** my tenant becomes `Restricted` and stays Restricted for 90 days before the normal Suspended lifecycle; my website stays live and existing bookings, invoices and export keep working (Pilot Rules #6).

---

## E10 · Super Admin and support

**US-10.1 Review first publish** (FR-SA-010)
As an SSPL reviewer, I want automated checks and a preview in one place so that I can approve safely in minutes.
- **Given** a tenant submits a first publish, **when** I open the queue item, **then** I see GSTIN/PAN verification, phone/email verification, gateway connection, content scan flags, a preview link, and the SLA time left.
- **Given** the GSTIN is cancelled per the GST portal, **when** I review, **then** the check shows red and I can reject with a reason the tenant will see.

**US-10.2 Support access with consent** (FR-SA-030, FR-TA-177)
As a support executive, I want temporary access to a tenant's admin so that I can fix their configuration.
- **Given** a ticket from Workloft, **when** I request read-only access for 4 hours, **then** Workloft's Owner gets an approval request in-app and by email.
- **Given** the Owner approves, **when** I open their admin, **then** a banner reads "Viewing as SSPL Support (Divya), expires 15:40", customer phone numbers are masked unless write access was granted, and every page view is in both audit logs.
- **Given** the grant expires, **when** I click anything, **then** I'm returned to the Super Admin portal.

**US-10.3 Suspend a fraudulent tenant** (FR-SA-011, FR-PL-010, FR-PL-012, FR-TW-012)
As a Super Admin, I want to take down a site that is collecting money fraudulently so that customers are protected.
- **Given** a verified fraud report, **when** I suspend the tenant with reason "Fraud", **then** within about 2 minutes the website shows the suspended page in the tenant's theme ("Online booking is temporarily unavailable" with the tenant's contact details), new payments can't start, the tenant is notified, and the action is logged with my identity (10A, CEO-A1).
- **Given** the owner's publish was rendering when I suspended, **when** that render finishes, **then** it is discarded and the suspended page stays live; the owner's next publish attempt is rejected while Suspended (CEO-A1).
- **Given** the issue is resolved, **when** I reinstate the tenant, **then** the normal site is restored within about 2 minutes.
- **Given** it is before the Billing release, **when** anyone tries to move a tenant into Trial, PastDue or Restricted, **then** the API returns 409 (Q1).

**US-10.4 Create and set up a pilot tenant** (FR-SA-055, FR-SA-056, FR-SA-057, FR-TA-172)
As SSPL's pilot onboarding owner, I want to create a pilot's tenant and set it up before the owner joins so that the owner starts from a working site.
- **Given** a pilot request in "Pilot requests", **when** I open Pilots → Create pilot and submit business name, city and slug `workloft`, **then** a tenant is created in `Active` with the pilot flag, the action is audited, and the Owner invite is sent (single-use, 72 hours).
- **Given** I double-click Create, **when** both submits arrive, **then** only one tenant is created; **given** `workloft` is taken, **then** I see "Slug taken".
- **Given** the tenant has no Owner, is unpublished and has no live payments, **when** I add inventory, theme and content under the pre-handover setup grant, **then** each write is allowed and audited.
- **Given** the Owner has accepted the invite, **when** I try another write, **then** it is rejected with "Ask the owner for a support grant", and I need an Owner-approved grant (FR-SA-030) (R19).
- **Given** I resend the invite, **when** the owner opens the old link, **then** it no longer works.

**US-10.5 Triage pilot requests** (FR-MW-008, FR-SA-057)
As SSPL sales, I want every pilot request saved where I can see it so that no owner is lost.
- **Given** an owner submits the Book a demo form, **when** the email to sales fails, **then** the request is still in "Pilot requests" with an "Email to sales failed" badge (CEO-A2).
- **Given** a bot fills the hidden honeypot field, or the same IP sends a 6th request within an hour, **when** it submits, **then** it is rejected and nothing is saved.
- **Given** a message contains `<script>`, **when** I open the request, **then** it shows as plain text.

**US-10.6 Track pilot success** (FR-SA-058)
As SSPL, I want each pilot's paid online bookings per month so that I know whether the expansion gate is met.
- **Given** a pilot had 12 customer-made online bookings paid in March, of which 1 was later fully refunded, plus 2 staff-created bookings, **when** I open the scorecard, **then** March shows 11 for that pilot.
- **Given** a payment captured at 00:01 IST on 1 April, **when** the scorecard counts it, **then** it counts in April, not March (R20).
- **Given** 10 pilots have had 10 or more paid online bookings in each of the last 3 months, **when** I open the roll-up, **then** "Expansion gate met" shows yes, and I can export the scorecard as CSV (CEO-A4).

---

## E11 · Existing-website integration

**US-11.1 Booking app on my subdomain** (FR-EM-001, FR-EM-002)
As a tenant with a WordPress site, I want `book.acmehub.in` to open my booking app so that customers book without leaving my brand.
- **Given** I chose booking-only and added CNAME `book` as instructed, **when** verification passes, **then** `https://book.acmehub.in` serves my branded booking app with SSL, and the integration page shows copy-paste "Book now" button code and per-product links.

**US-11.2 Floating booking widget** (FR-EM-004) · Phase 2
As a tenant, I want a "Book a space" button on every page of my existing site so that booking is one click away.
- **Given** I paste the widget script into my site footer, **when** a visitor clicks the button on desktop, **then** the booking flow opens in a modal; **when** they proceed to login or payment, **then** it continues in a full browser tab so OTP and UPI work.
- **Given** my site's domain is not in my allowed-origins list, **when** the widget loads there, **then** it doesn't render and logs a console warning.

---

## E12 · Privacy and data rights

**US-12.1 Customer data request** (FR-BA-005, FR-TA-116)
As a customer, I want to download or delete my data so that I control my information.
- **Given** I request deletion from my profile, **when** the request is submitted, **then** the tenant sees it in the DPDP queue with a due date, and I get an acknowledgement.
- **Given** the tenant executes the deletion, **when** it completes, **then** my profile is anonymised, I can no longer log in, my invoices are retained in anonymised-access form only for statutory periods, and I'm notified.

**US-12.2 Tenant exports everything** (FR-PL-071, BR-22)
As a tenant owner, I want to export all my data so that I'm never locked in.
- **Given** I click "Export all data", **when** the job finishes, **then** I get an email with a download link (valid 24 hours, requires login) containing CSV/JSON for customers, companies, bookings, payments, invoices, leads, content, and a media zip.

---

## E13 · Memberships and credits (Phase 2)

**US-13.1 Sell a membership with meeting-room credits** (FR-TA-151, FR-TA-152, FR-TA-153)
As a tenant owner, I want a "Dedicated Desk ₹7,000/month incl. 5 meeting room hours" plan so that members get value and I get predictable revenue.
- **Given** a member on this plan, **when** a new billing cycle starts, **then** 5 hours of credits are granted, last cycle's unused credits expire (rollover disabled), and a renewal invoice was sent 5 days earlier.

**US-13.2 Pay with credits** (FR-BA-022)
As a member, I want to use my included hours for meeting rooms so that I don't pay twice.
- **Given** I have 3 credit hours, **when** I book 4 hours, **then** 3 hours are paid by credits and I pay for 1 hour online, and the invoice shows the credit line at ₹0 with the paid hour taxed normally.
