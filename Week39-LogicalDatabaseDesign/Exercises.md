# Week 39 — Logical Database Design: Exercises

> [!IMPORTANT] How to Complete These Exercises
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 39 Theory material. Refer to the theory sections indicated in brackets when you need help.

---

## Exercise 1: TrailShop Project Task — Build the Schema

**Goal:** Convert the TrailShop ER diagram (from Week 38) into a complete PostgreSQL relational schema.

### Instructions

Write `CREATE TABLE` statements for all five TrailShop tables:

1. `categories`
2. `customers`
3. `products`
4. `orders`
5. `order_items`

### Requirements

For each table, you must:

- Choose appropriate PostgreSQL data types for every column (justify at least 3 choices in writing)
- Define primary keys (surrogate or composite as appropriate)
- Define foreign keys with explicit `ON DELETE` and `ON UPDATE` actions (justify each choice)
- Add `NOT NULL`, `UNIQUE`, `CHECK`, and `DEFAULT` constraints where appropriate
- Create tables in the correct dependency order
- Follow the naming conventions from Theory Section 8

### Deliverables

1. A single `.sql` file with all five `CREATE TABLE` statements (executable in PostgreSQL)
2. A short written document (1–2 pages) containing:
   - Justification for 3 data type choices (e.g., why `NUMERIC(10,2)` for price instead of `REAL`)
   - Justification for each FK action choice (e.g., why CASCADE on `order_items.order_id`)
   - One design decision you made that wasn't specified in the requirements (e.g., whether shipping address is optional)

### Bonus Challenge

After creating the tables, insert sample data:
- At least 5 categories
- At least 8 products (across at least 3 categories)
- At least 3 customers
- At least 4 orders (across at least 2 customers)
- At least 10 order items

Verify that your constraints work by attempting at least 2 invalid inserts and showing the error messages.

> [!NOTE] Your SQL
>
> ```sql
> -- Paste key CREATE TABLE statements or link to your .sql file contents here
>
>
> ```

> [!NOTE] Your Answer
>
> *(Paste written justifications for data types, FK actions, and design decisions here.)*
>
>
>
>

## Exercise 2: Theory Review Questions

Answer each question in 2–4 sentences. Reference the relevant theory section.

1. List the seven phases of the database development lifecycle in order. Which phase is this week's focus? *(Section 1)*

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

2. Explain the transformation rule for mapping a 1:N relationship to the relational model. Why is the foreign key placed on the "many" side? *(Section 3.2)*

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

3. What is a junction table? When is it needed? Give an example not from TrailShop. *(Section 3.3)*

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

4. When mapping a 1:1 relationship, how do you decide which table gets the foreign key? *(Section 3.4)*

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

5. How does the mapping of a weak entity differ from a strong entity? What happens to the primary key? *(Section 3.5)*

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

6. Why should you never use `REAL` or `DOUBLE PRECISION` for monetary values? What should you use instead? *(Section 4.1)*

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

7. What is the difference between `TIMESTAMP` and `TIMESTAMPTZ`? Which should you prefer and why? *(Section 4.3)*
> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>




8. Explain the difference between `CASCADE` and `RESTRICT` as foreign key delete actions. Give a scenario where each is appropriate. *(Section 6)*
> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>




9. What is an insertion anomaly? Give an example and explain how proper schema design prevents it. *(Section 7)*
> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>




10. What is the difference between a surrogate key and a natural key? Give one advantage of each. *(Section 9)*
> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>




11. Why does PostgreSQL fold unquoted identifiers to lowercase? How does `snake_case` naming help? *(Section 8)*

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

12. What does `SET NULL` do as a foreign key action? When would you use it instead of `CASCADE`? *(Section 6)*
> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>




---

## Exercise 3: Transformation Exercise — Hotel Booking System

### Given ER Diagram

A hotel booking system has the following entities and relationships:

**Entities:**

1. **Hotel** — hotel_id (PK), name, city, star_rating, phone
2. **Room** (weak entity, owned by Hotel) — room_number (partial key), room_type, floor, price_per_night, has_balcony
3. **Guest** — guest_id (PK), first_name, last_name, email, phone, passport_number
4. **Booking** — booking_id (PK), check_in_date, check_out_date, total_amount, status
5. **Service** — service_id (PK), name, description, price (e.g., "Room Service", "Spa", "Airport Shuttle")

**Relationships:**

- Hotel (1) → Room (N): A hotel has many rooms. Each room belongs to exactly one hotel. (Identifying relationship — Room is weak.)
- Guest (1) → Booking (N): A guest can make many bookings. Each booking belongs to one guest.
- Booking (M) ↔ Room (N): A booking can include multiple rooms, and a room can appear in many bookings (over time). The junction records the specific dates.
- Booking (M) ↔ Service (N): A booking can use multiple services, and a service can be used by many bookings. The junction records the date used and quantity.

### Task

1. Write `CREATE TABLE` statements for ALL tables (including junction tables).
2. For each table:
   - Choose appropriate data types
   - Define PK, FK, NOT NULL, UNIQUE, CHECK, and DEFAULT constraints
   - Specify ON DELETE and ON UPDATE actions for all FKs
3. Create the tables in the correct dependency order.
4. Explain why Room is a weak entity and how its PK reflects this.

> [!NOTE] Your SQL
>
> ```sql
> -- Write your CREATE TABLE statements here
>
>
> ```

> [!NOTE] Your Answer
>
> *(Explain why Room is a weak entity and how its PK reflects this.)*
>
>
>
>

---

## Exercise 4: Data Type Selection

> [!NOTE] Your Answers
> Fill in the **Your Data Type** and **Justification** columns in the table below.
>

For each column described below, choose the best PostgreSQL data type and write a brief justification (1–2 sentences). Do NOT just pick `VARCHAR` or `TEXT` for everything — think carefully about validation, storage, and query needs.

| # | Column Description | Your Data Type | Justification |
|---|---|---|---|
| 1 | Employee salary (exact, up to €999,999.99) | | |
| 2 | Number of items in stock (never negative, max ~50,000) | | |
| 3 | Whether a user's email is verified | | |
| 4 | Customer's date of birth | | |
| 5 | Product description (variable length, could be several paragraphs) | | |
| 6 | Country code (always exactly 2 letters, like "FI", "US") | | |
| 7 | IP address of a login attempt | | |
| 8 | Order total (exact, up to €9,999,999.99) | | |
| 9 | GPS latitude of a store location | | |
| 10 | A unique identifier for API tokens that must be globally unique across distributed systems | | |
| 11 | Duration of a video in seconds (always a whole number) | | |
| 12 | Timestamp of when a record was last modified (users in multiple time zones) | | |
| 13 | A Finnish phone number like "+358 40 123 4567" | | |
| 14 | A percentage discount (0.00% to 100.00%) | | |
| 15 | A product's color options (e.g., a product comes in "red", "blue", "green") | | |

---

## Exercise 5: Constraint Design

For each business rule below, write the appropriate PostgreSQL constraint. Provide the constraint as it would appear inside a `CREATE TABLE` statement or as an `ALTER TABLE` statement.

### Part A: Single-Column Constraints

1. "A product's weight must be greater than zero (if provided)."

2. "Every customer must have an email address."

3. "Product names must be unique."

4. "An employee's hire date defaults to today if not specified."

5. "Order status can only be one of: 'new', 'confirmed', 'shipped', 'delivered', 'returned'."

> [!NOTE] Your SQL
>
> ```sql
> -- Write constraints 1–5 here
>
>
> ```

### Part B: Multi-Column Constraints

6. "A flight's arrival time must be after its departure time."

7. "In the `enrollments` table, the combination of `student_id` and `course_id` must be unique (a student can only enroll in a course once)."

8. "A discount percentage must be between 0 and 100, inclusive."

> [!NOTE] Your SQL
>
> ```sql
> -- Write constraints 6–8 here
>
>
> ```

### Part C: Foreign Key Constraints with Actions

9. "When a department is deleted, all employees in that department should have their `department_id` set to NULL (they become unassigned)."

10. "When a customer is deleted, prevent the deletion if the customer has any orders."

11. "When an author is deleted, all their blog posts should be deleted automatically."

12. "When a course is deleted, all enrollments for that course should be removed."

> [!NOTE] Your SQL
>
> ```sql
> -- Write constraints 9–12 here
>
>
> ```

---

## Submission Checklist

- [ ] Exercise 1: `.sql` file with all CREATE TABLE statements + written justifications
- [ ] Exercise 2: All 12 theory review answers
- [ ] Exercise 3: Hotel booking schema with all tables and explanations
- [ ] Exercise 4: Data type selections with justifications for all 15 columns
- [ ] Exercise 5: All 12 constraints written in valid PostgreSQL syntax
