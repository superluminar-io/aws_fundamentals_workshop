# AWS Basics and Core Services Workshop

## Project Overview

The purpose of this workshop is to provide a foundational understanding of AWS services and to guide participants through hands-on labs that will solidify their learning. We will cover essential AWS services, key concepts, and best practices, ensuring that you have the knowledge and skills required to effectively utilize AWS for various projects. The agenda for this workshop follows:

### Core Concepts

#### 1. Overview of AWS

- **Introduction to AWS and project overview**
- **Key concepts and terminology**
- **Benefits of using AWS**

#### 2. Global AWS Infrastructure

- **Detailed look at AWS regions and availability zones**
- **Understanding edge locations and AWS Global Accelerator**

#### 3. Cloud Computing Models (IaaS, PaaS, SaaS)

- **Understanding the different cloud computing models**
- **Use cases and examples for IaaS, PaaS, and SaaS**

### Labs

#### 1. Introduction to AWS CDK

- **Overview of AWS CDK**
- **Setting up the AWS CDK environment**
- **Basic concepts: stacks, constructs, and apps**
- **Hands-On: Set up the initial project environment using CDK**
  - Install AWS CDK and set up a new CDK project
  - Create a basic stack

#### 2. Identity and Access Management (IAM)

- **Introduction to IAM**
- **More details on IAM best practices and security implications**
- **Best practices for managing IAM**
- **Hands-On: Set up IAM roles and policies using CDK**
  - Create IAM roles and policies

#### 3. Networking and Security Groups

- **Overview of Networking and Security**
- **Setting up a VPC for the project using CDK**
- **Configuring security groups and network ACLs**
- **Hands-On: Set up VPC and security groups using CDK**
  - Define a VPC with subnets and route tables
  - Configure security groups for the EC2 and RDS instances

#### 4. Basic AWS Services

- **Introduction to core services (EC2, S3, RDS, etc.)**
- **Understanding the AWS Management Console and CLI**
- **Hands-On: Deploy basic services using CDK**
  - Use CDK to create an S3 bucket and an EC2 instance

#### 5. Amazon RDS

- **Introduction to Amazon RDS**
- **Choosing the right RDS instance type for your application**
- **Configuring and managing RDS instances**
- **Best practices for RDS performance and cost optimization**
- **Hands-On: Set up RDS for relational data storage using CDK**
  - Create an RDS instance with CDK
  - Configure security groups and database parameters
