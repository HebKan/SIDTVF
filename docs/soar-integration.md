# SOAR Integration Guide

> **Version:** 1.0 | **Last Updated:** 2026-03-26
>
> This guide describes how to connect SIDTVF detections to Security Orchestration, Automation, and Response (SOAR) platforms so that alerts automatically trigger evidence collection, enrichment, and containment actions.

---

## Why SOAR Integration Matters

A SIDTVF detection rule firing in a SIEM is the beginning of a response process, not the end. Without automation:
- A SOC analyst must manually collect logs, query HR systems, check the user's manager, and assess severity — under time pressure
- Steps get skipped, especially during high-volume periods or off-hours
- Response times vary widely by analyst experience

SOAR integration enforces consistent, repeatable execution of the first 3–5 steps of every playbook, regardless of who is on duty.

**What to automate vs. what to keep manual:**

| Automate | Keep Manual |
|----------|------------|
| Log collection and preservation | Access revocation (requires Legal approval gate) |
| HR data enrichment (is user departing?) | Employee notification |
| Ticket creation and assignment | Law enforcement referral |
| Alert deduplication | Breach notification determination |
| Initial evidence hashing | Termination decision |
| Analyst notification | Any action requiring HR or Legal sign-off |

---

## Platform-Specific Integration

### Microsoft Sentinel Automation (Logic Apps)

Sentinel Automation Rules trigger Logic Apps when an analytics rule fires. For SIDTVF detections built in KQL:

**Step 1 — Create an Automation Rule**
```
In Sentinel → Automation → Create → Automation Rule
- Trigger: When alert is created
- Condition: Alert name contains "DE-001" OR "DE-002" (etc.)
- Action: Run playbook → [Logic App name]
```

**Step 2 — Logic App Structure for DE-001**

```
Trigger: Sentinel Alert
    ↓
Get Entity Details (parse AccountName from alert)
    ↓
[HTTP] Query HR system API: GET /employees/{AccountName}
    → Returns: department, manager, employment_status, last_day
    ↓
[Condition] Is employment_status = "departing" AND days_remaining < 30?
    → Yes: Set severity = Critical
    → No: Set severity = High
    ↓
[Sentinel] Update incident: add HR enrichment as comment
    ↓
[Teams/Slack] Post to #security-alerts:
    "DE-001 Alert: {user} uploaded {bytes}MB to {domain}.
     HR Status: {employment_status}. Days remaining: {days_remaining}.
     Severity: {severity}. Ticket: {ticket_url}"
    ↓
[ServiceNow/Jira] Create incident ticket with all enrichment data
    ↓
[Sentinel] Assign incident to On-Call SOC analyst
```

**Logic App connector requirements:**
- Microsoft Sentinel connector (built-in)
- HTTP connector (for HR system API calls)
- Microsoft Teams or Slack connector
- ServiceNow, Jira, or your ITSM connector

**Key considerations:**
- The Logic App must have Managed Identity permissions to read Sentinel incidents
- HR system API authentication requires a service account — do not hardcode credentials; use Azure Key Vault
- Add a timeout handler — if the HR system is unavailable, the Logic App should proceed with a "HR data unavailable" note, not fail the automation

---

### Splunk SOAR (formerly Phantom)

Splunk SOAR playbooks are Python-based automation workflows. For SIDTVF detections built in SPL:

**Step 1 — Ingest SIEM Alerts into SOAR**

Configure Splunk SIEM → Splunk SOAR event ingestion:
```
In Splunk SIEM: Create a saved search for each DE-001 SPL query
Enable "Send to SOAR" on the search action
Map fields: src_user → user, bytes_out → upload_bytes, dest_host → cloud_domain
```

**Step 2 — SOAR Playbook Structure (Python pseudo-code)**

```python
# SIDTVF-DE-001-Response Playbook
# Trigger: Event with label "insider_threat_cloud_upload"

def on_start(container):
    # Phase 1: Enrich with HR data
    hr_data = query_hr_system(container.user)
    phantom.act("run query",
                parameters=[{"query": f"SELECT * FROM employees WHERE username='{container.user}'"}],
                assets=["hr_database"],
                callback=hr_enrichment_callback)

def hr_enrichment_callback(action, success, container, results):
    hr_record = results[0] if results else {}
    days_remaining = hr_record.get("days_until_last_day", 999)

    # Phase 2: Set severity based on HR context
    if days_remaining <= 30:
        phantom.set_severity(container=container, severity="critical")
        phantom.add_comment(container=container,
                           comment=f"DEPARTING EMPLOYEE: {days_remaining} days remaining")

    # Phase 3: Collect supporting evidence
    phantom.act("run query",
                parameters=[{"query": f"SELECT * FROM proxy_logs WHERE user='{container.user}' "
                                     f"AND _time > now()-24h AND dest IN ({CLOUD_DOMAINS})"}],
                assets=["splunk"],
                callback=evidence_collection_callback)

def evidence_collection_callback(action, success, container, results):
    # Hash and store evidence
    evidence_summary = summarize_evidence(results)
    phantom.add_attachment(container=container, content=evidence_summary)

    # Phase 4: Create ticket and notify
    phantom.act("create ticket",
                parameters=[{
                    "summary": f"[DE-001] Insider Threat Alert: {container.user}",
                    "description": evidence_summary,
                    "priority": container.severity
                }],
                assets=["jira"])

    phantom.act("send message",
                parameters=[{
                    "destination": ["soc-on-call@yourdomain.com"],
                    "subject": f"[SIDTVF DE-001] {container.user} — {container.upload_bytes}MB upload",
                    "body": evidence_summary
                }],
                assets=["smtp"])

    # DECISION GATE: Do NOT automate access revocation
    # Prompt analyst for manual decision
    phantom.prompt(container=container,
                   user="on_call_analyst",
                   message=f"DE-001 alert for {container.user}. Evidence attached. "
                           f"HR status: {days_remaining} days remaining.\n"
                           f"ACTION REQUIRED: Review evidence and decide: Escalate / Close / Monitor",
                   respond_in_mins=30)
```

**Important:** The `phantom.prompt()` call creates a manual decision gate. Access revocation is never automated — it always requires analyst confirmation and Legal awareness.

---

### Palo Alto XSOAR

XSOAR uses YAML-defined playbooks with Python automation scripts. For SIDTVF integration:

**Step 1 — Create an Incident Type**
```yaml
# In XSOAR: Settings → Incident Types → New
name: "SIDTVF Insider Threat Alert"
playbook: "SIDTVF-Triage-Playbook"
hours_before_sla_breach: 24
custom_fields:
  - sidtvf_scenario_id (Short Text)
  - insider_hr_status (Short Text)
  - upload_volume_mb (Number)
  - cloud_domain (Short Text)
```

**Step 2 — Playbook Outline (XSOAR)**

```
Task 1 (automatic): Extract fields from alert (user, bytes, domain, timestamp)
Task 2 (automatic): Enrich user from AD/LDAP (get manager, department, title)
Task 3 (automatic): Query HR system — employment status and termination date
Task 4 (automatic): Query DepartingEmployees watchlist — is user on it?
Task 5 (automatic): Collect 24h proxy history for user from SIEM
Task 6 (automatic): Hash all collected evidence and attach to incident
Task 7 (automatic): Calculate SIDTVF risk score (upload volume + HR status + history)
Task 8 (manual): Analyst reviews enrichment — sets severity and disposition
Task 9 (conditional):
  IF severity = Critical AND analyst approves:
    → Notify CISO + Legal (automated email)
    → Create HR ticket (automated)
    → WAIT for Legal confirmation before any access action
  ELSE:
    → Add to monitoring queue
    → Set next review date
```

**Integration packs required:**
- Active Directory / LDAP (built-in)
- Generic REST API (for HR system)
- SIEM integration (Splunk or Sentinel, depending on your stack)
- Email integration (for notifications)
- ServiceNow / Jira (for ticket creation)

---

## Generic Webhook Integration

For organizations without a dedicated SOAR platform, a webhook-based automation can provide the most important capabilities using simple HTTP calls.

**Minimal automation using cloud functions (AWS Lambda / Azure Functions):**

```python
# Triggered by SIEM webhook when DE-001 alert fires
import json, requests, hashlib, datetime

def handler(event, context):
    alert = json.loads(event['body'])
    user = alert['src_user']
    bytes_out = alert['total_bytes_out']
    domain = alert['cloud_domain']

    # 1. Enrich from HR system
    hr = requests.get(f"https://hr-api.internal/employees/{user}",
                      headers={"Authorization": f"Bearer {HR_API_TOKEN}"}).json()

    # 2. Collect supporting evidence (SIEM API query)
    evidence = query_siem_for_user_history(user, hours=24)

    # 3. Create incident ticket
    ticket = create_jira_ticket({
        "summary": f"[SIDTVF DE-001] {user} uploaded {bytes_out/1048576:.0f}MB to {domain}",
        "description": format_evidence(evidence, hr),
        "priority": "Critical" if hr.get("days_until_last_day", 999) < 30 else "High"
    })

    # 4. Notify SOC
    post_to_slack(channel="#soc-alerts",
                  message=f":rotating_light: DE-001 Alert: `{user}` uploaded "
                          f"{bytes_out/1048576:.0f}MB to `{domain}`. "
                          f"HR status: {hr.get('employment_status', 'unknown')}. "
                          f"Ticket: {ticket['url']}")

    return {"statusCode": 200}
```

This approach requires no SOAR license and can be implemented in a few hours.

---

## SOAR Decision Gate Implementation

Every SIDTVF playbook contains DECISION GATEs — points where human judgment is required. These must never be automated. Implement them in SOAR as:

| SOAR Platform | Decision Gate Implementation |
|--------------|----------------------------|
| Sentinel Logic Apps | Approval action in Logic App (emails approver; workflow waits for response) |
| Splunk SOAR | `phantom.prompt()` — blocks playbook until analyst responds |
| XSOAR | Manual task — workflow pauses until marked complete |
| Webhook / Lambda | Post to Slack with approval buttons (Slack interactive components) |

**Never automate:**
- Account suspension or access revocation
- Employee notification
- Law enforcement contact
- Breach notification determination
- Any action requiring Legal or HR sign-off

---

## Testing Your SOAR Integration

After building your automation, verify it using the SIDTVF simulation tests:

1. Run [simulation/tests/SIM-DE-001.md](../simulation/tests/SIM-DE-001.md)
2. Observe whether the SIEM alert triggers the SOAR playbook
3. Confirm that all enrichment steps complete successfully
4. Verify the analyst notification arrives with correct data
5. Confirm the DECISION GATE blocks on automated action
6. Complete cleanup per the simulation test cleanup steps

Document the end-to-end alert-to-ticket time. Target: < 5 minutes for automated steps.
