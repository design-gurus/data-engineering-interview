# SQL Patterns

Ten query shapes answer most SQL questions in a data engineering interview. Each one below has a minimal snippet in standard SQL. Time values are stored as integer seconds (`event_ts`) or integer day numbers (`day_no`) where arithmetic is needed, because date arithmetic syntax differs between engines.

## 1. Top N per group

```sql
WITH ranked AS (
  SELECT show_id, episode_id, plays,
         ROW_NUMBER() OVER (PARTITION BY show_id ORDER BY plays DESC) AS rn
  FROM episodes
)
SELECT show_id, episode_id, plays
FROM ranked
WHERE rn <= 3;
```

`ROW_NUMBER` returns exactly 3 rows per group. Use `DENSE_RANK` to keep ties.

## 2. Latest row per key (deduplication)

```sql
WITH ranked AS (
  SELECT contact_id, email, updated_at,
         ROW_NUMBER() OVER (PARTITION BY contact_id ORDER BY updated_at DESC) AS rn
  FROM contacts_raw
)
SELECT contact_id, email, updated_at
FROM ranked
WHERE rn = 1;
```

## 3. Running total

```sql
SELECT card_id, tap_ts, fare,
       SUM(fare) OVER (PARTITION BY card_id ORDER BY tap_ts
                       ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_fare
FROM taps;
```

State the frame explicitly. The default frame with ORDER BY treats rows with equal sort values as one group.

## 4. Moving average

```sql
SELECT day_no, orders,
       AVG(orders) OVER (ORDER BY day_no
                         ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS avg_7d
FROM daily_orders;
```

This assumes one row per day. A missing day makes the window cover more than 7 calendar days.

## 5. Change from the previous row

```sql
SELECT day_no, orders,
       orders - LAG(orders) OVER (ORDER BY day_no) AS change_from_previous
FROM daily_orders;
```

## 6. Gaps and islands

```sql
WITH numbered AS (
  SELECT device_id, day_no,
         day_no - ROW_NUMBER() OVER (PARTITION BY device_id ORDER BY day_no) AS grp
  FROM device_days
)
SELECT device_id, MIN(day_no) AS first_day, MAX(day_no) AS last_day, COUNT(*) AS days
FROM numbered
GROUP BY device_id, grp;
```

Consecutive days share the same difference between the day number and the row number.

## 7. Sessionization

```sql
WITH flagged AS (
  SELECT user_id, event_ts,
         CASE WHEN event_ts - LAG(event_ts) OVER (PARTITION BY user_id ORDER BY event_ts) > 1800
              THEN 1 ELSE 0 END AS is_new_session
  FROM page_views
)
SELECT user_id, event_ts,
       SUM(is_new_session) OVER (PARTITION BY user_id ORDER BY event_ts
                                 ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS session_no
FROM flagged;
```

A gap above 1,800 seconds starts a new session. Each user's first session is number 0.

## 8. Rows with no match (anti-join)

```sql
SELECT c.customer_id
FROM customers AS c
WHERE NOT EXISTS (
  SELECT 1 FROM orders AS o WHERE o.customer_id = c.customer_id
);
```

Prefer `NOT EXISTS` to `NOT IN`. If the subquery of a `NOT IN` returns one null, the result is empty.

## 9. Funnel with conditional aggregation

```sql
SELECT
  COUNT(DISTINCT CASE WHEN step = 'quote_started' THEN user_id END) AS started,
  COUNT(DISTINCT CASE WHEN step = 'quote_completed' THEN user_id END) AS completed,
  COUNT(DISTINCT CASE WHEN step = 'policy_bought' THEN user_id END) AS bought
FROM funnel_events;
```

This counts users at each step without checking order. An ordered funnel compares the time of each step per user.

## 10. Point-in-time join

```sql
SELECT s.sale_id, d.region
FROM sales AS s
JOIN dim_store AS d
  ON d.store_id = s.store_id
 AND s.sale_day >= d.valid_from
 AND (s.sale_day < d.valid_to OR d.valid_to IS NULL);
```

Each sale matches the dimension row that was valid on its day. See [slowly changing dimensions](../concepts/slowly-changing-dimensions.md).

## Three checks before saying "done"

1. **Grain.** State what one output row represents, and confirm no join multiplied the rows.
2. **Nulls.** Check each join key and each aggregated column.
3. **Ties.** Say which ranking function was chosen and why.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-sql-patterns) teaches each pattern in its Advanced SQL module, plus retention, cohorts, and query plans.
- Lesson: [Ranking and Ties](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/ranking-and-ties?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-sql-patterns), in Grokking the Data Engineering Interview
- Lesson: [Gaps and Islands](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/gaps-and-islands?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-sql-patterns), in Grokking the Data Engineering Interview
- Lesson: [Sessionizing Events](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/sessionizing-events?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-sql-patterns), in Grokking the Data Engineering Interview
- [Grokking SQL for Tech Interviews](https://www.designgurus.io/course/grokking-sql-for-tech-interviews?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-sql-patterns) covers the foundations and has practice problems.
- [The SQL round](../rounds/sql.md) explains what interviewers look for.
- Free answer: [SQL query optimization interview questions](https://www.designgurus.io/answers/detail/what-are-sql-query-optimization-interview-questions?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-sql-patterns), which covers the follow-up questions about query speed.
