# Trips Planner Service AI Development Prompts

> **AI Assistant Guide** 🤖  
> Specialized prompts for developing the trips planner service with AI assistance

## Service Overview Prompt

```markdown
I'm developing a Trips Planner Service for a family management application called HomeManager. 

**Service Purpose:**
- Manage family trip planning and coordination
- Create and organize trip itineraries
- Track trip expenses with receipt management
- Provide AI-powered travel suggestions
- Handle multi-participant trip coordination

**Technology Stack:**
- ASP.NET Core 10 with Minimal APIs
- Amazon DocumentDB (MongoDB compatible)
- Amazon Bedrock (Claude models for AI features)
- Google Maps Platform API
- Amazon S3 (receipt storage)
- xUnit with Testcontainers

**Key Features:**
- CRUD operations for trips
- Itinerary management with scheduling
- Expense tracking with categories
- AI travel suggestions (activities, restaurants, accommodations)
- Budget optimization recommendations
- Multi-user trip participation

Please help me implement [specific feature] following the established patterns from the todo-service and budget-service.
```

## Domain Modeling Prompts

### Trip Entity Design
```markdown
Help me design the Trip entity for a family trips planner service.

**Requirements:**
- Each trip belongs to a family
- Has destination (city, country, coordinates, timezone)
- Date range (start/end dates)
- Status tracking (planning, confirmed, in-progress, completed, cancelled)
- Budget management with category breakdowns
- Multiple participants with roles (organizer, participant)
- Audit fields (created by, timestamps)

**Considerations:**
- Use record types for immutability
- Support for different currencies
- Flexible budget categories
- Participant management
- Integration with AI suggestions

Please create the C# record types with proper validation and relationships.
```

### Itinerary Management
```markdown
Design the itinerary system for trip planning with these requirements:

**Itinerary Features:**
- Multiple item types: accommodation, transportation, activity, meal, other
- Time-based scheduling with start/end times
- Location integration with Google Maps
- Booking status tracking
- Cost estimation
- Notes and booking references

**Data Considerations:**
- Link to parent trip
- Support for overlapping time slots
- Location enrichment with place IDs
- Flexible cost tracking
- Booking workflow support

Create the entity models and service interfaces for comprehensive itinerary management.
```

### Expense Tracking
```markdown
Create an expense tracking system for trip management:

**Requirements:**
- Link expenses to specific trips
- Category-based organization (accommodation, food, activities, etc.)
- Multi-currency support
- Receipt attachment (S3 integration)
- Track who paid for each expense
- Date-based expense recording

**Features Needed:**
- CRUD operations for expenses
- Receipt upload handling
- Expense reporting and analytics
- Budget vs actual tracking
- Currency conversion support

Design the expense entity and related service layer.
```

## API Development Prompts

### RESTful API Design
```markdown
Design RESTful API endpoints for the trips planner service:

**Trip Management:**
- GET /api/trips (with filtering: status, date range, pagination)
- POST /api/trips (create new trip)
- GET /api/trips/{tripId}
- PUT /api/trips/{tripId}
- DELETE /api/trips/{tripId}
- POST /api/trips/{tripId}/participants (add participant)
- DELETE /api/trips/{tripId}/participants/{userId}

**Itinerary Management:**
- GET /api/trips/{tripId}/itinerary
- POST /api/trips/{tripId}/itinerary
- PUT /api/trips/{tripId}/itinerary/{itemId}
- DELETE /api/trips/{tripId}/itinerary/{itemId}

**Expense Management:**
- GET /api/trips/{tripId}/expenses
- POST /api/trips/{tripId}/expenses
- PUT /api/trips/{tripId}/expenses/{expenseId}
- DELETE /api/trips/{tripId}/expenses/{expenseId}
- POST /api/trips/{tripId}/expenses/{expenseId}/receipt

**AI Features:**
- GET /api/trips/{tripId}/ai/suggestions?type=activities|restaurants|accommodations|transportation|budget-optimization

Implement using ASP.NET Core Minimal APIs with proper validation, error handling, and OpenAPI documentation.
```

### Minimal API Implementation
```markdown
Convert these controller-based endpoints to ASP.NET Core Minimal APIs:

**Current Controller Structure:**
[Shows existing controller code]

**Requirements:**
- Use endpoint groups for organization
- Implement proper request/response DTOs
- Add comprehensive OpenAPI documentation
- Include validation attributes
- Handle authentication/authorization
- Implement proper error responses

Create the minimal API endpoints with proper routing, validation, and documentation.
```

## Data Access Prompts

### MongoDB Repository Pattern
```markdown
Implement MongoDB repositories for the trips planner service:

**Collections Needed:**
- trips (main trip data)
- itinerary (trip itinerary items)
- expenses (trip expenses)

**Repository Requirements:**
- Generic repository pattern
- Async operations throughout
- Proper indexing strategy
- Complex filtering support
- Pagination implementation
- Aggregation queries for reporting

**MongoDB Features to Use:**
- Document relationships
- Compound indexes
- Aggregation pipelines
- Text search capabilities
- Geospatial queries for location-based features

Create the repository interfaces and MongoDB implementations with proper error handling and logging.
```

### Complex Queries
```markdown
Help me implement complex MongoDB queries for trip analytics:

**Query Requirements:**
1. Trip dashboard data (upcoming trips, recent expenses, budget status)
2. Expense reporting by category and date range
3. Travel patterns analysis (frequent destinations, spending habits)
4. Itinerary optimization (time conflicts, travel distance calculations)
5. Budget vs actual analysis with variance reporting

**Technical Considerations:**
- Use MongoDB aggregation pipelines
- Implement proper indexing for performance
- Handle multi-collection joins
- Support real-time data updates
- Optimize for family-scoped queries

Create the aggregation pipelines and repository methods for these analytics features.
```

## AI Integration Prompts

### Amazon Bedrock Integration
```markdown
Implement AI-powered travel suggestions using Amazon Bedrock:

**AI Features to Build:**
1. Activity suggestions based on destination and family preferences
2. Restaurant recommendations with dietary considerations
3. Accommodation suggestions within budget
4. Transportation options and optimization
5. Budget optimization recommendations
6. Itinerary conflict detection and resolution

**Technical Requirements:**
- Use Claude models via Amazon Bedrock
- Implement proper prompt engineering
- Handle AI response parsing
- Implement fallback mechanisms
- Cache suggestions for performance
- Track AI usage for cost management

**Integration Points:**
- Trip creation (auto-generate suggestions)
- Itinerary planning (real-time suggestions)
- Budget optimization (ongoing analysis)
- Expense categorization (AI-assisted)

Create the AI service layer with proper error handling and response processing.
```

### Prompt Engineering
```markdown
Design effective prompts for travel AI suggestions:

**Context to Include:**
- Trip destination and dates
- Family composition and preferences
- Budget constraints
- Existing itinerary items
- Local timezone and cultural considerations

**Prompt Types Needed:**
1. Activity suggestions prompt
2. Restaurant recommendations prompt
3. Accommodation search prompt
4. Transportation planning prompt
5. Budget optimization prompt
6. Itinerary conflict resolution prompt

**Response Format Requirements:**
- Structured JSON responses
- Consistent data format
- Cost estimates included
- Location coordinates when applicable
- Confidence scores for recommendations

Create comprehensive prompt templates with proper context injection and response parsing.
```

## External Service Integration

### Google Maps Integration
```markdown
Integrate Google Maps Platform APIs for location services:

**APIs to Use:**
- Places API (location search and details)
- Geocoding API (address to coordinates)
- Distance Matrix API (travel time calculations)
- Time Zone API (timezone information)

**Features to Implement:**
- Location autocomplete for trip destinations
- Place details enrichment for itinerary items
- Travel time calculations between itinerary items
- Geospatial search for nearby attractions
- Location validation and normalization

**Technical Considerations:**
- API key management and rotation
- Rate limiting and quota management
- Caching strategies for API responses
- Error handling for API failures
- Cost optimization through intelligent caching

Create the Google Maps service layer with proper abstraction and error handling.
```

### File Upload Service
```markdown
Implement receipt upload functionality using Amazon S3:

**Requirements:**
- Secure file upload to S3
- Image format validation (JPEG, PNG, PDF)
- File size limits and validation
- Virus scanning integration
- Metadata extraction (OCR for expense data)
- Thumbnail generation for images

**Security Considerations:**
- Pre-signed URLs for secure uploads
- Content-Type validation
- File name sanitization
- Access control (family-scoped access)
- Audit logging for file operations

**Integration Points:**
- Expense creation workflow
- Receipt attachment to existing expenses
- Bulk receipt processing
- Receipt data extraction with AI

Create the file upload service with proper security and validation.
```

## Testing Prompts

### Unit Testing Strategy
```markdown
Create comprehensive unit tests for the trips planner service:

**Test Categories:**
1. Domain model validation and business logic
2. Service layer operations and error handling
3. Repository pattern with mocked dependencies
4. AI service integration with mocked responses
5. API endpoint validation and response formatting

**Testing Frameworks:**
- xUnit for test framework
- Moq for mocking dependencies
- FluentAssertions for readable assertions
- AutoFixture for test data generation
- Bogus for realistic fake data

**Test Scenarios to Cover:**
- Happy path operations
- Validation failures and error scenarios
- Concurrency and race condition handling
- AI service failures and fallbacks
- External API integration failures

Create test classes with comprehensive coverage and realistic test scenarios.
```

### Integration Testing
```markdown
Design integration tests for the trips planner service:

**Integration Test Scenarios:**
1. End-to-end trip creation workflow
2. Itinerary management with time conflict detection
3. Expense tracking with receipt upload
4. AI suggestion generation and processing
5. Multi-user trip participation workflows

**Infrastructure Requirements:**
- Testcontainers for MongoDB
- LocalStack for AWS services simulation
- Test doubles for external APIs
- Database migration and cleanup
- Test data seeding strategies

**Test Environment Setup:**
- Isolated test databases
- Mock external services
- Configuration management
- Parallel test execution
- CI/CD integration

Create integration test setup with proper isolation and realistic scenarios.
```

## Performance & Optimization

### Caching Strategy
```markdown
Design a comprehensive caching strategy for the trips planner service:

**Data to Cache:**
- AI-generated travel suggestions
- Google Maps API responses (places, geocoding)
- Trip dashboard data
- Frequently accessed itinerary data
- Exchange rates for currency conversion

**Caching Layers:**
- Redis for distributed caching
- In-memory caching for hot data
- HTTP response caching for API endpoints
- CDN caching for static content (receipts)

**Cache Invalidation Strategies:**
- Time-based expiration for external API data
- Event-driven invalidation for user data
- Cache warming for predictable access patterns
- Graceful degradation when cache is unavailable

Implement the caching infrastructure with proper monitoring and metrics.
```

### Database Optimization
```markdown
Optimize MongoDB performance for the trips planner service:

**Indexing Strategy:**
- Compound indexes for common query patterns
- Text indexes for search functionality
- Geospatial indexes for location queries
- Sparse indexes for optional fields

**Query Optimization:**
- Aggregation pipeline optimization
- Projection to limit returned fields
- Pagination with proper sorting
- Connection pooling configuration

**Performance Monitoring:**
- Query execution time tracking
- Index usage analysis
- Connection pool monitoring
- Document size optimization

Create the optimal database configuration and monitoring setup.
```

## Security & Compliance

### Data Security
```markdown
Implement comprehensive security for the trips planner service:

**Authentication & Authorization:**
- JWT token validation
- Family-scoped data access
- Role-based permissions (organizer vs participant)
- API rate limiting per user/family

**Data Protection:**
- Encryption at rest for sensitive data
- HTTPS enforcement for all communications
- PII data anonymization for analytics
- Secure file handling for receipts

**Privacy Compliance:**
- GDPR compliance for EU users
- Data retention policies
- User data export functionality
- Right to deletion implementation

**Security Monitoring:**
- Authentication failure tracking
- Suspicious activity detection
- API abuse monitoring
- Security event logging

Create the security infrastructure with proper audit trails and compliance features.
```

## Deployment & DevOps

### Containerization
```markdown
Create Docker containers for the trips planner service:

**Container Requirements:**
- Multi-stage build for optimized images
- Security scanning integration
- Health check implementation
- Configuration through environment variables
- Proper logging configuration

**Kubernetes Deployment:**
- Deployment manifests with resource limits
- Service discovery configuration
- ConfigMap and Secret management
- Horizontal pod autoscaling
- Ingress configuration for external access

**CI/CD Pipeline:**
- Automated testing in containers
- Security scanning for vulnerabilities
- Image registry integration
- Automated deployment to staging/production
- Rollback strategies for failed deployments

Create the complete containerization and deployment pipeline.
```

### Monitoring & Observability
```markdown
Implement comprehensive monitoring for the trips planner service:

**Metrics to Track:**
- API response times and error rates
- Database query performance
- AI service usage and costs
- External API call patterns
- User engagement metrics

**Logging Strategy:**
- Structured logging with correlation IDs
- Business event logging
- Error tracking and alerting
- Performance bottleneck identification
- Security event monitoring

**Health Checks:**
- Application health endpoints
- Dependency health monitoring
- Readiness and liveness probes
- Circuit breaker patterns
- Graceful degradation monitoring

**Alerting Rules:**
- SLA violation alerts
- Error rate threshold alerts
- Performance degradation notifications
- Security incident alerts
- Cost threshold warnings

Create the complete observability stack with dashboards and alerting.
```

## Migration & Maintenance

### Data Migration
```markdown
Plan data migration strategies for the trips planner service:

**Migration Scenarios:**
- Legacy data import from existing systems
- Schema evolution and versioning
- Data format changes and transformations
- Index recreation and optimization
- Performance impact minimization

**Migration Tools:**
- Custom migration scripts
- Database versioning tools
- Data validation and verification
- Rollback procedures
- Progress monitoring and reporting

Create comprehensive migration procedures with proper testing and validation.
```

These prompts provide comprehensive guidance for AI-assisted development of the trips planner service, covering all aspects from initial design through deployment and maintenance.