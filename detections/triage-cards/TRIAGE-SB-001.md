# SOC Triage Card: SB-001 — Targeted Database Deletion

> **Print this card and keep it at your analyst station.**
> **TIME IS CRITICAL. Every minute of delay increases data loss. Start recovery in parallel with investigation.**

---

## Alert Summary

| Field | What You're Seeing |
|-------|-------------------|
| **Rule** | DET-SB-001 — DROP TABLE / DELETE DATABASE / backup system events |
| **Trigger** | Privileged user executed database destruction commands (DROP, DELETE, TRUNCATE on multiple objects) |
| **Risk** | Catastrophic data loss; business disruption; possible criminal sabotage |
| **Actor type** | MAL (disgruntled employee — often linked to recent negative employment event) |

---

## IMMEDIATE: Is This Happening Right Now?

Answer in the first 2 minutes:

- [ ] Can the application connect to the database? (Test: try to open the application or query the DB)
- [ ] Are database services running? (Check operations dashboard or monitoring tool)
- [ ] When did the DROP events occur — past or ongoing? (Check alert timestamp vs. now)

**If database is DOWN or destruction is ONGOING: Skip to Step 4 immediately. Alert the DBA on-call now.**

---

## Step 1 — Confirm the Alert (3 minutes)

- [ ] The DROP/DELETE events are on production or business-critical databases (not test/dev)
- [ ] The account is a human account, not an application service account
- [ ] The events occurred outside of a scheduled maintenance window (check change management)
- [ ] Multiple objects (tables, databases) were affected — not just a single accidental delete

Record:
- Database server: _______________________________
- Database name(s) affected: _______________________________
- Account that performed the action: _______________________________
- Time of first DROP event: _______________________________
- Number of objects destroyed: _______________________________

---

## Step 2 — Three Questions (5 minutes, parallel with Step 4 if CRITICAL)

**Q1: What is the employment status of the account holder?**
- Check HR immediately: is this person recently terminated, on a PIP, or in a dispute?
- Also check: was a termination announced today or this week?
- Result: [ ] Active, no known issues [ ] PIP/disciplinary history [ ] Recently terminated [ ] Unknown

**Q2: Were backup systems targeted at the same time?**
- Check if backup deletion events, backup device removal, or backup service stops appear in the same session
- Result: [ ] Yes — CRITICAL (recovery may be impossible) [ ] No [ ] Unknown

**Q3: Are there scheduled tasks or logic bombs?**
- Check for `CREATE EVENT`, `CREATE JOB`, `schtasks`, or `cron` entries created by this account recently
- Result: [ ] Suspicious tasks found [ ] No suspicious tasks [ ] Logs unavailable

---

## Step 3 — Severity Decision

| Conditions | Severity | Action |
|-----------|----------|--------|
| Production DB destroyed AND backups compromised | **CRITICAL** | Immediate all-hands. Call CISO + DBA + Legal NOW. |
| Production DB destroyed, backups intact | **CRITICAL** | Immediate escalation. Recovery begins NOW. |
| Non-production DB destroyed, account is terminated employee | **HIGH** | Escalate to IR within 15 minutes. |
| Destruction attempt was blocked/failed | **MEDIUM** | IR investigation; determine if further attempts expected. |
| Single table dropped in test environment | **LOW** | Verify with change management; may be authorized. |

> **Default for SB-001: Treat as CRITICAL until proven otherwise.**

---

## Step 4 — What To Do Right Now (CRITICAL response)

**These actions run in PARALLEL — split responsibilities now:**

**Track A — Evidence (IR):**
1. Export DB audit logs for the affected server — NOW, before they roll
2. Export endpoint and authentication logs for the account — 30 days
3. Screenshot the current DB server state (running processes, connections)
4. Hash all collected logs immediately (SHA-256)
5. Do NOT reimage or restart any affected server without Legal sign-off

**Track B — Recovery (DBA on-call):**
1. Assess backup integrity: when is the last clean backup? Is it intact?
2. Begin DR runbook — do not wait for full investigation scope
3. Determine RPO gap: what data is lost between backup and deletion?
4. Notify application owners of outage and estimated recovery time

**Track C — Notification (SOC Lead):**
1. Notify CISO: "SB-001 CRITICAL — [account] — [database] — [time]"
2. Notify Legal: insider sabotage event in progress — they must be on immediately
3. Notify HR: confirm employment status of [account]; escalation may require HR coordination
4. Open CRITICAL incident ticket

**Track D — Containment (IAM — after IR Lead authorization):**
1. IR Lead authorizes: revoke all active sessions for the account (AD, DB, VPN, cloud)
2. Disable the account in Active Directory / Entra ID
3. Revoke SSH keys, DB credentials, API tokens
4. Check for any service accounts the user may have created — disable pending review

---

## Step 5 — Look for Logic Bombs (15 minutes)

Before completing containment, check for planted persistence:

```sql
-- SQL Server: Check for suspicious scheduled jobs
SELECT name, enabled, description FROM msdb.dbo.sysjobs
WHERE date_created > DATEADD(day, -30, GETDATE())
ORDER BY date_created DESC;

-- Check for scheduled tasks (Windows)
-- Run: schtasks /query /fo LIST /v | findstr /i "task\|run\|trigger"

-- Check for stored procedures recently modified
SELECT name, modify_date FROM sys.objects
WHERE type='P' AND modify_date > DATEADD(day, -30, GETDATE());
```

If ANY suspicious scheduled jobs or modified stored procedures are found: treat as an active, ongoing attack with potential future triggers. Preserve and escalate.

---

## What NOT To Do

- **Do NOT delay recovery waiting for a complete investigation** — recovery and investigation run in parallel
- **Do NOT reimage the database server** without Legal and IR Lead sign-off — it destroys evidence
- **Do NOT restart database services** without DBA confirmation (restart may overwrite transaction logs needed for forensic recovery)
- **Do NOT notify the suspected account holder** if still employed
- **Do NOT mark as false positive** because "this person wouldn't do this" — database deletions by named human accounts are almost always intentional at this scale

---

## Common False Positives

| Scenario | How to Rule It Out |
|----------|-------------------|
| Scheduled maintenance / purge job | Verify against approved change request; check if the executing account is a service account |
| DBA accidentally dropped wrong table | User reports immediately; no cover-up; only one object affected |
| Decommissioning cleanup | Verify against decommissioning ticket in ITSM |
| Automated test teardown | Target database is labeled as test/dev; executing account is CI/CD service account |

---

## Escalation Contacts

| Role | Contact | When |
|------|---------|------|
| DBA on-call | [contact] | Immediately — parallel with IR |
| IR Lead | [contact] | Immediately |
| CISO | [contact] | Within 15 minutes for CRITICAL |
| Legal | [contact] | Within 15 minutes — must be on the incident |
| HR | [contact] | Within 30 minutes — employment status needed |

---

**Full playbook:** `playbooks/examples/PB-SB-001-database-deletion.md`
**Full scenario:** `scenarios/sabotage/SB-001-targeted-database-deletion.md`
