# Notes on Window Functions in SQL

**Start:** Take a table of `sales_person` which contains:
* `sales_person_id`
* `order_no`
* `date`
* `order_amt`

Now apply several queries on it.

---

## 1. Total company sales

```sql
SELECT SUM(order_amt) 
FROM sales_person;
```

This gives the **total sales of the company**.

---

## 2. Total sales per salesman

```sql
SELECT sales_person_id, SUM(order_amt) 
FROM sales_person 
GROUP BY sales_person_id;
```

This gives the **sales sum of each salesman**.

---

👉 **Problem:** The above query only returns `sales_person_id` and total amount. If we want the **whole table with one extra column** showing the sales total per salesman, we **can't just write it in SELECT** — because non-aggregated columns must appear in `GROUP BY`.

So we need a different approach → **Window Functions**.

---

## 3. Window function with `OVER()`

```sql
SELECT *,
       SUM(order_amt) OVER() AS total_company_sales
FROM sales_person;
```

Here, by writing `OVER()`, a **window size equal to the entire table** is created.

* This window size is **fixed for each row** - hence the output is same for each row.
* It repeats the **total company sales** in each row.

**Q: Do we need to give an alias to the new column created by window function?**
**A:** Not mandatory, but highly recommended for readability. Without alias, you'll get a generic column name like `sum` which is confusing.

**Q: Why does each row have the same value?**
**A:** Because the window size is fixed (entire table) for every row. SQL calculates SUM across all rows and shows that same total in each row.

---

## 4. Partitioning by salesman

If we want sales by each salesman in every row:

```sql
SELECT *,
       SUM(order_amt) OVER(PARTITION BY sales_person_id) AS sales_by_person
FROM sales_person;
```

Now each partition is one salesman, and the window size = number of rows for that salesman.

---

## 5. Running totals with `ORDER BY`

```sql
SELECT *,
       SUM(order_amt) OVER(ORDER BY date) AS running_total
FROM sales_person;
```

* Adding `ORDER BY date` inside `OVER()` makes the window grow row by row.
* This gives a **cumulative sum** (running total).
* The last row = total company sales.

If we combine **PARTITION BY + ORDER BY**, then within each salesman, the cumulative sum resets.

```sql
SELECT *,
       SUM(order_amt) OVER(PARTITION BY sales_person_id ORDER BY date) AS running_total_per_person
FROM sales_person;
```

---

## 6. Key idea of window functions

* `OVER()` → whole table (fixed window).
* `PARTITION BY` → creates smaller windows (one per group).
* `ORDER BY` → makes the window grow step by step (running total).
* `PARTITION BY + ORDER BY` → running totals **per group**.

⚠️ **Important:** Window function output is row-based and only valid in the `SELECT` (or subquery SELECT).

* We **cannot use it directly in WHERE** because WHERE is evaluated **before** window functions are calculated.
* If you need to filter on window function results, use a subquery or CTE.

```sql
-- This WON'T work
SELECT * FROM sales_person 
WHERE SUM(order_amt) OVER() > 1000;

-- This WILL work
SELECT * FROM (
    SELECT *, SUM(order_amt) OVER() AS total_sales
    FROM sales_person
) t 
WHERE total_sales > 1000;
```

---

## Common Window Functions

### Aggregate Functions
* `SUM`, `AVG`, `MIN`, `MAX` → aggregate window functions
* `COUNT` → counting within window

### Ranking Functions
* `ROW_NUMBER()` → unique sequential numbers (1, 2, 3, 4, 5...)
* `RANK()` → ranking with gaps (1, 2, 2, 4, 5...)
* `DENSE_RANK()` → ranking without gaps (1, 2, 2, 3, 4...)
* `PERCENT_RANK()` → rank expressed as a percentage of distribution

👉 **Difference:**
* `RANK` leaves gaps if there are ties.
* `DENSE_RANK` doesn't leave gaps.
* `ROW_NUMBER` always gives unique numbers even for ties.

### Example with ranking:

```sql
SELECT sales_person_id, order_amt,
       ROW_NUMBER() OVER(ORDER BY order_amt DESC) as row_num,
       RANK() OVER(ORDER BY order_amt DESC) as rank_val,
       DENSE_RANK() OVER(ORDER BY order_amt DESC) as dense_rank_val
FROM sales_person;
```

**Sample Output:**
| sales_person_id | order_amt | row_num | rank_val | dense_rank_val |
|----------------|-----------|---------|----------|----------------|
| 101            | 5000      | 1       | 1        | 1              |
| 102            | 4000      | 2       | 2        | 2              |
| 103            | 4000      | 3       | 2        | 2              |
| 104            | 3000      | 4       | 4        | 3              |

---

## Lead & Lag Functions

* `LAG(column, n, default)` → looks back **n rows**.
* `LEAD(column, n, default)` → looks ahead **n rows**.

**Example:**

```sql
SELECT sales_person_id, date, order_amt,
       LAG(order_amt, 1, 0) OVER(ORDER BY date) AS prev_order,
       LEAD(order_amt, 1, 0) OVER(ORDER BY date) AS next_order,
       LAG(order_amt, 2, 0) OVER(ORDER BY date) AS two_orders_back
FROM sales_person;
```

* `LAG(order_amt, 1, 0)` → takes order_amt from **1 row back**.
* If not available, returns **0** instead of NULL.
* `LEAD(order_amt, 1, 0)` → takes order_amt from **1 row ahead**.

These are very useful for **comparing current row with previous/next row**, calculating differences, growth rates, etc.

```sql
-- Calculate day-over-day growth
SELECT date, order_amt,
       order_amt - LAG(order_amt, 1, 0) OVER(ORDER BY date) AS daily_growth
FROM sales_person;
```

---

## Frame Specification (Advanced)

You can also control the exact window frame using `ROWS` or `RANGE`:

```sql
-- Sum of current row + 2 preceding rows
SELECT *, 
       SUM(order_amt) OVER(ORDER BY date ROWS 2 PRECEDING) AS rolling_3day_sum
FROM sales_person;

-- Sum from start of partition to current row (default behavior)
SELECT *, 
       SUM(order_amt) OVER(ORDER BY date ROWS UNBOUNDED PRECEDING) AS running_total
FROM sales_person;

-- Sum of current row + 1 preceding + 1 following
SELECT *, 
       SUM(order_amt) OVER(ORDER BY date ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS centered_sum
FROM sales_person;
```

---

## Real-world Example → Gaps and Islands

**Problem:** Collapse consecutive rows with the same status.

**Input (orders table):**

| order_id | status            | status_date |
|----------|-------------------|-------------|
| 11700    | New               | 2016-01-03  |
| 11700    | Inventory Check   | 2016-01-04  |
| 11700    | Inventory Check   | 2016-01-05  |
| 11700    | Inventory Check   | 2016-01-06  |
| 11700    | Awaiting Signoff  | 2016-01-09  |
| 11700    | Awaiting Signoff  | 2016-01-10  |
| 11700    | In Warehouse      | 2016-01-12  |
| 11700    | Delivery          | 2016-01-21  |
| 11700    | Delivery          | 2016-01-22  |

**Desired Output (collapsed ranges):**

| order_id | status           | from_date  | to_date    |
|----------|------------------|------------|------------|
| 11700    | New              | 2016-01-03 | 2016-01-03 |
| 11700    | Inventory Check  | 2016-01-04 | 2016-01-06 |
| 11700    | Awaiting Signoff | 2016-01-09 | 2016-01-10 |
| 11700    | In Warehouse     | 2016-01-12 | 2016-01-12 |
| 11700    | Delivery         | 2016-01-21 | 2016-01-22 |

---

### SQL Solution (Row Number Trick)

```sql
SELECT 
    order_id,
    status,
    MIN(status_date) AS from_date,
    MAX(status_date) AS to_date
FROM (
    SELECT 
        order_id,
        status,
        status_date,
        ROW_NUMBER() OVER (ORDER BY status_date) 
          - ROW_NUMBER() OVER (PARTITION BY status ORDER BY status_date) AS grp
    FROM orders
    WHERE order_id = 11700
) t
GROUP BY order_id, status, grp
ORDER BY from_date;
```

---

### Dry Run Explanation

**Step 1:** Add two row numbers

* `RN1` = sequential row numbers across all rows ordered by status_date
* `RN2` = sequential row numbers **within each status group**

| status           | status_date | RN1 | RN2 |
|------------------|-------------|-----|-----|
| New              | 2016-01-03  | 1   | 1   |
| Inventory Check  | 2016-01-04  | 2   | 1   |
| Inventory Check  | 2016-01-05  | 3   | 2   |
| Inventory Check  | 2016-01-06  | 4   | 3   |
| Awaiting Signoff | 2016-01-09  | 5   | 1   |
| Awaiting Signoff | 2016-01-10  | 6   | 2   |
| In Warehouse     | 2016-01-12  | 7   | 1   |
| Delivery         | 2016-01-21  | 8   | 1   |
| Delivery         | 2016-01-22  | 9   | 2   |

**Step 2:** Calculate `grp = RN1 - RN2`

| status           | status_date | RN1 | RN2 | grp (RN1-RN2) |
|------------------|-------------|-----|-----|---------------|
| New              | 2016-01-03  | 1   | 1   | 0             |
| Inventory Check  | 2016-01-04  | 2   | 1   | 1             |
| Inventory Check  | 2016-01-05  | 3   | 2   | 1             |
| Inventory Check  | 2016-01-06  | 4   | 3   | 1             |
| Awaiting Signoff | 2016-01-09  | 5   | 1   | 4             |
| Awaiting Signoff | 2016-01-10  | 6   | 2   | 4             |
| In Warehouse     | 2016-01-12  | 7   | 1   | 6             |
| Delivery         | 2016-01-21  | 8   | 1   | 7             |
| Delivery         | 2016-01-22  | 9   | 2   | 7             |

👉 **Key insight:** For consecutive rows with the same status, `grp` stays constant!
👉 When status changes, `grp` jumps to a new value.

**Step 3:** Group by `(order_id, status, grp)` and aggregate

This groups consecutive rows with the same status together, then we take MIN/MAX dates.

✅ This is a classic **"gaps and islands" problem** solved using window functions.

---

## Quick Reference Table

| Scenario | Syntax | Result |
|----------|--------|---------|
| Whole table aggregate | `SUM(col) OVER()` | Same value in all rows |
| Per-group aggregate | `SUM(col) OVER(PARTITION BY grp)` | Same value within each group |
| Running total | `SUM(col) OVER(ORDER BY col)` | Cumulative sum |
| Per-group running total | `SUM(col) OVER(PARTITION BY grp ORDER BY col)` | Cumulative sum per group |
| Row numbering | `ROW_NUMBER() OVER(ORDER BY col)` | 1, 2, 3, 4... |
| Ranking with gaps | `RANK() OVER(ORDER BY col)` | 1, 2, 2, 4... |
| Ranking without gaps | `DENSE_RANK() OVER(ORDER BY col)` | 1, 2, 2, 3... |
| Previous row value | `LAG(col, 1) OVER(ORDER BY col)` | Value from 1 row back |
| Next row value | `LEAD(col, 1) OVER(ORDER BY col)` | Value from 1 row ahead |

---

## Common Use Cases

1. **Running totals and moving averages** - financial reports, sales tracking
2. **Ranking and top-N queries** - leaderboards, performance analysis  
3. **Comparing with previous/next values** - growth calculations, trend analysis
4. **Gaps and islands problems** - grouping consecutive records
5. **Percentiles and statistical analysis** - data distribution analysis
6. **Deduplication based on ranking** - keeping only the latest record per group

Remember: Window functions are powerful for analytical queries where you need both detail rows AND aggregate information together!