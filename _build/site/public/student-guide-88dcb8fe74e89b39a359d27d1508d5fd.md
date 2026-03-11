---
title: Setup Guide for Students
subtitle: Getting Started with SQL Homework
date: 2026-03-11
---


---

## Choose Your Setup Path

There are two ways to work on assignments. **Option A is strongly recommended.**

| | Option A — Coder (recommended) | Option B — Local install |
|---|---|---|
| **Setup time** | ~5 minutes | 20–30 minutes |
| **PostgreSQL** | Pre-installed and running | You install it |
| **Works on** | Any browser | Mac, Linux, Windows |
| **Internet required** | Yes | No (after setup) |

---

## ✅ Option A — Use Coder (Recommended)

Coder gives you a fully configured VSCode environment in your browser —
PostgreSQL is already installed, running, and the database is pre-loaded
for each assignment. No installation required.

### Step A1 — Open the Workspace

1. Go to [coder.cs.calvin.edu](https://coder.cs.calvin.edu)
2. Log in with your Calvin credentials
3. Open your workspace (or create one if prompted)
4. Click **Open in VS Code** — this launches a full VSCode session
   in your browser with everything already configured

### Step A2 — Accept the Assignment

1. Click the invitation link your instructor shared
2. Click **Accept this assignment**
3. Wait a few seconds, then refresh the page
4. Click the link to your newly created repository
5. Copy the repository URL (green **Code** button and copy SSH URL)

### Step A3 — Clone the Repository

Open the terminal inside the Coder VSCode session and run:

```bash
git clone https://github.com/[YOUR-REPO-URL-HERE] git@github.com:26sp-cs354A/hw1.git
cd [HOMEWORK-NAME]
code .
```

### Step A4 — Set Up Python

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

You should see `(venv)` at the start of your terminal prompt.

> **Important:** Activate the virtual environment every time you open
> a new terminal window before running any Python commands.

### Step A5 — Connect VSCode to the Database

The repository includes a pre-configured database connection.

1. Click the **SQLTools icon** in the VSCode left sidebar
   (looks like a database cylinder)
2. Under **Connections**, you should see a connection already listed
3. Click it to connect — a green checkmark confirms success

The database is already populated with data. Skip straight to writing queries.

### Step A6 — Continue

Jump to [Step 7 — Explore the Database](#step-7--explore-the-database) below.
The remaining steps are identical for both options.

---

## 💻 Option B — Local Install

Use this option if you prefer to work offline or already have PostgreSQL installed.

### Step B1 — Install Tools

| Tool | What it is | Download |
|---|---|---|
| **PostgreSQL 16** | The database engine | See below |
| **Git** | Version control to submit your work | git-scm.com |
| **VSCode** | Code editor | code.visualstudio.com |
| **SQLTools** | VSCode extension to run SQL queries | Install from VSCode marketplace |
| **SQLTools PostgreSQL Driver** | Connects SQLTools to PostgreSQL | Install from VSCode marketplace |
| **Python 3.10+** | Runs the grader locally | python.org |

> **Note for Windows users:** After installing Git, use **Git Bash**
> as your terminal for all commands in this guide.

### Step B2 — Install PostgreSQL

**Mac:**
```bash
brew install postgresql@16
brew services start postgresql@16
```

Add PostgreSQL to your PATH (add this line to `~/.zshrc` or `~/.bash_profile`):
```bash
export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"
```

Then reload your shell:
```bash
source ~/.zshrc
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

**Windows:**
```bash
winget install PostgreSQL.PostgreSQL.16
```

After installation, verify PostgreSQL is running:
```bash
psql --version
```

You should see something like `psql (PostgreSQL) 16.x`.

### Step B3 — Accept the Assignment

1. Click the invitation link your instructor shared
2. Click **Accept this assignment**
3. Wait a few seconds, then refresh the page
4. Click the link to your newly created repository
5. Copy the repository URL (green **Code** button → copy HTTPS URL)

### Step B4 — Clone the Repository

```bash
git clone https://github.com/[YOUR-REPO-URL-HERE]
cd [HOMEWORK-NAME]
code .
```

### Step B5 — Set Up Python

**Mac / Linux:**
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**Windows:**
```bash
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

> **Important:** Activate the virtual environment every time you open
> a new terminal window before running any Python commands.

### Step B6 — Create the Database

```bash
bash setup.sh
```

This creates the database, loads the schema, and inserts the seed data.

> **Note:** On Linux/Windows, the script runs as the `postgres` superuser.
> If prompted for a password you don't know, see Troubleshooting below.

### Step B7 — Connect VSCode to the Database

1. Click the **SQLTools icon** in the VSCode left sidebar
2. Under **Connections**, you should see a connection already listed
3. Click it to connect — a green checkmark confirms success

If the connection does not appear automatically:

```
SQLTools sidebar
→ Add new connection
→ PostgreSQL
→ Fill in:
    Host:     localhost
    Port:     5432
    Database: [DATABASE NAME]
    Username: [your system username]
    Password: (leave blank if using Homebrew on Mac)
→ Save connection
```

---

## Step 7 — Explore the Database

Before writing queries, take a moment to understand the data.

In the SQLTools sidebar, expand your connection to see the tables.
Click any table to see its columns.

You can also run a quick exploration query — open any `.sql` file,
type the following, and press the **Run** button:

```sql
-- See all tables
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public';
```

```sql
-- Preview a table (replace 'students' with any table name)
SELECT * FROM students LIMIT 5;
```

---

## Step 8 — Write Your Queries

Each question has its own `.sql` file (e.g. `q1.sql`, `q2.sql`).

Open a question file. You will see something like this:

```sql
-- ============================================================
-- Question 1 (10 points)
-- List the name and GPA of all students with a GPA above 3.5
-- Order results by GPA descending
--
-- Expected columns: name, gpa
-- ============================================================

-- Write your query below:
```

Write your query below the comment block:

```sql
-- Write your query below:
SELECT name, gpa
FROM students
WHERE gpa > 3.5
ORDER BY gpa DESC;
```

To run the query and see results, click the **Run** button
in the top right of the editor, or press `Ctrl+Enter` (Windows/Linux)
/ `Cmd+Enter` (Mac).

Results appear in a table at the bottom of the screen.

---

## Step 9 — Check Your Score Locally

Before submitting, run the grader to see how many questions
you have correct:

```bash
python grade.py
```

Output will look like:

```
✅ q1: correct   (10 pts)
❌ q2: incorrect  (0 pts)
✅ q3: correct   (10 pts)
💥 q4: error — syntax error at or near "FORM"  (0 pts)
⬜ q5: empty file  (0 pts)

Score: 20/50
```

Fix any issues and run `grade.py` again until you are happy with your score.

---

## Step 10 — Submit Your Work

When you are ready to submit:

```bash
git add .
git commit -m "completed homework"
git push
```

GitHub Actions will automatically run the grader on your submission.

To see your results:

```
Go to your repo on GitHub
→ Click the Actions tab
→ Click the most recent workflow run
→ See ✅ or ❌ per question and your total score
```

You can push as many times as you want before the deadline.
Each push triggers a new grading run.

---

## Resetting the Database

If you accidentally modify the data (with INSERT, UPDATE, or DELETE)
and want to start fresh:

**Option A (Coder):** The database is shared infrastructure — contact
your instructor to have it reloaded, or ask them to run the reset for you.

**Option B (Local):**
```bash
bash reset.sh
```

This drops the database entirely and recreates it from scratch
with the original schema and seed data.

---

## Common Problems

### Cannot connect to the database in SQLTools (Option A — Coder)
Make sure your Coder workspace is running and you opened VSCode
from within it. Try refreshing the page and reopening the workspace.

### Cannot connect to the database in SQLTools (Option B — Local)
Make sure PostgreSQL is running:
```bash
# Mac (Homebrew)
brew services start postgresql@16

# Linux
sudo systemctl start postgresql

# Windows — open the Services panel and start "postgresql-x64-16"
```
Then try connecting again. Double-check that the username and database
name in the connection match what `setup.sh` created.

### `setup.sh` prompts for a password I don't know (Option B)
On Linux/Windows, PostgreSQL uses a `postgres` superuser with a password
set during installation. Try:
```bash
sudo -u postgres bash setup.sh    # Linux
```
On Mac with Homebrew, no password is needed.

### `pip install` fails
Make sure your virtual environment is activated — you should
see `(venv)` in your terminal prompt. If not:
```bash
source venv/bin/activate    # Mac/Linux/Coder
venv\Scripts\activate       # Windows
```

### `python grade.py` says "connection refused" (Option B)
PostgreSQL is not running. Start it using the commands above.

### I get `💥 syntax error` on a question
There is a SQL syntax error in your query. Read the error message
carefully — it usually tells you exactly where the problem is.
Common mistakes: typo in a keyword (`FORM` instead of `FROM`),
missing comma between column names, missing semicolon at the end.

### I accidentally pushed with empty query files
No problem — just write your queries and push again.
Each push creates a new grading run.

---

## Quick Reference

| Task | Option A (Coder) | Option B (Local) |
|---|---|---|
| Open environment | [coder.cs.calvin.edu](https://coder.cs.calvin.edu) | Open VSCode locally |
| Database setup | Already done ✅ | `bash setup.sh` |
| Reset database | Contact instructor | `bash reset.sh` |
| Activate venv | `source venv/bin/activate` | same (Mac/Linux) / `venv\Scripts\activate` (Win) |
| Check your score | `python grade.py` | `python grade.py` |
| Submit | `git add . && git commit -m "msg" && git push` | same |

---

## Getting Help

- Use the **feedback pull request** on your repo to ask your instructor questions
- Bring specific error messages to office hours or tutoring sessions
- Check the **Actions tab** on GitHub for detailed grading logs

Good luck! 🐘