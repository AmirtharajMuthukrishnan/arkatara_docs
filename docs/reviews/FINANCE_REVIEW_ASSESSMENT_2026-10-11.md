# Finance review: finding validation and decision handoff

Date: 2026-10-11 (Asia/Calcutta). Status: ASSESSMENT COMPLETE; OWNER ANSWERS AND QUALIFIED CA/LEGAL REVIEW PENDING. This is review evidence, not a change to canonical business requirements or authorization to implement fixes.

The owner requested analysis first, a combined questionnaire second, and fixes only after answering and finalizing the flow. Application source, models, migrations, tests, branches and commits were not changed in this assessment. The root documentation tracking list was extended only for these review artifacts.

## Source, authority and verification limits

The complete [Claude report](ARKA_TARA_FINANCE_REVIEW_2026-10-10.md) was found in its temporary scratchpad and copied verbatim into this directory. SHA-256: 2D77E40F1BCB95769F1D216D3C170FCBD8AB3205C9808F56F403F0660D2F4012. Its [probe source](FINANCE_REVIEW_PROBE_SOURCE_2026-10-10.txt) is retained verbatim as historical review evidence, not installed as application tests. SHA-256: DDB49F7B5D4BAE438710B9A3B1F4C9E4249AD221A0C4B32DF15E2E00EFD31BFD.

The report identifies its reviewer as Claude and explicitly says it is not CA or legal advice. It does not close CA-01, CA-02, CA-03 or LR-01 through LR-04. If a qualified CA separately approved any conclusion, obtain the actual written advice, scope, effective date and reviewer identity before recording signoff.

Inspection covers the current backend working tree on feature/task-4-models, HEAD 28dacf5, including existing uncommitted changes and forward integrity definitions through trials/0011. This assessment independently inspected requirements, model fields, snapshot contracts, relevant guard bodies and the retained probes. It did not start a database or rerun the probes/tests. The 86 passing finance tests, fresh migration result and 18 reproduced test cases across 17 probes remain reviewer-reported results. The previous 472-test run remains the separate evidence recorded in the [earlier correction checkpoint](TASK4_REREVIEW_CORRECTIONS_2026-10-10.md).

Authority: [BUSINESS_CONTEXT](../BUSINESS_CONTEXT.md), [BUSINESS_RULES](../BUSINESS_RULES.md), [ARCHITECTURE](../ARCHITECTURE.md), [DATA_MODEL](../DATA_MODEL.md), [DECISIONS](../DECISIONS.md), [STATE_MACHINES](../STATE_MACHINES.md), [COMPLIANCE_AND_FINANCE](../COMPLIANCE_AND_FINANCE.md) and [TASKS](../TASKS.md). D-26 keeps Task 4 at persistence/integrity/migration/model-test scope; D-28 requires evidence-based finding validation; D-21 still gates final UI design.

## Overall assessment

The report identifies real integrity defects and real unfinished finance persistence. It also overstates several conclusions and presents some policy-dependent changes as independent fixes. Refunds/settlements are explicitly pending Task 4 work; different purchase, invoice and cash totals do not inherently establish an accounting defect. No live finance workflow or professional finance readiness is established.

Retain the existing protections: exact source-linked allocations, preserved captures, separate upfront application, immutable issued documents, capture reconciliation, city-specific stock, normal combined visits, physical receipt/QC before availability, guest ownership and explicit unconfigured policies. Correct only the validated gaps after the requested decision stage.

| Finding | Disposition | Main qualification | Required answer/gate |
| --- | --- | --- | --- |
| FIN-01 | VALID | Numeric adjustments are not represented in legal sale/charge components. | BUSINESS DECISION BD-04/BD-05; CA REVIEW CA-01/CA-02; LR-01 for charges |
| FIN-02 | PARTIAL | Tax JSON/contracts exist; persistence enforcement is incomplete. Exclusive price calculation is not intrinsically impossible. | CA REVIEW CA-02; BUSINESS DECISION BD-04 |
| FIN-03 | PARTIAL | Issuance particulars/issuer eligibility are weak; requirements depend on document type and applicable seller regime. | CA REVIEW CA-02/CA-03; LEGAL REVIEW LR-02/LR-03 |
| FIN-04 | VALID | Jurisdiction and effective registration history are unfinished. | BUSINESS DECISION BD-11; CA REVIEW CA-02/CA-03 |
| FIN-05 | PARTIAL | Missing refund/settlement lifecycle is real; successful collection must remain an immutable historical fact. | BUSINESS DECISION BD-01/BD-02/BD-10; CA REVIEW CA-01/CA-02 |
| FIN-06 | PARTIAL | Invoice coverage and legal acceptance timing are missing; payment is not the universal invoice trigger. | BUSINESS DECISION BD-05; CA REVIEW CA-02; LEGAL REVIEW LR-03 |
| FIN-07 | PARTIAL | Duplicate invoice coverage and numbering gaps are real; credit notes do not automatically allow reinvoicing the same sale. | CA REVIEW CA-02 |
| FIN-08 | PARTIAL | Correction direction, provenance, chronology and credit budgets are missing; do not cap debit notes as credits or prohibit all late commercial credits. | BUSINESS DECISION BD-10; CA REVIEW CA-02 |
| FIN-09 | PARTIAL | Impossible chronology is a defect; close/reopen and late-recording rules need a reviewed policy. | CA REVIEW CA-02/CA-03; owner operating controls |
| FIN-10 | VALID | A sealed purchase can use unconfigured tax and its invoice can use inconsistent treatment. | CA REVIEW CA-02 supplies actual treatment; integrity principle already approved |
| FIN-11 | PARTIAL | Neutral upfront record lacks defined semantics; evidence requirements differ for receipt, quote, fee invoice and refund voucher. | BUSINESS DECISION BD-01; CA REVIEW CA-01/CA-02 |
| FIN-12 | VALID | Merchant reference lacks reliable receiving-entity/account attribution; a different payee/seller needs explicit reviewed arrangements. | BUSINESS DECISION BD-11; CA REVIEW CA-03; LEGAL REVIEW LR-02 |
| FIN-13 | PARTIAL | TEST captures are valid in development; production isolation is a future enforced boundary, not a global prohibition of test records. | Task 8/10 environment readiness |
| FIN-14 | PARTIAL | Cost/ownership/acquisition persistence is missing; blanket hallmark assertions need qualification. | BUSINESS DECISION BD-09/BD-11/BD-12; CA REVIEW CA-03; LEGAL REVIEW LR-03 |
| FIN-15 | PARTIAL | Current PAN/declaration rule has official support; absence is not only a Gold issue, and applicability/exemptions must be reviewed. | CA REVIEW CA-02; LEGAL REVIEW LR-04 for identity data |
| FIN-16 | PARTIAL | Controlled retention/erasure is unfinished; immutable history is intentional and cannot simply be relaxed. | LEGAL REVIEW LR-04; CA REVIEW CA-02/CA-03 |
| FIN-17 | PARTIAL | Reporting/export definitions are pending; cash, invoices and recognized revenue are distinct facts, and no new general ledger is automatically required. | CA REVIEW CA-01/CA-02/CA-03; Task 10 metrics |

## FIN-01: adjustments absent from corresponding sale/charge documents

Evidence: [signed adjustment](../../arkatara_backend/billing/models.py:118), [balance arithmetic](../../arkatara_backend/billing/models.py:143), [seal identity](../../arkatara_backend/trials/migrations/0010_purchase_balance_integrity.py:12), [invoice line equality](../../arkatara_backend/trials/migrations/0007_postcapture_integrity.py:219) and [invoice total equality](../../arkatara_backend/trials/migrations/0007_postcapture_integrity.py:256). R-19/R-25/R-27/R-28 require authoritative pricing and preserved selling/document facts.

Root cause: an attributed signed number changes payable consideration without a matching typed discount/charge and appropriate documentary/tax semantics.

Smallest correction: when authorized, fail closed on nonzero adjustments at executable financial sealing until their approved calculation/document treatment exists. Then represent permitted central discounts or charges explicitly, with correct net/tax/gross arithmetic. A separate service may need a separate document. This is a capability gate, not an invented permanent no-discount business rule. Agents never gain manual price override permission.

Regression: reject positive/negative unapproved adjustments via ORM and SQL; reconcile approved components with documents, payment and upfront application; test zero final balance. Existing positive-adjustment expectations in [test_upfront_application](../../arkatara_backend/tests/test_upfront_application.py:122) need deliberate revision.

Consequences: purchase pricing, billing, upfront/refund budgets, analytics and frontend amount contracts. Owner Q28; CA Q16 and Q1-Q5.

## FIN-02: persisted tax semantics are incomplete

Evidence: [HistoricalPriceSnapshot tax contract](../../arkatara_backend/billing/snapshots.py:56) already records components and exact decimal inputs. [TaxEvidence](../../arkatara_backend/billing/models.py:15), purchase-item price snapshots and financial-document line snapshots are not bound to that full contract. [financial_snapshot_objects](../../arkatara_backend/trials/migrations/0006_lifecycle_financial_integrity.py:257) checks object shape/non-emptiness, not fiscal arithmetic.

Root cause: a useful value contract does not enforce transaction-level classification, tax components, effective policy or net/tax/gross/round-off consistency.

Qualification: P02 fixes a purchase at one amount, then attempts a different invoice amount. It establishes the equality constraint, not the impossibility of deriving a gross purchase amount from an exclusive base price. JSON can represent tax; arbitrary unvalidated JSON cannot establish correctness.

Smallest correction: typed monetary/classification/component facts for calculations and reporting, plus retained versioned calculation evidence. Reuse the existing contract where appropriate; avoid duplicate snapshot hierarchies. Require reviewed effective treatment without inventing rates or rounding.

Regression: malformed/arbitrary reviewed evidence, missing components, inconsistent tax/gross sums, both inclusive/exclusive calculation inputs, rounding boundaries and historical policy changes.

Consequences: catalogue pricing, authoritative purchase seal, taxable upfront collections if applicable, invoice/correction lines, tax reporting and API amount breakdown. Owner Q26-Q28; CA Q6-Q9.

## FIN-03: document particulars and issuer applicability

Evidence: [LegalEntity](../../arkatara_backend/compliance/models.py:27) permits a DRAFT reference and optional registration string. [issue binding](../../arkatara_backend/trials/migrations/0007_postcapture_integrity.py:225) checks non-empty snapshots, identity, reviewed evidence and totals, not full legally applicable particulars.

Root cause: generic evidence gates do not yet establish that a specific document kind is legally complete or that its issuer has the applicable status/registration on its issue date.

Smallest correction: per-kind required-particular contracts and applicable effective issuer/registration validation. Preserve document identity and rendered evidence when generation is implemented. Evaluate historical eligibility at the legal issue date, not by today's active flag.

Do not impose positive challan value, signature evidence or GST registration on every possible document/seller indiscriminately. The CA/legal reviewer must specify applicable fields, exemptions and valuation. A model named TAX_INVOICE is not professional confirmation of compliance.

Regression: missing particulars, ineffective issuer/registration, conditional particulars/exemptions, manifest/value agreement and preserved historical issuer snapshots after prospective changes.

Consequences: dispatch, tax/receipt documents, legal entities and future document rendering/notifications. Owner Q19-Q21; CA Q10-Q14/Q23; Legal L2-L3.

## FIN-04: supplying and recipient jurisdiction/registration

Evidence: [Hub](../../arkatara_backend/markets/models.py:27), [booking contact/address](../../arkatara_backend/trials/models.py:50) and [single registration string](../../arkatara_backend/compliance/models.py:39) lack reliable state/effective registration persistence.

Root cause: reference identity foundations have not been completed with the registration/location/supply facts required by CA-02/CA-03.

Smallest correction: effective entity registration identity, supplying location/state evidence, recipient/delivery jurisdiction snapshots and reviewed place-of-supply treatment. PIN data assists validation but is not an unconditional legal determination. Preserve controlled prospective entity changes and historical snapshots rather than freezing every reference forever.

Regression: reviewed same/interstate cases, wrong registration/state, effective-date overlap/expiry boundaries and historical records unaffected by later updates.

Consequences: markets/hubs/service areas, seller/provider attribution, billing, stock transfers and future Hyderabad. Owner Q19-Q21; CA Q21-Q23; Legal L2-L3. No implicit intercity inventory fulfillment.

## FIN-05: refunds, retention and settlement persistence

Evidence: [BookingCharge](../../arkatara_backend/payments/models.py:53), [PaymentContext](../../arkatara_backend/payments/models.py:145) and [gateway kinds](../../arkatara_backend/payments/models.py:369). Refunds, settlements and bank matching are explicitly pending in [coverage matrix](../DATA_MODEL.md:21) and Task 4.M9.

Root cause: receipt/application evidence is implemented, but separate repayment, retained-money, chargeback/reversal, fee/settlement and bank evidence remains unfinished.

Qualification: COLLECTED and PAID should remain historical successes. Refunding does not mean voiding the original capture. A deposit-liability equation is conditional on actual classification and must include reviewed reversal/correction effects; it is not a universal revenue rule for every upfront fee. Refund rules must also support valid after-sale refunds of previously applied funds through explicit reversal/correction provenance.

Smallest correction: finish coherent refund/disposition/settlement/bank-match persistence with shared source budgets and separate immutable events. Preserve gross money, fees/taxes and net bank settlements. Add provider-specific adapters/workflows at their later task.

Regression: repeated/concurrent/in-flight refunds, over-refund, cross-entity/currency, failed/refund reversal, partial retention/application, chargebacks, duplicate provider facts, partial/aggregated settlements and unmatched bank receipts.

Consequences: all collection/application/refund budgets, cancellations, no-purchase visits, after-sale cases and CA exports. Owner Q22-Q25/Q33-Q35/Q38; CA Q1-Q5/Q18-Q20. No live money before required lifecycle readiness.

## FIN-06: supply acceptance, payment and invoice/handover timing

Evidence: [SALE custody guard](../../arkatara_backend/trials/migrations/0008_visit_corrections.py:202) requires paid confirmed purchase but not issued invoice coverage. [invoice guard](../../arkatara_backend/trials/migrations/0007_postcapture_integrity.py:256) requires CONFIRMED purchase.

Root cause: payment-backed purchase confirmation currently supplies the sale-state boundary; legal acceptance/issue timing and document coverage have not been independently completed.

Smallest correction: after CA defines the applicable supply/approval event, preserve it independently from payment and guard completed handover with required issued document coverage. Use atomic issue/handover where applicable. Track removal/return/supply dates needed by the reviewed approval regime. Do not gate every tax issue solely on cash receipt.

Regression: paid piece without required invoice cannot complete handover; wrong invoice item coverage; acceptance/issue/payment ordering; paid and deferred-payment approved cases if enabled; no-purchase trial return does not become a sale; stale out-on-approval exceptions remain visible.

Consequences: trial/visit, purchase acceptance, stock custody, invoice issuance and pending-payment policy. Owner Q29/Q36-Q39; CA Q10-Q13; Legal L3. No arbitrary six-month runtime constant before applicability is reviewed.

## FIN-07: duplicate sale invoice coverage and numbering

Evidence: [DocumentSeries](../../arkatara_backend/billing/models.py:342), [series/number uniqueness](../../arkatara_backend/billing/models.py:398), [document/item uniqueness](../../arkatara_backend/billing/models.py:447) and [issue check](../../arkatara_backend/trials/migrations/0007_postcapture_integrity.py:247).

Root cause: within-document identity is protected, but the same sale item can be invoiced again in another issued document; numbering lacks reviewed issuance allocation/scope.

Smallest correction: protect initial invoice coverage per actual sale line, allocate numbers transactionally in reviewed registration/type/period scope and retain cancellations/gaps. Do not add lifetime one-invoice-per-booking or one-sale-ever-per-physical-unit constraints.

A credit note preserves the original invoice and does not automatically authorize reinvoicing the same sale. Actual return/resale is a new sale. Rule 46 permits one or multiple series; the report's proposed single series is a design choice, not the exact statutory mandate.

Regression: concurrent duplicate coverage, number races/retry/rollback, wrong type/year/registration, legal format/length, preserved cancelled numbers, credits not reopening old-sale coverage, legitimate separate sale or split coverage.

Consequences: purchase/invoice coverage, returns/resales, series, exports and invoice notifications. Owner Q40; CA Q14-Q15.

## FIN-08: correction direction, original-line provenance and budgets

Evidence: [generic ADJUSTMENT and corrects reference](../../arkatara_backend/billing/models.py:373), [header correction check](../../arkatara_backend/trials/migrations/0007_postcapture_integrity.py:240) and missing corresponding adjustment-line provenance in the line branch at 210.

Root cause: header identity is preserved without explicit credit/debit effect, corrected-original-line identity, cumulative credit budget or chronological relation.

Smallest correction: distinct financial direction and corrected document/line links, attributable reason/approval, chronology and locked remaining-credit budgets. Include valid prior adjustments. Debit-note increases are not simply capped like refunds. Separate commercial credit/refund eligibility from statutory tax-reduction eligibility.

Regression: cross-document/booking items, earlier-than-original dates, concurrent excessive credits, prior debit/credit balances, partial/full returns, duplicate correction and late commercial credit without an unauthorized tax reduction.

Consequences: returns, refunds, invoice history, tax periods and accounting export. Owner Q31-Q35; CA Q15-Q17/Q19.

## FIN-09: chronology and accounting/return-period control

Evidence: [observed time validation](../../arkatara_backend/common/verification.py:18) bounds future observations; [payment allocation guard](../../arkatara_backend/trials/migrations/0007_postcapture_integrity.py:341) lacks receipt-to-allocation lower bound; issue/correction guard lacks corresponding original/supply lower bounds.

Root cause: individual timestamps are checked without business-event chronology or reviewed period-close authority.

Smallest correction: explicit occurrence/recording/verification/posting distinctions, attributable close/reopen evidence and guard genuinely impossible ordering. A receipt may arrive late while retaining its true occurrence date. Do not use a blanket small backdating window to reject legitimate imports/provider delays.

Regression: allocation before qualifying capture, correction before original, invoice before applicable acceptance, closed-period writes and authorized reopen, late-arriving genuine evidence and immutable recording/audit facts.

Consequences: payment imports, invoice/correction timing, CA reporting and settlement reconciliation. Owner Q35; CA Q27.

## FIN-10: reviewed tax and sealed purchase/invoice consistency

Evidence: [purchase tax reference](../../arkatara_backend/billing/models.py:66), [purchase seal](../../arkatara_backend/trials/migrations/0007_postcapture_integrity.py:177), [balance seal](../../arkatara_backend/trials/migrations/0010_purchase_balance_integrity.py:29) and separate invoice tax check at trials/0007:248.

Root cause: UNCONFIGURED is valid in drafts but can reach authoritative financial sealing; invoice evidence has independent and potentially contradictory provenance.

Smallest correction: financially sealed calculation requires valid reviewed/effective treatment; invoice facts must agree with the sealed purchase facts. Keep unconfigured drafts useful. Equality of one FK may be sufficient in a simple design, but explicit later corrections/equivalent facts must not be rejected blindly.

Regression: unconfigured or inapplicable tax cannot seal; conflicting document facts rejected; approved consistent facts, prospective tax change and fully upfront-funded zero-balance path.

Consequences: purchase, obligations, payment amounts, invoice snapshots and calculation policy. CA Q6-Q9 supplies treatment; no owner choice can waive consistency.

## FIN-11: generic upfront document semantics

Evidence: [UPFRONT_RECORD kind](../../arkatara_backend/billing/models.py:374) and [issue amount/currency check](../../arkatara_backend/trials/migrations/0007_postcapture_integrity.py:253) do not distinguish receipt, service invoice or quotation.

Root cause: policy-neutral placeholder lacks the preconditions/provenance of a particular statutory or commercial document.

Smallest correction: after CA-01 classification, enable precise document kinds and reviewed rules. Receipts prove qualifying money received; refund acknowledgments prove the appropriate refund state; service tax invoices follow applicable supply/issue rules, not a universal captured-money prerequisite. Migrate neutrally preserved legacy evidence forward without inventing its legal meaning.

Regression: no receipt for merely quoted/authorized money, exact source/entity/currency/partial amount, duplicate receipts, fee invoice timing and unknown classification gate.

Consequences: booking charge, refund, billing and notification. Owner Q22-Q25; CA Q1-Q5.

## FIN-12: receiving entity and merchant/bank account identity

Evidence: [provider/account strings](../../arkatara_backend/payments/models.py:219), [receipt account fields](../../arkatara_backend/payments/models.py:365), [seller](../../arkatara_backend/billing/models.py:63) and the existing string-based matching/allocation guards.

Root cause: technical merchant identity does not prove the effective legal receiving party or settlement account.

Smallest correction: effective provider-account identity linked to receiving entity and reviewed settlement bank identity; preserve source attribution. Validate the initial single-entity arrangement if selected. Any collect-on-behalf/inter-entity arrangement requires its own reviewed agreements/evidence rather than a universal assumption or unrecorded exception.

Regression: wrong payee/provider account/entity/environment, effective-account changes, unmatched receipts and explicit approved agency/transfer scenarios if introduced.

Consequences: upfront/final receipts, seller invoices, bank settlement, entity books and future company transitions. Owner Q19-Q20; CA Q21/Q23; Legal L2.

## FIN-13: development/test versus production money

Evidence: TEST/LIVE environment fields in [attempt](../../arkatara_backend/payments/models.py:221) and [receipt](../../arkatara_backend/payments/models.py:367) are compared for identity but not bound to a trusted deployment policy. Current development fixtures intentionally use TEST.

Root cause: provider evidence environment is recorded but future live runtime enforcement is not complete.

Smallest correction: authoritative deployment/provider environment binding and fail-closed collection/allocation/confirmation gates, with isolated development evidence. Enforce at the boundary claimed by the implementation, including SQL if database enforcement is claimed. Never globally change all fixtures to LIVE or trust a caller-supplied environment label.

Regression: production rejects TEST/misconfigured accounts; local/test accepts authorized TEST evidence; no cross-environment matching/allocation; missing configuration fails closed; genuine late events retain the correct identity.

Consequences: adapter credentials, deployment/configuration, payments and test fixtures. Tasks 8/10; no new commercial approval needed to preserve environment isolation.

## FIN-14: acquisition, ownership, cost, weight and purity evidence

Evidence: [InventoryUnit](../../arkatara_backend/inventory/models.py:35) has physical identity/custody fields without acquisition, cost or legal owner evidence. Full exception/after-sale scope remains pending Task 4.M5.

Root cause: physical movement foundations do not yet contain all reviewed valuation/supplier/ownership facts.

Smallest correction: coherent acquisition/supplier-document/owner/cost evidence and applicable per-piece weights/purity/hallmark references, plus attributable write-off/disposition value. Represent consignment or own manufacture only if selected; no procurement ERP is needed.

Silver HUID is voluntary under the cited BIS scheme; advertising 925 does not by itself establish a universal HUID requirement. Gold obligations have applicability and exemptions to verify. Preserve actual evidence and make only supported product claims.

Regression: missing required acquisition provenance, prospective owner transfer preserving old evidence, owned/consigned valuation treatment, weight precision, applicable hallmark validation, duplicate/invalid piece evidence and authorized loss/QC dispositions.

Consequences: inventory, supplier input-credit evidence, legal entity, pricing, margin/COGS and later Gold. Owner Q20/Q32/Q36/Q41; CA Q20; Legal L2-L3.

## FIN-15: high-value identity requirements

Evidence: [customer snapshot](../../arkatara_backend/billing/models.py:65) is free-form; issue guards do not enforce applicable PAN/declaration evidence. Retained P13 issues a fictional INR 250,000 sale with only probe metadata.

Root cause: the future transaction-level identity/compliance gate and restricted evidence storage are absent.

The official Income-tax Rules 2026 Rule 159 table entry 16 supports the report's more-than-INR-200,000 goods/services threshold and permitted Form 97 declaration, subject to the rule's applicability/exemptions. This requirement can affect Silver too. It should not wait merely because Gold has not launched. Format validation alone does not establish all required verification/reporting obligations.

Smallest correction: after qualified applicability review, effective configurable thresholds, restricted PAN/declaration evidence and seal/issue checks or a reviewed launch transaction cap. Keep threshold obligations distinct from cash restrictions, PMLA and full KYC; do not collect unnecessary identification for every guest.

Regression: below/equal/above reviewed threshold, exemptions, incomplete declaration/PAN evidence, restricted access/logging, digital and future cash distinction, grouping/anti-splitting rules if applicable and both materials.

Consequences: guest/account booking contact contracts, billing, privacy and later payment methods. Owner Q30/Q42; CA Q24-Q25; Legal L4.

## FIN-16: retention and controlled personal-data erasure

Evidence: [immutable booking identity/snapshots](../../arkatara_backend/trials/migrations/0004_checkout_integrity.py:101), R-35 and LR-04. Generic destructive edits are deliberately blocked; no legally reviewed retention/erasure mechanism exists.

Root cause: history protection is implemented ahead of record-class retention, restricted evidence separation and authorized erasure.

Smallest correction after advice: retention basis/start event/holds per record class, separate operational contact data from statutory document copies where appropriate, and guarded attributable erasure/anonymization. Restrict access after purpose expiry even while lawful records remain retained. Keep statutory history and permanent guest/account ownership reliable.

A single longest number of years after financial year-end cannot encode all relevant periods or starts. DPDP commencement is phased; counsel must address retained processing/security records as well as purpose-limited operational data before any complete-deletion promise.

Regression: early/unauthorized erasure, retained document preservation, due-date/hold calculation, guest/account closure separation, controlled expired-draft treatment and no continued disclosure through caches/exports.

Consequences: booking/customer models, documents, provider logs, backups, support access and reporting. Owner Q5-Q7/Q14-Q15; CA Q26; Legal L4.

## FIN-17: financial definitions, reconciliation and exports

Evidence: pending [Task 4.M11 analytics coverage](../DATA_MODEL.md:23), existing distinct Purchase/FinancialDocument/GatewayTransaction facts and CA-01/CA-03.

Root cause: reporting definitions and controlled accounting export mapping are not complete.

Qualification: captured cash, invoiced amounts, GMV and recognized revenue can legitimately differ. Invoice issue date is not a universal revenue recognition rule. Fix inconsistent source records, then define timing and adjustments with the CA. Canonical requirements do not require building a new accounting ledger inside Django.

Smallest correction: metric dictionary, authoritative source/date/provenance and CA-approved export mapping, with reconciliation of gross receipts, upfront balances, refunds/fees and bank settlements. Query-only projections are acceptable; preserve minimized analytics evidence when required.

Regression: no invoice/event double-counting, upfront money excluded from jewellery revenue until its reviewed treatment applies, credits/refunds/chargebacks by reviewed basis, partial/aggregated settlements, date-boundary reconciliation and true margin costs. Do not treat phone counts as proven unique people.

Consequences: billing/payments/inventory/analytics, accounting tool and investor reporting. Owner Q34-Q35; CA Q18-Q20/Q26-Q28.

## Research checks and limits

- [CBIC Rule 46](https://taxinformation.cbic.gov.in/content-page/explore-rules/1000136/1000001): tax-invoice particulars and one or multiple financial-year-unique series with the applicable character/length restriction. Exact Arka Tara numbering remains CA review.
- [Official movement-on-approval circular index](https://gstcouncil.gov.in/node/4721): approval and movement treatment must be reviewed independently of payment.
- [Official cancellation/compensation circular](https://www.gstcouncil.gov.in/sites/default/files/2024-06/cir-178-08-2022-cgst.pdf): scenario-specific treatment; no blanket conclusion that every retained amount or damage payment has one tax result.
- [Official Rule 159](https://www.incometaxindia.gov.in/documents/20117/42998/Rule-159_2026-03-25_06-11-40_0d46f6_en.pdf/afebd591-1b53-4913-0d57-dc518fa4a5aa?t=1774449122128&version=2.0) and [notified 2026 Rules](https://www.incometax.gov.in/iec/foportal/sites/default/files/2026-03/En-Notified-IT-Rules-2026-20-03-2026.pdf): current rule/form verified; CA must specify applicability, verification, reporting and exceptions.
- [BIS October 2025 newsletter](https://www.bis.gov.in/wp-content/uploads/2025/10/BIS-NEWSLETTER-OCTOBER-2025.pdf) and [jeweller FAQs](https://www.bis.gov.in/hallmarking-jewellers/?lang=en): voluntary Silver HUID scheme; product/invoice obligations need material/article-specific review.
- [Official DPDP Rules](https://www.meity.gov.in/static/uploads/2025/11/53450e6e5dc0bfa85ebd78686cadad39.pdf) and [commencement](https://www.meity.gov.in/static/uploads/2025/11/c56ceae6c383460ca69577428d36828b.pdf): phased duties and record-class retention/privacy review.
- [CCPA price-disclosure enforcement](https://www.pib.gov.in/PressReleasePage.aspx?PRID=2171807&lang=2&reg=3): tax-inclusive price claims cannot be followed by surprise GST at checkout.
- [Government announcement, 10 September 2026](https://www.pib.gov.in/newsite/erelcontent.aspx?lang=2&reg=48&relid=294532): announces e-commerce amendments effective 1 January 2027. Obtain the notified text/applicability assessment for promotional price history, grievances and other affected obligations; this announcement does not itself close LR-04.

Exact current product GST/HSN/SAC applicability, state-specific Rule 138F notifications, seller registration eligibility, PMLA applicability, correction/filing practices and legal retention matrix remain UNRESOLVED. Do not implement the report's secondary-source rates, deadlines or exemptions as approved constants.

## Decision and implementation handoff

Use the [combined questionnaire](OWNER_AND_REVIEWER_QUESTIONS_2026-10-11.md): previous owner Q1-Q18 retained, additional owner Q19-Q43, revised CA Q1-Q27 retaining original report IDs, additional CA Q28 on accounting recognition, and Legal L1-L6. Recommendations are proposals, not approvals.

After answers: check compatibility, record explicit approved changes with superseded history, preserve unanswered BD/CA/LR gates, and map authorized corrections to existing Task 4.M1-M12 and later workflow tasks. No task renumbering, new Task 11, UI design or microservice expansion.

Candidate correction order, subject to answers: purchase/document tax consistency and typed components; invoice/correction coverage and chronology; effective entity/provider/jurisdiction facts; refunds/source budgets/settlements; acquisition/valuation; restricted high-value/retention evidence; reporting/export definitions. Include ORM/raw-SQL/concurrency and fresh/upgrade/rollback checks appropriate to the claimed guards.

Migration changes must be additive. Preflight existing sealed/issued/applied evidence rather than changing old rows to fit new guards or rewriting applied migrations. Document recovery and deployment implications. Independent Claude re-review and exact-commit CI remain separate evidence gates.

Search/UI impact: this assessment introduces only review documentation. Financial, identity and provider evidence belongs on protected/non-indexable surfaces; future price metadata must reflect reviewed visible totals. D-20/D-21 remain binding.
