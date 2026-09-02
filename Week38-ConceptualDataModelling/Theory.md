# Week 38 — Conceptual Data Modelling

## Chapter 3: "Mapping the Territory"

Last week you learned the rules of relational databases — how tables work, what keys do, and how integrity constraints keep data trustworthy. But here's the thing: you shouldn't start building tables right away. That would be like a construction crew pouring concrete before an architect has drawn a single blueprint.

Imagine TrailShop's founders walk into your office and say: "We need the database to handle products, customers, orders, categories, and stock levels. Oh, and eventually loyalty points, supplier info, reviews, and wishlists." If you jump straight into `CREATE TABLE` statements, you'll end up reworking everything three times. You need a **map** first — a visual model that captures *what* the business needs before you decide *how* to store it.

That map is called a **conceptual data model**, and the most popular technique for creating one is the **Entity-Relationship (ER) model**. This week you'll learn to think like a data architect: sketching the big picture before writing a single line of SQL.

> "Weeks of coding can save you hours of planning." — Anonymous developer proverb

---

## Learning Objectives

By the end of this chapter you will be able to:

- Explain why data modelling should precede database implementation
- Distinguish between external, conceptual, logical, and physical data models
- Describe the three-schema architecture and data independence
- Define entities, attributes, and relationships in the ER model
- Classify entities as strong, weak, independent, dependent, or characteristic
- Classify attributes as simple, composite, multivalued, derived, or key
- Identify binary, unary (recursive), and ternary relationships
- Read and apply cardinality and participation constraints
- Draw ER diagrams using crow's foot notation
- Resolve many-to-many relationships using junction tables
- Apply ER diagram best practices

---

## 1. Why Model Before Building

### 1.1 The Software Engineering Analogy

In software engineering, no competent team ships a large application without first gathering requirements, designing architecture, and sketching interfaces. The same applies to databases — arguably even more so, because restructuring a database after it's loaded with production data is painful, risky, and expensive.

Consider this parallel:

| Software Development | Database Development |
|---|---|
| Requirements document | Data requirements specification |
| Architecture diagram | Conceptual data model (ER diagram) |
| Detailed design | Logical schema (table definitions) |
| Source code | Physical implementation (SQL DDL) |
| Testing | Data validation and integrity checks |

Just as you wouldn't code a complex application without a design document, you shouldn't create database tables without a data model.

### 1.2 The Cost of Skipping Modelling

If TrailShop's founders said "just make a table for everything," you might end up with:

- A `products` table that stores customer info in extra columns "because it was convenient"
- An `orders` table with 30 columns because nobody thought about separating order items
- Duplicate data scattered across tables with no clear relationships
- No way to add suppliers or reviews without restructuring everything

A conceptual model prevents these problems by forcing you to think about **what data exists**, **how it relates**, and **what rules govern it** — before you commit to any technical decisions.

### 1.3 What the Textbook Says

*Database Design* (Watt & Eng), Chapter 5, introduces data modelling as "an iterative process" that begins at a high level of abstraction and is refined until it can be implemented. Chapter 13 describes the full database development process and places conceptual modelling as a critical early phase — one that feeds into logical design and ultimately physical implementation.

The key insight: **conceptual modelling is technology-independent**. You don't think about PostgreSQL, MySQL, or Oracle at this stage. You think about the *business*.

---

## 2. Levels of Data Models

Not all models serve the same purpose. Data models exist at different levels of abstraction, each addressing different audiences and questions.

### 2.1 The Four Levels

*Database Design*, Chapter 5, describes these degrees of data abstraction:

**External Level (View Level)**
This is the level individual users or applications see. Different people need different views of the same data.

- The **warehouse manager** sees: product names, stock quantities, reorder thresholds
- The **marketing team** sees: product names, descriptions, categories, prices
- The **customer** sees: product names, prices, images, reviews

Each of these is an **external schema** (also called a subschema or user view). None of them shows the complete picture — and that's intentional. Users only see what they need.

**Conceptual Level**
This is the unified, organization-wide view of *all* the data and the relationships between data elements. It answers: "What information does the organization need to track?"

The conceptual schema describes:
- All entities (things we store data about)
- Their attributes (properties)
- The relationships between entities
- Constraints and business rules

It does NOT describe how data is physically stored or how individual users see it.

**Logical Level**
This translates the conceptual model into the structures of a specific *type* of database system (relational, document, graph, etc.). For a relational database, this means:
- Tables with columns and data types
- Primary keys and foreign keys
- Constraints

The logical model is still independent of a specific DBMS product but commits to the relational model's rules.

**Physical Level**
This deals with how data is actually stored on disk: file structures, indexes, partitioning, storage parameters, tablespaces. This is DBMS-specific.

### 2.2 How the Levels Relate

```
┌─────────────────────────────────────────────┐
│            EXTERNAL LEVEL                   │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│  │ View A  │ │ View B  │ │ View C  │  ...  │
│  │(Warehouse│ │(Marketing│ │(Customer│       │
│  │ Manager)│ │  Team)  │ │  App)   │       │
│  └────┬────┘ └────┬────┘ └────┬────┘       │
│       │           │           │             │
├───────┴───────────┴───────────┴─────────────┤
│            CONCEPTUAL LEVEL                 │
│                                             │
│   Unified view of ALL data + relationships  │
│   (ER Diagram lives here)                   │
│                                             │
├─────────────────────────────────────────────┤
│            LOGICAL LEVEL                    │
│                                             │
│   Tables, columns, keys, constraints        │
│   (Relational schema lives here)            │
│                                             │
├─────────────────────────────────────────────┤
│            PHYSICAL LEVEL                   │
│                                             │
│   Files, indexes, storage, partitions       │
│   (PostgreSQL internals live here)          │
│                                             │
└─────────────────────────────────────────────┘
```

This layered architecture is often called the **ANSI-SPARC three-schema architecture** (external, conceptual, internal). Chapter 5 of the textbook discusses this architecture in detail.

### 2.3 Where We Are This Week

This week (Week 38) focuses on the **conceptual level** — creating an ER diagram. Next week (Week 39) we'll move to the **logical level** — translating the ER diagram into relational tables with data types and constraints.

---

## 3. Data Abstraction and Data Independence

### 3.1 The Three-Schema Architecture in Detail

The three-schema architecture, originally proposed by the ANSI/SPARC committee in 1975, defines three distinct layers of schema. Chapter 5 (Watt & Eng) uses this as a foundation for understanding why we model at different levels.

**External Schema (Subschema)**
Each user group has its own external schema defining what data they can see and how it appears to them. In PostgreSQL, external schemas are implemented as **views**:

```sql
-- External schema for the warehouse team
CREATE VIEW warehouse_view AS
SELECT product_id, name, stock_quantity
FROM products
WHERE stock_quantity < 20;
```

The warehouse team sees only low-stock products. They don't know (or care) about prices, categories, or how the data is physically stored.

**Conceptual Schema**
The single, unified description of all data in the organization. This is the "truth" — the complete model that all external schemas are derived from.

For TrailShop, the conceptual schema would describe:
- Categories, Products, Customers, Orders, and OrderItems as entities
- All their attributes
- All the relationships and constraints between them

**Internal Schema (Physical Schema)**
The description of how data is physically stored: data file structures, index definitions, compression, partitioning. In PostgreSQL, this includes tablespaces, index types (B-tree, hash, GIN, GiST), and storage parameters.

### 3.2 Data Independence

The real power of this architecture is **data independence** — the ability to change one level without affecting the others.

**Logical Data Independence**
You can change the conceptual schema without changing the external schemas (user views). For example:

- You split the `products` table into `products` and `product_details`
- The warehouse team's view still works because you redefine it to `JOIN` the two tables
- The users see no difference

This is hard to achieve in practice but is the ideal.

**Physical Data Independence**
You can change the internal schema without changing the conceptual or external schemas. For example:

- You move the database from one disk to another
- You add an index on `products.name` to speed up searches
- You switch from B-tree to hash indexing
- None of the SQL queries or views change

Physical data independence is easier to achieve and is a major benefit of using a DBMS. Chapter 5 emphasizes that a key goal of the three-schema architecture is to maximize data independence.

### 3.3 Why This Matters for TrailShop

As TrailShop grows, the database will change:

| Change | Level Affected | Other Levels Affected? |
|---|---|---|
| Add a `suppliers` table | Conceptual + Logical | External views don't break |
| Add an index on `orders.order_date` | Physical | Nothing else changes |
| Create a new view for the accounting team | External | Conceptual and physical are untouched |
| Move database to a faster server | Physical | Nothing else changes |
| Split `products` into normalized tables | Conceptual + Logical | External views need updating (ideally not) |

---

## 4. The Entity-Relationship (ER) Model

### 4.1 A Brief History

In 1976, **Peter Pin-Shan Chen** published a landmark paper titled "The Entity-Relationship Model: Toward a Unified View of Data." Before Chen's work, there was no standard way to visualize database structures at the conceptual level. Developers jumped straight from informal descriptions to implementation — and made lots of mistakes.

Chen proposed three simple building blocks:

1. **Entities** — things we store data about
2. **Attributes** — properties of those things
3. **Relationships** — how entities are connected

These three concepts remain the foundation of conceptual data modelling to this day.

### 4.2 Where ER Fits in the Textbook

*Database Design*, Chapter 4, surveys various types of data models (hierarchical, network, relational, ER, object-oriented). The ER model is classified as a **conceptual model** — it's used for communication and planning, not for direct implementation.

Chapter 8 is dedicated entirely to the ER model: entity types, relationship types, attributes, and how to build an ER diagram. We'll reference Chapter 8 extensively throughout this section.

### 4.3 Why ER Diagrams?

ER diagrams serve as a **communication tool** between:

- **Business stakeholders** who know the domain but not databases
- **Database designers** who know databases but may not know the domain
- **Developers** who will write applications against the database
- **Future maintainers** who need to understand the system

An ER diagram is language- and technology-neutral. You can show it to TrailShop's founders and they can validate it: "Yes, one category has many products. Yes, one customer can place many orders."

---

## 5. Entities — Expanded

### 5.1 What Is an Entity?

An **entity** is a "thing" in the real world that the organization wants to track. It can be:

- A physical object (a product, a warehouse)
- A person (a customer, an employee)
- An event (an order, a shipment)
- A concept (a category, a department)

Each individual occurrence of an entity is called an **entity instance** (or entity occurrence). "Alpine Pro Hiking Boots" is an instance of the Product entity.

An **entity type** (or entity set) is the collection of all instances that share the same attributes: all products, all customers, all orders.

### 5.2 Strong vs Weak Entities

Chapter 8 (Watt & Eng) distinguishes between strong and weak entities:

**Strong Entity**
A strong entity can be uniquely identified by its own attributes. It does not depend on any other entity for its existence or identification.

- **Product** is a strong entity — each product has a `product_id` that uniquely identifies it regardless of any other entity.
- **Customer** is a strong entity — each customer has a `customer_id`.
- **Category** is a strong entity.

**Weak Entity**
A weak entity cannot be uniquely identified by its own attributes alone. It depends on a related strong entity (called the **owner** or **identifying entity**) for its identification.

Classic example: **OrderItem**. An order item only makes sense in the context of a specific order. The attributes of an order item (like `line_number = 1`) are not unique on their own — line 1 could appear in every order. You need the combination of `order_id` + `line_number` to uniquely identify an order item.

Properties of weak entities:
- They always participate in an **identifying relationship** with their owner
- Their primary key includes the owner's primary key (forming a composite key)
- If the owner is deleted, the weak entity instances should also be deleted

More examples beyond TrailShop:

| Weak Entity | Owner Entity | Why It's Weak |
|---|---|---|
| Room | Building | Room 101 exists in many buildings; needs building_id + room_number |
| Dependent (insurance) | Employee | A dependent is identified through the employee |
| Chapter | Book | Chapter 3 exists in many books |
| Transaction | Bank Account | Transaction #1 per account, not globally unique |

### 5.3 Entity Classification (Extended)

Chapter 8 classifies entities into three categories based on their relationships:

**Independent Entity (Kernel Entity)**
An entity that can exist on its own, without requiring any relationship to another entity. It has its own primary key that is not derived from any other entity.

In TrailShop:
- `Category` — exists independently; doesn't need products to exist
- `Customer` — exists independently; doesn't need orders to exist
- `Product` — exists independently (though it has a relationship to Category, it can be identified on its own)

**Dependent Entity**
An entity whose existence depends on one or more other entities. It cannot exist without the entity it depends on. Weak entities are a specific type of dependent entity.

In TrailShop:
- `Order` — depends on `Customer` (an order must belong to a customer)
- `OrderItem` — depends on both `Order` and `Product`

**Characteristic Entity**
A special kind of dependent entity that serves to provide additional detail about another entity — essentially a multivalued attribute that has been promoted to its own entity.

Examples:
- `ProductImage` — stores multiple images for a product (a product can have 0 to many images)
- `PhoneNumber` — if a customer can have multiple phone numbers
- `ProductReview` — each review characterizes a product

Characteristic entities typically have a mandatory one-to-many relationship with their parent entity.

---

## 6. Attributes — Expanded

### 6.1 What Is an Attribute?

An **attribute** is a property that describes an entity. Each attribute has a **name** and a **domain** (the set of permitted values). Chapter 8 (Watt & Eng) covers attribute types in detail.

For the Product entity, attributes might include:
- `product_id` (identifier)
- `name` (what it's called)
- `price` (how much it costs)
- `weight_kg` (how heavy it is)
- `stock_quantity` (how many are in stock)
- `description` (detailed text)

### 6.2 Simple (Atomic) Attributes

A **simple attribute** cannot be meaningfully subdivided. It holds a single, atomic value.

Examples:
- `price` — 149.99 is one value, not divisible into meaningful parts
- `stock_quantity` — 42 is one value
- `email` — "customer@example.com" (typically treated as atomic)

In the relational model, all attributes should ideally be simple (this is the first normal form requirement — more on that in later weeks).

### 6.3 Composite Attributes

A **composite attribute** can be divided into smaller, meaningful sub-attributes.

Example: A customer's **address** is composite:

```
address
├── street        → "Koulukatu 1"
├── city          → "Mikkeli"
├── postal_code   → "50100"
└── country       → "Finland"
```

Another example: **full_name** could be split into `first_name` and `last_name`.

**How composite attributes map to the relational model:**
You don't store a single "address" column. Instead, you decompose it into separate columns: `street`, `city`, `postal_code`, `country`. This allows you to query and sort by individual components (e.g., "find all customers in Mikkeli").

### 6.4 Multivalued Attributes

A **multivalued attribute** can hold multiple values for a single entity instance.

Examples:
- A product might have multiple **tags**: "waterproof", "lightweight", "bestseller"
- A customer might have multiple **phone numbers**: home, work, mobile
- A product might have multiple **images**

**How multivalued attributes map to the relational model:**
You cannot store multiple values in a single cell (that would violate first normal form). Instead, you create a separate table:

```
Product: Alpine Pro Hiking Boots
  Tags: ["waterproof", "durable", "bestseller"]

  Becomes:
  
  product_tags table:
  | product_id | tag          |
  |------------|--------------|
  | 101        | waterproof   |
  | 101        | durable      |
  | 101        | bestseller   |
```

Alternatively, PostgreSQL supports array types (`TEXT[]`), but a separate table is the traditional relational approach and is more flexible for querying.

### 6.5 Derived Attributes

A **derived attribute** is one whose value can be calculated from other attributes. It is not stored directly but computed when needed.

Examples:
- `age` can be derived from `date_of_birth` and today's date
- `total_price` on an order item can be derived from `quantity * unit_price`
- `order_total` can be derived by summing all order item totals
- `years_as_customer` can be derived from `registration_date`

**How derived attributes map to the relational model:**
Generally, you do NOT store derived attributes. Instead, you compute them in queries:

```sql
SELECT quantity * unit_price AS line_total
FROM order_items
WHERE order_id = 1001;
```

In some cases, you might store derived values for performance reasons (denormalization), but this creates the risk of the stored value becoming inconsistent with the source data.

### 6.6 Key Attributes

A **key attribute** is an attribute (or set of attributes) that uniquely identifies each entity instance. In ER diagrams, key attributes are typically underlined.

- `product_id` is the key attribute of Product
- `customer_id` is the key attribute of Customer
- `category_id` is the key attribute of Category

For weak entities, the key attribute is a **partial key** — it only uniquely identifies instances in combination with the owner entity's key.

### 6.7 Attribute Summary Table

| Attribute Type | Definition | ER Notation | Relational Mapping |
|---|---|---|---|
| Simple | Atomic, indivisible value | Plain oval | Single column |
| Composite | Divisible into sub-attributes | Oval with connected sub-ovals | Multiple columns |
| Multivalued | Multiple values per instance | Double-bordered oval | Separate table |
| Derived | Calculated from other attributes | Dashed oval | Computed in queries (usually not stored) |
| Key | Uniquely identifies entity | Underlined attribute name | PRIMARY KEY column |

---

## 7. Relationships — Expanded

### 7.1 What Is a Relationship?

A **relationship** is a meaningful association between two or more entities. Chapter 8 (Watt & Eng) defines a relationship type as a set of associations among entity types.

In TrailShop:
- A **Category** *contains* many **Products** → relationship between Category and Product
- A **Customer** *places* an **Order** → relationship between Customer and Order
- An **Order** *includes* **OrderItems** → relationship between Order and OrderItem

Each relationship has:
- A **name** (often a verb: "places", "contains", "includes")
- A **degree** (how many entity types participate)
- **Cardinality** (how many instances can participate)
- **Participation** (whether participation is mandatory or optional)

### 7.2 Binary Relationships

A **binary relationship** involves exactly two entity types. This is the most common type.

Examples:
- Customer **places** Order (two entities: Customer, Order)
- Product **belongs to** Category (two entities: Product, Category)
- Order **contains** OrderItem (two entities: Order, OrderItem)

### 7.3 Unary (Recursive) Relationships

A **unary relationship** (also called recursive) involves a single entity type related to itself.

Example 1: **Employee manages Employee**
An employee can manage other employees. The manager is also an employee.

```
┌──────────┐
│ Employee │──── manages ────┐
│          │                 │
│          │◄────────────────┘
└──────────┘
```

Example 2: **Product is_accessory_for Product**
A phone case is an accessory for a phone. Both are products.

Example 3: **Category is_subcategory_of Category**
"Hiking Boots" is a subcategory of "Footwear." Both are categories.

In TrailShop, a recursive relationship could model product accessories or category hierarchies.

### 7.4 Ternary Relationships

A **ternary relationship** involves three entity types simultaneously. These are less common but sometimes necessary.

Example: **Supplier** supplies **Product** to **Warehouse**
You need all three entities to describe the relationship — the same supplier might supply different products to different warehouses.

Ternary relationships are harder to implement in the relational model and are often decomposed into multiple binary relationships with a junction entity.

### 7.5 Strong vs Weak Relationships

**Strong Relationship (Non-Identifying)**
Both entities can exist independently. The relationship links them but neither depends on the other for identification.

- Customer ↔ Order: The customer exists independently. While an order needs a customer, the order has its own `order_id`.

**Weak Relationship (Identifying)**
One entity (the weak entity) depends on the other for its identification. The weak entity's primary key includes the strong entity's primary key.

- Order ↔ OrderItem: An order item is identified by `order_id` + `line_number`. Without the order, the order item has no identity.

Chapter 8 discusses this distinction in the context of strong vs weak entities — a weak entity always participates in an identifying (weak) relationship with its owner.

### 7.6 Identifying vs Non-Identifying Relationships

This terminology is especially important when translating to the relational model:

| Type | Foreign Key Is Part of PK? | Example |
|---|---|---|
| **Identifying** | Yes — FK is part of the child's primary key | `order_items.order_id` is both FK and part of PK |
| **Non-identifying** | No — FK is just a regular column | `products.category_id` is FK but not part of PK |

---

## 8. Cardinality and Participation

### 8.1 What Is Cardinality?

**Cardinality** describes the maximum number of entity instances that can participate in a relationship instance. Chapter 9 (Watt & Eng) covers cardinality, connectivity, and how to derive them from business rules.

The three fundamental cardinality types:

| Cardinality | Meaning | TrailShop Example |
|---|---|---|
| **1:1** (One-to-One) | One A relates to at most one B, and vice versa | One customer has one loyalty profile |
| **1:N** (One-to-Many) | One A relates to many Bs, but each B relates to only one A | One category has many products |
| **M:N** (Many-to-Many) | One A relates to many Bs, and one B relates to many As | Many products appear in many orders |

### 8.2 Mandatory vs Optional Participation

**Participation** (also called **existence dependency**) describes whether every instance of an entity MUST participate in a relationship.

**Mandatory (Total Participation)**
Every instance of the entity MUST participate in the relationship.

- "Every product MUST belong to a category" → Product has mandatory participation in the Product-Category relationship
- "Every order item MUST be part of an order" → OrderItem has mandatory participation

**Optional (Partial Participation)**
Some instances of the entity MAY or MAY NOT participate.

- "A customer MAY place orders (or may have no orders yet)" → Customer has optional participation in the Customer-Order relationship
- "A category MAY have products (or may be empty)" → Category has optional participation

### 8.3 Min-Max (Structural) Notation

A precise way to express participation and cardinality is the **(min, max)** notation, where:
- **min** = the minimum number of times an entity instance must participate (0 = optional, 1+ = mandatory)
- **max** = the maximum number of times an entity instance can participate (1 = at most one, N or * = unlimited)

Examples for TrailShop:

```
Category (0,N) ──── contains ──── (1,1) Product
```

Reading this:
- A Category participates in the "contains" relationship **0 to N** times → a category can have zero or many products
- A Product participates in the "contains" relationship **1 to 1** times → every product belongs to exactly one category

```
Customer (0,N) ──── places ──── (1,1) Order
```

- A Customer places **0 to N** orders (optional, many possible)
- An Order belongs to **1 and only 1** customer (mandatory, exactly one)

### 8.4 Reading Cardinality: The Two-Question Method

For any binary relationship, ask two questions:

1. **"Given one A, how many Bs can it relate to?"** → This gives you the cardinality on the B side.
2. **"Given one B, how many As can it relate to?"** → This gives you the cardinality on the A side.

Example: Category ↔ Product
1. "Given one category, how many products can it have?" → **Many** (0 or more)
2. "Given one product, how many categories can it belong to?" → **One** (exactly one)

Result: Category 1:N Product

Example: Product ↔ Order
1. "Given one product, how many orders can it appear in?" → **Many**
2. "Given one order, how many products can it contain?" → **Many**

Result: Product M:N Order (resolved through OrderItem)

---

## 9. Crow's Foot Notation — Comprehensive Guide

### 9.1 Why Crow's Foot?

There are many ER diagram notations: Chen notation (ovals and diamonds), UML class diagrams, Barker notation, IDEF1X, and crow's foot notation. We use **crow's foot** because:

- It's the most widely used in industry and database tools
- It clearly shows cardinality and participation in a compact way
- Tools like pgModeler, dbdiagram.io, draw.io, Lucidchart, and ERDPlus all support it

### 9.2 The Symbols

Crow's foot notation places symbols at each end of a relationship line. The symbols closest to the entity describe the **maximum** cardinality, and the symbols just inside describe the **minimum** (participation).

**Maximum cardinality symbols (outer, closest to entity):**

| Symbol | Meaning |
|---|---|
| `──────┤` (single line / tick) | Maximum **one** |
| `──────<` (crow's foot / fork) | Maximum **many** |

**Minimum cardinality symbols (inner, next to the max symbol):**

| Symbol | Meaning |
|---|---|
| `──┤` (tick / perpendicular line) | Minimum **one** (mandatory) |
| `──O` (circle / zero) | Minimum **zero** (optional) |

### 9.3 All Four Endpoint Combinations

By combining the minimum and maximum symbols, you get four possible endpoints:

| Endpoint | Min | Max | Meaning | Symbol |
|---|---|---|---|---|
| **Exactly one (mandatory one)** | 1 | 1 | Must participate, only once | `──┤├──` (tick + tick) |
| **Zero or one (optional one)** | 0 | 1 | May not participate, at most once | `──O├──` (circle + tick) |
| **One or many (mandatory many)** | 1 | N | Must participate, can be many | `──┤<──` (tick + crow's foot) |
| **Zero or many (optional many)** | 0 | N | May not participate, can be many | `──O<──` (circle + crow's foot) |

### 9.4 Complete Reference Table of Crow's Foot Combinations

Here is every meaningful combination of endpoints between two entities A and B:

| A side | B side | Relationship Type | Example |
|---|---|---|---|
| `──┤├──` (exactly one) | `──┤<──` (one or many) | 1:N mandatory both sides | Department has employees; every employee in a dept, every dept has ≥1 employee |
| `──┤├──` (exactly one) | `──O<──` (zero or many) | 1:N mandatory A, optional B | Category has products; every product in a category, category may be empty |
| `──O├──` (zero or one) | `──O<──` (zero or many) | 1:N optional both sides | Manager manages employees; employee may have no manager, manager may have no reports |
| `──┤├──` (exactly one) | `──┤├──` (exactly one) | 1:1 mandatory both sides | Country has capital; every country has one, every capital belongs to one country |
| `──┤├──` (exactly one) | `──O├──` (zero or one) | 1:1 mandatory A, optional B | Employee has parking spot; every spot assigned, employee may lack one |
| `──O├──` (zero or one) | `──O├──` (zero or one) | 1:1 optional both sides | Employee has company car; either may exist without the other |
| `──O<──` (zero or many) | `──O<──` (zero or many) | M:N optional both sides | Students enroll in courses; student may have no courses, course may have no students |
| `──┤<──` (one or many) | `──┤<──` (one or many) | M:N mandatory both sides | Actors star in movies; every actor in ≥1 movie, every movie has ≥1 actor |

### 9.5 Reading a Crow's Foot Diagram

When reading a crow's foot diagram, always read **away from** the entity to determine what's on the other side:

```
┌──────────┐                           ┌──────────┐
│ Category │──────────┤├──────O<───────│ Product  │
└──────────┘                           └──────────┘
```

Reading from **Category** (left to right):
- The symbols near Product are `O<` → zero or many
- "One category relates to **zero or many** products"

Reading from **Product** (right to left):
- The symbols near Category are `┤├` → exactly one
- "One product belongs to **exactly one** category"

Combined: "Each category can have zero or many products. Each product belongs to exactly one category." → This is a **1:N** relationship with mandatory participation on the Product side and optional on the Category side.

### 9.6 ASCII Crow's Foot Shorthand

In text-based diagrams (like in this material), we use these conventions:

```
──||──  = exactly one (mandatory one)
──|O──  = zero or one (optional one)  
──|<──  = one or many (mandatory many)
──O<──  = zero or many (optional many)
```

Example:

```
Category ──||──────O<── Product
```

"Each product belongs to exactly one category. Each category has zero or many products."

---

## 10. Resolving Many-to-Many (M:N) Relationships

### 10.1 The Problem

A many-to-many relationship cannot be directly implemented in a relational database. Consider:

- One **Product** can appear in many **Orders**
- One **Order** can contain many **Products**

You can't put `order_id` in the `products` table (a product appears in many orders — which one would you store?). You can't put `product_id` in the `orders` table (an order has many products — which one would you store?).

### 10.2 The Solution: Junction Table

You introduce a new entity — a **junction table** (also called an associative entity, bridge table, linking table, or intersection table) — that sits between the two entities and holds the foreign keys to both.

For TrailShop, the junction table is **OrderItem**:

```
┌──────────┐         ┌────────────┐         ┌──────────┐
│  Order   │──||──|<─┤ OrderItem  ├─>|──||──│ Product  │
└──────────┘         └────────────┘         └──────────┘
```

- Each Order has one or many OrderItems (an order must have at least one item)
- Each OrderItem belongs to exactly one Order
- Each Product appears in zero or many OrderItems (a product may not have been ordered yet)
- Each OrderItem refers to exactly one Product

The junction table typically includes:
- Foreign key to the first entity (`order_id`)
- Foreign key to the second entity (`product_id`)
- Any attributes specific to the relationship (`quantity`, `unit_price` at time of order)

### 10.3 Junction Table Primary Key Options

Chapter 8 discusses two approaches:

**Option A: Composite Primary Key**
```
OrderItem PK = (order_id, product_id)
```
This means a product can appear only once per order. If the customer wants 3 of the same item, you use a `quantity` column.

**Option B: Surrogate Primary Key**
```
OrderItem PK = order_item_id (auto-generated)
```
Plus a unique constraint on `(order_id, product_id)` if needed. This allows more flexibility but adds an extra column.

For TrailShop, we'll use Option A (composite key) — it's cleaner and naturally prevents duplicate product lines in the same order.

### 10.4 More M:N Examples

| Entity A | Entity B | Junction Table | Junction Attributes |
|---|---|---|---|
| Student | Course | Enrollment | enrollment_date, grade |
| Actor | Movie | MovieCast | role_name, billing_order |
| Doctor | Patient | Appointment | appointment_date, diagnosis |
| Author | Book | BookAuthor | author_order |

---

## 11. ER Diagram Best Practices

### 11.1 Naming Conventions

**Entities:**
- Use **singular nouns**: `Customer` not `Customers`, `Product` not `Products`
- Use PascalCase or UPPERCASE: `OrderItem` or `ORDER_ITEM`
- Be specific: `Person` is vague; `Customer`, `Employee`, `Supplier` are specific

**Attributes:**
- Use **lowercase with underscores**: `first_name`, `order_date`, `unit_price`
- Be descriptive: `date` is ambiguous; `order_date`, `ship_date`, `birth_date` are clear
- Prefix foreign keys with the referenced table: `category_id` in `Product`

**Relationships:**
- Use **active verbs**: "places" (Customer places Order), "contains" (Order contains OrderItem), "belongs to" (Product belongs to Category)
- Read the relationship in both directions to verify it makes sense

### 11.2 Layout Tips

- Place the most important/central entity in the middle
- Arrange 1:N relationships so the "one" side is above or to the left
- Minimize crossing lines
- Keep related entities close together
- Use consistent spacing and alignment
- Group entities by business domain if the diagram is large

### 11.3 Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---|---|---|
| Storing multivalued data in one attribute | Violates atomicity (1NF) | Create a separate entity/table |
| Missing key attributes | Can't uniquely identify instances | Add a primary key attribute |
| Vague relationship names | "has" or "uses" — too generic | Use specific verbs: "places", "belongs to" |
| Wrong cardinality | Misunderstanding business rules | Re-read requirements, ask stakeholders |
| Forgetting participation constraints | Unclear if participation is mandatory or optional | Always specify min cardinality |
| M:N not resolved | Can't implement directly | Add junction entity |
| Redundant relationships | A→B and A→C→B when one path suffices | Remove the redundant path |
| Attributes on M:N relationships | Only valid for junction entity attributes | Move to junction entity |

---

## 12. The TrailShop ER Model — Complete

### 12.1 Entities and Their Attributes

Let's build the complete ER model for TrailShop's core business.

**Category** (Strong Entity)
- `category_id` (PK) — unique identifier
- `category_name` — name of the category (e.g., "Footwear", "Camping")
- `description` — optional text describing the category

**Product** (Strong Entity)
- `product_id` (PK) — unique identifier
- `name` — product name
- `description` — detailed product description
- `price` — current selling price
- `weight_kg` — product weight in kilograms (optional)
- `stock_quantity` — current inventory count
- `created_at` — when the product was added

**Customer** (Strong Entity)
- `customer_id` (PK) — unique identifier
- `first_name` — customer's first name
- `last_name` — customer's last name
- `email` — unique email address
- `phone` — phone number (optional)
- `street` — street address
- `city` — city
- `postal_code` — postal/ZIP code
- `country` — country
- `registered_at` — registration timestamp

**Order** (Dependent Entity)
- `order_id` (PK) — unique identifier
- `order_date` — when the order was placed
- `status` — order status (e.g., "pending", "shipped", "delivered")
- `shipping_address` — delivery address (could be composite)

**OrderItem** (Weak Entity — depends on Order)
- `order_id` (PK, FK) — references Order
- `product_id` (PK, FK) — references Product
- `quantity` — number of units ordered
- `unit_price` — price at time of order (snapshot, not derived from current product price)

### 12.2 Relationships

| Relationship | Entities | Cardinality | Participation |
|---|---|---|---|
| "belongs to" | Product → Category | N:1 | Mandatory (every product has a category) |
| "contains" | Category → Product | 1:N | Optional (category may be empty) |
| "places" | Customer → Order | 1:N | Optional (customer may have no orders) |
| "belongs to" | Order → Customer | N:1 | Mandatory (every order has a customer) |
| "contains" | Order → OrderItem | 1:N | Mandatory (every order has ≥1 item) |
| "belongs to" | OrderItem → Order | N:1 | Mandatory (identifying relationship) |
| "references" | OrderItem → Product | N:1 | Mandatory (every item is a product) |
| "appears in" | Product → OrderItem | 1:N | Optional (product may not be ordered yet) |

### 12.3 ASCII ER Diagram

```
┌─────────────────┐         ┌─────────────────────┐
│    CATEGORY      │         │      PRODUCT         │
├─────────────────┤         ├─────────────────────┤
│ category_id (PK)│         │ product_id (PK)      │
│ category_name   │         │ name                 │
│ description     │         │ description          │
│                 │         │ price                │
│                 │         │ weight_kg            │
│                 │         │ stock_quantity       │
│                 │         │ created_at           │
│                 │         │ category_id (FK)     │
└────────┬────────┘         └──────────┬──────────┘
         │                             │
         │  1          contains      N │
         └─────────────────────────────┘

┌─────────────────┐         ┌─────────────────────┐
│    CUSTOMER      │         │       ORDER          │
├─────────────────┤         ├─────────────────────┤
│ customer_id (PK)│         │ order_id (PK)        │
│ first_name      │         │ order_date           │
│ last_name       │         │ status               │
│ email           │         │ shipping_address     │
│ phone           │         │ customer_id (FK)     │
│ street          │         │                      │
│ city            │         └──────────┬───────────┘
│ postal_code     │                    │
│ country         │                    │ 1
│ registered_at   │                    │ contains
│                 │                    │ N
└────────┬────────┘         ┌──────────┴───────────┐
         │                  │     ORDER_ITEM        │
         │  1    places   N │ (weak entity)         │
         └─────────────────>├──────────────────────┤
                            │ order_id (PK, FK)    │
                            │ product_id (PK, FK)  │
                            │ quantity             │
                            │ unit_price           │
                            └──────────────────────┘
```

**Complete relationships with crow's foot:**

```
Category ──||──────O<── Product
Customer ──||──────O<── Order
Order    ──||──────|<── OrderItem
Product  ──||──────O<── OrderItem
```

Reading:
- "Each product belongs to exactly one category. Each category has zero or many products."
- "Each order belongs to exactly one customer. Each customer has zero or many orders."
- "Each order item belongs to exactly one order. Each order has one or many order items."
- "Each order item references exactly one product. Each product appears in zero or many order items."

### 12.4 Design Decisions Explained

Chapter 13 (Watt & Eng) provides guidelines for database design projects. Here are our key design decisions:

1. **Why `unit_price` in OrderItem?** — Product prices change over time. We store the price *at the time of the order* to preserve historical accuracy. If we just referenced the current product price, old orders would show wrong totals.

2. **Why composite PK for OrderItem?** — `(order_id, product_id)` naturally prevents the same product appearing twice in one order. The `quantity` attribute handles "3 of the same item."

3. **Why is `phone` optional?** — Not every customer provides a phone number. Business rule: phone is nice-to-have, not required.

4. **Why separate Category entity?** — Instead of storing category name as a text field in Product, a separate entity avoids data redundancy and ensures consistent naming.

5. **Why `weight_kg` is optional?** — Some products (like gift cards or digital items) might not have a meaningful weight.

---

## Key Terms

| Term | Definition |
|---|---|
| **Data model** | An abstract representation of data structures, relationships, and constraints |
| **Conceptual model** | A high-level, technology-independent model of business data and relationships |
| **Logical model** | A model that maps conceptual structures to a specific data model type (e.g., relational) |
| **Physical model** | A model that specifies storage details for a specific DBMS |
| **Three-schema architecture** | ANSI-SPARC framework with external, conceptual, and internal levels |
| **Data independence** | Ability to change one schema level without affecting others |
| **Entity** | A real-world thing about which data is stored |
| **Entity type** | A collection of entities sharing the same attributes |
| **Entity instance** | One specific occurrence of an entity type |
| **Strong entity** | An entity that can be uniquely identified by its own attributes |
| **Weak entity** | An entity that depends on another entity for identification |
| **Attribute** | A property that describes an entity |
| **Simple attribute** | An atomic, indivisible attribute |
| **Composite attribute** | An attribute that can be subdivided into meaningful parts |
| **Multivalued attribute** | An attribute that can hold multiple values per entity instance |
| **Derived attribute** | An attribute whose value is computed from other attributes |
| **Key attribute** | An attribute that uniquely identifies an entity instance |
| **Relationship** | A meaningful association between entity types |
| **Binary relationship** | A relationship between two entity types |
| **Unary relationship** | A relationship where an entity type relates to itself |
| **Ternary relationship** | A relationship involving three entity types |
| **Identifying relationship** | A relationship where the child's PK includes the parent's PK |
| **Cardinality** | The maximum number of instances in a relationship (1:1, 1:N, M:N) |
| **Participation** | Whether entity instances must (mandatory) or may (optional) participate |
| **Crow's foot notation** | An ER diagram notation using fork symbols to show cardinality |
| **Junction table** | A table that resolves an M:N relationship into two 1:N relationships |
| **ER diagram** | A visual representation of entities, attributes, and relationships |

---

## Reading Assignments

**Required:**
- *Database Design*, 2nd Edition — Chapters 5 (Data Modelling), 8 (ER Data Model), and 9 (Cardinality)
- Chapter 4 (Types of Data Models) — Sections on the ER model

**PostgreSQL Reference (for context — implementation comes next week):**
- PostgreSQL Docs: Data Definition — https://www.postgresql.org/docs/current/ddl.html

---

## Further Reading

- Peter Chen's original 1976 paper: "The Entity-Relationship Model: Toward a Unified View of Data" — https://dl.acm.org/doi/10.1145/320434.320440
- Lucidchart: ER Diagram Tutorial — https://www.lucidchart.com/pages/er-diagrams
- Visual Paradigm: ER Diagram Notation Reference — https://www.visual-paradigm.com/guide/data-modeling/what-is-entity-relationship-diagram/
- *Database Design*, 2nd Edition — Chapter 13 (Database Development Process) for the broader context of where ER modelling fits

---

## Summary

You've learned to think before building. Before writing a single `CREATE TABLE` statement, you now know how to step back and model the business domain using entities, attributes, and relationships. You understand the different levels of data abstraction — from external views down to physical storage — and why the three-schema architecture exists to protect against unnecessary changes rippling through the system.

You can classify entities (strong vs weak, independent vs dependent vs characteristic), attributes (simple, composite, multivalued, derived, key), and relationships (binary, unary, ternary, identifying vs non-identifying). You know how to read and apply cardinality constraints using crow's foot notation, and you know that many-to-many relationships must be resolved through junction tables.

Most importantly, you've built a complete ER model for TrailShop — five entities with all their attributes and relationships mapped out. This model is your blueprint.

**Next week:** You'll take this blueprint and transform it into actual PostgreSQL tables. You'll learn the transformation rules that convert an ER diagram into a relational schema, choose data types, define constraints, and write the `CREATE TABLE` statements that bring TrailShop's database to life.
