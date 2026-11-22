# Data Engineering Interview

> The free, open companion to **Grokking the Data Engineering Interview** by [DesignGurus.io](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme), created by Arslan Ahmad.

[![GitHub stars](https://img.shields.io/github/stars/design-gurus/data-engineering-interview?style=social)](https://github.com/design-gurus/data-engineering-interview/stargazers)
[![Last commit](https://img.shields.io/github/last-commit/design-gurus/data-engineering-interview)](https://github.com/design-gurus/data-engineering-interview/commits/main)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](LICENSE)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

A data engineering interview tests whether a candidate can move data from where it is produced to where it is used, and keep it correct on the way. The loop has five recurring rounds: SQL, coding, data modeling, pipeline design, and behavioral. Most public question lists give 2,000 questions with no order. This repository takes the opposite approach: each round explained, each concept with its reasoning, six design walkthroughs, and one curated question set.

Every page is short and ends with a link to the lesson-level material in the full course.

## Start here

| Your situation | Read this first |
|---|---|
| You have an interview booked and do not know the format | [The interview loop](rounds/interview-loop.md) |
| You have one week | [Question bank](cheat-sheets/question-bank.md), then [SQL patterns](cheat-sheets/sql-patterns.md) |
| You have a month | [Four-week plan](cheat-sheets/four-week-plan.md) |
| You are a software engineer moving into data | [Dimensional modeling](concepts/dimensional-modeling.md), then [Batch vs streaming](concepts/batch-vs-streaming.md) |
| You keep failing the design round | [Pipeline design](rounds/pipeline-design.md), then [Design a clickstream pipeline](questions/design-a-clickstream-pipeline.md) |

## The loop at a glance

| Round | Length | What it measures |
|---|---|---|
| [SQL](rounds/sql.md) | 45 to 60 minutes | Window functions, deduplication, and reasoning about the grain of a result |
| [Coding](rounds/coding.md) | 45 to 60 minutes | Python for parsing, grouping, and files larger than memory |
| [Data modeling](rounds/data-modeling.md) | 45 to 60 minutes | Facts, dimensions, grain, and history |
| [Pipeline design](rounds/pipeline-design.md) | 45 to 60 minutes | An end-to-end design with freshness, reruns, late data, and cost |
| [Behavioral](rounds/behavioral.md) | 30 to 45 minutes | Ownership of data incidents and work with data consumers |

Round names and lengths are commonly reported by candidates. They are not official for any company.

## Every page

### Rounds

| Page | What it covers |
|---|---|
| [The interview loop](rounds/interview-loop.md) | Stages from the recruiter call to the final rounds |
| [SQL](rounds/sql.md) | What the round tests, with two worked queries |
| [Data modeling](rounds/data-modeling.md) | A five-step method for a modeling question |
| [Pipeline design](rounds/pipeline-design.md) | A method and a time budget for the design round |
| [Coding](rounds/coding.md) | The five recurring Python problem shapes |
| [Behavioral](rounds/behavioral.md) | The stories a data engineer needs |

### Concepts

| Page | The decision it explains |
|---|---|
| [Batch vs streaming](concepts/batch-vs-streaming.md) | Choosing from a freshness requirement |
| [ETL vs ELT](concepts/etl-vs-elt.md) | Where the transformation runs |
| [Dimensional modeling](concepts/dimensional-modeling.md) | Facts, dimensions, star and snowflake schemas |
| [Slowly changing dimensions](concepts/slowly-changing-dimensions.md) | Keeping or discarding history |
| [Warehouse vs lake vs lakehouse](concepts/data-warehouse-vs-data-lake-vs-lakehouse.md) | Where the data is stored |
| [Partitioning and file formats](concepts/partitioning-and-file-formats.md) | Parquet, Avro, ORC, and partition columns |
| [Idempotency and backfills](concepts/idempotency-and-backfills.md) | Jobs that can run twice safely |
| [Delivery guarantees](concepts/delivery-guarantees.md) | What exactly-once requires |
| [Late data and watermarks](concepts/late-data-and-watermarks.md) | Events that arrive after their window |
| [Data quality and testing](concepts/data-quality-and-testing.md) | Checks that return failing rows |
| [Orchestration](concepts/orchestration.md) | Scheduling, dependencies, and retries |
| [Spark essentials](concepts/spark-essentials.md) | Shuffles, partitions, and skew |

### Design questions

| Page | The main difficulty |
|---|---|
| [Design a clickstream pipeline](questions/design-a-clickstream-pipeline.md) | Volume, duplicates, and sessions |
| [Design a daily sales warehouse](questions/design-a-daily-sales-warehouse.md) | Grain and history |
| [Design a change data capture pipeline](questions/design-a-change-data-capture-pipeline.md) | Ordering and deletes |
| [Design a real-time fraud feature pipeline](questions/design-a-real-time-fraud-feature-pipeline.md) | Latency and consistency between training and serving |
| [Design a data quality monitoring system](questions/design-a-data-quality-monitoring-system.md) | Useful alerts without noise |
| [Design a log analytics platform](questions/design-a-log-analytics-platform.md) | Cost of storage and of queries |

### Cheat sheets

| Page | Use it for |
|---|---|
| [Question bank](cheat-sheets/question-bank.md) | 60 questions grouped by round |
| [SQL patterns](cheat-sheets/sql-patterns.md) | Ten query shapes with a minimal snippet each |
| [Common mistakes](cheat-sheets/common-mistakes.md) | Twelve errors and their corrections |
| [Four-week plan](cheat-sheets/four-week-plan.md) | A study schedule of about 8 hours a week |

## FAQ

### What is asked in a data engineering interview?

Most loops ask SQL with window functions, a Python data manipulation problem, a data modeling question, and a pipeline design question. A behavioral round closes the loop. Some companies add a product sense round, in which the candidate defines a metric and then designs the tables that support it.

### What are the most common data engineer interview questions?

Five questions recur across companies. Three are SQL: keep the latest row for each key, find the top three items in each group, and compute a running total. Two are design: model a star schema for a business process, and design a daily pipeline that can be rerun safely. The [question bank](cheat-sheets/question-bank.md) lists 60.

### How is a data pipeline design interview different from a system design interview?

A system design interview centers on serving requests: latency, availability, and scale. A pipeline design interview centers on data correctness over time: freshness, reruns, late data, schema changes, and cost. The [pipeline design](rounds/pipeline-design.md) page gives a method for it.

### How hard is the SQL in a data engineer interview?

It is harder than the SQL in a software engineer interview. Joins and GROUP BY are assumed. The questions that decide the round use window functions, deduplication, gaps and islands, and sessionization. See [SQL](rounds/sql.md).

### Do data engineers get coding questions on algorithms?

Some companies ask one. The more common coding round uses Python on data: parsing files, grouping records, merging intervals, and processing input that does not fit in memory. See [Coding](rounds/coding.md).

### How long does it take to prepare?

An engineer who writes SQL at work usually needs four weeks at about 8 hours a week. The [four-week plan](cheat-sheets/four-week-plan.md) divides that time by round.

## Course lessons to start with

- [Window Functions](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/window-functions?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): the tool behind most SQL round questions, in Grokking the Data Engineering Interview
- [Gaps and Islands](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/gaps-and-islands?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): how to find streaks of consecutive rows, in Grokking the Data Engineering Interview
- [Facts Dimensions and Grain](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/facts-dimensions-and-grain?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): the parts of a dimensional model, in Grokking the Data Engineering Interview
- [Slowly Changing Dimensions](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/slowly-changing-dimensions?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): how a dimension table keeps history, in Grokking the Data Engineering Interview
- [Batch or Streaming](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/batch-or-streaming?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): how the freshness requirement decides the design, in Grokking the Data Engineering Interview
- [Idempotent Pipelines](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/idempotent-pipelines?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): how to make a rerun produce the same result, in Grokking the Data Engineering Interview
- [Event Time Windows and Watermarks](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/event-time-windows-and-watermarks?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): how a stream job handles late events, in Grokking the Data Engineering Interview
- [Partitioning and File Sizes](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/partitioning-and-file-sizes?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): how table layout affects query cost, in Grokking the Data Engineering Interview
- [Clickstream Pipeline: Requirements and Data Model](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/clickstream-pipeline-requirements-and-data-model?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): a full pipeline design answer from the first questions, in Grokking the Data Engineering Interview
- [Exactly Once in Kafka](https://www.designgurus.io/course-play/grokking-kafka-and-event-driven-architecture/doc/exactly-once-in-kafka?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): what the guarantee covers and where it stops, in Grokking Kafka and Event-Driven Architecture

## Free reading on DesignGurus.io

- Free article: [Data engineering system design](https://www.designgurus.io/blog/data-engineering-system-design?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): how to design a pipeline from source to serving layer.
- Free article: [Data engineering interviews](https://www.designgurus.io/blog/data-engineering-interviews-how-to-stand-out-in-a-competitive-field?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): how to prepare and what separates strong candidates.
- Free article: [Data warehouse design](https://www.designgurus.io/blog/data-warehouse-design?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): schemas, design steps, and best practices.
- Free article: [Change data capture](https://www.designgurus.io/blog/change-data-capture?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): how systems stay in sync in real time.
- Free article: [Exactly-once delivery](https://www.designgurus.io/blog/exactly-once-delivery?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): why the guarantee is hard and how Kafka provides it.
- Free article: [Kafka Streams vs Apache Flink vs Apache Storm](https://www.designgurus.io/blog/kafka-streams-apache-flink-apache-storm?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): a comparison of three stream processing engines.
- Free answer: [Batch processing vs stream processing](https://www.designgurus.io/answers/detail/what-is-batch-processing-vs-stream-processing-and-when-should-you-use-each?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme): when to use each approach.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme) covers every round on this page with worked lessons, diagrams, and assessments.
- [Grokking SQL for Tech Interviews](https://www.designgurus.io/course/grokking-sql-for-tech-interviews?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme) covers the SQL basics that the data engineering round assumes.
- [Grokking the System Design Interview](https://www.designgurus.io/course/grokking-the-system-design-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=readme) covers the general design round that some data engineering loops also include.

Related free guides: [Grokking System Design](https://github.com/design-gurus/grokking-system-design) and [System Design Numbers](https://github.com/design-gurus/system-design-numbers).

For the system design round, see [Grokking System Design](https://github.com/design-gurus/grokking-system-design), the free companion guide with 30 patterns, 59 question walkthroughs, and 35 cheat sheets.

## More free guides

Every guide below is free, follows the same short-page format, and comes from DesignGurus.io.

- **Core guides:** [Grokking System Design](https://github.com/design-gurus/grokking-system-design), [Grokking Coding Interview](https://github.com/design-gurus/grokking-coding-interview), [Grokking AI System Design](https://github.com/design-gurus/grokking-ai-system-design), [Design Patterns](https://github.com/design-gurus/design-patterns), [Best System Design Resources](https://github.com/design-gurus/best-system-design-resources), [Best System Design Courses](https://github.com/design-gurus/best-system-design-courses)
- **By interview round:** [System Design Numbers](https://github.com/design-gurus/system-design-numbers), [API Design Interview](https://github.com/design-gurus/api-design-interview), [Frontend System Design Interview](https://github.com/design-gurus/frontend-system-design-interview), [Mobile System Design Interview](https://github.com/design-gurus/mobile-system-design-interview), [Microservices Interview Questions](https://github.com/design-gurus/microservices-interview-questions), [Concurrency Interview](https://github.com/design-gurus/concurrency-interview)
- **By role:** [DevOps and SRE Interview](https://github.com/design-gurus/devops-sre-interview), [Forward Deployed Engineer Interview](https://github.com/design-gurus/forward-deployed-engineer-interview), [Solutions Architect Interview](https://github.com/design-gurus/solutions-architect-interview)
- **Seniority and behavioral:** [Staff Engineer Interview](https://github.com/design-gurus/staff-engineer-interview), [Engineering Manager Interview](https://github.com/design-gurus/engineering-manager-interview), [Amazon Leadership Principles](https://github.com/design-gurus/amazon-leadership-principles), [Software Engineer Levels](https://github.com/design-gurus/software-engineer-levels)
- **By company:** [Google Interview Guide](https://github.com/design-gurus/google-interview-guide), [Meta Interview Guide](https://github.com/design-gurus/meta-interview-guide), [Amazon Interview Guide](https://github.com/design-gurus/amazon-interview-guide), [Microsoft Interview Guide](https://github.com/design-gurus/microsoft-interview-guide), [Apple Interview Guide](https://github.com/design-gurus/apple-interview-guide), [Netflix Interview Guide](https://github.com/design-gurus/netflix-interview-guide), [OpenAI Interview Guide](https://github.com/design-gurus/openai-interview-guide)

## Contributing

Corrections and additions are welcome. Read [CONTRIBUTING.md](CONTRIBUTING.md) first.

## License

The content is licensed under [CC BY 4.0](LICENSE). See [NOTICE.md](NOTICE.md) for what the license covers and how to attribute.
