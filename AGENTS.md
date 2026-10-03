# ARKA TARA agent operating guide

## Scope and authority

ARKA TARA is a Bengaluru-first jewellery Try-at-Home business: "Order many. Try them at home. Buy any."

Tasks 1–3 are complete for their authorized scope; recorded CI/review and live gates remain explicit. Task 4 is in progress and was re-scoped on 2026-10-02 to cross-app model completion under D-26: models, integrity guards, migrations and model/database tests, not workflows or UI. Original storefront 4.1–4.7 now execute in Task 5, with IDs, source and evidence preserved. Keep ten tasks and all original IDs. D-20 search requirements and unresolved business/CA/legal gates remain binding. See docs/TASKS.md for the completion matrix and actual evidence; documentation updates do not implement the models.

Read [BUSINESS_CONTEXT](docs/BUSINESS_CONTEXT.md), [BUSINESS_RULES](docs/BUSINESS_RULES.md), [ARCHITECTURE](docs/ARCHITECTURE.md), [DATA_MODEL](docs/DATA_MODEL.md) and [DECISIONS](docs/DECISIONS.md) before proposing or implementing a task. Then consult [STATE_MACHINES](docs/STATE_MACHINES.md), [COMPLIANCE_AND_FINANCE](docs/COMPLIANCE_AND_FINANCE.md), [INTEGRATIONS](docs/INTEGRATIONS.md) and [TASKS](docs/TASKS.md) as relevant.

The single canonical shared documentation set is root docs/. Child repositories must reference it rather than duplicate full files. Do not move/delete documentation without explaining the reorganization and verifying useful content is preserved. Under BD-14, approved on 2026-09-23, a separate root Git repository tracks the shared docs, root guide and repository setup files. Both application repositories and local tools/data are excluded by the root tracking list. The documentation remote is published; see [README.md](README.md) for workspace setup. Do not add application code or submodules to the root repository.

## Working technical direction

Approved stack: Next.js/React, Django/DRF, PostgreSQL and a modular monolith. Public/customer API uses `/api/v1/`; staff API target is `/staff/api/v1/`. Task 1 records runtime versions; D-22 records AWS hosting direction, not deployed infrastructure. D-23–D-25 settle identity and booking boundaries.

Use Django Admin for back-office operations and a separate mobile staff portal. Add no microservices, Kubernetes or event streaming without a demonstrated requirement.

## Frontend design gate

Follow [D-21](docs/DECISIONS.md#d-21--frontend-visual-design-awaits-owner-input), the owner's 2026-10-02 scope clarification. Final UI design, visual direction, branding system and screen-level references have not been supplied. Do not independently design or finalize polished landing, category, product, cart, checkout, staff or related screens from roadmap descriptions. Existing Task 4 presentation is unapproved provisional work, not an accepted design baseline.

Until the owner supplies the applicable approved design, limit frontend work to necessary routes, contracts, server-rendering structure, metadata/SEO, API boundaries, non-indexing controls, accessibility-friendly semantic HTML, tests and minimal unstyled or clearly temporary functional placeholders where unavoidable. Current Task 4 has no new frontend implementation; D-26 transfers original storefront IDs to Task 5. Record technical progress separately and keep presentation acceptance pending. See the [canonical task gate](docs/TASKS.md#frontend-design-input-gate).

## Critical rules

- StaffUser remains AUTH_USER_MODEL; CustomerAccount is separate. Explicit principal/action/scope checks protect staff APIs. Customer signup/login/account checkout are prepared but disabled initially; guest checkout remains permanent. See D-23 and R-33–R-38.
- Guest bookings remain permanently customer-unlinked, including matching-phone accounts and signed-in guest checkout. No claim/import or indirect consent/audit linkage. Preserve immutable ownership and historical contact/address facts; never cascade account closure into transaction history.
- Booking-specific phone OTP precedes upfront payment; confirmation also requires verified collection and valid acceptance. Arrival OTP is a separate purpose. Amount, deposit/fee classification, treatment and stock-hold/late-payment rules remain partially unresolved under BD-01/BD-02 and CA/LR review.
- One booking normally means one customer home visit, with multiple category-specific boxes fulfilled together from the selected city's inventory. No independent checkouts merely because boxes differ.
- Each box has one category. Guest localStorage carts do not reserve stock and are never authoritative.
- Hub inventory is city-specific; no implicit intercity fulfilment. Material/purity and Product/Variant/physical Unit remain distinct.
- Backend controls price, deposit, tax, eligibility, availability and payment state. Agents cannot override authoritative prices.
- One Arrival OTP starts the trial, using the same challenge across WhatsApp/SMS. No second purchase OTP.
- Final-payment QR/link share one PaymentAttempt. Verified backend evidence controls payment; no screenshot/client-only confirmation.
- Trial, purchase, payment and invoice remain separate. Deposits do not reduce historical jewellery selling value.
- Returned trial stock requires physical hub receipt and QC before availability. Issued invoices, historical snapshots and movement/audit evidence must remain reliable.
- Bengaluru/Silver active; Hyderabad/Gold coming soon. Activation alone must not require schema redesign.

## Unresolved decisions and configuration

Respect BUSINESS DECISION, CA REVIEW and LEGAL REVIEW entries in DECISIONS.md. Do not silently resolve them through a default, schema constraint, copy or test. Mark dependencies and continue independent work.

Never hard-code market/material identity, PIN codes, GST rates, deposits, box/piece limits, category eligibility, reservation timeout or cart expiry when they belong in configuration. Seven days, two boxes and example deposits are unapproved suggestions.

## Standing search discoverability

Apply [D-20](docs/DECISIONS.md#d-20--standing-search-discoverability-and-task-4-continuation) to every change in Tasks 4–10. Assess crawling/indexing, stable URLs, metadata, semantic HTML, links, truthful structured data, mobile performance, duplicate content and future category/material/city expansion. Use the [task ownership map](docs/TASKS.md#standing-search-discoverability-requirement); build each feature at its natural stage, without duplicate work or Task 11.

Arka Tara's intended public domain is `arkatara.in`; this does not resolve legal identity or configure deployment. Keep critical public catalogue content server-rendered, private routes protected/non-indexable, and metadata consistent with verified visible facts. No invented prices, stock, locations, reviews, unsupported claims, keyword stuffing, doorway pages or ranking promises. Minor SEO additions do not reopen Tasks 1–3; explain a genuinely major architecture change before making it.

The requested staff host is `staff.arkatara.com` (D-23), not an implicit change to the public domain. Confirm ownership, aliases and actual credential/CORS/CSRF topology before deployment. Do not silently substitute a different domain.

## Business reference comments

The owner approved selective code-to-business references on 2026-09-25. Apply this convention to new work and relevant changes in both applications:

- Add a short explanation at important business validations, authority boundaries, policy gates and history-preservation code. Use stable rule/decision IDs, for example `Refs: BUSINESS_RULES.md R-07, R-09; DECISIONS.md BD-13.` Document filenames in these comments resolve to this workspace's canonical `docs/`, not to a second copy inside an application.
- Prefer existing `R-*` IDs for approved invariants and `D-*`, `BD-*`, `CA-*` or `LR-*` IDs for the relevant decision or review. Verify their actual text and current status. If a narrative detail has no rule/decision ID, cite an existing `BUSINESS_CONTEXT.md#section-anchor`; do not invent or renumber IDs for a comment.
- Explain why the boundary exists without repeating the full business documentation. Cite only directly relevant requirements; distinguish a foundation or future dependency from a fully enforced workflow. A reference to an unresolved decision never means it is approved.
- Keep comments close to the class/function/guard that owns the constraint. Avoid repeated annotations on every line, import, straightforward accessor or test. Keep task-to-file/symbol mapping in [TASKS.md](docs/TASKS.md#implementation-reference-map), rather than scattering task-number headers through source code.
- When changing a rule, moving its implementation or changing its scope, update affected comments and the central map in the same change. Keep stable IDs and preserve superseded decision history. Do not rewrite applied migrations merely to add references; map them in TASKS.md instead.
- References belong in developer comments/documentation, not customer-facing copy, validation messages, API payloads or business defaults. Check reference accuracy and existing style checks during review; a separate test suite for comment text is unnecessary.

## Engineering standards once implementation is authorized

- Keep domain logic explicit, typed where practical, modular and readable. Use consistent formatting/linting and narrow, reviewed dependencies; do not embed business logic in provider adapters or presentation code.
- Use precise monetary representation, server validation, transaction-safe inventory changes and idempotent financial processing.
- Use main/development/feature branches and focused reviews. Keep docs and decisions current with approved changes.
- Test invariants, invalid inputs, state/permission boundaries, concurrent reservation, duplicate/out-of-order callbacks, historical immutability and city/material activation; choose meaningful checks rather than tests that merely mirror code.
- Review migrations, preserve historical financial/inventory records, check upgrades and fresh setup, and document deployment/rollback implications. Never rewrite applied history casually.
- Separate local/staging/production secrets and configuration. Never commit secrets, expose provider credentials in the browser or store sensitive card data.
- Enforce staff assignment/RBAC, authenticated callbacks, OTP hashing/expiry/attempt limits, privacy-aware logs, rate limits and appropriate web security.
- Build audit/security foundations early; Task 10 is hardening and verification. Keep fakes restricted to development/test.
- Before each task, check its definition of done and any conflicts with the shared rules, architecture, model and decision register. A roadmap item does not silently resolve an open business/legal/tax question.

Later explicit owner instructions take precedence; record approved changes and retain the historical decision trail.
