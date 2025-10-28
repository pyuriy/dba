# Comprehensive SQL Lab: Mastering SQL with SQLite

Welcome to this hands-on lab for learning SQL using SQLite! SQLite is a lightweight, serverless database engine that's perfect for learning and prototyping. This lab builds on common SQL concepts but uses a realistic company database schema.

The lab is divided into sections: **Setup**, **Basic Queries**, **Intermediate Queries (Joins & Aggregations)**, **Advanced Queries**, and **Exercises with Solutions**. Each section includes SQL snippets you can run directly.

## Prerequisites
- Python 3.x (with `sqlite3` module, which is built-in).
- A text editor or IDE (e.g., VS Code).
- Optional: SQLite browser like DB Browser for SQLite for a GUI view.

## Setup: Creating the Sample Database
We'll create a company database with tables for `departments`, `employees`, `projects`, and a junction table `employee_projects` (for many-to-many relationships).

1. Create a file named `setup_db.py` and paste the following code:

```python
import sqlite3

# Connect to (or create) the database file
conn = sqlite3.connect('company.db')
c = conn.cursor()

# Drop tables if they exist (for clean restarts)
c.execute('DROP TABLE IF EXISTS employee_projects')
c.execute('DROP TABLE IF EXISTS projects')
c.execute('DROP TABLE IF EXISTS employees')
c.execute('DROP TABLE IF EXISTS departments')

# Create departments table
c.execute('''
CREATE TABLE departments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE
)
''')

# Insert departments
c.executemany('INSERT INTO departments (name) VALUES (?)', [('HR',), ('IT',), ('Sales',), ('Finance',)])

# Create employees table
c.execute('''
CREATE TABLE employees (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    department_id INTEGER NOT NULL,
    salary REAL CHECK (salary > 0),
    hire_date DATE,
    FOREIGN KEY (department_id) REFERENCES departments (id)
)
''')

# Insert employees
employees_data = [
    ('Alice Johnson', 1, 55000.0, '2020-01-15'),
    ('Bob Smith', 2, 65000.0, '2019-03-20'),
    ('Carol Davis', 2, 70000.0, '2018-07-10'),
    ('David Wilson', 3, 50000.0, '2021-11-05'),
    ('Eva Brown', 1, 60000.0, '2017-05-12'),
    ('Frank Miller', 4, 75000.0, '2016-09-18'),
    ('Grace Lee', 3, 52000.0, '2022-02-28'),
    ('Henry Taylor', 2, 68000.0, '2020-12-01')
]
c.executemany('INSERT INTO employees (name, department_id, salary, hire_date) VALUES (?, ?, ?, ?)', employees_data)

# Create projects table
c.execute('''
CREATE TABLE projects (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    department_id INTEGER NOT NULL,
    budget REAL CHECK (budget > 0),
    FOREIGN KEY (department_id) REFERENCES departments (id)
)
''')

# Insert projects
projects_data = [
    ('Payroll System', 1, 100000.0),
    ('Network Upgrade', 2, 150000.0),
    ('Sales CRM', 3, 80000.0),
    ('Financial Audit', 4, 120000.0)
]
c.executemany('INSERT INTO projects (name, department_id, budget) VALUES (?, ?, ?)', projects_data)

# Create employee_projects junction table
c.execute('''
CREATE TABLE employee_projects (
    employee_id INTEGER NOT NULL,
    project_id INTEGER NOT NULL,
    role TEXT,
    PRIMARY KEY (employee_id, project_id),
    FOREIGN KEY (employee_id) REFERENCES employees (id) ON DELETE CASCADE,
    FOREIGN KEY (project_id) REFERENCES projects (id) ON DELETE CASCADE
)
''')

# Insert assignments
assignments_data = [
    (1, 1, 'Manager'),      # Alice on Payroll
    (2, 2, 'Developer'),    # Bob on Network
    (3, 2, 'Lead Developer'), # Carol on Network
    (4, 3, 'Sales Rep'),    # David on Sales CRM
    (5, 1, 'HR Specialist'), # Eva on Payroll
    (6, 4, 'Auditor'),      # Frank on Financial Audit
    (7, 3, 'Sales Rep'),    # Grace on Sales CRM
    (2, 3, 'Consultant')    # Bob on Sales CRM
]
c.executemany('INSERT INTO employee_projects (employee_id, project_id, role) VALUES (?, ?, ?)', assignments_data)

conn.commit()
conn.close()

print("Database 'company.db' created successfully!")
print("Tables: departments, employees, projects, employee_projects")
```

2. Run the script: `python setup_db.py`.

3. To query the DB in Python, create a file `query.py` with:
```python
import sqlite3

conn = sqlite3.connect('company.db')
c = conn.cursor()

# Example query
for row in c.execute('SELECT * FROM employees LIMIT 3'):
    print(row)

conn.close()
```
Run it to verify.

Alternatively, use the SQLite CLI: `sqlite3 company.db` and run `.tables` or `SELECT * FROM employees;`.

## Section 1: Basic Queries
Focus: SELECT, WHERE, ORDER BY, LIMIT.

### 1.1 View All Employees
```sql
SELECT * FROM employees;
```
**Expected Output** (abbreviated):
| id | name          | department_id | salary  | hire_date   |
|----|---------------|---------------|---------|-------------|
| 1  | Alice Johnson | 1             | 55000.0 | 2020-01-15  |
| 2  | Bob Smith     | 2             | 65000.0 | 2019-03-20  |
... (8 rows total)

### 1.2 Select Specific Columns
```sql
SELECT name, salary FROM employees;
```

### 1.3 Filter with WHERE
- Employees in IT (dept_id=2):
```sql
SELECT name, salary FROM employees WHERE department_id = 2;
```
- Employees hired after 2020:
```sql
SELECT * FROM employees WHERE hire_date > '2020-12-31';
```
- Salaries between 55k and 65k:
```sql
SELECT name FROM employees WHERE salary BETWEEN 55000 AND 65000;
```

### 1.4 Sort and Limit
- Top 3 highest-paid employees:
```sql
SELECT name, salary FROM employees ORDER BY salary DESC LIMIT 3;
```
**Output**:
| name         | salary  |
|--------------|---------|
| Frank Miller | 75000.0 |
| Carol Davis  | 70000.0 |
| Henry Taylor | 68000.0 |

## Section 2: Intermediate Queries (Joins & Aggregations)
Focus: JOIN, GROUP BY, HAVING, Aggregate Functions.

### 2.1 Simple INNER JOIN
- Employee names with department names:
```sql
SELECT e.name, d.name AS department 
FROM employees e 
INNER JOIN departments d ON e.department_id = d.id;
```

### 2.2 LEFT JOIN
- All departments and their employees (shows empty for depts with no employees):
```sql
SELECT d.name AS department, e.name 
FROM departments d 
LEFT JOIN employees e ON d.id = e.department_id 
ORDER BY d.name;
```

### 2.3 Aggregations with GROUP BY
- Average salary per department:
```sql
SELECT d.name, AVG(e.salary) AS avg_salary, COUNT(e.id) AS emp_count
FROM departments d
INNER JOIN employees e ON d.id = e.department_id
GROUP BY d.id, d.name;
```
**Output**:
| name    | avg_salary | emp_count |
|---------|------------|-----------|
| Finance | 75000.0    | 1         |
| HR      | 57500.0    | 2         |
| IT      | 67666.6667 | 3         |
| Sales   | 51000.0    | 2         |

### 2.4 HAVING Clause
- Departments with more than 1 employee:
```sql
SELECT d.name, COUNT(e.id) AS emp_count
FROM departments d
INNER JOIN employees e ON d.id = e.department_id
GROUP BY d.id
HAVING COUNT(e.id) > 1;
```

### 2.5 Multi-Table Join
- Employees and their project roles:
```sql
SELECT e.name, p.name AS project, ep.role
FROM employees e
INNER JOIN employee_projects ep ON e.id = ep.employee_id
INNER JOIN projects p ON ep.project_id = p.id;
```

## Section 3: Advanced Queries
Focus: Subqueries, CASE, Window Functions, Views, Indexes.

### 3.1 Subquery in WHERE
- Employees earning more than the IT department average:
```sql
SELECT name, salary
FROM employees e
WHERE salary > (
    SELECT AVG(salary) FROM employees WHERE department_id = 2
);
```

### 3.2 Correlated Subquery
- Employees who earn more than the average in their own department:
```sql
SELECT e.name, e.salary, d.name AS dept
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
WHERE e.salary > (
    SELECT AVG(salary) FROM employees e2 WHERE e2.department_id = e.department_id
);
```

### 3.3 CASE Statement
- Categorize salary levels:
```sql
SELECT name, salary,
    CASE
        WHEN salary > 70000 THEN 'High'
        WHEN salary > 60000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_level
FROM employees
ORDER BY salary DESC;
```

### 3.4 Window Functions
- Rank employees by salary within their department:
```sql
SELECT e.name, d.name AS dept, e.salary,
    RANK() OVER (PARTITION BY d.id ORDER BY e.salary DESC) AS dept_rank
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
ORDER BY d.name, e.salary DESC;
```
**Output** (abbreviated):
| name         | dept    | salary  | dept_rank |
|--------------|---------|---------|-----------|
| Eva Brown    | HR      | 60000.0 | 1         |
| Alice Johnson| HR      | 55000.0 | 2         |
| Carol Davis  | IT      | 70000.0 | 1         |
...

### 3.5 Create a View
```sql
CREATE VIEW high_earners AS
SELECT e.name, e.salary, d.name AS dept
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
WHERE e.salary > 60000;

-- Query the view
SELECT * FROM high_earners;
```

### 3.6 Create an Index
```sql
CREATE INDEX idx_salary ON employees(salary);

-- Check with EXPLAIN (for optimization insight)
EXPLAIN QUERY PLAN SELECT * FROM employees WHERE salary > 60000;
```

## Section 4: Exercises
Try these on your own, then check solutions. Use `company.db`.

### Exercise 1: Basic (Easy)
List all employees hired in 2020 or later, sorted by hire_date ascending. Show name, hire_date, and department name.

### Exercise 2: Joins (Medium)
Find the total budget per department, including only departments with projects. Show department name and total budget.

### Exercise 3: Aggregations (Medium)
Which project has the most employees assigned? Show project name and count.

### Exercise 4: Subquery (Hard)
List employees who are not assigned to any project, with their department.

### Exercise 5: Window Function (Hard)
For each department, find the employee with the second-highest salary. Show name, salary, dept.

### Solutions
Run these in your SQLite session to verify.

**Ex 1:**
```sql
SELECT e.name, e.hire_date, d.name AS dept
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
WHERE e.hire_date >= '2020-01-01'
ORDER BY e.hire_date ASC;
```
**Output:** Alice, Henry, David, Grace (sorted by date).

**Ex 2:**
```sql
SELECT d.name, SUM(p.budget) AS total_budget
FROM departments d
INNER JOIN projects p ON d.id = p.department_id
GROUP BY d.id, d.name;
```
**Output:** HR:100000, IT:150000, Sales:80000, Finance:120000.

**Ex 3:**
```sql
SELECT p.name, COUNT(ep.employee_id) AS emp_count
FROM projects p
INNER JOIN employee_projects ep ON p.id = ep.project_id
GROUP BY p.id, p.name
ORDER BY emp_count DESC
LIMIT 1;
```
**Output:** Network Upgrade (2 employees; note: Sales CRM also 2, but this picks first).

**Ex 4:**
```sql
SELECT e.name, d.name AS dept
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
WHERE e.id NOT IN (
    SELECT DISTINCT employee_id FROM employee_projects
);
```
**Output:** Henry Taylor (IT).

**Ex 5:**
```sql
WITH ranked_emps AS (
    SELECT e.name, e.salary, d.name AS dept,
        ROW_NUMBER() OVER (PARTITION BY d.id ORDER BY e.salary DESC) AS rn
    FROM employees e
    INNER JOIN departments d ON e.department_id = d.id
)
SELECT name, salary, dept
FROM ranked_emps
WHERE rn = 2;
```
**Output:** Alice Johnson (55000, HR); Henry Taylor (68000, IT); David Wilson (50000, Sales). (Finance has only 1, so none.)

## Next Steps
- Add more data: INSERT new employees/projects.
- Transactions: Wrap updates in `BEGIN; ... COMMIT;`.
- Backup: Copy `company.db`.
- Explore: Use `PRAGMA table_info(employees);` for schema.

Practice daily! For errors, check SQLite docs or run `sqlite3 company.db ".schema"`. If stuck, modify queries step-by-step. Happy querying! 🚀
