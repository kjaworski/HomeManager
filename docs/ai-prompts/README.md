# AI-Assisted Development Prompts and Context

This directory contains carefully crafted prompts and context files designed to optimize AI-assisted development of the HomeManager system. These prompts are optimized for GitHub Copilot, Claude Sonnet, GPT models, and other AI development assistants.

## AI Development Strategy

The HomeManager project is designed from the ground up to leverage AI-assisted development efficiently. Our approach focuses on:

1. **Clear Context Provision** - Comprehensive documentation and examples
2. **Consistent Patterns** - Repeatable code patterns across services
3. **Explicit Requirements** - Well-defined interfaces and contracts
4. **Incremental Development** - Building complexity gradually with AI assistance

## Prompt Categories

### Service Implementation Prompts
- **[Identity Service Prompts](service-prompts/identity-service.md)** - Authentication, JWT, and user management
- **[Todo Service Prompts](service-prompts/todo-service.md)** - DynamoDB integration and task management
- **[Budget Service Prompts](service-prompts/budget-service.md)** - MongoDB integration and financial calculations
- **[AI Gateway Prompts](service-prompts/ai-gateway.md)** - Amazon Bedrock and MCP server integration

### Frontend Development Prompts
- **[React Web App Prompts](frontend-prompts/react-web.md)** - Next.js, state management, and UI components
- **[React Native Prompts](frontend-prompts/react-native.md)** - Mobile-specific features and navigation
- **[Shared Components Prompts](frontend-prompts/shared-components.md)** - Design system and reusable components

### Infrastructure Prompts
- **[AWS Infrastructure Prompts](infrastructure-prompts/aws-setup.md)** - EKS, databases, and cloud services
- **[CI/CD Pipeline Prompts](infrastructure-prompts/cicd-setup.md)** - GitHub Actions and deployment automation
- **[Monitoring Prompts](infrastructure-prompts/monitoring-setup.md)** - Observability and alerting

## AI Context Files

### Project Context
Essential context that should be provided to AI assistants:

#### System Overview Context
```markdown
# HomeManager System Context for AI Development

## Project Overview
HomeManager is a distributed microservices application for family home management including:
- Todo/task management with AI-powered suggestions
- Budget/expense tracking with intelligent categorization
- User authentication and family management
- AI-powered insights and recommendations

## Technology Stack
- Backend: .NET 10, ASP.NET Core WebAPI, C# latest
- Databases: PostgreSQL (Identity), DynamoDB (Todos), DocumentDB/MongoDB (Budget)
- Cloud: AWS (EKS, RDS, DynamoDB, DocumentDB, Bedrock)
- Frontend: React 18 + TypeScript (Web), React Native + TypeScript (Mobile)
- AI: Amazon Bedrock with Claude 3 Sonnet and Titan models
- Architecture: Microservices with event-driven communication

## Key Patterns
- Repository pattern for data access
- Service layer for business logic
- Event-driven communication between services
- JWT authentication with Cognito
- OpenAPI specifications for all APIs
- Comprehensive error handling and logging
```

#### Development Patterns Context
```csharp
// Standard service structure pattern for AI reference
namespace HomeManager.{ServiceName}
{
    // Controller pattern
    [ApiController]
    [Route("api/[controller]")]
    [Authorize]
    public class {Entity}Controller : ControllerBase
    {
        private readonly I{Entity}Service _service;
        
        public {Entity}Controller(I{Entity}Service service)
        {
            _service = service;
        }
        
        [HttpPost]
        public async Task<ActionResult<{Entity}Response>> Create{Entity}Async([FromBody] Create{Entity}Request request)
        {
            var result = await _service.Create{Entity}Async(request);
            return CreatedAtAction(nameof(Get{Entity}), new { id = result.Id }, result);
        }
    }
    
    // Service pattern
    public class {Entity}Service : I{Entity}Service
    {
        private readonly I{Entity}Repository _repository;
        private readonly ILogger<{Entity}Service> _logger;
        
        public {Entity}Service(I{Entity}Repository repository, ILogger<{Entity}Service> logger)
        {
            _repository = repository;
            _logger = logger;
        }
        
        public async Task<{Entity}> Create{Entity}Async(Create{Entity}Request request)
        {
            // Validation
            // Business logic
            // Data persistence
            // Event publishing
        }
    }
}
```

### Database Context Files

#### DynamoDB Patterns
```typescript
// DynamoDB access patterns for AI reference
interface TodoDynamoDBItem {
  PK: string;                    // USER#{userId}
  SK: string;                    // TODO#{date}#{priority}#{id}
  GSI1PK: string;               // FAMILY#{familyId}
  GSI1SK: string;               // {dueDate}
  id: string;
  title: string;
  description: string;
  status: 'pending' | 'completed' | 'cancelled';
  priority: 'low' | 'medium' | 'high';
  assignedTo: string;
  dueDate: string;
  createdAt: string;
  updatedAt: string;
}

// Access pattern examples
const accessPatterns = {
  getUserTodos: "Query PK = USER#{userId}",
  getFamilyTodos: "Query GSI1PK = FAMILY#{familyId}",
  getTodosByDueDate: "Query GSI1SK = {dueDate}",
  getHighPriorityTodos: "Query SK begins_with TODO#{date}#HIGH"
};
```

#### MongoDB Patterns
```typescript
// MongoDB document structure for AI reference
interface ExpenseDocument {
  _id: ObjectId;
  userId: string;
  familyId: string;
  amount: number;
  currency: string;
  category: {
    primary: string;
    secondary: string;
    aiGenerated: boolean;
    confidence: number;
  };
  merchant: string;
  description: string;
  receipt?: {
    imageUrl: string;
    ocrText: string;
    aiAnalysis: object;
  };
  location?: {
    type: "Point";
    coordinates: [number, number];
  };
  aiInsights: {
    isRecurring: boolean;
    pattern: string;
    suggestions: string[];
  };
  createdAt: Date;
  updatedAt: Date;
}
```

## Prompt Templates

### Service Implementation Template
```markdown
# Service Implementation Prompt Template

I need to implement the {ServiceName} for the HomeManager system.

## Context
- Service: {ServiceName} (e.g., Budget Service, Todo Service)
- Database: {DatabaseType} (PostgreSQL/DynamoDB/MongoDB)
- Framework: .NET 10, ASP.NET Core WebAPI
- Architecture: Microservices with repository pattern

## Requirements
{Specific requirements for the service}

## Implementation Needs
- [ ] Domain models and DTOs
- [ ] Repository interface and implementation
- [ ] Service layer with business logic
- [ ] API controllers with OpenAPI documentation
- [ ] Validation and error handling
- [ ] Unit and integration tests

## Patterns to Follow
{Reference to specific patterns from the implementation guides}

Please implement following the established patterns in the HomeManager codebase, ensuring proper error handling, logging, and testing.
```

### Frontend Component Template
```markdown
# Frontend Component Prompt Template

I need to implement a {ComponentName} component for the HomeManager React application.

## Context
- Framework: React 18 + TypeScript
- Styling: Tailwind CSS + Headless UI
- State: Zustand (global) + React Query (server state)
- Platform: {Web/Mobile/Shared}

## Requirements
{Specific component requirements}

## Design System
- Follow HomeManager design tokens
- Use shared UI components when possible
- Implement responsive design
- Include loading and error states

## Integration
- Use typed API client
- Implement proper error handling
- Include accessibility features
- Follow React best practices

Please implement with TypeScript, proper error handling, and following the established patterns.
```

### AI Integration Template
```markdown
# AI Integration Prompt Template

I need to implement AI functionality for {FeatureName} in the HomeManager system.

## Context
- AI Platform: Amazon Bedrock
- Model: Claude 3 Sonnet (primary), Titan (backup)
- Integration: MCP (Model Context Protocol) servers
- Service: AI Gateway Service

## Requirements
{Specific AI requirements}

## Implementation Pattern
- Use AI Gateway Service abstraction
- Implement caching for cost optimization
- Include fallback mechanisms
- Add usage tracking and cost monitoring

## Expected Input/Output
{Define clear input/output structures}

Please implement following the AI integration patterns, including proper error handling and cost optimization.
```

## Development Workflow with AI

### Phase-Based Development
Each development phase includes specific AI prompts:

#### Phase 1: Foundation Setup
```markdown
**Week 1 Prompts:**
- AWS infrastructure setup prompts
- Identity service implementation prompts
- Basic authentication flow prompts
- Database setup and configuration prompts

**Week 2 Prompts:**
- React application foundation prompts
- API client library implementation prompts
- Authentication UI component prompts
- CI/CD pipeline setup prompts
```

### AI Assistance Best Practices

#### Context Preparation
1. **Provide System Overview** - Always include the HomeManager system context
2. **Reference Patterns** - Point to specific implementation guides and patterns
3. **Include Examples** - Use existing code examples as reference
4. **Specify Constraints** - Clearly define requirements and limitations

#### Incremental Development
1. **Start Simple** - Begin with basic functionality
2. **Add Complexity Gradually** - Build features incrementally
3. **Test Each Step** - Validate before moving to next feature
4. **Refactor with AI** - Use AI to optimize and improve code

#### Quality Assurance
1. **Request Code Reviews** - Ask AI to review generated code
2. **Ask for Tests** - Generate comprehensive test cases
3. **Verify Patterns** - Ensure consistency with established patterns
4. **Optimize Performance** - Request performance improvements

## AI Model Optimization

### GitHub Copilot
- Use descriptive variable and function names
- Write clear comments explaining business logic
- Structure code with consistent patterns
- Implement interfaces before implementations

### Claude/GPT Prompts
- Provide comprehensive context in each prompt
- Ask for specific code patterns and explanations
- Request documentation and examples
- Iterate and refine based on feedback

### Model Context Protocol (MCP)
- Implement MCP servers for reusable AI functionality
- Create standardized interfaces for AI operations
- Build knowledge bases for domain-specific AI assistance
- Enable AI chaining for complex operations

## Continuous Improvement

### Prompt Refinement
- Track which prompts produce the best results
- Refine templates based on development experience
- Share successful patterns with the team
- Update documentation based on AI interactions

### Pattern Evolution
- Identify common AI-generated patterns
- Standardize successful approaches
- Update implementation guides
- Enhance AI context files

---

**Usage Instructions:**
1. Use these prompts as starting points for AI interactions
2. Customize templates for specific requirements
3. Provide comprehensive context for better results
4. Iterate and refine based on AI feedback
5. Document successful patterns for future use

This directory serves as a comprehensive resource for AI-assisted development of the HomeManager system, designed to maximize productivity while maintaining code quality and consistency.