# UC-01: Book Vaccination Slot

**Vaccination Cohort & Dose Scheduling System** — Use-Case Flow Specification
PES University — Dept. of CSE | Lab 1 | Problem Statement #17

| Attribute | Value |
|---|---|
| **Use Case ID** | UC-01 |
| **Use Case Name** | Book Vaccination Slot |
| **Primary Actor** | Citizen Registrant |
| **Supporting Actor** | Vaccination Officer *(publishes the slots being booked)* |
| **Scope** | Vaccination Cohort & Dose Scheduling System |
| **Level** | User goal |
| **Trigger** | The citizen requests an appointment for their next vaccine dose. |
| **Included Use Case** | Check Dose Eligibility — `«include»` (invoked at step 4) |
| **Traces to** | FR-002 (slot booking), FR-003 (minimum dose interval), NFR-002 (access control) |

---

## Preconditions

1. The Citizen Registrant is registered in the system with a valid vaccination profile (**FR-001**).
2. The citizen is authenticated and holds an active session (**NFR-002**).
3. The citizen has not already completed the full dose schedule for the vaccine.
4. At least one Vaccination Officer has published slots with remaining capacity for a future date (**FR-002**).
5. The minimum dose interval for the vaccine is configured in the system (**FR-003**).

---

## Postconditions

**Success guarantee**

1. A vaccination appointment is persisted, linked to the citizen, the centre and the time slot.
2. The selected slot's remaining capacity is decremented by exactly 1.
3. A booking reference number is issued to the citizen.
4. The appointment is visible to the citizen in their booking list, and to the Vaccination
   Officer on the centre's session roster.

**Minimal guarantee** *(if the use case ends unsuccessfully)*

5. No appointment is created and no slot capacity is consumed — the system is left in its prior state.

---

## Main Success Scenario

| # | Actor | Step |
|---|---|---|
| 1 | Citizen | Selects **Book Vaccination Slot** from the vaccination portal. |
| 2 | System | Retrieves the citizen's vaccination profile and complete dose history. |
| 3 | System | Determines the next due dose number for the citizen's vaccine schedule. |
| 4 | System | **`«include»` Check Dose Eligibility** — compares the days elapsed since the last recorded dose against the configured minimum interval for that vaccine (**FR-003**). |
| 5 | System | Confirms the citizen is eligible and proceeds. |
| 6 | Citizen | Selects a preferred vaccination centre and date range. |
| 7 | System | Displays the matching published slots, each with its remaining capacity. |
| 8 | Citizen | Selects one available slot. |
| 9 | System | Re-verifies that the slot still has capacity and reserves it atomically, so two concurrent bookings cannot consume the same unit. |
| 10 | System | Stores the appointment and decrements the slot's remaining capacity by 1. |
| 11 | System | Issues a booking confirmation carrying a unique reference number. |
| 12 | Citizen | Receives the confirmation. The use case ends successfully. |

---

## Alternate Flow

### A1 — Minimum Dose Interval Not Satisfied

**Branches at:** step 5 of the Main Success Scenario.
**Condition:** *Check Dose Eligibility* determines that the interval elapsed since the citizen's
last recorded dose is **less than** the configured minimum — for example, a Dose 2 request made
fewer than **28 days** after Dose 1.

| # | Actor | Step |
|---|---|---|
| 5a.1 | System | Blocks the booking attempt and does not display any bookable slots. |
| 5a.2 | System | Computes the earliest eligible date as *date of last dose + minimum interval*. |
| 5a.3 | System | Informs the citizen that the next dose cannot yet be booked, stating the earliest eligible date. |
| 5a.4 | System | Records the blocked attempt in the audit log. |
| 5a.5 | Citizen | Acknowledges the message. The use case ends without a booking; the citizen may repeat UC-01 on or after the earliest eligible date. |

**Postcondition for A1:** no appointment is created and no slot capacity is consumed;
the citizen's vaccination record is unchanged.

---

## Notes

**Why the eligibility check is an `«include»`.** Step 4 runs on *every* execution of this use
case — there is no path from step 3 to step 6 that skips it. That unconditional dependency is
exactly what the `«include»` relationship expresses in the use-case diagram, and it is what
prevents an unsafe early dose from ever being booked. Modelling it as an `«extend»` would
wrongly imply the safety check is optional.

**The same check is shared with the Vaccination Officer.** `Check Dose Eligibility` is also
`«include»`d by the officer's *Record Vaccination Dose* use case, so the interval rule is
re-evaluated before the vaccine is physically administered — a booking made legitimately in
advance can still turn out to be invalid if the earlier dose is later back-dated or corrected.
That shared `«include»` is what connects the Citizen Registrant's and Vaccination Officer's use
cases into one model rather than two independent halves.

**Relationship to FR-003.** Alternate flow A1 is the runtime behaviour of FR-003's acceptance
criterion *"Pass: Dose 2 booking is blocked if the interval since Dose 1 is < 28 days"*, and the
main success scenario is the behaviour of *"Pass: Dose 2 booking is permitted once 28 days or
more have elapsed."*
