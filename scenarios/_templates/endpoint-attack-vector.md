# Scenario Template: Endpoint Attack Vector

> Pre-populated for scenarios where the primary threat behavior occurs on a managed endpoint —
> including local file access, USB transfers, process execution, or local credential abuse.
> Copy to the appropriate category directory, rename, and fill in `[PLACEHOLDER]` fields.

---

## Scenario Card

| Field | Value |
|-------|-------|
| **Scenario ID** | [e.g., DE-006] |
| **Name** | [Short name] |
| **Version** | 1.0 |
| **Status** | Draft |
| **Created** | [YYYY-MM-DD] |
| **Last Updated** | [YYYY-MM-DD] |
| **Author** | [Name] |
| **Category** | [Category] |
| **Actor Archetype(s)** | [MAL / NEG / COM / TP] |
| **Severity** | [Critical / High / Medium / Low] |
| **Minimum Maturity Level** | 2 |
| **Attack Vector** | Endpoint |
| **Endpoint Platform** | [Windows / macOS / Linux / Mixed] |

---

## 1. Description

[Describe the endpoint-based threat behavior: what actions the insider takes locally on a managed device, what data is targeted or what system action is taken, and how the behavior evades standard controls.]

---

## 2. Narrative Case Study

[3–6 paragraphs. Include what the insider does on the endpoint, what native OS tools or legitimate applications they use, and why this was difficult to detect without endpoint telemetry.]

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | [MAL / NEG / COM / TP] |
| **Typical Role** | [Any role with access to sensitive local files or with IT/admin rights] |
| **Access Level** | [Standard User / Power User / Local Admin] |
| **Technical Sophistication** | [Low to Medium — most endpoint attacks use native OS tools] |
| **Motivation** | [Financial gain / IP theft / sabotage / negligent behavior] |
| **Endpoint Knowledge Used** | [e.g., "Uses native Windows commands to avoid triggering AV"; "Knows DLP is not configured to scan local print queues"] |

---

## 4. Attack Phases

### Phase 1 — Local Data Discovery
**Description:** Insider locates target data on the local machine or mapped network drives.
**Observable Actions:**
- File system browsing events outside user's normal directory scope (Windows Event 4663)
- Search queries targeting specific file extensions (.docx, .pdf, .xlsx, source code extensions)
- Access to network-mapped drives or UNC paths outside normal scope
- Bulk file open/read events in a short time window

### Phase 2 — Local Staging
**Description:** Insider aggregates target files into a staging directory or archive.
**Observable Actions:**
- New directory created in temp or user profile location containing copies of sensitive files
- Archive utility execution (7z.exe, WinRAR, tar) with a collection of files as input
- File copy operations (xcopy, robocopy, cp) targeting sensitive directories
- Large volume of FileCopied events in EDR within a short window

### Phase 3 — [Exfiltration or Local Action]
**Description:** [How the staged data is exfiltrated or how the local action (sabotage, fraud) is taken]
**Observable Actions:**
- [USB write event / removable media connection]
- [Local print job / screenshot capture]
- [Process execution for local destruction]

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs — Endpoint-Specific

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Bulk file read/copy events across multiple sensitive directories within 30 minutes | EDR (DeviceFileEvents, Windows Event 4663) | High |
| Archive utility (7-Zip, WinRAR, tar) executed against a collection of files | EDR (process creation events) | Medium |
| Removable USB storage device connected and written to on a managed endpoint | EDR / MDM / DLP endpoint agent | High |
| Print job submitted for a large number of documents outside business hours | Print server logs / endpoint DLP | Medium |
| Local file deletion following a bulk copy or archive operation | EDR (FileDeleted events) | Medium |
| New process execution (cmd.exe, PowerShell) accessing sensitive file paths | EDR (process and file event correlation) | Medium |
| Screenshot capture tool or screen recording software executed | EDR (process creation — Snipping Tool, ShareX, OBS) | Low–Medium |
| Access to files labeled as confidential or restricted (DLP classification aware) | Endpoint DLP | High |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Departing employee with local access to sensitive repositories | HR + IAM | High-risk combination |
| Endpoint enrolled in MDM but DLP agent is disabled or uninstalled | MDM / EDR | Control gap — investigate immediately |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Confidence |
|-------|-----------------------|---------------------|-----------|
| Phase 1 | Bulk file access volume spike vs. personal baseline | EDR file events + UEBA | High |
| Phase 1 | Access to sensitive directories outside user's historical scope | EDR + file path classification | Medium |
| Phase 2 | Archive utility executed against sensitive file paths | EDR process + file event correlation | Medium |
| Phase 3 | USB write event on managed endpoint | EDR / MDM / DLP | High |
| Phase 3 | [Scenario-specific detection] | | |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Collection | T1005 | Data from Local System | Phase 1 |
| Collection | T1560.001 | Archive Collected Data: Archive via Utility | Phase 2 |
| Exfiltration | T1052.001 | Exfiltration Over Physical Medium: Exfiltration over USB | Phase 3 (USB variant) |
| Defense Evasion | T1070.004 | Indicator Removal: File Deletion | Phase 3 (cover) |
| [Add additional techniques] | | | |

---

## 8. Legal and Privacy Considerations

- **Endpoint monitoring scope:** Ensure the acceptable use policy explicitly authorizes monitoring of endpoint file system activity, process execution, and removable media use.
- **Personal files on corporate devices:** In some jurisdictions, employees retain privacy rights over personal files stored on corporate devices. DLP and EDR monitoring should be scoped to business data categories.
- **USB controls:** If USB ports are blocked by policy, document this as a preventive control. If they are not blocked, document why (business need) and ensure monitoring compensates.
- **[Add scenario-specific considerations]**

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Preserve EDR telemetry for the affected endpoint (90-day window) | IR |
| 2 | Forensically image the endpoint before any remediation | IR / Forensics |
| 3 | Identify all files accessed, copied, or staged | IR |
| 4 | Notify Legal — determine scope and breach exposure | Legal |
| 5 | Revoke endpoint access and disable account (after Legal approval) | IAM / IT |

**Linked Playbook:** [PB-XX-NNN — create using `playbooks/_template.md`]

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | [YYYY-MM-DD] | [Author] | Initial creation |
