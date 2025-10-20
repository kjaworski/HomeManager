# HomeManager

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A modern home management solution built with ASP.NET Core WebAPI and React.

## Project Structure

This solution is designed to support multiple applications:

```text
HomeManager/
├── .vscode/                           # VS Code workspace configuration
│   ├── settings.json                 # Editor and C# DevKit settings
│   ├── tasks.json                    # Build, test, watch tasks
│   └── launch.json                   # Debug configuration
├── src/                               # Source code
│   └── Services/
│       └── HomeManager.TodoService/  # Todo service API (✓ Created)
├── tests/                             # Test projects
│   └── Services/
│       └── HomeManager.TodoService.Tests/ # Todo API tests (✓ Created)
├── docs/                              # Documentation (✓ Created)
├── HomeManager.sln                    # Solution file (✓ Created)
├── .gitignore                         # Git ignore rules (✓ Created)
└── README.md                          # This file
```

## Technology Stack

- **.NET 10** - Latest .NET framework
- **C# Latest** - Latest C# language features (configured with `LangVersion: latest`)
- **ASP.NET Core WebAPI** - RESTful API framework with top-level statements
- **Microsoft OpenAPI** - Built-in OpenAPI 3.0 support (native to .NET 10)
- **Scalar UI** - Modern, fast API documentation interface (Microsoft recommended)
- **xUnit** - Testing framework
- **Microsoft.AspNetCore.Mvc.Testing** - Integration testing

## Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0) (RC or later)
- [Visual Studio Code](https://code.visualstudio.com/) with C# extension

## Getting Started

### Building the Solution

```bash
# Restore dependencies
dotnet restore

# Build all projects
dotnet build

# Build specific project
dotnet build src/Services/HomeManager.TodoService
```

### Running the API

```bash
# Run the Todo service
dotnet run --project src/Services/HomeManager.TodoService

# Or navigate to the project directory
cd src/Services/HomeManager.TodoService
dotnet run
```

The Todo service will be available at:

- HTTPS: `https://localhost:7154` (when using HTTPS profile)
- HTTP: `http://localhost:5179` (default profile)

### Running Tests

```bash
# Run all tests
dotnet test

# Run tests with coverage
dotnet test --collect:"XPlat Code Coverage"

# Run specific test project
dotnet test tests/Services/HomeManager.TodoService.Tests
```

### Development

The project uses top-level statements in `Program.cs` for a clean, minimal setup. The API includes:

- Swagger/OpenAPI documentation
- Development exception handling
- HTTPS redirection
- Authorization support (ready for implementation)

## API Documentation

When running in development mode, you can access the interactive API documentation:

- **Scalar UI**: `http://localhost:5179/scalar/v1` - Modern, fast API documentation interface
- **OpenAPI Spec**: `http://localhost:5179/openapi/v1.json` - Machine-readable API specification
- **Quick Access**: `http://localhost:5179/` or `http://localhost:5179/swagger` - Redirects to Scalar UI

## 📚 Documentation

For comprehensive documentation including architecture decisions, implementation guides, and detailed technical information, see:

**[📖 Full Documentation](docs/README.md)**

Key documentation sections:
- [System Architecture](docs/architecture/system-overview.md) - Microservices design and service communication
- [Architecture Decision Records (ADRs)](docs/adr/) - Technical decisions and rationale
- [API Specifications](docs/api/README.md) - Detailed API contracts and standards
- [Implementation Guides](docs/implementation/README.md) - Step-by-step development guides
- [AI-Assisted Development](docs/ai-prompts/README.md) - Prompts and patterns for AI tools

## Future Enhancements

This repository is structured to accommodate:

1. **Second WebAPI Project** - Additional API services
2. **React Frontend** - Modern web interface
3. **Shared Libraries** - Common business logic and DTOs
4. **Microservices Architecture** - Scalable service design

## Development Guidelines

- Follow [C# Coding Conventions](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/inside-a-program/coding-conventions)
- Write unit tests for all business logic
- Use integration tests for API endpoints
- Document public APIs with XML comments
- Follow RESTful API design principles

## VS Code Configuration

This project is optimized for VS Code development:

- **`.vscode/settings.json`** - Workspace settings with C# DevKit configuration
- **`.vscode/tasks.json`** - Pre-configured tasks (build, test, watch, publish, clean)
- **`.vscode/launch.json`** - F5 debugging configuration with automatic Swagger opening
- **C# DevKit extension** - Provides IntelliSense, debugging, and project management
- **Integrated terminal** - For running dotnet commands
- **Git integration** - Version control with properly configured .gitignore

### Available VS Code Tasks

- **Build** - `Ctrl+Shift+P` → "Tasks: Run Task" → "build"
- **Test** - `Ctrl+Shift+P` → "Tasks: Run Task" → "test"
- **Watch** - `Ctrl+Shift+P` → "Tasks: Run Task" → "watch" (hot reload)
- **Debug** - Press `F5` to start debugging (opens Swagger automatically)

## Contributing

1. Follow the existing code style and conventions
2. Write tests for new features
3. Update documentation as needed
4. Create pull requests for review

## Documentation & References

### Microsoft .NET 10 & OpenAPI

- [Microsoft OpenAPI Documentation](https://aka.ms/aspnet/openapi) - Official .NET 10 OpenAPI guide
- [Generate OpenAPI documents](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/aspnetcore-openapi) - Comprehensive OpenAPI guide
- [ASP.NET Core Minimal APIs](https://learn.microsoft.com/en-us/aspnet/core/tutorials/min-web-api) - Microsoft's minimal API tutorial with OpenAPI
- [.NET 10 Release Notes](https://github.com/dotnet/core/tree/main/release-notes/10.0) - What's new in .NET 10

### Scalar API Documentation

- [Scalar Documentation](https://github.com/scalar/scalar) - Official Scalar GitHub repository  
- [Scalar ASP.NET Core](https://www.nuget.org/packages/Scalar.AspNetCore) - NuGet package for ASP.NET Core
- [Scalar Live Demo](https://github.com/scalar/scalar#features) - Interactive demo and features overview
- [Scalar vs Swagger UI](https://github.com/scalar/scalar#why-scalar) - Modern, fast API documentation interface

### Microsoft Recommendations

- [OpenAPI in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/aspnetcore-openapi) - Official Microsoft OpenAPI documentation
- [API Documentation Best Practices](https://learn.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger) - Microsoft's guidance
- [Minimal APIs](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis) - Official Microsoft documentation

## License

This project is licensed under the MIT License - see the LICENSE file for details.
