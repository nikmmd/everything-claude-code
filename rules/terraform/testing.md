# Terraform Testing

## Validation Pipeline

```bash
terraform fmt -check -recursive    # Format check
terraform validate                 # Syntax validation
tflint --recursive                 # Lint
tfsec .                            # Security scan
checkov -d .                       # Policy checks
terraform plan                     # Dry run
```

## Integration Testing with Terratest

```go
func TestVPCModule(t *testing.T) {
    terraformOptions := &terraform.Options{
        TerraformDir: "../modules/vpc",
        Vars: map[string]interface{}{
            "cidr_block": "10.0.0.0/16",
            "name":       "test-vpc",
        },
    }

    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)

    vpcID := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcID)
}
```

## Plan Testing

Always review `terraform plan` output for:
- Unexpected destroys/recreates
- Changes to immutable attributes
- Security group rule modifications
- IAM policy changes

Scope: `**/*.tf`, `**/*.tfvars`
