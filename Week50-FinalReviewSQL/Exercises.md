# Week 50 — Final Review: SQL — Exercises (Optional)

> [!IMPORTANT] Optional Exam Preparation
> **No submission is required this week.** These exercises are self-paced practice for the final exam. Work through as many as you find helpful.

## 15 Progressive Practice Queries

These queries increase in complexity. Each builds on concepts from the theory. You can run them against your completed TrailShop database (read-only practice) or any sample schema with similar tables. Try to solve them without looking at the solutions first.

### Query 1: Basic SELECT with filter

**Task:** List all products priced between €50 and €150, showing name and price, sorted by price.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT product_name, price
FROM products
WHERE price BETWEEN 50 AND 150
ORDER BY price;
```
</details>

### Query 2: COUNT with GROUP BY

**Task:** Count the number of products in each category.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT c.category_name, COUNT(*) AS product_count
FROM products p
JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_name
ORDER BY product_count DESC;
```
</details>

### Query 3: LEFT JOIN

**Task:** Show all categories and their products, including categories with no products.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT c.category_name, p.product_name, p.price
FROM categories c
LEFT JOIN products p ON c.category_id = p.category_id
ORDER BY c.category_name, p.product_name;
```
</details>

### Query 4: Aggregate with HAVING

**Task:** Find categories where the average product price exceeds €100.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT c.category_name, AVG(p.price) AS avg_price
FROM categories c
JOIN products p ON c.category_id = p.category_id
GROUP BY c.category_name
HAVING AVG(p.price) > 100
ORDER BY avg_price DESC;
```
</details>

### Query 5: Subquery in WHERE

**Task:** Find products priced above the overall average price.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT product_name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products)
ORDER BY price DESC;
```
</details>

### Query 6: Multi-table JOIN

**Task:** List all orders with customer name, order date, and total order value.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT o.order_id, c.first_name || ' ' || c.last_name AS customer,
       o.order_date,
       SUM(oi.quantity * oi.unit_price) AS order_total
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id, c.first_name, c.last_name, o.order_date
ORDER BY o.order_date DESC;
```
</details>

### Query 7: EXISTS

**Task:** Find customers who have ordered at least one product from the 'Climbing' category.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT DISTINCT c.first_name, c.last_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    JOIN categories cat ON p.category_id = cat.category_id
    WHERE o.customer_id = c.customer_id
      AND cat.category_name = 'Climbing'
);
```
</details>

### Query 8: CASE expression

**Task:** Classify products into price tiers: 'Budget' (<50), 'Standard' (50–149.99), 'Premium' (150+).

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT product_name, price,
    CASE
        WHEN price < 50 THEN 'Budget'
        WHEN price < 150 THEN 'Standard'
        ELSE 'Premium'
    END AS price_tier
FROM products
ORDER BY price;
```
</details>

### Query 9: CTE

**Task:** Use a CTE to find each customer's total spending, then show only the top 5.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
WITH customer_spending AS (
    SELECT c.customer_id, c.first_name, c.last_name,
           SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
)
SELECT first_name, last_name, total_spent
FROM customer_spending
ORDER BY total_spent DESC
LIMIT 5;
```
</details>

### Query 10: Set operation

**Task:** Find products that have been ordered but never reviewed (assuming a `product_reviews` table exists).

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT DISTINCT p.product_name
FROM products p
JOIN order_items oi ON p.product_id = oi.product_id
EXCEPT
SELECT DISTINCT p.product_name
FROM products p
JOIN product_reviews pr ON p.product_id = pr.product_id;
```
</details>

### Query 11: Correlated subquery

**Task:** For each product, show its price and the maximum price in its category.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT product_name, price,
       (SELECT MAX(p2.price)
        FROM products p2
        WHERE p2.category_id = p.category_id) AS max_in_category
FROM products p
ORDER BY category_id, price DESC;
```
</details>

### Query 12: COALESCE and LEFT JOIN

**Task:** Show all products with their review count (0 if never reviewed).

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT p.product_name, COALESCE(COUNT(pr.review_id), 0) AS review_count
FROM products p
LEFT JOIN product_reviews pr ON p.product_id = pr.product_id
GROUP BY p.product_id, p.product_name
ORDER BY review_count DESC;
```
</details>

### Query 13: Multiple aggregates

**Task:** Show category statistics: product count, min/max/avg price, and total stock.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT c.category_name,
       COUNT(*) AS products,
       MIN(p.price) AS min_price,
       MAX(p.price) AS max_price,
       ROUND(AVG(p.price), 2) AS avg_price,
       SUM(p.stock_qty) AS total_stock
FROM categories c
JOIN products p ON c.category_id = p.category_id
GROUP BY c.category_name
ORDER BY c.category_name;
```
</details>

### Query 14: Subquery in FROM

**Task:** Find the category with the highest total revenue.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT category_name, total_revenue
FROM (
    SELECT c.category_name,
           SUM(oi.quantity * oi.unit_price) AS total_revenue
    FROM categories c
    JOIN products p ON c.category_id = p.category_id
    JOIN order_items oi ON p.product_id = oi.product_id
    GROUP BY c.category_name
) AS category_revenue
ORDER BY total_revenue DESC
LIMIT 1;
```
</details>

### Query 15: Complex CTE with multiple steps

**Task:** Find customers whose spending is above the average customer spending.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
WITH customer_totals AS (
    SELECT c.customer_id, c.first_name, c.last_name,
           SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
),
avg_spending AS (
    SELECT AVG(total_spent) AS avg_total FROM customer_totals
)
SELECT ct.first_name, ct.last_name, ct.total_spent,
       ROUND(a.avg_total, 2) AS average_spending
FROM customer_totals ct
CROSS JOIN avg_spending a
WHERE ct.total_spent > a.avg_total
ORDER BY ct.total_spent DESC;
```
</details>

---

## Additional Challenge Queries

These 5 queries introduce advanced concepts. They go slightly beyond exam expectations but will deepen your understanding.

### Challenge 1: Window Function — RANK and ROW_NUMBER

**Task:** Rank products by price within each category. Show the rank and row number.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

<details>
<summary>Solution</summary>

```sql
SELECT product_name, category_id, price,
       RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS price_rank,
       ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS row_num
FROM products
ORDER BY category_id, price_rank;
```

**Explanation:** `RANK()` assigns the same rank to tied values and skips the next number. `ROW_NUMBER()` always gives a unique sequential number. `PARTITION BY` restarts the numbering for each category.
</details>

### Challenge 2: Complex CTE with Multiple Steps

**Task:** Find the month with the highest revenue and show which products contributed most to that month's sales (top 3 products).

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

<details>
<summary>Solution</summary>

```sql
WITH monthly_revenue AS (
    SELECT DATE_TRUNC('month', o.order_date) AS month,
           SUM(oi.quantity * oi.unit_price) AS revenue
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    GROUP BY DATE_TRUNC('month', o.order_date)
),
best_month AS (
    SELECT month FROM monthly_revenue ORDER BY revenue DESC LIMIT 1
),
best_month_products AS (
    SELECT p.product_name,
           SUM(oi.quantity * oi.unit_price) AS product_revenue
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    WHERE DATE_TRUNC('month', o.order_date) = (SELECT month FROM best_month)
    GROUP BY p.product_name
)
SELECT product_name, product_revenue
FROM best_month_products
ORDER BY product_revenue DESC
LIMIT 3;
```
</details>

### Challenge 3: Self-Join

**Task:** Find pairs of products in the same category with a price difference of less than €10.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT a.product_name AS product_a,
       b.product_name AS product_b,
       a.price AS price_a,
       b.price AS price_b,
       ABS(a.price - b.price) AS price_difference
FROM products a
JOIN products b ON a.category_id = b.category_id
                AND a.product_id < b.product_id
WHERE ABS(a.price - b.price) < 10
ORDER BY price_difference;
```

**Explanation:** The condition `a.product_id < b.product_id` prevents duplicate pairs (A,B) and (B,A) and prevents pairing a product with itself.
</details>

### Challenge 4: FULL OUTER JOIN Reconciliation

**Task:** Find discrepancies between the `products` table and an `inventory_audit` table — products missing from audit, audit entries without a matching product, and quantity mismatches.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

<details>
<summary>Solution</summary>

```sql
SELECT COALESCE(p.product_name, 'UNKNOWN (audit only)') AS product,
       p.stock_qty AS system_qty,
       ia.counted_qty AS audit_qty,
       CASE
           WHEN p.product_id IS NULL THEN 'In audit but not in products'
           WHEN ia.product_id IS NULL THEN 'In products but not in audit'
           WHEN p.stock_qty != ia.counted_qty THEN 'Quantity mismatch'
           ELSE 'OK'
       END AS status
FROM products p
FULL OUTER JOIN inventory_audit ia ON p.product_id = ia.product_id
WHERE p.product_id IS NULL
   OR ia.product_id IS NULL
   OR p.stock_qty != ia.counted_qty
ORDER BY status, product;
```

**Explanation:** `FULL OUTER JOIN` keeps rows from both sides. The `CASE` classifies each discrepancy type. The `WHERE` filters out matching rows.
</details>

### Challenge 5: Nested Subquery with EXISTS and NOT EXISTS

**Task:** Find customers who have ordered from every category in the database.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Solution</summary>

```sql
SELECT c.first_name, c.last_name
FROM customers c
WHERE NOT EXISTS (
    SELECT cat.category_id
    FROM categories cat
    WHERE NOT EXISTS (
        SELECT 1
        FROM orders o
        JOIN order_items oi ON o.order_id = oi.order_id
        JOIN products p ON oi.product_id = p.product_id
        WHERE o.customer_id = c.customer_id
          AND p.category_id = cat.category_id
    )
);
```

**Explanation:** This is the *relational division* pattern. The outer `NOT EXISTS` says "there is no category..." and the inner `NOT EXISTS` says "...for which this customer has never ordered a product." Combined: find customers where every category has at least one order.
</details>

---

## SQL Error Diagnosis

Each of the following SQL statements contains an error. Identify the problem and write the corrected version.

### Error 1

```sql
SELECT product_name, price, COUNT(*)
FROM products
WHERE category_id = 1;
```

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Problem & Fix</summary>

**Problem:** `COUNT(*)` is an aggregate function but `product_name` and `price` are not aggregated or grouped.

**Fix:**
```sql
SELECT product_name, price, COUNT(*) OVER () AS total_in_category
FROM products
WHERE category_id = 1;
```
Or if you want a count per product (which is just 1), you likely need `GROUP BY`:
```sql
SELECT category_id, COUNT(*) AS product_count
FROM products
WHERE category_id = 1
GROUP BY category_id;
```
</details>

### Error 2

```sql
SELECT product_name, price AS cost
FROM products
WHERE cost > 100;
```

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Problem & Fix</summary>

**Problem:** Column aliases defined in `SELECT` cannot be used in `WHERE` (execution order: WHERE runs before SELECT).

**Fix:**
```sql
SELECT product_name, price AS cost
FROM products
WHERE price > 100;
```
</details>

### Error 3

```sql
SELECT category_id, AVG(price)
FROM products
GROUP BY category_id
WHERE AVG(price) > 50;
```

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Problem & Fix</summary>

**Problem:** `WHERE` cannot contain aggregate functions. Also, `WHERE` must come before `GROUP BY`.

**Fix:**
```sql
SELECT category_id, AVG(price)
FROM products
GROUP BY category_id
HAVING AVG(price) > 50;
```
</details>

### Error 4

```sql
INSERT INTO products (product_name, price, stock_qty)
VALUES ('Trail Pole', -15.00, 10);
```

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Problem & Fix</summary>

**Problem:** If the table has `CHECK (price > 0)`, this violates the constraint (negative price).

**Fix:**
```sql
INSERT INTO products (product_name, price, stock_qty)
VALUES ('Trail Pole', 15.00, 10);
```
</details>

### Error 5

```sql
SELECT p.product_name, c.category_name
FROM products p
JOIN categories c ON p.category_id = categories.category_id;
```

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Problem & Fix</summary>

**Problem:** Mixed alias and full table name. Once you alias `categories` as `c`, you must use `c`.

**Fix:**
```sql
SELECT p.product_name, c.category_name
FROM products p
JOIN categories c ON p.category_id = c.category_id;
```
</details>

### Error 6

```sql
DELETE products
WHERE stock_qty = 0;
```

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Problem & Fix</summary>

**Problem:** Missing `FROM` keyword.

**Fix:**
```sql
DELETE FROM products
WHERE stock_qty = 0;
```
</details>

### Error 7

```sql
SELECT product_name
FROM products
ORDER BY price
LIMIT 5
OFFSET -2;
```

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Problem & Fix</summary>

**Problem:** `OFFSET` cannot be negative.

**Fix:**
```sql
SELECT product_name
FROM products
ORDER BY price
LIMIT 5
OFFSET 0;
```
</details>

### Error 8

```sql
SELECT c.category_name, COUNT(*) AS cnt
FROM products p
LEFT JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_name
HAVING cnt > 5;
```

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Problem & Fix</summary>

**Problem:** In PostgreSQL, you cannot use a column alias in `HAVING`. You must repeat the aggregate expression.

**Fix:**
```sql
SELECT c.category_name, COUNT(*) AS cnt
FROM products p
LEFT JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_name
HAVING COUNT(*) > 5;
```
</details>

---
