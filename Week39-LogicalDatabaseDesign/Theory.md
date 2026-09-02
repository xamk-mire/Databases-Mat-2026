# Week 39 — Logical Database Design

## Chapter 4: "From Blueprint to Building Plan"

Last week you created the blueprint — an ER diagram that maps TrailShop's data landscape. You identified entities, attributes, relationships, and cardinality constraints. But an ER diagram is like an architect's sketch: it shows *what* the building should look like, not *how* to actually construct it.

This week you become the construction engineer. You'll take the ER model and **transform** it into a relational schema — actual PostgreSQL tables with data types, primary keys, foreign keys, and constraints. Every entity becomes a table. Every attribute becomes a column. Every relationship becomes a foreign key (or a junction table). Every business rule becomes a constraint.

This transformation isn't random. There are formal **mapping rules** that guarantee your relational schema faithfully represents the ER model. Follow the rules, and the database will behave exactly as the model promised.

> "A good design implemented poorly is better than no design at all — but a good design implemented well is where the magic happens."

---

## Learning Objectives

By the end of this chapter you will be able to:

- Describe the database development lifecycle and where logical design fits
- Apply transformation rules to convert ER constructs into relational tables
- Select appropriate PostgreSQL data types for each column
- Define all types of constraints in PostgreSQL
- Choose appropriate foreign key actions (CASCADE, RESTRICT, etc.)
- Recognize design anomalies and understand why they matter
- Apply naming conventions consistently
- Compare surrogate and natural keys
- Write complete CREATE TABLE statements for the TrailShop schema

---

## 1. The Database Development Lifecycle

### 1.1 Where Logical Design Fits

*Database Design* (Watt & Eng), Chapter 13, describes the database development process as a series of phases that mirror the software development lifecycle (SDLC). The traditional waterfall model includes:

```
┌──────────────────────────────────┐
│  1. Requirements Gathering       │  ← "What does the business need?"
├──────────────────────────────────┤
│  2. Conceptual Design            │  ← ER diagram (Week 38)
├──────────────────────────────────┤
│  3. Logical Design               │  ← Relational schema (THIS WEEK)
├──────────────────────────────────┤
│  4. Physical Design              │  ← Indexes, storage, tuning
├──────────────────────────────────┤
│  5. Implementation               │  ← CREATE TABLE, load data
├──────────────────────────────────┤
│  6. Testing & Validation         │  ← Verify correctness
├──────────────────────────────────┤
│  7. Maintenance & Evolution      │  ← Ongoing changes
└──────────────────────────────────┘
```

In reality, these phases often overlap and iterate. You might discover during logical design that the ER model needs changes, or during implementation that a constraint is too restrictive. The waterfall model is a useful framework for understanding the *order* of decisions, even if real projects cycle back and forth.

### 1.2 The Phases in Brief

**Requirements Gathering** — Interview stakeholders, study existing systems, document what data the organization needs. Output: data requirements document.

**Conceptual Design** — Create a technology-independent ER model. Output: ER diagram (what you did in Week 38).

**Logical Design** — Transform the ER model into a relational schema. Choose data types, define keys and constraints. Output: complete table definitions. This is what we do this week.

**Physical Design** — Optimize for performance: create indexes, define storage parameters, plan partitioning. Output: physical schema with indexes and tuning.

**Implementation** — Execute the DDL statements, load initial data, deploy applications. Output: working database.

**Testing** — Verify data integrity, test queries, validate business rules, stress test. Output: validated system.

**Maintenance** — Handle schema changes, performance tuning, backups, and evolving requirements. Output: a healthy, evolving database.

---

## 2. Requirements Gathering

### 2.1 What It Involves

Before you can design anything, you need to understand what the business needs. Chapter 13 (Watt & Eng) emphasizes that requirements gathering is the most critical phase — mistakes here propagate through every subsequent step.

For TrailShop, a requirements document might include:

**Data Requirements:**
- The system must store product information including name, description, price, weight, and stock quantity
- Products are organized into categories; each product belongs to exactly one category
- The system must track customers with contact information and addresses
- Customers place orders; each order has a date, status, and shipping address
- Each order contains one or more items, each referencing a product with a quantity and the unit price at the time of ordering

**Business Rules:**
- Product prices must be positive
- Stock quantity cannot be negative
- Every product must have a category
- Every order must belong to a customer
- Category names must be unique
- Customer emails must be unique
- An order must have at least one item

### 2.2 From Requirements to Model

The requirements document feeds into both the ER diagram (conceptual design) and the relational schema (logical design). In practice, you often work on both simultaneously — sketching the ER diagram while refining requirements, and thinking about data types while reviewing the model.

---

## 3. Transformation Rules — Comprehensive

This is the core of logical design: systematic rules for converting every ER construct into relational tables. Chapter 10 (Watt & Eng) covers ER-to-relational mapping, and Chapter 13 provides additional guidelines.

### 3.1 Rule 1: Strong Entities → Tables

**Rule:** Each strong entity becomes a table. Each attribute becomes a column. The entity's key attribute becomes the primary key.

**Example: Category**

ER:
```
Category
  - category_id (PK)
  - category_name
  - description
```

SQL:
```sql
CREATE TABLE categories (
    category_id   INTEGER      PRIMARY KEY,
    category_name VARCHAR(50)  NOT NULL UNIQUE,
    description   TEXT
);
```

**Example: Customer**

ER:
```
Customer
  - customer_id (PK)
  - first_name
  - last_name
  - email
  - phone
  - street, city, postal_code, country (composite: address)
  - registered_at
```

SQL:
```sql
CREATE TABLE customers (
    customer_id   INTEGER       PRIMARY KEY,
    first_name    VARCHAR(50)   NOT NULL,
    last_name     VARCHAR(50)   NOT NULL,
    email         VARCHAR(100)  NOT NULL UNIQUE,
    phone         VARCHAR(20),
    street        VARCHAR(100)  NOT NULL,
    city          VARCHAR(50)   NOT NULL,
    postal_code   VARCHAR(10)   NOT NULL,
    country       VARCHAR(50)   NOT NULL DEFAULT 'Finland',
    registered_at TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);
```

Notice how the composite attribute "address" was **decomposed** into individual columns (`street`, `city`, `postal_code`, `country`).

### 3.2 Rule 2: One-to-Many (1:N) Relationships → Foreign Key on the "Many" Side

**Rule:** Add the primary key of the "one" side as a foreign key column in the "many" side table.

**Example: Category (1) → Product (N)**

The Product table gets a `category_id` column that references `categories.category_id`:

```sql
CREATE TABLE products (
    product_id     INTEGER        PRIMARY KEY,
    name           VARCHAR(100)   NOT NULL,
    description    TEXT,
    price          NUMERIC(10,2)  NOT NULL CHECK (price > 0),
    weight_kg      NUMERIC(6,2),
    stock_quantity INTEGER        NOT NULL DEFAULT 0
                                  CHECK (stock_quantity >= 0),
    created_at     TIMESTAMPTZ    NOT NULL DEFAULT NOW(),
    category_id    INTEGER        NOT NULL
                                  REFERENCES categories(category_id)
);
```

**Why on the "many" side?** Because each product belongs to ONE category — you can store that single reference in the product row. If you tried to store it on the Category side, you'd need to store multiple product IDs per category row, violating atomicity.

**Participation and NULLability:**
- Mandatory participation (every product MUST have a category) → `NOT NULL` on the FK column
- Optional participation (a product MAY have no category) → allow NULL on the FK column

### 3.3 Rule 3: Many-to-Many (M:N) → Junction Table

**Rule:** Create a new table (junction/associative table) containing the primary keys of both participating entities as foreign keys. The primary key of the junction table is typically the composite of both foreign keys.

**Example: Order (M) ↔ Product (N) resolved via OrderItem**

```sql
CREATE TABLE order_items (
    order_id   INTEGER      NOT NULL REFERENCES orders(order_id),
    product_id INTEGER      NOT NULL REFERENCES products(product_id),
    quantity   INTEGER      NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10,2) NOT NULL CHECK (unit_price > 0),
    PRIMARY KEY (order_id, product_id)
);
```

**Composite PK vs Surrogate PK:**

Option A — Composite primary key (shown above):
- `PRIMARY KEY (order_id, product_id)`
- Pros: natural, no extra column, prevents duplicate product lines per order
- Cons: composite keys are slightly more complex in JOIN queries

Option B — Surrogate primary key:
```sql
CREATE TABLE order_items (
    order_item_id SERIAL       PRIMARY KEY,
    order_id      INTEGER      NOT NULL REFERENCES orders(order_id),
    product_id    INTEGER      NOT NULL REFERENCES products(product_id),
    quantity      INTEGER      NOT NULL CHECK (quantity > 0),
    unit_price    NUMERIC(10,2) NOT NULL CHECK (unit_price > 0),
    UNIQUE (order_id, product_id)
);
```
- Pros: simpler references from other tables, single-column PK
- Cons: extra column, must add UNIQUE constraint separately

For TrailShop we'll use the composite key approach — it's cleaner for this use case.

### 3.4 Rule 4: One-to-One (1:1) → Foreign Key Placement Decision

**Rule:** Add a foreign key to one of the two tables. But which one?

Decision criteria:
- **If one side has mandatory participation and the other optional:** Put the FK on the mandatory side (it will always have a value).
- **If both sides are mandatory:** Either side works; choose the side that makes queries more natural.
- **If both sides are optional:** Put the FK on the side that is more likely to have the value. Mark the FK column as NULL-able.
- **Alternative:** Merge both entities into one table if they always exist together.

**Example:** Suppose each Customer has an optional LoyaltyProfile:

```sql
CREATE TABLE loyalty_profiles (
    profile_id     INTEGER     PRIMARY KEY,
    customer_id    INTEGER     NOT NULL UNIQUE
                               REFERENCES customers(customer_id),
    points_balance INTEGER     NOT NULL DEFAULT 0,
    tier           VARCHAR(20) NOT NULL DEFAULT 'bronze'
);
```

The FK (`customer_id`) is in `loyalty_profiles` with a UNIQUE constraint — this enforces the 1:1 cardinality. The customer doesn't need to have a profile (optional participation on the Customer side).

### 3.5 Rule 5: Weak Entities → Table with Mandatory FK (Part of PK)

**Rule:** Create a table for the weak entity. Include the owner entity's primary key as both a foreign key AND part of the composite primary key.

**Example: OrderItem (weak entity owned by Order)**

```sql
CREATE TABLE order_items (
    order_id   INTEGER       NOT NULL REFERENCES orders(order_id)
                             ON DELETE CASCADE,
    product_id INTEGER       NOT NULL REFERENCES products(product_id),
    quantity   INTEGER       NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10,2) NOT NULL CHECK (unit_price > 0),
    PRIMARY KEY (order_id, product_id)
);
```

Key aspects:
- `order_id` is both a FK to `orders` and part of the composite PK
- `ON DELETE CASCADE` — if an order is deleted, its items are automatically deleted (because the weak entity can't exist without its owner)
- The FK to `orders` is `NOT NULL` — mandatory participation

### 3.6 Rule 6: Multivalued Attributes → Separate Table

**Rule:** A multivalued attribute cannot be stored in a single column (violates first normal form). Create a separate table with a foreign key back to the original entity.

**Example:** If products can have multiple tags:

ER: Product has multivalued attribute `tags`

SQL:
```sql
CREATE TABLE product_tags (
    product_id INTEGER     NOT NULL REFERENCES products(product_id)
                           ON DELETE CASCADE,
    tag        VARCHAR(30) NOT NULL,
    PRIMARY KEY (product_id, tag)
);
```

Each tag gets its own row. The composite key `(product_id, tag)` prevents duplicate tags on the same product.

Another example — multiple phone numbers for a customer:

```sql
CREATE TABLE customer_phones (
    customer_id  INTEGER     NOT NULL REFERENCES customers(customer_id)
                             ON DELETE CASCADE,
    phone_type   VARCHAR(10) NOT NULL CHECK (phone_type IN ('home', 'work', 'mobile')),
    phone_number VARCHAR(20) NOT NULL,
    PRIMARY KEY (customer_id, phone_type)
);
```

### 3.7 Rule 7: Composite Attributes → Flattening

**Rule:** Decompose composite attributes into their individual components as separate columns.

We already saw this with the address attribute in Section 3.1. Here's the general pattern:

ER:
```
full_name (composite)
├── first_name
└── last_name
```

SQL: Two columns: `first_name VARCHAR(50)`, `last_name VARCHAR(50)`

ER:
```
address (composite)
├── street
├── city
├── postal_code
└── country
```

SQL: Four columns: `street`, `city`, `postal_code`, `country`

**When NOT to decompose:** If you never need to query or sort by individual parts, you might keep it as one column. But this is rare — it's almost always better to decompose.

### 3.8 Transformation Rules Summary

| ER Construct | Relational Mapping |
|---|---|
| Strong entity | Table with own PK |
| Weak entity | Table with composite PK (includes owner's PK as FK) |
| 1:N relationship | FK on the "many" side |
| M:N relationship | Junction table with FKs to both sides |
| 1:1 relationship | FK on one side (with UNIQUE constraint) |
| Multivalued attribute | Separate table with FK |
| Composite attribute | Decompose into individual columns |
| Derived attribute | Compute in queries; don't store |
| Key attribute | PRIMARY KEY |
| Optional participation | FK column allows NULL |
| Mandatory participation | FK column is NOT NULL |

---

## 4. PostgreSQL Data Types — Comprehensive Catalog

Choosing the right data type matters. It affects storage size, query performance, data validation, and what operations you can perform. Here is a comprehensive reference of PostgreSQL data types.

PostgreSQL documentation: https://www.postgresql.org/docs/current/datatype.html

### 4.1 Numeric Types

| Type | Size | Range | Use When |
|---|---|---|---|
| `SMALLINT` | 2 bytes | -32,768 to 32,767 | Small counters, ages, quantities under ~30K |
| `INTEGER` | 4 bytes | -2,147,483,648 to 2,147,483,647 | Most IDs, counts, general integers |
| `BIGINT` | 8 bytes | ±9.2 × 10¹⁸ | Very large counters, timestamps-as-integers |
| `NUMERIC(p,s)` | variable | up to 131,072 digits before decimal, 16,383 after | Exact precision: money, measurements |
| `REAL` | 4 bytes | ~6 decimal digits precision | Scientific data where slight imprecision is OK |
| `DOUBLE PRECISION` | 8 bytes | ~15 decimal digits precision | Scientific data, coordinates |
| `SERIAL` | 4 bytes | 1 to 2,147,483,647 | Auto-incrementing integer PK |
| `BIGSERIAL` | 8 bytes | 1 to 9.2 × 10¹⁸ | Auto-incrementing PK for huge tables |
| `SMALLSERIAL` | 2 bytes | 1 to 32,767 | Auto-incrementing PK for small lookup tables |

**For TrailShop:**
- `product_id`, `category_id`, `customer_id`, `order_id` → `INTEGER` or `SERIAL`
- `price`, `unit_price` → `NUMERIC(10,2)` (exact decimal, never use REAL/FLOAT for money!)
- `stock_quantity`, `quantity` → `INTEGER`

**Why never REAL or DOUBLE PRECISION for money?**
```sql
SELECT 0.1::REAL + 0.2::REAL;
-- Result: 0.30000001192092896 (not exactly 0.3!)

SELECT 0.1::NUMERIC + 0.2::NUMERIC;
-- Result: 0.3 (exact)
```

### 4.2 Character Types

| Type | Description | Use When |
|---|---|---|
| `VARCHAR(n)` | Variable-length with limit | Names, emails, most text with a natural upper bound |
| `CHAR(n)` | Fixed-length, padded with spaces | Fixed codes (ISO country: `CHAR(2)`, currency: `CHAR(3)`) |
| `TEXT` | Variable-length, unlimited | Descriptions, comments, long text with no practical limit |

In PostgreSQL, `VARCHAR` without a length limit and `TEXT` are functionally identical in performance. The difference is only in validation — `VARCHAR(100)` rejects values longer than 100 characters.

**For TrailShop:**
- `name` → `VARCHAR(100)`
- `email` → `VARCHAR(100)`
- `description` → `TEXT`
- `country` → `VARCHAR(50)` (or `CHAR(2)` if using ISO codes)
- `postal_code` → `VARCHAR(10)`

### 4.3 Date and Time Types

| Type | Size | Description | Example |
|---|---|---|---|
| `DATE` | 4 bytes | Date only (no time) | `'2026-09-15'` |
| `TIME` | 8 bytes | Time only (no date) | `'14:30:00'` |
| `TIMESTAMP` | 8 bytes | Date + time, no timezone | `'2026-09-15 14:30:00'` |
| `TIMESTAMPTZ` | 8 bytes | Date + time with timezone | `'2026-09-15 14:30:00+03'` |
| `INTERVAL` | 16 bytes | Duration / time span | `'2 hours 30 minutes'`, `'1 year 3 months'` |

**Best practice:** Almost always use `TIMESTAMPTZ` (timestamp with time zone) instead of `TIMESTAMP`. PostgreSQL stores `TIMESTAMPTZ` in UTC internally and converts to the session timezone on display. This avoids timezone bugs when users are in different zones.

**Interval examples:**
```sql
SELECT NOW() + INTERVAL '30 days';          -- 30 days from now
SELECT age('2026-09-15', '2000-03-21');     -- returns interval '26 years 5 months 25 days'
SELECT INTERVAL '2 hours' + INTERVAL '45 minutes';  -- '02:45:00'
```

**For TrailShop:**
- `order_date` → `TIMESTAMPTZ`
- `registered_at` → `TIMESTAMPTZ`
- `created_at` → `TIMESTAMPTZ`

### 4.4 Boolean Type

| Type | Size | Values |
|---|---|---|
| `BOOLEAN` | 1 byte | `TRUE`, `FALSE`, `NULL` |

Accepted literals: `TRUE`/`FALSE`, `'t'`/`'f'`, `'yes'`/`'no'`, `'on'`/`'off'`, `1`/`0`.

```sql
ALTER TABLE products ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT TRUE;
```

### 4.5 UUID Type

| Type | Size | Description |
|---|---|---|
| `UUID` | 16 bytes | Universally unique identifier (128-bit) |

```sql
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

CREATE TABLE example (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid()
);
```

UUIDs are useful when:
- You need globally unique IDs across systems (distributed databases, APIs)
- You don't want sequential IDs to reveal business information (how many customers you have)

### 4.6 Enumerated Types

PostgreSQL lets you define custom types with a fixed set of allowed values:

```sql
CREATE TYPE order_status AS ENUM ('pending', 'processing', 'shipped', 'delivered', 'cancelled');

CREATE TABLE orders (
    order_id   SERIAL        PRIMARY KEY,
    status     order_status  NOT NULL DEFAULT 'pending'
);
```

Pros: type-safe, self-documenting, compact storage
Cons: adding new values requires `ALTER TYPE`, harder to manage in migrations

**Alternative:** Use `VARCHAR` with a `CHECK` constraint for flexibility:
```sql
status VARCHAR(20) NOT NULL DEFAULT 'pending'
       CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))
```

### 4.7 Array Types

PostgreSQL supports arrays natively:

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name       VARCHAR(100) NOT NULL,
    tags       TEXT[]
);

INSERT INTO products (name, tags)
VALUES ('Alpine Pro Hiking Boots', ARRAY['waterproof', 'durable', 'bestseller']);

SELECT * FROM products WHERE 'waterproof' = ANY(tags);
```

Arrays are useful for simple lists that don't need their own table. But if you need to query, sort, or join on the array elements frequently, a separate table (Rule 6 from Section 3) is better.

### 4.8 Network Address Types

| Type | Size | Description |
|---|---|---|
| `CIDR` | 7 or 19 bytes | IPv4 or IPv6 network address |
| `INET` | 7 or 19 bytes | IPv4 or IPv6 host address (with optional subnet) |
| `MACADDR` | 6 bytes | MAC address |

```sql
CREATE TABLE login_log (
    id         SERIAL PRIMARY KEY,
    user_id    INTEGER NOT NULL,
    ip_address INET NOT NULL,
    logged_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

These types validate the format and support network-specific operations (containment, subnet checks).

### 4.9 JSON Types

| Type | Description |
|---|---|
| `JSON` | Stores JSON text (validated on input, stored as text) |
| `JSONB` | Stores JSON in binary format (faster queries, supports indexing) |

```sql
CREATE TABLE product_metadata (
    product_id INTEGER PRIMARY KEY REFERENCES products(product_id),
    metadata   JSONB NOT NULL DEFAULT '{}'
);

INSERT INTO product_metadata (product_id, metadata)
VALUES (101, '{"color": "brown", "waterproof": true, "materials": ["leather", "gore-tex"]}');

SELECT metadata->>'color' FROM product_metadata WHERE product_id = 101;
```

Use `JSONB` for semi-structured data that doesn't fit neatly into columns. But don't use it as an excuse to avoid proper schema design — if data has a consistent structure, use normal columns.

### 4.10 Data Type Quick-Reference for Common Scenarios

| What You're Storing | Recommended Type | Why |
|---|---|---|
| Auto-incrementing ID | `SERIAL` or `INTEGER` + sequence | Simple, compact |
| Money / prices | `NUMERIC(10,2)` | Exact decimal |
| Short text (names, codes) | `VARCHAR(n)` | Length limit prevents garbage |
| Long text (descriptions) | `TEXT` | No artificial limit |
| Dates without time | `DATE` | Smaller than timestamp |
| Timestamps | `TIMESTAMPTZ` | Timezone-aware |
| Yes/no flags | `BOOLEAN` | Self-documenting |
| Percentages | `NUMERIC(5,2)` | 0.00 to 100.00 |
| Email | `VARCHAR(254)` | RFC 5321 max length is 254 |
| Phone number | `VARCHAR(20)` | Phones have +, spaces, dashes — never use INTEGER |
| Postal code | `VARCHAR(10)` | Leading zeros matter (e.g., "01234") |
| Weight, dimensions | `NUMERIC(8,2)` | Exact decimal |
| IP address | `INET` | Built-in validation |
| Flexible key-value data | `JSONB` | Queryable semi-structured data |

---

## 5. Constraints — Full Coverage

Constraints are rules that PostgreSQL enforces on your data. They guarantee that only valid data enters the database. Chapter 15 of the textbook covers SQL DDL including constraints.

PostgreSQL documentation: https://www.postgresql.org/docs/current/ddl-constraints.html

### 5.1 NOT NULL

Ensures a column cannot contain NULL.

```sql
CREATE TABLE customers (
    customer_id INTEGER PRIMARY KEY,
    first_name  VARCHAR(50) NOT NULL,   -- mandatory
    last_name   VARCHAR(50) NOT NULL,   -- mandatory
    phone       VARCHAR(20)             -- optional (NULL allowed)
);
```

When to use: Any column where a missing value would be meaningless or violate a business rule.

### 5.2 UNIQUE

Ensures no two rows have the same value in the column (NULLs are allowed and don't count as duplicates).

```sql
CREATE TABLE customers (
    customer_id INTEGER PRIMARY KEY,
    email       VARCHAR(100) NOT NULL UNIQUE
);
```

A `UNIQUE` constraint can span multiple columns:
```sql
CREATE TABLE product_tags (
    product_id INTEGER NOT NULL,
    tag        VARCHAR(30) NOT NULL,
    UNIQUE (product_id, tag)
);
```

### 5.3 PRIMARY KEY

Combines `NOT NULL` and `UNIQUE`. Identifies each row. One per table.

```sql
CREATE TABLE categories (
    category_id INTEGER PRIMARY KEY,
    category_name VARCHAR(50) NOT NULL UNIQUE
);
```

Composite primary key:
```sql
CREATE TABLE order_items (
    order_id   INTEGER NOT NULL REFERENCES orders(order_id),
    product_id INTEGER NOT NULL REFERENCES products(product_id),
    PRIMARY KEY (order_id, product_id)
);
```

### 5.4 FOREIGN KEY (REFERENCES)

Links a column to the primary key (or unique column) of another table. Enforces referential integrity.

**Inline syntax:**
```sql
CREATE TABLE products (
    product_id  INTEGER PRIMARY KEY,
    category_id INTEGER NOT NULL REFERENCES categories(category_id)
);
```

**Table-level syntax (required for composite FKs):**
```sql
CREATE TABLE order_items (
    order_id   INTEGER NOT NULL,
    product_id INTEGER NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

### 5.5 CHECK

Enforces a condition that must be true for every row.

```sql
CREATE TABLE products (
    product_id     INTEGER PRIMARY KEY,
    price          NUMERIC(10,2) NOT NULL CHECK (price > 0),
    stock_quantity INTEGER NOT NULL CHECK (stock_quantity >= 0),
    weight_kg      NUMERIC(6,2) CHECK (weight_kg > 0)
);
```

Multi-column CHECK:
```sql
CREATE TABLE events (
    event_id   INTEGER PRIMARY KEY,
    start_date DATE NOT NULL,
    end_date   DATE NOT NULL,
    CHECK (end_date >= start_date)
);
```

Named CHECK constraint (useful for error messages):
```sql
CONSTRAINT positive_price CHECK (price > 0)
```

### 5.6 DEFAULT

Provides a value when no value is specified during INSERT.

```sql
CREATE TABLE products (
    product_id     SERIAL PRIMARY KEY,
    stock_quantity INTEGER NOT NULL DEFAULT 0,
    is_active      BOOLEAN NOT NULL DEFAULT TRUE,
    created_at     TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

`DEFAULT` is not a constraint in the strict sense — it doesn't reject data. It just fills in a value when one isn't provided.

### 5.7 EXCLUDE (Advanced)

Prevents overlapping ranges or conflicting values. Requires the `btree_gist` extension.

```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;

CREATE TABLE room_bookings (
    room_id    INTEGER NOT NULL,
    start_time TIMESTAMPTZ NOT NULL,
    end_time   TIMESTAMPTZ NOT NULL,
    EXCLUDE USING gist (
        room_id WITH =,
        tstzrange(start_time, end_time) WITH &&
    )
);
```

This prevents double-booking a room — no two bookings for the same room can have overlapping time ranges.

### 5.8 Constraints Summary Table

| Constraint | Purpose | Syntax Example |
|---|---|---|
| `NOT NULL` | Prevent missing values | `name VARCHAR(50) NOT NULL` |
| `UNIQUE` | Prevent duplicate values | `email VARCHAR(100) UNIQUE` |
| `PRIMARY KEY` | Unique row identifier (NOT NULL + UNIQUE) | `id INTEGER PRIMARY KEY` |
| `FOREIGN KEY` | Referential integrity | `REFERENCES other_table(pk_col)` |
| `CHECK` | Custom boolean condition | `CHECK (price > 0)` |
| `DEFAULT` | Auto-fill missing values | `DEFAULT NOW()` |
| `EXCLUDE` | Prevent overlapping/conflicting values | `EXCLUDE USING gist (...)` |

---

## 6. Foreign Key Actions — Expanded

When a referenced row is updated or deleted, what should happen to the rows that reference it? PostgreSQL provides five options.

### 6.1 The Five Actions

| Action | On DELETE behavior | On UPDATE behavior |
|---|---|---|
| `NO ACTION` (default) | Raise error if any child rows exist | Raise error if any child rows reference old value |
| `RESTRICT` | Same as NO ACTION but checked immediately (not deferrable) | Same |
| `CASCADE` | Delete all child rows automatically | Update FK values in child rows automatically |
| `SET NULL` | Set FK column(s) to NULL | Set FK column(s) to NULL |
| `SET DEFAULT` | Set FK column(s) to their DEFAULT value | Set FK column(s) to their DEFAULT value |

### 6.2 Syntax

```sql
CREATE TABLE order_items (
    order_id   INTEGER NOT NULL
               REFERENCES orders(order_id)
               ON DELETE CASCADE
               ON UPDATE CASCADE,
    product_id INTEGER NOT NULL
               REFERENCES products(product_id)
               ON DELETE RESTRICT
               ON UPDATE CASCADE
);
```

### 6.3 When to Use Each — Decision Guide

**CASCADE** — Use when child rows are meaningless without the parent.
- Delete an order → delete its order items (they're part of the order)
- Delete a customer → delete their orders? Maybe, maybe not. Consider soft-delete instead.

**RESTRICT / NO ACTION** — Use when deletion should be blocked if children exist.
- Delete a category → should fail if products still reference it (force the user to reassign products first)
- Delete a product → should fail if it appears in existing order items (historical data matters)

**SET NULL** — Use when the child should survive but lose its link.
- Delete a manager (employee) → set `manager_id` to NULL for their reports (they now have no manager)
- The FK column must allow NULL for this to work

**SET DEFAULT** — Use when a sensible default exists.
- Delete a category → set products to a "General" category (whose ID is the default)
- Rarely used in practice

### 6.4 Decision Flowchart

```
Parent row is being deleted. What happens to child rows?
│
├─ Are child rows meaningless without the parent?
│  YES → CASCADE
│  NO  ↓
│
├─ Should deletion be blocked if children exist?
│  YES → RESTRICT
│  NO  ↓
│
├─ Should children survive with no link?
│  YES → SET NULL  (FK column must allow NULL)
│  NO  ↓
│
├─ Is there a meaningful default value?
│  YES → SET DEFAULT
│  NO  → RESTRICT (safest default)
```

### 6.5 TrailShop FK Actions

| FK Relationship | ON DELETE | ON UPDATE | Reasoning |
|---|---|---|---|
| `products.category_id` → `categories` | RESTRICT | CASCADE | Don't delete a category that has products |
| `orders.customer_id` → `customers` | RESTRICT | CASCADE | Don't delete a customer with orders (consider soft-delete) |
| `order_items.order_id` → `orders` | CASCADE | CASCADE | Order items are part of the order |
| `order_items.product_id` → `products` | RESTRICT | CASCADE | Don't delete a product that appears in order history |

---

## 7. Design Anomalies — Brief Introduction

When a database schema is poorly designed, certain operations cause problems. Chapter 10 (Watt & Eng) introduces three types of **anomalies**:

### 7.1 Insertion Anomaly

You cannot insert certain data without inserting other unrelated data.

**Example:** If products and categories are stored in a single table:

| product_id | name | price | category_name | category_description |
|---|---|---|---|---|
| 101 | Alpine Pro Hiking Boots | 189.50 | Footwear | Shoes and boots |
| 102 | TrailMaster X4 Tent | 249.99 | Camping | Tents and sleeping gear |

Problem: You can't add a new category ("Cycling") without also adding a product. The category data requires a product row to exist.

### 7.2 Update Anomaly

Updating one fact requires updating multiple rows, risking inconsistency.

**Example:** If you rename "Camping" to "Camping & Outdoors," you must update *every* row that has `category_name = 'Camping'`. Miss one, and you have inconsistent data.

### 7.3 Deletion Anomaly

Deleting a row inadvertently removes unrelated data.

**Example:** If "TrailMaster X4 Tent" is the only product in "Camping" and you delete it, the entire "Camping" category information disappears — even though the category should still exist.

### 7.4 The Solution (Preview)

The solution to anomalies is **normalization** — decomposing tables to eliminate redundancy. We separated Category from Product precisely to avoid these anomalies. Normalization is covered in depth in Week 45.

---

## 8. Naming Conventions

Consistent naming makes your schema readable, maintainable, and less error-prone. Here's a comprehensive guide.

### 8.1 General Rules

| Rule | Convention | Example |
|---|---|---|
| Case | `snake_case` for everything | `order_items`, `first_name` |
| Table names | Plural nouns | `products`, `customers`, `order_items` |
| Column names | Singular descriptive | `first_name`, `order_date`, `unit_price` |
| Primary keys | `<singular_table_name>_id` | `product_id`, `customer_id` |
| Foreign keys | Same name as the PK they reference | `products.category_id` → `categories.category_id` |
| Boolean columns | Prefix with `is_`, `has_`, `can_` | `is_active`, `has_shipped` |
| Timestamps | Suffix with `_at` or `_on` | `created_at`, `shipped_on` |
| Constraints | `<table>_<column(s)>_<type>` | `products_price_check`, `customers_email_key` |

### 8.2 Why snake_case?

PostgreSQL folds unquoted identifiers to lowercase. If you write `CREATE TABLE OrderItems`, PostgreSQL stores it as `orderitems`. To preserve mixed case, you'd need quotes: `"OrderItems"` — which you then must use *every time* you reference it. This is tedious and error-prone.

`snake_case` avoids the problem entirely: `order_items` is exactly what PostgreSQL stores.

### 8.3 Singular vs Plural Table Names

This is a debate in the database community. We use **plural** because:
- A table contains *many* rows: the `customers` table holds many customers
- SQL reads naturally: `SELECT * FROM customers` ("select from the customers table")
- PostgreSQL system catalogs use plural (e.g., `pg_tables`, `pg_indexes`)

Some teams prefer singular. The most important thing is **consistency** — pick one and stick with it.

### 8.4 Reserved Words

Avoid using PostgreSQL reserved words as identifiers. Common traps:

| Tempting Name | Problem | Better Alternative |
|---|---|---|
| `user` | Reserved word | `app_user`, `customer` |
| `order` | Reserved word | `orders` (plural avoids it) |
| `name` | Not reserved, but very generic | `product_name`, `category_name` |
| `date` | Not reserved, but generic | `order_date`, `created_at` |
| `type` | Not reserved, but generic | `product_type`, `account_type` |
| `status` | Not reserved, but generic | `order_status` (or just `status` if unambiguous) |

Full list: https://www.postgresql.org/docs/current/sql-keywords-appendix.html

---

## 9. Surrogate vs Natural Keys

### 9.1 Definitions

**Natural Key:** A column (or columns) that has real-world meaning and naturally identifies each row.
- Email address for a customer
- ISBN for a book
- Social security number for a person
- Country code (ISO 3166) for a country

**Surrogate Key:** An artificial, system-generated value with no business meaning.
- Auto-incrementing integer (`SERIAL`)
- UUID (`gen_random_uuid()`)

### 9.2 Comparison

| Aspect | Natural Key | Surrogate Key |
|---|---|---|
| **Meaning** | Has business meaning | No business meaning |
| **Stability** | May change (email, name) | Never changes |
| **Size** | Variable (VARCHAR, composite) | Fixed (INTEGER = 4 bytes) |
| **JOIN performance** | Slower with wide keys | Faster with INTEGER |
| **Exposure** | Can reveal business data | Sequential integers reveal count |
| **External references** | Already used by the real world | Must be communicated separately |
| **Availability** | May not exist for all entities | Always available |
| **Uniqueness guarantee** | Depends on the real world | Guaranteed by the DBMS |

### 9.3 When to Use Which

**Use surrogate keys when:**
- There's no obvious natural key
- The natural key is long, composite, or might change
- You need consistent JOIN performance
- You're building an API (expose surrogates, not business data)

**Use natural keys when:**
- The natural key is short, stable, and universally recognized (ISO codes, currency codes)
- You're building a lookup/reference table

**TrailShop approach:** We use surrogate keys (`SERIAL` integers) for main entities and add `UNIQUE` constraints on natural identifiers where they exist (e.g., `email` for customers, `category_name` for categories).

---

## 10. Complete TrailShop Schema

Here are the complete `CREATE TABLE` statements for all five TrailShop tables, incorporating every concept from this chapter.

### 10.1 Creation Order

Tables must be created in dependency order — you can't reference a table that doesn't exist yet:

1. `categories` (no FK dependencies)
2. `customers` (no FK dependencies)
3. `products` (depends on `categories`)
4. `orders` (depends on `customers`)
5. `order_items` (depends on `orders` and `products`)

### 10.2 Categories

```sql
CREATE TABLE categories (
    category_id   SERIAL       PRIMARY KEY,
    category_name VARCHAR(50)  NOT NULL UNIQUE,
    description   TEXT
);
```

- `SERIAL` generates auto-incrementing IDs (surrogate key)
- `category_name` is `NOT NULL UNIQUE` — serves as a natural identifier
- `description` is optional (`NULL` allowed)

### 10.3 Customers

```sql
CREATE TABLE customers (
    customer_id   SERIAL        PRIMARY KEY,
    first_name    VARCHAR(50)   NOT NULL,
    last_name     VARCHAR(50)   NOT NULL,
    email         VARCHAR(254)  NOT NULL UNIQUE,
    phone         VARCHAR(20),
    street        VARCHAR(100)  NOT NULL,
    city          VARCHAR(50)   NOT NULL,
    postal_code   VARCHAR(10)   NOT NULL,
    country       VARCHAR(50)   NOT NULL DEFAULT 'Finland',
    registered_at TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);
```

- `email` has `UNIQUE` — natural identifier; `VARCHAR(254)` per RFC 5321
- `phone` is optional — not all customers provide one
- Address is decomposed from the composite attribute into four columns
- `country` defaults to `'Finland'` — most TrailShop customers are Finnish
- `registered_at` defaults to the current timestamp

### 10.4 Products

```sql
CREATE TABLE products (
    product_id     SERIAL         PRIMARY KEY,
    name           VARCHAR(100)   NOT NULL,
    description    TEXT,
    price          NUMERIC(10,2)  NOT NULL
                   CONSTRAINT products_price_positive CHECK (price > 0),
    weight_kg      NUMERIC(6,2)
                   CONSTRAINT products_weight_positive CHECK (weight_kg > 0),
    stock_quantity INTEGER        NOT NULL DEFAULT 0
                   CONSTRAINT products_stock_non_negative CHECK (stock_quantity >= 0),
    created_at     TIMESTAMPTZ    NOT NULL DEFAULT NOW(),
    category_id    INTEGER        NOT NULL
                   REFERENCES categories(category_id)
                   ON DELETE RESTRICT
                   ON UPDATE CASCADE
);
```

- `price` uses `NUMERIC(10,2)` — never float for money
- Named CHECK constraints for clear error messages
- `weight_kg` is optional but must be positive if provided
- `category_id` FK with `RESTRICT` on delete — can't delete a category that has products
- `ON UPDATE CASCADE` — if a category's ID changes (rare), products update automatically

### 10.5 Orders

```sql
CREATE TABLE orders (
    order_id    SERIAL       PRIMARY KEY,
    customer_id INTEGER      NOT NULL
                REFERENCES customers(customer_id)
                ON DELETE RESTRICT
                ON UPDATE CASCADE,
    order_date  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    status      VARCHAR(20)  NOT NULL DEFAULT 'pending'
                CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
    shipping_street      VARCHAR(100),
    shipping_city        VARCHAR(50),
    shipping_postal_code VARCHAR(10),
    shipping_country     VARCHAR(50)
);
```

- `customer_id` FK with `RESTRICT` — can't delete a customer who has orders
- `status` uses a CHECK constraint with allowed values (alternative: ENUM type)
- Shipping address columns are optional — the customer's address is the default
- `order_date` defaults to NOW()

### 10.6 Order Items

```sql
CREATE TABLE order_items (
    order_id   INTEGER        NOT NULL
               REFERENCES orders(order_id)
               ON DELETE CASCADE
               ON UPDATE CASCADE,
    product_id INTEGER        NOT NULL
               REFERENCES products(product_id)
               ON DELETE RESTRICT
               ON UPDATE CASCADE,
    quantity   INTEGER        NOT NULL
               CHECK (quantity > 0),
    unit_price NUMERIC(10,2)  NOT NULL
               CHECK (unit_price > 0),
    PRIMARY KEY (order_id, product_id)
);
```

- Composite PK `(order_id, product_id)` — each product appears at most once per order
- `order_id` FK with `CASCADE` — delete an order, its items go too (weak entity behavior)
- `product_id` FK with `RESTRICT` — can't delete a product with order history
- `unit_price` is the price snapshot at order time — not derived from `products.price`

### 10.7 The Complete Schema at a Glance

```
categories
├── category_id (PK, SERIAL)
├── category_name (VARCHAR, NOT NULL, UNIQUE)
└── description (TEXT, nullable)

customers
├── customer_id (PK, SERIAL)
├── first_name (VARCHAR, NOT NULL)
├── last_name (VARCHAR, NOT NULL)
├── email (VARCHAR, NOT NULL, UNIQUE)
├── phone (VARCHAR, nullable)
├── street (VARCHAR, NOT NULL)
├── city (VARCHAR, NOT NULL)
├── postal_code (VARCHAR, NOT NULL)
├── country (VARCHAR, NOT NULL, DEFAULT 'Finland')
└── registered_at (TIMESTAMPTZ, NOT NULL, DEFAULT NOW())

products
├── product_id (PK, SERIAL)
├── name (VARCHAR, NOT NULL)
├── description (TEXT, nullable)
├── price (NUMERIC, NOT NULL, CHECK > 0)
├── weight_kg (NUMERIC, nullable, CHECK > 0)
├── stock_quantity (INTEGER, NOT NULL, DEFAULT 0, CHECK >= 0)
├── created_at (TIMESTAMPTZ, NOT NULL, DEFAULT NOW())
└── category_id (FK → categories, NOT NULL, ON DELETE RESTRICT)

orders
├── order_id (PK, SERIAL)
├── customer_id (FK → customers, NOT NULL, ON DELETE RESTRICT)
├── order_date (TIMESTAMPTZ, NOT NULL, DEFAULT NOW())
├── status (VARCHAR, NOT NULL, DEFAULT 'pending', CHECK IN (...))
├── shipping_street (VARCHAR, nullable)
├── shipping_city (VARCHAR, nullable)
├── shipping_postal_code (VARCHAR, nullable)
└── shipping_country (VARCHAR, nullable)

order_items
├── order_id (PK, FK → orders, ON DELETE CASCADE)
├── product_id (PK, FK → products, ON DELETE RESTRICT)
├── quantity (INTEGER, NOT NULL, CHECK > 0)
└── unit_price (NUMERIC, NOT NULL, CHECK > 0)
```

---

## Key Terms

| Term | Definition |
|---|---|
| **Database development lifecycle** | The phases from requirements through implementation and maintenance |
| **Logical design** | Transforming a conceptual model into a relational schema with types and constraints |
| **Transformation rule** | A systematic rule for converting an ER construct into relational tables |
| **Junction table** | A table that resolves an M:N relationship, containing FKs to both participating tables |
| **Data type** | The kind of values a column can hold (INTEGER, VARCHAR, NUMERIC, etc.) |
| **SERIAL** | An auto-incrementing integer type in PostgreSQL |
| **NUMERIC(p,s)** | Exact decimal type with p total digits and s decimal places |
| **TIMESTAMPTZ** | Timestamp with time zone — stores in UTC, displays in session timezone |
| **NOT NULL** | Constraint preventing NULL values in a column |
| **UNIQUE** | Constraint preventing duplicate values |
| **PRIMARY KEY** | NOT NULL + UNIQUE; identifies each row |
| **FOREIGN KEY** | Constraint linking a column to a PK/UNIQUE in another table |
| **CHECK** | Constraint enforcing a custom condition |
| **DEFAULT** | A value assigned when no value is specified on INSERT |
| **CASCADE** | FK action: automatically delete/update child rows |
| **RESTRICT** | FK action: block delete/update if child rows exist |
| **SET NULL** | FK action: set FK to NULL when parent is deleted |
| **Design anomaly** | A problem (insertion, update, deletion) caused by poor schema design |
| **Surrogate key** | An artificial, system-generated key with no business meaning |
| **Natural key** | A key with real-world meaning (email, ISBN, SSN) |
| **snake_case** | Naming convention using lowercase and underscores |

---

## Reading Assignments

**Required:**
- *Database Design*, 2nd Edition — Chapters 10 (ER Modelling to Relational Mapping), 13 (Database Development Process), and 15 (SQL DDL)
- PostgreSQL Docs: Data Types — https://www.postgresql.org/docs/current/datatype.html
- PostgreSQL Docs: Constraints — https://www.postgresql.org/docs/current/ddl-constraints.html

---

## Further Reading

- PostgreSQL Docs: CREATE TABLE — https://www.postgresql.org/docs/current/sql-createtable.html
- PostgreSQL Docs: Data Definition — https://www.postgresql.org/docs/current/ddl.html
- PostgreSQL Wiki: Don't Do This — https://wiki.postgresql.org/wiki/Don't_Do_This (common PostgreSQL pitfalls)
- *Database Design*, 2nd Edition — Chapter 10 for detailed discussion of anomalies and normalization preview

---

## Summary

You've completed the journey from blueprint to building plan. Starting with the ER diagram you created last week, you applied systematic transformation rules to convert every entity, relationship, and attribute into PostgreSQL tables with proper data types and constraints.

You now know how to map strong entities to tables, place foreign keys on the "many" side of 1:N relationships, create junction tables for M:N relationships, handle weak entities with composite keys, and decompose composite and multivalued attributes. You can choose appropriate data types from PostgreSQL's rich catalog — and you know to never use floating point for money.

You've defined constraints that enforce business rules at the database level: NOT NULL, UNIQUE, CHECK, FOREIGN KEY with appropriate actions. You've adopted naming conventions that keep the schema readable, and you understand the trade-offs between surrogate and natural keys.

Most importantly, you've written the complete `CREATE TABLE` statements for TrailShop's five core tables. The database is ready to be built.

**Next week:** You'll populate these tables with data and start writing SQL queries to retrieve, filter, sort, and aggregate information — bringing TrailShop's database to life.
