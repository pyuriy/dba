To ensure that a sequential `id` column (e.g., primary key or auto-increment) is **not broken** and to **find missing numbers** in SQL, you can use a few techniques depending on your database system. Here's a general approach that works across most SQL dialects:

How to install and use SQLite is described [here](./SQLite-lab.md)

---

## 🧮 1. Detect Missing IDs Using a Sequence Generator

You can generate a full range of expected IDs and compare it to the actual ones.

### ✅ Example using a Common Table Expression (CTE):

```sql
WITH expected_ids AS (
    SELECT generate_series(
        (SELECT MIN(id) FROM your_table),
        (SELECT MAX(id) FROM your_table)
    ) AS id
)
SELECT e.id
FROM expected_ids e
LEFT JOIN your_table t ON e.id = t.id
WHERE t.id IS NULL;
```

- `generate_series()` is available in **PostgreSQL**.
- This returns all `id`s that **should exist** but are **missing**.

---

## 🧮 2. Alternative for MySQL (no `generate_series`)

You can simulate a sequence using a helper table or recursive CTE (MySQL 8+):

```sql
WITH RECURSIVE expected_ids AS (
    SELECT MIN(id) AS id FROM your_table
    UNION ALL
    SELECT id + 1 FROM expected_ids
    WHERE id < (SELECT MAX(id) FROM your_table)
)
SELECT e.id
FROM expected_ids e
LEFT JOIN your_table t ON e.id = t.id
WHERE t.id IS NULL;
```

- This works in **MySQL 8+**, **SQL Server**, and **SQLite** (with slight syntax tweaks).
- It builds a list of expected IDs and finds gaps.

---

## 🧮 3. Check for Gaps Between Consecutive Rows

If you just want to detect where gaps occur:

```sql
SELECT t1.id + 1 AS missing_id
FROM your_table t1
LEFT JOIN your_table t2 ON t1.id + 1 = t2.id
WHERE t2.id IS NULL
ORDER BY t1.id;
```

- This finds **single missing IDs** between existing rows.
- Works in most SQL dialects.

---

## 🧮 4. Count and Compare

To verify if the sequence is complete:

```sql
SELECT COUNT(*) AS actual_count,
       MAX(id) - MIN(id) + 1 AS expected_count
FROM your_table;
```

- If `actual_count < expected_count`, you have missing IDs.

---

Here's a **comprehensive SQLite lab** to help you practice detecting missing sequential IDs using recursive CTEs and related techniques. This lab is designed to reinforce your understanding of SQL logic, recursion, and data integrity checks.

---

## 🧪 SQLite Lab: Detecting Missing Sequential IDs

### 🧰 Prerequisites
- SQLite 3.8.3+ (supports recursive CTEs)
- SQLite CLI or DB browser (e.g., DB Browser for SQLite)
- Basic familiarity with SQL syntax and SELECT statements

---

### 🧩 Step 1: Create a Sample Table

```sql
DROP TABLE IF EXISTS people;

CREATE TABLE people (
    id INTEGER PRIMARY KEY,
    name TEXT
);
```

---

### 🧩 Step 2: Insert Sample Data with Gaps

```sql
INSERT INTO people (id, name) VALUES
(1, 'Alice'),
(2, 'Bob'),
(4, 'Charlie'),
(5, 'Diana'),
(7, 'Eve');
```

- Notice that IDs 3 and 6 are missing.

---

### 🧩 Step 3: View the Table

```sql
SELECT * FROM people ORDER BY id;
```

Expected output:
```
1 | Alice
2 | Bob
4 | Charlie
5 | Diana
7 | Eve
```

---

### 🧩 Step 4: Use Recursive CTE to Generate Expected ID Range

```sql
WITH RECURSIVE expected_ids(id) AS (
    SELECT MIN(id) FROM people
    UNION ALL
    SELECT id + 1 FROM expected_ids
    WHERE id < (SELECT MAX(id) FROM people)
)
SELECT expected_ids.id
FROM expected_ids
LEFT JOIN people ON expected_ids.id = people.id
WHERE people.id IS NULL;
```

Expected output:
```
3
6
```

---

### 🧩 Step 5: Count Actual vs Expected Rows

```sql
SELECT 
    COUNT(*) AS actual_count,
    MAX(id) - MIN(id) + 1 AS expected_count
FROM people;
```

This helps you verify if the sequence is complete.

---

### 🧩 Step 6: Find Gaps Between Consecutive IDs

```sql
SELECT p1.id + 1 AS missing_id
FROM people p1
LEFT JOIN people p2 ON p1.id + 1 = p2.id
WHERE p2.id IS NULL
ORDER BY p1.id;
```

This detects single-step gaps (e.g., missing 3 and 6).

---

### 🧩 Step 7: Bonus — Fill the Gaps Automatically (Optional)

If you want to insert placeholder rows for missing IDs:

```sql
WITH RECURSIVE expected_ids(id) AS (
    SELECT MIN(id) FROM people
    UNION ALL
    SELECT id + 1 FROM expected_ids
    WHERE id < (SELECT MAX(id) FROM people)
)
INSERT INTO people (id, name)
SELECT id, 'Missing'
FROM expected_ids
WHERE id NOT IN (SELECT id FROM people);
```

Then re-run:
```sql
SELECT * FROM people ORDER BY id;
```

---

### 🧩 Step 8: Clean Up (Optional)

```sql
DELETE FROM people WHERE name = 'Missing';
```

---

## 🧠 What You’ll Learn
- Recursive CTEs for sequence generation
- LEFT JOIN logic for gap detection
- Data integrity checks using counts and comparisons
- Optional automation for gap filling
