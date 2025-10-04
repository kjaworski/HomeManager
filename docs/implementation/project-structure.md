# Project Structure Guide - HomeManager

> **AI Development Context** 🤖  
> Comprehensive guide to HomeManager's monorepo structure and organization

## Repository Structure

```
HomeManager/
├── .github/                    # GitHub workflows and templates
│   ├── workflows/
│   │   └── ci-cd.yml          # CI/CD pipeline
│   ├── ISSUE_TEMPLATE/        # Issue templates
│   └── pull_request_template.md
├── docs/                      # Documentation
│   ├── api/                   # API specifications
│   ├── architecture/          # System design docs
│   ├── implementation/        # Implementation guides
│   └── ai-prompts/           # AI development prompts
├── src/                       # Source code
│   ├── HomeManager.Api/       # API Gateway
│   ├── HomeManager.Identity/  # Identity Service
│   ├── HomeManager.Todos/     # Todo Service
│   ├── HomeManager.Budget/    # Budget Service
│   ├── HomeManager.AI/        # AI Gateway Service
│   ├── HomeManager.Web/       # React Web App
│   ├── HomeManager.Mobile/    # React Native App
│   └── HomeManager.Shared/    # Shared libraries
├── tests/                     # Test projects
├── infrastructure/            # IaC and deployment
│   ├── terraform/             # Terraform modules
│   ├── kubernetes/            # K8s manifests
│   └── docker/               # Docker configurations
├── tools/                     # Development tools
└── scripts/                   # Build and deployment scripts
```

## Service Organization

### .NET Microservices Structure
```
src/HomeManager.ServiceName/
├── Domain/                    # Domain models and business logic
│   ├── Entities/             # Domain entities
│   ├── ValueObjects/         # Value objects
│   ├── Repositories/         # Repository interfaces
│   └── Services/             # Domain services
├── Application/              # Application layer
│   ├── Commands/             # CQRS commands
│   ├── Queries/              # CQRS queries
│   ├── Handlers/             # Command/query handlers
│   ├── Services/             # Application services
│   └── DTOs/                 # Data transfer objects
├── Infrastructure/           # Infrastructure layer
│   ├── Data/                 # Database contexts
│   ├── Repositories/         # Repository implementations
│   ├── Services/             # External service integrations
│   └── Configuration/        # Service configuration
├── Api/                      # Presentation layer
│   ├── Controllers/          # API controllers (if used)
│   ├── Endpoints/            # Minimal API endpoints
│   ├── Middleware/           # Custom middleware
│   └── Validators/           # Input validation
└── Properties/               # Assembly properties
```