# Master Test Plan

## 1. Document Purpose
### 1.1 Product Overview
The target of testing is an **IoT Node (Embedded Device)** firmware that collects telemetry and transmits data to a remote MQTT broker. The device features a Command Line Interface (CLI), a local offline buffer for data preservation, and automatic reconnection logic with exponential backoff.

### 1.2 Decision Support
This Master Test Plan defines the verification strategy for device stability under degraded network conditions (latency, data drops, prolonged offline states). It directly supports the **Go/No-Go production release decision** by verifying:
* **Data Integrity:** Zero telemetry loss during connection drops.
* **Network Reliability:** Safe reconnection behavior (no server flooding).
* **System Stability:** Stable RAM utilization under high stress without memory leaks.

## 2. System Overview
### 2.1 Hardware
* **Microcontroller Unit (MCU):** Generic 32-bit SoC (simulated via Wokwi environment).
* **Network Module:** Integrated Wi-Fi chip for local wireless connectivity.
* **Peripherals:** Built-in hardware timers for backoff intervals and system ticks.

### 2.2 Firmware
* **Architecture:** Event-driven firmware with non-blocking execution loops.
* **Core Modules:**
  * `CLI Handler` — Processes input commands and triggers system updates.
  * `Network Manager` — Controls connection states and reconnection loops.
  * `Buffer Manager` — Manages the FIFO queue and limits offline data storage.

### 2.3 Interfaces
* **Control Interface:** UART-based Command Line Interface (CLI) for manual configuration, status retrieval (`STATUS`), and stress injection (`STRESS`).
* **Communication Interface:** TCP/IP stack over Wi-Fi implementing the MQTT protocol transport layer.

### 2.4 External Dependencies
* **MQTT Broker:** Remote server destination (e.g., AWS IoT, HiveMQ, or local Mosquitto instance) handling telemetry publication.
* **Network Infrastructure:** Local Wi-Fi Access Point acting as the gateway to the internet.

### 2.5 Critical Data / Actuators
* **Critical Data:** 
  * Telemetry message payload with unique sequential Message IDs.
  * System state flags (`CONNECTED`, `BACKOFF`, `CONNECTING`, `DISCONNECTED`).
  * Free Heap memory counters tracked during network stress events.
* **Actuators:** Virtual network switch (simulated via `NET DOWN` / `NET UP` triggers) controlling the hardware network interface state.

## 3. Quality Objectives
The testing process focuses on validating four critical quality attributes essential for a production-ready IoT device operating in unstable environments:

### 3.1 Reliability & Fault Tolerance
* **Goal:** The device must gracefully handle unexpected network disconnects and autonomously recover without human intervention.
* **Criterion:** Upon connection loss, the device must safely transition to a `BACKOFF` state and execute a controlled retry algorithm.

### 3.2 Data Integrity (Zero Loss & Multi-delivery Prevention)
* **Goal:** Ensure telemetry data is neither permanently lost during offline states nor duplicated upon reconnection.
* **Criterion:** The local buffer must successfully store messages during network drops. After reconnection, exactly one copy of each buffered message must be delivered to the MQTT broker in the correct order.

### 3.3 Resource Optimization & Memory Stability
* **Goal:** The firmware must maintain a stable memory footprint under extreme stress and long-term operations.
* **Criterion:** Zero memory leaks. Dynamic memory allocation (Free Heap) must return to its baseline level after processing heavy bursts of network activity (`STRESS` command).

### 3.4 Scalability & Network Safety
* **Goal:** Prevent the device from overwhelming the backend infrastructure during mass network outages.
* **Criterion:** The reconnection logic must enforce an increasing backoff interval (Exponential Backoff) to completely avoid a "reconnect storm" (flooding the MQTT broker with rapid connection requests).

## 4. Scope
This section defines the precise functional requirements, non-functional characteristics, and interface commands included in this verification cycle.

### 4.1 In Scope
The evaluation covers the core firmware architecture mapped against the following official project requirements:

* **Functional & State Machine Verification:**
  * **[R1] Network States:** Validation of observable state transitions across `DISCONNECTED`, `CONNECTING`, `CONNECTED`, and `BACKOFF` states.
  * **[R2] Auto-reconnect:** Verification of autonomous recovery routines following connection drops, ensuring retry loops implement an increasing backoff interval to prevent a reconnect storm.
  * **[R5] Recovery:** Verification that the node restores the `CONNECTED` state and immediately flushes all accumulated telemetry upon a `NET UP` event.
* **Data & Buffer Management:**
  * **[R3] Offline Buffering:** Validation of local queue reliability for capturing and preserving telemetry payloads when publishing is impossible.
  * **[R4] Buffer Boundary:** Verification of a strict 8-message capacity limit enforcing a mandatory `DROP_OLDEST` overflow policy, with all drops explicitly recorded in system counters and logs.
  * **[R6] Duplicates:** Verification of message delivery constraints, ensuring a single telemetry Message ID is successfully published exactly once (zero multi-delivery allowance).
* **Non-Functional & Observability Metrics:**
  * **[R7] Memory Stability:** Dynamic memory allocation tracking to ensure the Free Heap footprint remains constant under intense workloads with zero systemic, irreversible memory decay.
  * **[R8] Evidence & Diagnostics:** Validation of the comprehensive diagnostic console output, verifying the exposure of network state, buffer occupancy/capacity, sent/dropped/duplicate counts, reconnect attempts, and free heap size.

### 4.2 Fault Injection & Interface Commands
The scope explicitly includes testing the system's runtime resilience using the following interactive CLI triggers:
* `NET DOWN` / `NET UP` — Manual simulation of hardware network link failures and restorations.
* `SEND` — Generation of individual telemetry items to verify nominal and offline queue mechanics.
* `STRESS <N>` — Automated bulk injection of `<N>` rapid telemetry entries to stress-test heap stability.
* `BUFFER` — Inspection of the current offline memory queue layout and message indexing.
* `STATUS` — Comprehensive system diagnostic extraction.
* `RESET STATS` — Counter baseline clearing for isolated scenario tracking.

## 5. Out of Scope
**Hardware Validation:** Physical PCB signal integrity, power consumption profiles, and hardware component thermal characteristics.
**Security Testing:** Firmware reverse-engineering protection, Wi-Fi WPA3 handshake vulnerability scanning, and MQTT TLS certificate validation.
**Production Infrastructure:** Performance, load capacity, and horizontal scaling capabilities of the cloud-hosted production MQTT broker.
**OTA Updates:** Verification of Over-the-Air firmware update processes and fallback rollbacks.

## 6. Assumptions / Constraints
This section outlines the operational constraints, environmental assumptions, and simulation boundaries that influence the execution and validity of this test suite.

### 6.1 Simulation Environment Limitations
* **Virtual Prototyping:** All testing is executed within the Wokwi virtual simulator inside VS Code. It assumes the underlying virtual hardware layer behaves deterministically.
* **Network Stack Abstraction:** The network link layer is simulated via simplified software hooks (`NET DOWN` / `NET UP`). Real-world wireless anomalies, such as radio frequency (RF) attenuation, physical obstacles, fading, and packet corruption, are out of scope.
* **Idealized Server Behavior:** The remote MQTT broker is mocked or accessed under stable conditions, assuming no processing delays or connection drops on the server side.

### 6.2 Tooling and Instrumentation Constraints
* **Digital Signal Capture:** Protocol analysis is dependent on the virtual logic analyzer output and `PulseView` interpretation. Timing resolution and channel configurations are bound to the software sampler limits.
* **Telemetry Monitoring:** Diagnostics rely exclusively on the firmware's internal instrumentation via the `STATUS` command. Metrics like Free Heap are based on the RTOS/allocator implementation within the microcontroller's core framework.

### 6.3 Resource and Hardware Constraints
* **Static Memory Profiles:** The simulated 32-bit MCU possesses fixed RAM limitations. The test plan operates under the assumption that the baseline memory footprint does not fluctuate due to external tasks beyond the firmware scope.
* **Sequential Execution:** The test harness interacts with the device via a single UART command line interface. It assumes no simultaneous multi-threaded race conditions occur on the control line interface itself.

## 7. Quality Risks

| Risk ID | Risk | Impact | Probability | Priority | Mitigation / Test |
|---|---|---|---|---|---|

## 8. Test Levels
- component / firmware function
- interface / protocol
- system
- recovery / fault injection
- network
- regression

## 9. Test Types
- functional
- boundary / negative
- protocol
- reliability
- recovery
- persistence
- network stress
- memory stability
- automation

## 10. Test Environment

| Environment | Purpose | Tools |
|---|---|---|
| Wokwi | Firmware/system simulation | ESP32 simulator |
| Logic analysis | Protocol evidence | PulseView / analyzer |
| Host automation | Regression | Python / Pytest |
| Real hardware | Electrical/physical validation | Physical bench |

## 11. Entry Criteria
Умови початку тестування.

## 12. Exit Criteria
Вимірювані умови завершення.

## 13. Regression Strategy
Що запускаємо після кожної значущої зміни.

## 14. Evidence Strategy
Який evidence потрібний для кожного типу finding:
logs, traces, screenshots, JSON reports, pytest output, metrics.

## 15. Defect Management
Severity, reproducibility, evidence, retest.

## 16. Residual Risks
Що залишається неперевіреним / ризиковим.

## 17. Release Recommendation
GO / GO WITH RISKS / NO-GO та аргументація.