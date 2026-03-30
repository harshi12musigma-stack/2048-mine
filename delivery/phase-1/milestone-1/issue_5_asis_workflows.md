# Issue #5 — As-Is Workflow Documentation
**Project:** alpha-test | **Client:** Mars India
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Date:** 2026-03-30
**Source:** W-01 (Priya Sharma), W-02 (Karthik Nair) session transcripts + user confirmation

---

> ✅ All workflows documented from confirmed session transcripts only.
> No content assumed. [TBD] marks items requiring further confirmation.

---

## Workflow Overview

Four as-is workflows documented across the full churn identification and retention lifecycle:

| # | Workflow | Owner | Frequency | Manual effort |
|---|---|---|---|---|
| W1 | Data extraction & aggregation | Karthik's team (dbt + Fivetran) | Daily (auto) + weekly report | ~30 min monitoring |
| W2 | At-risk list generation & manual processing | Deepa (coordinator) | Every Monday | ~2 hours |
| W3 | Retention campaign execution | Priya's team + Deepa | Weekly | High — 7-person desk + manual tools |
| W4 | Executive reporting | Analyst → Rohan | Weekly | ~1–2 hours (manual aggregation) |

---

## Workflow 1 — Data Extraction & Aggregation

**Owner:** Karthik Nair (Analytics & Data Lead) | **Cadence:** Daily automated + Monday report trigger

| Step | Actor | System | Data Used | Manual? | Output |
|---|---|---|---|---|---|
| 1.1 | Fivetran (automated) | Salesforce CRM → Snowflake | Customer master: customer_id, segment, region, registration_date | ❌ Automated | Snowflake table (nightly, ~2 AM) |
| 1.2 | Fivetran (automated) | Oracle e-commerce → Snowflake | Transactions: purchase dates, amounts, counts | ❌ Automated | Snowflake table (nightly, ~2 AM) |
| 1.3 | Custom Python script (fragile) | Capillary SFTP → S3 → Snowflake | Loyalty: tier, points balance, redemptions | ⚠️ Semi-auto — breaks on format change | Raw loyalty table in Snowflake |
| 1.4 | dbt (scheduled) | Snowflake marts | All sources joined + 90d/12m windows computed | ❌ Automated | `marts` schema — customer-level fact table |
| 1.5 | Redash (scheduled cron) | Snowflake marts | customers with days_since_last_purchase ≥ 45 (RC) or ≥ 60 (PED) | ❌ Automated | CSV emailed to Priya's team every Monday |

**Known issues:**
- dbt job is **full refresh** — rescans entire transaction history daily. Runtime: ~90 min on medium warehouse. No incremental model.
- Capillary script broke twice in last year due to upstream format changes — no DQ checks.
- Redash cron missed delivery 2 weeks prior to session due to server maintenance — not noticed until Wednesday (2-day blind spot).

---

## Workflow 2 — At-Risk List Generation & Manual Processing

**Owner:** Deepa (Retention Coordinator) | **Cadence:** Every Monday morning | **Effort:** ~2 hours

| Step | Actor | System | Action | Manual? | Output |
|---|---|---|---|---|---|
| 2.1 | Deepa | Email / Excel | Receives CSV from Redash cron | ✅ Manual receipt | Raw at-risk list |
| 2.2 | Deepa | Excel | Filter by segment: separate RC Direct from Pedigree Club | ✅ Manual | Two segment-split lists |
| 2.3 | Deepa | Excel | Remove customers contacted in last 2 weeks (suppress double-ping) | ✅ Manual | De-duped list |
| 2.4 | Deepa | Excel | Remove customers with a pending delivery (active order = fine) | ✅ Manual | Clean at-risk list |
| 2.5 | Deepa | Excel | Rough prioritisation by spend (high-spend first) | ✅ Manual | Prioritised list |
| 2.6 | Deepa | Excel / Salesforce | Distribute list to retention desk and campaign team | ✅ Manual email | Working list for W3 |

**Critical issues:**
- **Single point of failure:** Deepa is the only person who knows this full filter logic. Undocumented. If she is on leave, the process does not run.
- **Two thresholds in one sheet:** RC Direct at 45 days; Pedigree Club at ~60 days. The CSV doesn't distinguish them cleanly — manual filtering required every week.
- **No version control:** Prior week's list is overwritten when the new one is processed.
- **Ghost purchasers missed:** Customers with naturally low purchase frequency (e.g. buy every 8 weeks) never trigger the 45-day rule. They disappear without being flagged.
- **Channel-switch false positives:** Customers who switched to Amazon or vet clinic continue buying Royal Canin but disappear from portal data — flagged at-risk and contacted incorrectly. *"I just bought it yesterday, why are you calling?"*

---

## Workflow 3 — Retention Campaign Execution

**Owner:** Priya Sharma / 7-person retention desk | **Cadence:** Weekly (rolling)

### RC Direct (Royal Canin) — 3-step sequence:

| Step | Day | Channel | Tool | Manual? | Volume |
|---|---|---|---|---|---|
| 3.1 | Day 45 | WhatsApp | Kaleyra (WABA) | ✅ Template sent manually | 1,200–1,500 / week normal; ~3,000 peak |
| 3.2 | Day 50 (no response) | Email | Salesforce Marketing Cloud (SFMC) | ✅ Campaign set up manually | Subset of non-responders |
| 3.3 | Day 55 (still no response) | Outbound call | Phone — 7-person retention desk | ✅ Manual outbound calls | ~4,500–5,000 calls/week capacity |

### Pedigree Club — Email + WhatsApp only:

| Step | Channel | Tool | Manual? | Exception |
|---|---|---|---|---|
| 3.1 | Email | SFMC | ✅ Manual | — |
| 3.2 | WhatsApp | Kaleyra | ✅ Manual | Gold tier or >INR 20K annual spend → escalate to call |

**Critical issues:**
- **Two disconnected campaign tools:** SFMC (email) and Kaleyra (WhatsApp) are not integrated. Manual reconciliation required every week to determine who responded to what channel.
- **No holdout group:** Retention rate is measured as re-purchase within 30 days of outreach, but no control group exists. Reported 22% RC Direct and 14% Pedigree Club retention rates are likely inflated — organic returns not separated from model-driven saves.
- **Gold/Platinum over-contact:** No suppression tier. Happy high-value customers sometimes appear on the list and get contacted unnecessarily, creating friction. *"I just bought it yesterday, why are you calling?"*
- **Capacity constraint for Pedigree:** ~8× the RC Direct customer base, but no call capacity for most of them. Call escalation threshold (INR 20K) is an informal manager rule, not data-driven.

---

## Workflow 4 — Weekly Executive Reporting

**Owner:** Karthik's analyst team → Rohan (Priya's manager) | **Cadence:** Weekly | **Effort:** ~1–2 hours

| Step | Actor | System | Action | Manual? | Output |
|---|---|---|---|---|---|
| 4.1 | Analyst | Salesforce + Excel | Pull weekly retention outcomes (who was contacted, who purchased) | ✅ Manual SQL export | Raw outcomes data |
| 4.2 | Analyst | Excel | Calculate at-risk count, actioned count, recovery rate | ✅ Manual calculation | Weekly summary stats |
| 4.3 | Analyst | Email | Send summary to Priya + Rohan | ✅ Manual email | Weekly retention summary |

**Critical issues:**
- No self-service access for Rohan — entirely dependent on analyst availability each week.
- Recovery rate calculation doesn't exclude organic returns — same problem as W3.
- No trend visibility — each week's report is standalone, no longitudinal view.

---

## Data Lineage Map (As-Is)

```
[Salesforce CRM]      [Oracle e-commerce]     [Capillary Loyalty]
  customer master        transaction history       tier + points
       │                       │                       │
       └─── Fivetran ──────────┘              SFTP → S3 → Python script
                   │                                   │
           [Snowflake — raw tables]                    │
                   │                                   │
           [dbt full-refresh mart]  ←──────────────────┘
           (90d + 12m windows, ~90 min runtime)
                   │
           [Redash SQL query]
           (45d rule RC / 60d rule PED)
                   │
           [CSV emailed Monday]
                   │
           [Deepa — Excel manual processing]
           (2 hrs: filter, dedup, prioritise)
                   │
        ┌──────────┴──────────┐
        │                     │
  [Kaleyra WhatsApp]     [SFMC Email]
  (manual campaign)      (manual campaign)
        │                     │
        └──────────┬──────────┘
                   │
          [Manual reconciliation]
                   │
          [7-person retention desk]
          (outbound calls — RC Direct)
                   │
          [Re-purchase check — 30d]
          (no holdout, no attribution)
                   │
          [Manual Excel summary]
                   │
          [Email to Rohan / Priya]
```

---

## Manual Effort Summary

| Step | Effort/Week | Automation Opportunity |
|---|---|---|
| Capillary SFTP ingestion | ~30 min (when it breaks) | Replace with robust pipeline, DQ checks |
| Deepa's Monday processing | ~2 hours | Full automation — scoring replaces manual filter |
| Campaign tool reconciliation | ~1 hour | Unified output via Salesforce integration |
| Analyst reporting | ~1–2 hours | Dashboard for Rohan — self-service |
| **Total manual** | **~4–5 hours/week** | **~90% eliminable in Phase 1 + Phase 2** |

---

*Sources: W-01 Priya Sharma (business process, campaign execution), W-02 Karthik Nair (data architecture, dbt layer).
User confirmed scope: include full pipeline (Karthik data layer) + business process (Priya) + Rohan reporting.*
