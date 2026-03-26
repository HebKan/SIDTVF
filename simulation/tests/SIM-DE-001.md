# Simulation Test: SIM-DE-001

| Field | Value |
|-------|-------|
| **Simulation ID** | SIM-DE-001 |
| **Linked Scenario** | DE-001 — Cloud Storage Upload Exfiltration |
| **Linked Validation** | TC-DE-001 |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-26 |
| **Last Updated** | 2026-03-26 |
| **Estimated Duration** | 45 minutes + 1h observation window |
| **Required Environment** | Test or Staging — requires web proxy logging and/or endpoint agent |
| **Minimum Maturity Level** | 2 |

---

## Objective

Reproduce the behavioral pattern of a malicious insider staging files for exfiltration and uploading them to a personal cloud storage account. The simulation should produce proxy logs, endpoint file event logs, and — if configured — UEBA signals sufficient to trigger the DE-001 detection rules.

---

## Authorization

> Complete the [Authorization Checklist](../authorization-checklist.md) before proceeding. This test generates real network traffic to external cloud storage providers. Ensure that outbound access to the target domains is permitted from the test environment and that the SOC is notified.

---

## Prerequisites

### Environment
- [ ] Web proxy is active and logging to SIEM (CommonSecurityLog or equivalent)
- [ ] Endpoint agent (EDR) is deployed on the test workstation and logging to SIEM
- [ ] At least one of these detection rules is deployed: `DET-DE-001-SIGMA`, `DET-DE-001-KQL`, or `DET-DE-001-SPL`
- [ ] Test user account `sim.testuser@yourdomain.com` exists with standard employee permissions
- [ ] SOC has been notified of simulation window: `[start time]` to `[end time]`

### Test Data
- [ ] Synthetic dataset created — see Synthetic Data Setup below
- [ ] Test files contain no real sensitive data — only synthetic content

### Tools Required
- Web browser (Chrome or Edge) — for uploading to test cloud storage account
- File manager / Explorer — for staging files
- Test personal cloud storage account — create a dedicated `sim-test-xxxx@gmail.com` account for this purpose; never use a real personal account

---

## Synthetic Data Setup

Create the following file structure on the test workstation Desktop:

```
Desktop/
└── STAGING_TEST/
    ├── financials_Q4_FAKE.xlsx     (~15 MB — create with large random data)
    ├── customer_export_FAKE.csv    (~20 MB — 100k rows of fake names/emails)
    ├── source_backup_FAKE.zip      (~30 MB — zip of publicly available open source code)
    └── README_SYNTHETIC.txt        (note: "THIS IS SYNTHETIC TEST DATA FOR SIM-DE-001")
```

**Python snippet to generate synthetic CSV (65 MB total):**
```python
import csv, random, string

def rand_str(n): return ''.join(random.choices(string.ascii_letters, k=n))

with open('customer_export_FAKE.csv', 'w', newline='') as f:
    w = csv.writer(f)
    w.writerow(['id','name','email','account_num'])
    for i in range(500000):
        w.writerow([i, rand_str(12), f"{rand_str(8)}@fake-test.example", rand_str(16)])

print("Done — check file size is >60 MB")
```

**Total target size: >60 MB** — enough to exceed the DE-001 detection threshold (50 MB/hour).

---

## Simulation Steps

### Phase 1 — File Staging (Pre-upload Behavior)

This phase generates endpoint file event logs matching the "bulk file read" IOB pattern.

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 1.1 | Log in as `sim.testuser` on the test workstation | Endpoint agent / AD | Authentication event |
| 1.2 | Open File Explorer; navigate to `C:\Users\sim.testuser\Documents` | Endpoint agent | DeviceFileEvents: ActionType=FileRead |
| 1.3 | Select all files in the Documents folder (Ctrl+A) and copy to `Desktop\STAGING_TEST\` | Endpoint agent | Multiple FileRead/FileCopied events in < 2 minutes |
| 1.4 | Right-click `STAGING_TEST` folder → "Send to" → "Compressed (zipped) folder" | Endpoint agent | DeviceProcessEvents: process=explorer.exe, FileCreated: *.zip |

**Expected artifact:** 20+ file read events in under 3 minutes for `sim.testuser` — matches IOB-T-002 (Bulk file copy from network drive or local storage).

### Phase 2 — Personal Cloud Account Access

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 2.1 | Open Chrome browser; navigate to `drive.google.com` | Web proxy / DNS | DNS query for drive.google.com; HTTP GET to drive.google.com |
| 2.2 | Sign in with the dedicated simulation Google account (`sim-test-xxxx@gmail.com`) | Web proxy | HTTP POST to accounts.google.com (auth) |
| 2.3 | Note: do NOT sign in with a corporate Google Workspace account — must be a personal account | — | — |

**Expected artifact:** Authentication event to drive.google.com from corporate network, with personal (non-corporate) account.

### Phase 3 — Large File Upload

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 3.1 | On Google Drive, click "New" → "File upload" | Web proxy | HTTP POST to upload.googleapis.com |
| 3.2 | Select `customer_export_FAKE.csv` (~20 MB) and click Open | Web proxy | bytes_out > 20,971,520 for upload.googleapis.com |
| 3.3 | Wait for upload to complete | Web proxy | HTTP 200 response |
| 3.4 | Repeat: upload `financials_Q4_FAKE.xlsx` (~15 MB) | Web proxy | bytes_out > 15,728,640 for upload.googleapis.com |
| 3.5 | Repeat: upload `source_backup_FAKE.zip` (~30 MB) | Web proxy | bytes_out > 31,457,280 for upload.googleapis.com |

**Expected artifact:** Total bytes_out to `*.googleapis.com` or `drive.google.com` exceeds 52,428,800 (50 MB) within a 1-hour window. This is the primary threshold trigger for DE-001 detections.

### Phase 4 — (Optional) Departure Correlation Test

This step tests the departure-correlated variant of the detection (Query 4 in DE-001-kql.md and DE-001-spl.md).

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 4.1 | Before running Phases 1–3: add `sim.testuser@yourdomain.com` to the `DepartingEmployees` watchlist in Sentinel (or the equivalent lookup in Splunk) with a `last_day` 7 days from today | Sentinel watchlist | Watchlist updated |
| 4.2 | Execute Phases 1–3 as normal | — | — |
| 4.3 | After observation window: verify that the departure-correlated alert fired with `risk_score = CRITICAL` (within 7 days of last_day) | SIEM | Alert with risk_score field populated |
| 4.4 | Remove `sim.testuser` from the watchlist after the test | Sentinel watchlist | Watchlist entry removed |

---

## Expected Log Artifacts

| Log Source | Event Type | Key Field | Expected Value |
|------------|------------|-----------|----------------|
| Web proxy (CommonSecurityLog) | HTTP POST | DestinationHostName | `upload.googleapis.com` or `drive.google.com` |
| Web proxy | HTTP POST | SentBytes | > 52,428,800 |
| Endpoint (DeviceFileEvents) | FileCopied / FileRead | InitiatingProcessAccountName | `sim.testuser` |
| Endpoint (DeviceFileEvents) | FileCreated | FileName | `*.zip` |
| Endpoint (DeviceNetworkEvents) | Connection | RemoteUrl | `*googleapis.com` |

---

## Expected Detection Output

| Detection Rule | Expected Trigger | Time to Alert |
|---------------|-----------------|---------------|
| DET-DE-001-SIGMA | POST requests to cloud storage domain with bytes > 50 MB in 1h | < 10 minutes |
| DET-DE-001-KQL Query 1 | Proxy volume threshold exceeded | < 10 minutes |
| DET-DE-001-SPL Query 1 | `total_bytes_out > 52428800` in 1h window | < 10 minutes |
| DET-DE-001-KQL/SPL Query 4 | Departing employee upload (if Phase 4 executed) | < 10 minutes |

---

## Observation Window

Wait up to **60 minutes** after completing Phase 3 before declaring PASS or FAIL. Real-time proxy-based rules should fire within 10 minutes. Endpoint-based rules may take up to 15 minutes depending on event ingestion latency.

---

## Pass / Fail Criteria

| Result | Criteria |
|--------|----------|
| **PASS** | At least one DE-001 detection alert fires within 60 minutes, containing: correct username, source IP, destination domain, and bytes_out value |
| **PARTIAL** | Alert fires but is missing required fields, OR only one of multiple expected detections fires |
| **FAIL** | No alert fires within 60 minutes |

---

## Detection Blind Spots Validated by This Test

This simulation does **not** test the following known blind spots. Document these separately:

- **Incognito/Private browser**: Some proxy configurations log Incognito traffic identically to normal traffic. Others do not. Test separately if your proxy differentiates.
- **Mobile device on personal network**: Entirely out of scope for this test. Requires MDM/CASB mobile agent.
- **Slow exfiltration (< 5 MB/day)**: The 50 MB threshold will never fire. This is a known gap.
- **Encrypted tunnel / VPN bypass**: If the user routes through a personal VPN before uploading, proxy logs will not capture destination.

---

## Cleanup Steps

| Step | Action | Verified By | Date |
|------|--------|-------------|------|
| C1 | Delete all files from `Desktop\STAGING_TEST\` on the test workstation | | |
| C2 | Empty the test Google Drive account and sign out of the browser | | |
| C3 | Delete the synthetic test data files created in Step 0 | | |
| C4 | Remove `sim.testuser` from the DepartingEmployees watchlist (if Phase 4 was run) | | |
| C5 | Mark any generated SIEM alerts as "SIM-DE-001 — simulation test" in the ticketing system | | |
| C6 | Notify SOC that simulation window is closed | | |

---

## Run Log

| Run Date | Analyst | Environment | Result | Alerts Fired | Notes |
|----------|---------|-------------|--------|-------------|-------|
| | | | | | |

---

## Atomic Red Team Reference

| Technique | Atomic Test # | Notes |
|-----------|--------------|-------|
| T1567.002 | T1567.002-1 (if available) | ART test validates technique execution; SIM-DE-001 validates the full behavioral pattern including staging phase and departure correlation |
| T1560.001 | T1560.001-1 | ART tests archive creation; SIM-DE-001 validates that archive creation is correlated with the subsequent upload |
| T1005 | T1005-1 | ART tests file collection from local system |

---

## Linked Artifacts

| Type | ID | File |
|------|----|------|
| Scenario | DE-001 | scenarios/data-exfiltration/DE-001-cloud-storage-upload.md |
| Validation Test Case | TC-DE-001 | validation/test-cases/TC-DE-001.md |
| Detection (Sigma) | DET-DE-001-SIGMA | detections/sigma/DE-001-cloud-storage-upload.yml |
| Detection (KQL) | DET-DE-001-KQL | detections/queries/DE-001-kql.md |
| Detection (SPL) | DET-DE-001-SPL | detections/queries/DE-001-spl.md |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-26 | SIDTVF Core Team | Initial creation |
