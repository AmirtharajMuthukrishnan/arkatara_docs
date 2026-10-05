# Customer login and guest booking readiness review

Reviewed: 2026-10-02.

Implementation follow-up, 2026-10-04: D-27 adds guest browser sessions/scoped grants, separate recovery proof, selections/reservations and upfront payment evidence. See [current checkpoint](TASK4_CHECKOUT_MODELS_2026-10-04.md). The reviewed commit and observations below remain historical; customer authentication is still not implemented or enabled.

Decision alignment updated: 2026-10-03. **Superseded recommendations:** the original review proposed reusing StaffUser for customers and optionally claiming guest orders. Those proposals were not implemented and are replaced by [D-23–D-26](../DECISIONS.md#d-23--separate-staff-and-customer-identities-with-permanent-guest-checkout): separate CustomerAccount, permanently unlinked guest bookings, booking phone OTP before upfront payment, and all agreed persistence in Task 4. Evidence and test results below describe the original reviewed commit, not verification of these new requirements. The guidance below is aligned with the selected direction; the supersession record preserves the earlier recommendation history.

Implementation follow-up, 2026-10-03: the first CustomerAccount/draft booking/OTP evidence/guest grant/policy acceptance/assignment models and their database protections are now coded and tested. See the [separate foundation record](TASK4_MODEL_FOUNDATION_2026-10-03.md). This dated review is preserved as an earlier assessment; neither its old test evidence nor the new model tests establish customer authentication or full Task 4 completion.

Scope: assess the reviewed backend/database and frontend API boundary for optional customer authentication after guest launch. No application source, migration or application data was changed by this review or its decision-alignment update. Guest checkout remains permanent under R-04/R-34. Current scope and approval come from DECISIONS.md and TASKS.md, not from historical review observations.

Backend commit reviewed: `d8309fa923955f455b725110f19d7587fb8892ee`; application worktrees were clean. Existing root architecture/decision documentation changes were preserved.

## Assessment

Adding customer login later does not inherently require replacing the backend/database. Customer identity, booking ownership and phone verification were not implemented at the reviewed commit, so it was not ready for a one-switch conversion. Historical guest claims are now expressly excluded, not a missing future feature.

At the reviewed commit there was no implemented order/booking pointing to a guest contact instance. The fields existed in the roadmap, not a submission API or booking table. D-26 now moves the first booking and customer migrations into Task 4; Task 6 retains workflow implementation.

Selected direction: retain booking-specific contact/address snapshots and an immutable checkout mode with a protected nullable FK specifically to CustomerAccount, not AUTH_USER_MODEL. Guest ownership remains null permanently. Account-mode bookings derive ownership from trusted customer authentication at creation; later signup never changes earlier guest bookings.

This prepares the database relationship. Login, authorization and trustworthy historical ownership still require implementation and tests. Their effort must not be confused with the small size of the schema change.

## Evidence from the current system

| Area | Observed implementation | Consequence |
| --- | --- | --- |
| Authentication identity | `accounts/models.py:7` defines `StaffUser(AbstractUser)` with a public UUID. `config/settings/base.py:81` selects it as AUTH_USER_MODEL. | Keep it for staff; D-23 adds CustomerAccount outside this auth model, with a separate authentication contract. |
| Identity schema | `accounts/migrations/0001_initial.py` includes unique username, password hash, names, email and privilege flags. `is_staff` and `is_superuser` default to false. No phone, verified-phone, customer-profile or address fields exist. | These are staff foundations, not the selected customer schema. Customer identity/contact/recovery persistence belongs to 4.M1. |
| Sessions | `config/settings/base.py:16,42,45,114` installs sessions, session/auth middleware and DRF SessionAuthentication. Runtime SESSION_ENGINE is `django.contrib.sessions.backends.db`. | Session infrastructure is present, primarily supporting Admin. Customer-facing login/logout/me flows are absent. |
| Database | Both inspected local databases contain 27 public-schema tables, including `accounts_staffuser` and `django_session`. | Neither contains customer, trial-booking, delivery, purchase, payment or invoice tables. This is local evidence, not a production inspection. |
| Registered models | Django's runtime registry contains 18 application models. `trials`, `delivery`, `billing`, `payments` and `notifications` have no registered models or numbered migrations. | Their packages do not establish implemented transactional workflows. |
| API | `config/api_urls.py:9` registers catalogue list/detail, health, reference data and serviceability. | There is no booking submission, customer history, customer authentication, or guest claim endpoint. |
| Booking design at review time | The then-current DATA_MODEL.md proposed contact/address snapshots without mandatory login; TASKS.md marked Task 6 planned. | D-24 now specifies immutable checkout ownership; D-26 assigns persistence to Task 4. Original document line numbers are no longer current after the scope update. |
| Financial foundation | `billing/snapshots.py:103` supplies HistoricalPriceSnapshot as a value object. | It contains catalogue/price/tax evidence, not persisted customer, purchase or invoice records. Future financial workflow independence cannot yet be validated end to end. |
| Frontend transport | `src/lib/api.ts:120` defaults to no-store and credentials omitted; caller options can override these defaults. | Authenticated browser requests need an explicit transport/CSRF contract. The helper does not permanently prohibit credentials. |

Paths without a repository prefix in this table are backend paths, except the explicitly named frontend transport.

## Findings, ordered by migration and access risk

### 1. Replacing AUTH_USER_MODEL later would create avoidable migration work

The existing user table is referenced by four application foreign keys:

- `BusinessConfiguration.approval_recorded_by`, in `compliance/models.py:87`.
- `AuditEvent.actor`, in `compliance/models.py:166`.
- `CSVImportBatch.actor`, in `catalog/import_batches.py:20`.
- `InventoryMovement.actor`, in `inventory/models.py:155`.

All four use `settings.AUTH_USER_MODEL` and PROTECT. Django Admin LogEntry also references the user model, and group/permission many-to-many tables already exist. Session authentication retains user/backend identity in session data.

Using `settings.AUTH_USER_MODEL` is the correct reference pattern, but it does not make changing the setting after migrations automatic. Replacing StaffUser with an unrelated CustomerUser would require reviewing foreign keys, identity values, permissions/content types, migration dependencies and existing sessions. Some actor evidence is additionally protected by append-only PostgreSQL triggers; repointing historical actors is not an ordinary data cleanup.

Preserve the current auth table and staff primary keys. D-23 selects a separate CustomerAccount instead of the original review's shared-auth/profile recommendation. This is not a second AUTH_USER_MODEL. Staff permissions and historical actors remain staff-specific; customer authentication must load and authorize a customer principal through the selected reviewed components.

Django documents the migration complexity of changing AUTH_USER_MODEL after tables exist and supports related profile models. [Django 5.2 authentication customization](https://docs.djangoproject.com/en/5.2/topics/auth/customizing/#changing-to-a-custom-user-model-mid-project).

### 2. Authentication alone is not the staff authorization boundary

`config/settings/base.py:117` defaults to IsAuthenticated. This checks login status, not staff membership or booking ownership. The adjacent comment calls this a staff baseline, which would be insufficient once customers can authenticate.

A runtime probe using an unsaved, non-staff StaffUser and the existing test-only ProtectedExampleView returned HTTP 200. The same identity failed Django Admin access and raised PermissionDenied in the inventory service. This demonstrates a future authorization risk, not an exposed private production endpoint: the example view is test-only, and the current API routes are public reads.

The implemented private workflows already check more than authentication:

- `inventory/services.py:21` checks active status, staff status and the action permission.
- `catalog/imports.py:36` checks active staff and required import permissions.
- `catalog/media_services.py:187` checks active staff and media/product permissions.
- `catalog/storefront.py:26` checks active staff and publication permission.

Before enabling customers, customer endpoints must check ownership; staff endpoints must check staff/action/assignment scope. Customer creation must never accept staff flags, groups, arbitrary user IDs or privilege fields from a public payload. Admin user-management access also needs review once staff and customer identities coexist.

### 3. Booking ownership and contact history need separate fields

An editable account profile is not a historical delivery address. Updating a customer's name, phone or saved address must not retrospectively alter where a past booking was sent or what an issued invoice recorded.

The conceptual data model already calls for snapshots. Preserve that boundary in the first booking implementation. A session is temporary and must never become an order's foreign-key target. Logging out, session expiry or phone changes must not sever business identity.

Changing a guest-contact FK to an auth-table FK later is unnecessary. Use booking snapshots plus nullable CustomerAccount ownership and immutable mode from the beginning. That optional relationship supports account-mode bookings only; a guest row can never acquire it later. Account closure preserves required evidence without converting the booking to guest mode.

### 4. Shared audit attribution currently assumes every user actor is staff

`compliance/models.py:200` unconditionally sets `actor_label` to `staff:<public_id>` whenever actor_id is present. Passing a customer user to AuditEvent would therefore incorrectly identify that action as staff activity. Supplying a different actor_label to create() would not help, because save() overwrites it.

The current behavior matches the implemented privileged workflows. Before recording customer actions in this shared audit model, extend the attribution contract to distinguish staff, customer and named system activity. Record trusted action context rather than accepting a client-supplied actor kind. Preserve old immutable events as originally recorded.

`inventory/models.py:195` also uses a staff label. That remains correct while its services continue to require staff authorization; customer login is not a reason to turn stock operations into customer actions. This is a bounded audit extension, not a reason to replace inventory history.

### 5. Phone matching cannot safely establish historical ownership

There is no phone verification, customer login challenge, guest booking access token, ownership-claim evidence or account-linking workflow in the current source. The future Arrival OTP is a visit authorization challenge, not a customer login or historical-ownership credential.

A typed mobile number is unverified contact data. Even later OTP verification proves current control, not ownership of every historical booking entered with that number. Shared numbers, typing mistakes, reassigned numbers and changed contacts make an automatic bulk phone-match claim unsafe.

Historical guest claims are prohibited by D-24, even if a user later authenticates. Booking-specific guest credentials/recovery can permit access to that booking without associating it with a customer account. A public UUID, name, address or phone match alone is insufficient. Arrival OTP is not a reusable login token. Registered OTP login also needs a recovery design that protects a previous number-holder's history; uniqueness of active phone data does not solve recycling.

Phone numbers should be normalized strings, not integer keys. Booking contact numbers must not be globally unique. A future verified-login-identifier uniqueness constraint belongs to the authentication contract, not all guest submissions.

### 6. Customer sessions require a bounded HTTP integration change

The observations below describe existing Django staff sessions. D-23 does not select this machinery or cookies for CustomerAccount automatically. Customer transport, authentication components and recovery must be reviewed explicitly; apply CSRF/CORS guidance where relevant to the selected mechanism and actual staff/public hosts.

The backend already supports database sessions and production Secure cookies. Runtime HttpOnly is true and SameSite is Lax. CORS_ALLOW_CREDENTIALS is currently false by default; CORS_ALLOWED_ORIGINS and CSRF_TRUSTED_ORIGINS alone do not enable credentialed cross-origin requests.

The frontend helper defaults to omitted credentials. Add an explicit authenticated request path with the chosen cookie transport and CSRF handling. With separate frontend/API origins, test credentialed CORS and the actual cookie topology; a same-origin proxy is another deployment option. Server-rendered private requests need deliberate credential forwarding and private caching behavior.

Login endpoints need CSRF protection as well as login abuse controls. DRF's default session authentication only enforces CSRF on already authenticated requests, so relying on that alone for login is insufficient. [DRF SessionAuthentication guidance](https://www.django-rest-framework.org/api-guide/authentication/#sessionauthentication).

Keep the current anonymous catalogue projections anonymous. `catalog/storefront_api.py:97,105` explicitly disables authentication for public reads. Future account/booking pages must have ownership checks, private cache behavior and the existing D-20 non-indexing treatment. Public catalogue cache keys must not accidentally include private account responses.

## Recommended first booking schema

Illustrative fields aligned with D-24 below describe the Task 4 target, not applied code. The previous sample pointing to AUTH_USER_MODEL is superseded:

```text
TrialBooking
  id / public_reference       stable booking identity
  checkout_mode               immutable GUEST or ACCOUNT
  customer_account_id         protected nullable FK -> CustomerAccount
  contact_name_snapshot       name supplied for this booking
  contact_mobile_snapshot     normalized contact string
  address_snapshot            validated address facts for this booking
  market_id                   selected Market
  postal_code_snapshot        submitted/validated PIN or postal identifier
  ...                         boxes, state and approved policy evidence
```

Use protected historical relationships and account deactivation as the technical starting point; do not cascade account deletion into bookings or financial records. Final personal-data retention/anonymization treatment remains subject to the existing LR-04 and financial-history requirements.

The sample checkout payload remains name, mobile, address and PIN code, with the selected market supplied separately or within the request contract. An auth ID must not become a client-authoritative checkout field. Store the sample mobile as text. Address/serviceability validation still belongs to the backend and configured Market/ServiceArea rules.

| Moment | Account association | Historical contact/address |
| --- | --- | --- |
| Guest checkout, including while signed in | customer_account_id = NULL forever | Captured for this booking |
| New booking after authenticated login | Derived from trusted request identity | Captured for the new booking, independently of profile defaults |
| Later signup or attempted claim of guest booking | No association; claim/import is prohibited | Preserved unchanged |
| Logout, expired session or profile edit | Preserved | Preserved |

There is no account-linking evidence model or claim workflow to build. Record booking phone verification and policy acceptance independently; neither consent nor audit records may indirectly attach guest bookings to CustomerAccount. A mode/FK check constraint must be complemented by immutable-update protection, since both fields could otherwise be changed together into another valid combination.

Contact snapshots may be embedded or protected related records, but acceptance must require their presence; a OneToOne alone only limits the maximum count. A mutable shared CustomerContact is not historical evidence. Never deduplicate guests into people/accounts by phone or create an auth account for each guest.

## Work needed later

| Component | Expected change for customer login |
| --- | --- |
| Existing catalogue, markets and pricing inputs | No identity-driven schema rewrite. Public reads continue as designed. |
| Existing stock, custody and staff operation history | Keep current identities, staff guards and protected history. |
| Booking schema prepared as above | No FK target replacement; account checkout sets ownership at creation, guest ownership stays null forever. |
| Current model-first roadmap | 4.M1/4.M6 must supply the customer/booking relationship before Task 4 completion, not defer it to a later retrofit. |
| Accounts | Task 4 adds separate customer persistence; Task 6 adds reviewed customer login/logout/current-user, recovery, abuse controls and session/token lifecycle, initially disabled server-side. Preserve the existing staff auth table. |
| API authorization | Add customer ownership checks and keep explicit staff/action/assignment rules. |
| Audit | Support truthful staff/customer/system attribution and booking verification evidence without rewriting staff history or linking guest bookings to accounts. |
| Frontend and deployment | Credential transport, CSRF, login state, private rendering/cache controls and account screens. Final visuals remain subject to D-21. |
| Historical guest records | Keep permanently unlinked. Test denial of matching, claims and indirect associations; support booking-specific guest access and controlled recovery only. |
| Future payment/invoice/delivery workflows | Refer to stable bookings/purchases and historical snapshots; provider callbacks and background work must not require a live customer browser session. These workflows are not implemented yet. |

For account history queries, an index shaped around account plus booking time can be added when the actual query exists. Avoid speculative indexes on every contact field. Binding a retry to a session key would break across login/session rotation; booking request identity and request fingerprints should remain separate from transient session identity.

No guest-ownership backfill is permitted. Task 4 prepares account-mode ownership before checkout launches. Future migrations must preserve guest null ownership, stable booking identity and historical evidence; adding a column never manufactures verification or permission to link records.

Test attempted guest-to-account conversion through saves, bulk operations and supported database boundaries, as well as forged customer IDs and concurrent checkout retries. No ownership operation may re-trigger reservation, payment, stock or invoice effects.

No reliable effort estimate follows merely from having two tables. Authentication transport, recovery, abuse controls and testing still need design; guest-claim policy is settled as prohibited. Task 4 completes persistence for the agreed scope, not a production-ready authentication service.

## Business continuity, expansion and funding readiness

Owner follow-up, 2026-10-02: the guest launch must remain suitable for a durable startup, future cities, customer accounts and a mobile application. Business operations must support financial, accounting and compliance review. This reinforces the existing business context and R-13/R-26/R-27/R-28/R-30/R-31; it does not approve unresolved tax/legal policies or certify the unfinished application.

Guest checkout can retain commercial history without a registered customer. Earlier guest bookings/sales remain permanently unlinked to accounts and still appear in authorized operations, accounting exports and historical performance reports. Customer access and the company's ability to substantiate transactions are separate authorization questions.

An account launch or schema migration does not by itself invalidate earlier records. The engineering concern would be lost evidence, unexplained discrepancies or an inability to reconcile results. No technical assessment can guarantee an investor's reaction or a funding outcome, but there is no technical reason to discard guest-era transactions.

### One continuing business system

| Development stage | Intended extension | Historical continuity |
| --- | --- | --- |
| Guest website | Booking contact snapshots, business records and protected guest/staff access | Durable business IDs and evidence do not depend on a customer session. |
| Additional cities | Configure markets, hubs, service areas, inventory and reviewed operating policies | Reports preserve the city, responsible entity and facts applicable at the original event. Activation alone should not require a schema redesign. |
| Customer accounts | Enable prepared authentication and account checkout after readiness approval | Existing guest records stay permanently unlinked and remain available to authorized business reporting. |
| Mobile application | Add another client of the same versioned backend API; extend the authentication transport where needed | Website and mobile access the same authorized business records. A mobile app does not inherently require a separate database or business backend. |
| Necessary infrastructure replacement | Migrate and validate the continuing records or provide a controlled historical reporting archive | Source identity, document evidence, relationships and accounting reconciliation remain retrievable. |

Future schema migrations, new features and capacity work should be expected. The design target is controlled change with preserved business history, not a promise of zero future migrations or operating complications. Additional cities can bring new commercial and statutory requirements even when the data model already supports multiple markets.

### Evidence to preserve from the first live transaction

Keep stable booking, purchase, payment and document identifiers with the relationships between them. Retain original occurrence times separately from later import/link times, the actual responsible legal entity, currency and relevant market/hub facts. Preserve the applicable item, quantity, agreed price, tax and recipient snapshots. Keep actual dispatch, return, QC, cancellation and no-purchase outcomes where relevant.

Verified receipts, deposit obligations/applications, refunds, adjustments, provider fees and bank settlement evidence must remain distinguishable and reconcilable. A trial booking is not revenue; a collected deposit is not automatically jewellery revenue; net bank settlement is not the original gross sale. Revenue recognition and accounting mappings remain CA-01/CA-02/CA-03 decisions. Reconcile application records with the actual accounting books rather than treating an application dashboard as the books themselves.

Historical invoices and financial evidence must not be rebuilt from today's profile, catalogue, tax configuration or selling entity. Corrections must retain the original facts and their authorized adjustments. Changing from one legal entity to another must preserve the original seller rather than presenting earlier transactions as issued by the new entity. Existing requirements are detailed in [COMPLIANCE_AND_FINANCE.md](../COMPLIANCE_AND_FINANCE.md).

Funding reports should be reproducible from retained evidence, with documented definitions for booking conversion, completed sales, refunds, contribution and city performance. Do not count bookings as sales or silently include deposits in revenue. Guest mobile counts are not verified unique-person or registered-account counts; disclose deduplication assumptions and methodology changes after accounts launch. Account activation must neither associate old guest records nor silently rewrite previously reported metrics.

Use aggregate or redacted information for a pitch. Access to transaction-level personal data during authorized diligence requires a separate controlled disclosure process; an investor presentation does not require exposing raw delivery addresses or phone numbers.

### If a new database is eventually necessary

Before cutover, rehearse the migration and restoration using a protected representative copy. Preserve stable IDs where possible; otherwise keep an explicit, durable source-to-target identity mapping. Preserve original invoice numbers, original event times, entity attribution, issued files and supporting payment/stock evidence. Historical imports must not re-trigger fulfilment, collection, messaging or revenue-producing events.

Validate relationships and individual records as well as counts. Reconcile gross sale values, taxes, deposits and their balances, receipts, refunds, fees, settlements and stock by the relevant entity/currency/period. Record the expected handling of transactions in progress at cutover, control concurrent writes, and retain a tested recovery path that accounts for writes made after the switch. A row count or an untested backup alone is insufficient.

AWS DMS is one possible migration mechanism with source/target data validation; it does not replace business-total reconciliation or a cutover design, and it is not required merely to add an account column. [AWS DMS data validation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html).

Prefer continuous historical reporting in the evolved system. A controlled read-only archive can supplement it when an eventual replacement cannot represent every legacy structure, but retrieval and combined reports must be demonstrated before retiring the old runtime. Archive retention, access and personal-data treatment must follow the reviewed policy; this is not a direction to retain all personal data indefinitely.

### Guest checkout still handles personal data

Avoiding customer accounts reduces login, credential-recovery and profile-management work. It does not eliminate the responsibility to protect names, mobile numbers, delivery addresses and associated transactions. The DPDP Act defines personal data through identifiability rather than whether someone registered for an account. [DPDP Act, section 2](https://www.indiacode.nic.in/bitstream/123456789/22037/2/a2023-22.pdf).

Minimize collection, enforce staff/guest access, protect storage and transport, redact logs, test restoration and define the appropriate purpose, retention and correction/access processes from the guest launch. Name/mobile/address alone should not be assumed sufficient for every financial document; PIN/market validation and any CA-reviewed invoicing particulars still apply. No compulsory registration or extra speculative identity collection follows from this review.

Privacy and financial-record obligations must be reconciled, not addressed by either deleting all history when an account is removed or retaining every profile field forever. The published DPDP rules have phased commencement, so LR-04 must assess the requirements effective for the actual launch and later operations. [MeitY rules and enforcement timeline](https://www.meity.gov.in/documents/act-and-policies/digital-personal-data-protection-rules-2025-gDOxUjMtQWa?pageTitle=Digit). For applicable registered-person recordkeeping, the official CGST Rule 56 includes document/stock records and electronic edit/delete history; its application to the business remains part of CA/legal review. [CBIC Rule 56](https://taxinformation.cbic.gov.in/content-page/explore-rules/1000149/1000001).

### Present readiness versus required readiness

The reviewed foundations preserve audit and stock history, but customer booking, sale, payment, invoice and reconciliation workflows are still later tasks. Therefore the current application cannot yet be described as operationally or financially production-ready. Guest checkout is compatible with those requirements; it is not permission to postpone them until fundraising or multi-city expansion.

Under D-26, Task 4 establishes all agreed identity/booking/financial and related persistence, migrations and model tests. Task 5 consumes it for retained storefront/cart work; Task 6 implements booking phone OTP, guest access, initially disabled customer authentication and checkout ownership. Tasks 7–9 implement operations and integrations; Task 10 verifies restoration, reporting and launch readiness. Original task IDs and policy gates remain; no new acceptance has passed merely through this update.

This follow-up adds an architectural assessment only. The verification below was performed for the preceding code review; no additional application tests or database migrations were run for this documentation extension.

## Verification performed

- Read the canonical business, architecture, model, decision, relevant task and state-machine documents; reviewed implemented identity, API, service authorization, audit/history, migration and frontend transport boundaries.
- Inspected Django's runtime model registry and migration files. Confirmed there is only the initial accounts migration among the accounts/trials/delivery/billing/payments/notifications modules.
- Inspected table names in the two existing workspace PostgreSQL databases, `arkatara_foundation` and `arkatara_storefront_check`. Both have sessions/auth and foundation tables, with no customer/booking/financial transaction tables. No customer row data was queried.
- Ran the non-staff identity permission probe described above. It exercised the existing test-only API view and actual inventory authorization function without creating an account.
- Ran `test_api_permissions.py` and `test_environment.py`: 22 passed.
- Ran `test_api_permissions.py`, `test_foundation_models.py`, `test_foundation_admin.py`, `test_foundation_migrations.py` and `test_inventory_operations.py` against isolated PostgreSQL test storage: 75 passed. The two groups overlap; their counts are not a unique-test total.
- The first database test attempt used the default port 5432 and failed during connection setup. The workspace cluster actually uses 127.0.0.1:55432; the corrected run above passed. No application defect was inferred from that configuration failure.
- The local cluster was started for verification and returned to its stopped state afterward. Application source and schema were not modified; migrations exercised by pytest were in its disposable test database.

These dated checks validate the reviewed foundations, not the new Task 4 scope. They do not validate customer login, customer-owned bookings or invoice workflows. Required future tests cover cross-account access, forged IDs, recycled/shared/changed numbers, prohibited guest linking, booking OTP binding/replay, session/token expiry/revocation, applicable CSRF, staff credential separation and immutable historical contacts/documents.

Django's session storage is separate from domain foreign keys. A shared database session backend can support multiple application instances; changing to a shared cache-backed session configuration would be an infrastructure/authentication change, not an order-schema redesign. Cache-only session eviction/restarts can log users out, so storage choice still requires operational testing. [Django 5.2 session documentation](https://docs.djangoproject.com/en/5.2/topics/http/sessions/#configuring-the-session-engine).
