# Simulation Test: SIM-[SCENARIO_ID]

<!-- REMOVE ALL INSTRUCTIONAL COMMENTS BEFORE PUBLISHING -->
<!-- Replace [SCENARIO_ID] with the ID of the linked scenario (e.g., DE-001) -->

| Field | Value |
|-------|-------|
| **Simulation ID** | SIM-[SCENARIO_ID] |
| **Linked Scenario** | [SCENARIO_ID] — [Scenario Name] |
| **Linked Validation** | TC-[SCENARIO_ID] |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | YYYY-MM-DD |
| **Last Updated** | YYYY-MM-DD |
| **Estimated Duration** | [e.g., 30 minutes + 24h observation window] |
| **Required Environment** | [Test / Staging / Production-adjacent] |
| **Minimum Maturity Level** | [1–5] |

---

## Objective

<!-- 1-2 sentences: what behavioral pattern does this simulation reproduce, and what detection outcome should it produce? -->

---

## Authorization

> Before executing this simulation, complete the [Authorization Checklist](../authorization-checklist.md) and obtain all required approvals.

---

## Prerequisites

### Environment
- [ ] [Logging system] is active and ingesting into SIEM
- [ ] [Detection rule DET-SCENARIO_ID] is deployed and enabled
- [ ] A dedicated test user account exists with appropriate permissions: `sim.testuser@yourdomain.com`
- [ ] SOC team has been notified of scheduled simulation window

### Test Data
- [ ] Synthetic dataset created — see "Synthetic Data Setup" section below
- [ ] No real PII, PHI, financial data, or proprietary content is used

### Tools Required
<!-- List only native/legitimate tools. No attack frameworks. -->
- [Tool 1] — [why it's needed]
- [Tool 2] — [why it's needed]

---

## Synthetic Data Setup

<!-- Provide specific instructions for creating synthetic test data. Be concrete.
Examples:
- "Create a folder named 'SIM-TEST-DATA' on the Desktop containing 5 text files totaling >60 MB. Use lorem ipsum generator at [describe local tool]."
- "Create a synthetic patient record CSV with 500 rows of fake names/SSNs using the provided Python snippet."
-->

```
[Data creation instructions or script snippet]
```

---

## Simulation Steps

<!-- Each step must produce a specific log artifact. If it doesn't appear in a log, don't include it. -->

### Phase 1 — [Phase Name]

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 1.1 | [Specific action using a specific tool] | [Log source] | [Event type / field value] |
| 1.2 | | | |

### Phase 2 — [Phase Name]

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 2.1 | | | |

<!-- Add phases as needed. Match the scenario's attack phases. -->

---

## Expected Log Artifacts

After completing all simulation steps, the following log artifacts should be present:

| Log Source | Event Type | Key Field | Expected Value |
|------------|------------|-----------|----------------|
| [SIEM source] | [Event] | [Field name] | [Value or range] |

---

## Expected Detection Output

The following alert(s) should fire within the observation window:

| Detection Rule | Expected Trigger | Time to Alert (Target) |
|---------------|-----------------|------------------------|
| [DET-SCENARIO_ID-PLATFORM] | [What should cause the alert to fire] | [e.g., <10 minutes] |

If no alert fires within the observation window, record as **FAIL** and document in the Gap Analysis section.

---

## Observation Window

After completing the simulation steps, wait **[X hours/minutes]** before declaring PASS or FAIL. Some detections are scheduled (e.g., daily batch queries) and will not fire in real time.

| Detection Type | Observation Window |
|---------------|-------------------|
| Real-time rule | 10 minutes |
| Threshold/volume rule | Up to 1 hour |
| Scheduled/batch query | Up to 24 hours |

---

## Pass / Fail Criteria

| Result | Criteria |
|--------|----------|
| **PASS** | All expected alerts fire within the observation window AND contain the correct user, source, and timestamp fields |
| **PARTIAL** | Some but not all expected alerts fire OR alerts fire with missing/incorrect field data |
| **FAIL** | No alerts fire within the observation window |

---

## Gap Analysis

*Complete if result is PARTIAL or FAIL.*

| Gap | Category | Root Cause | Remediation | Owner | Target Date |
|-----|----------|------------|-------------|-------|-------------|
| | [ ] Missing log source | | | | |
| | [ ] Detection rule not deployed | | | | |
| | [ ] Threshold too high | | | | |
| | [ ] Log pipeline delay | | | | |
| | [ ] Other: ________ | | | | |

---

## Cleanup Steps

> Cleanup is mandatory. Do not mark this test as complete until all cleanup steps are verified.

| Step | Action | Verified By | Date |
|------|--------|-------------|------|
| C1 | Delete all synthetic test data files from test account | | |
| C2 | Remove any forwarding rules, scheduled tasks, or configuration changes created during simulation | | |
| C3 | Disable or delete the test user account (or reset to baseline state) | | |
| C4 | Mark any generated SIEM alerts as simulation-generated in the ticketing system | | |
| C5 | Confirm with SOC that simulation window is closed | | |

---

## Run Log

| Run Date | Analyst | Environment | Result | Alerts Fired | Notes |
|----------|---------|-------------|--------|-------------|-------|
| | | | | | |

---

## Atomic Red Team Reference

<!-- If an Atomic Red Team test covers the core technique, reference it here. -->

| Technique | Atomic Test # | Notes |
|-----------|--------------|-------|
| [T-XXXX] | [# or N/A] | [How ART test complements this simulation, or why there is no equivalent] |

---

## Linked Artifacts

| Type | ID | File |
|------|----|------|
| Scenario | [SCENARIO_ID] | scenarios/[category]/[file].md |
| Validation Test Case | TC-[SCENARIO_ID] | validation/test-cases/TC-[SCENARIO_ID].md |
| Linked Detection (Sigma) | DET-[SCENARIO_ID]-SIGMA | detections/sigma/[file].yml |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | YYYY-MM-DD | | Initial creation |
