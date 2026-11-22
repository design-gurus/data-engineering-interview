# Data Warehouse vs Data Lake vs Lakehouse

A data warehouse is a database built for analytic queries over structured tables. A data lake is a store of files in open formats on inexpensive object storage. A lakehouse is a data lake with a table format added on top, which gives files the transactional behavior of warehouse tables. The three differ in who controls the storage and what guarantees a reader gets.

## The comparison

| Aspect | Warehouse | Data lake | Lakehouse |
|---|---|---|---|
| Storage | Managed by the warehouse | Files in object storage | Files in object storage |
| Data shapes | Structured tables | Any: tables, logs, images | Mostly tables, with raw files beside them |
| Schema | Enforced when data is written | Applied when data is read | Enforced by the table format |
| Atomic changes | Yes | No | Yes |
| Engines that can read it | The warehouse itself | Many | Many |
| Typical users | Analysts with SQL | Engineers and data scientists | Both |

An atomic change either applies completely or does not apply at all. A reader never sees half of it.

## What a plain data lake lacks

A directory of files has no concept of a transaction. Three problems follow.

1. A reader that lists the directory during a write can see some new files and miss others.
2. A failed job can leave partial files behind.
3. Updating or deleting one row means rewriting whole files, with no record of which version is current.

## What a table format adds

A table format, such as Apache Iceberg or Delta Lake, keeps a metadata log beside the data files. Each commit to the log records which files make up the table at that moment.

| Feature | Meaning |
|---|---|
| Atomic commits | A write becomes visible in one step |
| Snapshots | Each version of the table can be read by its identifier |
| Time travel | A query can read the table as it was at an earlier point |
| Schema evolution | Columns can be added or renamed without rewriting files |
| Row-level changes | Updates, deletes, and merges are supported |

Old snapshots depend on old data files. A cleanup job that removes those files also ends the ability to read the snapshots that used them, so the retention settings of the two must agree.

## How to choose

| Situation | Reasonable choice |
|---|---|
| A small team, all structured data, SQL users | Warehouse |
| Large volumes of logs or media, several processing engines | Lakehouse |
| Machine learning teams that read files directly | Lakehouse |
| A strict need to avoid dependence on one vendor | Lakehouse with an open table format |
| Raw landing area for anything, before modeling | Data lake as the raw layer of either design |

A worked example helps here. A team of five analysts with 500 gigabytes of order data gains little from managing files, table maintenance, and compaction. A warehouse serves them with less work. A company with 40 terabytes of event logs, read by SQL, Spark, and model training jobs, benefits from one copy of the data in an open format.

## Cost points to mention

- Warehouses charge for compute by time or by data scanned, so partitioning and clustering reduce the bill directly.
- Object storage is inexpensive per gigabyte, and the cost moves to the compute engines that read it.
- A lakehouse needs maintenance jobs: compaction of small files and expiry of old snapshots.

## The sentence to say

"I will land raw files in object storage and model them in tables with a table format, because two engines need to read the same data. For a single SQL team I would choose a warehouse instead."

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-data-warehouse-vs-data-lake-vs-lakehouse) covers table formats, warehouse costs, and this choice in its Storage and Processing at Scale module.
- Lesson: [Table Formats Iceberg and Delta Lake](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/table-formats-iceberg-and-delta-lake?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-data-warehouse-vs-data-lake-vs-lakehouse), in Grokking the Data Engineering Interview
- [Grokking Database Fundamentals for Tech Interviews](https://www.designgurus.io/course/grokking-database-fundamentals-for-tech-interviews?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-data-warehouse-vs-data-lake-vs-lakehouse) covers the storage engine ideas underneath.
- [Partitioning and file formats](partitioning-and-file-formats.md) explains the files a lake holds.
- Free answer: [Data warehouse vs data lake in system design](https://www.designgurus.io/answers/detail/what-is-a-data-warehouse-vs-a-data-lake-and-how-do-they-fit-into-system-design?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-data-warehouse-vs-data-lake-vs-lakehouse), which shows where each store fits in a larger design.
- Free answer: [How to understand data lakes vs data warehouses for interviews](https://www.designgurus.io/answers/detail/how-to-understand-data-lakes-vs-data-warehouses-for-interviews?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-data-warehouse-vs-data-lake-vs-lakehouse), a comparison written for interview preparation.
