# Scenario Template: Credential Abuse Vulnerability Class

> Pre-populated for scenarios where the threat behavior centers on credential misuse —
> including shared credentials, stale accounts, credential theft, MFA bypass, or
> using valid credentials to access systems beyond their intended scope.
> Copy to the appropriate category directory, rename, and fill in `[PLACEHOLDER]` fields.

---

## Scenario Card

| Field | Value |
|-------|-------|
| **Scenario ID** | [e.g., AC-001] |
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
| **Vulnerability Class** | Credential Abuse |
| **Credential Type** | [Domain / Local / Cloud IAM / Service Account / Shared / API Key] |

---

## 1. Description

[Describe how credentials are misused. Specify whether credentials are stolen, shared, reused, or used beyond their intended scope. Clarify whether the actor is the legitimate credential owner or is using another's credentials.]

---

## 2. Narrative Case Study

[3–6 paragraphs. Describe how the credential weakness was exploited, what systems were accessed, and whether the legitimate credential owner was aware.]

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | [MAL / NEG / COM / TP] |
| **Typical Role** | [IT Admin, Help Desk, Developer, or any user involved in credential sharing or reuse] |
| **Access Level** | [Depends on credential type — note if privileged credentials are involved] |
| **Technical Sophistication** | [Low (shared creds require no skill) to High (credential harvesting requires tooling)] |
| **Motivation** | [Access to data or systems beyond authorization / covering tracks / coercion] |
| **Credential Knowledge Used** | [e.g., "Knows where shared passwords are stored in shared document"; "Has access to PAM system and can retrieve credentials"] |

---

## 4. Attack Phases

### Phase 1 — Credential Acquisition
**Description:** Actor obtains the credential they will misuse — either by accessing a shared credential store, harvesting from an endpoint, exploiting MFA weakness, or using credentials passed to them by another user.
**Observable Actions:**
- Access to shared credential document (SharePoint, wiki, shared drive)
- PAM system query for a password outside the user's authorized role
- MFA push request generated at an unusual time (MFA fatigue variant)
- Credential harvesting tool executed on endpoint (Mimikatz-like behavior)
- Help desk account reset request not initiated by the legitimate account owner

### Phase 2 — Authentication with Acquired Credential
**Description:** Actor authenticates using the acquired credential, often appearing as a legitimate user.
**Observable Actions:**
- Login from a workstation not historically associated with the credential (new device flag)
- Login from a geographic location inconsistent with the legitimate account holder
- Simultaneous sessions on the same account from different IPs (impossible concurrence)
- Service account authenticated interactively (service accounts should not have interactive logins)
- Login at a time outside the credential owner's established working hours

### Phase 3 — [Unauthorized Access and Action]
**Description:** [What the actor does with the acquired access — access data, modify settings, exfiltrate, etc.]
**Observable Actions:**
- [System access outside the credential's intended scope]
- [Data access or system action]

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs — Credential-Specific

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Login from a device not in the user's 90-day device history | SIEM (auth logs + device fingerprint) | High |
| Login from a geographic location inconsistent with registered location | SIEM + geolocation | High |
| Impossible travel: login from two locations > 500 km apart within 2 hours | SIEM + geolocation | High |
| MFA push approved after multiple consecutive denials (MFA fatigue) | MFA platform audit log | High |
| Service account authenticated interactively (not as a service) | SIEM (logon type 2 or 10 for service accounts) | High |
| Shared account used from a workstation not associated with any of the known users | SIEM | High |
| Account login immediately followed by high-volume data access (robotic speed) | SIEM + UEBA (velocity detection) | High |
| Password reset request without a preceding self-service or ticket | ITSM + identity audit log | Medium |
| Account with no MFA enrolled used to access sensitive systems | IAM audit | Medium |
| API key used from an IP outside the application's expected IP range | SIEM + API gateway logs | High |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Shared credential identified in an access review (multiple users on one account) | IAM / access review | Hygiene gap that enables this class |
| Stale account not reviewed in 90+ days still has active authentication | IAM / identity governance | Remove or disable immediately |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Confidence |
|-------|-----------------------|---------------------|-----------|
| Phase 1 | PAM query for credentials outside role scope | PAM audit log | High |
| Phase 1 | Access to shared credential document | SharePoint/DLP file access logs | Medium |
| Phase 2 | New device or new IP authentication | SIEM (auth logs) + UEBA | High |
| Phase 2 | Impossible travel detection | SIEM + geolocation | High |
| Phase 2 | Service account interactive login | SIEM (Windows Event 4624, LogonType 2) | High |
| Phase 3 | [Scenario-specific detection] | | |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Credential Access | T1078 | Valid Accounts | Phase 2 |
| Credential Access | T1621 | Multi-Factor Authentication Request Generation | Phase 1 (MFA fatigue) |
| Credential Access | T1552.001 | Unsecured Credentials: Credentials in Files | Phase 1 (shared doc) |
| Credential Access | T1003 | OS Credential Dumping | Phase 1 (harvesting variant) |
| Defense Evasion | T1078.002 | Valid Accounts: Domain Accounts | Phase 2 |
| [Add additional techniques] | | | |

---

## 8. Legal and Privacy Considerations

- **COM archetype consideration:** If credential abuse is consistent with an external compromise rather than insider malice, treat the legitimate account holder as a victim. Notify via out-of-band channel before any account action.
- **MFA fatigue context:** MFA fatigue attacks suggest a targeted external threat actor, not just an internal actor. Consider concurrent threat intel investigation alongside the insider investigation.
- **Shared account attribution:** Authentication from a shared account cannot be attributed to a specific individual without additional corroborating evidence (endpoint login, badge access, camera). Work with Legal before any formal action based solely on shared account logs.
- **Service account monitoring:** Service account monitoring may surface legitimate automation activity. Ensure exclusions for known automation pipelines are documented before deployment.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Force password reset on the misused credential | IAM |
| 2 | Revoke all active sessions on the misused credential | IAM |
| 3 | If COM suspected: notify legitimate account holder via out-of-band channel | SOC |
| 4 | Preserve authentication logs | IR |
| 5 | Identify and remediate the credential sharing or hygiene gap that enabled this | IAM + Security |
| 6 | Notify Legal | Legal |

**Linked Playbook:** [PB-XX-NNN — create using `playbooks/_template.md`]

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | [YYYY-MM-DD] | [Author] | Initial creation |
