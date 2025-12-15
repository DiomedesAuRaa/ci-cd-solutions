# Multi-Cloud Terraform Automation with CI/CD Pipelines

This repository provides reusable CI/CD pipeline configurations for automating Terraform infrastructure deployments across multiple cloud providers (AWS, Azure, GCP) using various CI/CD platforms.

## 🌐 Supported Cloud Providers

| Provider | Services | Workflows |
|----------|----------|-----------|
| AWS | EC2, EKS, RDS | Apply & Destroy |
| Azure | AKS | Apply & Destroy |
| GCP | GKE | Apply & Destroy |

## 🔧 Supported CI/CD Platforms

- **GitHub Actions** - Primary, fully implemented
- **Azure DevOps** - Pipeline template
- **GitLab CI** - Pipeline template
- **CircleCI** - Pipeline template
- **Jenkins** - Jenkinsfile template

## Overview

This setup runs Terraform commands automatically when triggered, with manual approval steps integrated for production deployments.

### Apply Workflow Steps

1. **Checkout the repository** - The latest code is checked out.
2. **Clone external Terraform repository** - A separate repository with Terraform configuration is cloned.
3. **Create Terraform Variables** - `terraform.tfvars` file is created with cloud credentials (using CI/CD secrets).
4. **Set up Terraform** - The required Terraform version is installed.
5. **Configure Cloud Credentials** - Cloud provider credentials are configured using secrets.
6. **Initialize Terraform** - Terraform is initialized.
7. **Validate Terraform** - Configuration is validated for syntax errors.
8. **Terraform Plan** - Terraform plan is created and saved to a `.tfplan` file.
9. **Manual Approval** - The deployment waits for approval if configured.
10. **Terraform Apply** - After approval, Terraform apply is executed with the generated plan.

## Prerequisites

- Cloud provider account with proper IAM/RBAC permissions for Terraform to manage infrastructure.
- CI/CD platform with secrets configured for cloud access.
- Terraform installed (if running locally for testing purposes).
- GitHub Environments configured for manual approval (optional).

## GitHub Secrets

Before running the workflows, ensure the following secrets are set in your repository:

### AWS Secrets
| Secret | Description |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | AWS access key for authentication |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key for authentication |
| `AWS_ACCOUNT_ID` | AWS account ID for assuming IAM roles |
| `AWS_ROLE_NAME` | Name of the IAM role to assume |

### Azure Secrets
| Secret | Description |
|--------|-------------|
| `ARM_CLIENT_ID` | Azure Service Principal client ID |
| `ARM_CLIENT_SECRET` | Azure Service Principal client secret |
| `ARM_SUBSCRIPTION_ID` | Azure subscription ID |
| `ARM_TENANT_ID` | Azure tenant ID |

### GCP Secrets
| Secret | Description |
|--------|-------------|
| `GCP_CREDENTIALS` | GCP service account JSON credentials |
| `GCP_PROJECT_ID` | GCP project ID |
| `GCP_REGION` | GCP region (optional, defaults to us-central1) |

## GitHub Environments (Optional)

This workflow integrates with GitHub Environments to provide manual approval for the `Terraform Apply` step:

1. Go to **Settings > Environments** in your GitHub repository.
2. Create a new environment (e.g., `production`).
3. Add **required reviewers** (e.g., team members) who need to approve the deployment.
4. Add any necessary secrets to the environment if required.

## Example Workflow Configuration

The workflows are configured to run manually through **workflow_dispatch** and can be extended to trigger on pushes to the `main` branch.

```yaml
name: 'Apply External Terraform Configuration'

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

concurrency:
  group: terraform-${{ github.ref }}
  cancel-in-progress: true

jobs:
  terraform:
    name: 'Terraform Apply'
    runs-on: ubuntu-latest
    timeout-minutes: 30
    environment: production  # Requires manual approval for the `production` environment
    steps:
      - name: Checkout Current Repository
        uses: actions/checkout@v4

      - name: Clone External Terraform Repository
        run: |
          git clone https://github.com/DiomedesAuRaa/aws-terraform-stack.git

      - name: List Cloned Directory
        run: ls -la aws-terraform-stack/eks

      - name: Create terraform.tfvars
        working-directory: aws-terraform-stack/eks
        run: |
          cat > terraform.tfvars <<EOF
          aws_access_key = "${{ secrets.AWS_ACCESS_KEY_ID }}"
          aws_secret_key = "${{ secrets.AWS_SECRET_ACCESS_KEY }}"
          region         = "us-east-1"
          EOF
          
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: latest

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Initialize Terraform
        working-directory: aws-terraform-stack/eks
        run: terraform init

      - name: Terraform Plan with Retry
        working-directory: aws-terraform-stack/eks
        run: |
          retries=5
          delay=10
          for i in $(seq 1 $retries); do
            terraform plan -out=tfplan && break
            echo "Terraform plan failed. Retrying ($i/$retries)..."
            sleep $delay
          done

      - name: Show Terraform Plan Output (Debugging)
        if: failure()  # Only runs if the previous step fails
        run: |
          echo "Terraform plan failed, printing debug information..."
          terraform plan

      - name: Terraform Apply
        working-directory: aws-terraform-stack/eks
        run: terraform apply -lock=false tfplan
        if: success()  # Only runs if terraform plan was successful
```

## Destroy Terraform Configuration

The destroy workflows are designed to safely tear down infrastructure provisioned by Terraform.

- **Triggered by**: Manual trigger (`workflow_dispatch`)
- **Steps**:
  1. Checkout the repository
  2. Clone the external Terraform repository
  3. Create necessary `terraform.tfvars` for credentials
  4. Setup Terraform with the latest version
  5. Configure cloud credentials via secrets
  6. Initialize Terraform
  7. Run `terraform destroy` to tear down the infrastructure

## Repository Structure

```
ci-cd-solutions/
├── .github/workflows/       # GitHub Actions workflows
│   ├── terraform-apply-*.yml
│   └── terraform-destroy-*.yml
├── azure-devops/            # Azure DevOps pipeline
├── circle-ci/               # CircleCI configuration
├── gitlab/                  # GitLab CI configuration
└── jenkins/                 # Jenkins pipeline
```

## Troubleshooting

### State Locking Issues
If Terraform cannot acquire a state lock, ensure that no other Terraform operations are running that are locking the state. You can disable the state lock by using the `-lock=false` flag with the `terraform apply` command.

> ⚠️ **Warning**: Using `-lock=false` is not recommended for production environments as it can lead to state corruption.

### Manual Approval Stuck
If the workflow is stuck on the approval step, ensure that the environment is configured correctly and that reviewers are listed for manual approval in your GitHub environment settings.

### Authentication Issues
- **AWS**: Verify IAM role trust policy allows the GitHub Actions OIDC provider
- **Azure**: Ensure Service Principal has required permissions
- **GCP**: Verify service account has necessary IAM roles

## Contributing

Feel free to fork and contribute to this repository. If you encounter issues or have suggestions for improvement, open an issue or submit a pull request.

## License

MIT License - see LICENSE file for details.