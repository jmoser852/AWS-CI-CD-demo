# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AWS CI/CD Demo - A demonstration repository showing how to implement CI/CD pipelines with GitHub Actions to deploy static websites to AWS S3. The live website is accessible at: http://cicd-demo-s3bucket-992382660379.s3-website-us-east-1.amazonaws.com

## Architecture

This project uses Infrastructure as Code (IaC) with a GitOps deployment model:

1. **Infrastructure Layer**: CloudFormation template (`cloudformation/cfn-static-website.yml`) defines the S3 bucket with static website hosting configuration, public access settings, and bucket policies.

2. **Application Layer**: Simple static HTML website files in `website/` directory.

3. **CI/CD Pipeline**: GitHub Actions workflow (`.github/workflows/deploy.yml`) orchestrates the deployment:
   - Uses OIDC authentication with AWS (no long-lived credentials)
   - AWS IAM role: `arn:aws:iam::992382660379:role/GitHubActionsRole`
   - Deploys CloudFormation stack first to ensure infrastructure exists
   - Syncs website files to S3 bucket after infrastructure is ready
   - Triggers on pushes to `main` branch or manual workflow dispatch

## Key AWS Resources

- **S3 Bucket**: `cicd-demo-s3bucket-992382660379` (configured for public website hosting)
- **CloudFormation Stack**: `AWS-CI-CD-Demo`
- **Region**: `us-east-1`

## Deployment Workflow

The deployment follows this sequence:
1. Code pushed to `main` branch triggers GitHub Actions workflow
2. Workflow assumes AWS IAM role via OIDC (no access keys needed)
3. CloudFormation stack is deployed/updated (`no-fail-on-empty-changeset: "1"` prevents failures when no changes detected)
4. Website files are synced to S3 bucket using `aws s3 sync`

## Testing Changes

To test changes to the website or infrastructure:

1. **Test infrastructure changes locally** (requires AWS credentials):
   ```bash
   aws cloudformation validate-template --template-body file://cloudformation/cfn-static-website.yml
   ```

2. **Deploy via GitHub Actions**:
   - Push to `main` branch for automatic deployment
   - Or trigger manual deployment via GitHub Actions UI (workflow_dispatch)

3. **Verify deployment**:
   - Check GitHub Actions run logs for deployment status
   - Visit website URL to confirm changes are live
   - Check CloudFormation stack status in AWS Console if needed

## Important Notes

- The S3 bucket name is hardcoded in both the CloudFormation template and the GitHub Actions workflow
- The bucket is configured for public read access (required for static website hosting)
- OIDC authentication is used instead of static AWS credentials for better security
- The workflow includes `no-fail-on-empty-changeset: "1"` to prevent pipeline failures when CloudFormation detects no changes
