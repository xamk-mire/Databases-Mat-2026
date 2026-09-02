# Week 40 — SQL Fundamentals: DDL, DML, and Building Your First Tables

## Chapter 5: "Breaking Ground"

The planning phase is over. You've spent the last few weeks understanding *why* databases exist, how the relational model works, how to draw ER diagrams, and how to transform those diagrams into logical table designs. The TrailShop founders are impressed — but they're getting impatient. "When do we actually *build* something?" they ask.

This week, you break ground. You'll take the logical design you created in Week 39 and turn it into a real, running PostgreSQL database. You'll create tables, define constraints, insert data, modify it, and delete it. By the end of this chapter, TrailShop will have a living database with real data inside it.

But first, you need to learn the language that makes all of this possible: **SQL**.

---

## Learning Objectives

By the end of this chapter you will be able to:

- Explain the history and purpose of SQL
- Distinguish between DDL, DML, DCL, and TCL sub-languages
- Create databases and tables with appropriate data types and constraints
- Use SERIAL and GENERATED ALWAYS AS IDENTITY for auto-incrementing keys
- Insert, update, and delete data using proper SQL syntax
- Alter and drop tables safely
- Identify and fix common SQL syntax mistakes

---

## 1. A Brief History of SQL

### 1.1 From IBM Research to Industry Standard

The story of SQL begins in the early 1970s at IBM's San Jose Research Laboratory in California. Two researchers — **Donald Chamberlin** and **Raymond Boyce** — were inspired by E.F. Codd's relational model (which you studied in Week 37) and set out to create a practical language that non-programmers could use to interact with relational databases.

Their first attempt, published in 1974, was called **SEQUEL** (Structured English Query Language). The name was later shortened to **SQL** due to a trademark conflict. Despite the name change, many people still pronounce SQL as "sequel" rather than spelling it out as "S-Q-L" — both pronunciations are acceptable.

> **Reference:** Watt & Eng, *Database Design — 2nd Edition*, Chapter 15, introduces SQL as "the standard language used to communicate with relational databases."

### 1.2 Early Implementations

IBM's first commercial SQL database was **System R** (a research prototype), followed by **SQL/DS** and **DB2** in the early 1980s. Meanwhile, a company called Relational Software Inc. (later renamed **Oracle Corporation**) beat IBM to market with the first commercially available SQL database in 1979.

By the mid-1980s, SQL had become the de facto standard for relational databases, with implementations from IBM, Oracle, Sybase, and others.

### 1.3 Standardization: ANSI and ISO

To prevent SQL from fragmenting into incompatible dialects, the American National Standards Institute (**ANSI**) published the first SQL standard in **1986**, followed by the International Organization for Standardization (**ISO**). Since then, the standard has been revised multiple times:

| Standard | Year | Key Additions |
|----------|------|---------------|
| SQL-86 | 1986 | First standard; basic queries |
| SQL-89 | 1989 | Integrity constraints |
| SQL-92 | 1992 | Major expansion: JOINs, CASE, subqueries, string functions |
| SQL:1999 | 1999 | Regular expressions, CTEs (WITH), triggers, OO features |
| SQL:2003 | 2003 | XML support, window functions, MERGE |
| SQL:2006 | 2006 | XQuery integration |
| SQL:2008 | 2008 | TRUNCATE, enhanced MERGE |
| SQL:2011 | 2011 | Temporal data (system-versioned tables) |
| SQL:2016 | 2016 | JSON support, row pattern matching |
| SQL:2023 | 2023 | Property Graph Queries, JSON enhancements |

### 1.4 SQL Today

Every major relational database — PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, SQLite — implements SQL, but each adds its own extensions and deviates slightly from the standard. In this course we use **PostgreSQL**, which is known for being one of the most standards-compliant open-source databases.

> **PostgreSQL documentation:** https://www.postgresql.org/docs/current/sql.html

### 1.5 Why Does History Matter?

Understanding SQL's history helps you make sense of the language's quirks. SQL was designed for *readability* — it intentionally looks like English sentences. That's why you write `SELECT name FROM products WHERE price > 50` rather than something like `products.filter(p => p.price > 50).map(p => p.name)`. The English-like syntax was a deliberate design choice from 1974 that persists to this day.

---

## 2. SQL Sub-Languages

SQL isn't one monolithic language — it's divided into sub-languages, each handling a different aspect of database work. Think of these as departments within the SQL organization:

### 2.1 DDL — Data Definition Language

DDL commands define the *structure* of your database. They create, modify, and remove database objects (tables, indexes, schemas, views).

| Command | Purpose |
|---------|---------|
| `CREATE TABLE` | Create a new table |
| `ALTER TABLE` | Modify table structure |
| `DROP TABLE` | Remove a table |
| `CREATE DATABASE` | Create a new database |
| `CREATE INDEX` | Create an index for performance |

> **Reference:** Watt & Eng, Chapter 15, covers DDL commands including CREATE TABLE syntax and constraints.

### 2.2 DML — Data Manipulation Language

DML commands work with the *data inside* tables. They add, read, modify, and remove rows.

| Command | Purpose |
|---------|---------|
| `INSERT INTO` | Add new rows |
| `SELECT` | Read/retrieve data |
| `UPDATE` | Modify existing rows |
| `DELETE` | Remove rows |

> **Reference:** Watt & Eng, Chapter 16, covers DML commands: INSERT, UPDATE, DELETE, and SELECT.

### 2.3 DCL — Data Control Language

DCL commands manage *permissions* — who can do what in the database.

| Command | Purpose |
|---------|---------|
| `GRANT` | Give permissions to a user/role |
| `REVOKE` | Remove permissions from a user/role |

Example:
```sql
GRANT SELECT, INSERT ON products TO shop_staff;
REVOKE DELETE ON products FROM shop_staff;
```

We won't use DCL extensively in this course, but it's important to know it exists — in production databases, security is critical.

### 2.4 TCL — Transaction Control Language

TCL commands manage *transactions* — groups of operations that must succeed or fail as a unit.

| Command | Purpose |
|---------|---------|
| `BEGIN` | Start a transaction |
| `COMMIT` | Save all changes in the transaction |
| `ROLLBACK` | Undo all changes in the transaction |
| `SAVEPOINT` | Set a point to roll back to |

Example:
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

If anything goes wrong between BEGIN and COMMIT, you can ROLLBACK to undo everything. We'll explore transactions in more depth later in the course.

---

## 3. CREATE DATABASE

Before creating tables, you need a database to put them in. In PostgreSQL, you create a database with:

```sql
CREATE DATABASE trailshop;
```

### 3.1 Specifying Encoding and Collation

In a real-world scenario, you'll want to specify the character encoding and collation (sorting rules):

```sql
CREATE DATABASE trailshop
    ENCODING = 'UTF8'
    LC_COLLATE = 'en_US.UTF-8'
    LC_CTYPE = 'en_US.UTF-8'
    TEMPLATE = template0;
```

- **ENCODING**: Determines which characters can be stored. UTF8 supports all Unicode characters — always use this unless you have a specific reason not to.
- **LC_COLLATE**: Determines sort order for strings. This affects ORDER BY on text columns.
- **LC_CTYPE**: Determines character classification (what's a letter, what's a digit).
- **TEMPLATE**: The template database to copy from. Use `template0` for a clean database with custom encoding.

> **PostgreSQL docs:** https://www.postgresql.org/docs/current/sql-createdatabase.html

### 3.2 Connecting to Your Database

After creating the database, connect to it:

```sql
\c trailshop
```

Or from the command line:
```bash
psql -d trailshop
```

---

## 4. CREATE TABLE — Comprehensive

This is the most important DDL command you'll learn this week. CREATE TABLE defines a new table's structure: its columns, their data types, and all constraints.

### 4.1 Full Syntax

Here's the general syntax for CREATE TABLE:

```sql
CREATE TABLE [IF NOT EXISTS] table_name (
    column_name  data_type  [column_constraints],
    column_name  data_type  [column_constraints],
    ...
    [table_constraints]
);
```

> **Reference:** Watt & Eng, Chapter 15, provides the syntax: "CREATE TABLE is followed by the table name, then in parentheses, column definitions separated by commas." The textbook emphasizes that each column definition includes at minimum a name and a data type.

### 4.2 Breaking Down Each Part

**Table name:** Use lowercase with underscores (snake_case). Use plural nouns for tables that hold collections: `products`, `customers`, `orders`.

**Column definition:** Each column needs:
1. A **name** (snake_case, descriptive)
2. A **data type** (what kind of data it holds)
3. Optional **constraints** (rules the data must follow)

**IF NOT EXISTS:** Prevents an error if the table already exists. Useful in scripts you might run multiple times:

```sql
CREATE TABLE IF NOT EXISTS categories (
    category_id  INTEGER  PRIMARY KEY,
    name         VARCHAR(100)  NOT NULL
);
```

### 4.3 Syntax Details and Punctuation

Pay careful attention to:
- Columns are separated by **commas**
- There is **no comma** after the last column/constraint
- The statement ends with a **semicolon**
- Parentheses must be **balanced**

---

## 5. Data Types — Quick Reference

You studied data types in Week 39. Here's a quick reference table for the types we use most often:

| Category | Type | Description | Example |
|----------|------|-------------|---------|
| Integer | `INTEGER` | Whole numbers (-2B to +2B) | `42` |
| Integer | `SMALLINT` | Small range (-32768 to 32767) | `5` |
| Integer | `BIGINT` | Very large whole numbers | `9000000000` |
| Decimal | `NUMERIC(p,s)` | Exact decimal (precision, scale) | `149.99` |
| Decimal | `DECIMAL(p,s)` | Synonym for NUMERIC | `149.99` |
| Text | `VARCHAR(n)` | Variable-length string (max n) | `'Hiking Boots'` |
| Text | `TEXT` | Unlimited-length string | Long descriptions |
| Text | `CHAR(n)` | Fixed-length string (padded) | `'FI'` |
| Boolean | `BOOLEAN` | TRUE / FALSE / NULL | `TRUE` |
| Date/Time | `DATE` | Date only | `'2026-08-19'` |
| Date/Time | `TIMESTAMP` | Date and time | `'2026-08-19 14:30:00'` |
| Date/Time | `TIMESTAMPTZ` | Timestamp with timezone | `'2026-08-19 14:30:00+03'` |

> **PostgreSQL docs:** https://www.postgresql.org/docs/current/datatype.html

### 5.1 Choosing the Right Type

Follow these guidelines:
- **Money:** Use `NUMERIC(10,2)` — never use floating point for money
- **IDs:** Use `INTEGER` (or `BIGINT` for very large tables)
- **Names/titles:** Use `VARCHAR(n)` with a reasonable limit
- **Descriptions:** Use `TEXT` (no practical length limit in PostgreSQL)
- **Yes/No flags:** Use `BOOLEAN`
- **Dates:** Use `DATE` for dates, `TIMESTAMPTZ` for timestamps (always prefer timezone-aware)

---

## 6. Constraints — Complete Reference

Constraints are rules enforced by the database engine. They prevent bad data from entering your tables. This is one of the most powerful features of relational databases — you define the rules once, and the database enforces them forever, regardless of which application is writing data.

> **Reference:** Watt & Eng, Chapter 15, discusses constraints: "Constraints limit the values that can be placed in a column or columns. They help to maintain data integrity."

### 6.1 NOT NULL

Prevents a column from containing NULL values. Use this for any column that *must always* have a value.

```sql
-- Column-level
name VARCHAR(100) NOT NULL
```

If you try to insert a row without providing a value for a NOT NULL column (and there's no DEFAULT), PostgreSQL will reject the insert with an error.

### 6.2 UNIQUE

Ensures all values in a column (or combination of columns) are distinct. NULLs are allowed — and multiple NULLs are considered distinct.

```sql
-- Column-level
email VARCHAR(255) UNIQUE

-- Table-level (composite unique)
UNIQUE (first_name, last_name, birth_date)
```

### 6.3 PRIMARY KEY

Combines NOT NULL + UNIQUE. Identifies each row uniquely. Every table should have exactly one primary key.

```sql
-- Column-level
product_id INTEGER PRIMARY KEY

-- Table-level (composite primary key)
PRIMARY KEY (order_id, product_id)
```

> **Reference:** Watt & Eng, Chapter 15: "A primary key is a constraint that uniquely identifies each row in a table."

### 6.4 FOREIGN KEY

Creates a link between two tables. The foreign key column must contain a value that exists in the referenced table's column (or be NULL, unless NOT NULL is also specified).

```sql
-- Column-level
category_id INTEGER REFERENCES categories(category_id)

-- Table-level (with ON DELETE/UPDATE actions)
FOREIGN KEY (category_id) REFERENCES categories(category_id)
    ON DELETE RESTRICT
    ON UPDATE CASCADE
```

**Referential actions (ON DELETE / ON UPDATE):**

| Action | Meaning |
|--------|---------|
| `RESTRICT` | Prevent deletion/update if referenced rows exist (default behavior) |
| `CASCADE` | Delete/update all referencing rows automatically |
| `SET NULL` | Set the FK column to NULL in referencing rows |
| `SET DEFAULT` | Set the FK column to its default value |
| `NO ACTION` | Similar to RESTRICT but checked at end of transaction |

> **Reference:** Watt & Eng, Chapter 15, covers foreign key constraints and referential integrity rules.

### 6.5 CHECK

Enforces a custom boolean condition on column values.

```sql
-- Column-level
price NUMERIC(10,2) CHECK (price > 0)
quantity INTEGER CHECK (quantity >= 0)

-- Table-level (can reference multiple columns)
CHECK (end_date > start_date)
```

### 6.6 DEFAULT

Sets a value automatically when no value is provided during INSERT.

```sql
created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
is_active BOOLEAN DEFAULT TRUE,
quantity INTEGER DEFAULT 0
```

DEFAULT is not technically a constraint (it doesn't restrict values), but it's specified in the same place and serves a similar purpose.

### 6.7 EXCLUDE (PostgreSQL-specific)

EXCLUDE constraints prevent overlapping or conflicting values. They're used with range types and GiST indexes. You won't need them for TrailShop, but it's good to know they exist.

```sql
-- Prevents overlapping time ranges for the same room
EXCLUDE USING gist (
    room_id WITH =,
    time_range WITH &&
);
```

### 6.8 Named Constraints

You can (and should) give your constraints explicit names. This makes error messages much more helpful:

```sql
CREATE TABLE products (
    product_id   INTEGER,
    name         VARCHAR(200) NOT NULL,
    price        NUMERIC(10,2),
    
    CONSTRAINT pk_products PRIMARY KEY (product_id),
    CONSTRAINT chk_products_price CHECK (price > 0),
    CONSTRAINT uq_products_name UNIQUE (name)
);
```

When a constraint is violated, PostgreSQL will report the constraint name in the error message. Compare:
- Named: `ERROR: violating constraint "chk_products_price"`
- Unnamed: `ERROR: new row violates check constraint "products_price_check"`

### 6.9 Column-Level vs Table-Level

**Column-level constraints** are declared immediately after the column's data type:
```sql
price NUMERIC(10,2) NOT NULL CHECK (price > 0)
```

**Table-level constraints** are declared after all columns, separated by commas:
```sql
CREATE TABLE order_items (
    order_id    INTEGER NOT NULL,
    product_id  INTEGER NOT NULL,
    quantity    INTEGER NOT NULL,
    
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    CHECK (quantity > 0)
);
```

**When must you use table-level?**
- Composite PRIMARY KEY (spanning multiple columns)
- Composite FOREIGN KEY
- CHECK constraints referencing multiple columns
- Named constraints (though these can technically be column-level too)

---

## 7. SERIAL vs GENERATED ALWAYS AS IDENTITY

Auto-incrementing primary keys are extremely common. PostgreSQL offers two approaches:

### 7.1 SERIAL (Legacy Approach)

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL
);
```

`SERIAL` is actually shorthand that:
1. Creates the column as `INTEGER NOT NULL`
2. Creates a sequence object (e.g., `products_product_id_seq`)
3. Sets the default value to `nextval('products_product_id_seq')`

Variants: `SMALLSERIAL` (SMALLINT), `BIGSERIAL` (BIGINT).

### 7.2 GENERATED ALWAYS AS IDENTITY (Modern Approach)

```sql
CREATE TABLE products (
    product_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(200) NOT NULL
);
```

This is the SQL standard way (SQL:2003+) to create auto-incrementing columns.

### 7.3 Comparison

| Feature | SERIAL | GENERATED ALWAYS AS IDENTITY |
|---------|--------|------------------------------|
| SQL Standard | No (PostgreSQL-specific) | Yes (SQL:2003) |
| Can be overridden | Yes (easy to accidentally insert manual values) | No (unless you use OVERRIDING SYSTEM VALUE) |
| Sequence ownership | Manual | Automatic |
| Portability | PostgreSQL only | Works in PostgreSQL, Oracle, DB2 |
| Recommendation | Legacy code | New projects |

### 7.4 Which Should You Use?

For new projects, prefer `GENERATED ALWAYS AS IDENTITY`. It's the standard, it's safer (prevents accidental manual inserts), and it's clearer about intent. However, `SERIAL` is so widespread in PostgreSQL tutorials and existing code that you must understand it.

In this course, we'll use `SERIAL` in most examples because:
- It's shorter and easier to read when learning
- Most online PostgreSQL tutorials use it
- The TrailShop sample scripts use it

> **PostgreSQL docs:** https://www.postgresql.org/docs/current/sql-createtable.html (see "Identity Columns")

---

## 8. Building the TrailShop Tables

Now let's put everything together and build the TrailShop database. Remember the five tables from your Week 39 logical design:

1. **categories** — product categories
2. **products** — the items TrailShop sells
3. **customers** — people who buy from TrailShop
4. **orders** — purchase transactions
5. **order_items** — individual items within each order (junction table)

### 8.1 Why Order Matters

You must create tables in **dependency order** — a table that references another table (via foreign key) must be created *after* the table it references.

The dependency chain for TrailShop:
```
categories  →  products  →  order_items  ←  orders  ←  customers
```

So the creation order is:
1. `categories` (no dependencies)
2. `customers` (no dependencies)
3. `products` (depends on categories)
4. `orders` (depends on customers)
5. `order_items` (depends on both orders and products)

### 8.2 Creating the Categories Table

```sql
CREATE TABLE categories (
    category_id  SERIAL PRIMARY KEY,
    name         VARCHAR(100) NOT NULL UNIQUE,
    description  TEXT
);
```

This is the simplest table — just an ID, a name (required and unique), and an optional description.

### 8.3 Creating the Customers Table

```sql
CREATE TABLE customers (
    customer_id  SERIAL PRIMARY KEY,
    first_name   VARCHAR(100) NOT NULL,
    last_name    VARCHAR(100) NOT NULL,
    email        VARCHAR(255) NOT NULL UNIQUE,
    created_at   TIMESTAMPTZ  NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

Key decisions:
- `email` is UNIQUE because each customer must have a distinct email address
- `created_at` uses a DEFAULT so we don't have to provide it on every INSERT
- We use `TIMESTAMPTZ` (timezone-aware) rather than `TIMESTAMP`

### 8.4 Creating the Products Table

```sql
CREATE TABLE products (
    product_id   SERIAL PRIMARY KEY,
    name         VARCHAR(200) NOT NULL,
    description  TEXT,
    price        NUMERIC(10,2) NOT NULL CHECK (price > 0),
    stock        INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
    category_id  INTEGER NOT NULL REFERENCES categories(category_id),
    created_at   TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

Key decisions:
- `price` has a CHECK constraint ensuring it's always positive — you can't have a product with a negative price
- `stock` defaults to 0 and cannot go negative
- `category_id` is a FOREIGN KEY referencing `categories` — every product must belong to a valid category
- The FK is NOT NULL, meaning every product *must* have a category

### 8.5 Creating the Orders Table

```sql
CREATE TABLE orders (
    order_id     SERIAL PRIMARY KEY,
    customer_id  INTEGER NOT NULL REFERENCES customers(customer_id),
    order_date   TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status       VARCHAR(20) NOT NULL DEFAULT 'pending'
                 CHECK (status IN ('pending', 'shipped', 'delivered', 'cancelled'))
);
```

Key decisions:
- `customer_id` links each order to a customer
- `status` uses a CHECK constraint to limit values to a predefined set — this is like an enum
- `order_date` defaults to the current timestamp

### 8.6 Creating the Order Items Table (Junction Table)

```sql
CREATE TABLE order_items (
    order_item_id  SERIAL PRIMARY KEY,
    order_id       INTEGER NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id     INTEGER NOT NULL REFERENCES products(product_id),
    quantity       INTEGER NOT NULL CHECK (quantity > 0),
    unit_price     NUMERIC(10,2) NOT NULL CHECK (unit_price > 0)
);
```

Key decisions:
- `ON DELETE CASCADE` on the order FK means: if an order is deleted, its items are automatically deleted too
- `unit_price` stores the price *at the time of purchase* — if the product's price changes later, the order record stays accurate
- `quantity` must be at least 1

### 8.7 The Complete Script

Here's everything together in a single script you can run:

```sql
-- TrailShop Database Creation Script
-- Run this in a fresh database

CREATE TABLE categories (
    category_id  SERIAL PRIMARY KEY,
    name         VARCHAR(100) NOT NULL UNIQUE,
    description  TEXT
);

CREATE TABLE customers (
    customer_id  SERIAL PRIMARY KEY,
    first_name   VARCHAR(100) NOT NULL,
    last_name    VARCHAR(100) NOT NULL,
    email        VARCHAR(255) NOT NULL UNIQUE,
    created_at   TIMESTAMPTZ  NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    product_id   SERIAL PRIMARY KEY,
    name         VARCHAR(200) NOT NULL,
    description  TEXT,
    price        NUMERIC(10,2) NOT NULL CHECK (price > 0),
    stock        INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
    category_id  INTEGER NOT NULL REFERENCES categories(category_id),
    created_at   TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    order_id     SERIAL PRIMARY KEY,
    customer_id  INTEGER NOT NULL REFERENCES customers(customer_id),
    order_date   TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status       VARCHAR(20) NOT NULL DEFAULT 'pending'
                 CHECK (status IN ('pending', 'shipped', 'delivered', 'cancelled'))
);

CREATE TABLE order_items (
    order_item_id  SERIAL PRIMARY KEY,
    order_id       INTEGER NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id     INTEGER NOT NULL REFERENCES products(product_id),
    quantity       INTEGER NOT NULL CHECK (quantity > 0),
    unit_price     NUMERIC(10,2) NOT NULL CHECK (unit_price > 0)
);
```

---

## 9. INSERT INTO — Comprehensive

Now that you have tables, let's fill them with data.

> **Reference:** Watt & Eng, Chapter 16, covers the INSERT statement: "INSERT is used to add rows to an existing table."

### 9.1 Basic Syntax: Single Row

```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```

The column list specifies *which* columns you're providing values for. Any columns not listed will receive their DEFAULT value (or NULL if no default is defined).

### 9.2 Inserting into TrailShop

```sql
INSERT INTO categories (name, description)
VALUES ('Footwear', 'Hiking boots, trail runners, and sandals');
```

Notice:
- We didn't include `category_id` — SERIAL handles that automatically
- String values are enclosed in **single quotes** (not double quotes!)
- The column order in the INSERT matches the value order

### 9.3 Multiple Rows in One Statement

```sql
INSERT INTO categories (name, description) VALUES
    ('Footwear', 'Hiking boots, trail runners, and sandals'),
    ('Backpacks', 'Day packs, overnight packs, and expedition packs'),
    ('Tents', 'One-person to family-size tents'),
    ('Clothing', 'Outdoor clothing for all seasons'),
    ('Accessories', 'Water bottles, headlamps, trekking poles');
```

This is more efficient than five separate INSERT statements because it requires only one round-trip to the database.

### 9.4 INSERT with RETURNING

PostgreSQL's `RETURNING` clause lets you see what was actually inserted (including auto-generated values):

```sql
INSERT INTO customers (first_name, last_name, email)
VALUES ('Anna', 'Virtanen', 'anna.v@email.com')
RETURNING customer_id, created_at;
```

Result:
```
 customer_id |          created_at
-------------+-------------------------------
           1 | 2026-08-19 14:30:00.123456+03
```

This is incredibly useful — you get the auto-generated ID back immediately without needing a separate query.

### 9.5 INSERT with DEFAULT Values

If you want a column to use its default explicitly:

```sql
INSERT INTO products (name, price, stock, category_id)
VALUES ('TrailMaster X4', 149.99, DEFAULT, 1);
```

Here, `stock` will receive the DEFAULT value of `0`.

You can also insert an entire row of defaults (rarely useful, but valid syntax):
```sql
INSERT INTO some_table DEFAULT VALUES;
```

### 9.6 INSERT from SELECT

You can insert data from another table or query:

```sql
INSERT INTO archived_orders (order_id, customer_id, order_date, status)
SELECT order_id, customer_id, order_date, status
FROM orders
WHERE status = 'delivered' AND order_date < '2026-01-01';
```

This is powerful for data migration, archiving, and batch operations.

### 9.7 Sample Data for TrailShop

Here's a complete set of sample data:

```sql
-- Categories
INSERT INTO categories (name, description) VALUES
    ('Footwear', 'Hiking boots, trail runners, and sandals'),
    ('Backpacks', 'Day packs, overnight packs, and expedition packs'),
    ('Tents', 'One-person to family-size tents'),
    ('Clothing', 'Outdoor clothing for all seasons'),
    ('Accessories', 'Water bottles, headlamps, trekking poles');

-- Customers
INSERT INTO customers (first_name, last_name, email) VALUES
    ('Anna', 'Virtanen', 'anna.v@email.com'),
    ('Mikko', 'Korhonen', 'mikko.k@email.com'),
    ('Sara', 'Mäkinen', 'sara.m@email.com'),
    ('Juha', 'Nieminen', 'juha.n@email.com'),
    ('Laura', 'Hämäläinen', 'laura.h@email.com');

-- Products
INSERT INTO products (name, description, price, stock, category_id) VALUES
    ('TrailMaster X4', 'Professional hiking boot with Gore-Tex lining', 149.99, 25, 1),
    ('LiteStep Pro', 'Lightweight trail runner for day hikes', 89.99, 40, 1),
    ('Summit 45L', 'Multi-day hiking backpack with rain cover', 199.99, 15, 2),
    ('DayTripper 20L', 'Compact day pack with hydration sleeve', 59.99, 50, 2),
    ('CloudNest 2P', 'Two-person ultralight tent', 349.99, 10, 3),
    ('StormShield 4P', 'Four-season family tent', 499.99, 5, 3),
    ('ThermoLayer Jacket', 'Insulated mid-layer for cold weather', 129.99, 30, 4),
    ('RainGuard Pro', 'Waterproof breathable rain jacket', 179.99, 20, 4),
    ('HydroFlask 1L', 'Insulated stainless steel water bottle', 34.99, 100, 5),
    ('LumaBeam 800', 'Rechargeable headlamp, 800 lumens', 44.99, 60, 5);

-- Orders
INSERT INTO orders (customer_id, status) VALUES
    (1, 'delivered'),
    (2, 'shipped'),
    (1, 'pending'),
    (3, 'delivered'),
    (4, 'pending');

-- Order Items
INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES
    (1, 1, 1, 149.99),
    (1, 9, 2, 34.99),
    (2, 3, 1, 199.99),
    (2, 7, 1, 129.99),
    (3, 5, 1, 349.99),
    (4, 2, 1, 89.99),
    (4, 4, 1, 59.99),
    (4, 10, 1, 44.99),
    (5, 6, 1, 499.99),
    (5, 8, 1, 179.99);
```

---

## 10. UPDATE — Comprehensive

UPDATE modifies existing rows in a table.

> **Reference:** Watt & Eng, Chapter 16, covers the UPDATE statement: "UPDATE is used to modify values in existing rows."

### 10.1 Basic Syntax

```sql
UPDATE table_name
SET column1 = new_value1,
    column2 = new_value2
WHERE condition;
```

### 10.2 Simple UPDATE

Change a product's price:

```sql
UPDATE products
SET price = 159.99
WHERE product_id = 1;
```

### 10.3 UPDATE with Expressions

You can use expressions in SET:

```sql
-- Increase all prices by 10%
UPDATE products
SET price = price * 1.10;

-- Decrease stock by 1 for a specific product
UPDATE products
SET stock = stock - 1
WHERE product_id = 3;
```

### 10.4 UPDATE Multiple Columns

```sql
UPDATE customers
SET first_name = 'Anneli',
    email = 'anneli.v@email.com'
WHERE customer_id = 1;
```

### 10.5 UPDATE with Subquery (Brief)

You can use a subquery to determine the new value:

```sql
UPDATE products
SET price = (SELECT AVG(price) FROM products WHERE category_id = 1)
WHERE product_id = 2;
```

This sets product 2's price to the average price of category 1. We'll cover subqueries in more detail in later weeks.

### 10.6 UPDATE with RETURNING

Just like INSERT, UPDATE supports RETURNING:

```sql
UPDATE products
SET price = 169.99
WHERE product_id = 1
RETURNING name, price;
```

### 10.7 The Critical Mistake: Missing WHERE

**⚠️ DANGER:** An UPDATE without a WHERE clause updates *every row in the table*:

```sql
-- THIS UPDATES ALL PRODUCTS!
UPDATE products
SET price = 9.99;
```

This is one of the most common and devastating mistakes beginners make. Always double-check your WHERE clause before running an UPDATE. In production, you'd typically:
1. Run a SELECT with the same WHERE first to verify which rows will be affected
2. Use a transaction (BEGIN/COMMIT) so you can ROLLBACK if something goes wrong

---

## 11. DELETE — Comprehensive

DELETE removes rows from a table.

> **Reference:** Watt & Eng, Chapter 16, covers the DELETE statement.

### 11.1 Basic Syntax

```sql
DELETE FROM table_name
WHERE condition;
```

### 11.2 Simple DELETE

```sql
DELETE FROM customers
WHERE customer_id = 5;
```

### 11.3 DELETE with Complex Conditions

```sql
DELETE FROM products
WHERE stock = 0 AND created_at < '2025-01-01';
```

### 11.4 The Critical Mistake: Missing WHERE (Again)

```sql
-- THIS DELETES ALL ROWS!
DELETE FROM products;
```

The same warning applies: always verify your WHERE clause. Unlike DROP TABLE, DELETE without WHERE removes all *data* but keeps the table structure.

### 11.5 Cascading Deletes

If you defined a foreign key with `ON DELETE CASCADE`, deleting a parent row automatically deletes related child rows:

```sql
-- This also deletes all order_items for order 1
-- (because order_items.order_id has ON DELETE CASCADE)
DELETE FROM orders WHERE order_id = 1;
```

Without CASCADE, attempting to delete a row that's referenced by another table will fail:

```sql
-- ERROR: violates foreign key constraint
-- (because products.category_id references categories)
DELETE FROM categories WHERE category_id = 1;
```

### 11.6 TRUNCATE vs DELETE

| Feature | DELETE | TRUNCATE |
|---------|--------|----------|
| Removes | Specific rows (with WHERE) or all rows | All rows only |
| Speed | Slower (row by row) | Very fast |
| Triggers | Fires row-level triggers | Does not fire row triggers |
| MVCC | Creates dead tuples | Instantly reclaims space |
| Rollback | Can be rolled back | Can be rolled back (in PostgreSQL) |
| Reset sequences | No | Yes (with RESTART IDENTITY) |

```sql
TRUNCATE TABLE order_items;
TRUNCATE TABLE order_items RESTART IDENTITY;  -- also resets serial
TRUNCATE TABLE orders CASCADE;  -- also truncates referencing tables
```

Use TRUNCATE when you need to empty a table quickly (e.g., during testing). Use DELETE when you need to remove specific rows or when triggers must fire.

---

## 12. Basic SELECT — Verifying Your Work

We'll cover SELECT comprehensively in Week 41. For now, here's just enough to verify your INSERT, UPDATE, and DELETE operations:

### 12.1 See All Data in a Table

```sql
SELECT * FROM categories;
SELECT * FROM products;
SELECT * FROM customers;
```

### 12.2 Select Specific Columns

```sql
SELECT name, price FROM products;
```

### 12.3 Filter Rows

```sql
SELECT name, price FROM products WHERE category_id = 1;
```

### 12.4 Count Rows

```sql
SELECT COUNT(*) FROM products;
SELECT COUNT(*) FROM order_items WHERE order_id = 1;
```

Use these queries to confirm your DML operations worked correctly.

---

## 13. ALTER TABLE

After creating a table, you'll often need to modify its structure. ALTER TABLE lets you add columns, remove columns, add constraints, and more — all without losing existing data.

### 13.1 ADD COLUMN

```sql
ALTER TABLE products
ADD COLUMN weight_kg NUMERIC(5,2);
```

The new column will be NULL for all existing rows (unless you specify a DEFAULT).

```sql
ALTER TABLE products
ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT TRUE;
```

### 13.2 DROP COLUMN

```sql
ALTER TABLE products
DROP COLUMN weight_kg;
```

**Warning:** This permanently removes the column and all its data. PostgreSQL doesn't ask for confirmation.

If other objects depend on the column (views, constraints):
```sql
ALTER TABLE products
DROP COLUMN weight_kg CASCADE;
```

### 13.3 RENAME COLUMN

```sql
ALTER TABLE products
RENAME COLUMN name TO product_name;
```

### 13.4 Change Data Type

```sql
ALTER TABLE products
ALTER COLUMN description TYPE VARCHAR(500);
```

This may fail if existing data can't be converted to the new type.

### 13.5 Set or Drop DEFAULT

```sql
-- Set a default
ALTER TABLE products
ALTER COLUMN stock SET DEFAULT 0;

-- Remove a default
ALTER TABLE products
ALTER COLUMN stock DROP DEFAULT;
```

### 13.6 Set or Drop NOT NULL

```sql
-- Add NOT NULL
ALTER TABLE products
ALTER COLUMN description SET NOT NULL;

-- Remove NOT NULL
ALTER TABLE products
ALTER COLUMN description DROP NOT NULL;
```

### 13.7 ADD CONSTRAINT

```sql
ALTER TABLE products
ADD CONSTRAINT chk_products_name_length CHECK (LENGTH(name) >= 2);
```

### 13.8 DROP CONSTRAINT

```sql
ALTER TABLE products
DROP CONSTRAINT chk_products_name_length;
```

### 13.9 RENAME TABLE

```sql
ALTER TABLE products
RENAME TO inventory;
```

---

## 14. DROP TABLE

DROP TABLE permanently removes a table and all its data.

### 14.1 Basic Syntax

```sql
DROP TABLE table_name;
```

### 14.2 IF EXISTS

Avoids an error if the table doesn't exist:

```sql
DROP TABLE IF EXISTS temp_data;
```

### 14.3 Order Matters

Just as you must CREATE tables in dependency order, you must DROP them in *reverse* dependency order. You can't drop a table that's still referenced by foreign keys in other tables.

For TrailShop, the drop order is:
```sql
DROP TABLE IF EXISTS order_items;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS categories;
```

### 14.4 CASCADE

If you want to force-drop a table regardless of dependencies:

```sql
DROP TABLE categories CASCADE;
```

This will also remove all foreign key constraints in other tables that reference `categories`. **Use with extreme caution** — it won't delete rows in referencing tables, but it removes the constraint that was protecting referential integrity.

---

## 15. Common Beginner Mistakes

Here are the most common mistakes students make when writing SQL for the first time:

| # | Mistake | Symptom | Fix |
|---|---------|---------|-----|
| 1 | Missing semicolon | Statement doesn't execute; prompt changes to `->` | Add `;` at the end |
| 2 | Using double quotes for strings | Error: column "Footwear" does not exist | Use single quotes: `'Footwear'` |
| 3 | Missing comma between columns | Syntax error near column name | Check all column separators |
| 4 | Extra comma after last column | Syntax error near `)` | Remove trailing comma |
| 5 | Wrong creation order | Error: relation "categories" does not exist | Create referenced tables first |
| 6 | UPDATE/DELETE without WHERE | All rows affected | Always verify WHERE; use transactions |
| 7 | Inserting into wrong column order | Data in wrong columns or type errors | Always list columns explicitly |
| 8 | Using reserved words as names | Syntax error | Rename, or use double quotes (not recommended) |
| 9 | NUMERIC precision too small | Error or silent truncation | Plan for growth: `NUMERIC(10,2)` not `NUMERIC(4,2)` |
| 10 | Forgetting NOT NULL on FK columns | Orphan rows with NULL foreign keys | Add NOT NULL unless NULL is intentionally allowed |

### 15.1 Mistake #1: The Semicolon

In psql (the PostgreSQL command-line tool), if you forget the semicolon, the prompt changes from `trailshop=#` to `trailshop-#`, indicating it's waiting for more input. Just type `;` and press Enter.

### 15.2 Mistake #2: Quote Types

In SQL:
- **Single quotes** (`'...'`) delimit **string values**: `'Footwear'`, `'anna@email.com'`
- **Double quotes** (`"..."`) delimit **identifiers** (table/column names): `"My Table"`, `"order"` (when using reserved words)

```sql
-- WRONG: PostgreSQL thinks Footwear is a column name
INSERT INTO categories (name) VALUES ("Footwear");

-- CORRECT: Footwear is a string value
INSERT INTO categories (name) VALUES ('Footwear');
```

### 15.3 Mistake #6: The Catastrophic Missing WHERE

This deserves extra emphasis. Before running any UPDATE or DELETE:

1. **Write the WHERE clause first**
2. **Run a SELECT** with the same condition to see what will be affected
3. **Use a transaction** for safety:

```sql
BEGIN;
DELETE FROM products WHERE product_id = 99;
-- Check the result
SELECT COUNT(*) FROM products;
-- If something looks wrong:
ROLLBACK;
-- If everything is fine:
COMMIT;
```

---

## 16. Key Terms

| Term | Definition |
|------|-----------|
| SQL | Structured Query Language; the standard language for relational databases |
| DDL | Data Definition Language — CREATE, ALTER, DROP |
| DML | Data Manipulation Language — INSERT, SELECT, UPDATE, DELETE |
| DCL | Data Control Language — GRANT, REVOKE |
| TCL | Transaction Control Language — BEGIN, COMMIT, ROLLBACK |
| Constraint | A rule enforced by the database to maintain data integrity |
| Primary Key | A column (or set of columns) that uniquely identifies each row |
| Foreign Key | A column whose values must match values in another table's column |
| SERIAL | PostgreSQL shorthand for an auto-incrementing integer column |
| CASCADE | An option that propagates an operation (delete/drop) to related objects |
| RETURNING | A PostgreSQL clause that returns values from INSERT/UPDATE/DELETE |
| TRUNCATE | A fast operation that removes all rows from a table |

---

## 17. Reading

### Required Reading

- Watt & Eng, *Database Design — 2nd Edition*, **Chapter 15** (SQL: DDL, CREATE TABLE, constraints)
- Watt & Eng, *Database Design — 2nd Edition*, **Chapter 16** (SQL: DML — INSERT, UPDATE, DELETE, basic SELECT)

### Further Reading

- PostgreSQL Documentation — CREATE TABLE: https://www.postgresql.org/docs/current/sql-createtable.html
- PostgreSQL Documentation — INSERT: https://www.postgresql.org/docs/current/sql-insert.html
- PostgreSQL Documentation — UPDATE: https://www.postgresql.org/docs/current/sql-update.html
- PostgreSQL Documentation — DELETE: https://www.postgresql.org/docs/current/sql-delete.html
- PostgreSQL Documentation — Data Types: https://www.postgresql.org/docs/current/datatype.html
- PostgreSQL Wiki — Don't Do This: https://wiki.postgresql.org/wiki/Don%27t_Do_This

---

## 18. Summary

This week you transformed your logical design into a real, running database. You learned that SQL is divided into sub-languages (DDL for structure, DML for data, DCL for permissions, TCL for transactions), and you practiced the core commands:

- **CREATE TABLE** with data types and constraints
- **INSERT INTO** for adding data (single row, multiple rows, with RETURNING)
- **UPDATE** for modifying data (always with WHERE!)
- **DELETE** for removing data (always with WHERE!)
- **ALTER TABLE** for changing structure after the fact
- **DROP TABLE** for removing tables entirely

The TrailShop database now exists and contains real data. The founders can see their products, customers, and orders in a structured, consistent format. But they're already asking questions: "Which products are selling best?" "Who are our top customers?" "What's the total revenue by category?"

Next week, you'll learn how to answer those questions — with the full power of SELECT, WHERE, JOINs, and aggregate functions. The real fun begins.
