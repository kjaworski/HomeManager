# Todo Service Implementation Guide

> **AI Development Context** 🤖  
> This guide provides comprehensive implementation details for AI-assisted development

## Overview

The Todo Service manages task creation, assignment, and tracking for families. It provides AI-powered suggestions, recurring task management, and family coordination features.

## Architecture

### Technology Stack
```yaml
Framework: ASP.NET Core 9 with Minimal APIs
Database: Amazon DynamoDB
Caching: Redis
AI Integration: Amazon Bedrock (Claude models)
SDK: AWS SDK for .NET
Testing: xUnit with Testcontainers
Documentation: Swagger/OpenAPI 3.0
```

### DynamoDB Table Design
```json
{
  "TableName": "HomeManager-Todos",
  "KeySchema": [
    {
      "AttributeName": "PK",
      "KeyType": "HASH"
    },
    {
      "AttributeName": "SK",
      "KeyType": "RANGE"
    }
  ],
  "AttributeDefinitions": [
    {
      "AttributeName": "PK",
      "AttributeType": "S"
    },
    {
      "AttributeName": "SK",
      "AttributeType": "S"
    },
    {
      "AttributeName": "GSI1PK",
      "AttributeType": "S"
    },
    {
      "AttributeName": "GSI1SK",
      "AttributeType": "S"
    }
  ],
  "GlobalSecondaryIndexes": [
    {
      "IndexName": "GSI1",
      "KeySchema": [
        {
          "AttributeName": "GSI1PK",
          "KeyType": "HASH"
        },
        {
          "AttributeName": "GSI1SK",
          "KeyType": "RANGE"
        }
      ],
      "Projection": {
        "ProjectionType": "ALL"
      }
    }
  ]
}
```

### Access Patterns
```yaml
Primary Access Patterns:
  - Get todo by ID: PK=TODO#{todoId}, SK=TODO
  - Get todos for family: PK=FAMILY#{familyId}, SK begins_with TODO#
  - Get todos by assignee: GSI1PK=USER#{userId}, GSI1SK begins_with TODO#
  - Get todos by status: GSI1PK=FAMILY#{familyId}#STATUS#{status}, GSI1SK=TODO#{timestamp}
  - Get recurring todos: PK=FAMILY#{familyId}, SK begins_with RECURRING#

Secondary Patterns:
  - Get todos by category: GSI1PK=FAMILY#{familyId}#CATEGORY#{category}
  - Get overdue todos: GSI1PK=OVERDUE, GSI1SK=TODO#{dueDate}
  - Get todo history: PK=TODO#{todoId}, SK begins_with HISTORY#
```

## Implementation

### 1. Entity Models
```csharp
// Todo.cs
namespace HomeManager.Todos.Domain.Entities;

public class Todo
{
    public string Id { get; set; } = null!;
    public string FamilyId { get; set; } = null!;
    public string Title { get; set; } = null!;
    public string? Description { get; set; }
    public TodoStatus Status { get; set; } = TodoStatus.Pending;
    public TodoPriority Priority { get; set; } = TodoPriority.Medium;
    public string? Category { get; set; }
    public string? AssignedTo { get; set; }
    public string? AssignedBy { get; set; }
    public DateTime? DueDate { get; set; }
    public int? EstimatedMinutes { get; set; }
    public int? ActualMinutes { get; set; }
    public RecurringConfig? Recurring { get; set; }
    public List<string> Tags { get; set; } = new();
    public List<TodoAttachment> Attachments { get; set; } = new();
    public DateTime? CompletedAt { get; set; }
    public string? CompletedBy { get; set; }
    public string? CompletionNotes { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
    public string CreatedBy { get; set; } = null!;
    
    // DynamoDB properties
    public string PK => $"TODO#{Id}";
    public string SK => "TODO";
    public string GSI1PK => !string.IsNullOrEmpty(AssignedTo) ? $"USER#{AssignedTo}" : $"FAMILY#{FamilyId}";
    public string GSI1SK => $"TODO#{CreatedAt:yyyy-MM-ddTHH:mm:ss.fffZ}";
    public string EntityType => "Todo";
}

public enum TodoStatus
{
    Pending,
    InProgress,
    Completed,
    Archived
}

public enum TodoPriority
{
    Low,
    Medium,
    High,
    Urgent
}

public class RecurringConfig
{
    public bool Enabled { get; set; }
    public RecurringPattern Pattern { get; set; }
    public int Interval { get; set; } = 1;
    public List<int> DaysOfWeek { get; set; } = new();
    public int? DayOfMonth { get; set; }
    public DateTime? EndDate { get; set; }
    public DateTime? LastGenerated { get; set; }
}

public enum RecurringPattern
{
    Daily,
    Weekly,
    Monthly,
    Yearly
}

public class TodoAttachment
{
    public string Id { get; set; } = null!;
    public string FileName { get; set; } = null!;
    public long FileSize { get; set; }
    public string MimeType { get; set; } = null!;
    public string Url { get; set; } = null!;
    public string UploadedBy { get; set; } = null!;
    public DateTime UploadedAt { get; set; }
}
```

### 2. Repository Implementation
```csharp
// ITodoRepository.cs
namespace HomeManager.Todos.Domain.Repositories;

public interface ITodoRepository
{
    Task<Todo?> GetByIdAsync(string todoId, CancellationToken cancellationToken = default);
    Task<PagedResult<Todo>> GetByFamilyAsync(string familyId, TodoQuery query, CancellationToken cancellationToken = default);
    Task<List<Todo>> GetByAssigneeAsync(string userId, TodoFilter filter, CancellationToken cancellationToken = default);
    Task<Todo> CreateAsync(Todo todo, CancellationToken cancellationToken = default);
    Task<Todo> UpdateAsync(Todo todo, CancellationToken cancellationToken = default);
    Task DeleteAsync(string todoId, CancellationToken cancellationToken = default);
    Task<List<Todo>> GetRecurringTodosAsync(string familyId, CancellationToken cancellationToken = default);
    Task<List<Todo>> GetOverdueTodosAsync(DateTime cutoffDate, CancellationToken cancellationToken = default);
}

// DynamoDbTodoRepository.cs
namespace HomeManager.Todos.Infrastructure.Repositories;

public class DynamoDbTodoRepository : ITodoRepository
{
    private readonly IAmazonDynamoDB _dynamoDb;
    private readonly ILogger<DynamoDbTodoRepository> _logger;
    private readonly string _tableName;
    
    public DynamoDbTodoRepository(
        IAmazonDynamoDB dynamoDb,
        IConfiguration configuration,
        ILogger<DynamoDbTodoRepository> logger)
    {
        _dynamoDb = dynamoDb;
        _logger = logger;
        _tableName = configuration["DynamoDB:TodosTable"] ?? "HomeManager-Todos";
    }
    
    public async Task<Todo?> GetByIdAsync(string todoId, CancellationToken cancellationToken = default)
    {
        _logger.LogInformation("Getting todo by ID: {TodoId}", todoId);
        
        var request = new GetItemRequest
        {
            TableName = _tableName,
            Key = new Dictionary<string, AttributeValue>
            {
                ["PK"] = new AttributeValue($"TODO#{todoId}"),
                ["SK"] = new AttributeValue("TODO")
            }
        };
        
        var response = await _dynamoDb.GetItemAsync(request, cancellationToken);
        
        if (!response.IsItemSet)
        {
            return null;
        }
        
        return MapFromDynamoDb(response.Item);
    }
    
    public async Task<PagedResult<Todo>> GetByFamilyAsync(
        string familyId,
        TodoQuery query,
        CancellationToken cancellationToken = default)
    {
        _logger.LogInformation("Getting todos for family: {FamilyId}", familyId);
        
        var request = new QueryRequest
        {
            TableName = _tableName,
            KeyConditionExpression = "PK = :pk AND begins_with(SK, :sk)",
            ExpressionAttributeValues = new Dictionary<string, AttributeValue>
            {
                [":pk"] = new AttributeValue($"FAMILY#{familyId}"),
                [":sk"] = new AttributeValue("TODO#")
            },
            ScanIndexForward = false, // Most recent first
            Limit = query.Limit
        };
        
        // Add filters
        if (query.Status.HasValue)
        {
            request.FilterExpression = "#status = :status";
            request.ExpressionAttributeNames = new Dictionary<string, string>
            {
                ["#status"] = "Status"
            };
            request.ExpressionAttributeValues.Add(":status", new AttributeValue(query.Status.ToString()));
        }
        
        if (!string.IsNullOrEmpty(query.LastEvaluatedKey))
        {
            request.ExclusiveStartKey = DeserializeLastEvaluatedKey(query.LastEvaluatedKey);
        }
        
        var response = await _dynamoDb.QueryAsync(request, cancellationToken);
        
        var todos = response.Items.Select(MapFromDynamoDb).ToList();
        
        return new PagedResult<Todo>
        {
            Items = todos,
            LastEvaluatedKey = response.LastEvaluatedKey?.Any() == true 
                ? SerializeLastEvaluatedKey(response.LastEvaluatedKey) 
                : null,
            HasMoreResults = response.LastEvaluatedKey?.Any() == true
        };
    }
    
    public async Task<Todo> CreateAsync(Todo todo, CancellationToken cancellationToken = default)
    {
        _logger.LogInformation("Creating new todo: {TodoId}", todo.Id);
        
        todo.CreatedAt = DateTime.UtcNow;
        todo.UpdatedAt = DateTime.UtcNow;
        
        var item = MapToDynamoDb(todo);
        
        // Add family index entry
        var familyIndexItem = new Dictionary<string, AttributeValue>(item)
        {
            ["PK"] = new AttributeValue($"FAMILY#{todo.FamilyId}"),
            ["SK"] = new AttributeValue($"TODO#{todo.Id}")
        };
        
        var transactRequest = new TransactWriteItemsRequest
        {
            TransactItems = new List<TransactWriteItem>
            {
                new TransactWriteItem
                {
                    Put = new Put
                    {
                        TableName = _tableName,
                        Item = item,
                        ConditionExpression = "attribute_not_exists(PK)"
                    }
                },
                new TransactWriteItem
                {
                    Put = new Put
                    {
                        TableName = _tableName,
                        Item = familyIndexItem
                    }
                }
            }
        };
        
        await _dynamoDb.TransactWriteItemsAsync(transactRequest, cancellationToken);
        
        return todo;
    }
    
    // Additional methods...
    
    private Todo MapFromDynamoDb(Dictionary<string, AttributeValue> item)
    {
        return new Todo
        {
            Id = item["Id"].S,
            FamilyId = item["FamilyId"].S,
            Title = item["Title"].S,
            Description = item.TryGetValue("Description", out var desc) ? desc.S : null,
            Status = Enum.Parse<TodoStatus>(item["Status"].S),
            Priority = Enum.Parse<TodoPriority>(item["Priority"].S),
            Category = item.TryGetValue("Category", out var cat) ? cat.S : null,
            AssignedTo = item.TryGetValue("AssignedTo", out var assignedTo) ? assignedTo.S : null,
            AssignedBy = item.TryGetValue("AssignedBy", out var assignedBy) ? assignedBy.S : null,
            DueDate = item.TryGetValue("DueDate", out var dueDate) ? DateTime.Parse(dueDate.S) : null,
            EstimatedMinutes = item.TryGetValue("EstimatedMinutes", out var est) ? int.Parse(est.N) : null,
            ActualMinutes = item.TryGetValue("ActualMinutes", out var actual) ? int.Parse(actual.N) : null,
            Tags = item.TryGetValue("Tags", out var tags) ? tags.SS : new List<string>(),
            CompletedAt = item.TryGetValue("CompletedAt", out var completed) ? DateTime.Parse(completed.S) : null,
            CompletedBy = item.TryGetValue("CompletedBy", out var completedBy) ? completedBy.S : null,
            CompletionNotes = item.TryGetValue("CompletionNotes", out var notes) ? notes.S : null,
            CreatedAt = DateTime.Parse(item["CreatedAt"].S),
            UpdatedAt = DateTime.Parse(item["UpdatedAt"].S),
            CreatedBy = item["CreatedBy"].S,
            Recurring = item.TryGetValue("Recurring", out var recurring) ? 
                JsonSerializer.Deserialize<RecurringConfig>(recurring.S) : null,
            Attachments = item.TryGetValue("Attachments", out var attachments) ? 
                JsonSerializer.Deserialize<List<TodoAttachment>>(attachments.S) ?? new List<TodoAttachment>() : 
                new List<TodoAttachment>()
        };
    }
    
    private Dictionary<string, AttributeValue> MapToDynamoDb(Todo todo)
    {
        var item = new Dictionary<string, AttributeValue>
        {
            ["PK"] = new AttributeValue(todo.PK),
            ["SK"] = new AttributeValue(todo.SK),
            ["GSI1PK"] = new AttributeValue(todo.GSI1PK),
            ["GSI1SK"] = new AttributeValue(todo.GSI1SK),
            ["EntityType"] = new AttributeValue(todo.EntityType),
            ["Id"] = new AttributeValue(todo.Id),
            ["FamilyId"] = new AttributeValue(todo.FamilyId),
            ["Title"] = new AttributeValue(todo.Title),
            ["Status"] = new AttributeValue(todo.Status.ToString()),
            ["Priority"] = new AttributeValue(todo.Priority.ToString()),
            ["CreatedAt"] = new AttributeValue(todo.CreatedAt.ToString("O")),
            ["UpdatedAt"] = new AttributeValue(todo.UpdatedAt.ToString("O")),
            ["CreatedBy"] = new AttributeValue(todo.CreatedBy)
        };
        
        // Add optional fields
        if (!string.IsNullOrEmpty(todo.Description))
            item["Description"] = new AttributeValue(todo.Description);
            
        if (!string.IsNullOrEmpty(todo.Category))
            item["Category"] = new AttributeValue(todo.Category);
            
        if (!string.IsNullOrEmpty(todo.AssignedTo))
            item["AssignedTo"] = new AttributeValue(todo.AssignedTo);
            
        if (todo.DueDate.HasValue)
            item["DueDate"] = new AttributeValue(todo.DueDate.Value.ToString("O"));
            
        if (todo.EstimatedMinutes.HasValue)
            item["EstimatedMinutes"] = new AttributeValue { N = todo.EstimatedMinutes.Value.ToString() };
            
        if (todo.Tags.Any())
            item["Tags"] = new AttributeValue { SS = todo.Tags };
            
        if (todo.Recurring != null)
            item["Recurring"] = new AttributeValue(JsonSerializer.Serialize(todo.Recurring));
            
        if (todo.Attachments.Any())
            item["Attachments"] = new AttributeValue(JsonSerializer.Serialize(todo.Attachments));
            
        return item;
    }
}
```

### 3. Service Implementation
```csharp
// ITodoService.cs
namespace HomeManager.Todos.Application.Services;

public interface ITodoService
{
    Task<Todo> CreateTodoAsync(CreateTodoRequest request, string userId, CancellationToken cancellationToken = default);
    Task<Todo> UpdateTodoAsync(string todoId, UpdateTodoRequest request, string userId, CancellationToken cancellationToken = default);
    Task<Todo> CompleteTodoAsync(string todoId, CompleteTodoRequest request, string userId, CancellationToken cancellationToken = default);
    Task<Todo> AssignTodoAsync(string todoId, AssignTodoRequest request, string userId, CancellationToken cancellationToken = default);
    Task<PagedResult<Todo>> GetFamilyTodosAsync(string familyId, TodoQuery query, CancellationToken cancellationToken = default);
    Task<List<TodoSuggestion>> GetAISuggestionsAsync(string familyId, string context, CancellationToken cancellationToken = default);
    Task ProcessRecurringTodosAsync(CancellationToken cancellationToken = default);
}

// TodoService.cs
public class TodoService : ITodoService
{
    private readonly ITodoRepository _todoRepository;
    private readonly IAIGatewayService _aiGateway;
    private readonly ILogger<TodoService> _logger;
    
    public TodoService(
        ITodoRepository todoRepository,
        IAIGatewayService aiGateway,
        ILogger<TodoService> logger)
    {
        _todoRepository = todoRepository;
        _aiGateway = aiGateway;
        _logger = logger;
    }
    
    public async Task<Todo> CreateTodoAsync(
        CreateTodoRequest request,
        string userId,
        CancellationToken cancellationToken = default)
    {
        _logger.LogInformation("Creating todo for family {FamilyId} by user {UserId}", request.FamilyId, userId);
        
        var todo = new Todo
        {
            Id = Guid.NewGuid().ToString(),
            FamilyId = request.FamilyId,
            Title = request.Title,
            Description = request.Description,
            Priority = request.Priority,
            Category = request.Category,
            AssignedTo = request.AssignedTo,
            AssignedBy = !string.IsNullOrEmpty(request.AssignedTo) ? userId : null,
            DueDate = request.DueDate,
            EstimatedMinutes = request.EstimatedMinutes,
            Tags = request.Tags ?? new List<string>(),
            Recurring = request.Recurring,
            CreatedBy = userId
        };
        
        // AI-powered category suggestion if not provided
        if (string.IsNullOrEmpty(todo.Category))
        {
            try
            {
                var categoryRequest = new CategorizeTodoRequest
                {
                    Title = todo.Title,
                    Description = todo.Description,
                    FamilyId = todo.FamilyId
                };
                
                var suggestions = await _aiGateway.CategorizeTodoAsync(categoryRequest, cancellationToken);
                todo.Category = suggestions.SuggestedCategory;
                
                _logger.LogInformation("AI suggested category '{Category}' for todo '{Title}'", 
                    todo.Category, todo.Title);
            }
            catch (Exception ex)
            {
                _logger.LogWarning(ex, "Failed to get AI category suggestion for todo '{Title}'", todo.Title);
            }
        }
        
        var createdTodo = await _todoRepository.CreateAsync(todo, cancellationToken);
        
        _logger.LogInformation("Successfully created todo {TodoId}", createdTodo.Id);
        
        return createdTodo;
    }
    
    public async Task<List<TodoSuggestion>> GetAISuggestionsAsync(
        string familyId,
        string context,
        CancellationToken cancellationToken = default)
    {
        _logger.LogInformation("Getting AI suggestions for family {FamilyId} with context '{Context}'", 
            familyId, context);
        
        try
        {
            // Get recent family todos for context
            var recentTodos = await _todoRepository.GetByFamilyAsync(
                familyId, 
                new TodoQuery { Limit = 50, SortBy = "CreatedAt", SortOrder = "desc" },
                cancellationToken);
            
            var request = new TodoSuggestionRequest
            {
                FamilyId = familyId,
                Context = context,
                RecentTodos = recentTodos.Items.Take(20).ToList(),
                Timeframe = "week"
            };
            
            var response = await _aiGateway.GenerateTodoSuggestionsAsync(request, cancellationToken);
            
            _logger.LogInformation("Generated {Count} AI suggestions for family {FamilyId}", 
                response.Suggestions.Count, familyId);
            
            return response.Suggestions;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to generate AI suggestions for family {FamilyId}", familyId);
            return new List<TodoSuggestion>();
        }
    }
    
    // Additional methods...
}
```

### 4. AI Integration
```csharp
// AI Gateway Service Integration
namespace HomeManager.Todos.Application.Services;

public class TodoAIService : ITodoAIService
{
    private readonly IAmazonBedrockRuntime _bedrock;
    private readonly ILogger<TodoAIService> _logger;
    
    public async Task<List<TodoSuggestion>> GenerateSmartSuggestionsAsync(
        string familyId,
        List<Todo> recentTodos,
        string context,
        CancellationToken cancellationToken = default)
    {
        var prompt = BuildSuggestionPrompt(familyId, recentTodos, context);
        
        var request = new InvokeModelRequest
        {
            ModelId = "anthropic.claude-3-sonnet-20240229-v1:0",
            Body = JsonSerializer.Serialize(new
            {
                anthropic_version = "bedrock-2023-05-31",
                max_tokens = 2000,
                temperature = 0.3,
                messages = new[]
                {
                    new
                    {
                        role = "user",
                        content = prompt
                    }
                }
            })
        };
        
        var response = await _bedrock.InvokeModelAsync(request, cancellationToken);
        var responseBody = JsonSerializer.Deserialize<ClaudeResponse>(response.Body);
        
        return ParseSuggestions(responseBody.Content.First().Text);
    }
    
    private string BuildSuggestionPrompt(string familyId, List<Todo> recentTodos, string context)
    {
        return $@"
You are an AI assistant for HomeManager, a family home management system.
Your task is to generate intelligent todo suggestions based on family patterns and context.

Family ID: {familyId}
Context: {context}

Recent Family Todos (last 20):
{string.Join("\n", recentTodos.Select(t => $"- {t.Title} (Category: {t.Category}, Status: {t.Status}, Created: {t.CreatedAt:yyyy-MM-dd})"))}

Based on the recent todo patterns, please suggest 3-5 new todos that would be helpful for this family.
Consider:
1. Recurring patterns in their existing todos
2. Seasonal or time-based needs
3. Categories they frequently use
4. Incomplete todos that might need follow-up
5. Common household tasks they might be missing

Return your response as a JSON array with this structure:
[
  {{
    "title": "Task title",
    "description": "Detailed description",
    "category": "suggested category",
    "priority": "Low|Medium|High|Urgent",
    "estimatedMinutes": 30,
    "reasoning": "Why this task is suggested",
    "confidence": 0.85
  }}
]

Ensure suggestions are:
- Practical and actionable
- Appropriate for the family's patterns
- Varied in priority and time requirements
- Include confidence scores between 0.0 and 1.0
";
    }
}
```

## Testing

### Unit Tests
```csharp
public class TodoServiceTests
{
    private readonly Mock<ITodoRepository> _repositoryMock;
    private readonly Mock<IAIGatewayService> _aiGatewayMock;
    private readonly Mock<ILogger<TodoService>> _loggerMock;
    private readonly TodoService _todoService;
    
    [Fact]
    public async Task CreateTodoAsync_WithValidRequest_ShouldCreateTodo()
    {
        // Arrange
        var request = new CreateTodoRequest
        {
            FamilyId = "family-123",
            Title = "Take out trash",
            Description = "Weekly trash pickup",
            Priority = TodoPriority.Medium
        };
        
        var userId = "user-456";
        
        _repositoryMock.Setup(x => x.CreateAsync(It.IsAny<Todo>(), It.IsAny<CancellationToken>()))
                      .ReturnsAsync((Todo t, CancellationToken ct) => t);
        
        // Act
        var result = await _todoService.CreateTodoAsync(request, userId);
        
        // Assert
        Assert.NotNull(result);
        Assert.Equal(request.Title, result.Title);
        Assert.Equal(request.FamilyId, result.FamilyId);
        Assert.Equal(userId, result.CreatedBy);
        
        _repositoryMock.Verify(x => x.CreateAsync(It.IsAny<Todo>(), It.IsAny<CancellationToken>()), Times.Once);
    }
}
```

### Integration Tests with Testcontainers
```csharp
public class TodoServiceIntegrationTests : IAsyncLifetime
{
    private readonly DynamoDbContainer _dynamoDbContainer;
    private IAmazonDynamoDB _dynamoDbClient;
    private TodoService _todoService;
    
    public TodoServiceIntegrationTests()
    {
        _dynamoDbContainer = new DynamoDbBuilder()
            .WithImage("amazon/dynamodb-local:latest")
            .Build();
    }
    
    public async Task InitializeAsync()
    {
        await _dynamoDbContainer.StartAsync();
        
        _dynamoDbClient = new AmazonDynamoDBClient(new AmazonDynamoDBConfig
        {
            ServiceURL = _dynamoDbContainer.GetConnectionString()
        });
        
        await CreateTestTable();
        
        var repository = new DynamoDbTodoRepository(_dynamoDbClient, /* configuration */, /* logger */);
        _todoService = new TodoService(repository, /* ai gateway */, /* logger */);
    }
    
    [Fact]
    public async Task TodoWorkflow_EndToEnd_ShouldWork()
    {
        // Test complete workflow: create, update, complete
        var createRequest = new CreateTodoRequest
        {
            FamilyId = "test-family",
            Title = "Integration test todo",
            Priority = TodoPriority.High
        };
        
        var created = await _todoService.CreateTodoAsync(createRequest, "test-user");
        Assert.NotNull(created);
        
        var completed = await _todoService.CompleteTodoAsync(
            created.Id, 
            new CompleteTodoRequest { Notes = "Done!" }, 
            "test-user");
            
        Assert.Equal(TodoStatus.Completed, completed.Status);
        Assert.Equal("test-user", completed.CompletedBy);
    }
    
    public async Task DisposeAsync()
    {
        await _dynamoDbContainer.StopAsync();
    }
}
```