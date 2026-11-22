# Data Quality and Testing

Data quality means that a table is correct enough for the decision it supports. In an interview, a vague promise to "add validation" earns little. A strong answer names specific checks, says where each one runs, and says what happens when one fails.

## Five dimensions, each as a measurable check

| Dimension | The question | An example check |
|---|---|---|
| Freshness | Did the data arrive on time? | The newest `loaded_at` is under 2 hours old |
| Volume | Did the expected amount arrive? | Today's row count is within 20 percent of the 7-day average |
| Validity | Are values allowed? | `status` is one of four known values, and `amount` is not negative |
| Uniqueness | Is each key present once? | No `order_id` appears twice |
| Consistency | Do related tables agree? | Every `customer_id` in orders exists in customers |

## Tests as queries that return failing rows

A data test is a query that returns the rows that break a rule. Zero rows means the test passes.

```sql
-- Uniqueness: any order_id that appears more than once
SELECT order_id, COUNT(*) AS copies
FROM fact_orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

```sql
-- Consistency: orders that point to a missing customer
SELECT o.order_id
FROM fact_orders AS o
LEFT JOIN dim_customer AS c
  ON c.customer_id = o.customer_id
WHERE c.customer_id IS NULL;
```

The same form works for null checks, accepted values, and ranges.

## Where checks run

| Stage | What is checked | On failure |
|---|---|---|
| At ingestion | Schema, required fields, file completeness | Quarantine the batch |
| After transformation | Uniqueness, relationships, business rules | Stop before publication |
| After publication | Freshness, volume, drift in key metrics | Alert the owning team |

The most useful pattern is write, audit, publish: write the new data to a staging location, run the checks, and replace the published table only if they pass. Consumers then keep yesterday's correct data instead of receiving today's incorrect data.

## Blocking checks and warning checks

Not every failure should stop the pipeline. Classify each check.

| Class | Example | Action |
|---|---|---|
| Blocking | Duplicate keys in a revenue table | Stop and page the owner |
| Warning | Row count 15 percent below average | Publish and notify |

Too many blocking checks cause missed deadlines. Too many warnings are ignored. A table with 5 blocking checks and 10 warnings is typical.

## Reconciliation

Reconciliation compares the pipeline's output with its source. For a payments table, compare two numbers for each day: the row count and the summed amount.

A worked example: the source reports 1,204,318 payments totaling 48.2 million dollars. The warehouse shows 1,204,318 rows and the same total, so the load is complete. A difference of 140,000 rows would suggest a duplicated batch. Set a tolerance, such as 0.1 percent, when the source is still changing at the time of comparison.

## Data contracts

A data contract is an agreement between the team that produces data and the teams that consume it. It states the schema, the meaning of each field, and the freshness that is promised. Adding an optional column is a safe change. Renaming or removing a column, or changing a type, breaks consumers and needs a notice period.

## Questions an interviewer may ask

- "A column was renamed upstream and a dashboard shows zero." A schema check at ingestion would have quarantined the batch.
- "Volume is naturally lower on weekends." Compare with the same weekday in earlier weeks.
- "Who is paged?" The team that owns the failing table, with lineage showing which downstream tables are affected.

## The sentence to say

"I will stage each load, run uniqueness and reconciliation checks as blocking tests, and publish only when they pass, so consumers never read a bad day."

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-data-quality-and-testing) has a Data Quality and Reliability module with five lessons.
- Lesson: [What Data Quality Means](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/what-data-quality-means?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-data-quality-and-testing), in Grokking the Data Engineering Interview
- Lesson: [Writing Data Tests](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/writing-data-tests?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-data-quality-and-testing), in Grokking the Data Engineering Interview
- [Design a data quality monitoring system](../questions/design-a-data-quality-monitoring-system.md) turns these checks into a platform.
- Free answer: [How to handle schema drift in JSON, Protobuf, and Avro](https://www.designgurus.io/answers/detail/how-would-you-handle-schema-drift-in-jsonprotobufavro?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-data-quality-and-testing), which covers the schema changes that a data contract guards against.
