# Inheritance

## In One Line
Inheritance means a child class can acquire the properties and behavior of a parent class, modeling an "is-a" relationship.

## Core Idea
When multiple classes share common behavior, you put that shared behavior in a parent class and let children extend it.

A `Dog` is an `Animal`. A `SavingsAccount` is a `BankAccount`. The child gets everything the parent has, and can add or override behavior.

This avoids duplicating the same code across similar classes and lets you treat related types uniformly.

## Why Do We Need It?
Without inheritance, if `SavingsAccount`, `CurrentAccount`, and `FixedDeposit` all need `deposit()` and `getBalance()`, you would copy-paste that logic into each class. When it changes, you update three places and inevitably miss one.

Inheritance also enables polymorphism. You can write code that accepts `BankAccount` and it automatically works with all subtypes. This is what makes systems extensible.

## What Goes Wrong Without It?
Common behavior gets duplicated across similar classes. That means inconsistent logic, more bugs, and painful maintenance.

You also lose the ability to treat related types uniformly. Instead of one method that accepts "any shape," you write separate methods for `Circle`, `Rectangle`, and `Triangle`.

## Simple Example
Bad design — duplicated code:

```java
class SavingsAccount {
    private double balance;
    public void deposit(double amt) { balance += amt; }
    public double getBalance() { return balance; }
}
class CurrentAccount {
    private double balance;
    public void deposit(double amt) { balance += amt; }
    public double getBalance() { return balance; }
}
```

Better design — shared behavior in a parent:

```java
class BankAccount {
    private double balance;
    public void deposit(double amt) { if (amt > 0) balance += amt; }
    public double getBalance() { return balance; }
}
class SavingsAccount extends BankAccount {
    private double interestRate;
}
class CurrentAccount extends BankAccount {
    private double overdraftLimit;
}
```

Common logic lives once. Each child adds only what is unique.

## Good vs Bad Usage
Bad — Stack is NOT a Vector:

```java
class Stack extends Vector { ... }
```

Stack supports push/pop. Vector supports insertion at any index. By extending Vector, Stack leaks operations that break its own abstraction.

Good — use composition when there is no true "is-a":

```java
class Stack<T> {
    private List<T> elements = new ArrayList<>();
    public void push(T item) { elements.add(item); }
    public T pop() { return elements.remove(elements.size() - 1); }
}
```

The internal list is hidden. Stack controls what is exposed.

## Composition vs Inheritance
Inheritance = "is-a" (Dog is-a Animal).
Composition = "has-a" (Car has-a Engine).

Use inheritance for genuine type hierarchies. Use composition when you just need to reuse functionality without a true subtype relationship.

Memory trick: if substituting the child everywhere the parent is used feels wrong, use composition instead.

## Real Meaning
Inheritance is not just about code reuse.

It is about modeling a type hierarchy where the child is truly a specialized version of the parent, and can be used anywhere the parent is expected.

Blindly extending a class just to get its methods is misuse.

## Strong Mental Model
Ask this question:

Can I use the child everywhere the parent is used, and does everything still make sense?

If yes, inheritance fits. If no, use composition.

## Interview-Ready Definition
Inheritance allows a child class to acquire the properties and behavior of a parent class, enabling code reuse and modeling "is-a" relationships. The child can extend or override the parent's behavior while being usable anywhere the parent type is expected.

## Interview-Ready Why
We use inheritance to avoid duplicating shared behavior across similar classes, to model real type hierarchies, and to enable polymorphism so code can work with parent types and automatically support all subtypes.

## Fast Recall
Think: inheritance models "is-a," composition models "has-a."

Think: if substitution feels wrong, prefer composition.

Think: keep hierarchies shallow — deep chains are fragile.

## Questions To Ask In Design
Is this truly an "is-a" relationship or just code reuse?

Would composition give me the same benefit with less coupling?

Can the child be substituted everywhere the parent is used without breaking anything?

Is my inheritance hierarchy deeper than 2 levels, and can I flatten it?
