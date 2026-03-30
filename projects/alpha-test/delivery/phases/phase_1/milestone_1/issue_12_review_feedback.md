# Issue #12 — BRD Review Sessions & Stakeholder Feedback
**Project:** alpha-test — Churn Prediction Pipeline (Airflow + Snowflake + Python)
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Completed:** 2026-03-30

---

## 1. Review Session Plan

### Session Schedule

| Session | Audience | Duration | Format | Owner |
|---|---|---|---|---|
| RS-01 | BSH Executive Sponsors + BU Heads | 60 min | Structured walkthrough of BRD sections 1–6 | TDS4 Project Leader |
| RS-02 | Analytics / Data Team (TDS3, TDS2) | 90 min | Deep-dive on sections 4–9 (as-is, to-be, KPIs, risks) | TDS3 Lead |
| RS-03 | IT / Infosec | 45 min | Section 7 (infrastructure assumptions) + open items OI-02, OI-03 | TDS3 + IT Lead |
| RS-04 | Retention Team | 30 min | Section 5 (future workflow consumption) | TDS4 |

### Agenda Template (RS-01 — Executive Sponsor Review)

```
[00:00–05:00] Introductions & context-setting
[05:00–20:00] Walk through: Problem Statement + Current State
[20:00–35:00] Walk through: Future State + Business Impact / ROI
[35:00–50:00] Walk through: KPIs + Success Criteria + Go/No-Go Gates
[50:00–60:00] Open items discussion + pre-sign-off decisions (OI-01 to OI-05)
```

---

## 2. Feedback Capture Template

*Each reviewer completes this form after the session.*

| Section | Reviewer Comment | Priority | Resolution |
|---|---|---|---|
| Stakeholder Map | | | |
| Problem Statement | | | |
| Current State | | | |
| Future State | | | |
| Business Impact | | | |
| KPIs & Success Criteria | | | |
| Assumptions | | | |
| Risk Register | | | |
| Out of Scope | | | |
| Open Items | | | |

---

## 3. Feedback Log (Session Outcomes)

*To be populated during review sessions. Pre-populated with anticipated feedback based on conflict register (Issue #3).*

### RS-01 — Executive Sponsor Session (Anticipated)

| # | Feedback Item | Source | Action Required | Status |
|---|---|---|---|---|
| F-01 | Confirm baseline churn rate — 2.5% is a benchmark, not actuals | BU Head | Pull actuals from CRM; update Section 6 | 🔲 Open |
| F-02 | Clarify scoring cadence decision — weekly vs daily for top segment | BSH Sponsor | Document decision from C-01 conflict resolution | 🔲 Open |
| F-03 | Confirm churn definition: account-level vs contact-level | BSH Sponsor | Lock definition; update BRD Section 3 + Assumptions A-13, A-14 | 🔲 Open |
| F-04 | Intervention recommendations — confirm Phase 2 scope only | BSH Sponsor | Confirm in out-of-scope section; TDS3 to acknowledge | 🔲 Open |

### RS-02 — Analytics Team Session (Anticipated)

| # | Feedback Item | Source | Action Required | Status |
|---|---|---|---|---|
| F-05 | Clarify feature store ownership — who maintains it after Phase 1? | TDS3 Lead | Add RACI entry for feature store maintenance | 🔲 Open |
| F-06 | Confirm MLflow decision before Phase 9 build begins | TDS3 | Chase IT for OI-03 resolution; fallback = Snowflake metadata | 🔲 Open |
| F-07 | Define retraining trigger — monthly fixed or drift-based? | TDS3 Lead | Update Section 5.1 workflow and KPI R-07 | 🔲 Open |
| F-08 | SHAP coverage — confirm top-20% scope from business side | Analytics Lead | BSH to confirm SHAP-20% constraint (from C-06) | 🔲 Open |

### RS-03 — IT / Infosec Session (Anticipated)

| # | Feedback Item | Source | Action Required | Status |
|---|---|---|---|---|
| F-09 | Confirm Snowflake compute tier and warehouse sizing | IT Lead | Update Section 8 Assumption A-09 | 🔲 Open |
| F-10 | Confirm key-pair auth is in place for Airflow-Snowflake | IT Lead | Update A-07 status → confirmed | 🔲 Open |
| F-11 | PII masking scope — which fields require masking? | IT / Compliance | Generate PII field inventory; feed into Bronze ingestion design | 🔲 Open |
| F-12 | CAB process for pipeline deploys — lightweight vs full | IT Lead | Document agreed process; reference in Section 5 + Phase 12 | 🔲 Open |

### RS-04 — Retention Team Session (Anticipated)

| # | Feedback Item | Source | Action Required | Status |
|---|---|---|---|---|
| F-13 | How will retention team access Snowflake Gold scores? (BI tool, direct SQL, CSV export?) | Retention Manager | Define consumption interface; update Section 5 Future Workflow 3 | 🔲 Open |
| F-14 | What risk tiers should trigger outreach? (All High? Top N? Threshold?) | Retention Manager | Define action threshold in BRD + downstream in Gold schema | 🔲 Open |

---

## 4. BRD Revision Tracker

| BRD Section | Revision Required | Triggered By | Owner | Status |
|---|---|---|---|---|
| Section 3 (Problem Statement) | Update churn definition once locked | F-03 | TDS3 | 🔲 Pending |
| Section 6 (Business Impact) | Replace benchmark churn rate with actuals | F-01 | TDS3 Analyst | 🔲 Pending |
| Section 7 (KPIs) | Confirm retraining trigger type | F-07 | TDS3 Lead | 🔲 Pending |
| Section 8 (Assumptions) | Update A-09, A-07 with confirmed values | F-09, F-10 | IT | 🔲 Pending |
| Section 10 (Out of Scope) | Confirm intervention recommendations deferred | F-04 | BSH | 🔲 Pending |

---

## 5. Review Session Completion Checklist

- [ ] RS-01 Executive sponsor session completed
- [ ] RS-02 Analytics team session completed
- [ ] RS-03 IT/Infosec session completed
- [ ] RS-04 Retention team session completed
- [ ] All F-01 to F-14 feedback items resolved or escalated
- [ ] BRD v1.1 issued with revisions incorporated
- [ ] All 5 Open Items (OI-01 to OI-05) resolved
- [ ] BRD ready for formal sign-off (Issue #13)

---

_Deliverable generated as part of Phase 1 / Milestone 1 — BRD execution loop._
