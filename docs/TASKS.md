# Implementation backlog

Status: TASKS 1–3 COMPLETE FOR THEIR AUTHORIZED SCOPE; TASK 4 IN PROGRESS\
Updated: 2026-10-03\
Implementation status: **TASKS 1–3 REMAIN COMPLETE FOR THE DELIVERED SCOPE. TASK 4 IS IN PROGRESS, RE-SCOPED TO CROSS-APP MODEL COMPLETION UNDER D-26. TASK 5 INHERITS EXISTING STOREFRONT WORK; ITS CART WORK AND TASKS 6–10 REMAIN PLANNED. RECORDED REVIEW/CI AND LIVE OPERATING GATES REMAIN EXPLICIT.**

The owner approved the business context and invariants, approved the Next.js/React + Django/DRF + PostgreSQL stack, and authorized Task 1 implementation on 2026-09-19. Task 2 was authorized on 2026-09-24 after successful Task 1 verification. Task 3 was authorized on 2026-09-25 and continued on 2026-09-26. On 2026-09-26 the owner made search discoverability a standing requirement, instructed that Tasks 1–3 stay completed unless a serious architecture issue is found, and authorized continuing with the next pending task after updating this plan if no blocking approval is needed. Task 4 originally proceeded under D-20; D-26 now governs its models-only scope. This does not approve unresolved business, CA or legal policies or activate a live pilot.

The owner's 2026-10-02 model-first instruction, [D-26](DECISIONS.md#d-26--task-4-cross-app-model-completion), supersedes the original Task 4 storefront scope. Task 4 now completes models, migrations and model/database verification for the agreed roadmap. Original 4.1–4.7 retain their IDs and move to execution under Task 5. [D-21](DECISIONS.md#d-21--frontend-visual-design-awaits-owner-input) still gates all presentation pending approved design references. The roadmap change itself implemented nothing; subsequent owner authorization to code settled models has produced the partial identity/booking foundation recorded below on 2026-10-03.

On 2026-09-21, the owner reconfirmed **finish Task 1 only** and initially kept BD-14 pending. On 2026-09-23, the owner approved the separate documentation-only root repository, resolving BD-14. The owner subsequently published all three repositories. The 2026-09-24 Task 2 instruction supersedes the earlier Task 1-only limit; it does not resolve any commercial or external-review policy.

This is the canonical shared backlog for both repositories. Consult [BUSINESS_CONTEXT.md](BUSINESS_CONTEXT.md), [BUSINESS_RULES.md](BUSINESS_RULES.md), [ARCHITECTURE.md](ARCHITECTURE.md), [DATA_MODEL.md](DATA_MODEL.md), [STATE_MACHINES.md](STATE_MACHINES.md), [COMPLIANCE_AND_FINANCE.md](COMPLIANCE_AND_FINANCE.md), [INTEGRATIONS.md](INTEGRATIONS.md), [DECISIONS.md](DECISIONS.md), and the applicable `AGENTS.md` before implementing any task. Do not maintain a duplicate backlog in either application repository.

## Backlog interpretation and gates

- The ten-task structure and all 95 original subtask IDs are preserved. D-26 adds 4.M1–4.M12 without renumbering original 4.1–4.7, whose execution moves to Task 5. Suggested implementation detail remains reviewable; an open business or external-review item is never resolved by approving a technical plan.
- One TrialBooking normally means one customer home visit. Multiple category-specific TrialBoxes travel together from the selected city's operational inventory, with a shared booking checkout. Do not infer independent box checkouts, one payment per box, or an approved multi-hub allocation policy.
- Numerical examples are **suggestions only**: seven-day cart retention and two boxes remain **BD-13**; INR 29/49 amounts remain **BD-01**, with **CA-01** for treatment. D-25 requires booking phone OTP before upfront payment and verified collection before confirmation for the launch flow. Amount/basis/classification and refund/application rules are still open; never introduce example defaults or silently use zero.
- All persistence required by Tasks 5–10 belongs to Task 4, including authentication support, booking verification/access, trial/delivery, financial history, notification work items and analytical evidence. Later tasks implement behaviour on these models. A schema-shaping unresolved decision blocks the affected model acceptance; missing approved operating values may remain explicitly unconfigured.
- All outstanding entries in [DECISIONS.md](DECISIONS.md) remain open until separately resolved. A policy dependency blocks the behaviour that would force that choice, not unrelated schema or interface work. Development fixtures must be explicitly labelled fictional scenarios, with no production seed silently promoting their values.
- Configuration can encode approved choices; merely adding a configurable field does not authorize choosing its value. Missing required production policy must cause a clear setup/readiness failure, not an invented fallback.
- Product, ProductVariant and uniquely tracked InventoryUnit are distinct. The operational tagging/scanning method, QC criteria and loss/damage handling remain **BUSINESS DECISION BD-09**.
- Tasks are a dependency-aware roadmap, not a requirement to postpone every later-named concern: secure configuration, permissions, integrity and audit foundations begin with Task 1; Task 10 deepens and verifies them. Add relevant event hooks when the underlying domain behaviour is implemented, then validate reporting in Task 10.
- Task 7 creates selection, Purchase and PurchaseItem domain behaviour before Task 8 adds live gateway collection, complete billing and reconciliation. Task 6 introduces provider contracts and development fakes; actual payment integration belongs to Task 8 and messaging integration to Task 9. Fake payment or messaging success is never evidence of a real payment or a production-ready customer flow.
- A live pilot before Task 8 has no approved collection/confirmation procedure (**BUSINESS DECISION BD-06**). Deferred integration does not imply cash, manually verified UPI, screenshots or permission to launch without trusted payment confirmation.
- Source-control tracking follows **BUSINESS DECISION BD-14**, approved on 2026-09-23: a documentation-only root repository excludes the two separate application repositories and local tools/data. Keep the canonical copy under root `docs/`; do not merge application histories, introduce submodules or duplicate documents. Documentation publication was verified on 2026-09-24.
- Provider choices, validation of the shared Razorpay QR/link flow and channel retry/fallback/OTP-resend/retention operations are **BUSINESS DECISION BD-15**. Customer-facing legal pages, privacy, consent, retention and applicable messaging/DLT obligations are **LEGAL REVIEW LR-04**. These gates apply wherever the corresponding work appears below, including Tasks 3, 4, 6, 7, 8, 9 and 10.

### Frontend design input gate

**Binding across all frontend work and acceptance criteria, including Tasks 4–10; BUSINESS DECISION D-21.** The owner has not yet supplied the final UI design, visual direction, branding system or screen-level designs. Roadmap descriptions such as "premium" and "mobile-first" describe intended outcomes and do not authorize agents to design them independently.

- Continue necessary technical foundations: routes, data contracts, server-rendering structure, metadata/SEO architecture, API integration boundaries, non-indexing/privacy controls, accessibility-friendly semantic HTML and tests. Backend and other design-independent work remain within the existing task authorization and policy gates.
- Where a screen is unavoidable for functional verification, use a minimal unstyled or clearly temporary placeholder. Do not create or finalize polished landing, category, product, cart, checkout, staff, payment, notification or related presentation before the owner supplies the applicable approved design reference.
- Record that reference and the screens it covers when supplied. Do not infer other designs from a single approved screen, brand name/domain, broad business aspirations, successful tests or earlier provisional UI.
- Preserve frontend requirements and IDs, with original 4.1–4.7 now executed in Task 5 under D-26. Track technical implementation separately from **AWAITING OWNER DESIGN INPUT**; design-dependent presentation, visual/mobile acceptance and final journey reviews stay pending. Task 4 model acceptance does not include final UI.
- Existing Task 4 styling, page composition and presentation copy are unapproved provisional work. Do not continue polishing them or adopt them as final branding. This clarification changes documentation/instructions only; existing source and prior technical evidence are retained.

### Reconciliation against the approved business context

The roadmap preserves hub inventory and catalogue-design/variant/physical-unit distinctions. D-23–D-25 settle separate identities, permanent guest ownership, booking OTP and upfront collection before confirmation. Expiry/late-payment outcomes, collection amount/treatment, price commitment, hub allocation, Gold rules and handover during uncertain payment remain open. Example state lists in [STATE_MACHINES.md](STATE_MACHINES.md) are not unconditional transitions or authorization to bypass those gates.

## Standing search-discoverability requirement

**Approved 2026-09-26; DECISIONS.md D-20.** The official brand is Arka Tara and the intended public domain is `arkatara.in`. For every change in Tasks 4–10, assess its effect on crawling, indexing, URLs, metadata, semantics, linking, structured data, mobile performance, duplicate content and future materials/categories/cities. Build the appropriate support with its owning feature; do not postpone everything until launch or implement speculative SEO systems. Search discoverability is not a ranking guarantee.

Branded intent includes Arka Tara, Arka Tara jewellery, arkatara, and the Bangalore/Bengaluru name variants. Discovery intent includes Silver/S925/925 jewellery, assisted home trial, rings, neck chains, bracelets, kadas, women's jewellery, daily/office wear and lightweight designs where the actual assortment supports those descriptions. These are audience needs, not strings to repeat mechanically. Write natural useful content; do not create doorway pages, keyword-stuffed copy or near-identical city/category combinations. Claims, stock, prices, locations, reviews and ratings must remain truthful and approved. SEO does not authorize publishing private draft media or bypassing commercial gates.

The following ownership map assigns implementation once. Each task's search acceptance criteria below are part of its existing Tests and Definition of Done; they add no new task or numbered roadmap IDs. Other tasks consume these capabilities rather than rebuilding them.

| Owning task | Search-related scope | Verification handoff |
| --- | --- | --- |
| 4 | Persistence for stable public identity, publication/media/metadata and redirect history where required; separation of private customer/financial data | Model/migration/history tests; no public page implementation in this scope |
| 5, including original 4.1–4.7 | Public catalogue rendering, stable URLs, metadata/Open Graph, semantic links, structured data, public media and mobile performance; deliberate query/pagination handling and non-indexable cart | API/rendered-page/cart/privacy checks; consume Task 4 schema and retained URL helpers; deployed verification in Task 10 |
| 6 | Non-indexable private checkout/booking surfaces and reviewed public policy-page treatment | Authorization, response/cache and metadata leakage tests |
| 7 | Non-indexable authenticated staff/delivery surfaces | Staff access and private-data response tests |
| 8 | Non-indexable payment/receipt/invoice/download surfaces | Financial access, response headers and token-leakage tests |
| 9 | Privacy of transactional links, previews and message redirects | Message/template/link tests; reuse destination access controls |
| 10 | Launch robots/sitemaps, HTTPS/preferred-domain consistency, redirects/404 checks, search-console readiness, submission/indexation/queries, Core Web Vitals and expansion verification | Recorded deployment/account evidence and ongoing monitoring |

Public path examples such as `/silver-jewellery/` or `/silver-rings/` are illustrative, not mandatory routing or taxonomy choices. Use stable identity and extensible routing rather than hard-coding Silver/Bengaluru. Create a city or search-intent landing page only when distinct, substantial customer value and verified operational facts justify it. Keep important public catalogue content readable without requiring client-side JavaScript; private records require access control, not merely a `noindex` directive or robots rule.

**Lightweight completed-task review, 2026-09-26:** No expensive SEO architecture blocker was found in Tasks 1–3. The existing Next.js App Router permits server rendering and page metadata; catalogue/reference public UUIDs and stable slug-compatible codes separate identity from names; markets/materials are data-driven; media has descriptive/alt-text capability and provider-neutral identity. These were originally Task 4 storefront responsibilities; D-26 now assigns their persistence to Task 4 and public implementation to Task 5. Private product/media/pricing drafts must remain private. No completed task is reopened and no major architecture change is required for this work. Deployment/domain/search-account readiness is later Task 10 scope; unresolved commercial content/pricing/availability stays gated rather than fabricated.

## Implementation reference map

**Scope change, 2026-10-02:** Existing 4.1–4.7 evidence below is historical storefront work now carried into Task 5, not proof that new Task 4 model scope is complete. Transaction persistence formerly deferred to Tasks 6–8 moves to 4.M6–4.M9; workflow/payment confirmation stays in those later tasks. The app coverage matrix in [DATA_MODEL.md](DATA_MODEL.md#task-4-model-coverage) and 4.M1–4.M12 are the pending model checklist. Add actual file/class/migration/test references as each part is implemented; do not label proposed files as delivered.

The owner approved selective traceability on 2026-09-25 (DECISIONS.md D-18). Source comments use `Refs:` with the stable IDs in [BUSINESS_RULES.md](BUSINESS_RULES.md) and [DECISIONS.md](DECISIONS.md); filenames resolve to this canonical `docs/` directory. The [shared convention](../AGENTS.md#business-reference-comments) applies to future work in both applications. This map lists important implementation boundaries and existing verification, not every line or utility. Update affected rows and comments when code or policy changes.

| Task item | Implementation entry points | Business reference and boundary | Existing verification |
| --- | --- | --- | --- |
| 4.M1 / 4.M2 / 4.M6 / 4.M7 / 4.M12, partial | [accounts/models.py](../arkatara_backend/accounts/models.py) (`CustomerAccount`); [compliance/policies.py](../arkatara_backend/compliance/policies.py) (`PolicyDocumentRevision`); [trials/models.py](../arkatara_backend/trials/models.py) (`TrialBooking`, `TrialBox`, `BookingPhoneChallenge`, `GuestBookingAccessGrant`, `BookingPolicyAcceptance`); [delivery/models.py](../arkatara_backend/delivery/models.py) (`DeliveryAssignment`, `ArrivalOTP`); [common/evidence.py](../arkatara_backend/common/evidence.py), [common/verification.py](../arkatara_backend/common/verification.py) | R-02, R-18, R-20–R-22, R-28, R-30, R-33–R-38; D-23–D-26. Separate identity, irreversible ownership, original snapshots, purpose-bound evidence and assignment history only. No authentication, accepted booking, operational authorization or payment workflow. | `test_booking_identity_models.py`, `test_booking_identity_migrations.py`; accounts migrations 0002–0003, compliance 0003–0004, trials 0001–0002, delivery 0001–0003. [Verification/recovery record](reviews/TASK4_MODEL_FOUNDATION_2026-10-03.md). No whole-app acceptance implied. |
| 1.1–1.4, 1.7 | Backend [settings](../arkatara_backend/config/settings/base.py), [environment parsing](../arkatara_backend/config/env.py), [API routes](../arkatara_backend/config/api_urls.py); frontend [API transport](../arkatara_frontend/src/lib/api.ts) (`getApiBaseUrl`, `requestJson`); each application's CI workflow and lockfile | D-02, D-14, D-16; R-07. Technical foundation and transport validation do not establish a booking or payment result. Routine setup needs no inline rule labels. | Backend `test_environment.py`, `test_health.py`, `test_api_permissions.py`; frontend `api.test.ts`, `error.test.tsx`; existing CI checks |
| 1.5 | [compliance/models.py](../arkatara_backend/compliance/models.py) (`LegalEntity`); [compliance/admin.py](../arkatara_backend/compliance/admin.py) (`LegalEntityAdmin`) | R-27, R-30; BD-11, CA-03, LR-02. Entity identity and audited maintenance are foundations; future transaction records must retain issuer snapshots. | `test_foundation_admin.py`, `test_foundation_models.py` |
| 1.6 | [compliance/configuration.py](../arkatara_backend/compliance/configuration.py) (`CONFIGURATION_DEFINITIONS`, `get_approved_configuration`); [compliance/models.py](../arkatara_backend/compliance/models.py) (`BusinessConfiguration`); [compliance/admin.py](../arkatara_backend/compliance/admin.py) (`BusinessConfigurationAdmin`) | R-07, R-28; BD-02, BD-13. Typed values, exact scope and preserved revisions; missing/conflicting approvals fail closed and Admin only creates drafts. | `test_foundation_models.py`, `test_foundation_admin.py`, `test_foundation_migrations.py` |
| 1.8 | [accounts/models.py](../arkatara_backend/accounts/models.py) (`StaffUser`); [compliance/models.py](../arkatara_backend/compliance/models.py) (`AuditEvent`); [compliance/audit.py](../arkatara_backend/compliance/audit.py) (`record_model_change`); [compliance/admin.py](../arkatara_backend/compliance/admin.py); [audit guard migration](../arkatara_backend/compliance/migrations/0002_foundation_integrity.py) | R-04, R-18, R-30. Staff attribution is distinct from customer login; atomic audit writes and protected evidence cover current privileged operations. Later domain actions still need their own audits. | `test_foundation_models.py`, `test_foundation_admin.py`, `test_foundation_migrations.py` |
| 2.1–2.2 | [markets/models.py](../arkatara_backend/markets/models.py) (`Market`, `Hub`); [approved market seed](../arkatara_backend/markets/migrations/0002_initial_markets.py) | R-08, R-10, R-13; D-04, D-07; BD-08, BD-11, LR-02. Hub means operational city/location, not legal ownership or an approved allocation policy. Initial states: [business context](BUSINESS_CONTEXT.md#initial-scope-and-future-direction). | `test_markets.py`, `test_reference_models.py`, `test_domain_migrations.py` |
| 2.3 | [markets/models.py](../arkatara_backend/markets/models.py) (`ServiceArea`); [markets/services.py](../arkatara_backend/markets/services.py) (`coverage_state`); [markets/api.py](../arkatara_backend/markets/api.py) (`ServiceabilityView`) | R-07–R-10; BD-07, BD-08, BD-13; D-17. Exact selected-market coverage; no fallback, conflict precedence or automatic booking authorization. | `test_markets.py`, `test_reference_api.py` |
| 2.4–2.5 | [catalog/models.py](../arkatara_backend/catalog/models.py) (`Material`, `Purity`); [approved material seed](../arkatara_backend/catalog/migrations/0002_initial_materials.py) | R-10, R-11, R-13; D-07, BD-12. Separate material/purity; only the approved initial identities are seeded. Initial states/S925: [business context](BUSINESS_CONTEXT.md#initial-scope-and-future-direction). | `test_reference_models.py`, `test_domain_migrations.py` |
| 2.6 | [catalog/models.py](../arkatara_backend/catalog/models.py) (`Category`); [markets/services.py](../arkatara_backend/markets/services.py) (`category_eligibility_revision`) | R-07, R-09, R-10; BD-13; D-17. Explicit eligibility revision for market/material/category; activation is not permission to book and no default eligibility is supplied. | `test_reference_api.py` |
| 2.7 | [catalog/models.py](../arkatara_backend/catalog/models.py) (`Product`, `ProductVariant`); [inventory/models.py](../arkatara_backend/inventory/models.py) (`InventoryUnit`); [common/reference.py](../arkatara_backend/common/reference.py) | R-08, R-11, R-15; D-17; BD-09, BD-11. Distinct identities, opaque SKU, validated definition changes and DRAFT physical units. Task 3's movement/custody/QC extension is mapped below; reservation remains later work. | `test_reference_models.py`, including concurrent definition protection |
| 2.8 | [catalog/pricing.py](../arkatara_backend/catalog/pricing.py) (`PriceRevision`, `exact_decimal`); [price history guard](../arkatara_backend/catalog/migrations/0003_price_history_guard.py) | R-12, R-28; BD-04, BD-12, CA-02; D-17. Immutable draft inputs and exact storage capacity, without a live formula, rounding policy or commitment time. | `test_pricing_foundation.py`, `test_domain_migrations.py` |
| 2.9 | [billing/snapshots.py](../arkatara_backend/billing/snapshots.py) (`HistoricalPriceSnapshot`, `_tax_evidence`) | R-27, R-28, R-31; BD-04, CA-02; D-17. Immutable copied evidence contract; transaction persistence/protection belongs to Task 4 under D-26; financial workflow confirmation remains Tasks 6–8. | `test_pricing_foundation.py` |
| Task 2 frontend/reference API | [markets/api.py](../arkatara_backend/markets/api.py) (`ReferenceDataView`); [catalog-contracts.ts](../arkatara_frontend/src/lib/catalog-contracts.ts) (`parseReferenceData`, `Availability`, `ServiceabilityResult`) | R-07, R-09, R-10, R-13; BD-08, BD-13; D-17. Data-driven display, validated response shapes and explicit backend bookability; coverage alone cannot enable a visit. | Backend `test_reference_api.py`; frontend `catalog-contracts.test.ts` |
| Task 2 frontend pricing/variants | [catalog-contracts.ts](../arkatara_frontend/src/lib/catalog-contracts.ts) (`ProductVariantSummary`, `DecimalString`, `PriceDisplay`) | R-11, R-12; BD-04; D-17. Distinct reference identities and exact decimal strings; draft representations are not approved customer prices. | `catalog-contracts.test.ts` |
| 3.1, 3.3, 3.7 | [common/admin.py](../arkatara_backend/common/admin.py) (`AuditedReferenceAdmin`, `ReferenceAdminForm`); [catalog/admin.py](../arkatara_backend/catalog/admin.py); [markets/admin.py](../arkatara_backend/markets/admin.py); [inventory/admin.py](../arkatara_backend/inventory/admin.py) | R-11, R-15, R-18, R-28, R-30; D-19. Draft maintenance, stale-edit/identity protection, restricted operation screens and filters; no approval or price override. | `test_catalog_onboarding.py`, `test_inventory_operations.py` |
| 3.2 | [catalog/media.py](../arkatara_backend/catalog/media.py) (`ProductMedia`); [media_services.py](../arkatara_backend/catalog/media_services.py) (`upload_product_media`, `update_product_media_metadata`, `media_download_response`); [storage.py](../arkatara_backend/catalog/storage.py); [onboarding_admin.py](../arkatara_backend/catalog/onboarding_admin.py); frontend [media-contracts.ts](../arkatara_frontend/src/lib/media-contracts.ts) | R-11, R-18, R-30; D-14, D-19, BD-15. Private asset identity, audited changes, configured type/size checks, provider-neutral storage; no public URL or production adapter. | Backend `test_product_media.py`; frontend `media-contracts.test.ts` |
| 3.4–3.5 | [inventory/services.py](../arkatara_backend/inventory/services.py) (`register_unit`, `receive_unit`, `record_qc`); [inventory/models.py](../arkatara_backend/inventory/models.py) (`InventoryMovement`, `InventoryUnit`); [configuration.py](../arkatara_backend/compliance/configuration.py); [movement guard](../arkatara_backend/inventory/migrations/0003_movement_history_guard.py); [downgrade guard](../arkatara_backend/inventory/migrations/0004_operational_downgrade_guard.py) | R-08, R-11, R-15, R-17, R-28, R-30; D-19, BD-09, BD-11. Gated receipt/QC, atomic evidence and idempotent retries. The customer-return/reservation/sale lifecycle remains unimplemented. | `test_inventory_operations.py`, `test_backoffice_migrations.py` |
| 3.6 | [catalog/imports.py](../arkatara_backend/catalog/imports.py) (`preview_import`, `apply_import`); [import_batches.py](../arkatara_backend/catalog/import_batches.py) (`CSVImportBatch`); [import guard](../arkatara_backend/catalog/migrations/0005_import_history_guard.py) | R-11, R-15, R-28, R-30; D-19, BD-09, BD-13. Atomic draft onboarding, UUID/SKU identity checks, retained request outcomes; no physical receiving or policy activation. | `test_catalog_onboarding.py` |
| 4.1, 4.5–4.6: publication | [storefront_models.py](../arkatara_backend/catalog/storefront_models.py) (`StorefrontPage`, `lock_page_tree`); [storefront.py](../arkatara_backend/catalog/storefront.py) (`change_publication`, `visible_pages`, `breadcrumbs`); [storefront_admin.py](../arkatara_backend/catalog/storefront_admin.py); [publication migration](../arkatara_backend/catalog/migrations/0006_storefrontpage.py) | R-07, R-09, R-11, R-30; D-20. Stable public page identity, reviewed content and attributed publish/withdraw actions; activation, assets, prices and booking remain separate. | Coverage authored in `test_storefront.py`; dated run evidence belongs to the retained storefront progress, now under Task 5. |
| 4.2–4.4, 4.7: public read boundary | [storefront.py](../arkatara_backend/catalog/storefront.py) (`public_variants`, `availability`, `product_detail`); [storefront_api.py](../arkatara_backend/catalog/storefront_api.py) (`CatalogueView`, `StorefrontPageView`); frontend [storefront.ts](../arkatara_frontend/src/lib/storefront.ts) and [storefront-server.ts](../arkatara_frontend/src/lib/storefront-server.ts) | R-07–R-11, R-13; D-20, BD-08, BD-13. Published facts and selected-market physical availability only; all results remain non-bookable, with private prices/media/specifications withheld. | Backend `test_storefront.py`; frontend `storefront.test.ts`; verification status follows the retained original Task 4 checks, not the new model checklist. |
| 4.1–4.6: server-rendered experience | [home page](../arkatara_frontend/src/app/page.tsx), [collections](../arkatara_frontend/src/app/collections/page.tsx), [collection detail](../arkatara_frontend/src/app/collections/[slug]/page.tsx), [product detail](../arkatara_frontend/src/app/products/[slug]/page.tsx), [how it works](../arkatara_frontend/src/app/how-it-works/page.tsx); [shared components](../arkatara_frontend/src/components/storefront.tsx) | R-04, R-07, R-09, R-13; D-20. Public semantic content, configured selectors, honest empty/error states and coming-soon presentation. Cart and booking actions remain Tasks 5–6. | `src/app/storefront-pages.test.tsx`, `src/components/storefront.test.tsx`; rendered-page/mobile verification follows the retained storefront record, now under Task 5. |
| 4.1, 4.5–4.6: metadata and canonical identity | [seo.ts](../arkatara_frontend/src/lib/seo.ts) (`pageMetadata`, `jsonLd`, `productSchema`, `breadcrumbSchema`); [root layout](../arkatara_frontend/src/app/layout.tsx) | D-20; R-07, R-13, R-27; BD-04, CA-02. Stable absolute canonicals, deliberate pagination/query policy and factual structured data. No invented offers, reviews, locations or legal identity; indexing is disabled by default. | `seo.test.ts`, `src/app/storefront-pages.test.tsx`; Task 10 owns deployed crawl/indexation checks. |
| 4.6: public image seam | [storefront-config.ts](../arkatara_frontend/src/lib/storefront-config.ts) (`catalogueMediaOrigins`); [storefront.ts](../arkatara_frontend/src/lib/storefront.ts) (`PublishedImage`); [next.config.ts](../arkatara_frontend/next.config.ts); [shared components](../arkatara_frontend/src/components/storefront.tsx) | D-19, D-20, BD-15. Explicit HTTPS delivery origins and validated alt/dimension data for responsive images; backend currently returns no public assets. This is not production media publication. | `storefront.test.ts`, `src/components/storefront.test.tsx`; actual public delivery and production media performance remain gated. |

Backend test filenames above resolve under [arkatara_backend/tests/](../arkatara_backend/tests/); frontend tests are beside their implementation in [src/lib/](../arkatara_frontend/src/lib/), [src/app/](../arkatara_frontend/src/app/) or [src/components/](../arkatara_frontend/src/components/). Test evidence and delivery status are recorded in each task's dated progress below. Applied migrations are linked here for traceability and are not rewritten to add comments.

**Traceability update 2026-09-25:** The implementation baseline is now committed locally as docs `3cb2bb2`, backend `e319e5f` and frontend `7bfbc3e`, superseding the earlier uncommitted note. This pass adds developer references and the ongoing convention only; it does not assert new remote CI/merge evidence or start Task 3.

**Traceability verification:** All 36 source reference annotations resolve to existing canonical IDs. The 91 local documentation links/anchors checked and mapped test paths resolve. Comparison with the committed baseline confirms unchanged executable Python and TypeScript/type syntax; only comments/docstrings and documentation changed. Backend/frontend lint and formatting and all repository diff checks pass. Applied migrations are unchanged; no additional application tests or reference-checking CI machinery were introduced for this documentation pass.

## 1. Project foundation, architecture and business configuration

**Status:** COMPLETE — VERIFIED 2026-09-24.

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
| 1.7: review and CI | Initial and follow-up PRs are merged. CI passed on the final backend/frontend main commits; evidence below. |
| 1.7: canonical documentation tracking | **APPROVED — BD-14 resolved 2026-09-23**. The owner published arkatara_docs; the root tracking boundary excludes both application repositories. |

**Documentation tracking update 2026-09-23:** The owner approved creating the root documentation repository after confirming that application code would remain in its existing repositories. The earlier BD-14 deferral is retained in the dated progress notes above as history and is superseded by this approval. Root Git contains only the nine canonical documents, root guide and repository setup files; GitHub remote configuration/publication is separate.

**Final remote verification 2026-09-24:** [Backend PR #2](https://github.com/AmirtharajMuthukrishnan/arkatara_backend/pull/2) is merged into `main` at `a8f5354`; its [main CI](https://github.com/AmirtharajMuthukrishnan/arkatara_backend/actions/runs/35901582302) succeeded. [Frontend PR #2](https://github.com/AmirtharajMuthukrishnan/arkatara_frontend/pull/2) is merged into `main` at `b1fc581`; its [main CI](https://github.com/AmirtharajMuthukrishnan/arkatara_frontend/actions/runs/35901741964) succeeded. [Documentation](https://github.com/AmirtharajMuthukrishnan/arkatara_docs) is published at `2df731b`. These verified results supersede earlier dated pending-commit/CI/BD-14 notes. No Task 1 completion gate remains.

**Branch note:** Application `development` branches are behind those verified `main` commits. Task 2 branches start from `origin/main` to include all fixes; synchronize the integration branches through review before merging Task 2. No remote branch was rewritten. All three worktrees use `feature/task-2-domain-model`.

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

**Business/account prerequisites:** **BUSINESS DECISION BD-11** for actual operating entity and ownership arrangements; **CA REVIEW CA-03** and **LEGAL REVIEW LR-02** for entity transition requirements; **BUSINESS DECISION BD-13/BD-02** for policy values. **BD-14** for canonical documentation source control was resolved on 2026-09-23; remote publication was verified on 2026-09-24. Domain design can proceed within its authorized scope without fabricating real registrations, addresses or entity identities.

**Tests:** Verify environment isolation, configuration validation, API version routing, denied unauthorized access, audit attribution and reproducible setup. CI must detect migration drift and exercise migrations against PostgreSQL where relevant. Demonstrate that changing current entity/configuration references cannot rewrite historical snapshots once those records exist.

**Definition of Done:** Approved technical baseline is reproducible; environments are separated; CI and migration checks run; module responsibilities and source-control workflow are documented; unresolved business settings remain explicit; the canonical documentation is accessible to both repositories under an approved tracking approach. Application and infrastructure work is reviewed independently of this documentation delivery.

**Non-goals:** Payment integration, messaging integration, production account activation, microservices and choosing unanswered commercial policies.

## 2. Future-proof market, material, catalogue and pricing model

**Status:** COMPLETE FOR AUTHORIZED DOMAIN-FOUNDATION SCOPE — LOCALLY VERIFIED. Authorized 2026-09-24; verified 2026-09-25; retained as complete under the owner's 2026-09-26 instruction. Recorded review/CI/merge steps remain distinct delivery evidence. Its original transaction-binding deferral is retained below with the D-26 handoff: persistence now Task 4, workflow binding Tasks 6–8.

| Task 2 area | Implementation and remaining boundary |
| --- | --- |
| 2.1–2.5 | Market, Hub, ServiceArea, Material and Purity models/migrations. Approved BLR/HYD and SILVER/GOLD launch labels plus S925 are seeded. No real hubs, service PINs or Gold purities are supplied. |
| 2.6 | Category activation and exact market/material/category eligibility lookup return explicit approved revision evidence; missing/conflicting policy fails closed. No eligible category or plan defaults. |
| 2.7 | Product, ProductVariant and InventoryUnit are distinct. Protected relationships, material/purity validation, stable identities and locked definition changes preserve physical/pricing evidence. Units remain DRAFT until Task 3 receiving/movement workflows. |
| 2.8 | Append-only DRAFT PriceRevision stores explicit FIXED/WEIGHT_BASED inputs, currency and evidence; exact decimal validation and PostgreSQL UPDATE/DELETE protection. No current-price selector, calculation, tax formula or live activation. |
| 2.9 | Immutable HistoricalPriceSnapshot value contract and stability tests implemented. **Original handoff to Tasks 6–8 superseded by D-26:** Task 4 owns booking/purchase/invoice persistence and protection; Tasks 6–8 bind facts at the approved BD-04/CA-02 commitment point. It is not claimed complete for nonexistent transaction tables. |
| Frontend/API | Read-only reference-data and serviceability endpoints, runtime-validated TypeScript contracts for references, variants, availability and exact draft price inputs. All coverage results return bookable=false. No storefront changes. |
| Operations | Configuration onboarding and approval dependencies documented in DATA_MODEL.md. Task 3 must add attributable Admin workflows before real configuration/stock onboarding. |

**Validation 2026-09-25:** PostgreSQL upgrade from Task 1, fresh test schema, seed reapplication, reversible price-history trigger, city/material isolation, coverage/eligibility gates and concurrent identity protection pass in the full **176-test backend suite**. Django system checks, migration drift/plan checks, backend lint and formatting pass. Frontend **62 tests**, lint, formatting, TypeScript and production build pass. Task 2 changes are local and uncommitted on `feature/task-2-domain-model`; Task 1's green remote CI does not cover them. Commit/review/CI/merge remain delivery steps. No Task 3 work or live commercial activation has started.

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

**Status:** COMPLETE FOR AUTHORIZED BACK-OFFICE SCOPE — LOCALLY VERIFIED; retained as complete under the owner's 2026-09-26 instruction. Live policy activation, production storage, review/CI/merge remain pending as recorded below.

**Implementation recorded 2026-09-26:** Work is on `feature/task-3-backoffice` in the three separate repositories, carrying the approved selective-reference work. No Task 3 commit/push or new remote CI result is claimed.

| Item | Delivered boundary |
| --- | --- |
| 3.1 | Audited draft/reference Admin for markets, hubs, service areas, materials, purities, categories, products and variants; inventory uses dedicated services. Status/approval bypasses and stale reference edits are blocked. Price revisions remain read-only. |
| 3.2 | ProductMedia main/additional/video/thumbnail metadata, private upload/download and audited metadata/retirement; provider-neutral storage with an explicit local/test adapter. Production vendor/types/size settings are unset. Frontend validates the private metadata contract only. |
| 3.3 | Existing opaque SKU uniqueness retained; each physical piece has a separate UUID. No barcode/RFID scheme inferred. |
| 3.4 | Gated registration → receipt → inspection path. Receipt produces QC_PENDING/HUB custody; passing QC produces AVAILABLE, failure QUARANTINED. Effective approved hub procedures and action permissions are required. No operational approvals are seeded. |
| 3.5 | Immutable InventoryMovement with state/custody/location observations, actor, event/record times, request key/fingerprint and procedure evidence. State, movement and audit are atomic; SQL UPDATE/DELETE triggers protect history. Legacy drafts receive no invented history. |
| 3.6 | UTF-8 CSV preview/apply for product/variant drafts and individually identified physical drafts; atomic validation, reference locking, retained batch outcome and safe retries. Imports never receive stock, approve prices or enable bookings. |
| 3.7 | Inventory Admin filters cover market, hub, material, category, status/physical availability, custody and size; searchable unit/movement history. |

**Verification 2026-09-26:** All **287 backend tests** pass against PostgreSQL, including fresh schema, Task 2 upgrade without fabricated custody/history, safe downgrade guards, staff permissions/CSRF, stale reference forms, raw SQL history protection, rollback after audit failure, repeated/concurrent inventory operations, CSV validation/retries and imports racing registration, and private media onboarding/download/retirement. Backend lint, formatting, Django checks, migration drift and applied migration plan pass. Frontend **81 tests**, TypeScript, lint, formatting and production build pass. Local test media limits/procedures are fictional fixtures; no production policy was selected. Remote CI/merge has not been performed for this branch.

**Operating gates:** BUSINESS DECISION BD-09 must supply reviewed receiving/QC procedures and physical identification/exception handling; configuration approval/activation requires its own reviewed workflow. BD-08 allocation, BD-11 ownership, CA REVIEW CA-03 and LEGAL REVIEW LR-02 remain unresolved. D-22 selects S3/CloudFront target storage; BD-15 still gates upload/access/processing settings, with applicable LR-04 retention/access review. Current Admin can prepare drafts but cannot approve those policies. No real stock, hub, PIN, product, price, ownership or policy data is seeded by Task 3.

**Later workflow boundary:** Reservation, dispatch, sale, customer returns, inter-hub transfer, reinspection and retirement are not implemented. Their candidate states below remain proposals; subsequent authorized work must add the corresponding evidence and guards. Physical AVAILABLE status does not make the public serviceability endpoint bookable. See D-19 and STATE_MACHINES.md for the implemented subset.

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

## 4. Cross-app model completion

**Status:** IN PROGRESS / RE-SCOPED by D-26 on 2026-10-02. On 2026-10-03 the first settled identity/booking models were implemented and verified. This is partial progress on 4.M1, 4.M2, 4.M6, 4.M7 and 4.M12, not completion of those items or Task 4.

**Implementation 2026-10-03:** Nine models now cover customer identity, policy versions, draft booking ownership/contact snapshots, category boxes, booking OTP evidence, guest credential digests, booking policy acceptance, staff assignment history and Arrival OTP evidence. Additive migrations retain existing staff/audit/stock history and refuse unsafe evidence-removing downgrade. [DATA_MODEL.md](DATA_MODEL.md#implemented-task-4-identity-and-booking-foundation) states exact current fields/limits; the [verification/recovery record](reviews/TASK4_MODEL_FOUNDATION_2026-10-03.md) distinguishes delivered code, remaining engineering and decision gates. No persistent local database migration, public/customer/staff endpoint, provider call, UI, live confirmation or operational policy was enabled by this work.

**Verification 2026-10-03:** Full backend PostgreSQL suite **370 passed**; Ruff lint/format checks and Django system check passed; migration drift check reported no changes. The suite includes 36 new model tests and five new migration tests. Fresh test schema, additive upgrades, empty rollback and refusal of evidence-removing rollback were exercised. CI/deployment and the remaining coverage matrix are not verified by this result.

**Objective:** Finish all backend model definitions and persistence integrity required by the agreed ten-task scope, so subsequent work builds flows on a coherent schema rather than introducing known missing domain tables piecemeal.

**Dependencies:** Tasks 1–3 and retained catalogue model work; D-23–D-25; the target [DATA_MODEL.md](DATA_MODEL.md). Review open decisions before fixing cardinalities or policy-dependent constraints. An unanswered schema-shaping decision is a named blocker, not permission to mark the app complete.

**Backend work:**

- **4.M1** Complete accounts persistence: retain `StaffUser` as `AUTH_USER_MODEL`; add separate `CustomerAccount`, customer contact/authentication/recovery support and staff scope/profile support where needed. Select reviewed auth components before finalizing their persistence. No guest-account merge/claim model; record feature-disabled customer rollout and independent identity lifecycles.
- **4.M2** Complete compliance persistence: legal parties, versioned configuration/policy, purpose-specific acceptance/consent and attributable audit evidence. Extend actor typing without treating customers as staff or linking guest bookings indirectly to accounts. Preserve existing history protections and explicit retention boundaries.
- **4.M3** Complete markets persistence: market/hub/service-area definitions and approved scheduling/coverage scope needed downstream. Keep city activation data-driven, custody distinct from ownership and unapproved hub/slot policy unconfigured.
- **4.M4** Complete catalogue persistence: product/variant/material/purity/category, pricing revisions, private/public media identity, storefront publication/metadata and redirect/alias history required by the roadmap. Reuse existing models; changing slugs or publishing media remains a later controlled workflow.
- **4.M5** Complete inventory persistence: physical units, commitments/reservations, allocation/custody/movement/QC evidence and applicable return/exception links. Enforce incompatible commitment exclusion and preserve original movements; do not approve allocation, damage or QC policy through a default.
- **4.M6** Complete trials persistence: versioned TrialPlans, unified `TrialBooking`, required historical contact/address facts, immutable guest/account mode and ownership, boxes/items, booking phone challenge/verification, booking-specific guest access and lifecycle/idempotency evidence. Guest `customer_account` remains permanently null, even when a phone matches a registered account.
- **4.M7** Complete delivery persistence: assignment/reassignment history, combined visit and outcomes, Arrival OTP, manifests/dispatch-document snapshots and custody references. Represent no purchase independently of financial settlement and hub return; no second purchase OTP.
- **4.M8** Complete billing persistence: Purchase/PurchaseItem snapshots, tax revisions, controlled invoice series, sealed invoice header/lines, adjustment and after-sale case/document references. Retain currency/issuer/customer evidence; operational tax/return decisions remain gated.
- **4.M9** Complete payments persistence: upfront obligation/policy context, PaymentContext/Attempt, verified receipts/provider events, allocations, refunds, gateway settlement batches, bank evidence and reconciliation links. Protect against duplicate receipts and over-allocation/refund; allow pending attempts to progress without rewriting immutable evidence.
- **4.M10** Complete notifications persistence: durable notification intent/work items, delivery attempts/status, template/purpose references and retry/deduplication evidence. Booking OTP, Arrival OTP and future customer authentication have distinct purposes; no provider sending is implemented here.
- **4.M11** Complete analytics persistence needed by Task 10: minimized attribution/event identity, provenance, deduplication and domain correlation. Map reviewed KPI/cost inputs to retained domain evidence, configuration or controlled imports so known reporting inputs are not a hidden model backlog. Record query-only projections rather than inventing tables; phone numbers are not unique-person identity and client events are not revenue evidence.
- **4.M12** Reconcile the app coverage matrix against every later task; map actual classes/files, migrations and model tests. Review FK/delete semantics, checks/indexes, cross-app dependencies and ORM/bulk/SQL history protection. Verify migrations on fresh and upgraded PostgreSQL databases, concurrency-sensitive persistence, recovery implications and no pending model drift. Resolve or explicitly block schema-shaping questions before declaring completion.

**Frontend work:** None beyond necessary model-contract impact notes. Preserve existing frontend source; no new screens, styling, API integration or final UI acceptance in this task. Frontend TypeScript types are not Django models and are updated with their consuming workflows.

**Admin/operations work:** Review schema ownership, privacy exposure, migration deployment/recovery and retained evidence. Do not add generic editing of financial/stock states or call new model registration a completed operating workflow.

**Business/account prerequisites:** D-23–D-25 are binding. BD-01/BD-02 are only partially resolved; unresolved quantities, booking/purchase cardinalities, recovery, retention, tax/document, hub/slot and return policies must not become accidental constraints. Values can stay gated where schema is representable; shape-changing questions must be decided before the affected model acceptance.

**Tests:** Verify guest/account check constraints and irreversible ownership, required accepted-booking snapshots, protected historical FKs, account/staff independence, purpose-bound challenge/access persistence, duplicate/expired challenge evidence, active inventory commitment races, exact monetary/currency bounds, receipt uniqueness, invoice header/line sealing and permitted correction evidence. Exercise ORM, bulk and database bypass boundaries where protection is claimed; use PostgreSQL for database-specific guarantees. Workflow/API/provider tests remain with their later tasks.

**Definition of Done:** Every app in the coverage matrix is reviewed and every required model for the agreed scope is implemented with migrations, integrity guards and passing relevant tests. Actual class/file/migration/test evidence is recorded; fresh/upgrade checks and deletion/history review pass. No known required model is silently deferred to Tasks 5–10. Policy values may remain explicitly unconfigured, but unresolved schema requirements prevent full completion. This is model readiness, not authentication, checkout, financial, compliance or launch readiness.

**Search acceptance criteria:** Preserve stable public catalogue identity and the persistence needed for approved publication, metadata and redirect history; keep customer/guest/staff/financial records outside public content. Task 5 owns API/rendering and Task 10 deployed SEO checks.

**Non-goals:** New workflow/API/UI implementation, issuing sessions/tokens, contacting providers, live policy activation, resetting databases or deleting migrations. Split model modules are acceptable if correctly registered; completing this scope does not prohibit reviewed migrations for genuinely new future requirements.

## 5. Storefront and guest Trial Cart

**Overall status:** INHERITS IN-PROGRESS STOREFRONT FOUNDATIONS; original 5.1–5.9 cart work remains planned. Final presentation awaits owner design input. Complete both retained blocks below for Task 5 acceptance.

<a id="4-customer-storefront-and-scalable-ui"></a>

### Retained storefront work (original 4.1–4.7)

**Execution owner since D-26:** Task 5. Original IDs, requirements, source and historical progress are retained below. References to Task 4 in dated continuation notes describe its former scope, not the current model-only task.

**Status:** IN PROGRESS — design-independent technical foundations may continue under D-20/D-21; visual/UI design and final presentation are AWAITING OWNER DESIGN INPUT. Existing policy/publication gates remain. Implementation evidence will be recorded after verification.

**Continuation 2026-09-27:** Development and verification resume under the same authorization. The SEO plan update preserves exactly ten tasks and all 95 original numbered subtasks; their identifiers and order were checked. This documentation verification does not claim application completion, deployment or live indexation.

**Scope clarification 2026-10-02:** The [frontend design input gate](#frontend-design-input-gate) applies to 4.1–4.6 and the presentation-related tests/Definition of Done below. Backend 4.7, frontend contracts, routes, SSR, SEO/non-indexing structures and other design-independent checks may continue. Existing styled pages are provisional and unapproved; this update neither finalizes nor removes them. Task 4 is not complete merely because its technical foundation passes tests.

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

**Search acceptance criteria — public page implementation (4.1–4.7):**

- Render meaningful landing, category and product content on the server, with semantic landmarks, a clear page heading hierarchy and crawlable navigation/internal links. Provide visible breadcrumbs whose links agree with the route hierarchy. Critical product/category information must not exist only after hydration.
- Establish clean, stable product/category paths with a canonical product identity independent of mutable names, SKU parsing, current material or selected city. Add narrowly scoped slug/metadata support where needed without redesigning the completed catalogue. Define normalization and slug-change history for the Task 10 redirect strategy; unknown or withdrawn routes must have deliberate status/visibility handling rather than an indexable success page for nonexistent content.
- Generate page-specific titles, meta descriptions, absolute canonical URLs and Open Graph metadata from approved public content and configured public origin. Reuse a consistent Arka Tara identity. Query variants, sort/filter combinations and pagination must have deliberate crawl/index/canonical rules; meaningful pagination must not all canonicalize to page one. Avoid unbounded crawlable query combinations, duplicate city/product copies and made-up URLs in navigation.
- Generate truthful Schema.org Organization and WebSite identity plus Product and BreadcrumbList data where their required factual content exists. Structured data must agree with visible content and canonical identity. Do not invent Offer prices/availability, reviews, ratings, addresses or logos to obtain rich results. LocalBusiness is conditional on verified appropriate business/location facts, not on an assumed public storefront. Incomplete draft data must not be promoted into a public claim.
- Deliver approved public media with useful factual alt text, appropriate dimensions/aspect ratios, responsive sizing and optimized formats/delivery where supported. Lazy-load noncritical imagery; avoid delaying the main visible image unnecessarily. Respect Task 3's private draft boundary and BD-15 production-storage choice. Reserve layout space and limit unnecessary JavaScript, font and video cost to support mobile Core Web Vitals.
- Tests inspect server-rendered HTML, metadata/canonicals, semantic links/headings, structured-data parity, invalid/duplicate URLs, media publication boundaries and mobile layout/performance. Verify extensibility using another configured category/material/market without generating thin location pages. Measure performance with available tools and record limitations; Task 10 owns deployed field monitoring and final launch verification.

**Non-goals:** Customer registration, Hyderabad/Gold launch, independent category checkouts, invented trust claims or customer promises about unanswered policies.

### Guest Trial Cart and multi-box logic (original 5.1–5.9)

**Status:** PLANNED / NOT STARTED.

**Objective:** Let customers assemble category-specific boxes into one intended home-visit booking without creating an account or reserving stock.

**Dependencies:** Task 4's completed model scope, Task 2 reference contracts and the retained storefront API/rendering block above; reviewed TrialPlan and booking structure. Production policy-dependent behaviour waits for BD-01/BD-03/BD-13 choices.

**Frontend work:**

- **5.1** Allow browsing without login.
- **5.2** Persist Trial Cart intent in `localStorage` across refresh, page/category navigation and browser restart for an approved configured period. Handle unavailable storage and stale or malformed data safely; local state is never proof of availability.
- **5.3** Implement configurable cart expiry. Seven days is a suggestion, pending BD-13, not a default approved by the roadmap.

Render category boxes and their approved plan limits clearly, guide the customer toward a combined booking, and explain authoritative revalidation results. Avoid storing sensitive checkout/address/payment data in the anonymous cart without an independently justified and reviewed need.

**Backend work:**

- **5.4** Enforce category-specific TrialBoxes: for example, a Ring box contains Rings only.
- **5.5** Support multiple category boxes, such as Ring + Chain, within one TrialBooking and delivery booking. Multiple boxes must not cause independent customer checkouts.
- **5.6** Implement configuration/validation workflows on Task 4's versioned TrialPlans; possible dimensions include Market, Material, Category, piece limit, upfront-collection policy and active flag. Do not select deposit scope or material-mixing policy implicitly. Never hard-code INR 29/49 or other amounts into frontend logic.
- **5.7** Make maximum TrialBoxes configurable. Two boxes is a suggested pilot value only, pending BD-13.
- **5.8** Ensure anonymous cart additions never reserve physical inventory.
- **5.9** Treat frontend price, category, deposit amount and availability as untrusted. Validate against authoritative backend facts at booking and subsequent authoritative operations.

**Admin/operations work:** Configure reviewed TrialPlans and limits, with validated activation and change history. Preserve what was offered/committed historically when plans later change.

**Business/account prerequisites:** **BUSINESS DECISION BD-01** for upfront collection basis/amount/classification/application under D-25; **CA REVIEW CA-01** for treatment; **BD-03** for counting pieces/variants, repeated category boxes, mixed materials and substitutions; **BD-13** for expiry, item and box limits and eligibility. Missing settings must not silently receive roadmap sample values.

**Tests:** Check anonymous access, local persistence/expiry with explicit test settings, malformed/tampered cart data, one-category invariants, multiple boxes in one checkout, plan changes and stale inventory. Confirm no InventoryUnit reservation or stock decrement occurs from any cart operation.

**Definition of Done:** A customer can assemble and restore valid intent without an account or inventory hold; multi-box intent stays attached to one booking journey; backend validations reject tampering. Only approved production settings are activated.

**Search acceptance criteria — cart and query state (5.1–5.9):** Keep carts/selection states out of indexing and sitemaps; do not place customer selections, contact information or tokens in public canonical/Open Graph URLs. Use this task's retained 4.1–4.7 public catalogue URL policy for links back to products/categories. Cart, attribution and UI-only query parameters must not create indexable copies or an unbounded crawl space; preserving a useful public filter/page uses the retained 4.1–4.7 deliberate policy. Tests cover direct cart access, noindex output, query normalization and absence of private state in public metadata while preserving local cart behaviour.

**Non-goals:** Reserving inventory, collecting deposits, choosing deposit scope from box count, creating customer accounts or assuming suggested numeric limits are approved.

## 6. Trial booking and inventory reservation engine

**Status:** PLANNED / NOT STARTED.

**Objective:** Convert guest or enabled account checkout into reliable, concurrency-safe bookings using Task 4 models, booking phone proof and verified upfront collection; preserve permanent guest separation and explicit operating-policy gates.

**Dependencies:** Tasks 1–5 and reviewed booking/inventory state machines. Reservation/acceptance behaviour depends on BD-02; address/service/slot behaviour on BD-07/BD-08/BD-13. External providers remain development fakes until their later tasks.

**Frontend work — checkout:**

- **6.1** Collect name, mobile, address and PIN code without requiring an account. Submit all boxes as one pending booking intent; complete purpose-bound booking phone OTP and terms evidence before initiating upfront payment. Show confirmation only after backend verification and valid acceptance. Account checkout uses authenticated ownership when enabled; guest checkout never creates or links an account.

**Backend work:**

- **6.2** Resolve serviceability from backend Market/ServiceArea configuration.
- **6.3** Revalidate each requested item: it exists, is active, belongs to the correct market/hub and category, is trial-eligible, and is available. Resolve actual physical allocation server-side according to the approved fulfilment policy.
- **6.4** Protect concurrent bookings with atomic database operations and appropriate transactional locking/constraints. One physical unit cannot be committed incompatibly or sold twice.
- **6.5** Implement atomic booking workflows on Task 4's TrialBooking → TrialBox → TrialBoxItem models, preserving immutable checkout mode/ownership and historical contact/address facts, the shared visit and joint fulfilment. Reject direct or indirect guest-account linking and contact-based order-history lookup.
- **6.6** Implement the reviewed booking lifecycle and allowed/forbidden transitions with explicit actor permissions and policy guards.
- **6.7** Support temporary reservation expiry for incomplete upfront-payment attempts once the hold policy is approved. D-25 requires booking OTP before payment initiation and verified upfront collection before confirmation; hold trigger/duration, release and late-payment handling remain BD-02. Payment success alone must not revive an expired commitment.
- **6.8** Create provider abstractions, considering PaymentProvider, NotificationProvider and OTPProvider, with fake/development implementations initially. Keep secure OTP domain logic separate from transport. Fakes must be confined to safe environments and visibly distinguishable from real verification.

**Customer authentication and access scope (D-23–D-25):** Implement reviewed customer signup/login/session-or-token, logout/revocation, account recovery and account-history authorization on 4.M1 persistence, disabled server-side initially. Use explicit customer principal checks; never accept customer credentials for staff operations. Guest access uses a booking-specific expiring/revocable grant, not phone lookup; no claim/import flow exists. Recycled-number recovery and safe disclosure of old account history are activation gates, not solved by OTP or phone uniqueness. Implement and test both checkout modes while launch keeps customer features disabled; a later release requires readiness approval, not a date switch.

**Frontend work — policy pages:**

- **6.9** Build Terms, Privacy Policy, Try-at-Home Policy, Refund/Cancellation Policy, Contact and grievance-information pages. Drafting/page support does not approve legal wording or unresolved policies; flag final content for **LEGAL REVIEW LR-04**, alongside the commercial-policy constraints in LR-01.

**Admin/operations work:** Provide booking visibility and controlled exception handling, reservation-expiry monitoring and reviewable state/history. Ensure retries do not create duplicate commitments. Document operational handling for conflicts and incomplete attempts once the relevant policies are approved.

**Business/account prerequisites:** **BUSINESS DECISION BD-01/BD-02** for deposit and reservation commitment; **BD-03** for quantities/substitution; **BD-07/BD-08/BD-13** for service areas, scheduling, hub sourcing and limits; **BD-06** for any live pilot before gateway integration. **LEGAL REVIEW LR-01** covers relevant policy constraints; other customer terms/privacy/grievance requirements must receive the applicable legal review recorded in [DECISIONS.md](DECISIONS.md).

**Tests:** Use PostgreSQL concurrency tests for competing allocations and rollback; verify replay-safe bookings, serviceability, category/market constraints, inactive/stale items and untrusted money/ownership fields. Exercise booking OTP expiry/replay, wrong purpose/phone/booking, concurrent consumption and phone changes before payment. Test guest and account checkout, signed-in guest separation, forbidden historical linking, cross-account access, disabled rollout, revocation/recovery and applicable CSRF/CORS. Test expiry/payment races only with approved policy fixtures and clearly isolated provider fakes. Check private caching and legal-page access.

**Definition of Done:** With approved settings, one combined booking is created atomically, ownership/snapshots remain protected, booking phone proof precedes payment initiation, and confirmation requires verified collection plus valid acceptance. Competing requests cannot overcommit stock. Guest access and prepared customer authentication/checkout pass identity/privacy tests; customer features remain disabled until readiness approval. Fakes never impersonate production evidence. Legal pages stay draft until reviewed.

**Search acceptance criteria — booking privacy and policy pages (6.1–6.9):** Keep checkout, booking confirmation/status and customer-specific routes non-indexable and absent from public sitemaps. Enforce booking-specific guest, owning-customer or authorized-staff access, appropriate private caching and responses before returning personal details; noindex is not authorization. Tokens, addresses, phone numbers and booking facts must not leak through titles, structured data, canonical URLs or previews. Reviewed public legal/contact pages may use Task 5's retained shared metadata/semantic conventions; do not index draft legal wording as approved policy. Tests cover unauthorized responses, caching, metadata leakage and reviewed-page links.

**Non-goals:** Razorpay integration, WhatsApp/SMS delivery, live pilot payment workarounds, invented upfront amounts/treatment, automatic customer-account activation, guest-order claims, intercity fulfilment or legal-compliance certification.

## 7. Delivery operations and delivery-agent portal

**Status:** PLANNED / NOT STARTED.

**Objective:** Control staff access, dispatch, attended trial, purchase selection and physical returns while preserving custody and authoritative pricing.

**Dependencies:** Task 4 models, Task 6 booking workflows and Task 5 catalogue/media support; reviewed assignment, OTP, inventory and booking state machines. Purchase selection/domain records are needed here before Task 8; genuine payment confirmation and complete production invoicing remain later dependencies for a live sale flow.

**Backend work:**

- **7.1** Implement staff provisioning, authentication, MFA and RBAC on Task 4's StaffUser/scope models, considering ADMIN, OPERATIONS, INVENTORY_MANAGER, DELIVERY_AGENT and FINANCE. Enforce staff identity, action and assignment/market/hub permissions; customer credentials and Django `is_staff` alone cannot authorize delivery operations.
- **7.2** Implement assignment/reassignment workflows using Task 4's DeliveryAssignment and visit-history models. A normal booking's boxes share one visit; do not infer separate assignments/checkouts from box count.

**Frontend work:**

- **7.3** Build the mobile staff portal on a dedicated host, requested as `staff.arkatara.com`, using `/staff/api/v1/` endpoints. Public `arkatara.in` remains unchanged; verify domain ownership/aliases and credential/CORS/CSRF boundaries before deployment. A delivery-detail path may be `/deliveries/{booking_id}`; host/path separation never replaces assignment authorization.
- **7.4** Show customer, address, phone, TrialBoxes, products, SKUs, photos and current state only within authorized access.

Provide OTP entry, item selection, controlled visit actions and accurate payment-pending/return states. Later Task 8 adds authoritative payment status and QR display. Do not imply that pressing a selection or completion button confirms payment.

**Backend work — trial and custody controls:**

- **7.5** Prevent agents from manually overriding authoritative prices through either the UI or API.
- **7.6** Support generation of a Delivery Challan / dispatch document distinct from a sale invoice. Production format, required contents and timing require **CA REVIEW CA-02** and **LEGAL REVIEW LR-03**.
- **7.7** Implement Arrival OTP generation, secure hashing/storage, expiry, attempt limits and verification timestamp. Do not log plaintext OTPs or expose them to agents as a substitute for customer verification. Delivery through real messaging providers is deferred to Task 9.
- **7.8** Allow backend Arrival OTP verification to transition the booking from `OUT_FOR_DELIVERY` to `IN_PERSON_TRIAL`, with authorized assignment and state checks.
- **7.9** Build selection/no-purchase workflows on Task 4's visit and Purchase/PurchaseItem models. An authorized assigned agent records chosen pieces or a no-purchase outcome; the backend prices selections and records the visit result without inventing a zero-value sale. Selection is not financial confirmation, and visit closure does not settle/refund money or restock units.
- **7.10** Do not require a second OTP solely for item purchase/selection.
- **7.11** Route unpurchased units through `RETURNING_TO_HUB` → `QC_PENDING` → `AVAILABLE`, with recorded return and successful QC. Units still physically with the agent must not become available; failed-QC outcomes remain restricted under the approved policy.

**Admin/operations work:** Manage assignments, packing/dispatch, movement evidence, return receipt, QC and restricted exceptions. Prepare an agent training flow that respects customer choice, price authority, data access and the approved response to failed visits/OTP verification.

**Business/account prerequisites:** Staff identities and approved access responsibilities; **BUSINESS DECISION BD-05/BD-06** for pending-payment handling/handover and pilot payment methods; **BD-07** for failed visits, rescheduling and exceptions; **BD-09** for tagging, loss/damage and QC; **BD-04** for price commitment; **CA REVIEW CA-02**, **LEGAL REVIEW LR-01/LR-03** for dispatch and custody responsibilities. These gates prevent a development trial demonstration from being presented as an authorized live operating process.

**Tests:** Check RBAC and direct-object access attempts, reassignment, forbidden transitions, single booking/multiple boxes, non-overridable prices and secure OTP expiry/replay/attempt limits. Verify selection integrity, no second OTP requirement, movement/custody consistency, physical-return evidence and failed/successful QC gating.

**Definition of Done:** Authorized agents can perform the reviewed development dispatch-to-trial-to-selection/return flow; backend OTP verification controls trial start; selected items create reliable purchase intent; stock cannot bypass return/QC or be falsely marked sold. Production dispatch documents and payment/handover policies require their own approvals.

**Search acceptance criteria — staff surfaces (7.1–7.11):** Apply non-indexing/private-cache behaviour to staff and delivery routes and exclude them from public navigation/sitemaps. Server-side authentication and assignment checks must prevent unauthorized content disclosure independently of robots directives. Page titles, previews and errors must not reveal customer, address, OTP, assignment or product-selection details to unauthenticated visitors. Extend the existing permission tests to response headers, rendered metadata and direct route requests; do not build separate staff SEO content.

**Non-goals:** Treating a selected item as paid, treating dispatch as sale, payment/messaging provider integration, unrestricted agent Admin access or silently deciding liability/handover policies.

## 8. Razorpay, GST, billing and reconciliation

**Status:** PLANNED / NOT STARTED. Intentionally later than core booking/trial logic.

**Objective:** Implement verified digital collection and a traceable financial record from deposit and sale through refunds, gateway settlements and bank matching.

**Dependencies:** Task 4 financial models and Tasks 6–7 domain workflows; reviewed provider interfaces and state machines. Resolve the policy/review questions needed by the actual live payment flow before activating it.

**Backend work:**

- **8.1** Implement upfront obligation/collection/application workflows on Task 4's BookingDeposit/payment models, separately from jewellery value and Purchase; do not derive the commercial deposit scope from box count.
- **8.2** Integrate Razorpay for booking deposits according to the approved policy. Calculate amounts server-side.
- **8.3** Implement the final Purchase calculation from authoritative selected items, reviewed prices/taxes and approved deposit application. Preserve snapshots and explain all components.
- **8.4** Create/update attempt records and append verified evidence using Task 4's PaymentAttempt/GatewayTransaction models with explicit links, currency/amount, state and provider references as appropriate.
- **8.5** Use one underlying current final-payment attempt/context for both Agent QR and customer payment URL. Exposing two access methods must not create independent payable transactions. Legitimate future retries or changed selections require controlled supersession and BD-05 policy, not a second concurrent charge for the same obligation.
- **8.6** Verify Razorpay signatures/webhooks; process events idempotently with checks against expected amount, currency, account and payment context. Handle duplicates, delayed delivery and out-of-order events without double confirmation, sale or document issuance.
- **8.7** Never store sensitive card information.
- **8.8** Retain gateway references needed for traceability and reconciliation, without placing secrets or unnecessary sensitive payment data in logs.
- **8.9** Implement reviewed configuration/lookup workflows on Task 4's TaxCode/HSN/rate/effective-date persistence. Do not scatter GST percentage constants through code.
- **8.10** Implement controlled numbering, issuance and reviewed correction workflows on Task 4's TaxInvoice/TaxInvoiceItem/adjustment models, preserving sealed headers and line membership/content.
- **8.11** Distinguish invoice total, deposit already received/applied and remaining balance. Applying a deposit must not silently change jewellery selling price. Illustrative INR 5,000 less INR 49 equals INR 4,951 outstanding only when that application policy is approved; the example does not approve the deposit amount or universal treatment.
- **8.12** Implement authorization, provider processing and reviewed adjustments using Task 4's traceable Refund/source-collection models. Refund requests are not automatically successful refunds.
- **8.13** Reconcile PaymentAttempt, gateway payment, gateway settlement and bank settlement, including identifiable fees/adjustments as reviewed. Surface unresolved mismatches instead of marking them matched automatically.
- **8.14** Mark every uncertain GST/accounting treatment **CA REVIEW** and obtain the required confirmation before enabling dependent production behaviour.

**Frontend work:** Present backend-calculated amounts, payment states and invoice/balance breakdown; expose the shared QR/link payment context on the agent/customer surfaces; show pending/failure/retry outcomes according to approved policy. Browser callbacks and screenshots cannot establish financial success.

**Admin/operations work:** Provide restricted payment/refund/invoice administration, gateway/bank reconciliation, mismatch queues and review evidence. Preserve immutable invoices and attributable adjustment records rather than editing historical financial facts. Document reconciliation ownership and escalation.

**Business/account prerequisites:** Final business/legal identity; business current account; applicable GST registration; CA-confirmed accounting policy and Trial Deposit treatment; Razorpay business account and KYC; settlement bank account. Business collections use business accounts. **BUSINESS DECISION BD-01/BD-02/BD-04/BD-05/BD-06/BD-10/BD-11**, **CA REVIEW CA-01/CA-02/CA-03** and relevant **LEGAL REVIEW LR-01/LR-02/LR-03** remain explicit gates. Provider product/API capabilities and requirements must be checked against current official documentation when implementation begins.

**Tests:** Verify server-side amounts, signatures, replay/out-of-order handling, duplicate callback/webhook races, shared QR/link context, controlled retries, late payment after reservation expiry and no double sale/collection. Check deposit application/refund/retention only against approved policy, immutable snapshots/invoices, numbering concurrency, tax effective dates, refunds and reconciliation mismatches. Complete test/live verification only within approved accounts and authorized scope.

**Definition of Done:** Verified backend records govern collection; one obligation is not accidentally paid twice through QR/link channels; financial records remain traceable and immutable where required; refunds and settlement mismatches can be reconciled. CA/legal approvals and account readiness for activated behaviour are recorded. Passing software tests alone is not a claim of tax/legal compliance.

**Search acceptance criteria — financial privacy (8.1–8.14):** Keep customer payment/status/receipt/invoice routes and document downloads non-indexable, appropriately access-controlled and privately cached. Apply suitable response-level indexing controls to non-HTML documents as well as page metadata. Exclude them from public sitemaps; never expose payment tokens, invoice customer data or balances in Open Graph/canonical/structured-data output. Verify unauthorized requests and shared QR/link destinations without relaxing payment-context authority. Public Product data remains Task 5's retained catalogue responsibility, not a copy of a private purchase or invoice.

**Non-goals:** Inventing GST rates, deposit recognition or retention rules, storing card details, silently choosing physical handover policy, replacing CA review with code or sending live messages before Task 9.

## 9. WhatsApp Cloud API and SMS / TRAI-DLT integration

**Status:** PLANNED / NOT STARTED. Intentionally after core business logic.

**Objective:** Deliver purpose-specific booking phone verification and approved transactional notifications, plus the same Arrival OTP/final-payment context across its approved channels, with appropriate provider setup and consent boundaries. Customer authentication messages remain gated with customer activation.

**Dependencies:** Task 6 provider contracts, Task 7 OTP/visit domain and Task 8 payment/invoice domain for corresponding messages. Templates may be prepared earlier; production sending requires account approvals and reviewed content.

**Backend work:**

- **9.1** Integrate WhatsApp Cloud API behind the reviewed notification boundary.
- **9.2** Store credentials/tokens securely with environment separation, least privilege and a rotation process.
- **9.3** Configure WhatsApp webhook/status callbacks with the provider's required verification, safe event processing and delivery-state tracking.
- **9.4** Create purpose-specific templates for Booking phone OTP, Booking confirmed, Out for delivery, Arrival OTP, Payment link, Payment success, Deposit refund, Invoice issued and Cancellation. Future customer signup/login/recovery templates remain disabled until customer activation and separately reviewed. Template wording must reflect approved policies and true domain events; delivery status is not business/payment truth.
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

**Tests:** Verify one Arrival OTP challenge across channels and no proof reuse between booking, arrival and customer authentication. Check retries, bounded resend/verification, callback validation/idempotency, secret redaction, provider failure and test/live isolation. Disabled customer features must not send login messages. Delivery cannot verify an OTP or set payment to paid; preserve QR/link identity and never infer marketing permission from service details.

**Definition of Done:** Approved transactional templates deliver via configured providers with traceable status and safe retries; OTP/payment authority remains in the business domain; prerequisites and permissions are satisfied for production sending; communications consent boundaries are implemented and reviewed.

**Search acceptance criteria — notification links (9.1–9.11):** Use the intended canonical public domain for ordinary catalogue links and preserve the protected payment/booking context for transactional links. Message redirects/previews must not expose OTPs, customer details, invoice data or bearer tokens to search metadata, public analytics or unrelated preview providers. Reuse destination controls from Tasks 6–8; link previews do not justify weakening access controls or creating public copies. Test template output, redaction and redirect containment without producing duplicate payable links or unsolicited SEO/marketing messages.

**Non-goals:** Marketing campaigns, assuming marketing consent, second purchase OTP, separate OTP per channel, duplicate payment creation or claims that DLT/provider approval establishes overall legal compliance.

## 10. Production hardening, analytics and expansion validation

**Status:** PLANNED / NOT STARTED.

**Objective:** Verify production readiness, secure operations, measurable unit economics and configuration-led expansion. This task completes/validates controls introduced earlier; it does not defer basic security or auditability until launch.

**Dependencies:** Relevant Tasks 1–9 and approved live operating policies, business accounts and external reviews. Expansion simulations use controlled test data; a successful simulation does not authorize a commercial launch.

**Backend work:**

- **10.1** Implement/validate event ingestion and reporting on Task 4's analytical persistence (not a deferred new model layer): `PRODUCT_VIEWED`, `TRIAL_ITEM_ADDED`, `TRIAL_BOX_COMPLETED`, `CHECKOUT_STARTED`, `DEPOSIT_STARTED`, `DEPOSIT_PAID`, `BOOKING_CONFIRMED`, `DISPATCHED`, `ARRIVAL_VERIFIED`, `TRIAL_STARTED`, `PRODUCT_SELECTED`, `FINAL_PAYMENT_STARTED`, `FINAL_PAYMENT_PAID`, `NO_PURCHASE`, `RETURN_QC_COMPLETED`. Define provenance, timestamps, correlation and deduplication; frontend behavioural events are not authoritative financial evidence.
- **10.2** Capture permitted acquisition attribution: `utm_source`, `utm_medium`, `utm_campaign`, `utm_content` and `referrer`, with reviewed retention/privacy controls and no unnecessary customer-sensitive data in analytics payloads.
- **10.3** Build KPI reporting for website visitors; Trial Cart add rate; Trial Box completion; booking conversion; deposit conversion; delivery success; trial-to-purchase conversion; zero-purchase rate; AOV; pieces tried versus purchased; category conversion; refund rate; delivery cost; inventory utilisation; inventory turnaround; agent utilisation; payment failure; refund speed. Define each numerator/denominator, cohort/time basis, exclusions and source so retries and multiple boxes do not inflate customer visits or sales. Relate revenue/conversion to visit, fulfilment, inventory and return/QC costs for pilot economics.
- **10.4** Complete privileged-action audit coverage for price changes, inventory changes, refunds, invoice actions, staff assignments and payment administrative actions. Preserve attribution and prior/new facts without storing secrets.
- **10.5** Validate HTTPS, applicable secure cookies/CSRF, rate limits, explicit customer/staff principal separation, staff MFA and permission/assignment checks, secret management, dependency scanning, webhook verification and backups. Verify guest history remains unlinked and customer rollout is disabled until recovery/privacy/security readiness is approved. Match the actual hosts and authentication transport.
- **10.6** Monitor server/API errors, payment failures, webhook failures, notification failures, inventory inconsistencies and background-job failures, with actionable ownership and alerts.
- **10.7** Simulate Gold activation: change `GOLD` from `COMING_SOON` to `ACTIVE`; add Gold product, purity, inventory and pricing configuration. Demonstrate the existing frontend/backend supports the reviewed scenario with no schema migration required solely because Gold launches. Gold's actual pricing, trial/handling policies and external reviews remain prerequisites for live activation.
- **10.8** Simulate Hyderabad activation: change `HYD` from `COMING_SOON` to `ACTIVE`; configure Hyderabad Hub, ServiceAreas, Inventory, TrialPlans and delivery agents. Demonstrate existing frontend/backend operation with Bengaluru and Hyderabad stock kept separate and no schema redesign solely for city activation.

**Frontend work:** Validate the complete mobile customer and staff journeys, accessibility, error/recovery states, attribution/events and accurate financial/status presentation. Run both expansion simulations through customer selectors, listings, cart, booking and authorized staff surfaces; do not only test database inserts.

**Admin/operations work:**

- **10.9** Write operational SOPs covering inventory receiving, product onboarding, packing, dispatch, arrival verification, customer trial, sale, payment failure, no purchase, return, QC, refund, damage, loss, and invoice correction/escalation. Link each SOP to approved policy, role permissions, evidence and exception handling; do not turn unresolved policy into an operating instruction. Keep shared SOP documentation canonical rather than copying it into both repositories.
- **10.10** Create and execute a production readiness checklist covering inventory reconciliation, concurrency testing, Razorpay test/live verification, webhook retry/idempotency testing, distinct booking/Arrival OTP flows, staff/customer credential isolation, permanent guest ownership, customer feature/recovery gates, CA invoice review, Delivery Challan review, refund testing, backup restore testing, legal pages, support/grievance workflow and monitoring. Record evidence, reviewer, unresolved blockers and release authorization rather than treating a checklist's existence as readiness.

**Business/account prerequisites:** Reviewed business KPI definitions, cost inputs and reporting responsibilities; approved support/grievance ownership; operational staffing/training; approved production provider accounts/configuration; resolved decisions for every enabled flow. **BUSINESS DECISION BD-12/BD-13** and related pricing, service, stock and trial policies gate real expansion; **CA REVIEW** and **LEGAL REVIEW** evidence must cover enabled products/entities/locations and customer/financial documentation. Monitoring, analytics and backup service selection/retention must be explicitly approved or recorded if still open.

**Tests:** Validate the event-to-KPI mapping against known scenario data, deduplication, correct visit/box/purchase denominators and financial truth sources. Exercise concurrency, access/abuse controls, provider retries, notification failures, inventory reconciliation, approved purchase/refund exceptions and real backup restoration in an appropriate environment. Run both activation simulations without schema changes for activation alone. Check that audits and alerts detect meaningful faults, not merely that log statements exist.

**Definition of Done:** Readiness evidence demonstrates reliable operations, traceability, restoration and monitoring; KPIs are defined and reconcile to trusted facts; both expansion simulations pass; reviewed SOPs and customer/legal/financial materials exist. Outstanding gates are listed explicitly and block only the affected live capabilities. Production release and real Gold/Hyderabad activation require their own authorized readiness decisions.

**Search acceptance criteria — launch and ongoing verification (10.1–10.10):**

- Implement and verify environment-aware `robots.txt` and XML sitemap generation, including eligible public product/category URLs and sitemap indexes if needed. Use canonical absolute URLs and meaningful modification dates. Omit drafts, inactive/duplicate/private/tokenized routes and noncanonical query combinations. Protect staging/preview environments; robots exclusions alone do not secure private data, and blocked crawling must not undermine intentional noindex verification.
- Verify HTTPS and preferred-domain consistency for `arkatara.in`, configured host aliases, canonical/Open Graph/schema URLs and sitemap locations. Implement the reviewed permanent 301 redirect strategy for moved slugs/URLs and host normalization; test redirect targets, chains, loops and query handling. Check genuine missing routes return proper 404 responses instead of soft-404 success pages. No DNS, domain purchase or production deployment is implied by the brand-domain instruction.
- Establish Google Search Console and Bing Webmaster Tools readiness: authorized property/account access, owner-approved verification method, sitemap submission and deployment checks. Record submission/verification evidence when access and a deployed site exist; otherwise retain a named readiness dependency. Check indexation of eligible pages after deployment and identify exclusions/canonical mismatches using available tools, without promising indexing or ranking.
- Verify branded discoverability for Arka Tara, arkatara and Bangalore/Bengaluru variants, and monitor meaningful non-branded Silver/S925/home-trial/category intent. Record impressions, clicks, relevant queries and landing-page performance where available; maintain natural useful content and do not claim rank guarantees. Search analytics retention/account ownership remain subject to existing BD-15/LR-04 gates where applicable.
- Measure mobile Core Web Vitals and page performance using lab checks before launch and available field data after deployment. Inspect rendered content, image/font/JavaScript cost and regressions on representative landing/category/product pages. Record evidence, constraints and remediation; do not claim field results from local scores.
- Extend the existing Gold/Hyderabad simulations to URLs, canonical identity, metadata, structured data, navigation and sitemap eligibility. Activation must not duplicate public catalogue identities, invent stock/local locations or require a catalogue rebuild. Substantial future city content requires actual customer value and verified service facts.
- Add these checks and their accountable owner to 10.10's readiness checklist and 10.9's relevant operational SOPs. Validate Task 5's retained 4.1–4.7 page features on the deployed configuration rather than reimplementing them. Unavailable production hosting/DNS/search-account access remains an explicit launch dependency; it does not reopen Tasks 1–3 or block independent Task 4 model implementation.

**Non-goals:** Launch approval by implication, investor-grade claims unsupported by evidence, collecting unnecessary personal analytics data, replacing professional reviews, introducing microservices for hypothetical scale or inventing business metrics from unreliable event counts.

## Progress and change discipline

**Documentation resequencing verified 2026-10-03:** Updated the nine canonical references, root/application guides and READMEs, and the dated readiness review. Checked all ten task headings, all 95 original subtask IDs retained once in their original relative order, the 12 new 4.M checklist items, and 225 local Markdown links/anchors across 16 files. `git diff --check` passes in the three repositories. This is documentation verification only: no application source, migrations or databases changed, and no new application test run or model-completion claim is made. Pre-existing AWS edits and the root tracking-list change were preserved.

After Task 4, a genuinely new requirement or discovered schema defect may need a reviewed migration and scope amendment. This is not a promise of zero future schema changes. Known required persistence cannot be deferred while marking Task 4 complete. Keep model readiness distinct from workflow/provider/UI and live acceptance.

Keep task/subtask identifiers stable. When work is later authorized and starts, record actual status, responsible owner, linked implementation/review evidence and relevant decision IDs. Mark a task complete only when its Definition of Done is satisfied; distinguish implemented scaffolding, development simulation, reviewed policy, provider-account readiness and production activation. Under D-21, also distinguish design-independent technical progress from presentation awaiting owner design input; do not mark gated UI acceptance complete from a build or contract test. Update this canonical file and linked decision records when approved scope changes; do not create competing backend/frontend roadmaps.
