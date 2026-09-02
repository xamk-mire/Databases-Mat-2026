# Week 49 — Exercises: Final Review — Database Design (Optional)

> [!IMPORTANT] Optional Exam Preparation
> **No submission is required this week.** These exercises are for your own review before the final exam. Work through as many as you find helpful — skip any you do not need.

These exercises bring together everything you have learned about database design throughout the course. You can optionally review your completed TrailShop schema, then tackle fresh design and normalization challenges similar to what you may see on the exam.

All SQL in these exercises targets **PostgreSQL**.

---

## Optional Exercise 1 — Self-Review: Your Completed TrailShop Schema

If you submitted your TrailShop project in Week 48, use these steps as a self-check — not for submission. Work through each step at your own pace to identify gaps before the exam.

### 1.1 Document Your Complete Schema

Write out every table in your TrailShop database using full `CREATE TABLE` statements. Include **all** columns, data types, constraints, and relationships. Do not leave anything out — if it exists in your database, it belongs here.

```sql
-- Example format (adapt to your actual tables):
CREATE TABLE customers (
    customer_id  SERIAL       PRIMARY KEY,
    email        VARCHAR(255) NOT NULL UNIQUE,
    first_name   VARCHAR(100) NOT NULL,
    last_name    VARCHAR(100) NOT NULL,
    created_at   TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
```

Make sure every table includes:
- A primary key
- All foreign keys with `REFERENCES` and `ON DELETE` actions
- `NOT NULL` where appropriate
- `CHECK`, `UNIQUE`, and `DEFAULT` constraints where they apply


> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

### 1.2 Create or Update Your ER Diagram

Draw (or update) an ER diagram that reflects the **final state** of your schema. Every table, column, and relationship should be visible. Use crow's foot notation for cardinalities. You can use any tool you like — draw.io, dbdiagram.io, Mermaid, or even pen and paper photographed.


> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

### 1.3 Verify Normalization

Go through each table in your schema and confirm it is in **3NF**. For every table, briefly state:
- What the primary key is
- That every non-key column depends on the whole key and nothing but the key
- If you have a deliberate exception to 3NF (e.g., a denormalized column for performance), document **why** you kept it


> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

### 1.4 List All Constraints with Business Rules

Create a table or list that maps every constraint in your schema to the business rule it enforces. For example:

| Table | Constraint | Business Rule |
|---|---|---|
| `products` | `CHECK (price > 0)` | Products cannot have zero or negative price |
| `order_items` | `CHECK (quantity >= 1)` | Every order line must contain at least one item |
| `customers` | `UNIQUE (email)` | Each customer registers with a unique email address |

Cover **all** your constraints — primary keys, foreign keys, NOT NULL, CHECK, UNIQUE, and DEFAULT.


> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

### 1.5 Self-Review

Open the **design quality checklist** from Theory Section 5. Go through every item and honestly evaluate your schema. Note any issues you find, then fix what you can. Document what you changed and why.


> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

---

## Optional Exercise 2 — Design Challenge: SoundWave

You are designing a complete database for a **music streaming service** called **SoundWave**. Here are the business requirements:

- **Users** can create accounts with an email, username, and a subscription tier (free, premium, or family).
- **Artists** have profiles with a name, bio, and one or more genres.
- **Albums** belong to an artist and have a title, release date, and cover image URL.
- **Songs** belong to an album and have a title, duration (in seconds), and track number.
- **Playlists** are created by users and have a name and description.
- Playlists contain songs — a song can appear in many playlists, and a playlist contains many songs.
- Users can **follow** artists.
- **Play history** tracks which user played which song and when.

### 2.1 Identify Entities and Attributes

List every entity you need and its attributes. Decide on data types and which columns should be required. Think about what needs to be unique.


> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

### 2.2 Describe Relationships and Cardinalities

For every relationship, state:
- Which entities are involved
- The cardinality (one-to-many, many-to-many)
- Which side holds the foreign key (or whether you need a junction table)

You can draw an ER diagram or describe the relationships in text — just be precise about cardinalities.


> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

### 2.3 Write CREATE TABLE Statements

Write complete `CREATE TABLE` statements for **all** tables. Here is a starting point to show the level of detail expected:

```sql
CREATE TABLE users (
    user_id           SERIAL        PRIMARY KEY,
    email             VARCHAR(255)  NOT NULL UNIQUE,
    username          VARCHAR(50)   NOT NULL UNIQUE,
    subscription_tier VARCHAR(10)   NOT NULL DEFAULT 'free'
                      CHECK (subscription_tier IN ('free', 'premium', 'family')),
    created_at        TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);
```

Now write the rest: `artists`, `artist_genres`, `albums`, `songs`, `playlists`, `playlist_songs`, `user_follows_artist`, and `play_history`.


> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>




### 2.4 Verify 3NF

For each table, confirm it is in 3NF. Pay special attention to:
- Did you put genres directly in the artists table as a comma-separated string? (That would violate 1NF.)
- Does `songs` have any transitive dependencies?
- Are your junction tables properly structured?




> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

### 2.5 Add Constraints

Make sure every table has appropriate:
- `PRIMARY KEY`
- `FOREIGN KEY ... REFERENCES` with `ON DELETE` actions
- `NOT NULL` on required columns
- `CHECK` constraints (e.g., duration must be positive, track number must be positive)
- `UNIQUE` where needed
- `DEFAULT` values where sensible


> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

### 2.6 Define and Justify ON DELETE Actions

For every foreign key, choose an `ON DELETE` action and explain your reasoning. Consider:

| FK Relationship | ON DELETE | Reasoning |
|---|---|---|
| `albums.artist_id → artists` | `CASCADE` or `RESTRICT`? | If an artist is deleted, should their albums disappear too, or should deletion be blocked? |
| `songs.album_id → albums` | ? | Think about data integrity vs. cleanup |
| `play_history.user_id → users` | ? | Should play history survive user deletion? |
| `playlist_songs.song_id → songs` | ? | What happens to playlists when a song is removed? |

There are no universally "correct" answers here — what matters is that you **think it through** and can defend your choice.

---

## Optional Exercise 3 — Normalization Quick Test

For each table below, determine:
1. What normal form is it currently in? (UNF, 1NF, 2NF, or 3NF)

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>
2. What specific violation exists (if any)?

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>



3. How would you fix it? Show the decomposed tables.

> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

### Table A

```sql
CREATE TABLE student_courses (
    student_id   INT          NOT NULL,
    student_name VARCHAR(100) NOT NULL,
    courses      TEXT         NOT NULL,  -- e.g., 'Math, Physics, Chemistry'
    PRIMARY KEY (student_id)
);
```

**Hint:** Look at the `courses` column. Does every cell contain a single atomic value?



> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


### Table B

```sql
CREATE TABLE order_items (
    order_id     INT            NOT NULL,
    product_id   INT            NOT NULL,
    product_name VARCHAR(200)   NOT NULL,
    unit_price   NUMERIC(10,2)  NOT NULL,
    quantity     INT            NOT NULL,
    PRIMARY KEY (order_id, product_id)
);
```

**Hint:** The primary key is composite. Does `product_name` depend on the **whole** key or just part of it?



> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


### Table C

```sql
CREATE TABLE employees (
    employee_id     INT          PRIMARY KEY,
    employee_name   VARCHAR(100) NOT NULL,
    department_id   INT          NOT NULL,
    department_name VARCHAR(100) NOT NULL
);
```

**Hint:** Does `department_name` depend on `employee_id`, or does it really depend on `department_id`?



> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


### Table D

```sql
CREATE TABLE categories (
    category_id   SERIAL       PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL UNIQUE,
    description   TEXT
);
```

**Hint:** Every non-key column depends on the primary key and nothing else. Is there a violation?



> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


### Table E

```sql
CREATE TABLE project_assignments (
    project_id       INT           NOT NULL,
    employee_id      INT           NOT NULL,
    employee_name    VARCHAR(100)  NOT NULL,
    employee_email   VARCHAR(255)  NOT NULL,
    project_name     VARCHAR(200)  NOT NULL,
    project_budget   NUMERIC(12,2) NOT NULL,
    hours_worked     INT           NOT NULL,
    PRIMARY KEY (project_id, employee_id)
);
```

**Hint:** This table has more than one problem. Which columns depend on only part of the key? Are there any transitive dependencies?



> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>


---

## Optional Exercise 4 — Peer Review Practice

Below is a schema for a **Library Management System**. It has **at least 8 intentional issues**. Your job is to find them all, explain why each is a problem, write constructive feedback, and then provide corrected `CREATE TABLE` statements.

Use the **design quality checklist** from Theory Section 5 and the **peer review guidelines** from Theory Section 7.

### The Flawed Schema

```sql
CREATE TABLE Books (
    title          VARCHAR(300)   NOT NULL,
    author_name    VARCHAR(200)   NOT NULL,
    author_email   VARCHAR(255),
    isbn           VARCHAR(13)    NOT NULL,
    genre          VARCHAR(50),
    price          VARCHAR(10),
    total_copies   INT            NOT NULL DEFAULT 1,
    available      INT            NOT NULL DEFAULT 1
);

CREATE TABLE members (
    member_id    SERIAL        PRIMARY KEY,
    full_name    VARCHAR(200)  NOT NULL,
    email        VARCHAR(255)  NOT NULL,
    phone        VARCHAR(20),
    join_date    DATE          NOT NULL DEFAULT CURRENT_DATE
);

CREATE TABLE Loans (
    loan_id      SERIAL         PRIMARY KEY,
    book_title   VARCHAR(300)   NOT NULL,
    member_id    INT            NOT NULL,
    loan_date    DATE           NOT NULL DEFAULT CURRENT_DATE,
    due_date     DATE           NOT NULL,
    return_date  DATE
);

CREATE TABLE fines (
    fine_id      SERIAL         PRIMARY KEY,
    loan_id      INT            NOT NULL,
    amount       NUMERIC(6,2)   NOT NULL,
    paid         BOOLEAN        NOT NULL DEFAULT FALSE
);
```

### 4.1 Review the Schema

Go through the schema systematically. For each issue you find, note:
- **What** the issue is
- **Where** it is (which table, which column or missing element)
- **Why** it is a problem
- **How** to fix it

Here are some categories to check:
- Primary keys — does every table have one?
- Foreign keys — are relationships properly linked?
- Data types — is every column using an appropriate type?
- Naming — are conventions consistent?
- Normalization — is every table in 3NF?
- Constraints — are business rules enforced?
- ON DELETE — are cascading behaviors defined?
- Derived data — is anything stored that could be calculated?




> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

### 4.2 Write Constructive Feedback

For each issue, write your feedback the way you would for a real peer review. Be specific, be kind, and explain the *why*. For example:

> "The `Books` table is missing a primary key. Without one, PostgreSQL cannot uniquely identify rows and foreign keys cannot reference this table. I would add a `book_id SERIAL PRIMARY KEY` column."


> [!NOTE] Your Answer
>
> *(Write your answer here.)*
>
>
>
>

### 4.3 Provide Corrected Statements

Rewrite all four tables (and add any new tables needed) with every issue fixed. Make sure the corrected schema:
- Uses consistent naming conventions
- Has proper primary and foreign keys
- Uses correct data types
- Is in 3NF
- Includes CHECK, UNIQUE, NOT NULL, and DEFAULT constraints where appropriate
- Defines ON DELETE actions on all foreign keys

---


> [!NOTE] Your SQL
>
> ```sql
> -- Write your query here
>
>
> ```

## Optional Self-Check

Use this list to gauge your readiness for design questions on the final exam. You do not need to submit anything.

- [ ] Documented your complete TrailShop schema with full CREATE TABLE statements
- [ ] Created or updated your ER diagram
- [ ] Verified 3NF for every table and documented any exceptions
- [ ] Mapped every constraint to a business rule
- [ ] Completed the self-review using the design quality checklist
- [ ] Designed the full SoundWave schema with all required tables and constraints
- [ ] Analyzed all five normalization tables and shown corrections
- [ ] Found at least 8 issues in the library schema and provided fixes
- [ ] Written constructive, specific peer review feedback

Good luck with your exam preparation — you have built up all the skills you need. Trust the process and be thorough.
