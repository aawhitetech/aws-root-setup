# AWS Terraform Management Setup Guide

## Introduction
This guide describes how to set up a `terraform-management` Organizational Unit (OU) within AWS Organizations to manage other OUs and resources using Terraform.

## Prerequisites

- Root AWS account with AWS Organizations set up.
- AWS CLI installed and configured.
- GitHub account (for GitHub Actions).

## Step-by-step Guide

### 1. **Set up the `terraform-management` OU**

Using AWS Management Console or AWS CLI, create a new OU named `terraform-management`.

### 2. **Setup the AWS Account for Terraform Management**

Create a new AWS account within the `terraform-management` OU.

### 3. **Sign into the `terraform-management` AWS Account**

**Cross-Account Role Switching**:
  - Log into the AWS Management Console of your master account.
  - Select IAM from the "Services" menu.
  - Choose "Switch Role" on the IAM dashboard.
  - Input the Terraform-management account details (Account ID) and the IAM role you want to assume (typically `OrganizationAccountAccessRole`).
  - Click "Switch Role".

### 4. **Create the Terraform Role**

- In the AWS Management Console for `terraform-management`, navigate to IAM > Roles > Create Role.
- Choose "Another AWS account" for trusted entities and provide the Account ID of where GitHub Actions will run.
- Attach permissions allowing Terraform to manage AWS resources.
- Name the role (e.g., `TerraformRole`).

### 5. **Set Up OIDC Provider for GitHub Actions**

1. **Create OIDC Provider**:
   - In IAM, choose `Identity providers`.
   - Select `Add provider`.
   - Set `Provider Type` to `OpenID Connect`.
   - Use `https://token.actions.githubusercontent.com` for `Provider URL`.
   - For `Audience`, enter `sts.amazonaws.com`.
   
2. **Edit Trust Relationship**:
   - Adjust the trust relationship JSON for your OIDC provider, GitHub organization, repo, and branch.
  
### 6. **Configure Terraform Backend with S3 and DynamoDB**

- **S3 Bucket**:
  - Create a bucket for Terraform state.
  - Enable versioning.
  - Apply AWS KMS encryption.
  
- **DynamoDB Table**:
  - Create a table for state locking with `LockID` as the primary key.
  
- **IAM Permissions**:
  - Grant the Terraform IAM user or role permissions to the S3 bucket and DynamoDB table.

### 7. **Setup GitHub Actions for CI/CD**

- **GitHub Secrets**:
   - In your GitHub repo settings, add the required secrets.
   
- **GitHub Actions Workflow**:
  - Create a `.github/workflows/terraform.yml` in your repo.
  - Use the AWS credentials stored as GitHub secrets.
  
  > See template: [Github Workflow Terraform Template](github_workflow_terraform_template.yml)

### 8. **Implement Terraform Code for Managing OUs**

- Use the AWS provider and backend configurations.
- Create Terraform code for OUs, policies, and other resources.

> See template: [Terraform Template](terraform_template.tf)

---

After following this guide, your `terraform-management` OU and the associated AWS account will be ready to manage other OUs and resources using Terraform. Before deploying to production, test all configurations in a non-production environment.