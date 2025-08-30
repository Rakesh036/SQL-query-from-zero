# MySQL Window Functions - Complete Study Notes

## 🎯 What Are Window Functions?

Window functions perform calculations across rows while keeping all original rows (unlike GROUP BY which collapses rows).

**Key Concept**: Window functions let you add analytical information to each row without losing the individual row data.

### Basic Syntax Pattern
```sql
SELECT 
    column1,
    column2,
    WINDOW_FUNCTION() OVER (
        [PARTITION BY column] 
        [ORDER BY column]
        [frame_specification]
    ) as result_column
FROM table_name;
```

### Key Components
- **WINDOW_FUNCTION()** - The function you want to apply
- **OVER()** - Defines the "window" of rows to consider
- **PARTITION BY** - Divides data into groups (like GROUP BY, but keeps all rows)
- **ORDER BY** - Sorts data within each partition
- **Frame specification** - Defines which rows within the partition to include

### Built-in vs User-Defined
- Window functions are **built-in** functions provided by MySQL
- Available since **MySQL 8.0** (major limitation in older versions)
- You cannot create custom window functions

---

## 📊 Phase 2: Core Ranking Functions

These are the most commonly used window functions for ranking and numbering rows.

### ROW_NUMBER()
Assigns a unique sequential number to each row within a partition.

```sql
SELECT 
    name,
    department,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as overall_rank,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank
FROM employees;

-- Result:
-- name | department | salary | overall_rank | dept_rank
-- Bob  | IT         | 70000  | 1            | 1
-- Jane | Sales      | 60000  | 2            | 1  
-- John | Sales      | 50000  | 3            | 2
```

### RANK()
Similar to ROW_NUMBER, but gives the same rank to tied values, then skips numbers.

```sql
-- Let's say we have tied salaries
SELECT 
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) as salary_rank,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as row_num
FROM employees;

-- If two people have 60000 salary:
-- name | salary | salary_rank | row_num
-- Bob  | 70000  | 1           | 1
-- Jane | 60000  | 2           | 2
-- Mike | 60000  | 2           | 3  <- Same rank as Jane
-- John | 50000  | 4           | 4  <- Skipped rank 3
```

### DENSE_RANK()
Like RANK, but doesn't skip numbers after ties.

```sql
SELECT 
    name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank,
    RANK() OVER (ORDER BY salary DESC) as regular_rank
FROM employees;

-- Result with tied salaries:
-- name | salary | dense_rank | regular_rank
-- Bob  | 70000  | 1          | 1
-- Jane | 60000  | 2          | 2
-- Mike | 60000  | 2          | 2
-- John | 50000  | 3          | 4  <- No gap in dense_rank
```

### Key Differences Summary
- **ROW_NUMBER()**: Always unique (1,2,3,4...)
- **RANK()**: Ties get same rank, skips numbers (1,2,2,4...)
- **DENSE_RANK()**: Ties get same rank, no gaps (1,2,2,3...)

### Practical Examples

**Find top 3 earners in each department:**
```sql
SELECT *
FROM (
    SELECT 
        name,
        department,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank
    FROM employees
) ranked
WHERE dept_rank <= 3;
```

**Find employees with duplicate salaries:**
```sql
SELECT *
FROM (
    SELECT 
        name,
        salary,
        COUNT(*) OVER (PARTITION BY salary) as salary_count
    FROM employees
) t
WHERE salary_count > 1;
```

---

## 📈 Phase 3: Aggregate Window Functions

Using familiar aggregate functions (SUM, COUNT, AVG) with the OVER clause to create running totals, moving averages, and cumulative statistics.

### Running Totals with SUM()

```sql
SELECT 
    name,
    department,
    salary,
    SUM(salary) OVER (ORDER BY salary) as running_total,
    SUM(salary) OVER (PARTITION BY department ORDER BY salary) as dept_running_total
FROM employees
ORDER BY salary;

-- Result:
-- name | department | salary | running_total | dept_running_total
-- John | Sales      | 50000  | 50000        | 50000
-- Jane | Sales      | 60000  | 110000       | 110000
-- Bob  | IT         | 70000  | 180000       | 70000
```

### Percentage of Total

```sql
SELECT 
    name,
    department,
    salary,
    SUM(salary) OVER () as total_payroll,  -- No PARTITION = entire table
    ROUND(salary * 100.0 / SUM(salary) OVER (), 2) as pct_of_total,
    ROUND(salary * 100.0 / SUM(salary) OVER (PARTITION BY department), 2) as pct_of_dept
FROM employees;

-- Result:
-- name | dept  | salary | total_payroll | pct_of_total | pct_of_dept
-- John | Sales | 50000  | 180000        | 27.78        | 45.45
-- Jane | Sales | 60000  | 180000        | 33.33        | 54.55
-- Bob  | IT    | 70000  | 180000        | 38.89        | 100.00
```

### COUNT() for Relative Position

```sql
SELECT 
    name,
    salary,
    COUNT(*) OVER () as total_employees,
    COUNT(*) OVER (ORDER BY salary) as employees_at_or_below,
    COUNT(*) OVER (ORDER BY salary DESC) as employees_at_or_above
FROM employees
ORDER BY salary;

-- Result:
-- name | salary | total_employees | at_or_below | at_or_above
-- John | 50000  | 3              | 1           | 3
-- Jane | 60000  | 3              | 2           | 2  
-- Bob  | 70000  | 3              | 3           | 1
```

### Moving Averages with AVG()

```sql
-- Let's use a sales table with dates
SELECT 
    sale_date,
    daily_sales,
    AVG(daily_sales) OVER () as overall_avg,
    AVG(daily_sales) OVER (ORDER BY sale_date 
                          ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg_3day
FROM daily_sales
ORDER BY sale_date;

-- This calculates a 3-day moving average
```

### Practical Business Examples

**1. Find employees earning above department average:**
```sql
SELECT *
FROM (
    SELECT 
        name,
        department,
        salary,
        AVG(salary) OVER (PARTITION BY department) as dept_avg
    FROM employees
) t
WHERE salary > dept_avg;
```

**2. Running total with reset by group:**
```sql
SELECT 
    department,
    name,
    salary,
    SUM(salary) OVER (PARTITION BY department ORDER BY salary 
                      ROWS UNBOUNDED PRECEDING) as dept_running_total
FROM employees
ORDER BY department, salary;
```

### Key Points
- Empty OVER() clause = entire result set
- PARTITION BY = creates separate calculations for each group
- ORDER BY in OVER = affects the "window frame" (which rows are included)
- You often need subqueries to filter on window function results

---

## 🎯 Phase 4: Frames & Boundaries

This is where window functions get really powerful! We'll learn to precisely control which rows are included in our calculations.

### Understanding Window Frames

A window frame defines exactly which rows around the current row to include in the calculation. Think of it as a "sliding window" that moves with each row.

### Basic Frame Syntax
```sql
OVER (
    [PARTITION BY column]
    ORDER BY column
    {ROWS | RANGE} BETWEEN frame_start AND frame_end
)
```

### ROWS vs RANGE
- **ROWS**: Counts physical row positions
- **RANGE**: Based on value differences (useful for dates, numbers)

### Frame Boundary Options
```sql
-- Frame start/end can be:
UNBOUNDED PRECEDING    -- From the very first row
n PRECEDING           -- n rows before current
CURRENT ROW          -- The current row  
n FOLLOWING          -- n rows after current
UNBOUNDED FOLLOWING  -- To the very last row
```

### ROWS Examples

**3-row moving average:**
```sql
SELECT 
    sale_date,
    daily_sales,
    AVG(daily_sales) OVER (
        ORDER BY sale_date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as moving_avg_3day
FROM daily_sales;

-- For each row, averages current row + 2 rows before it
```

**Running total (explicit frame):**
```sql
SELECT 
    name,
    salary,
    SUM(salary) OVER (
        ORDER BY salary 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_total
FROM employees;

-- Same as: SUM(salary) OVER (ORDER BY salary)
-- The frame is implied when you use ORDER BY
```

**Centered moving average:**
```sql
SELECT 
    sale_date,
    daily_sales,
    AVG(daily_sales) OVER (
        ORDER BY sale_date 
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) as centered_avg_3day
FROM daily_sales;

-- Averages: previous row + current row + next row
```

### RANGE Examples

**Sales within date range:**
```sql
SELECT 
    sale_date,
    daily_sales,
    SUM(daily_sales) OVER (
        ORDER BY sale_date 
        RANGE BETWEEN INTERVAL 7 DAY PRECEDING AND CURRENT ROW
    ) as sales_last_7_days
FROM daily_sales;

-- Sums all sales within the last 7 days (including current day)
```

**Values within numeric range:**
```sql
SELECT 
    name,
    salary,
    AVG(salary) OVER (
        ORDER BY salary 
        RANGE BETWEEN 5000 PRECEDING AND 5000 FOLLOWING
    ) as avg_similar_salaries
FROM employees;

-- Averages salaries within ±5000 of current salary
```

### Default Frame Behavior

**Important**: When you use ORDER BY without specifying a frame:
- Default is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`
- This can cause unexpected results with duplicate values!

```sql
-- These might give different results if there are duplicate salaries:
SELECT 
    name, salary,
    SUM(salary) OVER (ORDER BY salary) as default_frame,
    SUM(salary) OVER (ORDER BY salary ROWS UNBOUNDED PRECEDING) as explicit_rows
FROM employees;
```

### Key Points
- Always be explicit about frames when precision matters
- ROWS = physical position, RANGE = logical value
- Default frame behavior can be surprising with duplicates
- Frames let you create sophisticated sliding window calculations

---

## 🚀 Phase 5: Advanced Techniques

This final phase covers the most sophisticated window functions and real-world complex scenarios.

### LAG() and LEAD() - Accessing Other Rows

These functions let you peek at values from previous or future rows without self-joins.

**Basic LAG/LEAD:**
```sql
SELECT 
    sale_date,
    daily_sales,
    LAG(daily_sales) OVER (ORDER BY sale_date) as yesterday_sales,
    LEAD(daily_sales) OVER (ORDER BY sale_date) as tomorrow_sales,
    daily_sales - LAG(daily_sales) OVER (ORDER BY sale_date) as day_over_day_change
FROM daily_sales
ORDER BY sale_date;

-- Result shows current, previous, and next day's sales
```

**LAG/LEAD with offset and default:**
```sql
SELECT 
    employee_id,
    review_date,
    performance_score,
    LAG(performance_score, 1, 0) OVER (
        PARTITION BY employee_id 
        ORDER BY review_date
    ) as previous_score,
    LAG(performance_score, 2, 0) OVER (
        PARTITION BY employee_id 
        ORDER BY review_date  
    ) as two_reviews_ago
FROM performance_reviews;

-- LAG(column, offset, default_value)
-- offset = how many rows back (default 1)
-- default_value = what to return if no previous row exists
```

### NTILE() - Creating Buckets

Divides rows into a specified number of roughly equal groups.

```sql
SELECT 
    name,
    department,
    salary,
    NTILE(4) OVER (ORDER BY salary) as salary_quartile,
    NTILE(3) OVER (PARTITION BY department ORDER BY salary) as dept_tercile
FROM employees;

-- Creates 4 salary quartiles across all employees
-- Creates 3 groups within each department
```

### Advanced Pattern: Period-over-Period Analysis

**Month-over-month growth:**
```sql
SELECT 
    year_month,
    monthly_revenue,
    LAG(monthly_revenue) OVER (ORDER BY year_month) as prev_month,
    ROUND(
        (monthly_revenue - LAG(monthly_revenue) OVER (ORDER BY year_month)) * 100.0 
        / LAG(monthly_revenue) OVER (ORDER BY year_month), 2
    ) as mom_growth_pct,
    LAG(monthly_revenue, 12) OVER (ORDER BY year_month) as same_month_last_year,
    ROUND(
        (monthly_revenue - LAG(monthly_revenue, 12) OVER (ORDER BY year_month)) * 100.0 
        / LAG(monthly_revenue, 12) OVER (ORDER BY year_month), 2
    ) as yoy_growth_pct
FROM monthly_sales
ORDER BY year_month;
```

### Complex Real-World Scenarios

**1. Customer Lifetime Value Analysis:**
```sql
SELECT 
    customer_id,
    order_date,
    order_amount,
    SUM(order_amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) as lifetime_value,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) as order_number,
    DATEDIFF(
        order_date, 
        FIRST_VALUE(order_date) OVER (
            PARTITION BY customer_id 
            ORDER BY order_date
        )
    ) as days_since_first_order
FROM orders;
```

**2. Top N Within Groups with Ties:**
```sql
-- Get top 2 products by sales in each category, handling ties properly
SELECT *
FROM (
    SELECT 
        category,
        product_name,
        sales_amount,
        DENSE_RANK() OVER (
            PARTITION BY category 
            ORDER BY sales_amount DESC
        ) as sales_rank
    FROM product_sales
) ranked
WHERE sales_rank <= 2;
```

---

## 📋 Where Can You Use Window Functions?

### Usage Rules

**✅ CAN use in:**
- SELECT clause ✓
- ORDER BY clause ✓

**❌ CANNOT use in:**
- WHERE clause ❌
- GROUP BY clause ❌
- HAVING clause ❌

### Examples

```sql
-- ✅ WORKS
SELECT name, ROW_NUMBER() OVER (ORDER BY salary) as rn FROM employees;
SELECT name FROM employees ORDER BY ROW_NUMBER() OVER (ORDER BY salary);

-- ❌ DOESN'T WORK  
SELECT name FROM employees WHERE ROW_NUMBER() OVER (ORDER BY salary) = 1;

-- ✅ WORKAROUND - Use subquery
SELECT * FROM (
    SELECT name, ROW_NUMBER() OVER (ORDER BY salary) as rn 
    FROM employees
) WHERE rn = 1;
```

### SQL Execution Order

1. FROM / JOIN          -- Get tables
2. WHERE               -- Filter rows  
3. GROUP BY            -- Group rows
4. HAVING              -- Filter groups
5. **WINDOW FUNCTIONS**    -- Calculate window functions ⭐
6. SELECT              -- Choose columns
7. DISTINCT            -- Remove duplicates  
8. ORDER BY            -- Sort results
9. LIMIT               -- Limit results

**Key Point**: Window functions execute AFTER WHERE but BEFORE final SELECT, which is why you can't use them in WHERE clause!

---

## 🔧 Performance Tips

1. **Use indexes on PARTITION BY and ORDER BY columns:**
```sql
-- If you frequently use:
-- PARTITION BY department ORDER BY salary
-- Create index: CREATE INDEX idx_dept_salary ON employees(department, salary);
```

2. **Limit window function usage in WHERE clauses:**
```sql
-- Instead of this (which may not work):
SELECT * FROM employees 
WHERE ROW_NUMBER() OVER (ORDER BY salary DESC) <= 10;

-- Use subquery:
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (ORDER BY salary DESC) as rn
    FROM employees
) t WHERE rn <= 10;
```

3. **Consider Common Table Expressions (CTEs) for readability:**
```sql
WITH employee_rankings AS (
    SELECT 
        name, department, salary,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank,
        PERCENT_RANK() OVER (ORDER BY salary) as salary_percentile
    FROM employees
)
SELECT * 
FROM employee_rankings 
WHERE dept_rank <= 3 
   AND salary_percentile >= 0.75;
```

---

## 📊 Complete Function Reference

### Ranking Functions
```sql
ROW_NUMBER()    -- Always unique: 1,2,3,4...
RANK()         -- Ties same rank, skips: 1,2,2,4...  
DENSE_RANK()   -- Ties same rank, no gaps: 1,2,2,3...
NTILE(n)       -- Divide into n buckets
```

### Offset Functions
```sql
LAG(col, offset, default)    -- Previous row value
LEAD(col, offset, default)   -- Next row value
FIRST_VALUE(col)             -- First in window
LAST_VALUE(col)              -- Last in window
```

### Aggregate Functions
```sql
SUM() OVER()     -- Running totals, moving sums
AVG() OVER()     -- Moving averages  
COUNT() OVER()   -- Running counts
MIN()/MAX() OVER() -- Moving min/max
```

### Statistical Functions
```sql
PERCENT_RANK()   -- Percentile rank (0 to 1)
CUME_DIST()      -- Cumulative distribution
```

---

## 💡 Common Patterns

### Top N per Group
```sql
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) as rn
    FROM employees
) WHERE rn <= 3;
```

### Running Totals
```sql
SUM(amount) OVER (ORDER BY date ROWS UNBOUNDED PRECEDING)
```

### Moving Averages
```sql
AVG(sales) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)
```

### Period Comparisons
```sql
LAG(revenue) OVER (ORDER BY month) as prev_month,
LAG(revenue, 12) OVER (ORDER BY month) as same_month_last_year
```

### Percentage of Total
```sql
salary * 100.0 / SUM(salary) OVER () as pct_of_total,
salary * 100.0 / SUM(salary) OVER (PARTITION BY dept) as pct_of_dept
```

---

## 🚨 Important Reminders

1. **Can't use window functions in WHERE** - Always use subqueries for filtering
2. **Default frame with ORDER BY**: `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`
3. **ROWS vs RANGE**: ROWS = physical positions, RANGE = logical values
4. **Performance**: Index PARTITION BY and ORDER BY columns
5. **Subqueries are the universal workaround** - When in doubt, wrap in subquery

---

## 📈 Business Use Cases

- **Rankings**: Top performers, product rankings
- **Time Series**: Running totals, moving averages, growth rates
- **Cohort Analysis**: Customer lifetime value, retention
- **Financial**: Period-over-period comparisons, percentiles
- **Data Quality**: Finding duplicates, gaps in sequences

---

## 🔍 Complete Dry Run Example - Step by Step Mental Picture

### Sample Data: employees table
```
| id | name  | department | salary | hire_date  |
|----|-------|------------|--------|------------|
| 1  | Alice | Sales      | 50000  | 2022-01-15 |
| 2  | Bob   | Sales      | 60000  | 2021-03-20 |
| 3  | Carol | IT         | 70000  | 2020-06-10 |
| 4  | David | IT         | 55000  | 2023-02-28 |
| 5  | Eve   | Sales      | 45000  | 2023-05-12 |
```

### Query to Analyze:
```sql
SELECT 
    name,
    department,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as overall_rank,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank,
    SUM(salary) OVER (PARTITION BY department) as dept_total,
    AVG(salary) OVER () as company_avg
FROM employees
ORDER BY department, salary DESC;
```

### EXECUTION DRY RUN

#### Step 1: FROM clause - Load base data
```
Current working set:
| id | name  | department | salary | hire_date  |
|----|-------|------------|--------|------------|
| 1  | Alice | Sales      | 50000  | 2022-01-15 |
| 2  | Bob   | Sales      | 60000  | 2021-03-20 |
| 3  | Carol | IT         | 70000  | 2020-06-10 |
| 4  | David | IT         | 55000  | 2023-02-28 |
| 5  | Eve   | Sales      | 45000  | 2023-05-12 |
```

#### Step 2: Window Functions Execute (before final SELECT)

**Processing ROW_NUMBER() OVER (ORDER BY salary DESC)**

Sort all rows by salary DESC:
```
Sorted order: Carol(70000) → Bob(60000) → David(55000) → Alice(50000) → Eve(45000)
```

Assign row numbers:
- Carol gets 1 (highest salary)
- Bob gets 2  
- David gets 3
- Alice gets 4
- Eve gets 5 (lowest salary)

**Processing ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC)**

Create partitions:
```
IT Partition: Carol(70000), David(55000)
Sales Partition: Bob(60000), Alice(50000), Eve(45000)
```

Assign row numbers within each partition:

IT Partition (sorted by salary DESC):
- Carol gets 1 (highest in IT)
- David gets 2

Sales Partition (sorted by salary DESC):
- Bob gets 1 (highest in Sales)
- Alice gets 2
- Eve gets 3 (lowest in Sales)

**Processing SUM(salary) OVER (PARTITION BY department)**

Calculate sum for each partition:

IT Partition: 70000 + 55000 = 125000
Sales Partition: 60000 + 50000 + 45000 = 155000

Every row in IT gets 125000
Every row in Sales gets 155000

**Processing AVG(salary) OVER ()**

Calculate average of ALL salaries:
(70000 + 60000 + 55000 + 50000 + 45000) ÷ 5 = 280000 ÷ 5 = 56000

Every single row gets 56000

#### Step 3: SELECT clause - Choose columns with calculated values

```
| name  | department | salary | overall_rank | dept_rank | dept_total | company_avg |
|-------|------------|--------|--------------|-----------|------------|-------------|
| Alice | Sales      | 50000  | 4            | 2         | 155000     | 56000       |
| Bob   | Sales      | 60000  | 2            | 1         | 155000     | 56000       |
| Carol | IT         | 70000  | 1            | 1         | 125000     | 56000       |
| David | IT         | 55000  | 3            | 2         | 125000     | 56000       |
| Eve   | Sales      | 45000  | 5            | 3         | 155000     | 56000       |
```

#### Step 4: ORDER BY department, salary DESC

Final sorted result:
```
| name  | department | salary | overall_rank | dept_rank | dept_total | company_avg |
|-------|------------|--------|--------------|-----------|------------|-------------|
| Carol | IT         | 70000  | 1            | 1         | 125000     | 56000       |
| David | IT         | 55000  | 3            | 2         | 125000     | 56000       |
| Bob   | Sales      | 60000  | 2            | 1         | 155000     | 56000       |
| Alice | Sales      | 50000  | 4            | 2         | 155000     | 56000       |
| Eve   | Sales      | 45000  | 5            | 3         | 155000     | 56000       |
```

### KEY OBSERVATIONS

#### 1. Window Functions Preserve All Rows
Notice we still have all 5 original rows - nothing was collapsed like GROUP BY would do.

#### 2. PARTITION BY Creates Separate "Windows"
- `dept_rank`: IT employees ranked separately from Sales employees
- `dept_total`: Different totals for IT (125000) vs Sales (155000)

#### 3. Empty OVER() = Entire Result Set
- `company_avg`: Same value (56000) for every row because no PARTITION BY

#### 4. ORDER BY in OVER() Only Affects That Function
- `overall_rank` uses salary DESC for ranking
- Final ORDER BY is separate and affects display order only

#### 5. Each Row Gets Its Own Calculated Value
- Carol: overall_rank=1 (highest salary), dept_rank=1 (highest in IT)
- Bob: overall_rank=2 (second highest), dept_rank=1 (highest in Sales)

### MENTAL MODEL - The Story of This Query

Imagine you're MySQL processing this query:

1. "First, let me get all the employee data" (FROM clause)
2. "Now I need to calculate these window functions for each row..."

**For Alice's row:**
- "For overall_rank, I'll look at ALL employees sorted by salary. Alice has 50000, so she's 4th highest overall."
- "For dept_rank, I'll only look at Sales employees sorted by salary. Alice has 50000, Bob has 60000, Eve has 45000. So Alice is 2nd in Sales."
- "For dept_total, I'll sum all Sales salaries: 60000+50000+45000 = 155000"
- "For company_avg, I'll average ALL salaries: 280000÷5 = 56000"

**For Carol's row:**
- "For overall_rank, Carol has 70000 - highest overall, so rank 1"
- "For dept_rank, in IT partition, Carol has 70000 vs David's 55000, so rank 1 in IT"
- "For dept_total, IT sum: 70000+55000 = 125000"
- "For company_avg, same 56000 for everyone"

3. "Now I'll select the requested columns and sort by department, salary DESC"

### KEY INSIGHT - The Window Concept

Think of it like this:

1. **Start with your table**
2. **For each window function, MySQL creates a "view" of related rows**
   - `PARTITION BY department`: Creates IT group and Sales group
   - `ORDER BY salary DESC`: Sorts within each group
3. **Calculate the function for current row using its "window"**
4. **Attach result to that row**
5. **Move to next row and repeat**

**The key insight: Each row "sees" a different window of related rows based on the PARTITION BY and ORDER BY clauses!**

---

## 🎯 CTE Example: Median Calculation

```sql
WITH Ordered AS (
    SELECT 
        LAT_N,
        ROW_NUMBER() OVER (ORDER BY LAT_N) AS rn,
        COUNT(*) OVER () AS total_count
    FROM Station
)
SELECT AVG(LAT_N) AS median_lat
FROM Ordered
WHERE rn IN (FLOOR((total_count + 1)/2), CEIL((total_count + 1)/2));
```

**How it works:**
1. **CTE `Ordered`**: Sort values and assign row numbers, count total rows
2. **Main Query**: Find middle position(s) using FLOOR/CEIL math
3. **Median Logic**: 
   - Odd count (5): Position 3 → FLOOR=3, CEIL=3 → Same position
   - Even count (6): Positions 3,4 → FLOOR=3, CEIL=4 → Average both
4. **AVG()**: Handles both single value (odd) and two values (even)

This demonstrates using window functions to solve complex statistical problems that don't have built-in functions!

---

## 🧠 Critical Concepts to Keep in Mind

### The "Window" Mental Model
Think of window functions as creating a **dynamic spotlight** that moves with each row:
- For each row, the function "looks around" at related rows based on PARTITION BY and ORDER BY
- The "window" can be the entire table, a partition, or a specific frame of rows
- Each row gets its own personalized calculation based on what it can "see"

### Execution Order Reality Check
**Remember the execution sequence:**
```
1. FROM/JOIN → 2. WHERE → 3. GROUP BY → 4. HAVING → 
5. WINDOW FUNCTIONS → 6. SELECT → 7. ORDER BY → 8. LIMIT
```

This means:
- Window functions can't see the final SELECT aliases
- You can't filter on window function results in WHERE/HAVING
- Window functions run on the data AFTER filtering but BEFORE final sorting

### Frame Behavior Gotchas

**Default Frame Warning:**
```sql
-- These are DIFFERENT with duplicate values:
SUM(salary) OVER (ORDER BY hire_date)  -- RANGE frame (default)
SUM(salary) OVER (ORDER BY hire_date ROWS UNBOUNDED PRECEDING)  -- ROWS frame
```
- **RANGE** frame includes ALL rows with the same ORDER BY value
- **ROWS** frame counts physical positions only
- Always be explicit about frames when precision matters

### Common Anti-Patterns to Avoid

**1. Trying to filter directly:**
```sql
-- WRONG - This fails
SELECT * FROM employees WHERE ROW_NUMBER() OVER (ORDER BY salary) = 1;

-- CORRECT - Use subquery
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (ORDER BY salary) as rn FROM employees
) WHERE rn = 1;
```

**2. Forgetting about NULL handling:**
```sql
-- LAG/LEAD return NULL for missing values unless you specify default
LAG(salary, 1, 0) OVER (ORDER BY hire_date)  -- Returns 0 instead of NULL
```

**3. Performance trap:**
```sql
-- SLOW - No index support
ROW_NUMBER() OVER (ORDER BY CONCAT(first_name, last_name))

-- BETTER - Index on computed column or separate ORDER BY columns
ROW_NUMBER() OVER (ORDER BY last_name, first_name)
```

### Debugging Strategies

**1. Build complexity gradually:**
```sql
-- Start simple
SELECT name, salary FROM employees;

-- Add one window function
SELECT name, salary, ROW_NUMBER() OVER (ORDER BY salary) FROM employees;

-- Add partitioning
SELECT name, salary, ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary) FROM employees;
```

**2. Use meaningful aliases:**
```sql
-- Instead of: rn, rnk, sum_sal
-- Use descriptive names:
SELECT 
    ROW_NUMBER() OVER (...) as salary_rank_in_dept,
    SUM(salary) OVER (...) as running_dept_payroll
```

**3. Verify your partitions:**
```sql
-- Quick check: See your partitions
SELECT DISTINCT 
    department,
    COUNT(*) OVER (PARTITION BY department) as dept_size
FROM employees;
```

### Memory Aids

**RANK Function Differences:**
- **ROW_NUMBER()**: "Every person gets a unique number" (1,2,3,4,5...)
- **RANK()**: "Ties share rank, but we skip ahead" (1,2,2,4,5...)  
- **DENSE_RANK()**: "Ties share rank, no skipping" (1,2,2,3,4...)

**Frame Boundaries Memory Trick:**
```
UNBOUNDED PRECEDING = "From the very beginning"
N PRECEDING = "N steps backward"
CURRENT ROW = "Right here"
N FOLLOWING = "N steps forward"  
UNBOUNDED FOLLOWING = "To the very end"
```

### Real-World Wisdom

**When to use each approach:**
- **ROW_NUMBER()**: When you need exactly N results (top 10, pagination)
- **RANK()/DENSE_RANK()**: When ties matter (sports rankings, grade rankings)
- **LAG/LEAD**: Time series analysis, comparing periods
- **SUM/AVG with frames**: Moving averages, running totals
- **NTILE()**: Creating equal-sized groups for analysis

**Performance considerations:**
- Window functions can be expensive on large datasets
- Always index PARTITION BY and ORDER BY columns
- Consider if you can solve the problem with simpler aggregation first
- Use LIMIT in outer query when possible to reduce result set

### Final Reality Check Questions

Before writing any window function query, ask yourself:
1. **What is my "window"?** (What rows should each calculation consider?)
2. **Do I need partitioning?** (Separate calculations per group?)  
3. **What's my frame?** (All rows, or a sliding window?)
4. **Can I filter the results?** (Remember: subquery needed for filtering)
5. **Is this the simplest approach?** (Sometimes GROUP BY is cleaner)

**The Golden Rule:** Window functions are powerful but can be overkill. Use them when you need row-level calculations alongside aggregated insights. If you just need grouped summaries, stick with GROUP BY.

