# Orchestration

Orchestration is the scheduling and coordination of pipeline jobs: which job runs, when, after which other jobs, and what happens on failure. An orchestrator, such as Apache Airflow, does not process the data itself. It starts tasks on other systems and tracks their state.

## The core vocabulary

| Term | Meaning |
|---|---|
| Task | One unit of work, such as a single SQL statement or one Spark job |
| DAG | A directed acyclic graph: tasks connected by dependencies, with no cycles |
| Schedule | The rule that starts a run, such as daily at 2:00 |
| Data interval | The period of data a run is responsible for |
| Retry | An automatic new attempt after a failure |
| Sensor | A task that waits for a condition, such as a file arriving |
| Backfill | Running the DAG for past data intervals |

## The data interval is the key idea

A run scheduled at 2:00 on Tuesday usually processes Monday's data. The orchestrator passes that interval to the task as a parameter. A task that uses the parameter, and never the current clock time, gives the same output whenever it runs. This is what makes retries and [backfills](idempotency-and-backfills.md) correct.

```sql
-- The orchestrator fills in the two interval values for each run
DELETE FROM daily_revenue
WHERE order_date >= :interval_start AND order_date < :interval_end;

INSERT INTO daily_revenue (order_date, revenue)
SELECT order_date, SUM(net_amount)
FROM fact_orders
WHERE order_date >= :interval_start AND order_date < :interval_end
GROUP BY order_date;
```

## A small DAG

```
extract_orders ──┐
                 ├──> build_fact_orders ──> check_quality ──> publish
extract_refunds ─┘
```

The two extracts run in parallel. The fact table waits for both. Publication waits for the quality check, so a failed check stops bad data from reaching consumers.

## Three ways to start a pipeline

| Trigger | How it works | Weakness |
|---|---|---|
| Time | Start at a fixed time | Runs even if the upstream data is not ready |
| Sensor | Wait until an upstream file or partition exists | Holds a worker while waiting, and needs a timeout |
| Data event | Start when an upstream table is updated | Requires producers to publish that event |

A fixed time works until an upstream job runs late. Then the downstream job processes incomplete data without any error. Dependency on the data, not on the clock, removes that failure.

## Retries and timeouts

| Setting | A reasonable starting value | Reason |
|---|---|---|
| Retries | 2 or 3 | Most failures are temporary |
| Delay between retries | 5 minutes, increasing | Gives the failing system time to recover |
| Task timeout | 2 to 3 times the normal duration | Stops a stuck task from blocking the DAG |
| Deadline alert | Before consumers need the data | A late pipeline is noticed before a missing report is |

Retries are safe only when tasks are idempotent.

## What an orchestrator should not do

- **Process data in its own workers.** Heavy work belongs in the warehouse or a processing engine.
- **Hold business logic in scheduling code.** Keep transformations in SQL or job code that can be tested alone.
- **Pass large data between tasks.** Pass a location, such as a table name or a path.

## A worked example of a deadline

Finance needs revenue by 7:00. The pipeline normally takes 50 minutes. Starting at 2:00 leaves time for two retries of the longest task and a manual fix. An alert at 5:30 for any unfinished run gives the on-call engineer 90 minutes.

## Questions an interviewer may ask

- "The upstream table was late and your job ran anyway." Replace the time trigger with a sensor or a data event.
- "How do you rerun one failed day?" Clear that run, and the task reprocesses its own interval.
- "A DAG has 400 tasks and is slow to schedule." Group related steps, and move loops into the processing engine.

## The sentence to say

"Each task receives its data interval from the scheduler, waits for its upstream partition, and can be retried safely because it overwrites only that interval."

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-orchestration) covers orchestration with Airflow and scheduling between pipelines.
- [Data quality and testing](data-quality-and-testing.md) explains the check that guards publication.
