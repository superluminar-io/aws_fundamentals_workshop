# Lab 3: Networking and Security Groups

## Set up VPC and Security Groups Using CDK

In this hands-on section, you will learn how to set up a Virtual Private Cloud (VPC) with subnets and route tables, and configure security groups using the AWS Cloud Development Kit (CDK). This exercise will guide you through defining network infrastructure and security settings in code and deploying them to your AWS account.

## Prerequisites:

Ensure you have the necessary dependencies:

```bash
npm install @aws-cdk/aws-ec2
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

   This code sets up a VPC with both public and private subnets, configured with a NAT Gateway for internet access from private subnets.

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
      description: "Allow SSH and HTTP access to EC2 instance",
    });
    ec2SecurityGroup.addIngressRule(
      ec2.Peer.anyIpv4(),
      ec2.Port.tcp(22),
      "Allow SSH access"
    );
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
- **EC2 Security Group**: Allows SSH (port 22) and HTTP (port 80) access to the EC2 instance.
- **RDS Security Group**: Allows MySQL (port 3306) access from the EC2 security group.
- **Outputs**: Outputs the security group IDs for verification.

1. **Deploy the Stack**

   To deploy the stack to your AWS account, run the following command from the root directory of your CDK project:

   ```bash
   cdk deploy
   ```

   This command synthesizes the CloudFormation template from your CDK code and deploys the stack, creating the specified VPC and security groups in your account.

2. **Verify the Deployment**

   To verify the deployment:

   - Open the AWS Management Console.
   - Navigate to the VPC service and check the VPC, subnets, and route tables.
   - Navigate to the EC2 service and check the security groups.

By following these steps, you now have a fully functional VPC with public and private subnets and appropriate security groups for EC2 and RDS instances. This setup lays the groundwork for further network and security configurations in your AWS environment.
