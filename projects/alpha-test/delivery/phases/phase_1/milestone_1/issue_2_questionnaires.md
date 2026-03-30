# Issue #2 — Structured Stakeholder Questionnaires (Goals, KPIs, Pain Points)
**Project:** alpha-test — Churn Prediction Pipeline (Airflow + Snowflake + Python)
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Completed:** 2026-03-30

---

## Overview

Three questionnaire sets prepared — one per stakeholder group — to elicit goals, KPIs, pain points, and data context for the churn prediction pipeline.

---

## Questionnaire Set A — Business / Executive Stakeholders (BSH Sponsors, BU Heads)

### Section 1: Strategic Goals
1. What business outcome are you expecting from a churn prediction pipeline? (e.g., reduce churn rate by X%, increase retention revenue by $Y)
2. Which customer segment is the highest priority for churn intervention?
3. What is the current estimated annual revenue lost to churn?
4. What retention actions are currently taken — and at what threshold do you act?
5. What does "success" look like for this pipeline in 6 months? In 12 months?

### Section 2: KPIs & Success Metrics
6. What is the current churn rate (monthly/quarterly)?
7. Which KPIs do you track today for customer health? (e.g., NPS, CSAT, usage frequency, payment recency)
8. What churn rate reduction target would justify investment in this pipeline?
9. How do you measure retention campaign effectiveness today?
10. What reporting cadence do you expect from the prediction system? (Daily / Weekly / Monthly)

### Section 3: Pain Points
11. What is the biggest gap in your current approach to identifying at-risk customers?
12. Are there false-positive problems today — customers flagged as churning who don't leave?
13. Are there false-negative problems — customers who churn without warning?
14. What manual steps in your current process would you like automated?
15. Are there political/organizational barriers to acting on churn predictions?

---

## Questionnaire Set B — Data & Analytics Stakeholders (TDS3/TDS4, Data Leads)

### Section 1: Data Availability
1. What customer data sources are currently available? (transactional DB, CRM, event logs, billing, usage)
2. What is the historical depth of customer data available? (months/years)
3. Is customer data stored in Snowflake already, or does it need to be ingested?
4. What is the granularity of transaction/event data? (per-event, daily aggregate, monthly summary)
5. Are there external data sources to consider? (macroeconomic, weather, campaign data)

### Section 2: Data Quality
6. What known data quality issues exist? (nulls, duplicates, inconsistent IDs, schema drift)
7. How frequently does the source schema change?
8. Is there a master customer ID that links across all source systems?
9. What percentage of records have complete feature coverage?
10. Are there PII or data governance restrictions on any fields?

### Section 3: Current Analytics State
11. Has any churn modeling been attempted before? What was the outcome?
12. What features/signals have historically correlated with churn in your context?
13. What ML frameworks or tools are currently used by the team?
14. Is there a model registry or experiment tracking system in place?
15. What scoring frequency is needed? (real-time, daily batch, weekly batch)

### Section 4: Technical Constraints
16. What is the Airflow version and deployment type? (self-hosted, MWAA, Cloud Composer)
17. What is the Snowflake tier and are there compute/cost constraints?
18. Are there CI/CD pipelines in place for data/ML code?
19. What is the expected pipeline SLA? (max acceptable latency for scoring output)
20. Are there environment restrictions (dev/staging/prod) that affect pipeline design?

---

## Questionnaire Set C — Operations / IT Stakeholders (IT/Infosec, Platform Teams)

### Section 1: Infrastructure
1. What cloud provider and region is the Snowflake environment hosted in?
2. What network/VPC restrictions exist for Airflow-Snowflake connectivity?
3. Is there an existing data lake or object storage (S3/GCS/ADLS) that feeds Snowflake?
4. What authentication mechanism is used for Snowflake? (key-pair, OAuth, SSO)
5. Are there approved Python package lists or restrictions on library installations?

### Section 2: Security & Compliance
6. What data classification applies to customer data in scope? (PII, PCI, PHI)
7. What masking or anonymization is required before use in ML pipelines?
8. Are there regulatory requirements affecting data retention or model auditability? (GDPR, CCPA, HIPAA)
9. Who approves new Snowflake warehouse/role provisioning?
10. Is there a change management process for production pipeline deployments?

### Section 3: Monitoring & Operations
11. What monitoring tools are in place for Airflow DAGs? (alerting, SLA breach notification)
12. Who is on-call for pipeline failures? What is the escalation path?
13. What is the expected uptime SLA for the churn scoring service?
14. Are there existing runbooks for Snowflake or Airflow incidents?
15. What logging and audit trail requirements exist for ML model outputs?

---

## Questionnaire Administration Plan

| Stakeholder Group | Format | Owner | Target Completion |
|---|---|---|---|
| BSH Sponsors / BU Heads | 60-min structured interview | TDS4 Project Leader | Week 1 |
| Data/Analytics Leads (TDS3/TDS4) | 90-min working session | TDS3 Problem Solver | Week 1 |
| IT / Infosec | 45-min technical review | TDS3 + IT Lead | Week 1–2 |

---

## Next Steps
- [ ] Distribute questionnaires to identified stakeholders (from Issue #1 stakeholder map)
- [ ] Schedule interviews / working sessions
- [ ] Consolidate responses → Issue #3 (Capture and consolidate stakeholder expectations)

---

_Deliverable generated as part of Phase 1 / Milestone 1 — BRD execution loop._
