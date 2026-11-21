# Design a Clickstream Pipeline

A clickstream pipeline collects the events that users generate in a website or an app, such as page views and taps, and turns them into tables for analysis. It is the most common data pipeline design question. The difficulty is not the flow. It is volume, duplicate events, late events from mobile devices, and the grouping of events into sessions.

## Clarify first

| Question | Why it matters |
|---|---|
| Who uses the output, and for what? | Dashboards need daily aggregates, while analysts need event-level tables |
| How fresh must it be? | A daily batch job and a streaming job are different designs |
| How many events per day? | Decides storage layout and cost |
| How late can events arrive? | Decides the lookback period |
| Is there personal data in the events? | Decides masking and retention |

## Numbers for this walkthrough

A news reading app has 5 million daily active users who each produce about 40 events. That is 200 million events a day. At 1 kilobyte each, raw volume is 200 gigabytes a day and about 73 terabytes a year. Columnar compression typically reduces that by a factor of 5 to 10. Dashboards are read each morning, so a daily batch job with an hourly raw landing is enough.

## Data model

| Table | Grain | Purpose |
|---|---|---|
| `events_raw` | One row per received event, unedited | Rebuild anything |
| `events` | One row per unique event | Event-level analysis |
| `sessions` | One row per session | Engagement metrics |
| `daily_article_stats` | One row per article per day | Dashboards |

Each event carries an `event_id` created on the device, a `user_id`, an event time, and a receive time.

## Flow

1. Apps send events in small batches to a collection service, which writes them to an event log.
2. A loader writes the log to object storage each hour as raw files, partitioned by receive date.
3. A daily job reads raw files, removes duplicates and bot traffic, and writes `events` partitioned by event date.
4. A second job builds `sessions`, and a third builds the daily aggregates.
5. Quality checks run before the aggregates are published.

## The decisions that matter

**Duplicates.** Devices retry after a timeout, so the same event arrives twice. Keep one row per `event_id`. The identifier must be created on the device, because an identifier assigned at ingestion differs for each copy.

**Late events.** A phone without a connection sends its events hours or days later. Partition `events` by event date, and reprocess the last 3 days on every run. Each run overwrites those partitions, so the job stays [idempotent](../concepts/idempotency-and-backfills.md).

**Sessions.** A session is a run of events from one user with no gap longer than 30 minutes. Compute it with `LAG` and a running sum, as shown in [SQL patterns](../cheat-sheets/sql-patterns.md). A session can cross midnight, so the session job must read the previous day as well.

## What the interviewer probes

- "The app released a version that renamed a field." Validate against a schema at ingestion, and quarantine events that fail.
- "A bug double-counted views for two weeks." Rebuild the 14 affected partitions from `events_raw`.
- "Product now wants numbers within 5 minutes." Add a streaming job for a small set of counters, and keep the batch tables as the source of record.
- "What does a query cost?" Partition pruning limits a one-day query to about 1/365 of a year's data.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-clickstream-pipeline) designs this pipeline across three lessons, including the monthly bill.
- [Late data and watermarks](../concepts/late-data-and-watermarks.md) explains the lookback period.
- [Grokking Kafka and Event-Driven Architecture](https://www.designgurus.io/course/grokking-kafka-and-event-driven-architecture?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-clickstream-pipeline) covers the event log at the front of the flow.
- Free article: [Designing a real-time event pipeline with Kafka, Flink, and Elasticsearch](https://www.designgurus.io/blog/designing-event-pipeline-with-kafka-flink-elasticsearch?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-clickstream-pipeline), a worked design for a similar event flow.
