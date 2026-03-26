# SIDTVF — Scenario-Driven Insider Threat Detection & Validation Framework

> **Version:** 1.0 | **Status:** Active | **License:** MIT

SIDTVF is an open, structured methodology for building, testing, and continuously improving insider threat detection programs. Designed for security teams of any size, SIDTVF provides a common language, reusable templates, and measurable outcomes across all phases of insider threat operations.

---

## Why SIDTVF?

Insider threats are among the hardest security problems to solve because the adversary already has legitimate access. Traditional signature-based and perimeter defenses fail here. SIDTVF addresses this by providing:

- **Scenario-driven** threat modeling grounded in real-world behaviors, not hypothetical tooling
- **Validation-first** approach to testing whether your controls work before an incident proves they don't
- **Behavior-focused** detection that identifies what users do, not just what tools they use
- **Cross-team integration** connecting threat modeling, detection engineering, SOC, and incident response
- **Measurable outcomes** through defined metrics and a five-level program maturity model
- **Legal-first design** with built-in guidance for navigating the compliance and privacy landscape

---

## Framework Architecture

SIDTVF is organized into six interdependent layers:

```
┌──────────────────────────────────────────────────────────────────────┐
│                        SIDTVF Architecture                           │
├────────────────────────────┬─────────────────────────────────────────┤
│  Layer 1  │ Scenario       │ Define realistic insider threat behaviors│
│  Layer 2  │ Validation     │ Test whether controls detect them        │
│  Layer 3  │ Threat Hunting │ Proactively search for real activity     │
│  Layer 4  │ Integration    │ Feed findings to downstream teams        │
│  Layer 5  │ Mapping        │ Standardize to ATT&CK and compliance     │
│  Layer 6  │ Feedback Loop  │ Iterate and continuously improve         │
└────────────────────────────┴─────────────────────────────────────────┘
```

Full architecture documentation: [docs/framework-overview.md](docs/framework-overview.md)

---

## Repository Structure

```
SIDTVF/
├── README.md                              # This file — start here
├── CLAUDE.md                              # AI maintenance instructions
│
├── docs/
│   ├── framework-overview.md              # Full six-layer framework specification
│   ├── actor-archetypes.md                # Four insider threat actor definitions
│   ├── maturity-model.md                  # Program maturity levels and self-assessment
│   ├── legal-privacy.md                   # Legal, privacy, and compliance guidance
│   └── processes/                         # Step-by-step process guides and workflow diagrams
│       ├── scenario-authoring-guide.md    # How to create a new scenario
│       ├── validation-cycle-guide.md      # End-to-end validation methodology
│       ├── threat-hunting-guide.md        # Hypothesis-driven hunting process
│       ├── detection-development-guide.md # Detection engineering process
│       ├── workflow-diagrams.md           # ASCII workflow diagrams for all major processes
│       └── raci-matrix.md                 # RACI matrix across all program roles
│
├── scenarios/
│   ├── _template.md                       # Canonical scenario authoring template
│   ├── _templates/                        # Specialized sub-templates by vector/application/vuln class
│   │   ├── cloud-attack-vector.md         # Pre-populated for cloud-based scenarios
│   │   ├── email-attack-vector.md         # Pre-populated for email-based scenarios
│   │   ├── endpoint-attack-vector.md      # Pre-populated for endpoint-based scenarios
│   │   ├── m365-application.md            # Pre-populated for Microsoft 365 scenarios
│   │   ├── aws-application.md             # Pre-populated for AWS scenarios
│   │   ├── credential-abuse-vuln-class.md # Pre-populated for credential abuse scenarios
│   │   └── data-access-abuse-vuln-class.md # Pre-populated for data access abuse scenarios
│   ├── data-exfiltration/                 # DE — unauthorized data removal
│   ├── privilege-misuse/                  # PM — abuse of legitimate access
│   ├── unauthorized-access/               # UA — access beyond authorization
│   ├── sabotage/                          # SB — intentional disruption or destruction
│   ├── fraud/                             # FR — financial manipulation or theft
│   ├── ip-theft/                          # IP — intellectual property targeting
│   ├── third-party-risk/                  # TP — vendor, contractor, and supply chain
│   ├── policy-violation/                  # PV — non-malicious non-compliant behavior
│   └── account-compromise/                # AC — insider account used by external actor
│
├── detections/
│   ├── _template.md                       # Detection rule authoring template
│   ├── sigma/                             # Platform-agnostic Sigma rules
│   └── queries/                           # Platform-specific queries (KQL, SPL, YARA-L)
│
├── playbooks/
│   ├── _template.md                       # Response playbook authoring template
│   └── examples/                          # Completed playbook examples
│
├── hunting/
│   ├── _template.md                       # Hunting hypothesis authoring template
│   └── hypotheses/                        # Documented hunting hypotheses
│
├── validation/
│   ├── _template.md                       # Test case authoring template
│   └── test-cases/                        # Validation test cases
│
└── mapping/
    ├── mitre-attack.md                    # MITRE ATT&CK technique mapping table
    └── compliance.md                      # Compliance framework mapping table
```

---

## Actor Archetypes

All SIDTVF scenarios are tagged to one or more of four insider threat actor types:

| Code | Archetype | Description |
|------|-----------|-------------|
| `MAL` | Malicious Insider | Intentional harm by an authorized user |
| `NEG` | Negligent Insider | Unintentional harm through careless behavior |
| `COM` | Compromised Insider | Legitimate account hijacked by an external actor |
| `TP` | Third-Party / Contractor | Vendor, contractor, or partner misusing access |

Full profiles and detection implications: [docs/actor-archetypes.md](docs/actor-archetypes.md)

---

## Scenario Categories

| Code | Category | Examples |
|------|----------|---------|
| DE | Data Exfiltration | Cloud uploads, email forwarding, USB transfers |
| PM | Privilege Misuse | Accessing systems beyond job scope, backdoor creation |
| UA | Unauthorized Access | Credential sharing, after-hours system access |
| SB | Sabotage & Disruption | Database deletion, service termination, ransomware deployment |
| FR | Fraud & Financial Crime | Invoice manipulation, approval bypassing, account takeover |
| IP | Intellectual Property Theft | Source code theft, proprietary research exfiltration |
| TP | Third-Party Risk | Vendor account abuse, supply chain manipulation |
| PV | Policy Violation | Shadow IT, personal device use, unapproved storage |
| AC | Account Compromise | External actor using stolen insider credentials |

---

## Maturity Model Summary

| Level | Name | Characteristics |
|-------|------|-----------------|
| 1 | Initial | No formal program. Fully reactive. Detections are accidental. |
| 2 | Developing | Basic monitoring tools deployed. Some policies documented. |
| 3 | Defined | Formal program with documented scenarios, playbooks, and processes. |
| 4 | Managed | Metrics-driven. Regular validation cycles. Feedback loops active. |
| 5 | Optimizing | Continuous improvement. Predictive analytics. Cross-org sharing. |

Self-assessment questionnaire and roadmap: [docs/maturity-model.md](docs/maturity-model.md)

---

## Quick Start Guide

### Step 1 — Assess Your Program Maturity
Start with [docs/maturity-model.md](docs/maturity-model.md) to complete the self-assessment questionnaire. This tells you where to focus first.

### Step 2 — Understand the Legal Landscape
Before deploying any monitoring, review [docs/legal-privacy.md](docs/legal-privacy.md) for applicable laws and compliance obligations in your jurisdiction.

### Step 3 — Choose a Starting Scenario
New programs should start with high-confidence, high-impact scenarios. Recommended starting scenarios:

| Priority | Scenario ID | Name | Why Start Here |
|----------|-------------|------|----------------|
| 1 | DE-001 | Cloud Storage Upload Exfiltration | High prevalence, well-logged |
| 2 | DE-002 | Email Auto-Forwarding Rule | Easy to detect, high impact |
| 3 | PM-001 | Privilege Escalation via Access Abuse | Common, often undetected |
| 4 | UA-001 | After-Hours Unauthorized System Access | High signal, low noise |

### Step 4 — Run Validation
Use [validation/test-cases/](validation/test-cases/) to test whether your existing controls would detect the scenario before deploying new detections.

### Step 5 — Deploy Detections
Use [detections/sigma/](detections/sigma/) for platform-agnostic Sigma rules or [detections/queries/](detections/queries/) for platform-specific queries (KQL, SPL, YARA-L).

### Step 6 — Run a Threat Hunt
Use [hunting/hypotheses/](hunting/hypotheses/) to proactively search for evidence of activity matching the scenario in your environment.

### Step 7 — Prepare a Response Playbook
Use [playbooks/examples/](playbooks/examples/) for scenario-specific response guidance and adapt to your environment.

### Step 8 — Measure and Iterate
Track outcomes using the KPIs defined in [metrics/kpis.md](metrics/kpis.md), then feed results back into the scenario library.

---

## Process Documentation

Step-by-step guides for each major framework activity:

| Process | Guide |
|---------|-------|
| Creating a new scenario | [docs/processes/scenario-authoring-guide.md](docs/processes/scenario-authoring-guide.md) |
| Running a validation cycle | [docs/processes/validation-cycle-guide.md](docs/processes/validation-cycle-guide.md) |
| Conducting a threat hunt | [docs/processes/threat-hunting-guide.md](docs/processes/threat-hunting-guide.md) |
| Developing a detection rule | [docs/processes/detection-development-guide.md](docs/processes/detection-development-guide.md) |
| Workflow diagrams (all processes) | [docs/processes/workflow-diagrams.md](docs/processes/workflow-diagrams.md) |
| RACI matrix (all roles) | [docs/processes/raci-matrix.md](docs/processes/raci-matrix.md) |

---

## Key Principles

| Principle | Description |
|-----------|-------------|
| Behavior over Signature | Detect what users do, not the specific tools they use |
| Scenario before Detection | Define the threat first, then write the rule |
| Validate Continuously | Controls are tested regularly, not just after deployment |
| Legal First | All program activity must be reviewed against applicable law and policy |
| Measure Everything | Every program activity should produce a measurable outcome |
| Cross-Team by Design | Outputs explicitly feed detection engineering, SOC, and IR teams |

---

## Included Examples

The following worked examples are included in this repository:

| Type | ID | Name |
|------|----|------|
| Scenario | DE-001 | Cloud Storage Upload Exfiltration |
| Scenario | DE-002 | Email Auto-Forwarding Rule |
| Scenario | PM-001 | Privilege Escalation via Access Abuse |
| Scenario | UA-001 | After-Hours Unauthorized System Access |
| Scenario | SB-001 | Targeted Database Deletion |
| Scenario | TP-001 | Vendor Credential Abuse |
| Scenario | FR-001 | Invoice and Vendor Payment Manipulation |
| Scenario | IP-001 | Source Code and Proprietary Algorithm Theft |
| Scenario | PV-001 | Shadow IT Data Storage |
| Scenario | AC-001 | Insider Account Compromise by External Actor |
| Detection (Sigma) | DET-DE-001-SIGMA | Cloud Storage Exfiltration Sigma Rule |
| Detection (KQL) | DET-DE-001-KQL | Cloud Storage Exfiltration KQL Queries |
| Detection (SPL) | DET-DE-001-SPL | Cloud Storage Exfiltration Splunk SPL Queries |
| Playbook | PB-DE-001 | Cloud Storage Exfiltration Response |
| Playbook | PB-SB-001 | Database Deletion (Insider Sabotage) Response |
| Hunt Hypothesis | HYP-DE-001 | Cloud Storage Exfiltration Hunt |
| Hunt Hypothesis | HYP-DE-002 | Email Auto-Forwarding Rule Hunt |
| Validation | TC-DE-001 | Cloud Storage Exfiltration Test Case |

---

## Standards Alignment

| Standard | Alignment |
|----------|-----------|
| MITRE ATT&CK | All scenarios mapped to tactics and techniques |
| NIST SP 800-53 | Control family alignment in compliance mapping |
| ISO 27001:2022 | Annex A control mapping |
| CMU CERT ITTP | Actor archetype alignment |
| NIST SP 800-171 | CUI protection alignment |
| SOC 2 | Trust Service Criteria mapping |

Full mappings: [mapping/mitre-attack.md](mapping/mitre-attack.md) | [mapping/compliance.md](mapping/compliance.md)

---

## License

MIT License — see [LICENSE](LICENSE) for full text.

---

## Disclaimer

SIDTVF is intended for **authorized defensive cybersecurity purposes only**. All scenario simulations and control validation activities must be conducted in authorized environments following applicable legal review. Organizations are responsible for ensuring their use of this framework complies with applicable laws, regulations, and internal policies. The authors assume no liability for misuse.
