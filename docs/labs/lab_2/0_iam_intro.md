# Lab 2: Identity and Access Management (IAM)

## Introduction to IAM

AWS Identity and Access Management (IAM) is a web service that helps you securely control access to AWS resources. With IAM, you can manage access to AWS services and resources securely by specifying who (identity) or what (role) can access specific resources and how they can access them.

IAM allows you to:

- **Manage IAM users and their access**: Create and manage IAM users and assign them individual security credentials, such as access keys and passwords.
- **Manage IAM groups**: Organize IAM users into groups for easier permission management.
- **Manage IAM roles and their permissions**: Roles are similar to users but are intended to be assumable by anyone or anything that needs access to your resources.
- **Manage federated users and their permissions**: Grant AWS resources access to users managed outside of AWS in your corporate directory.
- **Apply multi-factor authentication (MFA)**: Add an extra layer of protection on top of user names and passwords.

**IAM Roles**

An IAM role is an AWS identity with permission policies that determine what the identity can and cannot do in AWS. Unlike an IAM user, you don't need to create a password or access keys for a role. Instead, IAM roles are meant to be assumable by anyone or anything (such as EC2 instances) that needs them. This makes roles ideal for granting temporary access to your AWS resources.

## Example of an IAM Role

Let's consider a scenario where an EC2 instance needs to read objects from an S3 bucket. Here's how you can set up an IAM role for this purpose:

1. **Create an IAM Role for EC2**

   Define a trust policy that allows EC2 instances to assume this role:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "ec2.amazonaws.com"
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   ```

   This trust policy specifies that the EC2 service can assume the role.

2. **Attach a Policy to the Role**

   Attach a policy to the role that grants permission to read objects from a specific S3 bucket:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::example-bucket/*"
       }
     ]
   }
   ```

   This policy grants read-only access to all objects within the `example-bucket` S3 bucket.

3. **Assign the Role to an EC2 Instance**

   When you launch an EC2 instance, you can assign this role to it. The instance will then inherit the permissions granted by the role, allowing it to read objects from the specified S3 bucket without requiring access keys.

## Complex IAM Policy Example

Let's create a more complex IAM policy that includes explicit denies, conditions, and allows. This policy will:

1. Allow read-only access to all S3 buckets.
2. Allow full access to a specific S3 bucket (`example-bucket`).
3. Deny access to delete objects in another specific S3 bucket (`restricted-bucket`), even if other policies allow it.

Here's the policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "arn:aws:s3:::*"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": ["arn:aws:s3:::*"]
    },
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::example-bucket/*"
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::restricted-bucket/*"
    }
  ]
}
```

## Explanation of the Policy

1. **General Read-Only Access to All S3 Buckets**:

   ```json
   {
     "Effect": "Allow",
     "Action": ["s3:GetObject", "s3:ListBucket"],
     "Resource": ["arn:aws:s3:::*"]
   }
   ```

   This statement allows the user to list and read objects from any S3 bucket.

2. **Full Access to Specific S3 Bucket**:

   ```json
   {
     "Effect": "Allow",
     "Action": ["s3:PutObject", "s3:GetObject", "s3:DeleteObject"],
     "Resource": "arn:aws:s3:::example-bucket/*"
   }
   ```

   This statement grants full access (read, write, delete) to all objects within the `example-bucket`.

3. **Explicit Deny for Deleting Objects in Another Bucket**:
   ```json
   {
     "Effect": "Deny",
     "Action": "s3:DeleteObject",
     "Resource": "arn:aws:s3:::restricted-bucket/*"
   }
   ```
   This statement explicitly denies the ability to delete objects in the `restricted-bucket`, regardless of any other permissions that might allow it.

## Order of Evaluation

When IAM evaluates a policy, it uses the following logic:

1. **By default, all requests are denied**: Access to resources is denied by default.

2. **An explicit allow overrides the default deny**: If a policy explicitly grants permission to an action, the action is allowed.

3. **An explicit deny overrides any allows**: If a policy explicitly denies permission to an action, the action is denied, even if another policy allows it.

**Troubleshooting IAM Permissions Issues**

When troubleshooting IAM permission issues, consider the following steps:

1. **Check IAM Policies**:

   - Verify that the policies attached to the user, group, or role grant the necessary permissions.
   - Ensure there are no explicit denies in any of the policies that might be overriding the allows.

2. **Use the IAM Policy Simulator**:

   - The IAM Policy Simulator lets you test and debug IAM policies. You can simulate API calls to see if the policies grant or deny access as expected.

3. **Enable AWS CloudTrail**:

   - Use CloudTrail to log API calls and analyze the logs to see which policies were evaluated and why a request was allowed or denied.

4. **Review Resource-Based Policies**:

   - If you are dealing with resource-based policies (such as S3 bucket policies), ensure that these policies also allow the necessary actions.

5. **Check Service Control Policies (SCPs)**:

   - If you are using AWS Organizations, ensure that SCPs attached to the organizational unit or account do not restrict the required actions.

6. **Use AWS IAM Access Analyzer**:
   - IAM Access Analyzer helps you identify the resources in your organization that are shared with an external entity and provides insights into how permissions are granted.

## IAM Best Practices

When working with IAM, consider the following best practices:

1. **Follow the principle of least privilege**: Only grant the permissions necessary to perform a task.
2. **Use IAM groups**: Assign permissions to groups and then add users to those groups.
3. **Rotate credentials regularly**: Change your passwords and access keys periodically, and remove unused credentials.
4. **Enable MFA**: Use multi-factor authentication for all users, especially those with elevated privileges.
5. **Use IAM roles for applications running on EC2**: Instead of storing AWS credentials in the EC2 instance.

## Using IAM Policy Variables

IAM policies can use variables to make them more flexible. For example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "${bucket_name}"
    }
  ]
}
```

By using the `${bucket_name}` variable, you can easily update the policy to apply to different buckets without having to create a new policy.

By understanding these detailed aspects of IAM, including how policies are structured, the order of evaluation, and how to troubleshoot permissions issues, you can better secure and manage access to your AWS resources.
