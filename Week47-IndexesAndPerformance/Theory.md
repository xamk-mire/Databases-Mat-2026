# Week 47 — Indexes and Query Performance

## Chapter 11: "The Slowdown"

TrailShop has grown from a small outdoor gear startup into a thriving e-commerce business. The products table has swelled to over 200,000 rows. The orders table holds millions of records spanning years of transactions. Customers are complaining that the search page takes forever to load, the admin dashboard freezes when staff try to pull sales reports, and the checkout process occasionally times out. The founders are panicking — they want to throw money at a bigger server.

But you know better. The problem isn't hardware — it's how the database is *finding* data. Right now, every query forces PostgreSQL to read every single row in the table, even when it only needs one. That's like searching for a word in a 1,000-page book by reading every page from start to finish. What you need is an *index* — and this week, you're going to learn how to build them, when to use them, and how to prove they work using PostgreSQL's query planner.

---

## Learning Objectives

By the end of this chapter you will be able to:

- Explain why queries slow down as tables grow
- Describe what an index is and how it maps values to row locations
- Explain how B-tree indexes achieve O(log n) lookup performance
- Identify and compare PostgreSQL's index types (B-tree, Hash, GIN, GiST, BRIN, SP-GiST)
- Create basic, unique, multi-column, partial, expression, and covering indexes
- Explain which indexes PostgreSQL creates automatically and which it does not
- Read and interpret EXPLAIN and EXPLAIN ANALYZE output
- Distinguish between Seq Scan, Index Scan, Index Only Scan, and Bitmap scans
- Compare Nested Loop, Hash Join, and Merge Join algorithms
- Decide when to create indexes and when to avoid them
- Monitor and maintain indexes over time
- Apply general query optimization techniques beyond indexing

---

## 1 — Why Queries Slow Down

### 1.1 Sequential Scan: Reading Everything

When you run a query like this:

```sql
SELECT * FROM products WHERE name = 'TrailMaster X4';
```

PostgreSQL must find rows that match the condition. Without an index, it has only one option: start at the first row of the table and check every single row until it reaches the end. This is called a **sequential scan** (Seq Scan).

For a small table with 50 products, this is instant. But what happens when the table grows?

### 1.2 O(n) Growth

A sequential scan has **O(n)** time complexity — the time it takes is directly proportional to the number of rows. If your table doubles in size, the query takes roughly twice as long.

| Rows in `products` | Approximate Seq Scan Time |
|---|---|
| 1,000 | 1 ms |
| 100,000 | 50 ms |
| 1,000,000 | 400 ms |
| 10,000,000 | 4,000 ms (4 seconds) |
| 100,000,000 | 40,000 ms (40 seconds) |

These are rough estimates to illustrate the trend. The exact times depend on hardware, row size, and caching — but the linear relationship holds.

### 1.3 The Library Analogy

Imagine you need to find a specific book in a library with 100,000 books. Without any catalog system, you'd have to walk through every shelf, check every spine, one by one. That could take hours.

But if the library has a card catalog — an alphabetical index that tells you the exact shelf and position — you can go straight to the book. That's what a database index does: it provides a shortcut so the database doesn't have to read everything.

As Watt & Eng note in *Database Design — 2nd Edition*: "Without indexes, the DBMS must perform a full table scan for every query, which becomes impractical as tables grow beyond a few thousand rows."

---

## 2 — What Is an Index?

### 2.1 The Book Index Analogy

A book index sits at the back and lists important terms in alphabetical order, each followed by page numbers where that term appears. You look up the term, get the page number, and flip directly to it.

A database index works the same way:

1. It stores **values** from one or more columns (like the terms in a book index)
2. Each value is paired with a **pointer** to the actual row on disk (like page numbers)
3. The values are organized in a structure that allows fast lookup (like alphabetical order)

### 2.2 A Separate Data Structure

An index is **not** part of the table itself. It's a separate data structure that PostgreSQL maintains alongside the table. When you insert, update, or delete rows, PostgreSQL must also update every index on that table. This is the fundamental trade-off: indexes speed up reads but add overhead to writes.

### 2.3 How an Index Maps Values to Rows

Consider the `products` table:

| product_id | name | price | stock_quantity | category_id |
|---|---|---|---|---|
| 1 | TrailMaster X4 | 149.99 | 12 | 1 |
| 2 | Alpine Tent Pro | 299.00 | 8 | 2 |
| 3 | Summit Harness | 89.50 | 25 | 3 |
| 4 | Canyon Boots | 179.99 | 15 | 1 |
| 5 | RidgeLine Pack | 129.00 | 20 | 4 |

An index on the `name` column would conceptually look like:

| Index Entry (name) | Pointer to Row |
|---|---|
| Alpine Tent Pro | → Row 2 |
| Canyon Boots | → Row 4 |
| RidgeLine Pack | → Row 5 |
| Summit Harness | → Row 3 |
| TrailMaster X4 | → Row 1 |

Notice that the index entries are **sorted**. This sorted order is what makes fast lookups possible — the database can use algorithms like binary search instead of checking every entry.

---

## 3 — B-Tree Indexes

### 3.1 The Default Index Type

When you create an index in PostgreSQL without specifying a type, you get a **B-tree** (balanced tree) index. B-tree is the most versatile and commonly used index type, supporting equality (`=`) and range queries (`<`, `>`, `<=`, `>=`, `BETWEEN`).

```sql
CREATE INDEX idx_products_name ON products(name);
```

This creates a B-tree index by default.

### 3.2 How B-Trees Work

A B-tree organizes data into a tree structure with three levels of nodes:

- **Root node**: The single top node, the entry point for every search
- **Internal nodes**: Intermediate nodes that guide the search downward
- **Leaf nodes**: The bottom level, containing the actual indexed values and pointers to table rows

Here's a simplified ASCII representation of a B-tree index on `product_id`:

```
                        [50]                          ← Root
                       /    \
                      /      \
              [20, 35]        [70, 85]                ← Internal
              /  |  \         /  |   \
             /   |   \       /   |    \
          [1-19][20-34][35-49] [50-69][70-84][85-100]  ← Leaf nodes
```

To find `product_id = 42`:

1. Start at root: 42 < 50, go left
2. At internal node: 42 > 35, go right
3. At leaf node [35–49]: find 42, follow pointer to the table row

That's **3 steps** to search through 100 values. A sequential scan would need up to 100 steps.

### 3.3 O(log n) Lookup

B-trees provide **O(log n)** lookup time. This is dramatically better than O(n) for large tables:

| Table Rows | Seq Scan Comparisons (O(n)) | B-tree Lookups (O(log n)) |
|---|---|---|
| 1,000 | 1,000 | ~10 |
| 100,000 | 100,000 | ~17 |
| 1,000,000 | 1,000,000 | ~20 |
| 10,000,000 | 10,000,000 | ~23 |
| 100,000,000 | 100,000,000 | ~27 |
| 1,000,000,000 | 1,000,000,000 | ~30 |

Even with a billion rows, a B-tree index finds any value in roughly 30 comparisons.

### 3.4 Pages and Disk I/O

PostgreSQL stores data in **8 KB pages** (also called blocks). Each B-tree node corresponds to one page on disk. A single page can hold hundreds of index entries, which is why real B-trees are very shallow — typically 3–4 levels deep even for tables with millions of rows.

Each level of the tree requires one **disk I/O** operation (reading one page). So finding a row in a million-row table might require only 3–4 disk reads instead of thousands.

### 3.5 Why "Balanced" Matters

The "B" in B-tree stands for "balanced." Every path from the root to any leaf node has the **same length**. This guarantees that lookup time is consistent regardless of which value you're searching for. Unlike a simple binary search tree (which can become lopsided and degrade to O(n)), a B-tree automatically rebalances itself during insertions and deletions.

---

## 4 — Index Types in PostgreSQL

PostgreSQL offers several index types, each optimized for different kinds of queries. Choosing the right type matters — using a B-tree where a GIN index is needed (or vice versa) can mean the difference between a fast query and a useless index.

For comprehensive documentation, see: [https://www.postgresql.org/docs/current/indexes.html](https://www.postgresql.org/docs/current/indexes.html)

### 4.1 B-Tree

The default and most commonly used index type. Supports equality and range operators: `=`, `<`, `>`, `<=`, `>=`, `BETWEEN`, `IN`, `IS NULL`, `IS NOT NULL`. Also supports `LIKE 'prefix%'` (but not `LIKE '%suffix'`).

```sql
CREATE INDEX idx_products_price ON products(price);

-- These queries can use the B-tree index:
SELECT * FROM products WHERE price = 149.99;
SELECT * FROM products WHERE price BETWEEN 100 AND 200;
SELECT * FROM products WHERE price > 50 ORDER BY price;
```

**When to use**: Most situations. If you're unsure which type to pick, B-tree is almost always the right choice.

### 4.2 Hash

Hash indexes use a hash function to map values to buckets. They support **only equality** comparisons (`=`).

```sql
CREATE INDEX idx_products_name_hash ON products USING hash (name);

-- This query can use the hash index:
SELECT * FROM products WHERE name = 'TrailMaster X4';

-- This query CANNOT use the hash index (range query):
SELECT * FROM products WHERE name > 'M';
```

**When to use**: Rarely. Hash indexes are slightly faster than B-tree for pure equality lookups and use less space, but they don't support range queries, ordering, or multi-column indexes. In most cases, B-tree is a better overall choice.

### 4.3 GIN (Generalized Inverted Index)

GIN indexes are designed for values that contain multiple elements — arrays, JSONB documents, and full-text search vectors. They index each individual element and point back to the rows containing it.

Imagine TrailShop adds a `tags` column to products:

```sql
ALTER TABLE products ADD COLUMN tags TEXT[];

UPDATE products SET tags = ARRAY['waterproof', 'hiking', 'bestseller']
WHERE product_id = 1;

UPDATE products SET tags = ARRAY['camping', 'lightweight', '4-season']
WHERE product_id = 2;

-- Create a GIN index on the array column
CREATE INDEX idx_products_tags ON products USING gin (tags);

-- Find all products tagged 'waterproof'
SELECT * FROM products WHERE tags @> ARRAY['waterproof'];
```

GIN is also the go-to index for full-text search:

```sql
ALTER TABLE products ADD COLUMN search_vector tsvector;

UPDATE products
SET search_vector = to_tsvector('english', name || ' ' || COALESCE(description, ''));

CREATE INDEX idx_products_search ON products USING gin (search_vector);

-- Full-text search query
SELECT * FROM products
WHERE search_vector @@ to_tsquery('english', 'waterproof & boots');
```

And for JSONB columns:

```sql
ALTER TABLE products ADD COLUMN attributes JSONB;

CREATE INDEX idx_products_attributes ON products USING gin (attributes);

-- Find products where attributes contain a specific key-value
SELECT * FROM products WHERE attributes @> '{"material": "leather"}';
```

**When to use**: Full-text search, array containment queries, JSONB queries.

### 4.4 GiST (Generalized Search Tree)

GiST indexes support complex data types that don't fit neatly into B-tree ordering — geometric shapes, ranges, and spatial data.

Imagine TrailShop has physical store locations:

```sql
CREATE TABLE stores (
    store_id    SERIAL PRIMARY KEY,
    store_name  VARCHAR(100),
    location    POINT
);

INSERT INTO stores (store_name, location) VALUES
    ('Helsinki Store', POINT(24.9384, 60.1699)),
    ('Tampere Store', POINT(23.7610, 61.4978)),
    ('Turku Store', POINT(22.2687, 60.4518));

-- Create a GiST index on the location column
CREATE INDEX idx_stores_location ON stores USING gist (location);

-- Find stores near a given coordinate
SELECT store_name
FROM stores
ORDER BY location <-> POINT(24.0, 60.2)
LIMIT 3;
```

GiST also works well with range types:

```sql
CREATE TABLE promotions (
    promo_id    SERIAL PRIMARY KEY,
    promo_name  VARCHAR(100),
    valid_range DATERANGE
);

CREATE INDEX idx_promotions_range ON promotions USING gist (valid_range);

-- Find promotions active today
SELECT * FROM promotions
WHERE valid_range @> CURRENT_DATE;
```

**When to use**: Spatial/geometric data, range types, nearest-neighbor queries.

### 4.5 BRIN (Block Range Index)

BRIN indexes are extremely small and efficient for **large, physically ordered tables** — tables where the values in a column naturally increase with the physical order of the rows. Time-series data and log tables are perfect candidates.

A BRIN index doesn't store every value. Instead, it stores the **minimum and maximum** value for each range of table pages (by default, 128 pages). This makes it tiny but still useful for range queries.

```sql
CREATE TABLE order_logs (
    log_id      SERIAL PRIMARY KEY,
    created_at  TIMESTAMP NOT NULL DEFAULT NOW(),
    order_id    INTEGER,
    event       VARCHAR(50)
);

-- BRIN works because rows are inserted chronologically,
-- so created_at naturally correlates with physical order
CREATE INDEX idx_logs_created ON order_logs USING brin (created_at);

-- Find logs from the last 24 hours
SELECT * FROM order_logs
WHERE created_at > NOW() - INTERVAL '24 hours';
```

**When to use**: Very large tables (millions+ rows) where the indexed column correlates with the physical insertion order. Especially timestamps in log/event tables.

### 4.6 SP-GiST (Space-Partitioned Generalized Search Tree)

SP-GiST supports partitioned search trees like quadtrees, k-d trees, and radix trees. It's useful for data that has a natural clustering structure that doesn't map well to balanced trees.

```sql
-- SP-GiST can be used for IP address ranges, phone number prefixes, etc.
CREATE INDEX idx_customers_phone ON customers USING spgist (phone);
```

**When to use**: Specialized scenarios involving hierarchical or space-partitioned data. Most applications won't need SP-GiST.

### 4.7 Index Type Comparison

| Index Type | Supported Operators | Best Use Case | Size |
|---|---|---|---|
| B-tree | `=`, `<`, `>`, `<=`, `>=`, `BETWEEN`, `LIKE 'prefix%'` | General purpose — most queries | Medium |
| Hash | `=` | Pure equality lookups | Small |
| GIN | `@>`, `<@`, `@@`, `?`, `?&`, `?\|` | Arrays, JSONB, full-text search | Large |
| GiST | `<<`, `>>`, `<@`, `@>`, `<->` | Spatial data, ranges, nearest-neighbor | Medium |
| BRIN | `<`, `<=`, `=`, `>=`, `>` | Large sequential tables (time-series, logs) | Very small |
| SP-GiST | `<<`, `>>`, `<@`, `@>` | Space-partitioned data (IP ranges, quadtrees) | Medium |

---

## 5 — Creating Indexes

### 5.1 Basic Index

The simplest form creates a B-tree index on a single column:

```sql
CREATE INDEX idx_products_name ON products(name);
```

Naming convention: `idx_tablename_columnname` makes indexes easy to identify.

### 5.2 Unique Index

A unique index enforces that no two rows can have the same value in the indexed column(s):

```sql
CREATE UNIQUE INDEX idx_customers_email ON customers(email);
```

This serves double duty: it speeds up lookups on `email` and prevents duplicate email addresses. Note that `CREATE TABLE ... UNIQUE(email)` automatically creates a unique index behind the scenes.

### 5.3 Multi-Column Indexes

You can index multiple columns together:

```sql
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);
```

**The leftmost prefix rule**: A multi-column index can be used for queries that filter on the leftmost column(s) of the index, but not for queries that skip the leading column.

```sql
-- ✓ Can use the index (filters on customer_id, the leftmost column)
SELECT * FROM orders WHERE customer_id = 42;

-- ✓ Can use the index (filters on both columns)
SELECT * FROM orders WHERE customer_id = 42 AND order_date > '2026-01-01';

-- ✗ Cannot efficiently use this index (skips customer_id)
SELECT * FROM orders WHERE order_date > '2026-01-01';
```

Think of it like a phone book sorted by last name, then first name. You can look up everyone named "Smith" (last name only) or "John Smith" (both), but you can't efficiently find everyone named "John" (first name only) — the phone book isn't sorted that way.

### 5.4 Partial Indexes

A partial index only indexes rows that match a condition. This makes the index smaller and more focused:

```sql
CREATE INDEX idx_orders_active
ON orders(order_date)
WHERE status = 'pending';
```

This index only contains rows where `status = 'pending'`. Queries that include this condition can use the smaller, faster index:

```sql
-- ✓ Uses the partial index
SELECT * FROM orders WHERE status = 'pending' AND order_date > '2026-06-01';

-- ✗ Cannot use the partial index (different status)
SELECT * FROM orders WHERE status = 'shipped' AND order_date > '2026-06-01';
```

Partial indexes are excellent when you frequently query a specific subset of data — active users, pending orders, recent records.

### 5.5 Expression Indexes

An expression index indexes the result of an expression or function rather than a raw column value:

```sql
CREATE INDEX idx_products_name_lower ON products(LOWER(name));
```

This enables case-insensitive searches to use the index:

```sql
-- ✓ Uses the expression index
SELECT * FROM products WHERE LOWER(name) = 'trailmaster x4';

-- ✗ Cannot use the expression index (no LOWER function)
SELECT * FROM products WHERE name = 'trailmaster x4';
```

The query must use the **exact same expression** as the index definition.

### 5.6 Covering Indexes (INCLUDE)

A covering index includes extra columns that aren't part of the search key but are needed in the query result. This enables **Index Only Scans** — PostgreSQL can answer the query entirely from the index without touching the table at all.

```sql
CREATE INDEX idx_orders_customer_covering
ON orders(customer_id)
INCLUDE (order_date, total_amount);
```

```sql
-- This can be an Index Only Scan — all needed columns are in the index
SELECT order_date, total_amount
FROM orders
WHERE customer_id = 42;
```

The `INCLUDE` columns are stored in the leaf nodes of the index but aren't part of the search tree structure. This means they don't affect sort order or search capability — they're just along for the ride.

### 5.7 Full Syntax Reference

```sql
CREATE [ UNIQUE ] INDEX [ CONCURRENTLY ] [ [ IF NOT EXISTS ] index_name ]
    ON [ ONLY ] table_name [ USING method ]
    ( { column_name | ( expression ) } [ COLLATE collation ]
      [ opclass [ ( opclass_parameter = value [, ... ] ) ] ]
      [ ASC | DESC ] [ NULLS { FIRST | LAST } ]
      [, ...] )
    [ INCLUDE ( column_name [, ...] ) ]
    [ NULLS [ NOT ] DISTINCT ]
    [ WITH ( storage_parameter [= value] [, ... ] ) ]
    [ TABLESPACE tablespace_name ]
    [ WHERE predicate ];
```

### 5.8 CONCURRENTLY: Creating Indexes in Production

By default, `CREATE INDEX` locks the table against writes for the duration of index creation. On a large table, this can take minutes — unacceptable in production.

The `CONCURRENTLY` option builds the index without blocking writes:

```sql
CREATE INDEX CONCURRENTLY idx_products_name ON products(name);
```

Trade-offs:
- Takes longer to build (must scan the table twice)
- Cannot be run inside a transaction
- If it fails partway through, you get an invalid index that must be dropped

Always use `CONCURRENTLY` when creating indexes on production tables that are actively receiving traffic.

---

## 6 — Automatic Indexes

### 6.1 Primary Keys and UNIQUE Constraints

PostgreSQL **automatically** creates an index when you define a primary key or a `UNIQUE` constraint:

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,  -- auto-creates unique index
    name VARCHAR(100) UNIQUE,       -- auto-creates unique index
    price NUMERIC(10,2)
);
```

You can verify this with:

```sql
\di products
```

### 6.2 Foreign Keys Do NOT Get Auto-Indexed

This is one of the most common performance mistakes in PostgreSQL. When you create a foreign key constraint, PostgreSQL does **not** automatically create an index on the foreign key column.

```sql
CREATE TABLE order_items (
    item_id     SERIAL PRIMARY KEY,
    order_id    INTEGER REFERENCES orders(order_id),   -- FK, no auto-index
    product_id  INTEGER REFERENCES products(product_id), -- FK, no auto-index
    quantity    INTEGER,
    unit_price  NUMERIC(10,2)
);
```

Why does this matter? Consider a typical TrailShop query:

```sql
SELECT o.order_id, p.name, oi.quantity, oi.unit_price
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE o.customer_id = 42;
```

Without an index on `order_items.order_id`, PostgreSQL must do a **sequential scan** of the entire `order_items` table for every order. With millions of order items, this is devastating.

The fix is simple but critical — always index your foreign key columns:

```sql
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
```

As Watt & Eng emphasize in *Database Design — 2nd Edition*: foreign key columns are frequently used in JOIN conditions and should almost always be indexed.

---

## 7 — EXPLAIN and EXPLAIN ANALYZE

Understanding *how* PostgreSQL executes your queries is essential for optimization. The `EXPLAIN` command shows you the **execution plan** — the strategy PostgreSQL chose.

For full documentation, see: [https://www.postgresql.org/docs/current/using-explain.html](https://www.postgresql.org/docs/current/using-explain.html)

### 7.1 EXPLAIN: The Estimated Plan

`EXPLAIN` shows what PostgreSQL *plans* to do without actually running the query:

```sql
EXPLAIN SELECT * FROM products WHERE name = 'TrailMaster X4';
```

Output (without an index):

```
Seq Scan on products  (cost=0.00..25.00 rows=1 width=64)
  Filter: ((name)::text = 'TrailMaster X4'::text)
```

### 7.2 EXPLAIN ANALYZE: The Actual Execution

`EXPLAIN ANALYZE` actually **runs** the query and shows both the plan and real execution statistics:

```sql
EXPLAIN ANALYZE SELECT * FROM products WHERE name = 'TrailMaster X4';
```

Output:

```
Seq Scan on products  (cost=0.00..25.00 rows=1 width=64)
                      (actual time=0.215..0.452 rows=1 loops=1)
  Filter: ((name)::text = 'TrailMaster X4'::text)
  Rows Removed by Filter: 999
Planning Time: 0.085 ms
Execution Time: 0.478 ms
```

### 7.3 Reading the Output

Each line in the output represents a **plan node**. Here's what the numbers mean:

| Field | Meaning |
|---|---|
| `cost=0.00..25.00` | Estimated cost: startup cost (0.00) to total cost (25.00). These are arbitrary units, not milliseconds. |
| `rows=1` | Estimated number of rows this node will output |
| `width=64` | Estimated average width of each row in bytes |
| `actual time=0.215..0.452` | Real time in milliseconds: time to first row .. time to last row |
| `rows=1` (actual) | Actual number of rows returned |
| `loops=1` | How many times this node was executed |
| `Rows Removed by Filter: 999` | Rows examined but didn't match the filter |

### 7.4 BUFFERS: Seeing Disk I/O

Adding `BUFFERS` shows how many pages were read:

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM products WHERE name = 'TrailMaster X4';
```

```
Seq Scan on products  (cost=0.00..25.00 rows=1 width=64)
                      (actual time=0.215..0.452 rows=1 loops=1)
  Filter: ((name)::text = 'TrailMaster X4'::text)
  Rows Removed by Filter: 999
  Buffers: shared hit=15
Planning Time: 0.085 ms
Execution Time: 0.478 ms
```

- `shared hit=15` — 15 pages read from the buffer cache (memory)
- `shared read=X` would mean pages read from disk (slower)

### 7.5 Format Options

PostgreSQL can output plans in different formats:

```sql
EXPLAIN (FORMAT JSON) SELECT * FROM products WHERE price > 100;
EXPLAIN (FORMAT YAML) SELECT * FROM products WHERE price > 100;
EXPLAIN (FORMAT TEXT) SELECT * FROM products WHERE price > 100;  -- default
```

JSON and YAML formats are useful for programmatic analysis and tools like pgMustard.

### 7.6 Warning: ANALYZE with DML Statements

`EXPLAIN ANALYZE` actually **executes** the query. For `SELECT` this is harmless, but for `INSERT`, `UPDATE`, or `DELETE` it will modify data. Always wrap DML analysis in a transaction:

```sql
BEGIN;
EXPLAIN ANALYZE DELETE FROM order_items WHERE quantity = 0;
ROLLBACK;
```

The `ROLLBACK` undoes the deletion, but you still get the execution plan with real statistics.

---

## 8 — Scan Types

When PostgreSQL reads data from a table, it uses one of several scanning strategies. The query planner chooses the strategy based on table size, available indexes, and the estimated selectivity of the query.

### 8.1 Sequential Scan (Seq Scan)

The simplest approach: read every row in the table from start to finish.

```sql
EXPLAIN SELECT * FROM products WHERE description LIKE '%waterproof%';
```

```
Seq Scan on products  (cost=0.00..30.00 rows=50 width=128)
  Filter: (description ~~ '%waterproof%')
```

**When PostgreSQL chooses Seq Scan**:
- No suitable index exists
- The query matches a large percentage of rows (index would be slower due to random I/O)
- The table is very small (a few pages)

### 8.2 Index Scan

PostgreSQL traverses the index to find matching entries, then fetches each matching row from the table (the "heap").

```sql
CREATE INDEX idx_products_name ON products(name);

EXPLAIN SELECT * FROM products WHERE name = 'TrailMaster X4';
```

```
Index Scan using idx_products_name on products  (cost=0.28..8.29 rows=1 width=64)
  Index Cond: ((name)::text = 'TrailMaster X4'::text)
```

The index scan does two things per matching row: (1) look up the pointer in the index, (2) fetch the actual row from the table. Step 2 involves **random I/O**, which is why PostgreSQL sometimes prefers a Seq Scan for queries that match many rows.

### 8.3 Index Only Scan

If the index contains **all columns** needed by the query, PostgreSQL can skip the table entirely:

```sql
CREATE INDEX idx_products_name_price ON products(name) INCLUDE (price);

EXPLAIN SELECT name, price FROM products WHERE name = 'TrailMaster X4';
```

```
Index Only Scan using idx_products_name_price on products  (cost=0.28..4.29 rows=1 width=36)
  Index Cond: (name = 'TrailMaster X4'::text)
```

Index Only Scans are the fastest scan type, but they require a check against the **visibility map** to ensure the data is up-to-date. Regular `VACUUM` keeps the visibility map current.

### 8.4 Bitmap Index Scan + Bitmap Heap Scan

This is a two-step process that combines the benefits of index lookup with the efficiency of sequential reading:

1. **Bitmap Index Scan**: Reads the index and builds a bitmap (a map of which table pages contain matching rows)
2. **Bitmap Heap Scan**: Reads those pages sequentially from the table

```sql
EXPLAIN SELECT * FROM products WHERE price BETWEEN 100 AND 200;
```

```
Bitmap Heap Scan on products  (cost=4.30..15.20 rows=150 width=64)
  Recheck Cond: ((price >= 100) AND (price <= 200))
  ->  Bitmap Index Scan on idx_products_price  (cost=0.00..4.26 rows=150 width=0)
        Index Cond: ((price >= 100) AND (price <= 200))
```

**When PostgreSQL chooses bitmap scans**: Moderate selectivity — too many matching rows for individual index lookups to be efficient, but few enough that scanning the entire table would be wasteful. Also used when combining multiple indexes with `AND`/`OR`.

### 8.5 Scan Type Comparison

| Scan Type | How It Works | When Chosen |
|---|---|---|
| Seq Scan | Reads every row in physical order | No index, low selectivity, tiny tables |
| Index Scan | Looks up index, fetches rows one by one | High selectivity, few matching rows |
| Index Only Scan | Reads only from the index | All needed columns in index, high selectivity |
| Bitmap Index Scan → Bitmap Heap Scan | Builds page bitmap from index, reads pages sequentially | Moderate selectivity, combining multiple conditions |

---

## 9 — Join Algorithms

When a query joins two or more tables, PostgreSQL must decide *how* to combine them. The choice of join algorithm has a significant impact on performance.

### 9.1 Nested Loop Join

The simplest algorithm. For each row in the outer table, scan the inner table for matching rows.

```sql
EXPLAIN ANALYZE
SELECT o.order_id, oi.product_id, oi.quantity
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.customer_id = 42;
```

```
Nested Loop  (cost=0.56..24.80 rows=5 width=16)
             (actual time=0.030..0.045 rows=5 loops=1)
  ->  Index Scan using idx_orders_customer_id on orders o
        (cost=0.28..8.29 rows=1 width=4)
        Index Cond: (customer_id = 42)
  ->  Index Scan using idx_order_items_order_id on order_items oi
        (cost=0.28..16.40 rows=5 width=16)
        Index Cond: (order_id = o.order_id)
```

**Complexity**: O(n × m) in the worst case, but with an index on the inner table, it becomes O(n × log m).

**When chosen**: Small outer result set with an indexed inner table. Very efficient for highly selective queries.

### 9.2 Hash Join

PostgreSQL builds a **hash table** from the smaller input, then probes it with each row from the larger input.

```sql
EXPLAIN ANALYZE
SELECT c.name, COUNT(o.order_id)
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.name;
```

```
HashAggregate  (cost=85.00..90.00 rows=500 width=40)
  ->  Hash Join  (cost=15.00..80.00 rows=2000 width=36)
        Hash Cond: (o.customer_id = c.customer_id)
        ->  Seq Scan on orders o  (cost=0.00..55.00 rows=2000 width=8)
        ->  Hash  (cost=10.00..10.00 rows=500 width=36)
              ->  Seq Scan on customers c  (cost=0.00..10.00 rows=500 width=36)
```

**How it works**:
1. Scan the smaller table (customers) and build a hash table keyed by `customer_id`
2. Scan the larger table (orders) and for each row, look up `customer_id` in the hash table

**Complexity**: O(n + m) — much better than nested loop for large, unindexed joins.

**When chosen**: Equality joins on unindexed columns, or when both tables are large.

### 9.3 Merge Join

Both inputs are sorted on the join key, then merged together in a single pass.

```sql
EXPLAIN ANALYZE
SELECT p.name, c.category_name
FROM products p
JOIN categories c ON p.category_id = c.category_id
ORDER BY p.category_id;
```

```
Merge Join  (cost=50.00..70.00 rows=1000 width=64)
  Merge Cond: (p.category_id = c.category_id)
  ->  Sort  (cost=40.00..42.50 rows=1000 width=40)
        Sort Key: p.category_id
        ->  Seq Scan on products p  (cost=0.00..25.00 rows=1000 width=40)
  ->  Sort  (cost=5.00..5.25 rows=10 width=24)
        Sort Key: c.category_id
        ->  Seq Scan on categories c  (cost=0.00..1.10 rows=10 width=24)
```

**How it works**: Like merging two sorted decks of cards — advance through both inputs simultaneously, matching on the join key.

**Complexity**: O(n log n + m log m) for the sorts, then O(n + m) for the merge.

**When chosen**: Both inputs are already sorted (or can be sorted cheaply), large datasets, range joins.

### 9.4 Join Algorithm Comparison

| Algorithm | Complexity | Best For | Requires |
|---|---|---|---|
| Nested Loop | O(n × m), O(n × log m) with index | Small outer + indexed inner | Index on inner table (ideally) |
| Hash Join | O(n + m) | Large unindexed equality joins | Enough memory for hash table |
| Merge Join | O(n log n + m log m) | Large pre-sorted datasets | Sortable join keys |

---

## 10 — Before/After Examples

Let's see the dramatic impact of indexes on real TrailShop queries.

### 10.1 Query 1: Product Lookup by Name

**Before** (no index on `name`):

```sql
EXPLAIN ANALYZE SELECT * FROM products WHERE name = 'TrailMaster X4';
```

```
Seq Scan on products  (cost=0.00..2500.00 rows=1 width=128)
                      (actual time=12.450..45.230 rows=1 loops=1)
  Filter: ((name)::text = 'TrailMaster X4'::text)
  Rows Removed by Filter: 199999
Planning Time: 0.120 ms
Execution Time: 45.280 ms
```

**After** (with index):

```sql
CREATE INDEX idx_products_name ON products(name);

EXPLAIN ANALYZE SELECT * FROM products WHERE name = 'TrailMaster X4';
```

```
Index Scan using idx_products_name on products  (cost=0.42..8.44 rows=1 width=128)
                                                (actual time=0.025..0.027 rows=1 loops=1)
  Index Cond: ((name)::text = 'TrailMaster X4'::text)
Planning Time: 0.150 ms
Execution Time: 0.045 ms
```

**Result**: 45 ms → 0.045 ms — a **1,000× improvement**.

### 10.2 Query 2: Order Items by Foreign Key

**Before** (no index on `order_items.order_id`):

```sql
EXPLAIN ANALYZE
SELECT * FROM order_items WHERE order_id = 12345;
```

```
Seq Scan on order_items  (cost=0.00..45000.00 rows=3 width=32)
                         (actual time=89.100..312.500 rows=3 loops=1)
  Filter: (order_id = 12345)
  Rows Removed by Filter: 2999997
Planning Time: 0.080 ms
Execution Time: 312.550 ms
```

**After**:

```sql
CREATE INDEX idx_order_items_order_id ON order_items(order_id);

EXPLAIN ANALYZE
SELECT * FROM order_items WHERE order_id = 12345;
```

```
Index Scan using idx_order_items_order_id on order_items  (cost=0.43..12.50 rows=3 width=32)
                                                          (actual time=0.018..0.025 rows=3 loops=1)
  Index Cond: (order_id = 12345)
Planning Time: 0.120 ms
Execution Time: 0.040 ms
```

**Result**: 312 ms → 0.04 ms — a **7,800× improvement**.

### 10.3 Query 3: Join Query with Algorithm Change

**Before** (no indexes on FK columns):

```sql
EXPLAIN ANALYZE
SELECT c.name, p.name AS product, oi.quantity
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE c.email = 'maria@example.com';
```

```
Hash Join  (cost=55000.00..95000.00 rows=5 width=64)
           (actual time=850.200..1520.400 rows=5 loops=1)
  Hash Cond: (oi.product_id = p.product_id)
  ->  Hash Join  (cost=45000.00..85000.00 rows=5 width=40)
        Hash Cond: (oi.order_id = o.order_id)
        ->  Seq Scan on order_items oi  (cost=0.00..45000.00 rows=3000000 width=16)
        ->  Hash  (...)
              ->  Hash Join  (cost=20.00..35.00 rows=3 width=24)
                    ->  Seq Scan on orders o  (...)
                    ->  Hash  (...)
                          ->  Seq Scan on customers c  (...)
                                Filter: (email = 'maria@example.com')
  ->  Hash  (...)
        ->  Seq Scan on products p  (cost=0.00..2500.00 rows=200000 width=24)
Planning Time: 1.200 ms
Execution Time: 1521.000 ms
```

**After** (with indexes on FK columns and email):

```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
CREATE INDEX idx_customers_email ON customers(email);

EXPLAIN ANALYZE
SELECT c.name, p.name AS product, oi.quantity
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE c.email = 'maria@example.com';
```

```
Nested Loop  (cost=1.14..45.80 rows=5 width=64)
             (actual time=0.050..0.120 rows=5 loops=1)
  ->  Nested Loop  (cost=0.85..37.00 rows=5 width=44)
        ->  Nested Loop  (cost=0.57..28.50 rows=3 width=12)
              ->  Index Scan using idx_customers_email on customers c
                    (cost=0.28..8.30 rows=1 width=36)
                    Index Cond: (email = 'maria@example.com')
              ->  Index Scan using idx_orders_customer_id on orders o
                    (cost=0.29..20.10 rows=3 width=8)
                    Index Cond: (customer_id = c.customer_id)
        ->  Index Scan using idx_order_items_order_id on order_items oi
              (cost=0.28..2.80 rows=2 width=16)
              Index Cond: (order_id = o.order_id)
  ->  Index Scan using products_pkey on products p
        (cost=0.29..1.75 rows=1 width=24)
        Index Cond: (product_id = oi.product_id)
Planning Time: 0.500 ms
Execution Time: 0.150 ms
```

**Result**: 1,521 ms → 0.15 ms — a **10,000× improvement**. The planner switched from Hash Joins with Seq Scans to Nested Loops with Index Scans because the indexes made targeted lookups far cheaper.

---

## 11 — When to Create Indexes

### 11.1 Decision Criteria

Not every column needs an index. Focus on columns that appear in:

- **WHERE** clauses — especially with equality or range conditions
- **JOIN** conditions — particularly foreign key columns
- **ORDER BY** clauses — avoids expensive sorting
- **GROUP BY** clauses — can speed up aggregation

### 11.2 High Selectivity

**Selectivity** measures what fraction of rows a query condition matches. High selectivity (few matching rows) benefits most from indexes.

| Column | Example Values | Selectivity | Index Useful? |
|---|---|---|---|
| `email` | Unique per customer | Very high | Yes |
| `product_id` (FK) | Many distinct products | High | Yes |
| `order_date` | Many distinct dates | High | Yes |
| `status` | 'pending', 'shipped', 'delivered' | Low | Usually no |
| `is_active` | TRUE, FALSE | Very low | No |

### 11.3 Practical Guidelines for TrailShop

```sql
-- Always index foreign keys
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
CREATE INDEX idx_products_category_id ON products(category_id);
CREATE INDEX idx_payments_order_id ON payments(order_id);

-- Index columns used in WHERE clauses
CREATE INDEX idx_customers_email ON customers(email);
CREATE INDEX idx_orders_order_date ON orders(order_date);

-- Multi-column index for common query pattern
CREATE INDEX idx_orders_status_date ON orders(status, order_date);
```

---

## 12 — When NOT to Create Indexes

Indexes are not free. Every index comes with costs, and over-indexing can be worse than under-indexing.

### 12.1 Small Tables

If a table has only a few hundred rows, a sequential scan is virtually instantaneous. The overhead of maintaining an index isn't worth the negligible lookup improvement.

```sql
-- The categories table has 10 rows — no index needed beyond the PK
SELECT * FROM categories WHERE category_name = 'Footwear';
-- Seq Scan is fine here: 10 rows, microseconds
```

### 12.2 Low Selectivity Columns

Indexing a column with very few distinct values (like a boolean `is_active` or a `status` with three possible values) is usually pointless. If 40% of rows match the condition, PostgreSQL will likely choose a Seq Scan anyway because fetching 40% of rows via random I/O (index scan) is slower than a sequential read.

```sql
-- Poor index candidate: low selectivity
CREATE INDEX idx_orders_status ON orders(status);  -- only 3-4 distinct values

-- Better approach: partial index if you query one specific value
CREATE INDEX idx_orders_pending ON orders(order_date) WHERE status = 'pending';
```

### 12.3 Write-Heavy Tables

Every `INSERT`, `UPDATE`, and `DELETE` must also update **every index** on the table. For write-heavy tables (like logging tables receiving thousands of inserts per second), excessive indexes can severely degrade write performance.

| Operation | Without Indexes | With 5 Indexes |
|---|---|---|
| INSERT 1 row | 1 table write | 1 table write + 5 index writes |
| UPDATE 1 column | 1 table write | 1 table write + affected index writes |
| DELETE 1 row | 1 table write | 1 table write + 5 index writes |

### 12.4 Cost-Benefit: Read/Write Ratio

Consider the read/write ratio of the table:

- **Read-heavy** (products catalog, reference data): More indexes are beneficial
- **Write-heavy** (event logs, analytics events): Minimize indexes
- **Balanced** (orders, user sessions): Index selectively based on query patterns

### 12.5 Over-Indexing Dangers

- **Wasted disk space**: Each index takes storage. A table with 10 indexes might have more total index storage than the table itself.
- **Slower writes**: Every write operation pays the index maintenance tax.
- **Planner confusion**: Too many indexes can cause the query planner to spend extra time evaluating options, and it might occasionally pick a suboptimal index.
- **Maintenance burden**: Unused indexes still consume resources during `VACUUM` and other maintenance operations.

---

## 13 — Index Maintenance

Indexes, like the tables they serve, require ongoing maintenance to stay healthy and performant.

### 13.1 VACUUM and Dead Tuples

PostgreSQL uses MVCC (Multi-Version Concurrency Control) — when you update or delete a row, the old version isn't immediately removed. It becomes a **dead tuple**. Over time, dead tuples accumulate and cause **bloat** — wasted space in both tables and indexes.

`VACUUM` reclaims space from dead tuples:

```sql
-- Manual vacuum on a specific table
VACUUM products;

-- Verbose output shows what was cleaned
VACUUM VERBOSE products;

-- VACUUM FULL rewrites the entire table (locks table, use sparingly)
VACUUM FULL products;
```

PostgreSQL runs **autovacuum** in the background, which automatically vacuums tables when dead tuples accumulate beyond a threshold. In most cases, autovacuum handles this for you — but monitoring is still important.

### 13.2 REINDEX: Rebuilding Bloated Indexes

Even with regular `VACUUM`, B-tree indexes can become bloated over time — pages that were once full may become sparsely populated. `REINDEX` rebuilds an index from scratch:

```sql
-- Rebuild a specific index
REINDEX INDEX idx_products_name;

-- Rebuild all indexes on a table
REINDEX TABLE products;

-- Non-blocking rebuild in production
REINDEX INDEX CONCURRENTLY idx_products_name;
```

### 13.3 Monitoring Index Usage

PostgreSQL tracks index usage statistics. Use these to find unused indexes that can be dropped:

```sql
-- See how often each index is used
SELECT
    schemaname,
    relname AS table_name,
    indexrelname AS index_name,
    idx_scan AS times_used,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;
```

An index with `idx_scan = 0` over a long period is a candidate for removal — it's consuming space and slowing writes without benefiting any query.

### 13.4 Sequential Scans vs Index Scans

Monitor whether tables are being scanned sequentially when they shouldn't be:

```sql
SELECT
    relname AS table_name,
    seq_scan,
    idx_scan,
    CASE WHEN seq_scan + idx_scan > 0
         THEN ROUND(100.0 * idx_scan / (seq_scan + idx_scan), 1)
         ELSE 0
    END AS index_scan_pct,
    n_live_tup AS row_count
FROM pg_stat_user_tables
WHERE n_live_tup > 10000
ORDER BY seq_scan DESC;
```

A large table with a high `seq_scan` count and low `idx_scan` count may be missing an index.

### 13.5 Index Size Monitoring

Track how much space your indexes consume:

```sql
SELECT
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexname::regclass)) AS index_size
FROM pg_indexes
WHERE schemaname = 'public'
ORDER BY pg_relation_size(indexname::regclass) DESC;
```

---

## 14 — Query Optimization Tips

Indexes are the most powerful optimization tool, but they're not the only one. Here are additional techniques for writing faster queries.

### 14.1 Avoid SELECT *

Fetching all columns forces PostgreSQL to read the full row, even if you only need two columns. This wastes I/O and prevents Index Only Scans.

```sql
-- Slow: fetches all columns
SELECT * FROM products WHERE category_id = 2;

-- Fast: fetches only what you need, may enable Index Only Scan
SELECT name, price FROM products WHERE category_id = 2;
```

### 14.2 EXISTS vs IN vs JOIN

For existence checks, `EXISTS` is usually the best choice because it can stop searching as soon as it finds one match:

```sql
-- Good: EXISTS stops at the first match
SELECT c.name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);

-- Often equivalent, but can be slower with large subquery results
SELECT c.name
FROM customers c
WHERE c.customer_id IN (
    SELECT customer_id FROM orders
);

-- JOIN may return duplicates if a customer has multiple orders
SELECT DISTINCT c.name
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```

### 14.3 Don't Wrap Indexed Columns in Functions

Applying a function to an indexed column in a `WHERE` clause prevents PostgreSQL from using the index:

```sql
-- ✗ Cannot use index on order_date (function applied to column)
SELECT * FROM orders WHERE EXTRACT(YEAR FROM order_date) = 2026;

-- ✓ Can use index on order_date (range condition on raw column)
SELECT * FROM orders
WHERE order_date >= '2026-01-01' AND order_date < '2027-01-01';
```

If you must use a function, create an expression index (see Section 5.5).

### 14.4 LIMIT with ORDER BY

When you need only the top N results, `LIMIT` combined with an appropriate index avoids sorting the entire result set:

```sql
-- With an index on order_date, PostgreSQL reads only 10 index entries
SELECT * FROM orders
ORDER BY order_date DESC
LIMIT 10;
```

### 14.5 Materialized Views

For expensive queries that are run frequently but don't need real-time data, materialized views store the precomputed result:

```sql
CREATE MATERIALIZED VIEW mv_sales_summary AS
SELECT
    p.name AS product_name,
    c.category_name,
    SUM(oi.quantity) AS total_sold,
    SUM(oi.quantity * oi.unit_price) AS total_revenue
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
GROUP BY p.name, c.category_name;

-- Query the materialized view (fast — reads precomputed data)
SELECT * FROM mv_sales_summary ORDER BY total_revenue DESC LIMIT 10;

-- Refresh when you need updated data
REFRESH MATERIALIZED VIEW mv_sales_summary;
```

### 14.6 The ANALYZE Command

PostgreSQL maintains **statistics** about the data in each column — the number of distinct values, most common values, histogram of value distribution. The query planner uses these statistics to estimate costs and choose execution plans.

`ANALYZE` updates these statistics:

```sql
-- Update statistics for a specific table
ANALYZE products;

-- Update statistics for all tables
ANALYZE;
```

PostgreSQL's autovacuum process runs `ANALYZE` automatically, but after bulk data loads or major changes, running it manually ensures the planner has accurate information.

---

## Key Terms

| Term | Definition |
|---|---|
| **Sequential Scan (Seq Scan)** | A scan that reads every row in a table from beginning to end |
| **Index** | A separate data structure that maps column values to row locations for fast lookup |
| **B-tree** | A balanced tree data structure used as the default index type; provides O(log n) lookup |
| **Hash Index** | An index using a hash function; supports only equality comparisons |
| **GIN (Generalized Inverted Index)** | An index type for multi-element values: arrays, JSONB, full-text search |
| **GiST (Generalized Search Tree)** | An index type for complex data: spatial, range, nearest-neighbor queries |
| **BRIN (Block Range Index)** | A compact index storing min/max per block range; ideal for large sequential tables |
| **Index Scan** | A scan that traverses an index then fetches matching rows from the table |
| **Index Only Scan** | A scan answered entirely from the index without accessing the table |
| **Bitmap Index Scan** | A scan that builds a bitmap of matching pages from the index, then reads those pages |
| **Execution Plan** | The strategy PostgreSQL chooses to execute a query, viewable with EXPLAIN |
| **Cost Model** | The system of arbitrary cost units PostgreSQL uses to estimate and compare plan efficiency |
| **Selectivity** | The fraction of rows a condition matches; high selectivity = few matching rows |
| **Covering Index** | An index that includes all columns needed by a query, enabling Index Only Scans |
| **Partial Index** | An index that only includes rows matching a WHERE condition |
| **Expression Index** | An index built on the result of a function or expression rather than a raw column |
| **Nested Loop Join** | A join algorithm that scans the inner table for each row of the outer table |
| **Hash Join** | A join algorithm that builds a hash table from one input and probes it with the other |
| **Merge Join** | A join algorithm that merges two sorted inputs |
| **VACUUM** | A maintenance operation that reclaims space from dead tuples |
| **Bloat** | Wasted space in tables and indexes caused by accumulated dead tuples |
| **Dead Tuple** | An obsolete row version left behind by UPDATE or DELETE under MVCC |
| **Visibility Map** | A structure tracking which table pages contain only visible (up-to-date) tuples |
| **Autovacuum** | A background process that automatically runs VACUUM and ANALYZE |

---

## Reading Assignments

1. **PostgreSQL Documentation — Indexes**
   [https://www.postgresql.org/docs/current/indexes.html](https://www.postgresql.org/docs/current/indexes.html)
   Read the overview and sections on index types, multi-column indexes, and partial indexes.

2. **PostgreSQL Documentation — Using EXPLAIN**
   [https://www.postgresql.org/docs/current/using-explain.html](https://www.postgresql.org/docs/current/using-explain.html)
   Read the full page. Practice reading EXPLAIN output on your own queries.

3. **Watt & Eng — Database Design, 2nd Edition**
   Review the chapter on physical database design and indexing strategies.

---

## Further Reading

- **Use The Index, Luke** — [https://use-the-index-luke.com/](https://use-the-index-luke.com/)
  An excellent free online book dedicated entirely to database indexing. Covers B-tree internals, multi-column indexes, partial indexes, and query tuning with clear explanations and diagrams.

- **pgMustard** — [https://www.pgmustard.com/](https://www.pgmustard.com/)
  A tool that analyzes PostgreSQL EXPLAIN output and provides actionable optimization tips. Useful for learning to read execution plans.

- **PostgreSQL Wiki — Performance Optimization**
  [https://wiki.postgresql.org/wiki/Performance_Optimization](https://wiki.postgresql.org/wiki/Performance_Optimization)
  Community-maintained collection of performance tips, configuration advice, and monitoring queries.

---

## Summary

This week you learned that query performance isn't magic — it's engineering. Without indexes, PostgreSQL must read every row in a table (O(n)), and performance degrades linearly as data grows. Indexes — primarily B-trees — provide O(log n) lookups by maintaining a sorted, balanced tree structure separate from the table data.

PostgreSQL offers specialized index types beyond B-tree: Hash for pure equality, GIN for arrays and full-text search, GiST for spatial data, and BRIN for large time-series tables. You learned to create basic, unique, multi-column, partial, expression, and covering indexes — and critically, that foreign key columns do **not** get automatic indexes.

With `EXPLAIN` and `EXPLAIN ANALYZE`, you can see exactly how PostgreSQL executes a query — which scan type it uses, which join algorithm it picks, and how long each step takes. You saw real before/after examples where proper indexing delivered improvements of 1,000× to 10,000×.

But indexing is not a silver bullet. Every index adds write overhead, consumes storage, and requires maintenance. The art is in choosing the *right* indexes for your query patterns while avoiding over-indexing.

**Next week**: We move from optimizing queries to managing the database itself — **database administration**. You'll learn about managing users and roles, securing data with permissions, backing up and restoring databases, and we'll take a first look at NoSQL alternatives.
