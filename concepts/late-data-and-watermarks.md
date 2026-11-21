# Late Data and Watermarks

Late data is an event that reaches the pipeline after the period it belongs to has already been processed. A watermark is a streaming system's estimate of how far event time has progressed, which it uses to decide when a time window is complete. Both exist because the time an event happened and the time it arrived are different.

## Two clocks

| Clock | Meaning | Example |
|---|---|---|
| Event time | When the event happened at the source | A phone records a tap at 10:59 |
| Processing time | When the pipeline received it | The phone reconnects and sends it at 11:20 |

Grouping by processing time is simple and gives wrong counts, because the 10:59 tap would be counted in the 11:00 hour. Grouping by event time gives correct counts and forces a decision about how long to wait.

## How a watermark works

The most common rule is: watermark equals the largest event time seen so far, minus a fixed delay.

A worked example with a 10-minute delay and 1-hour windows:

| Event arrives with event time | Largest event time seen | Watermark | Effect |
|---|---|---|---|
| 10:58 | 10:58 | 10:48 | The 10:00 window is open |
| 11:07 | 11:07 | 10:57 | Still open |
| 10:59 | 11:07 | 10:57 | Accepted into the 10:00 window |
| 11:12 | 11:12 | 11:02 | The 10:00 window closes and its result is emitted |
| 10:55 | 11:12 | 11:02 | Late: the window is already closed |

The window from 10:00 to 11:00 closes when the watermark passes 11:00. The last event in the table arrived after that point.

## The trade-off

| A longer delay | A shorter delay |
|---|---|
| Fewer events are late | More events are late |
| Results are emitted later | Results are emitted sooner |
| More state is held in memory | Less state is held |

Choose the delay from measured data. If 99 percent of events arrive within 2 minutes and 99.9 percent within 10 minutes, a 10-minute delay drops about 1 event in 1,000.

## What to do with an event that is late

1. **Drop it** and count how many were dropped.
2. **Allow lateness.** Keep the window's state for an extra period and emit a corrected result when a late event arrives. The consumer must accept updates.
3. **Send it to a side output**, a separate stream or table that a batch job later merges into the result.

## Late data in a batch pipeline

Batch pipelines have the same problem without the vocabulary. A daily job that runs at 2:00 misses events for yesterday that arrive at 9:00.

| Technique | How it works |
|---|---|
| Lookback window | Each run reprocesses the last N days, not only yesterday |
| Partition by event date | Late rows are written to the day they belong to, when that day is reprocessed |
| Restatement policy | Consumers are told that figures are provisional for N days and final afterward |

With a 3-day lookback, each day is computed 3 times. Compute cost roughly triples for that stage, which is the cost of correct numbers. The job must be [idempotent](idempotency-and-backfills.md) for this to work.

## Questions an interviewer may ask

- "One device sends events dated a week ahead." A watermark based on the maximum would jump forward and close every window. Validate timestamps, and reject values far in the future.
- "A source stops sending." Its watermark stops, and windows that depend on it never close. Mark idle sources so they are ignored.
- "Finance needs final numbers." Publish a daily figure as provisional and mark it final after the lookback period.

## The sentence to say

"I will group by event time with a ten-minute watermark, based on the measured arrival delay, and send anything later to a side table that a nightly job merges."

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-late-data-and-watermarks) computes a watermark step by step and covers late data in batch pipelines.
- [Grokking Kafka and Event-Driven Architecture](https://www.designgurus.io/course/grokking-kafka-and-event-driven-architecture?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-late-data-and-watermarks) covers the event log underneath.
- [Batch vs streaming](batch-vs-streaming.md) explains when a streaming design is needed at all.
