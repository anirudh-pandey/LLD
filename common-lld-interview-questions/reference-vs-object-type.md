# Reference Type vs Object Type

## In One Line
In Java, the reference type decides what you can access, but the actual object type decides which overridden method runs.

## Core Idea
When you write `Animal a = new Dog()`, two things are happening. `Animal` is the reference type — the "window" you are looking through. `Dog` is the actual object that exists in memory.

The compiler uses the reference type to decide what methods and fields you are allowed to call. But at runtime, if a method is overridden, Java uses the actual object type to decide which version of the method runs.

This distinction is the foundation of runtime polymorphism.

## The Three Cases

```java
class Animal {
    int age = 10;
    void eat() { System.out.println("Animal eats"); }
}
class Dog extends Animal {
    int age = 5;
    void eat() { System.out.println("Dog eats"); }
    void bark() { System.out.println("Dog barks"); }
}
```

**Case 1:** `Animal a = new Animal()`
You see `Animal` members. `Animal` methods run. Nothing surprising.

**Case 2:** `Animal a = new Dog()`
You can only access `Animal` members. But overridden methods run from `Dog`.

```java
a.eat();   // "Dog eats" — runtime polymorphism
a.bark();  // compile error — Animal has no bark()
a.age;     // 10 — fields use reference type
```

**Case 3:** `Dog a = new Dog()`
You can access everything — `Dog` members plus inherited `Animal` members. Overridden methods run from `Dog`.

## The Key Rules

There are two separate checks happening for any member access:

**Step 1 — Can I access it?** (compile time)
Always decided by the reference type. The compiler asks: does this type have this member? If no, compile error. It does not care what the actual object is.

**Step 2 — Which version runs?** (runtime)
Only happens after Step 1 passes.

Overridden instance methods: resolved at runtime using actual object type.

Fields: resolved at compile time using reference type. No polymorphism.

Static methods: resolved at compile time using reference type. They are hidden, not overridden.

**Important:** Step 1 is the gatekeeper. If the reference type does not have the member, you get a compile error and Step 2 never happens. This is why `A obj = new B(); obj.print()` fails even though B has `print()` — the compiler only sees A.

## Static Methods Example

```java
class Animal {
    static void type() { System.out.println("Animal"); }
}
class Dog extends Animal {
    static void type() { System.out.println("Dog"); }
}
```

```java
Animal a = new Dog();
a.type();   // "Animal" — static methods use reference type
Dog d = new Dog();
d.type();   // "Dog"
```

This is called method hiding, not overriding. Static methods belong to the class, not the object, so runtime polymorphism does not apply.

## Fields Example

```java
class Animal {
    int age = 10;
}
class Dog extends Animal {
    int age = 5;
}
```

```java
Animal a = new Dog();
System.out.println(a.age);   // 10 — reference type is Animal

Dog d = new Dog();
System.out.println(d.age);   // 5 — reference type is Dog
```

Fields are never polymorphic in Java. Even though the actual object is a `Dog`, `a.age` resolves using the `Animal` reference type.

## Why `Dog a = new Animal()` Is Invalid
Not every `Animal` is a `Dog`. If Java allowed this, you could call `a.bark()` on an object that has no `bark()`. So it prevents it at compile time.

## Casting
If you know the actual object is a `Dog`, you can cast:

```java
Animal a = new Dog();
Dog d = (Dog) a;     // valid, actual object is Dog
d.bark();            // works
```

But this throws `ClassCastException` at runtime if the actual object is not a `Dog`:

```java
Animal a = new Animal();
Dog d = (Dog) a;     // ClassCastException
```

## Strong Mental Model
Think of the reference type as a window. You are holding a `Dog`, but looking at it through an `Animal` window. You can only ask animal-level questions, but if the dog has redefined an animal behavior, the dog version runs.

## Quick Summary Table

| Code | See Members Of | Overridden Method Runs From |
|---|---|---|
| `Animal a = new Animal()` | `Animal` | `Animal` |
| `Animal a = new Dog()` | `Animal` only | `Dog` |
| `Dog a = new Dog()` | `Dog` + inherited | `Dog` |
| `Dog a = new Animal()` | — | compile error |

## Interview-Ready Answer
The reference type determines what members are accessible at compile time. The actual object type determines which overridden instance method executes at runtime. Fields and static methods are always resolved using the reference type, but overridden non-static methods use dynamic dispatch based on the actual object.

## Fast Recall
Think: reference type = what you can see.

Think: object type = what actually runs.

Think: fields and statics follow reference type, overridden methods follow object type.
