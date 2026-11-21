# Dimensional Modeling

Dimensional modeling is a way of shaping tables for analysis. It separates measurements, called facts, from the descriptive context around them, called dimensions. The approach was popularized by Ralph Kimball, and it remains the default vocabulary of the data modeling round.

## Facts and dimensions

| | Fact table | Dimension table |
|---|---|---|
| Holds | Measurements of a business event | Descriptions of who, what, where, and when |
| Columns | Numeric measures and keys to dimensions | Text and category attributes |
| Shape | Narrow and very long | Wide and short |
| Example | One row per product line on a receipt | One row per product |
| Typical size | Billions of rows | Thousands to millions of rows |

## Grain comes first

The grain is the exact meaning of one row in a fact table. "One row per product line on each sales receipt" is a grain. "Sales data" is not. Every measure and every dimension must agree with the grain. If a column would repeat or be empty for some rows, it belongs to a different fact table.

Choose the finest grain the source provides. Daily totals can be computed from receipt lines, and receipt lines cannot be recovered from daily totals.

## Three kinds of fact tables

| Kind | Grain | Example |
|---|---|---|
| Transaction | One row per event | A sale, a click |
| Periodic snapshot | One row per entity per period | Inventory per product per day |
| Accumulating snapshot | One row per process, updated as it advances | An order with its placed, shipped, and delivered dates |

## Star schema and snowflake schema

In a star schema, each dimension is one table joined directly to the fact table. In a snowflake schema, dimensions are normalized into several tables, so a product table points to a category table, which points to a department table.

| Aspect | Star | Snowflake |
|---|---|---|
| Joins per query | One per dimension | Several per dimension |
| Query simplicity | Higher | Lower |
| Repeated text in dimensions | Yes | No |
| Storage saved by normalizing | Not applicable | Small, because dimensions are small |
| Common default | Yes | Used when a hierarchy is shared or changes independently |

Dimensions are a tiny share of total storage, so the space a snowflake saves rarely justifies its extra joins. State the star schema as the default and give that reason.

## A small example

```
fact_sales (date_key, store_key, product_key, customer_key,
            quantity, net_amount)
dim_date     (date_key, day, week, month, year, is_holiday)
dim_store    (store_key, store_id, city, region)
dim_product  (product_key, product_id, name, category)
dim_customer (customer_key, customer_id, plan_tier)
```

Revenue by region and month is one query: sum `net_amount`, join two dimensions, and group by two attributes.

## Surrogate keys

A surrogate key is an integer that the warehouse generates for each dimension row. It replaces the source system's identifier as the join key. It is required when a dimension keeps history, because one store then has several rows and the source identifier no longer identifies one row. See [slowly changing dimensions](slowly-changing-dimensions.md).

## Why not one wide table?

A single wide table with every attribute copied onto every event is simple to query and fast on columnar engines. Its weakness is change: when a product is renamed, either billions of rows are rewritten or old rows keep the old name without anyone having chosen that. A dimensional model makes the choice explicit.

## The sentence to say

"The grain is one row per receipt line. I will use a star schema, because the dimensions are small and analysts will write fewer joins."

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-dimensional-modeling) has a Data Modeling module on grain, fact table kinds, and event data.
- [Grokking Relational Database Design and Modeling for Software Engineers](https://www.designgurus.io/course/grokking-relational-database-design-and-modeling-for-software-engineers?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-dimensional-modeling) covers the normalized models that application databases use.
- [The data modeling round](../rounds/data-modeling.md) gives a method for the interview.
- Free article: [Data warehouse design](https://www.designgurus.io/blog/data-warehouse-design?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-dimensional-modeling), which covers schemas, design steps, and best practices.
- Free article: [Normalization vs denormalization](https://www.designgurus.io/blog/normalization-vs-denormalization?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-dimensional-modeling), the trade-off behind star and snowflake schemas.
