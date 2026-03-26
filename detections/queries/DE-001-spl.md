# Detection Query: DET-DE-001-SPL

| Field | Value |
|-------|-------|
| **Detection ID** | DET-DE-001-SPL |
| **Linked Scenario** | DE-001 — Cloud Storage Upload Exfiltration |
| **Platform** | Splunk (SPL) |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-26 |
| **Last Updated** | 2026-03-26 |
| **Author** | SIDTVF Core Team |

**Data Source Requirements:**
- Splunk CIM-compliant web proxy data in the `Web` datamodel (Bluecoat, Zscaler, Squid, or equivalent)
- OR Splunk CIM-compliant endpoint data in the `Endpoint` datamodel (CrowdStrike, Carbon Black, Defender for Endpoint via Splunk add-on)
- HR departure watchlist as a Splunk lookup table (`departing_employees.csv`) — required for Query 4 only

---

## Query 1 — Proxy-Based Volume Threshold Detection

Detects large outbound uploads to personal cloud storage domains by a single user, measured by bytes sent through the web proxy. Uses a 50 MB threshold as the default; tune to your environment using the 90th percentile of legitimate upload sizes.

```spl
| tstats summariesonly=t
    sum(Web.bytes_out) as total_bytes_out
    count as request_count
    dc(Web.url) as distinct_urls
    values(Web.url) as sample_urls
  from datamodel=Web.Web
  where
    (Web.dest_host="*.icloud.com" OR Web.dest_host="*.dropbox.com"
      OR Web.dest_host="*.box.com" OR Web.dest_host="*.onedrive.live.com"
      OR Web.dest_host="*.googledrive.com" OR Web.dest_host="drive.google.com"
      OR Web.dest_host="*.wetransfer.com" OR Web.dest_host="*.mediafire.com"
      OR Web.dest_host="*.sendspace.com" OR Web.dest_host="*.mega.nz"
      OR Web.dest_host="*.4shared.com" OR Web.dest_host="*.zippyshare.com"
      OR Web.dest_host="*.anonfiles.com")
    AND (Web.http_method="POST" OR Web.http_method="PUT")
  by Web.src_user Web.dest_host _time span=1h
| rename Web.src_user as user Web.dest_host as cloud_domain
| where total_bytes_out > 52428800
| eval total_mb = round(total_bytes_out / 1048576, 1)
| eval alert_reason = "Upload volume " . total_mb . " MB to " . cloud_domain . " in 1h window"
| table _time user cloud_domain total_mb request_count distinct_urls sample_urls alert_reason
| sort -total_mb
```

**Tuning Notes:**
- Default threshold: 52,428,800 bytes (50 MB). Run baseline period of 30–90 days before setting final threshold.
- To find the 95th percentile baseline, run: `| tstats ... | stats p95(total_bytes_out) as p95_bytes`
- Add legitimate corporate cloud sync domains (e.g., `sharepoint.com`, `onedrive.com` with corporate tenant ID) to an exclusion lookup if needed.
- In Zscaler environments, the `dest_host` field may be `url_domain`; adjust field names accordingly.

---

## Query 2 — Endpoint-Based Detection (Process + Network Correlation)

Detects personal cloud sync clients (Dropbox, Google Drive, iCloud, OneDrive personal) running on endpoints and initiating large outbound network connections. Requires Splunk add-on for CrowdStrike, Carbon Black, or Microsoft Defender for Endpoint.

```spl
| tstats summariesonly=t
    sum(Endpoint.Network_Traffic.bytes_out) as total_bytes_out
    values(Endpoint.Network_Traffic.dest_host) as dest_hosts
    values(Endpoint.Processes.process_name) as processes
  from datamodel=Endpoint.Network_Traffic
  where
    Endpoint.Network_Traffic.dest_host IN (
      "*.dropbox.com", "*.icloud.com", "drive.google.com",
      "*.googledrive.com", "content.dropboxapi.com", "api.dropboxapi.com",
      "*.onedrive.live.com", "storage.live.com"
    )
  by Endpoint.Network_Traffic.src_user Endpoint.Network_Traffic.src_host _time span=1h
| rename
    Endpoint.Network_Traffic.src_user as user
    Endpoint.Network_Traffic.src_host as endpoint
| join type=left user endpoint [
    | tstats summariesonly=t
        values(Endpoint.Processes.process_name) as sync_processes
      from datamodel=Endpoint.Processes
      where
        Endpoint.Processes.process_name IN (
          "Dropbox.exe", "googledrivesync.exe", "GoogleDriveFS.exe",
          "OneDrive.exe", "iCloud.exe", "iCloudDrive.exe"
        )
      by Endpoint.Processes.user Endpoint.Processes.dest
    | rename Endpoint.Processes.user as user Endpoint.Processes.dest as endpoint
    ]
| where isnotnull(sync_processes)
| where total_bytes_out > 52428800
| eval total_mb = round(total_bytes_out / 1048576, 1)
| eval alert_type = "Personal cloud sync client upload: " . sync_processes . " — " . total_mb . " MB"
| table _time user endpoint sync_processes dest_hosts total_mb alert_type
| sort -total_mb
```

**Tuning Notes:**
- Extend the `process_name` list as new sync clients emerge.
- False positives may arise from IT-authorized backup tools. Maintain a `sanctioned_backup_processes.csv` lookup to exclude them.
- On macOS endpoints, process names differ (e.g., `bird` for iCloud, `Google Drive File Stream`).

---

## Query 3 — Cumulative Daily Upload Volume (Small Upload Detection)

Detects users who spread uploads across many small requests to stay below single-event thresholds. Aggregates total upload bytes per user per day and alerts when the daily total exceeds 200 MB to any personal cloud domain combination.

```spl
index=proxy
  (dest="*.icloud.com" OR dest="*.dropbox.com" OR dest="*.box.com"
    OR dest="drive.google.com" OR dest="*.googledrive.com"
    OR dest="*.wetransfer.com" OR dest="*.mega.nz"
    OR dest="*.onedrive.live.com" OR dest="storage.live.com")
  (http_method=POST OR http_method=PUT)
| eval upload_date = strftime(_time, "%Y-%m-%d")
| stats
    sum(bytes_out) as daily_bytes_out
    count as daily_request_count
    dc(dest) as distinct_domains
    values(dest) as domains
  by src_user upload_date
| where daily_bytes_out > 209715200
| eval daily_mb = round(daily_bytes_out / 1048576, 1)
| eval alert_reason = "Daily cumulative upload " . daily_mb . " MB across " . distinct_domains . " personal cloud domain(s)"
| table upload_date src_user daily_mb daily_request_count distinct_domains domains alert_reason
| sort -daily_mb
```

**Tuning Notes:**
- Default daily threshold: 209,715,200 bytes (200 MB). Lower to 100 MB for environments with strict data governance.
- This query runs most efficiently as a scheduled search (daily, looking back 24h). Set schedule to run at 00:15 to capture previous day.
- Correlate `distinct_domains > 2` as an additional risk multiplier — spreading uploads across multiple personal cloud services is a stronger signal.

---

## Query 4 — Departure-Correlated Upload Detection

Enriches upload detections with HR departure data to flag users within their last 30 days of employment. Requires a lookup table `departing_employees.csv` with at least the fields `username` and `last_day`.

**Lookup Table Format (`departing_employees.csv`):**
```csv
username,last_day,department,manager
jsmith,2026-04-15,Engineering,a.jones
mwilliams,2026-04-01,Finance,b.kumar
```

```spl
| tstats summariesonly=t
    sum(Web.bytes_out) as total_bytes_out
    count as request_count
    values(Web.dest_host) as cloud_domains
  from datamodel=Web.Web
  where
    (Web.dest_host IN (
      "*.icloud.com", "*.dropbox.com", "*.box.com", "drive.google.com",
      "*.googledrive.com", "*.wetransfer.com", "*.mega.nz",
      "*.onedrive.live.com", "storage.live.com"
    ))
    AND (Web.http_method="POST" OR Web.http_method="PUT")
  by Web.src_user _time span=1d
| rename Web.src_user as username
| lookup departing_employees.csv username OUTPUT last_day department manager
| where isnotnull(last_day)
| eval today = strftime(now(), "%Y-%m-%d")
| eval days_remaining = datediff(strptime(last_day, "%Y-%m-%d"), strptime(today, "%Y-%m-%d"), "day")
| where days_remaining >= 0 AND days_remaining <= 30
| eval total_mb = round(total_bytes_out / 1048576, 1)
| eval risk_score = case(
    days_remaining <= 7, "CRITICAL",
    days_remaining <= 14, "HIGH",
    days_remaining <= 30, "MEDIUM"
  )
| table _time username last_day days_remaining department manager total_mb request_count cloud_domains risk_score
| sort -total_mb
```

**Tuning Notes:**
- If no HR system integration is available, this lookup can be maintained manually by HR or a SOC analyst and uploaded to Splunk weekly.
- Alert on **any** upload activity for users with `days_remaining <= 7` regardless of threshold — the proximity to departure is itself a risk signal.
- Combine with Query 1 or Query 3 for a combined risk score: departing employee + high volume = immediate escalation.
- Consider enabling this query as a saved search with a daily summary alert sent to the DLP team and the departing employee's manager (after Legal review).

---

## Response Actions

When this detection fires:

1. Confirm the upload destination is a personal account (not a corporate-sanctioned service with a different URL structure).
2. Cross-reference with the scenario `DE-001` IOBs — check for staging artifacts on the endpoint (archive files, bulk file reads in the prior 24h).
3. Correlate with HR data: is this user currently employed, recently resigned, or on a performance improvement plan?
4. Escalate to Tier 2 or IR if: volume > 500 MB, user is within 30 days of departure, or multiple cloud domains are used in the same session.
5. Follow `PB-DE-001` for full response guidance.

**Linked Playbook:** [PB-DE-001](../../playbooks/examples/PB-DE-001-cloud-storage-exfil.md)
**Linked Scenario:** [DE-001](../../scenarios/data-exfiltration/DE-001-cloud-storage-upload.md)
**Linked Sigma Rule:** [DET-DE-001-SIGMA](../sigma/DE-001-cloud-storage-upload.yml)
**Linked KQL Queries:** [DET-DE-001-KQL](DE-001-kql.md)

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-26 | SIDTVF Core Team | Initial creation — 4 SPL queries |
