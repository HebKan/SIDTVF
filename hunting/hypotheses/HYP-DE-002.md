# Hunt Hypothesis: HYP-DE-002

| Field | Value |
|-------|-------|
| **Hypothesis ID** | HYP-DE-002 |
| **Linked Scenario** | DE-002 — Email Auto-Forwarding Rule |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-26 |
| **Last Updated** | 2026-03-26 |
| **Author** | SIDTVF Core Team |
| **Priority** | High |
| **Recommended Hunt Frequency** | Monthly |
| **Minimum Maturity Level** | 2 |

---

## Hypothesis Statements

### Primary Hypothesis

**IF** a user has created an email forwarding rule or delegate permission that routes messages to an external domain not on the organizational allowlist, **THEN** an insider data exfiltration channel may be active that bypasses DLP controls applied to outbound email — **VISIBLE IN** Microsoft 365 Unified Audit Log (`New-InboxRule`, `Set-InboxRule`, `Add-MailboxPermission` operations) or Exchange audit logs.

### Secondary Hypothesis

**IF** a terminated or recently resigned employee's mailbox has an active forwarding rule that was not removed during offboarding, **THEN** organizational email is continuing to reach an external destination after employment ended — **VISIBLE IN** Exchange Online mailbox forwarding settings cross-referenced with HR termination dates.

### Tertiary Hypothesis

**IF** forwarding rules are active on the mailboxes of employees with access to sensitive information (Finance, Legal, HR, Executive) that route to free webmail domains (gmail.com, yahoo.com, protonmail.com, tutanota.com), **THEN** high-value communications may be systematically exfiltrated — **VISIBLE IN** Exchange PowerShell audit or Microsoft 365 Unified Audit Log enriched with HR role data.

---

## Hunt Scope

| Parameter | Value |
|-----------|-------|
| **Data Sources** | Microsoft 365 Unified Audit Log; Exchange Online audit; Entra ID sign-in logs |
| **Time Window** | 90 days for initial hunt; ongoing for monthly recurring |
| **Population** | All active mailbox users; extend to recently terminated users (past 90 days) |
| **Exclusions** | IT-created rules with documented business justification; shared mailbox forwarding approved by management |
| **Required Permissions** | Exchange Online admin or Global Reader; Compliance admin for Unified Audit Log |

---

## Query 1 — Active External Forwarding Rules (Unified Audit Log — KQL)

Identifies all `New-InboxRule` and `Set-InboxRule` operations where the `ForwardTo` or `RedirectTo` targets are external email domains. Run in Microsoft Sentinel or Log Analytics.

```kql
// Hunt Query 1: External email forwarding rule creation/modification
// Source: OfficeActivity (Microsoft 365 Unified Audit Log)
// Requires: M365 diagnostic settings to Log Analytics / Sentinel enabled

OfficeActivity
| where TimeGenerated > ago(90d)
| where Operation in ("New-InboxRule", "Set-InboxRule")
| extend Parameters = parse_json(Parameters)
// Extract the forwarding/redirect targets from the rule parameters
| mv-expand param = Parameters
| where param.Name in ("ForwardTo", "RedirectTo", "ForwardAsAttachmentTo")
| extend ForwardTarget = tostring(param.Value)
// Identify external targets (not ending in the organization's domain)
| where ForwardTarget !endswith "@yourdomain.com"
      and ForwardTarget !endswith "@yourdomain.onmicrosoft.com"
      and ForwardTarget !contains "[EX:"         // exclude internal Exchange DN format
      and isnotempty(ForwardTarget)
// Flag high-risk free email domains
| extend HighRiskDomain = case(
    ForwardTarget has_any ("gmail.com", "yahoo.com", "hotmail.com",
        "outlook.com", "protonmail.com", "proton.me", "tutanota.com",
        "guerrillamail.com", "tempmail.com", "10minutemail.com"),
    "YES", "NO")
| project
    TimeGenerated,
    UserId,
    ClientIP,
    ForwardTarget,
    HighRiskDomain,
    Operation,
    ResultStatus,
    UserAgent
| sort by HighRiskDomain desc, TimeGenerated asc
```

**Expected Normal Results:** Few to none. IT-created forwarding rules should be documented and excludable. Any forwarding to free webmail domains is anomalous.

**Anomalous Results:** Any forwarding rule targeting external domains, especially to free webmail, personal domains, or domains associated with competitors.

---

## Query 2 — Currently Active Forwarding Rules (Exchange PowerShell Snapshot)

Point-in-time inventory of all active inbox rules with external forwarding or redirect actions. Run via Exchange Online PowerShell. This query does not require historical log data — it reads current mailbox state.

```powershell
# Hunt Query 2: Inventory all active external forwarding rules
# Requires: Exchange Online PowerShell module; Exchange Admin or Compliance role
# Run this monthly to establish a current-state baseline

# Get all mailboxes (for large tenants, use -ResultSize parameter in batches)
$mailboxes = Get-Mailbox -ResultSize Unlimited | Select-Object UserPrincipalName, DisplayName, Department

# For each mailbox, retrieve inbox rules that forward externally
$results = foreach ($mbx in $mailboxes) {
    $rules = Get-InboxRule -Mailbox $mbx.UserPrincipalName -ErrorAction SilentlyContinue |
        Where-Object {
            ($_.ForwardTo -ne $null -and $_.ForwardTo -notmatch "yourdomain\.com") -or
            ($_.RedirectTo -ne $null -and $_.RedirectTo -notmatch "yourdomain\.com") -or
            ($_.ForwardAsAttachmentTo -ne $null -and $_.ForwardAsAttachmentTo -notmatch "yourdomain\.com")
        }

    foreach ($rule in $rules) {
        [PSCustomObject]@{
            UserPrincipalName = $mbx.UserPrincipalName
            DisplayName       = $mbx.DisplayName
            Department        = $mbx.Department
            RuleName          = $rule.Name
            RuleEnabled       = $rule.Enabled
            ForwardTo         = ($rule.ForwardTo -join "; ")
            RedirectTo        = ($rule.RedirectTo -join "; ")
            ForwardAsAttach   = ($rule.ForwardAsAttachmentTo -join "; ")
            DeleteMessage     = $rule.DeleteMessage
            StopProcessing    = $rule.StopProcessingRules
        }
    }
}

# Output results — export for review or comparison with prior month
$results | Export-Csv -Path "C:\Hunting\ExternalForwardingRules_$(Get-Date -Format 'yyyyMMdd').csv" -NoTypeInformation
$results | Where-Object { $_.Department -in @("Finance", "Legal", "HR", "Executive") } |
    Select-Object UserPrincipalName, Department, RuleName, ForwardTo, RedirectTo |
    Format-Table -AutoSize
```

**Hunter Note:** Compare the output CSV against the prior month's snapshot using a diff tool. New rules that appear between cycles require immediate investigation. Rules with `DeleteMessage = True` are the most suspicious — they forward messages and then delete the originals, suppressing evidence.

---

## Query 3 — Post-Termination Active Forwarding Rules

Cross-references currently active forwarding rules against HR-provided list of terminated users. Identifies mailboxes where forwarding rules survived offboarding. Run in Microsoft Sentinel with a watchlist, or adapt the PowerShell above to cross-reference an HR export.

```kql
// Hunt Query 3: Forwarding rules on terminated user mailboxes
// Requires: TerminatedEmployees watchlist (Sentinel) with field: UserPrincipalName, TerminationDate

let TerminatedUsers = _GetWatchlist("TerminatedEmployees")
    | project UserPrincipalName = tolower(tostring(UserPrincipalName)),
              TerminationDate = todatetime(TerminationDate);

OfficeActivity
| where TimeGenerated > ago(90d)
// Look for any activity on terminated accounts — forwarding rules OR email access
| where Operation in (
    "New-InboxRule", "Set-InboxRule", "MailItemsAccessed",
    "MessageBind", "Add-MailboxPermission")
| extend NormalizedUser = tolower(UserId)
| join kind=inner TerminatedUsers on $left.NormalizedUser == $right.UserPrincipalName
// Flag activity that occurred AFTER termination date
| where TimeGenerated > TerminationDate
| project
    TimeGenerated,
    UserId,
    TerminationDate,
    DaysAfterTermination = datetime_diff("day", TimeGenerated, TerminationDate),
    Operation,
    ClientIP,
    Parameters
| sort by TerminationDate desc, TimeGenerated asc
```

**Expected Normal Results:** No mailbox operations on terminated accounts. Some organizations have brief grace periods for mail access; confirm policy.

**Anomalous Results:** Any operation on a terminated user's mailbox, especially `New-InboxRule` or `MailItemsAccessed`. Forwarding rules surviving offboarding indicate a gap in the identity lifecycle management process.

---

## Query 4 — Delegate Permissions Granted to External Users

Identifies `Add-MailboxPermission` operations where a delegate was granted FullAccess or SendAs to a mailbox, with the delegate being an external account. External delegates have full visibility into the mailbox and can read, send, and delete items.

```kql
// Hunt Query 4: External mailbox delegate permission grants
// Source: OfficeActivity

OfficeActivity
| where TimeGenerated > ago(90d)
| where Operation == "Add-MailboxPermission"
| extend Params = parse_json(Parameters)
| mv-expand Param = Params
| where Param.Name == "AccessRights"
| extend AccessRights = tostring(Param.Value)
| where AccessRights has_any ("FullAccess", "SendAs", "SendOnBehalf")
// Re-join to get the user being granted access
| join kind=inner (
    OfficeActivity
    | where Operation == "Add-MailboxPermission"
    | extend Params2 = parse_json(Parameters)
    | mv-expand Param2 = Params2
    | where Param2.Name == "User"
    | extend DelegateUser = tostring(Param2.Value)
    | project OperationId, DelegateUser
) on OperationId
| where DelegateUser !endswith "@yourdomain.com"
      and DelegateUser !endswith "@yourdomain.onmicrosoft.com"
      and DelegateUser !contains "NT AUTHORITY"
      and DelegateUser !contains "S-1-5"
| project
    TimeGenerated,
    GrantedBy = UserId,
    MailboxAffected = OfficeObjectId,
    DelegateUser,
    AccessRights,
    ClientIP
| sort by TimeGenerated desc
```

---

## Indicators Table

When reviewing hunt findings, classify findings using the following indicators:

| Indicator | Severity | Investigative Action |
|-----------|----------|---------------------|
| Forwarding to free webmail (gmail.com, protonmail.com) | High | Immediately verify with user; disable rule pending investigation |
| Forwarding rule with `DeleteMessage = True` | Critical | Treat as active exfiltration channel; escalate to IR |
| Rule active on terminated user mailbox | High | Disable rule; escalate to IR; assess data volume forwarded |
| Rule targeting a domain associated with a competitor | Critical | Escalate to IR and Legal |
| Delegate access granted to external account | High | Verify with mailbox owner; check for data exposure |
| Forwarding rule created outside business hours | Medium | Correlate with other IOBs; verify with user |
| Rule covers high-value subjects only (e.g., filter for "confidential" or "board") | Critical | Targeted exfiltration; escalate to IR and Legal |

---

## Escalation Criteria

Escalate to Tier 2 or incident response if any of the following are true:

- A forwarding rule targets a free webmail domain AND the user has access to sensitive data (Finance, Legal, M&A, HR)
- A forwarding rule includes `DeleteMessage = True` or `MarkAsRead = True` (evidence suppression)
- A forwarding rule is active on a terminated employee's mailbox
- A forwarding rule has been active for more than 30 days without any prior SOC review
- Multiple forwarding rules are found for the same user across different mail clients (Outlook, OWA, mobile)
- A forwarding rule targets a domain that was registered within the past 90 days (new domain = potential attacker-controlled domain)

---

## Hunt Cycle Log

| Date | Analyst | Scope | Findings | Disposition | Follow-Up |
|------|---------|-------|----------|-------------|-----------|
| (first run) | | | | | |

---

## Linked Artifacts

| Type | ID | File |
|------|----|------|
| Scenario | DE-002 | scenarios/data-exfiltration/DE-002-email-forwarding.md |
| Validation Test Case | TC-DE-002 | validation/test-cases/TC-DE-002.md (create) |
| Detection | DET-DE-002 | detections/ (create) |
| Playbook | PB-DE-002 | playbooks/examples/ (create) |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-26 | SIDTVF Core Team | Initial creation |
