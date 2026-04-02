# Issue #9 — Business KPIs & Success Criteria
**Project:** Mars India Churn Prediction Pipeline  
**Phase:** 1 — Requirement Gathering  
**Milestone:** 1 — Business Requirement Document  
**Date:** 2026-04-02  
**Status:** Complete  

---

## Overview

KPIs and success criteria are derived from confirmed stakeholder inputs across W-01/W-02/W-03 sessions. They are split across three layers: Business Outcomes, Model Performance, and Operational Performance.

---

## Layer 1 — Business Outcome KPIs

These are the metrics that define whether the project succeeded from Mars India's perspective.

| KPI | Baseline | Target | Measurement Method | Owner |
|-----|----------|--------|--------------------|-------|
| RC Direct churn rate | 31.4% | < 20% | Monthly cohort — % customers inactive >90d / total active start-of-month | Priya Sharma |
| Pedigree Club churn rate | ~18.5% (estimated) | < 15% | Same cohort method | Priya Sharma |
| Retention rate — RC Direct | ~22% (no holdout, inflated) | > 30% (with holdout-corrected baseline) | Re-purchase within 30d of campaign contact / total contacted | Priya Sharma |
| Revenue protected — RC Direct | — | INR 174.9 Cr/yr at 10pp churn reduction | Attributed revenue: saved customers × avg annual spend (INR 61,202) | Rohan |
| Weekly at-risk volume scored | Manual (1,200–1,500/week RC Direct) | Automated — 100% of scoreable population weekly | Count of customers scored per Sunday run | Harshita |

**Measurement window:** First two scored quarters post-launch (Q3 + Q4 2026)  
**Note:** Baseline retention rates (22% RC Direct, 14% PED Club) are self-reported without holdout. True holdout-corrected baseline to be established in first 4 weeks post-launch.

---

## Layer 2 — Model Performance KPIs

These are the technical thresholds that define model readiness for production deployment.

| KPI | Current State | Target | Notes |
|-----|--------------|--------|-------|
| Precision at top decile | ~15% (prior XGBoost — leakage-inflated) | > 40% | Primary model quality gate — approved by Harshita |
| AUC-ROC | Unknown (prior model not deployed) | > 0.75 | Minimum acceptable discriminatory power |
| Precision-Recall AUC | — | > 0.60 | More meaningful than ROC for imbalanced classes |
| Calibration (Brier score) | — | < 0.15 | Required for risk-tier probability thresholds to be meaningful |
| Bronze-tier recall | 0% (rule-based — only fires at 45d) | > 50% | Bronze has 60.7% churn rate — highest priority segment |
| Feature leakage check | Failed (prior XGBoost) | Pass — subscription_active excluded from training | Hard gate before any model goes to production |
| Class balance check | — | No model trained without addressing Bronze imbalance | LightGBM class_weight or SMOTE |

**Model sign-off gate:** All 7 KPIs must pass before promotion from staging to production.

---

## Layer 3 — Operational / Pipeline KPIs

These measure whether the pipeline runs reliably without human intervention.

| KPI | Target | Measurement |
|-----|--------|-------------|
| DAG success rate | > 99% over rolling 4-week window | Airflow task instance logs |
| Score delivery SLA | Scores in Snowflake by Monday 6 AM IST | Airflow DAG completion timestamp |
| Salesforce sync SLA | Scores in SFMC by Monday 9 AM IST | SFMC segment refresh timestamp (post Phase 1 integration confirmation) |
| Data freshness | Fivetran nightly run fresher than 48h at scoring time | DQ DAG freshness check task |
| DQ gate pass rate | 100% — no batch processed with null rate > 5% on key features | DQ DAG validation task |
| Capillary SFTP alert time | < 15 min from failure to Slack alert | DQ DAG alerting task |
| Manual interventions per month | 0 (target) — currently ~4/month (Deepa) | Incident log |

---

## Success Criteria by Milestone

### Phase 1 Complete (BRD approved + code-complete June 1)
- [ ] Stakeholder sign-off on churn definition (days_since_last_purchase ≥ 90)
- [ ] Feature leakage resolution documented (subscription_active decision)
- [ ] Infrastructure access confirmed (DPA signed, service account provisioned)
- [ ] LightGBM package approved in MWAA

### Model Staging (Target: mid-May 2026)
- [ ] Precision@decile > 40% on holdout test set
- [ ] AUC-ROC > 0.75
- [ ] No leakage features in production feature set
- [ ] Bronze-tier recall > 50%

### Pipeline UAT (Target: June 1–21 2026)
- [ ] 3 consecutive Sunday runs complete without manual intervention
- [ ] Scores delivered to Snowflake within SLA on all 3 runs
- [ ] Deepa UAT sign-off on scored list quality
- [ ] Rohan executive dashboard populated correctly

### Production Launch (Target: June 30 2026)
- [ ] ServiceNow change request approved
- [ ] Architecture board sign-off
- [ ] Production DAG deployed to MARS_PROD Snowflake account
- [ ] RC Direct holdout group established (suggested: 20% suppression control)

---

## KPI Ownership Matrix

| KPI Layer | Primary Owner | Reviewer | Sign-off |
|-----------|--------------|---------|---------|
| Business outcomes | Priya Sharma (Mars) | Rohan (Mars) | Rohan |
| Model performance | Harshita Gupta (Mu Sigma) | Karthik Nair (Mars) | Harshita + Karthik |
| Pipeline / operational | Karthik Nair (Mars) + Harshita | Rahul Mehta (Mars IT) | Rahul |

---

## Open Items

| ID | Item | Owner | Due |
|----|------|-------|-----|
| KPI-01 | Confirm holdout split % with Priya — suggested 20% suppression control | Priya | Week 1 post-DPA |
| KPI-02 | Mars Finance to validate INR 61,202 avg RC Direct annual spend on full 285K population | Mars Finance via Karthik | Pre-BRD sign-off |
| KPI-03 | Agree Brier score / calibration threshold with Harshita — 0.15 is a Mu Sigma recommendation | Harshita | Pre-model staging |
| KPI-04 | Confirm SFMC segment sync SLA — Monday 9 AM IST achievable given SFMC refresh windows? | Priya + Karthik | Week 2 post-DPA |
