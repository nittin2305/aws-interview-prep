# Lambda vs ECS Fargate

## Quick Decision Guide

**Use Lambda when:**
- Event-driven, short-duration tasks (< 15 minutes)
- Highly variable traffic (zero to millions instantly)
- Pay-per-invocation preferred
- No warm instance needed

**Use ECS Fargate when:**
- Long-running processes (> 15 minutes)
- Persistent connections (WebSocket, database)
- Need > 10 GB memory or > 6 vCPU
- Warm instance with immediate response needed

## Detailed Comparison

| Feature | Lambda | ECS Fargate |
|---------|--------|------------|
| Max runtime | 15 minutes | Unlimited |
| Max memory | 10 GB | 120 GB |
| Max vCPU | 6 vCPU | 16 vCPU |
| Cold start | 100ms - seconds | ~30s (task startup) |
| Scaling speed | Immediate | ~30 seconds |
| Pricing | Per invocation + duration | Per vCPU/memory-hour |
| Idle cost | $0 | Task uptime cost |
| State | Stateless (ephemeral /tmp 10 GB) | Stateless or EFS |
| VPC | Optional (adds cold start) | Always VPC |
| Max concurrency | 1,000 (default); request increase | ASG-based |
| Persistent connections | No | Yes |
| DaemonSet equivalent | No | EC2 launch type only |
| OS access | No | Limited (containers) |

## Cost Comparison Example

**Workload: 1M requests/day, 500ms avg duration, 512 MB**

Lambda:
- 1M invocations × $0.0000002 = $0.20
- Duration: 1M × 0.5s × 0.5 GB × $0.0000166667 = $4.17
- Total: **~$4.37/month**

Fargate (1 task running 24/7):
- 0.25 vCPU × 720 hrs × $0.04048 = $7.29
- 0.5 GB × 720 hrs × $0.004445 = $1.60
- Total: **~$8.89/month**

Lambda wins at bursty; Fargate wins at sustained high volume.

## Interview Talking Points
- "Lambda for event-driven, short tasks; Fargate for long-running services"
- "At sustained load Lambda gets expensive; model the math for your specific workload"
- "Cold starts in Lambda with JVM = SnapStart or Provisioned Concurrency; or switch to Fargate"
- "Fargate's 30s startup is often better than Lambda's 3s+ Java cold start"
