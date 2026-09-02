# Week 40 — Exercises: SQL Fundamentals

> [!IMPORTANT] How to Complete These Exercises
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

## Exercise 1: TrailShop Project Task

This week you'll build the TrailShop database from scratch and practice manipulating data.

### Task 1.1: Create the Database

1. Open your PostgreSQL terminal (psql) or pgAdmin
2. Create a new database called `trailshop`
3. Connect to it

### Task 1.2: Create All Tables

Write and execute the CREATE TABLE statements for all five TrailShop tables in the correct order:
- categories
- customers
- products
- orders
- order_items

**Requirements:**
- Use appropriate data types for each column
- Include all constraints from the theory (NOT NULL, UNIQUE, CHECK, FOREIGN KEY, DEFAULT)
- Use SERIAL for primary keys
- Ensure foreign keys reference the correct parent tables

**Verify** by running `\dt` in psql to list all tables.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```


### Task 1.3: Insert Sample Data

Insert the following data:

**Categories** (at least 5):
- Footwear, Backpacks, Tents, Clothing, Accessories

**Customers** (at least 5):
- Use Finnish-sounding names with realistic email addresses

**Products** (at least 10):
- At least 2 products per category
- Prices ranging from €20 to €500
- Various stock levels

**Orders** (at least 5):
- Different customers, different statuses

**Order Items** (at least 10):
- Multiple items in some orders, single items in others

**Verify** each insert with `SELECT * FROM table_name;`

### Task 1.4: Practice UPDATE

Perform the following updates and verify each one:

1. Increase the price of all products in the Footwear category by 10%
2. Change customer #3's email to a new address
3. Update the status of order #2 from 'shipped' to 'delivered'
4. Set the stock of 'HydroFlask 1L' to 85
5. Add a description to any product that currently has NULL in description

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```


### Task 1.5: Practice DELETE

1. Delete the most recently created order (and observe what happens to its order_items if you used CASCADE)
2. Try to delete a category that has products — what error do you get?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

3. Delete a customer who has no orders

### Task 1.6: Practice ALTER TABLE

1. Add a column `phone VARCHAR(20)` to the customers table
2. Add a column `weight_grams INTEGER` to the products table
3. Add a CHECK constraint to ensure `weight_grams > 0` (allow NULL though — not all products have weight recorded yet)
4. Rename the `stock` column in products to `quantity_in_stock`

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```


---

## Exercise 2: Theory Review Questions

Answer the following questions in your own words using the answer fields below:

1. What does SQL stand for, and why was the language designed to look like English?



> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
2. Explain the difference between DDL and DML. Give two example commands for each.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
3. What is the difference between DCL and TCL? When would you use each?



> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
4. Why must you create tables in a specific order? What determines that order?



> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
5. What is the difference between a column-level constraint and a table-level constraint? When *must* you use a table-level constraint?



> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
6. Explain the difference between `DELETE FROM products;` and `TRUNCATE TABLE products;`. When would you prefer each?



> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
7. What does `ON DELETE CASCADE` do on a foreign key? Give a real-world scenario where it's appropriate and one where it would be dangerous.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
8. Why should you store `unit_price` in the `order_items` table instead of just looking it up from the `products` table?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
9. What is the difference between SERIAL and GENERATED ALWAYS AS IDENTITY? Which would you use in a new project and why?


> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>




10. Explain why `UPDATE products SET price = 9.99;` is dangerous. What steps should you take before running any UPDATE statement?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
---

## Exercise 3: SQL Writing Exercises

Write the SQL statements for each task in the **Your SQL** fields below. Verify by running them when ready.

### 3.1 CREATE TABLE

Write a CREATE TABLE statement for a `suppliers` table with the following columns:
- supplier_id (auto-incrementing primary key)
- company_name (required, max 200 characters, must be unique)
- contact_name (max 150 characters)
- email (max 255 characters, required, unique)
- phone (max 20 characters)
- country (max 100 characters, required, default 'Finland')

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.2 CREATE TABLE with Foreign Key

Write a CREATE TABLE statement for a `product_reviews` table:
- review_id (auto-incrementing primary key)
- product_id (required, references products)
- customer_id (required, references customers)
- rating (required integer, must be between 1 and 5 inclusive)
- review_text (optional, unlimited length)
- created_at (required, defaults to current timestamp)

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.3 INSERT — Single Row

Write an INSERT statement to add a new category called 'Electronics' with description 'GPS devices, solar chargers, and tech gear'.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.4 INSERT — Multiple Rows

Write a single INSERT statement that adds three new customers:
- Eero Lahtinen, eero.l@email.com
- Maria Salminen, maria.s@email.com
- Petri Kallio, petri.k@email.com

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.5 INSERT with RETURNING

Write an INSERT statement that adds a new product called 'NorthStar GPS' priced at €229.99 with stock of 12 in category 'Electronics' (assume category_id = 6). Return the product_id and created_at.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.6 UPDATE — Simple

Write an UPDATE statement that changes the email of the customer with customer_id = 2 to 'mikko.korhonen@newmail.com'.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.7 UPDATE — Expression

Write an UPDATE statement that reduces the stock of all products by 1 where the stock is currently greater than 0.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.8 UPDATE — Multiple Columns

Write an UPDATE statement that changes order #3 to status 'cancelled' and sets a (hypothetical) cancelled_at timestamp to the current time. (Assume you've already added a cancelled_at column.)

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.9 DELETE — With Condition

Write a DELETE statement that removes all orders with status 'cancelled'.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.10 ALTER TABLE

Write the ALTER TABLE statements to:
a) Add a `discount_percent NUMERIC(5,2) DEFAULT 0 CHECK (discount_percent >= 0 AND discount_percent <= 100)` column to products
b) Drop the `description` column from categories
c) Add a composite unique constraint on (customer_id, product_id) in the product_reviews table (preventing a customer from reviewing the same product twice)

> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

---

## Exercise 4: Error Diagnosis

Each of the following SQL statements contains one or more errors. Identify the error(s) and write the corrected version.

### 4.1

```sql
CREATE TABLE warehouses
    warehouse_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    city VARCHAR(100
);
```

> [!NOTE] Error(s) Identified
>
> *(Describe what is wrong.)*
>
>
>
>
> [!NOTE] Corrected SQL
>
> ```sql
> -- Write the corrected statement here
>
>
> ```


### 4.2

```sql
INSERT INTO products (name, price, stock, category_id)
VALUES ("Alpine Sleeping Bag", 89.99, 20, 2);
```

> [!NOTE] Error(s) Identified
>
> *(Describe what is wrong.)*
>
>
>
>
> [!NOTE] Corrected SQL
>
> ```sql
> -- Write the corrected statement here
>
>
> ```


### 4.3

```sql
CREATE TABLE shipments (
    shipment_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id)
    shipped_date DATE NOT NULL,
    carrier VARCHAR(100)
);
```

> [!NOTE] Error(s) Identified
>
> *(Describe what is wrong.)*
>
>
>
>
> [!NOTE] Corrected SQL
>
> ```sql
> -- Write the corrected statement here
>
>
> ```


### 4.4

```sql
UPDATE products
SET price = price * 0.9
SET stock = stock + 10
WHERE category_id = 3;
```

> [!NOTE] Error(s) Identified
>
> *(Describe what is wrong.)*
>
>
>
>
> [!NOTE] Corrected SQL
>
> ```sql
> -- Write the corrected statement here
>
>
> ```


### 4.5

```sql
CREATE TABLE wishlists (
    wishlist_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
    product_id INTEGER NOT NULL REFERENCES products(product_id),
    added_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (customer_id, product_id)
);
```

> [!NOTE] Error(s) Identified
>
> *(Describe what is wrong.)*
>
>
>
>
> [!NOTE] Corrected SQL
>
> ```sql
> -- Write the corrected statement here
>
>
> ```


---

## Submission Checklist

- [ ] All 5 TrailShop tables created successfully
- [ ] Sample data inserted (at least 5 categories, 5 customers, 10 products, 5 orders, 10 order items)
- [ ] UPDATE exercises completed and verified
- [ ] DELETE exercises completed and verified
- [ ] ALTER TABLE exercises completed and verified
- [ ] Theory review questions answered
- [ ] SQL writing exercises completed
- [ ] Error diagnosis completed with corrections
- [ ] All inline answer fields completed
