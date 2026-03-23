# Single Responsibility Principle (SRP)

## In One Line
SRP means a class should have one clear responsibility, so it should change for only one kind of reason.

## Core Idea
A class should not try to do business logic, persistence, formatting, notifications, and validation all at once.

If one class is being pulled in different directions by different kinds of changes, that class is carrying multiple responsibilities.

The practical test is simple: if pricing rules change, should this class change? If database structure changes, should the same class also change? If yes, the design is mixed.

## Why Do We Need It?
We need SRP because unrelated changes should not collide in the same class.

Imagine an `Invoice` class that calculates totals, prints the invoice, and saves it to the database. A pricing rule change, a PDF format change, and a database schema change all hit the same file.

That makes the class fragile. Even a small change becomes risky because touching one concern may accidentally break another.

## What Goes Wrong Without It?
Without SRP, classes become dumping grounds. They grow fast because every new requirement finds a place inside the same file.

Testing also becomes messy. To test one behavior, you may need database setup, email mocks, logging stubs, and formatting dependencies even when they are not part of the thing you actually want to verify.

This is how "god classes" appear. They are hard to read, hard to change, and hard to trust.

## Simple Example
Bad design:

```java
class Invoice {
    public double calculateTotal() {
        return 1000;
    }

    public void printInvoice() {
        System.out.println("Printing invoice");
    }

    public void saveToDatabase() {
        System.out.println("Saving invoice");
    }
}
```

This class mixes business logic, presentation, and persistence.

Better design:

```java
class Invoice {
    public double calculateTotal() {
        return 1000;
    }
}

class InvoicePrinter {
    public void print(Invoice invoice) {
        System.out.println("Printing invoice");
    }
}

class InvoiceRepository {
    public void save(Invoice invoice) {
        System.out.println("Saving invoice");
    }
}
```

Now each class has one clear reason to change.

## Another Easy Example
A `UserService` should not register a user, hash the password, send the welcome email, write audit logs, and save directly to the database all by itself.

Those are related to the same flow, but they are not the same responsibility.

A better design keeps orchestration in one place and pushes email, persistence, and logging into focused collaborators.

## Real Meaning
SRP does not mean one method per class.

It also does not mean every class must be tiny.

Real SRP means one cohesive concern. A class can have multiple methods and still follow SRP if all of them serve the same responsibility.

## Comparison With Related Concept
SRP is often confused with "do one thing."

That phrase is too vague. SRP is stricter: one reason to change, not just one visible action.

Memory trick: one responsibility means one source of change pressure.

## Strong Mental Model
Ask this question:

If this class changes tomorrow, will all those changes come from the same kind of concern?

If the answer is no, the class probably has multiple responsibilities.

## Interview-Ready Definition
Single Responsibility Principle states that a class should have only one reason to change, which means it should own one cohesive responsibility. If a class handles multiple concerns like business rules, persistence, and presentation together, then unrelated changes get coupled and the design becomes harder to maintain.

## Interview-Ready Why
We use SRP to keep classes focused, reduce regression risk, improve readability, and make testing easier. When each class has one clear responsibility, changes stay local and the system becomes easier to evolve.

## Common Interview Questions
How do you identify whether a class has more than one responsibility?
Look at the change pressure on the class. If business rules, persistence, formatting, or other unrelated concerns can independently force edits, the class likely has multiple responsibilities.

Is a large class always an SRP violation?
No. Size alone does not decide SRP. A large class can still be cohesive, and a small class can still mix unrelated concerns.

How is SRP different from saying a class should do only one thing?
"One thing" is vague. SRP is more precise because it asks whether the class has only one reason to change.

Can a service class coordinate multiple collaborators and still follow SRP?
Yes. Coordination itself can be a valid responsibility, as long as the class is orchestrating the flow and not absorbing each collaborator's internal concern.

What is the risk of overapplying SRP?
You can fragment the design into too many tiny classes, which adds indirection and makes the system harder to follow without adding real value.

## Fast Recall
Think: one class, one reason to change.

Think: avoid god classes.

Think: split by responsibility, not by random methods.

## Questions To Ask In Design
What exactly is this class responsible for?

If database rules change, should this class change?

If UI or output format changes, should this class change?

Am I keeping unrelated concerns in the same class just for convenience?
