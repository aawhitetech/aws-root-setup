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

4. Create the Terraform Role in the Terraform-management Account:

    - Log into the AWS Management Console for your Terraform-management account.
    - Navigate to IAM -> Roles -> Create Role.
    - For trusted entities, choose "Another AWS account" and provide the Account ID of the account where GitHub Actions will be executed. This could be the Terraform-management account itself or another account, depending on where you're triggering the GitHub Actions from.
    - Attach necessary permissions to the role. This will typically be policies that allow Terraform to manage desired AWS resources.
    - Name the role, for example, TerraformRole, and create the role.

5. Create the Terraform User in the terraform-management Account:

    -    In the AWS Management Console for the Terraform-management account, go to IAM -> Users -> Add user.
    - Provide a username like TerraformUser.
    - Choose "Programmatic access" to get access key ID and secret access key.
    - On the permissions page, choose "Attach existing policies directly". Click on "Create policy" and use the following policy document:

```
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "sts:AssumeRole",
                "Resource": "arn:aws:iam::<Terraform-management-account-id>:role/TerraformRole"
            }
        ]
    }
```

- Replace <Terraform-management-account-id> with the actual account ID of the Terraform-management account.

- Review the policy, give it a name like AssumeTerraformRolePolicy, and create the policy.
Go back to the user creation wizard, refresh the policies list, and attach the AssumeTerraformRolePolicy to the TerraformUser.
Complete the user creation process.

6. Retrieve API Key/Secret:

    After creating the user with programmatic access, AWS will show you the Access Key ID and Secret Access Key:

    Make sure to store the Access Key ID and Secret Access Key securely. AWS will not show the secret access key again.
    You'll use these credentials to configure Terraform (or any other AWS SDK/tool) to perform AWS actions as TerraformUser, and from there, you can assume the TerraformRole.

    Remember to always handle AWS credentials with care. Avoid hardcoding them in any scripts or applications. Use environment variables, AWS configuration files, or secrets management tools to store and retrieve them securely.

7. Assuming the Role:

    When you set up GitHub Actions, you'll use the AWS sts:AssumeRole action to assume this Terraform role. This will return temporary credentials that can be used by Terraform.

8. GitHub Actions Setup:

    In your GitHub repository settings, add the following secrets:
    - AWS_ACCOUNT_ID: The Account ID of your Terraform-management account.
    - AWS_ROLE_TO_ASSUME: The name of the role you created in step 1 (e.g., TerraformGitHubActionsRole).
    - AWS_ACCESS_KEY_ID: The access key ID of an IAM user in the Terraform-management account or another AWS account that has permissions to assume the role created in step 1.
    - AWS_SECRET_ACCESS_KEY: The corresponding secret access key.
    - AWS_REGION: The corresponding aws region.

9. Configure Terraform Backend: S3 and DynamoDB
    - S3 Bucket:
        - Create an S3 bucket to store Terraform state.
        - Enable versioning on this bucket for state history.
        - Encrypt the bucket using AWS KMS.

    - DynamoDB Table:
        - Create a DynamoDB table for state locking.
        - Use a primary key named LockID.

    - IAM Permissions:
        - Grant your Terraform IAM user or role permissions to read/- - write to the S3 bucket and DynamoDB table.

10. Setup GitHub Actions for CI/CD
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