# Abstraction

## In One Line
Abstraction means showing only what the user needs to use, while hiding the internal complexity of how it is done.

## Core Idea
The user of an object should focus on what it does, not how it does it.

In simple words, abstraction gives a clean and simple way to use something complex.

## Why Do We Need It?
We need abstraction because real systems are full of internal details, and callers should not have to deal with all of them.

For example, if you use a payment service, you should just say `pay(amount)`. You should not have to know about gateway retries, token generation, fraud checks, or settlement logic.

Abstraction reduces mental overload, keeps responsibilities separate, and makes code easier to extend and maintain.

## What Goes Wrong Without It?
Without abstraction, low-level details leak into other parts of the code.

Then callers start knowing too much. They may repeat internal steps, misuse the flow, or become tightly coupled to implementation details.

This makes the system harder to read, harder to change, and easier to break.

## Simple Example
Bad design:

```java
class EmailService {
    public void connectToSMTP() {}
    public void authenticate() {}
    public void buildBody() {}
    public void sendEmail(String to, String body) {}
}
```

If callers have to manage all these steps, they now depend on internal details.

Better design:

```java
class NotificationService {
    public void send(String to, String message) {
        // handles connection, auth, formatting, retries internally
    }
}
```

Now the caller only needs:

```java
notificationService.send("user@example.com", "Hello");
```

That is abstraction.

## Another Easy Example
A coffee machine gives buttons like:
- espresso
- cappuccino
- latte

You do not deal with bean grinding, water temperature, or pressure control.

The machine hides the complexity and exposes only the useful operations.

## Real Meaning
Abstraction is not just using an interface or abstract class.

Those are tools.

Real abstraction means exposing the right level of detail to the caller.

## Good vs Bad Abstraction
Bad abstraction:

```java
interface PrinterService {
    void connectCable();
    void warmUpInk();
    void rotateRoller();
    void printBytes(byte[] data);
}
```

This exposes too many low-level details.

Better abstraction:

```java
interface PrinterService {
    void print(Document document);
}
```

The caller wants printing, not printer internals.

## Abstraction vs Encapsulation
Encapsulation is about protecting data and controlling how state changes.

Abstraction is about hiding unnecessary complexity and exposing only essential behavior.

Simple memory trick:
- Encapsulation protects correctness.
- Abstraction reduces complexity.

## Strong Mental Model
Ask this question:

What does the caller actually need to know?

If the caller knows too many internal details, the abstraction is weak.

## Interview-Ready Definition
Abstraction is the process of exposing only the essential behavior of an object or system while hiding unnecessary implementation details, so users can interact with it in a simple and meaningful way.

## Interview-Ready Why
We use abstraction to reduce complexity, separate concerns, hide implementation details, and make systems easier to use, change, and extend.

## Fast Recall
Think: show what matters, hide what does not.

Think: callers should know behavior, not internal process.

Think: good abstraction gives a simple interface to a complex system.

## Questions To Ask In Design
What does the caller actually need?

Am I exposing behavior or leaking implementation details?

Can I make this interaction simpler?

If internals change later, will callers remain unaffected?