# Terraform

Infrastructure as Code (IaC) tool. You write config files describing what you want — Terraform figures out how to create and manage it.

**Key difference from boto3:** boto3 is procedural (you write each step). Terraform is **declarative** — you describe desired state, it computes the diff and executes only what's needed.

Benefits: multi-cloud provisioning, consistent teardown, reproducible and versionable infra.

---

## Core Concepts

### Providers

Plugins for each cloud or service (AWS, Azure, GCP, Kubernetes, GitHub, etc.). Configured in a `provider` block.

### Resource

Actual infrastructure you want to create or manage:

```hcl
# Syntax: resource "<TYPE>" "<NAME>"
resource "aws_s3_bucket" "test_artifacts" {
  bucket = "netapp-test-results-2024"
}
```

### Data Source

Reads **existing** information from your cloud — doesn't create anything. Use to avoid hardcoding values that change (like AMI IDs):

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*"]
  }
}

# Reference: data.<TYPE>.<NAME>.<ATTRIBUTE>
resource "aws_instance" "app" {
  ami = data.aws_ami.ubuntu.id
}
```

---

## HCL — HashiCorp Configuration Language

Files have `.tf` extension.

```hcl
<BLOCK_TYPE> "<LABEL1>" "<LABEL2>" {
  argument = value
}
```

| Block | Labels | Example |
|---|---|---|
| `resource` | 2 | `resource "aws_s3_bucket" "my_bucket" {}` |
| `provider` | 1 | `provider "aws" {}` |
| `terraform` | 0 | `terraform { required_version = ">= 1.2" }` |

### Variables

Define in `variables.tf`, use with `var.<name>`:

```hcl
variable "region" {
  type    = string
  default = "us-east-1"
}

provider "aws" {
  region = var.region
}
```

### Outputs

Define in `outputs.tf`. Values are printed after `apply` and stored in state. Retrieve anytime with `terraform output`.

```hcl
output "instance_ip" {
  value = aws_instance.app.public_ip
}
```

Useful in automation pipelines — e.g., output the EC2 IP address for a downstream test script.

### Provider Block

Configures the provider itself:

```hcl
provider "aws" {
  region  = "us-east-1"
  profile = "my-profile"   # optional, from ~/.aws/credentials
}
```

---

## Modules

A reusable collection of resources packaged together. Instead of writing 20 resources for a VPC, use a community module:

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"
}
```

After adding a new module, always run `terraform init` again to download it.

---

## Core Workflow

```
terraform init
    │  Download providers and modules. Creates .terraform/ dir, locks versions.
    │  Re-run whenever you add a new provider or module.
    ▼
terraform fmt
    │  Format .tf files to HashiCorp canonical style.
    ▼
terraform validate
    │  Check syntax and internal consistency (typos, wrong args, missing required fields).
    │  Does NOT call any cloud APIs.
    ▼
terraform plan
    │  Show exact changes Terraform will make. Review before applying.
    │  In CI/CD: save with -out=plan.tfplan, then apply that exact plan.
    ▼
terraform apply
    │  Shows plan, prompts yes/no, then executes.
    ▼
terraform destroy
       Tears down all managed infrastructure.
```

---

## Terraform State

`terraform.tfstate` — records every resource Terraform manages: real IDs, attributes, current state. **State is the source of truth.**

**In teams:** always use remote state (e.g. S3 backend with DynamoDB locking) so the state file isn't local and multiple people/pipelines don't conflict.

### Plan symbols

| Symbol | Meaning | Notes |
|---|---|---|
| `+` | Create | New resource |
| `-` | Destroy | Resource removed from config |
| `~` | Update in place | Change a tag, resize — no downtime |
| `-/+` | Destroy and recreate | e.g. change VPC subnet — **causes downtime** |

`-/+` changes need careful handling in production — they delete the old resource before creating the new one.

---

## Key Interview Takeaways

- **Declarative vs procedural** — Terraform describes desired state, not steps. You say "I want this"; Terraform figures out how.
- **`plan` before `apply` always** — in production/CI/CD, always plan first. Save with `-out=plan.tfplan` and apply that exact plan.
- **State is source of truth** — Terraform needs state to know what it manages. Use remote state (S3 backend) in teams.
- **`-/+` means downtime** — destroy-and-recreate changes need careful handling in production.
- **Modules = reusability** — don't reinvent the wheel. Use community modules for VPCs, EKS clusters, etc.
