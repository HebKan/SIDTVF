# Scenario Template: Microsoft 365 Application

> Pre-populated for scenarios where the threat behavior occurs within the Microsoft 365
> ecosystem — including Exchange Online, SharePoint, OneDrive for Business, Teams, and
> Entra ID (Azure AD). Assumes M365 E3/E5 licensing with Unified Audit Log enabled.
> Copy to the appropriate category directory, rename, and fill in `[PLACEHOLDER]` fields.

---

## Scenario Card

| Field | Value |
|-------|-------|
| **Scenario ID** | [e.g., DE-007] |
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
| **Attack Vector** | Application — Microsoft 365 |
| **M365 Services Involved** | [Exchange / SharePoint / OneDrive / Teams / Entra ID / Purview / Other] |

---

## 1. Description

[Describe the threat behavior within the M365 ecosystem. Specify which M365 service is being abused and what data or functionality is targeted.]

---

## 2. Narrative Case Study

[3–6 paragraphs. Include which M365 features the insider used, why standard M365 security controls did not catch it, and how the Unified Audit Log or another M365 data source revealed the activity.]

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | [MAL / NEG / COM / TP] |
| **Typical Role** | [Any role with M365 access; higher risk for roles with SharePoint admin, Exchange admin, or Purview access] |
| **Access Level** | [Standard / Power User / M365 Admin] |
| **Technical Sophistication** | [Low to Medium — M365 features are user-accessible] |
| **M365 Knowledge Used** | [e.g., "Knows SharePoint permissions allow sharing externally"; "Aware that Teams chat export is not monitored"] |

---

## 4. Attack Phases

### Phase 1 — M365 Reconnaissance or Setup
**Description:** Insider identifies the M365 service or feature they will abuse, or configures a persistent access mechanism.
**Observable Actions:**
- SharePoint site browsing across departments outside user's normal access (SharePoint Unified Audit)
- External sharing enabled on a SharePoint library or OneDrive folder
- New guest user invitation to M365 tenant (Entra ID audit)
- Inbox rule creation (Exchange Unified Audit: New-InboxRule)
- eDiscovery or Content Search created by non-administrator (Purview audit)

### Phase 2 — Data Access or Collection
**Description:** Insider accesses or collects the target data within M365.
**Observable Actions:**
- SharePoint or OneDrive FileDownloaded / FileSyncDownloadedFull events at high volume
- Teams chat export or meeting recording access
- Exchange MailItemsAccessed events on mailboxes not owned by the user
- Search-Mailbox or New-ComplianceSearch executed by non-compliance account

### Phase 3 — [Exfiltration or Impact]
**Description:** [How data leaves M365 or how M365 is used to harm the organization]
**Observable Actions:**
- SharePoint/OneDrive file shared externally via anonymous link (SharingSet event)
- OneDrive sync to unmanaged personal device
- Teams message containing sensitive data sent to external federation
- Email forwarded via transport rule to external address

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs — M365-Specific

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| FileDownloaded volume significantly above user's 30-day SharePoint/OneDrive baseline | M365 Unified Audit Log → OfficeActivity (Sentinel) | High |
| External sharing link created for a document library containing sensitive content | M365 Unified Audit (SharingSet, AddedToGroup) | High |
| eDiscovery or Content Search run by non-Compliance admin | M365 Unified Audit (Purview operations) | High |
| Inbox rule with external ForwardTo address created | M365 Unified Audit (New-InboxRule) | High |
| Guest user added to M365 tenant without IT provisioning approval | Entra ID audit logs | Medium |
| MailItemsAccessed for a mailbox not owned by the authenticated user | M365 Unified Audit | High |
| OneDrive sync enabled on an unmanaged (personal) device | Entra ID sign-in logs + MDM enrollment status | High |
| Teams channel files bulk downloaded | M365 Unified Audit (Teams file events) | Medium |
| Admin consent grant to a third-party OAuth application | Entra ID audit (Consent to application) | High |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| User is a departing employee with SharePoint or Exchange admin rights | HR + M365 admin roles | Critical — review all admin role assignments at departure |
| User recently had their M365 license downgraded but retains legacy permissions | M365 admin + IAM | Signals potential stale access |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Confidence |
|-------|-----------------------|---------------------|-----------|
| Phase 1 | Alert on external sharing link creation for sensitive SharePoint content | M365 Unified Audit via Sentinel (OfficeActivity) | High |
| Phase 1 | Alert on inbox rule with external forwarding destination | M365 Unified Audit (New-InboxRule) | High |
| Phase 2 | Alert on FileDownloaded volume spike vs. 30-day baseline | OfficeActivity + UEBA | High |
| Phase 2 | Alert on eDiscovery or Content Search by non-Compliance account | Purview unified audit | High |
| Phase 3 | Alert on OneDrive sync to unmanaged device | Entra ID + Intune MDM | High |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Collection | T1114.003 | Email Collection: Email Forwarding Rule | Phase 1 (Exchange) |
| Collection | T1213.002 | Data from Information Repositories: SharePoint | Phase 2 |
| Collection | T1213.001 | Data from Information Repositories: Confluence (SharePoint analog) | Phase 2 |
| Exfiltration | T1567.002 | Exfiltration Over Web Service: Exfiltration to Cloud Storage | Phase 3 (OneDrive personal) |
| Persistence | T1098.003 | Account Manipulation: Additional Email Delegate Permissions | Phase 1 |

---

## 8. Legal and Privacy Considerations

- **M365 Unified Audit Log:** Must be explicitly enabled by an M365 admin (it is not on by default in all tenants). Verify audit log status before relying on this data source. E5 licensing provides longer retention (1 year standard, up to 10 years with add-on).
- **Teams message monitoring:** Teams chat content is generally subject to the same monitoring notice requirements as email content. Ensure acceptable use policy covers Teams communications.
- **eDiscovery and Purview:** If an investigation uses eDiscovery or Purview tools to access employee email or Teams content, ensure this access is authorized and documented — these tools are typically admin-only for good reason.
- **Guest user data:** Guest users provisioned in the M365 tenant are subject to the organization's data governance policies but may have their own GDPR/privacy rights depending on jurisdiction.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Preserve M365 Unified Audit logs for the affected user (use export or legal hold) | IR + M365 Admin |
| 2 | Revoke external sharing links and remove guest users created by the account | M365 Admin |
| 3 | Review and remove any inbox rules, OAuth app consents, or delegate permissions | M365 Admin |
| 4 | Apply a Microsoft Purview eDiscovery hold to preserve mailbox and OneDrive data | Legal + M365 Admin |
| 5 | Notify Legal — assess breach and data governance obligations | Legal |

**Required M365 Configuration for Detection:**
- Unified Audit Log: Enabled (M365 Admin Center → Compliance)
- Mailbox audit logging: Enabled for all users (default in M365 since 2019, verify for older tenants)
- Microsoft Defender for Cloud Apps (MCAS/MDCA): Connected to M365 for anomaly detection
- Microsoft Sentinel: OfficeActivity connector enabled

**Linked Playbook:** [PB-XX-NNN — create using `playbooks/_template.md`]

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | [YYYY-MM-DD] | [Author] | Initial creation |
