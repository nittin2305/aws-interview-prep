# Microservices on ECS Architecture

## Problem Statement
Decompose a monolith into microservices on ECS with independent deployability, service discovery, and centralized observability.

## Architecture Diagram

```mermaid
graph TB
  ALB["ALB"] --> SC["ECS Service Connect\n(Service Mesh)"]
  subgraph Services["ECS Services (Fargate)"]
    US["User Service"]
    OS["Order Service"]
    PS["Payment Service"]
    NS["Notification Service"]
  end
  SC --> US & OS & PS
  OS -->|async event| SQS["SQS"]
  SQS --> NS
  US --> RDS1["RDS PostgreSQL\n(User DB)"]
  OS --> DDB["DynamoDB\n(Order DB)"]
  PS --> RDS2["RDS PostgreSQL\n(Payment DB)"]
  All --> CW["CloudWatch\nContainer Insights + X-Ray"]
```

## Key Design Decisions
- **Database per service**: each service owns its data store
- **ECS Service Connect**: built-in service discovery + metrics; replaces Cloud Map DNS
- **Async communication**: SQS between services for non-blocking operations
- **ALB path routing**: `/api/users/*` → User Service; `/api/orders/*` → Order Service

## Scaling Strategy
- Per-service auto scaling (Application Auto Scaling)
- Target tracking: CPU or SQS queue depth per service
- Services scale independently; no coordinated scaling needed

## Security
- Task role per service: only accesses its own DB
- Service-to-service: mTLS via Service Connect or App Mesh
- Secrets Manager for DB credentials per service

## Interview Talking Points
- "Service per database is the core of microservices data isolation"
- "Synchronous calls via Service Connect; async via SQS for resilience"
- "ECS Service Connect replaces simple Cloud Map for built-in circuit breaking metrics"
