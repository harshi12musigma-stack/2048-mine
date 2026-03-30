# Issue #4 — Current Business Process Analysis
**Project:** alpha-test | **Client:** Mars India
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Sessions conducted:** 2026-03-31
**Source:** Confirmed session transcripts (W-01, W-02, W-03) — no assumptions

---

> ✅ All content in this document sourced from confirmed session transcripts.
> No content invented or assumed. [TBD] marks items requiring further confirmation.

---

## Session Register

| Session | Mars Attendee | Role | Mu Sigma |
|---|---|---|---|
| W-01 | Priya Sharma | Head of CRM & Retention, Mars Petcare India | Harshita Gupta |
| W-02 | Karthik Nair | Analytics & Data Lead, Mars India | Harshita Gupta, Vishwavani |
| W-03 | Rahul Mehta | IT Platform & Infrastructure Lead, Mars India | Harshita Gupta, Savikar |

---

## W-01 — Business Process (Priya Sharma)

### Current At-Risk Identification Process

**Step-by-step as-is workflow:**

| Step | Actor | System | Action | Manual? | Output |
|---|---|---|---|---|---|
| 1 | Karthik's team | Redash + SQL | Weekly query: customers with no purchase in last 45 days (RC Direct) or ~60 days (Pedigree Club) | Cron-scheduled | CSV emailed Monday morning |
| 2 | Deepa (coordinator) | Excel | Filter by segment (RC Direct vs Pedigree Club) | ✅ Manual — ~2 hrs | Segment-split list |
| 3 | Deepa | Excel | Remove customers contacted in last 2 weeks | ✅ Manual | De-duped list |
| 4 | Deepa | Excel | Remove customers with a pending delivery | ✅ Manual | Clean at-risk list |
| 5 | Deepa | Excel | Rough prioritisation by spend | ✅ Manual | Prioritised list |
| 6 | Priya's team | Salesforce / Kaleyra / Phone | Outreach per segment-tier sequence | Partially manual | Campaign execution |

**Total manual effort:** ~2 hours every Monday before the list is usable.
**Single point of failure:** Deepa. Only person who knows the full filter logic. Undocumented.
**Fragility:** Cron job missed 2 weeks ago due to server maintenance — not noticed until Wednesday.

---

### Retention Action Sequences

**RC Direct (Royal Canin):**
```
Day 45  → WhatsApp (personalised — vet name included)
Day 50  → Email (SFMC)
Day 55  → Outbound call (7-person retention desk)
```
- Retention desk capacity: 4,500–5,000 calls/week
- Normal at-risk volume: 1,200–1,500/week. Peak: ~3,000/week
- **Capacity is not a constraint for RC Direct**

**Pedigree Club:**
```
Email + WhatsApp only (no calls)
Exception: Gold tier or annual spend > INR 20,000 → escalate to call
```
- ~8× the RC Direct customer base — call capacity doesn't scale
- High-value threshold (INR 20K) is a manual manager decision, not a model

**Tooling:**
- Email: Salesforce Marketing Cloud (SFMC)
- WhatsApp: Kaleyra via WABA
- **Not integrated** — two separate tools, manual reconciliation every week

---

### Retention Measurement

| Metric | Current State | Problem |
|---|---|---|
| RC Direct reported retention rate | ~22% | No holdout group — likely inflated |
| Pedigree Club reported retention rate | ~14% | No holdout group — likely inflated |
| Measurement method | Re-purchase within 30 days of outreach | Cannot distinguish model-driven saves vs organic returns |

---

### Pain Points (Priya's Words)

1. **Too late to act** — *"By the time someone hits 45 days, they've mentally already decided."* Needs signals at 20–25 days.
2. **Ghost purchasers** — Customers with low baseline frequency (e.g. buy every 8 weeks) never trigger the 45-day rule. They disappear without being flagged.
3. **Channel-switch false positives** — Customers who move to Amazon or vet clinic continue buying Royal Canin, but disappear from portal data. Get wrongly flagged and contacted. *"I just bought it yesterday, why are you calling?"*
4. **No individual engagement trending** — Only aggregate campaign open rates available. Cannot see an individual's engagement declining week-on-week.
5. **Two disconnected campaign tools** — SFMC for email + Kaleyra for WhatsApp. Manual reconciliation every week.

---

### Output Requirements (Confirmed by Priya)

| Requirement | Detail |
|---|---|
| Landing zone | **Salesforce** — not just Snowflake. Must be importable into SFMC without IT involvement. |
| Scoring deadline | Scores in Salesforce by **Monday EOD**. Campaign brief to SFMC by Tuesday 6 PM for Wednesday send. **Sunday night scoring preferred.** |
| Risk tiers | **4 tiers:** High / Medium / Low / **Do Not Contact** (suppression for Gold + Platinum — currently absent, causing unnecessary outreach to happy customers) |
| Output consumers | Deepa (operational scored list) + Rohan / Priya's manager (executive dashboard: at-risk count, actioned, recovery rate) |

**New stakeholders identified:**

| Name | Role | Relevance |
|---|---|---|
| Priya Sharma | Head of CRM & Retention | Primary business stakeholder — confirmed in session |
| Deepa | Retention Coordinator | Key UAT participant — will use output directly |
| Rohan | Priya's Manager | Executive dashboard consumer |

---

## W-02 — Data & Analytics (Karthik Nair)

### Data Architecture (Confirmed)

| Source | System | In Snowflake? | Method | Quality |
|---|---|---|---|---|
| Customer master | Salesforce CRM | ✅ Yes | Fivetran | Production-quality |
| Transactions | Oracle (e-commerce portal) | ✅ Yes | Fivetran | Production-quality |
| Loyalty (points, tiers) | Capillary (third-party) | ✅ Yes (raw) | Daily SFTP → S3 → custom Python script | **Fragile — format changed twice in last year. No DQ checks.** |
| Feature aggregations | dbt `marts` schema | ✅ Yes | dbt full refresh daily | 90-min runtime — needs migration to incremental |
| Complaints | Freshdesk | ⚠️ Partial | Manual export (num_complaints_12m in sample was manual) | Production pipeline not built |
| Nielsen panel | External | Available | Aggregate only — not individual | Not useful for Phase 1 |

---

### Prior Modeling History

**XGBoost model built by Manish (internal DS), early 2025. Never deployed.**

- Test set accuracy: 90% — appeared excellent
- Production precision at top decile: ~15% — essentially useless
- **Root cause: Classic feature leakage**
  - `subscription_active` used as a feature — it IS the churn event, not a predictor
  - Raw `last_purchase_date` used instead of relative recency window
  - Model learned "days since last purchase is large → churned" — same as the rule
- **Lesson:** `subscription_active` must be excluded from features until temporal relationship with churn is formally confirmed
- Manish's notebooks available — Karthik to share

---

### Customer Population (Confirmed)

| Segment | Total Accounts | Scoreable (active) |
|---|---|---|
| RC Direct | ~285,000 | ~170,000 |
| Pedigree Club | ~1,100,000 | ~650,000 |

- **Practical training window: 2022 onwards** — pre-2022 data has migration gaps
- Volume is manageable for weekly batch scoring

---

### Data Quality Findings

| Signal | Reliability | Notes |
|---|---|---|
| Email open rate | ✅ High | Pixel-tracked via SFMC |
| WhatsApp engagement | ⚠️ ~90% | Webhook reliability — directionally correct, not precise |
| Identity bridge | ⚠️ Partial | ~68% RC Direct, ~55% Pedigree Club — flag incomplete-coverage records in output |
| Capillary loyalty data | ⚠️ Fragile | No DQ checks, format drift risk |
| Freshdesk complaints | ⚠️ Manual | Production pipeline not built — must scope as DE work item |

---

### Confirmed Technical Decisions from W-02

| Decision | Detail |
|---|---|
| Training window | 2022–present |
| Feature aggregations | Source from existing dbt mart, refactor to incremental model |
| Primary model | LightGBM |
| Baseline | Logistic Regression |
| Subscription leakage | `subscription_active` excluded from features pending formal review |
| Freshdesk ingestion | Add to Phase 1 DE scope |
| Identity coverage | Flag incomplete-bridge records in scoring output |

---

## W-03 — IT / Infrastructure (Rahul Mehta)

### Snowflake Environment

| Parameter | Value |
|---|---|
| Edition | Enterprise |
| Region | AWS ap-south-1 (Mumbai) |
| Non-prod account | MARS_DEV + MARS_STG schemas |
| Production account | Separate — MARS_PROD |
| PII in non-prod | Tokenised IDs, masked email/phone — same as sample |
| Mu Sigma access | Service account — write to MU_SIGMA_DEV schema in MARS_DEV |
| Warehouse for Mu Sigma | X-Small (dev). Scale-up via change request — not auto-provisioned |

---

### Airflow (MWAA)

| Parameter | Value |
|---|---|
| Type | MWAA (Amazon Managed) |
| Version | 2.8.1 |
| Python | 3.11 |
| Packages available | pandas, numpy, scikit-learn ✅ |
| LightGBM | ❌ Not installed — must submit package request (3–5 days turnaround) |
| Dev DAG deployment | Karthik's team has access |
| Production deployment | ServiceNow change request + architecture board + 1-week UAT = **2–4 weeks** |

---

### DPA Status

| Item | Detail |
|---|---|
| Current status | Under review |
| Clauses under review | 1. Data residency (India-region required) 2. Sub-processor clause |
| Sub-processor resolution | All processing in Mars Snowflake — no external infra ✅ Simplifies clause |
| Estimated completion | 3–4 weeks from session date |
| Data access estimate | Early May (safe: first week of May) |
| **Implication** | **April = design + specification only. No pipeline code against live data.** |

---

### Key Constraints

| Constraint | Detail | Impact |
|---|---|---|
| `vet_referred` field | Under InfoSec health data classification review. Expected ruling 2–3 weeks. | Design as optional feature — do NOT make load-bearing |
| Production change freeze | Oct 1 – Nov 15 annually | Phase 2 must be in production before Oct 1 |
| Production deployment cycle | 2–4 weeks from code complete | **Must reach code-complete by ~June 1 for June 30 deadline** |
| Service account provisioning | 2-week SLA | Flag early after DPA signed |

---

## Consolidated Open Items

| ID | Item | Priority | Owner | Required By |
|---|---|---|---|---|
| OI-W-01 | **Formal churn definition** — cross-functional alignment (CRM=45d, Marketing=60d, Finance=120d) | 🔴 Critical | Harshita + Mars | Before model training |
| OI-W-02 | Salesforce integration spec — Phase 1 in/out of scope | 🔴 High | Harshita + Mars | Before Phase 1 architecture |
| OI-W-03 | DPA completion | 🔴 Critical | Mars Legal | Target: late April |
| OI-W-04 | LightGBM package request — submit as soon as DPA signed | 🟡 Medium | Savikar | After DPA |
| OI-W-05 | Service account provisioning — flag early | 🟡 Medium | Savikar + Rahul | After DPA |
| OI-W-06 | `vet_referred` classification ruling | 🟡 Medium | Mars InfoSec | 2–3 weeks |
| OI-W-07 | Karthik to share prior XGBoost notebooks | 🟡 Medium | Karthik | This week |
| OI-W-08 | `subscription_active` feature vs label decision | 🔴 Critical | Harshita + Vishwavani | Before model training |
| OI-W-09 | Freshdesk production ingestion pipeline — DE scope item | 🟡 Medium | Savikar | Phase 1 DE backlog |
| OI-W-10 | Deepa to be included in UAT planning | 🟢 Low | Harshita + Priya | UAT phase |

---

*Source: Confirmed session transcripts from W-01 (Priya Sharma), W-02 (Karthik Nair), W-03 (Rahul Mehta) — conducted 2026-03-31.
No content invented or assumed. All findings attributed to specific session.*
