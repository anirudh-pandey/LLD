Here is your complete 2-3 day senior .NET/C# interview study plan.

---

# Senior .NET/C# Interview - 3-Day Study Plan

---

## DAY 1 - C# Core: OOP - SOLID - Generics - LINQ

| Time Block | Topic | Priority |
|---|---|---|
| 9:00-10:30 AM | OOP + SOLID Principles | [MUST] |
| 10:45-12:15 PM | Generics | [MUST] |
| 1:15-2:45 PM | LINQ | [MUST] |
| 3:00-4:30 PM | Java Coding Practice - Problems 1-4 | [MUST] |

---

### Block 1 - OOP + SOLID [MUST]

**OOP framing for interviewers:**

| Concept | One-liner | Say to interviewer |
|---|---|---|
| Encapsulation | Control internal state access | "Prevents invariants being broken externally" |
| Abstraction | Depend on contracts, not implementations | "Client code should not know how, only what" |
| Inheritance | IS-A hierarchy, reuse behaviour | "Powerful but easy to abuse - I prefer composition" |
| Polymorphism | Same interface, different runtime behaviour | "Enables Open/Closed - add behaviour without changing callers" |

---

**S - Single Responsibility**
```csharp
// Bad: one class handles persistence + notification
public class OrderService {
    public void PlaceOrder(Order o) { SaveToDb(o); SendEmail(o); }
}

// Good: each class has one reason to change
public class OrderService    { public void PlaceOrder(Order o) => _repo.Save(o); }
public class NotificationService { public void Notify(Order o) => _mailer.Send(o); }
```

**O - Open/Closed**
```csharp
// Bad: switch breaks OCP - every new type requires editing this method
double GetDiscount(string type) => type == "VIP" ? 0.2 : 0.1;

// Good: add new discount by adding a class, not changing existing code
public interface IDiscountStrategy { double GetRate(); }
public class VipDiscount      : IDiscountStrategy { public double GetRate() => 0.2; }
public class StandardDiscount : IDiscountStrategy { public double GetRate() => 0.1; }
```

**L - Liskov Substitution**
```csharp
// Classic trap: Square extends Rectangle
// Setting Width on a Square also changes Height - breaks Rectangle's contract
// Rectangle r = new Square(); r.Width = 5; r.Height = 3; Assert(r.Area == 15); // FAILS

// Interview answer: "If a subclass cannot be substituted without breaking
// caller assumptions, LSP is violated. Prefer composition here."
```

**I - Interface Segregation**
```csharp
// Bad: Robot cannot eat, so it throws NotImplementedException - a design smell
interface IWorker { void Work(); void Eat(); }

// Good: split by client need
interface IWorkable { void Work(); }
interface IFeedable  { void Eat(); }
public class Human  : IWorkable, IFeedable { ... }
public class Robot  : IWorkable            { ... }
```

**D - Dependency Inversion**
```csharp
// Bad: high-level module depends on a low-level concrete
class OrderService { private SqlOrderRepo _repo = new SqlOrderRepo(); }

// Good: depend on abstraction - injectable + testable
class OrderService {
    private readonly IOrderRepository _repo;
    public OrderService(IOrderRepository repo) => _repo = repo;
}
```

> **Common Trap:** Interviewers test LSP with Square/Rectangle. Be ready to explain why it violates LSP - setting Width on a Square silently changes its Height, breaking the Rectangle postcondition. The fix: prefer composition, or model them as separate types.

**Quick Revision Checklist - Day 1, Block 1:**
- [ ] Each OOP pillar in one line + one real-world analogy
- [ ] One code violation + fix for each SOLID principle
- [ ] Explain why deep inheritance hierarchies become rigid
- [ ] LSP: Square/Rectangle trap explained fluently

---

### Block 2 - Generics [MUST]

**Core concept:** Type-safe, reusable code. Checked at compile time. No boxing/unboxing overhead for value types.

**Generic class with constraints:**
```csharp
public class Repository<T> where T : class, IEntity, new()
{
    private readonly List<T> _store = new();

    public void Add(T item) => _store.Add(item);
    public IEnumerable<T> GetAll() => _store.AsReadOnly();
    public T? GetById(int id) => _store.FirstOrDefault(x => x.Id == id);
}

// Usage
var repo = new Repository<Product>();
repo.Add(new Product { Id = 1, Name = "Widget" });
```

**Generic method:**
```csharp
public T Deserialize<T>(string json) where T : class
    => JsonSerializer.Deserialize<T>(json)
       ?? throw new InvalidOperationException($"Failed to deserialize {typeof(T).Name}");
```

**Constraints cheat sheet:**

| Constraint | Meaning |
|---|---|
| `where T : class` | Reference type only |
| `where T : struct` | Value type only |
| `where T : new()` | Must have parameterless constructor |
| `where T : IEntity` | Must implement interface |
| `where T : BaseEntity` | Must derive from base class |
| `where T : class, IEntity, new()` | Combined constraints |

**Covariance / Contravariance [BONUS]:**
```csharp
// Covariance (out) - producer: can assign List<Dog> to IEnumerable<Animal>
IEnumerable<Animal> animals = new List<Dog>(); // works - IEnumerable<out T>

// Contravariance (in) - consumer: can assign Action<Animal> to Action<Dog>
Action<Animal> logAnimal = a => Console.WriteLine(a.Name);
Action<Dog>    logDog    = logAnimal; // works - Action<in T>
```

> **Common Trap:** "What is the difference between `List<object>` and `List<T>`?"
> `List<object>` loses type safety: you can add anything, and value types get boxed. `List<T>` is type-checked at compile time and zero-cost for value types.

**Quick Revision Checklist - Day 1, Block 2:**
- [ ] Generic class + method syntax without looking it up
- [ ] All five constraint types
- [ ] `List<object>` vs `List<T>` explanation (type safety + boxing)
- [ ] Real-world: `Repository<T>` eliminates copy-pasted CRUD per entity

---

### Block 3 - LINQ [MUST]

**Core concept:** Declarative, composable querying over `IEnumerable<T>` or `IQueryable<T>`. Lazily evaluated unless materialized.

**Fluent vs query syntax (prefer fluent in interviews):**
```csharp
// Fluent - preferred
var result = orders
    .Where(o => o.Total > 100)
    .OrderBy(o => o.Date)
    .Select(o => new { o.Id, o.Total })
    .ToList();

// Query - equivalent
var result = (from o in orders
              where o.Total > 100
              orderby o.Date
              select new { o.Id, o.Total }).ToList();
```

**High-frequency operators:**
```csharp
var data = new[] { 1, 2, 3, 4, 5, 6 };

data.Where(x => x % 2 == 0)          // filter:    [2,4,6]
data.Select(x => x * x)              // transform: [1,4,9,16,25,36]
data.GroupBy(x => x % 2 == 0)        // group by even/odd
data.SelectMany(x => x.Tags)         // flatten nested collections
data.Any(x => x > 5)                 // true
data.All(x => x > 0)                 // true
data.FirstOrDefault(x => x > 3)      // 4  (null-safe)
data.Distinct()                      // removes duplicates
data.Take(3).Skip(1)                 // [2,3]
data.Sum() / data.Count()            // 21 / 6
data.ToDictionary(x => x, x => x*x) // {1:1, 2:4, ...}
```

**Deferred execution - the key concept:**
```csharp
var query = data.Where(x => x > 2);  // NOT executed - just a description
// Execution happens here:
var list1 = query.ToList();           // materialized
var count = query.Count();            // re-executes the query

// Each .Count() / foreach re-runs the query unless you materialize
var snapshot = query.ToList();        // executed ONCE, stored
```

> **Common Trap:** "What happens if you modify the source collection after building the query but before calling ToList()?" The query sees the modified collection - it has not run yet. Materializing with `.ToList()` / `.ToArray()` takes a snapshot.

**`IEnumerable` vs `IQueryable`:**
```csharp
// IEnumerable - LINQ to Objects (filters in memory)
var result = productList.Where(p => p.Active); // loads all, filters in C#

// IQueryable - LINQ to SQL/EF (SQL translation)
var result = db.Products.Where(p => p.Active); // generates WHERE Active = 1
// The WHERE clause runs on the database, not in C#
```

**Quick Revision Checklist - Day 1, Block 3:**
- [ ] 8+ LINQ operators explained from memory
- [ ] Deferred execution explanation + source-mutation gotcha
- [ ] `IEnumerable` vs `IQueryable` - when SQL translation matters
- [ ] When to materialize (`.ToList()`) and why

---

## DAY 2 - Architecture: Async/Await - DI - Middleware - API Design - Security - Design Patterns

| Time Block | Topic | Priority |
|---|---|---|
| 9:00-10:30 AM | Async/Await + Exception Handling | [MUST] |
| 10:45-12:30 PM | DI + Middleware Pipeline | [MUST] |
| 1:30-3:00 PM | API Design + Security (JWT, XSS, CSRF) | [MUST] |
| 3:15-4:30 PM | Design Patterns | [MUST] |
| 4:30-5:00 PM | Microservices / Event-Driven (high-level) | [BONUS] |
| Evening | Java Coding Practice - Problems 5-8 | [MUST] |

---

### Block 1 - Async/Await + Exception Handling [MUST]

**Concept:** Non-blocking I/O. The thread returns to the thread pool while awaiting, improving throughput without spawning threads.

```csharp
public async Task<Order> GetOrderAsync(int id, CancellationToken ct = default)
{
    var order = await _repo.FindAsync(id, ct);
    if (order is null) throw new NotFoundException($"Order {id} not found");
    return order;
}
```

**Anti-patterns - the short list interviewers probe:**

| Anti-Pattern | Problem | Fix |
|---|---|---|
| `async void` | Exceptions are unobserved; cannot be awaited | `async Task` always (except event handlers) |
| `.Result` / `.Wait()` | Deadlocks in ASP.NET sync context | Always `await` |
| `async` without `await` | Runs synchronously anyway; compiler warning | Remove `async` or add `await` |
| Not threading `CancellationToken` | Cannot cancel I/O mid-flight | Pass `ct` through all layers |
| Sequential I/O that could be parallel | Poor throughput | `Task.WhenAll()` |

```csharp
// Sequential (slow - waits for each before starting the next)
var order = await GetOrderAsync(id, ct);
var user  = await GetUserAsync(userId, ct);

// Parallel (start both, await both - 2 I/O calls, 1 round-trip wait)
var orderTask = GetOrderAsync(id, ct);
var userTask  = GetUserAsync(userId, ct);
await Task.WhenAll(orderTask, userTask);
var order = await orderTask;
var user  = await userTask;
```

**Exception Handling - Senior Approach:**
```csharp
// Do not try-catch in every action - use global middleware
// .NET 8: IExceptionHandler or UseExceptionHandler

// Program.cs
builder.Services.AddProblemDetails();
app.UseExceptionHandler();  // catches unhandled exceptions, returns RFC 7807 ProblemDetails

// Domain exceptions (no stack trace noise, clean HTTP mapping)
public class NotFoundException : Exception {
    public NotFoundException(string msg) : base(msg) { }
}

// Controller stays clean
[HttpGet("{id}")]
public async Task<ActionResult<OrderDto>> Get(int id, CancellationToken ct)
{
    var order = await _service.GetOrderAsync(id, ct); // let exceptions bubble
    return Ok(order);
}
```

> **Common Trap:** "Should you try-catch in every controller action?" Senior answer: No. Use global exception handling middleware (`ProblemDetails` / `IExceptionHandler` in .NET 8). Add local try-catch only when you need a specific recovery path (e.g., retry on `TimeoutException`).

**Quick Revision Checklist - Day 2, Block 1:**
- [ ] `async void` vs `async Task` - why void is dangerous
- [ ] `.Result` deadlock path explained concisely
- [ ] `Task.WhenAll` pattern for parallel I/O
- [ ] `CancellationToken` - thread it through every layer
- [ ] Global exception handler vs local try-catch - which and when

---

### Block 2 - Dependency Injection + Middleware [MUST]

**DI in one sentence:** "Register implementations against abstractions; the container resolves and injects them. Keeps classes decoupled from construction."

**Service lifetimes:**

| Lifetime | Created | Destroyed | Use For |
|---|---|---|---|
| `Singleton` | App start | App stop | Stateless services, caches, config |
| `Scoped` | Per HTTP request | End of request | `DbContext`, unit-of-work, current-user |
| `Transient` | Per injection point | After use | Lightweight, stateless operations |

```csharp
// Program.cs
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();

// Factory registration (conditional)
builder.Services.AddScoped<IPaymentGateway>(sp =>
    sp.GetRequiredService<IConfiguration>()["Payment:Provider"] == "stripe"
        ? sp.GetRequiredService<StripeGateway>()
        : sp.GetRequiredService<PayPalGateway>());
```

> **Common Trap - Captive Dependency:** A `Singleton` holding a `Scoped` service. The Singleton is never disposed, so the `DbContext` instance is reused across requests - stale data, thread-safety issues. Fix: inject `IServiceScopeFactory` and create a scope manually when needed.

```csharp
// Bad
services.AddSingleton<IBackgroundWorker>(sp => {
    var db = sp.GetRequiredService<AppDbContext>(); // DbContext is Scoped!
    return new BackgroundWorker(db);
});

// Good
services.AddSingleton<IBackgroundWorker>(sp =>
    new BackgroundWorker(sp.GetRequiredService<IServiceScopeFactory>()));
// Inside BackgroundWorker:
using var scope = _factory.CreateScope();
var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
```

**Middleware Pipeline:**
```
Request  -->  HTTPS Redirection
         -->  Static Files
         -->  Routing
         -->  Authentication   <- "Who are you?"
         -->  Authorization    <- "Are you allowed?"
         -->  Custom Middleware
         -->  Endpoints / Controllers
Response <--  (flows back in reverse order)
```

Order is everything. `UseAuthentication()` must come before `UseAuthorization()`. `UseRouting()` before `UseAuthorization()`.

**Custom Middleware:**
```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next,
        ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation("-> {Method} {Path}",
            context.Request.Method, context.Request.Path);

        await _next(context); // call next middleware in pipeline

        _logger.LogInformation("<- {StatusCode}", context.Response.StatusCode);
    }
}

// Register
app.UseMiddleware<RequestLoggingMiddleware>();
```

**Quick Revision Checklist - Day 2, Block 2:**
- [ ] All three lifetimes + correct use case for each
- [ ] Captive dependency: what it is, how it causes bugs, how to fix it
- [ ] Middleware pipeline order (Auth before Authz) - why order matters
- [ ] Custom middleware structure: `InvokeAsync`, `RequestDelegate _next`

---

### Block 3 - API Design + Security [MUST]

**REST Conventions:**

| Action | Verb | Route | Success Status |
|---|---|---|---|
| Get all | `GET` | `/api/orders` | `200 OK` |
| Get one | `GET` | `/api/orders/{id}` | `200` / `404` |
| Create | `POST` | `/api/orders` | `201 Created` + `Location` header |
| Full update | `PUT` | `/api/orders/{id}` | `200` / `204` |
| Partial update | `PATCH` | `/api/orders/{id}` | `200` / `204` |
| Delete | `DELETE` | `/api/orders/{id}` | `204 No Content` |

```csharp
[HttpPost]
public async Task<ActionResult<OrderDto>> Create([FromBody] CreateOrderRequest req,
    CancellationToken ct)
{
    var order = await _service.CreateAsync(req, ct);
    return CreatedAtAction(nameof(Get), new { id = order.Id }, order);
}

[HttpGet("{id}")]
public async Task<ActionResult<OrderDto>> Get(int id, CancellationToken ct)
{
    var order = await _service.GetAsync(id, ct);
    return order is null ? NotFound() : Ok(order);
}
```

**Filters vs Middleware:**
- **Middleware:** App-wide, operates on raw `HttpContext`, runs before the MVC layer
- **Filters:** Within MVC pipeline (`ActionFilter`, `ExceptionFilter`, `AuthorizationFilter`) - useful for controller-scoped cross-cutting concerns (validation, timing, audit logging)

---

**JWT Authentication Flow:**
```
1. POST /auth/login  (credentials)
2. Server validates, creates JWT:
   Header.Payload.Signature
   Header:  { "alg": "HS256", "typ": "JWT" }
   Payload: { "sub": "u1", "role": "Admin", "exp": 1711234567 }
   Sig:     HMACSHA256(base64(header)+"."+base64(payload), secret)
3. Client stores JWT (httpOnly cookie - safer than localStorage)
4. Client sends: Authorization: Bearer <token>
5. Server validates signature + expiry on every request
```

```csharp
// Startup - JWT bearer setup
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(opt => {
        opt.TokenValidationParameters = new() {
            ValidateIssuer           = true,
            ValidateAudience         = true,
            ValidateLifetime         = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer       = config["Jwt:Issuer"],
            ValidAudience     = config["Jwt:Audience"],
            IssuerSigningKey  = new SymmetricSecurityKey(
                                    Encoding.UTF8.GetBytes(config["Jwt:Key"]))
        };
    });

// Usage
[Authorize(Roles = "Admin")]
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(int id, CancellationToken ct) { ... }
```

**Authentication vs Authorization:**
- **Authentication:** "Who are you?" - validate identity (JWT, cookie, API key)
- **Authorization:** "What can you do?" - validate permissions (roles, policies, claims)
- In ASP.NET: `app.UseAuthentication()` -> `app.UseAuthorization()` - order matters

---

**XSS vs CSRF:**

| | XSS (Cross-Site Scripting) | CSRF (Cross-Site Request Forgery) |
|---|---|---|
| Attack | Inject malicious script into your page | Trick user's browser into making authenticated requests |
| Target | The victim's browser / session | The victim's authenticated server session |
| Defense | Content-Security-Policy, encode output, sanitize input | Antiforgery tokens, `SameSite=Strict` cookies |
| JWT risk | HIGH if stored in `localStorage` (JS can read it) | LOW (JWT not auto-sent by browser) |

**JWT: Cookie vs localStorage:**

| | localStorage | httpOnly Cookie |
|---|---|---|
| XSS risk | High (JS can access) | Low (JS cannot access httpOnly) |
| CSRF risk | Low (not auto-sent) | Medium (need SameSite + antiforgery) |
| Server sessions | Not needed | Not needed |
| Recommendation | Avoid for JWTs | Prefer for web apps |

> **Common Trap:** "Is JWT stateless?" The token itself is stateless - no server-side session. But **revocation is the problem**: you cannot invalidate a JWT before it expires without a server-side blocklist - which re-introduces statefulness. Senior answer: short expiry (15 min) + rotating refresh tokens, or a revocation store (Redis).

**Quick Revision Checklist - Day 2, Block 3:**
- [ ] REST verbs + correct status codes by memory
- [ ] JWT structure: header, payload, signature - what is in each
- [ ] Authentication vs Authorization - one-line distinction
- [ ] XSS vs CSRF - what each is, how each is prevented
- [ ] JWT revocation problem + practical solutions
- [ ] `UseAuthentication` before `UseAuthorization` in pipeline

---

### Block 4 - Design Patterns [MUST] + Microservices [BONUS]

**Repository Pattern [MUST]:**
```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id, CancellationToken ct = default);
    Task<IEnumerable<Order>> GetAllAsync(CancellationToken ct = default);
    Task AddAsync(Order order, CancellationToken ct = default);
    Task SaveChangesAsync(CancellationToken ct = default);
}

public class EfOrderRepository : IOrderRepository
{
    private readonly AppDbContext _db;
    public EfOrderRepository(AppDbContext db) => _db = db;

    public async Task<Order?> GetByIdAsync(int id, CancellationToken ct = default)
        => await _db.Orders.FindAsync(new object[] { id }, ct);
    // ...
}
```

**Factory Pattern [MUST]:**
```csharp
public interface INotificationSender { Task SendAsync(string message); }
public class EmailSender : INotificationSender { ... }
public class SmsSender   : INotificationSender { ... }

public class NotificationFactory
{
    public INotificationSender Create(string channel) => channel switch {
        "email" => new EmailSender(),
        "sms"   => new SmsSender(),
        _       => throw new ArgumentException($"Unknown channel: {channel}")
    };
}
```

**Decorator Pattern [BONUS] - great for caching layer:**
```csharp
public class CachingOrderRepository : IOrderRepository
{
    private readonly IOrderRepository _inner;
    private readonly IMemoryCache _cache;

    public CachingOrderRepository(IOrderRepository inner, IMemoryCache cache)
    {
        _inner = inner;
        _cache = cache;
    }

    public async Task<Order?> GetByIdAsync(int id, CancellationToken ct = default)
    {
        if (_cache.TryGetValue($"order:{id}", out Order? cached)) return cached;
        var order = await _inner.GetByIdAsync(id, ct);
        _cache.Set($"order:{id}", order, TimeSpan.FromMinutes(5));
        return order;
    }
}
```

**Events + Delegates [MUST]:**
```csharp
public class OrderService
{
    // Event wraps delegate - restricts external invocation (they can subscribe but not fire it)
    public event EventHandler<Order>? OrderPlaced;

    public void PlaceOrder(Order order)
    {
        _repo.Save(order);
        OrderPlaced?.Invoke(this, order); // notify all subscribers
    }
}

// Subscriber
orderService.OrderPlaced += (sender, order) => emailService.Notify(order);
```

> **Common Trap:** "What is the difference between a delegate and an event?"
> A delegate is a raw function pointer - anyone can invoke it. An `event` is a delegate with access modifiers enforced: outside subscribers can only `+=` / `-=`, not invoke it directly. Events enforce the publisher/subscriber encapsulation.

**Microservices (High-Level) [BONUS]:**

Key vocabulary you should be able to discuss fluently:
- **API Gateway:** Single entry point, routes to downstream services
- **Circuit Breaker:** Fail fast when a downstream service is unhealthy (Polly in .NET)
- **Outbox Pattern:** Write event to DB alongside business data atomically; background job publishes to message bus - ensures at-least-once delivery
- **Saga Pattern:** Distributed transactions via compensating actions
- **Sync vs Async comms:** HTTP/REST (sync, tight coupling) vs message bus/RabbitMQ/Azure Service Bus (async, loose coupling)

**Quick Revision Checklist - Day 2, Block 4:**
- [ ] `Repository<T>` - interface + EF implementation from memory
- [ ] Factory pattern with `switch` expression syntax
- [ ] Decorator for caching/logging without changing the inner class
- [ ] Event vs delegate - external invocation restriction
- [ ] 3 microservices patterns named and described in one sentence each

---

## DAY 3 - Mock Day

| Time Block | Activity |
|---|---|
| 9:00-9:30 AM | SQL Optimization Quick Review |
| 9:30-10:00 AM | Day 1 + 2 rapid flashcard pass |
| 10:00-10:45 AM | Simulated 30-min Technical Q&A |
| 11:00-11:45 AM | Timed 30-min Coding Problem |
| 12:00-12:30 PM | Debrief + weak area patch |
| Afternoon | Java Practice Problems 9-10 + Behavior Cheat Sheet review |

---

### SQL Optimization - Senior Quick Notes

```sql
-- Index on a frequently filtered / joined column
CREATE INDEX idx_orders_customer ON Orders(CustomerId);

-- Covering index: includes all projected columns -> avoids heap lookup
CREATE INDEX idx_orders_status ON Orders(Status, OrderDate) INCLUDE (CustomerId, Total);

-- Never apply functions to indexed columns in WHERE - defeats the index
WHERE YEAR(OrderDate) = 2024              -- BAD: full table scan
WHERE OrderDate >= '2024-01-01'
  AND OrderDate <  '2025-01-01'          -- GOOD: range scan on index

-- N+1 query problem (EF)
var orders = db.Orders.ToList();
foreach (var o in orders)
    _ = o.Customer.Name;   // N additional queries

// Fix: eager load
var orders = db.Orders.Include(o => o.Customer).ToList();
```

> **Common Trap:** "What types of indexes are there?"
> Clustered (physically sorts the table - one per table, usually PK) vs Non-Clustered (separate structure with pointer to row - many per table). Covering index includes all needed columns so no row lookup is needed.

---

### Simulated 30-Minute Technical Q&A

*Answer each before reading the model answer.*

---

**Q1.** Walk me through the SOLID principles with a real example for each.
> Cover all 5 with a one-line definition + a concrete C# scenario. Tie each to a design decision, not just a textbook definition.

**Q2.** What is the difference between Scoped, Transient, and Singleton? What is a captive dependency?
> Lifetimes + the Scoped-inside-Singleton problem. Mention `IServiceScopeFactory` as the fix. Mention `DbContext` as the canonical Scoped example.

**Q3.** How does JWT work in ASP.NET Core? How do you revoke a JWT?
> Walk through the full flow (issue -> store -> send -> validate). Then address revocation: stateless token cannot be invalidated before expiry without a server-side blocklist. Practical solution: short expiry + refresh token rotation.

**Q4.** What is deferred execution in LINQ? Give me an example where it causes a real bug.
> Lazy evaluation - query runs when enumerated. Bug: enumerating after `DbContext` is disposed (common in async code where the scope ends before the iterator is consumed). Fix: materialize with `.ToList()` inside the using scope.

**Q5.** What is the difference between `async void` and `async Task`? When is `async void` acceptable?
> `async void` cannot be awaited and exceptions are unobserved - they crash the process. Only acceptable for event handlers (which require `void` by framework contract - e.g., button click).

**Q6.** Explain the middleware pipeline. Why does order matter?
> Request flows through in registration order, response flows back in reverse. Auth before Authz: you cannot check permissions on an unauthenticated identity. Static files before routing: serve static content without going through the MVC stack.

**Q7.** What is the difference between authentication and authorization in ASP.NET Core?
> Auth = identity ("who are you" - JWT validation). Authz = permissions ("what can you do" - roles/policies). `app.UseAuthentication()` before `app.UseAuthorization()`. Use `[Authorize(Roles="Admin")]` or `[Authorize(Policy="CanDelete")]`.

**Q8.** How do generics improve performance over `ArrayList` / `List<object>`?
> No boxing/unboxing for value types. Compile-time type safety eliminates runtime casts. JIT generates specialized code per type for structs. `ArrayList.Add(42)` boxes the int; `List<int>.Add(42)` does not.

**Q9.** What design patterns do you rely on regularly?
> Repository (persistence abstraction + testability), Factory (polymorphic object creation without the caller knowing the concrete type), Decorator (add cross-cutting concerns - caching, logging - without modifying the original class). Tie to a real project if possible.

**Q10.** Tell me about a technical trade-off you made on a project.
> Prepare one concrete story: problem context -> options considered -> what you chose -> why -> outcome. Frame around design decisions (e.g., chose eventual consistency over strong consistency to gain scalability, or chose synchronous HTTP over a message bus for simplicity at the cost of tight coupling).

---

### Timed 30-Minute Coding Problem (Java)

**Problem: Meeting Rooms II**

> Given an array of meeting time intervals `intervals[i] = [start, end]`, return the minimum number of conference rooms required.

**Step 1 - Clarify (say this out loud before coding):**
- "Are both bounds inclusive?"
- "Can the array be empty? Should I return 0?"
- "Can `start == end` - is that a zero-duration meeting, and does it conflict with another meeting at the same time?"
- "Are there any constraints on the value range for start/end?"

**Step 2 - Think aloud:**
> "If I separate all start times and sort them, and separately sort all end times, I can traverse them like a timeline. When I see a start, I need a room. When I see an end, a room becomes free. If the earliest end time is <= the current start, a room freed up - I do not need a new one. The max concurrent rooms at any point is my answer."

**Solution:**
```java
import java.util.Arrays;

class Solution {
    public int minMeetingRooms(int[][] intervals) {
        if (intervals == null || intervals.length == 0) return 0;

        int n = intervals.length;
        int[] starts = new int[n];
        int[] ends   = new int[n];

        for (int i = 0; i < n; i++) {
            starts[i] = intervals[i][0];
            ends[i]   = intervals[i][1];
        }

        Arrays.sort(starts);
        Arrays.sort(ends);

        int rooms = 0, maxRooms = 0;
        int s = 0, e = 0;

        while (s < n) {
            if (starts[s] < ends[e]) {  // new meeting starts before any ends
                rooms++;
                s++;
            } else {                     // a room freed up
                rooms--;
                e++;
            }
            maxRooms = Math.max(maxRooms, rooms);
        }
        return maxRooms;
    }
}
```

**Complexity:** O(n log n) time, O(n) space

**Debrief - check yourself:**
- Did you clarify before writing a line?
- Did you explain the two-array sort approach before coding it?
- Did you handle `null` / empty input?
- Can you explain why `starts[s] < ends[e]` uses strict `<`? (Back-to-back: meeting ending at 10, next starting at 10 - that is NOT an overlap, same room can be reused)
- Alternative: min-heap on end times - `PriorityQueue<Integer>`, O(n log n) but more conventional to explain

---

## Java Practice Problems (Problems 1-10)

---

### Problem 1 - K Closest Points to Origin [Arrays/Sorting]
**Problem:** Return the `k` closest points to origin `(0,0)` from an array of `[x,y]` points.

```java
import java.util.Arrays;

class Solution {
    public int[][] kClosest(int[][] points, int k) {
        Arrays.sort(points, (a, b) ->
            (a[0]*a[0] + a[1]*a[1]) - (b[0]*b[0] + b[1]*b[1]));
        return Arrays.copyOfRange(points, 0, k);
    }
}
```
**O(n log n) time - O(1) space**

**Follow-ups:**
1. "Can you do it in O(n) average?" -> QuickSelect - partial sort without sorting the whole array
2. "What if the input is a live stream?" -> Max-heap of size k - evict the farthest when heap exceeds k
3. "Why not use Math.sqrt?" -> Avoid it - comparing squared distances is sufficient and avoids floating-point error

---

### Problem 2 - Two Sum [HashMap]
**Problem:** Return indices of two numbers that add to `target`.

```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> seen = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (seen.containsKey(complement))
                return new int[]{ seen.get(complement), i };
            seen.put(nums[i], i);
        }
        return new int[]{};
    }
}
```
**O(n) time - O(n) space**

**Follow-ups:**
1. "What if you need all pairs, not just one?" -> Continue instead of returning; collect results
2. "What if the input is sorted?" -> Two pointers: O(n) time O(1) space
3. "What if values overflow int?" -> Use `long` arithmetic

---

### Problem 3 - Longest Substring Without Repeating Characters [Sliding Window]
**Problem:** Find the length of the longest substring with all distinct characters.

```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int lengthOfLongestSubstring(String s) {
        Map<Character, Integer> lastSeen = new HashMap<>();
        int maxLen = 0, left = 0;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            if (lastSeen.containsKey(c) && lastSeen.get(c) >= left) {
                left = lastSeen.get(c) + 1;  // shrink window past the duplicate
            }
            lastSeen.put(c, right);
            maxLen = Math.max(maxLen, right - left + 1);
        }
        return maxLen;
    }
}
```
**O(n) time - O(min(n,m)) space** (m = alphabet size)

**Follow-ups:**
1. "What is the window invariant?" -> `[left, right]` contains no duplicates
2. "Only lowercase letters?" -> Replace HashMap with `int[26]`
3. "Trace 'abcabcbb'" -> a(1), ab(2), abc(3), bca(reset, 3), cab(3), abc(3), bc(2), b(1) -> answer: 3

---

### Problem 4 - Valid Anagram [HashMap/Frequency]
**Problem:** Return true if `t` is an anagram of `s`.

```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) return false;
        int[] freq = new int[26];
        for (char c : s.toCharArray()) freq[c - 'a']++;
        for (char c : t.toCharArray()) freq[c - 'a']--;
        for (int f : freq) if (f != 0) return false;
        return true;
    }
}
```
**O(n) time - O(1) space**

**Follow-ups:**
1. "What if strings include Unicode?" -> Use `HashMap<Character, Integer>` instead of `int[26]`
2. "What is the sort-based approach?" -> Sort both -> compare; O(n log n) time, simpler code
3. "Extend to group anagrams from a list" -> Sorted string as HashMap key -> see Problem 8

---

### Problem 5 - Move Zeroes [Two Pointers]
**Problem:** Move all 0s to the end in-place, maintaining relative order of non-zeros.

```java
class Solution {
    public void moveZeroes(int[] nums) {
        int insertPos = 0;
        for (int num : nums)
            if (num != 0) nums[insertPos++] = num;
        while (insertPos < nums.length)
            nums[insertPos++] = 0;
    }
}
```
**O(n) time - O(1) space**

**Follow-ups:**
1. "Does this preserve relative order?" -> Yes - non-zeros are written to `insertPos` in original order
2. "Minimize writes?" -> Swap approach: only swap when `nums[left] == 0` and `nums[right] != 0`
3. "One pass with swaps?" -> `if (nums[i] != 0) swap(nums[insertPos++], nums[i])`

---

### Problem 6 - Coin Change [DP - ATM Denomination Pattern]
**Problem:** Given coin denominations and an amount, find the minimum number of coins. Return -1 if impossible.

```java
import java.util.Arrays;

class Solution {
    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1);  // sentinel: larger than any valid answer
        dp[0] = 0;

        for (int i = 1; i <= amount; i++)
            for (int coin : coins)
                if (coin <= i)
                    dp[i] = Math.min(dp[i], dp[i - coin] + 1);

        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```
**O(amount x |coins|) time - O(amount) space**

**Follow-ups:**
1. "Why not greedy (largest coin first)?" -> Fails for `coins=[1,3,4], amount=6` - greedy gives 4+1+1=3 coins; DP finds 3+3=2 coins
2. "What does `dp[i]` represent?" -> Minimum coins to make exactly amount `i`
3. "How would you reconstruct which coins were used?" -> Track a `parent[]` or `choice[]` array alongside `dp`

---

### Problem 7 - Find All Duplicates in Array [Arrays - Index as Hash]
**Problem:** In array of n integers where 1 <= nums[i] <= n, find all elements appearing twice. O(n) time, O(1) space.

```java
import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<Integer> findDuplicates(int[] nums) {
        List<Integer> result = new ArrayList<>();
        for (int i = 0; i < nums.length; i++) {
            int idx = Math.abs(nums[i]) - 1;
            if (nums[idx] < 0) {
                result.add(idx + 1);  // seen before - it is a duplicate
            } else {
                nums[idx] = -nums[idx];  // mark as visited
            }
        }
        return result;
    }
}
```
**O(n) time - O(1) space** (modifies input)

**Follow-ups:**
1. "Why does negation work here?" -> Values in [1,n] map directly to indices; negating marks "visited" without extra space
2. "What if values include 0?" -> 0 cannot be negated; clarify constraints upfront
3. "Can you restore the array?" -> Iterate and `Math.abs()` all elements

---

### Problem 8 - Group Anagrams [HashMap]
**Problem:** Given an array of strings, group anagrams together.

```java
import java.util.*;

class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<>();
        for (String s : strs) {
            char[] chars = s.toCharArray();
            Arrays.sort(chars);
            String key = new String(chars);
            map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
        }
        return new ArrayList<>(map.values());
    }
}
```
**O(n x k log k) time - O(n x k) space** (k = max string length)

**Follow-ups:**
1. "Can you avoid sorting?" -> Use character frequency array as key: "a2b1..." -> O(n x k) time
2. "What does `computeIfAbsent` do?" -> If key absent, creates a new list, puts it, and returns it - avoids null check boilerplate
3. "Very long strings?" -> Frequency-based key is faster (O(k) vs O(k log k))

---

### Problem 9 - Subarray Sum Equals K [Prefix Sum + HashMap]
**Problem:** Count subarrays that sum to `k`.

```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int subarraySum(int[] nums, int k) {
        Map<Integer, Integer> prefixCount = new HashMap<>();
        prefixCount.put(0, 1);  // empty prefix has sum 0
        int sum = 0, count = 0;

        for (int num : nums) {
            sum += num;
            count += prefixCount.getOrDefault(sum - k, 0);
            prefixCount.merge(sum, 1, Integer::sum);
        }
        return count;
    }
}
```
**O(n) time - O(n) space**

**Follow-ups:**
1. "Walk me through the logic." -> If `prefix[j] - prefix[i] == k`, then subarray `[i+1..j]` sums to k. Count how many previous prefix sums equal `currentSum - k`
2. "Does this work with negative numbers?" -> Yes - unlike the sliding window, prefix sum handles negatives
3. "Why initialize `{0:1}`?" -> Handles subarrays starting from index 0 (no prior prefix to subtract)

---

### Problem 10 - Container With Most Water [Two Pointers]
**Problem:** Given heights, find two lines forming the container with maximum water.

```java
class Solution {
    public int maxArea(int[] height) {
        int left = 0, right = height.length - 1, maxWater = 0;

        while (left < right) {
            int water = Math.min(height[left], height[right]) * (right - left);
            maxWater = Math.max(maxWater, water);

            if (height[left] < height[right]) left++;
            else right--;
        }
        return maxWater;
    }
}
```
**O(n) time - O(1) space**

**Follow-ups:**
1. "Why move the shorter side?" -> Moving the taller side can only decrease width with no chance of increasing height (bounded by the shorter side) - it is strictly non-optimal
2. "Does greedy movement ever miss the optimal?" -> No - if we skipped moving the shorter pointer, any new container with it fixed is also bounded by the same short height but with less width
3. "Brute force?" -> O(n^2) - check all pairs

---

## Coding Interview Behavior Cheat Sheet

### Before Writing a Single Line - Clarification Framework (CITE)

| Letter | What to ask |
|---|---|
| **C - Constraints** | Input size? Can values be negative? Empty array? Integer range (int overflow possible)? |
| **I - Input format** | Sorted? Unique values or duplicates? Graph directed/undirected? |
| **T - Target output** | Return indices or values? Any order acceptable? Single answer or all answers? |
| **E - Edge cases** | `k > n`? No valid answer - return -1 or empty? `null` input? Single element? |

**Script (say this aloud):**
> "Let me make sure I understand. Given [restate in your words], I need to return [output]. Before I start - a few clarifying questions: [ask 2-3 CITE questions]. Okay, with that, let me think through the approach."

---

### Thought Process Structure: Observe -> Approach -> Optimize -> Code

```
1. Observe:  "I notice that [key property - sorted, bounded values, monotonic, etc.]"
2. Approach: "Brute force would be O(n^2) because [reason].
              I can improve this using [technique] because [property enables it]."
3. Optimize: "That gives O(n) time / O(n) space.
              Can I reduce space? Let me think..."
4. Code:     [Narrate as you type]
             "I am using a HashMap for O(1) lookup instead of a linear scan."
             "I am initializing left=0 because the window starts empty."
             "This edge case at line X handles the empty input."
```

---

### Issue-Finding Framework (Given Buggy Code)

Go through this checklist systematically:

1. **Off-by-one:** `i < n` vs `i <= n`? Accessing `i+1` or `i-1` at array boundaries?
2. **Null / empty:** Is there a null check? What happens with an empty array or empty string?
3. **Integer overflow:** `a * b` where both are large ints -> should be `(long) a * b`
4. **Integer division:** `3 / 2 == 1` not `1.5` -> `(double) 3 / 2`
5. **Mutation of input:** Is the input modified when it should not be (caller's data corrupted)?
6. **Logic errors:** `&&` vs `||`? `<` vs `<=`? Is the condition testing what they intended?
7. **Unhandled cases:** Single element? All same elements? All zeros? k=0? k=n? Already sorted?

**How to communicate it:**
> "I see a potential issue on this line - what the code does here is [describe]. In the case where [specific input], this would [wrong outcome]. The fix would be to [correction]."

---

### Common Java Pitfalls

| Pitfall | Bad | Fix |
|---|---|---|
| Midpoint overflow | `(left + right) / 2` | `left + (right - left) / 2` |
| Multiplication overflow | `a * b` (both int) | `(long) a * b` |
| Integer division | `3 / 2 == 1` | `(double) 3 / 2` or `3.0 / 2` |
| String equality | `str1 == str2` | `str1.equals(str2)` |
| NPE on map lookup | `map.get(k).toString()` | `map.getOrDefault(k, 0)` |
| Modifying list while iterating | `for` + `list.remove()` | Collect removals, apply after |
| `char` subtraction | Forgetting `char` is unsigned | `c - 'a'` is fine; be careful with negatives |
| `Arrays.asList` is fixed-size | `Arrays.asList(...).add(x)` throws | Use `new ArrayList<>(Arrays.asList(...))` |

---

## Java Quick Reference Sheet

### Sorting

```java
// Primitive array - ascending
Arrays.sort(arr);

// Sort descending (requires Integer[], not int[])
Integer[] arr = { 3, 1, 2 };
Arrays.sort(arr, Comparator.reverseOrder());

// Sort 2D array by first element
Arrays.sort(points, (a, b) -> a[0] - b[0]);

// Sort by distance (float comparison - use Double.compare, not subtraction)
Arrays.sort(points, (a, b) -> Double.compare(
    a[0]*a[0] + a[1]*a[1],
    b[0]*b[0] + b[1]*b[1]));

// Sort List
list.sort(Comparator.comparingInt(String::length)
                    .thenComparing(Comparator.naturalOrder()));
```

### Priority Queue (Heap)

```java
// Min-heap (default) - smallest polled first
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(3); minHeap.offer(1);
minHeap.peek();   // 1 (no remove)
minHeap.poll();   // 1 (removes)

// Max-heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());

// Min-heap of int[] by first element (K Closest pattern)
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
pq.offer(new int[]{ dist, index });
int[] top = pq.poll();

// Max-heap of int[] - to keep k smallest, evict the largest
PriorityQueue<int[]> maxPQ = new PriorityQueue<>((a, b) -> b[0] - a[0]);
// if (maxPQ.size() > k) maxPQ.poll(); // evict farthest
```

### HashMap / HashSet Patterns

```java
Map<String, Integer> map = new HashMap<>();

map.getOrDefault(key, 0);                          // safe get
map.put(key, map.getOrDefault(key, 0) + 1);       // increment
map.merge(key, 1, Integer::sum);                   // cleaner increment
map.computeIfAbsent(key, k -> new ArrayList<>()).add(val);  // group into lists

// Iterate
for (Map.Entry<String, Integer> e : map.entrySet())
    System.out.println(e.getKey() + ": " + e.getValue());

// HashSet
Set<Integer> seen = new HashSet<>();
seen.add(x);
seen.contains(x);  // O(1)
seen.remove(x);
```

### Streams (Java) <-> LINQ (C#)

| LINQ (C#) | Java Stream |
|---|---|
| `.Where(x => x > 0)` | `.filter(x -> x > 0)` |
| `.Select(x => x * 2)` | `.map(x -> x * 2)` |
| `.SelectMany(x => x.Items)` | `.flatMap(x -> x.items.stream())` |
| `.OrderBy(x => x)` | `.sorted()` |
| `.OrderByDescending(x => x)` | `.sorted(Comparator.reverseOrder())` |
| `.ToList()` | `.collect(Collectors.toList())` |
| `.FirstOrDefault()` | `.findFirst().orElse(null)` |
| `.Any(x => x > 0)` | `.anyMatch(x -> x > 0)` |
| `.All(x => x > 0)` | `.allMatch(x -> x > 0)` |
| `.Count()` | `.count()` |
| `.Sum()` | `.mapToInt(x -> x).sum()` |
| `.GroupBy(x => x.Dept)` | `.collect(Collectors.groupingBy(x -> x.dept))` |
| `.Skip(n).Take(m)` | `.skip(n).limit(m)` |
| `.Distinct()` | `.distinct()` |
| `.ToDictionary(k, v)` | `.collect(Collectors.toMap(k, v))` |

```java
// Example: sum of squares of even numbers
int result = IntStream.rangeClosed(1, 10)
    .filter(x -> x % 2 == 0)
    .map(x -> x * x)
    .sum();

// Example: group words by first letter
Map<Character, List<String>> groups = words.stream()
    .collect(Collectors.groupingBy(w -> w.charAt(0)));
```

### Math Utilities

```java
Math.abs(-5)          // 5
Math.max(3, 7)        // 7
Math.min(3, 7)        // 3
Math.pow(2, 10)       // 1024.0  (double)
Math.sqrt(16)         // 4.0     (double)
Math.floor(3.7)       // 3.0
Math.ceil(3.2)        // 4.0
Math.round(3.5)       // 4L  (long)
Integer.MAX_VALUE     // 2_147_483_647
Integer.MIN_VALUE     // -2_147_483_648
Long.MAX_VALUE        // 9_223_372_036_854_775_807
(int) Math.sqrt(n)   // cast when using as index
```

---

## Master Revision Checklist (Night Before)

**C#/.NET - must answer these without hesitation:**
- [ ] All 4 OOP pillars + one-line each
- [ ] All 5 SOLID principles + one code violation + fix each
- [ ] Generic class with constraints syntax; `List<T>` vs `List<object>`
- [ ] LINQ deferred execution + `IEnumerable` vs `IQueryable`
- [ ] `async void` pitfall; `.Result` deadlock; `Task.WhenAll` pattern
- [ ] Singleton / Scoped / Transient + captive dependency trap
- [ ] Middleware pipeline order; custom middleware structure
- [ ] JWT flow + revocation problem; XSS vs CSRF; localStorage vs httpOnly cookie
- [ ] Repository + Factory + Decorator pattern code sketches

**Java Coding - before every problem:**
- [ ] Clarify: constraints, edge cases, output format
- [ ] Say brute force first, then improve
- [ ] Narrate while typing - every non-trivial line gets a sentence
- [ ] Check edge cases: null/empty, single element, overflow, all same values
