# HomeManager

A modern home management solution built with ASP.NET Core WebAPI and React.

## Project Structure

This solution is designed to support multiple applications:

```
HomeManager/
├── .vscode/                      # VS Code workspace configuration
│   ├── settings.json            # Editor and C# DevKit settings
│   ├── tasks.json               # Build, test, watch tasks
│   └── launch.json              # Debug configuration
├── src/                          # Source code
│   ├── HomeManager.Api/          # Main WebAPI project (✓ Created)
│   ├── HomeManager.SecondApi/    # Future: Second WebAPI project
│   └── HomeManager.Web/          # Future: React frontend
├── tests/                        # Test projects
│   ├── HomeManager.Api.Tests/    # API unit and integration tests (✓ Created)
│   └── HomeManager.SecondApi.Tests/ # Future: Second API tests
├── docs/                         # Documentation (✓ Created)
├── HomeManager.sln               # Solution file (✓ Created)
├── .gitignore                    # Git ignore rules (✓ Created)
└── README.md                     # This file
```

## Technology Stack

- **.NET 10** - Latest .NET framework
- **C# Latest** - Latest C# language features (configured with `LangVersion: latest`)
- **ASP.NET Core WebAPI** - RESTful API framework with top-level statements
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
dotnet build src/HomeManager.Api
```

### Running the API

```bash
# Run the API project
dotnet run --project src/HomeManager.Api

# Or navigate to the project directory
cd src/HomeManager.Api
dotnet run
```

The API will be available at:
- HTTPS: `https://localhost:7139`
- HTTP: `http://localhost:5139`

### Running Tests

```bash
# Run all tests
dotnet test

# Run tests with coverage
dotnet test --collect:"XPlat Code Coverage"

# Run specific test project
dotnet test tests/HomeManager.Api.Tests
```

### Development

The project uses top-level statements in `Program.cs` for a clean, minimal setup. The API includes:

- Swagger/OpenAPI documentation
- Development exception handling
- HTTPS redirection
- Authorization support (ready for implementation)

## API Documentation

When running in development mode, Swagger UI is available at:
- `https://localhost:7139/swagger`

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

### Available VS Code Tasks:
- **Build** - `Ctrl+Shift+P` → "Tasks: Run Task" → "build"
- **Test** - `Ctrl+Shift+P` → "Tasks: Run Task" → "test"
- **Watch** - `Ctrl+Shift+P` → "Tasks: Run Task" → "watch" (hot reload)
- **Debug** - Press `F5` to start debugging (opens Swagger automatically)

## Contributing

1. Follow the existing code style and conventions
2. Write tests for new features
3. Update documentation as needed
4. Create pull requests for review

## License

This project is licensed under the MIT License - see the LICENSE file for details.