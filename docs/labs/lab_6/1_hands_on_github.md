# Lab 6: Operations & Troubleshooting

## Setting Up CI/CD with GitHub Actions

### 1. Create a GitHub Repository:
   - Initialize a new GitHub repository for your CDK application.
   - Push your CDK application code to the repository.


### 2. Define the Workflow:

Create a `.github/workflows/deploy.yml` file in your repository
   
```bash
     mkdir -p .github/workflows
     touch .github/workflows/deploy.yml
```

Add the following workflow configuration example to the `deploy.yml` file:

```yaml
name: Deploy CDK Application

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '20'
          
      - name: Set up AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: eu-central-1 # Frankfurt

      - name: Install dependencies
        run: npm install

      - name: [Optional] Check for Changes
        run: npx cdk diff

      - name: Deploy to AWS
        run: npx cdk deploy --require-approval never
``` 

**Explanation of the code:**

The workflow configuration file defines the following steps for deployment:
- **Checkout**: Check out the repository code.
- **Setup Node.js**: Set up the Node.js environment.
- **Install Dependencies**: Install the necessary dependencies with `npm install`.
- **Check for Changes**: [optional step] Check for changes in the CloudFormation stack using `npx cdk diff`.
- **Deploy**: Deploy the CDK application to AWS with `npx cdk deploy`. `--require-approval never` flag is used to automatically approve the deployment.

By using `push.branches: [main]` we ensure, that this workflow will only trigger when code is pushed to the `main` branch.

### 3. Secrets Configuration:
   - Add the following secrets to your GitHub repository under `Settings > Secrets and variables > Actions`:
        - `AWS_ACCESS_KEY_ID`
        - `AWS_SECRET_ACCESS_KEY`

### 4. Deploy the Workflow
   - Push the workflow file to the repository.
   - Verify that the workflow triggers on code changes and successfully deploys the application.

You've successfully set up the automated deployment workflow for your CDK application using GitHub Actions. This workflow will trigger on code changes and deploy the application to AWS.

Let's take a look at some common operations and troubleshooting techniques for CDK applications next.