# SQL Phase 4: Joins & Relationships Tutorial

## Introduction to Phase 4
JOINs are the backbone of relational databases. They allow you to combine data from multiple tables to get meaningful information. **90% of real-world SQL queries use JOINs!** This is the most tested concept in placements.

---

## Understanding Table Relationships

### Sample Database Schema:

**employees table:**
| emp_id | name        | dept_id | salary | manager_id |
|--------|-------------|---------|--------|------------|
| 1      | John Smith  | 1       | 50000  | 3          |
| 2      | Sarah Jones | 2       | 45000  | 4          |
| 3      | Mike Wilson | 1       | 75000  | NULL       |
| 4      | Lisa Brown  | 2       | 60000  | NULL       |
| 5      | Tom Davis   | 1       | 55000  | 3          |
| 6      | Amy White   | 3       | 48000  | 7          |
| 7      | Bob Green   | 3       | 70000  | NULL       |

**departments table:**
| dept_id | dept_name | location  |
|---------|-----------|-----------|
| 1       | IT        | Bangalore |
| 2       | HR        | Mumbai    |
| 3       | Finance   | Delhi     |
| 4       | Marketing | Chennai   |

**projects table:**
| project_id | project_name | emp_id | start_date |
|------------|--------------|--------|------------|
| 101        | Website      | 1      | 2024-01-15 |
| 102        | Mobile App   | 3      | 2024-02-01 |
| 103        | Database     | 5      | 2024-01-10 |
| 104        | Analytics    | 1      | 2024-03-01 |
| 105        | Security     | 8      | 2024-02-15 |

---

## Lesson 1: INNER JOIN - The Most Common Join

### What is INNER JOIN?
INNER JOIN returns only records that have matching values in both tables.

### 1.1 Basic INNER JOIN Syntax
```sql
SELECT columns
FROM table1 
INNER JOIN table2 ON table1.column = table2.column;
```

### 1.2 Employee-Department JOIN
```sql
-- Get employee names with their department names
SELECT 
    e.name, 
    d.dept_name 
FROM employees e 
INNER JOIN departments d ON e.dept_id = d.dept_id;
```
**Result:**
| name        | dept_name |
|-------------|-----------|
| John Smith  | IT        |
| Sarah Jones | HR        |
| Mike Wilson | IT        |
| Lisa Brown  | HR        |
| Tom Davis   | IT        |
| Amy White   | Finance   |
| Bob Green   | Finance   |

**Notice:** Only employees with matching departments appear. Amy White (dept_id 3) appears because Finance department exists.

### 1.3 Table Aliases
Using aliases (e, d) makes queries cleaner and is essential for complex JOINs:
```sql
-- Without aliases (harder to read)
SELECT employees.name, departments.dept_name 
FROM employees 
INNER JOIN departments ON employees.dept_id = departments.dept_id;

-- With aliases (cleaner)
SELECT e.name, d.dept_name 
FROM employees e 
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

### 1.4 JOIN with WHERE and ORDER BY
```sql
-- IT employees with their department location, sorted by salary
SELECT 
    e.name, 
    e.salary, 
    d.dept_name, 
    d.location
FROM employees e 
INNER JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_name = 'IT'
ORDER BY e.salary DESC;
```

### Practice Time! 🎯
1. Get all employee names with their department locations
2. Find employees in Mumbai, show name and department
3. Get Finance employees with salary > 50000

**Answers:**
```sql
-- 1.
SELECT e.name, d.location 
FROM employees e 
INNER JOIN departments d ON e.dept_id = d.dept_id;

-- 2.
SELECT e.name, d.dept_name 
FROM employees e 
INNER JOIN departments d ON e.dept_id = d.dept_id
WHERE d.location = 'Mumbai';

-- 3.
SELECT e.name, e.salary 
FROM employees e 
INNER JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_name = 'Finance' AND e.salary > 50000;
```

---

## Lesson 2: LEFT JOIN - Include All from Left Table

### What is LEFT JOIN?
LEFT JOIN returns ALL records from the left table, and matching records from the right table. If no match, NULL values for right table columns.

### 2.1 Basic LEFT JOIN
```sql
-- Get ALL employees with their department info (even if no department match)
SELECT 
    e.name, 
    d.dept_name 
FROM employees e 
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```
**Result:** Same as INNER JOIN in our case because all employees have valid dept_id.

### 2.2 More Realistic LEFT JOIN Example
```sql
-- Get ALL employees with their project info (some might not have projects)
SELECT 
    e.name, 
    p.project_name
FROM employees e 
LEFT JOIN projects p ON e.emp_id = p.emp_id;
```
**Result:**
| name        | project_name |
|-------------|--------------|
| John Smith  | Website      |
| John Smith  | Analytics    |
| Sarah Jones | NULL         |
| Mike Wilson | Mobile App   |
| Lisa Brown  | NULL         |
| Tom Davis   | Database     |
| Amy White   | NULL         |
| Bob Green   | NULL         |

### 2.3 Finding Records Without Matches
```sql
-- Find employees who are NOT assigned to any project
SELECT e.name 
FROM employees e 
LEFT JOIN projects p ON e.emp_id = p.emp_id
WHERE p.emp_id IS NULL;
```
**Result:**
| name        |
|-------------|
| Sarah Jones |
| Lisa Brown  |
| Amy White   |
| Bob Green   |

---

## Lesson 3: RIGHT JOIN - Include All from Right Table

### What is RIGHT JOIN?
RIGHT JOIN returns ALL records from the right table, and matching records from the left table.

### 3.1 Basic RIGHT JOIN
```sql
-- Get ALL departments with their employees (even if no employees)
SELECT 
    d.dept_name, 
    e.name 
FROM employees e 
RIGHT JOIN departments d ON e.dept_id = d.dept_id;
```
**Result:**
| dept_name | name        |
|-----------|-------------|
| IT        | John Smith  |
| IT        | Mike Wilson |
| IT        | Tom Davis   |
| HR        | Sarah Jones |
| HR        | Lisa Brown  |
| Finance   | Amy White   |
| Finance   | Bob Green   |
| Marketing | NULL        |

**Notice:** Marketing department appears with NULL employee because no one works there.

---

## Lesson 4: SELF JOIN - Joining Table to Itself

### What is SELF JOIN?
When you need to compare rows within the same table or find relationships within one table.

### 4.1 Manager-Employee Relationship
```sql
-- Find each employee with their manager's name
SELECT 
    emp.name as employee_name,
    mgr.name as manager_name
FROM employees emp
LEFT JOIN employees mgr ON emp.manager_id = mgr.emp_id;
```
**Result:**
| employee_name | manager_name |
|---------------|--------------|
| John Smith    | Mike Wilson  |
| Sarah Jones   | Lisa Brown   |
| Mike Wilson   | NULL         |
| Lisa Brown    | NULL         |
| Tom Davis     | Mike Wilson  |
| Amy White     | Bob Green    |
| Bob Green     | NULL         |

### 4.2 Finding Colleagues (Same Department)
```sql
-- Find employees who work in the same department
SELECT 
    e1.name as employee1,
    e2.name as employee2,
    e1.dept_id
FROM employees e1
JOIN employees e2 ON e1.dept_id = e2.dept_id
WHERE e1.emp_id < e2.emp_id;  -- Avoid duplicate pairs
```

---

## Lesson 5: Multiple Table JOINs

### Real-world scenarios often need 3+ tables:

### 5.1 Three Table JOIN
```sql
-- Employee details with department and project information
SELECT 
    e.name,
    d.dept_name,
    p.project_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
LEFT JOIN projects p ON e.emp_id = p.emp_id;
```

### 5.2 Complex Business Query
```sql
-- Complete employee profile with manager info
SELECT 
    e.name as employee,
    d.dept_name,
    d.location,
    mgr.name as manager,
    p.project_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
LEFT JOIN employees mgr ON e.manager_id = mgr.emp_id
LEFT JOIN projects p ON e.emp_id = p.emp_id
ORDER BY d.dept_name, e.name;
```

---

## Lesson 6: JOINs with Aggregation

### This is where things get powerful for business analysis!

### 6.1 Count Employees per Department
```sql
SELECT 
    d.dept_name,
    COUNT(e.emp_id) as employee_count
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_id, d.dept_name;
```
**Result:**
| dept_name | employee_count |
|-----------|----------------|
| IT        | 3              |
| HR        | 2              |
| Finance   | 2              |
| Marketing | 0              |

### 6.2 Department Salary Analysis
```sql
SELECT 
    d.dept_name,
    COUNT(e.emp_id) as emp_count,
    COALESCE(SUM(e.salary), 0) as total_salary,
    COALESCE(AVG(e.salary), 0) as avg_salary
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_id, d.dept_name
ORDER BY avg_salary DESC;
```

### 6.3 Project Assignment Analysis
```sql
-- How many projects each employee is working on?
SELECT 
    e.name,
    COUNT(p.project_id) as project_count
FROM employees e
LEFT JOIN projects p ON e.emp_id = p.emp_id
GROUP BY e.emp_id, e.name
ORDER BY project_count DESC;
```

---

## Lesson 7: Advanced JOIN Scenarios

### 7.1 Finding Unmatched Records
```sql
-- Departments with no employees
SELECT d.dept_name 
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
WHERE e.emp_id IS NULL;
```

### 7.2 Many-to-Many Relationships
```sql
-- If employees can work on multiple projects (realistic scenario)
SELECT 
    e.name,
    d.dept_name,
    COUNT(p.project_id) as active_projects,
    GROUP_CONCAT(p.project_name) as project_list
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
LEFT JOIN projects p ON e.emp_id = p.emp_id
GROUP BY e.emp_id, e.name, d.dept_name;
```

---

## Lesson 8: Common Interview Patterns

### Pattern 1: Master-Detail Reports
**"Show each department with total employees and total salary budget"**
```sql
SELECT 
    d.dept_name,
    d.location,
    COUNT(e.emp_id) as total_employees,
    COALESCE(SUM(e.salary), 0) as total_budget
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_id, d.dept_name, d.location
ORDER BY total_budget DESC;
```

### Pattern 2: Hierarchy Queries
**"Show all managers with count of their direct reports"**
```sql
SELECT 
    mgr.name as manager,
    COUNT(emp.emp_id) as direct_reports
FROM employees mgr
LEFT JOIN employees emp ON mgr.emp_id = emp.manager_id
WHERE mgr.manager_id IS NULL  -- Only top-level managers
GROUP BY mgr.emp_id, mgr.name;
```

### Pattern 3: Cross-Reference Analysis
**"Find employees working on projects but show their department info too"**
```sql
SELECT 
    e.name,
    d.dept_name,
    p.project_name,
    p.start_date
FROM projects p
INNER JOIN employees e ON p.emp_id = e.emp_id
INNER JOIN departments d ON e.dept_id = d.dept_id
ORDER BY d.dept_name, e.name;
```

---

## Lesson 9: JOIN Performance & Best Practices

### 9.1 Always Use Table Aliases
```sql
-- Good practice
SELECT e.name, d.dept_name 
FROM employees e 
JOIN departments d ON e.dept_id = d.dept_id;
```

### 9.2 Be Specific About JOIN Type
```sql
-- Explicit is better than implicit
SELECT e.name, d.dept_name 
FROM employees e 
INNER JOIN departments d ON e.dept_id = d.dept_id;  -- Clear intent
```

### 9.3 JOIN Order Matters for Performance
```sql
-- Start with the table that will have fewer results after filtering
SELECT e.name, d.dept_name 
FROM departments d 
INNER JOIN employees e ON d.dept_id = e.dept_id
WHERE d.location = 'Mumbai';  -- Filter first, then join
```

---

## Real Placement Problems

### Problem 1: Employee Directory
**"Create a complete employee directory showing name, department, location, manager name"**

```sql
SELECT 
    e.name as employee,
    d.dept_name,
    d.location,
    COALESCE(m.name, 'No Manager') as manager
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
LEFT JOIN employees m ON e.manager_id = m.emp_id
ORDER BY d.dept_name, e.name;
```

### Problem 2: Project Allocation Report
**"Show department-wise project allocation with employee details"**

```sql
SELECT 
    d.dept_name,
    COUNT(DISTINCT e.emp_id) as total_employees,
    COUNT(p.project_id) as total_projects,
    ROUND(COUNT(p.project_id) * 1.0 / COUNT(DISTINCT e.emp_id), 2) as projects_per_employee
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
LEFT JOIN projects p ON e.emp_id = p.emp_id
GROUP BY d.dept_id, d.dept_name
ORDER BY projects_per_employee DESC;
```

### Problem 3: Missing Data Analysis
**"Find all data inconsistencies: employees without departments, departments without employees, projects without valid employees"**

```sql
-- Employees without valid departments
SELECT e.name, e.dept_id 
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_id IS NULL;

-- Departments without employees  
SELECT d.dept_name 
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
WHERE e.emp_id IS NULL;

-- Projects with invalid employee references
SELECT p.project_name, p.emp_id 
FROM projects p
LEFT JOIN employees e ON p.emp_id = e.emp_id
WHERE e.emp_id IS NULL;
```

---

## Lesson 10: Advanced JOIN Scenarios

### 10.1 Multiple Conditions in JOIN
```sql
-- Join employees and projects, but only recent projects
SELECT 
    e.name,
    p.project_name
FROM employees e
INNER JOIN projects p ON e.emp_id = p.emp_id 
                     AND p.start_date >= '2024-02-01';
```

### 10.2 JOIN with Aggregation from Previous Phase
```sql
-- Department performance: employee count, avg salary, project count
SELECT 
    d.dept_name,
    COUNT(DISTINCT e.emp_id) as employee_count,
    AVG(e.salary) as avg_salary,
    COUNT(p.project_id) as project_count
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
LEFT JOIN projects p ON e.emp_id = p.emp_id
GROUP BY d.dept_id, d.dept_name
HAVING employee_count > 0;
```

### 10.3 Self JOIN for Comparisons
```sql
-- Find employees earning more than their manager
SELECT 
    emp.name as employee,
    emp.salary as emp_salary,
    mgr.name as manager,
    mgr.salary as mgr_salary
FROM employees emp
INNER JOIN employees mgr ON emp.manager_id = mgr.emp_id
WHERE emp.salary > mgr.salary;
```

---

## Lesson 11: FULL OUTER JOIN (When Available)

### What is FULL OUTER JOIN?
Returns all records when there's a match in either table. Shows everything from both tables.

```sql
-- MySQL doesn't have FULL OUTER JOIN, but you can simulate it:
SELECT e.name, d.dept_name 
FROM employees e 
LEFT JOIN departments d ON e.dept_id = d.dept_id

UNION

SELECT e.name, d.dept_name 
FROM employees e 
RIGHT JOIN departments d ON e.dept_id = d.dept_id;
```

---

## Phase 4 Practice Problems

### Level 1: Basic JOINs
1. List all employees with their department names
2. Find all projects with employee names who are working on them
3. Show employees and their manager names (use SELF JOIN)

### Level 2: JOIN + Filtering
4. Find IT employees working on projects started in 2024
5. Show departments located in Mumbai with their employees
6. Find employees who earn more than any employee in HR department

### Level 3: Complex Analysis
7. Department efficiency: Show dept name, employee count, project count, and avg salary
8. Find employees who are managers (appear as manager_id for other employees)
9. Create organization chart showing employee → manager → manager's department

### Level 4: Advanced Scenarios
10. Find departments where all employees are working on projects
11. Show employees who are working on more projects than their manager
12. Find the highest-paid employee in each location

### Solutions:
```sql
-- 1.
SELECT e.name, d.dept_name 
FROM employees e 
INNER JOIN departments d ON e.dept_id = d.dept_id;

-- 2.
SELECT p.project_name, e.name 
FROM projects p 
INNER JOIN employees e ON p.emp_id = e.emp_id;

-- 3.
SELECT e.name as employee, m.name as manager 
FROM employees e 
LEFT JOIN employees m ON e.manager_id = m.emp_id;

-- 4.
SELECT e.name, p.project_name 
FROM employees e 
INNER JOIN departments d ON e.dept_id = d.dept_id
INNER JOIN projects p ON e.emp_id = p.emp_id
WHERE d.dept_name = 'IT' AND p.start_date >= '2024-01-01';

-- 5.
SELECT d.dept_name, e.name 
FROM departments d 
LEFT JOIN employees e ON d.dept_id = e.dept_id
WHERE d.location = 'Mumbai';

-- 6.
SELECT e.name, e.salary 
FROM employees e 
WHERE e.salary > (
    SELECT MAX(hr_emp.salary) 
    FROM employees hr_emp 
    INNER JOIN departments d ON hr_emp.dept_id = d.dept_id
    WHERE d.dept_name = 'HR'
);

-- 7.
SELECT 
    d.dept_name,
    COUNT(DISTINCT e.emp_id) as emp_count,
    COUNT(p.project_id) as project_count,
    AVG(e.salary) as avg_salary
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
LEFT JOIN projects p ON e.emp_id = p.emp_id
GROUP BY d.dept_id, d.dept_name;

-- 8.
SELECT DISTINCT m.name as manager 
FROM employees e 
INNER JOIN employees m ON e.manager_id = m.emp_id;

-- 9.
SELECT 
    e.name as employee,
    m.name as manager,
    md.dept_name as manager_dept
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id
LEFT JOIN departments md ON m.dept_id = md.dept_id;

-- 10.
SELECT d.dept_name 
FROM departments d
INNER JOIN employees e ON d.dept_id = e.dept_id
LEFT JOIN projects p ON e.emp_id = p.emp_id
GROUP BY d.dept_id, d.dept_name
HAVING COUNT(DISTINCT e.emp_id) = COUNT(DISTINCT p.emp_id);

-- 11.
SELECT 
    emp.name,
    emp_projects.project_count as emp_project_count,
    mgr_projects.project_count as mgr_project_count
FROM employees emp
INNER JOIN employees mgr ON emp.manager_id = mgr.emp_id
INNER JOIN (
    SELECT emp_id, COUNT(*) as project_count 
    FROM projects GROUP BY emp_id
) emp_projects ON emp.emp_id = emp_projects.emp_id
INNER JOIN (
    SELECT emp_id, COUNT(*) as project_count 
    FROM projects GROUP BY emp_id
) mgr_projects ON mgr.emp_id = mgr_projects.emp_id
WHERE emp_projects.project_count > mgr_projects.project_count;

-- 12.
SELECT 
    d.location,
    e.name,
    e.salary
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
INNER JOIN (
    SELECT d2.location, MAX(e2.salary) as max_salary
    FROM employees e2
    INNER JOIN departments d2 ON e2.dept_id = d2.dept_id
    GROUP BY d2.location
) location_max ON d.location = location_max.location 
              AND e.salary = location_max.max_salary;
```

---

## JOIN Types Quick Reference

### Visual Memory Aid:
```
Table A    Table B
   🔴        🔵
   
INNER JOIN:     🟣 (only matching parts)
LEFT JOIN:      🔴🟣 (all of A + matches)  
RIGHT JOIN:     🟣🔵 (matches + all of B)
FULL OUTER:     🔴🟣🔵 (everything)
```

### Syntax Templates:
```sql
-- INNER JOIN
SELECT a.col, b.col 
FROM tableA a 
INNER JOIN tableB b ON a.key = b.key;

-- LEFT JOIN  
SELECT a.col, b.col 
FROM tableA a 
LEFT JOIN tableB b ON a.key = b.key;

-- SELF JOIN
SELECT a.col, b.col 
FROM table a 
JOIN table b ON a.foreign_key = b.primary_key;

-- Multiple JOINs
SELECT a.col, b.col, c.col
FROM tableA a
JOIN tableB b ON a.key = b.key
LEFT JOIN tableC c ON b.key = c.key;
```

---

## Phase 4 Completion Checklist ✅

You're ready for Phase 5 when you can:
- [ ] Write INNER JOINs confidently
- [ ] Understand when to use LEFT JOIN vs INNER JOIN
- [ ] Create SELF JOINs for hierarchical data
- [ ] Combine JOINs with GROUP BY and aggregation
- [ ] Handle multiple table JOINs (3+ tables)
- [ ] Find unmatched records using JOINs
- [ ] Solve hierarchy/manager-employee problems
- [ ] Debug JOIN queries when results don't look right
- [ ] Solve 30-40 JOIN problems across different complexity levels

---

## Platform Strategy for Phase 4

### HackerRank:
- Complete "Basic Join" section entirely
- Move to "Advanced Join" section
- Focus on the challenging problems

### LeetCode:
- Problems: 175, 176, 177, 178, 180, 181
- Focus on Employee/Department themed problems
- Practice under time pressure (5-8 minutes per problem)

### GeeksforGeeks:
- SQL JOINs comprehensive section
- Practice their interview preparation problems

---

## Pro Tips for JOIN Mastery

1. **Draw the relationship first** - understand what you're connecting
2. **Start with INNER JOIN** - get the basic query working
3. **Add LEFT JOIN when you need "all records from left table"**
4. **Use meaningful aliases** - e for employees, d for departments
5. **Test with small datasets** - understand the logic before optimizing
