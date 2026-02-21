# High Availability Multi-AZ Architecture

## Problem Statement
Design a system achieving 99.99% availability (< 1hr downtime/year) within a single AWS region.

## Architecture Diagram

```mermaid
graph TB
  subgraph Region["us-east-1"]
    R53["Route 53\n(Failover routing)"]
    subgraph AZ1["AZ-1"]
      ALB1["ALB Node"] --> App1["App Servers\nASG"]
      App1 --> Cache1["ElastiCache\n(Primary)"]
      App1 --> DB1["Aurora\nPrimary"]
    end
    subgraph AZ2["AZ-2"]
      ALB2["ALB Node"] --> App2["App Servers\nASG"]
      App2 --> Cache2["ElastiCache\n(Replica)"]
      App2 --> DB2["Aurora\nReplica"]
    end
    subgraph AZ3["AZ-3"]
      ALB3["ALB Node"] --> App3["App Servers\nASG"]
      App3 --> Cache3["ElastiCache\n(Replica)"]
      App3 --> DB3["Aurora\nReplica"]
    end
  end
  R53 --> ALB1 & ALB2 & ALB3
```

## High Availability Components

| Component | HA Mechanism | RTO |
|-----------|------------|-----|
| ALB | Multi-AZ native | 0 (auto) |
| EC2/ECS | ASG across 3 AZs | < 5 min |
| ElastiCache | Multi-AZ, auto-failover | < 1 min |
| Aurora | 3-AZ replica | < 30s |
| Route 53 | Health checks + failover | < 60s |
| NAT Gateway | One per AZ | AZ-isolated |

## Design Principles for 99.99%
- Minimum 3 AZs for all stateful services
- No single instance for any tier
- Health checks everywhere (ALB, Route 53, ASG)
- Auto-healing: ASG replaces unhealthy instances automatically
- DB: Aurora Multi-AZ with read replicas (not just Multi-AZ)
- Dependency: map all external dependencies; health check each one

## Interview Talking Points
- "99.99% = 52 min downtime/year; 99.999% = 5 min — know your math"
- "Single AZ failure is the most common scenario; design for it explicitly"
- "NAT Gateway is AZ-specific — one per AZ, not shared"
- "ALB is Multi-AZ automatically but you still need instances in multiple AZs"
