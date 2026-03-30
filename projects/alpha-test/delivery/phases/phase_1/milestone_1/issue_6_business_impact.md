# Issue #6 — Business Impact Quantification (Time, Cost, Risk)
**Project:** alpha-test — Churn Prediction Pipeline (Airflow + Snowflake + Python)
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Completed:** 2026-03-30

---

## 1. Current State Cost — Manual Process

### 1.1 Analyst Time Cost

| Activity | Hours/Month | FTE Band | Est. Cost/Hour (USD) | Monthly Cost |
|---|---|---|---|---|
| CRM + Billing + S3 data extraction | 5 hrs | TDS2/TDS3 | $35 | $175 |
| Rule-based scoring + list generation | 3 hrs | TDS2 | $30 | $90 |
| Monthly churn report build | 3 hrs | TDS3 | $35 | $105 |
| Retention team CRM task creation | 16 hrs (2 days) | Account Manager | $40 | $640 |
| Ad hoc data requests from BU Head | 3 hrs | TDS3 | $35 | $105 |
| **Total** | **30 hrs/month** | | | **$1,115/month** |

**Annual analyst cost of current process: ~$13,380**

---

### 1.2 Revenue at Risk — Churn Leakage

*Note: Exact revenue figures to be confirmed with Business stakeholders during BRD sign-off. The below uses industry benchmarks for a mid-sized B2B/B2C operation — to be replaced with actuals.*

| Metric | Assumption | Value |
|---|---|---|
| Total active customers | From CRM — confirm at BRD | ~50,000 |
| Current monthly churn rate | To be confirmed — benchmark: 2–3% | ~2.5% |
| Customers churning per month | 50,000 × 2.5% | ~1,250 |
| Average annual revenue per customer | To confirm — benchmark: $500 | ~$500 |
| Monthly revenue lost to churn | 1,250 × ($500/12) | ~$52,000 |
| **Annual revenue lost to churn** | | **~$625,000** |

**If churn prediction pipeline reduces churn rate by 10%:**
→ ~125 customers retained per month
→ **~$62,500 annual revenue protected**

**If churn rate reduced by 15%:**
→ ~187 customers retained per month
→ **~$93,750 annual revenue protected**

---

### 1.3 Cost of False Positives (Current Rules-Based System)

| Metric | Estimate |
|---|---|
| Customers flagged as at-risk per month | ~3,000 (6% of base) |
| Estimated false positive rate (unvalidated rules) | ~40% |
| False positives per month | ~1,200 |
| Cost of unnecessary outreach per false positive | $15 (agent time + communication cost) |
| **Monthly cost of false positives** | **~$18,000** |
| **Annual wasted retention spend** | **~$216,000** |

**By reducing false positive rate from 40% to <10% (pipeline target):**
→ ~900 fewer unnecessary outreaches/month
→ **~$162,000 annual savings in wasted retention spend**

---

## 2. Pipeline Investment vs. Return

### 2.1 Estimated Build Cost (12-Week Pipeline)

| Phase | Effort | FTE Mix | Est. Cost |
|---|---|---|---|
| Phase 1: BRD + Requirements | 2 weeks | TDS3 × 2, TDS4 × 1 | ~$8,400 |
| Phase 5–7: Data Architecture + Modeling | 3 weeks | TDS3 × 3, TDS2 × 2 | ~$15,750 |
| Phase 9: Pipeline Development | 3 weeks | TDS3 × 3, TDS2 × 2 | ~$15,750 |
| Phase 10: QA + Validation | 2 weeks | TDS3 × 2, TDS2 × 2 | ~$8,400 |
| Phase 12: Deployment | 1 week | TDS3 × 2, TDS4 × 1 | ~$4,200 |
| Infra (Snowflake + Airflow setup) | Ongoing | IT | ~$5,000 |
| **Total Build Investment** | **12 weeks** | | **~$57,500** |

### 2.2 ROI Summary

| Metric | Annual Value |
|---|---|
| Revenue protected (10% churn reduction) | +$62,500 |
| Retention spend savings (false positive reduction) | +$162,000 |
| Analyst time savings (automation) | +$13,380 |
| **Total Annual Benefit** | **~$237,880** |
| Build Investment | ~$57,500 |
| **Payback Period** | **< 3 months** |
| **Year 1 Net ROI** | **~314%** |

---

## 3. Risk Quantification

| Risk | Likelihood | Impact | Estimated Cost if Realized |
|---|---|---|---|
| R-01: Data quality issues delay pipeline by 4+ weeks | Medium | High | +$19,250 in additional build cost |
| R-02: Snowflake cost overrun (daily scoring demand) | Low | Medium | +$15,000–30,000/year in compute |
| R-03: ML model underperforms (<60% recall) at launch | Low | High | Loss of confidence; 4-week re-tuning cycle (~$14,000) |
| R-04: CRM API rate limits block real-time ingestion | Medium | Medium | Requires batch workaround — 2-week delay |
| R-05: IT change management delays production deploy | Medium | Low | 2–4 week delay; no direct cost increase |
| R-06: Churn definition ambiguity (account vs contact level) | High | High | Rework of feature engineering — 3-week delay (~$11,250) |

**Highest priority risk to mitigate:** R-06 — Define "churn" and "customer" entity clearly in BRD before any data modeling begins.

---

## 4. Key Business Impact Summary

| Dimension | Current State | With Pipeline | Delta |
|---|---|---|---|
| Scoring frequency | Monthly | Weekly | 4× faster intervention |
| Analyst hours/month on scoring | 30 hrs | ~5 hrs (oversight only) | **-83%** |
| False positive rate | ~40% | Target < 10% | **-75%** |
| Revenue protected/year | Baseline | +$62,500–$93,750 | **+10–15% churn reduction** |
| Retention spend efficiency | ~60% | Target ~90% | **+30% improvement** |
| Reporting latency | Monthly (1–3 days lag) | Weekly automated | **Real-time visibility** |

---

_Deliverable generated as part of Phase 1 / Milestone 1 — BRD execution loop._
