# AWS Cost Optimization Best Practices

## Core Pillars

### 1. Right-Sizing
- Use AWS Compute Optimizer to identify over-provisioned resources
- Match instance types to actual workload requirements
- Review and adjust reserved instances regularly
- Use Graviton (ARM) instances where possible — up to 40% cheaper

### 2. Pricing Models
| Model | Discount vs On-Demand | Best For |
|---|---|---|
| On-Demand | 0% | Unpredictable, short-term |
| Savings Plans | up to 66% | Steady-state compute |
| Reserved Instances | up to 72% | Predictable workloads 1–3 yr |
| Spot Instances | up to 90% | Fault-tolerant, batch |

### 3. Storage Optimization
- Enable S3 Intelligent-Tiering for unknown access patterns
- Use S3 Lifecycle policies to transition to Glacier/Deep Archive
- Delete unattached EBS volumes and old snapshots
- Compress data before storing

### 4. Networking Costs
- Minimize cross-AZ data transfer — place resources in same AZ when latency allows
- Use VPC endpoints to avoid NAT Gateway charges for S3/DynamoDB
- Use CloudFront to cache content at edge and reduce origin traffic
- Consider Direct Connect for large-volume data transfer

### 5. Database Cost Reduction
- Use Aurora Serverless v2 for variable workloads
- Enable RDS stop/start for dev/test environments
- Use DynamoDB on-demand for unpredictable traffic; switch to provisioned + auto-scaling for predictable
- Leverage ElastiCache to reduce database reads

### 6. Lambda / Serverless
- Profile memory allocation — often over-provisioned
- Use ARM/Graviton2 (20% cheaper, up to 34% better price-performance)
- Minimise package size to reduce cold-start and billing duration

## Governance & Visibility
- Apply resource tagging policy (environment, team, project, cost-centre)
- Use AWS Cost Explorer, Budgets, and Cost Anomaly Detection
- Set budget alerts for 80% and 100% thresholds
- Review Trusted Advisor recommendations monthly

## Interview Questions
1. **How do you reduce EC2 costs for a steady-state web tier?** — Reserved Instances / Compute Savings Plans + Auto Scaling to scale in at night.
2. **S3 costs are high — what do you investigate?** — Storage class distribution, lifecycle policies, request metrics, replication costs, versioning retaining old objects.
3. **NAT Gateway bill is unexpectedly high.** — Check which resources are routing through it; add VPC endpoints for AWS services; move cross-AZ traffic to same-AZ.
