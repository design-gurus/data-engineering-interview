# ETL vs ELT

ETL stands for extract, transform, load: data is reshaped on a separate processing system before it reaches the warehouse. ELT stands for extract, load, transform: data is loaded in its raw form first and reshaped inside the warehouse with SQL. The difference is where the transformation runs and whether the raw data is kept.

## The comparison

| Aspect | ETL | ELT |
|---|---|---|
| Where transformation runs | A separate engine, before loading | Inside the warehouse, after loading |
| Raw data kept in the warehouse | Usually not | Yes |
| Language of transformation | Code in a general language, or a tool | Mostly SQL |
| Fixing a transformation bug | Extract again from the source | Rerun SQL over the raw tables |
| Warehouse storage used | Less | More |
| Warehouse compute used | Less | More |
| Sensitive fields | Can be removed before loading | Arrive in the warehouse unless masked at extraction |

## Why ELT became the common default

Two changes made ELT practical. Cloud warehouses separated storage from compute, so keeping raw data became inexpensive. Their engines also became fast enough to run large transformations in SQL. Keeping the raw layer has a large benefit: any table can be rebuilt after a bug without asking the source system for the data again.

## When ETL is still the right choice

- **Privacy.** A rule may forbid personal data from entering the warehouse. The field must then be removed or hashed before loading.
- **Heavy parsing.** Images, audio, or deeply nested logs are easier to process in code than in SQL.
- **Cost.** Filtering 95 percent of rows before loading avoids paying to store and scan them.
- **A fixed target.** The destination is an operational system that accepts only a finished format.

Many real pipelines combine both. A small transformation before loading removes sensitive fields, and the main modeling runs afterward in SQL.

## A worked comparison

A team loads 200 gigabytes of meter readings each day and discovers that a unit conversion has been wrong for 30 days.

| Approach | The repair |
|---|---|
| ETL with no raw copy | Request 30 days of data again from the source, which may no longer hold it |
| ELT with a raw layer | Correct the SQL and rerun it over 30 daily partitions |

The ELT repair is a few hours of compute. The ETL repair depends on another team and on the source system's retention period.

## The layers in an ELT design

| Layer | Content | Who reads it |
|---|---|---|
| Raw | A faithful copy of the source, append-only | Pipelines only |
| Cleaned | Typed, deduplicated, renamed columns | Engineers |
| Modeled | Facts, dimensions, aggregates | Analysts and dashboards |

Each layer is rebuilt from the one before it. No job edits the raw layer.

## Questions an interviewer may ask

- "Where would you remove personal data?" At extraction, or in a restricted raw zone with column masking.
- "How do you test SQL transformations?" With assertions that return failing rows. See [data quality and testing](data-quality-and-testing.md).
- "What does ELT cost?" Storage for the raw copy, plus warehouse compute for each transformation run.

## The sentence to say

"I will load the raw data first and transform it in the warehouse, so that I can rebuild any table after a bug. I will mask the two sensitive columns at extraction."

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-etl-vs-elt) builds one pipeline both ways and compares the cost.
- [Idempotency and backfills](idempotency-and-backfills.md) explains how the rerun is made safe.
