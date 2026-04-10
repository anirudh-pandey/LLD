# Service Lifetimes in .NET

## In One Line
Service lifetime controls how long a DI-created object lives — a fresh one every time (Transient), one per request (Scoped), or one for the whole app (Singleton).

## Core Idea
When you register a service in the DI container, you also decide its lifetime. This tells the container whether to create a new instance every time, reuse one within a request, or reuse one globally.

Picking the wrong lifetime leads to subtle bugs — stale data, disposed objects, or unexpected shared state. The lifetime should match how the service holds and shares state.

## Transient — "Disposable Gloves"
Every time any class asks for this service, a brand new instance is created. No sharing.

```csharp
builder.Services.AddTransient<IValidator, OrderValidator>();
```

If two classes in the same request both need `IValidator`, they get two separate objects. Use for lightweight, stateless services like validators, formatters, or GUID generators.

**Interview line:** "Transient means a new instance every time. Like disposable gloves — grab a fresh pair each time."

## Scoped — "A Notepad for One Meeting"
One instance per HTTP request. Every class within that request shares the same object. Next request gets a fresh one.

```csharp
builder.Services.AddScoped<AppDbContext>();
```

This is the classic lifetime for `DbContext`. Within one request, `OrderService` and `PaymentService` both get the same `DbContext` — so they share the same transaction. When the request ends, the `DbContext` is disposed.

**Interview line:** "Scoped means one instance per HTTP request. The textbook example is DbContext — all services in a request share one connection and transaction."

## Singleton — "The Office Printer"
One instance for the entire application. Created on first use, lives until the app shuts down. Every request and every class gets the same object.

```csharp
builder.Services.AddSingleton<ICacheService, InMemoryCacheService>();
```

Use for expensive-to-create, stateless, or thread-safe services like caches, configuration, or `HttpClientFactory`. Must be thread-safe since multiple requests hit it concurrently.

**Interview line:** "Singleton means one instance for the whole app. Like the office printer — everyone shares the same one."

## The Lifetime Dependency Rule
A service can only depend on services with an equal or longer lifetime.

| Service | Can depend on |
|---|---|
| Transient | Transient, Scoped, Singleton |
| Scoped | Scoped, Singleton |
| Singleton | Singleton only |

**Why?** A singleton lives forever. If it captures a scoped `DbContext`, that `DbContext` gets disposed when the request ends — but the singleton still holds a reference to it. Next request, the singleton tries to use a dead object → `ObjectDisposedException`.

This is called the **Captive Dependency** problem. .NET throws `InvalidOperationException` in Development mode to prevent it.

**Mental model:** A container can't hold water that evaporates before the container does.

## Config & Options Pattern
For static configuration, use `IOptions<T>` — it's registered as Singleton by default.

```csharp
builder.Services.Configure<PaymentSettings>(builder.Configuration.GetSection("PaymentSettings"));
```

If config can change at runtime: `IOptionsSnapshot<T>` (Scoped, refreshes per request) or `IOptionsMonitor<T>` (Singleton, live-reloads with callback).

## Strong Mental Model
Ask yourself: "How long should this object live, and who should share it?"

If nobody should share it → Transient. If a request should share it → Scoped. If the whole app should share it → Singleton.

## Interview-Ready Definition
.NET DI offers three lifetimes: Transient (new instance every time), Scoped (one per HTTP request), and Singleton (one for the whole app). The key rule is that a service can only depend on services with an equal or longer lifetime — otherwise you get a captive dependency where a long-lived service holds a dead short-lived one.

## Common Interview Questions
**When would you use Scoped over Transient?**
When multiple services in the same request need to share state — like a `DbContext` for a single unit of work.

**What happens if a Singleton depends on a Scoped service?**
The scoped service gets captured and held forever. When it's disposed at request end, the singleton uses a dead object. .NET throws an exception in Development to prevent this.

**Is Singleton thread-safe by default?**
No. The container creates one instance, but you must make it thread-safe yourself since multiple threads will access it simultaneously.

**What lifetime is DbContext registered as?**
Scoped. One per request so all repositories share the same connection and transaction.

**What's the difference between IOptions, IOptionsSnapshot, and IOptionsMonitor?**
`IOptions<T>` is Singleton, reads once. `IOptionsSnapshot<T>` is Scoped, refreshes per request. `IOptionsMonitor<T>` is Singleton but live-reloads on config file changes.

## Fast Recall
Think: Transient = new every time. Scoped = new every request. Singleton = new once, ever.
Think: DbContext is Scoped. Cache is Singleton. Validator is Transient.
Think: Long-lived service must never capture a short-lived one.

## Questions To Ask In Design
Does this service hold state that should be shared within a request?
Is this service expensive to create or does it need to live for the whole app?
Am I accidentally injecting a shorter-lived service into a longer-lived one?
Does this singleton need to be thread-safe?
