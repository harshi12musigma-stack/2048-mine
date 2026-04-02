# Issue #8 — Map Future Workflows with Automation Opportunities
**Project:** Mars India Churn Prediction Pipeline  
**Phase:** 1 — Requirement Gathering  
**Milestone:** 1 — Business Requirement Document  
**Date:** 2026-04-02  
**Status:** Complete  

---

## Overview

The future-state vision replaces the fragile, manual, recency-threshold-based process with a fully automated ML-driven weekly scoring and campaign-triggering pipeline. Every manual handoff and single-point-of-failure in the as-is state is eliminated.

---

## Future Workflow 1 — Weekly Scoring Pipeline (Automated)

**Trigger:** Airflow DAG scheduled Sunday 11 PM IST  
**Owner:** Automated (no human intervention required)

| Step | System | Action | Notes |
|------|--------|--------|-------|
| 1 | MWAA Airflow | DAG triggered on schedule | Python 3.11, MWAA 2.8.1 |
| 2 | Snowflake (dbt incremental) | Feature mart refresh — 90d/12m aggregates | Replaces dbt full-refresh (~90 min) with incremental model |
| 3 | Snowflake | Feature extraction to model input table | MU_SIGMA_DEV → MARS_DEV schema |
| 4 | MWAA (LightGBM) | Batch inference — score all scoreable customers | RC Direct: ~170K; PED Club: ~650K |
| 5 | Snowflake | Write output: customer_id, churn_score, risk_tier, recommended_action | 4-tier: High / Medium / Low / Do Not Contact |
| 6 | Salesforce (SFMC) | Automated sync from Snowflake → SFMC campaign segment | Replaces Deepa's manual list processing (~2 hrs/week) |
| 7 | MWAA | DAG completes — Slack/email alert to Harshita + Priya | Score delivery confirmation by Monday 6 AM IST |

**Automation gain:** Deepa's 2-hour Monday manual process → fully automated Sunday night  
**SPOF eliminated:** Deepa filter logic codified in DAG; any team member can operate

---

## Future Workflow 2 — Campaign Execution (Partially Automated)

**Trigger:** Scores in Salesforce by Monday 6 AM IST  
**Owner:** Priya Sharma (campaign lead) — review only, no list processing

| Step | System | Action | Notes |
|------|--------|--------|-------|
| 1 | SFMC | Auto-segment: High/Medium risk RC Direct for outbound sequence | Suppression: Do Not Contact tier excluded automatically |
| 2 | SFMC | Email send — Day 0 of 10-day sequence | Campaign brief by Tuesday 6 PM, Wednesday send |
| 3 | Kaleyra (WABA) | WhatsApp send — personalised with vet name for RC Direct | Replaces manual SFMC↔Kaleyra reconciliation |
| 4 | Retention desk | Outbound calls — High risk only, RC Direct (capacity: ~5K/week) | List extracted from scored output, no manual filter |
| 5 | SFMC + Snowflake | Response tracking — re-purchase within 30 days flags as saved | Future: holdout group assignment at step 1 |

**Automation gain:** Manual SFMC↔Kaleyra reconciliation eliminated; suppression logic automated  
**Open item:** SFMC integration spec — confirm Phase 1 scope with Priya and Karthik

---

## Future Workflow 3 — Reporting & Monitoring (Automated)

**Trigger:** Post-scoring (Monday morning)  
**Owner:** Automated — consumed by Rohan (executive), Harshita (operational)

| Step | System | Action | Notes |
|------|--------|--------|-------|
| 1 | Snowflake | Weekly summary table: at-risk count by tier, segment, region | Auto-refreshed after scoring DAG completes |
| 2 | SFMC / BI tool | Executive dashboard: at-risk count, actioned, recovery rate | Rohan's weekly summary — no manual prep |
| 3 | Snowflake | Model monitoring table: score distribution drift, class balance | Triggers alert if drift > threshold |
| 4 | MWAA | Data quality DAG: null checks, schema validation, Capillary freshness | Replaces silent Capillary SFTP failures |
| 5 | Airflow / Slack | Alert: pipeline failure, data freshness breach, DQ failure | Replaces reactive discovery of missed campaigns |

**Automation gain:** Rohan's weekly summary prep eliminated; Capillary silent failures caught automatically

---

## Future Workflow 4 — Model Retraining (Phase 2 — Out of Scope Phase 1)

| Step | Action | Notes |
|------|--------|-------|
| 1 | Automated retraining trigger | Triggered when precision@decile drops below 35% for 2+ consecutive weeks |
| 2 | Feature pipeline re-run | Same dbt incremental model, extended training window |
| 3 | Model validation | Champion/challenger comparison against production model |
| 4 | Deployment | New model pushed to scoring DAG after validation |

**Note:** Retraining infrastructure is out of scope for Phase 1. Phase 1 delivers a static model with monitoring. Retraining is Phase 2 scope.

---

## Automation Opportunity Summary

| As-Is Pain Point | Automation Solution | Phase |
|-----------------|-------------------|-------|
| 45-day recency threshold fires too late | ML model scores at 20–25 day signal decay | Phase 1 |
| Deepa SPOF — 2 hrs manual every Monday | Codified filter logic in Airflow DAG | Phase 1 |
| Capillary SFTP silent failures | DQ DAG with alerting | Phase 1 |
| Manual SFMC↔Kaleyra reconciliation | Automated suppression + segmentation | Phase 1 |
| No holdout — inflated retention rates | Holdout group assignment in campaign segment | Phase 1 |
| Ghost purchasers always flagged | ML features capture frequency trend vs absolute recency | Phase 1 |
| Channel-switch false positives | Identity bridge flag in model output | Phase 1 |
| No model monitoring | Drift detection + weekly summary table | Phase 1 |
| Manual retraining | Automated champion/challenger pipeline | Phase 2 |
| CRM integration | Direct Salesforce write | Phase 1 (TBC scope) |

---

## Open Items

| ID | Item | Owner | Due |
|----|------|-------|-----|
| FW-01 | Confirm SFMC integration scope for Phase 1 (automated sync vs manual export) | Priya + Karthik | Week 1 post-DPA |
| FW-02 | Holdout group assignment — confirm % split with Priya (suggested: 20% holdout) | Priya | Week 1 post-DPA |
| FW-03 | Kaleyra API availability for automated sends (vs SFMC-triggered) | Karthik + Rahul | Week 2 post-DPA |
| FW-04 | Drift alert thresholds — agree with Priya and Harshita | Harshita | Pre-UAT |
