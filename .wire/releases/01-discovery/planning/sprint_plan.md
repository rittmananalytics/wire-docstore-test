# Sprint Plan: Core Dynamics Data Platform Build

**Prepared by:** Wire Autopilot
**Date:** 2026-03-29
**Engagement:** data_platform_build
**Client:** Core Dynamics, Inc.
**Appetite:** Big batch — 22 weeks

---

## Overview

**Velocity assumption:** 5 story points per consultant-day
**Buffer:** 20% capacity buffer applied to all sprint estimates
**Sprint length:** 1 week per sprint
**Total sprints:** 22 (across all releases including discovery)
**Team composition:** Engagement Lead (Mark Rittman), Senior Analytics Engineer (Sophie Tanner), Data Engineer (Kofi Asante), Looker Developer (Irina Volkov), PM (Daniel Osei)

---

## Discovery Sprint (Weeks 1–2)

### Epic D-01: Project Kickoff and Requirements Validation

| Story | Points | Sprint | Owner |
|---|---|---|---|
| D-01-01: Project kickoff call — introductions, agenda, immediate actions | 1 | 1 | Daniel Osei |
| D-01-02: Workstream discovery sessions — Rachel Summers (Marketing) | 2 | 1 | Sophie Tanner |
| D-01-03: Workstream discovery sessions — Tara Obinna (Customer Success) | 2 | 1 | Mark Rittman |
| D-01-04: Workstream discovery sessions — Claire Ashworth (Product) | 2 | 1 | Sophie Tanner |
| D-01-05: Workstream discovery sessions — Harriet Drummond / Carlos Vega (Operations) | 2 | 1 | Mark Rittman |
| D-01-06: Data source audit — template distribution and collection | 2 | 1 | Kofi Asante |
| D-01-07: Problem definition document — generate, validate, approve | 3 | 2 | Mark Rittman |
| D-01-08: Pitch document — generate, validate, approve | 3 | 2 | Mark Rittman |
| D-01-09: Release brief — generate, validate, approve | 3 | 2 | Sophie Tanner |
| D-01-10: Sprint plan — generate, validate, approve | 3 | 2 | Sophie Tanner |

**Epic total: 23 points | Sprints 1–2**

---

## Release 02 — Foundation (Weeks 3–7)

### Epic F-01: GCP Infrastructure Setup

| Story | Points | Sprint | Owner |
|---|---|---|---|
| F-01-01: Provision GCP analytics-prod and analytics-dev projects | 3 | 3 | Kofi Asante |
| F-01-02: Configure IAM roles — data-engineers, analysts, looker-sa, dataform-runner | 3 | 3 | Kofi Asante |
| F-01-03: Enable all required APIs (BigQuery, Dataform, Cloud Composer, Pub/Sub, Secret Manager, Cloud Logging) | 1 | 3 | Kofi Asante |
| F-01-04: BigQuery dataset architecture — create all raw, staging, intermediate, warehouse datasets | 2 | 3 | Kofi Asante |

**Epic total: 9 points | Sprint 3**

### Epic F-02: Fivetran Pipeline Build

| Story | Points | Sprint | Owner |
|---|---|---|---|
| F-02-01: Configure Fivetran — Salesforce connector (6hr sync) | 3 | 3 | Kofi Asante |
| F-02-02: Configure Fivetran — HubSpot connector (1hr sync) | 3 | 3 | Kofi Asante |
| F-02-03: Configure Fivetran — Google Ads, LinkedIn Ads, Meta Ads connectors (daily) | 3 | 3 | Kofi Asante |
| F-02-04: Configure Fivetran — Outreach.io connector (6hr sync) | 2 | 4 | Kofi Asante |
| F-02-05: Configure Fivetran — CoreFM PostgreSQL CDC connector (15min sync) + Cloud SQL Auth Proxy setup | 5 | 4 | Kofi Asante |
| F-02-06: Configure Fivetran — Mixpanel connector (daily) | 2 | 4 | Kofi Asante |
| F-02-07: Configure Segment BigQuery Destination (streaming) | 2 | 4 | Kofi Asante |
| F-02-08: Configure Fivetran — Intercom, ChurnZero API Cloud Function, Zendesk, NetSuite connectors | 5 | 4 | Kofi Asante |
| F-02-09: Configure Fivetran — PagerDuty, GCP Billing Export, Google Cloud Monitoring, BambooHR | 3 | 5 | Kofi Asante |
| F-02-10: Build Clearbit Cloud Function (REST API → BigQuery, triggered on inbound lead) | 5 | 5 | Kofi Asante |
| F-02-11: Build ChurnZero Cloud Function (incremental REST API → BigQuery, daily 6am UTC) | 5 | 5 | Kofi Asante |
| F-02-12: Connector validation — 7-day observation period, < 0.1% error rate verification | 3 | 6 | Kofi Asante |

**Epic total: 41 points | Sprints 3–6**

### Epic F-03: Dataform Staging Models

| Story | Points | Sprint | Owner |
|---|---|---|---|
| F-03-01: Initialise Dataform repository — GitHub connection, project structure | 2 | 3 | Sophie Tanner |
| F-03-02: Staging models — Salesforce (accounts, contacts, opportunities, campaigns, contracts) | 5 | 4 | Sophie Tanner |
| F-03-03: Staging models — HubSpot (contacts, companies, form submissions, email sends, page views, UTM params) | 5 | 4 | Sophie Tanner |
| F-03-04: Staging models — Google Ads, LinkedIn Ads, Meta Ads | 5 | 5 | Sophie Tanner |
| F-03-05: Staging models — Outreach.io, Clearbit enrichment | 3 | 5 | Sophie Tanner |
| F-03-06: Staging models — CoreFM PostgreSQL (accounts, users, sessions, feature events, work orders, assets) + PII pseudonymisation | 8 | 5 | Sophie Tanner |
| F-03-07: Staging models — Mixpanel, Segment (with deduplication rules) | 5 | 5 | Sophie Tanner |
| F-03-08: Staging models — Intercom, ChurnZero, Zendesk | 5 | 6 | Sophie Tanner |
| F-03-09: Staging models — NetSuite, PagerDuty, GCP Billing, Google Cloud Monitoring, BambooHR | 5 | 6 | Sophie Tanner |
| F-03-10: Load feature taxonomy seed data (Leon Yip CSV → raw_corefm_db.feature_taxonomy) | 2 | 6 | Sophie Tanner |
| F-03-11: Shared dimensions — dim_date (with fiscal calendar Feb start), dim_account, dim_csm | 5 | 6 | Sophie Tanner |

**Epic total: 50 points | Sprints 3–6**

### Epic F-04: Cloud Composer Orchestration

| Story | Points | Sprint | Owner |
|---|---|---|---|
| F-04-01: Deploy Cloud Composer 2 environment | 3 | 6 | Kofi Asante |
| F-04-02: DAG — trigger Fivetran syncs in dependency order | 5 | 6 | Kofi Asante |
| F-04-03: DAG — run Dataform pipelines post-sync (nightly full refresh + intraday incremental) | 5 | 7 | Kofi Asante |
| F-04-04: DAG — data quality checks post-transformation | 3 | 7 | Kofi Asante |
| F-04-05: DAG — alerting on pipeline failures via Slack/email | 3 | 7 | Kofi Asante |

**Epic total: 19 points | Sprints 6–7**

### Epic F-05: Data Quality & Pipeline Health Dashboard

| Story | Points | Sprint | Owner |
|---|---|---|---|
| F-05-01: Dataform assertions — row counts, null rates, referential integrity, business logic sanity checks | 5 | 7 | Sophie Tanner |
| F-05-02: BigQuery audit log export to Cloud Logging | 2 | 7 | Kofi Asante |
| F-05-03: Pipeline health dashboard in Looker (run times, failure rates, row count trends) | 3 | 7 | Irina Volkov |

**Epic total: 10 points | Sprint 7**

**Release 02 total: 129 points | Sprints 3–7**

---

## Release 03 — Marketing Analytics (Weeks 8–12)

### Epic MA-01: Marketing Requirements and Data Model Design

| Story | Points | Sprint | Owner |
|---|---|---|---|
| MA-01-01: Marketing requirements specification and stakeholder sign-off | 3 | 8 | Sophie Tanner |
| MA-01-02: Marketing conceptual model — Campaign, Lead, Contact, Opportunity, Touch entities | 3 | 8 | Sophie Tanner |
| MA-01-03: Marketing pipeline design — source-to-warehouse flow for all marketing sources | 3 | 8 | Kofi Asante |
| MA-01-04: Marketing data model specification — all 6 mart_marketing models | 5 | 8 | Sophie Tanner |
| MA-01-05: Marketing mockups — MA-01, MA-02, MA-03 dashboard wireframes | 3 | 8 | Irina Volkov |

**Epic total: 17 points | Sprint 8**

### Epic MA-02: Marketing dbt Models

| Story | Points | Sprint | Owner |
|---|---|---|---|
| MA-02-01: `mart_marketing.fct_lead_funnel_events` — lead per stage transition, SAL stage included | 8 | 9 | Sophie Tanner |
| MA-02-02: `mart_marketing.fct_campaign_spend` — daily spend by channel, campaign, ad group | 5 | 9 | Sophie Tanner |
| MA-02-03: `mart_marketing.fct_attribution_touches` — all marketing touches per contact with type and channel | 8 | 9 | Sophie Tanner |
| MA-02-04: `mart_marketing.fct_opportunity_attribution` — 5 attribution models (U-shaped default), 180-day influenced window | 8 | 10 | Sophie Tanner |
| MA-02-05: `mart_marketing.dim_campaign` — channel, type, target segment | 3 | 9 | Sophie Tanner |
| MA-02-06: `mart_marketing.dim_contact` — with Clearbit firmographic enrichment | 3 | 9 | Sophie Tanner |
| MA-02-07: dbt schema.yml, tests, and documentation for all marketing models | 3 | 10 | Sophie Tanner |

**Epic total: 38 points | Sprints 9–10**

### Epic MA-03: Marketing Semantic Layer and Dashboards

| Story | Points | Sprint | Owner |
|---|---|---|---|
| MA-03-01: LookML — marketing views (campaign, lead, attribution touches, opportunity attribution, contact) | 5 | 10 | Irina Volkov |
| MA-03-02: LookML — marketing explores with joins; governed CAC, pipeline-to-spend metrics | 3 | 10 | Irina Volkov |
| MA-03-03: Dashboard MA-01: Demand Generation Performance (weekly, daily refresh) | 5 | 11 | Irina Volkov |
| MA-03-04: Dashboard MA-02: Multi-Touch Attribution Analysis (model selector, U-shaped default) | 5 | 11 | Irina Volkov |
| MA-03-05: Dashboard MA-03: Pipeline Contribution Report (sourced vs. influenced, monthly executive) | 3 | 11 | Irina Volkov |
| MA-03-06: Marketing UAT — Rachel Summers, Owen Brady, Niamh Collins | 3 | 12 | Sophie Tanner |

**Epic total: 24 points | Sprints 10–12**

**Release 03 total: 79 points | Sprints 8–12**

---

## Release 04 — Product Analytics (Weeks 8–13)

### Epic PB-01: Product Requirements and Data Model Design

| Story | Points | Sprint | Owner |
|---|---|---|---|
| PB-01-01: Product requirements specification (incorporating supplementary requirements: feature taxonomy, licence utilisation, onboarding steps) | 3 | 8 | Sophie Tanner |
| PB-01-02: Product conceptual model — Account, User, Session, Feature, WorkOrder, OnboardingStep entities | 3 | 8 | Sophie Tanner |
| PB-01-03: Product data model specification — all 5 mart_product models | 5 | 9 | Sophie Tanner |
| PB-01-04: Product mockups — PB-01, PB-02, PB-03 dashboard wireframes | 3 | 8 | Irina Volkov |

**Epic total: 14 points | Sprints 8–9**

### Epic PB-02: Product dbt Models

| Story | Points | Sprint | Owner |
|---|---|---|---|
| PB-02-01: `mart_product.fct_feature_usage` — daily feature usage per account per feature (Module/FeatureGroup/Feature hierarchy) | 8 | 9 | Sophie Tanner |
| PB-02-02: `mart_product.fct_session_events` — individual session events from Segment/Mixpanel (deduplicated) | 5 | 9 | Sophie Tanner |
| PB-02-03: `mart_product.fct_onboarding_funnel` — 8 Intercom onboarding steps per account | 5 | 10 | Sophie Tanner |
| PB-02-04: `mart_product.dim_account_product_score` — weighted composite score (DAU/MAU 25%, distinct features 20%, core activation 25%, NPS 15%, onboarding 15%) + licence_utilisation_pct | 8 | 10 | Sophie Tanner |
| PB-02-05: `mart_product.fct_workflow_completion` — key workflow completion rates | 3 | 10 | Sophie Tanner |
| PB-02-06: dbt schema.yml, tests, and documentation for all product models | 3 | 11 | Sophie Tanner |

**Epic total: 32 points | Sprints 9–11**

### Epic PB-03: Product Semantic Layer and Dashboards

| Story | Points | Sprint | Owner |
|---|---|---|---|
| PB-03-01: LookML — product views (feature usage, session events, onboarding funnel, product score, workflow completion) | 5 | 11 | Irina Volkov |
| PB-03-02: LookML — product explores with account joins; Product Engagement Score governed metric | 3 | 11 | Irina Volkov |
| PB-03-03: Dashboard PB-01: Product Adoption Overview (heatmap, DAU/MAU, activation funnel) | 5 | 12 | Irina Volkov |
| PB-03-04: Dashboard PB-02: Account-Level Product Usage (drill-through, engagement score trend) | 5 | 12 | Irina Volkov |
| PB-03-05: Dashboard PB-03: Onboarding Funnel Analysis (8-step funnel, stall detection, cohort analysis) | 5 | 12 | Irina Volkov |
| PB-03-06: Product UAT — Claire Ashworth, Leon Yip, Julia Mercer | 3 | 13 | Sophie Tanner |

**Epic total: 26 points | Sprints 11–13**

**Release 04 total: 72 points | Sprints 8–13**

---

## Release 05 — Customer Analytics (Weeks 13–19)

### Epic CA-01: Customer Requirements and Data Model Design

| Story | Points | Sprint | Owner |
|---|---|---|---|
| CA-01-01: Customer requirements specification — health model, expansion signals, churn model requirements | 3 | 13 | Sophie Tanner |
| CA-01-02: Customer conceptual model — Account, CSM, Renewal, HealthScore, ChurnRisk, Expansion entities | 3 | 13 | Sophie Tanner |
| CA-01-03: Customer data model specification — all 6 mart_customer models | 5 | 13 | Sophie Tanner |
| CA-01-04: Customer dashboard mockups — CA-01, CA-02 (with RLS), CA-03 | 3 | 13 | Irina Volkov |

**Epic total: 14 points | Sprint 13**

### Epic CA-02: Customer dbt Models

| Story | Points | Sprint | Owner |
|---|---|---|---|
| CA-02-01: `mart_customer.dim_account_health` — unified health score (product engagement, support, sentiment, financial signals, top-3 contributing signals) | 8 | 14 | Sophie Tanner |
| CA-02-02: `mart_customer.fct_renewal_pipeline` — renewals in rolling 180-day window | 5 | 14 | Sophie Tanner |
| CA-02-03: `mart_customer.fct_expansion_signals` — unused licences, module gaps, multi-site potential | 5 | 14 | Sophie Tanner |
| CA-02-04: `mart_customer.fct_csm_book_of_business` — per-CSM ARR, account count, health distribution, renewal coverage | 3 | 14 | Sophie Tanner |
| CA-02-05: `mart_customer.fct_nrr_grr` — monthly NRR/GRR by segment, cohort, CSM | 5 | 14 | Sophie Tanner |
| CA-02-06: `mart_customer.fct_churn_events` — historical churn/downsell events for BQML training | 3 | 14 | Sophie Tanner |
| CA-02-07: dbt schema.yml, tests, and documentation for all customer models | 3 | 15 | Sophie Tanner |

**Epic total: 32 points | Sprints 14–15**

### Epic CA-03: BigQuery ML Churn Risk Model

| Story | Points | Sprint | Owner |
|---|---|---|---|
| CA-03-01: Feature engineering — prepare training dataset from mart_customer.dim_account_health + fct_churn_events | 5 | 15 | Sophie Tanner |
| CA-03-02: Train BQML logistic regression model — 24-month historical window (ChurnZero 14mo + Salesforce proxy) | 5 | 15 | Sophie Tanner |
| CA-03-03: Evaluate model — AUC-ROC, precision/recall at 0.5 threshold, business-context trade-off analysis | 5 | 15 | Sophie Tanner |
| CA-03-04: Deploy model — `churn_probability` and `churn_risk_band` (Low/Medium/High/Critical) refreshed weekly | 3 | 16 | Sophie Tanner |
| CA-03-05: Model card — assumptions, limitations, retraining guidance; Tara Obinna sign-off | 3 | 16 | Sophie Tanner |

**Epic total: 21 points | Sprints 15–16**

### Epic CA-04: Customer Semantic Layer and Dashboards

| Story | Points | Sprint | Owner |
|---|---|---|---|
| CA-04-01: LookML — customer views (account health, renewal pipeline, expansion signals, CSM book, NRR/GRR, churn events) | 5 | 16 | Irina Volkov |
| CA-04-02: LookML — customer explores with row-level security (user attributes by CSM); NRR, GRR, Churn Risk governed metrics | 5 | 16 | Irina Volkov |
| CA-04-03: Dashboard CA-01: Customer Health Command Centre (health distribution, churn heatmap, renewals, top-10 at-risk, expansion list, NRR gauge) | 8 | 17 | Irina Volkov |
| CA-04-04: Dashboard CA-02: CSM Book of Business (RAG indicators, "accounts requiring action", per-CSM NRR, QBR prep button) | 5 | 17 | Irina Volkov |
| CA-04-05: Dashboard CA-03: CS Leadership Scorecard (NRR/GRR trend, churn by segment, CSM performance comparison, model performance metrics) | 5 | 17 | Irina Volkov |
| CA-04-06: Looker Scheduled Alerts — 4 alert types (health drop 15pts, "At Risk" entry, renewal 60 days < 70%, new Sev 1/2 on at-risk) via Slack | 5 | 18 | Irina Volkov |
| CA-04-07: Looker Action — "Flag for CSM Follow-up" → Salesforce task creation | 3 | 18 | Irina Volkov |
| CA-04-08: Looker Action — "Export QBR Data" → pre-populated Google Slides deck | 5 | 18 | Irina Volkov |
| CA-04-09: Customer UAT — Tara Obinna, Ben Tran, CSMs sample | 3 | 19 | Sophie Tanner |

**Epic total: 44 points | Sprints 16–19**

**Release 05 total: 111 points | Sprints 13–19**

---

## Release 06 — Operational Analytics (Weeks 14–19)

### Epic DO-01: Operations Requirements and Data Model Design

| Story | Points | Sprint | Owner |
|---|---|---|---|
| DO-01-01: Operations requirements specification | 3 | 14 | Sophie Tanner |
| DO-01-02: Operations data model specification — 4 mart_ops models | 3 | 14 | Sophie Tanner |
| DO-01-03: Operations dashboard mockups — DO-01, DO-02, DO-03 | 3 | 14 | Irina Volkov |

**Epic total: 9 points | Sprint 14**

### Epic DO-02: Operations dbt Models

| Story | Points | Sprint | Owner |
|---|---|---|---|
| DO-02-01: `mart_ops.fct_support_tickets` — SLA timers, resolution time, agent, customer tier, CSAT | 5 | 15 | Sophie Tanner |
| DO-02-02: `mart_ops.fct_incident_response` — PagerDuty incidents with MTTA, MTTR, escalation chain | 3 | 15 | Sophie Tanner |
| DO-02-03: `mart_ops.fct_infra_cost_by_customer` — daily GCP cost allocated by project/label mapping | 5 | 16 | Kofi Asante |
| DO-02-04: `mart_ops.fct_support_capacity` — ticket volume vs. headcount by team and week | 3 | 16 | Sophie Tanner |
| DO-02-05: dbt schema.yml, tests, and documentation for all operations models | 2 | 16 | Sophie Tanner |

**Epic total: 18 points | Sprints 15–16**

### Epic DO-03: Operations Semantic Layer and Dashboards

| Story | Points | Sprint | Owner |
|---|---|---|---|
| DO-03-01: LookML — operations views (support tickets, incident response, infra cost, support capacity) | 5 | 17 | Irina Volkov |
| DO-03-02: LookML — operations explores; SLA Compliance Rate governed metric | 3 | 17 | Irina Volkov |
| DO-03-03: Dashboard DO-01: Support Operations SLA Tracker (4-hour refresh, P1/P2/P3 compliance, live SLA breach list) | 5 | 18 | Irina Volkov |
| DO-03-04: Dashboard DO-02: Infrastructure Cost & Efficiency (GCP spend vs. budget, cost per customer, anomaly detection) | 5 | 18 | Irina Volkov |
| DO-03-05: Dashboard DO-03: Incident Response & On-Call Health (MTTA/MTTR trend, on-call burden, uptime by service) | 5 | 18 | Irina Volkov |
| DO-03-06: Operations UAT — Carlos Vega, Harriet Drummond, Sean Murphy | 3 | 19 | Sophie Tanner |

**Epic total: 26 points | Sprints 17–19**

**Release 06 total: 53 points | Sprints 14–19**

---

## Release 07 — Semantic Layer Governance & Handover (Weeks 20–22)

### Epic SL-01: Semantic Layer Governance Review

| Story | Points | Sprint | Owner |
|---|---|---|---|
| SL-01-01: Metric Definitions document — all 10 governed metrics with workstream sponsor sign-off | 5 | 20 | Sophie Tanner |
| SL-01-02: LookML governance review — content validation, access controls, PII field visibility | 3 | 20 | Irina Volkov |
| SL-01-03: Executive explore + LTV:CAC metric in executive.model.lkml | 3 | 20 | Irina Volkov |
| SL-01-04: Production deployment — promote all Looker content from dev to prod instance | 3 | 21 | Irina Volkov |
| SL-01-05: Row-level security audit — verify CSM and finance_viewers user group restrictions | 2 | 21 | Irina Volkov |

**Epic total: 16 points | Sprints 20–21**

### Epic SL-02: Documentation and Knowledge Transfer

| Story | Points | Sprint | Owner |
|---|---|---|---|
| SL-02-01: Data Engineering Runbook (D-23) — pipeline architecture, add-new-source, dev→prod promotion, incident response | 5 | 20 | Kofi Asante |
| SL-02-02: Semantic Layer Governance Guide (D-24) — LookML project structure, metric change governance | 5 | 20 | Irina Volkov |
| SL-02-03: Knowledge transfer session 1 — Data engineering (Amara Diallo's team), 2 hours | 3 | 21 | Kofi Asante |
| SL-02-04: Knowledge transfer session 2 — Looker development (analytics team), 2 hours | 3 | 21 | Irina Volkov |
| SL-02-05: Knowledge transfer session 3 — Dashboard usage and self-service (all stakeholders), 2 hours | 3 | 22 | Sophie Tanner |
| SL-02-06: Final UAT sign-off documentation (D-26) — all workstream sponsors | 3 | 22 | Daniel Osei |
| SL-02-07: Hypercare period setup — define escalation path and SLA for post-go-live support | 2 | 22 | Mark Rittman |

**Epic total: 24 points | Sprints 20–22**

**Release 07 total: 40 points | Sprints 20–22**

---

## Point Summary

| Release | Epics | Total Points | Sprints |
|---|---|---|---|
| Discovery | 1 | 23 | 1–2 |
| 02-foundation | 5 | 129 | 3–7 |
| 03-marketing-analytics | 3 | 79 | 8–12 |
| 04-product-analytics | 3 | 72 | 8–13 |
| 05-customer-analytics | 4 | 111 | 13–19 |
| 06-operational-analytics | 3 | 53 | 14–19 |
| 07-semantic-layer-governance | 2 | 40 | 20–22 |
| **Total** | **21** | **507** | **1–22** |

**Velocity check:** 507 points ÷ 22 weeks × 80% effective capacity = 18.5 points/week effective budget. This is achievable across a 5-person team at 5 points/consultant-day with the team working on parallel streams (Marketing/Product, Customer/Operations).

---

## Downstream Releases

| Release Name | Type | Scope Summary | Priority |
|---|---|---|---|
| 02-foundation | pipeline_only | GCP setup, Fivetran pipelines (all 18 sources), Dataform staging models, Cloud Composer orchestration, data quality framework | 1 |
| 03-marketing-analytics | full_platform | Marketing warehouse models (6 models), multi-touch attribution (5 models, U-shaped default, 180-day window), LookML marketing views/explores, 3 dashboards (MA-01, MA-02, MA-03) | 2 |
| 04-product-analytics | full_platform | Product warehouse models (5 models, feature hierarchy, engagement score, licence utilisation), LookML product views/explores, 3 dashboards (PB-01, PB-02, PB-03) | 2 |
| 05-customer-analytics | full_platform | Customer 360 model (6 models), BigQuery ML churn risk model, LookML customer views/explores with RLS, 3 dashboards + 4 Looker Alerts + 2 Looker Actions (CA-01 to CA-03, D-17, D-22) | 3 |
| 06-operational-analytics | dbt_development | Operations warehouse models (4 models), LookML operations views/explores, 3 dashboards (DO-01, DO-02, DO-03) | 4 |
| 07-semantic-layer-governance | dashboard_extension | Semantic layer audit, governed metric sign-off, executive model, production deployment, Data Engineering Runbook, Semantic Layer Governance Guide, 3 knowledge transfer sessions, final UAT sign-off | 5 |

---

*Document status: Generated by Wire Autopilot — self-reviewed and approved*
*Reviewed by: Wire Autopilot (self-review)*
*Date: 2026-03-29*
