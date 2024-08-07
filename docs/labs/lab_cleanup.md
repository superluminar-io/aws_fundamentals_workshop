# Clean-Up: Deleting AWS Resources

## Overview

After completing the labs, it's important to clean up the resources you have created to avoid incurring any unnecessary costs. This clean-up guide will help you delete all the AWS resources created during Labs 3, 4, and 5.

## Steps to Clean Up

### 1. Delete the Stack Using CDK

The simplest way to delete all resources created by your CDK stack is to destroy the stack itself. This will automatically clean up all resources managed by the stack.

1. **Open Your Terminal**

   Navigate to the root directory of your CDK project.

2. **Run the CDK Destroy Command**

   Run the following command to destroy the stack:

   ```bash
   cdk destroy --profile PROFILE_NAME
   ```

   Confirm the deletion when prompted. This will delete all resources created by the stack, including the VPC, subnets, security groups, EC2 instance, S3 bucket, and RDS instance.

### 2. Manual Verification (Optional)

To ensure all resources are properly deleted, you can manually verify the deletion using the AWS Management Console and CLI.

**AWS Management Console**:

- **EC2**: Navigate to the EC2 dashboard and verify that the instance is no longer listed.
- **S3**: Navigate to the S3 dashboard and verify that the bucket is deleted.
- **RDS**: Navigate to the RDS dashboard and verify that the database instance is deleted.
- **VPC**: Navigate to the VPC dashboard and verify that the VPC and associated resources (subnets, route tables, security groups) are deleted.

**AWS CLI**:

Run the following commands to verify the deletion of resources:

- **EC2 Instance**:

  ```bash
  aws ec2 describe-instances --filters "Name=tag:Name,Values=MyEC2Instance" --query "Reservations[*].Instances[*].InstanceId" --output table --profile PROFILE_NAME
  ```

  Ensure no instances are listed.

- **S3 Bucket**:

  ```bash
  aws s3 ls --profile PROFILE_NAME
  ```

  Ensure the bucket is not listed.

- **RDS Instance**:

  ```bash
  aws rds describe-db-instances --query "DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus]" --output table --profile PROFILE_NAME
  ```

  Ensure the instance is not listed.

- **VPC**:
  ```bash
  aws ec2 describe-vpcs --query "Vpcs[*].[VpcId,State,Tags]" --output table --profile PROFILE_NAME
  ```
  Ensure the VPC is not listed.

### 3. Remove Generated Secrets (Optional)

If you have generated secrets in AWS Secrets Manager, ensure they are also deleted.

- **AWS Management Console**:

  - Navigate to the Secrets Manager dashboard.
  - Select and delete any secrets created during the labs.

- **AWS CLI**:
  ```bash
  aws secretsmanager list-secrets --query "SecretList[*].[Name,ARN]" --output table --profile PROFILE_NAME
  ```
  Identify and delete the secrets:
  ```bash
  aws secretsmanager delete-secret --secret-id SECRET_ARN --force-delete-without-recovery --profile PROFILE_NAME
  ```

### 4. Check for Remaining Resources (Recommended)

After completing the above steps, it's a good practice to double-check for any remaining resources:

- Review the AWS Cost Explorer to ensure there are no unexpected charges.
- Check other AWS services not mentioned above (e.g., Lambda functions, CloudWatch logs) to ensure complete cleanup.

## Summary

By following these steps, you will ensure that all resources created during the labs are properly cleaned up, avoiding unnecessary costs. The `cdk destroy` command is the primary method for clean-up, but manual verification steps are provided for completeness. Always double-check your AWS account to confirm all resources have been removed.
