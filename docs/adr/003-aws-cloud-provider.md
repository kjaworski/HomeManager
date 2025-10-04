# ADR-003: AWS Cloud Provider Selection

**Status:** Accepted  
**Date:** 2024-10-03  
**Decision Makers:** Development Team  
**AI Development Context:** ✅ Optimized for learning AWS services with AI assistance

## Context

We need to select a cloud provider for hosting our HomeManager microservices application. The primary goals are learning modern cloud services, gaining NoSQL experience, and implementing AI-powered features efficiently.

## Decision

We will use **Amazon Web Services (AWS)** as our primary cloud provider, leveraging their comprehensive AI services and mature NoSQL offerings.

## Options Considered

### Option 1: Microsoft Azure
```yaml
Proposed Stack:
  API: Azure App Service
  Databases: Cosmos DB, Azure SQL Database
  AI: Azure OpenAI Service, Cognitive Services
  Containers: Azure Container Instances
  Messaging: Service Bus
  Storage: Blob Storage
  Auth: Azure AD B2C
```

**Pros:**
- Excellent .NET integration and tooling
- Azure OpenAI with GPT models
- Cosmos DB multi-model database
- Strong enterprise authentication (Azure AD)
- Integrated Visual Studio deployment

**Cons:**
- Cosmos DB pricing can be expensive for learning
- Less diverse NoSQL learning (single database technology)
- AI services less comprehensive than AWS
- Smaller ecosystem for third-party integrations

### Option 2: Google Cloud Platform (GCP)
```yaml
Proposed Stack:
  API: Cloud Run
  Databases: Firestore, Cloud SQL
  AI: Vertex AI, PaLM API
  Containers: Cloud Run
  Messaging: Pub/Sub
  Storage: Cloud Storage
  Auth: Firebase Authentication
```

**Pros:**
- Strong AI/ML capabilities with Vertex AI
- Firestore real-time capabilities
- Competitive pricing model
- Excellent Kubernetes integration

**Cons:**
- Limited NoSQL variety for learning
- Smaller ecosystem compared to AWS
- Less mature .NET support
- AI services less accessible than AWS

### Option 3: Amazon Web Services (AWS) ✅ **CHOSEN**
```yaml
Chosen Stack:
  API: Amazon EKS + Application Load Balancer
  Databases: RDS PostgreSQL, DynamoDB, DocumentDB
  AI: Amazon Bedrock + Claude/Titan models
  Containers: EKS with Fargate
  Messaging: Amazon SQS + EventBridge
  Storage: S3 + CloudFront
  Auth: Amazon Cognito
  Monitoring: CloudWatch + X-Ray
  CI/CD: AWS CodePipeline + GitHub Actions
```

**Pros:**
- **Diverse NoSQL Learning**: DynamoDB + DocumentDB different patterns
- **Comprehensive AI Platform**: Bedrock with multiple model providers
- **Mature Ecosystem**: Largest cloud provider with extensive documentation
- **Cost-Effective Learning**: Free tier and learning-friendly pricing
- **Industry Standard**: Most widely used in enterprise environments

**Cons:**
- Steeper learning curve due to service breadth
- Can be overwhelming with service choices
- Pricing complexity requires careful monitoring

## Detailed AWS Service Selection

### Compute and Orchestration

#### Amazon EKS (Elastic Kubernetes Service)
```yaml
Configuration:
  Node Groups: Fargate for serverless containers
  Auto Scaling: Horizontal Pod Autoscaler
  Load Balancing: Application Load Balancer with SSL
  Ingress: AWS Load Balancer Controller
  
Benefits:
  - Container orchestration learning
  - Production-ready scaling patterns
  - Integration with AWS services
  - Cost optimization with Fargate
```

**Alternative Considered:** ECS with Fargate
- **Chosen EKS because:** Kubernetes is industry standard, better learning value

### Database Services

#### Multi-Database Strategy
```yaml
Identity Service: Amazon RDS PostgreSQL
  Instance: db.t3.micro (free tier eligible)
  Multi-AZ: Development starts single-AZ
  Backup: 7-day automated backups
  Encryption: At-rest and in-transit

Todo Service: Amazon DynamoDB
  Billing: On-demand pricing
  Global Tables: Not needed initially
  Streams: Enabled for event processing
  Backup: Point-in-time recovery

Budget Service: Amazon DocumentDB
  Instance: db.t3.medium cluster
  Replica: Single instance initially
  Backup: Automated daily snapshots
  Encryption: TLS and at-rest encryption
```

**Learning Justification:**
- **PostgreSQL**: SQL patterns and ACID compliance
- **DynamoDB**: NoSQL access patterns and scaling
- **DocumentDB**: Document databases and aggregation pipelines

### AI and Machine Learning

#### Amazon Bedrock
```json
{
  "chosenModels": {
    "primary": "anthropic.claude-3-sonnet-20240229-v1:0",
    "backup": "amazon.titan-text-express-v1",
    "embeddings": "amazon.titan-embed-text-v1"
  },
  "capabilities": {
    "textGeneration": "Task suggestions, expense categorization",
    "embeddings": "Semantic search for todos and expenses",
    "reasoningChains": "Financial advice and budget analysis"
  },
  "pricing": {
    "model": "Pay-per-token",
    "inputTokens": "$0.003/1K tokens (Claude Sonnet)",
    "outputTokens": "$0.015/1K tokens (Claude Sonnet)"
  }
}
```

**Benefits Over Alternatives:**
- **Multiple Model Providers**: Claude, Titan, Llama in one service
- **No Infrastructure Management**: Serverless AI inference
- **Built-in Security**: IAM integration and VPC endpoints
- **Cost Predictable**: Pay-per-use pricing model

### Authentication and Authorization

#### Amazon Cognito
```yaml
User Pools:
  Configuration:
    - Email/password authentication
    - Multi-factor authentication optional
    - Password policies enforced
    - Email verification required
    
  Custom Attributes:
    - family_id: String
    - preferred_currency: String
    - ai_consent: Boolean
    
Identity Pools:
  Purpose: Temporary AWS credentials for direct service access
  Providers: User Pool as authentication provider
  Roles: 
    - AuthenticatedRole: Access to user's own data
    - UnauthenticatedRole: Limited read-only access
```

**Integration Pattern:**
```csharp
// JWT token validation in .NET services
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://cognito-idp.us-west-2.amazonaws.com/us-west-2_USERPOOLID";
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidAudience = "your-cognito-app-client-id"
        };
    });
```

### Storage and CDN

#### Amazon S3 + CloudFront
```yaml
S3 Buckets:
  homemanager-receipts-prod:
    Purpose: Receipt images and documents
    Versioning: Enabled
    Encryption: AES-256
    Lifecycle: Archive to IA after 90 days
    
  homemanager-ai-cache-prod:
    Purpose: AI model response caching
    TTL: 24 hours for most content
    Encryption: KMS encryption
    
CloudFront Distribution:
  Origin: S3 buckets
  Caching: Aggressive caching for static content
  SSL: CloudFront SSL certificate
  Compression: Enabled for text content
```

### Messaging and Events

#### Amazon EventBridge + SQS
```yaml
EventBridge:
  Custom Bus: homemanager-events
  Rules:
    - ExpenseCreated → Todo creation rule
    - BudgetExceeded → Notification rule
    - UserRegistered → Welcome workflow
    
SQS Queues:
  homemanager-ai-processing:
    Purpose: Async AI analysis tasks
    Visibility Timeout: 300 seconds
    Dead Letter Queue: Enabled
    
  homemanager-notifications:
    Purpose: Email and push notifications
    Batch Processing: Enabled
    FIFO: Not required for notifications
```

**Event-Driven Architecture:**
```csharp
// Event publishing from Budget service
public async Task<Expense> CreateExpenseAsync(CreateExpenseRequest request)
{
    var expense = await _repository.CreateAsync(expense);
    
    await _eventBridge.PutEventsAsync(new PutEventsRequest
    {
        Entries = new List<PutEventsRequestEntry>
        {
            new PutEventsRequestEntry
            {
                Source = "homemanager.budget",
                DetailType = "Expense Created",
                Detail = JsonSerializer.Serialize(new ExpenseCreatedEvent
                {
                    ExpenseId = expense.Id,
                    UserId = expense.UserId,
                    Amount = expense.Amount,
                    Category = expense.Category
                })
            }
        }
    });
    
    return expense;
}
```

### Monitoring and Observability

#### CloudWatch + X-Ray
```yaml
CloudWatch:
  Metrics:
    - Custom business metrics (expenses/day, todos completed)
    - Application performance metrics
    - Database performance metrics
    - AI service usage metrics
    
  Logs:
    - Centralized application logs
    - Structured JSON logging
    - Log retention: 30 days for development
    
  Dashboards:
    - Application health dashboard
    - Business metrics dashboard
    - Cost monitoring dashboard

X-Ray:
  Tracing: End-to-end request tracing
  Service Map: Visual service dependencies
  Performance Analysis: Bottleneck identification
  Error Analysis: Exception tracking and patterns
```

### CI/CD and Infrastructure

#### GitHub Actions + AWS CodePipeline
```yaml
GitHub Actions:
  Triggers: Push to main, pull requests
  Jobs:
    - Build and test .NET services
    - Build and push Docker images
    - Run security scans
    - Deploy to staging environment
    
AWS CodePipeline:
  Source: GitHub repository
  Build: CodeBuild for infrastructure deployment
  Deploy: CloudFormation stack updates
  Testing: Automated integration tests
```

**Infrastructure as Code:**
```yaml
# AWS CDK for infrastructure
AWS CDK Stack:
  - EKS cluster with Fargate profiles
  - RDS PostgreSQL instance
  - DynamoDB tables with GSIs
  - DocumentDB cluster
  - Cognito User Pool and Identity Pool
  - S3 buckets with policies
  - EventBridge custom bus
  - CloudWatch dashboards
```

## Cost Optimization Strategy

### Free Tier Utilization
```yaml
Always Free Services:
  - DynamoDB: 25 GB storage, 25 RCU/WCU
  - Lambda: 1M requests/month
  - CloudWatch: 10 custom metrics
  - S3: 5 GB storage
  - EventBridge: 1M events/month

12-Month Free Tier:
  - RDS: db.t3.micro 750 hours
  - EKS: Cluster management fee waived
  - Application Load Balancer: 750 hours
  - CloudFront: 50 GB data transfer
```

### Cost Monitoring
```yaml
AWS Budgets:
  Monthly Budget: $50 with 80% alert threshold
  Service-Level Budgets: 
    - AI Services: $20/month
    - Database Services: $15/month
    - Compute Services: $10/month
    
Cost Allocation Tags:
  - Service: identity|todo|budget|ai
  - Environment: dev|staging|prod
  - Owner: team-name
```

## Regional Strategy

### Primary Region: US-West-2 (Oregon)
```yaml
Justification:
  - Lower latency for West Coast users
  - Full service availability
  - Competitive pricing
  - Good connectivity to AI model endpoints
  
Service Placement:
  - All primary services in us-west-2
  - S3 Cross-Region Replication to us-east-1
  - CloudFront global edge locations
  - Future: DynamoDB Global Tables for multi-region
```

## Security Considerations

### IAM Strategy
```yaml
Principle of Least Privilege:
  Service Roles:
    - Each service has dedicated IAM role
    - Policies scoped to required resources only
    - Cross-service access via specific permissions
    
  User Access:
    - Developers: Limited AWS console access
    - Applications: Service-to-service roles only
    - Temporary credentials via AWS STS
```

### Data Encryption
```yaml
Encryption at Rest:
  - RDS: AWS KMS encryption
  - DynamoDB: Customer-managed KMS keys
  - DocumentDB: Cluster encryption enabled
  - S3: Server-side encryption (SSE-S3)
  
Encryption in Transit:
  - All service communication over TLS 1.2+
  - VPC endpoints for AWS service communication
  - Application Load Balancer SSL termination
```

### Network Security
```yaml
VPC Configuration:
  Public Subnets: Load balancers only
  Private Subnets: Application and database tiers
  Database Subnets: Database instances isolated
  
Security Groups:
  - Application tier: HTTP/HTTPS from load balancer
  - Database tier: Database port from application tier
  - No direct internet access to private resources
```

## Migration Path

### Phase 1: Basic Infrastructure
```yaml
Week 1-2:
  - VPC and networking setup
  - EKS cluster deployment
  - RDS PostgreSQL instance
  - Basic Cognito configuration
```

### Phase 2: Application Deployment
```yaml
Week 3-4:
  - Identity service deployment
  - DynamoDB table creation
  - Todo service deployment
  - Basic CI/CD pipeline
```

### Phase 3: Advanced Features
```yaml
Week 5-6:
  - DocumentDB cluster setup
  - Budget service deployment
  - EventBridge integration
  - AI service integration
```

### Phase 4: Production Readiness
```yaml
Week 7-8:
  - Monitoring and alerting
  - Security hardening
  - Performance optimization
  - Documentation completion
```

## Consequences

### Positive Consequences
1. **Comprehensive Learning**: Exposure to industry-standard AWS services
2. **AI Integration**: Native AI services reduce complexity
3. **Scalability**: Production-ready scaling patterns
4. **Cost Management**: Free tier provides learning budget
5. **Industry Relevance**: AWS skills highly valued in job market

### Negative Consequences
1. **Vendor Lock-in**: Heavy AWS integration reduces portability
2. **Complexity**: Many services to learn and integrate
3. **Cost Risk**: Easy to exceed budgets without monitoring
4. **Learning Curve**: Steep initial learning investment

### Mitigation Strategies
1. **Abstraction Layers**: Use interfaces to reduce direct AWS dependencies
2. **Cost Monitoring**: Strict budgets and alerts
3. **Documentation**: Comprehensive guides for each service
4. **Gradual Adoption**: Phase rollout to manage complexity

## Related ADRs
- [ADR-001: Microservices Approach](001-microservices-approach.md)
- [ADR-002: Database Strategy](002-database-strategy.md)
- [ADR-004: AI Integration Strategy](004-ai-integration.md)

---

**AI Development Notes:**
- AWS SDK patterns documented for consistent AI-assisted development
- Service-specific configuration templates for AI code generation
- Clear separation of concerns for each AWS service integration
- Cost optimization patterns built into architecture decisions