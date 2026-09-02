# Week 50 — Final Review: SQL

> **Exam review week.** No new concepts and no required work. Use this chapter as a SQL quick-reference handbook while preparing for the final exam.

## Chapter 14: "The SQL Reference"

The TrailShop database you built over Weeks 36–48 is a fully functional system — tables, relationships, constraints, indexes, roles, transactions, and more. This week is your comprehensive SQL review. Think of it as a reference handbook you can flip through before the exam. Every SQL feature you've learned is collected here in one place, organized by topic, with syntax references and TrailShop examples.

---

## Learning Objectives

By the end of this chapter you will be able to:

- Recall the full PostgreSQL syntax for DDL, DML, and query operations
- Use aggregation, joins, subqueries, and set operations fluently
- Write CTEs, CASE expressions, and transaction blocks
- Explain the logical query execution order
- Follow a consistent SQL style guide
- Identify when to use indexes and how to analyze query plans

---

## 1. SQL Quick Reference by Topic

This section is your master reference. Each subsection gives you the **syntax template**, **rules**, and a **TrailShop example**.

---

### 1.1 DDL — Data Definition Language

DDL commands define and modify the structure of your database objects.

> 📖 Watt & Eng, *Database Design — 2nd Edition*, Chapter 14: "SQL Data Definition"

#### CREATE TABLE

```sql
CREATE TABLE table_name (
    column_name data_type [constraints],
    ...
    [table_constraints]
);
```

**Common column constraints:**

| Constraint | Meaning |
|---|---|
| `PRIMARY KEY` | Unique identifier for each row |
| `NOT NULL` | Column cannot be empty |
| `UNIQUE` | No duplicate values allowed |
| `DEFAULT value` | Fallback value if none provided |
| `CHECK (condition)` | Must satisfy a Boolean condition |
| `REFERENCES table(col)` | Foreign key to another table |

**TrailShop example:**

```sql
CREATE TABLE products (
    product_id    SERIAL PRIMARY KEY,
    product_name  VARCHAR(200) NOT NULL,
    category_id   INTEGER NOT NULL REFERENCES categories(category_id),
    price         NUMERIC(10,2) NOT NULL CHECK (price > 0),
    stock_qty     INTEGER NOT NULL DEFAULT 0 CHECK (stock_qty >= 0),
    weight_kg     NUMERIC(5,2),
    created_at    TIMESTAMP NOT NULL DEFAULT NOW()
);
```

**Key rules:**
- `SERIAL` is shorthand for an auto-incrementing integer with a sequence.
- Table-level constraints let you name them: `CONSTRAINT chk_price CHECK (price > 0)`.
- PostgreSQL supports `IF NOT EXISTS`: `CREATE TABLE IF NOT EXISTS ...`

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/sql-createtable.html

#### ALTER TABLE

```sql
-- Add a column
ALTER TABLE table_name ADD COLUMN column_name data_type [constraints];

-- Drop a column
ALTER TABLE table_name DROP COLUMN column_name [CASCADE];

-- Rename a column
ALTER TABLE table_name RENAME COLUMN old_name TO new_name;

-- Change data type
ALTER TABLE table_name ALTER COLUMN column_name TYPE new_data_type;

-- Add a constraint
ALTER TABLE table_name ADD CONSTRAINT constraint_name constraint_definition;

-- Drop a constraint
ALTER TABLE table_name DROP CONSTRAINT constraint_name;

-- Set/drop default
ALTER TABLE table_name ALTER COLUMN column_name SET DEFAULT value;
ALTER TABLE table_name ALTER COLUMN column_name DROP DEFAULT;

-- Set/drop NOT NULL
ALTER TABLE table_name ALTER COLUMN column_name SET NOT NULL;
ALTER TABLE table_name ALTER COLUMN column_name DROP NOT NULL;
```

**TrailShop example:**

```sql
-- Add a description column to products
ALTER TABLE products ADD COLUMN description TEXT;

-- Add a unique constraint on product names within a category
ALTER TABLE products
    ADD CONSTRAINT uq_product_name_category UNIQUE (product_name, category_id);

-- Change stock_qty to allow larger numbers
ALTER TABLE products ALTER COLUMN stock_qty TYPE BIGINT;
```

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/sql-altertable.html

#### DROP TABLE

```sql
DROP TABLE [IF EXISTS] table_name [CASCADE | RESTRICT];
```

- `CASCADE` removes dependent objects (foreign keys, views).
- `RESTRICT` (default) refuses if dependencies exist.

```sql
-- Remove the old reviews table
DROP TABLE IF EXISTS product_reviews CASCADE;
```

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/sql-droptable.html

---

### 1.2 DML — Data Manipulation Language

DML commands modify the data stored in tables.

> 📖 Watt & Eng, Chapter 15: "SQL Data Manipulation"

#### INSERT

```sql
-- Single row
INSERT INTO table_name (col1, col2, ...)
VALUES (val1, val2, ...);

-- Multiple rows
INSERT INTO table_name (col1, col2, ...)
VALUES
    (val1a, val2a, ...),
    (val1b, val2b, ...),
    (val1c, val2c, ...);

-- Insert from a query
INSERT INTO table_name (col1, col2)
SELECT colA, colB FROM other_table WHERE condition;

-- Returning inserted data
INSERT INTO table_name (col1, col2)
VALUES (val1, val2)
RETURNING *;
```

**TrailShop example:**

```sql
INSERT INTO categories (category_name, description)
VALUES
    ('Hiking', 'Boots, poles, and accessories for trail hiking'),
    ('Climbing', 'Ropes, harnesses, and carabiners'),
    ('Camping', 'Tents, sleeping bags, and cookware')
RETURNING category_id, category_name;
```

**Common patterns:**
- Use `RETURNING` to get generated IDs back immediately.
- Use `ON CONFLICT` for upserts:

```sql
INSERT INTO products (product_id, product_name, price)
VALUES (1, 'TrailMaster X4', 159.99)
ON CONFLICT (product_id)
DO UPDATE SET price = EXCLUDED.price;
```

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/sql-insert.html

#### UPDATE

```sql
UPDATE table_name
SET col1 = value1,
    col2 = value2
WHERE condition
RETURNING *;
```

**TrailShop example:**

```sql
-- 10% price increase for all climbing gear
UPDATE products
SET price = price * 1.10,
    updated_at = NOW()
WHERE category_id = (SELECT category_id FROM categories WHERE category_name = 'Climbing')
RETURNING product_name, price;
```

**Rules:**
- Always include a `WHERE` clause unless you intentionally want to update every row.
- You can use subqueries in the `SET` clause or `WHERE` clause.
- `RETURNING` shows what was modified.

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/sql-update.html

#### DELETE

```sql
DELETE FROM table_name
WHERE condition
RETURNING *;
```

**TrailShop example:**

```sql
-- Remove discontinued products with zero stock
DELETE FROM products
WHERE stock_qty = 0 AND discontinued = TRUE
RETURNING product_id, product_name;
```

**Rules:**
- Without `WHERE`, all rows are deleted (use `TRUNCATE` for faster full-table deletion).
- Foreign key constraints may block deletion — use `ON DELETE CASCADE` or delete child rows first.

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/sql-delete.html

---

### 1.3 Queries — SELECT Statements

The `SELECT` statement is the most powerful and frequently used SQL command.

> 📖 Watt & Eng, Chapter 15: "Retrieval Queries in SQL"

#### Basic Syntax

```sql
SELECT [DISTINCT] column_list | *
FROM table_name [alias]
[WHERE condition]
[ORDER BY column [ASC|DESC]]
[LIMIT n]
[OFFSET m];
```

#### WHERE Operators

| Operator | Example |
|---|---|
| `=`, `!=`, `<>` | `price = 49.99` |
| `<`, `>`, `<=`, `>=` | `stock_qty > 0` |
| `BETWEEN ... AND ...` | `price BETWEEN 50 AND 100` |
| `IN (list)` | `category_id IN (1, 2, 3)` |
| `LIKE` / `ILIKE` | `product_name ILIKE '%trail%'` |
| `IS NULL` / `IS NOT NULL` | `description IS NOT NULL` |
| `AND`, `OR`, `NOT` | `price > 50 AND stock_qty > 0` |

#### ORDER BY

```sql
-- Multiple columns, mixed directions
SELECT product_name, price, stock_qty
FROM products
ORDER BY price DESC, product_name ASC;
```

- You can order by column position: `ORDER BY 2 DESC` (second column).
- `NULLS FIRST` / `NULLS LAST` controls NULL ordering.

#### LIMIT and OFFSET

```sql
-- Page 3 of results (10 per page)
SELECT product_name, price
FROM products
ORDER BY product_name
LIMIT 10 OFFSET 20;
```

- Always use `ORDER BY` with `LIMIT`/`OFFSET` for deterministic results.

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/sql-select.html

---

### 1.4 Aggregation

Aggregate functions compute a single result from a set of rows.

> 📖 Watt & Eng, Chapter 15: "Aggregate Functions and Grouping"

#### Aggregate Functions

| Function | Purpose | NULL handling |
|---|---|---|
| `COUNT(*)` | Count all rows | Counts NULLs |
| `COUNT(col)` | Count non-NULL values | Ignores NULLs |
| `COUNT(DISTINCT col)` | Count unique non-NULL values | Ignores NULLs |
| `SUM(col)` | Total of numeric values | Ignores NULLs |
| `AVG(col)` | Average of numeric values | Ignores NULLs |
| `MIN(col)` | Smallest value | Ignores NULLs |
| `MAX(col)` | Largest value | Ignores NULLs |

#### GROUP BY

```sql
SELECT category_id, COUNT(*) AS product_count, AVG(price) AS avg_price
FROM products
GROUP BY category_id;
```

**Rules:**
- Every column in `SELECT` must either be in `GROUP BY` or inside an aggregate function.
- You can group by multiple columns: `GROUP BY category_id, supplier_id`.

#### HAVING

```sql
-- Categories with more than 5 products
SELECT category_id, COUNT(*) AS product_count
FROM products
GROUP BY category_id
HAVING COUNT(*) > 5;
```

**WHERE vs HAVING:**
- `WHERE` filters individual rows *before* grouping.
- `HAVING` filters groups *after* aggregation.

**TrailShop example:**

```sql
-- Find customers who spent more than €500 total
SELECT c.customer_id, c.first_name, c.last_name,
       SUM(oi.quantity * oi.unit_price) AS total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.first_name, c.last_name
HAVING SUM(oi.quantity * oi.unit_price) > 500
ORDER BY total_spent DESC;
```

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/functions-aggregate.html

---

### 1.5 Joins

Joins combine rows from two or more tables based on a related column.

> 📖 Watt & Eng, Chapter 15: "Join Operations"

#### Join Types Comparison

| Join Type | Returns | Use When |
|---|---|---|
| `INNER JOIN` | Only matching rows from both tables | You only want rows that have a match |
| `LEFT JOIN` | All rows from left + matches from right | You want all left rows, even without matches |
| `RIGHT JOIN` | All rows from right + matches from left | Rarely used — rewrite as LEFT JOIN |
| `FULL OUTER JOIN` | All rows from both, NULLs where no match | You need a complete picture of both sides |
| `CROSS JOIN` | Cartesian product (every combination) | Generating combinations, test data |

#### Syntax

```sql
-- INNER JOIN
SELECT p.product_name, c.category_name
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id;

-- LEFT JOIN
SELECT c.category_name, p.product_name
FROM categories c
LEFT JOIN products p ON c.category_id = p.category_id;

-- RIGHT JOIN
SELECT p.product_name, s.supplier_name
FROM products p
RIGHT JOIN suppliers s ON p.supplier_id = s.supplier_id;

-- FULL OUTER JOIN
SELECT c.customer_id, o.order_id
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id;

-- CROSS JOIN
SELECT s.size_label, co.color_name
FROM sizes s
CROSS JOIN colors co;
```

#### Multiple Joins

```sql
SELECT o.order_id, c.first_name, p.product_name, oi.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE o.order_date >= '2026-01-01'
ORDER BY o.order_date DESC;
```

#### Self Join

```sql
-- Products in the same category (find related products)
SELECT a.product_name AS product, b.product_name AS related_product
FROM products a
JOIN products b ON a.category_id = b.category_id AND a.product_id < b.product_id;
```

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-JOIN

---

### 1.6 Subqueries

A subquery is a query nested inside another query.

> 📖 Watt & Eng, Chapter 15: "Nested Queries / Subqueries"

#### Types of Subqueries

| Type | Returns | Used In |
|---|---|---|
| Scalar | Single value | `SELECT`, `WHERE`, `HAVING` |
| Row | Single row | `WHERE` with row comparison |
| Table | Multiple rows/columns | `FROM`, `IN`, `EXISTS` |
| Correlated | References outer query | `WHERE`, `SELECT` |

#### Scalar Subquery

```sql
-- Products priced above average
SELECT product_name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

#### IN / NOT IN

```sql
-- Customers who have placed orders
SELECT first_name, last_name
FROM customers
WHERE customer_id IN (SELECT DISTINCT customer_id FROM orders);
```

#### EXISTS / NOT EXISTS

```sql
-- Categories that have at least one product
SELECT category_name
FROM categories c
WHERE EXISTS (
    SELECT 1 FROM products p WHERE p.category_id = c.category_id
);

-- Customers who have never ordered
SELECT first_name, last_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

#### ANY / ALL

```sql
-- Products cheaper than ALL climbing gear
SELECT product_name, price
FROM products
WHERE price < ALL (
    SELECT price FROM products
    WHERE category_id = (SELECT category_id FROM categories WHERE category_name = 'Climbing')
);

-- Products cheaper than ANY climbing gear (at least one)
SELECT product_name, price
FROM products
WHERE price < ANY (
    SELECT price FROM products
    WHERE category_id = (SELECT category_id FROM categories WHERE category_name = 'Climbing')
);
```

#### Correlated Subquery

```sql
-- For each product, show how its price compares to category average
SELECT product_name, price,
       (SELECT AVG(p2.price) FROM products p2 WHERE p2.category_id = p.category_id) AS category_avg
FROM products p;
```

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/functions-subquery.html

---

### 1.7 Set Operations

Set operations combine results from two or more queries.

> 📖 Watt & Eng, Chapter 15: "Set Operations"

#### Syntax

```sql
query1 UNION [ALL] query2
query1 INTERSECT [ALL] query2
query1 EXCEPT [ALL] query2
```

#### Rules

- Both queries must return the **same number of columns**.
- Corresponding columns must have **compatible data types**.
- Without `ALL`, duplicates are removed (like `DISTINCT`).
- Column names come from the **first** query.
- Use `ORDER BY` only at the very end (applies to the combined result).

#### Examples

```sql
-- All people (customers and employees) in one list
SELECT first_name, last_name, 'Customer' AS role FROM customers
UNION
SELECT first_name, last_name, 'Employee' AS role FROM employees
ORDER BY last_name;

-- Customers who are also employees
SELECT first_name, last_name FROM customers
INTERSECT
SELECT first_name, last_name FROM employees;

-- Customers who are NOT employees
SELECT first_name, last_name FROM customers
EXCEPT
SELECT first_name, last_name FROM employees;
```

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/queries-union.html

---

### 1.8 Conditional Expressions

#### CASE

```sql
-- Simple CASE
SELECT product_name,
    CASE category_id
        WHEN 1 THEN 'Hiking'
        WHEN 2 THEN 'Climbing'
        WHEN 3 THEN 'Camping'
        ELSE 'Other'
    END AS category_label
FROM products;

-- Searched CASE
SELECT product_name, price,
    CASE
        WHEN price >= 200 THEN 'Premium'
        WHEN price >= 100 THEN 'Mid-range'
        WHEN price >= 50  THEN 'Budget'
        ELSE 'Bargain'
    END AS price_tier
FROM products;
```

#### COALESCE

Returns the first non-NULL argument:

```sql
SELECT product_name, COALESCE(description, 'No description available') AS description
FROM products;
```

#### NULLIF

Returns NULL if both arguments are equal:

```sql
-- Avoid division by zero
SELECT product_name, total_revenue / NULLIF(total_orders, 0) AS revenue_per_order
FROM product_stats;
```

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/functions-conditional.html

---

### 1.9 Views

A view is a stored query that behaves like a virtual table.

> 📖 Watt & Eng, Chapter 16: "Database Views"

```sql
CREATE [OR REPLACE] VIEW view_name AS
SELECT ...;
```

**TrailShop example:**

```sql
CREATE OR REPLACE VIEW v_product_catalog AS
SELECT p.product_id, p.product_name, c.category_name,
       p.price, p.stock_qty,
       CASE WHEN p.stock_qty > 0 THEN 'In Stock' ELSE 'Out of Stock' END AS availability
FROM products p
JOIN categories c ON p.category_id = c.category_id;
```

**Benefits:**
- Simplifies complex queries for application code.
- Provides a security layer (expose only certain columns).
- Encapsulates business logic.

**Limitations:**
- Not all views are updatable (those with joins, aggregates, DISTINCT are read-only by default).
- No performance gain by default — the underlying query still runs each time.

```sql
-- Using a view
SELECT * FROM v_product_catalog WHERE category_name = 'Hiking';

-- Drop a view
DROP VIEW IF EXISTS v_product_catalog;
```

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/sql-createview.html

---

### 1.10 CTEs — Common Table Expressions

CTEs let you define temporary named result sets within a query using the `WITH` clause.

> 📖 Watt & Eng, Chapter 15: "Common Table Expressions"

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ... FROM cte_name;
```

#### Chaining CTEs

```sql
WITH monthly_sales AS (
    SELECT DATE_TRUNC('month', o.order_date) AS month,
           SUM(oi.quantity * oi.unit_price) AS revenue
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    GROUP BY DATE_TRUNC('month', o.order_date)
),
ranked_months AS (
    SELECT month, revenue,
           RANK() OVER (ORDER BY revenue DESC) AS rank
    FROM monthly_sales
)
SELECT month, revenue, rank
FROM ranked_months
WHERE rank <= 3;
```

**Benefits of CTEs:**
- Improve readability of complex queries.
- Allow step-by-step logic.
- Can be referenced multiple times in the main query.
- Recursive CTEs support hierarchical data (e.g., category trees).

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/queries-with.html

---

### 1.11 Transactions

A transaction groups multiple SQL statements into an atomic unit.

> 📖 Watt & Eng, Chapter 16: "Transactions and Concurrency"

```sql
BEGIN;
    -- statements here
COMMIT;
-- or ROLLBACK; to undo
```

#### SAVEPOINT

```sql
BEGIN;
    INSERT INTO orders (customer_id, order_date) VALUES (1, NOW());

    SAVEPOINT before_items;

    INSERT INTO order_items (order_id, product_id, quantity, unit_price)
    VALUES (currval('orders_order_id_seq'), 5, 2, 89.99);

    -- Oops, wrong product — roll back just the item
    ROLLBACK TO SAVEPOINT before_items;

    INSERT INTO order_items (order_id, product_id, quantity, unit_price)
    VALUES (currval('orders_order_id_seq'), 7, 2, 89.99);

COMMIT;
```

**ACID properties:**
- **Atomicity** — all or nothing
- **Consistency** — database stays valid
- **Isolation** — concurrent transactions don't interfere
- **Durability** — committed data survives crashes

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/tutorial-transactions.html

---

### 1.12 Indexes

Indexes speed up data retrieval at the cost of extra storage and slower writes.

> 📖 Watt & Eng, Chapter 16: "Indexing"

```sql
-- Create an index
CREATE INDEX idx_products_category ON products(category_id);

-- Unique index
CREATE UNIQUE INDEX idx_products_name_cat ON products(product_name, category_id);

-- Partial index (only index rows meeting a condition)
CREATE INDEX idx_products_in_stock ON products(product_id) WHERE stock_qty > 0;

-- Drop an index
DROP INDEX IF EXISTS idx_products_category;
```

#### EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE
SELECT product_name, price
FROM products
WHERE category_id = 2 AND price > 100;
```

This shows:
- The query plan (Seq Scan vs Index Scan)
- Estimated vs actual rows
- Execution time in milliseconds

**When to index:**
- Columns frequently used in `WHERE`, `JOIN`, `ORDER BY`
- Foreign key columns
- Columns used in `GROUP BY`

**When NOT to index:**
- Very small tables
- Columns with very low cardinality (e.g., Boolean)
- Tables with mostly write operations

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/indexes.html

---

### 1.13 Administration

#### Roles and Permissions

```sql
-- Create a role
CREATE ROLE app_readonly WITH LOGIN PASSWORD 'secret123';

-- Grant privileges
GRANT CONNECT ON DATABASE trailshop TO app_readonly;
GRANT USAGE ON SCHEMA public TO app_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_readonly;

-- Grant specific privileges
GRANT SELECT, INSERT, UPDATE ON products TO app_user;

-- Revoke privileges
REVOKE DELETE ON products FROM app_user;

-- Drop a role
DROP ROLE IF EXISTS app_readonly;
```

#### Backup and Restore

```bash
# Backup (plain SQL format)
pg_dump -U postgres -d trailshop -f trailshop_backup.sql

# Backup (custom compressed format)
pg_dump -U postgres -d trailshop -Fc -f trailshop_backup.dump

# Restore from plain SQL
psql -U postgres -d trailshop_new -f trailshop_backup.sql

# Restore from custom format
pg_restore -U postgres -d trailshop_new trailshop_backup.dump
```

> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/sql-grant.html
> 🔗 PostgreSQL docs: https://www.postgresql.org/docs/current/app-pgdump.html

---

## 2. Query Execution Order

When PostgreSQL processes a `SELECT` statement, it evaluates clauses in a specific logical order — **not** the order you write them:

```
┌─────────────────────────────────────────────────────────┐
│  Logical Execution Order                                │
├─────────────────────────────────────────────────────────┤
│  1. FROM        → Identify source tables                │
│  2. JOIN        → Combine tables based on ON condition  │
│  3. WHERE       → Filter individual rows                │
│  4. GROUP BY    → Form groups                           │
│  5. HAVING      → Filter groups                         │
│  6. SELECT      → Evaluate expressions, aliases         │
│  7. DISTINCT    → Remove duplicate rows                 │
│  8. ORDER BY    → Sort the result                       │
│  9. LIMIT/OFFSET→ Return a subset of rows              │
└─────────────────────────────────────────────────────────┘
```

**Why this matters:**
- You can't use a `SELECT` alias in `WHERE` (it hasn't been computed yet).
- You *can* use a `SELECT` alias in `ORDER BY` (it runs after `SELECT`).
- `HAVING` can only reference columns in `GROUP BY` or aggregate results.
- `WHERE` cannot contain aggregate functions — use `HAVING` instead.

**Example — this FAILS:**

```sql
SELECT product_name, price * 0.9 AS discounted
FROM products
WHERE discounted < 100;  -- ERROR: column "discounted" does not exist
```

**Fix — use the expression or a subquery/CTE:**

```sql
SELECT product_name, price * 0.9 AS discounted
FROM products
WHERE price * 0.9 < 100;
```

---

## 3. SQL Style Guide

Consistent formatting makes SQL readable and maintainable. Follow these conventions in your TrailShop project:

### Capitalization

- **Keywords**: UPPERCASE — `SELECT`, `FROM`, `WHERE`, `JOIN`, `ON`, `INSERT`, `UPDATE`, `DELETE`
- **Identifiers**: lowercase_snake_case — `product_name`, `order_id`, `category_id`
- **Aliases**: short lowercase — `p`, `c`, `o`, `oi`

### Indentation

- Use **4 spaces** (no tabs).
- Each major clause starts on a new line, left-aligned.
- Continuation lines are indented.

```sql
SELECT p.product_name,
       c.category_name,
       p.price
FROM products p
JOIN categories c ON p.category_id = c.category_id
WHERE p.price > 50
  AND p.stock_qty > 0
ORDER BY p.price DESC
LIMIT 20;
```

### Commas

- Place commas **at the end** of lines (trailing commas).
- One column per line in long `SELECT` lists.

### Commenting

```sql
-- Single-line comment for brief notes

/*
 * Multi-line comment for
 * explaining complex logic.
 */
```

### Naming Conventions

| Object | Convention | Example |
|---|---|---|
| Tables | Plural nouns | `products`, `order_items` |
| Columns | Singular descriptive | `product_name`, `unit_price` |
| Primary key | `table_singular_id` | `product_id`, `order_id` |
| Foreign key | Same as referenced PK | `category_id` in `products` |
| Indexes | `idx_table_columns` | `idx_products_category` |
| Views | `v_description` | `v_product_catalog` |
| Constraints | `type_table_columns` | `chk_products_price`, `uq_products_name` |

---

## 4. Key Terms — Master SQL Glossary

| Term | Definition |
|---|---|
| **DDL** | Data Definition Language — CREATE, ALTER, DROP |
| **DML** | Data Manipulation Language — INSERT, UPDATE, DELETE |
| **DQL** | Data Query Language — SELECT |
| **DCL** | Data Control Language — GRANT, REVOKE |
| **TCL** | Transaction Control Language — BEGIN, COMMIT, ROLLBACK |
| **Aggregate function** | Computes a single value from a set of rows |
| **Alias** | Temporary name for a table or column in a query |
| **Cartesian product** | Every combination of rows from two tables (CROSS JOIN) |
| **Correlated subquery** | A subquery that references columns from the outer query |
| **CTE** | Common Table Expression — a named temporary result set |
| **DISTINCT** | Removes duplicate rows from a result set |
| **Foreign key** | A column that references the primary key of another table |
| **Index** | A data structure that speeds up row retrieval |
| **Join** | Combines rows from multiple tables based on a condition |
| **NULL** | Represents missing or unknown data |
| **Primary key** | Unique identifier for each row in a table |
| **Query plan** | The database engine's strategy for executing a query |
| **RETURNING** | PostgreSQL clause that returns affected rows after DML |
| **Scalar subquery** | A subquery that returns exactly one value |
| **Schema** | A namespace that contains database objects |
| **Sequence** | Auto-incrementing number generator (behind SERIAL) |
| **Transaction** | A group of statements executed as one atomic unit |
| **View** | A stored query that behaves like a virtual table |
| **Window function** | Performs calculation across a set of rows related to current row |

---

## 5. Recommended Reading

- Watt & Eng, *Database Design — 2nd Edition*, Chapters 14–16 (SQL DDL, DML, Administration)
- PostgreSQL Official Tutorial: https://www.postgresql.org/docs/current/tutorial.html
- PostgreSQL SQL Reference: https://www.postgresql.org/docs/current/sql.html
- *SQL Style Guide* by Simon Holywell: https://www.sqlstyle.guide/

---

## 6. Summary

You've just reviewed the entire SQL toolkit that you've built up over 14 weeks:

- **DDL** to define structure (CREATE, ALTER, DROP)
- **DML** to manage data (INSERT, UPDATE, DELETE)
- **Queries** from simple SELECT to complex multi-join aggregations
- **Subqueries and CTEs** for breaking problems into steps
- **Set operations** for combining query results
- **Conditional logic** with CASE, COALESCE, NULLIF
- **Views** for abstraction and security
- **Transactions** for data integrity
- **Indexes** for performance
- **Roles and backups** for administration

You know the logical execution order. You have a style guide. You have a glossary.

**Next week** is the final exam. Use the optional exercises in this week's materials for self-paced practice — no submission is required.

---

*Next chapter: Week 51 — "The Journey's End" (Final Exam)*
