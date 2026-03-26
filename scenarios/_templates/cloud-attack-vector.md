# Scenario Template: Cloud Attack Vector

> Pre-populated for scenarios where the primary threat behavior occurs via cloud services
> (SaaS apps, IaaS/PaaS APIs, cloud storage, cloud-hosted applications).
> Copy to the appropriate category directory, rename, and fill in `[PLACEHOLDER]` fields.
> See base `scenarios/_template.md` for full field guidance.

---

## Scenario Card

| Field | Value |
|-------|-------|
| **Scenario ID** | [e.g., DE-003] |
| **Name** | [Short name] |
| **Version** | 1.0 |
| **Status** | Draft |
| **Created** | [YYYY-MM-DD] |
| **Last Updated** | [YYYY-MM-DD] |
| **Author** | [Name] |
| **Category** | [Category] |
| **Actor Archetype(s)** | [MAL / NEG / COM / TP] |
| **Severity** | [Critical / High / Medium / Low] |
| **Minimum Maturity Level** | 3 |
| **Attack Vector** | Cloud |
| **Primary Cloud Environment** | [AWS / Azure / GCP / M365 / Multi-Cloud / SaaS-specific] |

---

## 1. Description

[Describe the threat behavior. Focus on what the insider does in the cloud environment — which APIs, which services, what data is targeted.]

---

## 2. Narrative Case Study

[3–6 paragraphs. Include: which cloud services are involved, how the insider leverages legitimate cloud access, why the behavior evaded detection, and the eventual discovery.]

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | [MAL / NEG / COM / TP] |
| **Typical Role** | [e.g., Cloud Engineer, DevOps, Data Engineer — roles with IaaS/SaaS access] |
| **Access Level** | [Standard / Power / Admin] |
| **Technical Sophistication** | [Medium / High — cloud attacks require some technical familiarity] |
| **Motivation** | [Financial gain / IP theft / sabotage / carelessness] |
| **Cloud Knowledge Used** | [e.g., "Knows S3 bucket ACLs are not monitored by default"; "Aware that CloudTrail logging is disabled in dev accounts"] |

---

## 4. Attack Phases

### Phase 1 — Cloud Environment Reconnaissance
**Description:** Insider surveys cloud environment for valuable data stores or misconfigured resources.
**Observable Actions:**
- Cloud API calls to list buckets, blobs, or storage resources (ListBuckets, ListObjects, az storage container list)
- Console access to cloud storage or data services outside normal work scope
- IAM permission enumeration (GetAccountAuthorizationDetails, ListRoles)

### Phase 2 — [Target Access]
**Description:** [How the insider accesses the cloud resource containing the target data]
**Observable Actions:**
- [Cloud-specific access event]
- [API call or console action]

### Phase 3 — [Data Collection or Action]
**Description:** [How data is collected or the harmful action is taken in the cloud environment]
**Observable Actions:**
- [API download/copy/export call]
- [Data transfer event]

### Phase 4 — [Exfiltration or Impact]
**Description:** [How data leaves or how the cloud environment is harmed]
**Observable Actions:**
- [Outbound transfer or API-based exfiltration event]

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs — Cloud-Specific

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| ListBuckets / ListObjects API calls by a non-infrastructure account | AWS CloudTrail | Medium |
| GetObject API calls on a bucket outside the user's normal work scope | AWS CloudTrail / S3 access logs | High |
| Console login from a new IP or geographic location | AWS CloudTrail (ConsoleLogin), Azure Sign-in logs | High |
| API calls generating large data downloads (GetObject with high byte count) | CloudTrail + S3 server access logs | High |
| IAM policy changes outside an approved change window | CloudTrail (PutRolePolicy, AttachUserPolicy) | High |
| Cloud resource creation in an unexpected region | CloudTrail / Azure Activity Log | Medium |
| Storage ACL changes making a resource publicly accessible | CloudTrail (PutBucketAcl), Azure Blob policy events | High |
| Programmatic API access (access key) after business hours | CloudTrail (sourceIPAddress not AWS service) | Medium |

### Technical IOBs — Contextual

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Access key created and used within the same session (may indicate credential staging) | CloudTrail | Medium |
| API call volume significantly above the user's 30-day baseline | SIEM + CloudTrail | High |
| [Add scenario-specific IOBs here] | | |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Employee has access to cloud infrastructure but their role has changed | IAM / HR | Access may be over-provisioned |
| Departing employee holds cloud IAM permissions | HR + IAM | Trigger immediate access review |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Confidence |
|-------|-----------------------|---------------------|-----------|
| Phase 1 | IAM enumeration API calls from unexpected account | CloudTrail + SIEM | Medium |
| Phase 2 | Access to cloud storage outside role scope | CloudTrail + RBAC policy mapping | High |
| Phase 3 | High-volume GetObject/download from cloud storage | CloudTrail + S3/Blob access logs | High |
| Phase 4 | [Exfiltration detection opportunity] | [Data source] | [Confidence] |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Discovery | T1580 | Cloud Infrastructure Discovery | Phase 1 |
| Discovery | T1619 | Cloud Storage Object Discovery | Phase 1 |
| Collection | T1530 | Data from Cloud Storage Object | Phase 3 |
| Exfiltration | T1537 | Transfer Data to Cloud Account | Phase 4 |
| [Add additional techniques] | | | |

---

## 8. Legal and Privacy Considerations

- **Cloud audit log availability:** Ensure CloudTrail (AWS), Azure Activity Log, or GCP Audit Logs are enabled for all accounts in scope. Many cloud accounts have logging disabled by default in dev/test environments — attackers exploit this.
- **Cross-account activity:** Insider activity may span multiple cloud accounts. Ensure cross-account log aggregation is in scope.
- **Access key vs. console access:** API key-based access may not be attributed to an individual without key ownership mapping. Maintain an up-to-date inventory of access key ownership.
- **[Add scenario-specific considerations]**

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Preserve CloudTrail / audit logs before any account changes | IR |
| 2 | Revoke the relevant cloud credentials (access keys, console access) | IAM / Cloud Admin |
| 3 | Notify Legal — assess whether cloud data exposure triggers breach notification | Legal |
| 4 | Assess whether data was made publicly accessible (check bucket/blob ACLs) | IR + Cloud Admin |
| 5 | [Additional scenario-specific steps] | |

**Linked Playbook:** [PB-XX-NNN — create using `playbooks/_template.md`]

---

## 10. Linked Artifacts

| Artifact Type | ID | File |
|---------------|----|------|
| Validation Test Case | TC-[ID] | validation/test-cases/TC-[ID].md |
| Detection | DET-[ID] | detections/ |
| Hunt Hypothesis | HYP-[ID] | hunting/hypotheses/HYP-[ID].md |
| Response Playbook | PB-[ID] | playbooks/ |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | [YYYY-MM-DD] | [Author] | Initial creation |
