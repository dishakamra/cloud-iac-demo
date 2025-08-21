# Deploy Cloud Infrastructure with GitHub Copilot

## 📌 Project Specification
This repository demonstrates how to use **GitHub Copilot** to scaffold and deploy **Infrastructure as Code (IaC)** in a **public cloud** (AWS by default, but easily adaptable to Azure/GCP).  

The goal is to deploy:
- A minimal, production-ready **VPC/network**
- **One compute instance** (EC2 on AWS, VM on Azure, Compute Engine on GCP)
- A **storage bucket** (S3/Azure Storage/GCS) with hardened defaults
- **Terraform remote backend** (S3 + DynamoDB for state locking)
- **Workspaces** for dev/stage/prod
- **CI/CD with GitHub Actions** (plan on PR, apply on merge with approval)
- **Security scanning with tfsec**

---

## 🚀 Prerequisites
- **GitHub** account with **Copilot enabled**
- Cloud CLI configured:
  - AWS → `aws configure`
  - Azure → `az login`
  - GCP → `gcloud init`
- Tools:
  - [Terraform](https://developer.hashicorp.com/terraform/downloads) or [OpenTofu](https://opentofu.org)
  - [VS Code](https://code.visualstudio.com/) with Git & GitHub Copilot extension
- Security setup:
  - Prefer **OIDC or short-lived credentials**
  - Use a **sandbox cloud account**

---

## 🏗️ Project Setup

### 1. Initialize Repository
```bash
git clone <repo-url>
cd cloud-iac-demo
```

Create `README.md` (this file) and **define your infra spec clearly**. Copilot uses it for context.

---

### 2. Bootstrap Terraform State
- Create an S3 bucket + DynamoDB table for state locking:
```bash
cd bootstrap
terraform init
terraform apply
```

---

### 3. Core Terraform Project
Files:
- `main.tf` → AWS provider, backend, VPC, subnet, IGW, SG, EC2, S3
- `variables.tf` → region, project_name, environment, my_ip_cidr
- `outputs.tf` → vpc_id, subnet_id, instance_ip, bucket_name

Example:
```hcl
terraform {
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
  backend "s3" {}
}
```

---

### 4. Workspaces & Environments
```bash
terraform workspace new dev
terraform workspace select dev
terraform plan -var-file="env/dev.tfvars"
terraform apply -var-file="env/dev.tfvars"
```

---

### 5. Local Validation
```bash
make fmt
make validate
make tfsec
make plan TFVARS=env/dev.tfvars
```

---

### 6. CI/CD with GitHub Actions
- Add `.github/workflows/iac.yml`
- Workflow:
  - Run fmt, validate, tfsec
  - Run `terraform plan` on PR
  - Require approval for `terraform apply` on main
  - Use **OIDC IAM role** (no long-lived secrets)

---

## 🔄 Development Workflow
1. Create feature branch (`feat/vpc-setup`)
2. Commit & push changes
3. Open PR → pipeline runs checks + attaches plan
4. Review → merge to `main` → apply runs with approval

---

## ✅ Verification
```bash
aws ec2 describe-instances --filters "Name=tag:Project,Values=iac-demo"
aws s3 ls | grep iac-demo
```

---

## 🧹 Teardown
```bash
terraform destroy -var-file="env/dev.tfvars"
terraform workspace select default
terraform workspace delete dev
```

---
## Repository Structure
cloud-iac-demo/
├── bootstrap/ # Bootstrap Terraform state resources (S3 bucket, DynamoDB table)
│ └── main.tf
├── env/ # Environment-specific variables
│ ├── dev.tfvars
│ ├── stage.tfvars
│ └── prod.tfvars
├── .github/
│ └── workflows/
│ └── iac.yml # GitHub Actions CI/CD pipeline
├── main.tf # Core Terraform configuration
├── variables.tf # Input variables
├── outputs.tf # Output values
├── Makefile # Local automation (fmt, validate, tfsec, plan, apply)
└── README.md # Project documentation

## 🌍 Multi-Cloud Support
- **Azure:** Replace provider with `azurerm`, use resource group, VNet, Linux VM, Storage account.
- **GCP:** Replace provider with `google`, use VPC, Compute Engine, GCS bucket.
- Copilot can scaffold alternate configs if you describe them in `README.md`.

---

## 🔒 Best Practices
- Always review Copilot’s suggestions (treat it like a **junior pair programmer**).
- Use **tfsec/checkov** for security scanning.
- Enforce **least privilege IAM** roles for OIDC.
- Keep **separate states & accounts** for dev/stage/prod.
- Use **lifecycle rules** & free-tier resources for cost control.

---

## ✨ Prompt Patterns for Copilot
- **Provider & backend**  
  > "Write Terraform provider + remote backend (S3+DynamoDB) with variables for region/project/environment and default tags."
- **Secure storage bucket**  
  > "Harden S3 bucket: block public access, enable versioning, add lifecycle for noncurrent versions in dev only."
- **Network + compute**  
  > "Create VPC with public subnet, IGW, SG for SSH from my_ip_cidr, and a t3.micro EC2 instance."
- **CI/CD pipeline**  
  > "Generate GitHub Actions that runs fmt, validate, tfsec, plan on PR; apply on main with approval."

---

## 📖 References
- [Terraform Docs](https://developer.hashicorp.com/terraform/docs)
- [tfsec](https://aquasecurity.github.io/tfsec/)
- [GitHub Actions OIDC](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
