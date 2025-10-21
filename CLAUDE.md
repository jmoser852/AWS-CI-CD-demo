# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AWS CI/CD Demo - A demonstration repository showing how to implement multi-environment CI/CD pipelines with GitHub Actions to deploy static websites to AWS S3.

**Live Environments:**
- Production: http://cicd.jmoser.net
- Staging: http://cicd-staging.jmoser.net

## Architecture

This project demonstrates a **GitOps-based multi-environment deployment pipeline** with Infrastructure as Code:

### Three-Tier Pipeline Structure

1. **Test Job**: Validates CloudFormation template syntax before any deployment
2. **Staging Deployment**: Deploys to staging environment after tests pass
3. **Production Deployment**: Deploys to production only after successful staging deployment

### Key Architectural Components

1. **Infrastructure Layer** ([cloudformation/cfn-static-website.yml](cloudformation/cfn-static-website.yml)):
   - Parameterized CloudFormation template (accepts `BucketName` parameter)
   - Defines S3 bucket with static website hosting configuration
   - Configures public access settings and bucket policies
   - Outputs include WebsiteURL, BucketName, and BucketArn

2. **Application Layer**: Static HTML files in [website/](website/) directory
   - [index.html](website/index.html) - main landing page
   - [error.html](website/error.html) - error page

3. **CI/CD Pipeline** ([.github/workflows/ci-cd.yml](.github/workflows/ci-cd.yml)):
   - Uses GitHub Actions with **environment-specific configurations**
   - OIDC authentication with AWS (no long-lived credentials)
   - Each environment (staging, production) has its own:
     - GitHub Environment configuration
     - S3 bucket (via `BUCKET_NAME` variable)
     - CloudFormation stack name (`AWS-CI-CD-Demo-{ENVIRONMENT}`)
   - Environment variables are configured at the **repository level**, not hardcoded

## Environment Configuration

The pipeline uses GitHub repository variables and secrets:

- **Repository Variables** (per environment):
  - `AWS_REGION` - AWS region for deployment
  - `BUCKET_NAME` - S3 bucket name for the environment
  - `ENVIRONMENT` - Environment identifier (staging/production)

- **Repository Secrets**:
  - `IAM_ROLE` - ARN of the IAM role for OIDC authentication

## Deployment Flow

```
Push to main → Test → Deploy Staging → Deploy Production
```

Each deployment step:
1. Assumes AWS IAM role via OIDC
2. Deploys/updates CloudFormation stack with environment-specific parameters
3. Syncs website files to environment-specific S3 bucket using `aws s3 sync`

## Testing Infrastructure Changes

**Validate CloudFormation template locally** (requires AWS credentials):
```bash
aws cloudformation validate-template --template-body file://cloudformation/cfn-static-website.yml
```

**Deploy changes:**
- Push to `main` branch triggers automatic deployment pipeline
- Or use GitHub Actions UI for manual workflow dispatch
- Pipeline will automatically deploy to staging first, then production

**Verify deployment:**
- Check GitHub Actions logs for deployment status
- Visit staging URL first, then production URL
- Verify CloudFormation stack outputs in AWS Console if needed

## Important Notes

- CloudFormation stacks are **parameterized** - bucket names come from GitHub environment variables
- The workflow uses `no-fail-on-empty-changeset: "1"` to prevent failures when no infrastructure changes are detected
- OIDC authentication provides better security than static AWS credentials
- Production deployment requires successful staging deployment (job dependency)
