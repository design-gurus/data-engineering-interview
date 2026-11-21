# Design a Log Analytics Platform

A log analytics platform collects the logs that servers and applications write, stores them, and lets engineers search and aggregate them. The volume is very high and most logs are never read. The design is therefore a cost problem: keep recent logs fast to search, keep old logs inexpensive, and prevent a traffic spike from losing data.

## Clarify first

| Question | Why it matters |
|---|---|
| How much log data per day? | Sizes ingestion and storage |
| How long must logs be kept? | Decides storage tiers |
| What do users do: search for a string, or aggregate? | Decides the index and the file format |
| How soon must a log line be searchable? | Decides the ingestion path |
| Do logs contain personal data? | Decides masking and access control |

## Numbers for this walkthrough

A company runs 5,000 servers that each write about 2 gigabytes of logs a day. That is 10 terabytes a day. Engineers search the last 7 days during incidents. Compliance requires 1 year of retention.

- Recent tier: 7 days at 10 terabytes is 70 terabytes of raw logs.
- Archive tier: 365 days at 10 terabytes is 3,650 terabytes raw. Columnar compression by a factor of about 10 reduces that to roughly 365 terabytes.

## Flow

1. An agent on each server reads log files and sends lines in batches.
2. The batches go to an event log, which acts as a buffer between servers and storage.
3. A stream job parses each line into fields: time, service, level, and message.
4. The parsed records go to two destinations: a search index for the recent tier, and Parquet files in object storage for the archive.
5. A query service sends each query to the index or to the files according to its time range.

## The decisions that matter

**A buffer in front of storage.** During an incident, log volume can rise by a factor of 10 exactly when engineers need logs most. The event log absorbs the spike, and the index consumes it at its own rate. Without the buffer, the index is overloaded and agents drop lines.

**Two storage tiers.**

| Tier | Age | Store | Strength | Cost per gigabyte |
|---|---|---|---|---|
| Recent | 0 to 7 days | Search index | Text search in seconds | High |
| Archive | 8 to 365 days | Compressed columnar files, partitioned by date and service | Aggregates and rare searches | Low |

A full-text index can be as large as the data it indexes. Indexing 1 year instead of 7 days would multiply the most expensive part of the system by about 50.

**Structure at write time.** Parsing each line into fields once allows filtering by service and level without scanning message text. Lines that fail to parse are kept with a flag, never dropped.

**Sampling.** Debug-level logs can be 80 percent of the volume. Keeping all error logs and a sample of debug logs reduces cost sharply. State that this is a decision for the service owners.

## Failure handling

- **Duplicates.** An agent resends a batch after a timeout. A line identifier built from host, file, and byte offset lets the index overwrite the earlier copy.
- **A slow consumer.** Monitor consumer lag on the event log, and alert when lines are more than 5 minutes behind.
- **Retention.** A daily job deletes archive partitions older than 365 days. Partitioning by date makes that a directory removal.

## What the interviewer probes

- "A query over 6 months is slow." It reads the archive. Partition pruning by date and service limits the scan, and the user is shown the expected wait.
- "One service logs 10 times more than the others." Apply a quota per service at the agent or the buffer.
- "A log line contains a customer's email address." Mask known patterns in the parsing job, before either store receives the line.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-log-analytics-platform) covers partitioning, file sizes, and warehouse costs in its Storage and Processing at Scale module.
- [Grokking the System Design Interview](https://www.designgurus.io/course/grokking-the-system-design-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-log-analytics-platform) covers search indexes and message queues.
- [Partitioning and file formats](../concepts/partitioning-and-file-formats.md) explains the archive layout.
