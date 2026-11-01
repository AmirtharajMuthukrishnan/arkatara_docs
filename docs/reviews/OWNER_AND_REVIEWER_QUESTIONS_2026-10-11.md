# Owner and reviewer questions: combined decision worksheet

Date: 2026-10-11 (Asia/Calcutta). Status: DRAFT; ALL ANSWERS PENDING.

This worksheet combines the previous 18 owner questions with 25 additional owner questions derived from the [validated finance findings](FINANCE_REVIEW_ASSESSMENT_2026-10-11.md). It retains report CA question IDs 1-27 with more precise wording and adds CA Q28 on revenue recognition. The [original report](ARKA_TARA_FINANCE_REVIEW_2026-10-10.md) remains verbatim; its 12 OD decisions are mapped below. Legal L1-L6 identify professional dependencies.

The owner's sequence is binding: analyze, answer/finalize the flow, then implement authorized fixes. No answer is inferred from a recommendation, an unfilled field, the word “do,” Claude's report or elapsed time.

## How to answer

- Owner questions are BUSINESS DECISION items; Q18/Q19/Q43 require factual confirmation rather than a preferred answer.
- ★ marks Codex's proposed recommendation for Arka Tara. It is not a statistically “most opted” claim. No representative Arka Tara-specific customer preference or investor requirement has been established.
- Numbers are proposed configurable settings, not law, provider guarantees or current approved defaults.
- Reply with Q1=A, Q2=A and so on; give requested values/conditions. You may answer in batches, choose a custom option or mark PENDING.
- CA REVIEW and LEGAL REVIEW are answered from applicable qualified advice, not owner preference. Written advice should identify reviewer, scope, effective date, source/conditions and unresolved issues.
- Financing readiness is evaluated through traceable stock/money, sound books, supported claims and reproducible controls. No option guarantees funding or perfect compliance.
- Q1-Q18 retain their previous numbers and option meanings. Choosing an account design for later does not activate accounts at initial launch.
- Existing safeguards remain binding: backend authority, no free agent overrides, verified payment, separate staff/customer, immutable history and secure scoped access. Open gates are not silently removed.

Research informs the recommendations but does not supply unknown business economics. Guest-checkout studies are general/primarily US evidence; competitor jewellery policies are examples, not conversion forecasts or mandatory templates. Rupee amounts, margins, supplier costs and actual registration facts must come from the owner/CA.

## Original owner questions 1-18 and additional owner questions 19-43

### Q1. Should customer accounts be available at the initial launch?

Classification: BUSINESS DECISION. Status: PENDING. References: D-23/R-37; original Q1.

- **A** — ★ Guest checkout first; optional accounts in a later release.
- **B** — Guest checkout and optional accounts together at launch.

Recommendation rationale: Preserves D-23/R-37 and reduces simultaneous login/recovery/support work. Guest checkout remains permanent under either choice. [Baymard guest-checkout research](https://baymard.com/research-articles/make-guest-checkout-prominent)

**Owner answer:** __________

### Q2. When accounts become available, when should we encourage a guest to register?

Classification: BUSINESS DECISION. Status: PENDING. References: Original Q2.

- **A** — ★ After booking confirmation, with a separate signup entry also available.
- **B** — During checkout, while keeping guest checkout equally clear and optional.
- **C** — Only through a separate signup entry; no booking-related invitation.

Recommendation rationale: Keeps registration from interrupting booking. Benefits concern future account-owned bookings unless Q3 explicitly changes the ownership policy. [Baymard signup-timing research](https://baymard.com/research-articles/delayed-account-creation)

**Owner answer:** __________

### Q3. What should “claim a guest booking” mean?

Classification: BUSINESS DECISION. Status: PENDING. References: D-24/R-34; original Q3.

- **A** — ★ Accessing guest tracking only. The booking remains permanently guest-owned and never enters account history.
- **B** — Permit an explicit transfer of one guest booking into an account, using its existing booking credential and additional verification.

Recommendation rationale: Preserves D-24/R-34. B explicitly reopens both the permanent no-claim requirement and the proposed exclusion of guest bookings from My Orders. A matching phone or merely using the same browser never proves ownership.

**Owner answer:** __________

### Q4. How should a browser display multiple guest bookings?

Classification: BUSINESS DECISION. Status: PENDING. References: D-27/R-39; original Q4.

- **A** — ★ Show “Bookings available on this browser,” containing only bookings for which that browser holds valid access grants.
- **B** — Provide separate booking links without a combined list.

Recommendation rationale: Makes repeat guest use practical without creating an account. Each booking still has separately scoped permissions; no phone-based history lookup.

**Owner answer:** __________

### Q5. What information should we collect before verification succeeds?

Classification: BUSINESS DECISION; relevant LEGAL REVIEW also required. Status: PENDING. References: Original Q5; LR-04.

- **A** — ★ Collect only the contact needed for verification first; collect name and full delivery address afterward.
- **B** — Collect the full form first, keeping it in a restricted, short-lived server-side store until verification succeeds.

Recommendation rationale: Minimizes information from abandoned attempts. Temporary storage is still personal-data processing. Verification intent/challenge and abuse evidence must be permitted before successful verification; usable lifetime and legal retention are different. [Official DPDP Rules](https://www.meity.gov.in/static/uploads/2025/11/53450e6e5dc0bfa85ebd78686cadad39.pdf)

**Owner answer:** __________

### Q6. If someone explicitly signs up and verifies their email, but later abandons booking, should their account remain?

Classification: BUSINESS DECISION. Status: PENDING. References: Original Q6.

- **A** — ★ Yes. Deliberate, verified signup creates an account independently of booking completion.
- **B** — No. For a new customer, create the account only after their first booking is confirmed.

Recommendation rationale: Distinguishes an intentional signup from merely entering checkout. Unverified email never creates an account. B couples new account creation to booking/payment completion.

**Owner answer:** __________

### Q7. Should verified but unsubmitted checkout details be resumable?

Classification: BUSINESS DECISION; relevant LEGAL REVIEW also required. Status: PENDING. References: Original Q7; LR-04.

- **A** — ★ Allow resumption for up to 24 hours.
- **B** — Do not retain a resumable draft after the customer leaves.
- **C** — Allow resumption for up to 72 hours.

Recommendation rationale: A proposed bounded convenience window. Fresh proof is required as applicable when proceeding. This is not a stock hold, payment-in-progress expiry or statutory deletion deadline.

**Owner answer:** __________

### Q8. What should be the primary login method when accounts launch?

Classification: BUSINESS DECISION. Status: PENDING. References: Original Q8.

- **A** — ★ Email one-time code initially; keep the structure ready for passkeys and Google sign-in later.
- **B** — Passkeys from the first account release, with a separately designed recovery method.
- **C** — Google sign-in plus email-code fallback from the first account release.

Recommendation rationale: Simplifies the first optional account release, while accepting mailbox dependency. Email code login must not be called phishing-resistant, independent MFA or NIST-assured authentication. [NIST authentication guidance](https://pages.nist.gov/800-63-4/sp800-63b.html), [NN/g passwordless usability](https://www.nngroup.com/articles/passwordless-accounts/)

**Owner answer:** __________

### Q9. Which initial email-code operating profile should we use?

Classification: BUSINESS DECISION. Status: PENDING. References: Original Q9; BD-15.

- **A** — ★ Six digits, 10-minute validity, five attempts per challenge, and at least 60 seconds between sends.
- **B** — Six digits, five-minute validity, three attempts per challenge, and at least 90 seconds between sends.

Recommendation rationale: Proposed configurable settings, not mandatory standards. Additional rolling destination/account/browser/network budgets must prevent guessing and message floods; resend cannot reset failures. [OWASP authentication guidance](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

**Owner answer:** __________

### Q10. How long should an ordinary customer login session remain valid?

Classification: BUSINESS DECISION. Status: PENDING. References: Original Q10.

- **A** — ★ Log out after 60 minutes of inactivity, with a maximum session age of 24 hours.
- **B** — Log out after 30 minutes of inactivity, with a maximum session age of 12 hours.
- **C** — Log out after 60 minutes of inactivity, with a maximum session age of seven days.

Recommendation rationale: Proposed customer settings selected for bounded access and interrupted browsing. Server enforcement is required; staff need a separate policy. [OWASP session guidance](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)

**Owner answer:** __________

### Q11. When should trusted-browser phone login become available?

Classification: BUSINESS DECISION. Status: PENDING. References: Original Q11.

- **A** — ★ Add it after the first email-based account release has been validated.
- **B** — Include it in the first account release.

Recommendation rationale: Reduces simultaneous authentication/recovery paths. When enabled, account-bound browser evidence plus OTP to the previously bound phone is required; neither alone unlocks history.

**Owner answer:** __________

### Q12. If trusted-browser phone login is enabled, how long should browser recognition last?

Classification: BUSINESS DECISION. Status: PENDING. References: Original Q12.

- **A** — ★ Explicit opt-in; expire after 30 days, or after 14 days without use, whichever comes first.
- **B** — Explicit opt-in; expire after 90 days, or after 30 days without use, whichever comes first.

Recommendation rationale: Conservative proposed exposure window; B favors convenience. Trust never extends an expired login session. Ordinary use does not silently renew the absolute deadline.

**Owner answer:** __________

### Q13. What should “fresh email verification” mean for sensitive actions?

Classification: BUSINESS DECISION. Status: PENDING. References: Original Q13.

- **A** — ★ Require a new email code for each sensitive action; its authorization is single-use and expires after five minutes.
- **B** — Let one email verification authorize sensitive actions within a five-minute window.

Recommendation rationale: Action-specific authorization is appropriate for infrequent changes/exports. It proves current mailbox access, not a second independent factor. Ordinary logout must remain possible.

**Owner answer:** __________

### Q14. How should customers change their email or phone number?

Classification: BUSINESS DECISION. Status: PENDING. References: Original Q14; R-35.

- **A** — ★ Provide self-service changes with verification of the current email and the replacement contact.
- **B** — Initially require support assistance, with the same verification requirements.

Recommendation rationale: Reduces routine support work while preserving current-account proof and newly verified contact. Completed changes revoke affected sessions/trust and notify established channels; old booking/invoice facts remain historical. [OWASP email-change guidance](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

**Owner answer:** __________

### Q15. What happens when a customer loses access to their email inbox?

Classification: BUSINESS DECISION; relevant LEGAL REVIEW also required. Status: PENDING. References: Original Q15; LR-04.

- **A** — ★ Offer saved, single-use recovery codes. Without valid recovery evidence, support may assist with transactions but cannot automatically restore account access.
- **B** — Do not introduce recovery codes initially; customers must regain access through their email provider before recovering the account.

Recommendation rationale: Provides pre-established recovery evidence without making phone, address or order knowledge sufficient for account takeover. Any manual restoration exception needs its own reviewed proof process. [OWASP recovery guidance](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)

**Owner answer:** __________

### Q16. What should ordinary logout do to trusted-browser recognition?

Classification: BUSINESS DECISION. Status: PENDING. References: Original Q16.

- **A** — ★ End the session and remove customer-account trust from that browser.
- **B** — End the session but retain explicitly enabled browser trust; offer a separate “Forget this browser” action.

Recommendation rationale: Gives logout a clear meaning on shared browsers. Either way, logout everywhere revokes all account sessions/trust, and ordinary logout does not require a new email verification.

**Owner answer:** __________

### Q17. Which browser-facing deployment arrangement should we target?

Classification: BUSINESS DECISION. Status: PENDING. References: Original Q17; D-23.

- **A** — ★ Customer website at arkatara.in, with customer API requests under arkatara.in/api/v1/.
- **B** — Customer website at arkatara.in, with the API at api.arkatara.in.

Recommendation rationale: A single origin simplifies browser credentials and cross-origin configuration. Next.js and Django may remain separate services. Required cookie/CSRF protection remains. Requested staff host stays staff.arkatara.com pending ownership verification. [MDN cookie scope/protection](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie)

**Owner answer:** __________

### Q18. Which domain ownership statement is currently true?

Classification: BUSINESS DECISION. Status: PENDING. References: Original Q18; D-23.

- **A** — I control both arkatara.in and arkatara.com, including DNS access.
- **B** — I control arkatara.in only.
- **C** — Ownership or DNS access still needs verification.

Factual confirmation: Answer the actual fact; no recommended factual answer. Needed before requested staff host deployment.

**Owner answer:** __________

### Q19. What is the current legal seller and business-account readiness status?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-6; BD-11; FIN-03/04/12; CA-03/LR-02.

- **A** — An entity already exists and its applicable registrations/business bank/gateway accounts are ready.
- **B** — An entity exists, but registrations or business accounts are still pending.
- **C** — The seller entity has not yet been selected or formed.

Factual confirmation: Supply entity type/name and readiness status, not bank numbers, credentials or personal PAN in this questionnaire. For intended external equity, discuss a private limited company with CA/legal advisers; cost, current structure and transition must be considered. No incorporation is authorized by answering. [Startup India business structures](https://www.startupindia.gov.in/content/sih/en/startup-scheme/international/go-to-market-guide/types-of-businesses.html)

Additional facts: entity type/name, applicable registration status and business-account readiness: __________. Keep sensitive account identifiers/credentials out of this worksheet.

**Owner answer:** __________

### Q20. Who will own the jewellery offered initially?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-6; BD-11; FIN-12/14; CA-03/LR-02.

- **A** — ★ One operating seller owns purchased stock.
- **B** — Stock is supplier-owned/consigned.
- **C** — A mixture of owned, consigned or manufactured stock.

Recommendation rationale: Simplest initial attribution, cost and inventory books. Consignment/mixed sources require explicit agreements and source/valuation facts; hub custody alone never proves ownership. Enter which supplier/acquisition documents currently exist.

**Owner answer:** __________

### Q21. What geographical scope should the Bengaluru pilot serve?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-10; BD-07/08/13; FIN-04; CA-02/03.

- **A** — ★ Reviewed Bengaluru service areas within Karnataka only.
- **B** — Reviewed Bengaluru coverage including nearby border-state addresses.

Recommendation rationale: Limits initial jurisdiction scenarios while retaining future schema support. It does not approve particular PIN codes or broader Karnataka coverage. Border-state supply requires separate operational and tax review, not implicit intercity stock sourcing.

**Owner answer:** __________

### Q22. What commercial purpose should the upfront booking payment serve?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-1; BD-01; FIN-05/11; CA-01; LR-01.

- **A** — ★ Refundable booking security, with eligible funds applied/refunded under the chosen policy.
- **B** — An advance toward an eventual jewellery purchase.
- **C** — A separately disclosed trial/visit service fee.

Recommendation rationale: A customer-trust starting proposal consistent with buy-any/no-purchase visits, conditional on visit economics. The CA determines actual treatment from substance; calling it a deposit does not make it legally one. A service fee needs its own refund/tax/document policy. [CaratLane home-trial offer](https://www.caratlane.com/cart/try-at-home) illustrates low-commitment expectations; it does not override Arka Tara's approved upfront-collection requirement.

**Owner answer:** __________

### Q23. How should the upfront payment be calculated?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-1; BD-01; R-02.

- **A** — ★ One configured amount per home-visit booking.
- **B** — Configured amounts based on selected category/plan/box contents.
- **C** — A disclosed formula based on selected jewellery value.

Recommendation rationale: Matches one booking/one combined visit. Multiple boxes do not create separate checkouts. Enter the rupee amount or precise formula after cost review; no defensible rupee recommendation is possible from unknown visit costs/margins.

Required value: INR __________ per booking, or formula __________. Before setting it, supply estimated round-trip visit cost, QC/return cost, average sale margin and expected no-purchase share; rough estimates must be identified as estimates.

**Owner answer:** __________

### Q24. How should eligible refundable upfront funds be treated after the visit?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-1; BD-01; R-25; FIN-05/11.

- **A** — ★ Apply them to an approved purchase and refund unapplied funds, including a no-purchase visit.
- **B** — Refund them separately and collect the full purchase amount independently.

Recommendation rationale: Reduces duplicate money movement. Includes purchases below the refundable amount and partial purchases; upfront application never reduces historical jewellery selling value. Conditional on Q22: a service fee requires distinct reviewed treatment.

**Owner answer:** __________

### Q25. What should happen on cancellation, rescheduling or a no-show?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-1; BD-01/02/07; FIN-05; CA-01/LR-01.

- **A** — ★ Start with a disclosed refund/rescheduling policy; introduce retention only after cost evidence and CA/legal review.
- **B** — Introduce specifically reviewed cancellation/no-show retention from launch.

Recommendation rationale: Simple customer-facing starting policy. Supply separate outcomes for timely/late customer cancellation, business cancellation/failure, rescheduling and no-show. Any cutoff or retained rupee amount must be expressly entered and professionally reviewed; no blanket forfeiture or indefinite pending balance.

Scenario outcomes/cutoffs/retained amounts: customer cancellation __________; business cancellation/failure __________; reschedule __________; no-show __________. If retention is deferred, say so explicitly.

**Owner answer:** __________

### Q26. How should consumer-facing jewellery prices be displayed?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-2; BD-04; FIN-02; CA-02/LR-04.

- **A** — ★ Show the final GST-inclusive product price where applicable, with a clear tax/charge breakdown.
- **B** — Show base price and tax together with an equally clear final total, subject to legal approval of presentation.

Recommendation rationale: Avoids doorstep/checkout price surprises. All compulsory charges must be properly disclosed, and an inclusive-price claim must never be followed by unexpected GST. Backend net/tax/gross calculation support is a separate technical contract. [CCPA price-disclosure enforcement](https://www.pib.gov.in/PressReleasePage.aspx?PRID=2171807&lang=2&reg=3)

**Owner answer:** __________

### Q27. When should the jewellery price become committed?

Classification: BUSINESS DECISION. Status: PENDING. References: BD-04; FIN-01/02/10.

- **A** — ★ At confirmed booking, for an expressly defined validity period.
- **B** — At the visit, with disclosed recalculation and acceptance before payment.

Recommendation rationale: Appropriate starting proposal for a fixed-price Silver pilot, conditional on margins and visit timing. Enter price-guarantee validity and rescheduling/expiry treatment. Gold/weight-based pricing remains separately gated; no formula is silently selected.

Price validity and rescheduling/expiry treatment: __________. Future Gold price source/formula is PENDING unless separately approved.

**Owner answer:** __________

### Q28. Should launch support centrally approved discounts or additional charges beyond the upfront component selected in Q22?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-3; FIN-01/02; CA-02/LR-01/LR-04.

- **A** — ★ Neither initially.
- **B** — Centrally configured discounts only.
- **C** — Centrally configured discounts and separately disclosed charges.

Recommendation rationale: Simplest initial sale/document arithmetic. This choice does not remove the upfront component selected in Q22. All options preserve R-19: agents cannot manually override prices. B/C require typed calculation/document treatment; charges and promotion claims must be reviewed, not hidden in a signed number.

**Owner answer:** __________

### Q29. What happens when final payment is pending or unverifiable at the visit?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-4; BD-05/V-5; FIN-06; CA-02/LR-01/LR-03.

- **A** — ★ Keep selected pieces in agent custody; completed handover requires verified payment and the required issued documents.
- **B** — Offer a separately approved deferred-payment arrangement with explicit eligibility, debt/custody and collection controls.

Recommendation rationale: Preserves a clear paid-sale boundary and reduces collection risk. B is a material additional capability requiring business/CA/legal review, not informal agent discretion. CA separately identifies the legal invoice timing; temporary trying is not purchased handover.

**Owner answer:** __________

### Q30. Which payment methods should launch accept?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-5; BD-06; FIN-12/13/15.

- **A** — ★ UPI and cards through the approved business gateway.
- **B** — UPI only through that gateway.
- **C** — Digital payments plus cash under separately approved controls.

Recommendation rationale: Offers choice without cash handling. C cannot bypass cash limits, verified evidence or BD-06 readiness. Manual screenshots/client callbacks never establish payment under any option.

**Owner answer:** __________

### Q31. What voluntary after-sale return/exchange benefit should be offered for Silver?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-7; BD-10; FIN-05/08; CA-02/LR-01.

- **A** — ★ A reviewed 15-day window for qualifying unused/unworn Silver returns or exchanges.
- **B** — A reviewed seven-day window for qualifying unused/unworn Silver returns or exchanges.
- **C** — Mandatory remedies only initially, without an additional voluntary change-of-mind benefit.

Recommendation rationale: A proposed customer-trust position; pickup/QC/refund costs and exceptions must be checked before acceptance. Every option preserves mandatory remedies for defective/incorrect/nonconforming goods. Supply condition/category/personalization exclusions and pickup-cost policy; do not copy a competitor's exclusions blindly. [GIVA's published return policy](https://www.giva.co/pages/shipping-and-handling) provides a current market example, not evidence that its economics or full terms fit this business.

Qualifying conditions/exclusions, pickup cost and refund eligibility: __________. Mandatory remedies remain regardless of the voluntary window.

**Owner answer:** __________

### Q32. How should trial damage/loss and write-offs be handled?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-8; BD-09; FIN-14; LR-01/CA-02/03.

- **A** — ★ Ordinary trial wear is a business cost; documented abnormal damage/loss is reviewed by the founder/supervisor under legally approved terms, with no automatic customer charge.
- **B** — The business absorbs pilot damage/loss, with documented founder-approved write-offs and no customer damage charges.

Recommendation rationale: Separates ordinary experience from exceptional evidence and avoids automatic liability from an OTP or failed QC. Specify agent responsibility separately with legal advice; any customer charge, valuation or retention requires approval/evidence.

**Owner answer:** __________

### Q33. What operating target should apply to approved refunds?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-1/7; FIN-05; BD-01/10/15.

- **A** — ★ Initiate by the next business day after the applicable eligibility checks are complete.
- **B** — Initiate within three business days after applicable eligibility checks are complete.

Recommendation rationale: An internal processing target, not a guarantee of bank credit. Name the approver and reconciliation owner. Cancellation with no dispatched goods does not require physical return QC. Required remedies/timelines remain subject to law. [Razorpay normal-refund documentation](https://razorpay.com/docs/payments/refunds/normal/) distinguishes initiation from bank credit and original-source repayment.

**Owner answer:** __________

### Q34. How should initial accounting reconciliation be operated?

Classification: BUSINESS DECISION. Status: PENDING. References: FIN-05/17; OD-12; BD-15/CA-01/02/03.

- **A** — ★ Regular reconciliation and CA-reviewed exports/imports using the chosen accounting tool.
- **B** — Direct accounting-system integration from launch.

Recommendation rationale: Reduces integration work while preserving gross receipts, fees, tax on fees, refunds, settlements and bank matches. Enter the accounting tool, reconciliation owner, review frequency and finance approver. Invoice totals, cash and revenue remain separately defined.

Accounting tool __________; reconciliation frequency/owner __________; refund/financial approver __________.

**Owner answer:** __________

### Q35. Who can close and reopen accounting/reporting periods?

Classification: BUSINESS DECISION. Status: PENDING. References: FIN-09/17; CA-02/03.

- **A** — ★ Designated finance/owner role closes under a CA-agreed calendar; reopening needs attributable owner/finance approval and CA review where filed reports are affected.
- **B** — Require separate requester and approver for every close/reopen.

Recommendation rationale: A controlled starting workflow with an attributable reason/history. Supply the role/person and agreed close calendar. Legitimately late provider evidence remains recordable; professional advice determines posting/return corrections.

Closing calendar/approver/reopening controls: __________.

**Owner answer:** __________

### Q36. How should Gold and Hyderabad expansion be scheduled?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-11; BD-12/13; FIN-04/14/15; CA-02/03/LR-03.

- **A** — ★ Activate each independently after its own product, tax, stock, payment and operational gates pass.
- **B** — Plan a coordinated launch after both sets of gates pass.

Recommendation rationale: One change at a time aids learning and isolates operational risk. Gold can launch in Bengaluru, and Silver can launch in Hyderabad; Telangana registration is not a prerequisite for every Gold transaction. No legal prerequisite can be waived by either answer.

**Owner answer:** __________

### Q37. What initial stock-hold duration should apply after explicit payment initiation?

Classification: BUSINESS DECISION. Status: PENDING. References: BD-02; D-27/R-40; FIN-05/06.

- **A** — ★ A configured 15-minute ordinary payment window.
- **B** — A configured 10-minute ordinary payment window.
- **C** — A configured 30-minute ordinary payment window.

Recommendation rationale: Proposed operating duration, not a legal/provider benchmark. No stock hold occurs at draft/OTP. If capture is uncertain, expiry alone cannot release protected stock; reconciliation and attributable resolution are required. Confirm provider support before live activation.

**Owner answer:** __________

### Q38. What should happen to verified payment received after a booking can no longer be fulfilled?

Classification: BUSINESS DECISION. Status: PENDING. References: BD-02/05; D-27; FIN-05.

- **A** — ★ Reconcile the receipt, refund it under the approved process, and offer a fresh booking.
- **B** — Allow staff to seek fresh customer agreement and reviewed new availability/acceptance, otherwise refund.

Recommendation rationale: Simplest default outcome without reviving released stock or concealing receipts. B needs explicit new commitment and money provenance; neither answer allows automatic resurrection of an expired booking or silent cross-booking/account money transfer.

**Owner answer:** __________

### Q39. Can the purchase selection change while a payment request is outstanding?

Classification: BUSINESS DECISION. Status: PENDING. References: BD-05; R-23/24; FIN-01/05/06.

- **A** — ★ Freeze the selection until the existing attempt is safely resolved/cancelled, then issue a new authoritative version.
- **B** — Support pre-capture revision only after verified safe cancellation/supersession, with a new quote/attempt and retained old evidence.

Recommendation rationale: Minimizes stale quotes and accidental double collection. QR and link still share the same active final attempt. Unknown capture state must be reconciled, and late/extra real receipts remain evidence.

**Owner answer:** __________

### Q40. What is the normal invoicing arrangement for selected pieces from one visit?

Classification: BUSINESS DECISION. Status: PENDING. References: BD-05; R-02; FIN-07.

- **A** — ★ One combined purchase/invoice for the selected pieces across its category boxes.
- **B** — Deliberate split invoicing for specified selections, without independent box checkouts.

Recommendation rationale: Simple to explain/reconcile. This is a normal-flow policy, not lifetime uniqueness per booking or physical unit. Corrections and legitimate later sales retain separate evidence. Enter the actual reason/rule if B is needed.

**Owner answer:** __________

### Q41. What evidence should support Silver purity and hallmark claims at launch?

Classification: BUSINESS DECISION. Status: PENDING. References: BD-09/11/12; FIN-14; LR-03.

- **A** — ★ Source BIS-hallmarked Silver where feasible from eligible suppliers and retain actual per-piece/source evidence.
- **B** — Source unhallmarked verified-purity Silver with reviewed supplier/testing evidence, without unsupported BIS/HUID claims.

Recommendation rationale: Favors verifiable trust claims, subject to costs and sourcing. Silver HUID is voluntary under the cited scheme, so B is not inherently an invalid offering. Record supplier invoices, cost, owner and relevant weights in either case; claim only what actual evidence supports. [BIS Silver HUID guidance](https://www.bis.gov.in/wp-content/uploads/2025/10/BIS-NEWSLETTER-OCTOBER-2025.pdf)

**Owner answer:** __________

### Q42. Should launch accept transactions that trigger additional high-value identity requirements?

Classification: BUSINESS DECISION. Status: PENDING. References: FIN-15; BD-06/12; CA-02/LR-04.

- **A** — ★ Block those transactions until reviewed PAN/declaration and restricted-data controls are ready.
- **B** — Complete the applicable controls before Silver launch and permit those transactions.

Recommendation rationale: Simplest safe launch scope; threshold/applicability must be professionally confirmed and configured. These controls can apply to Silver, not just Gold. Do not collect unnecessary identity data from every guest. [Official Rule 159](https://www.incometaxindia.gov.in/documents/20117/42998/Rule-159_2026-03-25_06-11-40_0d46f6_en.pdf/afebd591-1b53-4913-0d57-dc518fa4a5aa?t=1774449122128&version=2.0) supports the reported threshold/form subject to exemptions and verification.

**Owner answer:** __________

### Q43. What qualified CA review evidence is actually available?

Classification: BUSINESS DECISION. Status: PENDING. References: OD-12; CA-01/02/03; D-28.

- **A** — A qualified CA has already supplied written advice; its scope/date and relevant answers can be recorded.
- **B** — A qualified CA is engaged, but written answers are pending.
- **C** — Only Claude/AI analysis is available so far; a qualified CA still needs to review.

Factual confirmation: Answer the actual status. Claude's archived report explicitly says it is not a CA or lawyer. Recommended process: obtain applicable written professional advice before activating dependent finance behaviour; an owner/AI selection cannot establish legal tax treatment.

Qualified review scope/date/evidence reference: __________. Expected first-year turnover and forecast basis for CA registration/HSN questions: __________.

**Owner answer:** __________

## Compatibility checks before finalizing the flow

- Q22 is commercial intention; CA Q1 determines actual classification. Q24's refundable-fund options cannot silently turn a genuine service fee into a refundable deposit.
- Q23 requires an explicit amount/formula. A “recommended options” answer without that value leaves it pending.
- Q25 requires a scenario-specific policy; business failure and customer cancellation are not silently equated.
- Q3B reopens D-24/R-34 and the proposed guest-history exclusion; it needs an explicit revised ownership/privacy rule before any claim model or workflow.
- Q6B changes signup timing and new account checkout creation; pending verification intentions must not be treated as customer accounts.
- Q7 concerns usable checkout details only. Paid/pending capture/financial records and legal preservation follow their separate rules.
- Q8A uses mailbox-based consumer authentication, not phishing-resistant or independent multi-factor assurance. Staff MFA remains separately required.
- Q11A defers trusted-browser phone login. Q12-Q16 may settle its future design without authorizing activation now.
- Q28B/C require calculation/document/promotion support. R-19 continues to prohibit independent agent price overrides.
- Q29B is an additional reviewed deferred-payment capability. It does not redefine financially confirmed purchases or authorize informal handover.
- Q37 timeout never releases capture-uncertain stock without reconciliation. Q38B needs explicit fresh commitment/source linkage, not resurrection of old reservations.
- Q36 keeps Gold and Hyderabad independent; Q42 applies high-value controls to any relevant material.
- Selecting recommendations cannot close CA/LR gates. Unanswered required values remain unset, with affected activation blocked.

## CA REVIEW questions

CA Q1-Q27 correspond to the finance report's original question IDs; wording is refined to avoid misleading yes/no binaries. CA Q28 is additional. The original wording is preserved in the archived report.

For each question select an evidence disposition:
- **A — PENDING:** qualified advice is not yet available.
- **B — REVIEWED:** written applicable advice and conditions are supplied.
- **C — NOT APPLICABLE:** qualified reviewer provides the reason and scope.

Recommended process: obtain B or justified C before enabling dependent behaviour. Do not select a tax rate, legal classification or exemption because it looks convenient. The actual answer can be yes/no, depends, a number or a scenario table as appropriate.

### CA Q1. For the actual Q22-Q25 contract and each upfront component, classify refundable security, advance against goods, service consideration, or another treatment; specify recognition, liability, tax point and required evidence.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Classification(s), conditions and entries; allow mixed components.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q2. If the payment is an advance against goods, identify the receipt-voucher requirement/timing and applicable advance-tax notification, including exceptions.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Required / not required / depends, with basis and document conditions.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q3. For any actual trial/visit service component, specify SAC, tax components/rates, supply timing and invoice particulars.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Effective classification/rate schedule; not a guessed number.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q4. Separately classify retained amounts for no-show, customer cancellation, business failure, completed no-purchase visit and damage/loss. Identify taxability, rate, evidence and document.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Scenario table; not one blanket yes/no.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q5. For each deposit/advance/service refund scenario, identify receipt/refund voucher, credit note or other record, tax adjustment, source linkage and timing.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Scenario/document matrix. Rule 51 is not assumed for every deposit refund.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q6. Confirm compliant price disclosure and net/tax/gross calculation for the chosen owner Q26 presentation, including compulsory charges.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Calculation/disclosure conditions; owner intent and legal requirements distinguished.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q7. Specify HSN/classification and applicable effective rate/components for each actual Silver product, and future Gold products separately.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Product classification schedule; confirm rather than assume 3% or one eight-digit code.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q8. Which HSN/SAC particulars/digit requirements apply to our document types, product mix and actual expected turnover?

Classification: CA REVIEW. Status: PENDING.

Expected answer: Required particulars, turnover scope, dates and exceptions.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q9. Provide the precise rounding specification for taxable values, component taxes, line/header totals, final payable and gateway minor units.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Worked examples and precision/round-off rules; line/invoice binary is insufficient.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q10. For the attended home-trial contract, identify when supply/acceptance occurs, the dispatch/return documents and applicable approval-sale deadline.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Event/document rules separate from payment; conditions and statutory basis.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q11. For Q29's approved handover flow, when must the invoice legally exist, including payment uncertainty or goods remaining with the customer?

Classification: CA REVIEW. Status: PENDING.

Expected answer: Event sequence and independent timestamps/evidence.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q12. Specify required delivery/return challan particulars, unit/consignment valuation, quantity/weight, signatures/exemptions and number/date rules.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Required fields and valuation/return-document policy per applicable kind.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q13. Verify transport/e-way/Rule 138F requirements for Karnataka Silver, Karnataka Gold, Telangana Silver and Telangana Gold, including routes and exceptions.

Classification: CA REVIEW. Status: PENDING.

Expected answer: State notification, effective date, applicable value threshold and document requirement; UNKNOWN if not verified.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q14. Approve numbering namespaces by registration, document kind and financial year; specify format/length, annual reset, location series and void/unused-number reporting.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Reviewed format/scope, allowing lawful multiple series; not an assumption that exactly one series is mandatory.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q15. Specify cancellation/correction rules separately for drafts, issued unsupplied transactions, clerical/value/tax errors, IRN where applicable and filed returns.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Permitted document/event/approval per scenario; avoid cancellation-or-credit-note false binary.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q16. Specify discount treatment and separately classify visit, delivery and genuine compensation/damage charges, including valuation and documents.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Scenario-specific tax/calculation/document rules; agents never gain price override permission.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q17. For after-sale return/exchange scenarios, distinguish commercial credit/refund from GST credit adjustment, conditions, deadlines and treatment after deadlines.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Scenario matrix; legal tax-adjustment cutoff is not a universal ban on commercial refunds.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q18. Specify eligibility/conditions and evidence for input credit on gateway fee taxes; mapping of gross receipts, fees, tax, adjustments and net settlement.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Accounting entries/evidence and applicable restrictions.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q19. For chargebacks, distinguish pending dispute, payment reversal, cancelled/returned supply and continuing receivable. What records/tax adjustments apply?

Classification: CA REVIEW. Status: PENDING.

Expected answer: Scenario-specific financial/GST treatment; no automatic credit-note assumption.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q20. Specify inventory valuation/cost allocation, eligible costs, impairment/NRV, owned versus consigned/manufactured stock, trial stock, write-offs and returns/resales.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Accounting method and source evidence per inventory arrangement.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q21. For actual Hyderabad establishments/supply arrangements, specify registration timing and interstate stock-transfer treatment by entity/GST registration.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Registration/transfer scenarios, documents and entries; material independent.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q22. For border-state or bill-to/ship-to cases, determine supplying registration/place of supply and components for our actual delivery arrangement.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Jurisdiction and exception matrix; no unconditional PIN-only determination.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q23. Given actual seller, goods/services mix, geography and turnover, specify GST-registration readiness, collection account ownership and permitted agency/inter-entity arrangements.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Required registrations, bank/gateway attribution and effective dates.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q24. Confirm current PAN/declaration threshold, relevant person/transaction scope, exemptions, verification, Form 97/equivalent evidence, reporting and retention obligations.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Current notified rule, threshold/strictness, conditions and restricted-data requirements. Applies to Silver as applicable.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q25. Determine whether and when PMLA/reporting obligations apply to our actual digital/cash/related transaction model, separately from income-tax cash restrictions.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Applicable regime or not applicable with basis; thresholds, grouping and evidence if applicable.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q26. With legal counsel, approve retention per record class: fields, legal basis, start event, minimum/maximum period, litigation holds, access, deletion/anonymization and provider/backup treatment.

Classification: CA REVIEW. Status: PENDING; coordinated LEGAL REVIEW LR-04.

Expected answer: Record-class schedule; not one longest number of years after financial-year end.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q27. Specify GST time-of-supply, document issue/reporting periods, true late recording, corrections and close/reopen handling without false backdating.

Classification: CA REVIEW. Status: PENDING.

Expected answer: Posting/return rules and worked date-boundary cases, distinct from immutable recording time.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### CA Q28. Which accounting framework applies to the selected entity, and which events determine jewellery/service revenue, receivables, upfront liabilities, refunds, fees and costs?

Classification: CA REVIEW. Status: PENDING.

Expected answer: Recognition policy and journal/export mapping; invoices minus credits is not presumed to equal recognized revenue.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

## LEGAL REVIEW questions

For each question select:
- **A — PENDING:** qualified legal review is required and unresolved.
- **B — REVIEWED:** applicable written legal advice/approved terms and conditions are supplied.
- **C — NOT APPLICABLE:** qualified reviewer supplies the basis and scope.

Recommended process: verify applicability and obtain reviewed conditions before activation. Owner preference cannot waive an applicable duty.

### L1. LR-01: Are the Q22-Q25/Q29/Q31-Q33 booking, cancellation, no-show, liability, voluntary-return and refund terms enforceable and consistent with mandatory consumer remedies?

Classification: LEGAL REVIEW. Status: PENDING.

Review scope: Confirm explicit agreement, ordinary wear versus liability, evidence, unilateral business cancellation, unfair terms and refund timing. OTP alone is not KYC or legal liability.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### L2. LR-02: Is the named seller/collector/stock owner arrangement legally effective, including consignment or later entity transition?

Classification: LEGAL REVIEW. Status: PENDING.

Review scope: Confirm contracting identity, agreements, registration/account attribution and forward transfers without relabelling historical transactions.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### L3. LR-03: Which dispatch/transport/product/purity/hallmark obligations apply to the actual launch goods, locations and movement?

Classification: LEGAL REVIEW. Status: PENDING.

Review scope: Verify article/district exemptions, actual claim evidence, supplier/source documentation and Gold/Hyderabad separately.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### L4. LR-04: Approve the personal-data lifecycle for unverified intents, drafts, guest/account history, statutory financial copies, identification evidence, provider logs and backups.

Classification: LEGAL REVIEW. Status: PENDING.

Review scope: Coordinate CA Q26. Separate functional expiry from legal/security retention; lawful data access/erasure, grievance and dispute holds. Confirm relevant DPDP commencement phases and duties.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### L5. LR-04: Which current and announced e-commerce/dark-pattern obligations apply at the intended launch date, including the announced 1 January 2027 amendments?

Classification: LEGAL REVIEW. Status: PENDING.

Review scope: Obtain notified text and applicability for total price/charges, promotional prior-price history, grievance handling, NCH and other required controls. Do not infer full applicability from a press announcement alone.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

### L6. LR-04: Approve guest/account recovery, document delivery and service-versus-marketing communications and consent/notice boundaries.

Classification: LEGAL REVIEW. Status: PENDING.

Review scope: Guest has no required email under the target design. Determine secure receipt/invoice delivery by booking access and selected channels, statutory contact facts, provider/DLT/WhatsApp requirements and minimized logs; signup/OTP does not imply marketing consent.

Disposition A/B/C: __________. Advice/conditions: __________. Reviewer/date/source/reference: __________.

## Mapping of all 12 report owner decisions

| Report ID | Combined questions / review |
| --- | --- |
| OD-1 upfront purpose/amount/outcomes | Q22-Q25, Q33/Q37-Q38; CA Q1-Q5; L1 |
| OD-2 tax-inclusive display | Q26-Q27; CA Q6-Q9; L5 |
| OD-3 discounts/additional charges | Q28; CA Q16; L1/L5; R-19 already prohibits manual agent overrides |
| OD-4 pending-payment handover | Q29/Q37-Q39; CA Q10-Q11; L1/L3 |
| OD-5 methods | Q30/Q42; CA Q24-Q25 |
| OD-6 seller/collector/stock owner | Q19-Q21; CA Q21-Q23; L2 |
| OD-7 returns/exchanges | Q31/Q33; CA Q15-Q17; L1 |
| OD-8 damage/loss/write-off | Q32; CA Q20; L1-L3 |
| OD-9 retention/erasure | Q5-Q7/Q14-Q15; CA Q26; L4 |
| OD-10 service geography | Q21; CA Q13/Q21-Q22; L3 |
| OD-11 Gold readiness | Q36/Q41-Q42; CA Q7/Q13/Q21/Q24; L3; separated from Hyderabad |
| OD-12 qualified CA evidence | Q43; CA Q1-Q28; does not claim Claude is a qualified CA |

## Research qualifications added after finance review

The [official Rule 159](https://www.incometaxindia.gov.in/documents/20117/42998/Rule-159_2026-03-25_06-11-40_0d46f6_en.pdf/afebd591-1b53-4913-0d57-dc518fa4a5aa?t=1774449122128&version=2.0) and [Income Tax Department form guidance](https://www.incometax.gov.in/iec/foportal/help/all-topics/e-filing-services/income-tax-forms?mobile-app=1) support the report's current rule/form references; actual exemptions, verification and reporting still need CA advice.

The [official DPDP Rules](https://www.meity.gov.in/static/uploads/2025/11/53450e6e5dc0bfa85ebd78686cadad39.pdf) and [commencement notification](https://www.meity.gov.in/static/uploads/2025/11/c56ceae6c383460ca69577428d36828b.pdf) require a phased, record-class privacy/retention assessment. Neither a 24-hour draft lifetime nor an invoice's financial retention period alone determines every record's lawful deletion time.

The [government's 10 September 2026 e-commerce announcement](https://www.pib.gov.in/newsite/erelcontent.aspx?lang=2&reg=48&relid=294532) describes changes effective 1 January 2027, including promotional-price history and consumer-facing controls. L5 requests the actual notified text/applicability review, rather than silently treating the finance report as a complete consumer-law review.

## Next step after answers

Validate contradictions and missing values, then record only explicit approvals with historical decision trail in canonical docs. Keep qualified review dependencies pending where evidence is absent. Implement only authorized targeted model/integrity corrections and relevant regressions within the existing task ownership; workflows, provider integration and final UI remain at their approved stages. This worksheet does not itself approve any answer or complete Task 4.
