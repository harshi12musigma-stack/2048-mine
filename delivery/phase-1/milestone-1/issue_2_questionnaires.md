# Issue #2 — Stakeholder Questionnaires
**Project:** alpha-test | **Client:** Mars India
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Date:** 2026-03-30

---

> ⚠️ **Source hierarchy:** All answers in this document are **user-provided (primary source)**.
> Data from `mars_churn_sample.csv` is used as **enrichment only** — to add quantitative depth
> to answers already given. Data does not substitute for human answers on any question.
> Fields marked **[TBD]** require confirmation from Mars client — not assumed or data-derived.

---

## 👷 Practitioner Doer — Data & Technical

### Q1: What data sources are available?

**Six source systems confirmed** (inferred from column groupings, to be validated with Mars):

| Source System | Columns Present | Notes |
|---|---|---|
| Customer / CRM | customer_id, segment, region, city, registration_date | Tokenised ID format (MARS-IN-XXXXXX) — PII already removed |
| Transaction DB | last_purchase_date, purchase_count_90d/12m, avg_order_value_inr, total_spend_12m_inr | INR currency — India market confirmed |
| Loyalty Platform | loyalty_tier, loyalty_points_balance | 4 tiers: Platinum, Gold, Silver, Bronze |
| Subscription System | subscription_active, days_since_subscription_change | Binary active flag |
| Marketing Engagement | email_open_rate_90d, whatsapp_engaged_90d, num_campaign_clicks_90d | Multi-channel engagement signals |
| Customer Service | num_complaints_12m | Dissatisfaction indicator |
| Vet Referral System | vet_referred | RC_Direct segment only |

**Full dataset details and additional sources** (beyond the 80-record sample): **[TBD — Mars to confirm]**

---

### Q2: Is data already in Snowflake?

**[TBD — requires client confirmation]**

The dataset structure (clean tokenised IDs, no raw PII, structured column naming) is consistent with data that has already passed through a warehouse landing layer. Pseudonymisation appears to be applied upstream. Whether the source is Snowflake or another system needs to be confirmed by Mars.

---

### Q3: Prior churn or retention modeling on Mars data?

**No evidence of prior ML model.** No `model_score`, `prev_risk_flag`, or `campaign_treatment` column exists. The only at-risk signal in the data is raw recency (`days_since_last_purchase`), consistent with a rule-based or RFM approach as the current state. No ML baseline exists in this dataset.

**Prior work at Mu Sigma on Mars:** [TBD — to be confirmed]

---

### Q4: Airflow deployment type?

**[TBD — requires infrastructure confirmation]**

---

### Q5: Known data quality issues?

Issues identified and flagged by the user:

| Issue | Detail | Action Required |
|---|---|---|
| Sample size | 80-record sample only — curated, not representative of full population | Full dataset required before model training |
| Churn label definition | `churn_label = 1` maps exactly to `days_since_last_purchase >= 90` in all 80 rows — definition is implicit, not documented | **Must be formally agreed before any model training begins** |
| Bronze tier churn rate | 60.7% (17/28 customers) — unusually high | Investigate in production data for label contamination or sampling bias |
| Vet referral signal | All 11 churned RC_Direct customers have `vet_referred = Y` — vet referral is not a retention signal; prescription lapse appears to be a real failure mode | Validate on larger population |
| Email open rate leakage | `email_open_rate` for all churned customers rounds to 0.0 — risk of post-churn data leakage (feature values recorded after churn event) | Check during EDA — may require temporal validation |

---

## 🧭 Practitioner Leader — Project Governance

### Q6: Expected scoring cadence?

**Weekly batch — confirmed.**

Rationale from user: Feature windows are 90-day and 12-month aggregates. The `days_since_last_purchase` distribution (2–197 days) shows risk building over weeks, not hours. Daily scoring would add no predictive signal over weekly for this feature set.

---

### Q7: Hard deadline for Phase 1 BRD delivery?

**[TBD]**

---

### Q8: What is explicitly out of scope for Phase 1?

The following are out of scope based on what the data does and does not support:

| Out of Scope Item | Reason |
|---|---|
| Real-time / event-driven scoring | No event timestamps or clickstream data present |
| Uplift / causal modeling | No treatment/control assignment column |
| Multi-touch attribution | No campaign sequence or journey data |
| Geographic / demographic segmentation beyond region+city | No age, household, or income data |

**Additional Phase 1 exclusions from Mars side:** [TBD]

---

### Q9: Team constraints?

**[TBD — Vishwavani and Savikar availability to be confirmed]**

---

## 👔 Business User — Mars Client

### Q10: Primary business goal?

**Protect RC_Direct (Royal Canin) revenue by reducing churn from 31.4%.**

Key context from user:
- RC_Direct active customers average **₹61,202/year** in spend
- PED_Club active customers average **₹17,719/year** — RC_Direct is **3.4× higher value**
- RC_Direct churn rate (31.4%) is higher than PED_Club (24.4%)
- RC_Direct is simultaneously the highest-value and highest-risk segment

Specific target (e.g. reduce churn by X%) and revenue protection goal: **[TBD from Mars]**

---

### Q11: Segment / product line priority?

**Both RC_Direct and PED_Club in scope. RC_Direct is the clear priority.**

| Segment | Product Line | Records | Churn Rate | Avg Active Spend/Year |
|---|---|---|---|---|
| RC_Direct | Royal Canin | 35 | 31.4% | ₹61,202 |
| PED_Club | Pedigree | 30 | 24.4% | ₹17,719 |
| PED_Club | Whiskas | 8 | [TBD] | [TBD] |
| PED_Club | Sheba | 7 | [TBD] | [TBD] |

RC_Direct's distinct vet-referral signal (`vet_referred`) is not present in PED_Club — separate feature engineering may be needed per segment.

---

### Q12: Current process for identifying at-risk customers?

**Implied rule-based recency engine** (user's assessment):

The `churn_label = 1` maps exactly to `days_since_last_purchase >= 90`. No `prev_risk_flag`, no score column, no treatment history column exists. This is consistent with a manual trigger: *"no purchase in 90 days = at risk."*

The pipeline must identify risk **before day 90**, using declining engagement signals visible in the data: `email_open_rate_90d`, `whatsapp_engaged_90d`, `num_campaign_clicks_90d`, `loyalty_points_balance`.

**Formal confirmation of current process from Mars:** [TBD — Issue #4 SME Walkthrough]

---

### Q13: PII / data governance restrictions?

**PII already handled in sample dataset:**
- `customer_id` uses tokenised format (MARS-IN-XXXXXX) — not PII
- No names, email addresses, phone numbers, or addresses present
- Pseudonymisation policy appears to be in place upstream

**Formal DPA, field-level data classification, and regulatory scope:** [TBD — Mars Legal, required before any production data use]

---

## 🏢 Business Leader — Mars Executive

### Q14: Success at 6 months post-launch?

**Target from user (to be validated with Mars executive):**
- Reduce overall churn from 27.5% to below 18% in the scored population
- RC_Direct specifically: from 31.4% to below 20%
- Priority intervention cohorts: Bronze tier (currently 60.7% churn) and Silver tier (18.5%)
- Gold and Platinum tiers have 0% churn in sample — no retention spend needed for these

**Formal success criteria from Mars:** [TBD]

---

### Q15: Cost of inaction?

**Quantified from user + data:**

In the 80-customer sample:
- 11 churned RC_Direct customers × ₹61,202 avg active annual spend = **~₹6.7L annualised revenue at risk** (sample only)
- Scales linearly to full customer population
- **100% of churned customers** have `email_open_rate ≈ 0` and `whatsapp_engaged = 0` — once churned, re-engagement through existing channels is effectively zero
- No win-back data exists in the dataset — **churn is one-way** in this customer base

**Full revenue impact at population scale:** [TBD — requires total customer count from Mars]

---

### Q16: Hard constraints on tooling / cloud / data sharing?

**[TBD — not determinable from data. Requires Mars IT/Legal confirmation]**

---

## Open Items

| ID | Item | Priority | Owner | Required By |
|---|---|---|---|---|
| OI-Q-01 | Formal churn label definition agreed with Mars | 🔴 Critical | Harshita + Mars | Before Issue #7 (Problem Statement) |
| OI-Q-02 | Full dataset volume and complete source system details | 🔴 High | Mars / Harshita | Before Issue #5 (As-Is Workflows) |
| OI-Q-03 | Snowflake / Airflow environment confirmation | 🔴 High | Mars IT | Before Phase 2 (Architecture) |
| OI-Q-04 | Specific business targets (churn reduction %, revenue goal) | 🟡 Medium | Mars Business Leader | Before Issue #7 |
| OI-Q-05 | Formal DPA and field-level data classification | 🔴 Critical | Mars Legal | Before any production data use |
| OI-Q-06 | Phase 1 deadline | 🟡 Medium | Harshita + Mars | Before Issue #9 (KPIs) |
| OI-Q-07 | Current manual at-risk process (formal confirmation) | 🟡 Medium | Mars Business User | Issue #4 (SME Walkthrough) |
| OI-Q-08 | Bronze tier churn rate validation on full population | 🟡 Medium | Harshita (DE/DS) | Before model training |
| OI-Q-09 | Email open rate temporal leakage check | 🟡 Medium | Harshita (DS) | EDA phase |
| OI-Q-10 | Team constraints — Vishwavani/Savikar availability | 🟡 Medium | Harshita | Before project planning |

---

*Primary source: User-provided answers (Harshita Gupta, 2026-03-30).
Data enrichment: mars_churn_sample.csv — used to add quantitative depth only, not to substitute for human answers.*
