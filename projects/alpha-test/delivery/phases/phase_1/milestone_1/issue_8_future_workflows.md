# Issue #8 — Future Workflow Mapping with Automation Opportunities
**Project:** alpha-test — Churn Prediction Pipeline (Airflow + Snowflake + Python)
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Completed:** 2026-03-30

---

## 1. To-Be Workflow Overview

The future state replaces 4 manual monthly workflows with 3 automated pipelines triggered by Airflow, storing all data in Snowflake and surfacing churn scores to the retention team weekly.

---

## Future Workflow 1: Automated Daily Data Ingestion Pipeline

**Trigger:** Airflow DAG scheduled daily at 02:00 UTC | **Owner:** Automated (TDS3 oversight) | **Latency target:** ≤ 2 hours

```
[Salesforce CRM API]      [PostgreSQL Billing DB]     [S3 Event Logs]
        │                         │                         │
        ▼                         ▼                         ▼
  [Airflow DAG: ingest_crm]  [Airflow DAG: ingest_billing]  [Airflow DAG: ingest_events]
  (incremental, CDC-ready)   (incremental watermark)        (partitioned by date)
        │                         │                         │
        └─────────────────────────┼─────────────────────────┘
                                  ▼
                        [Snowflake: BRONZE layer]
                        Raw tables, append-only, no transforms
                        Schema validation gate → fail = alert
                                  │
                        [Airflow DAG: transform_silver]
                        Cleanse, deduplicate, standardize IDs
                        Apply PII masking rules
                                  │
                        [Snowflake: SILVER layer]
                        Cleansed, conformed, joined tables
                        customer_activity, customer_profile, payment_events
```

**Automation Gain:** Eliminates 5 hrs/month of manual extraction across 3 source systems.

---

## Future Workflow 2: Weekly ML Churn Scoring Pipeline

**Trigger:** Airflow DAG scheduled Sunday 22:00 UTC | **Owner:** Automated (TDS3 oversight) | **SLA:** Scores in Gold by Monday 06:00 UTC

```
[Snowflake: SILVER layer]
customer_activity + customer_profile + payment_events
        │
        ▼
[Airflow DAG: build_features]
Feature engineering:
  - Recency: days since last purchase/login
  - Frequency: transaction count (30/60/90-day windows)
  - Monetary: avg spend, total spend, payment trend
  - Engagement: feature usage frequency, support ticket count
  - Lifecycle: tenure months, subscription age
        │
        ▼
[Snowflake: SILVER — feature_store table]
Reusable features (versioned)
        │
        ▼
[Airflow DAG: score_churn]
Load latest approved model from model registry
Apply model to all eligible customers (≥ 90 days history)
Generate: churn_probability (0–1), risk_tier (High / Medium / Low / Insufficient Data)
Generate: top-3 SHAP feature contributions for High risk tier
        │
        ▼
[Snowflake: GOLD — churn_scores table]
customer_id | score_date | churn_probability | risk_tier | top_features | model_version
        │
        ▼
[Airflow DAG: validate_scores]
Data quality checks:
  - Score distribution within expected range
  - No customers missing from eligible population
  - High-risk count within ±30% of prior week
  → PASS: mark run complete, notify retention team
  → FAIL: alert TDS3 on-call, pause downstream
```

**Automation Gain:** Eliminates 3 hrs/month of manual scoring; delivers scores 4× faster (weekly vs monthly).

---

## Future Workflow 3: Retention Team Consumption

**Trigger:** Churn scores available in Gold layer (Monday morning) | **Owner:** Retention Team

```
[Snowflake: GOLD — churn_scores]
        │
        ▼
[Retention Team Self-Service Query / BI Tool]
  → Filter by risk_tier = 'High'
  → Sort by churn_probability DESC
  → View top_features for intervention context
        │
        ▼
[CRM Manual Task Creation (Phase 1)]
  → Retention Manager creates outreach tasks (same as today)
  → Now prioritized by risk score magnitude
  → Context available: why this customer is at risk (top SHAP features)
        │
      [Phase 2 → Automated CRM task creation via Snowflake → CRM integration]
```

**Improvement vs today:** Retention team receives prioritized list with context every Monday — no more undifferentiated flat list. Manual CRM work remains Phase 1; automation in Phase 2.

---

## Future Workflow 4: Model Retraining Pipeline (Monthly)

**Trigger:** Airflow DAG scheduled 1st Sunday of each month | **Owner:** Automated (TDS3 approval gate)

```
[Snowflake: GOLD — churn_scores (historical)]
[Snowflake: SILVER — actual_churn_outcomes]  ← fed from CRM churned customer list
        │
        ▼
[Airflow DAG: retrain_model]
  - Pull 12-month feature + outcome window
  - Train XGBoost model with cross-validation
  - Evaluate: AUC, Precision, Recall, F1
  - Compare to current production model metrics
        │
        ▼
  [If new model ≥ production by ≥ 2% AUC]
    → Stage new model in model registry
    → Notify TDS3 lead for approval
    → TDS3 approves → promote to production
  [If no improvement]
    → Log metrics, keep current model, send alert
        │
        ▼
[Model Registry (MLflow / Snowflake metadata table)]
model_version | train_date | AUC | Precision | Recall | status
```

**Automation Gain:** Eliminates model drift; ensures scoring accuracy improves over time.

---

## 2. As-Is vs To-Be Comparison

| Dimension | As-Is | To-Be |
|---|---|---|
| Data freshness | Monthly manual pull | Daily automated ingestion |
| Scoring model | Static rules (18 months old) | ML model, monthly retraining |
| Scoring cadence | Monthly | Weekly (Sunday night) |
| Score availability | Monday–Wednesday (after analyst work) | Monday 06:00 UTC (guaranteed SLA) |
| Analyst effort | 30 hrs/month | ~5 hrs/month (oversight + approvals) |
| False positive rate | ~40% | Target < 10% |
| Retention prioritization | Flat list (no priority) | Ranked by churn probability |
| Intervention context | None | Top-3 risk factors (SHAP) |
| Model performance tracking | None | Monthly AUC/recall logged |
| Feedback loop | None | Actual churn outcomes feed retraining |

---

## 3. Automation Summary

| Workflow | Automation Level | Remaining Manual Work |
|---|---|---|
| Daily data ingestion | ✅ Fully automated | Schema change monitoring (quarterly) |
| Weekly churn scoring | ✅ Fully automated | Score quality review (TDS3, 1 hr/week) |
| Model retraining | ✅ Automated with human approval gate | TDS3 reviews and approves new model monthly |
| Retention consumption | 🟡 Partially automated | CRM task creation manual (Phase 2 for full automation) |
| Reporting | 🟡 Scores available in Snowflake | Dashboard build Phase 2 |

---

_Deliverable generated as part of Phase 1 / Milestone 1 — BRD execution loop._
