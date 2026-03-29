# Problem Definition
## Core Dynamics — data_platform_build

**Prepared by:** Wire Autopilot
**Date:** 2026-03-29
**Engagement:** data_platform_build
**Client:** Core Dynamics, Inc.

---

## 1. Executive Summary

Core Dynamics, Inc. is a Series C B2B SaaS company ($38.4M ARR) whose ability to hit its board-mandated NRR target of 115% is directly constrained by a lack of reliable, integrated data. Revenue-critical decisions — which customers are at churn risk, which marketing channels generate pipeline, which product features drive retention — are currently made from fragmented, manually reconciled data spread across six or more disconnected systems. This problem definition frames the full scope of that data deficit and establishes the criteria for a solution that genuinely resolves it.

---

## 2. Who Has the Problem

**Primary stakeholders:**

| Stakeholder | Role | Primary Data Need |
|---|---|---|
| Tara Obinna | VP Customer Success | Account-level health signals to drive proactive CS motions |
| Rachel Summers | VP Marketing | Multi-touch attribution to justify and optimise demand gen spend |
| Claire Ashworth | VP Product | Account-level feature adoption to prioritise roadmap and support CS expansion plays |
| Harriet Drummond | COO | SLA compliance, infrastructure cost, and support efficiency metrics |
| Priya Nair | CTO / Executive Sponsor | A trustworthy, governed platform that serves all analytics domains |
| Amara Diallo | Data Engineering Lead | A maintainable, documented platform that her team can own post-handover |
| James Petit | Finance Analyst | Automated metrics reconciliation to replace manual spreadsheet work |

**Secondary stakeholders:** All 14 CSMs (need per-account tooling), Owen Brady (Demand Gen), Ben Tran (CS Ops), Carlos Vega (Support Engineering), Sean Murphy (Platform Engineering).

---

## 3. What They Are Trying to Do

Core Dynamics stakeholders are trying to manage a growing enterprise SaaS business — retaining customers, acquiring new ones, building the right product, and running operations efficiently — using data. Specifically:

- **CS team**: Identify and intervene with at-risk accounts *before* they decline a renewal call, and identify expansion opportunities *proactively* rather than reactively
- **Marketing team**: Optimise a $2.1M annual demand generation budget by understanding which channels and campaigns actually generate closed-won pipeline — not just MQLs
- **Product team**: Understand which features are adopted, by which accounts, at what depth, and use that signal to prioritise the roadmap and identify accounts that are not realising core product value
- **Operations team**: Monitor SLA compliance, infrastructure cost per customer, and incident response metrics in near real-time
- **Finance / Executive**: Report a single, agreed MRR/NRR/GRR figure to the board with confidence, replacing the current situation where Sales, Finance, and CS each report different numbers

---

## 4. What Is in the Way

1. **No integrated data warehouse.** Customer health, product usage, revenue, marketing touches, and support tickets all live in separate SaaS tools. There is no mechanism to join them.

2. **No governed metric definitions.** MRR figures differ by up to 8% between Sales, Finance, and CS. CAC is calculated differently by different teams. There is no agreed single source of truth.

3. **Analyst bottleneck.** James Petit (the only dedicated analyst) spends ~60% of his time on manual CSV exports and VLOOKUP reconciliations. He cannot scale to serve four workstream sponsors simultaneously.

4. **Product data is engineer-only.** CoreFM product usage telemetry exists in the production PostgreSQL database but is only accessible to engineers via direct SQL. The CS team has no product usage visibility; the product team has no account-level data.

5. **ChurnZero health scores are superficial.** ChurnZero's health score is based primarily on login events. It does not incorporate product feature usage, support ticket history, NPS trends, or renewal signals. CSMs report low confidence in the scores.

6. **Marketing attribution is last-touch only.** HubSpot's native attribution credits the last click before MQL conversion. Paid social (LinkedIn, Meta) — which operates earlier in the funnel — receives almost no attribution credit despite influencing pipeline materially.

7. **Operational data is in spreadsheets.** SLA performance, infrastructure cost allocation, and on-call burden are tracked manually in Google Sheets with weekly update cycles and no alerting.

8. **No early warning for churn.** The CS team identifies churn risk reactively — when a customer declines a renewal call or escalates a complaint. Multiple historical churn events (e.g., Meridian Healthcare, $340K ARR) had identifiable leading indicators that were not joined together.

---

## 5. Current Workarounds

| Problem | Current Workaround | Cost |
|---|---|---|
| No integrated data | Manual CSV exports + VLOOKUP reconciliation | James Petit spends ~60% of time on this |
| No product usage in CS | CSMs query engineers directly or rely on quarterly NPS surveys | Slow, incomplete, engineer burden |
| Marketing attribution | HubSpot last-touch only; Niamh Collins does weekly manual HubSpot–Salesforce CSV reconciliation | 12% contact mismatch rate, LinkedIn invisible |
| MRR inconsistency | Each team maintains own spreadsheet with own calculation logic | 8% discrepancy, board confidence undermined |
| SLA tracking | Google Sheet updated once per week by support analyst | No intraday visibility, no alerting |
| CS health scoring | ChurnZero login-based score + CSM gut feel + QBR touchpoints | Reactive, unreliable, no early warning |
| QBR preparation | CSMs manually pull data from 4+ systems; takes 3–5 hours per QBR | 14 CSMs × multiple QBRs per year |

---

## 6. What "Solved" Looks Like

**Core Dynamics considers this problem solved when:**

1. **A single governed data warehouse (BigQuery)** receives automated feeds from all 18 in-scope source systems, with Dataform transformations producing consistent staging → intermediate → warehouse models. Dataform assertions pass for 3 consecutive daily runs with < 0.1% error rate.

2. **Agreed metric definitions** for MRR, NRR, GRR, CAC, and Product Engagement Score are codified in the Looker semantic layer, signed off by each workstream sponsor, and return the same number in every dashboard and report. The 8% MRR discrepancy is eliminated.

3. **The CS team can identify at-risk accounts proactively.** A Customer Health Command Centre dashboard surfaces the top 3 contributing risk signals per account (not just a single score) and a BigQuery ML churn probability model (AUC-ROC ≥ 0.75) provides a renewal probability for accounts renewing in the next 180 days. CSMs can see their full book of business with RAG indicators and receive Looker Alerts for threshold breaches.

4. **Marketing has multi-touch attribution.** Five attribution models (first-touch, last-touch, linear, time-decay, U-shaped) are calculated in the warehouse and selectable in the Looker dashboard. LinkedIn, Meta, and SDR outreach touchpoints are visible in closed-won journeys. The 90-day marketing-influenced attribution window is extended to 180 days.

5. **The product team has account-level usage visibility.** Feature adoption (at Module → Feature Group → Feature hierarchy), daily product engagement scores, and onboarding funnel completion are available in Looker, filterable by account, segment, and cohort. No engineer queries are required.

6. **Operations dashboards provide intraday SLA visibility.** Support SLA compliance (P1/P2/P3 by customer tier) refreshes every 4 hours. GCP infrastructure cost is allocated per customer daily. PagerDuty MTTA/MTTR trends are visible weekly.

7. **CSM QBR preparation is one-click.** A Looker Action on the CSM Book of Business dashboard generates a pre-populated Google Slides deck for any account, reducing QBR prep time from 3–5 hours to ~1 hour.

---

## 7. Constraints

| Constraint Type | Detail |
|---|---|
| Budget | $250,000 total engagement fee across 6 payment milestones |
| Timeline | 22 weeks (Phase 1: Wks 1–4, Phase 2: Wks 5–14, Phase 3: Wks 15–22) |
| Technology | Google Cloud (BigQuery, Cloud Composer, Dataform, Pub/Sub); Fivetran Business Critical; Looker Standard/Enterprise (30+ Standard users, 5 Developer users) |
| Legal / Compliance | PII pseudonymisation in CoreFM PostgreSQL staging layer requires sign-off from legal counsel (Diane Hooper) before implementation. GDPR/CCPA compliance review is out of scope. |
| Data history | At least 24 months of historical renewal data required for BQML churn model. ChurnZero only has 14 months — Salesforce opportunity data will be used as proxy for pre-ChurnZero period. |
| Access dependencies | GCP Owner/Editor roles needed by end of Week 1; CoreFM PostgreSQL network access (Cloud SQL Auth Proxy) needed before Phase 2; NetSuite integration role can take 2–3 weeks |
| Rittman Analytics team | Mark Rittman (Engagement Lead), Sophie Tanner (Sr Analytics Engineer), Kofi Asante (Data Engineer), Irina Volkov (Looker Developer), Daniel Osei (PM) |
| Stakeholder availability | Workstream sponsors (Rachel Summers, Claire Ashworth, Tara Obinna, Harriet Drummond) must be available for 1-hour requirements review (Week 1) and 2-hour UAT session (Phase 3) |

---

## 8. Previously Ruled Out

| Exclusion | Reason |
|---|---|
| Real-time streaming dashboards (sub-minute latency) | Out of scope; all dashboards refresh on 4-hour to daily schedules |
| Self-service ETL tooling / data catalogue (Dataplex, Alation) | Future phase; runbook covers operational documentation only |
| Predictive modelling beyond churn risk | Expansion propensity, LTV prediction, lead scoring explicitly out of scope |
| Salesforce / HubSpot / Zendesk configuration changes | Read-only access; no modifications to source system schemas or workflows |
| GDPR/CCPA compliance review or legal sign-off | Client legal team's responsibility; pseudonymisation implemented but not compliance-certified |
| Mobile or embedded analytics | Future phase; all Looker content via Looker web application only |
| Ongoing managed service | Post-engagement retainer can be scoped separately; this engagement is build and handover only |
| Training beyond 3 knowledge transfer sessions | Explicitly out of scope per SOW Section 10 |

---

## 9. Impact Table

| Current State | Desired State | Signal of Success |
|---|---|---|
| MRR reported with 8% variance between functions | Single governed MRR definition in Looker semantic layer | ±2% tolerance met; board reporting uses single figure |
| Analyst spends 60% of time on manual exports | Automated pipelines; analyst focuses on analysis | James Petit spends < 20% of time on data wrangling |
| CS identifies churn reactively | At-risk accounts surfaced 60–90 days before renewal with explainable signals | CSMs act on model alerts; churn rate improves |
| Marketing optimises on MQLs only | Multi-touch attribution visible for all closed-won deals | Pipeline-to-spend ratio optimisation replaces CPC/MQL optimisation |
| Product usage inaccessible to non-engineers | Account-level feature adoption in Looker, no engineer required | CS uses product score in weekly account reviews |
| SLA tracked once per week in spreadsheet | SLA compliance dashboard refreshing every 4 hours | Carlos Vega identifies SLA breaches before they occur |
| QBR prep takes 3–5 hours per account | One-click Looker Action generates pre-populated Google Slides deck | QBR prep time < 1 hour per account |
| NRR at 104% | Data-driven CS motion enables NRR improvement | NRR reaches 115% within 18 months of board commitment |

---

## 10. Open Questions

1. **Salesforce-HubSpot contact deduplication (R-01):** The 12% unmatched contact rate will reduce attribution coverage. Will Core Dynamics pursue a Salesforce data cleanup sprint to improve match rate, or accept the coverage gap?

2. **BQML training data (R-03):** With ChurnZero history limited to 14 months, the churn model will use Salesforce opportunity data as a proxy for the pre-ChurnZero period. Does Tara Obinna accept this approach, and has she reviewed the model card implications?

3. **Product engagement score weights (R-11):** The discovery session surfaced potential revisions to the engagement score weighting. Has the revised weighting been agreed by both Claire Ashworth and Tara Obinna?

4. **GCP Billing label coverage (R-05):** Sean Murphy needs to provide a label coverage audit before the operations discovery session to properly scope the infrastructure cost-per-customer dashboard (DO-02). Has this been completed?

5. **Segment / Mixpanel event deduplication (R-06):** Leon Yip needs to provide an event-to-source mapping to enable deduplication rules at the staging layer. Is this mapping available?

---

*Document status: Generated by Wire Autopilot — self-reviewed and approved*
*Reviewed by: Wire Autopilot (self-review)*
*Date: 2026-03-29*
