# Week 41 — Exercises: Querying Data

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

## Exercise 1: TrailShop Project Task — Business Questions

Use the TrailShop database you created in Week 40. Write SQL queries to answer each business question below. Run each query and verify the results make sense.

### Basic Queries (SELECT + WHERE)

1. List all products in the 'Footwear' category (show name, price, stock).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

2. Find all products priced between €50 and €150, sorted by price ascending.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

3. Show all customers whose last name starts with the letter 'M' or 'K'.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

4. List all orders with status 'pending' or 'shipped', sorted by order date (most recent first).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

5. Find all products that have the word 'Pro' or 'pro' somewhere in their name.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### Aggregate Queries

6. What is the total number of products in the database?

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

7. What is the average price of all products? Round to 2 decimal places.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

8. Which is the most expensive product and which is the cheapest? Show both in one query.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

9. How many orders does each customer have? Show customer name and order count, sorted by count descending.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

10. What is the total revenue (sum of quantity × unit_price from order_items) for each order status?

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### JOIN Queries

11. List all products with their category names (not just category_id). Sort by category name, then product name.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

12. Show each order with the customer's full name, order date, and status.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

13. Show a detailed breakdown of order #1: product name, quantity, unit price, and line total.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

14. Find all customers who have NOT placed any orders. (Hint: use LEFT JOIN + IS NULL pattern.)
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

15. For each category, show the category name, number of products, average price, and total inventory value (price × stock summed). Only include categories with total inventory value greater than €500.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

---

## Exercise 2: Theory Review Questions

Answer in your own words:

1. What is the logical execution order of a SQL query? Why does it matter?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>



2. What is the difference between WHERE and HAVING? Give an example of when you would use each.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

3. Explain the difference between COUNT(*), COUNT(column), and COUNT(DISTINCT column).

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

4. What is the difference between INNER JOIN and LEFT JOIN? When would you choose one over the other?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>



5. Why should you avoid SELECT * in production code?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>



6. What does DISTINCT do? On what level does it operate (columns or entire rows)?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>



7. Can you use a column alias in a WHERE clause? Why or why not?



> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
8. Explain what happens when you GROUP BY a column and there's a column in SELECT that isn't aggregated and isn't in GROUP BY.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

## Exercise 3: Query Writing Exercises

Write the SQL for each task. Use the TrailShop schema (categories, customers, products, orders, order_items).

### Simple SELECT + WHERE

**3.1** — Select product names and prices for all products with stock less than 15.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


**3.2** — Find all customers who registered (created_at) in 2026. Show first name, last name, and registration date.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


**3.3** — Show all products that do NOT have a description (description IS NULL).

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


### Multi-condition Filtering

**3.4** — Find products in category 1 OR category 2, priced above €100, with stock greater than 0. Sort by price descending.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


**3.5** — Find orders that are either 'delivered' or placed by customer_id 1. Show order_id, customer_id, status.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


### Aggregation

**3.6** — For each category_id, show the minimum, maximum, and average price. Round averages to 2 decimal places.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


**3.7** — Count how many distinct customers have placed at least one order.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


**3.8** — Find the total quantity of items sold across all orders.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


### JOIN Queries

**3.9** — Show each product name alongside its category name. Include all products (even if somehow a category was deleted — use LEFT JOIN).

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


**3.10** — List all orders showing: order_id, customer full name, order date, number of items in the order, and order total cost.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


**3.11** — Show all products that have NEVER been ordered. (Hint: LEFT JOIN order_items, then IS NULL on order_item_id.)

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


### GROUP BY + HAVING

**3.12** — Show categories where the average product price exceeds €100. Display category name and average price.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


**3.13** — Find customers who have placed more than 1 order. Show their name and order count.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


### Pagination

**3.14** — Write a paginated query that returns products 4 through 6 (page 2, page size 3), ordered by product_id.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


### Complex

**3.15** — Write a "sales report" query that shows: category name, total units sold (from order_items), total revenue, and number of distinct products sold — for each category. Sort by revenue descending.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```


---

## Exercise 4: Query Reading Exercise

For each query below, explain in **plain English** what it does and what the result would look like.

### 4.1

```sql
SELECT c.name, COUNT(p.product_id) AS num_products
FROM categories c
LEFT JOIN products p ON c.category_id = p.category_id
GROUP BY c.name
ORDER BY num_products DESC;
```

> [!NOTE]
> ***Error(s) Identified***
>
> *(Describe what is wrong.)*
>
>
>
>
> [!NOTE]
> ***Corrected SQL***
>
> ```sql
> -- Write the corrected statement here
>
>
> ```


### 4.2

```sql
SELECT first_name, last_name
FROM customers
WHERE customer_id NOT IN (
    SELECT DISTINCT customer_id FROM orders
);
```

> [!NOTE]
> ***Error(s) Identified***
>
> *(Describe what is wrong.)*
>
>
>
>
> [!NOTE]
> ***Corrected SQL***
>
> ```sql
> -- Write the corrected statement here
>
>
> ```


### 4.3

```sql
SELECT p.name, p.price, p.stock,
       p.price * p.stock AS inventory_value
FROM products p
WHERE p.stock > 0
ORDER BY inventory_value DESC
LIMIT 3;
```

> [!NOTE]
> ***Error(s) Identified***
>
> *(Describe what is wrong.)*
>
>
>
>
> [!NOTE]
> ***Corrected SQL***
>
> ```sql
> -- Write the corrected statement here
>
>
> ```


### 4.4

```sql
SELECT o.order_id,
       SUM(oi.quantity * oi.unit_price) AS order_total
FROM orders o
INNER JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.status <> 'cancelled'
GROUP BY o.order_id
HAVING SUM(oi.quantity * oi.unit_price) > 200
ORDER BY order_total DESC;
```

> [!NOTE]
> ***Error(s) Identified***
>
> *(Describe what is wrong.)*
>
>
>
>
> [!NOTE]
> ***Corrected SQL***
>
> ```sql
> -- Write the corrected statement here
>
>
> ```


### 4.5

```sql
SELECT c.first_name || ' ' || c.last_name AS customer,
       COUNT(DISTINCT o.order_id) AS num_orders,
       SUM(oi.quantity) AS total_items,
       ROUND(SUM(oi.quantity * oi.unit_price), 2) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC NULLS LAST;
```

> [!NOTE]
> ***Error(s) Identified***
>
> *(Describe what is wrong.)*
>
>
>
>
> [!NOTE]
> ***Corrected SQL***
>
> ```sql
> -- Write the corrected statement here
>
>
> ```


---

## Submission Checklist

- [ ] All 15 business questions answered with working SQL
- [ ] Theory review questions answered in your own words
- [ ] All 15 query writing exercises completed
- [ ] All 5 query reading explanations written
- [ ] Results verified by running queries against your TrailShop database
