# Scenario Card: TP-001 — Vendor Credential Abuse

| Field | Value |
|-------|-------|
| **Scenario ID** | TP-001 |
| **Name** | Vendor Credential Abuse |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-25 |
| **Last Updated** | 2026-03-25 |
| **Author** | SIDTVF Core Team |
| **Category** | Third-Party Risk |
| **Category Code** | TP |
| **Actor Archetype(s)** | TP, MAL |
| **Severity** | Critical |
| **Minimum Maturity Level** | 3 |

---

## 1. Description

A third-party vendor, contractor, or managed service provider misuses legitimate organizational access credentials — either through malicious intent, negligence, or because the vendor's own systems have been compromised — to access systems and data beyond the vendor's contracted scope. This scenario covers both deliberate misuse by a vendor employee and cases where the vendor's environment is compromised and the organization's credentials or access pathways are subsequently exploited by an external actor. Because vendors typically have broad, persistent, and privileged access to their clients' environments, this scenario often results in high-impact breaches.

---

## 2. Narrative Case Study

Harborview Medical System contracted with a regional IT managed services provider, Nexus IT Solutions, to manage their server infrastructure and provide 24/7 helpdesk support. As part of the engagement, Nexus IT was provisioned with domain admin credentials, VPN access, and access to Harborview's remote monitoring and management (RMM) platform.

The original contract had been scoped to server management and helpdesk. Over three years, access had never been formally reviewed, and Nexus IT's credentials had accumulated additional permissions as Harborview's IT team provisioned them "temporarily" for various projects — permissions that were never revoked.

Nexus IT employed a junior administrator, Rodrigo, who had been passed over for promotion and had recently started job hunting. Rodrigo discovered that his access to Harborview's network — via the shared Nexus IT VPN account — extended well beyond server management. He could access the electronic health record (EHR) system, the financial management system, and the HR platform.

Over a five-week period, Rodrigo used Harborview's RMM platform to execute scripts on 14 clinical workstations that harvested and compressed patient billing records. The data was then exfiltrated via an SFTP connection that the RMM platform's outbound rules permitted — consistent with legitimate administrative use. He extracted records for approximately 28,000 patients.

Harborview's security team had no visibility into activity performed through the RMM platform because they had trusted the vendor with administrative access and had not configured monitoring for vendor-initiated commands. The breach was discovered months later when patient data appeared for sale on a dark web forum — not through detection. An investigation confirmed Rodrigo's activity and found that Nexus IT had no audit logging for individual technician activity within the shared account.

The incident resulted in HIPAA breach notification for 28,000 patients, OCR investigation, and contract termination with Nexus IT. Harborview faced penalties and remediation costs exceeding $2.1M.

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | TP (primary), MAL (secondary — vendor employee acting with intent) |
| **Typical Role** | Managed service provider technician, IT contractor, software vendor support engineer, auditor with read access |
| **Access Level** | Power User to Administrator (vendor access is often highly privileged by necessity) |
| **Technical Sophistication** | Medium to High — vendor personnel are often technical professionals who understand the target environment |
| **Motivation** | Financial gain (data sale), theft of competitive intelligence, revenge against vendor employer, coercion |
| **Insider Knowledge Used** | Detailed knowledge of client network architecture (acquired through the vendor relationship); knowledge of which RMM or admin tools permit outbound data movement |

---

## 4. Attack Phases

### Phase 1 — Scope Enumeration

**Description:** The vendor employee discovers that their access extends beyond the contracted scope — either because they explore the environment or because they already knew from prior legitimate work.

**Observable Actions:**
- Vendor account accessing systems or directories not associated with their contracted service scope
- Enumeration queries against Active Directory or cloud IAM from a vendor account
- Access to network discovery tools or topology documentation
- Browsing of file shares or application menus beyond the vendor's service area

### Phase 2 — Target Identification

**Description:** The actor identifies data or systems of interest. Given their administrative access, this typically involves accessing system inventories, data classification metadata, or directly browsing high-value repositories.

**Observable Actions:**
- Vendor account accessing data repositories unrelated to contracted service (e.g., EHR system accessed by an infrastructure vendor)
- Queries against sensitive databases using vendor or shared admin credentials
- Access to file shares containing confidential data categories

### Phase 3 — Data Collection and Staging

**Description:** The actor collects target data — often using legitimate administrative tools (RMM scripts, PowerShell, database query tools) that are part of their normal toolset. This makes the activity harder to distinguish from legitimate work.

**Observable Actions:**
- Execution of scripts on client endpoints via RMM platform (vendor-initiated commands targeting data)
- File compression or collection commands executed on client systems
- Creation of staging directories or archives on client systems or vendor-accessible storage
- Database queries returning large datasets from sensitive tables

### Phase 4 — Exfiltration via Vendor Channel

**Description:** Data is exfiltrated using channels already permitted for vendor use — RMM file transfer, SFTP, vendor VPN tunnel, remote desktop session, or vendor-managed cloud storage. These channels may not be monitored because they are trusted.

**Observable Actions:**
- Large data transfers over vendor-authorized channels (RMM, SFTP, VPN)
- File transfer to a destination not associated with the vendor's normal operational destinations
- Data leaving the organization via vendor VPN to a non-vendor IP address
- SFTP or SCP transfer volume significantly above the vendor's normal baseline

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Vendor account accessing systems outside the documented service scope | SIEM (authentication logs correlated with vendor scope documentation) | High |
| Vendor RMM platform executing scripts that collect or compress data on client endpoints | EDR (process monitoring on client endpoints) | High |
| Large outbound data transfer via vendor VPN or SFTP channel, above vendor's historical baseline | NGFW / network monitoring | High |
| Vendor account access at times outside the vendor's contracted service hours or SLA window | SIEM (authentication logs correlated with vendor SLA) | High |
| Vendor account authenticating from an IP address not registered as a Nexus IT / vendor IP range | SIEM + vendor IP allowlist | High |
| Individual vendor employee cannot be attributed within shared vendor account (no sub-account audit trail) | Vendor account audit log | Medium (signals audit gap, not incident) |
| Vendor access to sensitive data categories (PHI, PII, financial) without a support ticket justifying the access | SIEM + ticketing system integration | High |
| Vendor account creating new local accounts or modifying group memberships on client systems | SIEM / EDR | High |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Vendor contract is expired or being renegotiated (tension exists) | Procurement / Vendor Management | Medium risk context |
| Vendor access has never been formally reviewed since initial provisioning | IAM / quarterly access review | Medium — signals hygiene gap |
| Vendor has experienced a recent personnel change (key contact left) | Vendor relationship manager | Low weight; context for investigation |
| Vendor is facing financial distress or operational difficulties | Vendor risk management | Low weight; context |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Detection Confidence |
|-------|-----------------------|---------------------|---------------------|
| Phase 1 — Scope Enumeration | Alert on vendor account accessing systems outside documented service scope | SIEM (scope policy alert) | High |
| Phase 2 — Target Identification | Alert on vendor account accessing sensitive data categories not associated with service | SIEM + data classification | High |
| Phase 3 — Collection | Alert on RMM-initiated scripts that create archives or collect data files | EDR (command line monitoring on endpoints) | High |
| Phase 4 — Exfiltration | Alert on large outbound transfer via vendor channel above vendor baseline | NGFW / proxy / SIEM | High |
| Phase 4 — Exfiltration | Alert on SFTP/RMM file transfer to destination IP outside vendor's registered IP range | NGFW | High |
| All Phases | Monthly vendor access review: audit all vendor account permissions vs. contracted scope | IAM / vendor management process | Medium (preventive) |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Initial Access | T1078.004 | Valid Accounts: Cloud Accounts | Phase 1 (if cloud environment) |
| Initial Access | T1195.001 | Supply Chain Compromise: Compromise Software Dependencies | Phase 1 (if RMM platform is the vector) |
| Discovery | T1087 | Account Discovery | Phase 1 — Scope Enumeration |
| Discovery | T1018 | Remote System Discovery | Phase 1 — Scope Enumeration |
| Collection | T1005 | Data from Local System | Phase 3 — Data Collection |
| Collection | T1560.001 | Archive Collected Data: Archive via Utility | Phase 3 — Staging |
| Exfiltration | T1048.002 | Exfiltration Over Alternative Protocol: Exfiltration Over Asymmetric Encrypted Non-C2 Protocol | Phase 4 (SFTP) |
| Command and Control | T1219 | Remote Access Software | Phase 3, Phase 4 (RMM platform abuse) |

---

## 8. Legal and Privacy Considerations

- **Contractual authority:** Before disabling vendor access, review the contract for notification requirements, SLA obligations, and breach provisions. Disabling access without contract review may create contractual liability. Coordinate with Procurement and Legal simultaneously.
- **Evidence from vendor systems:** Evidence of the incident may reside on the vendor's own systems (Nexus IT's internal audit logs, the vendor employee's workstation). You cannot compel production of this evidence without legal process (subpoena, civil discovery, law enforcement request). Legal must be engaged early.
- **Shared accounts:** Activity from shared vendor accounts (one account used by multiple vendor personnel) may be difficult to attribute to a specific individual. This is both a legal challenge and a vendor governance finding. Document the attribution challenge in your incident report.
- **HIPAA:** If PHI was involved, the vendor is a Business Associate under HIPAA. The Business Associate Agreement (BAA) governs the vendor's breach notification obligations. Review the BAA before communicating with the vendor about the incident.
- **Victim vs. vendor accountability:** If the vendor's environment was compromised and the actor is external (not the vendor's employee), the vendor may be a victim. Determine the nature of the incident before assigning liability.
- **Concurrent notification:** Third-party incidents often trigger concurrent notification obligations — to affected individuals, to regulators (OCR for HIPAA, SEC for public companies, DPA for GDPR), and to the vendor. Coordinate timing carefully.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Notify Legal and Procurement simultaneously before any vendor-facing action | Legal / Procurement |
| 2 | Revoke or restrict vendor access (after Legal reviews contract terms) | IAM / IT |
| 3 | Preserve all logs attributable to the vendor account (authentication, RMM, SFTP, network) | IR |
| 4 | Identify scope: what systems were accessed, what data was accessed, what was exfiltrated | IR |
| 5 | Review all vendor accounts for similar scope deviation or over-provisioning | IAM |
| 6 | Request the vendor's own investigation and audit logs (per contract provisions) | Legal / Procurement |
| 7 | Assess breach notification obligations (HIPAA BAA, GDPR, etc.) | Legal / Privacy |
| 8 | Review and remediate vendor account lifecycle management process | IAM / Vendor Management |
| 9 | Consider law enforcement referral | Legal |

**Linked Playbook:** PB-TP-001 (create using `playbooks/_template.md`)

---

## 10. Linked Artifacts

| Artifact Type | ID | File |
|---------------|----|------|
| Validation Test Case | TC-TP-001 | validation/test-cases/TC-TP-001.md (create using template) |
| Detection | DET-TP-001 | detections/ (create using template) |
| Hunt Hypothesis | HYP-TP-001 | hunting/hypotheses/HYP-TP-001.md (create using template) |

---

## 11. Related Scenarios

| Scenario ID | Name | Relationship |
|-------------|------|-------------|
| PM-001 | Privilege Escalation via Access Abuse | Vendor over-provisioning is a form of access escalation |
| DE-001 | Cloud Storage Upload Exfiltration | Alternative exfiltration channel may be used if RMM isn't available |
| AC-001 | Account Compromise (when created) | Vendor account may be the target of external compromise |

---

## 12. References

- CMU CERT — Insider Threat in the Supply Chain
- MITRE ATT&CK — T1195: Supply Chain Compromise; T1219: Remote Access Software
- CISA — Defending Against Software Supply Chain Attacks
- HIPAA Business Associate Agreement requirements (45 CFR §164.308(b))
- Verizon DBIR — Third-Party Involvement in Data Breaches

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-25 | SIDTVF Core Team | Initial creation |
