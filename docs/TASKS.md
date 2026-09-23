# Implementation backlog

Status: TASK 1 AUTHORIZED; LATER TASKS REMAIN PLANNED\
Recorded: 2026-09-19\
Implementation status: **TASK 1 IN PROGRESS; TASKS 2–10 PLANNED / NOT STARTED.**

The owner approved the business context and invariants, approved the Next.js/React + Django/DRF + PostgreSQL stack, and authorized Task 1 implementation on 2026-09-19. This authorization does not extend to Tasks 2–10 or resolve any open business, CA or legal decision.

On 2026-09-21, the owner reconfirmed **finish Task 1 only** and initially kept BD-14 pending. On 2026-09-23, the owner approved the separate documentation-only root repository, resolving BD-14 for local tracking. Remote setup/publication is separate. Task 2 remains unapproved and unstarted.

This is the canonical shared backlog for both repositories. Consult [BUSINESS_CONTEXT.md](BUSINESS_CONTEXT.md), [BUSINESS_RULES.md](BUSINESS_RULES.md), [ARCHITECTURE.md](ARCHITECTURE.md), [DATA_MODEL.md](DATA_MODEL.md), [STATE_MACHINES.md](STATE_MACHINES.md), [COMPLIANCE_AND_FINANCE.md](COMPLIANCE_AND_FINANCE.md), [INTEGRATIONS.md](INTEGRATIONS.md), [DECISIONS.md](DECISIONS.md), and the applicable `AGENTS.md` before implementing any task. Do not maintain a duplicate backlog in either application repository.

## Backlog interpretation and gates

- The original task and subtask numbering is preserved. Requirements are grouped below under all nine requested delivery fields. Suggested implementation detail remains proposed until documentation review; an open business or external-review item is never resolved by approving a technical plan.
- One TrialBooking normally means one customer home visit. Multiple category-specific TrialBoxes travel together from the selected city's operational inventory, with a shared booking checkout. Do not infer independent box checkouts, one payment per box, or an approved multi-hub allocation policy.
- Numerical examples are **suggestions only**: seven-day cart retention and two boxes per booking remain **BUSINESS DECISION BD-13**; INR 29/49 deposit examples and whether any deposit is charged remain **BUSINESS DECISION BD-01**, with **CA REVIEW CA-01** for treatment. Never introduce these as production defaults or policy assertions in tests.
- All outstanding entries in [DECISIONS.md](DECISIONS.md) remain open until separately resolved. A policy dependency blocks the behaviour that would force that choice, not unrelated schema or interface work. Development fixtures must be explicitly labelled fictional scenarios, with no production seed silently promoting their values.
- Configuration can encode approved choices; merely adding a configurable field does not authorize choosing its value. Missing required production policy must cause a clear setup/readiness failure, not an invented fallback.
- Product, ProductVariant and uniquely tracked InventoryUnit are distinct. The operational tagging/scanning method, QC criteria and loss/damage handling remain **BUSINESS DECISION BD-09**.
- Tasks are a dependency-aware roadmap, not a requirement to postpone every later-named concern: secure configuration, permissions, integrity and audit foundations begin with Task 1; Task 10 deepens and verifies them. Add relevant event hooks when the underlying domain behaviour is implemented, then validate reporting in Task 10.
- Task 7 creates selection, Purchase and PurchaseItem domain behaviour before Task 8 adds live gateway collection, complete billing and reconciliation. Task 6 introduces provider contracts and development fakes; actual payment integration belongs to Task 8 and messaging integration to Task 9. Fake payment or messaging success is never evidence of a real payment or a production-ready customer flow.
- A live pilot before Task 8 has no approved collection/confirmation procedure (**BUSINESS DECISION BD-06**). Deferred integration does not imply cash, manually verified UPI, screenshots or permission to launch without trusted payment confirmation.
- Source-control tracking follows **BUSINESS DECISION BD-14**, approved on 2026-09-23: a documentation-only root repository excludes the two separate application repositories and local tools/data. Keep the canonical copy under root `docs/`; do not merge application histories, introduce submodules or duplicate documents. Documentation remote setup/publication remains separate.
- Provider choices, validation of the shared Razorpay QR/link flow and channel retry/fallback/OTP-resend/retention operations are **BUSINESS DECISION BD-15**. Customer-facing legal pages, privacy, consent, retention and applicable messaging/DLT obligations are **LEGAL REVIEW LR-04**. These gates apply wherever the corresponding work appears below, including Tasks 3, 4, 6, 7, 8, 9 and 10.

### Reconciliation against the approved business context

No direct business contradiction was identified. The roadmap makes hub inventory ownership and the catalogue-design/variant/physical-unit distinction explicit. It refines engineering preferences and adds a proposed delivery sequence. Potential conflicts arise only if illustrative policies or lifecycle examples are treated as approved defaults: deposit-gated bookings, expiry/late-payment outcomes, price commitment, hub allocation, Gold operating rules and handover during uncertain payment are still open. Example state lists are candidates to refine in [STATE_MACHINES.md](STATE_MACHINES.md), not unconditional transitions or authorization to bypass decision gates.

## 1. Project foundation, architecture and business configuration

**Status:** IN PROGRESS — AUTHORIZED 2026-09-19.

**Progress recorded 2026-09-20:** The backend/frontend structures, module packages, environment settings, `/api/v1/`, foundational LegalEntity/configuration/audit models, lockfiles, formatting/linting/test tooling and CI definitions are implemented on `feature/task-1-foundation`. Local static checks and the frontend production build pass. PostgreSQL migration and database-backed test execution still require a running PostgreSQL 18 service or the first CI run; canonical-document source-control ownership remains BD-14. Task 1 is therefore not yet marked complete.

**Verification update 2026-09-21:** The original foundation branches were pushed and PR #1 in each repository was merged into `development`. The [backend CI run](https://github.com/AmirtharajMuthukrishnan/arkatara_backend/actions/runs/35463839569) passed at `43393a4`, including PostgreSQL migrations/tests; the [frontend CI run](https://github.com/AmirtharajMuthukrishnan/arkatara_frontend/actions/runs/35463386317) passed at `cb89c6b`. These results supersede the earlier outstanding-CI note. A follow-up review found configuration validation/history, audit enforcement, environment-isolation tests and frontend API path/error-handling gaps; fixes are being verified on `fix/task-1-verification`. BD-14 remains explicitly pending, so Task 1 is not complete.

**Resumed verification 2026-09-23:** The follow-up fixes are implemented locally in both repositories on `fix/task-1-verification`. Backend verification passed against an isolated PostgreSQL 18.6 database: **62 tests**, Django system checks, migration drift checks, migration from the original schema, fresh-schema tests and reversible audit-trigger migration tests. The API started against PostgreSQL and returned HTTP 200 at `/api/v1/health/`. Backend lint and formatting pass. Frontend verification completed before the interruption: **30 tests**, lint, formatting, TypeScript and production build passed; no further frontend code changes were made during the resumed work. These local fixes are not yet committed, pushed or covered by a new CI run; the original green CI results above apply to the earlier foundation commits only.

| Task 1 area | Current state |
| --- | --- |
| 1.1–1.4: structure, modules, environments and API versioning | Implemented and locally verified; staging/production configuration fails clearly when required settings are absent. Hosting/deployment provisioning remains outside this foundation work. |
| 1.5: LegalEntity continuity | Foundation and attributable Admin change history implemented. Actual transaction/issuer snapshots are required when the later transaction models are introduced; no real entity identity has been invented. |
| 1.6: configurable rules | Four typed setting contracts, exact-scope lookup, immutable ORM revisions and missing/approval/conflict errors implemented. Admin creates draft proposals only; no activation/retirement workflow or production values are enabled. |
| 1.8: auditability | Attributed, atomic Admin changes, durable staff identity, read-only audit Admin and PostgreSQL audit UPDATE/DELETE protection implemented and tested. |
| Frontend foundation | API origin/path containment, structured request errors and recoverable page errors implemented and verified. |
| 1.7: review and CI | Initial foundation PRs passed and merged. Follow-up fixes still require commits, pushes, PR review and green CI before merging. |
| 1.7: canonical documentation tracking | **APPROVED — BD-14 resolved 2026-09-23**. The root documentation repository tracks the canonical set and setup guide; both application repositories are excluded. Documentation remote setup/publication remains a separate step. |

**Documentation tracking update 2026-09-23:** The owner approved creating the root documentation repository after confirming that application code would remain in its existing repositories. The earlier BD-14 deferral is retained in the dated progress notes above as history and is superseded by this approval. Root Git contains only the nine canonical documents, root guide and repository setup files; GitHub remote configuration/publication is separate.

**Remaining completion gates:** review/CI/merge of the application follow-up fixes. BD-14's tracking decision is resolved; documentation remote setup/publication is still a separate pending step before a new collaborator can clone it remotely. Other unresolved commercial, CA and legal entries remain unset and gate their dependent later features. Task 2 is not authorized and has not started.

**Objective:** Establish a reproducible project structure, proposed architecture, environments, engineering standards and foundational business configuration without provider integrations.

**Dependencies:** The Next.js/React + Django/DRF + PostgreSQL modular-monolith baseline is approved. Detailed domain proposals remain reviewable and unresolved decision gates still apply.

**Backend work:**

- **1.1** Create the frontend/backend project structure around the preferred Next.js / React frontend, Django + Django REST Framework backend and PostgreSQL database. Use a modular monolith.
- **1.2** Establish cohesive Django modules, initially considering `accounts`, `catalog`, `markets`, `inventory`, `trials`, `delivery`, `payments`, `billing`, `notifications`, `compliance` and `analytics`; refine names and responsibilities as appropriate without introducing independent services.
- **1.3** Separate local, staging and production environments, each with distinct secrets and configuration. Document reproducible setup and prevent production credentials or data from becoming local fixtures.
- **1.4** Establish API versioning beginning with `/api/v1/`.
- **1.5** Design LegalEntity support that preserves the issuer/party responsible for historical records through future changes in business structure. Do not assume one permanent entity or rewrite past transactions when a new entity is introduced.
- **1.6** Establish validated configuration for changeable rules, including maximum boxes, cart expiry, reservation timeout and category trial eligibility. Preserve historical commitments separately from current configuration and leave unresolved production values unset.
- **1.7** Establish the `main`, `development` and feature-branch workflow, formatting, linting, meaningful test frameworks, CI and migration checks. Track the single canonical documentation set in the separate root repository approved under BD-14; keep remote setup/publication separate. The earlier documentation-only prohibition on creating branches is superseded by the explicit Task 1 and root-repository authorizations.
- **1.8** Give important models suitable timestamps and auditability; identify immutable records, attributable privileged changes and integrity-sensitive writes from the beginning.

**Frontend work:** Establish the Next.js / React structure, API boundary, environment separation, formatting/linting/tests and predictable error handling. Keep business policy authoritative on the backend and prepare for data-driven market/material activation. No customer storefront implementation is implied before its later task.

**Admin/operations work:** Document local setup, staging/production separation, permission boundaries, migration generation/review/application discipline, deployment responsibilities and recovery expectations. Plan restricted staff administration and configuration audit history. Choose only the minimum operational tooling needed after review.

**Business/account prerequisites:** **BUSINESS DECISION BD-11** for actual operating entity and ownership arrangements; **CA REVIEW CA-03** and **LEGAL REVIEW LR-02** for entity transition requirements; **BUSINESS DECISION BD-13/BD-02** for policy values. **BD-14** for canonical documentation source control was resolved on 2026-09-23; remote setup/publication remains separate. Domain design can proceed within its authorized scope without fabricating real registrations, addresses or entity identities.

**Tests:** Verify environment isolation, configuration validation, API version routing, denied unauthorized access, audit attribution and reproducible setup. CI must detect migration drift and exercise migrations against PostgreSQL where relevant. Demonstrate that changing current entity/configuration references cannot rewrite historical snapshots once those records exist.

**Definition of Done:** Approved technical baseline is reproducible; environments are separated; CI and migration checks run; module responsibilities and source-control workflow are documented; unresolved business settings remain explicit; the canonical documentation is accessible to both repositories under an approved tracking approach. Application and infrastructure work is reviewed independently of this documentation delivery.

**Non-goals:** Payment integration, messaging integration, production account activation, microservices and choosing unanswered commercial policies.

## 2. Future-proof market, material, catalogue and pricing model

**Status:** PLANNED / NOT STARTED.

**Objective:** Model the business so Hyderabad and Gold can be enabled through configuration, catalogue/inventory onboarding and reviewed operating policies without major schema, backend or frontend redesign.

**Dependencies:** Task 1 foundations and reviewed [DATA_MODEL.md](DATA_MODEL.md); rule changes that require decisions must be gated rather than inferred from future expansion examples.

**Backend work:**

- **2.1** Create Market with initial business configuration `BLR / Bengaluru / ACTIVE` and `HYD / Hyderabad / COMING_SOON`.
- **2.2** Create Hub; inventory belongs to specific hubs within a market. No implicit intercity fulfilment.
- **2.3** Create data-driven ServiceArea support. Do not hard-code Bengaluru PIN-code checks in Python or frontend logic.
- **2.4** Create Material, initially `SILVER / ACTIVE` and `GOLD / COMING_SOON`.
- **2.5** Model Purity separately from Material. S925, 14K, 18K and 22K illustrate supported concepts, not approval to sell every example.
- **2.6** Create Category with activation and trial eligibility. Ring, Chain, Bracelet, Kada, Earring and Stud are examples; initial emphasis remains Rings, Chains, Bracelets and Kadas.
- **2.7** Separate Product (catalogue design), ProductVariant (selectable specification) and InventoryUnit (unique physical jewellery piece). Never use one Product row for all three responsibilities.
- **2.8** Support pricing concepts `FIXED` and `WEIGHT_BASED`. Silver may initially use fixed pricing; the actual pilot choice, Gold formula/components and commitment point remain subject to BD-04. Support reasonable future Gold pricing without redesign, without inventing its commercial formula.
- **2.9** Snapshot historical agreed prices and applicable taxes in transaction records. Never regenerate historical financial values from current Product, price or tax configuration.

**Frontend work:** Define typed/API-facing market, material, category, variant, availability and price-display contracts that can render active/coming-soon states. Avoid frontend constants that equate the entire business with Bengaluru or Silver. Backend results govern bookability and financial values.

**Admin/operations work:** Prepare reviewed configuration onboarding for markets, hubs, service areas, materials, purities, categories and pricing. Use business identifiers independently of display names and preserve change history. Document which future expansion settings require approvals before activation.

**Business/account prerequisites:** **BUSINESS DECISION BD-04** for price formula/commitment; **BD-08** for fulfilling hub and within-city multi-hub sourcing; **BD-12** for Gold offerings/operations; **BD-13** for service areas, eligibility and limits; **BD-11**, **CA REVIEW CA-03**, **LEGAL REVIEW LR-02** for entity/stock ownership; **CA REVIEW CA-02** for tax configuration. Approved city/material launch states do not resolve these policies.

**Tests:** Verify market/hub isolation, valid material/purity relationships, inactive offerings, separate design/variant/unit identity, backend serviceability and historical snapshot stability. Use explicit scenario fixtures for fixed/weight-based representability; do not assert an unapproved production formula.

**Definition of Done:** Core entities and constraints support the approved invariants, reviewed pricing representations and historical evidence. A second market/material can be represented without schema duplication; coming-soon offerings cannot be booked. Remaining Gold or fulfilment choices stay visible in the decision log.

**Non-goals:** Launching Hyderabad or Gold, approving a particular Gold purity or price formula, intercity fulfilment or arbitrary configuration frameworks for hypothetical use cases.

## 3. Django Admin, product onboarding and inventory operations

**Status:** PLANNED / NOT STARTED.

**Objective:** Provide the initial back office through Django Admin and traceable physical inventory operations.

**Dependencies:** Tasks 1–2; reviewed InventoryUnit lifecycle and movement model. Security and audit controls apply before privileged Admin mutations are enabled.

**Backend work:**

- **3.1** Configure administration for Markets, Hubs, Materials, Purities, Categories, Products, Variants and Inventory.
- **3.2** Support main and additional images, video and thumbnails behind an object-storage abstraction. Keep media metadata, access rules and provider-specific storage concerns separate.
- **3.3** Implement SKU identifiers and uniqueness rules. Business logic must use actual domain attributes rather than parse meaning from a SKU string.
- **3.4** Implement a reviewed InventoryUnit lifecycle. Candidate states are `AVAILABLE`, `RESERVATION_PENDING`, `RESERVED`, `PACKED`, `OUT_FOR_DELIVERY`, `IN_PERSON_TRIAL`, `SOLD`, `RETURNING_TO_HUB`, `QC_PENDING`, `DAMAGED` and `RETIRED`. Refine states and guards using [STATE_MACHINES.md](STATE_MACHINES.md); do not infer deposit or QC policies from a candidate name.
- **3.5** Implement append-only InventoryMovement history sufficient to reconstruct each physical unit's location, custody, movement and relevant condition changes. Correct mistakes using attributable subsequent records rather than silently rewriting history.
- **3.6** Support practical bulk onboarding, considering CSV/import tooling with validation, clear errors and safe repeat/import behaviour.
- **3.7** Provide useful Admin filters for Market, Hub, Material, Category, Status, Availability and Size.

**Frontend work:** None beyond agreed media/catalogue API contracts and development verification needed by the later storefront. The doorstep agent workflow is a separate Task 7 interface.

**Admin/operations work:** Provide role-appropriate catalogue maintenance, inventory receiving, condition recording, location/custody actions and bulk onboarding. Prevent generic Admin edits from bypassing transition guards, unit uniqueness, append-only history or audit attribution.

**Business/account prerequisites:** **BUSINESS DECISION BD-09** for physical tags/scanning, receiving procedures, QC criteria and damage/loss dispositions; **BD-08** for hub allocation; **BD-11**, **CA REVIEW CA-03**, **LEGAL REVIEW LR-02** for ownership evidence. Production object storage requires approved account, region/access settings and credential management as described in [INTEGRATIONS.md](INTEGRATIONS.md).

**Tests:** Verify unit/SKU uniqueness at the appropriate level, permission boundaries, forbidden transitions, movement history preservation, atomic state/history changes, rejected invalid imports and reliable reprocessing. Check that returned or failed-QC units cannot become available through unrestricted Admin actions.

**Definition of Done:** Staff can onboard and locate stock, inspect unit history, use filters and media, and operate approved transitions without compromising custody evidence. Required unresolved operating rules are gated. The Admin remains the initial back office.

**Non-goals:** A separate custom back-office application, selecting a barcode/RFID scheme without approval, completing sale/payment workflows or replacing the Task 7 agent portal with Admin.

## 4. Customer storefront and scalable UI

**Status:** PLANNED / NOT STARTED.

**Objective:** Build a premium, trustworthy, mobile-first jewellery catalogue suited to Instagram/Facebook traffic.

**Dependencies:** Tasks 1–3 catalogue, media, configuration and market/hub availability contracts. Trial-cart and booking behaviours are delivered in Tasks 5–6; do not present a placeholder flow as a completed booking.

**Frontend work:**

- **4.1** Build the mobile-first landing page with Hero, Try-at-Home USP, Categories, Featured products, How it works, Trust, FAQ and Legal/footer sections. Use the approved promise: “Order many. Try them at home. Buy any.”
- **4.2** Add the Market selector: Bengaluru active and Hyderabad coming soon. Disabled Hyderabad may say “We're scaling to Hyderabad”; it must not create invalid bookings/orders.
- **4.3** Add the Material selector: Silver active and Gold coming soon. Disabled Gold may say “Gold collection coming soon”.
- **4.4** Drive later Gold/Hyderabad activation from backend configuration, without duplicate UI development or material/city-specific page forks.
- **4.5** Build category listing pages.
- **4.6** Build product detail pages with relevant photos/videos, SKU/design code, price, material, purity, size, weight, availability, reviewed tax wording and Try-at-Home CTA.

**Backend work:**

- **4.7** Ensure product APIs respect selected Market/Hub inventory and approved serviceability. Bengaluru-unavailable stock must not appear as immediately bookable Bengaluru inventory. Customer intent must not itself choose an unauthorized hub or imply a multi-hub sourcing policy.

**Admin/operations work:** Maintain merchandising, category/featured content, availability and media through the back office. Review customer-facing promises against actual operations and keep coming-soon information consistent across pages.

**Business/account prerequisites:** Approved brand assets and truthful trust/FAQ content; **BUSINESS DECISION BD-04** for price wording/commitment; **BD-12/BD-13** for future offerings and eligibility; **CA REVIEW CA-02** for tax wording where applicable; **LEGAL REVIEW** for customer-facing claims and required disclosures as recorded in [DECISIONS.md](DECISIONS.md). Do not fabricate certifications, testimonials or policy guarantees.

**Tests:** Check mobile usability, accessibility, active/coming-soon presentation, data-driven activation, catalogue/media behaviour and market-specific bookability. Verify that direct URLs or manipulated frontend state cannot bypass backend restrictions.

**Definition of Done:** Reviewed catalogue pages work on mobile, show authoritative availability and prices, communicate active/coming-soon states accurately, and can render the configured expansion scenarios. Trial CTAs integrate with later tasks before a live end-to-end launch.

**Non-goals:** Customer registration, Hyderabad/Gold launch, independent category checkouts, invented trust claims or customer promises about unanswered policies.

## 5. Anonymous Trial Cart and multi-box logic

**Status:** PLANNED / NOT STARTED.

**Objective:** Let customers assemble category-specific boxes into one intended home-visit booking without creating an account or reserving stock.

**Dependencies:** Tasks 2 and 4 catalogue/configuration contracts; reviewed TrialPlan and booking structure. Production policy-dependent behaviour waits for BD-01/BD-03/BD-13 choices.

**Frontend work:**

- **5.1** Allow browsing without login.
- **5.2** Persist Trial Cart intent in `localStorage` across refresh, page/category navigation and browser restart for an approved configured period. Handle unavailable storage and stale or malformed data safely; local state is never proof of availability.
- **5.3** Implement configurable cart expiry. Seven days is a suggestion, pending BD-13, not a default approved by the roadmap.

Render category boxes and their approved plan limits clearly, guide the customer toward a combined booking, and explain authoritative revalidation results. Avoid storing sensitive checkout/address/payment data in the anonymous cart without an independently justified and reviewed need.

**Backend work:**

- **5.4** Enforce category-specific TrialBoxes: for example, a Ring box contains Rings only.
- **5.5** Support multiple category boxes, such as Ring + Chain, within one TrialBooking and delivery booking. Multiple boxes must not cause independent customer checkouts.
- **5.6** Create configurable TrialPlans; possible dimensions include Market, Material, Category, piece limit, deposit amount and active flag. Model representability without selecting deposit scope or material-mixing policy. Never hard-code INR 29/49 or other amounts into frontend logic.
- **5.7** Make maximum TrialBoxes configurable. Two boxes is a suggested pilot value only, pending BD-13.
- **5.8** Ensure anonymous cart additions never reserve physical inventory.
- **5.9** Treat frontend price, category, deposit amount and availability as untrusted. Validate against authoritative backend facts at booking and subsequent authoritative operations.

**Admin/operations work:** Configure reviewed TrialPlans and limits, with validated activation and change history. Preserve what was offered/committed historically when plans later change.

**Business/account prerequisites:** **BUSINESS DECISION BD-01** for deposit existence/scope/amount/application; **CA REVIEW CA-01** for treatment; **BD-03** for counting pieces/variants, repeated category boxes, mixed materials and substitutions; **BD-13** for expiry, item and box limits and eligibility. Missing settings must not silently receive roadmap sample values.

**Tests:** Check anonymous access, local persistence/expiry with explicit test settings, malformed/tampered cart data, one-category invariants, multiple boxes in one checkout, plan changes and stale inventory. Confirm no InventoryUnit reservation or stock decrement occurs from any cart operation.

**Definition of Done:** A customer can assemble and restore valid intent without an account or inventory hold; multi-box intent stays attached to one booking journey; backend validations reject tampering. Only approved production settings are activated.

**Non-goals:** Reserving inventory, collecting deposits, choosing deposit scope from box count, creating customer accounts or assuming suggested numeric limits are approved.

## 6. Trial booking and inventory reservation engine

**Status:** PLANNED / NOT STARTED.

**Objective:** Convert anonymous intent into reliable, concurrency-safe booking and reservation records while keeping commitment and payment policies explicit.

**Dependencies:** Tasks 1–5 and reviewed booking/inventory state machines. Reservation/acceptance behaviour depends on BD-02; address/service/slot behaviour on BD-07/BD-08/BD-13. External providers remain development fakes until their later tasks.

**Frontend work — checkout:**

- **6.1** Collect minimal checkout information: name, mobile, address and PIN code, without requiring a customer account. Submit all boxes as one booking intent and show backend validation/availability results.

**Backend work:**

- **6.2** Resolve serviceability from backend Market/ServiceArea configuration.
- **6.3** Revalidate each requested item: it exists, is active, belongs to the correct market/hub and category, is trial-eligible, and is available. Resolve actual physical allocation server-side according to the approved fulfilment policy.
- **6.4** Protect concurrent bookings with atomic database operations and appropriate transactional locking/constraints. One physical unit cannot be committed incompatibly or sold twice.
- **6.5** Implement TrialBooking → TrialBox → TrialBoxItem, preserving the normal shared customer/address/visit and joint fulfilment model.
- **6.6** Implement the reviewed booking lifecycle and allowed/forbidden transitions with explicit actor permissions and policy guards.
- **6.7** Support temporary reservation expiry for incomplete deposit/payment attempts once the hold/deposit policy is approved. Hold duration, release triggers, whether deposit is required and late-payment handling remain BD-02; do not make `PAYMENT_PENDING` compulsory for every booking.
- **6.8** Create provider abstractions, considering PaymentProvider, NotificationProvider and OTPProvider, with fake/development implementations initially. Keep secure OTP domain logic separate from transport. Fakes must be confined to safe environments and visibly distinguishable from real verification.

**Frontend work — policy pages:**

- **6.9** Build Terms, Privacy Policy, Try-at-Home Policy, Refund/Cancellation Policy, Contact and grievance-information pages. Drafting/page support does not approve legal wording or unresolved policies; flag final content for **LEGAL REVIEW LR-04**, alongside the commercial-policy constraints in LR-01.

**Admin/operations work:** Provide booking visibility and controlled exception handling, reservation-expiry monitoring and reviewable state/history. Ensure retries do not create duplicate commitments. Document operational handling for conflicts and incomplete attempts once the relevant policies are approved.

**Business/account prerequisites:** **BUSINESS DECISION BD-01/BD-02** for deposit and reservation commitment; **BD-03** for quantities/substitution; **BD-07/BD-08/BD-13** for service areas, scheduling, hub sourcing and limits; **BD-06** for any live pilot before gateway integration. **LEGAL REVIEW LR-01** covers relevant policy constraints; other customer terms/privacy/grievance requirements must receive the applicable legal review recorded in [DECISIONS.md](DECISIONS.md).

**Tests:** Use PostgreSQL concurrency tests for competing allocations and rollback; verify replay-safe booking requests, serviceability, category/market constraints, inactive/stale items and untrusted financial fields. Test hold expiry racing payment or booking changes using approved policies, with fake provider contracts where integration is deferred. Check legal-page access and customer-data permissions.

**Definition of Done:** With approved policy settings, one combined booking is created atomically, invalid intent is rejected, competing requests cannot overcommit stock, and every transition is guarded/auditable. Fakes exercise interfaces without impersonating production evidence. Legal pages remain draft until reviewed.

**Non-goals:** Razorpay integration, WhatsApp/SMS delivery, a live pilot payment workaround, deposit-required assumptions, intercity fulfilment or legal-compliance certification.

## 7. Delivery operations and delivery-agent portal

**Status:** PLANNED / NOT STARTED.

**Objective:** Control staff access, dispatch, attended trial, purchase selection and physical returns while preserving custody and authoritative pricing.

**Dependencies:** Tasks 1–3 and 6; catalogue/media support from Task 4; reviewed assignment, OTP, inventory and booking state machines. Purchase selection/domain records are needed here before Task 8; genuine payment confirmation and complete production invoicing remain later dependencies for a live sale flow.

**Backend work:**

- **7.1** Create authenticated staff accounts and RBAC, considering ADMIN, OPERATIONS, INVENTORY_MANAGER, DELIVERY_AGENT and FINANCE. Apply server-side permission checks, including assignment-specific agent access.
- **7.2** Create DeliveryAssignment and controlled assignment/reassignment history. A normal booking's boxes share one visit; do not infer separate assignments/checkouts from box count.

**Frontend work:**

- **7.3** Build a mobile-friendly staff page such as `/staff/deliveries/{booking_id}`. Agents see only permitted/assigned bookings; the route shape alone must not grant access.
- **7.4** Show customer, address, phone, TrialBoxes, products, SKUs, photos and current state only within authorized access.

Provide OTP entry, item selection, controlled visit actions and accurate payment-pending/return states. Later Task 8 adds authoritative payment status and QR display. Do not imply that pressing a selection or completion button confirms payment.

**Backend work — trial and custody controls:**

- **7.5** Prevent agents from manually overriding authoritative prices through either the UI or API.
- **7.6** Support generation of a Delivery Challan / dispatch document distinct from a sale invoice. Production format, required contents and timing require **CA REVIEW CA-02** and **LEGAL REVIEW LR-03**.
- **7.7** Implement Arrival OTP generation, secure hashing/storage, expiry, attempt limits and verification timestamp. Do not log plaintext OTPs or expose them to agents as a substitute for customer verification. Delivery through real messaging providers is deferred to Task 9.
- **7.8** Allow backend Arrival OTP verification to transition the booking from `OUT_FOR_DELIVERY` to `IN_PERSON_TRIAL`, with authorized assignment and state checks.
- **7.9** Build the selection workflow: the customer chooses pieces, the agent records those choices and the backend creates Purchase/PurchaseItem records and computes authoritative selection values. Their existence does not establish a financially confirmed sale.
- **7.10** Do not require a second OTP solely for item purchase/selection.
- **7.11** Route unpurchased units through `RETURNING_TO_HUB` → `QC_PENDING` → `AVAILABLE`, with recorded return and successful QC. Units still physically with the agent must not become available; failed-QC outcomes remain restricted under the approved policy.

**Admin/operations work:** Manage assignments, packing/dispatch, movement evidence, return receipt, QC and restricted exceptions. Prepare an agent training flow that respects customer choice, price authority, data access and the approved response to failed visits/OTP verification.

**Business/account prerequisites:** Staff identities and approved access responsibilities; **BUSINESS DECISION BD-05/BD-06** for pending-payment handling/handover and pilot payment methods; **BD-07** for failed visits, rescheduling and exceptions; **BD-09** for tagging, loss/damage and QC; **BD-04** for price commitment; **CA REVIEW CA-02**, **LEGAL REVIEW LR-01/LR-03** for dispatch and custody responsibilities. These gates prevent a development trial demonstration from being presented as an authorized live operating process.

**Tests:** Check RBAC and direct-object access attempts, reassignment, forbidden transitions, single booking/multiple boxes, non-overridable prices and secure OTP expiry/replay/attempt limits. Verify selection integrity, no second OTP requirement, movement/custody consistency, physical-return evidence and failed/successful QC gating.

**Definition of Done:** Authorized agents can perform the reviewed development dispatch-to-trial-to-selection/return flow; backend OTP verification controls trial start; selected items create reliable purchase intent; stock cannot bypass return/QC or be falsely marked sold. Production dispatch documents and payment/handover policies require their own approvals.

**Non-goals:** Treating a selected item as paid, treating dispatch as sale, payment/messaging provider integration, unrestricted agent Admin access or silently deciding liability/handover policies.

## 8. Razorpay, GST, billing and reconciliation

**Status:** PLANNED / NOT STARTED. Intentionally later than core booking/trial logic.

**Objective:** Implement verified digital collection and a traceable financial record from deposit and sale through refunds, gateway settlements and bank matching.

**Dependencies:** Tasks 1, 2, 6 and 7 domain foundations; reviewed financial models, provider interfaces and state machines. Resolve the policy/review questions needed by the actual live payment flow before activating it.

**Backend work:**

- **8.1** Model BookingDeposit separately from jewellery value and Purchase; do not derive the commercial deposit scope from box count.
- **8.2** Integrate Razorpay for booking deposits according to the approved policy. Calculate amounts server-side.
- **8.3** Implement the final Purchase calculation from authoritative selected items, reviewed prices/taxes and approved deposit application. Preserve snapshots and explain all components.
- **8.4** Create PaymentAttempt and GatewayTransaction records with explicit links, currency/amount, state and provider references as appropriate.
- **8.5** Use one underlying current final-payment attempt/context for both Agent QR and customer payment URL. Exposing two access methods must not create independent payable transactions. Legitimate future retries or changed selections require controlled supersession and BD-05 policy, not a second concurrent charge for the same obligation.
- **8.6** Verify Razorpay signatures/webhooks; process events idempotently with checks against expected amount, currency, account and payment context. Handle duplicates, delayed delivery and out-of-order events without double confirmation, sale or document issuance.
- **8.7** Never store sensitive card information.
- **8.8** Retain gateway references needed for traceability and reconciliation, without placing secrets or unnecessary sensitive payment data in logs.
- **8.9** Implement configurable tax concepts, including TaxCode, HSN, rate and effective dates. Do not scatter GST percentage constants through code.
- **8.10** Implement TaxInvoice and TaxInvoiceItem with immutable historical contents, controlled numbering and reviewed issue/correction workflows.
- **8.11** Distinguish invoice total, deposit already received/applied and remaining balance. Applying a deposit must not silently change jewellery selling price. Illustrative INR 5,000 less INR 49 equals INR 4,951 outstanding only when that application policy is approved; the example does not approve the deposit amount or universal treatment.
- **8.12** Implement traceable Refund records, authorization, provider state and links to original collections and relevant adjustments. Refund requests are not automatically successful refunds.
- **8.13** Reconcile PaymentAttempt, gateway payment, gateway settlement and bank settlement, including identifiable fees/adjustments as reviewed. Surface unresolved mismatches instead of marking them matched automatically.
- **8.14** Mark every uncertain GST/accounting treatment **CA REVIEW** and obtain the required confirmation before enabling dependent production behaviour.

**Frontend work:** Present backend-calculated amounts, payment states and invoice/balance breakdown; expose the shared QR/link payment context on the agent/customer surfaces; show pending/failure/retry outcomes according to approved policy. Browser callbacks and screenshots cannot establish financial success.

**Admin/operations work:** Provide restricted payment/refund/invoice administration, gateway/bank reconciliation, mismatch queues and review evidence. Preserve immutable invoices and attributable adjustment records rather than editing historical financial facts. Document reconciliation ownership and escalation.

**Business/account prerequisites:** Final business/legal identity; business current account; applicable GST registration; CA-confirmed accounting policy and Trial Deposit treatment; Razorpay business account and KYC; settlement bank account. Business collections use business accounts. **BUSINESS DECISION BD-01/BD-02/BD-04/BD-05/BD-06/BD-10/BD-11**, **CA REVIEW CA-01/CA-02/CA-03** and relevant **LEGAL REVIEW LR-01/LR-02/LR-03** remain explicit gates. Provider product/API capabilities and requirements must be checked against current official documentation when implementation begins.

**Tests:** Verify server-side amounts, signatures, replay/out-of-order handling, duplicate callback/webhook races, shared QR/link context, controlled retries, late payment after reservation expiry and no double sale/collection. Check deposit application/refund/retention only against approved policy, immutable snapshots/invoices, numbering concurrency, tax effective dates, refunds and reconciliation mismatches. Complete test/live verification only within approved accounts and authorized scope.

**Definition of Done:** Verified backend records govern collection; one obligation is not accidentally paid twice through QR/link channels; financial records remain traceable and immutable where required; refunds and settlement mismatches can be reconciled. CA/legal approvals and account readiness for activated behaviour are recorded. Passing software tests alone is not a claim of tax/legal compliance.

**Non-goals:** Inventing GST rates, deposit recognition or retention rules, storing card details, silently choosing physical handover policy, replacing CA review with code or sending live messages before Task 9.

## 9. WhatsApp Cloud API and SMS / TRAI-DLT integration

**Status:** PLANNED / NOT STARTED. Intentionally after core business logic.

**Objective:** Deliver approved transactional notifications and the same Arrival OTP/payment context through reliable WhatsApp and SMS channels, with appropriate account setup and consent boundaries.

**Dependencies:** Task 6 provider contracts, Task 7 OTP/visit domain and Task 8 payment/invoice domain for corresponding messages. Templates may be prepared earlier; production sending requires account approvals and reviewed content.

**Backend work:**

- **9.1** Integrate WhatsApp Cloud API behind the reviewed notification boundary.
- **9.2** Store credentials/tokens securely with environment separation, least privilege and a rotation process.
- **9.3** Configure WhatsApp webhook/status callbacks with the provider's required verification, safe event processing and delivery-state tracking.
- **9.4** Create templates for Booking confirmed, Out for delivery, Arrival OTP, Payment link, Payment success, Deposit refund, Invoice issued and Cancellation. Template wording must reflect approved policies and true domain events; delivery status is not business/payment truth.
- **9.5** Complete the required SMS/DLT business registration and approvals before production sending; software scaffolding alone cannot complete those external prerequisites.
- **9.6** Integrate a compatible SMS provider behind SMSProvider abstraction.
- **9.7** Deliver one underlying Arrival OTP through WhatsApp and SMS. Do not generate a different code per channel; preserve one challenge's expiry, attempt count and verification semantics. Retry/delivery handling must not expose the plaintext OTP in ordinary logs.
- **9.8** Send the final payment URL through WhatsApp/SMS according to approved transactional messaging rules.
- **9.9** Ensure the agent QR and customer payment link resolve to the same PaymentAttempt/context. A notification resend must not create a new payable obligation.
- **9.10** Send post-payment receipt/invoice communications only from authoritative domain events and reviewed document state.
- **9.11** Separate transactional/service messages from marketing messages. A phone number supplied for purchase/service does not automatically grant marketing consent.

**Frontend work:** Show appropriate notification/delivery feedback and permitted resend options without revealing secrets or allowing messaging retries to mutate financial state. Give agents a clear, reviewed recovery process for message delivery problems. Consent/disclosure UI must match the reviewed legal and provider requirements.

**Admin/operations work:** Maintain approved template versions, restricted recipient/message access, delivery-failure monitoring, provider status and escalation procedures. Correlate NotificationEvents to bookings, OTP challenges, payments and documents; avoid duplicate sends caused by worker/webhook retries.

**Business/account prerequisites:** WhatsApp: Meta Business Portfolio, WhatsApp Business Account, dedicated business phone number, Meta developer application, appropriate permissions and approved templates where required. SMS: applicable DLT/Principal Entity registration, approved sender/header, approved message templates and compatible SMS provider. Verify current official provider and Indian messaging requirements at implementation time. Applicable privacy, consent, retention and communications requirements need **LEGAL REVIEW** recorded in [DECISIONS.md](DECISIONS.md); provider choice and operating content need **BUSINESS DECISION** where not approved.

**Tests:** Verify one OTP challenge across channels, delivery retries without generating independent codes, bounded resend/verification behaviour, callback validation/idempotency, secret redaction, failed provider handling and test/live isolation. Confirm notification delivery cannot set a payment to paid, link/QR identity is preserved, and marketing permission is not inferred from service contact details.

**Definition of Done:** Approved transactional templates deliver via configured providers with traceable status and safe retries; OTP/payment authority remains in the business domain; prerequisites and permissions are satisfied for production sending; communications consent boundaries are implemented and reviewed.

**Non-goals:** Marketing campaigns, assuming marketing consent, second purchase OTP, separate OTP per channel, duplicate payment creation or claims that DLT/provider approval establishes overall legal compliance.

## 10. Production hardening, analytics and expansion validation

**Status:** PLANNED / NOT STARTED.

**Objective:** Verify production readiness, secure operations, measurable unit economics and configuration-led expansion. This task completes/validates controls introduced earlier; it does not defer basic security or auditability until launch.

**Dependencies:** Relevant Tasks 1–9 and approved live operating policies, business accounts and external reviews. Expansion simulations use controlled test data; a successful simulation does not authorize a commercial launch.

**Backend work:**

- **10.1** Implement/validate analytics events: `PRODUCT_VIEWED`, `TRIAL_ITEM_ADDED`, `TRIAL_BOX_COMPLETED`, `CHECKOUT_STARTED`, `DEPOSIT_STARTED`, `DEPOSIT_PAID`, `BOOKING_CONFIRMED`, `DISPATCHED`, `ARRIVAL_VERIFIED`, `TRIAL_STARTED`, `PRODUCT_SELECTED`, `FINAL_PAYMENT_STARTED`, `FINAL_PAYMENT_PAID`, `NO_PURCHASE`, `RETURN_QC_COMPLETED`. Define provenance, timestamps, correlation and deduplication; frontend behavioural events are not authoritative financial evidence.
- **10.2** Capture permitted acquisition attribution: `utm_source`, `utm_medium`, `utm_campaign`, `utm_content` and `referrer`, with reviewed retention/privacy controls and no unnecessary customer-sensitive data in analytics payloads.
- **10.3** Build KPI reporting for website visitors; Trial Cart add rate; Trial Box completion; booking conversion; deposit conversion; delivery success; trial-to-purchase conversion; zero-purchase rate; AOV; pieces tried versus purchased; category conversion; refund rate; delivery cost; inventory utilisation; inventory turnaround; agent utilisation; payment failure; refund speed. Define each numerator/denominator, cohort/time basis, exclusions and source so retries and multiple boxes do not inflate customer visits or sales. Relate revenue/conversion to visit, fulfilment, inventory and return/QC costs for pilot economics.
- **10.4** Complete privileged-action audit coverage for price changes, inventory changes, refunds, invoice actions, staff assignments and payment administrative actions. Preserve attribution and prior/new facts without storing secrets.
- **10.5** Validate security hardening: HTTPS, secure cookies where applicable, CSRF protection, rate limiting, staff permission checks, staff MFA if practical, secret management, dependency scanning, webhook signature verification and database backup. Design decisions must reflect the actual deployment and authentication approach.
- **10.6** Monitor server/API errors, payment failures, webhook failures, notification failures, inventory inconsistencies and background-job failures, with actionable ownership and alerts.
- **10.7** Simulate Gold activation: change `GOLD` from `COMING_SOON` to `ACTIVE`; add Gold product, purity, inventory and pricing configuration. Demonstrate the existing frontend/backend supports the reviewed scenario with no schema migration required solely because Gold launches. Gold's actual pricing, trial/handling policies and external reviews remain prerequisites for live activation.
- **10.8** Simulate Hyderabad activation: change `HYD` from `COMING_SOON` to `ACTIVE`; configure Hyderabad Hub, ServiceAreas, Inventory, TrialPlans and delivery agents. Demonstrate existing frontend/backend operation with Bengaluru and Hyderabad stock kept separate and no schema redesign solely for city activation.

**Frontend work:** Validate the complete mobile customer and staff journeys, accessibility, error/recovery states, attribution/events and accurate financial/status presentation. Run both expansion simulations through customer selectors, listings, cart, booking and authorized staff surfaces; do not only test database inserts.

**Admin/operations work:**

- **10.9** Write operational SOPs covering inventory receiving, product onboarding, packing, dispatch, arrival verification, customer trial, sale, payment failure, no purchase, return, QC, refund, damage, loss, and invoice correction/escalation. Link each SOP to approved policy, role permissions, evidence and exception handling; do not turn unresolved policy into an operating instruction. Keep shared SOP documentation canonical rather than copying it into both repositories.
- **10.10** Create and execute a production readiness checklist covering inventory reconciliation, concurrency testing, Razorpay test/live verification, webhook retry/idempotency testing, OTP flow, CA invoice review, Delivery Challan review, refund testing, backup restore testing, legal pages, support/grievance workflow and monitoring. Record evidence, reviewer, unresolved blockers and release authorization rather than treating a checklist's existence as readiness.

**Business/account prerequisites:** Reviewed business KPI definitions, cost inputs and reporting responsibilities; approved support/grievance ownership; operational staffing/training; approved production provider accounts/configuration; resolved decisions for every enabled flow. **BUSINESS DECISION BD-12/BD-13** and related pricing, service, stock and trial policies gate real expansion; **CA REVIEW** and **LEGAL REVIEW** evidence must cover enabled products/entities/locations and customer/financial documentation. Monitoring, analytics and backup service selection/retention must be explicitly approved or recorded if still open.

**Tests:** Validate the event-to-KPI mapping against known scenario data, deduplication, correct visit/box/purchase denominators and financial truth sources. Exercise concurrency, access/abuse controls, provider retries, notification failures, inventory reconciliation, approved purchase/refund exceptions and real backup restoration in an appropriate environment. Run both activation simulations without schema changes for activation alone. Check that audits and alerts detect meaningful faults, not merely that log statements exist.

**Definition of Done:** Readiness evidence demonstrates reliable operations, traceability, restoration and monitoring; KPIs are defined and reconcile to trusted facts; both expansion simulations pass; reviewed SOPs and customer/legal/financial materials exist. Outstanding gates are listed explicitly and block only the affected live capabilities. Production release and real Gold/Hyderabad activation require their own authorized readiness decisions.

**Non-goals:** Launch approval by implication, investor-grade claims unsupported by evidence, collecting unnecessary personal analytics data, replacing professional reviews, introducing microservices for hypothetical scale or inventing business metrics from unreliable event counts.

## Progress and change discipline

Keep task/subtask identifiers stable. When work is later authorized and starts, record actual status, responsible owner, linked implementation/review evidence and relevant decision IDs. Mark a task complete only when its Definition of Done is satisfied; distinguish implemented scaffolding, development simulation, reviewed policy, provider-account readiness and production activation. Update this canonical file and linked decision records when approved scope changes; do not create competing backend/frontend roadmaps.
