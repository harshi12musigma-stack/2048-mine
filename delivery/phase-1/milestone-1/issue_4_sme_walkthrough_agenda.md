# Issue #4 — SME Walkthrough: Session Agenda & Facilitation Guide
**Project:** alpha-test | **Client:** Mars India
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Date:** 2026-03-30
**Document type:** EXTERNAL_ACTION — Agenda template only. Outcomes written after session.

---

> ⚠️ **This document is a facilitation guide — not a deliverable.**
> Conduct the session with Mars stakeholders, fill in the notes sections below,
> then reply **/confirm-done** with your session notes.
> The actual Issue #4 deliverable will be written from your confirmed notes.

---

## Sessions to Conduct

Three sessions recommended — can be combined if stakeholder availability is limited:

| Session | Audience | Duration | Owner |
|---|---|---|---|
| **W-01** | Mars Business User (CRM / retention team) | 60 min | Harshita |
| **W-02** | Mars Data / Analytics contact (if available) | 60 min | Harshita + Vishwavani |
| **W-03** | Mars IT / platform contact (if available) | 45 min | Harshita + Savikar |

*If only one Mars contact is available for now, run W-01 and note gaps for W-02/W-03.*

---

## W-01 — Business Process Walkthrough (Mars CRM / Retention Team)

**Goal:** Understand the current manual process for identifying and acting on at-risk customers.

**Opening framing (say this to the interviewee):**
> *"We're not here to evaluate anything — we want to understand exactly how things work today so we design the pipeline around your actual workflow, not assumptions. Walk us through a typical week in terms of how you currently manage customer retention for Royal Canin and Pedigree."*

---

### Section A: Current At-Risk Identification Process

| Question | Notes (fill during session) |
|---|---|
| How do you currently identify customers who might be at risk of churning? | |
| What is the trigger — is it time-based (e.g. no purchase in X days), complaint-based, or something else? | |
| How frequently does this process run — daily, weekly, monthly? | |
| Who runs it — a person, a system, or a scheduled report? | |
| How long does it take end-to-end? | |
| What data do you look at when flagging someone as at-risk? | |
| Is there a formal definition of "churned" — when do you consider a customer lost? | |

---

### Section B: Retention Actions

| Question | Notes |
|---|---|
| Once an at-risk customer is identified, what happens next? | |
| Who decides what action to take — person or automated rule? | |
| What channels are used for outreach? (WhatsApp, email, phone, vet clinic, in-store) | |
| What is the typical outreach sequence and timing? | |
| Do you track whether the outreach worked? How? | |
| What is your current retention rate (% of at-risk customers who come back after outreach)? | |
| What does a "successful" retention intervention look like to you? | |

---

### Section C: Pain Points & Gaps

| Question | Notes |
|---|---|
| What is the biggest frustration with how things work today? | |
| What information do you wish you had when deciding who to contact? | |
| Are there customers you know are at risk but the current process misses? | |
| Are there customers you contact who turn out to be fine — false positives? | |
| What would make your job easier from a data / tooling perspective? | |

---

### Section D: Output Requirements

| Question | Notes |
|---|---|
| If we gave you a weekly list of at-risk customers with a risk score — what would you do with it? | |
| What format works best — Snowflake table, CSV file, direct CRM integration? | |
| Who else on your team would need access to this list? | |
| What is the minimum lead time you need before a campaign goes out? (e.g. scores by Monday to campaign by Wednesday) | |

---

## W-02 — Data & Analytics Walkthrough (Mars Data Team)

**Goal:** Understand the data architecture, what systems feed the features, and any modeling history.

| Question | Notes |
|---|---|
| Where does the customer data live — Snowflake, another warehouse, raw files? | |
| Who owns the data pipelines today? | |
| How are the 90-day and 12-month purchase aggregations currently computed? | |
| Is there a feature store or are features recomputed on demand? | |
| Has any ML modeling been done on this data before? What was the outcome? | |
| What does the full customer population look like — how many customers are in scope? | |
| Are there other data sources not in the sample we should know about? | |
| What is the data refresh cadence — how fresh is the data at any given time? | |

---

## W-03 — IT / Platform Walkthrough (Mars IT/Infosec)

**Goal:** Understand infrastructure constraints and data governance boundaries.

| Question | Notes |
|---|---|
| Is Snowflake the primary warehouse? What tier/edition? | |
| Is Airflow already deployed — if so, what type (self-hosted / MWAA / Cloud Composer)? | |
| What Python version is supported in the pipeline runtime? | |
| Are there approved package lists for the data environment? | |
| What are the data sharing constraints — can Mu Sigma read Mars Snowflake directly, or is there a data transfer process? | |
| Is the DPA signed or in process? What is the expected completion date? | |
| Are there dev/staging/prod Snowflake environments? | |
| What is the change management / deployment approval process? | |

---

## Post-Session: Notes Template

After each session, fill this in and share with the team:

```
SESSION: W-01 / W-02 / W-03
DATE: 
ATTENDEES (Mars side): 
ATTENDEES (Mu Sigma side): 

KEY FINDINGS:
1.
2.
3.

SURPRISES / THINGS WE DID NOT EXPECT:
1.
2.

OPEN ITEMS FROM SESSION:
1.
2.

CONFIRMED DECISIONS:
1.
2.
```

---

## When Done

Reply: **/confirm-done** with your session notes pasted in.
I will write the Issue #4 deliverable (current process documentation) from your confirmed notes.

**Do not proceed to Issue #5 until at least W-01 is complete.**
W-05 (As-Is Workflows) directly depends on what W-01 surfaces.
