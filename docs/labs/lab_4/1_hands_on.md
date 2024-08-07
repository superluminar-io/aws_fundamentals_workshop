# Lab 4: Core AWS Services

## Deploy Core Services Using CDK

In this hands-on section, you will use AWS CDK to create an S3 bucket and an EC2 instance. You will also perform tasks in the AWS Management Console such as checking CloudWatch, adding a bucket policy, and running a CLI command to retrieve the instance ID.

## Prerequisites

For this and Lab 5, we will be continuing with the same stack and adding services to it. If you don't still have that stack, you can copy the code from here: FUTURE_LINK

Ensure you have the necessary dependencies:

```bash
npm install @aws-cdk/aws-ec2 @aws-cdk/aws-s3 @aws-cdk/aws-ssm
```

## Use CDK to Create an S3 Bucket and an EC2 Instance

1. **Open Your CDK Project**

   Navigate to your existing CDK project directory.

2. **Extend the Stack File**

   Open the stack file located in the `lib` directory (e.g., `lib/my-cdk-app-stack.ts` for a TypeScript project). Add the following code to create an S3 bucket and an EC2 instance:

   ```typescript
   import * as cdk from "aws-cdk-lib";
   import { Construct } from "constructs";
   import * as ec2 from "aws-cdk-lib/aws-ec2";
   import * as s3 from "aws-cdk-lib/aws-s3";
   import * as iam from "aws-cdk-lib/aws-iam";

   export class MyCdkAppStack extends cdk.Stack {
     constructor(scope: Construct, id: string, props?: cdk.StackProps) {
       super(scope, id, props);

       // Create a VPC
       const vpc = new ec2.Vpc(this, "MyVpc", {
         maxAzs: 3,
         natGateways: 1,
         subnetConfiguration: [
           {
             cidrMask: 24,
             name: "public",
             subnetType: ec2.SubnetType.PUBLIC,
           },
           {
             cidrMask: 24,
             name: "private",
             subnetType: ec2.SubnetType.PRIVATE_WITH_NAT,
           },
         ],
       });

       // Security Group for EC2 instance
       const ec2SecurityGroup = new ec2.SecurityGroup(
         this,
         "EC2SecurityGroup",
         {
           vpc,
           allowAllOutbound: true,
           description: "Allow HTTP access to EC2 instance",
         }
       );
       ec2SecurityGroup.addIngressRule(
         ec2.Peer.anyIpv4(),
         ec2.Port.tcp(80),
         "Allow HTTP access"
       );

       // IAM role for EC2 instance to use SSM
       const role = new iam.Role(this, "SSMRole", {
         assumedBy: new iam.ServicePrincipal("ec2.amazonaws.com"),
       });

       role.addManagedPolicy(
         iam.ManagedPolicy.fromAwsManagedPolicyName(
           "AmazonSSMManagedInstanceCore"
         )
       );

       // Create an EC2 instance
       const ec2Instance = new ec2.Instance(this, "MyEC2Instance", {
         vpc,
         instanceType: ec2.InstanceType.of(
           ec2.InstanceClass.T2,
           ec2.InstanceSize.MICRO
         ),
         machineImage: ec2.MachineImage.latestAmazonLinux2(),
         securityGroup: ec2SecurityGroup,
         vpcSubnets: { subnetType: ec2.SubnetType.PUBLIC },
         role: role,
       });

       // Create an S3 bucket
       const bucket = new s3.Bucket(this, "MyBucket", {
         removalPolicy: cdk.RemovalPolicy.DESTROY,
         autoDeleteObjects: true,
       });

       // Add a bucket policy
       bucket.addToResourcePolicy(
         new iam.PolicyStatement({
           actions: ["s3:GetObject"],
           resources: [bucket.arnForObjects("*")],
           principals: [new iam.AnyPrincipal()],
         })
       );

       // Output the bucket name and EC2 instance ID
       new cdk.CfnOutput(this, "BucketName", {
         value: bucket.bucketName,
       });
       new cdk.CfnOutput(this, "EC2InstanceId", {
         value: ec2Instance.instanceId,
       });
     }
   }
   ```

## Explanation of the Code

- **VPC**: Sets up a VPC with public and private subnets and a NAT Gateway.
- **EC2 Security Group**: Allows HTTP (port 80) access to the EC2 instance.
- **EC2 Instance**: Creates an EC2 instance in the public subnet with the specified security group and IAM role for SSM access.
- **S3 Bucket**: Creates an S3 bucket with a destroy policy.
- **Bucket Policy**: Adds a policy to the S3 bucket allowing public read access.
- **Outputs**: Outputs the S3 bucket name and EC2 instance ID for verification.

3. **Deploy the Stack**

   To deploy the stack to your AWS account, run the following command from the root directory of your CDK project:

   ```bash
   cdk deploy
   ```

   This command synthesizes the CloudFormation template from your CDK code and deploys the stack, creating the specified VPC, security groups, EC2 instance, and S3 bucket in your account.

4. **Verify the Deployment**

   - **AWS Management Console**:

     - Navigate to the S3 service and find the bucket created by the stack.
     - Navigate to the EC2 service and find the instance created by the stack.

   - **AWS CLI**:
     Run the following command to retrieve the instance ID:
     ```bash
     aws ec2 describe-instances --filters "Name=tag:Name,Values=MyEC2Instance" --query "Reservations[*].Instances[*].InstanceId" --output text
     ```

5. **Check CloudWatch**

   - **AWS Management Console**:
     - Navigate to the CloudWatch service.
     - Check the logs for the EC2 instance and the S3 bucket to ensure they are operating as expected.
     - Set up alarms if needed to monitor the health and performance of the resources.

6. **Verify a Bucket Policy**

   - **AWS Management Console**:
     - Navigate to the S3 service, select the created bucket.
     - Go to the "Permissions" tab and verify the bucket policy is in place.

   Example of verifying bucket policy:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::${YOUR_BUCKET_NAME}/*"
       }
     ]
   }
   ```

## Lab Architecture

Before we proceed with verifying the deployment, let's take a moment to review the architecture we've built in this lab:

![Core Services Lab Architecture](media/lab_4_arch.drawio.svg)

This diagram illustrates the key components of our lab:

1. A Virtual Private Cloud (VPC) with public and private subnets spread across multiple Availability Zones, which we set up in the previous lab.
2. An EC2 instance launched in the public subnet, which we can connect to using Systems Manager Session Manager.
3. Security groups controlling inbound and outbound traffic for our EC2 instance.
4. An S3 bucket for storing objects, with a bucket policy controlling access.
5. IAM roles and policies managing permissions for the EC2 instance and S3 bucket access.
6. CloudWatch for monitoring and logging of our EC2 instance and S3 bucket activities.

This architecture demonstrates a secure and scalable setup for core AWS services, allowing us to manage compute resources and object storage while maintaining proper security controls and monitoring capabilities.

Now, let's proceed with verifying the deployment of these resources:

### Connecting to Your EC2 Instance

1. **Open the AWS Systems Manager Console**

   Go to the AWS Management Console and navigate to the Systems Manager service.

2. **Navigate to Session Manager**

   In the Systems Manager console, look for Session Manager in the left navigation pane under the "Instances & Nodes" section.

3. **Start a Session**

   - Select the instance you want to connect to from the list of managed instances.
   - Click the "Start session" button.

   This will open a browser-based shell session to your instance, allowing you to manage it without needing an SSH connection.

## Checkpoint

At this point, you should have:

- Created an S3 bucket using CDK
- Launched an EC2 instance in the public subnet
- Configured the EC2 instance to use Systems Manager Session Manager
- Created a CloudWatch alarm for the EC2 instance
- Successfully connected to the EC2 instance using Session Manager

If you're encountering issues, check the following:

- Verify that the S3 bucket was created successfully
- Ensure the EC2 instance has the correct IAM role for Systems Manager access
- Check that the VPC endpoints for Systems Manager are correctly configured
- Verify that the CloudWatch alarm is set up with the correct metrics

## Best Practices and Security Considerations

### EC2 Instance Management

1. Use IAM roles instead of storing AWS credentials on EC2 instances.
2. Regularly patch and update your EC2 instances to maintain security.
3. Use Amazon CloudWatch for monitoring and set up alarms for critical metrics.
4. Implement proper security group rules to control inbound and outbound traffic.
5. Use Amazon EC2 Auto Scaling to automatically adjust capacity based on demand.
6. Use EC2 Instance Metadata Service Version 2 (IMDSv2) for improved security.

### S3 Security

1. Implement S3 bucket policies to control access to your data.
2. Enable versioning on buckets to protect against accidental deletions or overwrites.
3. Use S3 server-side encryption for data at rest.
4. Implement lifecycle policies to manage object retention and reduce costs.

### General Security

1. Follow the principle of least privilege when assigning permissions.
2. Enable AWS CloudTrail to log API calls for your account.
3. Regularly review and audit your security configurations.
4. Use AWS Config to assess, audit, and evaluate the configurations of your AWS resources.

### Performance and Cost Optimization

1. Choose the right instance types based on your workload requirements.
2. Use Amazon EC2 Spot Instances for flexible, fault-tolerant applications to reduce costs.
3. Implement caching strategies using services like Amazon ElastiCache to improve performance.
4. Use AWS Trusted Advisor to get real-time guidance on best practices for cost optimization, security, fault tolerance, and performance improvement.

## S3 Bucket Policies and Versioning

When creating an S3 bucket, consider implementing these security features:

1. Bucket Policy: Restrict access to your S3 bucket using a bucket policy. Here's an example that allows read access only from a specific IAM role:

```typescript
const myBucketPolicy = new s3.BucketPolicy(this, "MyBucketPolicy", {
  bucket: myBucket,
});

myBucketPolicy.document.addStatements(
  new iam.PolicyStatement({
    actions: ["s3:GetObject"],
    resources: [myBucket.arnForObjects("*")],
    principals: [new iam.ArnPrincipal("arn:aws:iam::123456789012:role/MyRole")],
  })
);
```

2. Versioning: Enable versioning to keep multiple variants of objects in the bucket:

```typescript
const myBucket = new s3.Bucket(this, "MyBucket", {
  versioned: true,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
  autoDeleteObjects: true,
});
```

Excellent! You have now successfully created and deployed an S3 bucket and an EC2 instance using AWS CDK. You've also verified their configurations and monitored their operations with CloudWatch. This lab has expanded your understanding of managing basic AWS services both programmatically and through the AWS console.
