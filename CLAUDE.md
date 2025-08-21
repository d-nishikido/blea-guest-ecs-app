# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a BLEA (Baseline Environment on AWS) Guest ECS Application sample - an AWS CDK project that deploys a containerized web application using ECS/Fargate with Aurora PostgreSQL database, CloudFront distribution, and comprehensive monitoring.

## Key Commands

### Development & Build
```bash
npm run build          # Compile TypeScript to JavaScript
npm run watch          # Watch for changes and compile
npm run clean          # Clean build artifacts and CDK output
npm test               # Run Jest unit tests
```

### CDK Deployment
```bash
npm run synth          # Synthesize CDK stacks (both standard and pipeline versions)
npx cdk deploy         # Deploy stack to AWS
npx cdk diff           # Compare deployed stack with current state
npx cdk destroy        # Remove stack from AWS
```

### CDK Pipeline Deployment
```bash
npm run synth:pipelines  # Synthesize CDK Pipelines stack specifically
```

## Architecture

### Stack Structure
The application is divided into three main stacks:

1. **BLEAEcsAppStack** (`lib/stack/blea-guest-ecs-app-sample-stack.ts`)
   - Core backend infrastructure
   - VPC networking setup
   - ECS cluster with Fargate service
   - Aurora PostgreSQL database
   - Application Load Balancer
   - KMS encryption keys

2. **BLEAEcsAppFrontendStack** (`lib/stack/blea-guest-ecs-app-frontend-stack.ts`)
   - CloudFront distribution
   - WAFv2 web ACL (deployed in us-east-1)
   - Optional custom domain support

3. **BLEAEcsAppMonitoringStack** (`lib/stack/blea-guest-ecs-app-monitoring-stack.ts`)
   - CloudWatch dashboards
   - Synthetic canary monitoring
   - Alarms and notifications

### Key Constructs
- **Networking** (`lib/construct/networking.ts`): VPC with public/private subnets
- **Datastore** (`lib/construct/datastore.ts`): Aurora PostgreSQL cluster
- **EcsApp** (`lib/construct/ecsapp.ts`): ECS service with auto-scaling
- **Frontend** (`lib/construct/frontend.ts`): CloudFront and WAF configuration
- **Monitoring** (`lib/construct/monitoring.ts`): SNS topics for alarms
- **Dashboard** (`lib/construct/dashboard.ts`): CloudWatch dashboards
- **Canary** (`lib/construct/canary.ts`): Synthetic monitoring

### Entry Points
- **Standard deployment**: `bin/blea-guest-ecs-app-sample.ts`
- **CDK Pipelines deployment**: `bin/blea-guest-ecs-app-sample-via-cdk-pipelines.ts`

## Configuration

All environment-specific parameters are centralized in `parameter.ts`:
- VPC CIDR ranges
- Monitoring email and Slack channels
- Optional custom domain settings
- Pipeline configuration (repository, branch, connection ARN)

### Environment Parameters
- `devParameter`: Development environment configuration
- `devPipelineParameter`: Pipeline account configuration

## Testing

The project includes Jest tests with CDK snapshot testing:
- Test files: `test/*.test.ts`
- Snapshots: `test/__snapshots__/`
- Custom snapshot serializer for ignoring asset hashes

## CDK Pipeline Support

When using CDK Pipelines:
1. Bootstrap accounts with trust relationship: `cdk bootstrap --trust PIPELINE_ACCOUNT_ID`
2. Configure CodeStar connection for GitHub repository
3. Update `devPipelineParameter` in `parameter.ts` with connection ARN

## Cross-Region References

The stacks use `crossRegionReferences: true` to enable:
- Frontend stack in us-east-1 (for WAFv2)
- Backend and monitoring stacks in ap-northeast-1 (or configured region)
- Automatic handling of cross-region exports/imports