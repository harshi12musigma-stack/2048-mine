# Business Requirement Document (BRD)
## Churn Prediction Pipeline — Airflow + Snowflake + Python
**Project:** alpha-test | **Version:** v1.0 | **Date:** 2026-03-30
**Status:** DRAFT — Pending Stakeholder Review & Sign-off

---

## Document Control

| Field | Detail |
|---|---|
| Project Name | Churn Prediction Pipeline |
| Prepared By | Ved (Mu Sigma Digital Employee) |
| Review Owner | openclaw-control-ui |
| Tech Stack | Apache Airflow · Snowflake · Python (XGBoost, Pandas, SHAP) |
| Delivery Timeline | 12 weeks |
| Document Version | 1.0 |
| Classification | Internal — Mu Sigma Confidential |

---

## Table of Contents

1. Executive Summary
2. Stakeholder Map & RACI
3. Problem Statement
4. Current State (As-Is)
5. Future State (To-Be)
6. Business Impact & ROI
7. Business KPIs & Success Criteria
8. Assumptions
9. Risk Register
10. Out of Scope
11. Open Items & Pre-Sign-off Decisions
12. Sign-off

---

## 1. Executive Summary

The business is losing approximately **$625,000 per year** in revenue to customer churn, with an additional **$216,000 wasted annually** on poorly targeted retention campaigns (current false positive rate: ~40%). The existing process is entirely manual — consuming 30 analyst hours per month — and operates on a monthly cadence that is too slow for timely intervention.

This project will deliver an **automated, ML-driven churn prediction pipeline** built on Apache Airflow and Snowflake that:
- Ingests customer data daily from CRM, billing, and event log sources
- Scores all eligible customers weekly using an XGBoost-based model
- Delivers ranked churn scores to Snowflake Gold by Monday 06:00 UTC
- Targets AUC ≥ 0.80, Recall ≥ 0.75, and False Positive Rate ≤ 10%
- Protects an estimated **$62,500–$93,750 in annual revenue** and saves **~$162,000** in wasted retention spend
- Delivers **ROI of ~314%** with a payback period of under 3 months

---

## 2. Stakeholder Map & RACI

### 2.1 Executive / BSH Sponsors

| Name | Role | Location |
|---|---|---|
| Alokesh Das | Business Unit Head (BSH) | Bangalore ITPL |
| Chandrasekar Balasubramanian | Business Unit Head (BSH) | Bangalore ITPL |
| Rajdeep Roy Choudhury | Business Unit Head (BSH) | Bangalore ITPL |
| Tanmay Sengupta | Business Unit Head | Austin |
| Richa Gupta | Business Unit Head | Austin |
| Zubin Dowlaty | Head of Innovation & Development | Austin |
| Naseer Ahmed Naqash | Head - IT & Infosec | Bangalore ITPL |

### 2.2 Delivery Team Bands

| Band | Persona | Headcount | Project Role |
|---|---|---|---|
| TDS4 | Project Leader | 359 | Governance, milestone approvals, client liaison |
| TDS3 | Problem Solver | 458 | Pipeline development, ML model, QA lead |
| TDS2 | Analytical Builder | 288 | ETL build, transformation logic |
| TDS1 | Curious Explorer | 433 | Data profiling, EDA, documentation |

### 2.3 RACI Summary

| Activity | BSH | TDS4 | TDS3 | TDS2 | TDS1 | IT |
|---|---|---|---|---|---|---|
| BRD sign-off | A | R | C | I | I | I |
| Data provisioning | I | A | C | I | I | R |
| Architecture design | C | A | R | C | I | C |
| Pipeline development | I | A | C | R | C | I |
| ML model build | I | A | R | C | C | I |
| QA & validation | I | A | R | R | C | I |
| Deployment | C | A | R | C | I | R |

*R=Responsible · A=Accountable · C=Consulted · I=Informed*

*→ Full detail: `issue_1_stakeholders.md`*

---

## 3. Problem Statement

> The business is losing approximately $625,000 per year in revenue due to customer churn, with an additional $216,000 wasted annually on retention campaigns triggered by an unvalidated, rule-based scoring system with a ~40% false positive rate. The current process is entirely manual — requiring 30 analyst hours per month — and operates on a monthly cadence that is too slow for timely intervention. There is no ML-driven churn prediction capability, no automated data pipeline, and no feedback loop between retention outcomes and scoring logic.
>
> The objective is to build an automated, ML-driven churn prediction pipeline on Airflow and Snowflake that scores customers weekly, reduces false positives to <10%, and enables the retention team to intervene before churn occurs.

### Problem Decomposition

| Dimension | Root Cause |
|---|---|
| **Data** | Fragmented sources (CRM + Billing + S3); no pipeline; no schema contracts |
| **Analytics** | Static rules, 18 months old, no ML, no retraining |
| **Operational** | Monthly cadence, no prioritization, no feedback loop |
| **Infrastructure** | Snowflake and Airflow provisioned but unused |

*→ Full detail: `issue_7_problem_statement.md`*

---

## 4. Current State (As-Is)

### 4.1 As-Is Process Summary

| Workflow | Duration | Effort | Key Problem |
|---|---|---|---|
| Monthly data extraction | 1× month | 5 hrs | Manual SQL from 3 disconnected systems |
| Rule-based churn scoring | 1× month | 3 hrs | 40% false positive rate, rules 18 months stale |
| Retention campaign execution | 2 weeks/month | 2 days | No prioritization, 40% outcome capture rate |
| Monthly churn report | 1× month | 3 hrs | No real-time access, analyst-dependent |

**Total analyst effort:** 30 hrs/month | **Process cadence:** Monthly | **Feedback loop:** None

### 4.2 Pain Points (Top 5)

1. No unified customer data source — joins fail due to ID mismatches
2. Rule-based scoring unvalidated → ~40% false positive rate
3. Monthly cadence too slow — customers churn before intervention
4. No feedback loop — retention outcomes never feed back to scoring
5. 30 hrs/month analyst time consumed by mechanical work

### 4.3 Current Systems

| System | Type | Status |
|---|---|---|
| Salesforce CRM | Customer profile, support, outreach | Operational |
| PostgreSQL Billing DB | Payments, subscriptions | Operational |
| S3 Event Logs | Login, usage events (JSON) | Operational |
| Snowflake | Analytics warehouse | Provisioned, underutilized |
| Airflow | Orchestration | Available, no DAGs built |

*→ Full detail: `issue_4_current_processes.md`, `issue_5_asis_workflows.md`*

---

## 5. Future State (To-Be)

### 5.1 To-Be Pipeline Architecture

```
[CRM / Billing DB / S3] → [Airflow: Daily Ingest] → [Snowflake BRONZE]
                                                             ↓
                                               [Airflow: transform_silver]
                                                             ↓
                                                   [Snowflake SILVER]
                                               (cleansed + feature_store)
                                                             ↓
                    [Airflow: score_churn — Sunday 22:00 UTC]
                                                             ↓
                                                   [Snowflake GOLD]
                                             churn_scores table
                                     (available Monday 06:00 UTC)
                                                             ↓
                                           [Retention Team Consumption]
                                         (ranked + SHAP context per customer)
```

### 5.2 Key Improvements vs Current State

| Dimension | As-Is | To-Be |
|---|---|---|
| Scoring cadence | Monthly | Weekly |
| False positive rate | ~40% | < 10% |
| Analyst effort | 30 hrs/month | ~5 hrs/month |
| Scoring model | Static rules | ML (XGBoost, monthly retraining) |
| Retention context | None | Top-3 SHAP risk factors |
| Feedback loop | None | Actual outcomes → monthly retraining |

*→ Full detail: `issue_8_future_workflows.md`*

---

## 6. Business Impact & ROI

| Metric | Value |
|---|---|
| Annual revenue lost to churn (current) | ~$625,000 |
| Revenue protected by 10% churn reduction | ~$62,500/year |
| Wasted retention spend (current) | ~$216,000/year |
| Savings from FPR reduction to <10% | ~$162,000/year |
| Analyst time cost savings | ~$13,380/year |
| **Total Annual Benefit** | **~$237,880** |
| Build Investment | ~$57,500 |
| **Year 1 ROI** | **~314%** |
| **Payback Period** | **< 3 months** |

*→ Full detail: `issue_6_business_impact.md`*

---

## 7. KPIs & Success Criteria

### Business KPIs
- Monthly churn rate: ≤ 2.25% (from ~2.5% baseline)
- Retention campaign conversion: ≥ 25%
- Wasted retention spend: ≤ $4,500/month

### Pipeline KPIs
- DAG success rate: ≥ 99%
- Scoring SLA compliance: ≥ 98% (scores by Monday 06:00 UTC)
- Data quality pass rate: ≥ 97%

### Model KPIs
- AUC-ROC: ≥ 0.80
- Recall (High Risk): ≥ 0.75
- False Positive Rate: ≤ 10%
- F1 Score: ≥ 0.70

### Go/No-Go Gates

| Gate | Phase | Decision Maker |
|---|---|---|
| G-01 | BRD sign-off | BSH Sponsor |
| G-02 | Data profiling pass | Analytics Lead |
| G-03 | Pipeline staging validation | TDS4 |
| G-04 | Model quality (AUC ≥ 0.80) | Analytics Lead + BSH |
| G-05 | Production go-live | TDS4 + IT Lead |

*→ Full detail: `issue_9_kpis.md`*

---

## 8. Assumptions

| # | Assumption | Risk if Wrong |
|---|---|---|
| A-01 | Stable customer_id across CRM, Billing, S3 | High — joins fail |
| A-02 | ≥ 12 months historical data available | High — model training insufficient |
| A-03 | Actual churn outcomes in CRM (labeled data) | Critical — supervised ML blocked |
| A-05 | PII masking rules approved before build | High — ingestion blocked |
| A-07 | Airflow and Snowflake environments ready | High — development blocked |
| A-13 | Single agreed churn definition, locked at BRD | Critical — all downstream work at risk |
| A-14 | "Customer" = account-level entity | High — data model restructures |

*→ Full list (14 assumptions): `issue_10_assumptions_risks.md`*

---

## 9. Risk Register Summary

| # | Risk | Level | Key Mitigation |
|---|---|---|---|
| R-01 | No stable customer_id | 🔴 High | Data profiling in Phase 5 before build |
| R-02 | Insufficient labeled churn history | 🔴 High | Validate in Phase 5; proxy label fallback |
| R-03 | PII not classified before ingestion | 🔴 High | Compliance sign-off in Week 1–2 |
| R-06 | Churn definition changes after modeling begins | 🔴 High | Lock definition at BRD sign-off |
| R-05 | Model AUC < 0.80 at launch | 🟡 Medium | Iterative tuning; rule-based fallback |
| R-09 | Airflow instability in production | 🟡 Medium | Staging tests; retry logic; on-call alerts |

*→ Full register (13 risks): `issue_10_assumptions_risks.md`*

---

## 10. Out of Scope (Phase 1)

| Item | Reason |
|---|---|
| Automated CRM task creation | Phase 2 |
| Real-time scoring | Phase 2 |
| Intervention recommendation engine | Phase 2 — BSH deferred |
| Executive churn dashboard | Phase 2 |
| Multi-model ensemble / AutoML | Phase 1 = XGBoost baseline |
| GDPR / CCPA full compliance | Running in parallel — not blocking |

---

## 11. Open Items (Resolve Before BRD Sign-off)

| # | Item | Owner | Due |
|---|---|---|---|
| OI-01 | Confirm baseline churn rate (last 6 months avg) | TDS3 Analyst | Week 1 |
| OI-02 | Confirm Snowflake compute tier and monthly cost ceiling | IT | Week 1 |
| OI-03 | Get formal IT decision on MLflow vs Snowflake metadata tracking | IT | Week 2 |
| OI-04 | Define "customer" entity — account vs contact vs subscription | BSH Sponsor | Week 1 |
| OI-05 | Confirm churn definition and lock it formally | BSH Sponsor | Week 1 |

---

## 12. Sign-off

| Role | Name | Signature | Date |
|---|---|---|---|
| Project Owner / Approver | openclaw-control-ui | ___________________ | __________ |
| BSH Executive Sponsor | ___________________ | ___________________ | __________ |
| Analytics Lead (TDS3) | ___________________ | ___________________ | __________ |
| IT / Infosec Lead | ___________________ | ___________________ | __________ |

---

*This BRD was compiled from Issues #1–#10 of the Phase 1 execution loop.*
*Supporting deliverables located at: `projects/alpha-test/delivery/phases/phase_1/milestone_1/`*
