# Response Playbook: PB-SB-001

| Field | Value |
|-------|-------|
| **Playbook ID** | PB-SB-001 |
| **Linked Scenario** | SB-001 — Targeted Database Deletion |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-26 |
| **Last Updated** | 2026-03-26 |
| **Author** | SIDTVF Core Team |
| **Severity** | Critical |
| **Classification** | RESTRICTED — Internal Use Only |

---

## Overview

This playbook governs the organizational response to a confirmed or suspected insider sabotage event involving intentional deletion or destruction of databases, backup systems, or critical infrastructure. The SB-001 scenario typically involves a privileged user (DBA, infrastructure engineer, or sysadmin) with elevated access who acts to cause maximum disruption, often around the time of a negative employment event (termination, demotion, PIP).

**Time is the highest-priority variable in this playbook.** Data destruction is partially or fully recoverable only if recovery procedures are initiated quickly and the actor has not also destroyed backups. Parallelizing notification, evidence preservation, and recovery is required.

---

## Prerequisites

Before executing this playbook, confirm that:

- [ ] An alert or report has been received indicating database objects have been deleted, dropped, or rendered inaccessible
- [ ] At least one of the following is available: SIEM alert, database audit log, storage snapshot, or on-call notification from operations
- [ ] The IR lead is notified and the playbook owner is assigned
- [ ] A bridge line or incident channel is established for cross-team coordination
- [ ] Legal has been notified that a potential insider sabotage event is in progress (do not wait for confirmation)

---

## Severity Classification

| Level | Criteria | SLA — Triage | SLA — Containment | SLA — Legal Notify |
|-------|----------|-------------|-------------------|--------------------|
| **Critical** | Production databases deleted/encrypted; backup integrity unknown; business impact active | 15 min | 1 hour | Immediate |
| **High** | Non-production databases deleted; backups intact; no confirmed business impact yet | 30 min | 2 hours | 2 hours |
| **Medium** | Deletion attempt blocked or reversed; limited scope confirmed | 1 hour | 4 hours | 4 hours |

> **Default severity for SB-001 is Critical until proven otherwise.** Treat every database deletion alert at Critical severity until the scope and backup integrity are confirmed.

---

## Phase 1 — Triage and Initial Assessment

**Objective:** Confirm whether a destructive event is occurring or has occurred. Establish scope and initial timeline.

| Step | Action | Owner | Time Target |
|------|--------|-------|-------------|
| 1.1 | Identify the alert source: which database(s), which host(s), what operation(s) | SOC Tier 1 | T+0:05 |
| 1.2 | Check database availability — can the application connect? Are services up? | SOC / Ops | T+0:10 |
| 1.3 | Identify the account that performed the destructive operation from database audit log | SOC Tier 2 | T+0:15 |
| 1.4 | Determine the account's current employment status — active, terminated, on notice? | SOC + HR | T+0:15 |
| 1.5 | Check whether backup deletion or alteration events appear in the log alongside the database deletion | SOC Tier 2 | T+0:20 |
| 1.6 | Escalate to Critical if: production databases affected, backup integrity unknown, or account is recently terminated | IR Lead | T+0:20 |
| 1.7 | Open incident ticket; assign IR lead and database recovery owner | IR Lead | T+0:20 |

**DECISION GATE — Triage:**
> After Step 1.6, determine severity level. If **Critical or High**, immediately proceed to Phase 2 (Notification) in **parallel** with Phase 3 (Evidence Preservation) and Phase 4 (Business Continuity). Do not wait for full scope before notifying.

---

## Phase 2 — Immediate Notification

**Objective:** Ensure all required stakeholders are engaged within the first 30 minutes.

| Step | Action | Owner | Time Target |
|------|--------|-------|-------------|
| 2.1 | Notify CISO and CTO/CIO — this is a potential insider sabotage event | IR Lead | T+0:25 |
| 2.2 | Notify Legal — they must be on the incident from the start for privilege protection and evidence handling | IR Lead | T+0:25 |
| 2.3 | Notify HR — employment status of the suspected account holder may need to be confirmed or actioned | IR Lead + Legal | T+0:30 |
| 2.4 | Notify database recovery team / DBA lead — initiate recovery assessment immediately | IR Lead + Ops | T+0:25 |
| 2.5 | If the organization has cyber insurance, notify the insurance carrier per policy terms — many policies require notification within 24–72 hours | Legal + Finance | T+2:00 |
| 2.6 | Do NOT notify the suspected account holder at this stage — they may still have active access or co-conspirators | IR Lead | — |

---

## Phase 3 — Evidence Preservation

**Objective:** Preserve all forensic evidence before any remediation or recovery actions that might overwrite logs.

> **Critical:** Evidence preservation must begin immediately and run in parallel with recovery. Recovery procedures can destroy forensic evidence. Coordinate carefully.

| Step | Action | Owner | Time Target |
|------|--------|-------|-------------|
| 3.1 | Export database audit logs from the affected database platform (SQL Server, Oracle, PostgreSQL, MySQL) covering at least the prior 72 hours | IR | T+0:30 |
| 3.2 | Snapshot or export OS-level logs for the database server: authentication events, process execution logs, cron/scheduled tasks | IR | T+0:45 |
| 3.3 | Export SIEM events for the actor's account across all data sources for the prior 30 days | IR | T+0:45 |
| 3.4 | Capture a forensic image (or logical copy) of the actor's workstation if possible — do not perform without Legal guidance | IR + Legal | T+2:00 |
| 3.5 | Preserve all backup system logs — whether backups were accessed, modified, or deleted | IR + DBA | T+0:30 |
| 3.6 | Screenshot or export current database server state: running processes, open connections, file system | IR | T+0:20 |
| 3.7 | Hash all collected log files (SHA-256) and store in an evidence locker accessible only to IR and Legal | IR | T+1:00 |
| 3.8 | Do not delete, reimage, or recycle any affected system without Legal sign-off | IR + Legal | Ongoing |

---

## Phase 4 — Containment

**Objective:** Stop any ongoing destructive activity and prevent the actor from causing additional damage.

| Step | Action | Owner | Time Target |
|------|--------|-------|-------------|
| 4.1 | Revoke all active sessions for the suspected account — AD, database platform, VPN, cloud console | IAM | T+0:30 |
| 4.2 | Disable the account in Active Directory / Entra ID | IAM | T+0:30 |
| 4.3 | Revoke SSH keys, database credentials, and API tokens associated with the account | IAM + DevOps | T+0:45 |
| 4.4 | Revoke VPN certificates and remote access capabilities | IAM + Network | T+0:45 |
| 4.5 | Block the account from all cloud consoles (AWS IAM, Azure AD, GCP IAM) | IAM + Cloud | T+0:45 |
| 4.6 | Identify any service accounts the actor may have created or modified in the prior 30 days — disable pending review | IAM + DBA | T+2:00 |
| 4.7 | Review for scheduled tasks or cron jobs the actor may have planted to trigger delayed destruction | IR + DBA + DevOps | T+1:00 |

**DECISION GATE — Containment:**
> Before proceeding to recovery, confirm: (1) account access fully revoked, (2) no active sessions remain, (3) no scheduled tasks or logic bombs identified. If scheduled tasks are found, treat as a separate IR event.

---

## Phase 5 — Business Continuity and Recovery

**Objective:** Restore critical systems to service as quickly as possible while preserving evidence.

| Step | Action | Owner | Time Target |
|------|--------|-------|-------------|
| 5.1 | Assess backup integrity: when was the last clean backup? Is it accessible? Has it been modified? | DBA + Ops | T+0:30 |
| 5.2 | Invoke the database disaster recovery (DR) runbook — restore from last known good backup | DBA + Ops | T+1:00 |
| 5.3 | Determine Recovery Point Objective (RPO) gap: what data was lost between the last backup and the deletion event? | DBA + Business Owner | T+2:00 |
| 5.4 | Notify application owners and business units affected by the outage | IR Lead + Business Owner | T+0:30 |
| 5.5 | If no clean backup exists: engage database forensics specialists — deleted rows may be partially recoverable from transaction logs or storage blocks | IR + External Forensics | T+2:00 |
| 5.6 | Validate restored databases before re-enabling application access — check row counts against pre-incident baselines | DBA + Application Owner | T+4:00 |
| 5.7 | Enable enhanced monitoring on all database platforms while investigation is ongoing | IR + DBA | T+2:00 |

---

## Phase 6 — Investigation

**Objective:** Reconstruct the full timeline of the actor's activity, scope of damage, and method of access.

| Step | Action | Owner | Time Target |
|------|--------|-------|-------------|
| 6.1 | Reconstruct a timeline of the actor's activity for the 30 days prior to the event — what was accessed, modified, deleted? | IR | T+4:00 |
| 6.2 | Determine the entry point: was this an authorized login from a corporate device, or did they use a backdoor/VPN/remote access tool? | IR | T+4:00 |
| 6.3 | Identify whether any data was exfiltrated before the deletion occurred — destruction is sometimes preceded by theft | IR | T+4:00 |
| 6.4 | Determine whether the actor acted alone or in coordination — review communication logs (email, Slack, Teams) per Legal guidance | IR + Legal | T+8:00 |
| 6.5 | Document the full scope: databases deleted, tables dropped, backup systems touched, services stopped | IR + DBA | T+8:00 |
| 6.6 | Identify what drove the action: employment event (termination, PIP, demotion) that should have triggered enhanced monitoring | IR + HR | T+8:00 |
| 6.7 | Determine whether this was a logic bomb (delayed execution) — review scheduled tasks, triggers, and stored procedures for malicious code | IR + DBA | T+4:00 |

---

## Phase 7 — Legal and HR Escalation

**Objective:** Ensure appropriate legal, employment, and law enforcement actions are taken based on confirmed findings.

| Step | Action | Owner | Time Target |
|------|--------|-------|-------------|
| 7.1 | Provide Legal with the complete evidence package: timeline, logs, hashed artifacts | IR | T+24:00 |
| 7.2 | Legal assesses applicable criminal exposure: Computer Fraud and Abuse Act (CFAA), state computer crime statutes, and economic sabotage statutes | Legal | T+24:00 |
| 7.3 | Legal advises on law enforcement referral — FBI Cyber Division for significant CFAA violations; local law enforcement for jurisdictional matters | Legal | T+48:00 |
| 7.4 | HR initiates termination procedures if the actor is still employed, coordinating timing with Legal to avoid destroying evidence | HR + Legal | T+24:00 |
| 7.5 | If the actor is already terminated, Legal advises on civil remedies — lawsuit for damages is often available under CFAA and state computer crime law | Legal | T+48:00 |
| 7.6 | Assess breach notification obligations: did any regulated data (PHI, PII, financial records) become inaccessible or exposed during the event? | Legal + Privacy | T+48:00 |

---

## Phase 8 — Post-Incident Review and Control Improvement

**Objective:** Identify control gaps and take corrective actions to prevent recurrence.

| Step | Action | Owner | Time Target |
|------|--------|-------|-------------|
| 8.1 | Conduct a blameless post-incident review — focus on process and control failures, not individual fault | IR Lead + Engineering | T+7d |
| 8.2 | Identify the HR trigger event that preceded the attack — was enhanced monitoring warranted and was it in place? | IR + HR | T+7d |
| 8.3 | Review database access provisioning: did the actor have access that exceeded their job requirements? | IAM + DBA | T+7d |
| 8.4 | Assess backup security: could an insider with DBA access delete or encrypt backups? Deploy immutable backup solutions if not in place | DBA + Ops | T+30d |
| 8.5 | Deploy or tune detection rules for the IOBs in scenario SB-001 | Detection Engineering | T+30d |
| 8.6 | Update the SB-001 scenario card if new behavior patterns were observed | SA | T+14d |
| 8.7 | Publish incident lessons learned (sanitized) to the security team — build organizational knowledge | IR Lead | T+14d |

---

## False Positive Considerations

Database deletion events may arise from non-malicious causes. Before escalating to a full insider sabotage response, confirm:

| False Positive Scenario | Distinguishing Factor | Resolution |
|------------------------|----------------------|-----------|
| Scheduled database maintenance (purge old tables) | Maintenance window is documented; changes match a known maintenance script | Verify against change management records |
| Automated test environment teardown | Target database is tagged as non-production; deletion is scripted | Verify with DevOps — confirm target was a test environment |
| DBA error (dropped wrong table) | User immediately reports the error; no cover-up behavior observed | Treat as high-priority incident for recovery; no criminal referral |
| Decommissioned system cleanup | Deletion follows a formal decommissioning ticket | Verify against ITSM/CMDB records |
| Backup rotation policy execution | Backup deletions match documented retention policy | Verify against backup management system logs |

**Key distinguishing indicators for genuine insider sabotage:**
- Backup systems targeted at the same time as live databases
- Access to systems not within the actor's normal job scope
- Activity occurs within a few days of a known employment event (termination, PIP)
- Scheduled tasks or logic bombs are found — accidental deletion does not leave behind persistence mechanisms

---

## Key Contacts

| Role | Contact | When to Engage |
|------|---------|----------------|
| IR Lead | [Your IR Lead] | Immediately at triage |
| CISO | [Your CISO] | Phase 2 — within 25 minutes |
| Legal Counsel | [Your Legal Team] | Phase 2 — within 25 minutes |
| DBA Lead | [Your DBA Lead] | Phase 2 — within 25 minutes |
| HR Director | [Your HR Director] | Phase 2 — within 30 minutes |
| Cyber Insurance Carrier | [Your Carrier] | Phase 2 — within 2 hours |
| FBI Cyber Division (US) | IC3.gov | Phase 7 — if criminal referral warranted |

---

## Linked Artifacts

| Type | ID | File |
|------|----|------|
| Scenario | SB-001 | scenarios/sabotage/SB-001-targeted-database-deletion.md |
| Validation Test Case | TC-SB-001 | validation/test-cases/TC-SB-001.md (create) |
| Detection | DET-SB-001 | detections/ (create) |
| Hunt Hypothesis | HYP-SB-001 | hunting/hypotheses/HYP-SB-001.md (create) |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-26 | SIDTVF Core Team | Initial creation |
