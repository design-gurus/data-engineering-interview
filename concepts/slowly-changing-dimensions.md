# Slowly Changing Dimensions

A slowly changing dimension is a dimension table whose attributes change occasionally, such as a store's region or a customer's plan. The design question is what to do with the old value. The numbered types below are the standard answers, and type 2 is the one interviewers ask about most.

## The types

| Type | What happens on a change | History kept | Use when |
|---|---|---|---|
| 0 | Nothing. The original value stays. | The original only | The attribute must never change, such as a signup date |
| 1 | The old value is overwritten | None | The old value was an error, or history has no use |
| 2 | A new row is added, and the old row is closed | Full | Past facts must be reported with the value valid at the time |
| 3 | A "previous value" column is added | One prior value | Only the last change matters |

## One change under each type

Store 17 moves from the North region to the West region on the first day of March.

**Type 1** overwrites the row:

| store_key | store_id | region |
|---|---|---|
| 501 | 17 | West |

Every past sale of store 17 is now reported under West, including sales made while it was in North.

**Type 2** closes the old row and adds a new one:

| store_key | store_id | region | valid_from | valid_to | is_current |
|---|---|---|---|---|---|
| 501 | 17 | North | Jan 1 | Mar 1 | false |
| 734 | 17 | West | Mar 1 | (open) | true |

Sales before March carry `store_key` 501, and sales after carry 734. Each sale is reported under the region that was true when it happened.

**Type 3** adds a column:

| store_key | store_id | region | previous_region |
|---|---|---|---|
| 501 | 17 | West | North |

Only one earlier value survives. A second move would discard North.

## What type 2 requires

- **A surrogate key.** `store_id` 17 now has two rows, so the fact table must join on `store_key`.
- **Validity columns.** `valid_from` and `valid_to` define when each row applied. Use a half-open interval, where `valid_from` is included and `valid_to` is excluded, so that no instant belongs to two rows.
- **A lookup at load time.** When a sale is loaded, the pipeline finds the dimension row whose interval contains the sale date.

```sql
SELECT s.sale_id, d.store_key
FROM sales_staging AS s
JOIN dim_store AS d
  ON d.store_id = s.store_id
 AND s.sale_date >= d.valid_from
 AND (s.sale_date < d.valid_to OR d.valid_to IS NULL);
```

This is called a point-in-time join: each fact is matched to the dimension row that was valid at that time.

## Choosing a type per attribute

The type is chosen for each attribute, not for the whole table. In one store dimension, a corrected spelling of the city is type 1, while the region is type 2.

| Question | If yes |
|---|---|
| Would a past report change if the value changed? Should it? | Type 1 if it should, type 2 if it should not |
| Does the attribute change many times a day? | It is not slowly changing. Move it to a fact table. |

## Follow-up questions to expect

- "A sale arrives for a store that is not yet in the dimension." Assign it to a reserved "unknown" row, then update the fact when the store arrives.
- "How large does type 2 make the table?" 2,000 stores that each change twice a year add 4,000 rows a year, which is negligible.
- "How do you query the current view?" Filter on `is_current = true`.

## Go deeper

- [Grokking the Data Engineering Interview](https://www.designgurus.io/course/grokking-the-data-engineering-interview?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-slowly-changing-dimensions) covers slowly changing dimensions and point-in-time joins with a worked example.
- Lesson: [Slowly Changing Dimensions](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/slowly-changing-dimensions?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-slowly-changing-dimensions), in Grokking the Data Engineering Interview
- Lesson: [Point in Time Joins](https://www.designgurus.io/course-play/grokking-the-data-engineering-interview/doc/point-in-time-joins?utm_source=github&utm_medium=repo&utm_campaign=data-engineering-interview&utm_content=concepts-slowly-changing-dimensions), in Grokking the Data Engineering Interview
- [Dimensional modeling](dimensional-modeling.md) explains facts, dimensions, and surrogate keys.
