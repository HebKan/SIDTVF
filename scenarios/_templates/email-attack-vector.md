# Scenario Template: Email Attack Vector

> Pre-populated for scenarios where the primary threat behavior occurs via email —
> including forwarding rules, bulk export, recipient manipulation, or phishing facilitation.
> Copy to the appropriate category directory, rename, and fill in `[PLACEHOLDER]` fields.

---

## Scenario Card

| Field | Value |
|-------|-------|
| **Scenario ID** | [e.g., DE-005] |
| **Name** | [Short name] |
| **Version** | 1.0 |
| **Status** | Draft |
| **Created** | [YYYY-MM-DD] |
| **Last Updated** | [YYYY-MM-DD] |
| **Author** | [Name] |
| **Category** | [Category] |
| **Actor Archetype(s)** | [MAL / NEG / COM / TP] |
| **Severity** | [Critical / High / Medium / Low] |
| **Minimum Maturity Level** | 2 |
| **Attack Vector** | Email |
| **Email Platform** | [Microsoft 365 / Google Workspace / On-premises Exchange / Other] |

---

## 1. Description

[Describe the email-based threat behavior. Specify: what the insider does with email (forwarding, bulk export, rule creation, recipient manipulation), what data is exposed, and to where.]

---

## 2. Narrative Case Study

[3–6 paragraphs. Include how the insider used email platform features, how the behavior persisted over time without detection, and how it was eventually discovered.]

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | [MAL / NEG / COM / TP] |
| **Typical Role** | [Roles where email contains high-value content: Sales, Legal, Finance, HR, Executive] |
| **Access Level** | Standard User |
| **Technical Sophistication** | Low (email rule creation requires no technical skill) |
| **Motivation** | [Financial gain / competitive intelligence / negligent convenience] |
| **Email Knowledge Used** | [e.g., "Knows how to create inbox rules"; "Aware that forwarding rules survive account suspension"] |

---

## 4. Attack Phases

### Phase 1 — Mail Rule or Export Configuration
**Description:** Insider creates a forwarding rule, export task, or delegation setting to capture ongoing email traffic.
**Observable Actions:**
- New-InboxRule operation in Microsoft 365 Unified Audit Log with external ForwardTo destination
- Gmail filter created with forwarding to external address
- Exchange mailbox delegate permission granted to external address
- eDiscovery export job created by non-administrator account

### Phase 2 — Passive Email Capture
**Description:** The configured rule runs automatically; the insider takes no further action.
**Observable Actions:**
- Mail transport logs showing messages forwarded to external domain
- Forwarding volume matching the rule's criteria over days or weeks
- Email departing the organization to a free email provider (gmail.com, yahoo.com, protonmail.com)

### Phase 3 — [Scenario-Specific Phase]
**Description:** [How the captured email is used or the behavior escalates]
**Observable Actions:**
- [Observable action]

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs — Email-Specific

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Inbox rule created with external forwarding destination | M365 Unified Audit Log (New-InboxRule), Google Workspace Admin Log | High |
| Inbox rule ForwardTo points to a free email provider domain | Exchange audit / M365 compliance | High |
| Rule name designed to blend in ("Filter", "Archive", "Junk") | Exchange audit log | Medium |
| Mail transport log shows forwarding to external domain not on approved list | Mail transport logs / SIEM | High |
| Forwarding rule still active on a disabled account (post-termination) | Exchange admin audit | High |
| eDiscovery export or content search run by non-administrator | M365 Unified Audit Log | High |
| Mailbox delegate permission granted to external account | Exchange admin audit | High |
| Email volume to external domain significantly above user's baseline | SIEM + mail transport logs | Medium |
| Email to competitor domain outside normal business context | Mail transport logs + domain classification | Medium |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Departing employee with client-facing or commercially sensitive role | HR | Critical risk multiplier |
| Employee with access to M&A, legal, or financial email content | HR role classification | Increases impact risk |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Confidence |
|-------|-----------------------|---------------------|-----------|
| Phase 1 | Alert on any inbox rule creation with external ForwardTo | M365 Unified Audit Log / Google Workspace Admin | High |
| Phase 1 | Periodic full audit of all mailboxes for external forwarding rules | Exchange PowerShell (Get-InboxRule) scheduled task | High |
| Phase 2 | Alert on sustained forwarding to free email domain from single mailbox | Mail transport logs + SIEM | High |
| Phase 2 | Post-termination mailbox audit before account purge | Offboarding checklist + Exchange admin | High |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Collection | T1114.003 | Email Collection: Email Forwarding Rule | Phase 1, Phase 2 |
| Exfiltration | T1048 | Exfiltration Over Alternative Protocol | Phase 2 |
| Collection | T1114.001 | Email Collection: Local Email Collection | (if export-based variant) |
| Persistence | T1098.003 | Account Manipulation: Additional Email Delegate Permissions | Phase 1 (delegate variant) |

---

## 8. Legal and Privacy Considerations

- **Audit log retention:** M365 mailbox audit logging must be explicitly enabled and retained ≥90 days. Verify before relying on this data source for any investigation.
- **Rule review vs. content review:** Reviewing the existence and configuration of mail rules is an administrative action (reviewing system configuration) and does not constitute reading email content. This distinction matters legally.
- **Personal email content:** Do not attempt to access the external personal account to which data was forwarded without legal process.
- **Offboarding:** Forwarding rule removal must be a formal checklist item at termination. Failure to complete this creates both security and compliance exposure.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Disable the forwarding rule immediately | IT / Exchange Admin |
| 2 | Preserve audit log showing rule creation (date, creator, criteria) | IR |
| 3 | Enumerate all forwarded email (date range, subjects, count) | IR / Legal |
| 4 | Assess data sensitivity — what was in the forwarded email? | IR + Business Unit |
| 5 | Notify Legal — determine breach or trade secret exposure | Legal |

**Linked Playbook:** [PB-XX-NNN — create using `playbooks/_template.md`]

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | [YYYY-MM-DD] | [Author] | Initial creation |
