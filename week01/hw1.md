--- 
title: Homework 1
subtitle: Starting with SQL
---

- **Due:** Friday, March 28 at 11:55pm
- **Points:** 100 (20 questions × 5 points each)
- **Submission:** via GitHub Classroom

---

## Overview

In this assignment you will write your first SQL queries against a manga library database. The goal is to get comfortable with the core building blocks of SQL: selecting data, filtering it, and sorting it.

You will work with a PostgreSQL database running locally on your machine via Docker, write your queries in VSCode, and submit your work through GitHub Classroom.

---

## Learning Objectives

By the end of this assignment you will be able to:

- Write `SELECT` statements to retrieve specific columns from a table
- Use `WHERE` to filter rows based on conditions
- Use `ORDER BY` to sort results in ascending and descending order
- Use `DISTINCT` to eliminate duplicate values
- Combine conditions with `AND`, `OR`, and `NOT`
- Use `LIKE` to match patterns in text
- Use `BETWEEN` to filter within a range

---

## Accepting the Assignment

1. Click the [GitHub Classroom invitation link](https://classroom.github.com/a/s72gAHjF).
2. Accept the assignment. This creates a private repository for you.
3. Open your Coder workspace at **coder.cs.calvin.edu**.
4. Clone your repository in the terminal:
   ```bash
   git clone <your-repo-url>
   ```
   Replace `<your-repo-url>` with the URL GitHub gives you after accepting.
5. Open the cloned folder in VSCode.

---

## What You Are Building

You will write 20 SQL queries against a **manga** database. The database has one table:

| Column           | Type           | Description                        |
|------------------|----------------|------------------------------------|
| `id`             | `SERIAL`       | Primary key                        |
| `title`          | `VARCHAR(200)` | Manga title                        |
| `author`         | `VARCHAR(200)` | Author name                        |
| `genre`          | `VARCHAR(100)` | Genre (e.g. Shonen, Seinen, Shojo) |
| `status`         | `VARCHAR(50)`  | Publication status (Ongoing, Completed, Hiatus) |
| `volumes`        | `INTEGER`      | Number of published volumes        |
| `year_published` | `INTEGER`      | Year first published               |
| `avg_rating`     | `NUMERIC(3,1)` | Average rating (0.0–5.0)           |

---

## Setup

### Step 1 — Install VSCode Extensions

Open VSCode and install these two extensions (if you do not already have them):

- **SQLTools** — search `mtxr.sqltools` in the Extensions sidebar
- **SQLTools PostgreSQL/Cockroach Driver** — search `mtxr.sqltools-driver-pg`

### Step 2 — Create the Database

In the VSCode terminal, run the autograder once:

```bash
python3 grade.py
```

The first run will automatically create the `manga` database and load all the data. You will see the message:

```
Database not found — creating and loading data...
```

This is expected. After that, every question will show a red X because you have not written any queries yet.

### Step 3 — Connect SQLTools

1. Click the **database icon** in the left sidebar to open SQLTools.
2. Click **Add New Connection**.
3. Choose **PostgreSQL**.
4. Fill in:
   - **Server**: `localhost`
   - **Port**: `5432`
   - **Database**: `manga`
   - **Username**: `admin`
   - **Password**: `admin`
5. Click **Test Connection**, then **Save Connection**.

You can now run queries interactively from within VSCode.

---

## Writing Your Queries

Your repository contains 20 SQL files: `q1.sql` through `q20.sql`.

Each file has a comment at the top describing the question and the expected output columns. Write your SQL query on the blank line below the comments.

**Example** — if `q1.sql` looks like this:

```sql
-- Question 1: List all manga titles in the database, ordered alphabetically.
-- Expected columns: title
```

You would add your query so the file becomes:

```sql
-- Question 1: List all manga titles in the database, ordered alphabetically.
-- Expected columns: title
SELECT title FROM manga ORDER BY title;
```

### Running a Query Interactively

1. Open a `.sql` file.
2. Select (highlight) your query.
3. Press **Cmd+Shift+P** (Mac) or **Ctrl+Shift+P** (Windows/Linux).
4. Type and select **SQLTools: Run Selected Query**.
5. The results will appear in a panel below.

This lets you test and debug your queries before grading.

---

## Checking Your Score

Run the autograder at any time:

```bash
python3 grade.py
```

You will see output like:

```
==================================================
  Homework 1 — Autograder
==================================================

  Q01: ✅  (5/5 pts)
  Q02: ❌  (0/5 pts)
  ...

==================================================
  Total: 5/100 pts
==================================================
```

Each question is **all-or-nothing**: your query must return exactly the correct columns and rows to earn the 5 points.

---

## Questions Summary

| #  | Task | Expected Columns |
|----|------|------------------|
| 1  | All manga titles, ordered alphabetically | `title` |
| 2  | Title and author of every manga | `title, author` |
| 3  | All distinct genres | `genre` |
| 4  | Titles and average ratings, highest to lowest | `title, avg_rating` |
| 5  | Titles of all manga that are 'Ongoing' | `title` |
| 6  | Title and volumes where volumes > 20, by volumes descending | `title, volumes` |
| 7  | Manga published 2010 or later, title and year | `title, year_published` |
| 8  | Manga with an average rating of exactly 5.0 | `title, avg_rating` |
| 9  | Titles and genres of manga in the 'Shonen' genre | `title, genre` |
| 10 | Titles of 'Completed' manga, ordered alphabetically | `title` |
| 11 | All distinct statuses | `status` |
| 12 | Title, author, year of manga published before 2000, by year ascending | `title, author, year_published` |
| 13 | Manga with avg_rating > 4.0 AND volumes > 10 | `title, avg_rating, volumes` |
| 14 | Titles starting with the letter 'N', ordered alphabetically | `title` |
| 15 | Manga with exactly 1 volume | `title, volumes` |
| 16 | Manga NOT in the 'Shonen' genre, ordered by genre | `title, genre` |
| 17 | Manga published between 2000 and 2010 (inclusive), by year | `title, year_published` |
| 18 | Manga with rating between 3.0 and 4.0 (inclusive), by rating descending | `title, avg_rating` |
| 19 | Manga where author's name contains 'Oda' | `title, author` |
| 20 | Manga that are 'Completed' OR have avg_rating > 4.8, by title | `title, status, avg_rating` |

---

## Grading

| Questions  | Points Each | Total   |
|------------|-------------|---------|
| Q01 – Q20  | 5 pts       | 100 pts |

The autograder also runs automatically on GitHub each time you push. You can view your results in the **Actions** tab of your repository.

---

## Submitting

When you are done (or want to save progress):

```bash
git add -A
git commit -m "Submit Homework 1"
git push
```

You can push as many times as you like. Only your latest push is graded.

---

## Troubleshooting

### The database got messed up

If you accidentally ran `INSERT`, `UPDATE`, or `DELETE` and the data is wrong, reset it:

```bash
bash reset.sh
```

This drops and recreates the database from scratch. You do **not** need this for normal use.

### grade.py says "file not found"

Make sure your query files are named `q1.sql` through `q20.sql` and are in the **root** of your repository (not in a subfolder).

### SQLTools cannot connect

Make sure you ran `python3 grade.py` at least once first to create the database. Verify the connection settings match: `localhost`, port `5432`, database `manga`, user `admin`, password `admin`.
