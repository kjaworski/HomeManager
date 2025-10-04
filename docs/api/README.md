# API Specifications - HomeManager Services

This directory contains detailed API specifications for all HomeManager microservices. Each service follows OpenAPI 3.0 specification format for consistency and tool compatibility.

## Service APIs

### Core Services
- **[Identity Service API](identity-service.yaml)** - User authentication, authorization, and family management
- **[Todo Service API](todo-service.yaml)** - Task creation, assignment, and tracking
- **[Budget Service API](budget-service.yaml)** - Expense tracking and budget management
- **[AI Gateway API](ai-gateway.yaml)** - AI-powered insights and recommendations

### Supporting Services
- **[Notification Service API](notification-service.yaml)** - Push notifications and alerts
- **[File Storage API](file-storage.yaml)** - Receipt images and document management

## API Standards

### Common Patterns

#### Authentication
All APIs use JWT Bearer token authentication:
```yaml
security:
  - bearerAuth: []

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

#### Error Responses
Consistent error response format across all services:
```yaml
components:
  schemas:
    ErrorResponse:
      type: object
      required:
        - error
        - message
        - timestamp
      properties:
        error:
          type: string
          description: Error code
          example: VALIDATION_ERROR
        message:
          type: string
          description: Human-readable error message
          example: The request contains invalid data
        details:
          type: array
          items:
            type: object
          description: Detailed validation errors
        timestamp:
          type: string
          format: date-time
          description: Error timestamp
        traceId:
          type: string
          description: Request trace ID for debugging
```

#### Pagination
Standard pagination for list endpoints:
```yaml
components:
  schemas:
    PaginatedResponse:
      type: object
      required:
        - data
        - pagination
      properties:
        data:
          type: array
          items:
            type: object
        pagination:
          type: object
          required:
            - page
            - limit
            - total
            - totalPages
          properties:
            page:
              type: integer
              minimum: 1
              description: Current page number
            limit:
              type: integer
              minimum: 1
              maximum: 100
              description: Items per page
            total:
              type: integer
              minimum: 0
              description: Total number of items
            totalPages:
              type: integer
              minimum: 0
              description: Total number of pages
            hasNext:
              type: boolean
              description: Whether there are more pages
            hasPrev:
              type: boolean
              description: Whether there are previous pages
```

#### Filtering and Sorting
Standard query parameters for list endpoints:
```yaml
parameters:
  - name: sort
    in: query
    description: Sort field and direction
    schema:
      type: string
      pattern: '^[a-zA-Z_][a-zA-Z0-9_]*:(asc|desc)$'
      example: 'created_at:desc'
  - name: filter
    in: query
    description: Filter criteria in format field:operator:value
    schema:
      type: array
      items:
        type: string
        pattern: '^[a-zA-Z_][a-zA-Z0-9_]*:(eq|ne|gt|gte|lt|lte|like|in):.*$'
      example: ['status:eq:active', 'amount:gte:100']
```

### HTTP Status Codes

Standard status codes used across all APIs:

#### Success Codes
- `200 OK` - Successful GET, PUT, PATCH requests
- `201 Created` - Successful POST requests
- `204 No Content` - Successful DELETE requests

#### Client Error Codes
- `400 Bad Request` - Invalid request data
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource not found
- `409 Conflict` - Resource conflict (e.g., duplicate email)
- `422 Unprocessable Entity` - Validation errors

#### Server Error Codes
- `500 Internal Server Error` - Unexpected server error
- `502 Bad Gateway` - Upstream service error
- `503 Service Unavailable` - Service temporarily unavailable

### Request/Response Headers

#### Required Request Headers
```yaml
headers:
  Authorization:
    description: JWT Bearer token
    required: true
    example: "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  Content-Type:
    description: Request content type
    required: true (for POST/PUT/PATCH)
    example: "application/json"
  X-Request-ID:
    description: Unique request identifier for tracing
    required: false
    example: "550e8400-e29b-41d4-a716-446655440000"
```

#### Standard Response Headers
```yaml
headers:
  X-Request-ID:
    description: Request identifier for tracing
    example: "550e8400-e29b-41d4-a716-446655440000"
  X-Rate-Limit-Remaining:
    description: Number of requests remaining in current window
    example: "99"
  X-Rate-Limit-Reset:
    description: Rate limit reset time
    example: "1609459200"
```

### Data Formats

#### Date/Time
All timestamps use ISO 8601 format in UTC:
```yaml
type: string
format: date-time
example: "2024-10-03T14:30:00.000Z"
```

#### Currency
Currency amounts use decimal format with currency code:
```yaml
amount:
  type: number
  format: decimal
  multipleOf: 0.01
  example: 156.78
currency:
  type: string
  pattern: '^[A-Z]{3}$'
  example: "USD"
```

#### UUIDs
All entity IDs use UUID v4 format:
```yaml
type: string
format: uuid
example: "550e8400-e29b-41d4-a716-446655440000"
```

## Testing

### API Testing Tools
- **OpenAPI Validation**: Ensure API specs match implementation
- **Contract Testing**: Verify API contracts between services
- **Load Testing**: Performance testing with realistic data
- **Security Testing**: Authentication and authorization testing

### Test Data
Each API specification includes example requests and responses for:
- Happy path scenarios
- Validation error cases
- Authentication/authorization failures
- Edge cases and error conditions

### Mock Servers
API specifications can be used to generate mock servers for:
- Frontend development
- Integration testing
- Documentation examples
- Client SDK generation

## Code Generation

### Client SDKs
API specifications support automatic generation of:
- TypeScript client for React applications
- Swift client for iOS applications
- Kotlin client for Android applications
- C# client for server-to-server communication

### Server Stubs
Specifications can generate server stubs for:
- ASP.NET Core controllers
- Validation models
- Documentation sites
- Testing scaffolds

## Documentation

### Interactive Documentation
Each API specification generates interactive documentation with:
- Endpoint exploration
- Request/response examples
- Authentication testing
- Schema visualization

### Integration Guides
Each service includes integration examples for:
- Common use cases
- Error handling patterns
- Rate limiting strategies
- Caching recommendations

---

**AI Development Notes:**
- OpenAPI specifications optimized for AI-assisted API development
- Consistent patterns for easy AI code generation
- Comprehensive examples for AI training and reference
- Clear error handling patterns for robust AI-generated code