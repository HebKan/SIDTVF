# Guide: Using Scenario Cards

Scenario cards are the foundation of SIDTVF. Every detection rule, playbook, hunt hypothesis, and test case is built on top of a scenario. This guide explains how to read one and how to apply it.

---

## What a Scenario Card Contains

Each scenario card has six sections:

| Section | What it tells you |
|---------|-------------------|
| **Scenario Card header** | ID, severity, actor type, maturity requirement |
| **Description** | One-paragraph plain-language summary of the threat |
| **Narrative Case Study** | A realistic story showing the scenario in a real enterprise |
| **Actor Profile** | Who executes this and why (access level, motivation, sophistication) |
| **Attack Phases** | Step-by-step breakdown of how the scenario unfolds |
| **Indicators of Behavior (IOBs)** | Observable signals — what to look for in logs |
| **Detection Guidance** | Minimum viable detection, blind spots, and MITRE mapping |
| **Response Guidance** | Immediate priorities and links to the full playbook |

---

## Reading the Header Fields

Before reading the full card, check these three fields:

- **Severity** — Critical/High/Medium/Low. Determines your detection priority and escalation path. See [docs/severity-classification.md](../severity-classification.md) if the rating seems off.
- **Minimum Maturity Level** — If your program is below this level, you likely lack the log sources or tooling to detect this scenario reliably. Build up to it rather than deploying a detection you cannot validate.
- **Actor Archetype** — Tells you whether you're looking for intentional (MAL), accidental (NEG), externally-driven (COM), or third-party (TP) behavior. This changes the response path significantly.

---

## Choosing a Starting Scenario

If you're building a new program, don't start with the most complex scenario — start with the one you can actually detect and validate with your current tooling.

**Good starting criteria:**
1. Your organization already collects the required log source (check the IOBs section)
2. The scenario's Minimum Maturity Level matches your current program level
3. The actor archetype fits your highest-priority risk (most programs start with MAL or NEG)

**Recommended first scenarios by maturity level:**

| Maturity | Recommended start |
|----------|------------------|
| Level 1–2 | DE-001 (cloud storage upload), DE-002 (email forwarding) |
| Level 2–3 | UA-001 (after-hours access), PM-001 (privilege misuse) |
| Level 3+ | SB-001 (database deletion), FR-001 (invoice fraud), IP-001 (IP theft) |

---

## Using a Scenario Card Day-to-Day

**When scoping a detection build:**
Read the IOBs section. Each IOB names the log source required. If you don't have that log source, the detection cannot fire — fix the logging gap first.

**When triaging an alert:**
Read the Attack Phases section. Match observed activity to a phase. This tells you where in the kill chain the actor is and what they might do next.

**When running a tabletop:**
Use the Narrative Case Study as your inject storyline. It's written to be readable by non-technical participants.

**When briefing leadership:**
Use the Description field (plain language) and the Severity field. The narrative provides context for why the scenario matters.

---

## Common Mistakes

- **Deploying detections without reading IOBs** — If the log source isn't present, the detection is a false confidence measure, not a real control.
- **Treating severity as fixed** — Severity is a starting point, not gospel. If your environment has specific controls or exposure that changes the risk, note it and apply the scoring worksheet in [docs/severity-classification.md](../severity-classification.md).
- **Skipping the Actor Profile** — The response path for a MAL (malicious) actor is very different from NEG (negligent). Don't engage HR and legal the same way for both.

---

## Related

- Authoring a new scenario: [docs/processes/scenario-authoring-guide.md](../processes/scenario-authoring-guide.md)
- Scenario template: [scenarios/_template.md](../../scenarios/_template.md)
- All scenario cards: [scenarios/](../../scenarios/)
- Machine-readable index: [scenarios/index.yaml](../../scenarios/index.yaml)
