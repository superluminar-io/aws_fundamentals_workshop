# Lab 3: Networking and Security Groups

## Set up VPC and Security Groups Using CDK

In this hands-on section, you will learn how to set up a Virtual Private Cloud (VPC) with subnets and route tables, and configure security groups using the AWS Cloud Development Kit (CDK). This exercise will guide you through defining network infrastructure and security settings in code and deploying them to your AWS account. Additionally, you will learn to use AWS Systems Manager Session Manager for secure instance access, avoiding the need for open SSH ports.

## Prerequisites:

Ensure you have the necessary dependencies:

```bash
npm install @aws-cdk/aws-ec2 @aws-cdk/aws-ssm
```

## Define a VPC with Subnets and Route Tables

1. **Open Your CDK Project**

   Navigate to your existing CDK project directory.

2. **Define the VPC in Your Stack**

   Open the stack file located in the `lib` directory (e.g., `lib/my-cdk-app-stack.ts` for a TypeScript project). Add the following code to define a VPC with public and private subnets:

   ```typescript
   import * as cdk from "aws-cdk-lib";
   import { Construct } from "constructs";
   import * as ec2 from "aws-cdk-lib/aws-ec2";
   import * as ssm from "aws-cdk-lib/aws-ssm";

   export class MyCdkAppStack extends cdk.Stack {
     constructor(scope: Construct, id: string, props?: cdk.StackProps) {
       super(scope, id, props);

       // Create a VPC
       const vpc = new ec2.Vpc(this, "MyVpc", {
         maxAzs: 3, // Default is all AZs in the region
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

       // Output the VPC ID
       new cdk.CfnOutput(this, "VpcId", {
         value: vpc.vpcId,
       });
     }
   }
   ```

   > **Important:** Unlike the previous labs, we will be extending this stack in the following labs. It's crucial to keep this code as is before moving on to the next lab. This will serve as the foundation for our upcoming work with AWS services.

   This code sets up a VPC with both public and private subnets, configured with a NAT Gateway for internet access from private subnets.

## Understanding CIDR Blocks in VPC Configuration

In the VPC configuration, we use CIDR blocks to define the IP address ranges for our subnets:

- The VPC uses a CIDR block of /16, which provides 65,536 available IP addresses.
- Public and private subnets use /24 CIDR blocks, each providing 256 IP addresses.

This configuration allows for efficient IP address allocation while maintaining a clear separation between public and private resources.

## VPC Network Diagram

Here's a visual representation of our VPC structure:

```
[Insert a network diagram showing the VPC with public and private subnets]
```

## Configure Security Groups

**Add Security Groups to Your Stack**

Extend the stack file to include security groups:

```typescript
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
    const ec2SecurityGroup = new ec2.SecurityGroup(this, "EC2SecurityGroup", {
      vpc,
      allowAllOutbound: true,
      description: "Allow HTTP access to EC2 instance",
    });
    ec2SecurityGroup.addIngressRule(
      ec2.Peer.anyIpv4(),
      ec2.Port.tcp(80),
      "Allow HTTP access"
    );

    // Security Group for RDS instance
    const rdsSecurityGroup = new ec2.SecurityGroup(this, "RDSSecurityGroup", {
      vpc,
      allowAllOutbound: true,
      description: "Allow MySQL access to RDS instance",
    });
    rdsSecurityGroup.addIngressRule(
      ec2SecurityGroup,
      ec2.Port.tcp(3306),
      "Allow MySQL access from EC2 instance"
    );

    // Output the Security Group IDs
    new cdk.CfnOutput(this, "EC2SecurityGroupId", {
      value: ec2SecurityGroup.securityGroupId,
    });
    new cdk.CfnOutput(this, "RDSSecurityGroupId", {
      value: rdsSecurityGroup.securityGroupId,
    });
  }
}
```

## Explanation of the Code

- **VPC**: Sets up a VPC with public and private subnets and a NAT Gateway.
- **EC2 Security Group**: Allows HTTP (port 80) access to the EC2 instance.
- **RDS Security Group**: Allows MySQL (port 3306) access from the EC2 security group.
- **Outputs**: Outputs the security group IDs for verification.

## Using AWS Systems Manager Session Manager

AWS Systems Manager Session Manager is a fully managed AWS service that provides secure and auditable instance management without needing to open inbound ports, manage bastion hosts, or manage SSH keys. It offers a browser-based shell and CLI access to your instances.

### Enabling Session Manager

To use Session Manager, ensure the following prerequisites are met:

1. **Install SSM Agent**: The SSM Agent must be installed and running on the EC2 instances. Most Amazon Machine Images (AMIs) have the SSM Agent pre-installed.
2. **IAM Role**: Your EC2 instances must have an IAM role with the necessary permissions to communicate with the Systems Manager service.

**Modify the CDK Stack to Attach IAM Role**

Extend the stack file to include an IAM role for the EC2 instance:

```typescript
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
    const ec2SecurityGroup = new ec2.SecurityGroup(this, "EC2SecurityGroup", {
      vpc,
      allowAllOutbound: true,
      description: "Allow HTTP access to EC2 instance",
    });
    ec2SecurityGroup.addIngressRule(
      ec2.Peer.anyIpv4(),
      ec2.Port.tcp(80),
      "Allow HTTP access"
    );

    // Security Group for RDS instance
    const rdsSecurityGroup = new ec2.SecurityGroup(this, "RDSSecurityGroup", {
      vpc,
      allowAllOutbound: true,
      description: "Allow MySQL access to RDS instance",
    });
    rdsSecurityGroup.addIngressRule(
      ec2SecurityGroup,
      ec2.Port.tcp(3306),
      "Allow MySQL access from EC2 instance"
    );

    // IAM role for EC2 instance to use SSM
    const role = new iam.Role(this, "SSMRole", {
      assumedBy: new iam.ServicePrincipal("ec2.amazonaws.com"),
    });

    role.addManagedPolicy(
      iam.ManagedPolicy.fromAwsManagedPolicyName("AmazonSSMManagedInstanceCore")
    );

    // Launch an EC2 instance with the IAM role
    new ec2.Instance(this, "Instance", {
      vpc,
      instanceType: new ec2.InstanceType("t2.micro"),
      machineImage: ec2.MachineImage.latestAmazonLinux(),
      securityGroup: ec2SecurityGroup,
      role: role,
    });

    // Output the Security Group IDs
    new cdk.CfnOutput(this, "EC2SecurityGroupId", {
      value: ec2SecurityGroup.securityGroupId,
    });
    new cdk.CfnOutput(this, "RDSSecurityGroupId", {
      value: rdsSecurityGroup.securityGroupId,
    });
  }
}
```

### Connecting to Your EC2 Instance

1. **Open the AWS Systems Manager Console**

   Go to the AWS Management Console and navigate to the Systems Manager service.

2. **Navigate to Session Manager**

   In the Systems Manager console, look for Session Manager in the left navigation pane under the "Instances & Nodes" section.

3. **Start a Session**

   - Select the instance you want to connect to from the list of managed instances.
   - Click the "Start session" button.

   This will open a browser-based shell session to your instance, allowing you to manage it without needing an SSH connection.

## Deploy the Stack

To deploy the stack to your AWS account, run the following command from the root directory of your CDK project:

```bash
cdk deploy
```

This command synthesizes the CloudFormation template from your CDK code and deploys the stack, creating the specified VPC, security groups, and EC2 instance with the IAM role for Systems Manager access.

## Verify the Deployment

To verify the deployment:

- **VPC**: Open the AWS Management Console and navigate to the VPC service. Check the VPC, subnets, and route tables to ensure they were created correctly.
- **Security Groups**: Navigate to the EC2 service and check the security groups to ensure they have the correct rules.
- **Session Manager Access**: Use the Systems Manager console to start a session with your EC2 instance and verify that you can access it without SSH.

## Explanation of AWS Systems Manager Session Manager

AWS Systems Manager Session Manager is a powerful tool that provides the following benefits:

- **Secure Access**: Eliminates the need to open inbound ports (such as SSH or RDP) and manage bastion hosts, thereby reducing security risks.
- **Auditability**: All session activity is logged in AWS CloudTrail, providing an audit trail of access and actions taken on the instances.
- **Ease of Use**: Allows administrators to manage instances using a web browser or AWS CLI, simplifying access management.

### Benefits of Using Session Manager

1. **Enhanced Security**: By not requiring open ports for SSH or RDP, Session Manager reduces the attack surface of your instances. Access is managed through IAM policies, which can be fine-tuned for granular control.
2. **Audit and Compliance**: Every session is logged, making it easier to meet compliance and auditing requirements. You can review session logs to monitor activity and detect any unauthorized actions.
3. **No Need for SSH Keys**: Managing SSH keys can be cumbersome and risky if not handled properly. Session Manager eliminates the need for key management, streamlining access control.

### Setting Up Permissions for Session Manager

Ensure your EC2 instances have the necessary IAM role with the `AmazonSSMManagedInstanceCore` policy attached. This policy provides the required permissions for the instance to communicate with Systems Manager.

```typescript
const role = new iam.Role(this, "SSMRole", {
  assumedBy: new iam.ServicePrincipal("ec2.amazonaws.com"),
});

role.addManagedPolicy(
  iam.ManagedPolicy.fromAwsManagedPolicyName("AmazonSSMManagedInstanceCore")
);
```

By following these steps, you now have a secure, auditable, and easy-to-use method for managing your EC2 instances using AWS Systems Manager Session Manager. This approach aligns with best practices for securing and managing AWS infrastructure.
