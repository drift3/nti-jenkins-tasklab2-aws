Automated Infrastructure Deployment with Terraform, Jenkins, and Floci

An end-to-end Infrastructure as Code (IaC) CI/CD pipeline built to automate cloud infrastructure provisioning on AWS (simulated locally using Floci) using Terraform and orchestrated through Jenkins.

📐 Architecture Overview

The pipeline automates the entire process from code commit to infrastructure provisioning and deployment monitoring:
1. GitHub Repository: Stores all Terraform configuration files and the Jenkinsfile.
2. Jenkins CI/CD Server: Running inside a Docker container on RHEL, listens for triggers, checks out the code, handles workspace management, requests manual approval, and executes Terraform CLI commands.
3. Floci (AWS Simulation): Emulates AWS APIs locally, providing isolated testing for VPCs, Subnets, and Gateways without incurring actual cloud costs.

📁 Repository Structure

├── vpc.tf              # Defines Network Infrastructure (VPC, Subnets, Gateways)
├── variables.tf        # Input variable definitions
├── provider.tf         # AWS Provider setup configured for Floci endpoints (via host bridge IP 172.17.0.1)
├── dev                 # Environment-specific configuration file for Development
└── Jenkinsfile         # Jenkins declarative pipeline script

🔄 CI/CD Pipeline Stages

The Jenkinsfile orchestrates the execution into clear, sequential stages:
[ Checkout ] -> [ Terraform Init ] -> [ Workspace Select ] -> [ Plan & Approve ] -> [ Apply ] -> [ Post Actions ]

1. Checkout: Clones the latest version of the repository from GitHub.
2. Terraform Init: Initializes the working directory and installs required AWS provider plugins (`terraform init`).
3. Workspace Select: Dynamically switches or creates the designated Terraform workspace (`dev`).
4. Terraform Plan:
  - Generates an execution plan (`terraform plan`).
  - Manual Gate: Prompts the user with an Approve / Abort decision before modifying infrastructure.
5. Apply: Executes the approved changes against the target environment (`terraform apply`).
6. Post Actions: Triggered automatically after completion.

☁️ Simulated Infrastructure (Floci)

The stack provisions and manages simulated AWS components locally:
- VPC & Subnets: Custom networking setup (`dev-vpc` with CIDR `10.0.0.0/16`).
- Internet Gateways: Routing traffic from the custom VPC to the local environment.

🚀 Getting Started

Prerequisites
Ensure you have the following installed on your host machine:
- Red Hat Enterprise Linux (RHEL) host
- Docker & Docker Compose
- Terraform CLI (manually configured or installed inside the Jenkins container)
- Running Jenkins container instance

Local Setup & Testing
1. Clone the repository:
   ```bash
   git clone https://github.com/drift3/nti-jenkins-tasklab2-aws.git
   cd nti-jenkins-tasklab2-aws
   ```

2. Start Floci Emulator locally via Docker Compose.

3. Configure Jenkins Pipeline:
   - Create a new Pipeline Job in your Jenkins Dashboard.
   - Set the definition to Pipeline script from SCM.
   - Point the repository URL to your GitHub repo and set the script path to `Jenkinsfile`.
