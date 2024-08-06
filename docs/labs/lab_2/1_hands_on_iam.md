# Lab 2: Identity and Access Management (IAM)

## Set up IAM Roles and Policies Using CDK

In this hands-on section, you will create an S3 bucket with a destroy policy, a Lambda function that writes a "Hello World" file to the bucket, and IAM roles and policies to manage permissions. This exercise will guide you through defining IAM roles and policies in code, deploying them to your AWS account, and verifying the setup.

**What is Amazon S3?**

Amazon Simple Storage Service (Amazon S3) is an object storage service that offers industry-leading scalability, data availability, security, and performance. Customers of all sizes and industries can use S3 to store and protect any amount of data for a range of use cases, such as websites, mobile applications, backup and restore, archive, enterprise applications, IoT devices, and big data analytics.

**What is AWS Lambda?**

AWS Lambda is a serverless compute service that lets you run code without provisioning or managing servers. Lambda runs your code only when needed and scales automatically, from a few requests per day to thousands per second. You can use Lambda to run code for virtually any type of application or backend service, all with zero administration.

## Principle of Least Privilege

The principle of least privilege is a fundamental concept in security that states that a user, program, or process should have only the bare minimum privileges necessary to perform its function. In the context of IAM, this means granting only the permissions required for a specific task or role, and nothing more. This practice helps to minimize the potential damage from errors or malicious actions.

## IAM Components Relationship

Here's a diagram showing the relationship between IAM users, roles, and policies:

```
[Insert a diagram here showing IAM users, roles, and policies]
```

## Create IAM Roles, Policies, and Resources

1. **Set Up Your CDK Project**

   Install the necessary dependencies with the following commands:

   ```bash
   npm install @aws-cdk/aws-iam @aws-cdk/aws-lambda @aws-cdk/aws-s3 @aws-cdk/aws-s3-deployment
   ```

2. **Create an S3 Bucket and Lambda Function with Incorrect Permissions**

   Open the stack file located in the `lib` directory (e.g., `lib/my-cdk-app-stack.ts` for a TypeScript project). Add the following code:

   ```typescript
   import * as cdk from "aws-cdk-lib";
   import { Construct } from "constructs";
   import * as iam from "aws-cdk-lib/aws-iam";
   import * as lambda from "aws-cdk-lib/aws-lambda";
   import * as s3 from "aws-cdk-lib/aws-s3";

   export class MyCdkAppStack extends cdk.Stack {
     constructor(scope: Construct, id: string, props?: cdk.StackProps) {
       super(scope, id, props);

       // Create an S3 bucket with a destroy policy
       const bucket = new s3.Bucket(this, "MyBucket", {
         removalPolicy: cdk.RemovalPolicy.DESTROY,
         autoDeleteObjects: true,
       });

       // Define an IAM Role for Lambda
       const lambdaRole = new iam.Role(this, "LambdaRole", {
         assumedBy: new iam.ServicePrincipal("lambda.amazonaws.com"),
       });

       // Incorrect IAM Policy (missing s3:PutObject permission)
       const incorrectPolicy = new iam.Policy(this, "IncorrectPolicy", {
         statements: [
           new iam.PolicyStatement({
             actions: ["s3:GetObject"],
             resources: [bucket.bucketArn + "/*"],
           }),
         ],
       });

       // Attach the incorrect policy to the Lambda role
       lambdaRole.attachInlinePolicy(incorrectPolicy);

       // Create a Lambda function
       const lambdaFunction = new lambda.Function(this, "MyLambda", {
         runtime: lambda.Runtime.NODEJS_14_X,
         handler: "index.handler",
         code: lambda.Code.fromInline(`
           const AWS = require('aws-sdk');
           const s3 = new AWS.S3();
           exports.handler = async function(event) {
             const params = {
               Bucket: process.env.BUCKET_NAME,
               Key: 'hello.txt',
               Body: 'Hello World',
               ContentType: 'text/plain',
             };
             await s3.putObject(params).promise();
             return {
               statusCode: 200,
               body: 'File written!',
             };
           };
         `),
         environment: {
           BUCKET_NAME: bucket.bucketName,
         },
         role: lambdaRole,
       });

       // Output the bucket name
       new cdk.CfnOutput(this, "BucketName", {
         value: bucket.bucketName,
       });

       // Output the Lambda function ARN
       new cdk.CfnOutput(this, "LambdaArn", {
         value: lambdaFunction.functionArn,
       });
     }
   }
   ```

3. **Deploy the Stack with Incorrect Permissions**

   Deploy the stack:

   ```bash
   cdk deploy
   ```

   This deployment will fail when the Lambda function tries to write to the S3 bucket because it does not have the necessary `s3:PutObject` permission.

4. **Update the IAM Policy with Correct Permissions**

   Update the stack file to correct the IAM policy by adding the `s3:PutObject` permission:

   ```typescript
   // Correct IAM Policy
   const correctPolicy = new iam.Policy(this, "CorrectPolicy", {
     statements: [
       new iam.PolicyStatement({
         actions: ["s3:GetObject", "s3:PutObject"],
         resources: [bucket.bucketArn + "/*"],
       }),
     ],
   });

   // Attach the correct policy to the Lambda role
   lambdaRole.attachInlinePolicy(correctPolicy);
   ```

5. **Deploy the Stack with Correct Permissions**

   Deploy the stack again with the correct permissions:

   ```bash
   cdk deploy
   ```

   This deployment will succeed, and the Lambda function will be able to write the "Hello World" file to the S3 bucket.

6. **Verify the Deployment**

   To verify the deployment:

   - Open the AWS Management Console.
   - Navigate to the S3 service and find the bucket created by the stack.
   - Check the bucket contents for a file named `hello.txt` with the content "Hello World".

7. **Tear Down the Resources**

   To clean up and delete the resources created by this stack, run:

   ```bash
   cdk destroy
   ```

   This command will delete the S3 bucket, Lambda function, and IAM roles created by the stack.

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

Well done! You've navigated through creating and managing IAM roles and policies using AWS CDK, explored how permissions affect resource access, and confirmed the setup by writing a file to an S3 bucket with a Lambda function. Your environment is now prepared for the next lab.
