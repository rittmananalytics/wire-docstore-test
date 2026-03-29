# Release Brief: Core Dynamics Data Platform Build

**Prepared by:** Wire Autopilot
**Date:** 2026-03-29
**Engagement:** data_platform_build
**Client:** Core Dynamics, Inc.
**Engagement Reference:** RA-2026-0041

---

## 1. Executive Summary

This release brief formally authorises and defines the scope of the Core Dynamics data platform engagement delivered by Rittman Analytics. Over 22 weeks, Rittman Analytics will design, build, and hand over a production-grade Google Cloud data stack (BigQuery, Fivetran, Dataform, Cloud Composer, Looker) serving four analytics workstreams — Marketing, Product, Customer, and Operations — underpinned by a governed Looker semantic layer. The engagement is structured as six downstream delivery releases following this discovery sprint, with a total value of $250,000.

The primary outcome is enabling Core Dynamics to progress toward its board-mandated NRR target of 115% within 18 months, through data-driven customer success, marketing optimisation, and product-led growth insights.

---

## 2. Problem Being Solved

Core Dynamics cannot make confident, data-driven decisions because:
- Revenue metrics (MRR, NRR, CAC) differ by up to 8% between business functions
- Customer health is assessed reactively using superficial ChurnZero scores that ignore product usage, support history, and financial signals
- Marketing attribution is last-touch only — paid social channels are invisible in pipeline attribution
- Product usage data is inaccessible to non-engineers
- Operational metrics (SLA, infrastructure cost, incident response) are tracked in Google Sheets with weekly update cycles

The full problem framing is documented in `.wire/releases/01-discovery/planning/problem_definition.md`.

---

## 3. Deliverables

| ID | Deliverable | Workstream | Phase | Acceptance Criteria |
|---|---|---|---|---|
| D-01 | GCP project setup, IAM, and environment configuration | Foundation | 1 | GCP projects provisioned; IAM roles configured; required APIs enabled |
| D-02 | Fivetran connector configuration (all 18 sources) | Foundation | 1–2 | All connectors sync with < 0.1% error rate over 7-day observation period |
| D-03 | Dataform repository — staging models (all sources) | Foundation | 2 | All staging models pass not-null, uniqueness, and referential integrity assertions |
| D-04 | Intermediate and warehouse models (all workstreams) | All | 2–3 | Row counts within ±1% of source; agreed business metrics within ±2% of Finance spreadsheets |
| D-05 | Cloud Composer DAGs for orchestration | Foundation | 2 | DAGs complete successfully; Slack/email alerts fire on simulated failure |
| D-06 | Data quality assertions + pipeline health dashboard | Foundation | 2 | Assertions pass for 3 consecutive daily runs; pipeline health dashboard live in Looker |
| D-07 | Dashboard MA-01: Demand Generation Performance | Marketing | 3 | Rachel Summers + Owen Brady sign off in UAT |
| D-08 | Dashboard MA-02: Multi-Touch Attribution Analysis | Marketing | 3 | 5 attribution models available; U-shaped default; Rachel Summers sign off |
| D-09 | Dashboard MA-03: Pipeline Contribution Report | Marketing | 3 | Marketing-sourced vs. influenced pipeline split; executive sign off |
| D-10 | Dashboard PB-01: Product Adoption Overview | Product | 3 | Feature hierarchy (Module/FeatureGroup/Feature) filterable; Claire Ashworth sign off |
| D-11 | Dashboard PB-02: Account-Level Product Usage | Product | 3 | Per-account drill-through with product engagement score trend; CS sign off |
| D-12 | Dashboard PB-03: Onboarding Funnel Analysis | Product | 3 | 8 onboarding steps visible; Julia Mercer + Tara Obinna sign off |
| D-13 | BigQuery ML Churn Risk Model | Customer | 3 | AUC-ROC ≥ 0.75; precision ≥ 60% at 0.5 threshold; model card delivered |
| D-14 | Dashboard CA-01: Customer Health Command Centre | Customer | 3 | Top 3 risk signals per account visible; Tara Obinna + Ben Tran sign off |
| D-15 | Dashboard CA-02: CSM Book of Business (with RLS) | Customer | 3 | Row-level security by CSM; all CSMs sign off in UAT |
| D-16 | Dashboard CA-03: CS Leadership Scorecard | Customer | 3 | NRR/GRR trend vs. target; Tara Obinna + David Park sign off |
| D-17 | Looker Scheduled Alerts (churn risk, health drop) | Customer | 3 | 4 alert types configured; test alerts fire successfully via Slack |
| D-18 | Dashboard DO-01: Support Operations SLA Tracker | Operations | 3 | 4-hour refresh; P1/P2/P3 SLA compliance visible; Carlos Vega + Harriet Drummond sign off |
| D-19 | Dashboard DO-02: Infrastructure Cost & Efficiency | Operations | 3 | Cost per customer visible; GCP spend vs. budget gauge; Sean Murphy sign off |
| D-20 | Dashboard DO-03: Incident Response & On-Call Health | Operations | 3 | MTTA/MTTR trend; on-call burden per engineer; Tobias Hecht sign off |
| D-21 | LookML project with governed semantic layer | All | 2–3 | All governed metrics defined; Amara Diallo + workstream leads sign off on Metric Definitions document |
| D-22 | Looker Actions (Salesforce task creation, QBR export) | Customer | 3 | "Flag for CSM Follow-up" creates Salesforce task; "Export QBR Data" generates Google Slides |
| D-23 | Data Engineering Runbook | Foundation | 3 | Covers architecture, add-new-source process, dev→prod promotion, incident response |
| D-24 | Semantic Layer Governance Guide | All | 3 | Covers LookML project structure, metric definitions, change governance process |
| D-25 | Knowledge transfer sessions (×3) | All | 3 | 3 sessions delivered; Amara Diallo + team sign off |
| D-26 | UAT sign-off documentation | All | 3 | All workstream sponsors sign UAT documents |

---

## 4. Downstream Releases

The following delivery releases will be executed after this discovery sprint:

| Release Name | Type | Scope Summary | Dependencies | Priority |
|---|---|---|---|---|
| 02-foundation | pipeline_only | GCP setup, all 18 Fivetran connectors, Dataform staging models, Cloud Composer DAGs, data quality framework | None | 1 |
| 03-marketing-analytics | full_platform | Marketing warehouse models, multi-touch attribution (5 models), LookML marketing views/explores, 3 dashboards (MA-01, MA-02, MA-03) | 02-foundation complete | 2 |
| 04-product-analytics | full_platform | Product warehouse models, engagement score, onboarding funnel, LookML product views/explores, 3 dashboards (PB-01, PB-02, PB-03) | 02-foundation complete | 2 |
| 05-customer-analytics | full_platform | Customer 360 model, BigQuery ML churn risk, LookML customer views/explores, 3 dashboards + Looker Alerts + Looker Actions (CA-01 to CA-03, D-17, D-22) | 04-product-analytics complete | 3 |
| 06-operational-analytics | dbt_development | Operations warehouse models, LookML operations views/explores, 3 dashboards (DO-01, DO-02, DO-03) | 02-foundation complete | 4 |
| 07-semantic-layer-governance | dashboard_extension | Semantic layer audit, executive model (LTV:CAC), governance review, final documentation (D-23, D-24), 3 knowledge transfer sessions (D-25), UAT sign-off (D-26) | All upstream releases | 5 |

---

## 5. Out of Scope

- Real-time streaming dashboards (sub-minute latency)
- Data catalogue (Dataplex, Alation)
- Predictive modelling beyond the churn risk model (expansion propensity, LTV prediction, lead scoring)
- Source system configuration changes (Salesforce, HubSpot, Zendesk)
- GDPR/CCPA compliance review or legal sign-off
- Mobile or embedded analytics
- Ongoing managed service post-engagement

---

## 6. Timeline

| Milestone | Description | Week | Amount |
|---|---|---|---|
| M1 | Contract signature | — | $42,500 |
| M2 | Phase 1 complete — all raw data in BigQuery | End Wk 4 | $42,500 |
| M3 | Phase 2 complete — all warehouse models signed off | End Wk 14 | $55,000 |
| M4 | Phase 3 — Marketing & Product dashboards signed off (D-07 to D-12) | End Wk 17 | $42,500 |
| M5 | Phase 3 — Customer & Operations dashboards signed off (D-13 to D-20) | End Wk 20 | $42,500 |
| M6 | Final sign-off, documentation, and knowledge transfer | End Wk 22 | $25,000 |
| **Total** | | | **$250,000** |

**Estimated duration:** 22 weeks from kickoff (2 February 2026).
**Estimated completion:** Week of 5 July 2026.

---

## 7. Budget

**Total engagement value:** $250,000 USD (fixed price)
**Standard day rate (out-of-scope work):** $2,200/day
**Expenses:** Travel billed at cost + 10% handling fee; prior written approval required
**Payment terms:** Net-30 from invoice date on milestone completion

---

## 8. Assumptions and Dependencies

1. GCP Owner/Editor access provisioned by end of Week 1 (Amara Diallo, pre-approved by Priya Nair)
2. Fivetran Business Critical account with sufficient MAR procured by Core Dynamics
3. Looker Standard/Enterprise licence with 30+ Standard + 5 Developer user seats procured by Core Dynamics
4. All API credentials and OAuth tokens provided within 5 business days of written request
5. CoreFM PostgreSQL network access (Cloud SQL Auth Proxy) provisioned before Phase 2 start
6. Workstream sponsors available for 1-hour requirements review (Week 1) and 2-hour UAT session (Phase 3)
7. Minimum 24 months historical renewal data available; Salesforce opportunity data will supplement ChurnZero history
8. Rittman Analytics surfaces data quality issues but does not cleanse upstream source data
9. Salesforce-HubSpot bidirectional sync functioning with consistent Contact.Id fields

---

## 9. Risks

| ID | Risk | Impact | Mitigation |
|---|---|---|---|
| R-01 | Salesforce-HubSpot 12% contact mismatch reduces attribution coverage | High | Fuzzy matching at staging layer; document coverage gap |
| R-03 | ChurnZero only 14 months history (vs. 24 months required for BQML) | High | Salesforce opportunity data as proxy; model card documents limitation |
| R-05 | GCP Billing label coverage unknown — may limit cost-per-customer dashboard | High | Sean Murphy to provide label audit; scope DO-02 appropriately |
| R-07 | CoreFM PostgreSQL network access not yet provisioned | High | Kofi + Sean to agree Cloud SQL Auth Proxy config; required before Phase 2 |
| R-08 | NetSuite integration role creation can take 2–3 weeks | Medium | Process started immediately; Priya Nair actioned |
| R-10 | PII pseudonymisation requires legal sign-off before implementation | Medium | Diane Hooper introduced; target sign-off by end of Week 2 |

---

## 10. Governance

**Engagement Lead (RA):** Mark Rittman
**Primary Technical Contact (client):** Amara Diallo
**Executive Sponsor (client):** Priya Nair

**Metric Definitions sign-off:** Each workstream sponsor signs off on the metric definitions affecting their domain before those models are promoted to the warehouse layer.

**Change management:** Any work outside this SoW requires written approval before proceeding. Rittman Analytics will raise change requests promptly. Standard day rate: $2,200/day.

**Status reporting:** Weekly status report circulated by Daniel Osei every Monday.

---

## 11. Acceptance Criteria Summary

| Deliverable Category | Criterion |
|---|---|
| Data Pipelines (D-02 to D-05) | < 0.1% Fivetran error rate over 7 days; Dataform assertions pass 3 consecutive runs; Cloud Composer DAGs complete successfully |
| Warehouse Models (D-04) | PK uniqueness and not-null tests pass; row counts ±1% of source; key metrics ±2% of Finance spreadsheets |
| BQML Churn Model (D-13) | AUC-ROC ≥ 0.75; precision ≥ 60% at 0.5 threshold; model card reviewed and signed off by Tara Obinna |
| Dashboards (D-07 to D-20) | All tiles load within 10 seconds; data matches agreed metric definitions; workstream sponsor UAT sign-off |
| LookML Semantic Layer (D-21) | Content validation passes; all governed metrics return consistent values across dashboards |
| Documentation (D-23, D-24) | Reviewed and accepted by Amara Diallo |
| Knowledge Transfer (D-25) | 3 sessions delivered; participant feedback collected |

---

## 12. Sign-off

| Role | Name | Signature | Date |
|---|---|---|---|
| Engagement Lead (RA) | Mark Rittman | [Signature required before sprint plan] | — |
| Executive Sponsor (client) | Priya Nair | [Signature required before sprint plan] | — |
| Primary Technical Contact (client) | Amara Diallo | [Signature required before sprint plan] | — |

---

*Document status: Generated by Wire Autopilot — self-reviewed and approved*
*Reviewed by: Wire Autopilot (self-review)*
*Date: 2026-03-29*
