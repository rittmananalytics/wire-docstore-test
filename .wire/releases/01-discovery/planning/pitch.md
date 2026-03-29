# Pitch — data_platform_build
## Core Dynamics, Inc.

**Prepared by:** Wire Autopilot
**Date:** 2026-03-29
**Engagement:** data_platform_build
**Client:** Core Dynamics, Inc.
**Engagement Reference:** RA-2026-0041

---

## 1. Problem

Core Dynamics has a board mandate — grow NRR from 104% to 115% in 18 months — and no data infrastructure capable of supporting it.

The CS team cannot see churn risk proactively. The marketing team cannot prove which spend generates revenue. The product team cannot see which features are used by which accounts. Finance cannot reconcile MRR figures across functions. Operations tracks SLA compliance in a Google Sheet updated once a week. All of this is a data problem: 18 source systems, zero governed integration, one analyst (James Petit) manually reconciling CSV exports to paper over the gaps.

The human cost is measurable: 60% of James Petit's working time on manual data work. A $340K ARR churn event (Meridian Healthcare) with visible leading indicators that no one joined together in time. 14 CSMs spending 3–5 hours per QBR pulling data from four systems. Marketing unable to defend a $2.1M budget to the board because LinkedIn attribution is invisible.

Without a data platform that centralises, governs, and activates this data, Core Dynamics cannot make the decisions its NRR target requires.

---

## 2. Appetite

**22 weeks. $250,000 fixed price. Five-person Rittman Analytics team.**

This is the full scope of the engagement. The timeline is fixed by the SOW — Phase 1 (Foundation, Wks 1–4), Phase 2 (Data Modelling & Semantic Layer, Wks 5–14), Phase 3 (Dashboards, UAT, Deployment, Wks 15–22).

We are not building a general-purpose data platform. We are building exactly what Core Dynamics needs to move NRR from 104% to 115% — no more, no less. Every delivery decision is subject to the 22-week, $250K constraint.

---

## 3. Solution

### The Shaped Approach

Build a Google Cloud ELT data platform — BigQuery as the warehouse, Fivetran for ingestion, Dataform for transformation, Cloud Composer for orchestration, Looker for the semantic layer and dashboards — that answers the five questions Core Dynamics currently cannot answer:

1. **Which accounts are at risk of churning?** (Customer Analytics)
2. **Which marketing spend generates revenue?** (Marketing Analytics)
3. **Which product features drive retention?** (Product Analytics)
4. **Are we meeting SLA commitments, and what is infrastructure costing per customer?** (Operational Analytics)
5. **What is our single, agreed MRR/NRR/CAC?** (Governed Semantic Layer)

### Architecture in Brief

```
18 Source Systems
    ↓
Fivetran (16 standard connectors) + Cloud Functions (ChurnZero, Clearbit)
    ↓
BigQuery Raw Datasets (raw_salesforce, raw_hubspot, raw_corefm_db, ...)
    ↓
Dataform Staging → Integration → Warehouse (mart_marketing, mart_product,
mart_customer, mart_ops, shared dimensions)
    ↓
Looker LookML Semantic Layer (governed metrics, RLS, 10 dashboards)
```

Cloud Composer 2 orchestrates the full pipeline (nightly full refresh + intraday incremental for high-priority sources). BigQuery ML logistic regression provides churn risk scores. A Looker Action generates one-click QBR Google Slides decks.

### Deliverables by Release

| Release | Scope | Key Outputs |
|---|---|---|
| 02-foundation | Pipeline only | GCP setup, 18 Fivetran/custom connectors, Dataform staging (all sources), Cloud Composer DAGs, data quality assertions |
| 03-marketing-analytics | Full platform | Marketing Dataform models, 5-model multi-touch attribution, LookML, 3 dashboards (Campaign Performance, Attribution Comparison, Pipeline Influence) |
| 04-product-analytics | Full platform | Product Dataform models, engagement score, onboarding funnel, LookML, 3 dashboards (Feature Adoption, Account Health Score, Onboarding Funnel) |
| 05-customer-analytics | Full platform | Customer 360, BigQuery ML churn model, LookML with RLS, 3 dashboards + Looker Alerts + QBR Action |
| 06-operational-analytics | dbt development | Support SLA, GCP cost, incident metrics, 3 dashboards (SLA Tracker, Infrastructure Cost, Incident Response) |
| 07-semantic-layer-governance | Dashboard extension | Semantic layer audit, governance guide, 3 KT sessions, final UAT |

### Key Design Decisions

- **ELT over ETL** — BigQuery processing is cheap; raw data retention enables re-transformation without re-extraction
- **Dataform over dbt** — team has deep Dataform expertise; native GCP integration simplifies DevOps
- **Cloud Composer 2 over Cloud Workflows** — Airflow provides mature dependency management; Amara Diallo's team has existing Airflow familiarity
- **BambooHR column blocklist** — salary and personal data excluded at Fivetran layer; proactive data minimisation
- **PII pseudonymisation at staging** — CoreFM `user_id` SHA-256 hashed at staging; email/name columns blocked before they enter the warehouse
- **U-shaped attribution as default** — per Rachel Summers' explicit requirement; 180-day attribution window

---

## 4. Rabbit Holes

These are known risks with contained, agreed mitigations. They are not unknowns — they are documented and managed.

### R1: ChurnZero history gap (14 months vs 24 months required for BQML)
**Risk**: The churn model needs 24 months of renewal outcome data. ChurnZero only has 14 months.
**Mitigation**: Use Salesforce opportunity history as a proxy for the pre-ChurnZero period. This must be accepted by Tara Obinna before the model is built (open question R-03).
**Constraint**: Model precision may be slightly lower than the ≥ 60% target. If Salesforce proxy data proves insufficient, we can relax the precision target or extend the observation window — but we cannot go outside scope to source additional historical data.

### R2: 12% HubSpot–Salesforce contact mismatch
**Risk**: 12% of marketing journeys have no Salesforce counterpart, making them invisible to attribution.
**Mitigation**: Document the coverage gap explicitly in the Marketing Analytics dashboard. Attribution model results will carry a "12% coverage gap" disclaimer until Core Dynamics resolves the deduplication (client responsibility per R-01).
**Constraint**: We will not perform data cleanup in Salesforce or HubSpot — that is explicitly out of scope.

### R3: Segment + Mixpanel double-counting
**Risk**: The same product events may appear in both Segment (streaming to BigQuery) and Mixpanel (via Fivetran). Double-counting inflates usage metrics.
**Mitigation**: Build an `int__product_events__deduplicated` integration model in Release 04. Leon Yip must provide the event-to-source mapping before this model is built. If the mapping is delayed, Release 04 ships with a documented deduplication caveat.
**Constraint**: We will not modify the Segment or Mixpanel configurations in the source systems.

### R4: NetSuite provisioning lead time (2–3 weeks)
**Risk**: The NetSuite integration role can take 2–3 weeks to provision. If delayed past Week 4, NetSuite data will not be available for Phase 2.
**Mitigation**: Start NetSuite provisioning request in Week 1. If delayed, Release 02 ships with NetSuite connector in "pending" state. Financial metrics that require NetSuite will be flagged as incomplete until the connector goes live.
**Constraint**: No workaround for NetSuite data — it must be provisioned by Core Dynamics.

### R5: CoreFM PostgreSQL network access
**Risk**: The Cloud SQL Auth Proxy for CoreFM PostgreSQL CDC has not yet been provisioned. Without it, product usage data cannot be ingested.
**Mitigation**: Provisioning is a Week 1 dependency. Amara Diallo's team owns this. If delayed, Release 02 ships with the CoreFM connector staged but inactive.
**Constraint**: Legal sign-off from Diane Hooper is also required before CoreFM staging models go live (separate dependency). Both must be resolved before product usage data enters the warehouse.

### R6: GCP Billing label coverage
**Risk**: Sean Murphy's GCP cost labels are inconsistently applied across projects. Without consistent labels, per-customer infrastructure cost allocation is inaccurate.
**Mitigation**: Sean Murphy must complete a label coverage audit before Release 06. If < 80% coverage, the Infrastructure Cost dashboard ships with a "partial allocation" disclaimer.
**Constraint**: We will not retroactively re-tag GCP resources — that is client responsibility.

---

## 5. No-Gos

These are explicitly out of scope. They will not be built regardless of client request. Any work in these areas is billable at $2,200/day against the out-of-scope rate.

| No-Go | Reason |
|---|---|
| Real-time streaming dashboards (sub-minute latency) | SOW Section 10. Segment streaming to BigQuery is ~15-minute latency — sufficient for all use cases identified. |
| Data catalogue / data discovery tooling | SOW Section 10. Out of scope and not required for the NRR mandate. |
| Predictive models beyond churn risk | SOW Section 10. Expansion propensity, LTV prediction, lead scoring are future phases. |
| Source system configuration changes | SOW Section 10. We read from source systems; we do not modify them. |
| GDPR/CCPA compliance review | SOW Section 10. We implement pseudonymisation as specified; legal review is Core Dynamics' responsibility. |
| Mobile or embedded analytics | SOW Section 10. Looker web app only. |
| Ongoing managed service | SOW Section 10. Engagement ends with 3 KT sessions and a runbook. |
| Salesforce data cleanup | Discovery. Client responsibility (R-01). |
| Retroactive UTM parameter remediation | Discovery. Niamh Collins to address going forward (R-04). |

---

## 6. How We Know It Worked

The engagement is a success when these conditions are verifiably met:

| Signal | Measurement |
|---|---|
| MRR discrepancy eliminated | Single Looker definition returns same figure as Finance + CS + Sales reports; ±2% tolerance |
| James Petit's time freed | Self-reported < 20% of time on manual data work (vs 60% today) |
| Churn model live and performant | BQML model in production; AUC-ROC ≥ 0.75; precision ≥ 60% at 0.5 threshold |
| Multi-touch attribution visible | 5 models selectable in Looker; U-shaped default; LinkedIn/Meta visible in closed-won journeys |
| Product usage self-service | CS team can filter Looker by account/feature without engineering support |
| SLA visibility intraday | SLA dashboard refreshes every 4 hours; no end-of-day surprises |
| QBR prep < 1 hour | One-click Looker Action generates Google Slides deck in < 5 minutes |
| Pipeline reliability | Fivetran connectors < 0.1% error rate; Dataform assertions passing daily |

---

*Document status: Generated by Wire Autopilot — validated and self-approved*
*Reviewed by: Wire Autopilot (self-review)*
*Review date: 2026-03-29*
