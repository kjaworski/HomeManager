# Trips Planner Service Implementation Guide

> **AI Development Context** 🤖  
> This guide provides comprehensive implementation details for AI-assisted development

## Overview

The Trips Planner Service manages family trip planning, itinerary creation, expense tracking, and AI-powered travel suggestions. It provides comprehensive travel coordination features for families to plan, organize, and track their trips efficiently.

## Architecture

### Technology Stack
```yaml
Framework: ASP.NET Core 10 with Minimal APIs
Database: Amazon DocumentDB (MongoDB compatible)
Caching: Amazon ElastiCache Redis
AI Integration: Amazon Bedrock (Claude models)
Maps Integration: Google Maps Platform API
SDK: AWS SDK for .NET
Testing: xUnit with Testcontainers
Documentation: Swagger/OpenAPI 3.0
File Storage: Amazon S3 (receipt uploads)
```

### DocumentDB Collection Design
```json
{
  "trips": {
    "_id": "ObjectId",
    "tripId": "uuid",
    "familyId": "uuid",
    "title": "string",
    "description": "string",
    "destination": {
      "city": "string",
      "country": "string",
      "coordinates": {
        "latitude": "number",
        "longitude": "number"
      },
      "timezone": "string"
    },
    "startDate": "ISODate",
    "endDate": "ISODate",
    "status": "string", // planning, confirmed, in-progress, completed, cancelled
    "budget": {
      "totalBudget": "decimal",
      "currency": "string",
      "categoryBudgets": {
        "accommodation": "decimal",
        "transportation": "decimal",
        "food": "decimal",
        "activities": "decimal",
        "shopping": "decimal",
        "other": "decimal"
      }
    },
    "participants": [
      {
        "userId": "uuid",
        "role": "string", // organizer, participant
        "joinedAt": "ISODate"
      }
    ],
    "createdBy": "uuid",
    "createdAt": "ISODate",
    "updatedAt": "ISODate",
    "version": "number"
  },
  
  "itinerary": {
    "_id": "ObjectId",
    "itemId": "uuid",
    "tripId": "uuid",
    "title": "string",
    "description": "string",
    "type": "string", // accommodation, transportation, activity, meal, other
    "startTime": "ISODate",
    "endTime": "ISODate",
    "location": {
      "name": "string",
      "address": "string",
      "coordinates": {
        "latitude": "number",
        "longitude": "number"
      },
      "placeId": "string"
    },
    "estimatedCost": "decimal",
    "bookingStatus": "string", // not-booked, pending, confirmed, cancelled
    "bookingReference": "string",
    "notes": "string",
    "createdAt": "ISODate",
    "updatedAt": "ISODate"
  },
  
  "expenses": {
    "_id": "ObjectId",
    "expenseId": "uuid",
    "tripId": "uuid",
    "amount": "decimal",
    "currency": "string",
    "category": "string", // accommodation, transportation, food, activities, shopping, other
    "description": "string",
    "paidBy": "uuid",
    "expenseDate": "ISODate",
    "receipt": {
      "fileName": "string",
      "fileUrl": "string",
      "uploadedAt": "ISODate"
    },
    "createdAt": "ISODate",
    "updatedAt": "ISODate"
  }
}
```

### Indexes
```javascript
// Trips Collection
db.trips.createIndex({ "familyId": 1, "startDate": -1 })
db.trips.createIndex({ "tripId": 1 }, { unique: true })
db.trips.createIndex({ "status": 1, "startDate": -1 })
db.trips.createIndex({ "participants.userId": 1 })

// Itinerary Collection
db.itinerary.createIndex({ "tripId": 1, "startTime": 1 })
db.itinerary.createIndex({ "itemId": 1 }, { unique: true })
db.itinerary.createIndex({ "type": 1, "bookingStatus": 1 })

// Expenses Collection
db.expenses.createIndex({ "tripId": 1, "expenseDate": -1 })
db.expenses.createIndex({ "expenseId": 1 }, { unique: true })
db.expenses.createIndex({ "paidBy": 1, "expenseDate": -1 })
db.expenses.createIndex({ "category": 1 })
```

## Core Domain Models

### Trip Entity
```csharp
public record Trip(
    Guid Id,
    Guid FamilyId,
    string Title,
    string? Description,
    Destination Destination,
    DateOnly StartDate,
    DateOnly EndDate,
    TripStatus Status,
    TripBudget? Budget,
    List<TripParticipant> Participants,
    Guid CreatedBy,
    DateTime CreatedAt,
    DateTime UpdatedAt
);

public record Destination(
    string City,
    string Country,
    Coordinates? Coordinates,
    string? Timezone
);

public record Coordinates(double Latitude, double Longitude);

public record TripBudget(
    decimal TotalBudget,
    string Currency,
    Dictionary<string, decimal>? CategoryBudgets
);

public record TripParticipant(
    Guid UserId,
    ParticipantRole Role,
    DateTime JoinedAt
);

public enum TripStatus
{
    Planning,
    Confirmed,
    InProgress,
    Completed,
    Cancelled
}

public enum ParticipantRole
{
    Organizer,
    Participant
}
```

### Itinerary Entity
```csharp
public record ItineraryItem(
    Guid Id,
    Guid TripId,
    string Title,
    string? Description,
    ItineraryItemType Type,
    DateTime StartTime,
    DateTime? EndTime,
    Location? Location,
    decimal? EstimatedCost,
    BookingStatus BookingStatus,
    string? BookingReference,
    string? Notes,
    DateTime CreatedAt,
    DateTime UpdatedAt
);

public record Location(
    string Name,
    string? Address,
    Coordinates? Coordinates,
    string? PlaceId
);

public enum ItineraryItemType
{
    Accommodation,
    Transportation,
    Activity,
    Meal,
    Other
}

public enum BookingStatus
{
    NotBooked,
    Pending,
    Confirmed,
    Cancelled
}
```

### Expense Entity
```csharp
public record TripExpense(
    Guid Id,
    Guid TripId,
    decimal Amount,
    string Currency,
    ExpenseCategory Category,
    string Description,
    Guid PaidBy,
    DateOnly ExpenseDate,
    Receipt? Receipt,
    DateTime CreatedAt,
    DateTime UpdatedAt
);

public record Receipt(
    string FileName,
    string FileUrl,
    DateTime UploadedAt
);

public enum ExpenseCategory
{
    Accommodation,
    Transportation,
    Food,
    Activities,
    Shopping,
    Other
}
```

## Service Implementation

### Trip Management Service
```csharp
public interface ITripService
{
    Task<PagedResult<Trip>> GetTripsAsync(Guid familyId, TripFilterOptions? filters = null, PaginationOptions? pagination = null);
    Task<Trip?> GetTripByIdAsync(Guid tripId, Guid familyId);
    Task<Trip> CreateTripAsync(CreateTripRequest request, Guid familyId, Guid userId);
    Task<Trip> UpdateTripAsync(Guid tripId, UpdateTripRequest request, Guid familyId, Guid userId);
    Task<bool> DeleteTripAsync(Guid tripId, Guid familyId, Guid userId);
    Task<bool> AddParticipantAsync(Guid tripId, Guid userId, Guid familyId);
    Task<bool> RemoveParticipantAsync(Guid tripId, Guid userId, Guid familyId);
}

public class TripService : ITripService
{
    private readonly ITripRepository _tripRepository;
    private readonly IAITravelSuggestionService _aiService;
    private readonly IEventBus _eventBus;
    private readonly ILogger<TripService> _logger;

    public TripService(
        ITripRepository tripRepository,
        IAITravelSuggestionService aiService,
        IEventBus eventBus,
        ILogger<TripService> logger)
    {
        _tripRepository = tripRepository;
        _aiService = aiService;
        _eventBus = eventBus;
        _logger = logger;
    }

    public async Task<Trip> CreateTripAsync(CreateTripRequest request, Guid familyId, Guid userId)
    {
        var trip = new Trip(
            Id: Guid.NewGuid(),
            FamilyId: familyId,
            Title: request.Title,
            Description: request.Description,
            Destination: request.Destination,
            StartDate: request.StartDate,
            EndDate: request.EndDate,
            Status: TripStatus.Planning,
            Budget: request.Budget,
            Participants: [new TripParticipant(userId, ParticipantRole.Organizer, DateTime.UtcNow)],
            CreatedBy: userId,
            CreatedAt: DateTime.UtcNow,
            UpdatedAt: DateTime.UtcNow
        );

        await _tripRepository.CreateAsync(trip);
        
        // Publish event for AI suggestions generation
        await _eventBus.PublishAsync(new TripCreatedEvent(trip.Id, familyId, trip.Destination));
        
        _logger.LogInformation("Trip {TripId} created for family {FamilyId}", trip.Id, familyId);
        
        return trip;
    }

    // Additional methods...
}
```

### Itinerary Service
```csharp
public interface IItineraryService
{
    Task<List<ItineraryItem>> GetItineraryAsync(Guid tripId);
    Task<ItineraryItem> AddItineraryItemAsync(Guid tripId, CreateItineraryItemRequest request);
    Task<ItineraryItem> UpdateItineraryItemAsync(Guid itemId, UpdateItineraryItemRequest request);
    Task<bool> DeleteItineraryItemAsync(Guid itemId);
    Task<List<ItineraryItem>> GenerateAISuggestedItineraryAsync(Guid tripId);
}

public class ItineraryService : IItineraryService
{
    private readonly IItineraryRepository _repository;
    private readonly ITripService _tripService;
    private readonly IAITravelSuggestionService _aiService;
    private readonly IGoogleMapsService _mapsService;
    private readonly ICurrentUserService _currentUserService;

    public ItineraryService(
        IItineraryRepository repository,
        ITripService tripService,
        IAITravelSuggestionService aiService,
        IGoogleMapsService mapsService,
        ICurrentUserService currentUserService)
    {
        _repository = repository;
        _tripService = tripService;
        _aiService = aiService;
        _mapsService = mapsService;
        _currentUserService = currentUserService;
    }

    public async Task<ItineraryItem> AddItineraryItemAsync(Guid tripId, CreateItineraryItemRequest request)
    {
        // Validate trip exists and user has access
        var familyId = await _currentUserService.GetFamilyIdAsync();
        var trip = await _tripService.GetTripByIdAsync(tripId, familyId);
        if (trip == null)
            throw new TripNotFoundException(tripId);

        // Enrich location data with Google Maps
        var enrichedLocation = await _mapsService.EnrichLocationAsync(request.Location);

        var item = new ItineraryItem(
            Id: Guid.NewGuid(),
            TripId: tripId,
            Title: request.Title,
            Description: request.Description,
            Type: request.Type,
            StartTime: request.StartTime,
            EndTime: request.EndTime,
            Location: enrichedLocation,
            EstimatedCost: request.EstimatedCost,
            BookingStatus: BookingStatus.NotBooked,
            BookingReference: null,
            Notes: request.Notes,
            CreatedAt: DateTime.UtcNow,
            UpdatedAt: DateTime.UtcNow
        );

        await _repository.CreateAsync(item);
        return item;
    }
}
```

### AI Travel Suggestion Service
```csharp
public interface IAITravelSuggestionService
{
    Task<List<AISuggestion>> GetTravelSuggestionsAsync(Guid tripId, SuggestionType type);
    Task<List<ItineraryItem>> GenerateItinerarySuggestionsAsync(Trip trip);
    Task<BudgetOptimizationSuggestion> OptimizeBudgetAsync(Trip trip, List<TripExpense> expenses);
}

public class AITravelSuggestionService : IAITravelSuggestionService
{
    private readonly IAmazonBedrockRuntime _bedrockClient;
    private readonly ITripRepository _tripRepository;
    private readonly IGoogleMapsService _mapsService;

    public AITravelSuggestionService(
        IAmazonBedrockRuntime bedrockClient,
        ITripRepository tripRepository,
        IGoogleMapsService mapsService)
    {
        _bedrockClient = bedrockClient;
        _tripRepository = tripRepository;
        _mapsService = mapsService;
    }

    public async Task<List<AISuggestion>> GetTravelSuggestionsAsync(Guid tripId, SuggestionType type)
    {
        // Note: This would typically use a repository directly for internal calls
        // or have an internal overload that bypasses family validation
        var trip = await _tripRepository.GetByIdAsync(tripId);
        if (trip == null) return [];

        var prompt = type switch
        {
            SuggestionType.Activities => BuildActivitiesPrompt(trip),
            SuggestionType.Restaurants => BuildRestaurantsPrompt(trip),
            SuggestionType.Accommodations => BuildAccommodationsPrompt(trip),
            SuggestionType.Transportation => BuildTransportationPrompt(trip),
            _ => throw new ArgumentException("Invalid suggestion type")
        };

        var response = await _bedrockClient.InvokeModelAsync(new InvokeModelRequest
        {
            ModelId = "anthropic.claude-3-sonnet-20240229-v1:0",
            Body = new MemoryStream(JsonSerializer.SerializeToUtf8Bytes(new
            {
                anthropic_version = "bedrock-2023-05-31",
                max_tokens = 2000,
                messages = new[]
                {
                    new
                    {
                        role = "user",
                        content = new[]
                        {
                            new { type = "text", text = prompt }
                        }
                    }
                }
            }))
        });

        // Parse response and convert to AISuggestion objects
        return await ParseAISuggestionsAsync(response, type);
    }

    private string BuildActivitiesPrompt(Trip trip)
    {
        return $@"
As a travel expert, suggest 5-7 activities for a family trip to {trip.Destination.City}, {trip.Destination.Country}.

Trip Details:
- Destination: {trip.Destination.City}, {trip.Destination.Country}
- Dates: {trip.StartDate} to {trip.EndDate}
- Duration: {(trip.EndDate.ToDateTime(TimeOnly.MinValue) - trip.StartDate.ToDateTime(TimeOnly.MinValue)).Days} days
- Budget: {trip.Budget?.TotalBudget} {trip.Budget?.Currency}
- Group: Family trip

For each activity, provide:
1. Activity name
2. Brief description (50-100 words)
3. Estimated cost per person
4. Best time to visit
5. Why it's great for families
6. Location/address if specific

Format as JSON array with objects containing: title, description, estimatedCost, location, rating, metadata.
";
    }
}
```

## API Endpoints Implementation

### Trip Management Controller
```csharp
[ApiController]
[Route("api/trips")]
[Authorize]
public class TripsController : ControllerBase
{
    private readonly ITripService _tripService;
    private readonly ICurrentUserService _currentUser;

    [HttpGet]
    public async Task<ActionResult<PagedResult<TripDto>>> GetTrips(
        [FromQuery] TripStatus? status,
        [FromQuery] DateOnly? dateFrom,
        [FromQuery] DateOnly? dateTo,
        [FromQuery] int page = 1,
        [FromQuery] int limit = 20)
    {
        var familyId = await _currentUser.GetFamilyIdAsync();
        var filters = new TripFilterOptions(status, dateFrom, dateTo);
        var pagination = new PaginationOptions(page, limit);
        
        var trips = await _tripService.GetTripsAsync(familyId, filters, pagination);
        return Ok(trips.ToDto());
    }

    [HttpPost]
    public async Task<ActionResult<TripDto>> CreateTrip([FromBody] CreateTripRequest request)
    {
        var familyId = await _currentUser.GetFamilyIdAsync();
        var userId = _currentUser.GetUserId();
        
        var trip = await _tripService.CreateTripAsync(request, familyId, userId);
        return CreatedAtAction(nameof(GetTrip), new { tripId = trip.Id }, trip.ToDto());
    }

    [HttpGet("{tripId:guid}")]
    public async Task<ActionResult<TripDto>> GetTrip(Guid tripId)
    {
        var familyId = await _currentUser.GetFamilyIdAsync();
        var trip = await _tripService.GetTripByIdAsync(tripId, familyId);
        
        if (trip == null)
            return NotFound();
            
        return Ok(trip.ToDto());
    }
}
```

### Minimal API Alternative
```csharp
public static class TripsEndpoints
{
    public static void MapTripsEndpoints(this IEndpointRouteBuilder app)
    {
        var group = app.MapGroup("/api/trips")
            .WithTags("Trips")
            .RequireAuthorization();

        group.MapGet("/", GetTrips)
            .WithName("GetTrips")
            .WithSummary("Get trips for the current family")
            .WithOpenApi();

        group.MapPost("/", CreateTrip)
            .WithName("CreateTrip")
            .WithSummary("Create a new trip")
            .WithOpenApi();

        group.MapGet("/{tripId:guid}", GetTrip)
            .WithName("GetTrip")
            .WithSummary("Get trip by ID")
            .WithOpenApi();

        group.MapPut("/{tripId:guid}", UpdateTrip)
            .WithName("UpdateTrip")
            .WithSummary("Update a trip")
            .WithOpenApi();

        group.MapDelete("/{tripId:guid}", DeleteTrip)
            .WithName("DeleteTrip")
            .WithSummary("Delete a trip")
            .WithOpenApi();
    }

    private static async Task<IResult> GetTrips(
        ITripService tripService,
        ICurrentUserService currentUser,
        TripStatus? status = null,
        DateOnly? dateFrom = null,
        DateOnly? dateTo = null,
        int page = 1,
        int limit = 20)
    {
        var familyId = await currentUser.GetFamilyIdAsync();
        var filters = new TripFilterOptions(status, dateFrom, dateTo);
        var pagination = new PaginationOptions(page, limit);
        
        var trips = await tripService.GetTripsAsync(familyId, filters, pagination);
        return Results.Ok(trips.ToDto());
    }

    private static async Task<IResult> CreateTrip(
        CreateTripRequest request,
        ITripService tripService,
        ICurrentUserService currentUser)
    {
        var familyId = await currentUser.GetFamilyIdAsync();
        var userId = currentUser.GetUserId();
        
        var trip = await tripService.CreateTripAsync(request, familyId, userId);
        return Results.Created($"/api/trips/{trip.Id}", trip.ToDto());
    }
}
```

## Data Access Layer

### Repository Pattern
```csharp
public interface ITripRepository
{
    Task<PagedResult<Trip>> GetTripsAsync(Guid familyId, TripFilterOptions? filters, PaginationOptions? pagination);
    Task<Trip?> GetByIdAsync(Guid tripId);
    Task<Trip> CreateAsync(Trip trip);
    Task<Trip> UpdateAsync(Trip trip);
    Task<bool> DeleteAsync(Guid tripId);
}

public class MongoTripRepository : ITripRepository
{
    private readonly IMongoCollection<Trip> _collection;
    private readonly ILogger<MongoTripRepository> _logger;

    public MongoTripRepository(IMongoDatabase database, ILogger<MongoTripRepository> logger)
    {
        _collection = database.GetCollection<Trip>("trips");
        _logger = logger;
    }

    public async Task<PagedResult<Trip>> GetTripsAsync(
        Guid familyId, 
        TripFilterOptions? filters, 
        PaginationOptions? pagination)
    {
        var filterBuilder = Builders<Trip>.Filter;
        var filter = filterBuilder.Eq(t => t.FamilyId, familyId);

        if (filters?.Status.HasValue == true)
            filter &= filterBuilder.Eq(t => t.Status, filters.Status.Value);

        if (filters?.DateFrom.HasValue == true)
            filter &= filterBuilder.Gte(t => t.StartDate, filters.DateFrom.Value);

        if (filters?.DateTo.HasValue == true)
            filter &= filterBuilder.Lte(t => t.EndDate, filters.DateTo.Value);

        var totalCount = await _collection.CountDocumentsAsync(filter);
        
        var trips = await _collection
            .Find(filter)
            .Sort(Builders<Trip>.Sort.Descending(t => t.StartDate))
            .Skip((pagination?.Page - 1 ?? 0) * (pagination?.Limit ?? 20))
            .Limit(pagination?.Limit ?? 20)
            .ToListAsync();

        return new PagedResult<Trip>(
            trips,
            totalCount,
            pagination?.Page ?? 1,
            pagination?.Limit ?? 20
        );
    }

    public async Task<Trip> CreateAsync(Trip trip)
    {
        await _collection.InsertOneAsync(trip);
        _logger.LogInformation("Created trip {TripId} for family {FamilyId}", trip.Id, trip.FamilyId);
        return trip;
    }
}
```

## Configuration

### Program.cs Setup
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Authentication & Authorization
builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer("Bearer", options =>
    {
        options.Authority = builder.Configuration["Authentication:Authority"];
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateAudience = false
        };
    });

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("FamilyMember", policy =>
        policy.Requirements.Add(new FamilyMemberRequirement()));
});

builder.Services.AddScoped<IAuthorizationHandler, FamilyMemberHandler>();

// MongoDB Configuration
builder.Services.Configure<MongoDbSettings>(
    builder.Configuration.GetSection("MongoDb"));

builder.Services.AddSingleton<IMongoClient>(serviceProvider =>
{
    var settings = serviceProvider.GetRequiredService<IOptions<MongoDbSettings>>().Value;
    return new MongoClient(settings.ConnectionString);
});

builder.Services.AddScoped<IMongoDatabase>(serviceProvider =>
{
    var client = serviceProvider.GetRequiredService<IMongoClient>();
    var settings = serviceProvider.GetRequiredService<IOptions<MongoDbSettings>>().Value;
    return client.GetDatabase(settings.DatabaseName);
});

// AWS Services
builder.Services.AddAWSService<IAmazonBedrockRuntime>();
builder.Services.AddAWSService<IAmazonS3>();

// Application Services
builder.Services.AddScoped<ITripService, TripService>();
builder.Services.AddScoped<IItineraryService, ItineraryService>();
builder.Services.AddScoped<IExpenseService, ExpenseService>();
builder.Services.AddScoped<IAITravelSuggestionService, AITravelSuggestionService>();

// Repositories
builder.Services.AddScoped<ITripRepository, MongoTripRepository>();
builder.Services.AddScoped<IItineraryRepository, MongoItineraryRepository>();
builder.Services.AddScoped<IExpenseRepository, MongoExpenseRepository>();

// External Services
builder.Services.AddHttpClient<IGoogleMapsService, GoogleMapsService>();

var app = builder.Build();

// Configure pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();

// Use either controllers OR minimal APIs, not both for same routes
// app.MapControllers(); // Commented out to avoid duplicate mapping for /api/trips
app.MapTripsEndpoints();

app.Run();
```

### Configuration Settings
```json
{
  "MongoDb": {
    "ConnectionString": "mongodb://localhost:27017",
    "DatabaseName": "homemanager-trips"
  },
  "AWS": {
    "Region": "us-east-1",
    "Bedrock": {
      "ModelId": "anthropic.claude-3-sonnet-20240229-v1:0"
    },
    "S3": {
      "BucketName": "homemanager-receipts"
    }
  },
  "GoogleMaps": {
    "ApiKey": "your-google-maps-api-key"
  }
}
```

## Testing

### Unit Tests
```csharp
public class TripServiceTests
{
    private readonly Mock<ITripRepository> _mockRepository;
    private readonly Mock<IAITravelSuggestionService> _mockAIService;
    private readonly Mock<IEventBus> _mockEventBus;
    private readonly Mock<ILogger<TripService>> _mockLogger;
    private readonly TripService _service;

    public TripServiceTests()
    {
        _mockRepository = new Mock<ITripRepository>();
        _mockAIService = new Mock<IAITravelSuggestionService>();
        _mockEventBus = new Mock<IEventBus>();
        _mockLogger = new Mock<ILogger<TripService>>();
        
        _service = new TripService(
            _mockRepository.Object,
            _mockAIService.Object,
            _mockEventBus.Object,
            _mockLogger.Object);
    }

    [Fact]
    public async Task CreateTripAsync_ValidRequest_ReturnsTrip()
    {
        // Arrange
        var familyId = Guid.NewGuid();
        var userId = Guid.NewGuid();
        var request = new CreateTripRequest(
            "Paris Vacation",
            "Family trip to Paris",
            new Destination("Paris", "France", null, null),
            DateOnly.FromDateTime(DateTime.Today.AddDays(30)),
            DateOnly.FromDateTime(DateTime.Today.AddDays(37)),
            null,
            []
        );

        _mockRepository.Setup(r => r.CreateAsync(It.IsAny<Trip>()))
            .Returns<Trip>(trip => Task.FromResult(trip));

        // Act
        var result = await _service.CreateTripAsync(request, familyId, userId);

        // Assert
        Assert.NotNull(result);
        Assert.Equal(request.Title, result.Title);
        Assert.Equal(familyId, result.FamilyId);
        Assert.Equal(userId, result.CreatedBy);
        
        _mockRepository.Verify(r => r.CreateAsync(It.IsAny<Trip>()), Times.Once);
        _mockEventBus.Verify(e => e.PublishAsync(It.IsAny<TripCreatedEvent>()), Times.Once);
    }
}
```

### Integration Tests
```csharp
[Collection("MongoDB")]
public class TripRepositoryIntegrationTests : IAsyncLifetime
{
    private readonly MongoDbFixture _mongoFixture;
    private readonly ITripRepository _repository;

    public TripRepositoryIntegrationTests(MongoDbFixture mongoFixture)
    {
        _mongoFixture = mongoFixture;
        _repository = new MongoTripRepository(
            mongoFixture.Database,
            Mock.Of<ILogger<MongoTripRepository>>());
    }

    [Fact]
    public async Task GetTripsAsync_WithFilters_ReturnsFilteredTrips()
    {
        // Arrange
        var familyId = Guid.NewGuid();
        var trips = new[]
        {
            CreateTestTrip(familyId, TripStatus.Planning, DateOnly.FromDateTime(DateTime.Today.AddDays(10))),
            CreateTestTrip(familyId, TripStatus.Confirmed, DateOnly.FromDateTime(DateTime.Today.AddDays(20))),
            CreateTestTrip(familyId, TripStatus.Planning, DateOnly.FromDateTime(DateTime.Today.AddDays(30)))
        };

        foreach (var trip in trips)
        {
            await _repository.CreateAsync(trip);
        }

        var filters = new TripFilterOptions(TripStatus.Planning, null, null);

        // Act
        var result = await _repository.GetTripsAsync(familyId, filters, null);

        // Assert
        Assert.Equal(2, result.Items.Count);
        Assert.All(result.Items, trip => Assert.Equal(TripStatus.Planning, trip.Status));
    }

    public async Task InitializeAsync()
    {
        await _mongoFixture.ResetDatabaseAsync();
    }

    public Task DisposeAsync() => Task.CompletedTask;

    private static Trip CreateTestTrip(Guid familyId, TripStatus status, DateOnly startDate)
    {
        return new Trip(
            Guid.NewGuid(),
            familyId,
            "Test Trip",
            null,
            new Destination("Test City", "Test Country", null, null),
            startDate,
            startDate.AddDays(7),
            status,
            null,
            [],
            Guid.NewGuid(),
            DateTime.UtcNow,
            DateTime.UtcNow
        );
    }
}
```

## Security Considerations

### Authentication & Authorization
```csharp
[Authorize(Policy = "FamilyMember")]
public class TripsController : ControllerBase
{
    // Trip operations require family membership
}

public class FamilyMemberRequirement : IAuthorizationRequirement { }

public class FamilyMemberHandler : AuthorizationHandler<FamilyMemberRequirement>
{
    private readonly ICurrentUserService _currentUser;
    private readonly IFamilyService _familyService;

    public FamilyMemberHandler(ICurrentUserService currentUser, IFamilyService familyService)
    {
        _currentUser = currentUser;
        _familyService = familyService;
    }

    protected override async Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        FamilyMemberRequirement requirement)
    {
        var userId = _currentUser.GetUserId();
        var familyId = await _currentUser.GetFamilyIdAsync();
        
        if (await _familyService.IsFamilyMemberAsync(userId, familyId))
        {
            context.Succeed(requirement);
        }
    }
}
```

### Data Privacy
- Encrypt sensitive trip data at rest
- Use HTTPS for all communications
- Implement data retention policies
- Support GDPR data export/deletion

## Deployment

### Docker Configuration
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY ["HomeManager.TripsPlanner.csproj", "."]
RUN dotnet restore "./HomeManager.TripsPlanner.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "HomeManager.TripsPlanner.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "HomeManager.TripsPlanner.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "HomeManager.TripsPlanner.dll"]
```

### Kubernetes Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trips-planner-service
  namespace: homemanager
spec:
  replicas: 2
  selector:
    matchLabels:
      app: trips-planner-service
  template:
    metadata:
      labels:
        app: trips-planner-service
    spec:
      containers:
      - name: trips-planner
        image: homemanager/trips-planner:latest
        ports:
        - containerPort: 80
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: "Production"
        - name: MongoDb__ConnectionString
          valueFrom:
            secretKeyRef:
              name: mongodb-connection
              key: connection-string
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

## Monitoring & Observability

### Health Checks
```csharp
builder.Services.AddHealthChecks()
    .AddMongoDb(
        builder.Configuration["MongoDb:ConnectionString"],
        name: "mongodb",
        timeout: TimeSpan.FromSeconds(5))
    // Add custom health checks for external services
    .AddUrlGroup(
        new Uri("https://maps.googleapis.com/maps/api"),
        name: "googlemaps",
        timeout: TimeSpan.FromSeconds(5));

app.MapHealthChecks("/health");
app.MapHealthChecks("/health/ready");
```

### Metrics & Logging
```csharp
public class TripService : ITripService
{
    private static readonly Meter _meter = new("HomeManager.TripsPlanner");
    private static readonly Counter<int> TripsCreated = 
        _meter.CreateCounter<int>("trips.created", "trips", "Number of trips created");
    
    private static readonly Histogram<double> TripCreationDuration = 
        _meter.CreateHistogram<double>("trips.creation.duration", "ms", "Trip creation duration");

    private static readonly ActivitySource ActivitySource = new("HomeManager.TripsPlanner");

    private readonly ITripRepository _tripRepository;
    private readonly IEventBus _eventBus;
    private readonly ILogger<TripService> _logger;

    public TripService(
        ITripRepository tripRepository,
        IEventBus eventBus,
        ILogger<TripService> logger)
    {
        _tripRepository = tripRepository;
        _eventBus = eventBus;
        _logger = logger;
    }

    public async Task<Trip> CreateTripAsync(CreateTripRequest request, Guid familyId, Guid userId)
    {
        using var activity = ActivitySource.StartActivity("TripService.CreateTrip");
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            var trip = new Trip(
                Id: Guid.NewGuid(),
                FamilyId: familyId,
                Title: request.Title,
                Description: request.Description,
                Destination: request.Destination,
                StartDate: request.StartDate,
                EndDate: request.EndDate,
                Status: TripStatus.Planning,
                Budget: request.Budget,
                Participants: [new TripParticipant(userId, ParticipantRole.Organizer, DateTime.UtcNow)],
                CreatedBy: userId,
                CreatedAt: DateTime.UtcNow,
                UpdatedAt: DateTime.UtcNow
            );

            await _tripRepository.CreateAsync(trip);
            
            // Publish event for AI suggestions generation
            await _eventBus.PublishAsync(new TripCreatedEvent(trip.Id, familyId, trip.Destination));
            
            _logger.LogInformation("Trip {TripId} created for family {FamilyId}", trip.Id, familyId);
            
            TripsCreated.Add(1, new("status", "success"));
            TripCreationDuration.Record(stopwatch.ElapsedMilliseconds);
            return trip;
        }
        catch (Exception ex)
        {
            TripsCreated.Add(1, new("status", "failure"));
            TripCreationDuration.Record(stopwatch.ElapsedMilliseconds);
            _logger.LogError(ex, "Failed to create trip for family {FamilyId}", familyId);
            throw;
        }
    }
}
```

This comprehensive implementation guide provides everything needed to build a robust trips planner service for the HomeManager application, including AI-powered suggestions, expense tracking, and itinerary management.