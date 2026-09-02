# Week 46 — Transactions and Concurrency

## Chapter 10: "Black Friday"

It's November, and TrailShop is bracing for its biggest sale of the year. The founders are excited — but you're worried. What happens when 200 customers try to buy the last 50 pairs of Trail Runner X *at the same time*? What if the system crashes halfway through processing an order — money debited but items never shipped? What if two warehouse workers update the same stock count simultaneously?

These aren't hypothetical problems. They happen every day in production systems. The solution lies in **transactions** — the database's mechanism for guaranteeing correctness even when things go wrong or multiple operations happen simultaneously.

As the PostgreSQL documentation states: "Transactions are a fundamental concept of all database systems. The essential point of a transaction is that it bundles multiple steps into a single, all-or-nothing operation."

Reference: [PostgreSQL Tutorial — Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html)

---

## Learning Objectives

By the end of this chapter you will be able to:

- Define what a transaction is and explain its purpose
- Describe the ACID properties with real-world examples
- Use BEGIN, COMMIT, ROLLBACK, and SAVEPOINT to control transactions
- Identify the four concurrency problems (dirty read, non-repeatable read, phantom read, lost update)
- Explain PostgreSQL's isolation levels and their trade-offs
- Describe how locking and MVCC enable concurrent access
- Recognize and prevent deadlocks

---

## 1. What Is a Transaction?

### 1.1 Definition

A **transaction** is a sequence of one or more SQL operations that are treated as a **single logical unit of work**. Either ALL operations in the transaction succeed (commit), or NONE of them take effect (rollback).

### 1.2 Real-World Analogies

#### Bank Transfer

When you transfer €100 from Account A to Account B:
1. Debit €100 from Account A
2. Credit €100 to Account B

These two steps **must** happen together. If the system crashes after step 1 but before step 2, the €100 vanishes. A transaction ensures both happen or neither happens.

#### ATM Withdrawal

1. Verify PIN
2. Check balance ≥ withdrawal amount
3. Debit account
4. Dispense cash
5. Print receipt

If the machine jams after debiting but before dispensing cash, the debit must be reversed. The whole sequence is one transaction.

#### TrailShop Order

1. Verify product is in stock
2. Create order record
3. Create order_items records
4. Decrease stock_quantity
5. Charge payment

If payment fails at step 5, steps 2–4 must be undone. The customer shouldn't be charged for an order that wasn't completed, and stock shouldn't decrease for items that weren't sold.

### 1.3 Transaction Boundaries

A transaction has a clear **start** and **end**:

```
BEGIN → [SQL operations] → COMMIT (success)
                        → ROLLBACK (failure/cancellation)
```

Everything between BEGIN and COMMIT/ROLLBACK is one atomic unit.

---

## 2. ACID Properties

ACID is the set of guarantees that make transactions reliable. The term was coined by Andreas Reuter and Theo Härder in 1983.

Reference: [PostgreSQL — Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)

### 2.1 Atomicity — "All or Nothing"

**Definition**: A transaction is an indivisible unit. Either all its operations complete successfully, or none of them take effect.

**TrailShop example**: An order involves inserting into `orders`, inserting into `order_items`, and updating `products.stock_quantity`. If the stock update fails (e.g., constraint violation because stock would go negative), the entire order — including the already-inserted rows — is rolled back.

**What happens when atomicity is violated**: Partial operations persist. The system is left in an inconsistent state — an order exists but stock wasn't decremented, or payment was processed but no order was recorded.

### 2.2 Consistency — "Valid State to Valid State"

**Definition**: A transaction transforms the database from one valid state to another. All constraints (CHECK, UNIQUE, FOREIGN KEY, NOT NULL) must be satisfied after the transaction completes.

**TrailShop example**: If `stock_quantity` has a CHECK constraint `stock_quantity >= 0`, a transaction that would reduce stock below zero violates consistency and is rejected.

**What happens when consistency is violated**: The database contains data that breaks business rules — negative stock, orders referencing non-existent customers, duplicate primary keys.

### 2.3 Isolation — "Transactions Don't Interfere"

**Definition**: Concurrent transactions execute as if they were running one at a time (serially). One transaction's intermediate state is invisible to other transactions.

**TrailShop example**: Customer A and Customer B both try to buy the last Trail Runner X. Isolation ensures that one transaction sees the stock decrease from 1 to 0, and the other sees 0 stock and gets rejected — rather than both seeing 1 and both succeeding.

**What happens when isolation is violated**: Concurrency problems — dirty reads, non-repeatable reads, phantom reads, lost updates (detailed in Section 6).

### 2.4 Durability — "Committed Data Survives"

**Definition**: Once a transaction is committed, its changes are permanent — even if the system crashes immediately after.

**Implementation**: PostgreSQL writes committed data to the Write-Ahead Log (WAL) on disk before confirming the commit. Even if the server loses power, the WAL can replay committed transactions during recovery.

**TrailShop example**: After the system says "Order confirmed," the order persists even if the database server crashes one millisecond later.

**What happens when durability is violated**: The system confirms an action, but the data is lost after a crash. Customers think their order was placed, but it's gone.

---

## 3. Transaction Control

### 3.1 BEGIN / START TRANSACTION

Starts a new transaction explicitly:

```sql
BEGIN;
-- or equivalently:
START TRANSACTION;
```

Both forms are equivalent in PostgreSQL.

### 3.2 COMMIT

Makes all changes in the current transaction permanent:

```sql
BEGIN;
INSERT INTO orders (customer_id, order_date, status)
VALUES (1, CURRENT_DATE, 'pending');
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (currval('orders_order_id_seq'), 101, 2, 89.99);
UPDATE products SET stock_quantity = stock_quantity - 2 WHERE product_id = 101;
COMMIT;
```

After COMMIT, the changes are visible to all other transactions and survive crashes.

### 3.3 ROLLBACK

Undoes all changes in the current transaction:

```sql
BEGIN;
UPDATE products SET stock_quantity = stock_quantity - 100 WHERE product_id = 101;
-- Oops, that's too many! Undo everything.
ROLLBACK;
```

After ROLLBACK, it's as if the transaction never happened.

### 3.4 SAVEPOINT

Creates a named checkpoint within a transaction. You can roll back to a savepoint without aborting the entire transaction.

```sql
BEGIN;

INSERT INTO orders (customer_id, order_date, status)
VALUES (1, CURRENT_DATE, 'pending');

SAVEPOINT after_order;

INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (currval('orders_order_id_seq'), 999, 1, 50.00);
-- Product 999 doesn't exist — foreign key error!

ROLLBACK TO SAVEPOINT after_order;
-- The order INSERT is preserved, but the failed order_item is undone.

INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (currval('orders_order_id_seq'), 101, 1, 89.99);
-- This one works.

COMMIT;
```

### 3.5 ROLLBACK TO SAVEPOINT

Rolls back all operations *after* the named savepoint, but keeps the transaction open and preserves everything before the savepoint.

```sql
ROLLBACK TO SAVEPOINT my_savepoint;
```

### 3.6 RELEASE SAVEPOINT

Removes a savepoint without rolling back. The savepoint is no longer available for rollback, but the work done after it is preserved.

```sql
RELEASE SAVEPOINT my_savepoint;
```

This is useful for cleanup — it signals "I no longer need this checkpoint."

### 3.7 Full Syntax Reference

```sql
BEGIN [ WORK | TRANSACTION ] [ transaction_mode [, ...] ];
COMMIT [ WORK | TRANSACTION ];
ROLLBACK [ WORK | TRANSACTION ];
SAVEPOINT savepoint_name;
ROLLBACK TO [ SAVEPOINT ] savepoint_name;
RELEASE [ SAVEPOINT ] savepoint_name;
```

Reference: [PostgreSQL — SQL Commands](https://www.postgresql.org/docs/current/sql-begin.html)

---

## 4. Autocommit Mode

### 4.1 How PostgreSQL Handles It

By default, PostgreSQL operates in **autocommit mode**: every individual SQL statement is automatically wrapped in its own transaction. If you type:

```sql
UPDATE products SET price = 99.99 WHERE product_id = 101;
```

Without an explicit `BEGIN`, PostgreSQL internally does:
```sql
BEGIN;
UPDATE products SET price = 99.99 WHERE product_id = 101;
COMMIT;
```

This means every statement is immediately committed. There's no way to undo it (short of writing another UPDATE).

### 4.2 When to Use Explicit Transactions

Use explicit `BEGIN ... COMMIT` when:
- You have **multiple related statements** that must succeed or fail together.
- You're doing **destructive operations** and want the ability to ROLLBACK.
- You need a specific **isolation level**.

### 4.3 Disabling Autocommit

In `psql`, autocommit is controlled by the `\set AUTOCOMMIT off` variable:

```
\set AUTOCOMMIT off
```

Now every statement starts an implicit transaction that you must explicitly COMMIT or ROLLBACK. Most applications and ORMs handle transaction boundaries programmatically.

---

## 5. Complete TrailShop Transaction Examples

### 5.1 Order Processing

```sql
BEGIN;

-- 1. Create the order
INSERT INTO orders (customer_id, order_date, status)
VALUES (3, CURRENT_DATE, 'pending')
RETURNING order_id;
-- Assume returns order_id = 1050

-- 2. Add items
INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES
(1050, 101, 2, 89.99),
(1050, 202, 1, 45.00);

-- 3. Decrease stock
UPDATE products SET stock_quantity = stock_quantity - 2 WHERE product_id = 101;
UPDATE products SET stock_quantity = stock_quantity - 1 WHERE product_id = 202;

-- 4. Verify stock didn't go negative (optional explicit check)
DO $$
BEGIN
    IF EXISTS (SELECT 1 FROM products WHERE product_id IN (101, 202) AND stock_quantity < 0) THEN
        RAISE EXCEPTION 'Insufficient stock';
    END IF;
END $$;

COMMIT;
```

If *any* step fails — the entire order, all item inserts, and all stock updates are undone.

### 5.2 Stock Transfer Between Stores (Hypothetical)

```sql
BEGIN;

SAVEPOINT before_transfer;

-- Remove from source store
UPDATE store_inventory
SET quantity = quantity - 20
WHERE store_id = 1 AND product_id = 101;

-- Verify source didn't go negative
IF (SELECT quantity FROM store_inventory WHERE store_id = 1 AND product_id = 101) < 0 THEN
    ROLLBACK TO SAVEPOINT before_transfer;
    -- Handle error
END IF;

-- Add to destination store
UPDATE store_inventory
SET quantity = quantity + 20
WHERE store_id = 2 AND product_id = 101;

COMMIT;
```

### 5.3 Payment Processing (Simplified)

```sql
BEGIN;

-- Mark order as processing
UPDATE orders SET status = 'processing' WHERE order_id = 1050;

SAVEPOINT before_payment;

-- Attempt to record payment
INSERT INTO payments (order_id, amount, payment_method, payment_date)
VALUES (1050, 224.98, 'credit_card', NOW());

-- If payment gateway returns failure:
-- ROLLBACK TO SAVEPOINT before_payment;
-- UPDATE orders SET status = 'payment_failed' WHERE order_id = 1050;
-- COMMIT;

-- On success:
UPDATE orders SET status = 'paid' WHERE order_id = 1050;
COMMIT;
```

---

## 6. Concurrency Problems

When multiple transactions execute simultaneously, they can interfere with each other. These are the four classic problems.

### 6.1 Dirty Read

**Definition**: Transaction A reads data that Transaction B has written but **not yet committed**. If B rolls back, A has read data that never officially existed.

**Timeline:**

```
Time    Transaction A                Transaction B
─────   ────────────────────────     ────────────────────────
T1                                   BEGIN
T2                                   UPDATE products SET price = 50.00
                                     WHERE product_id = 101;  (was 89.99)
T3      BEGIN
T4      SELECT price FROM products
        WHERE product_id = 101;
        → Reads 50.00 (UNCOMMITTED!)
T5                                   ROLLBACK  (price goes back to 89.99)
T6      -- A is using price 50.00,
        -- but it never really was 50.00!
```

**TrailShop impact**: Customer sees a discounted price that was never actually saved. Order is placed at wrong price.

**PostgreSQL behavior**: PostgreSQL **never** allows dirty reads, even at its lowest isolation level (READ COMMITTED).

### 6.2 Non-Repeatable Read

**Definition**: Transaction A reads the same row twice and gets **different values** because Transaction B modified and committed it in between.

**Timeline:**

```
Time    Transaction A                Transaction B
─────   ────────────────────────     ────────────────────────
T1      BEGIN
T2      SELECT stock_quantity FROM
        products WHERE product_id = 101;
        → Returns 10
T3                                   BEGIN
T4                                   UPDATE products SET stock_quantity = 5
                                     WHERE product_id = 101;
T5                                   COMMIT
T6      SELECT stock_quantity FROM
        products WHERE product_id = 101;
        → Returns 5 (DIFFERENT!)
T7      -- A's logic based on "10 in stock"
        -- is now wrong!
```

**TrailShop impact**: A report starts calculating with stock = 10, then later in the same report sees stock = 5. The report is internally inconsistent.

### 6.3 Phantom Read

**Definition**: Transaction A executes a query twice and gets **different sets of rows** because Transaction B inserted or deleted rows that match A's query condition.

**Timeline:**

```
Time    Transaction A                Transaction B
─────   ────────────────────────     ────────────────────────
T1      BEGIN
T2      SELECT COUNT(*) FROM orders
        WHERE customer_id = 1;
        → Returns 3
T3                                   BEGIN
T4                                   INSERT INTO orders (customer_id, ...)
                                     VALUES (1, ...);
T5                                   COMMIT
T6      SELECT COUNT(*) FROM orders
        WHERE customer_id = 1;
        → Returns 4 (PHANTOM row appeared!)
```

**TrailShop impact**: A loyalty calculation counts 3 orders (status: "Regular"), but by the time it applies the discount, there are 4 orders (should be "VIP"). The customer gets the wrong discount.

### 6.4 Lost Update

**Definition**: Two transactions read the same value, both modify it based on what they read, and one overwrites the other's change.

**Timeline:**

```
Time    Transaction A                Transaction B
─────   ────────────────────────     ────────────────────────
T1      BEGIN                        BEGIN
T2      SELECT stock_quantity FROM   SELECT stock_quantity FROM
        products WHERE id = 101;     products WHERE id = 101;
        → Reads 10                   → Reads 10
T3      -- Customer A buys 3        -- Customer B buys 2
T4      UPDATE products SET          UPDATE products SET
        stock_quantity = 10 - 3      stock_quantity = 10 - 2
        WHERE id = 101;              WHERE id = 101;
        -- Sets to 7                 -- Sets to 8
T5      COMMIT                       COMMIT
T6      -- Final stock: 8 (should be 5!)
        -- A's purchase "disappeared"
```

**TrailShop impact**: 5 items were sold but only 2 were subtracted from stock. Inventory becomes wrong. Orders may be accepted for items that don't exist.

---

## 7. Isolation Levels

Isolation levels control how much transactions can "see" each other's work. Higher isolation = fewer problems, but potentially lower performance.

Reference: [PostgreSQL — Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)

### 7.1 The Four SQL Standard Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|-------------------|--------------|
| READ UNCOMMITTED | Possible | Possible | Possible |
| READ COMMITTED | Prevented | Possible | Possible |
| REPEATABLE READ | Prevented | Prevented | Possible* |
| SERIALIZABLE | Prevented | Prevented | Prevented |

*In PostgreSQL, REPEATABLE READ also prevents phantom reads (stricter than the SQL standard requires).

### 7.2 READ UNCOMMITTED

The SQL standard's lowest level. Allows dirty reads.

**PostgreSQL behavior**: PostgreSQL does NOT actually implement READ UNCOMMITTED. If you set it, you get READ COMMITTED instead. This is documented behavior — PostgreSQL considers dirty reads too dangerous to ever allow.

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
-- In PostgreSQL, this is silently treated as READ COMMITTED
```

### 7.3 READ COMMITTED (PostgreSQL Default)

Each statement within a transaction sees only data committed *before that statement began*. Different statements within the same transaction can see different committed data.

```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;  -- (default, optional)

SELECT stock_quantity FROM products WHERE product_id = 101;
-- Sees whatever was committed at this moment

-- Meanwhile, another transaction commits a change...

SELECT stock_quantity FROM products WHERE product_id = 101;
-- May see a DIFFERENT value (non-repeatable read possible)
COMMIT;
```

**When to use**: Most OLTP operations. Good balance of consistency and performance.

### 7.4 REPEATABLE READ

The transaction sees a **snapshot** of the database as of the start of the transaction. All reads within the transaction see the same consistent data, regardless of what other transactions commit.

```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SELECT stock_quantity FROM products WHERE product_id = 101;
-- Returns 10

-- Another transaction commits: stock becomes 5

SELECT stock_quantity FROM products WHERE product_id = 101;
-- STILL returns 10 (snapshot from transaction start)

COMMIT;
```

**PostgreSQL bonus**: In PostgreSQL, REPEATABLE READ also prevents phantom reads (thanks to MVCC/snapshot isolation).

**Conflict handling**: If your transaction tries to UPDATE a row that another committed transaction already modified, PostgreSQL throws:
```
ERROR: could not serialize access due to concurrent update
```
Your application must catch this and retry the transaction.

### 7.5 SERIALIZABLE

The strictest level. Guarantees that concurrent transactions produce the same result as if they ran one after another (serially). PostgreSQL uses **Serializable Snapshot Isolation (SSI)** to detect conflicts.

```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- All operations here guaranteed to be conflict-free
-- If a conflict is detected, PostgreSQL aborts the transaction:
-- ERROR: could not serialize access due to read/write dependencies

COMMIT;
```

**When to use**: When correctness is critical and you can't tolerate any anomaly. Financial calculations, inventory management where every unit matters.

**Trade-off**: More transactions may be aborted and need retry logic.

### 7.6 Setting Isolation Levels

```sql
-- For the current transaction only:
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- ...
COMMIT;

-- Or combined:
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- ...
COMMIT;

-- For the entire session:
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### 7.7 PostgreSQL-Specific Behavior Summary

| Feature | PostgreSQL Behavior |
|---------|-------------------|
| READ UNCOMMITTED | Treated as READ COMMITTED |
| Dirty reads | Never possible (not even at READ UNCOMMITTED) |
| REPEATABLE READ | Also prevents phantom reads (stricter than standard) |
| Default level | READ COMMITTED |
| Conflict resolution | Aborts transaction with serialization error |
| Implementation | MVCC (Multi-Version Concurrency Control) |

---

## 8. Locking

### 8.1 Why Locks Exist

Locks prevent conflicting concurrent access to the same data. They're the mechanism that enforces isolation.

### 8.2 Row-Level Locks

PostgreSQL uses row-level locks to allow maximum concurrency. When a transaction modifies a row, it acquires an **exclusive lock** on that row. Other transactions that want to modify the same row must wait.

```sql
-- Transaction A:
BEGIN;
UPDATE products SET stock_quantity = 8 WHERE product_id = 101;
-- Row 101 is now locked. Other UPDATEs to this row will wait.
-- SELECTs still work! (MVCC — readers don't block writers)
COMMIT;  -- Lock released
```

### 8.3 SELECT FOR UPDATE

Explicitly locks rows for a future UPDATE, preventing other transactions from modifying them:

```sql
BEGIN;

-- Lock the row immediately (other transactions wanting this row will wait)
SELECT stock_quantity FROM products
WHERE product_id = 101
FOR UPDATE;
-- Returns 10, and the row is now locked

-- Safe to compute and update — no one else can change it
UPDATE products SET stock_quantity = 10 - 3 WHERE product_id = 101;

COMMIT;
```

This solves the **lost update** problem! By locking the row before reading, you ensure no one else modifies it between your read and your write.

**Variants:**
- `FOR UPDATE` — strongest; blocks other FOR UPDATE, UPDATE, DELETE
- `FOR NO KEY UPDATE` — blocks updates but allows operations that don't affect the key
- `FOR SHARE` — allows other FOR SHARE but blocks FOR UPDATE
- `FOR KEY SHARE` — weakest; only blocks operations that modify the key

### 8.4 Table-Level Locks

Some operations require locking entire tables:

```sql
LOCK TABLE products IN ACCESS EXCLUSIVE MODE;
```

Table locks are rarely needed in application code. PostgreSQL acquires them automatically for DDL operations (ALTER TABLE, DROP TABLE).

| Lock Mode | Conflicts With | Used For |
|-----------|---------------|----------|
| ACCESS SHARE | ACCESS EXCLUSIVE | SELECT |
| ROW SHARE | EXCLUSIVE, ACCESS EXCLUSIVE | SELECT FOR UPDATE |
| ROW EXCLUSIVE | SHARE, SHARE ROW EXCLUSIVE, EXCLUSIVE, ACCESS EXCLUSIVE | UPDATE, INSERT, DELETE |
| ACCESS EXCLUSIVE | All modes | ALTER TABLE, DROP TABLE |

### 8.5 Advisory Locks (Brief)

Application-defined locks that don't correspond to any table or row. Useful for coordinating application-level logic:

```sql
-- Acquire an advisory lock (ID: 12345)
SELECT pg_advisory_lock(12345);

-- ... do something that should be exclusive ...

-- Release
SELECT pg_advisory_unlock(12345);
```

Advisory locks are managed entirely by your application — PostgreSQL just provides the locking mechanism.

### 8.6 How Locks Interact with Transactions

- Locks are **held until the transaction ends** (COMMIT or ROLLBACK).
- You cannot manually release a row lock before the transaction ends.
- Longer transactions hold locks longer, potentially blocking other operations.
- **Keep transactions short** to minimize lock contention.

---

## 9. MVCC — Multi-Version Concurrency Control

### 9.1 The Key Insight

Traditional locking databases make readers wait for writers and writers wait for readers. PostgreSQL takes a different approach: **readers never block writers, and writers never block readers**.

How? By keeping **multiple versions** of each row.

### 9.2 How It Works

When a transaction updates a row, PostgreSQL doesn't overwrite the old row. Instead, it:
1. Marks the old version as "expired" (but doesn't delete it yet).
2. Creates a **new version** of the row with the updated values.

Both versions coexist temporarily. Different transactions see different versions depending on their snapshot.

### 9.3 Transaction IDs (XIDs)

Every transaction gets a unique, incrementing **transaction ID (XID)**. Each row version has two hidden system columns:
- `xmin` — the XID of the transaction that created this version
- `xmax` — the XID of the transaction that deleted/updated this version (0 if still "live")

```sql
-- You can see these:
SELECT xmin, xmax, * FROM products WHERE product_id = 101;
```

### 9.4 Visibility Rules (High-Level)

A row version is visible to a transaction if:
1. The creating transaction (`xmin`) committed **before** the reading transaction's snapshot.
2. The row hasn't been deleted/updated by a committed transaction that's visible to the reader.

This is how PostgreSQL implements REPEATABLE READ — the transaction takes a snapshot at the start and only sees versions created by transactions that committed before the snapshot.

### 9.5 Why Readers Don't Block Writers

When Transaction A reads product 101, it reads the version visible to its snapshot. When Transaction B updates product 101, it creates a *new* version. A still sees the old version; B works with the new version. No conflict, no waiting.

### 9.6 VACUUM — Cleaning Up Old Versions

Old row versions (no longer visible to any active transaction) accumulate as "dead tuples." The `VACUUM` process reclaims this space:

```sql
VACUUM products;          -- Marks space as reusable
VACUUM FULL products;     -- Rewrites the table (more aggressive, locks table)
```

PostgreSQL's **autovacuum** runs automatically in the background. You rarely need to run VACUUM manually.

---

## 10. Deadlocks

### 10.1 What Is a Deadlock?

A **deadlock** occurs when two or more transactions each hold a lock that the other needs, creating a circular wait from which neither can proceed.

### 10.2 How Deadlocks Happen

```
Time    Transaction A                Transaction B
─────   ────────────────────────     ────────────────────────
T1      BEGIN                        BEGIN
T2      UPDATE products              
        SET stock = stock - 1
        WHERE product_id = 101;
        -- Locks row 101
T3                                   UPDATE products
                                     SET stock = stock - 1
                                     WHERE product_id = 202;
                                     -- Locks row 202
T4      UPDATE products
        SET stock = stock - 1
        WHERE product_id = 202;
        -- WAITS (B holds lock on 202)
T5                                   UPDATE products
                                     SET stock = stock - 1
                                     WHERE product_id = 101;
                                     -- WAITS (A holds lock on 101)
T6      -- DEADLOCK! Both waiting for each other forever.
```

### 10.3 How PostgreSQL Detects Deadlocks

PostgreSQL runs a **deadlock detector** (by default every second, controlled by `deadlock_timeout`). When detected, PostgreSQL **aborts one** of the transactions:

```
ERROR: deadlock detected
DETAIL: Process 1234 waits for ShareLock on transaction 5678;
        blocked by process 5679.
        Process 5679 waits for ShareLock on transaction 5677;
        blocked by process 1234.
HINT: See server log for query details.
```

The aborted transaction must be retried by the application.

### 10.4 Prevention Strategies

#### Strategy 1: Lock Ordering

Always lock resources in the **same order** across all transactions. If every transaction locks product 101 before product 202, deadlocks between them are impossible.

```sql
-- Both transactions should lock in product_id order
BEGIN;
UPDATE products SET stock = stock - 1 WHERE product_id = 101;
UPDATE products SET stock = stock - 1 WHERE product_id = 202;
COMMIT;
```

#### Strategy 2: Lock Timeout

Set a maximum time to wait for a lock:

```sql
SET lock_timeout = '5s';
-- If a lock isn't acquired within 5 seconds, the statement fails
-- instead of waiting indefinitely
```

#### Strategy 3: Use SELECT FOR UPDATE with NOWAIT or SKIP LOCKED

```sql
-- NOWAIT: fail immediately if row is locked
SELECT * FROM products WHERE product_id = 101 FOR UPDATE NOWAIT;
-- ERROR: could not obtain lock on row

-- SKIP LOCKED: skip already-locked rows (useful for work queues)
SELECT * FROM products WHERE stock_quantity > 0
ORDER BY product_id
LIMIT 1
FOR UPDATE SKIP LOCKED;
```

#### Strategy 4: Keep Transactions Short

The shorter the transaction, the less time locks are held, and the smaller the window for deadlocks.

#### Strategy 5: Use Higher Isolation Levels

At SERIALIZABLE level, PostgreSQL may abort transactions before deadlocks form, using its SSI conflict detection.

---

## 11. Key Terms

| Term | Definition |
|------|-----------|
| Transaction | A sequence of operations treated as a single atomic unit |
| ACID | Atomicity, Consistency, Isolation, Durability |
| Atomicity | All operations succeed or none take effect |
| Consistency | Database moves from one valid state to another |
| Isolation | Concurrent transactions don't interfere with each other |
| Durability | Committed data survives system failures |
| COMMIT | Makes transaction's changes permanent |
| ROLLBACK | Undoes all changes in the current transaction |
| SAVEPOINT | Named checkpoint within a transaction for partial rollback |
| Autocommit | Mode where each statement is its own transaction |
| Dirty read | Reading uncommitted data from another transaction |
| Non-repeatable read | Reading same row twice gets different values |
| Phantom read | Re-executing a query gets different set of rows |
| Lost update | Two transactions overwrite each other's changes |
| Isolation level | Setting that controls what concurrency problems are prevented |
| READ COMMITTED | Default in PostgreSQL; sees only committed data per statement |
| REPEATABLE READ | Sees a snapshot from transaction start |
| SERIALIZABLE | Guarantees serial equivalence |
| Row-level lock | Lock on individual rows allowing maximum concurrency |
| SELECT FOR UPDATE | Explicitly locks rows for upcoming modification |
| MVCC | Multi-Version Concurrency Control; multiple row versions coexist |
| Deadlock | Circular wait where transactions block each other permanently |
| WAL | Write-Ahead Log; ensures durability by logging before applying |

---

## 12. Reading

### Required

- PostgreSQL Documentation: [Tutorial — Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html)
- PostgreSQL Documentation: [Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)

### Further Reading

- PostgreSQL Documentation: [Explicit Locking](https://www.postgresql.org/docs/current/explicit-locking.html)
- PostgreSQL Documentation: [MVCC Introduction](https://www.postgresql.org/docs/current/mvcc-intro.html)
- PostgreSQL Documentation: [Routine Vacuuming](https://www.postgresql.org/docs/current/routine-vacuuming.html)
- Kleppmann, M. (2017). *Designing Data-Intensive Applications*, Chapter 7: Transactions.

---

## 13. Summary

This chapter covered the mechanisms that keep databases correct under real-world conditions:

- **Transactions** bundle multiple operations into atomic units — all succeed or all fail.
- **ACID properties** guarantee atomicity, consistency, isolation, and durability.
- **Transaction control** (BEGIN, COMMIT, ROLLBACK, SAVEPOINT) gives you precise control over transaction boundaries.
- **Concurrency problems** (dirty read, non-repeatable read, phantom read, lost update) arise when transactions aren't properly isolated.
- **Isolation levels** let you choose the trade-off between safety and performance, with READ COMMITTED as PostgreSQL's practical default.
- **Locking** (row-level, SELECT FOR UPDATE) prevents conflicting modifications.
- **MVCC** enables PostgreSQL's "readers don't block writers" approach through multiple row versions.
- **Deadlocks** are circular lock waits that PostgreSQL detects and resolves by aborting one transaction.

---

## Coming Next: Week 47 — Indexing and Performance

Your database is correct and handles concurrency — but is it *fast*? Next week you'll learn how indexes work under the hood, when to create them, and how to use EXPLAIN ANALYZE to find and fix slow queries. TrailShop's Black Friday traffic demands performance.
