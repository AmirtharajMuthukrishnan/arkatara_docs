# ARKA TARA agent operating guide

## Scope and authority

ARKA TARA is a Bengaluru-first jewellery Try-at-Home business: "Order many. Try them at home. Buy any."

Task 1 implementation is authorized as of 2026-09-19. Implement only the approved foundation scope while keeping later domain behavior and unresolved decisions gated.

Read [BUSINESS_CONTEXT](docs/BUSINESS_CONTEXT.md), [BUSINESS_RULES](docs/BUSINESS_RULES.md), [ARCHITECTURE](docs/ARCHITECTURE.md), [DATA_MODEL](docs/DATA_MODEL.md) and [DECISIONS](docs/DECISIONS.md) before proposing or implementing a task. Then consult [STATE_MACHINES](docs/STATE_MACHINES.md), [COMPLIANCE_AND_FINANCE](docs/COMPLIANCE_AND_FINANCE.md), [INTEGRATIONS](docs/INTEGRATIONS.md) and [TASKS](docs/TASKS.md) as relevant.

The single canonical shared documentation set is root docs/. Child repositories must reference it rather than duplicate full files. Do not move/delete documentation without explaining the reorganization and verifying useful content is preserved. Under BD-14, approved on 2026-09-23, a separate root Git repository tracks the shared docs, root guide and repository setup files. Both application repositories and local tools/data are excluded by the root tracking list. The documentation remote is to be configured separately; see [README.md](README.md) for workspace setup. Do not add application code or submodules to the root repository.

## Working technical direction

Approved stack: Next.js/React frontend, Django/DRF backend, PostgreSQL, modular monolith, /api/v1/. Exact framework/runtime versions are selected and recorded during Task 1; hosting remains unselected.

Use Django Admin for back-office operations and a separate mobile staff portal. Add no microservices, Kubernetes or event streaming without a demonstrated requirement.

## Critical rules

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
