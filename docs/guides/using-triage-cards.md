# Guide: Using SOC Triage Cards

Triage cards are single-page reference documents for SOC analysts. They answer one question: **given this alert, what do I do right now?** This guide explains how to work through one efficiently.

---

## When to Use a Triage Card

Pull the relevant triage card (`detections/triage-cards/TRIAGE-[SCENARIO_ID].md`) as soon as an alert fires for a scenario you have a card for. The card is designed to be worked through in 5–10 minutes for a first-pass assessment.

Do not wait for a full investigation before using the card. Triage cards are for the first 15 minutes, not the first 15 hours.

---

## Triage Card Structure

Every triage card follows this sequence:

### 1. Immediate Check
A quick sanity check at the top of the card. Confirms the alert is real before you spend any time on it. Typically: verify the user and destination exist, confirm the timestamp is plausible, rule out known automation.

### 2. Decision Questions
Two to four yes/no questions that determine severity. Work through each one in order — they are sequenced so that a "yes" to an early question may let you skip later ones.

Example (DE-001):
- Is the destination domain on the HR watchlist or departure tracker?
- Is this a staging pattern (multiple small uploads before a larger one)?
- Has this domain been seen from this user before?

### 3. Severity Decision Table
Maps your answers to a severity tier (Critical / High / Medium / Low). Use this to set your ticket priority and determine the escalation path.

### 4. Step-by-Step Actions
Numbered response steps for each severity tier. Follow these in order. Steps are labeled with the responsible team (`[SOC]`, `[IR]`, `[Legal]`, `[HR]`) so you know immediately who to loop in.

### 5. What NOT to Do
A short list of actions that are explicitly prohibited at the triage stage — typically: do not contact the subject, do not revoke access without IR Lead approval, do not inform the subject's manager.

### 6. False Positive Table
Common benign explanations for the alert with distinguishing factors. If the activity matches a false positive pattern, document it and close or downgrade.

---

## Working Through a Triage Card: Step-by-Step

1. **Open the triage card** for the fired scenario. It's at `detections/triage-cards/TRIAGE-[SCENARIO_ID].md`.
2. **Run the Immediate Check.** If it fails (e.g., user doesn't exist, timestamp is clearly wrong), close as false positive and document.
3. **Answer each Decision Question** using SIEM/XDR queries and context you have available. Note your answers — you'll need them for the escalation handoff.
4. **Look up your severity** in the Severity Decision Table using your answers.
5. **Follow the Step-by-Step Actions** for your severity tier. Do not skip steps. Decision gates (e.g., "obtain IR Lead approval before proceeding") exist for legal and HR reasons.
6. **Check the False Positive Table** if anything feels off. Document your reasoning either way.
7. **Hand off** to IR with your answers, the severity determination, and any artifacts you collected (log snippets, screenshot of activity timeline).

---

## Escalation Thresholds

| Severity | Default escalation |
|----------|--------------------|
| Critical | Page IR Lead immediately; notify manager on duty; Legal/HR notification within 1 hour |
| High | Create IR ticket; notify IR Lead within 4 hours; preserve evidence |
| Medium | Create ticket; assign to standard investigation queue; no immediate escalation required |
| Low | Log and monitor; close if no additional signals within 48 hours |

---

## Common Mistakes

- **Contacting the subject** before IR Lead authorization. This destroys the investigation.
- **Revoking access immediately** — for Critical alerts, access revocation is an IR decision, not a SOC decision. Premature revocation tips off the actor and can destroy evidence.
- **Skipping the False Positive Table** — many high-volume alerts have well-understood benign explanations. Check before escalating.
- **Treating triage as investigation** — the triage card gets you to a severity determination and initial actions. The full investigation is the playbook's job.

---

## Related

- After triage — executing a playbook: [docs/guides/using-playbooks.md](using-playbooks.md)
- All triage cards: [detections/triage-cards/](../../detections/triage-cards/)
- SOAR automation for alert routing: [docs/soar-integration.md](../soar-integration.md)
