# AWS Infrastructure Implementation Guide

This guide provides step-by-step instructions for setting up the AWS infrastructure for the HomeManager system.

## Overview

The HomeManager system uses AWS cloud services for hosting, data storage, and AI capabilities. This guide covers the setup of all required AWS resources.

## Prerequisites

- AWS CLI installed and configured
- AWS CDK installed (`npm install -g aws-cdk`)
- Docker installed
- Node.js 18+ installed
- .NET 10 SDK installed

## AWS Services Used

### Core Infrastructure
- **Amazon ECS/Fargate**: Container hosting for microservices
- **Amazon RDS**: PostgreSQL database for persistent data
- **Amazon ElastiCache**: Redis for caching and session storage
- **Amazon S3**: File storage and static assets
- **Amazon CloudFront**: CDN for web application

### AI Services
- **Amazon Bedrock**: Foundation models for AI features
- **Amazon API Gateway**: AI Gateway service routing
- **AWS Lambda**: Serverless functions for AI processing

### Security & Monitoring
- **Amazon Cognito**: User authentication and authorization
- **AWS WAF**: Web application firewall
- **Amazon CloudWatch**: Monitoring and logging
- **AWS X-Ray**: Distributed tracing

## Infrastructure Setup

### 1. Environment Preparation

```bash
# Configure AWS CLI
aws configure

# Bootstrap CDK (first time only)
cdk bootstrap

# Clone infrastructure repository
git clone https://github.com/kjaworski/HomeManager-Infrastructure.git
cd HomeManager-Infrastructure
```

### 2. Core Infrastructure Deployment

```bash
# Install dependencies
npm install

# Deploy VPC and networking
cdk deploy HomeManager-Network-Stack

# Deploy databases
cdk deploy HomeManager-Database-Stack

# Deploy container infrastructure
cdk deploy HomeManager-Container-Stack
```

### 3. Database Setup

```sql
-- Connect to RDS PostgreSQL instance
-- Create databases for each service
CREATE DATABASE homemanager_identity;
CREATE DATABASE homemanager_todo;
CREATE DATABASE homemanager_budget;

-- Create service users
CREATE USER identity_service WITH PASSWORD 'secure_password';
CREATE USER todo_service WITH PASSWORD 'secure_password';
CREATE USER budget_service WITH PASSWORD 'secure_password';

-- Grant permissions
GRANT ALL PRIVILEGES ON DATABASE homemanager_identity TO identity_service;
GRANT ALL PRIVILEGES ON DATABASE homemanager_todo TO todo_service;
GRANT ALL PRIVILEGES ON DATABASE homemanager_budget TO budget_service;
```

### 4. Service Deployment

```bash
# Build and push Docker images
./scripts/build-and-push.sh

# Deploy services
cdk deploy HomeManager-Services-Stack

# Deploy AI Gateway
cdk deploy HomeManager-AI-Stack
```

### 5. Frontend Deployment

```bash
# Deploy web application
cdk deploy HomeManager-Web-Stack

# Deploy mobile app distribution
cdk deploy HomeManager-Mobile-Stack
```

## Configuration

### Environment Variables

Create `.env` files for each environment:

```env
# Production environment
AWS_REGION=us-east-1
RDS_ENDPOINT=homemanager-prod.cluster-xyz.us-east-1.rds.amazonaws.com
REDIS_ENDPOINT=homemanager-prod.cache.amazonaws.com
S3_BUCKET=homemanager-prod-storage
BEDROCK_REGION=us-east-1
COGNITO_USER_POOL_ID=us-east-1_AbCdEfGhI
```

### Service Discovery

Services use AWS Service Discovery for inter-service communication:

```yaml
# service-discovery.yml
services:
  identity-service:
    endpoint: identity.homemanager.local
    port: 8080
  todo-service:
    endpoint: todo.homemanager.local
    port: 8080
  budget-service:
    endpoint: budget.homemanager.local
    port: 8080
```

## Security Configuration

### IAM Roles and Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": "arn:aws:bedrock:*:*:foundation-model/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::homemanager-*/*"
    }
  ]
}
```

### VPC Security Groups

```typescript
// Security group for ECS services
const ecsSecurityGroup = new ec2.SecurityGroup(this, 'EcsSecurityGroup', {
  vpc,
  description: 'Security group for ECS services',
  allowAllOutbound: true
});

// Allow inbound traffic from ALB
ecsSecurityGroup.addIngressRule(
  albSecurityGroup,
  ec2.Port.tcp(8080),
  'Allow traffic from ALB'
);
```

## Monitoring and Logging

### CloudWatch Configuration

```typescript
// Create log groups for each service
const logGroup = new logs.LogGroup(this, 'ServiceLogGroup', {
  logGroupName: '/aws/ecs/homemanager/identity-service',
  retention: logs.RetentionDays.THIRTY_DAYS
});

// Create custom metrics
const customMetric = new cloudwatch.Metric({
  namespace: 'HomeManager',
  metricName: 'ActiveUsers',
  dimensionsMap: {
    Service: 'IdentityService'
  }
});
```

### Alarms and Notifications

```typescript
// Create SNS topic for alerts
const alertTopic = new sns.Topic(this, 'AlertTopic');

// Create CloudWatch alarms
const highErrorRateAlarm = new cloudwatch.Alarm(this, 'HighErrorRate', {
  metric: apiGateway.metricServerError(),
  threshold: 10,
  evaluationPeriods: 2
});

highErrorRateAlarm.addAlarmAction(new snsActions.SnsAction(alertTopic));
```

## Backup and Disaster Recovery

### RDS Automated Backups

```typescript
const database = new rds.DatabaseCluster(this, 'Database', {
  engine: rds.DatabaseClusterEngine.auroraPostgres({
    version: rds.AuroraPostgresEngineVersion.VER_15_4
  }),
  backup: {
    retention: cdk.Duration.days(30),
    preferredWindow: '03:00-04:00'
  },
  deletionProtection: true
});
```

### S3 Cross-Region Replication

```typescript
// Setup cross-region replication for critical data
const replicationRule = {
  id: 'ReplicateToSecondaryRegion',
  status: 'Enabled',
  prefix: 'critical-data/',
  destination: {
    bucket: 'homemanager-backup-us-west-2',
    storageClass: 'STANDARD_IA'
  }
};
```

## Cost Optimization

### Resource Tagging

```typescript
// Apply consistent tagging
cdk.Tags.of(this).add('Project', 'HomeManager');
cdk.Tags.of(this).add('Environment', props.environment);
cdk.Tags.of(this).add('CostCenter', 'Engineering');
```

### Auto Scaling Configuration

```typescript
// Configure ECS auto scaling
const scalableTarget = service.autoScaleTaskCount({
  minCapacity: 1,
  maxCapacity: 10
});

scalableTarget.scaleOnCpuUtilization('CpuScaling', {
  targetUtilizationPercent: 70,
  scaleInCooldown: cdk.Duration.minutes(5),
  scaleOutCooldown: cdk.Duration.minutes(2)
});
```

## Deployment Scripts

### Build and Deploy Script

```bash
#!/bin/bash
# build-and-deploy.sh

set -e

echo "Building Docker images..."
docker build -t homemanager/identity-service ./src/HomeManager.Identity
docker build -t homemanager/todo-service ./src/HomeManager.Todo
docker build -t homemanager/budget-service ./src/HomeManager.Budget

echo "Pushing to ECR..."
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

docker tag homemanager/identity-service:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/homemanager/identity-service:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/homemanager/identity-service:latest

echo "Deploying infrastructure..."
cdk deploy --all --require-approval never

echo "Deployment complete!"
```

## Troubleshooting

### Common Issues

1. **ECS Task Fails to Start**
   - Check CloudWatch logs
   - Verify environment variables
   - Check security group rules

2. **Database Connection Issues**
   - Verify RDS security groups
   - Check connection strings
   - Validate IAM roles

3. **Bedrock API Errors**
   - Check IAM permissions
   - Verify model availability in region
   - Review rate limits

### Useful Commands

```bash
# Check ECS service status
aws ecs describe-services --cluster homemanager-cluster --services identity-service

# View CloudWatch logs
aws logs tail /aws/ecs/homemanager/identity-service --follow

# Check RDS status
aws rds describe-db-clusters --db-cluster-identifier homemanager-db
```

## Next Steps

1. Set up monitoring dashboards
2. Configure automated testing pipeline
3. Implement blue-green deployment
4. Set up disaster recovery procedures
5. Configure cost monitoring and alerts

For more information, see:
- [Development Setup Guide](development-setup.md)
- [CI/CD Pipeline Guide](cicd-pipeline.md)
- [Monitoring Setup Guide](monitoring-setup.md)
