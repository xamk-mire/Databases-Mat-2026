# Week 41 — Querying Data: SELECT, WHERE, JOINs, and Aggregates

## Chapter 6: "Answering the Questions"

Last week you built the TrailShop database and filled it with data. The founders were thrilled — finally, everything is in one place, properly structured, and consistent. But now they're at your desk every morning with questions:

- "Which products cost more than €100?"
- "How many orders has Anna placed?"
- "What's our total revenue this month?"
- "Which categories have the most products?"
- "Show me all customers who haven't ordered anything yet."

These questions are the entire *point* of having a database. Data sitting in tables is only valuable when you can extract meaningful information from it. This week, you learn to unlock that value using the full power of `SELECT` — the most versatile and frequently used SQL statement.

---

## Learning Objectives

By the end of this chapter you will be able to:

- Write SELECT queries with specific columns, aliases, and DISTINCT
- Filter data using WHERE with all comparison and logical operators
- Sort results with ORDER BY (including NULLS FIRST/LAST)
- Paginate results with LIMIT/OFFSET
- Summarize data with aggregate functions (COUNT, SUM, AVG, MIN, MAX)
- Group data with GROUP BY and filter groups with HAVING
- Join multiple tables using INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL OUTER JOIN, and CROSS JOIN
- Explain the logical order of query execution
- Write complex multi-table queries

---

## 1. SELECT in Depth

> **Reference:** Watt & Eng, *Database Design — 2nd Edition*, Chapter 16, covers the SELECT statement as the primary means of retrieving data from a relational database.

### 1.1 Basic SELECT

At its simplest, SELECT retrieves data from one table:

```sql
SELECT column1, column2
FROM table_name;
```

### 1.2 SELECT * (All Columns)

```sql
SELECT * FROM products;
```

The asterisk means "all columns." This is convenient for quick exploration but **avoid it in production code** because:
- If columns are added/removed, your query returns different results
- You transfer more data than needed
- It makes code harder to understand

### 1.3 Selecting Specific Columns

```sql
SELECT name, price, stock
FROM products;
```

Always select only the columns you actually need. This is clearer and more efficient.

### 1.4 Column Aliases

Aliases rename columns in the output. Use the `AS` keyword:

```sql
SELECT name AS product_name,
       price AS unit_price_eur,
       stock AS quantity_available
FROM products;
```

The `AS` keyword is optional (but recommended for clarity):
```sql
SELECT name product_name, price unit_price_eur
FROM products;
```

Aliases are especially useful with calculated expressions:
```sql
SELECT name,
       price,
       price * 1.24 AS price_with_vat
FROM products;
```

### 1.5 DISTINCT — Removing Duplicates

DISTINCT eliminates duplicate rows from the result:

```sql
-- What statuses exist in our orders?
SELECT DISTINCT status FROM orders;

-- What categories have products?
SELECT DISTINCT category_id FROM products;
```

DISTINCT applies to the *entire row* — all selected columns must match for two rows to be considered duplicates:

```sql
-- Unique combinations of category and price range
SELECT DISTINCT category_id, 
       CASE WHEN price < 50 THEN 'budget'
            WHEN price < 200 THEN 'mid'
            ELSE 'premium' END AS price_tier
FROM products;
```

### 1.6 Expressions and Calculations

You can use expressions anywhere you'd put a column name:

```sql
SELECT name,
       price,
       stock,
       price * stock AS inventory_value
FROM products;
```

PostgreSQL supports standard arithmetic operators: `+`, `-`, `*`, `/`, `%` (modulo).

String concatenation uses `||`:
```sql
SELECT first_name || ' ' || last_name AS full_name
FROM customers;
```

---

## 2. WHERE — Filtering Rows

The WHERE clause filters which rows are returned. Only rows where the condition evaluates to TRUE are included.

> **Reference:** Watt & Eng, Chapter 16, discusses WHERE criteria: "The WHERE clause limits the rows returned by specifying conditions that must be met."

### 2.1 Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal to | `price = 149.99` |
| `<>` or `!=` | Not equal to | `status <> 'cancelled'` |
| `<` | Less than | `price < 100` |
| `>` | Greater than | `stock > 0` |
| `<=` | Less than or equal | `price <= 200` |
| `>=` | Greater than or equal | `stock >= 10` |

```sql
SELECT name, price FROM products WHERE price > 100;
SELECT name, stock FROM products WHERE stock = 0;
SELECT * FROM orders WHERE status <> 'cancelled';
```

### 2.2 Logical Operators: AND, OR, NOT

**AND** — both conditions must be true:
```sql
SELECT name, price, stock
FROM products
WHERE price > 50 AND stock > 20;
```

**OR** — at least one condition must be true:
```sql
SELECT * FROM orders
WHERE status = 'pending' OR status = 'shipped';
```

**NOT** — inverts a condition:
```sql
SELECT name FROM products
WHERE NOT category_id = 1;
```

### 2.3 Operator Precedence

SQL evaluates operators in this order:
1. `NOT`
2. `AND`
3. `OR`

This means `A OR B AND C` is evaluated as `A OR (B AND C)`, **not** `(A OR B) AND C`.

Always use parentheses to make your intent clear:
```sql
-- Products that are either cheap OR both expensive and in stock
SELECT name, price, stock FROM products
WHERE price < 30 OR (price > 200 AND stock > 0);
```

### 2.4 BETWEEN

Tests if a value falls within a range (inclusive on both ends):

```sql
SELECT name, price FROM products
WHERE price BETWEEN 50 AND 150;

-- Equivalent to:
-- WHERE price >= 50 AND price <= 150
```

### 2.5 IN

Tests if a value matches any value in a list:

```sql
SELECT * FROM orders
WHERE status IN ('pending', 'shipped');

-- Equivalent to:
-- WHERE status = 'pending' OR status = 'shipped'
```

IN is cleaner and more readable than chaining OR conditions, especially with many values.

### 2.6 LIKE and Pattern Matching

LIKE matches text against a pattern using wildcards:
- `%` matches any sequence of characters (including empty)
- `_` matches exactly one character

```sql
-- Products starting with 'Trail'
SELECT name FROM products WHERE name LIKE 'Trail%';

-- Products with 'Pack' anywhere in the name
SELECT name FROM products WHERE name LIKE '%Pack%';

-- Products with exactly 4 characters
SELECT name FROM products WHERE name LIKE '____';
```

**LIKE is case-sensitive** in PostgreSQL. For case-insensitive matching, use:

### 2.7 ILIKE (PostgreSQL Extension)

```sql
-- Case-insensitive search
SELECT name FROM products WHERE name ILIKE '%jacket%';
-- Matches: 'ThermoLayer Jacket', 'RAIN JACKET', 'jacket pro'
```

> **PostgreSQL docs:** https://www.postgresql.org/docs/current/functions-matching.html

### 2.8 SIMILAR TO (Brief)

SIMILAR TO combines LIKE-style syntax with basic regular expressions. It supports `|` for alternation and `()` for grouping:

```sql
SELECT name FROM products
WHERE name SIMILAR TO '%(Boot|Shoe|Sandal)%';
```

In practice, most developers use LIKE/ILIKE for simple patterns or PostgreSQL's `~` operator for full regex. SIMILAR TO sits awkwardly in between.

### 2.9 IS NULL / IS NOT NULL

NULL is special — you cannot compare it with `=` or `<>`. You must use IS NULL or IS NOT NULL:

```sql
-- Products without a description
SELECT name FROM products WHERE description IS NULL;

-- Products that DO have a description
SELECT name FROM products WHERE description IS NOT NULL;
```

**Common mistake:**
```sql
-- WRONG: This never returns rows (NULL = NULL is not TRUE)
SELECT name FROM products WHERE description = NULL;

-- CORRECT:
SELECT name FROM products WHERE description IS NULL;
```

---

## 3. ORDER BY

ORDER BY sorts the result set.

### 3.1 Basic Ordering

```sql
-- Cheapest first
SELECT name, price FROM products ORDER BY price;

-- Same (ASC is the default)
SELECT name, price FROM products ORDER BY price ASC;

-- Most expensive first
SELECT name, price FROM products ORDER BY price DESC;
```

### 3.2 Multiple Columns

```sql
-- Sort by category, then by price within each category
SELECT name, category_id, price
FROM products
ORDER BY category_id ASC, price DESC;
```

### 3.3 NULLS FIRST / NULLS LAST

By default, NULLs sort last in ASC order and first in DESC order. You can override this:

```sql
SELECT name, description
FROM products
ORDER BY description NULLS FIRST;

SELECT name, description
FROM products
ORDER BY description DESC NULLS LAST;
```

### 3.4 Ordering by Expression

You can ORDER BY a calculation or expression:

```sql
SELECT name, price, stock, price * stock AS inventory_value
FROM products
ORDER BY price * stock DESC;
```

You can also reference by column position (less readable, but valid):
```sql
SELECT name, price FROM products ORDER BY 2 DESC;
```

---

## 4. LIMIT and OFFSET

### 4.1 LIMIT

Restricts the number of rows returned:

```sql
-- Top 5 most expensive products
SELECT name, price FROM products
ORDER BY price DESC
LIMIT 5;
```

### 4.2 OFFSET

Skips a number of rows before starting to return results:

```sql
-- Products 6-10 (skip first 5)
SELECT name, price FROM products
ORDER BY price DESC
LIMIT 5 OFFSET 5;
```

### 4.3 Pagination Pattern

For page-based results (e.g., 10 items per page):

```sql
-- Page 1
SELECT * FROM products ORDER BY product_id LIMIT 10 OFFSET 0;
-- Page 2
SELECT * FROM products ORDER BY product_id LIMIT 10 OFFSET 10;
-- Page 3
SELECT * FROM products ORDER BY product_id LIMIT 10 OFFSET 20;
-- General formula: OFFSET = (page_number - 1) * page_size
```

### 4.4 FETCH FIRST (SQL Standard)

The SQL standard equivalent of LIMIT:

```sql
SELECT name, price FROM products
ORDER BY price DESC
FETCH FIRST 5 ROWS ONLY;
```

Both LIMIT and FETCH FIRST work in PostgreSQL; LIMIT is more common in PostgreSQL-specific code.

> **PostgreSQL docs:** https://www.postgresql.org/docs/current/queries-limit.html

---

## 5. Aggregate Functions

Aggregate functions compute a single value from a set of rows.

> **Reference:** Watt & Eng, Chapter 16, covers aggregate functions: "Aggregate functions allow us to perform calculations on groups of rows."

### 5.1 COUNT

```sql
-- Total number of products
SELECT COUNT(*) FROM products;

-- Number of products with a description (ignores NULLs)
SELECT COUNT(description) FROM products;

-- Number of distinct categories that have products
SELECT COUNT(DISTINCT category_id) FROM products;
```

**Key distinctions:**
- `COUNT(*)` — counts all rows, including those with NULLs
- `COUNT(column)` — counts rows where that column is NOT NULL
- `COUNT(DISTINCT column)` — counts unique non-NULL values

### 5.2 SUM

```sql
-- Total value of all inventory
SELECT SUM(price * stock) AS total_inventory_value FROM products;

-- Total revenue from order items
SELECT SUM(quantity * unit_price) AS total_revenue FROM order_items;
```

SUM ignores NULL values. If all values are NULL, SUM returns NULL (not 0).

### 5.3 AVG

```sql
-- Average product price
SELECT AVG(price) AS average_price FROM products;

-- Average with rounding
SELECT ROUND(AVG(price), 2) AS average_price FROM products;
```

AVG also ignores NULLs. This means AVG of (10, NULL, 20) = 15, not 10.

### 5.4 MIN and MAX

```sql
SELECT MIN(price) AS cheapest,
       MAX(price) AS most_expensive
FROM products;

SELECT MIN(order_date) AS first_order,
       MAX(order_date) AS latest_order
FROM orders;
```

### 5.5 NULL Handling in Aggregates

All aggregate functions (except COUNT(*)) skip NULL values:

| Data | COUNT(*) | COUNT(col) | SUM | AVG |
|------|----------|------------|-----|-----|
| 10, 20, NULL, 30 | 4 | 3 | 60 | 20 |

If you want NULLs treated as 0, use COALESCE:
```sql
SELECT AVG(COALESCE(discount_percent, 0)) FROM products;
```

### 5.6 Combining Aggregates

You can use multiple aggregates in one SELECT:

```sql
SELECT COUNT(*) AS total_products,
       ROUND(AVG(price), 2) AS avg_price,
       MIN(price) AS min_price,
       MAX(price) AS max_price,
       SUM(stock) AS total_stock
FROM products;
```

---

## 6. GROUP BY

GROUP BY divides rows into groups based on column values, then applies aggregate functions to each group.

### 6.1 Basic GROUP BY

```sql
-- Count products per category
SELECT category_id, COUNT(*) AS product_count
FROM products
GROUP BY category_id;
```

### 6.2 The Golden Rule

**Every column in SELECT must either be in an aggregate function OR in the GROUP BY clause.**

```sql
-- VALID: category_id is in GROUP BY, COUNT is an aggregate
SELECT category_id, COUNT(*) FROM products GROUP BY category_id;

-- INVALID: name is not in GROUP BY and not aggregated
SELECT category_id, name, COUNT(*) FROM products GROUP BY category_id;
-- ERROR: column "products.name" must appear in GROUP BY clause
```

Why? If you're grouping by category, there are multiple product names per category — SQL doesn't know which one to show.

### 6.3 Grouping by Multiple Columns

```sql
-- Average price per category per price range
SELECT category_id,
       CASE WHEN price < 100 THEN 'budget' ELSE 'premium' END AS tier,
       ROUND(AVG(price), 2) AS avg_price,
       COUNT(*) AS count
FROM products
GROUP BY category_id, 
         CASE WHEN price < 100 THEN 'budget' ELSE 'premium' END;
```

### 6.4 GROUP BY with ORDER BY

```sql
-- Categories ordered by product count (most first)
SELECT category_id, COUNT(*) AS product_count
FROM products
GROUP BY category_id
ORDER BY product_count DESC;
```

---

## 7. HAVING

HAVING filters *groups* (after GROUP BY), just as WHERE filters *rows* (before GROUP BY).

### 7.1 Basic HAVING

```sql
-- Categories with more than 2 products
SELECT category_id, COUNT(*) AS product_count
FROM products
GROUP BY category_id
HAVING COUNT(*) > 2;
```

### 7.2 WHERE vs HAVING

| | WHERE | HAVING |
|--|-------|--------|
| Filters | Individual rows | Groups |
| Position | Before GROUP BY | After GROUP BY |
| Can use aggregates? | No | Yes |
| Applied when? | Before grouping | After grouping |

```sql
-- WHERE filters rows BEFORE grouping,
-- HAVING filters groups AFTER grouping
SELECT category_id, AVG(price) AS avg_price
FROM products
WHERE stock > 0            -- only consider in-stock products
GROUP BY category_id
HAVING AVG(price) > 100;   -- only show categories where avg > 100
```

Think of it this way:
1. **WHERE** removes rows you don't want to consider at all
2. **GROUP BY** groups the remaining rows
3. **HAVING** removes groups that don't meet your criteria

### 7.3 Complex HAVING

```sql
-- Categories where total inventory value exceeds €1000
SELECT category_id,
       SUM(price * stock) AS inventory_value
FROM products
GROUP BY category_id
HAVING SUM(price * stock) > 1000;
```

---

## 8. JOINs — Combining Data from Multiple Tables

Joins are arguably the most powerful feature of SQL. They let you combine data from two or more tables based on related columns.

> **Reference:** Watt & Eng, Chapter 16, section "Joining Tables": "A join combines rows from two or more tables based on a related column between them."

### 8.1 Why JOINs Exist

In a properly normalized database, information is spread across multiple tables to avoid redundancy. The `products` table has a `category_id` but not the category name. The `orders` table has a `customer_id` but not the customer's name. To get a complete picture, you need to *join* tables together.

### 8.2 INNER JOIN

Returns only rows that have matching values in both tables.

**Syntax:**
```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;
```

**Example — Products with their category names:**
```sql
SELECT p.name AS product_name,
       p.price,
       c.name AS category_name
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id;
```

What happens: For each product row, PostgreSQL finds the matching category row (where category_ids match) and combines them into one result row. Products with no matching category (impossible here due to NOT NULL + FK) would be excluded.

### 8.3 LEFT (OUTER) JOIN

Returns all rows from the left table, plus matching rows from the right table. If no match exists, NULL values fill in for the right table's columns.

```sql
-- All customers, with their orders (if any)
SELECT c.first_name, c.last_name, o.order_id, o.status
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

Customers who have never ordered will appear with NULL for order_id and status.

**Finding unmatched rows (the "anti-join" pattern):**
```sql
-- Customers who have NEVER placed an order
SELECT c.first_name, c.last_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

This is one of the most useful patterns in SQL — finding rows in one table that have no related rows in another.

### 8.4 RIGHT (OUTER) JOIN

Returns all rows from the right table, plus matching rows from the left table. It's the mirror image of LEFT JOIN.

```sql
SELECT p.name, c.name AS category_name
FROM products p
RIGHT JOIN categories c ON p.category_id = c.category_id;
```

In practice, RIGHT JOIN is rarely used because you can always rewrite it as a LEFT JOIN by swapping the table order. Most developers prefer LEFT JOIN for consistency.

### 8.5 FULL OUTER JOIN

Returns all rows from both tables. Where no match exists, NULLs fill in.

```sql
SELECT c.name AS category_name, p.name AS product_name
FROM categories c
FULL OUTER JOIN products p ON c.category_id = p.category_id;
```

Use case: reconciliation — finding items that exist in one table but not the other (from either direction).

### 8.6 CROSS JOIN

Returns the Cartesian product — every row from the first table combined with every row from the second table. If table A has 5 rows and table B has 10 rows, the result has 50 rows.

```sql
SELECT c.name AS category, s.name AS size
FROM categories c
CROSS JOIN (VALUES ('S'), ('M'), ('L'), ('XL')) AS s(name);
```

CROSS JOIN is rarely needed, but useful for generating combinations (e.g., all possible category-size pairs).

### 8.7 NATURAL JOIN (Avoid)

NATURAL JOIN automatically joins on columns with the same name in both tables:

```sql
-- Joins on category_id because both tables have it
SELECT * FROM products NATURAL JOIN categories;
```

**Why to avoid it:** It's implicit and fragile. If you add a column with the same name to both tables later (e.g., `name` or `created_at`), the join condition silently changes. Always use explicit JOIN ON conditions.

### 8.8 Joining on Multiple Conditions

Sometimes you need more than one condition in the ON clause:

```sql
SELECT *
FROM order_items oi
INNER JOIN products p 
    ON oi.product_id = p.product_id
    AND oi.unit_price = p.price;
```

This only matches order items where the recorded unit_price equals the current product price (i.e., price hasn't changed since the order).

### 8.9 Table Aliases

When joining tables, aliases make queries much more readable:

```sql
-- Without aliases (verbose and hard to read)
SELECT products.name, categories.name
FROM products
INNER JOIN categories ON products.category_id = categories.category_id;

-- With aliases (clean and readable)
SELECT p.name, c.name AS category
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id;
```

Aliases are *required* when you join a table to itself (self-join) or use the same table multiple times.

---

## 9. Joining 3+ Tables

Real-world queries often need to traverse multiple relationships. You chain JOINs one after another:

### 9.1 Three-Table Join

**"Show each order item with the product name and customer name":**

```sql
SELECT c.first_name || ' ' || c.last_name AS customer,
       p.name AS product,
       oi.quantity,
       oi.unit_price
FROM order_items oi
INNER JOIN orders o ON oi.order_id = o.order_id
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN products p ON oi.product_id = p.product_id;
```

### 9.2 Reading Complex JOINs

When you encounter a multi-join query, read it like building blocks:
1. Start with the FROM table (the "base")
2. Each JOIN adds one more table to the mix
3. The ON clause tells you *how* they connect
4. Follow the foreign key relationships

For the query above:
- Start with `order_items` (the detail rows)
- Join `orders` to find out which order each item belongs to
- Join `customers` to get the customer's name via the order
- Join `products` to get the product's name

### 9.3 Four-Table Join with Category

```sql
SELECT c.first_name || ' ' || c.last_name AS customer,
       cat.name AS category,
       p.name AS product,
       oi.quantity,
       oi.unit_price,
       oi.quantity * oi.unit_price AS line_total
FROM order_items oi
INNER JOIN orders o ON oi.order_id = o.order_id
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN products p ON oi.product_id = p.product_id
INNER JOIN categories cat ON p.category_id = cat.category_id
ORDER BY customer, category;
```

---

## 10. Query Execution Order

SQL has a logical order of execution that differs from the order you *write* the clauses. Understanding this order explains many common errors.

### 10.1 The Logical Order

```
1. FROM / JOIN     — Determine the data source(s)
2. WHERE           — Filter individual rows
3. GROUP BY        — Group remaining rows
4. HAVING          — Filter groups
5. SELECT          — Choose columns / compute expressions
6. DISTINCT        — Remove duplicates
7. ORDER BY        — Sort the result
8. LIMIT / OFFSET  — Restrict the number of rows returned
```

### 10.2 Why This Matters

**You can't use a column alias in WHERE:**
```sql
-- WRONG: price_with_vat doesn't exist yet at WHERE stage
SELECT name, price * 1.24 AS price_with_vat
FROM products
WHERE price_with_vat > 100;

-- CORRECT: use the original expression
SELECT name, price * 1.24 AS price_with_vat
FROM products
WHERE price * 1.24 > 100;
```

**You can use a column alias in ORDER BY:**
```sql
-- VALID: ORDER BY runs after SELECT
SELECT name, price * 1.24 AS price_with_vat
FROM products
ORDER BY price_with_vat DESC;
```

**You can't use aggregates in WHERE:**
```sql
-- WRONG: WHERE runs before GROUP BY, so aggregates don't exist yet
SELECT category_id, AVG(price) FROM products
WHERE AVG(price) > 100
GROUP BY category_id;

-- CORRECT: use HAVING for aggregate conditions
SELECT category_id, AVG(price) FROM products
GROUP BY category_id
HAVING AVG(price) > 100;
```

---

## 11. Progressive TrailShop Queries

Let's apply everything with increasingly complex queries against the TrailShop database. For each query, I'll explain the thought process.

### Query 1: All products, sorted by price

**Question:** "Show me all products with their prices, cheapest first."

**Thought process:** Simple SELECT with ORDER BY. No filtering needed.

```sql
SELECT name, price
FROM products
ORDER BY price ASC;
```

### Query 2: Products over €100

**Question:** "Which products cost more than €100?"

**Thought process:** Add a WHERE clause to filter on price.

```sql
SELECT name, price
FROM products
WHERE price > 100
ORDER BY price DESC;
```

### Query 3: Products by keyword

**Question:** "Find all products with 'Pro' in the name."

**Thought process:** Use ILIKE for case-insensitive pattern matching.

```sql
SELECT name, price, stock
FROM products
WHERE name ILIKE '%pro%';
```

### Query 4: Product count per category

**Question:** "How many products are in each category?"

**Thought process:** I need to count rows grouped by category. I also want the category name (not just the ID), so I need a JOIN.

```sql
SELECT c.name AS category, COUNT(*) AS product_count
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
GROUP BY c.name
ORDER BY product_count DESC;
```

### Query 5: Average price per category

**Question:** "What's the average product price in each category?"

**Thought process:** Similar to above but with AVG instead of COUNT.

```sql
SELECT c.name AS category,
       ROUND(AVG(p.price), 2) AS avg_price
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
GROUP BY c.name
ORDER BY avg_price DESC;
```

### Query 6: Total inventory value

**Question:** "What's the total value of all products in stock?"

**Thought process:** Multiply price × stock for each product, then SUM. Single aggregate, no GROUP BY needed.

```sql
SELECT SUM(price * stock) AS total_inventory_value
FROM products;
```

### Query 7: Customer order history

**Question:** "Show all orders with customer names."

**Thought process:** Need to JOIN orders with customers to get names.

```sql
SELECT c.first_name || ' ' || c.last_name AS customer,
       o.order_id,
       o.order_date,
       o.status
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
ORDER BY o.order_date DESC;
```

### Query 8: Order details with product names

**Question:** "Show the items in order #1 with product names and line totals."

**Thought process:** Join order_items with products, filter by order_id.

```sql
SELECT p.name AS product,
       oi.quantity,
       oi.unit_price,
       oi.quantity * oi.unit_price AS line_total
FROM order_items oi
INNER JOIN products p ON oi.product_id = p.product_id
WHERE oi.order_id = 1;
```

### Query 9: Revenue per customer

**Question:** "How much has each customer spent in total?"

**Thought process:** Join customers → orders → order_items, then SUM the line totals grouped by customer.

```sql
SELECT c.first_name || ' ' || c.last_name AS customer,
       SUM(oi.quantity * oi.unit_price) AS total_spent
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC;
```

### Query 10: Customers who never ordered

**Question:** "Which customers have never placed an order?"

**Thought process:** LEFT JOIN customers to orders, then find where order_id IS NULL (the anti-join pattern).

```sql
SELECT c.first_name, c.last_name, c.email
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

### Query 11: Best-selling products

**Question:** "Which products have been ordered more than once (total quantity > 1)?"

**Thought process:** Aggregate order_items by product, filter with HAVING, join products for names.

```sql
SELECT p.name,
       SUM(oi.quantity) AS total_sold
FROM order_items oi
INNER JOIN products p ON oi.product_id = p.product_id
GROUP BY p.product_id, p.name
HAVING SUM(oi.quantity) > 1
ORDER BY total_sold DESC;
```

### Query 12: Complete order summary

**Question:** "Show a summary of each order: customer name, number of items, and total cost."

**Thought process:** Multi-table join with aggregation. Group by order and customer.

```sql
SELECT o.order_id,
       c.first_name || ' ' || c.last_name AS customer,
       o.status,
       SUM(oi.quantity) AS total_items,
       SUM(oi.quantity * oi.unit_price) AS order_total
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id, c.first_name, c.last_name, o.status
ORDER BY o.order_id;
```

---

## 12. Key Terms

| Term | Definition |
|------|-----------|
| SELECT | SQL statement for retrieving data from tables |
| WHERE | Clause that filters rows based on conditions |
| ORDER BY | Clause that sorts the result set |
| LIMIT / OFFSET | Clauses that restrict and paginate results |
| Aggregate function | A function that computes one value from many rows (COUNT, SUM, AVG, MIN, MAX) |
| GROUP BY | Clause that groups rows for aggregate calculations |
| HAVING | Clause that filters groups (post-aggregation) |
| JOIN | Combines rows from two or more tables based on related columns |
| INNER JOIN | Returns only matching rows from both tables |
| LEFT JOIN | Returns all rows from the left table, with NULLs for non-matching right rows |
| FULL OUTER JOIN | Returns all rows from both tables, with NULLs where no match |
| CROSS JOIN | Produces the Cartesian product of two tables |
| Alias | A temporary name for a column or table within a query |
| DISTINCT | Eliminates duplicate rows from the result |
| Anti-join | Pattern using LEFT JOIN + IS NULL to find unmatched rows |

---

## 13. Reading

### Required Reading

- Watt & Eng, *Database Design — 2nd Edition*, **Chapter 16** — all sections on SELECT, WHERE, aggregate functions, and joining tables

### Further Reading

- PostgreSQL Documentation — SELECT: https://www.postgresql.org/docs/current/sql-select.html
- PostgreSQL Documentation — Aggregate Functions: https://www.postgresql.org/docs/current/functions-aggregate.html
- PostgreSQL Documentation — Joins: https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-JOIN
- PostgreSQL Tutorial — Joins with Venn Diagrams: https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-joins/

---

## 14. Summary

This week you learned the most important SQL skill: extracting meaningful information from your database. You started with basic SELECT and built up to complex multi-table queries with aggregation:

- **SELECT** retrieves specific columns; use aliases and expressions for clarity
- **WHERE** filters individual rows using comparison operators, BETWEEN, IN, LIKE, and NULL checks
- **ORDER BY** sorts results; LIMIT/OFFSET provide pagination
- **Aggregate functions** (COUNT, SUM, AVG, MIN, MAX) summarize groups of rows
- **GROUP BY** creates groups; HAVING filters those groups
- **JOINs** combine related tables — INNER for matches only, LEFT for everything-plus-matches, and more

The TrailShop founders can now get instant answers to their business questions. But as the database grows, they'll need more sophisticated techniques — subqueries, views, and performance optimization. Next week is a review and exam week — a chance to consolidate everything from Weeks 36-41 before moving forward.
