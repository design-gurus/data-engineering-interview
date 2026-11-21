# Delivery Guarantees

A delivery guarantee states how many times a record can affect the output when failures occur. There are three: at-most-once, at-least-once, and exactly-once. The third is the most misunderstood. No system delivers a message exactly once over an unreliable network. What can be achieved is an output that looks as if each record had been processed once.

## The three guarantees

| Guarantee | On failure | Risk | Typical use |
|---|---|---|---|
| At-most-once | The record may be lost | Missing data | Metrics where a small loss is acceptable |
| At-least-once | The record may be processed again | Duplicates | The default for most pipelines |
| Exactly-once (effectively) | The record affects the output once | Higher complexity and some latency | Money, counts that must reconcile |

## Where duplicates come from

A consumer reads a record, writes a result, and then records its position in the log. That position is called an offset.

1. The consumer reads the record at offset 41.
2. It writes the result to the database.
3. It crashes before saving the offset.
4. After the restart it reads offset 41 again and writes the result a second time.

Saving the offset before writing reverses the problem: a crash between the two steps loses the record. The order of those two steps is the entire difference between at-least-once and at-most-once.

## What exactly-once really requires

Two approaches make the output correct despite a repeat.

| Approach | How it works | Requirement |
|---|---|---|
| Idempotent sink | A repeated write has no further effect | Each record carries a stable unique key, and the sink upserts on it |
| Transactional commit | The output and the offset are committed together, or not at all | The sink and the offset store take part in one transaction |

At-least-once delivery with an idempotent sink is the simplest path, and it is the one to propose first.

A stream processing engine applies the second approach internally. It takes a checkpoint, which is a consistent saved copy of its state and its input offsets. After a failure it restores the last checkpoint and replays input from those offsets. The state is then exact. The external output is exact only if the sink is idempotent or transactional as well.

## The boundary of the guarantee

Exactly-once holds inside the boundary of the systems that cooperate. A side effect outside it, such as sending an email or calling a payment provider, can still happen twice. The fix is the same idea: send a unique request key and let the receiver discard repeats.

## Deduplication as a third option

When the sink cannot upsert, the pipeline can remove duplicates itself.

- **In a stream.** Keep the keys seen within a time limit, and drop a record whose key is already present. The limit bounds the memory used.
- **In a batch table.** Keep one row per key with `ROW_NUMBER`, as shown in [SQL patterns](../cheat-sheets/sql-patterns.md).

A worked example: 20 million events a day with a 0.1 percent retry rate produce about 20,000 duplicate events a day. In a revenue table that is a visible error, so the table is keyed on the event identifier and loaded with a merge.

## Questions an interviewer may ask

- "Does the message broker give exactly-once?" Only for flows that stay inside it and use its transactions. Many stream libraries default to at-least-once.
- "What is the unique key?" An identifier created by the producer when the event happens, never one created at ingestion.
- "What does exactly-once cost?" Transactions add latency, and deduplication state uses memory.

## The sentence to say

"I will assume at-least-once delivery and make the sink idempotent by merging on the event identifier, which gives an exactly-once result without distributed transactions."

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-delivery-guarantees) covers exactly-once from stream to table in its Streaming module.
- [Grokking Kafka and Event-Driven Architecture](https://www.designgurus.io/course/grokking-kafka-and-event-driven-architecture?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-delivery-guarantees) covers offsets, consumer groups, and transactions.
- [Idempotency and backfills](idempotency-and-backfills.md) covers the same property for batch jobs.
