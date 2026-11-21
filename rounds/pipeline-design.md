# The Pipeline Design Round

The pipeline design round is the data engineer's system design interview. The candidate designs the path of data from a source to the tables that people query. A request-serving design is judged on latency and availability. A pipeline design is judged on whether the data is correct, how fresh it is, what happens on a rerun, and what it costs.

## A seven-step method for 45 minutes

| Step | Minutes | The question it answers |
|---|---|---|
| 1. Consumers | 4 | Who reads the output, and which questions do they ask? |
| 2. Freshness and volume | 5 | How old may the data be, and how much arrives each day? |
| 3. Sources | 4 | Is each source a database, an event stream, or a file drop? |
| 4. Data model | 7 | What is the grain of the output tables? |
| 5. Flow | 10 | Ingest, raw storage, transformation, serving |
| 6. Failure handling | 8 | Reruns, duplicates, late data, schema changes |
| 7. Cost and monitoring | 4 | What does it cost each month, and how is a failure noticed? |

The last 3 minutes are for the interviewer's follow-up questions.

## The numbers to ask for

Three numbers decide most of the design. Ask for them in the first 10 minutes.

- **Freshness requirement.** A dashboard read each morning can be served by a nightly batch job. A fraud check that must respond in under 1 second cannot.
- **Daily volume.** 50 million events a day at 1 kilobyte each is 50 gigabytes a day, about 18 terabytes a year before compression.
- **Allowed lateness.** If mobile events can arrive 3 days late, the design needs a way to reopen closed days.

## The standard shape

Most answers use the same four layers. Name each layer and state what it guarantees.

| Layer | Content | Guarantee |
|---|---|---|
| Raw | Source data exactly as received, never edited | Anything downstream can be rebuilt from it |
| Cleaned | Parsed, typed, deduplicated records | One row per real event |
| Modeled | Facts and dimensions, or aggregates | Stable definitions for analysts |
| Serving | Tables or an index shaped for one consumer | Fast reads |

## Step 6 is where the round is decided

Many candidates draw the flow and stop. Interviewers then ask these four questions, so answer them before being asked.

1. **What if the job runs twice?** Each run overwrites one partition, so the second run replaces the first instead of adding to it. This property is called [idempotency](../concepts/idempotency-and-backfills.md).
2. **What if data arrives late?** Rerun the last N days on each run, or use a [watermark](../concepts/late-data-and-watermarks.md) in a streaming job.
3. **What if the schema changes?** Keep the raw layer, validate against an agreed schema, and quarantine batches that fail.
4. **What if a bug is found after 3 months?** Recompute the affected partitions from raw data in a controlled backfill.

## Worked example in brief

"Design a daily revenue pipeline for a marketplace." Orders live in an operational database, and the finance team needs figures by 7:00 each morning. Volume is 2 million orders a day. A nightly export lands in the raw layer partitioned by order date. A transformation job overwrites the last 3 daily partitions of the fact table, which absorbs late corrections. A reconciliation check compares the row count and the summed amount against the source, and blocks publication when the difference exceeds 0.1 percent.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-pipeline-design) has a Pipeline Design module and complete multi-lesson designs.
- [Grokking the System Design Interview](https://www.designgurus.io/course/grokking-the-system-design-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-pipeline-design) covers the general design round and its building blocks.
- [Design a clickstream pipeline](../questions/design-a-clickstream-pipeline.md) applies this method to one question.
- Free article: [Data engineering system design](https://www.designgurus.io/blog/data-engineering-system-design?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-pipeline-design), which designs a pipeline from start to finish.
- Free answer: [What a data pipeline is and how to design one](https://www.designgurus.io/answers/detail/what-is-a-data-pipeline-and-how-do-you-design-one-for-processing-big-data-etlelt-systems?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-pipeline-design), which covers ETL and ELT systems for large data volumes.
