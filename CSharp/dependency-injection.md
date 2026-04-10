# Dependency Injection & Inversion of Control

## In One Line
Instead of a class creating what it needs, it receives its dependencies from the outside — that's DI. The principle behind it (letting someone else control what you use) is called Inversion of Control.

## Core Idea
A class should never `new` up its own dependencies. It should declare what it needs through its constructor and let the caller (or a container) provide it.

This way, the class only knows the shape (the interface), never the substance (the concrete class). The decision of *which* implementation to use lives in one central place — `Program.cs` — not scattered across constructors.

IoC is the principle ("don't control your dependencies"), DI is the technique ("receive them via constructor"), and the DI Container is the tool that wires it all up automatically.

## Why Do We Need It?
Without DI, classes are tightly coupled. If `OrderService` does `new SqlOrderRepository()` inside itself, you can't swap that to Mongo or a fake without editing the class.

DI lets you program against interfaces. Swap `SqlOrderRepository` with `MongoOrderRepository` by changing one line in `Program.cs`. The `OrderService` doesn't change at all.

It also makes unit testing trivial — inject a mock `IOrderRepository` instead of hitting a real database.

## What Goes Wrong Without It?
Every class picks its own dependencies. Changing one implementation means editing every class that uses it.

Testing becomes painful because you can't isolate a class from its real dependencies. You end up needing a database running just to test business logic.

Business rules and infrastructure choices get mixed together. The class that calculates order totals also decides which email provider to use.

## Simple Example
Bad design — class creates its own dependency:

```csharp
public class NotificationService {
    private readonly EmailSender _sender = new EmailSender();
    public void Notify(string msg) => _sender.Send(msg);
}
```

Can't swap `EmailSender` for `SmsSender`. Can't test without sending real emails.

Better design — class asks for what it needs:

```csharp
public class NotificationService {
    private readonly INotificationSender _sender;
    public NotificationService(INotificationSender sender) => _sender = sender;
    public void Notify(string msg) => _sender.Send(msg);
}
```

Now register in `Program.cs`: `builder.Services.AddTransient<INotificationSender, SmtpEmailSender>();`

## How the DI Container Resolves Dependencies
The container uses reflection. It reads the constructor, sees what parameters are needed, looks each one up in its registry, and recursively builds the entire dependency tree from the leaves up.

If `OrderService` needs `IOrderRepository`, and `SqlOrderRepository` needs `AppDbContext`, the container creates `AppDbContext` first, then `SqlOrderRepository(dbContext)`, then `OrderService(repo)`.

If the constructor has a parameter the container can't resolve (like a raw `string`), it throws `InvalidOperationException`. Fix this by using `IOptions<T>` for config or a factory lambda for custom construction.

## Multiple Implementations for One Interface
**Last registered wins:** If you register two implementations, injecting `INotificationSender` gives the last one.

**Get all with `IEnumerable<T>`:** Inject `IEnumerable<INotificationSender>` to get every registered implementation. Each one self-identifies via a property, and calling code picks the right one at runtime. This is the Strategy pattern — adding a new payment method means just creating a class and registering it. No switch statements to update.

**Keyed services (.NET 8+):** Register with `AddKeyedTransient<IPaymentService, UpiPayment>("upi")` and resolve with `[FromKeyedServices("upi")]` or `GetRequiredKeyedService<T>("upi")` at runtime.

## Strong Mental Model
Ask yourself: "Who decides which implementation this class uses — the class itself, or someone outside?"

If the class decides, you have tight coupling. If someone outside decides, you have DI.

## Interview-Ready Definition
Dependency Injection is a technique where a class receives its dependencies through its constructor instead of creating them. The DI Container in .NET automatically resolves the full dependency graph using reflection, building objects from the leaves up. Inversion of Control is the underlying principle — the class gives up control of *what* it depends on, and the composition root (`Program.cs`) makes that decision.

## Interview-Ready Why
DI gives us loose coupling, easy testability, and a single place to configure all wiring. Without it, classes are glued to specific implementations, making the codebase rigid and hard to test.

## Common Interview Questions
**What's the difference between IoC and DI?**
IoC is the principle (don't control your dependencies). DI is the technique (receive them via constructor). DI is one way to achieve IoC.

**How does the DI container know what to inject?**
It reads the constructor via reflection, matches each parameter type to its registry, and recursively resolves the entire graph.

**What if two implementations are registered for the same interface?**
The last one wins for single injection. Use `IEnumerable<T>` to get all, or keyed services (.NET 8+) for named resolution.

**Can you resolve different implementations at runtime?**
Yes — use a factory delegate (`Func<string, IPaymentService>`), the Strategy pattern with `IEnumerable<T>`, or keyed services with `GetRequiredKeyedService`.

## Fast Recall
Think: Class says "give me what I need" — not "I'll make what I need."
Think: Constructor = wishlist. Container = Santa.
Think: One registration line in Program.cs replaces every `new` scattered across the codebase.

## Questions To Ask In Design
Should this class know about the concrete implementation, or just an interface?
Where does the decision of which implementation to use belong?
Can I test this class in isolation without real infrastructure?
Would adding a new implementation require changing existing code?
