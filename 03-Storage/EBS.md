# EBS — Senior Interview Guide

## Volume Types Comparison

| Type | Category | Max IOPS | Max Throughput | Max Size | Key Characteristic |
|------|----------|---------|----------------|---------|-------------------|
| gp3 | SSD General | 16,000 | 1,000 MB/s | 16 TB | IOPS/throughput independent of size |
| gp2 | SSD General | 16,000 | 250 MB/s | 16 TB | IOPS = 3x GiB (burst credits) |
| io2 Block Express | SSD Provisioned | 256,000 | 4,000 MB/s | 64 TB | Sub-millisecond latency; multi-attach |
| io1 | SSD Provisioned | 64,000 | 1,000 MB/s | 16 TB | High performance; legacy |
| st1 | HDD Throughput | 500 | 500 MB/s | 16 TB | Sequential big data |
| sc1 | HDD Cold | 250 | 250 MB/s | 16 TB | Lowest cost; cold data |

**gp3 is the default and recommended**: 3,000 IOPS and 125 MB/s baseline included; increase independently

---

## gp2 Burst Credit Mechanics

- Baseline: 3 IOPS per GiB (minimum 100 IOPS)
- Burst: up to 3,000 IOPS (volumes < 1000 GiB)
- Credit bucket: earns 3 credits/GiB/second; max 5.4M credits (3,000 IOPS × 30 min)
- **Problem**: when credits exhausted, drops to baseline IOPS — performance cliff
- **gp3 eliminates this**: 3,000 IOPS baseline regardless of size; no burst credits

```
gp2 1 GiB volume: baseline = 3 IOPS; burst = 3,000 IOPS; credits drain in <2 minutes
gp2 100 GiB volume: baseline = 300 IOPS; burst = 3,000 IOPS; longer credit period
gp2 334+ GiB volume: baseline = 1,002+ IOPS; never needs to burst
gp3 any volume: always 3,000 IOPS baseline; pay extra for up to 16,000
```

---

## Multi-Attach (io1/io2)

- Attach single EBS volume to multiple EC2 instances simultaneously (up to 16)
- Instances must be in same AZ
- Use case: shared storage for clustered applications (Oracle RAC, DRBD)
- Application must manage concurrent writes (EBS does not provide distributed lock)
- Not supported by most standard file systems — use cluster-aware FS

---

## Snapshots

- Incremental: first snapshot is full; subsequent only changed blocks
- Stored in S3 (managed by AWS; not in your bucket)
- **Fast Snapshot Restore (FSR)**: pre-warms snapshot; volumes created from FSR snapshot have full performance immediately (vs normal: need initialization I/O to avoid first-access latency)
- **Snapshot Archive**: 75% cheaper storage for rarely accessed snapshots; retrieval takes 24-72 hours
- Cross-region copy: for DR; can be automated with Data Lifecycle Manager

---

## EBS Encryption

- **At-rest encryption**: AES-256; data, snapshots, and volumes derived from snapshots are all encrypted
- **In-transit**: encrypted via NVMe/SRD between EC2 and EBS
- KMS key used per volume
- **Default encryption**: enable account-level default encryption — all new volumes encrypted automatically
- Snapshots of encrypted volumes are encrypted; snapshots of unencrypted are not
- **Trap**: You cannot encrypt an existing unencrypted volume directly; must snapshot → copy with encryption → create new volume

---

## RAID on EBS

| RAID Level | IOPS | Fault Tolerance | Use Case |
|-----------|------|----------------|---------|
| RAID 0 | Additive (4 × 10K = 40K IOPS) | None (any volume failure = data loss) | Max performance, stateless |
| RAID 1 | Same as single volume | Yes (mirroring) | Durability + redundancy |
| RAID 5/6 | Not recommended on EBS | N/A | Parity overhead reduces performance |

---

## NVMe Instance Store vs EBS

| Feature | Instance Store (NVMe) | EBS |
|---------|----------------------|-----|
| Persistence | Lost on stop/terminate | Persists independently |
| IOPS | Millions (NVMe flash) | Up to 256,000 (io2 BE) |
| Latency | Sub-100 microseconds | Sub-millisecond |
| Cost | Included in instance price | Additional per GB-month |
| HA | None (AZ-bound, ephemeral) | AZ-bound but persists |
| Use case | Temp files, caches, Hadoop HDFS | Databases, persistent data |

---

## Most Asked Senior Interview Questions

**Q1: gp2 vs gp3 — why should you migrate to gp3?**
- gp3: 3,000 IOPS + 125 MB/s baseline vs gp2: baseline = 3×GiB IOPS
- gp3 is ~20% cheaper than gp2 at same size
- gp3 IOPS and throughput are tunable independently of volume size
- No burst credit exhaustion risk
- **Migration**: modify volume type in-place; no data loss, no downtime (brief performance dip during optimization)

**Q2: How do you recover data from an EBS volume in a failed AZ?**
- EBS is AZ-bound; if AZ fails, volume may be inaccessible
- Recovery: use the most recent snapshot (cross-region copy for DR)
- Best practice: automate snapshots with Data Lifecycle Manager
- For critical data: use RDS Multi-AZ (synchronous replication) or EFS (multi-AZ by design)

**Q3: What happens to EBS volumes when you terminate an instance?**
- Root volume: deleted by default (`DeleteOnTermination: true`)
- Additional data volumes: NOT deleted by default
- **Best practice**: review `DeleteOnTermination` flag per volume; set explicitly in launch template
- **Trap**: Can change `DeleteOnTermination` on running instance for root volume — yes, via `modify-instance-attribute`

**Q4: How do you move an EBS volume to another AZ?**
- Cannot move directly: EBS volumes are AZ-locked
- Process: snapshot → create new volume from snapshot in target AZ
- Use Data Lifecycle Manager for automated cross-AZ snapshot management

**Q5: EBS encryption after the fact — how do you encrypt an unencrypted volume?**
1. Create snapshot of unencrypted volume
2. Copy snapshot with encryption enabled (KMS key)
3. Create new volume from encrypted snapshot
4. Stop instance; detach old volume; attach new encrypted volume
5. Start instance; test; terminate old unencrypted volume
- Alternative: enable default encryption going forward — only affects new volumes

**Q6: When would you use io2 Block Express instead of gp3?**
- When you need > 16,000 IOPS (gp3 max)
- Sub-millisecond consistent latency (SAP HANA, Oracle, SQL Server)
- io2 Block Express: up to 256,000 IOPS, 4,000 MB/s, 64 TB
- Multi-attach for clustered databases
- Cost: 4-5x more expensive than gp3 — only use when actually needed

**Q7: How do you optimize EBS for a write-heavy database?**
- Use gp3 with provisioned IOPS matching workload requirements
- Enable EBS-optimized instances (dedicated bandwidth to EBS)
- Use io2/io2 BE for highest IOPS requirements
- RAID 0 across multiple volumes if single volume IOPS insufficient
- Use NVMe instance store for temp tablespace (fastest)
- Monitor: `VolumeReadOps`, `VolumeWriteOps`, `VolumeQueueLength` (should be < 1 per volume)

---

## Scenario-Based Questions

**Scenario 1: gp2 volume showing IOPS throttling during peak hours**
- Diagnosis: check `BurstBalance` CloudWatch metric — dropping to 0
- Root cause: burst credits exhausted; baseline insufficient
- Options:
  1. Upgrade to gp3: set IOPS to sustained requirement (e.g., 5,000 IOPS, cheaper than io1)
  2. Increase gp2 volume size (increases baseline IOPS) — less efficient
  3. Use gp3 with 5,000 IOPS: $0.08/GB + $0.005/provisioned IOPS = cost-effective

**Scenario 2: Database needs multi-AZ storage with high IOPS**
- Single EBS cannot span AZs — not the right solution
- Options:
  1. RDS Multi-AZ: managed synchronous replication; use for managed databases
  2. EC2 with io2 BE + application-level replication (e.g., MySQL replication)
  3. FSx for NetApp ONTAP: multi-AZ NFS/iSCSI storage (high IOPS, multi-AZ)
- EBS is appropriate for single-AZ, high-performance storage; not for multi-AZ directly

---

## Real-world Failure Cases

**1. EBS volume stuck in detaching state**
- Force detach: `aws ec2 detach-volume --volume-id vol-xxx --force`
- May corrupt file system if I/O was in progress — always unmount before detaching

**2. EBS throughput throttling (not IOPS)**
- gp3 max throughput: 1,000 MB/s; gp2: 250 MB/s
- Sequential scan of large tables hitting throughput cap
- Fix: use st1 for sequential big data; or split across multiple volumes in RAID 0

**3. Snapshot taking too long / incomplete**
- Large volumes (16 TB) can take hours for first snapshot
- Fix: run snapshots during low-I/O periods; use concurrent incremental snapshots (AWS handles parallelism)

**4. Instance launch failing — gp2 I/O credit exhaustion**
- Boot volume is gp2 8 GiB; baseline = 24 IOPS; exhausted during boot
- Fix: upgrade to gp3; or increase volume size; or use larger volume size gp2 (100 GiB+)

---

## Cost Optimization

- **Migrate gp2 to gp3**: 20% cost savings, better baseline performance
- **Right-size provisioned IOPS**: don't provision 10,000 IOPS when average is 1,000
- **Snapshot lifecycle policies**: DLM to auto-delete old snapshots (major cost culprit)
- **Snapshot archive**: 75% cheaper for compliance snapshots not needing fast restore
- **sc1 for cold data**: $0.015/GB vs gp3 $0.08/GB = 5x cheaper for infrequent access

---

## Security Considerations

- **Default encryption**: enable at account level via `EnableEbsEncryptionByDefault`
- **KMS**: use CMK (not AWS-managed) for key control + rotation + cross-account sharing
- **Snapshot sharing**: encrypted snapshots can be shared with specific accounts; KMS key must also be shared
- **Delete on termination**: set deliberately for each volume type in launch template
- **CloudTrail**: track `CreateVolume`, `CreateSnapshot`, `DeleteSnapshot` API calls

---

## EBS vs EFS vs Instance Store

| Feature | EBS | EFS | Instance Store |
|---------|-----|-----|----------------|
| Access | Single instance (multi-attach: io1/io2) | Multiple instances/AZs | Single instance |
| Persistence | Yes | Yes | No (ephemeral) |
| Performance | Up to 256K IOPS | General Purpose or Max I/O | Millions IOPS |
| Multi-AZ | No (AZ-bound) | Yes | No |
| Protocol | Block (NVMe) | NFS | Block (NVMe) |
| Best for | Databases, OS volumes | Shared filesystems, CMS | Temp storage, caches |

---

## Quick Revision Bullets

- gp3: 3,000 IOPS + 125 MB/s baseline; IOPS/throughput independent of size; migrate from gp2
- gp2 burst credits: earn at 3/GiB/sec; exhaustion = performance cliff
- io2 Block Express: 256K IOPS, 4K MB/s, multi-attach, sub-ms latency
- EBS snapshots: incremental; stored in S3 (managed); cross-region copy for DR
- FSR (Fast Snapshot Restore): pre-warmed volumes; no initialization I/O penalty
- Default encryption at account level: all new volumes encrypted; cannot encrypt in-place
- gp2→gp3: in-place modification, no downtime, 20% cheaper
- Instance store: ephemeral, millions IOPS, included in price, lost on stop/terminate
