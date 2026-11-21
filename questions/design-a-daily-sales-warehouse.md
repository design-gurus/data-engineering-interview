# Design a Daily Sales Warehouse

This question asks for a warehouse that reports a retailer's sales each morning. It combines the data modeling round and the pipeline design round in one answer. The flow is simple, so the interviewer's attention goes to the grain of the fact table, the history kept in dimensions, and corrections such as refunds.

## Clarify first

| Question | Why it matters |
|---|---|
| Which reports are needed? | They define the dimensions |
| At what time must the data be ready? | Sets the deadline for the nightly run |
| Do stores or products change attributes? | Decides which dimensions keep history |
| How are refunds recorded in the source? | Decides how corrections enter the fact table |
| How many stores and receipt lines per day? | Sizes the fact table |

## Numbers for this walkthrough

A retailer has 1,200 stores. Each store records about 8,000 receipt lines a day, which gives 9.6 million fact rows a day and about 3.5 billion a year. Finance reads the reports at 7:00.

## Data model

**Grain:** one row for each product line on each receipt.

| Table | Type | Notes |
|---|---|---|
| `fact_sales` | Transaction fact | Quantity, net amount, discount amount, and four dimension keys |
| `dim_date` | Dimension | Built once for many years ahead |
| `dim_store` | Dimension, type 2 | Stores move between regions |
| `dim_product` | Dimension, mixed | Category is type 2, a corrected name is type 1 |
| `dim_customer` | Dimension, type 2 | Loyalty tier changes |
| `fact_inventory_daily` | Periodic snapshot | One row per product per store per day |

Type 2 means that a change adds a new row and closes the old one. See [slowly changing dimensions](../concepts/slowly-changing-dimensions.md).

## Flow

1. Extract the day's receipts and the changed dimension records from the store systems after closing time.
2. Land both in the raw layer, partitioned by business date.
3. Load dimensions first, applying type 1 and type 2 rules.
4. Load `fact_sales`, looking up the dimension row valid on the sale date.
5. Reconcile against the source, then publish.

Dimensions load before facts, because each fact row needs the surrogate key of its dimension rows.

## The decisions that matter

**Refunds.** A refund is a new row with a negative quantity and amount, dated on the refund day and linked to the original receipt. The original row is never edited. Sums stay correct, and the history of what was reported remains explainable.

**Late stores.** One store's file is missing at 3:00. Either hold the whole publication or publish with that store marked incomplete. State the choice and the reason. For finance figures, an explicit "incomplete" flag is usually better than a silent gap.

**Aggregates.** Most dashboards read daily totals by store. A summary table of 1,200 rows a day answers them without scanning 9.6 million.

## Reconciliation

Compare two numbers per store per day with the source system: the count of receipt lines and the summed net amount. Block publication if any store differs by more than 0.1 percent. See [data quality and testing](../concepts/data-quality-and-testing.md).

## What the interviewer probes

- "A product moved category. Which category do last year's sales show?" The category at the time of sale, because the category attribute is type 2.
- "The nightly job ran twice." It overwrites the partition for the business date, so totals do not change.
- "An analyst sums inventory across a week." Inventory is semi-additive. Use the last day's value or an average.
- "How would you add an online channel?" Add a channel dimension, or a second fact table if the grain differs.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-daily-sales-warehouse) covers grain, snapshot facts, and point-in-time joins in its Data Modeling module.
- [Grokking Relational Database Design and Modeling for Software Engineers](https://www.designgurus.io/course/grokking-relational-database-design-and-modeling-for-software-engineers?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-daily-sales-warehouse) covers the source-side models these tables are extracted from.
- [The data modeling round](../rounds/data-modeling.md) gives the method used here.
- Free article: [Data warehouse design](https://www.designgurus.io/blog/data-warehouse-design?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=questions-design-a-daily-sales-warehouse), which covers schemas, design steps, and best practices.
