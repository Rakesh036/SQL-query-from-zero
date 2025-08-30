# CTE (Common Table Expression) - Complete Study Notes

## What Are CTEs?

A Common Table Expression (CTE) is a named temporary result set that exists only for the duration of a single SQL statement. Think of it as creating a "virtual table" that you can reference in your main query.

**Key Concept**: CTEs make complex queries readable by breaking them into logical, named steps.

### Basic Syntax
```sql
WITH cte_name AS (
    -- CTE query definition
    SELECT column1, column2, ...
    FROM table_name
    WHERE conditions
)
-- Main query using the CTE
SELECT * 
FROM cte_name
WHERE more_conditions;
```

### CTEs vs Other Approaches

**CTE vs Subquery:**
```sql
-- Using Subquery (harder to read)
SELECT *
FROM (
    SELECT name, department, salary,
           ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as rn
    FROM employees
) ranked
WHERE rn <= 3;

-- Using CTE (more readable)
WITH ranked_employees AS (
    SELECT name, department, salary,
           ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as rn
    FROM employees
)
SELECT *
FROM ranked_employees
WHERE rn <= 3;
```

**CTE vs Temporary Table:**
- **CTE**: Exists only during query execution, no storage overhead
- **Temporary Table**: Physical storage, persists during session, requires cleanup

**CTE vs View:**
- **CTE**: Query-scoped, defined inline
- **View**: Database object, reusable across queries

---

## Types of CTEs

### 1. Simple (Non-Recursive) CTEs

Used for breaking complex queries into readable parts.

```sql
WITH sales_summary AS (
    SELECT 
        department,
        SUM(sales_amount) as total_sales,
        AVG(sales_amount) as avg_sales,
        COUNT(*) as num_sales
    FROM sales
    WHERE sale_date >= '2023-01-01'
    GROUP BY department
)
SELECT 
    department,
    total_sales,
    ROUND(total_sales * 100.0 / SUM(total_sales) OVER (), 2) as pct_of_total
FROM sales_summary
WHERE total_sales > 10000
ORDER BY total_sales DESC;
```

### 2. Multiple CTEs

You can define multiple CTEs in a single query.

```sql
WITH 
high_performers AS (
    SELECT employee_id, name, department, salary
    FROM employees
    WHERE salary > 60000
),
dept_stats AS (
    SELECT 
        department,
        COUNT(*) as total_employees,
        AVG(salary) as avg_salary
    FROM employees
    GROUP BY department
)
SELECT 
    hp.name,
    hp.department,
    hp.salary,
    ds.avg_salary,
    hp.salary - ds.avg_salary as above_avg
FROM high_performers hp
JOIN dept_stats ds ON hp.department = ds.department
ORDER BY above_avg DESC;
```

### 3. Recursive CTEs

Used for hierarchical data like organizational charts, family trees, or graph traversal.

**Basic Structure:**
```sql
WITH RECURSIVE cte_name AS (
    -- Anchor member (base case)
    SELECT columns FROM table WHERE base_condition
    
    UNION ALL
    
    -- Recursive member (iterative case)
    SELECT columns FROM table 
    JOIN cte_name ON recursive_condition
)
SELECT * FROM cte_name;
```

**Example: Employee Hierarchy**
```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor: Find all top-level managers (no boss)
    SELECT employee_id, name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Find employees reporting to current level
    SELECT e.employee_id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT 
    employee_id,
    name,
    level,
    REPEAT('  ', level - 1) || name as hierarchy_display
FROM employee_hierarchy
ORDER BY level, name;
```

---

## Complete Dry Run Example

### Sample Data Setup
```sql
-- employees table
| employee_id | name    | department | salary | manager_id |
|-------------|---------|------------|--------|------------|
| 1           | Alice   | Sales      | 50000  | 3          |
| 2           | Bob     | Sales      | 60000  | 3          |
| 3           | Carol   | Sales      | 80000  | NULL       |
| 4           | David   | IT         | 55000  | 5          |
| 5           | Eve     | IT         | 75000  | NULL       |
| 6           | Frank   | IT         | 65000  | 5          |

-- sales table  
| sale_id | employee_id | sale_date  | amount |
|---------|-------------|------------|--------|
| 1       | 1           | 2023-01-15 | 1000   |
| 2       | 1           | 2023-02-20 | 1500   |
| 3       | 2           | 2023-01-10 | 2000   |
| 4       | 2           | 2023-03-05 | 1800   |
| 5       | 4           | 2023-01-25 | 800    |
```

### Query to Analyze:
```sql
WITH 
employee_performance AS (
    SELECT 
        e.employee_id,
        e.name,
        e.department,
        e.salary,
        COALESCE(SUM(s.amount), 0) as total_sales,
        COUNT(s.sale_id) as num_sales
    FROM employees e
    LEFT JOIN sales s ON e.employee_id = s.employee_id
    GROUP BY e.employee_id, e.name, e.department, e.salary
),
dept_rankings AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY total_sales DESC) as sales_rank,
        AVG(total_sales) OVER (PARTITION BY department) as dept_avg_sales
    FROM employee_performance
)
SELECT 
    name,
    department,
    salary,
    total_sales,
    sales_rank,
    CASE 
        WHEN total_sales > dept_avg_sales THEN 'Above Average'
        ELSE 'Below Average'
    END as performance_category
FROM dept_rankings
WHERE sales_rank <= 2
ORDER BY department, sales_rank;
```

## Step-by-Step Execution Dry Run

### Step 1: Execute First CTE - employee_performance

**Process:**
1. Join employees with sales (LEFT JOIN to include all employees)
2. Group by employee attributes
3. Calculate total_sales and num_sales per employee

**Result of employee_performance CTE:**
```
| employee_id | name  | department | salary | total_sales | num_sales |
|-------------|-------|------------|--------|-------------|-----------|
| 1           | Alice | Sales      | 50000  | 2500        | 2         |
| 2           | Bob   | Sales      | 60000  | 3800        | 2         |
| 3           | Carol | Sales      | 80000  | 0           | 0         |
| 4           | David | IT         | 55000  | 800         | 1         |
| 5           | Eve   | IT         | 75000  | 0           | 0         |
| 6           | Frank | IT         | 65000  | 0           | 0         |
```

### Step 2: Execute Second CTE - dept_rankings

**Process:**
1. Take employee_performance result as input
2. Add ROW_NUMBER() partitioned by department, ordered by total_sales DESC
3. Calculate dept_avg_sales for each department

**Calculations:**
- **Sales dept average:** (2500 + 3800 + 0) / 3 = 2100
- **IT dept average:** (800 + 0 + 0) / 3 = 266.67

**Sales Department Ranking (by total_sales DESC):**
- Bob (3800) gets rank 1
- Alice (2500) gets rank 2  
- Carol (0) gets rank 3

**IT Department Ranking (by total_sales DESC):**
- David (800) gets rank 1
- Eve (0) gets rank 2 (first by some tiebreaker)
- Frank (0) gets rank 3

**Result of dept_rankings CTE:**
```
| employee_id | name  | dept  | salary | total_sales | num_sales | sales_rank | dept_avg_sales |
|-------------|-------|-------|--------|-------------|-----------|------------|----------------|
| 1           | Alice | Sales | 50000  | 2500        | 2         | 2          | 2100           |
| 2           | Bob   | Sales | 60000  | 3800        | 2         | 1          | 2100           |
| 3           | Carol | Sales | 80000  | 0           | 0         | 3          | 2100           |
| 4           | David | IT    | 55000  | 800         | 1         | 1          | 266.67         |
| 5           | Eve   | IT    | 75000  | 0           | 0         | 2          | 266.67         |
| 6           | Frank | IT    | 65000  | 0           | 0         | 3          | 266.67         |
```

### Step 3: Main Query Execution

**Process:**
1. Filter WHERE sales_rank <= 2
2. Add CASE statement for performance_category
3. Apply ORDER BY department, sales_rank

**Performance Category Logic:**
- Alice: 2500 > 2100 → 'Above Average'
- Bob: 3800 > 2100 → 'Above Average'
- David: 800 > 266.67 → 'Above Average'
- Eve: 0 < 266.67 → 'Below Average'

**Final Result:**
```
| name  | department | salary | total_sales | sales_rank | performance_category |
|-------|------------|--------|-------------|------------|---------------------|
| David | IT         | 55000  | 800         | 1          | Above Average       |
| Eve   | IT         | 75000  | 0           | 2          | Below Average       |
| Bob   | Sales      | 60000  | 3800        | 1          | Above Average       |
| Alice | Sales      | 50000  | 2500        | 2          | Above Average       |
```

## The Mental Journey

**What MySQL is "thinking":**

1. **"Let me build the first temporary table (employee_performance)"**
   - "I'll join employees with sales, group by employee, and calculate totals"
   - "Alice sold 1000+1500=2500, Bob sold 2000+1800=3800, others have no sales"

2. **"Now I'll build the second temporary table (dept_rankings)"**  
   - "Using the first table, I'll rank people within departments by sales performance"
   - "In Sales: Bob is #1 (3800), Alice is #2 (2500), Carol is #3 (0)"
   - "In IT: David is #1 (800), Eve and Frank tied at #2 and #3 (0 each)"

3. **"Finally, I'll execute the main query"**
   - "Filter to top 2 in each department"
   - "Compare each person's sales to their department average"
   - "Sort by department, then rank"

---

## Recursive CTE Dry Run

### Sample Hierarchical Data:
```sql
-- employees table for hierarchy
| employee_id | name    | manager_id |
|-------------|---------|------------|
| 1           | CEO     | NULL       |
| 2           | VP_Sales| 1          |
| 3           | VP_IT   | 1          |
| 4           | Manager1| 2          |
| 5           | Manager2| 2          |
| 6           | Dev1    | 3          |
| 7           | Sales1  | 4          |
| 8           | Sales2  | 4          |
```

### Recursive Query:
```sql
WITH RECURSIVE org_chart AS (
    -- Anchor: Top level (CEO)
    SELECT employee_id, name, manager_id, 1 as level, name as path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Next level down
    SELECT 
        e.employee_id, 
        e.name, 
        e.manager_id, 
        oc.level + 1,
        oc.path || ' -> ' || e.name
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.employee_id
)
SELECT employee_id, name, level, path
FROM org_chart
ORDER BY level, employee_id;
```

### Recursive Execution Steps:

**Iteration 1 (Anchor):**
```
| employee_id | name | manager_id | level | path |
|-------------|------|------------|-------|------|
| 1           | CEO  | NULL       | 1     | CEO  |
```

**Iteration 2 (First Recursive):**
Join employees with current org_chart (just CEO row):
```
| employee_id | name     | manager_id | level | path         |
|-------------|----------|------------|-------|--------------|
| 1           | CEO      | NULL       | 1     | CEO          |
| 2           | VP_Sales | 1          | 2     | CEO -> VP_Sales |
| 3           | VP_IT    | 1          | 2     | CEO -> VP_IT    |
```

**Iteration 3 (Second Recursive):**
Join employees with VP_Sales and VP_IT rows:
```
| employee_id | name     | manager_id | level | path                    |
|-------------|----------|------------|-------|-------------------------|
| 1           | CEO      | NULL       | 1     | CEO                     |
| 2           | VP_Sales | 1          | 2     | CEO -> VP_Sales         |
| 3           | VP_IT    | 1          | 2     | CEO -> VP_IT            |
| 4           | Manager1 | 2          | 3     | CEO -> VP_Sales -> Manager1 |
| 5           | Manager2 | 2          | 3     | CEO -> VP_Sales -> Manager2 |
| 6           | Dev1     | 3          | 3     | CEO -> VP_IT -> Dev1        |
```

**Iteration 4 (Third Recursive):**
Join employees with Manager1, Manager2, Dev1 rows:
```
Final result includes Sales1 and Sales2 at level 4
```

**Process stops when no more matches are found.**

---

## Practical Use Cases

### 1. Data Analysis Pipeline
```sql
WITH 
raw_data AS (
    SELECT * FROM sales WHERE sale_date >= '2023-01-01'
),
cleaned_data AS (
    SELECT 
        customer_id,
        product_id,
        sale_date,
        amount,
        CASE WHEN amount < 0 THEN 0 ELSE amount END as clean_amount
    FROM raw_data
    WHERE amount IS NOT NULL
),
aggregated_data AS (
    SELECT 
        customer_id,
        COUNT(*) as num_purchases,
        SUM(clean_amount) as total_spent,
        AVG(clean_amount) as avg_purchase
    FROM cleaned_data
    GROUP BY customer_id
)
SELECT * FROM aggregated_data WHERE total_spent > 1000;
```

### 2. Ranking and Filtering
```sql
WITH ranked_products AS (
    SELECT 
        product_name,
        category,
        sales_amount,
        DENSE_RANK() OVER (PARTITION BY category ORDER BY sales_amount DESC) as rank
    FROM product_sales
)
SELECT * FROM ranked_products WHERE rank <= 3;
```

### 3. Hierarchical Queries
```sql
-- Find all subordinates of a manager
WITH RECURSIVE subordinates AS (
    SELECT employee_id, name, manager_id, 1 as level
    FROM employees
    WHERE employee_id = 2  -- Starting manager
    
    UNION ALL
    
    SELECT e.employee_id, e.name, e.manager_id, s.level + 1
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.employee_id
)
SELECT * FROM subordinates WHERE level > 1;
```

### 4. Complex Reporting
```sql
WITH 
monthly_sales AS (
    SELECT 
        DATE_FORMAT(sale_date, '%Y-%m') as month,
        SUM(amount) as monthly_total
    FROM sales
    GROUP BY DATE_FORMAT(sale_date, '%Y-%m')
),
sales_with_growth AS (
    SELECT 
        month,
        monthly_total,
        LAG(monthly_total) OVER (ORDER BY month) as prev_month,
        monthly_total - LAG(monthly_total) OVER (ORDER BY month) as growth
    FROM monthly_sales
)
SELECT 
    month,
    monthly_total,
    prev_month,
    growth,
    CASE 
        WHEN growth > 0 THEN 'Growing'
        WHEN growth < 0 THEN 'Declining'
        ELSE 'Stable'
    END as trend
FROM sales_with_growth;
```

---

## CTE Best Practices

### Readability Guidelines
1. **Use descriptive names:** `high_value_customers` not `cte1`
2. **One logical step per CTE:** Don't cram multiple transformations
3. **Comment complex logic:** Explain business rules
4. **Consistent formatting:** Align CTE definitions

### Performance Considerations
1. **CTEs are not materialized** - they're executed for each reference
2. **Multiple references = multiple executions:**
```sql
-- This executes expensive_cte twice
WITH expensive_cte AS (SELECT * FROM big_table WHERE complex_condition)
SELECT a.col FROM expensive_cte a
JOIN expensive_cte b ON a.id = b.parent_id;  -- CTE executed again!
```

3. **Consider temporary tables for complex CTEs used multiple times**
4. **MySQL optimizes simple CTEs well** - don't avoid them for performance
5. **Recursive CTEs have limits** - default max recursion depth is 1000

### Common Pitfalls

**1. Scope Confusion:**
```sql
-- WRONG - CTE not available here
SELECT * FROM my_table;
WITH my_cte AS (SELECT * FROM my_table)
SELECT * FROM my_cte;

-- CORRECT - CTE before main query
WITH my_cte AS (SELECT * FROM my_table)
SELECT * FROM my_cte;
```

**2. Forward References:**
```sql
-- WRONG - cte2 references cte3 before it's defined
WITH 
cte2 AS (SELECT * FROM cte3),  -- Error!
cte3 AS (SELECT * FROM table1)
SELECT * FROM cte2;

-- CORRECT - Define in dependency order
WITH 
cte3 AS (SELECT * FROM table1),
cte2 AS (SELECT * FROM cte3)
SELECT * FROM cte2;
```

**3. Recursive Infinite Loops:**
```sql
-- Dangerous - could loop infinitely
WITH RECURSIVE bad_recursion AS (
    SELECT 1 as n
    UNION ALL
    SELECT n + 1 FROM bad_recursion  -- No termination condition!
)
SELECT * FROM bad_recursion;

-- SAFE - Add termination condition
WITH RECURSIVE safe_recursion AS (
    SELECT 1 as n
    UNION ALL
    SELECT n + 1 FROM safe_recursion WHERE n < 10
)
SELECT * FROM safe_recursion;
```

---

## CTEs vs Alternatives Decision Matrix

| Use CTE When: | Use Subquery When: | Use Temp Table When: | Use View When: |
|---------------|-------------------|----------------------|----------------|
| Query is complex and needs breaking down | Simple, one-time logic | Multiple complex references | Logic reused across queries |
| Need multiple related calculations | Single calculation needed | Need indexing on intermediate results | Want to encapsulate business logic |
| Hierarchical/recursive data | Non-recursive scenarios | Performance critical | Standard reporting needs |
| Code readability is priority | Query is simple enough inline | Need to persist between statements | Multiple users need same logic |

---

## Memory Aids and Mental Models

### The "Recipe" Analogy
Think of CTEs like cooking steps:
```sql
WITH 
prep_ingredients AS (SELECT chopped, cleaned FROM raw_materials),
mixed_batter AS (SELECT * FROM prep_ingredients WHERE ready),
baked_cake AS (SELECT *, 'cooked' as status FROM mixed_batter)
SELECT * FROM baked_cake;  -- Final dish
```

Each CTE is a preparation step, and the main query is serving the final dish.

### The "Building Blocks" Model
- **Simple CTEs**: Stack building blocks (each CTE builds on previous ones)
- **Recursive CTEs**: Self-replicating blocks (each iteration adds more of the same pattern)

### Recursive CTE Mental Model
Think of recursive CTEs as:
1. **Plant a seed** (anchor member)
2. **Grow branches** (recursive member) 
3. **Stop when no more growth** (termination condition)

### Key Questions Before Using CTEs
1. **Is this query complex enough to benefit from breaking down?**
2. **Will the CTE be referenced multiple times?** (Consider temp table)
3. **Am I dealing with hierarchical data?** (Recursive CTE candidate)
4. **Could this logic be reused elsewhere?** (Consider a view instead)
5. **Is performance critical?** (Profile CTE vs alternatives)

CTEs are powerful tools for creating readable, maintainable SQL. Use them to break complex problems into logical steps, but be mindful of performance implications and choose the right tool for each scenario.

---

## Critical Concepts to Keep in Mind

### The CTE Lifecycle Reality

CTEs exist only during query execution - they're not stored anywhere:
- **Creation**: Defined when query starts
- **Execution**: Calculated when referenced
- **Destruction**: Gone when query completes
- **Multiple References**: Re-executed each time (potential performance cost)

### Execution Order and Dependency Chain

CTEs must be defined in dependency order:
```sql
-- CORRECT dependency chain
WITH 
step1 AS (SELECT * FROM base_table),
step2 AS (SELECT * FROM step1 WHERE condition),  -- Uses step1
step3 AS (SELECT * FROM step2 JOIN other_table)  -- Uses step2
SELECT * FROM step3;  -- Main query uses final step
```

**Mental Model**: Think of CTEs as a recipe - you can't use flour mixture before you've mixed the flour.

### Performance Reality Check

**CTE Re-execution Trap:**
```sql
-- PERFORMANCE WARNING - expensive_calc runs TWICE
WITH expensive_calc AS (
    SELECT id, complex_function(big_column) as result
    FROM huge_table
)
SELECT a.id, a.result, b.result
FROM expensive_calc a
JOIN expensive_calc b ON a.id = b.parent_id;  -- Re-executed here!
```

**Solution**: Use temporary table for multiple references to expensive CTEs.

**Optimization Guidelines:**
- Simple CTEs: MySQL optimizes well, use freely
- Complex CTEs referenced once: Perfect use case
- Complex CTEs referenced multiple times: Consider temp tables
- Recursive CTEs: Watch recursion depth and termination conditions

### Recursive CTE Mental Traps

**The Infinite Loop Danger:**
Always ask: "What stops this recursion?"
```sql
-- DANGEROUS - No termination condition
WITH RECURSIVE endless AS (
    SELECT 1 as n, 'start' as path
    UNION ALL
    SELECT n + 1, path || '->' || (n+1) FROM endless  -- Never stops!
)
-- This will hit recursion limit or run forever

-- SAFE - Clear termination
WITH RECURSIVE controlled AS (
    SELECT 1 as n, 'start' as path
    UNION ALL
    SELECT n + 1, path || '->' || (n+1) FROM controlled 
    WHERE n < 10  -- Explicit stop condition
)
```

**Recursive Debugging Strategy:**
1. Start with just the anchor query
2. Add one iteration manually to verify logic
3. Add recursion with very low limit (WHERE level < 3)
4. Gradually increase limit once logic is verified

### Scope and Naming Gotchas

**CTE Scope Rules:**
- CTEs only exist within the WITH statement they're defined in
- Can't reference CTEs from outer queries
- Can't forward-reference (use CTE before it's defined)

**Naming Conflicts:**
```sql
-- CONFUSING - shadows table name
WITH employees AS (SELECT * FROM employees WHERE active = 1)
SELECT * FROM employees;  -- Which employees? CTE or table?

-- CLEARER - distinctive names
WITH active_employees AS (SELECT * FROM employees WHERE active = 1)
SELECT * FROM active_employees;
```

### Common Anti-Patterns

**1. Over-Engineering Simple Queries:**
```sql
-- OVERKILL for simple case
WITH simple_filter AS (
    SELECT * FROM users WHERE age > 18
)
SELECT name FROM simple_filter;

-- BETTER - direct approach
SELECT name FROM users WHERE age > 18;
```

**2. Chain Dependencies Unnecessarily:**
```sql
-- OVERCOMPLICATED
WITH 
step1 AS (SELECT * FROM sales),
step2 AS (SELECT * FROM step1),  -- Adds no value
step3 AS (SELECT * FROM step2 WHERE amount > 100)
SELECT * FROM step3;

-- SIMPLER
WITH filtered_sales AS (
    SELECT * FROM sales WHERE amount > 100
)
SELECT * FROM filtered_sales;
```

**3. Mixing Concerns in Single CTE:**
```sql
-- CONFUSING - does filtering AND aggregation
WITH messy_cte AS (
    SELECT 
        department,
        COUNT(*) as emp_count,
        AVG(salary) as avg_sal
    FROM employees 
    WHERE hire_date > '2020-01-01'  -- Filtering
    AND salary > 50000              -- More filtering
    GROUP BY department             -- Aggregation
    HAVING COUNT(*) > 5             -- More filtering
)

-- CLEARER - separate concerns
WITH 
filtered_employees AS (
    SELECT * FROM employees 
    WHERE hire_date > '2020-01-01' AND salary > 50000
),
dept_stats AS (
    SELECT 
        department,
        COUNT(*) as emp_count,
        AVG(salary) as avg_sal
    FROM filtered_employees
    GROUP BY department
    HAVING COUNT(*) > 5
)
```

### Debugging Strategies

**Progressive Building:**
```sql
-- Step 1: Test each CTE individually
WITH my_cte AS (SELECT * FROM table WHERE condition)
SELECT * FROM my_cte;  -- Verify this works

-- Step 2: Add next CTE
WITH 
my_cte AS (...),
next_cte AS (SELECT * FROM my_cte WHERE other_condition)
SELECT * FROM next_cte;  -- Verify chain works

-- Step 3: Build complete query
```

**Data Validation Checks:**
```sql
WITH 
my_data AS (SELECT ...),
data_check AS (
    SELECT 
        COUNT(*) as total_rows,
        COUNT(DISTINCT id) as unique_ids,
        MIN(date_col) as min_date,
        MAX(date_col) as max_date
    FROM my_data
)
SELECT * FROM data_check;  -- Sanity check before main logic
```

### Memory Aids

**CTE vs Subquery Decision:**
- **CTE**: "I need to reference this logic multiple times or it's complex"
- **Subquery**: "This is simple, one-time logic"

**Recursive CTE Pattern:**
Remember the "Tree Growth" model:
1. **Seed** (anchor): WHERE parent_id IS NULL
2. **Growth** (recursive): JOIN on parent-child relationship  
3. **Stop** (termination): WHERE level < max_depth

**Performance Heuristic:**
- **Single reference**: CTE is fine
- **Multiple references + simple logic**: CTE is fine
- **Multiple references + complex logic**: Consider temp table
- **Reused across queries**: Create a view

### Reality Check Questions

Before writing any CTE, ask:
1. **Does this actually improve readability?** (Don't CTE for the sake of CTEs)
2. **Am I breaking down logic logically?** (Each CTE should have a clear purpose)
3. **Will this CTE be referenced multiple times?** (Performance consideration)
4. **Could this be solved more simply?** (Sometimes a subquery is clearer)
5. **For recursive: What stops the recursion?** (Infinite loop prevention)

### The Goldilocks Principle

**Too Little Structure:**
```sql
-- Hard to read, everything in one query
SELECT ... FROM (SELECT ... FROM (SELECT ... FROM table) ...) ...
```

**Too Much Structure:**
```sql
-- Over-engineered for simple case
WITH step1 AS (...), step2 AS (...), step3 AS (...) -- Each adds minimal value
```

**Just Right:**
```sql
-- Each CTE serves a clear, logical purpose
WITH 
clean_data AS (-- Data cleansing step),
enriched_data AS (-- Add calculated fields),
final_report AS (-- Business logic aggregation)
```

### Final Wisdom

CTEs are a readability tool first, optimization tool second. They should make your query easier to understand, debug, and maintain. If a CTE doesn't clearly improve the code, it's probably unnecessary.

The best CTE usage follows the principle: "Each CTE should represent a distinct logical step that a human would naturally think through when solving the problem manually."