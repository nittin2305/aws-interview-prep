# AWS Master Cheat Sheet

## Compute

| Service | Key Fact |
|---|---|
| EC2 | Virtual machine; choose instance family (C=compute, R=RAM, I=I/O, G=GPU) |
| Auto Scaling | Scale EC2 by schedule, metric, or predictive policy |
| Lambda | Max 15 min, 10 GB RAM, 512 MB–10 GB /tmp, 1000 default concurrency |
| ECS (Fargate) | Serverless containers; task = pod; service = deployment |
| EKS | Managed Kubernetes; node groups or Fargate profiles |
| Elastic Beanstalk | PaaS; uploads code → manages EC2, ALB, ASG |
| Batch | Managed batch computing on EC2/Fargate |

## Storage

| Service | Key Fact |
|---|---|
| S3 | 11 9s durability; 99.99% availability (Standard); unlimited objects; max object 5 TB |
| S3 Glacier Instant | Millisecond retrieval; 90-day min storage |
| S3 Glacier Flexible | Minutes–hours retrieval; 90-day min |
| S3 Deep Archive | 12–48 h retrieval; cheapest; 180-day min |
| EBS gp3 | Default SSD; 3000 IOPS baseline; up to 16 000 IOPS |
| EBS io2 | 99.999% durability; up to 64 000 IOPS; multi-attach |
| EFS | NFS; shared across AZs; auto-scaling capacity |
| FSx for Windows | SMB/NTFS; AD integration |
| FSx for Lustre | High-performance parallel FS; HPC / ML |
| Storage Gateway | Hybrid; File/Volume/Tape gateway |

## Networking

| Service | Key Fact |
|---|---|
| VPC | Logically isolated network; max /16 CIDR |
| Subnet | /16 to /28; 5 IPs reserved by AWS per subnet |
| Security Group | Stateful; allow rules only; instance-level |
| NACL | Stateless; allow + deny; subnet-level; rules evaluated in order |
| NAT Gateway | Outbound internet for private subnets; per-AZ; ~$0.045/hr + data |
| Internet Gateway | Bidirectional internet access for public subnets |
| VPC Peering | 1-to-1; non-transitive; no overlapping CIDRs |
| Transit Gateway | Hub-and-spoke; transitive routing; cross-account/region |
| PrivateLink | Private connectivity to services without internet |
| ALB | HTTP/HTTPS; path/host routing; WAF integration; Layer 7 |
| NLB | TCP/UDP/TLS; ultra-low latency; static IP; Layer 4 |
| GWLB | Layer 3; inline traffic inspection via appliances |
| Route 53 | DNS; health checks; routing policies (simple/weighted/latency/failover/geolocation/multivalue) |
| CloudFront | CDN; 450+ PoPs; Origin Shield; Lambda@Edge |
| Direct Connect | Dedicated 1/10/100 Gbps link; lower latency + cost for large transfer |
| Site-to-Site VPN | IPSec over internet; up to 1.25 Gbps per tunnel |
| Global Accelerator | Anycast IPs; routes traffic via AWS global network |

## Databases

| Service | Key Fact |
|---|---|
| RDS | Managed relational (MySQL/PostgreSQL/Oracle/SQL Server/MariaDB) |
| Aurora MySQL | 5× faster than MySQL; 6-way replication across 3 AZs |
| Aurora PostgreSQL | 3× faster than PostgreSQL; backtrack feature |
| Aurora Serverless v2 | Fine-grained auto-scaling in 0.5 ACU increments |
| Aurora Global DB | < 1 s cross-region replication; 16 read replicas per region |
| DynamoDB | Serverless NoSQL; single-digit ms; 99.999% multi-region HA |
| DynamoDB DAX | In-memory cache; microsecond reads; API-compatible |
| DynamoDB Streams | Change data capture; triggers Lambda |
| DynamoDB Global Tables | Active-active multi-region replication |
| ElastiCache Redis | Rich data types; persistence; Cluster Mode; pub/sub |
| ElastiCache Memcached | Simple; multi-threaded; no persistence |
| Neptune | Managed graph DB (Gremlin / SPARQL) |
| QLDB | Immutable ledger; cryptographically verifiable |
| DocumentDB | MongoDB-compatible managed document DB |
| Keyspaces | Apache Cassandra-compatible |
| Timestream | Time-series DB; auto-tiering hot→warm→cold |

## Security

| Service | Key Fact |
|---|---|
| IAM | Users, Groups, Roles, Policies; no region; global |
| STS | Temporary credentials; AssumeRole / AssumeRoleWithWebIdentity |
| KMS | Managed symmetric/asymmetric keys; 256-bit AES; automatic rotation |
| CloudHSM | Dedicated HSM; FIPS 140-2 Level 3; customer manages keys |
| Secrets Manager | Automatic rotation for RDS, Redshift, DocumentDB, custom |
| Parameter Store | Hierarchy of config values; SecureString uses KMS |
| Cognito User Pool | Authentication (login, MFA, OAuth2) |
| Cognito Identity Pool | Federated identities → AWS credentials |
| WAF | Layer 7 firewall; IP/geo/rate/managed rules |
| Shield Standard | Free DDoS protection (L3/L4) |
| Shield Advanced | $3000/mo; L7 + DDoS cost protection |
| GuardDuty | Threat detection via ML on CloudTrail, VPC Flow Logs, DNS |
| Macie | PII discovery in S3 |
| Inspector | Vulnerability scanning for EC2 and containers |
| Security Hub | Aggregate findings; CIS/PCI/NIST benchmarks |
| Config | Configuration recorder + rules; compliance evaluation |
| CloudTrail | API audit log; 90-day free; S3 for long-term |
| Detective | Investigate security findings; graph analysis |

## Data & Analytics

| Service | Key Fact |
|---|---|
| Kinesis Data Streams | Real-time; up to 7-day retention; shard = 1 MB/s in, 2 MB/s out |
| Kinesis Firehose | Near-real-time (60 s min); automatic scaling; no custom consumers |
| Kinesis Data Analytics | SQL / Apache Flink on streaming data |
| MSK (Kafka) | Managed Kafka; up to 30 brokers |
| S3 + Athena | Serverless SQL on S3; pay per query (per TB scanned) |
| Glue | Serverless ETL; Data Catalog; Crawlers |
| EMR | Managed Hadoop/Spark/Hive/Presto |
| Redshift | Columnar data warehouse; RA3 nodes; Redshift Spectrum for S3 |
| QuickSight | Serverless BI; ML Insights; SPICE in-memory engine |
| Lake Formation | Data lake governance; fine-grained column/row permissions |
| Data Exchange | Subscribe to third-party datasets |

## Application Integration

| Service | Key Fact |
|---|---|
| SQS Standard | At-least-once; best-effort order; unlimited throughput |
| SQS FIFO | Exactly-once; strict order; 3000 msg/s with batching |
| SNS | Push pub/sub; fan-out; 12.5M subscriptions per topic |
| EventBridge | Event bus; schema registry; rules → targets; SaaS integration |
| Step Functions | State machine orchestration; Standard (1 yr) or Express (5 min) |
| AppSync | Managed GraphQL; real-time via WebSocket |
| API Gateway | REST / HTTP / WebSocket APIs; throttling; usage plans |
| MQ | Managed ActiveMQ / RabbitMQ; lift-and-shift messaging |

## DevOps / Management

| Service | Key Fact |
|---|---|
| CloudFormation | IaC; stacks + stack sets; drift detection |
| CDK | Code-first IaC; synthesises to CloudFormation |
| CodeCommit | Managed Git (being deprecated — use GitHub/GitLab) |
| CodeBuild | Managed build service; Docker-based |
| CodeDeploy | Deployment automation; EC2/Lambda/ECS; blue/green |
| CodePipeline | CI/CD orchestration; integrates with GitHub, ECR, etc. |
| Systems Manager | Patch Manager, Session Manager, Parameter Store, Automation |
| OpsWorks | Managed Chef/Puppet |
| Service Catalog | Approved product portfolios for self-service provisioning |
| Control Tower | Landing zone; guardrails; account vending machine |
| Organizations | Multi-account; SCPs; consolidated billing |
| Trusted Advisor | Cost, security, performance, fault tolerance checks |
| Well-Architected Tool | Review workload against 6 pillars |

## Cost Management

| Service | Key Fact |
|---|---|
| Cost Explorer | Visualise and forecast spend; RI/SP recommendations |
| Budgets | Alert at % of budget; supports cost/usage/RI/SP |
| Cost Anomaly Detection | ML-based; alerts on unexpected spend |
| Compute Savings Plans | 1 or 3 yr; covers EC2, Lambda, Fargate |
| EC2 Instance Savings Plans | Specific instance family + region; highest discount |
| Reserved Instances | 1 or 3 yr; Standard (highest discount) or Convertible |
| Spot Instances | Up to 90% discount; interruption with 2-min warning |

## Exam / Interview Quick-Fire

| Question | Answer |
|---|---|
| S3 max object size | 5 TB (multipart upload required > 5 GB) |
| Lambda max timeout | 15 minutes |
| RDS automated backup retention | 0–35 days |
| DynamoDB max item size | 400 KB |
| Kinesis shard write throughput | 1 MB/s or 1000 records/s |
| SQS max message size | 256 KB |
| SQS visibility timeout max | 12 hours |
| SQS retention max | 14 days |
| CloudFront max TTL | 365 days |
| VPC max CIDR size | /16 |
| VPC min CIDR size | /28 |
| Max VPCs per region (default) | 5 |
| EBS gp3 max IOPS | 16,000 |
| EBS io2 max IOPS | 64,000 |
| ALB layers | Layer 7 (HTTP/HTTPS) |
| NLB layers | Layer 4 (TCP/UDP/TLS) |
| Route 53 TTL minimum | 0 seconds |
| Aurora storage increment | 10 GB auto-grow |
| IAM policy max size | 6,144 characters (inline), 6,144 (managed) |
| EC2 On-Demand max vCPUs (default) | 32 |
