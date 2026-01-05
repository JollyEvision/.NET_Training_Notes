# .NET_Training_Notes

# ASP.NET Core Authorization -- Role-Based & Policy-Based

## 1. Authentication vs Authorization

-   **Authentication**: Who the user is (JWT, Cookies, OAuth, Identity)
-   **Authorization**: What the user can do (Roles, Policies, Claims)

------------------------------------------------------------------------

## 2. Role-Based Authorization

### What is it?

Authorization based on **roles** assigned to a user.

### How it works

-   Roles are stored in:
    -   JWT claims (`role`)
    -   ASP.NET Identity roles
-   `[Authorize(Roles = "Admin")]` checks if user has the role

### Example

``` csharp
[Authorize(Roles = "Admin")]
public IActionResult GetAdminsOnly()
{
    return Ok("Admin access");
}
```

### Multiple Roles

``` csharp
[Authorize(Roles = "Admin,Manager")]
```

(User must have **at least one** role)

### Pros

-   Simple
-   Easy to understand
-   Good for small apps

### Cons

-   Hard to scale
-   Not flexible
-   Logic tied to role names

------------------------------------------------------------------------

## 3. Policy-Based Authorization

### What is it?

Authorization based on **rules (policies)** instead of roles.

A policy can check: - Roles - Claims - Custom logic - Requirements +
Handlers

### Define Policy

``` csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));

    options.AddPolicy("AgeAbove18", policy =>
        policy.RequireClaim("Age", "18"));
});
```

### Use Policy

``` csharp
[Authorize(Policy = "AdminOnly")]
public IActionResult SecureEndpoint()
{
    return Ok("Policy access granted");
}
```

------------------------------------------------------------------------

## 4. Policy with Multiple Conditions

``` csharp
options.AddPolicy("AdminWithEmail", policy =>
{
    policy.RequireRole("Admin");
    policy.RequireClaim("EmailVerified", "true");
});
```

(User must satisfy **ALL conditions**)

------------------------------------------------------------------------

## Authentication and Authorization using Identity framework and JWT
Reference Video:  
https://www.youtube.com/watch?v=99-r3Y48SYE

---

## ASP.NET Identity Framework – What & Why

### What is ASP.NET Identity?
A membership system for ASP.NET Core.

Manages:
- Users
- Passwords (secure hashing)
- Roles
- Claims
- Logins / Logouts
- Tokens (reset password, email confirmation)

Works with:
- MVC
- Web API
- Razor Pages

---

### Why use Identity?
Without Identity, you must manually handle:
- Password hashing
- User storage
- Authentication cookies
- Authorization logic
- Security edge cases

Identity provides secure, tested, best-practice implementation.

---

## Core Identity Building Blocks

### 1. IdentityUser
Represents a user.

Has built-in properties like:
- UserName
- Email
- PasswordHash
- LockoutEnabled

You usually extend it:

```csharp
public class ApplicationUser : IdentityUser
{
    public string FullName { get; set; }
}
```

---

### 2. IdentityRole
Represents roles like Admin, User.  
Stored as records in the database.

---

### 4. Managers

| Manager | Responsibility |
|------|---------------|
| UserManager<TUser> | Create users, change passwords, roles |
| SignInManager<TUser> | Login / Logout |
| RoleManager<TRole> | Create & manage roles |

---

### 6. IdentityDbContext
EF Core DbContext.Extend this class to add auth data to your connected database

Creates tables:
- AspNetUsers
- AspNetRoles
- AspNetUserRoles
- AspNetUserClaims
- AspNetUserLogins
- AspNetUserTokens

---

### 7. AddIdentity<ApplicationUser, IdentityRole>

```csharp
.AddIdentity<ApplicationUser, IdentityRole>()
```

It tells ASP.NET Core:
- Use ApplicationUser as user entity
- Use IdentityRole as role entity
- Register all Identity services into DI

It registers:
- UserManager
- SignInManager
- RoleManager
- Password hashing
- Cookie authentication
- Token providers

What it does NOT do:
- Does not configure database
- Does not create tables

That’s why we add:

```csharp
.AddEntityFrameworkStores<AppDbContext>()
```

---
Install Package: Microsoft.AspNetCore.Authentication.JwtBearer


## JWT Token Creation

JWT Flow:
1. Create claims
2. Create security key
3. Create signing credentials
4. Create JWT token
5. Return token string

---

### Claims

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Email, loginUser.UserName),
    new Claim(ClaimTypes.Role, "Admin"),
};
```

Claims = user identity data  
Roles are just claims.

---

### Security Key

```csharp
var securityKey = new SymmetricSecurityKey(
    Encoding.UTF8.GetBytes(_config["Jwt:Key"])
);
```

---

### appsettings.json

```json
"Jwt": {
  "Key": "YOURKEY",
  "Issuer": "ipaddress",
  "Audience": "ipaddress"
}
```

---

## JWT Authentication Configuration (Program.cs)

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters()
    {
        ValidateActor = true,
        ValidateIssuer = true,
        ValidateAudience = true,
        RequireExpirationTime = true,
        ValidateIssuerSigningKey = true,
        ValidIssuer = builder.Configuration.GetSection("Jwt:Issuer").Value,
        ValidAudience = builder.Configuration.GetSection("Jwt:Audience").Value,
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(
                builder.Configuration.GetSection("Jwt:Key").Value
            )
        )
    };
});
```

---

## JWT Token Generation

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Email, loginUser.UserName),
    new Claim(ClaimTypes.Role, "Admin"),
};

var securityKey = new SymmetricSecurityKey(
    Encoding.UTF8.GetBytes(_config.GetSection("Jwt:Key").Value)
);

var signingCreds = new SigningCredentials(
    securityKey,
    SecurityAlgorithms.HmacSha512Signature
);

var securityToken = new JwtSecurityToken(
    claims: claims,
    expires: DateTime.Now.AddMinutes(60),
    issuer: _config.GetSection("Jwt:Issuer").Value,
    audience: _config.GetSection("Jwt:audience").Value,
    signingCredentials: signingCreds
);

string tokenString = new JwtSecurityTokenHandler()
    .WriteToken(securityToken);

return tokenString;
```

# Swagger (OpenAPI) in ASP.NET Core – Complete Explanation

Swagger (OpenAPI) is used in ASP.NET Core Web API to:
- **Document APIs**
- **Test endpoints via UI**
- **Support API versioning**
- **Expose request/response schemas**

ASP.NET Core uses **Swashbuckle.AspNetCore** to integrate Swagger.

---

## 1. Required NuGet Packages

```bash
dotnet add package Swashbuckle.AspNetCore.Swagger
dotnet add package Swashbuckle.AspNetCore.SwaggerGen
dotnet add package Swashbuckle.AspNetCore.SwaggerUI
```

## What Swagger Generates
- Swagger generates:
- Swagger JSON → /swagger/{version}/swagger.json
- Swagger UI → /swagger
Each API version has its own Swagger document.

## Basic Swagger Registration
```csharp
builder.Services.AddSwaggerGen();
if(app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```
## ConfigureSwaggerOptions implements IConfigureOptions<SwaggerGenOptions>
It is required when you want to use swagger UI with multiple versions in web app.Without it onlt first version will run and other versions will be failed to fetch.

```csharp
using Asp.Versioning.ApiExplorer;
using Microsoft.Extensions.Options;
using Microsoft.OpenApi;
using Swashbuckle.AspNetCore.SwaggerGen;

namespace WebAPIDemo.Options
{
    public class ConfigureSwaggerOptions : IConfigureOptions<SwaggerGenOptions>
    {
        private readonly IApiVersionDescriptionProvider _provider;
        public ConfigureSwaggerOptions(IApiVersionDescriptionProvider provider)
        {
            _provider = provider;
        }
        void IConfigureOptions<SwaggerGenOptions>.Configure(SwaggerGenOptions options)
        {
            foreach (var description in _provider.ApiVersionDescriptions)
            {
                var info = new OpenApiInfo
                {
                    Title = $"Sample API: {description.ApiVersion}",
                    Version = description.ApiVersion.ToString(),
                    Description  ="Api with versioning support"
                };
                options.SwaggerDoc(description.GroupName, info);
            }
            options.DocInclusionPredicate((docName, apiDesc) => apiDesc.GroupName == docName);
        }
    }
}

```
## Caching in .net web api

Caching = storing frequently used data in fast memory so you don’t compute or fetch it again
There are 3 main levels of caching:
1. In-Memory Caching (Application Level)(In .net IMemoryCache)
- Data stored inside the API process RAM
- Lives as long as the application is running

### CACHE INVALIDATION
When do we clear cache?
- User role updated
- User deactivated
- Password changed

## Inject Memory Cache:

```csharp
builder.Services.AddMemoryCache(); //registers in memory cache

// Add cache services 
private readonly IMemoryCache _cache;
public UsersController(IMemoryCache cache)
{
    _cache = cache;
}

//Basic Cache Pattern
public async Task<IActionResult> GetUsers()
{
    if (!_cache.TryGetValue("users", out List<User> users))
    {
        users = await _db.Users.ToListAsync();

        _cache.Set("users", users, TimeSpan.FromMinutes(5));
    }

    return Ok(users);
}

//Cache Expiration Options
new MemoryCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5),
    SlidingExpiration = TimeSpan.FromMinutes(2),
    Priority = CacheItemPriority.High
};
```
Option - Description
AbsoluteExpiration -	Cache expires after fixed time
SlidingExpiration -	Resets timer on access
Priority -	Used during memory pressure

### Important IMemoryCache Methods

 - TryGetValue() -	Check if key exists
 - Get() -	Get cached value
 - Set() -	Store value
 - Remove() -	Delete cache
 - GetOrCreate() - Get or create cache

### Cache Invalidation
```csharp
_cache.Remove("users");
```
### Limitations of Memory Caching
 - Not shared across servers
 - Cache cleared on app restart
 - Not suitable for load-balanced environments
   
2. Response Caching (HTTP Level)
Response caching stores the entire HTTP response so repeated requests return cached responses without executing the controller again.
- Caching entire HTTP responses
- Based on:
    URL
    Headers
    Query params

```csharp
// Register in program.cs:
builder.Services.AddResponseCaching();
app.UseResponseCaching();
```

### Response cahcing in controllers
```csharp
[HttpGet]
[ResponseCache(Duration = 60)]
public IActionResult GetProducts()
{
    return Ok(products);
}
```

### Response cache properties:
 - Duration - Cache time (seconds)
 - Location - Client / Any / None
 - NoStore - Disable caching
 - VaryByHeader - Cache per header
 - VaryByQueryKeys - Cache per query string

### Limitations of response caching
 - Works only for GET & HEAD
 - Not suitable for authenticated users
 - No fine-grained invalidation
 - Cannot cache POST responses

## Localization
Localization is the process of making an application support multiple languages and cultures by:
Translating text/messages
Formatting dates, numbers, currency
Adapting UI & validation messages

Example:
Hello → नमस्ते (Hindi)
12/31/2025 → 31/12/2025
₹1,23,456.50 → $123,456.50
 
