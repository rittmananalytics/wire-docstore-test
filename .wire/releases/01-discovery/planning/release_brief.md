# Release Brief — data_platform_build
## Core Dynamics, Inc.

**Prepared by:** Wire Autopilot
**Date:** 2026-03-29
**Engagement:** data_platform_build
**Client:** Core Dynamics, Inc.
**Engagement Reference:** RA-2026-0041

---

## 1. Overview

This release brief defines the full delivery structure for the Core Dynamics data platform engagement. It maps the problem definition and pitch into six concrete releases, specifies their scope, sequencing, dependencies, success criteria, and team assignments. It is the authoritative reference for what will be built, in what order, and to what standard.

**Engagement goal:** Deliver a governed Google Cloud data platform that enables Core Dynamics to increase NRR from 104% to 115% within 18 months of engagement completion.

---

## 2. Release Structure

The engagement is structured as six delivery releases following an initial discovery sprint:

| Release | Name | Type | Weeks | Primary Owner |
|---|---|---|---|---|
| 01-discovery | Discovery Sprint | discovery | Wks 1–2 | Mark Rittman |
| 02-foundation | Data Pipeline Foundation | pipeline_only | Wks 1–4 | Kofi Asante |
| 03-marketing-analytics | Marketing Analytics | full_platform | Wks 5–9 | Sophie Tanner |
| 04-product-analytics | Product Analytics | full_platform | Wks 8–13 | Sophie Tanner |
| 05-customer-analytics | Customer Analytics | full_platform | Wks 10–16 | Sophie Tanner + Kofi Asante |
| 06-operational-analytics | Operational Analytics | dbt_development | Wks 12–17 | Kofi Asante |
| 07-semantic-layer-governance | Semantic Layer & Governance | dashboard_extension | Wks 15–22 | Irina Volkov |

Releases 03–06 run partially in parallel (see Section 5: Timeline). Release 07 is the final phase — it runs UAT, knowledge transfer, and deployment across all workstreams.

---

## 3. Release Specifications

### Release 02: Data Pipeline Foundation
**Type:** pipeline_only
**Duration:** Weeks 1–4 (20 business days)
**Owner:** Kofi Asante (Data Engineer)

**Scope:**
- GCP project setup: BigQuery datasets (raw, staging, integration, warehouse, monitoring), IAM roles (data-engineers, analysts, looker-service-account, dataform-runner), Secret Manager, Cloud Logging
- Fivetran connectors (16): Salesforce, HubSpot, Google Ads, LinkedIn Ads, Meta Ads, Outreach.io, Mixpanel, Intercom, Zendesk, NetSuite, PagerDuty, BambooHR, CoreFM PostgreSQL (CDC), Segment BigQuery Destination, GCP Billing Export
- Custom Cloud Functions (2): ChurnZero API connector (daily, incremental, GCS watermark), Clearbit enrichment connector (HubSpot webhook trigger, Pub/Sub rate-limiting)
- Dataform staging models (18 sources): all following `stg_<source>__<entity>` naming; PII pseudonymisation for CoreFM; BambooHR column blocklist
- Shared dimensions: `dim_date`, `dim_account`, `dim_csm` (used by all downstream releases)
- Cloud Composer 2 DAG: `core_dynamics_daily_pipeline` (nightly + intraday incremental)
- Dataform assertions: not_null, unique, referential_integrity, row_count_threshold, business_sanity, freshness — on all staging and shared dimension models
- Data quality monitoring dataset: `pipeline_run_log`, `assertion_results`

**Artifacts:**
- Requirements specification
- Pipeline architecture design
- Dataform staging SQL (all 18 sources)
- Cloud Composer DAG code
- Cloud Function source code (ChurnZero, Clearbit)
- Deployment runbook (GCP setup, Fivetran configuration, Dataform repository)
- Data quality test suite

**Dependencies:**
- GCP Owner/Editor roles provisioned by end of Week 1 (Core Dynamics: Priya Nair / Amara Diallo)
- CoreFM PostgreSQL read replica accessible via Cloud SQL Auth Proxy (Core Dynamics: Amara Diallo — Week 1)
- Legal sign-off from Diane Hooper on PII pseudonymisation approach (target: end of Week 2)
- NetSuite integration role provisioned — expected 2–3 weeks; connector may ship in "pending" state
- Fivetran Business Critical licence procured (Core Dynamics)
- Looker Standard/Enterprise licence procured (30+ Standard + 5 Developer seats)

**Success criteria:**
- All 18 Fivetran/custom connectors active; < 0.1% error rate over 7-day observation period
- All staging models running without errors; Dataform assertions passing for 3 consecutive daily runs
- Cloud Composer DAG completing nightly without manual intervention
- `dim_date`, `dim_account`, `dim_csm` populated and assertion-clean

**Safety gate:** Pipeline activation requires explicit confirmation before any connector is switched to production mode.

---

### Release 03: Marketing Analytics
**Type:** full_platform
**Duration:** Weeks 5–9 (partially overlapping with Release 04 from Week 8)
**Owner:** Sophie Tanner (Sr Analytics Engineer) + Irina Volkov (Looker Developer)

**Scope:**
- Dataform integration models: `int__marketing_touches`, `int__campaign_performance`, `int__funnel_stage`
- Dataform warehouse models: `fct_attribution_touches`, `fct_campaign_spend`, `fct_pipeline_influenced`, `dim_campaign`, `dim_channel`
- 5-model multi-touch attribution: first-touch, last-touch, linear, time-decay, U-shaped (U-shaped is default; 180-day window)
- Attribution join logic: HubSpot contact → Salesforce opportunity via email address match; 12% mismatch gap documented
- SAL stage included in HubSpot funnel (between MQL and SQL per Rachel Summers' requirement)
- LookML views: `marketing_touches`, `campaign_performance`, `attribution_comparison`
- LookML explores: `marketing_attribution` (primary), `campaign_roi`
- Governed metrics in LookML: `marketing_qualified_leads`, `cost_per_mql`, `pipeline_influenced_revenue`, `marketing_sourced_arr`, `roas_by_channel`
- Looker dashboards (3):
  - Campaign Performance Dashboard (spend, MQL, CAC by channel/campaign/period)
  - Attribution Comparison Dashboard (all 5 models side-by-side; channel contribution)
  - Pipeline Influence Dashboard (marketing-touched opportunities in pipeline; revenue attribution)

**Dependencies:**
- Release 02 staging models for Salesforce, HubSpot, Google Ads, LinkedIn Ads, Meta Ads, Outreach.io complete and assertion-clean
- LookML project created (Irina Volkov, Week 5)
- 12% contact mismatch documented and accepted by Rachel Summers (discovery open question R-01)
- UTM parameter non-compliance flagged in dashboard; Niamh Collins to address going forward

**Success criteria:**
- All 5 attribution models returning results for the last 12 months of closed-won opportunities
- U-shaped model is default in Looker Attribution Comparison Dashboard
- LinkedIn and Meta channels visible in closed-won journeys (not zero)
- `marketing_sourced_arr` and `roas_by_channel` return consistent results against Finance's figures (±5% tolerance)

---

### Release 04: Product Analytics
**Type:** full_platform
**Duration:** Weeks 8–13 (overlaps Release 03 from Week 8, overlaps Release 05 from Week 10)
**Owner:** Sophie Tanner (Sr Analytics Engineer) + Irina Volkov (Looker Developer)

**Scope:**
- Dataform integration models: `int__feature_usage`, `int__session_events`, `int__onboarding_steps`, `int__product_events__deduplicated` (Segment + Mixpanel deduplication)
- Dataform warehouse models: `fct_feature_usage`, `fct_session_events`, `fct_onboarding_completion`, `dim_feature` (Module → Feature Group → Feature hierarchy), `dim_account_product_score`
- Feature hierarchy from CoreFM PostgreSQL: Module → Feature Group → Feature (3 levels)
- Daily engagement score: composite of DAU/MAU ratio, feature breadth, core activation, NPS trend, onboarding completion — weights to be signed off by Claire Ashworth + Tara Obinna (open question R-11)
- 8-step onboarding checklist tracked in Intercom events
- Licence utilisation: `contracted_user_seats` from Salesforce `User_Seats__c` field (field completeness to be confirmed by Amara Diallo)
- LookML views: `feature_usage`, `product_engagement`, `onboarding_funnel`, `account_product_score`
- LookML explores: `product_adoption` (primary), `onboarding`
- Governed metrics: `dau_mau_ratio`, `feature_adoption_rate`, `onboarding_completion_rate`, `product_engagement_score`, `licence_utilisation_rate`
- Looker dashboards (3):
  - Feature Adoption Dashboard (module/feature group/feature level; filterable by account, segment, cohort)
  - Account Health Score Dashboard (product engagement score breakdown per account; CSM-ready)
  - Onboarding Funnel Dashboard (8-step completion rates; stuck accounts)

**Dependencies:**
- Release 02 staging models for CoreFM PostgreSQL, Mixpanel, Segment, Intercom complete
- Legal sign-off from Diane Hooper (required before CoreFM staging models go live)
- Leon Yip event-to-source mapping for Segment/Mixpanel deduplication (open question R-06)
- Engagement score weights signed off by Claire Ashworth + Tara Obinna (open question R-11)
- `User_Seats__c` field population rate confirmed by Amara Diallo

**Success criteria:**
- `dim_account_product_score` populated for all 312 active accounts
- Feature hierarchy (Module → Feature Group → Feature) visible in Looker without engineer query
- CS team can filter Feature Adoption Dashboard by account without Slack-to-engineering request
- Onboarding funnel shows step-level drop-off for the last 6 cohorts

---

### Release 05: Customer Analytics
**Type:** full_platform
**Duration:** Weeks 10–16
**Owner:** Sophie Tanner (Sr Analytics Engineer) + Kofi Asante (Data Engineer) + Irina Volkov (Looker Developer)

**Scope:**
- Dataform integration models: `int__customer_360`, `int__renewal_history`, `int__support_health`, `int__churn_signals`
- Dataform warehouse models: `fct_renewal_outcomes`, `fct_support_interactions`, `dim_csm_assignment`, `mart_customer__churn_risk`
- Customer 360: joins Salesforce (account, opportunity, contract), ChurnZero (health, NPS, activity), Zendesk (tickets, CSAT), product score (from Release 04)
- BigQuery ML churn model: logistic regression; features = product engagement score, support escalation rate, NPS trend, DAU/MAU, days-since-last-login, DCP (days-to-contract-renewal), champion departure indicator; training data = 24 months (14 months ChurnZero + Salesforce opportunity proxy for pre-ChurnZero period)
- Model performance targets: AUC-ROC ≥ 0.75, precision ≥ 60% at 0.5 threshold
- Top-3 contributing signals per at-risk account (SHAP-equivalent feature importance in BQML)
- Renewal probabilities for accounts due in next 180 days
- LookML with row-level security (RLS): CSMs see only their assigned accounts; managers see full portfolio
- LookML views: `customer_health`, `churn_risk`, `renewal_pipeline`, `csm_performance`
- LookML explores: `customer_360` (primary), `churn_risk`
- Governed metrics: `customer_health_score`, `churn_risk_score`, `renewal_probability`, `arr_at_risk`, `nrr`, `grr`, `net_arr_change`
- Looker dashboards (3):
  - Customer Health Command Centre (top at-risk accounts; top-3 signals per account; CSM-filtered)
  - Renewal Pipeline Dashboard (accounts due in 180 days; risk tier; recommended action)
  - NRR & Revenue Retention Dashboard (NRR/GRR trends; cohort analysis; expansion vs contraction)
- Looker Alerts: weekly email digest to CSMs for accounts where churn risk score changes by > 10 points
- Looker Action: one-click QBR Google Slides generation (account snapshot, health score, top risks, open tickets, renewal details)

**Dependencies:**
- Release 02 staging models for Salesforce, ChurnZero, Zendesk complete
- Release 04 `dim_account_product_score` complete (product score is a churn model feature)
- Tara Obinna acceptance of Salesforce opportunity proxy for BQML training data (open question R-03)
- Google Slides API access configured for Looker Action
- Looker Alerts email integration configured

**Success criteria:**
- BQML churn model in production; AUC-ROC ≥ 0.75; precision ≥ 60%
- Customer Health Command Centre shows top-3 contributing signals per at-risk account
- All 14 CSMs can access their account portfolio in Looker (RLS verified)
- QBR Looker Action generates a populated Google Slides deck in < 5 minutes for any account
- NRR, GRR, and net ARR change all return the same number as the Finance baseline (±2% tolerance)

---

### Release 06: Operational Analytics
**Type:** dbt_development
**Duration:** Weeks 12–17 (overlaps Release 05)
**Owner:** Kofi Asante (Data Engineer) + Irina Volkov (Looker Developer)

**Scope:**
- Dataform integration models: `int__sla_events`, `int__infrastructure_cost`, `int__incident_metrics`
- Dataform warehouse models: `fct_sla_compliance`, `fct_infrastructure_cost_daily`, `fct_incident_response`, `mart_ops__sla_summary`
- SLA compliance: Zendesk ticket events → SLA policy → first-response and resolution time vs contractual SLA tiers; 4-hour refresh
- Infrastructure cost allocation: GCP Billing Export → cost per project/service → mapped to customer via resource labels (subject to Sean Murphy label coverage audit)
- Incident metrics: PagerDuty → MTTA (mean time to acknowledge), MTTR (mean time to resolve), on-call distribution by engineer, incident trend by severity
- LookML views: `sla_compliance`, `infrastructure_cost`, `incident_response`
- LookML explores: `operational_health`
- Governed metrics: `sla_compliance_rate`, `avg_first_response_time`, `avg_resolution_time`, `cost_per_customer_monthly`, `mtta`, `mttr`
- Looker dashboards (3):
  - Support SLA Tracker (SLA compliance rate by tier; breach risk; intraday refresh)
  - Infrastructure Cost & Efficiency (GCP cost per customer; MoM trend; cost-per-seat)
  - Incident Response Dashboard (MTTA/MTTR trends; on-call burden; severity breakdown)

**Dependencies:**
- Release 02 staging models for Zendesk, PagerDuty, GCP Billing complete
- Sean Murphy GCP Billing label coverage audit complete (open question R-05)
- SLA tier definitions from Carlos Vega (support contract tiers)

**Success criteria:**
- SLA Tracker refreshes every 4 hours without manual trigger
- Infrastructure cost dashboard shows per-customer allocation with > 80% label coverage
- MTTA/MTTR calculated correctly against PagerDuty incident data; values consistent with what Carlos Vega currently tracks manually

---

### Release 07: Semantic Layer Governance
**Type:** dashboard_extension
**Duration:** Weeks 15–22 (final phase; overlaps Releases 05 and 06)
**Owner:** Irina Volkov (Looker Developer) + Mark Rittman (Engagement Lead)

**Scope:**
- Semantic layer audit: review all LookML views, explores, and measures across Releases 03–06; identify redundant measures, inconsistent naming, missing descriptions
- Governance guide: LookML naming conventions, measure definition standards, field certification workflow, access control matrix
- Metric certification: MRR, NRR, GRR, CAC, ARR — signed off by all workstream sponsors
- UAT: 2-hour sessions with Rachel Summers, Claire Ashworth, Tara Obinna, Harriet Drummond; issues logged and resolved
- Knowledge transfer sessions (3):
  - Session 1 (Wk 18): Data engineering runbook (Fivetran, Cloud Composer, Dataform); for Amara Diallo and Kofi Asante
  - Session 2 (Wk 20): Looker governance and LookML development; for Amara Diallo and future LookML developers
  - Session 3 (Wk 21): Dashboard usage and Looker Actions; for CS, Marketing, Product workstream leads and CSMs
- Documentation: data engineering runbook, semantic layer governance guide, dashboard user guide (per workstream)
- Production deployment: final Dataform release, Looker content folders, user provisioning (30+ Standard seats, 5 Developer seats)

**Dependencies:**
- All delivery releases (03–06) UAT-ready
- All workstream sponsors available for 2-hour UAT sessions (Weeks 17–19)
- Google Cloud production access confirmed for final deployment
- Looker user list from Ben Tran (CS Ops) and Daniel Osei (PM)

**Success criteria:**
- All governed metrics (MRR, NRR, GRR, CAC) return the same figure across every dashboard (±2% tolerance)
- UAT sign-off received from all 4 workstream sponsors
- 3 KT sessions delivered; attendance confirmed for all required participants
- Data engineering runbook covers all 18 connectors, Dataform release process, and Composer DAG restart procedures
- All 30+ Standard users provisioned; Looker welcome email sent

---

## 4. Team and Responsibilities

| Team Member | Role | Primary Releases |
|---|---|---|
| Mark Rittman | Engagement Lead | All releases (oversight), 01-discovery, 07-governance |
| Sophie Tanner | Sr Analytics Engineer | 03-marketing-analytics, 04-product-analytics, 05-customer-analytics |
| Kofi Asante | Data Engineer | 02-foundation, 05-customer-analytics, 06-operational-analytics |
| Irina Volkov | Looker Developer | 03-marketing-analytics, 04-product-analytics, 05-customer-analytics, 06-operational-analytics, 07-governance |
| Daniel Osei | Project Manager | All releases (tracking, client communication, risk log) |

**Client contacts by release:**

| Release | Client Lead | Client Reviewer |
|---|---|---|
| 02-foundation | Amara Diallo | Priya Nair |
| 03-marketing-analytics | Rachel Summers | Owen Brady |
| 04-product-analytics | Claire Ashworth | Leon Yip |
| 05-customer-analytics | Tara Obinna | Ben Tran |
| 06-operational-analytics | Harriet Drummond | Carlos Vega / Sean Murphy |
| 07-governance | All workstream sponsors | Marcus Elwood (final sign-off) |

---

## 5. Timeline

```
Week:   1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22
01-discovery [==]
02-foundation    [========]
03-marketing                  [=============]
04-product               [====================]
05-customer                         [=================]
06-operations                            [===========]
07-governance                                      [=================]
```

**Phase 1** (Wks 1–4): Foundation + Discovery
**Phase 2** (Wks 5–14): All four analytics workstreams (overlapping)
**Phase 3** (Wks 15–22): Governance, UAT, KT, deployment

---

## 6. Risks and Mitigations

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| GCP access delayed past Week 1 | Medium | High | Escalate to Priya Nair (CTO); no Release 02 work can begin without GCP access |
| CoreFM PostgreSQL access delayed | Medium | High | Release 02 ships without CoreFM connector active; Release 04 Product Analytics timeline slides |
| NetSuite provisioning > 3 weeks | Medium | Medium | NetSuite connector ships in "pending" state; operational finance metrics flagged as incomplete |
| Legal sign-off (Diane Hooper) delayed | Low | High | CoreFM staging models not deployed until sign-off received; product data blocked |
| Segment/Mixpanel deduplication mapping late | Medium | Medium | Release 04 ships with deduplication caveat; CSMs informed that product metrics may be slightly overstated |
| BQML precision < 60% target | Low | Medium | Tune classification threshold; relax to 55% with sponsor sign-off; document clearly |
| UAT issues identified in Wk 17–19 | Medium | Medium | Buffer weeks 20–22 provide remediation time; critical issues only are scope for closure |

---

## 7. Open Items Requiring Client Action

| Item | Owner | Required By |
|---|---|---|
| GCP Owner/Editor role provisioning | Priya Nair / Amara Diallo | End of Week 1 |
| CoreFM PostgreSQL read replica + Cloud SQL Auth Proxy | Amara Diallo | End of Week 1 |
| Fivetran Business Critical licence procurement | Amara Diallo | End of Week 1 |
| Looker licence procurement (30+ Standard, 5 Developer) | Amara Diallo | End of Week 2 |
| Legal sign-off on PII pseudonymisation (Diane Hooper) | Mark Rittman to coordinate | End of Week 2 |
| NetSuite integration role provisioning | Amara Diallo | Week 1 (ASAP — 2–3 week lead time) |
| Salesforce-HubSpot contact deduplication decision (R-01) | Rachel Summers | Before Release 03 kickoff (Week 5) |
| BQML training data proxy acceptance (R-03) | Tara Obinna | Before Release 05 kickoff (Week 10) |
| Product engagement score weight sign-off (R-11) | Claire Ashworth + Tara Obinna | Before Release 04 model build (Week 9) |
| GCP Billing label coverage audit (R-05) | Sean Murphy | Before Release 06 kickoff (Week 12) |
| Segment/Mixpanel event-to-source mapping (R-06) | Leon Yip | Before Release 04 int model build (Week 10) |
| `User_Seats__c` field population rate confirmation | Amara Diallo | Before Release 04 model build (Week 9) |
| SLA tier definitions | Carlos Vega | Before Release 06 kickoff (Week 12) |

---

*Document status: Generated by Wire Autopilot — validated and self-approved*
*Reviewed by: Wire Autopilot (self-review)*
*Review date: 2026-03-29*
