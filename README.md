# Shared GitHub Actions Workflows

Centralized, reusable CI/CD workflows for projects across the `@Steven-Robinson` GitHub account.

## Reusable Workflows

### `tofu-pipeline.yml`

Automates OpenTofu (and Terraform) formatting, validation, planning, and deployment using AWS IAM OpenID Connect (OIDC) authentication without long-lived static AWS access keys.

- **Authentication**: AWS IAM OIDC federation (`sts:AssumeRoleWithWebIdentity`)
- **State Locking**: Native S3 conditional write locking (`use_lockfile = true`) — no DynamoDB required
- **Secrets Support**: Supports repository secrets via `secrets: inherit` (`AWS_ROLE_ARN`, `TF_STATE_BUCKET`)

### Usage in Caller Repositories

Create `.github/workflows/infra.yml` in your project repository:

```yaml
name: "Infrastructure Deployment"

on:
  push:
    branches: [main]
    paths:
      - "terraform/**"
      - ".github/workflows/infra.yml"
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    uses: Steven-Robinson/github-workflows/.github/workflows/tofu-pipeline.yml@main
    secrets: inherit
    with:
      working_directory: "terraform"
      aws_region: "eu-west-1"
```

## Backend Configuration Standard

Caller repositories use remote S3 backend with native S3 locking. The bucket name can be passed via repository secret `TF_STATE_BUCKET` (ideal for public repos) or declared directly:

```hcl
terraform {
  required_version = ">= 1.5.0"

  backend "s3" {
    key          = "<project-name>/terraform.tfstate"
    region       = "eu-west-1"
    encrypt      = true
    use_lockfile = true
  }
}
```
