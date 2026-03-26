# Validation Cycle Guide

> **Version:** 1.0 | **Last Updated:** 2026-03-26

This guide describes how to plan, execute, document, and act on a SIDTVF validation cycle — the structured process of testing whether deployed detection rules actually fire when their target behaviors are simulated.

---

## What Is a Validation Cycle?

A validation cycle is a scheduled, authorized exercise in which detection rules are tested against their linked scenarios in a controlled environment. It answers the question: *"If this scenario were happening right now, would we know?"*

Validation is distinct from penetration testing. It is not adversarial. It is methodical quality assurance for detection rules.

---

## Cadence

| Trigger | Cycle Type |
|---------|-----------|
| Quarterly schedule | Full cycle — all Active scenarios |
| New detection deployed | Targeted — test the new rule before it goes to production |
| Significant platform change (SIEM migration, proxy replacement, EDR upgrade) | Full cycle — re-validate all rules against the new data pipeline |
| Incident identified a detection gap | Targeted — validate the new or fixed rule |
| ATT&CK version update | Targeted — re-validate rules for any modified techniques |

---

## Step-by-Step Process

### Step 1 — Plan the Cycle

**1a. Scope selection**
For a full quarterly cycle: include all scenarios with `Status: Active` and at least one linked detection with `Status: Active`.
For a targeted cycle: list only the specific detection IDs being validated.

**1b. Obtain authorization**
All validation activities require written authorization. At minimum:
- Security Manager sign-off
- Legal acknowledgment if any test will run in or against production systems
- SOC notification so they do not escalate test alerts as real incidents

Document authorization in the test case file before execution.

**1c. Prepare the test environment**
- Confirm the test user account exists and has appropriate permissions
- Confirm the detection rule is active (not in draft/paused state)
- Confirm the data pipeline is flowing (check last log timestamp for each required source)
- Confirm synthetic test data is ready (non-sensitive, clearly labeled)

### Step 2 — Execute Test Cases

For each test case (`validation/test-cases/TC-[ID].md`):

1. Follow steps exactly as written in the "Test Steps" section
2. Record the start time
3. Execute each step in sequence
4. Record whether the detection fired, the time elapsed, and the field values in the alert
5. Complete the PASS / FAIL / PARTIAL classification
6. Execute cleanup steps before moving to the next test case

**PASS criteria:**
- Detection fired within the expected time window
- Alert contains the correct user, source, and behavioral fields
- No unintended false positives generated for other users

**PARTIAL criteria:**
- Detection fired but fields are missing, incorrect, or time exceeded
- Detection fired for some but not all test sub-cases

**FAIL criteria:**
- No detection fired within 2× the expected time window
- Detection fired for the wrong user or from an incorrect data source

### Step 3 — Document Results

For every execution, update the "Test Results Log" section of the test case file:
- Date, executor, environment, result, time-to-alert, field accuracy
- For FAIL or PARTIAL: complete the Gap Analysis fields (root cause, gap category, remediation, owner, target date)

Gap categories:
- **Missing log source** — required data is not being collected
- **Misconfigured rule** — logic error, wrong field name, incorrect threshold
- **Threshold too high** — rule logic is correct but threshold is set above realistic behavior
- **No rule exists** — scenario is Active but no detection has been built yet
- **Data pipeline failure** — logs are collected but not reaching the SIEM

### Step 4 — Compile the Cycle Report

After all test cases are executed, produce a cycle summary:

```
Validation Cycle Report
Period: [Quarter] [Year]
Executed By: [Name]
Authorization Reference: [Ticket/Sign-off ID]

Total test cases executed:   [N]
PASS:                        [N] ([%])
PARTIAL:                     [N] ([%])
FAIL:                        [N] ([%])

Gaps identified:             [N]
  Missing log source:        [N]
  Misconfigured rule:        [N]
  Threshold too high:        [N]
  No rule exists:            [N]
  Data pipeline failure:     [N]

Priority gaps (Critical/High scenarios only):
  [Gap ID] — [Scenario ID] — [Root cause] — [Owner] — [Target date]
```

Share the report with the Detection Engineering lead, Program Owner, and CISO.

### Step 5 — Track and Close Gaps

Each gap identified in Step 3 becomes a tracked item:
- Open a ticket in the Detection Engineering backlog
- Tag it with the scenario ID and gap category
- Set the target date from the test case (should be ≤ 90 days from identification)
- Track closure against KPI-08 (Control Gap Closure Rate)

When a gap is remediated, re-execute only the relevant test case sub-test to confirm resolution before closing the ticket.

### Step 6 — Update KPI Dashboard

After the cycle is complete, update KPI-01 (Detection Coverage Rate) and KPI-02 (Validation Pass Rate) in the quarterly metrics report. Reference `metrics/kpis.md` for formulas.

---

## Test Environment Options

| Option | When to Use | Risks |
|--------|------------|-------|
| Dedicated test tenant | Preferred for all SaaS/cloud tests | Requires ongoing maintenance |
| Isolated lab network | Good for endpoint and network tests | May not reflect production data pipelines |
| Production with safeguards | Only when test tenant is unavailable | Requires tighter authorization; SOC must be notified; test data must be clearly labeled |

---

## Changelog

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-03-26 | Initial guide |
