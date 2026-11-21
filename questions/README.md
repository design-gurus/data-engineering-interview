# Design Questions

Six pipeline design questions, each at the level of approach and trade-offs. Every page follows the same order: what to clarify, the numbers, the data model, the flow, the two or three decisions that matter, and what the interviewer probes. None is a complete solution. Use the [pipeline design method](../rounds/pipeline-design.md) with each one.

| Question | Type | The main difficulty |
|---|---|---|
| [Design a clickstream pipeline](design-a-clickstream-pipeline.md) | Batch with event data | Volume, duplicates, and sessions |
| [Design a daily sales warehouse](design-a-daily-sales-warehouse.md) | Batch with dimensional modeling | Grain and history |
| [Design a change data capture pipeline](design-a-change-data-capture-pipeline.md) | Database replication | Ordering and deletes |
| [Design a real-time fraud feature pipeline](design-a-real-time-fraud-feature-pipeline.md) | Streaming | Latency, and consistency between training and serving |
| [Design a data quality monitoring system](design-a-data-quality-monitoring-system.md) | Platform | Useful alerts without noise |
| [Design a log analytics platform](design-a-log-analytics-platform.md) | High-volume ingestion | Cost of storage and of queries |

Start with the clickstream pipeline. It is the most frequently reported question, and its parts reappear in the others.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-readme) works through complete designs across several lessons each, with diagrams and cost arithmetic.
