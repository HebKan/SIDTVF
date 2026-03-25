# Validation Test Case: TC-DE-001 — Cloud Storage Upload Exfiltration

## Test Case Card

| Field | Value |
|-------|-------|
| **Test Case ID** | TC-DE-001 |
| **Name** | Personal Cloud Storage Upload Detection Validation |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-25 |
| **Last Executed** | Never |
| **Author** | SIDTVF Core Team |
| **Linked Scenario** | DE-001 — Cloud Storage Upload Exfiltration |
| **Linked Detection** | DET-DE-001-SIGMA, DET-DE-001-KQL |
| **Validation Type** | Detection Test |
| **Test Environment** | Dedicated test environment or production with documented safeguards (see authorization) |
| **Authorization Required** | Written approval from Security Manager. Legal review required if executing in production environment. |

> **WARNING:** This test case involves uploading data to external cloud storage. Execute only with test (non-sensitive, synthetic) data. Never use real customer data, employee PII, or classified/confidential information in test uploads. All test data must be clearly labeled "SIDTVF TEST DATA — NOT SENSITIVE."

---

## 1. Objective

Validate that the deployed detection rules (DET-DE-001-SIGMA and/or DET-DE-001-KQL) correctly generate an alert when a user uploads a volume of data exceeding the configured threshold to a personal cloud storage service from a corporate-managed endpoint.

Secondary objective: Validate that the proxy log data pipeline from corporate endpoint → web proxy → SIEM is functioning correctly and that alert latency is within acceptable bounds (< 10 minutes from upload to alert).

---

## 2. Prerequisites

- [ ] Detection rule DET-DE-001-SIGMA is deployed and in alert mode in the test SIEM (not in development/monitoring-only mode)
- [ ] OR: KQL detection queries from DET-DE-001-KQL are scheduled in Microsoft Sentinel with a run frequency of ≤ 5 minutes
- [ ] A dedicated test user account exists with a documented history less than 30 days old (to avoid triggering baseline anomaly detections prematurely)
- [ ] A corporate-managed test endpoint is available and enrolled in MDM/EDR
- [ ] The test endpoint routes traffic through the corporate web proxy
- [ ] The web proxy is confirmed to be logging to the SIEM (verify last log timestamp before executing the test)
- [ ] Synthetic test files have been created: minimum 5 files totaling 60 MB (to exceed the 50 MB threshold); all labeled "SIDTVF TEST DATA — NOT SENSITIVE"
- [ ] Written authorization from Security Manager is on file with date and name
- [ ] The alert triage queue has been notified that a test alert will fire (to prevent unnecessary escalation)
- [ ] A personal cloud storage test account has been created specifically for this test (do NOT use an existing personal account with real data)

---

## 3. Test Environment

| Field | Value |
|-------|-------|
| **Environment Type** | Test preferred; Production-with-safeguards requires additional authorization |
| **Test User Account** | [e.g., testuser-de001@[testdomain or prod domain]] |
| **Source System** | Corporate-managed endpoint with web proxy enforced, EDR installed |
| **Target System** | Personal Google Drive test account (created for this test, no real data) |
| **Test Data** | 5 synthetic files, each ~12 MB, totaling ~60 MB, named "SIDTVF-TEST-[number]-NOT-SENSITIVE.txt" |
| **Detection System** | SIEM with DET-DE-001-SIGMA deployed or KQL queries scheduled |
| **Monitoring Points** | Web proxy logs, EDR (DeviceNetworkEvents), DLP platform, SIEM alert queue |

---

## 4. Test Steps

### Sub-Test A — Single Large Upload (Threshold Trigger)

| Step | Action | Expected System Response |
|------|--------|------------------------|
| A1 | Log in to the test endpoint with the test user account | Authentication event logged |
| A2 | Open a web browser (use the same browser type as typical corporate users — e.g., Chrome) | Browser process visible in EDR |
| A3 | Navigate to `drive.google.com` using the test personal Google Drive account | Proxy log records the navigation to `drive.google.com`; URL categorization may classify as "Personal Storage" |
| A4 | Upload all 5 synthetic test files (totaling ~60 MB) using the browser's drag-and-drop or "New > File Upload" function | POST request(s) to `drive.google.com` recorded in proxy log with `SentBytes` ≥ 52,428,800 (50 MB); DLP may alert on upload |
| A5 | After upload completes, wait up to 10 minutes for the SIEM detection to process | Detection alert should appear in the SIEM alert queue |
| A6 | Check the SIEM alert queue for a DET-DE-001 alert attributed to the test user account | Alert should be present with correct user, domain, and byte count fields |
| A7 | Record the time from step A4 completion to alert appearance in the SIEM | Time-to-alert metric |

### Sub-Test B — Multiple Small Uploads (Cumulative Threshold Trigger)

> This sub-test validates Query 3 from DET-DE-001-KQL (cumulative daily upload detection).
> Execute this sub-test in a separate session if the primary threshold-based alert already fired.

| Step | Action | Expected System Response |
|------|--------|------------------------|
| B1 | Log in to the test endpoint with the test user account | Authentication logged |
| B2 | Upload 2 synthetic test files (~12 MB each) to the test Google Drive account | Two POST requests logged; each below the 50 MB per-request threshold individually |
| B3 | Wait 15 minutes, then upload 2 more files (~12 MB each) | Two more POST requests logged |
| B4 | Wait 15 minutes, then upload the remaining 1 file (~12 MB) | Cumulative total now ~60 MB for the day |
| B5 | Run the cumulative daily upload KQL query manually (DET-DE-001-KQL Query 3) | Query should return the test user with DailyMBUploaded ≥ 57 MB |
| B6 | Note whether the cumulative detection fired automatically (if scheduled) or only on manual execution | Documents whether per-request threshold alone is sufficient or if cumulative monitoring is needed |

---

## 5. Expected Results

**PASS Criteria for Sub-Test A:**

- [ ] The proxy log contains at least one POST request from the test user account to `drive.google.com` with `SentBytes` ≥ 52,428,800
- [ ] A SIEM alert is generated by the DET-DE-001 detection rule within 10 minutes of upload completion
- [ ] The alert correctly identifies: test user account, target domain (`drive.google.com`), byte volume > 50 MB
- [ ] No false positive alerts were generated for other users during the test window

**PASS Criteria for Sub-Test B:**

- [ ] The cumulative KQL query (Query 3, DET-DE-001-KQL) returns the test user with daily upload volume > 50 MB when run manually
- [ ] If the query is scheduled, it fires automatically when the cumulative threshold is crossed

**Detection alert must contain:**
- Alert name: "Insider Threat - Personal Cloud Storage Upload via Web Browser" (Sigma) or "DE-001 Cloud Upload Exfiltration Detected" (KQL)
- User field: The test user account name
- Domain field: `drive.google.com`
- Bytes field: Consistent with the test upload volume (≥ 50 MB)
- Triggered within: 10 minutes from test upload completion

**FAIL Criteria:**
- No alert fires within 15 minutes of completing a 60 MB upload to `drive.google.com`
- Alert fires but user account is missing or incorrect
- Alert fires but byte count is missing, zero, or significantly incorrect

---

## 6. Test Results Log

### Execution: [YYYY-MM-DD] — TEMPLATE ENTRY

| Field | Value |
|-------|-------|
| **Executed By** | [Name] |
| **Date** | [To be completed on execution] |
| **Environment** | [Test / Production-With-Safeguards] |
| **Result** | [PASS / FAIL / PARTIAL] |
| **Sub-Test A Result** | [PASS / FAIL / PARTIAL] |
| **Sub-Test B Result** | [PASS / FAIL / PARTIAL] |
| **Alert Fired?** | [Yes / No / Partial] |
| **Time to Alert (Sub-Test A)** | [Minutes] |
| **Alert Fields Correct?** | [Yes / No — describe discrepancy] |
| **False Positives Generated** | [Count] |
| **Notes** | [Observations] |

**Gap Analysis (complete if FAIL or PARTIAL):**
- Root cause:
- Gap category:
- Remediation:
- Remediation owner:
- Target date:
- Re-test date:

---

## 7. Cleanup Steps

| Step | Action |
|------|--------|
| 1 | Delete all test files uploaded to the Google Drive test account during the test |
| 2 | Delete the Google Drive test account (or document it as a test-only account with no real data) |
| 3 | Close any test browser sessions and log off the test user account |
| 4 | Notify the SOC alert triage team that the test window is closed and any remaining DET-DE-001 alerts from the test account can be closed as test artifacts |
| 5 | Document test completion in the test results log above |
| 6 | Archive the proxy and SIEM logs from the test window with a note identifying them as TC-DE-001 test data |

---

## 8. Linked Artifacts

| Artifact | ID | File |
|----------|----|------|
| Scenario | DE-001 | [scenarios/data-exfiltration/DE-001-cloud-storage-upload.md](../../scenarios/data-exfiltration/DE-001-cloud-storage-upload.md) |
| Detection (Sigma) | DET-DE-001-SIGMA | [detections/sigma/DE-001-cloud-storage-upload.yml](../../detections/sigma/DE-001-cloud-storage-upload.yml) |
| Detection (KQL) | DET-DE-001-KQL | [detections/queries/DE-001-kql.md](../../detections/queries/DE-001-kql.md) |
| Hunt Hypothesis | HYP-DE-001 | [hunting/hypotheses/HYP-DE-001.md](../../hunting/hypotheses/HYP-DE-001.md) |
| Response Playbook | PB-DE-001 | [playbooks/examples/PB-DE-001-cloud-storage-exfil.md](../../playbooks/examples/PB-DE-001-cloud-storage-exfil.md) |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-25 | SIDTVF Core Team | Initial creation |
