# Issue #3 — Stakeholder Expectations & Conflicts
**Project:** alpha-test | **Client:** Mars India
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Date:** 2026-03-30

---

> **Source:** User-provided answers (Harshita Gupta). Data from mars_churn_sample.csv used as enrichment only.
> Fields marked **[TBD]** require Mars client confirmation.

---

## 👷 Practitioner Doer — Technical Expectations & Constraints

### Technical Constraints Identified

| Constraint | Detail | Status |
|---|---|---|
| Snowflake tier | Data appears pre-aggregated upstream (90d/12m windows already computed). Read/write architecture needs confirmation before pipeline design. | **[TBD — Mars IT]** |
| Python version | Default 3.10+, but MWAA pins Python to its managed runtime version | **[TBD — confirm MWAA version with Mars IT]** |
| Library restrictions | Enterprise CPG clients commonly have InfoSec-approved package lists. Must confirm before building pipeline runtime. | **[TBD — Mars IT/InfoSec]** |
| Data volume at scale | Sample is 80 rows; production likely hundreds of thousands. Snowflake query cost for 90d/12m feature aggregation windows at full scale needs testing. | **Risk — test early in Phase 2** |

### ML Framework Recommendation

| Framework | Role | Rationale |
|---|---|---|
| **LightGBM** | Primary model | Fast on tabular data; handles Bronze-tier class imbalance (60.7% churn); produces calibrated probabilities needed for threshold-based scoring |
| **Logistic Regression** | Baseline | Interpretable; fast to validate against existing rule-based approach; easy for Mars stakeholders to interrogate |
| Deep learning | ❌ Not Phase 1 | No justification given feature count |
| Neural collaborative filtering | ❌ Not Phase 1 | Not applicable for this problem type |
| Uplift/causal models | ❌ Not Phase 1 | No treatment/control data available |

*Mu Sigma recommendation — open to override if Mars has mandate (e.g. existing SageMaker or Azure ML environment with preferred frameworks).*

---

## 🧭 Practitioner Leader — Phase 1 Delivery Expectations

### Top 3 Phase 1 Must-Deliver Items

**1. Agreed churn label definition ratified by Mars**
The dataset shows `churn_label = 1` maps exactly to `days_since_last_purchase >= 90`. This is implicit — not documented or formally agreed. If Mars redefines churn (e.g. 60 days, or subscription cancellation), feature windows, training data, and pipeline logic all require rework. **This is the highest-risk single item in Phase 1.**

**2. Production-ready weekly scoring DAG in Airflow**
Not a Jupyter notebook. Not a script someone runs manually. The current state shows no prior scoring infrastructure in the data (no `prev_risk_flag`, no score column). Phase 1 must deliver a DAG that runs on schedule, produces the output table, and removes the manual/rule-based dependency completely.

**3. Scored output table consumable by Mars CRM team without DS involvement**
The model's value is zero if the retention team can't act on it independently. Delivery = agreed output schema + decision threshold + handoff process. Not just model accuracy metrics.

### Internal Pressures & Risks

| Risk | Detail | Mitigation |
|---|---|---|
| Harshita allocation | Leading alongside parallel commitments — full allocation to Mars not assumed | Clarify allocation % before Phase 2 kicks off |
| Vishwavani/Savikar allocation | Not formally locked — competing projects create resourcing risk during build window | Lock allocation in project planning before architecture phase |
| Hard deadline: 2026-06-30 | ~13 weeks from March 31 for: data access → EDA → feature engineering → model training → pipeline build → UAT → production | **Any delay in Mars DPA or data access directly compresses build time. DPA must be prioritised immediately.** |

---

## 👔 Business User — Mars Expectations

### Top 3 Mars Expectations

**1. Early warning before day 90**
The data shows all churned customers have `email_open_rate ≈ 0` and `whatsapp_engaged = 0` at the point of churn. By day 90 it is already too late. Mars's CRM team needs signals at **30–45 days of declining engagement** — using `email_open_rate_90d`, `whatsapp_engaged_90d`, `loyalty_points_balance`, and `num_campaign_clicks_90d` as early indicators.

**2. Ranked score list directly loadable into campaign tool**
Weekly delivery format: `customer_id | churn_score | risk_tier | recommended_action`. Simple enough for the CRM team to act on without a data scientist in the room. No confusion matrix, no AUC curves — just a ranked, tiered, actionable list.

**3. Bronze-tier prioritisation**
Bronze tier has a 60.7% churn rate (17 of 28 customers in sample). Retention budget is finite. The model must surface highest-probability churners and avoid wasting spend on Gold and Platinum customers who have 0% churn rate.

---

## ⚠️ Conflicts Register

### C-01: Feature Leakage Risk — Subscription Status
**Severity: 🔴 High | Between: Practitioner Doer (DE) vs Practitioner Doer (DS)**

`subscription_active = N` and `days_since_subscription_change` are both present in the data and perfectly correlated with churn in the sample. The DE team will naturally include them as features. The DS team must resolve:

> Is subscription cancellation a **cause** of churn (valid feature) or **synonymous** with churn (leakage)?

If `subscription_active` represents the churn event itself, it cannot be used as a predictor — it would inflate model performance artificially and produce useless scores in production.

**Resolution required:** Feature specification meeting before any model training begins. Default: exclude subscription fields from feature set until Mars confirms the temporal relationship between subscription cancellation and churn.

---

### C-02: Feature Aggregation Ownership
**Severity: 🟡 Medium | Between: DE team (Snowflake SQL) vs DS team (experimentation)**

The 90d and 12m aggregation windows in the dataset are already pre-computed. DE will own this aggregation logic in Snowflake SQL. DS may need different window sizes during experimentation (e.g., 30d, 60d, 180d windows for feature selection). If these are not agreed upfront:
- DE builds the production feature pipeline with one set of windows
- DS trains the model on a different set of windows
- Production scoring diverges from training — model underperforms

**Resolution required:** Feature store ownership agreement before parallel DE/DS work begins. Proposed: DS defines feature spec (including window sizes) → DE implements in Snowflake → DS trains on Snowflake output (not ad-hoc local aggregations).

---

## 🏢 Business Leader — Strategic Expectations

### Single Most Important Outcome
**Reduce RC_Direct churn from 31.4% to below 20% within two scored quarters post-launch.**

RC_Direct customers average ₹61,202/year vs ₹17,719 for PED_Club — 3.4× more valuable per account. Every percentage point of RC_Direct churn reduction delivers disproportionately more ROI than equivalent PED_Club improvement. The pipeline's business case lives or dies on RC_Direct retention.

### Phase 1 — Explicitly NOT Expected

| Excluded Item | Why |
|---|---|
| Real-time scoring | No event stream — all features are batch aggregates |
| Automated model retraining | Requires monitoring infrastructure not in scope yet |
| CRM campaign execution integration | Output table is Phase 1; CRM integration is Phase 2 |
| Recommendation engine (what offer to make) | No historical treatment/response data — cannot train |
| Lifetime value (LTV) modelling | No pricing, margin, or CLV columns in data |
| Churn reason / root-cause analysis | `num_complaints_12m` insufficient as sole qualitative signal |

---

## Open Items from This Issue

| ID | Item | Priority | Owner | Required By |
|---|---|---|---|---|
| OI-E-01 | **Churn label definition formally agreed with Mars** | 🔴 Critical | Harshita + Mars | Before model training |
| OI-E-02 | Feature leakage resolution — subscription fields in/out of feature set | 🔴 Critical | Harshita (DS) + Mars | Feature spec sign-off |
| OI-E-03 | Feature aggregation ownership agreement (DE vs DS) | 🟡 Medium | Harshita | Before parallel work starts |
| OI-E-04 | Snowflake tier + MWAA Python version confirmation | 🟡 Medium | Mars IT | Architecture phase |
| OI-E-05 | InfoSec-approved package list from Mars | 🟡 Medium | Mars IT/InfoSec | Pipeline build phase |
| OI-E-06 | Harshita + Vishwavani/Savikar allocation locked | 🟡 Medium | Harshita | Before project planning |
| OI-E-07 | DPA signed — required before any production data use | 🔴 Critical | Mars Legal | Immediately |

---

*Primary source: User-provided answers — Harshita Gupta, 2026-03-30.
Data from mars_churn_sample.csv used for quantitative enrichment only.*
