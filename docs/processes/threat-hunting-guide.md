# Threat Hunting Guide

> **Version:** 1.0 | **Last Updated:** 2026-03-26

This guide describes how to plan and execute scenario-driven threat hunts using SIDTVF hypotheses. It covers hypothesis selection, data source preparation, query execution, finding classification, and feeding results back into the framework.

---

## Hunting Philosophy

SIDTVF hunting is **hypothesis-driven and scenario-anchored**. Every hunt starts with a specific hypothesis linked to a defined scenario, not with an open-ended "let's see what's interesting" query.

This approach ensures:
- Results are actionable and tied to defined threat behaviors
- Hunting effort is prioritized by scenario severity and detection coverage gaps
- Findings feed back into the scenario library rather than disappearing into analyst notebooks

Hunting assumes that some threats have already bypassed existing detections. It is not a replacement for detection rules — it is a second line of discovery.

---

## Hunt Selection and Prioritization

Run hunts in this priority order:

| Priority | Criteria | Why |
|----------|----------|-----|
| 1 | Scenarios with no active detection (KPI-01 gap) | Highest blind-spot risk |
| 2 | Scenarios with a recent FAIL in validation | Detection may have been bypassed |
| 3 | High or Critical severity scenarios | Highest impact if activity is present |
| 4 | Scenarios not hunted in the past 90 days | Ensure regular coverage |
| 5 | Scenarios matching current threat intelligence | Relevance to active threat landscape |

---

## Step-by-Step Process

### Step 1 — Select and Review the Hypothesis

Open the target hypothesis file (`hunting/hypotheses/HYP-[ID].md`). Review:
- The hypothesis statement(s) — confirm they are still relevant
- The last hunt date — note the gap since last execution
- The linked scenario — re-read the attack phases and IOBs before running queries
- The data source requirements — confirm all required sources are available

If the hypothesis has never been executed, this is a baseline hunt. Document the baseline findings carefully — they establish the "normal" picture for future comparisons.

### Step 2 — Scope the Hunt

Define before opening any query tool:
- **Time window:** Default is 90 days back (or since the last hunt date, whichever is more recent)
- **User population:** All users? Users in a specific department? Users in the departing employee watchlist?
- **Systems:** All endpoints? Production servers only? Cloud environments only?
- **Data sources:** Which tables or log sources will you query?

Documenting scope in advance prevents scope creep and ensures consistent execution across quarterly cycles.

### Step 3 — Prepare Queries

Copy the queries from the hypothesis file into your query tool (SIEM, EDR console, etc.). Before running:
- Replace any placeholder values (domain names, user population filters, time ranges) with your environment-specific values
- Adjust thresholds based on your organization's baseline if you have prior hunt results to reference
- Confirm the required tables exist and contain recent data (run a simple `count()` query first)

### Step 4 — Execute and Analyze

Run each query in the hypothesis. For each result set:

**Ask three questions:**
1. Is this pattern consistent with the "expected/normal" description in the hypothesis?
2. Does any result match one or more IOBs from the linked scenario?
3. Does correlation with other data (HR status, concurrent access events, geographic context) change the significance of the result?

**Do not investigate every anomaly in real time.** Log all anomalies in a working notes document during the hunt. After running all queries, triage the anomalies collectively before escalating.

### Step 5 — Classify Findings

For each identified anomaly, assign a classification:

| Classification | Definition | Next Action |
|---------------|------------|-------------|
| **True Threat Activity** | Behavior matches scenario with high confidence; evidence of actual insider threat | Escalate to IR immediately |
| **Suspicious** | Behavior matches scenario but context is ambiguous; needs investigation | Escalate to SOC for triage within 24 hours |
| **Policy Violation** | Behavior is real but appears non-malicious (NEG archetype) | Route to HR/manager for awareness |
| **False Positive** | Anomaly has a legitimate, documented explanation | Document explanation; consider adding to query exclusion list |
| **Clean** | No anomalies found; hypothesis not supported by current data | Document and close |

### Step 6 — Document the Hunt Cycle

Update the "Hunt Findings Log" section of the hypothesis file with:
- Date executed, hunter name, time period covered
- Finding classification (use the classifications above)
- Summary of what was found (or confirmed not found)
- Actions taken

Even a "Clean" result must be documented. A consistent record of clean hunts is evidence that the monitoring environment is healthy.

### Step 7 — Feed Back Into the Framework

After every hunt, regardless of outcome:
- **If True Threat or Suspicious:** Escalate per the linked playbook and document the incident
- **If new behavior observed not covered by any scenario:** Draft a new scenario (see scenario authoring guide)
- **If hunt reveals a detection gap:** Open a gap ticket for Detection Engineering; this feeds KPI-08
- **If query thresholds needed significant adjustment:** Update the hypothesis file with the revised thresholds
- **Update KPI-07** (Hunt Hypothesis Conversion Rate) in the quarterly report

---

## Evidence Handling During Hunts

If a hunt identifies what appears to be **active** threat activity:

1. **Stop the hunt** — do not continue querying in ways that might alert the subject
2. **Preserve your query results** — export and save immediately
3. **Contact Legal** before any further action
4. **Do not confront or notify the subject** without Legal guidance
5. **Escalate to IR Lead** via out-of-band channel (phone, in-person)

Hunt data (queries and results) for live investigations must be treated as potential evidence. Follow the same chain of custody procedures as any incident investigation.

---

## Changelog

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-03-26 | Initial guide |
