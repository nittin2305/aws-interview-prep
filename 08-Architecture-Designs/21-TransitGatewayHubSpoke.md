# Transit Gateway Hub-Spoke Architecture

## Problem Statement
Connect 50+ VPCs across 3 AWS accounts with centralized egress, inspection, and DNS.

## Architecture Diagram

```mermaid
graph TB
  TGW["Transit Gateway\n(Network Account)"]
  subgraph SharedServices["Shared Services VPC"]
    DNS["R53 Resolver\n(Outbound Endpoint)"]
    AD["AWS Managed AD"]
    Monitor["Monitoring\n(CloudWatch Agent)"]
  end
  subgraph Egress["Egress VPC"]
    NAT["NAT Gateway"]
    NWFW["AWS Network Firewall\n(Egress Inspection)"]
    IGW["Internet Gateway"]
  end
  subgraph Prod["Production VPCs (10)"]
    P1["Prod VPC 1"]
    P2["Prod VPC 2"]
  end
  subgraph Dev["Development VPCs (10)"]
    D1["Dev VPC 1"]
    D2["Dev VPC 2"]
  end
  P1 & P2 & D1 & D2 --> TGW
  TGW --> SharedServices & Egress
  Egress --> Internet
  
  subgraph RTables["TGW Route Tables"]
    ProdRT["Prod RT\n(→ Shared, Egress only)"]
    DevRT["Dev RT\n(→ Shared, Egress only)"]
    SharedRT["Shared RT\n(→ All VPCs)"]
    EgressRT["Egress RT\n(→ All VPCs)"]
  end
```

## Route Table Segmentation
- **Prod RT**: propagates Shared + Egress only; Prod VPCs cannot reach Dev
- **Dev RT**: propagates Shared + Egress only; Dev VPCs cannot reach Prod
- **Shared RT**: propagates all VPCs; Shared Services reachable from everyone
- **Egress RT**: propagates all VPCs; Egress VPC reachable from everyone

## Centralized Egress with Inspection
1. All VPCs: `0.0.0.0/0 → TGW`
2. TGW routes to Egress VPC
3. Egress VPC: traffic hits Network Firewall → NAT Gateway → IGW
4. Network Firewall: DNS filtering, domain-based allow/deny, TLS inspection

## Interview Talking Points
- "Centralized egress = single NAT Gateway (cost savings) + single inspection point"
- "TGW route tables are the segmentation mechanism; not ACLs or SGs"
- "Shared Services VPC is accessible from all; Prod and Dev cannot reach each other"
- "RAM sharing: one TGW serves multiple accounts — owner manages route tables"
