# Vaccination Cohort & Dose Scheduling System

**PES University — Dept. of CSE**
**Lab 1: Requirements Engineering & UML Use-Case Modelling**
**Problem Statement #17 | Healthcare & Telemedicine**

**NAME: DARSHAN P PAWAR**
**SRN: PES2UG24CS143**
**SECTION: C**
---

## Problem Context & Overview

A public health administration platform to organize demographic vaccination rollouts,
enforce minimum interval days between doses, schedule center slots, and generate
verifiable QR digital vaccination certificates.

**Target Stakeholders / Actors:** Citizen Registrant, Vaccination Officer

---

## Deliverables

| # | Deliverable | Location |
|---|-------------|----------|
| 1 | Complete Requirements Table (FR-001 to FR-005, NFR-001 & NFR-002) | [requirements/requirements.md](requirements/requirements.md) |
| 2 | UML Use-Case Diagram (with `«include»` and `«extend»`) | [uml/use-case-diagram.png](uml/use-case-diagram.png) |
| 3 | Use-Case Flow Specification (UC-01: Book Vaccination Slot) | [use-case-flow/book-vaccination-slot.md](use-case-flow/book-vaccination-slot.md) |

---

## UML Use-Case Diagram

![UML use-case diagram for the Vaccination Cohort & Dose Scheduling System, showing the Citizen Registrant and Vaccination Officer actors, nine use cases inside the system boundary, two include relationships into Check Dose Eligibility, and one extend relationship onto View Vaccination Certificate](uml/use-case-diagram.png)

Editable source: [uml/use-case-diagram.drawio](uml/use-case-diagram.drawio) — open at
[app.diagrams.net](https://app.diagrams.net/).

---

## Repository Structure

```text
.
├── README.md
├── docs/
│   └── 17_SE_Lab1_SE_Problem_Statements.pdf   # original problem statement
├── requirements/
│   └── requirements.md                        # 5 FRs + 2 NFRs with full attributes
├── uml/
│   ├── use-case-diagram.drawio                # editable diagrams.net source
│   └── use-case-diagram.png                   # exported diagram
└── use-case-flow/
    └── book-vaccination-slot.md               # UC-01 flow specification
```

---

## Summary of the Model

**Actors**
- **Citizen Registrant** — registers and maintains a profile, books vaccination slots, views and
  downloads the vaccination certificate, and can verify a QR certificate.
- **Vaccination Officer** — manages centre slots, records administered doses, generates
  certificates, and verifies QR certificates.

**Key relationships modelled**
- `Book Vaccination Slot` **«include»** `Check Dose Eligibility` — the 28-day minimum dose
  interval check is *mandatory* on every booking attempt (traces to **FR-003**).
- `Record Vaccination Dose` **«include»** `Check Dose Eligibility` — the same check is
  *mandatory* again before a dose is physically administered (traces to **FR-003**).
- `Download QR Certificate` **«extend»** `View Vaccination Certificate` — downloading the
  signed QR certificate is *optional* behaviour on top of viewing it (traces to **FR-005**).

**How the two actors' use cases connect**

The diagram is a single connected model, not two separate halves:

- `Check Dose Eligibility` is a **shared included use case** — reached by `«include»` from the
  citizen's `Book Vaccination Slot` *and* from the officer's `Record Vaccination Dose`. Factoring
  common behaviour out of two use cases is exactly what `«include»` exists for.
- `Verify QR Certificate` is a **shared use case** associated with *both* actors — a citizen can
  check their own certificate and an officer can verify one at a checkpoint.

---

## Note

This lab is a **requirements specification and UML use-case modelling** exercise.
No application implementation is required or included.
