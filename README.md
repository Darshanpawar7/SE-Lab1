# Vaccination Cohort & Dose Scheduling System

**PES University — Dept. of CSE**
**Lab 1: Requirements Engineering & UML Use-Case Modelling**
**Problem Statement #17 | Healthcare & Telemedicine**

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
| 2 | UML Use-Case Diagram (with `<<include>>` and `<<extend>>`) | [uml/use-case-diagram.png](uml/use-case-diagram.png) |
| 3 | Use-Case Flow Specification (UC-01: Book Vaccination Slot) | [use-case-flow/book-vaccination-slot.md](use-case-flow/book-vaccination-slot.md) |

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
- **Citizen Registrant** — registers, books vaccination slots, views status and certificate.
- **Vaccination Officer** — manages centre slots, records administered doses, generates and verifies certificates.

**Key relationships modelled**
- `Book Vaccination Slot` **«include»** `Check Dose Eligibility` — the 28-day minimum dose
  interval check is *mandatory* on every booking attempt (traces to **FR-003**).
- `Record Vaccination Dose` **«include»** `Generate Vaccination Certificate` — recording a dose
  mandatorily issues the certificate for the completed schedule (traces to **FR-005**).
- `Download QR Certificate` **«extend»** `View Vaccination Certificate` — downloading the
  signed QR certificate is *optional* behaviour on top of viewing it (traces to **FR-005**).

---

## Note

This lab is a **requirements specification and UML use-case modelling** exercise.
No application implementation is required or included.
