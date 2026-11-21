# Partitioning and File Formats

Two layout decisions control how much data an analytic query must read. The file format decides how values are arranged inside a file. Partitioning decides how files are grouped into directories. A query is fast when it can skip most files and most columns.

## Row storage and columnar storage

A row format stores all the fields of one record together. A columnar format stores all the values of one column together.

| Aspect | Row format | Columnar format |
|---|---|---|
| Reading 3 of 50 columns | Reads every field of every row | Reads only the 3 columns |
| Writing one record | One small append | Buffered, then written in groups |
| Compression | Moderate | High, because similar values sit together |
| Best for | Record-by-record transfer and writes | Analytic scans and aggregates |

## The three formats to know

| Format | Layout | Schema | Typical use |
|---|---|---|---|
| Avro | Row | Stored with the data, supports evolution | Messages in a stream, records between services |
| Parquet | Columnar | Stored in the file footer | The default for analytic tables |
| ORC | Columnar | Stored in the file | Analytic tables, common in Hive-based systems |

A common pipeline uses both kinds: Avro for events in transit and Parquet for tables at rest.

## How a Parquet file lets a reader skip data

A Parquet file is divided into row groups, and each row group holds one chunk per column. The footer stores statistics for each chunk, including the minimum and maximum value.

- **Column pruning.** A query that needs 3 columns reads 3 chunks per row group.
- **Predicate pushdown.** For `WHERE amount > 1000`, the reader skips any row group whose maximum `amount` is below 1,000.

Skipping works well only when similar values are stored near each other, so sorting data on a commonly filtered column before writing improves it.

## Partitioning

Partitioning places rows with the same value of a column in the same directory, such as `event_date=03-14/`. A query that filters on that column reads only the matching directories. This is called partition pruning.

| A good partition column | A poor partition column |
|---|---|
| Appears in most query filters | Rarely filtered |
| Has tens to thousands of distinct values | Has millions of distinct values, such as a user identifier |
| Produces partitions of at least hundreds of megabytes | Produces partitions of a few kilobytes |

Date is the usual first choice, because most queries filter on time and most pipelines load by day.

## The small files problem

Every file has a fixed cost to list, open, and read its footer. A table made of many tiny files is slow even when the total data is small.

A worked example: 100 gigabytes a day partitioned by date and by 10,000 customer identifiers gives 10,000 directories a day, with files of about 10 megabytes each. After one year the table holds over 3.6 million files. Partitioning by date alone and sorting by customer inside each file gives about 400 files a day of 256 megabytes.

A practical target is files between 128 megabytes and 1 gigabyte. A compaction job merges small files into larger ones on a schedule.

## Questions an interviewer may ask

- "Why not partition by user?" Too many distinct values, which creates millions of small files.
- "The query filters on a column that is not the partition column." Sort or cluster on it, so file statistics can skip data.
- "Streaming writes produce a file every minute." Add compaction, or write less often.

## The sentence to say

"I will store the table as Parquet, partitioned by event date and sorted by customer, with a daily compaction job to keep files near 256 megabytes."

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-partitioning-and-file-formats) opens a Parquet file and reads its metadata in the Storage and Processing at Scale module.
- [Grokking Database Fundamentals for Tech Interviews](https://www.designgurus.io/course/grokking-database-fundamentals-for-tech-interviews?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-partitioning-and-file-formats) covers compression and encoding.
- [Spark essentials](spark-essentials.md) explains how an engine reads these files.
