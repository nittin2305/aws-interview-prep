# Performance Efficiency Best Practices

## Compute
- Choose the right instance family: compute-optimised (C), memory-optimised (R/X), storage-optimised (I/D)
- Use Graviton3 (C7g, M7g) for best price-performance on general workloads
- Profile before optimising — use AWS Compute Optimizer and X-Ray
- Enable Enhanced Networking (ENA) for high-throughput, low-latency networking

## Caching Strategy
- **CloudFront:** Static assets, API responses (TTL-based or cache-control headers)
- **ElastiCache Redis:** Session state, leaderboards, real-time counters, DB read cache
- **ElastiCache Memcached:** Simple, horizontally scalable object cache
- **DAX:** In-memory cache for DynamoDB — microsecond reads

### Cache Invalidation
- Use versioned asset URLs (e.g., `app.v2.js`) to avoid stale cache issues
- Set appropriate TTLs — short for dynamic data, long for static assets
- Use cache-aside pattern for DB queries

## Database Performance
- Use read replicas to offload read traffic from primary
- Enable query caching at application layer (not MySQL query cache — deprecated)
- Choose appropriate DynamoDB partition key to distribute load evenly
- Use RDS Proxy to pool connections and reduce overhead

## Async / Decoupling
- Offload long-running tasks to SQS + worker fleet
- Use S3 presigned URLs to shift large uploads off application servers
- Use SNS fan-out to parallelise downstream processing

## Content Delivery
- CloudFront with S3 Origin for static sites (Origin Access Control)
- Lambda@Edge / CloudFront Functions for lightweight request transformation
- Global Accelerator for non-HTTP workloads needing low latency

## Observability for Performance
- Use X-Ray to trace latency hotspots across services
- CloudWatch Container Insights / Lambda Insights for detailed metrics
- Use Contributor Insights on DynamoDB to identify hot keys

## Interview Questions
1. **API response times degrade under load.** — Profile with X-Ray, check database connection limits (add RDS Proxy), check for N+1 query patterns, add ElastiCache layer, scale out application tier.
2. **DynamoDB latency spikes at certain times.** — Identify hot partitions via CloudWatch metrics / Contributor Insights; redesign partition key to distribute load; consider DAX for read-heavy workloads.
3. **How do you optimise a Lambda function's cold start?** — Use Provisioned Concurrency for latency-sensitive paths; trim package size; prefer runtimes with fast init (Python/Node); keep handler lean (move init code outside handler).
