# Big Data Pipeline Architecture

## Problem Statement
Build an end-to-end big data pipeline processing 10 TB/day with batch ETL, data quality, and BI dashboards.

## Architecture Diagram

```mermaid
graph LR
  subgraph Sources["Data Sources"]
    RDS["Production RDS\n(CDC via DMS)"]
    S3Raw["S3 Raw Files\n(Partner drops)"]
    API["External APIs\n(Lambda ingestion)"]
  end
  subgraph Lake["Data Lake (S3)"]
    Raw["Raw Zone"]
    Processed["Processed Zone\n(Parquet)"]
    Curated["Curated Zone\n(Aggregated)"]
  end
  subgraph Processing["Processing"]
    DMS["DMS CDC"] --> Raw
    S3Raw --> Raw
    API --> Raw
    GlueETL["Glue ETL\n(PySpark)"] --> Processed
    GlueDQ["Glue Data Quality"] --> Processed
    GlueAgg["Glue ETL\n(Aggregation)"] --> Curated
  end
  subgraph Serving["Serving Layer"]
    Redshift["Redshift\n(COPY from S3)"]
    Athena["Athena\n(ad-hoc)"]
    QB["QuickSight\n(BI Dashboards)"]
  end
  Processed --> GlueDQ
  GlueDQ -->|Pass| GlueAgg
  GlueDQ -->|Fail| Quarantine["Quarantine S3\n+ Alert"]
  Curated --> Redshift --> QB
  Processed --> Athena
```

## Pipeline Orchestration
- **AWS Step Functions**: orchestrate Glue jobs sequentially
- **EventBridge Scheduler**: trigger daily at 02:00 UTC
- **Failure handling**: SNS alert + DLQ for failed Glue jobs
- **Monitoring**: Glue job metrics → CloudWatch → Dashboard

## Data Quality Rules (Glue Data Quality)
```
Rules = [
  IsComplete "customer_id",
  IsUnique "order_id",
  ColumnValues "amount" between 0 and 1000000,
  DataFreshness "created_at" with threshold 2 days
]
```

## Interview Talking Points
- "CDC from DMS means you're getting real-time changes without impacting production DB"
- "Data quality checks before writing to curated layer prevents bad data in BI"
- "Step Functions for pipeline orchestration: visibility, retry logic, failure handling"
- "Parquet + partitioning reduces Redshift COPY time and Athena query costs"
