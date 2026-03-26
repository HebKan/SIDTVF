# Detection and Artifact Coverage Matrix

> **Version:** 1.0 | **Last Updated:** 2026-03-26
>
> This matrix shows the artifact coverage status for every active SIDTVF scenario.
> Update this file whenever a new artifact is created or a scenario is added.
> See [CLAUDE.md](../CLAUDE.md) for cascade update rules.

---

## How to Read This Matrix

| Symbol | Meaning |
|--------|---------|
| ✓ | Complete and published |
| P | Planned (in backlog) |
| — | Not applicable for this scenario type |

**Artifact types:**
- **Sigma** — Platform-agnostic Sigma detection rule
- **KQL** — Microsoft Sentinel / Log Analytics query
- **SPL** — Splunk SPL query
- **Playbook** — Incident response playbook
- **Hunt** — Threat hunting hypothesis
- **Test Case** — Validation test case (procedural)
- **Triage Card** — SOC analyst quick-reference card
- **Simulation** — Layer 7 behavioral simulation test

---

## Coverage Matrix

| Scenario | Sigma | KQL | SPL | Playbook | Hunt | Test Case | Triage Card | Simulation |
|----------|-------|-----|-----|----------|------|-----------|-------------|------------|
| **DE-001** Cloud Storage Upload | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **DE-002** Email Auto-Forwarding | P | P | P | P | ✓ | P | ✓ | ✓ |
| **DE-003** AI Tool Data Leakage | P | P | P | P | P | P | P | P |
| **PM-001** Privilege Escalation | P | P | P | P | P | P | P | P |
| **UA-001** After-Hours Access | P | P | P | P | P | P | P | P |
| **SB-001** Database Deletion | P | P | P | ✓ | P | P | ✓ | ✓ |
| **TP-001** Vendor Credential Abuse | P | P | P | P | P | P | P | P |
| **FR-001** Invoice Manipulation | P | P | P | P | P | P | P | P |
| **IP-001** Source Code Theft | P | P | P | P | P | P | P | P |
| **PV-001** Shadow IT Storage | P | P | P | P | P | P | P | P |
| **AC-001** Account Compromise | P | P | P | P | P | P | P | P |

**Coverage totals:** 6 of 88 artifacts complete (7%) — framework is in active development.

---

## Priority Build Queue

Use this queue to determine which artifacts to create next. Priority is based on scenario severity, detection gap, and estimated number of organizations affected.

| Priority | Artifact | Scenario | Rationale |
|----------|---------|----------|-----------|
| 1 | Sigma + KQL | DE-002 | Email forwarding is the second most common exfiltration vector; no detection rule exists |
| 2 | Sigma + KQL | PM-001 | Privilege abuse is highly prevalent in healthcare and finance |
| 3 | Playbook | DE-002 | Playbook gap for a scenario with hunt coverage |
| 4 | All artifacts | UA-001 | After-hours access is high-prevalence, easy to detect |
| 5 | Sigma + Playbook | AC-001 | Account compromise is rising; AiTM phishing makes this urgent |
| 6 | KQL + SPL | DE-003 | AI tool usage is accelerating; detection gap is industry-wide |
| 7 | Sigma + KQL | TP-001 | Third-party risk is a regulatory priority (DORA, NIST CSF 2.0) |
| 8 | All artifacts | SB-001 | Sabotage has a playbook and simulation; detection rules needed |
| 9 | All artifacts | FR-001 | Financial fraud scenarios are SOX-relevant; most orgs lack detection |
| 10 | All artifacts | IP-001 | IP theft scenarios are DTSA-relevant; high-value targets need coverage |

---

## Coverage by Artifact Type

| Artifact Type | Complete | Planned | Total | % Complete |
|--------------|----------|---------|-------|-----------|
| Sigma Rules | 1 | 10 | 11 | 9% |
| KQL Queries | 1 | 10 | 11 | 9% |
| SPL Queries | 1 | 10 | 11 | 9% |
| Playbooks | 2 | 9 | 11 | 18% |
| Hunt Hypotheses | 2 | 9 | 11 | 18% |
| Validation Test Cases | 1 | 10 | 11 | 9% |
| Triage Cards | 2 | 9 | 11 | 18% |
| Simulation Tests | 3 | 8 | 11 | 27% |

---

## Changelog

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-03-26 | Initial matrix — 11 scenarios, 6 complete artifacts |
