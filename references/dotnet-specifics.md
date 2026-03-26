# .NET — Language-Specific Analysis

Read this file when the project is detected as .NET (*.csproj, *.sln, or global.json present).

## Stack Detection

### Version Detection
```bash
# .NET version
grep "TargetFramework" *.csproj **/*.csproj 2>/dev/null
cat global.json 2>/dev/null

# Check for specific version indicators
grep "netcoreapp\|net5\|net6\|net7\|net8" *.csproj **/*.csproj 2>/dev/null
```

### Common Stack Variants

| Variant | Key Indicators | Era |
|---------|---------------|-----|
| .NET Core 2.0/2.1 | `netcoreapp2.0` / `netcoreapp2.1` | 2017–2019 |
| .NET Core 3.0/3.1 | `netcoreapp3.0` / `netcoreapp3.1` | 2019–2021 |
| .NET 5 | `net5.0` | 2020–2021 |
| .NET 6 | `net6.0` | 2021–2023 |
| .NET 7/8 | `net7.0` / `net8.0` | 2022+ |
| ASP.NET Core Web API | `Microsoft.AspNetCore.*` packages | Core 2.0+ |
| ASP.NET Core MVC | Views + Controllers pattern | Core 2.0+ |
| Blazor | `Microsoft.AspNetCore.Components.*` | Core 3.0+ |

---

## .NET-Specific Analysis Points

### 1. Project Structure

```bash
# Find solution and project files
find . -name "*.sln" -o -name "*.csproj" | head -20
```

Common layouts:
```
Solution/
├── Solution.sln
├── src/
│   ├── Project.Api/              # Web API project
│   │   ├── Controllers/
│   │   ├── Program.cs
│   │   └── Startup.cs            # .NET Core 2.x/3.x (merged into Program.cs in 6+)
│   ├── Project.Application/      # Business logic
│   │   ├── Services/
│   │   ├── DTOs/
│   │   └── Interfaces/
│   ├── Project.Domain/           # Domain models
│   │   ├── Entities/
│   │   └── Repositories/
│   └── Project.Infrastructure/   # Data access, external services
│       ├── Data/
│       ├── Repositories/
│       └── Migrations/
└── tests/
    └── Project.Tests/
```

Check:
- [ ] Solution structure: mono-project vs multi-project
- [ ] Clean Architecture / Onion Architecture layers?
- [ ] Project reference dependencies

### 2. Startup / Host Configuration

#### .NET Core 2.x / 3.x
```csharp
// Startup.cs — two methods
public class Startup
{
    public void ConfigureServices(IServiceCollection services) { ... }
    public void Configure(IApplicationBuilder app) { ... }
}
```

#### .NET 6+
```csharp
// Program.cs — minimal hosting
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
var app = builder.Build();
app.MapControllers();
app.Run();
```

**Critical**: Don't convert Startup.cs pattern to minimal hosting or vice versa.

### 3. Dependency Injection

```bash
# Check DI registrations
grep -rn "services\.Add\|builder\.Services\.Add\|AddScoped\|AddTransient\|AddSingleton" --include="*.cs" | head -20
```

Check:
- [ ] Registration pattern: Scoped / Transient / Singleton usage
- [ ] Extension method grouping: `services.AddApplicationServices()`, `services.AddInfrastructure()`
- [ ] Third-party DI containers: Autofac, Ninject (rare in Core)
- [ ] Interface-implementation mapping conventions

### 4. Controller Patterns

```bash
# Find controllers
find . -name "*Controller.cs" | head -20
grep -rn "\[ApiController\]\|\[Route\]\|\[HttpGet\]\|\[HttpPost\]" --include="*.cs" | head -20
```

Check:
- [ ] `[ApiController]` attribute present?
- [ ] Route attribute style: `[Route("api/[controller]")]` vs `[Route("api/users")]`
- [ ] Action return type: `IActionResult`, `ActionResult<T>`, `Task<ActionResult<T>>`
- [ ] Model binding: `[FromBody]`, `[FromQuery]`, `[FromRoute]`
- [ ] Response wrapping: custom `ApiResponse<T>` or raw types
- [ ] Swagger attributes: `[ProducesResponseType]`, `[SwaggerOperation]`

### 5. Entity Framework Core

```bash
# Check EF Core usage
grep -rn "DbContext\|DbSet\|entity\.Property\|modelBuilder\|Migration" --include="*.cs" | head -20
```

Check:
- [ ] DbContext location and configuration style
- [ ] Fluent API vs Data Annotations for entity configuration
- [ ] Migration strategy: EF Core migrations, SQL scripts, or Evolve
- [ ] Query patterns: LINQ, raw SQL, stored procedures
- [ ] Tracking behavior: default or NoTracking
- [ ] Eager loading patterns: `.Include()`, `.ThenInclude()`
- [ ] Repository pattern wrapping EF Core?
- [ ] Unit of Work pattern?

### 6. Configuration

```bash
# Configuration files
find . -name "appsettings*.json" | head -10
grep -rn "IConfiguration\|IOptions<\|Configuration\[" --include="*.cs" | head -10
```

Check:
- [ ] Configuration binding: `IOptions<T>`, `IConfiguration.GetSection()`, direct indexer
- [ ] Environment-specific configs: `appsettings.Development.json`, `appsettings.Production.json`
- [ ] User secrets for development
- [ ] Custom configuration sections and their naming

### 7. Authentication / Authorization

```bash
grep -rn "\[Authorize\]\|AddAuthentication\|AddJwtBearer\|UseAuthentication\|services\.AddIdentity" --include="*.cs" | head -20
```

Check:
- [ ] Auth scheme: JWT Bearer, Cookie, Identity, IdentityServer
- [ ] Authorization: Role-based, Policy-based, Claims-based
- [ ] `[Authorize]` attribute usage (global vs per-controller vs per-action)
- [ ] Custom auth handlers or requirements

### 8. Error Handling

```bash
# Find exception handling middleware
grep -rn "UseExceptionHandler\|ExceptionMiddleware\|IExceptionFilter\|try\s*{" --include="*.cs" | head -20
find . -name "*Exception.cs" | head -10
```

Check:
- [ ] Global exception handling: middleware or filter
- [ ] Custom exception types
- [ ] ProblemDetails usage (RFC 7807)
- [ ] Error response format

### 9. Logging

```bash
grep -rn "ILogger\|_logger\|LogInformation\|LogError\|LogWarning" --include="*.cs" | head -20
grep -rn "Serilog\|NLog\|Log4Net" --include="*.cs" *.csproj 2>/dev/null
```

Check:
- [ ] Logging framework: built-in `ILogger<T>`, Serilog, NLog
- [ ] Structured logging or message templates
- [ ] Log level conventions
- [ ] Correlation ID / request tracing

### 10. Middleware Pipeline

### 11. Using Directive Consistency
- Check `using` statement ordering: `System.*` → `Microsoft.*` → third-party → project namespaces
- Check for `global using` directives (.NET 6+) — if present, AI should follow the same pattern
- Check `using static` patterns: `using static System.Math` — when and where used
- Check for `using` aliases: `using Json = Newtonsoft.Json.JsonConvert` — are they consistent?
- Check if `<ImplicitUsings>` is enabled — affects which `using` statements are needed explicitly

```bash
grep -rn "app\.Use\|app\.Map\|app\.Run" --include="*.cs" | head -20
```

Document the middleware order in `Startup.cs` or `Program.cs` — order matters:
1. Exception handling
2. HTTPS redirection
3. Static files
4. Routing
5. CORS
6. Authentication
7. Authorization
8. Endpoints

---

## Common Anti-Patterns in Legacy .NET

Things AI will want to "fix" but should NOT:

1. **Startup.cs + Program.cs pattern** — AI will try to convert to .NET 6+ minimal hosting. Don't.
2. **Synchronous code** — AI will try to make everything async. Only if the project is already async.
3. **Repository + Unit of Work over EF Core** — AI may want to remove the abstraction layer. Don't.
4. **Data Annotations on entities** — AI may try to convert to Fluent API. Follow the project's pattern.
5. **MediatR / CQRS** — If the project uses it, follow it. If not, don't introduce it.
6. **Nullable reference types** — AI will add `?` annotations. Only if the project has `<Nullable>enable</Nullable>`.
7. **Global using statements** — Only for .NET 6+.
8. **Record types** — Only for C# 9+ (.NET 5+).
9. **Primary constructors** — Only for C# 12 (.NET 8).
10. **File-scoped namespaces** — Only for C# 10 (.NET 6+).

---

## Known Version Boundaries (Reference)

> The following are feature boundary references for known versions. If the project uses a version not listed here, analyze based on actually detected version features and note version constraints in the generated rules.

### .NET Core 2.x
- `Startup.cs` with `ConfigureServices` and `Configure`
- `IHostingEnvironment` (not `IWebHostEnvironment`)
- `AddMvc()` not `AddControllers()`
- `ILogger` available but may use third-party

### .NET Core 3.x
- `IWebHostEnvironment` replaces `IHostingEnvironment`
- `AddControllers()` / `AddControllersWithViews()` / `AddRazorPages()`
- System.Text.Json becomes default (not Newtonsoft.Json)
- Worker services support

### .NET 5
- Single target framework moniker `net5.0`
- C# 9: records, init-only setters, top-level statements
- Package reference trimming

### .NET 6
- Minimal hosting model (no Startup.cs needed)
- Global usings
- File-scoped namespaces
- C# 10 features
- Hot reload

### .NET 7/8
- Rate limiting middleware built-in
- Output caching
- C# 11/12 features (primary constructors, collection expressions)

Check `TargetFramework` in .csproj to determine available C# language features.

### Other Versions
For .NET 9+, or other versions not listed above — detect the actual framework API and C# language features from the project's code. Record differences from the known versions above and flag as `LOW_CONFIDENCE` for user confirmation.

---

## Key Dependencies to Track

| Dependency | Why It Matters |
|-----------|---------------|
| `Microsoft.EntityFrameworkCore` | EF Core version — API changes between versions |
| `Swashbuckle.AspNetCore` | Swagger/OpenAPI generation |
| `AutoMapper` | Object mapping — version-specific API |
| `MediatR` | CQRS mediator pattern |
| `FluentValidation` | Validation library |
| `Serilog` | Structured logging |
| `Polly` | Resilience/retry policies |
| `Dapper` | Micro ORM — often used alongside EF Core |
| `Newtonsoft.Json` | JSON handling — may still be used over System.Text.Json |
| `StackExchange.Redis` | Redis client |
| `MassTransit` / `RabbitMQ.Client` | Message queue |
| `IdentityServer4` / `Duende.IdentityServer` | Authentication server |
