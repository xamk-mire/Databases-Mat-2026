# Week 51 — Final Exam

> **Final exam week.** No new course content. Use this chapter for exam guidance, the study checklist, and optional practice in the Exercises file.

## Epilogue: "The Journey's End"

You've come a long way. Fourteen weeks ago, you walked into TrailShop as a junior developer tasked with taming a chaotic mess of spreadsheets. By Week 48, you had built a real, production-quality relational database from scratch — designed it, normalized it, secured it, optimized it, and connected it to an application.

Now there is one final milestone: the **course exam**. This chapter helps you prepare by organizing everything you've learned into a structured review, giving you guidance on what to expect, and pointing you toward what comes *after* this course.

---

## Learning Objectives

By the end of this chapter you will be able to:

- Understand the exam format and types of questions
- Use a structured checklist to identify topics needing review
- Reflect on your learning journey
- Identify next steps for continued database learning

---

## 1. Exam Guidance

### What to Expect

The exam tests your understanding across the entire course. Questions fall into three categories:

| Question Type | Description | Weight |
|---|---|---|
| **Conceptual / Short Answer** | Explain database concepts in your own words | ~30% |
| **Practical SQL** | Write SQL statements to solve given problems | ~50% |
| **Design Scenarios** | Analyze requirements, draw schemas, normalize | ~20% |

### Conceptual Questions

These test whether you *understand* the concepts, not just memorize syntax. Examples:
- "Explain what referential integrity means and how PostgreSQL enforces it."
- "Why would you use a transaction when transferring stock between warehouses?"
- "What is the difference between DELETE and TRUNCATE?"

**Tip:** Answer in your own words. Use a brief example if it helps clarify.

### Practical SQL Questions

You'll be given a schema description and asked to write SQL. Examples:
- "Write a query to find the top 3 customers by total spending."
- "Create a table with appropriate constraints for the given requirements."
- "Write a transaction that safely moves an order from 'pending' to 'shipped'."

**Tip:** Start with the structure (SELECT...FROM...WHERE), then fill in details. Use aliases. Don't forget semicolons.

### Design Scenarios

These ask you to think like a designer:
- "Given these business requirements, design a normalized schema (3NF)."
- "Identify the functional dependencies in this table and normalize to BCNF."
- "This table has redundancy. What problems can occur? How would you fix it?"

**Tip:** Draw the tables with their columns. Mark PKs and FKs. Check each normal form systematically.

### What is NOT on the Exam

- Memorizing exact PostgreSQL error messages
- Writing application code (Python, Node.js, etc.)
- ORM-specific syntax (Django, SQLAlchemy)
- Cloud provider–specific features (AWS RDS configuration, etc.)
- Obscure PostgreSQL extensions or contrib modules

---

## 2. Structured Study Checklist

Use this checklist to guide your revision. For each item, ask yourself: "Can I explain this clearly and write SQL for it?" If not, revisit the corresponding week's theory.

### Fundamentals (Weeks 36–37)

- [ ] **Data vs Information** — Raw facts vs processed, contextual meaning
- [ ] **File-based system problems** — Redundancy, isolation, integrity, concurrency
- [ ] **Database & DBMS definition** — Organized collection + software to manage it
- [ ] **Database characteristics** — Self-describing, program-data independence, multiple views
- [ ] **Relational model** — Tables (relations), rows (tuples), columns (attributes)
- [ ] **Keys** — Candidate, primary, foreign, composite, surrogate
- [ ] **Referential integrity** — FK must match an existing PK or be NULL
- [ ] **Entity vs Relationship** — Things we store data about vs connections between them
- [ ] **Cardinality** — One-to-one, one-to-many, many-to-many (requires junction table)

### Data Types and Constraints (Week 38)

- [ ] **PostgreSQL data types** — INTEGER, NUMERIC, VARCHAR, TEXT, BOOLEAN, DATE, TIMESTAMP, SERIAL
- [ ] **Choosing appropriate types** — Match the real-world data (price = NUMERIC, not FLOAT)
- [ ] **Constraints** — PRIMARY KEY, NOT NULL, UNIQUE, CHECK, DEFAULT, FOREIGN KEY
- [ ] **Named constraints** — Syntax and benefits for maintenance

### Basic SQL (Weeks 39–40)

- [ ] **CREATE TABLE** — Full syntax with column and table constraints
- [ ] **INSERT, UPDATE, DELETE** — Syntax and common patterns
- [ ] **SELECT with WHERE** — Comparison, BETWEEN, IN, LIKE, IS NULL
- [ ] **ORDER BY, LIMIT, OFFSET** — Sorting and pagination
- [ ] **Aggregate functions** — COUNT, SUM, AVG, MIN, MAX
- [ ] **GROUP BY and HAVING** — Grouping rules, WHERE vs HAVING

### Joins and Relationships (Weeks 41–42)

- [ ] **INNER JOIN** — Only matching rows
- [ ] **LEFT/RIGHT JOIN** — Preserve all rows from one side
- [ ] **FULL OUTER JOIN** — All rows from both sides
- [ ] **CROSS JOIN** — Cartesian product
- [ ] **Self-join** — Joining a table to itself
- [ ] **Multi-table joins** — Chaining 3+ tables
- [ ] **Join conditions** — ON clause, multiple conditions

### Subqueries and Advanced Queries (Weeks 43–44)

- [ ] **Scalar subquery** — Returns one value, used in SELECT or WHERE
- [ ] **IN / NOT IN** — Match against a list from a subquery
- [ ] **EXISTS / NOT EXISTS** — Check for row existence
- [ ] **ANY / ALL** — Compare against a set
- [ ] **Correlated subquery** — References outer query columns
- [ ] **CTEs (WITH clause)** — Named temporary result sets, chaining
- [ ] **Set operations** — UNION, INTERSECT, EXCEPT
- [ ] **CASE, COALESCE, NULLIF** — Conditional logic in queries

### Database Design and Normalization (Weeks 45–46)

- [ ] **Functional dependencies** — X → Y means X uniquely determines Y
- [ ] **1NF** — Atomic values, no repeating groups
- [ ] **2NF** — No partial dependencies (on part of composite key)
- [ ] **3NF** — No transitive dependencies (non-key → non-key)
- [ ] **BCNF** — Every determinant is a candidate key
- [ ] **Denormalization** — When and why to intentionally break normal forms
- [ ] **ER diagrams** — Entities, attributes, relationships, cardinalities

### Views, Transactions, and Performance (Weeks 47–48)

- [ ] **Views** — CREATE VIEW, benefits (simplicity, security), limitations
- [ ] **Transactions** — BEGIN, COMMIT, ROLLBACK, SAVEPOINT
- [ ] **ACID properties** — Atomicity, Consistency, Isolation, Durability
- [ ] **Indexes** — CREATE INDEX, B-tree, when to use, when to avoid
- [ ] **EXPLAIN ANALYZE** — Reading query plans, Seq Scan vs Index Scan
- [ ] **Query optimization** — Avoiding SELECT *, using appropriate indexes

### Administration and Security (Week 48)

- [ ] **Roles** — CREATE ROLE, LOGIN, SUPERUSER, INHERIT
- [ ] **Privileges** — GRANT, REVOKE on databases, schemas, tables
- [ ] **Principle of least privilege** — Only grant what's needed
- [ ] **pg_dump / pg_restore** — Backup and restore strategies
- [ ] **Schemas** — Namespaces for organizing database objects

### Application Integration (Week 48)

- [ ] **Connection strings** — Host, port, database, user, password
- [ ] **SQL injection** — What it is, parameterized queries as prevention
- [ ] **ORMs** — Concept, benefits, trade-offs vs raw SQL

---

## 3. Reflection Exercise

Before the exam, take 15 minutes to reflect on your learning. Write brief answers to these questions (for yourself — not graded):

1. **What database concept was hardest for you to grasp? How did you eventually understand it?**

2. **If you had to explain normalization to a friend who has never taken this course, what analogy would you use?**

3. **What would you do differently if you started the TrailShop project again from scratch?**

4. **Which SQL skill do you feel most confident about? Which one needs more practice?**

5. **How has your thinking about data organization changed since Week 36?**

---

## 4. What's Next?

This course gave you a solid foundation. Here's where you can go from here:

### Window Functions

Functions like `ROW_NUMBER()`, `RANK()`, `LAG()`, `LEAD()`, and `SUM() OVER (...)` let you perform calculations across related rows without collapsing them into groups. They're essential for reporting, analytics, and leaderboard-style queries. PostgreSQL has excellent window function support.

> 🔗 https://www.postgresql.org/docs/current/tutorial-window.html

### Stored Procedures and Functions

PostgreSQL lets you write server-side logic using PL/pgSQL (or even Python, JavaScript). Stored procedures encapsulate complex business logic inside the database, reducing round-trips between your application and the server. They're powerful but should be used judiciously.

> 🔗 https://www.postgresql.org/docs/current/plpgsql.html

### Triggers

A trigger automatically executes a function when a specific event occurs (INSERT, UPDATE, DELETE). Use them for audit logging, enforcing complex business rules, or maintaining derived data. Be careful — triggers add hidden complexity.

> 🔗 https://www.postgresql.org/docs/current/trigger-definition.html

### ORMs and Application Frameworks

Object-Relational Mappers like SQLAlchemy (Python), Prisma (JavaScript/TypeScript), or Hibernate (Java) translate between your programming language's objects and database tables. They speed up development but can generate inefficient queries if you don't understand the SQL underneath — which you now do.

### Design Patterns

Patterns like Repository, Unit of Work, and CQRS (Command Query Responsibility Segregation) help organize database access in larger applications. They separate concerns and make code more testable and maintainable.

### Cloud Databases

Services like AWS RDS, Google Cloud SQL, and Azure Database for PostgreSQL let you run PostgreSQL without managing the server yourself. You still write the same SQL, but you gain automated backups, scaling, and high availability. Understanding the fundamentals you've learned makes cloud databases much easier to work with.

### Data Warehousing and Analytics

When databases grow very large and are used primarily for analytics (not transactions), you enter the world of data warehousing. Technologies like columnar storage, star schemas, and tools like Apache Spark or BigQuery build on the relational concepts you already know.

---

## 5. Summary

You started this course with spreadsheets and chaos. You're ending it with:

- A **solid understanding** of the relational model and why it exists
- The ability to **design** a normalized database schema from business requirements
- Confidence in **writing SQL** — from simple SELECT to complex multi-CTE queries
- Knowledge of **transactions, indexes, and security** for production databases
- Experience **connecting** a database to a real application
- A **completed TrailShop project** (submitted in Week 48) that demonstrates all of the above

Databases are everywhere. Every web application, mobile app, enterprise system, and data pipeline relies on them. The skills you've built here will serve you in every software project you work on.

**Good luck on the final exam.** You've earned every bit of the knowledge you now carry.

---

## Recommended Reading

- Watt & Eng, *Database Design — 2nd Edition* — Full text review for exam preparation
- PostgreSQL Official Documentation: https://www.postgresql.org/docs/current/
- *Designing Data-Intensive Applications* by Martin Kleppmann — For those who want to go deeper

---

*End of course materials. Good luck on the exam!*
