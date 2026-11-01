# Task 4 re-review validation and correction, 2026-10-10

The owner supplied Claude's re-review and authorized continuation on 2026-10-10. Its source title identifies the 2026-10-07 correction set; the report supplies no reviewed commit ID or execution date. The exact report is retained below. It independently reports 429 passing tests in 819 seconds and six scratch probes. These are **reviewer-reported results**, not a new Codex run. The original correction checkpoint remains [preserved](TASK4_FINDING_FIXES_2026-10-07.md). This cycle concerns the existing uncommitted application working tree; no independent signoff, merge, production migration or deployment is implied.

## Validation before correction

Canonical D-26/D-28 and R-14/R-16/R-25/R-28/R-30 remain authoritative. Source and the supplied probe code were inspected before correction. Original IDs/severity are retained; where the reviewer did not specify a severity, none is invented.

| Finding / Claude severity | Disposition and evidence / root cause | Correction and regression scope |
| --- | --- | --- |
| V-1 / HIGH | VALID. `HomeVisit.assignment` is immutable, its one normal visit cannot be replaced, and ended assignments prevent progression. Committed release requires dispatch-origin sale/QC evidence. Routine reassignment therefore strands stock. DATA_MODEL requires assignment history; BD-07 does not approve additional visits. | Keep one combined visit; attributable predispatch assignment supersession, never-dispatched closure/disposition and preserved physical/payment evidence. Test before/after READY, invalid/raw updates, hub-only release and history. Commercial booking cancellation/refund remains gated. |
| V-2 / MEDIUM | VALID structural gap; PARTIAL proposed numerical bound. Only opaque balance JSON explains upfront application, and no source-linked budget prevents repeated use. COMPLIANCE_AND_FINANCE requires collection/application/refund distinctions. Payable above jewellery value is not universally invalid while tax/adjustment treatment remains open. | Immutable attributed calculation basis, signed explicit adjustment and source-allocation applications; currency/booking/entity agreement, locked source limits and authoritative sealed balance. No automatic credit/refund policy. Test repeated/concurrent application, overdraw, mismatches, zero balance and sealed history. BD-01/CA-01/CA-02 remain open. |
| V-3 / MEDIUM | VALID lock-order and I/O boundary defects. `AuditedReferenceAdmin.get_object` takes a row lock before the advisory latch in `save_model`; checkout does the reverse. `storage.put` executes inside the latch-held transaction. Claimed runtime causation is unproved; current storage is LOCAL and S3 is a future risk. | Acquire latch before Admin row lock; stage unique private object before transaction, then reauthorize/reload/validate metadata and audit atomically, with safe cleanup. Add real Admin/checkout and blocked-storage regressions. Narrower locks require measurement/design, not a rushed rewrite. |
| V-4 / LOW | VALID. Visit state/outcome lacks attributable event history; original outcome is write-once and has no correction/supersession. DATA_MODEL requires both. | Append-only attributed visit evidence and outcome supersession; preserve original header outcome and physical/financial facts. Test actor/time/identity/chain boundaries, immutable originals, unchanged custody/financial facts and raw bypasses. |
| V-5 / INFO | DECISION REQUIRED. The protected-purchase predicate permits only current agent-held dispatched pieces; later successful money remains recordable, but post-departure fulfilment is unavailable. BD-05 has no approved answer. | Record a provisional capability gate and request the owner's policy. No automatic refund, retention, handover or post-departure acceptance. Independent fixes continue. |
| V-6 / LOW | VALID technical identity inconsistency. New plan/charge scopes use codes while existing configuration contracts use public UUIDs. Historical configuration is immutable. | Use public UUIDs for new plan/charge bindings, preserve identified legacy code-scoped records, reject reinterpretation and unsupported dimensions. Add ORM/SQL and migration history tests. |
| V-7 / INFO | PARTIAL readiness dependency, not proof of an invented tax rule. Every issued document needs explicit REVIEWED treatment; dispatch requires an issued dispatch record. Actual dispatch is blocked by CA-02/LR-03 readiness. | Make this fail-closed dependency explicit; preserve unconfigured treatment and separate movement from sale. Do not bypass CA/legal review. |
| N-1 / not supplied | PARTIAL. Task 6 ownership is recorded, but its numbered work/DoD can state audited activation more clearly. | Strengthen existing 6.6 and DoD/tests; retain ten tasks and all original IDs. No activation workflow built in model work. |
| N-6 / not supplied | VALID reproducibility/maintenance debt; raw file-line totals include historical/reverse definitions. Ignored generators are unsafe as an implicit authority. | Declare frozen migration SQL hand-maintained, add a maintained effective-function inspection tool and checks, and mark scratch generators obsolete. No blanket removal of integrity or unreviewed trigger framework. |
| S-1 / not supplied | PARTIAL. Reviewer says recycled-number checks were never recorded, but TASKS already names them in customer activation gates. Verification ownership can be clearer. | Add explicit future recycled-number recovery/non-disclosure test requirements to existing 10.5/10.10. Recovery workflows remain later work. Retain booking-secret + fresh proof and LR-04 gate. |

Claude reports F-1/F-2/F-3/F-5/F-6/F-11/F-12 fixed, F-7 still fixed, F-4/F-8/F-10/F-13 partially fixed, F-9 largely fixed and F-14 structurally fixed. N-2 remains partial, N-3/N-8/N-10 deferred with owners, N-4 closed by D-22, N-5 fixed, N-7/N-9 unchanged decisions. These dispositions are review evidence, not a claim that the whole Task 4 matrix is complete.

## Correction, migrations and checks

The correction pass is complete and locally verified for the validated findings within D-26's model/integrity scope. Earlier applied/frozen migrations remain unchanged; new corrections are additive. No commit, push, merge or deployment was performed. This is not completion of the whole Task 4 coverage matrix or independent review signoff.

- **Visit continuity and history:** `delivery/visits.py` retains one combined `HomeVisit`, adds append-only `VisitEvidence`, attributed predispatch assignment supersession and original-outcome-preserving corrections. `CLOSED_BEFORE_DISPATCH` and `NEVER_DISPATCHED` permit physical hub-stock closure without pretending dispatch or refund occurred. Post-dispatch transfer, revisits and commercial outcomes remain gated.
- **Authoritative balances:** `billing/models.py` adds `PurchaseBalanceEvidence` and `UpfrontApplication`; `common/fields.py` rejects inexact signed monetary inputs. Sealing proves payable equals explicit basis less exact source-linked applications. Locked aggregate limits prevent repeated source use, including after a purchase is voided. Jewellery selling value stays separate. Zero final balances keep a protected financial/document obligation without provider collection; internal collection and purchase confirmation must commit together. The interrupted-zero-collection regression proves rollback permits voiding and return/QC/physical release while preserving collected source funds.
- **Lock and storage boundaries:** `common/admin.py` acquires the integrity latch before Admin row locks. The storefront hierarchy helper takes the integrity latch before its tree latch. Private object storage is staged before the metadata transaction; `catalog/media_services.py` reloads and reauthorizes current actor/product/source, commits metadata and audit together, and removes only the newly staged unreferenced object after failure. The global latch remains coarse; no throughput claim or production S3 integration is implied.
- **Identity and maintenance:** new plan/charge configuration bindings use public UUIDs, while explicit version-1 records preserve historic code scopes. Unsupported dimensions and new version-1 reinterpretation are rejected in Python/SQL. `scripts/inspect_integrity_guards.py` inspects effective frozen forward literals, excludes reverse SQL and compares bodies with PostgreSQL. SQL is hand-maintained; ignored scratch generators were marked obsolete. `scripts/export_model_reference.py` regenerates the canonical annotated snapshot without querying a database.
- **Task ownership:** existing 6.6 acceptance explicitly owns audited configuration activation; existing 10.5/10.10 own recycled-number recovery and non-disclosure verification. All ten tasks and original numbered IDs remain. No activation or recovery workflow was added by this correction pass.

### Migration and recovery boundaries

New schema migrations are `delivery/0005_visit_evidence_and_predispatch_closure`, `billing/0003_upfront_application_balance_evidence`, `payments/0005_charge_scope_identity_version` and `trials/0009_plan_scope_identity_version`. New frozen guard migrations are `trials/0008_visit_corrections`, `0010_purchase_balance_integrity` and `0011_atomic_source_funded_confirmation`. Use Django's dependency graph; do not apply SQL fragments independently.

Use a maintenance window with writers disabled and a verified backup. After the billing evidence schema is installed, `python scripts/inspect_integrity_guards.py --check-pending-balances` detects legacy SEALED selections without authoritative balance evidence. Migration 0010 also refuses these pending selections before installing the new guards. Resolve them under the prior compatible implementation or use separately reviewed forward recovery; no opaque historical JSON is automatically converted into monetary evidence. Completed/void historical purchases remain preserved. Migration 0011 refuses pre-existing zero-balance collections whose purchase is unconfirmed. These are rollout gates, not claims of an automatic historical recovery workflow.

Evidence-bearing downgrade guards refuse loss of visit, balance/application, UUID-scope and zero-obligation evidence. Prefer compatible code rollback or reviewed forward recovery after records exist. Historical applied migrations were not rewritten.

### Local verification

All tests use disposable PostgreSQL 18 databases; no production database was migrated. The following are actual local results:

- The broad targeted correction set passed **148 tests in 79.64 seconds**, including the zero-balance interruption case. After the final two additions, the final lock file passed **4 tests in 20.05 seconds** and the final migration file passed **3 tests in 22.73 seconds**. The latter proves a real legacy SEALED selection blocks both the diagnostic and migration 0010 without rewriting its facts, then a fixture-only VOID resolution permits upgrade. It does not implement a business recovery/refund procedure.
- `python -m ruff check .` passed; `python -m ruff format --check .` reported **118 files already formatted**.
- `manage.py check` reported no issues; `manage.py makemigrations --check --dry-run` reported no changes.
- Model-reference regeneration and `--check` passed. Canonical business reference IDs, local document links/anchors, all **95 original numbered IDs**, all **ten tasks** and the verbatim retained Claude attachment were checked. `git diff --check` passed in all three repositories.
- The effective-function inspector reads **41 forward function bodies**; migration tests verify their exact match with fresh and upgraded PostgreSQL. Reverse SQL is not used as a forward definition.
- The final full suite passed **472 tests in 493.92 seconds (8 minutes 13 seconds)**, exit 0, on the frozen final application/migration/test tree. It includes all final lock, legacy-upgrade and zero-balance interruption regressions. The earlier full run was intentionally interrupted for the zero-balance correction and is not reported as a pass.

The full-suite command from the workspace root is:

```powershell
arkatara_backend\.venv\Scripts\python.exe .tools\run_backend_checks.py -m pytest -x -q --tb=short
```

The local wrapper only supplies private loopback PostgreSQL credentials, test settings and unique disposable database/temp paths. The repository-native command is `python -m pytest -x -q --tb=short` with equivalent test environment configuration. Credentials were not printed or committed. Targeted commands use the same wrapper/flags with the nine correction files, then `tests/test_lock_boundaries.py` and `tests/test_rereview_migrations.py` respectively. The local review PostgreSQL server was stopped successfully after verification; its data was preserved.

### Re-review handoff and retained gates

Claude should independently review this final working tree against the retained original report, emphasizing V-1 reassignment/closure, V-2 source reuse and zero-balance interruption, V-3 both lock hierarchies/storage boundaries, V-4 append-only corrections, V-6 legacy UUID migration and migration preflights/recovery. Codex's internal audits and local tests do not constitute independent Claude signoff. Re-review of these new corrections and CI on the eventual exact commit remain pending under D-28.

BD-01/BD-02/BD-05/BD-07/BD-09, applicable CA/LR review, live activation/storage/provider/security work and the rest of Task 4's cross-app coverage remain explicit dependencies. The owner has not supplied BD-05 post-departure payment/handover policy. READY/dispatch still depends on reviewed issued movement evidence under CA-02/LR-03. No business, tax, refund, late-payment or revisit default was invented.

**D-20/D-21 assessment:** these changes add no public routes or presentation. Existing server-rendered catalogue content, stable URLs, visible metadata and private/non-indexable staff/financial boundaries remain unchanged; UI acceptance remains pending the owner's approved design.

## Original independent report

```text
Written for: you (the owner) and Codex, which will act on the corrections.

# Re-review of the Task 4 corrections (2026-10-07)

**Verdict:** both HIGH findings are fixed, and I proved it against a real database. Most other findings are fixed or properly assigned to later tasks. But the corrections introduced a new HIGH problem: reassigning a delivery agent before dispatch permanently strands the booking and its jewellery. They also introduced a deadlock path. And the code still has no record of how the upfront payment is applied to the final purchase amount. I'd fix the first two before committing this as "corrections complete".

## How I verified

- I read the fix report, every changed model, and the trigger functions that are actually in effect after migrations `trials/0005`–`0007`. Each migration redefines earlier functions, so the effective versions are spread across all three. I also read the new tests and the TASKS.md and DATA_MODEL.md updates.
- **Full backend suite: 429 passed** in 819 s on a disposable PostgreSQL test database. That matches the fix report's count.
- Ruff lint and format, `manage.py check`, `makemigrations --check` and `git diff --check` all pass in all three repos.
- **Probes:** I ran six of my own tests from my scratchpad, outside the repo. Two confirm fixes; four confirm the problems below.
- I started the local database for these runs and stopped it afterwards. No repository file was changed.

## Status of each finding

| ID | Status | Evidence |
|---|---|---|
| F-1 Booking closed during capture | **FIXED** | Probe: cancelling or expiring the booking, or voiding the charge, while capture is running is now rejected. The booking guard is at [0007:90-98](arkatara_backend/trials/migrations/0007_postcapture_integrity.py#L90-L98). Marking paid, committing stock and allocating money all now require the booking to be awaiting payment. |
| F-2 Captured money with no exit | **FIXED** | Probe: a capture arriving after the payment window closed now resolves cleanly. An attributed resolution closes the window, the hold is released, the charge voided and the booking expired. The same unit can then be reserved again. Whether that money is refunded or retained is still an open business decision (BD-02/BD-05, CA-01). |
| F-3 Unmatched receipts | **FIXED** | `GatewayTransaction.attempt` is now optional, and a later match is a separate attributed record. Provider account, environment and currency must agree. |
| F-4 Nothing after confirmation | **PARTIALLY FIXED** | The normal flow now exists and is guarded: dispatch → verified trial → sale, or return → hub receipt → QC → release. SOLD stock is frozen. Non-standard paths are stuck; see V-1. |
| F-5 Inactive catalogue/hub references | **FIXED** | Holds and commitments require an active market, hub, category, product, variant, material and purity ([0005:22-29](arkatara_backend/trials/migrations/0005_review_integrity.py#L22-L29)). |
| F-6 Unattributed reconciliation | **FIXED** | Closing a payment window or releasing a hold needs a linked resolution. That resolution has a staff member or named system as its actor, and its totals must match the current receipts. |
| F-7 Lint | Still fixed | Ruff passes. |
| F-8 Documentation | **PARTIALLY FIXED** | D-27 is now committed in the docs repo, so my approval concern is closed. The TASKS.md map and model coverage are updated. Remaining drift is listed under V-2 and S-1. The backend work is still entirely uncommitted. |
| F-9 Tests | **LARGELY FIXED** | About 36 new tests, and they check the failure cases, not just the normal path. Gaps are listed in actions 1–3 below. |
| F-10 Lock ordering | **PARTIALLY FIXED, new risk** | The trigger lock-order problem is gone, solved with one global lock; see V-3. |
| F-11 Policy identity | **FIXED** | Checked in both Python and SQL. A minor naming inconsistency remains (V-6). |
| F-12 Grant to a revoked session | **FIXED** | Checked in Python and SQL ([0005:166](arkatara_backend/trials/migrations/0005_review_integrity.py#L166)). |
| F-13 Tax and document records | **PARTIALLY FIXED (structure)** | Tax evidence is explicitly "unconfigured" until reviewed, never zero. Issued documents are sealed. A CA REVIEW is still needed for the actual treatment. |
| F-14 Final payment model | **FIXED structurally** | A payment window or allocation now targets exactly one obligation: the upfront charge or the purchase. The purchase payment path is tested. See V-2. |
| N-1 Activation workflow | **Owner recorded; not built** | [TASKS.md:594](docs/TASKS.md#L594) assigns it to Task 6, but it's not a numbered Task 6 item. It's still a launch blocker. |
| N-2 Staff access | **PARTIALLY FIXED** | Staff and group admin are now superuser-only and audited (that part is fixed). Staff scope is stored but not enforced. MFA is assigned to Task 10. |
| N-3 Proxy/TLS settings, N-8 security headers, N-10 OTP rate limits | Deferred, owners recorded | [TASKS.md:594](docs/TASKS.md#L594). |
| N-4 Server-rendered vs static frontend | Closed by decision | D-22 kept. |
| N-5 Stray diagram | **FIXED** | Deleted from both repos; the deletion is not yet committed. |
| N-6 Complexity | **Worsened** | Trigger SQL grew from about 600 to about 2,800 lines. The generator scripts that produce it (`.tools/build_*_guards.py`) aren't in any repository, and one of them already produced a bug. |
| N-7, N-9 | Unchanged decisions | No default was invented, which is correct. |
| S-1 Recycled phone numbers | Deferred, but not recorded | TASKS.md never mentions recycled-number verification. |

## New issues found

### V-1 · HIGH · Reassigning an agent before dispatch strands the booking and its stock (confirmed)

- **Evidence:**
  - `HomeVisit` is a one-to-one field on the booking, and its `assignment` can never change ([delivery/visits.py:19-22, 50-58](arkatara_backend/delivery/visits.py#L19-L58)).
  - The visit guard refuses to advance a visit whose assignment has ended ([0006:172](arkatara_backend/trials/migrations/0006_lifecycle_financial_integrity.py#L172)).
  - Committed stock can only be released through a custody chain that starts with dispatch.
  - A CONFIRMED booking has no allowed transitions.
- **Probe:** after a routine pre-dispatch reassignment, all six ways out are rejected:
  1. advancing the visit;
  2. pointing the visit at the new assignment;
  3. creating a second visit;
  4. dispatching the piece;
  5. releasing the hold;
  6. cancelling the booking.

  The piece stays committed and unsellable permanently.
- **Why it matters:** agent reassignment is ordinary operations, and DATA_MODEL explicitly anticipates it. The same dead end applies to any pre-dispatch problem. This is the original F-4 "frozen stock" problem, now limited to non-standard paths.
- **Fix:**
  - Let the visit follow the booking's current open assignment.
  - Replace the one-to-one field with "at most one open visit per booking".
  - Allow a committed hold that never left the hub to be released with attributed evidence. The commercial cancellation policy can stay gated; the physical unwind can't.
  - Add tests for reassignment before READY and after READY.
- **When:** before Task 7.

### V-2 · MEDIUM · How the upfront payment reduces the purchase amount isn't recorded (confirmed)

- **Evidence:**
  - `Purchase.payable_minor` is unconstrained apart from being positive.
  - The upfront credit exists only as JSON in `balance_snapshot` ([billing/models.py:66-68](arkatara_backend/billing/models.py#L66-L68)).
- **Probe:** the database accepts a payable amount above the jewellery value. It also accepts two purchases on the same booking that each claim the same 49-unit credit.
- **Why it matters:** this contradicts DATA_MODEL ([line 238](docs/DATA_MODEL.md#L238)) and COMPLIANCE_AND_FINANCE: "prevent the same funds from being applied or refunded twice".
- **Fix:** add an application record (upfront allocation → purchase, amount, currency) with locked limits, and bound `payable_minor` by it.
- **When:** before Task 7/8. Whether the upfront money is applied at all is still BD-01/CA-01, but the record has to exist either way.

### V-3 · MEDIUM · The global lock creates new problems

- **What it is:** [common/serialization.py](arkatara_backend/common/serialization.py) takes one global PostgreSQL lock. Every model save and every statement on about 40 tables waits on it, so all writes in the system now run one at a time. That includes Admin, CSV import, media, checkout and staff admin.
- **Problems:**
  1. **Deadlock (confirmed, error 40P01).** Admin edits of catalogue and market records lock the row first ([common/admin.py:54-63](arkatara_backend/common/admin.py#L54-L63)) and the global lock second. A checkout write that holds the global lock and needs that row then deadlocks with the edit. The fix was meant to remove exactly this kind of deadlock.
  2. **Media upload blocks everything.** The upload holds the global lock while writing the file ([media_services.py:223-239](arkatara_backend/catalog/media_services.py#L223-L239)). With S3, one upload would block every checkout and webhook write.
  3. **The suite now takes about 14 minutes instead of about 3.5.**
- **Fix:**
  - Take the global lock before row locks in the Admin path.
  - Move file writes outside the lock.
  - Add a deadlock regression test.
  - Plan narrower per-booking locks before scaling.

### V-4 · LOW · Visits have no recorded actor and no way to correct an outcome

DATA_MODEL ([line 222](docs/DATA_MODEL.md#L222)) asks for both. A wrongly recorded outcome can't be corrected, because the outcome field can only be written once.

### V-5 · INFO · BUSINESS DECISION REQUIRED (BD-05)

The schema only lets a sale complete while the piece is still with the agent at the visit. A payment that completes after the agent leaves can't become a sale; it can only be rejected and the money retained for a refund. That may be the right policy, but it decides an open question, so please confirm it.

### V-6 · LOW · Inconsistent scope identifiers

Plan and charge policy scopes use market and category codes. Earlier configuration keys use public UUIDs.

### V-7 · INFO · Dispatch now requires a CA-reviewed document

No visit can be dispatched until an issued dispatch document with REVIEWED tax evidence exists. That makes the outcome of CA-02/LR-03 a hard prerequisite for the first dispatch.

## Re-review checks

- **Root causes:** F-1 and F-2 are fixed at the database layer, not just patched in tests.
- **Weakened tests:** none. The earlier assertions remain; the fixtures now activate references and use attributed reconciliation.
- **Weakened invariants:** none found. One unit still allows only one live hold, and allocation limits still hold.
- **New coupling:**
  - Every write path now depends on one lock.
  - Delivery depends on billing: dispatch needs an issued document.
- **Migrations:** they only add schema, the guards still refuse destructive downgrades, and both fresh setup and the upgrade tests pass. No existing data was rewritten.
- **Security boundary:** staff admin is tighter, and no new endpoints were added.

## What Codex should do next

1. **Fix V-1:** make the visit follow reassignment, allow at most one open visit per booking, and add the never-dispatched release path. Add tests.
2. **Fix V-3:** correct the lock order in Admin, move file writes outside the lock, and add a deadlock test.
3. **Design V-2:** add the upfront-application record and bound the payable amount, then get the BD-01/CA-01 answer.
4. **Commit the guard generators** to the backend repo (or declare the SQL hand-maintained), and freeze further trigger growth (N-6).
5. **Update TASKS.md:** make N-1 a numbered Task 6 item in its definition of done, and add the recycled-number check (S-1) to Task 10.
6. **Get your answer on V-5** and record it in DECISIONS.md.
7. **Commit and run CI:** commit the backend and docs on feature branches. The backend correction set is still entirely uncommitted.

## Decision

| | |
|---|---|
| Launch confidence | NOT READY (expected at this stage) |
| Architectural confidence | ACCEPTABLE WITH DEBT |
| Overengineering risk | **HIGH**, in the database-guard layer, up from MODERATE |
| Business integrity risk | MODERATE. F-1 and F-2 are fixed; V-1 is serious but no live operation uses it yet. |

**Would I build the next stage on this? YES, WITH CONDITIONS:** fix V-1 and the V-3 lock order first, then commit and run CI.

My probes are in `C:\Users\91956\AppData\Local\Temp\claude\c--Users-91956-Desktop-ARKA-TARA\595d1d27-1914-4db5-ae77-fd81de4ca42e\scratchpad\review_probes_v2\test_reverify_probes.py`. The V-1, V-2 and V-3 probes can become regression tests once those fixes land.
```
