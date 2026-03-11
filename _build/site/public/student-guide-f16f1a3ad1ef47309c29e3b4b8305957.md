# Getting Started with SQL Homework
### Setup Guide for Students

---

## What You Will Need

Before you start, install the following tools:

| Tool | What it is | Download |
|---|---|---|
| **PostgreSQL 16** | The database engine | See Step 1 below |
| **Git** | Version control to submit your work | git-scm.com |
| **VSCode** | Code editor | code.visualstudio.com |
| **SQLTools** | VSCode extension to run SQL queries | Install from VSCode marketplace |
| **SQLTools PostgreSQL Driver** | Connects SQLTools to PostgreSQL | Install from VSCode marketplace |
| **Python 3.10+** | Runs the grader locally | python.org |

> **Note for Windows users:** After installing Git, use **Git Bash**
> as your terminal for all commands in this guide.

---

## Step 1 — Install PostgreSQL

Choose the instructions for your operating system.

### Mac
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

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### Windows
```bash
winget install PostgreSQL.PostgreSQL.16
```

After installation, open a new terminal and verify PostgreSQL is running:
```bash
psql --version
```

You should see something like `psql (PostgreSQL) 16.x`.

---

## Step 2 — Accept the Assignment

1. Click the invitation link your instructor shared
2. Click **Accept this assignment**
3. Wait a few seconds, then refresh the page
4. Click the link to your newly created repository
5. Copy the repository URL (green **Code** button → copy HTTPS URL)

---

## Step 3 — Clone the Repository

Open a terminal and run:

```bash
git clone https://github.com/[YOUR-REPO-URL-HERE]
cd [HOMEWORK-NAME]
```

Then open it in VSCode:
```bash
code .
```

---

## Step 4 — Set Up Python

Create a virtual environment so the required packages don't
conflict with anything else on your machine:

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

You should see `(venv)` at the start of your terminal prompt,
confirming the environment is active.

> **Important:** You need to activate the virtual environment
> every time you open a new terminal window before running
> any Python commands.

---

## Step 5 — Create the Database

Run the setup script to create the database and load the data:

**Mac / Linux:**
```bash
bash setup.sh
```

**Windows (Git Bash):**
```bash
bash setup.sh
```

This script creates a new database, loads the schema, and inserts the seed data.
You should see output confirming the database was created successfully.

> **Note:** The script uses the default PostgreSQL superuser (`postgres` on Linux/Windows,
> your system username on Mac via Homebrew). If you are prompted for a password
> and don't know it, see the Troubleshooting section below.

---

## Step 6 — Connect VSCode to the Database

The repository includes a pre-configured database connection.
To use it:

1. Click the **SQLTools icon** in the VSCode left sidebar
   (looks like a database cylinder)
2. Under **Connections**, you should see a connection already listed
3. Click it to connect
4. A green checkmark confirms you are connected

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
in the top right of the editor, or press `Ctrl+Enter` (Windows)
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

Fix any issues and run `grade.py` again until you are happy
with your score.

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

**Mac / Linux:**
```bash
bash reset.sh
```

**Windows (Git Bash):**
```bash
bash reset.sh
```

This drops the database entirely and recreates it from scratch
with the original schema and seed data.

---

## Common Problems

### PostgreSQL is not running
Start it with:
```bash
# Mac (Homebrew)
brew services start postgresql@16

# Linux
sudo systemctl start postgresql

# Windows — open the Services panel and start "postgresql-x64-16"
```

### Cannot connect to the database in SQLTools
Make sure PostgreSQL is running (see above), then try connecting again.
Double-check that the username and database name in the connection match
what was created by `setup.sh`.

### `setup.sh` or `reset.sh` prompts for a password I don't know
On Linux/Windows, PostgreSQL installs a `postgres` superuser with a
password set during installation. If you don't remember it, ask your
instructor or try running:
```bash
sudo -u postgres psql
```
On Mac with Homebrew, no password is needed — your system username is the superuser.

### `pip install` fails
Make sure your virtual environment is activated — you should
see `(venv)` in your terminal prompt. If not:
```bash
source venv/bin/activate    # Mac/Linux
venv\Scripts\activate       # Windows
```

### `python grade.py` says "connection refused"
PostgreSQL is not running. Start it (see above).

### I get `💥 syntax error` on a question
There is a SQL syntax error in your query. Read the error message
carefully — it usually tells you exactly where the problem is.
Common mistakes:
- Typo in a keyword (e.g. `FORM` instead of `FROM`)
- Missing comma between column names
- Missing semicolon at the end

### I accidentally pushed with empty query files
No problem — just write your queries and push again.
Each push creates a new grading run.

---

## Quick Reference

| Task | Command |
|---|---|
| Start PostgreSQL (Mac) | `brew services start postgresql@16` |
| Start PostgreSQL (Linux) | `sudo systemctl start postgresql` |
| Create/load database | `bash setup.sh` |
| Reset database | `bash reset.sh` |
| Activate venv (Mac/Linux) | `source venv/bin/activate` |
| Activate venv (Windows) | `venv\Scripts\activate` |
| Check your score | `python grade.py` |
| Submit | `git add . && git commit -m "msg" && git push` |

---

## Getting Help

- Use the **feedback pull request** on your repo to ask your instructor questions
- Bring specific error messages to office hours or tutoring sessions
- Check the **Actions tab** on GitHub for detailed grading logs

Good luck! 🐘