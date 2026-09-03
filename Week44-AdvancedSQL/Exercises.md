# Week 44 — Advanced SQL: Exercises

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

---

## 1. TrailShop Project Task

Continue building the TrailShop database. This week, use advanced query techniques to extract meaningful insights.

### Task 1.1: Subqueries

Write the following queries using subqueries:

1. Find all products priced above the average price (use a scalar subquery).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
2. Find all customers who have placed more than 2 orders (use a subquery in WHERE with IN).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
3. Show each product's name, price, and how much it differs from its category's average price (use a correlated subquery or derived table).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### Task 1.2: EXISTS

1. Use `EXISTS` to find all customers who have ordered at least one product from the "Footwear" category.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
2. Use `NOT EXISTS` to find all products that have never been ordered.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### Task 1.3: CASE Expressions

1. Add a `price_tier` column to a product query that labels products as 'Budget' (< €25), 'Standard' (€25–€74.99), or 'Premium' (≥ €75).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
2. Write a query that counts orders by status using CASE inside COUNT.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### Task 1.4: Create Three Views

Create the following views in your TrailShop database:

1. `v_product_catalog` — Shows product name, category name, price, and stock status ('In Stock', 'Low Stock' for < 5 units, 'Out of Stock' for 0).
2. `v_customer_summary` — Shows each customer's full name, email, number of orders, and total spent.
3. `v_monthly_revenue` — Shows revenue grouped by month.

Verify each view works with a simple SELECT.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your CREATE VIEW statements here
>
>
> ```

### Task 1.5: UNION

1. Write a UNION query that combines product names from two different categories into one list (without duplicates).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
2. Write a query using EXCEPT to find categories that have products but no orders.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### Task 1.6: CTE

1. Write a CTE that calculates total revenue per category, then use it to find categories with above-average revenue.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
2. Write a query with two CTEs: one for customer order counts, another for average order count, then find "top customers" (above average).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

---

## 2. Theory Review Questions

Answer these in your own words (2–4 sentences each):

1. What is the difference between an INNER JOIN and a LEFT JOIN? When would you choose LEFT JOIN?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>


2. Explain what a correlated subquery is. Why can it be slower than an uncorrelated subquery?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>


3. What is the difference between `IN` and `EXISTS`? When might you prefer EXISTS?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>


4. What rules must two SELECT statements follow to be combined with UNION?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>


5. Explain the difference between UNION and UNION ALL. When would you use each?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>


6. What is a view? Name two advantages of using views.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
7. What does `WITH CHECK OPTION` do on an updatable view?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>


8. How does a CTE differ from a derived table (subquery in FROM)? What's the main advantage of CTEs?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

## 3. Query Writing Exercises

Write SQL queries for the following (use the TrailShop schema):

1. Find the most expensive product in each category using a correlated subquery.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
2. List all customers along with their most recent order date. Include customers who have never ordered (show NULL).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
3. Find categories where the total revenue exceeds €500 (use a subquery or CTE).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
4. Write a query that shows each order's total and whether it's 'Small' (< €50), 'Medium' (€50–€149.99), or 'Large' (≥ €150).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
5. Create a view called `v_never_ordered` that shows products that have never appeared in an order_item.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
6. Use EXCEPT to find products that exist in the products table but have never been ordered.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
7. Write a CTE that ranks categories by number of products, then selects only the top 3.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
8. Use `NOT EXISTS` to find categories that have no products.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
9. Write a query using CASE that creates a "customer loyalty" label: 'New' (1 order), 'Regular' (2–5 orders), 'VIP' (6+ orders).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
10. Use GREATEST to find the higher value between each product's current price and the average price of all products in its category.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
11. Write a query using COALESCE that shows order shipping dates, substituting 'Not yet shipped' for NULL values.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
12. Use a scalar subquery in the SELECT clause to show each product alongside the total number of times it has been ordered.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

---

## 4. Query Comparison Exercise

For each pair of queries below, both produce the same result. Explain which approach you'd prefer and why (consider readability, performance, and maintainability).
> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### Pair A: Finding customers with orders

**Version 1 — Subquery with IN:**
```sql
SELECT first_name, last_name
FROM customers
WHERE customer_id IN (SELECT customer_id FROM orders);
```

**Version 2 — JOIN:**
```sql
SELECT DISTINCT c.first_name, c.last_name
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id;
```

### Pair B: Finding products never ordered

**Version 1 — LEFT JOIN + IS NULL:**
```sql
SELECT p.product_name
FROM products p
LEFT JOIN order_items oi ON oi.product_id = p.product_id
WHERE oi.order_item_id IS NULL;
```

**Version 2 — NOT EXISTS:**
```sql
SELECT p.product_name
FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items oi WHERE oi.product_id = p.product_id
);
```

### Pair C: Customer total spending

**Version 1 — Derived table (subquery in FROM):**
```sql
SELECT sub.full_name, sub.total_spent
FROM (
    SELECT c.first_name || ' ' || c.last_name AS full_name,
           SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM customers c
    JOIN orders o ON o.customer_id = c.customer_id
    JOIN order_items oi ON oi.order_id = o.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
) sub
WHERE sub.total_spent > 200;
```

**Version 2 — CTE:**
```sql
WITH customer_totals AS (
    SELECT c.first_name || ' ' || c.last_name AS full_name,
           SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM customers c
    JOIN orders o ON o.customer_id = c.customer_id
    JOIN order_items oi ON oi.order_id = o.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
)
SELECT full_name, total_spent
FROM customer_totals
WHERE total_spent > 200;
```

For each pair, discuss: readability, performance (in PostgreSQL), and when you'd choose one over the other.
> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
