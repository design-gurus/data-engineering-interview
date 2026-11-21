# Common Mistakes

Twelve errors that candidates make repeatedly in data engineering interviews, grouped by round. Each has the correction beside it. Most of them come from describing only the case in which nothing fails.

## SQL

| Mistake | Why it costs points | Correction |
|---|---|---|
| Writing the query before stating the output grain | A join silently multiplies rows and the totals are wrong | Say "one row per customer per day" first, then write |
| Ignoring ties in a top N question | The result has more or fewer rows than expected | Choose `ROW_NUMBER`, `RANK`, or `DENSE_RANK` aloud and give the reason |
| Using `NOT IN` with a column that can be null | One null makes the result empty | Use `NOT EXISTS` |
| Not testing the query | An error stays hidden until the interviewer finds it | Trace a three-row example that includes a tie and a null |

## Coding

| Mistake | Why it costs points | Correction |
|---|---|---|
| Reading the whole file into memory | The first follow-up question is a larger file | Iterate line by line, and state the memory cost |
| Splitting CSV lines on commas | A quoted field with a comma breaks the parse | Use the `csv` module |
| Raising an error on the first bad row | Real files always contain bad rows | Quarantine the row, count it, and continue |

## Data modeling

| Mistake | Why it costs points | Correction |
|---|---|---|
| Drawing tables before declaring the grain | Measures and dimensions do not agree with each other | State the grain in one sentence first |
| Overwriting every dimension change | Past reports change without anyone deciding that | Choose type 1 or type 2 for each attribute |
| Storing a ratio as a measure | Averages of ratios are wrong | Store the numerator and the denominator |

## Pipeline design

| Mistake | Why it costs points | Correction |
|---|---|---|
| Proposing streaming without a freshness requirement | Cost and complexity rise with no stated benefit | Ask how old the data may be, then choose |
| Describing the flow and stopping | Reruns, late data, and schema changes are where the round is decided | Cover all three before being asked |

## A worked example of the last mistake

**The question:** "Design a daily revenue pipeline."

**An incomplete answer:** "A nightly job reads orders from the database, aggregates them in the warehouse, and a dashboard reads the result."

**The same answer, completed with four sentences:**

1. "Each run overwrites the partition for its order date, so a retry never doubles revenue."
2. "Each run also reprocesses the previous 3 days, because refunds and late orders arrive after the day closes."
3. "The raw extract is kept, so I can rebuild any day after a bug."
4. "Before publication, a check compares the row count and the total with the source, within 0.1 percent."

The four sentences take under a minute and answer the four most common follow-up questions.

## Mistakes in how the answer is delivered

- **Silence while thinking.** The interviewer scores reasoning and cannot score what was not said. State the approach before writing.
- **No numbers.** "A lot of data" gives no basis for a design. "200 million events a day, about 200 gigabytes" does.
- **Naming tools instead of properties.** "I would use a message broker" says little. "I need a durable buffer that preserves order per key" explains the choice, and the tool follows from it.
- **Defending a first answer.** When the interviewer adds a constraint, change the design and say what changed.

## A short self-check for any answer

| Round | Ask yourself |
|---|---|
| SQL | Did I state the grain, the tie rule, and the null behavior? |
| Coding | Did I handle a bad row and a large input? |
| Data modeling | Did I state the grain, and which attributes keep history? |
| Pipeline design | Did I cover reruns, late data, schema change, and cost? |
| Behavioral | Did I give a number and a lasting change? |

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=cheat-sheets-common-mistakes) opens many lessons with a broken output and repairs it, which is practice for exactly these mistakes.
- [The pipeline design round](../rounds/pipeline-design.md) gives the method that prevents the last two.
