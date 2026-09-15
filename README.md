# Oslo Bike Data Platform (dbt + GCP + Spark)

This repository contains an end-to-end analytics pipeline for Oslo Bysykkel (city bike) data.
It ingests real-time GBFS feeds and historical trip CSVs into BigQuery, then builds curated dimensional models and marts with dbt.

## What this project does

- Collects GBFS feeds (`station_status`, `station_information`) from Entur
- Publishes feed payloads to Pub/Sub and archives raw JSON in Cloud Storage
- Runs Spark jobs to load raw GBFS and historical trip data into BigQuery
- Transforms raw data into staging, dimensions, facts, and marts using dbt
- Produces analytics-ready tables for station availability, uptime, and trip metrics

## Repository structure

- `src/` – Python ingestion/orchestration jobs
  - `api_to_bucket.py` – Cloud Function/Run entrypoint for feed fetch + Pub/Sub + GCS archive
  - `pyspark_gbfs_raw_load.py` – Spark job loading GBFS raw JSON from GCS to BigQuery raw tables
  - `historical_bucket_to_bq.py` – Spark job loading historical CSV trip files to BigQuery
  - `realtime_pubsub_to_spark.py` – Spark Structured Streaming pipeline from Pub/Sub to BigQuery
  - `trigger_spark_job.py` – Cloud Function to submit Dataproc Spark jobs
- `models/` – Main dbt project models
  - `src/` – source declarations
  - `staging/` – flattened GBFS/trips staging models + schema tests
  - `dimensions/` – station/date/tariff dimensions
  - `facts/` – station status history/latest, uptime, and trips facts
  - `marts/` – business-facing marts (availability, uptime, trip metrics)
- `models_demo/` – sample dbt tutorial models (jaffle shop)
- `data/` – monthly historic trip CSV files (`01_2025.csv` ... `05_2025.csv`)
- `libs/` – Spark connector JARs
- `dbt_project.yml` – dbt project and model materialization configuration

## Data model overview

### Sources

- `gbfs.raw_station_status`
- `gbfs.raw_station_information`
- `trips.raw_historic_trips`

### Staging

- `stg_station_status`
- `stg_station_information`
- `stg_station_tariffs`
- `stg_historic_trips`

### Dimensions

- `dim_date`
- `dim_stations`
- `dim_tariff`

### Facts

- `fact_station_status`
- `fact_station_status_history`
- `fact_station_status_latest`
- `fact_station_uptime`
- `fact_trips`

### Marts

- `mart_station_availability`
- `mart_station_uptime`
- `mart_trip_metrics`

## Tech stack

- **dbt** (BigQuery adapter)
- **Google BigQuery**
- **Google Cloud Storage**
- **Google Pub/Sub**
- **Google Dataproc (PySpark)**
- **Python** (`requests`, `google-cloud-*`, `functions-framework`)

## Prerequisites

- Python 3.10+
- dbt Core + dbt-bigquery
- Access to a GCP project with BigQuery, GCS, Pub/Sub, and Dataproc enabled
- A configured `~/.dbt/profiles.yml` profile matching `profile: default`
- `DBT_TARGET_DATABASE` environment variable set for source resolution

## Local dbt workflow

From repository root:

```bash
dbt deps
dbt run
dbt test
dbt docs generate
```

## Notes on execution

- `dbt_project.yml` routes models into layered schemas:
  - `data_management_2_arun` (raw/source area)
  - `stg` (staging)
  - `analytics` (dimensions, facts, marts)
- Some ingestion scripts include environment-variable defaults for project/bucket/table names; override these in each deployment environment.

## Dependencies

Python runtime dependencies are listed in `src/requirements.txt`.
