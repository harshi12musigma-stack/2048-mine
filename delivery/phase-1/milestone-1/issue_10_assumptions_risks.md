# Issue #10 — Assumptions & Risks with Mitigation Approach
**Project:** Mars India Churn Prediction Pipeline  
**Phase:** 1 — Requirement Gathering  
**Milestone:** 1 — Business Requirement Document  
**Date:** 2026-04-02  
**Status:** Complete  

---

## Overview

All assumptions and risks are grounded in confirmed stakeholder inputs from W-01/W-02/W-03 sessions and the Mars data sample. Risk ratings are: **Critical (C)**, **High (H)**, **Medium (M)**, **Low (L)**.

---

## Assumptions

### Data & Modeling

| ID | Assumption | Source | Validation Required |
|----|-----------|--------|-------------------|
| A-01 | Churn label is defined as `days_since_last_purchase ≥ 90` | Implicit in sample + Priya W-01 | **Must be formally ratified** — cross-functional alignment required (CRM + Finance + Analytics) |
| A-02 | Training window: 2022–present. Pre-2022 data has migration gaps and will not be used | Karthik W-02 | Low — Karthik confirmed |
| A-03 | `subscription_active` is NOT used as a model feature (leakage risk) | Karthik W-02 — prior XGBoost failure | **Must be confirmed** before feature engineering begins |
| A-04 | Feature aggregations (90d/12m windows) will be sourced from existing dbt mart, refactored to incremental | Karthik W-02 | Medium — incremental refactor complexity TBD |
| A-05 | `vet_referred` will be available as a feature post InfoSec classification ruling | Rahul W-03 | Awaiting InfoSec ruling — ~2 weeks |
| A-06 | Identity bridge (`CUST_MASTER.ID_BRIDGE`) covers ~68% RC Direct, ~55% PED Club — low-coverage customers will have bridge flag in output | Karthik W-02 | Low — Karthik confirmed coverage stats |
| A-07 | LightGBM primary model; Logistic Regression as baseline. No deep learning in Phase 1 | Mu Sigma recommendation | Confirmed |
| A-08 | WhatsApp engagement signal (`whatsapp_engaged_90d`) is directionally correct at ~90% webhook reliability | Karthik W-02 | Acceptable for model training — flagged in feature notes |

### Infrastructure

| ID | Assumption | Source | Validation Required |
|----|-----------|--------|-------------------|
| A-09 | MWAA Python 3.11 with LightGBM package approved within 3–5 days of submission | Rahul W-03 | **Submit package request immediately post-DPA** |
| A-10 | Snowflake X-Small warehouse sufficient for dev/staging; scale-up request for production | Rahul W-03 | Validate after first full-population inference run |
| A-11 | Fivetran nightly runs complete by 2 AM; Sunday night scoring will use transactions up to Friday (1–2 day lag) | Karthik W-02 | Acceptable — confirmed |
| A-12 | Service account provisioning SLA: 2 weeks from request | Rahul W-03 | **Flag to Rahul immediately post-DPA** |
| A-13 | DPA will be signed by early May 2026 (~3–4 weeks from March 31) | Rahul W-03 | **Track weekly — single biggest schedule risk** |

### Project & Delivery

| ID | Assumption | Source | Validation Required |
|----|-----------|--------|-------------------|
| A-14 | Code-complete deadline: June 1, 2026. Production deployment: June 30, 2026 | Rahul W-03 + Harshita | Confirmed — non-negotiable |
| A-15 | Harshita Gupta is project lead (BSH). Vishwavani and Savikar are DS/DE delivery resources | Harshita confirmed | Allocation % not formally locked — resourcing risk |
| A-16 | No Mu Sigma infrastructure used — all processing within Mars Snowflake (AWS ap-south-1) | Rahul W-03 | Confirmed — DPA sub-processor clause resolved |
| A-17 | Freshdesk production ingestion pipeline will be built as part of Phase 1 DE scope | Karthik W-02 | **Confirm with Karthik — not yet explicitly scoped** |

---

## Risk Register

### Critical Risks

| ID | Risk | Probability | Impact | Mitigation |
|----|------|------------|--------|-----------|
| R-01 | **DPA delayed beyond early May** — all data access blocked; no pipeline code against live data until signed | High | Critical | Track weekly. Begin all design/spec work in April using sample data. Escalate to Harshita + Zubin if delay >2 weeks past expected signing. |
| R-02 | **Churn label not formally agreed** — model trained on wrong definition; entire retraining required | Medium | Critical | BRD sign-off must include explicit churn definition ratification. Block model training until label agreed by Priya + Finance + Karthik. |
| R-03 | **Feature leakage (subscription_active)** — prior XGBoost failed at production (15% precision at decile) due to this | High | Critical | Hard gate: subscription_active excluded from feature set unless InfoSec confirms it is a leading indicator (not the churn event). Decision required pre-EDA. |

### High Risks

| ID | Risk | Probability | Impact | Mitigation |
|----|------|------------|--------|-----------|
| R-04 | **June 30 deadline missed** — 13-week window with 4-week DPA gap leaves ~9 weeks for EDA, modelling, pipeline, UAT, deployment | High | High | Parallel-track: April = design + spec + synthetic data EDA; May = live data access + feature engineering; June 1 = code-complete hard stop. |
| R-05 | **vet_referred excluded by InfoSec** — removes the strongest RC Direct signal (100% of 11 churned RC Direct in sample had vet_referred=Y) | Medium | High | Design feature set without vet_referred as primary; treat as enhancement. LightGBM with remaining features should still exceed 40% precision@decile. |
| R-06 | **Harshita / Vishwavani / Savikar allocation conflicts** — team not formally ring-fenced | Medium | High | Escalate allocation confirmation to Srini and Zubin. Get written allocation before DPA signed. |
| R-07 | **Capillary SFTP format change** — occurred twice in last year; would break loyalty feature ingestion | Medium | High | DQ DAG includes Capillary schema validation task. Alert fires within 15 min of failure. Manual override: last-known-good loyalty data from previous week. |
| R-08 | **Snowflake dbt full-refresh ~90 min** — current model; Sunday night scoring window may be tight | Medium | High | Phase 1 DE scope: refactor dbt mart to incremental model. Target: <15 min refresh. Validate before UAT. |

### Medium Risks

| ID | Risk | Probability | Impact | Mitigation |
|----|------|------------|--------|-----------|
| R-09 | **Identity bridge gap (~32% RC Direct uncovered)** — cross-system features unavailable for these customers | High | Medium | Flag in model output: `id_bridge_matched` column. Treat as a feature (missingness may correlate with churn). Monitor % of High-risk tier without bridge match. |
| R-10 | **LightGBM package request delayed or rejected** — MWAA InfoSec approval required | Low | Medium | Contingency: serialize LightGBM model to ONNX + use existing scikit-learn ONNX runtime (already installed). Or use sklearn GradientBoostingClassifier as fallback. |
| R-11 | **Bronze-tier label contamination** — 60.7% churn rate in sample may reflect sampling bias, not real population rate | Medium | Medium | Full-population EDA post-DPA to validate Bronze churn rate. If contaminated, re-stratify training set. |
| R-12 | **Email open rate data leakage** — all churned customers in sample have open_rate ~0; may be post-churn measurement | Medium | Medium | EDA: validate that open_rate measurement precedes churn label by ≥30 days. If post-churn, exclude or use lagged feature. |
| R-13 | **SFMC integration scope creep** — automated Snowflake→SFMC sync may expand Phase 1 unexpectedly | Medium | Medium | Define Phase 1 output as: scored Snowflake table + CSV export. SFMC automated sync is Phase 1 stretch goal, not baseline. Priya to confirm. |

### Low Risks

| ID | Risk | Probability | Impact | Mitigation |
|----|------|------------|--------|-----------|
| R-14 | **Production freeze Oct 1–Nov 15** — no production changes. June 30 launch avoids this window | Low | Low | Launch by June 30 confirmed. No action needed unless timeline slips. |
| R-15 | **Freshdesk production ingestion pipeline** — num_complaints_12m was manual export in sample | Low | Low | Scope Freshdesk ingestion as Phase 1 DE task. Low complexity — REST API. Allocate 3–5 days. |
| R-16 | **Ghost purchasers — low-frequency customers always appear at-risk** | Low | Medium | LightGBM features include purchase_count_12m and purchase_count_90d ratio — frequency trend should separate ghost purchasers from true at-risk. Validate in EDA. |

---

## Risk Summary

| Severity | Count | Top Priority |
|---------|-------|-------------|
| Critical | 3 | R-01 (DPA), R-02 (Churn label), R-03 (Feature leakage) |
| High | 5 | R-04 (Deadline), R-05 (vet_referred), R-06 (Team allocation) |
| Medium | 5 | R-09 to R-13 |
| Low | 3 | R-14 to R-16 |

**Immediate actions required (before DPA signed):**
1. Get churn label definition formally agreed — document in BRD (R-02)
2. Resolve subscription_active leakage decision — document in feature spec (R-03)
3. Confirm team allocation with Srini/Zubin (R-06)
4. Submit LightGBM package request to Rahul's team — queue it ready for DPA signing (R-10)
5. Begin April design-only work using sample data (R-01 mitigation)
