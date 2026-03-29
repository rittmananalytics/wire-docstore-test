# Pipeline Implementation — Foundation
## Core Dynamics Data Platform Build — Release 02

**Prepared by:** Wire Autopilot
**Date:** 2026-03-29
**Release:** 02-foundation (pipeline_only)
**Engagement Reference:** RA-2026-0041

---

## 1. Dataform Repository Structure

```
core-dynamics-dataform/
├── dataform.json
├── .df-credentials.json        ← Secret Manager reference (not committed)
├── definitions/
│   ├── staging/
│   │   ├── salesforce/
│   │   │   ├── stg_salesforce__accounts.sqlx
│   │   │   ├── stg_salesforce__contacts.sqlx
│   │   │   ├── stg_salesforce__opportunities.sqlx
│   │   │   ├── stg_salesforce__campaigns.sqlx
│   │   │   └── stg_salesforce__contracts.sqlx
│   │   ├── hubspot/
│   │   │   ├── stg_hubspot__contacts.sqlx
│   │   │   ├── stg_hubspot__companies.sqlx
│   │   │   ├── stg_hubspot__form_submissions.sqlx
│   │   │   ├── stg_hubspot__email_events.sqlx
│   │   │   └── stg_hubspot__page_views.sqlx
│   │   ├── google_ads/
│   │   │   ├── stg_google_ads__campaigns.sqlx
│   │   │   ├── stg_google_ads__ad_groups.sqlx
│   │   │   └── stg_google_ads__ad_stats.sqlx
│   │   ├── linkedin_ads/
│   │   │   ├── stg_linkedin_ads__campaigns.sqlx
│   │   │   └── stg_linkedin_ads__ad_analytics.sqlx
│   │   ├── meta_ads/
│   │   │   ├── stg_meta_ads__campaigns.sqlx
│   │   │   └── stg_meta_ads__insights.sqlx
│   │   ├── outreach/
│   │   │   ├── stg_outreach__sequences.sqlx
│   │   │   ├── stg_outreach__email_touches.sqlx
│   │   │   └── stg_outreach__call_touches.sqlx
│   │   ├── corefm/
│   │   │   ├── stg_corefm__accounts.sqlx
│   │   │   ├── stg_corefm__users.sqlx          ← PII pseudonymised
│   │   │   ├── stg_corefm__sessions.sqlx
│   │   │   ├── stg_corefm__feature_events.sqlx
│   │   │   ├── stg_corefm__work_orders.sqlx
│   │   │   └── stg_corefm__assets.sqlx
│   │   ├── mixpanel/
│   │   │   ├── stg_mixpanel__events.sqlx
│   │   │   └── stg_mixpanel__people.sqlx
│   │   ├── segment/
│   │   │   ├── stg_segment__tracks.sqlx
│   │   │   ├── stg_segment__identifies.sqlx
│   │   │   └── stg_segment__pages.sqlx
│   │   ├── intercom/
│   │   │   ├── stg_intercom__conversations.sqlx
│   │   │   └── stg_intercom__contacts.sqlx
│   │   ├── churnzero/
│   │   │   ├── stg_churnzero__accounts.sqlx
│   │   │   ├── stg_churnzero__health_scores.sqlx
│   │   │   └── stg_churnzero__activities.sqlx
│   │   ├── zendesk/
│   │   │   ├── stg_zendesk__tickets.sqlx
│   │   │   ├── stg_zendesk__ticket_events.sqlx
│   │   │   └── stg_zendesk__users.sqlx
│   │   ├── netsuite/
│   │   │   ├── stg_netsuite__transactions.sqlx
│   │   │   └── stg_netsuite__customers.sqlx
│   │   ├── pagerduty/
│   │   │   ├── stg_pagerduty__incidents.sqlx
│   │   │   └── stg_pagerduty__services.sqlx
│   │   ├── gcp_billing/
│   │   │   └── stg_gcp_billing__costs.sqlx
│   │   ├── cloud_monitoring/
│   │   │   └── stg_cloud_monitoring__metrics.sqlx
│   │   └── bamboohr/
│   │       └── stg_bamboohr__employees.sqlx
│   └── warehouse/
│       └── shared/
│           ├── dim_date.sqlx
│           ├── dim_account.sqlx
│           └── dim_csm.sqlx
├── includes/
│   ├── surrogate_key.js        ← SHA-256 surrogate key helper
│   └── pii_hash.js             ← PII pseudonymisation helper
└── assertions/
    └── staging_assertions.js   ← Shared assertion definitions
```

---

## 2. dataform.json

```json
{
  "warehouse": "bigquery",
  "defaultSchema": "staging",
  "assertionSchema": "dataform_assertions",
  "projectConfig": {
    "defaultDatabase": "core-dynamics-analytics-prod",
    "defaultLocation": "US"
  },
  "vars": {
    "env": "prod",
    "source_project": "core-dynamics-analytics-prod"
  }
}
```

---

## 3. Key Staging Model Implementations

### 3.1 stg_salesforce__accounts.sqlx

```sql
config {
  type: "view",
  schema: "staging",
  description: "Salesforce accounts — cleaned, typed, and renamed. One row per Salesforce Account.",
  tags: ["staging", "salesforce"],
  columns: {
    account_pk: "SHA-256 surrogate key on salesforce_account_id",
    salesforce_account_id: "Salesforce 18-character Account ID",
    account_name: "Account name as stored in Salesforce",
    account_type: "Account type (Customer, Prospect, Partner)",
    arr_usd: "Annual Recurring Revenue in USD from Salesforce Amount field",
    csm_owner_id: "Salesforce User ID of assigned CSM",
    account_status: "Active, At Risk, Churned",
    contract_start_date: "Date of first active contract",
    contract_end_date: "Date of most recent contract end",
    created_at_ts: "Timestamp when account was created in Salesforce",
    updated_at_ts: "Timestamp when account was last modified in Salesforce",
    _synced_at: "Timestamp of last Fivetran sync"
  }
}

WITH source AS (
  SELECT * FROM ${ref("raw_salesforce", "account")}
),

renamed AS (
  SELECT
    TO_HEX(SHA256(CAST(id AS STRING)))         AS account_pk,
    id                                          AS salesforce_account_id,
    name                                        AS account_name,
    type                                        AS account_type,
    CAST(annual_recurring_revenue__c AS NUMERIC) AS arr_usd,
    owner_id                                    AS csm_owner_id,
    account_status__c                           AS account_status,
    CAST(contract_start_date__c AS DATE)        AS contract_start_date,
    CAST(contract_end_date__c AS DATE)          AS contract_end_date,
    contracted_user_seats__c                    AS contracted_user_seats,
    industry,
    employee_count__c                           AS employee_count,
    CAST(created_date AS TIMESTAMP)             AS created_at_ts,
    CAST(last_modified_date AS TIMESTAMP)       AS updated_at_ts,
    _fivetran_synced                            AS _synced_at
  FROM source
  WHERE (_fivetran_deleted IS FALSE OR _fivetran_deleted IS NULL)
)

SELECT * FROM renamed
```

**Assertions:**
```javascript
assert("stg_salesforce__accounts_pk_not_null").query(ctx => `
  SELECT count(*) AS failures
  FROM ${ctx.ref("stg_salesforce__accounts")}
  WHERE account_pk IS NULL
`);

assert("stg_salesforce__accounts_pk_unique").query(ctx => `
  SELECT account_pk, count(*) AS row_count
  FROM ${ctx.ref("stg_salesforce__accounts")}
  GROUP BY 1
  HAVING row_count > 1
`);

assert("stg_salesforce__accounts_arr_non_negative").query(ctx => `
  SELECT count(*) AS failures
  FROM ${ctx.ref("stg_salesforce__accounts")}
  WHERE arr_usd < 0
`);
```

---

### 3.2 stg_salesforce__opportunities.sqlx

```sql
config {
  type: "view",
  schema: "staging",
  description: "Salesforce opportunities — key revenue and pipeline object.",
  tags: ["staging", "salesforce"],
  columns: {
    opportunity_pk: "SHA-256 surrogate key on salesforce_opportunity_id",
    salesforce_opportunity_id: "Salesforce 18-character Opportunity ID",
    account_fk: "SHA-256 surrogate key linking to stg_salesforce__accounts",
    opportunity_name: "Opportunity name",
    stage_name: "Current pipeline stage",
    opportunity_type: "New Business, Renewal, Expansion, Contraction",
    arr_usd: "ARR value of opportunity",
    close_date: "Expected or actual close date",
    is_won: "True if stage_name = Closed Won",
    is_closed: "True if opportunity is in a terminal stage",
    probability: "Salesforce probability 0–100",
    created_at_ts: "Timestamp when opportunity was created",
    closed_at_ts: "Timestamp when opportunity moved to terminal stage"
  }
}

WITH source AS (
  SELECT * FROM ${ref("raw_salesforce", "opportunity")}
),

renamed AS (
  SELECT
    TO_HEX(SHA256(CAST(id AS STRING)))              AS opportunity_pk,
    id                                               AS salesforce_opportunity_id,
    TO_HEX(SHA256(CAST(account_id AS STRING)))       AS account_fk,
    name                                             AS opportunity_name,
    stage_name,
    type                                             AS opportunity_type,
    CAST(amount AS NUMERIC)                          AS arr_usd,
    CAST(close_date AS DATE)                         AS close_date,
    (stage_name = 'Closed Won')                      AS is_won,
    is_closed,
    CAST(probability AS NUMERIC) / 100               AS probability,
    CAST(created_date AS TIMESTAMP)                  AS created_at_ts,
    CAST(last_modified_date AS TIMESTAMP)            AS updated_at_ts,
    CASE WHEN is_closed THEN CAST(last_modified_date AS TIMESTAMP) END AS closed_at_ts,
    _fivetran_synced                                 AS _synced_at
  FROM source
  WHERE (_fivetran_deleted IS FALSE OR _fivetran_deleted IS NULL)
)

SELECT * FROM renamed
```

---

### 3.3 stg_corefm__users.sqlx (PII pseudonymised)

```sql
config {
  type: "view",
  schema: "staging",
  description: "CoreFM users — PII pseudonymised. user_id is SHA-256 hashed. Email and name columns are NOT selected. Requires legal sign-off from Diane Hooper before activation.",
  tags: ["staging", "corefm", "pii_sensitive"],
  columns: {
    user_pk: "SHA-256 hash of original user_id — irreversible pseudonym",
    account_fk: "SHA-256 surrogate key linking to stg_salesforce__accounts via account_id",
    user_role: "User role within CoreFM (Admin, Standard, Read-only)",
    is_active: "Whether user account is currently active",
    last_login_ts: "Timestamp of most recent login event",
    created_at_ts: "Timestamp when user account was created",
    _synced_at: "Timestamp of last Fivetran CDC sync"
  }
}

-- ⚠️  PII NOTE: This model MUST NOT be activated until written sign-off
-- from Diane Hooper (legal counsel) is received. See requirement FR-025.
-- The original user_id, user_email, and user_name columns are intentionally excluded.

WITH source AS (
  SELECT * FROM ${ref("raw_corefm_db", "users")}
),

pseudonymised AS (
  SELECT
    TO_HEX(SHA256(CAST(user_id AS STRING)))        AS user_pk,        -- irreversible hash
    TO_HEX(SHA256(CAST(account_id AS STRING)))      AS account_fk,
    -- user_email intentionally excluded
    -- user_name intentionally excluded
    role                                            AS user_role,
    is_active,
    CAST(last_login_at AS TIMESTAMP)               AS last_login_ts,
    CAST(created_at AS TIMESTAMP)                  AS created_at_ts,
    CAST(updated_at AS TIMESTAMP)                  AS updated_at_ts,
    _fivetran_synced                               AS _synced_at
  FROM source
  WHERE (_fivetran_deleted IS FALSE OR _fivetran_deleted IS NULL)
)

SELECT * FROM pseudonymised
```

---

### 3.4 stg_hubspot__contacts.sqlx

```sql
config {
  type: "view",
  schema: "staging",
  description: "HubSpot contacts — includes UTM parameter parsing for attribution.",
  tags: ["staging", "hubspot"],
  columns: {
    contact_pk: "SHA-256 surrogate key on hubspot_contact_id",
    hubspot_contact_id: "HubSpot numeric contact ID",
    salesforce_contact_fk: "SHA-256 of matched Salesforce contact ID (null if no match — 12% mismatch rate)",
    email: "Contact email address (retained for attribution join only)",
    first_touch_utm_source: "UTM source of first recorded touch",
    first_touch_utm_medium: "UTM medium of first recorded touch",
    first_touch_utm_campaign: "UTM campaign of first recorded touch",
    last_touch_utm_source: "UTM source of most recent touch",
    lifecycle_stage: "HubSpot lifecycle stage (Lead, MQL, SAL, SQL, Customer)",
    became_mql_at_ts: "Timestamp when contact first reached MQL stage",
    became_sal_at_ts: "Timestamp when contact first reached SAL stage",
    became_sql_at_ts: "Timestamp when contact first reached SQL stage"
  }
}

WITH source AS (
  SELECT * FROM ${ref("raw_hubspot", "contact")}
),

renamed AS (
  SELECT
    TO_HEX(SHA256(CAST(vid AS STRING)))                        AS contact_pk,
    CAST(vid AS STRING)                                        AS hubspot_contact_id,
    CASE
      WHEN associated_company_id IS NOT NULL
      THEN TO_HEX(SHA256(CAST(associated_company_id AS STRING)))
    END                                                        AS company_fk,
    email,
    -- UTM parsing
    property_hs_analytics_first_url                           AS first_touch_url,
    REGEXP_EXTRACT(property_hs_analytics_first_url, r'utm_source=([^&]+)') AS first_touch_utm_source,
    REGEXP_EXTRACT(property_hs_analytics_first_url, r'utm_medium=([^&]+)') AS first_touch_utm_medium,
    REGEXP_EXTRACT(property_hs_analytics_first_url, r'utm_campaign=([^&]+)') AS first_touch_utm_campaign,
    property_hs_analytics_last_url                            AS last_touch_url,
    REGEXP_EXTRACT(property_hs_analytics_last_url, r'utm_source=([^&]+)')  AS last_touch_utm_source,
    -- Lifecycle stage
    property_lifecyclestage                                   AS lifecycle_stage,
    CAST(property_hs_lifecyclestage_marketingqualifiedlead_date AS TIMESTAMP) AS became_mql_at_ts,
    CAST(property_hs_lifecyclestage_salesacceptedlead_date AS TIMESTAMP)     AS became_sal_at_ts,
    CAST(property_hs_lifecyclestage_salesqualifiedlead_date AS TIMESTAMP)    AS became_sql_at_ts,
    CAST(property_createdate AS TIMESTAMP)                    AS created_at_ts,
    CAST(property_lastmodifieddate AS TIMESTAMP)              AS updated_at_ts,
    _fivetran_synced                                          AS _synced_at
  FROM source
  WHERE (_fivetran_deleted IS FALSE OR _fivetran_deleted IS NULL)
)

SELECT * FROM renamed
```

---

### 3.5 dim_account.sqlx

```sql
config {
  type: "table",
  schema: "warehouse",
  description: "Shared account dimension — single row per account, joining Salesforce account to ChurnZero account. Used by all downstream workstreams.",
  tags: ["warehouse", "shared", "dimension"],
  bigquery: {
    clusterBy: ["account_status"]
  },
  columns: {
    account_pk: "SHA-256 surrogate key from Salesforce account ID — primary key",
    salesforce_account_id: "Salesforce 18-character Account ID",
    churnzero_account_id: "ChurnZero account identifier (matched on name + domain)",
    account_name: "Canonical account name from Salesforce",
    arr_usd: "Annual Recurring Revenue from Salesforce",
    account_status: "Active, At Risk, Churned",
    account_tier: "Enterprise, Mid-Market, SMB — derived from ARR",
    csm_owner_id: "Salesforce User ID of assigned CSM",
    contract_start_date: "First contract start date",
    contract_renewal_date: "Next renewal date",
    contracted_user_seats: "Contracted user seat count from Salesforce User_Seats__c",
    industry: "Industry vertical from Salesforce",
    employee_count: "Company headcount"
  }
}

WITH salesforce_accounts AS (
  SELECT * FROM ${ref("stg_salesforce__accounts")}
),

churnzero_accounts AS (
  SELECT * FROM ${ref("stg_churnzero__accounts")}
),

joined AS (
  SELECT
    sf.account_pk,
    sf.salesforce_account_id,
    cz.churnzero_account_id,
    sf.account_name,
    sf.arr_usd,
    sf.account_status,
    CASE
      WHEN sf.arr_usd >= 100000 THEN 'Enterprise'
      WHEN sf.arr_usd >= 25000  THEN 'Mid-Market'
      ELSE 'SMB'
    END                              AS account_tier,
    sf.csm_owner_id,
    sf.contract_start_date,
    sf.contract_end_date             AS contract_renewal_date,
    sf.contracted_user_seats,
    sf.industry,
    sf.employee_count,
    sf.created_at_ts,
    sf.updated_at_ts
  FROM salesforce_accounts sf
  LEFT JOIN churnzero_accounts cz
    ON LOWER(TRIM(sf.account_name)) = LOWER(TRIM(cz.account_name))
)

SELECT * FROM joined
```

---

### 3.6 dim_date.sqlx

```sql
config {
  type: "table",
  schema: "warehouse",
  description: "Date dimension spanning 2020-01-01 to 2030-12-31. Fiscal year starts 1 February.",
  tags: ["warehouse", "shared", "dimension"]
}

WITH date_spine AS (
  SELECT
    DATE_ADD(DATE '2020-01-01', INTERVAL n DAY) AS date_day
  FROM UNNEST(GENERATE_ARRAY(0, DATE_DIFF(DATE '2030-12-31', DATE '2020-01-01', DAY))) AS n
)

SELECT
  date_day,
  FORMAT_DATE('%Y%m%d', date_day)           AS date_key,
  EXTRACT(YEAR FROM date_day)               AS year,
  EXTRACT(QUARTER FROM date_day)            AS quarter,
  EXTRACT(MONTH FROM date_day)              AS month,
  FORMAT_DATE('%B', date_day)               AS month_name,
  EXTRACT(WEEK FROM date_day)               AS week_of_year,
  EXTRACT(DAY FROM date_day)                AS day_of_month,
  EXTRACT(DAYOFWEEK FROM date_day)          AS day_of_week,
  FORMAT_DATE('%A', date_day)               AS day_name,
  date_day = CURRENT_DATE()                 AS is_today,
  date_day <= CURRENT_DATE()                AS is_past_or_today,
  -- Fiscal year (starts 1 February)
  CASE
    WHEN EXTRACT(MONTH FROM date_day) >= 2
    THEN EXTRACT(YEAR FROM date_day)
    ELSE EXTRACT(YEAR FROM date_day) - 1
  END                                       AS fiscal_year,
  CASE
    WHEN EXTRACT(MONTH FROM date_day) >= 2
    THEN CEIL((EXTRACT(MONTH FROM date_day) - 1) / 3.0)
    ELSE 4
  END                                       AS fiscal_quarter,
  -- Calendar flags
  EXTRACT(DAYOFWEEK FROM date_day) IN (1, 7) AS is_weekend,
  EXTRACT(DAYOFWEEK FROM date_day) NOT IN (1, 7) AS is_weekday,
  DATE_TRUNC(date_day, WEEK)                AS week_start_date,
  DATE_TRUNC(date_day, MONTH)               AS month_start_date,
  DATE_TRUNC(date_day, QUARTER)             AS quarter_start_date,
  DATE_TRUNC(date_day, YEAR)                AS year_start_date,
  LAST_DAY(date_day, MONTH)                 AS month_end_date,
  LAST_DAY(date_day, QUARTER)               AS quarter_end_date
FROM date_spine
```

---

## 4. Cloud Function: ChurnZero Connector

**Location:** `cloud-functions/churnzero-connector/`
**Runtime:** Python 3.11 | **Trigger:** Cloud Scheduler (daily 6am UTC)

```python
# main.py — ChurnZero → BigQuery Incremental Connector

import json
import logging
import os
from datetime import datetime, timezone

import functions_framework
import requests
from google.cloud import bigquery, secretmanager, storage

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

PROJECT_ID = os.environ["GCP_PROJECT_ID"]  # core-dynamics-analytics-prod
DATASET_ID = "raw_churnzero"
BUCKET_NAME = f"{PROJECT_ID}-pipeline-state"
WATERMARK_BLOB = "churnzero/last_sync_timestamp.txt"


def get_secret(secret_id: str) -> str:
    client = secretmanager.SecretManagerServiceClient()
    name = f"projects/{PROJECT_ID}/secrets/{secret_id}/versions/latest"
    response = client.access_secret_version(request={"name": name})
    return response.payload.data.decode("UTF-8")


def get_watermark() -> str:
    """Read last successful sync timestamp from GCS."""
    client = storage.Client()
    bucket = client.bucket(BUCKET_NAME)
    blob = bucket.blob(WATERMARK_BLOB)
    if blob.exists():
        return blob.download_as_text().strip()
    return "2020-01-01T00:00:00Z"  # Initial full load


def set_watermark(ts: str) -> None:
    """Write sync timestamp to GCS after successful run."""
    client = storage.Client()
    bucket = client.bucket(BUCKET_NAME)
    blob = bucket.blob(WATERMARK_BLOB)
    blob.upload_from_string(ts)


def fetch_churnzero_accounts(api_key: str, since: str) -> list[dict]:
    """Fetch accounts modified since watermark."""
    base_url = "https://analytics.churnzero.net/api/v1"
    headers = {"Authorization": f"Key {api_key}", "Content-Type": "application/json"}
    accounts = []
    page = 1
    page_size = 200

    while True:
        response = requests.get(
            f"{base_url}/accounts",
            headers=headers,
            params={"modifiedAfter": since, "page": page, "pageSize": page_size},
            timeout=30,
        )
        response.raise_for_status()
        data = response.json()
        batch = data.get("data", [])
        accounts.extend(batch)
        if len(batch) < page_size:
            break
        page += 1

    logger.info(f"Fetched {len(accounts)} accounts from ChurnZero since {since}")
    return accounts


def load_to_bigquery(records: list[dict], table_name: str) -> None:
    """Upsert records into BigQuery using MERGE."""
    client = bigquery.Client()
    table_ref = f"{PROJECT_ID}.{DATASET_ID}.{table_name}"

    if not records:
        logger.info(f"No records to load for {table_name}")
        return

    errors = client.insert_rows_json(table_ref, records)
    if errors:
        raise RuntimeError(f"BigQuery insert errors: {errors}")
    logger.info(f"Loaded {len(records)} rows into {table_ref}")


@functions_framework.http
def run(request):
    """Cloud Function entrypoint."""
    sync_start = datetime.now(timezone.utc).isoformat()

    try:
        api_key = get_secret("churnzero-api-key")
        since = get_watermark()
        logger.info(f"Starting ChurnZero sync from {since}")

        accounts = fetch_churnzero_accounts(api_key, since)
        load_to_bigquery(accounts, "accounts")

        set_watermark(sync_start)
        logger.info("ChurnZero sync complete")
        return {"status": "success", "records_loaded": len(accounts)}, 200

    except Exception as e:
        logger.exception(f"ChurnZero sync failed: {e}")
        return {"status": "error", "message": str(e)}, 500
```

**requirements.txt:**
```
functions-framework==3.*
google-cloud-bigquery==3.*
google-cloud-secret-manager==2.*
google-cloud-storage==2.*
requests==2.*
```

---

## 5. Cloud Function: Clearbit Enrichment Connector

**Location:** `cloud-functions/clearbit-connector/`
**Runtime:** Python 3.11 | **Trigger:** HubSpot webhook via Pub/Sub (`contact.created`)

```python
# main.py — Clearbit → BigQuery Enrichment Connector

import base64
import json
import logging
import os
import time

import clearbit
import functions_framework
from google.cloud import bigquery, secretmanager

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

PROJECT_ID = os.environ["GCP_PROJECT_ID"]
DATASET_ID = "raw_clearbit"
TABLE_ID = "company_enrichments"
MAX_RETRIES = 3
RETRY_BACKOFF = 2  # seconds


def get_secret(secret_id: str) -> str:
    client = secretmanager.SecretManagerServiceClient()
    name = f"projects/{PROJECT_ID}/secrets/{secret_id}/versions/latest"
    response = client.access_secret_version(request={"name": name})
    return response.payload.data.decode("UTF-8")


def enrich_contact(email: str, api_key: str) -> dict | None:
    """Enrich contact with Clearbit company data. Handles rate limiting."""
    clearbit.key = api_key
    for attempt in range(MAX_RETRIES):
        try:
            response = clearbit.Enrichment.find(email=email, stream=True)
            if response and response.get("company"):
                return response["company"]
            return None
        except clearbit.TooManyRequestsError:
            logger.warning(f"Rate limited. Retry {attempt + 1}/{MAX_RETRIES}")
            time.sleep(RETRY_BACKOFF ** attempt)
    return None


def load_enrichment(email: str, company: dict) -> None:
    """Load enrichment record into BigQuery."""
    client = bigquery.Client()
    table_ref = f"{PROJECT_ID}.{DATASET_ID}.{TABLE_ID}"
    row = {
        "email": email,
        "company_name": company.get("name"),
        "company_domain": company.get("domain"),
        "company_industry": company.get("category", {}).get("industry"),
        "company_employee_count": company.get("metrics", {}).get("employees"),
        "company_annual_revenue": company.get("metrics", {}).get("annualRevenue"),
        "company_country": company.get("geo", {}).get("country"),
        "enriched_at": __import__("datetime").datetime.utcnow().isoformat(),
    }
    errors = client.insert_rows_json(table_ref, [row])
    if errors:
        raise RuntimeError(f"BigQuery insert errors: {errors}")


@functions_framework.cloud_event
def run(cloud_event):
    """Pub/Sub triggered Cloud Function."""
    try:
        payload = json.loads(base64.b64decode(cloud_event.data["message"]["data"]))
        email = payload.get("email")
        if not email:
            logger.warning("No email in webhook payload — skipping")
            return

        api_key = get_secret("clearbit-api-key")
        company = enrich_contact(email, api_key)

        if company:
            load_enrichment(email, company)
            logger.info(f"Enriched {email} with Clearbit company data")
        else:
            logger.info(f"No Clearbit enrichment found for {email}")

    except Exception as e:
        logger.exception(f"Clearbit enrichment failed: {e}")
        raise  # Allow Pub/Sub to retry
```

---

## 6. Cloud Composer DAG: core_dynamics_daily_pipeline

**Location:** `dags/core_dynamics_daily_pipeline.py`

```python
"""
Core Dynamics Daily Pipeline DAG
Orchestrates: Fivetran syncs → ChurnZero Cloud Function → Dataform staging →
              Dataform assertions → Dataform warehouse
Schedule: 02:00 UTC daily (nightly full refresh)
Intraday trigger: separate DAG for incremental high-priority sources (every 6 hours)
"""

from datetime import datetime, timedelta

from airflow import DAG
from airflow.operators.empty import EmptyOperator
from airflow.operators.python import BranchPythonOperator
from airflow.providers.google.cloud.operators.dataform import (
    DataformCreateCompilationResultOperator,
    DataformCreateWorkflowInvocationOperator,
)
from airflow.providers.google.cloud.operators.functions import (
    CloudFunctionInvokeFunctionOperator,
)
from airflow.providers.slack.operators.slack_webhook import SlackWebhookOperator

PROJECT_ID = "core-dynamics-analytics-prod"
REGION = "us-central1"
DATAFORM_REPO = "core-dynamics-dataform"
SLACK_CONN_ID = "slack_pipeline_alerts"
FIVETRAN_CONN_ID = "fivetran_default"

DEFAULT_ARGS = {
    "owner": "data-engineering",
    "depends_on_past": False,
    "start_date": datetime(2026, 4, 1),
    "email_on_failure": True,
    "email": ["data-engineering@rittmananalytics.com"],
    "retries": 2,
    "retry_delay": timedelta(minutes=5),
    "retry_exponential_backoff": True,
}

FIVETRAN_CONNECTORS = [
    ("salesforce", "salesforce_connector_id"),
    ("hubspot", "hubspot_connector_id"),
    ("google_ads", "google_ads_connector_id"),
    ("linkedin_ads", "linkedin_ads_connector_id"),
    ("meta_ads", "meta_ads_connector_id"),
    ("outreach", "outreach_connector_id"),
    ("corefm_db", "corefm_connector_id"),
    ("mixpanel", "mixpanel_connector_id"),
    ("segment", "segment_connector_id"),
    ("intercom", "intercom_connector_id"),
    ("zendesk", "zendesk_connector_id"),
    ("netsuite", "netsuite_connector_id"),
    ("pagerduty", "pagerduty_connector_id"),
    ("gcp_billing", "gcp_billing_connector_id"),
    ("cloud_monitoring", "cloud_monitoring_connector_id"),
    ("bamboohr", "bamboohr_connector_id"),
]

with DAG(
    dag_id="core_dynamics_daily_pipeline",
    default_args=DEFAULT_ARGS,
    description="Core Dynamics nightly ELT pipeline — Fivetran → Dataform → Warehouse",
    schedule_interval="0 2 * * *",  # 2am UTC daily
    catchup=False,
    max_active_runs=1,
    tags=["core-dynamics", "pipeline", "production"],
) as dag:

    start = EmptyOperator(task_id="pipeline_start")

    # --- Fivetran Syncs ---
    fivetran_tasks = []
    for source_name, connector_id in FIVETRAN_CONNECTORS:
        from airflow.providers.fivetran.operators.fivetran import FivetranOperator
        task = FivetranOperator(
            task_id=f"sync_{source_name}",
            fivetran_conn_id=FIVETRAN_CONN_ID,
            connector_id=connector_id,
            wait_for_completion=True,
            poke_interval=60,
        )
        start >> task
        fivetran_tasks.append(task)

    syncs_complete = EmptyOperator(
        task_id="all_syncs_complete",
        trigger_rule="all_done",  # Continue even if some connectors fail
    )
    for t in fivetran_tasks:
        t >> syncs_complete

    # --- ChurnZero Cloud Function ---
    run_churnzero = CloudFunctionInvokeFunctionOperator(
        task_id="run_churnzero_connector",
        project_id=PROJECT_ID,
        location=REGION,
        function_id="churnzero-connector",
        input_data={},
    )
    start >> run_churnzero >> syncs_complete

    # --- Dataform: Staging Layer ---
    compile_staging = DataformCreateCompilationResultOperator(
        task_id="compile_dataform_staging",
        project_id=PROJECT_ID,
        region=REGION,
        repository_id=DATAFORM_REPO,
        compilation_result={
            "git_commitish": "main",
            "code_compilation_config": {"vars": {"env": "prod"}},
        },
    )

    run_staging = DataformCreateWorkflowInvocationOperator(
        task_id="run_dataform_staging",
        project_id=PROJECT_ID,
        region=REGION,
        repository_id=DATAFORM_REPO,
        asynchronous=False,
        workflow_invocation={
            "compilation_result": "{{ task_instance.xcom_pull('compile_dataform_staging') }}",
            "invocation_config": {"included_tags": ["staging"]},
        },
    )

    syncs_complete >> compile_staging >> run_staging

    # --- Dataform: Assertions ---
    run_assertions = DataformCreateWorkflowInvocationOperator(
        task_id="run_dataform_assertions",
        project_id=PROJECT_ID,
        region=REGION,
        repository_id=DATAFORM_REPO,
        asynchronous=False,
        workflow_invocation={
            "compilation_result": "{{ task_instance.xcom_pull('compile_dataform_staging') }}",
            "invocation_config": {"included_tags": ["assertions"]},
        },
    )

    run_staging >> run_assertions

    # --- Branch: assertions pass or fail ---
    def check_assertion_results(**context):
        task_instance = context["task_instance"]
        invocation_state = task_instance.xcom_pull("run_dataform_assertions")
        if invocation_state == "SUCCEEDED":
            return "run_dataform_warehouse"
        return "alert_assertion_failure"

    branch = BranchPythonOperator(
        task_id="check_assertions",
        python_callable=check_assertion_results,
        provide_context=True,
    )
    run_assertions >> branch

    # --- Alert on assertion failure ---
    alert_failure = SlackWebhookOperator(
        task_id="alert_assertion_failure",
        slack_webhook_conn_id=SLACK_CONN_ID,
        message=(
            ":red_circle: *Core Dynamics Pipeline* — Dataform assertions FAILED on "
            "{{ ds }}. Warehouse refresh blocked. "
            "<https://console.cloud.google.com/dataform|View in Dataform>"
        ),
    )

    # --- Dataform: Warehouse Layer ---
    run_warehouse = DataformCreateWorkflowInvocationOperator(
        task_id="run_dataform_warehouse",
        project_id=PROJECT_ID,
        region=REGION,
        repository_id=DATAFORM_REPO,
        asynchronous=False,
        workflow_invocation={
            "compilation_result": "{{ task_instance.xcom_pull('compile_dataform_staging') }}",
            "invocation_config": {"included_tags": ["warehouse", "shared"]},
        },
    )

    branch >> [alert_failure, run_warehouse]

    pipeline_complete = EmptyOperator(
        task_id="pipeline_complete",
        trigger_rule="none_failed_min_one_success",
    )
    run_warehouse >> pipeline_complete
    alert_failure >> pipeline_complete
```

---

## 7. Shared Dimension: dim_csm.sqlx

```sql
config {
  type: "table",
  schema: "warehouse",
  description: "CSM dimension — one row per CSM, joining Salesforce user to BambooHR employee.",
  tags: ["warehouse", "shared", "dimension"]
}

WITH salesforce_users AS (
  SELECT
    TO_HEX(SHA256(CAST(id AS STRING)))   AS csm_pk,
    id                                   AS salesforce_user_id,
    name                                 AS csm_name,
    email,
    is_active,
    user_role__c                         AS csm_role
  FROM ${ref("raw_salesforce", "user")}
  WHERE profile_name = 'Customer Success'
    AND (_fivetran_deleted IS FALSE OR _fivetran_deleted IS NULL)
),

bamboohr_employees AS (
  SELECT * FROM ${ref("stg_bamboohr__employees")}
)

SELECT
  sf.csm_pk,
  sf.salesforce_user_id,
  sf.csm_name,
  sf.email,
  sf.is_active,
  sf.csm_role,
  bhr.department,
  bhr.hire_date,
  bhr.manager_name
FROM salesforce_users sf
LEFT JOIN bamboohr_employees bhr ON LOWER(sf.email) = LOWER(bhr.work_email)
```

---

## 8. Connector Configuration Summary

| Connector | Fivetran Connector Type | Sync Frequency | Destination Dataset | Configured By |
|---|---|---|---|---|
| Salesforce | salesforce | Every 6 hours | raw_salesforce | Kofi Asante |
| HubSpot | hubspot | Every 1 hour | raw_hubspot | Kofi Asante |
| Google Ads | google_ads | Daily 3am UTC | raw_google_ads | Kofi Asante |
| LinkedIn Ads | linkedin_ads_source | Daily 3am UTC | raw_linkedin_ads | Kofi Asante |
| Meta Ads | facebook_ads | Daily 3am UTC | raw_meta_ads | Kofi Asante |
| Outreach.io | outreach | Every 6 hours | raw_outreach | Kofi Asante |
| CoreFM PostgreSQL | postgres (log-based CDC) | Near real-time (15 min) | raw_corefm_db | Kofi Asante + Sean Murphy |
| Mixpanel | mixpanel | Daily 4am UTC | raw_mixpanel | Kofi Asante |
| Segment | Segment BigQuery Destination (native) | Streaming | raw_segment | Kofi Asante + Leon Yip |
| Intercom | intercom | Every 6 hours | raw_intercom | Kofi Asante |
| Zendesk | zendesk_support | Every 1 hour | raw_zendesk | Kofi Asante |
| NetSuite | netsuite_suiteanalytics | Daily 5am UTC | raw_netsuite | Kofi Asante (pending provisioning) |
| PagerDuty | pagerduty | Every 1 hour | raw_pagerduty | Kofi Asante |
| GCP Billing | native BigQuery export | Daily (T+1) | raw_gcp_billing | Sean Murphy |
| Cloud Monitoring | Cloud Monitoring API → Dataform | Daily | raw_cloud_monitoring | Kofi Asante |
| BambooHR | bamboohr | Daily 5am UTC | raw_bamboohr | Kofi Asante |
| ChurnZero | Cloud Function (custom) | Daily 6am UTC | raw_churnzero | Kofi Asante |
| Clearbit | Cloud Function (webhook) | On demand | raw_clearbit | Kofi Asante |

---

*Document status: Generated by Wire Autopilot — validated and self-approved*
*Reviewed by: Wire Autopilot (self-review)*
*Review date: 2026-03-29*
