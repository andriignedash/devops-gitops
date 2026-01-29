# Django App GitOps Repository

This repository serves as the **GitOps source of truth** for the Django application deployed to Amazon EKS.

## Overview

This repository contains Helm charts that define the desired state of the Django application in Kubernetes. Changes to this repository are automatically detected and applied by Argo CD.

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CI/CD Pipeline Flow                          │
│                                                                       │
│  1. Developer pushes code to devops repo (lesson-8-9 branch)        │
│  2. Jenkins pipeline builds Docker image with Kaniko                │
│  3. Image is pushed to Amazon ECR                                   │
│  4. Jenkins updates image.tag in this repository (values.yaml)      │
│  5. Argo CD detects changes and auto-syncs to EKS                   │
│  6. Application is deployed/updated in Kubernetes                   │
└─────────────────────────────────────────────────────────────────────┘
```

## Repository Structure

```
devops-gitops/
├── README.md
└── charts/
    └── django-app/
        ├── Chart.yaml
        ├── values.yaml          # Image tag updated by Jenkins CI
        └── templates/
            ├── deployment.yaml
            └── service.yaml
```

## How It Works

### Automatic Updates

The `values.yaml` file is automatically updated by Jenkins CI pipeline:

1. Jenkins builds a new Docker image
2. Image is tagged with git commit SHA
3. Jenkins clones this repository
4. Updates `image.tag` in `values.yaml`
5. Commits and pushes changes to `main` branch

### Argo CD Sync

Argo CD monitors this repository and:

- Detects changes within ~30 seconds
- Automatically syncs changes to Kubernetes cluster
- Performs rolling updates for zero-downtime deployments

## Configuration

### Image Configuration

Edit `charts/django-app/values.yaml` to configure:

```yaml
image:
  repository: <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/lesson-8-9-ecr
  tag: "latest"
```

**Note:** The `image.tag` value is automatically updated by Jenkins. Manual changes will be overwritten on next CI run.

### Application Settings

Modify `values.yaml` to adjust:

- Replica count
- Resource limits/requests
- Service type and ports
- Environment variables

## Related Repositories

- **Infrastructure Repository:** [devops](https://github.com/andriignedash/devops) (branch: `lesson-8-9`)
  - Contains: Terraform, Jenkins, Argo CD configuration
  - Contains: Jenkinsfile for CI/CD pipeline

## Important Notes

- **DO NOT** add secrets to this repository
- **DO NOT** add Terraform or CI/CD configuration here
- This repository is **GitOps source of truth only**
- All changes should come through Jenkins CI or manual `values.yaml` updates

## Verification Commands

```bash
# Check current image tag
grep -A 2 "image:" charts/django-app/values.yaml

# View recent commits (should show Jenkins updates)
git log --oneline -5

# Check Argo CD application status
kubectl get applications -n argocd
```

## Author

**Project:** GoIT DevOps Lesson 8-9  
**Date:** 2026-01-29
