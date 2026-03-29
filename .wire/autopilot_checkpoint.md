# Autopilot Checkpoint

## Configuration
- Engagement: data_platform_build
- Client: Core Dynamics
- Lead: Mark Rittman
- SOW: sow.md
- Linear: Team RIT, project ra-autopilot-test-0e3a53c420f5 (create_in_existing)
- Document store: both (Confluence WAEE, Notion 3329ce3e-29d6-81f8-8340-f717a111dd0b)
- Branch: feature/data_platform_build

## SOW Summary

Core Dynamics, Inc. is a Series C B2B SaaS company ($38.4M ARR, 312 enterprise accounts) selling CoreFM — a facility and asset management platform. The board has mandated NRR improvement from 104% to 115% within 18 months. The engagement delivers a unified Google Cloud data platform (BigQuery, Dataform, Cloud Composer, Fivetran, Looker) across a 22-week, $250K engagement in three phases.

**Core problems:** No single source of truth across Salesforce, ChurnZero, Mixpanel, NetSuite, Zendesk, and CoreFM PostgreSQL; analyst bottleneck (60% of time on manual exports); MRR discrepancy of ~8% between functions; no product usage visibility at account level; last-touch-only marketing attribution; operational data trapped in spreadsheets.

**Data sources (18 total):** Salesforce, HubSpot, Google Ads, LinkedIn Ads, Meta Ads, Outreach.io, Clearbit, CoreFM PostgreSQL (CDC), Mixpanel, Segment, Intercom, ChurnZero (custom API), Zendesk, NetSuite, PagerDuty, GCP Billing Export, Google Cloud Monitoring, BambooHR.

**Key deliverables:** GCP infrastructure & IAM; Fivetran pipelines (all sources); Dataform staging/intermediate/warehouse models; Cloud Composer orchestration; governed LookML semantic layer; 10 Looker dashboards across 4 workstreams; BigQuery ML churn risk model; Looker alerts; 3 knowledge transfer sessions; data engineering runbook; semantic layer governance guide.

**Technology:** BigQuery, Dataform, Cloud Composer 2 (Airflow), Fivetran, Looker (LookML), BigQuery ML, Pub/Sub, Secret Manager, Cloud Logging.

**Timeline:** 22 weeks — Phase 1 (Wks 1–4): Foundation; Phase 2 (Wks 5–14): Data modelling & semantic layer; Phase 3 (Wks 15–22): Dashboards, UAT, deployment. Total fees: $250K.

**Key constraints:** GCP access required by end of Week 1; Fivetran Business Critical license needed; Looker Standard/Enterprise with 30+ Standard + 5 Developer seats; CoreFM PostgreSQL PII pseudonymisation requires legal sign-off (Diane Hooper); 24 months historical data needed for BQML model.

**Key risks confirmed:** 12% Salesforce-HubSpot contact mismatch rate; ChurnZero only 14 months history (vs 24 months needed); 30% UTM parameter non-compliance; Segment/Mixpanel double-counting risk; CoreFM PostgreSQL network access not yet provisioned.

**Discovery session findings:**
- Tara Obinna (VP CS): Wants explainable churn signals ("top 3 reasons"), not just a score. Prefers higher sensitivity (more false positives OK). One-click QBR data pull is high priority.
- Rachel Summers (VP Marketing): U-shaped attribution as default model. 180-day marketing-influenced attribution window (not 90 days). HubSpot funnel includes SAL stage between MQL and SQL.
- Product analytics: Feature hierarchy (Module → Feature Group → Feature). Licence utilisation metric (`contracted_user_seats` from Salesforce). 8-step onboarding checklist tracked in Intercom.

## Engagement Structure
- Discovery release: .wire/releases/01-discovery/
- Delivery releases: (to be determined by sprint plan)

## Completed Phases
- engagement_setup: complete
- discovery_sprint: complete
- 02-foundation: complete

## Current Phase
02-foundation: complete

## Delivery Releases to Execute
| Release | Type | Scope |
|---------|------|-------|
| 02-foundation | pipeline_only | GCP setup, Fivetran pipelines (18 sources), Dataform staging, Cloud Composer, data quality |
| 03-marketing-analytics | full_platform | Marketing models, multi-touch attribution (5 models), LookML, 3 dashboards |
| 04-product-analytics | full_platform | Product models, engagement score, onboarding funnel, LookML, 3 dashboards |
| 05-customer-analytics | full_platform | Customer 360, BQML churn model, LookML with RLS, 3 dashboards + alerts + actions |
| 06-operational-analytics | dbt_development | Operations models, LookML, 3 dashboards |
| 07-semantic-layer-governance | dashboard_extension | Semantic layer audit, governance, documentation, 3 KT sessions, final UAT |

## Key Context
- Data sources: Salesforce, HubSpot, Google Ads, LinkedIn Ads, Meta Ads, Outreach.io, Clearbit, CoreFM PostgreSQL, Mixpanel, Segment, Intercom, ChurnZero, Zendesk, NetSuite, PagerDuty, GCP Billing, Google Cloud Monitoring, BambooHR
- Key entities: Account, Contact, Opportunity, Campaign, Lead, Feature, Session, SupportTicket, Incident, Contract, CSM
- Deliverables: GCP infrastructure, Fivetran pipelines, Dataform models, Cloud Composer DAGs, LookML project, 10 dashboards, BQML churn model, runbooks, training sessions
- Technologies: BigQuery, Dataform, Cloud Composer, Fivetran, Looker, BigQuery ML
- SOW timeline: 22 weeks, $250K

## Decisions Made
(none yet)

## Blocked Artifacts
(none)
