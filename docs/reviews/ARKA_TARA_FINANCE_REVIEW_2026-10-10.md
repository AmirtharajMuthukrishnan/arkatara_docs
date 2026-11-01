# ARKA TARA — Finance, accounting and tax-compliance review

Date: 2026-10-10 · Reviewer: Claude (independent, read-only) · Not a CA or lawyer; nothing here is tax or legal advice.

Scope: the current **uncommitted** working tree of `arkatara_backend` (branch `feature/task-4-models`, on top of `28dacf5`) and the root `docs/` working tree. No repository file was changed.

## 1. Verdict

The system is careful about never losing or double-spending money and stock, but it cannot yet produce correct GST documents or close the books. Today the database would let you hand a customer jewellery without a tax invoice, issue two invoices for one sale, collect an amount different from what the invoice says, date an invoice into the previous financial year, and keep a customer's upfront payment forever with no way to record a refund or a retained amount. It also does not know which state a hub or customer is in, so it cannot choose CGST+SGST versus IGST, and it has nowhere to store a GSTIN per state, an item's cost, its HSN code or a hallmark (HUID). None of this has caused harm yet because no selling, payment or billing workflow is live, but these are gaps in the database design itself, so they must be fixed before Tasks 7–8 build the selling and billing flows on top. There is no CA advice anywhere in the repository, so every tax treatment below is still a question, not an answer.

**Launch readiness (finance/compliance): NOT READY.** **Build the next stage on this? YES, WITH CONDITIONS** — see section 8.

## 2. What I read and ran

- **Read:** `docs/COMPLIANCE_AND_FINANCE.md`, `DECISIONS.md`, `DATA_MODEL.md`, `TASKS.md`, `BUSINESS_RULES.md`, `BUSINESS_CONTEXT.md`, review records of 2026-10-07 and 2026-10-10. `docs/BUSINESS_FACTS.md` (first file in the brief) **does not exist**. A repository-wide search found **no CA advice on file**.
- **Code:** `billing/models.py`, `billing/snapshots.py`, `payments/models.py`, `compliance/models.py`, `inventory/models.py`, `inventory/reservations.py`, `delivery/models.py`, `delivery/visits.py`, `trials/models.py`, `trials/checkout.py`, `common/*`; the effective PL/pgSQL guards were read from a live database with `pg_get_functiondef` (effective sources: `trials/migrations/0006`, `0007`, `0008`, `0010`, `0011`).
- **Disposable database:** a brand-new PostgreSQL 18 cluster created in my scratchpad (127.0.0.1:55491). Fresh `migrate` of the working tree succeeded; `makemigrations --check` reported no changes. Cluster stopped afterwards.
- **Baseline:** the project's own finance-relevant tests (`test_lifecycle_financial_models`, `test_upfront_application`, `test_checkout_models`, `test_review_integrity`, `test_visit_corrections`) — **86 passed in 43 s** on that cluster. I did not run the full suite (Codex reports 472 passed).
- **Probes:** `scratchpad/fin_probes/test_fin_probes.py` — 17 probes (18 test cases). Each probe **passes only if the suspected flaw reproduces**. **All 18 reproduced.** Rejection paths were checked to fail for the intended reason. Probe IDs (P01–P17) are cited per finding. All amounts in probes are tiny fixture values in paise.

## 3. Status of earlier items

There were **no earlier FIN- items** (searched the repository). Earlier CA/LR/BD entries and finance-related review findings:

| ID | Covers | Status now | This review adds |
| --- | --- | --- | --- |
| CA-01 | Deposit recognition and tax | OPEN — no CA advice on file. Upfront charge is correctly kept neutral. | FIN-05, FIN-11 |
| CA-02 | GST, documents, numbering, corrections, refunds | OPEN. Issued documents are sealed (good), but required particulars, tax lines, numbering scope and credit/debit notes cannot be represented. | FIN-01–03, FIN-06–10 |
| CA-03 | Per-entity books and transfers | OPEN. | FIN-04, FIN-12, FIN-14 |
| LR-01 | Custody, liability, charges, retention legality | OPEN. | FIN-01, FIN-05 |
| LR-02 | Legal seller and ownership | OPEN. A DRAFT, unregistered entity can sell and invoice. | FIN-03, FIN-12, FIN-14 |
| LR-03 | Dispatch and product documentation | OPEN. Dispatch is correctly blocked until a reviewed dispatch document exists (V-7). | FIN-03, FIN-06, FIN-14 |
| LR-04 | Privacy, retention, erasure | OPEN. Erasure is currently impossible by design. | FIN-16 |
| BD-01 | Upfront amount, basis, refund/retention | PARTIAL (D-25: collect before confirming). Amount/classification/outcomes open. | FIN-05, FIN-11 |
| BD-02 | Holds, expiry, late payment | PARTIAL (D-27 trigger fixed). Late receipts have no exit. | FIN-05 |
| BD-04 | Price commitment, Gold pricing | OPEN. Adds: tax-inclusive or tax-exclusive prices. | FIN-02 |
| BD-05 / V-5 | Pending payment and handover | OPEN decision. Also decides invoice timing. | FIN-06 |
| BD-06 | Payment methods | OPEN. No cash path exists (good). | FIN-15 |
| BD-09 | Damage, loss, QC disposition | OPEN. No write-off value record. | FIN-14 |
| BD-10 | After-sale returns and refunds | OPEN. | FIN-05, FIN-08 |
| BD-11 | Seller entity and stock ownership | OPEN. | FIN-04, FIN-12, FIN-14 |
| BD-15 | Provider settings, settlement imports, retention ops | OPEN. | FIN-05, FIN-16 |
| F-13 (2026-10-04) | No tax fields | Still PARTIAL: tax exists only as "UNCONFIGURED/REVIEWED" JSON. | FIN-02 |
| N-9 | Decimal prices vs integer paise, rounding | OPEN (correctly no default invented). | FIN-02 |
| V-2 | Upfront money applied to a purchase without limits | **FIXED structurally**: applications are now source-linked and bounded; project tests pass on my database (concurrency not re-probed by me). The same change introduced the signed "adjustment" behind FIN-01. | FIN-01 |
| V-7 | Dispatch needs a reviewed document | Stands; correct fail-closed gate. | — |

## 4. New findings

Severity: **HIGH** = will create wrong tax/accounting records or block a statutory duty once workflows run; **MEDIUM** = weakens evidence or controls, fix before live money; **LOW** = fix before scale or Gold. **CONFIRMED** = reproduced by a probe. Every finding separates the structural flaw, the CA question and the owner decision.

### FIN-01 · HIGH · Customers can pay an amount the tax invoice will never show — CONFIRMED (P01)

- **Structural flaw:** the amount due is `jewellery value + adjustment − upfront money applied`, where "adjustment" is any signed amount (a discount or an extra charge). A tax invoice may only contain lines equal to the selected pieces and must total exactly the jewellery value. Any discount or extra charge is therefore collected but never appears on any tax invoice.
- **Evidence:** `billing/models.py:106-152` (`PurchaseBalanceEvidence.adjustment_minor`); `trials/migrations/0010_purchase_balance_integrity.py:17` (payable identity); `trials/migrations/0007_postcapture_integrity.py:222` (invoice line must equal the purchase item) and the issue check below it (invoice must equal the confirmed jewellery value). P01: piece ₹5.00, adjustment −₹1.00 → customer paid ₹4.00; an invoice for ₹4.00 was refused ("Invoice lines require exact historical selected purchase pieces"); an invoice for ₹5.00 was issued. With +₹1.00: paid ₹6.00, invoiced ₹5.00.
- **Why it matters:** a doorstep discount missing from the invoice means GST is charged on the higher value and the books show money owed that will never arrive. An extra charge (for example a visit or damage fee) collected without an invoice is an unreported taxable supply; supplying without an invoice or with an incorrect invoice attracts a penalty of ₹10,000 or the tax evaded, whichever is higher (Section 122(1)(i)). Example: a ₹199 fee on 500 visits a month is ₹99,500 a month of receipts with no tax document.
- **CA question:** Q16. **Owner decision:** OD-3 (will agents ever discount or add charges?).
- **Fix:** model each adjustment as a typed invoice line (a discount inside the same rate, or a separate charge line with its own HSN/SAC and tax evidence). Change the issue check so the invoice total equals the amount due **before** upfront application; upfront money stays a payment, not an invoice line. Until the CA and owner answer, add a database check `adjustment_minor = 0` (fail closed). **Timing:** before Task 7 pricing.

### FIN-02 · HIGH · Tax is not a real field anywhere; an invoice cannot show GST on top of a price — CONFIRMED (P02, P03)

- **Structural flaw:** no typed HSN, taxable value, rate, CGST/SGST/IGST amount or round-off exists on purchases or invoice lines. Tax facts live in a free-form JSON blob that only has to be non-empty. Because the invoice must equal the jewellery value, "price + GST" is impossible; only "price already includes GST" can be represented, and its tax split cannot be checked.
- **Evidence:** `billing/models.py:15-49` (`TaxEvidence`), `:273-295` (`PurchaseItem`: amount + JSON), `:429-455` (`FinancialDocumentLine`: amount + JSON). P02: an invoice of ₹5.15 (₹5.00 + 3%) was refused whichever way it was built; ₹5.00 was accepted. P03: an invoice was issued with tax evidence `{"anything": "at all"}`.
- **Why it matters:** silver jewellery (HSN 7113) carries 3% GST (1.5% CGST + 1.5% SGST within a state), unchanged by the September 2025 rate rationalisation (secondary sources; confidence medium-high; CA to confirm). GST returns need rate-wise and HSN-wise values that JSON cannot reliably supply. If the CA advises tax-exclusive pricing, no invoice could be issued at all. Rounding (N-9) is also undefined.
- **CA question:** Q6–Q9. **Owner decision:** OD-2 (prices including GST?).
- **Fix:** typed tax lines per invoice line — HSN, quantity/unit, taxable value, component (CGST/SGST/IGST/cess), exact-decimal rate, amount in paise — plus header totals and an explicit round-off, with database checks that components add up. Rates come from effective-dated configuration, never constants. Same structure on the upfront charge if CA-01 makes it taxable. **Timing:** before Task 7/8.

### FIN-03 · HIGH · An unregistered, draft entity can issue "tax invoices" with none of the legally required details — CONFIRMED (P03, P04, P15)

- **Structural flaw:** issuing only checks that the issuer/recipient/line JSON is non-empty and that some tax review exists. Nothing requires the seller to be ACTIVE and GST-registered, or the document to carry GSTIN, addresses, the recipient's state and code, HSN, description, quantity, taxable value, rate, tax, place of supply or signature. A delivery challan can be issued at nil value.
- **Evidence:** `financial_snapshot_objects` (`trials/migrations/0006`) only tests "is a non-empty object"; `compliance/models.py:27-62` (`LegalEntity` defaults to DRAFT; registration may be blank). P03: invoice issued with `{"no": "gstin"}`, `{"no": "state"}`, `{"no": "hsn"}`. P15: seller entity DRAFT with blank registration → tax invoice issued. P04: dispatch document worth ₹0 issued for a real piece.
- **Why it matters:** a tax invoice is issued by "the registered person" (Section 31(1), Rule 46). Rule 46 lists the mandatory particulars, including a serial number of at most 16 characters unique for the financial year, and the recipient's name, address, delivery address and State name and code for unregistered recipients when the value is ₹50,000 or more. Rule 55 requires the challan to show HSN, description, quantity and taxable value. Missing particulars expose you to the Section 122(1)(i) penalty.
- **CA question:** Q10, Q12, Q14. **Owner decision:** OD-6 (seller entity and registration).
- **Fix:** issue guard: issuer entity ACTIVE with a GST registration effective on the issue date for the supplying state (FIN-04); validate required particulars per document type (typed columns preferred, at minimum a versioned schema checked in the database); keep a rendered-document hash/reference and signature evidence; challan value = sum of unit values and greater than zero. **Timing:** before the first real dispatch (Task 7) and invoice (Task 8).

### FIN-04 · HIGH · The system does not know which state anything is in — CONFIRMED (P12, P17)

- **Structural flaw:** `Hub` has only code, name and market; `Market` only a country; the booking address is free text plus PIN; `LegalEntity` has one `tax_registration_number` and no PAN, address or state. There is no "hub X is registered under GSTIN 29… in Karnataka" and no customer state. The single registration field can be overwritten through the ORM without an audit event (Admin edits are audited). Existing documents keep their snapshots when an entity changes (good), but nothing records which GSTIN was effective on which date.
- **Evidence:** `markets/models.py:27-36`, `compliance/models.py:27-62`, `trials/models.py:53-57`. P17 printed the real field lists; P12 rewrote the registration with no audit row.
- **Why it matters:** for goods that move, the place of supply is where the movement ends — the customer's address (IGST Act Section 10(1)(a)). Same state as the hub: CGST+SGST; different state: IGST. If a Bengaluru service area ever includes a PIN outside Karnataka (for example Hosur, Tamil Nadu), tax paid as CGST+SGST by mistake must be paid again as IGST and the first amount claimed back (Section 77). GST registration is state-wise and each GSTIN is a distinct person, so moving stock from a Karnataka hub to a Telangana hub is itself a taxable supply needing an invoice and IGST. Hyderabad therefore needs a Telangana registration in the data model.
- **CA question:** Q21–Q23. **Owner decisions:** OD-6, OD-10.
- **Fix:** add `GstRegistration` (entity, state code, GSTIN, address, effective from/to, status, history); link each hub to a registration with effective dates; store the recipient state and state code on the booking/recipient snapshot (derived from PIN and confirmed); compute and snapshot place of supply and tax type on every document; one series per registration. Guard entity identity fields once issued documents reference them. **Timing:** before Task 8; mandatory before Hyderabad.

### FIN-05 · HIGH · Money can come in but can never go out — CONFIRMED (P08)

- **Structural flaw:** once upfront money is collected the charge is final (`COLLECTED` has no exit) and the payment window (`PAID`) is final. Gateway evidence can only be AUTHORISATION or CAPTURE — not REFUND, CHARGEBACK or REVERSAL. No models exist for refunds, retained or forfeited amounts, gateway settlements, gateway fees, GST on fees or bank receipts (4.M9 lists these as pending).
- **Evidence:** `payments/models.py:53` (charge transitions), `:145-149` (context transitions), `:369-371` (transaction kinds). P08: voiding a collected charge was refused (ORM: "transition not permitted"; SQL: "Close reconciled contexts before voiding obligation"); inserting a REFUND fact failed the `gateway_transaction_kind` check; no model name contains refund/settlement/retention; the ₹1.00 deposit stays "owed" with no possible closing record.
- **Why it matters:** after a no-purchase visit, a partial purchase or a cancellation, the upfront money stays on the books as owed to the customer forever, and the real refund or retained amount has nowhere to go — so the deposit-liability balance an auditor or investor asks for will be wrong. Gateways settle net of fees plus 18% GST on fees; without settlement and fee records, bank deposits will not match sales and the input credit on fees cannot be supported. A deposit is not consideration until applied (Section 2(31)); a retained amount may become taxable — cancellation charges are taxed like the main supply under Circular 178/10/2022, while some forfeitures are not. The system must record which happened.
- **CA question:** Q1–Q5, Q18, Q19. **Owner decision:** OD-1, OD-7.
- **Fix:** complete 4.M9 — `Refund` (request, approval, provider submission, verified outcome; idempotent; bounded by unconsumed source money including in-flight refunds), a retention/forfeiture record (policy version, reason, amount, tax-document link), REFUND/CHARGEBACK/REVERSAL gateway facts, `GatewaySettlement` with per-transaction lines (gross, fee, tax on fee, net, adjustments), bank statement lines and matches. Database invariant: deposit liability = collected − applied − refunded − retained ≥ 0. **Timing:** before any live money.

### FIN-06 · HIGH · Jewellery can be handed over as "sold" with no tax invoice; invoice timing follows payment, not supply — CONFIRMED (P09)

- **Structural flaw:** a piece can move to SOLD and customer custody as soon as the purchase is paid, whether or not a tax invoice exists, and the invoice can be issued any time later or never. Conversely, a tax invoice can only be issued after full payment.
- **Evidence:** `trials/migrations/0008_visit_corrections.py:202` (sale requires a confirmed purchase, no invoice check); `0007` issue check (tax invoice requires a CONFIRMED purchase). P09: unit SOLD with CUSTOMER custody and zero tax invoices for the booking.
- **Why it matters:** home trial is "goods sent on approval": move them under a delivery challan and issue the invoice when the customer accepts (Section 31(7); CBIC Circular 10/10/2017-GST; Rule 55(4)). An invoice must exist before or at delivery (Section 31(1)). Supplying without an invoice → ₹10,000 or the tax evaded, whichever is higher (Section 122(1)(i)). If goods stay with a customer (an exception), the invoice falls due six months from removal regardless (Section 31(7)) — nothing tracks that clock. E-way bills are generally not required for Chapter 71 jewellery, but a State may require intra-state e-way information for these goods above at least ₹2 lakh under Rule 138F; Kerala has done so. Karnataka's position is **unknown** (sources conflict) — needs the Karnataka notification.
- **CA question:** Q10, Q11, Q13. **Owner decision:** OD-4 (BD-05/V-5 handover).
- **Fix:** the SALE custody event must require an ISSUED tax invoice containing that purchase item (or issue the invoice and record the sale in one transaction). Let the CA decide whether issue requires payment; tie issue to acceptance/handover. Add an "out on approval since" monitor that alarms well before six months, and an optional e-way bill/Part A reference on dispatch documents. **Timing:** before Task 7 doorstep sale.

### FIN-07 · MEDIUM · One sale can be invoiced twice; numbering is not tied to GSTIN, financial year or document type — CONFIRMED (P04, P05)

- **Structural flaw:** nothing stops a second tax invoice for the same pieces. A numbering series belongs to an entity, not to a GSTIN, financial year or document type; challans, receipts, invoices and corrections can share one series; the caller supplies the number (gaps possible); the printed number can exceed 16 characters.
- **Evidence:** `billing/models.py:342-356` (`DocumentSeries`), `:382`, `:398-404` (only `(series, number)` is unique). P05: two ISSUED invoices for one purchase. P04: dispatch #1 and a tax invoice in one series; `ARKATARA-BLR-TAX-INVOICE/98765432109876` accepted.
- **Why it matters:** two invoices mean GST paid twice and sales overstated, or a later credit note to undo it. Rule 46 requires a consecutive serial of at most 16 characters, unique per financial year; mixed series create unexplained gaps in the document summary filed with returns.
- **CA question:** Q14, Q15.
- **Fix:** a purchase item may appear on at most one ISSUED tax invoice unless superseded by an issued credit note; series = (GST registration, document type, financial year) with a check that the issue date falls in that year; numbers allocated from a locked counter inside the issuing transaction; allowed characters and ≤16 rendered characters enforced; explicit cancellation records. **Timing:** before Task 8.

### FIN-08 · MEDIUM · Corrections have no direction, no limit, no deadline and can reference another customer's goods — CONFIRMED (P06)

- **Structural flaw:** one generic ADJUSTMENT kind with a positive amount: credit note (reduces) and debit note (increases) are indistinguishable; corrections can exceed the original; a correction can be dated before the invoice it corrects; its lines can reference another booking's stock.
- **Evidence:** `billing/models.py:374-380`, `:386`; `0007:241` (only booking/currency/obligation sameness checked). P06: two corrections of ₹50 each against a ₹5 invoice (20×), one dated 30 days before the invoice, plus a line pointing at another booking's reserved piece — all issued.
- **Why it matters:** Section 34 treats credit and debit notes differently; credit notes for a year's supplies must be declared by 30 November of the following year or the annual-return date, whichever is earlier. Unbounded credits could erase sales and GST without evidence a CA would accept.
- **CA question:** Q15, Q17. **Owner decision:** OD-7 (returns policy, BD-10).
- **Fix:** CREDIT_NOTE and DEBIT_NOTE kinds with typed tax deltas; cumulative credits capped at the original taxable value and tax per line; issue date not before the original; lines must reference lines of the corrected document; reason code; Section 34 deadline flag. **Timing:** before Task 8.

### FIN-09 · MEDIUM · No period close: invoices, payments and applications can be back-dated — CONFIRMED (P04, P06, P14)

- **Structural flaw:** business-date fields only reject future times (`common/verification.py:18-22`); there is no accounting or GST period and no lock.
- **Evidence:** P04: invoice dated 20 March 2026 (previous financial year), before the booking existed and before the sale. P06: correction dated before its original. P14: payment applied 200 days before the money was captured.
- **Why it matters:** after a month's GST return or the year-end close, records must not silently move into that period; back-dating changes filed figures and the year-end deposit balance. GST requires a log of every edit to electronic records (Rule 56(8)), and company accounting software must keep an audit trail that cannot be disabled (Companies (Accounts) Rules, Rule 3(1), from 1 April 2023).
- **CA question:** Q27. **Owner decision:** monthly close calendar and who may reopen.
- **Fix:** `AccountingPeriod`/return period with OPEN/CLOSED and attributable close/reopen; database guard rejecting business dates inside closed periods; keep the immutable recorded time beside the business date and limit skew unless an attributed back-dating approval exists; `allocated_at` ≥ receipt time; invoice date ≥ acceptance/confirmation; correction date ≥ original. **Timing:** before Task 8.

### FIN-10 · MEDIUM · A sale can be priced with tax unconfigured and invoiced under a different treatment — CONFIRMED (P10)

- **Structural flaw:** a purchase may reference UNCONFIGURED tax evidence through seal, payment and handover; the invoice references its own, possibly different, tax evidence.
- **Evidence:** `billing/models.py:66`, `:385`; seal/confirm checks in `0007` ignore tax status. P10: purchase with unconfigured tax confirmed, piece SOLD, invoice issued under a different reviewed evidence row.
- **Why it matters:** the amount collected was not calculated with the tax the invoice declares.
- **CA question:** none. **Owner decision:** none.
- **Fix:** sealing requires REVIEWED tax evidence effective at seal; the invoice must use the purchase's tax evidence or typed purchase tax lines. **Timing:** before Task 7 pricing.

### FIN-11 · MEDIUM · The "upfront record" document has no defined legal meaning and can be issued for money never received — CONFIRMED (P07)

- **Structural flaw:** one UPFRONT_RECORD kind, checked only for amount and currency equal to the charge, with no link to an actual receipt.
- **Evidence:** `billing/models.py:374-380`; `0007` issue check. P07: UPFRONT_RECORD issued while the charge was still QUOTED and nothing had been captured.
- **Why it matters:** depending on CA-01 it must be a receipt voucher (an advance against goods: no GST at receipt under Notification 66/2017, but a Rule 50 receipt voucher is still required), an invoice for a trial fee (taxable), or nothing at all (a pure security deposit, Section 2(31)). A receipt voucher for money never received is a false document.
- **CA question:** Q1–Q5. **Owner decision:** OD-1.
- **Fix:** after the CA answers, replace with explicit kinds — RECEIPT_VOUCHER (requires a linked captured allocation), FEE_INVOICE (typed tax), REFUND_VOUCHER (linked to a refund) — each with its preconditions. **Timing:** before real money in Task 6/8.

### FIN-12 · MEDIUM · You cannot prove which legal entity received a payment — CONFIRMED (P11)

- **Structural flaw:** the merchant account is a free string; nothing maps it to a legal entity or bank account; the upfront payee and the seller may be different entities.
- **Evidence:** `payments/models.py:219-221`, `:365-367`; the receipt guard only compares strings. P11: upfront payee entity A, seller entity B, both collected through the same merchant-account string, invoice issued by B.
- **Why it matters:** books per entity and any future entity transition (BD-11) need receipts in the entity whose account actually received them; mixing creates inter-entity balances nobody recorded.
- **CA question:** Q23. **Owner decision:** OD-6.
- **Fix:** `ProviderAccount` (provider, account id, environment, legal entity, settlement bank account, effective dates); attempts and receipts reference it; database check that payee/seller equals the account's entity, or an explicit inter-entity record. **Timing:** before Task 8.

### FIN-13 · MEDIUM · Test-mode money can confirm bookings, complete sales and back tax invoices — CONFIRMED (P03)

- **Structural flaw:** the environment (TEST/LIVE) is recorded but no guard uses it.
- **Evidence:** `payments/models.py:221`, `:367`; no SQL guard checks `LIVE`. P03: the confirmed purchase and issued invoice are backed only by TEST captures.
- **Why it matters:** a misconfigured key in production would create sales, invoices and GST liability for money that does not exist.
- **Fix:** a per-database deployment-environment setting enforced in the allocation guard: production accepts only LIVE receipts; other environments only TEST. **Timing:** before Task 8.

### FIN-14 · MEDIUM (HIGH before Gold) · Stock has no cost, owner, weight or hallmark record — CONFIRMED (P17)

- **Structural flaw:** a physical unit records only id, variant, hub, status, custody and timestamps.
- **Evidence:** `inventory/models.py:35-120`; P17 printed the field list.
- **Why it matters:** you cannot value closing stock (including pieces out on trial at 31 March, which are still yours), calculate cost of goods sold or margin, value damage/loss write-offs, link supplier invoices for input credit, or prove which entity owns the stock. For gold, BIS hallmarking with HUID is mandatory and the sale bill must show each article's net weight of precious metal, purity (carat and fineness) and hallmarking charges. HUID hallmarking for silver became available on a voluntary basis from 1 September 2025 (IS 2112:2025); if you advertise "925", per-piece hallmark evidence is how you prove it.
- **CA question:** Q20. **Owner decision:** OD-8, OD-11 (BD-09, BD-12).
- **Fix:** unit acquisition (supplier, supplier invoice reference, cost, date, owning entity), weights (gross/net, unit, source), hallmark (HUID, purity mark, assaying centre, date), and a write-off/disposition record with value and approval. **Timing:** acquisition/cost before receiving live stock; hallmark before Gold.

### FIN-15 · LOW now (HIGH before Gold) · High-value sales complete with no PAN or declaration — CONFIRMED (P13)

- **Structural flaw:** a ₹2,50,000 sale was confirmed and invoiced with no PAN or identity in the customer snapshot.
- **Why it matters:** the income-tax rules require PAN, or a declaration (Form 97 has replaced Form 60 under the Income-tax Rules, 2026), for a purchase of goods above ₹2 lakh per transaction (confidence medium — confirm the exact rule). Receiving ₹2 lakh or more in cash from one person in a day, a transaction or an event is prohibited — now Section 186 of the Income-tax Act, 2025 (formerly 269ST), with a 100% penalty (confidence medium). Dealers in precious metals become PMLA reporting entities for cash transactions of ₹10 lakh or more. There is no cash path today (good); silver baskets rarely cross ₹2 lakh, gold will.
- **CA question:** Q24, Q25. **Owner decision:** OD-5.
- **Fix:** configurable threshold check at seal: above it require a format-validated PAN or a Form 97 reference plus KYC evidence; keep the payer details returned by the gateway; never add a cash method without BD-06 and these controls. **Timing:** before Gold.

### FIN-16 · MEDIUM · Personal data can never be erased, and nothing knows how long records must be kept — CONFIRMED (P16)

- **Structural flaw:** anonymising a booking's name/address is refused by the database ("Checkout identity and snapshots are immutable"); no retention or erasure fields exist.
- **Why it matters:** the DPDP Rules, 2025 were notified on 14 November 2025; their erasure and retention duties apply from about 13 May 2027, and personal data must be erased when the purpose is served unless another law requires keeping it. GST requires keeping records — including delivery challans, receipt and refund vouchers — for 72 months from the annual-return due date (Section 36, Rule 56(1)); companies keep books for 8 financial years (Companies Act, s.128(5)). Invoice and challan copies must stay; contact data on no-purchase bookings may have to go.
- **CA question:** Q26. **Legal:** LR-04. **Owner decision:** OD-9.
- **Fix:** a retention schedule per record class with its legal basis and a computed retain-until date; separate statutory document snapshots (kept) from operational personal data (erasable); a guarded, audited anonymisation that the database allows only after retain-until. **Timing:** design now; implement before May 2027 and before customer accounts.

### FIN-17 · MEDIUM · No single source of truth for sales, GMV, deposits or refunds; no accounting export — CONFIRMED by inspection and P01/P05/P09

- **Structural flaw:** the analytics app has no models (4.M11 pending); there is no ledger or journal export.
- **Why it matters:** investors will ask for GMV versus recognised sales, sales by city and entity, trial-to-sale conversion, the deposit liability and refunds. Today confirmed purchases, issued invoices and captured cash can each give a different answer (P01: cash ≠ invoice; P05: two invoices per sale; P09: sale without invoice).
- **Fix:** a metric dictionary in docs tying each metric to one authoritative record and date (for example recognised sales = issued tax invoices minus credit notes by issue date, as the CA confirms; GMV = confirmed purchases; deposit liability per FIN-05), a reconciliation report that shows the differences, and a CA-approved journal-export mapping. **Timing:** design with Task 8; validate in Task 10.

## 5. What is sound (do not rebuild)

- Append-only and immutable evidence enforced by PostgreSQL triggers; protected foreign keys, no cascade deletes.
- Upfront money kept separate from jewellery value; applications bounded by captured source and balance, and cannot cross booking, entity or currency (V-2).
- Allocations cannot exceed the receipt or the obligation; upfront receipts cannot pay purchase balances; capture uncertainty keeps stock protected; late receipts are retained.
- "Unconfigured tax" never becomes 0%; issued documents seal header and lines.
- SOLD requires a paid purchase; returns need hub receipt and QC.
- No card data, no cash path, no Admin editing of financial records; dispatch blocked until a reviewed document exists (V-7).

## 6. Questions for the CA

Each can be answered yes/no or with a number. IDs refer to `docs/DECISIONS.md`.

1. (CA-01) For GST, is our upfront booking payment (a) a refundable security deposit that is not consideration until applied, (b) an advance against the jewellery sale, or (c) payment for a separate trial/visit service? Answer a, b or c.
2. (CA-01) If (b): must we issue a Rule 50 receipt voucher when it is received, even though no GST is payable on advances for goods? Yes/No.
3. (CA-01) If (c): which SAC code and what GST rate apply? Numbers.
4. (CA-01/LR-01) If we keep all or part of the upfront payment after a no-show, cancellation or no-purchase, is GST payable on the kept amount? Yes/No. If yes, what rate, and is a tax invoice required? Number; Yes/No.
5. (CA-01) When we refund the upfront payment, is a Rule 51 refund voucher required? Yes/No.
6. (CA-02/BD-04) Should displayed prices include GST? Yes/No.
7. (CA-02) What is the 8-digit HSN code for our S925 silver jewellery, and is the rate 3% (1.5% CGST + 1.5% SGST within Karnataka)? Number; Yes/No.
8. (CA-02) How many HSN digits must our B2C invoices show at our expected turnover? 0, 4, 6 or 8.
9. (CA-02) Do we round GST per line (Yes) or per invoice (No)? Must the invoice total be rounded to the nearest rupee with a separate round-off line? Yes/No.
10. (CA-02/LR-03) Is a home trial "goods sent on approval" under Section 31(7), moved on a Rule 55 delivery challan, with the tax invoice issued when the customer accepts the piece? Yes/No.
11. (CA-02) Must the tax invoice be issued before the agent hands over the purchased piece, even if payment confirmation arrives later? Yes/No.
12. (CA-02) Must the delivery challan state a value per piece? Yes/No. Is a separate document needed for the pieces returning to the hub? Yes/No.
13. (CA-02) Is an e-way bill or Rule 138F Part A information required for intra-state movement of silver jewellery in Karnataka? Yes/No. Above what consignment value? Number. The same for gold in Telangana.
14. (CA-02) One invoice series per GSTIN per financial year, with separate series for challans, credit notes, receipt vouchers and refund vouchers? Yes/No. May numbering restart every 1 April? Yes/No.
15. (CA-02) May an issued invoice be cancelled before it is reported in GSTR-1 (Yes), or must every correction be a credit note (No)?
16. (CA-02) If an agent gives a discount at the door, must it appear on the invoice to reduce taxable value under Section 15(3)(a)? Yes/No. Is a visit, delivery or damage charge taxable? Yes/No; rate? Number.
17. (CA-02/BD-10) For after-sale returns, do we issue a credit note and reduce output GST? Yes/No. Is the deadline 30 November after the financial year (or the annual-return date if earlier)? Yes/No.
18. (CA-02) Can we claim input credit on the GST charged on Razorpay fees? Yes/No. Record sales gross with fees as an expense? Yes/No.
19. (CA-02) Should a chargeback be recorded as a credit note against the sale? Yes/No.
20. (CA-02/CA-03) Value inventory by specific identification at purchase cost? Yes/No. Include pieces with customers on trial at 31 March in our closing stock? Yes/No.
21. (CA-03) Must the seller hold a Telangana GSTIN before the first Hyderabad sale? Yes/No. Is moving stock from a Karnataka hub to a Telangana hub a taxable supply needing a tax invoice and IGST? Yes/No.
22. (CA-02) If a Bengaluru delivery address is outside Karnataka, do we charge IGST? Yes/No.
23. (CA-03) Do we need GST registration before collecting the first upfront payment, given our expected first-year turnover? Yes/No. Must the gateway and bank accounts be in the seller entity's name? Yes/No.
24. (Income tax) For a jewellery purchase above ₹2,00,000 in one transaction, must we collect PAN or a Form 97 declaration under the Income-tax Rules, 2026? Yes/No. What is the threshold? Number.
25. (PMLA) If we never accept cash, does any PMLA reporting duty apply to us? Yes/No.
26. (CA/LR-04) What is the longest retention period that applies to our invoices, challans, receipts, refunds and gateway/bank records, in years after the financial year ends? Number.
27. (CA-02) For GST periods, does the invoice date govern even if the invoice is recorded late? Yes/No. May we ever back-date an invoice? Yes/No.

## 7. Decisions for the owner

1. **OD-1 (BD-01):** Is the upfront payment a refundable deposit, an advance on the purchase, or a fee? How much, and when is it refunded, applied or kept? Needs CA answers 1–5.
2. **OD-2 (BD-04):** Show prices including GST? Recommend deciding with the CA before any price is published.
3. **OD-3:** Will agents ever give a discount or add a charge at the door? If "no", Codex can block adjustments entirely for now (simplest and safest).
4. **OD-4 (BD-05/V-5):** If payment is pending at the door, does the piece stay with the customer? Recommend: no handover before verified payment and an issued invoice.
5. **OD-5 (BD-06):** Payment methods. Recommend: UPI/cards through Razorpay only, no cash at launch.
6. **OD-6 (BD-11):** Who is the legal seller (proprietorship, LLP or company)? Whose bank and Razorpay accounts receive money? Who owns the stock? When is it GST-registered? Recommend: one entity, registered in Karnataka, with its own current account and Razorpay account before Task 8.
7. **OD-7 (BD-10):** After-sale return and exchange policy.
8. **OD-8 (BD-09/LR-01):** Who bears damage or loss, and who approves write-offs?
9. **OD-9 (LR-04):** Retention and erasure policy after legal advice.
10. **OD-10:** Limit the Bengaluru service area to Karnataka PIN codes for launch? This avoids IGST cases until FIN-04 is built.
11. **OD-11 (BD-12):** Do not launch Gold until hallmark/HUID, weight, PAN capture and Telangana registration support exist.
12. **OD-12:** Engage a CA now, send them section 6, and record their written answers in `DECISIONS.md` (CA-01–CA-03).

## 8. What Codex should do next (in order)

Follow D-28: first record a disposition (VALID / PARTIAL / INVALID / DECISION REQUIRED) for each FIN item with evidence; do not change requirements to fit the code; invent no rates, thresholds or defaults; keep all BD/CA/LR gates open.

1. **Record this report** and dispositions under `docs/reviews/`; map FIN-01–FIN-17 in TASKS.md. *(now)*
2. **Policy-independent database fixes**, each with a regression test converted from my probe (expecting rejection): *(before Task 7)*
   1. FIN-07: one issued tax invoice per purchase item (supersede only by credit note); series typed by document type, financial year and registration; number allocated inside the issuing transaction; ≤16 characters.
   2. FIN-06: SALE requires an issued invoice for that item (or atomic issue + sale).
   3. FIN-09: date-order guards (`allocated_at` ≥ receipt; invoice ≥ acceptance; correction ≥ original; bounded skew) plus a period-lock model.
   4. FIN-10: seal requires REVIEWED tax; invoice uses the purchase's tax evidence.
   5. FIN-08: split ADJUSTMENT into CREDIT_NOTE/DEBIT_NOTE with caps, ordering and line linkage.
   6. FIN-13: LIVE-only receipts in production.
   7. FIN-12: `ProviderAccount` → `LegalEntity`.
   8. FIN-01 (interim): `adjustment_minor = 0` until the CA and owner decide.
   9. FIN-03: issuer must be ACTIVE with an effective registration; required-particular validation hooks; challan value > 0.
3. **Schema shape, no values:** FIN-04 (`GstRegistration`, hub link, recipient state code, place-of-supply snapshot) and FIN-02 (typed tax lines and totals; rates from configuration). *(before Task 7/8)*
4. **Complete 4.M9:** FIN-05 refunds, retention, chargebacks, settlements, fees, bank lines and the deposit-liability invariant. *(before any live money)*
5. **Complete 4.M5:** FIN-14 acquisition, cost, owner, weight, hallmark and write-off records. *(before live stock receipt; hallmark before Gold)*
6. **FIN-11:** split upfront document types once CA-01 is answered. *(before real money)*
7. **FIN-15:** configurable PAN/Form 97 threshold capture. *(before Gold)*
8. **FIN-16:** retention schedule and guarded anonymisation design. *(design now; before May 2027)*
9. **FIN-17:** metric dictionary, reconciliation report and journal-export specification. *(with Task 8)*
10. **Docs:** update COMPLIANCE_AND_FINANCE.md and DATA_MODEL.md to reflect the above while keeping CA/LR gates open. Then request a Claude re-review.

## 9. Launch readiness and next stage

- **Finance/compliance launch readiness: NOT READY.** Blockers: FIN-01 to FIN-06; CA-01/CA-02/CA-03 answers; entity, GST registration, bank and gateway accounts (BD-11); refunds and reconciliation (FIN-05).
- **Would I let the next stage be built on this? YES, WITH CONDITIONS.** Continue the remaining Task 4 models and Tasks 5–6 (storefront; booking with development fakes) — they do not depend on invoice structure. **Do not** start Task 7 purchase pricing or Task 8 billing until the structural fixes in section 8, steps 2–3, have landed and the CA's answers to questions 1–16 are recorded. **Do not** take any real money until FIN-05, FIN-12 and FIN-13 are also fixed.

## 10. Legal sources (checked 2026-10-10)

Confidence reflects source quality; the CA must confirm before production use.

| Point | Source | Confidence |
| --- | --- | --- |
| Goods on approval: challan, invoice at acceptance | [CBIC Circular 10/10/2017-GST](https://gstcouncil.gov.in/sites/default/files/2024-06/circularno-10-gst.pdf) | High |
| Invoice timing; six-month rule | [Section 31 CGST Act](https://www.knowyourgst.com/gstlaw/the-central-goods-and-services-tax-act-2017/31-tax-invoice-31/) | High |
| Invoice particulars, 16 characters, ₹50,000 State code | [Rule 46 (CBIC)](https://taxinformation.cbic.gov.in/content/html/tax_repository/gst/rules/cgst_rules/active/chapter6/rule46_v1.00.html), [summary](https://gstzen.in/a/tax-invoice-cgst-rule-46.html) | High |
| Delivery challan particulars | [Rule 55](https://gstzen.in/a/transportation-of-goods-without-issue-of-invoice-cgst-rule-55.html) | High |
| Silver jewellery 3% after Sept 2025 | [Tally HSN 7113](https://tallysolutions.com/gst/hsn-code-7113-product-classification-gst-rate-business-filing-guide/), [GST 2.0 summary](https://www.basunivesh.com/gst-on-gold-and-silver-after-gst-2-0/) | Medium-high |
| E-way bill exemption for Chapter 71; Kerala exception | [GSTN advisory 6 Feb 2025](https://gstgyaan.com/advisory-on-e-way-bill-generation-for-goods-under-chapter-71-dated-06-02-2025) | High |
| Rule 138F State option (≥ ₹2 lakh) | [Rule 138F text](https://gstgyaan.com/rule-138f-information-to-be-furnished-in-case-of-intra-state-movement-of-gold-precious-stones-etc-and-generation-of-e-way-bills); Karnataka claim only in [one secondary source](https://www.kanakkupillai.com/learn/e-way-bill-applicability-for-gold-and-precious-stones/) | High / **Low for Karnataka** |
| No GST on advances for goods; receipt voucher still required | [GSTZen](https://gstzen.in/a/gst-on-advances-received-for-future-supplies.html) | High |
| Deposit not consideration until applied | [Section 2(31)](https://gstgyaan.com/section-2(31)-definition-of-consideration-under-cgst-act) | High |
| Cancellation charges vs forfeiture | [Circular 178/10/2022 analysis](https://www.grantthornton.in/insights/blogs/taxability-of-liquidated-damages-penalties-and-similar-charges/) | Medium |
| Discounts on invoice | [Section 15](https://www.knowyourgst.com/gstlaw/cgst-act/15-value-of-taxable-supply-15/) | High |
| HSN digits on invoices | [Notification 78/2020-CT](https://www.gstcouncil.gov.in/sites/default/files/2024-05/notfctn-78-central-tax-english-2020.pdf) | High |
| Credit note deadline | [Section 34](https://caclub.in/section-34-of-cgst-act-2017-credit-and-debit-notes/) | High |
| Retention 72 months | [Section 36](https://www.knowyourgst.com/gstlaw/cgst-act/36-period-of-retention-of-accounts-36/) | High |
| Documents to keep; edit log | [Rule 56](https://studycafe.in/rule-56-cgst-rules-maintenance-accounts-registered-persons-16500.html) | High |
| Penalty for no/incorrect invoice | [Section 122 (CBIC)](https://taxinformation.cbic.gov.in/content/html/tax_repository/gst/acts/2017_CGST_act/active/chapter19/section122_v1.00.html) | High |
| Place of supply; wrong tax type | [ICAI WIRC material](https://old.wirc-icai.org/images/material/Refresher-Course-GST-Place-Supply.pdf) | High |
| State-wise registration; stock transfer | [VJM Global](https://www.vjmglobal.com/feeds/blog/gst-registration-branch-office-another-state) | Medium-high |
| Company audit trail; 8-year books | [Taxguru](https://taxguru.in/company-law/demystifying-audit-trail-edit-log.html), [Section 128](https://ca2013.com/128-books-of-account/) | High |
| Cash receipt limit (Section 186, IT Act 2025) | [eztax](https://eztax.in/income-tax-act-2025/section-186), [VisaVerge](https://www.visaverge.com/news/new-cash-transaction-limits-under-income-tax-act-take-effect-april-1-2026/) | Medium |
| PAN rules 2026; Form 97 | [Business Today, 14 Jun 2026](https://www.businesstoday.in/personal-finance/news/story/pan-rules-2026-property-deals-cash-deposits-insurance-payments-get-major-overhaul-536731-2026-06-14) | Medium |
| PMLA ₹10 lakh cash for jewellers | [RSM India](https://www.rsm.global/india/sites/default/files/media/RSM%20India/Publications/2021/rsm_india_newsflash_-_record_of_cash_transactions_in_gems_jewellery_over_and_above_rs._10_lakhs_mandatory_under_pmla.pdf) | High |
| Silver HUID voluntary from 1 Sept 2025 | [GJEPC](https://gjepc.org/solitaire/?p=33098), [The Week/PTI](https://www.theweek.in/wire-updates/business/2025/09/04/del79-biz-silver-hallmarking.html) | High |
| Hallmarked-article bill contents | [BIS consumer FAQ](https://www.bis.gov.in/hallmarking-overview/hallmarking-faqs/common-consumer-faq) | High |
| DPDP Rules 2025 timeline | [PIB](https://static.pib.gov.in/WriteReadData/specificdocs/documents/2025/nov/doc20251117695301.pdf), [Mondaq](https://www.mondaq.com/india/data-protection/1708164/digital-personal-data-protection-rules-2025-notified) | High |
| Gateway fee GST, settlement timing | [Razorpay pricing explainer](https://razorpay.com/blog/razorpay-payment-gateway-pricing-explained/) | Medium |

**Unknown, and what is needed:** seller entity type and GST registration status (no `BUSINESS_FACTS.md`; BD-11 open); expected turnover (needed for HSN digits and registration questions); Karnataka and Telangana Rule 138F notifications (need the State notifications); exact Income-tax Rules, 2026 rule numbers (need the notified rules text).
