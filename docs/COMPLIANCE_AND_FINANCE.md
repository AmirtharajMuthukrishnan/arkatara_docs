# Compliance and finance

Status: Software requirements and proposed evidence design for owner review; no compliance certification\
Updated: 2026-10-04

This document defines what the software must preserve and support. It does not decide the business's tax position or certify legal compliance. Open questions retain the BUSINESS DECISION, CA REVIEW and LEGAL REVIEW labels in [DECISIONS.md](DECISIONS.md).

## Ownership and readiness

Before Task 8 live collection, establish the final operating/selling identity, business current account, applicable GST registration, CA-confirmed accounting policy and Trial Deposit treatment, Razorpay business account/KYC and settlement bank account. These are roadmap prerequisites, not claims that registration already exists or is always required in a particular form.

| Matter | Classification and owner |
| --- | --- |
| Initial entity, inventory ownership plan and transition timing | BUSINESS DECISION BD-11 |
| Legal seller, contracting party and effective ownership/transition arrangements | LEGAL REVIEW LR-02 |
| Entity-specific books, registrations, invoices and transfers | CA REVIEW CA-03 |
| Deposit commercial terms | BUSINESS DECISION BD-01 |
| Deposit recognition, tax and accounting | CA REVIEW CA-01 |
| GST, HSN, invoice/challan requirements, numbering scope, refunds and corrections | CA REVIEW CA-02 |
| Movement/product documentation and responsibilities | LEGAL REVIEW LR-03 |
| Customer/agent liability and commercial policy constraints | LEGAL REVIEW LR-01 |
| Privacy, policies, grievance information, consent and retention | LEGAL REVIEW LR-04 |

Record reviewed outcomes and evidence before dependent production behaviour is enabled. Independent business modelling can proceed without pretending these answers are known.

## Required distinctions

- TrialBooking records a trial commitment; it is not automatically revenue, a Purchase, a payment or an invoice.
- Physical dispatch records custody and movement, not a completed sale.
- Purchase/PurchaseItem records customer selection and the applicable price evidence; tentative selection must not masquerade as a paid sale.
- BookingDeposit records a separate purpose for money collected. Collection, application, retention and refund are distinct facts.
- PaymentAttempt is the intended collection context; GatewayTransaction is provider evidence. A request or screenshot is not proof of receipt.
- Gateway settlement and bank receipt are later reconciliation facts, not synonyms for the customer's successful payment.
- A refund, a commercial return, a stock receipt and a financial adjustment document are different facts. One does not automatically prove the others happened.

Revenue recognition, deposit classification and exact issue dates are CA REVIEW questions. The application must not select them merely by choosing an enum name.

## GST and tax configuration

The proposed structure should retain tax classification/HSN, applicable registration and jurisdiction facts, effective dates, rate components, taxable value, tax amounts, currency and calculation/rounding evidence as appropriate to the reviewed treatment.

Tax lookup may use current configuration when establishing a new transaction. Historical invoices must use their own snapshots, never current Product, TaxCode or LegalEntity values.

Do not scatter a GST percentage through code or assume every material, charge or deposit has the same treatment. Do not infer tax-inclusive/exclusive wording or weight-based valuation from the initial Silver pilot.

CA REVIEW CA-02 must settle classifications, rates, applicability, place-of-supply treatment, rounding, taxable components, price-adjustment treatment, registrations, required customer details, and whether additional processes such as e-invoicing or movement reporting apply. No threshold, rate or exemption is selected here.

CBIC's published invoice/movement rules distinguish invoice particulars and documentation for specified non-invoice movements. They are a reference for review, not a conclusion that one document is sufficient for this business. The linked page contains older rule presentation; a CA/legal reviewer must verify applicable amended rules before production. [CBIC invoice and movement-document reference](https://cbic-gst.gov.in/gst-invoice-rules.html).

## Historical entity and price snapshots

At the relevant commitment/document event, preserve enough facts to explain:

- The issuer/seller identity, registration identifiers where applicable, address and relevant legal-entity reference.
- The customer/delivery/recipient details required by the reviewed document rules, without depending on a mutable current customer profile.
- Product description, SKU/design and variant, material, purity, quantity, relevant unit/weight facts and physical-item linkage where appropriate.
- Pricing mode, approved calculation inputs/components, agreed values, permitted adjustments, tax classification/rates/components, totals and currency.
- Applicable policy/version, event timing, authoritative source references and the relationship to prior funds.

For WEIGHT_BASED pricing, preserve the actual inputs and result used; do not silently choose a formula, commodity rate source, making charge, wastage or price-lock time. Those remain BD-04/BD-12 and CA-02.

Entity or catalogue edits after issue must not alter prior financial records. A future company cannot simply become the issuer of documents previously issued by another entity.

## Invoice issue, numbering and correction

Software requirements:

1. Distinguish a working purchase/calculation or document draft from an issued TaxInvoice.
2. Allocate controlled numbers at the approved issuance boundary with concurrency protection. Preserve number, series, issuing entity and reviewed period/scope.
3. Prevent duplicate issued identifiers within that approved scope. Do not assume the correct series scope merely from current single-city operation.
4. Freeze issued invoice header, line snapshots and line membership: no adding/removing/reassigning lines after issue. Preserve currency, issuer and recipient facts independently of mutable references. A stored rendered document is useful evidence, but does not replace the underlying frozen values.
5. Retain original issued records. Corrections use linked, authorized adjustment/correction evidence appropriate to the reviewed rules; never overwrite an old invoice to make totals match.
6. Record actor, reason, timing and relationship to the original for issue, cancellation/void where permitted, adjustment and refund activity.

CA REVIEW CA-02 determines numbering format/scope, legally relevant issue timing, treatment of abandoned numbers and permitted correction documents. Immutability is finalized; the exact corrective accounting workflow is not.

## Trial Deposit

D-25 requires upfront collection before booking confirmation for the launch flow, after booking phone verification. BD-01/BD-02 are partially resolved only to that extent. Amount, basis, deposit/fee classification, application/refund/retention and expiry/late-payment outcomes remain open. Commercial BD-01, accounting/tax CA-01 and legal LR-01 decisions remain separate; required collection does not imply any particular refund, credit or revenue treatment.

Illustration of the approved distinction only:

| Fact | Amount |
| --- | ---: |
| Jewellery invoice total | INR 5,000 |
| Previously collected deposit applied to that purchase | INR 49 |
| Remaining amount due in this example | INR 4,951 |

The jewellery is not silently sold for INR 4,951. This example neither chooses INR 49 as a fee nor establishes that every deposit is applied to every purchase. No-purchase, cancellation, no-show, partial purchase and deposit-exceeding-purchase outcomes remain open.

Proposed evidence must connect collection, later application, any refund/retention and the relevant booking/purchase. Prevent the same funds from being applied or refunded twice. Amounts available for further application/refund must be explainable from authoritative history.

## Dispatch documentation

Support Delivery Challan or other reviewed dispatch documentation separately from sale invoicing.

The proposed record should identify document reference/date, responsible entity, selected market, originating hub(s) where allowed, booking/assignment, recipient/address, physical items/quantities, condition/custody evidence and any values required by the reviewed format.

Retain dispatch and return evidence even if there is no purchase. Dispatch document contents must not be regenerated from an altered packing list or catalogue without preserving the original version.

CA REVIEW CA-02 and LEGAL REVIEW LR-03 determine the production format, required fields, issue timing, copy/transport requirements, applicability and any additional movement/product obligations. This document does not conclude whether multiple same-city hubs can fulfil one visit; BD-08 remains open.

## Payment confirmation and traceability

Implemented persistence, 2026-10-04 (D-27): BookingCharge preserves the quoted/accepted amount, explicit currency, payee snapshot and policy reference/snapshot. PaymentContext/Attempt preserve intended collection; GatewayTransaction records immutable authorization/capture facts; PaymentAllocation applies captured money under locked receipt/obligation limits. A capture is not revenue recognition, a jewellery invoice or bank settlement. The term BookingCharge deliberately leaves deposit/fee classification open. Refund, sale application, settlement, bank matching and invoice persistence still need the remaining Task 4 scope.

Late, duplicate and extra money must be distinguished: duplicate event/transaction identity is rejected; a distinct real receipt is retained, including after booking expiry. It cannot restore released inventory or allocate to a closed context. Commercial refund/reconciliation treatment remains BD-01/BD-02 and CA/LR review. These model tests use fictional provider facts, not live verified financial events.

The backend must calculate amounts and bind collection to its booking/purchase purpose, expected amount, currency, entity/provider account and an identifiable payment context.

Agent QR and customer URL access that same final PaymentAttempt. Confirm provider identity, amount, currency and context before applying verified payment evidence. Duplicate, delayed, out-of-order or conflicting notifications must not create a second purchase, stock sale, invoice, deposit application or refund.

A transport timeout is uncertain evidence, not proof that no collection happened. Resolve provider state before a retry can create duplicate collection. Do not let a stale link for a previous selection silently pay a new amount. BD-05/BD-15 govern the commercial retry/selection behaviour and product flow.

Store the gateway references needed for verification and reconciliation, with access and retention controls. Never store sensitive card data. Razorpay's security guidance calls for protected secrets, backend payment evidence and authenticated webhooks; the exact product integration must be checked in Task 8. [Razorpay security checklist](https://security.razorpay.com/security/checklist/).

## Refunds and adjustments

Keep requested, approved, submitted, pending and verified refund outcomes distinguishable; concrete lifecycle proposals are in [STATE_MACHINES.md](STATE_MACHINES.md).

A Refund must connect to original successful payment evidence, business reason, amount/currency, provider reference and relevant purchase/deposit/adjustment. Prevent concurrent or repeated refund requests from exceeding the refundable amount.

Refund permission does not establish that an item has returned, passed QC or become sellable. Likewise, receiving an item does not prove that money has been refunded.

BD-01/BD-10 select commercial policies, LR-01 addresses legal constraints, and CA-01/CA-02 determine accounting/document treatment. Never mark a refund paid solely because an agent requested it or a provider accepted an asynchronous request.

## Settlement and bank reconciliation

Plan for a traceable chain:

PaymentAttempt -> verified GatewayTransaction -> gateway settlement entry/batch -> actual bank settlement evidence.

Maintain relationships rather than assuming one payment equals one bank deposit. Preserve gross receipts, gateway fees/taxes where reported, refunds, adjustments, settlement identifiers, expected/actual dates and bank references as separate evidence.

Reconciliation should reveal unmatched payments, settlement entries and bank receipts; amount/currency/entity mismatches; duplicates; timing differences; missing refunds; and provider adjustments. A net bank deposit must not overwrite gross sale or tax history.

Use controlled matching, review status, responsible actor and explanation for resolved discrepancies. Accounting mappings and treatment of fees, disputes or adjustments require CA REVIEW CA-02. Exact imports/provider reports and operational ownership are BD-15 design inputs.

## Audit, access and privacy

D-23 keeps StaffUser and CustomerAccount separate, with explicitly scoped authentication and staff permissions. D-24 keeps guest bookings permanently unlinked to registered accounts, including matching-phone accounts. Customer order history and authorized business reporting are distinct access purposes; old guest transactions remain auditable business records without becoming account history.

Guest checkout still processes personal data. OTP is evidence of phone-channel control at a time, not permanent personal identity, KYC, contractual acceptance or automatic legal liability. Preserve the required requester/recipient/payment evidence and policy acceptance without assuming those parties are identical. Guest consent/audit records must not become an indirect customer-account link. Account closure follows reviewed retention/anonymization, never cascading deletion of financial/stock history; do not invent retention periods.

Capture attributable evidence for price/configuration changes, physical movements, QC, refunds, invoice actions, staff assignments and payment administration. Retain actor/service identity, time, reason, affected record and safe before/after or event details.

Restrict financial and personal records to permitted staff. Assignment restrictions apply at the backend, not just through hidden UI elements. Keep raw OTPs, secrets, card data and unnecessary customer details out of logs/analytics.

Append-only business history requires access controls and tested preservation; an ordinary log table alone is not a claim of tamper-proofness. Backup restoration and audit retrieval must be exercised.

LEGAL REVIEW LR-04 must settle retention, access/deletion requirements and customer-facing policy wording. Preserve required financial evidence while ensuring a reviewed treatment of personal data; do not invent a retention period.

## Review evidence and later verification

Task 4 now owns all agreed persistence, migrations and model/history verification under D-26. Tasks 6–9 own booking/operational/payment/messaging behaviour; Task 10 verifies live readiness. A completed model layer alone cannot be described as financially operational or legally compliant.

Funding reports must reconcile to retained sale, collection, refund, settlement and inventory evidence across guest and account eras. Define each metric and preserve original city/entity/time facts. Do not equate bookings with sales, deposits with revenue or phone numbers with proven unique people. Use aggregate/redacted pitch data; transaction-level diligence needs controlled access and appropriate review, not public exposure of addresses/phones.

Before dependent release, retain review records for applicable GST/registration, deposits, invoice samples and numbering, challan samples, refund/correction treatment, entity attribution, customer terms/privacy and grievance handling.

Task 8 tests include duplicate payments/refunds, altered amounts, invalid signatures, reconciliation mismatches, unchanged historical values after catalogue/tax/entity edits and deposit application without reducing jewellery value. Task 10 includes CA document review, gateway test/live verification, backup restore, inventory reconciliation and operational escalation exercises.

The exact backlog and production readiness checklist remain canonical in [TASKS.md](TASKS.md). None of these checks or account prerequisites has been completed merely by writing this documentation.
