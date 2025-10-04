# AWS Infrastructure Setup AI Prompts

> **AI Context File for HomeManager AWS Infrastructure**

This file contains optimized prompts for AI-assisted AWS infrastructure setup and management.

## Infrastructure Overview Context

```markdown
# AWS Infrastructure Context for HomeManager

## System Architecture
- Microservices deployed on Amazon EKS
- Multi-AZ deployment for high availability
- PostgreSQL RDS for Identity Service
- DynamoDB for Todo Service
- DocumentDB for Budget Service
- S3 for file storage
- CloudFront for CDN
- Amazon Bedrock for AI features

## Requirements
- Production-ready scalability
- Security best practices
- Cost optimization
- Monitoring and observability
- Disaster recovery
- CI/CD integration
```

## Core Infrastructure Prompts

### VPC and Networking Setup
```markdown
Create AWS VPC infrastructure for HomeManager microservices system.

Requirements:
- Multi-AZ deployment across 3 availability zones
- Public and private subnets
- NAT gateways for outbound internet access
- Security groups with least privilege
- Network ACLs for additional security
- VPC endpoints for AWS services

Components needed:
- VPC with CIDR 10.0.0.0/16
- Public subnets: 10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24
- Private subnets: 10.0.11.0/24, 10.0.12.0/24, 10.0.13.0/24
- Database subnets: 10.0.21.0/24, 10.0.22.0/24, 10.0.23.0/24
- Internet Gateway
- NAT Gateways in each public subnet
- Route tables

Security groups:
- ALB security group (HTTP/HTTPS from internet)
- EKS node security group (internal communication)
- RDS security group (PostgreSQL from EKS)
- DocumentDB security group (MongoDB from EKS)

Generate Terraform or CloudFormation templates with:
- Proper resource tagging
- Cost allocation tags
- Security best practices
- Documentation
```

### EKS Cluster Configuration
```markdown
Create Amazon EKS cluster for HomeManager microservices.

Requirements:
- Kubernetes 1.28+
- Multi-AZ node groups
- Auto-scaling configuration
- Managed node groups with mixed instance types
- Fargate profile for system workloads
- RBAC configuration
- Cluster logging enabled

Node groups:
- System node group: t3.medium (2-5 nodes)
- Application node group: c5.large (3-10 nodes)
- Spot instances for cost optimization

Add-ons:
- AWS Load Balancer Controller
- EBS CSI Driver
- VPC CNI
- CoreDNS
- kube-proxy

Security:
- Pod security standards
- Network policies
- Service mesh (Istio)
- Secret management with AWS Secrets Manager

Generate:
- EKS cluster configuration
- Node group definitions
- RBAC policies
- Add-on configurations
- Security policies
```

### Database Infrastructure
```markdown
Set up managed database services for HomeManager.

PostgreSQL RDS (Identity Service):
- Multi-AZ deployment
- db.t3.medium instance class
- 100GB storage with auto-scaling
- Automated backups (7 days retention)
- Performance Insights enabled
- Parameter group optimization
- Security group restrictions

DynamoDB (Todo Service):
- On-demand billing mode
- Global Secondary Indexes
- Point-in-time recovery
- Encryption at rest
- VPC endpoints
- DynamoDB Streams for change capture

DocumentDB (Budget Service):
- 3-node cluster across AZs
- db.t3.medium instances
- Automated backups
- Encryption at rest and in transit
- VPC endpoint
- Monitoring with CloudWatch

Generate:
- Database cluster configurations
- Security group rules
- Parameter groups
- Backup strategies
- Monitoring setup
```

### Storage and CDN Setup
```markdown
Configure S3 and CloudFront for HomeManager file storage and web delivery.

S3 Buckets:
- homemanager-files-prod (user uploads)
- homemanager-web-assets-prod (static assets)
- homemanager-backups-prod (database backups)
- homemanager-logs-prod (application logs)

S3 Configuration:
- Versioning enabled
- Lifecycle policies for cost optimization
- Cross-region replication for disaster recovery
- Server-side encryption with KMS
- Public access blocked
- Bucket policies for application access

CloudFront:
- Global distribution with edge locations
- Origin Access Control for S3
- Custom domain with SSL certificate
- Caching strategies for different content types
- Security headers
- WAF integration

Generate:
- S3 bucket policies
- CloudFront distribution configuration
- Lambda@Edge functions for authentication
- SSL certificate management
- Monitoring and alerting
```

### AI Services Configuration
```markdown
Set up Amazon Bedrock and related AI services for HomeManager.

Bedrock Configuration:
- Model access for Claude 3 Sonnet and Haiku
- VPC endpoints for private access
- IAM roles and policies
- Cost monitoring and budgets
- Usage tracking and analytics

Integration Requirements:
- Secure API access from EKS pods
- Request/response logging
- Error handling and retry logic
- Rate limiting and throttling
- Cost optimization strategies

Supporting Services:
- Amazon Textract for receipt processing
- Amazon Comprehend for text analysis
- Amazon Translate for multi-language support
- CloudWatch for monitoring

Generate:
- Bedrock service configuration
- IAM policies for AI service access
- VPC endpoints
- Monitoring dashboards
- Cost allocation and budgets
```

## Security and Compliance

### Security Configuration
```markdown
Implement comprehensive security for HomeManager AWS infrastructure.

Identity and Access Management:
- IAM roles for EKS service accounts
- Cross-account access policies
- MFA enforcement
- Regular access reviews
- Principle of least privilege

Encryption:
- KMS keys for different services
- Encryption at rest for all data stores
- TLS 1.3 for data in transit
- Certificate management with ACM

Network Security:
- VPC Flow Logs
- Security groups with minimal access
- Network ACLs
- WAF rules for web applications
- DDoS protection with Shield

Monitoring and Compliance:
- CloudTrail for API logging
- Config for compliance monitoring
- GuardDuty for threat detection
- Security Hub for centralized security
- Inspector for vulnerability scanning

Generate:
- IAM policies and roles
- KMS key policies
- Security group rules
- WAF rules
- Compliance monitoring setup
```

### Disaster Recovery
```markdown
Design disaster recovery strategy for HomeManager.

RTO/RPO Requirements:
- RTO: 4 hours maximum
- RPO: 1 hour maximum
- Multi-region deployment capability
- Automated failover procedures

Backup Strategy:
- RDS automated backups and snapshots
- DynamoDB point-in-time recovery
- DocumentDB automated backups
- S3 cross-region replication
- EKS cluster backup with Velero

Disaster Recovery Setup:
- Secondary region deployment (us-west-2)
- DNS failover with Route 53
- Data replication strategies
- Infrastructure as Code for rapid rebuild
- Runbook automation

Testing:
- Monthly DR testing
- Automated failover testing
- Performance validation
- Documentation updates

Generate:
- DR infrastructure templates
- Failover procedures
- Testing automation
- Monitoring and alerting
- Recovery documentation
```

## Monitoring and Observability

### Monitoring Setup
```markdown
Implement comprehensive monitoring for HomeManager infrastructure.

CloudWatch Configuration:
- Custom metrics for application performance
- Log aggregation from all services
- Dashboards for different stakeholders
- Alerting rules with SNS notifications
- Cost monitoring and budgets

Application Monitoring:
- Distributed tracing with X-Ray
- Application performance monitoring
- Error tracking and alerting
- User experience monitoring
- Business metrics tracking

Infrastructure Monitoring:
- EKS cluster health
- Database performance metrics
- Network performance
- Storage utilization
- Security events

Alerting Strategy:
- Critical alerts: immediate notification
- Warning alerts: daily summary
- Info alerts: weekly reports
- Escalation procedures
- On-call rotation setup

Generate:
- CloudWatch dashboards
- Metric filters and alarms
- SNS topics and subscriptions
- X-Ray tracing configuration
- Runbook automation
```

## Cost Optimization

### Cost Management
```markdown
Implement cost optimization strategies for HomeManager AWS infrastructure.

Compute Optimization:
- Reserved Instances for baseline capacity
- Spot Instances for development/testing
- Auto-scaling policies
- Right-sizing recommendations
- Scheduled scaling for predictable workloads

Storage Optimization:
- S3 Intelligent Tiering
- Lifecycle policies for data archival
- EBS volume optimization
- Snapshot lifecycle management
- Data compression strategies

Database Optimization:
- Reserved capacity for DynamoDB
- RDS Reserved Instances
- Performance optimization
- Query optimization
- Connection pooling

Monitoring and Governance:
- Cost allocation tags
- Budget alerts
- Cost anomaly detection
- Regular cost reviews
- Resource usage optimization

Generate:
- Cost optimization policies
- Budget configurations
- Tagging strategies
- Automation scripts
- Cost reporting dashboards
```

## Automation and DevOps

### Infrastructure as Code
```markdown
Create Infrastructure as Code templates for HomeManager.

Terraform Modules:
- VPC and networking
- EKS cluster
- Database services
- Storage and CDN
- Security configurations
- Monitoring setup

Organization:
- Environment-specific configurations
- Shared modules for reusability
- State management with remote backend
- Variable management
- Documentation

CI/CD Integration:
- Terraform validation in pipelines
- Plan and apply automation
- State drift detection
- Security scanning
- Cost estimation

Best Practices:
- Version control for all infrastructure
- Peer review process
- Testing with Terratest
- Documentation generation
- Change management procedures

Generate:
- Terraform module structure
- Variable definitions
- Output configurations
- Documentation
- Testing framework
```