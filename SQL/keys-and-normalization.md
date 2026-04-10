# SQL Keys, Normalization & Schema Design

## In One Line
Keys identify and relate rows. Normalization removes redundancy. Together they make schemas correct and maintainable.

---

## Part 1 — All Types of Keys

### Primary Key
Uniquely identifies every row. One per table. Always NOT NULL and UNIQUE.

```sql
CREATE TABLE Employee (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100)
);
```

### Foreign Key
Points to a primary key in another table. Enforces referential integrity — you can't insert a value that doesn't exist in the referenced table. No orphan rows.

```sql
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES Customers(customer_id)
);
```

### Unique Key
Guarantees no duplicates in a column, but it's not the PK. A table can have many unique constraints. NULLs are usually allowed (one NULL per column, varies by DB).

Use it for columns like `email` or `phone` — not the identity, but still must be distinct.

```sql
ALTER TABLE Users ADD CONSTRAINT uq_email UNIQUE (email);
```

### Candidate Key
Any column (or combo) that *could* be the PK — unique and not null. A table can have multiple. You pick one as the PK; the rest become alternate keys.

Example: In `Users`, both `user_id` and `email` uniquely identify a row. Both are candidate keys. Pick `user_id` as PK; `email` becomes an alternate key.

### Alternate Key
A candidate key you didn't pick as PK. Enforce it with a UNIQUE constraint.

### Super Key
Any set of columns that uniquely identifies a row — even if it has unnecessary extras. `{employee_id, name}` is a super key even though `employee_id` alone is enough. Mostly a theory concept; interviewers ask about it occasionally.

### Composite Key
A key made of two or more columns. Typical in junction tables for many-to-many relationships.

```sql
CREATE TABLE StudentCourse (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id)
);
```

Neither column is unique alone, but together they uniquely identify an enrollment.

### Natural Key vs Surrogate Key
- **Natural key** — a real-world value that identifies the row: `email`, `SSN`, `isbn`.
- **Surrogate key** — a system-generated ID: auto-increment `int` or UUID.

**Use surrogate keys** for almost everything. Real-world values change (emails, phone numbers, company names). Surrogate keys never do.

**Use natural keys** only when the value is truly stable and unlikely to ever change — like country codes (`US`, `IN`) or currency codes (`USD`).

**Rule of thumb:** Surrogate key as PK + natural key as UNIQUE constraint. Best of both worlds.

---

## Part 2 — Key Selection Cheat Sheet

| Question | Answer |
|---|---|
| What uniquely identifies this row in the real world? | That's your candidate key |
| Can that value change over time? | If yes → use a surrogate PK instead |
| Is it a long string? | Avoid as PK — slow joins, bloated FKs |
| Multiple candidate keys? | Pick the most stable + compact as PK, UNIQUE the rest |
| Is this a junction table? | Composite PK of both FKs |

---

## Part 3 — Normalization Step by Step

Normalization structures tables to avoid redundancy and prevent anomalies (bad updates, bad inserts, bad deletes). Done in stages — called normal forms.

### Why Bother?
Imagine one flat table with customer name, address, and order items all in one row. If the customer moves, you update 500 rows. Miss one? Now there are two addresses for the same customer — that is an **update anomaly**.

Normalization stores each fact exactly once.

### Unnormalized (UNF) — The Starting Point

| OrderID | CustomerName | CustomerCity | Items |
|---------|-------------|-------------|-------|
| 1 | Rahul | Delhi | Pen, Notebook |
| 2 | Priya | Mumbai | Laptop |

Problems: `Items` has multiple values in one cell. Customer info is repeated. No clear key.

---

### First Normal Form (1NF) — One value per cell

**Rule:** Every cell must hold a single atomic value. No comma-separated lists, no arrays.

**Fix:** Split multi-valued rows into separate rows.

| OrderID | CustomerName | CustomerCity | Item |
|---------|-------------|-------------|------|
| 1 | Rahul | Delhi | Pen |
| 1 | Rahul | Delhi | Notebook |
| 2 | Priya | Mumbai | Laptop |

Composite key: `{OrderID, Item}`. Every cell now holds one value.

**Still wrong:** Customer info duplicated across rows for the same order.

---

### Second Normal Form (2NF) — Depend on the whole key

**Rule:** 1NF + every non-key column depends on the *entire* PK (not just part of it).

Only matters with composite keys. If a column depends on only *one part* of the key, that is a **partial dependency** — move it out.

**Fix:** `CustomerName` and `CustomerCity` only depend on `OrderID`, not the full `{OrderID, Item}` key. Split them into a separate table.

**Orders:**

| OrderID | CustomerName | CustomerCity |
|---------|-------------|-------------|
| 1 | Rahul | Delhi |
| 2 | Priya | Mumbai |

**OrderItems:**

| OrderID | Item |
|---------|------|
| 1 | Pen |
| 1 | Notebook |
| 2 | Laptop |

**Still wrong:** `CustomerCity` depends on `CustomerName`, not on `OrderID`. That is a **transitive dependency**.

---

### Third Normal Form (3NF) — No non-key column depends on another non-key column

**Rule:** 2NF + no non-key column depends on another non-key column.

**Fix:** `CustomerCity → CustomerName → OrderID` is a chain. Break it by pulling customers into their own table.

**Orders:**

| OrderID | CustomerID |
|---------|-----------|
| 1 | 101 |
| 2 | 102 |

**Customers:**

| CustomerID | CustomerName | CustomerCity |
|-----------|-------------|-------------|
| 101 | Rahul | Delhi |
| 102 | Priya | Mumbai |

**OrderItems:**

| OrderID | Item |
|---------|------|
| 1 | Pen |
| 1 | Notebook |
| 2 | Laptop |

Now each fact lives in one place. Rahul moves cities? Update exactly one row.

---

### Boyce-Codd Normal Form (BCNF)
**Rule:** 3NF + every determinant (the left side of any functional dependency) must be a candidate key.

In practice, most 3NF tables are already in BCNF. The gap only appears with overlapping composite candidate keys — a rare edge case.

**Interview line:** "BCNF tightens 3NF — it requires that every functional dependency's left side is a candidate key." That's enough.

---

### Normalization Summary Table

| Normal Form | What it removes | One-liner |
|------------|----------------|-----------|
| 1NF | Multi-valued cells | One value per cell |
| 2NF | Partial dependencies (on part of composite key) | Whole key |
| 3NF | Transitive dependencies (non-key → non-key) | Nothing but the key |
| BCNF | Determinants that aren't candidate keys | Every determinant is a key |

**Mnemonic:** Non-key columns must depend on **the key, the whole key, and nothing but the key** — so help me Codd.

---

## Part 4 — When to Denormalize

Normalization is great for correctness. But for read-heavy systems, too many joins slow things down.

**Denormalize when:**
- Reads far outweigh writes (dashboards, analytics, reporting)
- A join is run constantly on data that rarely changes
- You've measured the performance cost and it's real

**Do it safely:**
- Keep the normalized source of truth
- Sync the redundant copy via triggers, app logic, or batch jobs
- Leave a comment explaining why — so no one "fixes" it later

**Interview line:** "I normalize first for correctness, then selectively denormalize for performance where measurements justify it."

---

## Part 5 — Schema Design Walkthrough

Follow these steps when asked to design a schema in an interview:

1. **Identify entities** — list the nouns from requirements: User, Order, Product, Payment, Address.
2. **Map relationships** — User *places* Order (1:N). Order *contains* Products (M:N). User *has* Addresses (1:N).
3. **Define PKs** — surrogate PK for each entity. Mark natural candidate keys as UNIQUE.
4. **Add FKs** — 1:N: FK on the "many" side. M:N: junction table with composite PK.
5. **Assign attributes** — each column belongs to the table whose PK it depends on (this is just 2NF/3NF applied intuitively).
6. **Check for redundancy** — same fact in two places? Ask: "If this changes, how many rows do I update?"
7. **Add constraints** — NOT NULL for required fields, UNIQUE for natural keys, CHECK for domain rules, FKs for all relationships.
8. **Think about queries** — what are the most common reads? Add indexes on WHERE/JOIN/ORDER BY columns. Denormalize only if justified.

### Mini Example — E-Commerce Schema

```sql
CREATE TABLE Customers (
    customer_id INT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    price DECIMAL(10,2) NOT NULL CHECK (price > 0)
);

CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES Customers(customer_id)
);

CREATE TABLE OrderItems (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```

> `unit_price` is stored in `OrderItems` even though `Products.price` exists. This is intentional — the price at order time may differ from the current price. Historically accurate records require this deliberate redundancy.

---

## Common Interview Questions

**Q: What are the different types of keys?**
PK (identifies rows), FK (relates tables), Unique (no duplicates, not PK), Candidate (could be PK), Alternate (candidate not chosen), Super (any unique combo including extras), Composite (multi-column key).

**Q: Natural key vs surrogate key — which to use?**
Default to surrogate PK + natural key as UNIQUE. Natural keys change (emails, names), are longer, and make migrations painful. Surrogate keys are stable and compact.

**Q: What is normalization and how far should I go?**
It's structuring tables to store each fact once. Go to 3NF by default — removes repeating groups (1NF), partial deps (2NF), transitive deps (3NF). Beyond 3NF is rarely needed.

**Q: When is denormalization acceptable?**
When read performance clearly outweighs write consistency, and you've measured the cost. Always keep the normalized source of truth and sync the copy.

**Q: How do you handle many-to-many relationships?**
Junction table with a composite PK of both FKs. Add extra columns if the relationship itself has data (e.g., `quantity`, `enrolled_on`).

**Q: Difference between 2NF and 3NF?**
2NF: remove partial deps (non-key depending on *part* of a composite key). 3NF: remove transitive deps (non-key depending on *another* non-key column).

---

## Fast Recall

**Keys** — PK identifies, FK relates, UNIQUE constrains, composite combines.

**Normalize** — the key, the whole key, and nothing but the key.

**Design** — entities → relationships → keys → attributes → constraints → access patterns.

## Questions To Ask During Schema Design

- Does every table have a clear PK — surrogate or natural?
- Is the same fact stored in more than one place?
- For every FK, is the relationship 1:N or M:N?
- Which queries run most often — do I need denormalization or extra indexes?
