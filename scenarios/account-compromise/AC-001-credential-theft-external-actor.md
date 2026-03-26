# Scenario Card: AC-001 — Insider Account Compromise by External Actor

| Field | Value |
|-------|-------|
| **Scenario ID** | AC-001 |
| **Name** | Insider Account Compromise by External Actor |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-26 |
| **Last Updated** | 2026-03-26 |
| **Author** | SIDTVF Core Team |
| **Category** | Account Compromise |
| **Category Code** | AC |
| **Actor Archetype(s)** | COM |
| **Severity** | Critical |
| **Minimum Maturity Level** | 2 |

---

## 1. Description

An external threat actor obtains valid credentials for a legitimate organizational user and uses those credentials to access systems, exfiltrate data, or establish persistence — appearing in logs as the compromised insider. The legitimate account holder is typically unaware their credentials have been stolen. This scenario is distinguished from malicious insider activity by the behavioral context surrounding the authentication: impossible travel, new device, unusual hours, and activity speed inconsistent with human behavior. Early identification of the COM archetype is critical to avoid treating a victim as a suspect.

---

## 2. Narrative Case Study

Marcus was a financial analyst at a publicly traded manufacturing company. He had been with the firm for three years, was well-regarded, and had standard access to the firm's financial reporting system, SharePoint, and a read-only connection to the ERP system.

Marcus received an email that appeared to come from the firm's IT department, informing him that his Microsoft 365 account was at risk of being locked due to suspicious activity and asking him to verify his credentials via a linked page. The page was a convincing replica of the Microsoft 365 sign-in portal. Marcus entered his username and password. The page then prompted for his MFA code, which Marcus entered, believing he was completing a security verification. In doing so, he handed the attacker a valid session token.

Within 14 minutes of Marcus entering his credentials, an authentication event was logged from an IP address in Eastern Europe — while Marcus was at his desk in the firm's Chicago office. The attacker, now authenticated as Marcus, accessed SharePoint and began downloading documents from the firm's Investor Relations folder — materials not yet publicly disclosed.

The attacker downloaded 43 documents in 22 minutes at a speed no human could achieve through a web browser. They then attempted to access the ERP system but were blocked by a network segmentation rule. The session ended with no further activity.

The attack was detected during a routine Sentinel query review the following morning — a threat hunter running the HYP-AC-001 hypothesis query identified the impossible travel flag (Chicago + Eastern Europe within 14 minutes) and the velocity anomaly (43 downloads in 22 minutes vs. Marcus's typical 2–3 downloads per session). The investigation confirmed Marcus was a victim. His credentials were reset, his session was revoked, and a full review of the downloaded documents confirmed they included material non-public information (MNPI), triggering a securities disclosure review.

Marcus had not been briefed on phishing-resistant MFA (passkeys or FIDO2 hardware keys). After the incident, the firm deployed hardware security keys for all employees with access to financial reporting systems.

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | COM (Compromised Insider) |
| **Legitimate User's Role** | Any — finance, operations, engineering, HR; roles with access to high-value data are higher-risk targets |
| **Access Level** | Standard User (attacker inherits whatever the legitimate user has) |
| **External Actor Sophistication** | Medium to High — credential phishing and session hijacking are well-established techniques |
| **External Actor Motivation** | Data theft (financial, strategic), espionage, ransomware staging, MNPI for insider trading |
| **Why This Account Was Targeted** | Known access level (from LinkedIn or OSINT); department (finance, HR, legal are common targets); phishing campaign (untargeted) |

---

## 4. Attack Phases

### Phase 1 — Credential Acquisition (External)
**Description:** External actor obtains valid credentials through phishing, credential stuffing, malware, or purchasing from a credential marketplace.
**Observable Actions:**
- Phishing email delivered to the user (email gateway alert, if filtered)
- User navigates to a lookalike domain (proxy logs)
- MFA push request generated from an unexpected IP (MFA platform log)
- User approves MFA prompt from an unexpected device or location
- Credential stuffing attempts against the identity provider (repeated failed logins from external IP, then a success)

### Phase 2 — Initial Access
**Description:** External actor authenticates using the compromised credential and establishes an active session.
**Observable Actions:**
- Successful authentication from a new IP address or geographic location not in the user's history
- Impossible travel: concurrent or near-concurrent sessions from two geographically distant locations
- New device fingerprint (browser, OS, user-agent string different from user's normal profile)
- Session start time outside the user's established working hours

### Phase 3 — Rapid Data Collection
**Description:** External actor quickly accesses and collects target data. Robotic activity speed is a distinguishing signal.
**Observable Actions:**
- File download velocity far exceeding any human-possible rate (e.g., 40 files in 5 minutes)
- Navigation directly to high-value directories without browsing (suggesting prior reconnaissance or a targeted attack)
- API calls in a sequence consistent with automation, not human navigation
- Download of files not in the user's recent access history

### Phase 4 — Exfiltration and Session Termination
**Description:** Collected data is exfiltrated (often through the already-established session, as the attacker is operating within a legitimate application) and the session ends.
**Observable Actions:**
- Large data download via the authenticated session
- Session ends cleanly (logout or timeout)
- No further login attempts after the session — attacker has what they came for

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Impossible travel: two successful logins from locations > 500 km apart within 2 hours | SIEM (auth logs + geolocation) | High |
| Login from a country or region not in the user's 90-day login history | SIEM + geolocation | High |
| New device fingerprint (browser, OS, user-agent) with no prior history for this user | Entra ID sign-in logs / SIEM | High |
| MFA approval from a location inconsistent with the user's current location | MFA platform + geolocation | High |
| Activity velocity anomaly: >10 file downloads or API calls per minute (robotic speed) | SIEM + UEBA | High |
| Login to high-sensitivity application immediately following initial authentication (no warm-up behavior) | SIEM (session timeline) | Medium |
| User-agent string inconsistent with the user's standard browser/OS combination | Auth logs + SIEM | High |
| Successful login following a burst of failed password attempts from the same IP | SIEM (auth logs) | High |
| OAuth token or refresh token from a new client application the user has never authorized | Entra ID / IdP logs | High |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| User recently reported clicking a suspicious link | Help desk / user self-report | Immediate indicator — trigger out-of-band verification |
| User's email address appeared in a credential breach dump (Have I Been Pwned or similar) | Threat intel feed | Medium — flag for proactive MFA review |
| User has no MFA enrolled or uses SMS-based MFA (lower phishing resistance) | IAM / MFA enrollment report | Background risk context |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Confidence |
|-------|-----------------------|---------------------|-----------|
| Phase 1 | Lookalike domain navigation from corporate device | Web proxy + threat intel feed | Medium |
| Phase 1 | MFA push from unexpected location | MFA platform audit log | High |
| Phase 2 | Impossible travel detection | SIEM + geolocation | High |
| Phase 2 | New device / new IP login | SIEM + UEBA (device baseline) | High |
| Phase 3 | Activity velocity anomaly (files/minute) | SIEM + UEBA | High |
| Phase 3 | Access to sensitive directories not in user's recent history | SIEM + file access logs | High |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Initial Access | T1566.002 | Phishing: Spearphishing Link | Phase 1 |
| Credential Access | T1621 | Multi-Factor Authentication Request Generation | Phase 1 (MFA fatigue/bypass) |
| Credential Access | T1539 | Steal Web Session Cookie | Phase 1 (session hijacking variant) |
| Initial Access | T1078 | Valid Accounts | Phase 2 |
| Collection | T1213 | Data from Information Repositories | Phase 3 |
| Collection | T1530 | Data from Cloud Storage Object | Phase 3 |
| Exfiltration | T1041 | Exfiltration Over C2 Channel | Phase 4 |

---

## 8. Legal and Privacy Considerations

- **The legitimate user is a victim:** Do not treat the account holder as a suspect until evidence of collusion is found. Notify them via out-of-band channel (phone, in-person) before taking any action on their account.
- **Out-of-band notification:** Do not use corporate email or Teams to notify the user — the attacker may still have access to those channels. Use phone or in-person communication.
- **MNPI / securities law:** If the attacker accessed material non-public financial information, securities disclosure counsel must be engaged immediately. The SEC expects timely notification of material cybersecurity incidents.
- **Law enforcement:** Credential theft and unauthorized computer access by an external actor may warrant FBI Cyber Division referral (CFAA) or local law enforcement engagement. Coordinate timing with Legal.
- **Incident notification:** Many cyber insurance policies require notification within 24–72 hours of a confirmed breach. Check policy terms immediately.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Revoke all active sessions for the compromised account | IAM |
| 2 | Force immediate password and MFA credential reset | IAM |
| 3 | Notify the legitimate account holder via phone or in-person — they are a victim | SOC / Manager |
| 4 | Preserve all authentication and activity logs for the session | IR |
| 5 | Identify the full scope: what was accessed, downloaded, or modified | IR |
| 6 | Notify Legal immediately — assess securities, breach notification, and law enforcement obligations | Legal |
| 7 | Scan the user's device for credential-harvesting malware | IR / Endpoint |
| 8 | Enroll the user in phishing-resistant MFA (FIDO2/passkey) before re-enabling access | IAM |

**Linked Playbook:** PB-AC-001 (create using `playbooks/_template.md`)

---

## 10. Linked Artifacts

| Artifact Type | ID | File |
|---------------|----|------|
| Validation Test Case | TC-AC-001 | validation/test-cases/TC-AC-001.md (create) |
| Detection | DET-AC-001 | detections/ (create) |
| Hunt Hypothesis | HYP-AC-001 | hunting/hypotheses/HYP-AC-001.md (create) |

---

## 11. Related Scenarios

| Scenario ID | Name | Relationship |
|-------------|------|-------------|
| UA-001 | After-Hours Unauthorized System Access | COM scenario often presents as after-hours access anomaly |
| DE-001 | Cloud Storage Upload Exfiltration | Exfiltration channel may be used within the compromised session |
| PV-001 | Shadow IT Data Storage | Personal cloud accounts used for shadow IT are a phishing attack surface |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-26 | SIDTVF Core Team | Initial creation |
