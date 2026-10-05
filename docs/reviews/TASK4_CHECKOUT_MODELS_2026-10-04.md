# Task 4 Guest Checkout Model Extension

Date: 2026-10-04. Status: IMPLEMENTED CHECKOUT PERSISTENCE; TASK 4 OVERALL REMAINS IN PROGRESS.

Authority: the owner explicitly requested implementation of the discussed guest checkout/session, inventory and payment models within Task 4, synchronization of all necessary references, and completion of this change. D-27 records the approved flow. D-23/D-24 identity separation, permanent guest privacy, D-25 booking OTP and D-26 model-only scope remain binding. No launch or unresolved commercial/CA/legal approval is implied.

## Delivered Scope

Seventeen new concrete models extend the previous 27 to 44. The complete source and field inventory is [ALL_BACKEND_MODELS_REFERENCE.md](../ALL_BACKEND_MODELS_REFERENCE.md). Existing StaffUser/CustomerAccount and booking ownership/history remain intact.

| Module | New models |
| --- | --- |
| trials/checkout.py | GuestBrowserSession, GuestBookingRecoveryGrant, GuestRecoveryPhoneChallenge, TrialPlanRevision, TrialBoxItem, TrialItemAllocation, IdempotencyRecord |
| inventory/reservations.py | InventoryReservation |
| payments/models.py | BookingCharge, PaymentContext, PaymentAttempt, GatewayEvent, GatewayTransaction, PaymentAllocation |
| notifications/models.py | NotificationEvent, NotificationAttempt, OutboxCommand |

TrialBooking gains verification/acceptance/sealing/confirmation evidence and pre-visit statuses. TrialBox gains plan reference/snapshot. GuestBookingAccessGrant replaces its own token digest with browser_session and scope. Existing grant secrets are migrated, not discarded. Shared LifecycleModel freezes identity/snapshots and recorded outcomes; MinorUnitField refuses non-integer coercion even in bulk ORM preparation. New configuration contracts name plan/charge policy references without supplying values.

Catalogue stock observation now excludes active reservations, while still returning `bookable: false`. All guest, identity and financial records remain private; no indexable routes, prices, public media or schema.org claims were added. Frontend changes are documentation only and preserve pre-existing README/guide edits.

## Flow Mapping

The HTTP/worker actions below are the later implementation contract, **not currently registered endpoints or running jobs**.

| Action | Persistence |
| --- | --- |
| Establish guest browser continuity | GuestBrowserSession digest and validity; Task 6 sends raw secret through a protected cookie |
| Submit name/mobile/address and selections | TrialBooking DRAFT; TrialBox/TrialBoxItem; BookingPolicyAcceptance; QUOTED BookingCharge; browser-specific GuestBookingAccessGrant; scoped IdempotencyRecord |
| Send booking OTP | Unused BookingPhoneChallenge; NotificationEvent/Attempt and SEND_NOTIFICATION outbox intent |
| Verify OTP and finalize review | Consume that challenge; seal complete selection/acceptance graph; bind proof/deadline; booking VERIFIED; no stock reservation |
| Explicitly start payment | One transaction accepts quote, creates OPEN context, HELD reservations and exact-unit allocations, CREATING attempt and CREATE_PROVIDER_ORDER command; booking PAYMENT_PENDING. Shortage rolls back the whole operation |
| Provider order created | Assign immutable provider order ID; attempt PENDING; outbox outcome recorded |
| Receive authenticated provider evidence | Deduplicated GatewayEvent and immutable AUTHORISATION/CAPTURE GatewayTransaction facts; provider identity/account/environment checked by the future adapter |
| Begin capture | Complete selection required; context CAPTURING, reservations CAPTURE_PENDING, attempt CAPTURE_PENDING and CAPTURE_PAYMENT work |
| Verify and allocate capture | PaymentAllocation within receipt/charge limits; attempt SUCCEEDED, reservations COMMITTED, charge COLLECTED, context PAID; booking CONFIRMED with timestamp |
| Confirm/view booking | Confirmation notification intent plus authorized reads; no client callback alone can establish confirmation |
| Lose cookie | Booking-specific recovery grant and fresh GuestRecoveryPhoneChallenge; single-use redemption to a browser session; future service issues that booking's access grant |
| Unpaid expiry/uncertain outcome | Explicit context closure/reconciliation and reservation/allocation release; retain booking and financial evidence. Capture uncertainty cannot expire directly; late/extra receipts remain unallocated to closed contexts |

State storage does not authenticate callers or prove external events. Task 6/8 services must enforce current policy, permission, CSRF, phone proof, scoped idempotency and a consistent booking/charge/context/unit lock order. Provider calls run after commit via durable work. Unknown remote outcomes need fetch/reconciliation; UUID deduplication cannot alone guarantee remote exactly-once behavior.

## Integrity and Boundaries

- Guest/account mode and ownership, historical contacts, selection/plan snapshots, provider identities and financial evidence cannot be reassigned or silently rewritten through supported ORM or guarded SQL writes.
- Exactly one live reservation per physical unit is database-enforced. Quantity/variant/booking/city bindings and complete-selection checks protect payment progression. Held stock stays physically at HUB; its reservation determines availability.
- A captured receipt can be retained despite late arrival or mismatch. Applying it requires the correct currency/obligation and live capture context; locked aggregate checks prevent concurrent over-allocation. Authorization is not capture, capture is not revenue, and booking confirmation is not a sale/invoice/settlement.
- Identity/financial/history FKs use PROTECT, with SQL history guards and evidence-aware downgrade refusal. Privileged operators can disable triggers; runtime database privileges must be restricted. Retention guards do not decide lawful privacy retention.
- Booking, recovery and arrival OTPs are separate purpose-bound tables. Raw codes/credentials/card data are not routine stored payloads. Async OTP sending requires a short-lived protected payload reference and a reviewed purge mechanism in Task 9.
- After a terminal pre-visit cancellation/expiry, this checkpoint has no backward reset transition. Post-confirmation delivery/custody, retries/rebooking, amendments, refunds and commercial exceptions need their remaining reviewed models/workflows. No such policy is invented by this schema.

## Migration and Recovery

Apply the full graph, not a partial per-app deployment:

- trials 0003 adds the expanded schema; 0004 migrates each legacy guest digest into one browser session, makes the session reference required, removes the old grant digest and installs integrity guards.
- inventory 0005–0006 adds the reservation table and cross-app constraints.
- payments 0001 adds obligation, context/attempt and receipt/allocation tables.
- notifications 0001–0002 adds delivery/outbox tables and their cross-app references.

Existing bookings/grants keep primary/public IDs, contact facts, digest value and expiry/revocation history. Legacy draft boxes can retain null plans; advancing them requires complete reviewed input. The migration does not invent OTP verification, payment, physical movements, staff actors, commercial approvals or financial classification. It does not group grants by phone or create/link customer accounts.

The guard migration refuses rollback when checkout evidence exists, including legacy booking/grant rows. Empty-schema rollback and additive upgrades are exercised in tests. After use, retain data and choose a compatible application rollback or a reviewed forward correction. Old application code expecting grant.token_digest is not compatible with the final schema; coordinate deployment. Backup/restore and staging rehearsals remain release requirements.

Only isolated PostgreSQL test databases were migrated during this work. The persistent development database and any staging/production databases were not migrated. No migrations were deleted or historical applied migration files rewritten.

## Verification

- Full backend PostgreSQL suite at the complete-extension checkpoint: **391 passed**.
- Subsequent focused checkout/model/migration suite after adding reserved-stock projection and incomplete-selection checks: **23 passed**.
- Coverage includes guest/account confirmation paths, OTP-without-holds, invalid proof/ownership, session rotation and multiple grants, recovery purpose/single use, request scoping, exact monetary representation, concurrent reservations, concurrent over-allocation, duplicate provider evidence, late receipts, capture uncertainty, notification binding and lossless legacy migration/downgrade refusal.
- Final focused run after strict bulk monetary validation: **23 passed**. Ruff lint passed; formatting passed for 105 files; Django system check reported no issues; migration drift check reported no changes. The generated reference matched registered fields/source, and local Markdown file links resolved across all 15 root/reference files. Earlier full-suite interruption was a Windows pytest temporary-directory permission issue; the authorized rerun passed. No live provider, customer authentication, deployed security/VAPT, CI or launch readiness is claimed.

## Remaining Task 4

The requested guest-through-upfront-payment model extension is delivered. The broader matrix still requires customer contact/authentication/recovery and staff scope, additional consent/audit scope, scheduling/coverage review, catalogue alias/publication completion, dispatch/visit/return/QC and custody extensions, purchase/tax/invoice/adjustment models, refund/settlement/bank reconciliation, additional notification subjects and analytics evidence. See [Task 4](../TASKS.md#4-cross-app-model-completion) for ownership. These remain Task 4 persistence work, not a claim that later tasks can omit their models.

Timeout/OTP/session limits, charge amount/basis/classification, live capture capability, late-payment/refund handling, hub sourcing and reviewed legal/accounting treatment remain explicit gates. No sample duration, deposit, box count or tax rate is a production default.
