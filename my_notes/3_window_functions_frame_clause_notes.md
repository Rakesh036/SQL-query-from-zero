# Window Functions - Frame Clause & Advanced Concepts

## The Problem We Face

Let's say we want to calculate a **3-day rolling average** of sales for each salesperson. 

**Traditional approach using JOIN/subquery:**
```sql
SELECT s1.sales_person_id, s1.date, s1.order_amt,
       (SELECT AVG(s2.order_amt) 
        FROM sales_person s2 
        WHERE s2.sales_person_id = s1.sales_person_id 
          AND s2.date BETWEEN s1.date - INTERVAL 2 DAY AND s1.date
       ) as rolling_3day_avg
FROM sales_person s1
ORDER BY s1.sales_person_id, s1.date;
```

**Problems with this approach:**
- **Complex to read** - nested subquery is hard to understand
- **Performance issues** - correlated subquery runs for every row
- **Hard to maintain** - what if we want 5-day rolling average? Need to change logic
- **Not flexible** - difficult to add multiple rolling calculations

**Solution:** Frame Clause in Window Functions! 

It gives us **precise control over which rows to include** in our calculation window, making rolling averages, sliding windows, and cumulative calculations much easier.

---

## Sample Data

Let's use this `sales_person` table:

| sales_person_id | date       | order_amt |
|----------------|------------|-----------|
| 101            | 2023-01-01 | 1000      |
| 101            | 2023-01-02 | 1500      |
| 101            | 2023-01-03 | 2000      |
| 101            | 2023-01-04 | 1200      |
| 101            | 2023-01-05 | 1800      |
| 102            | 2023-01-01 | 800       |
| 102            | 2023-01-02 | 1100      |
| 102            | 2023-01-03 | 900       |

---

## What is Frame Clause?

**Frame clause** allows us to define exactly **which rows to include** in our window function calculation relative to the current row.

**Q: Why do we need frame clause when we already have PARTITION BY and ORDER BY?**
**A:** PARTITION BY and ORDER BY define the "big picture" - which group and in what order. But frame clause defines the "small picture" - exactly which rows around the current row should be included in calculation.

Think of it like this:
- **PARTITION BY** → Which rows belong to my group? 
- **ORDER BY** → In what sequence should I process them?
- **Frame Clause** → For each row, which nearby rows should I include in my calculation?

---

## Frame Clause Syntax

```sql
aggregate_function() OVER(
    [PARTITION BY column] 
    [ORDER BY column] 
    [ROWS|RANGE|GROUPS BETWEEN start AND end]
)
```

### Frame Types:
1. **ROWS** → Physical number of rows
2. **RANGE** → Logical range based on values  
3. **GROUPS** → Groups of rows with same ORDER BY value

### Frame Boundaries:
- **UNBOUNDED PRECEDING** → From the very first row of partition
- **n PRECEDING** → n rows before current row
- **CURRENT ROW** → The current row
- **n FOLLOWING** → n rows after current row  
- **UNBOUNDED FOLLOWING** → Till the very last row of partition

---

## Example 1: Rolling 3-Day Average

**Problem:** Calculate 3-day rolling average (current day + 2 previous days)

```sql
SELECT sales_person_id, date, order_amt,
       AVG(order_amt) OVER(
           PARTITION BY sales_person_id 
           ORDER BY date 
           ROWS 2 PRECEDING
       ) as rolling_3day_avg
FROM sales_person
ORDER BY sales_person_id, date;
```

**Q: What does "ROWS 2 PRECEDING" mean?**
**A:** Include current row + 2 rows before it. Total window size = 3 rows maximum.

**Dry Run for sales_person_id = 101:**

| date       | order_amt | Window Rows Used | Calculation | rolling_3day_avg |
|------------|-----------|------------------|-------------|------------------|
| 2023-01-01 | 1000      | [1000]           | 1000/1      | 1000             |
| 2023-01-02 | 1500      | [1000, 1500]     | (1000+1500)/2 | 1250           |
| 2023-01-03 | 2000      | [1000, 1500, 2000] | (1000+1500+2000)/3 | 1500    |
| 2023-01-04 | 1200      | [1500, 2000, 1200] | (1500+2000+1200)/3 | 1567    |
| 2023-01-05 | 1800      | [2000, 1200, 1800] | (2000+1200+1800)/3 | 1667    |

**Q: Why does the first row only use 1 value instead of 3?**
**A:** Because there are no preceding rows available. Frame adjusts automatically to available data.

---

## Example 2: Centered 3-Day Moving Average

**Problem:** Calculate centered moving average (previous day + current + next day)

```sql
SELECT sales_person_id, date, order_amt,
       AVG(order_amt) OVER(
           PARTITION BY sales_person_id 
           ORDER BY date 
           ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
       ) as centered_3day_avg
FROM sales_person
ORDER BY sales_person_id, date;
```

**Dry Run for sales_person_id = 101:**

| date       | order_amt | Window Rows Used | Calculation | centered_3day_avg |
|------------|-----------|------------------|-------------|-------------------|
| 2023-01-01 | 1000      | [1000, 1500]     | (1000+1500)/2 | 1250            |
| 2023-01-02 | 1500      | [1000, 1500, 2000] | (1000+1500+2000)/3 | 1500      |
| 2023-01-03 | 2000      | [1500, 2000, 1200] | (1500+2000+1200)/3 | 1567      |
| 2023-01-04 | 1200      | [2000, 1200, 1800] | (2000+1200+1800)/3 | 1667      |
| 2023-01-05 | 1800      | [1200, 1800]     | (1200+1800)/2 | 1500            |

**Q: Why first and last rows have only 2 values?**
**A:** First row has no preceding row, last row has no following row. Frame adjusts to available data.

---

## Example 3: Running Total vs Current + Next 2 Rows

**Problem:** Compare running total with sum of current + next 2 rows

```sql
SELECT sales_person_id, date, order_amt,
       -- Running total (default frame)
       SUM(order_amt) OVER(
           PARTITION BY sales_person_id 
           ORDER BY date
       ) as running_total,
       -- Current + next 2 rows
       SUM(order_amt) OVER(
           PARTITION BY sales_person_id 
           ORDER BY date 
           ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING
       ) as next_3_sum
FROM sales_person
ORDER BY sales_person_id, date;
```

**Dry Run for sales_person_id = 101:**

| date       | order_amt | Running Total | Next 3 Sum | Explanation |
|------------|-----------|---------------|-------------|-------------|
| 2023-01-01 | 1000      | 1000          | 4500        | Running: [1000], Next3: [1000,1500,2000] |
| 2023-01-02 | 1500      | 2500          | 4700        | Running: [1000,1500], Next3: [1500,2000,1200] |
| 2023-01-03 | 2000      | 4500          | 5000        | Running: [1000,1500,2000], Next3: [2000,1200,1800] |
| 2023-01-04 | 1200      | 5700          | 3000        | Running: [1000,1500,2000,1200], Next3: [1200,1800] |
| 2023-01-05 | 1800      | 7500          | 1800        | Running: [1000,1500,2000,1200,1800], Next3: [1800] |

---

## Default Frame Rules (Important for Interviews!)

**Q: What happens if I don't specify frame clause?**
**A:** SQL uses default frame rules:

1. **With ORDER BY but no frame** → Default is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`
2. **No ORDER BY** → Default is `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` (entire partition)

```sql
-- These two are IDENTICAL:
SUM(order_amt) OVER(PARTITION BY sales_person_id ORDER BY date)
SUM(order_amt) OVER(PARTITION BY sales_person_id ORDER BY date 
                   RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
```

**Q: What's the difference between ROWS and RANGE?**
**A:** 
- **ROWS** → Counts physical rows (1, 2, 3...)
- **RANGE** → Based on ORDER BY column values (dates, numbers...)

---

## ROWS vs RANGE Example

Let's say we have duplicate dates:

| sales_person_id | date       | order_amt |
|----------------|------------|-----------|
| 101            | 2023-01-01 | 1000      |
| 101            | 2023-01-01 | 1500      | ← Same date!
| 101            | 2023-01-02 | 2000      |

```sql
-- ROWS: Counts physical rows
SUM(order_amt) OVER(ORDER BY date ROWS 1 PRECEDING)

-- RANGE: Includes all rows with same ORDER BY value  
SUM(order_amt) OVER(ORDER BY date RANGE 1 PRECEDING)
```

**ROWS Result:**
| date       | order_amt | Window Used | Sum |
|------------|-----------|-------------|-----|
| 2023-01-01 | 1000      | [1000]      | 1000|
| 2023-01-01 | 1500      | [1000,1500] | 2500|
| 2023-01-02 | 2000      | [1500,2000] | 3500|

**RANGE Result:**
| date       | order_amt | Window Used | Sum |
|------------|-----------|-------------|-----|
| 2023-01-01 | 1000      | [1000,1500] | 2500| ← Includes both rows with same date
| 2023-01-01 | 1500      | [1000,1500] | 2500| ← Same result
| 2023-01-02 | 2000      | [1000,1500,2000] | 4500| ← Includes all previous date rows

**Q: When to use ROWS vs RANGE?**
**A:** 
- **ROWS** → When you want exact number of physical rows (most common)
- **RANGE** → When you want to include all rows with same ORDER BY value

---

## Advanced Frame Examples

### 1. First and Last Value in Window

```sql
SELECT sales_person_id, date, order_amt,
       FIRST_VALUE(order_amt) OVER(
           PARTITION BY sales_person_id ORDER BY date 
           ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING
       ) as first_in_window,
       LAST_VALUE(order_amt) OVER(
           PARTITION BY sales_person_id ORDER BY date 
           ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING
       ) as last_in_window
FROM sales_person;
```

### 2. Percentage of Rolling Total

```sql
SELECT sales_person_id, date, order_amt,
       order_amt * 100.0 / SUM(order_amt) OVER(
           PARTITION BY sales_person_id ORDER BY date 
           ROWS 2 PRECEDING
       ) as pct_of_rolling_total
FROM sales_person;
```

### 3. Difference from Rolling Average

```sql
SELECT sales_person_id, date, order_amt,
       AVG(order_amt) OVER(
           PARTITION BY sales_person_id ORDER BY date 
           ROWS 2 PRECEDING
       ) as rolling_avg,
       order_amt - AVG(order_amt) OVER(
           PARTITION BY sales_person_id ORDER BY date 
           ROWS 2 PRECEDING
       ) as diff_from_avg
FROM sales_person;
```

---

## Common Frame Patterns (Interview Cheat Sheet)

| Pattern | Frame Clause | Use Case |
|---------|--------------|----------|
| Running total | `ROWS UNBOUNDED PRECEDING` | Cumulative sum |
| Rolling N-period | `ROWS (N-1) PRECEDING` | Moving average |
| Centered window | `ROWS BETWEEN N PRECEDING AND N FOLLOWING` | Smoothing |
| Forward looking | `ROWS BETWEEN CURRENT ROW AND N FOLLOWING` | Future projections |
| Full partition | `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` | Grand total |
| Current row only | `ROWS CURRENT ROW` | No aggregation across rows |

---

## Function-Specific Parameters

Some window functions have their **own parameters** separate from the OVER() clause:

### LAG and LEAD
```sql
LAG(column, offset, default_value) OVER(...)
LEAD(column, offset, default_value) OVER(...)
```

**Example:**
```sql
SELECT sales_person_id, date, order_amt,
       LAG(order_amt, 1, 0) OVER(PARTITION BY sales_person_id ORDER BY date) as prev_amt,
       LAG(order_amt, 2, -1) OVER(PARTITION BY sales_person_id ORDER BY date) as two_days_ago,
       LEAD(order_amt, 1, 9999) OVER(PARTITION BY sales_person_id ORDER BY date) as next_amt
FROM sales_person;
```

**Q: What does LAG(order_amt, 2, -1) mean?**
**A:** Get order_amt from 2 rows back. If not available, return -1 instead of NULL.

### NTILE - Split into Buckets
```sql
NTILE(number_of_buckets) OVER(...)
```

**Example:**
```sql
SELECT sales_person_id, order_amt,
       NTILE(3) OVER(ORDER BY order_amt DESC) as sales_quartile
FROM sales_person;
```

This splits all salespeople into 3 buckets based on order_amt (top performers, middle, bottom).

### NTH_VALUE - Get Nth Value in Window
```sql
NTH_VALUE(column, N) OVER(...)
```

**Example:**
```sql
SELECT sales_person_id, date, order_amt,
       NTH_VALUE(order_amt, 2) OVER(
           PARTITION BY sales_person_id ORDER BY date 
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) as second_order_amt
FROM sales_person;
```

Gets the 2nd order amount for each salesperson.

---

## Performance Tips

**Q: Do frame clauses affect performance?**
**A:** Yes! Here are the performance implications:

1. **ROWS is faster than RANGE** - simpler calculation
2. **Smaller frames are faster** - less data to process
3. **Unbounded frames can be expensive** - especially UNBOUNDED FOLLOWING
4. **Multiple window functions** - try to use same OVER() clause to reuse calculations

**Good:**
```sql
SELECT *,
  SUM(order_amt) OVER(PARTITION BY sales_person_id ORDER BY date) as running_sum,
  AVG(order_amt) OVER(PARTITION BY sales_person_id ORDER BY date) as running_avg
FROM sales_person;
```

**Better (reuses same window):**
```sql
SELECT *,
  SUM(order_amt) OVER w as running_sum,
  AVG(order_amt) OVER w as running_avg
FROM sales_person
WINDOW w AS (PARTITION BY sales_person_id ORDER BY date);
```

---

## Summary: Frame Clause Benefits

1. **Precise control** over which rows to include
2. **Easy rolling calculations** - no complex self-joins
3. **Better performance** - optimized by database
4. **More readable** - intent is clear from syntax
5. **Flexible** - easy to change window size
6. **Handles edge cases** - automatically adjusts for missing data

**Interview Key Points:**
- Frame clause defines **which rows around current row** to include
- **ROWS** counts physical rows, **RANGE** uses ORDER BY values
- Default frame with ORDER BY is **UNBOUNDED PRECEDING to CURRENT ROW**
- Frame boundaries: UNBOUNDED PRECEDING/FOLLOWING, N PRECEDING/FOLLOWING, CURRENT ROW
- Use **ROWS** for most cases, **RANGE** only when you need value-based windows