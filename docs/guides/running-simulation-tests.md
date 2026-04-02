# Guide: Running Simulation Tests

Simulation tests validate detection coverage by replicating the *behavioral pattern* of an insider threat scenario — volume, timing, sequence, and target — using only legitimate enterprise tools and synthetic data. This guide covers how to run one safely.

---

## How Simulation Differs from Validation

| | Validation Test (TC-[ID]) | Simulation Test (SIM-[ID]) |
|-|--------------------------|---------------------------|
| **Tests** | Whether a specific rule fires | Whether the full detection chain fires across realistic behavior |
| **Environment** | Isolated test environment | Near-production environment for realistic log ingestion |
| **Data volume** | Minimal | Realistic (mimics actual exfiltration volume and timing) |
| **Duration** | Minutes | 30 minutes to several hours |
| **Authorization bar** | Standard change management | Formal written authorization required before any step |

Run validation tests first. Only move to simulation when you need to validate realistic behavioral patterns or cross-system correlation.

---

## Before You Start: Non-Negotiable Requirements

**Authorization checklist** (`simulation/authorization-checklist.md`) must be completed and signed before any simulation step is executed. This is not optional. Running a simulation without authorization exposes your organization to legal and HR risk regardless of technical intent.

The checklist covers:
- Scope authorization (which systems, which accounts, which time window)
- Legal and compliance review
- Change management ticket
- Technical prerequisites
- Post-test cleanup obligations

---

## Synthetic Data Requirement

Every simulation test uses **synthetic data only**. Never use:
- Real employee PII or personal information
- Real PHI, MNPI, or financial data
- Real customer records or production credentials
- Source code from live repositories

Synthetic data must be clearly labeled as test data in the file name or content (e.g., `SYNTHETIC_TEST_DATA_DO_NOT_USE.csv`). This protects you if the data is discovered during the test.

---

## Running a Simulation Test: Step-by-Step

### Step 1 — Read the Simulation Test File
Open `simulation/tests/SIM-[SCENARIO_ID].md`. Review the full test before starting:
- **Authorization reference** — confirm the authorization checklist is complete
- **Prerequisites** — environment, accounts, synthetic data, tools
- **Phases** — simulation tests are broken into phases (staging, execution, correlation)
- **Expected log artifacts** — what logs each phase should produce
- **Pass/fail criteria** — specific rule names, observation windows, required fields
- **Cleanup steps** — mandatory; review these before starting

### Step 2 — Set Up Synthetic Data
Follow the synthetic data setup instructions. Typically this means generating a test file matching the size and format described in the test. The test file often includes a Python snippet or a `dd` command — use it exactly.

Verify the synthetic data is accessible to the test account before proceeding.

### Step 3 — Execute Each Phase
Work through the simulation phases in order. Each phase has a table showing:
- The action to take
- The expected log source
- The expected log event

Record the exact timestamp of each action. You will need this to scope your log queries.

**Do not improvise.** If a step is ambiguous, stop and clarify. Simulation tests are designed to produce specific log artifacts — deviating from the steps changes what gets logged.

### Step 4 — Observe the Detection Window
Wait for the observation window defined in the test. Simulation tests have two observation windows:
- **Real-time** — for rules that evaluate as events arrive
- **Ingestion delay** — for rules that run on scheduled queries after a lag

Check both. Some detections fire immediately; correlation rules may take 30–90 minutes.

### Step 5 — Record Results

For each expected detection in the test's pass/fail criteria table:

| Result | Action |
|--------|--------|
| Detection fired within window | Pass — record alert ID, timestamp, matched fields |
| Detection fired outside window | Partial pass — note the lag; this may affect incident response SLAs |
| Detection did not fire | Fail — document as detection gap in `docs/coverage-matrix.md` |
| Unexpected detections fired | Investigate — may indicate noisy rules needing tuning |

### Step 6 — Cleanup (Mandatory)
Every simulation test has explicit cleanup steps. **These are not optional.** Cleanup includes:
- Deleting synthetic files from all locations (including cloud destinations if used)
- Removing test accounts or resetting them
- Removing any forwarding rules, scheduled tasks, or configuration changes
- Verifying cleanup in the relevant log source

Document cleanup completion with timestamps in the simulation test's run log section.

---

## Blind Spot Documentation

Each simulation test has a **blind spots validated** section. After running the test, confirm which blind spots were validated as expected:
- Which attack vectors did the test exercise?
- Which vectors remained outside the test's scope?
- Did the test surface any unexpected coverage gaps?

Document this in your program's coverage matrix.

---

## Related

- Before simulation — run validation tests: [docs/guides/running-validation-tests.md](running-validation-tests.md)
- Authorization checklist: [simulation/authorization-checklist.md](../../simulation/authorization-checklist.md)
- All simulation tests: [simulation/tests/](../../simulation/tests/)
- Simulation layer overview: [simulation/README.md](../../simulation/README.md)
