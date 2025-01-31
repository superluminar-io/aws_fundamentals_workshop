# Lab 6: Operations & Troubleshooting

## Setting Up CI/CD with GitHub Actions

1. **Create a GitHub Repository:**
   - Initialize a new GitHub repository for your CDK application.
   - Push your CDK application code to the repository.

2. **Define the Workflow:**
   - Create a `.github/workflows/deploy.yml` file in your repository with the following stages:
      - **Checkout**: Check out the repository code.
      - **Setup Node.js**: Set up the Node.js environment.
      - **Install Dependencies**: Install the necessary dependencies.
      - **Check Changes**: Build and synthesize the CDK application.
      - **Deploy**: Deploy the synthesized CloudFormation template using AWS CLI.

3. **Configure the Workflow:**
   - Ensure the workflow includes steps to install dependencies, build the application, and deploy the CloudFormation template.

4. **Secrets Configuration:**
   - Add the following secrets to your GitHub repository under `Settings > Secrets and variables > Actions`:
        - `AWS_ACCESS_KEY_ID`
        - `AWS_SECRET_ACCESS_KEY`

5. **Deploy the Workflow**:
   - Push the workflow file to the repository.
   - Verify that the workflow triggers on code changes and successfully deploys the application.

### Example GitHub Actions Workflow

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

      - name: Install dependencies
        run: npm install

      - name: Synthesize CloudFormation template
        run: npx cdk synth

      - name: Check for changes
        run: npx cdk diff

      - name: Deploy to AWS
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: eu-central-1
        run: npx cdk deploy --require-approval never
``` 

You've successfully set up the automated deployment workflow for your CDK application using GitHub Actions. This workflow will trigger on code changes and deploy the application to AWS.