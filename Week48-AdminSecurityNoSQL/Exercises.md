# Week 48 — Exercises: Database Administration, Security, and NoSQL

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting. This week includes the **last required TrailShop project work** — complete the weekly tasks and the final project submission at the end of this file.

These exercises let you practice the core topics from this week: managing access with roles and privileges, backing up and restoring databases, defending against SQL injection, and exploring NoSQL concepts — including PostgreSQL's own JSONB support. Work through them in order; the TrailShop project task builds on everything else.

> **Reminder:** All SQL in these exercises targets PostgreSQL. Use `psql` or pgAdmin to run your commands.

---

## TrailShop Project Task

This task expands your TrailShop database with real-world administration and security work. You will create roles, test permissions, practice backup/restore, defend against injection, and prototype a new feature using both relational and document-style design.

**Tables you are working with:**
`products`, `customers`, `orders`, `order_items`, `categories`, `payments`

---

### Task 1 — Create Roles and Grant Privileges

Create three roles that reflect common access patterns in a web application. Connect to your TrailShop database as a superuser and run the following:

**a) Analyst role — read-only access to everything**

```sql
CREATE ROLE ts_analyst LOGIN PASSWORD 'analyst_pass';

GRANT USAGE ON SCHEMA public TO ts_analyst;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO ts_analyst;
```

**b) Warehouse role — read everything, update stock on products**

```sql
CREATE ROLE ts_warehouse LOGIN PASSWORD 'warehouse_pass';

GRANT USAGE ON SCHEMA public TO ts_warehouse;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO ts_warehouse;
GRANT UPDATE (stock_quantity) ON products TO ts_warehouse;
```

**c) Application role — read/write access for the web app**

```sql
CREATE ROLE ts_app LOGIN PASSWORD 'app_pass';

GRANT USAGE ON SCHEMA public TO ts_app;
GRANT SELECT, INSERT, UPDATE ON products TO ts_app;
GRANT SELECT, INSERT, UPDATE ON orders TO ts_app;
GRANT SELECT, INSERT, UPDATE ON order_items TO ts_app;
GRANT SELECT, INSERT, UPDATE ON customers TO ts_app;
```

> Think about it: why does `ts_app` not get DELETE on any table? Why does `ts_warehouse` only get UPDATE on a single column?



> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

### Task 2 — Test Permissions

Connect to the database as each role and verify that permissions work as expected. Document your results.

**a) Connect as ts_analyst:**

```sql
-- This should succeed:
SELECT * FROM products LIMIT 5;

-- This should fail:
INSERT INTO products (name, price) VALUES ('Test', 9.99);

-- This should also fail:
UPDATE products SET stock_quantity = 0 WHERE product_id = 1;
```

**b) Connect as ts_warehouse:**

```sql
-- This should succeed:
SELECT * FROM orders LIMIT 5;

-- This should succeed (column-level grant):
UPDATE products SET stock_quantity = 50 WHERE product_id = 1;

-- This should fail (no UPDATE on price):
UPDATE products SET price = 0.01 WHERE product_id = 1;

-- This should fail:
DELETE FROM products WHERE product_id = 1;
```

**c) Connect as ts_app:**

```sql
-- This should succeed:
INSERT INTO customers (first_name, last_name, email)
VALUES ('Test', 'User', 'test@example.com');

-- This should succeed:
UPDATE products SET price = 29.99 WHERE product_id = 1;

-- This should fail (no access to payments):
SELECT * FROM payments;

-- This should fail:
DROP TABLE products;
```

**Document your results:** For each query, record whether it succeeded or failed and what error message you received. Does every result match your expectations?



> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

### Task 3 — Backup with pg_dump

Create a backup of your TrailShop database in custom format. Run this in your operating system terminal (not inside psql):

```bash
pg_dump -U postgres -Fc trailshop > trailshop_backup.dump
```

Verify the backup file was created:

```bash
# Check file exists and has a reasonable size
ls -lh trailshop_backup.dump
```

You can also inspect the backup contents without restoring:

```bash
pg_restore --list trailshop_backup.dump
```

> The `-Fc` flag creates a custom-format archive. This format supports selective restore (individual tables) and is compressed automatically.

---

### Task 4 — Selective Restore

Practice restoring a single table from your backup.

**a) Drop the order_items table:**

```sql
DROP TABLE order_items;
```

**b) Restore only that table from the backup:**

```bash
pg_restore -U postgres -d trailshop -t order_items trailshop_backup.dump
```

**c) Verify the restore worked:**

```sql
SELECT COUNT(*) FROM order_items;
SELECT * FROM order_items LIMIT 5;
```

> If you get a foreign key error during restore, you may need to restore the table without constraints first, then add them back. This is a common real-world challenge.

---

### Task 5 — SQL Injection Exercise

**a) Vulnerable code — do NOT use this pattern:**

```python
# VULNERABLE — string concatenation builds the SQL
def search_products(search_term):
    query = "SELECT * FROM products WHERE name LIKE '%" + search_term + "%'"
    cursor.execute(query)
    return cursor.fetchall()

# An attacker could pass:
# search_term = "'; DROP TABLE products; --"
# Resulting SQL:
# SELECT * FROM products WHERE name LIKE '%'; DROP TABLE products; --%'
```

**b) Safe version — parameterized query:**

```python
# SAFE — the database driver handles escaping
def search_products(search_term):
    query = "SELECT * FROM products WHERE name LIKE %s"
    cursor.execute(query, (f"%{search_term}%",))
    return cursor.fetchall()
```

**Explain the difference:** In the vulnerable version, user input is pasted directly into the SQL string, so the database cannot distinguish between code and data. In the parameterized version, the query structure is sent separately from the values — the database knows exactly which parts are SQL and which are data, making injection impossible.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

### Task 6 — Design a Reviews Feature (Relational vs. Document)

Design a product reviews feature for TrailShop using both approaches.

**a) Relational design:**

```sql
CREATE TABLE reviews (
    review_id   SERIAL PRIMARY KEY,
    product_id  INTEGER NOT NULL REFERENCES products(product_id),
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
    rating      SMALLINT NOT NULL CHECK (rating BETWEEN 1 AND 5),
    title       VARCHAR(200),
    body        TEXT,
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**b) JSON document design (as you might store it in MongoDB or a JSONB column):**

```json
{
  "product_id": 42,
  "product_name": "Trail Runner Pro",
  "reviews": [
    {
      "customer": { "id": 7, "name": "Anna M." },
      "rating": 5,
      "title": "Best trail shoes ever",
      "body": "Comfortable from day one, great grip on wet rocks.",
      "created_at": "2026-10-15T14:30:00Z"
    },
    {
      "customer": { "id": 12, "name": "Erik L." },
      "rating": 3,
      "title": "Good but runs narrow",
      "body": "Quality is fine but I needed a half size up.",
      "created_at": "2026-11-02T09:15:00Z"
    }
  ]
}
```

**c) Comparison — write your own version of these points:**

1. **Data integrity:** The relational design enforces foreign keys and check constraints at the database level; the document design relies on application code to validate.
2. **Read performance:** The document design returns a product and all its reviews in a single read with no joins; the relational design requires a JOIN.
3. **Write flexibility:** Adding a new field (e.g., `helpful_count`) is trivial in the document — just include it. In the relational model you need an `ALTER TABLE`.
4. **Querying across reviews:** "Find all reviews with rating >= 4" is a simple `WHERE` clause in the relational model but requires reaching into nested arrays in the document model.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

## Theory Review

Answer each question in your own words. Refer back to the lecture material when needed.

1. **What is the difference between a role and a user in PostgreSQL?**
   *(Hint: In modern PostgreSQL, the distinction is simpler than you might expect.)*


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>



2. **What does `GRANT USAGE ON SCHEMA` do, and why is it needed before granting table-level privileges?**


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>



3. **What are default privileges and when would you use them?**
   *(Hint: think about tables that will be created in the future.)*


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>



4. **What is the difference between `pg_dump` and `pg_dumpall`?**
   *(Consider: what does each one back up?)*


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>



5. **Explain how SQL injection works in 2–3 sentences.**


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>



6. **Why are parameterized queries the primary defense against SQL injection?**
   *(What fundamental problem do they solve?)*


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>



7. **Name four types of NoSQL databases and give one use case for each.**
   *(Document, key-value, column-family, graph — what is each good at?)*


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>



8. **What is the CAP theorem?**
   *(Briefly explain the three properties and the trade-off.)*


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>



9. **What is JSONB and how does it differ from JSON in PostgreSQL?**
   *(Think about storage, indexing, and performance.)*


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>



10. **When would you choose a document database over a relational database?**
    *(Give at least two concrete scenarios.)*

---


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>



## Security Exercise — Spot and Fix the Vulnerabilities

Below are three code snippets that contain SQL injection vulnerabilities. For each one: identify the vulnerability, show a malicious input that would exploit it, and rewrite the code safely.

---

### Vulnerability 1 — Product Search

```python
# VULNERABLE
def search_products(category, min_price):
    query = f"SELECT * FROM products WHERE category = '{category}' AND price >= {min_price}"
    cursor.execute(query)
    return cursor.fetchall()
```

**Malicious input:**

```
category = "Shoes' OR '1'='1' --"
```

This would produce:

```sql
SELECT * FROM products WHERE category = 'Shoes' OR '1'='1' --' AND price >= 10
```

The `--` comments out the rest, and `'1'='1'` is always true — the attacker sees every product.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
**Safe version:**

```python
def search_products(category, min_price):
    query = "SELECT * FROM products WHERE category = %s AND price >= %s"
    cursor.execute(query, (category, min_price))
    return cursor.fetchall()
```

---

### Vulnerability 2 — User Login

```python
# VULNERABLE
def login(username, password):
    query = f"SELECT * FROM customers WHERE email = '{username}' AND password_hash = '{password}'"
    cursor.execute(query)
    user = cursor.fetchone()
    return user is not None
```

**Malicious input:**

```
username = "admin@trailshop.com' --"
password = "anything"
```

This produces:

```sql
SELECT * FROM customers WHERE email = 'admin@trailshop.com' --' AND password_hash = 'anything'
```

The password check is commented out entirely — the attacker logs in as any user.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
**Safe version:**

```python
def login(username, password):
    query = "SELECT * FROM customers WHERE email = %s AND password_hash = %s"
    cursor.execute(query, (username, password))
    user = cursor.fetchone()
    return user is not None
```

> In a real application you would also hash the password in your application code before comparing — never store or compare plaintext passwords.

---

### Vulnerability 3 — Inserting User Data

```python
# VULNERABLE
def add_customer(first_name, last_name, email):
    query = f"""INSERT INTO customers (first_name, last_name, email)
                VALUES ('{first_name}', '{last_name}', '{email}')"""
    cursor.execute(query)
    connection.commit()
```

**Malicious input:**

```
email = "test@x.com'); DELETE FROM customers; --"
```

This produces:

```sql
INSERT INTO customers (first_name, last_name, email)
VALUES ('Anna', 'Smith', 'test@x.com'); DELETE FROM customers; --')
```

The attacker's input closes the INSERT and adds a DELETE that wipes the entire customers table.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
**Safe version:**

```python
def add_customer(first_name, last_name, email):
    query = """INSERT INTO customers (first_name, last_name, email)
               VALUES (%s, %s, %s)"""
    cursor.execute(query, (first_name, last_name, email))
    connection.commit()
```

---

## NoSQL Design Exercise — Blog System

Design a blog system with **users**, **posts**, **comments**, and **tags** using both relational and document approaches.

### Relational Design

```sql
CREATE TABLE users (
    user_id    SERIAL PRIMARY KEY,
    username   VARCHAR(50) UNIQUE NOT NULL,
    email      VARCHAR(255) UNIQUE NOT NULL,
    bio        TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE tags (
    tag_id   SERIAL PRIMARY KEY,
    name     VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE posts (
    post_id    SERIAL PRIMARY KEY,
    user_id    INTEGER NOT NULL REFERENCES users(user_id),
    title      VARCHAR(300) NOT NULL,
    body       TEXT NOT NULL,
    status     VARCHAR(20) DEFAULT 'draft' CHECK (status IN ('draft', 'published', 'archived')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE post_tags (
    post_id INTEGER REFERENCES posts(post_id) ON DELETE CASCADE,
    tag_id  INTEGER REFERENCES tags(tag_id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, tag_id)
);

CREATE TABLE comments (
    comment_id SERIAL PRIMARY KEY,
    post_id    INTEGER NOT NULL REFERENCES posts(post_id) ON DELETE CASCADE,
    user_id    INTEGER NOT NULL REFERENCES users(user_id),
    body       TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Document Design

**User document:**

```json
{
  "_id": "user_1",
  "username": "mountain_hiker",
  "email": "hiker@example.com",
  "bio": "I hike and write about it.",
  "created_at": "2026-06-01T10:00:00Z"
}
```

**Post document (with embedded comments and tags):**

```json
{
  "_id": "post_42",
  "author": {
    "user_id": "user_1",
    "username": "mountain_hiker"
  },
  "title": "Best Trails in Lakeland Finland",
  "body": "If you love forests and lakes, these trails are for you...",
  "status": "published",
  "tags": ["hiking", "finland", "nature"],
  "comments": [
    {
      "user_id": "user_5",
      "username": "trail_runner",
      "body": "Great list! I would add the Repovesi loop.",
      "created_at": "2026-06-15T08:30:00Z"
    },
    {
      "user_id": "user_8",
      "username": "nordic_adventures",
      "body": "Visited three of these last summer. Highly recommend!",
      "created_at": "2026-06-16T14:20:00Z"
    }
  ],
  "created_at": "2026-06-14T12:00:00Z",
  "updated_at": "2026-06-14T12:00:00Z"
}
```

**Comment-heavy post document (alternative — comments in a separate collection):**

```json
{
  "_id": "comment_101",
  "post_id": "post_42",
  "user_id": "user_5",
  "username": "trail_runner",
  "body": "Great list! I would add the Repovesi loop.",
  "created_at": "2026-06-15T08:30:00Z"
}
```

### Comparison

Write your own analysis using these five points as a starting framework:

1. **Reading a full post with comments:** The document model returns everything in one query. The relational model needs joins across `posts`, `comments`, `users`, and `post_tags`. Document wins for this read pattern.

2. **Updating a username:** In the relational model you update one row in `users`. In the document model the username is duplicated in every post and comment — you must update all of them. Relational wins for data consistency.

3. **Querying across posts:** "Find all posts tagged 'hiking' with more than 5 comments" is straightforward in SQL with GROUP BY and HAVING. In the document model you need to query inside arrays, which is possible but less natural.

4. **Schema flexibility:** Adding a `likes_count` to posts requires ALTER TABLE in the relational model. In documents you just start including the field — old documents without it still work fine.

5. **Data integrity:** The relational model guarantees (via foreign keys) that every comment references a real post and a real user. Documents have no such built-in enforcement — an application bug could create orphaned or inconsistent data.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

## JSONB Practice

For these exercises, first create a table to work with:

```sql
CREATE TABLE product_reviews (
    review_id   SERIAL PRIMARY KEY,
    product_id  INTEGER NOT NULL REFERENCES products(product_id),
    review_data JSONB NOT NULL,
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO product_reviews (product_id, review_data) VALUES
(1, '{"rating": 5, "title": "Excellent!", "body": "Best purchase this year.", "author": {"name": "Anna", "verified": true}, "tags": ["quality", "value"]}'),
(1, '{"rating": 3, "title": "Decent", "body": "Good but overpriced.", "author": {"name": "Erik", "verified": false}, "tags": ["overpriced"]}'),
(2, '{"rating": 4, "title": "Very good", "body": "Comfortable and durable.", "author": {"name": "Maria", "verified": true}, "tags": ["comfort", "durable"]}'),
(2, '{"rating": 1, "title": "Broke after a week", "body": "Returned it immediately.", "author": {"name": "Juha", "verified": true}, "tags": ["defective"]}'),
(3, '{"rating": 5, "title": "Perfect gift", "body": "Bought this for a friend, they loved it.", "author": {"name": "Liisa", "verified": false}, "tags": ["gift", "quality"]}');
```

---

### Query 1 — Extract text with `->>` operator

Select the review title and author name from all reviews:

```sql
SELECT
    review_data->>'title'                AS review_title,
    review_data->'author'->>'name'       AS author_name,
    (review_data->>'rating')::INTEGER    AS rating
FROM product_reviews;
```

> The `->>` operator extracts a value as text. The `->` operator extracts it as a JSON object (useful for nested access).

---

### Query 2 — Check for a key with `?` operator

Find all reviews that contain a `tags` key:

```sql
SELECT review_id, review_data->>'title' AS title
FROM product_reviews
WHERE review_data ? 'tags';
```

Now try the opposite — find reviews that do NOT have a `tags` key:

```sql
SELECT review_id, review_data->>'title' AS title
FROM product_reviews
WHERE NOT (review_data ? 'tags');
```

---

### Query 3 — Containment with `@>` operator

Find all reviews by verified authors:

```sql
SELECT review_id, review_data->>'title' AS title
FROM product_reviews
WHERE review_data @> '{"author": {"verified": true}}';
```

Find all reviews with a rating of 5:

```sql
SELECT review_id, review_data->>'title' AS title
FROM product_reviews
WHERE review_data @> '{"rating": 5}';
```

> The `@>` operator checks if the left JSONB value contains the right JSONB value. It works with nested structures.

---

### Query 4 — Update with `jsonb_set`

Add a `helpful_count` field to a specific review:

```sql
UPDATE product_reviews
SET review_data = jsonb_set(review_data, '{helpful_count}', '0')
WHERE review_id = 1;
```

Update the author name in a review:

```sql
UPDATE product_reviews
SET review_data = jsonb_set(review_data, '{author,name}', '"Anna M."')
WHERE review_id = 1;
```

Verify your changes:

```sql
SELECT review_data FROM product_reviews WHERE review_id = 1;
```

> The second argument to `jsonb_set` is a path (as a text array), and the third is the new value (as JSONB).

---

### Query 5 — GIN Index

Create a GIN index on the JSONB column:

```sql
CREATE INDEX idx_reviews_data ON product_reviews USING GIN (review_data);
```

This index accelerates queries that use `@>`, `?`, `?|`, and `?&` operators. For example:

```sql
-- This query benefits from the GIN index:
SELECT review_id, review_data->>'title' AS title
FROM product_reviews
WHERE review_data @> '{"rating": 5}';
```

You can verify the index is being used with EXPLAIN:

```sql
EXPLAIN SELECT review_id, review_data->>'title'
FROM product_reviews
WHERE review_data @> '{"rating": 5}';
```

> With a small table the planner may choose a sequential scan anyway. GIN indexes show their value on larger datasets — thousands of rows and up.

---

## Final TrailShop Project Submission

This is the **last required project work** for the course. After completing the weekly tasks above, submit your complete TrailShop project package as described below. Weeks 49–51 are reserved for optional exam review — no further project submissions are required.

### Submission Requirements

Your final TrailShop project submission must include:

1. **Complete SQL script** (`trailshop_final.sql`) that:
   - Creates all tables with appropriate data types and constraints
   - Establishes all relationships (foreign keys)
   - Inserts sample data (at least 5 rows per table)
   - Includes at least 2 views
   - Includes at least 3 indexes (with justification in comments)
   - Creates at least 2 roles with appropriate privileges
   - Contains at least 1 transaction demonstration

2. **ER diagram** (image or PDF) showing your final schema with:
   - All entities and their attributes
   - Primary keys marked
   - Relationships with cardinality notation
   - Foreign keys indicated

3. **Query collection** (`trailshop_queries.sql`) containing at least 10 queries that demonstrate:
   - Multi-table joins (at least 3 tables)
   - Aggregation with GROUP BY and HAVING
   - Subqueries (at least one correlated)
   - A CTE
   - CASE expressions
   - Set operations (at least one)

4. **Reflection document** (`reflection.md`) — 300–500 words answering:
   - What was the most challenging part of building this database?
   - If you were to redesign it, what would you change and why?
   - How does your final schema differ from your Week 38 version? What drove the changes?
   - What did you learn about database design that you didn't expect?

### Grading Criteria

| Criterion | Weight |
|---|---|
| Schema design (appropriate tables, types, constraints) | 25% |
| Relationships and referential integrity | 20% |
| Query variety and correctness | 25% |
| Security, indexes, and transactions | 15% |
| Reflection quality and depth | 15% |

---

## Summary

After completing these exercises you should be able to:

- Create roles and grant fine-grained privileges in PostgreSQL
- Back up a database and restore individual tables
- Recognize SQL injection vulnerabilities and fix them with parameterized queries
- Design data using both relational tables and JSON documents
- Query and manipulate JSONB data in PostgreSQL

Good work — these are skills you will use in every database project going forward. Submit your final TrailShop project package when you have finished this week's tasks.
