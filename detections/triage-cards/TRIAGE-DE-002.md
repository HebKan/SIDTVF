# SOC Triage Card: DE-002 — Email Auto-Forwarding Rule

> **Print this card and keep it at your analyst station.**
> **Email forwarding rules are silent — the user may not know you found them.**

---

## Alert Summary

| Field | What You're Seeing |
|-------|-------------------|
| **Rule** | HYP-DE-002 KQL Query 1 / Exchange PowerShell snapshot |
| **Trigger** | User created or modified an inbox rule that forwards email to an external address |
| **Risk** | Ongoing, automated email exfiltration — may have been active for days or weeks before detection |
| **Actor type** | MAL (departing employee or industrial spy) or NEG (setting up convenience forward unaware of policy) |

---

## Step 1 — Confirm the Rule Exists (5 minutes)

Before doing anything else, verify the rule is still active:

```powershell
Get-InboxRule -Mailbox "[username]" | Where-Object {
    $_.ForwardTo -ne $null -or $_.RedirectTo -ne $null
} | Select-Object Name, ForwardTo, RedirectTo, DeleteMessage, Enabled
```

Record:
- Rule name: _______________________________
- Forward target: _______________________________
- DeleteMessage setting: [ ] True [ ] False
- Rule enabled: [ ] True [ ] False
- Rule created date (if visible): _______________________________

---

## Step 2 — Four Questions Before You Escalate (10 minutes)

**Q1: Is the forwarding target a free webmail or personal domain?**
- High-risk targets: `gmail.com` · `yahoo.com` · `hotmail.com` · `protonmail.com` · `proton.me` · `tutanota.com`
- Unknown domain? Run a WHOIS lookup — when was the domain registered? New domains (< 90 days) are red flags.
- Result: [ ] Free webmail [ ] Unknown external domain [ ] Business domain (may be legitimate)

**Q2: Is DeleteMessage set to True?**
- This is the highest-severity indicator. If `DeleteMessage = True`, the rule forwards the email AND deletes the original — leaving no trace in the user's sent/inbox.
- Result: [ ] Yes — CRITICAL [ ] No

**Q3: Is this user in the DepartingEmployees watchlist?**
- Check watchlist or HR: `SELECT * FROM departing_employees WHERE username = '[user]'`
- Result: [ ] Yes — last day: _______ [ ] No [ ] Unknown

**Q4: How long has the rule been active?**
- Check UAL for the `New-InboxRule` event date: when was this rule created?
- How many days of email may have been forwarded? _______ days
- Result: [ ] < 7 days [ ] 7–30 days [ ] > 30 days

---

## Step 3 — Severity Decision

| Conditions | Severity | Action |
|-----------|----------|--------|
| DeleteMessage=True, any target | **CRITICAL** | Escalate to IR immediately — active evidence destruction |
| Target is free webmail (gmail, protonmail, etc.) AND Q3=Yes | **CRITICAL** | Escalate to IR immediately |
| Target is free webmail, rule active > 30 days | **HIGH** | Escalate to IR within 30 minutes |
| Target is unknown domain, rule active > 7 days | **HIGH** | Escalate to IR within 30 minutes |
| Target is business domain, first offense, no departure | **MEDIUM** | Tier 2 investigation; verify with user |
| Target is business domain, rule just created, user has business reason | **LOW** | Verify with user; may be legitimate (shared mailbox setup) |

---

## Step 4 — What To Do Right Now

**If CRITICAL:**
1. DO NOT disable the rule yet — this tips off the subject and may destroy evidence
2. Open a CRITICAL incident ticket immediately
3. Notify IR Lead: "DE-002 CRITICAL — [user] — forwarding to [target] — DeleteMessage=[True/False]"
4. Preserve: Screenshot the rule settings; export UAL logs for this user for 90 days
5. Let Legal decide whether and when to disable the rule

**If HIGH:**
1. Open a HIGH incident ticket
2. Assign to IR for investigation within 30 minutes
3. Preserve: Export proxy logs and UAL for 30 days
4. Do NOT disable the rule until IR Lead authorizes

**If MEDIUM:**
1. Open an investigation ticket
2. Tier 2 contacts the user's manager (not the user directly) to verify business purpose
3. If no business purpose: disable rule with manager authorization; document

**If LOW:**
1. Verify with the user via their manager — may be a legitimate setup
2. If legitimate, document with change management; add to exception list
3. If not legitimate, treat as a policy violation and disable with manager authorization

---

## Volume Assessment (for IR handoff)

If escalating, estimate the scope before handing off:

1. Check when the rule was created (UAL `New-InboxRule` event timestamp)
2. Estimate email volume: check MailItemsAccessed events or message tracking logs
3. Look for high-value subjects: did the rule apply to all messages, or only to filtered subjects (e.g., "budget," "board," "confidential")? Targeted rules are more serious than catch-all rules.

---

## What NOT To Do

- **Do NOT disable the rule** without IR Lead authorization — preserving the live configuration is evidence
- **Do NOT contact the user** before IR and Legal clear it
- **Do NOT mark as benign** just because the user is not on the departure list — external recruitment and long-horizon pre-positioning exist
- **Do NOT ignore** a rule with `DeleteMessage=True` regardless of other indicators

---

## Escalation Contacts

| Role | Contact | When |
|------|---------|------|
| Tier 2 SOC | [contact] | All MEDIUM+ alerts |
| IR Lead | [contact] | All HIGH+ alerts |
| Legal | [contact] | Before disabling the rule or contacting the user |
| Exchange Admin | [contact] | For rule removal after IR authorization |

---

**Full hunt hypothesis:** `hunting/hypotheses/HYP-DE-002.md`
**Full scenario:** `scenarios/data-exfiltration/DE-002-email-forwarding.md`
