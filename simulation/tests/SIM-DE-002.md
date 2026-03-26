# Simulation Test: SIM-DE-002

| Field | Value |
|-------|-------|
| **Simulation ID** | SIM-DE-002 |
| **Linked Scenario** | DE-002 — Email Auto-Forwarding Rule |
| **Linked Validation** | TC-DE-002 |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-26 |
| **Last Updated** | 2026-03-26 |
| **Estimated Duration** | 20 minutes + 24h observation window (for scheduled detection) |
| **Required Environment** | Microsoft 365 test tenant or isolated test mailbox |
| **Minimum Maturity Level** | 2 |

---

## Objective

Reproduce the creation of an email forwarding rule that routes messages to an external personal email address. The simulation should produce Microsoft 365 Unified Audit Log events (`New-InboxRule`, `MailItemsAccessed`) and trigger the DE-002 detection rules.

---

## Authorization

> Complete the [Authorization Checklist](../authorization-checklist.md) before proceeding. This simulation creates real Exchange inbox rules in the M365 tenant. Ensure cleanup is completed within 4 hours of the test and that the M365 admin team is aware.

---

## Prerequisites

### Environment
- [ ] M365 Unified Audit Log is enabled for the test tenant (`Set-AdminAuditLogConfig -UnifiedAuditLogIngestionEnabled $true`)
- [ ] Audit logs are flowing to SIEM (Microsoft Sentinel or Splunk with M365 add-on)
- [ ] Test mailbox `sim.testuser@yourdomain.com` exists and has Exchange Online license
- [ ] A dedicated external test email address exists for receiving forwarded mail: `sim-forward-test@outlook.com` (create a fresh Outlook.com account for this purpose)
- [ ] Exchange audit logging is enabled: `Set-MailboxAuditBypassAssociation` is not set for the test account
- [ ] SOC has been notified of simulation window

### Tools Required
- Outlook Web App (OWA) at `outlook.office.com` — for creating the forwarding rule via GUI
- Exchange Online PowerShell — for verification and cleanup
- Microsoft Sentinel or Splunk — for alert observation

---

## Simulation Steps

### Phase 1 — Forwarding Rule Creation via OWA

This is the primary simulation step. Creates a `New-InboxRule` event in the Unified Audit Log.

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 1.1 | Log in to OWA as `sim.testuser@yourdomain.com` | Entra ID sign-in log | SignInLogs: UserPrincipalName=sim.testuser |
| 1.2 | Click Settings (gear icon) → View all Outlook settings → Mail → Rules | — | — |
| 1.3 | Click "Add new rule" | — | — |
| 1.4 | Name the rule: `SIM-TEST-RULE-DELETE-ME` | — | — |
| 1.5 | Condition: "Apply to all messages" | — | — |
| 1.6 | Action: "Forward to" → enter `sim-forward-test@outlook.com` | — | — |
| 1.7 | Save the rule | OfficeActivity (UAL) | Operation=New-InboxRule; Parameters contain ForwardTo=sim-forward-test@outlook.com |

**Expected artifact:** `OfficeActivity` event with `Operation = "New-InboxRule"` and `Parameters` containing the external forwarding address within 15 minutes of rule creation.

### Phase 2 — Triggering the Rule (Optional but Recommended)

This phase validates that forwarding is actually happening — useful for testing downstream mail-flow-based detection.

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 2.1 | From any internal account, send a test email to `sim.testuser@yourdomain.com` with subject: `[SIMULATION TEST - DISREGARD]` | Exchange message tracking | Message delivered to sim.testuser |
| 2.2 | Verify that `sim-forward-test@outlook.com` receives the forwarded message | External mailbox | Message received |
| 2.3 | In SIEM: search for `MailItemsAccessed` events for `sim.testuser` matching the delivery time | OfficeActivity (UAL) | Operation=MailItemsAccessed |

### Phase 3 — Post-Termination Simulation (Optional)

Tests the post-termination forwarding detection (HYP-DE-002 Query 3).

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 3.1 | Add `sim.testuser@yourdomain.com` to the `TerminatedEmployees` Sentinel watchlist with `TerminationDate` = yesterday's date | Sentinel watchlist | Watchlist updated |
| 3.2 | Re-run Phase 2 (send a test email) — the Unified Audit Log event will now show activity after the "termination date" | OfficeActivity | MailItemsAccessed with timestamp > TerminationDate |
| 3.3 | Verify that the HYP-DE-002 Query 3 returns this as a post-termination event | SIEM | KQL query result: DaysAfterTermination > 0 |
| 3.4 | Remove `sim.testuser` from the watchlist after verification | Sentinel watchlist | Entry removed |

---

## Expected Log Artifacts

| Log Source | Event Type | Key Field | Expected Value |
|------------|------------|-----------|----------------|
| OfficeActivity (UAL) | New-InboxRule | Operation | `New-InboxRule` |
| OfficeActivity (UAL) | New-InboxRule | UserId | `sim.testuser@yourdomain.com` |
| OfficeActivity (UAL) | New-InboxRule | Parameters | Contains `ForwardTo` = `sim-forward-test@outlook.com` |
| Entra ID SignInLogs | Sign-in | UserPrincipalName | `sim.testuser@yourdomain.com` |
| OfficeActivity (UAL) | MailItemsAccessed | Operation | `MailItemsAccessed` (Phase 2) |

---

## Expected Detection Output

| Detection Rule | Expected Trigger | Time to Alert |
|---------------|-----------------|---------------|
| HYP-DE-002 Query 1 (KQL) | New-InboxRule with external ForwardTo | < 15 minutes (real-time rule) |
| HYP-DE-002 Query 2 (PowerShell) | Rule appears in monthly snapshot | Next scheduled run |
| HYP-DE-002 Query 3 (KQL) | Activity after simulated termination date (Phase 3) | < 15 minutes |

> **Note:** The PowerShell snapshot (Query 2) is a scheduled manual hunt, not a real-time alert. It will not fire immediately — it should appear the next time the hunt is run.

---

## Observation Window

| Detection Type | Observation Window |
|---------------|-------------------|
| Real-time Sentinel analytics rule | 15 minutes |
| UAL ingestion delay | Allow up to 90 minutes for UAL events to appear in Sentinel |
| Scheduled hunt query | Next manual run |

---

## Pass / Fail Criteria

| Result | Criteria |
|--------|----------|
| **PASS** | UAL event `New-InboxRule` appears in Sentinel within 90 minutes AND the KQL detection fires an alert containing the correct user and external forwarding address |
| **PARTIAL** | UAL event appears but detection does not fire (rule misconfiguration gap); OR detection fires but is missing the forwarding target field |
| **FAIL** | UAL event does not appear in Sentinel within 90 minutes (logging pipeline gap) |

---

## Common Failure Modes

| Failure Mode | Likely Cause | Remediation |
|-------------|-------------|-------------|
| No UAL event in Sentinel | UAL ingestion connector not enabled | Enable the Microsoft 365 data connector in Sentinel |
| UAL event appears but parameters are empty | Audit log schema change | Check OfficeActivity schema; adjust Parameters parsing in KQL |
| Alert fires but shows wrong user | Multi-user test environment collision | Use a dedicated, isolated test mailbox |
| Rule creation succeeds but no forwarding occurs | Exchange transport rules blocking | Check transport rules; this may be a valid detection control |

---

## Cleanup Steps

| Step | Action | Verified By | Date |
|------|--------|-------------|------|
| C1 | Delete the `SIM-TEST-RULE-DELETE-ME` rule from `sim.testuser`'s inbox via OWA or PowerShell: `Remove-InboxRule -Identity "SIM-TEST-RULE-DELETE-ME" -Mailbox sim.testuser@yourdomain.com` | | |
| C2 | Verify rule is removed: `Get-InboxRule -Mailbox sim.testuser@yourdomain.com` | | |
| C3 | Remove `sim.testuser` from the TerminatedEmployees watchlist (if Phase 3 was run) | | |
| C4 | Delete the test forwarding address account (`sim-forward-test@outlook.com`) or document it as a persistent test account for future simulations | | |
| C5 | Mark any generated SIEM alerts as "SIM-DE-002 — simulation test" in the ticketing system | | |
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
| T1114.003 | No direct ART equivalent | ART does not cover M365 inbox rule creation; this simulation fills that gap |
| T1098.003 | No direct ART equivalent | Delegate permission grants also lack ART coverage |

---

## Linked Artifacts

| Type | ID | File |
|------|----|------|
| Scenario | DE-002 | scenarios/data-exfiltration/DE-002-email-forwarding.md |
| Hunt Hypothesis | HYP-DE-002 | hunting/hypotheses/HYP-DE-002.md |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-26 | SIDTVF Core Team | Initial creation |
