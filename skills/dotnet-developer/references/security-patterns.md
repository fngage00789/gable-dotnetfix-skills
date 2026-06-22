# Security patterns

## JWT auth (bearer)
```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(o =>
    {
        o.TokenValidationParameters = new()
        {
            ValidateIssuer = true,            ValidIssuer = cfg["Jwt:Issuer"],
            ValidateAudience = true,          ValidAudience = cfg["Jwt:Audience"],
            ValidateLifetime = true,          ClockSkew = TimeSpan.FromSeconds(30),
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(cfg["Jwt:Key"]!))
        };
    });
builder.Services.AddAuthorization();
```
- signing key in Key Vault / env, never appsettings.json.
- prefer asymmetric (RS256) for multi-service; HS256 ok single-issuer.

## issue token
```csharp
var claims = new[] { new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
                     new Claim(ClaimTypes.Role, user.Role) };
var token = new JwtSecurityToken(issuer, audience, claims,
    expires: DateTime.UtcNow.AddMinutes(15),     // short-lived
    signingCredentials: new(key, SecurityAlgorithms.HmacSha256));
// pair with a refresh token (rotated, stored hashed, revocable).
```

## RBAC + policies
```csharp
builder.Services.AddAuthorization(o =>
{
    o.AddPolicy("CanApprove", p => p.RequireRole("Manager", "Director"));
    o.AddPolicy("Adult", p => p.RequireClaim("age").RequireAssertion(c =>
        int.Parse(c.User.FindFirstValue("age")!) >= 18));
});
// endpoint: .RequireAuthorization("CanApprove")
```

## OWASP top hits — .NET defenses
- **injection**: parameterize SQL (EF/Dapper params), never concat.
- **broken access**: check resource ownership server-side, not just role. don't trust client ids.
- **secrets**: user-secrets (dev), Key Vault/env (prod). scan repo (gitleaks).
- **mass assignment**: bind to DTOs, never entities. allowlist fields.
- **XSS** (Blazor): framework encodes by default; `MarkupString` is the danger — sanitize.
- **CSRF**: antiforgery token on cookie-auth forms; bearer-token APIs are immune.
- **SSRF**: validate/allowlist outbound URLs from user input.

## headers + transport
```csharp
app.UseHsts();
app.UseHttpsRedirection();
app.Use(async (ctx, next) => {
    ctx.Response.Headers["X-Content-Type-Options"] = "nosniff";
    ctx.Response.Headers["X-Frame-Options"] = "DENY";
    ctx.Response.Headers["Content-Security-Policy"] = "default-src 'self'";
    await next();
});
```

## passwords / secrets at rest
- hash with ASP.NET `PasswordHasher<T>` (PBKDF2) or Argon2. never MD5/SHA-only.
- data protection keys persisted (Key Vault/blob) for multi-instance.

## CORS (tighten)
```csharp
builder.Services.AddCors(o => o.AddPolicy("app", p =>
    p.WithOrigins("https://app.example.com").AllowAnyHeader().AllowAnyMethod()));
```
- never `AllowAnyOrigin()` with credentials.
