# All Backend Models: Annotated Reference

Snapshot: 2026-10-03, current working tree (including uncommitted Task 4 work).

**Reference only. Do not import, run, or use this file as a replacement models module.** The actual Django models remain in their owning apps. This document consolidates their current code and comments for reading; it does not change schema, register duplicate models, apply migrations or complete Task 4. It is a dated source snapshot, not a second independently maintained schema specification.

There are **27 project-defined concrete Django models across seven apps**. All eleven domain apps are accounted for below. Shared abstract models/managers and the existing billing snapshot value object are included separately; they are not additional domain tables. Django's inherited staff fields and framework-owned tables are identified explicitly.

The current order record is **TrialBooking**. It remains **DRAFT-only**: the accepted-booking lifecycle, TrialPlan/items, reservation, purchases, payment/invoice and related models are not all implemented. Proposed future models are listed as pending, never presented as existing code.

## Reading Map

- [accounts](#app-accounts): [StaffUser](#model-accounts-staffuser), [CustomerAccount](#model-accounts-customeraccount).
- [trials](#app-trials): [TrialBooking](#model-trials-trialbooking), [TrialBox](#model-trials-trialbox), [BookingPhoneChallenge](#model-trials-bookingphonechallenge), [GuestBookingAccessGrant](#model-trials-guestbookingaccessgrant), [BookingPolicyAcceptance](#model-trials-bookingpolicyacceptance).
- [delivery](#app-delivery): [DeliveryAssignment](#model-delivery-deliveryassignment), [ArrivalOTP](#model-delivery-arrivalotp).
- [markets](#app-markets): [Market](#model-markets-market), [Hub](#model-markets-hub), [ServiceArea](#model-markets-servicearea).
- [catalog](#app-catalog): [Material](#model-catalog-material), [Purity](#model-catalog-purity), [Category](#model-catalog-category), [Product](#model-catalog-product), [ProductVariant](#model-catalog-productvariant), [CSVImportBatch](#model-catalog-csvimportbatch), [ProductMedia](#model-catalog-productmedia), [PriceRevision](#model-catalog-pricerevision), [StorefrontPage](#model-catalog-storefrontpage).
- [inventory](#app-inventory): [InventoryUnit](#model-inventory-inventoryunit), [InventoryMovement](#model-inventory-inventorymovement).
- [compliance](#app-compliance): [PolicyDocumentRevision](#model-compliance-policydocumentrevision), [LegalEntity](#model-compliance-legalentity), [BusinessConfiguration](#model-compliance-businessconfiguration), [AuditEvent](#model-compliance-auditevent).
- [Pending apps](#pending-apps): billing, payments, notifications and analytics.
- [Shared bases and guards](#shared-bases-and-guards): inherited model fields, validation and retained evidence.
- [Billing value object](#billing-value-object): HistoricalPriceSnapshot, not a database table.
- [Database safeguards](#database-safeguards): migration protections that are not visible in model source alone.
- [Framework tables](#framework-tables): Django Admin, auth, content types, sessions and generated join tables.

## How to Read Fields and Relationships

- Field inventories come from the loaded Django model registry, including inherited fields. Reverse relation accessors are not duplicate stored fields. Source blocks preserve the actual definitions, methods, Meta constraints, imports and comments.
- `FK -> Model` means many rows can refer to one parent; `OneToOne` adds uniqueness. `PROTECT` prevents ordinary Django deletion of a referenced parent; SQL history guarantees require the relevant constraints/triggers too. `ManyToMany` uses a generated join table, not a scalar column.
- Nullability is a database storage property. `blank` is a validation/form property, not an authentication or authorization rule. A nullable field may still be conditionally required by checks or `clean()`.
- Defaults shown below are the actual field defaults, not proposed launch settings. `No declared default` does not mean the database rejects every omitted field: generated keys/timestamps and Django empty-value behavior still apply.
- `public_id` is an opaque UUID identifier, never an access token. Internal numeric IDs are separate. AuditEvent uses `event_id` as its UUID primary key.
- Field `choices` alone are not SQL check constraints. Explicit Meta constraint names and indexes are listed; automatic primary/unique/FK indexes and migration SQL guards must also be considered.
- Model validation, database integrity and workflow authorization are different layers. A valid row is not proof of payment, verified phone possession, consent, stock custody or permission to access it. Never expose complete model fields automatically through a public serializer.
- References such as `BUSINESS_RULES.md R-34` in source comments resolve to the canonical [business rules](BUSINESS_RULES.md) and [decision register](DECISIONS.md). [DATA_MODEL.md](DATA_MODEL.md) remains the canonical target/implementation distinction, with [Task 4 status](TASKS.md#4-cross-app-model-completion).

<a id="app-accounts"></a>

## accounts

<a id="model-accounts-staffuser"></a>

### StaffUser

**Table:** `accounts_staffuser`. **Source:** [accounts/models.py](../arkatara_backend/accounts/models.py).

Staff authentication identity and the sole Django AUTH_USER_MODEL. Django Admin and operational staff identities belong here; customers do not.

Most fields come from AbstractUser: password hash, username, active/Admin/superuser flags, group membership and permissions. is_staff grants Admin eligibility, not delivery assignment authority.

Staff MFA, staff-host routing and action/assignment/market/hub authorization are not supplied by this model alone. Preserve staff attribution in historical records.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `password` | CharField(128) | no / required | `No declared default` | Django encoded password or unusable-password marker; never store plaintext. |
| `last_login` | DateTimeField | yes / allowed | `No declared default` | Framework login observation, not proof of an active session. |
| `is_superuser` | BooleanField | no / required | `False` | Django permission bypass flag for trusted superusers, never a customer/agent role. |
| `username` | CharField(150); unique | no / required | `No declared default` | Django staff login identifier; not a customer booking phone. |
| `first_name` | CharField(150) | no / allowed | `No declared default` | Optional staff identity detail inherited from AbstractUser. |
| `last_name` | CharField(150) | no / allowed | `No declared default` | Optional staff identity detail inherited from AbstractUser. |
| `email` | EmailField(254) | no / allowed | `No declared default` | Inherited staff email; the current field is not unique. |
| `is_staff` | BooleanField | no / required | `False` | Django Admin eligibility flag; delivery authorization is separate. |
| `is_active` | BooleanField | no / required | `True` | Framework staff active flag; must be respected by authentication/authorization. |
| `date_joined` | DateTimeField | no / required | `django.utils.timezone.now` | Staff account creation timestamp supplied by Django. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `groups` | ManyToManyField -> auth.Group | n/a / allowed | `No declared default` | Django role/permission groups via a framework-managed join table. |
| `user_permissions` | ManyToManyField -> auth.Permission | n/a / allowed | `No declared default` | Direct Django permissions via a framework-managed join table. |

**Explicit Meta constraints:** None declared directly; inspect field uniqueness, inherited validation and migration safeguards.

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-accounts-customeraccount"></a>

### CustomerAccount

**Table:** `accounts_customeraccount`. **Source:** [accounts/models.py](../arkatara_backend/accounts/models.py).

A separate, opaque customer identity with a mutable display name and lifecycle. It is intentionally not a Django authentication user or a staff profile.

PROVISIONED is the initial state. Closure requires closed_at and cannot reopen or reassign the identity. Retained bookings use a protected FK; guest bookings never obtain that FK.

No phone/email credentials, login principal, signup endpoint, session/recovery support or customer permissions are implemented here. Privacy erasure needs a reviewed process, not arbitrary history deletion.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `display_name` | CharField(200) | no / allowed | `No declared default` | Optional presentation label, not historical recipient identity. |
| `status` | CharField(16) | no / required | `CustomerAccount.Status.PROVISIONED` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `PROVISIONED`, `ACTIVE`, `SUSPENDED`, `CLOSED`. |
| `closed_at` | DateTimeField | yes / allowed | `No declared default` | One-way customer closure evidence; must agree with CLOSED status. |

**Explicit Meta constraints:** `customer_account_status_valid`, `customer_account_closure_valid`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

### Source: accounts/models.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [accounts/models.py](../arkatara_backend/accounts/models.py).

```python
import uuid

from django.contrib.auth.models import AbstractUser
from django.core.exceptions import ValidationError
from django.db import models

from common.evidence import RetainedModel
from common.verification import validate_observed_at


# REFERENCE NOTE (documentation only; not present in application source):
# Staff authentication identity and the sole Django AUTH_USER_MODEL. Django Admin and
# operational staff identities belong here; customers do not.
# Most fields come from AbstractUser: password hash, username, active/Admin/superuser flags,
# group membership and permissions. is_staff grants Admin eligibility, not delivery
# assignment authority.
# Staff MFA, staff-host routing and action/assignment/market/hub authorization are not
# supplied by this model alone. Preserve staff attribution in historical records.
class StaffUser(AbstractUser):
    """Staff identity leaves guest browsing and booking free of a login requirement.

    Assignment permissions belong to the later staff workflow, not this identity alone.
    Registered customers use CustomerAccount, never this permission-bearing table.
    Refs: BUSINESS_RULES.md R-04, R-18, R-33; DECISIONS.md D-23.
    """

    public_id = models.UUIDField(default=uuid.uuid4, unique=True, editable=False)

    class Meta:
        verbose_name = "staff user"
        verbose_name_plural = "staff users"


# REFERENCE NOTE (documentation only; not present in application source):
# A separate, opaque customer identity with a mutable display name and lifecycle. It is
# intentionally not a Django authentication user or a staff profile.
# PROVISIONED is the initial state. Closure requires closed_at and cannot reopen or reassign
# the identity. Retained bookings use a protected FK; guest bookings never obtain that FK.
# No phone/email credentials, login principal, signup endpoint, session/recovery support or
# customer permissions are implemented here. Privacy erasure needs a reviewed process, not
# arbitrary history deletion.
class CustomerAccount(RetainedModel):
    """Stable customer identity, not an authenticated principal or a staff account.

    Contact/credential recovery and login transport require separate design. No
    phone-based lookup, auth property, staff permission or signup is enabled here.
    Refs: BUSINESS_RULES.md R-33, R-34, R-37; DECISIONS.md D-23.
    """

    class Status(models.TextChoices):
        PROVISIONED = "PROVISIONED", "Provisioned / not enabled"
        ACTIVE = "ACTIVE", "Active"
        SUSPENDED = "SUSPENDED", "Suspended"
        CLOSED = "CLOSED", "Closed"

    display_name = models.CharField(max_length=200, blank=True)
    status = models.CharField(max_length=16, choices=Status.choices, default=Status.PROVISIONED)
    closed_at = models.DateTimeField(null=True, blank=True, validators=[validate_observed_at])

    class Meta(RetainedModel.Meta):
        constraints = [
            models.CheckConstraint(
                condition=models.Q(status__in=["PROVISIONED", "ACTIVE", "SUSPENDED", "CLOSED"]),
                name="customer_account_status_valid",
            ),
            models.CheckConstraint(
                condition=models.Q(status="CLOSED", closed_at__isnull=False)
                | (~models.Q(status="CLOSED") & models.Q(closed_at__isnull=True)),
                name="customer_account_closure_valid",
            ),
        ]

    def validate_existing_identity(self, previous):
        super().validate_existing_identity(previous)
        if previous.status == self.Status.CLOSED and (
            self.status != previous.status or self.closed_at != previous.closed_at
        ):
            raise ValidationError("Closed customer identity cannot be reactivated or reassigned.")

    def __str__(self):
        return str(self.public_id)
```

<a id="app-trials"></a>

## trials

<a id="model-trials-trialbooking"></a>

### TrialBooking

**Table:** `trials_trialbooking`. **Source:** [trials/models.py](../arkatara_backend/trials/models.py).

The single booking identity shared by guest and registered-account checkout. It stores the original recipient name/phone/address/country/PIN independently of profile data.

Database checks require GUEST with no account or ACCOUNT with an account. Update guards permanently protect both mode and owner, including an attempted simultaneous mode/FK rewrite. Phone is not unique identity.

Only DRAFT is currently allowed. Original contact facts are already immutable; corrected input means new intent at this stage. Confirmed lifecycle, plan/items, financial/stock evidence and versioned amendments remain unfinished.

request_id is unique request identity and request_fingerprint represents canonical input evidence. These fields support later idempotency logic; no booking submission or OTP/payment workflow is enabled by this model.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `checkout_mode` | CharField(8) | no / required | `No declared default` | Immutable GUEST or ACCOUNT selection; governs whether customer_account must be null. Choices: `GUEST`, `ACCOUNT`. |
| `customer_account` | ForeignKey -> accounts.CustomerAccount; PROTECT | yes / allowed | `No declared default` | Account-mode ownership only. Permanently null for a guest booking. |
| `market` | ForeignKey -> markets.Market; PROTECT | no / required | `No declared default` | Selected market/city reference, not an automatically chosen fulfilment hub. |
| `status` | CharField(16) | no / required | `TrialBooking.Status.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`. |
| `contact_name` | CharField(200) | no / required | `No declared default` | Original recipient name snapshot; not read from a mutable customer profile. |
| `contact_phone` | CharField(16) | no / required | `No declared default` | Original normalized E.164 phone snapshot; no uniqueness or account matching. |
| `address` | TextField | no / required | `No declared default` | Original delivery address text snapshot. Versioned amendments are not implemented. |
| `country_code` | CharField(2) | no / required | `No declared default` | Explicit country identity used by the relevant model's format/consistency validation. |
| `postal_code` | CharField(20) | no / required | `No declared default` | Configured coverage PIN or snapshotted recipient PIN, depending on this model. |
| `request_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Unique request identity for later replay/idempotency handling. |
| `request_fingerprint` | CharField(64) | no / required | `No declared default` | Digest of canonical request facts; it does not itself implement replay handling. |

**Explicit Meta constraints:** `booking_checkout_owner_valid`, `booking_draft_only`, `booking_contact_required`

**Explicit Meta indexes:** `booking_market_state_idx`, `booking_customer_time_idx`

<a id="model-trials-trialbox"></a>

### TrialBox

**Table:** `trials_trialbox`. **Source:** [trials/models.py](../arkatara_backend/trials/models.py).

A category-specific container belonging to one combined TrialBooking, not a separate checkout.

Both booking and category are protected references; the box record is append-only. There is deliberately no unique constraint on booking/category while repeated-category policy is open.

TrialPlan linkage, item/quantity semantics, exact-unit allocation and their snapshots are still pending. A box row alone does not reserve stock.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `recorded_at` | DateTimeField | no / allowed | `auto_now_add` | Database-recorded evidence time, distinct from an observed event time. |
| `booking` | ForeignKey -> trials.TrialBooking; PROTECT | no / required | `No declared default` | Protected parent booking; not a separate checkout or payment. |
| `category` | ForeignKey -> catalog.Category; PROTECT | no / required | `No declared default` | Category identity; a box has exactly one category. |

**Explicit Meta constraints:** None declared directly; inspect field uniqueness, inherited validation and migration safeguards.

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-trials-bookingphonechallenge"></a>

### BookingPhoneChallenge

**Table:** `trials_bookingphonechallenge`. **Source:** [trials/models.py](../arkatara_backend/trials/models.py).

Storage for the phone challenge used before upfront booking payment, purpose-separated from customer authentication and ArrivalOTP by model/table and subject.

booking fixes the subject and phone must match its immutable contact snapshot. Shared VerificationChallenge fields retain the hash, explicit bounds and monotonic verification/consumption evidence.

No OTP generation, sending, secret comparison, rate-limit service or payment initiation exists here. Future workflow must verify and consume under lock; channel control is not identity, KYC or automatic liability.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `phone` | CharField(16) | no / required | `No declared default` | Subject-bound normalized E.164 number; channel possession is not permanent identity. |
| `secret_hash` | CharField(255) | no / required | `No declared default` | Encoded secret hash, never a plaintext OTP or a message body. |
| `issued_at` | DateTimeField | no / required | `django.utils.timezone.now` | Observed credential/challenge creation time. |
| `expires_at` | DateTimeField | no / required | `No declared default` | Explicit expiry boundary, with no approved duration supplied by this schema. |
| `max_attempts` | PositiveSmallIntegerField | no / required | `No declared default` | Explicit positive attempt bound, with no production default. |
| `attempts_used` | PositiveSmallIntegerField | no / required | `0` | Recorded attempt count; cannot decrease or exceed the bound. |
| `verified_at` | DateTimeField | yes / allowed | `No declared default` | Retained verification outcome; setting a timestamp is not the verification service. |
| `consumed_at` | DateTimeField | yes / allowed | `No declared default` | Retained consumption evidence; future business use must be atomically guarded. |
| `invalidated_at` | DateTimeField | yes / allowed | `No declared default` | Retained invalidation evidence; invalidated challenges cannot be reused. |
| `booking` | ForeignKey -> trials.TrialBooking; PROTECT | no / required | `No declared default` | Protected parent booking; not a separate checkout or payment. |

**Explicit Meta constraints:** `trials_bookingphonechallenge_attempts`, `trials_bookingphonechallenge_expiry`, `trials_bookingphonechallenge_verified`, `trials_bookingphonechallenge_consumed`, `trials_bookingphonechallenge_invalidated`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-trials-guestbookingaccessgrant"></a>

### GuestBookingAccessGrant

**Table:** `trials_guestbookingaccessgrant`. **Source:** [trials/models.py](../arkatara_backend/trials/models.py).

A revocable, expiring credential digest scoped to exactly one GUEST booking. It is not a CustomerAccount session or phone-based order-history lookup.

Only the digest is stored; booking/token/issue/expiry binding is immutable and recorded revocation cannot be undone. Account-mode bookings are rejected as grant targets.

Token issuance, hashing protocol, authorization, scopes at the endpoint and recovery remain later implementation. Possession checks must not be replaced by knowing a booking ID or phone.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `booking` | ForeignKey -> trials.TrialBooking; PROTECT | no / required | `No declared default` | Protected parent booking; not a separate checkout or payment. |
| `token_digest` | CharField(64); unique | no / required | `No declared default` | Unique digest of a booking-specific guest credential, not the raw token. |
| `issued_at` | DateTimeField | no / required | `django.utils.timezone.now` | Observed credential/challenge creation time. |
| `expires_at` | DateTimeField | no / required | `No declared default` | Explicit expiry boundary, with no approved duration supplied by this schema. |
| `revoked_at` | DateTimeField | yes / allowed | `No declared default` | One-way revocation of this guest-access credential. |

**Explicit Meta constraints:** `guest_grant_expiry`, `guest_grant_revocation`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-trials-bookingpolicyacceptance"></a>

### BookingPolicyAcceptance

**Table:** `trials_bookingpolicyacceptance`. **Source:** [trials/models.py](../arkatara_backend/trials/models.py).

Append-only evidence that a specific booking accepted a particular policy document revision at a recorded time, with request identity/fingerprint.

It has no CustomerAccount FK: guest policy evidence must not indirectly attach a guest order to registered account history.

This row is not evidence of OTP verification or blanket marketing consent. The future checkout workflow must record genuine acceptance and respect LR-04.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `recorded_at` | DateTimeField | no / allowed | `auto_now_add` | Database-recorded evidence time, distinct from an observed event time. |
| `booking` | ForeignKey -> trials.TrialBooking; PROTECT | no / required | `No declared default` | Protected parent booking; not a separate checkout or payment. |
| `policy_revision` | ForeignKey -> compliance.PolicyDocumentRevision; PROTECT | no / required | `No declared default` | Exact retained policy wording/version accepted by this booking. |
| `accepted_at` | DateTimeField | no / required | `No declared default` | Recorded acceptance observation, distinct from OTP verification and record creation. |
| `request_id` | UUIDField(32); unique | no / required | `No declared default` | Unique request identity for later replay/idempotency handling. |
| `request_fingerprint` | CharField(64) | no / required | `No declared default` | Digest of canonical request facts; it does not itself implement replay handling. |

**Explicit Meta constraints:** None declared directly; inspect field uniqueness, inherited validation and migration safeguards.

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

### Source: trials/models.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [trials/models.py](../arkatara_backend/trials/models.py).

```python
"""Account-free booking identity and evidence; no booking acceptance workflow yet."""

import uuid

from django.core.exceptions import ValidationError
from django.core.validators import RegexValidator
from django.db import models
from django.db.models import F, Q
from django.utils import timezone

from common.evidence import AppendOnlyModel, RetainedModel
from common.verification import (
    VerificationChallenge,
    digest_validator,
    phone_validator,
    validate_observed_at,
)


# REFERENCE NOTE (documentation only; not present in application source):
# The single booking identity shared by guest and registered-account checkout. It stores the
# original recipient name/phone/address/country/PIN independently of profile data.
# Database checks require GUEST with no account or ACCOUNT with an account. Update guards
# permanently protect both mode and owner, including an attempted simultaneous mode/FK
# rewrite. Phone is not unique identity.
# Only DRAFT is currently allowed. Original contact facts are already immutable; corrected
# input means new intent at this stage. Confirmed lifecycle, plan/items, financial/stock
# evidence and versioned amendments remain unfinished.
# request_id is unique request identity and request_fingerprint represents canonical input
# evidence. These fields support later idempotency logic; no booking submission or
# OTP/payment workflow is enabled by this model.
class TrialBooking(RetainedModel):
    """One immutable checkout identity and original recipient snapshot for both modes.

    Only draft intent is writable until the acceptance/payment/stock workflow exists.
    Phone edits require new intent, never rewriting a previously verified snapshot.
    Refs: BUSINESS_RULES.md R-01, R-02, R-34, R-35; DECISIONS.md D-24, D-25.
    """

    class CheckoutMode(models.TextChoices):
        GUEST = "GUEST", "Guest"
        ACCOUNT = "ACCOUNT", "Customer account"

    class Status(models.TextChoices):
        DRAFT = "DRAFT", "Draft / not confirmed"

    checkout_mode = models.CharField(max_length=8, choices=CheckoutMode.choices)
    customer_account = models.ForeignKey(
        "accounts.CustomerAccount",
        null=True,
        blank=True,
        on_delete=models.PROTECT,
        related_name="bookings",
    )
    market = models.ForeignKey("markets.Market", on_delete=models.PROTECT, related_name="bookings")
    status = models.CharField(max_length=16, choices=Status.choices, default=Status.DRAFT)
    contact_name = models.CharField(max_length=200)
    contact_phone = models.CharField(max_length=16, validators=[phone_validator])
    address = models.TextField()
    country_code = models.CharField(
        max_length=2, validators=[RegexValidator(r"\A[A-Z]{2}\Z", "Use a country code.")]
    )
    postal_code = models.CharField(max_length=20)
    request_id = models.UUIDField(default=uuid.uuid4, unique=True, editable=False)
    request_fingerprint = models.CharField(max_length=64, validators=[digest_validator])

    immutable_fields = (
        "public_id",
        "checkout_mode",
        "customer_account_id",
        "market_id",
        "contact_name",
        "contact_phone",
        "address",
        "country_code",
        "postal_code",
        "request_id",
        "request_fingerprint",
    )

    class Meta(RetainedModel.Meta):
        constraints = [
            models.CheckConstraint(
                condition=Q(checkout_mode="GUEST", customer_account__isnull=True)
                | Q(checkout_mode="ACCOUNT", customer_account__isnull=False),
                name="booking_checkout_owner_valid",
            ),
            models.CheckConstraint(condition=Q(status="DRAFT"), name="booking_draft_only"),
            models.CheckConstraint(
                condition=~Q(contact_name="")
                & ~Q(contact_phone="")
                & ~Q(address="")
                & ~Q(country_code="")
                & ~Q(postal_code=""),
                name="booking_contact_required",
            ),
        ]
        indexes = [
            models.Index(
                fields=["market", "status", "created_at"], name="booking_market_state_idx"
            ),
            models.Index(
                fields=["customer_account", "created_at"], name="booking_customer_time_idx"
            ),
        ]

    def clean(self):
        super().clean()
        for field in ("contact_name", "address", "postal_code"):
            value = getattr(self, field)
            if not isinstance(value, str) or not value.strip() or value != value.strip():
                raise ValidationError({field: "Provide nonblank text without outer whitespace."})
        if self.country_code == "IN" and (
            len(self.postal_code) != 6
            or not self.postal_code.isascii()
            or not self.postal_code.isdigit()
            or self.postal_code.startswith("0")
        ):
            raise ValidationError({"postal_code": "Provide a valid six-digit Indian PIN."})
        if self.market_id and self.market.country_code != self.country_code:
            raise ValidationError(
                {"country_code": "The address must use the selected market country."}
            )

    def __str__(self):
        return str(self.public_id)


# REFERENCE NOTE (documentation only; not present in application source):
# A category-specific container belonging to one combined TrialBooking, not a separate
# checkout.
# Both booking and category are protected references; the box record is append-only. There
# is deliberately no unique constraint on booking/category while repeated-category policy is
# open.
# TrialPlan linkage, item/quantity semantics, exact-unit allocation and their snapshots are
# still pending. A box row alone does not reserve stock.
class TrialBox(AppendOnlyModel):
    """Category container, not a separate checkout or an approved plan/stock hold.

    Repeated categories are not prohibited while BD-03 remains open.
    Refs: BUSINESS_RULES.md R-01, R-02; DECISIONS.md BD-03.
    """

    booking = models.ForeignKey(TrialBooking, on_delete=models.PROTECT, related_name="boxes")
    category = models.ForeignKey(
        "catalog.Category", on_delete=models.PROTECT, related_name="trial_boxes"
    )


# REFERENCE NOTE (documentation only; not present in application source):
# Storage for the phone challenge used before upfront booking payment, purpose-separated
# from customer authentication and ArrivalOTP by model/table and subject.
# booking fixes the subject and phone must match its immutable contact snapshot. Shared
# VerificationChallenge fields retain the hash, explicit bounds and monotonic
# verification/consumption evidence.
# No OTP generation, sending, secret comparison, rate-limit service or payment initiation
# exists here. Future workflow must verify and consume under lock; channel control is not
# identity, KYC or automatic liability.
class BookingPhoneChallenge(VerificationChallenge):
    booking = models.ForeignKey(
        TrialBooking, on_delete=models.PROTECT, related_name="phone_challenges"
    )

    immutable_fields = (*VerificationChallenge.immutable_fields, "booking_id")

    def clean(self):
        super().clean()
        if self.booking_id and self.phone != self.booking.contact_phone:
            raise ValidationError({"phone": "Challenge phone must match this booking snapshot."})


# REFERENCE NOTE (documentation only; not present in application source):
# A revocable, expiring credential digest scoped to exactly one GUEST booking. It is not a
# CustomerAccount session or phone-based order-history lookup.
# Only the digest is stored; booking/token/issue/expiry binding is immutable and recorded
# revocation cannot be undone. Account-mode bookings are rejected as grant targets.
# Token issuance, hashing protocol, authorization, scopes at the endpoint and recovery
# remain later implementation. Possession checks must not be replaced by knowing a booking
# ID or phone.
class GuestBookingAccessGrant(RetainedModel):
    """Scoped guest credential digest, never an account-link or login credential.

    Token generation, authorization and recovery remain Task 6 workflows.
    Refs: BUSINESS_RULES.md R-34, R-37; DECISIONS.md D-24.
    """

    booking = models.ForeignKey(TrialBooking, on_delete=models.PROTECT, related_name="guest_grants")
    token_digest = models.CharField(max_length=64, unique=True, validators=[digest_validator])
    issued_at = models.DateTimeField(default=timezone.now, validators=[validate_observed_at])
    expires_at = models.DateTimeField()
    revoked_at = models.DateTimeField(null=True, blank=True, validators=[validate_observed_at])

    immutable_fields = ("public_id", "booking_id", "token_digest", "issued_at", "expires_at")

    class Meta(RetainedModel.Meta):
        constraints = [
            models.CheckConstraint(
                condition=Q(expires_at__gt=F("issued_at")), name="guest_grant_expiry"
            ),
            models.CheckConstraint(
                condition=Q(revoked_at__isnull=True) | Q(revoked_at__gte=F("issued_at")),
                name="guest_grant_revocation",
            ),
        ]

    def clean(self):
        super().clean()
        if self.booking_id and self.booking.checkout_mode != TrialBooking.CheckoutMode.GUEST:
            raise ValidationError({"booking": "Guest access cannot target an account booking."})
        if self.expires_at and timezone.is_naive(self.expires_at):
            raise ValidationError({"expires_at": "Use a timezone-aware expiry."})

    def validate_existing_identity(self, previous):
        super().validate_existing_identity(previous)
        if previous.revoked_at is not None and self.revoked_at != previous.revoked_at:
            raise ValidationError("A revoked credential cannot be restored or rewritten.")


# REFERENCE NOTE (documentation only; not present in application source):
# Append-only evidence that a specific booking accepted a particular policy document
# revision at a recorded time, with request identity/fingerprint.
# It has no CustomerAccount FK: guest policy evidence must not indirectly attach a guest
# order to registered account history.
# This row is not evidence of OTP verification or blanket marketing consent. The future
# checkout workflow must record genuine acceptance and respect LR-04.
class BookingPolicyAcceptance(AppendOnlyModel):
    """Acceptance evidence is booking-scoped, with no indirect guest account link.

    OTP possession is not consent; the checkout workflow supplies actual acceptance.
    Refs: BUSINESS_RULES.md R-32, R-34, R-36; DECISIONS.md LR-04.
    """

    booking = models.ForeignKey(
        TrialBooking, on_delete=models.PROTECT, related_name="policy_acceptances"
    )
    policy_revision = models.ForeignKey(
        "compliance.PolicyDocumentRevision",
        on_delete=models.PROTECT,
        related_name="booking_acceptances",
    )
    accepted_at = models.DateTimeField(validators=[validate_observed_at])
    request_id = models.UUIDField(unique=True)
    request_fingerprint = models.CharField(max_length=64, validators=[digest_validator])
```

<a id="app-delivery"></a>

## delivery

<a id="model-delivery-deliveryassignment"></a>

### DeliveryAssignment

**Table:** `delivery_deliveryassignment`. **Source:** [delivery/models.py](../arkatara_backend/delivery/models.py).

Preserves which StaffUser was assigned to a booking, who assigned them and when. Reassignment ends one row and creates another instead of overwriting the agent.

One open assignment per booking is database-enforced. Identity is immutable, closure needs an end reason, and ended assignments cannot reopen. A delivery agent need not have is_staff Admin access.

Storage does not enforce action/market/hub RBAC or authorize dispatch of the current DRAFT booking. Visits, outcomes, manifests, custody and scheduling policies remain unfinished.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `booking` | ForeignKey -> trials.TrialBooking; PROTECT | no / required | `No declared default` | Protected parent booking; not a separate checkout or payment. |
| `agent` | ForeignKey -> accounts.StaffUser; PROTECT | no / required | `No declared default` | Assigned StaffUser, never CustomerAccount. |
| `assigned_by` | ForeignKey -> accounts.StaffUser; PROTECT | no / required | `No declared default` | Staff identity that made the assignment. |
| `assigned_at` | DateTimeField | no / required | `No declared default` | Observed assignment time, not a promised delivery slot. |
| `ended_at` | DateTimeField | yes / allowed | `No declared default` | End of responsibility for this assignment; null identifies an open assignment. |
| `end_reason` | TextField | no / allowed | `No declared default` | Required nonblank closure/reassignment explanation when ended. |

**Explicit Meta constraints:** `booking_one_open_assignment`, `assignment_end_evidence`

**Explicit Meta indexes:** `agent_assignments_idx`

<a id="model-delivery-arrivalotp"></a>

### ArrivalOTP

**Table:** `delivery_arrivalotp`. **Source:** [delivery/models.py](../arkatara_backend/delivery/models.py).

The arrival-specific challenge belongs to a DeliveryAssignment and matches that assignment's booking phone.

It inherits the same bounded, retained challenge fields as BookingPhoneChallenge but uses a different table and subject. WhatsApp/SMS are delivery channels for one challenge, not separate verification purposes.

No second purchase OTP exists. Backend verification must later authorize the active visit transition; recording this model alone neither starts a trial nor confirms a sale.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `phone` | CharField(16) | no / required | `No declared default` | Subject-bound normalized E.164 number; channel possession is not permanent identity. |
| `secret_hash` | CharField(255) | no / required | `No declared default` | Encoded secret hash, never a plaintext OTP or a message body. |
| `issued_at` | DateTimeField | no / required | `django.utils.timezone.now` | Observed credential/challenge creation time. |
| `expires_at` | DateTimeField | no / required | `No declared default` | Explicit expiry boundary, with no approved duration supplied by this schema. |
| `max_attempts` | PositiveSmallIntegerField | no / required | `No declared default` | Explicit positive attempt bound, with no production default. |
| `attempts_used` | PositiveSmallIntegerField | no / required | `0` | Recorded attempt count; cannot decrease or exceed the bound. |
| `verified_at` | DateTimeField | yes / allowed | `No declared default` | Retained verification outcome; setting a timestamp is not the verification service. |
| `consumed_at` | DateTimeField | yes / allowed | `No declared default` | Retained consumption evidence; future business use must be atomically guarded. |
| `invalidated_at` | DateTimeField | yes / allowed | `No declared default` | Retained invalidation evidence; invalidated challenges cannot be reused. |
| `assignment` | ForeignKey -> delivery.DeliveryAssignment; PROTECT | no / required | `No declared default` | Protected staff assignment that scopes the arrival challenge. |

**Explicit Meta constraints:** `delivery_arrivalotp_attempts`, `delivery_arrivalotp_expiry`, `delivery_arrivalotp_verified`, `delivery_arrivalotp_consumed`, `delivery_arrivalotp_invalidated`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

### Source: delivery/models.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [delivery/models.py](../arkatara_backend/delivery/models.py).

```python
"""Assignment and arrival evidence, without dispatch, payment or return side effects."""

from django.conf import settings
from django.core.exceptions import ValidationError
from django.db import models
from django.db.models import F, Q

from common.evidence import RetainedModel
from common.verification import VerificationChallenge, validate_observed_at


# REFERENCE NOTE (documentation only; not present in application source):
# Preserves which StaffUser was assigned to a booking, who assigned them and when.
# Reassignment ends one row and creates another instead of overwriting the agent.
# One open assignment per booking is database-enforced. Identity is immutable, closure needs
# an end reason, and ended assignments cannot reopen. A delivery agent need not have
# is_staff Admin access.
# Storage does not enforce action/market/hub RBAC or authorize dispatch of the current DRAFT
# booking. Visits, outcomes, manifests, custody and scheduling policies remain unfinished.
class DeliveryAssignment(RetainedModel):
    """Retain who was responsible rather than overwriting an agent foreign key.

    End the current assignment before recording its replacement. This grants no
    permission to dispatch a draft booking, split visits or bypass custody checks.
    Refs: BUSINESS_RULES.md R-02, R-18, R-30, R-38; DECISIONS.md D-24, BD-07.
    """

    booking = models.ForeignKey(
        "trials.TrialBooking", on_delete=models.PROTECT, related_name="assignments"
    )
    agent = models.ForeignKey(
        settings.AUTH_USER_MODEL, on_delete=models.PROTECT, related_name="delivery_assignments"
    )
    assigned_by = models.ForeignKey(
        settings.AUTH_USER_MODEL, on_delete=models.PROTECT, related_name="made_delivery_assignments"
    )
    assigned_at = models.DateTimeField(validators=[validate_observed_at])
    ended_at = models.DateTimeField(null=True, blank=True, validators=[validate_observed_at])
    end_reason = models.TextField(blank=True)

    immutable_fields = ("public_id", "booking_id", "agent_id", "assigned_by_id", "assigned_at")

    class Meta(RetainedModel.Meta):
        constraints = [
            models.UniqueConstraint(
                fields=["booking"],
                condition=Q(ended_at__isnull=True),
                name="booking_one_open_assignment",
            ),
            models.CheckConstraint(
                condition=Q(ended_at__isnull=True, end_reason="")
                | (
                    Q(ended_at__gte=F("assigned_at"))
                    & Q(ended_at__isnull=False)
                    & ~Q(end_reason="")
                ),
                name="assignment_end_evidence",
            ),
        ]
        indexes = [
            models.Index(fields=["agent", "ended_at", "assigned_at"], name="agent_assignments_idx")
        ]

    def clean(self):
        super().clean()
        if self.ended_at and not self.end_reason.strip():
            raise ValidationError({"end_reason": "Explain the end of this assignment."})
        if self._state.adding:
            for field in ("agent", "assigned_by"):
                if getattr(self, f"{field}_id") and not getattr(self, field).is_active:
                    raise ValidationError(
                        {field: "New assignments require active staff identities."}
                    )

    def validate_existing_identity(self, previous):
        super().validate_existing_identity(previous)
        if previous.ended_at is not None and (
            self.ended_at != previous.ended_at or self.end_reason != previous.end_reason
        ):
            raise ValidationError("Ended assignments cannot be reopened or rewritten.")


# REFERENCE NOTE (documentation only; not present in application source):
# The arrival-specific challenge belongs to a DeliveryAssignment and matches that
# assignment's booking phone.
# It inherits the same bounded, retained challenge fields as BookingPhoneChallenge but uses
# a different table and subject. WhatsApp/SMS are delivery channels for one challenge, not
# separate verification purposes.
# No second purchase OTP exists. Backend verification must later authorize the active visit
# transition; recording this model alone neither starts a trial nor confirms a sale.
class ArrivalOTP(VerificationChallenge):
    """A different purpose and table from pre-payment booking phone verification.

    Both messaging channels deliver this one challenge; no purchase OTP exists.
    Refs: BUSINESS_RULES.md R-20, R-21, R-22, R-36.
    """

    assignment = models.ForeignKey(
        DeliveryAssignment, on_delete=models.PROTECT, related_name="arrival_challenges"
    )

    immutable_fields = (*VerificationChallenge.immutable_fields, "assignment_id")

    def clean(self):
        super().clean()
        if self.assignment_id and self.phone != self.assignment.booking.contact_phone:
            raise ValidationError({"phone": "Arrival phone must match the assigned booking."})
```

<a id="app-markets"></a>

## markets

<a id="model-markets-market"></a>

### Market

**Table:** `markets_market`. **Source:** [markets/models.py](../arkatara_backend/markets/models.py).

A city/market is data rather than a hard-coded branch. The inherited code, name and lifecycle fields allow future cities without a new model.

country_code, public identity and code are immutable through validated saves. Country validation checks format, not a full geopolitical registry.

ACTIVE is not enough to accept a booking: coverage, inventory, plans, prices and other readiness checks remain necessary.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `code` | SlugField(64); unique | no / required | `No declared default` | Reference identifier separate from a display name; its uniqueness scope follows the model. |
| `name` | CharField(200) | no / required | `No declared default` | Reference/display label; do not use as a permanent cross-system identifier. |
| `status` | CharField(16) | no / required | `ReferenceStatus.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `ACTIVE`, `COMING_SOON`, `INACTIVE`. |
| `country_code` | CharField(2) | no / required | `No declared default` | Explicit country identity used by the relevant model's format/consistency validation. |

**Explicit Meta constraints:** `markets_market_status_valid`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-markets-hub"></a>

### Hub

**Table:** `markets_hub`. **Source:** [markets/models.py](../arkatara_backend/markets/models.py).

An operational inventory location belonging to one Market. Multiple hubs can belong to a market.

Validated saves preserve hub identity and market association. code inherits global uniqueness in the current implementation.

A hub location does not establish legal stock ownership, authorize intercity fulfilment, or choose one-hub versus multi-hub booking sourcing.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `code` | SlugField(64); unique | no / required | `No declared default` | Reference identifier separate from a display name; its uniqueness scope follows the model. |
| `name` | CharField(200) | no / required | `No declared default` | Reference/display label; do not use as a permanent cross-system identifier. |
| `status` | CharField(16) | no / required | `ReferenceStatus.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `ACTIVE`, `COMING_SOON`, `INACTIVE`. |
| `market` | ForeignKey -> markets.Market; PROTECT | no / required | `No declared default` | Selected market/city reference, not an automatically chosen fulfilment hub. |

**Explicit Meta constraints:** `markets_hub_status_valid`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-markets-servicearea"></a>

### ServiceArea

**Table:** `markets_servicearea`. **Source:** [markets/models.py](../arkatara_backend/markets/models.py).

A versioned/effective postal coverage record for a selected market and country. The PIN/postal identifier is configuration data, not a customer order.

Active records need approval reference and effective start; end must follow start when both exist. Country must match the market, and Indian PIN format is validated.

Multiple rows may describe a scope. Existing coverage services detect overlapping effective approvals as conflicts; this model does not impose a routing priority or guarantee booking eligibility.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `market` | ForeignKey -> markets.Market; PROTECT | no / required | `No declared default` | Selected market/city reference, not an automatically chosen fulfilment hub. |
| `country_code` | CharField(2) | no / required | `No declared default` | Explicit country identity used by the relevant model's format/consistency validation. |
| `postal_code` | CharField(20) | no / required | `No declared default` | Configured coverage PIN or snapshotted recipient PIN, depending on this model. |
| `status` | CharField(16) | no / required | `ServiceArea.Status.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `ACTIVE`, `INACTIVE`. |
| `effective_from` | DateTimeField | yes / allowed | `No declared default` | Start of reference/policy applicability where configured. |
| `effective_to` | DateTimeField | yes / allowed | `No declared default` | Optional end of applicability, subject to the model's range checks. |
| `approval_reference` | CharField(500) | no / allowed | `No declared default` | Evidence reference required for configured activation where applicable. |

**Explicit Meta constraints:** `service_area_status_valid`, `service_area_effective_range_valid`, `service_area_active_evidence_required`

**Explicit Meta indexes:** `service_area_lookup_idx`

### Source: markets/models.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [markets/models.py](../arkatara_backend/markets/models.py).

```python
"""Market identity and reviewed postal coverage, independent of hub routing."""

from datetime import datetime

from django.core.exceptions import ValidationError
from django.core.validators import RegexValidator
from django.db import models
from django.utils import timezone

from common.reference import CodedReferenceModel, ValidatedReferenceModel


# REFERENCE NOTE (documentation only; not present in application source):
# A city/market is data rather than a hard-coded branch. The inherited code, name and
# lifecycle fields allow future cities without a new model.
# country_code, public identity and code are immutable through validated saves. Country
# validation checks format, not a full geopolitical registry.
# ACTIVE is not enough to accept a booking: coverage, inventory, plans, prices and other
# readiness checks remain necessary.
class Market(CodedReferenceModel):
    """Represent cities as configured data so expansion does not require city branches.

    Refs: BUSINESS_RULES.md R-10, R-13.
    """

    country_code = models.CharField(
        max_length=2,
        validators=[RegexValidator(r"\A[A-Z]{2}\Z", "Use a two-letter uppercase country code.")],
    )

    immutable_fields = (*CodedReferenceModel.immutable_fields, "country_code")


# REFERENCE NOTE (documentation only; not present in application source):
# An operational inventory location belonging to one Market. Multiple hubs can belong to a
# market.
# Validated saves preserve hub identity and market association. code inherits global
# uniqueness in the current implementation.
# A hub location does not establish legal stock ownership, authorize intercity fulfilment,
# or choose one-hub versus multi-hub booking sourcing.
class Hub(CodedReferenceModel):
    """Keep operational stock pools in a market without selecting routing or ownership.

    Refs: BUSINESS_RULES.md R-08; DECISIONS.md BD-08, BD-11.
    """

    market = models.ForeignKey(Market, on_delete=models.PROTECT, related_name="hubs")

    immutable_fields = (*CodedReferenceModel.immutable_fields, "market_id")


# REFERENCE NOTE (documentation only; not present in application source):
# A versioned/effective postal coverage record for a selected market and country. The
# PIN/postal identifier is configuration data, not a customer order.
# Active records need approval reference and effective start; end must follow start when
# both exist. Country must match the market, and Indian PIN format is validated.
# Multiple rows may describe a scope. Existing coverage services detect overlapping
# effective approvals as conflicts; this model does not impose a routing priority or
# guarantee booking eligibility.
class ServiceArea(ValidatedReferenceModel):
    """Represent reviewed postal coverage without supplying launch PIN codes or routes.

    Coverage alone does not prove the stock, price or plan requirements for booking.
    Refs: BUSINESS_RULES.md R-07, R-09, R-10; DECISIONS.md BD-08, BD-13.
    """

    class Status(models.TextChoices):
        DRAFT = "DRAFT", "Draft"
        ACTIVE = "ACTIVE", "Active"
        INACTIVE = "INACTIVE", "Inactive"

    market = models.ForeignKey(Market, on_delete=models.PROTECT, related_name="service_areas")
    country_code = models.CharField(
        max_length=2,
        validators=[RegexValidator(r"\A[A-Z]{2}\Z", "Use a two-letter uppercase country code.")],
    )
    postal_code = models.CharField(max_length=20)
    status = models.CharField(max_length=16, choices=Status.choices, default=Status.DRAFT)
    effective_from = models.DateTimeField(null=True, blank=True)
    effective_to = models.DateTimeField(null=True, blank=True)
    approval_reference = models.CharField(max_length=500, blank=True)

    immutable_fields = ("public_id", "market_id", "country_code", "postal_code")

    class Meta(ValidatedReferenceModel.Meta):
        ordering = ["market__code", "postal_code", "effective_from"]
        constraints = [
            models.CheckConstraint(
                condition=models.Q(status__in=["DRAFT", "ACTIVE", "INACTIVE"]),
                name="service_area_status_valid",
            ),
            models.CheckConstraint(
                condition=models.Q(effective_to__isnull=True)
                | models.Q(effective_from__isnull=True)
                | models.Q(effective_to__gt=models.F("effective_from")),
                name="service_area_effective_range_valid",
            ),
            models.CheckConstraint(
                condition=~models.Q(status="ACTIVE")
                | (models.Q(effective_from__isnull=False) & ~models.Q(approval_reference="")),
                name="service_area_active_evidence_required",
            ),
        ]
        indexes = [
            models.Index(
                fields=["market", "country_code", "postal_code", "status"],
                name="service_area_lookup_idx",
            )
        ]

    def clean(self) -> None:
        super().clean()
        errors = {}
        if (
            not isinstance(self.postal_code, str)
            or self.postal_code != self.postal_code.strip()
            or not self.postal_code.strip()
        ):
            errors["postal_code"] = "Use a nonempty postal identifier without outer whitespace."
        if self.country_code == "IN" and (
            not isinstance(self.postal_code, str)
            or len(self.postal_code) != 6
            or not self.postal_code.isascii()
            or not self.postal_code.isdigit()
            or self.postal_code.startswith("0")
        ):
            errors["postal_code"] = (
                "An Indian PIN code must contain six digits and not start with zero."
            )
        if self.market_id:
            country = (
                Market.objects.filter(pk=self.market_id)
                .values_list("country_code", flat=True)
                .first()
            )
            if country is not None and country != self.country_code:
                errors["country_code"] = "The service area must use its market's country."
        for field in ("effective_from", "effective_to"):
            value = getattr(self, field)
            if isinstance(value, datetime) and timezone.is_naive(value):
                errors[field] = "Use a timezone-aware effective timestamp."
        if self.status == self.Status.ACTIVE and (
            not isinstance(self.approval_reference, str)
            or not self.approval_reference.strip()
            or self.effective_from is None
        ):
            errors["status"] = "Active coverage requires an approval reference and effective start."
        if errors:
            raise ValidationError(errors)

    def __str__(self) -> str:
        return f"{self.market.code}: {self.country_code} {self.postal_code}"
```

<a id="app-catalog"></a>

## catalog

<a id="model-catalog-material"></a>

### Material

**Table:** `catalog_material`. **Source:** [catalog/models.py](../arkatara_backend/catalog/models.py).

The material family, separate from purity, product design and physical jewellery. All fields are inherited from CodedReferenceModel.

Stable code and UUID distinguish identity from a changeable display name. DRAFT is the creation default; activation is a separate readiness input.

Initial Silver/Gold identities are seed data in existing migrations, not enum-only limits. A future material can be represented without changing this class.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `code` | SlugField(64); unique | no / required | `No declared default` | Reference identifier separate from a display name; its uniqueness scope follows the model. |
| `name` | CharField(200) | no / required | `No declared default` | Reference/display label; do not use as a permanent cross-system identifier. |
| `status` | CharField(16) | no / required | `ReferenceStatus.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `ACTIVE`, `COMING_SOON`, `INACTIVE`. |

**Explicit Meta constraints:** `catalog_material_status_valid`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-catalog-purity"></a>

### Purity

**Table:** `catalog_purity`. **Source:** [catalog/models.py](../arkatara_backend/catalog/models.py).

A named purity standard belonging to one material, such as the approved initial S925 reference. Its identity is not itself a physical piece.

The material/code pair is unique. Validated saves preserve the material, code and UUID; ProductVariant checks that its selected purity belongs to its material.

Do not infer an approved Gold assortment, purity list, price formula or launch state from this storage.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `material` | ForeignKey -> catalog.Material; PROTECT | no / required | `No declared default` | Material family reference, distinct from purity and a physical piece. |
| `code` | SlugField(64) | no / required | `No declared default` | Reference identifier separate from a display name; its uniqueness scope follows the model. |
| `name` | CharField(200) | no / required | `No declared default` | Reference/display label; do not use as a permanent cross-system identifier. |
| `status` | CharField(16) | no / required | `ReferenceStatus.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `ACTIVE`, `COMING_SOON`, `INACTIVE`. |

**Explicit Meta constraints:** `purity_material_code_unique`, `catalog_purity_status_valid`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-catalog-category"></a>

### Category

**Table:** `catalog_category`. **Source:** [catalog/models.py](../arkatara_backend/catalog/models.py).

A category for product designs and category-specific trial boxes. Its fields come from CodedReferenceModel.

Creating or activating a category is distinct from configuring trial eligibility and plan limits for a market/material scope.

A TrialBox has one category. Repeated category boxes and item-counting policy remain separate decisions.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `code` | SlugField(64); unique | no / required | `No declared default` | Reference identifier separate from a display name; its uniqueness scope follows the model. |
| `name` | CharField(200) | no / required | `No declared default` | Reference/display label; do not use as a permanent cross-system identifier. |
| `status` | CharField(16) | no / required | `ReferenceStatus.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `ACTIVE`, `COMING_SOON`, `INACTIVE`. |

**Explicit Meta constraints:** `catalog_category_status_valid`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-catalog-product"></a>

### Product

**Table:** `catalog_product`. **Source:** [catalog/models.py](../arkatara_backend/catalog/models.py).

A catalogue design, not a selectable size/material configuration or a physical unit. It owns descriptive reference data and belongs to a category.

Once a design has variants with physical units or price records, normal saves refuse recategorization so those recorded definitions are not retargeted.

Names/descriptions are not transaction snapshots. StorefrontPage contains separately reviewed public content; future sale lines must preserve historical descriptions.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `code` | SlugField(64); unique | no / required | `No declared default` | Reference identifier separate from a display name; its uniqueness scope follows the model. |
| `name` | CharField(200) | no / required | `No declared default` | Reference/display label; do not use as a permanent cross-system identifier. |
| `status` | CharField(16) | no / required | `ReferenceStatus.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `ACTIVE`, `COMING_SOON`, `INACTIVE`. |
| `category` | ForeignKey -> catalog.Category; PROTECT | no / required | `No declared default` | Category identity; a box has exactly one category. |
| `description` | TextField | no / allowed | `No declared default` | Descriptive text; its publication/history rules depend on the owning model. |

**Explicit Meta constraints:** `catalog_product_status_valid`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-catalog-productvariant"></a>

### ProductVariant

**Table:** `catalog_productvariant`. **Source:** [catalog/models.py](../arkatara_backend/catalog/models.py).

A selectable configuration of a Product: material, optional purity, size and structured specification, identified by an opaque unique SKU.

Purity must match material. SKU and UUID are stable; defining attributes cannot change through validated saves once units or price revisions use them.

Each physical piece is a separate InventoryUnit. Nullable purity and an ACTIVE reference do not prove offering readiness or choose plan quantity/counting rules.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `product` | ForeignKey -> catalog.Product; PROTECT | no / required | `No declared default` | Catalogue design or page/media subject, not a physical stock unit. |
| `material` | ForeignKey -> catalog.Material; PROTECT | no / required | `No declared default` | Material family reference, distinct from purity and a physical piece. |
| `purity` | ForeignKey -> catalog.Purity; PROTECT | yes / allowed | `No declared default` | Optional purity reference; when present it must agree with variant material. |
| `sku` | CharField(100); unique | no / required | `No declared default` | Opaque unique variant identity. Never parse it for category, price or stock rules. |
| `size_label` | CharField(100) | no / allowed | `No declared default` | Human-readable selectable size/specification label. |
| `specification` | JSONField | no / allowed | `builtins.dict` | Structured variant specification; not a bag of authoritative client price data. |
| `status` | CharField(16) | no / required | `ReferenceStatus.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `ACTIVE`, `COMING_SOON`, `INACTIVE`. |

**Explicit Meta constraints:** `catalog_variant_status_valid`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-catalog-csvimportbatch"></a>

### CSVImportBatch

**Table:** `catalog_csvimportbatch`. **Source:** [catalog/import_batches.py](../arkatara_backend/catalog/import_batches.py).

An immutable record of a catalogue/physical-unit CSV onboarding operation, not the source CSV file itself.

Unique idempotency_key, payload hash, staff actor, reason, row count and per-row result support traceable retries and outcomes. Import services own payload matching and orchestration.

Both ORM and migration SQL protect import history. A successful import creates/reference-validates drafts; it does not imply physical receipt, QC or live saleability.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `idempotency_key` | UUIDField(32); unique | no / required | `No declared default` | Unique request/operation identity; services still check payload equality and outcomes. |
| `kind` | CharField(16) | no / required | `No declared default` | Discriminator whose valid values and relationships are specific to this model. Choices: `VARIANTS`, `UNITS`. |
| `actor` | ForeignKey -> accounts.StaffUser; PROTECT | no / required | `No declared default` | Staff actor FK. See model notes for optional named-system attribution where supported. |
| `payload_sha256` | CharField(64) | no / required | `No declared default` | Canonical import payload digest for retry comparison. |
| `reason` | TextField | no / required | `No declared default` | Attributable explanation for a controlled operation. |
| `row_count` | PositiveIntegerField | no / required | `No declared default` | Recorded import row count, not a physical stock-availability total. |
| `result` | JSONField | no / required | `No declared default` | Structured per-row import outcome retained as evidence. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |

**Explicit Meta constraints:** None declared directly; inspect field uniqueness, inherited validation and migration safeguards.

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-catalog-productmedia"></a>

### ProductMedia

**Table:** `catalog_productmedia`. **Source:** [catalog/media.py](../arkatara_backend/catalog/media.py).

Metadata and private object identity for a product image/video or source-linked thumbnail. Binary media is stored through a storage adapter, not in this table.

Product, role, source and object facts are preserved. A thumbnail must reference a same-product non-thumbnail source; a partial unique constraint limits the current main image.

Current states are DRAFT and RETIRED, not public publication. Validated upload/metadata services own writes; no public URL, consent, copyright clearance or sale permission is inferred.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `product` | ForeignKey -> catalog.Product; PROTECT | no / required | `No declared default` | Catalogue design or page/media subject, not a physical stock unit. |
| `source_media` | ForeignKey -> catalog.ProductMedia; PROTECT | yes / allowed | `No declared default` | Same-product source image/video required only for a thumbnail. |
| `role` | CharField(16) | no / required | `No declared default` | Media purpose: MAIN, ADDITIONAL, VIDEO or THUMBNAIL. Choices: `MAIN`, `ADDITIONAL`, `VIDEO`, `THUMBNAIL`. |
| `status` | CharField(16) | no / required | `ProductMedia.Status.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `RETIRED`. |
| `sort_order` | PositiveIntegerField | no / required | `0` | Presentation ordering value, not business authority. |
| `alt_text` | CharField(500) | no / required | `No declared default` | Required descriptive media text; no unsupported product claims. |
| `storage_backend` | CharField(50) | no / required | `No declared default` | Storage adapter identity, not a public URL or secret. |
| `object_key` | CharField(255); unique | no / required | `No declared default` | Unique private asset key scoped to product and asset identity. |
| `content_type` | CharField(50) | no / required | `No declared default` | Validated MIME type retained with the stored asset. |
| `byte_size` | PositiveBigIntegerField | no / required | `No declared default` | Positive asset length in bytes. |
| `sha256` | CharField(64) | no / required | `No declared default` | Recorded asset content digest. |

**Explicit Meta constraints:** `product_one_current_main_media`, `product_media_status_valid`, `product_media_role_valid`, `product_media_has_bytes`, `product_thumbnail_has_source`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-catalog-pricerevision"></a>

### PriceRevision

**Table:** `catalog_pricerevision`. **Source:** [catalog/pricing.py](../arkatara_backend/catalog/pricing.py).

An append-only proposal of fixed or weight-based pricing inputs for one variant/revision, with explicit currency and optional typed components.

ExactDecimalField and helpers reject floats, nonfinite numbers and excess precision. NUMERIC(24,6) is storage capacity, not the approved currency rounding rule.

Only DRAFT is allowed. No price-lock timing, active price selector, approved tax formula or financial transaction exists here; blank inputs must not be treated as zero.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `variant` | ForeignKey -> catalog.ProductVariant; PROTECT | no / required | `No declared default` | Selectable specification associated with this unit or price revision. |
| `revision` | PositiveIntegerField | no / required | `No declared default` | Positive version within the model's documented identity scope. |
| `status` | CharField(20) | no / required | `PriceRevision.Status.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`. |
| `mode` | CharField(20) | no / required | `No declared default` | FIXED or WEIGHT_BASED pricing input mode. Choices: `FIXED`, `WEIGHT_BASED`. |
| `currency` | CharField(3) | no / required | `No declared default` | Explicit uppercase three-letter currency code; no implicit currency default. |
| `fixed_amount` | ExactDecimalField | yes / allowed | `No declared default` | Optional exact fixed-price input; forbidden on a weight-based proposal. |
| `weight_value` | ExactDecimalField | yes / allowed | `No declared default` | Optional positive exact weight input, paired with weight_unit. |
| `weight_unit` | CharField(40) | no / allowed | `No declared default` | Explicit unit for a supplied weight_value. |
| `rate_amount` | ExactDecimalField | yes / allowed | `No declared default` | Optional exact monetary rate input, paired with rate_unit. |
| `rate_unit` | CharField(40) | no / allowed | `No declared default` | Explicit unit/basis for a supplied rate_amount. |
| `rate_source_reference` | CharField(500) | no / allowed | `No declared default` | Evidence/reference for a supplied rate; not a live rate provider. |
| `formula_reference` | CharField(500) | no / allowed | `No declared default` | Reference to a proposed pricing formula; no calculator is selected here. |
| `charges` | JSONField | no / allowed | `builtins.list` | Named exact monetary inputs in revision currency; no automatic total or tax formula. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |

**Explicit Meta constraints:** `price_revision_unique`, `price_revision_positive`, `price_revision_draft_only`, `price_revision_mode_valid`, `price_fixed_amount_nonnegative`, `price_weight_positive`, `price_rate_nonnegative`, `price_weight_has_no_fixed_amount`, `price_weight_has_unit`, `price_rate_has_unit`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-catalog-storefrontpage"></a>

### StorefrontPage

**Table:** `catalog_storefrontpage`. **Source:** [catalog/storefront_models.py](../arkatara_backend/catalog/storefront_models.py).

Reviewed public-page identity for exactly one material, category or product, with a protected subject reference, stable slug and optional collection parent.

Database checks require the matching single subject. Normal saves preserve route identity, prevent hierarchy cycles and require controlled publication/withdrawal; published text must be withdrawn before editing.

Publication grants no price, media, stock or booking approval. The current immutable slug is not a redirect-history model; redirect/alias persistence remains pending.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `kind` | CharField(16) | no / required | `No declared default` | Discriminator whose valid values and relationships are specific to this model. Choices: `MATERIAL`, `CATEGORY`, `PRODUCT`. |
| `material` | OneToOneField -> catalog.Material; PROTECT; unique | yes / allowed | `No declared default` | Material family reference, distinct from purity and a physical piece. |
| `category` | OneToOneField -> catalog.Category; PROTECT; unique | yes / allowed | `No declared default` | Category identity; a box has exactly one category. |
| `product` | OneToOneField -> catalog.Product; PROTECT; unique | yes / allowed | `No declared default` | Catalogue design or page/media subject, not a physical stock unit. |
| `parent` | ForeignKey -> catalog.StorefrontPage; PROTECT | yes / allowed | `No declared default` | Optional collection-page parent used for hierarchy and breadcrumbs; cycles are rejected. |
| `slug` | SlugField(140); unique | no / required | `No declared default` | Stable globally unique lowercase canonical path identity. |
| `title` | CharField(200) | no / required | `No declared default` | Reviewed visible public-page title, separate from internal catalogue labels. |
| `description` | TextField | no / required | `No declared default` | Descriptive text; its publication/history rules depend on the owning model. |
| `seo_title` | CharField(200) | no / allowed | `No declared default` | Optional factual metadata title; not a keyword-stuffing or ranking field. |
| `seo_description` | TextField | no / allowed | `No declared default` | Optional factual metadata description consistent with visible content. |
| `sort_order` | PositiveIntegerField | no / required | `0` | Presentation ordering value, not business authority. |
| `is_featured` | BooleanField | no / required | `False` | Merchandising placement flag, not publication or availability approval. |
| `status` | CharField(16) | no / required | `StorefrontPage.Status.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `PUBLISHED`, `WITHDRAWN`. |

**Explicit Meta constraints:** `storefront_status_valid`, `storefront_slug_lowercase`, `storefront_exactly_one_subject`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

### Source: catalog/models.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [catalog/models.py](../arkatara_backend/catalog/models.py).

```python
"""Catalogue designs and selectable specifications are not physical inventory."""

from django.core.exceptions import ValidationError
from django.db import models

from common.reference import CodedReferenceModel, ReferenceStatus, ValidatedReferenceModel


# REFERENCE NOTE (documentation only; not present in application source):
# The material family, separate from purity, product design and physical jewellery. All
# fields are inherited from CodedReferenceModel.
# Stable code and UUID distinguish identity from a changeable display name. DRAFT is the
# creation default; activation is a separate readiness input.
# Initial Silver/Gold identities are seed data in existing migrations, not enum-only limits.
# A future material can be represented without changing this class.
class Material(CodedReferenceModel):
    pass


# REFERENCE NOTE (documentation only; not present in application source):
# A named purity standard belonging to one material, such as the approved initial S925
# reference. Its identity is not itself a physical piece.
# The material/code pair is unique. Validated saves preserve the material, code and UUID;
# ProductVariant checks that its selected purity belongs to its material.
# Do not infer an approved Gold assortment, purity list, price formula or launch state from
# this storage.
class Purity(ValidatedReferenceModel):
    """Keep a material's standard distinct without approving a Gold assortment.

    Refs: BUSINESS_RULES.md R-11; DECISIONS.md BD-12.
    """

    material = models.ForeignKey(Material, on_delete=models.PROTECT, related_name="purities")
    code = models.SlugField(max_length=64)
    name = models.CharField(max_length=200)
    status = models.CharField(
        max_length=16, choices=ReferenceStatus.choices, default=ReferenceStatus.DRAFT
    )

    immutable_fields = ("public_id", "material_id", "code")

    class Meta(ValidatedReferenceModel.Meta):
        ordering = ["material__code", "code"]
        constraints = [
            models.UniqueConstraint(
                fields=["material", "code"], name="purity_material_code_unique"
            ),
            models.CheckConstraint(
                condition=models.Q(status__in=ReferenceStatus.values),
                name="catalog_purity_status_valid",
            ),
        ]

    def clean(self) -> None:
        super().clean()
        if not isinstance(self.name, str) or not self.name.strip():
            raise ValidationError({"name": "Provide a nonempty display name."})

    def __str__(self) -> str:
        return self.name


# REFERENCE NOTE (documentation only; not present in application source):
# A category for product designs and category-specific trial boxes. Its fields come from
# CodedReferenceModel.
# Creating or activating a category is distinct from configuring trial eligibility and plan
# limits for a market/material scope.
# A TrialBox has one category. Repeated category boxes and item-counting policy remain
# separate decisions.
class Category(CodedReferenceModel):
    """Trial eligibility requires scoped policy rather than following from existence.

    Refs: BUSINESS_RULES.md R-07, R-10; DECISIONS.md BD-13.
    """


# REFERENCE NOTE (documentation only; not present in application source):
# A catalogue design, not a selectable size/material configuration or a physical unit. It
# owns descriptive reference data and belongs to a category.
# Once a design has variants with physical units or price records, normal saves refuse
# recategorization so those recorded definitions are not retargeted.
# Names/descriptions are not transaction snapshots. StorefrontPage contains separately
# reviewed public content; future sale lines must preserve historical descriptions.
class Product(CodedReferenceModel):
    category = models.ForeignKey(Category, on_delete=models.PROTECT, related_name="products")
    description = models.TextField(blank=True)

    def validate_existing_identity(self, previous) -> None:
        # Recategorizing a used design would retarget its recorded units/prices.
        # Refs: BUSINESS_RULES.md R-11, R-28.
        super().validate_existing_identity(previous)
        if (
            self.category_id != previous.category_id
            and self.variants.filter(
                models.Q(inventory_units__isnull=False) | models.Q(price_revisions__isnull=False)
            ).exists()
        ):
            raise ValidationError(
                {"category": "A design with units or price records cannot be recategorized."}
            )


# REFERENCE NOTE (documentation only; not present in application source):
# A selectable configuration of a Product: material, optional purity, size and structured
# specification, identified by an opaque unique SKU.
# Purity must match material. SKU and UUID are stable; defining attributes cannot change
# through validated saves once units or price revisions use them.
# Each physical piece is a separate InventoryUnit. Nullable purity and an ACTIVE reference
# do not prove offering readiness or choose plan quantity/counting rules.
class ProductVariant(ValidatedReferenceModel):
    """A selectable specification with an opaque SKU, separate from physical pieces.

    Used definitions stay stable; historical transaction values still need snapshots.
    Refs: BUSINESS_RULES.md R-11, R-15, R-28.
    """

    product = models.ForeignKey(Product, on_delete=models.PROTECT, related_name="variants")
    material = models.ForeignKey(Material, on_delete=models.PROTECT, related_name="variants")
    purity = models.ForeignKey(
        Purity, null=True, blank=True, on_delete=models.PROTECT, related_name="variants"
    )
    sku = models.CharField(max_length=100, unique=True)
    size_label = models.CharField(max_length=100, blank=True)
    specification = models.JSONField(default=dict, blank=True)
    status = models.CharField(
        max_length=16, choices=ReferenceStatus.choices, default=ReferenceStatus.DRAFT
    )

    immutable_fields = ("public_id", "sku")
    definition_fields = ("product_id", "material_id", "purity_id", "size_label", "specification")

    class Meta(ValidatedReferenceModel.Meta):
        ordering = ["sku"]
        constraints = [
            models.CheckConstraint(
                condition=models.Q(status__in=ReferenceStatus.values),
                name="catalog_variant_status_valid",
            ),
        ]

    def clean(self) -> None:
        super().clean()
        errors = {}
        if not isinstance(self.sku, str) or not self.sku.strip() or self.sku != self.sku.strip():
            errors["sku"] = "Use a nonempty SKU without outer whitespace."
        if not isinstance(self.specification, dict):
            errors["specification"] = "Variant specification must be a JSON object."
        if self.purity_id and self.material_id:
            material = (
                Purity.objects.filter(pk=self.purity_id)
                .values_list("material_id", flat=True)
                .first()
            )
            if material is not None and material != self.material_id:
                errors["purity"] = "Purity must belong to the variant material."
        if errors:
            raise ValidationError(errors)

    def validate_existing_identity(self, previous) -> None:
        super().validate_existing_identity(previous)
        changed = [
            field
            for field in self.definition_fields
            if getattr(self, field) != getattr(previous, field)
        ]
        if changed and (self.inventory_units.exists() or self.price_revisions.exists()):
            raise ValidationError(
                {
                    self._meta.get_field(field).name: (
                        "A specification with recorded units or prices cannot be redefined."
                    )
                    for field in changed
                }
            )

    def __str__(self) -> str:
        return self.sku


# Register separately organized catalogue models without importing workflow services.
from catalog.import_batches import CSVImportBatch  # noqa: E402,F401
from catalog.media import ProductMedia  # noqa: E402,F401
from catalog.pricing import PriceRevision  # noqa: E402,F401
from catalog.storefront_models import StorefrontPage  # noqa: E402,F401
```

### Source: catalog/import_batches.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [catalog/import_batches.py](../arkatara_backend/catalog/import_batches.py).

```python
"""Immutable catalogue import outcomes, separate from onboarding orchestration."""

import uuid

from django.conf import settings
from django.db import models

from compliance.models import ImmutableQuerySet


# REFERENCE NOTE (documentation only; not present in application source):
# An immutable record of a catalogue/physical-unit CSV onboarding operation, not the source
# CSV file itself.
# Unique idempotency_key, payload hash, staff actor, reason, row count and per-row result
# support traceable retries and outcomes. Import services own payload matching and
# orchestration.
# Both ORM and migration SQL protect import history. A successful import
# creates/reference-validates drafts; it does not imply physical receipt, QC or live
# saleability.
class CSVImportBatch(models.Model):
    """Retain attributable import outcomes so a retry cannot repeat onboarding.

    Refs: BUSINESS_RULES.md R-11, R-28, R-30; DECISIONS.md BD-09.
    """

    public_id = models.UUIDField(default=uuid.uuid4, unique=True, editable=False)
    idempotency_key = models.UUIDField(unique=True)
    kind = models.CharField(max_length=16, choices=[("VARIANTS", "Variants"), ("UNITS", "Units")])
    actor = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.PROTECT)
    payload_sha256 = models.CharField(max_length=64)
    reason = models.TextField()
    row_count = models.PositiveIntegerField()
    result = models.JSONField()
    created_at = models.DateTimeField(auto_now_add=True)

    objects = ImmutableQuerySet.as_manager()

    class Meta:
        ordering = ["-created_at"]
        base_manager_name = "objects"
        permissions = [("import_catalogue", "Can preview and apply draft catalogue CSV imports")]

    def save(self, *args, **kwargs):
        if not self._state.adding:
            raise TypeError("Import outcomes are append-only.")
        self.full_clean()
        kwargs["force_insert"] = True
        return super().save(*args, **kwargs)

    def delete(self, *args, **kwargs):
        raise TypeError("Import outcomes are append-only.")
```

### Source: catalog/media.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [catalog/media.py](../arkatara_backend/catalog/media.py).

```python
"""Product-owned media identity, independent of storage hosts and publication."""

from django.core.exceptions import ValidationError
from django.db import models

from catalog.storage import KEY_PATTERN, validate_object_key
from common.reference import ValidatedReferenceModel, ValidatedReferenceQuerySet


# REFERENCE NOTE (documentation only; not present in application source):
# Prevents deleting product media through ordinary querysets; use the attributed retirement
# workflow.
class MediaQuerySet(ValidatedReferenceQuerySet):
    def delete(self):
        raise TypeError("Retire media through the attributed workflow; do not erase asset history.")


# REFERENCE NOTE (documentation only; not present in application source):
# Metadata and private object identity for a product image/video or source-linked thumbnail.
# Binary media is stored through a storage adapter, not in this table.
# Product, role, source and object facts are preserved. A thumbnail must reference a
# same-product non-thumbnail source; a partial unique constraint limits the current main
# image.
# Current states are DRAFT and RETIRED, not public publication. Validated upload/metadata
# services own writes; no public URL, consent, copyright clearance or sale permission is
# inferred.
class ProductMedia(ValidatedReferenceModel):
    """Upload metadata grants no public access or live offering permission.

    Refs: BUSINESS_RULES.md R-11, R-30; DECISIONS.md BD-15.
    """

    class Role(models.TextChoices):
        MAIN = "MAIN", "Main image"
        ADDITIONAL = "ADDITIONAL", "Additional image"
        VIDEO = "VIDEO", "Video"
        THUMBNAIL = "THUMBNAIL", "Thumbnail"

    class Status(models.TextChoices):
        DRAFT = "DRAFT", "Draft"
        RETIRED = "RETIRED", "Retired"

    product = models.ForeignKey("catalog.Product", on_delete=models.PROTECT, related_name="media")
    source_media = models.ForeignKey(
        "self", on_delete=models.PROTECT, null=True, blank=True, related_name="thumbnails"
    )
    role = models.CharField(max_length=16, choices=Role.choices)
    status = models.CharField(max_length=16, choices=Status.choices, default=Status.DRAFT)
    sort_order = models.PositiveIntegerField(default=0)
    alt_text = models.CharField(max_length=500)
    storage_backend = models.CharField(max_length=50, editable=False)
    object_key = models.CharField(max_length=255, unique=True, editable=False)
    content_type = models.CharField(max_length=50, editable=False)
    byte_size = models.PositiveBigIntegerField(editable=False)
    sha256 = models.CharField(max_length=64, editable=False)

    objects = MediaQuerySet.as_manager()
    immutable_fields = (
        "public_id",
        "product_id",
        "source_media_id",
        "storage_backend",
        "object_key",
        "content_type",
        "byte_size",
        "sha256",
        "role",
    )

    class Meta(ValidatedReferenceModel.Meta):
        app_label = "catalog"
        ordering = ["product", "sort_order", "pk"]
        constraints = [
            models.UniqueConstraint(
                fields=["product"],
                condition=models.Q(role="MAIN", status="DRAFT"),
                name="product_one_current_main_media",
            ),
            models.CheckConstraint(
                condition=models.Q(status__in=["DRAFT", "RETIRED"]),
                name="product_media_status_valid",
            ),
            models.CheckConstraint(
                condition=models.Q(role__in=["MAIN", "ADDITIONAL", "VIDEO", "THUMBNAIL"]),
                name="product_media_role_valid",
            ),
            models.CheckConstraint(
                condition=models.Q(byte_size__gt=0), name="product_media_has_bytes"
            ),
            models.CheckConstraint(
                condition=(models.Q(role="THUMBNAIL", source_media__isnull=False))
                | (~models.Q(role="THUMBNAIL") & models.Q(source_media__isnull=True)),
                name="product_thumbnail_has_source",
            ),
        ]

    def __str__(self):
        return f"{self.product_id}: {self.role} ({self.public_id})"

    def save(self, *args, **kwargs):
        if not getattr(self, "_service_write", False):
            raise TypeError("Use the attributed media upload/metadata workflow.")
        try:
            return super().save(*args, **kwargs)
        finally:
            self._service_write = False

    def clean(self):
        super().clean()
        if not isinstance(self.alt_text, str) or not self.alt_text.strip():
            raise ValidationError({"alt_text": "Describe the product media."})
        if type(self.sort_order) is not int or self.sort_order < 0:
            raise ValidationError({"sort_order": "Use a nonnegative integer for ordering."})
        if self.content_type:
            expected = (
                {"video/mp4"} if self.role == self.Role.VIDEO else {"image/png", "image/jpeg"}
            )
            if self.content_type not in expected:
                raise ValidationError(
                    {"role": "Media role does not match the validated file type."}
                )
        if self.object_key:
            validate_object_key(self.object_key)
            match = KEY_PATTERN.fullmatch(self.object_key)
            if self.product_id and match.group("product") != str(self.product.public_id):
                raise ValidationError(
                    {"product": "Media assets cannot be reassigned between designs."}
                )
            if match.group("asset") != str(self.public_id):
                raise ValidationError(
                    {"object_key": "Media identity must match its object reference."}
                )
        if self.role == self.Role.THUMBNAIL:
            if not self.source_media_id:
                raise ValidationError({"source_media": "A thumbnail requires its source media."})
            if self.source_media_id == self.pk:
                raise ValidationError({"source_media": "A thumbnail cannot reference itself."})
            if self.source_media.product_id != self.product_id:
                raise ValidationError(
                    {"source_media": "Thumbnail source must belong to this design."}
                )
            if self.source_media.role == self.Role.THUMBNAIL:
                raise ValidationError(
                    {"source_media": "Use the original image or video as source."}
                )
        elif self.source_media_id:
            raise ValidationError(
                {"source_media": "Only thumbnails have a source media reference."}
            )

    def validate_existing_identity(self, previous):
        super().validate_existing_identity(previous)
        if previous.status == self.Status.RETIRED and self.status != previous.status:
            raise ValidationError({"status": "Retired media cannot be restored silently."})

    def delete(self, *args, **kwargs):
        raise TypeError("Retire media through the attributed workflow; do not erase asset history.")
```

### Source: catalog/pricing.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [catalog/pricing.py](../arkatara_backend/catalog/pricing.py).

```python
"""Price proposals and exact inputs; commercial calculation remains BD-04/CA-02."""

import re
import uuid
from decimal import Decimal, InvalidOperation

from django.core.exceptions import ValidationError
from django.core.validators import DecimalValidator, RegexValidator
from django.db import models, router, transaction
from django.db.models import Q

from compliance.models import ImmutableQuerySet

# Storage capacity, not a currency exponent or a transaction rounding policy.
# Refs: DECISIONS.md D-17, BD-04, CA-02.
MAX_DIGITS = 24
DECIMAL_PLACES = 6


def exact_decimal(value, *, nonnegative=True) -> Decimal:
    """Reject binary floats, nonfinite numbers and precision loss; never round."""
    if isinstance(value, bool) or not isinstance(value, Decimal | str | int):
        raise ValidationError("Use an exact decimal string or Decimal, never a float.")
    try:
        result = Decimal(value)
    except InvalidOperation as error:
        raise ValidationError("Use a valid exact decimal.") from error
    if not result.is_finite():
        raise ValidationError("Decimal values must be finite.")
    DecimalValidator(MAX_DIGITS, DECIMAL_PLACES)(result)
    if nonnegative and result < 0:
        raise ValidationError("Decimal values must be nonnegative.")
    return result.copy_abs() if result.is_zero() else result


def decimal_text(value, *, nonnegative=True) -> str:
    return format(exact_decimal(value, nonnegative=nonnegative), "f")


def validate_currency(value) -> str:
    if not isinstance(value, str) or not re.fullmatch(r"[A-Z]{3}", value):
        raise ValidationError("Currency must be an explicit three-letter uppercase code.")
    return value


def text_value(value, *, required=True) -> str:
    if not isinstance(value, str) or (required and not value.strip()):
        raise ValidationError("A nonblank text value is required.")
    if value != value.strip():
        raise ValidationError("Text values must not contain surrounding whitespace.")
    return value


def validate_charges(value) -> list[dict]:
    """Typed monetary inputs in the revision currency, without a sum/formula."""
    if not isinstance(value, list):
        raise ValidationError("Charges must be a list of named exact monetary inputs.")
    normalized = []
    codes = set()
    for charge in value:
        if not isinstance(charge, dict) or not {"code", "amount"} <= charge.keys():
            raise ValidationError("Each charge requires code and amount.")
        if charge.keys() - {"code", "amount", "source_reference"}:
            raise ValidationError("Unknown charge fields are not accepted.")
        code = text_value(charge["code"])
        if code in codes:
            raise ValidationError("Charge codes must be unique within a revision.")
        codes.add(code)
        normalized.append(
            {
                "code": code,
                "amount": decimal_text(charge["amount"]),
                "source_reference": text_value(charge.get("source_reference", ""), required=False),
            }
        )
    return normalized


def validate_pricing_inputs(value) -> dict:
    """Accept partial draft evidence without inventing units, a source or a formula."""
    if not isinstance(value, dict):
        raise ValidationError("Pricing inputs must be an object.")
    allowed = {
        "weight_value",
        "weight_unit",
        "rate_amount",
        "rate_unit",
        "rate_source_reference",
        "formula_reference",
        "charges",
    }
    if value.keys() - allowed:
        raise ValidationError("Unknown pricing input fields are not accepted.")
    normalized = {}
    for amount_key, unit_key in (("weight_value", "weight_unit"), ("rate_amount", "rate_unit")):
        amount = value.get(amount_key)
        unit = value.get(unit_key)
        if (amount is None) != (unit in (None, "")):
            raise ValidationError("Each weight or rate value requires its explicit unit.")
        if amount is not None:
            number = exact_decimal(amount)
            if amount_key == "weight_value" and number <= 0:
                raise ValidationError("A supplied weight must be positive.")
            normalized[amount_key] = format(number, "f")
            normalized[unit_key] = text_value(unit)
    for name in ("rate_source_reference", "formula_reference"):
        if name in value:
            normalized[name] = text_value(value[name], required=False)
    if normalized.get("rate_source_reference") and "rate_amount" not in normalized:
        raise ValidationError("A rate source must identify a supplied rate.")
    if "charges" in value:
        normalized["charges"] = validate_charges(value["charges"])
    return normalized


# REFERENCE NOTE (documentation only; not present in application source):
# Custom Django field, not a table. Uses exact_decimal to reject unsupported monetary values
# instead of silently rounding.
class ExactDecimalField(models.DecimalField):
    def to_python(self, value):
        if value is None:
            return None
        return exact_decimal(value)


# REFERENCE NOTE (documentation only; not present in application source):
# An append-only proposal of fixed or weight-based pricing inputs for one variant/revision,
# with explicit currency and optional typed components.
# ExactDecimalField and helpers reject floats, nonfinite numbers and excess precision.
# NUMERIC(24,6) is storage capacity, not the approved currency rounding rule.
# Only DRAFT is allowed. No price-lock timing, active price selector, approved tax formula
# or financial transaction exists here; blank inputs must not be treated as zero.
class PriceRevision(models.Model):
    """Preserve fixed/weight input proposals without approving a payable price.

    Draft storage supplies neither a commercial formula nor a price-lock point;
    current-price selection and activation remain later reviewed workflows.
    Refs: BUSINESS_RULES.md R-12, R-28; DECISIONS.md BD-04, BD-12, CA-02.
    """

    class Mode(models.TextChoices):
        FIXED = "FIXED", "Fixed"
        WEIGHT_BASED = "WEIGHT_BASED", "Weight based"

    class Status(models.TextChoices):
        DRAFT = "DRAFT", "Draft"

    public_id = models.UUIDField(default=uuid.uuid4, unique=True, editable=False)
    variant = models.ForeignKey(
        "catalog.ProductVariant", on_delete=models.PROTECT, related_name="price_revisions"
    )
    revision = models.PositiveIntegerField()
    status = models.CharField(max_length=20, choices=Status.choices, default=Status.DRAFT)
    mode = models.CharField(max_length=20, choices=Mode.choices)
    currency = models.CharField(
        max_length=3,
        validators=[RegexValidator(r"\A[A-Z]{3}\Z", "Use an uppercase currency code.")],
    )
    fixed_amount = ExactDecimalField(
        max_digits=MAX_DIGITS, decimal_places=DECIMAL_PLACES, null=True, blank=True
    )
    weight_value = ExactDecimalField(
        max_digits=MAX_DIGITS, decimal_places=DECIMAL_PLACES, null=True, blank=True
    )
    weight_unit = models.CharField(max_length=40, blank=True)
    rate_amount = ExactDecimalField(
        max_digits=MAX_DIGITS, decimal_places=DECIMAL_PLACES, null=True, blank=True
    )
    rate_unit = models.CharField(max_length=40, blank=True)
    rate_source_reference = models.CharField(max_length=500, blank=True)
    formula_reference = models.CharField(max_length=500, blank=True)
    charges = models.JSONField(default=list, blank=True)
    created_at = models.DateTimeField(auto_now_add=True, editable=False)

    objects = ImmutableQuerySet.as_manager()

    class Meta:
        app_label = "catalog"
        base_manager_name = "objects"
        ordering = ["variant", "-revision"]
        constraints = [
            models.UniqueConstraint(fields=["variant", "revision"], name="price_revision_unique"),
            models.CheckConstraint(condition=Q(revision__gte=1), name="price_revision_positive"),
            models.CheckConstraint(condition=Q(status="DRAFT"), name="price_revision_draft_only"),
            models.CheckConstraint(
                condition=Q(mode__in=["FIXED", "WEIGHT_BASED"]), name="price_revision_mode_valid"
            ),
            models.CheckConstraint(
                condition=Q(fixed_amount__isnull=True) | Q(fixed_amount__gte=0),
                name="price_fixed_amount_nonnegative",
            ),
            models.CheckConstraint(
                condition=Q(weight_value__isnull=True) | Q(weight_value__gt=0),
                name="price_weight_positive",
            ),
            models.CheckConstraint(
                condition=Q(rate_amount__isnull=True) | Q(rate_amount__gte=0),
                name="price_rate_nonnegative",
            ),
            models.CheckConstraint(
                condition=Q(mode="FIXED") | Q(fixed_amount__isnull=True),
                name="price_weight_has_no_fixed_amount",
            ),
            models.CheckConstraint(
                condition=(Q(weight_value__isnull=True) & Q(weight_unit=""))
                | (Q(weight_value__isnull=False) & ~Q(weight_unit="")),
                name="price_weight_has_unit",
            ),
            models.CheckConstraint(
                condition=(Q(rate_amount__isnull=True) & Q(rate_unit=""))
                | (Q(rate_amount__isnull=False) & ~Q(rate_unit="")),
                name="price_rate_has_unit",
            ),
        ]

    def __str__(self):
        return f"{self.variant_id}@{self.revision} ({self.mode}, {self.status})"

    def save(self, *args, **kwargs):
        if not self._state.adding:
            raise TypeError("Price revisions are append-only; record a new revision.")
        from catalog.models import Product, ProductVariant

        database = kwargs.get("using") or router.db_for_write(type(self), instance=self)
        with transaction.atomic(using=database):
            if self.variant_id:
                variant = (
                    ProductVariant.objects.using(database)
                    .select_for_update()
                    .filter(pk=self.variant_id)
                    .first()
                )
                if variant is not None:
                    Product.objects.using(database).select_for_update().get(pk=variant.product_id)
            # Normalize Decimal charge inputs before JSONField's JSON validation.
            self.charges = validate_charges(self.charges)
            self.full_clean()
            kwargs["force_insert"] = True
            kwargs["using"] = database
            return super().save(*args, **kwargs)

    def delete(self, *args, **kwargs):
        raise TypeError("Price revisions are append-only; preserve recorded evidence.")

    def clean(self):
        super().clean()
        validate_currency(self.currency)
        if self.mode == self.Mode.WEIGHT_BASED and self.fixed_amount is not None:
            raise ValidationError({"fixed_amount": "A weight-based draft has no fixed price."})
        self.pricing_inputs()
        self.charges = validate_charges(self.charges)

    def pricing_inputs(self) -> dict:
        return validate_pricing_inputs(
            {
                "weight_value": self.weight_value,
                "weight_unit": self.weight_unit,
                "rate_amount": self.rate_amount,
                "rate_unit": self.rate_unit,
                "rate_source_reference": self.rate_source_reference,
                "formula_reference": self.formula_reference,
                "charges": self.charges,
            }
        )
```

### Source: catalog/storefront_models.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [catalog/storefront_models.py](../arkatara_backend/catalog/storefront_models.py).

```python
"""Stable public page identity, separate from offering activation and private assets."""

from contextlib import contextmanager
from contextvars import ContextVar

from django.core.exceptions import ValidationError
from django.core.validators import RegexValidator
from django.db import connections, models, router, transaction
from django.db.models.functions import Lower

from common.reference import ValidatedReferenceModel, ValidatedReferenceQuerySet

_publication_write = ContextVar("storefront_publication_write", default=False)


def lock_page_tree(database="default"):
    # Serialize hierarchy edits before taking page row locks, preventing concurrent cycles.
    with connections[database].cursor() as cursor:
        cursor.execute("SELECT pg_advisory_xact_lock(%s)", [674805010404])


@contextmanager
def publication_write():
    token = _publication_write.set(True)
    try:
        yield
    finally:
        _publication_write.reset(token)


# REFERENCE NOTE (documentation only; not present in application source):
# Prevents ordinary deletion of canonical page identity; use the withdrawal workflow.
class StorefrontQuerySet(ValidatedReferenceQuerySet):
    def delete(self):
        raise TypeError("Withdraw storefront pages; preserve their canonical identity.")


# REFERENCE NOTE (documentation only; not present in application source):
# Reviewed public-page identity for exactly one material, category or product, with a
# protected subject reference, stable slug and optional collection parent.
# Database checks require the matching single subject. Normal saves preserve route identity,
# prevent hierarchy cycles and require controlled publication/withdrawal; published text
# must be withdrawn before editing.
# Publication grants no price, media, stock or booking approval. The current immutable slug
# is not a redirect-history model; redirect/alias persistence remains pending.
class StorefrontPage(ValidatedReferenceModel):
    """Publishing content never approves prices, media delivery, eligibility or booking.

    Stable routes survive label changes and future market/material expansion.
    Refs: BUSINESS_RULES.md R-07, R-09, R-11, R-30; DECISIONS.md D-20.
    """

    class Kind(models.TextChoices):
        MATERIAL = "MATERIAL", "Material collection"
        CATEGORY = "CATEGORY", "Category collection"
        PRODUCT = "PRODUCT", "Product"

    class Status(models.TextChoices):
        DRAFT = "DRAFT", "Draft"
        PUBLISHED = "PUBLISHED", "Published"
        WITHDRAWN = "WITHDRAWN", "Withdrawn"

    kind = models.CharField(max_length=16, choices=Kind.choices)
    material = models.OneToOneField(
        "catalog.Material",
        null=True,
        blank=True,
        on_delete=models.PROTECT,
        related_name="storefront_page",
    )
    category = models.OneToOneField(
        "catalog.Category",
        null=True,
        blank=True,
        on_delete=models.PROTECT,
        related_name="storefront_page",
    )
    product = models.OneToOneField(
        "catalog.Product",
        null=True,
        blank=True,
        on_delete=models.PROTECT,
        related_name="storefront_page",
    )
    parent = models.ForeignKey(
        "self", null=True, blank=True, on_delete=models.PROTECT, related_name="children"
    )
    slug = models.SlugField(
        max_length=140,
        unique=True,
        validators=[
            RegexValidator(
                r"\A[a-z0-9]+(?:-[a-z0-9]+)*\Z", "Use lowercase words separated by hyphens."
            )
        ],
    )
    title = models.CharField(max_length=200)
    description = models.TextField()
    seo_title = models.CharField(max_length=200, blank=True)
    seo_description = models.TextField(blank=True)
    sort_order = models.PositiveIntegerField(default=0)
    is_featured = models.BooleanField(default=False)
    status = models.CharField(max_length=16, choices=Status.choices, default=Status.DRAFT)

    objects = StorefrontQuerySet.as_manager()
    immutable_fields = ("public_id", "slug", "kind", "material_id", "category_id", "product_id")

    class Meta(ValidatedReferenceModel.Meta):
        app_label = "catalog"
        ordering = ["sort_order", "slug"]
        permissions = [
            ("publish_storefrontpage", "Can publish reviewed storefront content"),
            ("withdraw_storefrontpage", "Can withdraw storefront content"),
        ]
        constraints = [
            models.CheckConstraint(
                condition=models.Q(status__in=["DRAFT", "PUBLISHED", "WITHDRAWN"]),
                name="storefront_status_valid",
            ),
            models.CheckConstraint(
                condition=models.Q(slug=Lower("slug")), name="storefront_slug_lowercase"
            ),
            models.CheckConstraint(
                condition=(
                    models.Q(
                        kind="MATERIAL",
                        material__isnull=False,
                        category__isnull=True,
                        product__isnull=True,
                    )
                    | models.Q(
                        kind="CATEGORY",
                        category__isnull=False,
                        material__isnull=True,
                        product__isnull=True,
                    )
                    | models.Q(
                        kind="PRODUCT",
                        product__isnull=False,
                        material__isnull=True,
                        category__isnull=True,
                    )
                ),
                name="storefront_exactly_one_subject",
            ),
        ]

    @property
    def path(self):
        prefix = "products" if self.kind == self.Kind.PRODUCT else "collections"
        return f"/{prefix}/{self.slug}"

    def __str__(self):
        return self.title

    def clean(self):
        super().clean()
        expected = {
            self.Kind.MATERIAL: (True, False, False),
            self.Kind.CATEGORY: (False, True, False),
            self.Kind.PRODUCT: (False, False, True),
        }.get(self.kind)
        actual = (bool(self.material_id), bool(self.category_id), bool(self.product_id))
        if actual != expected:
            raise ValidationError("Link exactly the subject appropriate to this page kind.")
        for field in ("title", "description"):
            if not isinstance(getattr(self, field), str) or not getattr(self, field).strip():
                raise ValidationError({field: "Supply reviewed, nonblank page content."})
        visited = {self.pk} if self.pk is not None else set()
        current = self.parent_id
        while current is not None:
            if current in visited:
                raise ValidationError({"parent": "Page hierarchy cannot contain a cycle."})
            visited.add(current)
            ancestor = type(self).objects.filter(pk=current).only("parent_id", "kind").first()
            if ancestor is None:
                break
            if ancestor.kind == self.Kind.PRODUCT:
                raise ValidationError({"parent": "Only collection pages may be parents."})
            current = ancestor.parent_id

    def validate_existing_identity(self, previous):
        super().validate_existing_identity(previous)
        if not _publication_write.get():
            if self.status != previous.status:
                raise ValidationError({"status": "Use the publish or withdraw workflow."})
            if previous.status == self.Status.PUBLISHED:
                raise ValidationError("Withdraw published content before editing it.")

    def save(self, *args, **kwargs):
        if self._state.adding and self.status != self.Status.DRAFT:
            raise ValidationError({"status": "New storefront content must start as a draft."})
        database = kwargs.get("using") or router.db_for_write(type(self), instance=self)
        with transaction.atomic(using=database):
            lock_page_tree(database)
            return super().save(*args, **kwargs)

    def delete(self, *args, **kwargs):
        raise TypeError("Withdraw storefront pages; preserve their canonical identity.")
```

<a id="app-inventory"></a>

## inventory

<a id="model-inventory-inventoryunit"></a>

### InventoryUnit

**Table:** `inventory_inventoryunit`. **Source:** [inventory/models.py](../arkatara_backend/inventory/models.py).

One uniquely tracked physical piece, linked to a variant and its operational hub. public_id is separate from the variant SKU and any future scanning/tagging scheme.

New pieces start DRAFT/UNKNOWN. Reviewed receiving/QC services own state/custody changes; AVAILABLE requires HUB custody. Variant/hub identity is preserved by normal model guards.

Current lifecycle covers receipt and inspection only. Reservation, agent custody, dispatch, sale, return and ownership-transfer persistence are not implemented by these statuses.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `variant` | ForeignKey -> catalog.ProductVariant; PROTECT | no / required | `No declared default` | Selectable specification associated with this unit or price revision. |
| `hub` | ForeignKey -> markets.Hub; PROTECT | no / required | `No declared default` | Operational location identity, not legal stock ownership. |
| `status` | CharField(16) | no / required | `InventoryUnit.Status.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `QC_PENDING`, `AVAILABLE`, `QUARANTINED`, `RETIRED`. |
| `custody` | CharField(16) | no / required | `InventoryUnit.Custody.UNKNOWN` | Current implemented physical custody state; UNKNOWN/HUB only in this foundation. Choices: `UNKNOWN`, `HUB`. |

**Explicit Meta constraints:** `inventory_unit_status_valid`, `inventory_unit_custody_valid`, `available_unit_at_hub`

**Explicit Meta indexes:** `inventory_unit_lookup_idx`

<a id="model-inventory-inventorymovement"></a>

### InventoryMovement

**Table:** `inventory_inventorymovement`. **Source:** [inventory/models.py](../arkatara_backend/inventory/models.py).

Append-only evidence of a unit's registration, hub receipt or inspection result, including before/after facts and staff attribution.

Observation time differs from recorded time. Idempotency key/fingerprint identify the operation; non-registration records retain an approved procedure revision and copied snapshot.

Reviewed stock services create this evidence atomically with stock changes; SQL guards protect history. It does not yet cover dispatch/sale/return or determine legal stock ownership.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `unit` | ForeignKey -> inventory.InventoryUnit; PROTECT | no / required | `No declared default` | Protected identity of the exact physical piece whose movement is recorded. |
| `action` | CharField(16) | no / required | `No declared default` | Observed inventory operation or audit action, depending on the owning model. Choices: `REGISTER`, `RECEIVE`, `QC_PASS`, `QC_FAIL`. |
| `from_status` | CharField(16) | no / allowed | `No declared default` | State before the recorded inventory operation. |
| `to_status` | CharField(16) | no / required | `No declared default` | State after the recorded inventory operation. Choices: `DRAFT`, `QC_PENDING`, `AVAILABLE`, `QUARANTINED`, `RETIRED`. |
| `from_hub` | ForeignKey -> markets.Hub; PROTECT | yes / allowed | `No declared default` | Prior location reference where applicable, not proof of a completed transfer. |
| `to_hub` | ForeignKey -> markets.Hub; PROTECT | no / required | `No declared default` | Recorded destination hub identity for this operation. |
| `from_custody` | CharField(16) | no / required | `No declared default` | Custody before the operation. Choices: `UNKNOWN`, `HUB`. |
| `to_custody` | CharField(16) | no / required | `No declared default` | Custody after the operation. Choices: `UNKNOWN`, `HUB`. |
| `actor` | ForeignKey -> accounts.StaffUser; PROTECT | no / required | `No declared default` | Staff actor FK. See model notes for optional named-system attribution where supported. |
| `actor_label` | CharField(255) | no / required | `No declared default` | Copied staff UUID label or permitted named-system identity; not a customer link. |
| `occurred_at` | DateTimeField | no / required | `No declared default` | Event/observation time. In AuditEvent it is generated on insertion. |
| `recorded_at` | DateTimeField | no / allowed | `auto_now_add` | Database-recorded evidence time, distinct from an observed event time. |
| `reason` | TextField | no / required | `No declared default` | Attributable explanation for a controlled operation. |
| `source_reference` | CharField(500) | no / required | `No declared default` | Supporting evidence/procedure/source reference, not fabricated approval. |
| `idempotency_key` | UUIDField(32); unique | no / required | `No declared default` | Unique request/operation identity; services still check payload equality and outcomes. |
| `request_fingerprint` | CharField(64) | no / required | `No declared default` | Digest of canonical request facts; it does not itself implement replay handling. |
| `procedure_revision` | ForeignKey -> compliance.BusinessConfiguration; PROTECT | yes / allowed | `No declared default` | Protected configuration revision used for a physical operation. |
| `procedure_snapshot` | JSONField | no / allowed | `builtins.dict` | Copied approval/scope/value evidence, independent of later configuration lookups. |

**Explicit Meta constraints:** `inventory_movement_action_valid`

**Explicit Meta indexes:** `unit_movement_time_idx`

### Source: inventory/models.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [inventory/models.py](../arkatara_backend/inventory/models.py).

```python
"""Physical stock identity and append-only evidence for reviewed hub operations."""

import uuid
from contextlib import contextmanager
from contextvars import ContextVar

from django.conf import settings
from django.core.exceptions import ValidationError
from django.db import models
from django.utils import timezone

from common.reference import ValidatedReferenceModel, ValidatedReferenceQuerySet
from compliance.models import ImmutableQuerySet

_operational_write = ContextVar("inventory_operational_write", default=False)


@contextmanager
def _recording_operation():
    """Internal service boundary; never expose this as an Admin/API bypass flag."""
    token = _operational_write.set(True)
    try:
        yield
    finally:
        _operational_write.reset(token)


# REFERENCE NOTE (documentation only; not present in application source):
# Retains physical identities instead of treating deletion as an inventory operation.
class InventoryUnitQuerySet(ValidatedReferenceQuerySet):
    def delete(self):
        raise TypeError("Physical identities are retained; deletion is not a stock operation.")


# REFERENCE NOTE (documentation only; not present in application source):
# One uniquely tracked physical piece, linked to a variant and its operational hub.
# public_id is separate from the variant SKU and any future scanning/tagging scheme.
# New pieces start DRAFT/UNKNOWN. Reviewed receiving/QC services own state/custody changes;
# AVAILABLE requires HUB custody. Variant/hub identity is preserved by normal model guards.
# Current lifecycle covers receipt and inspection only. Reservation, agent custody,
# dispatch, sale, return and ownership-transfer persistence are not implemented by these
# statuses.
class InventoryUnit(ValidatedReferenceModel):
    """Keep physical identity distinct from receipt, inspection and availability.

    Reviewed receiving/QC services own changes; reservation, dispatch, sale and
    customer-return workflows remain later dependencies.
    Refs: BUSINESS_RULES.md R-08, R-11, R-17; DECISIONS.md BD-09.
    """

    class Status(models.TextChoices):
        DRAFT = "DRAFT", "Draft / not operationally available"
        QC_PENDING = "QC_PENDING", "Awaiting inspection"
        AVAILABLE = "AVAILABLE", "Available at hub"
        QUARANTINED = "QUARANTINED", "Quarantined"
        RETIRED = "RETIRED", "Retired"

    class Custody(models.TextChoices):
        UNKNOWN = "UNKNOWN", "Not physically verified"
        HUB = "HUB", "Verified at hub"

    variant = models.ForeignKey(
        "catalog.ProductVariant", on_delete=models.PROTECT, related_name="inventory_units"
    )
    hub = models.ForeignKey("markets.Hub", on_delete=models.PROTECT, related_name="inventory_units")
    status = models.CharField(max_length=16, choices=Status.choices, default=Status.DRAFT)
    custody = models.CharField(max_length=16, choices=Custody.choices, default=Custody.UNKNOWN)

    objects = InventoryUnitQuerySet.as_manager()

    immutable_fields = ("public_id", "variant_id", "hub_id")

    class Meta(ValidatedReferenceModel.Meta):
        ordering = ["public_id"]
        constraints = [
            models.CheckConstraint(
                condition=models.Q(
                    status__in=["DRAFT", "QC_PENDING", "AVAILABLE", "QUARANTINED", "RETIRED"]
                ),
                name="inventory_unit_status_valid",
            ),
            models.CheckConstraint(
                condition=models.Q(custody__in=["UNKNOWN", "HUB"]),
                name="inventory_unit_custody_valid",
            ),
            models.CheckConstraint(
                condition=~models.Q(status="AVAILABLE") | models.Q(custody="HUB"),
                name="available_unit_at_hub",
            ),
        ]
        indexes = [
            models.Index(fields=["hub", "variant", "status"], name="inventory_unit_lookup_idx")
        ]
        permissions = [
            ("receive_inventoryunit", "Can record physical hub receipt"),
            ("inspect_inventoryunit", "Can record hub quality inspection"),
        ]

    def validate_existing_identity(self, previous) -> None:
        super().validate_existing_identity(previous)
        if not _operational_write.get() and (
            self.status != previous.status or self.custody != previous.custody
        ):
            raise ValidationError("Use the receiving or inspection workflow to change stock state.")

    def save(self, *args, **kwargs):
        if self._state.adding and (
            self.status != self.Status.DRAFT or self.custody != self.Custody.UNKNOWN
        ):
            raise ValidationError("New physical identities start as unverified drafts.")
        return super().save(*args, **kwargs)

    def delete(self, *args, **kwargs):
        raise TypeError("Physical identities are retained; deletion is not a stock operation.")

    def lock_related_identity(self, database: str) -> None:
        # Share the variant/product locks used by definition edits, so a concurrent
        # first unit cannot be inserted after an edit checked for existing units.
        from catalog.models import Product, ProductVariant

        if self.variant_id:
            variant = (
                ProductVariant.objects.using(database)
                .select_for_update()
                .filter(pk=self.variant_id)
                .first()
            )
            if variant is not None:
                Product.objects.using(database).select_for_update().get(pk=variant.product_id)

    def __str__(self) -> str:
        return str(self.public_id)


# REFERENCE NOTE (documentation only; not present in application source):
# Append-only evidence of a unit's registration, hub receipt or inspection result, including
# before/after facts and staff attribution.
# Observation time differs from recorded time. Idempotency key/fingerprint identify the
# operation; non-registration records retain an approved procedure revision and copied
# snapshot.
# Reviewed stock services create this evidence atomically with stock changes; SQL guards
# protect history. It does not yet cover dispatch/sale/return or determine legal stock
# ownership.
class InventoryMovement(models.Model):
    """Preserve what was observed and which reviewed procedure authorized the action.

    Hub location is not stock ownership; corrections require new reviewed actions.
    Refs: BUSINESS_RULES.md R-15, R-28, R-30; DECISIONS.md BD-09, BD-11.
    """

    class Action(models.TextChoices):
        REGISTER = "REGISTER", "Draft registration"
        RECEIVE = "RECEIVE", "Physical hub receipt"
        QC_PASS = "QC_PASS", "Inspection passed"
        QC_FAIL = "QC_FAIL", "Inspection failed"

    public_id = models.UUIDField(default=uuid.uuid4, unique=True, editable=False)
    unit = models.ForeignKey(InventoryUnit, on_delete=models.PROTECT, related_name="movements")
    action = models.CharField(max_length=16, choices=Action.choices)
    from_status = models.CharField(max_length=16, blank=True)
    to_status = models.CharField(max_length=16, choices=InventoryUnit.Status.choices)
    from_hub = models.ForeignKey(
        "markets.Hub",
        null=True,
        blank=True,
        on_delete=models.PROTECT,
        related_name="outgoing_inventory_movements",
    )
    to_hub = models.ForeignKey(
        "markets.Hub", on_delete=models.PROTECT, related_name="incoming_inventory_movements"
    )
    from_custody = models.CharField(max_length=16, choices=InventoryUnit.Custody.choices)
    to_custody = models.CharField(max_length=16, choices=InventoryUnit.Custody.choices)
    actor = models.ForeignKey(
        settings.AUTH_USER_MODEL, on_delete=models.PROTECT, related_name="inventory_movements"
    )
    actor_label = models.CharField(max_length=255, editable=False)
    occurred_at = models.DateTimeField()
    recorded_at = models.DateTimeField(auto_now_add=True, editable=False)
    reason = models.TextField()
    source_reference = models.CharField(max_length=500)
    idempotency_key = models.UUIDField(unique=True)
    request_fingerprint = models.CharField(max_length=64, editable=False)
    procedure_revision = models.ForeignKey(
        "compliance.BusinessConfiguration",
        null=True,
        blank=True,
        on_delete=models.PROTECT,
        related_name="inventory_movements",
    )
    procedure_snapshot = models.JSONField(default=dict, blank=True)

    objects = ImmutableQuerySet.as_manager()

    class Meta:
        ordering = ["recorded_at", "pk"]
        base_manager_name = "objects"
        indexes = [models.Index(fields=["unit", "occurred_at"], name="unit_movement_time_idx")]
        constraints = [
            models.CheckConstraint(
                condition=models.Q(action__in=["REGISTER", "RECEIVE", "QC_PASS", "QC_FAIL"]),
                name="inventory_movement_action_valid",
            ),
        ]

    def __str__(self):
        return f"{self.unit.public_id}: {self.action}"

    def save(self, *args, **kwargs):
        if not self._state.adding:
            raise TypeError("Inventory movement evidence is append-only.")
        if not _operational_write.get():
            raise ValidationError("Inventory movements are written by reviewed stock workflows.")
        self.actor_label = f"staff:{self.actor.public_id}"
        self.full_clean()
        kwargs["force_insert"] = True
        return super().save(*args, **kwargs)

    def delete(self, *args, **kwargs):
        raise TypeError("Inventory movement evidence is append-only.")

    def clean(self):
        super().clean()
        if self.occurred_at is not None and timezone.is_naive(self.occurred_at):
            raise ValidationError({"occurred_at": "Use a timezone-aware observation time."})
        if not self.reason.strip() or not self.source_reference.strip():
            raise ValidationError("Record both a reason and supporting source reference.")
        if self.action != self.Action.REGISTER and not self.procedure_revision_id:
            raise ValidationError("Physical operations require their reviewed procedure revision.")
        if not isinstance(self.procedure_snapshot, dict):
            raise ValidationError({"procedure_snapshot": "Procedure evidence must be an object."})
```

<a id="app-compliance"></a>

## compliance

<a id="model-compliance-policydocumentrevision"></a>

### PolicyDocumentRevision

**Table:** `compliance_policydocumentrevision`. **Source:** [compliance/policies.py](../arkatara_backend/compliance/policies.py).

Stores the exact policy wording and language for a numbered document revision. The document/language/revision combination is unique.

content_sha256 is computed from content on normal save. Append-only ORM and database guards preserve previously recorded wording; BookingPolicyAcceptance references this record.

A stored version is not legal approval, marketing consent or a selected retention period. Approval and publication/acceptance workflows remain separate.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `recorded_at` | DateTimeField | no / allowed | `auto_now_add` | Database-recorded evidence time, distinct from an observed event time. |
| `document_code` | SlugField(80) | no / required | `No declared default` | Policy document identity independent of revision/language. |
| `revision` | PositiveIntegerField | no / required | `No declared default` | Positive version within the model's documented identity scope. |
| `language_code` | CharField(20) | no / required | `No declared default` | Language of the retained wording; not an approval state. |
| `content` | TextField | no / required | `No declared default` | Exact policy text retained for later acceptance evidence. |
| `content_sha256` | CharField(64) | no / required | `No declared default` | SHA-256 computed from the UTF-8 policy content during normal save. |

**Explicit Meta constraints:** `policy_document_revision_unique`, `policy_revision_positive`, `policy_content_required`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-compliance-legalentity"></a>

### LegalEntity

**Table:** `compliance_legalentity`. **Source:** [compliance/models.py](../arkatara_backend/compliance/models.py).

Identifies a legal party that may later be responsible for stock or sales. Its presence does not select the operating company or establish tax registration validity.

Names, registration and effective dates are reference data. The explicit database check protects the date range; this model does not freeze historical issuer details.

Future issued documents must copy the actual issuer details at the relevant time. Audited Admin maintenance is outside this source excerpt; BD-11, CA-03 and LR-02 remain gates.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `code` | SlugField(40); unique | no / required | `No declared default` | Reference identifier separate from a display name; its uniqueness scope follows the model. |
| `registered_name` | CharField(255) | no / required | `No declared default` | Current legal registered name; future documents need an issuer snapshot. |
| `display_name` | CharField(255) | no / allowed | `No declared default` | Optional presentation label, not historical recipient identity. |
| `country_code` | CharField(2) | no / required | `No declared default` | Explicit country identity used by the relevant model's format/consistency validation. |
| `tax_registration_number` | CharField(64) | no / allowed | `No declared default` | Recorded registration reference; this field does not verify validity. |
| `status` | CharField(16) | no / required | `LegalEntity.Status.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `ACTIVE`, `INACTIVE`. |
| `effective_from` | DateField | yes / allowed | `No declared default` | Start of reference/policy applicability where configured. |
| `effective_to` | DateField | yes / allowed | `No declared default` | Optional end of applicability, subject to the model's range checks. |

**Explicit Meta constraints:** `legal_entity_effective_range_valid`

**Explicit Meta indexes:** None declared directly; automatic field/relationship indexes may still exist.

<a id="model-compliance-businessconfiguration"></a>

### BusinessConfiguration

**Table:** `compliance_businessconfiguration`. **Source:** [compliance/models.py](../arkatara_backend/compliance/models.py).

Versioned, scoped business configuration. namespace/key identify a registered setting; scope is its dimensions and scope_key is the normalized lookup identity.

value is validated against the configuration registry. A unique namespace/key/scope_key/version prevents duplicate revisions. Non-draft revisions require approval fields, but these do not create approval on their own.

Ordinary ORM writes are append-only; this foundation does not claim a database UPDATE/DELETE trigger for configuration. Do not add sample deposits, limits, expiry values or scope precedence as defaults.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `id` | BigAutoField; PK | no / allowed | `Database-generated` | Internal numeric primary key; public-facing identity normally uses public_id. |
| `created_at` | DateTimeField | no / allowed | `auto_now_add` | When this row was first recorded. |
| `updated_at` | DateTimeField | no / allowed | `auto_now` | Last normal save timestamp; not a replacement for an audit trail. |
| `public_id` | UUIDField(32); unique | no / required | `uuid.uuid4` | Stable opaque public identity. It is an identifier, not an access credential. |
| `namespace` | SlugField(80) | no / required | `No declared default` | Registered business configuration domain. |
| `key` | SlugField(120) | no / required | `No declared default` | Registered setting name within namespace. |
| `scope_key` | CharField(200) | no / required | `'global'` | Deterministic normalized identity derived from the scope object. |
| `scope` | JSONField | no / allowed | `builtins.dict` | Configuration dimensions; exact-scope semantics are supplied by the registry/services. |
| `value` | JSONField | no / required | `No declared default` | Typed configuration payload validated against the registered setting contract. |
| `version` | PositiveIntegerField | no / required | `1` | Positive configuration revision number within namespace/key/scope. |
| `status` | CharField(16) | no / required | `BusinessConfiguration.Status.DRAFT` | Model-specific lifecycle only; inspect this model's choices and guards, not another app's status. Choices: `DRAFT`, `ACTIVE`, `RETIRED`. |
| `effective_from` | DateTimeField | yes / allowed | `No declared default` | Start of reference/policy applicability where configured. |
| `effective_to` | DateTimeField | yes / allowed | `No declared default` | Optional end of applicability, subject to the model's range checks. |
| `approval_reference` | CharField(500) | no / allowed | `No declared default` | Evidence reference required for configured activation where applicable. |
| `approval_recorded_by` | ForeignKey -> accounts.StaffUser; PROTECT | yes / allowed | `No declared default` | Staff identity recording the approval evidence. |
| `approved_at` | DateTimeField | yes / allowed | `No declared default` | Recorded approval observation, not proof that an unresolved decision is approved. |

**Explicit Meta constraints:** `business_configuration_version_unique`, `business_configuration_version_positive`, `business_configuration_effective_range_valid`

**Explicit Meta indexes:** `business_config_lookup_idx`

<a id="model-compliance-auditevent"></a>

### AuditEvent

**Table:** `compliance_auditevent`. **Source:** [compliance/models.py](../arkatara_backend/compliance/models.py).

An append-only account of a privileged action, with resource identity, reason, changes, metadata and optional correlation UUID.

actor points only to StaffUser. Without a staff FK, a named system: actor_label is required. event_id is the primary key; there is no implicit numeric id on this model.

SQL migration protection complements ORM guards. Actual services must write audits atomically with the change. Customer attribution is not implemented; arbitrary metadata must not become a guest-to-account linking mechanism.

| Field | Type / relationship | Null / blank | Default / generated value | Explanation |
| --- | --- | --- | --- | --- |
| `event_id` | UUIDField(32); PK | no / required | `uuid.uuid4` | Audit UUID primary key; this model has no implicit numeric id. |
| `actor` | ForeignKey -> accounts.StaffUser; PROTECT | yes / allowed | `No declared default` | Staff actor FK. See model notes for optional named-system attribution where supported. |
| `actor_label` | CharField(255) | no / allowed | `No declared default` | Copied staff UUID label or permitted named-system identity; not a customer link. |
| `action` | CharField(120) | no / required | `No declared default` | Observed inventory operation or audit action, depending on the owning model. |
| `resource_type` | CharField(120) | no / required | `No declared default` | Audit target kind, not a foreign-key guarantee. |
| `resource_identifier` | CharField(255) | no / required | `No declared default` | Recorded audit target identity; authorization cannot rely on this string alone. |
| `occurred_at` | DateTimeField | no / allowed | `auto_now_add` | Event/observation time. In AuditEvent it is generated on insertion. |
| `reason` | TextField | no / allowed | `No declared default` | Attributable explanation for a controlled operation. |
| `changes` | JSONField | no / allowed | `builtins.dict` | Structured before/after change evidence. Avoid unnecessary personal data. |
| `metadata` | JSONField | no / allowed | `builtins.dict` | Minimized supplementary audit context; must not indirectly link guest history to accounts. |
| `correlation_id` | UUIDField(32) | yes / allowed | `No declared default` | Optional correlation UUID across related operational evidence. |

**Explicit Meta constraints:** None declared directly; inspect field uniqueness, inherited validation and migration safeguards.

**Explicit Meta indexes:** `audit_resource_time_idx`, `audit_actor_time_idx`

### Source: compliance/policies.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [compliance/policies.py](../arkatara_backend/compliance/policies.py).

```python
"""Versioned policy content; storing a revision does not make it legally approved."""

import hashlib

from django.core.exceptions import ValidationError
from django.db import models

from common.evidence import AppendOnlyModel


# REFERENCE NOTE (documentation only; not present in application source):
# Stores the exact policy wording and language for a numbered document revision. The
# document/language/revision combination is unique.
# content_sha256 is computed from content on normal save. Append-only ORM and database
# guards preserve previously recorded wording; BookingPolicyAcceptance references this
# record.
# A stored version is not legal approval, marketing consent or a selected retention period.
# Approval and publication/acceptance workflows remain separate.
class PolicyDocumentRevision(AppendOnlyModel):
    """Keep the exact content against which later acceptance is recorded.

    No commercial/legal approval, retention period or production wording is seeded.
    Refs: BUSINESS_RULES.md R-28, R-32, R-35; DECISIONS.md LR-04.
    """

    document_code = models.SlugField(max_length=80)
    revision = models.PositiveIntegerField()
    language_code = models.CharField(max_length=20)
    content = models.TextField()
    content_sha256 = models.CharField(max_length=64, editable=False)

    class Meta(AppendOnlyModel.Meta):
        constraints = [
            models.UniqueConstraint(
                fields=["document_code", "language_code", "revision"],
                name="policy_document_revision_unique",
            ),
            models.CheckConstraint(
                condition=models.Q(revision__gt=0), name="policy_revision_positive"
            ),
            models.CheckConstraint(condition=~models.Q(content=""), name="policy_content_required"),
        ]

    def clean(self):
        super().clean()
        if not self.content.strip() or not self.language_code.strip():
            raise ValidationError("Policy content and language must be nonblank.")

    def save(self, *args, **kwargs):
        if not isinstance(self.content, str):
            raise ValidationError({"content": "Policy content must be text."})
        self.content_sha256 = hashlib.sha256(self.content.encode("utf-8")).hexdigest()
        return super().save(*args, **kwargs)
```

### Source: compliance/models.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [compliance/models.py](../arkatara_backend/compliance/models.py).

```python
import uuid

from django.conf import settings
from django.core.exceptions import ValidationError
from django.db import models
from django.db.models import Q

from common.models import TimeStampedModel
from compliance.configuration import configuration_scope_key, validate_configuration_value
from compliance.policies import PolicyDocumentRevision  # noqa: F401


# REFERENCE NOTE (documentation only; not present in application source):
# Existing manager for revision/audit/import/movement records. Refuses ORM updates, deletes
# and bulk writes; database coverage must be checked separately for each model.
class ImmutableQuerySet(models.QuerySet):
    def update(self, **kwargs):
        raise TypeError("Recorded revisions and audit events are append-only.")

    def delete(self):
        raise TypeError("Recorded revisions and audit events are append-only.")

    def bulk_create(self, *args, **kwargs):
        raise TypeError("Use individually validated append-only writes; bulk inserts are disabled.")

    def bulk_update(self, *args, **kwargs):
        raise TypeError("Recorded revisions and audit events are append-only.")


# REFERENCE NOTE (documentation only; not present in application source):
# Identifies a legal party that may later be responsible for stock or sales. Its presence
# does not select the operating company or establish tax registration validity.
# Names, registration and effective dates are reference data. The explicit database check
# protects the date range; this model does not freeze historical issuer details.
# Future issued documents must copy the actual issuer details at the relevant time. Audited
# Admin maintenance is outside this source excerpt; BD-11, CA-03 and LR-02 remain gates.
class LegalEntity(TimeStampedModel):
    """Identify parties without inventing the actual operating or stock-owning entity.

    Later transaction records must snapshot the applicable party; this mutable
    reference cannot by itself preserve historical issuer details.
    Refs: BUSINESS_RULES.md R-27; DECISIONS.md BD-11, CA-03, LR-02.
    """

    class Status(models.TextChoices):
        DRAFT = "DRAFT", "Draft"
        ACTIVE = "ACTIVE", "Active"
        INACTIVE = "INACTIVE", "Inactive"

    public_id = models.UUIDField(default=uuid.uuid4, unique=True, editable=False)
    code = models.SlugField(max_length=40, unique=True)
    registered_name = models.CharField(max_length=255)
    display_name = models.CharField(max_length=255, blank=True)
    country_code = models.CharField(max_length=2)
    tax_registration_number = models.CharField(max_length=64, blank=True)
    status = models.CharField(max_length=16, choices=Status.choices, default=Status.DRAFT)
    effective_from = models.DateField(null=True, blank=True)
    effective_to = models.DateField(null=True, blank=True)

    class Meta:
        ordering = ["code"]
        constraints = [
            models.CheckConstraint(
                condition=Q(effective_to__isnull=True)
                | Q(effective_from__isnull=True)
                | Q(effective_to__gte=models.F("effective_from")),
                name="legal_entity_effective_range_valid",
            )
        ]

    def __str__(self) -> str:
        return self.registered_name


# REFERENCE NOTE (documentation only; not present in application source):
# Versioned, scoped business configuration. namespace/key identify a registered setting;
# scope is its dimensions and scope_key is the normalized lookup identity.
# value is validated against the configuration registry. A unique
# namespace/key/scope_key/version prevents duplicate revisions. Non-draft revisions require
# approval fields, but these do not create approval on their own.
# Ordinary ORM writes are append-only; this foundation does not claim a database
# UPDATE/DELETE trigger for configuration. Do not add sample deposits, limits, expiry values
# or scope precedence as defaults.
class BusinessConfiguration(TimeStampedModel):
    """Append policy revisions so later commitments can retain their original evidence.

    Typed values and approval fields do not supply approved production settings.
    Refs: BUSINESS_RULES.md R-28; DECISIONS.md BD-02, BD-13.
    """

    class Status(models.TextChoices):
        DRAFT = "DRAFT", "Draft"
        ACTIVE = "ACTIVE", "Active"
        RETIRED = "RETIRED", "Retired"

    public_id = models.UUIDField(default=uuid.uuid4, unique=True, editable=False)
    namespace = models.SlugField(max_length=80)
    key = models.SlugField(max_length=120)
    scope_key = models.CharField(max_length=200, default="global")
    scope = models.JSONField(default=dict, blank=True)
    value = models.JSONField()
    version = models.PositiveIntegerField(default=1)
    status = models.CharField(max_length=16, choices=Status.choices, default=Status.DRAFT)
    effective_from = models.DateTimeField(null=True, blank=True)
    effective_to = models.DateTimeField(null=True, blank=True)
    approval_reference = models.CharField(max_length=500, blank=True)
    approval_recorded_by = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        null=True,
        blank=True,
        on_delete=models.PROTECT,
        related_name="recorded_configuration_approvals",
    )
    approved_at = models.DateTimeField(null=True, blank=True)

    objects = ImmutableQuerySet.as_manager()

    class Meta:
        ordering = ["namespace", "key", "scope_key", "-version"]
        base_manager_name = "objects"
        constraints = [
            models.UniqueConstraint(
                fields=["namespace", "key", "scope_key", "version"],
                name="business_configuration_version_unique",
            ),
            models.CheckConstraint(
                condition=Q(version__gte=1),
                name="business_configuration_version_positive",
            ),
            models.CheckConstraint(
                condition=Q(effective_to__isnull=True)
                | Q(effective_from__isnull=True)
                | Q(effective_to__gt=models.F("effective_from")),
                name="business_configuration_effective_range_valid",
            ),
        ]
        indexes = [
            models.Index(
                fields=["namespace", "key", "scope_key", "status"],
                name="business_config_lookup_idx",
            )
        ]

    def clean(self) -> None:
        super().clean()
        self.scope_key = configuration_scope_key(self.scope)
        validate_configuration_value(self.namespace, self.key, self.value)
        if self.status != self.Status.DRAFT and (
            not self.approval_reference.strip()
            or not self.approval_recorded_by_id
            or not self.approved_at
            or not self.effective_from
        ):
            raise ValidationError(
                {"status": "Approved revisions require approval reference, actor and timestamps."}
            )
        if self.status == self.Status.DRAFT and (
            self.approval_reference or self.approval_recorded_by_id or self.approved_at
        ):
            raise ValidationError({"status": "Draft revisions cannot claim commercial approval."})

    def save(self, *args, **kwargs):
        if not self._state.adding:
            raise TypeError("Configuration revisions are append-only; record a new version.")
        self.scope_key = configuration_scope_key(self.scope)
        self.full_clean()
        # Never update an existing primary key passed into a newly constructed instance.
        kwargs["force_insert"] = True
        return super().save(*args, **kwargs)

    def delete(self, *args, **kwargs):
        raise TypeError("Configuration revisions are append-only; preserve their history.")

    def __str__(self) -> str:
        return f"{self.namespace}.{self.key}:{self.scope_key}@{self.version}"


# REFERENCE NOTE (documentation only; not present in application source):
# An append-only account of a privileged action, with resource identity, reason, changes,
# metadata and optional correlation UUID.
# actor points only to StaffUser. Without a staff FK, a named system: actor_label is
# required. event_id is the primary key; there is no implicit numeric id on this model.
# SQL migration protection complements ORM guards. Actual services must write audits
# atomically with the change. Customer attribution is not implemented; arbitrary metadata
# must not become a guest-to-account linking mechanism.
class AuditEvent(models.Model):
    """Retain attributable change evidence instead of editing the account of an action.

    Each privileged workflow must write its evidence in the same transaction.
    Refs: BUSINESS_RULES.md R-30.
    """

    event_id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    actor = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        null=True,
        blank=True,
        on_delete=models.PROTECT,
        related_name="audit_events",
    )
    actor_label = models.CharField(max_length=255, blank=True)
    action = models.CharField(max_length=120)
    resource_type = models.CharField(max_length=120)
    resource_identifier = models.CharField(max_length=255)
    occurred_at = models.DateTimeField(auto_now_add=True, editable=False)
    reason = models.TextField(blank=True)
    changes = models.JSONField(default=dict, blank=True)
    metadata = models.JSONField(default=dict, blank=True)
    correlation_id = models.UUIDField(null=True, blank=True)

    objects = ImmutableQuerySet.as_manager()

    class Meta:
        ordering = ["-occurred_at"]
        base_manager_name = "objects"
        indexes = [
            models.Index(
                fields=["resource_type", "resource_identifier", "occurred_at"],
                name="audit_resource_time_idx",
            ),
            models.Index(fields=["actor", "occurred_at"], name="audit_actor_time_idx"),
        ]

    def save(self, *args, **kwargs):
        if not self._state.adding:
            raise TypeError("Audit events are append-only.")
        if self.actor_id:
            self.actor_label = f"staff:{self.actor.public_id}"
        elif not self.actor_label.startswith("system:") or not self.actor_label[7:].strip():
            raise ValidationError(
                {"actor_label": "Identify a staff actor or a named system actor."}
            )
        for field_name in ("changes", "metadata"):
            if not isinstance(getattr(self, field_name), dict):
                raise ValidationError({field_name: "Audit evidence must be a JSON object."})
        self.full_clean()
        kwargs["force_insert"] = True
        return super().save(*args, **kwargs)

    def delete(self, *args, **kwargs):
        raise TypeError("Audit events are append-only.")
```

<a id="pending-apps"></a>

## Pending Apps and Known Missing Models

The following installed apps currently register **no project-defined database models**. Empty apps are not completed domains. This snapshot does not invent placeholders or choose open policies to make the list appear complete.

| App | Current implementation | Required scope still pending |
| --- | --- | --- |
| billing | HistoricalPriceSnapshot value object only, included below | Purchase/PurchaseItem, tax revisions, invoice series/header/lines, adjustments and after-sale document/case persistence |
| payments | No Django model registered | Upfront obligations, PaymentContext/Attempt, receipts/events, allocation/refund and settlement/bank reconciliation evidence |
| notifications | No Django model registered | Durable notification intents, templates/purpose references, attempts, delivery/retry/deduplication evidence |
| analytics | No Django model registered | Minimized attribution/events, deduplication/provenance and reviewed reporting inputs/projections |

Other implemented apps are also incomplete for Task 4. Customer contact/authentication/recovery and staff scope; TrialPlan/TrialBoxItem and allocation; InventoryReservation and later custody/movements; accepted-booking transitions/amendments; visits/outcomes/manifests; and public media/redirect history remain required work where applicable. Model design must distinguish missing engineering from schema-shaping business/CA/legal gates. See the [coverage matrix](DATA_MODEL.md#task-4-model-coverage) and [implementation checkpoint](reviews/TASK4_MODEL_FOUNDATION_2026-10-03.md).

Guest order ownership will remain null permanently when customer authentication is introduced. Do not add a merge/claim model, contact-match backfill or indirect account-history link. Conversely, account-mode ownership remains attached to its original customer identity, not its current contact number.

<a id="shared-bases-and-guards"></a>

## Shared Bases and Guards

These modules do not register separate concrete tables. Abstract fields are copied into their concrete descendants; querysets and custom fields are Python support classes. Original helpers/constants are included so the model guards are readable. This is still a source reference, not a standalone implementation of all imported services/configuration/validators.

### Source: common/models.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [common/models.py](../arkatara_backend/common/models.py).

```python
from django.db import models


# REFERENCE NOTE (documentation only; not present in application source):
# Abstract base adding created_at and updated_at to each concrete descendant. It has no
# table and does not itself make timestamps or business history immutable.
class TimeStampedModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True, editable=False)
    updated_at = models.DateTimeField(auto_now=True, editable=False)

    class Meta:
        abstract = True
```

### Source: common/reference.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [common/reference.py](../arkatara_backend/common/reference.py).

```python
"""Validated reference writes; these primitives grant no operating approval."""

import uuid

from django.core.exceptions import ValidationError
from django.db import models, router, transaction

from common.models import TimeStampedModel


# REFERENCE NOTE (documentation only; not present in application source):
# Shared reference choices, not a database model: DRAFT, ACTIVE, COMING_SOON and INACTIVE.
class ReferenceStatus(models.TextChoices):
    # Reference activation is one input to readiness, never booking permission.
    # Refs: BUSINESS_RULES.md R-09, R-13; DECISIONS.md D-17.
    DRAFT = "DRAFT", "Draft"
    ACTIVE = "ACTIVE", "Active"
    COMING_SOON = "COMING_SOON", "Coming soon"
    INACTIVE = "INACTIVE", "Inactive"


# REFERENCE NOTE (documentation only; not present in application source):
# Manager/queryset boundary refusing bulk writes that bypass ordinary identity validation.
# This is not a database trigger.
class ValidatedReferenceQuerySet(models.QuerySet):
    def update(self, **kwargs):
        raise TypeError("Use validated individual saves for reference and identity changes.")

    def bulk_create(self, *args, **kwargs):
        raise TypeError("Bulk writes bypass reference validation and are not supported.")

    def bulk_update(self, *args, **kwargs):
        raise TypeError("Bulk writes bypass reference validation and are not supported.")


# REFERENCE NOTE (documentation only; not present in application source):
# Abstract UUID/reference base. Normal saves lock existing rows, validate immutable identity
# and run full_clean. Queryset bulk writes are refused; SQL bypass protection is
# model/migration-specific, not universal.
class ValidatedReferenceModel(TimeStampedModel):
    """Keep referenced identities stable through validated, serialized normal writes.

    These guards do not implement stock reservation or financial workflows.
    Refs: BUSINESS_RULES.md R-11, R-28; DECISIONS.md D-17.
    """

    public_id = models.UUIDField(default=uuid.uuid4, unique=True, editable=False)

    objects = ValidatedReferenceQuerySet.as_manager()

    # Concrete classes include relation attnames (for example, market_id) here.
    immutable_fields = ("public_id",)

    class Meta:
        abstract = True
        base_manager_name = "objects"

    def validate_existing_identity(self, previous) -> None:
        changed = [
            field
            for field in self.immutable_fields
            if getattr(self, field) != getattr(previous, field)
        ]
        if changed:
            raise ValidationError(
                {
                    self._meta.get_field(field).name: "Recorded identity cannot be reassigned."
                    for field in changed
                }
            )

    def lock_related_identity(self, database: str) -> None:
        """Concrete physical-identity writes may lock their parent definitions."""

    def save(self, *args, **kwargs):
        database = kwargs.get("using") or router.db_for_write(type(self), instance=self)
        with transaction.atomic(using=database):
            if not self._state.adding:
                previous = type(self).objects.using(database).select_for_update().get(pk=self.pk)
                self.validate_existing_identity(previous)
            self.lock_related_identity(database)
            self.full_clean()
            if self._state.adding:
                # An explicitly supplied existing primary key must not overwrite a row.
                kwargs["force_insert"] = True
            kwargs["using"] = database
            return super().save(*args, **kwargs)


# REFERENCE NOTE (documentation only; not present in application source):
# Abstract reference base with stable code, display name and reference lifecycle. A code is
# configuration identity; ACTIVE does not establish operational readiness.
class CodedReferenceModel(ValidatedReferenceModel):
    code = models.SlugField(max_length=64, unique=True)
    name = models.CharField(max_length=200)
    status = models.CharField(
        max_length=16, choices=ReferenceStatus.choices, default=ReferenceStatus.DRAFT
    )

    immutable_fields = ("public_id", "code")

    class Meta(ValidatedReferenceModel.Meta):
        abstract = True
        ordering = ["code"]
        constraints = [
            models.CheckConstraint(
                condition=models.Q(status__in=ReferenceStatus.values),
                name="%(app_label)s_%(class)s_status_valid",
            )
        ]

    def clean(self) -> None:
        super().clean()
        if not isinstance(self.name, str) or not self.name.strip():
            raise ValidationError({"name": "Provide a nonempty display name."})

    def __str__(self) -> str:
        return self.name
```

### Source: common/evidence.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [common/evidence.py](../arkatara_backend/common/evidence.py).

```python
"""Validated, retained records; business authorization belongs to explicit workflows."""

import uuid

from django.core.exceptions import ValidationError
from django.db import models, router, transaction

from common.reference import ValidatedReferenceModel, ValidatedReferenceQuerySet


# REFERENCE NOTE (documentation only; not present in application source):
# Adds delete refusal for retained models; ordinary updates/bulk inserts remain blocked by
# the reference queryset.
class RetainedQuerySet(ValidatedReferenceQuerySet):
    def delete(self):
        raise TypeError("Recorded evidence must be retained.")


# REFERENCE NOTE (documentation only; not present in application source):
# Abstract retained-record base adding delete refusal to validated reference saves.
# Corresponding concrete-table database guards live in migrations.
class RetainedModel(ValidatedReferenceModel):
    objects = RetainedQuerySet.as_manager()

    class Meta(ValidatedReferenceModel.Meta):
        abstract = True

    def delete(self, *args, **kwargs):
        raise TypeError("Recorded evidence must be retained.")


# REFERENCE NOTE (documentation only; not present in application source):
# Refuses update on append-only evidence as well as inherited bulk/delete bypasses.
class AppendOnlyQuerySet(RetainedQuerySet):
    def update(self, **kwargs):
        raise TypeError("Recorded evidence is append-only.")


# REFERENCE NOTE (documentation only; not present in application source):
# Abstract evidence base with public UUID and recorded_at. New rows validate and insert;
# normal update/delete is refused. Concrete-table SQL guards are separate.
class AppendOnlyModel(models.Model):
    """Shared persistence guard for new evidence, with corresponding SQL migrations.

    A record is evidence, not permission or proof that an external event occurred.
    Refs: BUSINESS_RULES.md R-28, R-30; DECISIONS.md D-24.
    """

    public_id = models.UUIDField(default=uuid.uuid4, unique=True, editable=False)
    recorded_at = models.DateTimeField(auto_now_add=True, editable=False)

    objects = AppendOnlyQuerySet.as_manager()

    class Meta:
        abstract = True
        base_manager_name = "objects"

    def save(self, *args, **kwargs):
        if not self._state.adding:
            raise ValidationError("Recorded evidence is append-only.")
        database = kwargs.get("using") or router.db_for_write(type(self), instance=self)
        with transaction.atomic(using=database):
            self.lock_related_records(database)
            self.full_clean()
            kwargs["using"] = database
            kwargs["force_insert"] = True
            return super().save(*args, **kwargs)

    def delete(self, *args, **kwargs):
        raise TypeError("Recorded evidence is append-only.")

    def lock_related_records(self, database):
        """Concrete records can serialize validation of their referenced identities."""
```

### Source: common/verification.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [common/verification.py](../arkatara_backend/common/verification.py).

```python
"""Challenge storage only; no OTP issuance, verification or authentication endpoints."""

from datetime import datetime

from django.contrib.auth.hashers import identify_hasher
from django.core.exceptions import ValidationError
from django.core.validators import RegexValidator
from django.db import models
from django.db.models import F, Q
from django.utils import timezone

from common.evidence import RetainedModel

phone_validator = RegexValidator(r"\A\+[1-9][0-9]{1,14}\Z", "Use an E.164 phone number.")
digest_validator = RegexValidator(r"\A[0-9a-f]{64}\Z", "Use a lowercase SHA-256 digest.")


def validate_observed_at(value):
    if not isinstance(value, datetime) or timezone.is_naive(value):
        raise ValidationError("Use a timezone-aware timestamp.")
    if value > timezone.now():
        raise ValidationError("An observed event cannot be in the future.")


# REFERENCE NOTE (documentation only; not present in application source):
# Abstract storage shared by booking and arrival challenges, not a shared auth identity. Its
# fields and constraints are copied into each concrete challenge table; no common challenge
# table is created.
class VerificationChallenge(RetainedModel):
    """Bind a hashed secret and monotonic lifecycle to a concrete purpose/subject.

    No security limits are chosen here. The future verification service must lock
    this row, check the secret, increment attempts and consume it atomically.
    Refs: BUSINESS_RULES.md R-20, R-36; DECISIONS.md D-25, BD-15.
    """

    phone = models.CharField(max_length=16, validators=[phone_validator])
    secret_hash = models.CharField(max_length=255)
    issued_at = models.DateTimeField(default=timezone.now, validators=[validate_observed_at])
    expires_at = models.DateTimeField()
    max_attempts = models.PositiveSmallIntegerField()
    attempts_used = models.PositiveSmallIntegerField(default=0)
    verified_at = models.DateTimeField(null=True, blank=True, validators=[validate_observed_at])
    consumed_at = models.DateTimeField(null=True, blank=True, validators=[validate_observed_at])
    invalidated_at = models.DateTimeField(null=True, blank=True, validators=[validate_observed_at])

    immutable_fields = (
        "public_id",
        "phone",
        "secret_hash",
        "issued_at",
        "expires_at",
        "max_attempts",
    )

    class Meta(RetainedModel.Meta):
        abstract = True
        constraints = [
            models.CheckConstraint(
                condition=Q(max_attempts__gt=0) & Q(attempts_used__lte=F("max_attempts")),
                name="%(app_label)s_%(class)s_attempts",
            ),
            models.CheckConstraint(
                condition=Q(expires_at__gt=F("issued_at")),
                name="%(app_label)s_%(class)s_expiry",
            ),
            models.CheckConstraint(
                condition=Q(verified_at__isnull=True)
                | (
                    Q(verified_at__gte=F("issued_at"))
                    & Q(verified_at__lt=F("expires_at"))
                    & Q(attempts_used__gt=0)
                ),
                name="%(app_label)s_%(class)s_verified",
            ),
            models.CheckConstraint(
                condition=Q(consumed_at__isnull=True)
                | (
                    Q(verified_at__isnull=False)
                    & Q(consumed_at__gte=F("verified_at"))
                    & Q(consumed_at__lt=F("expires_at"))
                ),
                name="%(app_label)s_%(class)s_consumed",
            ),
            models.CheckConstraint(
                condition=Q(invalidated_at__isnull=True)
                | (
                    Q(invalidated_at__gte=F("issued_at"))
                    & (Q(verified_at__isnull=True) | Q(verified_at__lte=F("invalidated_at")))
                    & (Q(consumed_at__isnull=True) | Q(consumed_at__lte=F("invalidated_at")))
                ),
                name="%(app_label)s_%(class)s_invalidated",
            ),
        ]

    def clean(self):
        super().clean()
        try:
            identify_hasher(self.secret_hash).decode(self.secret_hash)
        except (ValueError, TypeError) as error:
            raise ValidationError(
                {"secret_hash": "Use a supported encoded secret hash."}
            ) from error
        if self.expires_at and timezone.is_naive(self.expires_at):
            raise ValidationError({"expires_at": "Use a timezone-aware expiry."})
        if self._state.adding and (
            self.attempts_used or self.verified_at or self.consumed_at or self.invalidated_at
        ):
            raise ValidationError("New challenges start unused and unverified.")

    def validate_existing_identity(self, previous):
        super().validate_existing_identity(previous)
        if self.attempts_used < previous.attempts_used:
            raise ValidationError("Challenge attempts cannot decrease.")
        if previous.verified_at and self.attempts_used != previous.attempts_used:
            raise ValidationError("Verified challenges cannot accept further attempts.")
        if (
            self.verified_at
            and not previous.verified_at
            and (self.attempts_used <= previous.attempts_used)
        ):
            raise ValidationError("Verification must record a new bounded attempt.")
        for field in ("verified_at", "consumed_at", "invalidated_at"):
            old = getattr(previous, field)
            if old is not None and getattr(self, field) != old:
                raise ValidationError("Recorded challenge outcomes cannot be rewritten.")
        if previous.invalidated_at or previous.consumed_at:
            if any(
                getattr(self, field) != getattr(previous, field)
                for field in ("attempts_used", "verified_at", "consumed_at", "invalidated_at")
            ):
                raise ValidationError("A closed challenge cannot be reused.")
```

<a id="billing-value-object"></a>

## Billing Value Object

HistoricalPriceSnapshot is a frozen dataclass with one internal `_serialized` string. Its constructor accepts a versioned payload with `schema_version`, `catalogue`, `price_revision_id`, `price_revision`, `mode`, `currency`, `agreed_amount`, `inputs` and `tax`. It validates and stores a defensive serialized copy; `to_dict()` returns a new copy. It has no Django manager, primary key, migration or table.

The catalogue payload copies identifiers/descriptions rather than depending on future mutable product labels. Tax is explicitly UNCONFIGURED or RECORDED with policy/component evidence; missing tax is never inferred to be zero. This object neither calculates an approved selling price nor persists a sale or invoice. Future transaction models must attach/protect its output at the approved commitment boundary.

### Source: billing/snapshots.py

Original model/module code, with all original comments retained. Additional comments are explicitly marked `REFERENCE NOTE`. Imports intentionally still point to the real modules; this is not a replacement module.

Source of truth: [billing/snapshots.py](../arkatara_backend/billing/snapshots.py).

```python
"""Immutable price/tax evidence for later transaction records, never a calculator.

Recording a supplied agreed amount does not choose when the business commits it,
approve a price revision, prove payment or issue an invoice (BD-04 and CA-02).
"""

import json
from dataclasses import dataclass
from uuid import UUID

from django.core.exceptions import ValidationError

from catalog.pricing import decimal_text, text_value, validate_currency, validate_pricing_inputs


def _identifier(value) -> str:
    try:
        return str(UUID(str(value)))
    except (ValueError, AttributeError, TypeError) as error:
        raise ValidationError("Snapshot identifiers must be UUIDs.") from error


def _catalogue(value) -> dict:
    required = {
        "product_id",
        "variant_id",
        "product_name",
        "variant_sku",
        "category_code",
        "material_code",
    }
    optional = {"product_code", "purity_code", "specifications"}
    if not isinstance(value, dict) or not required <= value.keys():
        raise ValidationError("Snapshot catalogue identity and description are required.")
    if value.keys() - (required | optional):
        raise ValidationError("Unknown catalogue snapshot fields are not accepted.")
    result = {
        key: _identifier(item) if key.endswith("_id") else text_value(item)
        for key, item in value.items()
        if key not in {"specifications", "purity_code"}
    }
    if "purity_code" in value:
        result["purity_code"] = (
            None if value["purity_code"] is None else text_value(value["purity_code"])
        )
    if "specifications" in value:
        specifications = value["specifications"]
        if not isinstance(specifications, dict):
            raise ValidationError("Snapshot specifications must be a text-to-text object.")
        result["specifications"] = {
            text_value(key): text_value(item) for key, item in specifications.items()
        }
    return result


def _tax_evidence(value) -> dict:
    """Missing reviewed tax is explicitly unconfigured, never an inferred zero rate.

    Recording supplied facts does not establish their legal/accounting approval.
    Refs: BUSINESS_RULES.md R-27, R-31; DECISIONS.md CA-02.
    """
    if not isinstance(value, dict):
        raise ValidationError("Tax evidence must explicitly identify its configuration state.")
    if value.get("state") == "UNCONFIGURED":
        if set(value) != {"state"}:
            raise ValidationError("Unconfigured tax must not imply a rate or tax amount.")
        return {"state": "UNCONFIGURED"}
    required = {"state", "policy_reference", "components"}
    optional = {"classification_reference", "treatment_reference", "rounding_reference"}
    if value.get("state") != "RECORDED" or not required <= value.keys():
        raise ValidationError("Recorded tax requires a policy reference and explicit components.")
    if value.keys() - (required | optional):
        raise ValidationError("Unknown tax evidence fields are not accepted.")
    result = {key: text_value(item) for key, item in value.items() if key != "components"}
    if not isinstance(value["components"], list):
        raise ValidationError("Tax components must be a list.")
    components = []
    codes = set()
    for component in value["components"]:
        keys = {"code", "rate", "rate_unit", "taxable_amount", "amount"}
        if not isinstance(component, dict) or set(component) != keys:
            raise ValidationError(
                "Tax components require code, rate/unit, taxable amount and amount."
            )
        code = text_value(component["code"])
        if code in codes:
            raise ValidationError("Tax component codes must be unique.")
        codes.add(code)
        components.append(
            {
                "code": code,
                "rate": decimal_text(component["rate"]),
                "rate_unit": text_value(component["rate_unit"]),
                "taxable_amount": decimal_text(component["taxable_amount"]),
                "amount": decimal_text(component["amount"]),
            }
        )
    result["components"] = components
    return result


@dataclass(frozen=True, slots=True, init=False)
# REFERENCE NOTE (documentation only; not present in application source):
# Frozen value object, not a Django model/table. Stores an independently serialized copy of
# supplied catalogue, price and tax evidence. It is not an invoice, payment, calculator or
# persisted sale.
class HistoricalPriceSnapshot:
    """Freeze supplied price/tax facts independently of later catalogue edits.

    This value contract does not persist a sale, commit a price or issue an invoice;
    later transaction models must attach and protect the recorded snapshot.
    Refs: BUSINESS_RULES.md R-27, R-28; DECISIONS.md BD-04, CA-02.
    """

    _serialized: str

    def __init__(self, data: dict):
        required = {
            "schema_version",
            "catalogue",
            "price_revision_id",
            "price_revision",
            "mode",
            "currency",
            "agreed_amount",
            "inputs",
            "tax",
        }
        if not isinstance(data, dict) or set(data) != required:
            raise ValidationError(
                "Snapshot requires the complete versioned price evidence contract."
            )
        if type(data["schema_version"]) is not int or data["schema_version"] != 1:
            raise ValidationError("Unsupported price snapshot schema version.")
        if type(data["price_revision"]) is not int or data["price_revision"] < 1:
            raise ValidationError("Price revision must be a positive integer.")
        if data["mode"] not in ("FIXED", "WEIGHT_BASED"):
            raise ValidationError("Snapshot pricing mode must be FIXED or WEIGHT_BASED.")
        normalized = {
            "schema_version": 1,
            "catalogue": _catalogue(data["catalogue"]),
            "price_revision_id": _identifier(data["price_revision_id"]),
            "price_revision": data["price_revision"],
            "mode": data["mode"],
            "currency": validate_currency(data["currency"]),
            "agreed_amount": decimal_text(data["agreed_amount"]),
            "inputs": validate_pricing_inputs(data["inputs"]),
            "tax": _tax_evidence(data["tax"]),
        }
        object.__setattr__(
            self, "_serialized", json.dumps(normalized, sort_keys=True, allow_nan=False)
        )

    def to_dict(self) -> dict:
        """Return an independent JSON-compatible copy for a future immutable record."""
        return json.loads(self._serialized)
```

<a id="database-safeguards"></a>

## Database Safeguards Outside the Models

This document copies model definitions, not all migration SQL or workflow services. Reading only `save()`/`clean()` would miss part of the actual integrity boundary. These existing migrations are relevant:

| Migration | Additional protection / limit |
| --- | --- |
| [compliance/0002_foundation_integrity.py](../arkatara_backend/compliance/migrations/0002_foundation_integrity.py) | Audit history database enforcement; inspect the migration for the exact installed checks/guard |
| [catalog/0003_price_history_guard.py](../arkatara_backend/catalog/migrations/0003_price_history_guard.py) | Append-only price evidence |
| [catalog/0005_import_history_guard.py](../arkatara_backend/catalog/migrations/0005_import_history_guard.py) | Append-only import evidence |
| [inventory/0003_movement_history_guard.py](../arkatara_backend/inventory/migrations/0003_movement_history_guard.py) | Append-only inventory movement evidence |
| [inventory/0004_operational_downgrade_guard.py](../arkatara_backend/inventory/migrations/0004_operational_downgrade_guard.py) | Refuses unsafe removal of operational inventory evidence/protection |
| [accounts/0003_customer_identity_guard.py](../arkatara_backend/accounts/migrations/0003_customer_identity_guard.py) | Retained customer identity and irreversible closure; nonempty downgrade guard |
| [compliance/0004_policy_history_guard.py](../arkatara_backend/compliance/migrations/0004_policy_history_guard.py) | Append-only policy history; nonempty downgrade guard |
| [trials/0002_booking_evidence_guards.py](../arkatara_backend/trials/migrations/0002_booking_evidence_guards.py) | Immutable booking ownership/snapshots, retained boxes/acceptances, challenge binding/outcomes and guest grant identity/revocation; nonempty downgrade guard |
| [delivery/0003_assignment_evidence_guards.py](../arkatara_backend/delivery/migrations/0003_assignment_evidence_guards.py) | Assignment history and Arrival OTP challenge guard; nonempty downgrade guard |

Initial/schema migrations also create the fields, foreign keys, unique constraints, checks and indexes shown above. ORM-only protections on older reference/publication/media/configuration paths must not be described as universal SQL immutability. Privileged database operators can disable triggers or truncate tables; restricted runtime permissions and reviewed operations remain essential. Retention guards do not decide lawful retention/erasure policies.

<a id="framework-tables"></a>

## Framework-Owned Tables

These installed Django models are not app-owned business models and their framework source is not duplicated here. StaffUser already lists its inherited AbstractUser fields above.

| Model | Table | Stored fields / responsibility |
| --- | --- | --- |
| admin.LogEntry | django_admin_log | id, action_time, user, content_type, object_id, object_repr, action_flag, change_message; Admin activity history, not a substitute for domain AuditEvent |
| auth.Permission | auth_permission | id, name, content_type, codename; Django permission definitions |
| auth.Group | auth_group | id, name, permissions M2M; staff permission groups |
| contenttypes.ContentType | django_content_type | id, app_label, model; Django model-type registry |
| sessions.Session | django_session | session_key, session_data, expire_date; framework session storage, not implemented CustomerAccount authentication |

Generated many-to-many join tables connect StaffUser to auth.Group/auth.Permission and auth.Group to auth.Permission. Django's migration recorder is infrastructure, not a customer/order model. Reverse relationship accessors do not imply more business tables. No additional custom business model was omitted from the current application registry.

## Snapshot Maintenance and Verification

This file is a reading aid requested by the owner. Actual code and canonical decisions take precedence after any change. Refresh the source blocks and registry-derived field inventories together; do not edit copied classes here to implement a feature.

The snapshot is checked for coverage of every currently registered project model, inclusion of inherited fields, preservation of original source/comments, Python syntax of each fenced source block, and valid local reference paths. No database migration, provider action, application-code edit or Task 4 acceptance is performed by creating it. Existing tests recorded at the implementation checkpoint are not rerun or re-certified merely by this documentation copy.
