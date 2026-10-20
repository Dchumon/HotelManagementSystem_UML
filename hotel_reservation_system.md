# 🏨 Sunrise Hotel Reservation System — Object-Oriented Analysis & Design (OOAD)

An end-to-end Object-Oriented Analysis and Design (OOAD) specification for the **Sunrise Hotel Reservation System**, modeled using standard Unified Modeling Language (UML) specifications in Visual Paradigm.

---

## 📌 Project Overview

The **Sunrise Hotel Reservation System** is an enterprise-level hospitality management platform designed to automate guest bookings, streamline multi-tier administrative workflows, handle room inventory dynamically, and integrate third-party payment gateways.

### Key Objectives
* **Self-Service Guest Portal:** Room discovery, availability checking, reservation customization, secure payment transactions, and feedback management.
* **Administrative Control Hub:** Inventory management (CRUD operations on rooms), dynamic/seasonal pricing configurations, reservation handling, check-in/out workflows, and automated reporting.
* **Asynchronous Notifications & Billing:** Real-time invoice creation, receipt generation, and multi-channel notifications (Email/SMS).

---

## 📑 Table of Contents
1. [Use Case Diagram & Narration](#1-use-case-diagram--narration)
2. [Class Diagram & Structural Architecture](#2-class-diagram--structural-architecture)
3. [Sequence Diagrams & Interaction Flows](#3-sequence-diagrams--interaction-flows)
   - [Guest Lifecycle Flow](#31-guest-booking--checkout-interaction-flow)
   - [Admin Operational Flow](#32-administrative-management-interaction-flow)
4. [Activity Diagram & Workflow Logic](#4-activity-diagram--workflow-logic)
5. [Technical Notation Summary](#5-technical-notation-summary)

---

## 1. Use Case Diagram & Narration

The Use Case Diagram defines the behavioral requirements and functional boundaries between external actors and system capabilities.

![Use Case Diagram](./docs/use_case_diagram.png)

### Actors Involved
* **Guest (Primary Actor):** An external end-user who explores offerings, creates bookings, pays for reservations, and submits feedback.
* **Admin (Primary/Internal Actor):** Hotel staff and management responsible for operational workflows, room inventories, pricing structures, and analytical reporting.
* **Payment Gateway (Secondary/System Actor):** External third-party financial processor validating transactions and initiating digital invoicing.

### Boundary Breakdown & Core Use Cases
* **Guest Boundary:**
  * `Browse hotel branches`, `Check Room Availability`, `Compare room Type & prices`, and `View amenities`.
  * `Register/Login`, `Manages Profile`, `Save Payment Methods`.
  * `Make reservation` — includes mandatory checkouts and payment processing; extended by optional `Modify Booking` and `Cancel Booking`.
  * `Checkout` — includes `Complete payment` and `View bill`; extended optionally by `Provide ratings/feedback`.
* **Admin Boundary:**
  * `Manage room inventory` (includes `Create room`, `Define room types`, `Update room`, `Remove room information`).
  * `Manage Pricing/Promotion` (includes `Set seasonal rate`, `Set pricing rules`, `Configure promotions/discounts`).
  * `Manage reservation` (includes `Confirm bookings`, `Assign rooms`; extended by `Modify bookings` and `Cancel bookings`).
  * `Perform guest check in/out`, `Process Guest service request`, `Record payments`, `Update room status`, and `Generates reports`.
* **System Boundary:**
  * `Process Online Payment transaction`, `Auto-generates confirmation & invoice`, `Send email/Notification`, and `Enforce access control`.

### UML Relationship Semantics
* **`<<include>>` (Mandatory Inclusions):** Denotes non-negotiable sub-routines. For example, `Checkout` unconditionally requires `Complete payment` and `View bill`.
* **`<<extend>>` (Conditional Extensions):** Denotes optional or contingency-based workflows. For instance, `Make reservation` is extended by `Modify Booking` or `Cancel Booking` only when triggered by guest request.

---

## 2. Class Diagram & Structural Architecture

The Class Diagram establishes the static structural schema, capturing attributes, operations, visibility modifiers, multiplicity constraints, and architectural relationships.

![Class Diagram](./docs/class_diagram.png)

### Key Relationships & Multiplicities

| Source Entity | Relationship Type | Target Entity | Multiplicity | Architectural Rationale |
| :--- | :--- | :--- | :--- | :--- |
| `User` | **Generalization (Inheritance)** | `Admin`, `Guest` | `1 : 1` | Encapsulates shared identity fields (`userId`, `username`, `email`, `phone`, `password`) and common access routines (`login()`, `register()`). |
| `Hotel Branch` | **Composition** | `Room` | `1 : 1..*` | A strict life-cycle dependency: rooms cannot exist independently of a physical hotel branch entity. |
| `Room` | **Association** | `Pricing` | `1..* : 1` | Decouples dynamic pricing logic (seasonal surcharges, discounts) from physical room states. |
| `Guest` | **Association** | `Reservation` | `1 : 0..*` | A registered guest can exist with zero bookings or maintain multiple reservation records over time. |
| `Reservation` | **Association** | `Room` | `0..* : 1..*` | Maps reserved inventory to physical accommodations upon assignment. |
| `Payment` | **Dependency / Trigger** | `Invoice`, `Notification` | `1 : 1` | Successful settlement deterministically generates an invoice record and fires notification hooks. |
| `Guest` | **Association** | `Feedback`, `Service Request` | `1 : 0..*` | Guests can raise zero or more service incidents and submit reviews post-stay. |

---

## 3. Sequence Diagrams & Interaction Flows

The Sequence Diagrams capture the dynamic runtime collaboration of lifelines over time through synchronous requests, asynchronous dispatching, and conditional frames.

### 3.1 Guest Booking & Checkout Interaction Flow

Models end-to-end guest operations from authentication and real-time room queries to payment verification, stay requests, and departure.

![Guest Sequence Diagram](./docs/guest_sequence_diagram.png)

#### Dynamic Interaction Sequence
1. **Authentication:** Guest inputs credentials; `System` validates against `Guest Database` (`1.1`–`1.3`).
2. **Room Discovery:** Guest initiates an availability search; system queries `Room` lifelines and presents options (`3.1`–`3.6`).
3. **Reservation Evaluation (`alt` Fragment):**
   * *Room Available:* System issues `create reservation` on the `Reservation` lifeline and returns confirmation.
   * *Room Unavailable:* System routes a termination fault back to the interface.
4. **Financial Settlement (`alt` Fragment):**
   * System invokes `process payment` against the external `Payment Gateway`.
   * *Success Branch:* Payment confirms $\rightarrow$ triggers `generate invoice` on `Invoice` $\rightarrow$ dispatches email/SMS alerts via `Notification`.
   * *Failure Branch:* Emits payment rejection alert for guest retry.
5. **In-Stay & Checkout:** Guest logs a `Service Request` during accommodation; checkout retrieves invoice totals, transitions room states to dirty/maintenance, and enters an `opt` fragment for post-stay reviews.

---

### 3.2 Administrative Management Interaction Flow

Models internal backend administration covering room lifecycle management, dynamic pricing engines, reservation overrides, and operational reporting.

![Admin Sequence Diagram](./docs/admin_sequence_diagram.png)

#### Dynamic Interaction Sequence
1. **Inventory Manipulation (`opt` Fragments):** Admin creates, updates, or deletes room inventory via `Hotel Database` and `Room` lifelines.
2. **Dynamic Pricing Control:** Admin configures seasonal modifiers, floor rates, and discount multipliers within the `Pricing` module.
3. **Booking Adjudication (`alt` Fragment):** Admin inspects pending bookings; confirms and assigns rooms, or rejects and initiates automated cancellations.
4. **Front-Desk Check-In/Out (`alt` Fragment):** Validates reservation states, grants guest access, updates room occupancy, collects settlement, and triggers receipts.
5. **Analytical Reporting:** Admin queries operational metrics; `Report` lifeline aggregates database statistics (occupancy, revenue, maintenance) and renders structured summaries.

---

## 4. Activity Diagram & Workflow Logic

The Activity Diagram models systemic control flow across four designated swimlanes: **Guest**, **Reservation System**, **Payment Gateway**, and **Admin**.

![Activity Diagram](./docs/activity_diagram.png)

```
[Start Node]
     │
     ▼
[Guest: Login/Register] ──► [System: Validate Input]
                                   │
                    ┌──────────────┴──────────────┐
              [Invalid]                         [Valid]
                    │                              │
                    ▼                              ▼
          [Return Error Prompt]          [Update Profile / Search]
                                                   │
                                                   ▼
                                         [Check Room Vacancy]
                                                   │
                                    ┌──────────────┴──────────────┐
                              [Unavailable]                   [Available]
                                    │                              │
                                    ▼                              ▼
                             [Loop back to Search]          [Select Room & Pay]
                                                                   │
                                                                   ▼
                                                       [Payment Gateway Validation]
                                                                   │
                                                    ┌──────────────┴──────────────┐
                                                [Failed]                      [Success]
                                                    │                              │
                                                    ▼                              ▼
                                             [Prompt Retry]            ======= FORK NODE =======
                                                                       ├── Generate Invoice
                                                                       ├── Save Booking to DB
                                                                       └── Send Email/SMS Notification
                                                                       ======= JOIN NODE =======
                                                                                   │
                                                                                   ▼
                                                                        [Admin Check-in & Stay]
                                                                                   │
                                                                                   ▼
                                                                        [Admin Checkout & Bill]
                                                                                   │
                                                                                   ▼
                                                                           [Fork: Bill & Review]
                                                                                   │
                                                                                   ▼
                                                                              [End Node]
```

### Key Workflow Highlights
* **Guard Conditions:** Branch decisions dictate execution flow (e.g., `[room available?]`, `[payment successful?]`, `[reservation valid]`).
* **Concurrency (Fork & Join):** Upon transaction clearance, a UML fork node spawns concurrent operations: generating the invoice, writing booking records to the database, and dispatching multi-channel notifications before synchronizing at a join node.
* **Swimlane Responsibilities:** Maintains clean segregation between presentation-tier triggers, backend business logic, payment service endpoints, and front-desk administrative approvals.

---

## 5. Technical Notation Summary

| UML Mechanism | Notation Style | Applied Domain Use Case |
| :--- | :--- | :--- |
| **Generalization** | Solid line with hollow arrowhead | `Admin` & `Guest` inheriting from `User` |
| **Composition** | Solid line with filled diamond | `Hotel Branch` owning `Room` |
| **Association** | Solid directed line with multiplicity labels | `Guest` making `Reservation` (`1` to `0..*`) |
| **Include Relationship** | Dashed arrow labelled `<<include>>` | Mandatory steps (e.g., `Checkout` $\rightarrow$ `Complete payment`) |
| **Extend Relationship** | Dashed arrow labelled `<<extend>>` | Optional branches (e.g., `Make reservation` $\leftarrow$ `Cancel Booking`) |
| **Combined Fragments** | `alt` (alternative), `opt` (optional) | Conditional execution branches in Sequence lifelines |
| **Parallelization** | Heavy solid synchronization bars (Fork/Join) | Concurrent transaction handling in Activity workflows |

---

## 🛠 Tools & Specifications
* **Modeling Software:** Visual Paradigm Community Edition
* **Standard:** Unified Modeling Language (UML) 2.5
* **Target Architecture:** Object-Oriented Multi-tier Hotel Management Architecture