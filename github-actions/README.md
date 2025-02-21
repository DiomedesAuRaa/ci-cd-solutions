# GitHub Actions Workflow for Terraform Deployment

This directory contains a GitHub Actions workflow (`deploy-terraform.yml`) that automates the deployment of Terraform code across multiple environments: **dev**, **stage**, and **prod**.

## Workflow Overview

1. **Validate**: Runs `terraform validate` to check the syntax and configuration of the Terraform code.
2. **Plan (Dev)**: Runs `terraform plan` for the `dev` environment using the `dev.tfvars` file.
3. **Apply (Dev)**: Runs `terraform apply` for the `dev` environment if the plan succeeds.
4. **Plan (Stage)**: Runs `terraform plan` for the `stage` environment using the `stage.tfvars` file. This step only runs if the `dev` apply succeeds.
5. **Apply (Stage)**: Runs `terraform apply` for the `stage` environment if the plan succeeds.
6. **Plan (Prod)**: Runs `terraform plan` for the `prod` environment using the `prod.tfvars` file. This step only runs if the `stage` apply succeeds.
7. **Approve (Prod)**: Waits for manual approval before applying changes to the `prod` environment.
8. **Apply (Prod)**: Runs `terraform apply` for the `prod` environment if manual approval is granted.

## How to Use

1. **Set Up Terraform Code**:
   - Ensure your Terraform code is in a separate repository (e.g., `your-username/your-terraform-re
