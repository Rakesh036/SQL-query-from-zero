# SQL Mastery Roadmap for Campus Placements

## Phase 1: Foundation (Week 1-2)
**Duration:** 10-14 days  
**Goal:** Understand database basics and simple queries

### What You'll Learn:
- Database concepts (tables, rows, columns)
- Basic SQL syntax
- SELECT statements
- WHERE clause
- ORDER BY and LIMIT
- Basic data types

### Why This Phase:
This builds your fundamental understanding. Without these basics, you can't progress to complex queries. Most placement tests start with these concepts.

### Key Topics:
1. **Database Structure**
   - Understanding tables and relationships
   - Primary keys concept
   
2. **SELECT Statements**
   ```sql
   SELECT column1, column2 FROM table_name;
   SELECT * FROM employees;
   ```

3. **Filtering Data**
   ```sql
   SELECT * FROM employees WHERE salary > 50000;
   SELECT name FROM students WHERE age BETWEEN 18 AND 25;
   ```

4. **Sorting Results**
   ```sql
   SELECT * FROM products ORDER BY price DESC;
   SELECT * FROM employees ORDER BY name ASC LIMIT 10;
   ```

### Practice Platforms:
- **HackerRank:** Basic Select challenges
- **LeetCode:** Easy SQL problems (595, 584, 183)
- **GeeksforGeeks:** SQL basics section

---

## Phase 2: Data Filtering & Functions (Week 3-4)
**Duration:** 10-14 days  
**Goal:** Master conditional logic and built-in functions

### What You'll Learn:
- Advanced WHERE conditions
- String functions
- Date/time functions
- Mathematical functions
- NULL handling
- Pattern matching

### Why This Phase:
Placement tests heavily focus on data manipulation and cleaning. These skills show your ability to work with real-world messy data.

### Key Topics:
1. **Advanced Filtering**
   ```sql
   SELECT * FROM orders WHERE order_date >= '2023-01-01' 
   AND status IN ('completed', 'shipped');
   ```

2. **String Functions**
   ```sql
   SELECT UPPER(name), LENGTH(email) FROM users;
   SELECT * FROM products WHERE name LIKE '%phone%';
   ```

3. **Date Functions**
   ```sql
   SELECT YEAR(hire_date), MONTH(hire_date) FROM employees;
   SELECT * FROM orders WHERE DATEDIFF(NOW(), order_date) <= 30;
   ```

4. **NULL Handling**
   ```sql
   SELECT COALESCE(phone, 'N/A') FROM contacts;
   SELECT * FROM users WHERE email IS NOT NULL;
   ```

### Practice Focus:
- **HackerRank:** Advanced Select, Basic Aggregation
- **LeetCode:** Problems 196, 175, 181
- **GeeksforGeeks:** SQL functions practice

---

## Phase 3: Aggregation & Grouping (Week 5-6)
**Duration:** 10-14 days  
**Goal:** Master data summarization and analytical queries

### What You'll Learn:
- Aggregate functions (COUNT, SUM, AVG, MAX, MIN)
- GROUP BY clause
- HAVING clause
- Multiple grouping levels

### Why This Phase:
This is where SQL becomes powerful for data analysis. 70% of placement SQL questions involve some form of aggregation. Companies want to see if you can derive insights from data.

### Key Topics:
1. **Basic Aggregation**
   ```sql
   SELECT COUNT(*) FROM employees;
   SELECT AVG(salary), MAX(salary) FROM employees;
   ```

2. **GROUP BY**
   ```sql
   SELECT department, COUNT(*) as emp_count 
   FROM employees GROUP BY department;
   ```

3. **HAVING Clause**
   ```sql
   SELECT department, AVG(salary) 
   FROM employees 
   GROUP BY department 
   HAVING AVG(salary) > 60000;
   ```

4. **Multiple Grouping**
   ```sql
   SELECT department, job_title, COUNT(*) 
   FROM employees 
   GROUP BY department, job_title;
   ```

### Practice Focus:
- **HackerRank:** Aggregation section (all problems)
- **LeetCode:** Problems 182, 197, 511
- **GeeksforGeeks:** GROUP BY and HAVING exercises

---

## Phase 4: Joins & Relationships (Week 7-8)
**Duration:** 14 days  
**Goal:** Master table relationships and complex data retrieval

### What You'll Learn:
- INNER JOIN
- LEFT/RIGHT JOIN
- FULL OUTER JOIN
- SELF JOIN
- Multiple table joins

### Why This Phase:
This is the make-or-break phase for placements. Join questions are extremely common and test your understanding of relational databases. Most complex real-world queries involve joins.

### Key Topics:
1. **INNER JOIN**
   ```sql
   SELECT e.name, d.department_name 
   FROM employees e 
   INNER JOIN departments d ON e.dept_id = d.id;
   ```

2. **LEFT JOIN**
   ```sql
   SELECT c.name, o.order_date 
   FROM customers c 
   LEFT JOIN orders o ON c.id = o.customer_id;
   ```

3. **SELF JOIN**
   ```sql
   SELECT e1.name as employee, e2.name as manager
   FROM employees e1
   JOIN employees e2 ON e1.manager_id = e2.id;
   ```

4. **Multiple Joins**
   ```sql
   SELECT c.name, o.order_date, p.product_name
   FROM customers c
   JOIN orders o ON c.id = o.customer_id
   JOIN order_items oi ON o.id = oi.order_id
   JOIN products p ON oi.product_id = p.id;
   ```

### Practice Focus:
- **HackerRank:** Basic Join, Advanced Join
- **LeetCode:** Problems 570, 577, 580, 262
- **GeeksforGeeks:** SQL Joins comprehensive practice

---

## Phase 5: Subqueries & Advanced Concepts (Week 9-10)
**Duration:** 14 days  
**Goal:** Handle complex nested queries and advanced SQL features

### What You'll Learn:
- Correlated and non-correlated subqueries
- EXISTS and NOT EXISTS
- Window functions (ROW_NUMBER, RANK, DENSE_RANK)
- Common Table Expressions (CTEs)

### Why This Phase:
This separates good candidates from great ones. Advanced concepts show deep SQL understanding and are often used in senior-level questions during placements.

### Key Topics:
1. **Subqueries**
   ```sql
   SELECT name FROM employees 
   WHERE salary > (SELECT AVG(salary) FROM employees);
   ```

2. **Correlated Subqueries**
   ```sql
   SELECT name, salary FROM employees e1
   WHERE salary > (SELECT AVG(salary) FROM employees e2 
                   WHERE e2.department = e1.department);
   ```

3. **Window Functions**
   ```sql
   SELECT name, salary, 
   RANK() OVER (ORDER BY salary DESC) as salary_rank
   FROM employees;
   ```

4. **Common Table Expressions**
   ```sql
   WITH dept_avg AS (
     SELECT department, AVG(salary) as avg_sal
     FROM employees GROUP BY department
   )
   SELECT * FROM dept_avg WHERE avg_sal > 70000;
   ```

### Practice Focus:
- **HackerRank:** Alternative Queries, Advanced Select
- **LeetCode:** Medium/Hard problems (180, 184, 185)
- **GeeksforGeeks:** Advanced SQL queries

---

## Phase 6: Optimization & Interview Prep (Week 11-12)
**Duration:** 14 days  
**Goal:** Performance optimization and interview-specific preparation

### What You'll Learn:
- Query optimization techniques
- Index understanding
- Execution plans
- Complex real-world scenarios
- Time/space complexity in SQL

### Why This Phase:
Placements often include optimization questions. Understanding performance shows you can write production-ready code, not just functional queries.

### Key Topics:
1. **Query Optimization**
   - Using indexes effectively
   - Avoiding unnecessary JOINs
   - Proper WHERE clause placement

2. **Interview Patterns**
   - Ranking problems
   - Running totals
   - Date-based analytics
   - Finding duplicates/gaps

3. **Complex Scenarios**
   ```sql
   -- Find employees earning more than their department average
   SELECT e.name, e.salary, d.dept_name
   FROM employees e
   JOIN departments d ON e.dept_id = d.id
   WHERE e.salary > (
     SELECT AVG(salary) 
     FROM employees e2 
     WHERE e2.dept_id = e.dept_id
   );
   ```

### Practice Focus:
- **HackerRank:** All remaining challenges
- **LeetCode:** Hard problems (262, 601, 1479)
- **GeeksforGeeks:** Interview preparation section

---

## Daily Study Schedule Recommendation

### Week 1-6 (Phases 1-3):
- **Theory:** 1 hour/day
- **Practice:** 1.5-2 hours/day
- **Platform rotation:** Alternate between HackerRank and LeetCode

### Week 7-12 (Phases 4-6):
- **Theory:** 30 minutes/day
- **Practice:** 2-2.5 hours/day
- **Mock interviews:** 2-3 sessions/week

## Platform-Specific Strategy

### HackerRank:
- Complete domains in order
- Focus on SQL badge completion
- Great for structured learning

### LeetCode:
- Sort by acceptance rate (start high)
- Focus on company-specific questions
- Practice under time pressure

### GeeksforGeeks:
- Use for concept clarification
- Good for Indian placement patterns
- Practice their interview section

## Key Success Metrics

### Phase 1-2: 
- Solve 5-7 problems daily
- 90%+ accuracy on basic queries

### Phase 3-4:
- Solve 3-5 problems daily
- Understand joins intuitively

### Phase 5-6:
- Solve 2-4 complex problems daily
- Optimize existing solutions

## Final Tips for Placement Success

1. **Time Management:** Allocate 2 minutes for easy, 5 minutes for medium, 10 minutes for hard problems during practice
2. **Pattern Recognition:** Keep a notebook of common query patterns
3. **Mock Tests:** Take timed tests weekly from Week 6 onwards
4. **Company Research:** Focus on SQL patterns of your target companies
5. **Code Review:** Always review and optimize your accepted solutions
