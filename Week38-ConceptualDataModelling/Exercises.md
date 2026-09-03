# Week 38 — Conceptual Data Modelling: Exercises

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 38 Theory material. Refer to the theory sections indicated in brackets when you need help.

---

## Exercise 1: TrailShop Project Task — Create the ER Diagram

**Goal:** Create a complete Entity-Relationship diagram for the TrailShop database using crow's foot notation.

### Instructions

Using the entity descriptions from Theory Section 12, create an ER diagram that includes:

1. **All five entities**: Category, Product, Customer, Order, OrderItem
2. **All attributes** for each entity (as listed in Section 12.1)
3. **Primary keys** clearly marked (underline or "PK" label)
4. **Foreign keys** clearly marked (dashed underline or "FK" label)
5. **Relationships** between entities with:
   - Relationship name (verb)
   - Crow's foot notation showing cardinality and participation
6. **Identify weak entities** — mark OrderItem as a weak entity

### Requirements

- Use crow's foot notation (see Theory Section 9)
- You may use any tool: draw.io, Lucidchart, ERDPlus, dbdiagram.io, or even pen and paper (photograph and submit)
- The diagram must be readable — avoid crossing lines where possible
- Include a brief legend explaining your notation if using pen and paper

### Deliverables

- The ER diagram (image or link to online tool)
- A short written paragraph (3–5 sentences) explaining one design decision you made — for example, why OrderItem is a weak entity, or why `unit_price` is stored in OrderItem instead of being looked up from Product.

---

## Exercise 2: Theory Review Questions

Answer each question in 2–4 sentences. Reference the relevant theory section.

1. Why should you create a conceptual data model before writing SQL? Give two specific reasons. *(Section 1)*

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

2. What is the difference between the conceptual level and the logical level of a data model? *(Section 2)*

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

3. Explain logical data independence with an example. *(Section 3)*

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

4. Explain physical data independence with an example. *(Section 3)*

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

5. What is the difference between a strong entity and a weak entity? Give one example of each (not from TrailShop). *(Section 5)*

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

6. What is a composite attribute? How does it differ from a multivalued attribute? Give an example of each. *(Section 6)*

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

7. What is a derived attribute? Why is it usually not stored in the database? *(Section 6)*

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

8. Explain the difference between a binary relationship and a unary (recursive) relationship. Give an example of each. *(Section 7)*
> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>




9. What is the difference between an identifying relationship and a non-identifying relationship? How does this affect the child table's primary key? *(Section 7)*
> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>




10. In crow's foot notation, what does the following endpoint mean: a circle followed by a crow's foot (fork)? *(Section 9)*

11. Why can't a many-to-many (M:N) relationship be directly implemented in a relational database? What is the solution? *(Section 10)*

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

12. A business rule states: "Every employee must belong to exactly one department, and every department must have at least one employee." Express this using min-max notation for both sides. *(Section 8)*

---

## Exercise 3: ER Diagram Reading Exercise

### Diagram A: Library System

Study the following ER description and answer the questions below.

```
┌──────────┐                        ┌──────────┐
│  AUTHOR  │──||──────O<────────────│   BOOK   │
└──────────┘                        └─────┬────┘
                                          │
                                    ||    │
                                          │
                                    O<    │
                                          │
                                   ┌──────┴─────┐
                                   │    LOAN     │
                                   └──────┬──────┘
                                          │
                                    ||    │
                                          │
                                    O<    │
                                          │
                                   ┌──────┴──────┐
                                   │   MEMBER    │
                                   └─────────────┘
```

Relationships (in crow's foot):
- Author `──||──────O<──` Book
- Book `──||──────O<──` Loan
- Member `──||──────O<──` Loan

**Questions:**

a) Can an author exist without having written any books? Explain using the notation.
> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

b) Can a book exist without being loaned? Explain using the notation.
> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

c) What type of entity is Loan in this diagram? Is it a junction/associative entity? Why?


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

d) What is the cardinality of the Author-Book relationship? Is this realistic? What might be a more accurate model?


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

e) What attributes would you add to the Loan entity?


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### Diagram B: School System

```
STUDENT ──O|──────O<── ENROLLMENT ──>|──||── COURSE
                                        │
                                    ||  │
                                        │
                                    O<  │
                                        │
                                   TEACHER
```

Relationships:
- Student `──O|──────O<──` Enrollment (a student may have zero or many enrollments)
- Enrollment `──||──────||──` Course (each enrollment is for exactly one course)
- Teacher `──||──────O<──` Course (each course has zero or many sections, each taught by exactly one teacher)

**Questions:**

a) Can a student exist without being enrolled in any course?


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

b) Can a course exist without having any enrolled students?


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

c) What is the cardinality between Student and Course (through Enrollment)?


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

d) Can a teacher exist without teaching any courses?


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

e) Is the Teacher-Course relationship 1:1 or 1:N? What does this imply about team teaching?


> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

## Exercise 4: ER Diagram Creation — Gym/Fitness Center

### Scenario

FitZone is a local gym and fitness center. They need a database to manage their operations. Here are the business rules:

1. The gym has **members**. Each member has an ID, first name, last name, email, phone, date of birth, and membership start date.

2. The gym offers **membership plans** (e.g., "Basic", "Premium", "Student"). Each plan has a plan ID, name, monthly price, and description. Each member subscribes to exactly one plan. A plan can have many members.

3. The gym has **trainers** (employees who lead classes). Each trainer has an ID, first name, last name, specialization (e.g., "Yoga", "CrossFit"), and hire date.

4. The gym offers **classes** (e.g., "Morning Yoga", "HIIT Blast"). Each class has an ID, name, day of the week, start time, end time, and maximum capacity. Each class is led by exactly one trainer, but a trainer can lead many classes.

5. Members can **register** for classes. A member can register for many classes, and a class can have many registered members. The registration records the registration date.

6. The gym has **equipment** (treadmills, dumbbells, etc.). Each piece of equipment has an ID, name, type, purchase date, and status ("working", "maintenance", "retired").

7. When equipment breaks, a **maintenance request** is created. Each request has an ID, request date, description of the problem, status ("open", "in progress", "closed"), and resolution date. Each request is for exactly one piece of equipment. One piece of equipment can have many maintenance requests over time.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### Task

1. Identify all entities and their attributes (including key attributes).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
2. Identify all relationships with their cardinality and participation constraints.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
3. Draw a complete ER diagram using crow's foot notation.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
4. Identify any entity that might be considered a weak entity or a junction/associative entity. Justify your answer.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
5. Are there any M:N relationships? If so, what junction entity resolves them?



> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
---

## Exercise 5: Find and Correct the Errors

The following ER diagram description contains **four errors**. Find each error, explain why it's wrong, and provide the correction.

### Scenario: Online Bookstore

**Entities and attributes:**

1. **Books**
   - book_id (PK)
   - title
   - author_name
   - price
   - genres (stores "Fiction, Mystery, Thriller" as a comma-separated string)

2. **Customer**
   - customer_id (PK)
   - full_name
   - address

3. **Purchase**
   - purchase_id (PK)
   - purchase_date
   - total_amount

**Relationships:**
- Books to Customer: M:N (implemented directly — no junction table)
- Customer to Purchase: 1:N (one customer, many purchases)
- Books to Purchase: no relationship defined

### Your Task

Find the four errors in this design and for each one:

a) State what the error is
> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
b) Explain why it's a problem (reference the relevant theory section)
> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
c) Describe how to fix it
> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

**Hints:** Think about multivalued attributes, M:N relationships, entity naming conventions, and missing relationships.

---

## Submission Checklist

- [ ] Exercise 1: ER diagram + design decision paragraph
- [ ] Exercise 2: All 12 theory review answers
- [ ] Exercise 3: All questions answered for both Diagram A and Diagram B
- [ ] Exercise 4: Entity list, relationship list, ER diagram, and justifications
- [ ] Exercise 5: Four errors identified with explanations and corrections
