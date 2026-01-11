# Terraform EC2 Ubuntu Demo

This repository provisions **Ubuntu EC2 instances** in AWS using Terraform. It demonstrates dynamic AMI lookup from Canonical, Free Tier–eligible instance types, and scaling from 1 to 3 instances.

---

## 📋 Prerequisites

- **AWS account** with IAM user permissions for EC2
- **Terraform CLI** (>= 1.2)
- **AWS CLI** configured with credentials
- **Git** for version control
- **SSH key pair** created in AWS (optional, for login)

---

## 📂 Project Structure

- `terraform.tf` → main configuration file
- `.gitignore` → excludes state/cache files
- `README.md` → documentation (this file)

Ignored files:
```
*.tfstate
*.tfstate.backup
.terraform/
.terraform.lock.hcl
crash.log
```

---

## 🚀 Step-by-Step Usage

### 1. Configure AWS credentials
```bash
aws configure
```
Verify:
```bash
aws sts get-caller-identity
```

---

### 2. Terraform configuration

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.92"
    }
  }
  required_version = ">= 1.2"
}

provider "aws" {
  region = "us-east-1"
}

# Ubuntu 22.04 LTS (Jammy) from Canonical
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical's AWS account ID

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

resource "aws_instance" "app_server" {
  count         = 3
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro" # Free Tier eligible

  tags = {
    Name = "Terraform_Ubuntu_Demo-${count.index}"
  }
}

output "public_ips" {
  value       = aws_instance.app_server[*].public_ip
  description = "Public IPs for SSH"
}
```

---

### 3. Initialize, validate, and plan
```bash
terraform init
terraform validate
terraform plan -out=tfplan
```

---

### 4. Apply the plan
```bash
terraform apply tfplan
```

---

### 5. Scale up or down
- Change `count` in `aws_instance` resource:
  - `count = 1` → one instance
  - `count = 3` → three instances
```bash
terraform plan -out=tfplan
terraform apply tfplan
```

---

### 6. Destroy resources
```bash
terraform destroy -auto-approve
```

---

## 🛡️ Best Practices

- **Never commit state files** (`.tfstate`, `.backup`) to GitHub.
- **Use remote state** (S3 + DynamoDB) for team projects.
- **Pin provider versions** for reproducibility.
- **Tag releases** in Git for version tracking.

---

## 🔗 Git Workflow

```bash
git init
echo "*.tfstate" >> .gitignore
echo ".terraform/" >> .gitignore
echo "*.tfstate.backup" >> .gitignore
echo ".terraform.lock.hcl" >> .gitignore
echo "crash.log" >> .gitignore

git add terraform.tf .gitignore README.md
git commit -m "Terraform EC2 Ubuntu demo"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```
---

