# Issue #9 — Business KPIs & Success Criteria
**Project:** alpha-test — Churn Prediction Pipeline (Airflow + Snowflake + Python)
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Completed:** 2026-03-30

---

## 1. KPI Framework

KPIs organized across three layers: **Business Outcomes → Pipeline Performance → Model Quality**

---

## Layer 1: Business Outcome KPIs (Owned by BSH Sponsors / BU Heads)

| KPI | Definition | Baseline (Current) | Target (6 months post-launch) | Measurement Cadence |
|---|---|---|---|---|
| Monthly Churn Rate | % of active customers who churn in a given month | ~2.5% (to confirm) | ≤ 2.25% (10% reduction) | Monthly |
| Revenue Lost to Churn | Monthly revenue attributable to churned customers | ~$52,000/month (benchmark) | ≤ $46,800/month | Monthly |
| Retention Campaign Conversion Rate | % of at-risk customers successfully retained after outreach | To be baselined | ≥ 25% (from ~15% today) | Per campaign cycle |
| Wasted Retention Spend | Cost of outreach to customers who were not actually at risk | ~$18,000/month | ≤ $4,500/month (75% reduction) | Monthly |
| Customer Lifetime Value (avg) | Avg revenue per retained customer over 12 months | To be baselined | Increase by ≥ 5% (due to retention focus) | Quarterly |

---

## Layer 2: Pipeline Performance KPIs (Owned by TDS4 / TDS3)

| KPI | Definition | Target | Measurement Cadence |
|---|---|---|---|
| DAG Success Rate | % of Airflow DAG runs completing without failure | ≥ 99% | Weekly |
| Scoring SLA Compliance | % of weeks where churn scores available in Gold by Monday 06:00 UTC | ≥ 98% | Weekly |
| Data Freshness | Max lag between source system event and Snowflake Bronze availability | ≤ 24 hours | Daily |
| Pipeline Latency (end-to-end) | Time from DAG trigger to Gold layer availability | ≤ 4 hours | Per run |
| Data Quality Pass Rate | % of Bronze → Silver validation checks passing | ≥ 97% | Daily |
| Eligible Customer Coverage | % of active customers with ≥ 90 days history that receive a churn score | ≥ 95% | Weekly |
| Analyst Time on Pipeline | Hours per month spent on manual pipeline intervention / oversight | ≤ 5 hours/month | Monthly |

---

## Layer 3: Model Quality KPIs (Owned by TDS3 / Analytics Lead)

| KPI | Definition | Target | Measurement Cadence |
|---|---|---|---|
| AUC-ROC | Area under ROC curve on held-out test set | ≥ 0.80 | At each model version |
| Precision (High Risk Tier) | % of High Risk customers who actually churn | ≥ 65% | At each model version |
| Recall (High Risk Tier) | % of actual churners captured in High Risk tier | ≥ 75% | At each model version |
| False Positive Rate | % of non-churners incorrectly flagged as High Risk | ≤ 10% | Weekly scoring run |
| F1 Score | Harmonic mean of precision and recall | ≥ 0.70 | At each model version |
| Model Drift Index | KL-divergence between current and baseline score distributions | ≤ 0.10 | Weekly |
| Retraining Trigger Rate | Frequency of automatic retraining triggers (drift-based) | ≤ 2×/month (indicates stability) | Monthly |
| SHAP Coverage | % of High Risk customers with top-3 feature explanations available | 100% | Weekly |

---

## 2. KPI Dashboard Requirements (Phase 2 — For Reference)

The following KPI views should be available post-pipeline launch:

| Dashboard | Audience | Refresh | Key Metrics |
|---|---|---|---|
| Churn Overview | BU Head / BSH | Weekly | Monthly churn rate, revenue at risk, high-risk customer count |
| Retention Performance | Retention Manager | Weekly | Campaign conversion rate, outreach coverage, false positive rate |
| Pipeline Health | TDS3 / TDS4 | Daily | DAG success rate, data freshness, scoring SLA compliance |
| Model Performance | Analytics Lead | Monthly | AUC, precision, recall, F1, drift index |

---

## 3. Baseline Measurement Plan

The following baselines must be confirmed and documented before pipeline go-live to enable post-launch comparison:

| Metric | Baseline Source | Owner | Target Confirmation Date |
|---|---|---|---|
| Monthly churn rate (last 6 months avg) | CRM historical export | TDS3 Analyst | Week 2 of BRD |
| Average revenue per customer | Billing DB | Finance / BU Head | Week 2 of BRD |
| Current retention campaign conversion rate | CRM outreach history | Retention Manager | Week 2 of BRD |
| Monthly false positive count (current rules) | Manual scoring audit | TDS2 Analyst | Week 2 of BRD |
| Analyst hours/month on churn work | Self-reported time log | TDS2/TDS3 | Week 1 of BRD |

---

## 4. Success Gate Summary (Go/No-Go at Each Phase)

| Gate | Phase | Criteria | Decision Maker |
|---|---|---|---|
| G-01 | After Phase 1 (BRD) | All stakeholders signed off; baseline metrics captured; conflicts resolved | BSH Sponsor |
| G-02 | After Phase 5 (Data Profiling) | ≥ 80% of required features available with acceptable quality | Analytics Lead |
| G-03 | After Phase 9 (Pipeline Dev) | DAG success rate ≥ 99% in staging; end-to-end latency ≤ 4 hrs | TDS4 |
| G-04 | After Phase 10 (QA) | Model AUC ≥ 0.80, FPR ≤ 10% on held-out test set | Analytics Lead + BSH |
| G-05 | Production Go-Live | All validation checks pass; IT sign-off; runbook ready | TDS4 + IT Lead |

---

_Deliverable generated as part of Phase 1 / Milestone 1 — BRD execution loop._
