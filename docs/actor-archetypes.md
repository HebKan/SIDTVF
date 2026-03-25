# Insider Threat Actor Archetypes

> **Document Version:** 1.0 | **Status:** Active | **Last Updated:** 2026-03-25

---

## Overview

SIDTVF defines four insider threat actor archetypes. Every scenario in the framework is tagged to one or more of these archetypes. Understanding the archetype matters because it directly affects:

- The likely **motivation** and therefore the goal of the activity
- The observable **behavioral signals** that distinguish threat activity from normal work
- The appropriate **detection strategy** (behavioral analytics vs. anomaly detection vs. rule-based)
- The appropriate **response actions** (HR, legal, technical containment, law enforcement referral)

Archetypes are not mutually exclusive. A scenario may involve multiple archetypes simultaneously — for example, a malicious insider who is also a third-party contractor (MAL + TP).

---

## Summary Table

| Code | Archetype | Harm Intent | Primary Driver | Detection Focus |
|------|-----------|-------------|----------------|----------------|
| `MAL` | Malicious Insider | Intentional | Financial, revenge, ideology, coercion | Anomalous access patterns, bulk data access, exfiltration behaviors |
| `NEG` | Negligent Insider | Unintentional | Carelessness, ignorance, convenience | Policy violation patterns, misconfiguration events, phishing indicators |
| `COM` | Compromised Insider | External-driven | Credential theft, social engineering, malware | Impossible travel, unusual hours, behavioral deviation from baseline |
| `TP` | Third-Party / Contractor | Variable | Access overprovision, weak oversight, intent | Access outside scope, unusual data volumes, off-hours activity |

---

## Archetype 1 — Malicious Insider (`MAL`)

### Definition

A malicious insider is an individual with authorized access to organizational systems, data, or facilities who intentionally causes harm to the organization. The harm may be directed at data, systems, finances, or reputation. The individual understands that their actions are unauthorized or harmful.

### Common Motivations

| Motivation | Description | Example Indicator |
|------------|-------------|-------------------|
| Financial Gain | Selling proprietary data, trade secrets, or customer records to competitors or criminal actors | Unusual contact with competitors; sudden financial changes |
| Disgruntlement | Revenge against the organization, a manager, or colleagues following a perceived injustice | Termination notice; demotion; disciplinary action preceding activity |
| Ideology | Acting on political, ideological, or activist principles (e.g., leaking to press or foreign actors) | Unusual external contact; encrypted communications |
| Coercion | Forced or pressured by an external threat actor to perform actions | Behavioral changes; signs of personal stress; unusual after-hours logins |
| Competitive Advantage | Stealing data to benefit a competitor or a new employer | Job change forthcoming; communication with new employer |
| Ego / Curiosity | Accessing systems or data out of curiosity or to demonstrate capability | Access to systems far outside role scope |

### Behavioral Profile

**Pre-incident indicators (may precede harmful activity by weeks or months):**
- Expressed dissatisfaction with management, compensation, or the organization
- Looming resignation, termination, or announcement of departure
- Unusual personal financial stress (observable through behavior or HR context)
- Excessive interest in work outside their normal scope
- Requests for access that exceed job requirements

**Technical behavioral signals:**
- Bulk access to files, databases, or repositories not required for current work
- Downloads or exports significantly above personal baseline
- After-hours access to sensitive systems
- Use of personal cloud storage, personal email, or removable media
- Attempts to cover tracks (log deletion, disabling DLP agents, anti-forensic behavior)
- Access to other users' data or impersonating other accounts

### Detection Strategy

| Strategy | What to Look For |
|----------|----------------|
| Behavioral Baselining | Deviation from the individual's established access, download, and communication patterns |
| Bulk Data Access Detection | Access to unusually large numbers of files or records in a short window |
| Exfiltration Channel Monitoring | Uploads to personal cloud, forwarded emails, USB writes, large outbound transfers |
| Termination Workflow Integration | Automatically flag accounts 30 days before and after announced departure |
| Privileged Access Monitoring | Any privileged account activity outside of approved change windows or job function |

### Response Considerations

- Involve HR and Legal before any confrontation or account action
- Preserve evidence before disabling accounts (imaging, log preservation)
- Do not alert the individual prematurely — covert investigation may be required
- Consider law enforcement referral for criminal activity (data theft, fraud, sabotage)
- Document chain of custody for any evidence collected

---

## Archetype 2 — Negligent Insider (`NEG`)

### Definition

A negligent insider is an individual with authorized access who causes harm unintentionally through carelessness, ignorance of security policy, or prioritizing convenience over security. The individual does not intend harm but their actions create material risk or actual damage.

### Common Drivers

| Driver | Description | Example |
|--------|-------------|---------|
| Convenience | Bypassing security controls to get work done faster | Using personal Dropbox because SharePoint is "too slow" |
| Ignorance | Lack of awareness about what is permitted or what constitutes a risk | Forwarding work email to personal account "to work from home" |
| Policy Ambiguity | Genuine uncertainty about what is allowed | Using an unapproved AI tool because policy on AI doesn't exist yet |
| Phishing / Social Engineering | Being deceived into performing a harmful action | Clicking a credential-harvesting link |
| Misconfiguration | Incorrectly configuring a system or access control | Making an S3 bucket public unintentionally |
| Fatigue / Rushing | Making security mistakes under time pressure | Sending sensitive file to wrong email recipient |

### Behavioral Profile

Negligent insiders are often high-volume, high-noise risks — their individual behaviors may be low severity, but collectively they represent a significant aggregate risk.

**Technical behavioral signals:**
- Use of personal storage services (Google Drive, Dropbox, iCloud) to store work data
- Email forwarding to personal accounts
- Weak passwords, credential reuse across personal and work accounts
- Failure to follow multi-factor authentication enrollment
- Accessing work systems from unmanaged devices or open public networks
- Sharing credentials with colleagues for convenience

### Detection Strategy

| Strategy | What to Look For |
|----------|----------------|
| DLP Policy Violation Monitoring | Unapproved storage destinations, email to personal accounts |
| Unmanaged Device Detection | MDM non-compliance, access from unregistered device fingerprints |
| Phishing Simulation Metrics | Click rates; users repeatedly failing phishing simulations |
| MFA Enrollment and Bypass | Accounts without MFA; MFA bypass attempts |
| Shadow IT Discovery | Unsanctioned applications detected via network traffic or SSO logs |

### Response Considerations

- Response is predominantly educational and procedural, not investigative
- First-occurrence response is typically training, policy acknowledgment, or manager notification
- Repeated violations may escalate to HR and formal disciplinary action
- Distinguish clearly between negligent and malicious behavior before escalating — the outcomes are very different
- Consider whether the negligence is enabled by poor tooling or policy (if the approved tool is worse than the personal one, users will use the personal one)

---

## Archetype 3 — Compromised Insider (`COM`)

### Definition

A compromised insider is an individual whose legitimate access credentials, accounts, or devices have been obtained and are being used by an external threat actor. The insider may be entirely unaware that their account is being misused. The harmful activity is being conducted *by* an external adversary *through* an insider identity.

### How Compromise Occurs

| Compromise Method | Description |
|------------------|-------------|
| Phishing / Spear Phishing | Credentials stolen via deceptive email |
| Credential Stuffing | Credentials from a third-party breach reused on corporate systems |
| Malware / Keylogger | Credentials harvested from infected endpoint |
| MFA Fatigue Attack | User approves fraudulent MFA push after repeated prompts |
| Pass-the-Hash / Pass-the-Ticket | Lateral movement using captured session tokens |
| Physical Theft | Laptop, phone, or token stolen or lost |
| Social Engineering | Credentials obtained via impersonation (e.g., fake IT helpdesk) |

### Behavioral Profile

The compromised insider presents uniquely because the account behavior may be technically consistent with the insider's role — the adversary has done reconnaissance and knows how the account is normally used. Key distinguishing signals are contextual and temporal rather than content-based.

**Technical behavioral signals:**
- Login from an unusual geographic location not consistent with the user's travel
- Impossible travel: login from two geographically distant locations within a physically impossible timeframe
- Login at times far outside the user's established working pattern (e.g., 3 AM local time)
- Login from a new device or browser profile
- User-agent string inconsistent with the user's normal browser/OS combination
- Successful login followed immediately by elevated activity (bulk access, export) — unusual for human behavior
- MFA approval after prolonged pause or out of sequence with normal patterns

### Detection Strategy

| Strategy | What to Look For |
|----------|----------------|
| Impossible Travel Detection | Two logins from geographically distant IPs within an impossible time window |
| New Device / New IP Baseline | Alert on first login from a device or IP not in the user's 90-day history |
| Time-of-Day Anomaly | Login significantly outside the user's established active hours |
| Velocity Anomaly | Activity rate (files accessed, queries executed) far above the user's normal pattern |
| MFA Pattern Analysis | MFA approvals at unusual times; MFA fatigue push sequences |
| Session Hijacking Signals | Token reuse from different IP mid-session; rapid context switching |

### Response Considerations

- First response is verification, not confrontation — contact the legitimate user through an out-of-band channel (phone, in-person) to determine if the activity is theirs
- Do not alert via corporate email or IM — the attacker may be monitoring those channels
- Simultaneously preserve logs before the session terminates
- Enforce session revocation and credential reset immediately upon confirmation of compromise
- The legitimate insider is a victim and should not be treated as a suspect unless evidence of collusion exists
- Notify the insider of the compromise and provide clear guidance on next steps (password reset, device scan)

---

## Archetype 4 — Third-Party / Contractor (`TP`)

### Definition

A third-party insider is an individual who has been granted access to organizational systems, data, or facilities in a vendor, contractor, partner, or supply chain capacity, and who either intentionally or negligently causes harm using that access. The access is legitimate but the activity falls outside the authorized scope.

### Why Third-Party Insiders Are Distinct

Third-party insiders represent a unique challenge because:

- They often have limited visibility in HR systems — behavioral context (job changes, disgruntlement) is not available to the organization
- They may operate under the governance of their own employer, which may have conflicting incentives
- Access is often over-provisioned because scope is poorly defined at contract time
- Activity may occur during off-hours or from off-site locations that are harder to baseline
- Legal and contractual response options differ from those applicable to direct employees

### Sub-Types

| Sub-Type | Description |
|----------|-------------|
| Contractor / Consultant | Individual hired temporarily for a project; may retain access beyond project end |
| Managed Service Provider (MSP) | Third-party managing IT systems with broad, often privileged access |
| Software Vendor / SaaS Provider | Access through software supply chain (e.g., update mechanism, support access) |
| Business Partner | Organization with API or system integration access |
| Auditor / Assessor | External party with temporary read access for audit or compliance purposes |

### Behavioral Profile

**Access governance signals (often visible before incidents occur):**
- Access that outlasts the project or contract period
- Access to systems unrelated to the vendor's stated scope
- Accounts that are never used but remain active
- Generic or shared credentials used by multiple vendor personnel

**Technical behavioral signals during incidents:**
- Access outside the vendor's contracted service hours
- Access to data volumes or systems inconsistent with the contracted service
- Data exfiltration through vendor-managed channels (RMM tools, support tunnels, backup systems)
- Lateral movement from a vendor-managed system to production systems
- New accounts or backdoors created within a vendor's access scope

### Detection Strategy

| Strategy | What to Look For |
|----------|----------------|
| Third-Party Account Lifecycle | Accounts active past contract end date; accounts provisioned without formal request |
| Scope Deviation | Third-party accessing systems outside their contracted scope |
| RMM / Remote Tool Monitoring | Unusual commands, file transfers, or connections through vendor remote access tools |
| Access Volume Baseline | Third-party accessing abnormally large volumes of data relative to service scope |
| Off-Hours Access | Access at times inconsistent with the vendor's geographic location and time zone |

### Response Considerations

- Third-party incidents require immediate coordination with the vendor's security and legal teams
- Review the contract for breach notification and cooperation clauses before action
- Coordinate with Procurement and Vendor Management — they own the relationship
- Consider whether the incident is vendor-side (e.g., the vendor's own systems were compromised) vs. an insider at the vendor acting deliberately
- Access revocation may affect service continuity — involve operations teams before disabling critical vendor accounts
- Understand the contractual obligations around evidence sharing with and from the vendor

---

## Archetype Comparison: Detection Approach

| Dimension | MAL | NEG | COM | TP |
|-----------|-----|-----|-----|-----|
| Primary detection data source | UEBA, DLP, file access logs | DLP policy logs, email logs, MDM | Authentication logs, UEBA | Access logs, third-party activity logs |
| Detection model type | Behavioral anomaly + threshold | Policy-based + threshold | Contextual anomaly (time, location, device) | Scope-based + behavioral |
| False positive risk | Medium | High (volume) | Medium | Medium |
| Investigative timeline | Often weeks | Usually immediate | Usually rapid | Depends on cooperation |
| HR involvement | Yes — early | Yes — early | No (victim) until collusion confirmed | Procurement/Legal first |
| Legal involvement | Yes — early | Case by case | Always (victim notification) | Always (contract review) |

---

## References

- Carnegie Mellon University CERT Insider Threat Center — Insider Threat Research and Resources
- NIST SP 800-53 Rev 5 — PS (Personnel Security) and AC (Access Control) control families
- CISA Insider Threat Mitigation Resources
- Verizon Data Breach Investigations Report — Insider Threat statistics and patterns
