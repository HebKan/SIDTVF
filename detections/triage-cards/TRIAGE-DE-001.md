# SOC Triage Card: DE-001 — Cloud Storage Upload Exfiltration

> **Print this card and keep it at your analyst station.**
> **Do not close an alert until you have answered all three decision questions.**

---

## Alert Summary

| Field | What You're Seeing |
|-------|-------------------|
| **Rule** | DET-DE-001-SIGMA / KQL / SPL |
| **Trigger** | User uploaded > 50 MB to a personal cloud storage domain in a 1-hour window |
| **Risk** | Data exfiltration before or during departure; staging for later transfer |
| **Actor type** | MAL (departing employee) or NEG (unaware of policy) |

---

## Step 1 — Confirm the Alert (5 minutes)

Check each item. If any item is false, escalate immediately (do not wait for all items).

- [ ] The destination domain is a **personal** cloud storage service (not corporate OneDrive, SharePoint, or sanctioned Box)
- [ ] The upload volume exceeds your threshold — confirm total bytes: _________ MB
- [ ] The alert is not in a known maintenance window or approved testing window
- [ ] The user is a real, active employee (not a service account or test account)

**Personal cloud domains to look for:**
`drive.google.com` · `*.dropbox.com` · `*.icloud.com` · `*.box.com` (personal) · `*.onedrive.live.com` · `*.mega.nz` · `*.wetransfer.com` · `*.mediafire.com`

---

## Step 2 — Three Questions Before You Escalate (10 minutes)

Answer these before making any escalation or containment decision:

**Q1: Is this user in the DepartingEmployees watchlist?**
- Check watchlist or query HR system: `SELECT * FROM departing_employees WHERE username = '[user]'`
- Result: [ ] Yes — last day: _______ [ ] No [ ] Unknown

**Q2: What file types were staged on the endpoint before the upload?**
- Check endpoint agent logs (DeviceFileEvents) for 2 hours before the alert
- Look for: `.zip` archive creation, bulk file reads, copy from network shares
- Result: [ ] Suspicious staging pattern found [ ] Normal file activity [ ] Logs unavailable

**Q3: Has this user accessed this cloud domain before?**
- Check 30-day proxy history: is this the first time? Was there a pattern building?
- Result: [ ] First time ever [ ] Occasional use (< 5x in 30 days) [ ] Regular pattern

---

## Step 3 — Severity Decision

| Score | Severity | Action |
|-------|----------|--------|
| Q1=Yes AND volume > 200 MB | **CRITICAL** | Escalate to IR Lead immediately. Do NOT revoke access yet. |
| Q1=Yes OR Q2=Yes | **HIGH** | Escalate to Tier 2. Notify IR Lead within 30 minutes. |
| Q1=No AND Q2=No AND volume 50–200 MB | **MEDIUM** | Tier 2 investigation. Review within 4 hours. |
| First-time user, volume 50–100 MB, no staging | **LOW** | User education. Review. May be policy violation only (see PV-001). |

---

## Step 4 — What To Do Right Now

**If CRITICAL or HIGH:**
1. Open an incident ticket immediately — do NOT wait for Tier 2 to do this
2. Preserve: Export proxy logs for this user for the past 30 days NOW (logs may roll)
3. Notify IR Lead: "DE-001 alert — [user] — [volume] MB — [HR status]"
4. Do NOT contact the user, the user's manager, or revoke access without IR Lead authorization
5. Do NOT send any alerts to the user's email account (they may see it)

**If MEDIUM:**
1. Open an investigation ticket
2. Assign to Tier 2 for review within 4 hours
3. Note HR status and staging pattern in the ticket

**If LOW:**
1. Open a policy violation ticket
2. Assign to the user's manager for awareness
3. Note the specific upload and time in the ticket

---

## What NOT To Do

- **Do NOT revoke access** without IR Lead + Legal authorization
- **Do NOT contact the user** until IR Lead or Legal advises
- **Do NOT mark as false positive** just because the user "seems unlikely" — confirm with the evidence
- **Do NOT wait** if logs are about to roll — preserve first, investigate second

---

## Escalation Contacts

| Role | Contact | When |
|------|---------|------|
| Tier 2 SOC | [contact] | All MEDIUM+ alerts |
| IR Lead | [contact] | All HIGH+ alerts within 30 min |
| Legal | [contact] | Before any containment action |
| HR | [contact] | When employment status confirmed as relevant |

---

## Common False Positives

| Scenario | How to Rule It Out |
|----------|-------------------|
| IT admin running authorized backup | Verify against change management — ask Tier 2 |
| User syncing corporate OneDrive to personal device | Destination should be `onedrive.live.com` personal vs. `sharepoint.com` corporate |
| Developer using GitHub personal for public open-source work | Check file types — source code going to public repo is still a policy review item |
| Marketing team uploading to WeTransfer for client delivery | Verify with manager — if approved, document and whitelist for that user |

---

**Full playbook:** `playbooks/examples/PB-DE-001-cloud-storage-exfil.md`
**Full scenario:** `scenarios/data-exfiltration/DE-001-cloud-storage-upload.md`
