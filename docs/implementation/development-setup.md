# Development Environment Setup - HomeManager

> **AI-Assisted Development Ready** 🤖  
> This guide is optimized for GitHub Copilot, Claude Sonnet, and GPT-assisted development

## Prerequisites

### Required Software
```yaml
Core Development Tools:
  - Visual Studio Code (latest)
  - .NET 10 SDK
  - Node.js 18+ (for frontend and tooling)
  - Docker Desktop
  - Git
  - AWS CLI v2
  
Database Tools:
  - PostgreSQL 15+ (local development)
  - DynamoDB Local
  - MongoDB Community Edition
  
AI Development Tools:
  - GitHub Copilot (VS Code extension)
  - Claude for VS Code (optional)
  - AWS Toolkit for VS Code
```

### System Requirements
```yaml
Minimum Requirements:
  - RAM: 16GB (32GB recommended for AI tools)
  - Storage: 50GB free space
  - CPU: 4+ cores (8+ cores recommended)
  - OS: Windows 10/11, macOS 10.15+, or Ubuntu 20.04+
  
Recommended for AI Development:
  - RAM: 32GB+ for better AI tool performance
  - SSD: Fast storage for Docker and databases
  - CPU: Modern multi-core processor for compilation
```

## Step 1: Clone and Setup Repository

### Repository Setup
```bash
# Clone the repository
git clone https://github.com/kjaworski/HomeManager.git
cd HomeManager

# Install Node.js dependencies for tooling
npm install

# Restore .NET dependencies
dotnet restore

# Build the solution
dotnet build
```

### Environment Configuration
```bash
# Copy environment template
cp .env.template .env.local

# Edit .env.local with your configuration
# See Environment Variables section below
```

## Step 2: VS Code Configuration

### Required Extensions
```json
{
  "recommendations": [
    "ms-dotnettools.csharp",
    "ms-dotnettools.csdevkit",
    "github.copilot",
    "github.copilot-chat",
    "ms-vscode.vscode-typescript-next",
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-json",
    "redhat.vscode-yaml",
    "ms-vscode-remote.remote-containers",
    "amazonwebservices.aws-toolkit-vscode"
  ]
}
```

### VS Code Settings
```json
{
  "dotnet.defaultSolution": "HomeManager.sln",
  "github.copilot.enable": {
    "*": true,
    "yaml": true,
    "plaintext": false,
    "markdown": true
  },
  "github.copilot.editor.enableAutoCompletions": true,
  "csharp.semanticHighlighting.enabled": true,
  "omnisharp.enableEditorConfigSupport": true,
  "files.exclude": {
    "**/bin": true,
    "**/obj": true,
    "**/node_modules": true
  }
}
```

## Step 3: Database Setup

### PostgreSQL (Identity Service)
```bash
# Using Docker
docker run --name homemanager-postgres \
  -e POSTGRES_DB=homemanager_identity \
  -e POSTGRES_USER=homemanager \
  -e POSTGRES_PASSWORD=development123 \
  -p 5432:5432 \
  -d postgres:15

# Verify connection
psql -h localhost -U homemanager -d homemanager_identity
```

### DynamoDB Local (Todo Service)
```bash
# Using Docker
docker run --name homemanager-dynamodb \
  -p 8000:8000 \
  -d amazon/dynamodb-local

# Create tables
aws dynamodb create-table \
  --endpoint-url http://localhost:8000 \
  --table-name Todos \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

### MongoDB (Budget Service)
```bash
# Using Docker
docker run --name homemanager-mongodb \
  -e MONGO_INITDB_ROOT_USERNAME=homemanager \
  -e MONGO_INITDB_ROOT_PASSWORD=development123 \
  -e MONGO_INITDB_DATABASE=homemanager_budget \
  -p 27017:27017 \
  -d mongo:6

# Verify connection
mongosh "mongodb://homemanager:development123@localhost:27017/homemanager_budget"
```

## Step 4: Environment Variables

### .env.local Configuration
```bash
# Application Settings
ASPNETCORE_ENVIRONMENT=Development
ASPNETCORE_URLS=https://localhost:7001;http://localhost:5001

# Database Connections
ConnectionStrings__Identity="Host=localhost;Database=homemanager_identity;Username=homemanager;Password=development123"
ConnectionStrings__DynamoDB="http://localhost:8000"
ConnectionStrings__MongoDB="mongodb://homemanager:development123@localhost:27017/homemanager_budget"

# JWT Configuration
Jwt__Secret="your-super-secret-jwt-key-for-development-only-change-in-production"
Jwt__Issuer="HomeManager.Development"
Jwt__Audience="HomeManager.Api"
Jwt__ExpiryMinutes=60

# AWS Configuration (for local development)
AWS_PROFILE=homemanager-dev
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=test
AWS_SECRET_ACCESS_KEY=test

# AI Configuration
Bedrock__Region=us-east-1
Bedrock__DefaultModel=anthropic.claude-3-sonnet-20240229-v1:0
Bedrock__MaxTokens=4096

# Feature Flags
Features__AIEnabled=true
Features__NotificationsEnabled=false
Features__FileStorageEnabled=true
```

## Step 5: Development Workflow

### Running Services Locally
```bash
# Start all databases
docker-compose -f docker-compose.dev.yml up -d

# Run Identity Service
cd src/HomeManager.Identity
dotnet run

# Run Todo Service (in another terminal)
cd src/HomeManager.Todos
dotnet run

# Run Budget Service (in another terminal)
cd src/HomeManager.Budget
dotnet run

# Run API Gateway (in another terminal)
cd src/HomeManager.Api
dotnet run
```

### Using VS Code Tasks
```json
// .vscode/tasks.json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Start All Services",
      "type": "shell",
      "command": "docker-compose",
      "args": ["-f", "docker-compose.dev.yml", "up", "-d"],
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "Run API",
      "type": "shell",
      "command": "dotnet",
      "args": ["run", "--project", "src/HomeManager.Api"],
      "group": "build",
      "isBackground": true
    }
  ]
}
```

## Step 6: AI Development Setup

### GitHub Copilot Configuration
```bash
# Verify Copilot is working
# In VS Code, open a C# file and start typing:
# "public class User" - Copilot should suggest completions

# Enable Copilot Chat
# Ctrl+Shift+P > "GitHub Copilot: Open Chat"
```

### AI Context Files
```markdown
<!-- .copilotcontext -->
# HomeManager Development Context

## Project Overview
HomeManager is a microservices-based family management system built with:
- Backend: .NET 10, ASP.NET Core
- Frontend: React, Next.js
- Databases: PostgreSQL, DynamoDB, MongoDB
- Cloud: AWS (EKS, Bedrock, S3)
- AI: Amazon Bedrock with Claude models

## Coding Standards
- C# 12 with nullable reference types
- Minimal APIs with OpenAPI documentation
- xUnit for testing with Testcontainers
- Clean Architecture with CQRS patterns
```

### AI Prompt Templates
```yaml
# Common AI Development Prompts
Service Creation: |
  Create a new microservice for HomeManager called {ServiceName} that handles {Description}.
  
  Requirements:
  - .NET 10 with minimal APIs
  - Clean Architecture with CQRS
  - Database: {DatabaseType}
  - Include comprehensive error handling
  - Add OpenAPI documentation
  - Follow HomeManager coding standards

API Endpoint: |
  Create a REST API endpoint for {Entity} management in HomeManager.
  
  Requirements:
  - CRUD operations with proper HTTP methods
  - Request/Response DTOs with validation
  - Error handling with standard responses
  - OpenAPI documentation with examples
  - Family-scoped access control

Test Generation: |
  Generate comprehensive tests for the {Component} in HomeManager.
  
  Requirements:
  - Unit tests with xUnit
  - Integration tests with Testcontainers
  - Cover happy path and edge cases
  - Mock external dependencies
  - Follow AAA (Arrange, Act, Assert) pattern
```

## Step 7: Testing Setup

### Unit Testing
```bash
# Run all tests
dotnet test

# Run tests with coverage
dotnet test --collect:"XPlat Code Coverage"

# Run specific test project
dotnet test tests/HomeManager.Identity.Tests/
```

### Integration Testing
```bash
# Start test databases
docker-compose -f docker-compose.test.yml up -d

# Run integration tests
dotnet test --filter Category=Integration

# Cleanup test environment
docker-compose -f docker-compose.test.yml down -v
```

## Step 8: Frontend Development

### React Web Application
```bash
# Navigate to frontend directory
cd src/HomeManager.Web

# Install dependencies
npm install

# Start development server
npm run dev

# Open browser to http://localhost:3000
```

### Frontend Configuration
```javascript
// next.config.js
module.exports = {
  env: {
    API_BASE_URL: process.env.API_BASE_URL || 'http://localhost:5001',
    AI_FEATURES_ENABLED: process.env.AI_FEATURES_ENABLED || 'true'
  },
  experimental: {
    serverComponentsExternalPackages: ['@homemanager/shared']
  }
};
```

## Troubleshooting

### Common Issues
```yaml
Issue: "dotnet command not found"
Solution: |
  - Install .NET 10 SDK from https://dotnet.microsoft.com/download
  - Restart terminal/VS Code after installation
  - Verify with: dotnet --version

Issue: "Database connection failed"
Solution: |
  - Verify Docker containers are running: docker ps
  - Check connection strings in .env.local
  - Ensure databases are properly initialized
  - Test connections with database clients

Issue: "GitHub Copilot not working"
Solution: |
  - Verify Copilot subscription is active
  - Check VS Code extension is enabled
  - Restart VS Code
  - Sign out and sign back into GitHub account

Issue: "Build errors after git pull"
Solution: |
  - Clean solution: dotnet clean
  - Restore packages: dotnet restore
  - Rebuild: dotnet build
  - Update VS Code extensions if needed
```

### Performance Optimization
```yaml
Slow Build Times:
  - Enable parallel builds in VS Code
  - Exclude bin/obj folders from file watcher
  - Use SSD storage for better I/O performance
  - Increase Docker memory allocation

High Memory Usage:
  - Close unused VS Code windows
  - Disable unnecessary extensions
  - Increase swap file size
  - Monitor Docker container memory usage
```

## Development Resources

### Documentation Links
- [.NET 10 Documentation](https://docs.microsoft.com/en-us/dotnet/)
- [ASP.NET Core Guide](https://docs.microsoft.com/en-us/aspnet/core/)
- [GitHub Copilot Documentation](https://docs.github.com/en/copilot)
- [AWS SDK for .NET](https://docs.aws.amazon.com/sdk-for-net/)
- [Amazon Bedrock Developer Guide](https://docs.aws.amazon.com/bedrock/)

### AI Development Resources
- [Effective GitHub Copilot Usage](https://github.blog/2022-09-14-8-things-you-didnt-know-you-could-do-with-github-copilot/)
- [Claude API Documentation](https://docs.anthropic.com/claude/reference/)
- [Prompt Engineering Guide](https://www.promptingguide.ai/)

### HomeManager Specific
- [Architecture Overview](../architecture/system-overview.md)
- [API Documentation](../api/README.md)
- [Deployment Guide](deployment-guide.md)
- [Contributing Guidelines](../../CONTRIBUTING.md)