# SNS vs EventBridge

## Quick Decision Guide

**Use SNS when:**
- Simple pub/sub fan-out
- Push notifications (SMS, email, mobile push)
- Multiple subscribers to same topic
- No content-based routing needed

**Use EventBridge when:**
- Content-based routing (filter by event fields)
- Multiple event sources (AWS services, SaaS, custom)
- Need event archive + replay
- Schema registry for event documentation
- Complex event patterns (matching)

## Detailed Comparison

| Feature | SNS | EventBridge |
|---------|-----|------------|
| Routing | Topic-based | Content-based (JSON field matching) |
| Sources | Your apps + AWS services | 100+ AWS services + SaaS + custom |
| Targets | SQS, Lambda, HTTP, SMS, Email, mobile | Lambda, SQS, SNS, Step Functions, API GW, many more |
| Event filtering | Message attributes only | Full JSON field matching (nested, array) |
| Archive + Replay | No | Yes (rule-based archiving) |
| Schema Registry | No | Yes |
| Dead letter queue | Yes (per subscription) | Yes (per rule) |
| Event ordering | No | No |
| Throughput | Very high | High (but has TPS limits per rule) |
| Cost | $0.50/M notifications | $1/M events |
| FIFO | No | No |
| Cross-account | Yes (topic policy) | Yes (event buses) |
| Cross-region | Yes (replication) | Yes (cross-region rules) |

## Content-Based Routing Example

SNS message filter (limited):
```json
{"eventType": ["OrderCreated"]}
```

EventBridge pattern (powerful):
```json
{
  "source": ["myapp.orders"],
  "detail-type": ["OrderCreated"],
  "detail": {
    "amount": [{"numeric": [">", 1000]}],
    "status": ["PENDING"],
    "customerTier": ["premium"]
  }
}
```

## When to Use Both
SNS + SQS fan-out pattern (classic):
- SNS topic → multiple SQS queues
- Still useful for: HTTP/Email/SMS endpoints; SQS fan-out

EventBridge + SNS:
- EventBridge for routing + archiving
- SNS as EventBridge target for push notification fan-out

## Interview Talking Points
- "EventBridge is SNS evolved: richer routing, archive/replay, schema registry"
- "SNS still needed for SMS, email, mobile push — EventBridge can't do those directly"
- "EventBridge Archive + Replay is the killer feature: retroactively add new consumers to historical events"
- "At high volume, SNS is cheaper; at moderate volume with complex routing, EventBridge wins"
