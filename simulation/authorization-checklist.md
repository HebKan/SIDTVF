# Simulation Authorization Checklist

> **IMPORTANT:** Complete this checklist and obtain all required approvals BEFORE executing any SIDTVF simulation test. Unauthorized simulation activity may constitute computer fraud under applicable law, trigger security alerts that consume real SOC resources, and create legal liability for the organization and the individual executing the test.

---

## Required Before Every Simulation Test

### 1. Scope Authorization

- [ ] The simulation target environment is a **test or staging environment** OR explicit written authorization has been obtained from the system owner for a production-adjacent run
- [ ] The specific scenario to be simulated is identified: `SIM-_______`
- [ ] The specific systems and accounts to be involved are listed in a change request or test plan
- [ ] The simulation has a defined **start time**, **end time**, and **point of contact**

**Authorization obtained from (system owner):** ___________________________

**Date authorized:** ___________________________

### 2. Legal and Compliance Review

- [ ] Legal counsel has confirmed that simulation activity in this environment is permissible under applicable employment law and monitoring agreements
- [ ] Employees whose accounts will be used as test accounts (if any) have been informed that their account may be used in simulation activity — OR dedicated test accounts have been created that do not correspond to real employees
- [ ] The simulation will not involve real PII, PHI, financial records, or intellectual property

### 3. Change Management

- [ ] A change request or test notification has been submitted to the relevant change management system
- [ ] The SOC or security operations team is aware that a simulation is scheduled — to prevent wasted alert investigation time
- [ ] A "simulation in progress" marker has been agreed upon (e.g., a specific tag in alert notes) so that SOC analysts know that an active alert may be simulation-generated

**Change ticket / notification reference:** ___________________________

### 4. Technical Prerequisites

- [ ] A dedicated **test user account** has been created (e.g., `sim.testuser@yourdomain.com`)
- [ ] The test account has been provisioned with access equivalent to the actor in the scenario (e.g., DBA access for SIM-SB-001)
- [ ] A synthetic test dataset has been created — no real sensitive data will be used
- [ ] All logging is confirmed active (proxy logs, endpoint agent, SIEM ingestion, database audit logging — whichever is required for the specific test)
- [ ] A cleanup plan is prepared and a cleanup assignee is designated

**Cleanup assignee:** ___________________________

### 5. Post-Test Obligations

- [ ] The test execution log will be completed and retained for at least 90 days
- [ ] Any alerts generated will be marked as simulation-generated in the ticketing system within 24 hours of test completion
- [ ] Cleanup will be completed within **4 hours** of test completion
- [ ] Test results will be shared with the detection engineering team within 5 business days

---

## Approvals

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Test Lead | | | |
| System Owner | | | |
| SOC Lead (notification) | | | |
| Legal (if production environment) | | | |

---

## Environment Classification

| Environment Type | Approvals Required | Real Data Permitted |
|-----------------|-------------------|---------------------|
| Dedicated test environment (no prod data) | Test Lead + System Owner | No — synthetic only |
| Staging environment (prod-like config, no prod data) | Test Lead + System Owner + SOC notification | No — synthetic only |
| Production environment (exceptional) | All four approvers above + written Legal sign-off | Absolutely not |

> **Note:** Most simulation tests can and should be run in a dedicated test environment. Production simulation is rarely necessary and significantly increases risk and compliance complexity.
