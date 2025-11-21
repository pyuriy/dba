# Comprehensive PostgreSQL Lab

This lab builds on the PostgreSQL cheat sheet, providing hands-on exercises to reinforce key concepts. It's designed for beginners to intermediate users. You'll need PostgreSQL installed (version 16+ recommended; download from [postgresql.org](https://www.postgresql.org/download/)). Use `psql` or a GUI like pgAdmin/DBeaver for execution.

## Prerequisites and Setup
1. **Install PostgreSQL**: Follow the official guide for your OS. Start the server: `sudo systemctl start postgresql` (Linux) or equivalent.
2. **Create a superuser**: Log in as postgres: `sudo -u postgres psql`. Set a password: `ALTER USER postgres PASSWORD 'yourpassword';`.
3. **Create lab database**: In `psql`:
   ```sql
   CREATE DATABASE pg_lab;
   \c pg_lab;
   ```
4. **Enable extensions** (for UUIDs/JSON): 
   ```sql
   CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
   CREATE EXTENSION IF NOT EXISTS "pgcrypto";  -- For password hashing
   ```
5. **Run all SQL in this lab** in the `pg_lab` database. Save scripts to files (e.g., `lab1.sql`) and execute with `\i lab1.sql`.

**Tips**: 
- Use `\x` for expanded output on complex queries.
- Check errors with `\errverbose`.
- Clean up at end: `DROP DATABASE pg_lab;`.

## Lab 1: DDL - Creating and Modifying Tables
**Objectives**: Practice creating databases, tables, constraints, and altering structures.

### Step 1: Create Sample Tables
Run this script to set up a simple e-commerce schema:
```sql
-- Users table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,  -- Use crypt() for real hashing
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE
);

-- Products table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) CHECK (price > 0),
    category VARCHAR(50) DEFAULT 'General'
);

-- Orders table with foreign key
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    product_id INTEGER REFERENCES products(id) ON DELETE SET NULL,
    quantity INTEGER DEFAULT 1 CHECK (quantity > 0),
    order_date DATE DEFAULT CURRENT_DATE
);
```

**Task 1.1**: Describe the tables. Expected: See columns, types, constraints.
```sql
\d users
\d products
\d orders
```

**Task 1.2**: Add a column to `users` for phone number (VARCHAR(20), optional).
```sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
\d users  -- Verify
```

**Task 1.3**: Rename `quantity` in `orders` to `qty` and make it NOT NULL.
```sql
ALTER TABLE orders RENAME COLUMN quantity TO qty;
ALTER TABLE orders ALTER COLUMN qty SET NOT NULL;
\d orders
```

**Task 1.4**: Drop the `phone` column.
```sql
ALTER TABLE users DROP COLUMN phone;
```

**Expected Output (for \d users)**:
```
               Table "public.users"
    Column     |          Type          | Collation | Nullable |               Default
---------------+------------------------+-----------+----------+-----------------------------------
 id            | integer                |           | not null | nextval('users_id_seq'::regclass)
 username      | character varying(50)  |           | not null |
 email         | character varying(100) |           | not null |
 password_hash | text                   |           | not null |
 created_at    | timestamp with time zone |       |          | CURRENT_TIMESTAMP
 is_active     | boolean                |           |          | true
Indexes:
    "users_pkey" PRIMARY KEY, btree (id)
    "users_email_key" UNIQUE CONSTRAINT, btree (email)
    "users_username_key" UNIQUE CONSTRAINT, btree (username)
```

## Lab 2: DML - Inserting, Updating, and Deleting Data
**Objectives**: Manipulate data with INSERT, UPDATE, DELETE.

### Step 2: Populate Data
Insert sample records:
```sql
-- Insert users
INSERT INTO users (username, email, password_hash) VALUES
('alice', 'alice@email.com', crypt('pass123', gen_salt('bf'))),
('bob', 'bob@email.com', crypt('pass456', gen_salt('bf'))),
('charlie', 'charlie@email.com', crypt('pass789', gen_salt('bf')));

-- Insert products
INSERT INTO products (name, price, category) VALUES
('Laptop', 999.99, 'Electronics'),
('Book', 19.99, 'Literature'),
('Coffee Mug', 9.99, 'Home');

-- Insert orders
INSERT INTO orders (user_id, product_id, qty) VALUES
(1, 1, 1),  -- Alice buys Laptop
(2, 3, 2),  -- Bob buys 2 Mugs
(1, 2, 1);  -- Alice buys Book
```

**Task 2.1**: Query all data.
```sql
SELECT * FROM users;
SELECT * FROM products;
SELECT * FROM orders;
```

**Task 2.2**: Update Bob's email and deactivate Charlie.
```sql
UPDATE users SET email = 'bob.new@email.com' WHERE username = 'bob';
UPDATE users SET is_active = FALSE WHERE username = 'charlie';
SELECT * FROM users;  -- Verify
```

**Task 2.3**: Delete orders for inactive users (should delete none yet, but test logic).
```sql
DELETE FROM orders WHERE user_id IN (SELECT id FROM users WHERE is_active = FALSE);
SELECT COUNT(*) FROM orders;  -- Should still be 3
```

**Task 2.4**: Insert a duplicate email (should fail due to UNIQUE).
```sql
-- This will error: INSERT INTO users (username, email, password_hash) VALUES ('dave', 'alice@email.com', 'hash');
-- Fix: Use a unique email.
```

**Expected Output (SELECT * FROM orders)**:
```
 id | user_id | product_id | qty | order_date
----+---------+------------+-----+------------
  1 |       1 |          1 |   1 | 2025-11-21
  2 |       2 |          3 |   2 | 2025-11-21
  3 |       1 |          2 |   1 | 2025-11-21
(3 rows)
```

## Lab 3: Basic Queries and JOINs
**Objectives**: Filter data, join tables, handle NULLs.

**Task 3.1**: Select active users with emails.
```sql
SELECT username, email FROM users WHERE is_active = TRUE;
-- Expected: alice and bob
```

**Task 3.2**: Find orders with product names and total price (qty * price). Use INNER JOIN.
```sql
SELECT o.id, u.username, p.name, (o.qty * p.price) AS total
FROM orders o
INNER JOIN users u ON o.user_id = u.id
INNER JOIN products p ON o.product_id = p.id
ORDER BY total DESC;
```
**Expected**:
```
 id | username |    name     |  total
----+----------+-------------+--------
  1 | alice    | Laptop      | 999.99
  3 | alice    | Book        |  19.99
  2 | bob      | Coffee Mug  |  19.98
```

**Task 3.3**: Left join to show all products, even without orders.
```sql
SELECT p.name, COUNT(o.id) AS order_count
FROM products p
LEFT JOIN orders o ON p.id = o.product_id
GROUP BY p.name;
-- Expected: Laptop:1, Book:1, Coffee Mug:2
```

**Task 3.4**: Filter orders after a date (use future date for none; adjust to CURRENT_DATE - INTERVAL '1 day').
```sql
SELECT * FROM orders WHERE order_date > CURRENT_DATE - INTERVAL '1 day';
```

## Lab 4: Aggregates, GROUP BY, and Functions
**Objectives**: Summarize data, use string/date functions.

**Task 4.1**: Total revenue by category.
```sql
SELECT p.category, SUM(o.qty * p.price) AS revenue
FROM orders o
JOIN products p ON o.product_id = p.id
GROUP BY p.category
HAVING SUM(o.qty * p.price) > 10
ORDER BY revenue DESC;
```
**Expected**:
```
 category    |  revenue
-------------+----------
 Electronics | 999.99
 Home        |  19.98
 Literature  |  19.99
```

**Task 4.2**: Use text functions – Concat username and order count.
```sql
SELECT username || ' has ' || COUNT(o.id)::TEXT || ' orders' AS summary
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE is_active = TRUE
GROUP BY username;
```
**Expected**: 'alice has 2 orders', 'bob has 1 orders'

**Task 4.3**: Date functions – Orders in the last 7 days (simulate with recent dates).
First, update an order date:
```sql
UPDATE orders SET order_date = CURRENT_DATE - INTERVAL '3 days' WHERE id = 1;
```
Then:
```sql
SELECT EXTRACT(DAY FROM (CURRENT_DATE - order_date)) AS days_ago, COUNT(*)
FROM orders
GROUP BY days_ago;
-- Expected: 0:2, 3:1
```

**Task 4.4**: Handle NULLs – Average price, ignoring NULL (but none here).
```sql
SELECT AVG(price) AS avg_price, COALESCE(AVG(price), 0) AS safe_avg FROM products;
```

## Lab 5: Transactions, Indexes, and Performance
**Objectives**: Ensure data integrity, optimize queries.

### Step 5: Transactions
**Task 5.1**: Atomic update – Transfer order from Alice to Bob (rollback test).
```sql
BEGIN;
UPDATE orders SET user_id = 2 WHERE id = 3;  -- Move Alice's book to Bob
SELECT username, COUNT(*) FROM users u JOIN orders o ON u.id = o.user_id GROUP BY username;
-- Check: alice:1, bob:2
ROLLBACK;  -- Revert
-- Verify: SELECT ... (back to alice:2, bob:1)
```

**Task 5.2**: Commit a real change.
```sql
BEGIN;
INSERT INTO orders (user_id, product_id, qty) VALUES (3, 1, 1);  -- Charlie buys Laptop (inactive, but allowed)
COMMIT;
SELECT * FROM orders;  -- Now 4 rows
```

### Indexes
**Task 5.3**: Create index on email, query it.
```sql
CREATE INDEX idx_users_email ON users(email);
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@email.com';  -- Should use index
```

**Task 5.4**: Drop unused index.
```sql
DROP INDEX idx_users_email;
```

## Lab 6: Advanced Features - Views, JSON, Triggers
**Objectives**: Explore views, JSONB, and automation.

### Views
**Task 6.1**: Create a view for active user orders.
```sql
CREATE VIEW active_orders AS
SELECT u.username, p.name, o.qty, o.order_date
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN products p ON o.product_id = p.id
WHERE u.is_active = TRUE;

SELECT * FROM active_orders;
-- Expected: 3 rows (Alice's 2, Bob's 1)
```

### JSONB
**Task 6.2**: Add JSONB column to products for metadata.
```sql
ALTER TABLE products ADD COLUMN metadata JSONB DEFAULT '{}'::JSONB;
UPDATE products SET metadata = '{"stock': 10, 'color': 'black'}'::JSONB WHERE id = 1;
SELECT name, metadata->>'stock' AS stock FROM products WHERE id = 1;
-- Expected: 10
```

### Triggers
**Task 6.3**: Create trigger to auto-set created_at on insert (already default, but example for update).
```sql
CREATE OR REPLACE FUNCTION update_created_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.created_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trig_users_update
BEFORE UPDATE ON users
FOR EACH ROW EXECUTE FUNCTION update_created_at();

UPDATE users SET username = 'alice_updated' WHERE id = 1;
SELECT created_at FROM users WHERE id = 1;  -- Updated timestamp
```

**Cleanup**:
```sql
DROP VIEW active_orders;
DROP TRIGGER trig_users_update ON users;
DROP FUNCTION update_created_at();
ALTER TABLE products DROP COLUMN metadata;
```

## Lab 7: Backup, Restore, and CSV Import/Export
**Objectives**: Manage data persistence.

### Backup
**Task 7.1**: Backup the database.
```bash
# From terminal
pg_dump -U postgres -d pg_lab > pg_lab_backup.sql
# Or schema only: pg_dump -s -U postgres -d pg_lab > schema.sql
```

### CSV Operations
**Task 7.2**: Export users to CSV.
```sql
\copy users TO 'users.csv' CSV HEADER;
```
(Exit psql, check file: `head users.csv`)

**Task 7.3**: Import new data (create temp file first, e.g., echo "dave,dave@email.com,hash,true" > temp_users.csv)
```sql
-- Assume temp_users.csv: username,email,password_hash,is_active (skip id)
\copy users (username, email, password_hash, is_active) FROM 'temp_users.csv' CSV;
SELECT * FROM users;  -- Now 4 active users
```

### Restore Test
**Task 7.4**: Drop and restore (caution: loses data).
```sql
DROP TABLE orders;  -- Test
-- From terminal: psql -U postgres -d pg_lab < pg_lab_backup.sql
-- Then: SELECT * FROM orders;  -- Back!
```

## Conclusion and Challenges
- **Verify**: Run `SELECT COUNT(*) FROM users;` (should be 4+), total orders revenue >1000.
- **Challenges**:
  1. Create a function to calculate user lifetime value (SUM total orders).
     ```sql
     CREATE FUNCTION user_ltv(user_id INT) RETURNS DECIMAL AS $$
     SELECT COALESCE(SUM(o.qty * p.price), 0) FROM orders o JOIN products p ON o.product_id = p.id WHERE o.user_id = $1;
     $$ LANGUAGE SQL;
     SELECT user_ltv(1);  -- Alice: ~1020
     ```
  2. Add FULL TEXT SEARCH index on product names: `CREATE INDEX idx_products_fts ON products USING GIN(to_tsvector('english', name)); SELECT * FROM products WHERE to_tsvector('english', name) @@ to_tsquery('laptop');`
  3. Partition orders by date (advanced): See docs for `PARTITION BY RANGE (order_date)`.

Practice iteratively. For issues, check PostgreSQL logs (`SHOW log_directory;`). Extend with real data imports. Happy querying!
