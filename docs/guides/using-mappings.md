# Guide: Using MITRE ATT&CK and Compliance Mappings

SIDTVF maintains two cross-reference tables: a MITRE ATT&CK technique mapping and a compliance framework mapping. This guide explains how to read and apply each.

---

## MITRE ATT&CK Mapping

File: `mapping/mitre-attack.md`

### What it contains

A table mapping each SIDTVF scenario to one or more MITRE ATT&CK Enterprise techniques. Each row shows:
- Scenario ID and name
- Primary tactic (e.g., Exfiltration, Impact, Credential Access)
- Technique ID(s) and names
- Sub-technique IDs where applicable

### How to use it

**Coverage gap analysis:** Sort by technique to identify which ATT&CK techniques have no corresponding SIDTVF scenario. These are detection blind spots at the technique level. Prioritize scenario authoring for high-prevalence techniques without coverage.

**Detection rule tagging:** When deploying detection rules in your SIEM, tag each rule with its ATT&CK technique ID. This enables ATT&CK Navigator overlays showing which techniques are actively monitored.

**Purple team communication:** Share the ATT&CK mapping with your red team or purple team. They can plan exercises targeting techniques where your scenario coverage or detection coverage is weakest.

**Regulatory alignment:** Several compliance frameworks (NIST SP 800-53, CMMC) reference ATT&CK techniques directly. Use the mapping to connect SIDTVF scenarios to control requirements.

### Keeping it current

Update the ATT&CK mapping when:
- A new scenario is added (add a row)
- ATT&CK releases a new version with relevant new techniques or sub-techniques (update technique IDs and names)
- A scenario's attack phases are revised significantly (update the technique mapping)

Current ATT&CK version is noted at the top of the mapping file.

---

## Compliance Framework Mapping

File: `mapping/compliance.md`

### What it contains

A matrix mapping each SIDTVF scenario to specific controls across multiple compliance frameworks:
- NIST SP 800-53
- ISO 27001:2022 (Annex A)
- HIPAA Security Rule
- PCI DSS v4.0
- SOC 2 Trust Service Criteria
- GDPR (relevant articles)

### How to use it

**Compliance gap analysis:** Filter by framework. For each scenario that applies to your industry, the mapped controls represent the baseline you should be able to demonstrate to auditors.

**Audit preparation:** When an auditor asks "how do you address [control]?", look up the control in the compliance mapping to find the SIDTVF scenarios that address it. Your scenario cards, detection rules, and playbooks are your evidence.

**Risk register updates:** The mapping connects insider threat scenarios to specific control failures. Use this to populate your risk register with named controls rather than vague "insider threat risk" entries.

**Scope prioritization:** Healthcare organizations should focus on HIPAA-mapped scenarios first. Financial institutions should focus on SOX and GLBA mappings. Technology companies should focus on DTSA-relevant scenarios. Use the mapping to filter down to what matters for your industry.

---

## Interpreting Partial Mappings

Not every scenario maps to every compliance framework, and not every control maps to a specific scenario. Gaps in the mapping table mean one of two things:

1. **The scenario is not relevant to that framework** — e.g., a sabotage scenario (SB-001) has limited PHI implications if the destroyed system doesn't hold patient data
2. **The mapping has not been completed yet** — check `docs/coverage-matrix.md` to see which mappings are marked as in-progress

If you identify a missing mapping that should exist, add it per the contribution guide (`CONTRIBUTING.md`).

---

## Using Mappings for Board Reporting

When presenting to a board or audit committee:
1. Pull the compliance mapping filtered to your primary regulatory framework
2. Map each covered scenario to its control
3. Present the coverage matrix (`docs/coverage-matrix.md`) showing which scenarios have active detections vs. gaps
4. Use the executive briefing template (`docs/executive-briefing-template.md`) to structure the presentation

---

## Related

- ATT&CK mapping: [mapping/mitre-attack.md](../../mapping/mitre-attack.md)
- Compliance mapping: [mapping/compliance.md](../../mapping/compliance.md)
- Coverage matrix: [docs/coverage-matrix.md](../coverage-matrix.md)
- Legal and privacy guidance: [docs/legal-privacy.md](../legal-privacy.md)
- Executive briefing template: [docs/executive-briefing-template.md](../executive-briefing-template.md)
