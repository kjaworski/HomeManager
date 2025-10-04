# ADR-004: AI Integration Strategy with Amazon Bedrock

**Status:** Accepted  
**Date:** 2024-10-03  
**Decision Makers:** Development Team  
**AI Development Context:** ✅ Optimized for AI-assisted development and MCP server integration

## Context

We need to integrate AI capabilities throughout the HomeManager application to provide intelligent features like expense categorization, budget analysis, task prioritization, and proactive recommendations. The goal is to create a system that learns from user behavior and provides valuable insights.

## Decision

We will implement AI integration using **Amazon Bedrock** as the primary AI platform, with a dedicated **AI Gateway Service** that implements **Model Context Protocol (MCP) servers** for structured AI interactions.

## Options Considered

### Option 1: Direct OpenAI API Integration
```csharp
// Direct API calls to OpenAI
public class ExpenseCategorizationService
{
    private readonly HttpClient _httpClient;
    
    public async Task<string> CategorizeExpenseAsync(string description, decimal amount)
    {
        var prompt = $"Categorize this expense: {description}, Amount: ${amount}";
        var response = await _httpClient.PostAsync("https://api.openai.com/v1/chat/completions", content);
        return ParseCategory(response);
    }
}
```

**Pros:**
- Direct access to latest GPT models
- Comprehensive API documentation
- Large community and examples
- Advanced features like function calling

**Cons:**
- Vendor lock-in to OpenAI
- API key management complexity
- No cloud provider integration
- Higher latency for AWS-hosted services
- Limited model diversity

### Option 2: Azure OpenAI Service
```csharp
// Azure OpenAI integration
services.AddAzureOpenAI(options =>
{
    options.Endpoint = "https://homemanager.openai.azure.com/";
    options.ApiKey = configuration["AzureOpenAI:ApiKey"];
    options.DeploymentName = "gpt-4-deployment";
});
```

**Pros:**
- Enterprise-grade security and compliance
- Integration with Azure services
- SLA guarantees
- Regional data residency

**Cons:**
- Limited to Microsoft ecosystem
- Requires Azure infrastructure
- Model availability depends on region
- Higher cost than direct OpenAI

### Option 3: Amazon Bedrock with MCP Servers ✅ **CHOSEN**
```csharp
// AI Gateway Service with MCP protocol
public class AIGatewayService
{
    private readonly BedrockRuntimeClient _bedrockClient;
    private readonly IMcpServerManager _mcpManager;
    
    public async Task<ExpenseAnalysisResult> AnalyzeExpenseAsync(AnalyzeExpenseRequest request)
    {
        // Use MCP server for structured AI interaction
        var mcpServer = await _mcpManager.GetServerAsync("expense-analysis");
        var result = await mcpServer.InvokeToolAsync("categorize_expense", request);
        return result;
    }
}
```

**Pros:**
- **Multiple Model Providers**: Claude, Titan, Llama, Cohere in one platform
- **AWS Integration**: Native integration with other AWS services
- **MCP Protocol**: Structured, reusable AI interactions
- **Cost Optimization**: Per-token pricing with no base costs
- **Security**: IAM integration and VPC endpoints
- **Serverless**: No infrastructure management

## Detailed AI Architecture

### AI Gateway Service Design

#### Core Components
```yaml
AI Gateway Service:
  Components:
    - Bedrock Runtime Client
    - MCP Server Manager
    - AI Response Cache (Redis)
    - Request/Response Logging
    - Token Usage Tracking
    - Error Handling & Retry Logic
    
  Responsibilities:
    - Abstract AI model interactions
    - Implement prompt engineering patterns
    - Cache frequently requested analyses
    - Track usage and costs
    - Handle rate limiting and retries
    - Provide consistent error responses
```

#### MCP Server Implementation
```csharp
// MCP Server for expense analysis
public class ExpenseAnalysisMcpServer : IMcpServer
{
    public string Name => "expense-analysis";
    public string Version => "1.0.0";
    
    private readonly BedrockRuntimeClient _bedrock;
    private readonly IPromptTemplateService _promptService;
    
    public async Task<McpToolResult> InvokeToolAsync(string toolName, object parameters)
    {
        return toolName switch
        {
            "categorize_expense" => await CategorizeExpenseAsync((AnalyzeExpenseRequest)parameters),
            "detect_fraud" => await DetectFraudAsync((FraudDetectionRequest)parameters),
            "suggest_budget" => await SuggestBudgetAsync((BudgetSuggestionRequest)parameters),
            _ => throw new ArgumentException($"Unknown tool: {toolName}")
        };
    }
    
    private async Task<McpToolResult> CategorizeExpenseAsync(AnalyzeExpenseRequest request)
    {
        var prompt = await _promptService.BuildPromptAsync("expense-categorization", new
        {
            Description = request.Description,
            Amount = request.Amount,
            Merchant = request.Merchant,
            UserHistory = request.UserSpendingPattern,
            Context = request.AdditionalContext
        });
        
        var response = await _bedrock.InvokeModelAsync(new InvokeModelRequest
        {
            ModelId = "anthropic.claude-3-sonnet-20240229-v1:0",
            Body = MemoryStream.FromJson(new
            {
                anthropic_version = "bedrock-2023-05-31",
                max_tokens = 1000,
                messages = new[]
                {
                    new { role = "user", content = prompt }
                }
            })
        });
        
        return ParseExpenseAnalysis(response);
    }
}
```

### Model Selection Strategy

#### Primary Models Configuration
```json
{
  "models": {
    "primary": {
      "text_generation": "anthropic.claude-3-sonnet-20240229-v1:0",
      "reasoning": "Claude 3 Sonnet for complex analysis and reasoning",
      "cost_per_1k_input_tokens": 0.003,
      "cost_per_1k_output_tokens": 0.015,
      "context_length": 200000,
      "use_cases": [
        "Expense categorization",
        "Budget analysis",
        "Financial advice",
        "Task prioritization"
      ]
    },
    "backup": {
      "text_generation": "amazon.titan-text-express-v1",
      "reasoning": "Cost-effective backup for simple tasks",
      "cost_per_1k_input_tokens": 0.0008,
      "cost_per_1k_output_tokens": 0.0016,
      "context_length": 8000,
      "use_cases": [
        "Simple categorization",
        "Text summarization",
        "Basic recommendations"
      ]
    },
    "embeddings": {
      "model": "amazon.titan-embed-text-v1",
      "reasoning": "Semantic search and similarity",
      "cost_per_1k_tokens": 0.0001,
      "dimension": 1536,
      "use_cases": [
        "Expense similarity search",
        "Todo recommendation",
        "Duplicate detection"
      ]
    }
  }
}
```

#### Model Selection Logic
```csharp
public class ModelSelectionService
{
    public string SelectModelForTask(AITaskType taskType, int estimatedTokens, decimal maxCost)
    {
        return taskType switch
        {
            AITaskType.ExpenseCategorization when estimatedTokens < 1000 => "amazon.titan-text-express-v1",
            AITaskType.BudgetAnalysis => "anthropic.claude-3-sonnet-20240229-v1:0",
            AITaskType.FinancialAdvice => "anthropic.claude-3-sonnet-20240229-v1:0",
            AITaskType.SimpleRecommendation when maxCost < 0.01m => "amazon.titan-text-express-v1",
            _ => "anthropic.claude-3-sonnet-20240229-v1:0"
        };
    }
}
```

### AI-Powered Features

#### 1. Intelligent Expense Categorization
```csharp
public class ExpenseCategorizationFeature
{
    private readonly IAIGatewayService _aiGateway;
    
    public async Task<ExpenseCategory> CategorizeAsync(CreateExpenseRequest request)
    {
        var analysisRequest = new AnalyzeExpenseRequest
        {
            Description = request.Description,
            Amount = request.Amount,
            Merchant = request.Merchant,
            Location = request.Location,
            UserSpendingPattern = await GetUserSpendingPatternAsync(request.UserId),
            FamilyContext = await GetFamilySpendingContextAsync(request.FamilyId)
        };
        
        var result = await _aiGateway.AnalyzeExpenseAsync(analysisRequest);
        
        return new ExpenseCategory
        {
            Primary = result.PrimaryCategory,
            Secondary = result.SecondaryCategory,
            Confidence = result.Confidence,
            Reasoning = result.Reasoning,
            SuggestedTags = result.SuggestedTags,
            IsPotentialDuplicate = result.IsPotentialDuplicate,
            DuplicateConfidence = result.DuplicateConfidence
        };
    }
}
```

**Prompt Template:**
```yaml
expense-categorization:
  system: |
    You are an expert financial categorization assistant. Analyze expenses and categorize them based on the user's spending patterns and family context.
    
    Available categories: {categories}
    User's typical spending: {user_spending_pattern}
    Family context: {family_context}
    
    Return a JSON response with:
    - primary_category: Main category
    - secondary_category: Subcategory
    - confidence: 0.0-1.0 confidence score
    - reasoning: Brief explanation
    - suggested_tags: Array of relevant tags
    - is_potential_duplicate: Boolean
    - duplicate_confidence: 0.0-1.0 if duplicate detected
    
  user: |
    Expense Details:
    Description: {description}
    Amount: ${amount}
    Merchant: {merchant}
    Location: {location}
    
    Please categorize this expense and explain your reasoning.
```

#### 2. Smart Budget Analysis and Recommendations
```csharp
public class BudgetAnalysisFeature
{
    public async Task<BudgetInsights> AnalyzeBudgetAsync(string userId, DateTime month)
    {
        var expenses = await GetMonthlyExpensesAsync(userId, month);
        var budget = await GetBudgetAsync(userId, month);
        var historicalData = await GetHistoricalSpendingAsync(userId, 6); // 6 months
        
        var analysisRequest = new BudgetAnalysisRequest
        {
            CurrentExpenses = expenses,
            BudgetLimits = budget,
            HistoricalSpending = historicalData,
            DaysRemainingInMonth = (month.AddMonths(1) - DateTime.Now).Days
        };
        
        var insights = await _aiGateway.AnalyzeBudgetAsync(analysisRequest);
        
        return new BudgetInsights
        {
            OverspendingRisk = insights.OverspendingRisk,
            Recommendations = insights.Recommendations,
            CategoryAlerts = insights.CategoryAlerts,
            SavingOpportunities = insights.SavingOpportunities,
            PredictedMonthEnd = insights.PredictedMonthEnd
        };
    }
}
```

#### 3. Intelligent Task Generation from Expenses
```csharp
public class ExpenseToTaskFeature
{
    public async Task<IEnumerable<SuggestedTask>> GenerateTasksFromExpenseAsync(Expense expense)
    {
        // Large purchases might need review tasks
        if (expense.Amount > 500)
        {
            var taskSuggestion = await _aiGateway.SuggestTaskAsync(new TaskSuggestionRequest
            {
                TriggerType = "large_expense",
                ExpenseDetails = expense,
                UserPreferences = await GetUserTaskPreferencesAsync(expense.UserId)
            });
            
            return taskSuggestion.SuggestedTasks;
        }
        
        // Recurring expenses might need automation tasks
        if (await IsRecurringExpenseAsync(expense))
        {
            return await GenerateAutomationTasksAsync(expense);
        }
        
        return Enumerable.Empty<SuggestedTask>();
    }
}
```

#### 4. Proactive Financial Insights
```csharp
public class ProactiveInsightsFeature
{
    public async Task<DailyInsights> GenerateDailyInsightsAsync(string userId)
    {
        var context = await BuildUserContextAsync(userId);
        
        var insightsRequest = new ProactiveInsightsRequest
        {
            RecentExpenses = context.RecentExpenses,
            UpcomingBills = context.UpcomingBills,
            BudgetStatus = context.BudgetStatus,
            Goals = context.FinancialGoals,
            WeatherData = context.LocalWeather, // For seasonal spending insights
            CalendarEvents = context.UpcomingEvents
        };
        
        var insights = await _aiGateway.GenerateInsightsAsync(insightsRequest);
        
        return new DailyInsights
        {
            Summary = insights.Summary,
            Alerts = insights.Alerts,
            Recommendations = insights.Recommendations,
            MotivationalMessage = insights.MotivationalMessage,
            LearningTips = insights.LearningTips
        };
    }
}
```

### Caching and Performance Strategy

#### AI Response Caching
```csharp
public class AICacheService
{
    private readonly IDistributedCache _cache;
    private readonly ILogger<AICacheService> _logger;
    
    public async Task<T> GetOrComputeAsync<T>(string cacheKey, Func<Task<T>> computeFunc, TimeSpan? expiry = null)
    {
        // Check cache first
        var cached = await _cache.GetStringAsync(cacheKey);
        if (cached != null)
        {
            _logger.LogDebug("Cache hit for key: {CacheKey}", cacheKey);
            return JsonSerializer.Deserialize<T>(cached);
        }
        
        // Compute new result
        var result = await computeFunc();
        
        // Cache the result
        var serialized = JsonSerializer.Serialize(result);
        await _cache.SetStringAsync(cacheKey, serialized, new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = expiry ?? TimeSpan.FromHours(24)
        });
        
        _logger.LogDebug("Cached result for key: {CacheKey}", cacheKey);
        return result;
    }
    
    public string GenerateCacheKey(string operation, params object[] parameters)
    {
        var hash = string.Join("|", parameters.Select(p => p?.ToString() ?? "null"));
        return $"ai:{operation}:{Hash.ComputeHash(hash)}";
    }
}
```

#### Cache Strategy by Feature
```yaml
Expense Categorization:
  Cache Key: "ai:categorize:{merchant}:{amount_range}:{description_hash}"
  TTL: 30 days
  Reason: Same merchant/amount patterns likely have same categories
  
Budget Analysis:
  Cache Key: "ai:budget:{user_id}:{month}:{expenses_hash}"
  TTL: 1 day
  Reason: Analysis changes as new expenses are added
  
Task Suggestions:
  Cache Key: "ai:tasks:{expense_type}:{amount_range}:{user_preferences_hash}"
  TTL: 7 days
  Reason: Task suggestions vary by user preferences
  
Daily Insights:
  Cache Key: "ai:insights:{user_id}:{date}"
  TTL: 12 hours
  Reason: Insights should be fresh but can be reused during the day
```

### Cost Management and Monitoring

#### Token Usage Tracking
```csharp
public class AIUsageTracker
{
    private readonly IMetricsCollector _metrics;
    
    public async Task TrackUsageAsync(string userId, string model, int inputTokens, int outputTokens, decimal cost)
    {
        // Track metrics for monitoring
        _metrics.Increment("ai.requests.total", new[] { ("model", model), ("user", userId) });
        _metrics.Histogram("ai.tokens.input", inputTokens, new[] { ("model", model) });
        _metrics.Histogram("ai.tokens.output", outputTokens, new[] { ("model", model) });
        _metrics.Histogram("ai.cost.request", cost, new[] { ("model", model) });
        
        // Store detailed usage for billing and analysis
        await _usageRepository.RecordUsageAsync(new AIUsageRecord
        {
            UserId = userId,
            Model = model,
            InputTokens = inputTokens,
            OutputTokens = outputTokens,
            Cost = cost,
            Timestamp = DateTime.UtcNow
        });
    }
}
```

#### Cost Controls
```csharp
public class AICostController
{
    public async Task<bool> CanMakeRequestAsync(string userId, string model, int estimatedTokens)
    {
        var userUsage = await GetMonthlyUsageAsync(userId);
        var estimatedCost = CalculateCost(model, estimatedTokens);
        
        // Check user limits
        if (userUsage.MonthlyCost + estimatedCost > GetUserMonthlyLimit(userId))
        {
            return false;
        }
        
        // Check overall system limits
        var systemUsage = await GetSystemUsageAsync();
        if (systemUsage.DailyCost + estimatedCost > GetDailyLimit())
        {
            return false;
        }
        
        return true;
    }
}
```

### Error Handling and Resilience

#### Retry Strategy
```csharp
public class AIRetryPolicy
{
    public static IAsyncPolicy<T> CreatePolicy<T>()
    {
        return Policy
            .Handle<BedrockException>(ex => IsRetriableError(ex))
            .WaitAndRetryAsync(
                retryCount: 3,
                sleepDurationProvider: retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
                onRetry: (outcome, timespan, retryCount, context) =>
                {
                    Console.WriteLine($"Retry {retryCount} for {context.OperationKey} in {timespan}");
                });
    }
    
    private static bool IsRetriableError(BedrockException ex)
    {
        return ex.ErrorCode switch
        {
            "ThrottlingException" => true,
            "ServiceUnavailableException" => true,
            "InternalServerException" => true,
            _ => false
        };
    }
}
```

#### Fallback Strategies
```csharp
public class AIFallbackService
{
    public async Task<ExpenseCategory> CategorizeWithFallbackAsync(AnalyzeExpenseRequest request)
    {
        try
        {
            // Try primary AI model
            return await _aiGateway.AnalyzeExpenseAsync(request);
        }
        catch (Exception ex) when (IsAIServiceError(ex))
        {
            // Fallback to rule-based categorization
            return await _ruleBasedCategorizationService.CategorizeAsync(request);
        }
    }
    
    private async Task<ExpenseCategory> RuleBasedCategorization(AnalyzeExpenseRequest request)
    {
        // Implement rule-based fallback logic
        var category = _categoryRules.Apply(request.Description, request.Merchant);
        return new ExpenseCategory
        {
            Primary = category,
            Confidence = 0.6f, // Lower confidence for rule-based
            Reasoning = "Categorized using rule-based fallback"
        };
    }
}
```

### Development and Testing

#### AI Service Testing
```csharp
public class AIGatewayServiceTests
{
    [Test]
    public async Task CategorizeExpense_ShouldReturnValidCategory()
    {
        // Arrange
        var mockBedrock = new Mock<BedrockRuntimeClient>();
        var service = new AIGatewayService(mockBedrock.Object);
        
        var request = new AnalyzeExpenseRequest
        {
            Description = "Starbucks Coffee",
            Amount = 5.99m,
            Merchant = "Starbucks"
        };
        
        // Act
        var result = await service.AnalyzeExpenseAsync(request);
        
        // Assert
        Assert.That(result.PrimaryCategory, Is.EqualTo("food_and_dining"));
        Assert.That(result.Confidence, Is.GreaterThan(0.8));
    }
    
    [Test]
    public async Task AnalyzeExpense_WithInvalidInput_ShouldFallbackGracefully()
    {
        // Test fallback mechanisms
    }
}
```

## AI Development Workflow Integration

### GitHub Copilot Optimization
```yaml
AI-Friendly Code Patterns:
  - Clear interface definitions for AI services
  - Comprehensive XML documentation
  - Consistent naming conventions
  - Well-defined request/response models
  - Standard error handling patterns
  
Copilot Context Files:
  - .github/copilot-instructions.md
  - docs/ai-patterns.md
  - src/shared/AI.Contracts/
  - examples/ai-service-usage.cs
```

### Prompt Engineering Templates
```yaml
# Store in docs/ai-prompts/
expense-categorization.yaml:
  system_prompt: |
    You are an expert financial assistant...
  examples:
    - input: "Starbucks Downtown Seattle $6.50"
      output: '{"category": "food_and_dining", "confidence": 0.95}'
  validation_rules:
    - confidence must be between 0.0 and 1.0
    - category must be from predefined list
```

## Consequences

### Positive Consequences
1. **Intelligent Automation**: Reduces manual categorization and analysis
2. **Personalized Experience**: AI learns from user patterns
3. **Proactive Insights**: Helps users make better financial decisions
4. **Scalable Architecture**: MCP servers provide reusable AI components
5. **Cost Optimization**: Smart caching and model selection reduce costs

### Negative Consequences
1. **Complexity**: AI integration adds architectural complexity
2. **Cost Management**: AI usage costs need careful monitoring
3. **Reliability**: Dependency on external AI services
4. **Data Privacy**: AI processing requires careful data handling

### Mitigation Strategies
1. **Fallback Systems**: Rule-based alternatives for AI failures
2. **Cost Controls**: Usage limits and monitoring
3. **Data Anonymization**: Remove PII before AI processing
4. **Comprehensive Testing**: Mock services for testing

## Related ADRs
- [ADR-001: Microservices Approach](001-microservices-approach.md)
- [ADR-003: AWS Cloud Provider](003-aws-cloud-provider.md)
- [ADR-005: Frontend Architecture](005-frontend-architecture.md)

---

**AI Development Notes:**
- MCP server patterns designed for easy AI-assisted development
- Clear interfaces and contracts for GitHub Copilot optimization
- Comprehensive error handling patterns for reliable AI integration
- Template-driven prompt engineering for consistent AI interactions