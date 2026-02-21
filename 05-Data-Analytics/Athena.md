# Athena — Senior Interview Guide

## Core Concepts

- **Serverless SQL** on S3; no infrastructure to manage
- Based on Presto/Trino (distributed SQL engine)
- Pay per query: $5 per TB of data scanned
- Queries S3 directly; schema defined in AWS Glue Data Catalog
- Supports: CSV, JSON, ORC, Parquet, Avro, textfiles

---

## File Format Performance

| Format | Compression | Columnar | Query Speed | Best For |
|--------|------------|---------|-------------|---------|
| CSV | No | No | Slowest (scan all) | Raw logs, ingestion |
| JSON | Optional | No | Slow | Semi-structured |
| Parquet | Yes (Snappy) | Yes | Fast | Analytics, BI |
| ORC | Yes (Zlib) | Yes | Fast | Hive ecosystem |
| Avro | Yes | No (row) | Medium | Schema evolution |

**Recommendation**: convert to Parquet or ORC for production; 3-5x compression + columnar access = fewer bytes scanned = lower cost + faster queries

---

## Partitioning

- Partition data in S3 by frequently filtered columns (date, region, tenant)
- Athena skips unneeded partitions = less data scanned = lower cost + faster
- **Hive-style partitioning**: `s3://bucket/logs/year=2024/month=01/day=15/`
- Add partitions: `MSCK REPAIR TABLE` (slow, scans all S3 prefixes) or `ALTER TABLE ADD PARTITION` (manual, fast)

### Partition Projection
- Define partition patterns in table properties (no Glue catalog partition metadata needed)
- Eliminates partition metadata reads for large partition counts (>1000 partitions)
- Example: date-range projection — auto-generate partition list without metadata lookups
- Up to 10x faster for highly partitioned tables

---

## Federated Queries

- Query data outside S3: RDS, Redshift, DynamoDB, ElastiCache, HBase, on-premises
- Lambda data source connector per source type
- Results materialized in S3 (spill location)
- Use case: JOIN S3 data with live RDS/DynamoDB data without ETL

---

## Athena for Apache Spark

- Run Spark workloads without managing clusters
- Serverless Spark; auto-provisions capacity
- Jupyter-compatible notebooks
- Use case: complex ML feature engineering, advanced analytics

---

## Cost Optimization

- **Columnar formats**: Parquet/ORC reduces scan by 80-95%
- **Partitioning**: date partitioning reduces daily scans from full table to one day
- **Compression**: reduces bytes scanned; Snappy good balance of speed/compression
- **Workgroups**: set data scan limits per workgroup/query; prevent runaway queries
- **Result caching**: Athena caches query results for same query + same data (reuse within 24h)

---

## Most Asked Senior Interview Questions

**Q1: How do you minimize Athena query costs?**
1. Convert to Parquet/ORC (columnar scanning reduces cost 80-95%)
2. Partition by date and common filter columns
3. Compress data (Snappy for Parquet)
4. Use Partition Projection for large partition counts
5. Push down predicates (WHERE clause on partition columns)
6. Set workgroup scan limit: `bytesScannedCutoffPerQuery`

**Q2: MSCK REPAIR TABLE vs ALTER TABLE ADD PARTITION — when to use each?**
- MSCK REPAIR TABLE: automatically discovers all S3 partitions; slow for thousands of partitions (scans every S3 prefix)
- ALTER TABLE ADD PARTITION: manually specify partition; fast; predictable
- **Best practice**: use Glue Crawler or Glue ETL to add partitions after each load; or Partition Projection (no metadata needed at all)

**Q3: Athena vs Redshift — when does Athena win?**
- Ad-hoc queries on raw S3 data with no loading
- Pay-per-query model (queries few times/day)
- Data already in S3 (logs, CloudTrail, VPC flow logs)
- No ongoing infrastructure cost when not querying
- Athena loses to Redshift for: high-concurrency BI, complex JOINs on large tables, predictable continuous workloads

**Q4: How does Athena handle schema evolution?**
- CSV/JSON: add columns at end; Athena tolerates extra columns
- Parquet/ORC: Parquet supports schema evolution (add/rename columns); Athena reads by name not position
- Use Hudi or Iceberg tables (Athena native support): ACID transactions, schema evolution, time travel

**Q5: Federated query — what are the limitations?**
- Lambda connector has 15-min timeout; large queries may time out
- Spill to S3: queries that exceed Lambda memory spill intermediate data to S3 ($5/TB scanned applies to spill)
- Cannot always push predicates to source (depends on connector)
- Performance: much slower than native Athena on S3 (network + Lambda overhead)
- Use for: occasional JOIN with live data; not for high-frequency queries

**Q6: How do you query VPC Flow Logs efficiently with Athena?**
1. VPC Flow Logs → S3 (with date partitioning)
2. Create Athena table with correct schema for flow log format
3. Enable Partition Projection for date-based partitions
4. Query: filter by specific IPs, ports, or REJECT actions
5. Cost: Parquet format not available for raw flow logs (they're plain text); use Firehose to convert to Parquet first

---

## Scenario-Based Questions

**Scenario 1: 10 TB/day of application logs, ad-hoc queried by security team**
- Solution:
  1. Firehose → S3 (Parquet format + date partitioning)
  2. Glue Data Catalog schema
  3. Athena workgroup for security team with 1 TB scan limit per query
  4. Partition Projection on date column
  5. Result caching enabled
  6. Cost: ~$0.50-2/query (200-400 GB scanned with good partitioning)

**Scenario 2: Analyze CloudTrail logs across entire organization**
- CloudTrail Organization trail → S3 (organization/account/region/date structure)
- Athena table over S3 with partition on account_id + date
- Partition Projection for date range
- Queries: find all `DeleteBucket` calls, failed IAM auth events, root usage
- Cost: CloudTrail logs compress well with Parquet; minimal scan cost

---

## Real-world Failure Cases

**1. Query scanning entire dataset despite WHERE clause**
- Cause: partition column not used in WHERE; or MSCK REPAIR TABLE not run
- Fix: verify partition pruning in EXPLAIN; ensure WHERE uses exact partition column name; run MSCK REPAIR or add partitions

**2. Athena query timeout on very large JOIN**
- Cause: joining two large unpartitioned tables; 30-min Athena query timeout
- Fix: pre-aggregate or pre-filter before JOIN; convert to Parquet + partition; use Redshift for complex multi-table JOINs

**3. Athena quota limits hit**
- 20 concurrent queries per account limit
- Fix: use multiple workgroups; request quota increase; queue queries; use Athena for non-time-critical queries

**4. Federated query Lambda timeout**
- Large RDS table federated query; Lambda times out
- Fix: add WHERE clause to reduce result set; or ETL data to S3 periodically instead of federated query

---

## Cost Optimization Table

| Optimization | Estimated Savings |
|-------------|------------------|
| CSV → Parquet | 80-95% |
| Add date partitioning | 90% (if daily queries) |
| Snappy compression | 40-60% |
| Partition Projection | 5-20% (metadata reads) |
| Result caching | 100% for repeated identical queries |
| Workgroup scan limits | Prevents expensive mistakes |

---

## Security Considerations

- **Workgroup IAM**: control who can create/run queries in which workgroup
- **S3 bucket policy**: Athena uses IAM role; restrict which buckets/prefixes accessible
- **Encryption**: query results encrypted in S3; data at rest encrypted with KMS
- **Lake Formation**: fine-grained column/row-level access control on Glue catalog tables
- **Audit**: CloudTrail records all Athena API calls; CloudWatch for query metrics

---

## Quick Revision Bullets

- Athena = serverless SQL on S3; $5/TB scanned; pay nothing when not querying
- Parquet/ORC = columnar = 80-95% cost reduction vs CSV/JSON
- Partitioning = filter-based S3 pruning; Partition Projection = no metadata lookups
- Federated queries via Lambda connectors (RDS, DynamoDB, etc.)
- MSCK REPAIR TABLE = slow partition discovery; ALTER TABLE ADD PARTITION = manual, fast
- Workgroup scan limits prevent runaway costs
- Iceberg/Hudi tables: ACID + schema evolution + time travel on Athena
- Result caching: free reuse of same query results within 24h
