# Response Playbook: PB-DE-001 — Cloud Storage Exfiltration

## Playbook Card

| Field | Value |
|-------|-------|
| **Playbook ID** | PB-DE-001 |
| **Name** | Cloud Storage Upload Exfiltration Response |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-25 |
| **Last Updated** | 2026-03-25 |
| **Author** | SIDTVF Core Team |
| **Linked Scenario** | DE-001 — Cloud Storage Upload Exfiltration |
| **Trigger** | Alert DET-DE-001-SIGMA or DET-DE-001-KQL fires, or analyst identifies suspicious cloud upload behavior during threat hunt or routine review |
| **Primary Owner** | Security Operations Center (SOC) / Incident Response (IR) |
| **Stakeholders** | SOC, IR, Legal, HR, Data Privacy, Executive (for Critical) |

---

## 1. Overview

This playbook guides the response to a confirmed or suspected insider data exfiltration event via personal cloud storage services (Google Drive, Dropbox, Box, OneDrive Personal, MEGA, WeTransfer, etc.). The goal is to: stop ongoing exfiltration if present; preserve all relevant evidence in forensically sound condition; determine the full scope of data exposed; meet legal and compliance obligations; and take appropriate action against the individual.

A successful response results in a complete scope determination (what was taken), a documented chain of custody for all evidence, appropriate notification to affected parties (if required), and remediation of the access or configuration gap that enabled the scenario.

---

## 2. Prerequisites

- [ ] Access to web proxy or NGFW logs in SIEM covering at least 90 days
- [ ] Access to endpoint telemetry (EDR) for the affected device
- [ ] Access to DLP logs and alert queue
- [ ] Legal counsel on-call or reachable within 2 hours
- [ ] HR business partner on-call or reachable within 4 hours
- [ ] IAM team able to execute account action (disable, restrict) within 1 hour on request
- [ ] Data Privacy/Compliance officer identified for breach notification decisions
- [ ] Forensics capability available for image acquisition if needed

---

## 3. Severity Classification

| Severity | Criteria | Response SLA |
|----------|----------|-------------|
| **Critical** | Data volume > 500 MB OR confirmed sensitive data (PII, PHI, PCI, trade secrets, source code) OR departing employee OR law enforcement referral likely | 30 min to IR engagement; Legal within 1 hour |
| **High** | Data volume 50–500 MB OR sensitive data category suspected but unconfirmed | 2 hours to IR engagement; Legal within 4 hours |
| **Medium** | Data volume < 50 MB OR internal/unclassified data only | 4 hours; Legal at investigator's discretion |
| **Low** | Single small upload to approved service, probable policy misunderstanding | Next business day; HR notification recommended |

---

## 4. Response Steps

### Phase 1 — Triage and Verification

**Goal:** Confirm whether the alert represents true insider activity or a false positive, and classify initial severity.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 1.1 | Review the triggering alert: identify the user account, source IP, target cloud domain, and total bytes uploaded | SOC | Document findings |
| 1.2 | Pull proxy/endpoint logs for the past 30 days for this user to establish the upload baseline | SOC | Compare against the user's 90-day historical average |
| 1.3 | Identify the data classification of the uploaded content if possible (check DLP alert details, file names, or directory paths) | SOC | Sensitive data escalates severity |
| 1.4 | Check HR system for the user's employment status and any flagged events (departure notice, PIP, disciplinary action) | SOC / IR | Departure flag escalates to Critical minimum |
| 1.5 | Check whether the destination cloud domain is an approved business service for this user | SOC | If approved and documented, may be false positive — verify with manager |

**[DECISION GATE — TRIAGE]**
- If activity is verified as legitimate and authorized → close as false positive, document justification, proceed to false positive process
- If activity cannot be confirmed legitimate within 30 minutes → treat as true positive and proceed to Phase 2

---

### Phase 2 — Initial Notification

**Goal:** Notify the required stakeholders immediately and get Legal engaged before taking action.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 2.1 | Notify the IR lead and open a formal incident ticket | SOC | Use secure communication channel — not the affected user's usual channels |
| 2.2 | Notify Legal within the SLA for the classified severity | SOC / IR Lead | Provide initial findings: user, data volume, data classification estimate, employment status |
| 2.3 | [Critical only] Notify CISO and Data Privacy officer | IR Lead | Required if PII, PHI, PCI, or trade secret exfiltration is suspected |
| 2.4 | Do NOT notify or confront the subject employee at this stage without Legal instruction | All | Premature disclosure may compromise the investigation |

---

### Phase 3 — Evidence Preservation

**Goal:** Preserve all relevant logs and data in a forensically sound state before any account actions that might destroy evidence.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 3.1 | Export and preserve proxy logs for the affected user for the past 90 days | IR | Save to immutable storage. Do not modify. |
| 3.2 | Export and preserve endpoint telemetry (EDR) for the affected device for the past 90 days | IR | Includes file access, process, and network events |
| 3.3 | Export and preserve DLP logs for the affected user and any triggered alerts | IR | |
| 3.4 | Preserve authentication logs (AD, Entra ID, VPN) for the affected account for the past 90 days | IR | |
| 3.5 | Request a forensic image of the affected endpoint if the upload volume is > 100 MB or if criminal referral is possible | IR / Forensics | Coordinate with Legal before imaging — chain of custody procedures apply |
| 3.6 | Document chain of custody for all preserved evidence | IR | Date, time, who collected, storage location, hash values |
| 3.7 | Place the affected account under a legal hold — notify IT and Legal | Legal | Prevents log rotation or data deletion for the account |

---

### Phase 4 — Containment

**Goal:** Stop ongoing exfiltration while minimizing destruction of evidence and avoiding premature detection by the subject.

**[DECISION GATE — CONTAINMENT]**
Legal must advise on account action timing. If the investigation may lead to criminal prosecution, a covert investigation window may be needed before overt action. Do NOT disable the account without Legal approval.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 4.1 | Block access to personal cloud storage domains at the web proxy or NGFW for the affected user (covert action — does not alert the user) | IR / IT | This stops ongoing exfiltration without alerting the subject |
| 4.2 | Enable enhanced DLP monitoring for all egress channels for the affected account | DLP / IR | Email, USB, printing, web upload |
| 4.3 | If Legal approves overt action: disable or restrict the account; revoke active sessions | IAM / IT | Notify subject per HR procedures |
| 4.4 | If the affected device is a corporate asset: isolate from network (if Criminal referral likely — coordinate with Legal) | IR | Forensic imaging should precede or accompany isolation |

---

### Phase 5 — Investigation

**Goal:** Determine the full scope of the incident — what was uploaded, to where, when, and by whom. Assess what data was exposed and to whom.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 5.1 | Identify all cloud storage upload events for this user in the past 12 months | IR | Look for a pattern, not just the triggering event |
| 5.2 | Enumerate all files accessed by the user in the 30 days preceding the first upload event | IR | File audit logs, EDR file events |
| 5.3 | Identify data classification of accessed files using DLP, file tagging, or manual review | IR + Data Owner | This determines breach notification obligations |
| 5.4 | Correlate HR events: departure date, performance events, access reviews, disciplinary history | Legal + HR | Context for motive and timeline |
| 5.5 | Determine whether the upload destination is a personal account or a shared/external account | IR / Legal | Shared accounts may indicate coordination with external parties |
| 5.6 | Check for other exfiltration channels used concurrently (email forwarding, USB writes, print jobs) | IR | See related scenarios DE-002 |
| 5.7 | Document findings in the incident report with a complete timeline | IR Lead | |

---

### Phase 6 — Escalation and Notification

**Goal:** Notify appropriate parties based on confirmed scope and legal obligations.

**[DECISION GATE — NOTIFICATION]**
Legal determines all external notification decisions. Do not contact regulators, customers, or media without Legal approval.

| Stakeholder | Notification Trigger | Owner | Channel |
|-------------|---------------------|-------|---------|
| HR Business Partner | Confirmed insider activity | Legal + IR Lead | Secure call |
| Data Privacy Officer | PII or PHI suspected in exfiltrated data | Legal | Secure email |
| CISO | Critical severity | IR Lead | Phone |
| Executive Leadership | Critical severity with high business impact | CISO + Legal | Secure briefing |
| Cyber Insurance Carrier | Confirmed breach (check policy for notification window) | Legal + Finance | Per policy requirements |
| Regulatory Body (HIPAA/OCR, GDPR DPA, SEC, etc.) | Confirmed breach of regulated data | Legal | Per regulatory requirements |
| Affected Individuals | Confirmed PII/PHI exposure | Legal + Privacy | Per breach notification law |
| Law Enforcement (FBI/Local) | Criminal referral warranted | Legal | Coordinate with General Counsel |

---

### Phase 7 — Employment Action

**Goal:** Take appropriate HR and legal action against the insider, coordinated with the investigation.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 7.1 | Legal advises HR on the appropriate course of action based on investigation findings | Legal | May include suspension, termination, civil action, or criminal referral |
| 7.2 | HR conducts employment action following their standard procedures, with Legal present | HR + Legal | |
| 7.3 | Revoke all access, credentials, and physical badges at the time of employment action | IAM / Physical Security | |
| 7.4 | Notify all relevant systems and teams of the access revocation (IT, vendors, building access) | IT / IAM | |
| 7.5 | Preserve all evidence and documentation for litigation or prosecution | Legal + IR | |

---

### Phase 8 — Recovery and Remediation

**Goal:** Close the access pathway that enabled the scenario and prevent recurrence.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 8.1 | Block all personal cloud storage domains at the network layer (if not already done as policy) | IT Security | Implement organization-wide, not just for the affected user |
| 8.2 | Enable and tune DLP rules for cloud upload detection (if not already active) | Detection Engineering | Reference DET-DE-001 |
| 8.3 | Implement or update the departing employee monitoring workflow | IAM / HR | Enhanced monitoring should begin at departure notification |
| 8.4 | Review and close any stale or over-provisioned access for other high-risk users | IAM | Quarterly access review |
| 8.5 | Update the acceptable use policy if the existing policy had ambiguity that may have enabled this scenario | Legal + HR | |

---

### Phase 9 — Post-Incident Review

**Goal:** Document, learn, and improve the program.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 9.1 | Complete the incident report within 5 business days of closure | IR Lead | |
| 9.2 | Conduct a post-incident review meeting with all stakeholder teams | IR Lead | Include: what happened, what worked, what failed, what to improve |
| 9.3 | Update DE-001 scenario card if new attack behaviors were observed | Scenario Author | Add as new IOBs or attack phases |
| 9.4 | Update detection rules if the exfiltration technique evaded current rules | Detection Engineering | |
| 9.5 | Update this playbook if steps were missing, unclear, or out of order | IR Lead | Increment version number |
| 9.6 | Report incident metrics to security leadership | IR Lead + CISO | Update KPI dashboard |

---

## 5. False Positive Considerations

| Scenario | How to Confirm False Positive |
|----------|------------------------------|
| Employee using personal Google Drive for an approved work project | Verify with manager and check for a documented business justification or exemption |
| Automated backup to cloud using a service account | Confirm the process name (EDR) is a known backup agent and the destination is a registered backup account |
| File upload was to a sanctioned business cloud instance, not personal | Confirm the target URL/domain corresponds to the organization's managed cloud tenant, not a personal account |
| Employee uploaded personal files (photos, personal docs) — policy violation but not IP theft | Classify as PV (Policy Violation), route to HR; do not escalate to Legal unless policy is clear and repeated |

---

## 6. Key Contacts

> Replace with actual contacts during deployment.

| Role | Name | Contact |
|------|------|---------|
| IR Lead | [TBD] | [Phone / Secure Channel] |
| Legal Counsel (On-Call) | [TBD] | [Phone] |
| HR Business Partner | [TBD] | [Phone / Email] |
| Data Privacy Officer | [TBD] | [Phone / Email] |
| CISO | [TBD] | [Phone] |
| IAM On-Call | [TBD] | [Phone / Ticket] |
| Cyber Insurance (Carrier + Policy #) | [TBD] | [Phone / Claims Portal] |

---

## 7. Linked Artifacts

| Artifact | ID | File |
|----------|----|------|
| Scenario | DE-001 | [scenarios/data-exfiltration/DE-001-cloud-storage-upload.md](../../scenarios/data-exfiltration/DE-001-cloud-storage-upload.md) |
| Validation Test Case | TC-DE-001 | [validation/test-cases/TC-DE-001.md](../../validation/test-cases/TC-DE-001.md) |
| Detection (Sigma) | DET-DE-001-SIGMA | [detections/sigma/DE-001-cloud-storage-upload.yml](../../detections/sigma/DE-001-cloud-storage-upload.yml) |
| Detection (KQL) | DET-DE-001-KQL | [detections/queries/DE-001-kql.md](../../detections/queries/DE-001-kql.md) |
| Hunt Hypothesis | HYP-DE-001 | [hunting/hypotheses/HYP-DE-001.md](../../hunting/hypotheses/HYP-DE-001.md) |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-25 | SIDTVF Core Team | Initial creation |
