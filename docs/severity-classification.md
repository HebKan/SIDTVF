# SIDTVF Severity Classification Reference

> **Document Version:** 2.0 | **Status:** Active | **Last Updated:** 2026-04-01

---

## Overview

Every SIDTVF scenario carries a single severity rating: **Critical**, **High**, **Medium**, or **Low**. Severity is derived from two assessments — **Impact** and **Likelihood** — combined in a matrix.

**Most scenarios can be rated in under two minutes using Part 1.** The detailed sub-dimension reference in Parts 6 and 7 exists for borderline cases where a quick read is ambiguous. You do not need to score every sub-dimension for every scenario.

> **Control validation and simulation tests** do not rate impact and likelihood from scratch — they inherit severity from the parent scenario. See Part 3.

Use this document when:
- Authoring a new scenario card and selecting the `Severity`, `Impact Score`, and `Likelihood Score` fields
- Reviewing or challenging an existing severity assignment
- Explaining ratings to non-technical stakeholders
- Calibrating SOC triage card escalation thresholds

---

## Part 1 — Quick Rating (Use This First)

Answer two questions. Look up the result in the matrix in Part 2.

---

### Question 1: Impact

**If this scenario plays out completely, what is the worst realistic outcome?**

Pick the single tier that best matches. You do not need every bullet to apply — one is enough.

| Tier | It applies if any of the following are true |
|------|---------------------------------------------|
| **Catastrophic** | Regulated data exposed at scale (>1,000 PII, PHI, or MNPI records; OR any trade secret or core IP regardless of volume) · Critical system destroyed or permanently unrecoverable · Estimated harm exceeds $1M · External regulatory notification is mandatory (HIPAA breach, GDPR Art. 33/34, SEC 8-K) |
| **Severe** | Sensitive data exposed at limited scale (<1,000 regulated records, or significant confidential business data) · Major business process disrupted for hours · Estimated harm $100K–$1M · Internal legal investigation required |
| **Moderate** | Internal non-regulated data exposed · Single system disrupted with recovery under one hour · Estimated harm under $100K · Policy violation with real organizational consequence but no external reporting obligation |
| **Minor** | No sensitive or regulated data involved · No service disruption · Negligible financial exposure · Fully reversible · Policy violation only |

> **Tip:** If you are uncertain between two tiers, choose the higher one and note the rationale. Severity can be revised downward with evidence; under-rating a Critical scenario creates unacceptable risk.

---

### Question 2: Likelihood

**How probable is this scenario in a typical enterprise environment?**

Consider access requirements, skill level, how easily the method evades monitoring, and how often this type of behavior is observed. Pick the single tier that best fits.

| Tier | Typical indicators |
|------|--------------------|
| **High** | Standard user access is sufficient · No technical skill beyond everyday tools · Behavior blends with normal activity, OR the attack method is inherently invisible to monitoring (physical or out-of-band: Bluetooth, USB, hardwire connection, screen photo, verbal disclosure) · Documented frequently across enterprises |
| **Medium** | Requires departmental or elevated access · Basic technical knowledge needed · Method is partially monitored (requires CASB, TLS inspection, or EDR to see) · Observed regularly but not ubiquitously |
| **Low** | Requires privileged or highly restricted access · Significant skill or insider knowledge required · Method transits well-monitored infrastructure by default · Rarely observed in industry reporting |

> **Attack method matters for likelihood.** An exfiltration via Bluetooth or a hardwire connection to a personal device produces no network or endpoint log regardless of actor behavior — that makes detection structurally harder and likelihood effectively higher. See the Attack Vector Reference Table in Part 7 for common methods and their detectability ratings.

---

## Part 2 — Severity Matrix

|  | **High Likelihood** | **Medium Likelihood** | **Low Likelihood** |
|--|---------------------|-----------------------|--------------------|
| **Catastrophic Impact** | **Critical** | **Critical** | **High** |
| **Severe Impact** | **High** | **High** | **Medium** |
| **Moderate Impact** | **High** | **Medium** | **Low** |
| **Minor Impact** | **Medium** | **Low** | **Low** |

**Rules:**
- Catastrophic impact is never below **High** regardless of likelihood — the potential harm alone justifies elevated response readiness.
- Moderate impact with High likelihood is **High**, not Medium — common, easy-to-execute behaviors warrant active detection even when individual harm is limited.
- Minor impact peaks at **Medium** — frequent occurrence does not override the ceiling on actual harm.

---

## Part 3 — Severity for Validation Tests and Simulation Tests

**Control validation tests** (TC-[ID]) and **simulation tests** (SIM-[ID]) do not describe a real incident — they describe a test of whether a detection fires. For these artifacts, severity answers a different question:

> *If this test reveals the detection does not work, how serious is that gap?*

The answer is the same as the severity of the parent scenario being tested.

| Artifact type | How to assign severity |
|---------------|----------------------|
| Scenario card | Rate using Parts 1 and 2 above |
| Detection rule | Inherits severity from the parent scenario |
| Validation test case (TC-[ID]) | Inherits severity from the parent scenario |
| Simulation test (SIM-[ID]) | Inherits severity from the parent scenario |
| Hunt hypothesis (HYP-[ID]) | Inherits severity from the parent scenario |
| Playbook (PB-[ID]) | Inherits severity from the parent scenario |

**Example:** A validation test for S3 exfiltration detection (TC-DE-001) is rated Critical — not because the test itself causes harm, but because a failed test means the organization cannot detect a Critical real-world scenario. The test's severity reflects the cost of the coverage gap, not the cost of the test.

You do not need to rate impact dimensions like data sensitivity or financial loss for a test case. The parent scenario has already done that work. Simply copy the parent scenario's severity to all linked artifacts.

---

## Part 4 — Severity Tiers Defined

### Critical

**Definition:** Catastrophic harm is possible, and the scenario is either frequently executed by standard-access employees or structurally difficult to monitor. Requires the highest detection priority, pre-approved response playbooks, and direct executive notification on confirmation.

**Characteristics:** Mass regulated data exposure · Mandatory external reporting triggered · >$1M financial harm · Operational destruction threatening business continuity · Often executable without elevated access or technical skill

**Examples:**

| Scenario | Why Critical |
|----------|-------------|
| **DE-001** — Cloud Storage Upload Exfiltration | Any employee can execute using a browser; method is partially monitored without CASB; mass PII/IP exposure at scale; breach notification triggered |
| **SB-001** — Targeted Database Deletion | Production data permanently destroyed; multi-day recovery; detection often arrives after damage is complete |
| **FR-001** — Invoice and Vendor Payment Manipulation | $1M+ wire fraud achievable via standard ERP and email tools; finance team access is all that is required |
| **IP-001** — Source Code Theft | Core IP and trade secrets; DTSA exposure; frequently motivated by departure; difficult to recover from once disclosed |
| **AC-001** — Account Compromise by External Actor | External actor inherits full insider access scope; activity is indistinguishable from legitimate use |

---

### High

**Definition:** Severe harm is realistic, or catastrophic harm requires access or skill that limits the actor pool. Requires active detection coverage and management-level escalation on confirmation.

**Characteristics:** Significant sensitive data exposed · $100K–$1M harm · Legal investigation or audit risk · Requires departmental access or basic technical knowledge

**Examples:**

| Scenario | Why High |
|----------|---------|
| **DE-002** — Email Auto-Forwarding Rule | Any M365 user; rule persists silently with no visible indicator; ongoing exfiltration of all email; frequently missed until departure |
| **UA-001** — After-Hours Unauthorized System Access | Standard credentials; well-documented pattern; common precursor to higher-severity scenarios |
| **PM-001** — Privilege Escalation via Access Abuse | Significant compliance exposure if admin-level systems are reached; requires IT role but not rare |
| **PV-001** — Shadow IT Data Storage | Any employee; extremely common; creates real data custody and governance risk even without malicious intent |

---

### Medium

**Definition:** Meaningful but contained harm, or limited-impact scenarios that are highly probable. Requires documented detection and response guidance, but not the same urgency as Critical or High.

**Characteristics:** Internal non-regulated data only · Under $100K harm · No external reporting obligation · Often negligent actor · Policy violation with real consequence

**Examples:**

| Scenario | Why Medium |
|----------|-----------|
| **DE-003** — AI Tool Data Leakage (negligent, no regulated data confirmed) | Extremely common; any employee; structurally invisible without CASB; harm is moderate when data type is internal/confidential rather than regulated |
| After-hours access to a non-sensitive shared drive with no download | Pattern worth monitoring; individual event is low-harm; policy violation only |

---

### Low

**Definition:** Minor harm with low probability, or a policy violation with no material organizational consequence. Still worth documenting as a potential early indicator, but does not justify SOC escalation on its own.

**Characteristics:** Non-sensitive data only · No financial exposure · No service impact · Fully reversible · HR action only

**Examples:**
- Employee installs unapproved personal software with no data transfer or network activity
- Employee accesses a coworker's non-sensitive shared folder without permission and reads one document
- Single USB transfer of an employee's own non-sensitive personal files at role end

> A single Low-severity event can be an early indicator of a higher-severity scenario when combined with other signals. Pattern analysis — not individual event severity — determines escalation in these cases.

---

## Part 5 — Scoring Worksheet

Use this for any scenario where the quick rating in Part 1 is ambiguous or where you need to document your rationale formally.

```
SCENARIO ID: __________   SCENARIO NAME: __________________________________
ARTIFACT TYPE: [ ] Scenario  [ ] Validation Test  [ ] Simulation Test  [ ] Other

─── FOR SCENARIO CARDS ONLY ─────────────────────────────────────────────────

IMPACT (pick the tier that matches; note which indicator applies)
  [ ] Catastrophic   Indicator: _______________________________________________
  [ ] Severe         Indicator: _______________________________________________
  [ ] Moderate       Indicator: _______________________________________________
  [ ] Minor          Indicator: _______________________________________________

IMPACT SCORE: ______________

LIKELIHOOD (pick the tier that best fits overall)
  [ ] High    [ ] Medium    [ ] Low

  Key factors driving this rating:
    Access required: __________________________________________________________
    Technical skill: __________________________________________________________
    Attack method / vector detectability: _____________________________________
    Observed frequency: _______________________________________________________

LIKELIHOOD SCORE: ______________

SEVERITY (matrix lookup): ______________

─── FOR VALIDATION TESTS AND SIMULATION TESTS ───────────────────────────────

  Parent scenario ID: __________   Parent scenario severity: __________
  Artifact severity (inherited): __________

─────────────────────────────────────────────────────────────────────────────
Assigned by: ________________   Date: ___________   Reviewed by: ____________
```

---

## Part 6 — Reference: Impact Factors

Use this section when the quick rating in Part 1 is ambiguous — for example, when a scenario touches multiple categories at different tiers and you need to anchor your decision.

**The Impact Score equals the highest applicable tier across these five factors.** You do not need to rate all five — skip factors that do not apply to the scenario.

| Factor | Catastrophic | Severe | Moderate | Minor |
|--------|-------------|--------|----------|-------|
| **Data** | >1,000 regulated records (PII/PHI/MNPI); any trade secret or core IP | <1,000 regulated records; significant confidential business data | Internal data only, no regulated types | No sensitive data |
| **Financial** | >$1M direct loss or regulatory fine; SEC enforcement trigger | $100K–$1M; material remediation or notification costs | $10K–$100K; limited remediation | <$10K; cleanup only |
| **Operational** | Critical system destroyed; multi-day recovery; business continuity threatened | Major process disrupted; hours of recovery | Single non-critical system; <1 hour recovery | No service impact |
| **Legal / Regulatory** | Mandatory external notification (HIPAA, GDPR Art. 33/34, SEC 8-K); criminal referral risk | Internal legal investigation; compliance audit triggered | Legal review required; no external obligation | HR action only; no legal involvement |
| **Reputational** | Public breach disclosure; customer notification letters; press exposure | Executive/board visibility; partner relationship risk | Team/department level only; no customer impact | Fully internal; no reputational risk |

---

## Part 7 — Reference: Likelihood Factors and Attack Vector Table

Use this section when the quick rating in Part 1 is ambiguous or when you need to document how a specific attack method affects detectability.

### Likelihood Factors

Two actor-behavior factors and one method factor combine to produce the likelihood tier.

| Factor | High contribution | Medium contribution | Low contribution |
|--------|------------------|---------------------|-----------------|
| **Actor access required** | Standard user access | Departmental or functional role | Elevated/privileged access or multi-party approval |
| **Technical skill required** | Everyday tools only (browser, email, USB, Bluetooth) | Basic scripting or CLI | Security control knowledge, custom tooling, or exploitation |
| **Actor evasion effort** | None — behavior blends or is unmonitored | Minimal adjustments (after-hours, personal account) | Deliberate staging, encrypted channels, or active log tampering |
| **Observed frequency** | Multiple enterprise incidents per year; well-documented | Regularly observed across industry | Occasional or rare; specialist actor profile |
| **Attack vector detectability** (method property, not actor behavior) | Method produces no log event by design — see table below | Method requires additional tools (CASB, EDR, TLS inspection) to see | Method transits fully monitored, inspectable infrastructure |

> **Key distinction:** Actor evasion effort is what the actor *chooses to do*. Attack vector detectability is a property of the *method itself* — a Bluetooth transfer leaves no network log whether the actor intended evasion or not.

### Attack Vector Reference Table

| Method | Access | Skill | Detectability | Notes |
|--------|--------|-------|---------------|-------|
| **Bluetooth file transfer** | Standard | None | **None — no log event** | Completely off-network; no corporate log without specific MDM/EDR Bluetooth policy |
| **USB / removable media** | Standard | None | **None without EDR policy** | Bypasses all network monitoring; EDR removable media policy captures it if deployed |
| **Hardwire / Thunderbolt to external drive** | Standard | Low | **None — no log event** | High throughput (GB/min); no network footprint; requires physical device access |
| **NFC transfer** | Standard | None | **None — no log event** | Very short range; rare; zero network footprint |
| **Screen photograph (personal camera/phone)** | Standard | None | **None — no log event** | No corporate digital footprint; captures anything rendered on screen |
| **Verbal or cognitive disclosure** (memorizing, dictating) | Standard | Low | **None — no log event** | No technical artifact; detectable only through behavioral observation |
| **Printing sensitive documents** | Standard | None | **None without print audit logging** | Print server audit logging often disabled or not SIEM-forwarded |
| **Personal webmail** (Gmail, ProtonMail) via corporate network | Standard | None | **Content invisible without TLS inspection** | Domain and volume visible to proxy; file content not inspectable without CASB |
| **Consumer AI tool** (ChatGPT, Gemini, Copilot) | Standard | None | **Content invisible without CASB** | HTTPS visible to proxy; session content not inspectable without CASB; invisible to endpoint DLP |
| **Unsanctioned cloud storage** (Dropbox, personal Drive) | Standard | None | **Content invisible without CASB** | Domain visible; file names and content not inspectable without TLS inspection or CASB |
| **Personal VPN / encrypted tunnel** | Standard | Low | **Metadata visible; content encrypted** | Traffic volume and destination IP visible to firewall; content not inspectable |
| **DNS exfiltration** | Elevated | High | **Metadata visible; payload encoded** | DNS volume anomaly detectable; content extraction requires specialist tooling |
| **HTTPS covert channel / steganography** | Elevated | High | **Metadata visible; payload hidden** | Legitimate-looking traffic; content decoding requires specialized detection |
| **Corporate email via gateway** | Standard | None | **Fully inspectable** | Email gateway, DLP, and archiving capture content by default |
| **Corporate file share / SharePoint** | Standard | None | **Fully inspectable** | Native audit logging captures copy, move, and download events |
| **SaaS API export** (Salesforce, Workday, ERP) | Functional | Low | **Fully inspectable** | API activity logged natively; export volume anomaly detectable |
| **SFTP / FTP to external server** | Standard | Low | **Destination and volume visible** | Unusual protocol for corporate environments; firewall logs destination; content not inspectable |

> If a scenario uses multiple vectors, apply the detectability rating of the **least visible method** — the actor will complete the exfiltration via the path that cannot be seen.

---

## Part 8 — Common Calibration Mistakes

| Mistake | Correct approach |
|---------|-----------------|
| Applying all five impact factors to every scenario | Skip factors that don't apply. A pure access-control violation has no data or financial dimension — rate only the factors that describe real harm. |
| Rating a validation or simulation test's impact as if real data were at risk | Tests inherit severity from the parent scenario. The test itself has no data impact — the gap it reveals does. |
| Rating Critical based on data type alone | Data type alone is not sufficient. A single accidental email with one PHI record is not Critical — check volume and likelihood too. |
| Downgrading severity because controls exist | Controls affect likelihood, not impact. Do not reduce the impact tier because your DLP would probably stop the exfiltration. |
| Treating a physical-vector scenario as Low likelihood because it requires physical access | Physical access to a device is a routine condition for most employees, especially at departure. Bluetooth and USB transfers require no elevated privilege. |
| Confusing actor evasion effort with attack vector detectability | These are independent. A Bluetooth transfer by a careless actor still produces no log event — the method's detectability is fixed regardless of intent. |
| Upgrading severity for motivated actors | Motivation belongs in the Actor Profile field. A disgruntled employee leaking a non-sensitive memo is still Medium at most. |
| Rating all departing employee scenarios as Critical | Severity depends on what data the departing employee can reach, not the departure itself. |

---

## Changelog

| Version | Date | Description |
|---------|------|-------------|
| 2.0 | 2026-04-01 | Restructured for simplicity: quick 2-question rating in Part 1 is the primary path; detailed sub-dimensions moved to reference sections (Parts 6–7); added Part 3 covering severity inheritance for validation tests and simulation tests; simplified scoring worksheet; consolidated calibration guide |
| 1.1 | 2026-03-31 | Added Dimension 5 (Attack Vector Detectability); added Attack Vector Reference Table; updated scoring worksheet |
| 1.0 | 2026-03-30 | Initial release |
