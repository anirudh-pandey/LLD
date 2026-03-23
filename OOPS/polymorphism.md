# Polymorphism

## In One Line
Polymorphism means the same method call can behave differently based on the actual object.

## Core Idea
In Java, polymorphism lets you write code against a parent type and still get child-specific behavior at runtime.

So instead of writing separate logic for `Circle`, `Rectangle`, and `Triangle`, you can write one flow for `Shape` and let each object handle itself.

This is what makes systems easier to extend without constantly editing old code.

Polymorphism is commonly seen in two forms: overloading and overriding.

## Overloading (Compile-Time Polymorphism)
Overloading means same method name, different parameter list, usually in the same class.

The compiler decides which method to call based on argument count and types.

```java
class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
}
```

```java
Calculator c = new Calculator();
System.out.println(c.add(2, 3));       // 5
System.out.println(c.add(2.5, 3.5));   // 6.0
System.out.println(c.add(1, 2, 3));    // 6
```

No runtime dispatch here. Method is fixed at compile time.

## Overriding (Runtime Polymorphism)
Overriding means a child class provides its own implementation of a parent method with the same signature.

At runtime, Java picks the implementation based on the actual object type.

```java
class Animal {
    void sound() { System.out.println("Animal sound"); }
}
class Dog extends Animal {
    @Override
    void sound() { System.out.println("Bark"); }
}
```

```java
Animal a1 = new Animal();
Animal a2 = new Dog();
System.out.print("a1: "); a1.sound();   // Animal sound
System.out.print("a2: "); a2.sound();   // Bark
```

Same method call, different behavior. This is the polymorphism that matters most in LLD design.

## Important Note (Interview Favorite)
There are two checks for method calls:

Step 1 (compile time): "Can I call this member?" This is checked using reference type.

Step 2 (runtime): "Which overridden version should run?" This is decided using actual object type.

So `A ref = new B(); ref.someMethod()` works only if `someMethod` exists in `A`. If it exists, runtime may still execute B's overridden implementation.

## Overloading vs Overriding (Quick Difference)
Overloading:
- Same class (usually)
- Same method name, different parameters
- Decided at compile time

Overriding:
- Parent-child relationship
- Same method signature
- Decided at runtime using actual object type

## Why Do We Need It?
Without polymorphism, every new type usually adds more `if-else` or `switch` branches in existing code.

That quickly becomes hard to maintain, easy to break, and painful to test.

With polymorphism, the caller stays stable. New behavior is added by creating a new class, not by modifying the caller.

## What Goes Wrong Without It?
Business logic gets centralized into giant conditional blocks.

Every time a new type is introduced, multiple old files change. That increases regression risk.

This also violates good design direction, because high-level code starts knowing too many low-level details.

## Simple Example
Bad design:

```java
class PaymentService {
    void pay(String mode, double amount) {
        if (mode.equals("CARD")) {
            System.out.println("Card payment");
        } else if (mode.equals("UPI")) {
            System.out.println("UPI payment");
        } else if (mode.equals("WALLET")) {
            System.out.println("Wallet payment");
        }
    }
}
```

Better design:

```java
interface PaymentMethod {
    void pay(double amount);
}
class CardPayment implements PaymentMethod {
    public void pay(double amount) { System.out.println("Card payment"); }
}
class UpiPayment implements PaymentMethod {
    public void pay(double amount) { System.out.println("UPI payment"); }
}
```

Caller code now depends on `PaymentMethod`, not every concrete type.

## Another Easy Example
A remote control has one `powerOn()` button, but the behavior differs for TV, AC, and projector.

Same action from user side, different execution by object side. That is polymorphism in plain life.

## Real Meaning
Polymorphism is not just "method overriding syntax."

Its real value is that calling code stays generic while object-specific behavior is selected automatically.

This is a major reason OOP systems remain extensible over time.

## Good vs Bad Example
Bad approach: every shape handled using string checks and conditionals in one class.

Good approach: each shape class implements its own `draw()` and caller only does `shape.draw()`.

## Comparison With Related Concept
Inheritance gives you hierarchy. Polymorphism gives you dynamic behavior through that hierarchy.

Memory trick: inheritance is structure, polymorphism is behavior.

## Strong Mental Model
Ask this:

If I add a new subtype tomorrow, can I avoid changing current high-level flow?

If yes, you are using polymorphism correctly.

## Interview-Ready Definition
Polymorphism is the ability to use a common parent type or interface and invoke the same method call while getting different behavior based on context. In Java, this appears as compile-time polymorphism (overloading) and runtime polymorphism (overriding), with runtime polymorphism being central to extensible LLD design.

## Interview-Ready Why
We use polymorphism to remove rigid type checks, reduce coupling, and support extension through new classes instead of editing existing core logic.

## Fast Recall
Think: one interface, many implementations.

Think: same call, different behavior.

Think: add new type, avoid changing old flow.

## Questions To Ask In Design
Am I relying on type checks instead of object behavior?

Can this logic be pushed into subtype implementations?

Will adding a new type force changes in existing high-level code?

Can I depend on an interface instead of concrete classes?
