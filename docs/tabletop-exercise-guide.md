# Tabletop Exercise Guide

> **Version:** 1.0 | **Last Updated:** 2026-03-26
>
> A tabletop exercise is a structured discussion where key stakeholders walk through a simulated insider threat scenario to test whether your people, processes, and playbooks are ready. It is not a technical test — no systems are touched. It is a conversation about what you would do.

---

## Why Run a Tabletop?

Detection rules and playbooks look complete on paper. Tabletops reveal what the paper cannot:
- Does the SOC analyst know who to call when the alert fires at 2am?
- Does Legal know they need to be involved before containment, not after?
- Does HR know their role in the investigation decision gate?
- Is the playbook actually executable in the time pressures of a real incident?

Most organizations discover in their first tabletop that the answer to at least two of these questions is no. That is the point.

---

## Format Overview

| Element | Detail |
|---------|--------|
| **Duration** | 2 hours (recommended) |
| **Participants** | 6–10 people |
| **Format** | Discussion-based; no live systems |
| **Facilitator** | Security program owner or external consultant |
| **Frequency** | Quarterly or after any material program change |
| **Scenario** | Select one SIDTVF scenario per exercise |

---

## Required Participants

| Role | Why They Must Attend |
|------|---------------------|
| SOC Lead / Tier 2 Analyst | Owns detection triage; must know escalation path |
| IR Lead | Owns evidence preservation and investigation |
| Legal Counsel | Must be involved before containment actions that affect an employee |
| HR Representative | Owns the employment decision gate; critical for termination or leave decisions |
| CISO | Sees cross-team gaps; owns escalation decisions |
| System / Data Owner | Provides context on what systems hold sensitive data and what the business impact is |
| *Optional:* Finance or Compliance | Relevant for FR-001, IP-001, and MNPI scenarios |
| *Optional:* Communications / PR | Relevant for scenarios with external notification obligations |

---

## Pre-Exercise Preparation (1 week before)

**Facilitator tasks:**
- [ ] Select the scenario (choose a scenario with a published playbook — start with DE-001 or SB-001)
- [ ] Read the full scenario card and playbook
- [ ] Prepare 5–7 inject cards (see template below) — specific events that unfold during the exercise
- [ ] Confirm participant attendance; send the scenario name (not the injects) in advance
- [ ] Book a 2-hour meeting room with whiteboard or shared screen

**Participant preparation (optional pre-read):**
- The scenario category overview (e.g., "this exercise involves a data exfiltration scenario")
- Their role in the relevant playbook section
- Do NOT distribute the inject details — surprises reveal real gaps

---

## Exercise Agenda (2 hours)

### 0:00–0:10 — Opening and Ground Rules

Facilitator reads aloud:
> "This is a tabletop exercise. No real systems will be touched. There are no wrong answers — the goal is to find gaps in our process, not to test individuals. Everything said in this room is for improvement purposes only. We will capture action items but not blame."

Confirm roles: go around the table and have each participant state their role in an actual incident.

### 0:10–0:20 — Scenario Background

Facilitator reads the scenario narrative (3–5 minutes) from the scenario card, stopping before the detection event. Ask: "What is your current monitoring coverage for this type of behavior? Do you have detection rules for this? When was the last time they were tested?"

Let the discussion run naturally for 5–10 minutes.

### 0:20–1:20 — Inject Sequence (60 minutes, 5–7 injects)

Deliver injects one at a time, with 8–10 minutes of discussion per inject. See inject card template below.

After each inject, ask:
1. "What do you do now?"
2. "Who makes that decision?"
3. "What information do you need that you don't have?"
4. "What could go wrong with that approach?"

Do not correct answers during the exercise. Note gaps on the whiteboard.

### 1:20–1:40 — Hot Wash

Facilitated debrief:
1. "What went well? What parts of our process are solid?"
2. "Where were there handoff gaps — moments where nobody knew who owned the next step?"
3. "Were there moments where Legal or HR should have been involved earlier than they were?"
4. "What assumptions did we make that we should validate in a real incident?"

Capture all gaps on the whiteboard.

### 1:40–2:00 — Action Items

Convert gap discussions to named action items with owners and dates:

| Gap | Remediation | Owner | Target Date |
|-----|------------|-------|-------------|
| | | | |

Distribute action items within 48 hours of the exercise.

---

## Inject Card Template

Injects are discrete events delivered by the facilitator during the exercise. Each inject should force a decision or reveal a gap. Design them in sequence to follow the scenario's attack phases.

```
INJECT [N] — [Title]
Time in scenario: [e.g., "Day 1, 14:30"]
---
[Describe what just happened. Be specific. Include details that force decisions.]

Example: "The SOC Tier 1 analyst receives an alert: a user with the username jsmith
has uploaded 340 MB to drive.google.com over the past 6 hours. The user's manager
is currently on a business trip in Singapore. HR confirms this user submitted a
resignation letter yesterday morning — their last day is in two weeks."

Discussion questions:
- What do you do in the next 30 minutes?
- Who do you notify, and in what order?
- Do you revoke access now? What are the risks of waiting?
```

---

## Sample Inject Sequence: DE-001 (Cloud Storage Exfiltration)

**Inject 1 — Initial Alert**
> The SIEM fires an alert at 2:47 PM: user `jsmith` has uploaded 185 MB to `drive.google.com` in the past 4 hours. This exceeds the 50 MB/hour threshold. `jsmith` is a Senior Analyst in the Finance team with access to M&A transaction data.

*Discussion:* What is the first action? Who owns triage? What information do you need before escalating?

**Inject 2 — HR Context**
> You've escalated to Tier 2. SOC has confirmed the upload is real and appears to be to a personal Google Drive account (not the corporate workspace). You ask HR to check `jsmith`'s status. HR responds: `jsmith` gave verbal notice to their manager 3 days ago; formal documentation has not yet been processed.

*Discussion:* Does verbal notice change your response? Do you involve Legal at this point? What is the risk of contacting `jsmith` directly?

**Inject 3 — Legal Gate**
> Legal has been notified. They advise: "Do not revoke access yet — we need to assess what data was uploaded before we take action. Revoking access prematurely may alert the subject and we may lose the ability to complete a forensic review. We also need to determine if this user had access to material non-public information."

*Discussion:* How do you respond to Legal's guidance? Does anyone disagree? What is the timeline pressure? What evidence preservation must happen immediately regardless of Legal's guidance?

**Inject 4 — Evidence Gap**
> IR has requested 90 days of proxy logs. IT responds: proxy logs are only retained for 14 days. The uploads began approximately 18 days ago based on circumstantial file access logs.

*Discussion:* What does this gap mean for your investigation? What can you recover? What would you do differently to prevent this gap in the future?

**Inject 5 — Subject Confrontation Decision**
> Legal and HR have reviewed the evidence. Legal advises you have sufficient evidence for an employment action. HR asks: "Do we need IT to revoke access before we meet with this employee, or during the meeting, or after?" Legal notes that the meeting must happen before the resignation date (8 business days away) to preserve the right to a company-directed separation.

*Discussion:* Who is in the meeting? What order of operations for the access revocation? What happens if the employee denies everything?

**Inject 6 — Escalation Trigger**
> During the HR meeting, the employee admits to uploading files to Google Drive but states they were "just personal copies for reference." They are visibly distressed. HR asks whether to continue the conversation or pause for the employee to obtain legal representation.

*Discussion:* What does Legal recommend? Does the answer change if the data included MNPI? What documentation is required at this point?

**Inject 7 — Notification Assessment**
> Post-meeting, Legal confirms that 3 of the uploaded files contained customer names and account identifiers meeting the definition of personal data under CCPA and potentially GDPR (the company has EU customers). Legal asks: "Do we have a breach notification obligation?"

*Discussion:* Who makes the breach notification determination? What is the timeline? Does the SOC have the documentation needed for the notification?

---

## Inject Sequences for Other Scenarios

| Scenario | Key Inject Topics |
|----------|------------------|
| DE-002 (Email Forwarding) | Rule discovery timing, post-termination access, volume assessment, mail provider contact |
| SB-001 (Database Deletion) | Parallel recovery + evidence preservation, backup integrity, criminal referral decision |
| PM-001 (Privilege Abuse) | HIPAA breach assessment, shared credential discovery, scope of patient records accessed |
| AC-001 (Account Compromise) | Distinguishing insider from external actor, impossible travel response, session termination |
| FR-001 (Invoice Fraud) | Finance notification, wire recall possibility, auditor involvement, law enforcement referral |

---

## Action Item Tracker

Use this table in the post-exercise report:

| # | Gap Identified | Remediation Action | Owner | Target Date | Status |
|---|---------------|-------------------|-------|-------------|--------|
| 1 | | | | | Pending |
| 2 | | | | | Pending |
| 3 | | | | | Pending |

---

## Exercise Record

Keep a record of each completed tabletop:

| Date | Scenario Used | Participants | Top 3 Gaps | Action Items Completed By |
|------|--------------|-------------|-----------|--------------------------|
| | | | | |
