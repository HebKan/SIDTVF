# Scenario Card: PM-001 — Privilege Escalation via Access Abuse

| Field | Value |
|-------|-------|
| **Scenario ID** | PM-001 |
| **Name** | Privilege Escalation via Access Abuse |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-25 |
| **Last Updated** | 2026-03-25 |
| **Author** | SIDTVF Core Team |
| **Category** | Privilege Misuse |
| **Category Code** | PM |
| **Actor Archetype(s)** | MAL |
| **Severity** | High |
| **Minimum Maturity Level** | 3 |

---

## 1. Description

An insider with standard-level access systematically identifies and exploits weakly enforced access controls to gain access to systems, data, or functions beyond their authorized scope. Unlike external privilege escalation, this scenario uses legitimate pathways — requesting access that is not denied, reusing temporary elevated access beyond its intended window, using shared credentials, or leveraging administrative tools left accessible to non-admins. The goal is typically to access more valuable data or to take actions (such as modifying configurations or removing records) that their assigned role would not normally permit.

---

## 2. Narrative Case Study

Derek worked as a mid-level IT systems analyst at a healthcare organization. His role involved maintaining configuration documentation for clinical systems, which gave him read access to a broad range of administrative portals — but no write access or access to patient records.

Over a period of several months, Derek quietly tested which administrative interfaces did not enforce role-based access controls consistently. He discovered that the organization's legacy clinical data warehouse had an administrative console that had never been locked down — it was accessible from the internal network to any authenticated employee. He also discovered that two IT administrators frequently left remote desktop sessions open in a shared IT workspace, and that a shared "admin" account was used by the infrastructure team and its password was stored in a shared document he had read access to.

Using the shared admin credentials, Derek accessed the clinical data warehouse and began pulling patient demographic records and insurance information in batches. Over six weeks, he extracted data on approximately 40,000 patients. His motivation was financial — he intended to sell the data to a healthcare fraud operator he had contacted through an online forum.

Derek's activity was discovered not by the security team, but by a compliance audit that identified database access logs showing a large number of queries from a user account (the shared admin) at times when the infrastructure team was not on duty. The investigation revealed Derek's access to the shared credentials and his misuse of the unprotected administrative console.

The organization faced HIPAA breach notification obligations for 40,000 patients, regulatory penalties, and civil litigation. The shared credential and unprotected console — two straightforward misconfigurations — were the direct enablers of the breach.

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | MAL |
| **Typical Role** | IT analyst, system administrator, help desk, developer, database administrator |
| **Access Level** | Standard User or Power User with some administrative tool access |
| **Technical Sophistication** | Medium — understands how access controls work and how to probe for gaps |
| **Motivation** | Financial gain (data sale, fraud), competitive intelligence, curiosity, revenge |
| **Insider Knowledge Used** | Knowledge of which systems are poorly enforced; awareness of shared credentials; knowledge of which administrative tools are accessible from the internal network |

---

## 4. Attack Phases

### Phase 1 — Reconnaissance

**Description:** The insider surveys the environment to identify access control weaknesses. This may be passive (noticing things while doing normal work) or active (deliberately querying systems for accessible interfaces).

**Observable Actions:**
- Accessing administrative portals or documentation that is not required for current tasks
- Network scanning or enumeration from a workstation (rare in this scenario but possible)
- Accessing password management documentation or shared credential stores
- Browsing internal wikis or documentation for system architecture or admin credentials

### Phase 2 — Access Acquisition

**Description:** The insider obtains or uses elevated access — either by exploiting a weak control (unprotected console), using shared credentials, or leveraging access that was temporarily granted and never revoked.

**Observable Actions:**
- Login to systems using shared, service, or admin accounts
- Authentication from an account not normally associated with the target system
- Access to a system from a workstation or user account not historically associated with it
- Request for temporary elevated access that is then used beyond its intended scope
- Access during times when the legitimate owner of elevated credentials is not on duty

### Phase 3 — Abuse of Access

**Description:** The insider uses their acquired access to take actions — accessing restricted data, modifying records, disabling controls, or extracting information — that their legitimate role would not permit.

**Observable Actions:**
- Queries or access to data repositories outside the user's role scope
- Database queries generating results significantly larger than any previous legitimate query
- Configuration changes outside an approved change window
- Deletion or modification of records without a change ticket
- Access to personnel files, financial records, patient data, or other sensitive data categories

### Phase 4 — Exfiltration or Exploitation

**Description:** The insider uses the data or access gained for their intended purpose — exfiltrating data, committing fraud, sabotaging systems, or establishing persistence.

**Observable Actions:**
- (If data exfiltration): Overlaps with DE scenarios — large file transfers, cloud uploads
- (If fraud): Anomalous transactions, record modification in financial systems
- (If sabotage): Record deletion, configuration changes, service disruption

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Login to a privileged system from a user account not historically associated with that system | SIEM (authentication logs) | High |
| Use of a shared, service, or admin account from an unexpected workstation or user context | SIEM / PAM platform | High |
| Database queries returning large result sets from a user without DBA role | Database activity monitoring (DAM) | High |
| Access to a system during hours when the system's normal administrators are not on duty, attributed to a shared credential | SIEM | High |
| Temporary access request fulfilled but access not revoked after the project or ticket closes | IAM / ticketing system + access review | Medium |
| Access to administrative consoles not associated with the user's documented role | SIEM / PAM | High |
| Access to sensitive data categories (HR, patient, financial) not consistent with job function | SIEM / DAM | High |
| User accessing significantly more data objects than their 90-day baseline | UEBA | Medium |
| Configuration changes submitted outside an approved change window by a non-admin account | Change management + SIEM | High |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Access review found stale or over-provisioned accounts for this user | IAM / quarterly access review | Medium — signals poor hygiene that enables this scenario |
| Employee has expressed interest in systems outside their role | Management report | Low weight; contextual only |
| Employee in a position that provides visibility into access controls but no legitimate need to use them | Role documentation / HR | Baseline context |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Detection Confidence |
|-------|-----------------------|---------------------|---------------------|
| Phase 1 — Reconnaissance | Access to administrative consoles outside role scope | SIEM (auth logs correlated with RBAC policy) | Medium |
| Phase 2 — Access Acquisition | Shared/admin account used from unexpected workstation | PAM / SIEM | High |
| Phase 2 — Access Acquisition | Access to system by user with no prior history for that system | SIEM / UEBA (new resource access pattern) | High |
| Phase 3 — Abuse | Large data query from non-DBA account | Database Activity Monitoring (DAM) | High |
| Phase 3 — Abuse | Access to sensitive data category by non-authorized role | SIEM correlated with data classification | High |
| Phase 4 — Exfiltration | Large file transfer following database access | SIEM + network logs | High |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Discovery | T1087.002 | Account Discovery: Domain Account | Phase 1 — Reconnaissance |
| Discovery | T1083 | File and Directory Discovery | Phase 1 — Reconnaissance |
| Privilege Escalation | T1078.002 | Valid Accounts: Domain Accounts | Phase 2 — Access Acquisition |
| Credential Access | T1552.001 | Unsecured Credentials: Credentials in Files | Phase 2 (shared cred in document) |
| Collection | T1530 | Data from Cloud Storage Object | Phase 3 — Abuse |
| Collection | T1005 | Data from Local System | Phase 3 — Abuse |
| Exfiltration | T1567.002 | Exfiltration Over Web Service: Exfiltration to Cloud Storage | Phase 4 — Exfiltration |

---

## 8. Legal and Privacy Considerations

- **Shared credential use:** Evidence gathered from a shared account may be harder to attribute to a specific individual. Preserve all access path evidence (which workstation, which session, what time). Work with Legal before acting on shared-credential attribution.
- **HIPAA implications:** If patient data was accessed by an unauthorized insider, HIPAA breach notification obligations apply. Engage Legal immediately to scope the breach and assess notification timelines (typically 60 days from discovery).
- **Database activity monitoring:** DAM tools collect operational data about queries and results. Ensure that DAM data retention complies with applicable privacy law and that data collected does not itself constitute a secondary privacy risk.
- **Access control documentation:** During the investigation, document the access control gap that enabled the scenario (unprotected console, shared credentials, stale access). This documentation is both a remediation requirement and potential liability context — consult Legal before disclosing externally.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Disable the misused account(s) and revoke the acquired access at direction of Legal | IAM / IT |
| 2 | Preserve all logs: authentication, database query logs, network, endpoint | IR |
| 3 | Identify the full scope of access: what systems, what data, what timeframe | IR |
| 4 | Remediate the access control gap that enabled the scenario (remove shared credentials, lock the unprotected console) | IT Security |
| 5 | Notify Legal — assess breach obligations, criminal referral potential | Legal |
| 6 | Engage HR if the insider is still employed | Legal + HR |
| 7 | Conduct access review for all accounts with similar over-provisioning patterns | IAM |
| 8 | Review and revoke all temporary access grants that were not closed on schedule | IAM |

**Linked Playbook:** PB-PM-001 (create using `playbooks/_template.md`)

---

## 10. Linked Artifacts

| Artifact Type | ID | File |
|---------------|----|------|
| Validation Test Case | TC-PM-001 | validation/test-cases/TC-PM-001.md (create using template) |
| Detection | DET-PM-001 | detections/ (create using template) |
| Hunt Hypothesis | HYP-PM-001 | hunting/hypotheses/HYP-PM-001.md (create using template) |

---

## 11. Related Scenarios

| Scenario ID | Name | Relationship |
|-------------|------|-------------|
| DE-001 | Cloud Storage Upload Exfiltration | Frequently follows PM-001 — escalated access is then used to stage and exfiltrate data |
| UA-001 | After-Hours Unauthorized System Access | Often co-occurs — access abuse may happen after hours to reduce visibility |
| SB-001 | Targeted Database Deletion | Escalated access is a prerequisite for this scenario |

---

## 12. References

- CMU CERT — Insider Threat Study: IT Sabotage and Fraud
- MITRE ATT&CK — T1078: Valid Accounts
- NIST SP 800-53 — AC-6: Least Privilege
- HIPAA Security Rule — 45 CFR §164.312(a)(1): Access Control

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-25 | SIDTVF Core Team | Initial creation |
