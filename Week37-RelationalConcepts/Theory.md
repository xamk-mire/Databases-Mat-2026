# Week 37 — The Relational Data Model: Theory

## Chapter 2: "Laying the Foundation"

You've convinced the TrailShop founders that spreadsheets won't scale. They're on board with using a database — but before you start creating tables, you need to understand the *rules of the game*. Relational databases aren't just "spreadsheets with better software." They have a formal mathematical foundation that guarantees data stays organized, consistent, and trustworthy.

The founders have handed you a spreadsheet export of their current product catalog: names, categories, prices, stock quantities. Your job this week is to figure out the right structure — not by guessing, but by applying the principles of the relational model.

This is one of the most important weeks of the entire course. The concepts you learn here — relations, keys, integrity rules, constraints — will appear in every single topic from here on. Take your time, read carefully, and make sure you can explain each concept in your own words before moving to the exercises.

---

## Learning Objectives

By the end of this chapter you will be able to:

- Explain the historical origins and mathematical foundations of the relational model
- Define and use the formal terminology: relation, tuple, attribute, domain, degree, cardinality
- Identify and distinguish between all types of keys (superkey, candidate, primary, composite, foreign, alternate, secondary, surrogate, natural)
- Explain null values and their behavior in SQL operations
- Apply entity integrity and referential integrity rules
- Translate business rules into PostgreSQL constraints
- Describe the different types of relationships (1:1, 1:N, M:N)
- Write CREATE TABLE statements with appropriate constraints

---

## 1. The Relational Model — Origins and Foundations

### 1.1 Codd's Revolution

In 1970, **Edgar F. Codd**, a British computer scientist working at IBM's San Jose Research Laboratory, published the paper "A Relational Model of Data for Large Shared Data Banks" in the journal *Communications of the ACM*. This paper is widely regarded as one of the most influential in the history of computer science.

At the time, databases used hierarchical and network models — navigating data required following physical pointers through complex structures. Programmers had to know *how* data was stored to retrieve it. Codd's insight was radical: separate the *logical* representation of data from its *physical* storage. Let users describe *what* they want, and let the system figure out *how* to get it.

As discussed in *Database Design* (2nd Ed., Ch. 7), the relational model is based on the mathematical concept of a **relation**, which is essentially a set of tuples. This mathematical foundation is what gives the model its power — operations on relations are well-defined, predictable, and provably correct.

### 1.2 Codd's 12 Rules

In 1985, Codd published 12 rules (actually 13, numbered 0–12) that a system must satisfy to be considered "fully relational." While no commercial DBMS satisfies all 13 rules perfectly, they set the standard. Key rules include:

- **Rule 0 (Foundation Rule):** A relational DBMS must manage data entirely through its relational capabilities.
- **Rule 1 (Information Rule):** All information in the database is represented explicitly as values in tables.
- **Rule 2 (Guaranteed Access):** Every value is accessible by a combination of table name, column name, and primary key value.
- **Rule 3 (Systematic Treatment of NULL):** NULL is supported for representing missing or inapplicable data, distinct from any actual value.
- **Rule 6 (Comprehensive Data Sublanguage):** The system must support a language (SQL) that handles data definition, manipulation, integrity constraints, and authorization.
- **Rule 12 (Nonsubversion):** If the system provides a low-level interface, it must not bypass the relational security or integrity constraints.

You don't need to memorize all 12 rules, but understanding their spirit helps: the relational model aims for *simplicity*, *consistency*, and *data independence*.

### 1.3 Mathematical Foundations

The relational model is grounded in two branches of mathematics:

**Set theory:** A relation is a subset of the Cartesian product of its domains. In plain language: a table is a set of rows, and each row is an ordered combination of values drawn from the column domains.

**First-order predicate logic:** Each row in a table is a fact (a true proposition). The row `(101, 'Alpine Pro Hiking Boots', 189.50, 42, 1)` in the `products` table asserts: "There exists a product with ID 101, named Alpine Pro Hiking Boots, priced at 189.50, with 42 units in stock, in category 1."

This mathematical foundation is why SQL works so reliably — it's not based on ad hoc conventions but on formal logic with provable properties.

### 1.4 From Theory to Practice

The gap between relational theory and real-world SQL implementations is small but worth noting:

| Relational Theory | SQL Practice |
|---|---|
| Relations are sets (no duplicates) | Tables can have duplicate rows unless constrained |
| Tuples have no order | Rows have no guaranteed order, but ORDER BY can sort results |
| Attributes have no order | Columns have a defined order (position), but it rarely matters |
| NULL doesn't exist in pure theory | NULL is a core part of SQL |

SQL is not a perfect implementation of the relational model, but it's close enough to be enormously useful. Understanding the theory helps you write better SQL and avoid subtle mistakes.

---

## 2. Relations, Tuples, and Attributes

### 2.1 Terminology: Three Sets of Names

The relational model uses formal mathematical terms, everyday informal terms, and SQL terms — all referring to the same concepts. Here's the complete mapping (Watt & Eng, Ch. 7):

| Formal (Relational Theory) | Informal (Everyday) | SQL |
|---|---|---|
| Relation | Table | Table |
| Tuple | Row / Record | Row |
| Attribute | Column / Field | Column |
| Domain | Pool of legal values | Data type + CHECK constraints |
| Relation schema | Table definition | CREATE TABLE statement |
| Relation instance | Table contents at a moment in time | Result of SELECT * |
| Degree | Number of columns | Number of columns |
| Cardinality | Number of rows | COUNT(*) |

You'll encounter all three sets of terms in textbooks, documentation, and job interviews. Being fluent in all of them is important.

### 2.2 TrailShop Example Tables

Let's look at TrailShop's data organized relationally:

**categories**

| category_id | category_name |
|---|---|
| 1 | Footwear |
| 2 | Camping |
| 3 | Climbing |
| 4 | Hiking |
| 5 | Clothing |

**products**

| product_id | name | price | stock_quantity | category_id |
|---|---|---|---|---|
| 101 | Alpine Pro Hiking Boots | 189.50 | 42 | 1 |
| 102 | TrailMaster X4 Tent | 249.99 | 15 | 2 |
| 103 | GripWall Climbing Shoes | 159.99 | 19 | 3 |
| 104 | Summit 45L Backpack | 129.00 | 28 | 4 |
| 105 | StormShield Rain Jacket | 99.95 | 55 | 5 |
| 106 | Basecamp 2P Tent | 199.00 | 22 | 2 |
| 107 | RockHold Harness | 89.99 | 31 | 3 |
| 108 | TrailRunner Socks (3-pack) | 24.99 | 120 | 1 |

**customers**

| customer_id | first_name | last_name | email | city |
|---|---|---|---|---|
| 501 | Laura | Virtanen | laura.v@email.fi | Helsinki |
| 502 | Mikko | Korhonen | mikko.k@email.fi | Tampere |
| 503 | Anna | Mäkelä | anna.m@email.fi | Turku |
| 504 | Juha | Nieminen | juha.n@email.fi | Oulu |

Each table is a **relation**. Each row is a **tuple**. Each column is an **attribute**.

### 2.3 Reading a Relation Formally

The `products` relation can be described formally as:

```
products(product_id: INTEGER, name: VARCHAR, price: NUMERIC,
         stock_quantity: INTEGER, category_id: INTEGER)
```

This notation shows the **relation schema** — the relation name and the list of attributes with their domains.

A specific tuple is a set of values drawn from these domains:

```
(102, 'TrailMaster X4 Tent', 249.99, 15, 2)
```

This tuple asserts a fact: "Product 102 is named TrailMaster X4 Tent, costs 249.99, has 15 units in stock, and belongs to category 2."

---

## 3. Domains

A **domain** is the set of all allowed values for a particular attribute. Think of it as a "pool" of valid values a column can draw from (Watt & Eng, Ch. 7).

### 3.1 Examples for TrailShop

| Attribute | Domain Description | PostgreSQL Type | Example Values |
|---|---|---|---|
| `product_id` | Positive integers | `INTEGER` | 101, 102, 103 |
| `name` | Non-empty text up to 100 characters | `VARCHAR(100)` | 'Alpine Pro Hiking Boots' |
| `price` | Positive decimal numbers | `NUMERIC(10,2)` | 189.50, 249.99 |
| `stock_quantity` | Non-negative integers | `INTEGER` | 0, 15, 42 |
| `category_id` | Positive integers matching a category | `INTEGER` | 1, 2, 3, 4, 5 |
| `email` | Valid email format, up to 255 chars | `VARCHAR(255)` | 'laura.v@email.fi' |
| `order_date` | Valid dates | `DATE` | '2026-09-15' |
| `is_active` | True or False | `BOOLEAN` | TRUE, FALSE |

### 3.2 Domain Integrity

**Domain integrity** means that every value in a column must belong to the column's domain. If someone tries to insert "free" as a price, or -5 as a stock quantity, that violates domain integrity — those values don't belong to the domain.

In PostgreSQL, domains are enforced through a combination of:
- **Data types** (e.g., `INTEGER` rejects text)
- **CHECK constraints** (e.g., `CHECK (price > 0)` rejects zero and negatives)
- **NOT NULL constraints** (rejects missing values when the domain requires a value)

PostgreSQL also supports formal `CREATE DOMAIN` for reusable domain definitions:

```sql
CREATE DOMAIN positive_price AS NUMERIC(10,2)
  CHECK (VALUE > 0);

CREATE DOMAIN nonneg_integer AS INTEGER
  CHECK (VALUE >= 0);
```

You can then use these in table definitions:

```sql
CREATE TABLE products (
    product_id     INTEGER         PRIMARY KEY,
    name           VARCHAR(100)    NOT NULL,
    price          positive_price  NOT NULL,
    stock_quantity nonneg_integer  NOT NULL DEFAULT 0
);
```

### 3.3 Why Domains Matter

Domains prevent nonsensical operations. Consider: should you be able to add `product_id` + `category_id`? Arithmetically it's possible (both are integers), but logically it's meaningless — they come from different domains. While SQL won't prevent this operation, understanding domains helps you design tables where columns have clear, well-defined meanings.

---

## 4. Degree and Cardinality

Two simple measurements describe any relation:

### 4.1 Degree

The **degree** of a relation is the number of attributes (columns).

- The `products` table has degree **5** (product_id, name, price, stock_quantity, category_id)
- The `categories` table has degree **2** (category_id, category_name)
- The `customers` table has degree **5** (customer_id, first_name, last_name, email, city)

Degree is part of the schema — it tends to be **stable**. You don't add columns every day. Adding a column requires an `ALTER TABLE` command and careful consideration of its impact.

### 4.2 Cardinality

The **cardinality** of a relation is the number of tuples (rows).

- The `products` table currently has cardinality **8**
- The `categories` table has cardinality **5**
- The `customers` table has cardinality **4**

Cardinality changes constantly as data is inserted, updated, or deleted. Right now TrailShop has 8 products, but by next month it could be 80 or 800.

### 4.3 Why These Terms Matter

When you're designing queries and thinking about performance, degree and cardinality are fundamental. A table with high cardinality (millions of rows) needs indexes to be searched efficiently. A table with high degree (50+ columns) might indicate a design problem that normalization could fix (you'll study normalization in Week 45).

---

## 5. Properties of a Relation

A proper relation must satisfy six properties (Watt & Eng, Ch. 7). Violating any of these means your structure is not a true relation.

### 5.1 Unique Name

No two relations in the same schema share a name.

**Violation example:** You try to create a second table called `products` while one already exists.

```sql
CREATE TABLE products (...);  -- Already exists
-- ERROR: relation "products" already exists
```

### 5.2 No Duplicate Tuples

Every row must be distinct — at minimum, one combination of columns must be unique across all rows.

**Violation example:** The products table contains two identical rows:

| product_id | name | price | stock_quantity | category_id |
|---|---|---|---|---|
| 101 | Alpine Pro Hiking Boots | 189.50 | 42 | 1 |
| 101 | Alpine Pro Hiking Boots | 189.50 | 42 | 1 |

In pure relational theory, this is impossible (a relation is a *set*, and sets don't contain duplicates). In SQL, you prevent this by defining a primary key — which is what makes `product_id` essential.

### 5.3 Atomic Entries (First Normal Form)

Each cell contains a **single, indivisible value** — not a list, not a set, not a nested table.

**Violation example (TrailShop):** Putting multiple categories in one cell:

| product_id | name | categories |
|---|---|---|
| 101 | Alpine Pro Hiking Boots | Footwear, Hiking |

The value "Footwear, Hiking" is not atomic — it contains two values. This makes querying difficult. Want all footwear products? You'd have to parse strings instead of using simple SQL. The correct approach is to use a separate table or a junction table for products-to-categories.

This property is closely related to **First Normal Form (1NF)**, which you'll study in Week 45.

### 5.4 Entries from the Same Domain

All values in a column come from the same domain. You can't have a `price` column where some rows contain numbers and others contain text.

**Violation example:** `price` column contains `189.50` in one row and `"call for quote"` in another. This is impossible in a properly typed SQL table (the column is defined as `NUMERIC`), but it happens all the time in spreadsheets.

### 5.5 Distinct Attribute Names

No two columns in the same table share a name.

**Violation example:** A products table with two columns both named `name` — one for the product name and one for the brand name. SQL won't allow this. Use distinct names: `product_name`, `brand_name`.

### 5.6 Order Insignificant

Neither rows nor columns have an inherent order.

- **Row order:** The database doesn't guarantee that rows come back in any particular order unless you explicitly use `ORDER BY`.
- **Column order:** While columns do have a positional order in SQL (defined by the `CREATE TABLE` statement), you should never rely on it. Always reference columns by name.

**TrailShop consequence:** Don't write code that assumes "the first column is the product ID." Always use `SELECT product_id, name FROM products`, not `SELECT * FROM products` and then assume positions.

---

## 6. Keys — Comprehensive Coverage

Keys are the backbone of the relational model. They uniquely identify rows and connect tables to each other. This section covers every type of key you need to know, as described in *Database Design* (2nd Ed., Ch. 8).

### 6.1 Superkey

A **superkey** is any set of attributes that uniquely identifies every tuple in a relation. A superkey may contain extra (unnecessary) attributes.

**TrailShop examples** (for the `products` table):
- `{product_id}` — uniquely identifies each product ✓
- `{product_id, name}` — also unique, but `name` is unnecessary (redundant) ✓
- `{product_id, name, price}` — still unique, but has two extra attributes ✓
- `{name}` — unique if all product names are different ✓ (but risky)
- `{price}` — NOT a superkey, because two products could have the same price ✗

Every candidate key is a superkey, but not every superkey is a candidate key.

### 6.2 Candidate Key

A **candidate key** is a *minimal* superkey — you can't remove any attribute from it and still have uniqueness.

**Formal definition:** A set of attributes K is a candidate key of relation R if:
1. **Uniqueness:** No two distinct tuples in R have the same values for K
2. **Minimality (irreducibility):** No proper subset of K has the uniqueness property

**TrailShop examples** (for `products`):
- `{product_id}` — candidate key (unique and minimal)
- `{product_id, name}` — NOT a candidate key (unique but not minimal — `product_id` alone is sufficient)
- `{name}` — candidate key IF we guarantee all product names are unique

**Common mistake:** Confusing superkeys with candidate keys. `{product_id, name}` is a superkey but not a candidate key because it's not minimal.

### 6.3 Primary Key

The **primary key (PK)** is the candidate key you choose as the official row identifier for the table.

**Rules:**
- Every table must have exactly one primary key
- A primary key can never contain NULL values (entity integrity rule)
- Primary key values must be unique across all rows
- Once chosen, the primary key should be stable — it shouldn't change

**TrailShop choices:**
- `products` → PK: `product_id`
- `categories` → PK: `category_id`
- `customers` → PK: `customer_id`

**How to choose a primary key:**
- Prefer a single column over a combination (simpler)
- Prefer a value that never changes (IDs over names)
- Prefer a value that has no business meaning (surrogate keys — see Section 6.9)
- Ensure it's never NULL

```sql
CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,
    -- ...
);
```

### 6.4 Composite Key

A **composite key** is a key made of two or more columns together. Neither column alone is unique, but the combination is.

**TrailShop example:** Consider an `order_items` table:

| order_id | product_id | quantity | unit_price |
|---|---|---|---|
| 1001 | 101 | 1 | 189.50 |
| 1001 | 102 | 2 | 249.99 |
| 1002 | 101 | 1 | 189.50 |
| 1002 | 105 | 3 | 99.95 |

- `order_id` alone isn't unique (order 1001 appears twice)
- `product_id` alone isn't unique (product 101 appears twice)
- `(order_id, product_id)` together IS unique — each product appears at most once per order

```sql
CREATE TABLE order_items (
    order_id   INTEGER REFERENCES orders(order_id),
    product_id INTEGER REFERENCES products(product_id),
    quantity   INTEGER NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10,2) NOT NULL,
    PRIMARY KEY (order_id, product_id)
);
```

**Common mistake:** Using composite keys when a simpler surrogate key would suffice. Composite keys are appropriate when the combination naturally identifies the row (like order + product in line items).

### 6.5 Foreign Key

A **foreign key (FK)** is a column (or set of columns) in one table that references the primary key of another table. It creates a link between the two tables.

**TrailShop example:** `products.category_id` is a foreign key that references `categories.category_id`. This means every product must belong to a valid category.

```
categories                      products
┌─────────────┬───────────┐    ┌────────────┬────────────┬────────────┐
│ category_id │ name      │    │ product_id │ name       │ category_id│
├─────────────┼───────────┤    ├────────────┼────────────┼────────────┤
│ 1           │ Footwear  │◄───│ 101        │ Boots      │ 1          │
│ 2           │ Camping   │◄───│ 102        │ Tent       │ 2          │
│ 3           │ Climbing  │    │ 103        │ Shoes      │ 3          │
└─────────────┴───────────┘    └────────────┴────────────┴────────────┘
```

The arrows show the relationship: each `category_id` in `products` points to a row in `categories`.

**Key rules about foreign keys:**
- A FK must reference the primary key (or a unique key) of another table
- A FK can be NULL if the business allows it (e.g., "category not yet assigned")
- A FK value must match an existing PK value in the referenced table, or be NULL — this is **referential integrity**

```sql
CREATE TABLE products (
    product_id     INTEGER       PRIMARY KEY,
    name           VARCHAR(100)  NOT NULL,
    price          NUMERIC(10,2) NOT NULL CHECK (price > 0),
    stock_quantity INTEGER       NOT NULL DEFAULT 0,
    category_id    INTEGER       REFERENCES categories(category_id)
);
```

**Common mistake:** Forgetting to create the referenced table first. If `categories` doesn't exist yet, creating `products` with a FK reference to it will fail.

### 6.6 Alternate Key

Any candidate key that was **not** chosen as the primary key is called an **alternate key**.

**TrailShop example:** If both `product_id` and `name` are candidate keys for `products`, and you choose `product_id` as the PK, then `name` becomes an alternate key. You'd typically enforce it with a `UNIQUE` constraint:

```sql
CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,
    name       VARCHAR(100) NOT NULL UNIQUE,  -- alternate key
    -- ...
);
```

### 6.7 Secondary Key

A **secondary key** is a column (or columns) used frequently for searching or sorting, but not necessarily unique. It's not a formal concept in the relational model — it's a practical consideration for performance.

**TrailShop example:** You often search products by `category_id` or sort by `price`. These are secondary keys — you'd create **indexes** on them to speed up queries:

```sql
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_price ON products(price);
```

You'll learn about indexes in detail in Week 47.

### 6.8 Surrogate Key

A **surrogate key** is an artificial key with no business meaning — typically an auto-incrementing integer or a UUID. Its only purpose is to uniquely identify rows.

**TrailShop example:** `product_id` is a surrogate key. It's a number assigned by the database. It doesn't encode any information about the product — it's just an ID.

```sql
CREATE TABLE products (
    product_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    -- The database automatically generates 1, 2, 3, ...
    name       VARCHAR(100) NOT NULL,
    -- ...
);
```

**Advantages of surrogate keys:**
- Simple (single column, integer)
- Stable (never changes)
- Compact (small storage, fast joins)
- Independent of business data

### 6.9 Natural Key

A **natural key** is a key that has real-world business meaning — it's drawn from the data itself, not generated artificially.

**Examples:**
- ISBN for books
- Social security number for citizens (though privacy concerns make this a bad PK choice)
- Email address for users (unique, but can change)
- Country code (e.g., 'FI', 'SE', 'NO')

**TrailShop example:** A product's **EAN/barcode** could be a natural key:

```sql
CREATE TABLE products (
    ean            VARCHAR(13)    PRIMARY KEY,  -- natural key
    name           VARCHAR(100)   NOT NULL,
    -- ...
);
```

**Natural vs Surrogate — which to choose?**

| Criterion | Surrogate Key | Natural Key |
|---|---|---|
| Business meaning | None | Yes |
| Stability | Always stable | May change (emails, names) |
| Simplicity | Single integer | May be long or composite |
| Universality | Always available | May not exist for all entities |
| Human readability | Low (`product_id = 104`) | High (`ean = '6430012345678'`) |

In practice, most database designers prefer **surrogate keys** for primary keys, with natural keys enforced as `UNIQUE` alternate keys when appropriate.

### 6.10 Summary of Key Types

| Key Type | Definition | TrailShop Example |
|---|---|---|
| Superkey | Any set of attributes that uniquely identifies rows | `{product_id}`, `{product_id, name}` |
| Candidate key | Minimal superkey | `{product_id}`, possibly `{name}` |
| Primary key | Chosen candidate key; never NULL | `product_id` |
| Alternate key | Candidate key not chosen as PK | `name` (if unique) |
| Composite key | Key with 2+ attributes | `(order_id, product_id)` in order_items |
| Foreign key | References another table's PK | `products.category_id` → `categories.category_id` |
| Secondary key | Used for searching/indexing | `category_id` in products (for filtering) |
| Surrogate key | Artificial, no business meaning | Auto-generated `product_id` |
| Natural key | Real-world, meaningful value | EAN barcode, ISBN, email |

---

## 7. Null Values

A **NULL** in a database means "value unknown" or "value not applicable." It is NOT the same as zero, an empty string, or a space (Watt & Eng, Ch. 8).

### 7.1 When NULLs Occur

- **Unknown:** A new product hasn't been weighed yet → `weight_kg` is NULL
- **Not applicable:** A digital gift card has no physical weight → `weight_kg` is NULL
- **Not yet assigned:** A product hasn't been assigned to a category yet → `category_id` is NULL (if allowed)

### 7.2 NULL Behavior in Arithmetic

Any arithmetic operation involving NULL produces NULL:

```sql
SELECT NULL + 5;          -- Result: NULL
SELECT NULL * 100;        -- Result: NULL
SELECT NULL - NULL;       -- Result: NULL
SELECT 10 / NULL;         -- Result: NULL
```

**TrailShop impact:** If a product's `weight_kg` is NULL and you calculate shipping as `weight_kg * 2.50`, the result is NULL — not zero, not an error, just NULL. Your application must handle this.

### 7.3 NULL Behavior in Comparisons

Comparing anything to NULL never returns TRUE — it returns **UNKNOWN** (a third truth value beyond TRUE and FALSE):

```sql
SELECT * FROM products WHERE weight_kg = NULL;    -- Returns NO rows (wrong!)
SELECT * FROM products WHERE weight_kg IS NULL;    -- Correct way to check for NULL
SELECT * FROM products WHERE weight_kg IS NOT NULL; -- Finds rows with values
```

**Critical rule:** Never use `= NULL` or `!= NULL`. Always use `IS NULL` or `IS NOT NULL`.

```sql
-- These all return UNKNOWN (effectively FALSE in WHERE clauses):
SELECT NULL = NULL;       -- UNKNOWN (not TRUE!)
SELECT NULL <> NULL;      -- UNKNOWN
SELECT NULL > 5;          -- UNKNOWN
SELECT NULL = 0;          -- UNKNOWN
SELECT NULL = '';          -- UNKNOWN
```

**Two NULLs are not considered equal to each other.** This surprises many beginners.

### 7.4 NULL Behavior in Aggregations

Aggregate functions generally **ignore** NULLs (except `COUNT(*)`):

```sql
-- Suppose 3 products have weight_kg values: 2.5, NULL, 1.8
SELECT COUNT(*)           FROM products;  -- 3 (counts all rows)
SELECT COUNT(weight_kg)   FROM products;  -- 2 (counts non-NULL values only)
SELECT AVG(weight_kg)     FROM products;  -- 2.15 (averages 2.5 and 1.8, ignores NULL)
SELECT SUM(weight_kg)     FROM products;  -- 4.3  (sums 2.5 and 1.8, ignores NULL)
```

Notice that `AVG` divides by 2 (the non-NULL count), not by 3 (the total row count). This can lead to unexpected results if you're not aware of it.

### 7.5 NULL Rules Summary

| Rule | Example |
|---|---|
| Primary keys can **never** be NULL | `product_id` must always have a value |
| Foreign keys *can* be NULL (if allowed) | `category_id` could be NULL = "not categorized yet" |
| NULLs propagate in arithmetic | `NULL + 5 = NULL` |
| NULLs make comparisons UNKNOWN | `NULL = NULL` is UNKNOWN, not TRUE |
| Use `IS NULL` / `IS NOT NULL` | Never use `= NULL` |
| Aggregates ignore NULLs | `AVG` averages only non-NULL values |

### 7.6 Dealing with NULLs in Practice

PostgreSQL provides functions to handle NULLs:

```sql
-- COALESCE: return the first non-NULL value
SELECT name, COALESCE(weight_kg, 0) AS weight_kg
FROM products;
-- If weight_kg is NULL, display 0 instead

-- NULLIF: return NULL if two values are equal
SELECT NULLIF(stock_quantity, 0) AS stock_or_null
FROM products;
-- Returns NULL if stock is 0, otherwise returns the stock value
```

You will practice working with NULLs in the exercises.

---

## 8. Integrity Rules

Integrity rules are the laws that keep your database trustworthy. As described in *Database Design* (2nd Ed., Ch. 9), there are two fundamental integrity rules in the relational model.

### 8.1 Entity Integrity

> **Rule:** Every table must have a primary key, and no part of that primary key can be NULL.

**Why?** Because if a primary key were NULL, you couldn't uniquely identify the row — and the whole system of identification and referencing breaks down. A row without an identity is a row you can't reliably find, update, or delete.

**TrailShop demonstration:**

```sql
-- This will FAIL because product_id (PK) cannot be NULL
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (NULL, 'Mystery Product', 49.99, 10, 2);
```

PostgreSQL error:
```
ERROR:  null value in column "product_id" of relation "products" violates
        not-null constraint
DETAIL:  Failing row contains (null, Mystery Product, 49.99, 10, 2).
```

**With composite keys**, entity integrity applies to *every* column in the key. If the PK of `order_items` is `(order_id, product_id)`, then *neither* `order_id` *nor* `product_id` can be NULL.

```sql
-- This also FAILS — both parts of a composite PK must be non-NULL
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (NULL, 101, 1, 189.50);
```

### 8.2 Referential Integrity

> **Rule:** A foreign key value must either match an existing primary key value in the referenced table, or be NULL (if NULLs are allowed for that FK).

This prevents **orphan records** — rows that reference something that doesn't exist. An orphan record is like a package addressed to a house that was demolished — it has nowhere to go.

**TrailShop demonstrations:**

```sql
-- FAIL: category_id 99 does not exist in the categories table
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (109, 'Ghost Product', 59.99, 5, 99);
```

PostgreSQL error:
```
ERROR:  insert or update on table "products" violates foreign key constraint
        "products_category_id_fkey"
DETAIL:  Key (category_id)=(99) is not present in table "categories".
```

```sql
-- SUCCEED: category_id 2 exists in categories
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (109, 'AeroLite Tent', 279.00, 10, 2);
```

```sql
-- SUCCEED (if category_id allows NULL): product with no category yet
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (110, 'Uncategorized Item', 15.00, 100, NULL);
```

### 8.3 Referential Integrity on DELETE and UPDATE

What happens when you try to delete a category that has products referencing it?

```sql
-- Try to delete category 1 (Footwear)
-- Products 101 and 108 reference this category!
DELETE FROM categories WHERE category_id = 1;
```

By default, PostgreSQL **rejects** this with a foreign key violation. But you can configure different behaviors — see Section 10 (Foreign Key Actions).

### 8.4 What About Duplicate Primary Keys?

This isn't technically an integrity rule, but a consequence of the PK's uniqueness constraint:

```sql
-- FAIL: product_id 101 already exists
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (101, 'Duplicate Product', 39.99, 8, 1);
```

PostgreSQL error:
```
ERROR:  duplicate key value violates unique constraint "products_pkey"
DETAIL:  Key (product_id)=(101) already exists.
```

---

## 9. Constraints in PostgreSQL — Comprehensive Coverage

Constraints are the mechanism PostgreSQL uses to enforce integrity rules. Here is every major constraint type, with syntax and TrailShop examples. For full reference, see: https://www.postgresql.org/docs/current/ddl-constraints.html

### 9.1 NOT NULL

Prevents NULL values in a column.

```sql
name VARCHAR(100) NOT NULL
```

**When to use:** When a value is always required. Every product must have a name; every order must have a date.

### 9.2 UNIQUE

Ensures no two rows have the same value in this column (NULLs are allowed and don't count as duplicates).

```sql
email VARCHAR(255) UNIQUE
```

**When to use:** For alternate keys and columns that must be distinct. Category names should be unique; customer email addresses should be unique.

### 9.3 PRIMARY KEY

Combines NOT NULL + UNIQUE. Identifies each row.

```sql
product_id INTEGER PRIMARY KEY

-- Composite primary key (table-level constraint):
PRIMARY KEY (order_id, product_id)
```

### 9.4 FOREIGN KEY (REFERENCES)

Links a column to the primary key of another table.

```sql
-- Column-level syntax:
category_id INTEGER REFERENCES categories(category_id)

-- Table-level syntax (required for composite FKs):
FOREIGN KEY (category_id) REFERENCES categories(category_id)
```

### 9.5 CHECK

Enforces a custom boolean condition. The row is rejected if the condition evaluates to FALSE.

```sql
price NUMERIC(10,2) CHECK (price > 0)
stock_quantity INTEGER CHECK (stock_quantity >= 0)

-- Multi-column check (table-level):
CHECK (end_date > start_date)
```

### 9.6 DEFAULT

Provides a value when none is specified during INSERT.

```sql
stock_quantity INTEGER NOT NULL DEFAULT 0
created_at TIMESTAMP NOT NULL DEFAULT NOW()
is_active BOOLEAN NOT NULL DEFAULT TRUE
```

**Note:** DEFAULT is not strictly a constraint (it doesn't restrict data), but it's commonly listed alongside constraints because it's part of column definitions.

### 9.7 EXCLUDE

An advanced constraint unique to PostgreSQL that prevents overlapping or conflicting values. Often used with range types.

```sql
-- Prevent overlapping booking periods for the same room
CREATE TABLE room_bookings (
    room_id    INTEGER,
    period     DATERANGE,
    EXCLUDE USING gist (room_id WITH =, period WITH &&)
);
```

You won't need EXCLUDE in this course, but it's good to know it exists.

### 9.8 Complete CREATE TABLE Example

Here's a complete TrailShop schema using all common constraint types:

```sql
CREATE TABLE categories (
    category_id   INTEGER       PRIMARY KEY,
    category_name VARCHAR(50)   NOT NULL UNIQUE,
    description   TEXT          DEFAULT ''
);

CREATE TABLE products (
    product_id     INTEGER       PRIMARY KEY,
    name           VARCHAR(100)  NOT NULL UNIQUE,
    price          NUMERIC(10,2) NOT NULL CHECK (price > 0),
    stock_quantity INTEGER       NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    weight_kg      NUMERIC(6,2)  CHECK (weight_kg > 0),  -- NULL allowed (weight unknown)
    category_id    INTEGER       NOT NULL REFERENCES categories(category_id),
    created_at     TIMESTAMP     NOT NULL DEFAULT NOW()
);

CREATE TABLE customers (
    customer_id INTEGER       PRIMARY KEY,
    first_name  VARCHAR(50)   NOT NULL,
    last_name   VARCHAR(50)   NOT NULL,
    email       VARCHAR(255)  NOT NULL UNIQUE,
    city        VARCHAR(100)
);

CREATE TABLE orders (
    order_id    INTEGER   PRIMARY KEY,
    customer_id INTEGER   NOT NULL REFERENCES customers(customer_id),
    order_date  DATE      NOT NULL DEFAULT CURRENT_DATE,
    status      VARCHAR(20) NOT NULL DEFAULT 'pending'
                CHECK (status IN ('pending', 'shipped', 'delivered', 'cancelled'))
);

CREATE TABLE order_items (
    order_id   INTEGER       NOT NULL REFERENCES orders(order_id),
    product_id INTEGER       NOT NULL REFERENCES products(product_id),
    quantity   INTEGER       NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10,2) NOT NULL CHECK (unit_price > 0),
    PRIMARY KEY (order_id, product_id)
);
```

Notice:
- `categories` must be created before `products` (because `products` references it)
- `customers` must be created before `orders`
- `orders` and `products` must be created before `order_items`

### 9.9 Naming Constraints

PostgreSQL auto-generates constraint names, but you can (and should, for production code) name them explicitly for clearer error messages:

```sql
CREATE TABLE products (
    product_id     INTEGER       CONSTRAINT pk_products PRIMARY KEY,
    name           VARCHAR(100)  CONSTRAINT nn_products_name NOT NULL,
    price          NUMERIC(10,2) NOT NULL
                   CONSTRAINT chk_products_price_positive CHECK (price > 0),
    category_id    INTEGER       NOT NULL
                   CONSTRAINT fk_products_category
                   REFERENCES categories(category_id)
);
```

When a constraint is violated, the error message includes the constraint name — making debugging much easier.

---

## 10. Foreign Key Actions

When a referenced row is deleted or updated, what should happen to the rows that reference it? PostgreSQL supports five options for both `ON DELETE` and `ON UPDATE` (Watt & Eng, Ch. 9):

### 10.1 The Options

| Action | ON DELETE behavior | ON UPDATE behavior |
|---|---|---|
| `RESTRICT` | Prevent deletion if referenced | Prevent update if referenced |
| `NO ACTION` | Same as RESTRICT (checked at end of statement) | Same as RESTRICT |
| `CASCADE` | Delete referencing rows too | Update FK values to match new PK |
| `SET NULL` | Set FK to NULL | Set FK to NULL |
| `SET DEFAULT` | Set FK to its default value | Set FK to its default value |

### 10.2 TrailShop Examples

**CASCADE — Delete category and all its products:**

```sql
CREATE TABLE products (
    -- ...
    category_id INTEGER REFERENCES categories(category_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- If you delete category 2 (Camping), ALL products in category 2
-- are automatically deleted too!
DELETE FROM categories WHERE category_id = 2;
-- Products 102 and 106 are automatically deleted
```

**Use with caution!** CASCADE can delete more data than you expect.

**RESTRICT — Prevent deletion if products exist:**

```sql
CREATE TABLE products (
    -- ...
    category_id INTEGER REFERENCES categories(category_id)
        ON DELETE RESTRICT
);

-- This FAILS if any products reference category 2
DELETE FROM categories WHERE category_id = 2;
-- ERROR: update or delete on table "categories" violates foreign key constraint
```

This is the **safest default** — it forces you to explicitly handle dependent data.

**SET NULL — Remove the link but keep the product:**

```sql
CREATE TABLE products (
    -- ...
    category_id INTEGER REFERENCES categories(category_id)
        ON DELETE SET NULL
);

-- Deleting category 2 sets category_id to NULL for products 102 and 106
DELETE FROM categories WHERE category_id = 2;
-- Products 102 and 106 now have category_id = NULL (uncategorized)
```

This only works if the FK column allows NULLs (no `NOT NULL` constraint).

### 10.3 Choosing the Right Action

| Scenario | Recommended Action |
|---|---|
| Deleting a customer should delete their orders | `CASCADE` (or `RESTRICT` if orders should be archived) |
| Deleting a category should NOT delete products | `RESTRICT` or `SET NULL` |
| Renaming a category's ID should update all products | `ON UPDATE CASCADE` |
| Deleting an employee should keep their past records | `SET NULL` (if allowed) or `RESTRICT` |

In practice, **`RESTRICT`** (or `NO ACTION`) is the safest default. Use `CASCADE` only when you've carefully considered the implications.

---

## 11. Business Rules and Constraints

A **business rule** is a statement that defines or constrains some aspect of the business. Your job as a database designer is to translate these rules into enforceable constraints (Watt & Eng, Ch. 9).

### 11.1 Types of Business Rules

**Structural rules** define the structure of data:
- "Every product has a name, price, and category"
- "Every order belongs to exactly one customer"

**Behavioral rules** define valid data values:
- "Product price must be greater than zero"
- "Stock quantity cannot be negative"
- "Order status must be one of: pending, shipped, delivered, cancelled"

**Referential rules** define relationships:
- "Every product must belong to an existing category"
- "Every order must be placed by an existing customer"

### 11.2 Mapping Business Rules to Constraints

| TrailShop Business Rule | Constraint Type | SQL Implementation |
|---|---|---|
| Every product must have a name | `NOT NULL` | `name VARCHAR(100) NOT NULL` |
| Product price must be positive | `CHECK` | `CHECK (price > 0)` |
| Category names must be unique | `UNIQUE` | `category_name VARCHAR(50) UNIQUE` |
| Stock cannot be negative | `CHECK` | `CHECK (stock_quantity >= 0)` |
| Every product must belong to a category | `NOT NULL` + `FK` | `category_id INTEGER NOT NULL REFERENCES categories` |
| Customer emails must be unique | `UNIQUE` | `email VARCHAR(255) UNIQUE` |
| Order status must be valid | `CHECK` | `CHECK (status IN ('pending','shipped','delivered','cancelled'))` |
| Each product appears at most once per order | `PK` | `PRIMARY KEY (order_id, product_id)` |
| Order quantity must be positive | `CHECK` | `CHECK (quantity > 0)` |
| New products default to 0 stock | `DEFAULT` | `stock_quantity INTEGER DEFAULT 0` |
| Deleting a category must not orphan products | FK action | `ON DELETE RESTRICT` |
| A product's discount cannot exceed its price | `CHECK` | `CHECK (discount <= price)` |

### 11.3 Business Rules That Can't Be Expressed as Simple Constraints

Some rules are too complex for basic constraints:
- "A customer can have at most 3 pending orders at the same time" → requires a trigger or application logic
- "Product price can only increase, never decrease" → requires a trigger
- "Orders placed after 5 PM are processed the next business day" → application logic

You'll learn about triggers in later weeks. For now, focus on rules that map to standard constraints.

---

## 12. Relationships and Cardinality

Tables don't exist in isolation — they relate to each other. The type of relationship is described by its **cardinality** (Watt & Eng, Ch. 8).

### 12.1 One-to-One (1:1)

One row in Table A relates to exactly one row in Table B, and vice versa.

**TrailShop example:** Each product has exactly one detailed description (stored separately for performance):

```
products (1) ──── (1) product_details
```

```sql
CREATE TABLE product_details (
    product_id  INTEGER PRIMARY KEY REFERENCES products(product_id),
    full_description TEXT,
    care_instructions TEXT,
    warranty_months INTEGER
);
```

The PK of `product_details` is also the FK to `products` — this enforces the 1:1 relationship.

**When to use 1:1:** When a table has many columns that are rarely needed, splitting them into a separate table can improve performance. Also used when access permissions differ between the main and detail data.

### 12.2 One-to-Many (1:N)

One row in Table A relates to many rows in Table B. This is the **most common** relationship type.

**TrailShop examples:**
- One **category** has many **products** (but each product belongs to one category)
- One **customer** has many **orders** (but each order belongs to one customer)
- One **order** has many **order_items** (but each line item belongs to one order)

```
categories (1) ──── (N) products
customers  (1) ──── (N) orders
orders     (1) ──── (N) order_items
```

The "many" side always holds the foreign key. `products` has `category_id` (the FK pointing to `categories`), not the other way around.

### 12.3 Many-to-Many (M:N)

Many rows in Table A relate to many rows in Table B. This cannot be represented directly with a foreign key — it requires a **junction table** (also called a linking table, bridge table, or associative entity).

**TrailShop example:** Products can have multiple tags (e.g., "waterproof", "lightweight", "sale"), and each tag can apply to multiple products:

```
products (M) ──── (N) tags
```

Implemented with a junction table:

```sql
CREATE TABLE tags (
    tag_id   INTEGER      PRIMARY KEY,
    tag_name VARCHAR(30)  NOT NULL UNIQUE
);

CREATE TABLE product_tags (
    product_id INTEGER REFERENCES products(product_id),
    tag_id     INTEGER REFERENCES tags(tag_id),
    PRIMARY KEY (product_id, tag_id)
);
```

Another example: The `order_items` table is itself a junction table implementing the M:N relationship between `orders` and `products`.

### 12.4 Crow's Foot Notation Preview

In Week 38, you'll learn **Entity-Relationship (ER) diagrams** using **crow's foot notation** to visually represent relationships:

```
categories ──┤├──── products
              1:N

customers  ──┤├──── orders
              1:N

products   ──┤├──── product_tags ────┤├── tags
              M:N (via junction table)
```

For now, just know that these visual notations exist and that you'll be drawing them soon.

---

## 13. Key Terms

| Term | Definition |
|---|---|
| **Relation** | A table — a named, two-dimensional structure of rows and columns |
| **Tuple** | A row in a relation — represents one entity instance |
| **Attribute** | A column in a relation — represents one property |
| **Domain** | The set of all permitted values for an attribute |
| **Domain integrity** | The rule that all values must belong to the attribute's domain |
| **Degree** | The number of attributes (columns) in a relation |
| **Cardinality** | The number of tuples (rows) in a relation |
| **Superkey** | Any set of attributes that uniquely identifies all tuples |
| **Candidate key** | A minimal superkey (no redundant attributes) |
| **Primary key (PK)** | The chosen candidate key; never NULL, always unique |
| **Alternate key** | A candidate key not chosen as the primary key |
| **Composite key** | A key composed of two or more attributes |
| **Foreign key (FK)** | An attribute that references the primary key of another table |
| **Secondary key** | A column used for indexing/searching, not necessarily unique |
| **Surrogate key** | An artificial key with no business meaning (auto-generated ID) |
| **Natural key** | A key drawn from real-world data (ISBN, email, etc.) |
| **Null** | A marker indicating a value is unknown or not applicable |
| **Entity integrity** | Rule: every table must have a PK, and no part of it can be NULL |
| **Referential integrity** | Rule: a FK must match an existing PK value or be NULL |
| **Orphan record** | A row whose FK references a non-existent PK |
| **Constraint** | A rule enforced by the DBMS to maintain data validity |
| **Business rule** | A policy or condition that the database must enforce |
| **Junction table** | A table that implements a many-to-many relationship |
| **CASCADE** | FK action: propagate deletion/update to referencing rows |
| **RESTRICT** | FK action: prevent deletion/update if references exist |
| **COALESCE** | SQL function returning the first non-NULL argument |
| **First Normal Form (1NF)** | Requires atomic values in every cell |

---

## 14. Reading Assignments

**Required:**
- *Database Design*, 2nd Edition by Adrienne Watt & Nelson Eng — Chapters 7, 8, and 9
  - Chapter 7: The Relational Data Model
  - Chapter 8: Keys and Integrity (keys, null values)
  - Chapter 9: Integrity Rules and Constraints
- PostgreSQL Docs: Data Definition — https://www.postgresql.org/docs/current/ddl.html
- PostgreSQL Docs: Constraints — https://www.postgresql.org/docs/current/ddl-constraints.html

**Further Reading:**
- E.F. Codd, "A Relational Model of Data for Large Shared Data Banks" (1970) — https://www.seas.upenn.edu/~zives/03f/cis550/codd.pdf
- Codd's 12 Rules — https://en.wikipedia.org/wiki/Codd%27s_12_rules
- PostgreSQL Docs: CREATE TABLE — https://www.postgresql.org/docs/current/sql-createtable.html
- PostgreSQL Docs: CREATE DOMAIN — https://www.postgresql.org/docs/current/sql-createdomain.html
- GeeksforGeeks: Relational Model in DBMS — https://www.geeksforgeeks.org/relational-model-in-dbms/
- W3Schools SQL Tutorial — https://www.w3schools.com/sql/
- Wikipedia: Relational Model — https://en.wikipedia.org/wiki/Relational_model

---

## 15. Summary

This week you've built the theoretical foundation for everything that follows in this course. Here's what you've learned:

- **The relational model** was proposed by E.F. Codd in 1970, grounded in set theory and predicate logic. It revolutionized data management by separating logical structure from physical storage.
- **Relations, tuples, and attributes** are the formal terms for tables, rows, and columns. A relation has a fixed degree (number of columns) and a changing cardinality (number of rows).
- **Domains** define the set of valid values for each attribute, enforced through data types and CHECK constraints.
- **Properties of a relation** — unique name, no duplicates, atomic entries, same domain per column, distinct attribute names, and insignificant order — define what makes a table a proper relation.
- **Keys** are the backbone: superkeys, candidate keys, primary keys, foreign keys, composite keys, alternate keys, surrogate keys, and natural keys each play a specific role.
- **NULL** means "unknown" or "not applicable" — not zero, not empty. NULLs propagate in arithmetic, make comparisons return UNKNOWN, and require `IS NULL` to test.
- **Entity integrity** says primary keys can never be NULL. **Referential integrity** says foreign keys must match existing primary keys (or be NULL).
- **Constraints** (`NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`) are how PostgreSQL enforces these rules.
- **Foreign key actions** (`CASCADE`, `RESTRICT`, `SET NULL`, etc.) control what happens when referenced data is deleted or updated.
- **Business rules** are translated into constraints to prevent bad data from entering the system.
- **Relationships** (1:1, 1:N, M:N) describe how tables connect. M:N relationships require junction tables.

**Next week:** You'll step back and look at the bigger picture — modelling TrailShop's entire business as an Entity-Relationship diagram before writing a single line of SQL. Get ready for Chapter 3: "Drawing the Blueprint."
