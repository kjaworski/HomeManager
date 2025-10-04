# AI Gateway Implementation Guide

> **AI Development Context** 🤖  
> Central AI service coordinator with Amazon Bedrock integration

## Overview

The AI Gateway orchestrates all AI functionality in HomeManager, integrating with Amazon Bedrock, MCP servers, and local AI models to provide intelligent features across all services.

## Technology Stack

```yaml
Framework: ASP.NET Core 9 with Minimal APIs
AI Platform: Amazon Bedrock (Claude 3 Sonnet/Haiku)
MCP Integration: Model Context Protocol servers
Caching: Redis for response caching
Monitoring: CloudWatch and X-Ray
Security: IAM roles and VPC endpoints
```

## Implementation Highlights

### Core Services
```csharp
public interface IAIGatewayService
{
    Task<TodoSuggestionResponse> GenerateTodoSuggestionsAsync(
        TodoSuggestionRequest request, 
        CancellationToken cancellationToken = default);
        
    Task<ReceiptAnalysisResponse> AnalyzeReceiptAsync(
        ReceiptAnalysisRequest request, 
        CancellationToken cancellationToken = default);
        
    Task<BudgetInsightsResponse> GenerateBudgetInsightsAsync(
        BudgetInsightsRequest request, 
        CancellationToken cancellationToken = default);
        
    Task<ChatResponse> ProcessChatAsync(
        ChatRequest request, 
        CancellationToken cancellationToken = default);
}
```

### Bedrock Integration
```csharp
public class BedrockService
{
    private readonly IAmazonBedrockRuntime _bedrock;
    
    public async Task<string> InvokeClaudeAsync(
        string prompt, 
        string modelId = "anthropic.claude-3-sonnet-20240229-v1:0")
    {
        var request = new InvokeModelRequest
        {
            ModelId = modelId,
            Body = JsonSerializer.Serialize(new
            {
                anthropic_version = "bedrock-2023-05-31",
                max_tokens = 4000,
                messages = new[] { new { role = "user", content = prompt } }
            })
        };
        
        var response = await _bedrock.InvokeModelAsync(request);
        // Process response...
    }
}
```

### Key Features
- Multi-model AI routing (Sonnet for complex, Haiku for fast)
- Response caching and optimization
- Request batching and rate limiting
- Error handling and fallback strategies
- Cost monitoring and budget controls
- Security and compliance enforcement