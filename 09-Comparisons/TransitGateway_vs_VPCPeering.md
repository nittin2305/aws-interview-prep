# Transit Gateway vs VPC Peering

## Quick Decision Guide

**Use VPC Peering when:**
- 2-4 VPCs with simple connectivity needs
- Lowest possible latency required (no hop through TGW)
- No transitive routing needed
- Cost is primary concern

**Use Transit Gateway when:**
- 5+ VPCs
- Need transitive routing (A → B → C)
- On-premises connectivity to multiple VPCs via single attachment
- Need network segmentation via route tables
- Central egress/inspection architecture

## Detailed Comparison

| Feature | VPC Peering | Transit Gateway |
|---------|------------|----------------|
| Max connections | 125 per VPC | Thousands |
| Transitive routing | No | Yes |
| Bandwidth | No limit | Scales automatically |
| On-prem via DX/VPN | Per-peering | Single TGW attachment |
| Route management | Per-VPC route tables (n^2 complexity) | Centralized TGW route tables |
| Cross-account | Yes | Yes (RAM sharing) |
| Cross-region | Yes | Yes (TGW peering, static routes) |
| Cost | $0.01/GB cross-AZ | $0.05/hr + $0.02/GB |
| Latency | Lower (direct) | Slightly higher (extra hop) |
| Overlapping CIDRs | Not supported | Not supported |
| BGP | No | Yes (with DX/VPN) |
| Multicast | No | Yes |

## Cost Analysis at Scale

**5 VPCs full mesh (peering):**
- 10 peering connections × $0.01/GB = $0.10/GB
- 10 peering connections management complexity

**5 VPCs via TGW:**
- 5 attachments × $0.05/hr = $0.25/hr = $180/month
- $0.02/GB data processed
- Break-even: depends on GB volume; TGW wins at management scale

**10 VPCs full mesh:**
- 45 peering connections vs 10 TGW attachments
- Route tables: 9 routes per VPC (complex) vs 1-few TGW route tables (simple)
- TGW wins clearly at 10+ VPCs

## The Transitivity Trap
VPC Peering is NON-TRANSITIVE:
```
A ↔ B peered
B ↔ C peered
A CANNOT reach C through B
```
TGW IS transitive:
```
A → TGW → C (via route table entries)
```

## Interview Trap
"We have 3 VPCs — should I use TGW?"
Answer: Depends on what else you need:
- Need on-prem connectivity to all 3? → TGW (single DX/VPN attachment)
- Need transitive routing? → TGW
- Pure VPC-to-VPC with stable topology? → Peering (cheaper, simpler)
