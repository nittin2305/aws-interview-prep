# AWS Security Best Practices

## Identity & Access Management
- Apply **least-privilege** principle to every IAM entity
- Enforce MFA on root account and all human users
- Use IAM roles for EC2/Lambda/ECS — never embed credentials in code
- Rotate access keys regularly; prefer short-lived credentials via STS
- Use AWS Organizations SCPs to enforce guardrails across accounts
- Enable IAM Access Analyzer to detect overly permissive policies

## Data Protection
- Encrypt all data at rest using AWS KMS (SSE-S3, SSE-KMS, or client-side)
- Enforce TLS 1.2+ for all data in transit
- Store secrets in AWS Secrets Manager or Parameter Store (SecureString)
- Enable S3 Block Public Access at account and bucket level
- Enable S3 versioning and MFA Delete on critical buckets

## Network Security
- Use private subnets for application and database tiers
- Apply restrictive Security Groups (deny by default, explicit allow)
- Use NACLs as a second layer of defence at subnet boundary
- Enable VPC Flow Logs and centralise to S3/CloudWatch
- Use AWS WAF to protect public-facing APIs and CloudFront distributions
- Use AWS Shield Advanced for DDoS protection on critical endpoints

## Detection & Monitoring
- Enable AWS CloudTrail in all regions; store logs in a dedicated logging account
- Enable AWS Config to detect configuration drift
- Enable GuardDuty across all accounts for threat detection
- Enable Security Hub to aggregate findings
- Set CloudWatch alarms on CloudTrail events (root login, policy changes)

## Incident Response
1. Isolate: detach IAM roles, modify security groups to deny all
2. Investigate: CloudTrail, VPC Flow Logs, GuardDuty findings
3. Contain: revoke temporary credentials, snapshot affected instances
4. Eradicate: patch/rebuild affected resources
5. Recover: restore from clean backup, apply lessons learned

## Compliance Controls
- Use AWS Config rules mapped to CIS Benchmarks / PCI-DSS / HIPAA
- Use Control Tower for multi-account governance
- Enable Macie for S3 sensitive data discovery (PII, credentials)

## Interview Questions
1. **An IAM access key was accidentally committed to GitHub.** — Immediately deactivate the key, create a new one, audit CloudTrail for usage, rotate and invalidate any derived credentials.
2. **How do you prevent privilege escalation in IAM?** — Deny `iam:PassRole`, `iam:CreatePolicyVersion`, `iam:AttachRolePolicy` except to admin roles; use permission boundaries.
3. **Design a secure multi-account AWS environment.** — AWS Organizations + Control Tower, SCPs for guardrails, dedicated Security & Logging accounts, centralised CloudTrail, GuardDuty, Security Hub.
