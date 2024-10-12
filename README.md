# Central-ci-cd
# CI/CD Reusable GitHub Actions Workflows

This repository contains reusable GitHub Actions workflows for deploying any application across development, staging, and production environments using GitOps principles. These pipelines provide a standardized CI/CD process that can be easily integrated into any project.

## Workflow Files

- **`dev-staging.yml`**: This file defines the workflow for development and staging environments.
- **`production.yml`**: This file defines the workflow for the production environment.

## Pipelines Overview

### Development and Staging Workflow (`dev-staging.yml`)
This pipeline is designed to automate the build, testing, security scanning, and deployment to development and staging environments. The steps are:

1. **Build the Docker image** for the application.
2. **Run tests** to validate the application.
3. **Trivy security scan** for vulnerabilities.
4. **Deploy to development environment** (deployment will be defined in your app repository). After a manual review, you can proceed to the next stage.
5. **Deploy to staging environment** (defined in your app repository). After a manual review, deployment to staging will proceed.

### Production Workflow (`production.yml`)
This pipeline focuses on deploying your application to production based on a Git tag release. The steps are:

1. **Build the Docker image** using the Git tag.
2. **Deploy to production environment** (deployment configuration is in your app repository).

## Required Inputs

When calling these reusable workflows in your project, you will need to provide the following inputs:

- **App Name**: The name of the application being deployed.
- **Path to the Values File**: The path to the environment-specific values file in your GitOps repository.
- **Docker Hub Username and Password**: Credentials for pushing the Docker image to Docker Hub.
- **GitOps Repo Name**: The name of the GitOps repository where the deployments are managed.
- **GitOps Repo Token**: A token that grants access to your GitOps repository for updating deployments.

## How to Use

To use these workflows in your project, simply reference the workflows from this repository in your GitHub Actions YAML file. You’ll need to pass the required inputs for each environment, ensuring your deployments follow the defined GitOps principles.

```yaml
jobs:
  deploy:
    uses: <repo-name>/.github/workflows/dev-staging.yml@main
    with:
      app_name: <your-app-name>
      values_file: <path-to-your-values-file>
      docker_username: ${{ secrets.DOCKER_USERNAME }}
      docker_password: ${{ secrets.DOCKER_PASSWORD }}
      gitops_repo: <your-gitops-repo>
      gitops_token: ${{ secrets.GITOPS_TOKEN }}
