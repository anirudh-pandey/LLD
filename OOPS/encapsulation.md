# Encapsulation

## In One Line
Encapsulation means an object should keep its data and the rules around that data together, and should not let the outside world change its state in any random way.

## Core Idea
A class should control its own state.

Outside code should ask the object to do something, not directly change its internals.

The real goal is not just hiding data. The goal is protecting correctness.

## Why Do We Need It?
We need encapsulation so that the object stays valid and its rules stay in one place.

For example, a bank account should decide how money is added or removed. Other classes should not directly change the balance.

It also reduces coupling. If callers use methods like `deposit()` and `withdraw()`, the internal implementation can change later without affecting the rest of the code.

## What Goes Wrong Without It?
Without encapsulation, any part of the code can break the object's rules.

That leads to invalid objects. A bank account may get a negative balance, or an order may jump straight from `CREATED` to `DELIVERED`.

Business rules also get scattered. One caller may validate properly, another may forget. Then bugs become harder to trace because the state can be changed from many places.

## Simple Example
Bad design:

```java
class BankAccount {
	public double balance;
}
```

Now any code can do:

```java
account.balance = -500;
```

This breaks the rule that balance should never go below zero.

Better design:

```java
class BankAccount {
	private double balance;

	public void deposit(double amount) {
		if (amount <= 0) throw new IllegalArgumentException();
		balance += amount;
	}

	public void withdraw(double amount) {
		if (amount <= 0 || amount > balance) throw new IllegalArgumentException();
		balance -= amount;
	}

	public double getBalance() {
		return balance;
	}
}
```

Here the object protects its own rule: `balance >= 0`.

## Real Meaning
Encapsulation is not just:

- making fields `private`
- adding getters and setters

Real encapsulation means:

- the class owns its state
- the class decides how that state can change
- the class protects its invariants

## Getters/Setters Are Not Enough
This is still weak design:

```java
class Employee {
	private int salary;

	public void setSalary(int salary) {
		this.salary = salary;
	}
}
```

Why? Because external code can still set invalid values.

Better approach: allow only meaningful operations and validate before changing state.

For example, `reviseSalary(increment)` is better than `setSalary(value)` in many cases.

## Hiding Data vs Protecting Meaning
Hiding data means not exposing fields directly.

Protecting meaning means allowing state changes only through valid business operations.

Example:
`balance` being `private` is hiding data.

Allowing balance to change only via `deposit()` and `withdraw()` is protecting meaning.

## Strong Mental Model
Ask this question:

Who owns the right to change this state?

If the answer is "anyone", the design is weak.
If the answer is "the object itself through controlled methods", the design is stronger.

## Another Good Example
Order status should not be changed like this:

```java
order.status = DELIVERED;
```

Because it may skip valid transitions such as:

`CREATED -> PAID -> SHIPPED -> DELIVERED`

Better:
- `markPaid()`
- `ship()`
- `deliver()`

This ensures the workflow stays valid.

## Interview-Ready Definition
Encapsulation is the practice of bundling data and related behavior into one unit while restricting direct access to internal state, so the object can enforce its own rules, maintain consistency, and hide implementation details.

## Interview-Ready Why
We use encapsulation so that an object can protect its validity, centralize business rules, reduce coupling, and prevent misuse from external code.

## Fast Recall
Think: the object should control its own state.

Think: outside code should use behavior, not fields.

Think: `private` helps, but real encapsulation is about protecting invariants.

## Questions To Ask In Design
What state does this class own?

What rules must always remain true?

Should outside code change this directly?

Can I expose behavior instead of exposing data?