# SQL Phase 6: Optimization & Interview Mastery

## Duration: 14 days (Week 11-12)
## Goal: Master query performance, advanced patterns, and ace technical interviews

---

## Phase 6 Learning Objectives

By the end of this phase, you will:
- Optimize SQL queries for better performance
- Understand index usage and query execution plans
- Master complex interview patterns
- Handle edge cases and data quality issues
- Write production-ready SQL code
- Confidently solve any placement SQL question

---

## Lesson 1: Query Optimization Fundamentals

### 1.1 Understanding Query Execution Order
```sql
-- SQL execution order (not the order you write!)
-- 1. FROM
-- 2. WHERE  
-- 3. GROUP BY
-- 4. HAVING
-- 5. SELECT
-- 6. ORDER BY
-- 7. LIMIT
```

### 1.2 Index-Friendly Query Writing
```sql
-- ❌ Index not used (function on column)
SELECT * FROM employees WHERE YEAR(hire_date) = 2022;

-- ✅ Index can be used
SELECT * FROM employees WHERE hire_date >= '2022-01-01' AND hire_date < '2023-01-01';

-- ❌ Inefficient pattern matching
SELECT * FROM employees WHERE name LIKE '%smith%';

-- ✅ Efficient pattern matching (when possible)
SELECT * FROM employees WHERE name LIKE 'smith%';
```

### 1.3 JOIN Optimization Techniques
```sql
-- ❌ Inefficient: Large result set then filter
SELECT e.name, d.dept_name 
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
WHERE d.location = 'Bangalore';

-- ✅ Efficient: Filter first, then join
SELECT e.name, d.dept_name 
FROM employees e
INNER JOIN (
    SELECT dept_id, dept_name 
    FROM departments 
    WHERE location = 'Bangalore'
) d ON e.dept_id = d.dept_id;
```

---

## Lesson 2: Advanced Interview Patterns

### 2.1 The "Island and Gap" Problem Pattern
```sql
-- Find consecutive days with sales (islands)
WITH sales_with_prev AS (
    SELECT 
        sale_date,
        LAG(sale_date) OVER (ORDER BY sale_date) as prev_date,
        DATEDIFF(sale_date, LAG(sale_date) OVER (ORDER BY sale_date)) as day_diff
    FROM sales
    GROUP BY sale_date
),
island_groups AS (
    SELECT 
        sale_date,
        SUM(CASE WHEN day_diff != 1 OR day_diff IS NULL THEN 1 ELSE 0 END) 
        OVER (ORDER BY sale_date) as island_id
    FROM sales_with_prev
)
SELECT 
    MIN(sale_date) as island_start,
    MAX(sale_date) as island_end,
    COUNT(*) as consecutive_days
FROM island_groups
GROUP BY island_id
HAVING COUNT(*) >= 2  -- At least 2 consecutive days
ORDER BY island_start;
```

### 2.2 The "Running Calculation" Pattern
```sql
-- Running percentage of total sales
SELECT 
    emp_id,
    sale_date,
    amount,
    SUM(amount) OVER (ORDER BY sale_date) as running_total,
    ROUND(
        SUM(amount) OVER (ORDER BY sale_date) * 100.0 / 
        SUM(amount) OVER (), 
        2
    ) as running_percentage
FROM sales
ORDER BY sale_date;
```

### 2.3 The "Pivot-like" Analysis Pattern
```sql
-- Department headcount by age group (without PIVOT function)
SELECT 
    d.dept_name,
    COUNT(CASE WHEN e.age BETWEEN 20 AND 29 THEN 1 END) as twenties,
    COUNT(CASE WHEN e.age BETWEEN 30 AND 39 THEN 1 END) as thirties,
    COUNT(CASE WHEN e.age BETWEEN 40 AND 49 THEN 1 END) as forties,
    COUNT(*) as total_employees
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_id, d.dept_name;
```

---

## Lesson 3: Advanced CTE Patterns

### 3.1 Multi-Step Data Processing
```sql
-- Complex employee evaluation system
WITH 
-- Step 1: Calculate individual metrics
employee_metrics AS (
    SELECT 
        e.emp_id,
        e.name,
        e.dept_id,
        e.salary,
        e.age,
        EXTRACT(YEAR FROM CURRENT_DATE) - EXTRACT(YEAR FROM e.hire_date) as tenure_years,
        COALESCE(COUNT(s.sale_id), 0) as sales_count,
        COALESCE(SUM(s.amount), 0) as total_sales,
        COALESCE(AVG(s.amount), 0) as avg_sale_amount
    FROM employees e
    LEFT JOIN sales s ON e.emp_id = s.emp_id
    GROUP BY e.emp_id, e.name, e.dept_id, e.salary, e.age, e.hire_date
),
-- Step 2: Calculate department benchmarks
dept_benchmarks AS (
    SELECT 
        dept_id,
        AVG(salary) as dept_avg_salary,
        AVG(total_sales) as dept_avg_sales,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) as dept_median_salary
    FROM employee_metrics
    GROUP BY dept_id
),
-- Step 3: Calculate company benchmarks
company_benchmarks AS (
    SELECT 
        AVG(salary) as company_avg_salary,
        AVG(total_sales) as company_avg_sales,
        PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY salary) as company_90th_salary
    FROM employee_metrics
),
-- Step 4: Score employees
employee_scores AS (
    SELECT 
        em.*,
        db.dept_avg_salary,
        db.dept_avg_sales,
        cb.company_avg_salary,
        cb.company_90th_salary,
        -- Scoring system (out of 100)
        CASE WHEN em.salary >= cb.company_90th_salary THEN 30
             WHEN em.salary >= db.dept_avg_salary THEN 20
             WHEN em.salary >= cb.company_avg_salary THEN 15
             ELSE 10 END +
        CASE WHEN em.total_sales >= db.dept_avg_sales * 1.5 THEN 25
             WHEN em.total_sales >= db.dept_avg_sales THEN 20
             WHEN em.total_sales > 0 THEN 15
             ELSE 5 END +
        CASE WHEN em.tenure_years >= 5 THEN 25
             WHEN em.tenure_years >= 3 THEN 20
             WHEN em.tenure_years >= 1 THEN 15
             ELSE 10 END +
        CASE WHEN em.sales_count >= 3 THEN 20
             WHEN em.sales_count >= 2 THEN 15
             WHEN em.sales_count >= 1 THEN 10
             ELSE 5 END as total_score
    FROM employee_metrics em
    INNER JOIN dept_benchmarks db ON em.dept_id = db.dept_id
    CROSS JOIN company_benchmarks cb
)
-- Step 5: Final ranking and categorization
SELECT 
    name,
    dept_id,
    salary,
    total_sales,
    total_score,
    RANK() OVER (ORDER BY total_score DESC) as company_rank,
    RANK() OVER (PARTITION BY dept_id ORDER BY total_score DESC) as dept_rank,
    CASE 
        WHEN total_score >= 90 THEN 'Star Performer'
        WHEN total_score >= 75 THEN 'High Performer'
        WHEN total_score >= 60 THEN 'Good Performer'
        WHEN total_score >= 45 THEN 'Average Performer'
        ELSE 'Needs Improvement'
    END as performance_category
FROM employee_scores
ORDER BY total_score DESC;
```

### 3.2 Recursive CTE for Complex Hierarchies
```sql
-- Complete organizational structure with salary budgets
WITH RECURSIVE org_structure AS (
    -- Base case: Top-level managers
    SELECT 
        emp_id,
        name,
        salary,
        manager_id,
        dept_id,
        0 as level,
        CAST(name AS CHAR(1000)) as hierarchy_path,
        salary as team_salary_budget
    FROM employees 
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: Add subordinates
    SELECT 
        e.emp_id,
        e.name,
        e.salary,
        e.manager_id,
        e.dept_id,
        os.level + 1,
        CONCAT(os.hierarchy_path, ' -> ', e.name),
        os.team_salary_budget + e.salary
    FROM employees e
    INNER JOIN org_structure os ON e.manager_id = os.emp_id
)
SELECT 
    name,
    level,
    hierarchy_path,
    salary,
    team_salary_budget,
    (SELECT COUNT(*) FROM org_structure os2 WHERE os2.hierarchy_path LIKE CONCAT(os1.hierarchy_path, '%')) - 1 as total_subordinates
FROM org_structure os1
ORDER BY level, name;
```

---

## Lesson 4: Performance Optimization Strategies

### 4.1 Subquery vs JOIN Performance
```sql
-- ❌ Slower: Correlated subquery runs for each row
SELECT name, salary
FROM employees e
WHERE salary > (
    SELECT AVG(salary) 
    FROM employees e2 
    WHERE e2.dept_id = e.dept_id
);

-- ✅ Faster: Calculate once, then join
WITH dept_averages AS (
    SELECT dept_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY dept_id
)
SELECT e.name, e.salary
FROM employees e
INNER JOIN dept_averages da ON e.dept_id = da.dept_id
WHERE e.salary > da.avg_salary;
```

### 4.2 Window Function vs Self-Join
```sql
-- ❌ Complex self-join approach
SELECT DISTINCT
    e1.name,
    e1.salary,
    e1.dept_id
FROM employees e1
INNER JOIN employees e2 ON e1.dept_id = e2.dept_id
GROUP BY e1.emp_id, e1.name, e1.salary, e1.dept_id
HAVING e1.salary >= AVG(e2.salary);

-- ✅ Cleaner window function approach
SELECT name, salary, dept_id
FROM (
    SELECT 
        name, 
        salary, 
        dept_id,
        AVG(salary) OVER (PARTITION BY dept_id) as dept_avg_salary
    FROM employees
) ranked
WHERE salary >= dept_avg_salary;
```

### 4.3 EXISTS vs IN Performance
```sql
-- Generally faster for large datasets
SELECT name FROM employees e
WHERE EXISTS (
    SELECT 1 FROM sales s WHERE s.emp_id = e.emp_id
);

-- Can be slower with large subquery results
SELECT name FROM employees
WHERE emp_id IN (SELECT emp_id FROM sales);
```

---

## Lesson 5: Data Quality and Edge Case Handling

### 5.1 Comprehensive NULL Handling
```sql
-- Robust sales report handling various NULL scenarios
WITH clean_sales_data AS (
    SELECT 
        COALESCE(s.emp_id, -1) as emp_id,  -- Handle missing employee references
        COALESCE(e.name, 'Unknown Employee') as emp_name,
        COALESCE(d.dept_name, 'No Department') as dept_name,
        COALESCE(s.amount, 0) as sale_amount,
        COALESCE(s.sale_date, '1900-01-01') as sale_date,
        CASE 
            WHEN s.amount IS NULL THEN 'Missing Amount'
            WHEN s.amount <= 0 THEN 'Invalid Amount'
            WHEN e.emp_id IS NULL THEN 'Invalid Employee'
            ELSE 'Valid'
        END as data_quality_flag
    FROM sales s
    LEFT JOIN employees e ON s.emp_id = e.emp_id
    LEFT JOIN departments d ON e.dept_id = d.dept_id
)
SELECT 
    dept_name,
    COUNT(*) as total_records,
    COUNT(CASE WHEN data_quality_flag = 'Valid' THEN 1 END) as valid_records,
    SUM(CASE WHEN data_quality_flag = 'Valid' THEN sale_amount ELSE 0 END) as valid_sales_total,
    ROUND(
        COUNT(CASE WHEN data_quality_flag = 'Valid' THEN 1 END) * 100.0 / COUNT(*), 
        2
    ) as data_quality_percentage
FROM clean_sales_data
GROUP BY dept_name
ORDER BY data_quality_percentage DESC;
```

### 5.2 Handling Division by Zero and Edge Cases
```sql
-- Safe calculations with comprehensive error handling
SELECT 
    e.name,
    e.salary,
    COALESCE(s.sales_count, 0) as sales_count,
    COALESCE(s.total_sales, 0) as total_sales,
    -- Safe division with meaningful defaults
    CASE 
        WHEN COALESCE(s.sales_count, 0) = 0 THEN 0
        ELSE ROUND(COALESCE(s.total_sales, 0) / s.sales_count, 2)
    END as avg_sale_amount,
    -- Performance ratio with bounds checking
    CASE 
        WHEN e.salary = 0 THEN 'Invalid Salary'
        WHEN COALESCE(s.total_sales, 0) = 0 THEN 'No Sales'
        ELSE ROUND(COALESCE(s.total_sales, 0) / e.salary, 4)
    END as sales_to_salary_ratio
FROM employees e
LEFT JOIN (
    SELECT 
        emp_id, 
        COUNT(*) as sales_count, 
        SUM(amount) as total_sales
    FROM sales 
    GROUP BY emp_id
) s ON e.emp_id = s.emp_id;
```

---

## Lesson 6: Complex Interview Problem Patterns

### 6.1 Market Basket Analysis Pattern
```sql
-- Find employees who work in same department and have similar sales patterns
WITH employee_sales_profile AS (
    SELECT 
        e.emp_id,
        e.name,
        e.dept_id,
        COUNT(s.sale_id) as sales_frequency,
        AVG(s.amount) as avg_sale_amount,
        CASE 
            WHEN AVG(s.amount) >= 20000 THEN 'High Value'
            WHEN AVG(s.amount) >= 15000 THEN 'Medium Value'
            ELSE 'Low Value'
        END as sales_tier
    FROM employees e
    LEFT JOIN sales s ON e.emp_id = s.emp_id
    GROUP BY e.emp_id, e.name, e.dept_id
)
SELECT 
    esp1.name as employee1,
    esp2.name as employee2,
    esp1.dept_id,
    esp1.sales_tier,
    ABS(esp1.avg_sale_amount - esp2.avg_sale_amount) as sales_similarity
FROM employee_sales_profile esp1
INNER JOIN employee_sales_profile esp2 ON esp1.dept_id = esp2.dept_id 
    AND esp1.emp_id < esp2.emp_id  -- Avoid duplicates
    AND esp1.sales_tier = esp2.sales_tier
WHERE ABS(esp1.avg_sale_amount - esp2.avg_sale_amount) < 2000
ORDER BY esp1.dept_id, sales_similarity;
```

### 6.2 Time Series Analysis Pattern
```sql
-- Month-over-month growth analysis with trend identification
WITH monthly_metrics AS (
    SELECT 
        DATE_FORMAT(sale_date, '%Y-%m') as month,
        COUNT(DISTINCT emp_id) as active_sellers,
        COUNT(*) as total_transactions,
        SUM(amount) as total_revenue,
        AVG(amount) as avg_transaction_value
    FROM sales
    GROUP BY DATE_FORMAT(sale_date, '%Y-%m')
),
growth_analysis AS (
    SELECT 
        *,
        LAG(total_revenue) OVER (ORDER BY month) as prev_month_revenue,
        LAG(active_sellers) OVER (ORDER BY month) as prev_month_sellers,
        LAG(avg_transaction_value) OVER (ORDER BY month) as prev_avg_transaction
    FROM monthly_metrics
),
trend_calculation AS (
    SELECT 
        *,
        CASE 
            WHEN prev_month_revenue IS NULL THEN 'First Month'
            WHEN prev_month_revenue = 0 THEN 'No Previous Data'
            ELSE ROUND(
                ((total_revenue - prev_month_revenue) * 100.0 / prev_month_revenue), 
                2
            )
        END as revenue_growth_percent,
        CASE 
            WHEN total_revenue > COALESCE(prev_month_revenue, 0) THEN 'Growing'
            WHEN total_revenue = COALESCE(prev_month_revenue, 0) THEN 'Stable'
            ELSE 'Declining'
        END as trend_direction
    FROM growth_analysis
)
SELECT 
    month,
    total_revenue,
    active_sellers,
    avg_transaction_value,
    revenue_growth_percent,
    trend_direction,
    -- 3-month moving average
    AVG(total_revenue) OVER (
        ORDER BY month 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as three_month_avg
FROM trend_calculation
ORDER BY month;
```

---

## Lesson 7: Company-Specific Problem Types

### 7.1 TCS/Infosys Style: Business Logic Heavy
```sql
-- Employee bonus calculation with complex business rules
WITH 
bonus_calculation AS (
    SELECT 
        e.emp_id,
        e.name,
        e.salary,
        e.dept_id,
        EXTRACT(YEAR FROM CURRENT_DATE) - EXTRACT(YEAR FROM e.hire_date) as tenure,
        COALESCE(SUM(s.amount), 0) as total_sales,
        COUNT(s.sale_id) as sales_transactions,
        -- Base bonus percentage based on tenure
        CASE 
            WHEN EXTRACT(YEAR FROM CURRENT_DATE) - EXTRACT(YEAR FROM e.hire_date) >= 5 THEN 15
            WHEN EXTRACT(YEAR FROM CURRENT_DATE) - EXTRACT(YEAR FROM e.hire_date) >= 3 THEN 12
            WHEN EXTRACT(YEAR FROM CURRENT_DATE) - EXTRACT(YEAR FROM e.hire_date) >= 1 THEN 8
            ELSE 5
        END as base_bonus_percent
    FROM employees e
    LEFT JOIN sales s ON e.emp_id = s.emp_id
    GROUP BY e.emp_id, e.name, e.salary, e.dept_id, e.hire_date
),
performance_multipliers AS (
    SELECT 
        *,
        -- Performance multiplier based on sales
        CASE 
            WHEN total_sales >= 50000 THEN 1.5
            WHEN total_sales >= 30000 THEN 1.3
            WHEN total_sales >= 15000 THEN 1.1
            WHEN total_sales > 0 THEN 1.0
            ELSE 0.8
        END as performance_multiplier,
        -- Department factor
        CASE 
            WHEN dept_id = 1 THEN 1.2  -- IT gets tech bonus
            WHEN dept_id = 3 THEN 1.1  -- Finance gets accuracy bonus
            ELSE 1.0
        END as dept_multiplier
    FROM bonus_calculation
)
SELECT 
    name,
    salary,
    total_sales,
    tenure,
    base_bonus_percent,
    performance_multiplier,
    dept_multiplier,
    ROUND(
        salary * (base_bonus_percent / 100.0) * performance_multiplier * dept_multiplier,
        2
    ) as final_bonus,
    ROUND(
        (salary * (base_bonus_percent / 100.0) * performance_multiplier * dept_multiplier) / salary * 100,
        2
    ) as effective_bonus_percent
FROM performance_multipliers
ORDER BY final_bonus DESC;
```

### 7.2 Microsoft/Google Style: Algorithm-Heavy
```sql
-- Find the longest streak of consecutive sales days for each employee
WITH daily_sales AS (
    SELECT DISTINCT 
        emp_id, 
        sale_date,
        ROW_NUMBER() OVER (PARTITION BY emp_id ORDER BY sale_date) as row_num
    FROM sales
),
streak_groups AS (
    SELECT 
        emp_id,
        sale_date,
        row_num,
        DATE_SUB(sale_date, INTERVAL row_num DAY) as streak_group
    FROM daily_sales
),
streak_lengths AS (
    SELECT 
        emp_id,
        streak_group,
        COUNT(*) as streak_length,
        MIN(sale_date) as streak_start,
        MAX(sale_date) as streak_end
    FROM streak_groups
    GROUP BY emp_id, streak_group
),
max_streaks AS (
    SELECT 
        emp_id,
        MAX(streak_length) as longest_streak
    FROM streak_lengths
    GROUP BY emp_id
)
SELECT 
    e.name,
    ms.longest_streak,
    sl.streak_start,
    sl.streak_end,
    DATEDIFF(sl.streak_end, sl.streak_start) + 1 as days_span
FROM max_streaks ms
INNER JOIN employees e ON ms.emp_id = e.emp_id
INNER JOIN streak_lengths sl ON ms.emp_id = sl.emp_id 
    AND ms.longest_streak = sl.streak_length
ORDER BY ms.longest_streak DESC, e.name;
```

---

## Lesson 8: Advanced Window Functions

### 8.1 Complex Ranking Scenarios
```sql
-- Multi-criteria employee ranking with tie-breaking
WITH employee_rankings AS (
    SELECT 
        e.emp_id,
        e.name,
        e.dept_id,
        e.salary,
        e.hire_date,
        COALESCE(SUM(s.amount), 0) as total_sales,
        COUNT(s.sale_id) as sales_count,
        -- Multiple ranking criteria
        ROW_NUMBER() OVER (
            PARTITION BY e.dept_id 
            ORDER BY COALESCE(SUM(s.amount), 0) DESC, e.salary DESC, e.hire_date ASC
        ) as sales_rank,
        DENSE_RANK() OVER (
            PARTITION BY e.dept_id 
            ORDER BY e.salary DESC
        ) as salary_rank,
        PERCENT_RANK() OVER (
            PARTITION BY e.dept_id 
            ORDER BY COALESCE(SUM(s.amount), 0)
        ) as sales_percentile
    FROM employees e
    LEFT JOIN sales s ON e.emp_id = s.emp_id
    GROUP BY e.emp_id, e.name, e.dept_id, e.salary, e.hire_date
),
quartile_analysis AS (
    SELECT 
        *,
        NTILE(4) OVER (PARTITION BY dept_id ORDER BY total_sales) as sales_quartile,
        CASE 
            WHEN sales_percentile >= 0.9 THEN 'Top 10%'
            WHEN sales_percentile >= 0.75 THEN 'Top 25%'
            WHEN sales_percentile >= 0.5 THEN 'Top 50%'
            ELSE 'Bottom 50%'
        END as performance_tier
    FROM employee_rankings
)
SELECT 
    name,
    dept_id,
    salary,
    total_sales,
    sales_rank,
    salary_rank,
    sales_quartile,
    performance_tier,
    ROUND(sales_percentile * 100, 1) as sales_percentile_display
FROM quartile_analysis
ORDER BY dept_id, sales_rank;
```

### 8.2 Lead/Lag for Comparative Analysis
```sql
-- Employee progression tracking
SELECT 
    e.name,
    e.dept_id,
    EXTRACT(YEAR FROM s.sale_date) as year,
    SUM(s.amount) as yearly_sales,
    LAG(SUM(s.amount)) OVER (
        PARTITION BY e.emp_id 
        ORDER BY EXTRACT(YEAR FROM s.sale_date)
    ) as prev_year_sales,
    LEAD(SUM(s.amount)) OVER (
        PARTITION BY e.emp_id 
        ORDER BY EXTRACT(YEAR FROM s.sale_date)
    ) as next_year_sales,
    -- Year-over-year growth calculation
    CASE 
        WHEN LAG(SUM(s.amount)) OVER (
            PARTITION BY e.emp_id 
            ORDER BY EXTRACT(YEAR FROM s.sale_date)
        ) IS NULL THEN 'First Year'
        WHEN LAG(SUM(s.amount)) OVER (
            PARTITION BY e.emp_id 
            ORDER BY EXTRACT(YEAR FROM s.sale_date)
        ) = 0 THEN 'No Previous Sales'
        ELSE CONCAT(
            ROUND(
                (SUM(s.amount) - LAG(SUM(s.amount)) OVER (
                    PARTITION BY e.emp_id 
                    ORDER BY EXTRACT(YEAR FROM s.sale_date)
                )) * 100.0 / LAG(SUM(s.amount)) OVER (
                    PARTITION BY e.emp_id 
                    ORDER BY EXTRACT(YEAR FROM s.sale_date)
                ), 2
            ), '%'
        )
    END as yoy_growth
FROM employees e
INNER JOIN sales s ON e.emp_id = s.emp_id
GROUP BY e.emp_id, e.name, e.dept_id, EXTRACT(YEAR FROM s.sale_date)
ORDER BY e.name, year;
```

---

## Lesson 9: Interview-Specific Problem Solving

### 9.1 The "Nth Highest" Pattern (Multiple Approaches)
```sql
-- Method 1: Using DENSE_RANK() - Most reliable
SELECT name, salary
FROM (
    SELECT 
        name, 
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) as salary_rank
    FROM employees
) ranked
WHERE salary_rank = 3;  -- 3rd highest

-- Method 2: Using LIMIT with OFFSET - For unique salaries
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 2;  -- Skip top 2, get the 3rd

-- Method 3: Using subquery - Traditional approach
SELECT name, salary
FROM employees e1
WHERE 2 = (
    SELECT COUNT(DISTINCT salary)
    FROM employees e2
    WHERE e2.salary > e1.salary
);
```

### 9.2 Finding Duplicates and Data Inconsistencies
```sql
-- Comprehensive duplicate detection with context
WITH duplicate_analysis AS (
    SELECT 
        name,
        email,
        dept_id,
        COUNT(*) as occurrence_count,
        GROUP_CONCAT(emp_id ORDER BY emp_id) as emp_ids,
        MIN(hire_date) as first_hire,
        MAX(hire_date) as last_hire,
        COUNT(DISTINCT salary) as unique_salaries,
        MIN(salary) as min_salary,
        MAX(salary) as max_salary
    FROM employees
    GROUP BY name, email, dept_id
    HAVING COUNT(*) > 1
)
SELECT 
    name,
    email,
    dept_id,
    occurrence_count,
    emp_ids,
    first_hire,
    last_hire,
    DATEDIFF(last_hire, first_hire) as days_between_hires,
    CASE 
        WHEN unique_salaries = 1 THEN 'Same Salary'
        ELSE CONCAT('Salary Range: ', min_salary, ' - ', max_salary)
    END as salary_info,
    'Potential duplicate or rehire' as issue_type
FROM duplicate_analysis
ORDER BY occurrence_count DESC, name;
```

### 9.3 Median and Percentile Calculations
```sql
-- Calculate department salary statistics including median
WITH salary_stats AS (
    SELECT 
        dept_id,
        COUNT(*) as emp_count,
        MIN(salary) as min_salary,
        MAX(salary) as max_salary,
        AVG(salary) as avg_salary,
        STDDEV(salary) as salary_stddev
    FROM employees
    GROUP BY dept_id
),
salary_percentiles AS (
    SELECT 
        dept_id,
        -- Calculate various percentiles
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY salary) as q1_salary,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) as median_salary,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY salary) as q3_salary,
        PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY salary) as p90_salary,
        PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY salary) as p95_salary
    FROM employees
    GROUP BY dept_id
)
SELECT 
    d.dept_name,
    ss.emp_count,
    ss.min_salary,
    sp.q1_salary,
    sp.median_salary,
    ss.avg_salary,
    sp.q3_salary,
    sp.p90_salary,
    ss.max_salary,
    ROUND(ss.salary_stddev, 2) as salary_stddev,
    -- Salary distribution analysis
    CASE 
        WHEN ss.salary_stddev / ss.avg_salary < 0.1 THEN 'Very Uniform'
        WHEN ss.salary_stddev / ss.avg_salary < 0.2 THEN 'Uniform'
        WHEN ss.salary_stddev / ss.avg_salary < 0.3 THEN 'Moderate Variation'
        ELSE 'High Variation'
    END as salary_distribution
FROM departments d
INNER JOIN salary_stats ss ON d.dept_id = ss.dept_id
INNER JOIN salary_percentiles sp ON d.dept_id = sp.dept_id
ORDER BY ss.avg_salary DESC;
```

---

## Lesson 10: Production-Ready Query Patterns

### 10.1 Comprehensive Error Handling and Logging
```sql
-- Production-style sales analysis with audit trail
WITH data_validation AS (
    SELECT 
        s.sale_id,
        s.emp_id,
        s.amount,
        s.sale_date,
        e.name as emp_name,
        d.dept_name,
        -- Data quality checks
        CASE 
            WHEN s.amount IS NULL THEN 'NULL_AMOUNT'
            WHEN s.amount <= 0 THEN 'INVALID_AMOUNT'
            WHEN s.emp_id IS NULL THEN 'NULL_EMPLOYEE'
            WHEN e.emp_id IS NULL THEN 'EMPLOYEE_NOT_FOUND'
            WHEN s.sale_date > CURRENT_DATE THEN 'FUTURE_DATE'
            WHEN s.sale_date < '2020-01-01' THEN 'TOO_OLD'
            ELSE 'VALID'
        END as validation_status,
        -- Business rule checks
        CASE 
            WHEN s.amount > 100000 THEN 'HIGH_VALUE_TRANSACTION'
            WHEN s.amount > 50000 THEN 'MEDIUM_VALUE_TRANSACTION'
            ELSE 'NORMAL_TRANSACTION'
        END as transaction_category
    FROM sales s
    LEFT JOIN employees e ON s.emp_id = e.emp_id
    LEFT JOIN departments d ON e.dept_id = d.dept_id
),
processed_sales AS (
    SELECT 
        *,
        -- Only process valid data for metrics
        CASE 
            WHEN validation_status = 'VALID' THEN amount 
            ELSE 0 
        END as clean_amount,
        -- Create audit information
        CONCAT(
            'Record processed on: ', CURRENT_TIMESTAMP, 
            ' | Status: ', validation_status,
            ' | Category: ', transaction_category
        ) as audit_log
    FROM data_validation
)
SELECT 
    dept_name,
    COUNT(*) as total_transactions,
    COUNT(CASE WHEN validation_status = 'VALID' THEN 1 END) as valid_transactions,
    SUM(clean_amount) as total_valid_revenue,
    AVG(CASE WHEN validation_status = 'VALID' THEN amount END) as avg_valid_amount,
    -- Error summary
    GROUP_CONCAT(
        DISTINCT validation_status 
        ORDER BY validation_status 
        SEPARATOR ', '
    ) as issues_found,
    ROUND(
        COUNT(CASE WHEN validation_status = 'VALID' THEN 1 END) * 100.0 / COUNT(*),
        2
    ) as data_quality_score
FROM processed_sales
WHERE dept_name IS NOT NULL
GROUP BY dept_name
ORDER BY data_quality_score DESC;
```

### 10.2 Dynamic Query Building Pattern
```sql
-- Flexible reporting based on different time periods
WITH date_ranges AS (
    SELECT 
        'Last 7 Days' as period_name,
        DATE_SUB(CURRENT_DATE, INTERVAL 7 DAY) as start_date,
        CURRENT_DATE as end_date
    UNION ALL
    SELECT 
        'Last 30 Days',
        DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY),
        CURRENT_DATE
    UNION ALL
    SELECT 
        'Last 90 Days',
        DATE_SUB(CURRENT_DATE, INTERVAL 90 DAY),
        CURRENT_DATE
    UNION ALL
    SELECT 
        'Year to Date',
        DATE(CONCAT(YEAR(CURRENT_DATE), '-01-01')),
        CURRENT_DATE
),
period_metrics AS (
    SELECT 
        dr.period_name,
        COUNT(DISTINCT s.emp_id) as active_employees,
        COUNT(*) as total_sales,
        SUM(s.amount) as total_revenue,
        AVG(s.amount) as avg_sale_amount,
        MAX(s.amount) as max_sale,
        MIN(s.amount) as min_sale
    FROM date_ranges dr
    LEFT JOIN sales s ON s.sale_date BETWEEN dr.start_date AND dr.end_date
    GROUP BY dr.period_name, dr.start_date, dr.end_date
)
SELECT 
    period_name,
    active_employees,
    total_sales,
    FORMAT(total_revenue, 2) as formatted_revenue,
    ROUND(avg_sale_amount, 2) as avg_sale_amount,
    max_sale,
    min_sale,
    -- Performance indicators
    CASE 
        WHEN period_name = 'Last 7 Days' AND total_sales > 50 THEN 'Excellent'
        WHEN period_name = 'Last 30 Days' AND total_sales > 200 THEN 'Excellent'
        WHEN period_name = 'Last 90 Days' AND total_sales > 500 THEN 'Excellent'
        WHEN total_sales > 0 THEN 'Good'
        ELSE 'Needs Attention'
    END as performance_status
FROM period_metrics
ORDER BY FIELD(period_name, 'Last 7 Days', 'Last 30 Days', 'Last 90 Days', 'Year to Date');
```

---

## Lesson 11: Index Strategy and Performance Analysis

### 11.1 Query Plan Analysis Queries
```sql
-- Analyze table statistics for optimization decisions
SELECT 
    'employees' as table_name,
    COUNT(*) as row_count,
    COUNT(DISTINCT dept_id) as unique_departments,
    COUNT(DISTINCT manager_id) as unique_managers,
    AVG(salary) as avg_salary,
    STDDEV(salary) as salary_variance,
    COUNT(CASE WHEN hire_date > DATE_SUB(CURRENT_DATE, INTERVAL 1 YEAR) THEN 1 END) as new_hires_last_year
FROM employees

UNION ALL

SELECT 
    'sales' as table_name,
    COUNT(*) as row_count,
    COUNT(DISTINCT emp_id) as unique_employees,
    COUNT(DISTINCT DATE(sale_date)) as unique_sale_days,
    AVG(amount) as avg_amount,
    STDDEV(amount) as amount_variance,
    COUNT(CASE WHEN sale_date > DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY) THEN 1 END) as recent_sales
FROM sales;
```

### 11.2 Index Recommendation Logic
```sql
-- Identify columns that would benefit from indexing
WITH column_usage_analysis AS (
    SELECT 
        'employees.dept_id' as column_name,
        COUNT(DISTINCT dept_id) as cardinality,
        COUNT(*) as total_rows,
        ROUND(COUNT(DISTINCT dept_id) * 100.0 / COUNT(*), 2) as selectivity,
        'Frequently used in JOINs and WHERE clauses' as usage_pattern,
        'High' as index_priority
    FROM employees
    
    UNION ALL
    
    SELECT 
        'employees.salary',
        COUNT(DISTINCT salary),
        COUNT(*),
        ROUND(COUNT(DISTINCT salary) * 100.0 / COUNT(*), 2),
        'Used in range queries and ORDER BY',
        CASE 
            WHEN COUNT(DISTINCT salary) * 100.0 / COUNT(*) > 50 THEN 'High'
            WHEN COUNT(DISTINCT salary) * 100.0 / COUNT(*) > 20 THEN 'Medium'
            ELSE 'Low'
        END
    FROM employees
    
    UNION ALL
    
    SELECT 
        'sales.emp_id',
        COUNT(DISTINCT emp_id),
        COUNT(*),
        ROUND(COUNT(DISTINCT emp_id) * 100.0 / COUNT(*), 2),
        'Foreign key, frequently used in JOINs',
        'High'
    FROM sales
)
SELECT 
    column_name,
    cardinality,
    total_rows,
    selectivity,
    usage_pattern,
    index_priority,
    CASE 
        WHEN selectivity > 80 THEN 'Excellent for indexing'
        WHEN selectivity > 50 THEN 'Good for indexing'
        WHEN selectivity > 20 THEN 'Consider composite index'
        ELSE 'Low benefit from indexing'
    END as index_recommendation
FROM column_usage_analysis
ORDER BY 
    FIELD(index_priority, 'High', 'Medium', 'Low'),
    selectivity DESC;
```

---

## Lesson 12: Advanced Interview Questions & Solutions

### 12.1 The "Department Tournament" Problem
```sql
-- Find departments where every employee earns more than 
-- the average salary of any other department
WITH dept_salary_stats AS (
    SELECT 
        d.dept_id,
        d.dept_name,
        MIN(e.salary) as min_dept_salary,
        MAX(e.salary) as max_dept_salary,
        AVG(e.salary) as avg_dept_salary,
        COUNT(e.emp_id) as emp_count
    FROM departments d
    INNER JOIN employees e ON d.dept_id = e.dept_id
    GROUP BY d.dept_id, d.dept_name
),
cross_dept_comparison AS (
    SELECT 
        dss1.dept_name as dept1,
        dss2.dept_name as dept2,
        dss1.min_dept_salary as dept1_min,
        dss2.avg_dept_salary as dept2_avg,
        CASE 
            WHEN dss1.min_dept_salary > dss2.avg_dept_salary THEN 1 
            ELSE 0 
        END as dominates
    FROM dept_salary_stats dss1
    CROSS JOIN dept_salary_stats dss2
    WHERE dss1.dept_id != dss2.dept_id
),
dominant_departments AS (
    SELECT 
        dept1,
        COUNT(*) as departments_dominated,
        (SELECT COUNT(*) FROM dept_salary_stats) - 1 as total_other_departments
    FROM cross_dept_comparison
    WHERE dominates = 1
    GROUP BY dept1
)
SELECT 
    dd.dept1 as department,
    dd.departments_dominated,
    dd.total_other_departments,
    CASE 
        WHEN dd.departments_dominated = dd.total_other_departments THEN 'Dominates All'
        WHEN dd.departments_dominated >= dd.total_other_departments * 0.75 THEN 'Dominates Most'
        WHEN dd.departments_dominated >= dd.total_other_departments * 0.5 THEN 'Dominates Half'
        ELSE 'Limited Dominance'
    END as dominance_level,
    dss.avg_dept_salary,
    dss.emp_count
FROM dominant_departments dd
INNER JOIN dept_salary_stats dss ON dd.dept1 = dss.dept_name
ORDER BY dd.departments_dominated DESC;
```

### 12.2 The "Customer Lifecycle" Pattern
```sql
-- Complete customer journey analysis
WITH customer_lifecycle AS (
    SELECT 
        c.customer_id,
        c.name,
        c.join_date,
        MIN(o.order_date) as first_order_date,
        MAX(o.order_date) as last_order_date,
        COUNT(o.order_id) as total_orders,
        SUM(o.amount) as lifetime_value,
        AVG(o.amount) as avg_order_value,
        -- Calculate days between first and last order
        DATEDIFF(MAX(o.order_date), MIN(o.order_date)) as active_period_days,
        -- Days since last order
        DATEDIFF(CURRENT_DATE, MAX(o.order_date)) as days_since_last_order,
        -- Customer segment based on RFM analysis
        CASE 
            WHEN DATEDIFF(CURRENT_DATE, MAX(o.order_date)) <= 30 THEN 'Recent'
            WHEN DATEDIFF(CURRENT_DATE, MAX(o.order_date)) <= 90 THEN 'Moderate'
            WHEN DATEDIFF(CURRENT_DATE, MAX(o.order_date)) <= 180 THEN 'At Risk'
            ELSE 'Inactive'
        END as recency_segment,
        CASE 
            WHEN COUNT(o.order_id) >= 10 THEN 'High Frequency'
            WHEN COUNT(o.order_id) >= 5 THEN 'Medium Frequency'
            WHEN COUNT(o.order_id) >= 2 THEN 'Low Frequency'
            ELSE 'One-time'
        END as frequency_segment,
        CASE 
            WHEN SUM(o.amount) >= 50000 THEN 'High Value'
            WHEN SUM(o.amount) >= 20000 THEN 'Medium Value'
            WHEN SUM(o.amount) >= 5000 THEN 'Low Value'
            ELSE 'Minimal Value'
        END as monetary_segment
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.name, c.join_date
),
rfm_scoring AS (
    SELECT 
        *,
        -- RFM Score calculation
        CASE 
            WHEN recency_segment = 'Recent' THEN 4
            WHEN recency_segment = 'Moderate' THEN 3
            WHEN recency_segment = 'At Risk' THEN 2
            ELSE 1
        END +
        CASE 
            WHEN frequency_segment = 'High Frequency' THEN 4
            WHEN frequency_segment = 'Medium Frequency' THEN 3
            WHEN frequency_segment = 'Low Frequency' THEN 2
            ELSE 1
        END +
        CASE 
            WHEN monetary_segment = 'High Value' THEN 4
            WHEN monetary_segment = 'Medium Value' THEN 3
            WHEN monetary_segment = 'Low Value' THEN 2
            ELSE 1
        END as rfm_score
    FROM customer_lifecycle
)
SELECT 
    name,
    total_orders,
    lifetime_value,
    recency_segment,
    frequency_segment,
    monetary_segment,
    rfm_score,
    CASE 
        WHEN rfm_score >= 11 THEN 'Champions'
        WHEN rfm_score >= 9 THEN 'Loyal Customers'
        WHEN rfm_score >= 7 THEN 'Potential Loyalists'
        WHEN rfm_score >= 5 THEN 'At Risk'
        ELSE 'Lost Customers'
    END as customer_segment,
    -- Action recommendations
    CASE 
        WHEN rfm_score >= 11 THEN 'Reward and retain'
        WHEN rfm_score >= 9 THEN 'Upsell opportunities'
        WHEN rfm_score >= 7 THEN 'Engagement campaigns'
        WHEN rfm_score >= 5 THEN 'Reactivation needed'
        ELSE 'Win-back campaigns'
    END as recommended_action
FROM rfm_scoring
ORDER BY rfm_score DESC, lifetime_value DESC;
```

---

## Lesson 13: Interview Simulation Questions

### 13.1 Multi-Table Complex Analysis
```sql
-- Question: "Find departments where the top performer earns 
-- more than 3x the department average, and show the salary gap"
WITH dept_performance AS (
    SELECT 
        d.dept_name,
        d.dept_id,
        AVG(e.salary) as dept_avg_salary,
        MAX(e.salary) as top_salary,
        MIN(e.salary) as min_salary,
        COUNT(e.emp_id) as emp_count,
        STDDEV(e.salary) as salary_stddev
    FROM departments d
    INNER JOIN employees e ON d.dept_id = e.dept_id
    GROUP BY d.dept_id, d.dept_name
),
top_performers AS (
    SELECT 
        e.dept_id,
        e.name as top_performer_name,
        e.salary as top_performer_salary,
        ROW_NUMBER() OVER (PARTITION BY e.dept_id ORDER BY e.salary DESC) as salary_rank
    FROM employees e
)
SELECT 
    dp.dept_name,
    dp.emp_count,
    ROUND(dp.dept_avg_salary, 2) as dept_avg_salary,
    tp.top_performer_name,
    tp.top_performer_salary,
    ROUND(tp.top_performer_salary / dp.dept_avg_salary, 2) as salary_multiple,
    ROUND(tp.top_performer_salary - dp.dept_avg_salary, 2) as salary_gap,
    ROUND(dp.salary_stddev, 2) as salary_stddev,
    CASE 
        WHEN tp.top_performer_salary / dp.dept_avg_salary > 5 THEN 'Extreme Inequality'
        WHEN tp.top_performer_salary / dp.dept_avg_salary > 3 THEN 'High Inequality'
        WHEN tp.top_performer_salary / dp.dept_avg_salary > 2 THEN 'Moderate Inequality'
        ELSE 'Balanced'
    END as salary_distribution_type
FROM dept_performance dp
INNER JOIN top_performers tp ON dp.dept_id = tp.dept_id 
WHERE tp.salary_rank = 1
    AND tp.top_performer_salary > dp.dept_avg_salary * 3
ORDER BY salary_multiple DESC;
```

### 13.2 Time-Based Complex Query
```sql
-- Question: "Show quarter-over-quarter sales growth by department 
-- and identify departments with declining trends"
WITH quarterly_sales AS (
    SELECT 
        e.dept_id,
        d.dept_name,
        YEAR(s.sale_date) as year,
        QUARTER(s.sale_date) as quarter,
        CONCAT(YEAR(s.sale_date), '-Q', QUARTER(s.sale_date)) as year_quarter,
        COUNT(*) as transactions,
        SUM(s.amount) as revenue,
        COUNT(DISTINCT s.emp_id) as active_sellers,
        AVG(s.amount) as avg_transaction
    FROM sales s
    INNER JOIN employees e ON s.emp_id = e.emp_id
    INNER JOIN departments d ON e.dept_id = d.dept_id
    GROUP BY e.dept_id, d.dept_name, YEAR(s.sale_date), QUARTER(s.sale_date)
),
growth_calculation AS (
    SELECT 
        *,
        LAG(revenue) OVER (
            PARTITION BY dept_id 
            ORDER BY year, quarter
        ) as prev_quarter_revenue,
        LAG(year_quarter) OVER (
            PARTITION BY dept_id 
            ORDER BY year, quarter
        ) as prev_quarter,
        -- Calculate growth percentage
        CASE 
            WHEN LAG(revenue) OVER (
                PARTITION BY dept_id 
                ORDER BY year, quarter
            ) IS NULL THEN 'First Quarter'
            WHEN LAG(revenue) OVER (
                PARTITION BY dept_id 
                ORDER BY year, quarter
            ) = 0 THEN 'No Previous Revenue'
            ELSE CONCAT(
                ROUND(
                    (revenue - LAG(revenue) OVER (
                        PARTITION BY dept_id 
                        ORDER BY year, quarter
                    )) * 100.0 / LAG(revenue) OVER (
                        PARTITION BY dept_id 
                        ORDER BY year, quarter
                    ), 2
                ), '%'
            )
        END as qoq_growth,
        -- Growth trend indicator
        CASE 
            WHEN revenue > LAG(revenue) OVER (
                PARTITION BY dept_id 
                ORDER BY year, quarter
            ) THEN 1
            WHEN revenue = LAG(revenue) OVER (
                PARTITION BY dept_id 
                ORDER BY year, quarter
            ) THEN 0
            ELSE -1
        END as growth_direction
    FROM quarterly_sales
),
trend_analysis AS (
    SELECT 
        dept_id,
        dept_name,
        COUNT(*) as quarters_analyzed,
        SUM(growth_direction) as net_trend_score,
        COUNT(CASE WHEN growth_direction = 1 THEN 1 END) as growth_quarters,
        COUNT(CASE WHEN growth_direction = -1 THEN 1 END) as decline_quarters,
        AVG(revenue) as avg_quarterly_revenue
    FROM growth_calculation
    WHERE prev_quarter_revenue IS NOT NULL
    GROUP BY dept_id, dept_name
)
SELECT 
    ta.dept_name,
    ta.quarters_analyzed,
    ta.growth_quarters,
    ta.decline_quarters,
    ta.net_trend_score,
    ROUND(ta.avg_quarterly_revenue, 2) as avg_quarterly_revenue,
    CASE 
        WHEN ta.net_trend_score > ta.quarters_analyzed * 0.5 THEN 'Strong Growth'
        WHEN ta.net_trend_score > 0 THEN 'Moderate Growth'
        WHEN ta.net_trend_score = 0 THEN 'Stable'
        WHEN ta.net_trend_score > -(ta.quarters_analyzed * 0.5) THEN 'Moderate Decline'
        ELSE 'Strong Decline'
    END as trend_classification,
    -- Latest quarter performance
    (SELECT CONCAT(gc.year_quarter, ': ', FORMAT(gc.revenue, 2))
     FROM growth_calculation gc
     WHERE gc.dept_id = ta.dept_id
     ORDER BY gc.year DESC, gc.quarter DESC
     LIMIT 1) as latest_quarter_info
FROM trend_analysis ta
ORDER BY ta.net_trend_score DESC;
```

---

## Lesson 14: Mock Interview Questions

### Practice Question Set A: Algorithm & Logic
```sql
-- Q1: Find employees who have sales in every month of the current year
WITH monthly_sales AS (
    SELECT DISTINCT 
        emp_id, 
        MONTH(sale_date) as month
    FROM sales 
    WHERE YEAR(sale_date) = YEAR(CURRENT_DATE)
),
emp_month_count AS (
    SELECT 
        emp_id, 
        COUNT(DISTINCT month) as months_with_sales
    FROM monthly_sales
    GROUP BY emp_id
)
SELECT 
    e.name,
    emc.months_with_sales,
    'Has sales in every month' as achievement
FROM employees e
INNER JOIN emp_month_count emc ON e.emp_id = emc.emp_id
WHERE emc.months_with_sales = 12;  -- All 12 months

-- Q2: Find the median salary by department (without PERCENTILE_CONT)
WITH salary_rankings AS (
    SELECT 
        dept_id,
        salary,
        ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary) as row_asc,
        ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) as row_desc,
        COUNT(*) OVER (PARTITION BY dept_id) as total_count
    FROM employees
)
SELECT 
    d.dept_name,
    CASE 
        WHEN sr.total_count % 2 = 1 THEN 
            -- Odd number of employees: take middle value
            (SELECT salary FROM salary_rankings sr2 
             WHERE sr2.dept_id = sr.dept_id 
               AND sr2.row_asc = (sr.total_count + 1) / 2)
        ELSE 
            -- Even number: average of two middle values
            (SELECT AVG(salary) FROM salary_rankings sr2 
             WHERE sr2.dept_id = sr.dept_id 
               AND sr2.row_asc IN (sr.total_count / 2, sr.total_count / 2 + 1))
    END as median_salary
FROM salary_rankings sr
INNER JOIN departments d ON sr.dept_id = d.dept_id
WHERE sr.row_asc = sr.row_desc  -- This gives us one row per department
ORDER BY median_salary DESC;
```

### Practice Question Set B: Business Logic
```sql
-- Q3: Calculate employee performance score based on multiple factors
WITH performance_metrics AS (
    SELECT 
        e.emp_id,
        e.name,
        e.dept_id,
        e.salary,
        DATEDIFF(CURRENT_DATE, e.hire_date) / 365 as tenure_years,
        COUNT(s.sale_id) as total_sales_count,
        COALESCE(SUM(s.amount), 0) as total_sales_value,
        COALESCE(AVG(s.amount), 0) as avg_sale_value,
        -- Attendance calculation (assuming 250 working days/year)
        LEAST(
            DATEDIFF(CURRENT_DATE, e.hire_date) / 365 * 250, 
            COUNT(DISTINCT DATE(s.sale_date)) * 1.5  -- Estimate attendance based on sales activity
        ) as estimated_attendance_score
    FROM employees e
    LEFT JOIN sales s ON e.emp_id = s.emp_id
    GROUP BY e.emp_id, e.name, e.dept_id, e.salary, e.hire_date
),
scoring_system AS (
    SELECT 
        *,
        -- Sales performance score (40% weightage)
        CASE 
            WHEN total_sales_value >= 100000 THEN 40
            WHEN total_sales_value >= 75000 THEN 35
            WHEN total_sales_value >= 50000 THEN 30
            WHEN total_sales_value >= 25000 THEN 25
            WHEN total_sales_value > 0 THEN 20
            ELSE 10
        END as sales_score,
        -- Consistency score (30% weightage)
        CASE 
            WHEN total_sales_count >= 50 THEN 30
            WHEN total_sales_count >= 30 THEN 25
            WHEN total_sales_count >= 20 THEN 20
            WHEN total_sales_count >= 10 THEN 15
            WHEN total_sales_count > 0 THEN 10
            ELSE 5
        END as consistency_score,
        -- Experience bonus (20% weightage)
        CASE 
            WHEN tenure_years >= 5 THEN 20
            WHEN tenure_years >= 3 THEN 15
            WHEN tenure_years >= 2 THEN 12
            WHEN tenure_years >= 1 THEN 8
            ELSE 5
        END as experience_score,
        -- Quality score based on average transaction (10% weightage)
        CASE 
            WHEN avg_sale_value >= 5000 THEN 10
            WHEN avg_sale_value >= 3000 THEN 8
            WHEN avg_sale_value >= 2000 THEN 6
            WHEN avg_sale_value >= 1000 THEN 4
            ELSE 2
        END as quality_score
    FROM performance_metrics
)
SELECT 
    name,
    dept_id,
    total_sales_value,
    total_sales_count,
    ROUND(tenure_years, 1) as tenure_years,
    sales_score,
    consistency_score,
    experience_score,
    quality_score,
    (sales_score + consistency_score + experience_score + quality_score) as total_performance_score,
    RANK() OVER (ORDER BY (sales_score + consistency_score + experience_score + quality_score) DESC) as overall_rank,
    RANK() OVER (PARTITION BY dept_id ORDER BY (sales_score + consistency_score + experience_score + quality_score) DESC) as dept_rank,
    CASE 
        WHEN (sales_score + consistency_score + experience_score + quality_score) >= 90 THEN 'Exceptional'
        WHEN (sales_score + consistency_score + experience_score + quality_score) >= 75 THEN 'Outstanding'
        WHEN (sales_score + consistency_score + experience_score + quality_score) >= 60 THEN 'Good'
        WHEN (sales_score + consistency_score + experience_score + quality_score) >= 45 THEN 'Average'
        ELSE 'Needs Improvement'
    END as performance_grade
FROM scoring_system
ORDER BY total_performance_score DESC;
```

---

## Lesson 15: Interview Preparation Strategy

### 15.1 Common Interview Question Categories

#### Category 1: Ranking and Top-N Problems
```sql
-- Template for ranking problems
SELECT 
    [columns],
    ROW_NUMBER() OVER (PARTITION BY [group] ORDER BY [criteria]) as rank,
    DENSE_RANK() OVER (PARTITION BY [group] ORDER BY [criteria]) as dense_rank,
    RANK() OVER (PARTITION BY [group] ORDER BY [criteria]) as rank_with_ties
FROM [table]
WHERE rank <= N;  -- Get top N
```

#### Category 2: Date and Time Problems
```sql
-- Template for date-based analysis
SELECT 
    DATE_FORMAT(date_column, '%Y-%m') as period,
    COUNT(*) as count,
    LAG(COUNT(*)) OVER (ORDER BY DATE_FORMAT(date_column, '%Y-%m')) as prev_count,
    -- Growth calculation template
    CASE 
        WHEN LAG(COUNT(*)) OVER (ORDER BY DATE_FORMAT(date_column, '%Y-%m')) IS NULL THEN 'N/A'
        ELSE CONCAT(
            ROUND(
                (COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY DATE_FORMAT(date_column, '%Y-%m'))) * 100.0 / 
                LAG(COUNT(*)) OVER (ORDER BY DATE_FORMAT(date_column, '%Y-%m')), 2
            ), '%'
        )
    END as growth
FROM [table]
GROUP BY DATE_FORMAT(date_column, '%Y-%m');
```

#### Category 3: Duplicate Detection
```sql
-- Template for finding duplicates
WITH duplicate_check AS (
    SELECT 
        [identifying_columns],
        COUNT(*) as occurrence_count,
        GROUP_CONCAT([primary_key]) as related_ids
    FROM [table]
    GROUP BY [identifying_columns]
    HAVING COUNT(*) > 1
)
SELECT * FROM duplicate_check
ORDER BY occurrence_count DESC;
```

### 15.2 Time Management During Interviews
- **2-minute problems**: Basic SELECT, WHERE, simple JOINs
- **5-minute problems**: Aggregation, GROUP BY, window functions
- **10-minute problems**: Complex JOINs, subqueries, CTEs
- **15-minute problems**: Multiple CTEs, recursive queries, optimization

---

## Lesson 16: Final Optimization Masterclass

### 16.1 Query Rewriting for Performance
```sql
-- Before: Inefficient nested subqueries
SELECT 
    e.name,
    (SELECT COUNT(*) FROM sales s WHERE s.emp_id = e.emp_id) as sales_count,
    (SELECT SUM(amount) FROM sales s WHERE s.emp_id = e.emp_id) as total_sales,
    (SELECT AVG(amount) FROM sales s WHERE s.emp_id = e.emp_id) as avg_sale
FROM employees e
WHERE e.dept_id = 1;

-- After: Optimized with single JOIN
SELECT 
    e.name,
    COALESCE(s.sales_count, 0) as sales_count,
    COALESCE(s.total_sales, 0) as total_sales,
    COALESCE(s.avg_sale, 0) as avg_sale
FROM employees e
LEFT JOIN (
    SELECT 
        emp_id,
        COUNT(*) as sales_count,
        SUM(amount) as total_sales,
        AVG(amount) as avg_sale
    FROM sales
    GROUP BY emp_id
) s ON e.emp_id = s.emp_id
WHERE e.dept_id = 1;
```

### 16.2 Memory and Resource Optimization
```sql
-- Efficient pagination for large result sets
SELECT 
    e.emp_id,
    e.name,
    e.salary,
    ROW_NUMBER() OVER (ORDER BY e.salary DESC) as row_num
FROM employees e
WHERE e.salary BETWEEN 50000 AND 100000  -- Filter first to reduce dataset
ORDER BY e.salary DESC
LIMIT 20 OFFSET 40;  -- Page 3 (assuming 20 records per page)

-- Memory-efficient alternative using cursor-based pagination
SELECT 
    e.emp_id,
    e.name,
    e.salary
FROM employees e
WHERE e.salary < 75000  -- Last salary from previous page
ORDER BY e.salary DESC
LIMIT 20;
```

---

## Lesson 17: Advanced Problem Solving Patterns

### 17.1 The "Running Balance" Problem
```sql
-- Calculate running account balance with transaction history
WITH transaction_history AS (
    SELECT 
        account_id,
        transaction_date,
        transaction_type,
        amount,
        -- Convert debits to negative amounts
        CASE 
            WHEN transaction_type = 'DEBIT' THEN -amount
            ELSE amount
        END as signed_amount,
        ROW_NUMBER() OVER (
            PARTITION BY account_id 
            ORDER BY transaction_date, transaction_id
        ) as transaction_sequence
    FROM account_transactions
)
SELECT 
    account_id,
    transaction_date,
    transaction_type,
    amount,
    signed_amount,
    SUM(signed_amount) OVER (
        PARTITION BY account_id 
        ORDER BY transaction_date, transaction_sequence
        ROWS UNBOUNDED PRECEDING
    ) as running_balance,
    -- Flag potential issues
    CASE 
        WHEN SUM(signed_amount) OVER (
            PARTITION BY account_id 
            ORDER BY transaction_date, transaction_sequence
            ROWS UNBOUNDED PRECEDING
        ) < 0 THEN 'OVERDRAFT'
        WHEN SUM(signed_amount) OVER (
            PARTITION BY account_id 
            ORDER BY transaction_date, transaction_sequence
            ROWS UNBOUNDED PRECEDING
        ) > 1000000 THEN 'HIGH_BALANCE'
        ELSE 'NORMAL'
    END as balance_status
FROM transaction_history
ORDER BY account_id, transaction_date, transaction_sequence;
```

### 17.2 The "Cohort Analysis" Pattern
```sql
-- Customer retention cohort analysis
WITH customer_cohorts AS (
    SELECT 
        customer_id,
        DATE_FORMAT(MIN(order_date), '%Y-%m') as cohort_month,
        MIN(order_date) as first_order_date
    FROM orders
    GROUP BY customer_id
),
cohort_periods AS (
    SELECT 
        cc.customer_id,
        cc.cohort_month,
        o.order_date,
        PERIOD_DIFF(
            DATE_FORMAT(o.order_date, '%Y%m'),
            DATE_FORMAT(cc.first_order_date, '%Y%m')
        ) as period_number
    FROM customer_cohorts cc
    INNER JOIN orders o ON cc.customer_id = o.customer_id
),
cohort_table AS (
    SELECT 
        cohort_month,
        period_number,
        COUNT(DISTINCT customer_id) as customers_active
    FROM cohort_periods
    GROUP BY cohort_month, period_number
),
cohort_sizes AS (
    SELECT 
        cohort_month,
        COUNT(DISTINCT customer_id) as cohort_size
    FROM customer_cohorts
    GROUP BY cohort_month
)
SELECT 
    ct.cohort_month,
    cs.cohort_size,
    ct.period_number,
    ct.customers_active,
    ROUND(
        ct.customers_active * 100.0 / cs.cohort_size, 2
    ) as retention_rate,
    -- Retention category
    CASE 
        WHEN ct.period_number = 0 THEN 'New Customers'
        WHEN ROUND(ct.customers_active * 100.0 / cs.cohort_size, 2) >= 80 THEN 'Excellent Retention'
        WHEN ROUND(ct.customers_active * 100.0 / cs.cohort_size, 2) >= 60 THEN 'Good Retention'
        WHEN ROUND(ct.customers_active * 100.0 / cs.cohort_size, 2) >= 40 THEN 'Average Retention'
        WHEN ROUND(ct.customers_active * 100.0 / cs.cohort_size, 2) >= 20 THEN 'Poor Retention'
        ELSE 'Very Poor Retention'
    END as retention_quality
FROM cohort_table ct
INNER JOIN cohort_sizes cs ON ct.cohort_month = cs.cohort_month
WHERE ct.period_number <= 12  -- First 12 months only
ORDER BY ct.cohort_month, ct.period_number;
```

---

## Lesson 18: Final Interview Preparation

### 18.1 Company-Specific Question Patterns

#### Amazon/Flipkart Style Questions
```sql
-- Product recommendation based on purchase patterns
WITH customer_product_affinity AS (
    SELECT 
        o1.customer_id,
        oi1.product_id as purchased_product,
        oi2.product_id as also_purchased_product,
        COUNT(*) as co_purchase_frequency
    FROM orders o1
    INNER JOIN order_items oi1 ON o1.order_id = oi1.order_id
    INNER JOIN orders o2 ON o1.customer_id = o2.customer_id AND o1.order_id != o2.order_id
    INNER JOIN order_items oi2 ON o2.order_id = oi2.order_id
    WHERE oi1.product_id != oi2.product_id
    GROUP BY o1.customer_id, oi1.product_id, oi2.product_id
    HAVING COUNT(*) >= 2  -- At least 2 co-purchases
),
product_recommendations AS (
    SELECT 
        purchased_product,
        also_purchased_product,
        SUM(co_purchase_frequency) as total_co_purchases,
        COUNT(DISTINCT customer_id) as customers_count,
        ROUND(
            SUM(co_purchase_frequency) * 1.0 / COUNT(DISTINCT customer_id), 2
        ) as avg_co_purchase_rate
    FROM customer_product_affinity
    GROUP BY purchased_product, also_purchased_product
)
SELECT 
    p1.product_name as main_product,
    p2.product_name as recommended_product,
    pr.total_co_purchases,
    pr.customers_count,
    pr.avg_co_purchase_rate,
    RANK() OVER (
        PARTITION BY pr.purchased_product 
        ORDER BY pr.avg_co_purchase_rate DESC
    ) as recommendation_rank
FROM product_recommendations pr
INNER JOIN products p1 ON pr.purchased_product = p1.product_id
INNER JOIN products p2 ON pr.also_purchased_product = p2.product_id
ORDER BY pr.purchased_product, recommendation_rank
LIMIT 100;  -- Top recommendations only
```

#### Banking/Finance Style Questions
```sql
-- Fraud detection pattern analysis
WITH transaction_patterns AS (
    SELECT 
        account_id,
        transaction_date,
        amount,
        transaction_type,
        merchant_category,
        -- Time-based features
        HOUR(transaction_timestamp) as transaction_hour,
        DAYOFWEEK(transaction_date) as day_of_week,
        -- Amount-based features
        AVG(amount) OVER (
            PARTITION BY account_id 
            ORDER BY transaction_date 
            ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
        ) as avg_amount_30days,
        STDDEV(amount) OVER (
            PARTITION BY account_id 
            ORDER BY transaction_date 
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) as amount_volatility_7days,
        -- Frequency features
        COUNT(*) OVER (
            PARTITION BY account_id 
            ORDER BY transaction_date 
            RANGE BETWEEN INTERVAL 1 DAY PRECEDING AND CURRENT ROW
        ) as transactions_today,
        COUNT(*) OVER (
            PARTITION BY account_id 
            ORDER BY transaction_date 
            RANGE BETWEEN INTERVAL 7 DAY PRECEDING AND CURRENT ROW
        ) as transactions_7days
    FROM transactions
),
fraud_scoring AS (
    SELECT 
        *,
        -- Anomaly detection scoring
        CASE 
            WHEN amount > avg_amount_30days * 5 THEN 20  -- Unusually high amount
            WHEN amount > avg_amount_30days * 3 THEN 10
            ELSE 0
        END +
        CASE 
            WHEN transaction_hour BETWEEN 0 AND 5 THEN 15  -- Late night transactions
            WHEN transaction_hour BETWEEN 22 AND 23 THEN 5
            ELSE 0
        END +
        CASE 
            WHEN transactions_today > 10 THEN 25  -- Too many transactions in one day
            WHEN transactions_today > 5 THEN 10
            ELSE 0
        END +
        CASE 
            WHEN day_of_week IN (1, 7) AND merchant_category = 'ATM' THEN 10  -- Weekend ATM
            ELSE 0
        END as fraud_risk_score
    FROM transaction_patterns
)
SELECT 
    account_id,
    transaction_date,
    amount,
    transaction_type,
    merchant_category,
    fraud_risk_score,
    CASE 
        WHEN fraud_risk_score >= 40 THEN 'HIGH RISK - Block Transaction'
        WHEN fraud_risk_score >= 25 THEN 'MEDIUM RISK - Manual Review'
        WHEN fraud_risk_score >= 15 THEN 'LOW RISK - Monitor'
        ELSE 'NORMAL'
    END as risk_assessment,
    -- Additional context for review
    CONCAT(
        'Avg 30d: ', ROUND(avg_amount_30days, 2),
        ' | Today: ', transactions_today,
        ' | Hour: ', transaction_hour
    ) as context_info
FROM fraud_scoring
WHERE fraud_risk_score > 0  -- Only show potentially risky transactions
ORDER BY fraud_risk_score DESC, transaction_date DESC;
```

---

## Lesson 19: Phase 6 Assessment & Practice Problems

### Assessment Problem 1: Data Analytics Challenge
**Scenario**: You're analyzing an e-commerce platform's performance

**Question**: Create a comprehensive dashboard query that shows:
- Monthly revenue trends with growth rates
- Top-performing products by category
- Customer segmentation based on purchase behavior
- Seasonal patterns identification

```sql
-- Solution Template:
WITH monthly_trends AS (
    -- Your implementation here
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') as month,
        SUM(total_amount) as revenue,
        COUNT(DISTINCT customer_id) as unique_customers,
        COUNT(*) as total_orders,
        AVG(total_amount) as avg_order_value
    FROM orders
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
),
-- Continue with product analysis, customer segmentation, etc.
```

### Assessment Problem 2: Complex Business Logic
**Scenario**: Employee performance evaluation system

**Question**: Design a query that calculates employee rankings based on:
- Sales performance (40% weight)
- Customer satisfaction scores (30% weight)  
- Attendance (20% weight)
- Team collaboration metrics (10% weight)

Include tie-breaking logic and performance categories.

### Assessment Problem 3: Optimization Challenge
**Question**: Given a slow-performing query, rewrite it for better performance:

```sql
-- Original slow query:
SELECT DISTINCT
    c.customer_name,
    (SELECT COUNT(*) FROM orders o WHERE o.customer_id = c.customer_id) as order_count,
    (SELECT SUM(total_amount) FROM orders o WHERE o.customer_id = c.customer_id) as total_spent
FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id)
ORDER BY total_spent DESC;

-- Your optimized version should go here
```

---

## Final Assessment Checklist

### Technical Skills Verification
- [ ] Can write efficient JOINs for multiple tables
- [ ] Understands window functions and their use cases
- [ ] Can optimize queries for better performance
- [ ] Handles NULL values and edge cases properly
- [ ] Uses CTEs for complex multi-step logic
- [ ] Applies proper indexing strategies
- [ ] Writes clean, readable, maintainable SQL

### Interview Readiness Check
- [ ] Solves ranking problems in under 5 minutes
- [ ] Handles date/time calculations confidently
- [ ] Explains query execution plans
- [ ] Identifies performance bottlenecks
- [ ] Demonstrates business logic implementation
- [ ] Shows data quality awareness
- [ ] Communicates solution approach clearly

---

## Phase 6 Practice Schedule (14 Days)

### Week 11: Deep Dive Practice
**Days 1-3: Optimization Focus**
- 2 hours/day: Query optimization exercises
- 1 hour/day: Performance analysis practice
- Practice problems: HackerRank Advanced Select, LeetCode Hard problems

**Days 4-7: Complex Pattern Mastery**
- 2.5 hours/day: Advanced CTE and window function problems
- 30 minutes/day: Review optimization techniques
- Mock interviews: 2 sessions focusing on complex queries

### Week 12: Interview Simulation
**Days 8-10: Company-Specific Practice**
- TCS/Infosys style: Business logic heavy problems (1.5 hours)
- FAANG style: Algorithm and efficiency focused (1.5 hours)
- Banking/Finance: Data analysis and fraud detection (1 hour)

**Days 11-14: Final Sprint**
- Daily mock interviews (45 minutes each)
- Solve 5-8 hard problems daily
- Review and optimize all previous solutions
- Time-pressure practice (solve problems in interview conditions)

---

## Resource Recommendations for Phase 6

### Essential Practice Platforms
1. **LeetCode**: Focus on Hard SQL problems (262, 601, 1479, 1892, 1917)
2. **HackerRank**: Complete Advanced Join and Alternative Queries domains
3. **GeeksforGeeks**: SQL Interview Questions section
4. **StrataScratch**: Real company interview questions
5. **SQLBolt**: Advanced exercises for optimization

### Mock Interview Resources
1. **Pramp**: Free peer-to-peer SQL interviews
2. **InterviewBit**: SQL interview preparation track
3. **Coding Ninjas**: SQL placement preparation course
4. **YouTube**: "SQL Interview Questions" channels for pattern recognition

### Performance Analysis Tools
1. **MySQL Workbench**: Query execution plan analysis
2. **phpMyAdmin**: Index usage statistics
3. **Online SQL validators**: Query optimization suggestions

---

## Success Metrics for Phase 6

### Week 11 Targets
- Solve 15+ advanced optimization problems
- Complete 3 company-style complex queries
- Achieve 90%+ accuracy on timed practice tests
- Explain 5 different optimization techniques confidently

### Week 12 Targets  
- Solve any LeetCode Hard SQL problem in under 15 minutes
- Pass 5+ mock interview sessions with positive feedback
- Demonstrate 3 different approaches for common problem patterns
- Show proficiency in query plan analysis and optimization

### Final Assessment Criteria
- **Speed**: Solve medium problems in 5-7 minutes, hard in 10-15 minutes
- **Accuracy**: 95%+ on first submission
- **Code Quality**: Clean, optimized, production-ready queries
- **Problem-Solving**: Multiple solution approaches, optimization awareness
- **Communication**: Clear explanation of approach and trade-offs

---

## Conclusion: You're Placement-Ready!

After completing Phase 6, you will have mastered:
- **Advanced SQL patterns** used in real-world scenarios
- **Query optimization** for production environments  
- **Complex problem-solving** with multiple solution approaches
- **Interview-specific techniques** for top tech companies
- **Performance analysis** and bottleneck identification
- **Business logic implementation** in SQL
- **Data quality and edge case handling**

### Final Tips for Interview Success

1. **Practice Under Pressure**: Always time yourself during practice
2. **Think Aloud**: Explain your approach during interviews
3. **Start Simple**: Begin with a basic solution, then optimize
4. **Handle Edge Cases**: Always consider NULL values and empty datasets
5. **Test Your Logic**: Mentally verify your query with sample data
6. **Stay Calm**: Break complex problems into smaller, manageable parts

### Post-Phase 6 Maintenance
- Solve 2-3 problems weekly to maintain sharpness
- Review optimization techniques monthly
- Stay updated with new SQL features and best practices
- Participate in coding communities for continuous learning

**Remember**: SQL mastery is about pattern recognition and systematic problem-solving. With Phase 6 complete, you have both the technical skills and interview confidence needed for top-tier placements!

---

## Quick Reference: Interview Day Checklist

### Before the Interview
- [ ] Review common SQL functions and their syntax
- [ ] Practice 2-3 warm-up problems  
- [ ] Prepare questions about the company's data infrastructure
- [ ] Review your Phase 6 notes on optimization patterns

### During the Interview
- [ ] Read the problem statement twice
- [ ] Ask clarifying questions about data assumptions
- [ ] Start with a simple approach, then optimize
- [ ] Explain your thought process clearly
- [ ] Test your solution with edge cases
- [ ] Discuss alternative approaches if time permits

### After the Interview
- [ ] Note down any new patterns you encountered
- [ ] Practice similar problems for next interviews
- [ ] Update your optimization knowledge based on feedback

**Good luck with your placements! You're well-prepared to tackle any SQL challenge.**