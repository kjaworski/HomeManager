# Identity Service Implementation Guide

> **AI Development Context** 🤖  
> This guide provides comprehensive implementation details for AI-assisted development

## Overview

The Identity Service handles user authentication, authorization, and family management for the HomeManager system. It provides JWT-based authentication and multi-tenant family isolation.

## Architecture

### Technology Stack
```yaml
Framework: ASP.NET Core 9 with Minimal APIs
Database: PostgreSQL 15+
Authentication: JWT with refresh tokens
ORM: Entity Framework Core 9
Validation: FluentValidation
Testing: xUnit with Testcontainers
Documentation: Swagger/OpenAPI 3.0
```

### Database Schema
```sql
-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    display_name VARCHAR(150),
    profile_picture_url TEXT,
    timezone VARCHAR(50) DEFAULT 'UTC',
    language VARCHAR(10) DEFAULT 'en-US',
    email_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Families table
CREATE TABLE families (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(200) NOT NULL,
    description TEXT,
    admin_id UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Family members junction table
CREATE TABLE family_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    family_id UUID NOT NULL REFERENCES families(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role VARCHAR(20) NOT NULL DEFAULT 'member',
    joined_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(family_id, user_id)
);

-- Refresh tokens table
CREATE TABLE refresh_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token_hash VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    revoked_at TIMESTAMP WITH TIME ZONE
);
```

## Implementation

### 1. Entity Models
```csharp
// User.cs
namespace HomeManager.Identity.Domain.Entities;

public class User
{
    public Guid Id { get; set; }
    public string Email { get; set; } = null!;
    public string PasswordHash { get; set; } = null!;
    public string FirstName { get; set; } = null!;
    public string LastName { get; set; } = null!;
    public string? DisplayName { get; set; }
    public string? ProfilePictureUrl { get; set; }
    public string Timezone { get; set; } = "UTC";
    public string Language { get; set; } = "en-US";
    public bool EmailVerified { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
    
    // Navigation properties
    public ICollection<FamilyMember> FamilyMemberships { get; set; } = new List<FamilyMember>();
    public ICollection<Family> AdministratedFamilies { get; set; } = new List<Family>();
    public ICollection<RefreshToken> RefreshTokens { get; set; } = new List<RefreshToken>();
}

// Family.cs
public class Family
{
    public Guid Id { get; set; }
    public string Name { get; set; } = null!;
    public string? Description { get; set; }
    public Guid AdminId { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
    
    // Navigation properties
    public User Admin { get; set; } = null!;
    public ICollection<FamilyMember> Members { get; set; } = new List<FamilyMember>();
}

// FamilyMember.cs
public class FamilyMember
{
    public Guid Id { get; set; }
    public Guid FamilyId { get; set; }
    public Guid UserId { get; set; }
    public FamilyRole Role { get; set; } = FamilyRole.Member;
    public DateTime JoinedAt { get; set; }
    
    // Navigation properties
    public Family Family { get; set; } = null!;
    public User User { get; set; } = null!;
}

public enum FamilyRole
{
    Member,
    Admin
}
```

### 2. Database Context
```csharp
// IdentityDbContext.cs
namespace HomeManager.Identity.Infrastructure.Data;

public class IdentityDbContext : DbContext
{
    public IdentityDbContext(DbContextOptions<IdentityDbContext> options) : base(options)
    {
    }
    
    public DbSet<User> Users { get; set; }
    public DbSet<Family> Families { get; set; }
    public DbSet<FamilyMember> FamilyMembers { get; set; }
    public DbSet<RefreshToken> RefreshTokens { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // User configuration
        modelBuilder.Entity<User>(entity =>
        {
            entity.ToTable("users");
            entity.HasKey(e => e.Id);
            entity.HasIndex(e => e.Email).IsUnique();
            entity.Property(e => e.Email).HasMaxLength(255).IsRequired();
            entity.Property(e => e.FirstName).HasMaxLength(100).IsRequired();
            entity.Property(e => e.LastName).HasMaxLength(100).IsRequired();
            entity.Property(e => e.DisplayName).HasMaxLength(150);
            entity.Property(e => e.Timezone).HasMaxLength(50).HasDefaultValue("UTC");
            entity.Property(e => e.Language).HasMaxLength(10).HasDefaultValue("en-US");
            entity.Property(e => e.CreatedAt).HasDefaultValueSql("NOW()");
            entity.Property(e => e.UpdatedAt).HasDefaultValueSql("NOW()");
        });
        
        // Family configuration
        modelBuilder.Entity<Family>(entity =>
        {
            entity.ToTable("families");
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).HasMaxLength(200).IsRequired();
            entity.Property(e => e.CreatedAt).HasDefaultValueSql("NOW()");
            entity.Property(e => e.UpdatedAt).HasDefaultValueSql("NOW()");
            
            entity.HasOne(e => e.Admin)
                  .WithMany(u => u.AdministratedFamilies)
                  .HasForeignKey(e => e.AdminId)
                  .OnDelete(DeleteBehavior.Restrict);
        });
        
        // FamilyMember configuration
        modelBuilder.Entity<FamilyMember>(entity =>
        {
            entity.ToTable("family_members");
            entity.HasKey(e => e.Id);
            entity.HasIndex(e => new { e.FamilyId, e.UserId }).IsUnique();
            entity.Property(e => e.Role).HasConversion<string>();
            entity.Property(e => e.JoinedAt).HasDefaultValueSql("NOW()");
            
            entity.HasOne(e => e.Family)
                  .WithMany(f => f.Members)
                  .HasForeignKey(e => e.FamilyId)
                  .OnDelete(DeleteBehavior.Cascade);
                  
            entity.HasOne(e => e.User)
                  .WithMany(u => u.FamilyMemberships)
                  .HasForeignKey(e => e.UserId)
                  .OnDelete(DeleteBehavior.Cascade);
        });
        
        // RefreshToken configuration
        modelBuilder.Entity<RefreshToken>(entity =>
        {
            entity.ToTable("refresh_tokens");
            entity.HasKey(e => e.Id);
            entity.Property(e => e.TokenHash).HasMaxLength(255).IsRequired();
            entity.Property(e => e.CreatedAt).HasDefaultValueSql("NOW()");
            
            entity.HasOne(e => e.User)
                  .WithMany(u => u.RefreshTokens)
                  .HasForeignKey(e => e.UserId)
                  .OnDelete(DeleteBehavior.Cascade);
        });
    }
}
```

### 3. Services Implementation
```csharp
// IUserService.cs
namespace HomeManager.Identity.Application.Services;

public interface IUserService
{
    Task<UserRegistrationResult> RegisterAsync(UserRegistrationRequest request, CancellationToken cancellationToken = default);
    Task<LoginResult> LoginAsync(LoginRequest request, CancellationToken cancellationToken = default);
    Task<TokenResult> RefreshTokenAsync(RefreshTokenRequest request, CancellationToken cancellationToken = default);
    Task<UserProfile> GetProfileAsync(Guid userId, CancellationToken cancellationToken = default);
    Task<UserProfile> UpdateProfileAsync(Guid userId, UpdateProfileRequest request, CancellationToken cancellationToken = default);
}

// UserService.cs
public class UserService : IUserService
{
    private readonly IdentityDbContext _context;
    private readonly IPasswordHasher _passwordHasher;
    private readonly IJwtTokenGenerator _tokenGenerator;
    private readonly ILogger<UserService> _logger;
    
    public UserService(
        IdentityDbContext context,
        IPasswordHasher passwordHasher,
        IJwtTokenGenerator tokenGenerator,
        ILogger<UserService> logger)
    {
        _context = context;
        _passwordHasher = passwordHasher;
        _tokenGenerator = tokenGenerator;
        _logger = logger;
    }
    
    public async Task<UserRegistrationResult> RegisterAsync(
        UserRegistrationRequest request,
        CancellationToken cancellationToken = default)
    {
        _logger.LogInformation("Registering new user with email: {Email}", request.Email);
        
        // Check if user already exists
        var existingUser = await _context.Users
            .FirstOrDefaultAsync(u => u.Email == request.Email, cancellationToken);
            
        if (existingUser != null)
        {
            throw new UserAlreadyExistsException($"User with email {request.Email} already exists");
        }
        
        // Create new user
        var user = new User
        {
            Id = Guid.NewGuid(),
            Email = request.Email.ToLowerInvariant(),
            PasswordHash = _passwordHasher.HashPassword(request.Password),
            FirstName = request.FirstName,
            LastName = request.LastName,
            DisplayName = request.DisplayName ?? $"{request.FirstName} {request.LastName[0]}.",
            CreatedAt = DateTime.UtcNow,
            UpdatedAt = DateTime.UtcNow
        };
        
        _context.Users.Add(user);
        await _context.SaveChangesAsync(cancellationToken);
        
        _logger.LogInformation("Successfully registered user {UserId}", user.Id);
        
        return new UserRegistrationResult
        {
            UserId = user.Id,
            Email = user.Email,
            Success = true
        };
    }
    
    public async Task<LoginResult> LoginAsync(
        LoginRequest request,
        CancellationToken cancellationToken = default)
    {
        _logger.LogInformation("User login attempt for email: {Email}", request.Email);
        
        var user = await _context.Users
            .Include(u => u.FamilyMemberships)
            .ThenInclude(fm => fm.Family)
            .FirstOrDefaultAsync(u => u.Email == request.Email.ToLowerInvariant(), cancellationToken);
            
        if (user == null || !_passwordHasher.VerifyPassword(request.Password, user.PasswordHash))
        {
            _logger.LogWarning("Failed login attempt for email: {Email}", request.Email);
            throw new InvalidCredentialsException("Invalid email or password");
        }
        
        // Generate tokens
        var accessToken = _tokenGenerator.GenerateAccessToken(user);
        var refreshToken = _tokenGenerator.GenerateRefreshToken();
        
        // Store refresh token
        var refreshTokenEntity = new RefreshToken
        {
            Id = Guid.NewGuid(),
            UserId = user.Id,
            TokenHash = _passwordHasher.HashPassword(refreshToken),
            ExpiresAt = DateTime.UtcNow.AddDays(30),
            CreatedAt = DateTime.UtcNow
        };
        
        _context.RefreshTokens.Add(refreshTokenEntity);
        await _context.SaveChangesAsync(cancellationToken);
        
        _logger.LogInformation("Successful login for user {UserId}", user.Id);
        
        return new LoginResult
        {
            AccessToken = accessToken,
            RefreshToken = refreshToken,
            ExpiresIn = 3600, // 1 hour
            User = MapToUserProfile(user)
        };
    }
    
    // Additional methods implementation...
    
    private static UserProfile MapToUserProfile(User user)
    {
        return new UserProfile
        {
            Id = user.Id,
            Email = user.Email,
            FirstName = user.FirstName,
            LastName = user.LastName,
            DisplayName = user.DisplayName,
            ProfilePictureUrl = user.ProfilePictureUrl,
            Timezone = user.Timezone,
            Language = user.Language,
            CreatedAt = user.CreatedAt,
            UpdatedAt = user.UpdatedAt
        };
    }
}
```

### 4. API Endpoints
```csharp
// Program.cs - API endpoint configuration
var builder = WebApplication.CreateBuilder(args);

// Configure services
builder.Services.AddDbContext<IdentityDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("Identity")));

builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddScoped<IFamilyService, FamilyService>();
builder.Services.AddScoped<IPasswordHasher, BCryptPasswordHasher>();
builder.Services.AddScoped<IJwtTokenGenerator, JwtTokenGenerator>();

var app = builder.Build();

// Configure endpoints
app.MapPost("/auth/register", async (
    UserRegistrationRequest request,
    IUserService userService,
    CancellationToken cancellationToken) =>
{
    try
    {
        var result = await userService.RegisterAsync(request, cancellationToken);
        return Results.Created($"/users/{result.UserId}", result);
    }
    catch (UserAlreadyExistsException ex)
    {
        return Results.Conflict(new ErrorResponse
        {
            Error = "USER_ALREADY_EXISTS",
            Message = ex.Message,
            Timestamp = DateTime.UtcNow
        });
    }
    catch (Exception ex)
    {
        return Results.BadRequest(new ErrorResponse
        {
            Error = "REGISTRATION_FAILED",
            Message = "Registration failed due to invalid data",
            Timestamp = DateTime.UtcNow
        });
    }
})
.WithName("RegisterUser")
.WithTags("Authentication")
.WithOpenApi();

app.MapPost("/auth/login", async (
    LoginRequest request,
    IUserService userService,
    CancellationToken cancellationToken) =>
{
    try
    {
        var result = await userService.LoginAsync(request, cancellationToken);
        return Results.Ok(result);
    }
    catch (InvalidCredentialsException ex)
    {
        return Results.Unauthorized();
    }
})
.WithName("LoginUser")
.WithTags("Authentication")
.WithOpenApi();

// Additional endpoints...

app.Run();
```

## Testing

### Unit Tests
```csharp
// UserServiceTests.cs
namespace HomeManager.Identity.Tests.Unit.Services;

public class UserServiceTests
{
    private readonly Mock<IPasswordHasher> _passwordHasherMock;
    private readonly Mock<IJwtTokenGenerator> _tokenGeneratorMock;
    private readonly Mock<ILogger<UserService>> _loggerMock;
    private readonly IdentityDbContext _context;
    private readonly UserService _userService;
    
    public UserServiceTests()
    {
        var options = new DbContextOptionsBuilder<IdentityDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;
            
        _context = new IdentityDbContext(options);
        _passwordHasherMock = new Mock<IPasswordHasher>();
        _tokenGeneratorMock = new Mock<IJwtTokenGenerator>();
        _loggerMock = new Mock<ILogger<UserService>>();
        
        _userService = new UserService(
            _context,
            _passwordHasherMock.Object,
            _tokenGeneratorMock.Object,
            _loggerMock.Object);
    }
    
    [Fact]
    public async Task RegisterAsync_WithValidRequest_ShouldCreateUser()
    {
        // Arrange
        var request = new UserRegistrationRequest
        {
            Email = "test@example.com",
            Password = "SecurePassword123!",
            FirstName = "John",
            LastName = "Doe"
        };
        
        _passwordHasherMock.Setup(x => x.HashPassword(request.Password))
                          .Returns("hashed_password");
        
        // Act
        var result = await _userService.RegisterAsync(request);
        
        // Assert
        Assert.True(result.Success);
        Assert.Equal(request.Email, result.Email);
        
        var user = await _context.Users.FirstOrDefaultAsync(u => u.Email == request.Email);
        Assert.NotNull(user);
        Assert.Equal(request.FirstName, user.FirstName);
        Assert.Equal(request.LastName, user.LastName);
    }
    
    [Fact]
    public async Task RegisterAsync_WithExistingEmail_ShouldThrowException()
    {
        // Arrange
        var existingUser = new User
        {
            Id = Guid.NewGuid(),
            Email = "existing@example.com",
            PasswordHash = "hash",
            FirstName = "Existing",
            LastName = "User",
            CreatedAt = DateTime.UtcNow,
            UpdatedAt = DateTime.UtcNow
        };
        
        _context.Users.Add(existingUser);
        await _context.SaveChangesAsync();
        
        var request = new UserRegistrationRequest
        {
            Email = "existing@example.com",
            Password = "password",
            FirstName = "Test",
            LastName = "User"
        };
        
        // Act & Assert
        await Assert.ThrowsAsync<UserAlreadyExistsException>(
            () => _userService.RegisterAsync(request));
    }
}
```

### Integration Tests
```csharp
// IdentityServiceIntegrationTests.cs
namespace HomeManager.Identity.Tests.Integration;

public class IdentityServiceIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;
    
    public IdentityServiceIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = _factory.CreateClient();
    }
    
    [Fact]
    public async Task RegisterUser_WithValidData_ShouldReturnCreated()
    {
        // Arrange
        var request = new UserRegistrationRequest
        {
            Email = "integration@test.com",
            Password = "SecurePassword123!",
            FirstName = "Integration",
            LastName = "Test"
        };
        
        var json = JsonSerializer.Serialize(request);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        
        // Act
        var response = await _client.PostAsync("/auth/register", content);
        
        // Assert
        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
        
        var responseContent = await response.Content.ReadAsStringAsync();
        var result = JsonSerializer.Deserialize<UserRegistrationResult>(responseContent);
        
        Assert.NotNull(result);
        Assert.True(result.Success);
        Assert.Equal(request.Email, result.Email);
    }
}
```

## Deployment Configuration

### Docker Configuration
```dockerfile
# Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY ["src/HomeManager.Identity/HomeManager.Identity.csproj", "src/HomeManager.Identity/"]
RUN dotnet restore "src/HomeManager.Identity/HomeManager.Identity.csproj"
COPY . .
WORKDIR "/src/src/HomeManager.Identity"
RUN dotnet build "HomeManager.Identity.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "HomeManager.Identity.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "HomeManager.Identity.dll"]
```

### Kubernetes Deployment
```yaml
# k8s/identity-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: identity-service
  labels:
    app: identity-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: identity-service
  template:
    metadata:
      labels:
        app: identity-service
    spec:
      containers:
      - name: identity-service
        image: homemanager/identity-service:latest
        ports:
        - containerPort: 80
        env:
        - name: ConnectionStrings__Identity
          valueFrom:
            secretKeyRef:
              name: postgres-connection
              key: connection-string
        - name: Jwt__Secret
          valueFrom:
            secretKeyRef:
              name: jwt-secret
              key: secret
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: identity-service
spec:
  selector:
    app: identity-service
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```