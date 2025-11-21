# Comprehensive PostgreSQL Cheat Sheet

This cheat sheet covers essential PostgreSQL commands, syntax, and examples, organized by category. It's compiled from reliable sources for quick reference. Use `psql` for interactive sessions.

## 1. Connecting and psql Meta-Commands

### Connection
- Connect to server: `psql -U username -h host -d database`
- Connect to database: `\c database_name`
- Exit: `\q`
- Show connection info: `\conninfo`
- Change password: `\password [username]`

### Listing and Inspection
| Command | Description | Example |
|---------|-------------|---------|
| `\l` or `\l+` | List databases | `\l` |
| `\dn` or `\dn+` | List schemas | `\dn` |
| `\dt` or `\dt schema.*` | List tables | `\dt public.*` |
| `\d table_name` or `\d+ table_name` | Describe table | `\d users` |
| `\du` or `\du+` | List users/roles | `\du` |
| `\df` or `\df schema` | List functions | `\df public` |
| `\dp` | List table privileges | `\dp` |
| `\?` | List all psql commands | |
| `\h` | Help on SQL commands | `\h SELECT` |

### Output and Editing
- Toggle expanded output: `\x [on\|off]`
- Toggle timing: `\timing [on\|off]`
- Edit query buffer: `\e [file]`
- Show query: `\p`
- Reset buffer: `\r`
- Save history: `\s [file]`
- Output to file: `\o file`
- Execute file: `\i file`
- Copy table to/from file: `\copy table FROM 'file.csv' CSV HEADER`

### Variables and Misc
- Set variable: `\set name value`
- Unset variable: `\unset name`
- Change directory: `\cd [dir]`
- Shell command: `\! command`

## 2. Data Types

| Category | Types | Examples |
|----------|-------|----------|
| Numeric | `smallint`, `integer`, `bigint`, `decimal(p,s)`, `numeric(p,s)`, `real`, `double precision`, `serial`, `bigserial` | `id SERIAL PRIMARY KEY` |
| Character | `char(n)`, `varchar(n)`, `text` | `name VARCHAR(100)` |
| Boolean | `boolean` | `active BOOLEAN DEFAULT true` |
| Date/Time | `date`, `time`, `timestamp`, `timestamptz`, `interval` | `created_at TIMESTAMP DEFAULT NOW()` |
| Binary | `bytea` | |
| JSON | `json`, `jsonb` | `data JSONB` |
| Network | `cidr`, `inet`, `macaddr` | |
| Geometric | `point`, `line`, `box` | |
| UUID | `uuid` | `id UUID DEFAULT gen_random_uuid()` |
| Bit String | `bit(n)`, `bit varying(n)` | |
| Money | `money` | |

## 3. DDL (Data Definition Language)

### Databases
- Create: `CREATE DATABASE db_name [WITH OWNER user_name];`
- Drop: `DROP DATABASE IF EXISTS db_name;`
- Rename: `ALTER DATABASE old_name RENAME TO new_name;`
- List: `\l`

### Tables
- Create basic:  
  ```sql
  CREATE TABLE table_name (
    col1 data_type [constraints],
    col2 data_type
  );
  ```
  Example:  
  ```sql
  CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE
  );
  ```
- Create with foreign key:  
  ```sql
  CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id)
  );
  ```
- Drop: `DROP TABLE IF EXISTS table_name CASCADE;`
- Truncate: `TRUNCATE TABLE table_name;`

### ALTER TABLE
| Operation | Syntax/Example |
|-----------|----------------|
| Add column | `ALTER TABLE table ADD COLUMN col data_type;` |
| Drop column | `ALTER TABLE table DROP COLUMN col;` |
| Rename column | `ALTER TABLE table RENAME COLUMN old TO new;` |
| Change type | `ALTER TABLE table ALTER COLUMN col TYPE new_type;` |
| Set/Drop default | `ALTER TABLE table ALTER COLUMN col SET DEFAULT value;` <br> `ALTER TABLE table ALTER COLUMN col DROP DEFAULT;` |
| Set/Drop NOT NULL | `ALTER TABLE table ALTER COLUMN col SET NOT NULL;` <br> `ALTER TABLE table ALTER COLUMN col DROP NOT NULL;` |
| Add constraint | `ALTER TABLE table ADD CONSTRAINT chk CHECK (col > 0);` |
| Rename table | `ALTER TABLE old RENAME TO new;` |

### Schemas
- Create: `CREATE SCHEMA IF NOT EXISTS schema_name;`
- Drop: `DROP SCHEMA IF EXISTS schema_name CASCADE;`

### Users/Roles
- Create user: `CREATE USER username WITH PASSWORD 'password';`
- Drop user: `DROP USER IF EXISTS username;`
- Alter password: `ALTER ROLE username WITH PASSWORD 'new_password';`
- List: `\du` or `SELECT rolname FROM pg_roles;`

### Permissions (GRANT/REVOKE)
- Grant on database: `GRANT ALL PRIVILEGES ON DATABASE db TO user;`
- Grant connect: `GRANT CONNECT ON DATABASE db TO user;`
- Grant on schema: `GRANT USAGE ON SCHEMA schema TO user;`
- Grant on all tables: `GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO user;`
- Grant on specific table: `GRANT SELECT ON table TO user;`
- Grant on functions: `GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO user;`
- Revoke: Replace `GRANT` with `REVOKE`.

### Indexes and Constraints
- Create index: `CREATE INDEX idx_name ON table (col);`
- Unique constraint: `ALTER TABLE table ADD UNIQUE (col);`
- Check constraint: `ALTER TABLE table ADD CHECK (col > 0);`
- Primary key: `ALTER TABLE table ADD PRIMARY KEY (col);`

## 4. DML (Data Manipulation Language)

### INSERT
- All columns: `INSERT INTO table VALUES (val1, val2);`
- Specific columns: `INSERT INTO table (col1, col2) VALUES (val1, val2);`
- With default (e.g., SERIAL): `INSERT INTO table (col) VALUES (DEFAULT, 'value');`
- Multiple rows: `INSERT INTO table (col) VALUES ('a'), ('b');`

### SELECT
- Basic: `SELECT col1, col2 FROM table;`
- All: `SELECT * FROM table;`
- Limit: `SELECT * FROM table LIMIT 5 OFFSET 2;`
- Distinct: `SELECT DISTINCT col FROM table;`

#### WHERE and Filtering
- Condition: `SELECT * FROM table WHERE col = 'value';`
- LIKE: `SELECT * FROM table WHERE col LIKE 'A%';` (starts with A)
- IN: `SELECT * FROM table WHERE col IN (1, 2);`
- BETWEEN: `SELECT * FROM table WHERE col BETWEEN 1 AND 10;`
- IS NULL: `SELECT * FROM table WHERE col IS NULL;`
- Operators: `=`, `!=`, `>`, `<`, `AND`, `OR`, `NOT`

### UPDATE
- Basic: `UPDATE table SET col1 = val1 WHERE condition;`
Example: `UPDATE users SET email = 'new@example.com' WHERE id = 1;`

### DELETE
- All: `DELETE FROM table;`
- Specific: `DELETE FROM table WHERE condition;`

## 5. Advanced Queries

### JOINs
```sql
SELECT t1.col, t2.col
FROM table1 t1
[INNER|LEFT|RIGHT|FULL] JOIN table2 t2 ON t1.id = t2.id;
```

### Aggregates and GROUP BY
| Function | Description | Example |
|----------|-------------|---------|
| `COUNT(*)` | Row count | `SELECT COUNT(*) FROM table;` |
| `COUNT(DISTINCT col)` | Unique count | `SELECT COUNT(DISTINCT col) FROM table;` |
| `SUM(col)` | Sum | `SELECT SUM(salary) FROM employees;` |
| `AVG(col)` | Average | `SELECT AVG(age) FROM users GROUP BY species;` |
| `MIN(col)`, `MAX(col)` | Min/Max | `SELECT MIN(age), MAX(age) FROM users;` |

- GROUP BY: `SELECT col, AVG(val) FROM table GROUP BY col HAVING AVG(val) > 10 ORDER BY AVG(val) DESC;`

### Subqueries and Window Functions
- Subquery: `SELECT * FROM table WHERE id IN (SELECT id FROM other WHERE cond);`
- Window (e.g., ROW_NUMBER): `SELECT ROW_NUMBER() OVER (PARTITION BY col ORDER BY val) FROM table;`

## 6. Functions

### Text Functions
| Function | Description | Example |
|----------|-------------|---------|
| `LENGTH(str)` | Length | `LENGTH('hello')` → 5 |
| `LOWER(str)`, `UPPER(str)` | Case | `LOWER('Hi')` → 'hi' |
| `INITCAP(str)` | Title case | `INITCAP('hello world')` → 'Hello World' |
| `CONCAT(str1, str2)` | Concat (ignores NULL) | `CONCAT('Hi', ' ', 'there')` → 'Hi there' |
| `str1 || str2` | Concat (NULL propagates) | `'Hi' || ' there'` → 'Hi there' |
| `SUBSTRING(str, start, len)` | Substring | `SUBSTRING('hello', 1, 3)` → 'hel' |
| `REPLACE(str, old, new)` | Replace | `REPLACE('hi hi', 'hi', 'bye')` → 'bye bye' |

### Numeric Functions
| Function | Description | Example |
|----------|-------------|---------|
| `ROUND(num, decimals)` | Round | `ROUND(3.14159, 2)` → 3.14 |
| `ABS(num)` | Absolute | `ABS(-5)` → 5 |
| `MOD(a, b)` or `a % b` | Modulo | `13 % 5` → 3 |
| `SQRT(num)` | Square root | `SQRT(16)` → 4 |
| `POWER(base, exp)` | Power | `POWER(2, 3)` → 8 |

### Date/Time Functions
- Current: `CURRENT_DATE`, `CURRENT_TIME`, `CURRENT_TIMESTAMP`
- Cast: `'2023-01-01'::DATE`
- Interval: `INTERVAL '1 day'`
- Add: `'2023-01-01'::DATE + INTERVAL '1 month'`
- Extract: `EXTRACT(YEAR FROM '2023-01-01'::DATE)` → 2023
- Date diff: `'2023-01-02'::DATE - '2023-01-01'::DATE` → 1
- Truncate: `DATE_TRUNC('month', '2023-01-15'::DATE)` → '2023-01-01'

### NULL Handling
- COALESCE: `COALESCE(col, 'default')` – First non-NULL
- NULLIF(a, b): `NULLIF(col, 0)` – NULL if equal

## 7. Transactions (TCL)
- Begin: `BEGIN;`
- Commit: `COMMIT;`
- Rollback: `ROLLBACK;`
- Savepoint: `SAVEPOINT sp1; ROLLBACK TO sp1;`

## 8. Backup and Restore
### pg_dump (Backup)
- All DBs: `pg_dumpall -U user > all.sql`
- Single DB: `pg_dump -U user -d db > db.sql`
- Schema only: `pg_dump -s db > schema.sql`
- Data only: `pg_dump -a db > data.sql`

### Restore
- psql: `psql -U user db < db.sql`
- pg_restore: `pg_restore -U user -d db -c db.dump`

### CSV
- Export: `\copy (SELECT * FROM table) TO 'file.csv' CSV HEADER`
- Import: `\copy table FROM 'file.csv' CSV HEADER`

## 9. System Queries
- Server version: `SHOW SERVER_VERSION;`
- Current user: `SELECT current_user;`
- Current DB: `SELECT current_database();`
- Config: `SHOW config_file;`
- All settings: `SHOW ALL;`

## 10. Configuration Files
- `postgresql.conf`: Server settings (e.g., `listen_addresses = '*'`).
- `pg_hba.conf`: Authentication (e.g., `host all all 0.0.0.0/0 md5`).
- Restart: `sudo systemctl restart postgresql`

For more, refer to official docs. Examples tested on PostgreSQL 17 (as of 2025).
