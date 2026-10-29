# Embedded QA Portfolio 

Ласкаво просимо до мого портфоліо з автоматизації тестування вбудованих систем!

##  Структура репозиторію

*  [README.md](README.md) — Опис проєкту, цілі, підхід, як користуватись.
*  [master-test-plan.md](master-test-plan.md) — Повний план тестування, критерії, покриття.
*  [wokwi/](wokwi/) — Симуляції схем, проєкти та конфігурації Wokwi.
*  [pulseview/](pulseview/) — Захоплення та аналіз сигналів (логічний аналіз).
*  [pytest/](pytest/) — Автотести на Python (unit, integration, CLI тощо).
*  [network-stress/](network-stress/) — Сценарії стрес-тестів мережі та результати.
*  [bug-reports/](bug-reports/) — Звіти про знайдені дефекти та відтворення.
*  [resume/](resume/) — Резюме та профільні матеріали.

## 🔬 Core Project Focus: Smart IoT Node Validation

The practical foundation of this portfolio centers on the exhaustive quality verification of an **unstable-network resilient IoT Node**. The device is engineered to process telemetry, buffer payloads locally during internet outages, and stream records over an MQTT transport layer. 

### Tested Technical Requirements:
* **[R1–R2] Network State Machine:** Validating autonomous recovery routines and evaluating exponential backoff algorithms to prevent server-side reconnect storms.
* **[R3–R4] Buffer Management:** Stress-testing the local FIFO queue boundaries under an expected strict 8-message cap with a explicit `DROP_OLDEST` overflow policy.
* **[R5–R6] Data Integrity:** Verifying absolute delivery tracking to guarantee zero data drop rates while completely preventing post-reconnect telemetry duplication.
* **[R7–R8] Non-Functional Resilience:** Inspecting low-level runtime safety, executing dynamic heap allocations tracking, and auditing real-time telemetry diagnostics via custom CLI hooks.

## 🛠️ Technical Stack & Tooling

* **Programming & Frameworks:** Python 3.x, `pytest`, `pyserial` (automated CLI command injections and log parsing).
* **Hardware & Simulation:** Wokwi Virtual SoC (ESP32 target environment), Virtual Logic Analyzers.
* **Protocol & Signal Analysis:** PulseView / sigrok (digital signal troubleshooting for UART/MQTT transport bounds).
* **Version Control & CI/CD:** Git, GitHub Actions ecosystem for automated verification sweeps.



## Getting Started (Local Setup)

Follow these simple steps to set up the automation framework on your local machine:

# Step 1: Clone the Repository

Open your terminal and run the command to download the project:
git clone https://github.com
Move inside the project folder:
cd embedded-qa-portfolio

# Step 2: Prepare Python Environment

Create an isolated virtual environment to avoid dependency conflicts:
python -m venv .venv
Activate the environment based on your Operating System:
• For Windows (PowerShell): .venv\Scripts\Activate.ps1
• For Linux / macOS: source .venv/bin/activate

# Step 3: Install Required Dependencies

Ensure your package manager is up-to-date and install the test requirements:
pip install --upgrade pip
pip install -r pytest/requirements.txt

# Step 4: Run the Automated Tests

Execute the entire test suite with clean, short traceback logs:
pytest pytest/ --tb=short -v

##  Professional Contact

* **QA Engineer:** Oleksandr Sultanov
* **GitHub Profile:** [@Sultanovoleksandr](https://github.com/Sultanovoleksandr)
* **LinkedIn:** [Connect with me on LinkedIn](https://www.linkedin.com/in/oleksandr-sultanov/) 