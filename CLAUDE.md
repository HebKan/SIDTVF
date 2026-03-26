# CLAUDE.md — AI Maintenance Instructions for SIDTVF

This file instructs Claude (or any AI assistant) on how to maintain consistency across the SIDTVF framework when changes are requested. Read this file before modifying any part of the repository.

---

## Repository Purpose

SIDTVF is an insider threat detection and validation framework designed for use by security teams worldwide. It must remain:

- **Internally consistent** — terminology, IDs, and cross-references must match across all files
- **Template-compliant** — all examples must follow the canonical templates in `_template.md` files
- **Practically grounded** — all scenarios, detections, and playbooks should reflect realistic enterprise environments
- **Legally aware** — content must not prescribe specific legal actions; it must present options with clear jurisdiction guidance

---

## File Hierarchy and Ownership

| File / Directory | Purpose | When to Edit |
|-----------------|---------|--------------|
| `README.md` | Navigation hub and quick start | When structure, scenario list, or examples change |
| `CLAUDE.md` | This file | When authoring conventions or framework structure changes |
| `docs/framework-overview.md` | Authoritative framework spec | When layers, principles, or governance model changes |
| `docs/actor-archetypes.md` | Four actor type definitions | When actor types or sub-types are added or changed |
| `docs/maturity-model.md` | Maturity levels and self-assessment | When maturity criteria or domains are revised |
| `docs/legal-privacy.md` | Legal and compliance guidance | When regulations change or new jurisdictions are added |
| `docs/processes/*.md` | Step-by-step process guides and RACI matrix | When process steps, workflow diagrams, or RACI ownership changes |
| `scenarios/_template.md` | Canonical scenario schema | When scenario fields are added or removed |
| `scenarios/_templates/*.md` | Specialized sub-templates (attack vector, application, vuln class) | When a new sub-template type is added or fields change |
| `scenarios/*/**.md` | Individual scenario files | When a specific scenario is added, updated, or deprecated |
| `detections/_template.md` | Detection rule schema | When detection fields are added or removed |
| `detections/sigma/*.yml` | Sigma detection rules | When detection logic changes or new rules are added |
| `detections/queries/*.md` | Platform-specific queries | When query logic changes for KQL, SPL, YARA-L |
| `playbooks/_template.md` | Response playbook schema | When playbook fields are added or removed |
| `playbooks/examples/*.md` | Full playbook examples | When response procedures change |
| `hunting/_template.md` | Hunting hypothesis schema | When hypothesis fields are added or removed |
| `hunting/hypotheses/*.md` | Individual hunt hypotheses | When hypotheses are added, updated, or closed |
| `validation/_template.md` | Test case schema | When test case fields are added or removed |
| `validation/test-cases/*.md` | Validation test cases | When test procedures or expected results change |
| `mapping/mitre-attack.md` | ATT&CK technique mapping | When scenarios are added or ATT&CK version is updated |
| `mapping/compliance.md` | Compliance framework mapping | When scenarios or compliance controls change |
| `metrics/kpis.md` | KPI definitions | When metrics or measurement methods change |

---

## Naming Conventions

### Scenario IDs

Format: `[CATEGORY_CODE]-[NNN]`

| Category | Code | Example |
|----------|------|---------|
| Data Exfiltration | DE | DE-001 |
| Privilege Misuse | PM | PM-001 |
| Unauthorized Access | UA | UA-001 |
| Sabotage & Disruption | SB | SB-001 |
| Fraud & Financial | FR | FR-001 |
| Intellectual Property Theft | IP | IP-001 |
| Third-Party Risk | TP | TP-001 |
| Policy Violation | PV | PV-001 |
| Account Compromise | AC | AC-001 |

Numbers are zero-padded to three digits and assigned sequentially. Never reuse a retired ID. Use `Status: Deprecated` instead.

### Detection IDs

Format: `DET-[SCENARIO_ID]-[PLATFORM]`

Examples:
- `DET-DE-001-SIGMA` — Sigma rule for DE-001
- `DET-DE-001-KQL` — KQL query for DE-001
- `DET-DE-001-SPL` — Splunk SPL for DE-001

### Playbook IDs

Format: `PB-[SCENARIO_ID]`

Example: `PB-DE-001`

### Hunting Hypothesis IDs

Format: `HYP-[SCENARIO_ID]`

Example: `HYP-DE-001`

### Validation Test Case IDs

Format: `TC-[SCENARIO_ID]`

Example: `TC-DE-001`

### File Names

- Scenario files: `[SCENARIO_ID]-[short-slug].md` (e.g., `DE-001-cloud-storage-upload.md`)
- Detection Sigma: `[SCENARIO_ID]-[short-slug].yml` (e.g., `DE-001-cloud-storage-upload.yml`)
- Detection queries: `[SCENARIO_ID]-[platform].md` (e.g., `DE-001-kql.md`)
- Playbook files: `PB-[SCENARIO_ID]-[short-slug].md` (e.g., `PB-DE-001-cloud-storage-exfil.md`)
- Hypothesis files: `HYP-[SCENARIO_ID].md` (e.g., `HYP-DE-001.md`)
- Test case files: `TC-[SCENARIO_ID].md` (e.g., `TC-DE-001.md`)

---

## Actor Archetype Codes

Always use the following codes exactly when tagging scenarios:

| Code | Archetype |
|------|-----------|
| `MAL` | Malicious Insider |
| `NEG` | Negligent Insider |
| `COM` | Compromised Insider |
| `TP` | Third-Party / Contractor |

Scenarios may have multiple actor codes if applicable (e.g., `MAL, TP`).

---

## Consistency Rules

When a change is made in one file, the following cascade updates are required:

### If a new scenario is added:

1. Create the scenario file in the correct `scenarios/[category]/` directory using `scenarios/_template.md`
2. Add the scenario to `README.md` under the "Included Examples" table
3. Add a row to `mapping/mitre-attack.md` with the MITRE technique mapping
4. Add a row to `mapping/compliance.md` if the scenario has clear compliance implications
5. Create corresponding artifacts using templates:
   - `detections/_template.md` → new detection file
   - `hunting/_template.md` → new hypothesis file
   - `validation/_template.md` → new test case file
   - `playbooks/_template.md` → new playbook file
6. Update `metrics/kpis.md` if the scenario introduces a new measurement category

### If a scenario is modified:

1. Update the scenario file
2. Update the `Version` field and `Last Updated` date in the scenario card
3. Update all linked detection, hunting, validation, and playbook files if the behavior or detection logic changed
4. Update `mapping/mitre-attack.md` if the ATT&CK mapping changed
5. Update `README.md` if the scenario name changed

### If a scenario is deprecated:

1. Set `Status: Deprecated` in the scenario file header — do NOT delete the file
2. Add a `Deprecation Reason` field explaining why
3. Add a `Superseded By` field if a replacement scenario exists
4. Remove or update references in `README.md`, `mapping/mitre-attack.md`, and `mapping/compliance.md`

### If a template field is added or removed:

1. Update the relevant `_template.md` file
2. Update ALL existing scenario/detection/playbook/hunting/validation files to include or remove that field
3. Update this `CLAUDE.md` file if the convention section needs to change
4. Update `docs/framework-overview.md` if the change affects the framework architecture

### If the maturity model changes:

1. Update `docs/maturity-model.md`
2. Review all scenario files — each scenario has a `Minimum Maturity Level` field that may need updating
3. Update `README.md` maturity model summary table

### If a new compliance framework is added:

1. Add a column to `mapping/compliance.md`
2. Update `docs/legal-privacy.md` if the framework has privacy implications
3. Update `README.md` Standards Alignment table

---

## Content Standards

### Scenario Narratives

- Narrative case studies must be plausible in a real enterprise setting
- Do not use real company names, real person names, or real breach data
- Use generic but realistic job titles (e.g., "a senior engineer in the Infrastructure team")
- Narratives should read clearly to a non-technical security manager as well as a detection engineer
- Length: 3–6 paragraphs

### Detection Logic

- All Sigma rules must include: `title`, `id` (UUID), `status`, `description`, `logsource`, `detection`, `falsepositives`, and `level`
- KQL and SPL queries must include comments explaining each clause
- Always note the data source required (e.g., "Requires Microsoft Defender for Endpoint" or "Requires Cloudtrail logging enabled")

### Playbooks

- Response steps must be sequenced in a defined order with numbered steps
- Each step must identify the responsible team (e.g., `[SOC]`, `[IR]`, `[Legal]`, `[HR]`)
- Preserve all legal and HR decision gates — never auto-escalate to termination or law enforcement
- Playbooks must include a "False Positive Considerations" section

### Legal Content

- Never prescribe legal advice — use hedging language ("organizations should consult legal counsel")
- Always present options rather than mandates when legal requirements vary by jurisdiction
- When referencing laws, use full name on first reference, then abbreviation (e.g., "Electronic Communications Privacy Act (ECPA)")

---

## Version Tracking

Each scenario, detection, playbook, and hypothesis file contains a `Version` field. Follow semantic versioning (major.minor):

- **Major version bump (e.g., 1.0 → 2.0):** Breaking change to behavior description, detection logic, or response procedure
- **Minor version bump (e.g., 1.0 → 1.1):** Clarifications, additional context, additional detection queries, or MITRE mapping updates

The framework-level version (shown in `README.md` and `docs/framework-overview.md`) follows the same convention and is bumped when the structure, templates, or core principles change.

---

## What NOT to Do

- Do not delete scenario or detection files — deprecate them instead
- Do not change scenario IDs once assigned — IDs are permanent identifiers
- Do not add real threat intelligence (IOCs, specific actor names, real breach details)
- Do not include specific vendor product names as the only solution — use generic descriptions with product examples
- Do not prescribe legal actions — always recommend consulting legal counsel
- Do not combine multiple scenarios into one file — one file per scenario
- Do not write detection rules that could enable surveillance beyond the scope of insider threat detection
