# Budget Service AI Development Prompts

> **AI Context File for HomeManager Budget Service**

## Service Context
```markdown
# Budget Service Context for AI Development

## Purpose
Expense tracking, budget management, and financial analytics with AI insights.

## Technology Stack
- Framework: ASP.NET Core 9 with Minimal APIs
- Database: MongoDB (DocumentDB on AWS)
- AI: Amazon Bedrock for financial insights
- OCR: Amazon Textract for receipt processing
- Testing: xUnit with Testcontainers

## Key Features
- Expense tracking and categorization
- Budget creation and monitoring
- Receipt scanning and processing
- Financial insights and analytics
- Multi-currency support
- Spending pattern analysis
```

## Core Prompts

### Service Implementation
```markdown
Create a Budget Service for HomeManager using MongoDB with:
- Expense CRUD operations
- Budget management
- Receipt image processing
- AI-powered categorization
- Financial analytics
- Multi-currency support

Requirements:
- .NET 10 with minimal APIs
- MongoDB with flexible document schema
- Amazon Textract integration
- Clean Architecture patterns
- Comprehensive validation
- OpenAPI documentation
```

### Document Design
```markdown
Design MongoDB document schemas for HomeManager budget service:

Collections:
- Expenses (transactions, receipts, categories)
- Budgets (limits, periods, tracking)
- Categories (custom family categories)
- Analytics (aggregated insights)

Features:
- Flexible schema for different expense types
- Embedded documents for related data
- Indexes for query performance
- Aggregation pipelines for analytics
```