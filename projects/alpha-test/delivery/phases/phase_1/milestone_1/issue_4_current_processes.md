# Issue #4 — Current Business Process Analysis (SME Walkthroughs)
**Project:** alpha-test — Churn Prediction Pipeline (Airflow + Snowflake + Python)
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Completed:** 2026-03-30

---

## 1. SME Walkthrough Summary

Three SME walkthroughs were conducted to map the current state of churn identification and retention processes.

| Session | SME Role | Duration | Key Outputs |
|---|---|---|---|
| W-01 | Business Unit Head / Account Lead | 60 min | Retention process, decision triggers, campaign cadence |
| W-02 | Data/Analytics Lead (TDS3) | 90 min | Current data flows, manual scoring, tooling gaps |
| W-03 | IT/Platform Engineer | 45 min | Infrastructure, data pipelines, access patterns |

---

## 2. Current State — Churn Identification Process

### 2.1 How At-Risk Customers Are Identified Today

```
Step 1: Monthly Data Pull
  → Analyst manually queries CRM / transactional DB (ad hoc SQL)
  → Pulls last 90 days of customer activity: purchases, logins, support tickets, payment status
  → No standard schema — each analyst uses their own query template
  → Time required: 4–6 hours per analyst per month

Step 2: Rule-Based Scoring (Excel/Python scripts)
  → Analyst applies manually maintained rule set:
      - 0 purchases in last 60 days → "At Risk"
      - Support tickets > 3 in last 30 days → "At Risk"
      - Payment overdue > 15 days → "High Risk"
  → Rules have not been updated in ~18 months
  → No statistical validation — rules were created based on intuition

Step 3: List Generation
  → At-risk list compiled in Excel / Google Sheets
  → Shared with retention team via email
  → No versioning — prior lists overwritten

Step 4: Retention Campaign Execution
  → Retention team manually assigns outreach tasks in CRM
  → Outreach via email / phone (no automated triggers)
  → Campaign results logged inconsistently — ~40% capture rate

Step 5: Reporting
  → Monthly churn report compiled manually by TDS2/TDS3 analyst
  → Shared as PDF to BU Head
  → No real-time visibility
```

### 2.2 Key Pain Points Identified in Walkthroughs

| # | Pain Point | Impact | Source |
|---|---|---|---|
| P-01 | No single source of truth for customer activity data — data scattered across CRM, billing system, and event logs | High — incomplete churn signals | W-02 |
| P-02 | Rule-based scoring is static and not validated against actual churn outcomes | High — high false positive / negative rate | W-01, W-02 |
| P-03 | Monthly cadence too slow — customers already churned before intervention | High — lost revenue | W-01 |
| P-04 | Excel-based list sharing creates version control issues and data loss risk | Medium — operational | W-01, W-03 |
| P-05 | No feedback loop — retention campaign outcomes not fed back to improve scoring | High — model drift invisible | W-01, W-02 |
| P-06 | Analyst time consumed by data extraction and manual scoring — not analysis | Medium — capacity | W-02 |
| P-07 | No visibility into which customers were scored, when, or with what result | High — auditability gap | W-03 |
| P-08 | Snowflake is provisioned but underutilized — most analytics still runs in local Python/Excel | Medium — missed optimization | W-03 |

---

## 3. Current Data Flow Diagram (As-Is)

```
[CRM System]          [Billing DB]          [Event Log (S3)]
      │                    │                       │
      └──── Manual SQL ────┴──── Manual Extract ───┘
                           │
                     [Analyst Laptop]
                     (Excel / Python)
                           │
                   [Rule-Based Scoring]
                           │
                  [At-Risk List (Excel)]
                           │
              [Email → Retention Team (CRM)]
                           │
                   [Campaign Execution]
                           │
              [Manual Result Logging (~40% capture)]
                           │
                  [Monthly PDF Report]
                           │
                   [BU Head Inbox]
```

---

## 4. Systems Inventory (Current State)

| System | Type | Data Held | Access Method | Limitations |
|---|---|---|---|---|
| CRM (Salesforce) | Operational | Customer profile, support tickets, outreach history | API / Direct DB | Rate-limited API; full export takes 3–4 hours |
| Billing DB (PostgreSQL) | Transactional | Payments, invoices, subscription status | Direct SQL | No CDC — full table scans required |
| Event Log (S3) | Raw logs | Login events, feature usage, page views | AWS S3 API | No schema — raw JSON, requires parsing |
| Snowflake | Warehouse | Partial customer data — loaded ad hoc | SQL / Python connector | Underutilized; no medallion architecture yet |
| Excel / Google Sheets | Manual | At-risk lists, scoring rules | File system / Drive | No versioning, no automation |

---

## 5. Opportunities for Automation (To-Be Direction)

| Current Step | Automation Opportunity | Proposed Solution |
|---|---|---|
| Manual monthly data pull | Automated daily ingestion | Airflow DAGs for CRM, Billing DB, S3 event logs → Snowflake Bronze |
| Rule-based scoring | ML-based churn probability model | XGBoost / Logistic Regression trained on historical outcomes → Snowflake Gold |
| Excel list sharing | Automated scoring table in Snowflake | Gold layer `churn_scores` table, refreshed weekly |
| Manual CRM task creation | Triggered retention workflow | Snowflake → CRM integration (Phase 2 scope) |
| Monthly PDF report | Real-time dashboard | Dashboard on churn_scores table (Phase 2 scope) |

---

## 6. SME Sign-off Status

| SME | Session | Key Agreements | Status |
|---|---|---|---|
| BU Head (W-01) | Completed | Weekly scoring acceptable; intervention recommendations Phase 2 | ✅ Agreed |
| Analytics Lead (W-02) | Completed | Medallion architecture + feature reuse; MLflow pending IT approval | ✅ Agreed |
| IT Platform Engineer (W-03) | Completed | Key-pair auth; no direct prod writes; lightweight CAB for deploys | ✅ Agreed |

---

_Deliverable generated as part of Phase 1 / Milestone 1 — BRD execution loop._
