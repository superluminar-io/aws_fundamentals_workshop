# Lab 6: Operations & Troubleshooting

## Connecting to your ECS Fargate Container

In this hands-on section, you will extend the existing CDK project from the previous labs to be able to connect to the ECS Fargate containers.
You will update the definition of the cluster, as well, as the service, update the `TaskExecutionRole` and add a KMS Key to be able to connect to the container securely.

## Prerequisites

For this lab, continue with the stack you created in Labs 3, 4 and 5. If you need to start fresh or restore the previous setup, use the following link to get the starting code: [Lab 4 Stack](https://github.com/superluminar-io/aws_fundamentals_workshop_labs/blob/main/lab_4/lib/aws-fundamentals-workshop-labs-stack.ts).

Additionally, you need to install the AWS Session Manager Plugin. You can find the installation instructions [here](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html).

## Update our ECS Fargate cluster with CDK

### 1. Open Your CDK Project

   Navigate to your existing CDK project directory.

### 2. Extend the Stack File

   Open the stack file located in the `lib` directory (e.g., `lib/aws-fundamentals-workshop-labs-stack.ts` for a TypeScript project). Add the following code to create an RDS instance:

```typescript
import {CfnOutput, RemovalPolicy, Stack, StackProps} from 'aws-cdk-lib'
import {
  SubnetType,
  Vpc,
  SecurityGroup,
  Port, Peer,
} from 'aws-cdk-lib/aws-ec2'
import {
  ArnPrincipal, Effect,
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
  LogDrivers
} from "aws-cdk-lib/aws-ecs";
import {ApplicationLoadBalancer, ApplicationProtocol} from "aws-cdk-lib/aws-elasticloadbalancingv2";
import {Key} from "aws-cdk-lib/aws-kms";

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

    // Security Group for EC2 instance
    const ec2SecurityGroup = new SecurityGroup(this, 'EC2SecurityGroup', {
      vpc,
      allowAllOutbound: true,
      description: 'Allow HTTP access to EC2 instance',
    })
    ec2SecurityGroup.addIngressRule(
      Peer.anyIpv4(),
      Port.tcp(80),
      'Allow HTTP access'
    )

    // Create a KMS key for access to the ECS cluster via SSM
    const ksmEncryptionKey = new Key(this, 'ECSClusterKey', {
      enableKeyRotation: true,
    });
    
    // Create the ECS Cluster
    const cluster = new Cluster(this, 'FargateCluster', {
      vpc,
      executeCommandConfiguration: { kmsKey: ksmEncryptionKey }
    });

    // Create a Fargate Task Definition with a Container
    const fargateTaskDefinition = new FargateTaskDefinition(this, 'TaskDef');
    fargateTaskDefinition.addContainer('AppContainer', {
      containerName: 'web',
      image: ContainerImage.fromRegistry('nginx:latest'),
      memoryLimitMiB: 512,
      cpu: 256,
      logging: LogDrivers.awsLogs({streamPrefix: 'myApp/nginx'}),
      portMappings: [{containerPort: 80}],
    });
    
    fargateTaskDefinition.addToTaskRolePolicy(
      new PolicyStatement({
        effect: Effect.ALLOW,
        actions: ['ssmmessages:CreateControlChannel', 'ssmmessages:CreateDataChannel', 'ssmmessages:OpenControlChannel', 'ssmmessages:OpenDataChannel'],
        resources: ['*']
      }),
    )

    fargateTaskDefinition.addToTaskRolePolicy(
      new PolicyStatement({
        effect: Effect.ALLOW,
        actions: ['kms:Decrypt'],
        resources: [ksmEncryptionKey.keyArn]
      }),
    );

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
        containerPort: 80,
        newTargetGroupId: 'ecs_nginx',
        listener: ListenerConfig.applicationListener(listener, {
          protocol: ApplicationProtocol.HTTP,
        }),
      },
    );

    // Security Group for RDS instance
    const rdsSecurityGroup = new SecurityGroup(this, 'RDSSecurityGroup', {
      vpc,
      allowAllOutbound: true,
      description: 'Allow MySQL access to RDS instance',
    })
    rdsSecurityGroup.addIngressRule(
      ec2SecurityGroup,
      Port.tcp(3306),
      'Allow MySQL access from EC2 instance'
    )
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

    // Output the Security Group IDs
    new CfnOutput(this, 'EC2SecurityGroupId', {
      value: ec2SecurityGroup.securityGroupId,
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

#### Code Explaination

Before we proceed with verifying the deployment, let's take a moment to review　we've built in this lab:
* Added an AWS KMS Key `ksmEncryptionKey` to our stack. This encryption key is AWS-managed and will be used to enable secure access to the containers via SSM.
* Updated the ECS Cluster definition to include the `executeCommandConfiguration` property with the KMS key.
* Update the ECS Service definition to include the `enableExecuteCommand` property set to `true`.
* Added the necessary IAM permissions to the Task Role to allow the ECS Task to interact with the KMS key and SSM.


### 3. Deploy the Stack

   To deploy the stack to your AWS account, run the following command from the root directory of your CDK project:

   ```bash
   cdk deploy --profile PROFILE_NAME
   ```

   This command synthesizes the CloudFormation template from your CDK code and deploys the stack, creating the specified VPC, security groups, and RDS instance in your account.

### 4. Verify the Deployment
- **AWS Management Console**:

    - Navigate to the ECS service and get the ARN of the service, as well as, of the running task.

- **AWS CLI**:
  Run the following command to retrieve the RDS instance details:
  ```bash
  aws ecs execute-command \
    --cluster CLUSTER_ARN \
    --task TASK_ARN \
    --container CONTAINER_NAME --interactive --command "/bin/sh"
  ```

If you get the following message
`An error occurred (InvalidParameterException) when calling the ExecuteCommand operation: The execute command failed because execute command was not enabled when the task was run or the execute command agent isn’t running. Wait and try again or run a new task with execute command enabled and try again.`
you need to restart the task.

Quick-check if the task is enabled for ECS Exec:
```bash
aws ecs describe-tasks \
--cluster CLUSTER_ARN \
--tasks TASK_ARN | grep enableExecuteCommand
```
If the result is ` "enableExecuteCommand": true,` you can re-run ` aws ecs execute-command`.

- **On the Container**:
    Run the following command to see the directory structure within the container:
    ```bash
    ls -la 
    ```
    You should see the directory structure of the container. Play around with some other commands like `df`,`touch file_name.txt`, etc.

## Summary of Steps

- Installed `aws-session-manager` in addition to the AWS CLI.
- Updated the cluster and service configuration to enable ECS Exec.
- Added addition policies to the Task Execution Role.
- Connected to the `web` container through AWS SSM with AWS CLI.

You've successfully connected to your container. This lab enhance not only your oper your operations knowledge, but the journey through the AWS Fundamentals Workshop, as well.
Congratulations on completing the workshop!


  
  

