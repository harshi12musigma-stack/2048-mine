# Issue #7 — Problem Statement (Aligned with Business Objectives)
**Project:** alpha-test — Churn Prediction Pipeline (Airflow + Snowflake + Python)
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Completed:** 2026-03-30

---

## 1. Formal Problem Statement

> **The business is losing approximately $625,000 per year in revenue due to customer churn, with an additional $216,000 wasted annually on retention campaigns triggered by an unvalidated, rule-based scoring system with a ~40% false positive rate. The current process is entirely manual — requiring 30 analyst hours per month — and operates on a monthly cadence that is too slow for timely intervention. There is no ML-driven churn prediction capability, no automated data pipeline, and no feedback loop between retention outcomes and scoring logic.**
>
> **The objective is to build an automated, ML-driven churn prediction pipeline on Airflow and Snowflake that scores customers weekly, reduces false positives to <10%, and enables the retention team to intervene before churn occurs — protecting an estimated $62,500–$93,750 in annual revenue and saving ~$162,000 in wasted retention spend.**

---

## 2. Problem Decomposition (MECE)

### 2.1 Data Problem
- No unified, reliable customer data asset — data is fragmented across CRM, Billing DB, and S3 event logs
- No automated ingestion pipeline — data must be manually pulled monthly
- No schema contracts or data quality guarantees across source systems

### 2.2 Analytics Problem
- No ML model for churn prediction — current rules are static, unvalidated, and 18 months out of date
- No feature engineering or signal amplification — raw transactional signals not enriched
- No model retraining or drift detection — model accuracy degrades silently over time

### 2.3 Operational Problem
- Monthly cadence too slow — customers churn before intervention
- No prioritization within at-risk list — high-risk and marginal customers treated identically
- No feedback loop — retention outcomes not captured or used to improve the model

### 2.4 Infrastructure Problem
- Snowflake is provisioned but not fully utilized as the analytics layer
- Airflow is available but no DAGs exist for data or ML pipelines
- No CI/CD for pipeline code; no data quality gates; no deployment automation

---

## 3. Objectives (Linked to Business Goals)

| # | Objective | Target | Timeline |
|---|---|---|---|
| O-01 | Build automated data ingestion pipeline (CRM + Billing + Events → Snowflake Bronze/Silver/Gold) | 100% automated, daily refresh | By Week 6 |
| O-02 | Train and deploy ML churn prediction model | AUC ≥ 0.80; recall ≥ 0.75 on high-risk segment | By Week 9 |
| O-03 | Reduce false positive rate from ~40% to <10% | FPR < 10% on production scoring run | By Week 10 |
| O-04 | Deliver weekly churn scores to Snowflake Gold layer | Available by 06:00 UTC every Monday | By Week 10 |
| O-05 | Eliminate 25+ analyst hours/month of manual scoring work | Analyst oversight only (~5 hrs/month) | By Week 10 |
| O-06 | Protect $62,500–$93,750 in annual revenue | Measured at 6-month post-launch review | By Week 52 |

---

## 4. Out of Scope (Phase 1)

| Item | Reason |
|---|---|
| Automated CRM task creation from churn scores | Phase 2 — CRM integration work |
| Real-time / event-driven scoring | Phase 2 — weekly batch sufficient for Phase 1 |
| Intervention recommendation engine | Phase 2 — deferred by BSH sponsor (C-03 from conflict register) |
| Executive churn dashboard | Phase 2 — downstream of scoring pipeline |
| Multi-model ensemble / AutoML | Considered but deferred — XGBoost baseline first |
| GDPR / CCPA full compliance review | Compliance team to run in parallel — not blocking Phase 1 build |

---

## 5. Success Criteria

| Criterion | Measurement Method | Threshold |
|---|---|---|
| Pipeline uptime | Airflow DAG success rate | ≥ 99% weekly runs succeed |
| Model AUC | Held-out test set evaluation | ≥ 0.80 |
| Recall (high-risk segment) | Confusion matrix on test set | ≥ 0.75 |
| False positive rate | Confusion matrix on test set | ≤ 10% |
| Scoring latency | Time from DAG trigger to Gold table availability | ≤ 4 hours |
| Analyst time savings | Monthly hours log | ≤ 5 hrs/month remaining |
| Revenue impact | 6-month churn rate comparison (pre vs post) | ≥ 10% churn rate reduction |

---

## 6. Alignment to Mu Sigma Delivery Standards

This problem statement aligns with:
- **muAoPS Category 1: Customer Strategy** — Churn Prediction & Retention
- **Techniques:** XGBoost, Logistic Regression, SHAP, Survival Analysis (if needed)
- **Delivery framework:** Data Engineering (Airflow + Snowflake medallion) + Data Science (ML scoring)
- **Decision consumed by:** Retention team (weekly at-risk list from Snowflake Gold)

---

_Deliverable generated as part of Phase 1 / Milestone 1 — BRD execution loop._
