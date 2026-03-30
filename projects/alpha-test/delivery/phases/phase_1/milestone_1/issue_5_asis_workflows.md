# Issue #5 — As-Is Workflow Documentation (Systems, Data Usage, Manual Steps)
**Project:** alpha-test — Churn Prediction Pipeline (Airflow + Snowflake + Python)
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Completed:** 2026-03-30

---

## 1. Workflow Inventory

Four distinct workflows documented below, mapped across people, systems, data, and manual steps.

---

## Workflow 1: Monthly Customer Data Extraction

**Owner:** TDS2/TDS3 Analyst | **Frequency:** Monthly (1st week) | **Duration:** 4–6 hours

| Step | Actor | System | Data Used | Manual? | Output |
|---|---|---|---|---|---|
| 1.1 | Analyst | CRM (Salesforce) | Customer profiles, support tickets, outreach history (last 90 days) | ✅ Manual SQL export | CSV file (~500K rows) |
| 1.2 | Analyst | Billing DB (PostgreSQL) | Payment status, subscription type, invoices | ✅ Manual SQL | CSV file |
| 1.3 | Analyst | S3 Event Log | Login events, feature usage clicks (JSON format) | ✅ Boto3 script (ad hoc) | Parsed CSV (~2M rows) |
| 1.4 | Analyst | Analyst Laptop | Join CRM + Billing + Events on customer_id | ✅ Manual Pandas script | Merged DataFrame |
| 1.5 | Analyst | Excel / Google Sheets | Final merged dataset | ✅ Manual copy-paste | Master monthly extract |

**Known Issues:**
- No schema contract between source systems — joins fail intermittently due to ID format mismatches
- S3 log parsing script breaks on schema changes (~quarterly)
- Full extract from CRM API takes 3–4 hours due to rate limits

---

## Workflow 2: Rule-Based Churn Scoring

**Owner:** TDS2 Analyst | **Frequency:** Monthly | **Duration:** 2–3 hours

| Step | Actor | System | Data Used | Manual? | Output |
|---|---|---|---|---|---|
| 2.1 | Analyst | Excel | Master monthly extract | ✅ Manual formula application | Scored customer list |
| 2.2 | Analyst | Excel | Scoring rules (hardcoded): 0 purchases/60d = At Risk; tickets > 3/30d = At Risk; payment overdue > 15d = High Risk | ✅ Manual — rules in Excel cells | Risk label per customer |
| 2.3 | Analyst | Excel | Scored list | ✅ Manual filter and sort | "High Risk" and "At Risk" sub-lists |
| 2.4 | Analyst | Email | At-risk list (Excel attachment) | ✅ Manual email | List delivered to retention team |

**Known Issues:**
- Scoring rules last updated ~18 months ago — no validation against actual churn outcomes
- No audit trail — prior month's scored list overwritten on save
- Retention team receives file but has no way to verify provenance or data freshness

---

## Workflow 3: Retention Campaign Execution

**Owner:** Retention Team (Account Managers) | **Frequency:** Monthly | **Duration:** 1–2 weeks

| Step | Actor | System | Data Used | Manual? | Output |
|---|---|---|---|---|---|
| 3.1 | Retention Manager | Email | At-risk list from Analyst | ✅ Manual receipt and review | Task assignment decisions |
| 3.2 | Retention Manager | CRM (Salesforce) | Customer contact details | ✅ Manual CRM task creation (one by one) | Outreach tasks in CRM |
| 3.3 | Account Manager | CRM / Phone / Email | Customer profile, risk reason (not always available) | ✅ Manual outreach | Customer contact attempt |
| 3.4 | Account Manager | CRM | Outreach result (called, emailed, no answer, retained, lost) | ✅ Manual CRM note entry | Outcome log (partial) |
| 3.5 | Retention Manager | Excel | Collected outcomes | ✅ Manual aggregation | Campaign summary (monthly) |

**Known Issues:**
- CRM task creation is manual — ~2 days of admin work per campaign cycle
- ~40% of outreach outcomes are not logged in CRM (logged in personal notes or not at all)
- No prioritization within the at-risk list — all customers treated equally regardless of risk score magnitude
- No feedback to analytics team on which interventions worked

---

## Workflow 4: Monthly Churn Reporting

**Owner:** TDS3 Analyst | **Frequency:** Monthly | **Duration:** 3–4 hours

| Step | Actor | System | Data Used | Manual? | Output |
|---|---|---|---|---|---|
| 4.1 | Analyst | CRM | Churned customers (last 30 days) | ✅ Manual SQL export | Churn count |
| 4.2 | Analyst | Excel | Churn count + total active customers | ✅ Manual calculation | Churn rate % |
| 4.3 | Analyst | PowerPoint | Churn rate + campaign summary | ✅ Manual slide build | Monthly churn report (PDF) |
| 4.4 | Analyst | Email | PDF report | ✅ Manual email to BU Head | Delivered |

**Known Issues:**
- Report reflects state of data at time of extraction — no real-time accuracy
- No standardized template — report format varies by analyst
- BU Head has no self-service access — entirely dependent on analyst availability
- No trend visualization or model performance tracking

---

## 2. Manual Steps Summary (Automation Candidates)

| Workflow | Manual Step | Effort (hrs/month) | Automation Priority |
|---|---|---|---|
| W1 | CRM data extraction | 2–3 hrs | 🔴 High |
| W1 | Billing DB extraction | 1 hr | 🔴 High |
| W1 | S3 log parsing | 1–2 hrs | 🔴 High |
| W1 | Cross-system join | 1 hr | 🔴 High |
| W2 | Rule-based scoring | 2–3 hrs | 🔴 High |
| W2 | At-risk list generation | 0.5 hrs | 🔴 High |
| W3 | CRM task creation | 2 days | 🟡 Medium (Phase 2) |
| W3 | Outcome logging | Ongoing | 🟡 Medium (Phase 2) |
| W4 | Churn rate calculation | 1 hr | 🔴 High |
| W4 | Report generation | 2–3 hrs | 🟡 Medium (Phase 2 dashboard) |

**Total analyst time consumed by manual process: ~15–20 hours/month**
**Estimated automation coverage (Phase 1):** ~70% of manual effort eliminated

---

## 3. Data Lineage Map (As-Is)

```
[Salesforce CRM]          [PostgreSQL Billing]       [S3 Event Logs]
  customer_id               customer_id                 user_id
  support_tickets           payment_status              event_type
  contact_history           invoice_amount              timestamp
  account_status            subscription_type           feature_name
       │                         │                          │
       ▼                         ▼                          ▼
  [Manual SQL Export]     [Manual SQL Export]    [Ad-hoc Boto3 Script]
  (monthly, ~3–4 hrs)     (monthly, ~1 hr)       (monthly, ~1–2 hrs)
       │                         │                          │
       └─────────────── Manual Pandas Join ─────────────────┘
                                 │
                         [Master CSV Extract]
                         (Analyst Laptop)
                                 │
                    [Excel Rule-Based Scoring]
                                 │
                      [At-Risk List (Excel)]
                                 │
                        [Email Distribution]
                                 │
                    [CRM Manual Task Creation]
                                 │
                       [Outreach Execution]
                                 │
              [Partial Outcome Logging in CRM (~40%)]
                                 │
                     [Monthly Excel Aggregation]
                                 │
                       [PowerPoint PDF Report]
                                 │
                         [BU Head Email]
```

---

_Deliverable generated as part of Phase 1 / Milestone 1 — BRD execution loop._
