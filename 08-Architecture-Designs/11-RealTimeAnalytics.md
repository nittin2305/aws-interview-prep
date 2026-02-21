# Real-Time Analytics Architecture

## Problem Statement
Build a real-time analytics platform for a gaming company: leaderboards, session analytics, live event tracking for 10 million concurrent players.

## Architecture Diagram

```mermaid
graph LR
  GameClient["Game Clients"] --> APIGW["API Gateway\nWebSocket"]
  APIGW --> Lambda["Lambda\n(event processor)"]
  Lambda --> KDS["Kinesis Data Streams"]
  Lambda --> DDB["DynamoDB\n(game state)"]
  KDS --> Flink["Kinesis Analytics Flink\n(real-time aggregations)"]
  Flink --> Redis["ElastiCache Redis\n(leaderboard Sorted Set)"]
  Flink --> CW["CloudWatch\n(live dashboards)"]
  DDB --> Streams["DynamoDB Streams"] --> Analytics["Lambda → Redshift\n(historical analytics)"]
  Redis --> LeaderboardAPI["Leaderboard API\n(sub-ms reads)"]
```

## Leaderboard with Redis Sorted Sets
```
ZADD leaderboard:global <score> <player_id>   # O(log N)
ZREVRANK leaderboard:global <player_id>        # player's global rank
ZREVRANGE leaderboard:global 0 99 WITHSCORES  # top 100
ZINCRBY leaderboard:global <delta> <player_id> # score update
```

## Scaling at 10M Concurrent Players
- API Gateway WebSocket: horizontal auto-scale
- Lambda: 1,000+ concurrent; request limit increase
- DynamoDB: On-Demand; partition key = player_id (high cardinality)
- Redis: Cluster Mode Enabled; 50 shards × 16,384 slots
- Flink: auto-scaling application (Kinesis Analytics)

## Interview Talking Points
- "Redis Sorted Sets are purpose-built for leaderboards: O(log N) all operations"
- "DynamoDB On-Demand for gaming: spikes at event launch are unpredictable"
- "WebSocket API Gateway for push-based updates without polling"
