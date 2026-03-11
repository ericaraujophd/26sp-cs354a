# Assignment 1 — Starting with SQL

- **Due:** Friday, March 28 at 11:55pm
>- **Points:** 100 (20 questions × 5 points each)
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

## The Database

The database is called **manga** and contains a single table:

### `manga`

| Column | Type | Description |
|---|---|---|
| `manga_id` | integer | Primary key |
| `title` | text | Title of the manga |
| `author` | text | Author's name |
| `genre` | text | Genre (e.g. Shonen, Shojo, Seinen) |
| `volumes` | integer | Number of volumes published |
| `year_published` | integer | Year the first volume was published |
| `status` | text | `'Ongoing'`, `'Completed'`, or `'Hiatus'` |
| `avg_rating` | numeric | Average reader rating (0.0 – 5.0) |

---

## Setup

Follow the [Student Setup Guide](student-guide.md) to get your environment ready. Once set up, you should be able to:

1. Start the database with `docker compose up -d`
2. Connect to it in VSCode via the SQLTools extension
3. Run queries directly in the editor

---

## Questions

Each question has its own `.sql` file (`q1.sql` through `q20.sql`). Open each file, read the instructions in the comment block, and write your query below the comment.

| # | Description | Points |
|---|---|---|
| 1 | List all manga titles, ordered alphabetically | 5 |
| 2 | Show the title and author of every manga | 5 |
| 3 | List all distinct genres in the database | 5 |
| 4 | Show titles and average ratings, ordered highest to lowest | 5 |
| 5 | List titles of all manga with status `'Ongoing'` | 5 |
| 6 | Show titles and volumes of manga with more than 20 volumes, ordered by volumes descending | 5 |
| 7 | List titles and year published of manga published in 2010 or later | 5 |
| 8 | Show all manga with an average rating of exactly 5.0 | 5 |
| 9 | List titles and genres of all manga in the `'Shonen'` genre | 5 |
| 10 | Show titles of all `'Completed'` manga, ordered alphabetically | 5 |
| 11 | List all distinct statuses in the database | 5 |
| 12 | Show title, author, and year of manga published before 2000, ordered by year ascending | 5 |
| 13 | List titles, ratings, and volumes of manga with rating above 4.0 AND more than 10 volumes | 5 |
| 14 | Show titles of manga whose title starts with the letter `'N'`, ordered alphabetically | 5 |
| 15 | List titles and volumes of manga that have exactly 1 volume | 5 |
| 16 | Show titles and genres of manga that are NOT in the `'Shonen'` genre, ordered by genre | 5 |
| 17 | List titles and year of manga published between 2000 and 2010 (inclusive), ordered by year | 5 |
| 18 | Show titles and ratings of manga with a rating between 3.0 and 4.0 (inclusive), ordered by rating descending | 5 |
| 19 | List titles and authors of manga where the author's name contains `'Oda'` | 5 |
| 20 | Show titles, status, and ratings of manga that are either `'Completed'` OR have a rating above 4.8, ordered by title | 5 |

---

## Tips

- Run `python grade.py` locally before submitting to see your score
- You can push as many times as you want — each push triggers a new grading run
- If you accidentally modify the data, reset the database with:
  ```bash
  docker compose down -v && docker compose up -d
  ```
- If a query returns no rows, double-check your `WHERE` conditions — it may be correct, or it may be filtering too aggressively

---

## Submitting

When you are ready to submit:

```bash
git add .
git commit -m "completed hw01"
git push
```

Then go to your repository on GitHub → **Actions tab** → click the latest run to see your score.

---

## Grading

Your score is calculated automatically by the grader. Each question is worth **5 points** for a total of **100 points**. Partial credit is not awarded per question — the result set must be correct.

The grader compares your result set against the expected output. This means:

- A different but equivalent query will receive full marks as long as it produces the correct rows
- Column order must match what is specified in the question
- Extra or missing columns will result in 0 for that question

---

*Questions? Use the feedback pull request on your repository or bring your error messages to office hours.*