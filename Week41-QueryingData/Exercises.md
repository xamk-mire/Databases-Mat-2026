# Week 41 — Exercises: Querying Data

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

## Exercise 1: TrailShop Project Task — Business Questions

Use the TrailShop database you created in Week 40. Write SQL queries to answer each business question below. Run each query and verify the results make sense.

> [!IMPORTANT]
> **_Tools to use_**
> You can use both the pgAdmin or psql (terminal) to test the queries

## Basic Queries (SELECT + WHERE)

### 1. List all products in the 'Footwear' category (show name, price, stock).

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT p.name, p.price, p.stock
> FROM products p
> JOIN product_categories pc ON p.product_id = pc.product_id
> JOIN categories c ON pc.category_id = c.category_id
> WHERE c.name = 'Footwear';
> ```

### 2. Find all products priced between €50 and €150, sorted by price ascending.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT name, price
> FROM products
> WHERE price BETWEEN 50 AND 150
> ORDER BY price ASC;
> ```

### 3. Find all customers whose last name starts with 'M' or 'K'.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT first_name, last_name
> FROM customers
> WHERE last_name LIKE 'M%'
>    OR last_name LIKE 'K%';
> ```

### 4. Find all orders with status 'pending' or 'shipped', sorted by most recent first.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT order_id, order_date, status
> FROM orders
> WHERE status IN ('pending', 'shipped')
> ORDER BY order_date DESC;
> ```

### 5. Find all products with 'Pro' or 'pro' in their name.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT name, price, stock
> FROM products
> WHERE name ILIKE '%pro%';
> ```

### 6. Count the total number of products.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT COUNT(*) AS total_products
> FROM products;
> ```

### 7. Calculate the average product price, rounded to two decimal places.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT ROUND(AVG(price), 2) AS average_price
> FROM products;
> ```

### 8. Find the most expensive and cheapest products using one query.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT name, price
> FROM products
> WHERE price = (SELECT MAX(price) FROM products)
>    OR price = (SELECT MIN(price) FROM products);
> ```

## Aggregate Queries (GROUP BY + HAVING)

### 9. Count the number of orders placed by each customer, sorted by order count descending.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT
>     c.first_name || ' ' || c.last_name AS customer_name,
>     COUNT(o.order_id) AS order_count
> FROM customers c
> LEFT JOIN orders o ON c.customer_id = o.customer_id
> GROUP BY c.customer_id, c.first_name, c.last_name
> ORDER BY order_count DESC;
> ```

### 10. Calculate total revenue for each order status.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT
>     o.status,
>     SUM(oi.quantity * oi.unit_price) AS total_revenue
> FROM orders o
> JOIN order_items oi ON o.order_id = oi.order_id
> GROUP BY o.status
> ORDER BY o.status;
> ```

## JOIN Queries

### 11. List all products with their category names. Products in two categories should appear twice. Sort by category, then product name.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT
>     p.name AS product_name,
>     c.name AS category_name
> FROM products p
> JOIN product_categories pc ON p.product_id = pc.product_id
> JOIN categories c ON pc.category_id = c.category_id
> ORDER BY category_name, product_name;
> ```

### 12. List each order with the customer's full name, order date, and status.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT
>     c.first_name || ' ' || c.last_name AS customer_name,
>     o.order_id,
>     o.order_date,
>     o.status
> FROM orders o
> JOIN customers c ON o.customer_id = c.customer_id
> ORDER BY o.order_date DESC;
> ```

### 13. Show a detailed breakdown of order #1, including product name, quantity, unit price, and line total.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT
>     p.name AS product_name,
>     oi.quantity,
>     oi.unit_price,
>     oi.quantity * oi.unit_price AS line_total
> FROM order_items oi
> JOIN products p ON oi.product_id = p.product_id
> WHERE oi.order_id = 1;
> ```

### 14. Find all customers who have never placed an order. Use LEFT JOIN and IS NULL.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT
>     c.first_name,
>     c.last_name,
>     c.email
> FROM customers c
> LEFT JOIN orders o ON c.customer_id = o.customer_id
> WHERE o.order_id IS NULL;
> ```

## Advanced Aggregate Query

### 15. For each category, show the category name, number of products, average price, and total inventory value (price × stock). Only include categories with a total inventory value greater than €500.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT
>     c.name AS category_name,
>     COUNT(p.product_id) AS product_count,
>     ROUND(AVG(p.price), 2) AS average_price,
>     SUM(p.price * p.stock) AS total_inventory_value
> FROM categories c
> JOIN product_categories pc ON c.category_id = pc.category_id
> JOIN products p ON pc.product_id = p.product_id
> GROUP BY c.category_id, c.name
> HAVING SUM(p.price * p.stock) > 500
> ORDER BY total_inventory_value DESC;
> ```

---

## Exercise 2: Theory Review Questions

Answer in your own words:

1. What is the logical execution order of a SQL query? Why does it matter?

> [!NOTE]
> **_Your Answer_**
>
>The order is FROM, WHERE, GROUP BY, HAVING, SELECT, DISTINCT, ORDER BY, and LIMIT. SQL follows this order to find the data, filter rows, group them, and show the final result. Understanding this order helps me write correct queries and understand how SQL works.

2. What is the difference between WHERE and HAVING? Give an example of when you would use each.

> [!NOTE]
> **_Your Answer_**
>
>WHERE filters individual rows before grouping, while HAVING filters groups after GROUP BY. For example, I use WHERE to find products that cost more than €50. I use HAVING to show only categories that have more than five products.

3. Explain the difference between COUNT(\*), COUNT(column), and COUNT(DISTINCT column).

> [!NOTE]
> **_Your Answer_**
>
>COUNT(*) counts all rows in the result. COUNT(column) counts only rows where the column is not NULL. COUNT(DISTINCT column) counts unique, non-NULL values. These functions help me count records and find how many different values exist in a table.

4. What is the difference between INNER JOIN and LEFT JOIN? When would you choose one over the other?

> [!NOTE]
> **_Your Answer_**
>
>INNER JOIN returns only rows that have matching values in both tables. LEFT JOIN returns all rows from the left table, even if there is no match in the right table. I use INNER JOIN when I need matching records and LEFT JOIN when I also want to find records without a match, such as customers who have not placed any orders.

5. Why should you avoid SELECT \* in production code?

> [!NOTE]
> **_Your Answer_**
>
> SELECT * returns all columns from a table, even if I do not need them. This can retrieve unnecessary data and make the query harder to understand. If someone changes the table later, the results may also change. It is better to select only the columns I need.

6. What does DISTINCT do? On what level does it operate (columns or entire rows)?

> [!NOTE]
> **_Your Answer_**
>
> DISTINCT removes duplicate rows from the query result. It works on the combination of all selected columns. For example, SELECT DISTINCT status FROM orders shows each order status only once, even if many orders have the same status.

7. Can you use a column alias in a WHERE clause? Why or why not?

> [!NOTE]
> **_Your Answer_**
>
> No, normally I cannot use a column alias directly in WHERE because WHERE is processed before SELECT, where the alias is created. For example, if I create an alias called total_price in SELECT, I cannot normally use it in WHERE. Instead, I can use the original expression in the WHERE condition.
>
 8. Explain what happens when you GROUP BY a column and there's a column in SELECT that isn't aggregated and isn't in GROUP BY.

> [!NOTE]
> **_Your Answer_**
>
> Every selected column must either be included in GROUP BY or used with an aggregate function such as COUNT, SUM, or AVG. Otherwise, SQL usually returns an error because it cannot determine which value to display for each group.
---

## Exercise 3: Query Writing Exercises (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Write the SQL for each task. Use the TrailShop schema (categories, customers, products, product_categories, orders, order_items).

### Simple SELECT + WHERE

**3.1** — Select product names and prices for all products with stock less than 15.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.2** — Find all customers who registered (created_at) in 2026. Show first name, last name, and registration date.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.3** — Show all products that do NOT have a description (description IS NULL).

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### Multi-condition Filtering

**3.4** — Find products in category 1 OR category 2, priced above €100, with stock greater than 0. Sort by price descending.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.5** — Find orders that are either 'delivered' or placed by customer_id 1. Show order_id, customer_id, status.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### Aggregation

**3.6** — For each category, show the category name, minimum, maximum, and average product price. Round averages to 2 decimal places. Join through `product_categories`.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.7** — Count how many distinct customers have placed at least one order.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.8** — Find the total quantity of items sold across all orders.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### JOIN Queries

**3.9** — Show each product name alongside its category name. Include all products (even if somehow a category was deleted — use LEFT JOIN).

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.10** — List all orders showing: order_id, customer full name, order date, number of items in the order, and order total cost.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.11** — Show all products that have NEVER been ordered. (Hint: LEFT JOIN order_items, then IS NULL on order_item_id.)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### GROUP BY + HAVING

**3.12** — Show categories where the average product price exceeds €100. Display category name and average price.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.13** — Find customers who have placed more than 1 order. Show their name and order count.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### Pagination

**3.14** — Write a paginated query that returns products 4 through 6 (page 2, page size 3), ordered by product_id.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### Complex

**3.15** — Write a "sales report" query that shows: category name, total units sold (from order_items), total revenue, and number of distinct products sold — for each category. Sort by revenue descending.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

---

## Exercise 4: Query Reading Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each query below, explain in **plain English** what it does and what the result would look like.

### 4.1

```sql
SELECT c.name, COUNT(p.product_id) AS num_products
FROM categories c
LEFT JOIN product_categories pc ON pc.category_id = c.category_id
LEFT JOIN products p ON p.product_id = pc.product_id
GROUP BY c.name
ORDER BY num_products DESC;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
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
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
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
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
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
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
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
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- Write the corrected statement here
>
>
> ```

---

## Submission Checklist

**Required**

- [ ] All 15 business questions answered with working SQL
- [ ] Theory review questions answered in your own words
- [ ] Results of the business questions verified by running them against your TrailShop database

**Recommended practice**

- [ ] All 15 query writing exercises completed
- [ ] All 5 query reading explanations written
