# Spark Essentials

Apache Spark is a distributed processing engine that splits a dataset into partitions and runs the same code on each partition in parallel. Most Spark interview questions reduce to one subject: the shuffle, which is the movement of rows between machines so that related rows meet. A candidate who can find and reduce shuffles can answer nearly every performance question.

## Five terms

| Term | Meaning |
|---|---|
| Partition | A chunk of the data processed by one task |
| Transformation | An operation that defines a new dataset, such as `filter` or `join` |
| Action | An operation that returns a result or writes output, such as `count` or `write` |
| Lazy evaluation | Transformations are only recorded, and work starts when an action runs |
| Stage | A group of steps that run without a shuffle |

## Narrow and wide transformations

| | Narrow | Wide |
|---|---|---|
| Input needed | Each output partition reads one input partition | Each output partition reads many input partitions |
| Shuffle | No | Yes |
| Examples | `select`, `filter`, `withColumn`, `map` | `groupBy`, `join`, `distinct`, `orderBy`, `repartition` |
| Cost | Low | High: rows are written to disk and sent over the network |

Each wide transformation ends one stage and starts another. Counting the stages in a query plan is a quick estimate of its cost.

## The two joins

| Join | How it works | Use when |
|---|---|---|
| Broadcast hash join | The small table is copied to every executor, and no shuffle of the large table is needed | One side is small, by default under about 10 megabytes and often raised to hundreds |
| Sort-merge join | Both sides are shuffled by the join key, sorted, and merged | Both sides are large |

Joining a 500 gigabyte fact table to a 5 megabyte dimension should be a broadcast join. If the plan shows a sort-merge join, the engine did not know the small table's size, and a broadcast hint corrects it.

## Skew

Skew means that a few keys hold a large share of the rows. After a shuffle, all rows with one key go to one partition, so one task receives far more data than the others.

**The symptom.** 199 of 200 tasks finish in 1 minute, and one task runs for 40 minutes.

| Remedy | How it works |
|---|---|
| Adaptive query execution | The engine splits oversized partitions at run time. It is on by default in current versions. |
| Broadcast the other side | Removes the shuffle, when that side is small enough |
| Salting | Add a random number from 0 to N-1 to the hot key, so its rows spread across N partitions |
| Handle the hot key separately | Process the largest key in its own query and union the results |
| Filter nulls | A null join key is a frequent hidden hot key |

## How many partitions

A common target is 100 to 200 megabytes of data per partition. A shuffle of 100 gigabytes therefore needs about 500 to 1,000 partitions. The default of 200 shuffle partitions is too few for that job and too many for a 1 gigabyte job. Adaptive query execution merges small shuffle partitions automatically.

Too few partitions cause memory errors and disk spills. Too many cause scheduling overhead and small output files.

## Reducing the work before the shuffle

1. Filter rows and select only needed columns as early as possible.
2. Read partitioned columnar files, so the engine skips data. See [partitioning and file formats](partitioning-and-file-formats.md).
3. Aggregate before joining when the join does not need row-level detail.
4. Cache a dataset only when it is reused several times, and release it afterward.

## Questions an interviewer may ask

- "A job became slow as data grew." Check the plan for a join that changed from broadcast to sort-merge, and check for skew.
- "What is the difference between `repartition` and `coalesce`?" `repartition` shuffles and can increase the count. `coalesce` merges partitions without a full shuffle and can only decrease it.
- "Why did the job write 200 tiny files?" One file is written per partition. Reduce partitions before writing.

## The sentence to say

"I would read the plan first, count the shuffles, and check task durations for skew before changing any configuration."

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-spark-essentials) reads a real Spark plan and resolves a skewed join.
- Lesson: [How Spark Runs a Job](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/how-spark-runs-a-job?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-spark-essentials), in Grokking the Data Engineering Interview
- Lesson: [Spark Joins and Skew](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/spark-joins-and-skew?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-spark-essentials), in Grokking the Data Engineering Interview
- [Grokking the System Design Interview](https://www.designgurus.io/course/grokking-the-system-design-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-spark-essentials) covers the MapReduce idea that the shuffle comes from.
