# Compliance Framework Mapping

> **Document Version:** 1.0 | **Status:** Active | **Last Updated:** 2026-03-25
>
> This document maps SIDTVF scenarios and program activities to major compliance and regulatory frameworks.
> It is intended to help organizations identify which SIDTVF scenarios are most relevant to their compliance obligations,
> and to support evidence gathering for audits and assessments.
>
> This is a guidance document, not legal advice. Organizations must confirm compliance interpretations with their legal, compliance, and audit teams.

---

## Frameworks Covered

| Code | Framework | Version / Publication |
|------|-----------|-----------------------|
| NIST-53 | NIST SP 800-53 | Revision 5 (2020) |
| ISO27001 | ISO/IEC 27001 | 2022 |
| SOC2 | SOC 2 | AICPA Trust Services Criteria (2017, updated) |
| HIPAA | HIPAA Security Rule | 45 CFR Part 164 |
| PCIDSS | PCI DSS | Version 4.0 (2022) |
| GDPR | EU General Data Protection Regulation | 2016/679 |
| CMMC | Cybersecurity Maturity Model Certification | Level 2 (CMMC 2.0) |
| SOX | Sarbanes-Oxley Act | Section 302, 404 |

---

## Part 1 — Scenario-to-Compliance Mapping

This table identifies which compliance frameworks have specific control requirements that each SIDTVF scenario helps address or validate.

| Scenario ID | Scenario Name | NIST-53 | ISO27001 | SOC2 | HIPAA | PCIDSS | GDPR | CMMC |
|-------------|---------------|---------|----------|------|-------|--------|------|------|
| DE-001 | Cloud Storage Upload Exfiltration | AC-4, SI-4, AU-6 | A.5.12, A.8.11, A.8.16 | CC6.7, CC7.2 | §164.312(a)(1), §164.312(b) | Req 7.2, Req 12.3 | Art. 32 | AC.L2-3.1.3 |
| DE-002 | Email Auto-Forwarding Rule | AC-4, AU-6, SI-4 | A.5.12, A.8.11 | CC6.7, CC7.2 | §164.312(a)(1) | Req 7.2, Req 10.7 | Art. 32 | AC.L2-3.1.3 |
| PM-001 | Privilege Escalation via Access Abuse | AC-2, AC-6, AC-17 | A.5.15, A.8.2 | CC6.3, CC6.6 | §164.312(a)(1) | Req 7.2, Req 8.6 | Art. 32 | AC.L2-3.1.1, AC.L2-3.1.2 |
| UA-001 | After-Hours Unauthorized System Access | AC-2, AU-6, SI-4 | A.8.16, A.5.15 | CC6.3, CC7.2 | §164.312(b) | Req 10.7, Req 8.6 | Art. 32 | AC.L2-3.1.1 |
| SB-001 | Targeted Database Deletion | CP-9, CP-10, SI-4 | A.8.13, A.5.30 | CC7.3, A1.2 | §164.310(d)(2) | Req 12.10, Req 10.7 | Art. 32 | RE.L2-3.2.2 |
| TP-001 | Vendor Credential Abuse | SA-9, AC-2, SI-4 | A.5.19, A.5.22 | CC9.2, CC6.3 | §164.308(b) | Req 12.8, Req 8.6 | Art. 28, Art. 32 | SR.L2-3.15.1 |

---

## Part 2 — Program Activity-to-Compliance Mapping

This table maps core SIDTVF program activities (across all six layers) to compliance controls. Use this during audits to demonstrate that your insider threat program satisfies specific control requirements.

### Governance and Policy (Layer 1 inputs + Layer 4 outputs)

| SIDTVF Activity | NIST-53 | ISO27001 | SOC2 | HIPAA | PCIDSS | GDPR | CMMC |
|----------------|---------|----------|------|-------|--------|------|------|
| Insider threat policy documentation | PM-2, PS-1 | A.5.1, A.6.1 | CC1.4 | §164.308(a)(2) | Req 12.1, Req 12.5 | Art. 24 | AC.L2-3.1.1 |
| Employee acceptable use policy | PS-6, AT-2 | A.6.2 | CC1.4 | §164.308(a)(5) | Req 12.4, Req 12.6 | Art. 13 | AT.L2-3.2.1 |
| Third-party vendor risk assessment | SA-9, SR-3 | A.5.19, A.5.22 | CC9.2 | §164.308(b) | Req 12.8 | Art. 28 | SR.L2-3.15.1 |
| Employee separation procedures | PS-4, AC-2 | A.6.5 | CC6.2 | §164.308(a)(3) | Req 8.8 | Art. 17 | AC.L2-3.1.1 |

### Monitoring and Detection (Layer 2 + Layer 3)

| SIDTVF Activity | NIST-53 | ISO27001 | SOC2 | HIPAA | PCIDSS | GDPR | CMMC |
|----------------|---------|----------|------|-------|--------|------|------|
| User activity monitoring | AC-2, AU-6, SI-4 | A.8.16 | CC6.3, CC7.2 | §164.312(b) | Req 10.6, Req 10.7 | Art. 6, Art. 13 | AC.L2-3.1.1 |
| Privileged access monitoring | AC-6, AU-9, SI-4 | A.8.2, A.8.18 | CC6.3 | §164.312(a)(1) | Req 7.2, Req 10.2 | Art. 32 | AC.L2-3.1.6 |
| DLP deployment and tuning | SI-4, AC-21 | A.5.12, A.8.11 | CC6.7 | §164.312(a)(1) | Req 7.2, Req 12.5.1 | Art. 25, Art. 32 | AC.L2-3.1.3 |
| Audit log collection and retention | AU-2, AU-9, AU-11 | A.8.15 | CC7.2 | §164.312(b), §164.316(b)(2) | Req 10.3, Req 10.5, Req 10.7 | Art. 5(e) | AU.L2-3.3.1 |
| Threat hunting (proactive) | RA-3, SI-4 | A.8.16, A.5.24 | CC7.2 | §164.308(a)(1) | Req 10.6 | Art. 32 | IR.L2-3.6.1 |

### Validation and Testing (Layer 2)

| SIDTVF Activity | NIST-53 | ISO27001 | SOC2 | HIPAA | PCIDSS | GDPR | CMMC |
|----------------|---------|----------|------|-------|--------|------|------|
| Control validation testing | CA-2, CA-7 | A.8.29 | CC4.1 | §164.308(a)(1) | Req 12.10.2 | Art. 32(1)(d) | CA.L2-3.12.1 |
| Penetration testing (insider-scoped) | CA-8, RA-5 | A.8.29 | CC4.1 | §164.308(a)(8) | Req 11.4 | Art. 32 | CA.L2-3.12.4 |

### Incident Response (Layer 4 — IR consumer)

| SIDTVF Activity | NIST-53 | ISO27001 | SOC2 | HIPAA | PCIDSS | GDPR | CMMC |
|----------------|---------|----------|------|-------|--------|------|------|
| Incident response playbooks | IR-1, IR-8 | A.5.26 | CC7.3, CC7.5 | §164.308(a)(6) | Req 12.10 | Art. 33 | IR.L2-3.6.1 |
| Evidence preservation and forensics | AU-9, IR-4, IR-6 | A.5.28 | CC7.3 | §164.312(b) | Req 12.10.4 | Art. 33 | IR.L2-3.6.2 |
| Breach notification procedures | IR-6, SI-12 | A.5.24 | CC7.3 | §164.408, §164.410 | Req 12.10.3 | Art. 33, Art. 34 | IR.L2-3.6.2 |

### Metrics and Continuous Improvement (Layer 6)

| SIDTVF Activity | NIST-53 | ISO27001 | SOC2 | HIPAA | PCIDSS | GDPR | CMMC |
|----------------|---------|----------|------|-------|--------|------|------|
| Program maturity assessment | PM-6, CA-7 | A.5.36, Ch. 9 | CC4.1 | §164.308(a)(1) | Req 12.1 | Art. 24 | CA.L2-3.12.3 |
| KPI tracking and reporting | PM-6, PM-9 | A.5.36 | CC4.2 | §164.308(a)(1) | Req 12.1 | Art. 24 | CA.L2-3.12.3 |

---

## Part 3 — Control Detail by Framework

### NIST SP 800-53 Rev 5 — Key Control Families

| Control Family | Most Relevant Controls | SIDTVF Coverage |
|---------------|----------------------|----------------|
| AC — Access Control | AC-2 (Account Management), AC-4 (Information Flow), AC-6 (Least Privilege), AC-17 (Remote Access) | PM-001, UA-001, DE-001 |
| AU — Audit and Accountability | AU-2 (Event Logging), AU-6 (Audit Review), AU-9 (Protection of Audit Information), AU-11 (Audit Retention) | All scenarios |
| CA — Assessment, Authorization, Monitoring | CA-2 (Control Assessments), CA-7 (Continuous Monitoring), CA-8 (Penetration Testing) | Validation layer |
| IR — Incident Response | IR-1 (Policy), IR-4 (Handling), IR-6 (Reporting), IR-8 (Plan) | Playbook layer |
| PM — Program Management | PM-2 (Senior Information Security Officer), PM-6 (Security Measures), PM-9 (Risk Management) | Governance |
| PS — Personnel Security | PS-1 (Policy), PS-4 (Termination), PS-6 (Access Agreements) | Governance, DE-001, DE-002 |
| SA — System and Services Acquisition | SA-9 (External Services) | TP-001 |
| SI — System and Information Integrity | SI-4 (System Monitoring), SI-12 (Information Management) | All scenarios |
| SR — Supply Chain Risk Management | SR-3 (Supply Chain Controls) | TP-001 |

---

### ISO/IEC 27001:2022 — Annex A Controls

| Control | Description | Relevant SIDTVF Scenarios |
|---------|-------------|--------------------------|
| A.5.1 | Policies for information security | All — governance |
| A.5.12 | Classification of information | DE-001, DE-002 |
| A.5.15 | Access control | PM-001, UA-001 |
| A.5.19 | Information security in supplier relationships | TP-001 |
| A.5.22 | Monitoring, review and change management of supplier services | TP-001 |
| A.5.24 | Information security incident management | All scenarios — playbooks |
| A.5.26 | Response to information security incidents | All scenarios — playbooks |
| A.5.28 | Collection of evidence | SB-001, all forensic activities |
| A.5.30 | ICT readiness for business continuity | SB-001 |
| A.5.36 | Compliance with policies | Governance, metrics |
| A.6.1 | Screening (personnel) | Governance |
| A.6.2 | Terms and conditions of employment | Governance, acceptable use policy |
| A.6.5 | Responsibilities after termination | DE-001, DE-002, offboarding |
| A.8.2 | Privileged access rights | PM-001 |
| A.8.11 | Data masking | DE-001, DE-002 |
| A.8.13 | Information backup | SB-001 |
| A.8.15 | Logging | All scenarios |
| A.8.16 | Monitoring activities | UA-001, all detection activities |
| A.8.18 | Use of privileged utility programs | PM-001 |
| A.8.29 | Security testing in development and acceptance | Validation layer |

---

### HIPAA Security Rule — Insider Threat Relevant Standards

| Standard | CFR Reference | SIDTVF Program Activity |
|----------|--------------|------------------------|
| Administrative Safeguards | | |
| Security Management Process | §164.308(a)(1) | Risk analysis, ITP program governance |
| Workforce Security | §164.308(a)(3) | Personnel screening, access authorization |
| Information Access Management | §164.308(a)(4) | Least privilege, access controls (PM-001) |
| Security Awareness and Training | §164.308(a)(5) | User training, policy acknowledgment |
| Security Incident Procedures | §164.308(a)(6) | Incident response playbooks |
| Contingency Plan | §164.308(a)(7) | Backup and recovery (SB-001) |
| Evaluation | §164.308(a)(8) | Periodic assessment, validation testing |
| Business Associate Contracts | §164.308(b) | Third-party vendor agreements (TP-001) |
| Technical Safeguards | | |
| Access Control | §164.312(a)(1) | IAM, least privilege (PM-001, UA-001) |
| Audit Controls | §164.312(b) | Logging, SIEM, monitoring |
| Integrity | §164.312(c)(1) | Data integrity controls (SB-001) |
| Person or Entity Authentication | §164.312(d) | MFA, authentication monitoring |
| Transmission Security | §164.312(e)(1) | Encryption of data in transit |

---

### PCI DSS 4.0 — Most Relevant Requirements

| Requirement | Description | SIDTVF Coverage |
|------------|-------------|----------------|
| Req 7.2 | Access control systems managing access to system components | PM-001, UA-001 |
| Req 8.6 | Application and system accounts are managed | PM-001, TP-001 |
| Req 8.8 | Policies and procedures for user identification management | Governance, DE-001 (offboarding) |
| Req 10.2 | Audit logs capture all required events | All scenarios |
| Req 10.3 | Audit logs are protected from destruction and unauthorized modifications | All scenarios — evidence preservation |
| Req 10.5 | Retain audit log history | All scenarios — retention |
| Req 10.6 | Time synchronization is maintained | All scenarios — correlation |
| Req 10.7 | Failures of critical security controls are detected and reported | UA-001, SB-001 |
| Req 11.4 | Network intrusions and unexpected file changes are detected | DE-001, PM-001 |
| Req 12.1 | Information security policy | Governance |
| Req 12.3 | Risks to the CDE and cardholder data are formally evaluated | DE-001 |
| Req 12.5.1 | Responsibilities for protecting cardholder data are defined | Governance |
| Req 12.6 | Security awareness education | Governance — negligent insider |
| Req 12.8 | Third-party service provider risks are managed | TP-001 |
| Req 12.10 | Suspected and confirmed security incidents are responded to | All scenarios — playbooks |

---

## Part 4 — Audit Evidence Map

Use this section to identify which SIDTVF artifacts serve as audit evidence for specific compliance requirements.

| Compliance Requirement | Evidence Type | SIDTVF Artifact |
|----------------------|---------------|----------------|
| Evidence of security monitoring | Active detection rules | `detections/sigma/`, `detections/queries/` |
| Evidence of control testing | Validation test results | `validation/test-cases/TC-*.md` (executed results) |
| Evidence of incident response planning | Documented playbooks | `playbooks/examples/PB-*.md` |
| Evidence of threat hunting | Documented hunt results | `hunting/hypotheses/HYP-*.md` (hunt cycle log) |
| Evidence of risk assessment | Scenario library | `scenarios/**/*.md` |
| Evidence of log retention | Validation test results (data source requirements) | `validation/test-cases/*.md` (prerequisites section) |
| Evidence of continuous monitoring | Program metrics | `metrics/kpis.md` (completed KPI dashboard) |
| Evidence of third-party risk management | TP scenarios and vendor assessments | `scenarios/third-party-risk/*.md` |
| Evidence of workforce security | Legal and privacy guidance | `docs/legal-privacy.md` |

---

## Changelog

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-03-25 | Initial mapping for scenarios DE-001, DE-002, PM-001, UA-001, SB-001, TP-001 |
