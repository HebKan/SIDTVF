# Scenario Card: SB-001 — Targeted Database Deletion

| Field | Value |
|-------|-------|
| **Scenario ID** | SB-001 |
| **Name** | Targeted Database Deletion |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-25 |
| **Last Updated** | 2026-03-25 |
| **Author** | SIDTVF Core Team |
| **Category** | Sabotage & Disruption |
| **Category Code** | SB |
| **Actor Archetype(s)** | MAL |
| **Severity** | Critical |
| **Minimum Maturity Level** | 3 |

---

## 1. Description

A disgruntled or departing insider with database or system administrator access deliberately deletes, corrupts, or destroys production data with the intent to cause operational disruption, financial harm, or reputational damage to the organization. The actor typically targets systems they know are business-critical and may disable or manipulate backup mechanisms to maximize recovery time. This scenario represents one of the highest-impact insider threat events and is strongly associated with imminent or recent separation from the organization.

---

## 2. Narrative Case Study

James was a senior database administrator at a regional insurance carrier. He had held his position for eleven years and considered himself indispensable. When the company was acquired and a new CTO announced a technology consolidation, James's role was restructured and he was informed that his position would be eliminated in 90 days.

James accepted the redundancy package without outward complaint, but internally he began planning retaliation. In the final week of his employment, he used his still-active administrative credentials to access five production databases that supported the carrier's policy management system — the core of the company's operations.

Over three sessions across two evenings, James executed SQL DELETE and DROP TABLE commands targeting the active policy records table (2.1 million rows), the claims history table (890,000 rows), and the agent commission records table. He then connected to the backup server and deleted the most recent four daily backups for each affected database, leaving only week-old cold storage backups that were not immediately accessible. He also disabled the scheduled backup job for two of the three databases before logging out.

The deletion was discovered at 6:47 AM the following business day when the policy management system began returning null results and the operations team attempted to restore from backup. The discovery triggered a major incident: the organization was unable to process claims or issue policies for approximately 11 hours while IT worked to restore from cold storage.

The forensic investigation traced all activity to James's account and recovered timestamped logs confirming the deletions occurred during James's final week of employment. James was prosecuted under the Computer Fraud and Abuse Act and civil damages were sought. The organization's cyber insurance policy covered $4.2M of an estimated $6.8M total loss.

The incident would have been preventable — and detectable within minutes — had the organization implemented a control preventing database DROP TABLE and mass DELETE operations outside of an approved change window, and had backup systems been monitored for unauthorized deletion events.

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | MAL |
| **Typical Role** | Database administrator, system administrator, DevOps engineer, IT manager, cloud infrastructure engineer |
| **Access Level** | Administrator |
| **Technical Sophistication** | High — understands how databases work, how backups are structured, and how to maximize damage |
| **Motivation** | Revenge (most common), protest against management decision, coercion (rare), financial (rare — sometimes paired with ransom demand) |
| **Insider Knowledge Used** | Knowledge of which databases are most critical to operations; knowledge of backup architecture and where backups are stored; knowledge of how to maximize recovery time |

---

## 4. Attack Phases

### Phase 1 — Target Identification

**Description:** The actor identifies which systems and data stores will cause maximum operational disruption if destroyed. This may have been passively known for years or actively surveyed in the period leading up to the action.

**Observable Actions:**
- Unusual access to backup configuration documentation or backup management consoles
- Enumeration of database schemas or table inventories not accessed in normal work
- Access to DR (disaster recovery) runbooks or system dependency documentation
- Review of backup schedules and retention configurations

### Phase 2 — Backup Manipulation

**Description:** Before or during the deletion, the actor disables, deletes, or corrupts backup data to extend recovery time and maximize damage. This is the most sophisticated phase and may happen in a separate session from the deletion.

**Observable Actions:**
- Deletion of backup files from backup server or cloud backup storage
- Disabling of scheduled backup jobs
- Modification of backup retention policies
- Deletion of Volume Shadow Copies (Windows VSS) or snapshot schedules
- Access to backup management consoles outside of normal change windows

### Phase 3 — Data Destruction

**Description:** The actor executes destructive commands against target databases or file systems.

**Observable Actions:**
- Execution of DROP TABLE, TRUNCATE TABLE, or mass DELETE SQL statements in production
- File deletion commands at scale (del /s /q, rm -rf) against production data directories
- Encryption of files (if ransomware variant or to deny recovery)
- Execution of destructive scripts not associated with approved change tickets
- Database process termination followed by deletion of data files

### Phase 4 — Exiting and Misdirection

**Description:** The actor logs out and attempts to create plausible deniability or misdirect the investigation.

**Observable Actions:**
- Normal logout at the end of the session
- Possible creation of false audit artifacts (scripted activity from other accounts to create confusion)
- In rare cases, leaving a note or message (often recovered in forensics)
- Continued normal behavior in remaining workdays if action taken before the last day

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Mass DELETE or DROP TABLE SQL statement executed in production outside an approved change window | Database Activity Monitoring (DAM) | High |
| Backup files deleted from backup server by a user account that is not a backup service account | Backup system logs + SIEM | High |
| Scheduled backup job disabled outside an approved change window | Backup system audit log | High |
| Volume Shadow Copy (VSS) or snapshot deletion | Windows Event Log (EventID 524), cloud snapshot audit log | High |
| Large number of database rows deleted in a single session (threshold: > 10,000 rows outside a bulk operation ticket) | DAM | High |
| Access to backup management console by a non-backup-admin account | SIEM (authentication logs) | High |
| Destructive commands executed from an account within 7 days of a known departure date | SIEM correlated with HR system | High |
| Database data files deleted or overwritten at the OS level | EDR file monitoring | High |
| DROP INDEX or schema modification in production without a change ticket | DAM + change management | High |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Employee has been notified of termination, layoff, or role elimination | HR | Critical risk indicator — trigger immediate access review and enhanced monitoring for all admin accounts held |
| Employee has expressed anger, resentment, or threats | Management / HR incident report | Escalate to insider threat team immediately |
| Employee has a history of disciplinary action | HR | Contextual risk factor |
| Admin account holder is in their final 30 days of employment | HR offboarding list | Should trigger automatic enhanced monitoring of all privileged operations |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Detection Confidence |
|-------|-----------------------|---------------------|---------------------|
| Phase 1 — Target Identification | Alert on access to backup configuration systems by non-backup-admin accounts | SIEM | Medium |
| Phase 2 — Backup Manipulation | Alert on backup file deletion or backup job disablement outside change window | Backup system audit log + SIEM | High |
| Phase 2 — Backup Manipulation | Alert on VSS deletion or snapshot purge outside approved window | Windows Event Log / Cloud audit log | High |
| Phase 3 — Data Destruction | Alert on DROP TABLE or mass DELETE (> threshold rows) in production | DAM (real-time alerting required) | High |
| Phase 3 — Data Destruction | Alert on execution of bulk delete scripts outside change window | DAM + change management integration | High |
| All Phases | Departure-correlated monitoring: automatically escalate alert thresholds for all admin operations by accounts in their final 30 days | SIEM + HR integration | High |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Discovery | T1083 | File and Directory Discovery | Phase 1 — Target Identification |
| Discovery | T1082 | System Information Discovery | Phase 1 — Target Identification |
| Impact | T1485 | Data Destruction | Phase 3 — Data Destruction |
| Impact | T1490 | Inhibit System Recovery | Phase 2 — Backup Manipulation |
| Impact | T1489 | Service Stop | Phase 3 (if database service is stopped) |
| Defense Evasion | T1070.004 | Indicator Removal: File Deletion | Phase 4 — Misdirection |
| Impact | T1486 | Data Encrypted for Impact | Phase 3 (ransomware variant) |

---

## 8. Legal and Privacy Considerations

- **Immediate legal engagement:** Sabotage by an insider is typically criminal under the Computer Fraud and Abuse Act (US) or equivalent laws in other jurisdictions. Legal must be engaged immediately — evidence handling protocol is critical.
- **Evidence preservation:** Do not restore from backup before forensic imaging of affected systems. Evidence on the deleted databases (transaction logs, pre-deletion state) may be needed for prosecution. Work with Legal to sequence forensic imaging before operational recovery.
- **Chain of custody:** Maintain strict chain of custody for all evidence. Forensic copies should be handled by qualified examiners, not by the operations team.
- **Law enforcement:** Depending on the scale of damage, law enforcement referral may be appropriate. In the US, FBI Cyber Division handles significant CFAA violations. Coordinate timing with Legal to avoid compromising the investigation.
- **Cyber insurance:** Notify cyber insurance carrier immediately per policy requirements. Delayed notification may affect coverage.
- **Business continuity:** The tension between operational recovery and forensic preservation is real — Legal and IR must jointly manage this decision.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Declare major incident — activate IRP | IR Lead |
| 2 | Notify Legal immediately — do not proceed with HR action until Legal advises | Legal |
| 3 | Forensically image affected systems before any restoration activity | IR / Forensics |
| 4 | Identify and revoke all remaining access for the identified account | IAM / IT |
| 5 | Identify all accounts and systems touched in the malicious session | IR |
| 6 | Assess backup availability and determine recovery path with IT ops | IR + IT Ops |
| 7 | Notify executive leadership and Board if appropriate given the scale | CISO / Legal |
| 8 | Notify cyber insurance carrier | Legal / Finance |
| 9 | Coordinate law enforcement referral decision | Legal |
| 10 | Conduct forensic analysis to establish timeline and full scope | IR / Forensics |

**Linked Playbook:** PB-SB-001 (create using `playbooks/_template.md`)

---

## 10. Linked Artifacts

| Artifact Type | ID | File |
|---------------|----|------|
| Validation Test Case | TC-SB-001 | validation/test-cases/TC-SB-001.md (create using template) |
| Detection | DET-SB-001 | detections/ (create using template) |
| Hunt Hypothesis | HYP-SB-001 | hunting/hypotheses/HYP-SB-001.md (create using template) |

---

## 11. Related Scenarios

| Scenario ID | Name | Relationship |
|-------------|------|-------------|
| PM-001 | Privilege Escalation via Access Abuse | Administrative access is a prerequisite for this scenario |
| UA-001 | After-Hours Unauthorized System Access | Sabotage is almost always executed after hours |
| DE-001 | Cloud Storage Upload Exfiltration | Sometimes precedes sabotage — data theft then destruction |

---

## 12. References

- CMU CERT — Insider Threat Study: IT Sabotage
- MITRE ATT&CK — T1485: Data Destruction; T1490: Inhibit System Recovery
- DOJ CFAA prosecutions — United States v. various (IT sabotage cases)
- NIST SP 800-184 — Guide for Cybersecurity Event Recovery
- FBI — Combating Insider Threats

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-25 | SIDTVF Core Team | Initial creation |
