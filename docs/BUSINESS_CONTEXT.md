# Business context

Status: APPROVED BUSINESS CONTEXT; D-23–D-26 update identity, booking and model-first sequencing; implementation and policy gates remain explicit\
Updated: 2026-10-18

This is the canonical business narrative for both repositories. Binding rules live in [BUSINESS_RULES.md](BUSINESS_RULES.md); finalized decisions, unresolved questions and source reconciliation live in [DECISIONS.md](DECISIONS.md). Technical proposals do not silently resolve commercial, accounting or legal questions.

## Nature of the business

ARKA TARA is a jewellery marketplace beginning in Bengaluru, India. It provides an attended Try-at-Home experience, with the first marketplace commercial decisions approved under [D-29](DECISIONS.md#d-29--marketplace-sellers-platform-funded-purchase-benefit-and-combined-visits).

Vendors remain the jewellery sellers/invoice issuers; Arka Tara facilitates the visit and may generate documents in the vendor's name under appropriate authorization. Arka Tara receives the upfront trial amount and funds the promised purchase benefit while preserving the vendor's agreed sale consideration. One combined visit may contain jewellery from several vendors, with separate seller-specific sale, invoice and settlement evidence. Arka Tara never owns jewellery; hub custody does not establish platform ownership. The father's existing Abimannan Jewellers is the prospective first vendor; its legal invoice particulars and piece-title evidence remain onboarding dependencies.

The owner plans to create Arkatara Private Limited, but incorporation has not started. No earlier Arka Tara entity, marketplace or platform has operated; this is the first implementation of the idea. The father's business exists independently. These owner-reported facts do not establish a registered company/name, completed vendor onboarding or permission to erase development/migration history.

This 2026-10-18 checkpoint selects commercial responsibility, not the charge's tax classification, benefit-allocation formula, commission, payout amount or an approved provider arrangement. Remaining account/contact/OTP and exception flows still need explicit reconciliation before their dependent refactor; the older journey and implementation checkpoints below do not constitute the completed marketplace flow.

The subsequent [D-30 commercial checkpoint](DECISIONS.md#d-30--vendor-commission-and-selected-upfront-refund-outcomes) selects percentage commission/base without a rate, specified cancellation/failure refunds and a purchase-capped benefit with refund of unused upfront money. These choices refine the earlier open questions; exact amounts, accounting treatment, detailed exceptions, after-sale outcomes and operational activation remain pending.

[D-31](DECISIONS.md#d-31--booking-time-item-price-commitment-and-promotion-combinations) locks both fixed and rate-based jewellery item prices at confirmed booking, with vendors accepting that exposure. The customer still chooses pieces after trying them. Vendors authorize the best eligible single offer per item; one eligible Arka-funded coupon may combine with that offer and the upfront benefit within approved limits while preserving vendor consideration. Exact quote validity, exceptions, eligibility/allocation and accounting treatment remain pending; Gold is not activated by this direction.

[D-32](DECISIONS.md#d-32--original-visit-price-protection-customer-rescheduling-and-delegated-publication) records committed-price protection and controlled staff publication under vendor delegation. Its customer-rescheduling flow was subsequently superseded by [D-33](DECISIONS.md#d-33--immediate-trial-orders-and-configurable-weighted-upfront-pricing): the product accepts immediate trial orders only, without future appointment slots or rescheduling. Platform/vendor-caused delay still requires honoring the commitment or the appropriate cancellation/refund remedy. Actual mandates, fulfilment limits and financial/professional treatment remain to be established.

The upfront calculation combines per-piece bases with weighted cart/behaviour/demand metrics. New metrics and changing weights must be supported through attributable policy versions, preserving the evidence behind each quote. Exact arithmetic, bases/coefficients, limits and whether to show a cart estimate or only a checkout quote remain open under D-33. Frequent optimization does not reprice an already accepted payment or reserve jewellery merely because it is in a cart.

> Order many. Try them at home. Buy any.

Customers select several designs, try physical jewellery at home and buy only what they choose after evaluating fit, appearance, skin-tone suitability, comfort and how the real piece compares with its online presentation. They do not need to buy the jewellery upfront. D-25 requires phone verification followed by upfront booking collection before confirmation; its amount, basis, deposit/fee classification and application/refund/retention remain BUSINESS DECISION and CA/LEGAL REVIEW items.

This is intended to become a durable operating company, not remain an experimental website. The pilot must validate actual customer behaviour and unit economics before unnecessary complexity is introduced.

## Customers and brand

The primary audience is young earning professionals in large Indian cities, especially technology-heavy cities such as Bengaluru and Hyderabad.

The brand should feel modern, premium, trustworthy, simple, technology-enabled and convenient. The storefront should resemble a premium catalogue and avoid a registration wall or the feel of a traditional jewellery shop website.

Mobile use is central because Instagram/Facebook acquisition is expected to matter. Navigation, product media, serviceability, trial selection and checkout should remain understandable on a phone.

## Initial scope and future direction

| Dimension | Initial position | Future direction |
| --- | --- | --- |
| Market | BLR / Bengaluru / ACTIVE | HYD / Hyderabad / COMING_SOON, expected next operating city |
| Material | SILVER / ACTIVE | GOLD / COMING_SOON, intended relatively soon after validating Silver |
| Purity | S925 for the initial Silver offering | Separate from material; 14K, 18K and 22K are Gold examples, not an approved exclusive range |
| Categories | Primarily Rings, Chains, Bracelets and Kadas | Other categories may be added; Earring and Stud are roadmap examples |
| Customers | Guest browsing and local Trial Cart; booking phone OTP, no required account | Separate optional CustomerAccount support prepared now, disabled initially; later activation after readiness review, with guest checkout permanently retained |
| Operations | Internal delivery/sales agents; hub inventory in the chosen city | Additional hubs, agents, cities and volume |
| Payment integration | Razorpay is the intended provider, implemented later in Task 8 | Exact live-pilot collection policy and gateway flow remain review items |
| Messaging | WhatsApp Cloud API and SMS planned in Task 9 | Business accounts, provider selection and reviewed templates required |

The UI should show Bengaluru and Silver as available and Hyderabad and Gold as coming soon. Coming-soon options are not bookable.

Hyderabad activation should primarily mean activating its market, configuring hubs and service areas, onboarding inventory and Trial Plans, and assigning staff. Gold activation should primarily mean activating its material, configuring purities, products, pricing and inventory. Neither activation alone should require schema redesign or duplicate storefront development.

## Booking and home visit

One TrialBooking normally represents one customer home visit. It may contain multiple category-specific TrialBoxes fulfilled together from the selected city's operational inventory.

Example: one Ring Trial Box and one Chain Trial Box belong to one booking, customer and address; they travel together and are handled during the same visit.

Multiple boxes do not imply multiple independent checkouts. They do not decide the number of deposits, purchases or payment attempts either. Box composition and those financial policies remain explicit decisions.

Under D-33 this is an immediate-order home trial, not an appointment for a later date; there are no future slots or rescheduling options. The internal booking record still distinguishes intent, payment and acceptance. Exceptional failed/repeat or split visits remain unapproved. Whether stock can be combined from multiple hubs within one city remains unresolved, with no implicit intercity fulfilment.

## Normal operating journey

1. The customer browses the selected city's available catalogue without logging in.
2. The customer builds category-specific Trial Boxes. Local browser persistence preserves temporary selections; it does not reserve stock.
3. A pending booking attempt supplies minimal name/mobile/address/PIN information. The backend revalidates eligibility, serviceability, prices, plan rules and inventory. Verify a booking-specific phone OTP and record applicable details/terms before initiating upfront payment.
4. Under D-27, the explicit payment-start action acquires temporary inventory holds, after booking OTP; draft/OTP actions hold nothing. Confirm only after trusted captured-payment evidence and valid booking/inventory acceptance. Amount/classification, hold duration and late-payment/refund outcomes remain gated; payment alone cannot revive expired stock commitments. This booking OTP is distinct from Arrival OTP.
5. Operations prepares and dispatches jewellery with appropriate movement documentation.
6. An assigned, authenticated agent attends the visit. The customer receives an Arrival OTP; the agent enters it and backend verification starts the in-person trial.
7. The customer tries pieces and selects what to purchase. The agent records the selection and the backend calculates the payable amount. There is no second selection/purchase OTP.
8. Verified payment evidence determines the financial outcome. Pending-payment handover and exception policies remain open.
9. Purchased stock becomes SOLD as part of a financially confirmed purchase. Unpurchased stock returns to the hub and passes QC before becoming AVAILABLE.

A visit's progress, a purchase, payment, invoice issuance and return/QC completion are separate facts. Completing one does not prove the others complete.

## Identity and guest privacy

D-27 makes guest booking tracking mandatory through a secure anonymous browser session with individual booking grants. Several bookings can remain accessible after refresh without creating a customer account. Loss/expiry of browser access uses a booking-specific recovery link plus separate fresh verification; it never turns a phone number into a history lookup. The 2026-10-04 Task 4 implementation supplies this persistence and checkout/payment evidence, not working HTTP sessions or gateway integrations.

Staff use separately provisioned StaffUser identities, Django Admin and a dedicated staff portal/API. Registered customers use a separate CustomerAccount and authentication boundary; no shared customer/staff privilege table is planned. The requested staff host is `staff.arkatara.com` with `/staff/api/v1/`; public `arkatara.in` remains the intended storefront domain. Ownership, aliases and deployment/cookie topology must be verified before launch (D-23).

Guest and account checkout share one booking model with historical contact/address snapshots. A guest booking remains permanently unlinked to any registered account, even if its phone matches or the buyer is signed in. No later claim/import flow is planned. Guest tracking is booking-specific; registered history contains only account-owned bookings. Recycled numbers are not permanent identity, and optional customer authentication needs a reviewed recovery design before activation.

Guest means no platform account, not anonymous to the business. Names, phones and addresses remain protected personal data. OTP proves current channel control, not KYC or automatic legal responsibility; payer, requester and recipient are not necessarily the same person.

## Operating and funding philosophy

> Simple enough to launch now, structured enough to scale later.

Prefer simple implementation, a strong relational business model, clear domain boundaries, configurable rules, useful tests and durable documentation.

Use Django Admin for the initial back office; use a dedicated mobile staff interface at the customer's home. Keep the operation understandable to the founder, delivery staff and future engineers.

Funding readiness means reliable records and operations: controlled access, traceable inventory, clean financial evidence, audit history, reproducible deployments, separate environments, protected secrets and measurable performance. It does not require microservices, Kubernetes, event streaming or premature distributed systems.

Older guest bookings remain valid business records when accounts or a mobile app launch; do not replace the transactional database or discard history for that change. Authorized aggregate reporting must distinguish bookings, sales, upfront receipts, refunds and settlements. Phone counts are not proven unique-customer counts. No software schema guarantees funding or legal compliance.

Under D-26, Task 4 completes models, migrations and model integrity tests across backend apps for the agreed roadmap. Task 5 inherits original storefront 4.1–4.7 plus cart work; Tasks 6–10 implement the flows and integrations. Model completeness is not production readiness, and schema-changing unresolved questions must be exposed before Task 4 is declared complete.

The future legal structure may differ from the initial one. Records must continue to identify the entity that actually owned stock, received funds or issued documents at the relevant time. Entity selection and transition arrangements remain BUSINESS DECISION, CA REVIEW and LEGAL REVIEW items.

## Pilot learning and unit economics

Measure acquisition and the full trial-to-purchase journey, including zero-purchase visits. Evaluate contribution after delivery, returned-item handling, payment/refund costs and inventory tied up in trials; revenue alone does not describe this model.

The roadmap includes conversion, AOV, pieces tried versus purchased, category performance, refund speed, agent use, inventory utilisation and turnaround. Their detailed event/KPI backlog is in [TASKS.md](TASKS.md). Metric formulas, costs and attribution should be documented when implemented, not reconstructed from mutable catalogue data.

## Reading map

- [BUSINESS_RULES.md](BUSINESS_RULES.md): finalized invariants, configurable settings and unresolved rules.
- [ARCHITECTURE.md](ARCHITECTURE.md): proposed application structure and module boundaries.
- [DATA_MODEL.md](DATA_MODEL.md): proposed entities, relationships, constraints and historical evidence.
- [STATE_MACHINES.md](STATE_MACHINES.md): proposed transitions and decision-dependent guards.
- [COMPLIANCE_AND_FINANCE.md](COMPLIANCE_AND_FINANCE.md): financial requirements and external review.
- [INTEGRATIONS.md](INTEGRATIONS.md): later provider integrations and boundaries.
- [TASKS.md](TASKS.md): the complete ten-task implementation backlog.
- [DECISIONS.md](DECISIONS.md): authority, finalized decisions, open questions and consolidation record.
