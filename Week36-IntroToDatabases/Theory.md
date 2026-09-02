# Week 36 — Introduction to Databases: Theory

## Chapter 1: "Welcome to TrailShop"

Congratulations — you've just been hired as a junior developer at **TrailShop**, a growing online store that sells outdoor equipment: hiking boots, tents, backpacks, climbing gear, and more. Business is booming, but behind the scenes things are chaotic. The founders have been tracking everything in spreadsheets and text files scattered across laptops and shared drives. Product info is duplicated in three different files. Prices don't match between the website sheet and the inventory sheet. When two people try to edit the same file at the same time, someone's changes get lost.

Your first mission isn't to write code — it's to understand *why* this mess exists and what a proper database can do about it.

This week's material covers the foundational concepts you'll need for everything that follows in this course. Take your time with it — the ideas here are not difficult individually, but together they form the lens through which you'll see every database topic from here on.

---

## Learning Objectives

By the end of this chapter you will be able to:

- Distinguish between data and information and explain why the distinction matters
- Identify and explain the five major problems of a file-based approach to data management
- Define what a database and a DBMS are, and describe the DBMS's core functions
- Explain the three-schema architecture and the concept of data independence
- Describe the key characteristics and benefits of the database approach
- Outline the historical evolution of database systems
- Compare different types of DBMS (relational, object-oriented, distributed)
- Explain why PostgreSQL is used in this course and describe its key features
- Set up PostgreSQL and create your first database

---

## 1. Data vs Information

**Data** consists of raw, unprocessed facts — numbers, text, dates — that on their own carry little meaning. For example:

```
"TrailMaster X4", 149.99, 12
```

That's data. Three values sitting next to each other with no context. Is 149.99 a price? A weight? A distance? Is 12 the quantity in stock, or the product's rating out of 20?

It becomes **information** when you give it context and structure:

> "The product *TrailMaster X4* is priced at €149.99 and we have 12 units in stock."

Information is data that has been processed, organized, and presented in a way that is useful for decision-making. As Watt and Eng explain in Chapter 1 of *Database Design* (2nd Ed.), "Data consists of raw facts... Information is the result of processing raw data to reveal its meaning" (Watt & Eng, Ch. 1).

### Why This Distinction Matters

At TrailShop, the founders have plenty of *data* — thousands of rows in spreadsheets. But when the CEO asks "Which product category generated the most revenue last quarter?", nobody can answer quickly. The data exists, but it hasn't been structured in a way that makes extracting information easy. A well-designed database bridges this gap.

### The Data–Information–Knowledge Hierarchy

It's useful to think of a hierarchy:

1. **Data** — raw facts: `"TrailMaster X4", 149.99, 12`
2. **Information** — data with context: "TrailMaster X4 costs €149.99 and has 12 units in stock"
3. **Knowledge** — information combined with experience: "TrailMaster X4 sells well in spring; we should increase stock by March"
4. **Wisdom** — knowledge applied with judgment: "We should negotiate a bulk discount with the tent supplier before the spring rush"

A database handles levels 1 and 2. Levels 3 and 4 are where humans (and business intelligence tools) come in — but they can only work if the data foundation is solid.

---

## 2. The File-Based System and Its Problems

Before databases existed, organizations stored data in separate files managed by individual programs. Each department would create its own files, write its own programs to access them, and manage them independently. As Watt describes in Chapter 1 of *Database Design* (2nd Ed.), "the file-based system was an early attempt to computerize the manual filing system" (Watt & Eng, Ch. 1).

TrailShop's founders are essentially doing the same thing with their spreadsheets. Let's look at what goes wrong — in detail.

### 2.1 Data Redundancy and Inconsistency

**What it is:** The same piece of data is stored in multiple places. When one copy is updated but the others aren't, you get **data inconsistency** — different versions of the same fact.

**TrailShop scenario:** The product "Alpine Pro Hiking Boots" appears in three files:

| File | Price Listed | Stock Listed |
|---|---|---|
| `inventory.xlsx` | €189.50 | 42 |
| `website_products.csv` | €179.00 (sale price was supposed to end last week) | 38 |
| `orders.txt` | €189.50 | (not tracked here) |

A customer sees €179.00 on the website, places an order, and then the warehouse ships at €189.50. The customer complains. The marketing team blames the web team. The web team blames whoever forgot to update the CSV. Nobody wins.

**Root cause:** There is no single authoritative source of truth. Each file is an independent copy.

**The database solution:** Store each fact exactly once. If the price of hiking boots is €189.50, that value lives in one place — the `products` table. Every application reads from the same source.

### 2.2 Data Isolation

**What it is:** Data is scattered across different files, possibly in different formats. Combining data from multiple sources requires writing custom scripts or doing manual copy-paste.

**TrailShop scenario:** The sales team wants to find out which customers ordered camping gear in the last month. Customer data is in `customers.xlsx`. Order data is in `orders.txt` (a pipe-delimited text file that an old POS system exports). Product categories are in a Google Sheet. To answer this question, someone has to:

1. Export the Google Sheet as CSV
2. Write a Python script to parse the pipe-delimited orders file
3. Match customer IDs between the spreadsheet and the text file
4. Filter by date and category
5. Hope that the customer IDs are consistent across files (spoiler: they're not always)

This takes hours. In a database, the same answer takes one SQL query and a few seconds.

**Root cause:** Data is trapped in silos with incompatible formats.

### 2.3 Integrity Problems

**What it is:** There is no mechanism to enforce rules about what data is valid. Invalid data enters the system because nothing prevents it.

**TrailShop scenario:** An intern enters a new product into `inventory.xlsx`:

| product_id | name | price | stock | category |
|---|---|---|---|---|
| 42 | | -15.00 | abc | |

No name, a negative price, a non-numeric stock value, and no category. In a spreadsheet, this row is accepted without complaint. Nobody notices until a week later when the website shows a product with no name priced at minus fifteen euros.

**Root cause:** Spreadsheets have no concept of mandatory fields, data types, or validation rules that are enforced automatically and consistently.

**The database solution:** Constraints. You define rules like `price > 0`, `name NOT NULL`, `stock_quantity >= 0` — and the DBMS rejects any data that violates them, instantly.

### 2.4 Security Problems

**What it is:** You cannot easily control who sees what data, or who can modify it. Access control is all-or-nothing: either someone can open the file, or they can't.

**TrailShop scenario:** The marketing team needs access to the product catalog to update descriptions. But the same spreadsheet also contains wholesale costs, supplier contact details, and profit margins — confidential data the marketing team shouldn't see. The founders share the file anyway because there's no way to hide individual columns.

**Root cause:** File-system permissions operate at the file level, not at the data level.

**The database solution:** Role-based access control. You can grant the marketing team `SELECT` permission on `name`, `description`, and `price` columns, while restricting `wholesale_cost` and `supplier_id` to the procurement team.

### 2.5 Concurrency Access Issues

**What it is:** When multiple users try to read and write the same data simultaneously, changes can be lost or data can become corrupted.

**TrailShop scenario:** It's Monday morning. The warehouse manager opens `inventory.xlsx` to update stock levels after a weekend shipment. At the same time, the procurement officer opens the same file to update supplier prices. They both make changes for 30 minutes. The warehouse manager saves first. Then the procurement officer saves — and overwrites all of the warehouse manager's changes. The founders call this "the Monday morning spreadsheet fight."

**Root cause:** Files support only one writer at a time (or they don't, and data gets corrupted).

**The database solution:** Concurrency control. The DBMS uses techniques like locking and multi-version concurrency control (MVCC) to allow multiple users to work with the same data simultaneously without conflicts. PostgreSQL, specifically, uses MVCC — you will learn about this in detail in Week 46.

### 2.6 Dependency on Programs

There's one more problem that Watt & Eng highlight (Ch. 1): **program-data dependence**. In a file-based system, the structure of the data is embedded in the programs that access it. If you change the format of a file (say, you add a column to your CSV), every program that reads that file must be updated. This makes maintenance expensive and error-prone.

---

## 3. The Database Approach

### 3.1 What Is a Database?

A **database** is a shared collection of logically related data, designed to meet the information needs of an organization. As defined in *Database Design* (2nd Ed.), a database represents some aspect of the real world and is designed for a specific purpose (Watt & Eng, Ch. 1).

A database has these properties:

- It represents some aspect of the real world (TrailShop's products, customers, orders)
- It is a logically coherent collection — not just a random dump of files
- It is designed and built for a specific purpose
- Data is stored in **fields** (individual pieces of data)
- Fields are grouped into **tables** (structured collections of related fields)

### 3.2 What Is a DBMS?

A **Database Management System (DBMS)** is a collection of programs that enables you to create, maintain, and control access to a database. It sits between your applications and the stored data, handling all the complexity for you (Watt & Eng, Ch. 1).

Think of it like this: a spreadsheet file is a notebook you carry around. A DBMS is a professional librarian who catalogs everything, enforces borrowing rules, and makes sure nothing gets lost.

The key idea is **abstraction**: application programs don't need to know how data is physically stored. They communicate with the DBMS, and the DBMS handles the rest.

### 3.3 The Three-Schema Architecture

One of the most important concepts in database theory is the **three-schema architecture** (also called the ANSI/SPARC architecture). It separates the database into three levels of abstraction, as described in *Database Design* (2nd Ed., Ch. 2):

```
┌─────────────────────────────────────────────────────┐
│               External Level                         │
│  (Individual user views — what each user/app sees)   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐          │
│  │ Marketing│  │ Warehouse│  │   Web    │          │
│  │   View   │  │   View   │  │   App    │          │
│  └──────────┘  └──────────┘  └──────────┘          │
├─────────────────────────────────────────────────────┤
│               Conceptual Level                       │
│  (The community view — the full logical structure    │
│   of the entire database, independent of physical    │
│   storage)                                           │
├─────────────────────────────────────────────────────┤
│               Internal Level                         │
│  (Physical storage — how data is actually stored     │
│   on disk: file organization, indexes, compression)  │
└─────────────────────────────────────────────────────┘
```

**External level (view level):** Each user or application sees only the data relevant to them. The marketing team sees product names, descriptions, and prices. The warehouse team sees product IDs, stock quantities, and shelf locations. Both are looking at the same underlying data, but through different windows.

**Conceptual level (logical level):** This is the complete logical structure of the entire database — all tables, columns, data types, relationships, and constraints. It describes *what* data is stored and how it's related, without specifying *how* it's physically stored. When you write `CREATE TABLE products (...)`, you're working at the conceptual level.

**Internal level (physical level):** This is how data is actually stored on disk — file formats, index structures, data compression, buffer management. You rarely interact with this level directly; the DBMS handles it for you.

### 3.4 Data Independence

The three-schema architecture enables **data independence** — the ability to change one level without affecting the others. This is a central advantage of the database approach (Watt & Eng, Ch. 2).

**Logical data independence:** You can change the conceptual schema (e.g., add a new column to a table) without affecting the external views or application programs that don't use that column.

**Physical data independence:** You can change how data is physically stored (e.g., move the database to a faster disk, add an index) without changing the logical structure or the applications.

**TrailShop example:** Suppose you add a `weight_kg` column to the `products` table. The warehouse application, which was already designed to show weight, can immediately display it. The marketing application, which doesn't care about weight, continues working without any changes. That's logical data independence in action.

### 3.5 The DBMS as a Gatekeeper

The DBMS sits between applications and the physical data, acting as a gatekeeper. No application accesses data directly — everything goes through the DBMS. This allows the DBMS to:

- Check that all integrity constraints are satisfied before accepting data
- Verify that the user has permission to perform the requested operation
- Coordinate simultaneous access by multiple users
- Maintain a log of all operations for backup and recovery
- Optimize query execution for performance

---

## 4. DBMS Core Functions

A DBMS does far more than just store data. Here are its core functions, with TrailShop examples for each. These functions are described across Chapters 1–3 of *Database Design* (2nd Ed.).

### 4.1 Data Definition

The DBMS provides a **Data Definition Language (DDL)** for defining the structure of the database — tables, columns, data types, constraints, and relationships.

```sql
-- DDL example: defining a TrailShop table
CREATE TABLE products (
    product_id     INTEGER       PRIMARY KEY,
    name           VARCHAR(100)  NOT NULL,
    price          NUMERIC(10,2) NOT NULL CHECK (price > 0),
    stock_quantity INTEGER       NOT NULL DEFAULT 0
);
```

### 4.2 Data Manipulation

The DBMS provides a **Data Manipulation Language (DML)** for inserting, updating, deleting, and querying data.

```sql
-- DML examples
INSERT INTO products VALUES (1, 'TrailMaster X4 Tent', 249.99, 15);
SELECT name, price FROM products WHERE price < 200;
UPDATE products SET stock_quantity = stock_quantity - 1 WHERE product_id = 1;
DELETE FROM products WHERE product_id = 999;
```

### 4.3 The Data Dictionary (System Catalog)

Every DBMS maintains a **data dictionary** (also called a **system catalog**) — a special set of tables that stores metadata: information about the database's own structure. This includes table names, column names, data types, constraints, indexes, users, and permissions.

In PostgreSQL, you can peek at the data dictionary:

```sql
-- List all tables in the current database
SELECT table_name FROM information_schema.tables
WHERE table_schema = 'public';

-- Show column details for the products table
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'products';
```

This is what makes a database **self-describing** — the database contains not only the data but also a complete description of its own structure. Compare this to a CSV file: if you open `inventory.csv`, you see column headers and values, but there's no information about data types, constraints, or relationships. You have to know (or guess) what each column means.

### 4.4 Concurrency Control

When multiple users access the database simultaneously, the DBMS ensures their operations don't interfere with each other. It uses techniques such as:

- **Locking:** Temporarily preventing other users from modifying data that is being changed
- **MVCC (Multi-Version Concurrency Control):** Creating "snapshots" of data so each user sees a consistent view, even while others are making changes. PostgreSQL uses MVCC.

**TrailShop example:** The warehouse manager reduces stock for product 101 at the same time a customer places an order for the same product through the website. Without concurrency control, both operations might read stock as 42, each subtract 1, and both write 41 — losing one decrement. With concurrency control, the final stock correctly becomes 40.

### 4.5 Backup and Recovery

The DBMS provides mechanisms to recover data after failures — hardware crashes, power outages, software bugs, or human error.

- **Transaction logging:** Every change is recorded in a log. If a failure occurs, the DBMS can replay or undo operations to restore a consistent state.
- **Point-in-time recovery:** You can restore the database to any specific moment in time.

**TrailShop example:** A developer accidentally runs `DELETE FROM products;` without a `WHERE` clause. Panic! But with proper backups and transaction logs, the DBA restores the database to the state from five minutes before the deletion. No data is permanently lost.

### 4.6 Security and Authorization

The DBMS controls who can do what:

```sql
-- Create a role for the marketing team
CREATE ROLE marketing_team;

-- Grant read-only access to product names and prices
GRANT SELECT (product_id, name, price) ON products TO marketing_team;

-- Deny access to wholesale cost data
-- (By not granting it, it's automatically denied)
```

This is vastly more granular than file-system permissions.

### 4.7 Integrity Enforcement

The DBMS automatically checks constraints on every data modification:

```sql
-- This will be REJECTED because price must be > 0
INSERT INTO products (product_id, name, price, stock_quantity)
VALUES (999, 'Free Item', 0, 100);
-- ERROR: new row for relation "products" violates check constraint
```

You don't need to write validation code in every application — the database enforces the rules centrally. You will practice this in the exercises.

---

## 5. Characteristics and Benefits of the Database Approach

Let's expand on the key characteristics introduced in *Database Design* (2nd Ed., Ch. 1–3):

### 5.1 Self-Describing Nature

A database contains both the data and a description of its own structure (metadata). When you connect to the TrailShop database and run `\dt` in psql, you see a list of tables. Run `\d products` and you see columns, data types, and constraints. This information is stored in the system catalog.

**TrailShop example:** A new developer joins TrailShop. They connect to the database and explore:

```
trailshop=# \dt
         List of relations
 Schema |    Name     | Type  | Owner
--------+-------------+-------+-------
 public | categories  | table | admin
 public | products    | table | admin

trailshop=# \d products
                   Table "public.products"
     Column      |          Type          | Nullable | Default
-----------------+------------------------+----------+---------
 product_id      | integer                | not null |
 name            | character varying(100) | not null |
 price           | numeric(10,2)          | not null |
 stock_quantity  | integer                | not null | 0
 category_id     | integer                | not null |
```

Without reading any documentation, the new developer already knows the table structure. The database is self-documenting.

### 5.2 Program-Data Independence

In a file-based system, programs are tightly coupled to the structure of the data files they read. If you add a column to a CSV, every script that parses that CSV might break.

In a database system, applications interact with data through the DBMS using SQL. If you add a column to a table, existing queries that don't reference that column continue to work unchanged.

**TrailShop example:** The `products` table originally has five columns. Later, you add a `weight_kg` column:

```sql
ALTER TABLE products ADD COLUMN weight_kg NUMERIC(6,2);
```

The website's product listing query — `SELECT name, price FROM products;` — continues to work perfectly. It doesn't know or care that `weight_kg` was added. That's program-data independence.

### 5.3 Multiple Views

Different users need to see different subsets or combinations of data. The DBMS supports this through **views** — virtual tables defined by queries.

```sql
-- A view for the marketing team: products with category names
CREATE VIEW marketing_products AS
SELECT p.name, p.price, c.category_name
FROM products p
JOIN categories c ON p.category_id = c.category_id;

-- A view for the warehouse: stock levels only
CREATE VIEW warehouse_stock AS
SELECT product_id, name, stock_quantity
FROM products;
```

Each team sees exactly what they need.

### 5.4 Multi-User Support with Concurrency Control

As discussed in Section 4.4, the DBMS handles simultaneous access. Multiple users can read and write data concurrently without corrupting it.

### 5.5 Transaction Processing

A **transaction** is a logical unit of work that must either complete entirely or not at all. This property is called **atomicity** and is part of the ACID properties (Atomicity, Consistency, Isolation, Durability) — a cornerstone of reliable database systems.

**TrailShop example:** A customer places an order. This involves:
1. Insert a row into the `orders` table
2. Insert rows into the `order_items` table
3. Decrease `stock_quantity` in `products` for each item ordered
4. Charge the customer's payment method

If step 3 succeeds but step 4 fails (payment declined), you don't want the stock to be decremented. A transaction groups all four steps: if any step fails, all changes are rolled back.

```sql
BEGIN;
  INSERT INTO orders (order_id, customer_id, order_date) VALUES (1001, 55, NOW());
  INSERT INTO order_items (order_id, product_id, quantity) VALUES (1001, 101, 1);
  UPDATE products SET stock_quantity = stock_quantity - 1 WHERE product_id = 101;
  -- If payment fails here, we ROLLBACK and nothing is saved
COMMIT;
```

You will study transactions in depth in Week 46.

### 5.6 Controlled Redundancy

Databases don't eliminate redundancy entirely — sometimes intentional duplication improves performance (e.g., denormalization). But the redundancy is *controlled* and *documented*, unlike the chaotic duplication of file-based systems.

### Summary Table

| Characteristic | What It Means for TrailShop |
|---|---|
| **Self-describing nature** | The database stores not just data but also a description of its own structure (metadata) |
| **Program-data independence** | If you change how product data is stored, your applications don't break |
| **Multiple views** | The warehouse team sees stock levels; marketing sees product descriptions |
| **Multi-user support** | Several employees can work with the data simultaneously without conflicts |
| **Controlled redundancy** | Duplication is minimized and managed intentionally |
| **Data sharing** | All departments access one authoritative source of truth |
| **Integrity constraints** | Rules like "price > 0" are enforced automatically |
| **Access control** | Fine-grained permissions: who can read, insert, update, delete |
| **Data independence** | Physical storage can change without affecting applications |
| **Transaction processing** | Operations either complete fully or not at all — no half-done updates |
| **Backup and recovery** | The system can restore data after failures |

---

## 6. A Brief History of Database Systems

Understanding where databases came from helps you appreciate why the relational model dominates today. As discussed in *Database Design* (2nd Ed., Ch. 6), database systems have evolved through several generations.

### 6.1 Flat Files (1950s–1960s)

The earliest computer systems stored data in plain text files — no structure, no relationships, no query language. Programs read files sequentially from beginning to end. This was fine for simple applications (payroll, inventory lists) but became unmanageable as data grew.

### 6.2 Hierarchical Model (1960s)

IBM's **IMS (Information Management System)**, released in 1966, introduced the hierarchical model. Data was organized in a tree structure — parent-child relationships only. Navigating the tree required knowing its structure in advance.

**Limitation:** If your data didn't fit a strict tree hierarchy (e.g., a product belonging to multiple categories), the model became awkward.

### 6.3 Network Model (1960s–1970s)

The **CODASYL** committee proposed the network model, which allowed more flexible relationships — a child could have multiple parents. This solved some limitations of the hierarchical model but introduced complexity: programmers had to navigate data by following pointers between records.

### 6.4 Relational Model (1970–present)

In 1970, **Edgar F. Codd**, a researcher at IBM, published the paper "A Relational Model of Data for Large Shared Data Banks." This was a revolution. Codd proposed:

- Data should be organized in **relations** (tables)
- Relationships should be expressed through shared values (keys), not physical pointers
- A high-level, declarative language should be used to query data (what later became SQL)
- The mathematical foundation of set theory and predicate logic should guarantee correctness

The relational model was simpler, more flexible, and more theoretically grounded than anything before it. By the 1980s, relational databases (Oracle, DB2, Ingres) dominated the market. Today, the relational model is still the foundation of the vast majority of business databases.

You will study the relational model in detail starting in Week 37.

### 6.5 Object-Relational and Object-Oriented (1990s)

As object-oriented programming became popular, researchers explored databases that could store complex objects directly (not just flat rows and columns). Pure object-oriented databases never achieved mainstream adoption, but **object-relational** extensions were added to existing systems. PostgreSQL, in fact, started as an object-relational research project at UC Berkeley.

### 6.6 NoSQL and NewSQL (2000s–present)

With the rise of web-scale applications (Google, Facebook, Amazon), new database types emerged to handle massive volumes of data with flexible schemas:

- **Document databases** (MongoDB): store data as JSON-like documents
- **Key-value stores** (Redis): simple key → value lookups, extremely fast
- **Column-family stores** (Cassandra): optimized for distributed, write-heavy workloads
- **Graph databases** (Neo4j): optimized for data with complex relationships (social networks, recommendation engines)

These are collectively called **NoSQL** ("Not Only SQL") databases. They trade some relational guarantees (like ACID transactions) for scalability and flexibility. However, relational databases remain the best choice for most business applications where data integrity and complex queries are important.

### Timeline Summary

| Era | Model | Key Systems |
|---|---|---|
| 1950s–60s | Flat files | Custom programs |
| 1960s | Hierarchical | IBM IMS |
| 1960s–70s | Network | CODASYL, IDMS |
| 1970–present | Relational | Oracle, PostgreSQL, MySQL, SQL Server |
| 1990s | Object-relational | PostgreSQL, Oracle |
| 2000s–present | NoSQL | MongoDB, Redis, Cassandra, Neo4j |

---

## 7. Types of DBMS

As described in *Database Design* (2nd Ed., Ch. 6), database management systems can be classified in several ways.

### 7.1 By Data Model

**Relational DBMS (RDBMS):** The most common type. Data is stored in tables with rows and columns. Relationships are expressed through foreign keys. Queried with SQL. Examples: PostgreSQL, MySQL, Oracle, Microsoft SQL Server, SQLite.

**Object-Oriented DBMS (OODBMS):** Data is stored as objects (as in object-oriented programming). Supports inheritance, encapsulation, and polymorphism. Examples: ObjectDB, db4o. Rare in practice.

**Document DBMS:** Data is stored as semi-structured documents (typically JSON or BSON). Schema is flexible — different documents in the same collection can have different fields. Examples: MongoDB, CouchDB.

**Graph DBMS:** Data is stored as nodes and edges, optimized for traversing relationships. Examples: Neo4j, Amazon Neptune.

### 7.2 By Number of Users

- **Single-user:** Only one user at a time (e.g., SQLite on a mobile device)
- **Multi-user:** Many users simultaneously (e.g., PostgreSQL serving a web application)

### 7.3 By Distribution

- **Centralized:** The DBMS and data reside on a single machine
- **Distributed:** Data is spread across multiple machines, possibly in different locations. The DBMS coordinates access transparently.
- **Cloud-based:** The database runs in the cloud (e.g., Amazon RDS, Google Cloud SQL, Azure Database for PostgreSQL). You manage the data; the cloud provider manages the infrastructure.

### 7.4 Why These Categories Matter

For TrailShop, you'll use a **relational, multi-user, centralized** DBMS — PostgreSQL. But as TrailShop grows, they might add a document database for product reviews (flexible schema) or a cache layer with Redis (key-value store for speed). Understanding the landscape helps you make informed decisions.

---

## 8. Tables, Rows, Columns, Schemas, and Metadata

Let's look at how TrailShop's data would look in a database. Here are two related tables:

**categories**

| category_id | category_name | description |
|---|---|---|
| 1 | Footwear | Boots, shoes, and socks for outdoor activities |
| 2 | Camping | Tents, sleeping bags, and camp accessories |
| 3 | Climbing | Harnesses, ropes, and climbing shoes |
| 4 | Hiking | Backpacks, trekking poles, and accessories |
| 5 | Clothing | Jackets, pants, and layering gear |

**products**

| product_id | name | price | stock_quantity | category_id |
|---|---|---|---|---|
| 1 | TrailMaster X4 Tent | 249.99 | 15 | 2 |
| 2 | Alpine Pro Hiking Boots | 189.50 | 42 | 1 |
| 3 | Summit 45L Backpack | 129.00 | 28 | 4 |
| 4 | GripWall Climbing Shoes | 159.99 | 19 | 3 |
| 5 | StormShield Rain Jacket | 99.95 | 55 | 5 |

Notice how the `category_id` column in `products` links each product to a row in `categories`. This is a **foreign key** relationship — you'll study it formally in Week 37.

### Key Terminology

- **Table** (also called a relation): the entire structure above — named, with defined columns
- **Row** (also called a record or tuple): one horizontal entry, e.g., the data for "Alpine Pro Hiking Boots"
- **Column** (also called a field or attribute): one vertical category of data, e.g., `price`
- **Schema**: the design of the database — which tables exist, what columns they have, what types of data each column holds, and what constraints apply
- **Metadata**: "data about data" — the schema itself is metadata. It tells you that `price` is a numeric column, `name` is text, `category_id` is an integer that references the `categories` table, etc.

### Metadata Examples

| Metadata | What It Tells You |
|---|---|
| Table name: `products` | A table called "products" exists |
| Column: `price`, Type: `NUMERIC(10,2)` | The price column stores decimal numbers with up to 10 digits, 2 after the decimal point |
| Constraint: `price > 0` | Prices must be positive |
| Constraint: `category_id REFERENCES categories` | Each product must belong to an existing category |
| Column: `name`, Nullable: `NO` | Every product must have a name |

---

## 9. Database vs Spreadsheet: A Detailed Comparison

You might wonder: "Can't I just use Excel for everything?" For small, single-user scenarios, sure. But here's why databases win as complexity grows:

| Aspect | Text Files | Spreadsheets | Database (DBMS) |
|---|---|---|---|
| **Structure** | None / ad hoc | Grid of cells | Formally defined schema |
| **Data types** | Everything is text | Basic (number, text, date) | Rich types (INTEGER, NUMERIC, VARCHAR, BOOLEAN, DATE, TIMESTAMP, JSON, arrays, etc.) |
| **Data integrity** | No enforcement | Minimal (some cell validation) | Constraints enforced automatically on every operation |
| **Multi-user access** | Very difficult | Limited (file locking, conflict-prone) | Built-in concurrency control (MVCC in PostgreSQL) |
| **Security** | File-system level only | Sheet protection (easily bypassed) | Role-based access control, column-level permissions |
| **Data volume** | Becomes unwieldy fast | ~1 million rows max (Excel) | Billions of rows with proper indexing |
| **Querying** | Custom scripts | Filters, VLOOKUP, pivot tables | Powerful query language (SQL) with joins, subqueries, aggregations |
| **Relationships** | Manual cross-referencing | VLOOKUP (fragile) | Foreign keys with referential integrity |
| **Backup/Recovery** | Manual copies | Manual copies / version history | Automated, point-in-time recovery, transaction logs |
| **Audit trail** | None | Limited (track changes) | Full transaction logging, triggers for audit tables |
| **Programmability** | Scripts are external | VBA macros | Stored procedures, functions, triggers — built into the database |
| **Scalability** | Poor | Poor | Excellent — indexes, partitioning, replication |

For TrailShop: a spreadsheet might work when you have 20 products and one employee. But as the catalog grows to thousands of items, with hundreds of customers, thousands of orders, and a team of 10+ employees accessing data simultaneously — you need a database. There's no way around it.

### A Realistic Tipping Point

When should an organization switch from spreadsheets to a database? There's no single answer, but here are warning signs:

- **Multiple people need to edit the same data** — concurrency problems begin
- **Data exceeds a few thousand rows** — spreadsheet performance degrades; sorting and filtering become slow
- **You need to combine data from different sources** — JOIN operations are trivial in SQL but painful in spreadsheets
- **You need audit trails** — knowing who changed what, and when
- **You need enforced rules** — preventing invalid data from entering the system
- **You're writing scripts to manipulate spreadsheets** — at that point, you're essentially building a bad database; build a good one instead

TrailShop hit all six of these warning signs simultaneously. The spreadsheet era is over.

---

## 10. When NOT to Use a Database

It's worth mentioning that databases aren't always the right answer. For some situations, simpler solutions work fine:

- **Small, single-user data sets** — a personal budget tracker with 100 rows doesn't need PostgreSQL. A spreadsheet or even a text file works.
- **Configuration files** — application settings are typically stored in JSON, YAML, or environment variables — not databases.
- **Temporary, throwaway data** — if you're writing a quick script to process a CSV and produce a report, there's no need to load it into a database first.
- **Static data that never changes** — a lookup table of country codes can live in a JSON file.

The key question is: **will multiple users or applications share and modify this data?** If yes, you almost certainly need a database. If no, simpler options may suffice.

---

## 11. PostgreSQL — Our DBMS of Choice

Throughout this course you'll use **PostgreSQL** (often called "Postgres"). Let's get to know it.

### 11.1 Brief History

PostgreSQL's roots go back to 1986, when Professor Michael Stonebraker at UC Berkeley started the **POSTGRES** project (short for "Post-Ingres") as a research project in object-relational database technology. In 1996, it was renamed to PostgreSQL when SQL support was added. It has been under continuous open-source development for over 30 years, with a global community of contributors.

For more on the history, see: https://www.postgresql.org/docs/current/history.html

### 11.2 Key Features

PostgreSQL is:

- **Free and open-source** — no licensing costs, no vendor lock-in
- **Standards-compliant** — closer to the SQL standard than most other databases
- **ACID-compliant** — full transaction support with MVCC
- **Extensible** — you can create custom data types, functions, operators, and even index types
- **Feature-rich** — supports advanced data types (JSON/JSONB, arrays, hstore, geometric types), full-text search, common table expressions, window functions, and much more
- **Reliable** — used in production by companies like Apple, Instagram, Spotify, Reddit, and the International Space Station
- **Well-documented** — the PostgreSQL documentation (https://www.postgresql.org/docs/current/) is widely regarded as some of the best open-source documentation available

### 11.3 How PostgreSQL Compares

| Feature | PostgreSQL | MySQL | SQL Server | Oracle |
|---|---|---|---|---|
| Cost | Free | Free (Community) | Paid (Express is free but limited) | Paid (expensive) |
| SQL compliance | High | Medium | Medium-High | High |
| JSON support | Excellent (JSONB) | Basic | Good | Good |
| Extensibility | Excellent | Limited | Limited | Good |
| Open source | Yes | Yes (but Oracle-owned) | No | No |
| Advanced types | Arrays, hstore, ranges, geometric | Limited | Limited | Good |

For this course, PostgreSQL is ideal because it's free, standards-compliant, feature-rich, and widely used in industry. What you learn with PostgreSQL will transfer to any other SQL database.

### 11.4 Tools for Working with PostgreSQL

You'll interact with PostgreSQL using:

- **psql** — the built-in command-line tool. Fast, powerful, and available on every PostgreSQL installation.
- **pgAdmin** — a graphical administration tool with a web-based interface. Great for visual exploration.
- **DBeaver** — a free, universal database tool that supports PostgreSQL and many other databases.
- **VS Code extensions** — such as the PostgreSQL extension for running queries from your editor.
- Any other SQL client of your choice.

### 11.5 Basic psql Commands

Once you've installed PostgreSQL and connected with psql, here are commands you'll use constantly:

| Command | What It Does |
|---|---|
| `\l` | List all databases |
| `\c dbname` | Connect to a database |
| `\dt` | List all tables in the current database |
| `\d tablename` | Describe a table's structure |
| `\du` | List all users/roles |
| `\q` | Quit psql |
| `\?` | Show all psql commands |
| `\h SQL_COMMAND` | Show help for a specific SQL command |

You will practice these in the exercises.

---

## 12. Key Terms

| Term | Definition |
|---|---|
| **Data** | Raw, unprocessed facts (numbers, text, dates) |
| **Information** | Data that has been processed and given context to become meaningful |
| **Knowledge** | Information combined with experience and interpretation |
| **Data element** | The smallest unit of data that has meaning (a single field value) |
| **Data redundancy** | Unnecessary duplication of data across files or systems |
| **Data inconsistency** | Contradictory copies of the same data in different locations |
| **Data isolation** | Data trapped in separate files/formats, difficult to combine |
| **Data integrity** | The accuracy and consistency of data throughout its lifecycle |
| **Data independence** | The ability to change storage or structure without affecting applications |
| **DBMS** | Database Management System — software that creates, maintains, and controls databases |
| **RDBMS** | Relational DBMS — a DBMS based on the relational model (tables, SQL) |
| **Database** | A shared collection of logically related data designed for a specific purpose |
| **Table** | A structured set of data organized in rows and columns |
| **Field / Column / Attribute** | A single category of data in a table (e.g., `price`) |
| **Row / Record / Tuple** | A single entry in a table representing one entity (e.g., one product) |
| **Schema** | The formal definition of a database's structure |
| **Metadata** | Data that describes the structure and properties of other data |
| **Data dictionary** | The system catalog — metadata stored by the DBMS about its own structure |
| **Three-schema architecture** | The ANSI/SPARC model separating databases into external, conceptual, and internal levels |
| **SQL** | Structured Query Language — the standard language for querying and managing relational databases |
| **DDL** | Data Definition Language — SQL commands for defining structure (CREATE, ALTER, DROP) |
| **DML** | Data Manipulation Language — SQL commands for working with data (SELECT, INSERT, UPDATE, DELETE) |
| **Concurrency** | Multiple users accessing or modifying data simultaneously |
| **Transaction** | A logical unit of work that must either complete fully or not at all (atomic) |
| **ACID** | Atomicity, Consistency, Isolation, Durability — properties of reliable transactions |
| **View** | A virtual table showing a specific subset or combination of data |
| **Primary key** | A column (or set of columns) that uniquely identifies each row — more in Week 37 |
| **Foreign key** | A column that references the primary key of another table — more in Week 37 |
| **NoSQL** | A category of non-relational databases optimized for specific use cases |

---

## 13. Reading Assignments

**Required:**
- *Database Design*, 2nd Edition by Adrienne Watt & Nelson Eng — Chapters 1–3
  - Chapter 1: Data and Information, File-Based Systems, DBMS
  - Chapter 2: The Database Approach, Three-Schema Architecture
  - Chapter 3: Characteristics and Benefits of the Database Approach
- PostgreSQL Tutorial: Getting Started — https://www.postgresql.org/docs/current/tutorial-start.html

**Further Reading:**
- *Database Design*, 2nd Edition — Chapter 6 (Classification of Database Management Systems)
- E.F. Codd, "A Relational Model of Data for Large Shared Data Banks" (1970) — the original paper: https://www.seas.upenn.edu/~zives/03f/cis550/codd.pdf
- Oracle: "What Is a Database?" — https://www.oracle.com/database/what-is-database/
- IBM: "What Is a Relational Database?" — https://www.ibm.com/topics/relational-databases
- PostgreSQL: A Brief History — https://www.postgresql.org/docs/current/history.html
- Wikipedia: Database — https://en.wikipedia.org/wiki/Database
- Wikipedia: ACID — https://en.wikipedia.org/wiki/ACID

---

## 14. Summary

You've taken the first step into TrailShop's database journey. Here's what you've learned:

- **Data vs information:** Raw facts become useful only when structured and given context.
- **File-based problems:** Redundancy, isolation, integrity issues, security gaps, and concurrency nightmares plague unstructured data storage.
- **The database approach:** A DBMS provides a controlled, shared, self-describing environment that solves all five problems.
- **Three-schema architecture:** External, conceptual, and internal levels provide data independence.
- **DBMS functions:** Data definition, manipulation, the data dictionary, concurrency control, security, integrity enforcement, and backup/recovery.
- **History:** From flat files to hierarchical to network to relational (Codd, 1970) to object-relational to NoSQL — each generation solved problems the previous one couldn't.
- **Types of DBMS:** Relational (our focus), document, graph, key-value — each suited to different use cases.
- **PostgreSQL:** A free, standards-compliant, feature-rich RDBMS that you'll use throughout this course.

You've also set up your own PostgreSQL instance and created the `trailshop` database. It's empty for now, but not for long.

**Next week:** You'll learn how relational databases organize data using a formal model — relations, keys, and integrity rules. You'll discover the mathematical foundation that keeps TrailShop's data accurate and trustworthy. Get ready for Chapter 2: "Laying the Foundation."
