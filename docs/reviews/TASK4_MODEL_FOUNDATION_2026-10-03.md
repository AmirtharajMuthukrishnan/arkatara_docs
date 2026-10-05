# Task 4 Identity and Booking Model Foundation

Date: 2026-10-03. Status: PARTIAL IMPLEMENTATION, NOT TASK 4 COMPLETION.

Historical checkpoint: D-27's [2026-10-04 checkout extension](TASK4_CHECKOUT_MODELS_2026-10-04.md) supersedes the DRAFT-only booking and grant-owned credential structure described below. This record preserves its original scope/test evidence; it is not the current complete field reference.

Authority: the owner requested completion of Task 4, beginning with models whose decisions are clear, then instructed continuation. D-23 through D-26 and R-33 through R-38 remain binding. No open business, CA or legal decision is resolved by this implementation.

## Delivered Model Scope

| App | Actual model | Implemented responsibility |
| --- | --- | --- |
| accounts | CustomerAccount | Stable customer identity and lifecycle, separate from StaffUser/auth permissions; closure retains identity and references |
| compliance | PolicyDocumentRevision | Append-only policy version/language/content with computed SHA-256 |
| trials | TrialBooking | DRAFT intent, immutable guest/account mode and owner, market/contact/address snapshot, unique request UUID and fingerprint |
| trials | TrialBox | Protected booking/category container, without unapproved category uniqueness or quantity semantics |
| trials | BookingPhoneChallenge | Hashed booking-specific challenge, immutable phone binding, explicit expiry/attempt bounds and monotonic outcomes |
| trials | GuestBookingAccessGrant | Booking-specific guest credential digest and expiry/revocation, without any account relationship |
| trials | BookingPolicyAcceptance | Immutable booking/policy/time/request evidence, separate from marketing consent and customer identity |
| delivery | DeliveryAssignment | Protected staff actor/agent references, one open assignment per booking, retained end/reassignment evidence |
| delivery | ArrivalOTP | Separate assignment/booking-phone-bound challenge table; no purchase OTP |

The registered models are in each app's `models.py`, except `PolicyDocumentRevision`, which is imported from `compliance/policies.py`. Shared retained/append-only guards and challenge fields live in `common/evidence.py` and `common/verification.py`. No workflow signals or provider adapters were added.

Original recipient data is immutable at this stage, including draft intent. Corrected contact input requires new intent; accepted-booking amendments/versioning remain unfinished model work. DRAFT is deliberately the only currently representable booking state. This prevents the partial schema from claiming confirmed bookings without the missing policy, payment and stock evidence; it is not the final lifecycle.

CustomerAccount has no credential fields or `is_authenticated` property. It is not accepted by existing staff authentication. Later customer authentication must supply a reviewed customer principal, transport, recovery, revocation and permission boundary. There is no registration/login endpoint or customer checkout enabled today. Staff assignment storage does not grant agent permissions or require Django Admin access.

## Database and Migration Boundary

- New migrations: accounts 0002-0003; compliance 0003-0004; trials 0001-0002; delivery 0001-0003. Existing applied migrations are unchanged. Cross-app dependencies are explicit; the shared challenge trigger is installed before the arrival trigger and removed after it.
- ORM saves validate records; bulk mutation/deletion is refused for these retained models. PostgreSQL checks/triggers also protect ownership, snapshots, challenge binding/outcomes, guest-grant revocation, policy/acceptance/box history and assignment identity. A partial unique index resolves competing open assignments.
- Existing staff UUIDs, credentials, audit attribution and inventory identity survive additive upgrades. There is no guest-to-account backfill, new approval seed, invented OTP verification, payment or inventory movement.
- Empty development schemas can migrate back and forward. Once new identities/evidence exist, guard migrations refuse downgrade before removing their protection. Use reviewed forward recovery; do not drop triggers, rewrite applied history or reset databases to bypass this protection.
- Database owners can disable triggers or truncate tables. These guards are not protection from a privileged operator; production runtime privileges, backups, restoration, access control and monitoring remain release obligations. Personal-data retention/erasure under LR-04 still requires controlled design, not an assumption of perpetual retention.
- Migrations were exercised on disposable PostgreSQL test databases. They have **not** been applied to the persistent local foundation database or any staging/production database. Deployment must review the migration plan, take verified backups and run the integrity checks against the intended database before enabling dependent code.

## Verification

- Full backend regression suite on local PostgreSQL: **370 passed in 70.33 seconds**. This includes 36 new identity/booking model tests and five new identity/booking migration tests.
- `ruff check .`: passed. `ruff format --check .`: 96 files already formatted.
- `manage.py check --settings=config.settings.test`: no issues.
- `manage.py makemigrations --check --dry-run --settings=config.settings.test`: no changes detected, using the existing local PostgreSQL connection.
- Root and backend `git diff --check`: passed. No CI, deployed environment or provider integration was exercised in this continuation.

Targeted tests cover ORM and direct SQL bypasses, guest/account ownership, same-phone independence, historical contact preservation, policy/assignment evidence, challenge expiry/attempt/consumption boundaries, concurrent assignments/consumption, additive upgrades, empty rollback and refusal of evidence-removing rollback.

The existing Task 2 upgrade test now removes dependent trials/delivery migrations when targeting the older catalogue schema. Its original draft-unit preservation assertions remain; the change fixes the expanded dependency graph rather than weakening history checks. A full-suite Windows temporary-directory permission failure was retried with approved elevated execution.

## Remaining Task 4 Work

Every coverage-matrix row remains subject to full acceptance. This checkpoint must not be described as complete customer auth, complete Orders, financial readiness or live operations.

| Scope | Remaining engineering / decision dependency |
| --- | --- |
| 4.M1 | Contact/credential/session/recovery persistence and operational staff scope. Review authentication components and recycled-number recovery before fixing this shape. No second AUTH_USER_MODEL. |
| 4.M2 | Customer-purpose consent and typed audit attribution, privacy/retention boundaries and complete policy integration. Preserve guest unlinkability, including indirect evidence. LR-04 remains open. |
| 4.M3-4.M4 | Downstream scheduling/coverage review, public media identity and alias/redirect history. Existing reference/publication models are retained; open hub/slot policy is not selected. |
| 4.M5-4.M6 | Reservations/allocations, TrialPlan revisions, item/quantity semantics, accepted lifecycle and amendment/transition evidence. BD-01/BD-02/BD-03/BD-04/BD-08/BD-13 distinguish shape decisions from values that may stay unconfigured. The earlier question about physical-piece quantity versus per-piece rows is still unanswered. |
| 4.M7 | Visit/outcome evidence, dispatch manifests/documents and custody references. Preserve no-purchase separately from payment settlement and physical return/QC; BD-05/BD-07/BD-08/CA-02/LR-03 remain explicit. |
| 4.M8-4.M9 | Purchase/tax/invoice/adjustment persistence, payable contexts/attempts, receipts/allocations/refunds and settlement/bank reconciliation. No deposit basis, invoice numbering scope, tax treatment or retry/handover policy is inferred. |
| 4.M10-4.M11 | Durable notification work/attempts and minimized analytics evidence/projections. These are still pending engineering, not all blocked merely because operating settings remain open. |
| 4.M12 | Complete the full dependency/coverage review and corresponding migration, integrity and concurrency verification after remaining models are implemented. |

Missing policy values need not prevent independent schema work. A schema-shaping uncertainty must be resolved before the affected model is accepted; representing a configurable field alone does not approve a commercial rule. No whole-app checklist item is closed by this first foundation.

## Exposure and Operational Limits

No API, serializer, Admin editing surface, frontend route, search metadata or public projection was added. New records remain private persistence; later consumers must enforce explicit principal/action/ownership/assignment scope and no-index/private-cache controls. Tests here do not verify endpoint authorization that has not been implemented.

The challenge tables store evidence and bounded state, not proof of identity or legal responsibility. They do not generate/send/compare OTPs or trigger payment. Future consumers must lock and validate the live challenge, configured purpose, expiry and subject, then consume it atomically with the authorized business transition. A credential digest alone is not an implemented guest-access service.

No real contact/order data, approved security timeouts, payment policy, GST rate, stock hold, terms or production message is seeded. Fixtures are fictional. Existing frontend work and the external desktop model draft were left untouched.
