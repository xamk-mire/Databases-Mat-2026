# Week 45 — Normalization

## Chapter 9: "The Data Quality Audit"

TrailShop is running smoothly. Orders are flowing, products are tracked, customers are happy. But then a pattern emerges: a supplier changes their company name, and you have to update it in 47 rows. A seasonal product gets discontinued, and deleting it accidentally removes the only record that a certain supplier exists. A new product can't be added because there's no order to attach it to yet.

These aren't bugs in your queries — they're bugs in your *design*. The founders call an emergency meeting: "We thought databases were supposed to *prevent* data problems?" They're right. The issue is that the table structure allows **anomalies** — and the science of fixing them is called **normalization**.

As Watt writes in Chapter 12 of *Database Design — 2nd Edition*: "Normalization is the branch of relational theory that provides design insights. It is the process of determining how much redundancy exists in a table and provides techniques for reducing it."

This week you'll learn to diagnose and fix structural problems using the formal theory of normalization — functional dependencies, normal forms, and dependency diagrams.

---

## Learning Objectives

By the end of this chapter you will be able to:

- Explain the purpose and history of normalization
- Identify insertion, update, and deletion anomalies
- Define functional dependencies and identify them from data and business rules
- Apply Armstrong's axioms and derived inference rules
- Draw dependency diagrams
- Normalize a table through 1NF, 2NF, 3NF, and BCNF
- Evaluate an existing schema against normalization standards
- Recognize when denormalization is appropriate

---

## 1. What Is Normalization?

### 1.1 Historical Context

In 1970, E.F. Codd introduced the relational model. But having tables wasn't enough — Codd realized that *how* you structure those tables mattered enormously. In 1971 and 1972, he published papers introducing **First Normal Form (1NF)**, **Second Normal Form (2NF)**, and **Third Normal Form (3NF)**.

The key insight: just because data is in a table doesn't mean it's well-organized. Poorly structured tables create redundancy, and redundancy creates anomalies.

### 1.2 Goals of Normalization

Normalization aims to:

1. **Eliminate redundancy** — Each fact is stored in exactly one place.
2. **Prevent anomalies** — Inserting, updating, or deleting data doesn't cause unexpected side effects.
3. **Preserve data integrity** — The structure enforces correctness.
4. **Produce a clear, logical design** — Each table represents one concept.

As Watt & Eng state in Chapter 12: "The goal is to create a set of relations where each relation represents a single concept and where the facts can be maintained without anomalies."

### 1.3 The Normalization Process

Normalization is **decomposition** — splitting a poorly structured table into multiple well-structured tables. The process is:

1. Start with a table (possibly denormalized)
2. Identify functional dependencies
3. Check against each normal form (1NF → 2NF → 3NF → BCNF)
4. Decompose until no violations remain
5. Verify no data is lost (lossless-join decomposition)

### 1.4 How Normal Forms Relate

Each normal form builds on the previous one:

```
1NF ⊂ 2NF ⊂ 3NF ⊂ BCNF
```

A table in 3NF is automatically in 2NF and 1NF. A table in BCNF is automatically in 3NF. You normalize *incrementally* — fix 1NF violations first, then 2NF, then 3NF.

---

## 2. Data Anomalies

### 2.1 The Root Cause: Redundancy

Anomalies arise when the same fact is stored in multiple rows. Consider this poorly designed table:

**order_details_flat** (a denormalized design):

| order_id | customer_name | customer_email | product_name | category | quantity | price |
|----------|--------------|----------------|--------------|----------|----------|-------|
| 1001 | Anna Virtanen | anna@mail.fi | Trail Runner X | Footwear | 2 | 89.99 |
| 1001 | Anna Virtanen | anna@mail.fi | Hiking Poles | Equipment | 1 | 45.00 |
| 1002 | Mikko Lehto | mikko@mail.fi | Trail Runner X | Footwear | 1 | 89.99 |
| 1003 | Anna Virtanen | anna@mail.fi | Rain Jacket Pro | Clothing | 1 | 129.99 |

Anna's name and email appear in three rows. "Trail Runner X" and its category appear in two rows. This redundancy is the source of all anomalies.

### 2.2 Insertion Anomaly

**Problem**: You cannot store certain facts without storing other, unrelated facts.

**TrailShop example**: You want to add a new product "Mountain Tent" in category "Camping" to the catalog. But in `order_details_flat`, every row requires an `order_id` and `customer_name`. You can't record the product until someone orders it.

Similarly, you can't add a new customer until they place an order.

### 2.3 Update Anomaly

**Problem**: When a fact changes, you must update it in every row where it appears. Miss one, and you have inconsistent data.

**TrailShop example**: Anna Virtanen changes her email address. In the denormalized table, her email appears in three rows. If you update only two of them, you now have conflicting information — which email is correct?

```sql
-- Dangerous: might miss rows
UPDATE order_details_flat
SET customer_email = 'anna.new@mail.fi'
WHERE customer_name = 'Anna Virtanen';
-- What if her name is misspelled in one row?
```

### 2.4 Deletion Anomaly

**Problem**: Deleting a row accidentally removes facts that should be preserved.

**TrailShop example**: Mikko Lehto's order 1002 is cancelled and deleted. But his was the only row — deleting it also removes the fact that "Trail Runner X" costs 89.99 and belongs to "Footwear" (well, Anna's rows still have that, but imagine Mikko had ordered a unique product).

More dramatically: if a product was only ever ordered once, deleting that order loses all record that the product exists.

### 2.5 Summary of Anomalies

| Anomaly | Cause | Consequence |
|---------|-------|-------------|
| Insertion | Facts bundled together that shouldn't be | Can't add X without unrelated Y |
| Update | Same fact in multiple rows | Inconsistency if not all copies updated |
| Deletion | Facts bundled together that shouldn't be | Deleting X accidentally deletes Y |

As Watt & Eng emphasize in Chapter 10: "These anomalies are not software bugs — they are design flaws that no amount of careful programming can fully prevent."

---

## 3. Functional Dependencies

### 3.1 What Is a Functional Dependency?

A **functional dependency** (FD) is a constraint between two sets of attributes. We write:

```
X → Y
```

This reads: "X **functionally determines** Y" or "Y is **functionally dependent** on X."

It means: if two rows have the same value for X, they *must* have the same value for Y.

As Watt & Eng define in Chapter 11 of *Database Design — 2nd Edition*: "A functional dependency exists when the value of one attribute (or set of attributes) uniquely determines the value of another attribute (or set of attributes)."

**X** is the **determinant** (the left side).  
**Y** is the **dependent** (the right side).

### 3.2 Examples

From the `products` table:

```
product_id → product_name, price, stock_quantity, category_id
```

This means: knowing the product_id, you can determine exactly one product_name, price, etc. Two rows with the same product_id will always have the same product_name.

From the `customers` table:

```
customer_id → first_name, last_name, email, city
email → customer_id, first_name, last_name, city
```

Both `customer_id` and `email` are determinants — they each uniquely identify a customer.

### 3.3 How to Identify Functional Dependencies

There are two sources:

#### From business rules (preferred — more reliable):
- "Each product has exactly one price" → `product_id → price`
- "Each order belongs to exactly one customer" → `order_id → customer_id`
- "A student can enroll in a course only once" → `{student_id, course_id} → grade`

#### From examining data (less reliable — might miss edge cases):
Look at the sample data and ask: "If I know X, do I always know exactly one Y?"

**Warning**: Data alone can't prove an FD exists — it can only *disprove* one. If you find two rows with the same X but different Y values, the FD `X → Y` does NOT hold. But just because current data satisfies the pattern doesn't guarantee future data will.

### 3.4 Formal Notation Examples

Single attributes:
```
A → B          (A determines B)
```

Multiple attributes on the left (composite determinant):
```
{A, B} → C    (A and B together determine C)
```

Multiple attributes on the right:
```
A → B, C, D   (A determines B, C, and D — shorthand for A → B, A → C, A → D)
```

### 3.5 Armstrong's Axioms

William Armstrong proved in 1974 that these three rules are **sound and complete** — any valid FD can be derived from them, and they never produce invalid FDs. Referenced in Chapter 11 of Watt & Eng.

#### Reflexivity (Trivial dependency)

If Y is a subset of X, then X → Y.

```
{product_id, product_name} → product_id     (trivial — always true)
{A, B} → A                                   (trivial)
```

This is "obvious" — knowing a set of attributes, you automatically know any subset.

#### Augmentation

If X → Y, then XZ → YZ (adding the same attributes to both sides preserves the dependency).

```
If product_id → price,
then {product_id, category_id} → {price, category_id}
```

#### Transitivity

If X → Y and Y → Z, then X → Z.

```
If order_item_id → product_id,
and product_id → product_name,
then order_item_id → product_name.
```

### 3.6 Derived Inference Rules

From Armstrong's axioms, we can derive additional useful rules:

#### Union Rule

If X → Y and X → Z, then X → YZ.

```
If customer_id → first_name and customer_id → email,
then customer_id → {first_name, email}.
```

#### Decomposition Rule

If X → YZ, then X → Y and X → Z.

```
If customer_id → {first_name, last_name},
then customer_id → first_name and customer_id → last_name.
```

#### Pseudotransitivity

If X → Y and WY → Z, then WX → Z.

```
If order_id → customer_id,
and {customer_id, product_id} → discount_rate,
then {order_id, product_id} → discount_rate.
```

### 3.7 Closure of a Set of Attributes

The **closure** of a set of attributes X (written X⁺) under a set of FDs is all attributes that can be determined by X.

**Example**: Given FDs for a table:
```
A → B
B → C
A → D
```

The closure of {A} is:
1. Start: {A}
2. A → B: add B → {A, B}
3. B → C: add C → {A, B, C}
4. A → D: add D → {A, B, C, D}

So A⁺ = {A, B, C, D}. If these are all attributes in the table, then A is a **candidate key** (it determines everything).

### 3.8 Dependency Diagrams

A **dependency diagram** visually shows which attributes determine which others. As described in Chapter 11 of Watt & Eng, these diagrams help identify partial and transitive dependencies.

**How to draw:**
1. Write all attributes in a row (or grouped logically).
2. Underline or box the primary key attributes.
3. Draw arrows from determinants to dependents.
4. Use different line styles for partial vs. full dependencies.

**Example** for a poorly designed table:

```
Table: enrollment(student_id, course_id, student_name, course_name, grade)
PK: {student_id, course_id}

                    student_name
                   ↗
[student_id, course_id] → grade
         ↓
    student_id → student_name    (PARTIAL dependency!)
    course_id  → course_name     (PARTIAL dependency!)
```

The arrows from individual key components to non-key attributes indicate partial dependencies (violates 2NF).

---

## 4. Partial Dependencies

### 4.1 Definition

A **partial dependency** occurs when a non-key attribute is functionally dependent on **part** of a composite primary key, rather than the whole key.

Reference: Chapter 11 of Watt & Eng discusses partial dependencies as the specific violation that 2NF addresses.

### 4.2 When Does It Apply?

Partial dependencies can only occur in tables with **composite primary keys** (keys made of two or more columns). If your PK is a single column, partial dependencies are impossible.

### 4.3 Example

**Table**: order_items(order_id, product_id, quantity, product_name, product_price)  
**PK**: {order_id, product_id}

FDs:
```
{order_id, product_id} → quantity         (FULL dependency — correct)
product_id → product_name, product_price  (PARTIAL dependency — violation!)
```

`product_name` depends only on `product_id`, not on the full key `{order_id, product_id}`. This is a partial dependency.

### 4.4 Why It's a Problem

If product_id 101 is "Trail Runner X" at €89.99, this fact is repeated in every order_item row for that product. Change the price? Must update every row. Delete the last order containing it? Lose the product info.

---

## 5. Transitive Dependencies

### 5.1 Definition

A **transitive dependency** occurs when a non-key attribute depends on another non-key attribute (which itself depends on the primary key).

```
PK → A → B     (B is transitively dependent on PK through A)
```

Reference: Chapter 11 of Watt & Eng: "A transitive dependency exists when a non-key attribute is determined by another non-key attribute."

### 5.2 Example

**Table**: products(product_id, product_name, category_id, category_name)  
**PK**: product_id

FDs:
```
product_id → product_name, category_id, category_name
category_id → category_name
```

Here, `category_name` is transitively dependent on `product_id` through `category_id`:
```
product_id → category_id → category_name
```

### 5.3 Why It's a Problem

Category names are repeated for every product in that category. If "Outdoor Gear" is renamed to "Outdoor Equipment," you must update every product row in that category.

### 5.4 Identifying Transitive Dependencies

Ask: "Is there a non-key column that determines another non-key column?"

If yes → transitive dependency → violates 3NF.

---

## 6. Normal Forms

### 6.1 First Normal Form (1NF)

#### Rule

A table is in **1NF** if:
1. All attribute values are **atomic** (indivisible) — no lists, sets, or repeating groups in a single cell.
2. Each row is unique (has a primary key).
3. There are no repeating groups (no "item1, item2, item3" columns).

Reference: Chapter 12 of Watt & Eng defines 1NF as requiring atomic values and no repeating groups.

#### Violations

**Non-atomic values:**
| customer_id | name | phone_numbers |
|---|---|---|
| 1 | Anna | 040-1234567, 050-7654321 |

**Repeating groups (multiple columns for the same concept):**
| order_id | product1 | qty1 | product2 | qty2 | product3 | qty3 |
|---|---|---|---|---|---|---|
| 1001 | Shoes | 1 | Poles | 2 | NULL | NULL |

#### Fix: Convert to 1NF

**For non-atomic values** — create a separate row for each value (or a separate table):

| customer_id | name | phone_number |
|---|---|---|
| 1 | Anna | 040-1234567 |
| 1 | Anna | 050-7654321 |

Better: Create a `customer_phones` table:
```sql
CREATE TABLE customer_phones (
    customer_id INTEGER REFERENCES customers(customer_id),
    phone_number VARCHAR(20),
    phone_type VARCHAR(10),
    PRIMARY KEY (customer_id, phone_number)
);
```

**For repeating groups** — move repeating data to a new table:

```sql
CREATE TABLE order_items (
    order_id INTEGER REFERENCES orders(order_id),
    product_id INTEGER REFERENCES products(product_id),
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id)
);
```

### 6.2 Second Normal Form (2NF)

#### Rule

A table is in **2NF** if:
1. It is in 1NF, AND
2. Every non-key attribute is **fully functionally dependent** on the *entire* primary key (no partial dependencies).

Reference: Chapter 12 of Watt & Eng: "2NF requires that every non-key attribute be dependent on the whole key."

#### When Does 2NF Apply?

**Only tables with composite primary keys can violate 2NF.** If your PK is a single column, you're automatically in 2NF (assuming 1NF).

#### Example Violation

**Table**: enrollment(student_id, course_id, student_name, course_name, grade)  
**PK**: {student_id, course_id}

FDs:
```
{student_id, course_id} → grade          (full dependency ✓)
student_id → student_name                 (partial dependency ✗)
course_id → course_name                   (partial dependency ✗)
```

#### Fix: Decompose

Remove partially dependent attributes into their own tables:

```
students(student_id PK, student_name)
courses(course_id PK, course_name)
enrollment(student_id, course_id, grade)  PK: {student_id, course_id}
```

Now every non-key attribute depends on the *full* key of its table.

### 6.3 Third Normal Form (3NF)

#### Rule

A table is in **3NF** if:
1. It is in 2NF, AND
2. No non-key attribute is **transitively dependent** on the primary key (no non-key attribute determines another non-key attribute).

Reference: Chapter 12 of Watt & Eng: "3NF requires that no non-key attribute is dependent on another non-key attribute."

#### The Informal Rule

"Every non-key attribute must depend on **the key, the whole key, and nothing but the key** — so help me Codd."

This famous phrase captures all three normal forms:
- "the key" → must have a key (1NF)
- "the whole key" → full dependency (2NF)
- "nothing but the key" → no transitive dependencies (3NF)

#### Example Violation

**Table**: products_v1(product_id PK, product_name, category_id, category_name)

FDs:
```
product_id → product_name, category_id, category_name
category_id → category_name    (transitive dependency!)
```

`category_name` depends on `category_id`, which is a non-key attribute.

#### Fix: Decompose

```
products(product_id PK, product_name, category_id FK)
categories(category_id PK, category_name)
```

Now `category_name` depends directly on the key of its own table.

### 6.4 Boyce-Codd Normal Form (BCNF)

#### Rule

A table is in **BCNF** if:
- For **every** functional dependency X → Y, X is a **superkey** (a candidate key or superset of one).

In other words: every determinant must be a candidate key.

Reference: Chapter 12 of Watt & Eng: "BCNF is a stricter form of 3NF. A relation is in BCNF if, and only if, every determinant is a candidate key."

#### Difference from 3NF

3NF allows a non-key attribute to determine a *key* attribute (which is rare but possible). BCNF does not allow this.

#### Example 1 (from textbook)

**Table**: teaching(student_id, subject, teacher)  
**Constraint**: Each teacher teaches only one subject. Each subject can be taught by multiple teachers.

Candidate keys: {student_id, subject} and {student_id, teacher}

FD: teacher → subject

This FD violates BCNF because `teacher` is not a candidate key (it's not sufficient to determine the whole row). But it doesn't violate 3NF because `subject` is part of a candidate key.

#### Example 2

**Table**: address(zip_code, street, city)  
**Constraint**: A zip code determines the city (in many countries).

FD: zip_code → city

If the PK is {zip_code, street}, then zip_code is a proper subset of the key that determines `city` — partial dependency (violates 2NF, and therefore BCNF too).

#### When to Apply BCNF

In practice, most well-designed 3NF schemas are also in BCNF. The exceptions are rare and involve overlapping candidate keys. For this course, reaching 3NF is usually sufficient.

---

## 7. Step-by-Step Normalization Walkthrough

Let's normalize a TrailShop denormalized table from scratch.

### Starting Table

**order_report** — a flat export from the old system:

| order_id | order_date | customer_id | customer_name | customer_email | product_id | product_name | category_id | category_name | quantity | unit_price |
|---|---|---|---|---|---|---|---|---|---|---|
| 1001 | 2025-03-15 | 1 | Anna Virtanen | anna@mail.fi | 101 | Trail Runner X | 10 | Footwear | 2 | 89.99 |
| 1001 | 2025-03-15 | 1 | Anna Virtanen | anna@mail.fi | 202 | Hiking Poles | 20 | Equipment | 1 | 45.00 |
| 1002 | 2025-03-16 | 2 | Mikko Lehto | mikko@mail.fi | 101 | Trail Runner X | 10 | Footwear | 1 | 89.99 |

### Step 1: Identify All Functional Dependencies

From business rules:
```
order_id → order_date, customer_id
customer_id → customer_name, customer_email
product_id → product_name, category_id, unit_price
category_id → category_name
{order_id, product_id} → quantity
```

### Step 2: Check 1NF

✓ All values are atomic (no lists, no repeating groups).  
✓ Rows are unique given the composite key {order_id, product_id}.  
**Result**: Already in 1NF.

### Step 3: Identify the Primary Key

The PK is `{order_id, product_id}` — this combination uniquely identifies each row.

### Step 4: Check 2NF — Look for Partial Dependencies

With PK = {order_id, product_id}, check if any non-key attribute depends on *part* of the key:

- `order_id → order_date, customer_id, customer_name, customer_email` — **PARTIAL** (depends only on order_id)
- `product_id → product_name, category_id, category_name, unit_price` — **PARTIAL** (depends only on product_id)
- `{order_id, product_id} → quantity` — full dependency ✓

**Violations found!** Decompose:

**orders**(order_id PK, order_date, customer_id, customer_name, customer_email)  
**products_v1**(product_id PK, product_name, category_id, category_name, unit_price)  
**order_items**(order_id, product_id, quantity) PK: {order_id, product_id}

**Result**: Now in 2NF.

### Step 5: Check 3NF — Look for Transitive Dependencies

**In `orders`**:
```
order_id → customer_id → customer_name, customer_email
```
`customer_name` and `customer_email` are transitively dependent through `customer_id`. **Violation!**

**In `products_v1`**:
```
product_id → category_id → category_name
```
`category_name` is transitively dependent through `category_id`. **Violation!**

**In `order_items`**: No transitive dependencies (only one non-key attribute: quantity). ✓

Decompose again:

**customers**(customer_id PK, customer_name, customer_email)  
**orders**(order_id PK, order_date, customer_id FK→customers)  
**categories**(category_id PK, category_name)  
**products**(product_id PK, product_name, category_id FK→categories, unit_price)  
**order_items**(order_id FK, product_id FK, quantity) PK: {order_id, product_id}

### Step 6: Verify 3NF

For each table, check: does every non-key attribute depend on "the key, the whole key, and nothing but the key"?

- **customers**: customer_id → customer_name, customer_email ✓
- **orders**: order_id → order_date, customer_id ✓
- **categories**: category_id → category_name ✓
- **products**: product_id → product_name, category_id, unit_price ✓
- **order_items**: {order_id, product_id} → quantity ✓

**All tables in 3NF!**

### Step 7: Check BCNF

Every determinant in every table is a candidate key. ✓ All tables are also in BCNF.

---

## 8. Dependency Diagrams

### 8.1 Purpose

Dependency diagrams give you a visual map of how attributes relate. They make it easy to spot partial and transitive dependencies.

### 8.2 Notation

```
[Underlined] = Primary key attribute
─────────→   = Full functional dependency  
- - - - →   = Partial dependency (violates 2NF)
═══════════→ = Transitive dependency (violates 3NF)
```

### 8.3 Example: The Denormalized order_report

```
┌─────────────────────────────────────────────────────────────────────┐
│ PK: [order_id, product_id]                                          │
│                                                                     │
│ [order_id] ─ ─ ─ → order_date                                     │
│            ─ ─ ─ → customer_id ════→ customer_name                 │
│                                  ════→ customer_email               │
│                                                                     │
│ [product_id] ─ ─ → product_name                                    │
│              ─ ─ → category_id ════→ category_name                 │
│              ─ ─ → unit_price                                      │
│                                                                     │
│ [order_id, product_id] ──→ quantity  (full dependency)             │
└─────────────────────────────────────────────────────────────────────┘
```

The dashed arrows (─ ─ →) show partial dependencies. The double arrows (════→) show transitive dependencies. Both must be eliminated for 3NF.

### 8.4 After Normalization

Each table has a clean diagram:

```
customers:    [customer_id] ──→ customer_name, customer_email
orders:       [order_id] ──→ order_date, customer_id
categories:   [category_id] ──→ category_name
products:     [product_id] ──→ product_name, category_id, unit_price
order_items:  [order_id, product_id] ──→ quantity
```

All arrows originate from the full primary key. No partial or transitive dependencies remain.

---

## 9. When NOT to Normalize

Normalization isn't always the right answer. There are legitimate reasons to keep some redundancy.

### 9.1 Performance (Denormalization)

Highly normalized schemas require many JOINs. For read-heavy systems with millions of rows, those JOINs can be expensive. **Denormalization** deliberately introduces controlled redundancy for performance.

**Example**: Storing `category_name` directly in the products table to avoid a JOIN in a frequently-run query.

**Important**: Denormalize *after* you've designed a normalized schema and *measured* a performance problem. Don't skip normalization because "JOINs are slow" — modern databases handle JOINs efficiently.

### 9.2 Reporting and Analytics

Data warehouses and reporting databases often use denormalized **star schemas** or **snowflake schemas** because:
- Analytical queries scan large datasets and benefit from fewer JOINs.
- Data is loaded in batches (not updated row-by-row), so update anomalies don't apply.

### 9.3 Historical Snapshots

Sometimes you *need* redundancy to preserve history:
- An order should record the price *at the time of sale*, not the current price.
- A shipped order should record the customer's *delivery address at that time*.

This isn't really denormalization — it's a different fact. The current price and the order price are different attributes.

### 9.4 Practical Guidelines

| Situation | Recommendation |
|-----------|---------------|
| OLTP (transactional) system | Normalize to 3NF |
| Reporting/analytics | Consider denormalization |
| Small tables (< 10k rows) | Normalize — JOIN cost negligible |
| Frequently joined lookup tables | Normalize — PostgreSQL handles it well |
| Write-heavy tables | Normalize — reduces update anomalies |
| Read-heavy, rarely updated | Consider strategic denormalization |

---

## 10. TrailShop Schema Audit

Let's evaluate TrailShop's current schema against 3NF.

### Current Schema

```sql
customers (customer_id PK, first_name, last_name, email, city)
categories (category_id PK, category_name)
products (product_id PK, product_name, category_id FK, price, stock_quantity)
orders (order_id PK, customer_id FK, order_date, status)
order_items (order_item_id PK, order_id FK, product_id FK, quantity, unit_price)
```

### Analysis

**customers**:
- PK: customer_id
- FDs: customer_id → first_name, last_name, email, city
- Any transitive dependencies? Could `city → region` exist? Only if we stored region. Currently clean.
- **Verdict: 3NF ✓**

**categories**:
- PK: category_id
- FDs: category_id → category_name
- **Verdict: 3NF ✓**

**products**:
- PK: product_id
- FDs: product_id → product_name, category_id, price, stock_quantity
- No non-key attribute determines another non-key attribute.
- **Verdict: 3NF ✓**

**orders**:
- PK: order_id
- FDs: order_id → customer_id, order_date, status
- **Verdict: 3NF ✓**

**order_items**:
- PK: order_item_id (surrogate)
- Also unique: {order_id, product_id} (if a product appears once per order)
- FDs: order_item_id → order_id, product_id, quantity, unit_price
- **Verdict: 3NF ✓**

**Design note on `unit_price` in order_items**: This is *intentional* redundancy. It records the price at the time of sale. If `products.price` changes later, existing orders keep their historical price. This is correct practice — not a normalization violation.

### Conclusion

TrailShop's schema is well-normalized (3NF/BCNF). The only "redundancy" (`unit_price` in `order_items`) is a deliberate historical snapshot.

---

## 11. Key Terms

| Term | Definition |
|------|-----------|
| Normalization | Process of organizing tables to reduce redundancy and prevent anomalies |
| Functional dependency (FD) | Constraint X → Y meaning X uniquely determines Y |
| Determinant | Left side of a functional dependency |
| Dependent | Right side of a functional dependency |
| Partial dependency | Non-key attribute depends on part of a composite key (violates 2NF) |
| Transitive dependency | Non-key attribute depends on another non-key attribute (violates 3NF) |
| 1NF | Atomic values, no repeating groups, has a primary key |
| 2NF | 1NF + no partial dependencies |
| 3NF | 2NF + no transitive dependencies |
| BCNF | Every determinant is a candidate key |
| Decomposition | Splitting a table into multiple tables to eliminate violations |
| Anomaly | Unintended side effect of poor table design (insertion, update, deletion) |
| Candidate key | Minimal set of attributes that uniquely identifies every row |
| Closure | Set of all attributes determinable from a given set of attributes |
| Armstrong's axioms | Reflexivity, augmentation, transitivity — the foundation of FD inference |
| Denormalization | Deliberately introducing redundancy for performance |

---

## 12. Reading

### Required

- Watt & Eng, *Database Design — 2nd Edition*, **Chapter 10**: Data Anomalies
- Watt & Eng, *Database Design — 2nd Edition*, **Chapter 11**: Functional Dependencies
- Watt & Eng, *Database Design — 2nd Edition*, **Chapter 12**: Normalization (1NF, 2NF, 3NF, BCNF)

### Further Reading

- PostgreSQL Documentation: [CREATE TABLE](https://www.postgresql.org/docs/current/sql-createtable.html) — constraints that enforce normalized design
- Codd, E.F. (1970). "A Relational Model of Data for Large Shared Data Banks." *Communications of the ACM*.
- Date, C.J. (2003). *An Introduction to Database Systems*, 8th Edition — for deeper normalization theory.

---

## 13. Summary

This chapter took you from observing data problems to understanding the formal theory behind fixing them:

- **Data anomalies** (insertion, update, deletion) arise from redundancy caused by poor table design.
- **Functional dependencies** formalize the relationships between attributes — they are the diagnostic tool.
- **Armstrong's axioms** provide the logical foundation for reasoning about FDs.
- **Normal forms** give you a systematic checklist: 1NF (atomic values) → 2NF (no partial dependencies) → 3NF (no transitive dependencies) → BCNF (every determinant is a key).
- **Dependency diagrams** visualize the problems and confirm fixes.
- **Denormalization** is sometimes appropriate — but only after you understand what you're sacrificing.

TrailShop's schema passed the audit. That's because it was designed with normalization in mind from the start. Now you understand *why* it was designed that way.

---

## Coming Next: Week 46 — Transactions and Concurrency

What happens when two customers try to buy the last item at the same time? How do you ensure that a multi-step order process either completes entirely or not at all? Next week: transactions, ACID properties, and the challenges of concurrent access. Get ready for Black Friday at TrailShop.
