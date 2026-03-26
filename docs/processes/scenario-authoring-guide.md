# Scenario Authoring Guide

> **Version:** 1.0 | **Last Updated:** 2026-03-26

This guide walks a practitioner through the end-to-end process of creating a new SIDTVF scenario from inception to Active status.

---

## When to Author a New Scenario

Create a new scenario when:
- A threat hunt identifies a behavior not yet in the library
- An incident reveals an attack pattern not covered by any existing scenario
- Industry reporting describes a new insider threat technique relevant to your environment
- A periodic gap review identifies an uncovered ATT&CK technique in scope

Do **not** create a new scenario if the behavior is a variant of an existing one — update the existing scenario card instead (minor version bump).

---

## Step-by-Step Process

### Step 1 — Assign an ID and Reserve the Slot

1. Check the highest existing ID in the target category directory (e.g., `scenarios/data-exfiltration/`)
2. Increment by one and zero-pad to three digits (e.g., if `DE-003` exists, the next is `DE-004`)
3. Add a placeholder row to `mapping/mitre-attack.md` and `README.md` with `Status: Draft`
4. Announce the new ID in your team channel to prevent duplicate assignment

### Step 2 — Draft the Scenario Card

Copy `scenarios/_template.md` (or the appropriate sub-template from `scenarios/_templates/`) to the correct category directory.

Rename the file: `[SCENARIO-ID]-[short-slug].md`

Fill in fields in this order:
1. Header fields (ID, name, version, dates, actor archetype, severity)
2. Description (one to three plain-language sentences — write this last if unsure)
3. Narrative Case Study (3–6 paragraphs, fictional but realistic)
4. Actor Profile
5. Attack Phases (most important: define observable actions per phase)
6. IOBs — Technical (derive these directly from the observable actions in phases)
7. IOBs — Contextual
8. Detection Opportunities (map phases to data sources)
9. MITRE ATT&CK Mapping (use current ATT&CK Enterprise version)
10. Legal and Privacy Considerations (scenario-specific, not generic)
11. Response Guidance (brief — full procedure goes in the playbook)
12. Linked Artifacts (fill in stubs; link back once artifacts are created)

**Key authoring rules:**
- Behaviors describe what the insider *does*, not what tool they use
- Each attack phase must have at least two observable actions
- IOBs must be observable events in logs — not inferences or conclusions
- Never use real company names, real person names, or real breach data

### Step 3 — Peer Review

Share the draft with at least one other security team member for a technical review. Reviewer checks:
- Are behaviors realistic in an enterprise setting?
- Are observable actions actually visible in the named data sources?
- Are MITRE ATT&CK mappings accurate at the technique/sub-technique level?
- Is severity classification justified?

Address feedback and update the scenario card. Increment the version if significant changes were made.

### Step 4 — Legal Review

Determine whether legal review is required:

| Monitoring Type in Scenario | Legal Review Required? |
|----------------------------|----------------------|
| File access and network metadata | No (standard monitoring) |
| Email metadata (to/from/subject) | Recommended |
| Email or message content | Yes |
| Biometric or location data | Yes |
| After-hours or behavioral profiling | Recommended |
| HR-correlated monitoring | Yes |

If required, share the draft with Legal. Specifically ask Legal to review the "Legal and Privacy Considerations" section and confirm it is accurate for your jurisdiction(s).

### Step 5 — Activate and Create Linked Artifacts

Once peer and legal reviews are complete:
1. Set `Status: Active` in the scenario card
2. Update `mapping/mitre-attack.md` with the full technique mapping
3. Update `mapping/compliance.md` if applicable
4. Update `README.md` Included Examples table
5. Create stub files for linked artifacts using their templates:
   - `detections/_template.md` → detection file
   - `hunting/_template.md` → hypothesis file
   - `validation/_template.md` → test case file
   - `playbooks/_template.md` → playbook file

Artifacts do not need to be fully completed before the scenario is activated, but stubs should exist with the correct IDs.

### Step 6 — Assign Detection and Validation Owners

Notify Detection Engineering that a new active scenario needs a detection rule.
Assign a validation owner who will execute the test case once the detection is deployed.

---

## Scenario Severity Reference

| Severity | Criteria |
|----------|----------|
| Critical | High likelihood + high impact (trade secret theft, mass PII, sabotage) |
| High | High likelihood OR high impact |
| Medium | Moderate likelihood and impact |
| Low | Low likelihood or low impact; primarily policy violations |

---

## Common Authoring Mistakes

| Mistake | Correct Approach |
|---------|-----------------|
| Describing tool-specific behavior ("uses Rclone to upload") | Describe the behavior ("uploads files to personal cloud storage") |
| IOBs that are inferences ("employee is planning to leave") | IOBs must be observable log events ("file access volume 3× above baseline") |
| Attack phases with no observable actions | Every phase needs ≥2 observable actions that appear in real logs |
| Mapping to a parent technique when a sub-technique exists | Always use the most specific applicable sub-technique |
| Generic legal notes ("consult legal counsel") | Scenario-specific legal notes ("email content monitoring requires two-party consent in California") |

---

## Changelog

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-03-26 | Initial guide |
