# Detection Development Guide

> **Version:** 1.0 | **Last Updated:** 2026-03-26

This guide walks a detection engineer through building, testing, deploying, and maintaining SIDTVF detection rules. All SIDTVF detections are scenario-anchored — a rule must be linked to an active scenario before it is deployed.

---

## Prerequisite: Scenario Must Be Active

Never write a detection rule without a linked, Active scenario. The scenario defines:
- What behavior the rule is detecting
- What data source the rule must query
- What the observable actions look like (which become the rule's selection logic)
- What false positives to expect

If no Active scenario exists for the behavior you want to detect, author the scenario first.

---

## Detection Rule Types

| Type | When to Use | Examples |
|------|------------|---------|
| **Rule-based** | Well-defined event with low variance | Email forwarding rule created, DROP TABLE executed outside change window |
| **Threshold** | Normal behavior exists but excessive volume is suspicious | Files accessed > 3× personal baseline, upload volume > 50 MB |
| **Behavioral / UEBA** | Pattern only detectable across a time window | After-hours access pattern, impossible travel |
| **Correlation** | Threat signal requires ≥2 independent data sources | File access spike + cloud upload within same session |

---

## Step-by-Step Process

### Step 1 — Derive Logic from the Scenario

Read the linked scenario's "Attack Phases" and "Detection Opportunities" sections. For each detection opportunity:
- Identify the data source (proxy, EDR, auth logs, DAM, etc.)
- Identify the specific field(s) that distinguish threat behavior from normal activity
- Identify the threshold or condition that separates signal from noise

Write this out in plain English before writing any query syntax:

> "Alert when a user's `SentBytes` to a domain in the personal cloud domain list exceeds 50 MB in a single session, where the domain is not in the approved business cloud list."

### Step 2 — Choose the Platform

Decide which format(s) to produce:
- **Sigma** (platform-agnostic, stored in `detections/sigma/`) — required for all new rules
- **KQL** (Microsoft Sentinel / Defender XDR) — if the organization uses M365/Azure
- **SPL** (Splunk) — if the organization uses Splunk
- **YARA-L** (Chronicle/Google SecOps) — if applicable

Always produce Sigma first. Platform-specific queries are translations of the Sigma rule.

### Step 3 — Write the Detection Rule

Use `detections/_template.md` (for query-based rules) or the Sigma YAML structure.

Key quality checks while writing:
- **Field names:** Verify field names against your actual log schema before committing. Wrong field names cause silent failures.
- **Case sensitivity:** Most SIEM field comparisons are case-sensitive. Use `|contains` or case-insensitive operators where appropriate.
- **Exclusions:** Add known-good exclusions from the start (service accounts, backup processes, approved cloud tenants). Exclusions are cheaper to add now than to tune later.
- **Comments:** Every non-obvious clause must have an inline comment explaining what it tests and why.

### Step 4 — Assign the Detection ID and File Name

Follow the naming convention from `CLAUDE.md`:
- Sigma file: `detections/sigma/[SCENARIO-ID]-[short-slug].yml`
- Query file: `detections/queries/[SCENARIO-ID]-[platform].md`
- Detection ID in the file header: `DET-[SCENARIO-ID]-[PLATFORM]`

### Step 5 — Peer Review

Have another detection engineer review the rule before deployment. Reviewer checks:
- Does the rule logic match the scenario's observable actions?
- Are exclusions correct and not overly broad?
- Are comments sufficient for a new analyst to understand the rule?
- Does the Sigma convert cleanly to the target platform?

### Step 6 — Validate Before Deploying

**Do not deploy to production without running the linked test case first.**

Execute `validation/test-cases/TC-[SCENARIO-ID].md` in a test environment. The rule must achieve a PASS result before production deployment. A PARTIAL result may be acceptable with documented justification, but a FAIL must be resolved.

If no test case exists, create one before proceeding.

### Step 7 — Deploy and Set Initial Alert State

Deploy to the target SIEM/XDR. For the first 14 days after deployment:
- Set the rule to **alert-only** (not automated response/playbook trigger)
- Monitor the false positive rate daily
- Tune exclusions if the false positive rate exceeds 20:1 in the first week

After the stabilization period, set the final alert severity and enable any automated enrichment or playbook triggers.

### Step 8 — Link the Detection to the Scenario

Update the linked scenario card's "Linked Artifacts" section with the new detection ID and file path. Update `mapping/mitre-attack.md` if the rule covers a technique not yet listed.

### Step 9 — Maintenance Triggers

Re-review and re-validate a detection rule when:
- The underlying data source changes (log format update, new proxy, EDR upgrade)
- The scenario card is updated with new attack phases or IOBs
- The false positive rate degrades above the 10:1 KPI target
- ATT&CK version update changes technique mappings
- Annually, as part of the scenario currency review

---

## Quality Standards Reference

| Standard | Requirement |
|----------|------------|
| Sigma rules | Must include: `title`, `id` (UUID), `status`, `description`, `logsource`, `detection`, `falsepositives`, `level` |
| KQL/SPL queries | Must include inline comments on every non-trivial clause |
| Data source notation | Must specify the exact log source and any required configuration (e.g., "Requires M365 Unified Audit Log with mailbox audit enabled") |
| Exclusion documentation | Every exclusion must have a comment explaining what it excludes and why |
| Tuning guidance | Every detection file must include a tuning section with threshold adjustment guidance |

---

## Changelog

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-03-26 | Initial guide |
