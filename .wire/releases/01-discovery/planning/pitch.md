# Pitch: Core Dynamics Data Platform Build

**Prepared by:** Wire Autopilot
**Date:** 2026-03-29
**Engagement:** data_platform_build
**Client:** Core Dynamics, Inc.

---

## 1. Problem Statement

Core Dynamics is a Series C B2B SaaS company ($38.4M ARR, 312 accounts) with a board target of NRR 115% within 18 months. That target cannot be achieved through gut feel. The company needs to know, in real time, which customers are at churn risk, which marketing investments generate pipeline, and which product features drive retention. Today, none of those questions can be answered reliably.

Data exists — across Salesforce, HubSpot, ChurnZero, Mixpanel, CoreFM PostgreSQL, Zendesk, NetSuite, PagerDuty, and a dozen more systems — but it is fragmented, manually reconciled, and inconsistent. MRR differs by 8% between Sales, Finance, and CS. The single analyst spends 60% of his time on VLOOKUP reconciliations. The CS team identifies churn after customers say no. Marketing can't see LinkedIn's contribution to pipeline. Product usage data is inaccessible without an engineer.

The solution is a unified, governed data platform on Google Cloud — not a marginal improvement to the current situation, but a complete replacement of manual data workflows with an automated, trustworthy stack that every business function can use self-service.

---

## 2. Appetite

**Big batch — 22 weeks**

This is a substantial, multi-workstream engagement covering infrastructure, four analytics domains, a governed semantic layer, and a BigQuery ML model. A 6-week small-batch scope would not be sufficient to deliver the integrated foundation that makes the individual analytics workstreams valuable. The 22-week timeline in the SOW is the right container.

**Budget: $250,000**

At $2,200/day, this represents approximately 114 consultant-days across the full team (Engagement Lead, Senior Analytics Engineer, Data Engineer, Looker Developer, Project Manager). This is a fixed-price engagement with defined milestones.

---

## 3. Appetite Statement

We have 22 weeks and $250K to build a production-grade Google Cloud data platform for Core Dynamics. Within that budget, we will deliver the foundation (GCP, Fivetran, Dataform), four analytics workstreams (Marketing, Product, Customer, Operations), a governed Looker semantic layer, and a BigQuery ML churn risk model. We will not build a data catalogue, embedded analytics, or real-time streaming infrastructure — those are future phases. Every delivery decision will be tested against this appetite before scope is added.

---

## 4. Solution

The shaped solution is a **three-layer Google Cloud data stack** with a governed semantic layer serving purpose-built dashboards.

**Layer 1 — Foundation (Phase 1, Weeks 1–4):**
Fivetran ingests all 18 source systems into BigQuery raw datasets. Cloud Composer orchestrates sync schedules. A custom Cloud Function handles ChurnZero (no native Fivetran connector) and Clearbit enrichment. CoreFM PostgreSQL is connected via Fivetran log-based CDC + Cloud SQL Auth Proxy. All staging models are built in Dataform with column-level documentation, type casting, and PII pseudonymisation (user_id hashed at staging layer for CoreFM data, pending Diane Hooper's legal sign-off).

**Layer 2 — Transformation (Phase 2, Weeks 5–14):**
Dataform builds a four-workstream warehouse:
- **Marketing mart**: `fct_lead_funnel_events`, `fct_campaign_spend`, `fct_attribution_touches`, `fct_opportunity_attribution` (5 attribution models: first-touch, last-touch, linear, time-decay, U-shaped; U-shaped as default). SAL stage added to funnel. 180-day marketing-influenced attribution window.
- **Product mart**: `fct_feature_usage` (Module/FeatureGroup/Feature hierarchy), `dim_account_product_score` (composite engagement score with licence_utilisation_pct), `fct_onboarding_funnel` (8 Intercom steps).
- **Customer mart**: `dim_account_health` (combining product, support, sentiment, financial signals), `fct_renewal_pipeline`, `fct_expansion_signals`. BigQuery ML logistic regression churn risk model trained on 24 months of renewal outcomes (using Salesforce opportunity data as proxy for pre-ChurnZero period).
- **Operations mart**: `fct_support_tickets` (Zendesk SLA timers), `fct_incident_response` (PagerDuty MTTA/MTTR), `fct_infra_cost_by_customer` (GCP Billing allocated by label).

Shared dimensions: `dim_date` (fiscal calendar starting 1 Feb), `dim_account` (master account joining Salesforce, ChurnZero, product score), `dim_csm`.

**Layer 3 — Semantic Layer & Dashboards (Phase 3, Weeks 15–22):**
LookML project (`core_dynamics_looker/`) with 5 models (marketing, product, customer, operations, executive), governed metric definitions (MRR, NRR, GRR, CAC, LTV, LTV:CAC, Product Engagement Score, Churn Risk Probability, SLA Compliance Rate), row-level security for CSM dashboards, and 10 purpose-built dashboards.

**Looker Actions:** "Flag for CSM Follow-up" → Salesforce task creation. "Export QBR Data" → pre-populated Google Slides deck.

**Looker Alerts:** Health score drops 15+ points in 7 days; account enters "At Risk" band; renewal 60 days out with < 70% probability; new Sev 1/2 ticket on at-risk account.

**Fat-marker verdict:** Foundation is complete and stable before workstream builds begin. Workstreams run in dependency order (Marketing/Product in parallel first, then Customer which depends on Product, then Operations). Semantic layer built incrementally alongside each workstream. Final governance audit in Week 21.

---

## 5. Rabbit Holes

1. **Salesforce-HubSpot contact deduplication.** The 12% mismatch rate will reduce attribution coverage but not block delivery. Implementing fuzzy matching at the staging layer (if required) is scoped. A full Salesforce data cleanup is explicitly out of scope and a client responsibility.

2. **ChurnZero historical data gap.** With only 14 months of ChurnZero data, the BQML model will use Salesforce opportunity data as a proxy for the pre-ChurnZero period. We will not delay the model to wait for more ChurnZero history to accumulate.

3. **GCP Billing label completeness.** Infrastructure cost-per-customer (DO-02) depends on GCP resource labels mapping to customers. If label coverage is low, the dashboard will show a partial view — we will document the gap rather than retrospectively relabelling resources.

4. **CoreFM PostgreSQL schema complexity.** The operational DB has PII and a complex schema. We will pseudonymise at staging (one-way hash on user_id) and build product marts on account_id as the join key. We will not attempt to reverse-engineer undocumented schema relationships without Amara Diallo's input.

5. **BigQuery ML model accuracy expectations.** We target AUC-ROC ≥ 0.75 and precision ≥ 60% at 0.5 threshold. If the reduced training window (14 months ChurnZero + Salesforce proxy) means we cannot hit this target, we will document the limitation in the model card and set realistic expectations with Tara Obinna — not inflate reported accuracy.

---

## 6. No-Gos

- **No real-time streaming dashboards.** Sub-minute latency is out of scope. Minimum refresh is 4 hours (Support SLA dashboard).
- **No data catalogue** (Dataplex, Alation). Future phase.
- **No predictive models beyond churn risk.** Expansion propensity, LTV prediction, lead scoring: not in scope.
- **No source system modifications.** Salesforce, HubSpot, Zendesk schemas and workflows will not be changed.
- **No GDPR/CCPA compliance review.** PII pseudonymisation implemented; legal sign-off is the client's responsibility.
- **No mobile or embedded analytics.** Looker web application only.
- **No ongoing managed service.** Build and handover only. Post-engagement retainer is a separate commercial conversation.
- **No additional training** beyond the 3 knowledge transfer sessions in the deliverables.

---

## 7. Open Questions

1. **Salesforce data cleanup (R-01):** Will Core Dynamics run a Salesforce contact deduplication sprint to reduce the 12% HubSpot-Salesforce mismatch, or accept the attribution coverage gap?

2. **Product engagement score weighting (R-11):** Have the revised signal weights been agreed by both Claire Ashworth and Tara Obinna? This must be signed off before the `dim_account_product_score` model is built.

3. **Segment/Mixpanel deduplication (R-06):** Leon Yip to provide event-to-source mapping to enable deduplication at the staging layer. When will this be available?

4. **GCP Billing label coverage (R-05):** Sean Murphy to provide label coverage audit. Without this, the scope of DO-02 (Infrastructure Cost & Efficiency dashboard) cannot be finalised.

5. **Legal sign-off on PII pseudonymisation (R-10):** Priya Nair to introduce Diane Hooper. Target sign-off by end of Week 2. What is the confirmed timeline?

---

## 8. Downstream Releases

Based on the SOW phasing and the complexity of each workstream, the following delivery releases are proposed:

| Release Name | Type | Scope Summary | Priority |
|---|---|---|---|
| 02-foundation | pipeline_only | GCP setup, Fivetran connectors, Dataform staging models, Cloud Composer orchestration, data quality framework | 1 |
| 03-marketing-analytics | full_platform | Marketing data model, multi-touch attribution, LookML marketing views, 3 dashboards (MA-01, MA-02, MA-03) | 2 |
| 04-product-analytics | full_platform | Product data model, engagement score, onboarding funnel, LookML product views, 3 dashboards (PB-01, PB-02, PB-03) | 2 |
| 05-customer-analytics | full_platform | Customer 360, BigQuery ML churn model, LookML customer views, 3 dashboards + alerts (CA-01, CA-02, CA-03), Looker Actions | 3 |
| 06-operational-analytics | dbt_development | Operations data model, LookML operations views, 3 dashboards (DO-01, DO-02, DO-03) | 4 |
| 07-semantic-layer-governance | dashboard_extension | Semantic layer audit, LookML governance review, executive dashboard, Looker Actions, documentation & training | 5 |

**Dependency order:** 02-foundation must complete before any analytics workstream begins. 05-customer-analytics depends on 04-product-analytics (product engagement score feeds customer health). 03-marketing-analytics and 04-product-analytics can run in parallel. 06-operational-analytics is partially parallelisable with 05. 07-semantic-layer-governance runs last.

---

## 9. Betting Table Case

- Core Dynamics has a $250K budget committed and a clear board mandate (NRR 115% in 18 months) — the commercial case for this investment is not in question.
- The technical approach (BigQuery + Fivetran + Dataform + Looker) is proven at this scale and well within Rittman Analytics' core competency.
- All 18 data source connections are established technologies (16 via standard Fivetran connectors, 2 via Cloud Function + REST API). No novel integration risk.
- The SOW has been reviewed and signed; stakeholders are engaged and available. Discovery sessions with all four workstream sponsors are complete.
- The phased delivery structure means value is delivered incrementally — Foundation complete in Week 4, first analytics dashboards in Weeks 15–17 — rather than in a single big-bang delivery.

---

## 10. Metrics for Success

| Metric | Target | Measurement |
|---|---|---|
| Pipeline data quality | All Dataform assertions pass for 3 consecutive daily runs; Fivetran < 0.1% error rate over 7 days | Automated assertions + Fivetran monitoring |
| Metric reconciliation | MRR/NRR/CAC variance between functions reduced to ≤ 2% | Stakeholder sign-off on Metric Definitions document |
| Churn model performance | AUC-ROC ≥ 0.75; precision ≥ 60% at 0.5 threshold | BigQuery ML evaluation metrics on held-out test set |
| Dashboard adoption | All 4 workstream sponsors actively using dashboards by Week 22 | Looker usage analytics |
| Analyst time reclaimed | James Petit spends < 20% of time on manual data work (from ~60%) | Self-reported at knowledge transfer session |
| QBR prep time | CSM QBR prep time < 1 hour per account (from 3–5 hours) | Self-reported by CSMs at UAT session |
| NRR trajectory | Board receives single, agreed NRR figure at next board meeting post-engagement | [To confirm with client — NRR target is 18-month horizon] |

---

*Document status: Generated by Wire Autopilot — self-reviewed and approved*
*Reviewed by: Wire Autopilot (self-review)*
*Date: 2026-03-29*
