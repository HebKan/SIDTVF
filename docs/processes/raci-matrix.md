# SIDTVF RACI Matrix

> **Version:** 1.0 | **Last Updated:** 2026-03-26
>
> **R** = Responsible (does the work) | **A** = Accountable (owns the outcome) | **C** = Consulted | **I** = Informed

---

## Roles

| Code | Role |
|------|------|
| PO | Program Owner / ITP Manager |
| SA | Scenario Author (SME / Threat Analyst) |
| DE | Detection Engineer |
| TH | Threat Hunter |
| SOC | SOC Analyst |
| IR | Incident Responder |
| LEG | Legal Counsel |
| HR | HR Business Partner |
| IAM | Identity & Access Management |
| CISO | CISO / Security Leadership |
| AUDIT | Audit / Compliance |
| PROC | Procurement / Vendor Management |

---

## Program Governance

| Activity | PO | SA | DE | TH | SOC | IR | LEG | HR | IAM | CISO | AUDIT |
|----------|----|----|----|----|-----|----|-----|----|-----|------|-------|
| Insider threat program policy | A/R | C | I | I | I | I | C | C | I | C | C |
| Program scope definition | A/R | C | I | I | I | I | C | C | C | C | I |
| Annual maturity assessment | A/R | C | C | C | I | C | I | I | I | I | C |
| Quarterly KPI reporting | A/R | I | I | I | I | I | I | I | I | R | I |
| Legal review of monitoring activities | A | C | I | I | I | I | R | C | I | I | C |
| Employee monitoring notice | A | I | I | I | I | I | R | R | I | I | C |

---

## Scenario Layer

| Activity | PO | SA | DE | TH | SOC | IR | LEG | HR | IAM | CISO | AUDIT |
|----------|----|----|----|----|-----|----|-----|----|-----|------|-------|
| Identify candidate scenarios | C | A/R | C | C | C | C | I | I | I | I | I |
| Author scenario card | I | A/R | C | C | I | I | C | I | I | I | I |
| Peer review scenario | C | A | R | C | I | I | I | I | I | I | I |
| Legal review of scenario | A | C | I | I | I | I | R | C | I | I | I |
| Activate scenario | A/R | R | I | I | I | I | C | I | I | I | I |
| Deprecate scenario | A/R | R | C | C | I | I | I | I | I | I | I |
| Update MITRE ATT&CK mapping | I | R | C | I | I | I | I | I | I | I | I |
| Update compliance mapping | I | R | I | I | I | I | C | I | I | I | C |

---

## Detection and Monitoring Layer

| Activity | PO | SA | DE | TH | SOC | IR | LEG | HR | IAM | CISO | AUDIT |
|----------|----|----|----|----|-----|----|-----|----|-----|------|-------|
| Detection rule development | I | C | A/R | C | I | I | I | I | I | I | I |
| Detection rule peer review | I | C | A | R | C | I | I | I | I | I | I |
| Detection rule deployment | I | I | A/R | I | C | I | I | I | I | I | I |
| False positive tuning | I | C | A/R | C | R | I | I | I | I | I | I |
| Alert triage (Level 1) | I | I | I | I | A/R | C | I | I | I | I | I |
| Alert escalation decision | I | I | I | I | A | R | C | I | I | I | I |
| DLP policy configuration | A | C | R | I | I | I | C | I | C | I | C |
| Privileged access monitoring | I | I | R | I | C | I | C | I | A | I | I |

---

## Validation Layer

| Activity | PO | SA | DE | TH | SOC | IR | LEG | HR | IAM | CISO | AUDIT |
|----------|----|----|----|----|-----|----|-----|----|-----|------|-------|
| Schedule validation cycle | A/R | I | C | I | I | I | I | I | I | I | I |
| Obtain validation authorization | A/R | I | C | I | I | I | C | I | I | C | C |
| Author test cases | I | C | A/R | C | I | I | I | I | I | I | I |
| Execute test cases | I | I | C | A/R | I | I | I | I | I | I | I |
| Document test results | I | I | C | A/R | I | I | I | I | I | I | I |
| Gap analysis and tracking | A | I | R | C | I | I | I | I | I | I | I |
| Gap remediation | I | I | A/R | I | I | I | I | I | I | I | I |

---

## Threat Hunting Layer

| Activity | PO | SA | DE | TH | SOC | IR | LEG | HR | IAM | CISO | AUDIT |
|----------|----|----|----|----|-----|----|-----|----|-----|------|-------|
| Select hunt hypotheses | C | C | C | A/R | I | I | I | I | I | I | I |
| Develop hunt queries | I | C | C | A/R | I | I | I | I | I | I | I |
| Execute threat hunts | I | I | I | A/R | C | I | I | I | I | I | I |
| Classify hunt findings | I | I | I | A/R | C | C | I | I | I | I | I |
| Escalate True Threat findings | I | I | I | R | C | A | C | I | I | I | I |
| Document hunt results | I | I | I | A/R | I | I | I | I | I | I | I |
| Feed findings back to scenarios | I | R | C | C | I | C | I | I | I | I | I |

---

## Incident Response Layer

| Activity | PO | SA | DE | TH | SOC | IR | LEG | HR | IAM | CISO | AUDIT |
|----------|----|----|----|----|-----|----|-----|----|-----|------|-------|
| Initial incident declaration | I | I | I | I | R | A | I | I | I | I | I |
| Legal notification | I | I | I | I | C | R | A | I | I | C | I |
| Evidence preservation | I | I | I | I | C | A/R | C | I | I | I | I |
| Forensic investigation | I | I | I | I | I | A/R | C | I | I | I | I |
| Account containment decision | I | I | I | I | I | C | A | C | R | C | I |
| Account disablement | I | I | I | I | I | C | C | I | A/R | I | I |
| Employment action | I | I | I | I | I | I | C | A/R | I | I | I |
| Breach notification (internal) | I | I | I | I | I | C | A/R | C | I | R | I |
| Breach notification (regulatory) | I | I | I | I | I | C | A/R | I | I | C | C |
| Post-incident review | A | R | C | C | C | R | C | C | I | I | I |
| Scenario/detection update from incident | I | A/R | R | C | I | C | I | I | I | I | I |

---

## Third-Party Risk

| Activity | PO | SA | DE | TH | SOC | IR | LEG | HR | IAM | CISO | AUDIT | PROC |
|----------|----|----|----|----|-----|----|-----|----|-----|------|-------|------|
| Vendor access provisioning | C | I | I | I | I | I | C | I | A/R | I | I | C |
| Vendor access periodic review | A | I | I | I | I | I | C | I | R | I | C | C |
| Vendor access revocation | I | I | I | I | I | C | C | I | A/R | I | I | C |
| Third-party incident response | I | I | I | I | C | A/R | R | I | C | C | I | R |
| Vendor contract review (security) | A | I | I | I | I | I | R | I | I | I | C | C |

---

## Changelog

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-03-26 | Initial RACI matrix |
