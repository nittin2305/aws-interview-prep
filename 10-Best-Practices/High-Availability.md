# High Availability & Reliability Best Practices

## Foundational Concepts
- **RTO (Recovery Time Objective):** Maximum acceptable downtime
- **RPO (Recovery Point Objective):** Maximum acceptable data loss
- **Availability %:** 99.9% = 8.7 h/yr downtime; 99.99% = 52 min/yr; 99.999% = 5 min/yr

## Multi-AZ Design
- Deploy all stateful resources (RDS, ElastiCache) with Multi-AZ enabled
- Spread EC2 Auto Scaling Groups across ≥ 3 AZs
- Use ALB (which is inherently multi-AZ) instead of single EC2 endpoints
- Place NAT Gateways in each AZ to avoid single points of failure

## Multi-Region Design
- Use Route 53 health checks + failover / latency routing
- Replicate data: S3 Cross-Region Replication, Aurora Global Database, DynamoDB Global Tables
- Keep secondary region warm (active-active or active-passive)
- Use AWS Global Accelerator for deterministic global routing

## Auto Scaling Best Practices
- Use predictive scaling for recurring traffic patterns
- Set scale-out aggressive, scale-in conservative (cool-down periods)
- Health check grace period should exceed application start time
- Test scaling policies under realistic load

## Health Checks & Circuit Breakers
- Configure ALB/NLB health checks with appropriate intervals and thresholds
- Use ECS health checks at both task and load-balancer level
- Implement circuit-breaker pattern in service mesh (App Mesh / Istio)

## Chaos Engineering
- Use AWS Fault Injection Simulator (FIS) to inject failures
- Regularly run Game Days to test runbooks
- Test AZ failure, API throttling, and instance termination scenarios

## Backup & Restore
- Use AWS Backup for centralised policy-driven backup
- Test restores regularly — untested backups are not backups
- Store backups in a separate AWS account to protect against ransomware

## Interview Questions
1. **Walk me through designing a 99.99% available web application on AWS.** — Multi-AZ ALB → Auto Scaling Group (3 AZs) → RDS Multi-AZ / Aurora → Route 53 health checks → CloudFront.
2. **How do you detect and auto-remediate an unhealthy EC2 instance?** — EC2 Auto Recovery for system failures; ASG health checks terminate and replace failed instances; CloudWatch + Systems Manager for deep remediation.
3. **What's your multi-region failover strategy?** — Active-passive: Route 53 failover records + health checks. Data: Aurora Global Database (< 1 s RPO). Runbook: automated Lambda to promote secondary on alarm.
