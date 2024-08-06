# Lab 4: Core AWS Services

In this lab, you will gain an understanding of some of the core AWS services, including EC2, S3, and RDS, and learn how to navigate and use the AWS Management Console and Command Line Interface (CLI).

## Introduction to Core Services

AWS offers a wide range of cloud services, but some of the core services that are fundamental to most applications include EC2, S3, and RDS. Let's take a closer look at each of these services.

## Amazon EC2 (Elastic Compute Cloud)

Amazon EC2 provides scalable computing capacity in the AWS Cloud. Using EC2 eliminates your need to invest in hardware upfront, so you can develop and deploy applications faster. You can use Amazon EC2 to launch as many or as few virtual servers as you need, configure security and networking, and manage storage. EC2 allows you to scale up or down to handle changes in requirements or spikes in popularity, reducing your need to forecast traffic.

Key features:

- **Instances**: Virtual servers that run applications.
- **AMI (Amazon Machine Image)**: Preconfigured templates for instances.
- **Instance Types**: Various configurations of CPU, memory, storage, and networking capacity.
- **Security Groups**: Firewall rules that control traffic to instances.
- **Elastic IPs**: Static IP addresses for dynamic cloud computing.

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

   - To launch an EC2 instance: Navigate to the EC2 service, select "Instances", click "Launch Instance", and follow the steps.
   - To create an S3 bucket: Navigate to the S3 service, click "Create bucket", and follow the prompts.
   - To set up an RDS database: Navigate to the RDS service, click "Create database", and choose your database engine and settings.

2. **AWS CLI**:
   - To launch an EC2 instance:
     ```bash
     aws ec2 run-instances --image-id ami-0abcdef1234567890 --count 1 --instance-type t2.micro --key-name MyKeyPair --security-group-ids sg-0123456789abcdef0 --subnet-id subnet-6e7f829e
     ```
   - To create an S3 bucket:
     ```bash
     aws s3 mb s3://my-bucket
     ```
   - To set up an RDS database:
     ```bash
     aws rds create-db-instance --db-instance-identifier mydatabase --allocated-storage 20 --db-instance-class db.t2.micro --engine mysql --master-username admin --master-user-password password --backup-retention-period 3
     ```

By understanding these core AWS services and how to use the AWS Management Console and CLI, you will be well-equipped to manage your AWS resources effectively. This foundational knowledge will enable you to build, deploy, and manage applications in the AWS Cloud.
