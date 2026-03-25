# Scenario Card Template

> Copy this file to the appropriate category directory and rename it `[SCENARIO-ID]-[short-slug].md`
> Remove all instructional comments (lines beginning with `>`) before publishing.
> See [CLAUDE.md](../CLAUDE.md) for naming conventions and lifecycle rules.

---

## Scenario Card

| Field | Value |
|-------|-------|
| **Scenario ID** | [e.g., DE-001] |
| **Name** | [Short, descriptive name] |
| **Version** | 1.0 |
| **Status** | Draft / Active / Deprecated |
| **Created** | [YYYY-MM-DD] |
| **Last Updated** | [YYYY-MM-DD] |
| **Author** | [Name or team] |
| **Category** | [Full category name, e.g., Data Exfiltration] |
| **Category Code** | [Code, e.g., DE] |
| **Actor Archetype(s)** | [MAL / NEG / COM / TP — select all that apply] |
| **Severity** | Critical / High / Medium / Low |
| **Minimum Maturity Level** | [1–5 — minimum program maturity required to detect this scenario] |

> **Severity Guide:**
> - **Critical:** High likelihood, catastrophic impact (trade secret theft, mass PII exfiltration, sabotage)
> - **High:** High likelihood or high impact (significant data loss, major compliance exposure)
> - **Medium:** Moderate likelihood or impact (policy violations with meaningful consequence)
> - **Low:** Low likelihood or low impact (minor policy deviation, unlikely to cause lasting harm)

---

## 1. Description

> One to three sentences. Describe the threat behavior in plain language: who is doing what, to what data or system, with what likely impact. This description must be understandable to a non-technical security manager.

[Description here]

---

## 2. Narrative Case Study

> Write a realistic case study of this scenario playing out in a real enterprise environment. Use a fictional company and fictional individuals. 3–6 paragraphs. Describe the insider's situation, motivation, how the activity unfolded, how (or whether) it was detected, and the outcome. This narrative helps teams visualize the scenario and understand its context.

[Narrative case study here]

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | [MAL / NEG / COM / TP] |
| **Typical Role** | [e.g., Engineer, Finance Manager, Contractor, System Administrator] |
| **Access Level** | [Standard User / Power User / Privileged User / Administrator] |
| **Technical Sophistication** | Low / Medium / High |
| **Motivation** | [e.g., Financial gain, disgruntlement, carelessness, external coercion] |
| **Insider Knowledge Used** | [What organizational knowledge makes this scenario possible — e.g., "knows which data stores hold customer PII"] |

---

## 4. Attack Phases

> Describe the scenario as a sequential series of phases. Each phase represents a distinct stage in the insider's behavior chain. Label each phase with a name. Include a brief description and the observable actions (what would be visible in logs or behavior) for each phase.

### Phase 1 — [Phase Name]

**Description:** [What is happening in this phase]

**Observable Actions:**
- [Observable action 1]
- [Observable action 2]

### Phase 2 — [Phase Name]

**Description:** [What is happening in this phase]

**Observable Actions:**
- [Observable action 1]
- [Observable action 2]

### Phase 3 — [Phase Name]

**Description:** [What is happening in this phase]

**Observable Actions:**
- [Observable action 1]
- [Observable action 2]

> Add additional phases as needed. Most scenarios have 3–6 phases.

---

## 5. Indicators of Behavior (IOBs)

> IOBs are observable signals that, individually or in combination, indicate that this scenario may be occurring. List them as discrete, observable events — not inferences. IOBs can be technical (log events) or contextual (HR signals).

### Technical IOBs

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| [Observable log event or behavior] | [Where this is visible — e.g., SIEM, DLP, EDR] | High / Medium / Low |

> Signal Strength:
> - **High** — Strong indication of this specific behavior; low base rate in normal operations
> - **Medium** — Possible indication; requires correlation with other signals
> - **Low** — Weak signal; high false positive rate; useful only in combination

### Contextual IOBs

> Contextual IOBs are non-technical signals from HR, management, or other sources that increase the likelihood of insider threat activity. These are not grounds for monitoring on their own but are important context for investigations.

| IOB | Source | Notes |
|-----|--------|-------|
| [Contextual signal, e.g., "Employee gave notice two weeks ago"] | [HR / Management / Access review] | [How to use this signal] |

---

## 6. Detection Opportunities

> For each attack phase, identify where detection could occur and what data source would enable it.

| Phase | Detection Opportunity | Required Data Source | Detection Confidence |
|-------|-----------------------|---------------------|---------------------|
| [Phase name] | [What to look for] | [Log source, tool] | High / Medium / Low |

---

## 7. MITRE ATT&CK Mapping

> Map each phase or behavior to the most specific applicable MITRE ATT&CK technique. Use the current version of ATT&CK Enterprise. Include tactic and technique. Sub-techniques are preferred where applicable.

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| [e.g., Collection] | [e.g., T1213] | [e.g., Data from Information Repositories] | [Phase name] |

---

## 8. Legal and Privacy Considerations

> Identify specific legal or privacy considerations relevant to detecting or investigating this scenario. These should be scenario-specific, not generic. Reference [docs/legal-privacy.md](../docs/legal-privacy.md) for general guidance.

- [Specific consideration 1, e.g., "Email content monitoring may require specific consent language in US states with two-party consent laws"]
- [Specific consideration 2]

---

## 9. Response Guidance

> High-level response priorities for this scenario. The full response playbook is a separate artifact. This section provides the immediate decision framework.

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | [Immediate first action] | [SOC / IR / Legal / HR] |
| 2 | [Second action] | [Owner] |
| 3 | [Third action] | [Owner] |

**Linked Playbook:** [PB-XX-NNN or "Not yet created"]

---

## 10. Linked Artifacts

| Artifact Type | ID | File |
|---------------|----|------|
| Validation Test Case | TC-[ID] | [validation/test-cases/TC-[ID].md] |
| Detection (Sigma) | DET-[ID]-SIGMA | [detections/sigma/[ID]-[slug].yml] |
| Detection (KQL) | DET-[ID]-KQL | [detections/queries/[ID]-kql.md] |
| Hunt Hypothesis | HYP-[ID] | [hunting/hypotheses/HYP-[ID].md] |
| Response Playbook | PB-[ID] | [playbooks/PB-[ID]-[slug].md] |

---

## 11. Related Scenarios

| Scenario ID | Name | Relationship |
|-------------|------|-------------|
| [ID] | [Name] | [e.g., "Precursor — access often escalated before this scenario"] |

---

## 12. References

- [External reference 1]
- [External reference 2]

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | [YYYY-MM-DD] | [Author] | Initial creation |
