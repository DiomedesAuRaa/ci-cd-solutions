# Terraform GitHub Workflows for Multi-Cloud Environments

This repository contains GitHub Actions workflows to apply and destroy infrastructure for **AWS**, **Azure**, and **GCP** environments using Terraform. Below are the workflow files included for each cloud provider:

## Workflow Files

### AWS Workflows

#### RDS Environment
- **`terraform-apply-rds.yml`**: Creates or updates AWS RDS infrastructure.
- **`terraform-destroy-rds.yml`**: Destroys AWS RDS infrastructure.

#### EC2 Environment
- **`terraform-apply-ec2.yml`**: Creates or updates AWS EC2 infrastructure.
- **`terraform-destroy-ec2.yml`**: Destroys AWS EC2 infrastructure.

#### EKS Environment
- **`terraform-apply-eks.yml`**: Creates or updates AWS EKS (Kubernetes) infrastructure.
- **`terraform-destroy-eks.yml`**: Destroys AWS EKS infrastructure.

### Azure Workflows

#### AKS Environment
- **`terraform-apply-aks.yml`**: Creates or updates Azure AKS (Kubernetes) infrastructure.
- **`terraform-destroy-aks.yml`**: Destroys Azure AKS infrastructure.

### GCP Workflows

#### GKE Environment
- **`terraform-apply-gke.yml`**: Creates or updates GCP GKE (Kubernetes) infrastructure.
- **`terraform-destroy-gke.yml`**: Destroys GCP GKE infrastructure.

## Required Secrets

### AWS Secrets
- `AWS_ACCESS_KEY_ID` – AWS access key for authentication
- `AWS_SECRET_ACCESS_KEY` – AWS secret key for authentication
- `AWS_ACCOUNT_ID` – AWS account ID for IAM role assumption
- `AWS_ROLE_NAME` – Name of the IAM role to assume

### Azure Secrets
- `ARM_CLIENT_ID` – Azure Service Principal client ID
- `ARM_CLIENT_SECRET` – Azure Service Principal client secret
- `ARM_SUBSCRIPTION_ID` – Azure subscription ID
- `ARM_TENANT_ID` – Azure tenant ID

### GCP Secrets
- `GCP_CREDENTIALS` – GCP service account JSON credentials
- `GCP_PROJECT_ID` – GCP project ID
- `GCP_REGION` – GCP region (optional, defaults to us-central1)
