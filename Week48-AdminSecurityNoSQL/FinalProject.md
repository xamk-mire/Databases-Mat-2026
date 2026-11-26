# Week 48 — Final TrailShop Project Submission

> [!IMPORTANT]
> **Last required project work.** Finish Tasks 1–2 and the theory review in [Exercises.md](Exercises.md) before you submit this package. Weeks 49–51 are reserved for optional exam review. No further project submissions are required after this one.

This package is the complete TrailShop database you have built across the course, plus an extension you design yourself. Place every file in your repository folder `Week48-AdminSecurityNoSQL/`.

## What you are submitting

1. `trailshop_final.sql` — schema, sample data, views, indexes, roles, and a transaction
2. An ER diagram (image or PDF)
3. `trailshop_queries.sql` — at least 10 queries
4. `reflection.md` — 300–500 words

---

## 1. Complete SQL script (`trailshop_final.sql`)

The script rebuilds the whole TrailShop database in PostgreSQL. Write the statements in dependency order so the script runs in `psql` or pgAdmin: tables that other tables reference come first, then the tables that reference them, then sample data, views, indexes, roles, and the transaction.

### Existing tables

Keep these seven tables, with their primary keys, foreign keys, and other constraints:

`products`, `customers`, `orders`, `order_items`, `categories`, `product_categories`, `payments`

### New table or tables

Add at least one new table. Add more than one when the feature needs it, for example a main table plus a junction table, or a header table plus line items.

Choose the feature yourself. Product reviews, suppliers, returns, and wishlists are examples. Above the `CREATE TABLE` statement (for your new tables), write a short SQL comment that names the business purpose: what TrailShop stores here that the seven existing tables leave out.

Each new table includes:

- A primary key
- Data types that fit the values
- `NOT NULL`, `UNIQUE`, and `CHECK` where a business rule needs them
- At least one foreign key to one of the seven existing tables
- An `ON DELETE` action on that foreign key (`CASCADE`, `RESTRICT`, or `SET NULL`), with a short comment that justifies the choice

### Sample data

Insert at least 5 rows into every table, including each new table. Insert parent rows before child rows. Every foreign key in a new table references a primary key value inserted earlier in the script.

### Views, indexes, roles, and a transaction

- **Views:** at least 2.
- **Indexes:** at least 3. Above each index, a comment names the query that index supports.
- **Roles:** at least 2. After each role, grant the privileges that role receives.
- **Transaction:** at least 1, using `BEGIN` and then either `COMMIT` or `ROLLBACK`. A comment states what the transaction demonstrates, for example a stock update and an order line that succeed or fail together.

---

## 2. ER diagram

Submit an image or PDF, for example `diagrams/er-diagram.png`. The diagram shows the final schema:

- Every entity and its attributes, including the new table or tables
- Primary keys marked
- Foreign keys indicated
- Cardinality on every relationship, including relationships from the new table or tables

---

## 3. Query collection (`trailshop_queries.sql`)

Write at least 10 queries that run against the schema created by `trailshop_final.sql`. Above each query, a comment states the business question the query answers.

The set includes all of the following:

- A join of at least 3 tables
- Aggregation with `GROUP BY` and `HAVING`
- At least one correlated subquery
- A CTE
- A `CASE` expression
- A set operation (`UNION`, `INTERSECT`, or `EXCEPT`)
- At least one query that uses a table you added

One query may demonstrate more than one of these techniques. The file still contains at least 10 queries.

---

## 4. Reflection (`reflection.md`)

Write 300–500 words in your own words, answering:

- What was the most challenging part of building this database?
- If you were to redesign it, what would you change and why?
- How does your final schema differ from your Week 38 version? What drove the changes?
- What new table or tables did you add, and why did that feature need its own table or tables?
- What did you learn about database design that you didn't expect?

---

## Grading Criteria

| Criterion                                                                                     | Weight |
| --------------------------------------------------------------------------------------------- | ------ |
| Schema design (appropriate tables, types, constraints, including the new table or tables)     | 25%    |
| Relationships and referential integrity (including foreign keys from the new table or tables) | 20%    |
| Query variety and correctness                                                                 | 25%    |
| Security, indexes, and transactions                                                           | 15%    |
| Reflection quality and depth                                                                  | 15%    |

---

## Submission Checklist

- [ ] `trailshop_final.sql` rebuilds the database in dependency order, with sample data, at least 2 views, at least 3 indexes, at least 2 roles, and 1 transaction
- [ ] At least one new table added (primary key, constraints, foreign key, `ON DELETE` justification, at least 5 rows, and a short purpose comment)
- [ ] ER diagram submitted, including the new table or tables
- [ ] `trailshop_queries.sql` submitted, with at least 10 commented queries and at least one query that uses a table you added
- [ ] `reflection.md` submitted (300–500 words, every prompt answered)
- [ ] All files placed in `Week48-AdminSecurityNoSQL/`
