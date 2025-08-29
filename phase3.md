# SQL Phase 3: Aggregation & Grouping Tutorial

## Introduction to Phase 3
This is the phase where SQL becomes a powerful analytical tool. You'll learn to summarize data, find patterns, and generate business reports. **80% of placement SQL questions involve some form of aggregation!**

---

## Sample Tables for This Phase:

**employees table:**
| id | name        | department | salary | age | hire_date  | manager_id |
|----|-------------|------------|--------|-----|------------|------------|
| 1  | John Smith  | IT         | 50000  | 25  | 2022-03-15 | 5          |
| 2  | Sarah Jones | HR         | 45000  | 30  | 2021-01-20 | 6          |
| 3  | Mike Wilson | IT         | 55000  | 28  | 2020-05-10 | 5          |
| 4  | Lisa Brown  | Finance    | 48000  | 26  | 2023-02-14 | 7          |
| 5  | Tom Davis   | IT         | 75000  | 35  | 2019-08-10 | NULL       |
| 6  | Amy White   | HR         | 60000  | 32  | 2020-11-05 | NULL       |
| 7  | Bob Green   | Finance    | 70000  | 40  | 2018-06-15 | NULL       |

**orders table:**
| order_id | customer_name | amount | order_date | status    |
|----------|---------------|--------|------------|-----------|
| 1        | Alice         | 1200   | 2024-01-15 | Completed |
| 2        | Bob           | 800    | 2024-01-20 | Pending   |
| 3        | Alice         | 1500   | 2024-02-10 | Completed |
| 4        | Charlie       | 600    | 2024-02-15 | Cancelled |
| 5        | Bob           | 2000   | 2024-03-01 | Completed |

---

## Lesson 1: Basic Aggregate Functions

### The Big 5 Aggregate Functions:
1. **COUNT()** - How many?
2. **SUM()** - What's the total?
3. **AVG()** - What's the average?
4. **MAX()** - What's the highest?
5. **MIN()** - What's the lowest?

### 1.1 COUNT() - Counting Records
```sql
-- How many employees do we have?
SELECT COUNT(*) as total_employees FROM employees;
```
**Result:** 7

```sql
-- How many employees have phone numbers?
SELECT COUNT(phone) as employees_with_phone FROM employees;
-- Note: COUNT(column) ignores NULL values
```

```sql
-- Count unique departments
SELECT COUNT(DISTINCT department) as unique_departments FROM employees;
```
**Result:** 3

### 1.2 SUM() - Adding Values
```sql
-- What's the total salary expense?
SELECT SUM(salary) as total_salary_expense FROM employees;
```
**Result:** 403000

```sql
-- Total completed order value
SELECT SUM(amount) as total_completed FROM orders 
WHERE status = 'Completed';
```
**Result:** 4700

### 1.3 AVG() - Average Values
```sql
-- What's the average salary?
SELECT AVG(salary) as average_salary FROM employees;
```
**Result:** 57571.43

```sql
-- Average age of IT employees
SELECT AVG(age) as avg_it_age FROM employees 
WHERE department = 'IT';
```

### 1.4 MAX() and MIN()
```sql
-- Highest and lowest salaries
SELECT 
    MAX(salary) as highest_salary,
    MIN(salary) as lowest_salary 
FROM employees;
```
**Result:**
| highest_salary | lowest_salary |
|----------------|---------------|
| 75000          | 45000         |

### Practice Time! 🎯
1. Count total number of orders
2. Find the sum of all order amounts
3. What's the average order amount?
4. Find the earliest and latest order dates

**Answers:**
```sql
-- 1.
SELECT COUNT(*) FROM orders;

-- 2.
SELECT SUM(amount) FROM orders;

-- 3.
SELECT AVG(amount) FROM orders;

-- 4.
SELECT MIN(order_date) as earliest, MAX(order_date) as latest FROM orders;
```

---

## Lesson 2: GROUP BY - The Game Changer

### Why GROUP BY?
Instead of one summary for the entire table, you can get summaries for each category/group.

### 2.1 Basic GROUP BY
```sql
-- Count employees by department
SELECT department, COUNT(*) as employee_count 
FROM employees 
GROUP BY department;
```
**Result:**
| department | employee_count |
|------------|----------------|
| IT         | 3              |
| HR         | 2              |
| Finance    | 2              |

### 2.2 GROUP BY with Different Aggregates
```sql
-- Department salary analysis
SELECT 
    department,
    COUNT(*) as emp_count,
    SUM(salary) as total_salary,
    AVG(salary) as avg_salary,
    MAX(salary) as max_salary,
    MIN(salary) as min_salary
FROM employees 
GROUP BY department;
```
**Result:**
| department | emp_count | total_salary | avg_salary | max_salary | min_salary |
|------------|-----------|--------------|------------|------------|------------|
| IT         | 3         | 180000       | 60000      | 75000      | 50000      |
| HR         | 2         | 105000       | 52500      | 60000      | 45000      |
| Finance    | 2         | 118000       | 59000      | 70000      | 48000      |

### 2.3 GROUP BY with WHERE
**Important:** WHERE comes BEFORE GROUP BY
```sql
-- Average salary by department for employees hired after 2020
SELECT 
    department, 
    AVG(salary) as avg_salary
FROM employees 
WHERE hire_date > '2020-01-01'
GROUP BY department;
```

### 2.4 Multiple Column Grouping
```sql
-- Count employees by department and age group
SELECT 
    department,
    CASE 
        WHEN age < 30 THEN 'Young'
        WHEN age < 40 THEN 'Middle'
        ELSE 'Senior'
    END as age_group,
    COUNT(*) as count
FROM employees 
GROUP BY department, age_group;
```

---

## Lesson 3: HAVING Clause - Filtering Groups

### Why HAVING?
WHERE filters individual rows, but HAVING filters groups after aggregation.

### 3.1 Basic HAVING
```sql
-- Find departments with more than 2 employees
SELECT department, COUNT(*) as emp_count 
FROM employees 
GROUP BY department 
HAVING COUNT(*) > 2;
```
**Result:**
| department | emp_count |
|------------|-----------|
| IT         | 3         |

### 3.2 HAVING with Different Conditions
```sql
-- Departments with average salary above 55000
SELECT 
    department, 
    AVG(salary) as avg_salary 
FROM employees 
GROUP BY department 
HAVING AVG(salary) > 55000;
```

### 3.3 WHERE vs HAVING - Critical Difference
```sql
-- Find departments with avg salary > 55000, considering only employees hired after 2019
SELECT 
    department, 
    COUNT(*) as emp_count,
    AVG(salary) as avg_salary 
FROM employees 
WHERE hire_date > '2019-01-01'  -- Filter rows BEFORE grouping
GROUP BY department 
HAVING AVG(salary) > 55000;     -- Filter groups AFTER aggregation
```

### The Golden Rule:
- **WHERE:** Filters individual rows (before grouping)
- **HAVING:** Filters groups (after grouping)

---

## Lesson 4: Advanced Grouping Scenarios

### 4.1 Customer Analysis Example
```sql
-- Customer spending analysis
SELECT 
    customer_name,
    COUNT(*) as total_orders,
    SUM(amount) as total_spent,
    AVG(amount) as avg_order_value,
    MAX(amount) as largest_order
FROM orders 
WHERE status = 'Completed'
GROUP BY customer_name 
ORDER BY total_spent DESC;
```
**Result:**
| customer_name | total_orders | total_spent | avg_order_value | largest_order |
|---------------|--------------|-------------|-----------------|---------------|
| Alice         | 2            | 2700        | 1350.00         | 1500          |
| Bob           | 1            | 2000        | 2000.00         | 2000          |

### 4.2 Time-based Analysis
```sql
-- Monthly order summary
SELECT 
    YEAR(order_date) as order_year,
    MONTH(order_date) as order_month,
    COUNT(*) as order_count,
    SUM(amount) as monthly_revenue
FROM orders 
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY order_year, order_month;
```

### 4.3 Conditional Aggregation
```sql
-- Count different order statuses per customer
SELECT 
    customer_name,
    COUNT(CASE WHEN status = 'Completed' THEN 1 END) as completed_orders,
    COUNT(CASE WHEN status = 'Pending' THEN 1 END) as pending_orders,
    COUNT(CASE WHEN status = 'Cancelled' THEN 1 END) as cancelled_orders
FROM orders 
GROUP BY customer_name;
```

---

## Lesson 5: Real Interview Problems

### Problem 1: Employee Department Analysis
**Question:** "Find departments where the average salary is above the company average"

```sql
-- Step 1: Find company average
SELECT AVG(salary) FROM employees; -- Let's say it's 57571

-- Step 2: Find departments above this average
SELECT 
    department,
    AVG(salary) as dept_avg_salary
FROM employees 
GROUP BY department 
HAVING AVG(salary) > (SELECT AVG(salary) FROM employees);
```

### Problem 2: Customer Segmentation
**Question:** "Classify customers as 'High Value' (>1500 total), 'Medium Value' (500-1500), 'Low Value' (<500)"

```sql
SELECT 
    customer_name,
    SUM(amount) as total_spent,
    CASE 
        WHEN SUM(amount) > 1500 THEN 'High Value'
        WHEN SUM(amount) >= 500 THEN 'Medium Value'
        ELSE 'Low Value'
    END as customer_segment
FROM orders 
WHERE status = 'Completed'
GROUP BY customer_name;
```

### Problem 3: Performance Analysis
**Question:** "Find months where total revenue was above average monthly revenue"

```sql
-- Complex but common in placements
SELECT 
    YEAR(order_date) as year,
    MONTH(order_date) as month,
    SUM(amount) as monthly_revenue
FROM orders 
WHERE status = 'Completed'
GROUP BY YEAR(order_date), MONTH(order_date)
HAVING SUM(amount) > (
    SELECT AVG(monthly_total) FROM (
        SELECT SUM(amount) as monthly_total
        FROM orders 
        WHERE status = 'Completed'
        GROUP BY YEAR(order_date), MONTH(order_date)
    ) as monthly_sums
);
```

---

## Phase 3 Practice Problems

### Level 1: Basic Aggregation
1. Count total employees in each department
2. Find the highest salary in each department
3. Calculate total salary expense by department

### Level 2: GROUP BY + HAVING
4. Find departments with more than 1 employee
5. Find departments where average age is above 30
6. Find customers with more than 1 completed order

### Level 3: Complex Analysis
7. For each department, show count of employees and percentage of total workforce
8. Find customers who have placed orders in multiple months
9. Calculate running department statistics (min, max, avg salary) for departments with avg salary > 50000

### Solutions:
```sql
-- 1.
SELECT department, COUNT(*) FROM employees GROUP BY department;

-- 2.
SELECT department, MAX(salary) FROM employees GROUP BY department;

-- 3.
SELECT department, SUM(salary) FROM employees GROUP BY department;

-- 4.
SELECT department, COUNT(*) FROM employees 
GROUP BY department HAVING COUNT(*) > 1;

-- 5.
SELECT department, AVG(age) FROM employees 
GROUP BY department HAVING AVG(age) > 30;

-- 6.
SELECT customer_name, COUNT(*) FROM orders 
WHERE status = 'Completed' 
GROUP BY customer_name HAVING COUNT(*) > 1;

-- 7.
SELECT 
    department, 
    COUNT(*) as dept_count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM employees), 2) as percentage
FROM employees 
GROUP BY department;

-- 8.
SELECT customer_name, COUNT(DISTINCT MONTH(order_date)) as months_active
FROM orders 
GROUP BY customer_name 
HAVING COUNT(DISTINCT MONTH(order_date)) > 1;

-- 9.
SELECT 
    department,
    COUNT(*) as emp_count,
    MIN(salary) as min_salary,
    MAX(salary) as max_salary,
    AVG(salary) as avg_salary
FROM employees 
GROUP BY department 
HAVING AVG(salary) > 50000;
```

---

## Lesson 6: ORDER OF EXECUTION

**This is CRUCIAL for interviews!** SQL executes in this order:

1. **FROM** - Which table(s)
2. **WHERE** - Filter individual rows
3. **GROUP BY** - Create groups
4. **HAVING** - Filter groups
5. **SELECT** - Choose columns (aggregates calculated here)
6. **ORDER BY** - Sort final result
7. **LIMIT** - Limit final result

### Example demonstrating execution order:
```sql
SELECT department, AVG(salary) as avg_sal          -- 5. Calculate average
FROM employees                                       -- 1. From employees table
WHERE age > 25                                      -- 2. Only employees > 25
GROUP BY department                                 -- 3. Group by department
HAVING AVG(salary) > 50000                         -- 4. Only groups with avg > 50k
ORDER BY avg_sal DESC                              -- 6. Sort by average salary
LIMIT 2;                                           -- 7. Show top 2 only
```

---

## Lesson 7: Common Aggregation Patterns

### Pattern 1: Top N per Group
"Find the highest paid employee in each department"
```sql
-- Using window functions (Phase 5 topic, but good to see)
SELECT department, MAX(salary) as max_salary 
FROM employees 
GROUP BY department;
```

### Pattern 2: Percentage Analysis
"What percentage of total orders does each customer represent?"
```sql
SELECT 
    customer_name,
    COUNT(*) as customer_orders,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM orders), 2) as percentage
FROM orders 
GROUP BY customer_name;
```

### Pattern 3: Growth Analysis
"Compare this year vs last year performance"
```sql
SELECT 
    YEAR(order_date) as order_year,
    COUNT(*) as order_count,
    SUM(amount) as total_revenue
FROM orders 
WHERE status = 'Completed'
GROUP BY YEAR(order_date)
ORDER BY order_year;
```

---

## Lesson 8: Aggregation with Conditions

### CASE WHEN in Aggregation
Very powerful for creating custom metrics:

```sql
-- Count orders by status
SELECT 
    COUNT(CASE WHEN status = 'Completed' THEN 1 END) as completed_count,
    COUNT(CASE WHEN status = 'Pending' THEN 1 END) as pending_count,
    COUNT(CASE WHEN status = 'Cancelled' THEN 1 END) as cancelled_count
FROM orders;
```

### Conditional Sums
```sql
-- Revenue analysis by status
SELECT 
    customer_name,
    SUM(CASE WHEN status = 'Completed' THEN amount ELSE 0 END) as completed_revenue,
    SUM(CASE WHEN status = 'Pending' THEN amount ELSE 0 END) as pending_revenue
FROM orders 
GROUP BY customer_name;
```

---

## Common Interview Questions & Solutions

### Question 1: Department Performance
**"Find departments with average salary above company average"**

```sql
SELECT 
    department, 
    AVG(salary) as dept_avg
FROM employees 
GROUP BY department 
HAVING AVG(salary) > (SELECT AVG(salary) FROM employees);
```

### Question 2: Customer Analytics
**"Find customers who have spent more than $1000 in completed orders"**

```sql
SELECT 
    customer_name, 
    SUM(amount) as total_spent
FROM orders 
WHERE status = 'Completed'
GROUP BY customer_name 
HAVING SUM(amount) > 1000;
```

### Question 3: Business Intelligence
**"Create a monthly sales report showing order count, total revenue, and average order value"**

```sql
SELECT 
    YEAR(order_date) as year,
    MONTH(order_date) as month,
    COUNT(*) as order_count,
    SUM(amount) as total_revenue,
    AVG(amount) as avg_order_value
FROM orders 
WHERE status = 'Completed'
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY year, month;
```

---

## Critical Concepts for Placements

### 1. Aggregate Functions Don't Mix with Regular Columns
```sql
-- WRONG (will cause error)
SELECT name, COUNT(*) FROM employees;

-- CORRECT
SELECT department, COUNT(*) FROM employees GROUP BY department;
```

### 2. WHERE vs HAVING
```sql
-- Filter individual rows, then group
SELECT department, AVG(salary) 
FROM employees 
WHERE age > 25                    -- Individual row filter
GROUP BY department 
HAVING AVG(salary) > 50000;       -- Group filter
```

### 3. NULL Handling in Aggregations
```sql
-- COUNT(*) counts all rows including NULL
-- COUNT(column) ignores NULL values
SELECT 
    COUNT(*) as total_rows,
    COUNT(phone) as rows_with_phone
FROM employees;
```

---

## Phase 3 Practice Strategy

### Week 5 Focus:
- **Day 1-3:** Master basic aggregate functions
- **Day 4-5:** Understand GROUP BY thoroughly  
- **Day 6-7:** Practice WHERE + GROUP BY combinations

### Week 6 Focus:
- **Day 8-10:** Master HAVING clause
- **Day 11-12:** Complex grouping scenarios
- **Day 13-14:** Mixed problems and review

### Platform Recommendations:
- **HackerRank:** Complete entire "Aggregation" section
- **LeetCode:** Problems 182, 511, 512, 577
- **GeeksforGeeks:** SQL GROUP BY comprehensive exercises

---

## Phase 3 Completion Checklist ✅

You're ready for Phase 4 when you can:
- [ ] Use all 5 aggregate functions confidently
- [ ] Write GROUP BY queries without hesitation
- [ ] Understand the difference between WHERE and HAVING
- [ ] Handle multiple column grouping
- [ ] Solve conditional aggregation problems
- [ ] Understand SQL execution order
- [ ] Combine aggregation with string/date functions from Phase 2
- [ ] Solve 25-30 aggregation problems successfully

---

## Quick Reference: Aggregation

```sql
-- Basic aggregates
SELECT COUNT(*), SUM(column), AVG(column), MAX(column), MIN(column)
FROM table_name;

-- GROUP BY template
SELECT 
    grouping_column,
    aggregate_function(column)
FROM table_name 
WHERE condition                    -- Optional: filter rows
GROUP BY grouping_column 
HAVING aggregate_condition         -- Optional: filter groups
ORDER BY column;                   -- Optional: sort results

-- Multiple grouping
SELECT col1, col2, COUNT(*)
FROM table_name 
GROUP BY col1, col2;
```
