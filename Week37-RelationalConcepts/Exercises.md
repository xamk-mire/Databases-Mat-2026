# Week 37 — Exercises & Project Task

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 37 Theory material. Complete all sections.

---

## Part 1: TrailShop Project Task

### Task 1: Identify Keys

Using the `products`, `categories`, and `customers` tables shown in Section 2 of this week's Theory material, answer:

1. What is the primary key of the `products` table? Why is it a good choice?

> [!NOTE]
> ***Your Answer***
>
> The primary key of the products table is `product_id`. I think it is a good choice because every product has its own ID, so it can be used to identify each product. It is also more stable than using something like the product name or price.
>
>
>


2. What is the primary key of the `categories` table?

> [!NOTE]
> ***Your Answer***
>
> The primary key of the categories table is `category_id`. It is used to identify each category separately.
>
>
>
>


3. What is the foreign key in the `products` table? What does it reference?

> [!NOTE]
> ***Your Answer***
>
> The foreign key in the products table is `category_id`. It references `category_id` in the categories table. This connects each product to its category.
>
>
>
>


4. Is `name` in `products` a candidate key? Under what assumption? What would make it unsuitable as a primary key?


> [!NOTE]
> ***Your Answer***
>
> Yes, `name` can be a candidate key if every product has a different name. But I don't think it is a good primary key because the product name can change, and two products could have the same name. So, `product_id` is a better choice.
>
>
>
>
5. Give an example of a **superkey** for the `products` table that is NOT a candidate key. Explain why it's not minimal.

> [!NOTE]
> ***Your Answer***
>
> One example is `{product_id, name}`. It is a superkey because it can identify each product. But it is not a candidate key because `product_id` alone is already enough to identify the product, so `name` is not needed.
>
>
>
>

6. Give an example of a **composite key** using a hypothetical `order_items` table. Explain why neither column alone would be sufficient.

> [!NOTE]
> ***Your Answer***
>
>A good example is `(order_id, product_id)`. `order_id` alone is not enough because one order can have more than one product. `product_id` alone is also not enough because the same product can be in different orders. Together, they can identify one product in one order.
>
>
>
>

7. Is `email` in `customers` a candidate key? What makes it different from `customer_id` as a PK choice? *(See Section 6.9 on natural vs surrogate keys.)*

> [!NOTE]
> ***Your Answer***
>
>Yes, `email` can be a candidate key if every customer has a unique email address. The difference is that email has a real meaning and can change. `customer_id` is just an ID made for the database, so it is more stable and is a better choice for the primary key.
>
>
>
>

### Task 2: Define Business Rules

List **5 business rules** for TrailShop. For each rule, specify:
- The rule in plain English
- Which constraint type(s) would enforce it
- Which table and column the constraint applies to
- The SQL syntax for the constraint

Example:

| Business Rule | Constraint Type | Table.Column | SQL |
|---|---|---|---|
| Every product must have a price greater than zero | CHECK | products.price | `CHECK (price > 0)` |
| ... | ... | ... | ... |

Think about rules for customers, orders, and categories — not just products.

> [!NOTE]
> ***Your Answer***
>
> | Business Rule | Constraint Type | Table.Column | SQL Syntax |
> |---|---|---|---|
> | Every product must have a name | NOT NULL | products.name | `name VARCHAR(100) NOT NULL` |
> | Every product must have a price greater than zero | CHECK | products.price | `CHECK (price > 0)` |
> | Category names must be unique | UNIQUE | categories.category_name | `UNIQUE (category_name)` |
> | Customer email must be unique | UNIQUE | customers.email | `UNIQUE (email)` |
> | Every order must belong to an existing customer | NOT NULL + FOREIGN KEY | orders.customer_id | `customer_id INTEGER NOT NULL REFERENCES customers(customer_id)` |
### Task 3: Integrity Violations

For each SQL statement below, predict whether it will **succeed** or **fail**. If it fails, explain which integrity rule or constraint is violated and what error message you'd expect. Assume the schema from Section 9.8 of the Theory material.

```sql
-- Statement A
INSERT INTO categories (category_id, category_name)
VALUES (NULL, 'Cycling');

-- Statement B
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (109, 'AeroLite Tent', 279.00, 10, 2);

-- Statement C
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (110, 'BudgetBoots', -5.00, 25, 1);

-- Statement D
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (103, 'Duplicate Shoes', 99.99, 5, 3);

-- Statement E
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (111, 'CloudWalker Sandals', 65.00, 40, 10);

-- Statement F
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (112, NULL, 89.99, 20, 1);

-- Statement G
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (113, 'LightStep Shoes', 149.00, -3, 1);

-- Statement H
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (1001, 101, 0, 189.50);
```

> [!NOTE]
> ***Your Answer***
>
> Statement A: FAIL — category_id is the primary key, so it cannot be NULL. This violates the PRIMARY KEY constraint.


Statement B: SUCCESS — The product_id is unique, the price and stock quantity are valid, and category_id 2 exists.

Statement C: FAIL — The price is -5.00, which is not greater than zero. This violates the CHECK constraint on price.

Statement D: FAIL — product_id 103 already exists. This violates the PRIMARY KEY constraint.

Statement E: FAIL — category_id 10 does not exist in the categories table. This violates the FOREIGN KEY constraint.

Statement F: FAIL — The product name is NULL, but the name is required. This violates the NOT NULL constraint.

Statement G: FAIL — stock_quantity is -3, which is negative. This violates the CHECK constraint on stock_quantity.

Statement H: FAIL — quantity is 0, but the quantity must be greater than zero. This violates the CHECK constraint on quantity.
>
>
>

### Task 4: Foreign Key Actions

Consider the following scenario using the schema from Theory Section 9.8:

1. You want to delete category 2 ("Camping") from the `categories` table. Products 102 and 106 reference this category. What happens with:
   - `ON DELETE RESTRICT`?
   - `ON DELETE CASCADE`?
   - `ON DELETE SET NULL`? (Assume `category_id` in `products` allows NULL for this question)

2. Which foreign key action would you recommend for the TrailShop `products.category_id` → `categories.category_id` relationship? Justify your choice in 2–3 sentences.

> [!NOTE]
> ***Your Answer***
>
>1. ON DELETE RESTRICT:
The delete will fail because products 102 and 106 still reference category 2.

2. ON DELETE CASCADE:
The delete will succeed. Category 2 will be deleted, and products 102 and 106 will also be deleted.

3. ON DELETE SET NULL:
The delete will succeed. Category 2 will be deleted, and the category_id of products 102 and 106 will be changed to NULL.

4. Recommendation:
I would use ON DELETE RESTRICT for the products.category_id → categories.category_id relationship. This prevents a category from being deleted while products still depend on it, so it helps avoid accidental deletion of product data.

>
>
>




---

## Part 2: Theory Review Questions

Answer each question in 2–4 sentences unless otherwise specified. Reference the Theory material sections as needed.

### Short-Answer Questions

**Q1.** Define the following terms in your own words: relation, tuple, attribute, domain. Give one TrailShop example for each.

> [!NOTE]
> ***Your Answer***
>
> A relation is a table in a database. A tuple is one row, and an attribute is one column. A domain is the set of allowed values for an attribute. For example, the products table is a relation, one product row is a tuple, name is an attribute, and the allowed values for price form its domain.
>
>
>
>

*(See Sections 2 and 3 of this week's Theory material.)*

**Q2.** What makes a candidate key different from a primary key? Can a table have more than one candidate key?


> [!NOTE]
> ***Your Answer***
>
> A candidate key is a minimal key that can uniquely identify a row. A primary key is the candidate key chosen as the main key for the table. Yes, a table can have more than one candidate key, but only one is chosen as the primary key.
>
>
>

*(See Section 6 of this week's Theory material.)*

**Q3.** Explain entity integrity in your own words. Why can't a primary key be NULL?


> [!NOTE]
> ***Your Answer***
>
> Entity integrity means that every table must have a primary key, and the primary key cannot be NULL. A primary key is used to identify each row, so if it were NULL, we could not reliably identify that row.
>
>
>
>

*(See Section 8.1 of this week's Theory material.)*

**Q4.** What happens when referential integrity is violated? Give a concrete TrailShop example — show the SQL statement and the expected error.

> [!NOTE]
> ***Your Answer***
>
> Referential integrity is violated when a foreign key refers to a value that does not exist in the referenced table. For example:

INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (120, 'Test Product', 50.00, 5, 999);

This should fail because category_id 999 does not exist in the categories table. The error would be a foreign key constraint violation.
>
>
>
>

*(See Section 8.2 of this week's Theory material.)*

**Q5.** Explain the difference between a surrogate key and a natural key. Give an example of each for a `books` table in a library database.

> [!NOTE]
> ***Your Answer***
>
>A surrogate key is an ID created by the database and has no real-world meaning. A natural key comes from real-world data and has a meaning. For example, book_id could be a surrogate key, while ISBN could be a natural key for a book.
>
>
>
>

*(See Section 6.8–6.9 of this week's Theory material.)*

**Q6.** What is a NULL value? Why is `WHERE price = NULL` wrong? What should you write instead?


> [!NOTE]
> ***Your Answer***
>
>NULL means that a value is unknown or not applicable. `WHERE price = NULL` is wrong because NULL is not compared using `=` in SQL. We should use `WHERE price IS NULL` instead.
>
>
>

*(See Section 7 of this week's Theory material.)*

**Q7.** What is a junction table? When is it needed? Give an example.

> [!NOTE]
> ***Your Answer***
>
> A junction table is used to connect two tables in a many-to-many relationship. For example, the order_items table can connect orders and products, because one order can contain many products and one product can appear in many orders.
>
>
>
>

*(See Section 12.3 of this week's Theory material.)*

**Q8.** Describe the three types of relationships (1:1, 1:N, M:N). For each, give one TrailShop example.

> [!NOTE]
> ***Your Answer***
>
>A 1:1 relationship means one row is related to one row, a 1:N relationship means one row can be related to many rows, and an M:N relationship means many rows can be related to many rows. In TrailShop, customers and orders are 1:N because one customer can have many orders. Orders and products are M:N because an order can contain many products and a product can be in many orders, using order_items.
>
>
>
>

*(See Section 12 of this week's Theory material.)*

**Q9.** What is the difference between `ON DELETE CASCADE` and `ON DELETE RESTRICT`? When would you use each?


> [!NOTE]
> ***Your Answer***
>
>ON DELETE CASCADE deletes the related rows when the referenced row is deleted. ON DELETE RESTRICT prevents the delete if related rows still exist. CASCADE can be used when dependent rows should also be deleted, while RESTRICT can be used when we want to protect related data.
>
>
>
>

*(See Section 10 of this week's Theory material.)*

**Q10.** Explain what "atomic entries" means in the context of relation properties. Give an example of a violation.

> [!NOTE]
> ***Your Answer***
>
>Atomic entries mean that each cell should contain one single value, not a list of values. For example, storing "Footwear, Hiking" in one categories cell is not atomic because it contains two values. These values should be stored separately.
>
>
>
>

*(See Section 5.3 of this week's Theory material.)*

### True/False

For each statement, write **True** or **False** and correct any false statements.

1. False. A candidate key is a minimal superkey, so a superkey is not always a candidate key.

2. True.

3. False. NULL = NULL evaluates to UNKNOWN in SQL. Use IS NULL to check for NULL.

4. False. A foreign key can be NULL if the column allows NULL values.

5. True.

6. False. The degree of a relation is the number of columns (attributes). The number of rows is the cardinality.
### Matching Exercise

Match each term (1–12) with its definition (A–L).

| # | Term |
|---|---|
| 1 | Superkey |
| 2 | Candidate key |
| 3 | Composite key |
| 4 | Foreign key |
| 5 | Alternate key |
| 6 | Surrogate key |
| 7 | Natural key |
| 8 | Orphan record |
| 9 | Domain |
| 10 | Junction table |
| 11 | Cardinality |
| 12 | COALESCE |
| Letter | Definition |
|---|---|
| A | The set of all permitted values for an attribute |
| B | A key composed of two or more attributes |
| C | A row whose FK references a non-existent PK — forbidden by referential integrity |
| D | An artificial key with no business meaning (e.g., auto-generated ID) |
| E | A candidate key not chosen as the primary key |
| F | Any set of attributes that uniquely identifies every tuple |
| G | A minimal superkey — no attribute can be removed without losing uniqueness |
| H | A column that references the primary key of another table |
| I | The number of tuples (rows) in a relation |
| J | A key drawn from real-world data with business meaning |
| K | A table implementing a many-to-many relationship |
| L | A SQL function that returns the first non-NULL argument |


> [!NOTE]
> ***Your Answers***
>
> > | # | Your Match |
> |---|---|
> | 1 | F |
> | 2 | G |
> | 3 | B |
> | 4 | H |
> | 5 | E |
> | 6 | D |
> | 7 | J |
> | 8 | C |
> | 9 | A |
> | 10 | K |
> | 11 | I |
> | 12 | L |

---

## Part 3: SQL Practice — Constraints in Action

These exercises test your understanding of constraints. You do NOT need to run these in PostgreSQL (but you may if you'd like to verify your answers).

### Exercise 3.1: Predict the Outcome

Given the following table definitions:

```sql
CREATE TABLE departments (
    dept_id   INTEGER      PRIMARY KEY,
    dept_name VARCHAR(50)  NOT NULL UNIQUE
);

CREATE TABLE employees (
    emp_id    INTEGER       PRIMARY KEY,
    name      VARCHAR(100)  NOT NULL,
    salary    NUMERIC(10,2) NOT NULL CHECK (salary >= 0),
    dept_id   INTEGER       NOT NULL REFERENCES departments(dept_id)
);
```

Assume these rows already exist:

```sql
INSERT INTO departments VALUES (1, 'Engineering');
INSERT INTO departments VALUES (2, 'Marketing');
INSERT INTO employees VALUES (100, 'Alice', 75000, 1);
INSERT INTO employees VALUES (101, 'Bob', 65000, 2);
```

For each statement below, predict: **SUCCESS** or **FAIL**? If fail, name the violated constraint.

```sql
-- 1
INSERT INTO employees VALUES (102, 'Carol', 70000, 1);

-- 2
INSERT INTO employees VALUES (103, 'Dan', -5000, 1);

-- 3
INSERT INTO employees VALUES (100, 'Eve', 80000, 2);

-- 4
INSERT INTO employees VALUES (104, 'Frank', 60000, 5);

-- 5
INSERT INTO departments VALUES (3, 'Engineering');

-- 6
INSERT INTO employees VALUES (105, NULL, 55000, 2);

-- 7
DELETE FROM departments WHERE dept_id = 1;

-- 8
INSERT INTO employees VALUES (106, 'Grace', 0, 2);


**Your Answer**

1. SUCCESS

2. FAIL — The salary is less than 0, so it violates the CHECK constraint.

3. FAIL — The emp_id 100 already exists, so it violates the PRIMARY KEY constraint.

4. FAIL — The dept_id 5 does not exist in the departments table, so it violates the FOREIGN KEY constraint.

5. FAIL — The department name 'Engineering' already exists, so it violates the UNIQUE constraint.

6. FAIL — The name is NULL, so it violates the NOT NULL constraint.

7. FAIL — Department 1 cannot be deleted because employees still reference it through the FOREIGN KEY.

8. SUCCESS

### Exercise 3.2: Write the Constraints

Given these business rules for a **bookstore database**, write the `CREATE TABLE` statements with appropriate constraints:

1. Every book has a unique ISBN (13 characters), a title (required), a price (must be positive), and a publication year.
2. Every author has an ID, a first name (required), and a last name (required).
3. A book can have multiple authors, and an author can write multiple books.
4. Every book belongs to exactly one genre. Genres have an ID and a unique name.
5. Publication year must be between 1450 and the current year.

*(Hint: you'll need at least 4 tables, including a junction table for the M:N relationship.)*

**Your SQL**

```sql
CREATE TABLE genres (
    genre_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    genre_name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE books (
    isbn CHAR(13) PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    price NUMERIC(10,2) NOT NULL CHECK (price > 0),
    publication_year INTEGER NOT NULL
        CHECK (
            publication_year BETWEEN 1450
            AND EXTRACT(YEAR FROM CURRENT_DATE)
        ),
    genre_id INTEGER NOT NULL
        REFERENCES genres(genre_id)
);

CREATE TABLE authors (
    author_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL
);

CREATE TABLE book_authors (
    isbn CHAR(13) NOT NULL
        REFERENCES books(isbn),
    author_id INTEGER NOT NULL
        REFERENCES authors(author_id),
    PRIMARY KEY (isbn, author_id)
);

## Part 4: Design Exercise — Library System

A small public library needs a database. Here is a description of their requirements:

> The library has a collection of **books**. Each book has an ISBN, a title, a publication year, and belongs to one genre (Fiction, Non-Fiction, Science, History, etc.). The library may own multiple **copies** of the same book — each copy has a unique barcode sticker.
>
> The library has registered **members**. Each member has a member number, name, email, and phone. Members can **borrow** copies. Each borrowing records which member borrowed which copy, the borrow date, the due date, and the return date (NULL if not yet returned).
>
> **Rules:**
> - A member can borrow at most 5 copies at any given time.
> - The due date is always 14 days after the borrow date.
> - A copy cannot be borrowed if it's currently not returned (return_date IS NULL).

### Your Tasks

1. **Identify the tables** you would need (list them with their columns).
2. **Identify the primary key** for each table. Are they surrogate or natural keys? Justify your choices.
3. **Identify all foreign keys** and the tables they reference.
4. **Identify any candidate keys** beyond the primary key (alternate keys).
5. **List the business rules** from the description and map each to a constraint type. Which rules cannot be enforced by simple constraints?


> [!NOTE]
> ***Your Answer***
>
>### 1. Tables and columns

**genres**
- genre_id
- genre_name

**books**
- isbn
- title
- publication_year
- genre_id

**copies**
- barcode
- isbn

**members**
- member_number
- name
- email
- phone

**borrowings**
- borrowing_id
- member_number
- barcode
- borrow_date
- due_date
- return_date

### 2. Primary keys

- genres.genre_id — surrogate key. It is a database-generated ID and does not have business meaning.
- books.isbn — natural key. ISBN comes from real-world book data and identifies the book.
- copies.barcode — natural key. The barcode identifies each physical copy.
- members.member_number — natural key. It is the number used to identify a library member.
- borrowings.borrowing_id — surrogate key. It is a database-generated ID for each borrowing record.

### 3. Foreign keys

- books.genre_id → genres.genre_id
- copies.isbn → books.isbn
- borrowings.member_number → members.member_number
- borrowings.barcode → copies.barcode

### 4. Candidate keys / alternate keys

- genres.genre_name can be an alternate key because each genre name should be unique.
- members.email can be an alternate key if the library requires every member to have a unique email address.

### 5. Business rules and constraints

| Business Rule | Constraint Type | Explanation |
|---|---|---|
| Every book belongs to one genre | NOT NULL + FOREIGN KEY | books.genre_id must refer to an existing genre. |
| Every copy belongs to a book | NOT NULL + FOREIGN KEY | copies.isbn must refer to an existing book. |
| Every copy has a unique barcode | PRIMARY KEY | copies.barcode identifies each physical copy. |
| A member can borrow at most 5 copies at a time | Trigger or application logic | This requires counting active borrowings, so a simple CHECK constraint is not enough. |
| Due date is 14 days after borrow date | CHECK | due_date must be borrow_date + 14 days. |
| A copy cannot have two active borrowings | UNIQUE partial index | Only one borrowing with return_date IS NULL is allowed for each barcode. |
| return_date can be NULL when the copy has not been returned | NULL allowed | NULL means the borrowing is still active. |
>
>
>
>
6. **Write the CREATE TABLE statements** for at least the `books`, `copies`, and `borrowings` tables with full constraints.

---
### 6. CREATE TABLE statements
CREATE TABLE genres (
    genre_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    genre_name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE books (
    isbn CHAR(13) PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    publication_year INTEGER NOT NULL
        CHECK (
            publication_year BETWEEN 1450
            AND EXTRACT(YEAR FROM CURRENT_DATE)
        ),
    genre_id INTEGER NOT NULL
        REFERENCES genres(genre_id)
);

CREATE TABLE copies (
    barcode VARCHAR(50) PRIMARY KEY,
    isbn CHAR(13) NOT NULL
        REFERENCES books(isbn)
);

CREATE TABLE members (
    member_number INTEGER PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    email VARCHAR(255) UNIQUE,
    phone VARCHAR(30)
);

CREATE TABLE borrowings (
    borrowing_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    member_number INTEGER NOT NULL
        REFERENCES members(member_number),
    barcode VARCHAR(50) NOT NULL
        REFERENCES copies(barcode),
    borrow_date DATE NOT NULL,
    due_date DATE NOT NULL
        CHECK (due_date = borrow_date + 14),
    return_date DATE
);

CREATE UNIQUE INDEX unique_active_borrowing_per_copy
ON borrowings(barcode)
WHERE return_date IS NULL;

## Submission Checklist

- [x] Task 1: Key identification answers (Part 1)
- [x] Task 2: Business rules table with 5 rules (Part 1)
- [x] Task 3: Integrity violation predictions with explanations (Part 1)
- [x] Task 4: Foreign key action analysis (Part 1)
- [x] Theory Review Questions answered (Part 2)
- [x] SQL Practice — constraint predictions and bookstore CREATE TABLE (Part 3)
- [x] Library System design exercise (Part 4)
