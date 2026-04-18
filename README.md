# CostPatrol AWS CloudFormation Stack

CloudFormation template that creates a read-only IAM role for [CostPatrol](https://costpatrol.io) to scan your AWS account for cost optimization opportunities.

## What it creates

A single IAM role called `CostPatrolReadOnly` with read-only permissions across your AWS account. It can only **describe** and **list** resources. It cannot create, modify, or delete anything.

## Permissions summary

| Category | Actions | What CostPatrol checks |
|---|---|---|
| **Cost & Billing** | `ce:GetCostAndUsage`, `ce:GetCostForecast`, `budgets:DescribeBudgets` | Cost trends, forecasts, budget alerts |
| **Compute** | `ec2:DescribeInstances`, `ec2:DescribeVolumes`, `lambda:ListFunctions`, `ecs:DescribeServices`, `eks:DescribeCluster` | Idle EC2, oversized instances, previous-gen types, Lambda memory/architecture, ECS task sizing, EKS node groups |
| **Databases** | `rds:DescribeDBInstances`, `rds:DescribeDBClusters`, `dynamodb:DescribeTable`, `elasticache:DescribeCacheClusters` | Idle RDS, Aurora cluster sprawl, DynamoDB billing mode, ElastiCache sizing |
| **Storage** | `s3:ListAllMyBuckets`, `s3:GetLifecycleConfiguration`, `ec2:DescribeSnapshots` | Missing S3 lifecycle policies, unattached EBS volumes, stale snapshots |
| **Networking** | `ec2:DescribeNatGateways`, `elasticloadbalancing:DescribeLoadBalancers`, `cloudfront:ListDistributions` | NAT Gateway vs VPC endpoints, idle load balancers, CloudFront price class |
| **Monitoring** | `cloudwatch:GetMetricData`, `logs:DescribeLogGroups`, `cloudtrail:DescribeTrails` | CloudWatch log retention, metric stream filtering |

Every action is **Describe**, **List**, or **Get**. No Put, Create, Update, or Delete.

## Security

- **Cross-account access** secured with ExternalId (prevents confused deputy attacks)
- **Session duration** limited to 1 hour with temporary credentials
- **Revoke anytime** by deleting the CloudFormation stack

## Parameters

| Parameter | Description |
|---|---|
| `CostPatrolAccountId` | CostPatrol's AWS Account ID (provided during onboarding) |
| `ExternalId` | Unique external ID for secure access (provided during onboarding) |
| `Environment` | Optional: `production` or `non-production` for threshold tuning |

## Deploy

The template is deployed automatically during CostPatrol onboarding at [costpatrol.io/account](https://costpatrol.io/account). The "Launch in AWS Console" button pre-fills all parameters.

To deploy manually:

```bash
aws cloudformation create-stack \
  --stack-name CostPatrolReadOnly \
  --template-body file://costpatrol-read-only-role.yaml \
  --parameters \
    ParameterKey=CostPatrolAccountId,ParameterValue=YOUR_COSTPATROL_ACCOUNT_ID \
    ParameterKey=ExternalId,ParameterValue=YOUR_EXTERNAL_ID \
  --capabilities CAPABILITY_NAMED_IAM
```

## Revoke access

```bash
aws cloudformation delete-stack --stack-name CostPatrolReadOnly
```

This removes the IAM role and all permissions immediately.

## Links

- [CostPatrol](https://costpatrol.io) -- AWS cost optimization tool
- [How it works](https://costpatrol.io/aws-cost-optimization-tool) -- Full product overview
- [Security](https://costpatrol.io/security) -- Security details
- [mscloudtech.io](https://mscloudtech.io) -- Parent company
