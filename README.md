# AWS Terraform Management Setup Guide
## Introduction

This guide will walk you through the process of setting up a terraform-management OU (Organizational Unit) within AWS Organizations. This OU will be used to manage other OUs and their associated resources using Terraform.
## Prerequisites

- Root AWS account with AWS Organizations set up.
- AWS CLI installed and configured.
- GitHub account (for GitHub Actions).

## Step-by-step Guide
1. Create the `terraform-management` OU

    Using AWS Management Console or AWS CLI, create a new OU called `terraform-management`.

2. Setup the AWS Account for Terraform Management

    - Create a new AWS account within the `terraform-management` OU.
3. Sign in to the `terraform-management` AWS account.

    #### Cross-Account Role Switching:

    - Log in to the AWS Management Console of the master account or another AWS account where you have permissions to assume a role in the Terraform-management account.
    - Navigate to the "Services" menu and select "IAM".
    - In the IAM dashboard, choose "Switch Role" on the upper right.
    - Provide the details of the Terraform-management account (Account ID) and the name of the IAM role (Commonly `OrganizationAccountAccessRole`) you want to assume.
    - Click "Switch Role". You should now be logged into the Terraform-management account with the permissions of the assumed role.

4. Setup an IAM user or role with administrative permissions for Terraform operations.
    - Securely store the access and secret keys for later use in GitHub Actions.

5. Configure Terraform Backend: S3 and DynamoDB
    - S3 Bucket:
        - Create an S3 bucket to store Terraform state.
        - Enable versioning on this bucket for state history.
        - Encrypt the bucket using AWS KMS.

    - DynamoDB Table:
        - Create a DynamoDB table for state locking.
        - Use a primary key named LockID.

    - IAM Permissions:
        - Grant your Terraform IAM user or role permissions to read/- - write to the S3 bucket and DynamoDB table.

6. Setup GitHub Actions for CI/CD
    - Secrets: In your GitHub repository settings, add the following secrets:
        - AWS_ACCESS_KEY_ID: The access key ID of the Terraform IAM user or role.
        - AWS_SECRET_ACCESS_KEY: The secret access key of the Terraform IAM user or role.

    - GitHub Actions Workflow:
        - Create a new .github/workflows/terraform.yml file in your - - repository.
        - Define the workflow steps for Terraform init, plan, and apply. Use the AWS credentials stored as GitHub secrets.

    - See template: [Github Workflow Terraform Template](github_workflow_terraform_template.yml)

7. Implement Terraform Code for Managing OUs

In your Terraform code:

- Define the AWS provider and backend configuration - [Terraform Template](terraform_template.tf)

- Define Terraform code to manage OUs, policies, and other AWS resources as needed.

After following the steps above, your terraform-management OU and the associated AWS account should be set up to manage other OUs and AWS resources through Terraform with CI/CD pipelines in GitHub Actions.

Always ensure to test the setup and configurations in a non-production environment before deploying to production.