# Week 48 — Database Administration, Security, and NoSQL

## Chapter 12: "Growing the Team"

TrailShop is thriving. Orders are flowing in, the product catalog has grown to thousands of items, and new team members are joining every month. The marketing department wants to run their own analytics queries. The warehouse team needs access to inventory data. A third-party delivery app is connecting through an API. Suddenly, it's not just *you* touching the database — it's an entire organization. And with more people come more questions: Who should be able to see customer addresses? Can the intern accidentally delete the orders table? What happens if someone writes a bad query in the API and a hacker exploits it?

Your role is evolving. You're no longer just designing tables and writing queries — you're becoming a database *administrator*. That means managing who can access what, keeping the data safe through backups, defending against attacks, and starting to think about whether a relational database is always the right tool for every job. This week, you'll learn the skills that separate a developer who *uses* a database from one who *runs* one.

---

## Learning Objectives

By the end of this chapter you will be able to:

- Create and manage PostgreSQL roles and users with appropriate attributes
- Grant and revoke privileges at table, schema, column, and database level
- Explain how PostgreSQL authentication works through pg_hba.conf
- Perform database backups and restores using pg_dump, pg_restore, and pg_dumpall
- Explain what SQL injection is and how to prevent it
- Describe the four major NoSQL database categories and their use cases
- Compare relational and document-based representations of the same data
- Use PostgreSQL JSONB columns to store and query semi-structured data
- Decide when to use a relational database versus a NoSQL database

---

## 1 — Roles and Users

In a single-developer project you might connect to PostgreSQL as a superuser and never think twice. In a production system with multiple team members, applications, and services, that approach is dangerous. PostgreSQL uses a **role-based access control** system to manage who can connect and what they can do once connected.

As Watt & Eng note in *Database Design — 2nd Edition*, controlling access to data is a fundamental responsibility of database administration — the database is only as secure as its weakest account.

### 1.1 CREATE ROLE vs CREATE USER

PostgreSQL has two commands for creating accounts, but they are almost identical:

```sql
CREATE ROLE marketing_analyst;
CREATE USER warehouse_manager;
```

The only difference is that `CREATE USER` is equivalent to `CREATE ROLE ... WITH LOGIN`. A role without `LOGIN` cannot connect to the database directly — it functions as a **group** that other roles can be members of.

```sql
-- These two are identical:
CREATE USER app_user WITH PASSWORD 'secure_password_here';
CREATE ROLE app_user WITH LOGIN PASSWORD 'secure_password_here';
```

In practice, many administrators use `CREATE ROLE` for everything and explicitly add `LOGIN` when needed, making the intent clear.

### 1.2 Role Attributes

Every role has a set of boolean attributes that control its capabilities at the server level. These are separate from object-level privileges (covered in Section 2).

| Attribute | Meaning |
|---|---|
| `LOGIN` | Role can connect to the database. Without this, the role is a group only. |
| `SUPERUSER` | Bypasses all permission checks. Use sparingly — like a master key. |
| `CREATEDB` | Role can create new databases. |
| `CREATEROLE` | Role can create, alter, and drop other roles. |
| `INHERIT` | Role automatically inherits privileges of roles it is a member of. Default is `INHERIT`. |
| `REPLICATION` | Role can initiate streaming replication. Required for physical backups. |
| `BYPASSRLS` | Role bypasses Row-Level Security policies. |
| `CONNECTION LIMIT n` | Maximum number of concurrent connections for this role. `-1` means unlimited. |

You can combine multiple attributes in a single statement:

```sql
CREATE ROLE admin WITH LOGIN SUPERUSER CREATEDB CREATEROLE PASSWORD 'strong_pass';
```

To deny an attribute, prefix it with `NO`:

```sql
CREATE ROLE readonly_user WITH LOGIN NOSUPERUSER NOCREATEDB NOCREATEROLE PASSWORD 'read_pass';
```

> **Reference:** [PostgreSQL — Database Roles](https://www.postgresql.org/docs/current/user-manag.html)

### 1.3 Role Membership (Groups)

Roles can be members of other roles, which is how PostgreSQL implements **groups**. When a role is a member of another role and has the `INHERIT` attribute, it automatically gains all privileges of the parent role.

```sql
-- Create a group role (no LOGIN — it's just a container for privileges)
CREATE ROLE marketing_team;

-- Create individual users
CREATE ROLE alice WITH LOGIN PASSWORD 'alice_pass';
CREATE ROLE bob WITH LOGIN PASSWORD 'bob_pass';

-- Add users to the group
GRANT marketing_team TO alice;
GRANT marketing_team TO bob;
```

Now both Alice and Bob inherit whatever privileges are granted to `marketing_team`. If you later grant `SELECT` on the `orders` table to `marketing_team`, both Alice and Bob can immediately query it — without any changes to their individual accounts.

### 1.4 Role Hierarchy: Nested Group Membership

Groups can be members of other groups, creating a hierarchy:

```sql
-- Create a base group with read access
CREATE ROLE read_access;

-- Create a more powerful group that includes read access
CREATE ROLE write_access;
GRANT read_access TO write_access;

-- Create an admin group that includes write access (and therefore read access)
CREATE ROLE admin_access;
GRANT write_access TO admin_access;
```

This creates a privilege hierarchy: `admin_access` → `write_access` → `read_access`. A role that is a member of `admin_access` inherits everything from both `write_access` and `read_access`.

### 1.5 ALTER ROLE

You can change role attributes after creation:

```sql
-- Give an existing role the ability to create databases
ALTER ROLE marketing_analyst CREATEDB;

-- Change a password
ALTER ROLE app_user WITH PASSWORD 'new_secure_password';

-- Set a connection limit
ALTER ROLE app_user CONNECTION LIMIT 10;

-- Remove superuser privilege
ALTER ROLE former_admin NOSUPERUSER;

-- Set a session default for a role
ALTER ROLE reporting_user SET statement_timeout = '30s';
```

### 1.6 DROP ROLE

Dropping a role requires that the role owns no objects and has no granted privileges, otherwise PostgreSQL will refuse:

```sql
DROP ROLE marketing_analyst;
-- ERROR: role "marketing_analyst" cannot be dropped because some objects depend on it
```

To clean up before dropping:

```sql
-- Reassign all owned objects to another role
REASSIGN OWNED BY marketing_analyst TO postgres;

-- Drop all privileges granted to the role
DROP OWNED BY marketing_analyst;

-- Now the role can be dropped
DROP ROLE marketing_analyst;
```

### 1.7 TrailShop Role Setup

Let's set up a realistic role structure for TrailShop:

```sql
-- Group roles (no login)
CREATE ROLE trailshop_readonly;
CREATE ROLE trailshop_readwrite;
CREATE ROLE trailshop_admin;

-- Build hierarchy
GRANT trailshop_readonly TO trailshop_readwrite;
GRANT trailshop_readwrite TO trailshop_admin;

-- Individual users
CREATE ROLE marketing_analyst WITH LOGIN PASSWORD 'mktg_pass_2026';
CREATE ROLE warehouse_manager WITH LOGIN PASSWORD 'wh_pass_2026';
CREATE ROLE app_user WITH LOGIN PASSWORD 'app_pass_2026' CONNECTION LIMIT 50;
CREATE ROLE admin WITH LOGIN SUPERUSER PASSWORD 'admin_pass_2026';

-- Assign users to groups
GRANT trailshop_readonly TO marketing_analyst;
GRANT trailshop_readwrite TO warehouse_manager;
GRANT trailshop_admin TO admin;
GRANT trailshop_readonly TO app_user;
```

The marketing analyst can read data but not modify it. The warehouse manager can read and write. The admin can do everything. The application user has read access and a connection limit to prevent runaway connection pools from overwhelming the server.

---

## 2 — Privileges

Roles define *who* can connect. **Privileges** define *what* they can do once connected. PostgreSQL's privilege system is granular — you can control access at the database, schema, table, and even column level.

As *Database Design — 2nd Edition* (Watt & Eng) emphasizes, the principle of **least privilege** means every user should have only the minimum access required for their job — no more.

### 2.1 GRANT and REVOKE Syntax

The basic syntax is straightforward:

```sql
GRANT privilege_list ON object TO role;
REVOKE privilege_list ON object FROM role;
```

### 2.2 Table-Level Privileges

Tables support the following privileges:

| Privilege | Allows |
|---|---|
| `SELECT` | Read rows from the table |
| `INSERT` | Add new rows |
| `UPDATE` | Modify existing rows |
| `DELETE` | Remove rows |
| `TRUNCATE` | Empty the entire table (faster than DELETE, no row-by-row logging) |
| `REFERENCES` | Create foreign keys that reference this table |
| `TRIGGER` | Create triggers on this table |

Examples:

```sql
-- Grant read access to a group
GRANT SELECT ON products TO trailshop_readonly;

-- Grant read and write access
GRANT SELECT, INSERT, UPDATE, DELETE ON products TO trailshop_readwrite;

-- Grant all privileges
GRANT ALL PRIVILEGES ON products TO trailshop_admin;
```

### 2.3 Schema-Level Privileges

Before a role can interact with objects inside a schema, it needs schema-level access:

| Privilege | Allows |
|---|---|
| `USAGE` | Access objects within the schema (required to use any table in the schema) |
| `CREATE` | Create new objects (tables, views, etc.) within the schema |

```sql
-- Allow the readonly group to access objects in the public schema
GRANT USAGE ON SCHEMA public TO trailshop_readonly;

-- Allow the admin group to also create new objects
GRANT USAGE, CREATE ON SCHEMA public TO trailshop_admin;
```

Without `USAGE` on a schema, a role cannot see or interact with any tables inside it — even if it has been granted `SELECT` on those tables.

### 2.4 Database-Level Privileges

| Privilege | Allows |
|---|---|
| `CONNECT` | Connect to the database |
| `CREATE` | Create new schemas in the database |
| `TEMPORARY` | Create temporary tables |

```sql
GRANT CONNECT ON DATABASE trailshop TO trailshop_readonly;
GRANT CONNECT, CREATE ON DATABASE trailshop TO trailshop_admin;
```

### 2.5 Column-Level Privileges

Sometimes you need finer control. For example, the marketing team should see customer names and cities but not their email addresses or phone numbers:

```sql
GRANT SELECT (customer_id, first_name, last_name, city, country)
  ON customers
  TO marketing_analyst;
```

Now `marketing_analyst` can run:

```sql
SELECT customer_id, first_name, city FROM customers;  -- Works
SELECT email FROM customers;                           -- ERROR: permission denied
```

Column-level privileges work for `SELECT`, `INSERT`, `UPDATE`, and `REFERENCES`.

### 2.6 Default Privileges

When you create new tables, they don't automatically inherit existing grants. The `ALTER DEFAULT PRIVILEGES` command sets up privileges that are automatically applied to objects created in the future:

```sql
-- Any table created by admin in the public schema will automatically
-- be readable by the readonly group
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT ON TABLES TO trailshop_readonly;

-- Future tables will automatically be writable by the readwrite group
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO trailshop_readwrite;
```

This prevents the common mistake of creating a new table and forgetting to grant access to it.

### 2.7 ALL PRIVILEGES Shorthand

Instead of listing every privilege individually:

```sql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO trailshop_admin;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO trailshop_admin;
```

### 2.8 WITH GRANT OPTION

By default, a role that receives a privilege cannot pass it on to others. The `WITH GRANT OPTION` clause changes this:

```sql
GRANT SELECT ON products TO marketing_analyst WITH GRANT OPTION;
```

Now `marketing_analyst` can grant `SELECT` on `products` to other roles. Use this sparingly — it makes privilege management harder to track.

### 2.9 The PUBLIC Role

`PUBLIC` is a special implicit role that includes every role in the system. Granting to `PUBLIC` gives access to everyone:

```sql
-- Let anyone query the product catalog (it's public information)
GRANT SELECT ON products TO PUBLIC;
```

By default, PostgreSQL grants `CONNECT` and `CREATE TEMPORARY TABLES` on new databases to `PUBLIC`, and `USAGE` on the `public` schema. In a secure setup, you may want to revoke these:

```sql
REVOKE ALL ON DATABASE trailshop FROM PUBLIC;
REVOKE ALL ON SCHEMA public FROM PUBLIC;
```

### 2.10 Complete TrailShop Privilege Setup

Putting it all together:

```sql
-- Revoke default public access
REVOKE ALL ON DATABASE trailshop FROM PUBLIC;
REVOKE ALL ON SCHEMA public FROM PUBLIC;

-- Schema access
GRANT USAGE ON SCHEMA public TO trailshop_readonly;
GRANT USAGE, CREATE ON SCHEMA public TO trailshop_admin;

-- Database access
GRANT CONNECT ON DATABASE trailshop TO trailshop_readonly;
GRANT CONNECT ON DATABASE trailshop TO trailshop_readwrite;
GRANT CONNECT, CREATE ON DATABASE trailshop TO trailshop_admin;

-- Table access for readonly
GRANT SELECT ON ALL TABLES IN SCHEMA public TO trailshop_readonly;

-- Table access for readwrite (inherits readonly via group membership)
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO trailshop_readwrite;

-- Sequence access (needed for INSERT with serial/identity columns)
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO trailshop_readwrite;

-- Full access for admin
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO trailshop_admin;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO trailshop_admin;

-- Default privileges for future tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT ON TABLES TO trailshop_readonly;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT INSERT, UPDATE, DELETE ON TABLES TO trailshop_readwrite;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT USAGE ON SEQUENCES TO trailshop_readwrite;

-- Column-level restriction: marketing can't see customer email/phone
REVOKE SELECT ON customers FROM marketing_analyst;
GRANT SELECT (customer_id, first_name, last_name, city, country)
  ON customers TO marketing_analyst;
```

> **Reference:** [PostgreSQL — GRANT](https://www.postgresql.org/docs/current/sql-grant.html)

---

## 3 — Authentication

Roles and privileges control *what* a user can do, but **authentication** controls *whether they can connect at all*. PostgreSQL handles authentication through a configuration file called `pg_hba.conf` (HBA stands for Host-Based Authentication).

### 3.1 pg_hba.conf Overview

The `pg_hba.conf` file is typically located in the PostgreSQL data directory. Each line specifies a rule in this format:

```
TYPE   DATABASE   USER        ADDRESS          METHOD
```

PostgreSQL reads the file top-to-bottom and uses the **first matching rule**. If no rule matches, the connection is rejected.

| Field | Meaning |
|---|---|
| TYPE | `local` (Unix socket), `host` (TCP/IP), `hostssl` (SSL only), `hostnossl` |
| DATABASE | Database name, `all`, or comma-separated list |
| USER | Role name, `all`, or comma-separated list |
| ADDRESS | IP address/range (for host types), not used for `local` |
| METHOD | Authentication method to use |

### 3.2 Authentication Methods

| Method | Description | Use Case |
|---|---|---|
| `trust` | No password required — anyone who can connect is accepted | Local development only. **Never in production.** |
| `password` | Plain-text password sent over the network | Insecure. Avoid. |
| `md5` | MD5-hashed password challenge | Legacy. Being replaced by scram-sha-256. |
| `scram-sha-256` | Modern salted challenge-response authentication | **Recommended for production.** |
| `peer` | OS username must match database role name | Local Unix connections. |
| `ident` | Like peer, but for TCP connections via an ident server | Rare. |
| `cert` | Client SSL certificate authentication | High-security environments. |

### 3.3 Connection Security: SSL/TLS

For any connection over a network, you should use SSL/TLS encryption to prevent eavesdropping. PostgreSQL supports SSL natively. The `hostssl` connection type in `pg_hba.conf` requires SSL for matching connections.

### 3.4 Example pg_hba.conf Entries

```
# Local connections: trust for the postgres superuser (development only)
local   all             postgres                                trust

# Local connections: scram-sha-256 for all other users
local   all             all                                     scram-sha-256

# IPv4 connections from the application server
host    trailshop       app_user        192.168.1.100/32        scram-sha-256

# IPv4 connections from the office network (marketing, warehouse)
host    trailshop       all             192.168.1.0/24          scram-sha-256

# SSL-only connections from anywhere (remote admin)
hostssl trailshop       admin           0.0.0.0/0               scram-sha-256

# Reject everything else (implicit — but explicit is clearer)
host    all             all             0.0.0.0/0               reject
```

After editing `pg_hba.conf`, you must reload the configuration:

```sql
SELECT pg_reload_conf();
```

Or from the command line:

```bash
pg_ctl reload
```

---

## 4 — Backup and Restore

Databases fail. Disks crash. Humans make mistakes. A `DELETE` without a `WHERE` clause can wipe out years of data in milliseconds. **Backups are not optional** — they are the single most important administrative task.

As Watt & Eng stress in *Database Design — 2nd Edition*, a database without a backup strategy is a liability, not an asset.

### 4.1 pg_dump: Logical Backups

`pg_dump` creates a logical backup of a single database — a file containing SQL statements (or a compressed archive) that can recreate the database.

#### Output Formats

| Format | Flag | Description |
|---|---|---|
| Plain SQL | `-Fp` (default) | Human-readable SQL script. Can be restored with `psql`. |
| Custom | `-Fc` | Compressed, non-human-readable. Supports selective restore. **Recommended.** |
| Directory | `-Fd` | One file per table, supports parallel dump. Good for large databases. |
| Tar | `-Ft` | Tar archive format. |

#### Basic Usage

```bash
# Plain SQL dump
pg_dump -U postgres -Fp trailshop > trailshop_backup.sql

# Custom format (compressed, recommended)
pg_dump -U postgres -Fc trailshop -f trailshop_backup.dump

# Directory format with parallel jobs
pg_dump -U postgres -Fd trailshop -j 4 -f trailshop_backup_dir/
```

#### Selective Dumps

```bash
# Dump only the products table
pg_dump -U postgres -Fc -t products trailshop -f products_only.dump

# Dump only tables in a specific schema
pg_dump -U postgres -Fc -n inventory trailshop -f inventory_schema.dump

# Dump only the schema (structure), no data
pg_dump -U postgres -Fc --schema-only trailshop -f trailshop_structure.dump

# Dump only the data, no schema
pg_dump -U postgres -Fc --data-only trailshop -f trailshop_data.dump
```

### 4.2 pg_restore: Restoring Custom Format Backups

`pg_restore` works with custom (`-Fc`), directory (`-Fd`), and tar (`-Ft`) format backups. For plain SQL backups, use `psql` instead.

```bash
# Restore an entire database from custom format
pg_restore -U postgres -d trailshop_restored trailshop_backup.dump

# Restore only the products table
pg_restore -U postgres -d trailshop -t products trailshop_backup.dump

# Restore only a specific schema
pg_restore -U postgres -d trailshop -n inventory trailshop_backup.dump

# List the contents of a backup (what's inside?)
pg_restore -l trailshop_backup.dump
```

For plain SQL backups:

```bash
psql -U postgres -d trailshop_restored -f trailshop_backup.sql
```

### 4.3 pg_dumpall: Server-Wide Backups

`pg_dump` exports a single database but does *not* include server-wide objects like roles, tablespaces, and database definitions. For those, use `pg_dumpall`:

```bash
# Dump everything: all databases, all roles
pg_dumpall -U postgres > full_server_backup.sql

# Dump only roles and other global objects (no database data)
pg_dumpall -U postgres --globals-only > globals_backup.sql
```

A common strategy is to combine `pg_dumpall --globals-only` (for roles) with individual `pg_dump -Fc` backups for each database.

### 4.4 Backup Strategies

| Strategy | Description | Pros | Cons |
|---|---|---|---|
| Full backup | Complete dump at regular intervals | Simple, self-contained | Slow for large databases, large files |
| Incremental | Only changes since last backup | Smaller, faster | More complex to restore |
| Continuous archiving (WAL) | Archive Write-Ahead Log files continuously | Enables point-in-time recovery | Requires more setup and storage |

### 4.5 Point-in-Time Recovery (PITR)

PostgreSQL writes every change to the **Write-Ahead Log (WAL)** before applying it to the actual data files. By archiving WAL files, you can restore a database to *any point in time* — not just the moment of the last backup.

The concept:

1. Take a **base backup** (a copy of the data directory)
2. Continuously archive WAL files as they are produced
3. To recover: restore the base backup, then replay WAL files up to the desired point in time

This is critical for production systems where you might need to recover to the state "five minutes before the accidental DELETE."

### 4.6 pg_basebackup: Physical Backups

`pg_basebackup` creates a physical copy of the entire PostgreSQL data directory, which is used as the starting point for PITR:

```bash
pg_basebackup -U replication_user -D /backups/base_backup -Ft -z -P
```

| Flag | Meaning |
|---|---|
| `-D` | Destination directory |
| `-Ft` | Tar format |
| `-z` | Compress with gzip |
| `-P` | Show progress |

### 4.7 TrailShop Backup Walkthrough

A practical backup routine for TrailShop:

```bash
# 1. Backup global objects (roles, etc.)
pg_dumpall -U postgres --globals-only -f /backups/globals_$(date +%Y%m%d).sql

# 2. Backup the trailshop database in custom format
pg_dump -U postgres -Fc trailshop -f /backups/trailshop_$(date +%Y%m%d).dump

# 3. Verify the backup by listing its contents
pg_restore -l /backups/trailshop_$(date +%Y%m%d).dump | head -20
```

To restore into a fresh database:

```bash
# Create a new empty database
createdb -U postgres trailshop_restored

# Restore global objects first (roles needed for ownership)
psql -U postgres -f /backups/globals_20261125.sql

# Restore the database
pg_restore -U postgres -d trailshop_restored /backups/trailshop_20261125.dump
```

> **Reference:** [PostgreSQL — Backup and Restore](https://www.postgresql.org/docs/current/backup.html)

---

## 5 — SQL Injection

You've secured the database with roles and privileges, set up authentication, and configured backups. But there's another threat that comes not from inside the database, but from the *application layer*: **SQL injection**.

SQL injection is consistently ranked among the most dangerous web application vulnerabilities. It has been used to steal millions of records, destroy databases, and bypass authentication in real-world attacks.

### 5.1 What Is SQL Injection?

SQL injection occurs when **untrusted user input is included directly in a SQL query** without proper handling. The database cannot distinguish between the developer's intended SQL and the attacker's injected SQL — it executes everything.

As Watt & Eng note in *Database Design — 2nd Edition*, security is not just a database concern — it spans the entire application stack, and the boundary between user input and SQL commands must be guarded carefully.

### 5.2 How It Works: String Concatenation

Consider a login form where the application builds a query by concatenating user input:

```python
# VULNERABLE — never do this
username = request.form['username']
password = request.form['password']

query = "SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'"
cursor.execute(query)
```

If the user enters `alice` and `secret123`, the query becomes:

```sql
SELECT * FROM users WHERE username = 'alice' AND password = 'secret123'
```

That works as expected. But what if the user enters something malicious?

### 5.3 Classic Attack Examples

#### Login Bypass

An attacker enters in the username field:

```
' OR '1'='1
```

The query becomes:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = ''
```

Since `'1'='1'` is always true, this returns all users — and the application logs the attacker in as the first user (often the admin).

#### Data Exfiltration with UNION SELECT

An attacker manipulates a product search to extract data from other tables:

```
' UNION SELECT username, password, NULL, NULL FROM users --
```

The resulting query:

```sql
SELECT product_id, name, price, description FROM products
WHERE name LIKE '' UNION SELECT username, password, NULL, NULL FROM users --'
```

The `--` comments out the rest of the original query. The attacker now sees usernames and passwords displayed alongside product results.

#### Data Destruction

The most dramatic example:

```
'; DROP TABLE users; --
```

The resulting query:

```sql
SELECT * FROM users WHERE username = ''; DROP TABLE users; --' AND password = ''
```

This executes two statements: a harmless SELECT, then `DROP TABLE users`. The entire users table is gone.

#### Blind SQL Injection

In blind injection, the attacker doesn't see query results directly but infers information from the application's behavior (e.g., different error messages, response times). For example, an attacker might determine if the first character of a password is 'a' by injecting a condition that causes a noticeable delay:

```sql
' AND (SELECT CASE WHEN (substring(password,1,1)='a') THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users WHERE username='admin') --
```

If the page takes 5 seconds to load, the first character is 'a'. The attacker repeats this character by character.

### 5.4 Defense Strategy 1: Parameterized Queries (Primary Defense)

The most effective defense is to **never concatenate user input into SQL**. Instead, use parameterized queries (also called prepared statements), where the query structure and the data are sent separately to the database:

```python
import psycopg2

# SAFE — parameterized query
conn = psycopg2.connect("dbname=trailshop user=app_user")
cur = conn.cursor()

username = request.form['username']
password = request.form['password']

cur.execute(
    "SELECT * FROM users WHERE username = %s AND password = %s",
    (username, password)
)
```

The `%s` placeholders are not string formatting — they are parameters that psycopg2 sends to PostgreSQL separately from the query text. Even if the user enters `' OR '1'='1`, PostgreSQL treats it as a literal string value, not as SQL code.

Here is the vulnerable version for comparison:

```python
# VULNERABLE — string concatenation
query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
cur.execute(query)

# SAFE — parameterized query
cur.execute(
    "SELECT * FROM users WHERE username = %s AND password = %s",
    (username, password)
)
```

### 5.5 Defense Strategy 2: Input Validation and Sanitization

While parameterized queries are the primary defense, validating input provides an additional layer:

- Reject input that contains unexpected characters
- Enforce length limits
- Validate data types (e.g., ensure a product ID is an integer)
- Use allowlists rather than blocklists when possible

```python
import re

product_id = request.form['product_id']

# Validate: product_id must be a positive integer
if not re.match(r'^\d+$', product_id):
    return "Invalid product ID", 400
```

Input validation alone is **not sufficient** — always use parameterized queries as well.

### 5.6 Defense Strategy 3: Least Privilege

The TrailShop web application connects as `app_user`, which has only `SELECT` privileges. Even if an attacker achieves SQL injection, they cannot:

- `DROP TABLE` (no `DROP` privilege)
- `DELETE FROM orders` (no `DELETE` privilege)
- `INSERT INTO users` (no `INSERT` privilege on users table)

This limits the blast radius of a successful attack.

### 5.7 Defense Strategy 4: ORM Protections

Object-Relational Mappers (ORMs) like SQLAlchemy, Django ORM, and Hibernate automatically use parameterized queries when you use their query APIs:

```python
# Django ORM — automatically parameterized
user = User.objects.filter(username=username, password=password).first()
```

However, most ORMs also allow raw SQL, which can still be vulnerable if misused:

```python
# Django raw SQL — STILL VULNERABLE if you concatenate
User.objects.raw(f"SELECT * FROM users WHERE username = '{username}'")

# Django raw SQL — safe with parameters
User.objects.raw("SELECT * FROM users WHERE username = %s", [username])
```

### 5.8 Defense Strategy 5: Web Application Firewalls (WAF)

A Web Application Firewall sits between users and the application, scanning HTTP requests for common attack patterns. WAFs can block many injection attempts but should be considered a **supplementary defense**, not a primary one — skilled attackers can often bypass WAFs with encoding tricks.

### 5.9 Second-Order SQL Injection

In second-order injection, the malicious input is stored safely in the database but then used unsafely in a later query. For example:

1. A user registers with the username `admin'--`
2. The registration uses parameterized queries, so the username is stored safely
3. Later, an admin panel query builds SQL using the stored username with string concatenation
4. The injection executes

The defense is the same: **always use parameterized queries**, even when the data comes from your own database.

> **Reference:** [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)

---

## 6 — NoSQL: Beyond the Relational Model

Everything you've learned this semester has been about relational databases — and for good reason. Relational databases are the right choice for most applications. But they're not the *only* choice. Some problems are awkward to model relationally, and some systems have performance or scalability needs that push beyond what traditional relational databases handle easily.

**NoSQL** ("Not Only SQL") is an umbrella term for databases that store and retrieve data in ways other than the traditional table-with-rows-and-columns model.

### 6.1 Why NoSQL?

Relational databases assume:

- Data has a fixed, predictable structure (schema)
- Relationships between entities are important
- Consistency is the top priority (ACID transactions)
- The database runs on a single server (or a small cluster)

These assumptions work well for most business applications. But consider:

- A product catalog where every product has *different* attributes (a tent has "capacity" and "weight," a GPS has "battery life" and "screen size")
- A social media feed that needs to handle millions of reads per second with low latency
- An IoT system ingesting billions of sensor readings per day
- A recommendation engine that needs to traverse complex webs of relationships

In these cases, a different data model might be a better fit.

### 6.2 CAP Theorem

The **CAP theorem** (Brewer, 2000) states that a distributed database can guarantee at most two of three properties:

| Property | Meaning |
|---|---|
| **Consistency** | Every read returns the most recent write |
| **Availability** | Every request receives a response (even if it's not the latest data) |
| **Partition Tolerance** | The system continues to operate even if network communication between nodes fails |

Since network partitions *will* happen in any distributed system, the real choice is between **consistency** and **availability** during a partition:

- **CP systems** (Consistency + Partition tolerance): Refuse to respond rather than return stale data. Example: traditional relational databases, MongoDB (default).
- **AP systems** (Availability + Partition tolerance): Always respond, even if the data might be slightly out of date. Example: Cassandra, DynamoDB.

### 6.3 Eventual Consistency

Many NoSQL databases use **eventual consistency**: after a write, all replicas *will eventually* converge to the same value, but there's a window where different nodes might return different results.

Think of it like updating a shared Google Doc: if two people are editing simultaneously, they might briefly see different versions, but eventually the document syncs up. For many use cases (social media likes, product view counts, cached data), this is perfectly acceptable.

### 6.4 Document Databases (MongoDB)

Document databases store data as **documents** — typically JSON-like objects. Each document can have a different structure, and documents can contain nested objects and arrays.

**Key concepts:**

- **Document**: A JSON/BSON object (like a row, but flexible)
- **Collection**: A group of documents (like a table, but no enforced schema)
- **Embedded documents**: Nested objects within a document (denormalization)

**TrailShop product as a MongoDB document:**

```json
{
  "_id": "prod_1001",
  "name": "Alpine Pro Tent",
  "category": "Tents",
  "price": 299.99,
  "stock_quantity": 45,
  "attributes": {
    "capacity": "3-person",
    "weight_kg": 2.8,
    "waterproof_rating": "3000mm",
    "seasons": ["spring", "summer", "autumn"]
  },
  "reviews": [
    {
      "user": "hiker_jane",
      "rating": 5,
      "comment": "Survived a storm in Lapland!",
      "date": "2026-08-15"
    },
    {
      "user": "weekend_camper",
      "rating": 4,
      "comment": "Great tent, a bit heavy for long treks.",
      "date": "2026-09-02"
    }
  ]
}
```

Notice how the reviews are **embedded** inside the product document. In a relational database, these would be in a separate `reviews` table with a foreign key. In a document database, related data is stored together, which makes reads very fast — one query retrieves everything.

**When to use document databases:**

- Content management systems
- Product catalogs with varying attributes
- User profiles with flexible fields
- Applications where data is read much more often than written
- Rapid prototyping where the schema is evolving

### 6.5 Key-Value Stores (Redis)

The simplest NoSQL model: every piece of data is stored as a **key** mapped to a **value**. Think of it as a giant hash map / dictionary.

```
SET "session:user123" "{\"username\":\"alice\",\"cart\":[101,205]}"
GET "session:user123"

SET "product:1001:views" "15432"
INCR "product:1001:views"
```

**Key characteristics:**

- Extremely fast (data stored in memory)
- No query language for the values — you look up by key
- Values can be strings, lists, sets, hashes, or sorted sets

**Use cases:**

- **Caching**: Store frequently accessed query results (`product:1001:details`)
- **Session management**: Store user sessions with automatic expiration
- **Rate limiting**: Count API requests per user per minute
- **Counters**: Real-time product view counts, like counts
- **Queues**: Task queues for background processing

**TrailShop example — caching product page views:**

```
SET "cache:product:1001" "{\"name\":\"Alpine Pro Tent\",\"price\":299.99}" EX 3600
```

This caches the product data for 3600 seconds (1 hour). Subsequent requests read from Redis instead of hitting the PostgreSQL database, dramatically reducing database load.

### 6.6 Column-Family Stores (Cassandra)

Column-family databases organize data into **rows** identified by a key, where each row can have a different set of **columns**. They are optimized for writing massive amounts of data across many servers.

**Key characteristics:**

- Designed for very high write throughput
- Data is distributed across a cluster of nodes
- Eventual consistency (AP system)
- Queries must follow the partition key — you design the schema around your queries

**Use cases:**

- Time-series data (sensor readings, metrics, logs)
- IoT data ingestion
- Event tracking and analytics
- Any workload with huge write volumes

**Example: TrailShop order events in Cassandra-style thinking:**

```
Row key: "customer_123"
  → column: "2026-11-20T10:30:00" → {"order_id": 5001, "total": 149.99}
  → column: "2026-11-21T14:15:00" → {"order_id": 5023, "total": 89.50}
  → column: "2026-11-22T09:00:00" → {"order_id": 5041, "total": 324.00}
```

### 6.7 Graph Databases (Neo4j)

Graph databases store data as **nodes** (entities) and **edges** (relationships between entities). Both nodes and edges can have **properties** (key-value pairs).

**Key concepts:**

- **Node**: An entity (a customer, a product, a category)
- **Edge/Relationship**: A connection between two nodes, with a type and direction
- **Properties**: Key-value data attached to nodes or edges

**TrailShop example — "customers who bought X also bought Y":**

```
(:Customer {name: "Alice"}) -[:PURCHASED]-> (:Product {name: "Alpine Pro Tent"})
(:Customer {name: "Alice"}) -[:PURCHASED]-> (:Product {name: "Hiking Poles"})
(:Customer {name: "Bob"})   -[:PURCHASED]-> (:Product {name: "Alpine Pro Tent"})
(:Customer {name: "Bob"})   -[:PURCHASED]-> (:Product {name: "Camp Stove"})
```

A graph query to find "products bought by customers who also bought the Alpine Pro Tent":

```cypher
MATCH (p:Product {name: "Alpine Pro Tent"})<-[:PURCHASED]-(c:Customer)-[:PURCHASED]->(other:Product)
WHERE other.name <> "Alpine Pro Tent"
RETURN other.name, COUNT(c) AS co_purchases
ORDER BY co_purchases DESC
```

In a relational database, this would require multiple JOINs. In a graph database, it's a natural traversal.

**Use cases:**

- Social networks (friends of friends)
- Recommendation engines
- Fraud detection (finding suspicious patterns in transaction networks)
- Knowledge graphs
- Network and IT infrastructure mapping

### 6.8 NoSQL Comparison Table

| Type | Data Model | Example DB | Best For | Not Good For |
|---|---|---|---|---|
| Document | JSON-like documents | MongoDB, CouchDB | Flexible schemas, catalogs, CMS | Complex joins, multi-document transactions |
| Key-Value | Key → Value pairs | Redis, DynamoDB | Caching, sessions, counters | Complex queries, relationships |
| Column-Family | Rows with dynamic columns | Cassandra, HBase | Time-series, IoT, high write volume | Ad-hoc queries, complex reads |
| Graph | Nodes and edges | Neo4j, Amazon Neptune | Relationships, recommendations, networks | Bulk data processing, simple CRUD |
| Relational | Tables with fixed schema | PostgreSQL, MySQL | Structured data, ACID, complex queries | Massive horizontal scale, flexible schema |

> **Reference:** [MongoDB — Introduction](https://www.mongodb.com/docs/manual/introduction/)

---

## 7 — TrailShop Comparison: Relational vs Document

To make the difference concrete, let's model the same data — product reviews — in both relational and document form.

### 7.1 Relational Approach

```sql
CREATE TABLE products (
    product_id   SERIAL PRIMARY KEY,
    name         VARCHAR(200) NOT NULL,
    category     VARCHAR(100),
    price        NUMERIC(10,2) NOT NULL,
    stock_qty    INTEGER DEFAULT 0
);

CREATE TABLE product_attributes (
    attribute_id SERIAL PRIMARY KEY,
    product_id   INTEGER REFERENCES products(product_id),
    attr_name    VARCHAR(100) NOT NULL,
    attr_value   VARCHAR(200) NOT NULL
);

CREATE TABLE reviews (
    review_id    SERIAL PRIMARY KEY,
    product_id   INTEGER REFERENCES products(product_id),
    username     VARCHAR(100) NOT NULL,
    rating       INTEGER CHECK (rating BETWEEN 1 AND 5),
    comment      TEXT,
    review_date  DATE DEFAULT CURRENT_DATE
);
```

To get a product with its attributes and reviews, you need JOINs:

```sql
SELECT p.name, p.price, pa.attr_name, pa.attr_value, r.username, r.rating, r.comment
FROM products p
LEFT JOIN product_attributes pa ON p.product_id = pa.product_id
LEFT JOIN reviews r ON p.product_id = r.product_id
WHERE p.product_id = 1001;
```

### 7.2 Document Approach

```json
{
  "_id": "prod_1001",
  "name": "Alpine Pro Tent",
  "category": "Tents",
  "price": 299.99,
  "stock_qty": 45,
  "attributes": {
    "capacity": "3-person",
    "weight_kg": 2.8,
    "waterproof_rating": "3000mm"
  },
  "reviews": [
    {"username": "hiker_jane", "rating": 5, "comment": "Survived a storm in Lapland!", "date": "2026-08-15"},
    {"username": "weekend_camper", "rating": 4, "comment": "Great tent, a bit heavy.", "date": "2026-09-02"}
  ]
}
```

One query retrieves everything — no JOINs needed.

### 7.3 Comparison

| Criterion | Relational (PostgreSQL) | Document (MongoDB) |
|---|---|---|
| Schema enforcement | Strict — all rows follow the same structure | Flexible — each document can differ |
| Adding a new attribute | Requires ALTER TABLE or an EAV pattern | Just add a field to the document |
| Querying relationships | JOINs — powerful but can be slow | Embedded data — fast reads, but duplication |
| Data integrity | FOREIGN KEY, CHECK, UNIQUE constraints | Application-level validation (mostly) |
| Transaction support | Full ACID across multiple tables | ACID per document; multi-document transactions added in MongoDB 4.0+ |
| Storage efficiency | Normalized — no duplication | Denormalized — data may be duplicated |
| Update complexity | Update one row, all references see the change | Must update every copy of duplicated data |
| Best for this scenario | Reviews that need independent querying, analytics | Product pages that need fast single-document reads |

---

## 8 — PostgreSQL JSONB: Best of Both Worlds

What if you want the reliability and query power of PostgreSQL *and* the flexibility of JSON documents? PostgreSQL's **JSONB** data type gives you exactly that.

JSONB stores JSON data in a binary format that supports indexing and efficient querying. It's not a toy feature — it's a production-grade capability that many teams use instead of (or alongside) a dedicated document database.

### 8.1 Creating a Table with JSONB

```sql
CREATE TABLE product_reviews (
    review_id    SERIAL PRIMARY KEY,
    product_id   INTEGER REFERENCES products(product_id),
    review_data  JSONB NOT NULL,
    created_at   TIMESTAMPTZ DEFAULT NOW()
);
```

Here, `review_data` can hold any JSON structure — different reviews can have different fields.

### 8.2 Inserting JSON Data

```sql
INSERT INTO product_reviews (product_id, review_data) VALUES
(1001, '{
    "username": "hiker_jane",
    "rating": 5,
    "comment": "Survived a storm in Lapland!",
    "verified_purchase": true,
    "photos": ["img001.jpg", "img002.jpg"],
    "pros": ["waterproof", "lightweight", "easy setup"],
    "cons": ["expensive"]
}'),
(1001, '{
    "username": "weekend_camper",
    "rating": 4,
    "comment": "Great tent, a bit heavy for long treks.",
    "verified_purchase": false,
    "pros": ["spacious", "durable"],
    "cons": ["heavy", "no footprint included"]
}');
```

Notice how one review has a `photos` array and the other doesn't. This flexibility is impossible with a fixed relational schema.

### 8.3 Querying JSONB Data

PostgreSQL provides several operators for querying JSONB:

| Operator | Returns | Description |
|---|---|---|
| `->` | JSONB | Get JSON object field by key |
| `->>` | TEXT | Get JSON object field as text |
| `@>` | BOOLEAN | Does the JSON contain the given value? |
| `?` | BOOLEAN | Does the JSON contain the given key? |
| `#>` | JSONB | Get value at a nested path |
| `#>>` | TEXT | Get value at a nested path as text |

#### Extracting Fields

```sql
-- Get the username as text
SELECT review_data->>'username' AS username,
       review_data->>'rating' AS rating
FROM product_reviews
WHERE product_id = 1001;
```

#### Filtering by JSON Content

```sql
-- Find all 5-star reviews
SELECT review_data->>'username' AS reviewer,
       review_data->>'comment' AS comment
FROM product_reviews
WHERE (review_data->>'rating')::INTEGER = 5;

-- Find reviews that contain a specific value
SELECT review_data->>'username' AS reviewer
FROM product_reviews
WHERE review_data @> '{"verified_purchase": true}';

-- Find reviews that have a "photos" key
SELECT review_data->>'username' AS reviewer
FROM product_reviews
WHERE review_data ? 'photos';
```

#### Nested Path Access

```sql
-- Get the first photo from a review
SELECT review_data #>> '{photos, 0}' AS first_photo
FROM product_reviews
WHERE review_data ? 'photos';
```

### 8.4 Updating JSONB Data

#### Using jsonb_set()

```sql
-- Update the rating of a specific review
UPDATE product_reviews
SET review_data = jsonb_set(review_data, '{rating}', '3')
WHERE review_id = 1;

-- Add a new field to an existing review
UPDATE product_reviews
SET review_data = jsonb_set(review_data, '{helpful_votes}', '12')
WHERE review_id = 1;
```

#### Using the || (Concatenation) Operator

```sql
-- Merge new fields into existing JSON
UPDATE product_reviews
SET review_data = review_data || '{"edited": true, "edit_date": "2026-11-25"}'
WHERE review_id = 1;
```

### 8.5 GIN Indexes on JSONB

Without an index, JSONB queries scan every row. A **GIN (Generalized Inverted Index)** index makes `@>`, `?`, `?|`, and `?&` operators fast:

```sql
CREATE INDEX idx_review_data ON product_reviews USING GIN (review_data);
```

Now queries like `WHERE review_data @> '{"verified_purchase": true}'` can use the index instead of scanning the entire table.

For queries that only access specific keys, a targeted expression index may be more efficient:

```sql
CREATE INDEX idx_review_rating ON product_reviews ((review_data->>'rating'));
```

### 8.6 SQL/JSON Path Queries

PostgreSQL 12+ supports the SQL/JSON path language for more expressive queries:

```sql
-- Find reviews where any element in the "pros" array equals "waterproof"
SELECT review_data->>'username' AS reviewer
FROM product_reviews
WHERE jsonb_path_exists(review_data, '$.pros[*] ? (@ == "waterproof")');
```

### 8.7 Complete TrailShop JSONB Example

A flexible product table that combines relational structure with JSONB flexibility:

```sql
CREATE TABLE products_flexible (
    product_id   SERIAL PRIMARY KEY,
    name         VARCHAR(200) NOT NULL,
    category     VARCHAR(100) NOT NULL,
    price        NUMERIC(10,2) NOT NULL,
    stock_qty    INTEGER DEFAULT 0,
    attributes   JSONB DEFAULT '{}'::jsonb,
    metadata     JSONB DEFAULT '{}'::jsonb
);

-- Insert products with different attributes
INSERT INTO products_flexible (name, category, price, stock_qty, attributes) VALUES
('Alpine Pro Tent', 'Tents', 299.99, 45,
 '{"capacity": "3-person", "weight_kg": 2.8, "waterproof_rating": "3000mm", "seasons": 3}'),
('TrailNav GPS', 'Electronics', 199.99, 120,
 '{"battery_life_hours": 16, "screen_size_inches": 2.6, "waterproof": true, "connectivity": ["USB-C", "Bluetooth"]}'),
('Summit Sleeping Bag', 'Sleeping', 149.99, 80,
 '{"temperature_rating_c": -15, "fill": "synthetic", "weight_kg": 1.2, "packed_size_cm": "35x20"}');

-- Query: find all products under 1.5 kg
SELECT name, price, attributes->>'weight_kg' AS weight
FROM products_flexible
WHERE (attributes->>'weight_kg')::NUMERIC < 1.5;

-- Query: find all waterproof products
SELECT name, category
FROM products_flexible
WHERE attributes @> '{"waterproof": true}'
   OR attributes ? 'waterproof_rating';

-- Index for fast attribute queries
CREATE INDEX idx_product_attrs ON products_flexible USING GIN (attributes);
```

### 8.8 When to Use JSONB vs Separate Tables

| Use JSONB When | Use Separate Tables When |
|---|---|
| Attributes vary widely across rows | All rows share the same structure |
| Schema changes frequently | Schema is stable |
| You query the JSON data occasionally | You query and filter the data frequently |
| Data is mostly read as a blob | Data is joined, aggregated, or reported on |
| You need flexible metadata storage | You need referential integrity (foreign keys) |
| Rapid prototyping and iteration | Production systems with strict data quality needs |

A common pattern is to use relational columns for data you query and filter on (price, category, stock) and JSONB for variable metadata (product-specific attributes, user preferences, feature flags).

---

## 9 — When Relational vs NoSQL: A Decision Framework

The question is not "which is better?" but "which is better *for this problem*?"

### 9.1 Decision Matrix

| Criterion | Choose Relational | Choose NoSQL |
|---|---|---|
| Data structure | Fixed, predictable schema | Variable, evolving schema |
| Relationships | Complex relationships between entities | Few or embedded relationships |
| Consistency | ACID transactions required | Eventual consistency acceptable |
| Query patterns | Complex queries, JOINs, aggregations | Simple lookups by key or document |
| Scale | Vertical (bigger server) | Horizontal (more servers) |
| Data volume | Moderate (GBs to low TBs) | Massive (TBs to PBs) |
| Write throughput | Moderate | Very high (millions per second) |
| Development speed | Stable schema, long-term projects | Rapid prototyping, evolving requirements |
| Team expertise | SQL skills widely available | Requires learning new query paradigms |
| Tooling and ecosystem | Mature, extensive | Growing, varies by database |

### 9.2 Hybrid Approaches

In practice, many production systems use **both** relational and NoSQL databases:

- **PostgreSQL** for core business data (orders, customers, inventory) where ACID matters
- **Redis** for caching frequently accessed data and managing sessions
- **MongoDB** or **PostgreSQL JSONB** for product catalogs with variable attributes
- **Elasticsearch** for full-text search across product descriptions
- **Cassandra** for high-volume event logging and analytics

The TrailShop team might start with PostgreSQL for everything, then add Redis for caching when traffic grows, and later add Elasticsearch for search — without replacing PostgreSQL.

### 9.3 The Golden Rule

**Start relational, add NoSQL when needed.**

Relational databases are the safest default choice. They handle most workloads well, have decades of tooling and expertise behind them, and PostgreSQL in particular keeps adding features (like JSONB) that cover many NoSQL use cases. Only reach for a specialized NoSQL database when you have a *specific* problem that relational databases handle poorly — and you've measured it, not just assumed it.

---

## Key Terms

| Term | Definition |
|---|---|
| **Role** | A database entity that can own objects and have privileges; may or may not be able to log in |
| **User** | A role with the LOGIN attribute — can connect to the database |
| **GRANT** | SQL command to give privileges on objects to roles |
| **REVOKE** | SQL command to remove privileges from roles |
| **Privilege** | A specific permission (SELECT, INSERT, etc.) on a database object |
| **pg_dump** | PostgreSQL utility for creating logical backups of a single database |
| **pg_restore** | PostgreSQL utility for restoring backups created by pg_dump in non-plain formats |
| **Backup** | A copy of database data and/or structure that can be used for recovery |
| **Restore** | The process of recreating a database from a backup |
| **SQL injection** | An attack where malicious SQL is inserted through user input into a query |
| **Parameterized query** | A query where user input is passed as parameters, not concatenated into the SQL string |
| **Prepared statement** | A precompiled SQL statement that uses parameter placeholders, preventing injection |
| **NoSQL** | An umbrella term for databases that use data models other than relational tables |
| **Document database** | A NoSQL database that stores data as JSON-like documents (e.g., MongoDB) |
| **JSON** | JavaScript Object Notation — a lightweight text format for structured data |
| **JSONB** | PostgreSQL's binary JSON type, supporting indexing and efficient querying |
| **Key-value store** | A NoSQL database that maps keys to values, optimized for simple lookups (e.g., Redis) |
| **Column-family store** | A NoSQL database organized by columns rather than rows, optimized for writes (e.g., Cassandra) |
| **Graph database** | A NoSQL database that stores nodes and relationships, optimized for traversals (e.g., Neo4j) |
| **CAP theorem** | States that a distributed system can guarantee at most two of: Consistency, Availability, Partition tolerance |
| **Eventual consistency** | A model where replicas will converge to the same value over time, but may differ briefly |
| **pg_hba.conf** | PostgreSQL's Host-Based Authentication configuration file |

---

## Reading Assignments

1. **PostgreSQL Documentation — Database Roles:**
   [https://www.postgresql.org/docs/current/user-manag.html](https://www.postgresql.org/docs/current/user-manag.html)
   Read Chapters 22.1 through 22.4 for a thorough understanding of role management.

2. **PostgreSQL Documentation — GRANT:**
   [https://www.postgresql.org/docs/current/sql-grant.html](https://www.postgresql.org/docs/current/sql-grant.html)
   Study the full syntax and examples for granting privileges at various levels.

3. **PostgreSQL Documentation — Backup and Restore:**
   [https://www.postgresql.org/docs/current/backup.html](https://www.postgresql.org/docs/current/backup.html)
   Read Chapters 26.1 (SQL Dump) and 26.2 (File System Level Backup) for practical backup knowledge.

4. **OWASP SQL Injection Prevention Cheat Sheet:**
   [https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
   Essential reading for anyone building applications that interact with databases.

---

## Further Reading

- **MongoDB Introduction:**
  [https://www.mongodb.com/docs/manual/introduction/](https://www.mongodb.com/docs/manual/introduction/)
  A well-written introduction to document databases and MongoDB concepts.

- **AWS — What Is NoSQL?:**
  [https://aws.amazon.com/nosql/](https://aws.amazon.com/nosql/)
  A clear, vendor-neutral overview of NoSQL database types and their use cases.

- **PostgreSQL Documentation — JSON Types:**
  [https://www.postgresql.org/docs/current/datatype-json.html](https://www.postgresql.org/docs/current/datatype-json.html)
  Complete reference for JSON and JSONB data types in PostgreSQL.

- **Redis Documentation:**
  [https://redis.io/docs/](https://redis.io/docs/)
  If you're curious about key-value stores and caching, Redis has excellent documentation.

---

## Summary

This week you made the leap from database *developer* to database *administrator*. You learned to manage access with **roles and privileges**, ensuring each team member sees only what they need. You configured **authentication** through `pg_hba.conf` to control who can connect and how. You built a **backup and restore** strategy so TrailShop can survive any disaster. You confronted **SQL injection** — one of the most persistent threats in web security — and learned that **parameterized queries** are the primary defense.

Then you expanded your horizons beyond the relational model. You explored four families of **NoSQL databases** — document, key-value, column-family, and graph — each optimized for different workloads. You saw how PostgreSQL's **JSONB** type lets you combine relational reliability with document flexibility, often eliminating the need for a separate NoSQL database. And you developed a framework for deciding when to use each approach.

TrailShop's database is now secure, backed up, and ready for whatever comes next. **This week concludes the TrailShop course project** — submit your final project package as described in the Week 48 Exercises.

Weeks 49–51 are reserved for the **final exam**. Those weeks provide optional theory and practice materials for review; no new project work or submissions are required.
