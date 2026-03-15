# Terraform / Terragrunt Coding Style

## Naming

- Resources: `snake_case` — `aws_ecs_service.web_app`
- Variables: `snake_case` — `var.cluster_name`
- Outputs: `snake_case` — `output.alb_dns_name`
- Files: `main.tf`, `variables.tf`, `outputs.tf`, `versions.tf`, `locals.tf`

## Numbered Layer Organization

Use numbered prefixes to establish clear dependency ordering:

```
infrastructure/
├── 100-vpc/                    # Networking & foundation
│   ├── terragrunt.hcl
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── 101-security-groups/
├── 102-route53/
│
├── 200-rds/                    # Persistent / data layer
├── 201-s3/
├── 202-elasticache/
├── 203-sqs/
│
├── 300-ecs/                    # Application layer
├── 301-alb/
├── 302-cloudfront/
├── 303-lambda/
│
├── 400-eks/                    # Clustering (reserved)
├── 401-eks-addons/
├── 402-argocd/
│
└── _modules/                   # Reusable modules
    ├── vpc/
    ├── ecs-service/
    └── rds-cluster/
```

**Layer numbering:**
| Range | Layer | Purpose |
|-------|-------|---------|
| 100-199 | Networking & Foundation | VPC, subnets, security groups, DNS, NAT, VPN |
| 200-299 | Persistent / Data | RDS, S3, ElastiCache, SQS, DynamoDB, secrets |
| 300-399 | Application | ECS, ALB, CloudFront, Lambda, API Gateway |
| 400-499 | Clustering | EKS, node groups, addons, service mesh |

Resources in higher layers depend on lower layers. Never create upward dependencies.

## Terragrunt Structure

```
live/
├── terragrunt.hcl              # Root config (remote state, provider)
├── _envcommon/                  # Shared terragrunt configs
│   ├── vpc.hcl
│   ├── rds.hcl
│   └── ecs.hcl
├── dev/
│   ├── env.hcl                 # Environment-specific vars
│   ├── 100-vpc/
│   │   └── terragrunt.hcl     # include + dependency blocks
│   ├── 200-rds/
│   │   └── terragrunt.hcl
│   └── 300-ecs/
│       └── terragrunt.hcl
└── prod/
    ├── env.hcl
    ├── 100-vpc/
    ├── 200-rds/
    └── 300-ecs/
```

## Terragrunt Patterns

```hcl
# live/dev/300-ecs/terragrunt.hcl
include "root" {
  path = find_in_parent_folders()
}

include "envcommon" {
  path   = "${dirname(find_in_parent_folders())}/_envcommon/ecs.hcl"
  expose = true
}

dependency "vpc" {
  config_path = "../100-vpc"
}

dependency "rds" {
  config_path = "../200-rds"
}

inputs = {
  vpc_id     = dependency.vpc.outputs.vpc_id
  subnet_ids = dependency.vpc.outputs.private_subnet_ids
  db_url     = dependency.rds.outputs.connection_string
}
```

## Variables

- Always include `description` and `type`
- Use `validation` blocks for constraints
- Use `sensitive = true` for secrets
- Prefer `locals` for computed values

## State Management

- Always use remote state (S3 + DynamoDB)
- One state file per numbered component (Terragrunt handles this)
- Never store secrets in state — use SSM Parameter Store or Secrets Manager
- Lock state during operations

## Tools

- **Linting**: `tflint --recursive`
- **Security**: `tfsec .`, `checkov -d .`
- **Formatting**: `terraform fmt -recursive`
- **Validation**: `terraform validate`
- **Docs**: `terraform-docs markdown .`
- **Terragrunt**: `terragrunt run-all plan`, `terragrunt run-all apply`
