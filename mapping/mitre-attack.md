# MITRE ATT&CK Technique Mapping

> **Document Version:** 1.0 | **Status:** Active | **Last Updated:** 2026-03-25
>
> **ATT&CK Version:** Enterprise ATT&CK v15
>
> This document maps all active SIDTVF scenarios to MITRE ATT&CK tactics and techniques.
> Update this document when new scenarios are added or when ATT&CK version is updated.
> See [CLAUDE.md](../CLAUDE.md) for update instructions.

---

## Coverage Summary

| Tactic | # Techniques Covered | Active Scenarios |
|--------|---------------------|-----------------|
| Initial Access (TA0001) | 4 | UA-001, TP-001, AC-001 |
| Privilege Escalation (TA0004) | 1 | PM-001 |
| Defense Evasion (TA0005) | 3 | PM-001, SB-001, IP-001 |
| Credential Access (TA0006) | 3 | PM-001, AC-001 |
| Discovery (TA0007) | 4 | PM-001, UA-001, SB-001, TP-001 |
| Collection (TA0009) | 6 | DE-001, DE-002, PM-001, SB-001, TP-001, IP-001, AC-001 |
| Command and Control (TA0011) | 1 | TP-001 |
| Exfiltration (TA0010) | 5 | DE-001, DE-002, PM-001, TP-001, IP-001, PV-001 |
| Impact (TA0040) | 5 | SB-001, FR-001 |

---

## Full Technique Mapping Table

| SIDTVF ID | Scenario Name | Tactic | ATT&CK ID | Technique Name | Notes |
|-----------|---------------|--------|-----------|----------------|-------|
| DE-001 | Cloud Storage Upload Exfiltration | Collection | T1213 | Data from Information Repositories | Phase 1 — staging |
| DE-001 | Cloud Storage Upload Exfiltration | Collection | T1005 | Data from Local System | Phase 1 — staging |
| DE-001 | Cloud Storage Upload Exfiltration | Collection | T1560.001 | Archive Collected Data: Archive via Utility | Phase 1 — compression |
| DE-001 | Cloud Storage Upload Exfiltration | Exfiltration | T1567.002 | Exfiltration Over Web Service: Exfiltration to Cloud Storage | Phase 3 — upload |
| DE-001 | Cloud Storage Upload Exfiltration | Defense Evasion | T1070.004 | Indicator Removal: File Deletion | Phase 4 — cover |
| DE-002 | Email Auto-Forwarding Rule | Collection | T1114.003 | Email Collection: Email Forwarding Rule | Phase 1 — rule creation |
| DE-002 | Email Auto-Forwarding Rule | Exfiltration | T1048.003 | Exfiltration Over Alternative Protocol: Exfiltration Over Unencrypted Non-C2 Protocol | Phase 2 — forwarding |
| DE-002 | Email Auto-Forwarding Rule | Persistence | T1098.003 | Account Manipulation: Additional Email Delegate Permissions | Phase 3 — scope expansion |
| PM-001 | Privilege Escalation via Access Abuse | Discovery | T1087.002 | Account Discovery: Domain Account | Phase 1 — recon |
| PM-001 | Privilege Escalation via Access Abuse | Discovery | T1083 | File and Directory Discovery | Phase 1 — recon |
| PM-001 | Privilege Escalation via Access Abuse | Privilege Escalation | T1078.002 | Valid Accounts: Domain Accounts | Phase 2 — access acquisition |
| PM-001 | Privilege Escalation via Access Abuse | Credential Access | T1552.001 | Unsecured Credentials: Credentials in Files | Phase 2 — shared creds |
| PM-001 | Privilege Escalation via Access Abuse | Collection | T1530 | Data from Cloud Storage Object | Phase 3 — abuse |
| PM-001 | Privilege Escalation via Access Abuse | Collection | T1005 | Data from Local System | Phase 3 — abuse |
| PM-001 | Privilege Escalation via Access Abuse | Exfiltration | T1567.002 | Exfiltration Over Web Service: Exfiltration to Cloud Storage | Phase 4 — exfiltration |
| UA-001 | After-Hours Unauthorized System Access | Initial Access | T1078.002 | Valid Accounts: Domain Accounts | Phase 1, Phase 2 |
| UA-001 | After-Hours Unauthorized System Access | Discovery | T1082 | System Information Discovery | Phase 2 — system access |
| UA-001 | After-Hours Unauthorized System Access | Discovery | T1087 | Account Discovery | Phase 2 — system access |
| UA-001 | After-Hours Unauthorized System Access | Collection | T1005 | Data from Local System | Phase 3 — data access |
| UA-001 | After-Hours Unauthorized System Access | Collection | T1213 | Data from Information Repositories | Phase 3 — data access |
| UA-001 | After-Hours Unauthorized System Access | Impact | T1565.001 | Data Manipulation: Stored Data Manipulation | Phase 3 — fraud variant |
| SB-001 | Targeted Database Deletion | Discovery | T1083 | File and Directory Discovery | Phase 1 — targeting |
| SB-001 | Targeted Database Deletion | Discovery | T1082 | System Information Discovery | Phase 1 — targeting |
| SB-001 | Targeted Database Deletion | Impact | T1485 | Data Destruction | Phase 3 — destruction |
| SB-001 | Targeted Database Deletion | Impact | T1490 | Inhibit System Recovery | Phase 2 — backup manipulation |
| SB-001 | Targeted Database Deletion | Impact | T1489 | Service Stop | Phase 3 — if DB service stopped |
| SB-001 | Targeted Database Deletion | Defense Evasion | T1070.004 | Indicator Removal: File Deletion | Phase 4 — misdirection |
| SB-001 | Targeted Database Deletion | Impact | T1486 | Data Encrypted for Impact | Phase 3 — ransomware variant |
| TP-001 | Vendor Credential Abuse | Initial Access | T1078.004 | Valid Accounts: Cloud Accounts | Phase 1 — cloud environments |
| TP-001 | Vendor Credential Abuse | Initial Access | T1195.001 | Supply Chain Compromise: Compromise Software Dependencies | Phase 1 — RMM vector |
| TP-001 | Vendor Credential Abuse | Discovery | T1087 | Account Discovery | Phase 1 — scope enumeration |
| TP-001 | Vendor Credential Abuse | Discovery | T1018 | Remote System Discovery | Phase 1 — scope enumeration |
| TP-001 | Vendor Credential Abuse | Collection | T1005 | Data from Local System | Phase 3 — collection |
| TP-001 | Vendor Credential Abuse | Collection | T1560.001 | Archive Collected Data: Archive via Utility | Phase 3 — staging |
| TP-001 | Vendor Credential Abuse | Exfiltration | T1048.002 | Exfiltration Over Alternative Protocol: Exfiltration Over Asymmetric Encrypted Non-C2 Protocol | Phase 4 — SFTP |
| TP-001 | Vendor Credential Abuse | Command and Control | T1219 | Remote Access Software | Phase 3, Phase 4 — RMM |
| FR-001 | Invoice and Vendor Payment Manipulation | Impact | T1565.001 | Data Manipulation: Stored Data Manipulation | Phase 2 — vendor master modification |
| FR-001 | Invoice and Vendor Payment Manipulation | Defense Evasion | T1070.006 | Indicator Removal: Timestomp | Phase 4 — cover (if timestamps modified) |
| FR-001 | Invoice and Vendor Payment Manipulation | Impact | T1496 | Resource Hijacking | Phase 4 — payment redirection |
| IP-001 | Source Code and Proprietary Algorithm Theft | Collection | T1213 | Data from Information Repositories | Phase 1, Phase 2 |
| IP-001 | Source Code and Proprietary Algorithm Theft | Collection | T1005 | Data from Local System | Phase 2 |
| IP-001 | Source Code and Proprietary Algorithm Theft | Collection | T1560.001 | Archive Collected Data: Archive via Utility | Phase 3 |
| IP-001 | Source Code and Proprietary Algorithm Theft | Exfiltration | T1567.002 | Exfiltration Over Web Service: Exfiltration to Cloud Storage | Phase 3 |
| IP-001 | Source Code and Proprietary Algorithm Theft | Exfiltration | T1052.001 | Exfiltration Over Physical Medium: Exfiltration over USB | Phase 3 |
| IP-001 | Source Code and Proprietary Algorithm Theft | Defense Evasion | T1070.003 | Indicator Removal: Clear Command History | Phase 4 |
| PV-001 | Shadow IT Data Storage | Exfiltration | T1567.002 | Exfiltration Over Web Service: Exfiltration to Cloud Storage | Phase 3 — unintentional |
| AC-001 | Insider Account Compromise by External Actor | Initial Access | T1566.002 | Phishing: Spearphishing Link | Phase 1 |
| AC-001 | Insider Account Compromise by External Actor | Credential Access | T1621 | Multi-Factor Authentication Request Generation | Phase 1 — MFA fatigue/bypass |
| AC-001 | Insider Account Compromise by External Actor | Credential Access | T1539 | Steal Web Session Cookie | Phase 1 — session hijacking variant |
| AC-001 | Insider Account Compromise by External Actor | Initial Access | T1078 | Valid Accounts | Phase 2 |
| AC-001 | Insider Account Compromise by External Actor | Collection | T1213 | Data from Information Repositories | Phase 3 |
| AC-001 | Insider Account Compromise by External Actor | Collection | T1530 | Data from Cloud Storage Object | Phase 3 |
| AC-001 | Insider Account Compromise by External Actor | Exfiltration | T1041 | Exfiltration Over C2 Channel | Phase 4 |
| DE-003 | AI Tool Data Leakage | Collection | T1213 | Data from Information Repositories | Phase 2 — internal data access |
| DE-003 | AI Tool Data Leakage | Collection | T1005 | Data from Local System | Phase 2 — local file access |
| DE-003 | AI Tool Data Leakage | Exfiltration | T1567.002 | Exfiltration Over Web Service: Exfiltration to Cloud Storage | Phase 3 — upload to AI platform |
| DE-003 | AI Tool Data Leakage | Exfiltration | T1048.003 | Exfiltration Over Unencrypted Non-C2 Protocol | Phase 3 — when no HTTPS inspection |

---

## Technique Cross-Reference (Technique → Scenarios)

This table shows which scenarios cover each ATT&CK technique — useful for identifying detection coverage gaps by technique.

| ATT&CK ID | Technique Name | Scenarios That Cover It |
|-----------|----------------|------------------------|
| T1005 | Data from Local System | DE-001, DE-003, PM-001, UA-001, TP-001, IP-001 |
| T1018 | Remote System Discovery | TP-001 |
| T1041 | Exfiltration Over C2 Channel | AC-001 |
| T1048.002 | Exfiltration Over Asymmetric Encrypted Non-C2 Protocol | TP-001 |
| T1048.003 | Exfiltration Over Unencrypted Non-C2 Protocol | DE-002, DE-003 |
| T1052.001 | Exfiltration Over Physical Medium: Exfiltration over USB | IP-001 |
| T1070.003 | Indicator Removal: Clear Command History | IP-001 |
| T1070.004 | Indicator Removal: File Deletion | DE-001, SB-001 |
| T1070.006 | Indicator Removal: Timestomp | FR-001 |
| T1078 | Valid Accounts | AC-001 |
| T1078.002 | Valid Accounts: Domain Accounts | PM-001, UA-001 |
| T1078.004 | Valid Accounts: Cloud Accounts | TP-001 |
| T1082 | System Information Discovery | UA-001, SB-001 |
| T1083 | File and Directory Discovery | PM-001, SB-001 |
| T1087 | Account Discovery | UA-001, TP-001 |
| T1087.002 | Account Discovery: Domain Account | PM-001 |
| T1098.003 | Account Manipulation: Additional Email Delegate Permissions | DE-002 |
| T1114.003 | Email Collection: Email Forwarding Rule | DE-002 |
| T1195.001 | Supply Chain Compromise: Compromise Software Dependencies | TP-001 |
| T1213 | Data from Information Repositories | DE-001, DE-003, UA-001, IP-001, AC-001 |
| T1219 | Remote Access Software | TP-001 |
| T1485 | Data Destruction | SB-001 |
| T1486 | Data Encrypted for Impact | SB-001 |
| T1489 | Service Stop | SB-001 |
| T1490 | Inhibit System Recovery | SB-001 |
| T1496 | Resource Hijacking | FR-001 |
| T1530 | Data from Cloud Storage Object | PM-001, AC-001 |
| T1539 | Steal Web Session Cookie | AC-001 |
| T1552.001 | Unsecured Credentials: Credentials in Files | PM-001 |
| T1560.001 | Archive Collected Data: Archive via Utility | DE-001, TP-001, IP-001 |
| T1565.001 | Data Manipulation: Stored Data Manipulation | UA-001, FR-001 |
| T1566.002 | Phishing: Spearphishing Link | AC-001 |
| T1567.002 | Exfiltration Over Web Service: Exfiltration to Cloud Storage | DE-001, DE-003, PM-001, IP-001, PV-001 |
| T1621 | Multi-Factor Authentication Request Generation | AC-001 |

---

## Coverage Gap Analysis

The following ATT&CK techniques frequently appear in insider threat incidents but are **not yet covered** by any SIDTVF scenario. These represent candidates for future scenario development.

| ATT&CK ID | Technique Name | Recommended New Scenario |
|-----------|----------------|--------------------------|
| T1078.001 | Valid Accounts: Default Accounts | AC-002 (Default Credential Abuse) |
| T1078.003 | Valid Accounts: Local Accounts | UA-002 (Local Account Abuse) |
| T1110 | Brute Force | AC-002 (Credential Stuffing variant) |
| T1133 | External Remote Services | AC-002 (VPN/remote access abuse) |
| T1136 | Create Account | PM-002 (Backdoor Account Creation) |
| T1222 | File and Directory Permissions Modification | PM-002 (Access Control Manipulation) |
| T1531 | Account Access Removal | SB-002 (Sabotage via Account Lockout) |
| T1537 | Transfer Data to Cloud Account | DE-003 (Cloud-to-Cloud Exfiltration) |
| T1552.002 | Unsecured Credentials: Credentials in Registry | PM-001 variant |
| T1562.001 | Impair Defenses: Disable or Modify Tools | SB-002 (Defensive Control Sabotage) |

---

## ATT&CK Version Notes

This mapping was created against MITRE ATT&CK Enterprise v15. When a new ATT&CK version is released:

1. Review technique IDs for changes (IDs do not typically change, but names and sub-technique assignments may)
2. Review the "Coverage Gap Analysis" section against the new version's technique list
3. Update the `ATT&CK Version` note at the top of this document
4. Add a changelog entry below

---

## Changelog

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-03-25 | Initial mapping for scenarios DE-001, DE-002, PM-001, UA-001, SB-001, TP-001 |
| 1.1 | 2026-03-26 | Added mappings for FR-001, IP-001, PV-001, AC-001; updated coverage summary and cross-reference table |
| 1.2 | 2026-03-26 | Added DE-003 (AI Tool Data Leakage) technique mappings |
