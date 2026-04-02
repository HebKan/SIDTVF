# Guide: Running Threat Hunts

Threat hunting assumes that some insider threat activity has already bypassed your detections. This guide explains how to execute a hunt from a SIDTVF hypothesis file.

---

## When to Hunt

Hunt when:
- A detection gap has been identified through validation testing
- A scenario is Active but has no deployed detection yet
- An incident revealed a behavior not captured in existing rules
- A new scenario card has been authored but detection coverage is incomplete
- Periodic scheduled hunts (recommended: at least quarterly per Active scenario)

Hunting is not a replacement for detections — it's a supplement for what detections miss.

---

## Hypothesis File Structure

Open the relevant hypothesis file: `hunting/hypotheses/HYP-[SCENARIO_ID].md`

| Section | What it contains |
|---------|-----------------|
| **Hypothesis statement** | The specific behavioral assumption being tested ("If an insider is exfiltrating via X, then Y evidence will be present in Z data source") |
| **Data sources required** | Log sources needed to execute the hunt |
| **Hunt queries** | SIEM queries to test the hypothesis |
| **Finding classification** | How to categorize what you find |
| **False positive guidance** | Common benign explanations |
| **Feedback path** | Where to route confirmed or suspicious findings |

---

## Executing a Hunt: Step-by-Step

### Step 1 — Review the Hypothesis
Read the hypothesis statement carefully. Confirm:
- You have access to the required data sources
- The data covers the time window you want to hunt (most hunts cover 30–90 days back)
- You understand what a positive finding looks like vs. a false positive

### Step 2 — Run the Hunt Queries
Execute each query in the hypothesis file against your SIEM. For each:
- Start with a broad query to understand baseline volume
- Narrow with the filters in the hypothesis to isolate anomalous activity
- Note any results that cannot be immediately explained by business context

### Step 3 — Investigate Hits
For each result that passes the false positive check:
1. Establish context — who is the user, what is their role, what systems should they access?
2. Review timeline — is there corroborating activity before or after this event (access changes, departures, volume spikes)?
3. Cross-reference — does the user appear in any HR watchlists, departure notifications, or prior alerts?

### Step 4 — Classify Each Finding

| Classification | Criteria | Next action |
|---------------|----------|-------------|
| **True Threat Activity** | Clear policy or legal violation; intent is evident | Escalate to IR immediately; open playbook |
| **Suspicious — Unconfirmed** | Anomalous but not conclusively malicious | Document; monitor; re-hunt in 2 weeks |
| **Policy Violation** | Non-malicious but non-compliant behavior | Route to HR or compliance per program policy |
| **False Positive** | Behavior has a clear benign explanation | Document the explanation for rule tuning |
| **Clean** | No anomalous activity found | Document as negative finding; update hypothesis status |

### Step 5 — Document and Feed Back

Record all findings — including clean ones. A documented negative hunt finding is evidence that your data is queryable and your hypothesis is testable.

**Route findings:**
- True threats and suspicious findings → IR team; open or update the linked playbook
- Detection gaps revealed → Detection engineering; file a new rule improvement task
- New behaviors not in any scenario → Scenario authoring team; draft a new scenario card
- False positive patterns → Detection engineering; tune existing rules

---

## Interpreting a Clean Result

A clean hunt result does not mean the scenario is not occurring. It means:
- No evidence found in the data sources queried, for the time window covered

It does **not** mean:
- The scenario has never occurred
- The scenario is not currently occurring via an unmonitored channel
- Your detection coverage is complete

Review the scenario's **Detection Blind Spots** section. If the scenario can occur via a method that produces no log event (Bluetooth, USB, verbal), a clean query result is expected and cannot rule out the behavior.

---

## Hunt Cadence

| Scenario Severity | Recommended hunt frequency |
|-------------------|---------------------------|
| Critical | Monthly |
| High | Quarterly |
| Medium | Semi-annually or event-driven |
| Low | Annually or event-driven |

---

## Related

- All hunt hypotheses: [hunting/hypotheses/](../../hunting/hypotheses/)
- Hypothesis template: [hunting/_template.md](../../hunting/_template.md)
- Hunt process methodology: [docs/processes/threat-hunting-guide.md](../processes/threat-hunting-guide.md)
- After a positive finding — executing a playbook: [docs/guides/using-playbooks.md](using-playbooks.md)
