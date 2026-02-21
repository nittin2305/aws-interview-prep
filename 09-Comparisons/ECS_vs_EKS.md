# ECS vs EKS

## Quick Decision Guide

**Use ECS when:**
- AWS-native team; no K8s expertise required
- Want simplest container orchestration
- Don't need K8s ecosystem (Helm, Operators, CRDs)
- No multi-cloud portability required
- Want zero control plane cost (ECS is free)

**Use EKS when:**
- Team already knows Kubernetes
- Need K8s ecosystem (service mesh, operators, KEDA, Argo)
- Multi-cloud portability requirement
- Need advanced scheduling (affinity, taints, custom schedulers)
- Large organization with dedicated platform team

## Detailed Comparison

| Feature | ECS | EKS |
|---------|-----|-----|
| Learning curve | Low | High (K8s complexity) |
| Control plane cost | Free | $0.10/hr (~$73/month) |
| Managed upgrades | Yes (always latest) | Semi-manual (sequential minor versions) |
| Ecosystem | AWS-native | K8s ecosystem (Helm, Operators) |
| Service mesh | App Mesh / Service Connect | Istio, Linkerd |
| Autoscaling | App Auto Scaling | HPA, VPA, KEDA, Karpenter |
| Spot integration | Fargate Spot, Capacity Providers | Karpenter (best-in-class) |
| Multi-cloud | No | Yes (same K8s API everywhere) |
| Windows containers | Yes | Yes |
| GPU workloads | EC2 launch type | EC2 node groups |
| Max task/pod size | 16 vCPU, 120 GB | Node size limit |
| Blue/Green deploy | CodeDeploy native | ArgoCD, Flagger |
| GitOps | Basic | ArgoCD, Flux (mature) |
| RBAC | IAM only | K8s RBAC + IAM |

## Migration Complexity

ECS → EKS migration:
1. Rewrite task definitions as K8s Deployments/StatefulSets
2. Replace ECS Service Connect with Istio/Linkerd
3. Replace IAM task roles with IRSA
4. Replace CodeDeploy with ArgoCD
5. Replace ALB/NLB with AWS LB Controller
- Effort: weeks to months depending on complexity

## Cost Comparison

| Configuration | Monthly Cost |
|--------------|-------------|
| ECS Fargate (same workload) | $X |
| EKS + Fargate | $X + $73 (control plane) |
| EKS + Karpenter Spot | $X × 0.5 + $73 |
| Break-even | EKS usually wins at 50+ containers with Spot |

## Interview Talking Points
- "I'd use ECS for a small team that doesn't have K8s expertise — zero ops overhead"
- "I'd use EKS when the organization already has K8s experience or needs multi-cloud"
- "The $73/month EKS control plane cost is irrelevant at scale; Karpenter saves much more"
- "Service Connect in ECS is closing the gap with K8s service mesh for simple use cases"
