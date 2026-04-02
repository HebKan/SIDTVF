# SIDTVF — Scenario-Driven Insider Threat Detection & Validation Framework

> **Version:** 1.1 | **Status:** Active | **License:** MIT

SIDTVF is an open methodology for building, testing, and continuously improving insider threat detection programs. It provides a common language, reusable templates, and measurable outcomes across all phases of insider threat operations.

---

## Framework Architecture

SIDTVF is organized into seven interdependent layers:

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
│  Layer 7  │ Simulation     │ Validate detections via behavioral tests │
└────────────────────────────┴─────────────────────────────────────────┘
```

Full specification: [docs/framework-overview.md](docs/framework-overview.md)

---

## Where to Start

**New to the framework?** → [docs/getting-started.md](docs/getting-started.md)

**Know your role? Jump directly to your component guide:**

| Role | Start here |
|------|-----------|
| Security engineer / program manager | [How to use scenario cards](docs/guides/using-scenario-cards.md) |
| Detection engineer | [How to deploy detection rules](docs/guides/using-detection-rules.md) |
| SOC analyst | [How to use triage cards](docs/guides/using-triage-cards.md) |
| Incident responder | [How to execute a playbook](docs/guides/using-playbooks.md) |
| Threat hunter | [How to run a threat hunt](docs/guides/running-threat-hunts.md) |
| Red team / control validator | [How to run validation tests](docs/guides/running-validation-tests.md) · [How to run simulation tests](docs/guides/running-simulation-tests.md) |
| Program manager / CISO | [How to track metrics and KPIs](docs/guides/tracking-metrics.md) · [How to use compliance mappings](docs/guides/using-mappings.md) |

---

## Repository Structure

```
SIDTVF/
├── docs/                   # Framework documentation, guides, policies, and references
│   ├── guides/             # Component how-to guides (start here for each artifact type)
│   ├── processes/          # Step-by-step authoring guides and workflow diagrams
│   ├── policies/           # Policy templates (program charter, monitoring notice, AUP)
│   └── starter-packs/      # Industry-specific quick-start guides
├── scenarios/              # Scenario cards by category + index.yaml
├── detections/             # Sigma rules, platform queries (KQL/SPL), and triage cards
├── playbooks/              # Incident response playbooks
├── hunting/                # Threat hunting hypotheses
├── validation/             # Control validation test cases
├── simulation/             # Behavioral simulation tests (Layer 7)
├── mapping/                # MITRE ATT&CK and compliance framework mappings
└── metrics/                # KPI definitions and measurement targets
```

Full artifact index: [scenarios/index.yaml](scenarios/index.yaml)

---

## Actor Archetypes

| Code | Archetype | Description |
|------|-----------|-------------|
| `MAL` | Malicious Insider | Intentional harm by an authorized user |
| `NEG` | Negligent Insider | Unintentional harm through careless behavior |
| `COM` | Compromised Insider | Legitimate account hijacked by an external actor |
| `TP` | Third-Party / Contractor | Vendor, contractor, or partner misusing access |

Details and detection implications: [docs/actor-archetypes.md](docs/actor-archetypes.md)

---

## Scenario Categories

| Code | Category | Examples |
|------|----------|---------|
| DE | Data Exfiltration | Cloud uploads, email forwarding, USB transfers |
| PM | Privilege Misuse | Accessing systems beyond job scope, backdoor creation |
| UA | Unauthorized Access | Credential sharing, after-hours system access |
| SB | Sabotage & Disruption | Database deletion, service termination |
| FR | Fraud & Financial Crime | Invoice manipulation, approval bypassing |
| IP | Intellectual Property Theft | Source code theft, proprietary research exfiltration |
| TP | Third-Party Risk | Vendor account abuse, supply chain manipulation |
| PV | Policy Violation | Shadow IT, personal device use, unapproved storage |
| AC | Account Compromise | External actor using stolen insider credentials |

---

## Maturity Model

| Level | Name | Characteristics |
|-------|------|-----------------|
| 1 | Initial | No formal program. Fully reactive. |
| 2 | Developing | Basic monitoring deployed. Some policies documented. |
| 3 | Defined | Formal program with scenarios, playbooks, and processes. |
| 4 | Managed | Metrics-driven. Regular validation cycles. Feedback loops active. |
| 5 | Optimizing | Continuous improvement. Cross-org sharing. |

Self-assessment: [docs/maturity-model.md](docs/maturity-model.md)

---

## Key Principles

| Principle | Description |
|-----------|-------------|
| Behavior over Signature | Detect what users do, not what tools they use |
| Scenario before Detection | Define the threat first, then write the rule |
| Method and Vector Matter | Physical/out-of-band methods (Bluetooth, USB, hardwire) produce no network log — severity and detection strategy must account for this |
| Validate Continuously | Test controls regularly, not just after deployment |
| Legal First | All monitoring must be reviewed against applicable law before deployment |
| Measure Everything | Every program activity should produce a measurable outcome |

---

## Documentation Index

| Document | Purpose |
|----------|---------|
| [docs/getting-started.md](docs/getting-started.md) | Day 1 / Week 1 / Month 1 onboarding |
| [docs/framework-overview.md](docs/framework-overview.md) | Full seven-layer architecture specification |
| [docs/severity-classification.md](docs/severity-classification.md) | Severity rating reference with impact/likelihood criteria |
| [docs/actor-archetypes.md](docs/actor-archetypes.md) | Four actor type definitions and detection implications |
| [docs/maturity-model.md](docs/maturity-model.md) | Maturity levels and self-assessment questionnaire |
| [docs/legal-privacy.md](docs/legal-privacy.md) | Legal and compliance guidance by jurisdiction |
| [docs/coverage-matrix.md](docs/coverage-matrix.md) | Artifact coverage status per scenario |
| [docs/tabletop-exercise-guide.md](docs/tabletop-exercise-guide.md) | 2-hour tabletop guide with inject cards |
| [docs/soar-integration.md](docs/soar-integration.md) | SOAR integration (Sentinel, Splunk SOAR, XSOAR) |
| [docs/executive-briefing-template.md](docs/executive-briefing-template.md) | Quarterly CISO/board briefing template |
| [docs/processes/raci-matrix.md](docs/processes/raci-matrix.md) | RACI matrix across all program roles |
| [docs/processes/workflow-diagrams.md](docs/processes/workflow-diagrams.md) | Workflow diagrams for all major processes |
| [mapping/mitre-attack.md](mapping/mitre-attack.md) | MITRE ATT&CK technique mapping |
| [mapping/compliance.md](mapping/compliance.md) | Compliance framework mapping (NIST, ISO, HIPAA, PCI, GDPR) |
| [metrics/kpis.md](metrics/kpis.md) | KPI definitions and measurement targets |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution guide and review process |

### Industry Starter Packs

| Industry | Guide |
|----------|-------|
| Healthcare and Life Sciences | [docs/starter-packs/healthcare.md](docs/starter-packs/healthcare.md) |
| Financial Services | [docs/starter-packs/finance.md](docs/starter-packs/finance.md) |
| Technology and Software | [docs/starter-packs/technology.md](docs/starter-packs/technology.md) |

### Policy Templates

| Template | File |
|----------|------|
| Insider Threat Program Charter | [docs/policies/program-charter.md](docs/policies/program-charter.md) |
| Employee Monitoring Notice | [docs/policies/monitoring-notice.md](docs/policies/monitoring-notice.md) |
| Acceptable Use Policy — AI Tools Addendum | [docs/policies/aup-addendum.md](docs/policies/aup-addendum.md) |

---

## Standards Alignment

| Standard | Alignment |
|----------|-----------|
| MITRE ATT&CK | All scenarios mapped to tactics and techniques |
| NIST SP 800-53 | Control family alignment in compliance mapping |
| ISO 27001:2022 | Annex A control mapping |
| CMU CERT ITTP | Actor archetype alignment |
| SOC 2 | Trust Service Criteria mapping |

---

## License

MIT License — see [LICENSE](LICENSE) for full text.

---

## Disclaimer

SIDTVF is intended for **authorized defensive cybersecurity purposes only**. All scenario simulations and control validation activities must be conducted in authorized environments following applicable legal review. Organizations are responsible for ensuring their use of this framework complies with applicable laws, regulations, and internal policies. The authors assume no liability for misuse.
