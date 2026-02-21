# RDS vs DynamoDB

## Quick Decision Guide

**Use RDS (Aurora) when:**
- Complex queries with JOINs and aggregations
- ACID transactions across multiple tables
- Schema is well-defined and relatively stable
- Ad-hoc query patterns not yet known
- Team expertise in SQL

**Use DynamoDB when:**
- Known, simple access patterns (key-value, range queries)
- Single-digit millisecond latency at any scale
- Massive scale (millions of req/s)
- Global active-active needed
- Serverless + auto-scale required

## Detailed Comparison

| Feature | RDS/Aurora | DynamoDB |
|---------|-----------|---------|
| Data model | Relational (tables, rows) | Key-value / document |
| Query language | SQL (complex JOINs, aggregations) | API (GetItem, Query, Scan) |
| Transactions | Full ACID | TransactWriteItems (100 items max) |
| Consistency | Strong by default | Eventual (or strongly consistent for extra cost) |
| Latency | 1-10ms (same AZ) | Single-digit ms |
| Scale | Vertical + read replicas | Horizontal, automatic |
| Max IOPS | Provisioned or Aurora auto | Millions (on-demand) |
| Max data | 128 TB (Aurora) | Unlimited |
| Global | Aurora Global DB (<1s RPO) | Global Tables (active-active) |
| Schema | Fixed | Flexible (schemaless) |
| JOINs | Yes | No (application-side joins) |
| Indexes | Many (any column) | GSI (20 max), LSI (5 max) |
| Cost model | Per instance-hour | Per request or capacity unit |
| Ops overhead | Medium | Low (fully managed) |

## When the Answer is Nuanced

**"Use DynamoDB" trap**: DynamoDB is NOT always faster
- Sequential scans: RDS wins (better optimizer)
- Complex aggregations: RDS wins (GROUP BY, window functions)
- Ad-hoc reporting: Athena on DynamoDB export vs RDS read replica

**"Use RDS" trap**: RDS is NOT always ACID-safe
- Multi-table transactions in microservices: distributed transaction problem
- Solution: Saga pattern or use DynamoDB TransactWriteItems for same-table atomicity

## Interview Talking Points
- "DynamoDB requires knowing access patterns upfront; RDS is more flexible for ad-hoc"
- "DynamoDB global tables = active-active; Aurora Global = active-passive with < 1s RPO"
- "For a new product where requirements evolve: RDS is safer; switch to DynamoDB when patterns stabilize"
- "Scaling: DynamoDB scales infinitely; RDS hits limits and requires different strategies"
