# Scenario Card: DE-001 — Cloud Storage Upload Exfiltration

| Field | Value |
|-------|-------|
| **Scenario ID** | DE-001 |
| **Name** | Cloud Storage Upload Exfiltration |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-25 |
| **Last Updated** | 2026-03-25 |
| **Author** | SIDTVF Core Team |
| **Category** | Data Exfiltration |
| **Category Code** | DE |
| **Actor Archetype(s)** | MAL, NEG |
| **Severity** | Critical |
| **Minimum Maturity Level** | 2 |

---

## 1. Description

An insider uploads sensitive organizational data — including intellectual property, customer records, source code, or financial data — to a personal or unauthorized cloud storage service (e.g., Google Drive, Dropbox, Box, OneDrive personal). The data is staged for removal from organizational control, either to benefit a competitor, a future employer, or for personal financial gain. This scenario also includes negligent insiders who upload work data to personal accounts for convenience without understanding the risk.

---

## 2. Narrative Case Study

Meridian Analytics, a mid-sized financial data firm, employed a senior data scientist named Marcus who had been with the company for six years. In his final four months of employment — before quietly accepting a role at a direct competitor — Marcus systematically downloaded the firm's proprietary customer segmentation models, training datasets, and algorithm documentation to his work laptop.

Over a three-week period, Marcus accessed the company's internal data repository each evening after most colleagues had logged off. He organized the files into a folder structure on his local machine, then uploaded them in batches to a personal Google Drive account using his corporate laptop's browser. He did not use any specialized tools — he simply accessed drive.google.com and dragged files into a web upload interface.

Meridian's DLP system flagged two of the uploads as containing financial data keywords, but the alerts were queued in a backlog. The network monitoring system generated no alert because the uploads were made via HTTPS to a domain (drive.google.com) that was allowed on the corporate web filter for personal productivity use.

When Marcus resigned and gave two weeks' notice, the departing-employee workflow triggered a routine access review. A security analyst reviewing Marcus's recent activity noticed the pattern: 847 file accesses in the final month vs. an average of 94 per month in the prior year, and 23 uploads to cloud.google.com via the corporate proxy, where Marcus's personal history showed near-zero cloud uploads previously.

The investigation confirmed Marcus had exfiltrated approximately 14 GB of data, including the firm's most commercially sensitive algorithm suite. Legal counsel was engaged, and a civil action was filed. However, because Meridian's DLP had not been tuned to detect personal Google Drive uploads specifically, and because the proxy logs had not been actively monitored, the exfiltration had gone undetected for 19 days.

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | MAL (primary), NEG (secondary — also applies to accidental policy violators) |
| **Typical Role** | Any role with access to sensitive data: data scientist, engineer, sales, finance, HR |
| **Access Level** | Standard User to Power User |
| **Technical Sophistication** | Low (browser-based upload requires no technical skill) |
| **Motivation** | MAL: financial gain, competitive advantage, revenge. NEG: convenience, ignorance of policy |
| **Insider Knowledge Used** | Knowledge of which data repositories contain the most valuable data; knowledge of what is and is not blocked by the web filter |

---

## 4. Attack Phases

### Phase 1 — Data Discovery and Staging

**Description:** The insider identifies and accesses the target data. Files are often accessed from multiple systems or repositories and organized locally before exfiltration.

**Observable Actions:**
- Accesses data repositories significantly above their historical baseline (volume or frequency)
- Accesses files or directories outside their normal working scope
- Creates new local folders or copies of files on the endpoint
- May compress files (ZIP, 7z, tar) to reduce upload time and file count

### Phase 2 — Channel Selection

**Description:** The insider selects an upload destination. Technically unsophisticated insiders use whichever personal cloud service they already have. More deliberate insiders may create a new account specifically for the transfer to reduce traceability.

**Observable Actions:**
- Browser navigation to personal cloud storage URLs (drive.google.com, dropbox.com, box.com, onedrive.live.com, icloud.com, mega.nz, etc.)
- First-time access to a personal cloud service domain from a corporate device
- Use of a browser profile or private browsing mode not consistent with work use

### Phase 3 — Upload Execution

**Description:** Files are uploaded via browser or installed desktop sync clients. Large volumes may be uploaded in multiple sessions spread across days or weeks to avoid volume-based detection.

**Observable Actions:**
- Large HTTPS POST requests to cloud storage domains
- High outbound data volume to a single external domain in a short window
- Multiple upload sessions from the same source IP over several days
- DLP alert generated for content policy match (if DLP is tuned for this)
- File sync client processes running on the endpoint (Dropbox.exe, GoogleDriveFS.exe)

### Phase 4 — Cover and Continuation

**Description:** In deliberate cases, the insider may attempt to cover their tracks or continue normal work activities to avoid suspicion. Some insiders delete local copies of files after upload.

**Observable Actions:**
- Deletion of local staging folders after upload sessions
- Clearing of browser history or download history
- Continuing normal business activity the following day
- No unusual login or access patterns following the upload

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Volume of files accessed per day significantly above personal 90-day baseline | SIEM (file access logs, EDR) | High |
| Access to files or directories not accessed in the past 6 months | EDR, file access audit logs | Medium |
| HTTPS POST requests to known personal cloud storage domains | Web proxy / NGFW logs | High |
| Outbound data volume to personal cloud domain exceeds 500 MB in a single session | Web proxy / DLP | High |
| Compression utility (ZIP, 7-zip, WinRAR) used on a collection of data files | EDR process logs | Medium |
| DLP policy match for sensitive data categories (PII, financial data, IP keywords) | DLP platform | High |
| Personal cloud sync client installed on corporate endpoint | MDM / EDR | Medium |
| Browser private mode used in combination with cloud storage domain access | EDR (browser logs) | Medium |
| Access patterns shift to after-hours (outside user's established working hours) | SIEM (auth + file access logs) | Medium |
| Departure-correlated spike: activity increase within 60 days of resignation or termination notice | SIEM correlated with HR system | High |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Employee has given or is expected to give notice of departure | HR system / Management | Significantly elevates risk; trigger enhanced monitoring of data access |
| Employee has expressed dissatisfaction or grievances | HR / Management | Low-weight signal; cannot be acted on alone |
| Employee has accepted a role at a competitor (if known) | HR / Management | High-weight signal; trigger immediate access review |
| Employee's performance review is pending or recent rating was negative | HR | Low-weight contextual signal |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Detection Confidence |
|-------|-----------------------|---------------------|---------------------|
| Phase 1 — Staging | Spike in file access volume vs. 90-day personal baseline | File audit logs (Windows Event 4663, Linux auditd) ingested into SIEM/UEBA | High |
| Phase 1 — Staging | Access to repositories outside normal scope | SIEM correlated with role-based access policy | Medium |
| Phase 2 — Channel Selection | Browser navigation to personal cloud domains | Web proxy logs, NGFW with URL categorization | High |
| Phase 3 — Upload | Large HTTPS POST to cloud storage domain | Web proxy / NGFW SSL inspection | High |
| Phase 3 — Upload | DLP policy match on content | DLP (endpoint or network) | High (if tuned) |
| Phase 3 — Upload | Outbound data volume anomaly | NGFW, proxy logs | Medium |
| Phase 4 — Cover | File deletion spike following access spike | EDR file deletion events | Medium |

---

## 7. Minimum Viable Detection

> **If you can only implement one control:** Deploy a daily cumulative upload volume query against your web proxy logs. Alert when any single user's total bytes sent to personal cloud storage domains exceeds 200 MB in a 24-hour period. This single rule catches both large single uploads AND adversaries who split uploads into small batches to evade per-session thresholds. It requires only proxy logs — no endpoint agent, no DLP, no UEBA.
>
> Query reference: `detections/queries/DE-001-spl.md` Query 3 (cumulative daily volume), or `detections/queries/DE-001-kql.md` Query 3.

---

## 7a. Detection Blind Spots

The following known gaps exist in DE-001 detection coverage. Document these so your team understands what is and is not covered.

| Blind Spot | Why It Evades Detection | Mitigating Control |
|-----------|------------------------|-------------------|
| Personal device on home or cellular network | Completely outside corporate proxy visibility | MDM/CASB mobile agent; policy prohibiting sensitive data on personal devices |
| Incognito / Private browser mode | Proxy logs typically cannot distinguish Incognito mode from normal browsing | Endpoint agent captures regardless of browser mode |
| Personal VPN or proxy bypass | User routes traffic through a personal VPN before uploading; destination appears as VPN endpoint | Full-tunnel VPN policy; CASB agent-based detection |
| Slow exfiltration (< 5 MB/day over weeks) | Well below any volume threshold; pattern only detectable over 60–90 day lookback | UEBA behavioral baseline over extended window |
| Uploading via corporate-sanctioned tool (OneDrive personal partition) | Destination domain is the same as the corporate tool | CASB with user identity awareness to distinguish personal vs. corporate account |
| Screenshot exfiltration (phone camera) | Completely invisible to all electronic monitoring | Physical security controls; clean desk policy |
| Air-gap via USB to personal device followed by upload | USB write event is detectable; connection to personal device then uploads is outside visibility | USB write detection + endpoint DLP |

---

## 8. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Collection | T1213 | Data from Information Repositories | Phase 1 — Staging |
| Collection | T1005 | Data from Local System | Phase 1 — Staging |
| Collection | T1560.001 | Archive Collected Data: Archive via Utility | Phase 1 — Staging (compression) |
| Exfiltration | T1567.002 | Exfiltration Over Web Service: Exfiltration to Cloud Storage | Phase 3 — Upload |
| Defense Evasion | T1070.004 | Indicator Removal: File Deletion | Phase 4 — Cover |

---

## 8. Legal and Privacy Considerations

- **Web proxy monitoring:** Review of HTTPS traffic to personal cloud domains is generally permissible with notice. SSL/TLS inspection requires clear disclosure in the acceptable use policy and may require additional consent in some jurisdictions.
- **DLP content inspection:** Content-aware DLP that reads file content triggers higher privacy sensitivity. Ensure the acceptable use policy explicitly authorizes content inspection and that data retention for DLP logs is defined.
- **Personal accounts on corporate devices:** In jurisdictions with strong personal privacy protections (Germany, California), monitoring activity on personal accounts accessed from corporate devices may require careful scoping — consult Legal.
- **Departure-correlated monitoring:** Increasing monitoring intensity based on HR events (notice of departure, negative review) must be documented as a program-wide policy and cannot be applied selectively in a discriminatory manner.
- **Evidence handling:** If criminal referral is possible (trade secret theft), preserve logs and evidence per legal hold requirements before disabling the account.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Preserve all relevant logs (proxy, file access, DLP, endpoint) before account changes | SOC / IR |
| 2 | Notify Legal and determine whether covert or overt investigation is appropriate | Legal |
| 3 | Identify the full scope of data accessed and uploaded (what, how much, when) | IR |
| 4 | Engage HR for parallel action if employment consequences are anticipated | Legal + HR |
| 5 | Disable or restrict the user's account at the direction of Legal | IT / IAM |
| 6 | Contact the destination cloud provider's legal team if data recovery is needed | Legal |
| 7 | Assess whether law enforcement referral is warranted | Legal |
| 8 | Notify affected parties if personal data was exfiltrated (breach notification obligations) | Legal |

**Linked Playbook:** [PB-DE-001](../../playbooks/PB-DE-001-cloud-storage-exfil.md)

---

## 10. Linked Artifacts

| Artifact Type | ID | File |
|---------------|----|------|
| Validation Test Case | TC-DE-001 | [validation/test-cases/TC-DE-001.md](../../validation/test-cases/TC-DE-001.md) |
| Detection (Sigma) | DET-DE-001-SIGMA | [detections/sigma/DE-001-cloud-storage-upload.yml](../../detections/sigma/DE-001-cloud-storage-upload.yml) |
| Detection (KQL) | DET-DE-001-KQL | [detections/queries/DE-001-kql.md](../../detections/queries/DE-001-kql.md) |
| Hunt Hypothesis | HYP-DE-001 | [hunting/hypotheses/HYP-DE-001.md](../../hunting/hypotheses/HYP-DE-001.md) |
| Response Playbook | PB-DE-001 | [playbooks/PB-DE-001-cloud-storage-exfil.md](../../playbooks/PB-DE-001-cloud-storage-exfil.md) |

---

## 11. Related Scenarios

| Scenario ID | Name | Relationship |
|-------------|------|-------------|
| DE-002 | Email Auto-Forwarding Rule | Alternative exfiltration channel; often used by same actor archetype |
| PM-001 | Privilege Escalation via Access Abuse | Frequently precedes DE-001 — insider escalates access to reach more valuable data |
| UA-001 | After-Hours Unauthorized System Access | Often co-occurs with DE-001 — staging and upload may occur after hours |

---

## 12. References

- CERT Insider Threat Center — Insider Threat Spotlight: Data Exfiltration
- MITRE ATT&CK — T1567.002: Exfiltration to Cloud Storage
- CISA — Identifying and Mitigating Insider Threats
- Verizon DBIR — Insider Threat Patterns

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-25 | SIDTVF Core Team | Initial creation |
