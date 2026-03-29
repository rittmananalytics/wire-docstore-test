# Requirements Specification — Foundation
## Core Dynamics Data Platform Build — Release 02

**Prepared by:** Wire Autopilot
**Date:** 2026-03-29
**Release:** 02-foundation (pipeline_only)
**Engagement Reference:** RA-2026-0041

---

## 1. Executive Summary

The Foundation release establishes the complete Google Cloud data infrastructure for Core Dynamics. This is the prerequisite for all analytics workstreams: no marketing attribution, customer health, product usage, or operational reporting is possible until reliable, automated data pipelines land all source data into BigQuery with consistent, tested staging models.

The scope covers GCP infrastructure provisioning, 18 Fivetran data connectors (with 2 custom Cloud Functions for ChurnZero and Clearbit), a Dataform transformation layer (staging and shared dimension models), Cloud Composer orchestration, and a data quality framework. Delivery occupies Weeks 3–7 (Phase 1 and early Phase 2) of the 22-week engagement.

---

## 2. Business Context

**Client:** Core Dynamics, Inc. — Series C B2B SaaS, $38.4M ARR, 312 enterprise accounts
**Problem:** No centralised data infrastructure. Pipelines are ad-hoc scripts on developer laptops or scheduled Google Sheets imports. There is no data warehouse, no transformation layer, and no monitoring.
**Strategic goal:** Enable all four analytics workstreams (Marketing, Product, Customer, Operations) that will collectively power a board-mandated NRR improvement from 104% to 115%.

---

## 3. Stakeholders

| Name | Role | Department | Involvement |
|---|---|---|---|
| Amara Diallo | Data Engineering Lead | Engineering | Primary Technical Contact — sign-off authority for Foundation milestone |
| Priya Nair | CTO | Engineering | Executive Sponsor — approves GCP access provisioning |
| Sean Murphy | Platform Engineering | Engineering | CoreFM PostgreSQL network access; GCP Billing export; PagerDuty |
| Kofi Asante | Data Engineer (RA) | Rittman Analytics | Primary builder for Foundation release |
| Sophie Tanner | Sr Analytics Engineer (RA) | Rittman Analytics | Dataform staging models, data model standards |
| Daniel Osei | Project Manager (RA) | Rittman Analytics | Milestone tracking, access provisioning coordination |

---

## 4. Functional Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| FR-001 | Provision GCP analytics-prod and analytics-dev projects | Both projects exist; IAM roles configured; required APIs enabled |
| FR-002 | Configure IAM following least-privilege principles | `data-engineers`, `analysts`, `looker-service-account`, `dataform-runner` roles created and tested |
| FR-003 | Enable required GCP APIs | BigQuery, Dataform, Cloud Composer, Pub/Sub, Secret Manager, Cloud Logging APIs all enabled |
| FR-004 | Configure Fivetran connector for Salesforce | Connector active, syncing every 6 hours, < 0.1% error rate over 7-day observation |
| FR-005 | Configure Fivetran connector for HubSpot | Connector active, syncing every 1 hour |
| FR-006 | Configure Fivetran connectors for Google Ads, LinkedIn Ads, Meta Ads | 3 connectors active, syncing daily at 3am UTC |
| FR-007 | Configure Fivetran connector for Outreach.io | Connector active, syncing every 6 hours |
| FR-008 | Configure Fivetran CDC connector for CoreFM PostgreSQL | Log-based CDC connector active, 15-minute sync; Cloud SQL Auth Proxy configured with Sean Murphy |
| FR-009 | Configure Fivetran connector for Mixpanel | Connector active, syncing daily at 4am UTC |
| FR-010 | Configure Segment BigQuery Destination | Streaming destination active; events landing in BigQuery within 5 minutes |
| FR-011 | Configure Fivetran connector for Intercom | Connector active, syncing every 6 hours |
| FR-012 | Build ChurnZero Cloud Function | Cloud Function calling ChurnZero REST API; incremental sync logic; landing data in `raw_churnzero`; scheduled daily 6am UTC |
| FR-013 | Configure Fivetran connector for Zendesk | Connector active, syncing every 1 hour |
| FR-014 | Configure Fivetran connector for NetSuite | Connector active, syncing daily at 5am UTC (integration role provisioning started immediately due to 2–3 week lead time) |
| FR-015 | Configure Fivetran connector for PagerDuty | Connector active, syncing every 1 hour |
| FR-016 | Configure GCP Billing Export to BigQuery | Native export active in `raw_gcp_billing` dataset |
| FR-017 | Configure Google Cloud Monitoring export | Cloud Monitoring API → Dataform daily export pipeline |
| FR-018 | Configure Fivetran connector for BambooHR | Connector active, syncing daily at 5am UTC |
| FR-019 | Build Clearbit Cloud Function | Cloud Function triggered by HubSpot webhook on new inbound lead; enrichment data lands in BigQuery |
| FR-020 | Build Dataform staging model for Salesforce | `stg_salesforce__accounts`, `stg_salesforce__contacts`, `stg_salesforce__opportunities`, `stg_salesforce__campaigns`, `stg_salesforce__contracts` — with column docs, type casting, freshness config |
| FR-021 | Build Dataform staging model for HubSpot | `stg_hubspot__contacts`, `stg_hubspot__companies`, `stg_hubspot__form_submissions`, `stg_hubspot__email_events`, `stg_hubspot__page_views` — with UTM parameter parsing |
| FR-022 | Build Dataform staging models for ad platforms | `stg_google_ads__*`, `stg_linkedin_ads__*`, `stg_meta_ads__*` covering campaigns, ad groups, ads, spend, impressions, clicks |
| FR-023 | Build Dataform staging models for Outreach.io | `stg_outreach__sequences`, `stg_outreach__email_touches`, `stg_outreach__call_touches` |
| FR-024 | Build Dataform staging models for CoreFM PostgreSQL | `stg_corefm__accounts`, `stg_corefm__users` (PII pseudonymised — user_id hashed), `stg_corefm__sessions`, `stg_corefm__feature_events`, `stg_corefm__work_orders`, `stg_corefm__assets` |
| FR-025 | PII pseudonymisation in CoreFM staging | `user_id` one-way hashed at staging layer; only `account_id` retained as join key; approach documented and signed off by Diane Hooper before implementation |
| FR-026 | Build Dataform staging models for Mixpanel and Segment | `stg_mixpanel__events`, `stg_segment__events` with deduplication rules (Leon Yip event-to-source mapping required) |
| FR-027 | Build Dataform staging models for Intercom, ChurnZero, Zendesk | `stg_intercom__*`, `stg_churnzero__*`, `stg_zendesk__*` |
| FR-028 | Build Dataform staging models for NetSuite, PagerDuty, GCP Billing, Cloud Monitoring, BambooHR | All remaining source staging models |
| FR-029 | Load feature taxonomy seed data | `raw_corefm_db.feature_taxonomy` table loaded from Leon Yip's CSV mapping event_name to Module/FeatureGroup/Feature |
| FR-030 | Build shared dimension models | `dim_date` (fiscal calendar starting 1 February), `dim_account`, `dim_csm` |
| FR-031 | Deploy Cloud Composer 2 environment | Environment active in GCP; DAGs deployable |
| FR-032 | Build Fivetran trigger DAG | DAG orchestrates all Fivetran syncs in dependency order |
| FR-033 | Build Dataform pipeline DAG | DAG triggers Dataform after Fivetran sync (nightly full refresh + intraday incremental for high-priority tables) |
| FR-034 | Build data quality check DAG | Post-transformation data quality checks; Dataform assertions pass |
| FR-035 | Build alerting DAG | Slack/email alerts fire on pipeline failure; test alert verified |
| FR-036 | Implement Dataform assertions | Row counts, null rates, referential integrity, business logic sanity checks (e.g., ARR never negative) on all staging and shared dimension models |
| FR-037 | BigQuery audit log export | Cloud Logging export configured for BigQuery audit logs |
| FR-038 | Pipeline health dashboard in Looker | Dashboard showing pipeline run times, failure rates, row count trends — live by end of Phase 2 |

---

## 5. Non-Functional Requirements

| Category | Requirement |
|---|---|
| **Reliability** | Fivetran connectors must maintain < 0.1% error rate over a 7-day observation period |
| **Data freshness** | High-priority sources (Salesforce, HubSpot, Zendesk, PagerDuty) must refresh within 1 hour; CoreFM PostgreSQL within 15 minutes; batch sources within 24 hours |
| **Data quality** | Dataform assertions must pass for 3 consecutive daily runs before Phase 1 milestone sign-off |
| **Security** | All credentials stored in GCP Secret Manager; no credentials in code; IAM least-privilege enforced |
| **PII protection** | CoreFM user PII pseudonymised at staging layer before any downstream use; legal sign-off required before implementation |
| **Observability** | Pipeline failures generate Slack/email alerts within 5 minutes of failure detection |
| **Maintainability** | All Dataform models documented with column-level descriptions; Data Engineering Runbook delivered at end of engagement |
| **Scalability** | BigQuery architecture uses dataset separation by source and by layer (raw → staging → integration → warehouse); partitioning and clustering applied to high-volume tables |

---

## 6. Data Requirements

| Source System | Connection Method | Sync Frequency | Raw Dataset | Owner |
|---|---|---|---|---|
| Salesforce | Fivetran Standard | Every 6 hours | raw_salesforce | Greg Fontaine / Ben Tran |
| HubSpot | Fivetran Standard | Every 1 hour | raw_hubspot | Niamh Collins |
| Google Ads | Fivetran Standard | Daily 3am UTC | raw_google_ads | Owen Brady |
| LinkedIn Ads | Fivetran Standard | Daily 3am UTC | raw_linkedin_ads | Owen Brady |
| Meta Ads | Fivetran Standard | Daily 3am UTC | raw_meta_ads | Owen Brady |
| Outreach.io | Fivetran Standard | Every 6 hours | raw_outreach | Niamh Collins |
| Clearbit | REST API → Cloud Function | On-demand (inbound leads) | raw_clearbit | Niamh Collins |
| CoreFM PostgreSQL | Fivetran Log-based CDC | Near real-time (15 min) | raw_corefm_db | Amara Diallo |
| Mixpanel | Fivetran Standard | Daily 4am UTC | raw_mixpanel | Leon Yip |
| Segment | Segment BigQuery Destination | Streaming | raw_segment | Leon Yip |
| Intercom | Fivetran Standard | Every 6 hours | raw_intercom | Julia Mercer |
| ChurnZero | Cloud Function (REST API) | Daily 6am UTC | raw_churnzero | Ben Tran |
| Zendesk | Fivetran Standard | Every 1 hour | raw_zendesk | Carlos Vega |
| NetSuite | Fivetran Standard | Daily 5am UTC | raw_netsuite | Sandra Kowalski |
| PagerDuty | Fivetran Standard | Every 1 hour | raw_pagerduty | Sean Murphy |
| GCP Billing Export | Native BigQuery Export | Daily | raw_gcp_billing | Sean Murphy |
| Google Cloud Monitoring | Cloud Monitoring API → Dataform | Daily | raw_cloud_monitoring | Sean Murphy |
| BambooHR | Fivetran Standard | Daily 5am UTC | raw_bamboohr | Mei Lin |

---

## 7. Technical Requirements

| Category | Requirement |
|---|---|
| **Platform** | Google Cloud Platform — BigQuery, Dataform, Cloud Composer 2 (Airflow), Pub/Sub, Secret Manager, Cloud Logging |
| **Ingestion** | Fivetran Business Critical (Core Dynamics account); custom Cloud Functions for ChurnZero and Clearbit |
| **Transformation** | Dataform repository connected to Core Dynamics GitHub organisation; three-layer model (staging → intermediate → warehouse) |
| **Orchestration** | Cloud Composer 2 environment; DAGs for Fivetran triggers, Dataform pipeline, data quality, alerting |
| **GCP Projects** | `core-dynamics-analytics-prod` and `core-dynamics-analytics-dev` (parallel development project) |
| **Access** | GCP Owner role on dev project; Editor on prod; provisioned by Amara Diallo by end of Week 1 |
| **Network** | CoreFM PostgreSQL via Cloud SQL Auth Proxy (Kofi Asante + Sean Murphy to configure) |
| **PII** | Legal sign-off (Diane Hooper) required before CoreFM PII pseudonymisation implementation |

---

## 8. Deliverables

| Deliverable | Wire Artifact | Acceptance Criteria |
|---|---|---|
| GCP project setup, IAM, environment configuration (D-01) | pipeline → infrastructure | Projects provisioned; IAM roles verified |
| Fivetran connectors — all 18 sources (D-02) | pipeline | < 0.1% error rate over 7-day observation; all connectors syncing on schedule |
| Dataform repository — staging models (D-03) | pipeline | All staging models pass Dataform assertions; column-level documentation complete |
| Cloud Composer DAGs (D-05) | pipeline | All 4 DAGs complete successfully; failure alert verified |
| Data quality assertions + pipeline health dashboard (D-06) | data_quality | Assertions pass 3 consecutive daily runs; dashboard live in Looker |

---

## 9. Timeline

| Week | Milestone | Owner |
|---|---|---|
| 3 | GCP projects, IAM, APIs; Salesforce, HubSpot, Google/LinkedIn/Meta Ads connectors; Dataform repo initialised | Kofi Asante, Sophie Tanner |
| 4 | Remaining Fivetran connectors; ChurnZero + Clearbit Cloud Functions; CoreFM PostgreSQL CDC connector; staging models for priority sources | Kofi Asante, Sophie Tanner |
| 5 | All remaining staging models; feature taxonomy seed; Cloud Composer environment deployed | Sophie Tanner, Kofi Asante |
| 6 | Cloud Composer DAGs (Fivetran trigger, Dataform pipeline, data quality, alerting); shared dimension models | Kofi Asante, Sophie Tanner |
| 7 | Dataform assertions; BigQuery audit log export; pipeline health dashboard; 7-day connector validation observation period | Sophie Tanner, Irina Volkov |

**Milestone M2:** All raw data landing in BigQuery; staging models passing assertions. Sign-off: Amara Diallo. **Target: End of Week 7.**

---

## 10. Assumptions and Dependencies

1. GCP Owner/Editor access provisioned by end of Week 2 (Amara Diallo, pre-approved by Priya Nair)
2. Fivetran Business Critical account with sufficient MAR — Core Dynamics to procure
3. CoreFM PostgreSQL network access via Cloud SQL Auth Proxy — Sean Murphy and Kofi Asante to configure by start of Week 4
4. NetSuite integration role creation started immediately — can take 2–3 weeks (R-08)
5. Legal sign-off on PII pseudonymisation from Diane Hooper — target end of Week 2 (R-10)
6. Leon Yip to provide Segment/Mixpanel event-to-source mapping before Segment/Mixpanel staging models are built (R-06)
7. Leon Yip to provide feature taxonomy CSV before feature_taxonomy seed table is loaded (FR-029)
8. GCP Billing labels — Sean Murphy to provide coverage audit before DO-02 scope is finalised (R-05)

---

## 11. Risks and Mitigations

| Risk | Impact | Likelihood | Mitigation |
|---|---|---|---|
| CoreFM PostgreSQL network access not provisioned (R-07) | High — blocks Product and Customer workstreams | Medium | Kofi + Sean aligning this week; target: before Phase 2 |
| NetSuite role creation 2–3 week lead time (R-08) | Medium — delays Customer workstream NetSuite data | Medium | Process started immediately; Priya actioned |
| PII pseudonymisation legal delay (R-10) | Medium — blocks CoreFM staging model build | Low | Diane Hooper introduced; target sign-off end Week 2 |
| Segment/Mixpanel event deduplication requires client input (R-06) | Medium — double-counting in product models | Medium | Leon Yip to provide mapping; staging models built conservatively until mapping received |
| ChurnZero custom connector more complex than anticipated (R-09) | Medium | Low | Buffer allocated in Phase 1; Kofi has experience with similar REST API connectors |

---

## 12. Scope Management

**In scope:** All items described in FR-001 through FR-038 above.

**Out of scope (Foundation release):**
- Intermediate and warehouse models (delivered in workstream releases 03–06)
- Dashboard development (delivered in workstream releases 03–07)
- Real-time streaming (sub-minute latency)
- Data catalogue (Dataplex, Alation)
- Source system configuration changes

**Change process:** Any scope changes require written approval from Mark Rittman and Amara Diallo before work begins. Standard rate $2,200/day for out-of-scope work.

---

*Document status: Generated by Wire Autopilot — self-reviewed and approved*
*Reviewed by: Wire Autopilot (self-review)*
*Date: 2026-03-29*
