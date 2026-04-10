# Request Pipeline (ASP.NET Core)

## In One Line
The request pipeline is the ordered chain of middleware that every HTTP request passes through before reaching your controller, and every response passes back through on the way out.

## Core Idea
When a request hits your app, it does not go straight to the controller. It flows through a series of middleware components — each one does a small job like logging, authentication, or error handling.

Each middleware can inspect the request, pass it forward by calling `next()`, and then inspect the response on the way back. Think of it like airport security checkpoints — you pass through each one before reaching your gate.

The pipeline is configured once at startup in `Program.cs`. The order you register middleware is the exact order they execute. This order matters a lot.

## Why Do We Need It?
Without a pipeline, you would have to stuff every cross-cutting concern — logging, auth, error handling, CORS — directly into your controller actions. That is messy and repetitive.

The pipeline gives you separation of concerns. Each middleware handles one job, runs for every request, and is written once. Your controllers stay focused on business logic.

It also gives you control. You can reject bad requests early (short-circuit) without wasting resources reaching the controller. For example, if authentication fails, the auth middleware returns a 401 immediately.

## What Goes Wrong Without It?
Without a pipeline, cross-cutting logic gets scattered across controllers. One controller remembers to check auth, another forgets. One logs the request, another does not.

Error handling becomes inconsistent. Some actions catch exceptions, others let them bubble up and crash. There is no single place to handle failures uniformly.

You also lose the ability to short-circuit. Every request, even obviously bad ones, would travel all the way to the controller before being rejected.

## Simple Example
Bad design — doing everything in the controller:

```csharp
[HttpGet("/orders")]
public IActionResult GetOrders()
{
    Log("Request received");              // logging
    if (!IsAuthenticated()) return Unauthorized(); // auth
    if (!IsAuthorized()) return Forbid();          // authorization
    var result = _service.GetOrders();
    Log("Response sent");
    return Ok(result);
}
```

Every controller action repeats this. Messy, fragile, easy to forget a step.

Better design — pipeline handles it:

```csharp
// Program.cs
app.UseExceptionHandler("/error");
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

```csharp
[Authorize]
[HttpGet("/orders")]
public IActionResult GetOrders()
{
    return Ok(_service.GetOrders());  // clean, focused
}
```

Auth, error handling, and logging are handled before the controller is ever reached.

## Key Middleware Concepts
`app.Use()` — adds middleware that calls `next()` to pass the request forward. Runs on the way in and on the way out.

`app.Run()` — terminal middleware. Does NOT call `next()`. Ends the pipeline. Always placed last.

`app.Map()` — branches the pipeline based on the request path. Different paths can have different middleware chains.

`RequestDelegate` — the delegate type that represents the next middleware in the chain. When you call `await next()`, you are invoking this delegate.

## Real Meaning
The pipeline is not just "middleware runs before the controller."

- Each middleware wraps the next one like nested layers. Request flows in, response flows back out through the same layers in reverse.
- Order of registration = order of execution. Swap two lines and your app can break.
- Short-circuiting is a feature, not a hack. It is how auth middleware rejects requests early.

## Custom Middleware Example

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Before: " + context.Request.Path);
    await next();
    Console.WriteLine("After: " + context.Response.StatusCode);
});
```

Or as a class with `InvokeAsync(HttpContext context)` and a constructor that takes `RequestDelegate next`.

## Strong Mental Model
Ask this question:

If I removed this middleware from the pipeline, what would break or go unprotected?

If the answer is "nothing," you probably do not need it. If the answer is "auth stops working" or "exceptions go unhandled," it belongs in the pipeline.

## Interview-Ready Definition
The request pipeline in ASP.NET Core is an ordered sequence of middleware components configured at startup. Each middleware can inspect or modify the request, pass it to the next middleware by calling `next()`, and then process the response on the way back. The order of registration determines execution order, and any middleware can short-circuit the pipeline by not calling `next()`.

## Interview-Ready Why
We need the pipeline to handle cross-cutting concerns like authentication, logging, error handling, and CORS in a centralized, reusable way — so controllers stay clean and every request is processed consistently without duplicating logic.

## Common Interview Questions
**What is middleware in ASP.NET Core?**
A component in the request pipeline that handles a specific concern. It receives the request, optionally processes it, calls `next()` to pass it forward, and can also process the response on the way back.

**Does the order of middleware matter?**
Yes. Middleware runs in the exact order it is registered. For example, `UseAuthentication` must come before `UseAuthorization`, otherwise authorization runs without knowing who the user is.

**What is short-circuiting?**
When a middleware does not call `next()` and returns a response directly. The remaining middleware and the controller are never reached. Example: auth middleware returning 401.

**What is the difference between `app.Use()` and `app.Run()`?**
`Use()` calls `next()` and passes the request forward. `Run()` is terminal — it does not call `next()` and ends the pipeline.

**How do you write custom middleware?**
Either inline with `app.Use(async (context, next) => { ... })` or as a class with a constructor taking `RequestDelegate` and an `InvokeAsync(HttpContext)` method.

## Fast Recall
Think: Request flows through middleware in order, response flows back in reverse.
Think: Order of registration = order of execution. Swap two lines, app breaks.
Think: Short-circuit = don't call `next()`, respond immediately.

## Questions To Ask In Design
Does this concern apply to every request, or only specific routes?
Should this middleware run before or after authentication?
Can I reject this request early without reaching the controller?
Am I duplicating logic in controllers that belongs in a middleware?
