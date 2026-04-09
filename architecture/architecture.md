# System Architecture

This document defines the system architecture of the automated multimeter verification platform, encompassing hardware design, software orchestration, communication protocols, and data management layers.

---

## High-Level System Overview

The platform is structured into four functional layers to ensure modularity and maintainability:

* **Application Layer:** LabVIEW-based control system and user interface.
* **Communication Layer:** Industrial protocols (GPIB, SPI, ODBC).
* **Hardware Layer:** Custom PCB switching matrix and laboratory instrumentation.
* **Data Layer:** PostgreSQL relational database and automated reporting engine.

---

## System Diagram

<div align="center">
  <img src="../docs/images/architecture/system-architecture.svg" width="800" alt="High-level system architecture">
  <p><em>High-level system architecture and inter-layer communication.</em></p>
</div>

---

## Hardware Architecture

The hardware subsystem manages physical signal routing, galvanic isolation, and electrical safety.

### Core Components
* **Control Unit:** Arduino Nano (ATmega328P).
* **Output Expansion:** 74HC595 shift registers for high-density relay control.
* **Isolation:** Optocouplers for digital signal protection.
* **Power Stage:** MOSFET drivers for relay actuation.
* **Switching Matrix:** Configurable relay-based array for multi-instrument routing.

### Operational Logic
The hardware layer facilitates dynamic signal routing between:
* Reference DC/AC voltage and current sources.
* Programmable electronic loads and rheostats.
* Digital Multimeters Under Test (DUT).

<div align="center">
  <img src="../docs/images/architecture/hardware-architecture.svg" width="800" alt="Hardware block diagram">
  <p><em>Hardware block diagram and signal routing logic.</em></p>
</div>

---

## Software Architecture (LabVIEW)

The control software utilizes a **Queued Message Handler (QMH)** design pattern, enabling parallel execution, responsiveness, and asynchronous error handling.

### Modular Components
* **User Interface (UI) Loop:** Manages event-driven user interaction and real-time visualization.
* **Coordinator Loop:** Orchestrates high-level task scheduling and module synchronization.
* **Functional Modules:** Encapsulated logic for business rules and measurement validation.
* **Measurement State Machine:** Manages the deterministic execution of verification sequences.

Each module operates in an independent thread, communicating via thread-safe message queues.

<div align="center">
  <img src="../docs/images/architecture/software-architecture.svg" width="800" alt="LabVIEW software architecture">
  <p><em>LabVIEW parallel message-based architecture.</em></p>
</div>

---

## Data and Reporting Layer

* **Database Engine:** PostgreSQL (Relational DBMS).
* **Server Host:** Raspberry Pi 5.
* **Data Model:** Fully normalized (3rd Normal Form) schema.
* **Objective:** Ensures strict measurement traceability, historical data analysis, and automated compliance reporting.

<div align="center">
  <img src="../docs/images/architecture/database-architecture.svg" width="800" alt="Database schema overview">
  <p><em>Relational database model and data flow.</em></p>
</div>

---

## Communication Protocols

| Interface | Specification | Primary Function |
| :--- | :--- | :--- |
| **GPIB (IEEE-488)** | NI-VISA | Instrument control and data acquisition |
| **SPI / USB** | LINX Interfacing | Low-level relay matrix control via Arduino |
| **LAN (ODBC)** | PostgreSQL Driver | Persistent data storage and querying |
| **CLI / System Exec** | Python Runtime | Triggering of automated PDF reporting scripts |

---

## Process Control Logic

The verification sequence is governed by a Finite State Machine (FSM), ensuring a deterministic transition between connection, stabilization, measurement, and logging phases.

<div align="center">
  <img src="../docs/images/architecture/state-machine.svg" width="500" alt="Execution state machine">
  <p><em>Finite State Machine for automated measurement sequences.</em></p>
</div>

---

## Design Principles

### Modularity
Independent subsystems allow for hardware upgrades (e.g., changing an instrument) or software updates (e.g., new database schema) with minimal impact on the overall system.

### Parallelism and Asynchrony
The LabVIEW architecture ensures the UI remains fully responsive during long-duration measurement cycles, as acquisition and processing run in background threads.

### Scalability and Extensibility
* **New Devices:** Added via database configuration without requiring code changes.
* **New Procedures:** Implemented as discrete states within the measurement FSM.

### Separation of Concerns

| Layer | Functional Responsibility |
| :--- | :--- |
| **Hardware** | Electrical switching and physical signal routing |
| **Communication** | Protocol abstraction (GPIB, SPI, LAN) |
| **Application** | Orchestration, logic, and user interface |
| **Data Storage** | Relational persistence and traceability |
| **Reporting** | Data post-processing and document generation |

---

## Technology Rationale

### Development Tools
* **PCB Design (KiCad):** Selected for its professional-grade features and lack of licensing constraints for academic research.
* **Microcontroller (Arduino Nano):** Chosen for its reliability in simple I/O tasks and seamless integration with LabVIEW via LINX.
* **Database (PostgreSQL):** Selected over file-based systems (CSV/Excel) to ensure relational integrity, multi-user access, and long-term traceability.

---

## Component Roles

* **Primary Controller (PC):** Central orchestration and execution of the LabVIEW application.
* **Relay Controller (Arduino):** Performs low-level switching operations as commanded by the PC.
* **Switching Matrix:** Custom PCB for physical interface between instruments and DUT.
* **Reference Instrumentation:** High-stability HP/Agilent sources and multimeters.
* **Database Host (Raspberry Pi 5):** Dedicated Linux server for measurement persistence.
* **Reporting Engine (Python):** Post-test script for data retrieval and PDF generation.

---

## Summary

The system follows a layered architecture designed for industrial-grade reliability, scalability, and long-term maintainability. By isolating the **Application Layer** (LabVIEW) from the **Data Layer** (PostgreSQL) and the **Physical Layer** (Relay Matrix), the platform achieves a high degree of modularity.

The integration of a **Queued Message Handler (QMH)** ensures a responsive and deterministic control environment, while the use of a relational database guarantees 100% traceability for every measurement. This architectural approach not only meets the current requirements for multimeter verification but also provides a scalable foundation for future instrumentation and complex test procedure expansions.

