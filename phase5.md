# SQL Phase 5: Subqueries & Advanced Concepts Tutorial

## Introduction to Phase 5
Welcome to advanced SQL! This phase covers the sophisticated techniques that demonstrate deep SQL knowledge. These concepts are commonly used in senior-level questions and real-world complex data analysis.

---

## Enhanced Sample Tables:

**employees table:**
| emp_id | name        | dept_id | salary | age | hire_date  | manager_id |
|--------|-------------|---------|--------|-----|------------|------------|
| 1      | John Smith  | 1       | 50000  | 25  | 2022-03-15 | 3          |
| 2      | Sarah Jones | 2       | 45000  | 30  | 2021-01-20 | 4          |
| 3      | Mike Wilson | 1       | 75000  | 35  | 2019-05-10 | NULL       |
| 4      | Lisa Brown  | 2       | 60000  | 32  | 2020-11-05 | NULL       |
| 5      | Tom Davis   | 1       | 55000  | 28  | 2021-06-20 | 3          |
| 6      | Amy White   | 3       | 48000  | 26  | 2022-09-12 | 7          |
| 7      | Bob Green   | 3       | 70000  | 40  | 2018-06-15 | NULL       |
| 8      | Kate Black  | 1       | 52000  | 27  | 2023-01-10 | 3          |
| 9      | Sam Brown   | 2       | 47000  | 29  | 2022-08-05 | 4          |

**departments table:**
| dept_id | dept_name | location  | budget  |
|---------|-----------|-----------|---------|
| 1       | IT        | Bangalore | 500000  |
| 2       | HR        | Mumbai    | 300000  |
| 3       | Finance   | Delhi     | 400000  |

**sales table:**
| sale_id | emp_id | amount | sale_date  | region |
|---------|--------|--------|------------|---------|
| 1       | 1      | 15000  | 2024-01-15 | North   |
| 2       | 3      | 25000  | 2024-01-20 | South   |
| 3       | 5      | 18000  | 2024-02-10 | North   |
| 4       | 1      | 12000  | 2024-02-15 | North   |
| 5       | 8      | 22000  | 2024-03-01 | East    |

---

## Lesson 1: Introduction to Subqueries

### What is a Subquery?
A subquery is a query inside another query. Think of it as asking a question to get information needed to answer a bigger question.

### 1.1 Simple Subquery in WHERE
```sql
-- Find employees earning more than the average salary
SELECT name, salary 
FROM employees 
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**How it works:**
1. Inner query calculates: `(SELECT AVG(salary) FROM employees)` → 57889
2. Outer query becomes: `WHERE salary > 57889`

**Result:**
| name        | salary |
|-------------|--------|
| Mike Wilson | 75000  |
| Lisa Brown  | 60000  |
| Bob Green   | 70000  |

### 1.2 Subquery in SELECT
```sql
-- Show each employee's salary and how it compares to company average
SELECT 
    name, 
    salary,
    (SELECT AVG(salary) FROM employees) as company_avg,
    salary - (SELECT AVG(salary) FROM employees) as difference
FROM employees;
```

### 1.3 Subquery with IN
```sql
-- Find employees working in departments located in Mumbai or Delhi
SELECT name, dept_id 
FROM employees 
WHERE dept_id IN (
    SELECT dept_id 
    FROM departments 
    WHERE location IN ('Mumbai', 'Delhi')
);
```

### Practice Time! 🎯
1. Find employees older than the average age
2. Find employees in the department with the highest budget
3. Show employee names and their salary difference from department average

**Answers:**
```sql
-- 1.
SELECT name, age FROM employees 
WHERE age > (SELECT AVG(age) FROM employees);

-- 2.
SELECT name FROM employees 
WHERE dept_id = (SELECT dept_id FROM departments ORDER BY budget DESC LIMIT 1);

-- 3.
SELECT 
    e.name, 
    e.salary,
    e.salary - dept_avg.avg_sal as difference
FROM employees e
INNER JOIN (
    SELECT dept_id, AVG(salary) as avg_sal 
    FROM employees 
    GROUP BY dept_id
) dept_avg ON e.dept_id = dept_avg.dept_id;
```

---

## Lesson 2: Correlated Subqueries

### What Makes it "Correlated"?
The inner query references columns from the outer query. It runs once for each row of the outer query.

### 2.1 Basic Correlated Subquery
```sql
-- Find employees earning more than their department average
SELECT 
    name, 
    salary, 
    dept_id
FROM employees e1
WHERE salary > (
    SELECT AVG(salary) 
    FROM employees e2 
    WHERE e2.dept_id = e1.dept_id  -- This makes it correlated!
);
```

**How it works:**
- For each employee (e1), calculate the average salary in their department (e2)
- Compare their salary to their department average

### 2.2 Correlated Subquery with EXISTS
```sql
-- Find employees who have made sales
SELECT name 
FROM employees e
WHERE EXISTS (
    SELECT 1 
    FROM sales s 
    WHERE s.emp_id = e.emp_id
);
```
**Result:**
| name        |
|-------------|
| John Smith  |
| Mike Wilson |
| Tom Davis   |
| Kate Black  |

### 2.3 NOT EXISTS
```sql
-- Find employees who have NOT made any sales
SELECT name 
FROM employees e
WHERE NOT EXISTS (
    SELECT 1 
    FROM sales s 
    WHERE s.emp_id = e.emp_id
);
```

---

## Lesson 3: Subqueries in FROM Clause (Derived Tables)

### 3.1 Basic Derived Table
```sql
-- Department statistics as a derived table
SELECT 
    dept_stats.dept_name,
    dept_stats.avg_salary,
    dept_stats.emp_count
FROM (
    SELECT 
        d.dept_name,
        AVG(e.salary) as avg_salary,
        COUNT(e.emp_id) as emp_count
    FROM departments d
    LEFT JOIN employees e ON d.dept_id = e.dept_id
    GROUP BY d.dept_id, d.dept_name
) dept_stats
WHERE dept_stats.emp_count > 2;
```

### 3.2 Ranking with Derived Tables
```sql
-- Find the 2nd highest paid employee
SELECT name, salary 
FROM (
    SELECT name, salary,
           ROW_NUMBER() OVER (ORDER BY salary DESC) as salary_rank
    FROM employees
) ranked_employees
WHERE salary_rank = 2;
```

---

## Lesson 4: Window Functions - The Power Tools

### What are Window Functions?
Window functions perform calculations across related rows without collapsing the result into groups.

### 4.1 ROW_NUMBER()
```sql
-- Assign row numbers based on salary ranking
SELECT 
    name, 
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as salary_rank
FROM employees;
```
**Result:**
| name        | salary | salary_rank |
|-------------|--------|-------------|
| Mike Wilson | 75000  | 1           |
| Bob Green   | 70000  | 2           |
| Lisa Brown  | 60000  | 3           |
| Tom Davis   | 55000  | 4           |
| Kate Black  | 52000  | 5           |

### 4.2 RANK() and DENSE_RANK()
```sql
-- Compare RANK vs DENSE_RANK with ties
SELECT 
    name, 
    salary,
    RANK() OVER (ORDER BY salary DESC) as rank_with_gaps,
    DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank
FROM employees;
```

**Difference:**
- **RANK():** Skips numbers after ties (1, 2, 2, 4, 5)
- **DENSE_RANK():** No gaps after ties (1, 2, 2, 3, 4)

### 4.3 PARTITION BY - Window Functions by Group
```sql
-- Rank employees within each department
SELECT 
    name, 
    dept_id,
    salary,
    RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) as dept_rank
FROM employees;
```
**Result:**
| name        | dept_id | salary | dept_rank |
|-------------|---------|--------|-----------|
| Mike Wilson | 1       | 75000  | 1         |
| Tom Davis   | 1       | 55000  | 2         |
| Kate Black  | 1       | 52000  | 3         |
| John Smith  | 1       | 50000  | 4         |
| Lisa Brown  | 2       | 60000  | 1         |
| Sam Brown   | 2       | 47000  | 2         |
| Sarah Jones | 2       | 45000  | 3         |

### 4.4 Other Useful Window Functions
```sql
-- Advanced window functions
SELECT 
    name, 
    salary,
    LAG(salary) OVER (ORDER BY salary) as prev_salary,
    LEAD(salary) OVER (ORDER BY salary) as next_salary,
    SUM(salary) OVER (ORDER BY salary) as running_total
FROM employees
ORDER BY salary;
```

---

## Lesson 5: Common Table Expressions (CTEs)

### What are CTEs?
CTEs make complex queries readable by breaking them into logical steps.

### 5.1 Basic CTE Syntax
```sql
-- Department averages using CTE
WITH dept_averages AS (
    SELECT 
        dept_id,
        AVG(salary) as avg_salary,
        COUNT(*) as emp_count
    FROM employees
    GROUP BY dept_id
)
SELECT 
    d.dept_name,
    da.avg_salary,
    da.emp_count
FROM dept_averages da
INNER JOIN departments d ON da.dept_id = d.dept_id
WHERE da.avg_salary > 50000;
```

### 5.2 Multiple CTEs
```sql
-- Complex analysis with multiple CTEs
WITH 
high_performers AS (
    SELECT emp_id, name, salary
    FROM employees 
    WHERE salary > (SELECT AVG(salary) FROM employees)
),
dept_stats AS (
    SELECT 
        dept_id,
        AVG(salary) as avg_salary
    FROM employees
    GROUP BY dept_id
)
SELECT 
    hp.name,
    hp.salary,
    d.dept_name,
    ds.avg_salary as dept_avg
FROM high_performers hp
INNER JOIN employees e ON hp.emp_id = e.emp_id
INNER JOIN departments d ON e.dept_id = d.dept_id
INNER JOIN dept_stats ds ON e.dept_id = ds.dept_id;
```

### 5.3 Recursive CTEs (Advanced)
```sql
-- Organization hierarchy (find all reports under a manager)
WITH RECURSIVE org_chart AS (
    -- Base case: top-level managers
    SELECT emp_id, name, manager_id, 1 as level
    FROM employees 
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: their reports
    SELECT e.emp_id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.emp_id
)
SELECT name, level FROM org_chart ORDER BY level, name;
```

---

## Lesson 6: Advanced Subquery Patterns

### 6.1 ANY and ALL Operators
```sql
-- Find employees earning more than ANY employee in Finance
SELECT name, salary 
FROM employees 
WHERE salary > ANY (
    SELECT salary 
    FROM employees e
    INNER JOIN departments d ON e.dept_id = d.dept_id
    WHERE d.dept_name = 'Finance'
);

-- Find employees earning more than ALL employees in Finance
SELECT name, salary 
FROM employees 
WHERE salary > ALL (
    SELECT salary 
    FROM employees e
    INNER JOIN departments d ON e.dept_id = d.dept_id
    WHERE d.dept_name = 'Finance'
);
```

### 6.2 Subqueries for Complex Filtering
```sql
-- Find departments where all employees earn above company average
SELECT d.dept_name 
FROM departments d
WHERE NOT EXISTS (
    SELECT 1 
    FROM employees e 
    WHERE e.dept_id = d.dept_id 
    AND e.salary <= (SELECT AVG(salary) FROM employees)
);
```

### 6.3 Top N per Group using Subqueries
```sql
-- Find top 2 highest paid employees in each department
SELECT name, dept_id, salary
FROM (
    SELECT 
        name, 
        dept_id, 
        salary,
        ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) as rn
    FROM employees
) ranked
WHERE rn <= 2;
```

---

## Lesson 7: Real Interview Problem Patterns

### Pattern 1: "Find Second Highest" Problems
**Very common in placements!**

```sql
-- Method 1: Using subquery
SELECT MAX(salary) as second_highest
FROM employees 
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Method 2: Using window functions (better)
SELECT DISTINCT salary as second_highest
FROM (
    SELECT salary, 
           DENSE_RANK() OVER (ORDER BY salary DESC) as salary_rank
    FROM employees
) ranked
WHERE salary_rank = 2;
```

### Pattern 2: "Above Average in Their Group"
```sql
-- Employees earning above their department average
SELECT 
    e.name,
    e.salary,
    d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2 
    WHERE e2.dept_id = e.dept_id
);
```

### Pattern 3: "Growth/Trend Analysis"
```sql
-- Monthly sales growth comparison
WITH monthly_sales AS (
    SELECT 
        DATE_FORMAT(sale_date, '%Y-%m') as month,
        SUM(amount) as total_sales
    FROM sales
    GROUP BY DATE_FORMAT(sale_date, '%Y-%m')
)
SELECT 
    month,
    total_sales,
    LAG(total_sales) OVER (ORDER BY month) as prev_month_sales,
    total_sales - LAG(total_sales) OVER (ORDER BY month) as growth
FROM monthly_sales;
```

---

## Lesson 8: Window Functions Deep Dive

### 8.1 Ranking Functions Comparison
```sql
SELECT 
    name, 
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as row_num,
    RANK() OVER (ORDER BY salary DESC) as rank_val,
    DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank,
    PERCENT_RANK() OVER (ORDER BY salary DESC) as percent_rank
FROM employees;
```

### 8.2 Aggregate Window Functions
```sql
-- Running totals and moving averages
SELECT 
    name,
    salary,
    SUM(salary) OVER (ORDER BY emp_id) as running_total,
    AVG(salary) OVER (ORDER BY emp_id ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg_3
FROM employees;
```

### 8.3 Practical Business Analytics
```sql
-- Sales performance with rankings and comparisons
SELECT 
    e.name,
    s.amount,
    s.sale_date,
    -- Rank within all sales
    RANK() OVER (ORDER BY s.amount DESC) as overall_rank,
    -- Rank within their department
    RANK() OVER (PARTITION BY e.dept_id ORDER BY s.amount DESC) as dept_rank,
    -- Compare to previous sale by same employee
    LAG(s.amount) OVER (PARTITION BY s.emp_id ORDER BY s.sale_date) as prev_sale,
    -- Running total of employee's sales
    SUM(s.amount) OVER (PARTITION BY s.emp_id ORDER BY s.sale_date) as emp_running_total
FROM sales s
INNER JOIN employees e ON s.emp_id = e.emp_id;
```

---

## Lesson 9: Advanced CTE Patterns

### 9.1 CTE for Complex Calculations
```sql
-- Employee performance scorecard
WITH 
employee_stats AS (
    SELECT 
        e.emp_id,
        e.name,
        e.salary,
        COUNT(s.sale_id) as total_sales,
        COALESCE(SUM(s.amount), 0) as total_revenue
    FROM employees e
    LEFT JOIN sales s ON e.emp_id = s.emp_id
    GROUP BY e.emp_id, e.name, e.salary
),
performance_metrics AS (
    SELECT 
        *,
        CASE 
            WHEN total_revenue > 30000 THEN 'High'
            WHEN total_revenue > 15000 THEN 'Medium'
            ELSE 'Low'
        END as performance_tier
    FROM employee_stats
)
SELECT 
    name,
    salary,
    total_sales,
    total_revenue,
    performance_tier
FROM performance_metrics
ORDER BY total_revenue DESC;
```

### 9.2 CTE for Data Quality Analysis
```sql
-- Find data inconsistencies
WITH 
salary_analysis AS (
    SELECT 
        dept_id,
        AVG(salary) as avg_dept_salary,
        MIN(salary) as min_dept_salary,
        MAX(salary) as max_dept_salary
    FROM employees
    GROUP BY dept_id
)
SELECT 
    e.name,
    e.salary,
    d.dept_name,
    sa.avg_dept_salary,
    CASE 
        WHEN e.salary > sa.max_dept_salary * 1.5 THEN 'Potential Outlier'
        WHEN e.salary < sa.min_dept_salary * 0.5 THEN 'Potential Error'
        ELSE 'Normal'
    END as salary_flag
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
INNER JOIN salary_analysis sa ON e.dept_id = sa.dept_id;
```

---

## Lesson 10: Complex Problem Solving

### Problem 1: Top Performers Analysis
**"Find employees who rank in top 2 by salary in their department AND have above-average sales"**

```sql
WITH 
dept_salary_ranks AS (
    SELECT 
        emp_id,
        name,
        dept_id,
        salary,
        RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) as dept_salary_rank
    FROM employees
),
employee_sales AS (
    SELECT 
        emp_id,
        COALESCE(SUM(amount), 0) as total_sales
    FROM employees e
    LEFT JOIN sales s ON e.emp_id = s.emp_id
    GROUP BY emp_id
),
avg_sales AS (
    SELECT AVG(total_sales) as company_avg_sales
    FROM employee_sales
)
SELECT 
    dsr.name,
    dsr.dept_id,
    dsr.salary,
    es.total_sales
FROM dept_salary_ranks dsr
INNER JOIN employee_sales es ON dsr.emp_id = es.emp_id
CROSS JOIN avg_sales avs
WHERE dsr.dept_salary_rank <= 2 
  AND es.total_sales > avs.company_avg_sales;
```

### Problem 2: Gap Analysis
**"Find missing employee IDs in sequence"**

```sql
WITH RECURSIVE id_sequence AS (
    SELECT 1 as id
    UNION ALL
    SELECT id + 1 
    FROM id_sequence 
    WHERE id < (SELECT MAX(emp_id) FROM employees)
)
SELECT s.id as missing_id
FROM id_sequence s
LEFT JOIN employees e ON s.id = e.emp_id
WHERE e.emp_id IS NULL;
```

---

## Platform-Specific Advanced Problems

### HackerRank Focus:
- **Alternative Queries section:** Master ALL problems here
- **Advanced Select:** Window function problems
- Focus on ranking and percentile problems

### LeetCode Hard Problems:
- **Problem 185:** Department Top Three Salaries
- **Problem 262:** Trips and Users  
- **Problem 601:** Human Traffic of Stadium
- **Problem 1479:** Sales by Day of the Week

### Sample LeetCode-Style Problem:
**"Write a query to find employees who earn the top 3 salaries in each department"**

```sql
WITH ranked_employees AS (
    SELECT 
        e.name,
        e.salary,
        d.dept_name,
        DENSE_RANK() OVER (PARTITION BY d.dept_id ORDER BY e.salary DESC) as salary_rank
    FROM employees e
    INNER JOIN departments d ON e.dept_id = d.dept_id
)
SELECT name, dept_name, salary
FROM ranked_employees 
WHERE salary_rank <= 3
ORDER BY dept_name, salary_rank;
```

---

## Phase 5 Practice Problems

### Level 1: Basic Subqueries
1. Find employees hired after the average hire date
2. Find departments with budget higher than average budget
3. List employees earning less than the maximum salary in HR department

### Level 2: Correlated Subqueries  
4. Find employees who are the highest paid in their department
5. Find employees older than the average age in their department
6. List employees who have made more sales than the average for their department

### Level 3: Window Functions
7. Rank employees by total sales amount
8. Find running total of salaries when ordered by hire date
9. For each employee, show their sales rank within their department

### Level 4: Complex CTEs
10. Create a report showing each employee's performance compared to department and company averages
11. Find employees whose salary growth rate is above department average
12. Identify departments where top performer earns more than 2x the department average

### Solutions:
```sql
-- 1.
SELECT name, hire_date FROM employees 
WHERE hire_date > (SELECT AVG(hire_date) FROM employees);

-- 2.
SELECT dept_name, budget FROM departments 
WHERE budget > (SELECT AVG(budget) FROM departments);

-- 3.
SELECT name, salary FROM employees 
WHERE salary < (
    SELECT MAX(e.salary) 
    FROM employees e 
    INNER JOIN departments d ON e.dept_id = d.dept_id 
    WHERE d.dept_name = 'HR'
);

-- 4.
SELECT name, salary FROM employees e1
WHERE salary = (
    SELECT MAX(salary) FROM employees e2 
    WHERE e2.dept_id = e1.dept_id
);

-- 5.
SELECT name, age FROM employees e1
WHERE age > (
    SELECT AVG(age) FROM employees e2 
    WHERE e2.dept_id = e1.dept_id
);

-- 6.
SELECT e.name FROM employees e
WHERE (
    SELECT COALESCE(SUM(amount), 0) FROM sales WHERE emp_id = e.emp_id
) > (
    SELECT AVG(emp_sales.total)
    FROM (
        SELECT emp_id, SUM(amount) as total 
        FROM sales GROUP BY emp_id
    ) emp_sales
);

-- 7.
WITH employee_sales AS (
    SELECT 
        e.name, 
        COALESCE(SUM(s.amount), 0) as total_sales
    FROM employees e
    LEFT JOIN sales s ON e.emp_id = s.emp_id
    GROUP BY e.emp_id, e.name
)
SELECT 
    name, 
    total_sales,
    RANK() OVER (ORDER BY total_sales DESC) as sales_rank
FROM employee_sales;

-- 8.
SELECT 
    name, 
    salary, 
    hire_date,
    SUM(salary) OVER (ORDER BY hire_date) as running_total
FROM employees;

-- 9.
SELECT 
    e.name,
    d.dept_name,
    COALESCE(SUM(s.amount), 0) as total_sales,
    RANK() OVER (PARTITION BY e.dept_id ORDER BY COALESCE(SUM(s.amount), 0) DESC) as dept_sales_rank
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
LEFT JOIN sales s ON e.emp_id = s.emp_id
GROUP BY e.emp_id, e.name, d.dept_name, e.dept_id;

-- 10.
WITH 
employee_metrics AS (
    SELECT 
        e.emp_id,
        e.name,
        e.salary,
        e.dept_id,
        COALESCE(SUM(s.amount), 0) as total_sales
    FROM employees e
    LEFT JOIN sales s ON e.emp_id = s.emp_id
    GROUP BY e.emp_id, e.name, e.salary, e.dept_id
),
benchmarks AS (
    SELECT 
        dept_id,
        AVG(salary) as dept_avg_salary,
        AVG(total_sales) as dept_avg_sales
    FROM employee_metrics
    GROUP BY dept_id
),
company_benchmarks AS (
    SELECT 
        AVG(salary) as company_avg_salary,
        AVG(total_sales) as company_avg_sales
    FROM employee_metrics
)
SELECT 
    em.name,
    em.salary,
    em.total_sales,
    b.dept_avg_salary,
    cb.company_avg_salary,
    CASE 
        WHEN em.salary > cb.company_avg_salary AND em.total_sales > cb.company_avg_sales THEN 'Star Performer'
        WHEN em.salary > b.dept_avg_salary AND em.total_sales > b.dept_avg_sales THEN 'Dept High Performer'
        ELSE 'Standard'
    END as performance_category
FROM employee_metrics em
INNER JOIN benchmarks b ON em.dept_id = b.dept_id
CROSS JOIN company_benchmarks cb;
```

---

## Lesson 11: Interview-Winning Techniques

### Technique 1: Breaking Down Complex Problems
**When faced with a complex question:**
1. **Identify what you need to calculate**
2. **Work backwards from the final result** 
3. **Use CTEs to break into logical steps**
4. **Test each step independently**

### Technique 2: Performance Considerations
```sql
-- Instead of multiple subqueries (slow)
SELECT name 
FROM employees 
WHERE salary > (SELECT AVG(salary) FROM employees)
  AND age > (SELECT AVG(age) FROM employees);

-- Use JOIN with aggregated data (faster)
WITH company_stats AS (
    SELECT AVG(salary) as avg_salary, AVG(age) as avg_age
    FROM employees
)
SELECT e.name 
FROM employees e
CROSS JOIN company_stats cs
WHERE e.salary > cs.avg_salary AND e.age > cs.avg_age;
```

### Technique 3: Handling Edge Cases
```sql
-- Always consider NULL values and empty results
SELECT 
    name,
    COALESCE(
        (SELECT COUNT(*) FROM sales WHERE emp_id = e.emp_id), 
        0
    ) as sales_count
FROM employees e;
```

---

## Lesson 12: Advanced Window Function Applications

### 12.1 Business Analytics with Windows
```sql
-- Comprehensive sales analytics
SELECT 
    e.name,
    s.sale_date,
    s.amount,
    -- Ranking
    ROW_NUMBER() OVER (PARTITION BY s.emp_id ORDER BY s.sale_date) as sale_sequence,
    RANK() OVER (ORDER BY s.amount DESC) as amount_rank,
    -- Comparisons
    LAG(s.amount) OVER (PARTITION BY s.emp_id ORDER BY s.sale_date) as prev_sale,
    s.amount - LAG(s.amount) OVER (PARTITION BY s.emp_id ORDER BY s.sale_date) as sale_growth,
    -- Aggregates
    SUM(s.amount) OVER (PARTITION BY s.emp_id ORDER BY s.sale_date) as cumulative_sales,
    AVG(s.amount) OVER (PARTITION BY s.emp_id) as emp_avg_sale
FROM sales s
INNER JOIN employees e ON s.emp_id = e.emp_id
ORDER BY e.name, s.sale_date;
```

### 12.2 Percentiles and Distribution
```sql
-- Salary distribution analysis
SELECT 
    name,
    salary,
    NTILE(4) OVER (ORDER BY salary) as salary_quartile,
    PERCENT_RANK() OVER (ORDER BY salary) as salary_percentile,
    CUME_DIST() OVER (ORDER BY salary) as cumulative_distribution
FROM employees;
```

---

## Common Interview Questions & Expert Solutions

### Question 1: "Consecutive Records Problem"
**"Find employees with consecutive sales on consecutive days"**

```sql
WITH daily_sales AS (
    SELECT 
        emp_id,
        sale_date,
        ROW_NUMBER() OVER (PARTITION BY emp_id ORDER BY sale_date) as rn,
        DATE_SUB(sale_date, INTERVAL ROW_NUMBER() OVER (PARTITION BY emp_id ORDER BY sale_date) DAY) as group_date
    FROM sales
),
consecutive_groups AS (
    SELECT 
        emp_id,
        group_date,
        COUNT(*) as consecutive_days
    FROM daily_sales
    GROUP BY emp_id, group_date
    HAVING COUNT(*) >= 2
)
SELECT DISTINCT e.name
FROM consecutive_groups cg
INNER JOIN employees e ON cg.emp_id = e.emp_id;
```

### Question 2: "Department Comparison Analysis"
**"Find departments performing better than company average in both salary and sales"**

```sql
WITH 
dept_performance AS (
    SELECT 
        d.dept_id,
        d.dept_name,
        AVG(e.salary) as avg_salary,
        COALESCE(AVG(s.amount), 0) as avg_sale
    FROM departments d
    LEFT JOIN employees e ON d.dept_id = e.dept_id
    LEFT JOIN sales s ON e.emp_id = s.emp_id
    GROUP BY d.dept_id, d.dept_name
),
company_benchmarks AS (
    SELECT 
        AVG(salary) as company_avg_salary,
        AVG(amount) as company_avg_sale
    FROM employees e
    LEFT JOIN sales s ON e.emp_id = s.emp_id
)
SELECT 
    dp.dept_name,
    dp.avg_salary,
    dp.avg_sale
FROM dept_performance dp
CROSS JOIN company_benchmarks cb
WHERE dp.avg_salary > cb.company_avg_salary 
  AND dp.avg_sale > cb.company_avg_sale;
```

---

## Phase 5 Completion Checklist ✅

By the end of Phase 5, you should master:
- ✅ All types of subqueries (scalar, row, table)
- ✅ Correlated vs non-correlated subqueries
- ✅ EXISTS, NOT EXISTS, ANY, ALL operators
- ✅ Window functions (RANK, ROW_NUMBER, LAG, LEAD)
- ✅ Common Table Expressions (CTEs)
- ✅ Recursive CTEs
- ✅ Complex analytical queries

---

## Phase 5 Final Assessment Problems

### Problem Set A: Core Concepts (Must Solve All)

**A1. Multi-level Subquery Challenge**
```sql
-- Find employees whose salary is above their department average 
-- AND above the overall company average
SELECT 
    e.name,
    e.salary,
    d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
WHERE e.salary > (SELECT AVG(salary) FROM employees)  -- Company avg
  AND e.salary > (  -- Department avg
    SELECT AVG(salary) 
    FROM employees e2 
    WHERE e2.dept_id = e.dept_id
  );
```

**A2. Complex EXISTS Pattern**
```sql
-- Find departments that have employees in ALL age groups (20s, 30s, 40s)
SELECT d.dept_name
FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e1 
    WHERE e1.dept_id = d.dept_id AND e1.age BETWEEN 20 AND 29
)
AND EXISTS (
    SELECT 1 FROM employees e2 
    WHERE e2.dept_id = d.dept_id AND e2.age BETWEEN 30 AND 39
)
AND EXISTS (
    SELECT 1 FROM employees e3 
    WHERE e3.dept_id = d.dept_id AND e3.age BETWEEN 40 AND 49
);
```

**A3. Advanced Window Function**
```sql
-- For each employee, show their sales rank within department 
-- and identify if they're in top 50% of their department
WITH sales_by_emp AS (
    SELECT 
        e.emp_id,
        e.name,
        e.dept_id,
        COALESCE(SUM(s.amount), 0) as total_sales
    FROM employees e
    LEFT JOIN sales s ON e.emp_id = s.emp_id
    GROUP BY e.emp_id, e.name, e.dept_id
)
SELECT 
    name,
    total_sales,
    RANK() OVER (PARTITION BY dept_id ORDER BY total_sales DESC) as dept_rank,
    COUNT(*) OVER (PARTITION BY dept_id) as dept_size,
    CASE 
        WHEN RANK() OVER (PARTITION BY dept_id ORDER BY total_sales DESC) <= 
             COUNT(*) OVER (PARTITION BY dept_id) / 2.0 
        THEN 'Top 50%'
        ELSE 'Bottom 50%'
    END as performance_tier
FROM sales_by_emp;
```

### Problem Set B: Interview Simulation (Time: 20 minutes each)

**B1. Classic "Nth Highest" Variation**
```sql
-- Find the 3rd highest salary in each department
WITH dept_salary_ranks AS (
    SELECT 
        name,
        salary,
        dept_id,
        DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) as salary_rank
    FROM employees
)
SELECT 
    d.dept_name,
    dsr.name,
    dsr.salary
FROM dept_salary_ranks dsr
INNER JOIN departments d ON dsr.dept_id = d.dept_id
WHERE dsr.salary_rank = 3;
```

**B2. Growth Analysis Problem**
```sql
-- Find employees whose sales show consistent growth (each sale > previous sale)
WITH employee_sales_ordered AS (
    SELECT 
        s.emp_id,
        s.amount,
        s.sale_date,
        LAG(s.amount) OVER (PARTITION BY s.emp_id ORDER BY s.sale_date) as prev_amount
    FROM sales s
),
growth_check AS (
    SELECT 
        emp_id,
        COUNT(*) as total_sales,
        SUM(CASE WHEN amount > COALESCE(prev_amount, 0) THEN 1 ELSE 0 END) as growth_sales
    FROM employee_sales_ordered
    GROUP BY emp_id
)
SELECT e.name
FROM growth_check gc
INNER JOIN employees e ON gc.emp_id = e.emp_id
WHERE gc.total_sales > 1 
  AND gc.growth_sales = gc.total_sales - 1;  -- All sales except first show growth
```

### Problem Set C: Advanced Patterns (Company-Specific)

**C1. Hierarchical Data (Microsoft/Amazon Style)**
```sql
-- Find all employees and their management chain depth
WITH RECURSIVE mgmt_chain AS (
    -- Base: Top-level managers
    SELECT emp_id, name, manager_id, 0 as chain_depth
    FROM employees 
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Add subordinates
    SELECT e.emp_id, e.name, e.manager_id, mc.chain_depth + 1
    FROM employees e
    INNER JOIN mgmt_chain mc ON e.manager_id = mc.emp_id
)
SELECT 
    name,
    chain_depth,
    CASE 
        WHEN chain_depth = 0 THEN 'CEO/Director'
        WHEN chain_depth = 1 THEN 'Department Head'
        WHEN chain_depth = 2 THEN 'Team Lead'
        ELSE 'Individual Contributor'
    END as hierarchy_level
FROM mgmt_chain
ORDER BY chain_depth, name;
```

**C2. Complex Business Logic (TCS/Infosys Style)**
```sql
-- Employee Performance Scorecard with multiple criteria
WITH 
performance_metrics AS (
    SELECT 
        e.emp_id,
        e.name,
        e.dept_id,
        e.salary,
        EXTRACT(YEAR FROM CURRENT_DATE) - EXTRACT(YEAR FROM e.hire_date) as tenure_years,
        COALESCE(COUNT(s.sale_id), 0) as sales_count,
        COALESCE(SUM(s.amount), 0) as total_revenue
    FROM employees e
    LEFT JOIN sales s ON e.emp_id = s.emp_id
    GROUP BY e.emp_id, e.name, e.dept_id, e.salary, e.hire_date
),
department_benchmarks AS (
    SELECT 
        dept_id,
        AVG(salary) as dept_avg_salary,
        AVG(total_revenue) as dept_avg_revenue
    FROM performance_metrics
    GROUP BY dept_id
),
scoring AS (
    SELECT 
        pm.*,
        db.dept_avg_salary,
        db.dept_avg_revenue,
        -- Scoring logic
        CASE WHEN pm.salary >= db.dept_avg_salary THEN 25 ELSE 0 END +
        CASE WHEN pm.total_revenue >= db.dept_avg_revenue THEN 25 ELSE 0 END +
        CASE WHEN pm.tenure_years >= 3 THEN 25 ELSE 0 END +
        CASE WHEN pm.sales_count >= 2 THEN 25 ELSE 0 END as performance_score
    FROM performance_metrics pm
    INNER JOIN department_benchmarks db ON pm.dept_id = db.dept_id
)
SELECT 
    name,
    performance_score,
    CASE 
        WHEN performance_score >= 75 THEN 'Outstanding'
        WHEN performance_score >= 50 THEN 'Good'
        WHEN performance_score >= 25 THEN 'Average'
        ELSE 'Needs Improvement'
    END as performance_rating
FROM scoring
ORDER BY performance_score DESC;
```

---

## Phase 5 Mastery Indicators

You're ready for Phase 6 when you can:
1. ✅ Write subqueries without referring to examples
2. ✅ Solve "Nth highest" problems in under 3 minutes
3. ✅ Use window functions for ranking and analysis
4. ✅ Write readable CTEs for complex problems
5. ✅ Solve HackerRank "Alternative Queries" with 90%+ accuracy
6. ✅ Complete LeetCode medium SQL problems in 10-15 minutes

---
