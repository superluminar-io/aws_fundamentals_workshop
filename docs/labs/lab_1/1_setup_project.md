# Lab 1: Introduction to AWS CDK

This lab introduces you to AWS CDK (Cloud Development Kit) and guides you through setting up your initial project environment. You'll learn how to install the CDK toolkit, create a new project, and deploy a simple stack.

## Set up the Initial Project Environment Using CDK

**Install AWS CDK and Set Up a New CDK Project**

To get started with AWS CDK, you need to install the AWS CDK Toolkit and set up a new CDK project. Follow these steps to set up your initial project environment:

1. **Install the AWS CDK Toolkit**:
   If you haven't already installed the AWS CDK Toolkit, you can do so using npm. Open your terminal and run the following command:

   ```bash
   npm install -g aws-cdk
   ```

   This command installs the CDK Toolkit globally on your machine, making the `cdk` command available from any directory.

   To verify the installation, run:

   ```bash
   cdk --version
   ```

2. **Create a New CDK Project**:
   Navigate to the directory where you want to create your new CDK project. We'll name our project `aws-fundamentals-workshop`. Run the following command to initialize a new CDK application:

   ```bash
   cdk init app --language=typescript --name=aws-fundamentals-workshop
   ```

   This command sets up a new CDK project named `aws-fundamentals-workshop` with a basic directory structure and necessary configuration files.

   After the project is created, change into the project directory:

   ```bash
   cd aws-fundamentals-workshop
   ```

## Lab Architecture

Before we proceed, let's take a look at the architecture we'll be building in this lab:

![Initial Setup Lab Architecture](media/lab_1_arch.drawio.svg)

This diagram illustrates the key components of our lab:

1. A CDK application that defines our infrastructure as code.
2. A CloudFormation stack that will be generated from our CDK code.
3. A simple CloudFormation output that we'll create to verify our setup.

This basic architecture allows us to demonstrate the CDK deployment process and ensure our environment is correctly set up for the subsequent labs.

## Configure AWS Credentials with SSO

If you are using AWS Single Sign-On (SSO) to manage your AWS credentials, follow these steps to configure your AWS CLI and CDK to use SSO:

1. **Install and Configure AWS CLI**:
   Ensure you have the AWS CLI version 2 installed. If not, you can download it from the [AWS CLI version 2 installation guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html). Once installed, configure the AWS CLI to use SSO:

   ```bash
   aws configure sso
   ```

   Follow the prompts to enter your SSO start URL, region, SSO user name, and other details. This setup allows the AWS CLI to use your SSO credentials.

2. **Login with SSO**:
   After configuring SSO, log in using your SSO credentials:
   ```bash
   aws sso login
   ```
   This command opens a browser window where you can log in with your SSO credentials. Once logged in, the AWS CLI and CDK will use the SSO session for authentication.

## Create a Basic Stack with a CloudFormation Output

After setting up the project and configuring SSO, you can define your first stack. A stack in CDK is a collection of AWS resources that you manage as a single unit. Follow these steps to create a basic stack that outputs a simple message:

1. **Open the Stack File**:
   Open the stack file located in the `lib` directory. The stack file is named based on your project name, for example, `lib/my-cdk-app-stack.ts` for a TypeScript project.

2. **Define a CloudFormation Output**:
   Edit the stack file to define a CloudFormation output. Below is an example of a basic stack that creates a CloudFormation output using TypeScript:

   ```typescript
   import * as cdk from "aws-cdk-lib";
   import { Construct } from "constructs";

   export class MyCdkAppStack extends cdk.Stack {
     constructor(scope: Construct, id: string, props?: cdk.StackProps) {
       super(scope, id, props);

       // Define a CloudFormation output
       new cdk.CfnOutput(this, "MyOutput", {
         value: "Hello, CDK!",
         description: "A simple CloudFormation output to prove the setup works",
       });
     }
   }
   ```

3. **Deploy the Stack**:
   To deploy the stack to your AWS account, run the following command from the root directory of your CDK project:

   ```bash
   cdk deploy
   ```

   This command synthesizes the CloudFormation template from your CDK code and deploys the stack, creating the specified CloudFormation output in your account.

4. **Verify the Deployment**:
   After the deployment is complete, you can verify the CloudFormation output using the AWS Management Console:
   - Open the AWS Management Console.
   - Navigate to the CloudFormation service.
   - Find and select the stack you just deployed.
   - In the stack details, go to the "Outputs" tab to see the output value "Hello, CDK!".

## Checkpoint

At this point, you should have:

- Installed the AWS CDK Toolkit
- Created a new CDK project
- Defined a basic stack with a CloudFormation output
- Successfully deployed the stack using `cdk deploy`
- Verified the output in the AWS CloudFormation console

If you're encountering issues, check the following:

- Ensure Node.js and npm are correctly installed and in your PATH
- Verify that you have the latest version of AWS CDK installed
- Check that your AWS credentials are properly configured
- Make sure you have the necessary permissions to create CloudFormation stacks

## Best Practices and Security Considerations

1. Use IAM roles instead of storing AWS credentials on EC2 instances.
2. Regularly update your CDK version to benefit from the latest features and security improvements.
3. Use version control (like Git) to track changes to your CDK code.
4. Implement the principle of least privilege when defining IAM roles and policies.
5. Use CDK's built-in security checks to identify potential security issues in your infrastructure.

## Reset the Stack for the Next Lab

To ensure the environment is clean for the next lab, follow these steps to delete the stack and clean up your project:

1. **Delete the Stack**:
   To delete the stack from your AWS account, run the following command from the root directory of your CDK project:

   ```bash
   cdk destroy
   ```

   Confirm the deletion when prompted. This command removes all the resources defined in your stack from your AWS account.

2. **Clean Up the Stack File**:
   Open the stack file in the `lib` directory and remove the code you added. Your stack file should look like this after cleaning up:

   ```typescript
   import * as cdk from "aws-cdk-lib";
   import { Construct } from "constructs";

   export class MyCdkAppStack extends cdk.Stack {
     constructor(scope: Construct, id: string, props?: cdk.StackProps) {
       super(scope, id, props);

       // The stack is empty for the next lab
     }
   }
   ```

Great work! You've set up your initial project environment using AWS CDK, deployed a basic stack, and cleaned up the environment for the next lab. This process has given you a strong starting point for building more complex cloud applications with AWS CDK.
