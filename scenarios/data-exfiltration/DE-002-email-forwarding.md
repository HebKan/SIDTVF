# Scenario Card: DE-002 — Email Auto-Forwarding Rule

| Field | Value |
|-------|-------|
| **Scenario ID** | DE-002 |
| **Name** | Email Auto-Forwarding Rule |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-25 |
| **Last Updated** | 2026-03-25 |
| **Author** | SIDTVF Core Team |
| **Category** | Data Exfiltration |
| **Category Code** | DE |
| **Actor Archetype(s)** | MAL, NEG |
| **Severity** | High |
| **Minimum Maturity Level** | 2 |

---

## 1. Description

An insider creates an auto-forwarding rule in their corporate email client that silently copies or redirects all incoming and outgoing email — or emails matching defined criteria — to a personal email address or external account. This creates a persistent, passive exfiltration channel that continues to operate without further action by the insider, often surviving password resets and extending beyond the employment period if not explicitly disabled.

---

## 2. Narrative Case Study

Elena was a Regional Director of Sales at a pharmaceutical distribution company. Six months before her contract renewal, she became convinced her manager intended to replace her and began quietly planning her next move. She had spent five years building relationships with the company's top 200 healthcare provider accounts and knew that her contact book and account intelligence were her most valuable asset when moving to a competitor.

Over a single lunch break, Elena opened her corporate Outlook Web App, navigated to the email rules settings, and created a forwarding rule: all emails from any sender matching a list of 23 key account domains would be automatically forwarded to her personal Gmail address. She named the rule "Filter Junk" and added the rule alongside two legitimate junk mail filters to obscure it among real rules.

Over the following four months, approximately 1,400 emails were silently forwarded to Elena's personal inbox. These emails included contract renewal discussions, pricing concessions, account notes, and contact details for key decision-makers. The company's email gateway logged the forwarding activity, but it was not monitored.

When Elena resigned and joined a competitor three months later, the company noticed she had immediately contacted several top accounts. A routine email audit, conducted only after the competitor's surprisingly rapid account conversions triggered suspicion, discovered the forwarding rule — which was still active on her now-disabled corporate account (the account had been disabled but the mailbox had not been purged).

The auto-forwarding rule had persisted for seven months. Had the email system's administrative audit log been reviewed at departure, the rule would have been discovered immediately.

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | MAL (primary), NEG (secondary — employees sometimes create forwarding rules without realizing the security implications) |
| **Typical Role** | Sales, business development, legal, executive, HR, finance — roles where email contains high-value relationship or confidential data |
| **Access Level** | Standard User |
| **Technical Sophistication** | Low — creating an email forwarding rule requires no technical expertise |
| **Motivation** | MAL: competitive advantage at a new employer, client relationship theft, intelligence gathering. NEG: working from personal device, convenience |
| **Insider Knowledge Used** | Knowledge of which accounts or topics carry the most valuable business intelligence; knowledge of how to navigate email client settings |

---

## 4. Attack Phases

### Phase 1 — Rule Creation

**Description:** The insider accesses email client settings and creates a forwarding or redirect rule. This is a one-time action that takes under two minutes and leaves a minimal footprint.

**Observable Actions:**
- Access to email settings page (OWA, Outlook client, Gmail settings)
- Email rule creation event logged in Exchange audit log, Microsoft 365 Unified Audit Log, or Google Workspace Admin Log
- New inbox rule created with external forwarding destination

### Phase 2 — Passive Data Collection

**Description:** The rule runs automatically, forwarding copies of matching emails to the external destination. The insider takes no further action. This phase may continue for months or years without any active behavior by the insider.

**Observable Actions:**
- Email forwarding events logged in mail transport logs
- Emails leaving the organization to a personal email domain (gmail.com, yahoo.com, hotmail.com, protonmail.com, etc.)
- Volume of external mail may be elevated if all email is forwarded
- Forwarded emails have the same subject line as the originals — visible in transport logs

### Phase 3 — Optional Scope Expansion

**Description:** Sophisticated actors may modify the rule to capture additional email categories over time, especially as a separation date approaches.

**Observable Actions:**
- Modification of existing email rule parameters
- New rules added to broaden capture criteria
- Sudden increase in forwarding volume

### Phase 4 — Persistence Beyond Employment

**Description:** If the forwarding rule is not explicitly removed during the offboarding process, it may continue to forward new email if the account is only disabled (not purged) or if a mail contact is left behind.

**Observable Actions:**
- Forwarding events from a disabled account's mailbox
- Forwarded emails dated after the employee's last day

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Email forwarding rule created with external destination | Exchange/M365 Unified Audit Log (New-InboxRule operation), Google Workspace Admin Log | High |
| Inbox rule with forwarding action pointing to non-corporate domain | Exchange PowerShell, M365 Compliance Center | High |
| Email forwarding to a free email provider domain (gmail, yahoo, hotmail, protonmail) | Mail transport logs, SIEM | High |
| Bulk email leaving the organization from a single user mailbox | Mail transport logs | Medium |
| Forwarding rule named to blend with legitimate rules ("Filter," "Junk," "Archive") | Exchange audit log | Medium |
| Forwarding active on an account that has been disabled or after last day of employment | Exchange / M365 admin logs | High |
| User-created mail contact that enables forwarding to external address | Exchange admin audit log | Medium |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Employee is leaving for a competitor | HR / Management | Strong risk multiplier — trigger immediate email rule audit |
| Employee has client-facing or commercially sensitive role | HR role classification | Increases impact risk |
| Employee recently accessed competitor or job-search websites | Web proxy logs | Low-weight; cannot be acted on alone |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Detection Confidence |
|-------|-----------------------|---------------------|---------------------|
| Phase 1 — Rule Creation | Alert on creation of any inbox rule with external forwarding destination | M365 Unified Audit Log / Google Workspace Admin Log | High |
| Phase 1 — Rule Creation | Periodic audit of all mailboxes for forwarding rules pointing to external domains | Exchange PowerShell / Admin Center | High |
| Phase 2 — Collection | Alert on sustained email forwarding from single mailbox to external domain | Mail transport logs / SIEM | High |
| Phase 2 — Collection | Daily summary of external forwarding volume per mailbox | SIEM / mail gateway | Medium |
| Phase 4 — Persistence | Post-offboarding mailbox audit for active forwarding rules before account purge | Offboarding checklist + Exchange admin | High |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Collection | T1114.003 | Email Collection: Email Forwarding Rule | Phase 1, Phase 2 |
| Exfiltration | T1048.003 | Exfiltration Over Alternative Protocol: Exfiltration Over Unencrypted Non-C2 Protocol | Phase 2 |
| Persistence | T1098.003 | Account Manipulation: Additional Email Delegate Permissions | Phase 3 (if delegate access used instead of rule) |

---

## 8. Legal and Privacy Considerations

- **Audit log availability:** Detection depends on email audit logging being enabled. In Microsoft 365, mailbox audit logging must be explicitly enabled and retained for at least 90 days. Default retention may be insufficient for investigations. Verify retention settings before relying on this data source.
- **Personal email content:** Do not attempt to access or subpoena the contents of the personal email account to which data was forwarded without a legal hold order or law enforcement involvement.
- **Rule review scope:** Periodic audits of all mailbox rules are administratively permissible because they involve administrative review of system configuration, not reading email content. This distinction is important legally and should be documented.
- **Offboarding:** Forwarding rule removal should be a standard item in the employee offboarding checklist. Failure to complete offboarding properly may be a policy or compliance finding independent of any malicious intent.
- **Negligent forwarding:** If the actor was NEG (e.g., set up forwarding to work from home without realizing it was prohibited), HR response replaces or precedes legal action. Ensure investigation distinguishes intent before escalating.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Disable the forwarding rule immediately upon discovery | IT / Exchange Admin |
| 2 | Preserve the audit log showing rule creation date, creator, and rule criteria | IR |
| 3 | Identify all emails forwarded (date range, count, subject lines) | IR / Legal |
| 4 | Assess the sensitivity of forwarded content to determine impact severity | IR + Business Unit |
| 5 | Notify Legal — determine whether trade secret, PII breach, or other legal exposure exists | Legal |
| 6 | Engage HR for employment action if insider is still employed | Legal + HR |
| 7 | Assess breach notification obligations if customer or employee PII was forwarded | Legal / Privacy |
| 8 | Conduct full mailbox audit for additional rules or anomalies | IR |

**Linked Playbook:** PB-DE-002 (not yet created — use `playbooks/_template.md`)

---

## 10. Linked Artifacts

| Artifact Type | ID | File |
|---------------|----|------|
| Validation Test Case | TC-DE-002 | validation/test-cases/TC-DE-002.md (create using template) |
| Detection (Sigma) | DET-DE-002-SIGMA | detections/sigma/DE-002-email-forwarding.yml (create using template) |
| Hunt Hypothesis | HYP-DE-002 | hunting/hypotheses/HYP-DE-002.md (create using template) |

---

## 11. Related Scenarios

| Scenario ID | Name | Relationship |
|-------------|------|-------------|
| DE-001 | Cloud Storage Upload Exfiltration | Alternative exfiltration channel; may be used simultaneously |
| IP-001 | Intellectual Property Theft (when created) | Email forwarding is a common IP theft vector |

---

## 12. References

- Microsoft — Find and Investigate Malicious Email Rules
- MITRE ATT&CK — T1114.003: Email Forwarding Rule
- CISA Alert AA20-296A — Email Forwarding Rules Abuse

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-25 | SIDTVF Core Team | Initial creation |
