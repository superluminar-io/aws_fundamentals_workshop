# Lab 4: Core AWS Services

In this lab, you will gain an understanding of some of the core AWS services, including ECS, S3, and RDS, and learn how to navigate and use the AWS Management Console and Command Line Interface (CLI).

## Introduction to Core Services

AWS offers a wide range of cloud services, but some of the core services that are fundamental to most applications today include ECS, S3, and RDS. Let's take a closer look at each of these services.

## Amazon ECS (Elastic Container Service)

Amazon ECS provides highly scalable container orchestration in the AWS Cloud. Using ECS eliminates the need to manage servers, allowing you to develop and deploy containerized applications faster. With ECS, you can run and scale container workloads, configure networking and security, and manage storage. ECS automatically scales to handle changes in demand or traffic spikes, reducing the need for manual capacity planning.

Key Features:

- **Clusters**: Groups of managed compute resources for running containers.
- **Tasks**: One or more containers that run together as a unit.
- **Services**: Manage and scale tasks to ensure high availability.
- **Task Definitions**: Blueprints defining how containers should be deployed and configured.
- **IAM Roles & Security Groups**: Access control and network rules for container workloads.


## Amazon S3 (Simple Storage Service)

Amazon S3 is an object storage service that offers industry-leading scalability, data availability, security, and performance. Customers of all sizes and industries can use S3 to store and protect any amount of data for a range of use cases, such as websites, mobile applications, backup and restore, archive, enterprise applications, IoT devices, and big data analytics.

Key features:

- **Buckets**: Containers for storing objects.
- **Objects**: Files and metadata stored in buckets.
- **Storage Classes**: Different tiers for different use cases (e.g., Standard, Glacier).
- **Access Control**: Fine-grained control over who can access data.
- **Data Management**: Versioning, lifecycle policies, and replication.

## Amazon RDS (Relational Database Service)

Amazon RDS makes it easy to set up, operate, and scale a relational database in the cloud. It provides cost-efficient and resizable capacity while automating time-consuming administration tasks such as hardware provisioning, database setup, patching, and backups. This allows you to focus on your applications so you can give them the fast performance, high availability, security, and compatibility they need.

Key features:

- **Database Engines**: Support for MySQL, PostgreSQL, MariaDB, Oracle, and SQL Server.
- **Automated Backups**: Automatic backups of your databases.
- **Multi-AZ Deployments**: High availability and failover support.
- **Read Replicas**: Improve read performance by creating replicas.
- **Security**: Network isolation and encryption at rest and in transit.

## Understanding the AWS Management Console and CLI

### AWS Management Console

The AWS Management Console is a web-based interface for accessing and managing your AWS resources. It provides an easy-to-navigate user interface where you can manage services, monitor your account, and configure resources.

Key features:

- **Dashboard**: Centralized view of your AWS resources.
- **Service Navigation**: Easy access to AWS services through a search bar and service list.
- **Resource Management**: Create, manage, and delete resources.
- **Monitoring and Alerts**: View metrics and set up alarms for your resources.
- **Billing and Cost Management**: Monitor your AWS usage and costs.

### AWS CLI (Command Line Interface)

The AWS CLI is a unified tool to manage your AWS services. With just one tool to download and configure, you can control multiple AWS services from the command line and automate them through scripts.

Key features:

- **Command Line Access**: Manage AWS services using commands.
- **Automation**: Automate tasks with shell scripts and batch files.
- **Configuration**: Easily configure credentials and default settings.
- **Scripting Support**: Integrate with your existing scripts and tools.
- **Comprehensive Coverage**: Support for all AWS services.

### Examples of Basic Commands

1. **AWS Management Console**:

   - To create an S3 bucket: Navigate to the S3 service, click "Create bucket", and follow the prompts.
   - To set up an RDS database: Navigate to the RDS service, click "Create database", and choose your database engine and settings.
   - To launch an ECS cluster with a single Task: Navigate to the ECS service, 
     - create a Cluster, 
     - create a Service,
     - register a Task Definition,
     - create an Application Load Balancer,
     - register the ECS Service with the Target Group,

   2. **AWS CLI**:
      - To create an S3 bucket:
        ```bash
        aws s3 mb s3://my-bucket-name
        ```
      - To create an RDS database instance:
        ```bash
        aws rds create-db-instance \
            --db-instance-identifier mydatabase \
            --allocated-storage 20 \
            --db-instance-class db.t2.micro \
            --engine mysql \
            --master-username admin \
            --master-user-password mypassword123 \
            --backup-retention-period 3
        ```
      - To create an ECS cluster and start a single task:
        ```bash
        # Create an ECS Cluster
        aws ecs create-cluster --cluster-name FargateCluster
        

        #Register the Task Definition from a JSON file
        aws ecs register-task-definition --cli-input-json file://task-definition.json
    
        #Create a Fargate Service
        aws ecs create-service \
        --cluster FargateCluster \
        --service-name FargateService \
        --task-definition TaskDef \
        --desired-count 1 \
        --launch-type FARGATE \
        --network-configuration "awsvpcConfiguration={subnets=[<SUBNET_ID>],securityGroups=[<SECURITY_GROUP_ID>],assignPublicIp=ENABLED}"
 
        # Create an Application Load Balancer
        aws elbv2 create-load-balancer \
        --name LoadBalancer \
        --type application \
        --scheme internet-facing \
        --subnets <SUBNET_ID_1> <SUBNET_ID_2>
    
        # Retrieve the Load Balancer ARN
        LB_ARN=$(aws elbv2 describe-load-balancers --names LoadBalancer --query 'LoadBalancers[0].LoadBalancerArn' --output text)
    
        # Create a Listener on port 80
        aws elbv2 create-listener \
        --load-balancer-arn $LB_ARN \
        --protocol HTTP \
        --port 80 \
        --default-actions Type=forward,TargetGroupArn=<TARGET_GROUP_ARN>
    
        # Create a Target Group for the ECS Service
        aws elbv2 create-target-group \
        --name ecs_nginx \
        --protocol HTTP \
        --port 80 \
        --vpc-id <VPC_ID> \
        --target-type ip
    
        # Retrieve the Target Group ARN
        TG_ARN=$(aws elbv2 describe-target-groups --names ecs_nginx --query 'TargetGroups[0].TargetGroupArn' --output text)
    
        # Register the ECS Service with the Target Group
        aws elbv2 register-targets \
        --target-group-arn $TG_ARN \
        --targets Id=<TASK_PRIVATE_IP>,Port=80
    
        # Modify the Listener to forward requests to the ECS Target Group
        aws elbv2 modify-listener \
        --listener-arn $(aws elbv2 describe-listeners --load-balancer-arn $LB_ARN --query 'Listeners[0].ListenerArn' --output text) \
        --default-actions Type=forward,TargetGroupArn=$TG_ARN
         ```

By understanding these core AWS services and how to use the AWS Management Console and CLI, you'll be well-equipped to manage your AWS resources effectively. This foundational knowledge will enable you to build, deploy, and manage applications in the AWS Cloud.
