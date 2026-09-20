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
> *(product_id is the primary key. It is a good choice because it is unique for every product, never changes, and clearly identifies each row without relying on data that could change (like name or price))*
>
>
>
>


2. What is the primary key of the `categories` table?

> [!NOTE]
> ***Your Answer***
>
> *(category_id)*
>
>
>
>


3. What is the foreign key in the `products` table? What does it reference?

> [!NOTE]
> ***Your Answer***
>
> *(category_id is the foreign key. It references categories.category_id)*
>
>
>
>


4. Is `name` in `products` a candidate key? Under what assumption? What would make it unsuitable as a primary key?


> [!NOTE]
> ***Your Answer***
>
> *(It could be a candidate key only if all product names are guaranteed unique. It is unsuitable as a PK because names can change, duplicate names can happen, and searching/joining with text is slower than using a numeric ID. )*
>
>
>
>
5. Give an example of a **superkey** for the `products` table that is NOT a candidate key. Explain why it's not minimal.

> [!NOTE]
> ***Your Answer***
>
> *([product_id name] is a superkey but not a candidate key.It uniquely identifies rows,but it is not minimal product _id alone is enought removing name still keeps uniqueness.)*
>
>
>
>

6. Give an example of a **composite key** using a hypothetical `order_items` table. Explain why neither column alone would be sufficient.

> [!NOTE]
> ***Your Answer***
>
> *(Composite key: {order_id, product_id}. Neither column alone works — one order has many products, and one product appears in many orders. Only together do they uniquely identify one line item.)*
>
>
>
>

7. Is `email` in `customers` a candidate key? What makes it different from `customer_id` as a PK choice? *(See Section 6.9 on natural vs surrogate keys.)*

> [!NOTE]
> ***Your Answer***
>
> *(Yes, email is a candidate key (unique per customer). However, it is a natural key — it has business meaning and can change (people change emails). customer_id is a surrogate key — it has no business meaning, never changes, and is stable.)*
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
> *(List your 5 business rules with constraint types, table/column, and SQL syntax.)*
>  | Business Rule | Constraint Type | Table.Column | SQL |
> |---|---|---|---|
> | Every product must have a price greater than zero | CHECK | products.price | `CHECK (price > 0)` |
> | Product stock quantity cannot be negative | CHECK | products.stock_quantity | `CHECK (stock_quantity >= 0)` |
> | Every customer email address must be unique | UNIQUE | customers.email | `UNIQUE (email)` |
> | Category names cannot be empty (NULL) | NOT NULL | categories.category_name | `NOT NULL` |
> | Order total amount must be non-negative | CHECK | orders.total_amount | `CHECK (total_amount >= 0)` |
>

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
> *(For each statement A–H, write SUCCESS or FAIL and explain any violation.)*
> | Statement | Result | Explanation & Integrity Rule |
> |---|---|---|
> | **Statement A** | **FAIL** | Violates Entity Integrity / `NOT NULL` constraint: Primary key `category_id` cannot be NULL. |
> | **Statement B** | **SUCCESS** | Valid insert; all attributes adhere to constraints and category 2 exists in the categories table. |
> | **Statement C** | **FAIL** | Violates Domain Integrity / `CHECK` constraint: `price = -5.00` violates `CHECK (price > 0)`. |
> | **Statement D** | **FAIL** | Violates Entity Integrity / `PRIMARY KEY` constraint: `product_id = 103` already exists (duplicate key). |
> | **Statement E** | **FAIL** | Violates Referential Integrity / `FOREIGN KEY` constraint: `category_id = 10` does not exist in `categories`. |
> | **Statement F** | **FAIL** | Violates Domain Integrity / `NOT NULL` constraint: Product `name` cannot be NULL. |
> | **Statement G** | **FAIL** | Violates Domain Integrity / `CHECK` constraint: `stock_quantity = -3` violates `CHECK (stock_quantity >= 0)`. |
> | **Statement H** | **FAIL** | Violates Domain Integrity / `CHECK` constraint: `quantity = 0` violates `CHECK (quantity > 0)`. |
### Task 4: Foreign Key Actions

Consider the following scenario using the schema from Theory Section 9.8:

1. You want to delete category 2 ("Camping") from the `categories` table. Products 102 and 106 reference this category. What happens with:
   - `ON DELETE RESTRICT`? Delete blocked.Products still reference this category ,deletion is rejected.
   - `ON DELETE CASCADE`? Category and all its products are deleted.
   - `ON DELETE SET NULL`? (Assume `category_id` in `products` allows NULL for this question) Category deleted;products.category_id set to Null

2. Which foreign key action would you recommend for the TrailShop `products.category_id` → `categories.category_id` relationship? Justify your choice in 2–3 sentences.

> [!NOTE]
> ***Your Answer***
>
> *(Delete blocked. Products still reference this category; deletion is rejected.
> Catedory and all its products are deleted
> Category deleted; products.category_id set to NULL.)*
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
> *(Relation = table with rows & columns 
> Tuple = one row/record
> Attribute =  one column/field 
> Domain = allowed values for an attribute.)*
>
>
>
>

*(See Sections 2 and 3 of this week's Theory material.)*

**Q2.** What makes a candidate key different from a primary key? Can a table have more than one candidate key?


> [!NOTE]
> ***Your Answer***
>
> *(A candidate key can uniquely identify rows — there can be multiple. A primary key is the one chosen from candidates as the main identifier. Yes, a table can have multiple candidate key.)*
>
>
>
>

*(See Section 6 of this week's Theory material.)*

**Q3.** Explain entity integrity in your own words. Why can't a primary key be NULL?


> [!NOTE]
> ***Your Answer***
>
> *(Every primary key value must be unique and not NULL. If PK were NULL, the row could not be uniquely identified — it would break the foundation of the relational model.)*
>
>
>
>

*(See Section 8.1 of this week's Theory material.)*

**Q4.** What happens when referential integrity is violated? Give a concrete TrailShop example — show the SQL statement and the expected error.

> [!NOTE]
> ***Your Answer***
> INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (999, 'Trek Pole', 29.99, 10, 9999);
-- ERROR: insert or update on table "products" violates foreign key constraint "products_category_id_fkey"
-- DETAIL: Key (category_id)=(9999) is not present in table "categories".)*
>
>
>
>

*(See Section 8.2 of this week's Theory material.)*

**Q5.** Explain the difference between a surrogate key and a natural key. Give an example of each for a `books` table in a library database.

> [!NOTE]
> ***Your Answer***
>
> *(Surrogate = artificial ID with no business meaning (e.g., book_id)
> Natural = from real-world data with meaning (e.g., ISBN)*
>
>
>
>

*(See Section 6.8–6.9 of this week's Theory material.)*

**Q6.** What is a NULL value? Why is `WHERE price = NULL` wrong? What should you write instead?


> [!NOTE]
> ***Your Answer***
>
> *(NULL means "unknown or missing value". WHERE price = NULL is always unknown — use WHERE price IS NULL instead. NULL is not equal to anything, including itself.)*
>
>
>
>

*(See Section 7 of this week's Theory material.)*

**Q7.** What is a junction table? When is it needed? Give an example.

> [!NOTE]
> ***Your Answer***
>
> *( table connecting two others for a many-to-many relationship. Example: book_authors(book_id, author_id) links books and authors _one book has many authors,one author many books.)*
>
>
>
>

*(See Section 12.3 of this week's Theory material.)*

**Q8.** Describe the three types of relationships (1:1, 1:N, M:N). For each, give one TrailShop example.

> [!NOTE]
> ***Your Answer***
>
> *(1:1 - one customer = one loyalty account
> 1:N -  one category = many products
> M:N -one product = many orders; one order = many products)*
>
>
>
>

*(See Section 12 of this week's Theory material.)*

**Q9.** What is the difference between `ON DELETE CASCADE` and `ON DELETE RESTRICT`? When would you use each?


> [!NOTE]
> ***Your Answer***
>
> *(RESTRICT —  block deletion if referenced
> CASCADE — delete referencing rows too
> Use RESTRICT  when data should be preserved; use CASCADE when dependent data should be removed together.)*
>
>
>
>

*(See Section 10 of this week's Theory material.)*

**Q10.** Explain what "atomic entries" means in the context of relation properties. Give an example of a violation.

> [!NOTE]
> ***Your Answer***
>
> *Each cell holds one single value, not a list or combination. Violation: storing 'Red, Blue, Green' in one color column — should be separate rows or a separate table.)*
>
>
>
>

*(See Section 5.3 of this week's Theory material.)*

### True/False

For each statement, write **True** or **False** and correct any false statements.

1. False _ A superkey is always a candidate key.
2. True_A primary key can consist of more than one column.
3. False_NULL = NULL evaluates to TRUE in SQL.
4. False_ A foreign key must always be NOT NULL.
5. True_Referential integrity ensures that every FK value matches an existing PK value (or is NULL).
6. False_ The degree of a relation is the number of rows.

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
> | # | Your Match |
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
> | 10 |K | 
> | 11 |I |
> | 12 |L |
>

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
```

### Exercise 3.2: Write the Constraints
CREATE TABLE genres (
    genre_id INT PRIMARY KEY,
    genre_name VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE books (
    isbn CHAR(13) PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    price NUMERIC(10,2) NOT NULL CHECK (price > 0),
    publication_year INT CHECK (publication_year BETWEEN 1450 AND EXTRACT(YEAR FROM CURRENT_DATE)),
    genre_id INT NOT NULL REFERENCES genres(genre_id)
);

CREATE TABLE authors (
    author_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL
);

CREATE TABLE book_authors (
    isbn CHAR(13) REFERENCES books(isbn) ON DELETE CASCADE,
    author_id INT REFERENCES authors(author_id) ON DELETE CASCADE,
    PRIMARY KEY (isbn, author_id)
);

Given these business rules for a **bookstore database**, write the `CREATE TABLE` statements with appropriate constraints:

1. Every book has a unique ISBN (13 characters), a title (required), a price (must be positive), and a publication year.
2. Every author has an ID, a first name (required), and a last name (required).
3. A book can have multiple authors, and an author can write multiple books.
4. Every book belongs to exactly one genre. Genres have an ID and a unique name.
5. Publication year must be between 1450 and the current year.

*(Hint: you'll need at least 4 tables, including a junction table for the M:N relationship.)*

---

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
> *(1. Tables & Columns:
> books — isbn, title, pub_year, genre_id
> genres — genre_id, genre_name .)*
> copies — barcode, isbn, condition
> members — member_no, name, email, phone
> borrowings —borrowing_id, barcode, member_no, borrow_date, due_date, return_date
> 2. Primary Keys:
> books.isbn — natural key (real-world meaning)
> genres.genre_id — surrogate key (simple numeric ID)
> copies.barcode — natural key (unique sticker)
> members.member_no —natural/surrogate (assigned ID)
> borrowings.borrowing_id — surrogate key (auto-generated, never changes)
> 3. Foreign Keys:
> copies.isbn →books.isbn
> books.genre_id → genres.genre_id
> borrowings.barcode →copies.barcode
> borrowings.member_no → members.member_no
> 4. Candidate Keys:
> members.email is unique → alternate key
> books.isbn is PK, no others needed
> 5. Business Rules & Constraints:
> Email unique → UNIQUE
> Due date = borrow_date + 14 → CHECK or trigger
> Max 5 borrows per member → Cannot be done with simple constraints — needs trigger or application logic
> Cannot borrow if already out → Cannot be done with simple constraints — needs trigger or check

6. **Write the CREATE TABLE statements** for at least the `books`, `copies`, and `borrowings` tables with full constraints.
CREATE TABLE books (
    isbn CHAR(13) PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    publication_year INT CHECK (publication_year BETWEEN 1450 AND EXTRACT(YEAR FROM CURRENT_DATE)),
    genre_id INT NOT NULL REFERENCES genres(genre_id)
);

CREATE TABLE copies (
    barcode VARCHAR(50) PRIMARY KEY,
    isbn CHAR(13) NOT NULL REFERENCES books(isbn) ON DELETE CASCADE
);

CREATE TABLE borrowings (
    borrowing_id SERIAL PRIMARY KEY,
    member_id INT NOT NULL REFERENCES members(member_id),
    barcode VARCHAR(50) NOT NULL REFERENCES copies(barcode),
    borrow_date DATE NOT NULL DEFAULT CURRENT_DATE,
    due_date DATE NOT NULL CHECK (due_date >= borrow_date),
    return_date DATE CHECK (return_date >= borrow_date)
);

-- Partial unique index to enforce that an unreturned copy cannot be borrowed again
CREATE UNIQUE INDEX idx_active_borrowing_per_copy 
ON borrowings(barcode) 
WHERE return_date IS NULL;
---

## Submission Checklist

- [ ] Task 1: Key identification answers (Part 1)
- [ ] Task 2: Business rules table with 5 rules (Part 1)
- [ ] Task 3: Integrity violation predictions with explanations (Part 1)
- [ ] Task 4: Foreign key action analysis (Part 1)
- [ ] Theory Review Questions answered (Part 2)
- [ ] SQL Practice — constraint predictions and bookstore CREATE TABLE (Part 3)
- [ ] Library System design exercise (Part 4)
