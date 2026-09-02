# Week 51 — Final Exam — Exercises (Optional)

> [!IMPORTANT] Exam Preparation
> These exercises are for your own exam preparation. **No submission is required.** The TrailShop project was submitted in Week 48.

## Practice Exam

This mini mock exam simulates the format and difficulty of the final exam. Time yourself: aim to complete it in 90 minutes.

---

### Section A: Conceptual Questions (5 questions)

**A1.** Explain the difference between a *candidate key* and a *primary key*. Give an example using a table of your choice.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

<details>
<summary>Model Answer</summary>

A **candidate key** is any column (or combination of columns) that can uniquely identify every row in a table. A **primary key** is the candidate key chosen to be the official row identifier.

Example: In a `customers` table, both `customer_id` and `email` might be candidate keys (both are unique). We choose `customer_id` as the primary key because it's shorter, stable, and not personally identifiable.
</details>

**A2.** What are the ACID properties of transactions? Briefly explain each one.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

<details>
<summary>Model Answer</summary>

- **Atomicity**: All statements in a transaction succeed or all fail — no partial execution.
- **Consistency**: A transaction brings the database from one valid state to another (constraints are satisfied).
- **Isolation**: Concurrent transactions don't see each other's uncommitted changes.
- **Durability**: Once committed, the data survives crashes (written to persistent storage).
</details>

**A3.** A colleague suggests: "Let's just put all our data in one big table instead of splitting it into many tables." Explain at least 3 problems with this approach.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

<details>
<summary>Model Answer</summary>

1. **Data redundancy**: Repeating data (e.g., customer name on every order row) wastes storage and creates inconsistency risk.
2. **Update anomalies**: Changing a customer's address requires updating many rows; missing one creates inconsistency.
3. **Deletion anomalies**: Deleting the last order for a customer might lose the customer's information entirely.
4. **Insert anomalies**: Can't store a new customer until they place an order.
5. **Performance**: Queries must scan a massive table instead of targeted smaller tables.
</details>

**A4.** What is the difference between `WHERE` and `HAVING`? Why can't you use aggregate functions in `WHERE`?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

<details>
<summary>Model Answer</summary>

`WHERE` filters individual rows **before** grouping occurs. `HAVING` filters groups **after** aggregation. You can't use aggregates in `WHERE` because at that point in query execution, groups haven't been formed yet — there's nothing to aggregate.
</details>

**A5.** Explain what an index is and describe one situation where adding an index would *hurt* performance rather than help.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Model Answer</summary>

An index is a separate data structure (typically a B-tree) that speeds up row lookup by maintaining a sorted reference to table data. 

An index hurts performance on a table with very frequent INSERT/UPDATE/DELETE operations and rare reads (e.g., a logging table). Every write must also update the index, adding overhead without benefit since the data is rarely queried.
</details>

---

### Section B: SQL Writing Tasks (5 questions)

Use this schema for questions B1–B5:

```
customers(customer_id PK, first_name, last_name, email, city, registration_date)
products(product_id PK, product_name, category_id FK, price, stock_qty)
categories(category_id PK, category_name)
orders(order_id PK, customer_id FK, order_date, status)
order_items(item_id PK, order_id FK, product_id FK, quantity, unit_price)
```

**B1.** Write a query to find the total revenue (sum of quantity × unit_price) for each category, showing only categories with revenue above €1000. Sort by revenue descending.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Model Answer</summary>

```sql
SELECT c.category_name,
       SUM(oi.quantity * oi.unit_price) AS total_revenue
FROM categories c
JOIN products p ON c.category_id = p.category_id
JOIN order_items oi ON p.product_id = oi.product_id
GROUP BY c.category_name
HAVING SUM(oi.quantity * oi.unit_price) > 1000
ORDER BY total_revenue DESC;
```
</details>

**B2.** Write a query to find customers who have placed orders but have never ordered anything from the 'Camping' category. Use EXISTS or NOT EXISTS.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Model Answer</summary>

```sql
SELECT c.first_name, c.last_name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
)
AND NOT EXISTS (
    SELECT 1
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    JOIN categories cat ON p.category_id = cat.category_id
    WHERE o.customer_id = c.customer_id
      AND cat.category_name = 'Camping'
);
```
</details>

**B3.** Create a table `wishlists` where each customer can add products to their wishlist. A customer cannot add the same product twice. Include appropriate constraints.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Model Answer</summary>

```sql
CREATE TABLE wishlists (
    wishlist_id  SERIAL PRIMARY KEY,
    customer_id  INTEGER NOT NULL REFERENCES customers(customer_id),
    product_id   INTEGER NOT NULL REFERENCES products(product_id),
    added_date   TIMESTAMP NOT NULL DEFAULT NOW(),
    CONSTRAINT uq_wishlist_customer_product UNIQUE (customer_id, product_id)
);
```
</details>

**B4.** Write a transaction that creates a new order for customer 1, adds two items, and decreases the stock for each product. Use a SAVEPOINT.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Model Answer</summary>

```sql
BEGIN;

INSERT INTO orders (customer_id, order_date, status)
VALUES (1, NOW(), 'pending');

SAVEPOINT after_order;

INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (currval('orders_order_id_seq'), 3, 1, 79.99);

UPDATE products SET stock_qty = stock_qty - 1 WHERE product_id = 3;

SAVEPOINT after_first_item;

INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (currval('orders_order_id_seq'), 7, 2, 45.00);

UPDATE products SET stock_qty = stock_qty - 2 WHERE product_id = 7;

COMMIT;
```
</details>

**B5.** Write a CTE that calculates each customer's total spending and their rank among all customers, then return only the top 5.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Model Answer</summary>

```sql
WITH customer_spending AS (
    SELECT c.customer_id, c.first_name, c.last_name,
           SUM(oi.quantity * oi.unit_price) AS total_spent,
           RANK() OVER (ORDER BY SUM(oi.quantity * oi.unit_price) DESC) AS spending_rank
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
)
SELECT first_name, last_name, total_spent, spending_rank
FROM customer_spending
WHERE spending_rank <= 5;
```
</details>

---

### Section C: Design and Normalization (3 questions)

**C1.** The following table stores order information:

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


```
orders_flat(order_id, order_date, customer_name, customer_email, customer_city,
            product_name, product_price, quantity, category_name)
```

Identify the problems with this design and normalize it to 3NF. Show the resulting tables with their columns, primary keys, and foreign keys.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

<details>
<summary>Model Answer</summary>

**Problems:**
- Customer data repeated for every order (redundancy)
- Product/category data repeated for every order item
- Update anomaly: changing a customer's city requires updating many rows
- Deletion anomaly: deleting the last order loses customer data

**3NF decomposition:**

```
customers(customer_id PK, customer_name, customer_email, customer_city)
categories(category_id PK, category_name)
products(product_id PK, product_name, product_price, category_id FK→categories)
orders(order_id PK, order_date, customer_id FK→customers)
order_items(item_id PK, order_id FK→orders, product_id FK→products, quantity)
```

Each table has no partial or transitive dependencies.
</details>

**C2.** Given this table and functional dependencies, determine the highest normal form and normalize if needed:

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


```
Table: shipments(order_id, product_id, warehouse_id, warehouse_city, ship_date, quantity)
Primary Key: (order_id, product_id)

FDs:
  (order_id, product_id) → warehouse_id, ship_date, quantity
  warehouse_id → warehouse_city
```

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

<details>
<summary>Model Answer</summary>

**Current normal form:** 2NF (no partial dependencies since all non-key attributes depend on the full key). But there is a **transitive dependency**: `(order_id, product_id) → warehouse_id → warehouse_city`.

This violates **3NF**.

**Normalized to 3NF:**

```
shipments(order_id, product_id, warehouse_id FK, ship_date, quantity)
    PK: (order_id, product_id)

warehouses(warehouse_id PK, warehouse_city)
```
</details>

**C3.** A music streaming service needs to store: artists, albums, songs, playlists, and users. A song belongs to one album, an album belongs to one artist. Users can create playlists containing many songs; a song can be in many playlists. Design a normalized schema (show tables, columns, PKs, FKs, and relationship cardinalities).

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

<details>
<summary>Model Answer</summary>

```
artists(artist_id PK, artist_name, country)

albums(album_id PK, album_title, release_year, artist_id FK→artists)

songs(song_id PK, song_title, duration_seconds, album_id FK→albums)

users(user_id PK, username, email)

playlists(playlist_id PK, playlist_name, user_id FK→users, created_date)

playlist_songs(playlist_id FK→playlists, song_id FK→songs, position)
    PK: (playlist_id, song_id)
```

**Cardinalities:**
- artist → albums: 1:N
- album → songs: 1:N
- user → playlists: 1:N
- playlist ↔ songs: M:N (via playlist_songs)
</details>

---

### Section D: Scenario Analysis (2 questions)

**D1.** A banking application transfers €100 from Account A to Account B. Write the SQL transaction. Then explain what happens if the system crashes after the first UPDATE but before COMMIT.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Model Answer</summary>

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'B';

COMMIT;
```

**If the system crashes before COMMIT:** The transaction is not committed, so PostgreSQL's durability mechanism (WAL — Write-Ahead Log) ensures that upon recovery, the incomplete transaction is **rolled back**. Neither account is modified. This is the **Atomicity** guarantee — all or nothing.
</details>

**D2.** A query on the `order_items` table is slow. You run `EXPLAIN ANALYZE` and see a "Seq Scan" on 5 million rows with a filter on `product_id`. Propose a solution, write the SQL to implement it, and explain what trade-offs your solution introduces.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

<details>
<summary>Model Answer</summary>

**Solution:** Create an index on the filtered column.

```sql
CREATE INDEX idx_order_items_product ON order_items(product_id);
```

**After creating the index**, re-run `EXPLAIN ANALYZE` — you should see "Index Scan" instead of "Seq Scan."

**Trade-offs:**
- **Positive**: SELECT queries filtering by `product_id` are dramatically faster (O(log n) lookup vs O(n) scan).
- **Negative**: INSERT/UPDATE/DELETE on `order_items` become slightly slower because the index must be maintained. The index also uses additional disk space (typically 1–3% of table size for a single-column B-tree).
</details>

---

## Study Checklist — Self-Assessment

Print this page and check off each item you feel confident about. Items left unchecked are your priority for review.

### Database Fundamentals
- [ ] I can explain what a database is and why we use one instead of files
- [ ] I can describe the relational model (tables, rows, columns, keys)
- [ ] I can identify primary keys, foreign keys, and candidate keys
- [ ] I can explain referential integrity and what happens on FK violations
- [ ] I can describe one-to-one, one-to-many, and many-to-many relationships

### SQL — Data Definition
- [ ] I can write CREATE TABLE with appropriate data types
- [ ] I can add all types of constraints (PK, FK, NOT NULL, UNIQUE, CHECK, DEFAULT)
- [ ] I can use ALTER TABLE to modify an existing table
- [ ] I can use DROP TABLE (and understand CASCADE vs RESTRICT)

### SQL — Data Manipulation
- [ ] I can INSERT single and multiple rows
- [ ] I can use INSERT...SELECT and ON CONFLICT (upsert)
- [ ] I can UPDATE rows with conditions and subqueries
- [ ] I can DELETE rows safely (always with WHERE)
- [ ] I can use RETURNING to see affected rows

### SQL — Queries
- [ ] I can write SELECT with WHERE, ORDER BY, LIMIT, OFFSET
- [ ] I can use all comparison operators (BETWEEN, IN, LIKE, IS NULL)
- [ ] I can use aggregate functions (COUNT, SUM, AVG, MIN, MAX)
- [ ] I can use GROUP BY correctly (all non-aggregate columns must be grouped)
- [ ] I can use HAVING to filter groups
- [ ] I know the difference between WHERE and HAVING

### SQL — Joins
- [ ] I can write INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL OUTER JOIN
- [ ] I can join 3 or more tables in one query
- [ ] I can write a self-join
- [ ] I understand when to use each join type

### SQL — Advanced Queries
- [ ] I can write scalar, table, and correlated subqueries
- [ ] I can use IN, EXISTS, ANY, ALL with subqueries
- [ ] I can write CTEs with the WITH clause
- [ ] I can chain multiple CTEs
- [ ] I can use UNION, INTERSECT, EXCEPT
- [ ] I can write CASE expressions (simple and searched)
- [ ] I can use COALESCE and NULLIF

### Database Design
- [ ] I can identify functional dependencies in a table
- [ ] I can determine whether a table is in 1NF, 2NF, 3NF, or BCNF
- [ ] I can normalize a table to 3NF step by step
- [ ] I can design a schema from business requirements
- [ ] I can draw an ER diagram with correct notation

### Transactions, Indexes, and Admin
- [ ] I can write a transaction with BEGIN, COMMIT, ROLLBACK
- [ ] I can use SAVEPOINT and ROLLBACK TO SAVEPOINT
- [ ] I can explain ACID properties
- [ ] I can create an index and explain when it helps
- [ ] I can read basic EXPLAIN ANALYZE output
- [ ] I can create roles and use GRANT/REVOKE
- [ ] I can perform pg_dump and pg_restore

### Query Execution and Style
- [ ] I can recite the logical query execution order (FROM → ... → LIMIT)
- [ ] I can explain why aliases can't be used in WHERE
- [ ] I follow consistent SQL formatting (uppercase keywords, snake_case names)

---

## Final Advice

1. **Don't memorize — understand.** If you understand *why* normalization exists, you can figure out the rules during the exam.
2. **Practice writing SQL by hand.** Close the computer, take a pen and paper, and write queries. Syntax errors you make on paper are the ones you'll fix before the exam.
3. **Review your TrailShop project.** It covers nearly every topic on the exam. Re-reading your own code is the fastest way to refresh.
4. **Sleep well.** A rested brain solves SQL problems faster than a caffeinated, sleep-deprived one.

Good luck. You've got this.
