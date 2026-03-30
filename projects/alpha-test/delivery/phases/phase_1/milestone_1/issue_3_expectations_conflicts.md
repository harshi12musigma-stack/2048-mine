# Issue #3 — Stakeholder Expectations & Conflicts Consolidation
**Project:** alpha-test — Churn Prediction Pipeline (Airflow + Snowflake + Python)
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Completed:** 2026-03-30

---

## 1. Consolidated Stakeholder Expectations

### 1.1 Business / Executive Stakeholders

| Expectation | Priority | Notes |
|---|---|---|
| Reduce customer churn rate by 10–15% within 6 months of go-live | High | Baseline churn rate to be confirmed in BRD |
| Weekly batch scoring — predictions refreshed every Monday morning | High | Drives retention campaign cadence |
| Churn risk score per customer available in Snowflake for downstream dashboards | High | Consumption layer requirement |
| Intervention recommendations alongside churn scores (not just scores) | Medium | Adds complexity — phase 2 consideration |
| Executive dashboard showing churn trend, model accuracy, and campaign ROI | Medium | Separate SDLC track or downstream deliverable |
| Model explainability — "why is this customer flagged?" | Medium | SHAP values or feature contribution |
| < 10% false positive rate on high-risk segment | High | Reduces wasted retention spend |

### 1.2 Data & Analytics Stakeholders

| Expectation | Priority | Notes |
|---|---|---|
| Full Bronze → Silver → Gold medallion architecture in Snowflake | High | Aligns with existing Mu Sigma DE standards |
| Airflow DAGs modular and reusable across pipeline stages | High | Maintainability requirement |
| ML model trained on ≥ 12 months of historical data | High | Seasonal signal capture |
| Model retraining pipeline automated (monthly cadence) | Medium | Drift detection trigger preferred over fixed cadence |
| Feature store pattern — reusable features for future models | Medium | Nice-to-have for Phase 1; mandatory by Phase 2 |
| Unit tests and data quality checks embedded in pipeline | High | Non-negotiable for production |
| Model versioning and experiment tracking (MLflow or equivalent) | Medium | To be confirmed against available tooling |

### 1.3 IT / Infosec Stakeholders

| Expectation | Priority | Notes |
|---|---|---|
| All PII fields masked or tokenized before ML pipeline ingestion | High | Compliance non-negotiable |
| Airflow connections to Snowflake via key-pair authentication only | High | OAuth not supported in current env |
| All code in version-controlled repo before deployment to production | High | Git-gate mandatory |
| No direct writes to production Snowflake from development environments | High | Environment segregation |
| Pipeline SLA: scoring output available by 06:00 UTC every Monday | High | Business rhythm dependency |
| Audit log of all model prediction writes to Snowflake | Medium | Regulatory readiness |

---

## 2. Conflict Register

| # | Conflict Description | Parties Involved | Severity | Resolution Approach |
|---|---|---|---|---|
| C-01 | Business wants **daily** scoring; IT says Snowflake compute budget supports **weekly** only | Business BU Heads vs IT/Infosec | High | Escalate to BSH sponsor; propose tiered approach (weekly default, daily for top 5% risk segment only) |
| C-02 | Analytics team wants **MLflow** for experiment tracking; IT says only **approved tools** list applies and MLflow is not yet cleared | Data/Analytics TDS3 vs IT | Medium | IT to assess MLflow for approval; fallback = custom metadata table in Snowflake |
| C-03 | Business requests **intervention recommendations** in Phase 1; Analytics team says this is Phase 2 scope (adds 3–4 weeks) | Business Sponsors vs Analytics | Medium | Descope recommendations to Phase 2; confirm with BSH sponsor in BRD sign-off |
| C-04 | Business expects churn scores for **all customers**; Data lead says 30% of customers have insufficient history for reliable scoring | Business vs Data | High | Define minimum history threshold (e.g., ≥ 3 months); flag thin-data customers as "Insufficient History" rather than scored |
| C-05 | IT requires **change management approval** for each DAG deployment; Analytics wants **CI/CD automated deploy** | IT vs Analytics | Medium | Agree on lightweight CAB process for pipeline changes (auto-approve for non-schema-altering changes) |
| C-06 | Business wants model explainability (SHAP) on every scored customer; Analytics says SHAP at scale adds significant compute cost | Business vs Analytics | Low | SHAP for top 20% risk band only; summary stats for rest |

---

## 3. Alignment Decisions (Pre-BRD)

The following decisions are recommended to be ratified at the BRD kick-off meeting:

| Decision | Recommended Resolution | Owner |
|---|---|---|
| Scoring frequency | **Weekly batch** (Monday 06:00 UTC) with daily micro-refresh for top 5% risk segment | BSH Sponsor |
| MLflow approval | **30-day IT assessment** — Snowflake metadata fallback in the interim | IT Lead |
| Intervention recommendations | **Descoped to Phase 2** — confirmed in BRD | Project BSH |
| Minimum scoring eligibility | **≥ 90 days of customer history** required; others flagged as `INSUFFICIENT_DATA` | Analytics Lead |
| Deployment process | **Lightweight CAB** — auto-approve non-breaking pipeline changes; full CAB for schema changes | IT + Analytics |
| SHAP coverage | **Top 20% risk segment** only — full SHAP; bottom 80% — feature weight summary | Analytics Lead |

---

## 4. Open Items for BRD Sign-off

- [ ] Confirm baseline churn rate (current state) from Business team
- [ ] Confirm Snowflake compute tier and monthly cost ceiling from IT
- [ ] Get formal decision on MLflow vs custom tracking from IT
- [ ] Ratify scoring frequency decision with BSH sponsor
- [ ] Define "customer" entity — is it account-level, contact-level, or subscription-level?

---

_Deliverable generated as part of Phase 1 / Milestone 1 — BRD execution loop._
