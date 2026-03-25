# Threat Hunting Hypothesis: HYP-DE-001 — Cloud Storage Upload Exfiltration

## Hypothesis Card

| Field | Value |
|-------|-------|
| **Hypothesis ID** | HYP-DE-001 |
| **Name** | Insider Cloud Storage Upload Exfiltration Hunt |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-25 |
| **Last Hunted** | Never |
| **Author** | SIDTVF Core Team |
| **Linked Scenario** | DE-001 — Cloud Storage Upload Exfiltration |
| **Hunt Cadence** | Quarterly (increase to Monthly for high-risk periods — e.g., fiscal year end, M&A, layoffs) |
| **Data Sources Required** | Web proxy logs (CommonSecurityLog), EDR (DeviceNetworkEvents, DeviceFileEvents), DLP logs, HR departing employee list |
| **MITRE ATT&CK** | Exfiltration — T1567.002: Exfiltration to Cloud Storage |

---

## 1. Hypothesis Statements

**Primary Hypothesis:**
IF an insider is exfiltrating data via personal cloud storage, THEN we would observe HTTPS POST/PUT requests to personal cloud storage domains (Google Drive, Dropbox, Box, MEGA, etc.) from corporate endpoints with outbound byte volumes significantly above the sending user's 90-day historical baseline.

**Secondary Hypothesis:**
IF an insider is staging data before cloud exfiltration, THEN we would observe a spike in file access events (reads and copies) across multiple sensitive directories in the days preceding the upload events, correlating with the same user account and endpoint.

**Tertiary Hypothesis:**
IF existing detections are misconfigured or incomplete, THEN an insider could have completed a cloud upload exfiltration without triggering any alert, and evidence of this activity would still be visible in raw proxy logs that were not queried at the time.

---

## 2. Hunt Scope

| Field | Value |
|-------|-------|
| **Time Window** | Past 90 days (or since the date of the last hunt cycle) |
| **User Population** | All users with access to sensitive data repositories; priority sub-population: users in departing employee watchlist, users on PIP, users who received a negative performance review in the past 6 months |
| **Systems in Scope** | All corporate-managed endpoints; all proxy-enforced internet egress paths |
| **Data Sources** | Web proxy logs (CommonSecurityLog), EDR endpoint logs (DeviceNetworkEvents, DeviceFileEvents), DLP event logs, Active Directory / Entra ID user list, HR departure watchlist |

---

## 3. Hunt Queries

### Query 1 — Identify Users with Above-Baseline Cloud Uploads (Proxy Logs)

**Tests:** Primary hypothesis — is any user uploading more data to personal cloud than their own historical baseline suggests?

This query computes each user's average daily upload volume to personal cloud storage over the lookback window, identifies the statistical mean and standard deviation, and surfaces users whose recent upload volume exceeds their baseline by more than 2 standard deviations.

```kql
// HYP-DE-001 — Query 1: Above-Baseline Cloud Upload Detection
// Requires: CommonSecurityLog (proxy logs in Sentinel)
// Looks back 90 days and computes per-user upload baseline vs. recent activity

let PersonalCloudDomains = dynamic([
    "drive.google.com",
    "dropbox.com",
    "api.dropboxapi.com",
    "content.dropboxapi.com",
    "onedrive.live.com",
    "api.onedrive.com",
    "box.com",
    "upload.box.com",
    "mega.nz",
    "icloud.com",
    "wetransfer.com",
    "gofile.io"
]);

let LookbackDays = 90d;
let RecentWindow = 7d;  // "Recent" = last 7 days; "Historical" = 90 days minus last 7

// Step 1: Build historical baseline (90 days, excluding most recent 7 days)
let HistoricalBaseline = CommonSecurityLog
| where TimeGenerated between (ago(LookbackDays) .. ago(RecentWindow))
| where RequestMethod in ("POST", "PUT")
| where DestinationHostName has_any (PersonalCloudDomains)
    or RequestURL has_any (PersonalCloudDomains)
| where SourceUserName !in~ ("svc-backup", "svc-monitoring")
| summarize DailyBytes = sum(SentBytes) by SourceUserName, DayKey = format_datetime(TimeGenerated, "yyyy-MM-dd")
| summarize
    HistMeanDailyBytes = avg(DailyBytes),
    HistStdDevDailyBytes = stdev(DailyBytes),
    HistDaysWithUploads = count()
    by SourceUserName;

// Step 2: Compute recent activity (last 7 days)
let RecentActivity = CommonSecurityLog
| where TimeGenerated >= ago(RecentWindow)
| where RequestMethod in ("POST", "PUT")
| where DestinationHostName has_any (PersonalCloudDomains)
    or RequestURL has_any (PersonalCloudDomains)
| where SourceUserName !in~ ("svc-backup", "svc-monitoring")
| summarize
    RecentTotalBytes = sum(SentBytes),
    RecentUploadCount = count(),
    TargetDomains = make_set(DestinationHostName)
    by SourceUserName;

// Step 3: Join and compute Z-score
RecentActivity
| join kind=leftouter HistoricalBaseline on SourceUserName
| extend
    RecentDailyAvg = RecentTotalBytes / 7.0,
    ZScore = iff(HistStdDevDailyBytes > 0,
        (RecentTotalBytes / 7.0 - HistMeanDailyBytes) / HistStdDevDailyBytes,
        0.0)
| where RecentTotalBytes > 0  // Only show users with recent uploads
| project
    User = SourceUserName,
    RecentTotalMB = round(RecentTotalBytes / 1048576.0, 2),
    RecentUploadCount,
    HistoricalAvgDailyMB = round(HistMeanDailyBytes / 1048576.0, 2),
    DeviationScore = round(ZScore, 2),
    TargetDomains
| extend Priority = case(
    ZScore > 3.0, "HIGH — Significant deviation from baseline",
    ZScore > 2.0, "MEDIUM — Moderate deviation from baseline",
    RecentTotalBytes > 52428800 and ZScore <= 0, "MEDIUM — Large volume, no prior history",
    "LOW — Minor deviation"
)
| sort by RecentTotalMB desc
```

---

### Query 2 — Departing Employee Cloud Upload Summary

**Tests:** Primary hypothesis with HR correlation — did any departing employee upload data to personal cloud in the past 90 days?

```kql
// HYP-DE-001 — Query 2: Departing Employee Cloud Upload Hunt
// Requires: CommonSecurityLog + Sentinel Watchlist "DepartingEmployees"
// This query is the highest-priority hunt target — any positive finding is significant

let PersonalCloudDomains = dynamic([
    "drive.google.com",
    "dropbox.com",
    "onedrive.live.com",
    "box.com",
    "mega.nz",
    "icloud.com",
    "wetransfer.com"
]);

let DepartingEmployees = _GetWatchlist("DepartingEmployees")
    | project UserPrincipalName, DepartureDate, RiskLevel;

CommonSecurityLog
| where TimeGenerated >= ago(90d)
| where RequestMethod in ("POST", "PUT")
| where DestinationHostName has_any (PersonalCloudDomains)
| join kind=inner DepartingEmployees
    on $left.SourceUserName == $right.UserPrincipalName
| summarize
    TotalBytesSent = sum(SentBytes),
    UploadCount = count(),
    EarliestUpload = min(TimeGenerated),
    LatestUpload = max(TimeGenerated),
    TargetDomains = make_set(DestinationHostName)
    by SourceUserName, DepartureDate, RiskLevel
| project
    User = SourceUserName,
    DepartureDate,
    VendorRiskLevel = RiskLevel,
    TotalMBUploaded = round(TotalBytesSent / 1048576.0, 2),
    UploadCount,
    TargetDomains,
    EarliestUpload,
    LatestUpload
| extend HuntFinding = "ESCALATE — Departing employee cloud upload detected"
| sort by TotalMBUploaded desc
```

---

### Query 3 — Pre-Upload File Access Staging Correlation (Endpoint)

**Tests:** Secondary hypothesis — did users with cloud upload activity also access an unusually high volume of files in the days before uploading?

```kql
// HYP-DE-001 — Query 3: File Access Staging Correlation
// Requires: DeviceFileEvents (MDE) + results from Query 1 to identify users of interest
// Identifies file access spikes in the week preceding a cloud upload event

// First, identify users of interest from Query 1 results
// Substitute actual user list from Query 1 output
let UsersOfInterest = dynamic([
    "user1@company.com",
    "user2@company.com"
    // Replace with users surfaced by Query 1 and Query 2
]);

let StagingWindow = 7d;       // Days before the upload to look for staging activity

DeviceFileEvents
| where TimeGenerated >= ago(30d + StagingWindow)
| where InitiatingProcessAccountName has_any (UsersOfInterest)
| where ActionType in ("FileRead", "FileCopied", "FileCreated")
// Focus on sensitive paths — adjust for your environment
| where FolderPath has_any (
    "\\Finance\\",
    "\\Legal\\",
    "\\HR\\",
    "\\Confidential\\",
    "\\Source Code\\",
    "\\Contracts\\",
    "\\Customer Data\\"
    // Add your organization's sensitive directory names
)
| summarize
    FilesAccessed = count(),
    UniqueFiles = dcount(FileName),
    UniqueDirectories = dcount(FolderPath),
    FirstAccess = min(TimeGenerated),
    LastAccess = max(TimeGenerated)
    by DeviceName, InitiatingProcessAccountName, bin(TimeGenerated, 1d)
| extend DayOfAccess = format_datetime(TimeGenerated, "yyyy-MM-dd")
| order by InitiatingProcessAccountName asc, TimeGenerated asc
```

---

### Query 4 — Personal Cloud Domain Access by First-Time Users

**Tests:** Primary hypothesis — did any user access a personal cloud storage domain for the first time in the past 30 days, suggesting a new exfiltration channel was opened?

```kql
// HYP-DE-001 — Query 4: First-Time Personal Cloud Domain Access
// Identifies users who accessed personal cloud storage for the first time in the last 30 days
// High signal for deliberate exfiltration (less likely to be accidental for first-time use)

let PersonalCloudDomains = dynamic([
    "drive.google.com",
    "dropbox.com",
    "onedrive.live.com",
    "box.com",
    "mega.nz",
    "icloud.com",
    "wetransfer.com"
]);

let RecentWindow = 30d;
let HistoricalWindow = 365d;

// Users who accessed personal cloud recently
let RecentCloudUsers = CommonSecurityLog
| where TimeGenerated >= ago(RecentWindow)
| where DestinationHostName has_any (PersonalCloudDomains)
| summarize by SourceUserName;

// Users who had prior cloud access in the historical window (not new users)
let PriorCloudUsers = CommonSecurityLog
| where TimeGenerated between (ago(HistoricalWindow) .. ago(RecentWindow))
| where DestinationHostName has_any (PersonalCloudDomains)
| summarize by SourceUserName;

// First-time users = recent access with NO prior history
RecentCloudUsers
| where SourceUserName !in (PriorCloudUsers)
| join kind=inner (
    CommonSecurityLog
    | where TimeGenerated >= ago(RecentWindow)
    | where DestinationHostName has_any (PersonalCloudDomains)
    | summarize
        TotalBytes = sum(SentBytes),
        UploadCount = count(),
        TargetDomains = make_set(DestinationHostName),
        FirstAccess = min(TimeGenerated)
        by SourceUserName
) on SourceUserName
| project
    User = SourceUserName,
    TotalMBAccessed = round(TotalBytes / 1048576.0, 2),
    UploadCount,
    TargetDomains,
    FirstAccessToPersonalCloud = FirstAccess
| extend HuntNote = "First-time personal cloud storage access — verify business justification"
| sort by TotalMBAccessed desc
```

---

## 4. Indicators to Look For

| Indicator | Significance | Confidence |
|-----------|-------------|-----------|
| User's recent upload volume > 2 SD above their 90-day historical baseline | Strong signal of anomalous behavior | High |
| Any upload to personal cloud by a departing employee | Very strong signal — any volume warrants investigation | High |
| File access spike in sensitive directories in the 7 days before upload events | Confirms staging behavior preceding exfiltration | High |
| First-time access to personal cloud storage from a corporate device | May indicate new exfiltration channel being established | Medium |
| Multiple cloud storage domains accessed by the same user in a short window | May indicate the user is testing which channels are unblocked | Medium |
| Upload activity occurring after 9 PM or before 7 AM local time | Combined with above signals, suggests deliberate evasion | Medium |

---

## 5. Expected vs. Anomalous Results

**Normal (expected) results:**
- Occasional small uploads (<5 MB) to personal cloud domains (background browsing, personal use)
- Consistent with the user's established personal cloud use pattern
- No correlation with file access spikes or departing employee status

**Anomalous results (investigate these):**
- Upload volume > 50 MB to personal cloud by any user in the hunt window
- Upload volume significantly above the user's historical baseline (Z-score > 2)
- Any upload by a departing employee, regardless of volume
- First-time personal cloud upload from a user who has never done so previously
- File access spikes preceding upload events by 1–7 days
- Upload to personal cloud domain using a browser's private/incognito mode (check user-agent)

---

## 6. Hunt Findings Log

### Hunt Cycle: 2026-03-25

| Field | Value |
|-------|-------|
| **Hunter** | SIDTVF Core Team |
| **Date Executed** | 2026-03-25 |
| **Time Period Covered** | Not yet executed — placeholder |
| **Finding Classification** | Pending |
| **Summary** | Initial hypothesis documentation — hunt not yet executed |
| **Actions Taken** | None — this is the initial hypothesis creation |

> Replace this entry with actual hunt results each time this hypothesis is executed.

---

## 7. Escalation Criteria

**Escalate to IR immediately if:**
- Any departing employee has uploaded > 10 MB to personal cloud in the past 90 days
- Any user has a Z-score > 3.0 (more than 3 standard deviations above their baseline)
- Any user has uploaded > 500 MB to personal cloud in the hunt window, regardless of baseline deviation
- A file access spike + upload event correlation is confirmed for the same user

**Route to SOC for triage if:**
- Upload volume > 50 MB but < 500 MB with no HR flag and Z-score between 1.5 and 3.0
- First-time personal cloud access with upload volume 10–50 MB

**Close as clean if:**
- All users in scope show upload patterns consistent with their historical baseline
- No departing employees in the watchlist have cloud upload events
- Upload volumes are small (< 10 MB) with no other correlated signals

---

## 8. Linked Artifacts

| Artifact | ID | File |
|----------|----|------|
| Scenario | DE-001 | [scenarios/data-exfiltration/DE-001-cloud-storage-upload.md](../../scenarios/data-exfiltration/DE-001-cloud-storage-upload.md) |
| Detection (Sigma) | DET-DE-001-SIGMA | [detections/sigma/DE-001-cloud-storage-upload.yml](../../detections/sigma/DE-001-cloud-storage-upload.yml) |
| Detection (KQL) | DET-DE-001-KQL | [detections/queries/DE-001-kql.md](../../detections/queries/DE-001-kql.md) |
| Playbook | PB-DE-001 | [playbooks/examples/PB-DE-001-cloud-storage-exfil.md](../../playbooks/examples/PB-DE-001-cloud-storage-exfil.md) |
| Validation | TC-DE-001 | [validation/test-cases/TC-DE-001.md](../../validation/test-cases/TC-DE-001.md) |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-25 | SIDTVF Core Team | Initial hypothesis creation |
