# Issue #2 — Stakeholder Questionnaires
**Project:** alpha-test | **Client:** Mars India
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Date:** 2026-03-30
**Data source:** mars_churn_sample.csv (80 records, confirmed real Mars data)

---

> ✅ Sections marked **CONFIRMED FROM DATA** are answered from mars_churn_sample.csv analysis.
> ⚠️ Sections marked **TBD** require confirmation from Mars client contacts.

---

## 👷 Practitioner Doer — Data & Technical

### Q1: What data sources are available?
**CONFIRMED FROM DATA**

Primary source: `mars_churn_sample.csv`

| Column | Type | Description |
|---|---|---|
| customer_id | String | Mars India customer ID (format: MARS-IN-XXXXXX) |
| segment | Categorical | RC_Direct (35), PED_Club (45) |
| product_line | Categorical | Royal Canin, Pedigree, Whiskas, Sheba |
| region | Categorical | South, West, North, East, Central |
| city | String | Customer city |
| registration_date | Date | Customer since date |
| last_purchase_date | Date | Most recent purchase |
| days_since_last_purchase | Integer | Recency signal |
| purchase_count_90d | Integer | Frequency — 90-day window |
| purchase_count_12m | Integer | Frequency — 12-month window |
| avg_order_value_inr | Integer | Monetary signal (INR) |
| total_spend_12m_inr | Integer | Monetary signal (INR) |
| loyalty_tier | Categorical | Platinum, Gold, Silver, Bronze |
| loyalty_points_balance | Integer | Loyalty engagement signal |
| vet_referred | Boolean (Y/N) | Veterinary referral indicator |
| email_open_rate_90d | Float | Email engagement signal |
| whatsapp_engaged_90d | Boolean (0/1) | WhatsApp engagement |
| num_campaign_clicks_90d | Integer | Campaign response signal |
| num_complaints_12m | Integer | Service dissatisfaction signal |
| subscription_active | Boolean (Y/N) | Active subscription indicator |
| days_since_subscription_change | Integer | Subscription stability signal |
| **churn_label** | **Binary (0/1)** | **Target variable — present in dataset** |

**Additional sources needed (TBD):** Full historical dataset beyond 80-record sample; CRM system name; transaction database details; any external data (vet records, campaign history).

---

### Q2: Is data already in Snowflake?
**TBD** — Sample provided as CSV. Full ingestion pipeline needs to be scoped once the complete data architecture from Mars is confirmed.

---

### Q3: Prior churn modeling on Mars data?
**TBD** — Not confirmed at this stage. To be asked of Mars team.

---

### Q4: Airflow deployment type?
**TBD** — Not confirmed. To be confirmed with infrastructure/IT contact.

---

### Q5: Known data quality issues?
**CONFIRMED FROM DATA (sample)**

| Check | Result |
|---|---|
| Null values | ✅ Zero nulls across all 22 columns (80-record sample) |
| Customer ID format | ✅ Consistent MARS-IN-XXXXXX format |
| Churn label coverage | ✅ Present for all 80 records |
| Date format | ✅ Consistent YYYY-MM-DD |
| Numeric ranges | ✅ Plausible — no obvious outliers in sample |

**Full dataset quality assessment:** Required once complete dataset is available. Sample too small (80 records) for production modeling — full volume TBD.

---

## 🧭 Practitioner Leader — Project Governance

### Q6: Expected scoring cadence?
**TBD** — Weekly batch is a reasonable starting point for petcare loyalty programs; to be confirmed with Mars.

### Q7: Hard deadline for Phase 1 BRD?
**TBD**

### Q8: What is explicitly out of scope for Phase 1?
**TBD** — To be confirmed. Suggested candidates: real-time scoring, CRM integration, automated campaign triggers.

### Q9: Team constraints?
**TBD** — Vishwavani and Savikar availability to be confirmed.

---

## 👔 Business User — Business Goals & Context

### Q10: Primary business goal?
**PARTIALLY CONFIRMED FROM DATA**

Based on the dataset structure and churn label presence:
- **Confirmed:** Predict customer churn for Mars India petcare/food customers
- **Confirmed segments:** RC_Direct (Royal Canin) and PED_Club (Pedigree)
- **Confirmed market:** India (INR currency, Indian regions and cities)
- **Churn rate in sample:** 27.5% (22 of 80 customers churned)
- **Target:** Reduce churn rate and enable proactive retention interventions

*Specific business target (e.g. reduce churn by X%) — TBD from Mars.*

---

### Q11: Which segment / product line is priority?
**CONFIRMED FROM DATA**

| Product Line | Count | % of Sample |
|---|---|---|
| Royal Canin | 35 | 43.8% |
| Pedigree | 30 | 37.5% |
| Whiskas | 8 | 10.0% |
| Sheba | 7 | 8.8% |

All four product lines present. Both segments (RC_Direct, PED_Club) in scope. Priority between them — **TBD from Mars.**

---

### Q12: Current process for identifying at-risk customers?
**TBD** — Unknown at this stage. Key question for SME walkthrough (Issue #4).

---

### Q13: PII / data governance restrictions?
**PARTIALLY CONFIRMED FROM DATA**

- No names, emails, phone numbers, or addresses in sample dataset
- Customer IDs are internal Mars format (MARS-IN-XXXXXX) — not PII
- Data appears to be behavioural/transactional — lower PII risk
- **Formal data governance / DPA status with Mars:** TBD — must be confirmed before any production data sharing

---

## 🏢 Business Leader — Strategic Context

### Q14: Success at 6 months post-launch?
**TBD** — To be confirmed with Mars executive sponsor once onboarded.

### Q15: Cost of inaction?
**PARTIALLY INFERRED FROM DATA**

At 27.5% churn rate in sample:
- If representative of full customer base, over 1 in 4 customers is churning
- Exact revenue impact TBD — requires total customer count and avg revenue per customer from Mars

### Q16: Hard constraints on tooling / cloud / data sharing?
**TBD** — To be confirmed with Mars IT/legal.

---

## Open Items from Questionnaire

| ID | Item | Owner | Required By |
|---|---|---|---|
| OI-Q-01 | Full dataset volume and complete source system details | Mars / Harshita | Issue #5 (As-Is Workflows) |
| OI-Q-02 | Snowflake / Airflow environment confirmation | Mars IT / Harshita | Issue #6 (Business Impact) |
| OI-Q-03 | Prior modeling history on Mars data | Mars / Harshita | Issue #7 (Problem Statement) |
| OI-Q-04 | Specific business targets (churn reduction %, revenue goal) | Mars Business Leader | Issue #7 |
| OI-Q-05 | Formal data governance / DPA status | Mars Legal | Before any production data use |
| OI-Q-06 | Scoring cadence and deadline | Mars + Harshita | Issue #9 (KPIs) |
| OI-Q-07 | Current manual process for at-risk identification | Mars Business User | Issue #4 (SME Walkthrough) |

---

*Answers sourced from: (1) mars_churn_sample.csv direct analysis, (2) user-provided context. TBD items require Mars client input — not assumed.*
