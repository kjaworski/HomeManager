# ADR-001: Microservices Architecture Approach

**Status:** Accepted  
**Date:** 2024-10-03  
**Decision Makers:** Development Team  
**AI Development Context:** ✅ Optimized for GitHub Copilot and LLM assistance

## Context

We need to choose the overall architecture pattern for the HomeManager system. The system will include:
- User identity and family management
- Todo/task management with AI features
- Budget tracking and financial insights
- Future trip planning capabilities
- Multiple React frontend applications

## Decision

We will implement a **microservices architecture** with domain-driven design principles.

## Options Considered

### Option 1: Monolithic Architecture
```csharp
// Single ASP.NET Core application
HomeManager.Api/
├── Controllers/
│   ├── AuthController.cs
│   ├── TodoController.cs
│   ├── BudgetController.cs
│   └── AIController.cs
├── Services/
└── Data/
```

**Pros:**
- Simpler initial development
- Easier local development setup
- Single deployment unit
- ACID transactions across domains

**Cons:**
- Tight coupling between domains
- Single point of failure
- Difficult to scale individual components
- Technology lock-in
- Large team coordination challenges

### Option 2: Modular Monolith
```csharp
// Single deployment with modular boundaries
HomeManager.Api/
├── Modules/
│   ├── Identity/
│   ├── Todo/
│   ├── Budget/
│   └── AI/
└── Shared/
```

**Pros:**
- Clear domain boundaries
- Simpler than microservices
- Easy to extract to microservices later
- Shared infrastructure

**Cons:**
- Still single deployment unit
- Shared database complications
- Module boundary enforcement challenges
- Less learning opportunity for distributed systems

### Option 3: Microservices Architecture ✅ **CHOSEN**
```yaml
Services:
  - Identity Service (RDS PostgreSQL)
  - Todo Service (DynamoDB)
  - Budget Service (DocumentDB)
  - AI Gateway (MCP Server)
  - API Gateway (YARP)

Frontend:
  - Todo React App
  - Budget React App
  - Trips React App (future)
```

**Pros:**
- Independent scaling and deployment
- Technology diversity (SQL + NoSQL)
- Team autonomy
- Fault isolation
- Learning opportunity for distributed systems
- AI service isolation
- Better for polyglot persistence

**Cons:**
- Increased complexity
- Network communication overhead
- Distributed transaction challenges
- More DevOps overhead
- Initial development complexity

## Rationale

### Technical Reasons
1. **Polyglot Persistence**: Different domains have different data needs
   - Identity: Strong consistency (SQL)
   - Todo: Fast reads/writes (NoSQL)
   - Budget: Complex documents (MongoDB)

2. **AI Integration**: Separate AI services allow for:
   - Independent AI model updates
   - Service-specific AI optimization
   - MCP server protocol implementation

3. **Scalability**: Different services have different scaling needs
   - Todo service: High read/write frequency
   - Budget service: Complex analytical queries
   - AI services: Compute-intensive operations

### Learning Goals Alignment
1. **Cloud-Native Development**: Microservices align with AWS services
2. **NoSQL Experience**: Different services can use different databases
3. **AI Integration**: MCP servers fit naturally with microservices
4. **DevOps Practices**: Each service has its own CI/CD pipeline

### Future Flexibility
1. **Technology Evolution**: Can adopt new tech per service
2. **Team Growth**: Teams can own specific services
3. **Third-party Integration**: Easier to integrate external services

## Implementation Strategy

### Phase 1: Start with Key Services
```yaml
Initial Services:
  - Identity Service (Foundation)
  - Todo Service (Core functionality)
  - API Gateway (Routing)
```

### Phase 2: Add Complexity
```yaml
Additional Services:
  - Budget Service
  - AI Gateway with MCP servers
  - Notification Service
```

### Phase 3: Advanced Features
```yaml
Advanced Services:
  - Trip Planning Service
  - Analytics Service
  - Recommendation Engine
```

## Service Boundaries

### Identity Service
```yaml
Responsibilities:
  - User authentication and authorization
  - Family management
  - User profiles and preferences
  - JWT token management

Data Owned:
  - Users table
  - Families table
  - User roles and permissions
```

### Todo Service
```yaml
Responsibilities:
  - Task creation and management
  - Task assignment and tracking
  - Due date management
  - Task categories and tags

Data Owned:
  - Todo items
  - Task categories
  - Task assignments
  - Recurring task patterns
```

### Budget Service
```yaml
Responsibilities:
  - Expense tracking
  - Budget planning and monitoring
  - Financial categorization
  - Expense reports and analytics

Data Owned:
  - Expenses and income
  - Budget plans
  - Financial categories
  - Spending patterns
```

### AI Gateway Service
```yaml
Responsibilities:
  - MCP server coordination
  - Cross-service AI operations
  - Context aggregation for AI
  - AI model management

Data Owned:
  - AI conversation history
  - User AI preferences
  - AI model configurations
  - MCP server registrations
```

## Communication Patterns

### Synchronous Communication
```csharp
// HTTP calls for real-time requests
public async Task<TodoResponse> GetTodos(string userId)
{
    // Validate user via Identity service
    var user = await _identityClient.ValidateUserAsync(userId);
    
    // Get todos from Todo service
    var todos = await _todoClient.GetUserTodosAsync(userId);
    
    return todos;
}
```

### Asynchronous Communication
```csharp
// Events for eventual consistency
public async Task CreateExpense(CreateExpenseRequest request)
{
    var expense = await _repository.CreateAsync(request);
    
    // Publish event for other services
    await _eventBus.PublishAsync(new ExpenseCreatedEvent
    {
        UserId = expense.UserId,
        Amount = expense.Amount,
        Category = expense.Category
    });
}
```

## Consequences

### Positive Consequences
1. **Independent Development**: Teams can work on different services
2. **Technology Flexibility**: Can use best tool for each domain
3. **Scalability**: Scale services independently based on load
4. **Fault Isolation**: One service failure doesn't bring down system
5. **Learning Opportunity**: Hands-on experience with distributed systems

### Negative Consequences
1. **Increased Complexity**: More moving parts to manage
2. **Network Latency**: Inter-service communication overhead
3. **Data Consistency**: Need to handle eventual consistency
4. **DevOps Overhead**: Multiple deployments and monitoring
5. **Development Setup**: More complex local development environment

### Mitigation Strategies
1. **Use Docker Compose**: Simplify local development
2. **API Gateway**: Single entry point for frontends
3. **Service Mesh**: Handle cross-cutting concerns
4. **Monitoring**: Comprehensive observability stack
5. **Documentation**: Clear service contracts and boundaries

## Compliance

This decision supports:
- **Learning Goals**: Microservices, cloud-native, AI integration
- **Technology Stack**: .NET 10, AWS, NoSQL databases
- **AI Development**: MCP servers, polyglot persistence
- **DevOps**: CI/CD per service, container deployment

## Related ADRs
- [ADR-002: Database Strategy](002-database-strategy.md)
- [ADR-003: AWS Cloud Provider](003-aws-cloud-provider.md)
- [ADR-004: AI Integration Strategy](004-ai-integration.md)

---

**AI Development Notes:**
- Each service follows similar patterns for consistency
- Service templates available for rapid scaffolding
- Clear boundaries reduce cognitive load for AI-assisted development
- Consistent error handling and telemetry across services