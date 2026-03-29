# Engagement Context

## Engagement Details

| Field | Value |
|---|---|
| Engagement Name | data_platform_build |
| Client Name | Core Dynamics |
| Created Date | 2026-03-29 |
| Engagement Lead | Mark Rittman |
| Repo Mode | combined |

## Client Background

Core Dynamics, Inc. is a mid-market B2B SaaS company headquartered in Austin, Texas (offices in London and Singapore). They sell CoreFM — a cloud-based facility and asset management platform — to 312 enterprise customers across commercial real estate, manufacturing, healthcare, and higher education. FY2025 ARR: $38.4M, ~480 employees, Series C funded.

**Board target:** Improve NRR from 104% to 115% within 18 months.

## Engagement Summary

22-week Google Cloud & Looker data platform engagement covering five workstreams:
- **Foundation:** BigQuery, Fivetran, Dataform, Cloud Composer orchestration
- **Marketing Analytics:** Multi-touch attribution, campaign ROI, funnel analysis
- **Product Analytics:** Feature adoption, account usage scoring, onboarding funnel
- **Customer Analytics:** Customer 360, churn risk model (BigQuery ML), CSM tooling
- **Operational Analytics:** Support SLA, infrastructure cost, incident response

## Primary Contacts

| Role | Name |
|---|---|
| Engagement Lead (RA) | Mark Rittman |
| Senior Analytics Engineer (RA) | Sophie Tanner |
| Data Engineer (RA) | Kofi Asante |
| Looker Developer (RA) | Irina Volkov |
| Project Manager (RA) | Daniel Osei |
| Primary Technical Contact (client) | Amara Diallo (Data Engineering Lead) |
| Executive Sponsor (client) | Priya Nair (CTO) |

## Technology Stack

- **Data Warehouse:** Google BigQuery
- **Ingestion:** Fivetran (standard connectors) + custom Cloud Functions (ChurnZero, Clearbit)
- **Transformation:** Dataform (staging → intermediate → warehouse)
- **Orchestration:** Cloud Composer 2 (Airflow DAGs)
- **Semantic Layer / BI:** Looker (LookML)
- **ML:** BigQuery ML (churn risk logistic regression)
- **Infrastructure:** GCP (BigQuery, Pub/Sub, Secret Manager, Cloud Logging, Cloud Monitoring)

## Repository Configuration

- **Repo mode:** combined — `.wire/` lives directly in this repository
- **Client code repo:** same repository

## Issue Tracker

- **Linear:** Team RIT, project ra-autopilot-test-0e3a53c420f5 (create_in_existing mode)

## Document Store

**Provider**: Both

**Confluence Space**: WAEE
**Confluence Parent Page**: [Core Dynamics data_platform_build — Wire Documents](https://rittmananalytics.atlassian.net/wiki/spaces/WAEE/pages/3367665668/Core+Dynamics+data_platform_build+Wire+Documents)
All generated artifacts will be published as child pages of this Confluence page.

**Notion Parent Page**: [Core Dynamics data_platform_build — Wire Documents](https://www.notion.so/3329ce3e29d681f88340f717a111dd0b)
All generated artifacts will be published as sub-pages of this Notion page.
