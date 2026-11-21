# Design a Real-Time Fraud Feature Pipeline

A fraud model scores each payment before it is approved. It needs features, which are numeric inputs such as "number of payments by this card in the last 10 minutes". This question asks for the pipeline that computes those features and serves them within the time limit of a payment. It is the standard streaming design question for data engineers.

## Clarify first

| Question | Why it matters |
|---|---|
| How long may the feature lookup take? | Decides the serving store |
| How fresh must a feature be? | Seconds requires streaming |
| Which windows are needed? | Decides how much state the job holds |
| How is the model trained? | Training data must match serving data |
| What happens if the pipeline is down? | The payment flow needs a fallback |

## Numbers for this walkthrough

The system handles 2,000 payments per second at peak, about 60 million a day. The scoring service allows 20 milliseconds for the feature lookup. Features cover windows of 10 minutes, 1 hour, and 24 hours for each card and each merchant.

## Two kinds of features

| Kind | Example | Computed by | Freshness |
|---|---|---|---|
| Streaming | Payments by this card in the last 10 minutes | A stream job | Seconds |
| Batch | Average payment amount by this card over 90 days | A daily job | One day |

Only features that change the decision within minutes belong in the streaming path. Everything else is cheaper and simpler in batch.

## Flow

1. The payment service publishes each payment event to an event log, keyed by card.
2. A stream job maintains windowed counts and sums per card and per merchant.
3. The job writes each updated value to an online store, a low-latency key-value database.
4. The scoring service reads the features for a card with one key lookup.
5. The same events are also written to object storage as the offline record for training.
6. A daily batch job computes long-window features and loads them into the same online store.

## The decisions that matter

**Consistency between training and serving.** A model trained on features computed one way and served features computed another way performs worse in production. This mismatch is called training-serving skew. Define each feature once, and generate both the online values and the training set from that definition.

**Point-in-time correctness.** A training row for a payment at 10:05 must use feature values as they were at 10:05. Using a later value leaks information from the future into the training set. Store feature values with timestamps, and join each payment to the latest value at or before its event time.

**Window design.** A sliding 24-hour window updated on every event holds a large state. Keeping 1-minute buckets and summing the last 1,440 buckets on read limits the state per key and gives results accurate to 1 minute.

## Failure handling

- **Duplicates.** A restart replays events. Make the count idempotent by tracking the payment identifiers already applied within the window. See [delivery guarantees](../concepts/delivery-guarantees.md).
- **Late events.** Use a short [watermark](../concepts/late-data-and-watermarks.md), such as 30 seconds, because a late feature has no value to a decision already made.
- **Pipeline outage.** The scoring service falls back to batch features and a stricter rule set. Each feature carries its update time, so the service can detect a stale value.

## What the interviewer probes

- "One merchant receives 5 percent of all payments." That key is skewed. Split its aggregation across sub-keys and combine them.
- "How do you add a new feature?" Compute its history from the offline record first, so training data exists on the first day.
- "How do you know that a feature is wrong?" Compare the online value with a batch recomputation for a sample of keys each day.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-real-time-fraud-feature-pipeline) covers windows, watermarks, and exactly-once output in its Streaming module.
- [Grokking Kafka and Event-Driven Architecture](https://www.designgurus.io/course/grokking-kafka-and-event-driven-architecture?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-real-time-fraud-feature-pipeline) covers the event log and keyed partitions.
- [Grokking the System Design Interview](https://www.designgurus.io/course/grokking-the-system-design-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-real-time-fraud-feature-pipeline) covers low-latency key-value stores and caching.
