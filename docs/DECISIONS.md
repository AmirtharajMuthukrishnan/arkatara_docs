# Architecture and business decision log

Updated: 2026-10-04

## Authority and status

Approved business context and explicit owner instructions are binding. D-23 through D-26 record the owner's 2026-10-02 identity, booking-verification and roadmap decisions. Detailed entity fields and lifecycle refinements not settled by those decisions remain subject to implementation review and the open policy gates.

The owner approved the proposed Next.js/React + Django/DRF + PostgreSQL stack and authorized Task 1 implementation on 2026-09-19. Task 1 is verified complete. Task 2 was authorized on 2026-09-24 after that verification. The owner authorized Task 3 on 2026-09-25 and subsequently requested continuation. On 2026-09-26 the owner instructed that Tasks 1–3 remain complete for their delivered scope, made search discoverability a standing requirement and authorized continuing with the next pending task after the plan update if no blocking decision needs approval. Task 4 originally proceeded under D-20; D-26 now scopes it to cross-app model completion. Recorded delivery/CI and live-operation gates remain visible; no unresolved business, CA or legal decision is resolved by this authorization.

The owner's 2026-10-02 clarification (D-21) limits frontend work to design-independent technical foundations until the owner supplies approved UI/design references. This narrows the presentation scope of earlier continuation instructions without removing frontend requirements or blocking independent backend work.

On 2026-09-21, the owner explicitly requested finishing Task 1 only and keeping BD-14 pending. Root repository creation was deferred at that time. On 2026-09-23, the owner approved creating the separate documentation-only root repository described in BD-14 below, superseding that deferral. The owner subsequently published all three repositories; remote verification on 2026-09-24 confirmed the application fixes merged with green CI and the canonical documentation published. The latest Task 2 authorization supersedes the earlier Task 1-only scope.

The interrupted Task 1 verification resumed on 2026-09-23 under the same scope. Its implementation and test evidence are recorded in [TASKS.md](TASKS.md); this does not resolve any open policy or approve Task 2.

Examples and suggestions do not decide open rules. Open entries use exactly one classification: BUSINESS DECISION, CA REVIEW or LEGAL REVIEW. Commercial choices, accounting interpretation and legal permissibility require distinct outcomes.

Dates below record this documentation consolidation; they do not invent the date of a future legal, tax or operational approval.

## Finalized decisions

### D-27 — Persistent guest tracking and payment-start stock reservations

- Status: APPROVED implementation direction by the owner's 2026-10-04 instruction to implement the discussed checkout models within Task 4 and synchronize references. This extends D-24/D-25 without enabling workflows or settling commercial values.
- Guest continuity: Guest checkout remains account-free and permanently customer-unlinked. Mandatory booking tracking uses `GuestBrowserSession` (opaque credential digest, issuance/expiry/revocation/rotation) and `GuestBookingAccessGrant` (one booking, one browser session, scope and validity). Task 6 sends the raw random credential in a host-only Secure/HttpOnly cookie, with reviewed SameSite, CSRF and origin handling. Multiple grants support multiple bookings without overwriting earlier orders. Refresh does not erase a valid cookie; cookie loss/expiry needs recovery, not phone-based order enumeration.
- Recovery: A single-booking recovery secret plus a fresh `GuestRecoveryPhoneChallenge` can restore access to that booking. Recovery proof is distinct from booking OTP, account login and Arrival OTP. A recovery link alone or a mobile number alone must not reveal historical orders. Grant redemption and subsequent access issuance are Task 6 workflows; TTL, delivery and abuse limits remain reviewed configuration.
- Booking sequence: Draft intent and immutable selections/terms -> booking phone verification -> review -> explicit payment-start action -> atomic whole-selection reservation -> provider order/payment -> verified captured collection and allocation -> confirmed booking. Draft creation, OTP send and OTP verification never reserve stock. Seal the completed box/item/acceptance graph before advancing to VERIFIED; do not seal the booking header before inserting its contents.
- Stock: The payment context supplies the explicit expiry. `InventoryReservation` retains HELD/CAPTURE_PENDING/COMMITTED or release evidence, and `TrialItemAllocation` binds requested variants to physical units. PostgreSQL excludes incompatible live reservations. Starting capture protects the holds; ambiguous provider outcomes require reconciliation before release. Expiry does not delete booking tracking/history. A late or extra receipt remains financial evidence and cannot revive a released booking automatically.
- Payments: Implement a neutral `BookingCharge`, versioned `PaymentContext`, mutable `PaymentAttempt`, authenticated-event inbox, immutable `GatewayTransaction` and `PaymentAllocation`. Authorization, capture, business acceptance, revenue recognition and settlement stay separate. Integer minor-unit amounts and explicit currency are required; no deposit/fee or tax classification is selected. Manual authorization/capture is the intended integration path to evaluate, not proof every live gateway/account/method supports it.
- Reliability: Store principal/operation/key/fingerprint/result identity in `IdempotencyRecord`. Reuse the same key for retries of the same action; a changed request needs a different key. Durable `OutboxCommand`, `NotificationEvent` and `NotificationAttempt` retain intended effects and delivery progress. They do not send messages, validate gateway signatures or guarantee remote exactly-once processing by their existence.
- Migration: Preserve existing booking and grant IDs, contacts, expiry/revocation and each old token digest through an additive per-grant session backfill. No account linking, fabricated payment, policy activation or migration-history reset. Evidence-removing downgrade is refused after use; prefer compatible code rollback or reviewed forward recovery.
- Remaining gates: BD-02's hold trigger is now resolved to explicit payment initiation. Duration, late-payment/refund outcomes and retry/cancellation policy remain open. BD-01 amount/basis/classification, BD-03 quantities, BD-08 hub sourcing, BD-13 limits, BD-15 provider/operating settings and CA/LR reviews remain open. Checkout models are implemented and tested in Task 4; account-authentication, delivery, sale/invoice, refunds/settlement and other matrix coverage still need their remaining model work. Customer rollout remains disabled under D-23/R-37.
- Evidence: [Checkout implementation and recovery record](reviews/TASK4_CHECKOUT_MODELS_2026-10-04.md). No live deployment or legal/compliance certification is implied.

### D-26 — Task 4 cross-app model completion

- Status: APPROVED by the owner on 2026-10-02. Task 4 remains in progress, now with models-only scope across the backend domain apps. The documentation change itself implemented nothing. Subsequent owner instructions explicitly authorized coding the models whose decisions are clear; the [2026-10-03 foundation record](reviews/TASK4_MODEL_FOUNDATION_2026-10-03.md) documents the first partial implementation, not overall completion or approval of remaining policies.
- Decision: Complete the persistence design and implementation needed by the existing ten-task roadmap in Task 4: model definitions, relationships, constraints/indexes, historical-record protections, migrations and focused model/database tests. Include accounts, compliance, markets, catalog, inventory, trials, delivery, billing, payments, notifications and analytics. Reuse existing foundations and framework persistence; do not invent tables for packages that need none or consolidate well-organized model modules into one large file.
- Acceptance: Maintain the app-by-app coverage matrix and map each required model to its actual class/file, migration and tests. Verify fresh setup and upgrade from the current PostgreSQL schema, protected deletion/history and relevant database concurrency invariants. Missing commercial values may remain unconfigured; unresolved decisions that determine required schema shape block completion of the affected model scope. Empty classes or a completed `models.py` checklist are insufficient.
- Sequencing: Preserve all ten tasks and all 95 original subtask IDs. Add 4.M1–4.M12 for the model work. Move execution/acceptance of original storefront subtasks 4.1–4.7 into Task 5 alongside 5.1–5.9, retaining their IDs, existing source and dated verification. Tasks 6–10 implement workflows, authorization, APIs, providers, reporting and release checks using Task 4's models rather than deferring required tables to those tasks.
- Boundaries: Tasks 1–3 remain complete for their authorized scope. D-21 still gates presentation; it now gates the retained storefront work in Task 5, not Task 4's model-only acceptance. D-20's search requirements follow their feature owners. Customer/staff authentication persistence belongs to Task 4; customer authentication services belong to Task 6 and staff operations/authentication to Task 7. No migration deletion, database reset or application-code change is authorized by this documentation update.
- Future changes: This completes models for the agreed scope, not every hypothetical future requirement. New requirements or discovered integrity defects require reviewed additive migrations and a recorded scope amendment; do not hide known missing models in later tasks or promise that no future migrations will ever be needed.

### D-25 — Booking phone verification before upfront payment

- Status: APPROVED flow direction from the owner's 2026-10-02 instruction; commercial amounts, accounting classification and legal terms remain gated.
- Decision: Create pending booking intent, verify a purpose-bound booking phone OTP, retain the submitted booking details and applicable terms evidence, then initiate the upfront payment. Confirm the booking only after trusted backend payment evidence and valid eligibility/inventory acceptance. A pending row is not a confirmed order. Apply this booking verification to guest and registered checkout; an existing account session is not itself booking-specific verification.
- Challenge controls: Bind proof to the booking attempt, phone and purpose; enforce hashing, expiry, attempt/resend limits and atomic single-use consumption. A changed phone invalidates its previous proof. Retain verification evidence without plaintext OTPs. Booking verification, Arrival OTP and customer signup/login/recovery are separate purposes and cannot substitute for one another. Arrival OTP still starts the trial and uses one challenge across WhatsApp/SMS; no OTP is added for purchase selection.
- Meaning: OTP shows control of that communication channel at that time. It does not establish permanent identity, KYC, consent to every policy or automatic legal liability. Requester, recipient and payer can differ; retain the relevant transaction evidence without equating them.
- Partial resolution: BD-01 and BD-02 now require upfront collection before confirmation for the described launch flow. Whether that collection is a deposit/fee for legal/accounting purposes, its amount/basis, application/refund/retention, stock-hold timing/duration, expiry and late-payment handling remain unresolved. CA-01 and LR-01 are not resolved. Do not substitute a zero amount or an invented default.
- Delivery: Task 4 owns challenge, verification, acceptance and payment persistence; Task 6 owns the secure booking workflow and provider contracts; Task 7 owns Arrival OTP; Task 8 integrates verified collection and Task 9 messaging. No live launch using fake OTP delivery or fake payment evidence.

### D-24 — Unified bookings and durable transaction history

- Status: APPROVED order architecture direction on 2026-10-02. Specific field names and unresolved business cardinalities remain subject to review; the external `arkatara_order_models.py` draft is not approved wholesale.
- Decision: Use one `TrialBooking` for guest and account checkout, with an immutable checkout mode, a nullable protected `CustomerAccount` reference and historical booking contact/address facts. Guest mode always has a null customer reference; account mode gets its customer from trusted authentication. Account closure must not cascade-delete bookings or convert account bookings into guest bookings.
- Ownership: Guest bookings remain permanently unlinked to registered accounts, including later accounts with the same phone/email and guest checkout chosen while signed in. No automatic association, claim/import endpoint or indirect customer-history linkage through consent/audit records. Guest access is booking-specific; account history is ownership-based, never a phone-number lookup.
- Structure: Preserve `TrialBooking -> TrialBox -> TrialBoxItem -> InventoryUnit allocation`. A box has one category, requests identify variants and allocations identify physical units. One normal booking is one combined home visit. Do not replace these distinctions with a generic order-item-to-unit relationship or independent box checkouts.
- Operations: Preserve assignment history and visit outcomes separately from selected purchases, payment and stock custody. Agents may record selected pieces or no purchase; backend rules control prices and confirmed sale. Visit closure never automatically settles money, refunds funds or returns stock to availability. Hub receipt and QC remain required.
- Evidence: Keep working payment attempts separate from immutable verified receipts/events, allocations, refunds, settlement batches and bank matching. Preserve issued invoice headers and line membership/content, currency, issuer, customer and price/tax snapshots. A generic append-only ledger or mutable profile reference is not a substitute. Use database constraints, protected history and transaction-safe services at their owning layers.
- Continuity: Later customer-account/mobile-app enablement uses these existing records; it does not require a new transactional database or deletion/reassignment of older orders. Guest transactions remain business/accounting evidence, but are not registered-customer order history. Investor reporting must distinguish visits, purchases, payments and revenue, and must not count phone numbers as proven unique people.
- Review limitation: The supplied draft does not satisfy these requirements unchanged: identity can be reassigned, contact completeness and cross-row stock/state integrity are insufficient, and payment/invoice/settlement history needs separation. DATA_MODEL.md defines the target coverage; later implementation must supply actual tests, not rely on the draft's comments or state setter.

### D-23 — Separate staff and customer identities with permanent guest checkout

- Status: APPROVED by the owner on 2026-10-02 after considering shared versus separate identity tables. The selected direction is separate tables and authentication boundaries, not a requirement to write two custom cryptographic systems.
- Staff: Keep `accounts.StaffUser` as Django's sole `AUTH_USER_MODEL`; it serves staff provisioning, Admin permissions and staff attribution. Delivery agents are staff roles with operational profiles where needed. `is_staff` controls Admin access, not the business definition of a delivery agent. Enforce action permissions, market/hub/assignment scope and staff MFA before live access.
- Customers: Add a separate `CustomerAccount`, not Django's auth model and never a source of staff privileges. Prepare identity/authentication support before launch but keep customer signup/login/account checkout disabled server-side initially. Later activation, tentatively around six months, requires explicit readiness approval rather than an automatic date. Guest checkout remains available permanently; guest phone verification never creates an account.
- Authentication: Staff and customers use separately scoped authentication and explicit principal-type authorization. If tokens are used, validate their issuer/audience/type and revocation semantics; customer credentials must never authorize staff endpoints. A generic `IsAuthenticated` check is insufficient. Reuse reviewed authentication components; provider choice and JWT-versus-session/cookie transport are not selected by this decision.
- Routing: Use a dedicated staff frontend host and `/staff/api/v1/` API namespace. The owner requested `staff.arkatara.com`; the previously approved public domain remains `arkatara.in`. Confirm ownership, DNS/aliases and cross-origin/cookie deployment configuration before launch rather than silently changing either domain. Separate hosts/routes do not replace backend authorization and do not require another backend/database.
- Independent identities: A person may independently be staff and customer, even with the same phone/email. Never promote, merge or link those identities by contact matching. Staff deactivation and customer account closure have different workflows; both preserve required historical evidence.
- Privacy/security: Account-free does not mean anonymous to the business. Guest names, phones and addresses remain personal data. Phone and email are contact methods, not permanent proof of personhood. Recycled-number/account-recovery risk also affects registered OTP login and must be addressed before customer-account activation; phone OTP alone must not grant a new number-holder old account history. Recovery/reverification design, retention and privacy wording remain explicit security/LR-04 readiness dependencies.
- Supersession: Earlier proposals for a shared customer/staff auth table or later claiming guest orders are not the selected design. The dated customer-account readiness review retains its implementation observations, but its former shared-auth and guest-claim recommendations are superseded by this decision and D-24.

### D-22 — AWS live infrastructure architecture for first six months

- Classification: TECHNICAL ARCHITECTURE DECISION.
- Status: APPROVED hosting direction from the owner on 2026-10-02 for the first six months of live operation. This selects the target AWS shape; actual account setup, DNS, certificates, secrets, CI/CD credentials, production data migration and launch verification remain implementation work.
- Decision: Host the public application in one primary AWS Region, preferably Asia Pacific (Mumbai) `ap-south-1` for Indian customer latency and data locality, unless account/service availability or measured latency forces a documented exception. Use a managed, multi-AZ architecture: Amazon CloudFront plus AWS WAF at the edge, AWS Amplify Hosting for the Next.js SSR frontend, Amazon ECS on AWS Fargate behind an Application Load Balancer for the Django/DRF backend, Amazon Aurora PostgreSQL-Compatible or Amazon RDS for PostgreSQL in private subnets for the database, Amazon ElastiCache for Redis for cache/rate-limit/session/queue support where needed, Amazon S3 for private and public media/object storage, Amazon ECR for backend images, AWS Secrets Manager/SSM Parameter Store for secrets, Amazon CloudWatch for logs/metrics/alarms and AWS Backup/RDS automated backups for recovery.
- Backend path: Run the Django modular monolith as horizontally scalable Fargate tasks in private subnets. Expose only the ALB through HTTPS, route API traffic under `/api/v1/`, and keep Admin/staff routes protected by backend authentication, least-privilege IAM/security groups and non-indexing controls. Run background jobs as a separate ECS service or scheduled ECS tasks using the same application image; do not introduce microservices, Kubernetes or an event-streaming platform for the pilot.
- Frontend path: Deploy the Next.js storefront/staff frontend through Amplify Hosting with SSR/SSG support and the `arkatara.in` custom domain. Keep public catalogue pages server-rendered and cacheable where truthful; keep private/staff/customer-specific pages uncacheable and non-indexable. CloudFront/WAF remains the public edge for TLS, caching, bot/rate controls and host normalization.
- Database path: Production data lives in private PostgreSQL with automated backups, point-in-time recovery, encryption at rest and restricted runtime roles. Start with one writer plus a standby/high-availability posture appropriate to budget; add an Aurora/RDS read replica only when observed read load requires it. Inventory, booking, payment and invoice consistency remain in the relational database; do not split data stores by domain.
- Storage/media path: Store original private catalogue/media objects in S3 with block-public-access. Publish only reviewed public derivatives through CloudFront/S3 using origin access control and explicit cache policy. No private draft media, unit identity, customer documents or provider evidence is exposed through public object URLs.
- Environments: Maintain at least staging and production as separate AWS environments with separate databases, buckets, secrets, provider credentials and domain names. Local development remains local. Staging may use smaller instances and lower task counts but must match security boundaries closely enough to validate migrations, callbacks, media, redirects, SEO controls and operational runbooks.
- First-six-month sizing principle: Start small but production-safe, then scale from measured traffic. Use autoscaling on ECS services, ALB/CloudFront/WAF metrics, database CPU/connections/IOPS, Redis memory/evictions and application queue depth. Increase task counts, add read replicas, tune cache TTLs and raise WAF/rate limits deliberately from evidence rather than pre-buying a large fixed fleet.
- Operations: Use infrastructure as code, preferably AWS CDK or Terraform, before production. Capture CloudWatch alarms for 5xx rate, latency, task health, database saturation, Redis saturation, backup failure, certificate expiry, queue backlog, payment webhook failures and storage errors. Verify restore drills before launch. Keep rollback/forward-fix procedures documented.
- Alternatives considered: AWS App Runner for the backend, EC2 instances, Kubernetes/EKS, Lambda-first APIs, Vercel/third-party frontend hosting and a multi-region active-active launch. ECS Fargate plus managed AWS data services is selected because it fits the existing Django monolith, supports predictable scaling and operational control, avoids premature platform complexity and keeps the first-six-month spend adjustable.
- Consequences: Update the architecture docs from “hosting unselected” to this AWS target. Task 10 still owns deployed HTTPS, preferred-domain, robots/sitemap, redirect, search-console, Core Web Vitals, backup/restore and launch-readiness evidence. This decision does not resolve BD-01..BD-15, CA-01..CA-03 or LR-01..LR-04, and does not approve live payment, messaging, legal copy, tax configuration or frontend visual design.

### D-21 — Frontend visual design awaits owner input

- Subsequent sequencing change, 2026-10-02: D-26 transfers original storefront 4.1–4.7 to execution under Task 5. References below to Task 4 presentation describe the original scope; the design gate remains binding there and on all other frontend work. Current Task 4 has models-only acceptance.
- Classification: BUSINESS DECISION.
- Status: APPROVED scope clarification from the owner on 2026-10-02. The design gate is binding; final UI design, visual direction, branding system and screen-level references have NOT yet been provided or approved.
- Decision: Do not independently design or finalize frontend presentation from task descriptions, broad brand aspirations or existing provisional screens. The gate covers the customer landing, category and product pages, cart, checkout, staff portal and related interfaces throughout the existing roadmap.
- Work permitted before design input: Necessary routes, data contracts, server-rendering structure, metadata/SEO architecture, API integration boundaries, non-indexing and privacy controls, accessibility-friendly semantic HTML and tests. Use only minimal unstyled or clearly temporary functional placeholders where unavoidable. Backend and other design-independent implementation may continue within their authorized scope and existing policy gates.
- Work awaiting owner input: Polished layouts, visual hierarchy, typography, colours, brand assets/system, screen composition and final customer-facing presentation. SEO or a successful technical test/build does not authorize choosing these. Record the owner's approved design reference and its applicable screen scope before implementing that presentation; do not infer approval for other screens.
- Existing work: The visual styling, page composition and presentation copy already produced during Task 4 are unapproved provisional work, not accepted branding or a final design. Do not continue polishing or treat them as an approved baseline. This documentation update does not remove or replace existing source files or erase technical verification history.
- Acceptance and sequencing: Keep the ten tasks and original subtask IDs unchanged. Frontend requirements remain in their owning tasks; report technical progress separately from design-dependent work. A task whose Definition of Done includes reviewed presentation is not fully complete until the relevant approved design is implemented and validated. Task 4 remains in progress with its visual/UI portion awaiting owner design input.
- Authority: This explicit clarification takes precedence over any earlier wording that could imply independent visual-design authorization under D-20 or the roadmap. D-20's search requirements and all unresolved business, CA and legal gates remain in force. The pending design input is a BUSINESS DECISION dependency, not permission to invent a design.

### D-20 — Standing search discoverability and Task 4 continuation

- Subsequent sequencing change, 2026-10-02: D-26 makes Task 4 model completion and transfers original 4.1–4.7 storefront execution to Task 5 with IDs/evidence retained. Historical Task 4 catalogue references below describe the earlier scope; search obligations remain in force under the updated ownership map.
- Status: APPROVED by the owner on 2026-09-26; applies to every remaining task. Task 4 implementation is in progress following the required plan review and lightweight architecture check; development and verification continued on 2026-09-27 under the same authority.
- Decision: Treat SEO/search discoverability as a normal cross-cutting development requirement within the existing Tasks 4–10. Preserve the ten-task plan and all original subtask IDs; do not create Task 11, replace the roadmap or reopen Tasks 1–3 for minor SEO improvements. Put implementation and acceptance checks with the feature that owns them; use the [single ownership map](TASKS.md#standing-search-discoverability-requirement).
- Brand/domain: Arka Tara's intended official public domain is `arkatara.in`. Support branded Arka Tara/arkatara and Bangalore/Bengaluru search intent, plus relevant non-branded Silver/S925 jewellery, assisted home-trial and category intent through natural useful content. This records brand/web identity, not legal-entity identity, domain-account access, DNS configuration or deployment approval. BD-11/LR-02 remain open.
- Content and truth: No ranking guarantee, keyword stuffing, doorway pages, near-identical location combinations, fake ratings/reviews/testimonials/locations/stock or unsupported superiority/purity claims. Structured data must describe verified visible public content. LocalBusiness markup is conditional on appropriate confirmed facts; an attended home-trial service does not imply a public shop address.
- Architecture: Preserve clean stable product/category URLs, canonical identity, server-rendered public content, factual metadata/schema, useful linking, mobile performance and extensibility across categories/materials/markets. The user's sample paths are illustrative, not mandatory. SEO does not authorize disclosure of private catalogue/media drafts, customer records, payment documents or staff data; indexing controls do not replace authorization.
- Lightweight Tasks 1–3 review: No expensive SEO architecture blocker found. Next.js App Router supports server rendering and metadata; existing public UUIDs and stable slug-compatible codes support public routes without replacing identity; material/market references are data-driven; media already retains descriptive/alt-text capability and provider-neutral identity. Public-page slugs/metadata, responsive published media and catalogue rendering are natural additive Task 4 work. The private draft/pricing/media boundaries remain intact.
- Reason: Search and customer usability depend on page, data and URL decisions made during feature work; adding them only after launch risks avoidable rework. Future Gold/city expansion must not require rebuilding the catalogue.
- Alternatives considered: A separate SEO Task 11, reopening completed foundations, or implementing every launch feature immediately. Retaining the plan with explicit ownership and staged acceptance criteria matches the owner's instruction and avoids duplicate work.
- Consequences and authority: With no serious architecture blocker identified, continue the next pending task, Task 4, immediately. No major foundation redesign is approved. A later serious architecture change must first explain its problem, future cost, recommendation and effect on existing functionality. Robots/sitemaps, deployed domain/redirect checks, search-console verification/submission, indexation and query/Core Web Vitals monitoring belong to Task 10. Existing business, account, privacy, tax, legal, publication and live-operation gates remain binding; independent storefront work may proceed without inventing their answers.
- Subsequent clarification, 2026-10-02: D-21 gates visual/UI design pending owner-provided approved references. This continuation and its search requirements authorize design-independent frontend foundations, not independent polished presentation.

### D-19 — Task 3 back-office and onboarding boundary

- Status: Technical implementation under the owner's Task 3 authorization, 2026-09-25; verification continued 2026-09-26. This records implementation choices, not approval of unresolved operating policies.
- Reference Admin supports audited draft creation and permitted descriptive edits, immutable identity and stale-form protection. Activation, pricing approval and configuration approval are separate workflows; generic forms cannot approve them.
- Inventory registration creates an unverified DRAFT identity at an intended hub. Actual receipt records HUB custody and QC_PENDING; a recorded inspection can produce AVAILABLE or QUARANTINED. Receiving and inspection require separate permissions and effective approved procedure references scoped exactly to the hub UUID. No procedure values are seeded. This gated onboarding route does not decide BD-09's receiving/QC criteria or require every future stock source to follow one immutable process.
- Registration, receipt and QC produce atomic, attributable movements/audits with request fingerprints and idempotency. Movement history is append-only at ORM and PostgreSQL UPDATE/DELETE boundaries. Known custody is distinct from legal ownership. Earlier draft units are retained without fabricated receipt or registration history. Downgrade is blocked before removing movement protection once stock evidence exists.
- CSV onboarding supports draft product/variant definitions and individually identified physical drafts. Preview writes nothing; apply rechecks authorization, references and conflicts under locks and commits the file atomically. Identical retries return the recorded outcome; changed requests cannot reuse an identifier. Limits of 500 rows and 1,000,000 bytes are technical request bounds, not commercial stock or Trial Plan limits.
- Product media supports main/additional images, video and associated thumbnails, immutable asset identity and audited metadata/retirement. Uploads remain private drafts. The provider-neutral interface has only an explicitly configured local/test adapter; production provider, allowed MIME types and byte limit remain unset under BD-15. No public media URL or storefront publication is enabled. Frontend work is limited to the versioned metadata contract.
- Alternatives considered: editable stock status dropdowns, direct CSV database writes, automatic availability on import and a hard-coded public storage vendor. Explicit services, retained outcomes and private unconfigured storage preserve evidence and unanswered choices.
- Remaining BUSINESS DECISION gates: BD-09 procedures, tagging and exception/disposition; BD-08 allocation; BD-11 ownership; BD-15 production storage and upload settings. CA REVIEW CA-03 and LEGAL REVIEW LR-02 remain necessary for ownership arrangements, with LR-04 for applicable retention/access policy. Existing configuration Admin only authors drafts; reviewed activation/supersession must be implemented before live receiving/QC. Reservation, dispatch, sale, customer returns, transfers, reinspection and retirement actions are not implemented by this scope.

### D-18 — Selective implementation traceability

- Status: APPROVED by the owner on 2026-09-25, applicable to existing crucial boundaries and future work in both applications.
- Decision: Add concise rule/decision references beside important business validations, authority boundaries, unresolved-policy gates and history-preservation code. Use the existing stable IDs; refer to a business-context section only when it adds necessary narrative without an equivalent rule/decision.
- Maintenance: Keep the canonical task-to-file/symbol map in [TASKS.md](TASKS.md#implementation-reference-map). The [root operating guide](../AGENTS.md#business-reference-comments) defines the convention; both application guides reference it. Update affected references alongside implementation or policy changes.
- Reason: Make the business purpose discoverable while limiting duplicate explanations and references that become outdated. Routine code does not need task headers or a reference on each line.
- Scope: Documentation and developer comments only. This instruction does not authorize Task 3, change application behavior, resolve open business/CA/legal choices or imply that referenced future workflows are implemented. Applied migrations remain unchanged.

### D-17 — Task 2 reference, pricing and evidence boundary

- Status: Technical implementation under the owner's Task 2 authorization, 2026-09-24; verification continued 2026-09-25. This is not approval of any unresolved commercial value.
- Decision: Use stable UUID/code identities; separate Market/Hub/ServiceArea, Material/Purity/Category, Product/ProductVariant/InventoryUnit; use explicit draft states and read-only public reference/coverage endpoints.
- Coverage is exact by selected market, country and postal identifier. Effective overlapping approved rows produce CONFLICT. No PIN-to-hub precedence, intercity fallback or booking authorization is inferred. Category eligibility uses the existing approved-configuration lookup scoped by market/material/category public UUIDs, with no global fallback.
- Pricing: immutable DRAFT PriceRevision rows represent FIXED/WEIGHT_BASED inputs. NUMERIC(24,6) is storage capacity only; reject excess precision, binary floats and nonfinite input rather than rounding. Currency, units and evidence are explicit. No live formula, rate provider, price selection or commitment timing is implemented.
- HistoricalPriceSnapshot is a versioned immutable value contract containing copied identity/descriptions, source price revision, exact supplied amount/inputs and separately explicit tax evidence. Original handoff: Tasks 6–8 were to persist it in transactions. D-26 now moves that persistence/database protection to Task 4; Tasks 6–8 still bind snapshots at the approved commitment point. Task 2 does not create placeholder purchases or invoices.
- InventoryUnit has only DRAFT status in this foundation. Task 3 must introduce audited receiving/movement/transition workflows; hub means operational location, not legal ownership. Generic saves cannot reassign a unit or redefine a variant already referenced by units or prices.
- Alternatives considered: hard-coded city/material enumerations, global postal uniqueness, a live fixed-price default, early transaction tables and assumed Gold formulas. These would force policies or scope that are not approved.
- Remaining gates: BUSINESS DECISION BD-04/BD-08/BD-09/BD-11/BD-12/BD-13; CA REVIEW CA-02/CA-03; LEGAL REVIEW LR-02, plus other decision dependencies of later workflows. No existing unresolved entry is closed by this technical implementation.

### BD-14 — Canonical documentation source control

- Classification: BUSINESS DECISION.
- Status: RESOLVED — APPROVED by the owner on 2026-09-23, effective immediately for local documentation tracking.
- Original question: How the canonical shared root docs and root AGENTS.md will be tracked, reviewed and distributed alongside two existing separate Git repositories.
- Decision: Create a separate Git repository at `ARKA TARA/`. Track the nine canonical `docs/*.md` files, root `AGENTS.md`, a concise repository `README.md`, `.gitignore` and `.gitattributes`. Exclude the two application repositories, `.tools/`, databases, secrets and other unlisted files. Preserve the existing application histories and remotes; do not introduce submodules or duplicate shared documentation.
- Review/distribution approach: Use the approved main/development/feature workflow for documentation changes. Reference companion documentation commits or PRs from dependent application work. A future developer obtains the documentation repository and places the two independent application clones beneath it so `../docs/` remains valid. Root README documents the setup.
- Reason: Keep one canonical context with its own reviewable change history, while preserving the existing separate backend/frontend repositories.
- Alternatives discussed: Keeping the shared documents untracked while the decision was pending; a separate documentation repository at the workspace root (selected). Duplicating shared files remains prohibited by D-01.
- Approval evidence: The owner confirmed the proposed outer-level repository boundary, then instructed: "lets create a new repo for the ARKA TARA/ folder". This records a repository-maintenance choice, not an investor endorsement or a requirement to choose a particular repository count.
- Consequences and remaining work: The owner published [arkatara_docs](https://github.com/AmirtharajMuthukrishnan/arkatara_docs); remote verification completed on 2026-09-24. Application repositories remain independent. Task 1 verification and the later Task 2 authorization are recorded in TASKS.md; neither resolves other business/CA/legal entries.
- History: Proposed and deferred on 2026-09-21; approved on 2026-09-23. Prior no-initialization guidance is superseded only for this explicitly scoped root documentation repository.

### D-16 — Task 1 technology baseline

- Decision: Implement the foundation with Node.js 24 LTS, Next.js 16.3.5, React 19.3.0, TypeScript 6.0.3, Python 3.13, Django 5.2.17 LTS, DRF 3.18.1 and PostgreSQL 18.
- Status: APPROVED stack and AUTHORIZED Task 1; exact compatible versions selected during implementation on 2026-09-19.
- Reason: Use supported releases with a long-lived Django baseline and current PostgreSQL support while preserving framework compatibility.
- Alternatives considered: Django 6.1 was current but Django 5.2 is the current LTS; TypeScript 7 and ESLint 10 were tested but rejected because the current Next.js lint toolchain does not yet support them.
- Consequences: Dependency lockfiles are authoritative. Upgrades require passing lint, type, migration, PostgreSQL, test and production-build checks. Hosting is selected under D-22 as an AWS managed-services architecture for live operation. Documentation tracking is now resolved under BD-14; remote publication was verified on 2026-09-24.
- Recorded: 2026-09-19; implementation verification continued 2026-09-20.

### D-01 — One canonical shared documentation set

- Decision: Keep shared project/business documentation only in root docs/, with root AGENTS.md and short repository-specific AGENTS.md pointers.
- Status: APPROVED — latest explicit instruction.
- Reason: Future developers and agents must recover one consistent source of truth.
- Alternatives considered: Duplicating full files in both repositories (previous arrangement); canonical shared directory (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: The four old shared copies were superseded after preservation checks. Root documentation tracking follows the separately approved BD-14; the single-copy rule remains binding.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-02 — Modular monolith and disciplined simplicity

- Decision: Prefer a modular monolith with strong relational modelling, clear responsibilities and simple operations.
- Status: APPROVED — business principle and roadmap.
- Reason: Pilot scale does not justify distributed operational overhead; growth still needs clean boundaries.
- Alternatives considered: Microservices/Kubernetes/event streaming now; unstructured single-domain application; modular monolith (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: No separate services without a demonstrated later need. Technical module ownership is proposed in ARCHITECTURE.md.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-03 — Booking, category boxes and one normal visit

- Decision: Use TrialBooking -> TrialBox -> TrialBoxItem; normally one customer home visit with multiple single-category boxes fulfilled together within the selected city.
- Status: APPROVED — business clarification.
- Reason: The customer is booking an attended trial of several selections, not independent purchases of boxes.
- Alternatives considered: One checkout per box; category-mixed boxes; one combined booking with category-specific boxes (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: No automatic separate checkouts or fee/payment cardinality from box count; BD-03, BD-07 and BD-08 remain open.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-04 — Hub inventory within distinct city pools

- Decision: Physical stock belongs to hubs in a market; availability and reservation respect the selected city.
- Status: APPROVED — roadmap refinement.
- Reason: Expansion requires separate operational inventory and traceable custody.
- Alternatives considered: Global undifferentiated stock; implicit intercity fulfilment; hub-specific city inventory (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: Same-city multiple-hub sourcing remains BD-08; no automatic cross-city fallback.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-05 — Guest customer experience and non-reserving cart

- Subsequent clarification, 2026-10-02: D-23/D-24 retain guest checkout permanently alongside optional registered checkout and prohibit later guest-account linking. D-25 adds booking phone verification without requiring an account.
- Decision: No required customer account; localStorage preserves temporary trial selections, and cart actions never reserve inventory.
- Status: APPROVED — business and roadmap.
- Reason: Reduce browsing friction without letting untrusted browser state commit scarce physical stock.
- Alternatives considered: Registration wall; cart-triggered reservation; guest cart with backend revalidation (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: Backend revalidates booking inputs. Seven-day expiry and two-box limit remain suggestions under BD-13.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-06 — Material, purity and product identity

- Decision: Separate Material/Purity and Product/ProductVariant/InventoryUnit; support FIXED and WEIGHT_BASED pricing structure.
- Status: APPROVED — roadmap refinement.
- Reason: A design is not an individual piece, and Gold is an intended expansion.
- Alternatives considered: is_silver flag; one row for design and physical stock; distinct concepts (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: Purity range, pricing formula, price-lock timing, tagging and mixed-material rules remain open.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-07 — Initial availability and configuration-driven expansion

- Decision: BLR/Bengaluru and Silver are ACTIVE; HYD/Hyderabad and Gold are COMING_SOON.
- Status: APPROVED — explicit initial scope.
- Reason: Launch lean while making the intended product/market direction visible.
- Alternatives considered: Hard-coded single city/metal; separate future storefronts; data-driven activation (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: Coming-soon offerings cannot book. Gold/Hyderabad launch alone must not require schema redesign.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-08 — Staff workflow and one Arrival OTP

- Subsequent clarification, 2026-10-02: D-23 specifies the separate staff identity/host/API boundary. D-25's pre-payment booking OTP is distinct from Arrival OTP; the no-purchase-selection-OTP rule is unchanged.
- Decision: Use dedicated authenticated staff UI for assigned bookings; backend Arrival OTP verification starts the trial; no second purchase OTP.
- Status: APPROVED — business and roadmap.
- Reason: Give agents an appropriate doorstep workflow and preserve a meaningful arrival check.
- Alternatives considered: Using Django Admin at the doorstep; second selection OTP; dedicated portal and one arrival challenge (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: Agents cannot change authoritative prices. WhatsApp/SMS carry the same code; exact challenge/resend settings remain BD-15.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-09 — Payment authority and shared customer access

- Decision: Backend verification is authoritative; final-payment QR and URL access the same PaymentAttempt; webhook processing is idempotent.
- Status: APPROVED — business and roadmap.
- Reason: Prevent false confirmation and duplicate collection caused by two presentation channels.
- Alternatives considered: Screenshots/client callbacks as proof; independent QR/link charges; one verified payment context (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: Provider capability/mapping still needs validation. Changed selections, retries and uncertain-payment handover remain BD-05/BD-15.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-10 — Deposit and sale separation

- Subsequent partial resolution, 2026-10-02: D-25 requires upfront collection before confirmation, preceded by booking phone OTP. The original policy-unselected wording below is historical for whether collection is required; amount, basis, legal/accounting classification and application/refund/retention remain open.
- Decision: Keep deposit collection/application distinct from jewellery selling value and invoice total.
- Status: APPROVED — invariant; policy NOT decided.
- Reason: Prepaid money is not automatically a price reduction.
- Alternatives considered: Netting deposit out of product price; separately recording consideration and prior funds (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: Task 8 plans deposit capability; whether, how much and on what terms to charge remains BD-01/CA-01.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-11 — Financial history and legal-entity continuity

- Decision: Control invoice numbers, preserve immutable issued invoices and price/tax/entity snapshots, trace payments/refunds/settlements and audit privileged actions.
- Status: APPROVED — business and roadmap.
- Reason: Enable clean accounting, review and future company transitions without rewriting history.
- Alternatives considered: Recomputing old totals from catalogue; overwriting issuer details; preserving historical evidence (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: Entity identity, document rules and accounting treatment remain BD-11, CA-01..03 and LR-02..03.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-12 — Return custody and QC

- Decision: Unpurchased dispatched jewellery must physically return to the hub and pass QC before availability; movements are append-only.
- Status: APPROVED — business and roadmap.
- Reason: Protect inventory accuracy and readiness for the next trial.
- Alternatives considered: Automatic restock on cancellation/visit closure; physical return plus QC (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: Damage, loss, failed QC and after-sale returns need BD-09/BD-10 policies; SOLD history is not erased.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-13 — Back office and integration sequencing

- Decision: Use Django Admin for initial operations; core booking/OTP/payment boundaries use development fakes before Razorpay in Task 8 and messaging in Task 9.
- Status: APPROVED — explicit roadmap.
- Reason: Keep core business logic independent of external account readiness and avoid an early custom operations app.
- Alternatives considered: Building all integrations first; a separate initial back-office application; phased integration (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: No fake collection or messaging may imply a live-pilot policy. BD-06 remains open; Task 1 excludes payment/messaging integration.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-14 — Security and engineering discipline

- Decision: Use separate local/staging/production secrets, /api/v1/, main/development/feature branches, formatting/linting/CI/tests, migration discipline and auditability.
- Status: APPROVED — roadmap requirements.
- Reason: Reliable delivery and control are foundational rather than later fundraising additions.
- Alternatives considered: Deferring all security/audit work until Task 10; adopting baseline controls early (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: Task 10 verifies and hardens controls. No project scaffolding, CI or infrastructure is implemented by these documents.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

### D-15 — Service messaging and marketing are distinct

- Decision: Do not infer marketing consent from a purchase/service phone number; keep transactional/service messaging separate.
- Status: APPROVED — explicit roadmap.
- Reason: A booking phone number has a defined service purpose.
- Alternatives considered: Treating any phone number as marketing consent; separate purposes and reviewed consent (selected). These describe design tradeoffs, not a claim that the owner separately evaluated every alternative.
- Consequences: Final wording, templates, channel eligibility and retention require LR-04 and operational settings BD-15.
- Recorded: 2026-09-19; source: approved business context and/or the explicit roadmap and documentation instruction.

## Proposed technical decisions requiring document review

| ID | Decision proposed | Reason | Alternatives considered | Consequences/status |
| --- | --- | --- | --- | --- |
| P-01 | Use Next.js/React, Django/DRF and PostgreSQL | Owner's preferred stack and relational modular-monolith direction | Another stack has not been evaluated or authorized | **APPROVED 2026-09-19**; exact supported versions are selected during Task 1; AWS live hosting is selected under D-22 |
| P-02 | Use the module ownership and in-process workflow boundaries in ARCHITECTURE.md | Preserve business domains without service proliferation | Direct cross-module writes; separate services; coordinated monolith | Modular-monolith direction approved; D-23–D-26 refine identity and ownership. Task 4 completes persistence; later tasks implement workflows. |
| P-03 | Use the conceptual entities, constraints, reservations and evidence records in DATA_MODEL.md | Cover the supplied list while protecting physical and financial history | Single generic Order/Product record; domain-specific records | D-24 approves booking/history direction and D-26 assigns all agreed model scope to Task 4. Existing implementation is distinguished from target coverage; unsettled fields/cardinalities still require review. |
| P-04 | Keep visit progression, financial outcomes and return/QC outcomes distinct | One flat status cannot truthfully represent independent payment/custody facts | Unqualified COMPLETED or NO_PURCHASE terminal booking states; separate lifecycle facts | Separation approved by D-24; exact named states and policy-dependent guards in STATE_MACHINES.md are not silently approved commercial policy. |
| P-05 | Use provider interfaces and development fakes before later live integrations | Separate business correctness from gateway/messaging availability | Direct provider SDK use throughout domains; bounded adapters | PROPOSED FOR OWNER REVIEW in INTEGRATIONS.md; exact gateway-product mapping remains BD-15 |

## Business and review decision register

Status: BD-14 IS RESOLVED; BD-01/BD-02 ARE PARTIALLY RESOLVED BY D-25; OTHER OPEN QUESTIONS REMAIN UNRESOLVED. D-22 selects AWS hosting/storage direction without resolving all BD-15 provider/operational settings.

**2026-10-02 partial resolution:** Upfront collection before booking confirmation is now the approved launch direction, preceded by booking phone OTP. Original BD-01/BD-02 questions below are retained for history, not permission to omit that sequence. Amount, basis, classification, application/refund/retention, stock-hold trigger/duration, expiry and late-payment treatment still require decisions and applicable CA/legal review.

**2026-10-04 further partial resolution:** D-27 now fixes the hold trigger at explicit payment initiation, after booking OTP. Earlier statements that the trigger remains open describe the earlier checkpoint. Timeout values, provider capture capability, safe reconciliation and commercial expiry/late-payment outcomes remain gated.

BD-14 was deferred on 2026-09-21 and approved on 2026-09-23. Its original ID/question remain in the register to preserve the history. The owner subsequently published the documentation remote; verification completed on 2026-09-24.

The original 19 entries and their IDs are preserved verbatim below. The roadmap supplies additional structure, but it does not silently choose the outstanding policies.

| ID | Classification | Question or scope | Flexibility to preserve |
| --- | --- | --- | --- |
| BD-01 | BUSINESS DECISION | Whether a trial deposit is charged; whether it applies per box, plan or booking; its amount and application to purchases; treatment after no purchase, partial purchase, purchase below the deposit amount, cancellation, rescheduling or no-show. | Keep the trial offering, jewellery value, money collected, credit applied and any refund or retained amount distinguishable. Box count must not silently determine the deposit rule. |
| BD-02 | BUSINESS DECISION | When a booking is accepted and stock is reserved; whether deposit payment is a prerequisite; hold duration; cancellation and expiry; stock release; treatment of payment received after expiry. | Keep customer intent, booking commitment, stock availability and payment facts distinct. Do not assume a payment event always establishes a valid reservation. |
| BD-03 | BUSINESS DECISION | How designs, sizes, variants and physical pieces count toward Trial Plans; permitted quantities; repeated boxes of the same category; mixing Silver and Gold in a category box or booking; substitutions and customer agreement. | Preserve category-specific boxes and the distinction between a selected offering and the physical stock supplied. Do not interpret the approved single-category rule as a single-material rule. |
| BD-04 | BUSINESS DECISION | Whether price is committed at booking or at the visit; how changed prices are disclosed and accepted; Gold pricing policy. | Keep prices displayed, commitments made and final agreed sale values distinguishable. Preserve transaction history without choosing a price-lock point or pricing formula. |
| BD-05 | BUSINESS DECISION | Handling pending, failed or unverifiable payments; the point of physical handover; changed selections after a payment request; retries and further attempts. | Keep selection, amount due, payment outcome and physical custody distinct. Preserve the approved single underlying payment context for QR/link access without forbidding legitimate future retries. |
| BD-06 | BUSINESS DECISION | Permitted payment methods and trusted confirmation procedures for a live pilot while payment-gateway integration is deferred. | Keep collection method and verification evidence explicit. Do not silently assume cash, manual UPI, a screenshot-based process or a completed gateway integration. |
| BD-07 | BUSINESS DECISION | Service areas and visit-slot allocation, visit timing, rescheduling, cancellations, failed visits and no-shows; any exceptional repeat or split visit. | Preserve the normal one-booking/one-home-visit model while retaining the ability to explain visit outcomes. Do not invent split visits or independent box checkouts. |
| BD-08 | BUSINESS DECISION | Selection of a fulfilling hub and whether one booking may source stock from multiple hubs within the selected city. | Distinguish city-level eligibility from hub custody and joint fulfilment. Preserve one normal combined visit; do not introduce implicit intercity fulfilment. |
| BD-09 | BUSINESS DECISION | Operational identification and tracking of physical pieces; missing, damaged or substituted pieces; QC criteria and the disposition of failed-QC stock. | Preserve stock identity, custody and condition history to the detail later required. A returned or failed-QC piece must not automatically become available. |
| BD-10 | BUSINESS DECISION | Customer-facing rules for purchased-item returns, exchanges and refunds, including eligibility, timing and commercial conditions. | Distinguish after-sale activity from unpurchased trial returns, and distinguish a commercial return decision from movement of stock and repayment of money. |
| BD-11 | BUSINESS DECISION | Initial operating/selling entity, intended inventory ownership arrangements, and the business timing and scope of a future company transition. | Preserve the ability to identify the party responsible for a transaction or stockholding at the relevant time. Do not relabel historical transactions as belonging to a future company. |
| BD-12 | BUSINESS DECISION | Gold-specific offerings, purities, trial limits, eligibility, handling, fulfilment and launch conditions. | Keep material and purity separate and allow policies to vary where approved. Do not assume Gold operates identically to Silver or that the illustrative 18K example is the only Gold purity. |
| BD-13 | BUSINESS DECISION | Initial values and future scope of configurable limits and eligibility: boxes per booking, items per plan/box, category eligibility, serviceable PIN codes, cart retention and other availability rules within the approved initial city/material scope. | Keep policy settings separate from historical commitments. No illustrative item count, deposit, timeout or cart lifetime becomes a default. Reservation timing is covered by BD-02; initial Bengaluru/Silver availability remains approved. |
| CA-01 | CA REVIEW | Recognition and tax treatment of deposits; treatment when applied, refunded or retained; relation to jewellery consideration and reconciliation. | Preserve the original collection, later application or refund, relevant sale and timing without selecting an accounting or tax interpretation. |
| CA-02 | CA REVIEW | Applicable tax treatment and rates; production delivery/dispatch and sale documents; invoice numbering and timing; permitted correction records; financial treatment of price adjustments, returns, exchanges, refunds and settlements. | Preserve transaction and movement facts, historical prices/taxes, document identity and links to later adjustments. Historical invoices remain immutable; exact production requirements are not declared compliant. |
| CA-03 | CA REVIEW | Entity-specific accounting, invoicing and inventory treatment, including financial-record continuity and any transfer associated with a future entity transition. | Preserve original entity attribution and historical evidence; distinguish original transactions from any later transfer or adjustment. |
| LR-01 | LEGAL REVIEW | Customer and agent custody responsibilities; liability for missing or damaged pieces; any proposed charges; legal constraints on deposit, cancellation, return, exchange and refund policies. | Preserve relevant custody, condition and agreed-term evidence without presuming liability, enforceability or a right to retain money. |
| LR-02 | LEGAL REVIEW | Legal ownership of inventory, contracting and selling-party identity, and the legal arrangements needed for any future company/entity transition. | Preserve historical parties, ownership evidence and the distinction between business plans and legally effective changes. |
| LR-03 | LEGAL REVIEW | Applicable dispatch, transport and product-documentation obligations and the allocation of legal responsibility for those requirements. | Keep movement, sale and supporting evidence distinguishable. Coordinate with CA-02 without assuming that an example document or software record establishes compliance. |
| BD-14 | BUSINESS DECISION | How the canonical shared root docs and root AGENTS.md will be tracked, reviewed and distributed alongside two existing separate Git repositories. **RESOLVED 2026-09-23**; outcome and approval history are recorded above. | One canonical documentation-only root repository; application histories/remotes stay separate and excluded. No submodules or duplicated shared files. Documentation publication was verified on 2026-09-24. |
| BD-15 | BUSINESS DECISION | Remaining integration/provider choices, validation of the Razorpay product flow for shared QR/link access, channel retry/fallback and OTP resend settings, retention/monitoring operations and whether email is needed. | Keep providers behind boundaries, one shared final-payment context and one Arrival OTP challenge across channels. Do not confuse an operational setting with legal permission or a provider capability already verified. |
| LR-04 | LEGAL REVIEW | Final Terms, Privacy, Try-at-Home and Refund/Cancellation wording; contact/grievance obligations; personal-data retention/access/deletion; service versus marketing consent; applicable WhatsApp/SMS/DLT requirements. | Keep purpose, policy/version and consent evidence distinguishable; minimize customer data exposure. No template, website draft or provider integration is assumed to establish legal compliance. |

### Roadmap refinements that do not close open entries

- BD-01/BD-02: D-25 partially resolves the pre-payment booking OTP and upfront-collection-before-confirmation direction. It does not select amounts, classification, stock-hold timing or any refund/expiry outcome. Preserve the original questions and the partial-resolution note above.
- BD-03/BD-09: Product, ProductVariant and InventoryUnit separation is now explicit. Physical identification/tagging practices, counting, substitutions and exception handling still need decisions.
- BD-05: A shared QR and URL for one final PaymentAttempt is finalized. This does not prohibit future retries nor decide how to handle a stale outstanding request after selection changes.
- BD-08: Hub-specific inventory is explicit. It does not decide whether one combined visit may source multiple hubs within the chosen city.
- BD-13: localStorage is the requested persistence mechanism, but seven days and two boxes are suggested values only.
- CA-02: invoice immutability is settled; applicable tax, document details, numbering scope, timing and correction treatment still need review.
- BD-15/LR-04 are newly surfaced by the integration/legal-page roadmap; neither has an approved answer.

### Resolving a decision

Record the original ID, explicit outcome, reason, alternatives actually considered, consequences, approver, approval date, effective scope/timing and applicable CA/legal review evidence. Retain earlier outcomes and the facts governing historical commitments.

Before dependent implementation, reference the relevant IDs. Do not encode an unanswered question as a constraint, default, customer promise, tax treatment or test expectation. Continue work independent of unresolved answers.

## Reconciliation of source instructions

No genuine contradiction was found in the intended business model. The following instruction changes and apparent tensions need explicit treatment:

| Topic | Reconciliation |
| --- | --- |
| Previously duplicated shared docs vs latest structure rule | The latest explicit instruction supersedes duplication. One root docs/ set is canonical; child AGENTS.md files point to it. |
| Earlier "roadmap not supplied" instruction | Superseded: the roadmap is supplied, Task 1 is verified complete and Task 2 is explicitly authorized. Later task scope and unresolved decisions still require their own authority. |
| Optional deposit vs later upfront-payment instruction | Earlier roadmap wording specified capability only. D-25 now requires upfront collection before confirmation for the launch flow; its classification, amount and treatment remain BD-01/CA-01/LR-01 gates. |
| Suggested cart/box values | Seven days and two boxes are suggestions, not finalized configuration. |
| City inventory vs hub-specific inventory | Compatible refinement: each physical unit belongs to a hub in a city. Same-city sourcing policy remains open. |
| One normal visit vs assignment history or retries | Reassignment/history can be represented without approving split customer visits or separate box checkouts. BD-07 remains open. |
| Trial selection in Task 7 vs real gateway in Task 8 | Core purchase/selection logic can use development adapters; this does not authorize live unverified collection or handover. |
| Example booking states mix payment/visit/return facts | Lifecycle refinements are proposals; core finalized arrival and QC rules remain intact. |
| Gold weight support vs undecided price formulas | Structural support is required; actual valuation, price commitment and handling policies are not chosen. |
| Security/audits appear again in Task 10 | Foundational controls start earlier; Task 10 completes, exercises and hardens them. |
| Standing SEO vs existing ten-task plan | D-20 adds feature-owned acceptance criteria and a single ownership map within Tasks 4–10. Tasks 1–3 stay complete for their delivered scope; Task 4 proceeds after the lightweight review. No Task 11 or duplicate SEO roadmap is introduced. |
| Model-first Task 4 vs original storefront Task 4 | D-26 supersedes sequencing, not requirements: new 4.M1–4.M12 cover models; original 4.1–4.7 move intact to execution in Task 5. Existing work and D-21 design gates are preserved. |
| Shared Django auth / guest-claim proposals vs selected identity design | D-23 keeps StaffUser as AUTH_USER_MODEL and adds separate CustomerAccount authentication. D-24 prohibits all later guest-account linking, including claims and contact matching. |

## Canonical documentation and preservation record

The shared location is ARKA TARA/docs/. Root AGENTS.md is the concise operating guide; arkatara_backend/AGENTS.md and arkatara_frontend/AGENTS.md contain local guidance and relative pointers only.

The previously identical business-context.md and decision-register.md files under each repository were removed after preservation verification on 2026-09-19. Their useful content is retained in this canonical set:

| Previous useful content | Canonical destination |
| --- | --- |
| Business model, audience, launch scope, expansion, USP and unit economics | BUSINESS_CONTEXT.md |
| All 27 original invariants and the normal booking/visit clarification | BUSINESS_RULES.md; the financial example is retained in COMPLIANCE_AND_FINANCE.md |
| Domain responsibilities and architectural principles | ARCHITECTURE.md plus operational constraints in root AGENTS.md |
| All original BD-01..13, CA-01..03 and LR-01..03 unresolved entries | This file, preserved with original IDs and question/flexibility text |
| Decision authority and historical change handling | This file and root AGENTS.md |
| Financial traceability, immutability, deposit example and compliance limitation | COMPLIANCE_AND_FINANCE.md |
| New ten-task roadmap | TASKS.md, with all original subtask numbers |

Only superseded duplication instructions and the outdated statement that the roadmap had not yet been supplied are replaced. They are recorded above as superseded rather than carried forward as active instructions.

Shared root documents remain outside both child Git histories and are now tracked by their own root repository under approved BD-14. A standalone application clone or its CI checkout still does not include this context automatically; obtain the root documentation repository using its README layout. Its GitHub publication was verified on 2026-09-24.
