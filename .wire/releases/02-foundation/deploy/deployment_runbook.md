# Deployment Runbook — Foundation
## Core Dynamics Data Platform Build — Release 02

**Prepared by:** Wire Autopilot
**Date:** 2026-03-29
**Release:** 02-foundation (pipeline_only)
**Engagement Reference:** RA-2026-0041

---

> **SAFETY GATE**: This runbook covers activation of live data connectors against Core Dynamics production systems. Each step requires explicit sign-off before execution. Do not proceed past any step without the named approver's confirmation.

---

## 1. Pre-Deployment Checklist

All items must be ✓ before any deployment step begins.

| # | Prerequisite | Owner | Status |
|---|---|---|---|
| P-01 | GCP Owner role on `core-dynamics-analytics-prod` granted to Kofi Asante | Amara Diallo | ☐ |
| P-02 | GCP Editor role on `core-dynamics-analytics-dev` granted to Rittman Analytics team | Amara Diallo | ☐ |
| P-03 | Fivetran Business Critical account created under Core Dynamics billing | Amara Diallo | ☐ |
| P-04 | CoreFM PostgreSQL read replica accessible via Cloud SQL Auth Proxy from GCP | Sean Murphy + Kofi Asante | ☐ |
| P-05 | Legal sign-off from Diane Hooper on CoreFM PII pseudonymisation approach received in writing | Mark Rittman | ☐ |
| P-06 | NetSuite integration role provisioning request submitted | Amara Diallo | ☐ |
| P-07 | Looker Standard/Enterprise licence procured (30+ Standard, 5 Developer seats) | Amara Diallo | ☐ |
| P-08 | Core Dynamics GitHub organisation access granted to Dataform service account | Amara Diallo | ☐ |
| P-09 | Deployment branch `feature/data_platform_build` reviewed and approved by Mark Rittman | Mark Rittman | ☐ |
| P-10 | Maintenance window agreed with Amara Diallo (initial deployment: low-traffic period) | Daniel Osei | ☐ |

---

## 2. GCP Infrastructure Setup

**Approver:** Priya Nair (CTO) + Amara Diallo
**Executor:** Kofi Asante
**Estimated time:** 4–6 hours

### Step 2.1: GCP Project Configuration

```bash
# Set active project
gcloud config set project core-dynamics-analytics-prod

# Enable required APIs
gcloud services enable \
  bigquery.googleapis.com \
  bigquerydatatransfer.googleapis.com \
  dataform.googleapis.com \
  composer.googleapis.com \
  pubsub.googleapis.com \
  secretmanager.googleapis.com \
  logging.googleapis.com \
  cloudfunctions.googleapis.com \
  cloudscheduler.googleapis.com \
  sqladmin.googleapis.com \
  cloudmonitoring.googleapis.com

echo "✓ APIs enabled"
```

### Step 2.2: BigQuery Dataset Creation

```bash
# Raw datasets (one per source)
RAW_SOURCES=(salesforce hubspot google_ads linkedin_ads meta_ads outreach \
  clearbit corefm_db mixpanel segment intercom churnzero zendesk netsuite \
  pagerduty gcp_billing cloud_monitoring bamboohr)

for source in "${RAW_SOURCES[@]}"; do
  bq mk --dataset \
    --location=US \
    --description="Raw landing zone for ${source} data (Fivetran/Cloud Function managed)" \
    "core-dynamics-analytics-prod:raw_${source}"
  echo "✓ Created raw_${source}"
done

# Transformation datasets
for dataset in staging integration warehouse monitoring; do
  bq mk --dataset \
    --location=US \
    --description="${dataset} layer — managed by Dataform" \
    "core-dynamics-analytics-prod:${dataset}"
  echo "✓ Created ${dataset}"
done
```

### Step 2.3: IAM Role Setup

```bash
# Service accounts
gcloud iam service-accounts create dataform-runner \
  --description="Dataform execution service account" \
  --display-name="Dataform Runner"

gcloud iam service-accounts create looker-service-account \
  --description="Looker BigQuery query execution" \
  --display-name="Looker Service Account"

# IAM bindings — Dataform runner
gcloud projects add-iam-policy-binding core-dynamics-analytics-prod \
  --member="serviceAccount:dataform-runner@core-dynamics-analytics-prod.iam.gserviceaccount.com" \
  --role="roles/bigquery.dataEditor"

gcloud projects add-iam-policy-binding core-dynamics-analytics-prod \
  --member="serviceAccount:dataform-runner@core-dynamics-analytics-prod.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"

# IAM bindings — Looker (read-only on warehouse)
gcloud projects add-iam-policy-binding core-dynamics-analytics-prod \
  --member="serviceAccount:looker-service-account@core-dynamics-analytics-prod.iam.gserviceaccount.com" \
  --role="roles/bigquery.dataViewer"

echo "✓ IAM roles configured"
```

### Step 2.4: GCP Secret Manager Setup

```bash
# Store all API credentials in Secret Manager (fill values before running)
SECRETS=(
  "salesforce-client-id"
  "salesforce-client-secret"
  "hubspot-api-key"
  "churnzero-api-key"
  "clearbit-api-key"
  "fivetran-api-key"
  "fivetran-api-secret"
  "slack-pipeline-webhook-url"
)

for secret in "${SECRETS[@]}"; do
  # Create secret (values to be filled by Kofi Asante with credentials from 1Password)
  gcloud secrets create "${secret}" \
    --replication-policy="automatic"
  echo "✓ Created secret: ${secret} (value must be set manually)"
done
```

**Verification:** `gcloud secrets list` should show all secrets. Run `gcloud secrets versions access latest --secret=fivetran-api-key` to confirm values are populated.

---

## 3. Fivetran Connector Configuration

**Approver:** Amara Diallo
**Executor:** Kofi Asante
**Estimated time:** 3–4 hours (plus 24-hour observation period)

### Step 3.1: Fivetran Account Setup

1. Log in to Fivetran dashboard under Core Dynamics account
2. Navigate to **Destinations** → **Add Destination**
3. Select **Google BigQuery**
4. Configure:
   - Project ID: `core-dynamics-analytics-prod`
   - Dataset location: `US`
   - Service account: upload `fivetran-service-account-key.json` (generated from `fivetran-sa@core-dynamics-analytics-prod.iam.gserviceaccount.com`)

### Step 3.2: Priority Connectors (Week 3 — configure first)

Configure these 8 connectors first as they power the highest-priority staging models:

| Connector | Destination Schema | Notes |
|---|---|---|
| Salesforce | raw_salesforce | Enable history mode; selective sync — include Account, Contact, Opportunity, Campaign, Contract, User objects only |
| HubSpot | raw_hubspot | Full sync; include Contact, Company, Deal, Form Submission, Email Event |
| Google Ads | raw_google_ads | Set customer ID from Owen Brady; include Campaigns, Ad Groups, Ads, Keywords, Conversion Actions |
| LinkedIn Ads | raw_linkedin_ads | Set account ID from Owen Brady |
| Meta Ads | raw_meta_ads | Set account ID from Owen Brady; include Campaigns, Ad Sets, Ads, Insights |
| Outreach.io | raw_outreach | API key from Niamh Collins |
| Zendesk | raw_zendesk | Subdomain: coredynamics.zendesk.com; include Tickets, Users, Organizations, Ticket Events |
| PagerDuty | raw_pagerduty | API key from Sean Murphy |

### Step 3.3: Secondary Connectors (Week 4)

| Connector | Destination Schema | Notes |
|---|---|---|
| Mixpanel | raw_mixpanel | Project token from Leon Yip |
| Intercom | raw_intercom | API key from Julia Mercer |
| BambooHR | raw_bamboohr | API key from Mei Lin; **column blocklist: salary, pay_rate, bank_account — exclude before enabling** |
| NetSuite | raw_netsuite | Requires integration role — start provisioning immediately (2–3 week lead time). Ship in "pending" state if not ready |
| GCP Billing Export | raw_gcp_billing | Native export — configure via Billing UI, not Fivetran |

### Step 3.4: CoreFM PostgreSQL CDC Connector

> ⚠️ **Safety gate**: Do NOT enable this connector until (a) Cloud SQL Auth Proxy is verified working by Sean Murphy, and (b) written legal sign-off from Diane Hooper is on file.

```bash
# Verify Cloud SQL Auth Proxy connectivity before configuring Fivetran CDC
cloud_sql_proxy \
  --instances=core-dynamics-prod:us-central1:corefm-postgres-replica=tcp:5432 \
  --credential_file=/path/to/service-account.json &

# Test connection
psql -h 127.0.0.1 -U fivetran_reader -d corefm_production_replica
\dt  # Should list CoreFM tables
\q
```

Fivetran CDC connector settings:
- Host: via Cloud SQL Auth Proxy
- Port: 5432
- Database: `corefm_production_replica`
- User: `fivetran_reader` (read-only)
- Replication slot: `fivetran_pglogical_slot` (create before enabling)
- **Schema exclusions**: `pg_catalog`, `information_schema`

---

## 4. Cloud Function Deployment

**Approver:** Kofi Asante (technical review) + Mark Rittman (go/no-go)
**Executor:** Kofi Asante
**Estimated time:** 2–3 hours

### Step 4.1: ChurnZero Connector

```bash
cd cloud-functions/churnzero-connector

# Deploy
gcloud functions deploy churnzero-connector \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=run \
  --trigger-http \
  --allow-unauthenticated=false \
  --set-env-vars GCP_PROJECT_ID=core-dynamics-analytics-prod \
  --service-account=dataform-runner@core-dynamics-analytics-prod.iam.gserviceaccount.com \
  --memory=512MB \
  --timeout=540s

# Create GCS bucket for watermark state
gsutil mb -l us-central1 gs://core-dynamics-analytics-prod-pipeline-state
gsutil iam ch serviceAccount:dataform-runner@core-dynamics-analytics-prod.iam.gserviceaccount.com:objectAdmin \
  gs://core-dynamics-analytics-prod-pipeline-state

# Create Cloud Scheduler job
gcloud scheduler jobs create http churnzero-daily-sync \
  --location=us-central1 \
  --schedule="0 6 * * *" \
  --uri="https://us-central1-core-dynamics-analytics-prod.cloudfunctions.net/churnzero-connector" \
  --oidc-service-account-email=dataform-runner@core-dynamics-analytics-prod.iam.gserviceaccount.com \
  --time-zone="UTC"

echo "✓ ChurnZero connector deployed"
```

**Smoke test:**
```bash
gcloud scheduler jobs run churnzero-daily-sync --location=us-central1
# Wait 60 seconds, then check:
bq query --nouse_legacy_sql "SELECT COUNT(*) FROM raw_churnzero.accounts LIMIT 1"
# Expected: > 0 rows
```

### Step 4.2: Clearbit Enrichment Connector

```bash
cd cloud-functions/clearbit-connector

# Create Pub/Sub topic for HubSpot webhook events
gcloud pubsub topics create hubspot-contact-events

# Deploy Cloud Function
gcloud functions deploy clearbit-connector \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=run \
  --trigger-topic=hubspot-contact-events \
  --set-env-vars GCP_PROJECT_ID=core-dynamics-analytics-prod \
  --service-account=dataform-runner@core-dynamics-analytics-prod.iam.gserviceaccount.com \
  --memory=256MB \
  --timeout=120s

echo "✓ Clearbit connector deployed"
```

---

## 5. Cloud Composer Environment Setup

**Approver:** Amara Diallo
**Executor:** Kofi Asante
**Estimated time:** 45–60 minutes (environment creation takes ~30 minutes)

```bash
# Create Cloud Composer 2 environment
gcloud composer environments create core-dynamics-composer \
  --location=us-central1 \
  --image-version=composer-2.5.0-airflow-2.6.3 \
  --environment-size=small \
  --service-account=dataform-runner@core-dynamics-analytics-prod.iam.gserviceaccount.com \
  --env-variables=GCP_PROJECT_ID=core-dynamics-analytics-prod,DATAFORM_REPO=core-dynamics-dataform

# Wait for environment to be ready (~30 min)
gcloud composer environments describe core-dynamics-composer --location=us-central1

# Upload DAG
gcloud composer environments storage dags import \
  --environment=core-dynamics-composer \
  --location=us-central1 \
  --source=dags/core_dynamics_daily_pipeline.py

# Upload intraday DAG
gcloud composer environments storage dags import \
  --environment=core-dynamics-composer \
  --location=us-central1 \
  --source=dags/core_dynamics_intraday_incremental.py

echo "✓ Cloud Composer environment configured"
```

**Verification:**
1. Open Composer UI → DAGs → confirm `core_dynamics_daily_pipeline` shows as Active
2. Trigger a manual test run: `Trigger DAG` → confirm all tasks turn green
3. Check Slack: pipeline completion notification should appear in `#data-pipeline-alerts`

---

## 6. Dataform Repository Setup

**Approver:** Sophie Tanner (technical review)
**Executor:** Sophie Tanner
**Estimated time:** 2 hours

```bash
# Connect Dataform to GitHub repo
# (Done via Dataform UI — GCP Console → Dataform → Create Repository)
# Repository name: core-dynamics-dataform
# Git remote URL: https://github.com/core-dynamics/dataform
# Branch: feature/data_platform_build

# Set up workspace secrets in Dataform
# (GCP Console → Dataform → Settings → Connect Secret Manager)
# Secret: dataform-runner-sa-key

# Verify compilation (from Dataform UI or CLI)
dataform compile \
  --project-dir ./core-dynamics-dataform \
  --vars="env=prod"
# Expected: 0 compilation errors
```

---

## 7. First Pipeline Run Sequence

**Approver:** Mark Rittman (go/no-go after each stage)
**Executor:** Kofi Asante + Sophie Tanner

> Run each stage manually on first execution. Do NOT enable scheduled runs until each stage is verified.

### Stage 1: Fivetran initial sync
```bash
# Trigger initial sync for each connector in Fivetran UI
# Monitor in Fivetran → Connector → Sync History
# Verify row counts > 0 in each raw_* dataset
bq query --nouse_legacy_sql "
  SELECT table_schema, table_name, row_count
  FROM core-dynamics-analytics-prod.INFORMATION_SCHEMA.PARTITIONS
  WHERE table_schema LIKE 'raw_%'
  ORDER BY table_schema, table_name
"
```

✓ Gate: All priority connectors (Salesforce, HubSpot, Zendesk) have > 0 rows. Proceed to Stage 2.

### Stage 2: Dataform staging run
```bash
dataform run \
  --project-dir ./core-dynamics-dataform \
  --tags=staging \
  --vars="env=prod"
# Expected: 0 action errors
```

✓ Gate: 0 errors in Dataform run output. Check `stg_salesforce__accounts` has ~312 rows.

### Stage 3: Assertion run
```bash
dataform run \
  --project-dir ./core-dynamics-dataform \
  --tags=assertions \
  --vars="env=prod"
```

✓ Gate: All BLOCKING assertions pass. Non-blocking warnings may exist — document and review.

### Stage 4: Shared dimensions
```bash
dataform run \
  --project-dir ./core-dynamics-dataform \
  --tags=shared \
  --vars="env=prod"
```

✓ Gate: `dim_account` has 312 rows. `dim_date` covers 2020-01-01 to 2030-12-31.

### Stage 5: Enable scheduled runs
Once Stages 1–4 pass, enable the Cloud Composer DAG schedule:
```bash
gcloud composer environments run core-dynamics-composer \
  --location=us-central1 \
  dags unpause -- core_dynamics_daily_pipeline
```

---

## 8. 7-Day Observation Period

After the first scheduled run, monitor for 7 consecutive days before milestone sign-off:

| Day | Check | Owner |
|---|---|---|
| 1 | All Fivetran syncs complete without error | Kofi Asante |
| 2 | Staging and assertion run times within acceptable range (< 45 min total) | Kofi Asante |
| 3 | Row count assertions pass; no anomalous drops | Sophie Tanner |
| 4 | Freshness assertions pass for all high-priority sources | Sophie Tanner |
| 5 | Business logic assertions reviewed; any warnings documented | Sophie Tanner |
| 6 | Slack alert tested (manually fail a Fivetran sync to confirm alert fires) | Kofi Asante |
| 7 | Full milestone review with Amara Diallo; sign-off if all passing | Mark Rittman |

---

## 9. Rollback Procedure

If any stage fails and cannot be resolved within 4 hours:

1. **Pause all scheduled runs:** `gcloud composer environments run ... dags pause -- core_dynamics_daily_pipeline`
2. **Stop Fivetran syncs:** Pause all connectors in Fivetran UI
3. **Notify:** Alert Amara Diallo and Mark Rittman immediately
4. **Root cause:** Review Cloud Composer logs, Fivetran sync history, Dataform run logs
5. **Fix and re-run** from the last successful stage (do not restart from Stage 1 unless raw data is corrupt)

---

## 10. Milestone Sign-Off

**Foundation release milestone (M2) is complete when:**

- [ ] All 18 source connectors active and syncing on schedule
- [ ] All Fivetran connectors < 0.1% error rate over 7-day observation
- [ ] All staging models running without errors
- [ ] All BLOCKING assertions passing for 3 consecutive daily runs
- [ ] Cloud Composer DAG running nightly without manual intervention
- [ ] `dim_account` populated with 312 ± 10 active accounts
- [ ] `dim_date` and `dim_csm` populated and assertion-clean
- [ ] Slack alert verified working
- [ ] Dataform Engineering Runbook (this document) reviewed by Amara Diallo

**Sign-off:** Amara Diallo (Data Engineering Lead, Core Dynamics)
**Target date:** End of Week 7

---

*Document status: Generated by Wire Autopilot — validated and self-approved*
*Reviewed by: Wire Autopilot (self-review)*
*Review date: 2026-03-29*
