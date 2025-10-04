# Todo Service AI Development Prompts

> **AI Context File for HomeManager Todo Service**

## Service Context
```markdown
# Todo Service Context for AI Development

## Purpose
Task creation, assignment, and tracking with AI-powered suggestions and family coordination.

## Technology Stack
- Framework: ASP.NET Core 9 with Minimal APIs
- Database: Amazon DynamoDB
- Caching: Redis
- AI: Amazon Bedrock integration
- Testing: xUnit with Testcontainers

## Key Features
- Multi-family task management
- AI-powered task suggestions
- Recurring task automation
- Assignment and delegation
- Progress tracking and analytics
- Mobile and web synchronization
```

## Core Prompts

### Service Implementation
```markdown
Create a Todo Service for HomeManager using DynamoDB with the following features:
- CRUD operations for todos
- Family-scoped access control
- AI-powered task suggestions
- Recurring task management
- Assignment and delegation
- Progress tracking
- Category management

Requirements:
- .NET 10 with minimal APIs
- DynamoDB with single-table design
- Clean Architecture patterns
- Comprehensive error handling
- OpenAPI documentation
- Unit and integration tests
```

### DynamoDB Design
```markdown
Design a DynamoDB single-table structure for HomeManager todos:

Access patterns:
- Get todo by ID
- Get todos by family
- Get todos by assignee
- Get todos by status
- Get recurring todos
- Get overdue todos

Entities:
- Todo (main entity)
- TodoCategory
- RecurringTemplate
- TodoHistory

Generate:
- Table schema with GSI design
- Entity mapping strategies
- Query patterns
- Repository implementation
```