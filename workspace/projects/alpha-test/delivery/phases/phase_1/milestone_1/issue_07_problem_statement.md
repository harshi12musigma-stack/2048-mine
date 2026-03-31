# Issue #7 — Problem Statement Aligned with Business Objectives
**Project:** Mars Churn Prediction Pipeline
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** 1 — Business Requirement Document (BRD)
**Status:** Complete
**Date:** 2026-03-31
**Source:** Synthesised from Issues #1–6 confirmed session data (W-01/W-02/W-03)

---

## 1. The Problem — One Sentence

> **Mars India's RC Direct and Pedigree Club retention program operates on a fragile, rule-based recency trigger that fires too late, is dependent on a single person, and has no early warning signal — causing ~INR 548 Cr/year in avoidable revenue loss at current churn rates.**

---

## 2. Business Context

Mars India runs a pet food subscription and loyalty program across two customer segments:

| Segment | Active Customers | Churn Rate | Avg Annual Spend | Annual Revenue at Risk |
|---|---|---|---|---|
| RC Direct | 285,000 | **31.4%** | INR 61,202 | **~INR 548 Cr** |
| Pedigree Club | 1,100,000 | ~18.5% | INR 17,719 | ~INR 361 Cr |

RC Direct is the **priority segment** — 3.4× higher revenue per customer, higher churn rate, and a distinct vet-referral purchase pattern that creates a unique early-warning signal opportunity.

---

## 3. What Triggered This Project

Two converging failures made this a priority in early 2026:

1. **Capillary SFTP pipeline broke** — the at-risk list was not generated for a full week. ~1,200–1,500 at-risk customers received no outreach. The fragility of a cron-based SQL query as the backbone of a INR 500+ Cr retention program became unacceptable.

2. **Deepa SPOF exposed** — Deepa (Retention Coordinator) is the only person who knows the full list-filtering logic. The logic is undocumented. Any absence halts the entire weekly retention execution for 285K customers.

---

## 4. Where the Current Process Breaks Down

| Failure Mode | Impact |
|---|---|
| **Rule fires at day 45** — customer has already been silent for 6+ weeks | First outreach is reactive, not predictive. No signal at days 20–30 when engagement is declining. |
| **Ghost purchasers never trigger** | Low-frequency customers with natural long purchase cycles are always flagged as at-risk, burning outreach budget |
| **Channel-switch false positives** | Customers buying via Amazon or vet clinic appear as churning — no cross-channel identity reconciliation (~32% of RC Direct have incomplete identity bridge) |
| **Two disconnected campaign tools** | SFMC (email) and Kaleyra (WhatsApp) are not integrated — Deepa manually reconciles weekly (~2 hrs) |
| **No holdout group** | Reported retention rate of 22% RC Direct / 14% PED Club is inflated — no control group, no true program ROI measurement |
| **No individual engagement trending** | Only aggregate open rates — no per-customer signal of deteriorating engagement |

---

## 5. The Technical Problem

> **Binary classification at individual customer level:** predict which customers will churn (lapse > 90 days) within the next 30 days, using 90-day and 12-month behavioural aggregates — scored weekly in batch, output to Snowflake and Salesforce.

### Predictive Signals (Confirmed)

| Feature | Signal Type | Status |
|---|---|---|
| `days_since_last_purchase` | Primary recency | ✅ Use — but as model feature, not rule trigger |
| `email_open_rate_90d` | Engagement decay | ✅ Use |
| `whatsapp_engaged_90d` | Engagement decay | ✅ Use (directional — ~90% webhook reliability) |
| `loyalty_points_balance` | Brand attachment | ✅ Use |
| `purchase_count_90d` vs `purchase_count_12m` | Frequency deceleration | ✅ Use (derived feature) |
| `loyalty_tier` | Segment risk proxy | ✅ Use (Bronze = 60.7% churn rate) |
| `vet_referred` | RC Direct key signal | ⚠️ Conditional — pending InfoSec classification ruling (2–3 weeks) |
| `subscription_active` | **LEAKAGE RISK** | ❌ Exclude until resolved — may be synonymous with churn event, not a predictor |
| `days_since_subscription_change` | **LEAKAGE RISK** | ❌ Exclude — correlated with churn event |

### Model Approach

| Layer | Choice | Rationale |
|---|---|---|
| Primary model | LightGBM | Handles Bronze-tier class imbalance (60.7%), calibrated probabilities for threshold scoring |
| Baseline | Logistic Regression | Interpretable, fast to validate against rule-based current state |
| Evaluation target | Precision at top decile > 40% | Prior XGBoost (leaky) achieved 15% — this is the bar to beat |
| Training window | 2022–present | Pre-2022 has migration gaps (Karthik, W-02) |

---

## 6. What Decision This Pipeline Enables

> **Today:** Deepa manually decides who to call, email, and WhatsApp every Monday — using undocumented rules, ~2 hours of filtering, with no model, no holdout, and no early signal.

> **After this pipeline:** A weekly Airflow DAG runs Sunday night → scored list lands in Snowflake and Salesforce by Monday morning → 4-tier risk label (High / Medium / Low / Do Not Contact) → Deepa executes campaigns, not lists → Rohan views executive dashboard (at-risk count, actioned, recovery rate).

No DS involvement required in weekly execution. Deepa SPOF eliminated.

---

## 7. Measurable Success Criteria

| Metric | Current State | Target | Timeframe |
|---|---|---|---|
| RC Direct churn rate | 31.4% | **< 20%** | Within 2 scored quarters post-launch |
| Model precision at top decile | 15% (prior leaky model) | **> 40%** | At model evaluation |
| Weekly list generation | Manual, 2 hrs, fragile cron | **Automated, Sunday night, zero manual** | From launch |
| Retention rate measurement | Inflated (no holdout) | **Holdout-corrected baseline established** | At first scored quarter |
| RC Direct outreach cost | INR 13.10/customer/cycle, manually triggered | **Same cost, model-prioritised — budget concentrated on High/Medium risk** | From launch |

---

## 8. Consequence of Inaction

At current RC Direct churn rate of **31.4%:**
- ~89,550 RC Direct customers churn annually
- ~INR 548 Cr gross revenue lost per year
- ~1,700 RC Direct customers cross the 45-day threshold **every week** with no early signal-based outreach
- Every 4-week delay = ~6,800 customers who could have been reached earlier, weren't

The retention program is operating blind. The pipeline is not an optimisation — it is the **foundational instrument** for running a retention program at this scale.

---

## 9. Scope Boundaries

### In Scope — Phase 1
- Weekly batch scoring: RC Direct + Pedigree Club
- Output: Snowflake table + Salesforce-importable scored list (4-tier risk label)
- Airflow DAG (MWAA 2.8.1): end-to-end pipeline — feature engineering, inference, output
- Freshdesk ingestion pipeline (complaints feature — DE scope)
- Executive dashboard data model (Rohan weekly summary: at-risk count, actioned, recovery rate)

### Out of Scope — Phase 1
| Capability | Reason |
|---|---|
| Real-time event scoring | No event stream data — all features are batch aggregates |
| Automated model retraining | Requires monitoring infrastructure — Phase 2 |
| CRM/Salesforce integration (automated push) | Output table is Phase 1 — automated CRM write is Phase 2 |
| Uplift/causal modelling | No historical treatment/control assignment |
| LTV modelling | No pricing, margin, or CLV columns in data |
| Churn reason analysis | Insufficient qualitative signal in Phase 1 data |
| Multi-touch attribution | No campaign sequence data |

---

## 10. Open Items

| # | Item | Owner | Target |
|---|---|---|---|
| OI-7-01 | Formal churn label definition — align CRM (60d), Finance (120d), current system (45d/90d) | Priya + Finance + Karthik | Before model training |
| OI-7-02 | vet_referred InfoSec classification ruling | Rahul Mehta / InfoSec | ~2–3 weeks |
| OI-7-03 | subscription_active: confirm whether cause of churn or synonymous with churn event | Karthik + Priya | Before feature engineering |
| OI-7-04 | Holdout group design — agree on control group % before first scored run | Harshita + Priya | Before first production score |
