# Organization infrastructure & automated deployments

Welcome! This repository documents and standardizes the continuous integration and continuous deployment (CI/CD) pipeline used across all applications within the organization.

Every repository in this organization is automatically built and deployed to **Google Cloud Run** using GitHub Actions and Workload Identity Federation (WIF).

## Architecture Overview

1. **Trigger & Tenant Extraction**:
   - Pushing a version tag (`v*`) or dispatching the workflow manually triggers the deployment.
   - The workflow dynamically extracts the `ZEN_TENANT` name from the repository's name (e.g., repository `sample.custom.zenerp.app.br` maps to tenant `sample`).
2. **Secure Authentication (OIDC / WIF)**:
   - GitHub Actions exchanges short-lived OIDC tokens directly with Google Cloud via Workload Identity Federation, eliminating the need to store long-lived service account keys.
3. **Container Build & Registry**:
   - Docker images are built via Buildx and pushed directly to Google Artifact Registry (`southamerica-east1-docker.pkg.dev/custom-zenerp/cloud-run-source-deploy/...`).
4. **Cloud Run Deployment**:
   - The application is deployed to Google Cloud Run in the `southamerica-east1` region.
   - Injected runtime environment variables include `ZEN_TENANT`, alongside secure secrets fetched directly from **Google Secret Manager** (`zen-token`).
5. **Smoke Testing**:
   - A final post-deployment `curl` verification checks the root endpoint of the newly provisioned Cloud Run service URL.

## Standard Deployment Workflow (`deploy.yml`)

Add the `deploy.yml` workflow file to `.github/workflows/deploy.yml` in any new repository within the organization:
