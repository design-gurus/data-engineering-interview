# Design a Change Data Capture Pipeline

Change data capture (CDC) is the practice of copying every insert, update, and delete from an operational database to another system, usually a warehouse. The question tests whether the candidate understands that the warehouse table must match the source table despite changes arriving out of order, twice, or as deletes.

## Clarify first

| Question | Why it matters |
|---|---|
| How fresh must the warehouse copy be? | Minutes suggests log-based capture, a day allows a nightly extract |
| Does the warehouse need history, or only the current state? | Decides between a mirror table and a history table |
| Are rows deleted in the source? | Query-based capture cannot see deletes |
| How large are the tables, and how fast do they change? | Sizes the first full copy and the ongoing flow |
| May the source database carry extra load? | Limits how the first copy is taken |

## Two ways to capture changes

| Aspect | Query-based | Log-based |
|---|---|---|
| How | Select rows where `updated_at` is newer than the last run | Read the database's transaction log |
| Sees deletes | No | Yes |
| Sees every intermediate change | No, only the latest state | Yes |
| Load on the source | A query per run | Low |
| Setup effort | Low | Higher: log settings and a connector |

Propose log-based capture when deletes or freshness under an hour are required.

## Numbers for this walkthrough

A car rental company has a reservations table of 80 million rows that receives about 3 million changes a day. Analysts need the warehouse copy to be at most 15 minutes old.

## Flow

1. A connector takes an initial snapshot, which is a full copy of the table.
2. The connector then reads the transaction log and publishes one event per change to an event log, keyed by primary key.
3. Each event holds the operation type, the row after the change, and the position in the source log.
4. Every 10 minutes a job merges the new events into the warehouse table.
5. A daily check compares row counts and a checksum per date with the source.

## The decisions that matter

**Ordering.** Two updates to one reservation must be applied in source order. Keying events by primary key keeps them in one partition, which preserves their order. Within a merge batch, keep only the latest event per key, chosen by the source log position.

**Applying changes.** The merge is an upsert with a delete branch.

```sql
MERGE INTO reservations AS t
USING latest_changes AS s
  ON t.reservation_id = s.reservation_id
WHEN MATCHED AND s.op = 'delete' THEN DELETE
WHEN MATCHED THEN
  UPDATE SET status = s.status, updated_at = s.updated_at
WHEN NOT MATCHED AND s.op <> 'delete' THEN
  INSERT (reservation_id, status, updated_at)
  VALUES (s.reservation_id, s.status, s.updated_at);
```

The merge is idempotent. Replaying the same batch leaves the table unchanged, so at-least-once [delivery](../concepts/delivery-guarantees.md) is sufficient.

**History.** If analysts need past states, also append every event to a history table, or maintain a type 2 table with validity dates.

## What the interviewer probes

- "A column is added in the source." The connector emits the new field. The pipeline adds the column to the target and leaves old rows null.
- "The connector was down for 2 days and the database discarded its log." The position is lost, so a new snapshot is required. Monitor connector lag, and keep log retention longer than the longest expected outage.
- "The snapshot of 80 million rows overloads the source." Read from a replica, or snapshot in chunks by primary key range.
- "A delete arrives before its insert." Within a batch, the latest event per key decides the result. A delete for a missing row is ignored.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-change-data-capture-pipeline) designs CDC into a warehouse across three lessons, including snapshots and failures.
- [Grokking Kafka and Event-Driven Architecture](https://www.designgurus.io/course/grokking-kafka-and-event-driven-architecture?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-change-data-capture-pipeline) covers keyed partitions and ordering.
- [Grokking Database Fundamentals for Tech Interviews](https://www.designgurus.io/course/grokking-database-fundamentals-for-tech-interviews?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-change-data-capture-pipeline) covers the transaction log that capture reads.
