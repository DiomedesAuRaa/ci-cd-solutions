Here's a README template for your GitHub repository that explains how to set up and run the Terraform workflow with the GitHub Actions automation:

---

# Terraform Automation with GitHub Actions

This repository automates the process of running Terraform plans and applying them using GitHub Actions workflows. It uses a GitHub Actions CI/CD pipeline to deploy infrastructure on AWS, with approval steps integrated via GitHub Environments for manual deployment approval.

## Overview

This setup runs Terraform commands automatically when code is pushed to the `main` branch, with manual approval required before deploying changes to the production environment.

### Apply Workflow Steps:

1. **Checkout the repository** - The latest code is checked out.
2. **Clone external Terraform repository** - A separate repository with Terraform configuration is cloned.
3. **Create Terraform Variables** - `terraform.tfvars` file is created with AWS credentials (using GitHub secrets).
4. **Set up Terraform** - The required Terraform version is installed.
5. **Configure AWS credentials** - AWS credentials are configured using GitHub secrets.
6. **Initialize Terraform** - Terraform is initialized.
7. **Terraform Plan** - Terraform plan is created and saved to a `.tfplan` file.
8. **Manual Approval** - The deployment waits for approval if the `production` environment is configured with manual approval rules.
9. **Terraform Apply** - After approval, Terraform apply is executed with the generated plan.

## Prerequisites

- AWS account with proper IAM permissions for Terraform to manage infrastructure.
- GitHub repository with Secrets configured for AWS access (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, etc.).
- Terraform installed (if running locally for testing purposes).
- GitHub Environments configured for manual approval (e.g., `production` environment).

## GitHub Secrets

Before running the workflow, ensure that the following GitHub secrets are set in your repository:

- `AWS_ACCESS_KEY_ID` – AWS access key for authentication.
- `AWS_SECRET_ACCESS_KEY` – AWS secret key for authentication.
- `AWS_ACCOUNT_ID` – AWS account ID for assuming IAM roles.
- `AWS_ROLE_NAME` – Name of the IAM role to assume for Terraform operations.

## GitHub Environments (optional and not setup currently)

This workflow integrates with GitHub Environments to provide manual approval for the `Terraform Apply` step:

1. Go to **Settings > Environments** in your GitHub repository.
2. Create a new environment (e.g., `production`).
3. Add **required reviewers** (e.g., team members) who need to approve the deployment.
4. Add any necessary secrets to the environment if required.

## Workflow Configuration

The workflow is configured to run on pushes to the `main` branch and can also be triggered manually through **workflow_dispatch**.

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

The terraform-destroy.yml workflow is designed to destroy resources provisioned by Terraform, ensuring safe cleanup of your infrastructure.

    Triggered by: Manual trigger (workflow_dispatch).
    Steps:
        Checkout the repository.
        Clone the external Terraform repository.
        Create necessary terraform.tfvars for AWS credentials and region.
        Setup Terraform with the latest version.
        Configure AWS credentials via GitHub Secrets.
        Initialize Terraform.
        Run terraform destroy to tear down the infrastructure.

## Troubleshooting

- **State Locking Issues**: If Terraform cannot acquire a state lock, ensure that no other Terraform operations are running that are locking the state. You can disable the state lock by using the `-lock=false` flag with the `terraform apply` command. This is currently enabled and is not best practice. 
  
- **Manual Approval Stuck**: If the workflow is stuck on the approval step, ensure that the environment is configured correctly and that reviewers are listed for manual approval in your GitHub environment settings.

## Contributing

Feel free to fork and contribute to this repository. If you encounter issues or have suggestions for improvement, open an issue or submit a pull request.

---