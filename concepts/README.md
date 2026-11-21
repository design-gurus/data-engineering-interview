# Concepts

Each page explains one decision that data engineering interviews return to, with the reasoning behind it and the sentence to say in the room.

| Page | The decision |
|---|---|
| [Batch vs streaming](batch-vs-streaming.md) | How fresh the data must be, and what that costs |
| [ETL vs ELT](etl-vs-elt.md) | Whether to transform before or after loading |
| [Dimensional modeling](dimensional-modeling.md) | How to shape tables for analysis |
| [Slowly changing dimensions](slowly-changing-dimensions.md) | Whether to keep the history of an attribute |
| [Warehouse vs lake vs lakehouse](data-warehouse-vs-data-lake-vs-lakehouse.md) | Where the data lives |
| [Partitioning and file formats](partitioning-and-file-formats.md) | How files are laid out and encoded |
| [Idempotency and backfills](idempotency-and-backfills.md) | How a job runs twice without harm |
| [Delivery guarantees](delivery-guarantees.md) | How many times each record is processed |
| [Late data and watermarks](late-data-and-watermarks.md) | When a time window is considered complete |
| [Data quality and testing](data-quality-and-testing.md) | How bad data is stopped before publication |
| [Orchestration](orchestration.md) | How jobs are scheduled and ordered |
| [Spark essentials](spark-essentials.md) | Why a distributed job is slow |

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-readme) teaches each of these decisions with a running example and diagrams.
