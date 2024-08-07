# Lab 1: Introduction to AWS CDK

This lab provides an introduction to AWS Cloud Development Kit (CDK), covering its core concepts, setup process, and advantages over other AWS deployment methods.

## Overview of AWS CDK

The AWS Cloud Development Kit (CDK) is an open-source software development framework that allows developers to define cloud infrastructure using familiar programming languages such as TypeScript, JavaScript, Python, Java, and C#. By leveraging the power of these high-level programming languages, AWS CDK enables developers to create reusable cloud components, called constructs, which can be composed together to build complex cloud applications. The CDK abstracts the intricate details of AWS CloudFormation, making it easier to provision and manage AWS resources.

**Setting up the AWS CDK Environment**

To begin using AWS CDK, you'll need to set up your development environment. This involves several key steps:

1. **Install Node.js and npm**: The AWS CDK Toolkit is available as an npm package, so you need Node.js and npm installed on your machine. You can download and install Node.js from the official Node.js website.

2. **Install the AWS CDK Toolkit**: Once Node.js is installed, you can install the AWS CDK Toolkit globally using npm:

   ```bash
   npm install -g aws-cdk
   ```

   To verify the installation, run:

   ```bash
   cdk --version
   ```

   If you encounter permission errors, you may need to use sudo:

   ```bash
   sudo npm install -g aws-cdk
   ```

   For detailed installation instructions and troubleshooting, refer to the [official AWS CDK documentation](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_install).

   **Note:** Make sure to install the latest version of AWS CDK (v2) for the best experience.

   Before using the CDK CLI, ensure you have:

   1. [Installed the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
   2. [Configured your AWS credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)

3. **Configure AWS Credentials**: AWS CDK requires credentials to interact with your AWS account. You can configure your AWS credentials using the AWS CLI. If you haven't already installed the AWS CLI, you can download it from the AWS CLI installation guide. After installing, configure your credentials:

   If you're using AWS Single Sign-On (SSO), also known as AWS IAM Identity Center, you'll need to configure the AWS CLI to use SSO authentication. Here's how if you haven't done so already:

   1. Run the following command:

      ```bash
      aws configure sso
      ```

   2. You'll be prompted to enter the following information:

      - SSO start URL (https://your-domain.awsapps.com/start)
      - SSO Region (e.g., us-east-1)
      - SSO registration scopes (default is fine for most cases)

   3. The CLI will then open a browser window for you to log in to your SSO account.

   4. After successful login, you'll be asked to choose an AWS account and role to use.

   5. Finally, you'll be asked to name this profile. Choose a name that's meaningful to you.

   After this setup, you can use the profile by specifying it when running AWS CLI commands:

   ```bash
   aws s3 ls --profile your-profile-name
   ```

   Or set it as your default profile:

   ```bash
   export AWS_PROFILE=your-profile-name
   ```

   This method is more secure as it doesn't require long-lived access keys to be stored on your machine.

4. **Bootstrapping Your AWS Environment**: Before deploying your first CDK app, you need to bootstrap your AWS environment. Bootstrapping sets up the necessary resources that the AWS CDK needs to perform deployments (like an S3 bucket for storing files and an ECR repository for container images). To bootstrap, run the following command:
   ```bash
   cdk bootstrap aws://ACCOUNT-NUMBER/REGION
   ```
   Replace `ACCOUNT-NUMBER` with your AWS account number and `REGION` with your desired AWS region (e.g., `eu-central-1`).

**Basic Concepts: Stacks, Constructs, and Apps**

Understanding the core concepts of AWS CDK is crucial to effectively using the framework:

1. **Stacks**:

   - A stack is the fundamental deployment unit in AWS CDK. It represents a collection of AWS resources that you can manage as a single unit. When you deploy a stack, AWS CDK generates a CloudFormation template and uses it to provision and manage the resources defined in the stack.
   - Stacks can be deployed, updated, and deleted, and they allow you to encapsulate and manage all the resources required for a specific application or environment.

2. **Constructs**:

   - Constructs are the basic building blocks of AWS CDK applications. They are reusable cloud components that encapsulate AWS resources and their configurations. Constructs can range from low-level constructs, which directly represent AWS resources (like an S3 bucket or an EC2 instance), to high-level constructs, which represent complex architectures (like a complete web application stack).
   - Constructs can be composed together to form more complex constructs, promoting reuse and reducing the need to write boilerplate code.

3. **Apps**:
   - An app in AWS CDK serves as a container for one or more stacks. It defines the scope of deployment and orchestrates the lifecycle of the stacks it contains.
   - An app is instantiated in your main entry point file (e.g., `app.ts` or `app.py`), and you can define multiple stacks within the app to organize your resources logically and manage dependencies between them.

## Quick Explainer on AWS CloudFormation

AWS CloudFormation is the underlying service that AWS CDK uses to provision and manage AWS resources. CloudFormation allows you to define your infrastructure as code using JSON or YAML templates. These templates describe the resources and their configurations, and CloudFormation handles the provisioning, updating, and deletion of these resources in a predictable and orderly manner.

When you use AWS CDK, you write your infrastructure code in a high-level programming language, which the CDK then synthesizes into a CloudFormation template. This template is then used by CloudFormation to deploy the specified resources. This approach provides several benefits:

- **Infrastructure as Code (IaC)**: Allows you to version control your infrastructure and apply the same software engineering practices to your infrastructure code as you do with your application code.
- **Repeatability**: Ensures that your infrastructure is deployed in a consistent manner, reducing the chances of human error.
- **Automation**: Enables automation of resource provisioning and management, making it easier to scale and manage complex environments.

## State Management

AWS CloudFormation manages the state of your infrastructure by maintaining a record of the resources it has provisioned. This state management ensures that changes to your infrastructure are tracked and managed correctly. When you update a stack, CloudFormation compares the desired state (defined in your template) with the current state and makes the necessary changes to achieve the desired state. This process helps prevent configuration drift and ensures that your infrastructure remains consistent and predictable.

## Why CDK?

There are many options for deploying resources in AWS, including third party tools as well, let's look at some of the core AWS provided options.

**Comparison: AWS CDK vs. AWS Management Console, CLI, SDKs, and CloudFormation**

- **AWS CDK**: Use AWS CDK when you want to define your infrastructure using familiar programming languages and leverage the power of reusable constructs. CDK is ideal for developers who prefer writing code over manually configuring resources. It integrates well with existing development workflows and tools, supports testing, and enables version control of infrastructure code. AWS CDK also abstracts many of the complexities involved in writing raw CloudFormation templates, making infrastructure as code more accessible and easier to manage. Additionally, CDK offers CDK Pipelines, a high-level construct library that makes it easy to set up continuous delivery pipelines for your CDK applications.

- **AWS Management Console**: The AWS Management Console is a user-friendly graphical interface that is suitable for simple and ad-hoc tasks. It is ideal for beginners or for situations where you need to quickly configure or monitor resources. However, it is less suitable for managing large or complex environments due to its manual nature.

- **AWS CLI**: The AWS Command Line Interface (CLI) is a powerful tool for scripting and automation. It provides fine-grained control over AWS services and is suitable for tasks that require automation or are part of a larger workflow. The CLI is ideal for users who are comfortable with command-line interfaces and need to manage resources programmatically.

- **AWS SDKs**: AWS SDKs provide programmatic access to AWS services in various programming languages. They are ideal for integrating AWS services into custom applications and for automating complex workflows. SDKs are suitable for developers who need to interact with AWS services from within their application code.

- **AWS CloudFormation**: CloudFormation allows you to define your infrastructure as code using JSON or YAML templates. It is ideal for users who want to maintain infrastructure definitions in a declarative format and benefit from CloudFormation's robust state management and orchestration capabilities. CloudFormation is suitable for both developers and operations teams who need to manage infrastructure in a repeatable and consistent manner. However, writing and managing large CloudFormation templates can be complex and error-prone. AWS CDK simplifies this process by allowing you to use higher-level constructs and familiar programming languages to define your infrastructure.

By understanding these concepts and setting up the AWS CDK environment, you can start leveraging the power of AWS CDK to define and manage your cloud infrastructure programmatically. In the next section, we will dive into a hands-on exercise to set up an initial project environment using AWS CDK.
