# Sprint Plan — data_platform_build
## Core Dynamics, Inc.

**Prepared by:** Wire Autopilot
**Date:** 2026-03-29
**Engagement:** data_platform_build
**Client:** Core Dynamics, Inc.
**Engagement Reference:** RA-2026-0041

---

## 1. Planning Approach

The engagement is structured across 22 weeks in three phases. Weeks are the planning unit; sprints are 2 weeks each (11 sprints total). This sprint plan defines what is built in each sprint, who owns it, what the exit criteria are, and what the dependencies are.

Sprint velocity is fixed by team composition: Mark Rittman (engagement lead, part-time), Sophie Tanner (analytics engineer, full-time), Kofi Asante (data engineer, full-time), Irina Volkov (Looker developer, full-time from Week 5), Daniel Osei (PM, part-time). No additional resources are available within scope.

---

## 2. Sprint Summary

| Sprint | Weeks | Theme | Releases |
|---|---|---|---|
| S1 | 1–2 | Foundation setup + Discovery | 01-discovery, 02-foundation (start) |
| S2 | 3–4 | Foundation completion | 02-foundation |
| S3 | 5–6 | Marketing data modelling | 03-marketing-analytics |
| S4 | 7–8 | Marketing semantic layer + Product start | 03-marketing-analytics, 04-product-analytics |
| S5 | 9–10 | Marketing dashboards + Product modelling | 03-marketing-analytics, 04-product-analytics |
| S6 | 11–12 | Product semantic layer + Customer start | 04-product-analytics, 05-customer-analytics |
| S7 | 13–14 | Product dashboards + Customer modelling + Ops start | 04-product-analytics, 05-customer-analytics, 06-operational-analytics |
| S8 | 15–16 | Customer semantic layer + BQML + Ops modelling | 05-customer-analytics, 06-operational-analytics |
| S9 | 17–18 | Customer dashboards + Ops semantic layer + Governance start | 05-customer-analytics, 06-operational-analytics, 07-governance |
| S10 | 19–20 | Ops dashboards + UAT + KT sessions | 06-operational-analytics, 07-governance |
| S11 | 21–22 | UAT remediation + final deployment + KT sessions | 07-governance |

---

## 3. Sprint Detail

### Sprint 1 (Weeks 1–2): Foundation Setup + Discovery

**Goal:** Establish GCP infrastructure, provision all connectors, complete discovery sprint artifacts.

**Kofi Asante (Data Engineer):**
- Provision GCP project `core-dynamics-analytics-prod`; create all BigQuery datasets (raw_*, staging, integration, warehouse, monitoring)
- Configure IAM roles: data-engineers, analysts, looker-service-account, dataform-runner
- Configure Secret Manager; enable required GCP APIs (BigQuery, Composer, Pub/Sub, Cloud Functions, Cloud SQL, Logging)
- Set up Fivetran account; configure first 8 connectors: Salesforce, HubSpot, Google Ads, LinkedIn Ads, Meta Ads, Outreach.io, Zendesk, PagerDuty
- Begin CoreFM PostgreSQL Cloud SQL Auth Proxy setup (with Amara Diallo)
- Begin ChurnZero Cloud Function development

**Sophie Tanner (Sr Analytics Engineer):**
- Set up Dataform repository (connected to `feature/data_platform_build` branch)
- Build `stg_salesforce__accounts`, `stg_salesforce__contacts`, `stg_salesforce__opportunities`
- Build `stg_hubspot__contacts`, `stg_hubspot__companies`, `stg_hubspot__deals`, `stg_hubspot__engagements`
- Build `stg_zendesk__tickets`, `stg_zendesk__ticket_comments`, `stg_zendesk__users`
- Start `dim_date` shared dimension

**Mark Rittman (Engagement Lead):**
- Discovery sprint: generate, validate, and self-approve problem_definition, pitch, release_brief, sprint_plan
- Sync all discovery artifacts to Confluence WAEE and Notion
- Stakeholder kick-off (Priya Nair, Amara Diallo, workstream sponsors)
- Confirm open questions R-01, R-03, R-05, R-06, R-11 with relevant stakeholders
- Track GCP access, Fivetran licence, Looker licence provisioning

**Exit criteria:**
- GCP project live; all IAM roles assigned; BigQuery datasets exist
- First 8 Fivetran connectors active and syncing (even if rows are low)
- Salesforce, HubSpot, Zendesk staging models running in Dataform without errors
- All 4 discovery sprint artifacts approved and synced to Confluence + Notion

**Blocked if:** GCP Owner/Editor roles not provisioned by Day 3 of Week 1

---

### Sprint 2 (Weeks 3–4): Foundation Completion

**Goal:** All 18 source connectors active; all staging models built and assertion-clean; Cloud Composer DAG running.

**Kofi Asante:**
- Configure remaining 8 Fivetran connectors: Mixpanel, Intercom, NetSuite (or "pending" state), BambooHR, CoreFM PostgreSQL CDC (pending Diane Hooper sign-off), Segment BigQuery Destination, GCP Billing Export
- Deploy ChurnZero Cloud Function to Cloud Run; configure GCS watermark; set up Cloud Scheduler trigger (daily 6am UTC)
- Deploy Clearbit Cloud Function; configure HubSpot webhook trigger; set up Pub/Sub topic for rate-limiting
- Build `stg_pagerduty__incidents`, `stg_pagerduty__services`, `stg_bamboohr__employees`
- Build `stg_gcp_billing__costs`, `stg_cloud_monitoring__metrics`
- Build `stg_churnzero__accounts`, `stg_churnzero__health_scores`, `stg_churnzero__activities`
- Build `stg_netsuite__transactions`, `stg_netsuite__customers`
- Deploy Cloud Composer 2 environment (us-central1)
- Build `core_dynamics_daily_pipeline` DAG: Fivetran trigger tasks, Cloud Function trigger tasks, Dataform staging run, assertion run, warehouse trigger

**Sophie Tanner:**
- Build `stg_google_ads__campaigns`, `stg_google_ads__ads`, `stg_google_ads__keywords`
- Build `stg_linkedin_ads__campaigns`, `stg_linkedin_ads__ad_analytics`
- Build `stg_meta_ads__campaigns`, `stg_meta_ads__insights`
- Build `stg_outreach__sequences`, `stg_outreach__prospects`
- Build `stg_mixpanel__events`, `stg_mixpanel__people`
- Build `stg_segment__tracks`, `stg_segment__identifies`, `stg_segment__pages`
- Build `stg_intercom__conversations`, `stg_intercom__contacts`
- Build `stg_corefm__feature_events`, `stg_corefm__sessions`, `stg_corefm__users` (PII pseudonymisation applied; pending legal sign-off from Diane Hooper)
- Build `stg_clearbit__companies`
- Build `dim_account`, `dim_csm` shared dimensions
- Write Dataform assertions for all staging models and shared dimensions (not_null, unique, referential_integrity, row_count_threshold, business_sanity, freshness)

**Exit criteria:**
- All 18 source connectors active (NetSuite and CoreFM may be in "pending" state with documented blockers)
- All staging models running without errors; all assertions defined
- `dim_date`, `dim_account`, `dim_csm` populated and assertion-clean
- Cloud Composer DAG executing nightly; Slack alerts configured
- Data quality monitoring dataset (`pipeline_run_log`, `assertion_results`) populated
- Deployment runbook draft complete

**Blocked if:** Fivetran Business Critical licence not procured; CoreFM network access unresolved (connector ships in "pending" state — non-blocking)

---

### Sprint 3 (Weeks 5–6): Marketing Data Modelling

**Goal:** All marketing integration and warehouse models built; marketing attribution logic correct for all 5 models.

**Sophie Tanner:**
- Build `int__marketing_touches`: join HubSpot contacts to Salesforce opportunities via email match; flag the 12% contact mismatch gap
- Build `int__campaign_performance`: join Google Ads, LinkedIn Ads, Meta Ads spend data to HubSpot campaign attribution
- Build `int__funnel_stage`: HubSpot funnel including SAL stage (Lead → MQL → SAL → SQL → Opportunity → Closed Won)
- Build `fct_attribution_touches`: implement all 5 attribution models (first-touch, last-touch, linear, time-decay, U-shaped) as columns on the same fact table; 180-day attribution window
- Build `fct_campaign_spend`: daily spend by campaign/channel/ad group
- Build `fct_pipeline_influenced`: marketing-touched opportunities in pipeline; influenced ARR calculation
- Build `dim_campaign`, `dim_channel`
- Write assertions for all marketing models

**Irina Volkov (Looker Developer — starting Week 5):**
- Set up Looker project connected to BigQuery warehouse dataset
- Build LookML `marketing_touches` view (with all 5 attribution model measures)
- Build LookML `campaign_performance` view
- Build LookML `attribution_comparison` view
- Build `marketing_attribution` explore (primary); `campaign_roi` explore

**Exit criteria:**
- All 5 attribution models returning results for last 12 months of closed-won opportunities
- U-shaped model figures validated against Rachel Summers' expectations (sign-off meeting)
- Attribution coverage gap (12%) documented in LookML description field
- LookML views and explores deployable to Looker

---

### Sprint 4 (Weeks 7–8): Marketing Semantic Layer + Product Analytics Start

**Goal:** Marketing semantic layer complete with governed metrics; Campaign Performance dashboard live; Product Analytics modelling begun.

**Sophie Tanner:**
- Begin `int__feature_usage`: join CoreFM feature events to dim_feature hierarchy
- Begin `int__session_events`: CoreFM session events with account join
- Begin `int__onboarding_steps`: Intercom events mapped to 8-step checklist
- Build `dim_feature` (Module → Feature Group → Feature hierarchy from CoreFM schema)

**Irina Volkov:**
- Build LookML governed metrics for marketing workstream: `marketing_qualified_leads`, `cost_per_mql`, `pipeline_influenced_revenue`, `marketing_sourced_arr`, `roas_by_channel`
- Implement LookML `${marketing_sourced_arr}` using consistent definition (signed off by Rachel Summers + James Petit)
- Build Campaign Performance Dashboard (LookML-native Looker dashboard): spend, MQL, CAC by channel/campaign/period; date range filter; channel filter
- Begin Attribution Comparison Dashboard

**Exit criteria:**
- Campaign Performance Dashboard accessible in Looker (even if data is partial)
- All marketing governed metrics certified by Rachel Summers
- `dim_feature` built with Module → Feature Group → Feature hierarchy
- Feature usage integration models started

---

### Sprint 5 (Weeks 9–10): Marketing Dashboards + Product Analytics Modelling

**Goal:** All 3 marketing dashboards complete; product analytics models built through warehouse layer.

**Sophie Tanner:**
- Build `int__product_events__deduplicated`: Segment + Mixpanel deduplication using Leon Yip event-to-source mapping (if available; otherwise document caveat)
- Build `fct_feature_usage` (partitioned by date, clustered by account_id)
- Build `fct_session_events`
- Build `fct_onboarding_completion`: 8-step checklist completion per account per cohort
- Build `dim_account_product_score`: composite daily engagement score using weights approved by Claire Ashworth + Tara Obinna

**Irina Volkov:**
- Complete Attribution Comparison Dashboard (all 5 models side-by-side; channel contribution breakdown)
- Build Pipeline Influence Dashboard (marketing-touched pipeline; revenue attribution; SAL funnel)
- Begin LookML product workstream: `feature_usage` view, `product_engagement` view, `onboarding_funnel` view, `account_product_score` view
- Build `product_adoption` explore (primary)

**Exit criteria:**
- All 3 marketing dashboards complete and reviewed by Rachel Summers (Sprint 5 review meeting)
- `dim_account_product_score` populated for all 312 accounts
- Product feature usage models assertion-clean

**Blocked if:** Leon Yip event-to-source mapping not received — deduplication model ships with caveat annotation

---

### Sprint 6 (Weeks 11–12): Product Semantic Layer + Customer Analytics Start

**Goal:** Product semantic layer and dashboards complete; Customer 360 integration modelling begun; Ops start.

**Sophie Tanner:**
- Begin `int__customer_360`: join Salesforce accounts/opportunities/contracts to ChurnZero health/NPS/activity to dim_account_product_score
- Begin `int__renewal_history`: Salesforce opportunity history (24 months: 14 ChurnZero + 10 Salesforce proxy)
- Begin `int__support_health`: Zendesk ticket aggregate per account (open tickets, CSAT, escalations)

**Kofi Asante:**
- Build `stg_pagerduty` assertion fixes if needed from Sprint 2 monitoring data
- Begin `int__sla_events`: Zendesk ticket events → first-response time and resolution time vs SLA policy
- Begin `int__infrastructure_cost`: GCP Billing export → cost per project/service (pending Sean Murphy label audit)
- Begin `int__incident_metrics`: PagerDuty incidents → MTTA/MTTR per engineer per week

**Irina Volkov:**
- Build LookML governed metrics for product workstream: `dau_mau_ratio`, `feature_adoption_rate`, `onboarding_completion_rate`, `product_engagement_score`, `licence_utilisation_rate`
- Build Feature Adoption Dashboard (module/feature group/feature level; account/segment/cohort filter)
- Begin Account Health Score Dashboard

**Exit criteria:**
- Feature Adoption Dashboard accessible and reviewed by Claire Ashworth + Leon Yip
- `int__customer_360` running in Dataform (even if some columns pending)
- SLA events and incident metrics integration models started

---

### Sprint 7 (Weeks 13–14): Product Dashboards + Customer Modelling + Ops Modelling

**Goal:** All 3 product dashboards complete; BQML training dataset ready; Ops warehouse models complete.

**Sophie Tanner:**
- Build `int__churn_signals`: join product score, support health, NPS trend, renewal history, champion departure indicator
- Build `fct_renewal_outcomes`: historical renewal data for BQML training (24 months)
- Prepare BQML training dataset: feature engineering (engagement score, escalation rate, NPS slope, DAU/MAU, days-to-renewal, champion departure flag); validated with Tara Obinna
- Build `fct_support_interactions`: Zendesk ticket detail per account

**Kofi Asante:**
- Build `fct_sla_compliance` (partitioned by date): SLA policy compliance per ticket; first-response vs resolution time vs SLA tier
- Build `fct_infrastructure_cost_daily`: GCP cost per customer per day (labels permitting)
- Build `fct_incident_response`: PagerDuty MTTA/MTTR per incident; on-call engineer mapping
- Build `mart_ops__sla_summary`: pre-aggregated SLA compliance rate by tier/week

**Irina Volkov:**
- Complete Account Health Score Dashboard (product engagement score breakdown per account; CSM-ready)
- Build Onboarding Funnel Dashboard (8-step completion rates; stuck accounts; cohort comparison)
- Begin LookML `customer_health` view, `churn_risk` view, `renewal_pipeline` view
- Begin `customer_360` explore with RLS configuration

**Exit criteria:**
- All 3 product dashboards complete and reviewed by Claire Ashworth (Sprint 7 review meeting)
- BQML training dataset validated; Tara Obinna sign-off on features and proxy data approach
- All Ops warehouse models running assertion-clean

---

### Sprint 8 (Weeks 15–16): Customer Semantic Layer + BQML + Ops Dashboards

**Goal:** BQML churn model trained and validated; Customer Analytics LookML complete; Ops dashboards built.

**Sophie Tanner:**
- Train BigQuery ML logistic regression churn model on `fct_renewal_outcomes`
- Evaluate model: AUC-ROC, precision at 0.5 threshold; iterate on feature selection if metrics below target
- Extract top-3 contributing signals per at-risk account (BQML feature importance)
- Build `mart_customer__churn_risk`: churn risk scores + renewal probabilities for accounts due in 180 days + top-3 signals
- Build `dim_csm_assignment`

**Kofi Asante:**
- Configure Looker Alerts integration (email/Slack; weekly digest for CSMs)
- Configure Google Slides API for Looker QBR Action
- Technical review of Looker RLS implementation for CSM portfolios

**Irina Volkov:**
- Build LookML governed metrics for customer workstream: `customer_health_score`, `churn_risk_score`, `renewal_probability`, `arr_at_risk`, `nrr`, `grr`, `net_arr_change`
- Implement RLS in `customer_360` explore: CSMs see only assigned accounts; managers see full portfolio
- Build Customer Health Command Centre (top at-risk accounts; top-3 signals per account; CSM-filtered)
- Build Support SLA Tracker dashboard (SLA compliance rate by tier; breach risk; intraday refresh)
- Build Infrastructure Cost & Efficiency dashboard (GCP cost per customer; MoM trend; cost-per-seat)
- Build Incident Response dashboard (MTTA/MTTR trends; on-call burden; severity breakdown)

**Exit criteria:**
- BQML model AUC-ROC ≥ 0.75; precision ≥ 60% (or documented decision to relax with Tara Obinna sign-off)
- `mart_customer__churn_risk` populated for all accounts with renewals in next 180 days
- Customer Health Command Centre accessible with top-3 signals visible
- All 3 Ops dashboards built and data-complete
- RLS verified: CSM test accounts see only their portfolio

---

### Sprint 9 (Weeks 17–18): Customer Dashboards + Governance Start + KT Session 1

**Goal:** All 3 customer dashboards complete; QBR Action working; semantic layer audit begun; first KT delivered.

**Sophie Tanner:**
- Support Looker QBR Action testing (Google Slides generation for 3 test accounts)
- Semantic layer audit: review all LookML views across Releases 03–05 for redundant measures, inconsistent naming, missing descriptions
- Draft LookML naming convention guide

**Irina Volkov:**
- Build Renewal Pipeline Dashboard (accounts due in 180 days; risk tier; recommended action)
- Build NRR & Revenue Retention Dashboard (NRR/GRR trends; cohort analysis; expansion vs contraction)
- Configure Looker Alerts: weekly email digest to CSMs for accounts with churn score change > 10 points
- Build and test QBR Looker Action (one-click Google Slides; validate output format with Ben Tran)
- Begin metric certification: NRR, GRR, MRR — run figures against Finance baseline; document variance

**Mark Rittman:**
- UAT preparation: draft UAT scenarios for each workstream (5–8 scenarios per workstream sponsor)
- KT Session 1 delivery (Week 18): Data engineering runbook (Fivetran, Cloud Composer, Dataform) — audience: Amara Diallo, Kofi Asante
- Schedule UAT sessions with Rachel Summers, Claire Ashworth, Tara Obinna, Harriet Drummond (Weeks 17–19)

**Exit criteria:**
- All 3 customer dashboards complete and reviewed by Tara Obinna (Sprint 9 review meeting)
- QBR Action generates a Google Slides deck in < 5 minutes for any test account
- Looker Alerts delivering weekly digest to test CSM accounts
- KT Session 1 delivered; attendance confirmed
- UAT scenarios drafted and shared with workstream sponsors

---

### Sprint 10 (Weeks 19–20): UAT + Ops Dashboards Complete + KT Session 2

**Goal:** UAT sessions delivered for all workstreams; issues logged and prioritised; KT Session 2 delivered.

**All team:**
- Deliver UAT sessions (2 hours each): Rachel Summers (Marketing), Claire Ashworth (Product), Tara Obinna (Customer), Harriet Drummond (Operations)
- Log all UAT issues in Linear; triage into: must-fix (blocks sign-off) vs nice-to-have (post-engagement)
- Fix all must-fix UAT issues (allocated in Sprint 10 and Sprint 11 buffer)

**Irina Volkov:**
- Metric certification completion: CAC, MRR defined once in LookML; sign-off from all workstream sponsors
- LookML semantic layer governance guide: naming conventions, measure definition standards, field certification workflow, access control matrix
- Semantic layer audit report: redundant measures identified and resolved; descriptions complete

**Mark Rittman:**
- KT Session 2 delivery (Week 20): Looker governance and LookML development — audience: Amara Diallo and future LookML developers
- Escalate any unresolved UAT blockers to Priya Nair (CTO)

**Exit criteria:**
- All 4 UAT sessions delivered and issues logged
- Must-fix UAT issues in progress or resolved
- Metric certification: MRR, NRR, GRR, CAC all return same figure across all dashboards (±2% tolerance)
- KT Session 2 delivered; attendance confirmed

---

### Sprint 11 (Weeks 21–22): UAT Remediation + Final Deployment + KT Session 3

**Goal:** All UAT sign-offs received; production deployment complete; KT Session 3 delivered; engagement closed.

**All team:**
- Resolve all remaining must-fix UAT issues
- Obtain UAT sign-off from all 4 workstream sponsors
- Final Dataform release: tag `v1.0.0` on `feature/data_platform_build`; merge to main
- Production deployment: final LookML push to production; verify all dashboards accessible
- User provisioning: 30+ Standard Looker users; 5 Developer users (list from Ben Tran)
- Send Looker welcome email to all provisioned users

**Irina Volkov:**
- Final dashboard QA: spot-check 10% of dashboard tiles for data accuracy
- Document any known limitations (NetSuite gap if not resolved, deduplication caveat if mapping not received)
- Looker content folders organised: per-workstream folders with appropriate permissions

**Mark Rittman:**
- KT Session 3 delivery (Week 21): Dashboard usage and Looker Actions — audience: CS team leads, Marketing, Product, and Operations workstream leads + CSMs
- Final engagement summary document: what was built, what was deferred, post-engagement recommendations
- Handover meeting with Amara Diallo (technical) and Priya Nair (executive)
- Close all Linear issues; final status commit to `feature/data_platform_build`

**Exit criteria:**
- UAT sign-off received from all 4 workstream sponsors
- All governed metrics returning same figure across all dashboards (±2% tolerance)
- Production deployment complete; all 30+ users provisioned
- 3 KT sessions delivered; attendance records complete
- Data engineering runbook and semantic layer governance guide published to Confluence WAEE and Notion
- Linear project marked complete; final commit pushed

---

## 4. Dependencies Map

```
GCP access (Week 1)
    → 02-foundation all connectors
    → all subsequent releases

Fivetran licence (Week 1)
    → 02-foundation connectors
    → all staging models

Diane Hooper legal sign-off (Week 2)
    → stg_corefm staging models
    → 04-product-analytics (feature usage, engagement score)

Leon Yip event-to-source mapping (before Week 10)
    → int__product_events__deduplicated
    → 04-product-analytics product scores

Engagement score weights (before Week 9)
    → dim_account_product_score
    → 05-customer-analytics churn model features

Tara Obinna BQML proxy acceptance (before Week 10)
    → 05-customer-analytics BQML training dataset
    → Sprint 8 BQML training

Sean Murphy label audit (before Week 12)
    → fct_infrastructure_cost_daily
    → 06-operational-analytics Infrastructure Cost dashboard

Rachel Summers deduplication decision (before Week 5)
    → 03-marketing-analytics attribution gap documentation
    → Campaign Performance dashboard disclaimer
```

---

## 5. Risks to Sprint Plan

| Risk | Sprint Impact | Mitigation |
|---|---|---|
| GCP access delayed | S1 blocked — no work can start | Escalate to Priya Nair Day 1 |
| CoreFM PostgreSQL access delayed | S2 CoreFM staging; S4–S6 product models slide | Ship connector in "pending" state; product analytics timeline adjusts |
| Legal sign-off delayed past Week 2 | S4 product modelling blocked for CoreFM | Product models built without CoreFM; backfilled when sign-off received |
| Leon Yip mapping delayed past Week 10 | S5 deduplication model ships with caveat | Document known limitation; fix in post-engagement if needed |
| BQML AUC-ROC < 0.75 after Sprint 8 | S8/S9 remediation needed | One-sprint buffer in S9 for feature engineering iteration |
| UAT issues > 20 must-fix items | S10/S11 capacity risk | Triage strictly; defer nice-to-haves; escalate scope risk to Priya Nair |
| NetSuite never provisioned | 06-operational-analytics financial metrics incomplete | Document gap; operational dashboards ship without NetSuite-derived metrics |

---

*Document status: Generated by Wire Autopilot — validated and self-approved*
*Reviewed by: Wire Autopilot (self-review)*
*Review date: 2026-03-29*
