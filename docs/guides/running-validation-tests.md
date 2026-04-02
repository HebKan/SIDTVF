# Guide: Running Validation Tests

Validation tests answer a specific question: **does your existing detection actually fire for this scenario?** This guide covers how to execute a test case end-to-end.

---

## Validation vs. Simulation

| | Validation Test (TC-[ID]) | Simulation Test (SIM-[ID]) |
|-|--------------------------|---------------------------|
| **Purpose** | Confirm a detection rule fires as configured | Validate detection via realistic behavioral patterns |
| **Environment** | Can run in isolated test environment | Requires near-production environment for realistic logs |
| **Data** | Synthetic; minimal setup | Synthetic but volume/timing realistic |
| **Who runs it** | Detection engineer or security engineer | Red team or security engineer with IR Lead authorization |

Start with validation tests. Run simulation tests only after validation confirms your detection coverage baseline.

---

## Prerequisites

Before running any validation test:

- [ ] Written authorization from the environment owner
- [ ] Test environment confirmed isolated from production (or explicit approval for production testing)
- [ ] Legal review completed if the test involves monitoring of communication channels
- [ ] Change management ticket created if required by your organization
- [ ] Rollback plan confirmed for any configuration changes

---

## Running a Test Case: Step-by-Step

### Step 1 — Read the Test Case File
Open `validation/test-cases/TC-[SCENARIO_ID].md`. Review:
- **Test objective** — what exact detection behavior is being validated
- **Required log sources** — if these are missing, stop here and fix the logging gap first
- **Prerequisites** — accounts, tools, and environment configuration required
- **Pass/fail criteria** — the exact expected output (rule name, alert text, timeframe)

### Step 2 — Set Up the Test Environment
Follow the setup instructions in the test case. This typically includes:
- Creating a test user account with appropriate permissions
- Confirming required log sources are ingesting into your SIEM
- Noting the current timestamp (you'll need it to scope your log query after the test)

### Step 3 — Execute the Test Steps
Follow each numbered step in the test case exactly. Do not improvise. If a step is unclear, stop and clarify rather than approximating — the pass/fail criteria are calibrated to the documented steps.

Record the exact time of each action.

### Step 4 — Check for Alert Generation
After completing the test steps, wait for the observation window noted in the test case (typically 5–30 minutes for real-time rules; up to 90 minutes for rules with ingestion delay).

Then query your SIEM for the expected alert:
- Did the rule fire? Record the exact alert timestamp and content.
- Did the rule fire within the expected observation window?
- Did the alert contain the expected fields (user, destination, data volume)?

### Step 5 — Record Results

| Outcome | Meaning | Next step |
|---------|---------|-----------|
| **Pass** — alert fired within window with correct fields | Detection is working | Update test case with pass date; proceed to next scenario |
| **Partial** — alert fired but outside window or missing fields | Detection has a gap | Document the gap; file a detection improvement task |
| **Fail** — no alert generated | Detection does not cover this scenario | Escalate to detection engineering; do not mark scenario as covered |
| **False positive** — alert fired for a different reason | Rule needs tuning | Document the benign trigger; tune and re-test |

---

## What to Do When a Test Fails

A failed validation test is useful information — it means your detection coverage assumption was wrong.

1. **Check the log source** — confirm the expected logs were actually generated during the test. If not, the issue is upstream of the detection rule.
2. **Check the rule logic** — verify the detection rule's field names match your actual log schema. Schema mismatches are the most common failure cause.
3. **Check the threshold** — volume-based rules may not fire for single-event tests. Adjust the test to generate sufficient volume per the rule's threshold.
4. **File a detection gap** — document the failure in your coverage matrix (`docs/coverage-matrix.md`) and assign a remediation task to detection engineering.

---

## Cleanup

After every test:
- Delete the test user account or reset it to its pre-test state
- Remove any synthetic files or email rules created during the test
- Confirm log entries from the test are identifiable as test activity (add a note to your SIEM case or ticket)
- Close the change management ticket

---

## Related

- Test case template: [validation/_template.md](../../validation/_template.md)
- All test cases: [validation/test-cases/](../../validation/test-cases/)
- After validation — running a simulation: [docs/guides/running-simulation-tests.md](running-simulation-tests.md)
- Validation cycle methodology: [docs/processes/validation-cycle-guide.md](../processes/validation-cycle-guide.md)
