# Lab 5: Amazon RDS

## Set up RDS for Relational Data Storage Using CDK

In this hands-on section, you will extend the existing CDK project from the previous labs to create an RDS instance. You will configure security groups and database parameters to set up a secure and optimized relational database.

## Prerequisites

For this lab, continue with the stack you created in Labs 3 and 4. If you need to start fresh or restore the previous setup, use the following link to get the starting code: [Lab 4 Stack](https://github.com/superluminar-io/aws_fundamentals_workshop_labs/blob/main/lab_4/lib/aws-fundamentals-workshop-labs-stack.ts)

## Create an RDS Instance with CDK

1. **Open Your CDK Project**

   Navigate to your existing CDK project directory.

2. **Extend the Stack File**

   Open the stack file located in the `lib` directory (e.g., `lib/aws-fundamentals-workshop-labs-stack.ts` for a TypeScript project). Add the following code to create an RDS instance:

```typescript
import {aws_secretsmanager, CfnOutput, Duration, RemovalPolicy, Stack, StackProps} from 'aws-cdk-lib'
import {
  SubnetType,
  Vpc,
  SecurityGroup, InstanceType, InstanceClass, InstanceSize,
} from 'aws-cdk-lib/aws-ec2'
import {
  ArnPrincipal,
  PolicyStatement,
} from 'aws-cdk-lib/aws-iam'
import {BlockPublicAccess, Bucket} from 'aws-cdk-lib/aws-s3'
import {Construct} from 'constructs'
import {
  Cluster,
  ContainerImage,
  FargateService,
  FargateTaskDefinition,
  ListenerConfig,
  LogDrivers, Secret
} from "aws-cdk-lib/aws-ecs";
import {ApplicationLoadBalancer, ApplicationProtocol} from "aws-cdk-lib/aws-elasticloadbalancingv2";
import {
  Credentials,
  DatabaseInstance,
  DatabaseInstanceEngine,
  MysqlEngineVersion
} from "aws-cdk-lib/aws-rds";

export class AwsFundamentalsWorkshopLabsStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props)

    // Create a VPC
    const vpc = new Vpc(this, 'MyVpc', {
      natGateways: 1, // Default is one in each AZ, this creates only one instead of two.
      subnetConfiguration: [
        {
          cidrMask: 24,
          name: 'public',
          subnetType: SubnetType.PUBLIC,
        },
        {
          cidrMask: 24,
          name: 'private',
          subnetType: SubnetType.PRIVATE_WITH_EGRESS, // This creates a private subnet with egress access to the internet.
        },
      ],
    })


    // Security Group for RDS instance
    const rdsSecurityGroup = new SecurityGroup(this, 'RDSSecurityGroup', {
      vpc,
      allowAllOutbound: true,
      description: 'Allow MySQL access to RDS instance',
    })

    // Create a secret for the RDS instance
    const databaseCredentials = Credentials.fromGeneratedSecret('admin',
      {
        secretName: 'MyRDSSecret'
      }
    );
    const databaseSecret = aws_secretsmanager.Secret.fromSecretNameV2(this, 'MyRDSSecret', databaseCredentials.secretName!);

    // Create an RDS instance
    const rdsInstance = new DatabaseInstance(this, 'MyRDSInstance', {
      // Choose the MySQL engine version
      engine: DatabaseInstanceEngine.mysql({
        version: MysqlEngineVersion.VER_8_0_37,
      }),
      // select the VPC
      vpc,
      // select the instance type
      instanceType: InstanceType.of(InstanceClass.T3, InstanceSize.MICRO),
      // select the subnet type to deploy the RDS instance in
      vpcSubnets: { subnetType: SubnetType.PRIVATE_WITH_EGRESS },
      // select the security group we created
      securityGroups: [rdsSecurityGroup],
      // set the credentials to be generated in AWS Secrets Manager
      credentials: databaseCredentials, // Generates a secret in Secrets Manager
      // set the multi-az to false for a single-az deployment
      multiAz: false,
      // select the allocated storage
      allocatedStorage: 20,
      // select the max allocated storage
      maxAllocatedStorage: 100,
      // disallow major version upgrades
      allowMajorVersionUpgrade: false,
      // enable auto-minor version upgrades
      autoMinorVersionUpgrade: true,
      // set the backup retention to 7 days
      backupRetention: Duration.days(7),
      // disable deletion protection
      deletionProtection: false,
      // set the database name
      databaseName: 'MyDatabase',
    })


    // Create the ECS Cluster
    const cluster = new Cluster(this, 'FargateCluster', {
      vpc,
    });

    // Create a Fargate Task Definition with a Container
    const fargateTaskDefinition = new FargateTaskDefinition(this, 'TaskDef');
    fargateTaskDefinition.addContainer('AppContainer', {
      containerName: 'web',
      image: ContainerImage.fromRegistry('ghcr.io/superluminar-io/dct:latest'),
      memoryLimitMiB: 512,
      cpu: 256,
      logging: LogDrivers.awsLogs({streamPrefix: 'myApp/webapp'}),
      portMappings: [{containerPort: 8081}],
      secrets: {
        DB_HOST: Secret.fromSecretsManager(databaseSecret, 'host'),
        DB_USERNAME: Secret.fromSecretsManager(databaseSecret, 'username'),
        DB_PASSWORD: Secret.fromSecretsManager(databaseSecret, 'password'),
      }
    });


    // Create a Fargate Service
    const service = new FargateService(this, 'FargateService', {
      cluster,
      taskDefinition: fargateTaskDefinition,
      minHealthyPercent: 100,
      vpcSubnets: {subnetType: SubnetType.PRIVATE_WITH_EGRESS},
      enableExecuteCommand: true,
    });

    // Create an Application Load Balancer that listens on port 80
    const lb = new ApplicationLoadBalancer(this, 'LoadBalancer', {vpc, internetFacing: true});
    const listener = lb.addListener('LBListener', {port: 80});

    // Register the ECS Service as a target of the Application Load Balancer
    service.registerLoadBalancerTargets(
      {
        containerName: 'web',
        containerPort: 8081,
        newTargetGroupId: 'ecs_webapp',
        listener: ListenerConfig.applicationListener(listener, {
          protocol: ApplicationProtocol.HTTP,
        }),
      },
    );

    rdsInstance.connections.allowDefaultPortFrom(service, 'Allow access from ECS service')

    // Create an S3 bucket
    const bucket = new Bucket(this, 'MyBucket', {
      removalPolicy: RemovalPolicy.DESTROY,
      autoDeleteObjects: true,
      publicReadAccess: false, // Ensure the bucket is not publicly accessible
      blockPublicAccess: BlockPublicAccess.BLOCK_ALL, // Block all public access
    })

    // Add a bucket policy that allows access from the ECS service
    bucket.addToResourcePolicy(
      new PolicyStatement({
        actions: [
          's3:GetObject',
          's3:ListBucket',
          's3:PutObject',
          's3:DeleteObject',
          's3:DeleteBucket',
        ],
        resources: [bucket.bucketArn, bucket.arnForObjects('*')],
        principals: [new ArnPrincipal(service.taskDefinition.taskRole.roleArn)],
      })
    )

    // Output the RDS instance endpoint
    new CfnOutput(this, 'RDSInstanceEndpoint', {
      value: rdsInstance.dbInstanceEndpointAddress,
    })

    // Output the RDS instance identifier
    new CfnOutput(this, 'RDSInstanceIdentifier', {
      value: rdsInstance.instanceIdentifier,
    })

    // Output the RDS instance secret ARN
    new CfnOutput(this, 'RDSInstanceSecretArn', {
      value: rdsInstance.secret?.secretArn || '',
    })

    //Output the Load Balancer DNS Name for easy reference
    new CfnOutput(this, 'LoadBalancerDNS', {
      value: lb.loadBalancerDnsName,
      description: 'DNS Name of the Application Load Balancer',
    })

    // Output the bucket name for easy reference
    new CfnOutput(this, 'BucketName', {
      value: bucket.bucketName,
      description: 'Name of the S3 bucket',
    })

    new CfnOutput(this, 'RDSSecurityGroupId', {
      value: rdsSecurityGroup.securityGroupId,
    })

    // Output the VPC ID
    new CfnOutput(this, 'VpcId', {
      value: vpc.vpcId,
    })
  }
}
```

## Lab Architecture

Before we proceed with verifying the deployment, let's take a moment to review the architecture we've built in this lab:

![Database Lab Architecture](../../media/lab_5_arch.drawio.svg)

This diagram illustrates the key components of our lab:

1. A Virtual Private Cloud (VPC) with public and private subnets spread across multiple Availability Zones, which we set up in previous labs.
2. An ECS service launched in the private subnet.
3. An RDS MySQL instance deployed in a private subnet, providing a managed relational database service.
4. Security groups controlling inbound and outbound traffic for the RDS instance.
5. A NAT Gateway allowing the RDS instance in the private subnet to access the internet for updates and patches.
6. AWS Secrets Manager storing the credentials for the RDS instance, enhancing security.

This architecture demonstrates a secure and scalable setup for integrating compute and database resources:

- The RDS instance is protected in a private subnet, not directly accessible from the internet.
- The security group rules allow the ECS Fargate service to communicate with the RDS instance on the MySQL port (3306).
- By using Secrets Manager, we avoid hardcoding database credentials and can rotate them easily.

This setup provides a solid foundation for building applications that require both compute power and a relational database, while maintaining proper security controls and following AWS best practices.

## Explanation of the Code

- **VPC**: Sets up a VPC with public and private subnets and a NAT Gateway.
- **ESC cluster**: Launches an ECS Fargate cluster including a service an related tasks.
- **RDS Security Group**: Allows MySQL (port 3306) access from the ECS Fargate service security group.
- **RDS Instance**: Creates an RDS MySQL instance in the private subnet with generated credentials stored in AWS Secrets Manager.
  - Configures various parameters like instance type, storage, backups, and database name.
- **Outputs**: Outputs the RDS instance endpoint, secret ARN, and security group IDs for verification.

This setup creates a complete environment with both compute (ECS) and database (RDS) resources, properly secured within a VPC structure.

3. **Deploy the Stack**

   To deploy the stack to your AWS account, run the following command from the root directory of your CDK project:

   ```bash
   cdk deploy --profile PROFILE_NAME
   ```

   This command synthesizes the CloudFormation template from your CDK code and deploys the stack, creating the specified VPC, security groups, and RDS instance in your account.

4. **Verify the Deployment**

   - **AWS Management Console**:

     - Navigate to the RDS service and find the instance created by the stack.
     - Check the security groups to ensure they are configured correctly.

   - **AWS CLI**:
     Run the following command to retrieve the RDS instance details:
     ```bash
     aws rds describe-db-instances \
     --db-instance-identifier INSTANCE_IDENTIFIER \
     --query "DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus,Endpoint.Address]" \
     --output table \
     --profile PROFILE_NAME
     ```

5. **Connect to the RDS Instance from the EC2 Instance**

   To connect to your RDS instance from the EC2 instance, follow these steps:

   a. Connect to your EC2 instance using AWS Systems Manager Session Manager.

   b. Install the MySQL client on the EC2 instance:

   ```bash
   sudo yum install mysql -y
   ```

   c. Retrieve the RDS endpoint and credentials:

   - Get the RDS endpoint from the CDK output or the RDS console.
   - Retrieve the database credentials from AWS Secrets Manager on your computer's terminal (don't use the Session Manager session for this step):
     ```bash
     aws secretsmanager get-secret-value --secret-id SECRET_ARN --query SecretString --output text --profile PROFILE_NAME
     ```
     Replace SECRET_ARN with the actual Secret ARN from the CDK output.

   d. Connect to the RDS instance using the MySQL client from the Session Manager session using the RDS endpoint from the CDK output:

   ```bash
   mysql -h RDS_ENDPOINT -u admin -p
   ```

   Enter the password when prompted.

   e. Once connected, you can verify the connection by running a simple SQL command:

   ```sql
   SHOW DATABASES;
   ```

   If you can successfully connect and run SQL commands, you've verified the connection between your EC2 instance and the RDS instance.

## Checkpoint

At this point, you should have:

- Created an RDS instance in the private subnet
- Configured the database security group
- Set up a secret in AWS Secrets Manager for database credentials
- Modified the EC2 instance to allow communication with the RDS instance
- Successfully connected to the RDS instance from the EC2 instance

If you're encountering issues, check the following:

- Ensure the RDS instance is in the correct subnet group
- Verify that the security group allows traffic from the EC2 instance to the RDS instance
- Check that the secret in Secrets Manager is correctly formatted
- Make sure the EC2 instance has the necessary permissions to access Secrets Manager
- Verify that you can resolve the RDS endpoint from the EC2 instance

## Best Practices and Security Considerations

1. Enable encryption at rest for your RDS instances.
2. Use SSL/TLS to encrypt connections to your RDS instance.
3. Regularly back up your databases and test the restore process.
4. Use Multi-AZ deployments for high availability and failover support.
5. Implement performance insights to monitor database performance.

## Additional Information: RDS Backup, Restore, and Encryption

The following sections provide informational content about RDS backup, restore procedures, and encryption options. These are not part of the hands-on exercise but are important concepts to understand when working with RDS in production environments.

### RDS Backup and Restore Procedures (Informational)

1. Automated Backups:

   In production environments, you would typically enable automated backups when creating your RDS instance:

   ```typescript
   const myDatabase = new rds.DatabaseInstance(this, 'MyDatabase', {
     // ... other configuration ...
     backupRetention: cdk.Duration.days(7),
   })
   ```

2. Manual Snapshots:

   Before making major changes, you might create a manual snapshot:

   ```bash
   aws rds create-db-snapshot --db-instance-identifier my-database --db-snapshot-identifier my-snapshot --region <YOUR_REGION>
   ```

3. Restore from Snapshot:
   To restore from a snapshot, you would use the AWS Management Console or AWS CLI:
   ```bash
   aws rds restore-db-instance-from-db-snapshot --db-instance-identifier my-new-database --db-snapshot-identifier my-snapshot --region <YOUR_REGION>
   ```

### RDS Encryption Options (Informational)

1. Encryption at Rest:

   In a production environment, you would typically enable encryption at rest when creating your RDS instance:

   ```typescript
   const myDatabase = new rds.DatabaseInstance(this, 'MyDatabase', {
     // ... other configuration ...
     storageEncrypted: true,
     encryptionKey: kms.Key.fromKeyArn(
       this,
       'MyKey',
       'arn:aws:kms:region:account:key/key-id'
     ),
   })
   ```

2. Encryption in Transit:
   For secure connections, you would use SSL/TLS for connections to your RDS instance. In your application, you would ensure you're using an SSL-enabled connection string.

Note: The above examples are for illustrative purposes and are not part of this lab's hands-on exercises. They represent best practices for production environments.

## Summary of Steps

- Set up the VPC and security groups using AWS CDK.
- Create an RDS instance with the specified configurations.
- Deploy the stack and verify the RDS instance using the AWS Management Console and CLI.
- Configure database parameters and option groups to optimize the RDS instance.

You've successfully set up an RDS instance for relational data storage using AWS CDK. This lab completes your journey through creating and managing various AWS services using CDK, equipping you with practical skills for building and managing cloud infrastructure.
