# The Data Modeling Round

The data modeling round asks the candidate to design tables for analysis, not for an application. The interviewer names a business, such as a pharmacy chain, and asks for a model that answers its reporting questions. The strongest single habit is to declare the grain, meaning exactly what one row of the main table represents, before drawing any table.

## A five-step method

| Step | Minutes | What to do |
|---|---|---|
| 1. Questions | 5 | Ask which three or four business questions the model must answer |
| 2. Process and grain | 5 | Pick one business process and state the grain in one sentence |
| 3. Facts | 8 | List the numeric measures and classify each one |
| 4. Dimensions | 12 | List the descriptive tables and decide which keep history |
| 5. Test | 10 | Write one query for each business question against the model |

The remaining 5 minutes go to follow-up questions.

## Worked example: pharmacy sales

**Questions.** Revenue by store and week, the best-selling products in each region, and revenue by customer plan.

**Grain.** One row for each product line on each sales receipt. This is the finest grain available, so every coarser question can be answered by adding up.

**Fact table.** `fact_sales` with `quantity`, `unit_price`, `discount_amount`, and `net_amount`, plus one key for each dimension.

**Dimensions.**

| Dimension | Example attributes | History kept? |
|---|---|---|
| `dim_date` | Day, week, month, holiday flag | Not needed |
| `dim_product` | Name, category, manufacturer | Overwrite corrections |
| `dim_store` | City, region, format | Yes, because stores move between regions |
| `dim_customer` | Plan tier, signup date | Yes, because the plan changes |

**Test.** Revenue by region and week is a sum of `net_amount` joined to `dim_store` and `dim_date`. If a store moved regions last month, last year's sales must still be reported under the old region. That requirement is what makes `dim_store` a type 2 [slowly changing dimension](../concepts/slowly-changing-dimensions.md).

## Three kinds of measures

| Kind | Meaning | Example |
|---|---|---|
| Additive | Can be summed across every dimension | Net amount |
| Semi-additive | Can be summed across some dimensions only | Inventory on hand, which must not be summed across days |
| Non-additive | Cannot be summed at all | A ratio such as discount percent |

Store the parts of a ratio, not the ratio. A discount percent averaged across rows gives a wrong answer, while summed discount divided by summed gross gives the right one.

## Follow-up questions to expect

- How would the model change for a daily inventory level? (A periodic snapshot fact, one row per product per store per day.)
- What happens when a sale arrives before its customer record? (Point the fact at an "unknown" dimension row and correct it later.)
- Why not one wide table? (It is simpler to query, and harder to keep consistent when an attribute changes.)
- How many rows will the fact table hold in a year? (2,000 stores at 5,000 lines a day gives 10 million rows a day, about 3.65 billion a year.)

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-data-modeling) has a Data Modeling module covering grain, history, and point-in-time joins.
- [Grokking Relational Database Design and Modeling for Software Engineers](https://www.designgurus.io/course/grokking-relational-database-design-and-modeling-for-software-engineers?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=rounds-data-modeling) covers normalization, the starting point that analytic models depart from.
- [Dimensional modeling](../concepts/dimensional-modeling.md) explains facts, dimensions, and schema shapes.
