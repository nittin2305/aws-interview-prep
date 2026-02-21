# Microservices on EKS Architecture

## Problem Statement
Deploy Kubernetes-based microservices on EKS with full observability, GitOps deployment, and service mesh.

## Architecture Diagram

```mermaid
graph TB
  Traffic["Traffic"] --> ALB["AWS LB Controller\n(ALB Ingress)"]
  ALB --> Istio["Istio Ingress Gateway"]
  subgraph K8s["EKS Cluster"]
    subgraph NS1["Namespace: team-a"]
      SvcA["Service A"]
      SvcB["Service B"]
    end
    subgraph NS2["Namespace: team-b"]
      SvcC["Service C"]
    end
    Istio --> SvcA & SvcC
    SvcA <-->|mTLS via Envoy| SvcB
    SvcB <-->|mTLS via Envoy| SvcC
  end
  SvcA & SvcB & SvcC --> IRSA["IRSA\n(per-service IAM)"]
  IRSA --> AWS["AWS Services\n(S3, DynamoDB, SQS)"]
  Argo["ArgoCD\n(GitOps)"] --> K8s
```

## Key Design Decisions
- **IRSA**: per-service IAM roles via service accounts; no shared instance profiles
- **Istio service mesh**: mTLS between services, traffic management, circuit breaking
- **ArgoCD**: GitOps; desired state in Git; auto-reconcile to cluster
- **Namespace isolation**: team-based namespaces with network policies

## Scaling Strategy
- Karpenter: dynamic node provisioning with Spot + On-Demand mix
- HPA: Horizontal Pod Autoscaler on CPU/custom metrics
- KEDA: event-driven scaling (SQS queue depth, etc.)
- VPA: right-size pod resource requests

## Security
- OPA Gatekeeper: admission policies (no privileged, require resource limits)
- Network Policies (Cilium): deny-all default; explicit allow
- PSA: restricted mode for all production namespaces
- IRSA: no instance profile sharing

## Interview Talking Points
- "EKS for K8s ecosystem portability; ECS for AWS-native simplicity"
- "IRSA is the right way to give pods AWS access — never shared instance profiles"
- "ArgoCD GitOps: every deploy is a Git commit; full history and rollback"
- "Karpenter over Cluster Autoscaler: faster, smarter, any instance type"
