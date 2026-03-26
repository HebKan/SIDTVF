# SIDTVF Simulation Layer — Layer 7

> **Version:** 1.0 | **Status:** Active | **Last Updated:** 2026-03-26

---

## What Is the Simulation Layer?

The Simulation Layer provides **safe, authorized, behavioral simulation tests** that reproduce the observable log artifacts of insider threat activity — without using attack tooling, exploit code, or real sensitive data.

This is Layer 7 of the SIDTVF framework:

```
Layer 1 — Scenario          Define the threat behavior
Layer 2 — Validation        Test whether controls detect it (procedural)
Layer 3 — Threat Hunting    Search for real activity
Layer 4 — Integration       Feed findings downstream
Layer 5 — Mapping           Standardize to ATT&CK and compliance
Layer 6 — Feedback Loop     Continuously improve
Layer 7 — Simulation        Generate realistic behavioral artifacts to validate detections end-to-end
```

---

## Why a Simulation Layer?

Existing adversary simulation frameworks (Atomic Red Team, CALDERA, Cobalt Strike) are built around **external adversary TTPs** — they execute techniques using exploit code, C2 channels, and attack tooling. They are designed to answer: "Can this attack technique execute on this endpoint?"

Insider threat is fundamentally different:

| Dimension | Adversary Simulation | Insider Threat Simulation |
|-----------|---------------------|--------------------------|
| Tools used | Malware, exploits, C2 tools | Legitimate OS/cloud tools |
| Access method | Unauthorized exploitation | Authorized credentials |
| Detection signal | Malicious tool signature or behavior | Volume, timing, sequence, target selection |
| Test environment | Isolated lab preferred | Near-production required for realistic logs |
| Time dimension | Single event (seconds to minutes) | Pattern over days to weeks |
| Cross-system correlation | Single endpoint or network | Email + file system + network + HR data |

SIDTVF Simulation tests answer a different question: **"Does your detection pipeline produce the correct alert when an authorized user behaves in a way that matches a defined insider threat scenario?"**

---

## How Simulation Tests Differ from Validation Test Cases

Both Layer 2 (Validation) and Layer 7 (Simulation) test detection coverage. They are complementary:

| Aspect | Validation Test Case (TC-XXX) | Simulation Test (SIM-XXX) |
|--------|-------------------------------|---------------------------|
| **Purpose** | Verify the detection rule logic is correctly written | Verify the full pipeline from user behavior → log → SIEM → alert fires |
| **Method** | Often uses synthetic log injection or direct rule testing | Executes actual user actions using real tools in test environment |
| **Realism** | Lower — tests the rule, not the environment | Higher — tests the detection, the data pipeline, and the rule |
| **Prerequisites** | Detection rule deployed | Full logging stack deployed and test environment available |
| **Who runs it** | Detection engineer | Red team or purple team with SOC observer |
| **Output** | PASS/FAIL on rule logic | PASS/FAIL on end-to-end alert generation |

---

## Simulation Test Design Principles

### 1. Authorized Environments Only
Every simulation test must be explicitly authorized before execution. Use the [Authorization Checklist](authorization-checklist.md). Running simulation tests in a production environment without authorization constitutes unauthorized computer activity under most jurisdictions.

### 2. Behavior Over Tooling
Simulation tests use native, legitimate tools — the same tools a real user would use. They do not require attack frameworks, custom scripts, or malware. A cloud storage upload simulation uses the standard web browser or sync client. An email forwarding simulation uses the standard mail client.

### 3. Synthetic Data Only
All simulation tests use synthetic data: fake names, fake account numbers, fake source code, fake medical records. Never use real PII, PHI, financial data, or intellectual property — even in a test environment. Create realistic-looking synthetic datasets using the guidance in each test file.

### 4. Observable Artifacts Required
Every simulation step must produce a log entry. If a step does not produce a log entry that your detection pipeline can see, the step is useless for validation. Each test file specifies the exact log source and event that should appear.

### 5. Cleanup Is Mandatory
Every simulation test includes a mandatory cleanup section. All test accounts, forwarding rules, files, and configuration changes must be removed after the test. Incomplete cleanup leaves persistent security risks.

### 6. Time Compression Is Acceptable
Some scenarios involve behavior that naturally occurs over days or weeks (e.g., gradual data staging over 2 weeks before departure). Simulation tests allow **time compression** — running all steps in a single session — as long as the behavioral artifacts are clearly documented as simulated.

---

## Simulation Test ID Convention

Format: `SIM-[SCENARIO_ID]`

Examples:
- `SIM-DE-001` — Simulation for cloud storage upload exfiltration
- `SIM-DE-002` — Simulation for email auto-forwarding rule
- `SIM-SB-001` — Simulation for targeted database deletion

File naming: `SIM-[SCENARIO_ID].md` (e.g., `simulation/tests/SIM-DE-001.md`)

---

## Simulation Test Catalog

| ID | Scenario | Status | Linked Detection | Prerequisites |
|----|----------|--------|-----------------|---------------|
| SIM-DE-001 | Cloud Storage Upload Exfiltration | Active | DET-DE-001 | Web proxy, endpoint agent |
| SIM-DE-002 | Email Auto-Forwarding Rule | Active | DET-DE-002 | M365 Unified Audit Log |
| SIM-SB-001 | Targeted Database Deletion | Active | DET-SB-001 | Database audit logging, SIEM |
| SIM-PM-001 | Privilege Escalation via Access Abuse | Planned | DET-PM-001 | AD audit logging, UEBA |
| SIM-AC-001 | Account Compromise by External Actor | Planned | DET-AC-001 | Entra ID sign-in logs, Sentinel |

---

## Quick Start: Running Your First Simulation

1. **Read the Authorization Checklist** — [simulation/authorization-checklist.md](authorization-checklist.md) — and get required approvals before proceeding.
2. **Set up a test environment** — see the Prerequisites section of the specific test file.
3. **Create synthetic test data** — follow the data creation guidance in the test file.
4. **Execute the simulation steps** — the test file provides step-by-step instructions.
5. **Verify log generation** — check that each step produced the expected log artifact.
6. **Observe the SIEM** — wait for the detection to fire and document the result.
7. **Complete cleanup** — run all cleanup steps. Do not skip.
8. **Record the result** — log the test execution in the simulation test file's Run Log.

---

## Relationship to Atomic Red Team

Atomic Red Team tests can be used alongside SIDTVF Simulation tests. When a scenario has a direct Atomic Red Team test number, the simulation test file will reference it. However, most SIDTVF simulation tests **do not have Atomic Red Team equivalents** because Atomic Red Team does not cover behavioral patterns that use legitimate tools and authorized access.

For scenarios that have both:
- Run the Atomic Red Team test to validate technique-level detection (e.g., "does your EDR detect archive creation?")
- Run the SIDTVF simulation test to validate pipeline-level detection (e.g., "does your SIEM produce an insider threat alert when the full behavioral pattern is present?")

---

## Changelog

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-03-26 | Initial release of Simulation Layer |
