# Business Requirement Document (BRD)
## Mars India — Churn Prediction & Retention Pipeline

**Version:** 1.0 (Draft — Awaiting Stakeholder Sign-off)  
**Date:** 2026-04-02  
**Prepared by:** Harshita Gupta (BSH), Mu Sigma Labs  
**Project:** Mars India Churn Prediction Pipeline  
**Classification:** Confidential — Mars India × Mu Sigma  
**Review Deadline:** Pre-DPA sign-off (target: early May 2026)  

---

## Table of Contents

1. Executive Summary
2. Business Context & Problem Statement
3. Stakeholder Register
4. Current State — As-Is Workflows
5. Future State — To-Be Workflows & Automation
6. Business Requirements
7. Data Requirements
8. Business KPIs & Success Criteria
9. Assumptions
10. Risks & Mitigation
11. Constraints
12. Out of Scope — Phase 1
13. Open Items
14. Sign-off

---

## 1. Executive Summary

Mars India's RC Direct subscription model is experiencing a 31.4% annual churn rate — the highest across all loyalty tiers — despite RC Direct customers being the most valuable segment at INR 61,202 average annual spend. At current churn rates, approximately INR 548 Cr in gross annual revenue is at risk.

The existing retention process relies on a fragile, manual, recency-threshold-based system operated by a single coordinator (Deepa), which fires too late (45-day lag), misses nuanced engagement signals, and has no predictive capability. A Capillary SFTP failure in March 2026 caused a full week of missed retention campaigns.

**This project delivers:** A weekly ML-driven churn prediction pipeline (LightGBM) scoring ~170K RC Direct and ~650K Pedigree Club customers every Sunday night, outputting a risk-tiered scored list (High / Medium / Low / Do Not Contact) to Snowflake and Salesforce by Monday 6 AM IST — replacing all manual processing.

**Target:** RC Direct churn reduced from 31.4% to below 20% within two scored quarters post-launch. ROI: ~845x gross on Phase 1 build cost of INR 20.7L.

---

## 2. Business Context & Problem Statement

### 2.1 Background

Mars Petcare India operates two primary customer loyalty programs: **RC Direct** (Royal Canin — vet-referred subscription customers) and **Pedigree Club**. The retention function currently uses a rule-based recency engine — if a customer has not purchased in 45+ days, they enter the at-risk list for outbound campaign.

Two converging failures have triggered this initiative:
1. **Capillary SFTP failure (March 2026):** The at-risk list was not generated for a full week. ~1,700 RC Direct customers crossed the 45-day threshold with no early outreach.
2. **Deepa SPOF:** The single retention coordinator holds all filter logic in her head. Any absence halts the entire 285K-customer retention program.

### 2.2 Problem Statement

> *Build a weekly ML-driven churn prediction pipeline that identifies RC Direct and Pedigree Club customers at risk of lapsing before the current 45-day rule triggers, enabling Mars India to run targeted retention campaigns and reduce RC Direct churn from 31.4% to below 20%.*

### 2.3 Decision Enabled

Who to call, email, and WhatsApp this week — and who NOT to contact (Gold/Platinum suppression, channel fatigue avoidance). Output: a ranked, risk-tiered scored list ready in Salesforce by Monday morning, replacing all manual judgement.

---

## 3. Stakeholder Register

### Mu Sigma Team

| Name | Role | Responsibilities |
|------|------|-----------------|
| Harshita Gupta | Project Lead / BSH | Delivery governance, stakeholder communication, milestone approvals |
| Vishwavani | Data Scientist | ML model development, feature engineering |
| Savikar Garg | Data Engineer | Airflow DAG build, Snowflake pipeline |
| Srini | Manager | Delivery oversight |
| Zubin | CTO | Executive visibility |

### Mars India Team

| Name | Role | Responsibilities |
|------|------|-----------------|
| Priya Sharma | Head of CRM & Retention | Primary business stakeholder, campaign execution, output requirements owner |
| Rohan | CRM & Retention Manager | Executive dashboard consumer — weekly summary |
| Karthik Nair | Analytics & Data Lead | Data architecture, dbt marts, Snowflake access |
| Rahul Mehta | IT Platform & Infrastructure Lead | MWAA, Snowflake provisioning, DPA owner |
| Deepa | Retention Coordinator | Operational list consumer, key UAT participant |

---

## 4. Current State — As-Is Workflows

### 4.1 Churn Identification (Every Monday Morning)

1. Karthik team runs Redash SQL query → CSV emailed to Deepa
2. Deepa manually filters for 2 hours: segments, recently contacted, pending deliveries, rough spend prioritisation
3. List split manually across SFMC (email) and Kaleyra WABA (WhatsApp)
4. Outbound calls assigned from retention desk (7 agents, ~5K calls/week capacity)

**Failure modes:** Deepa SPOF, Capillary SFTP fragility, 45-day lag, no holdout group, manual SFMC↔Kaleyra reconciliation

### 4.2 Data Pipeline (As-Is)

| Source | System | Ingestion Method | Fragility |
|--------|--------|-----------------|-----------|
| Salesforce CRM | Snowflake | Fivetran nightly | Stable |
| Oracle ecommerce | Snowflake | Fivetran nightly | Stable |
| Capillary loyalty | Snowflake | SFTP → S3 → Python script | **High — format changed twice in 2025** |
| Freshdesk complaints | Manual export | No production pipeline | **Blocker — DE scope** |
| dbt feature mart | Snowflake | Full-refresh daily (~90 min) | Medium — runtime risk |

### 4.3 Churn Definition (Inconsistent)

| Team | Working Definition |
|------|--------------------|
| CRM (Priya) | Lapsed = 45 days, Churned = 90 days |
| Marketing | 60 days |
| Finance | 120 days |
| **BRD Requirement** | **Must be formally aligned before training begins** |

---

## 5. Future State — To-Be Workflows

### 5.1 Weekly Scoring Pipeline (Automated)

**Schedule:** Airflow DAG — Sunday 11 PM IST  

1. dbt incremental mart refresh (target: <15 min, replacing ~90 min full-refresh)
2. Feature extraction → model input table in MU_SIGMA_DEV
3. LightGBM batch inference — ~170K RC Direct + ~650K PED Club customers
4. Output written: `customer_id | churn_score | risk_tier | recommended_action | id_bridge_matched`
5. 4-tier output: **High / Medium / Low / Do Not Contact** (Gold + Platinum suppression)
6. Salesforce SFMC sync — scores available by Monday 6 AM IST
7. DAG completion alert → Slack/email to Harshita + Priya

### 5.2 Campaign Execution (Partially Automated)

- SFMC auto-segments High/Medium RC Direct for 10-day outbound sequence
- WhatsApp (Kaleyra): Day 0 | Email (SFMC): Day 5 | Outbound call: Day 10 (High risk only)
- Suppression: Do Not Contact tier automatically excluded — no manual filter
- Holdout group: 20% suppression control (to be confirmed with Priya)

### 5.3 Reporting & Monitoring (Automated)

- Weekly summary table: at-risk count by tier, segment, region → Rohan dashboard
- Model monitoring: score distribution drift alert (threshold TBD with Harshita)
- DQ DAG: null checks, schema validation, Capillary freshness alert (<15 min from failure)

---

## 6. Business Requirements

| ID | Requirement | Priority | Source |
|----|------------|----------|--------|
| BR-01 | System must score all scoreable customers weekly (RC Direct ~170K, PED Club ~650K) | Must Have | Priya W-01 |
| BR-02 | Scores must be available in Snowflake by Monday 6 AM IST | Must Have | Priya W-01 |
| BR-03 | Output must include 4-tier risk label: High / Medium / Low / Do Not Contact | Must Have | Priya W-01 |
| BR-04 | Gold and Platinum tier customers must be auto-suppressed (Do Not Contact) | Must Have | Priya W-01 |
| BR-05 | Scored list must be directly consumable by SFMC without IT involvement | Must Have | Priya W-01 |
| BR-06 | Deepa SPOF must be eliminated — scored list auto-generated without manual processing | Must Have | Harshita |
| BR-07 | Model must identify at-risk signals before 45-day threshold (target: 20–25 day early warning) | Must Have | Priya W-01 |
| BR-08 | Executive dashboard: weekly at-risk count, actioned count, recovery rate for Rohan | Must Have | Rohan (via Priya) |
| BR-09 | All processing must remain within Mars Snowflake — no data to external infrastructure | Must Have | Rahul W-03 / DPA |
| BR-10 | Capillary SFTP failure must generate alert within 15 min — no silent failures | Must Have | Karthik W-02 |
| BR-11 | Identity bridge flag (`id_bridge_matched`) must be included in model output | Should Have | Karthik W-02 |
| BR-12 | Holdout group (suggested 20%) for true retention rate baseline measurement | Should Have | Mu Sigma recommendation |
| BR-13 | Freshdesk complaint data ingestion pipeline | Should Have | Karthik W-02 |
| BR-14 | SFMC automated sync (Snowflake → SFMC direct) | Could Have | Priya W-01 — scope TBC |

---

## 7. Data Requirements

### 7.1 Source Systems

| Source | Key Fields | Location | DPA Required |
|--------|-----------|----------|-------------|
| Salesforce CRM | customer_id, segment, region, registration_date | Snowflake via Fivetran | Yes |
| Oracle transactions | last_purchase_date, purchase_count_90d, purchase_count_12m, avg_order_value_inr | Snowflake via Fivetran | Yes |
| Capillary loyalty | loyalty_tier, loyalty_points_balance | Snowflake via SFTP/S3 | Yes |
| Subscription system | subscription_active¹, days_since_subscription_change¹ | Snowflake | Yes |
| SFMC marketing | email_open_rate_90d, num_campaign_clicks_90d | Snowflake | Yes |
| Kaleyra WABA | whatsapp_engaged_90d | Snowflake | Yes |
| Freshdesk | num_complaints_12m | Not yet ingested — Phase 1 DE task | Yes |
| Vet referral system | vet_referred² | Snowflake | Yes |

¹ **Leakage risk — subscription_active and days_since_subscription_change must NOT be used as model features until leakage resolution confirmed.**  
² **Pending InfoSec classification ruling — treat as optional feature until cleared.**

### 7.2 Churn Label Definition (PENDING FORMAL AGREEMENT)

**Proposed:** `churn_label = 1` if `days_since_last_purchase ≥ 90` at scoring date  
**Requires sign-off from:** Priya Sharma (CRM) + Mars Finance + Karthik (Analytics)  
**Blocker:** Training cannot begin until this is formally ratified.

### 7.3 Feature Constraints

| Feature | Status | Notes |
|---------|--------|-------|
| subscription_active | ⚠️ Blocked | Leakage decision pending — exclude until resolved |
| vet_referred | ⚠️ Conditional | InfoSec ruling pending — design without as primary signal |
| email_open_rate_90d | ✅ Cleared | Validate 30-day lag before churn label in EDA |
| All other features | ✅ Cleared | Standard EDA and leakage checks in Phase 2 (Data Discovery) |

---

## 8. Business KPIs & Success Criteria

### Business Outcomes

| KPI | Baseline | Target | Measurement |
|-----|----------|--------|-------------|
| RC Direct churn rate | 31.4% | < 20% | Monthly cohort — % inactive >90d |
| Pedigree Club churn rate | ~18.5% | < 15% | Monthly cohort |
| RC Direct retention rate | ~22% (no holdout) | > 30% (holdout-corrected) | Re-purchase within 30d of contact |
| Revenue protected | — | INR 174.9 Cr/yr (at 10pp RC Direct reduction) | Attributed revenue model |

### Model Performance

| KPI | Target |
|-----|--------|
| Precision at top decile | > 40% |
| AUC-ROC | > 0.75 |
| Feature leakage check | Pass (subscription_active excluded) |
| Bronze-tier recall | > 50% |

### Pipeline Operational

| KPI | Target |
|-----|--------|
| DAG success rate | > 99% (4-week rolling) |
| Score delivery SLA | Snowflake by Monday 6 AM IST |
| Manual interventions/month | 0 (target) |
| DQ gate pass rate | 100% |

---

## 9. Assumptions

| ID | Assumption |
|----|-----------|
| A-01 | Churn = days_since_last_purchase ≥ 90 (pending formal ratification) |
| A-02 | Training window: 2022–present |
| A-03 | subscription_active excluded from feature set (leakage) |
| A-04 | dbt mart refactored to incremental for Phase 1 |
| A-05 | vet_referred available post InfoSec ruling (~2 weeks) |
| A-06 | Identity bridge coverage: ~68% RC Direct, ~55% PED Club |
| A-07 | LightGBM primary; Logistic Regression baseline |
| A-08 | DPA signed by early May 2026 |
| A-09 | Code-complete: June 1; Production: June 30 |
| A-10 | All processing within Mars Snowflake (AWS ap-south-1, Mumbai) |
| A-11 | Snowflake X-Small for dev/staging; scale-up for production |
| A-12 | Service account provisioning: 2-week SLA from DPA signing |

---

## 10. Risks Summary

| ID | Risk | Severity | Top Mitigation |
|----|------|---------|---------------|
| R-01 | DPA delay — no live data access until signed | Critical | Track weekly; April = design-only using sample |
| R-02 | Churn label not formally agreed | Critical | Block training until ratified in BRD sign-off |
| R-03 | Feature leakage (subscription_active) | Critical | Hard exclusion gate before EDA |
| R-04 | June 30 deadline missed | High | Parallel-track April design; June 1 code-complete |
| R-05 | vet_referred excluded by InfoSec | High | Design without as primary signal |
| R-06 | Team allocation not ring-fenced | High | Escalate to Srini/Zubin before DPA signed |

*Full risk register: see Issue #10 — Assumptions & Risks document.*

---

## 11. Constraints

| Constraint | Detail |
|-----------|--------|
| DPA-first | No pipeline code against live data until DPA signed — earliest early May 2026 |
| Data residency | All processing within Mars Snowflake — AWS ap-south-1 (Mumbai) |
| Production freeze | Oct 1 – Nov 15 annually — launch must complete by June 30 |
| MWAA version | Python 3.11, MWAA 2.8.1 — LightGBM package approval required |
| Production deployment | 2–4 weeks from code-complete (ServiceNow CR + architecture board + 1-week UAT) |
| Snowflake access | MU_SIGMA_DEV schema in MARS_DEV account only; production requires separate provisioning |

---

## 12. Out of Scope — Phase 1

| Item | Reason |
|------|--------|
| Real-time event scoring | No event stream / clickstream data available |
| Automated model retraining | Requires monitoring infrastructure — Phase 2 |
| CRM integration (Salesforce write) | Scope TBC with Priya — Phase 1 baseline: Snowflake table + CSV export |
| Recommendation engine | No historical treatment/response data |
| LTV modelling | No margin/CLV columns available |
| Churn reason analysis | Insufficient qualitative signals in Phase 1 data |
| Multi-touch attribution | No campaign sequence data |
| Uplift/causal modelling | No treatment/control assignment history |

---

## 13. Open Items

| ID | Item | Owner | Due | Blocker? |
|----|------|-------|-----|---------|
| OI-01 | Formal churn label ratification (≥90d) | Priya + Finance + Karthik | Pre-training | ✅ Yes |
| OI-02 | subscription_active leakage decision | Karthik | Pre-EDA | ✅ Yes |
| OI-03 | DPA signing | Rahul | Early May 2026 | ✅ Yes |
| OI-04 | vet_referred InfoSec ruling | Rahul / InfoSec | ~2 weeks | No |
| OI-05 | Team allocation confirmation (Vishwavani, Savikar %) | Harshita → Srini | Before DPA | No |
| OI-06 | LightGBM package request submission | Savikar → Rahul | Queue immediately | No |
| OI-07 | Service account provisioning request | Harshita → Rahul | Immediately post-DPA | No |
| OI-08 | Freshdesk production ingestion scope confirmation | Karthik | Week 1 post-DPA | No |
| OI-09 | SFMC integration scope (Phase 1 baseline vs automated sync) | Priya + Karthik | Pre-UAT | No |
| OI-10 | Holdout group % confirmation | Priya | Week 1 post-DPA | No |
| OI-11 | Mars Finance validation of INR 61,202 avg RC Direct spend | Karthik → Finance | Pre-sign-off | No |
| OI-12 | Rohan full name confirmation | Priya | Pre-BRD sign-off | No |

---

## 14. Sign-off

This BRD represents the agreed understanding of business requirements between Mu Sigma Labs and Mars India for the Churn Prediction Pipeline — Phase 1.

| Name | Role | Organisation | Signature | Date |
|------|------|-------------|-----------|------|
| Harshita Gupta | Project Lead / BSH | Mu Sigma | _______________ | ________ |
| Priya Sharma | Head of CRM & Retention | Mars India | _______________ | ________ |
| Karthik Nair | Analytics & Data Lead | Mars India | _______________ | ________ |
| Rahul Mehta | IT Platform & Infrastructure Lead | Mars India | _______________ | ________ |
| Rohan | CRM & Retention Manager | Mars India | _______________ | ________ |

**⚠️ Mandatory pre-sign-off conditions:**
1. Churn label formally agreed (OI-01)
2. subscription_active leakage decision documented (OI-02)
3. DPA status confirmed (in-progress is acceptable — sign-off can precede DPA signing)
4. Mars Finance validation of revenue figures (OI-11)
