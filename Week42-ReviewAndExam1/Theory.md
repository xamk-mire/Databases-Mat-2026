# Week 42 — Review and Exam 1 Preparation

## Chapter 7: "Taking Stock"

Seven weeks. That's how long you've been working at TrailShop — from inheriting a mess of spreadsheets to building a fully functional relational database with proper structure, constraints, and powerful queries. Before moving forward to more advanced topics, it's time to take stock of what you've learned, identify any gaps, and prepare for the first exam.

Think of this week as a checkpoint on a long hike. You pause, check the map, make sure your gear is in order, refill your water, and assess the trail ahead. The concepts you've learned so far are the *foundation* for everything that follows — normalization, views, subqueries, functions, and transaction management all build on the skills from Weeks 36–41.

---

## Learning Objectives

By the end of this chapter you will be able to:

- Summarize the key concepts from each previous week
- Explain how concepts from different weeks connect to each other
- Identify your own weak areas using the self-assessment checklist
- Avoid the most common exam mistakes
- Apply effective exam preparation strategies

---

## 1. Recap of Each Week

### 1.1 Week 36 — Introduction to Databases

You started with the fundamental question: *why do databases exist?* The answer came from seeing TrailShop's spreadsheet chaos — data redundancy, inconsistency, isolation, and integrity problems. You learned that a **DBMS** (Database Management System) solves these problems by centralizing data, enforcing rules, managing concurrent access, and providing recovery mechanisms.

You distinguished **data** (raw facts) from **information** (data with context and meaning). You explored the **three-level architecture** (external, conceptual, internal) that provides data independence — the ability to change the physical storage without affecting applications. You installed PostgreSQL and created your first database, taking the first concrete step from theory to practice.

Key takeaway: Databases exist to solve real problems that file-based systems cannot handle. The DBMS is not just a storage tool — it's an entire system for managing data integrity, concurrency, security, and recovery.

> **Reference:** Watt & Eng, Chapters 1–4

### 1.2 Week 37 — The Relational Data Model

With the "why" established, you moved to the "how" — specifically, the *relational model* that underpins PostgreSQL and virtually every business database. You learned that a **relation** is a table, a **tuple** is a row, and an **attribute** is a column. More importantly, you learned the *rules*:

**Keys** — candidate keys, primary keys, and foreign keys — provide the mechanism for unique identification and inter-table relationships. **Entity integrity** says primary keys can never be NULL. **Referential integrity** says foreign keys must reference existing primary key values (or be NULL). These aren't arbitrary rules — they're mathematical guarantees that your data remains consistent.

You practiced identifying keys in TrailShop's data and applied integrity constraints to ensure no orphan records or duplicate entries could exist.

Key takeaway: The relational model is a formal, mathematical system. Its constraints (entity integrity, referential integrity) are what make databases *trustworthy* — they guarantee that data stays consistent even as thousands of operations modify it.

> **Reference:** Watt & Eng, Chapters 7–9

### 1.3 Week 38 — Conceptual Data Modelling (ER Diagrams)

You stepped back from implementation to *design*. Before building tables, you need to understand the business domain. ER (Entity-Relationship) modelling gives you a visual language for capturing:

- **Entities** (things you need to store data about)
- **Attributes** (properties of those entities)
- **Relationships** (how entities connect to each other)
- **Cardinality** (how many of one entity relate to another: 1:1, 1:M, M:N)

You drew the TrailShop ER diagram, identifying that customers *place* orders (1:M), orders *contain* products (M:N, resolved via a junction entity), and products *belong to* categories (M:1). You learned that M:N relationships always require a junction (associative) entity.

Key takeaway: ER modelling is a *communication tool* — it lets you validate your understanding of the business with non-technical stakeholders before writing any SQL. A correct ER diagram saves weeks of rework.

> **Reference:** Watt & Eng, Chapters 8–9

### 1.4 Week 39 — Logical Database Design

You transformed your ER diagram into a **logical schema** — the actual table definitions ready for implementation. Each entity became a table. Attributes became columns with appropriate data types. Relationships became foreign keys. M:N relationships became junction tables.

You made critical decisions about data types (NUMERIC for money, VARCHAR vs TEXT, TIMESTAMPTZ for timestamps). You named constraints, chose between natural and surrogate keys, and documented your schema. The logical design is the bridge between the abstract ER model and the concrete SQL code.

Key takeaway: Logical design is where you make binding decisions about types, constraints, and structure. Getting this right is crucial because changing the schema later (with data already in it) is much harder than getting it right the first time.

> **Reference:** Watt & Eng, Chapters 9–11

### 1.5 Week 40 — SQL Fundamentals (DDL + DML)

You finally wrote real SQL. You learned that SQL is divided into sub-languages (DDL, DML, DCL, TCL) and practiced the core commands:

- **CREATE TABLE** with data types and all constraint types
- **INSERT INTO** — single row, multiple rows, with RETURNING
- **UPDATE** — with expressions, the critical WHERE clause
- **DELETE** — with WHERE, cascading deletes, TRUNCATE

You built the complete TrailShop database from scratch, inserted sample data, and practiced modifying and removing data safely. You learned that the most dangerous SQL mistakes involve missing WHERE clauses — and that transactions provide a safety net.

Key takeaway: DDL defines structure; DML manipulates data. Always verify your WHERE clause before running UPDATE or DELETE. Use transactions for safety.

> **Reference:** Watt & Eng, Chapters 15–16

### 1.6 Week 41 — Querying Data

The payoff week. You learned to extract meaningful information using the full power of SELECT:

- **WHERE** with all operators (comparison, logical, BETWEEN, IN, LIKE, IS NULL)
- **ORDER BY** for sorting, LIMIT/OFFSET for pagination
- **Aggregate functions** (COUNT, SUM, AVG, MIN, MAX) with proper NULL handling
- **GROUP BY** for categorized summaries, HAVING for filtering groups
- **JOINs** (INNER, LEFT, RIGHT, FULL OUTER, CROSS) for combining data across tables

You understood that SQL has a logical execution order (FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT) that explains why certain things work and others don't. You wrote 12 progressively complex TrailShop queries.

Key takeaway: SELECT is the most powerful and frequently used SQL statement. JOINs let you combine normalized data back together. The logical execution order explains SQL's behavior.

> **Reference:** Watt & Eng, Chapter 16

---

## 2. Connections Between Weeks

The topics from Weeks 36–41 aren't isolated — they form a coherent pipeline from problem to solution:

### 2.1 The Design Pipeline

```
Business Problem (Week 36)
    ↓ "We need a database"
Relational Theory (Week 37)
    ↓ "Here are the rules"
Conceptual Design (Week 38)
    ↓ "Here's what we're modelling"
Logical Design (Week 39)
    ↓ "Here's the table structure"
Implementation (Week 40)
    ↓ "Here's the running database"
Querying (Week 41)
    ↓ "Here's the information we needed"
```

Each week's output is the next week's input. Skip any step and you risk building the wrong thing, or building it wrong.

### 2.2 How Concepts Reinforce Each Other

| Concept from early weeks | How it appears in SQL |
|--------------------------|----------------------|
| Entity integrity (Week 37) | PRIMARY KEY constraint (Week 40) |
| Referential integrity (Week 37) | FOREIGN KEY constraint (Week 40) |
| Cardinality 1:M (Week 38) | FK in the "many" table (Week 40) |
| Cardinality M:N (Week 38) | Junction table (Week 40) |
| Attributes (Week 38) | Column definitions with types (Week 40) |
| Data types (Week 39) | CREATE TABLE column types (Week 40) |
| Joining tables (Week 41) | Follows FK relationships (Week 37/40) |

### 2.3 Why Understanding Matters

Students who memorize SQL syntax without understanding the relational model often write queries that *work* but are inefficient, fragile, or wrong. For example:
- If you don't understand referential integrity, you won't know why ON DELETE CASCADE is both powerful and dangerous
- If you don't understand cardinality, you won't predict whether a JOIN will produce more or fewer rows than the original table
- If you don't understand NULL semantics from Week 37, you'll write WHERE clauses that silently miss rows

---

## 3. Self-Assessment Checklist

Rate yourself on each item: ✓ (confident), ~ (somewhat), ✗ (need to review).

### Fundamentals (Weeks 36–37)

1. [ ] I can explain the problems of file-based data management
2. [ ] I can define what a DBMS is and list its key functions
3. [ ] I can distinguish between relations, tuples, and attributes
4. [ ] I can explain candidate keys, primary keys, and foreign keys
5. [ ] I can state entity integrity and referential integrity rules

### Design (Weeks 38–39)

6. [ ] I can identify entities, attributes, and relationships from a description
7. [ ] I can draw an ER diagram with correct cardinality notation
8. [ ] I can explain 1:1, 1:M, and M:N relationships
9. [ ] I can transform an ER diagram into table definitions
10. [ ] I can choose appropriate data types for common scenarios

### DDL (Week 40)

11. [ ] I can write CREATE TABLE with all constraint types
12. [ ] I can explain NOT NULL, UNIQUE, PRIMARY KEY, FOREIGN KEY, CHECK, DEFAULT
13. [ ] I can create tables in the correct dependency order
14. [ ] I can use ALTER TABLE to add/drop columns and constraints
15. [ ] I can explain SERIAL vs GENERATED ALWAYS AS IDENTITY

### DML (Weeks 40–41)

16. [ ] I can write INSERT statements (single row, multiple rows, RETURNING)
17. [ ] I can write UPDATE with WHERE (and explain the danger of omitting WHERE)
18. [ ] I can write DELETE with WHERE and explain CASCADE behavior
19. [ ] I can write SELECT with WHERE, ORDER BY, LIMIT
20. [ ] I can use aggregate functions (COUNT, SUM, AVG, MIN, MAX)
21. [ ] I can write GROUP BY with HAVING
22. [ ] I can write INNER JOIN and LEFT JOIN queries
23. [ ] I can join 3+ tables in a single query
24. [ ] I can explain the logical execution order of a query

---

## 4. Common Mistakes

Here are the mistakes that appear most frequently on exams and in homework submissions:

### 4.1 Confusing WHERE and HAVING

- WHERE filters *rows* (before grouping)
- HAVING filters *groups* (after grouping and aggregation)

**Wrong:** `WHERE COUNT(*) > 5` — aggregates can't be in WHERE
**Right:** `HAVING COUNT(*) > 5`

### 4.2 Forgetting GROUP BY Columns

Every non-aggregated column in SELECT must appear in GROUP BY.

**Wrong:** `SELECT name, category_id, COUNT(*) FROM products GROUP BY category_id`
**Right:** Either add `name` to GROUP BY or remove it from SELECT.

### 4.3 NULL Comparison with =

**Wrong:** `WHERE description = NULL`
**Right:** `WHERE description IS NULL`

NULL is not a value — it's the absence of a value. It can't be compared with equality operators.

### 4.4 Missing ON in JOINs

**Wrong:** `SELECT * FROM products INNER JOIN categories`
**Right:** `SELECT * FROM products INNER JOIN categories ON products.category_id = categories.category_id`

### 4.5 Wrong Table Creation Order

Creating a table with a FOREIGN KEY before the referenced table exists produces an error. Always create parent tables first.

### 4.6 UPDATE/DELETE Without WHERE

The most dangerous mistake. Always verify your WHERE clause — or use a transaction.

### 4.7 Using Double Quotes for String Values

In SQL, single quotes delimit strings: `'hello'`. Double quotes delimit identifiers: `"column name"`.

### 4.8 Confusing INNER JOIN and LEFT JOIN Results

- INNER JOIN drops rows with no match in the other table
- LEFT JOIN keeps all rows from the left table (filling NULLs for unmatched)

If you use INNER JOIN when you should use LEFT JOIN, you silently lose data.

### 4.9 Forgetting DISTINCT When Needed

Joins can produce duplicate rows when the relationship is 1:M. If you only need unique values, remember to use DISTINCT or structure your query carefully.

---

## 5. Exam Preparation Tips

### 5.1 Practice by Doing

Reading SQL is not the same as writing it. Open psql and practice:
- Create a small database from scratch
- Write 10 queries of increasing complexity
- Try to break things intentionally (violate constraints, omit WHERE)

### 5.2 Explain Queries in English

For any query you write, practice explaining it in plain English: "This query finds all customers who have spent more than €200 total by joining customers to orders to order_items, summing the line totals per customer, and filtering to groups where the sum exceeds 200."

### 5.3 Work Through Errors

When you get an error, read it carefully. PostgreSQL error messages tell you:
- What went wrong (e.g., "violates foreign key constraint")
- Which constraint was violated (if named)
- Which values caused the problem

### 5.4 Time Management on the Exam

- Read all questions first
- Start with questions you're confident about
- For SQL writing questions: write the query skeleton first (SELECT...FROM...WHERE), then fill in details
- Leave 5 minutes to review your answers

### 5.5 The Most Valuable Study Technique

Take the TrailShop schema and invent your own business questions, then write the SQL to answer them. If you can translate an English question into correct SQL, you understand the material.

---

## 6. Key Terms — Comprehensive Review

This master glossary combines key terms from all previous weeks.

| Term | Week | Definition |
|------|------|-----------|
| Data | 36 | Raw, unprocessed facts |
| Information | 36 | Data processed and given context |
| Database | 36 | An organized collection of related data |
| DBMS | 36 | Software system for managing databases |
| Data redundancy | 36 | Same data stored in multiple places |
| Data integrity | 36 | Accuracy and consistency of data |
| Relation | 37 | A table in the relational model |
| Tuple | 37 | A row in a relation |
| Attribute | 37 | A column in a relation |
| Domain | 37 | The set of allowed values for an attribute |
| Candidate key | 37 | A minimal set of attributes that uniquely identifies a tuple |
| Primary key | 37 | The chosen candidate key for a relation |
| Foreign key | 37 | An attribute that references a primary key in another relation |
| Entity integrity | 37 | Primary key values cannot be NULL |
| Referential integrity | 37 | FK values must match existing PK values (or be NULL) |
| NULL | 37 | Represents unknown or missing data |
| Entity | 38 | A thing about which data is stored |
| Attribute (ER) | 38 | A property of an entity |
| Relationship | 38 | An association between entities |
| Cardinality | 38 | The number of instances in a relationship (1:1, 1:M, M:N) |
| ER diagram | 38 | Visual representation of entities and relationships |
| Junction table | 38/39 | A table that resolves an M:N relationship |
| Logical schema | 39 | Table definitions with columns, types, and constraints |
| Data type | 39 | Defines what kind of data a column holds |
| Surrogate key | 39 | An artificial key (e.g., auto-incrementing ID) |
| Natural key | 39 | A key from the real-world data (e.g., email, ISBN) |
| SQL | 40 | Structured Query Language |
| DDL | 40 | Data Definition Language (CREATE, ALTER, DROP) |
| DML | 40 | Data Manipulation Language (INSERT, SELECT, UPDATE, DELETE) |
| DCL | 40 | Data Control Language (GRANT, REVOKE) |
| TCL | 40 | Transaction Control Language (BEGIN, COMMIT, ROLLBACK) |
| Constraint | 40 | A rule enforced by the database |
| SERIAL | 40 | PostgreSQL auto-incrementing column type |
| CASCADE | 40 | Propagates operations to dependent objects/rows |
| RETURNING | 40 | PostgreSQL clause that returns values from DML statements |
| SELECT | 41 | Statement for retrieving data |
| WHERE | 41 | Filters rows based on conditions |
| ORDER BY | 41 | Sorts the result set |
| GROUP BY | 41 | Groups rows for aggregate calculations |
| HAVING | 41 | Filters groups after aggregation |
| JOIN | 41 | Combines rows from multiple tables |
| INNER JOIN | 41 | Returns only matching rows |
| LEFT JOIN | 41 | Returns all left-table rows, NULLs for non-matches |
| Aggregate function | 41 | Computes a single value from many rows |
| Alias | 41 | Temporary name for a column or table in a query |

---

## 7. Reading

### Required Reading

- Review your notes and exercises from Weeks 36–41
- Watt & Eng, Chapters 1–4 (Week 36), Chapters 7–9 (Week 37–38), Chapters 9–11 (Week 39), Chapters 15–16 (Weeks 40–41)

### Further Reading

- PostgreSQL Official Tutorial: https://www.postgresql.org/docs/current/tutorial.html
- SQL Style Guide: https://www.sqlstyle.guide/

---

## 8. Summary

This review chapter consolidated six weeks of database fundamentals into a single reference. You've traced the complete path from understanding *why* databases exist, through the *theory* that makes them reliable, the *design process* that ensures correctness, the *SQL* that creates and fills them, and finally the *queries* that extract value.

The first exam will test your ability to:
- Explain database concepts in your own words
- Read and interpret SQL statements
- Write correct SQL for given scenarios
- Identify and fix errors in SQL code
- Apply design principles to new situations

After the exam, you'll move into more advanced territory — normalization, views, subqueries, functions, and transactions. Everything you've learned so far is the foundation for what comes next. Make sure it's solid.
