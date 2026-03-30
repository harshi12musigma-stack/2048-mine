# Issue #10 — Assumptions & Risk Register (with Mitigation)
**Project:** alpha-test — Churn Prediction Pipeline (Airflow + Snowflake + Python)
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Completed:** 2026-03-30

---

## 1. Assumptions Register

### 1.1 Data Assumptions

| # | Assumption | Impact if Wrong | Owner | Validation Method |
|---|---|---|---|---|
| A-01 | A stable `customer_id` field exists across CRM, Billing DB, and S3 event logs that can be used as the join key | High — feature engineering and scoring break | TDS3 | Data profiling in Phase 5 |
| A-02 | ≥ 12 months of historical customer transaction data is available in source systems | High — model training dataset insufficient | TDS3 | Data discovery in Phase 5 |
| A-03 | Actual churn outcomes (churned / not churned) are captured in CRM and can be extracted as labeled training data | High — supervised ML requires outcome labels | TDS3 | CRM data audit |
| A-04 | Source systems do not change schema more than once per quarter | Medium — pipeline breaks on schema drift | IT | Schema versioning agreement with IT |
| A-05 | PII masking rules are defined and approved by compliance team before pipeline build | High — blocked from ingestion without this | IT / Compliance | Compliance sign-off in Phase 1 |
| A-06 | At least 70% of active customers have ≥ 90 days of activity history | Medium — scoring coverage falls below target | TDS3 | Customer history analysis in Phase 5 |

### 1.2 Infrastructure Assumptions

| # | Assumption | Impact if Wrong | Owner | Validation Method |
|---|---|---|---|---|
| A-07 | Airflow is deployed and accessible; DAG development environment is ready | High — pipeline development blocked | IT | IT confirmation in Week 1 |
| A-08 | Snowflake warehouse is provisioned for dev, staging, and production environments | High — no place to write data | IT | IT confirmation in Week 1 |
| A-09 | Snowflake compute credits are sufficient for daily ingestion + weekly scoring | Medium — performance degradation or cost overrun | IT / Finance | Compute cost estimation in Phase 6 |
| A-10 | Network connectivity between Airflow and Snowflake / source systems is already established | High — DAG execution blocked | IT | Connectivity test in Week 1 |

### 1.3 Business Assumptions

| # | Assumption | Impact if Wrong | Owner | Validation Method |
|---|---|---|---|---|
| A-11 | Retention team will consume churn scores from Snowflake and act on them weekly | High — pipeline builds value but creates no outcome | Retention Manager | Retention team commitment in BRD |
| A-12 | Weekly scoring cadence is sufficient — no requirement for real-time scoring in Phase 1 | Medium — rescoping needed | BSH Sponsor | BRD sign-off |
| A-13 | A single definition of "churn" is agreed upon and stable (not redefined during the project) | High — feature engineering must be redone | BSH Sponsor | Churn definition locked at BRD |
| A-14 | "Customer" entity is defined at account level (not contact or subscription level) | High — entire data model restructures | BSH Sponsor | Entity definition locked at BRD |

---

## 2. Risk Register

### 2.1 Data Risks

| # | Risk | Likelihood | Impact | Risk Score | Mitigation | Contingency |
|---|---|---|---|---|---|---|
| R-01 | No stable customer_id across systems — join failures | Medium | Critical | 🔴 High | Data profiling in Phase 5 before build begins | Implement deterministic matching (name + email + phone) |
| R-02 | Insufficient labeled churn history (< 6 months outcomes) | Low | Critical | 🔴 High | Validate churn outcome availability in Phase 5 | Use proxy label (inactive 60+ days = churned) — document trade-off |
| R-03 | PII classification not completed before pipeline build | Medium | High | 🔴 High | Compliance review in Week 1–2 | Tokenize all customer fields in Bronze; de-tokenize only in secure Gold views |
| R-04 | Source schema changes during build (CRM API update) | Medium | Medium | 🟡 Medium | Schema versioning + Airflow schema-check tasks | Bronze layer absorbs raw — Silver logic updated independently |

### 2.2 Model Risks

| # | Risk | Likelihood | Impact | Risk Score | Mitigation | Contingency |
|---|---|---|---|---|---|---|
| R-05 | Model AUC < 0.80 on first train — business rejects pipeline | Low | High | 🟡 Medium | Feature engineering from domain knowledge + iterative tuning | Deploy rule-based model as fallback; report ML model as "V2 candidate" |
| R-06 | Churn definition changes after model training begins | High | Critical | 🔴 High | Lock definition at BRD sign-off; change control for definition updates | Re-train with new definition — 3-week delay budget reserved |
| R-07 | Model accuracy degrades significantly after 3 months (concept drift) | Medium | High | 🟡 Medium | Monthly automated retraining + drift monitoring | Manual retraining trigger with TDS3 notification |

### 2.3 Infrastructure Risks

| # | Risk | Likelihood | Impact | Risk Score | Mitigation | Contingency |
|---|---|---|---|---|---|---|
| R-08 | Snowflake compute cost overruns due to daily ingestion volume | Low | Medium | 🟢 Low | Cost monitoring alerts; warehouse auto-suspend | Move to incremental micro-batch instead of full scan |
| R-09 | Airflow environment instability (worker failures, memory limits) | Medium | High | 🟡 Medium | Staging environment testing before production; DAG retry logic | Manual trigger + email alert to TDS3 on-call |
| R-10 | CRM API rate limits block timely daily ingestion | Medium | Medium | 🟡 Medium | Incremental API pulls with exponential backoff | Off-peak scheduling (02:00–04:00 UTC window) |

### 2.4 Operational Risks

| # | Risk | Likelihood | Impact | Risk Score | Mitigation | Contingency |
|---|---|---|---|---|---|---|
| R-11 | Retention team does not adopt Snowflake-based score consumption | Medium | High | 🟡 Medium | Early retention team training; simple SQL views / BI layer | Build lightweight CSV export from Snowflake for transitional use |
| R-12 | BSH sponsor changes mid-project — BRD decisions overturned | Low | High | 🟡 Medium | BRD sign-off documented; decisions locked with named sponsors | Re-run BRD alignment with new sponsor; 2-week delay budget |
| R-13 | Key TDS3/TDS4 resource exits project mid-build | Low | High | 🟡 Medium | Documentation-first approach; pipeline as code; no single points of knowledge | Cross-train TDS2 on pipeline architecture |

---

## 3. Risk Heat Map Summary

| Risk Level | Count | Risks |
|---|---|---|
| 🔴 Critical/High | 4 | R-01, R-02, R-03, R-06 |
| 🟡 Medium | 7 | R-04, R-05, R-07, R-09, R-10, R-11, R-12, R-13 |
| 🟢 Low | 1 | R-08 |

**Top 3 risks to monitor at every milestone gate:**
1. **R-06** — Churn definition ambiguity (lock at BRD or all downstream work is at risk)
2. **R-01** — No stable customer_id (validate in Phase 5 before any feature engineering begins)
3. **R-03** — PII not classified (cannot ingest production data without this)

---

_Deliverable generated as part of Phase 1 / Milestone 1 — BRD execution loop._
