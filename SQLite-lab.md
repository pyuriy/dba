Here’s a complete guide to installing and initializing **SQLite** on **Ubuntu Linux 22.04** (or similar versions). This will get you set up for your SQL lab work, including recursive CTEs and integrity checks.

---

## 🧰 Step 1: Install SQLite

SQLite is lightweight and typically available via Ubuntu’s package manager.

```bash
sudo apt update
sudo apt install sqlite3
```

To verify installation:

```bash
sqlite3 --version
```

You should see something like `3.37.2` or newer.

---

## 📁 Step 2: Create a Working Directory

Organize your lab files in a dedicated folder:

```bash
mkdir ~/sqlite_lab
cd ~/sqlite_lab
```

---

## 📄 Step 3: Create a New SQLite Database

You can create a new database file by simply invoking `sqlite3` with a filename:

```bash
sqlite3 lab.db
```

This opens the SQLite shell and creates `lab.db` in your current directory.

---

## 🧪 Step 4: Initialize a Table

Inside the SQLite shell:

```sql
CREATE TABLE people (
    id INTEGER PRIMARY KEY,
    name TEXT
);

INSERT INTO people (id, name) VALUES
(1, 'Alice'),
(2, 'Bob'),
(4, 'Charlie'),
(5, 'Diana'),
(7, 'Eve');
```

To view the data:

```sql
SELECT * FROM people;
```

To exit the shell:

```sql
.quit
```

---

## 🧠 Step 5: Run SQL Scripts from File (Optional)

Create a file `init.sql` with your setup:

```sql
-- init.sql
CREATE TABLE people (
    id INTEGER PRIMARY KEY,
    name TEXT
);

INSERT INTO people (id, name) VALUES
(1, 'Alice'),
(2, 'Bob'),
(4, 'Charlie'),
(5, 'Diana'),
(7, 'Eve');
```

Then run it:

```bash
sqlite3 lab.db < init.sql
```

---

## 🧪 Step 6: Practice Queries

Back in the shell:

```bash
sqlite3 lab.db
```

Try recursive CTEs, gap detection, and more:

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

---

## 🧼 Step 7: Clean Up (Optional)

To remove the database:

```bash
rm lab.db
```

---

