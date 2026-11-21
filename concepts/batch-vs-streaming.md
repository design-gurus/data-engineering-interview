# Batch vs Streaming

Batch processing handles a bounded set of data on a schedule, such as all of yesterday's orders at 2:00. Stream processing handles each record, or each small group of records, shortly after it arrives. The choice follows from one number: how old the data may be when someone uses it. That number is the freshness requirement.

## The comparison

| Aspect | Batch | Streaming |
|---|---|---|
| Input | Bounded: a day, an hour, a file | Unbounded: events without an end |
| Freshness | Minutes to a day | Under a second to a few minutes |
| Correctness | Simple, because the input is complete | Harder, because data arrives late and out of order |
| Reprocessing | Rerun the job for a date | Replay the log, or run a separate batch job |
| Operating cost | Compute runs only during the job | Compute runs all day |
| Failure handling | Rerun the failed run | Restore state from a checkpoint |

A third option sits between them. Micro-batch processing runs a small batch job every few seconds or minutes. It gives freshness of about 1 minute with most of the simplicity of batch.

## Choosing from the freshness requirement

| The consumer needs data that is | Choose |
|---|---|
| Up to 1 day old | Daily batch |
| Up to 1 hour old | Hourly batch |
| Up to 5 minutes old | Micro-batch |
| Under 10 seconds old | Streaming |

Ask what decision the data supports. A finance report read once each morning gains nothing from data that is 5 seconds old. A fraud check that approves a payment needs it.

## A worked comparison

A utility collects readings from 2 million electricity meters every 15 minutes. That is 96 readings per meter per day, or 192 million readings a day.

- **Billing** needs monthly totals. A daily batch job is enough, and compute runs for perhaps 1 hour a day.
- **Outage detection** needs to notice a silent meter within 2 minutes. That requires a streaming job running 24 hours a day.

The same data therefore feeds two pipelines with different designs. Streaming costs more here, because its compute runs 24 times longer each day, and the outage use case justifies that cost while billing does not.

## What streaming adds to the design

A streaming answer must address four subjects that a batch answer can skip.

1. **Event time and processing time.** Event time is when something happened. Processing time is when the system received it. Results should be grouped by event time.
2. **Late data.** A [watermark](late-data-and-watermarks.md) decides how long to wait for late events.
3. **State.** A count over a 1-hour window must be held in memory or on disk and saved in checkpoints.
4. **Delivery guarantees.** A restart can process a record twice. See [delivery guarantees](delivery-guarantees.md).

## Lambda and Kappa in one paragraph

A Lambda architecture runs a streaming path for fresh results and a batch path that later replaces them with corrected results. It gives two codebases to keep consistent. A Kappa architecture uses the streaming path alone and reprocesses history by replaying the log. It gives one codebase and requires a log that retains enough history.

## The sentence to say

"The freshness requirement is one day, so I will use a daily batch job. If it drops below five minutes, I would move this stage to streaming and accept the higher operating cost."

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-batch-vs-streaming) covers this decision in its Pipeline Design and Streaming modules.
- [Grokking Kafka and Event-Driven Architecture](https://www.designgurus.io/course/grokking-kafka-and-event-driven-architecture?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-batch-vs-streaming) covers the log that streaming pipelines read from.
- [Grokking the System Design Interview](https://www.designgurus.io/course/grokking-the-system-design-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-batch-vs-streaming) covers batch and stream processing in a general design round.
- Free answer: [Batch processing vs stream processing](https://www.designgurus.io/answers/detail/what-is-batch-processing-vs-stream-processing-and-when-should-you-use-each?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-batch-vs-streaming), which explains when to use each approach.
- Free article: [Kafka Streams vs Apache Flink vs Apache Storm](https://www.designgurus.io/blog/kafka-streams-apache-flink-apache-storm?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-batch-vs-streaming), a comparison of three stream processing engines.
