# Terraform / Terragrunt Patterns

## Module Design

- Modules should be self-contained with clear inputs/outputs
- Use `for_each` over `count` for resources that need stable identifiers
- Tag all resources consistently
- Keep modules in `_modules/` directory, referenced by numbered layers

```hcl
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "terraform"
  }
}

resource "aws_ecs_service" "this" {
  for_each = var.services

  name            = each.key
  cluster         = aws_ecs_cluster.this.id
  task_definition = each.value.task_definition
  desired_count   = each.value.desired_count

  tags = merge(local.common_tags, { Service = each.key })
}
```

## Terragrunt Dependency Graph

Dependencies flow downward through layer numbers:

```hcl
# 300-ecs depends on 100-vpc and 200-rds
dependency "vpc" {
  config_path = "../100-vpc"
  mock_outputs = {
    vpc_id             = "vpc-mock"
    private_subnet_ids = ["subnet-mock"]
  }
}

dependency "rds" {
  config_path = "../200-rds"
  mock_outputs = {
    connection_string = "postgres://mock:5432/db"
  }
}
```

Always include `mock_outputs` so `terragrunt validate` works without deployed dependencies.

## DRY with _envcommon

```hcl
# _envcommon/ecs.hcl
terraform {
  source = "${dirname(find_in_parent_folders())}/_modules/ecs-service"
}

inputs = {
  cluster_name = "main"
  cpu          = 256
  memory       = 512
}
```

Environment overrides only what differs:

```hcl
# prod/300-ecs/terragrunt.hcl
include "envcommon" {
  path   = "${dirname(find_in_parent_folders())}/_envcommon/ecs.hcl"
  expose = true
}

inputs = {
  cpu    = 1024    # Override for prod
  memory = 2048
}
```

## Data Sources Over Hardcoding

```hcl
# GOOD: Look up dynamically
data "aws_vpc" "main" {
  filter {
    name   = "tag:Name"
    values = ["main-vpc"]
  }
}

# BAD: Hardcoded ID
resource "aws_subnet" "this" {
  vpc_id = "vpc-12345678"
}
```

## Conditional Resources

```hcl
resource "aws_cloudwatch_metric_alarm" "high_cpu" {
  count = var.enable_monitoring ? 1 : 0
  # ...
}
```

## Lifecycle Rules

```hcl
resource "aws_ecs_task_definition" "this" {
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_rds_instance" "this" {
  lifecycle {
    prevent_destroy = true  # Protect persistent layer resources
  }
}
```

## Security Patterns

- Use IAM roles, never access keys in code
- Encrypt at rest (KMS) and in transit (TLS)
- Use security groups with least-privilege rules
- Enable VPC flow logs and CloudTrail
- Use SCPs for guardrails at the org level
