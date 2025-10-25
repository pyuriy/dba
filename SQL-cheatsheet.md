# SQL Cheatsheet

This is a comprehensive reference for SQL (Structured Query Language), focusing on standard ANSI SQL with notes on common dialects (e.g., MySQL, PostgreSQL, SQLite). Syntax is shown in `backticks`. Examples assume a sample table `employees` with columns: `id` (INT), `name` (VARCHAR), `department` (VARCHAR), `salary` (DECIMAL), `hire_date` (DATE).

## 1. Data Types
| Category | Type | Description | Example |
|----------|------|-------------|---------|
| **Numeric** | `INT` / `INTEGER` | Whole numbers | `INT` |
| | `DECIMAL(p,s)` / `NUMERIC(p,s)` | Fixed-point decimals (precision p, scale s) | `DECIMAL(10,2)` |
| | `FLOAT` / `REAL` | Floating-point | `FLOAT` |
| **String** | `CHAR(n)` | Fixed-length string (n chars) | `CHAR(10)` |
| | `VARCHAR(n)` | Variable-length string (up to n chars) | `VARCHAR(255)` |
| | `TEXT` | Large text (unlimited in most DBs) | `TEXT` |
| **Date/Time** | `DATE` | YYYY-MM-DD | `2023-01-15` |
| | `TIME` | HH:MM:SS | `09:30:00` |
| | `TIMESTAMP` / `DATETIME` | YYYY-MM-DD HH:MM:SS | `2023-01-15 09:30:00` |
| **Boolean** | `BOOLEAN` | True/False | `TRUE` |
| **Other** | `BLOB` | Binary large object | N/A |

## 2. DDL (Data Definition Language)
### CREATE
- **Database**: `CREATE DATABASE db_name;`
- **Table**: 
  ```sql
  CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE DEFAULT CURRENT_DATE
  );
  ```
- **Index**: `CREATE INDEX idx_dept ON employees(department);`
- **View**: `CREATE VIEW high_earners AS SELECT * FROM employees WHERE salary > 50000;`

### ALTER
- **Add Column**: `ALTER TABLE employees ADD COLUMN email VARCHAR(100);`
- **Modify Column**: `ALTER TABLE employees MODIFY COLUMN salary DECIMAL(12,2);`
- **Drop Column**: `ALTER TABLE employees DROP COLUMN email;`
- **Add Constraint**: `ALTER TABLE employees ADD CONSTRAINT chk_salary CHECK (salary > 0);`

### DROP
- **Database**: `DROP DATABASE db_name;`
- **Table**: `DROP TABLE employees;`
- **Index**: `DROP INDEX idx_dept;`
- **View**: `DROP VIEW high_earners;`

## 3. DML (Data Manipulation Language)
### SELECT
- **Basic Query**: `SELECT * FROM employees;`
- **Specific Columns**: `SELECT name, salary FROM employees;`
- **Distinct**: `SELECT DISTINCT department FROM employees;`
- **Limit**: `SELECT * FROM employees LIMIT 10;` (MySQL/PostgreSQL; SQLite uses `LIMIT`)

#### Clauses
- **WHERE**: Filter rows
  ```sql
  SELECT * FROM employees WHERE department = 'IT' AND salary > 50000;
  ```
  - Operators: `=`, `!=`/`<>`, `>`, `<`, `>=`, `<=`, `IN (val1, val2)`, `LIKE '%pattern%'`, `BETWEEN 10000 AND 50000`, `IS NULL`
- **ORDER BY**: Sort
  ```sql
  SELECT * FROM employees ORDER BY salary DESC, name ASC;
  ```
- **GROUP BY**: Aggregate groups
  ```sql
  SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department;
  ```
- **HAVING**: Filter groups (post-GROUP BY)
  ```sql
  SELECT department, COUNT(*) FROM employees GROUP BY department HAVING COUNT(*) > 5;
  ```

### INSERT
- **Single Row**: 
  ```sql
  INSERT INTO employees (name, department, salary) VALUES ('Alice', 'HR', 60000);
  ```
- **Multiple Rows**: 
  ```sql
  INSERT INTO employees (name, department, salary) VALUES 
  ('Bob', 'IT', 70000), ('Carol', 'Sales', 55000);
  ```
- **From Query**: `INSERT INTO new_table SELECT * FROM employees;`

### UPDATE
```sql
UPDATE employees 
SET salary = salary * 1.1 
WHERE department = 'IT';
```

### DELETE
- **All Rows**: `DELETE FROM employees;`
- **Filtered**: `DELETE FROM employees WHERE salary < 30000;`

## 4. Joins
| Join Type | Description | Syntax Example |
|-----------|-------------|----------------|
| **INNER JOIN** | Matching rows only | `SELECT e.name, d.name FROM employees e INNER JOIN departments d ON e.dept_id = d.id;` |
| **LEFT JOIN** (LEFT OUTER) | All left + matching right | `... LEFT JOIN departments d ON ...` |
| **RIGHT JOIN** (RIGHT OUTER) | All right + matching left | `... RIGHT JOIN departments d ON ...` |
| **FULL OUTER JOIN** | All rows from both (not in MySQL) | `... FULL OUTER JOIN departments d ON ...` |
| **CROSS JOIN** | Cartesian product | `SELECT * FROM employees CROSS JOIN departments;` |
| **SELF JOIN** | Join table to itself | `SELECT e1.name, e2.name FROM employees e1 JOIN employees e2 ON e1.manager_id = e2.id;` |

- Aliases: Use `AS` or shorthand (e.g., `employees e`).

## 5. Subqueries
- **Scalar (Single Value)**: `SELECT name FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);`
- **In WHERE**: `SELECT * FROM employees WHERE department IN (SELECT name FROM departments WHERE location = 'NY');`
- **Correlated**: References outer query
  ```sql
  SELECT name FROM employees e 
  WHERE salary > (SELECT AVG(salary) FROM employees WHERE department = e.department);
  ```

## 6. Functions
### Aggregate Functions
| Function | Description | Example |
|----------|-------------|---------|
| `COUNT(*)` / `COUNT(column)` | Row/ non-null count | `SELECT COUNT(*) FROM employees;` |
| `SUM(column)` | Total | `SELECT SUM(salary) FROM employees;` |
| `AVG(column)` | Average | `SELECT AVG(salary) FROM employees;` |
| `MIN(column)` / `MAX(column)` | Min/Max | `SELECT MIN(hire_date) FROM employees;` |

### String Functions
| Function | Description | Example |
|----------|-------------|---------|
| `CONCAT(str1, str2)` | Concatenate | `SELECT CONCAT(name, ' - ', department) FROM employees;` |
| `UPPER(str)` / `LOWER(str)` | Case change | `SELECT UPPER(name) FROM employees;` |
| `SUBSTRING(str, start, len)` | Extract substring | `SELECT SUBSTRING(name, 1, 3) FROM employees;` |
| `LENGTH(str)` | Length | `SELECT LENGTH(name) FROM employees;` |
| `TRIM(str)` | Remove leading/trailing spaces | `SELECT TRIM('  hello  ');` |

### Date/Time Functions
| Function | Description | Example (PostgreSQL/MySQL) |
|----------|-------------|----------------------------|
| `CURRENT_DATE` / `NOW()` | Current date/time | `SELECT CURRENT_DATE;` |
| `DATE_ADD(date, INTERVAL 1 YEAR)` | Add interval (MySQL) | `SELECT DATE_ADD(hire_date, INTERVAL 1 YEAR);` |
| `EXTRACT(YEAR FROM date)` | Extract part (PostgreSQL) | `SELECT EXTRACT(YEAR FROM hire_date);` |
| `DATEDIFF(date1, date2)` | Days between (MySQL) | `SELECT DATEDIFF(NOW(), hire_date);` |

### Conditional
- **CASE**: 
  ```sql
  SELECT name, 
    CASE 
      WHEN salary > 70000 THEN 'High'
      WHEN salary > 50000 THEN 'Medium'
      ELSE 'Low'
    END AS level
  FROM employees;
  ```
- **COALESCE(val1, val2, ...)`: First non-null | `SELECT COALESCE(email, 'No email') FROM employees;`

## 7. Constraints
- **PRIMARY KEY**: Unique identifier, not null.
- **FOREIGN KEY**: References another table's PK: `FOREIGN KEY (dept_id) REFERENCES departments(id)`.
- **UNIQUE**: No duplicates.
- **NOT NULL**: Required value.
- **CHECK**: Custom rule, e.g., `CHECK (salary > 0)`.
- **DEFAULT**: Default value.

## 8. Transactions (TCL)
```sql
BEGIN TRANSACTION;  -- Or START TRANSACTION;
UPDATE employees SET salary = 60000 WHERE id = 1;
-- If error, ROLLBACK;
COMMIT;  -- Save changes
ROLLBACK;  -- Undo
SAVEPOINT sp1;  -- Partial rollback: ROLLBACK TO sp1;
```

## 9. DCL (Data Control Language)
- **GRANT**: `GRANT SELECT, INSERT ON employees TO user;`
- **REVOKE**: `REVOKE SELECT ON employees FROM user;`

## 10. Advanced
### Indexes
- **Create**: `CREATE UNIQUE INDEX idx_name ON employees(name);`
- **Types**: B-tree (default), Hash (exact matches).

### Views
- Updatable if simple: `CREATE VIEW dept_summary AS SELECT department, AVG(salary) FROM employees GROUP BY department;`
- Query: `SELECT * FROM dept_summary;`

### Window Functions (Analytic)
```sql
SELECT name, salary, 
  ROW_NUMBER() OVER (ORDER BY salary DESC) AS rank,
  AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
```
- Common: `ROW_NUMBER()`, `RANK()`, `LAG()`, `LEAD()`.

### Common Errors/Tips
- Semicolon (`;`) ends statements.
- Case-insensitive keywords, but case-sensitive identifiers in some DBs.
- Use `EXPLAIN` before queries for optimization: `EXPLAIN SELECT * FROM employees;`.
- Escaping: Use single quotes for strings: `'O\'Reilly'`.

For dialect-specific features (e.g., MySQL's `LIMIT` vs. PostgreSQL's `FETCH`), check your DBMS docs. Practice on tools like DB-Fiddle or SQLite Online.
