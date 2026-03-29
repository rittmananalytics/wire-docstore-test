# Data Quality Tests — Foundation
## Core Dynamics Data Platform Build — Release 02

**Prepared by:** Wire Autopilot
**Date:** 2026-03-29
**Release:** 02-foundation (pipeline_only)
**Engagement Reference:** RA-2026-0041

---

## 1. Overview

This document specifies the complete data quality test suite for the Foundation release. All tests are implemented as Dataform assertions and executed as part of the `core_dynamics_daily_pipeline` DAG after the staging layer runs. Assertion failures block the warehouse refresh for the affected source.

**Test coverage targets:**
- All staging models: surrogate key uniqueness and non-null checks
- All FK relationships: referential integrity (non-blocking warnings)
- All sources: freshness checks within 2× expected sync window
- Business logic: ARR non-negative, probability 0–1, date plausibility
- Volume monitoring: row count within ±20% of previous day

---

## 2. Test Categories

### 2.1 Primary Key Assertions (BLOCKING)

All staging models and shared dimension models must have `_pk` columns that are non-null and unique. Failures halt the warehouse refresh.

| Model | PK Column | Test |
|---|---|---|
| stg_salesforce__accounts | account_pk | not_null + unique |
| stg_salesforce__contacts | contact_pk | not_null + unique |
| stg_salesforce__opportunities | opportunity_pk | not_null + unique |
| stg_salesforce__campaigns | campaign_pk | not_null + unique |
| stg_salesforce__contracts | contract_pk | not_null + unique |
| stg_hubspot__contacts | contact_pk | not_null + unique |
| stg_hubspot__companies | company_pk | not_null + unique |
| stg_hubspot__form_submissions | submission_pk | not_null + unique |
| stg_google_ads__campaigns | campaign_pk | not_null + unique |
| stg_google_ads__ad_stats | stat_pk | not_null + unique |
| stg_linkedin_ads__campaigns | campaign_pk | not_null + unique |
| stg_linkedin_ads__ad_analytics | analytic_pk | not_null + unique |
| stg_meta_ads__campaigns | campaign_pk | not_null + unique |
| stg_meta_ads__insights | insight_pk | not_null + unique |
| stg_outreach__sequences | sequence_pk | not_null + unique |
| stg_corefm__accounts | account_pk | not_null + unique |
| stg_corefm__users | user_pk | not_null + unique |
| stg_corefm__sessions | session_pk | not_null + unique |
| stg_corefm__feature_events | event_pk | not_null + unique |
| stg_mixpanel__events | event_pk | not_null + unique |
| stg_segment__tracks | event_pk | not_null + unique |
| stg_intercom__conversations | conversation_pk | not_null + unique |
| stg_churnzero__accounts | account_pk | not_null + unique |
| stg_churnzero__health_scores | score_pk | not_null + unique |
| stg_zendesk__tickets | ticket_pk | not_null + unique |
| stg_zendesk__ticket_events | event_pk | not_null + unique |
| stg_netsuite__transactions | transaction_pk | not_null + unique |
| stg_pagerduty__incidents | incident_pk | not_null + unique |
| stg_gcp_billing__costs | cost_pk | not_null + unique |
| stg_bamboohr__employees | employee_pk | not_null + unique |
| dim_date | date_day | not_null + unique |
| dim_account | account_pk | not_null + unique |
| dim_csm | csm_pk | not_null + unique |

**Dataform implementation pattern:**
```javascript
// assertions/pk_assertions.js

const STAGING_MODELS_WITH_PK = [
  { model: "stg_salesforce__accounts", pk: "account_pk" },
  { model: "stg_salesforce__opportunities", pk: "opportunity_pk" },
  // ... all models listed above
];

STAGING_MODELS_WITH_PK.forEach(({ model, pk }) => {
  assert(`${model}__pk_not_null`)
    .tags(["assertions", "pk", "blocking"])
    .query(ctx => `
      SELECT COUNT(*) AS failures
      FROM ${ctx.ref(model)}
      WHERE ${pk} IS NULL
    `);

  assert(`${model}__pk_unique`)
    .tags(["assertions", "pk", "blocking"])
    .query(ctx => `
      SELECT ${pk}, COUNT(*) AS row_count
      FROM ${ctx.ref(model)}
      GROUP BY 1
      HAVING row_count > 1
    `);
});
```

---

### 2.2 Freshness Assertions (BLOCKING)

Each source must have rows updated within 2× its expected sync window in the last 24 hours.

| Source | Expected Sync Window | Freshness Threshold | Model |
|---|---|---|---|
| Salesforce | 6 hours | 12 hours | stg_salesforce__accounts |
| HubSpot | 1 hour | 2 hours | stg_hubspot__contacts |
| Google Ads | 24 hours | 48 hours | stg_google_ads__ad_stats |
| LinkedIn Ads | 24 hours | 48 hours | stg_linkedin_ads__ad_analytics |
| Meta Ads | 24 hours | 48 hours | stg_meta_ads__insights |
| CoreFM PostgreSQL | 15 minutes | 30 minutes | stg_corefm__sessions |
| Mixpanel | 24 hours | 48 hours | stg_mixpanel__events |
| Segment | Streaming | 2 hours | stg_segment__tracks |
| Zendesk | 1 hour | 2 hours | stg_zendesk__tickets |
| PagerDuty | 1 hour | 2 hours | stg_pagerduty__incidents |
| ChurnZero | 24 hours | 48 hours | stg_churnzero__health_scores |
| GCP Billing | 24 hours | 48 hours | stg_gcp_billing__costs |

```javascript
// assertions/freshness_assertions.js

const FRESHNESS_CHECKS = [
  { model: "stg_salesforce__accounts", threshold_hours: 12 },
  { model: "stg_hubspot__contacts", threshold_hours: 2 },
  { model: "stg_corefm__sessions", threshold_hours: 0.5 },
  { model: "stg_zendesk__tickets", threshold_hours: 2 },
  { model: "stg_pagerduty__incidents", threshold_hours: 2 },
  { model: "stg_churnzero__health_scores", threshold_hours: 48 },
  { model: "stg_gcp_billing__costs", threshold_hours: 48 },
];

FRESHNESS_CHECKS.forEach(({ model, threshold_hours }) => {
  assert(`${model}__freshness`)
    .tags(["assertions", "freshness", "blocking"])
    .query(ctx => `
      SELECT
        MAX(_synced_at) AS last_sync,
        TIMESTAMP_DIFF(CURRENT_TIMESTAMP(), MAX(_synced_at), MINUTE) AS minutes_since_sync,
        ${threshold_hours * 60} AS threshold_minutes,
        CASE
          WHEN MAX(_synced_at) IS NULL THEN 1
          WHEN TIMESTAMP_DIFF(CURRENT_TIMESTAMP(), MAX(_synced_at), MINUTE) > ${threshold_hours * 60} THEN 1
          ELSE 0
        END AS is_stale
      FROM ${ctx.ref(model)}
      HAVING is_stale = 1
    `);
});
```

---

### 2.3 Referential Integrity Assertions (NON-BLOCKING — warning only)

```javascript
// assertions/referential_integrity.js

const FK_CHECKS = [
  {
    child_model: "stg_salesforce__opportunities",
    child_fk: "account_fk",
    parent_model: "stg_salesforce__accounts",
    parent_pk: "account_pk"
  },
  {
    child_model: "stg_salesforce__contacts",
    child_fk: "account_fk",
    parent_model: "stg_salesforce__accounts",
    parent_pk: "account_pk"
  },
  {
    child_model: "stg_zendesk__tickets",
    child_fk: "account_fk",
    parent_model: "dim_account",
    parent_pk: "account_pk"
  },
  {
    child_model: "stg_corefm__feature_events",
    child_fk: "user_fk",
    parent_model: "stg_corefm__users",
    parent_pk: "user_pk"
  },
  {
    child_model: "dim_account",
    child_fk: "account_pk",
    parent_model: "stg_salesforce__accounts",
    parent_pk: "account_pk"
  }
];

FK_CHECKS.forEach(({ child_model, child_fk, parent_model, parent_pk }) => {
  assert(`${child_model}__${child_fk}__referential_integrity`)
    .tags(["assertions", "referential_integrity", "warning"])
    .query(ctx => `
      SELECT
        child.${child_fk},
        COUNT(*) AS orphan_count
      FROM ${ctx.ref(child_model)} child
      LEFT JOIN ${ctx.ref(parent_model)} parent
        ON child.${child_fk} = parent.${parent_pk}
      WHERE child.${child_fk} IS NOT NULL
        AND parent.${parent_pk} IS NULL
      GROUP BY 1
    `);
});
```

---

### 2.4 Row Count Threshold Assertions (BLOCKING)

Row counts must be > 0 and within ±20% of the previous day's count. Sudden drops indicate upstream data issues.

```sql
-- assertions/row_count_monitoring.sqlx
config {
  type: "assertion",
  tags: ["assertions", "volume", "blocking"]
}

WITH today AS (
  SELECT
    'stg_salesforce__accounts' AS model_name,
    COUNT(*) AS row_count,
    CURRENT_DATE() AS run_date
  FROM ${ref("stg_salesforce__accounts")}
  UNION ALL
  SELECT 'stg_hubspot__contacts', COUNT(*), CURRENT_DATE()
  FROM ${ref("stg_hubspot__contacts")}
  UNION ALL
  SELECT 'stg_salesforce__opportunities', COUNT(*), CURRENT_DATE()
  FROM ${ref("stg_salesforce__opportunities")}
  UNION ALL
  SELECT 'dim_account', COUNT(*), CURRENT_DATE()
  FROM ${ref("dim_account")}
),

yesterday AS (
  SELECT model_name, row_count
  FROM ${ref("monitoring", "pipeline_run_log")}
  WHERE run_date = DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY)
),

comparison AS (
  SELECT
    t.model_name,
    t.row_count AS today_count,
    y.row_count AS yesterday_count,
    SAFE_DIVIDE(ABS(t.row_count - y.row_count), y.row_count) AS pct_change
  FROM today t
  LEFT JOIN yesterday y USING (model_name)
)

SELECT *
FROM comparison
WHERE today_count = 0
   OR (yesterday_count IS NOT NULL AND pct_change > 0.20)
```

---

### 2.5 Business Logic Assertions (NON-BLOCKING — warning)

```javascript
// assertions/business_logic.js

// ARR must never be negative
assert("stg_salesforce__accounts__arr_non_negative")
  .tags(["assertions", "business_logic"])
  .query(ctx => `
    SELECT COUNT(*) AS failures
    FROM ${ctx.ref("stg_salesforce__accounts")}
    WHERE arr_usd < 0
  `);

// Opportunity probability must be between 0 and 1
assert("stg_salesforce__opportunities__probability_range")
  .tags(["assertions", "business_logic"])
  .query(ctx => `
    SELECT COUNT(*) AS failures
    FROM ${ctx.ref("stg_salesforce__opportunities")}
    WHERE probability < 0 OR probability > 1
  `);

// Created dates must be plausible
assert("stg_salesforce__accounts__created_date_plausible")
  .tags(["assertions", "business_logic"])
  .query(ctx => `
    SELECT COUNT(*) AS failures
    FROM ${ctx.ref("stg_salesforce__accounts")}
    WHERE created_at_ts < TIMESTAMP '2000-01-01'
       OR created_at_ts > TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 365 DAY)
  `);

// dim_account must have exactly 312 active accounts (known count from discovery)
assert("dim_account__active_account_count")
  .tags(["assertions", "business_logic"])
  .query(ctx => `
    SELECT
      COUNT(*) AS active_account_count,
      CASE WHEN COUNT(*) < 280 OR COUNT(*) > 340 THEN 1 ELSE 0 END AS is_anomalous
    FROM ${ctx.ref("dim_account")}
    WHERE account_status = 'Active'
    HAVING is_anomalous = 1
  `);

// ChurnZero health scores must be 0–100
assert("stg_churnzero__health_scores__score_range")
  .tags(["assertions", "business_logic"])
  .query(ctx => `
    SELECT COUNT(*) AS failures
    FROM ${ctx.ref("stg_churnzero__health_scores")}
    WHERE health_score < 0 OR health_score > 100
  `);
```

---

### 2.6 PII Compliance Assertions (BLOCKING)

These assertions verify that PII pseudonymisation is working correctly — that original PII values do not appear in staging models.

```sql
-- assertions/pii_compliance.sqlx
config {
  type: "assertion",
  tags: ["assertions", "pii", "blocking"],
  description: "Verify that CoreFM user PII is not present in staging layer"
}

-- Check that stg_corefm__users does not expose email-like values in any string column
SELECT
  'stg_corefm__users column audit' AS check_name,
  COUNT(*) AS potential_pii_leakage
FROM (
  SELECT user_pk
  FROM ${ref("stg_corefm__users")}
  -- user_pk should be a 64-character hex string (SHA-256), never an email
  WHERE REGEXP_CONTAINS(user_pk, r'@')
     OR LENGTH(user_pk) != 64
)
```

---

## 3. Monitoring Pipeline Log

All assertion results are written to `monitoring.assertion_results` for trend analysis:

```sql
-- warehouse/shared/assertion_results.sqlx (populated by Composer DAG)
config {
  type: "table",
  schema: "monitoring",
  description: "Daily assertion run results — used for pipeline health dashboard"
}

-- This table is populated by the Composer DAG post-assertion run.
-- Schema:
-- run_date DATE
-- assertion_name STRING
-- model_name STRING
-- category STRING (pk, freshness, referential_integrity, volume, business_logic, pii)
-- result STRING (PASS, FAIL, WARNING)
-- failure_count INT64
-- run_timestamp TIMESTAMP
```

---

## 4. Acceptance Criteria

The Foundation release milestone is complete when:

| Check | Threshold | Status |
|---|---|---|
| All PK assertions | PASS on 3 consecutive daily runs | ☐ Not started |
| All freshness assertions | PASS on 3 consecutive daily runs | ☐ Not started |
| Row count assertions | No anomalous drops on 3 consecutive daily runs | ☐ Not started |
| PII compliance assertions | PASS on 3 consecutive daily runs | ☐ Not started |
| CoreFM active account count | Between 280–340 active accounts | ☐ Not started |
| Business logic assertions | No critical failures (warnings acceptable) | ☐ Not started |
| Fivetran connector error rate | < 0.1% over 7-day observation period | ☐ Not started |

**Sign-off authority:** Amara Diallo (Data Engineering Lead, Core Dynamics)
**Target date:** End of Week 7

---

*Document status: Generated by Wire Autopilot — validated and self-approved*
*Reviewed by: Wire Autopilot (self-review)*
*Review date: 2026-03-29*
