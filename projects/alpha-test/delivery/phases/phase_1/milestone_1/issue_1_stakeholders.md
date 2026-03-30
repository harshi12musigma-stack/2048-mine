# Issue #1 — Identify Stakeholders Using Org Charts
**Project:** alpha-test — Churn Prediction Pipeline (Airflow + Snowflake + Python)
**Phase:** 1 — Requirement Gathering & Business Understanding
**Milestone:** Business Requirement Document (BRD)
**Completed:** 2026-03-30

---

## 1. Stakeholder Map

### 1.1 Executive / BSH Sponsors (Final Decision Authority)

| Name | Designation | Location | Role in Project |
|---|---|---|---|
| Alokesh Das | Business Unit Head | Bangalore ITPL | Executive BSH Sponsor |
| Chandrasekar Balasubramanian | Business Unit Head | Bangalore ITPL | Executive BSH Sponsor |
| Rajdeep Roy Choudhury | Business Unit Head | Bangalore ITPL | Executive BSH Sponsor |
| Manoj Balachandran | Business Unit Head | Bangalore ITPL | Executive BSH Sponsor |
| Zubin Dowlaty | Head of Innovation & Development | Austin | Innovation Alignment |
| Tanmay Sengupta | Business Unit Head | Austin | Business Stakeholder |
| Richa Gupta | Business Unit Head | Austin | Business Stakeholder |
| Nivedhan Narasimhan | Business Unit Head | Bangalore ITPL | Business Stakeholder |
| Shilpa P Bhat | Business Unit Head | Bangalore ITPL | Business Stakeholder |
| L Sivagnana Selvam | Head of Finance | Bangalore ITPL | Finance / Budget Owner |
| Naseer Ahmed Naqash | Head - IT & Infosec | Bangalore ITPL | Infrastructure & Security |

---

### 1.2 Project Delivery Team (TDS Bands)

| Band | Role | Count | Project Responsibility |
|---|---|---|---|
| TDS4 | Project Leader | 359 | Project governance, milestone approvals, client liaison |
| TDS3 | Problem Solver | 458 | Pipeline development, ML model execution, QA lead |
| TDS2 | Analytical Builder | 288 | ETL build, data transformation, pipeline logic |
| TDS1 | Curious Explorer | 433 | Data profiling, EDA, documentation, stakeholder research |
| AL | Apprentice Leader | 204 | Cross-functional coordination, skill bridging |

---

### 1.3 Technical / Platform SMEs

| Name | Designation | Location | Role in Project |
|---|---|---|---|
| Naseer Ahmed Naqash | Head - IT & Infosec | Bangalore ITPL | Infrastructure, Snowflake/Airflow provisioning, security clearance |

---

### 1.4 Account Context (For Churn Domain Alignment)

The churn prediction pipeline is generalizable across the following Mu Sigma accounts with active churn/retention use cases:

| Account | Domain Relevance |
|---|---|
| Walmart | Retail churn, customer 360, loyalty |
| Home Depot | Customer retention, transaction scoring |
| Mars | Consumer brand loyalty, switching analysis |
| Microsoft | Subscription churn, SaaS renewal |
| Southwest | Passenger churn, loyalty program attrition |
| NCLH | Cruise booking drop-off, repeat customer prediction |
| Dell | B2B customer retention, renewal scoring |
| Abbvie / J&J / Gilead Sciences / Bristol-Myers Squibb | Patient therapy adherence, HCP churn |

**Primary alignment:** Retail / CPG and B2B subscription verticals are the most directly applicable to Airflow + Snowflake churn pipeline architecture.

---

## 2. RACI Matrix

| Activity | BSH Sponsor | TDS4 (Project Leader) | TDS3 (Problem Solver) | TDS2 (Analytical Builder) | TDS1 (Curious Explorer) | IT / Infosec |
|---|---|---|---|---|---|---|
| Requirements sign-off | A | R | C | I | I | I |
| Data access & provisioning | I | A | C | I | I | R |
| Architecture design | C | A | R | C | I | C |
| Pipeline development | I | A | C | R | C | I |
| ML model build & scoring | I | A | R | C | C | I |
| QA & validation | I | A | R | R | C | I |
| Deployment | C | A | R | C | I | R |
| KPI reporting & monitoring | A | C | R | C | I | I |

**R** = Responsible | **A** = Accountable | **C** = Consulted | **I** = Informed

---

## 3. Stakeholder Engagement Plan

| Stakeholder Group | Engagement Mode | Frequency | Owner |
|---|---|---|---|
| BSH Sponsors | Milestone review + approval sessions | Per milestone | TDS4 |
| TDS4 Project Leaders | Weekly stand-up + state sync | Weekly | TDS4 |
| TDS3 Problem Solvers | Daily execution sync | Daily | TDS3 |
| TDS2 Analytical Builders | Daily coding review | Daily | TDS3 |
| TDS1 Curious Explorers | Task assignment + output review | Daily | TDS2/TDS3 |
| IT / Infosec | Infrastructure provisioning sessions | As needed | TDS4 |
| Account SMEs (domain) | Knowledge transfer sessions | Per phase | TDS3 |

---

## 4. Key Contacts for BRD Sign-off

Based on org chart data and project scope:

1. **Primary approver:** `openclaw-control-ui` (configured project owner)
2. **Executive sponsor:** Business Unit Heads (Alokesh Das, Chandrasekar Balasubramanian)
3. **Technical approver:** Naseer Ahmed Naqash (IT/Infosec — Airflow + Snowflake environments)
4. **Delivery lead:** TDS4 band lead assigned to the project

---

## 5. Next Steps

- [ ] Schedule BRD kick-off meeting with identified BSH sponsors
- [ ] Distribute stakeholder questionnaires (Issue #2)
- [ ] Capture expectations and conflicts from each stakeholder group (Issue #3)
- [ ] Confirm Snowflake + Airflow environment access with IT/Infosec

---

_Deliverable generated as part of Phase 1 / Milestone 1 — BRD execution loop._
