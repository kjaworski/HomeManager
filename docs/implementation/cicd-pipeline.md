# CI/CD Pipeline Implementation Guide

This guide provides comprehensive instructions for setting up the CI/CD pipeline for the HomeManager system using GitHub Actions.

## Overview

The HomeManager CI/CD pipeline automates building, testing, security scanning, and deployment across multiple environments. The pipeline supports microservices architecture with independent deployments.

## Pipeline Architecture

### Workflow Structure
- **Build and Test**: Compile code, run unit tests, integration tests
- **Security Scan**: CodeQL analysis, dependency vulnerability scanning
- **Quality Gates**: Code coverage, lint checks, documentation validation
- **Deployment**: Automated deployment to staging and production
- **Monitoring**: Post-deployment health checks and notifications

## GitHub Actions Workflows

### Main Build Pipeline

Create `.github/workflows/build-and-test.yml`:

```yaml
name: Build and Test

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  DOTNET_VERSION: '10.0.x'
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      api: ${{ steps.changes.outputs.api }}
      web: ${{ steps.changes.outputs.web }}
      mobile: ${{ steps.changes.outputs.mobile }}
      docs: ${{ steps.changes.outputs.docs }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            api:
              - 'src/**'
              - 'tests/**'
              - '*.sln'
            web:
              - 'web/**'
              - 'shared/**'
            mobile:
              - 'mobile/**'
              - 'shared/**'
            docs:
              - 'docs/**'
              - '*.md'

  build-api:
    needs: changes
    if: ${{ needs.changes.outputs.api == 'true' }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [Identity, Todo, Budget, AI.Gateway]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}
      
      - name: Restore dependencies
        run: dotnet restore src/HomeManager.${{ matrix.service }}
      
      - name: Build
        run: dotnet build src/HomeManager.${{ matrix.service }} --no-restore
      
      - name: Test
        run: dotnet test tests/HomeManager.${{ matrix.service }}.Tests --no-build --verbosity normal --collect:"XPlat Code Coverage"
      
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          file: tests/HomeManager.${{ matrix.service }}.Tests/coverage.cobertura.xml
          flags: ${{ matrix.service }}

  build-web:
    needs: changes
    if: ${{ needs.changes.outputs.web == 'true' }}
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: web/package-lock.json
      
      - name: Install dependencies
        run: npm ci
        working-directory: web
      
      - name: Lint
        run: npm run lint
        working-directory: web
      
      - name: Test
        run: npm run test:coverage
        working-directory: web
      
      - name: Build
        run: npm run build
        working-directory: web
        env:
          CI: false
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: web-build
          path: web/build/

  build-mobile:
    needs: changes
    if: ${{ needs.changes.outputs.mobile == 'true' }}
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: mobile/package-lock.json
      
      - name: Install dependencies
        run: npm ci
        working-directory: mobile
      
      - name: Lint
        run: npm run lint
        working-directory: mobile
      
      - name: Test
        run: npm run test
        working-directory: mobile
      
      - name: Build for Web
        run: npm run build:web
        working-directory: mobile

  validate-docs:
    needs: changes
    if: ${{ needs.changes.outputs.docs == 'true' }}
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Validate Markdown Links
        uses: gaurav-nelson/github-action-markdown-link-check@v1
        with:
          use-quiet-mode: 'yes'
          use-verbose-mode: 'yes'
          config-file: '.github/mlc_config.json'
          folder-path: '.'
          max-depth: -1
          check-modified-files-only: 'no'
          base-branch: 'main'
          file-extension: '.md'
      
      - name: Lint Markdown
        run: |
          npm install -g markdownlint-cli
          markdownlint docs/**/*.md README.md
```

### Security Scanning Pipeline

Create `.github/workflows/security-scan.yml`:

```yaml
name: Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * 1'  # Weekly on Monday at 2 AM

jobs:
  codeql:
    name: CodeQL Analysis
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write
    
    strategy:
      fail-fast: false
      matrix:
        language: [ 'csharp', 'javascript' ]
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: ${{ matrix.language }}
          queries: security-extended,security-and-quality
      
      - name: Setup .NET
        if: matrix.language == 'csharp'
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'
      
      - name: Build .NET
        if: matrix.language == 'csharp'
        run: dotnet build HomeManager.sln
      
      - name: Setup Node.js
        if: matrix.language == 'javascript'
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - name: Build JavaScript
        if: matrix.language == 'javascript'
        run: |
          cd web && npm ci && npm run build
          cd ../mobile && npm ci && npm run build:web
      
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2
        with:
          category: "/language:${{matrix.language}}"

  dependency-scan:
    name: Dependency Vulnerability Scan
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  secrets-scan:
    name: Secrets Detection
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Run TruffleHog
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: main
          head: HEAD
          extra_args: --debug --only-verified
```

### Deployment Pipeline

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy

on:
  push:
    branches: [ main ]
  workflow_run:
    workflows: ["Build and Test", "Security Scan"]
    types:
      - completed
    branches: [ main ]

env:
  AWS_REGION: us-east-1
  ECR_REGISTRY: ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.us-east-1.amazonaws.com

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/main' && github.event.workflow_run.conclusion == 'success'
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Login to ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Build and push Docker images
        run: |
          services=("identity" "todo" "budget" "ai-gateway")
          for service in "${services[@]}"; do
            docker build -t $ECR_REGISTRY/homemanager-$service:$GITHUB_SHA \
              -f src/HomeManager.${service^}/Dockerfile .
            docker push $ECR_REGISTRY/homemanager-$service:$GITHUB_SHA
            docker tag $ECR_REGISTRY/homemanager-$service:$GITHUB_SHA \
              $ECR_REGISTRY/homemanager-$service:staging
            docker push $ECR_REGISTRY/homemanager-$service:staging
          done
      
      - name: Deploy to ECS Staging
        run: |
          aws ecs update-service \
            --cluster homemanager-staging \
            --service identity-service \
            --force-new-deployment
          
          aws ecs update-service \
            --cluster homemanager-staging \
            --service todo-service \
            --force-new-deployment
          
          aws ecs update-service \
            --cluster homemanager-staging \
            --service budget-service \
            --force-new-deployment
      
      - name: Deploy Web App to S3
        run: |
          aws s3 sync web/build/ s3://homemanager-web-staging/ --delete
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID_STAGING }} \
            --paths "/*"
      
      - name: Run health checks
        run: |
          ./scripts/health-check.sh staging
      
      - name: Notify deployment
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          channel: '#deployments'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
          fields: repo,message,commit,author,action,eventName,ref,workflow

  deploy-production:
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Login to ECR
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Promote images to production
        run: |
          services=("identity" "todo" "budget" "ai-gateway")
          for service in "${services[@]}"; do
            docker pull $ECR_REGISTRY/homemanager-$service:staging
            docker tag $ECR_REGISTRY/homemanager-$service:staging \
              $ECR_REGISTRY/homemanager-$service:production
            docker push $ECR_REGISTRY/homemanager-$service:production
          done
      
      - name: Blue-Green Deployment
        run: |
          ./scripts/blue-green-deploy.sh production
      
      - name: Run production health checks
        run: |
          ./scripts/health-check.sh production
      
      - name: Create GitHub Release
        if: success()
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ github.run_number }}
          release_name: Release v${{ github.run_number }}
          body: |
            Changes in this Release:
            ${{ github.event.head_commit.message }}
            
            **Deployed Services:**
            - Identity Service
            - Todo Service
            - Budget Service
            - AI Gateway
            
            **Infrastructure:**
            - Web Application
            - Mobile Application
          draft: false
          prerelease: false
```

## Pipeline Configuration

### Environment Secrets

Configure the following secrets in GitHub repository settings:

```bash
# AWS Configuration
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
AWS_ACCOUNT_ID=123456789012

# Container Registry
ECR_REGISTRY=123456789012.dkr.ecr.us-east-1.amazonaws.com

# Deployment
CLOUDFRONT_DISTRIBUTION_ID_STAGING=E1234567890ABC
CLOUDFRONT_DISTRIBUTION_ID_PRODUCTION=E0987654321DEF

# Notifications
SLACK_WEBHOOK=https://hooks.slack.com/services/...

# Code Coverage
CODECOV_TOKEN=...
```

### Branch Protection Rules

Set up branch protection for `main` branch:

```json
{
  "required_status_checks": {
    "strict": true,
    "contexts": [
      "Build and Test",
      "Security Scan",
      "Validate Documentation"
    ]
  },
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": false
  },
  "restrictions": null
}
```

## Supporting Scripts

### Health Check Script

Create `scripts/health-check.sh`:

```bash
#!/bin/bash
# Health check script for deployed services

set -e

ENVIRONMENT=$1
BASE_URL="https://api-${ENVIRONMENT}.homemanager.com"

echo "Running health checks for ${ENVIRONMENT} environment..."

# Check each service
services=("identity" "todo" "budget" "ai-gateway")

for service in "${services[@]}"; do
  echo "Checking ${service} service..."
  
  response=$(curl -s -o /dev/null -w "%{http_code}" "${BASE_URL}/${service}/health")
  
  if [ "$response" -eq 200 ]; then
    echo "✅ ${service} service is healthy"
  else
    echo "❌ ${service} service is unhealthy (HTTP $response)"
    exit 1
  fi
done

# Check web application
echo "Checking web application..."
web_response=$(curl -s -o /dev/null -w "%{http_code}" "https://web-${ENVIRONMENT}.homemanager.com")

if [ "$web_response" -eq 200 ]; then
  echo "✅ Web application is healthy"
else
  echo "❌ Web application is unhealthy (HTTP $web_response)"
  exit 1
fi

echo "All health checks passed! 🎉"
```

### Blue-Green Deployment Script

Create `scripts/blue-green-deploy.sh`:

```bash
#!/bin/bash
# Blue-green deployment script

set -e

ENVIRONMENT=$1
CLUSTER="homemanager-${ENVIRONMENT}"

echo "Starting blue-green deployment for ${ENVIRONMENT}..."

# Get current task definitions
services=("identity-service" "todo-service" "budget-service" "ai-gateway")

for service in "${services[@]}"; do
  echo "Updating ${service}..."
  
  # Get current task definition
  current_task_def=$(aws ecs describe-services \
    --cluster $CLUSTER \
    --services $service \
    --query 'services[0].taskDefinition' \
    --output text)
  
  # Update service with new task definition
  aws ecs update-service \
    --cluster $CLUSTER \
    --service $service \
    --task-definition $service:latest \
    --deployment-configuration '{"maximumPercent": 200, "minimumHealthyPercent": 50}'
  
  # Wait for deployment to complete
  echo "Waiting for ${service} deployment to complete..."
  aws ecs wait services-stable \
    --cluster $CLUSTER \
    --services $service
  
  echo "✅ ${service} deployment completed"
done

echo "Blue-green deployment completed successfully! 🎉"
```

## Monitoring and Alerting

### Pipeline Metrics

Track key pipeline metrics:
- Build success rate
- Deployment frequency
- Lead time for changes
- Mean time to recovery
- Test coverage percentage

### Slack Notifications

Configure Slack notifications for:
- Build failures
- Security scan alerts
- Deployment status
- Health check failures

### CloudWatch Dashboards

Create dashboards for:
- Pipeline execution metrics
- Deployment success rates
- Service health status
- Performance metrics

## Troubleshooting

### Common Issues

1. **Build Failures**
   - Check dependency versions
   - Verify environment variables
   - Review test failures

2. **Deployment Failures**
   - Check AWS permissions
   - Verify ECR image availability
   - Review ECS service logs

3. **Health Check Failures**
   - Check service endpoints
   - Verify load balancer configuration
   - Review CloudWatch logs

### Useful Commands

```bash
# Check workflow status
gh run list --workflow="Build and Test"

# View workflow logs
gh run view <run-id> --log

# Restart failed workflow
gh run rerun <run-id>

# Check ECS deployment status
aws ecs describe-services --cluster homemanager-production --services identity-service
```

## Best Practices

1. **Fail Fast**: Run quick tests first, expensive tests later
2. **Parallel Execution**: Run independent jobs in parallel
3. **Caching**: Use dependency caching to speed up builds
4. **Security**: Scan for vulnerabilities early and often
5. **Monitoring**: Monitor pipeline performance and reliability
6. **Documentation**: Keep pipeline documentation up to date

## Next Steps

1. Set up advanced monitoring and alerting
2. Implement automated rollback procedures
3. Add performance testing to pipeline
4. Configure multi-region deployments
5. Implement canary deployments

For more information, see:
- [AWS Infrastructure Guide](aws-infrastructure.md)
- [Development Setup Guide](development-setup.md)
- [Monitoring Setup Guide](monitoring-setup.md)
