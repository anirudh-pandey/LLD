# Authentication & Authorization

## In One Line
Authentication verifies who you are; authorization decides what you are allowed to do.

## Core Idea
These are two separate steps that always happen in order. Authentication comes first — it reads a token or cookie from the request and figures out the user's identity. Authorization comes second — it checks whether that identity has permission to access the requested resource.

In ASP.NET Core, both are middleware components. `UseAuthentication()` populates `HttpContext.User` with the caller's identity and claims. `UseAuthorization()` reads `[Authorize]` attributes on endpoints and checks the user's claims against the requirements.

The glue between them is **claims**. Authentication produces claims (name, role, email, subscription level). Authorization consumes them.

## Why Do We Need It?
Without authentication, you have no idea who is calling your API. Every request is anonymous. You cannot personalize responses, audit actions, or restrict access.

Without authorization, every authenticated user can do everything — an intern can delete the database, a free-tier user can access premium features. Authorization enforces boundaries.

Separating the two keeps things clean. The auth middleware does not care about permissions. The authorization middleware does not care how the user proved their identity — JWT, cookie, OAuth — it just reads claims.

## What Goes Wrong Without It?
If you skip authentication and do ad-hoc checks in controllers, some endpoints forget to validate. One controller checks the token, another trusts the request blindly. Bugs are inconsistent and hard to trace.

If you hardcode authorization logic inside controllers, it scatters everywhere. Changing "only admins can delete" means hunting through every controller. With policies, you change it in one place.

Swapping the middleware order (`UseAuthorization` before `UseAuthentication`) means authorization runs before the user identity is set — every request fails or, worse, gets through unchecked.

## Simple Example
Bad design — manual auth in every action:

```csharp
[HttpDelete("/users/{id}")]
public IActionResult DeleteUser(int id)
{
    var token = Request.Headers["Authorization"];
    var user = ValidateToken(token);       // manual auth
    if (user == null) return Unauthorized();
    if (user.Role != "Admin") return Forbid(); // manual authz
    _service.Delete(id);
    return Ok();
}
```

Every action repeats this. Easy to forget, hard to maintain.

Better design — let middleware handle it:

```csharp
[Authorize(Roles = "Admin")]
[HttpDelete("/users/{id}")]
public IActionResult DeleteUser(int id)
{
    _service.Delete(id);
    return Ok();
}
```

Authentication and authorization happen before the action is ever reached. The controller stays clean.

## How Authentication Works (JWT Example)
At startup, you register the JWT bearer scheme:

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = config["Jwt:Issuer"],
            ValidAudience = config["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(config["Jwt:Key"]!))
        };
    });
```

When a request arrives, the middleware reads the `Authorization: Bearer <token>` header, validates the signature, expiry, issuer, and audience, extracts claims, and sets `HttpContext.User`. If the token is invalid or missing, it returns 401.

## How Authorization Works
Three levels, from simple to flexible:

**Simple** — just require a logged-in user:
```csharp
[Authorize]
```

**Role-based** — check the `Role` claim:
```csharp
[Authorize(Roles = "Admin")]
```

**Policy-based** — custom rules defined at startup:
```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("PremiumUser",
        policy => policy.RequireClaim("Subscription", "Premium"));
});
```
```csharp
[Authorize(Policy = "PremiumUser")]
```

Policies are the most powerful. They can combine roles, claims, and custom logic in one place.

## Real Meaning
Authentication is not just "checking a password." It is the middleware that turns a raw token into a populated `HttpContext.User` with claims.

Authorization is not just "checking a role." It is a framework that evaluates attributes, policies, and claims to decide access — centrally, not scattered in controllers.

Claims are not just key-value pairs. They are the contract between authentication and authorization. Auth produces them, authz consumes them.

## Strong Mental Model
Ask this question:

Does this endpoint need to know WHO is calling, or WHAT they are allowed to do — or both?

If "who" → authentication. If "what" → authorization. Almost always both.

## Interview-Ready Definition
Authentication is the process of verifying a caller's identity — typically by validating a JWT or cookie — and populating `HttpContext.User` with claims like name, role, and email. Authorization then checks those claims against requirements defined by `[Authorize]` attributes or policies to decide whether the user can access a specific endpoint.

## Interview-Ready Why
We need authentication to establish trusted identity, and authorization to enforce access control — both as middleware so that every request is handled consistently, without duplicating checks across controllers. Claims are the bridge: authentication produces them, authorization consumes them.

## Common Interview Questions
**What is the difference between authentication and authorization?**
Authentication = "who are you?" (identity). Authorization = "what can you do?" (permissions). Auth produces claims, authz reads them. 401 for auth failure, 403 for authz failure.

**What happens if you swap UseAuthentication and UseAuthorization?**
Authorization runs without knowing who the user is. Every request either fails or bypasses checks. Authentication must always come first.

**What are claims in .NET?**
Key-value pairs attached to a user identity (e.g., Role=Admin, Email=x@y.com). Authentication middleware extracts them from the token. Authorization middleware evaluates them against policies and attributes.

**What is the difference between role-based and policy-based authorization?**
Role-based checks a single `Role` claim. Policy-based can combine multiple claims, roles, and custom logic. Policies are more flexible and are defined once at startup.

**What HTTP status codes indicate auth failures?**
401 Unauthorized = authentication failed (who are you?). 403 Forbidden = authorization failed (you are not allowed).

## Fast Recall
Think: Authentication = who are you? Authorization = what can you do?
Think: Claims are the bridge — auth produces, authz consumes.
Think: 401 = identity unknown. 403 = identity known, permission denied.

## Questions To Ask In Design
Does this endpoint need authentication, authorization, or both?
Should this rule be a role check or a custom policy?
What claims does the token need to carry for my authorization rules to work?
Can this endpoint be [AllowAnonymous] or does it need protection?
