# AWS Fundamentals Workshop

## Overview

This repository contains the materials for the AWS Fundamentals Workshop, designed to provide a foundational understanding of AWS services through hands-on labs. The workshop covers essential AWS services, key concepts, and best practices, ensuring participants gain practical skills to effectively utilize AWS for various projects.

## Workshop Structure

The workshop is divided into two main sections:

1. **Core Concepts**: An introduction to AWS, including key terminology, benefits, global infrastructure, and cloud computing models.

2. **Labs**: Hands-on exercises covering:
   - Introduction to AWS CDK
   - Identity and Access Management (IAM)
   - Networking and Security Groups
   - Basic AWS Services (EC2, S3, etc.)
   - Amazon RDS

Each lab includes step-by-step instructions and practical exercises using AWS CDK to deploy and configure resources.

## Prerequisites

- Basic understanding of programming concepts
- Familiarity with command-line interfaces
- AWS account (most resources are free tier, some small costs may apply)
- Node.js and npm installed

## Installation and Setup

1. Clone this repository to your local machine:

   ```sh
   git clone https://github.com/superluminar-io/serverless-workshop.git
   cd serverless-workshop
   ```

2. Install project dependencies:

   ```sh
   npm install
   ```

## Running the Development Server

To start the development server for the workshop documentation:

1. Run the following command:

   ```sh
   npm start
   ```

2. Open your browser and navigate to `http://localhost:3000` to view the workshop documentation.

## Getting Started with Labs

1. Follow the setup instructions in the `docs/labs/lab_1/1_setup_project.md` file to prepare your environment.
2. Progress through each lab in order, following the instructions provided in the respective markdown files.

## Workshop Duration

The total estimated time for completing all labs is 4-5 hours.

## Additional Resources

- Each lab contains an "Additional Resources" section for further learning.
- After completing the workshop, refer to `docs/2_next.md` for suggested next steps in your AWS learning journey.

## Deploying to S3 with CloudFront

To deploy this workshop documentation to an S3 bucket and serve it via CloudFront, follow these steps:

1. **Build the documentation**:
   Run the following command to build the documentation:

   ```sh
   npm run build
   ```

   This will create a `build` directory with the static files.

2. **Create an S3 bucket**:
   Create an S3 bucket to host your static files. Make sure to enable static website hosting for the bucket.

3. **Upload files to S3**:
   Upload the contents of the `build` directory to your S3 bucket.

4. **Create a CloudFront distribution**:
   Create a CloudFront distribution with the S3 bucket as the origin.

5. **Update DNS (optional)**:
   If you're using a custom domain, update your DNS settings to point to the CloudFront distribution.

After deployment, the workshop documentation will be available via the CloudFront distribution URL.
