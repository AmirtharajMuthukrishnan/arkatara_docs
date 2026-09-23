# Business rules

Status: Approved business invariants and explicit roadmap requirements; settings and unresolved policies remain open\
Updated: 2026-09-19

This is the canonical rule reference. Read [DECISIONS.md](DECISIONS.md) before interpreting any example as policy.

## Rule classifications

- FINALIZED RULE: explicitly approved invariant or explicit requirement in the supplied roadmap.
- CONFIGURABLE RULE: the system must allow configuration, but a value is approved only when expressly recorded. Configurability does not resolve a policy.
- UNRESOLVED BUSINESS DECISION: no outcome selected. The linked register uses BUSINESS DECISION for commercial choices and CA REVIEW or LEGAL REVIEW for external interpretation.

The seven-day cart duration, two-box pilot limit, three/five-piece plans and INR 29/49 deposits are suggestions/examples. None is an approved production default.

## FINALIZED RULE: booking, trial and customer identity

| ID | Rule |
| --- | --- |
| R-01 | One TrialBooking may contain multiple TrialBoxes; each TrialBox belongs to exactly one category. A Ring box cannot contain Chains. |
| R-02 | One TrialBooking normally represents one customer home visit, with all boxes fulfilled together from the selected city's inventory for the same customer and address. Multiple boxes never imply independent customer checkouts. |
| R-03 | Browsing, Trial Cart, booking, reservation, dispatch, trial, selection, purchase, payment, invoice and inventory return remain distinct business concepts. A booking is not automatically a sale. |
| R-04 | The pilot has no required customer login, password or registration wall. Booking collects name, mobile, address and PIN code without requiring an account. |
| R-05 | Trial Cart selections persist locally through refresh, category/navigation changes and browser restart for a configured period. The roadmap selects localStorage; no customer account is required for this persistence. |
| R-06 | Adding to or restoring an anonymous cart never reserves stock. Browser selections are untrusted customer intent. |
| R-07 | Backend validation controls prices, availability, categories, eligibility, deposit, tax, serviceability and payment status. Stale or manipulated frontend data cannot establish a booking. |

## FINALIZED RULE: markets, catalogue and physical stock

| ID | Rule |
| --- | --- |
| R-08 | Inventory belongs to hubs within markets/cities. Bengaluru and Hyderabad stock are separate pools; no intercity fulfilment is assumed. |
| R-09 | Bookability must reflect selected-market/hub inventory and serviceability. Coming-soon or unavailable offerings must not produce valid bookings merely because the client displays them. |
| R-10 | City/Market, Hub, ServiceArea, Material, Purity and Category are foundational concepts. Serviceable PIN codes and eligibility are data, not hard-coded city checks. |
| R-11 | Material and purity are separate. Product design, ProductVariant and unique physical InventoryUnit are separate responsibilities. A catalogue row is not the physical piece. |
| R-12 | Catalogue and pricing structure must support FIXED and WEIGHT_BASED pricing. Silver using FIXED initially is a possibility; actual price formulas and price commitment timing remain open. |
| R-13 | Gold and Hyderabad activation alone must not require a core schema redesign, backend rewrite or duplicate frontend. Additional business requirements may still need explicitly approved work. |
| R-14 | Physical stock cannot be incompatibly committed or sold twice. Reservation must handle concurrent booking attempts transactionally. |
| R-15 | InventoryMovement history is append-only and must reconstruct physical movement/custody. SKU identifiers must not be parsed to infer business rules. |
| R-16 | Financially confirmed purchases mark the relevant inventory SOLD. Physical handover during uncertain payment is unresolved, not an agent assumption. |
| R-17 | Unpurchased dispatched items go through RETURNING_TO_HUB and QC_PENDING before AVAILABLE. They cannot become available while still with an agent, merely on booking cancellation, or before passing QC. |

## FINALIZED RULE: operations and verification

| ID | Rule |
| --- | --- |
| R-18 | Django Admin is the initial back office. Authenticated agents use a separate mobile staff interface for assigned/permitted bookings at the customer's home. |
| R-19 | The agent cannot manually override authoritative prices. The backend calculates the customer's selected purchase and final payable amount. |
| R-20 | Backend verification of the Arrival OTP transitions OUT_FOR_DELIVERY to IN_PERSON_TRIAL. Store the challenge securely, enforce expiry/attempt limits and record verification. Exact settings remain configurable. |
| R-21 | There is one Arrival OTP challenge/code delivered through WhatsApp and SMS when those integrations exist. Channel delivery must not generate independent codes. Resend/rotation settings remain open. |
| R-22 | There is no second OTP merely to confirm product selection. Successful verified payment provides the stronger purchase confirmation. |

## FINALIZED RULE: money, documents and evidence

| ID | Rule |
| --- | --- |
| R-23 | Agent QR and customer payment URL for the same final payable amount refer to the same PaymentAttempt/transaction context. Different presentation channels must not create independent payable charges. Retry and changed-selection rules remain open. |
| R-24 | Backend gateway verification and authenticated webhook processing determine payment state. Screenshots, client callbacks and agent assertions alone cannot mark payment successful. Processing must be idempotent. |
| R-25 | Trial Deposit is separate from jewellery price. Applying previously collected funds changes the amount outstanding, not the historical jewellery selling price. Whether/how to charge or apply a deposit remains unresolved. |
| R-26 | Business funds use business accounts. Payments, refunds, gateway transactions, gateway settlements and bank settlements must be traceable and reconcilable. No sensitive card data is stored. |
| R-27 | Issued historical invoices are immutable and numbers controlled. Sale values, applicable taxes and legal-entity details must be snapshotted. Never recalculate old financial facts from current catalogue/tax/entity configuration. |
| R-28 | Subsequent policy/configuration changes do not rewrite earlier commitments. Corrections/refunds must preserve original evidence; exact documents/tax treatment require CA REVIEW. |
| R-29 | Trial dispatch is not a sale. Support Delivery Challan/appropriate dispatch documentation and actual-sale Tax Invoice records, subject to reviewed format, applicability and timing. |
| R-30 | Privileged price, stock, refund, invoice, assignment and payment-administration actions require attributable audit history. |
| R-31 | Compliance-supporting software does not by itself establish legal/tax compliance. Uncertain treatment must be classified explicitly for review. |
| R-32 | Transactional/service messaging is separate from marketing. A service/purchase phone number does not automatically grant marketing consent. |

## CONFIGURABLE RULE: values and scopes

| Setting | Required flexibility | Unresolved dependency |
| --- | --- | --- |
| Market/material/category availability | Activate/deactivate offerings without duplicating business code or UI | Initial BLR/Silver active and HYD/Gold coming soon are finalized; future eligibility BD-12, BD-13 |
| Service areas and PIN codes | Data-driven selected-market eligibility, with hubs and fulfilment policy distinct | BD-07, BD-08, BD-13 |
| Trial Plans | Support relevant Market, Material, Category, piece-limit, deposit and active-state dimensions without assuming every combination is valid | BD-01, BD-03, BD-12, BD-13 |
| Box/item limits | Change limits without schema changes or frontend literals | BD-03, BD-13; two boxes and three/five pieces are not defaults |
| Deposit amount and applicability | Separate configured offering from actual collection/application/refund evidence | BD-01, CA-01 |
| Cart expiry | Configurable local retention; expired selections never reserve stock | BD-13; seven days is only a suggestion |
| Reservation timeouts | Configurable duration and explicit accepted/released/expired facts | BD-02; exact trigger and late-payment response open |
| Pricing and tax | Support pricing modes and effective-dated tax configuration with immutable transaction snapshots | BD-04, CA-02; no GST percentage or weight formula selected |
| OTP/security and provider settings | Configure reviewed challenge limits, retry/fallback settings and environment-specific credentials | BD-15, LR-04; never put secrets in public frontend configuration |

Changing policy configuration requires authorization and audit evidence. Applicable scope, precedence and effective timing must be explicit when defined; do not invent a policy engine or silently choose a default precedence.

## UNRESOLVED BUSINESS DECISION and review register

The authoritative questions and status are in [DECISIONS.md](DECISIONS.md). This index groups them without supplying answers.

- Deposits and their outcomes: BD-01; accounting/tax: CA-01.
- Booking commitment, reservations, expiry and late payments: BD-02.
- Plan quantities, variants, repeated-category boxes, mixed metals and substitutions: BD-03.
- Price commitment and Gold pricing: BD-04.
- Pending payment, retries, changed selection and physical handover: BD-05; pilot methods: BD-06.
- Scheduling, failed visits and exceptions: BD-07; same-city hub sourcing: BD-08.
- Physical identification, damage, loss and QC: BD-09.
- After-sale returns/exchanges/refunds: BD-10, LR-01, CA-02.
- Business entity and stock ownership: BD-11, LR-02, CA-03.
- Gold operating policy: BD-12; initial setting values: BD-13.
- Canonical-document version control: BD-14 was resolved on 2026-09-23 with the approved documentation-only root repository; remote setup/publication is separate. This does not resolve any commercial or review policy.
- Provider choices and operational integration settings: BD-15.
- Financial/document requirements: CA-02; dispatch/product legal requirements: LR-03.
- Privacy, customer-facing terms, consent, retention, marketing and messaging obligations: LR-04.

Do not turn a proposed state, relationship, default, screen message or test expectation into an answer to these decisions.
