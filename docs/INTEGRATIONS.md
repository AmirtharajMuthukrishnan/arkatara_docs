# Planned integrations

Status: TASK 3 PRIVATE MEDIA BOUNDARY IMPLEMENTED LOCALLY; external providers remain unconnected and subject to review\
Updated: 2026-10-03; external planning references last checked: 2026-09-19 (not revalidated by this scope update)

Use the [architecture](ARCHITECTURE.md), [finance requirements](COMPLIANCE_AND_FINANCE.md), [state machines](STATE_MACHINES.md) and [decision log](DECISIONS.md) together. Tasks 8 and 9 deliberately follow core business logic.

## Boundaries and sequencing

| Boundary | Responsibility | Must not own |
| --- | --- | --- |
| PaymentProvider | Create/fetch a collection context, normalize verified payment/refund evidence and expose provider references and settlement data as supported | Jewellery prices, deposit policy, business tax treatment, stock disposition or invoice mutation |
| NotificationProvider | Accept a business notification intent; route reviewed templates and track delivery attempts/status | Whether a booking/payment is actually successful |
| SMSProvider | Map authorized messages and registered metadata to the selected SMS service | Independent OTP generation or commercial policy |
| WhatsApp adapter | Map eligible message intents/templates and authenticated callbacks to Cloud API | Customer consent assumptions or purchase completion |
| OTP/domain boundary | Purpose-bound booking phone verification, Arrival OTP and future customer authentication challenges with hashing, expiry, atomic consumption and abuse controls | Reusing proof across purposes, choosing legal liability/consent, or treating message delivery as verification |
| ObjectStorage boundary | Store/retrieve media and controlled documents by durable object reference | Publicly exposing financial documents or embedding storage URLs as business identity |
| Optional EmailProvider | Send authorized email notifications if business need is confirmed | Requiring an email address for the guest pilot without a decision |
| Analytics/monitoring adapters | Record approved events and operational health | Financial source of truth or unrestricted copies of customer data |

Names are conceptual, not generated classes or a mandate for an abstract framework. Keep provider-specific payloads and credentials outside business workflows. Use development fakes in early tasks; fakes must not operate as production collection or customer-message services.

Task 1 excludes payment/messaging integration. Task 4 now supplies all required persistence, including durable work items, challenge evidence, provider attempts and settlement records. Task 6 implements booking phone verification, customer authentication/access (customer rollout disabled), provider contracts and development fakes; Task 7 implements staff authentication and arrival verification; Task 8 integrates Razorpay; Task 9 adds WhatsApp/SMS. Task 3's storage boundary remains; Task 5 completes public media delivery using Task 4 models. No live flow may rely on fakes.

## Authentication boundaries

D-23 keeps StaffUser as Django AUTH_USER_MODEL and CustomerAccount separate. Choose reviewed authentication components and record persistence requirements in 4.M1; this decision does not select JWT, Cognito or cookie sessions. Customer credentials must be rejected by `/staff/api/v1/`; staff endpoints also require action and assignment scope. Customer APIs require customer identity and ownership. Staff MFA is required before live access.

The requested staff host `staff.arkatara.com` differs from the public `arkatara.in` domain. Confirm ownership, aliases, cookie scope, CSRF/CORS and any proxy routing before deployment. Do not share broadly scoped credentials or infer identity isolation from DNS. Test actual origins and selected transport. Guest booking grants are scoped/revocable and never become account-history claims. Registered phone recovery must address number reassignment before customer activation.

## Razorpay — Task 8

### Prerequisites

The roadmap requires finalized business/legal identity, a business current account, applicable GST registration, CA-approved accounting/deposit policy, Razorpay business account/KYC and a settlement bank account. Keep test/live accounts, secrets and callbacks separate.

BUSINESS DECISION BD-06 remains necessary for any real pilot collection before the planned gateway integration. Development demonstrations are not permission to take unverified live payments.

### One collection context, two access paths

The backend calculates the payable amount and establishes one PaymentAttempt for that final amount. The agent's QR and the URL sent to the customer must resolve to that same collection context.

A candidate technical option is a QR that encodes the very same hosted payment URL. This is a proposal, not a chosen gateway product. A provider-native QR and a separate Payment Link must not be assumed to share one payable resource merely because the application labels both with the same booking ID.

BUSINESS DECISION BD-15 includes validating the selected Razorpay product flow, amount locking, expiry, completion and retry behaviour. Test that using one channel then opening/scanning the other cannot independently collect the same obligation again. The provider's Payment Links offering describes sharing links through customer channels, but that does not prove every QR/API combination meets this invariant. [Razorpay Payment Links](https://razorpay.com/payment-links/).

Do not mutate an existing collection amount silently when selection changes. Retain payment history and apply the future BD-05 policy for stale requests, cancellations and retries.

### Verification, failure handling and evidence

- Authenticate provider callbacks/signatures using the chosen product's current official mechanism and the original signed request data.
- Correlate account/environment, provider reference, intended purpose, amount and currency before accepting financial evidence.
- Process repeated or out-of-order events idempotently and preserve their receipt/processing outcome. A duplicate delivery is not a second transaction.
- Recover uncertain network outcomes by fetching/reconciling provider evidence rather than blind creation of another charge.
- Separate provider acceptance, authorized/captured or otherwise verified payable outcome, refund progress and settlement status according to the selected product.
- Record enough references and safe payload evidence for reconciliation and support. Keep secrets/card data out of storage, source control and browser bundles.
- Restrict payment/refund administrative operations and audit them. Client success callbacks can inform UX while the backend verifies.

Razorpay's guidance covers secret protection, backend payment evidence and webhook authentication. Its webhook validation documentation also describes duplicate and potentially out-of-order events. Recheck the applicable current product contract during implementation; no SDK version or endpoint is selected here. [Security checklist](https://security.razorpay.com/security/checklist/), [Razorpay webhook validation documentation](https://d6xcmfyh68wv8.cloudfront.net/docs/webhooks/validate-test/).

Tests must cover invalid signatures, wrong amount/currency/account, duplicate events, late success, stale selection, simultaneous QR/link use, ambiguous creation/refund timeouts and settlement mismatches. No provider-specific paid state may bypass the business guards.

## WhatsApp Cloud API — Task 9

### Business/account readiness

Prepare the Meta Business Portfolio, WhatsApp Business Account, dedicated business phone number, Meta developer application, appropriate permissions and approved message templates where required. Meta's official collection identifies the portfolio/account/phone assets and webhook subscription flow. [Meta Cloud API collection](https://www.postman.com/meta/whatsapp-business-platform/documentation/wlk6lh4/whatsapp-cloud-api).

Keep access tokens and app secrets on the backend; configure rotation, restricted staff access and environment isolation. Verify webhook subscriptions and authenticate event callbacks before updating notification status.

### Templates and purposes

The roadmap requires messages for:

- Booking phone OTP before upfront payment.
- Booking confirmed.
- Out for delivery.
- Arrival OTP.
- Payment link.
- Payment success.
- Deposit refund.
- Invoice issued.
- Cancellation.

Optional customer signup/login/recovery messages require separate purposes and reviewed templates before account activation; they remain disabled with the customer feature. Booking/Arrival OTPs are not reusable customer authentication messages.

Keep event intent separate from template language/version and provider approval/category. A notification should be sent only from the corresponding authoritative business event. For example, sending a payment link is not payment confirmation.

The current WhatsApp Business policy requires opt-in permission for contact and approved templates for business-initiated conversations. Template approval does not decide local legal consent obligations. LEGAL REVIEW LR-04 must confirm the project's notices, channel eligibility and service/marketing treatment; exact template wording is not finalized. [WhatsApp Business policy](https://whatsappbusiness.com/policy/).

Delivery/read callbacks describe message delivery only. They do not verify arrival, accept jewellery selection or prove financial receipt.

## SMS and Indian DLT/TRAI — Task 9

Select a compatible SMS provider behind SMSProvider. Roadmap prerequisites include relevant DLT/Principal Entity registration, approved sender/header and message templates.

TRAI's sender guidance identifies Principal Entity registration, headers, content templates and transmission of their identifiers for bulk communications, with consent requirements depending on the communication. LEGAL REVIEW LR-04 and the selected provider must confirm current applicability, classification, routing and approvals for this business; a provider account alone is not proof of readiness. [TRAI advice to senders](https://www.trai.gov.in/advice-to-senders).

Store required registration/template identifiers as protected operational configuration. Keep approved fixed/variable template structure and provider result codes traceable. Do not silently repurpose transactional templates for marketing.

## Shared Arrival OTP and payment link

Booking phone verification is a separate pre-payment step under D-25, bound to the pending booking, phone and purpose. Task 6 owns its verification semantics and Task 9 delivers its messages. The approved dual-channel requirement below is specifically for Arrival OTP; booking/login channel and fallback settings remain BD-15/LR-04. No purchase-selection OTP is added.

The delivery domain creates one arrival challenge, bound to the appropriate booking/visit. WhatsApp and SMS receive the same code from that challenge, not separately generated codes.

Securely retain verification material, expiry, attempt counters and verification timestamp. Avoid plaintext OTPs in long-lived storage or logs. If asynchronous sending requires short-lived sensitive payload retention, protect and expire it; the precise mechanism is a technical proposal to review, not a new commercial policy.

Channel retry does not itself rotate the challenge. Any explicit resend/rotation policy must invalidate or preserve prior challenges consistently under BD-15. Message failure cannot bypass OTP verification or mark the customer arrived. Agent authorization and booking state are checked on the backend.

Send the same final payment URL over eligible customer channels; display a QR for the same PaymentAttempt. A receipt/invoice notification is emitted from verified business/financial facts, not a click, scan or delivery receipt.

The planned dual-channel arrival delivery is an approved requirement. If a channel is unavailable or the customer is ineligible to receive it, failure/fallback handling must follow BD-15/LR-04 rather than being invented.

## Object storage — Task 3

Support main/additional images, video and thumbnails through durable media metadata and provider-neutral object references. D-22 selects S3/CloudFront as the target storage/delivery direction. Type/size limits, region/account/access configuration and reviewed publication/processing remain implementation and BD-15 gates.

Financial/dispatch documents and customer-related assets need access rules distinct from public catalogue media. Preserve issued document versions and references; changing a product image must not rewrite financial history.

Separate environments/buckets, least-privilege credentials and backup/retention controls must be planned. D-22's target choice does not provision buckets, CDN distributions or provider accounts; actual configuration and verification remain outstanding.

Task 3 implements `ObjectStorage` with `put/open/delete` operations and a local/test-only private adapter. Settings default to `PRODUCT_MEDIA_STORAGE=UNCONFIGURED`, no root, no byte limit and no allowed MIME types. The local adapter requires an explicit absolute dedicated root; staging/production cannot use it. Technical format support currently covers PNG/JPEG/MP4, with required allowlist, byte limit, structural container checks and agreement between detected type, extension and declared MIME. These checks do not decode/transcode media or scan for malware.

Admin uploads require active staff plus media-add/product-view permissions; downloads require media-view/product-view permissions. Assets are private attachments with no-store/nosniff headers. Metadata contains a durable UUID, role, product/source identity, type, size, digest and `delivery.state=PRIVATE`, without an object key or public URL. Metadata edits/retirement are audited; retirement retains the file and original asset identity. Mobile derivatives, public publication and CDN URLs require later delivery work and reviewed settings.

The upload service owns its database transaction and cleans up its own newly created object after a normal database/audit failure. Crash recovery, orphan reconciliation, scanning/transcoding as appropriate, retention and production access controls must be designed before a live adapter is enabled. The backend README records concrete local configuration and Admin usage. BUSINESS DECISION BD-15 and applicable LEGAL REVIEW LR-04 remain open.

## Task 4 public delivery contract in progress

Historical heading retained for links: D-26 moves original 4.1–4.7 execution to Task 5. References to Task 4 in this section describe retained earlier work; current Task 4 supplies persistence only. Existing public/private contracts stay intact.

The public storefront consumes anonymous read-only `/api/v1/catalogue/` and `/api/v1/catalogue/pages/<slug>/` responses through the existing bounded API transport. Server reads do not forward browser credentials, follow API redirects or treat cached client state as authority. Runtime validation checks public page identity, selected market/material, availability and pagination. Missing pages are distinct from API/network failure; collection/product outages propagate to the Next.js server error boundary rather than returning a successful empty catalogue or false 404. The home page can retain meaningful static content and an outage panel without adding noindex solely for feed failure. Invalid selections have a separate recovery state. See [ARCHITECTURE.md](ARCHITECTURE.md#task-4-public-catalogue-and-search-boundaries).

Task 4 does not expose Task 3 storage keys or private download routes. The current backend public media projection is empty. A future approved public image contract contains only public asset UUID, HTTPS URL, factual alt text and positive dimensions. The frontend's `CATALOGUE_MEDIA_ORIGINS` defaults to empty and permits only explicitly configured plain HTTPS origins; matching Next.js image configuration supports responsive delivery. Image URLs with credentials, signed query parameters, private Admin paths or unlisted origins are rejected. Adding an origin is not permission to publish private assets and does not select a production vendor. A live adapter still requires BD-15 settings, appropriate access/retention review and an explicit reviewed publication path.

The current responsive image component reserves layout dimensions and distinguishes the first detail image from lazy-loaded later images, but it cannot turn a private draft into approved public media. Production image processing, video delivery, malware/scanning or transcoding decisions, derivative generation, CDN behavior and actual media performance remain unconnected. Do not treat placeholder illustrations or fixture assets as photographed stock.

Public page metadata uses the intended `https://arkatara.in` identity under D-20. This is a content/URL configuration choice, not DNS, hosting, TLS or search-account provisioning. `SITE_INDEXING_ENABLED=false` keeps the local/prelaunch metadata non-indexable by default. Task 10 owns the coordinated preferred-domain, robots/sitemap, redirect, search-console and deployed performance/indexation checks; no Search Console or Bing account, verification token, submission or analytics vendor has been configured by Task 4.

## Email, analytics and monitoring

Email is optional pending BD-15. If later needed for invoices or support, choose a provider, sender-domain setup and purpose/consent policy then; do not add email as an unapproved mandatory checkout field.

Use provider-neutral event names and stable correlation IDs for analytics. The complete required event, attribution and KPI lists live in Task 10 of [TASKS.md](TASKS.md). Keep client behavioural events distinct from backend-confirmed booking, payment and refund facts so the dashboard cannot turn an unverified browser event into revenue.

Capture source timestamps, event identity and suitable business references, minimizing personal data. UTM/referrer values need validation and privacy-aware retention; do not send raw phone, address, OTP or financial-document contents to analytics.

Monitor API/server errors, payment/webhook/notification failures, inventory inconsistencies and background-job failures. Choose tools, operational alert ownership, retry limits and retention under BD-15/LR-04. Monitoring is not a replacement for audit and reconciliation records.

## Reliability and security across adapters

- Persist the domain action independently of provider delivery success; use a small durable pending-work record where needed rather than requiring an event-streaming platform.
- Retry only operations whose prior outcome and idempotency behaviour are understood. Record attempts and surface failures needing intervention.
- Prevent callbacks from crossing test/live accounts or environments. Reject invalid signatures and unexpected references.
- Do not hold a database stock lock while waiting on slow external network calls. Coordinate committed intentions and later outcomes explicitly.
- Restrict credentials and sensitive payloads, use HTTPS and operational rotation, and sanitize logs.
- Define fixtures/fakes for timeout, duplication, rejection, partial notification failure and asynchronous status changes.
- Treat provider/API/template changes as integration changes to review; do not hard-code stale rules into core domains.

## External reference limits and pending decisions

Sources were consulted on 2026-09-19 for planning. Primary references include Razorpay's product/security documentation, Meta's official collection/policy, and TRAI sender guidance. Some provider reference pages expose older examples; exact current SDK/API contracts and applicable legal rules must be rechecked when the integration task is implemented.

BUSINESS DECISION: BD-05, BD-06 and BD-15.\
CA REVIEW: CA-01, CA-02 and CA-03.\
LEGAL REVIEW: LR-01, LR-02, LR-03 and LR-04 as applicable.

This document creates no external accounts, permissions, templates, messages, payments or subscriptions.
