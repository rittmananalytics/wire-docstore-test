# Problem Definition
## Core Dynamics — data_platform_build

**Prepared by:** Wire Autopilot
**Date:** 2026-03-29
**Engagement:** data_platform_build
**Client:** Core Dynamics, Inc.
**Engagement Reference:** RA-2026-0041

---

## 1. Executive Summary

Core Dynamics, Inc. is a Series C B2B SaaS company ($38.4M ARR, 312 enterprise accounts) with a board-mandated target to increase Net Revenue Retention from 104% to 115% within 18 months. That target cannot be reached through improved instinct or manual effort — it requires data. Specifically, it requires knowing in advance which customers are at risk, which marketing investments generate revenue, which product features drive adoption and retention, and whether the operations team is meeting its SLA commitments.

None of those questions can currently be answered reliably. Data exists — across Salesforce, HubSpot, ChurnZero, Mixpanel, the CoreFM production database, Zendesk, NetSuite, and a dozen other systems — but it is fragmented, unreconciled, and operationally inaccessible to anyone without engineering skills and hours to spare.

This problem definition frames the full scope of that data deficit, identifies the stakeholders blocked by it, and establishes measurable criteria for what "solved" looks like.

---

## 2. Who Has the Problem

**Primary stakeholders — blocked by missing or unreliable data:**

| Stakeholder | Role | Primary Data Deficit |
|---|---|---|
| Tara Obinna | VP Customer Success | No early warning for at-risk accounts; ChurnZero health scores ignore product usage, support history, and financial signals |
| Rachel Summers | VP Marketing | Multi-touch attribution is invisible; $2.1M demand gen budget optimised only on last-touch MQL conversion |
| Claire Ashworth | VP Product | Product usage telemetry is engineer-only; no account-level feature adoption visibility for roadmap decisions |
| Harriet Drummond | COO | SLA compliance and infrastructure cost tracked manually in spreadsheets; no intraday operational view |
| Marcus Elwood | CEO | Board-level NRR reporting uses inconsistent numbers; no single source of truth |
| Amara Diallo | Data Engineering Lead | No formal data infrastructure to own or maintain; pipelines are ad-hoc scripts on developer laptops |
| James Petit | Finance Analyst | Spends ~60% of time on manual CSV exports and VLOOKUP reconciliations |

**Secondary stakeholders (downstream beneficiaries):** 14 CSMs (need per-account tooling), Owen Brady (Demand Gen), Ben Tran (CS Ops), Carlos Vega (Support Engineering Head), Sean Murphy (Platform Engineering), Leon Yip (Senior PM).

---

## 3. What They Are Trying to Do

Stakeholders across Core Dynamics are trying to manage a growing enterprise SaaS business with data that simply does not reach them in a usable form. The specific jobs-to-be-done:

- **CS team:** Identify accounts at risk of churning *before* they decline a renewal call, and surface expansion opportunities *proactively* — not reactively, and not from gut feel or quarterly touchpoints alone. As Tara Obinna stated: "We almost never identify risk proactively — we tend to find out when the account either raises a formal concern or when they decline the renewal call. By then it's usually too late."

- **Marketing team:** Optimise a $2.1M annual demand gen budget by understanding which channels and campaigns generate *revenue*, not just MQL volume. As Owen Brady described: "I know LinkedIn is working — but I can't prove it in a spreadsheet, which means it's always the first thing on the chopping block when marketing budgets get reviewed."

- **Product team:** Understand which CoreFM features are actually being used, by which accounts, at what depth — and use that signal to prioritise the product roadmap and identify accounts not realising core product value.

- **Finance / Executive:** Report a single, agreed NRR/MRR/CAC figure to the board. As Rachel Summers described: "I had a situation last quarter where I presented CAC to the board and got challenged by Sandra's team because their number was different... it was embarrassing."

- **Operations team:** Know in near real-time whether SLA commitments are being met, how GCP infrastructure costs are tracking per customer, and whether on-call burden is equitably distributed.

---

## 4. What Is in the Way

Eight distinct barriers prevent Core Dynamics stakeholders from having the data they need:

1. **No data warehouse.** All source systems are siloed. Joining Salesforce revenue data to ChurnZero health scores to Mixpanel product events to Zendesk tickets requires manual exports and VLOOKUP reconciliation — which only one analyst (James Petit) performs, at the cost of 60% of his working time.

2. **No governed metric definitions.** MRR figures differ by up to 8% between Sales, Finance, and Customer Success — each team applies different calculation logic and refresh cadences. The board has flagged this as a governance risk. There is no agreed single source of truth.

3. **Product data is engineer-only.** CoreFM generates rich product usage telemetry but it sits in a production PostgreSQL database, accessible only to engineers who can write SQL directly against the production replica. The CS team has no product usage visibility; the product team has no account-level data.

4. **ChurnZero health scores are unreliable.** ChurnZero scores are based primarily on login events. They do not incorporate feature-level usage, support escalation history, NPS trends, or renewal signals. CSMs report low confidence in the scores and fall back on gut feel.

5. **Marketing attribution is last-touch only.** HubSpot's native attribution credits the last click before MQL conversion. Paid social channels (LinkedIn, Meta) that operate earlier in the funnel receive effectively no attribution credit. The 12% contact mismatch between HubSpot and Salesforce means that 12% of marketing journeys are invisible to any attribution model.

6. **No early churn warning system.** Historical churn events — including a $340K ARR loss (Meridian Healthcare) — had visible leading indicators (declining product usage, support escalations, champion departure) that were never joined together. There is no mechanism to do so.

7. **Operational data is in spreadsheets.** SLA performance is tracked in a Google Sheet updated once per week. Infrastructure cost allocation is done manually by Sean Murphy monthly. PagerDuty incidents have no trend analysis. There is no intraday operational view.

8. **QBR preparation consumes CSM time.** Each QBR requires a CSM to manually pull data from Salesforce, ChurnZero, Zendesk, and Intercom — a 3–5 hour process per account, per QBR cycle.

---

## 5. Current Workarounds

| Problem | Current Workaround | Documented Cost |
|---|---|---|
| No integrated data | Manual CSV exports + VLOOKUP reconciliation | James Petit: ~60% of time |
| No CS health data | CSMs query engineers or rely on quarterly NPS surveys (6-month lag) | Reactive churn management; $340K Meridian loss cited |
| Marketing attribution | HubSpot last-touch only; Niamh Collins does weekly manual HubSpot–Salesforce reconciliation | 12% contact mismatch; LinkedIn contribution invisible |
| MRR inconsistency | Each function maintains own spreadsheet calculation | 8% variance; board confidence undermined |
| SLA tracking | Google Sheet updated weekly by support analyst | No intraday visibility; no alerting before breach |
| Product usage in CS | Manual engineer queries; QBR-to-QBR data gaps | At-risk accounts missed; expansion blind spots |
| QBR preparation | CSMs pull data from 4+ systems manually | 3–5 hours per QBR per CSM; 14 CSMs |

---

## 6. What "Solved" Looks Like

The problem is solved when the following measurable conditions are met:

1. **A single governed data warehouse** receives automated feeds from all 18 source systems. Fivetran connectors maintain < 0.1% error rate over a 7-day observation period. Dataform assertions pass for 3 consecutive daily runs.

2. **MRR, NRR, GRR, and CAC are defined once** in the Looker semantic layer, signed off by all workstream sponsors, and return the same number across every dashboard, alert, and API query. The 8% MRR discrepancy is eliminated (within ±2% tolerance to account for methodology improvements).

3. **At-risk accounts are surfaced before they churn.** The Customer Health Command Centre shows the top 3 contributing risk signals per at-risk account (not just a score), enabling CSMs to take specific action. A BigQuery ML churn model achieves AUC-ROC ≥ 0.75 and precision ≥ 60% at a 0.5 threshold, providing renewal probabilities for accounts due in the next 180 days.

4. **Marketing has multi-touch attribution across 5 models.** All five models (first-touch, last-touch, linear, time-decay, U-shaped) are calculated in the warehouse and selectable in Looker. U-shaped is the default (per Rachel Summers' requirement). The marketing-influenced attribution window is 180 days (extended from the initial 90-day assumption). LinkedIn and Meta contributions are visible in closed-won journeys.

5. **Product usage is self-service for non-engineers.** Feature adoption (at Module → Feature Group → Feature hierarchy), daily engagement scores, and onboarding funnel completion are available in Looker filtered by account, segment, and cohort. No engineer query is required.

6. **SLA compliance is visible every 4 hours.** The Support Operations SLA Tracker refreshes intraday. GCP infrastructure cost is allocated per customer daily. MTTA/MTTR trends are tracked weekly.

7. **CSM QBR preparation is one-click.** A Looker Action generates a pre-populated Google Slides deck for any account, reducing QBR prep from 3–5 hours to under 1 hour.

---

## 7. Constraints

| Type | Constraint |
|---|---|
| **Budget** | $250,000 total (fixed price). Out-of-scope rate: $2,200/day. |
| **Timeline** | 22 weeks: Phase 1 (Wks 1–4), Phase 2 (Wks 5–14), Phase 3 (Wks 15–22). |
| **Technology** | Google Cloud (BigQuery, Dataform, Cloud Composer 2, Pub/Sub, Secret Manager). Fivetran Business Critical. Looker Standard/Enterprise (30+ Standard users, 5 Developer users). |
| **Legal / PII** | CoreFM PostgreSQL user PII must be pseudonymised at the staging layer (one-way hash on `user_id`). Implementation requires written sign-off from Diane Hooper (legal counsel) — target end of Week 2. |
| **Data history** | BigQuery ML churn model requires 24 months of renewal outcome data. ChurnZero only has 14 months of history — Salesforce opportunity data will be used as a proxy for the pre-ChurnZero period. |
| **Access dependencies** | GCP Owner/Editor roles required by end of Week 1. CoreFM PostgreSQL network access (Cloud SQL Auth Proxy) required before Phase 2. NetSuite integration role can take 2–3 weeks to provision. |
| **Stakeholder availability** | Workstream sponsors (Rachel Summers, Claire Ashworth, Tara Obinna, Harriet Drummond) must be available for 1-hour requirements review (Week 1) and 2-hour UAT sessions (Phase 3). |
| **RA team capacity** | Fixed team: Mark Rittman (Engagement Lead), Sophie Tanner (Sr Analytics Engineer), Kofi Asante (Data Engineer), Irina Volkov (Looker Developer), Daniel Osei (PM). No additional resources within scope. |

---

## 8. Previously Ruled Out

The following are explicitly excluded from this engagement, based on SOW Section 10 and discovery session discussions:

| Exclusion | Source |
|---|---|
| Real-time streaming dashboards (sub-minute latency) | SOW Section 10 |
| Data catalogue or data discovery tooling (Dataplex, Alation) | SOW Section 10 |
| Predictive models beyond churn risk (expansion propensity, LTV prediction, lead scoring) | SOW Section 10 |
| Source system configuration changes (Salesforce, HubSpot, Zendesk schemas/workflows) | SOW Section 10 |
| GDPR/CCPA compliance review or legal sign-off (beyond pseudonymisation implementation) | SOW Section 10 |
| Mobile or embedded analytics (Looker web app only) | SOW Section 10 |
| Ongoing managed service post-engagement | SOW Section 10 |
| Additional training beyond 3 knowledge transfer sessions | SOW Section 10 |
| Salesforce data cleanup or HubSpot-Salesforce deduplication remediation | Discovery — client responsibility (R-01) |
| Retroactive UTM parameter remediation for historical campaigns | Discovery — Niamh Collins to address going forward (R-04) |

---

## 9. Impact Table

| Current State | Desired State | Measurable Signal |
|---|---|---|
| MRR reported with up to 8% variance between functions | Single governed MRR definition in Looker; ±2% tolerance | Board receives single, agreed figure; discrepancy eliminated |
| James Petit spends ~60% of time on manual data work | Automated pipelines; analyst freed for analysis | < 20% of time on data wrangling (self-reported) |
| Churn identified reactively at renewal call | At-risk accounts surfaced 60–90 days in advance with top-3 contributing signals | CSMs act on model alerts; $340K-class losses contested earlier |
| Marketing optimises on MQLs, last-touch attribution only | Multi-touch attribution with 5 models; pipeline-per-spend visible | Budget allocation decisions based on revenue attribution |
| Product usage data inaccessible without an engineer query | Account-level feature adoption in Looker, self-service | CS uses product score in weekly account reviews |
| SLA tracked weekly in a spreadsheet | SLA compliance dashboard refreshing every 4 hours | Breaches identified and actioned before end-of-day |
| QBR prep takes 3–5 hours per account | One-click Looker Action generates pre-populated Google Slides | QBR prep < 1 hour per account (self-reported by CSMs) |
| NRR at 104% | Data-driven CS and marketing motions power NRR growth | NRR reaches 115% within 18 months of board commitment |

---

## 10. Open Questions

The following questions require client confirmation before or during delivery:

1. **Salesforce-HubSpot contact deduplication (R-01):** The 12% contact mismatch rate will reduce marketing attribution coverage. Will Core Dynamics pursue a Salesforce data cleanup sprint, or accept the documented coverage gap in attribution reporting?

2. **BQML training data approach (R-03):** The churn model will use Salesforce opportunity history as a proxy for pre-ChurnZero renewal data. Has Tara Obinna reviewed and accepted this approach and its implications for model accuracy?

3. **Product engagement score weights (R-11):** Post-discovery, revised weighting across the 5 signals (DAU/MAU, feature breadth, core activation, NPS, onboarding) needs sign-off from both Claire Ashworth and Tara Obinna before the `dim_account_product_score` model is built.

4. **GCP Billing label coverage (R-05):** Sean Murphy's label coverage audit is needed to correctly scope the Infrastructure Cost & Efficiency dashboard (DO-02). Has this audit been completed?

5. **Segment/Mixpanel event deduplication (R-06):** Leon Yip needs to provide the event-to-source mapping to enable deduplication rules in the staging layer. When will this mapping be available?

6. **`User_Seats__c` Salesforce field completeness:** Leon Yip flagged that the `User_Seats__c` Salesforce field (needed for licence utilisation metric) may be null for older contracts. Amara Diallo to confirm field population rate before the licence utilisation metric is built.

---

*Document status: Generated by Wire Autopilot — validated and self-approved*
*Reviewed by: Wire Autopilot (self-review)*
*Review date: 2026-03-29*
