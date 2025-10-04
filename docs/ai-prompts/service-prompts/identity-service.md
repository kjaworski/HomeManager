# Identity Service AI Development Prompts

> **AI Context File for HomeManager Identity Service Development**

This file contains optimized prompts and context for AI-assisted development of the HomeManager Identity Service.

## Service Overview Context

```markdown
# Identity Service Context for AI Development

## Service Purpose
The Identity Service handles user authentication, authorization, and family management for HomeManager.

## Technology Stack
- Framework: ASP.NET Core 9 with Minimal APIs
- Database: PostgreSQL 15+ with Entity Framework Core
- Authentication: JWT with refresh tokens
- Validation: FluentValidation
- Testing: xUnit with Testcontainers

## Key Requirements
- Multi-tenant family isolation
- Secure password hashing with BCrypt
- JWT token management with refresh capability
- Family administration and member management
- Email verification workflow
- Comprehensive error handling
```

## Core Development Prompts

### Service Creation Prompt
```markdown
Create an Identity Service for HomeManager that handles user authentication and family management.

Requirements:
- .NET 10 with minimal APIs and Clean Architecture
- PostgreSQL database with Entity Framework Core
- JWT authentication with refresh tokens
- Family-based multi-tenancy
- Password hashing with BCrypt
- Comprehensive error handling
- OpenAPI documentation
- Unit and integration tests

Entities needed:
- User (id, email, password_hash, first_name, last_name, display_name, profile_picture_url, timezone, language, email_verified, created_at, updated_at)
- Family (id, name, description, admin_id, created_at, updated_at)
- FamilyMember (id, family_id, user_id, role, joined_at)
- RefreshToken (id, user_id, token_hash, expires_at, created_at, revoked_at)

API Endpoints:
- POST /auth/register - User registration
- POST /auth/login - User authentication
- POST /auth/refresh - Token refresh
- GET /users/profile - Get user profile
- PUT /users/profile - Update user profile
- GET /families - Get user's families
- POST /families - Create new family
- GET /families/{id}/members - Get family members
- POST /families/{id}/members - Invite family member

Follow HomeManager coding standards:
- Nullable reference types enabled
- Comprehensive error handling with custom exceptions
- Structured logging with correlation IDs
- Input validation with FluentValidation
- Repository pattern with interface segregation
```

### Authentication Logic Prompt
```markdown
Implement secure authentication logic for HomeManager Identity Service.

Requirements:
- BCrypt password hashing with appropriate work factor
- JWT access tokens (1 hour expiry) and refresh tokens (30 days)
- Secure token storage and validation
- Failed login attempt tracking
- Email verification workflow
- Password reset functionality

Security considerations:
- Timing attack prevention
- Secure random token generation
- Token blacklisting for logout
- Rate limiting for authentication endpoints
- Audit logging for security events

Generate the following components:
- IPasswordHasher interface and BCryptPasswordHasher implementation
- IJwtTokenGenerator interface and implementation
- IAuthenticationService with login/logout/refresh methods
- Custom exceptions for authentication failures
- Security middleware for JWT validation
```

### Database Design Prompt
```markdown
Design PostgreSQL database schema for HomeManager Identity Service.

Requirements:
- Support for multi-tenant family structure
- User management with secure password storage
- Family administration and member roles
- Refresh token management
- Audit trails and timestamps

Constraints:
- Email uniqueness across all users
- Family admin cannot be deleted while family exists
- Cascade delete for family member relationships
- Proper indexing for query performance

Generate:
- SQL DDL scripts for table creation
- Entity Framework Core entity configurations
- Database migrations
- Seed data for development
- Performance indexes

Follow PostgreSQL best practices:
- Use UUIDs for primary keys
- Proper foreign key constraints
- Timestamp columns with timezone
- Appropriate column types and constraints
```

### API Endpoint Implementation
```markdown
Implement REST API endpoints for HomeManager Identity Service using ASP.NET Core Minimal APIs.

Requirements:
- RESTful design with proper HTTP methods and status codes
- Request/response DTOs with validation
- OpenAPI documentation with examples
- Comprehensive error handling
- Logging and monitoring

Endpoints to implement:
1. POST /auth/register - User registration
2. POST /auth/login - User authentication  
3. POST /auth/refresh - Token refresh
4. GET /users/profile - Get current user profile
5. PUT /users/profile - Update user profile
6. GET /families - Get user's families
7. POST /families - Create new family
8. GET /families/{id}/members - Get family members
9. POST /families/{id}/members - Invite family member

For each endpoint:
- Input validation with FluentValidation
- Proper HTTP status codes (200, 201, 400, 401, 404, 409, 500)
- Structured error responses
- Request/response logging
- OpenAPI documentation with examples

Follow API design best practices:
- Consistent naming conventions
- Proper use of HTTP methods
- Meaningful error messages
- Security headers
```

## Testing Prompts

### Unit Test Generation
```markdown
Generate comprehensive unit tests for HomeManager Identity Service.

Test the following components:
- UserService with all CRUD operations
- FamilyService with member management
- AuthenticationService with login/logout/refresh
- PasswordHasher with various password scenarios
- JwtTokenGenerator with token validation

Test scenarios:
- Happy path scenarios
- Edge cases and error conditions
- Security validations
- Boundary conditions
- Null/empty input handling

Use:
- xUnit testing framework
- Moq for mocking dependencies
- FluentAssertions for readable assertions
- In-memory database for repository tests
- AAA (Arrange, Act, Assert) pattern

Generate tests with:
- Descriptive test names following convention
- Proper test data setup
- Comprehensive assertions
- Exception testing
- Async/await patterns
```

### Integration Test Prompt
```markdown
Create integration tests for HomeManager Identity Service using Testcontainers.

Requirements:
- Test against real PostgreSQL database
- End-to-end API testing
- Database migration testing
- Authentication flow testing
- Family management workflows

Test scenarios:
- User registration and login flow
- Family creation and member invitation
- Token refresh and validation
- Profile management
- Error handling and edge cases

Use:
- Testcontainers.PostgreSQL for database
- WebApplicationFactory for API testing
- HttpClient for endpoint testing
- Custom test fixtures for data setup
- Parallel test execution safety

Generate:
- Base test class with Testcontainers setup
- API endpoint integration tests
- Database operation tests
- Authentication workflow tests
- Performance and load tests
```

## Deployment Prompts

### Docker Configuration
```markdown
Create Docker configuration for HomeManager Identity Service.

Requirements:
- Multi-stage Dockerfile for optimized image size
- Support for development and production environments
- Proper security practices
- Health checks and monitoring

Generate:
- Dockerfile with multi-stage build
- docker-compose.yml for local development
- Environment variable configuration
- Health check endpoints
- Security scanning configuration

Follow Docker best practices:
- Non-root user execution
- Minimal base images
- Layer optimization
- Secret management
- Resource limits
```

### Kubernetes Deployment
```markdown
Create Kubernetes deployment manifests for HomeManager Identity Service.

Requirements:
- Scalable deployment configuration
- Service discovery and load balancing
- ConfigMap and Secret management
- Health checks and readiness probes
- Resource limits and requests

Generate:
- Deployment manifest
- Service manifest
- ConfigMap for configuration
- Secret for sensitive data
- Ingress for external access
- HorizontalPodAutoscaler

Follow Kubernetes best practices:
- Resource quotas
- Security contexts
- Pod disruption budgets
- Monitoring and logging
```

## Troubleshooting Prompts

### Common Issues Resolution
```markdown
Provide solutions for common Identity Service issues:

1. JWT token validation failures
2. Database connection issues
3. Password hashing problems
4. Family member permission errors
5. Email verification failures
6. Performance optimization
7. Security vulnerabilities
8. Deployment issues

For each issue, provide:
- Problem description
- Root cause analysis
- Step-by-step solution
- Prevention strategies
- Monitoring recommendations
```