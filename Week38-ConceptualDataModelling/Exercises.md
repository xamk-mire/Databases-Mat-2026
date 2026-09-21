# Week 38 — Conceptual Data Modelling: Exercises

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 38 Theory material. Refer to the theory sections indicated in brackets when you need help.

---

## Exercise 1: TrailShop Project Task — Create the ER Diagram

**Goal:** Create a complete Entity-Relationship diagram for the TrailShop database using crow's foot notation.

> **From Week 37:** Last week each product had a single `category_id` (Category 1:N Product). That cannot store a product in two categories. This week's diagram must **not** put `category_id` on Product. Use **ProductCategory** as the junction that resolves Category M:N Product (see Theory Section 1.4).

### Instructions

Using the entity descriptions from Theory Section 12, create an ER diagram that includes:

1. **All six entities**: Category, Product, ProductCategory, Customer, Order, OrderItem
2. **All attributes** for each entity (as listed in Section 12.1)
3. **Primary keys** clearly marked (underline or "PK" label)
4. **Foreign keys** clearly marked (dashed underline or "FK" label)
5. **Relationships** between entities with:
   - Relationship name (verb)
   - Crow's foot notation showing cardinality and participation
6. **Identify weak / junction entities** — mark OrderItem as a weak entity, and mark ProductCategory as the junction that resolves Category M:N Product. Do **not** draw a direct M:N line between Category and Product.

### Requirements

- Use crow's foot notation (see Theory Section 9)
- You may use any tool: draw.io, Lucidchart, ERDPlus, dbdiagram.io, or even pen and paper (photograph and submit)
- The diagram must be readable — avoid crossing lines where possible
- Include a brief legend explaining your notation if using pen and paper

### Deliverables

- The ER diagram (image or link to online tool)
- A short written paragraph (3–5 sentences) that **must** explain why Week 37's 1:N `products.category_id` is being replaced by ProductCategory. You may also discuss another design decision (for example why OrderItem is a weak entity, or why `unit_price` is stored in OrderItem).

> [!NOTE]
> ***Your Answer***
>
> **Diagram Representation:**
`[Customer] 1 ──< Places >── O< [Order] 1 ──< Contains >── O< [OrderItem] >O ──< Belongs To >── 1 [Product] >O ──< Classified By >── O< [ProductCategory] >O ──< Belongs To >── 1 [Category]`

**Design Decision Explanation:**
In Week 37, storing `category_id` directly in the `Product` table limited each product to belonging to exactly one category. Replacing this with the `ProductCategory` junction entity resolves the Many-to-Many (M:N) relationship between `Product` and `Category`, allowing a product to belong to multiple categories and a category to contain multiple products. Furthermore, `OrderItem` is a weak entity because it relies on `Order` for its primary key identification, and storing `unit_price` in `OrderItem` is critical to preserve historical pricing at the time of purchase.
>
>
>
>

---

## Exercise 2: Theory Review Questions

Answer each question in 2–4 sentences. Reference the relevant theory section. Question 11b is extra: it connects last week's 1:N category FK to this week's junction.

1. Why should you create a conceptual data model before writing SQL? Give two specific reasons. *(Section 1)*

> [!NOTE]
> ***Your Answer***
>
> *(First, it helps you understand the business requirements clearly without worrying about database-specific syntax. Second, it lets you spot
> design problems — such as missing entities or incorrect relationships — early, before you spend time building tables that would need to be
> rewritten later.)*
>
>
>
>

2. What is the difference between the conceptual level and the logical level of a data model? *(Section 2)*

> [!NOTE]
> ***Your Answer***
>
> *The conceptual level describes what data exists and how it relates to business rules — independent of any database system. The logical level
> translates those ideas into database structures — such as tables, columns, and keys — but still without specifying exactly which software or
> file formats will be used.)*
>
>
>
>

3. Explain logical data independence with an example. *(Section 3)*

> [!NOTE]
> ***Your Answer***
>
> *(Logical data independence means you can change the database structure — such as adding a new column or splitting a table — without breaking
> existing application programs. )*
>
>
>
>

4. Explain physical data independence with an example. *(Section 3)*

> [!NOTE]
> ***Your Answer***
>
> *(WPhysical data independence means you can change how data is stored on disk — such as indexing, file compression, or storage location —
> without changing the SQL or table definitions.)*
>
>
>
>

5. What is the difference between a strong entity and a weak entity? Give one example of each (not from TrailShop). *(Section 5)*

> [!NOTE]
> ***Your Answer***
>
> *(A strong entity has its own primary key and can exist independently — e.g., Student(student_id, name). A weak entity does not have a full
> primary key of its own and depends on another entity — e.g., OrderLine cannot exist without an Order; it uses the parent's key as part of its
> own.)*
>
>
>
>

6. What is a composite attribute? How does it differ from a multivalued attribute? Give an example of each. *(Section 6)*

> [!NOTE]
> ***Your Answer***
>
> *(A composite attribute can be broken into smaller meaningful parts but is stored as one value — e.g., full_address .
> A multivalued attribute holds multiple separate values for one record — e.g., a person's multiple phone numbers.)*
>
>
>
>

7. What is a derived attribute? Why is it usually not stored in the database? *(Section 6)*

> [!NOTE]
> ***Your Answer***
>
> *(A derived attribute's value is calculated from other data instead of being stored.It is usually not stored because the value changes over time and storing it would create redundant, easily outdated data. You recalculate it when needed)*
>
>
>
>

8. Explain the difference between a binary relationship and a unary (recursive) relationship. Give an example of each. *(Section 7)*
> [!NOTE]
> ***Your Answer***
>
> *(A binary relationship connects two different entity types — e.g., Customer places Order. A unary (recursive) relationship connects an entity
>  to itself — e.g., an Employee manages another Employee.)*
>




9. What is the difference between an identifying relationship and a non-identifying relationship? How does this affect the child table's primary key? *(Section 7)*
> [!NOTE]
> ***Your Answer***
>
> *(In an identifying relationship, the parent's primary key becomes part of the child's primary key — the child cannot exist without the parent.
> In a non-identifying relationship, the parent's key is just a regular foreign key in the child — the child can exist independently.)*
>




10. In crow's foot notation, what does the following endpoint mean: a circle followed by a crow's foot (fork)? *(Section 9)*
Answer (A circle means zero participation , and a crow's foot means many. Together they mean zero or more — the entity on that side can be related to zero, one, or many instances of the other entity.)
11. Why can't a many-to-many (M:N) relationship be directly implemented in a relational database? What is the solution? *(Section 10)*

> [!NOTE]
> ***Your Answer***
>
> *(A many-to-many relationship cannot be stored directly in tables because there is no single place to put the foreign key without creating
> duplicate or ambiguous references. The solution is to create a junction/associative table in between — with foreign keys to both sides — which
>  turns one M:N into two 1:N relationships.)*
>
>
>
>

11b. Last week TrailShop used `products.category_id` so each product belonged to exactly one category. Why is that insufficient, and what ER construct replaces it? *(Section 1.4)*

> [!NOTE]
> ***Your Answer***
>
> *(A single category_id allows only one category per product, but in reality a product can belong to multiple categories — for example, a tent
> could be listed under "Camping", "Shelter", and "New Arrivals". This limitation is replaced by a junction entity ProductCategory that connects
>  Product and Category, enabling any number of categories per product.)*
>
>
>
>

12. A business rule states: "Every employee must belong to exactly one department, and every department must have at least one employee." Express this using min-max notation for both sides. *(Section 8)*

> [!NOTE]
> ***Your Answer***
>
> *(Employee → Department: (1, 1) — exactly one department
> Department → Employee: (1, N) — at least one employee, many allowed)*
>
>
>
>

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
> *(Yes. The symbol O< near Book means zero or more — an author can have zero books. The || near Author means a book must have exactly one
> author)*
>
>
>
>

b) Can a book exist without being loaned? Explain using the notation.
> [!NOTE]
> ***Your Answer***
>
> *(Yes. O< near Loan means zero or more — a book may never be loaned. Loans always relate to exactly one book.)*
>
>
>
>

c) What type of entity is Loan in this diagram? Is it a junction/associative entity? Why?


> [!NOTE]
> ***Your Answer***
>
> *(Loan is an associative/junction entity — it connects Book and Member and may carry its own attributes (loan date, due date). It resolves the
>  many-to-many relationship between Member and Book.)*
>
>
>
>

d) What is the cardinality of the Author-Book relationship? Is this realistic? What might be a more accurate model?


> [!NOTE]
> ***Your Answer***
>
> *(The diagram shows one author writes many books, but a book has exactly one author. This is not realistic — books often have multiple authors.
> A more accurate model would use a junction table BookAuthor to allow many authors per book..)*
>
>
>
>

e) What attributes would you add to the Loan entity?


> [!NOTE]
> ***Your Answer***
>
> *(Loan ID, borrow date, due date, return date, loan status.)*
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
> *(Yes. O| means zero or one — a student may have zero enrollments.)*
>
>
>
>

b) Can a course exist without having any enrolled students?


> [!NOTE]
> ***Your Answer***
>
> *(Yes. The crow's foot near Enrollment means zero or more — a course can be created before anyone signs up.)*
>
>
>
>

c) What is the cardinality between Student and Course (through Enrollment)?


> [!NOTE]
> ***Your Answer***
>
> *(Many-to-many (M:N) — one student takes many courses; one course has many students. The Enrollment junction resolves this.)*
>
>
>
>

d) Can a teacher exist without teaching any courses?


> [!NOTE]
> ***Your Answer***
>
> *(Yes. O< means zero or more — a teacher may be hired but not yet assigned to any class.)*
>
>
>
>

e) Is the Teacher-Course relationship 1:1 or 1:N? What does this imply about team teaching?


> [!NOTE]
> ***Your Answer***
>
> *(One-to-many (1:N) — one teacher teaches many courses, but each course has exactly one teacher. This means team teaching is not supported in
>  this model; a course cannot have multiple instructors.)*
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

### Task

1. Identify all entities and their attributes (including key attributes).

> [!NOTE]
> ***Your Answer***
>
> *(Member — member_id (PK), first_name, last_name, email, phone, dob, start_date
> MembershipPlan — plan_id (PK), plan_name, monthly_price, description
> Trainer — trainer_id (PK), first_name, last_name, specialization, hire_date
> Class — class_id (PK), class_name, day_of_week, start_time, end_time, max_capacity
> Registration — member_id (FK), class_id (FK), registration_date
> Equipment — equip_id (PK), equip_name, equip_type, purchase_date, status
> MaintenanceRequest — request_id (PK), request_date, problem_desc, status, resolution_date.)*
>
>
>
>

2. Identify all relationships with their cardinality and participation constraints.

> [!NOTE]
> ***Your Answer***
>
> 1. **MembershipPlan to Member:** 1:N (One plan can have many members; each member subscribes to exactly one plan.)
> 2. **Trainer to Class:** 1:N (One trainer can lead many classes; each class is led by exactly one trainer.)
> 3. **Member to Class:** M:N (Resolved via Registration junction table — a member can register for many classes, and a class can have many registered members.)
> 4. **Equipment to MaintenanceRequest:** 1:N (One piece of equipment can have many maintenance requests; each request is for exactly one piece of equipment.)
>
>
>
>

3. Draw a complete ER diagram using crow's foot notation.

> [!NOTE]
> ***Your Answer***
>
> **Diagram Representation:**
`[MembershipPlan] 1 ──< Has >── O< [Member] 1 ──< Makes >── O< [Registration] >O ──< Belongs To >── 1 [Class] >O ──< Led By >── 1 [Trainer]`

`[Equipment] 1 ──< Has >── O< [MaintenanceRequest]`
>
>
>
>

4. Identify any entity that might be considered a weak entity or a junction/associative entity. Justify your answer.

> [!NOTE]
> ***Your Answer***
>
> *(Registration is a junction/associative entity — it connects Member and Class and carries the registration date.
> MaintenanceRequest could be considered a dependent entity — it cannot exist without Equipment, but it has its own independent key so it is not
>  strictly weak.)*
>
>
>
>

5. Are there any M:N relationships? If so, what junction entity resolves them?

> [!NOTE]
> ***Your Answer***
>
> *(Member ↔ Class is M:N — resolved by Registration junction table.*
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
## Error 1
a) State what the error is
> [!NOTE]
> ***Your Answer***
>
> *(genres stores multiple values in one field as "Fiction, Mystery, Thriller".)*
>
>
>
>

b) Explain why it's a problem (reference the relevant theory section)

> [!NOTE]
> ***Your Answer***
>
> *(This violates atomicity — each field should contain only one value. Searching by genre becomes difficult and unreliable..)*
>
>
>
>

c) Describe how to fix it

> [!NOTE]
> ***Your Answer***
>
> *(Create a separate 'Genre' table and a junction table 'BlookGenre'.)*
>
>
>## Error 2
a) State what the error is
> [!NOTE]
> ***Your Answer***
>
> *(Books ↔ Customer is defined as a direct M:N relationship with no junction table.)*
>
>
>
>

b) Explain why it's a problem (reference the relevant theory section)

> [!NOTE]
> ***Your Answer***
>
> *(M:N relationships cannot be implemented directly in a relational database — there is no place to store foreign keys..)*
>
>
>
>

c) Describe how to fix it

> [!NOTE]
> ***Your Answer***
>
> *(Add a junction table between them, such as `PurchaseItem '.)*
>
>
>## Error 3
a) State what the error is
> [!NOTE]
> ***Your Answer***
>
> *(No relationship exists between Books and Purchase.)*
>
>
>
>

b) Explain why it's a problem (reference the relevant theory section)

> [!NOTE]
> ***Your Answer***
>
> *(You cannot track which books are in which orders — essential data is missing.)*
>
>
>
>## Error 4
a) State what the error is
> [!NOTE]
> ***Your Answer***
>
> *(Author information is stored as a plain text string (author_name) directly inside the Books table.)*
>
>
>
>

b) Explain why it's a problem (reference the relevant theory section)

> [!NOTE]
> ***Your Answer***
>
> *(Storing author details directly in the Books table creates data redundancy and anomaly risks (Section 6 & Section 10). If an author writes
> multiple books, their name is repeated, making updates error-prone and preventing authors without books from existing in the system.)*
>
>
>
>

c) Describe how to fix it

> [!NOTE]
> ***Your Answer***
>
> *(Create a separate Author table (with author_id as PK) and link it to Books via a junction table BookAuthor to handle a Many-to-Many
> relationship, allowing books to have multiple authors and authors to write multiple books.)*
>
>
>

**Hints:** Think about multivalued attributes, M:N relationships, entity naming conventions, and missing relationships.

---

## Submission Checklist

- [ ] Exercise 1: ER diagram + design decision paragraph (including why Week 37's category FK is replaced)
- [ ] Exercise 2: All 12 theory review answers, plus 11b
- [ ] Exercise 3: All questions answered for both Diagram A and Diagram B
- [ ] Exercise 4: Entity list, relationship list, ER diagram, and justifications
- [ ] Exercise 5: Four errors identified with explanations and corrections
