# Budget Service Implementation Guide

> **AI Development Context** 🤖  
> MongoDB-based expense tracking and budget management service

## Overview

The Budget Service handles expense tracking, budget management, and financial analytics for families. It uses MongoDB for flexible document storage and provides AI-powered spending insights.

## Technology Stack

```yaml
Framework: ASP.NET Core 9 with Minimal APIs
Database: MongoDB (DocumentDB on AWS)
Caching: Redis
AI Integration: Amazon Bedrock for financial insights
ORM: MongoDB.Driver
Validation: FluentValidation
Testing: xUnit with Testcontainers
```

## Implementation Highlights

### Document Models
```csharp
public class Expense
{
    public ObjectId Id { get; set; }
    public string FamilyId { get; set; } = null!;
    public decimal Amount { get; set; }
    public string Description { get; set; } = null!;
    public string Category { get; set; } = null!;
    public DateTime Date { get; set; }
    public string CreatedBy { get; set; } = null!;
    public string? ReceiptUrl { get; set; }
    public List<string> Tags { get; set; } = new();
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
}

public class Budget
{
    public ObjectId Id { get; set; }
    public string FamilyId { get; set; } = null!;
    public string Category { get; set; } = null!;
    public decimal Amount { get; set; }
    public BudgetPeriod Period { get; set; }
    public DateTime StartDate { get; set; }
    public DateTime EndDate { get; set; }
    public decimal Spent { get; set; }
    public decimal Remaining => Amount - Spent;
    public float PercentUsed => Amount > 0 ? (float)(Spent / Amount * 100) : 0;
}
```

### Key Features
- Receipt image processing with AI
- Automatic expense categorization
- Budget tracking and alerts
- Spending pattern analysis
- Financial insights and recommendations
- Multi-currency support
- Recurring expense tracking