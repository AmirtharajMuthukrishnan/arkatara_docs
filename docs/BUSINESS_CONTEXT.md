# Business context

Status: APPROVED BUSINESS CONTEXT; roadmap documentation is pending owner review\
Updated: 2026-09-19

This is the canonical business narrative for both repositories. Binding rules live in [BUSINESS_RULES.md](BUSINESS_RULES.md); finalized decisions, unresolved questions and source reconciliation live in [DECISIONS.md](DECISIONS.md). Technical proposals do not silently resolve commercial, accounting or legal questions.

## Nature of the business

ARKA TARA is a jewellery commerce business beginning in Bengaluru, India. It provides an attended Try-at-Home experience with internal delivery/sales agents.

> Order many. Try them at home. Buy any.

Customers select several designs, try physical jewellery at home and buy only what they choose after evaluating fit, appearance, skin-tone suitability, comfort and how the real piece compares with its online presentation. They do not need to buy jewellery upfront merely to discover whether it suits them. Whether a separate trial deposit is charged remains a BUSINESS DECISION.

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
| Customers | Guest browsing, local Trial Cart and no required account | Future account policy remains unselected |
| Operations | Internal delivery/sales agents; hub inventory in the chosen city | Additional hubs, agents, cities and volume |
| Payment integration | Razorpay is the intended provider, implemented later in Task 8 | Exact live-pilot collection policy and gateway flow remain review items |
| Messaging | WhatsApp Cloud API and SMS planned in Task 9 | Business accounts, provider selection and reviewed templates required |

The UI should show Bengaluru and Silver as available and Hyderabad and Gold as coming soon. Coming-soon options are not bookable.

Hyderabad activation should primarily mean activating its market, configuring hubs and service areas, onboarding inventory and Trial Plans, and assigning staff. Gold activation should primarily mean activating its material, configuring purities, products, pricing and inventory. Neither activation alone should require schema redesign or duplicate storefront development.

## Booking and home visit

One TrialBooking normally represents one customer home visit. It may contain multiple category-specific TrialBoxes fulfilled together from the selected city's operational inventory.

Example: one Ring Trial Box and one Chain Trial Box belong to one booking, customer and address; they travel together and are handled during the same visit.

Multiple boxes do not imply multiple independent checkouts. They do not decide the number of deposits, purchases or payment attempts either. Box composition and those financial policies remain explicit decisions.

"Normally one visit" is deliberate: the business has not approved split visits or retry/rescheduling rules. Whether stock can be combined from multiple hubs within one city also remains unresolved. There is no operating assumption of intercity fulfilment.

## Normal operating journey

1. The customer browses the selected city's available catalogue without logging in.
2. The customer builds category-specific Trial Boxes. Local browser persistence preserves temporary selections; it does not reserve stock.
3. A booking attempt supplies minimal customer/contact/address information. The backend revalidates eligibility, serviceability, prices, plan rules and inventory.
4. Booking acceptance, deposit requirements and reservation timing follow rules still to be decided. They must not be inferred from this narrative.
5. Operations prepares and dispatches jewellery with appropriate movement documentation.
6. An assigned, authenticated agent attends the visit. The customer receives an Arrival OTP; the agent enters it and backend verification starts the in-person trial.
7. The customer tries pieces and selects what to purchase. The agent records the selection and the backend calculates the payable amount. There is no second selection/purchase OTP.
8. Verified payment evidence determines the financial outcome. Pending-payment handover and exception policies remain open.
9. Purchased stock becomes SOLD as part of a financially confirmed purchase. Unpurchased stock returns to the hub and passes QC before becoming AVAILABLE.

A visit's progress, a purchase, payment, invoice issuance and return/QC completion are separate facts. Completing one does not prove the others complete.

## Operating and funding philosophy

> Simple enough to launch now, structured enough to scale later.

Prefer simple implementation, a strong relational business model, clear domain boundaries, configurable rules, useful tests and durable documentation.

Use Django Admin for the initial back office; use a dedicated mobile staff interface at the customer's home. Keep the operation understandable to the founder, delivery staff and future engineers.

Funding readiness means reliable records and operations: controlled access, traceable inventory, clean financial evidence, audit history, reproducible deployments, separate environments, protected secrets and measurable performance. It does not require microservices, Kubernetes, event streaming or premature distributed systems.

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
