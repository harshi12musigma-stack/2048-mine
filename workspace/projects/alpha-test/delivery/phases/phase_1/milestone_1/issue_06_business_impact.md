# Issue #6 — Quantify Business Impact (Time, Cost, Risk)
**Project:** Mars Churn Prediction Pipeline
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** 1 — Business Requirement Document (BRD)
**Status:** Complete
**Date:** 2026-03-31
**Source:** W-01/W-02/W-03 confirmed session data + Harshita Gupta (2026-03-31)

---

## 1. Revenue at Risk

| Metric | RC Direct | Pedigree Club |
|---|---|---|
| Active customers | 285,000 | 1,100,000 |
| Scoreable population | 170,000 | 650,000 |
| Current churn rate | 31.4% | ~18.5% |
| Avg annual spend (active) | INR 61,202 | INR 17,719 |
| Gross revenue at risk (churning cohort) | **~INR 548 Cr/year** | ~INR 361 Cr/year |

> **RC Direct is the business priority.** 3.4× higher revenue per customer, higher churn rate, and a direct vet-referral signal unique to this segment.

---

## 2. Revenue Impact of 10pp Churn Reduction (RC Direct)

| Input | Value | Source |
|---|---|---|
| RC Direct active population | 285,000 | Karthik, W-02 |
| Avg 12-month spend per active customer | INR 61,202 | Derived from mars_churn_sample.csv |
| Customers saved at 10pp improvement | ~28,500 | 285K × 10% |
| Gross revenue protected | **~INR 174.9 Cr/year** | 28,500 × INR 61,202 |
| Phase 1 build cost | INR 20.7L | See Section 4 |
| Year 1 gross ROI | **~845×** | Revenue protected / Build cost |
| Conservative ROI (30% model attribution) | **~250×** | INR 52.3 Cr / INR 20.7L |

> **⚠️ Validation required:** INR 61,202 avg spend is derived from the 80-row sample. Mars Finance to confirm on full 285K population. ROI uses gross revenue saved (not net margin), and assumes 100% model attribution before holdout correction.

---

## 3. Time & Effort Impact

### Current Manual Effort (As-Is)

| Activity | Who | Frequency | Effort |
|---|---|---|---|
| At-risk list processing (filter, dedupe, prioritise) | Deepa (Coordinator) | Weekly, every Monday | ~2 hrs/week (~104 hrs/year) |
| Redash SQL query scheduling | Karthik team | Weekly, cron job | ~30 min maintenance/week |
| Campaign tool reconciliation (SFMC + Kaleyra) | Deepa | Weekly | ~45 min/week |
| Executive dashboard update for Rohan | Manual | Weekly | ~30 min/week |
| **Total manual effort (as-is)** | | | **~3.75 hrs/week (~195 hrs/year)** |

### Future State (To-Be)
- Weekly Airflow DAG runs Sunday night — zero manual trigger
- Scored output lands in Snowflake + Salesforce automatically
- Deepa effort: UAT review only (not weekly list building)
- Estimated manual effort reduction: **~85%** (~165 hrs/year recovered)

---

## 4. Build Cost — Phase 1

| Item | Cost (INR) | Assumptions |
|---|---|---|
| Mu Sigma team (DS + DE + 50% PL, 12 weeks) | 18,30,000 | Standard India consulting rates |
| Snowflake + MWAA infra (3 months, X-Small) | 2,40,000 | Snowflake Enterprise X-Small + MWAA small env |
| **Total Phase 1** | **20,70,000 (~INR 20.7L)** | |

> **⚠️ Pending confirmation:** Mu Sigma Finance to validate team billing rates. Rahul/IT to confirm actual Snowflake running cost from AWS bill.

---

## 5. Cost per Retention Outreach Attempt

| Channel | Est. Cost per Contact | Basis |
|---|---|---|
| Outbound call | INR 11.80 | 7 agents × INR 42K/mo ÷ 250K calls/year |
| Email (SFMC) | INR 0.50 | Standard SFMC per-email India rate |
| WhatsApp (Kaleyra WABA) | INR 0.80 | Business messaging rate |
| **RC Direct full sequence** | **~INR 13.10/customer/cycle** | WhatsApp → Email → Call |

> **⚠️ Pending validation:** Mars Finance to confirm Kaleyra contract rate and fully-loaded call centre cost (benefits, infrastructure, management overhead).

**Annual outreach cost (current, RC Direct):**
- Normal at-risk: 1,200–1,500/week × INR 13.10 × 52 weeks = **~INR 81.5L–INR 1.02 Cr/year**
- Peak weeks (up to 3,000/week): additional surge spend

---

## 6. Risk Quantification

| Risk | Type | Quantified Impact |
|---|---|---|
| **Deepa SPOF** | Operational | 1 person holds all filter logic. Any absence = no campaign that week = ~1,200–3,000 at-risk customers untouched |
| **Capillary data fragility** | Data | Format changed 2× in last year. Each incident = data gap in campaign targeting. Unquantified but recurring. |
| **No holdout group** | Measurement | Reported retention rates (22% RC Direct, 14% PED Club) are inflated. True baseline unknown — program ROI is unverifiable today. |
| **DPA delay (4 weeks)** | Timeline | Every week of DPA delay = 1 more week of avoidable churn during design-only period. ~1,200 RC Direct customers/week enter at-risk zone untouched by model. |
| **vet_referred InfoSec ruling** | Model quality | If field is excluded: removes a key RC Direct signal (all 11 churned RC Direct in sample had vet_referred=Y). Model precision on RC Direct segment may decline significantly. |
| **churn label misalignment** | Model validity | CRM: 60d, Finance: 120d, current system: 45d/90d. If label definition shifts post-training, model must be retrained from scratch. |
| **subscription_active leakage** | Model validity | Prior XGBoost (Manish, 2025) achieved 90% test accuracy / 15% production precision because of leakage. Same risk exists if not resolved. |
| **Code-complete deadline** | Timeline | Must reach code-complete by ~June 1 to allow 2–4 week prod deployment cycle before June 30 deadline. 12 weeks from April 1 = June 23. No buffer. |

---

## 7. Summary Business Case

| Dimension | Value |
|---|---|
| Gross revenue at risk (RC Direct, current churn) | ~INR 548 Cr/year |
| Revenue protected at 10pp improvement (RC Direct) | ~INR 174.9 Cr/year |
| Phase 1 build cost | ~INR 20.7L |
| Year 1 gross ROI (RC Direct only) | ~845× |
| Manual effort recovered per year | ~165 hrs |
| Primary risk if not actioned | Deepa SPOF + Capillary fragility = campaign halts; churn continues at 31.4% |

---

## 8. Open Items

| # | Item | Owner | Target |
|---|---|---|---|
| OI-6-01 | Validate INR 61,202 avg spend on full 285K population | Mars Finance | Before BRD sign-off |
| OI-6-02 | Confirm Kaleyra contract rate + fully-loaded call centre cost | Mars Finance | Before BRD sign-off |
| OI-6-03 | Confirm Mu Sigma billing rates for Phase 1 estimate | Mu Sigma Finance | Before BRD sign-off |
| OI-6-04 | Confirm Snowflake actual running cost from AWS bill | Rahul Mehta | Before BRD sign-off |
| OI-6-05 | Confirm Mars board-approved churn reduction target | Rohan / Priya | Before BRD sign-off |
