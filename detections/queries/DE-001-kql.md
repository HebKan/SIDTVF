# Detection Queries: DE-001 — Cloud Storage Upload Exfiltration

| Field | Value |
|-------|-------|
| **Detection ID** | DET-DE-001-KQL |
| **Name** | Personal Cloud Storage Upload Exfiltration (KQL) |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-25 |
| **Last Updated** | 2026-03-25 |
| **Author** | SIDTVF Core Team |
| **Linked Scenario** | DE-001 — Cloud Storage Upload Exfiltration |
| **Platform** | KQL (Microsoft Sentinel / Microsoft Defender XDR) |
| **Data Source(s)** | CommonSecurityLog (Proxy), DeviceNetworkEvents, DeviceFileEvents, OfficeActivity |
| **Detection Type** | Threshold + Behavioral |
| **MITRE ATT&CK** | Exfiltration — T1567.002: Exfiltration to Cloud Storage |
| **Severity** | High |
| **Confidence** | High |

---

## Query 1 — Proxy-Based Cloud Upload Detection (CommonSecurityLog)

**Data Source:** CommonSecurityLog (proxy or NGFW sending to Sentinel via Syslog/CEF)

**What it detects:** Large POST/PUT requests to known personal cloud storage domains from internal users, via proxy logs.

**Requirements:** Web proxy must be configured to send logs to Microsoft Sentinel via CEF/Syslog. URL categorization is not required but helpful for reducing false positives.

```kql
// DE-001: Personal Cloud Storage Upload via Proxy Logs
// Detects large data uploads to personal cloud storage services
// Threshold: 50 MB per request — adjust for your environment
// Requires: CommonSecurityLog table (proxy / NGFW via CEF)

let PersonalCloudDomains = dynamic([
    "drive.google.com",
    "dropbox.com",
    "api.dropboxapi.com",
    "content.dropboxapi.com",
    "onedrive.live.com",
    "api.onedrive.com",
    "1drv.ms",
    "box.com",
    "upload.box.com",
    "mega.nz",
    "icloud.com",
    "wetransfer.com",
    "gofile.io"
    // Add additional personal cloud domains as needed
]);

let UploadThresholdBytes = 52428800; // 50 MB — adjust as needed
let LookbackDays = 1d;              // Alert window — adjust as needed

CommonSecurityLog
| where TimeGenerated >= ago(LookbackDays)
// Filter to upload methods only
| where RequestMethod in ("POST", "PUT")
// Match against personal cloud storage domain list
| where DestinationHostName has_any (PersonalCloudDomains)
    or RequestURL has_any (PersonalCloudDomains)
// Filter to large uploads only
| where SentBytes >= UploadThresholdBytes
// Exclude known service accounts or automated processes
// Customize this list for your environment
| where SourceUserName !in~ ("svc-backup", "svc-monitoring", "system")
// Summarize by user and destination
| summarize
    TotalBytesSent = sum(SentBytes),
    UploadCount = count(),
    FirstUpload = min(TimeGenerated),
    LastUpload = max(TimeGenerated),
    TargetDomains = make_set(DestinationHostName),
    SourceIPs = make_set(SourceIP)
    by SourceUserName, bin(TimeGenerated, 1h)
// Raise alert threshold for summarized view
| where TotalBytesSent >= UploadThresholdBytes
| project
    TimeWindow = TimeGenerated,
    User = SourceUserName,
    TotalMBUploaded = round(TotalBytesSent / 1048576.0, 2),
    UploadCount,
    TargetDomains,
    SourceIPs,
    FirstUpload,
    LastUpload
| sort by TotalMBUploaded desc
```

**Expected Alert Fields:**
- `User` — the account performing the upload
- `TotalMBUploaded` — total megabytes uploaded in the time window
- `TargetDomains` — the personal cloud storage domains accessed
- `SourceIPs` — the workstation IP(s) used

---

## Query 2 — Endpoint-Based Cloud Upload Detection (DeviceNetworkEvents)

**Data Source:** DeviceNetworkEvents (Microsoft Defender for Endpoint)

**What it detects:** Network connections from managed endpoints to personal cloud storage domains with large outbound byte counts, without requiring a proxy.

**Requirements:** Microsoft Defender for Endpoint (MDE) deployed on managed endpoints with network telemetry enabled.

```kql
// DE-001: Personal Cloud Storage Upload via Endpoint Network Events
// Detects outbound connections to personal cloud storage from managed endpoints
// Requires: Microsoft Defender for Endpoint — DeviceNetworkEvents

let PersonalCloudDomains = dynamic([
    "drive.google.com",
    "dropbox.com",
    "api.dropboxapi.com",
    "onedrive.live.com",
    "box.com",
    "mega.nz",
    "icloud.com",
    "wetransfer.com"
]);

let UploadThresholdBytes = 10485760; // 10 MB — lower threshold for endpoint telemetry
let LookbackDays = 1d;

DeviceNetworkEvents
| where TimeGenerated >= ago(LookbackDays)
// Outbound connections only
| where ActionType == "ConnectionSuccess"
// Match personal cloud storage domains
| where RemoteUrl has_any (PersonalCloudDomains)
    or RemoteIP in (
        // Optionally add known cloud storage IP ranges
        // This is fragile — domain matching is preferred
    )
// Only include connections with meaningful outbound data
| where SentBytes >= UploadThresholdBytes
// Summarize per device and user
| summarize
    TotalBytesSent = sum(SentBytes),
    ConnectionCount = count(),
    TargetURLs = make_set(RemoteUrl, 10),
    FirstSeen = min(TimeGenerated),
    LastSeen = max(TimeGenerated)
    by DeviceName, InitiatingProcessAccountName, InitiatingProcessFileName, bin(TimeGenerated, 1h)
| project
    TimeWindow = TimeGenerated,
    Device = DeviceName,
    User = InitiatingProcessAccountName,
    ProcessName = InitiatingProcessFileName,
    TotalMBSent = round(TotalBytesSent / 1048576.0, 2),
    ConnectionCount,
    TargetURLs,
    FirstSeen,
    LastSeen
// Flag browser and cloud sync processes separately for triage context
| extend ProcessCategory = case(
    InitiatingProcessFileName in~ ("chrome.exe", "firefox.exe", "msedge.exe", "safari.exe"), "Browser Upload",
    InitiatingProcessFileName in~ ("Dropbox.exe", "GoogleDriveFS.exe", "OneDrive.exe", "Box.exe"), "Sync Client",
    "Other"
)
| sort by TotalMBSent desc
```

---

## Query 3 — Cumulative Daily Upload Volume (High-Volume Exfiltration)

**Data Source:** CommonSecurityLog (proxy)

**What it detects:** Users who upload large aggregate daily volumes to personal cloud storage, even if individual requests are below the per-request threshold. Designed to catch exfiltration via many small uploads.

```kql
// DE-001: Cumulative Daily Cloud Upload Volume per User
// Detects aggregate daily exfiltration across multiple smaller uploads
// Useful for catching methodical exfiltration designed to evade per-request thresholds

let PersonalCloudDomains = dynamic([
    "drive.google.com",
    "dropbox.com",
    "onedrive.live.com",
    "box.com",
    "mega.nz",
    "icloud.com",
    "wetransfer.com"
]);

let DailyThresholdBytes = 104857600; // 100 MB cumulative per day — adjust for your environment
let LookbackDays = 7d;              // Look back 7 days to spot trends

CommonSecurityLog
| where TimeGenerated >= ago(LookbackDays)
| where RequestMethod in ("POST", "PUT")
| where DestinationHostName has_any (PersonalCloudDomains)
    or RequestURL has_any (PersonalCloudDomains)
| where SourceUserName !in~ ("svc-backup", "svc-monitoring")
| summarize
    DailyBytesSent = sum(SentBytes),
    DailyUploadCount = count(),
    TargetDomains = make_set(DestinationHostName)
    by SourceUserName, DayOfUpload = format_datetime(TimeGenerated, "yyyy-MM-dd")
| where DailyBytesSent >= DailyThresholdBytes
| project
    DayOfUpload,
    User = SourceUserName,
    DailyMBUploaded = round(DailyBytesSent / 1048576.0, 2),
    DailyUploadCount,
    TargetDomains
| order by DailyMBUploaded desc
```

---

## Query 4 — Departure-Correlated Upload Detection (High-Priority Variant)

**Data Source:** CommonSecurityLog + HR watchlist

**What it detects:** Any cloud storage upload by an employee who is on a departure watchlist (e.g., has given notice or is flagged by HR). Uses a custom watchlist to correlate HR context with technical signals. Even small uploads become high-priority when the user is a departing employee.

**Requirements:** A Sentinel Watchlist named `DepartingEmployees` with a column `UserPrincipalName` containing UPNs of employees who have given notice or are flagged by HR.

```kql
// DE-001: Cloud Upload by Departing Employee (High-Priority Variant)
// Correlates HR context (departing employees) with any cloud upload activity
// Requires: CommonSecurityLog + Sentinel Watchlist "DepartingEmployees"

let PersonalCloudDomains = dynamic([
    "drive.google.com",
    "dropbox.com",
    "onedrive.live.com",
    "box.com",
    "mega.nz",
    "icloud.com",
    "wetransfer.com"
]);

// Load the departing employee watchlist (maintained by HR / IAM team)
let DepartingEmployees = _GetWatchlist("DepartingEmployees")
    | project UserPrincipalName, DepartureDate, RiskLevel;

CommonSecurityLog
| where TimeGenerated >= ago(30d)  // Look back 30 days for departing employee activity
| where RequestMethod in ("POST", "PUT", "GET")
| where DestinationHostName has_any (PersonalCloudDomains)
    or RequestURL has_any (PersonalCloudDomains)
// Join with departing employee watchlist
| join kind=inner DepartingEmployees
    on $left.SourceUserName == $right.UserPrincipalName
| summarize
    TotalBytesSent = sum(SentBytes),
    UploadCount = count(),
    FirstUpload = min(TimeGenerated),
    LastUpload = max(TimeGenerated),
    TargetDomains = make_set(DestinationHostName)
    by SourceUserName, DepartureDate, RiskLevel
| project
    User = SourceUserName,
    DepartureDate,
    RiskLevel,
    TotalMBSent = round(TotalBytesSent / 1048576.0, 2),
    UploadCount,
    TargetDomains,
    FirstUpload,
    LastUpload
| extend AlertSeverity = "High"  // All cloud uploads by departing employees are high-priority
| sort by TotalMBSent desc
```

---

## Data Source Requirements

| Requirement | Details | Mandatory? |
|-------------|---------|-----------|
| Web proxy logging to Sentinel | CommonSecurityLog via CEF/Syslog from proxy | Yes (Queries 1, 3, 4) |
| Proxy must log cs-bytes/SentBytes | Proxy must capture request body size | Yes |
| URL logging at the proxy | Full URL or at minimum domain-level logging | Yes |
| Microsoft Defender for Endpoint | DeviceNetworkEvents telemetry with SentBytes | Yes (Query 2) |
| Sentinel Watchlist — DepartingEmployees | Maintained by HR/IAM, updated at departure notification | Yes (Query 4 only) |

---

## Tuning Guidance

- **Threshold adjustments:** Start with higher thresholds (100 MB) in noisy environments and tune down as you validate false positive rates. Different departments may have different normal upload patterns.
- **Approved cloud storage exclusions:** If your organization uses Google Workspace with corporate-managed accounts, exclude the organizational Google Drive domain (`drive.google.com/a/yourdomain.com`). Consult your CASB or proxy vendor documentation for the correct exclusion syntax.
- **User-agent context:** Add user-agent parsing to distinguish browser uploads from desktop sync clients. Desktop sync clients (Dropbox.exe, GoogleDriveFS.exe) present a different risk profile and may require separate alert logic.
- **Time-of-day context:** Run a variant of Query 1 filtered to after-hours (before 7 AM or after 8 PM) with a lower threshold. After-hours uploads are a stronger signal.

---

## False Positive Considerations

| False Positive Scenario | How to Differentiate |
|------------------------|---------------------|
| Employee uploads large presentation to personal drive to work from home | Check whether the upload aligns with an active project or HR travel/remote work record |
| Developer pushing large build artifacts to cloud for personal project | Check process name (should be a dev tool, not a browser) and time pattern |
| Marketing team uploads video content to authorized cloud service | Add sanctioned cloud destination exclusions from your CASB or cloud policy |
| Executive uploading board materials to approved secure file transfer service | Maintain an approved file transfer domain exclusion list |

---

## Response Actions

1. Verify the upload is not authorized (check with user's manager)
2. Identify the full data volume and target domain(s)
3. Determine what data categories were uploaded (DLP, file access audit)
4. Preserve proxy and endpoint logs before any account action
5. Escalate to IR if upload volume is > 500 MB or data is sensitive

**Linked Playbook:** [PB-DE-001](../../playbooks/PB-DE-001-cloud-storage-exfil.md)

---

## Validation

**Test Case:** [TC-DE-001](../../validation/test-cases/TC-DE-001.md)

**Expected Result:** Query 1 should generate a result with `TotalMBUploaded` > 50, the user's account name, and the target cloud domain. The result should appear within the configured lookback window after a test upload is executed.

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-25 | SIDTVF Core Team | Initial creation — four KQL queries for Sentinel |
