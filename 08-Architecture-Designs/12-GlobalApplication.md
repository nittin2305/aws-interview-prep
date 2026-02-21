# Global Application Architecture

## Problem Statement
Design a globally distributed application serving users in North America, Europe, and Asia-Pacific with < 50ms latency globally.

## Architecture Diagram

```mermaid
graph TB
  CF["CloudFront\n(200+ PoPs globally)"]
  subgraph USEast["us-east-1"]
    API1["API Layer"] --> Aurora1["Aurora Global\n(Primary Write)"]
    API1 --> Cache1["ElastiCache"]
  end
  subgraph EUWest["eu-west-1"]
    API2["API Layer"] --> Aurora2["Aurora Global\n(Read Replica)"]
    API2 --> Cache2["ElastiCache"]
  end
  subgraph APSouth["ap-southeast-1"]
    API3["API Layer"] --> Aurora3["Aurora Global\n(Read Replica)"]
    API3 --> Cache3["ElastiCache"]
  end
  CF --> API1 & API2 & API3
  Aurora1 -->|< 1s storage replication| Aurora2 & Aurora3
```

## Latency Optimization
- **CloudFront**: static assets < 5ms from edge PoP
- **Dynamic API**: latency-based Route 53 routing → nearest region
- **Aurora Global DB reads**: EU users read from EU replica (< 1s behind primary)
- **Write routing**: all writes → us-east-1 primary (or use write forwarding)

## Consistency Tradeoffs
- Aurora Global replica: < 1s behind primary
- Users who write then immediately read in different region may see stale data
- Mitigation: session pinning (route user to same region for duration of session); or write forwarding

## Interview Talking Points
- "CloudFront + latency routing + Aurora Global = the global stack"
- "Aurora Global replicas have < 1s lag; acceptable for most reads"
- "Write forwarding adds latency (cross-region write); most apps have way more reads than writes"
- "Consider data residency: EU data must stay in EU for GDPR — regional isolation needed"
