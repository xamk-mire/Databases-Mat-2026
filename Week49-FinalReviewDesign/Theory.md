# Week 49 — Final Review: Database Design

> **Exam review week.** No new concepts and no required work. Use this chapter and the optional exercises to prepare for the final exam. The TrailShop project was completed in Week 48.

## Chapter 13: "The Design Review"

The trail outside TrailShop is covered in the first real snow of winter. Inside, the whiteboard in the back office is full — ER diagrams, table schemas, arrows connecting foreign keys to primary keys, sticky notes marking design decisions made weeks ago. You step back and look at the whole picture.

This is the moment every database designer reaches: the review. You built TrailShop over Weeks 36–48 — from vague requirements through conceptual models, logical design, normalization, and finally real PostgreSQL tables. Now you can ask yourself: *Is it good? Is it correct? Would it survive contact with real users and real data?*

Today you are not learning new concepts. You are reviewing everything you have learned this semester, pulling it together into a unified design process, and developing the critical eye you need to evaluate your own work — and the work of your peers.

---

## Learning Objectives

After this lecture, you will be able to:

- Describe the complete database design pipeline from requirements to implementation
- Identify common mistakes at each stage of the design process
- Apply a quality checklist to evaluate any relational database design
- Review the TrailShop reference schema as a complete, production-ready example
- Provide constructive peer feedback on database designs
- Define and use all key design terms from the course

---

## 1 — The Complete Design Process

### 1.1 The Pipeline

Database design is not a single activity — it is a pipeline of stages, each feeding the next. Mistakes made early cascade through every subsequent stage, becoming harder and more expensive to fix.

The pipeline:

```
Requirements → ER Model → Logical Design → Normalization → Implementation
```

Every stage transforms one artifact into the next. Skip a stage, and you introduce risk. Rush a stage, and you carry errors forward.

### 1.2 Stage Reference Table

| Stage | What You Produce | Key Decisions |
|-------|-----------------|---------------|
| **Requirements** | Written requirements document | What data matters? What questions will the system answer? Who are the users? |
| **ER Modelling** | Entity-Relationship diagram | What are the entities? How are they related? What are the cardinalities? |
| **Logical Design** | Relational schema (tables, columns, keys) | How do entities map to tables? Where do foreign keys go? What junction tables are needed? |
| **Normalization** | Refined schema in 3NF (or higher) | Are there redundancies? Partial dependencies? Transitive dependencies? |
| **Implementation** | SQL DDL (CREATE TABLE statements) | What data types? What constraints? What defaults? What ON DELETE actions? |

*Reference: Watt & Eng, "Database Design — 2nd Edition", Chapters 1–13 cover this complete pipeline.*

### 1.3 How Mistakes Cascade

Consider what happens when you miss an entity in the requirements stage:

1. **Requirements**: You forget that TrailShop needs to track suppliers
2. **ER Model**: No supplier entity appears in the diagram
3. **Logical Design**: No supplier table, no foreign key from products to suppliers
4. **Normalization**: Nothing to normalize — the data simply does not exist
5. **Implementation**: The database cannot answer "Who supplies this product?"

Now you must go back to the ER model, add the entity and relationships, redesign the logical schema, check normalization again, and write new DDL. The later you discover the missing piece, the more work you redo.

This is why we spend time on requirements and ER modelling — not because they are exciting, but because they are cheap places to make corrections.

### 1.4 Design Is Iterative

Real design is never purely linear. You will:

- Discover missing requirements during ER modelling
- Find normalization problems that force you to restructure tables
- Realize during implementation that a data type choice does not support a required query

This is normal. The pipeline gives you a *direction*, not a rigid sequence. But you should always know which stage you are working in and what artifact you are producing.

---

## 2 — ER Modelling Review

### 2.1 Common Mistakes Checklist

When reviewing an ER diagram, look for these frequent problems:

| # | Mistake | Example |
|---|---------|---------|
| 1 | Missing entity | Forgetting "payments" when the system needs payment tracking |
| 2 | Wrong cardinality | Marking orders → customers as M:N when it should be N:1 |
| 3 | Missing attributes | An entity with only a name and ID — where are the other details? |
| 4 | Composite attributes not decomposed | "address" as a single attribute instead of street, city, postal_code |
| 5 | Multi-valued attributes not resolved | "phone_numbers" stored in one attribute |
| 6 | Derived attributes stored | Storing "age" when you have "birth_date" |
| 7 | Missing relationships | Entities that exist but have no connection to the rest of the model |
| 8 | Redundant relationships | Two paths between the same entities that represent the same fact |

### 2.2 Quick Reference: Entity Types

| Type | Description | Example |
|------|-------------|---------|
| **Strong entity** | Exists independently, has its own primary key | Customer, Product |
| **Weak entity** | Depends on a strong entity for identification | OrderItem (depends on Order) |

### 2.3 Quick Reference: Relationship Types

| Cardinality | Meaning | Example |
|-------------|---------|---------|
| **1:1** | One instance relates to exactly one instance | Person — Passport |
| **1:N** | One instance relates to many instances | Customer — Orders |
| **M:N** | Many instances relate to many instances | Products — Categories (if multi-category) |

### 2.4 Quick Reference: Participation

| Type | Meaning | Notation |
|------|---------|----------|
| **Total** | Every instance must participate | Double line — every order MUST have a customer |
| **Partial** | Some instances may not participate | Single line — a customer MAY have zero orders |

### 2.5 Cardinality Determination Process

When you are unsure about cardinality, ask these questions:

1. Pick one instance of Entity A. How many instances of Entity B can it relate to? (one or many?)
2. Pick one instance of Entity B. How many instances of Entity A can it relate to? (one or many?)
3. Combine the answers: one-one → 1:1, one-many → 1:N, many-many → M:N

Always test with concrete examples from your domain. "Can one customer have many orders? Yes. Can one order belong to many customers? No." → 1:N from customer to order.

### 2.6 TrailShop ER Quick Reference

The TrailShop model includes:

- **Categories** (strong entity): category_id, name, description
- **Products** (strong entity): product_id, name, description, price, stock_quantity, category_id
- **Customers** (strong entity): customer_id, first_name, last_name, email, created_at
- **Orders** (strong entity): order_id, customer_id, order_date, status, total_amount
- **OrderItems** (weak entity): order_id + product_id, quantity, unit_price
- **Payments** (strong entity): payment_id, order_id, amount, payment_method, payment_date

Key relationships:
- Category —1:N— Product
- Customer —1:N— Order
- Order —1:N— OrderItem
- Product —1:N— OrderItem (M:N between Order and Product, resolved via OrderItem)
- Order —1:1— Payment

---

## 3 — Relational Design Review

### 3.1 Mapping Rules Quick Reference

| ER Construct | Relational Mapping | Notes |
|--------------|-------------------|-------|
| Strong entity | Table with own PK | Each attribute becomes a column |
| Weak entity | Table with composite PK (own partial key + owner FK) | Foreign key to owner is part of PK |
| 1:1 relationship | FK in either table (prefer the total-participation side) | Add UNIQUE constraint on FK |
| 1:N relationship | FK in the "many" side table | FK references PK of the "one" side |
| M:N relationship | Junction table with two FKs forming composite PK | May include relationship attributes |
| Multi-valued attribute | Separate table with FK back to owner | Each value gets its own row |
| Composite attribute | Decompose into individual columns | street, city, postal_code instead of address |

*Reference: Watt & Eng, Ch. 9 — "Mapping ER Models to Relational Schemas".*

### 3.2 Data Type Selection Guide

Choose the most specific data type that correctly represents your domain:

| Data Scenario | Recommended Type | Why |
|---------------|-----------------|-----|
| Unique identifier (auto) | `INTEGER GENERATED ALWAYS AS IDENTITY` | Standard surrogate key, auto-incrementing |
| Short text (name, title) | `VARCHAR(100)` | Bounded length prevents abuse |
| Long text (description) | `TEXT` | No practical limit needed |
| Email address | `VARCHAR(255)` | RFC max is 254 characters |
| Money / price | `NUMERIC(10,2)` | Exact decimal — never use FLOAT for money |
| Quantity (integer) | `INTEGER` | Whole items only |
| Date only (birth date) | `DATE` | No time component needed |
| Date + time (order placed) | `TIMESTAMP` | Full precision |
| Date + time + timezone | `TIMESTAMPTZ` | For international applications |
| Boolean flag | `BOOLEAN` | true/false only |
| Status with fixed options | `VARCHAR(20)` with CHECK | Or use an enum type |
| Percentage | `NUMERIC(5,2)` with CHECK (0–100) | Exact decimal with range constraint |
| UUID identifier | `UUID` | For distributed systems or public-facing IDs |

### 3.3 Constraint Design Patterns

Constraints enforce business rules at the database level — they are your last line of defense against bad data.

**PRIMARY KEY**: Every table must have one. Identifies each row uniquely.

```sql
customer_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

**FOREIGN KEY**: Links tables together. Enforces referential integrity.

```sql
category_id INTEGER NOT NULL REFERENCES categories(category_id)
```

**NOT NULL**: The column must have a value. Use on every column that must always be filled.

```sql
email VARCHAR(255) NOT NULL
```

**UNIQUE**: No two rows can have the same value. Use for natural keys and business identifiers.

```sql
email VARCHAR(255) NOT NULL UNIQUE
```

**CHECK**: Validates that values meet a condition.

```sql
price NUMERIC(10,2) NOT NULL CHECK (price >= 0)
stock_quantity INTEGER NOT NULL CHECK (stock_quantity >= 0)
status VARCHAR(20) NOT NULL CHECK (status IN ('pending', 'shipped', 'delivered', 'cancelled'))
```

**DEFAULT**: Provides a value when none is specified.

```sql
created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
status VARCHAR(20) NOT NULL DEFAULT 'pending'
```

### 3.4 ON DELETE / ON UPDATE Actions

When a referenced row is deleted or updated, what should happen to the rows that reference it?

| Action | Behavior | When to Use |
|--------|----------|-------------|
| **CASCADE** | Delete/update referencing rows automatically | Child rows that have no meaning without the parent (order_items when order is deleted) |
| **RESTRICT** | Prevent the delete/update entirely | When deletion would lose important data (don't delete a customer who has orders) |
| **SET NULL** | Set the FK column to NULL | When the reference is optional (product's category is removed — product can exist without category) |
| **SET DEFAULT** | Set the FK column to its default value | When a sensible default exists (reassign to "uncategorized") |
| **NO ACTION** | Same as RESTRICT but checked at end of transaction | When you want deferred constraint checking |

Choose the action that matches your business rule. Ask: "If the parent row disappears, what should happen to this child row?"

---

## 4 — Normalization Review

### 4.1 First Normal Form (1NF)

**Rule**: Every column contains only atomic (indivisible) values; no repeating groups.

**Violation**:
```sql
-- BAD: multiple phone numbers in one column
INSERT INTO customers (name, phones) VALUES ('Anna', '040-111, 050-222');
```

**Fix**: Create a separate `customer_phones` table with one row per phone number.

### 4.2 Second Normal Form (2NF)

**Rule**: In 2NF (and also 1NF), every non-key column depends on the *entire* primary key, not just part of it.

**Violation**:
```sql
-- BAD: product_name depends only on product_id, not on the full PK (order_id, product_id)
CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    product_name VARCHAR(100),  -- partial dependency!
    PRIMARY KEY (order_id, product_id)
);
```

**Fix**: Remove `product_name` — it belongs in the `products` table.

### 4.3 Third Normal Form (3NF)

**Rule**: In 3NF (and also 2NF), no non-key column depends on another non-key column (no transitive dependencies).

**Violation**:
```sql
-- BAD: city depends on postal_code, which depends on the PK
CREATE TABLE customers (
    customer_id INTEGER PRIMARY KEY,
    postal_code VARCHAR(5),
    city VARCHAR(100)  -- transitive dependency: customer_id → postal_code → city
);
```

**Fix**: Create a `postal_codes` table with (postal_code, city), reference it via FK.

### 4.4 Boyce-Codd Normal Form (BCNF)

BCNF is a stricter version of 3NF. A table is in BCNF if every determinant is a candidate key. In practice, most tables that are in 3NF are also in BCNF. The exceptions involve tables with multiple overlapping candidate keys — rare in typical application databases.

For this course, achieving 3NF is sufficient.

*Reference: Watt & Eng, Ch. 11–12 — "Normalization".*

### 4.5 Denormalization

Sometimes you deliberately break normal forms for performance:

- **Storing calculated totals** (e.g., `total_amount` in orders) to avoid recalculating from order_items on every read
- **Duplicating a name** in a child table for fast display without joins
- **Adding a counter column** (e.g., `order_count` in customers) to avoid COUNT queries

Rules for denormalization:

1. Only denormalize after you have a normalized design
2. Only denormalize when you have measured a real performance problem
3. Document what you denormalized and why
4. Ensure application logic keeps the denormalized data in sync

### 4.6 Normalization Checklist

- [ ] Every column contains only one value (1NF)
- [ ] No repeating groups or arrays in columns (1NF)
- [ ] All non-key columns depend on the full primary key (2NF)
- [ ] No column depends on another non-key column (3NF)
- [ ] Any intentional denormalization is documented and justified
- [ ] Junction tables have correct composite primary keys

---

## 5 — Design Quality Checklist

Use this checklist to evaluate any database design — your own or a peer's.

### The 10-Point Checklist

| # | Check | Question to Ask |
|---|-------|-----------------|
| 1 | **Primary Keys** | Does every table have a clear, appropriate primary key? |
| 2 | **Foreign Keys** | Are all relationships enforced with foreign key constraints? |
| 3 | **NOT NULL** | Are required columns marked NOT NULL? |
| 4 | **CHECK Constraints** | Are business rules enforced at the database level? |
| 5 | **3NF Compliance** | Is the schema free of redundancy and transitive dependencies? |
| 6 | **Naming Conventions** | Are names consistent, lowercase, snake_case, and meaningful? |
| 7 | **Referential Integrity** | Can orphan rows exist? Are FK actions defined? |
| 8 | **ON DELETE Actions** | Is every FK's delete behavior explicitly chosen and appropriate? |
| 9 | **Data Types** | Is each column's type the most specific correct choice? |
| 10 | **ER Consistency** | Does the implementation match the ER diagram? |

### Expanded Explanations

**1. Primary Keys**: Prefer surrogate keys (`INTEGER GENERATED ALWAYS AS IDENTITY`) for main entities. Use composite keys only for junction tables. Every table needs exactly one PK — no exceptions.

**2. Foreign Keys**: Every relationship in your ER diagram must be enforced with a FK constraint. Without FK constraints, the database cannot prevent orphan rows. If you draw a line in the ER, there must be a FK in the schema.

**3. NOT NULL**: Default to NOT NULL for everything, then relax to nullable only when you have a specific reason. A column that allows NULL is a column that can have missing data — make sure that is intentional.

**4. CHECK Constraints**: Prices should not be negative. Quantities should not be negative. Status should be one of a defined set. If you can express a rule as a CHECK, do it — the database is the last line of defense.

**5. 3NF Compliance**: Look at every non-key column. Does it depend only on the primary key? Or does it depend on another non-key column? If the latter, you have a normalization violation.

**6. Naming Conventions**: Tables are plural nouns (`customers`, `orders`). Columns are singular (`first_name`, `email`). Everything is lowercase snake_case. Foreign key columns match the referenced table's PK name (`customer_id` in orders references `customer_id` in customers).

**7. Referential Integrity**: If `orders.customer_id` references `customers.customer_id`, can you insert an order for customer 999 when customer 999 does not exist? With a FK constraint, no. Without one, yes — and your data becomes inconsistent.

**8. ON DELETE Actions**: Think about each FK: what happens when the parent is deleted? Should the children be deleted too (CASCADE)? Should the delete be blocked (RESTRICT)? Should the FK be set to NULL? Choose deliberately for each relationship.

**9. Data Types**: Do not use TEXT when VARCHAR(100) is appropriate. Do not use FLOAT for money. Do not use VARCHAR for dates. The type should match the domain as precisely as possible.

**10. ER Consistency**: After implementation, compare your CREATE TABLE statements back to your ER diagram. Every entity should be a table. Every relationship should be a FK. Every attribute should be a column. If they do not match, one of them is wrong.

---

## 6 — Complete TrailShop Reference Schema

This is the full TrailShop schema as it should look after the complete design process. Study each table, its constraints, and its design decisions.

### 6.1 Categories

```sql
CREATE TABLE categories (
    category_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name        VARCHAR(100) NOT NULL UNIQUE,
    description TEXT
);
```

**Self-review**:
- Why is `name` UNIQUE? Because two categories should not have the same name.
- Why is `description` nullable? Because a category can exist without a description.
- Why no ON DELETE concerns? Nothing references this table's PK yet — products do, see below.

### 6.2 Products

```sql
CREATE TABLE products (
    product_id     INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name           VARCHAR(200) NOT NULL,
    description    TEXT,
    price          NUMERIC(10,2) NOT NULL CHECK (price >= 0),
    stock_quantity INTEGER NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    category_id    INTEGER REFERENCES categories(category_id) ON DELETE SET NULL,
    created_at     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

**Self-review**:
- Why `NUMERIC(10,2)` for price? Exact decimal arithmetic — never use FLOAT for money.
- Why `ON DELETE SET NULL` for category? If a category is removed, the product should still exist — just without a category.
- Why `CHECK (price >= 0)`? A negative price makes no business sense.
- Why is `category_id` nullable? Because ON DELETE SET NULL requires it, and a product without a category is acceptable.

### 6.3 Customers

```sql
CREATE TABLE customers (
    customer_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name  VARCHAR(100) NOT NULL,
    last_name   VARCHAR(100) NOT NULL,
    email       VARCHAR(255) NOT NULL UNIQUE,
    created_at  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

**Self-review**:
- Why is `email` UNIQUE? Each customer account must have a distinct email.
- Why separate `first_name` and `last_name`? To allow sorting, searching, and addressing by either part.
- Why NOT NULL on names? A customer without a name is not a valid customer.

### 6.4 Orders

```sql
CREATE TABLE orders (
    order_id     INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id  INTEGER NOT NULL REFERENCES customers(customer_id) ON DELETE RESTRICT,
    order_date   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status       VARCHAR(20) NOT NULL DEFAULT 'pending'
                 CHECK (status IN ('pending', 'shipped', 'delivered', 'cancelled')),
    total_amount NUMERIC(10,2) NOT NULL DEFAULT 0 CHECK (total_amount >= 0)
);
```

**Self-review**:
- Why `ON DELETE RESTRICT`? You should not delete a customer who has order history — that data is valuable.
- Why is `status` constrained with CHECK? To prevent invalid status values like "maybe" or "xyz".
- Why is `total_amount` denormalized here? For performance — avoids recalculating from order_items on every read. This is a deliberate denormalization.

### 6.5 Order Items

```sql
CREATE TABLE order_items (
    order_id   INTEGER NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id INTEGER NOT NULL REFERENCES products(product_id) ON DELETE RESTRICT,
    quantity   INTEGER NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10,2) NOT NULL CHECK (unit_price >= 0),
    PRIMARY KEY (order_id, product_id)
);
```

**Self-review**:
- Why composite PK? This is a junction table resolving the M:N between orders and products.
- Why `ON DELETE CASCADE` for order? If an order is deleted, its line items are meaningless alone.
- Why `ON DELETE RESTRICT` for product? Deleting a product that was once ordered would destroy history.
- Why store `unit_price` here? Because product prices change over time — we freeze the price at the moment of purchase.
- Why `CHECK (quantity > 0)`? An order item with zero or negative quantity is invalid.

### 6.6 Payments

```sql
CREATE TABLE payments (
    payment_id     INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id       INTEGER NOT NULL UNIQUE REFERENCES orders(order_id) ON DELETE CASCADE,
    amount         NUMERIC(10,2) NOT NULL CHECK (amount > 0),
    payment_method VARCHAR(50) NOT NULL
                   CHECK (payment_method IN ('credit_card', 'debit_card', 'bank_transfer', 'cash')),
    payment_date   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

**Self-review**:
- Why `UNIQUE` on `order_id`? Each order has at most one payment (1:1 relationship).
- Why `ON DELETE CASCADE`? If an order is deleted, the payment record goes with it.
- Why `CHECK (amount > 0)`? A payment of zero or negative makes no sense.
- Why constrain `payment_method`? To limit values to supported payment types.

---

## 7 — Peer Review Guidelines

### 7.1 Before You Start

Before reviewing a peer's design:

1. Read their requirements document first — understand what the database is supposed to do
2. Look at their ER diagram — understand the conceptual model
3. Only then look at the SQL implementation
4. Keep the requirements open while reviewing — everything should trace back to them

### 7.2 What to Look For

When reviewing, check each of these areas:

1. **Completeness**: Does the schema support all stated requirements?
2. **Key design**: Does every table have an appropriate primary key?
3. **Relationships**: Are all FK constraints present and correct?
4. **Constraints**: Are business rules enforced with CHECK, NOT NULL, UNIQUE?
5. **Normalization**: Is the schema in 3NF (or are deviations justified)?
6. **Naming**: Are conventions consistent throughout?
7. **Data types**: Are types appropriate for the data?
8. **ON DELETE**: Are referential actions chosen deliberately?

### 7.3 How to Give Feedback

Good feedback is:

- **Specific**: Point to the exact table, column, or constraint
- **Explanatory**: Say *why* something is a problem, not just *that* it is
- **Constructive**: Suggest an improvement, don't just criticize
- **Respectful**: Acknowledge good decisions alongside problems
- **Prioritized**: Focus on correctness issues before style preferences

### 7.4 Sample Review Comments

**Bad feedback**:
> "Your schema is wrong."

**Good feedback**:
> "In the `orders` table, `customer_id` has no FOREIGN KEY constraint. This means the database would allow orders referencing non-existent customers. Add: `REFERENCES customers(customer_id) ON DELETE RESTRICT`."

**Bad feedback**:
> "Use better names."

**Good feedback**:
> "The column `dt` in the `orders` table is unclear — consider renaming to `order_date` so the meaning is obvious without looking at documentation."

**Bad feedback**:
> "This is not normalized."

**Good feedback**:
> "The `products` table has `category_name` alongside `category_id`. Since `category_name` depends on `category_id` (not on `product_id`), this is a transitive dependency violating 3NF. Remove `category_name` and join to the `categories` table when you need it."

### 7.5 Peer Review Template

Use this structure when writing a peer review:

```
## Peer Review: [Project Name]
Reviewer: [Your name]
Date: [Date]

### Overall Impression
[1-2 sentences on the overall quality and completeness]

### Strengths
- [Good decision 1]
- [Good decision 2]

### Issues Found

#### Issue 1: [Title]
- **Location**: [Table/column]
- **Problem**: [What is wrong and why it matters]
- **Suggestion**: [How to fix it]

#### Issue 2: [Title]
...

### Minor Suggestions
- [Style/naming preferences — lower priority]

### Summary
[1-2 sentences final assessment]
```

---

## Key Terms

A master glossary of database design terms used throughout this course:

| Term | Definition |
|------|-----------|
| **Entity** | A real-world object or concept represented in the database (e.g., Customer, Product) |
| **Attribute** | A property or characteristic of an entity (e.g., name, email, price) |
| **Relation** | A table in the relational model — a set of tuples with the same structure |
| **Tuple** | A single row in a relation; one instance of an entity |
| **Primary Key (PK)** | A column (or set of columns) that uniquely identifies each row in a table |
| **Foreign Key (FK)** | A column that references the primary key of another table, enforcing a relationship |
| **Candidate Key** | Any column or combination that could serve as the primary key (unique, not null) |
| **Composite Key** | A primary key made of two or more columns together |
| **Surrogate Key** | An artificial key with no business meaning (e.g., auto-generated integer) |
| **Natural Key** | A key with real-world meaning (e.g., email, social security number) |
| **Domain** | The set of allowed values for a column (data type + constraints) |
| **First Normal Form (1NF)** | Every column holds atomic values; no repeating groups |
| **Second Normal Form (2NF)** | 1NF + no partial dependencies on composite keys |
| **Third Normal Form (3NF)** | 2NF + no transitive dependencies between non-key columns |
| **Boyce-Codd Normal Form (BCNF)** | Every determinant is a candidate key — stricter than 3NF |
| **Normalization** | The process of organizing tables to reduce redundancy and dependency anomalies |
| **Denormalization** | Deliberately introducing redundancy for performance, with documented justification |
| **Functional Dependency** | Column B is functionally dependent on column A if each A value determines exactly one B value |
| **ER Diagram** | A visual model showing entities, their attributes, and relationships between them |
| **Cardinality** | The number of instances in one entity that relate to instances of another (1:1, 1:N, M:N) |
| **Participation** | Whether every instance must participate in a relationship (total) or may not (partial) |
| **Junction Table** | A table that resolves a many-to-many relationship by holding two foreign keys |
| **Referential Integrity** | The guarantee that every foreign key value points to an existing row in the referenced table |
| **Constraint** | A rule enforced by the database (PK, FK, NOT NULL, UNIQUE, CHECK, DEFAULT) |
| **DDL** | Data Definition Language — SQL commands that define structure (CREATE, ALTER, DROP) |
| **DML** | Data Manipulation Language — SQL commands that modify data (INSERT, UPDATE, DELETE) |
| **Schema** | The complete structure of a database — all tables, columns, types, and constraints |
| **Anomaly** | A problem caused by poor design: insertion, update, or deletion anomaly |
| **Redundancy** | The same fact stored in multiple places — leads to anomalies |

---

## Reading Assignments

### Required Review

- Watt, A. & Eng, N. (2014). *Database Design — 2nd Edition*. BCcampus Open Education.
  - Chapter 7: The Relational Data Model
  - Chapter 8: Entity-Relationship Model (review)
  - Chapter 9: Mapping ER Models to Relational Schemas
  - Chapter 11: Functional Dependencies
  - Chapter 12: Normalization

### PostgreSQL Documentation

- [CREATE TABLE](https://www.postgresql.org/docs/current/sql-createtable.html) — complete DDL syntax
- [Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html) — all constraint types
- [Data Types](https://www.postgresql.org/docs/current/datatype.html) — type reference

---

## Further Reading

- Your own course project documentation — review it with the design quality checklist from Section 5
- Compare your project schema against the TrailShop reference schema in Section 6 for patterns and completeness

---

## Summary

You have now reviewed the complete database design pipeline — from requirements through ER modelling, logical design, normalization, and implementation. You have a quality checklist to evaluate any schema, a complete reference implementation in TrailShop, and guidelines for giving and receiving peer feedback.

The key insight of this lecture: **good design is not about knowing the rules — it is about applying them systematically and catching mistakes before they become expensive to fix.**

Week 50 offers an optional SQL reference and practice queries. Week 51 is the final exam.
