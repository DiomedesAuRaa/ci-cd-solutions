# Jenkins Pipeline for Continuous Deployment

This directory contains a sample `Jenkinsfile` that implements a basic **Continuous Deployment (CD)** pipeline. The pipeline automates the process of building, testing, and deploying code to staging and production environments.

## Pipeline Stages

1. **Checkout**: Fetches the code from the specified Git repository and branch.
2. **Build**: Compiles or packages the code using a build script (`build.sh`).
3. **Test**: Runs unit and integration tests using a test script (`run-tests.sh`). If tests fail, the pipeline stops.
4. **Deploy to Staging**: Deploys the code to a staging environment using a deploy script (`deploy.sh`).
5. **Approve for Production**: Waits for manual approval before deploying to production.
6. **Deploy to Production**: Deploys the code to the production environment after approval.

## How to Use

1. **Set Up Jenkins**:
   - Install Jenkins on your server or use a cloud-based Jenkins instance.
   - Install the necessary plugins (e.g., Git, Pipeline, Credentials).

2. **Configure the Pipeline**:
   - Create a new pipeline job in Jenkins.
   - Point the job to this `Jenkinsfile` in your repository.
   - Configure credentials for cloud providers if needed.

3. **Add Scripts**:
   - Create `build.sh`, `run-tests.sh`, and `deploy.sh` scripts in your repository to handle the build, test, and deployment steps.

4. **Run the Pipeline**:
   - Trigger the pipeline manually or configure it to run automatically on code changes.

## Environment Variables

The pipeline uses the following environment variables that can be customized:

| Variable | Description | Default |
|----------|-------------|---------|
| `REPO_URL` | Git repository URL | `https://github.com/your-username/your-repo.git` |
| `BRANCH` | Git branch to build | `main` |
| `STAGING_ENV` | Staging environment name | `staging` |
| `PRODUCTION_ENV` | Production environment name | `production` |

## Example Scripts

### `build.sh`
```bash
#!/bin/bash
set -e
echo "Building the application..."
# Add your build commands here
# Example: npm install && npm run build
# Example: mvn clean package
echo "Build complete!"
```

### `run-tests.sh`
```bash
#!/bin/bash
set -e
echo "Running tests..."
# Add your test commands here
# Example: npm test
# Example: mvn test
echo "Tests complete!"
```

### `deploy.sh`
```bash
#!/bin/bash
set -e
ENVIRONMENT=$1

if [ -z "$ENVIRONMENT" ]; then
    echo "Error: Environment not specified"
    exit 1
fi

echo "Deploying to $ENVIRONMENT environment..."
# Add your deployment commands here
# Example: kubectl apply -f k8s/$ENVIRONMENT/
# Example: aws ecs update-service --cluster $ENVIRONMENT --service app
echo "Deployment to $ENVIRONMENT complete!"
```

## Required Jenkins Plugins

- **Git Plugin**: For source control management
- **Pipeline Plugin**: For pipeline as code
- **Credentials Plugin**: For managing secrets
- **Workspace Cleanup Plugin**: For cleaning up the workspace

## Troubleshooting

- **Pipeline stuck at approval**: Check that you have the necessary permissions to approve deployments
- **Build script fails**: Ensure scripts have execute permissions (`chmod +x *.sh`)
- **Credential errors**: Verify credentials are properly configured in Jenkins credential store