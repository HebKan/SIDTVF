# Validation Test Case Template

> Copy this file and rename it `TC-[SCENARIO-ID].md`.
> Place it in `validation/test-cases/`.
> Remove all instructional comments (lines beginning with `>`) before publishing.
>
> IMPORTANT: All validation test cases must be executed only in authorized, controlled environments
> with documented approval. Never execute test cases in production without explicit written authorization.

---

## Test Case Card

| Field | Value |
|-------|-------|
| **Test Case ID** | TC-[SCENARIO-ID] |
| **Name** | [Short descriptive name] |
| **Version** | 1.0 |
| **Status** | Draft / Active / Deprecated |
| **Created** | [YYYY-MM-DD] |
| **Last Executed** | [YYYY-MM-DD or "Never"] |
| **Author** | [Name or team] |
| **Linked Scenario** | [SCENARIO-ID] — [Scenario Name] |
| **Linked Detection** | [DET-SCENARIO-ID or "None"] |
| **Validation Type** | Detection Test / Prevention Test / Both |
| **Test Environment** | [e.g., "Dedicated test tenant", "Isolated lab network", "Production with safeguards"] |
| **Authorization Required** | [Who must approve execution — e.g., "Security Manager + Legal review for production environments"] |

---

## 1. Objective

> State the specific question this test case answers.
> A good objective is: "Does [detection rule / control] fire when [specific behavior] is performed?"

[Objective here]

---

## 2. Prerequisites

> What must be in place before this test case can be executed?

- [ ] [Prerequisite 1 — e.g., "Detection rule DET-[ID] is deployed and active in the test environment's SIEM"]
- [ ] [Prerequisite 2 — e.g., "Test user account created with appropriate permissions to simulate the scenario"]
- [ ] [Prerequisite 3 — e.g., "Written authorization from [role] obtained and on file"]
- [ ] [Prerequisite 4 — e.g., "Proxy logs are flowing to SIEM and the detection query has been confirmed to run without errors"]

---

## 3. Test Environment

| Field | Value |
|-------|-------|
| **Environment Type** | Test / Production-With-Safeguards |
| **Test User Account** | [e.g., "testuser-insider-01@testdomain.com"] |
| **Source System** | [e.g., "Managed test laptop, Windows 11, enrolled in MDM"] |
| **Target System / Data** | [e.g., "Non-sensitive synthetic test files in the test repository"] |
| **Detection System** | [e.g., "Microsoft Sentinel with DE-001 detection rule active"] |
| **Monitoring Points** | [e.g., "Proxy logs, EDR telemetry, DLP alerts"] |

---

## 4. Test Steps

> Provide numbered, discrete steps for executing the test. Steps should be specific enough for any analyst to reproduce them.

| Step | Action | Expected System Response |
|------|--------|------------------------|
| 1 | [Precise action to take] | [What should happen in logs or detection systems] |
| 2 | [Next action] | [Expected response] |
| N | [Final action] | [Final expected response] |

---

## 5. Expected Results

> Describe the specific outcome that constitutes a "PASS" — the detection fired correctly.

**PASS Criteria:**
- [ ] [Specific alert / log entry that must appear — be as precise as possible about field values]
- [ ] [Additional condition that must be met]

**Detection alert must contain:**
- Alert name: [Expected alert name]
- User field: [Expected user account]
- Data source: [Expected source]
- Triggered within: [Expected time from action to alert — e.g., "Within 5 minutes"]

---

## 6. Test Results Log

> Document each execution of this test case. One entry per execution.

### Execution: [YYYY-MM-DD]

| Field | Value |
|-------|-------|
| **Executed By** | [Name] |
| **Date** | [YYYY-MM-DD] |
| **Environment** | [Test / Production-With-Safeguards] |
| **Result** | PASS / FAIL / PARTIAL |
| **Alert Fired?** | Yes / No / Partial |
| **Time to Alert** | [Minutes from test action to alert appearance] |
| **Alert Fields Correct?** | Yes / No / Partial — [describe discrepancy] |
| **False Positives Generated** | [Count and description] |
| **Notes** | [Any observations, anomalies, or tuning recommendations] |

**Gap Analysis (if FAIL or PARTIAL):**
- Root cause: [Why did the detection not fire?]
- Gap category: Missing log source / Misconfigured rule / Threshold too high / No rule exists / Other
- Remediation: [What needs to be fixed]
- Remediation owner: [Team]
- Target remediation date: [Date]

---

## 7. Cleanup Steps

> After the test is complete, return the environment to its pre-test state.

| Step | Action |
|------|--------|
| 1 | [Cleanup action — e.g., "Delete all test files uploaded during the test"] |
| 2 | [Next cleanup action] |
| 3 | Document test completion and notify [team] that the test window is closed |

---

## 8. Linked Artifacts

| Artifact | ID | File |
|----------|----|------|
| Scenario | [SCENARIO-ID] | [scenarios/...] |
| Detection | DET-[SCENARIO-ID] | [detections/...] |
| Playbook | PB-[SCENARIO-ID] | [playbooks/...] |
| Hunt Hypothesis | HYP-[SCENARIO-ID] | [hunting/hypotheses/...] |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | [YYYY-MM-DD] | [Author] | Initial creation |
