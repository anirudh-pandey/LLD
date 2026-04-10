# .NET Core Project Files

## In One Line
These are the key files that every .NET project starts with — each one has a specific job in wiring up and running your application.

---

## The Files

### Program.cs — The Entry Point
This is where the application starts. Think of it as the main gate. When you run `dotnet run`, execution begins here.

In .NET 6+, `Program.cs` does everything: it creates the app host, registers services, sets up middleware, and starts listening for requests.

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();               // register services (DI)
builder.Services.AddScoped<IMyService, MyService>();

var app = builder.Build();

app.UseAuthentication();                         // middleware pipeline
app.UseAuthorization();
app.MapControllers();

app.Run();                                       // start the server
```

---

### Startup.cs — The Old Config Hub (Pre-.NET 6)
Before .NET 6, configuration was split into a separate class. It had exactly two methods, each with a distinct responsibility:

`ConfigureServices` — registers everything into the DI container (your services, DB context, auth). This runs first.

`Configure` — sets up the middleware pipeline (the order in which HTTP requests are processed). This runs after services are ready.

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        services.AddScoped<IMyService, MyService>();
    }

    public void Configure(IApplicationBuilder app)
    {
        app.UseRouting();
        app.UseAuthentication();
        app.UseAuthorization();
        app.UseEndpoints(e => e.MapControllers());
    }
}
```

In .NET 6+, `Startup.cs` is gone by default. Both methods were folded into `Program.cs`. You can still bring it back manually if you prefer separation.

---

### appsettings.json — The Configuration File
Stores settings you don't want hardcoded: connection strings, logging levels, feature flags, app-specific values.

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=...;Database=mydb;"
  },
  "Logging": {
    "LogLevel": { "Default": "Information" }
  }
}
```

`appsettings.Development.json` overrides values only in the Development environment. Same keys, different values. The correct file is loaded automatically based on the `ASPNETCORE_ENVIRONMENT` variable.

Never put real secrets (passwords, API keys) in `appsettings.json` for production. Use environment variables, Secret Manager (dev), or Azure Key Vault (prod).

---

### launchSettings.json (inside `Properties/`)
Controls how the app runs **locally during development only**. Sets the port, the environment name, and whether to auto-open the browser.

This file is **never deployed**. It has no role in staging or production.

---

### .csproj — The Project File
The blueprint of your project. Tells the .NET SDK: what framework to target, what NuGet packages to include, and build settings.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>
</Project>
```

---

### .sln — The Solution File
A container that groups multiple `.csproj` projects together (e.g., API project + class library + test project). You open the `.sln` in Visual Studio; `dotnet` CLI uses it to build all projects together.

---

### Controllers/
Where your HTTP endpoints live. Each controller handles requests for one resource. The `[HttpGet]`, `[HttpPost]` etc. attributes map methods to HTTP verbs.

---

### wwwroot/
Holds static files: CSS, JS, images. Only relevant if you're serving a web UI (MVC/Razor Pages). Pure Web API projects often don't have this.

---

## Key Concepts to Know

### Middleware Order Matters
Each `app.UseXxx()` call adds a step to the request pipeline. The order is top-to-bottom and it matters. A classic mistake is placing `UseAuthorization` before `UseAuthentication` — authorization runs without knowing who the user is, so it always fails.

Correct order:
```
Request → UseExceptionHandler → UseRouting → UseAuthentication → UseAuthorization → Endpoint
```

### ConfigureServices vs Configure
`ConfigureServices` = *what exists* in the app (the DI container).
`Configure` = *how requests flow* through the app (the middleware pipeline).
Think of it as: first you stock the warehouse, then you set up the conveyor belt.

---

## Interview-Ready Definitions

**Program.cs:** The application entry point. In .NET 6+, it creates the web application host, registers services into the DI container, configures the middleware pipeline, and starts the server with `app.Run()`.

**Startup.cs:** A class used in pre-.NET 6 apps to separate service registration (`ConfigureServices`) from middleware configuration (`Configure`). In .NET 6+, its responsibilities moved into `Program.cs`.

**appsettings.json:** A JSON configuration file for storing app settings like connection strings and logging levels. Environment-specific overrides (e.g., `appsettings.Development.json`) are loaded automatically based on the current environment.

**.csproj:** The MSBuild project file that defines the target framework, NuGet package dependencies, and build properties for a .NET project.

---

## Common Interview Questions

**Q: What changed between .NET 5 and .NET 6 in terms of project structure?**
.NET 6 introduced the minimal hosting model. `Startup.cs` was removed by default, and everything — service registration and middleware setup — was merged into `Program.cs`. Top-level statements also removed the need for an explicit `Main` method.

**Q: What's the difference between `ConfigureServices` and `Configure`?**
`ConfigureServices` registers services into the DI container (what your app has). `Configure` sets up the middleware pipeline (how requests are handled). They run in that order at startup.

**Q: What happens if you reverse `UseAuthentication` and `UseAuthorization`?**
Authorization will run before the user identity is established, so it will always treat the request as unauthenticated and deny access even for valid tokens. Order in the pipeline is critical.

**Q: Where should you store secrets like API keys or DB passwords?**
Not in `appsettings.json` for production. Use .NET Secret Manager for local dev, environment variables for CI/staging, or Azure Key Vault / AWS Secrets Manager for production.

**Q: What is the difference between `.csproj` and `.sln`?**
`.csproj` defines a single project (framework, packages, build settings). `.sln` is a solution file that groups multiple `.csproj` projects so they can be built and run together as one solution.

**Q: Is `launchSettings.json` deployed to production?**
No. It is a local development-only file used by the .NET CLI and Visual Studio to configure how the app starts on your machine. It has no effect outside of local development.

---

## Fast Recall
- Think: `Program.cs` = startup + DI + middleware, all in one place (.NET 6+)
- Think: `Startup.cs` = old way, split into "register" (`ConfigureServices`) and "configure" (`Configure`)
- Think: middleware order = top to bottom, authentication must come before authorization
