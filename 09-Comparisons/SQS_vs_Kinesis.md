# SQS vs Kinesis Data Streams

## Quick Decision Guide

**Use SQS when:**
- Task queue (each message processed by ONE consumer)
- Exactly-once processing (FIFO queue)
- Simple decoupling between services
- No ordering requirement (Standard) or strict order per group (FIFO)
- No replay needed

**Use Kinesis Data Streams when:**
- Multiple independent consumers need the same data (fan-out)
- Replay of historical events needed (configurable retention)
- Ordered events per partition key
- Real-time analytics alongside processing
- Events are a stream (time-series, IoT, clickstream)

## Detailed Comparison

| Feature | SQS Standard | SQS FIFO | Kinesis Data Streams |
|---------|-------------|---------|---------------------|
| Ordering | No | Strict per group | Per shard |
| Delivery | At-least-once | Exactly-once | At-least-once |
| Multiple consumers | No (1 consumer per message) | No | Yes (fan-out) |
| Replay | No | No | Yes (24h-365 days) |
| Throughput | Nearly unlimited | 300 TPS (3000 with batching) | 1 MB/s per shard |
| Consumer model | Pull (long-polling) | Pull | Pull (polling or EFO push) |
| Max message size | 256 KB | 256 KB | 1 MB |
| Visibility timeout | 12 hrs max | 12 hrs max | No visibility concept |
| DLQ | Yes | Yes | Yes (with Lambda ESM) |
| Retention | 14 days max | 14 days max | 24h-365 days |
| Cost | $0.40/M messages | $0.50/M messages | $0.015/shard/hr |
| Consumer parallelism | Many consumers compete | Many consumers compete | 1 consumer per shard (standard) |
| Fan-out | SNS+SQS fan-out | SNS+SQS fan-out | Native (EFO) |

## Architecture Patterns

**SQS for task distribution:**
```
Producer → SQS → Worker 1 (claims message, processes, deletes)
                → Worker 2 (waiting)
                → Worker 3 (waiting)
Only ONE worker processes each message.
```

**Kinesis for fan-out:**
```
Producer → Kinesis → Consumer 1 (analytics) [reads independently]
                  → Consumer 2 (alerts)    [reads independently]
                  → Consumer 3 (archival)  [reads independently]
ALL consumers read ALL messages.
```

## Interview Talking Points
- "SQS: work queue where tasks are claimed; Kinesis: event stream where all consumers see all events"
- "If you need fan-out with replay: Kinesis or EventBridge with archiving"
- "SQS FIFO exactly-once via deduplication ID; Kinesis is at-least-once (implement idempotency)"
- "Kinesis retention + replay is often the deciding factor — can you replay events for a new consumer?"
