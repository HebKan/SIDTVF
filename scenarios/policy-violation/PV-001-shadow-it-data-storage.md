# Scenario Card: PV-001 — Shadow IT Data Storage

| Field | Value |
|-------|-------|
| **Scenario ID** | PV-001 |
| **Name** | Shadow IT Data Storage |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-26 |
| **Last Updated** | 2026-03-26 |
| **Author** | SIDTVF Core Team |
| **Category** | Policy Violation |
| **Category Code** | PV |
| **Actor Archetype(s)** | NEG |
| **Severity** | Medium |
| **Minimum Maturity Level** | 2 |

---

## 1. Description

An employee stores organizational data — including sensitive documents, customer records, or work product — in an unsanctioned personal cloud storage account, personal device, or unapproved third-party application, without intent to cause harm. The motivation is convenience or a perceived gap in organizational tooling. Despite lacking malicious intent, the behavior creates material risk: data outside organizational control cannot be protected, audited, or recovered, and creates potential compliance exposure (GDPR, HIPAA, SOX).

---

## 2. Narrative Case Study

Sandra was a senior account manager at a financial advisory firm. Her role required frequent travel, and she often found herself needing to access client presentations and compliance documents from her personal iPad when working from hotels or airports. The firm's officially sanctioned document management system had poor mobile support, required VPN, and frequently timed out on hotel Wi-Fi.

Out of frustration, Sandra began syncing her work documents to her personal iCloud Drive account. Over 14 months, she accumulated approximately 2,200 documents, including client financial plans, signed advisory agreements, and internal pricing models for high-net-worth clients.

Sandra had no intent to misuse the data. She used it solely for work. She deleted most documents from iCloud when she was done with them. However, she did not consistently delete them, and some documents accumulated in her personal account for months.

The issue came to light during a routine IT compliance audit that used a cloud access security broker (CASB) to inventory cloud service usage across corporate devices. The audit found consistent iCloud uploads from Sandra's managed laptop, flagging it as an unapproved storage destination. The compliance team discovered 340 documents still resident in Sandra's personal iCloud account, including 89 containing client PII subject to state privacy law requirements.

Sandra cooperated fully. Legal assessed that the 89 documents containing PII created a potential breach notification obligation under the state consumer protection law, since the data had been outside the firm's control for an extended period without the firm's knowledge. The firm ultimately issued a precautionary notification to affected clients.

The incident was not driven by Sandra's malice — it was driven by a gap between the firm's policy and its tooling. The sanctioned alternative was too inconvenient, so Sandra used a personal tool that worked.

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | NEG |
| **Typical Role** | Any role that works remotely, travels frequently, or experiences friction with sanctioned tools |
| **Access Level** | Standard User |
| **Technical Sophistication** | Low — using personal cloud storage requires no technical skill |
| **Motivation** | Convenience; frustration with organizational tooling; working remotely without VPN friction |
| **Policy Knowledge Gap** | Often unaware that personal cloud storage constitutes a policy violation, or aware but underestimates the compliance risk |

---

## 4. Attack Phases

> Note: Shadow IT scenarios use "phases" to describe the behavior progression, not a malicious attack chain.

### Phase 1 — Sanctioned Tool Friction
**Description:** Employee encounters friction with the approved tool — poor performance, unavailability, poor mobile support, or missing features.
**Observable Actions:**
- Help desk tickets for the sanctioned tool from this user
- VPN connection failures preceding personal cloud uploads
- High latency or access errors on the sanctioned document system (service logs)

### Phase 2 — Personal Storage Adoption
**Description:** Employee begins using a personal cloud service as a workaround.
**Observable Actions:**
- First-time browser access to a personal cloud storage domain (iCloud.com, drive.google.com, dropbox.com) from a corporate device
- Installation of a personal cloud sync client on a corporate endpoint
- CASB alert on cloud service usage by an unenrolled application

### Phase 3 — Data Accumulation
**Description:** Work data accumulates in the personal account over time. Sensitivity increases as more documents are uploaded.
**Observable Actions:**
- Regular uploads to personal cloud domain from corporate device (daily or weekly pattern)
- DLP match on content uploaded to personal cloud (if DLP is deployed)
- CASB showing persistent data residency in an unsanctioned service

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Regular uploads to personal cloud storage domain from a managed corporate device | Web proxy logs / CASB | High |
| Personal cloud sync client (iCloud Drive, Dropbox, Google Drive personal) installed on corporate endpoint | MDM / EDR | High |
| DLP match on content uploaded to personal cloud service | DLP platform | High |
| CASB classification of an unsanctioned cloud service in use by this user | CASB | High |
| Sync client running in background during business hours and uploading periodically | EDR (process + network event correlation) | Medium |
| Uploads to personal cloud domains from unmanaged (personal) devices on the corporate Wi-Fi network | Network NAC / Wi-Fi access logs | Low–Medium |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Employee has filed help desk tickets for the sanctioned storage tool | ITSM | Explains motivation — should prompt tooling review, not punishment |
| Employee works remotely frequently or travels extensively | HR / Manager | Remote workers are disproportionately likely to use personal cloud for convenience |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Confidence |
|-------|-----------------------|---------------------|-----------|
| Phase 2 | First-time personal cloud domain access from corporate device | Web proxy + CASB | High |
| Phase 2 | Personal sync client installation on managed endpoint | MDM / EDR | High |
| Phase 3 | Regular upload pattern to personal cloud (not a one-time event) | Web proxy + CASB | High |
| Phase 3 | DLP match on content type (e.g., documents containing PII fields) | DLP endpoint agent | High |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Exfiltration | T1567.002 | Exfiltration Over Web Service: Exfiltration to Cloud Storage | Phase 3 |

> Note: Shadow IT policy violations map minimally to ATT&CK because they are not adversarial. The single applicable technique reflects the technical behavior, not malicious intent. This scenario is primarily governed by compliance frameworks rather than threat intelligence frameworks.

---

## 8. Legal and Privacy Considerations

- **Data outside organizational control:** Once data is in a personal cloud account, the organization has no ability to enforce retention, deletion, or access controls. If the data includes PII subject to GDPR, HIPAA, or state privacy law, this may trigger breach notification obligations even without malicious intent.
- **Response proportionality:** Shadow IT by a NEG actor does not warrant the same response as malicious data theft. Response should be educational and procedural first, escalating only for repeated violations or if sensitive data is involved. Avoid treating this as a criminal matter unless evidence of intent emerges.
- **Tooling review:** If shadow IT is widespread, it signals a gap in organizational tooling — not just individual misbehavior. The appropriate organizational response often includes improving the sanctioned alternative, not just punishing users of the unsanctioned one.
- **Data recovery:** Organizations have no legal authority to access or delete data from an employee's personal cloud account without their consent. Recovery requires the employee's voluntary cooperation. Legal should advise on whether a formal data recovery request is appropriate.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Contact the employee (or their manager) — explain the policy and the risk | HR / Manager |
| 2 | Request voluntary deletion of organizational data from the personal account | Manager + Legal |
| 3 | Assess whether any sensitive data categories (PII, PHI, PCI) are involved | Privacy / Legal |
| 4 | If sensitive data is confirmed — assess breach notification obligations | Legal |
| 5 | Address the tooling gap that motivated the behavior — submit ticket to IT for sanctioned tool improvement | Manager + IT |
| 6 | Provide policy acknowledgment or targeted training | HR |
| 7 | If repeated after first intervention — escalate to formal HR process | HR + Legal |

**Linked Playbook:** PB-PV-001 (create using `playbooks/_template.md`)

---

## 10. Linked Artifacts

| Artifact Type | ID | File |
|---------------|----|------|
| Validation Test Case | TC-PV-001 | validation/test-cases/TC-PV-001.md (create) |
| Detection | DET-PV-001 | detections/ (create) |

---

## 11. Related Scenarios

| Scenario ID | Name | Relationship |
|-------------|------|-------------|
| DE-001 | Cloud Storage Upload Exfiltration | Same technical behavior; different intent. PV-001 is NEG; DE-001 is MAL. Triage must distinguish intent. |
| AC-001 | Account Compromise | Personal cloud accounts containing work data are a secondary attack surface for external actors |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-26 | SIDTVF Core Team | Initial creation |
