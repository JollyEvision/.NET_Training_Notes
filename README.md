# .NET_Training_Notes

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

---
