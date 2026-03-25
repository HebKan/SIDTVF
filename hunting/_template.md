# Threat Hunting Hypothesis Template

> Copy this file and rename it `HYP-[SCENARIO-ID].md`.
> Place it in `hunting/hypotheses/`.
> Remove all instructional comments (lines beginning with `>`) before publishing.

---

## Hypothesis Card

| Field | Value |
|-------|-------|
| **Hypothesis ID** | HYP-[SCENARIO-ID] |
| **Name** | [Short descriptive name] |
| **Version** | 1.0 |
| **Status** | Draft / Active / Closed |
| **Created** | [YYYY-MM-DD] |
| **Last Hunted** | [YYYY-MM-DD or "Never"] |
| **Author** | [Name or team] |
| **Linked Scenario** | [SCENARIO-ID] — [Scenario Name] |
| **Hunt Cadence** | [Quarterly / Monthly / Ad Hoc / On-Alert] |
| **Data Sources Required** | [List of logs/tools required] |
| **MITRE ATT&CK** | [Tactic — Technique ID] |

---

## 1. Hypothesis Statement

> A hypothesis is a specific, testable statement that frames the hunt. It should follow this form:
> "IF [insider threat behavior is occurring], THEN [observable evidence] WOULD BE VISIBLE IN [data source]."
>
> Write one to three hypothesis statements for this scenario.

**Primary Hypothesis:**
IF [threat behavior], THEN [observable signal] WOULD BE VISIBLE IN [data source].

**Secondary Hypothesis (optional):**
IF [alternate behavior variant], THEN [alternate signal] WOULD BE VISIBLE IN [alternate data source].

---

## 2. Hunt Scope

| Field | Value |
|-------|-------|
| **Time Window** | [e.g., "Past 90 days" or "Since last hunt date"] |
| **User Population** | [e.g., "All users with access to sensitive data" or "Users in departing employee watchlist"] |
| **Systems in Scope** | [e.g., "All corporate endpoints" or "Production database servers"] |
| **Data Sources** | [Specific log sources, tables, or tools to query] |

---

## 3. Hunt Queries

> Provide the queries used to test this hypothesis. Include comments explaining what each section does.
> Queries may be KQL, SPL, SQL, or pseudocode depending on the analyst's environment.
> Label each query with what it is testing.

### Query 1 — [What this query tests]

```
[Query here]
```

### Query 2 — [What this query tests]

```
[Query here]
```

---

## 4. Indicators to Look For

> What specific patterns in the query results would indicate that the hypothesis is confirmed?

| Indicator | Significance | Confidence |
|-----------|-------------|-----------|
| [Result pattern] | [What it means] | High / Medium / Low |

---

## 5. Expected vs. Anomalous Results

> Describe what "normal" looks like in this dataset so analysts can distinguish signal from noise.

**Normal (expected) results:**
- [What typical clean results look like]

**Anomalous results (investigate these):**
- [What results should be escalated]

---

## 6. Hunt Findings Log

> Document the results of each hunt cycle here. One entry per hunt execution.

### Hunt Cycle: [YYYY-MM-DD]

| Field | Value |
|-------|-------|
| **Hunter** | [Name] |
| **Date Executed** | [YYYY-MM-DD] |
| **Time Period Covered** | [Start date — End date] |
| **Finding Classification** | Clean / Suspicious / True Threat Activity / Policy Violation / False Positive |
| **Summary** | [Brief description of what was found or confirmed not found] |
| **Actions Taken** | [e.g., "Escalated to IR", "Closed as clean", "New scenario added: [ID]"] |

---

## 7. Escalation Criteria

> When should findings from this hunt be escalated to incident response?

Escalate to IR immediately if:
- [Condition 1 — e.g., "Any user with > 500 MB uploaded to personal cloud in the hunt window"]
- [Condition 2]

Route to SOC for further triage if:
- [Condition — e.g., "Upload volume > 50 MB but < 500 MB with no HR flag"]

Close as clean if:
- [Condition — e.g., "No uploads to personal cloud domains detected in the hunt window"]

---

## 8. Linked Artifacts

| Artifact | ID | File |
|----------|----|------|
| Scenario | [SCENARIO-ID] | [scenarios/...] |
| Detection | DET-[SCENARIO-ID] | [detections/...] |
| Playbook | PB-[SCENARIO-ID] | [playbooks/...] |
| Validation | TC-[SCENARIO-ID] | [validation/...] |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | [YYYY-MM-DD] | [Author] | Initial creation |
