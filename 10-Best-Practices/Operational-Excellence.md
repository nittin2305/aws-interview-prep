# Operational Excellence Best Practices

## Infrastructure as Code
- Define all infrastructure in CloudFormation or Terraform — no manual console changes in production
- Store IaC in version control; use pull-request reviews before applying changes
- Use CloudFormation StackSets for multi-account/multi-region deployments
- Parameterise environments (dev/staging/prod) using variables / parameter files

## CI/CD
- Automate build, test, and deploy with CodePipeline or GitHub Actions
- Gate production deployments on passing tests and security scans
- Use blue/green or canary deployments to reduce deployment risk
- Store deployment artefacts in S3 / ECR with versioned tags

## Monitoring & Alerting
- Define SLOs (Service Level Objectives) and create alarms when they are breached
- Use structured logging (JSON) for easy querying in CloudWatch Logs Insights
- Correlate metrics, logs, and traces with AWS X-Ray and CloudWatch
- Set composite alarms to reduce noise (alert only when multiple signals fire)

## Runbooks & Incident Response
- Document runbooks for every alert that fires
- Use AWS Systems Manager Automation documents for self-healing remediations
- Conduct post-incident reviews (blameless post-mortems) after every incident
- Practise runbooks with regular Game Days

## Configuration Management
- Use AWS Systems Manager Parameter Store for non-secret configuration
- Use AWS Config rules to detect and auto-remediate drift
- Tag all resources with: environment, team, application, cost-centre
- Use Service Control Policies (SCPs) to prevent accidental deletion of critical resources

## Change Management
- Enable AWS CloudTrail for full API audit trail
- Use EventBridge rules to trigger automated responses to configuration changes
- Require change approval for production with AWS CodePipeline manual approval action

## Interview Questions
1. **Production is down and you have no runbook — what do you do?** — Acknowledge incident, create war-room, start with CloudWatch dashboards + alarms, trace with X-Ray, check recent deployments in CodePipeline, roll back if deployment is suspected cause.
2. **How do you ensure consistency across 50 AWS accounts?** — AWS Organizations + Control Tower, SCPs for mandatory controls, CloudFormation StackSets for baseline resources, AWS Config aggregator for compliance visibility.
3. **How do you detect configuration drift?** — AWS Config with managed rules; Config Recorder captures every configuration change; SNS notifications + automatic remediation via SSM Automation documents.
