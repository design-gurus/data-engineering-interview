# Idempotency and Backfills

A pipeline job is idempotent when running it twice for the same input produces the same output as running it once. A backfill is a planned recomputation of historical data, usually after a bug fix or a new column. Idempotency is what makes a backfill, and every ordinary retry, safe.

## Why the property matters

Jobs are retried constantly: after a timeout, a node failure, or an operator's manual rerun. A job that appends rows doubles its output on the second run. A revenue table that is 9 percent too high after a retried weekend is the classic incident.

## Four ways to write idempotently

| Method | How it works | Fits |
|---|---|---|
| Partition overwrite | Each run replaces the whole partition for its date | Daily and hourly batch tables |
| Delete then insert | Delete the rows for the run's key range, then insert, in one transaction | Tables without partitions |
| Merge (upsert) | Match on a unique key, update on a match, insert otherwise | Dimension tables and change data |
| Write, audit, publish | Write to a staging location, check it, then swap it in atomically | Tables that many consumers read |

A minimal merge looks like this:

```sql
MERGE INTO dim_customer AS t
USING customer_updates AS s
  ON t.customer_id = s.customer_id
WHEN MATCHED THEN
  UPDATE SET email = s.email, updated_at = s.updated_at
WHEN NOT MATCHED THEN
  INSERT (customer_id, email, updated_at)
  VALUES (s.customer_id, s.email, s.updated_at);
```

Running it twice with the same `customer_updates` leaves the same rows. The source must hold one row per key, or the merge is ambiguous.

## Two conditions that are easy to miss

1. **The run must be defined by its data interval, not by the clock.** A job that selects "the last 24 hours" returns different rows when rerun 3 hours later. A job that selects "order date equals the run's date" does not.
2. **The logic must be deterministic.** Calls to the current time, random values, or an unordered "first row" make two runs differ.

## Planning a backfill

A team finds that a unit conversion has been wrong for 18 months of meter readings. The repair is a backfill of about 550 daily partitions.

| Decision | Recommended practice |
|---|---|
| Scope | List the exact partitions and the downstream tables that depend on them |
| Order | Oldest first when later days depend on earlier ones, otherwise newest first so that current reports are corrected soonest |
| Throttling | Run a fixed number of partitions in parallel, so daily jobs still meet their deadlines |
| Validation | Compare row counts and sums before and after for a sample of days |
| Publication | Write to a new table version and switch readers once, not day by day |
| Communication | Tell consumers which numbers will change and when |

At 6 minutes per partition and 10 partitions in parallel, 550 partitions take about 5.5 hours.

## Backfills in a streaming system

A streaming job has no date partitions to rerun. Two options exist: replay the log from an earlier offset, if retention allows, or run a batch job over the stored raw events and merge its output. The second option is more common, because logs usually retain days and not months.

## Questions an interviewer may ask

- "The job failed after writing half its files." With partition overwrite through an atomic swap, readers never saw the partial output.
- "Late rows arrived for a day already processed." Rerun that partition. See [late data and watermarks](late-data-and-watermarks.md).
- "Downstream tables used the wrong data." Rerun them in dependency order after the source is corrected.

## The sentence to say

"Each run overwrites exactly one date partition, selected by the run's data interval, so a retry or a backfill replaces data and never duplicates it."

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-idempotency-and-backfills) has lessons on idempotent pipelines, backfills, and jobs that can rerun.
- [Orchestration](orchestration.md) explains how a scheduler passes the data interval to a job.
- Free article: [Idempotency in system design](https://www.designgurus.io/blog/grokking-idempotency?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-idempotency-and-backfills), which explains the same property for services and APIs.
