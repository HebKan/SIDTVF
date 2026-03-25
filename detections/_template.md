# Detection Rule Template

> Copy this file and rename it `[SCENARIO-ID]-[short-slug]-[platform].md` for query-based detections,
> or use the Sigma template below for Sigma rules saved as `.yml`.
> Remove all instructional comments (lines beginning with `>`) before publishing.

---

## Detection Card

| Field | Value |
|-------|-------|
| **Detection ID** | DET-[SCENARIO-ID]-[PLATFORM] |
| **Name** | [Short descriptive name] |
| **Version** | 1.0 |
| **Status** | Draft / Active / Deprecated |
| **Created** | [YYYY-MM-DD] |
| **Last Updated** | [YYYY-MM-DD] |
| **Author** | [Name or team] |
| **Linked Scenario** | [SCENARIO-ID] — [Scenario Name] |
| **Platform** | SIGMA / KQL / SPL / YARA-L / KQL / SQL / [Other] |
| **Data Source(s)** | [e.g., Microsoft Defender for Endpoint, Windows Security Event Log, Proxy Logs] |
| **Detection Type** | Rule-based / Behavioral / Anomaly / Threshold |
| **MITRE ATT&CK** | [Tactic — Technique ID: Technique Name] |
| **Severity** | Critical / High / Medium / Low |
| **Confidence** | High / Medium / Low |

> **Confidence Guide:**
> - **High** — This pattern strongly indicates the threat. False positive rate is low.
> - **Medium** — This pattern is suggestive but requires correlation with other signals.
> - **Low** — Weak indicator. Should only alert when combined with other signals or used for hunting.

---

## 1. Detection Logic

> Paste the query or rule logic here. For Sigma rules, see the Sigma section below.
> For platform-specific queries, include the full query with inline comments.

```
[Paste detection logic here]
```

---

## 2. Data Source Requirements

> List every data source, log type, or configuration required for this detection to function.

| Requirement | Details | Mandatory? |
|------------|---------|-----------|
| [Data source name] | [What must be enabled/configured] | Yes / No |

---

## 3. Tuning Guidance

> Explain what parameters should be adjusted for different environments and how to tune to reduce false positives.

- **Threshold adjustments:** [e.g., "Adjust the 500 MB upload threshold based on your organization's normal data transfer volume"]
- **Exclusions:** [e.g., "Exclude known backup service accounts and scheduled task accounts"]
- **Baselining period:** [e.g., "Requires 30 days of baseline data before reliable alerting"]

---

## 4. False Positive Considerations

> Document the most common false positive scenarios for this detection and how analysts should differentiate.

| False Positive Scenario | How to Differentiate |
|------------------------|---------------------|
| [Describe legitimate activity that might trigger this rule] | [How to confirm it is benign] |

---

## 5. Response Actions

> What should an analyst do when this detection fires? Reference the linked playbook.

1. [First triage step]
2. [Second triage step]
3. Escalate to [team] if [condition]

**Linked Playbook:** [PB-XX-NNN or "Not yet created"]

---

## 6. Validation

> Describe how to test that this detection fires correctly.

**Test Case:** [TC-SCENARIO-ID](../../validation/test-cases/TC-[SCENARIO-ID].md)

**Expected Result:** [Describe the alert that should be generated when the test case is executed]

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | [YYYY-MM-DD] | [Author] | Initial creation |

---

## Sigma Rule Template

> For Sigma rules, save as a `.yml` file in `detections/sigma/` instead of using this markdown format.
> The YAML block below is a starting template.

```yaml
title: [Short descriptive title]
id: [Generate a UUID — e.g., use uuidgen or an online UUID generator]
status: experimental  # experimental | test | stable | deprecated
description: |
  [One to three sentence description of what this rule detects and why it matters.
  Include the scenario ID this rule supports.]
references:
  - [URL to MITRE ATT&CK technique]
  - [URL to other relevant reference]
author: [Author name or team]
date: [YYYY-MM-DD]
modified: [YYYY-MM-DD]
tags:
  - attack.[tactic_name]       # e.g., attack.exfiltration
  - attack.t[TECHNIQUE_ID]     # e.g., attack.t1567.002
  - sidtvf.[SCENARIO-ID]       # e.g., sidtvf.de-001
logsource:
  category: [e.g., proxy, process_creation, file_event]
  product: [e.g., windows, linux, azure]
  service: [e.g., security, sysmon]  # if applicable
detection:
  selection:
    [field_name]: [value]
    # Add additional selection criteria
  filter_legitimate:
    [field_name]: [exclusion_value]
    # Add exclusions for known-good patterns
  condition: selection and not filter_legitimate
fields:
  - [Field1]
  - [Field2]
falsepositives:
  - [Describe common false positive scenario]
level: [critical | high | medium | low | informational]
```
