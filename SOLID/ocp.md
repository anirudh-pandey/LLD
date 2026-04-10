# Open/Closed Principle (OCP)

## In One Line
OCP means code should be easy to extend with new behavior without repeatedly modifying stable existing code.

## Core Idea
When a new requirement arrives, the ideal outcome is that you add a new class or implementation instead of reopening the same working class and inserting more conditions.

This does not mean old code is never changed. It means the design should create extension points in places where variation is expected.

In simple words, stable flow should stay stable, while changing behavior should be plugged in.

## Why Do We Need It?
We need OCP because some parts of software naturally keep growing in variants.

Payment methods, discount rules, notification channels, export formats, and tax calculators are common examples. If every new variant forces edits in the same central class, that class becomes a regression hotspot.

OCP reduces that risk. Existing code stays relatively untouched, and new behavior is added with less chance of breaking what already works.

## What Goes Wrong Without It?
Without OCP, systems grow through `if-else` chains and `switch` blocks.

One service slowly becomes the owner of every case in the system. Each new type means more branching, more retesting, and more chances to break old behavior.

This also makes code harder to read because the logic for all variants gets crowded into one place.

## Simple Example
Bad design:

```java
class PaymentService {
    public void pay(String type) {
        if (type.equals("CARD")) {
            System.out.println("Pay using card");
        } else if (type.equals("UPI")) {
            System.out.println("Pay using UPI");
        }
    }
}
```

Adding `WALLET` means modifying this same class again.

Better design:

```java
interface PaymentMethod {
    void pay();
}

class CardPayment implements PaymentMethod {
    public void pay() { System.out.println("Pay using card"); }
}

class UpiPayment implements PaymentMethod {
    public void pay() { System.out.println("Pay using UPI"); }
}

class PaymentService {
    public void process(PaymentMethod paymentMethod) {
        paymentMethod.pay();
    }
}
```

Now a new payment type is added by creating another implementation.

## Another Easy Example
A notification system often starts with email only.

If later SMS and push notification are added, a design based on `NotificationChannel` is easier to extend than a method full of channel-based conditions.

This is exactly where OCP pays off because variation is normal in that domain.

## Real Meaning
OCP does not mean "use interfaces everywhere."

It also does not mean "predict every future change."

Real OCP means identifying likely change points and designing those places so new behavior is added mostly by extension, not by repeatedly editing old logic.

## Comparison With Related Concept
SRP is about one reason to change.

OCP is about how to handle future variation when change does happen.

Memory trick: SRP keeps responsibilities focused. OCP keeps growing behavior from rewriting stable code.

## Strong Mental Model
Ask this question:

When a new case arrives, will I mostly add a new class or will I keep editing the same old decision-making code?

If the answer is repeated edits, OCP is weak.

## Interview-Ready Definition
Open/Closed Principle states that software entities should be open for extension but closed for modification. In practice, this means new behavior should preferably be introduced by adding new implementations around stable abstractions instead of repeatedly changing old working code.

## Interview-Ready Why
We use OCP to reduce regression risk and keep systems easier to grow. When a domain keeps gaining new variants, OCP helps us extend behavior with less disturbance to existing code.

## Common Interview Questions
Does OCP mean we should never modify old code?
No. It means we should avoid repeated modification in places where future variation is expected.

What if I am unsure whether future modifications will come?
Then start simple. Prefer a clean design that is easy to refactor later, and introduce abstraction when change becomes likely or patterns start repeating.

Is OCP always implemented using interfaces?
No. Interfaces are one tool. The principle is about extensibility, not about a specific syntax choice.

What is the danger of overapplying OCP?
You can create unnecessary abstractions for imaginary future cases, which makes the design harder to understand without giving real benefit.

How is OCP related to polymorphism?
Polymorphism is one of the main ways to implement OCP because it lets the caller depend on a stable abstraction while behavior varies by implementation.

## Fast Recall
Think: stable flow, variable behavior.

Think: add new code, avoid rewriting old code again and again.

Think: do not abstract for imaginary futures.

## Questions To Ask In Design
Where is variation likely in this system?

If a new case comes tomorrow, what code will I need to touch?

Am I creating a real extension point or just speculative abstraction?

Will extending this behavior require editing stable old logic again?
