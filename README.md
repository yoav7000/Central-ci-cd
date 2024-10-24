
# Central GitHub Actions Reusable Workflows
This repository provides reusable workflows for deploying applications that follow GitOps principles to multiple environments: **development**, **staging**, and **production**. The deployment process interacts with the application defined GitOps repository, updating the image tag in the application values file to the newly created image tag during the deployment. Deployment jobs in the Workflow are controlled using GitHub Action environments, allowing for fine-grained control and approval processes.

![ci-cd-wf](https://miro.medium.com/v2/resize:fit:1400/1*jJTE__BrpO2H6fmWf5VSDQ.png)

This repo contains two main workflows:

1. **Development and Staging CI/CD Workflow**:
   - Runs on pull requests to the `main` branch.
     - **Build the Docker image** for the application.
     - **Run tests** to validate the application.
     - **Trivy security scan** for vulnerabilities.
     - Deploys to **dev** and **staging** environments (using GitHub Action environments) after manual approval.
   
 ![Screenshot 2024-10-16 002037](https://github.com/user-attachments/assets/dabf2510-508a-474c-806e-9676c436caa1)

2. **Production CI/CD Workflow**:
   - Runs when a Git tag is created for the `main` branch.
     - **Build the Docker image** using the Git tag.
     - Deploys to **production** (using GitHub Action environments) after manual approval.

## Prerequisites

To use these workflows, ensure you have the following secrets and inputs defined in your repository settings:

- **Secrets**:
  - `DOCKERHUB_USERNAME`: DockerHub account username.
  - `DOCKERHUB_PASSWORD`: DockerHub account password.
  - `GITOPS_REPO_TOKEN`: Access token for your GitOps repository.

- **Inputs**:
  - `app_name`: The name of the application being deployed.
  - `gitops_repo`: The name of your GitOps repository where the application deployment is defined.
  - `gitops_values_path`: Path to the values file in your GitOps repository (e.g., `environments/{ENV_NAME}/values/backend-microservice-values.yaml`).

The `{ENV_NAME}` placeholder in the `gitops_values_path` is a convention used to dynamically insert the actual environment name (e.g., `dev` or `staging`) based on the environment being deployed.
### Additional Requirements
   - The application deployment must be controlled in an existing GitOps repository.
   - The application repository must contain a Dockerfile for building the Docker image.

## How to Use

### 1. Dev and Staging CI/CD Workflow

To configure the **development** and **staging** deployment pipeline, add the following to your repository’s workflow configuration:

```yaml
name: Dev and Staging CI/CD Workflow

on:
  pull_request:
    branches:
      - main

jobs:
  dev-staging-workflow:
    uses: yoav7000/Central-ci-cd/.github/workflows/dev-staging-feature-branch-wf.yaml@main
    with:
      app_name: backend-microservice
      gitops_values_path: environments/{ENV_NAME}/values/backend-microservice-values.yaml
      gitops_repo: ${{ vars.GITOPS_REPO }}
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_PASSWORD: ${{ secrets.DOCKERHUB_PASSWORD }}
      GITOPS_REPO_TOKEN: ${{ secrets.GITOPS_REPO_TOKEN }}
```

This workflow will automatically trigger upon creating a pull request to the `main` branch. After manual approval through GitHub Action environments, it will deploy the app to the **dev** and **staging** environments.

### 2. Production CI/CD Workflow

To configure the **production** deployment pipeline, create a Git tag (e.g., `v1.0.0`). Use the following configuration in your repository:

```yaml
name: Production CI/CD Workflow

on:
  push:
    tags:
      - 'v*'

jobs:
  prod-workflow:
    uses: yoav7000/Central-ci-cd/.github/workflows/prod-wf.yaml@main
    with:
      app_name: backend-microservice
      gitops_values_path: environments/{ENV_NAME}/values/backend-microservice-values.yaml
      gitops_repo: ${{ vars.GITOPS_REPO }}
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_PASSWORD: ${{ secrets.DOCKERHUB_PASSWORD }}
      GITOPS_REPO_TOKEN: ${{ secrets.GITOPS_REPO_TOKEN }}
```

This workflow will trigger upon tagging the `main` branch with a version tag starting with `v` (e.g., `v1.0.0`). After manual approval through GitHub Action environments, it will deploy the app to **production**.

### Manual Review Setup

While not mandatory, setting up manual review is **highly recommended** to control deployments and ensure they are supervised. To configure manual approval, go to **Settings** -> **Environments** in GitHub, and define required reviews under the "Required reviewers" field for each environment (e.g., `dev`, `staging`, `production`). This allows for an extra layer of control, ensuring deployments are reviewed and authorized by someone before proceeding.

## Conclusion

This repository provides a streamlined CI/CD process for deploying applications across multiple environments, integrating Docker image builds, trivy security scans, and GitOps principles. Make sure to configure your secrets and inputs properly for smooth execution.
