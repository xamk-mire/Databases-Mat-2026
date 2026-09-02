# Week 44 — Advanced SQL

## Chapter 8: "Levelling Up"

TrailShop is growing. You've built the core tables, inserted data, and written basic queries with WHERE clauses and simple JOINs. But the founders keep asking harder questions: "Which products have *never* been ordered?" "Show me each customer's spending compared to the average." "Can we get a single report that combines online and in-store sales?"

These questions can't be answered with a simple SELECT … WHERE. You need to reach for more powerful SQL tools — multi-table joins, subqueries, set operations, conditional logic, views, and Common Table Expressions (CTEs). This chapter is your toolkit upgrade.

As Watt & Eng write in *Database Design — 2nd Edition*, Chapter 16: "SQL provides a rich set of query capabilities that allow complex data retrieval through subqueries, set operations, and conditional expressions." We'll explore each of these in depth.

---

## Learning Objectives

By the end of this chapter you will be able to:

- Join three or more tables in a single query and explain the execution chain
- Use self-joins to compare rows within the same table
- Apply LEFT, RIGHT, and FULL OUTER JOINs to find unmatched rows
- Write subqueries in WHERE, FROM, SELECT, and HAVING clauses
- Choose between correlated and uncorrelated subqueries
- Use IN, EXISTS, NOT EXISTS, ANY, and ALL with subqueries
- Combine result sets with UNION, INTERSECT, and EXCEPT
- Write conditional expressions with CASE, COALESCE, NULLIF, GREATEST, and LEAST
- Create and manage views, including updatable views and materialized views
- Organize complex queries using CTEs (WITH clause)

---

## 1. Multi-Table Joins Revisited

### 1.1 The Join Chain — Connecting 3+ Tables

You already know how to join two tables. But real questions often span three, four, or more tables. The principle is simple: **each JOIN adds one more table to the result, linking it to a table already in the chain**.

Think of it like building a bridge network. Each JOIN clause is a bridge between two islands (tables). You can cross from island to island through the bridges.

#### The TrailShop Schema Reminder

```
customers (customer_id PK, first_name, last_name, email, city)
orders (order_id PK, customer_id FK, order_date, status)
order_items (order_item_id PK, order_id FK, product_id FK, quantity, unit_price)
products (product_id PK, product_name, category_id FK, price, stock_quantity)
categories (category_id PK, category_name)
```

#### Example: "List each customer's name, the products they ordered, and the category of each product"

This requires **four** tables: customers → orders → order_items → products. Optionally five if you want category names.

```sql
SELECT
    c.first_name,
    c.last_name,
    p.product_name,
    cat.category_name,
    oi.quantity
FROM customers c
JOIN orders o        ON o.customer_id = c.customer_id
JOIN order_items oi  ON oi.order_id = o.order_id
JOIN products p      ON p.product_id = oi.product_id
JOIN categories cat  ON cat.category_id = p.category_id
ORDER BY c.last_name, p.product_name;
```

**How to read this step by step:**

1. Start with `customers` — this is your anchor table.
2. `JOIN orders` — links each customer to their orders via `customer_id`.
3. `JOIN order_items` — links each order to its line items via `order_id`.
4. `JOIN products` — links each line item to its product via `product_id`.
5. `JOIN categories` — links each product to its category via `category_id`.

Each step narrows or expands the result depending on whether matching rows exist. With INNER JOIN (the default), only rows that match *at every step* appear in the final result.

#### Tips for Multi-Table Joins

- **Always use table aliases** (c, o, oi, p, cat) — they keep the query readable.
- **Think about the relationship path**: follow the foreign keys from one table to the next.
- **Order of JOINs doesn't affect results** (for INNER JOINs), but it affects readability. Put the logical chain in order.
- **Performance**: PostgreSQL's query optimizer reorders joins for efficiency regardless of how you write them, but clear logical ordering helps *you*.

### 1.2 Self-Joins

A **self-join** joins a table to itself. You treat the same table as if it were two separate tables by giving each copy a different alias.

#### When Do You Need a Self-Join?

- Comparing rows within the same table
- Hierarchical data (employees and managers, categories and subcategories)
- Finding pairs or duplicates

#### Example: "Find products that are in the same category and cost within €5 of each other"

```sql
SELECT
    p1.product_name AS product_a,
    p2.product_name AS product_b,
    p1.price AS price_a,
    p2.price AS price_b,
    cat.category_name
FROM products p1
JOIN products p2 ON p1.category_id = p2.category_id
                AND p1.product_id < p2.product_id
JOIN categories cat ON cat.category_id = p1.category_id
WHERE ABS(p1.price - p2.price) <= 5.00;
```

**Key detail**: `p1.product_id < p2.product_id` prevents duplicate pairs (A,B and B,A) and self-pairs (A,A).

#### Example: "Find customers in the same city"

```sql
SELECT
    c1.first_name || ' ' || c1.last_name AS customer_1,
    c2.first_name || ' ' || c2.last_name AS customer_2,
    c1.city
FROM customers c1
JOIN customers c2 ON c1.city = c2.city
                 AND c1.customer_id < c2.customer_id
ORDER BY c1.city;
```

---

## 2. Outer Joins in Depth

### 2.1 The Problem INNER JOIN Can't Solve

INNER JOIN only returns rows that have a match in *both* tables. But sometimes you need to see **all** rows from one table, even if they have no match in the other.

Real questions that require outer joins:
- "Which customers have never placed an order?"
- "Show all products, including those that have never been sold."
- "List all categories, even empty ones."

### 2.2 LEFT JOIN (LEFT OUTER JOIN)

Returns **all rows from the left table**, plus matched rows from the right table. If there's no match, the right-side columns are NULL.

```sql
SELECT
    c.first_name,
    c.last_name,
    o.order_id,
    o.order_date
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
ORDER BY c.last_name;
```

Customers who have never ordered appear with NULL in `order_id` and `order_date`.

### 2.3 The "Find Unmatched Rows" Pattern

This is one of the most useful patterns in SQL:

```sql
-- Customers who have NEVER placed an order
SELECT
    c.first_name,
    c.last_name,
    c.email
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
WHERE o.order_id IS NULL;
```

**How it works:**
1. LEFT JOIN includes all customers, even those without orders.
2. For customers without orders, all `orders` columns are NULL.
3. `WHERE o.order_id IS NULL` filters to *only* those unmatched rows.

This pattern is equivalent to `NOT EXISTS` (covered later in subqueries), but many developers find it more intuitive.

#### More TrailShop examples:

```sql
-- Products never ordered
SELECT p.product_name, p.price
FROM products p
LEFT JOIN order_items oi ON oi.product_id = p.product_id
WHERE oi.order_item_id IS NULL;

-- Categories with no products
SELECT cat.category_name
FROM categories cat
LEFT JOIN products p ON p.category_id = cat.category_id
WHERE p.product_id IS NULL;
```

### 2.4 RIGHT JOIN (RIGHT OUTER JOIN)

Returns **all rows from the right table**, plus matched rows from the left table. Logically equivalent to LEFT JOIN with the tables swapped.

```sql
-- Equivalent to the LEFT JOIN above, just reversed
SELECT
    c.first_name,
    c.last_name,
    o.order_id
FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.customer_id;
```

**In practice**: Most developers prefer LEFT JOIN and simply put the "all rows" table on the left. RIGHT JOIN is rarely used but exists for completeness.

### 2.5 FULL OUTER JOIN

Returns **all rows from both tables**. Rows without a match on either side get NULLs in the other table's columns.

```sql
-- Show all customers and all orders, even if somehow disconnected
SELECT
    c.first_name,
    c.last_name,
    o.order_id,
    o.order_date
FROM customers c
FULL OUTER JOIN orders o ON o.customer_id = c.customer_id;
```

FULL OUTER JOIN is less common in practice. It's useful when you're reconciling two data sets (e.g., comparing imported data against existing records).

### 2.6 Outer Joins with Multiple Tables

When chaining outer joins with inner joins, **order matters**. An INNER JOIN later in the chain can eliminate rows that the LEFT JOIN preserved.

```sql
-- WRONG: The inner join on categories eliminates products without a category
SELECT c.first_name, p.product_name, cat.category_name
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
LEFT JOIN order_items oi ON oi.order_id = o.order_id
LEFT JOIN products p ON p.product_id = oi.product_id
JOIN categories cat ON cat.category_id = p.category_id;  -- This kills NULLs!

-- CORRECT: Use LEFT JOIN throughout
SELECT c.first_name, p.product_name, cat.category_name
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
LEFT JOIN order_items oi ON oi.order_id = o.order_id
LEFT JOIN products p ON p.product_id = oi.product_id
LEFT JOIN categories cat ON cat.category_id = p.category_id;
```

---

## 3. Subqueries — Comprehensive

### 3.1 What Is a Subquery?

A **subquery** (also called an inner query or nested query) is a SELECT statement embedded inside another SQL statement. The outer statement is called the **outer query** or **main query**.

As Watt & Eng explain in Chapter 16 of *Database Design — 2nd Edition*, subqueries allow you to build complex queries by nesting simpler ones, using the result of one query as input to another.

Subqueries are enclosed in parentheses and can appear in several places:
- In the `WHERE` clause (most common)
- In the `FROM` clause (derived tables)
- In the `SELECT` clause (scalar subqueries)
- In the `HAVING` clause

### 3.2 Types of Subqueries

| Type | Returns | Example Use |
|------|---------|-------------|
| Scalar subquery | Single value (one row, one column) | Compare with =, <, > |
| Row subquery | Single row (multiple columns) | Compare with row constructors |
| Table subquery | Multiple rows and columns | Use with IN, EXISTS, or in FROM |
| Correlated subquery | Depends on outer row | Executes once per outer row |

### 3.3 Scalar Subqueries

A scalar subquery returns exactly **one value**. You can use it anywhere a single value is expected.

#### Example: "Products priced above the average"

```sql
SELECT product_name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

The subquery `(SELECT AVG(price) FROM products)` runs first, returns a single number (say, 45.50), and then the outer query becomes `WHERE price > 45.50`.

#### Example: "The most recent order for customer 1"

```sql
SELECT order_id, order_date, status
FROM orders
WHERE customer_id = 1
  AND order_date = (
      SELECT MAX(order_date)
      FROM orders
      WHERE customer_id = 1
  );
```

#### Scalar Subquery in SELECT

```sql
SELECT
    product_name,
    price,
    price - (SELECT AVG(price) FROM products) AS diff_from_avg
FROM products
ORDER BY diff_from_avg DESC;
```

### 3.4 Table Subqueries (Derived Tables)

A subquery in the `FROM` clause creates a temporary table that the outer query can select from. PostgreSQL **requires** you to give it an alias.

```sql
-- Average order total by customer, then find customers above $200
SELECT
    sub.first_name,
    sub.last_name,
    sub.total_spent
FROM (
    SELECT
        c.first_name,
        c.last_name,
        SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM customers c
    JOIN orders o ON o.customer_id = c.customer_id
    JOIN order_items oi ON oi.order_id = o.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
) AS sub
WHERE sub.total_spent > 200
ORDER BY sub.total_spent DESC;
```

**Note the `AS sub`** — without it, PostgreSQL returns an error: `ERROR: subquery in FROM must have an alias`.

### 3.5 Correlated Subqueries

A **correlated subquery** references columns from the outer query. It executes **once for each row** in the outer query — like a loop.

#### Example: "Products priced above the average for their category"

```sql
SELECT p.product_name, p.price, p.category_id
FROM products p
WHERE p.price > (
    SELECT AVG(p2.price)
    FROM products p2
    WHERE p2.category_id = p.category_id  -- References outer query!
);
```

For each product in the outer query, PostgreSQL:
1. Looks at that product's `category_id`
2. Runs the subquery to compute the average price *for that category*
3. Compares the product's price to that average

**Performance implication**: Correlated subqueries can be slow on large tables because they execute once per outer row. Often, you can rewrite them as JOINs or window functions for better performance.

### 3.6 Subqueries with IN

`IN` checks whether a value matches **any value** in a list or subquery result.

```sql
-- Customers who have placed at least one order
SELECT first_name, last_name
FROM customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id
    FROM orders
);

-- Products in the "Footwear" or "Accessories" categories
SELECT product_name, price
FROM products
WHERE category_id IN (
    SELECT category_id
    FROM categories
    WHERE category_name IN ('Footwear', 'Accessories')
);
```

### 3.7 Subqueries with EXISTS and NOT EXISTS

`EXISTS` returns TRUE if the subquery returns **at least one row**. It doesn't care about the actual values — only whether rows exist.

```sql
-- Customers who have placed at least one order (EXISTS version)
SELECT c.first_name, c.last_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

`NOT EXISTS` is the opposite — it returns TRUE when the subquery returns **no rows**.

```sql
-- Customers who have NEVER ordered (NOT EXISTS version)
SELECT c.first_name, c.last_name, c.email
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

**Why `SELECT 1`?** Because EXISTS only checks for row *existence*, the actual columns don't matter. `SELECT 1` is a convention that signals "I only care if rows exist."

**EXISTS vs IN**: EXISTS is often faster for large subquery results because it can stop as soon as it finds one matching row. IN must build the entire list first.

### 3.8 Subqueries with ANY and ALL

#### ANY (or SOME)

`ANY` returns TRUE if the comparison is true for **at least one** value in the subquery.

```sql
-- Products more expensive than ANY product in the "Camping" category
-- (i.e., more expensive than the cheapest camping product)
SELECT product_name, price
FROM products
WHERE price > ANY (
    SELECT p.price
    FROM products p
    JOIN categories cat ON cat.category_id = p.category_id
    WHERE cat.category_name = 'Camping'
);
```

#### ALL

`ALL` returns TRUE if the comparison is true for **every** value in the subquery.

```sql
-- Products more expensive than ALL products in the "Camping" category
-- (i.e., more expensive than the MOST expensive camping product)
SELECT product_name, price
FROM products
WHERE price > ALL (
    SELECT p.price
    FROM products p
    JOIN categories cat ON cat.category_id = p.category_id
    WHERE cat.category_name = 'Camping'
);
```

**Tip**: `> ALL (subquery)` is equivalent to `> (SELECT MAX(...) FROM ...)` and `< ALL (subquery)` is equivalent to `< (SELECT MIN(...) FROM ...)`.

### 3.9 Subquery vs JOIN — Comparison

| Aspect | Subquery | JOIN |
|--------|----------|------|
| Readability | Good for "does X exist?" questions | Good for "combine data from multiple tables" |
| Performance | Correlated subqueries can be slow | Usually optimized well by PostgreSQL |
| Result columns | Can only return outer query columns | Can return columns from all joined tables |
| NULL handling | NOT EXISTS handles NULLs correctly | LEFT JOIN + IS NULL handles NULLs correctly |
| Aggregation | Natural for comparisons with aggregates | Requires GROUP BY or derived tables |

**General guidance:**
- Use **JOIN** when you need columns from both tables in the result.
- Use **EXISTS/NOT EXISTS** when you're checking for the *existence* of related rows.
- Use **scalar subqueries** when you need to compare against an aggregate.
- Use **IN** for short, simple lists. Switch to EXISTS for large or complex conditions.

PostgreSQL's optimizer can often transform subqueries into joins internally, so performance differences are sometimes minimal. Write what's **clearest** first, then optimize if needed.

---

## 4. Set Operations

Set operations combine the **results** of two or more SELECT statements into a single result set. As the PostgreSQL documentation explains ([Combining Queries](https://www.postgresql.org/docs/current/queries-union.html)), these operations treat query results as mathematical sets.

### 4.1 Rules for Set Operations

Both SELECT statements must:
1. Return the **same number of columns**
2. Have **compatible data types** in corresponding columns (e.g., both integers, or both text)

Column names in the result come from the **first** SELECT.

### 4.2 UNION

Combines results and **removes duplicates** (like a mathematical set union).

```sql
-- All cities where we have customers OR orders shipped to
SELECT city FROM customers
UNION
SELECT shipping_city FROM orders;
```

### 4.3 UNION ALL

Combines results and **keeps all duplicates**. Faster than UNION because it skips the deduplication step.

```sql
-- All product mentions: those in stock + those ordered (with duplicates)
SELECT product_name, 'in_stock' AS source
FROM products
WHERE stock_quantity > 0
UNION ALL
SELECT p.product_name, 'ordered' AS source
FROM products p
JOIN order_items oi ON oi.product_id = p.product_id;
```

**When to use UNION ALL**: When you *know* there are no duplicates, or when you *want* duplicates (e.g., counting total occurrences).

### 4.4 INTERSECT

Returns only rows that appear in **both** result sets.

```sql
-- Customers who are in both the "Helsinki" city AND have placed orders
SELECT customer_id FROM customers WHERE city = 'Helsinki'
INTERSECT
SELECT customer_id FROM orders;
```

### 4.5 EXCEPT

Returns rows from the first query that are **not** in the second query (set difference).

```sql
-- Customers who have NOT placed any order
SELECT customer_id FROM customers
EXCEPT
SELECT customer_id FROM orders;
```

This is another way to find "unmatched" rows, alongside LEFT JOIN + IS NULL and NOT EXISTS.

### 4.6 Ordering and Limiting Set Operation Results

You can add `ORDER BY` and `LIMIT` at the end, after the last SELECT:

```sql
SELECT product_name, price FROM products WHERE category_id = 1
UNION
SELECT product_name, price FROM products WHERE category_id = 2
ORDER BY price DESC
LIMIT 10;
```

### 4.7 Combining More Than Two Queries

You can chain set operations:

```sql
SELECT city FROM customers
UNION
SELECT city FROM suppliers
UNION
SELECT shipping_city FROM orders;
```

PostgreSQL evaluates them left to right. Use parentheses to change precedence if needed.

---

## 5. Conditional Expressions

### 5.1 CASE — The SQL "If/Else"

CASE lets you add conditional logic directly in your queries. There are two forms.

#### Simple CASE

Compares an expression to a list of values:

```sql
SELECT
    order_id,
    status,
    CASE status
        WHEN 'pending'   THEN 'Awaiting processing'
        WHEN 'shipped'   THEN 'On the way'
        WHEN 'delivered' THEN 'Complete'
        ELSE 'Unknown'
    END AS status_description
FROM orders;
```

#### Searched CASE

Uses boolean conditions (more flexible):

```sql
SELECT
    product_name,
    price,
    CASE
        WHEN price < 20  THEN 'Budget'
        WHEN price < 50  THEN 'Mid-range'
        WHEN price < 100 THEN 'Premium'
        ELSE 'Luxury'
    END AS price_tier
FROM products;
```

#### CASE Inside Aggregates

This powerful pattern lets you do conditional counting and summing:

```sql
SELECT
    COUNT(*) AS total_orders,
    COUNT(CASE WHEN status = 'delivered' THEN 1 END) AS delivered,
    COUNT(CASE WHEN status = 'pending' THEN 1 END) AS pending,
    COUNT(CASE WHEN status = 'cancelled' THEN 1 END) AS cancelled
FROM orders;
```

```sql
-- Revenue by price tier
SELECT
    CASE
        WHEN p.price < 30 THEN 'Budget'
        WHEN p.price < 80 THEN 'Mid-range'
        ELSE 'Premium'
    END AS tier,
    SUM(oi.quantity * oi.unit_price) AS revenue
FROM order_items oi
JOIN products p ON p.product_id = oi.product_id
GROUP BY tier
ORDER BY revenue DESC;
```

### 5.2 COALESCE

Returns the **first non-NULL** argument. Perfect for providing default values.

```sql
-- Show "No phone" if phone is NULL
SELECT
    first_name,
    COALESCE(phone, 'No phone') AS contact_phone
FROM customers;

-- Use secondary email if primary is NULL
SELECT COALESCE(primary_email, secondary_email, 'no-email@unknown.com') AS email
FROM contacts;
```

### 5.3 NULLIF

Returns NULL if the two arguments are **equal**; otherwise returns the first argument. Useful for avoiding division by zero.

```sql
-- Avoid division by zero
SELECT
    product_name,
    total_revenue / NULLIF(units_sold, 0) AS avg_price_per_unit
FROM product_stats;
```

If `units_sold` is 0, `NULLIF(units_sold, 0)` returns NULL, and dividing by NULL gives NULL instead of an error.

### 5.4 GREATEST and LEAST

Return the largest or smallest value from a list of expressions.

```sql
-- Effective price is at least €5 (floor) and at most €500 (cap)
SELECT
    product_name,
    GREATEST(LEAST(price, 500), 5) AS effective_price
FROM products;

-- Most recent date between order_date and ship_date
SELECT order_id, GREATEST(order_date, ship_date) AS latest_date
FROM orders;
```

---

## 6. Views

### 6.1 What Is a View?

A **view** is a named query stored in the database. It acts like a virtual table — when you query a view, PostgreSQL runs the underlying SELECT statement and returns the results.

As the PostgreSQL documentation explains ([CREATE VIEW](https://www.postgresql.org/docs/current/sql-createview.html)), views provide abstraction, simplify complex queries, and can control data access.

### 6.2 Why Use Views?

- **Simplify complex queries** — write the join once, reuse it by name.
- **Security** — grant access to a view instead of the underlying tables, hiding sensitive columns.
- **Abstraction** — if the table structure changes, update the view definition without changing application queries.
- **Consistency** — ensure everyone uses the same business logic.

### 6.3 Creating Views

```sql
CREATE VIEW customer_order_summary AS
SELECT
    c.customer_id,
    c.first_name || ' ' || c.last_name AS full_name,
    COUNT(o.order_id) AS order_count,
    COALESCE(SUM(oi.quantity * oi.unit_price), 0) AS total_spent
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
LEFT JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY c.customer_id, c.first_name, c.last_name;
```

Now you can query it like a table:

```sql
SELECT * FROM customer_order_summary WHERE total_spent > 100;
```

### 6.4 CREATE OR REPLACE VIEW

Modifies an existing view without having to drop it first:

```sql
CREATE OR REPLACE VIEW customer_order_summary AS
SELECT
    c.customer_id,
    c.first_name || ' ' || c.last_name AS full_name,
    c.email,  -- Added column
    COUNT(o.order_id) AS order_count,
    COALESCE(SUM(oi.quantity * oi.unit_price), 0) AS total_spent
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
LEFT JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY c.customer_id, c.first_name, c.last_name, c.email;
```

**Restriction**: You can add columns to the end, but you cannot remove or rename existing columns with `CREATE OR REPLACE`.

### 6.5 DROP VIEW

```sql
DROP VIEW customer_order_summary;
DROP VIEW IF EXISTS customer_order_summary;  -- No error if it doesn't exist
```

### 6.6 Updatable Views vs Read-Only Views

A view is **automatically updatable** in PostgreSQL if it meets these conditions:
- Based on a single table (no joins)
- No aggregate functions, GROUP BY, HAVING, DISTINCT
- No set operations (UNION, etc.)
- No window functions

```sql
-- This view is updatable (simple single-table view)
CREATE VIEW active_products AS
SELECT product_id, product_name, price, stock_quantity
FROM products
WHERE stock_quantity > 0;

-- You can INSERT, UPDATE, DELETE through it
UPDATE active_products SET price = 29.99 WHERE product_id = 5;
```

Views with JOINs, aggregates, or DISTINCT are **read-only** — you can only SELECT from them.

### 6.7 WITH CHECK OPTION

Prevents INSERT/UPDATE through a view if the new row wouldn't be visible through the view:

```sql
CREATE VIEW active_products AS
SELECT product_id, product_name, price, stock_quantity
FROM products
WHERE stock_quantity > 0
WITH CHECK OPTION;

-- This would FAIL because stock_quantity = 0 doesn't satisfy the view's WHERE clause
UPDATE active_products SET stock_quantity = 0 WHERE product_id = 5;
-- ERROR: new row violates check option for view "active_products"
```

### 6.8 Materialized Views (Brief Introduction)

A **materialized view** stores the query result physically on disk (like a cached snapshot). It's faster to query but must be refreshed manually.

```sql
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT
    DATE_TRUNC('month', o.order_date) AS month,
    SUM(oi.quantity * oi.unit_price) AS revenue
FROM orders o
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY month;

-- Refresh when data changes
REFRESH MATERIALIZED VIEW monthly_sales;
```

Use materialized views for expensive reports that don't need real-time data.

### 6.9 Three TrailShop Views

```sql
-- View 1: Product catalog with category names
CREATE VIEW product_catalog AS
SELECT
    p.product_id,
    p.product_name,
    cat.category_name,
    p.price,
    p.stock_quantity,
    CASE
        WHEN p.stock_quantity = 0 THEN 'Out of stock'
        WHEN p.stock_quantity < 5 THEN 'Low stock'
        ELSE 'In stock'
    END AS availability
FROM products p
JOIN categories cat ON cat.category_id = p.category_id;

-- View 2: Order details (flattened for reporting)
CREATE VIEW order_details AS
SELECT
    o.order_id,
    o.order_date,
    c.first_name || ' ' || c.last_name AS customer_name,
    p.product_name,
    oi.quantity,
    oi.unit_price,
    oi.quantity * oi.unit_price AS line_total
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
JOIN products p ON p.product_id = oi.product_id;

-- View 3: Category performance
CREATE VIEW category_performance AS
SELECT
    cat.category_name,
    COUNT(DISTINCT p.product_id) AS product_count,
    COUNT(DISTINCT oi.order_item_id) AS times_ordered,
    COALESCE(SUM(oi.quantity * oi.unit_price), 0) AS total_revenue
FROM categories cat
LEFT JOIN products p ON p.category_id = cat.category_id
LEFT JOIN order_items oi ON oi.product_id = p.product_id
GROUP BY cat.category_name;
```

---

## 7. Common Table Expressions (CTEs) — The WITH Clause

### 7.1 What Is a CTE?

A **Common Table Expression** (CTE) is a temporary named result set defined within a single SQL statement using the `WITH` keyword. Think of it as creating a temporary view that only lasts for one query.

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ... FROM cte_name;
```

### 7.2 Why CTEs?

- **Readability**: Break complex queries into named, logical steps.
- **Reusability**: Reference the same CTE multiple times in the outer query.
- **Self-documentation**: The CTE name describes what the subresult represents.

### 7.3 Single CTE Example

```sql
-- Find customers whose total spending exceeds the average customer spending
WITH customer_totals AS (
    SELECT
        c.customer_id,
        c.first_name,
        c.last_name,
        SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM customers c
    JOIN orders o ON o.customer_id = c.customer_id
    JOIN order_items oi ON oi.order_id = o.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
)
SELECT
    first_name,
    last_name,
    total_spent
FROM customer_totals
WHERE total_spent > (SELECT AVG(total_spent) FROM customer_totals);
```

Notice how `customer_totals` is referenced **twice** — once in the main SELECT and once in the WHERE subquery. With a regular subquery, you'd have to write the same logic twice.

### 7.4 Multiple CTEs

You can define multiple CTEs, separated by commas. Later CTEs can reference earlier ones:

```sql
WITH
category_revenue AS (
    SELECT
        p.category_id,
        SUM(oi.quantity * oi.unit_price) AS revenue
    FROM products p
    JOIN order_items oi ON oi.product_id = p.product_id
    GROUP BY p.category_id
),
avg_category_revenue AS (
    SELECT AVG(revenue) AS avg_revenue
    FROM category_revenue
)
SELECT
    cat.category_name,
    cr.revenue,
    acr.avg_revenue,
    CASE
        WHEN cr.revenue > acr.avg_revenue THEN 'Above average'
        ELSE 'Below average'
    END AS performance
FROM category_revenue cr
JOIN categories cat ON cat.category_id = cr.category_id
CROSS JOIN avg_category_revenue acr
ORDER BY cr.revenue DESC;
```

### 7.5 Recursive CTEs (Brief Introduction)

Recursive CTEs allow a CTE to **reference itself**, enabling queries on hierarchical or graph-like data.

```sql
-- Example: Generate a number series (simple demonstration)
WITH RECURSIVE counter AS (
    SELECT 1 AS n              -- Base case (anchor)
    UNION ALL
    SELECT n + 1 FROM counter  -- Recursive step
    WHERE n < 10
)
SELECT n FROM counter;
```

A more practical example — if TrailShop had category hierarchies:

```sql
-- Hypothetical: categories with parent_id for subcategories
WITH RECURSIVE category_tree AS (
    SELECT category_id, category_name, parent_id, 0 AS depth
    FROM categories
    WHERE parent_id IS NULL  -- Top-level categories
    UNION ALL
    SELECT c.category_id, c.category_name, c.parent_id, ct.depth + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.category_id
)
SELECT * FROM category_tree ORDER BY depth, category_name;
```

Recursive CTEs need:
1. An **anchor member** (the non-recursive starting point)
2. A **recursive member** (references the CTE itself, joined with a base table)
3. A **termination condition** (usually implicit when no more rows match)

### 7.6 CTE vs Subquery — Readability Comparison

**Same query written as a derived table (subquery in FROM):**

```sql
SELECT sub.first_name, sub.last_name, sub.total_spent
FROM (
    SELECT c.first_name, c.last_name,
           SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM customers c
    JOIN orders o ON o.customer_id = c.customer_id
    JOIN order_items oi ON oi.order_id = o.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
) AS sub
WHERE sub.total_spent > 200;
```

**Same query written as a CTE:**

```sql
WITH customer_totals AS (
    SELECT c.first_name, c.last_name,
           SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM customers c
    JOIN orders o ON o.customer_id = c.customer_id
    JOIN order_items oi ON oi.order_id = o.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
)
SELECT first_name, last_name, total_spent
FROM customer_totals
WHERE total_spent > 200;
```

The CTE version reads top-to-bottom: "First, compute customer totals. Then, select those above 200." The subquery version forces you to read inside-out.

**Performance note**: In modern PostgreSQL (12+), CTEs are optimized inline by default (just like subqueries). In older versions, CTEs were always materialized (computed once and cached). You can force materialization with `WITH cte AS MATERIALIZED (...)` if needed.

---

## 8. Key Terms

| Term | Definition |
|------|-----------|
| Self-join | Joining a table to itself using different aliases |
| Outer join | JOIN that preserves unmatched rows (LEFT, RIGHT, FULL) |
| Subquery | A SELECT statement nested inside another statement |
| Correlated subquery | Subquery that references the outer query; executes per outer row |
| Scalar subquery | Subquery returning exactly one value |
| Derived table | Subquery in the FROM clause; must have an alias |
| EXISTS | Predicate that tests whether a subquery returns any rows |
| Set operation | UNION, INTERSECT, or EXCEPT combining two query results |
| CASE expression | Conditional logic that returns different values based on conditions |
| COALESCE | Returns the first non-NULL argument |
| View | Named stored query that acts as a virtual table |
| Materialized view | View whose results are physically stored and must be refreshed |
| CTE | Common Table Expression; temporary named result within WITH clause |
| Recursive CTE | CTE that references itself for hierarchical/iterative queries |

---

## 9. Reading

### Required

- Watt & Eng, *Database Design — 2nd Edition*, **Chapter 16**: SQL Subqueries, Set Operations, and Advanced Queries
- PostgreSQL Documentation: [Combining Queries (UNION, INTERSECT, EXCEPT)](https://www.postgresql.org/docs/current/queries-union.html)
- PostgreSQL Documentation: [CREATE VIEW](https://www.postgresql.org/docs/current/sql-createview.html)

### Further Reading

- PostgreSQL Documentation: [WITH Queries (CTEs)](https://www.postgresql.org/docs/current/queries-with.html)
- PostgreSQL Documentation: [Conditional Expressions](https://www.postgresql.org/docs/current/functions-conditional.html)
- PostgreSQL Documentation: [Subquery Expressions](https://www.postgresql.org/docs/current/functions-subquery.html)

---

## 10. Summary

This chapter upgraded your SQL toolkit significantly:

- **Multi-table joins** let you traverse the entire schema in one query by chaining JOIN clauses.
- **Self-joins** compare rows within the same table.
- **Outer joins** preserve unmatched rows — critical for "find what's missing" queries.
- **Subqueries** nest queries inside other queries, enabling comparisons with aggregates (scalar), existence checks (EXISTS), and membership tests (IN, ANY, ALL).
- **Set operations** combine complete result sets using UNION, INTERSECT, and EXCEPT.
- **CASE and COALESCE** add conditional logic directly in your SQL.
- **Views** store complex queries as reusable virtual tables with security benefits.
- **CTEs** make complex queries readable by breaking them into named steps.

---

## Coming Next: Week 45 — Normalization

You've been writing queries against TrailShop's tables — but who decided *this* was the right structure? Next week you'll learn the theory behind *designing* tables: normalization. You'll understand why some designs lead to data anomalies and how to systematically fix them. Get ready to audit TrailShop's schema.
