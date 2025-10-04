# ADR-002: Database Strategy - Polyglot Persistence

**Status:** Accepted  
**Date:** 2024-10-03  
**Decision Makers:** Development Team  
**AI Development Context:** ✅ Optimized for learning NoSQL patterns with AI assistance

## Context

We need to choose database technologies for each microservice in our HomeManager system. The goal is to gain experience with NoSQL databases while maintaining appropriate consistency models for each domain.

## Decision

We will implement a **polyglot persistence strategy** using different database technologies optimized for each service's specific needs:

- **Identity Service**: Amazon RDS PostgreSQL
- **Todo Service**: Amazon DynamoDB  
- **Budget Service**: Amazon DocumentDB (MongoDB)

## Options Considered

### Option 1: Single Database for All Services
```sql
-- PostgreSQL for everything
HomeManager Database:
├── users
├── families  
├── todos
├── expenses
├── budgets
└── ai_interactions
```

**Pros:**
- Simple to manage and deploy
- ACID transactions across all domains
- Consistent tooling and expertise
- Easier joins across domains

**Cons:**
- Single point of failure
- Scaling bottleneck
- No learning opportunity for NoSQL
- Not optimized for different access patterns

### Option 2: NoSQL for Everything
```json
// DynamoDB for all services
{
  "PK": "USER#123 | TODO#456 | EXPENSE#789",
  "SK": "METADATA | FAMILY#123 | CATEGORY#456",
  "data": { /* flexible schema */ }
}
```

**Pros:**
- Consistent NoSQL patterns
- High scalability
- Flexible schemas
- Learning opportunity

**Cons:**
- Complex for relational data (users, families)
- No ACID guarantees for critical operations
- Steep learning curve for all domains
- Over-engineering for some use cases

### Option 3: Polyglot Persistence ✅ **CHOSEN**
```yaml
Identity Service: RDS PostgreSQL
  - Strong consistency for authentication
  - Relational data (users ↔ families)
  - ACID compliance for security operations

Todo Service: DynamoDB
  - Fast read/write for task operations
  - Flexible schema for task metadata
  - Global secondary indexes for queries

Budget Service: DocumentDB (MongoDB)
  - Document storage for complex receipts
  - Aggregation pipeline for financial reports
  - Flexible schema for expense data
```

## Detailed Service Database Decisions

### Identity Service: RDS PostgreSQL

**Rationale:**
```sql
-- Strong consistency requirements
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    family_id UUID REFERENCES families(id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE families (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    owner_id UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW()
);

-- ACID compliance for security operations
BEGIN TRANSACTION;
    INSERT INTO users (...);
    INSERT INTO family_members (...);
    UPDATE family_member_count (...);
COMMIT;
```

**Benefits:**
- **ACID Compliance**: Critical for authentication and authorization
- **Relational Integrity**: User-family relationships
- **Mature Ecosystem**: Well-established identity patterns
- **SQL Familiarity**: Easier debugging and administration

**AWS Service:** Amazon RDS PostgreSQL with Multi-AZ for high availability

### Todo Service: DynamoDB

**Rationale:**
```json
// Optimized for todo access patterns
{
  "PK": "USER#123",                    // Get all todos for user
  "SK": "TODO#2024-10-03#HIGH#456",   // Sort by date, priority
  "GSI1PK": "FAMILY#789",             // Family-wide todo view
  "GSI1SK": "2024-10-03",             // Due date queries
  "title": "Fix kitchen faucet",
  "description": "The faucet is dripping",
  "status": "pending",
  "priority": "high",
  "assigned_to": "USER#124",
  "tags": ["plumbing", "urgent"],
  "created_at": "2024-10-03T10:00:00Z",
  "due_date": "2024-10-05T18:00:00Z",
  "ai_metadata": {
    "estimated_duration": 60,
    "difficulty": "medium",
    "tools_needed": ["wrench", "plumber_tape"]
  }
}
```

**Access Patterns:**
```javascript
// Primary patterns optimized in DynamoDB design
1. Get all todos for user: Query PK = "USER#123"
2. Get todos by due date: Query GSI1 where GSI1SK = "2024-10-03"
3. Get family todos: Query GSI1 where GSI1PK = "FAMILY#789"
4. Get high priority todos: Query with SK begins_with "TODO#date#HIGH"
```

**Benefits:**
- **Performance**: Single-digit millisecond response times
- **Scalability**: Automatic scaling to handle traffic spikes
- **Flexible Schema**: Easy to add AI metadata and new fields
- **Cost Effective**: Pay only for consumed capacity

**AWS Service:** Amazon DynamoDB with Global Secondary Indexes

### Budget Service: DocumentDB (MongoDB)

**Rationale:**
```json
// Complex financial documents
{
  "_id": "expense_123",
  "user_id": "USER#123",
  "family_id": "FAMILY#789",
  "amount": 156.78,
  "currency": "USD",
  "category": {
    "primary": "groceries",
    "subcategory": "weekly_shopping",
    "ai_detected": true,
    "confidence": 0.92
  },
  "receipt": {
    "store": "Whole Foods Market",
    "location": "Downtown Seattle",
    "items": [
      {
        "name": "Organic Milk",
        "price": 6.99,
        "quantity": 2,
        "category": "dairy",
        "unit": "gallon"
      },
      {
        "name": "Sourdough Bread",
        "price": 4.50,
        "quantity": 1,
        "category": "bakery"
      }
    ],
    "payment_method": "credit_card_ending_1234",
    "cashback": 1.56
  },
  "location": {
    "type": "Point",
    "coordinates": [-122.3328, 47.6061]
  },
  "ai_insights": {
    "spending_pattern": "normal",
    "budget_impact": "within_limits",
    "suggestions": [
      "Consider bulk buying for frequently purchased items",
      "Similar store prices in your area: Target $5.99, Safeway $7.29"
    ],
    "predicted_next_purchase": "2024-10-10"
  },
  "attachments": [
    {
      "type": "receipt_image",
      "s3_url": "s3://homemanager-receipts/user123/receipt_123.jpg"
    }
  ],
  "created_at": "2024-10-03T14:30:00Z",
  "updated_at": "2024-10-03T14:30:00Z"
}
```

**Aggregation Capabilities:**
```javascript
// MongoDB aggregation pipeline for financial reports
db.expenses.aggregate([
  {
    $match: {
      user_id: "USER#123",
      created_at: {
        $gte: new Date("2024-10-01"),
        $lt: new Date("2024-11-01")
      }
    }
  },
  {
    $group: {
      _id: "$category.primary",
      total_amount: { $sum: "$amount" },
      transaction_count: { $sum: 1 },
      avg_amount: { $avg: "$amount" }
    }
  },
  {
    $sort: { total_amount: -1 }
  }
]);
```

**Benefits:**
- **Document Storage**: Natural fit for complex receipt data
- **Aggregation Pipeline**: Powerful financial reporting capabilities
- **Flexible Schema**: Easy to add AI insights and new fields
- **Geospatial**: Location-based expense analysis

**AWS Service:** Amazon DocumentDB (MongoDB-compatible)

## Cross-Service Data Consistency

### Eventual Consistency Pattern
```csharp
// Handle cross-service data via events
public class ExpenseCreatedEvent
{
    public string ExpenseId { get; set; }
    public string UserId { get; set; }
    public decimal Amount { get; set; }
    public string Category { get; set; }
    public DateTime CreatedAt { get; set; }
}

// Todo service can react to budget events
public class BudgetEventHandler
{
    public async Task Handle(ExpenseCreatedEvent @event)
    {
        // Create todo for expensive purchases
        if (@event.Amount > 500)
        {
            await _todoService.CreateTodoAsync(new CreateTodoRequest
            {
                Title = $"Review large expense: {@event.Category}",
                Description = $"Amount: ${@event.Amount}",
                Priority = "medium",
                AssignedTo = @event.UserId
            });
        }
    }
}
```

### Data Synchronization Strategies
```yaml
User Data Consistency:
  - Identity service is source of truth for user data
  - Other services cache user info with TTL
  - Use events for user profile changes

Reference Data:
  - Categories managed in Budget service
  - Replicated to other services as needed
  - Eventual consistency acceptable

Transactional Data:
  - Each service owns its domain data
  - Cross-service operations via saga patterns
  - Compensation actions for failures
```

## Caching Strategy

### Redis Integration
```csharp
// Shared cache layer for frequently accessed data
public class UserCacheService
{
    private readonly IDistributedCache _cache;
    
    public async Task<User> GetUserAsync(string userId)
    {
        var cacheKey = $"user:{userId}";
        var cached = await _cache.GetStringAsync(cacheKey);
        
        if (cached != null)
            return JsonSerializer.Deserialize<User>(cached);
            
        // Cache miss - get from Identity service
        var user = await _identityService.GetUserAsync(userId);
        await _cache.SetStringAsync(cacheKey, 
            JsonSerializer.Serialize(user),
            TimeSpan.FromMinutes(15));
            
        return user;
    }
}
```

## Migration and Backup Strategies

### Database-Specific Backup
```yaml
RDS PostgreSQL:
  - Automated backups with point-in-time recovery
  - Cross-region snapshots for disaster recovery
  - Read replicas for backup without performance impact

DynamoDB:
  - Point-in-time recovery (PITR)
  - On-demand backups for major changes
  - DynamoDB Streams for change capture

DocumentDB:
  - Automated backups with 7-day retention
  - Manual snapshots before major updates
  - Cross-region replication for disaster recovery
```

## Development and Testing

### Local Development
```yaml
Docker Compose Setup:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: homemanager_identity
      
  dynamodb-local:
    image: amazon/dynamodb-local
    ports:
      - "8000:8000"
      
  mongodb:
    image: mongo:7
    environment:
      MONGO_INITDB_DATABASE: homemanager_budget
```

### Data Seeding
```csharp
// Consistent test data across databases
public class DatabaseSeeder
{
    public async Task SeedAsync()
    {
        // Seed PostgreSQL with test users and families
        await SeedIdentityDataAsync();
        
        // Seed DynamoDB with sample todos
        await SeedTodoDataAsync();
        
        // Seed MongoDB with sample expenses
        await SeedBudgetDataAsync();
    }
}
```

## Performance Considerations

### Database-Specific Optimizations
```yaml
PostgreSQL (Identity):
  - Connection pooling with PgBouncer
  - Indexes on frequently queried columns
  - Read replicas for read-heavy operations

DynamoDB (Todo):
  - Proper partition key design to avoid hot partitions
  - Global secondary indexes for alternate access patterns
  - DynamoDB Accelerator (DAX) for caching

DocumentDB (Budget):
  - Compound indexes for aggregation queries
  - Connection pooling for .NET driver
  - Read preference for analytics queries
```

## Consequences

### Positive Consequences
1. **Learning Opportunity**: Hands-on experience with multiple database types
2. **Optimized Performance**: Each database chosen for its strengths
3. **Independent Scaling**: Scale databases based on service needs
4. **Technology Diversity**: Exposure to SQL and NoSQL patterns

### Negative Consequences
1. **Increased Complexity**: Multiple database technologies to learn and manage
2. **Cross-Service Queries**: No joins across services
3. **Operational Overhead**: Different backup and monitoring strategies
4. **Data Consistency**: Eventually consistent cross-service operations

### Mitigation Strategies
1. **Comprehensive Documentation**: Clear patterns for each database
2. **Shared Utilities**: Common patterns in shared libraries
3. **Monitoring**: Database-specific monitoring and alerting
4. **Event Sourcing**: Clear audit trail for cross-service operations

## Related ADRs
- [ADR-001: Microservices Approach](001-microservices-approach.md)
- [ADR-003: AWS Cloud Provider](003-aws-cloud-provider.md)
- [ADR-004: AI Integration Strategy](004-ai-integration.md)

---

**AI Development Notes:**
- Each database follows established patterns for easier AI code generation
- Consistent error handling across different database types
- Clear data models for each service to guide AI-assisted development
- Database-specific optimization patterns documented for AI reference