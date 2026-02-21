# CloudFormation — Senior Interview Guide

## Core Concepts

- **Template**: JSON/YAML defining resources
- **Stack**: deployed instance of a template
- **Change Set**: preview changes before applying
- **Stack Set**: deploy stack to multiple accounts/regions
- **Drift Detection**: identify manual changes to stack resources
- **Rollback**: automatic on failure; configures resource state to previous

---

## Template Structure

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'My Stack'
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]
Mappings:
  EnvConfig:
    prod:
      InstanceType: m6i.large
    dev:
      InstanceType: t3.small
Conditions:
  IsProd: !Equals [!Ref Environment, prod]
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !FindInMap [EnvConfig, !Ref Environment, InstanceType]
Outputs:
  InstanceId:
    Value: !Ref MyInstance
    Export:
      Name: !Sub '${AWS::StackName}-InstanceId'
```

---

## Stack Features

### Nested Stacks
- Break large templates into reusable child stacks
- Parent stack manages child stack lifecycle
- Use case: VPC stack + ECS stack + RDS stack composed in parent

### Cross-Stack References (Outputs/Exports)
- Export values from one stack; import in another with `!ImportValue`
- Creates dependency: cannot delete exporting stack while importing stack exists

### Stack Sets
- Deploy same stack to multiple accounts + regions from management account
- Deployment order: sequential (one at a time) or parallel
- Failure handling: configurable (stop on failure vs continue)
- Permission models: self-managed (IAM roles) or service-managed (Organizations)

---

## Custom Resources

- Lambda-backed: run arbitrary code during stack create/update/delete
- Use cases: DNS registration, third-party API calls, resource types not natively supported
- Must send response to pre-signed S3 URL within 1 hour or stack fails

---

## Most Asked Senior Interview Questions

**Q1: How do you update a stack without downtime?**
- Use Change Set: preview changes before executing
- Blue/Green infrastructure: provision new resources alongside old; switch traffic; delete old
- `UpdateReplacePolicy: Retain`: retain resource when CloudFormation replaces it (don't delete old DB)
- `DeletionPolicy: Retain`: keep resource if stack is deleted

**Q2: How do you handle stateful resources (RDS, S3) in CloudFormation?**
- Set `DeletionPolicy: Retain` on stateful resources
- Set `UpdateReplacePolicy: Retain` to prevent replacement
- Use parameter for DB password; store in Secrets Manager; reference ARN in CF
- Never hardcode passwords in templates

**Q3: CloudFormation vs Terraform — key differences?**
| Feature | CloudFormation | Terraform |
|---------|--------------|----------|
| Ecosystem | AWS only | Multi-cloud |
| State | AWS managed (no state file) | Local or remote state file |
| Resource coverage | All AWS (sometimes faster for new services) | Large (community providers) |
| Rollback | Automatic on failure | Manual (`terraform destroy`) |
| Drift detection | Built-in | `terraform plan` |
| Language | JSON/YAML | HCL |
| Modules | Nested stacks, CDK | Terraform modules |

**Q4: How do you prevent accidental stack deletion?**
- Enable termination protection on stack
- DeletionPolicy: Retain on critical resources (RDS, S3)
- SCP: deny cloudformation:DeleteStack on production accounts
- Stack policy: deny updates to critical resources

**Q5: StackSets with AWS Organizations — how does it work?**
- Management or delegated admin account manages StackSets
- Service-managed: AWS automatically creates execution roles in member accounts
- Deploy to OU: all accounts in OU get the stack (new accounts auto-enrolled if configured)
- Use case: deploy security baselines, logging, GuardDuty to all accounts automatically

---

## Real-world Failure Cases

**1. Stack rollback failing — leaving stack in UPDATE_ROLLBACK_FAILED**
- Rollback tried to restore resource that was manually deleted
- Fix: `ContinueUpdateRollback` skipping the problematic resource; or manually recreate the resource
- Prevention: never manually modify stack resources; use drift detection

**2. Circular dependency in template**
- Resource A depends on B; B depends on A
- Fix: use DependsOn carefully; break circular deps with custom resources; pass ARNs as parameters

---

## Quick Revision Bullets

- Change Set: preview before apply; use for production changes
- DeletionPolicy: Retain = keep resource when stack deleted; Snapshot = snapshot before delete
- StackSets with Organizations: auto-deploy to all accounts in OU
- Custom Resources: Lambda-backed for unsupported resources
- Drift detection: find manual changes to stack resources
- Termination protection + DeletionPolicy: Retain = protect prod stacks
