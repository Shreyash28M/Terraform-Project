#  AWS Infrastructure Automation with Terraform

> **Production-grade Infrastructure as Code (IaC) — Provisioning a secure, scalable, multi-tier AWS architecture using Terraform with remote state management, modular design, and automated networking.**

# CI/CD Pipeline using Jenkins

 This pipeline automates:
- Docker image build & push to DockerHub
- Infrastructure provisioning using Terraform
- Deployment to AWS EKS cluster
- Kubernetes resource verification
---

<div align="center">

![Terraform](https://img.shields.io/badge/Terraform-1.x-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-Cloud-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![IaC](https://img.shields.io/badge/IaC-Infrastructure_as_Code-0078D4?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Production_Ready-28a745?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)
![Docker Official Images](https://img.shields.io/badge/Docker-Official_Images-0db7ed?style=for-the-badge&logo=docker&logoColor=white)
</div>

---

## 📌 Project Overview

This project demonstrates **end-to-end AWS infrastructure provisioning** using Terraform — one of the most in-demand skills in modern DevOps and Cloud Engineering roles.

The infrastructure follows AWS **Well-Architected Framework** principles:
- Multi-tier architecture with strict network isolation
- Public and private subnet separation
- Secure SSH access via Bastion Host
- Highly available application layer behind a Load Balancer
- Managed relational database in a private subnet

**No manual console clicks. No configuration drift. Everything is code.**

---

##  Architecture

```
Internet
    │
    ▼
[ Internet Gateway ]
    │
    ▼
[ Application Load Balancer ]   ◄──── Public Subnet
    │              │
    ▼              ▼
[ EC2 App 1 ]  [ EC2 App 2 ]   ◄──── Private Subnet (App Layer)
    │              │
    └──────┬───────┘
           ▼
    [ RDS Database ]             ◄──── Private Subnet (DB Layer)
           │
           ▼
    [ RDS Read Replica ]         ◄──── High Availability

[ Bastion Host ] ──── SSH ────► EC2 Instances
  (Public Subnet)               (Port 22, restricted SG)
```

---

## ⚙️ AWS Resources Provisioned

| Resource | Description |
|---|---|
| **VPC** | Custom Virtual Private Cloud with CIDR block |
| **Public Subnets** | For Load Balancer and Bastion Host (internet-facing) |
| **Private Subnets** | For EC2 App instances and RDS database (no direct internet access) |
| **Internet Gateway** | Enables outbound internet access from VPC |
| **NAT Gateway** | Allows private subnet instances to reach internet for updates |
| **Application Load Balancer (ALB)** | Distributes HTTP/HTTPS traffic across EC2 instances |
| **EC2 Instances** | Application servers running the web app on port 8080 |
| **Auto Scaling Group** | Dynamically scales EC2 capacity based on demand |
| **RDS (MySQL/PostgreSQL)** | Managed relational database in isolated private subnet |
| **RDS Read Replica** | High availability and read scaling |
| **Bastion Host** | Secure SSH jump server for accessing private instances |
| **Security Groups** | Least-privilege firewall rules for each tier |
| **Route Tables** | Routing rules for public and private subnets |
| **Key Pair** | SSH key management for EC2 access |
| **S3 + DynamoDB** | Remote Terraform state storage and state locking |

---

## 🗂️ Project Structure

```
Terraform-Project/
├── main.tf               # Root module — ties all resources together
├── variables.tf          # Input variable declarations
├── outputs.tf            # Output values (ALB DNS, instance IPs, etc.)
├── provider.tf           # AWS provider and region configuration
├── backend.tf            # Remote state: S3 bucket + DynamoDB lock table
│
├── modules/
│   ├── vpc/              # VPC, subnets, IGW, NAT, route tables
│   ├── ec2/              # EC2 instances, AMI selection, key pairs
│   ├── alb/              # Application Load Balancer + target groups
│   ├── rds/              # RDS instance, subnet group, parameter group
│   ├── security-groups/  # SGs for ALB, EC2, RDS, and Bastion
│   └── bastion/          # Bastion Host EC2 + SSH access rules
│
└── environments/
    ├── dev/              # Dev environment tfvars
    └── prod/             # Prod environment tfvars
```

---

## 🔐 Security Design

Security is not an afterthought — it is built into every layer:

- **ALB Security Group** — Accepts only ports 80/443 from the internet
- **EC2 Security Group** — Accepts traffic only from the ALB, not directly from the internet
- **RDS Security Group** — Accepts connections only from EC2 instances on the DB port
- **Bastion Security Group** — Accepts SSH (port 22) only from a specific trusted IP/CIDR
- **Private Subnets** — EC2 and RDS have no public IPs; they are unreachable from the internet
- **IAM Roles** — EC2 instances use instance profiles with least-privilege policies

```
Internet ──► ALB (443) ──► EC2 (8080) ──► RDS (3306)
                                ▲
                      Bastion Host (22 only from VPN IP)
```

---

##  How to Deploy

### Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/downloads) v1.x installed
- AWS CLI configured with appropriate IAM permissions
- An S3 bucket and DynamoDB table for remote state (or use `terraform init` to create)

### Step-by-step

```bash
# 1. Clone the repository
git clone https://github.com/Shreyash28M/Terraform-Project.git
cd Terraform-Project

# 2. Initialise Terraform (downloads providers, sets up remote backend)
terraform init

# 3. Review the execution plan — see exactly what will be created
terraform plan -var-file="environments/dev/terraform.tfvars"

# 4. Apply the infrastructure
terraform apply -var-file="environments/dev/terraform.tfvars"

# 5. Destroy when done (avoids unnecessary AWS costs)
terraform destroy -var-file="environments/dev/terraform.tfvars"
```

---

##  Key Variables

| Variable | Description | Default |
|---|---|---|
| `aws_region` | AWS region to deploy into | `us-east-1` |
| `vpc_cidr` | CIDR block for the VPC | `10.0.0.0/16` |
| `instance_type` | EC2 instance type | `t2.micro` |
| `db_instance_class` | RDS instance class | `db.t3.micro` |
| `db_engine` | Database engine | `mysql` |
| `allowed_ssh_cidr` | IP allowed to SSH via Bastion | `<your-ip>/32` |
| `environment` | Deployment environment tag | `dev` |

---

##  Outputs

After `terraform apply`, you'll see:

```
alb_dns_name        = "my-alb-123456.us-east-1.elb.amazonaws.com"
bastion_public_ip   = "54.x.x.x"
rds_endpoint        = "mydb.xxxx.us-east-1.rds.amazonaws.com"
vpc_id              = "vpc-xxxxxxxxxx"
private_subnet_ids  = ["subnet-xxx", "subnet-yyy"]
```

---

##  DevOps Concepts Demonstrated

This project covers skills directly tested in DevOps interviews:

| Concept | Implementation |
|---|---|
| **Infrastructure as Code** | 100% Terraform — no manual AWS Console |
| **Remote State Management** | S3 backend + DynamoDB state locking |
| **Modular Terraform** | Reusable modules for VPC, EC2, RDS, ALB, SG |
| **Multi-tier Architecture** | Web, App, and DB tiers with network isolation |
| **High Availability** | ALB + multiple EC2s + RDS replica across AZs |
| **Security Best Practices** | Least-privilege SGs, private subnets, Bastion SSH |
| **Environment Separation** | Dev / Prod via separate `.tfvars` files |
| **IaC Idempotency** | Safe to `apply` multiple times without side effects |

---

## 🔄 CI/CD Pipeline (Jenkins)

This project includes a **CI/CD pipeline using Jenkins** to automate Terraform-based infrastructure provisioning, ensuring consistent and production-ready deployments.

---

### ⚙️ Pipeline Workflow

The Jenkins pipeline performs the following stages:

1. **Checkout Code** – Pulls latest Terraform code from GitHub
2. **Terraform Init** – Initializes backend and providers
3. **Terraform Validate** – Ensures configuration correctness
4. **Terraform Plan** – Previews infrastructure changes
5. **Terraform Apply** – Deploys infrastructure (with approval gate)

This pipeline automates the complete DevOps workflow:

1. Builds and pushes Docker image to DockerHub (shreyash28m)
2. Provisions AWS infrastructure using Terraform
3. Deploys application to AWS EKS
4. Uses secure credential management
5. Includes manual approval before production deployment


---

###  Sample Jenkinsfile

```groovy
### 📌 Pipeline Code

```groovy
pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "ap-southeast-1"
        CLUSTER_NAME = "EKS_CLOUD"
        DOCKER_IMAGE = "shreyash28m/terraform-project"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Shreyash28M/Terraform-Project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE:latest .
                docker tag $DOCKER_IMAGE:latest $DOCKER_IMAGE:$BUILD_NUMBER
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE:latest
                    docker push $DOCKER_IMAGE:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Terraform Init & Validate') {
            steps {
                withCredentials([aws(credentialsId: 'aws-cred')]) {
                    dir('EKS-TF') {
                        sh 'terraform init'
                        sh 'terraform validate'
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                withCredentials([aws(credentialsId: 'aws-cred')]) {
                    dir('EKS-TF') {
                        sh 'terraform plan -out=tfplan'
                    }
                }
            }
        }

        stage('Manual Approval') {
            steps {
                input message: "Approve Terraform Apply?"
            }
        }

        stage('Terraform Apply') {
            steps {
                withCredentials([aws(credentialsId: 'aws-cred')]) {
                    dir('EKS-TF') {
                        sh 'terraform apply -auto-approve tfplan'
                    }
                }
            }
        }

        stage('Update Kubeconfig') {
            steps {
                withCredentials([aws(credentialsId: 'aws-cred')]) {
                    sh '''
                    aws eks --region $AWS_DEFAULT_REGION \
                    update-kubeconfig --name $CLUSTER_NAME
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                sed -i "s|IMAGE_PLACEHOLDER|$DOCKER_IMAGE:$BUILD_NUMBER|g" deployment.yaml
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl get pods'
                sh 'kubectl get svc'
            }
        }
    }
}
```

```
## 🔐 Credentials Management

Sensitive credentials are securely managed using **Jenkins Credentials Store**, following DevOps security best practices.

* **AWS Credentials (`aws-cred`)**

  * Used for Terraform provisioning and EKS cluster access
  * Configured using IAM Access Key and Secret Key

* **DockerHub Credentials (`docker-cred`)**

  * Used for authenticating and pushing Docker images
  * Stored as Username and Password

> ⚠️ **Security Note:** No credentials are hardcoded in the pipeline. All secrets are securely injected at runtime using Jenkins credential binding.

---

###  Why CI/CD Matters in This Project

* Eliminates manual infrastructure deployment
* Ensures consistent environments (Dev / Prod)
* Reduces human errors
* Demonstrates real-world DevOps practices

---

### 🔐 Prerequisites

* Jenkins installed and configured
* Terraform installed on Jenkins server
* AWS credentials configured securely (IAM role or access keys)
* Remote backend (S3 + DynamoDB) already set up


---
##  Tech Stack

```
Infrastructure:   Terraform · AWS
Compute:          EC2 (Amazon Linux 2) · Auto Scaling Group
Networking:       VPC · Subnets · IGW · NAT · Route Tables
Load Balancing:   Application Load Balancer (ALB)
Database:         Amazon RDS (MySQL/PostgreSQL)
Security:         Security Groups · IAM Roles · Key Pairs
State:            S3 (remote backend) · DynamoDB (state lock)
Access:           Bastion Host · SSH Key Authentication
```

---

## 📈 Why This Project Matters

Modern organisations deploy infrastructure **at scale**, and manual provisioning doesn't scale. This project proves the ability to:

- Write clean, modular, version-controlled infrastructure code
- Apply security-first network design on AWS
- Think in systems — not just individual resources
- Reduce human error through automation
- Enable consistent deployments across environments

These are exactly the skills companies look for in **DevOps Engineers**, **Cloud Engineers**, and **Site Reliability Engineers (SREs)**.

---

## 🙋 Author

**Shreyash M**
🔗 [GitHub](https://github.com/Shreyash28M)
📧 *Available for DevOps / Cloud Engineering roles*

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

<div align="center">
⭐ If this project helped you, please give it a star — it helps others find it too!
</div>
