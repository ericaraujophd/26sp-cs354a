---
title: "Final Project — Scripture & SQL"
subtitle: "CS354 · Database Management Systems · Calvin University"
subject: Final Project
authors:
  - name: Eric Araújo
    affiliation: Calvin University
    email: eric.araujo@calvin.edu
license: CC-BY-4.0
---

# Final Project — Scripture & SQL

## A Biblical Data Investigation

For thousands of years, scholars read the Bible one verse at a time — searching by hand, marking pages, keeping indexes. You are about to read all 31,102 verses at once.

The dataset you will work with is not just a collection of texts. It is a **knowledge graph of ancient civilization**, converted into a relational database and loaded into Supabase. Behind each table row is a thread connecting three millennia of recorded history: genealogies that span forty generations, places mapped to coordinates, and people referenced across books written centuries apart.

SQL gives you something no scribe, theologian, or historian ever had: the power to ask questions **at scale**. Who appears in both the Old and New Testaments? Which geographic region is mentioned most across all 66 books? How many generations of descendants does Abraham have in the recorded text? These are not theological questions — they are queries. And by the end of this project, you will have the tools to answer them.

---

## The Dataset

The database contains **nine tables** drawn from a Neo4j graph database of the King James Bible, enriched with biblical reference data.

| Table | Rows | What it holds |
|---|---|---|
| `testaments` | 2 | Old Testament and New Testament |
| `books` | 66 | All canonical books, in order |
| `chapters` | 1,189 | Every chapter, linked to its book |
| `verses` | 31,102 | Every verse in the KJV, with full text |
| `persons` | 3,069 | Named biblical persons |
| `places` | 1,274 | Geographic places, including coordinates |
| `verse_persons` | ~28,000 | Which persons appear in which verses |
| `verse_places` | ~6,000 | Which places appear in which verses |
| `genealogy` | 1,785 | Recorded parent–child relationships |

### The `verse_code` Format

Each verse has a unique `verse_code` in the format `BBCCCVVV` (8 digits):

- **BB** — book number (01–66, in canonical order)
- **CCC** — chapter number (001–150)
- **VVV** — verse number within the chapter (001–176)

For example, Genesis 1:1 is `01001001` and John 3:16 is `43003016`.

### The Junction Table Join Pattern

The tables `verse_persons`, `verse_places`, and `genealogy` link rows through the `id` column of each entity table. Always join them like this:

```sql
JOIN verse_persons vp ON vp.verse_id  = v.id
JOIN persons       p  ON vp.person_id = p.id
```

---

## Schema Reference

```{mermaid}
erDiagram
    testaments {
        int id PK
        text name
    }
    books {
        int id PK
        text title
        text short_name
        int book_order
        int testament_id FK
    }
    chapters {
        int id PK
        int chapter_num
        int book_id FK
    }
    verses {
        int id PK
        text verse_code "BBCCCVVV"
        int verse_num
        text verse_text
        int chapter_id FK
    }
    persons {
        int id PK
        text name
        text gender
        text title
    }
    places {
        int id PK
        text name
        text feature_type
        numeric latitude
        numeric longitude
    }
    verse_persons {
        int verse_id FK
        int person_id FK
    }
    verse_places {
        int verse_id FK
        int place_id FK
    }
    genealogy {
        int person_id FK
        int parent_id FK
    }

    testaments ||--o{ books : "contains"
    books ||--o{ chapters : "contains"
    chapters ||--o{ verses : "contains"
    verses ||--o{ verse_persons : "mentions"
    persons ||--o{ verse_persons : "appears in"
    verses ||--o{ verse_places : "mentions"
    places ||--o{ verse_places : "appears in"
    persons ||--o{ genealogy : "is parent"
    persons ||--o{ genealogy : "is child"
```

**Quick column reference**

```
testaments  (id, name)
books       (id, title, short_name, book_order, slug, osis_ref, testament_id)
chapters    (id, chapter_num, osis_ref, book_id)
verses      (id, verse_code, verse_num, verse_text, osis_ref, chapter_id)
persons     (id, person_ref, name, title, gender, slug, status)
places      (id, place_ref, name, feature_type, latitude, longitude, status)

verse_persons  (verse_id → verses.id,   person_id → persons.id)
verse_places   (verse_id → verses.id,   place_id  → places.id)
genealogy      (person_id → persons.id, parent_id → persons.id)
```

---

## What You Will Do

You and your partner will complete **15 guided SQL queries** and **one open investigation**. All work lives in a single file — `queries.py`.

### Your workflow

1. Accept the GitHub Classroom assignment from the link on Moodle.
2. Clone the repository to your machine (or open it on `coder.cs.calvin.edu`).
3. Create a Python virtual environment and install dependencies.
4. Copy `.env.example` to `.env`, then paste the database credentials from Moodle into it.
5. Run `python app.py` and open `http://localhost:5000`.
6. Read the docstring for each function in `queries.py`, write your SQL, save the file, and refresh the dashboard.
7. When all 15 structured queries are green, fill in the three `RESEARCH_*` strings at the bottom of `queries.py`.
8. Record a 2-minute video where both partners explain your research question, walk through your SQL, and share your findings. Upload it to YouTube (unlisted) or Vimeo and copy the link.
9. Commit and push your final `queries.py`: `git add queries.py`, `git commit -m "Complete final project"`, `git push`.
10. Submit the video URL through the Moodle assignment link.

| File | Edit it? |
|---|---|
| `queries.py` | ✅ Yes — the only file you need to touch |
| `app.py` | 🚫 No |
| `grade.py` | 🚫 No |
| `.env` | 🚫 Do not commit this file |

---

## Setup

```bash
# Clone (replace URL with your GitHub Classroom link)
git clone <your-repo-url>
cd final-project

# Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate      # Mac / Linux
.venv\Scripts\activate         # Windows

# Install dependencies
pip install -r requirements.txt

# Create your .env from the template, then paste the credentials from Moodle
cp .env.example .env
# Open .env and fill in the values posted on Moodle

# Start the dashboard
python app.py
# → open http://localhost:5000
```

---

## Exploring the Database with SQLTools (VS Code)

Before writing your queries in `queries.py`, it helps to browse the tables and run exploratory SQL directly inside VS Code. **SQLTools** is a free extension that gives you a database explorer panel and an in-editor SQL runner — no separate client needed.

### Step 1 — Install the extensions

Open the Extensions panel in VS Code (`Cmd+Shift+X` on Mac, `Ctrl+Shift+X` on Windows) and install both:

1. **SQLTools** — search for `SQLTools` by Matheus Teixeira (`mtxr.sqltools`)
2. **SQLTools PostgreSQL/CockroachDB Driver** — search for `SQLTools PostgreSQL` (`mtxr.sqltools-driver-pg`)

### Step 2 — Create a new connection

1. Click the **SQLTools icon** in the left Activity Bar (looks like a cylinder).
2. Click **Add New Connection**.
3. Choose **PostgreSQL** as the driver.
4. Fill in the connection form using the credentials from your `.env` file:

| Field | Value |
|---|---|
| Connection name | `Scripture & SQL` (or anything you like) |
| Server / Host | value of `SUPABASE_HOST` in your `.env` |
| Port | `5432` |
| Database | `postgres` |
| Username | value of `SUPABASE_USER` in your `.env` |
| Password | value of `SUPABASE_PASSWORD` in your `.env` |
| SSL | **Enabled** |
| Default schema | `bible` |

:::{note} Use port 5432, not 6543
The Flask app connects through Supabase's connection pooler (port 6543). For SQLTools, use the direct connection port **5432** — it supports all PostgreSQL features including table browsing.
:::

:::{note} The bible tables live in the `bible` schema
The database is shared with another course assignment, so you will see a `public` schema with unrelated tables when browsing. Ignore those. All nine bible tables (`verses`, `books`, `persons`, etc.) are in the `bible` schema. Setting **Default schema** to `bible` above means your queries will resolve table names automatically — you can write `SELECT * FROM verses` without any prefix.
:::

5. Click **Test Connection** to verify it works, then **Save**.

### Step 3 — Browse the tables

Once connected, expand the connection in the SQLTools sidebar. You will see a **Tables** folder listing all nine tables. Click any table name to see its columns and types. Right-click a table and choose **Select Top 10** to preview its data instantly.

### Step 4 — Run exploratory queries

Open any `.sql` file (or create a new one) and write a query. To run it:

- Highlight the query text and press `Ctrl+E` `Ctrl+E` (run selection), or
- Use the **Run on active connection** button in the editor toolbar.

Results appear in a panel at the bottom of the screen, paginated and copyable.

A few good queries to start with:

```sql
-- What do the first few verses look like?
SELECT b.title, c.chapter_num, v.verse_num, v.verse_text
FROM verses v
JOIN chapters c ON v.chapter_id = c.id
JOIN books    b ON c.book_id    = b.id
ORDER BY b.book_order, c.chapter_num, v.verse_num
LIMIT 20;

-- Who are the persons in the database?
SELECT name, gender, title FROM persons ORDER BY name LIMIT 30;

-- What types of places exist?
SELECT feature_type, COUNT(*) AS total
FROM places
GROUP BY feature_type
ORDER BY total DESC;
```

Use SQLTools to experiment freely — it is the ideal environment for drafting your Part 5 research query before copying the final version into `queries.py`.

---

## Query Tasks

### Part 1 — Basic DQL (15 pts)

Five queries covering `SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`, and `HAVING`. Each is worth 3 points.

| # | Title | Key skill |
|---|---|---|
| Q1 | Books in Canonical Order | JOIN across two tables |
| Q2 | Verse Counts per Book | COUNT + GROUP BY |
| Q3 | Chapters with More than 40 Verses | GROUP BY + HAVING |
| Q4 | Find a Verse by Reference | WHERE on `verse_code` |
| Q5 | Gender Distribution of Persons | COALESCE + GROUP BY |

### Part 2 — Joins & Subqueries (25 pts)

Five queries requiring joins across multiple tables or subqueries. Each is worth 5 points.

| # | Title | Key skill |
|---|---|---|
| Q6  | Most Mentioned Persons | Junction table join |
| Q7  | Most Mentioned Places | Junction table join |
| Q8  | Cross-Testament Persons | Conditional aggregation |
| Q9  | Books with No Place Mentions | Subquery or LEFT JOIN with NULL check |
| Q10 | Verses Mentioning a Person and a Place | Two subqueries combined |

### Part 3 — Window Functions (15 pts)

Three queries requiring window functions. Each is worth 5 points.

| # | Title | Points |
|---|---|---|
| Q11 | Verse Count Rank by Book | 5 |
| Q12 | Running Total of Verses | 5 |
| Q13 | Person Mention Percentile | 5 |

### Part 4 — Common Table Expressions (15 pts)

Two queries that must use a `WITH` clause.

| # | Title | Points |
|---|---|---|
| Q14 | Books Above Average Verse Count | 6 |
| Q15 | Descendants of a Biblical Figure (recursive CTE) | 9 |

### Part 5 — Open Investigation (30 pts)

Design and answer your own question about the biblical data. This part has two deliverables: code in `queries.py` and a short video.

#### Part 5a — Code

Fill in three strings at the bottom of `queries.py`:

- **`RESEARCH_QUESTION`** — one or two sentences describing what you are investigating and why it is interesting.
- **`RESEARCH_QUERY`** — a valid SQL query using at least one technique from the course.
- **`RESEARCH_FINDINGS`** — 3–5 sentences interpreting your results.

#### Part 5b — 2-Minute Video

Record a short video (maximum 2 minutes) in which both team members explain:

1. **Your question** — what you decided to investigate and why you chose it.
2. **Your query** — walk through your SQL briefly, explaining the key technique you used.
3. **Your findings** — what the data revealed, and what surprised you or made you think.

The video does not need to be polished — a screen recording with your voice is fine. Both partners must speak. Upload it to YouTube (unlisted) or Vimeo and submit the URL on Moodle.

:::{note} Part 5 is not auto-graded
The dashboard runs your query and displays live results, but assigns no points. Your instructor evaluates Part 5 based on the depth of your question, the correctness of your query, the quality of your written findings, and your video reflection.
:::

Some directions to consider:

- Which book has the highest ratio of place mentions to verse count?
- Are female persons mentioned more in the Old or New Testament?
- Which chapter contains the most unique persons mentioned in a single chapter?
- What is the average verse length (in characters) per book?
- How far does the genealogical record extend from a single ancestor?

---

## Grading

| Section | Points | How graded |
|---|---|---|
| Part 1 — Basic DQL | 15 | Automatic (hash comparison) |
| Part 2 — Joins & Subqueries | 25 | Automatic (hash comparison) |
| Part 3 — Window Functions | 15 | Automatic (hash comparison) |
| Part 4 — CTEs | 15 | Automatic (hash comparison) |
| Part 5 — Open Investigation | 30 | Instructor review (code + video) |
| **Total** | **100** | |

Each auto-graded query is all-or-nothing: full points if the result set matches exactly (correct data **and** correct ordering), zero otherwise.

---

## Submission

This project has **two separate submissions**, both due by the deadline posted on Moodle.

**1. Code — GitHub Classroom**

Push your completed `queries.py` to the GitHub Classroom repository:

```bash
git add queries.py
git commit -m "Complete final project"
git push
```

:::{warning} Do not commit your `.env` file
It contains database credentials. It is listed in `.gitignore` for a reason — never add it to your commit.
:::

**2. Video — Moodle**

Submit the URL of your 2-minute investigation video through the Moodle assignment link. Make sure the link is publicly accessible (or shared with your instructor) before submitting.

---

## Academic Integrity

All queries you submit must be your own team's work. Do not copy queries from other teams or use AI tools to write them for you. External references — documentation, textbooks, class notes — are appropriate and expected.