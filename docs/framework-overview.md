# SIDTVF Framework Overview

> **Document Version:** 1.1 | **Status:** Active | **Last Updated:** 2026-03-27

---

## 1. Introduction

The **Scenario-Driven Insider Threat Detection and Validation Framework (SIDTVF)** is a structured, open methodology for building and continuously improving insider threat detection programs. It provides security teams with a common language, a consistent artifact structure, and measurable outcomes across the full lifecycle of insider threat operations.

SIDTVF is not a product, a tool, or a compliance checklist. It is a **methodology** — a way of thinking about, organizing, and executing insider threat work so that the outputs of one team become the inputs of another, and so that the program improves with every cycle.

### Design Philosophy

SIDTVF is built on the following design choices:

1. **Behavior is the unit of analysis.** Insider threats are defined by what a person does, not by what tool they use. A threat that is tied to a specific tool disappears when the tool changes; a threat tied to a behavior persists.

2. **Scenarios precede detections.** Writing a detection rule before defining the threat behavior produces rules that are brittle, poorly understood, and unmeasurable. SIDTVF enforces a scenario-first workflow.

3. **Validation is not optional.** A detection rule that has never been tested against a real scenario provides false confidence. SIDTVF treats validation as a first-class activity, not an afterthought.

4. **Legal review is a prerequisite, not a follow-up.** Insider threat programs that bypass legal review have been shut down, faced litigation, or caused significant organizational harm. SIDTVF embeds legal and privacy checkpoints at the program level and at the scenario level.

5. **Feedback loops create improvement.** Detection gaps, hunting findings, and incident outcomes are all data that should flow back into the scenario library. Without this feedback, the program stagnates.

6. **Measurement enables justification.** Security teams that cannot measure their program's effectiveness cannot defend its budget, explain its value to leadership, or know when it is failing.

---

## 2. Framework Layers

SIDTVF is structured into seven layers. Each layer has defined inputs, activities, and outputs. Outputs from earlier layers become inputs to later layers. The feedback loop at Layer 6 returns outputs to Layer 1, creating a continuous improvement cycle. Layer 7 (Simulation) runs alongside the full cycle, providing behavioral evidence that validates detection coverage.

```
 ┌─────────────────────────────────────────────────────────────┐
 │                                                             │
 │   Layer 1: Scenario           ──► Layer 2: Validation       │
 │                                         │                   │
 │   Layer 6: Feedback Loop ◄──── Layer 3: Threat Hunting      │
 │         │                               │                   │
 │         ▼                      Layer 4: Integration         │
 │   Layer 1 (updated)                     │                   │
 │                               Layer 5: Mapping              │
 │                                                             │
 │   Layer 7: Simulation  ──► (feeds Layers 2, 3, and 6)       │
 │                                                             │
 └─────────────────────────────────────────────────────────────┘
```

---

### Layer 1 — Scenario

**Purpose:** Define realistic insider threat behaviors that the program aims to detect and respond to.

**Inputs:** Threat intelligence, past incidents, industry reporting, MITRE ATT&CK, organizational risk assessments.

**Activities:**
- Identify insider threat scenarios relevant to the organization's industry, data types, and access model
- Classify each scenario by actor archetype (Malicious, Negligent, Compromised, Third-Party)
- Document the attack phases: initial conditions, behavior chain, and potential impact
- Identify indicators of behavior (IOBs) — observable actions that distinguish threat activity from normal work
- Map scenarios to MITRE ATT&CK techniques
- Assign severity and priority

**Outputs:**
- Completed scenario cards (using `scenarios/_template.md`)
- Behavior indicator lists
- ATT&CK technique tags
- Severity ratings

**Key Principle:** Scenarios describe *what the insider does*, not what tool they use. Tools change. Behaviors persist.

---

### Layer 2 — Control Validation

**Purpose:** Test whether existing security controls would detect or prevent the behaviors defined in Layer 1.

**Inputs:** Scenario cards from Layer 1, inventory of existing controls and monitoring capabilities.

**Activities:**
- Design test cases for each scenario that simulate the defined behavior in an authorized, controlled environment
- Execute test cases and record whether the expected alert, log, or prevention fired
- Classify outcomes: Detected, Partially Detected, Not Detected, Prevented, False Positive Generated
- Identify root causes for detection failures (missing log source, misconfigured rule, coverage gap, etc.)

**Outputs:**
- Validation test cases (using `validation/_template.md`)
- Validation reports with outcome classification
- Control gap analysis
- Prioritized list of detection improvement opportunities

**Key Principle:** If you have not tested a control against a defined scenario, you do not know whether it works.

---

### Layer 3 — Threat Hunting

**Purpose:** Proactively search production environments for evidence of insider threat activity that existing detections may have missed.

**Inputs:** Scenario cards from Layer 1, validation gaps from Layer 2, historical data from SIEM/XDR/DLP.

**Activities:**
- Form a hypothesis: "If an insider were executing [scenario], what evidence would be visible in [data source]?"
- Develop queries to test the hypothesis
- Execute the hunt and document findings
- Classify findings: True Threat Activity, Suspicious (Unconfirmed), Policy Violation, False Positive, Clean
- Feed confirmed and suspicious findings to incident response

**Outputs:**
- Documented hunt hypotheses (using `hunting/_template.md`)
- Hunt queries (SIEM/XDR queries, log correlation logic)
- Hunt findings and anomalies
- Escalation inputs for incident response

**Key Principle:** Threat hunting assumes that some threats have already bypassed existing controls. Its purpose is to find what detection missed.

---

### Layer 4 — Integration

**Purpose:** Ensure that findings from Layers 2 and 3 are consumed and actioned by the appropriate downstream teams.

**Inputs:** Control gaps (Layer 2), hunt findings (Layer 3), confirmed threat activity (Layer 3).

**Downstream Consumers and Their Inputs:**

| Consumer | What They Receive | What They Do |
|----------|--------------------|--------------|
| Detection Engineering | Control gaps, detection improvement opportunities | Implement or improve detection rules; deploy to SIEM/XDR/DLP |
| Security Operations / SOC | Validated alert patterns, hunt findings | Improve triage accuracy; update alert response procedures |
| Incident Response | Confirmed or suspected threat activity, playbooks | Investigate; contain; remediate; report |
| Legal and HR | Confirmed threat findings with evidence documentation | Advise on response; participate in investigation as required |
| Risk Management | Control gap reports, program metrics | Update risk registers; adjust risk appetite and controls |

**Key Principle:** Findings that are not consumed and actioned have no value. Integration is what converts security activity into security outcomes.

---

### Layer 5 — Mapping and Standardization

**Purpose:** Normalize all framework artifacts to common industry standards, enabling consistent communication and measurable coverage.

**Mapping Targets:**

| Standard | What It Enables |
|----------|----------------|
| MITRE ATT&CK | Common technique vocabulary; detection coverage measurement |
| NIST SP 800-53 | Control mapping for federal and regulated environments |
| ISO 27001:2022 | Annex A control alignment for certification programs |
| SOC 2 | Trust Service Criteria mapping for service organizations |
| HIPAA Security Rule | PHI-specific control alignment |
| PCI DSS | Cardholder data environment alignment |
| GDPR / UK GDPR | Privacy-preserving monitoring requirements |
| CMU CERT ITTP | Actor archetype and attack pattern alignment |

**Outputs:**
- MITRE ATT&CK technique mapping table: [mapping/mitre-attack.md](../mapping/mitre-attack.md)
- Compliance control mapping table: [mapping/compliance.md](../mapping/compliance.md)

**Key Principle:** Standardization is not bureaucracy — it is the mechanism by which a team's work becomes intelligible to another team, another organization, or an auditor.

---

### Layer 7 — Behavioral Simulation

**Purpose:** Execute authorized, behavior-based simulation tests that validate whether detection rules fire as expected against realistic insider threat activity patterns — using legitimate native tools and synthetic data only.

**Inputs:** Active scenario cards (Layer 1), detection rules (Layer 2 outputs), validation gaps (Layer 2), authorization checklist (required pre-condition).

**Activities:**
- Design simulation tests that replicate the behavioral pattern defined in a scenario card (volume, timing, sequence, target) — not technique execution
- Generate synthetic data that mimics realistic data types without containing real PII, PHI, MNPI, or credentials
- Execute tests in authorized non-production or near-production environments and record which detections fired
- Measure observation windows (real-time vs. ingestion-delay paths) and identify detection timing gaps
- Feed simulation results to Layer 2 (validation) and Layer 6 (feedback) to close coverage gaps

**Outputs:**
- Completed simulation test files (using `simulation/_template.md`)
- Detection fire/no-fire evidence per rule per scenario
- Timing gap analysis (observation window results)
- Blind spot documentation — what the simulation confirmed was NOT detected

**Key Principle:** Simulation tests behavioral patterns (volume/timing/sequence/target), not tool-specific technique execution. A simulation that requires malicious software is not a SIDTVF simulation test.

---

### Layer 6 — Feedback and Continuous Improvement

**Purpose:** Ensure that operational experience — from detections, hunts, and incidents — continuously improves the scenario library and program posture.

**Feedback Inputs:**

| Source | Feedback Type |
|--------|--------------|
| Detection failures | Scenarios where alerts did not fire |
| False positive analysis | Scenarios generating too much noise; need tuning |
| Hunt findings | New behaviors not yet captured in scenarios |
| Incident outcomes | Confirmed attack paths that may become new scenarios |
| Near-miss analysis | Events that were almost missed, suggesting coverage gaps |
| Red team / purple team exercises | Simulated attack paths exposing detection blindspots |

**Activities:**
- Review feedback inputs quarterly (at minimum)
- Update existing scenario cards when new behavior variants are identified
- Create new scenarios when hunts or incidents reveal uncatalogued behaviors
- Retire scenarios that are no longer relevant to the organization's threat landscape
- Adjust KPI targets based on program maturity progression

**Outputs:**
- Updated or new scenario cards (returned to Layer 1)
- Revised detection rules (returned to Layer 2 for re-validation)
- Updated program metrics and maturity assessment

**Key Principle:** A program that does not improve from its own experience will be overtaken by the threat actors that study theirs.

---

## 3. Artifact Types and Relationships

SIDTVF produces the following artifact types. Each has a unique ID, follows a defined template, and links explicitly to related artifacts.

```
Scenario Card ──────────────────────────────────────────────────┐
     │                                                           │
     ├──► Validation Test Case (TC-[ID])                        │
     │         └──► Validation Report                           │
     │                                                           │
     ├──► Detection Rule (DET-[ID]-[PLATFORM])                  │
     │         ├── Sigma Rule (.yml)                            │
     │         ├── Platform Queries (.md)                       │
     │         └── SOC Triage Card (TRIAGE-[ID])               │
     │                                                           │
     ├──► Hunt Hypothesis (HYP-[ID])                            │
     │         └──► Hunt Findings                               │
     │                                                           │
     ├──► Response Playbook (PB-[ID])                           │
     │         └──► Incident Report                             │
     │                                                           │
     └──► Simulation Test (SIM-[ID])                            │
               └──► Detection Gap Evidence                      │
                                                                 │
All artifacts ──────────────────────────────────────────────────►
     ├──► MITRE ATT&CK Mapping
     └──► Compliance Mapping
```

---

## 4. Governance Model

### Roles and Responsibilities

SIDTVF is designed to function across multiple security teams. The following roles are defined for framework governance:

| Role | Responsibility |
|------|---------------|
| Framework Owner | Maintains framework structure, templates, and versioning. Approves structural changes. |
| Scenario Authors | Subject matter experts who author and maintain scenario cards. |
| Detection Engineers | Translate scenario behavior indicators into detection rules. Validate detection coverage. |
| Threat Hunters | Execute hypothesis-driven hunts against scenarios and feed findings back. |
| Incident Responders | Use playbooks for scenario-matched response; feed incident outcomes back to scenario authors. |
| Legal / Privacy Counsel | Advise on monitoring legality at program setup and when new scenarios involve sensitive data categories. |
| Program Metrics Owner | Collects, tracks, and reports KPIs; facilitates maturity assessments. |

### Scenario Lifecycle

```
Draft ──► Peer Review ──► Legal Review ──► Active ──► Deprecated
```

- **Draft:** Scenario is being authored. Not yet used for detection or hunting.
- **Peer Review:** Scenario reviewed by at least one other security team member for technical accuracy.
- **Legal Review:** Scenario reviewed by legal/privacy counsel if it involves monitoring of communication, HR-sensitive data, or PII.
- **Active:** Scenario is approved for use. Linked artifacts (detections, playbooks, hunt hypotheses) can be created.
- **Deprecated:** Scenario is retired. File is preserved with deprecation reason. ID is never reused.

### Version Control

- All artifacts are version-controlled in this repository
- Use semantic versioning (MAJOR.MINOR) per artifact
- Commit messages should reference artifact IDs (e.g., `Update DE-001: add lateral movement phase`)
- Breaking changes to templates require a major version bump and cascade updates to all existing artifacts

---

## 5. ID and Naming Conventions

See [CLAUDE.md](../CLAUDE.md) for the full, authoritative naming convention reference.

**Summary:**

| Artifact | ID Format | Example |
|----------|-----------|---------|
| Scenario | `[CAT]-[NNN]` | `DE-001` |
| Detection | `DET-[CAT]-[NNN]-[PLATFORM]` | `DET-DE-001-SIGMA` |
| Playbook | `PB-[CAT]-[NNN]` | `PB-DE-001` |
| Hunt Hypothesis | `HYP-[CAT]-[NNN]` | `HYP-DE-001` |
| Validation Test Case | `TC-[CAT]-[NNN]` | `TC-DE-001` |
| Simulation Test | `SIM-[CAT]-[NNN]` | `SIM-DE-001` |
| SOC Triage Card | `TRIAGE-[CAT]-[NNN]` | `TRIAGE-DE-001` |

---

## 6. Metrics and Measurement

SIDTVF programs should track the following metrics. Full KPI definitions, formulas, and targets are in [metrics/kpis.md](../metrics/kpis.md).

| Metric | Description | Target |
|--------|-------------|--------|
| Detection Coverage Rate | % of active scenarios with at least one validated detection | ≥ 80% |
| Validation Pass Rate | % of validation test cases where detection fired as expected | ≥ 90% |
| Time to Detect (TTD) | Mean time from behavior onset to alert generation | < 24 hours |
| False Positive Rate | Alerts generated per true positive | < 10:1 |
| Scenario Currency | % of active scenarios updated within the last 12 months | 100% |
| Mean Time to Respond (MTTR) | Mean time from alert to containment decision | < 4 hours |
| Hunting Hypothesis Conversion Rate | % of hypotheses resulting in confirmed or suspicious findings | Track as baseline |
| Control Gap Closure Rate | % of identified gaps addressed within 90 days | ≥ 70% |

---

## 7. Relationship to Other Frameworks

SIDTVF is designed to complement, not replace, existing security frameworks.

| Framework | Relationship to SIDTVF |
|-----------|----------------------|
| MITRE ATT&CK | SIDTVF scenarios are mapped to ATT&CK techniques. ATT&CK is the common vocabulary for technique-level communication. |
| CMU CERT ITTP | SIDTVF's actor archetypes align with CMU CERT's insider threat taxonomy. CERT research informs scenario development. |
| NIST SP 800-53 | SIDTVF scenarios can be mapped to NIST controls for compliance gap analysis. |
| NIST CSF | SIDTVF's validation and feedback loops align with the CSF's Detect, Respond, and Recover functions. |
| ISO 27001:2022 | SIDTVF supports A.6 (People), A.8 (Technology), and A.5 (Organizational) controls. |
| SANS Threat Hunting Framework | SIDTVF's hunting layer aligns with SANS hypothesis-driven hunting methodology. |

---

## 8. Changelog

| Version | Date | Description |
|---------|------|-------------|
| 1.1 | 2026-03-27 | Added Layer 7 (Behavioral Simulation); updated artifact diagram to include SIM and TRIAGE types; added simulation test ID conventions |
| 1.0 | 2026-03-25 | Initial framework release |
