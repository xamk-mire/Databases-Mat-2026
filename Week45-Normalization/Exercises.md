# Week 45 — Normalization: Exercises

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

---

## 1. TrailShop Project Task

### Task 1.1: Analyze a Denormalized Table

Consider this denormalized table that TrailShop's old system used:

```
supplier_products(
    supplier_id, supplier_name, supplier_email, supplier_city,
    product_id, product_name, category_id, category_name,
    supply_price, lead_time_days
)
```

Sample data:

| supplier_id | supplier_name | supplier_email | supplier_city | product_id | product_name | category_id | category_name | supply_price | lead_time_days |
|---|---|---|---|---|---|---|---|---|---|
| 1 | NordGear Oy | info@nordgear.fi | Helsinki | 101 | Trail Runner X | 10 | Footwear | 45.00 | 7 |
| 1 | NordGear Oy | info@nordgear.fi | Helsinki | 102 | Summit Boot | 10 | Footwear | 62.00 | 7 |
| 2 | OutdoorPro AB | sales@outdoorpro.se | Stockholm | 201 | Rain Jacket Pro | 30 | Clothing | 55.00 | 14 |
| 2 | OutdoorPro AB | sales@outdoorpro.se | Stockholm | 101 | Trail Runner X | 10 | Footwear | 43.00 | 10 |

**Tasks:**

1. Identify the primary key of this table.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
2. List ALL functional dependencies.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
3. Identify any insertion, update, and deletion anomalies (give specific examples).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
4. Normalize this table to 3NF. Show each step (1NF → 2NF → 3NF) with the resulting tables.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
5. Draw a dependency diagram for the original table showing partial and transitive dependencies.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### Task 1.2: Review Your Existing TrailShop Schema

1. List all functional dependencies in your current TrailShop schema (all tables).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
2. For each table, confirm it is in 3NF by verifying:
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
   - No partial dependencies (check tables with composite keys or surrogate keys)
   - No transitive dependencies
3. Explain why `unit_price` in `order_items` is NOT a normalization violation.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### Task 1.3: Add a Suppliers Table (Optional Extension)

If TrailShop wanted to track suppliers:

1. Design a normalized schema that includes suppliers. Consider:
   - Each product can have multiple suppliers
   - Each supplier offers different prices and lead times
   - Each supplier has contact info (name, email, city)
2. Write the `CREATE TABLE` statements.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
3. Verify your design is in 3NF.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

## 2. Theory Review Questions

Answer in your own words (2–4 sentences each):

1. What is normalization, and what problem does it solve?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>


2. Explain the difference between an insertion anomaly and a deletion anomaly, using an example.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
3. What is a functional dependency? Give an example from a university context (students, courses, professors).

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
4. What is the difference between a partial dependency and a transitive dependency?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>


5. State the "informal rule" for 3NF. Explain what each part means.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
6. Why can partial dependencies only occur in tables with composite primary keys?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>


7. Explain Armstrong's transitivity axiom with a concrete example.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
8. What is BCNF, and how does it differ from 3NF? In what rare case can a table be in 3NF but not BCNF?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
9. Give two legitimate reasons to denormalize a database.
10. What is the closure of an attribute set, and how would you use it to determine if an attribute set is a candidate key?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>


> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

---

## 3. FD Identification Exercise

For each table below, examine the sample data and the business rules, then list all functional dependencies.

### Table A: library_loans

**Business rules**: Each book has one ISBN and one title. Each member has one name. A member can borrow the same book multiple times (on different dates).

| loan_id | member_id | member_name | isbn | book_title | loan_date | return_date |
|---|---|---|---|---|---|---|
| 1 | M01 | Sari Niemi | 978-1-234 | SQL Basics | 2025-01-10 | 2025-01-24 |
| 2 | M02 | Timo Korhonen | 978-1-234 | SQL Basics | 2025-01-15 | 2025-01-29 |
| 3 | M01 | Sari Niemi | 978-2-567 | Data Design | 2025-02-01 | NULL |

### Table B: course_schedule

**Business rules**: Each room is in one building. Each course has one instructor. An instructor can teach multiple courses. A course meets in one room at a specific time slot.

| course_id | course_name | instructor_id | instructor_name | room_id | building | time_slot |
|---|---|---|---|---|---|---|
| CS101 | Intro to DB | I01 | Dr. Mäkelä | R201 | Main Hall | Mon 10-12 |
| CS102 | Algorithms | I02 | Dr. Järvinen | R105 | Tech Wing | Tue 14-16 |
| CS101 | Intro to DB | I01 | Dr. Mäkelä | R201 | Main Hall | Wed 10-12 |

### Table C: employee_projects

**Business rules**: Each employee belongs to one department. Each department has one manager. An employee can work on multiple projects with different hours.

| emp_id | emp_name | dept_id | dept_name | manager_id | project_id | hours |
|---|---|---|---|---|---|---|
| E1 | Liisa | D10 | Engineering | E5 | P1 | 20 |
| E1 | Liisa | D10 | Engineering | E5 | P2 | 15 |
| E2 | Pekka | D20 | Marketing | E6 | P1 | 30 |

---

## 4. Normalization Exercise

Normalize each table to 3NF. Show your work for each step.

### Table 1: event_registrations

| event_id | event_name | event_date | venue_id | venue_name | venue_city | attendee_id | attendee_name | ticket_type | ticket_price |
|---|---|---|---|---|---|---|---|---|---|
| E1 | Trail Run 2025 | 2025-06-15 | V1 | City Park Arena | Helsinki | A1 | Matti Aho | VIP | 75.00 |
| E1 | Trail Run 2025 | 2025-06-15 | V1 | City Park Arena | Helsinki | A2 | Sara Lund | Standard | 35.00 |
| E2 | Mountain Fest | 2025-07-20 | V2 | Summit Center | Tampere | A1 | Matti Aho | Standard | 40.00 |

**Business rules:**
- Each event has one name, date, and venue.
- Each venue has one name and city.
- Each attendee has one name.
- Ticket price depends on the event and ticket type combination.

**Steps:**
1. Identify the PK
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
2. List all FDs
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
3. Check 1NF, 2NF, 3NF
4. Decompose as needed
5. Write final CREATE TABLE statements
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### Table 2: store_inventory

| store_id | store_name | store_city | product_id | product_name | category | quantity | reorder_level | supplier_name |
|---|---|---|---|---|---|---|---|---|
| S1 | TrailShop Helsinki | Helsinki | 101 | Trail Runner X | Footwear | 50 | 10 | NordGear Oy |
| S1 | TrailShop Helsinki | Helsinki | 201 | Rain Jacket Pro | Clothing | 20 | 5 | OutdoorPro AB |
| S2 | TrailShop Tampere | Tampere | 101 | Trail Runner X | Footwear | 30 | 8 | NordGear Oy |

**Business rules:**
- Each store has one name and city.
- Each product has one name, category, and supplier.
- Quantity and reorder_level depend on the specific store-product combination.

**Steps:** Same as above.

---

## 5. True/False

For each statement, write TRUE or FALSE and briefly explain why.

1. A table with a single-column primary key can violate 2NF.
2. If a table is in 3NF, it is automatically in 2NF.
3. Normalization always improves query performance.
4. A functional dependency X → Y means that Y uniquely identifies X.
5. Storing a customer's current address in the orders table is always a normalization violation.
6. BCNF is stricter than 3NF — every table in BCNF is also in 3NF.
7. Armstrong's transitivity axiom states: if X → Y and Y → Z, then X → Z.
8. Denormalization should be done before normalization to save time.
