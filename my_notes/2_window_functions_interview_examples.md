# Window Functions - Common Interview Examples

Let's take a sample `employees` table for all examples:

| emp_id | name    | department | salary | hire_date  |
|--------|---------|------------|--------|------------|
| 1      | Alice   | IT         | 70000  | 2020-01-15 |
| 2      | Bob     | IT         | 80000  | 2019-03-10 |
| 3      | Charlie | IT         | 90000  | 2021-06-20 |
| 4      | David   | HR         | 60000  | 2020-08-05 |
| 5      | Eve     | HR         | 65000  | 2019-12-12 |
| 6      | Frank   | Finance    | 75000  | 2021-02-28 |
| 7      | Grace   | Finance    | 85000  | 2018-11-15 |
| 8      | Henry   | IT         | 95000  | 2022-01-10 |

---

## 1. Find Nth Highest Salary

**Problem:** Find the 3rd highest salary in the company.

```sql
SELECT DISTINCT salary
FROM (
    SELECT salary, 
           DENSE_RANK() OVER(ORDER BY salary DESC) as rank_sal
    FROM employees
) t
WHERE rank_sal = 3;
```

**Dry Run:**

Step 1: Apply DENSE_RANK()
| name    | salary | rank_sal |
|---------|--------|----------|
| Henry   | 95000  | 1        |
| Charlie | 90000  | 2        |
| Grace   | 85000  | 3        |
| Bob     | 80000  | 4        |
| Frank   | 75000  | 5        |
| Alice   | 70000  | 6        |
| Eve     | 65000  | 7        |
| David   | 60000  | 8        |

Step 2: Filter where rank_sal = 3
**Result:** 85000

**Q: Why DENSE_RANK instead of RANK?**
**A:** If there are salary ties, DENSE_RANK doesn't skip numbers. RANK would skip and might not give true "3rd highest".

---

## 2. Find Nth Highest Salary Per Department

**Problem:** Find the 2nd highest salary in each department.

```sql
SELECT department, name, salary
FROM (
    SELECT department, name, salary,
           DENSE_RANK() OVER(PARTITION BY department ORDER BY salary DESC) as rank_sal
    FROM employees
) t
WHERE rank_sal = 2;
```

**Dry Run:**

Step 1: Apply DENSE_RANK per department
| name    | department | salary | rank_sal |
|---------|------------|--------|----------|
| Henry   | IT         | 95000  | 1        |
| Charlie | IT         | 90000  | 2        |
| Bob     | IT         | 80000  | 3        |
| Alice   | IT         | 70000  | 4        |
| Grace   | Finance    | 85000  | 1        |
| Frank   | Finance    | 75000  | 2        |
| Eve     | HR         | 65000  | 1        |
| David   | HR         | 60000  | 2        |

Step 2: Filter where rank_sal = 2
**Result:**
| department | name    | salary |
|------------|---------|--------|
| IT         | Charlie | 90000  |
| Finance    | Frank   | 75000  |
| HR         | David   | 60000  |

---

## 3. Calculate Running Average Salary

**Problem:** Show each employee with their running average salary ordered by hire_date.

```sql
SELECT name, hire_date, salary,
       AVG(salary) OVER(ORDER BY hire_date ROWS UNBOUNDED PRECEDING) as running_avg
FROM employees
ORDER BY hire_date;
```

**Dry Run:**

Ordered by hire_date:
| name    | hire_date  | salary | running_avg |
|---------|------------|--------|-------------|
| Grace   | 2018-11-15 | 85000  | 85000       |
| Bob     | 2019-03-10 | 80000  | 82500       |
| Eve     | 2019-12-12 | 65000  | 76667       |
| Alice   | 2020-01-15 | 70000  | 75000       |
| David   | 2020-08-05 | 60000  | 72000       |
| Frank   | 2021-02-28 | 75000  | 72500       |
| Charlie | 2021-06-20 | 90000  | 75000       |
| Henry   | 2022-01-10 | 95000  | 77500       |

**Calculation:**
- Row 1: 85000/1 = 85000
- Row 2: (85000+80000)/2 = 82500
- Row 3: (85000+80000+65000)/3 = 76667
- And so on...

---

## 4. Find Median Salary

**Problem:** Calculate median salary using window functions.

```sql
SELECT DISTINCT
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) OVER() as median_salary
FROM employees;
```

**Alternative approach using ROW_NUMBER:**
```sql
SELECT AVG(salary) as median_salary
FROM (
    SELECT salary,
           ROW_NUMBER() OVER(ORDER BY salary) as row_asc,
           ROW_NUMBER() OVER(ORDER BY salary DESC) as row_desc
    FROM employees
) t
WHERE row_asc IN (row_desc, row_desc - 1, row_desc + 1);
```

**Dry Run for alternative approach:**

Step 1: Calculate row numbers
| salary | row_asc | row_desc |
|--------|---------|----------|
| 60000  | 1       | 8        |
| 65000  | 2       | 7        |
| 70000  | 3       | 6        |
| 75000  | 4       | 5        |
| 80000  | 5       | 4        |
| 85000  | 6       | 3        |
| 90000  | 7       | 2        |
| 95000  | 8       | 1        |

Step 2: Find middle values (where row_asc ≈ row_desc)
Middle rows: 75000 and 80000
**Median:** (75000 + 80000) / 2 = 77500

---

## 5. Calculate Salary Percentiles

**Problem:** Show each employee's salary percentile rank.

```sql
SELECT name, salary,
       PERCENT_RANK() OVER(ORDER BY salary) * 100 as percentile
FROM employees
ORDER BY salary;
```

**Dry Run:**

| name    | salary | percentile |
|---------|--------|------------|
| David   | 60000  | 0.0        |
| Eve     | 65000  | 14.3       |
| Alice   | 70000  | 28.6       |
| Frank   | 75000  | 42.9       |
| Bob     | 80000  | 57.1       |
| Grace   | 85000  | 71.4       |
| Charlie | 90000  | 85.7       |
| Henry   | 95000  | 100.0      |

**Formula:** PERCENT_RANK = (rank - 1) / (total_rows - 1)

---

## 6. Find Employees Earning More Than Department Average

**Problem:** List employees earning above their department's average salary.

```sql
SELECT name, department, salary, dept_avg
FROM (
    SELECT name, department, salary,
           AVG(salary) OVER(PARTITION BY department) as dept_avg
    FROM employees
) t
WHERE salary > dept_avg;
```

**Dry Run:**

Step 1: Calculate department averages
| name    | department | salary | dept_avg |
|---------|------------|--------|----------|
| Alice   | IT         | 70000  | 83750    |
| Bob     | IT         | 80000  | 83750    |
| Charlie | IT         | 90000  | 83750    |
| Henry   | IT         | 95000  | 83750    |
| David   | HR         | 60000  | 62500    |
| Eve     | HR         | 65000  | 62500    |
| Frank   | Finance    | 75000  | 80000    |
| Grace   | Finance    | 85000  | 80000    |

Step 2: Filter where salary > dept_avg
**Result:**
| name    | department | salary | dept_avg |
|---------|------------|--------|----------|
| Charlie | IT         | 90000  | 83750    |
| Henry   | IT         | 95000  | 83750    |
| Eve     | HR         | 65000  | 62500    |
| Grace   | Finance    | 85000  | 80000    |

---

## 7. Calculate Salary Growth Rate

**Problem:** Show each employee's salary compared to the previous employee (ordered by hire_date).

```sql
SELECT name, hire_date, salary,
       LAG(salary, 1) OVER(ORDER BY hire_date) as prev_salary,
       salary - LAG(salary, 1) OVER(ORDER BY hire_date) as salary_diff,
       ROUND(((salary - LAG(salary, 1) OVER(ORDER BY hire_date)) * 100.0 / 
              LAG(salary, 1) OVER(ORDER BY hire_date)), 2) as growth_rate_pct
FROM employees
ORDER BY hire_date;
```

**Dry Run:**

| name    | hire_date  | salary | prev_salary | salary_diff | growth_rate_pct |
|---------|------------|--------|-------------|-------------|-----------------|
| Grace   | 2018-11-15 | 85000  | NULL        | NULL        | NULL            |
| Bob     | 2019-03-10 | 80000  | 85000       | -5000       | -5.88           |
| Eve     | 2019-12-12 | 65000  | 80000       | -15000      | -18.75          |
| Alice   | 2020-01-15 | 70000  | 65000       | 5000        | 7.69            |
| David   | 2020-08-05 | 60000  | 70000       | -10000      | -14.29          |
| Frank   | 2021-02-28 | 75000  | 60000       | 15000       | 25.00           |
| Charlie | 2021-06-20 | 90000  | 75000       | 15000       | 20.00           |
| Henry   | 2022-01-10 | 95000  | 90000       | 5000        | 5.56            |

---

## 8. Find Top 3 Earners Per Department

**Problem:** Get top 3 highest-paid employees in each department.

```sql
SELECT department, name, salary, rank_sal
FROM (
    SELECT department, name, salary,
           DENSE_RANK() OVER(PARTITION BY department ORDER BY salary DESC) as rank_sal
    FROM employees
) t
WHERE rank_sal <= 3
ORDER BY department, rank_sal;
```

**Result:**
| department | name    | salary | rank_sal |
|------------|---------|--------|----------|
| Finance    | Grace   | 85000  | 1        |
| Finance    | Frank   | 75000  | 2        |
| HR         | Eve     | 65000  | 1        |
| HR         | David   | 60000  | 2        |
| IT         | Henry   | 95000  | 1        |
| IT         | Charlie | 90000  | 2        |
| IT         | Bob     | 80000  | 3        |

---

## 9. Calculate Moving Average (3-month window)

**Problem:** Calculate 3-employee moving average salary ordered by hire_date.

```sql
SELECT name, hire_date, salary,
       AVG(salary) OVER(ORDER BY hire_date ROWS 2 PRECEDING) as moving_avg_3
FROM employees
ORDER BY hire_date;
```

**Dry Run:**

| name    | hire_date  | salary | moving_avg_3 |
|---------|------------|--------|--------------|
| Grace   | 2018-11-15 | 85000  | 85000        |
| Bob     | 2019-03-10 | 80000  | 82500        |
| Eve     | 2019-12-12 | 65000  | 76667        |
| Alice   | 2020-01-15 | 70000  | 71667        |
| David   | 2020-08-05 | 60000  | 65000        |
| Frank   | 2021-02-28 | 75000  | 68333        |
| Charlie | 2021-06-20 | 90000  | 75000        |
| Henry   | 2022-01-10 | 95000  | 86667        |

**Explanation:** 
- Row 1: Only Grace (85000)
- Row 2: Grace + Bob (82500)
- Row 3: Grace + Bob + Eve (76667)
- Row 4: Bob + Eve + Alice (71667) - Grace drops out
- And so on...

---

## Key Interview Tips:

1. **Always use DENSE_RANK() for "Nth highest"** - avoids gaps with ties
2. **PARTITION BY is crucial** - resets calculations per group  
3. **Frame clauses (ROWS/RANGE)** - control exactly which rows to include
4. **LAG/LEAD for comparisons** - previous/next row analysis
5. **Subqueries often needed** - can't use window functions directly in WHERE

**Most Asked Questions:**
- Nth highest salary (company/department wise)
- Running totals and moving averages  
- Rank employees within departments
- Find employees above department average
- Calculate growth rates using LAG/LEAD