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

Here's a **fully automated Bash script** to install SQLite, initialize a sample database, and run your missing-ID detection lab on **Ubuntu 22.04**.

---

## 🧪 SQLite Lab Setup Script (`sqlite_lab.sh`)

```bash
#!/bin/bash

# Step 1: Install SQLite
echo "🔧 Installing SQLite..."
sudo apt update && sudo apt install -y sqlite3

# Step 2: Create working directory
echo "📁 Creating lab directory..."
mkdir -p ~/sqlite_lab
cd ~/sqlite_lab || exit

# Step 3: Create SQL initialization file
echo "📄 Writing SQL setup..."
cat <<EOF > init.sql
DROP TABLE IF EXISTS people;

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
EOF

# Step 4: Create recursive CTE query file
cat <<EOF > find_missing_ids.sql
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
EOF

# Step 5: Initialize database
echo "🧪 Initializing SQLite database..."
sqlite3 lab.db < init.sql

# Step 6: Run missing ID detection
echo "🔍 Running missing ID query..."
sqlite3 lab.db < find_missing_ids.sql

echo "✅ Lab setup complete. You can now explore ~/sqlite_lab/lab.db interactively with:"
echo "    sqlite3 ~/sqlite_lab/lab.db"
```

---

## 🧾 Instructions to Run

1. Save the script as `sqlite_lab.sh` in your home directory.
2. Make it executable:
   ```bash
   chmod +x sqlite_lab.sh
   ```
3. Run it:
   ```bash
   ./sqlite_lab.sh
   ```

---

## 🧠 What You’ll Get
- SQLite installed and ready
- A database `lab.db` with sample data
- A query to find missing IDs (e.g., 3 and 6)
- A clean workspace in `~/sqlite_lab`


