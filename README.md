# Automated Infrastructure Deployment with Terraform, Jenkins, and LocalStack

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Terraform](https://img.shields.io/badge/Terraform-1.5+-623CE4?logo=terraform)](https://www.terraform.io/)
[![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?logo=jenkins)](https://www.jenkins.io/)
[![LocalStack](https://img.shields.io/badge/LocalStack-AWS%20Simulation-000000?logo=localstack)](https://localstack.cloud/)

An end-to-end Infrastructure as Code (IaC) CI/CD pipeline built to automate cloud infrastructure provisioning on AWS (simulated using **LocalStack**) using **Terraform** and orchestrated through **Jenkins**.

---

## 📐 Architecture Overview

The pipeline automates the entire process from code commit to infrastructure provisioning and deployment monitoring:

1. **GitHub Repository**: Stores all Terraform configuration files and the `Jenkinsfile`.
2. **Jenkins CI/CD Server**: Listens for triggers, checks out the code, handles workspace management, requests manual approval for infrastructure changes, and executes Terraform commands.
3. **LocalStack (AWS Simulation)**: emulates AWS APIs locally, providing isolated testing for VPCs, Subnets, EC2 instances, and S3 buckets without incurring actual cloud costs.

---

## 📁 Repository Structure

```text
├── VPC.tf              # Defines Network Infrastructure (VPC, Subnets, Gateways)
├── Variables.tf        # Input variable definitions
├── Provider.tf         # AWS Provider setup configured for LocalStack endpoints
├── dev.tfvars          # Environment-specific configuration for Development
├── stg.tfvars          # Environment-specific configuration for Staging
├── prod.tfvars         # Environment-specific configuration for Production
└── Jenkinsfile         # Jenkins declarative pipeline script
```

---

## 🔄 CI/CD Pipeline Stages

The `Jenkinsfile` orchestrates the execution into clear, sequential stages:

```
[ Checkout ] -> [ Terraform Init ] -> [ Workspace Select ] -> [ Plan & Approve ] -> [ Apply ] -> [ Post Notifications ]
```

1. **Checkout**: Clones the latest version of the repository from GitHub.
2. **Terraform Init**: Initializes the working directory and installs required provider plugins (`terraform init`).
3. **Workspace Select**: Dynamically switches or creates the designated Terraform workspace (e.g., `dev`, `stg`, `prod`).
4. **Plan**:
   * Generates an execution plan (`terraform plan`).
   * **Manual Gate**: Prompts the user with an **Approve / Abort** decision before modifying resources.
5. **Apply**: Executes the approved changes against the target environment (`terraform apply`).
6. **Post Actions**: Triggered automatically after completion. Sends email notifications containing execution status (**Success** or **Failure**).

---

## ☁️ Simulated Infrastructure (LocalStack)

The stack provisions and manages simulated AWS components locally:

* **VPC & Subnets**: Custom networking setup with public and private subnets.
* **EC2 Instances**: Virtual compute instances.
* **S3 Buckets**: Object storage for application assets or state files.

---

## 🚀 Getting Started

### Prerequisites

Ensure you have the following installed on your host machine:

* [Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/)
* [Terraform CLI](https://developer.hashicorp.com/terraform/downloads)
* [LocalStack CLI](https://docs.localstack.cloud/getting-started/installation/)
* Running **Jenkins** instance with Docker and Terraform plugins configured.

### Local Setup & Testing

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/terraform-jenkins-localstack.git
   cd terraform-jenkins-localstack
   ```

2. **Start LocalStack:**
   ```bash
   localstack start -d
   ```

3. **Initialize Terraform locally (Optional):**
   ```bash
   terraform init
   terraform workspace new dev
   terraform plan -var-file="dev.tfvars"
   ```

4. **Configure Jenkins Pipeline:**
   * Create a new Pipeline Job in your Jenkins Dashboard.
   * Set the definition to **Pipeline script from SCM**.
   * Point the repository URL to this GitHub repo and set the script path to `Jenkinsfile`.

---

## 📧 Email Notifications Configuration

To enable failure and success email alerts in the `Post` stage:
1. Configure SMTP credentials under **Manage Jenkins > System > Extended E-mail Notification**.
2. Pass the recipient email variables within the `Jenkinsfile` environment block.

---

