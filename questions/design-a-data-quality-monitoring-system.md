# Design a Data Quality Monitoring System

This question asks for a platform that checks hundreds of tables and tells the right team when one is wrong. Running a check is the simple part. The design problem is deciding which checks exist, avoiding alerts that people learn to ignore, and showing which downstream tables a failure affects.

## Clarify first

| Question | Why it matters |
|---|---|
| How many tables and pipelines? | Decides how much must be automatic |
| Who defines the checks? | Table owners, a central team, or both |
| Should a failure stop a pipeline? | Separates blocking checks from monitoring |
| How fast must a problem be detected? | Decides when checks run |
| Is there lineage information? | Needed to find affected consumers |

Lineage is the record of which tables are built from which other tables.

## Numbers for this walkthrough

A company has 3,000 warehouse tables, of which 400 feed dashboards or external reports. About 1,500 pipeline runs complete each day.

## Two layers of checks

| Layer | Defined by | Covers | Examples |
|---|---|---|---|
| Automatic | The platform, for every table | Freshness, volume, schema | Last load time, row count against history, columns added or removed |
| Declared | The table owner | Business rules | Uniqueness, relationships, accepted values, reconciliation |

The automatic layer needs no configuration, so all 3,000 tables are covered from the first day. Owners add declared checks to the 400 important tables.

## Components

1. **Check definitions** stored in version control beside the pipeline code.
2. **A runner** that executes each check as a SQL query in the warehouse and returns failing rows.
3. **A results store** holding every result with its table, check, time, and measured value.
4. **An anomaly detector** that compares today's measurements with history.
5. **A router** that sends each alert to the owning team.
6. **A lineage graph** that lists the downstream tables of a failed table.

## Flow

When a pipeline finishes loading a table, it calls the runner. Blocking checks run before publication, following the write, audit, publish pattern described in [data quality and testing](../concepts/data-quality-and-testing.md). Monitoring checks run after publication and on a schedule, which also catches a table that silently stopped updating.

## The decisions that matter

**Thresholds.** A fixed rule such as "at least 1 million rows" breaks as the business grows. Compare with the same weekday in the previous 4 weeks, and alert when the value differs by more than a set number of standard deviations.

**Alert volume.** With 3,000 tables and 5 automatic checks each, a 1 percent false alert rate gives 150 false alerts a day. Nobody reads that many. Three measures reduce it: group alerts by root cause using lineage, send low-severity results to a daily summary, and page only for tables marked critical.

**Cost.** Checks are queries, and queries cost money. Run them on the newly loaded partition, not on the whole table. Collect row counts and null rates from table metadata where the warehouse provides it.

## What an alert must contain

| Field | Example |
|---|---|
| What failed | `fact_orders`: 14,200 duplicate `order_id` values |
| When | The load for yesterday's partition |
| Affected consumers | 3 dashboards and 12 downstream tables |
| Owner | The orders data team |
| Sample | 5 failing rows |

## What the interviewer probes

- "An upstream table fails and 40 downstream checks fail too." Use lineage to report one incident at the root and suppress the rest.
- "Teams disable noisy checks." Track each check's alert history, and review checks that fire often without a confirmed problem.
- "How is the platform's own value measured?" Count incidents found by checks against incidents first reported by consumers.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-data-quality-monitoring-system) covers data tests, contracts, reconciliation, and recovery in its Data Quality and Reliability module.
- Lesson: [Reconciliation and Anomaly Checks](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/reconciliation-and-anomaly-checks?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-data-quality-monitoring-system), in Grokking the Data Engineering Interview
- Lesson: [Monitoring and Recovering Pipelines](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/monitoring-and-recovering-pipelines?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-data-quality-monitoring-system), in Grokking the Data Engineering Interview
- [Orchestration](../concepts/orchestration.md) explains where a blocking check sits in a pipeline.
