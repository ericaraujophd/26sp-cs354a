# SQL Homework Assignment Guide
### GitHub Classroom + Local PostgreSQL + Hashed Autograding

---

## Overview

This guide describes the complete workflow for creating SQL homework assignments that:

- Give students access to PostgreSQL via **coder.cs.calvin.edu** (recommended)
  or a local install as a fallback
- Allow students to **write and test queries** directly in VSCode
- **Autograde submissions** automatically on every push via GitHub Actions
- Keep **answer keys private** using a two-repo model — questions, answers, and
  hash generation never touch the student repo

> **Student environment:** The primary path for students is Coder — a browser-based VSCode
> instance where PostgreSQL is pre-installed. Students connect via SQLTools using the manual
> Add New Connection flow (localhost, port 5432, username `admin`, password `admin`).
> Local install remains available as a fallback.

---

## Two-Repo Model

Each assignment uses two separate repositories:

| | Instructor repo | Student template repo |
|---|---|---|
| **Name** | `hw01-instructor` | `hw01` |
| **Visibility** | Private — never shared | Private template for GitHub Classroom |
| **Contains** | Schema, seed, questions, answers, `generate_hashes.py` | Student-facing files only |
| **Risk** | Zero — never cloned by students | Safe — no answer material |

You do all authoring work in the instructor repo. When ready to publish, a single
`publish.sh` script copies exactly the right files into the student template repo
and pushes it. You never touch the student repo directly, so accidental commits
of answers or unhashed content are structurally impossible.

```
hw01-instructor/          hw01/ (student template)
├── sql/                  ├── sql/
│   ├── 01_schema.sql  ──►│   ├── 01_schema.sql
│   └── 02_seed.sql    ──►│   └── 02_seed.sql
├── questions/            ├── q1.sql  (stubs only)
│   ├── q1_answer.sql     ├── q2.sql
│   └── q2_answer.sql     ├── ...
├── stubs/             ──►├── hashes.json         ← committed, safe to share
│   ├── q1.sql            ├── grade.py            ← reads hashes.json at runtime
│   └── q2.sql            ├── reset.sh
├── generate_hashes.py    ├── requirements.txt
├── hashes.json           ├── README.md
└── publish.sh            └── .github/workflows/autograder.yml
```

---

## How the Hashing Works

The autograder never compares query text — only result sets. This means:

- A student can write a completely different query than yours
- As long as it produces the **same rows**, they get full marks
- `hashes.json` is committed to the student repo and visible to students —
  but a hash reveals nothing about the answer

```
generate_hashes.py runs (instructor repo only)
        ↓
Auto-creates DB if missing → loads schema + seed → runs answer queries
        ↓
Produces hashes.json → publish.sh copies it to student repo
                                      ↓
                             grade.py reads hashes.json at runtime
                             auto-creates DB if missing on first run
                                      ↓
                    student query runs → result set → hashed → compared
```

Both `generate_hashes.py` and `grade.py` check whether the database exists
before connecting. If it does not, they create it and load the schema and seed
data automatically. Students never need to run a separate setup script — the
first `python grade.py` does everything.

`reset.sh` exists only as an escape hatch for when a student has accidentally
mutated data and needs a forced clean slate.

---

## One-Time Setup

Do this once for the entire course.

### 1. Install Tools on Your Machine

| Tool | URL |
|---|---|
| PostgreSQL 16 | See platform instructions below |
| VSCode | code.visualstudio.com |
| Claude extension for VSCode | Install from VSCode marketplace |

**Mac:**
```bash
brew install postgresql@16
brew services start postgresql@16
echo 'export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update && sudo apt install postgresql postgresql-contrib
sudo systemctl enable --now postgresql
```

**Windows:**
```bash
winget install PostgreSQL.PostgreSQL.16
```

### 2. Create a GitHub Organization for Your Course

```
github.com
→ Your profile photo (top right)
→ Your organizations
→ New organization
→ Free plan
→ Name: e.g. calvin-cs354
```

### 3. Set Up GitHub Classroom

```
classroom.github.com
→ New classroom
→ Select your organization
→ Name your classroom (e.g. CS354 Spring 2026)
```

### 4. Create the Two Repos per Assignment

For each assignment, create two empty repos in your GitHub org:

```
github.com/[YOUR-ORG]
→ New repository → hw01-instructor  (Private)
→ New repository → hw01            (Private)
```

---

## Per-Assignment Workflow

### Phase 0 — Prepare Your Source Files

Before invoking Claude, create the following four files in the instructor repo
directory. These are the only files you write by hand — Claude generates everything else.

| File | What it contains |
|---|---|
| `questions.md` | Numbered list of questions (one per line or section) |
| `sql/01_schema.sql` | `CREATE TABLE` statements for the assignment |
| `sql/02_seed.sql` | Realistic sample data (`INSERT` statements) |
| `queryanswers.sql` | All answer queries in order, separated by clearly numbered comments (e.g. `-- Q1`, `-- Q2`, …) |

> **Seed data tip:** Make sure at least one question over the correct answer returns
> a non-empty result, and that a `SELECT *` strategy fails on at least one question.

---

### Phase 1 — Generate Infrastructure with Claude Agent

With your four source files in place, open the **instructor repo** in VSCode,
switch to Agent mode, and use the following prompt. Fill in the fields marked
with brackets before sending.

---

```
I am a CS professor creating a SQL homework assignment using a two-repo model:
- An instructor repo (private) where I author everything
- A student template repo that receives only safe, published files

I have already written four source files in this workspace:
- `questions.md` — the numbered list of questions students must answer
- `sql/01_schema.sql` — the database schema (CREATE TABLE statements)
- `sql/02_seed.sql` — the seed data (INSERT statements)
- `queryanswers.sql` — all correct answer queries in order, separated by
  clearly numbered comments (e.g. `-- Q1`, `-- Q2`, …)

Do NOT regenerate or modify these files. Use them as the source of truth.

## Tech Stack
- PostgreSQL 16 (primary: Coder at coder.cs.calvin.edu; fallback: local install)
- Python + psycopg2-binary for the autograder
- GitHub Classroom
- VSCode as the student's development environment

## Database credentials
- Database name: [DATABASE NAME]
- Username: admin
- Password: admin
- Connection: localhost, port 5432
- Use these credentials consistently in generate_hashes.py, grade.py,
  reset.sh, and the README SQLTools connection instructions

## Assignment Details
- Assignment name: [HOMEWORK NAME]
- Topic: [TOPIC]
- Number of questions: [N]
- Points per question: [N]
- Database theme: [THEME]

## Please generate ALL of the following files:

### Instructor repo files (stay private, never published to students)

1. `questions/q[N]_answer.sql` — split `queryanswers.sql` into one file per
   question. Each file should contain the answer query with a brief comment
   header identifying the question number.
2. `generate_hashes.py` that:
   - Before running any queries, checks whether the database exists by connecting
     to the "postgres" maintenance database (user: admin, password: admin) and
     querying pg_database. If the database does not exist, it creates it and loads
     sql/01_schema.sql and sql/02_seed.sql via subprocess psql -f calls
     (passing -U admin and PGPASSWORD=admin)
   - Reads answer queries from questions/q[N]_answer.sql files
   - Runs each answer query
   - Normalizes results (sort rows, round floats to 4 decimals)
   - Hashes each result set with SHA256
   - Includes an ORDERED_QUESTIONS list for questions requiring ORDER BY
     (these are graded with row order preserved, not sorted)
   - Prints all hashes clearly
   - Saves hashes to hashes.json as:
     { "hashes": { "q1": "...", ... }, "ordered_questions": ["q3", ...] }
3. `publish.sh` — a script that:
   - Copies sql/01_schema.sql and sql/02_seed.sql to [STUDENT-REPO-PATH]/sql/
   - Copies stubs/q*.sql to [STUDENT-REPO-PATH]/
   - Copies hashes.json to [STUDENT-REPO-PATH]/
   - Copies grade.py, reset.sh, requirements.txt, README.md,
     and .github/ to [STUDENT-REPO-PATH]/
   - Runs git add, commit, and push in the student repo
   - Takes the student repo path as a command-line argument:
     bash publish.sh ../hw01

### Student-facing files (published to the student template repo via publish.sh)

4. `stubs/q[N].sql` — one stub file per question, derived from `questions.md`,
   containing:
   - A clear comment block explaining the question
   - The expected output columns listed in comments
   - A blank line where the student writes their query
5. `grade.py` that:
   - Reads hashes and ordered_questions from hashes.json at runtime:
     with open("hashes.json") as f: config = json.load(f)
   - Before running any student queries, checks whether the database exists
     by connecting to the "postgres" maintenance database (user: admin,
     password: admin) and querying pg_database. If the database does not exist,
     it creates it and loads sql/01_schema.sql and sql/02_seed.sql via subprocess
     psql -f calls (passing -U admin and PGPASSWORD=admin).
     Prints a clear message when auto-creating: "Database not found — creating
     and loading data..."
   - Connects to PostgreSQL and runs each student q[N].sql file
   - Normalizes results the same way as generate_hashes.py
     (sort rows unless order-sensitive, round floats to 4 decimals)
   - Hashes each result set and compares against hashes.json
   - Handles empty files, syntax errors gracefully
   - Prints ✅ or ❌ per question
   - Writes results.json with score per question and total
6. `reset.sh` — a shell script used only when a forced clean slate is needed
   (e.g. after accidentally modifying data with INSERT/UPDATE/DELETE). It:
   - Drops the database if it already exists
   - Creates a fresh database named [DATABASE NAME]
   - Loads schema and seed data via psql -f using -U admin and PGPASSWORD=admin
   - Works on Mac, Linux, and Windows via Git Bash
   - Includes a clear comment at the top: this is NOT needed for first-run
     setup — grade.py creates the database automatically on first run
7. `.github/workflows/autograder.yml` with EXACTLY this structure,
   substituting only [HOMEWORK NAME] (the assignment name, e.g. hw01-joins),
   [NUM QUESTIONS] (the number of questions as an integer), and
   [POINTS PER QUESTION] (points per question as an integer):

```yaml
name: Autograder
on:
  - push
  - workflow_dispatch
  - repository_dispatch

permissions:
  checks: write
  actions: read
  contents: read

jobs:
  grade:
    runs-on: runners-${{ github.repository_owner }}
    container:
      image: harbor.cs.calvin.edu/calvincs-public/devcontainer-cs354-autograde:latest
      env:
        PGUSER: admin
        PGPASSWORD: admin

    if: github.actor != 'github-classroom[bot]'
    steps:
      - name: Checkout repository
        uses: actions/checkout@v5

      - name: Start PostgreSQL
        run: /usr/local/bin/postgres-entrypoint.sh --start-only

      - name: Run autograder
        run: python3 grade.py

      - name: Post results as summary
        if: always()
        run: |
          python3 - <<'PYEOF'
          import json, os
          try:
              with open("results.json") as f:
                  data = json.load(f)
              total    = data["total_score"]
              max_s    = data["max_score"]
              questions = data["questions"]

              lines = [
                  "## [HOMEWORK NAME] \u2014 Grading Results\n",
                  f"**Score: {total}/{max_s} pts**\n",
                  "| Question | Result | Points |",
                  "|----------|--------|--------|",
              ]
              for i in range(1, [NUM QUESTIONS] + 1):
                  q     = questions.get(str(i), {})
                  score = q.get("score", 0)
                  st    = q.get("status", "fail")
                  icon  = "\u2705" if st == "pass" else "\u274c"
                  lines.append(f"| Q{i:02d} | {icon} | {score}/[POINTS PER QUESTION] |")

              summary = "\n".join(lines) + "\n"
              with open(os.environ["GITHUB_STEP_SUMMARY"], "a") as f:
                  f.write(summary)
          except Exception as e:
              print(f"Could not write summary: {e}")
          PYEOF
```
8. `requirements.txt` containing only:
    psycopg2-binary
9. `README.md` with:
    - Assignment overview and learning objectives
    - Prerequisites: PostgreSQL 16, VSCode, SQLTools, SQLTools PostgreSQL Driver
    - Step-by-step setup for both Coder and local install paths
    - A "Connecting SQLTools" section with manual connection instructions:
        Server: localhost, Port: 5432, Database: [DATABASE NAME],
        Username: admin, Password: admin
    - How to run queries: Cmd+Shift+P / Ctrl+Shift+P →
      "SQLTools: Run Selected Query"
    - How to check score: python grade.py
      (note: first run will auto-create the database — this is expected)
    - How to submit: git add, commit, push
    - Grading breakdown per question
    - Resetting the database: bash reset.sh (only needed if data was
      accidentally modified)

### Instructor repo gitignore
10. `.gitignore` that excludes:
    - hashes.json
    - venv/
    - __pycache__/
    - *.pyc
    - .env

## Constraints
- Do NOT use Docker
- Use psycopg2-binary (not psycopg2) everywhere
- PostgreSQL credentials are admin/admin throughout — in all Python scripts,
  reset.sh, and the README. Never use postgres/no-password
- Both generate_hashes.py and grade.py must auto-create the database if it
  does not exist — no separate setup step required
- grade.py must read hashes.json at runtime — no hardcoded hash values
- hashes.json is NOT gitignored in the student repo — it is safe to commit
- hashes.json IS gitignored in the instructor repo (regenerated each time)
- generate_hashes.py and questions/ never leave the instructor repo
- Normalization must be identical in generate_hashes.py and grade.py
- reset.sh must handle Mac (Homebrew), Linux, and Windows (Git Bash)
- Do NOT modify sql/01_schema.sql, sql/02_seed.sql, queryanswers.sql,
  or questions.md — these are the source of truth, already written
- All generated files must be consistent with the table names, column names,
  and credentials in the provided source files

## After generating all files, print a checklist:
1. Activate venv and run: python generate_hashes.py
   (it will create the database automatically on first run)
2. Verify hashes.json was created
3. Run grade.py with correct answers in stubs/ — confirm all ✅
4. Clear stubs/ back to blank — confirm all ❌
5. Run: bash publish.sh ../hw01
6. Verify student repo contains hashes.json but NOT generate_hashes.py
   or questions/
7. Push instructor repo to GitHub
8. Create assignment in GitHub Classroom pointing at the student template repo

Please create all files directly in the workspace.
```

---

### Fields to Fill In Per Assignment

| Field | Example |
|---|---|
| `[DATABASE NAME]` | `university` |
| `[HOMEWORK NAME]` | `hw02-joins` |
| `[TOPIC]` | `JOINs and aggregate functions` |
| `[N] questions` / `[NUM QUESTIONS]` | `5` |
| `[N] points` / `[POINTS PER QUESTION]` | `20` |
| `[STUDENT-REPO-PATH]` | `../hw02-joins` |

---

### Phase 2 — Generate the Hashes

After Claude creates all the files:

**1. Set up Python environment**
```bash
python3 -m venv venv
source venv/bin/activate      # Mac/Linux
venv\Scripts\activate         # Windows

pip install -r requirements.txt
```

**2. Run the hash generator**
```bash
python generate_hashes.py
```

On first run, you will see:
```
Database not found — creating and loading data...
q1: 7f3d9a2b1c8e4f6a5d2c8b9e3f1a4d7c
q2: 2a8f1c5e9b3d7a4f6c2e8b1d5a9f3c7e
q3: 9c3e7a1f5b8d2c6e4a9f1b7d3c5e8a2f

Saved to hashes.json
```

**3. Verify with correct answers**
```bash
# Temporarily copy correct answers into stubs/q1.sql etc.
python grade.py
# Expected output: all ✅
```

**4. Verify with blank stubs**
```bash
# Clear stubs/ back to blank
python grade.py
# Expected output: all ❌  — confirms grader is working correctly
```

---

### Phase 3 — Publish to the Student Repo

```bash
bash publish.sh ../[HOMEWORK-NAME]
```

This copies all student-facing files — including `hashes.json` — into the
student template repo, commits, and pushes. Verify the result:

```bash
cd ../[HOMEWORK-NAME]
git log --oneline -1          # confirm the publish commit is there
ls                            # confirm hashes.json is present
ls questions/ 2>/dev/null     # must return nothing — answers stay out
```

Then push the instructor repo separately:
```bash
cd ../[HOMEWORK-NAME]-instructor
git add .
git commit -m "hw01 initial setup"
git push -u origin main
```

---

### Phase 4 — Create the GitHub Classroom Assignment

```
classroom.github.com
→ Your classroom
→ New assignment
→ Title: [HOMEWORK NAME]
→ Repository prefix: [HOMEWORK NAME]
→ Private repositories: ✅
→ Template repository: [YOUR-ORG]/[HOMEWORK-NAME]
→ Next: Grading and feedback
→ Custom YAML tab → paste contents of autograder.yml
→ Protected file paths → type: .github/**/*  → Add Path
→ Enable feedback pull requests: ✅ (optional but recommended)
→ Create assignment
→ Copy invitation link → share with students
```

> **Note on Protected file paths:** Adding `.github/**/*` means that
> if a student modifies the autograder to give themselves full marks,
> GitHub Classroom will flag their submission with a warning label
> on your dashboard.

---

### Phase 5 — Student Flow

This is what students do after receiving the invitation link.
The primary environment is **Coder** (coder.cs.calvin.edu) — local install is the fallback.

**1. Accept the assignment**
```
Click invitation link
→ GitHub creates their own private repo automatically
→ Clone it inside the Coder workspace terminal (or locally)
```

**2. Set up Python**
```bash
python3 -m venv venv
source venv/bin/activate    # Mac/Linux/Coder
venv\Scripts\activate       # Windows
pip install -r requirements.txt
```

**3. Install SQLTools extensions**
```
Extensions panel (Ctrl+Shift+X / Cmd+Shift+X)
→ Install: SQLTools
→ Install: SQLTools PostgreSQL/Cockroach Driver
```

**4. Connect SQLTools to the database**
```
SQLTools icon (left sidebar)
→ Add New Connection → PostgreSQL
→ Fill in:
    Server:    localhost
    Port:      5432
    Database:  [DATABASE NAME]
    Username:  admin
    Password:  admin
→ Test Connection → Save Connection
→ Click the connection to connect
```

**5. Write and test queries**
```
Open q1.sql
→ Write query below the comment instructions
→ Cmd+Shift+P / Ctrl+Shift+P → "SQLTools: Run Selected Query"
→ Adjust until results look correct
```

**6. Check score (first run auto-creates the database)**
```bash
python grade.py
```

On first run:
```
Database not found — creating and loading data...
✅ q1: correct   (10 pts)
...
```

**7. Reset database if accidentally modified**
```bash
bash reset.sh    # only needed after accidental INSERT/UPDATE/DELETE
```

**8. Submit**
```bash
git add .
git commit -m "completed homework"
git push
```

**9. See results**
```
Go to their repo on GitHub
→ Actions tab
→ Latest workflow run
→ See ✅ or ❌ per question and total score
```

---

## What Lives Where

### Instructor repo (`hw01-instructor`) — private, never shared

| File | Purpose |
|---|---|
| `sql/01_schema.sql` | Source of truth for schema |
| `sql/02_seed.sql` | Source of truth for seed data |
| `questions/q[N]_answer.sql` | Correct answer queries |
| `generate_hashes.py` | Auto-creates DB if needed, runs answers, writes hashes.json |
| `hashes.json` | Gitignored — regenerated each time |
| `stubs/q[N].sql` | Blank question files for students |
| `publish.sh` | Publishes safe files to student repo |

### Student template repo (`hw01`) — published via `publish.sh`

| File | Purpose |
|---|---|
| `q1.sql`, `q2.sql`... | Student writes queries here |
| `hashes.json` | Read by grade.py at runtime — safe to commit |
| `grade.py` | Auto-creates DB if needed, reads hashes.json, grades queries |
| `sql/01_schema.sql` | Students can inspect the schema |
| `sql/02_seed.sql` | Students can inspect the seed data |
| `reset.sh` | Force-resets DB — only needed after accidental data modification |
| `requirements.txt` | Python dependencies |
| `.github/workflows/autograder.yml` | GitHub Actions grading |

---

## Troubleshooting

### psycopg2 installation errors
Always use `psycopg2-binary`, never `psycopg2`:
```bash
pip install psycopg2-binary
```

### generate_hashes.py or grade.py fails to auto-create the database
The most common cause is PostgreSQL not running:
```bash
# Mac (Homebrew)
brew services start postgresql@16
# Linux
sudo systemctl start postgresql
```
Also confirm that the `admin` role exists in your local PostgreSQL.
If it does not, create it:
```bash
psql -U postgres -c "CREATE ROLE admin WITH LOGIN PASSWORD 'admin' SUPERUSER;"
```

### Database not resetting cleanly
`reset.sh` uses `DROP DATABASE` which requires no active sessions.
Disconnect SQLTools in the sidebar before running it:
```bash
# Disconnect in SQLTools sidebar first, then:
bash reset.sh
```

### grade.py says "hashes.json not found"
The student repo is missing `hashes.json`. Re-run `publish.sh` from the
instructor repo — this file must be copied over before students can grade.

### Hash mismatch on correct answer
The most common cause is column ordering or float precision.
Re-run `generate_hashes.py` after any change to `02_seed.sql`,
then re-publish with `publish.sh`.

### GitHub Actions failing
Check the Actions tab in the student repo for the full log.
The most common cause is a syntax error in a `.sql` file that
crashes the grader before writing `results.json`.

---

*Guide created for CS354 — Database Management Systems*