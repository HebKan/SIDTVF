# SIDTVF Workflow Diagrams

> **Version:** 1.0 | **Last Updated:** 2026-03-26

ASCII workflow diagrams for each major SIDTVF process. Use these for onboarding, process documentation, and stakeholder briefings.

---

## 1. Framework Lifecycle (End-to-End)

```
  ┌─────────────────────────────────────────────────────────────────┐
  │                   SIDTVF Framework Lifecycle                    │
  └─────────────────────────────────────────────────────────────────┘

  Threat Intel / Incident / Hunt Finding / Gap Review
              │
              ▼
  ┌─────────────────────┐
  │  Layer 1: SCENARIO  │◄──────────────────────────────────────┐
  │  Author scenario    │                                        │
  │  Peer review        │                                        │
  │  Legal review       │                                        │
  │  Activate           │                                        │
  └────────┬────────────┘                                        │
           │                                                     │
           ▼                                                     │
  ┌─────────────────────┐    Gap found                          │
  │  Layer 2: VALIDATE  │──────────────────────────────────┐    │
  │  Build test case    │                                  │    │
  │  Execute in test    │    PASS                          │    │
  │  Document results   │──────────────────────────────┐  │    │
  └─────────────────────┘                              │  │    │
                                                       │  │    │
           ▼                                           │  │    │
  ┌─────────────────────┐                              │  │    │
  │  Layer 3: HUNTING   │                              │  │    │
  │  Form hypothesis    │                              │  │    │
  │  Execute hunt       │ Finding                      │  │    │
  │  Classify results   │──────────────────────────┐  │  │    │
  └─────────────────────┘                          │  │  │    │
                                                   │  │  │    │
           ▼                                       ▼  ▼  ▼    │
  ┌─────────────────────┐          ┌──────────────────────────┐│
  │  Layer 4: INTEGRATE │          │  Detection Engineering   ││
  │  Route findings to  │─────────►│  SOC / Triage           ││
  │  downstream teams   │          │  Incident Response       ││
  └─────────────────────┘          │  Legal & HR              ││
                                   └──────────────────────────┘│
           │                                                    │
           ▼                                                    │
  ┌─────────────────────┐                                       │
  │  Layer 5: MAPPING   │                                       │
  │  ATT&CK alignment   │                                       │
  │  Compliance mapping │                                       │
  └─────────────────────┘                                       │
           │                                                    │
           ▼                                                    │
  ┌─────────────────────┐                                       │
  │  Layer 6: FEEDBACK  │                                       │
  │  Review outcomes    │                                       │
  │  Update scenarios   │───────────────────────────────────────┘
  │  Improve detections │   (loop)
  └─────────────────────┘
```

---

## 2. Scenario Lifecycle

```
  Author drafts scenario
          │
          ▼
       [DRAFT]
          │
          ▼
   Peer review ──── Changes needed ──► (back to author)
          │
      Approved
          │
          ▼
  Legal review required? ──── No ──────────────────┐
          │                                         │
         Yes                                        │
          │                                         │
          ▼                                         │
   Legal reviews ──── Changes needed ──► (update)  │
          │                                         │
      Cleared                                       │
          │                                         ▼
          └──────────────────────────────► [ACTIVE]
                                               │
                              New behavior ────┤
                              variant found    │ (minor version bump)
                                               │
                              Breaking change  │
                              to behavior      │ (major version bump)
                                               │
                              No longer        ▼
                              relevant    [DEPRECATED]
                                          (never deleted)
```

---

## 3. Validation Cycle

```
  Quarterly schedule OR trigger event
          │
          ▼
  ┌───────────────────────┐
  │  1. PLAN              │
  │  Define scope         │
  │  Obtain authorization │
  │  Prepare environment  │
  └──────────┬────────────┘
             │
             ▼
  ┌───────────────────────┐
  │  2. EXECUTE           │
  │  Run each test case   │◄──────┐
  │  Record results       │       │ Next test case
  │  Execute cleanup      │───────┘
  └──────────┬────────────┘
             │
             ▼
  ┌───────────────────────┐
  │  3. CLASSIFY          │
  │  PASS / PARTIAL / FAIL│
  └──────────┬────────────┘
             │
      ┌──────┴──────┐
      │             │
    PASS          FAIL / PARTIAL
      │             │
      ▼             ▼
  KPI update    ┌───────────────────────┐
  (KPI-01,02)   │  4. GAP ANALYSIS      │
                │  Root cause           │
                │  Gap category         │
                │  Owner + target date  │
                └──────────┬────────────┘
                           │
                           ▼
                ┌───────────────────────┐
                │  5. REMEDIATE         │
                │  Fix detection rule   │
                │  Fix data pipeline    │
                │  Build missing rule   │
                └──────────┬────────────┘
                           │
                           ▼
                ┌───────────────────────┐
                │  6. RE-VALIDATE       │
                │  Re-run failed cases  │
                │  Confirm PASS         │
                └──────────┬────────────┘
                           │
                           ▼
                  Gap closed → KPI-08 update
```

---

## 4. Threat Hunting Process

```
  Scenario selected for hunting
          │
          ▼
  ┌───────────────────────┐
  │  1. HYPOTHESIS REVIEW │
  │  Read scenario IOBs   │
  │  Review data sources  │
  └──────────┬────────────┘
             │
             ▼
  ┌───────────────────────┐
  │  2. SCOPE DEFINITION  │
  │  Time window          │
  │  User population      │
  │  Data sources         │
  └──────────┬────────────┘
             │
             ▼
  ┌───────────────────────┐
  │  3. QUERY EXECUTION   │
  │  Run each query       │
  │  Note anomalies       │
  └──────────┬────────────┘
             │
             ▼
  ┌───────────────────────┐
  │  4. TRIAGE ANOMALIES  │
  │  Correlate signals    │
  │  Apply context        │
  └──────────┬────────────┘
             │
    ┌────────┴─────────────────────────────┐
    │         │              │             │
    ▼         ▼              ▼             ▼
  True    Suspicious    Policy Viol.   Clean /
  Threat  (ambiguous)   (NEG actor)    False Pos.
    │         │              │             │
    ▼         ▼              ▼             ▼
  Escalate  SOC triage   Route to HR   Document
  to IR     (24h SLA)    / Manager     and close
  NOW
    │         │              │             │
    └─────────┴──────────────┴─────────────┘
                         │
                         ▼
              ┌───────────────────────┐
              │  5. DOCUMENT & FEED   │
              │  Update HYP file      │
              │  Update scenario if   │
              │  new behavior found   │
              │  Update KPI-07        │
              └───────────────────────┘
```

---

## 5. Detection Development Lifecycle

```
  Active scenario identified (no detection exists)
          │
          ▼
  ┌───────────────────────┐
  │  1. LOGIC DERIVATION  │
  │  Read scenario phases │
  │  Define plain-English │
  │  detection statement  │
  └──────────┬────────────┘
             │
             ▼
  ┌───────────────────────┐
  │  2. RULE AUTHORING    │
  │  Write Sigma rule     │
  │  Add exclusions       │
  │  Add comments         │
  └──────────┬────────────┘
             │
             ▼
  ┌───────────────────────┐
  │  3. PEER REVIEW       │◄── Changes needed ──► (back to author)
  └──────────┬────────────┘
             │  Approved
             ▼
  ┌───────────────────────┐
  │  4. PRE-DEPLOY TEST   │
  │  Run TC-[ID] first    │◄── FAIL ──► Fix rule ──► Retest
  └──────────┬────────────┘
             │  PASS
             ▼
  ┌───────────────────────┐
  │  5. DEPLOY            │
  │  Alert-only 14 days   │
  │  Monitor FP rate      │
  └──────────┬────────────┘
             │
             ▼
  ┌───────────────────────┐
  │  6. TUNE & STABILIZE  │
  │  Adjust thresholds    │
  │  Add exclusions       │
  │  Set final severity   │
  └──────────┬────────────┘
             │
             ▼
  ┌───────────────────────┐
  │  7. LINK & DOCUMENT   │
  │  Update scenario card │
  │  Update ATT&CK map    │
  │  Update KPI-01        │
  └───────────────────────┘
             │
             ▼
  Annual review + re-validate on data source changes
```

---

## 6. Incident Response Decision Flow (Insider Threat)

```
  Alert fires OR hunt finding escalated
          │
          ▼
  ┌───────────────────────────┐
  │  TRIAGE                   │
  │  Identify: user, behavior,│
  │  data scope, HR status    │
  └──────────┬────────────────┘
             │
      Is this legitimate?
      ┌──────┴──────┐
      │             │
     Yes            No / Unknown
      │             │
      ▼             ▼
  Close as     ┌────────────────────────┐
  False Pos.   │  NOTIFY LEGAL          │◄── REQUIRED before account action
               │  (within SLA)          │
               └──────────┬─────────────┘
                          │
                 ┌────────┴───────────┐
                 │                    │
           Covert window         Overt action
           needed                approved
                 │                    │
                 ▼                    ▼
           Preserve logs       Contain account
           Investigate         Preserve logs
           (no account         Investigate
            action yet)
                 │                    │
                 └────────┬───────────┘
                          │
                          ▼
               ┌────────────────────────┐
               │  SCOPE DETERMINATION   │
               │  What was accessed?    │
               │  What was taken?       │
               │  How long? By whom?    │
               └──────────┬─────────────┘
                          │
               Breach triggers notification?
               ┌──────────┴───────────┐
               │                      │
              Yes                     No
               │                      │
               ▼                      ▼
          Legal drives           HR/Legal drive
          breach notification    employment action
               │                      │
               └──────────┬───────────┘
                          │
                          ▼
               ┌────────────────────────┐
               │  POST-INCIDENT         │
               │  Update scenario card  │
               │  Fix detection gaps    │
               │  Update KPIs           │
               └────────────────────────┘
```

---

## Changelog

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-03-26 | Initial workflow diagrams |
