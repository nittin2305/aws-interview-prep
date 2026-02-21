# Aurora vs RDS

## Quick Decision Guide

**Use RDS (standard) when:**
- Cost-sensitive; Aurora ~20% more expensive
- Running SQL Server or Oracle (not supported on Aurora)
- Simple workload; MySQL/PostgreSQL compatibility sufficient
- Familiar team prefers standard PostgreSQL engine behavior

**Use Aurora when:**
- Need faster failover (< 30s vs 60-120s)
- 15 read replicas vs 5 for RDS
- Auto-storage growth needed
- Aurora Global Database (< 1s RPO cross-region)
- Aurora Serverless v2 (variable workload)
- Want better performance at same cost

## Detailed Comparison

| Feature | RDS MySQL/PG | Aurora MySQL/PG |
|---------|-------------|----------------|
| Storage | EBS per instance | Shared distributed (6 copies, 3 AZs) |
| Storage limit | 64 TB | 128 TB |
| Storage auto-grow | Manual resize | Auto (10 GB increments) |
| IOPS | EBS-limited | Higher (distributed) |
| Multi-AZ failover | 60-120s | < 30s (typically < 15s) |
| Read replicas | Up to 5 (async) | Up to 15 (same storage, near-zero lag) |
| Replication lag | Yes (async) | Near zero (shared storage) |
| Global DB | No | Yes (< 1s RPO, < 1 min RTO) |
| Serverless | No | Yes (v2 recommended) |
| Backtrack | No | Yes (MySQL only, up to 72h) |
| DB Cloning | No | Yes (near-instant, copy-on-write) |
| Performance | Baseline | ~3-5x faster for writes (fewer log writes) |
| Engine compatibility | MySQL 5.7/8.0, PG 12-16 | MySQL 5.7/8.0, PG 12-16 (mostly compatible) |
| SQL Server | Yes | No |
| Oracle | Yes | No |
| Price premium | Baseline | ~20% more than equivalent RDS |
| Certification support | Yes | Limited (some enterprise apps) |

## When Aurora Isn't the Answer

1. **Oracle/SQL Server**: must use RDS (Aurora only MySQL/PostgreSQL-compatible)
2. **ISV certified applications**: some require specific PostgreSQL/MySQL version; Aurora has minor behavioral differences
3. **Extreme cost sensitivity**: RDS + manual HA is cheaper (but more ops work)
4. **Low-traffic, simple DB**: Aurora overhead not justified for dev/test

## Performance Deep Dive

Aurora write advantage:
- RDS: writes to EBS → log to standby → ack
- Aurora: writes to 4/6 distributed storage nodes → ack (no log shipping)
- Aurora writes are faster under high concurrency

Aurora read advantage (replicas):
- RDS replicas: lag from async log shipping
- Aurora replicas: shared storage → zero lag; reads always see writer's data

## Interview Talking Points
- "Aurora is MySQL/PG compatible but optimized storage layer is the key difference"
- "15 replicas with zero lag vs 5 replicas with lag — massive difference for read-heavy workloads"
- "Aurora Global Database vs RDS cross-region read replica: Aurora wins on RPO and RTO"
- "20% cost premium is worth it if you need the HA, performance, or Global DB features"
