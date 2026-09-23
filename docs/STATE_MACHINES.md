# Proposed state machines

Status: DOCUMENTATION FOR USER REVIEW. Recorded: 2026-09-19. No state machine has been implemented.

State names, transition shapes and mechanisms below are **PROPOSALS**. Approved invariants are binding; unanswered policy choices are not. A transition described as allowed means structurally permitted **only when every stated guard is satisfied and its BUSINESS DECISION, CA REVIEW or LEGAL REVIEW gate has been resolved for that behavior**. An unresolved gate is not an instruction to choose a default. All referenced IDs remain open in [DECISIONS.md](DECISIONS.md).

## Separate facts, common transition discipline

Use distinct state/evidence for booking/visit progress, reservation, physical custody/condition, purchase selection/confirmation, payment, invoice issuance, refund and notification delivery. A single `COMPLETED` flag cannot reliably describe all of these.

For every transition, record the previous/new state, actor or verified system event, occurrence/recording time, correlation/idempotency key and reason where appropriate. Enforce permissions and guards server-side inside the relevant transaction. An API request for a status is a command to validate, not permission to overwrite a status field. Retry the same action idempotently; reject invalid or stale requests without destroying evidence.

Allowed transitions are enumerated below; unlisted transitions are forbidden until explicitly designed and reviewed. Status read models may show useful summaries, but they cannot mutate the authoritative machine. Corrections add evidence and follow a reviewed correction workflow rather than editing away a prior event.

## TrialBooking: booking commitment and visit progress

Proposed states: `DRAFT`, `CONFIRMATION_PENDING`, `CONFIRMED`, `PREPARING`, `READY_FOR_DISPATCH`, `OUT_FOR_DELIVERY`, `IN_PERSON_TRIAL`, `VISIT_CONCLUDED`, `CANCELLED`.

The anonymous local cart is not a DRAFT booking and never reserves inventory. A persisted draft begins only when a server-side booking workflow is invoked. `CONFIRMATION_PENDING` does not declare that every booking owes a deposit. Payment state is separate. `VISIT_CONCLUDED` records the operational visit result; payment/reconciliation, returns and QC may require later work, subject to the approved handover/closure rule.

| From | To | Trigger and required guards | Open gates |
| --- | --- | --- | --- |
| DRAFT | CONFIRMATION_PENDING | Backend accepts a valid booking submission after rechecking serviceability, box categories, offering/configuration and authoritative calculations. Record the approval/hold requirements actually applicable. | BUSINESS DECISION BD-01, BD-02, BD-03, BD-04, BD-07, BD-08, BD-13. |
| CONFIRMATION_PENDING | CONFIRMED | All approved acceptance requirements met; any required unit commitments are valid and atomically recorded. Verified payment is a guard only if an approved policy makes it one. | BUSINESS DECISION BD-01, BD-02, BD-08. |
| CONFIRMED | PREPARING | Authorized operations workflow starts preparation against current allocations/manifest. | BUSINESS DECISION BD-08, BD-09. |
| PREPARING | READY_FOR_DISPATCH | Every required piece checked/packed; category composition and combined booking manifest agree; required dispatch evidence available. | BUSINESS DECISION BD-03, BD-08, BD-09; CA REVIEW CA-02; LEGAL REVIEW LR-03. |
| READY_FOR_DISPATCH | OUT_FOR_DELIVERY | Valid assignment, recorded handover and manifest dispatch; exact unit custody updated consistently. | BUSINESS DECISION BD-07, BD-08, BD-09; CA REVIEW CA-02; LEGAL REVIEW LR-01, LR-03. |
| OUT_FOR_DELIVERY | IN_PERSON_TRIAL | Assigned, authorized agent submits the one valid Arrival OTP; backend verifies expiry/attempt limits and atomically consumes it. Record arrival time and trial start once. | Approved OTP invariant; exceptional revisit behavior remains BUSINESS DECISION BD-07. |
| IN_PERSON_TRIAL | VISIT_CONCLUDED | Record customer selection/no-purchase outcome and every unit's sold/pending/return disposition and custody; enforce the approved payment/handover rule. | BUSINESS DECISION BD-05, BD-07, BD-09. |
| OUT_FOR_DELIVERY | VISIT_CONCLUDED | Explicit failed-visit/no-show/aborted-visit outcome, with custody and recovery actions recorded. Does not assert a trial occurred. | BUSINESS DECISION BD-07, BD-09 and deposit consequences BD-01. |
| DRAFT or CONFIRMATION_PENDING | CANCELLED | Explicit cancellation/expiry decision and safe release of relevant un-dispatched commitments under approved policy. Receipt/refund evidence remains. | BUSINESS DECISION BD-01, BD-02, BD-07. |
| CONFIRMED, PREPARING or READY_FOR_DISPATCH | CANCELLED | Authorized cancellation with unit-by-unit recovery/unpacking/disposition; no automatic stock availability if an inspection is required. | BUSINESS DECISION BD-01, BD-02, BD-07, BD-09. |

This is a proposed straight-through path. Rescheduling, re-packing and an exceptional repeat/split visit require an explicit transition design after BD-07/BD-08/BD-09, preserving earlier attempts and assignments. Do not implement an arbitrary backward status edit to approximate these cases. A cancellation request after dispatch is an operational event needing a reviewed visit/recovery outcome; it cannot erase the actual dispatch or release stock as if it remained at the hub.

Track `trial_outcome` independently, with candidate values such as unset, selected-for-purchase, no-purchase, failed-visit or aborted. A selection is still not a financially confirmed purchase. Proposed purchase/return/financial completion projections should identify outstanding work before presenting any overall completion indicator.

The roadmap examples `PAYMENT_PENDING` and `FINAL_PAYMENT_PENDING` are represented by payment context/attempt state, not mutually exclusive booking states. `NO_PURCHASE` is a visit outcome, and `COMPLETED` needs an approved definition before becoming a final business state. This avoids treating a no-purchase visit awaiting returns or a concluded visit with reconciliation work as fully settled.

Forbidden examples:

- DRAFT directly to OUT_FOR_DELIVERY or IN_PERSON_TRIAL.
- OUT_FOR_DELIVERY to IN_PERSON_TRIAL based on a frontend flag, agent assertion or notification-delivered event instead of backend OTP verification.
- Any payment webhook directly setting all trial items SOLD or resurrecting CANCELLED/expired booking commitments.
- Box-level status changes creating independent checkouts or bypassing the normal combined visit.
- CANCELLED or VISIT_CONCLUDED silently reset to a new visit; any approved exception must retain the old history.

## InventoryUnit: availability/disposition with separate custody

Proposed lifecycle states: `AVAILABLE`, `HELD`, `RESERVED`, `PACKED`, `OUT_FOR_DELIVERY`, `IN_PERSON_TRIAL`, `RETURNING_TO_HUB`, `QC_PENDING`, `SOLD`, `QUARANTINED`, `RETIRED`. A missing/lost flag and condition findings may be separate exception dimensions that always block availability.

`HELD` is optional temporary commitment, not a required deposit policy. `RESERVATION_PENDING` is not proposed as a durable stock state: a request being validated must not appear to have acquired a piece. Committed reservation records establish the guarantee. The exact hold/reserve sequence remains BUSINESS DECISION BD-02.

| From | To | Required trigger/guards | Open gates |
| --- | --- | --- | --- |
| AVAILABLE | HELD or RESERVED | Backend atomic eligibility check and exclusive active commitment; route chosen only by approved reservation policy. | BUSINESS DECISION BD-02, BD-08, BD-09. |
| HELD | RESERVED | Approved booking commitment conditions met before expiry; locks recheck the unit and hold. | BUSINESS DECISION BD-01, BD-02. |
| HELD or RESERVED | AVAILABLE | Approved expiry/cancellation/release; confirm no dispatch, custody change, sale or condition issue. Append release evidence. | BUSINESS DECISION BD-02, BD-09. |
| RESERVED | PACKED | Exact piece verified in the booking manifest and handover/preparation recorded. | BUSINESS DECISION BD-03, BD-08, BD-09. |
| PACKED | OUT_FOR_DELIVERY | Recorded dispatch, assigned custodian and reviewed transport-document prerequisites. | BUSINESS DECISION BD-07, BD-09; CA REVIEW CA-02; LEGAL REVIEW LR-03. |
| OUT_FOR_DELIVERY | IN_PERSON_TRIAL | Same verified arrival event as the booking/visit, applied once to the relevant manifest units. | Approved Arrival OTP rule; BUSINESS DECISION BD-09 for exceptional discrepancies. |
| IN_PERSON_TRIAL | SOLD | Exact piece selected in the current authoritative purchase version, financially confirmed under approved collection/allocation rules and required handover guard satisfied. | BUSINESS DECISION BD-04, BD-05, BD-06; CA REVIEW CA-01, CA-02. |
| IN_PERSON_TRIAL or OUT_FOR_DELIVERY | RETURNING_TO_HUB | Unpurchased/failed-visit recovery recorded with current custodian; pending payment requires reviewed disposition. | BUSINESS DECISION BD-05, BD-07, BD-09. |
| RETURNING_TO_HUB | QC_PENDING | Actual receiving hub verifies receipt and custody; discrepancy handled explicitly. | Approved return/QC invariant; BUSINESS DECISION BD-09 for inspection process. |
| QC_PENDING | AVAILABLE | Authorized passing QC, known hub custody and no conflicting commitment/condition issue. | Approved QC invariant; BUSINESS DECISION BD-09 for passing criteria. |
| QC_PENDING | QUARANTINED | Failed or indeterminate inspection; record evidence and block availability. | BUSINESS DECISION BD-09. |
| QUARANTINED | QC_PENDING | Approved remediation/reinspection with traceable custody and no active sale commitment. | BUSINESS DECISION BD-09. |
| QUARANTINED or QC_PENDING | RETIRED | Reviewed disposition with actor/reason and relevant ownership/accounting approval. | BUSINESS DECISION BD-09; CA REVIEW CA-02/CA-03 and LEGAL REVIEW LR-01/LR-02 where applicable. |
| PACKED | QC_PENDING | Pre-dispatch cancellation/unpacking with hub receipt and inspection route selected by approved process. | BUSINESS DECISION BD-02, BD-07, BD-09. |
| SOLD | QC_PENDING | Only a separately authorized after-sale return with actual hub receipt and original sale linkage; never a payment-event side effect. | BUSINESS DECISION BD-10; CA REVIEW CA-02; LEGAL REVIEW LR-01. |

Damage/loss discovered in another state requires a reviewed exception action that blocks new commitments while preserving the true current custody and previous obligations. Do not use QUARANTINED to claim physical hub receipt of a missing piece. A separate missing-item finding may explain why a return remains outstanding (BD-09, LR-01).

Forbidden examples:

- Anonymous cart to HELD/RESERVED.
- OUT_FOR_DELIVERY, IN_PERSON_TRIAL or RETURNING_TO_HUB directly to AVAILABLE.
- Failed QC directly to AVAILABLE or failed/pending payment directly to SOLD.
- SOLD automatically to AVAILABLE, RESERVED or RETURNING_TO_HUB because a webhook failed, a refund occurred or a trial was cancelled.
- Any allocation in one market silently fulfilled from another market.
- Any transition that erases custody/movement history or creates two incompatible active commitments.

Financial reversal and physical return are independent. An approved returned piece might eventually be resold after QC, but that does not delete the original sale or guarantee return eligibility.

## PaymentAttempt: collection facts

Proposed states: `CREATED`, `PENDING`, `SUCCEEDED`, `FAILED`, `EXPIRED`, `CANCELLED`. Keep provider event history and separate reconciliation/exception status. Local checkout expiry and a verified external receipt may differ; the record must preserve both facts.

| From | To | Required trigger/guards | Open gates |
| --- | --- | --- | --- |
| CREATED | PENDING | Backend creates/exposes provider collection request for the immutable amount/currency/purchase version. QR and URL reference the same final PaymentAttempt through the shared context. | BUSINESS DECISION BD-01, BD-04, BD-05, BD-06, BD-15; CA REVIEW CA-01/CA-02. |
| CREATED or PENDING | SUCCEEDED | Authenticated backend verification establishes a successful receipt for the right provider account, reference, amount and currency; persist and deduplicate evidence. | Real-provider integration/account prerequisites; business allocation consequences remain BD-02/BD-05 and CA-01/CA-02. |
| CREATED or PENDING | FAILED | Verified provider failure or a narrowly defined local pre-creation failure; distinguish failure to contact provider from proof no charge occurred. | Provider contract; BUSINESS DECISION BD-05 for customer/agent response. |
| CREATED or PENDING | EXPIRED | Approved local/provider expiry; record which scope expired. Does not assert that a delayed payment is impossible. | BUSINESS DECISION BD-02, BD-05. |
| CREATED or PENDING | CANCELLED | Approved cancellation/supersession with provider status checked where necessary; maintain uncertainty and stale-link evidence. | BUSINESS DECISION BD-05. |
| FAILED, EXPIRED or CANCELLED | SUCCEEDED | Exceptional verified late/corrective success. Append evidence, flag obligation/reservation mismatch and reconcile under approved policy. Never automatically restore stock or undo cancellation. | BUSINESS DECISION BD-02, BD-05; CA REVIEW CA-01/CA-02. |

Same-state duplicate events are idempotent observations, not new collection or allocation. `SUCCEEDED` cannot be downgraded by a stale failure callback. A later refund has its own record and state machine. A disputed/unmatched or overpaid receipt remains a real receipt even if it cannot be allocated to the intended purchase.

A retry or changed selection may require a new attempt/context version, but the exact rule is BUSINESS DECISION BD-05. Preserve earlier attempts and check in-flight receipts before replacing the active collectible context. Two actual successful receipts must not both fulfill the same balance; preserve excess/unallocated evidence and invoke an approved resolution process, rather than hiding the second receipt or automatically refunding it.

Forbidden examples:

- Browser success, screenshot, agent claim or notification receipt directly to SUCCEEDED.
- Changing an issued attempt's amount to match later selections without a versioned, approved workflow.
- SUCCEEDED back to PENDING/FAILED because a webhook arrived out of order.
- A successful deposit automatically confirming a cancelled/expired booking or selling items.
- Per-box QR/link requests creating separate charges for the same booking-level final obligation.
- Failure of payment messaging silently changing payment or purchase state.

## DeliveryAssignment: responsibility and visit execution

Proposed states: `ASSIGNED`, `PREPARING`, `READY`, `DISPATCHED`, `ARRIVAL_VERIFIED`, `VISIT_CONCLUDED`, `RETURN_HANDOVER_PENDING`, `CLOSED`, `CANCELLED`, `SUPERSEDED`. Outcome and exception details are separate.

| From | To | Required trigger/guards | Open gates |
| --- | --- | --- | --- |
| ASSIGNED | PREPARING | Authorized staff begin preparation for the assigned combined visit. | BUSINESS DECISION BD-07, BD-08, BD-09. |
| PREPARING | READY | Exact manifest and packing checks complete; approved documents available. | BUSINESS DECISION BD-09; CA REVIEW CA-02; LEGAL REVIEW LR-03. |
| READY | DISPATCHED | Recorded transfer of all dispatched pieces to the authorized custodian; aligns with booking and unit dispatch records. | BUSINESS DECISION BD-07, BD-08, BD-09. |
| DISPATCHED | ARRIVAL_VERIFIED | Backend verifies the single Arrival OTP against the current authorized assignment; starts booking trial and relevant unit trial states once. | Approved OTP invariant; exceptional revisit process BUSINESS DECISION BD-07; resend/fallback settings BD-15. |
| ARRIVAL_VERIFIED | VISIT_CONCLUDED | Outcome, selection and piece-by-piece custody/disposition recorded under the approved payment/handover rule. | BUSINESS DECISION BD-05, BD-07, BD-09. |
| DISPATCHED | VISIT_CONCLUDED | Approved failed-visit/no-show outcome with recovery plan; no invented trial/arrival verification event. | BUSINESS DECISION BD-07, BD-09. |
| VISIT_CONCLUDED | RETURN_HANDOVER_PENDING | Unpurchased pieces require hub return; record agent responsibility and outstanding manifest. | BUSINESS DECISION BD-09. |
| RETURN_HANDOVER_PENDING | CLOSED | Hub acknowledges all required handovers; discrepancies are explicitly resolved/escalated under approved closure criteria. QC may continue separately. | BUSINESS DECISION BD-09; LEGAL REVIEW LR-01 for liability questions. |
| VISIT_CONCLUDED | CLOSED | No pieces require return and every custody/handover obligation is satisfied under approved sale/visit closure policy. | BUSINESS DECISION BD-05, BD-07, BD-09. |
| ASSIGNED, PREPARING or READY | CANCELLED | Booking/assignment cancellation permitted; packed stock disposition and commitments handled separately. | BUSINESS DECISION BD-02, BD-07, BD-09. |
| ASSIGNED, PREPARING or READY | SUPERSEDED | Authorized reassignment with new assignment and explicit transfer of any preparation responsibilities. Preserve old assignment. | BUSINESS DECISION BD-07, BD-09. |

Reassignment after dispatch must be designed as an explicit custody transfer and authorization change, not changing `agent_id` in place; exact behavior remains BUSINESS DECISION BD-07/BD-09. A failed notification does not prevent an already recorded dispatch from being true. Closing an assignment does not declare that every returned unit passed QC or that every settlement reached the bank.

Forbidden examples:

- Unassigned agent action or exposing another agent's customer details solely by changing the URL.
- DISPATCHED directly to CLOSED while pieces remain in agent custody.
- ARRIVAL_VERIFIED based only on an agent button without backend OTP verification.
- Cancellation/reassignment erasing prior dispatch or transferring custody without evidence.
- CLOSED automatically setting returned units AVAILABLE or marking financial reconciliation complete.

## Refund: request, authorization and repayment

Proposed states: `REQUESTED`, `APPROVED`, `SUBMITTING`, `PENDING`, `SUCCEEDED`, `REJECTED`, `FAILED`, `CANCELLED`. An authorization actor and permission are required; this document does not select a particular approval hierarchy, automatic refund policy or customer entitlement.

| From | To | Required trigger/guards | Open gates |
| --- | --- | --- | --- |
| REQUESTED | APPROVED | Authorized decision under approved deposit/after-sale policy, source receipt identified, amount/currency valid, and refund capacity reserved against other in-flight/succeeded refunds. | BUSINESS DECISION BD-01/BD-10; CA REVIEW CA-01/CA-02; LEGAL REVIEW LR-01. |
| REQUESTED | REJECTED | Authorized policy decision with reason and required customer communication evidence. | Same applicable policy/review gates. |
| REQUESTED or APPROVED | CANCELLED | Authorized withdrawal before submission, after confirming no provider refund may already be in progress. | Applicable policy and provider contract. |
| APPROVED | SUBMITTING | Persist idempotent refund instruction before contacting provider; no second independent instruction for a retry. | Provider integration and reviewed finance permissions. |
| SUBMITTING | PENDING | Provider accepted or submission outcome is uncertain; retain correlation for recovery. A timeout is not proof of failure. | Provider verification contract. |
| SUBMITTING or PENDING | SUCCEEDED | Verified backend/provider evidence confirms repayment; allocate result once. | CA REVIEW CA-01/CA-02 for financial presentation. |
| SUBMITTING or PENDING | FAILED | Definitive verified failure, preserving earlier attempts and evidence. | Approved recovery process. |
| FAILED | SUBMITTING | Approved retry only after checking prior provider outcome and refund capacity; retain retry history/idempotency semantics. | BUSINESS DECISION BD-05/BD-10 where applicable; CA REVIEW CA-02. |
| FAILED | SUCCEEDED | Exceptional verified late/corrective success; stop incompatible retry, preserve evidence and flag reconciliation mismatch. | Provider contract and CA REVIEW CA-02. |

If a supposedly cancelled/rejected request has provider repayment evidence, record the actual external transaction and raise a reconciliation exception; do not suppress a real repayment to maintain a preferred request status. Any corrective state transition must be explicit and attributable.

Forbidden examples:

- Refund success from a frontend callback, staff screenshot or notification delivery alone.
- Retrying after a timeout as a new refund before resolving whether the first was executed.
- Total succeeded plus reserved in-flight refunds exceeding the verified refundable source capacity.
- SUCCEEDED back to PENDING/FAILED on stale events.
- Refund success directly restoring a sold unit to AVAILABLE or mutating an issued invoice.
- Automatic deposit refund/retention or after-sale entitlement while BD-01/BD-10 and applicable reviews remain unresolved.

## Cross-machine guards and verification scenarios

| Scenario | Required result |
| --- | --- |
| Two customers request the last eligible piece | Only one incompatible commitment can succeed; the other sees authoritative unavailability. Anonymous cart entries confer no priority. |
| Reservation expiry races verified payment | Preserve actual payment and reservation facts; lock/recheck before any acceptance/allocation. Resolution is gated by BD-02, not timestamp guesswork or automatic stock resurrection. |
| Selection changes while payment is pending | Preserve old amount/version and pending evidence. Apply only the approved BD-05 rule; never silently reuse the old payable context for a new amount. |
| Same callback is received repeatedly/out of order | At most one business effect per verified event/obligation action; no duplicate purchase, invoice, refund or stock movement. Record real additional receipts distinctly. |
| One booking contains Ring and Chain boxes | One normal visit and shared payment presentation; selections may span boxes. No separate checkout is created merely from category boundaries. |
| No purchase after a verified trial | Record no-purchase outcome and return all relevant units through hub receipt and QC. Deposit treatment remains BD-01/CA-01/LR-01. |
| A paid purchase is refunded | Preserve the sale/invoice; refund state and physical return/QC progress independently under approved after-sale policy. |
| Agent returns only part of a manifest | Received pieces progress individually to QC; missing pieces remain explicit outstanding custody/discrepancy evidence. Do not mark all stock available or erase responsibility. |
| Gold or Hyderabad is activated in a test configuration | Existing state machines still enforce category, selected-market inventory, authoritative pricing and reviewed policy gates; activation alone does not authorize unresolved Gold or fulfillment rules. |

These become meaningful tests during authorized implementation, using approved policy fixtures or clearly marked development scenarios. They must not encode example deposits, seven-day cart retention, two-box limits, refund entitlements, handover rules or price locks as production answers. See [ARCHITECTURE.md](ARCHITECTURE.md), [DATA_MODEL.md](DATA_MODEL.md) and [TASKS.md](TASKS.md).
