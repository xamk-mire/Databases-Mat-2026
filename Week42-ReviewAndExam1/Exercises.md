# Week 42 — Exercises: Review and Exam 1 Preparation

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

## Exercise 1: TrailShop Project Task — Review and Verify

Before the exam, verify that your TrailShop database is complete and correct.

### Task 1.1: Schema Verification

Run the following checks and fix any issues:

1. Confirm all 5 tables exist: `\dt`
2. Confirm column definitions: `\d categories`, `\d customers`, `\d products`, `\d orders`, `\d order_items`
3. Verify all PRIMARY KEY constraints exist
4. Verify all FOREIGN KEY constraints exist
5. Verify CHECK constraints on `price`, `stock`, `quantity`, `status`
6. Verify NOT NULL constraints on required columns

### Task 1.2: Data Verification

1. Run `SELECT COUNT(*) FROM table_name;` for each table — confirm you have data
2. Run a query that joins all 5 tables to confirm relationships work:

```sql
SELECT c.first_name, cat.name AS category, p.name AS product,
       oi.quantity, o.status
FROM order_items oi
JOIN orders o ON oi.order_id = o.order_id
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON oi.product_id = p.product_id
JOIN categories cat ON p.category_id = cat.category_id
ORDER BY c.first_name, o.order_id;
```

3. Test referential integrity: try to INSERT a product with a non-existent category_id — confirm it fails
4. Test CHECK constraint: try to UPDATE a product price to -5 — confirm it fails

---

## Exercise 2: PetStore Mini-Case

This is a complete practice case similar to what you might see on the exam. Work through it from start to finish.

### 2.1 Business Description

**PetStore** is a small shop selling pet supplies. They need a database to track:
- **Product categories**: name (e.g., "Dog Food", "Cat Toys", "Fish Supplies")
- **Products**: name, description, price, stock quantity, which category they belong to
- **Customers**: first name, last name, email, phone
- **Sales**: which customer bought what product, how many, the price at time of sale, and the date

### 2.2 ER Diagram (Conceptual)

Draw or describe the ER diagram for PetStore:
- Entities: categories, products, customers, sales
- Relationships: products belong to categories (M:1), sales link customers and products (junction-like)

### 2.3 CREATE TABLE Statements

Write the CREATE TABLE statements:

```sql
CREATE TABLE pet_categories (
    category_id   SERIAL PRIMARY KEY,
    name          VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE pet_products (
    product_id    SERIAL PRIMARY KEY,
    name          VARCHAR(200) NOT NULL,
    description   TEXT,
    price         NUMERIC(8,2) NOT NULL CHECK (price > 0),
    stock         INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
    category_id   INTEGER NOT NULL REFERENCES pet_categories(category_id)
);

CREATE TABLE pet_customers (
    customer_id   SERIAL PRIMARY KEY,
    first_name    VARCHAR(100) NOT NULL,
    last_name     VARCHAR(100) NOT NULL,
    email         VARCHAR(255) NOT NULL UNIQUE,
    phone         VARCHAR(20)
);

CREATE TABLE pet_sales (
    sale_id       SERIAL PRIMARY KEY,
    customer_id   INTEGER NOT NULL REFERENCES pet_customers(customer_id),
    product_id    INTEGER NOT NULL REFERENCES pet_products(product_id),
    quantity      INTEGER NOT NULL CHECK (quantity > 0),
    unit_price    NUMERIC(8,2) NOT NULL CHECK (unit_price > 0),
    sale_date     TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### 2.4 Sample Data

```sql
INSERT INTO pet_categories (name) VALUES
    ('Dog Food'), ('Cat Toys'), ('Fish Supplies'), ('Bird Cages'), ('Small Animal');

INSERT INTO pet_products (name, price, stock, category_id) VALUES
    ('Premium Kibble 10kg', 45.99, 30, 1),
    ('Grain-Free Wet Food 12-pack', 29.99, 50, 1),
    ('Laser Pointer', 12.99, 100, 2),
    ('Feather Wand', 8.99, 75, 2),
    ('Aquarium Filter 50L', 34.99, 20, 3),
    ('Tropical Fish Flakes', 6.99, 200, 3),
    ('Parakeet Cage Large', 89.99, 10, 4),
    ('Hamster Wheel', 14.99, 40, 5);

INSERT INTO pet_customers (first_name, last_name, email, phone) VALUES
    ('Liisa', 'Koivisto', 'liisa.k@email.com', '040-1234567'),
    ('Markus', 'Lehtinen', 'markus.l@email.com', NULL),
    ('Tiina', 'Järvinen', 'tiina.j@email.com', '050-9876543');

INSERT INTO pet_sales (customer_id, product_id, quantity, unit_price) VALUES
    (1, 1, 2, 45.99),
    (1, 3, 1, 12.99),
    (2, 5, 1, 34.99),
    (2, 6, 3, 6.99),
    (3, 4, 2, 8.99),
    (1, 7, 1, 89.99);
```

### 2.5 Practice Queries with Solutions

Write SQL for each, then check against the solution.

**Q1: List all products with their category names, sorted by category then price.**

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
>




<details>
<summary>Solution</summary>

```sql
SELECT pc.name AS category, pp.name AS product, pp.price
FROM pet_products pp
INNER JOIN pet_categories pc ON pp.category_id = pc.category_id
ORDER BY pc.name, pp.price;
```
</details>

**Q2: How much has each customer spent in total?**

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
>




<details>
<summary>Solution</summary>

```sql
SELECT c.first_name || ' ' || c.last_name AS customer,
       SUM(s.quantity * s.unit_price) AS total_spent
FROM pet_customers c
INNER JOIN pet_sales s ON c.customer_id = s.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC;
```
</details>

**Q3: Which categories have products that have never been sold?**

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
>




<details>
<summary>Solution</summary>

```sql
SELECT DISTINCT pc.name AS category
FROM pet_products pp
INNER JOIN pet_categories pc ON pp.category_id = pc.category_id
LEFT JOIN pet_sales s ON pp.product_id = s.product_id
WHERE s.sale_id IS NULL;
```
</details>

**Q4: What is the total revenue per category?**

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
>




<details>
<summary>Solution</summary>

```sql
SELECT pc.name AS category,
       COALESCE(SUM(s.quantity * s.unit_price), 0) AS revenue
FROM pet_categories pc
LEFT JOIN pet_products pp ON pc.category_id = pp.category_id
LEFT JOIN pet_sales s ON pp.product_id = s.product_id
GROUP BY pc.name
ORDER BY revenue DESC;
```
</details>

**Q5: Find the most purchased product (by total quantity sold).**

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
>




<details>
<summary>Solution</summary>

```sql
SELECT pp.name, SUM(s.quantity) AS total_sold
FROM pet_products pp
INNER JOIN pet_sales s ON pp.product_id = s.product_id
GROUP BY pp.product_id, pp.name
ORDER BY total_sold DESC
LIMIT 1;
```
</details>

---

## Exercise 3: Additional Practice — Mixed Topics

These 10 questions cover material from Weeks 36–41. Write your answers without looking at notes first, then check.

### 3.1 (Week 36 — Theory)
Name three problems of file-based data management and explain how a database solves each one.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 3.2 (Week 37 — Theory)
What is the difference between a candidate key and a primary key? Can a table have multiple candidate keys?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 3.3 (Week 37 — Theory)
State the referential integrity rule. What happens in PostgreSQL if you try to INSERT a row that violates it?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 3.4 (Week 38 — Design)
A library lends books to members. A member can borrow many books; a book can be borrowed by many members (over time). Draw or describe the ER model, including the junction entity.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 3.5 (Week 39 — Design)
For the library scenario above, write the CREATE TABLE statements (at least 3 tables with appropriate keys and constraints).

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.6 (Week 40 — DDL)
Write an ALTER TABLE statement that adds a `discount_percent NUMERIC(4,2) CHECK (discount_percent BETWEEN 0 AND 50)` column to a `products` table.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.7 (Week 40 — DML)
Write an INSERT statement that adds 3 rows to a `books` table with columns: title (VARCHAR), author (VARCHAR), isbn (CHAR(13)), price (NUMERIC). Make up realistic data.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.8 (Week 41 — Querying)
Explain in plain English what this query returns:
```sql
SELECT c.name, COUNT(p.product_id)
FROM categories c
LEFT JOIN products p ON c.category_id = p.category_id
GROUP BY c.name
HAVING COUNT(p.product_id) = 0;
```

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 3.9 (Week 41 — Querying)
Write a query that finds the top 3 customers by total spending, showing their full name and total amount spent.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.10 (Week 41 — Querying)
Explain why this query is invalid:
```sql
SELECT category_id, name, AVG(price)
FROM products
GROUP BY category_id;
```

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

## Exercise 4: Timed Practice Exercise (30 minutes)

Set a timer for 30 minutes. Complete as much as possible without looking at notes.

### Scenario: BookClub Database

A book club tracks its members, books, and reading sessions.

**Requirements:**
- Members: first_name, last_name, email (unique), join_date
- Books: title, author, isbn (unique, 13 chars), genre, page_count
- Reading sessions: which member read which book, start_date, end_date (nullable — still reading), rating (1-5, nullable — not yet rated)

### Tasks:

**Task A (5 min):** Write CREATE TABLE statements for all three tables with appropriate constraints.

**Task B (3 min):** Insert 3 members, 4 books, and 5 reading sessions.

**Task C (2 min):** Write an UPDATE that sets the end_date of member 1's reading of book 2 to today, and the rating to 4.

**Task D (5 min):** Write queries for:
1. All books that member 1 has read (show title, rating)
2. Average rating per book (only completed readings)
3. Members who have never rated any book below 3

**Task E (5 min):** Write a query showing: member name, number of books read, number of books currently reading (end_date IS NULL), average rating given.

**Task F (5 min):** Write a query finding books that NO member has read yet.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Paste your SQL for Tasks A–F here
>
>
> ```

**Task G (5 min):** Write ALTER TABLE statements to:
1. Add a `pages_read INTEGER DEFAULT 0` column to reading sessions
2. Add a CHECK constraint ensuring pages_read <= the book's page_count (Hint: this requires a trigger or application logic — explain why a simple CHECK can't do cross-table validation)

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your ALTER TABLE statements here
>
>
> ```

> [!NOTE]
> ***Your Answer***
>
> *(Explain why a simple CHECK constraint cannot validate pages_read against the book's page_count across tables.)*
>
>
>
>

---

## Submission Checklist

- [ ] TrailShop database verified (schema + data + constraints tested)
- [ ] PetStore mini-case completed (tables created, data inserted, all 5 queries working)
- [ ] All 10 mixed-topic questions answered
- [ ] Timed practice completed (note how much you finished in 30 minutes)
- [ ] Self-assessment checklist from Theory filled in — study weak areas before the exam
