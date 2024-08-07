# Lab 5: Amazon RDS

## Set up RDS for Relational Data Storage Using CDK

In this hands-on section, you will extend the existing CDK project from the previous labs to create an RDS instance. You will configure security groups and database parameters to set up a secure and optimized relational database.

## Prerequisites

For this lab, continue with the stack you created in Labs 3 and 4. If you need to start fresh or restore the previous setup, use the following link to get the starting code: FUTURE_LINK

Ensure you have the necessary dependencies:

```bash
npm install @aws-cdk/aws-ec2 @aws-cdk/aws-rds @aws-cdk/aws-secretsmanager
```

## Create an RDS Instance with CDK

1. **Open Your CDK Project**

   Navigate to your existing CDK project directory.

2. **Extend the Stack File**

   Open the stack file located in the `lib` directory (e.g., `lib/my-cdk-app-stack.ts` for a TypeScript project). Add the following code to create an RDS instance:

   ```typescript
   import * as cdk from "aws-cdk-lib";
   import { Construct } from "constructs";
   import * as ec2 from "aws-cdk-lib/aws-ec2";
   import * as rds from "aws-cdk-lib/aws-rds";
   import * as secretsmanager from "aws-cdk-lib/aws-secretsmanager";

   export class MyCdkAppStack extends cdk.Stack {
     constructor(scope: Construct, id: string, props?: cdk.StackProps) {
       super(scope, id, props);

       // Existing code to create a VPC and security groups
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
       new ec2.Instance(this, "MyEC2Instance", {
         vpc,
         instanceType: new ec2.InstanceType("t2.micro"),
         machineImage: ec2.MachineImage.latestAmazonLinux(),
         securityGroup: ec2SecurityGroup,
         vpcSubnets: { subnetType: ec2.SubnetType.PUBLIC },
         role: role,
       });

       // Security Group for RDS instance
       const rdsSecurityGroup = new ec2.SecurityGroup(
         this,
         "RDSSecurityGroup",
         {
           vpc,
           allowAllOutbound: true,
           description: "Allow MySQL access to RDS instance",
         }
       );
       rdsSecurityGroup.addIngressRule(
         ec2SecurityGroup,
         ec2.Port.tcp(3306),
         "Allow MySQL access from EC2 instance"
       );

       // Create an RDS instance
       const rdsInstance = new rds.DatabaseInstance(this, "MyRDSInstance", {
         engine: rds.DatabaseInstanceEngine.mysql({
           version: rds.MysqlEngineVersion.VER_8_0_19,
         }),
         vpc,
         instanceType: ec2.InstanceType.of(
           ec2.InstanceClass.BURSTABLE2,
           ec2.InstanceSize.MICRO
         ),
         vpcSubnets: { subnetType: ec2.SubnetType.PRIVATE_WITH_NAT },
         securityGroups: [rdsSecurityGroup],
         credentials: rds.Credentials.fromGeneratedSecret("admin"), // Generates a secret in Secrets Manager
         multiAz: false,
         allocatedStorage: 20,
         maxAllocatedStorage: 100,
         allowMajorVersionUpgrade: false,
         autoMinorVersionUpgrade: true,
         backupRetention: cdk.Duration.days(7),
         deletionProtection: false,
         databaseName: "MyDatabase",
       });

       // Output the RDS instance endpoint and credentials
       new cdk.CfnOutput(this, "RDSInstanceEndpoint", {
         value: rdsInstance.dbInstanceEndpointAddress,
       });
       new cdk.CfnOutput(this, "RDSInstanceSecretArn", {
         value: rdsInstance.secret?.secretArn || "",
       });

       // Output the security group IDs for verification
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
- **RDS Instance**: Creates an RDS MySQL instance in the private subnet with generated credentials stored in AWS Secrets Manager.
- **Outputs**: Outputs the RDS instance endpoint, secret ARN, and security group IDs for verification.

3. **Deploy the Stack**

   To deploy the stack to your AWS account, run the following command from the root directory of your CDK project:

   ```bash
   cdk deploy
   ```

   This command synthesizes the CloudFormation template from your CDK code and deploys the stack, creating the specified VPC, security groups, and RDS instance in your account.

4. **Verify the Deployment**

   - **AWS Management Console**:

     - Navigate to the RDS service and find the instance created by the stack.
     - Check the security groups to ensure they are configured correctly.

   - **AWS CLI**:
     Run the following command to retrieve the RDS instance details:
     ```bash
     aws rds describe-db-instances --db-instance-identifier MyRDSInstance --query "DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus,Endpoint.Address]" --output table
     ```

5. **Configure Database Parameters**

   - **Parameter Groups**: In the AWS Management Console, navigate to the RDS service, select your database instance, and configure the parameter group to fine-tune database settings.
   - **Option Groups**: Add necessary options to your RDS instance using option groups, such as enabling performance insights or additional monitoring tools.

## RDS Backup and Restore Procedures

1. Automated Backups:

   - Enable automated backups when creating your RDS instance:
     ```typescript
     const myDatabase = new rds.DatabaseInstance(this, "MyDatabase", {
       // ... other configuration ...
       backupRetention: cdk.Duration.days(7),
     });
     ```

2. Manual Snapshots:

   - Create a manual snapshot before major changes:
     ```bash
     aws rds create-db-snapshot --db-instance-identifier my-database --db-snapshot-identifier my-snapshot
     ```

3. Restore from Snapshot:
   - To restore from a snapshot, use the AWS Management Console or AWS CLI:
     ```bash
     aws rds restore-db-instance-from-db-snapshot --db-instance-identifier my-new-database --db-snapshot-identifier my-snapshot
     ```

## RDS Encryption Options

1. Encryption at Rest:

   - Enable encryption at rest when creating your RDS instance:
     ```typescript
     const myDatabase = new rds.DatabaseInstance(this, "MyDatabase", {
       // ... other configuration ...
       storageEncrypted: true,
       encryptionKey: kms.Key.fromKeyArn(
         this,
         "MyKey",
         "arn:aws:kms:region:account:key/key-id"
       ),
     });
     ```

2. Encryption in Transit:
   - Use SSL/TLS for connections to your RDS instance. In your application, ensure you're using an SSL-enabled connection string.

## Summary of Steps

- Set up the VPC and security groups using AWS CDK.
- Create an RDS instance with the specified configurations.
- Deploy the stack and verify the RDS instance using the AWS Management Console and CLI.
- Configure database parameters and option groups to optimize the RDS instance.

You've successfully set up an RDS instance for relational data storage using AWS CDK. This lab completes your journey through creating and managing various AWS services using CDK, equipping you with practical skills for building and managing cloud infrastructure.
