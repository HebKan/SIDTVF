# Scenario Card: UA-001 — After-Hours Unauthorized System Access

| Field | Value |
|-------|-------|
| **Scenario ID** | UA-001 |
| **Name** | After-Hours Unauthorized System Access |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-25 |
| **Last Updated** | 2026-03-25 |
| **Author** | SIDTVF Core Team |
| **Category** | Unauthorized Access |
| **Category Code** | UA |
| **Actor Archetype(s)** | MAL, COM |
| **Severity** | High |
| **Minimum Maturity Level** | 2 |

---

## 1. Description

An insider accesses organizational systems, applications, or data at unusual times — specifically outside their established working hours pattern — in ways that are inconsistent with their role, ongoing projects, or legitimate need. After-hours access is not inherently malicious; however, when combined with anomalous system scope, high-volume data access, or sensitive data categories, it is a strong indicator of deliberate unauthorized activity. This scenario also captures access by a compromised insider whose credentials are being used by an external actor operating in a different time zone.

---

## 2. Narrative Case Study

Priya was a senior database administrator at a global logistics company. Her standard hours were 8 AM to 6 PM in the Eastern US time zone, and her role required regular access to operational databases and configuration systems. She had administrative credentials to approximately 15 production systems.

For the first four years of her tenure, Priya's access logs showed consistent, predictable activity within her working hours. Then, over a three-month period beginning after a performance improvement plan was initiated, her access patterns changed markedly. Login events began appearing at 11 PM, 1 AM, and 2 AM — times she had never previously accessed the network.

During these after-hours sessions, Priya accessed the company's HR compensation database (not within her role scope), ran queries against the executive payroll records, and exported a report listing compensation, bonus structures, and performance ratings for the company's 340 most senior employees. She also accessed the accounts payable system three times in a single week and made changes to vendor banking records for two accounts.

When finance noticed that two vendor payments totaling $340,000 had been redirected to unknown accounts, an investigation was launched. The forensic team discovered that the after-hours access preceded the payment redirections and that the payroll data export had been uploaded to an external file-sharing service.

The investigation found that Priya's actions were a combination of financial fraud (vendor payment redirection) and data theft (executive compensation data intended to be sold to a competitor). The after-hours access pattern was the first clear indicator visible in logs — had alert rules been in place for after-hours access to sensitive systems from accounts with no prior after-hours history, the activity would have been detected within days rather than after financial loss had occurred.

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | MAL (primary), COM (secondary — compromised account used by external actor in a different time zone) |
| **Typical Role** | IT administrator, database administrator, finance, HR, any role with privileged access |
| **Access Level** | Power User to Administrator |
| **Technical Sophistication** | Medium to High (for malicious actors leveraging administrative access) |
| **Motivation** | MAL: financial fraud, data theft, sabotage, intelligence gathering. COM: external threat actor exploiting stolen credentials |
| **Insider Knowledge Used** | Knowledge of which systems are not monitored after hours; knowledge of which system configurations store the highest-value data |

---

## 4. Attack Phases

### Phase 1 — Baseline Observation

**Description:** The actor (malicious insider or external actor using compromised credentials) identifies that certain activities are less likely to be detected or interrupted after business hours. They begin shifting activity to low-visibility hours.

**Observable Actions:**
- First after-hours login from a user account with no prior after-hours history
- Login event at a time more than 2 standard deviations outside the user's historical access window
- VPN connection or authentication event at an unusual hour

### Phase 2 — System Access

**Description:** The actor accesses target systems — often systems outside their role scope or systems containing high-value data. After-hours sessions may be longer than typical daytime sessions.

**Observable Actions:**
- Authentication to systems not historically accessed by this user
- Session duration significantly longer than the user's typical sessions
- Multiple system authentications within a short window (rapid pivoting)
- Access from an unexpected geographic location (especially for COM scenario)

### Phase 3 — Data Access or System Action

**Description:** The actor takes the intended action — querying data, modifying records, exfiltrating files, making configuration changes, or staging data for later exfiltration.

**Observable Actions:**
- Database queries against sensitive tables (HR, payroll, financial, customer)
- File access to repositories outside the user's normal scope
- Configuration changes outside approved change windows
- Large data downloads or export operations
- Record modification in financial or transactional systems

### Phase 4 — Session Termination and Cover

**Description:** The actor ends the session and may attempt to minimize evidence. Some actors take no cover action, relying on the expectation that after-hours logs are not reviewed.

**Observable Actions:**
- Logout event or session timeout at an unusual hour
- Deletion of downloaded files (less common in this scenario)
- No follow-up access during regular hours that would explain the after-hours activity

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Login at a time more than 2 standard deviations outside the user's 90-day baseline working hours | SIEM (authentication logs + UEBA time-of-day baselining) | High |
| Access to systems the user has no prior authentication history for, during after-hours period | SIEM (authentication logs) | High |
| Privileged account session lasting more than 4 hours outside business hours with no on-call ticket | SIEM + change management integration | High |
| Database query against a sensitive table (HR, payroll, PII) from an account with no documented need for that data | DAM + role mapping | High |
| VPN authentication from a geographic location inconsistent with the user's registered location | VPN logs + geolocation | High (COM scenario) |
| Impossible travel: login from two geographically distant IPs within an impossible travel window | SIEM + geolocation | High (COM scenario) |
| After-hours access immediately preceding a financial transaction anomaly | SIEM + financial system logs (correlation) | High |
| Interactive login (vs. scheduled/service) at 11 PM–5 AM local time from a standard user account | Authentication logs | Medium |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Employee is on a performance improvement plan or has received negative performance feedback | HR | Medium risk multiplier |
| Employee has recently experienced a disruptive life event | HR / Management discretion | Low-weight; contextual only |
| Employee has unexplained after-hours access on multiple consecutive nights | SIEM trend | High — escalate pattern |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Detection Confidence |
|-------|-----------------------|---------------------|---------------------|
| Phase 1 — Baseline Observation | Alert on first after-hours authentication for accounts with no prior after-hours history | SIEM + UEBA time-of-day baseline | High |
| Phase 2 — System Access | Alert on authentication to sensitive systems outside business hours | SIEM (role-scoped alert) | High |
| Phase 2 — System Access | Impossible travel detection | SIEM + geolocation (COM scenario) | High |
| Phase 3 — Data Access | Database query to sensitive tables after hours | DAM | High |
| Phase 3 — Data Access | File access volume spike in an after-hours session | SIEM + file audit logs | Medium |
| Phase 3 — System Action | Configuration change outside approved change window | SIEM + change management | High |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Initial Access | T1078.002 | Valid Accounts: Domain Accounts | Phase 1, Phase 2 |
| Discovery | T1082 | System Information Discovery | Phase 2 — System Access |
| Discovery | T1087 | Account Discovery | Phase 2 — System Access |
| Collection | T1005 | Data from Local System | Phase 3 — Data Access |
| Collection | T1213 | Data from Information Repositories | Phase 3 — Data Access |
| Impact | T1565.001 | Data Manipulation: Stored Data Manipulation | Phase 3 — System Action (fraud variant) |

---

## 8. Legal and Privacy Considerations

- **Time-of-day baselining:** Behavioral baselines based on historical working hours are generally permissible under acceptable use policies. Ensure baseline methodology does not create discriminatory outcomes (e.g., flagging international employees or shift workers for after-hours work that is normal for their role).
- **Geolocation data:** Collection and use of employee geolocation data (from VPN IP, device GPS, or authentication location) may be restricted in some jurisdictions (EU GDPR, California, Germany). Consult Legal on lawful basis before implementing geolocation-based detection.
- **COM scenario — victim notification:** If after-hours access is attributed to a compromised account, the legitimate account holder is a victim and must be notified through a secure out-of-band channel (not corporate email) before their account is investigated for wrongdoing.
- **Financial fraud correlation:** If after-hours access correlates with financial fraud, preserve all evidence before any financial reversal attempts that might alter records needed for investigation.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Verify whether the activity is legitimate: check for on-call ticket, travel, approved overtime | SOC |
| 2 | If unverified, contact the account holder via out-of-band channel to confirm whether the access is theirs | SOC |
| 3 | If compromise suspected (COM scenario): revoke session, force credential reset, scan endpoint for malware | IR |
| 4 | If malicious insider suspected: preserve logs, notify Legal before account action | Legal / IR |
| 5 | Identify full scope of systems accessed and data touched in the after-hours session | IR |
| 6 | If financial system access is involved: notify Finance immediately for transaction review | IR + Finance |
| 7 | Engage HR if employment action is anticipated | Legal + HR |

**Linked Playbook:** PB-UA-001 (create using `playbooks/_template.md`)

---

## 10. Linked Artifacts

| Artifact Type | ID | File |
|---------------|----|------|
| Validation Test Case | TC-UA-001 | validation/test-cases/TC-UA-001.md (create using template) |
| Detection | DET-UA-001 | detections/ (create using template) |
| Hunt Hypothesis | HYP-UA-001 | hunting/hypotheses/HYP-UA-001.md (create using template) |

---

## 11. Related Scenarios

| Scenario ID | Name | Relationship |
|-------------|------|-------------|
| DE-001 | Cloud Storage Upload Exfiltration | After-hours access often precedes or occurs during exfiltration |
| PM-001 | Privilege Escalation via Access Abuse | After-hours sessions are used to abuse escalated access with lower detection risk |
| SB-001 | Targeted Database Deletion | Sabotage is almost always executed after hours |

---

## 12. References

- MITRE ATT&CK — T1078: Valid Accounts
- SANS — Insider Threat Behavioral Indicators
- CMU CERT — Unintentional Insider Threats
- NIST SP 800-53 — AU-6: Audit Record Review, Analysis, and Reporting

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-25 | SIDTVF Core Team | Initial creation |
