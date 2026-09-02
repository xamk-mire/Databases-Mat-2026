# Week 46 — Transactions and Concurrency: Exercises

> [!IMPORTANT] How to Complete These Exercises
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

---

## 1. TrailShop Project Task

### Task 1.1: Write an Order Transaction

Write a complete transaction that processes a new order in TrailShop:

1. Insert a new order for an existing customer.
2. Insert 2–3 order items.
3. Decrease `stock_quantity` for each ordered product.
4. Include a check that stock doesn't go negative (if it would, ROLLBACK).
5. Use a SAVEPOINT before the stock update step.
> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

Test the transaction by:
- Running it successfully with sufficient stock.
- Running it where stock is insufficient — verify the ROLLBACK works.

### Task 1.2: Observe READ COMMITTED Behavior

Open two psql sessions (or two query windows). Demonstrate a non-repeatable read:

1. Session A: `BEGIN;` then `SELECT stock_quantity FROM products WHERE product_id = 101;`
2. Session B: `BEGIN;` then `UPDATE products SET stock_quantity = stock_quantity - 5 WHERE product_id = 101;` then `COMMIT;`
3. Session A: `SELECT stock_quantity FROM products WHERE product_id = 101;` (observe different value)
4. Session A: `COMMIT;`

Document what you observe and explain why it happens at READ COMMITTED.

### Task 1.3: Observe Locking

Open two sessions and demonstrate row-level locking:

1. Session A: `BEGIN;` then `SELECT * FROM products WHERE product_id = 101 FOR UPDATE;`
2. Session B: Try `UPDATE products SET price = 99.99 WHERE product_id = 101;` — observe it waits.
3. Session A: `COMMIT;` — observe Session B proceeds.

Document the behavior.

### Task 1.4: Experiment with Isolation Levels

1. Repeat Task 1.2, but set Session A to `REPEATABLE READ`. Does the non-repeatable read still occur?


> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
2. In a REPEATABLE READ transaction, try to UPDATE a row that another transaction has already committed a change to. What error do you get?


> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
3. Document the differences between READ COMMITTED and REPEATABLE READ behavior.

---

## 2. Theory Review Questions

Answer in your own words (2–4 sentences each):

1. What is a transaction, and why is it important for data integrity?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


2. Explain the Atomicity property using a real-world example (not from the lecture notes).

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
3. What does Isolation guarantee? Why is it needed in a multi-user system?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


4. What is the difference between COMMIT and ROLLBACK?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


5. Explain what a SAVEPOINT is and give a scenario where it's useful.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
6. What is a dirty read? Why does PostgreSQL never allow it?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


7. Describe the lost update problem. How can SELECT FOR UPDATE prevent it?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


8. What is the default isolation level in PostgreSQL? What concurrency problems does it still allow?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


9. Explain MVCC in one paragraph: what is it, and what advantage does it provide?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


10. What is a deadlock, and how does PostgreSQL handle it when one is detected?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>



---

## 3. Scenario Analysis

For each scenario, identify: (a) which concurrency problem occurs, and (b) at which isolation level(s) it can happen.

### Scenario A

Transaction 1 reads customer 5's order count (3 orders). Transaction 2 inserts a new order for customer 5 and commits. Transaction 1 reads customer 5's order count again and gets 4.

### Scenario B

Transaction 1 reads product 101's price (€89.99). Transaction 2 updates product 101's price to €79.99 but then rolls back. If Transaction 1 had somehow seen €79.99, what problem would that be?

### Scenario C

Two warehouse workers simultaneously check stock for product 101 (both see 10). Worker A ships 3 units and sets stock to 7. Worker B ships 4 units and sets stock to 6. The correct stock should be 3.

---

## 4. Transaction Writing

Write complete SQL transactions for the following TrailShop scenarios. Include BEGIN, appropriate operations, error handling (SAVEPOINT where useful), and COMMIT.

### Transaction 1: Bulk Price Update

Update the price of all products in the "Footwear" category by +10%. If any product's new price would exceed €200, rollback the entire update.

### Transaction 2: Customer Account Deletion

Delete a customer and all their associated data:
1. Delete their order_items (for all their orders)

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
2. Delete their orders

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
3. Delete the customer record

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

Use SAVEPOINTs between each step in case of errors.

### Transaction 3: Inventory Restock

Restock 3 products (add specific quantities to stock). Use SELECT FOR UPDATE to lock the rows first, preventing lost updates. Include a check that the new stock doesn't exceed a maximum (e.g., 200).

---

## 5. True/False

For each statement, write TRUE or FALSE and briefly explain why.

1. In PostgreSQL, a single SQL statement outside of an explicit BEGIN/COMMIT is not a transaction.
2. ROLLBACK undoes all changes made since the last SAVEPOINT.
3. At READ COMMITTED isolation level, a transaction can never see another transaction's uncommitted changes.
4. REPEATABLE READ in PostgreSQL prevents phantom reads.
5. SELECT FOR UPDATE blocks other SELECT statements on the same rows.
6. If a deadlock occurs, PostgreSQL will wait indefinitely for it to resolve.
7. MVCC means PostgreSQL keeps multiple versions of each row, allowing readers and writers to work concurrently without blocking.
8. Once a transaction is committed, its changes can be undone with ROLLBACK.
