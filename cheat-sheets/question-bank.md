# Question Bank

Sixty questions, grouped by round. Each has a one-line pointer to the idea that answers it, not a full answer. Work through a group aloud with a timer: 10 minutes for a SQL or coding question, 40 for a design question.

## SQL (15)

1. Return the three best-selling products in each category. *`ROW_NUMBER` or `DENSE_RANK`, and state the tie rule.*
2. Keep the latest record for each customer. *`ROW_NUMBER` ordered by time descending, filter on 1.*
3. Compute a running total of spending per account. *`SUM` with an explicit frame.*
4. Compute a 7-day moving average of daily orders. *`ROWS BETWEEN 6 PRECEDING AND CURRENT ROW`, and mention missing days.*
5. Find each user's longest streak of consecutive login days. *Gaps and islands.*
6. Group page views into sessions with a 30-minute gap. *`LAG` and a running sum.*
7. Find customers who have never placed an order. *`NOT EXISTS` or a left join with a null filter.*
8. Compute week 1 retention for each signup week. *Cohort table joined to activity, with a clear denominator.*
9. Compute conversion at each step of a signup funnel. *Conditional aggregation.*
10. Find the second highest salary in each department. *`DENSE_RANK` equals 2.*
11. Find duplicate rows in a table. *`GROUP BY` the key and `HAVING COUNT(*) > 1`.*
12. Compute each day's change from the previous day. *`LAG`.*
13. Why can a window function not be used in `WHERE`? *`WHERE` runs before window functions. Use a CTE.*
14. What is the difference between `RANK` and `DENSE_RANK`? *`RANK` skips numbers after a tie.*
15. A join returned more rows than the left table has. Why? *The right side has repeated keys.*

See [SQL patterns](sql-patterns.md) for snippets.

## Coding (10)

16. Return the ten most frequent addresses in a log file. *`Counter`, reading line by line.*
17. Parse a CSV file in which fields contain commas. *The `csv` module.*
18. Merge overlapping time intervals. *Sort by start, then one pass.*
19. Process a file larger than memory. *Generators, chunks, and an external sort.*
20. Group records by key and total a field. *`defaultdict`.*
21. Remove duplicate records while keeping the first of each. *A set of keys already seen.*
22. Flatten a nested JSON record into columns. *Recursion with a key prefix.*
23. Find the top K items in a stream. *A heap of size K.*
24. Write a download job that is safe to rerun. *Temporary file, then atomic rename.*
25. Convert local timestamps to UTC. *Convert at the first step and keep the offset.*

## Data modeling (12)

26. Model sales for a retail chain. *Declare the grain first.*
27. What is the grain of a fact table? *The meaning of one row.*
28. What is the difference between a fact and a dimension? *Measurements and their context.*
29. Compare a star schema with a snowflake schema. *Fewer joins against less repetition.*
30. Explain slowly changing dimension types 1, 2, and 3. *Overwrite, new row, previous-value column.*
31. Why does a type 2 dimension need a surrogate key? *One source identifier now has several rows.*
32. How would you model daily inventory? *A periodic snapshot, which is semi-additive.*
33. A fact arrives before its dimension row. What do you do? *An "unknown" row, corrected later.*
34. When would you choose one wide table? *Simple queries, with a cost when attributes change.*
35. How do you store a ratio such as conversion rate? *Store the numerator and the denominator.*
36. How would you model an order that moves through several states? *An accumulating snapshot.*
37. Model events for a mobile app. *A wide event table, or a star, with the trade-off stated.*

## Pipeline design (15)

38. Design a clickstream pipeline. *Duplicates, late events, sessions.*
39. Design a pipeline that copies a database to a warehouse. *Change data capture and merge.*
40. When would you choose streaming over batch? *The freshness requirement.*
41. What makes a pipeline idempotent? *Overwrite or merge, selected by data interval.*
42. How would you backfill 18 months of data? *Scope, order, throttle, validate, publish once.*
43. How do you handle late-arriving data? *A lookback window or a watermark.*
44. Explain exactly-once processing. *At-least-once delivery and an idempotent sink.*
45. Compare ETL with ELT. *Where the transformation runs, and whether raw data is kept.*
46. Compare a warehouse, a data lake, and a lakehouse. *Storage control and transactional guarantees.*
47. How would you choose a partition column? *Common in filters, with moderate distinct values.*
48. What is the small files problem? *A fixed cost per file. Compact.*
49. Why is Parquet faster than CSV for analytics? *Column pruning and statistics.*
50. A Spark job has one task that runs far longer than the rest. Why? *Skew.*
51. How do you test a data pipeline? *Queries that return failing rows, before publication.*
52. An upstream job was late and yours ran on incomplete data. How do you prevent it? *Depend on data, not on the clock.*

## Behavioral (8)

53. Describe a time when published data was wrong.
54. Describe a pipeline you made more reliable.
55. Describe a disagreement with an analyst or a product manager.
56. Describe a migration you led.
57. Describe a time you traded quality for a deadline.
58. Describe a mistake you made and what changed afterward.
59. Describe how you explained a data problem to a non-technical person.
60. Describe a time you said no to a data request.

Each behavioral answer needs a measured result. See [the behavioral round](../rounds/behavioral.md).

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-question-bank) answers these questions in full lessons, with assessments for each module.
- [Grokking SQL for Tech Interviews](https://www.designgurus.io/course/grokking-sql-for-tech-interviews?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-question-bank) provides SQL practice problems.
- [Grokking Modern Behavioral Interview](https://www.designgurus.io/course/grokking-behavioral-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-question-bank) covers the behavioral questions.
- Free answer: [The top interview questions for data engineers](https://www.designgurus.io/answers/detail/what-are-the-top-interview-questions-for-data-engineers?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-question-bank), a second list to practice with.
