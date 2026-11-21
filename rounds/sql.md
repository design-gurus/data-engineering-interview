# The SQL Round

The SQL round for a data engineer assumes joins and GROUP BY and tests what comes after them. Four query families account for most questions: ranking within groups, deduplication, running calculations, and finding runs of consecutive rows. Nearly all of them depend on window functions.

A window function computes a value for each row from a set of related rows, without collapsing those rows the way GROUP BY does. The set is defined by three parts: PARTITION BY (which rows belong together), ORDER BY (their order), and an optional frame (which neighbors count).

## What the round tests

| Family | A typical question | The tool |
|---|---|---|
| Ranking | Top three products in each category | `ROW_NUMBER`, `RANK`, `DENSE_RANK` |
| Deduplication | Keep the latest record for each customer | `ROW_NUMBER` with a filter |
| Running calculations | Running total, 7-day moving average | `SUM` or `AVG` with a frame |
| Gaps and islands | Longest streak of consecutive days | Date minus row number |
| Sessionization | Group events separated by under 30 minutes | `LAG` and a running sum |
| Joins | Customers with no orders, latest price at order time | Anti-join, range join |

## Worked query 1: keep the latest row for each key

A contacts table receives several versions of each contact. The question asks for one row per contact, the most recent.

```sql
WITH ranked AS (
  SELECT
    contact_id,
    email,
    updated_at,
    ROW_NUMBER() OVER (
      PARTITION BY contact_id
      ORDER BY updated_at DESC
    ) AS rn
  FROM contacts_raw
)
SELECT contact_id, email, updated_at
FROM ranked
WHERE rn = 1;
```

The window result cannot be filtered in WHERE of the same query, because WHERE runs before window functions. The common table expression (CTE), a named subquery introduced by WITH, solves that. If two versions share the same `updated_at`, the choice between them is arbitrary. Say so, and add a second ORDER BY column to make it deterministic.

## Worked query 2: gaps and islands

A logins table has one row per user per login. The question asks for each streak of consecutive days.

```sql
WITH days AS (
  SELECT DISTINCT user_id, login_date
  FROM logins
),
numbered AS (
  SELECT
    user_id,
    login_date,
    ROW_NUMBER() OVER (
      PARTITION BY user_id
      ORDER BY login_date
    ) AS rn
  FROM days
)
SELECT
  user_id,
  MIN(login_date) AS streak_start,
  MAX(login_date) AS streak_end,
  COUNT(*) AS streak_days
FROM numbered
GROUP BY user_id, login_date - rn * INTERVAL '1' DAY;
```

Inside one streak, the date and the row number both rise by one each day. Their difference is therefore constant, and it changes when a day is missed. Grouping by that difference isolates each streak. The date arithmetic in the last line is written differently in each engine, so name the engine before writing it.

## Edge cases to say aloud

- **Ties.** `ROW_NUMBER` returns exactly N rows. `RANK` and `DENSE_RANK` can return more.
- **Nulls.** `COUNT(column)` skips nulls, and a null never equals another null in a join.
- **Duplicates.** A join to a table with repeated keys multiplies rows and inflates sums.
- **Frames.** With ORDER BY and no frame, the default frame ends at the current row, which makes `LAST_VALUE` return the current row.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-sql) has an Advanced SQL module with ten lessons on these families.
- [Grokking SQL for Tech Interviews](https://www.designgurus.io/course/grokking-sql-for-tech-interviews?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-sql) covers the joins and aggregation that this round assumes.
- [SQL patterns](../cheat-sheets/sql-patterns.md) lists ten query shapes with a snippet each.
- Free article: [How a SQL query is executed](https://www.designgurus.io/blog/sql-query-execution-process?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-sql), which explains how a database processes a query.
- Free answer: [SQL query optimization interview questions](https://www.designgurus.io/answers/detail/what-are-sql-query-optimization-interview-questions?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-sql), which covers the follow-up questions about query speed.
