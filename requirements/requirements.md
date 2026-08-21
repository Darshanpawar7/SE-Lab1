# Requirements Specification

**Vaccination Cohort & Dose Scheduling System**

PES University — Dept. of CSE
Lab 1: Requirements Engineering & UML Use-Case Modelling
Problem Statement #17 | Healthcare & Telemedicine

---

## 1. Scope

A public health administration platform to organize demographic vaccination rollouts,
enforce minimum interval days between doses, schedule center slots, and generate
verifiable QR digital vaccination certificates.

**Actors:** Citizen Registrant, Vaccination Officer

---

## 2. Requirements Table

Five functional requirements (FR-001 – FR-005) and two non-functional requirements
(NFR-001, NFR-002), each with ID, Type, Description, Priority, Acceptance Criteria and Rationale.

| ID | Type | Description | Priority | Acceptance Criteria | Rationale |
|----|------|-------------|----------|---------------------|-----------|
| **FR-001** | Functional — Registration | The system shall allow a Citizen Registrant to create and maintain a vaccination profile containing demographic and vaccination details. | High | **Pass:** a profile with all mandatory fields is stored and retrievable by its citizen ID. **Fail:** a profile is accepted with a missing/invalid mandatory field, or a duplicate national ID is created. | Cohort rollouts and dose scheduling are driven by demographic data, so a verified citizen profile is the precondition for every other function. |
| **FR-002** | Functional — Scheduling | The system shall let a Citizen Registrant view vaccination-centre slots and book an available one, and let a Vaccination Officer publish and manage those slots. | High | **Pass:** booking an available slot returns a confirmation and decrements that slot's remaining capacity by exactly 1. **Fail:** a slot at zero capacity or in the past can be booked, or two citizens are confirmed into the same unit of capacity. | Centre-level slot scheduling is the mechanism that spreads demand across centres and prevents overcrowding at vaccination sites. |
| **FR-003** | Functional — Business Rule | The system shall enforce the configured minimum interval between doses before permitting a citizen to book a subsequent dose. | High | **Pass:** Dose 2 booking is blocked when interval since Dose 1 < 28 days. **Pass:** Dose 2 booking is permitted on/after day 28. **Fail:** an early vaccination slot is confirmed. | Premature second doses are clinically unsafe and invalidate the vaccination schedule; the rule must be enforced by the system, not left to staff discretion. |
| **FR-004** | Functional — Record Keeping | The system shall allow a Vaccination Officer to record an administered dose (vaccine, dose number, batch, date, centre) against a registered citizen, updating that citizen's vaccination record. | High | **Pass:** a recorded dose appears in the citizen's history and immediately drives FR-003 eligibility. **Fail:** a user without the Vaccination Officer role can create or alter a dose record. | The dose history is the single source of truth for interval eligibility (FR-003) and certificate issuance (FR-005), so it must be complete and write-restricted. |
| **FR-005** | Functional — Certification | The system shall generate a digitally signed, QR-based vaccination certificate from a completed vaccination record, which the citizen can view and download and an officer can verify. | High | **Pass:** a completed schedule yields a QR certificate whose signature verifies against the issuing key and resolves to the correct citizen record. **Fail:** a tampered or forged QR code is reported as valid. | Citizens need portable, tamper-evident proof of vaccination that third parties can trust and check independently. |
| **NFR-001** | Performance & Security | The vaccination certificate verification endpoint shall authenticate digitally signed QR codes, offline and online, and return a verification result in under 150 ms. | High | **Pass:** benchmarking under simulated peak load confirms 150 ms verification latency and the required signature-authentication standard. **Fail:** latency exceeds 150 ms at peak, or an unsigned/invalid code verifies successfully. | Verification happens at centre gates and public checkpoints where queues form; it must be fast enough not to become the bottleneck, yet still cryptographically sound. |
| **NFR-002** | Security — Access Control | The system shall restrict access to citizen vaccination records and officer-only operations to authenticated users holding the appropriate role. | High | **Pass:** an unauthenticated request, or a Citizen Registrant attempting an officer-only operation, is rejected and audit-logged. **Fail:** any vaccination record is readable or writable without an authorised session. | Vaccination records are sensitive personal health data; unauthorised read exposes private information and unauthorised write would corrupt the eligibility and certificate chain. |

**Note on priority.** All seven are rated High because each derives directly from a capability
named in the problem statement — removing any one leaves the system unable to perform its stated
purpose. Genuinely lower-priority items are kept out of this table (see §4).

---

## 3. Detailed Requirement Specifications

### FR-001 — Citizen Registration and Profile Maintenance

| Attribute | Value |
|---|---|
| **Type** | Functional — Registration |
| **Priority** | High |
| **Actor** | Citizen Registrant |
| **Use case** | Register / Maintain Profile |

**Description.** The system shall allow a Citizen Registrant to create a vaccination profile
capturing name, date of birth, government identity number, contact details and cohort-relevant
attributes (age band, comorbidity, occupation category), and to subsequently update the mutable
subset of those details.

**Acceptance Criteria**
- **Pass:** submitting all mandatory fields with valid formats stores the profile and returns a unique citizen ID.
- **Pass:** an existing profile can be updated, and the change is reflected on next retrieval.
- **Fail:** a profile is accepted when a mandatory field is missing or malformed.
- **Fail:** a second profile is created for an already-registered government identity number.

**Rationale.** Demographic attributes determine which vaccination cohort a citizen belongs to,
and every downstream function — slot booking, interval checking, certificate issuance — is keyed
to a registered profile.

---

### FR-002 — Vaccination Slot Publication and Booking

| Attribute | Value |
|---|---|
| **Type** | Functional — Scheduling |
| **Priority** | High |
| **Actors** | Citizen Registrant, Vaccination Officer |
| **Use cases** | Book Vaccination Slot, Manage Vaccination Slots |

**Description.** The system shall allow a Vaccination Officer to publish vaccination-centre slots
with a date, time window, vaccine type and capacity, and allow a Citizen Registrant to browse
slots filtered by centre and date and book one for which they are eligible.

**Acceptance Criteria**
- **Pass:** booking an available slot returns a booking confirmation carrying a reference number.
- **Pass:** the slot's remaining capacity decreases by exactly 1 on confirmation.
- **Fail:** a slot whose remaining capacity is 0 accepts a booking.
- **Fail:** a slot with a start time in the past is offered as bookable.
- **Fail:** two concurrent bookings are both confirmed against the last remaining unit of capacity.

**Rationale.** Slot scheduling distributes demand across centres and time, which is what prevents
crowding at vaccination sites and lets officers plan vaccine stock per session.

---

### FR-003 — Minimum Dose Interval Enforcement

| Attribute | Value |
|---|---|
| **Type** | Functional — Business Rule |
| **Priority** | High |
| **Actors** | Citizen Registrant, Vaccination Officer (enforced by the system for both) |
| **Use case** | Check Dose Eligibility — `«include»` of Book Vaccination Slot *and* of Record Vaccination Dose |

**Description.** The system shall enforce dose interval rules (e.g. minimum 28 days after Dose 1)
before unlocking Dose 2 booking for a citizen. The interval shall be configurable per vaccine
type rather than hard-coded.

**Acceptance Criteria**
- **Pass:** Dose 2 booking is blocked if the interval since Dose 1 is < 28 days.
- **Pass:** Dose 2 booking is permitted once 28 days or more have elapsed.
- **Pass:** when blocked, the system reports the earliest eligible date.
- **Fail:** an early vaccination slot is confirmed.

**Rationale.** A second dose given too early is clinically ineffective and potentially unsafe.
Because the rule is safety-critical, it is enforced centrally rather than left to staff
discretion, and it is checked at **both** points where an invalid dose could slip through: when
the citizen reserves an appointment, and again when the officer records the dose as administered.
Modelling it as a shared mandatory `«include»` from both use cases — rather than an optional
check on one — is what makes it impossible to bypass from either direction.

---

### FR-004 — Vaccination Dose Recording

| Attribute | Value |
|---|---|
| **Type** | Functional — Record Keeping |
| **Priority** | High |
| **Actor** | Vaccination Officer |
| **Use cases** | Record Vaccination Dose |

**Description.** The system shall allow a Vaccination Officer to record the administration of a
dose against a registered citizen, capturing vaccine name, dose number, batch/lot number,
date-time of administration and vaccination centre, and shall update the citizen's cumulative
vaccination record accordingly.

**Acceptance Criteria**
- **Pass:** an officer records a dose with all required fields and it appears in the citizen's history.
- **Pass:** the newly recorded dose immediately governs FR-003 eligibility for the next dose.
- **Fail:** a Citizen Registrant, or any unauthenticated user, creates or modifies a dose record.
- **Fail:** a dose is recorded against a citizen ID that does not exist.

**Rationale.** The dose history is the authoritative input to both interval eligibility and
certificate generation. If it is incomplete or editable by the wrong party, both the safety rule
and the certificate lose their meaning.

---

### FR-005 — QR Vaccination Certificate

| Attribute | Value |
|---|---|
| **Type** | Functional — Certification |
| **Priority** | High |
| **Actors** | Citizen Registrant, Vaccination Officer |
| **Use cases** | Generate Vaccination Certificate, View Vaccination Certificate, Download QR Certificate (`«extend»`), Verify QR Certificate |

**Description.** The system shall generate a digitally signed vaccination certificate carrying a
QR code that encodes the citizen identifier, vaccine, doses administered and issue date. The
citizen may view it and optionally download it; an officer may scan it to verify authenticity.

**Acceptance Criteria**
- **Pass:** a completed vaccination schedule produces a certificate with a scannable QR code.
- **Pass:** scanning the QR code verifies the signature against the issuing key and resolves to the correct vaccination record.
- **Fail:** a QR code whose payload has been altered is reported as valid.
- **Fail:** a certificate is issued for a citizen with no recorded dose.

**Rationale.** Citizens need portable proof of vaccination accepted by parties who cannot access
the system's database directly; a digital signature is what makes that proof trustworthy offline.

---

### NFR-001 — Certificate Verification Performance & Security

| Attribute | Value |
|---|---|
| **Type** | Performance & Security |
| **Priority** | High |
| **Applies to** | Verify QR Certificate |

**Description.** The vaccination certificate verification endpoint shall authenticate digitally
signed QR codes offline and online, returning a verification result in under **150 ms** measured
at the 95th percentile under simulated peak load.

**Acceptance Criteria**
- **Pass:** benchmarking tests confirm the 150 ms target latency and the required security standard under simulated peak load.
- **Pass:** verification succeeds with no network connectivity, using the cached issuing public key.
- **Fail:** 95th-percentile latency exceeds 150 ms at the defined peak concurrency.
- **Fail:** a code signed with a non-issuing key verifies successfully.

**Rationale.** Verification occurs at centre entrances and public checkpoints where people queue.
It must be fast enough not to create a bottleneck and must work offline at rural centres, while
remaining cryptographically sound — speed must not be bought by skipping signature checks.

---

### NFR-002 — Authentication and Role-Based Access Control

| Attribute | Value |
|---|---|
| **Type** | Security — Access Control |
| **Priority** | High |
| **Applies to** | All use cases handling citizen data |

**Description.** The system shall require authentication for all access to citizen vaccination
records, and shall restrict officer-only operations (recording doses, managing slots, generating
certificates) to users holding the Vaccination Officer role.

**Acceptance Criteria**
- **Pass:** an unauthenticated request to any record endpoint is rejected with an authorisation error.
- **Pass:** a Citizen Registrant attempting an officer-only operation is denied, and the attempt is audit-logged.
- **Pass:** a citizen can read their own record and no other citizen's.
- **Fail:** any vaccination record is readable or writable without an authorised session.

**Rationale.** Vaccination records are sensitive personal health data. Unauthorised read exposes
private medical information; unauthorised write would corrupt the dose history that FR-003 and
FR-005 depend on, turning a data-protection failure into a patient-safety one.

---

## 4. Additional Non-Functional Requirement (beyond the two required)

The deliverable specifies NFR-001 and NFR-002. The following is recorded for completeness and is
**not** part of the required set.

### NFR-003 — Availability

| Attribute | Value |
|---|---|
| **Type** | Availability / Reliability |
| **Priority** | Medium |

**Description.** The system shall remain available to citizens and officers for at least **99.5%**
of published operational hours, and shall recover from a service failure within 15 minutes.

**Acceptance Criteria**
- **Pass:** measured uptime over a calendar month is ≥ 99.5% of operational hours.
- **Pass:** a simulated node failure is recovered within 15 minutes with no confirmed booking lost.
- **Fail:** a failure during operational hours causes confirmed bookings to be dropped.

**Rationale.** Vaccination drives run to fixed campaign windows; an outage during a session wastes
both thawed vaccine stock and citizens' scheduled appointments.

---

## 5. Traceability — Requirements to Use Cases

Every requirement maps to at least one use case in
[../uml/use-case-diagram.drawio](../uml/use-case-diagram.drawio), and every use case in the
diagram is labelled with the requirement it satisfies.

| # | Requirement | Use Case | Actor | Relationship shown |
|---|---|---|---|---|
| 1 | FR-001 | Register / Maintain Profile | Citizen Registrant | association |
| 2 | FR-002 | Book Vaccination Slot | Citizen Registrant | association |
| 3 | FR-003 | Check Dose Eligibility | — | **`«include»`** from Book Vaccination Slot **and** from Record Vaccination Dose |
| 4 | FR-005 | View Vaccination Certificate | Citizen Registrant | association |
| 5 | FR-005 | Download QR Certificate | — | **`«extend»`** onto View Vaccination Certificate |
| 6 | FR-002 | Manage Vaccination Slots | Vaccination Officer | association |
| 7 | FR-004 | Record Vaccination Dose | Vaccination Officer | association |
| 8 | FR-005 | Generate Vaccination Certificate | Vaccination Officer | association |
| 9 | FR-005, NFR-001 | Verify QR Certificate | **both actors** | association to Citizen Registrant **and** Vaccination Officer |
| — | NFR-002 | *(cross-cutting)* | both | applies to all record-handling use cases |

---

### 5.1 How the two actors' use cases are connected

The model is a **single connected graph** — not two independent halves that happen to share a
system boundary. Starting from either actor, every use case is reachable. Two relationships do
that work:

**Shared included use case — `Check Dose Eligibility`.**
It is `«include»`d from the citizen's `Book Vaccination Slot` *and* from the officer's
`Record Vaccination Dose`. The interval rule must hold at two distinct moments: when an
appointment is reserved, and again before the vaccine is physically administered — a booking made
legitimately in advance can still become invalid if the earlier dose is back-dated or corrected.
Factoring that common behaviour out of two use cases owned by two different actors is precisely
what `«include»` is for, and it is what links the two halves of the diagram.

**Shared use case — `Verify QR Certificate`.**
Associated with *both* actors: a Citizen Registrant verifies their own certificate before
travelling, and a Vaccination Officer verifies a presented certificate at a checkpoint. Same
system behaviour, two initiating actors, one use case.

**Why `«include»` for FR-003 and `«extend»` for FR-005.**
`Check Dose Eligibility` is `«include»`d because the interval rule is unconditional — modelling
it as optional would let an unsafe booking through. `Download QR Certificate` is an `«extend»`
because a citizen may view a certificate on screen without ever downloading it, so that behaviour
is conditional on the citizen requesting a copy.

---

## 6. Deliverable Checklist

| Requirement of the lab | Status |
|---|---|
| Exactly 5 Functional Requirements (FR-001 – FR-005) | ✅ |
| 2 Non-Functional Requirements (NFR-001 & NFR-002) | ✅ |
| Each has an ID | ✅ |
| Each has a Type | ✅ |
| Each has a Description | ✅ |
| Each has a Priority | ✅ |
| Each has Acceptance Criteria | ✅ |
| Each has a Rationale | ✅ |
