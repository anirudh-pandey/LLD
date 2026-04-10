# SQL Queries — Interview Revision Sheet

## In One Line
SQL queries let you ask a database exactly what data you want, how to filter it, combine it, group it, and shape it — all in one readable statement.

---

## How a Query Executes (The Mental Model)

Before writing queries, understand the order SQL actually runs things — it is **not** top to bottom.

**Logical execution order:**

```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT
```

You write `SELECT` first, but the engine reads `FROM` first. This is why you cannot use a column alias from `SELECT` inside `WHERE` — `WHERE` runs before `SELECT`.

Think of it like a pipeline: each step filters or transforms the data, then hands it to the next step.

---

## SELECT, FROM, WHERE — The Basics

```sql
SELECT name, salary
FROM Employee
WHERE department = 'Engineering' AND salary > 50000;
```

`FROM` picks the table. `WHERE` filters rows. `SELECT` picks which columns to show.

**Common operators in WHERE:** `=`, `!=` or `<>`, `>`, `<`, `>=`, `<=`, `BETWEEN`, `IN`, `LIKE`, `IS NULL`, `IS NOT NULL`, `AND`, `OR`, `NOT`.

```sql
-- Pattern matching
SELECT * FROM Employee WHERE name LIKE 'A%';       -- starts with A
SELECT * FROM Employee WHERE name LIKE '%kumar';    -- ends with kumar
SELECT * FROM Employee WHERE name LIKE '_a%';       -- second char is 'a'

-- IN and BETWEEN
SELECT * FROM Employee WHERE department IN ('HR', 'Finance');
SELECT * FROM Employee WHERE salary BETWEEN 40000 AND 80000;

-- NULL checks (never use = NULL, it won't work)
SELECT * FROM Employee WHERE manager_id IS NULL;
```

---

## Sorting and Limiting

```sql
SELECT name, salary
FROM Employee
ORDER BY salary DESC, name ASC
LIMIT 10;
```

`ORDER BY` sorts the result. `DESC` for high-to-low, `ASC` (default) for low-to-high. You can sort by multiple columns — it sorts by the first, then breaks ties with the second.

`LIMIT` caps the number of rows returned. In SQL Server, use `TOP` instead:
```sql
SELECT TOP 10 name, salary FROM Employee ORDER BY salary DESC;
```

---

## Aggregations — COUNT, SUM, AVG, MIN, MAX

Aggregations collapse many rows into one summary value.

```sql
SELECT department,
       COUNT(*) AS employee_count,
       AVG(salary) AS avg_salary,
       MAX(salary) AS max_salary
FROM Employee
GROUP BY department;
```

`GROUP BY` splits rows into buckets (one per department). Each aggregate function runs within each bucket.

**Key rule:** Every column in `SELECT` must either be in `GROUP BY` or inside an aggregate function. Otherwise the database does not know which value to pick from the group.

---

## HAVING — Filter After Grouping

`WHERE` filters rows before grouping. `HAVING` filters groups after aggregation.

```sql
-- Departments with more than 5 employees
SELECT department, COUNT(*) AS cnt
FROM Employee
GROUP BY department
HAVING COUNT(*) > 5;
```

**Memory trick:** WHERE = before grouping, HAVING = after grouping. You cannot use `WHERE COUNT(*) > 5` because `WHERE` runs before `GROUP BY` even sees the data.

---

## JOINs — Combining Tables

### INNER JOIN
Returns only rows that have a match in both tables.

```sql
SELECT e.name, d.department_name
FROM Employee e
INNER JOIN Department d ON e.department_id = d.id;
```

If an employee has no matching department, that employee is excluded.

### LEFT JOIN (LEFT OUTER JOIN)
Returns all rows from the left table, and matched rows from the right. Unmatched right side fills with NULL.

```sql
-- All employees, even those without a department
SELECT e.name, d.department_name
FROM Employee e
LEFT JOIN Department d ON e.department_id = d.id;
```

### RIGHT JOIN
Same idea but keeps all rows from the right table. In practice, most people just swap the table order and use LEFT JOIN.

### FULL OUTER JOIN
Keeps all rows from both tables. Unmatched sides get NULL.

### CROSS JOIN
Every row from table A paired with every row from table B. Produces A × B rows. Rarely needed, but useful for generating combinations.

```sql
SELECT s.size, c.color
FROM Sizes s CROSS JOIN Colors c;
```

### SELF JOIN
A table joined to itself. Common for hierarchical data (employee → manager).

```sql
-- Find each employee's manager name
SELECT e.name AS employee, m.name AS manager
FROM Employee e
LEFT JOIN Employee m ON e.manager_id = m.id;
```

### Join Decision Quick Reference
- Need rows from both sides? → `INNER JOIN`
- Need all rows from one side even without match? → `LEFT JOIN`
- Need all rows from both sides? → `FULL OUTER JOIN`
- Need all combinations? → `CROSS JOIN`
- Table relates to itself? → `SELF JOIN`

---

## Subqueries

A query inside another query. Works wherever you can put a table or a value.

### In WHERE (scalar subquery)
```sql
-- Employees earning above company average
SELECT name, salary
FROM Employee
WHERE salary > (SELECT AVG(salary) FROM Employee);
```

### In WHERE with IN
```sql
-- Employees in departments located in Mumbai
SELECT name
FROM Employee
WHERE department_id IN (
    SELECT id FROM Department WHERE city = 'Mumbai'
);
```

### In FROM (derived table / inline view)
```sql
-- Top department by average salary
SELECT dept, avg_sal
FROM (
    SELECT department AS dept, AVG(salary) AS avg_sal
    FROM Employee
    GROUP BY department
) AS dept_avg
ORDER BY avg_sal DESC
LIMIT 1;
```

### Correlated Subquery
References a column from the outer query. Runs once per row of the outer query — so it can be slow.

```sql
-- Employees earning more than the average of their own department
SELECT name, salary, department
FROM Employee e1
WHERE salary > (
    SELECT AVG(salary) FROM Employee e2 WHERE e2.department = e1.department
);
```

---

## EXISTS and NOT EXISTS

Checks whether a subquery returns any rows. Often faster than `IN` for large datasets.

```sql
-- Customers who have placed at least one order
SELECT name
FROM Customer c
WHERE EXISTS (
    SELECT 1 FROM Orders o WHERE o.customer_id = c.id
);

-- Customers who have never placed an order
SELECT name
FROM Customer c
WHERE NOT EXISTS (
    SELECT 1 FROM Orders o WHERE o.customer_id = c.id
);
```

---

## CASE — Conditional Logic

Like an if-else inside SQL.

```sql
SELECT name, salary,
    CASE
        WHEN salary > 100000 THEN 'High'
        WHEN salary > 50000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_band
FROM Employee;
```

You can also use CASE inside `ORDER BY`, `GROUP BY`, or aggregate functions.

---

## Window Functions — The Interview Favourite

Window functions compute a value across a set of rows related to the current row — without collapsing the rows like `GROUP BY` does.

**Syntax:** `FUNCTION() OVER (PARTITION BY ... ORDER BY ...)`

### ROW_NUMBER, RANK, DENSE_RANK

```sql
SELECT name, department, salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS row_num,
    RANK()       OVER (PARTITION BY department ORDER BY salary DESC) AS rnk,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dense_rnk
FROM Employee;
```

| Salary | ROW_NUMBER | RANK | DENSE_RANK |
|--------|-----------|------|------------|
| 90000  | 1         | 1    | 1          |
| 90000  | 2         | 1    | 1          |
| 80000  | 3         | 3    | 2          |
| 70000  | 4         | 4    | 3          |

- `ROW_NUMBER` — always unique, no ties
- `RANK` — same rank for ties, skips next number (1,1,3)
- `DENSE_RANK` — same rank for ties, does not skip (1,1,2)

### Classic Interview Pattern: Nth Highest Salary

```sql
-- 2nd highest salary per department
SELECT * FROM (
    SELECT name, department, salary,
        DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
    FROM Employee
) ranked
WHERE rnk = 2;
```

### Running Totals and Aggregates Over Windows

```sql
SELECT name, department, salary,
    SUM(salary) OVER (PARTITION BY department ORDER BY salary) AS running_total,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM Employee;
```

### LAG and LEAD — Access Previous / Next Rows

```sql
SELECT name, salary,
    LAG(salary, 1)  OVER (ORDER BY hire_date) AS prev_salary,
    LEAD(salary, 1) OVER (ORDER BY hire_date) AS next_salary
FROM Employee;
```

Useful for comparisons like "show each month's revenue vs previous month."

---

## Set Operations — UNION, INTERSECT, EXCEPT

Combine results from two queries (both must have the same number of columns and compatible types).

```sql
-- All names from both tables, no duplicates
SELECT name FROM Employee UNION SELECT name FROM Contractor;

-- All names including duplicates
SELECT name FROM Employee UNION ALL SELECT name FROM Contractor;

-- Names that appear in both
SELECT name FROM Employee INTERSECT SELECT name FROM Contractor;

-- Names in Employee but not in Contractor
SELECT name FROM Employee EXCEPT SELECT name FROM Contractor;
```

`UNION` removes duplicates (slower). `UNION ALL` keeps everything (faster).

---

## Common Table Expressions (CTEs)

A CTE is a named temporary result set. Makes complex queries readable.

```sql
WITH high_earners AS (
    SELECT name, department, salary
    FROM Employee
    WHERE salary > 80000
)
SELECT department, COUNT(*) AS count
FROM high_earners
GROUP BY department;
```

CTEs are also great for recursive queries (e.g., org hierarchy):
```sql
WITH RECURSIVE org AS (
    SELECT id, name, manager_id, 1 AS level
    FROM Employee WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id, o.level + 1
    FROM Employee e JOIN org o ON e.manager_id = o.id
)
SELECT * FROM org;
```

---

## INSERT, UPDATE, DELETE — Data Modification

```sql
-- Insert
INSERT INTO Employee (name, department, salary) VALUES ('Anirudh', 'Engineering', 75000);

-- Update
UPDATE Employee SET salary = 80000 WHERE name = 'Anirudh';

-- Delete
DELETE FROM Employee WHERE department = 'Temp';
```

**Safety tip:** Always write `WHERE` in UPDATE/DELETE first. Running `DELETE FROM Employee` without WHERE deletes everything.

---

## Common Interview Query Patterns

**1. Find duplicates:**
```sql
SELECT email, COUNT(*) FROM Users GROUP BY email HAVING COUNT(*) > 1;
```

**2. Delete duplicates (keep one):**
```sql
DELETE FROM Users WHERE id NOT IN (
    SELECT MIN(id) FROM Users GROUP BY email
);
```

**3. Second highest salary (no window function):**
```sql
SELECT MAX(salary) FROM Employee
WHERE salary < (SELECT MAX(salary) FROM Employee);
```

**4. Employees who earn more than their manager:**
```sql
SELECT e.name
FROM Employee e
JOIN Employee m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

**5. Department with highest average salary:**
```sql
SELECT department FROM Employee
GROUP BY department
ORDER BY AVG(salary) DESC
LIMIT 1;
```

**6. Year-over-year comparison (LAG):**
```sql
SELECT year, revenue,
    LAG(revenue) OVER (ORDER BY year) AS prev_year_revenue,
    revenue - LAG(revenue) OVER (ORDER BY year) AS growth
FROM YearlyRevenue;
```

---

## Fast Recall

Think: FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT (execution order).
Think: WHERE filters rows, HAVING filters groups.
Think: Window functions = aggregation without collapsing rows.
Think: RANK skips numbers on ties, DENSE_RANK does not, ROW_NUMBER is always unique.
Think: Use EXISTS for "does at least one matching row exist?" — often faster than IN.
