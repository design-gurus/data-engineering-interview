# Four-Week Plan

This plan prepares an engineer who already writes basic SQL for a data engineering loop in four weeks. It assumes about 8 hours a week: 1 hour on each of four weekdays and 4 hours across the weekend. Each week ends with a finish condition, so progress can be checked instead of assumed.

## How the 32 hours are divided

| Skill | Hours | Share |
|---|---|---|
| SQL | 9 | 28 percent |
| Pipeline design and concepts | 10 | 31 percent |
| Data modeling | 6 | 19 percent |
| Coding | 4 | 13 percent |
| Behavioral and mock rounds | 3 | 9 percent |

SQL and pipeline design receive the most time, because they appear in nearly every loop and carry the most weight at mid and senior levels.

## Week 1: SQL

| Day | Work |
|---|---|
| 1 | Read [the SQL round](../rounds/sql.md). Write both worked queries from memory. |
| 2 | [SQL patterns](sql-patterns.md) 1 to 5: ranking, deduplication, running totals, moving averages, `LAG`. |
| 3 | Patterns 6 and 7: gaps and islands, sessionization. |
| 4 | Patterns 8 to 10: anti-join, funnel, point-in-time join. |
| Weekend | Answer SQL questions 1 to 15 in the [question bank](question-bank.md), timed at 10 minutes each. |

**Finish condition:** 12 of the 15 questions solved correctly within the time limit, each tested on a small example.

## Week 2: Data modeling and coding

| Day | Work |
|---|---|
| 1 | [Dimensional modeling](../concepts/dimensional-modeling.md) and [the data modeling round](../rounds/data-modeling.md). |
| 2 | [Slowly changing dimensions](../concepts/slowly-changing-dimensions.md). Draw the type 2 example without notes. |
| 3 | [The coding round](../rounds/coding.md). Write both worked examples from memory. |
| 4 | Coding questions 16 to 25 in the question bank, choosing any four. |
| Weekend | Model three businesses in 40 minutes each: a retailer, a ride service, and a subscription app. Then [design a daily sales warehouse](../questions/design-a-daily-sales-warehouse.md). |

**Finish condition:** for each of the three models, the grain is stated in one sentence and one query is written for each business question.

## Week 3: Pipeline concepts

| Day | Work |
|---|---|
| 1 | [Batch vs streaming](../concepts/batch-vs-streaming.md) and [ETL vs ELT](../concepts/etl-vs-elt.md). |
| 2 | [Idempotency and backfills](../concepts/idempotency-and-backfills.md) and [orchestration](../concepts/orchestration.md). |
| 3 | [Delivery guarantees](../concepts/delivery-guarantees.md) and [late data and watermarks](../concepts/late-data-and-watermarks.md). |
| 4 | [Partitioning and file formats](../concepts/partitioning-and-file-formats.md) and [warehouse vs lake vs lakehouse](../concepts/data-warehouse-vs-data-lake-vs-lakehouse.md). |
| Weekend | [Spark essentials](../concepts/spark-essentials.md), [data quality and testing](../concepts/data-quality-and-testing.md), then [the pipeline design round](../rounds/pipeline-design.md). |

**Finish condition:** for each concept page, say "the sentence to say" from memory and explain one trade-off behind it.

## Week 4: Complete designs and mock rounds

| Day | Work |
|---|---|
| 1 | [Design a clickstream pipeline](../questions/design-a-clickstream-pipeline.md), spoken aloud in 40 minutes. |
| 2 | [Design a change data capture pipeline](../questions/design-a-change-data-capture-pipeline.md). |
| 3 | [Design a real-time fraud feature pipeline](../questions/design-a-real-time-fraud-feature-pipeline.md). |
| 4 | [The behavioral round](../rounds/behavioral.md). Write six stories with a number in each result. |
| Weekend | One full mock loop: a SQL round, a design round, and a behavioral round, 45 minutes each, with a partner or a recording. Review [common mistakes](common-mistakes.md) afterward. |

**Finish condition:** in the recorded design round, reruns, late data, schema change, and cost are all covered without a reminder.

## If there is less time

| Time available | Keep |
|---|---|
| 2 weeks | Week 1 and week 4, plus the idempotency and late data pages |
| 1 week | SQL patterns, the pipeline design round, the clickstream design, and three behavioral stories |
| 2 days | SQL patterns 1, 2, 6, and 7, and the four sentences in [common mistakes](common-mistakes.md) |

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-four-week-plan) follows the same order in more depth, with an assessment after each module and a final exam.
- [Grokking SQL for Tech Interviews](https://www.designgurus.io/course/grokking-sql-for-tech-interviews?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-four-week-plan) is the place to start if joins and GROUP BY still need practice.
- [The interview loop](../rounds/interview-loop.md) explains the format this plan prepares for.
