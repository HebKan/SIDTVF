# Guide: Using Response Playbooks

Playbooks provide step-by-step incident response procedures for confirmed or suspected insider threat scenarios. This guide explains when to open a playbook, how to read it, and how to adapt it to your environment.

---

## When to Open a Playbook

Open the relevant playbook (`playbooks/examples/PB-[SCENARIO_ID]-*.md`) when:
- SOC triage has elevated a case to **High** or **Critical** severity
- A threat hunt has returned **confirmed** or **suspicious** findings
- An HR or management escalation arrives with information suggesting insider activity

Do **not** open a playbook for every alert. The triage card handles low-confidence, early-stage assessment. The playbook is for cases where you have enough evidence to move into a structured investigation.

---

## Playbook Structure

| Section | Contents |
|---------|---------|
| **Overview** | Scenario ID, severity, and one-paragraph description of the incident type |
| **Immediate Actions** | Time-sensitive steps to take within the first hour |
| **Evidence Collection** | What to collect, where to find it, and chain-of-custody requirements |
| **Investigation Steps** | Structured analysis sequence to confirm, expand, or close the incident |
| **Containment** | When and how to contain — options are listed, not mandated |
| **Decision Gates** | Explicit checkpoints requiring Legal/HR/Executive sign-off before proceeding |
| **Notification** | Who to notify, in what order, and what to tell them |
| **False Positive Considerations** | Structured exit criteria if the investigation does not confirm the scenario |
| **Closure** | Documentation requirements and lessons-learned process |

---

## Reading Step Labels

Each step is labeled with the responsible team in brackets:

| Label | Team |
|-------|------|
| `[SOC]` | Security Operations Center — alert handling and initial evidence collection |
| `[IR]` | Incident Response — investigation lead and containment decisions |
| `[Legal]` | Legal counsel — advises on evidence handling, notifications, and scope |
| `[HR]` | Human Resources — employee relations, disciplinary decisions, termination |
| `[Exec]` | Executive / CISO — final authorization for high-consequence actions |

Steps are sequenced so that actions requiring legal or HR involvement appear at the correct point — not necessarily at the end. Do not reorder them.

---

## Decision Gates

Decision gates are explicit stops in the playbook where you must obtain authorization before proceeding. They look like:

> **DECISION GATE — [Legal + HR]:** Before taking any containment action that affects the subject's system access, obtain written authorization from Legal Counsel and HR.

Do not skip decision gates to move faster. They exist because premature containment (especially access revocation or system seizure) can:
- Tip off the subject and enable destruction of evidence
- Create legal liability if done without proper authorization
- Violate HR policy or employment law requirements

If you are blocked waiting for authorization, document the wait time and continue evidence-preservation steps that do not require authorization.

---

## Adapting to Your Environment

Playbooks are templates, not scripts. Before your first use, adapt:

1. **Contact details** — Replace role labels (`[Legal]`, `[HR]`) with actual names, phone numbers, and escalation paths for your organization.
2. **Tool names** — Replace generic references ("your SIEM", "your DLP console") with specific product names and URLs.
3. **Approval chains** — Adjust decision gate authorization requirements to match your organizational authority structure.
4. **Retention and evidence handling** — Replace generic guidance with your organization's actual legal hold and evidence chain-of-custody procedures.
5. **Notification templates** — Add jurisdiction-specific breach notification language if required by your legal/compliance team.

Store adapted versions in a restricted location — playbooks contain investigation procedures that should not be visible to potential subjects.

---

## Playbook vs. Triage Card

| | Triage Card | Playbook |
|-|-------------|---------|
| **When** | Alert fires | Case escalated to IR |
| **Time frame** | First 15 minutes | Hours to days |
| **User** | SOC analyst | IR lead + cross-functional team |
| **Goal** | Determine severity, initial actions | Complete investigation and response |
| **Length** | One page | Multi-section document |

---

## Related

- Before the playbook — triage: [docs/guides/using-triage-cards.md](using-triage-cards.md)
- All playbook examples: [playbooks/examples/](../../playbooks/examples/)
- Playbook template: [playbooks/_template.md](../../playbooks/_template.md)
- SOAR integration for automated playbook steps: [docs/soar-integration.md](../soar-integration.md)
