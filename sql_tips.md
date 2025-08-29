
---

# 🟢 Your SQL Query Structure

```sql
SELECT ...
FROM ...
WHERE ...
GROUP BY ...
HAVING ...
ORDER BY ...
LIMIT ...
```

---

## 1️⃣ Does this sequence always stay the same?

Yes ✅ the **syntax order is fixed**. You **must** write clauses in this order, otherwise MySQL will throw an error.

* But **execution order** (how DB internally evaluates) is different. Example: `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`.
  👉 So: Syntax order fixed, execution order different.

---

## 2️⃣ Is this list complete or more clauses exist?

Other useful clauses can appear too:

* `JOIN` (INNER, LEFT, RIGHT, FULL) → comes after `FROM`.
* `DISTINCT` → after `SELECT`.
* `UNION / INTERSECT / EXCEPT` → after full query (to combine multiple queries).
* `WINDOW FUNCTIONS` (`OVER`, `PARTITION BY`) → after `SELECT`.

So the **core skeleton is complete**, but in real queries you’ll often add `JOIN`, `DISTINCT`, `UNION`.

---

## 3️⃣ Variations in each clause

Let’s break it down:

### 🔹 `SELECT`

* Column list → `SELECT name, age`
* Aggregates → `SUM(), COUNT(), MAX(), MIN(), AVG()`
* Distinct → `SELECT DISTINCT city`
* Expressions → `SELECT salary*12 AS annual_salary`
* Window functions → `SELECT RANK() OVER (ORDER BY salary)`

### 🔹 `FROM`

* Base table → `FROM employees`
* Join → `FROM employees e JOIN dept d ON e.dept_id = d.id`
* Subquery → `FROM (SELECT ...) t`

### 🔹 `WHERE`

* Comparison → `WHERE age > 25`
* Logical → `WHERE city = 'Delhi' AND salary > 50000`
* Pattern → `WHERE name LIKE 'A%'`
* Range → `WHERE age BETWEEN 20 AND 30`
* Null check → `WHERE manager IS NULL`

### 🔹 `GROUP BY`

* Column grouping → `GROUP BY dept_id`
* Multi-column grouping → `GROUP BY dept_id, city`
* With expressions → `GROUP BY YEAR(joining_date)`

### 🔹 `HAVING`

* Aggregate filter → `HAVING COUNT(*) > 5`
* Multiple conditions → `HAVING AVG(salary) > 60000 AND MIN(age) < 25`

### 🔹 `ORDER BY`

* Ascending/descending → `ORDER BY salary DESC`
* Multiple → `ORDER BY dept_id ASC, salary DESC`
* With expressions → `ORDER BY LENGTH(name)`

### 🔹 `LIMIT`

* Top n → `LIMIT 5`
* Pagination → `LIMIT 10 OFFSET 20`

---

## 4️⃣ Using operators at each level

* `WHERE` → operators like `=, >, <, >=, <=, !=, LIKE, BETWEEN, IN, IS NULL`.
* `HAVING` → same operators but applied **after aggregation**.
* `ORDER BY` → can use functions/operators (`ORDER BY salary*12 DESC`).
* `SELECT` → can use `CASE WHEN` for conditional columns.

---

## 5️⃣ Basics to keep in mind (to avoid confusion)

* **WHERE vs HAVING**:

  * `WHERE` filters **before aggregation**.
  * `HAVING` filters **after aggregation**.
* **GROUP BY rule**: If you use GROUP BY, every column in `SELECT` must be either grouped or aggregated.
* **Execution order**: Always remember → `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`.
* **NULL handling**: Comparisons with NULL need `IS NULL` or `IS NOT NULL`.

---

## 6️⃣ Tips to solve SQL query problems

1. **Break the query into steps**:

   * First get raw data with `SELECT ... FROM`.
   * Then filter with `WHERE`.
   * Then aggregate with `GROUP BY + HAVING`.
   * Then sort with `ORDER BY`.
   * Then restrict with `LIMIT`.

2. **Start simple, then add complexity** → don’t try to write the full query in one shot.

3. **Draw sample data** (like a small 5-row table) → run your query mentally.

4. **Practice common patterns**:

   * “Nth highest salary” → `ORDER BY salary DESC LIMIT n-1,1`
   * “Duplicates” → `GROUP BY column HAVING COUNT(*) > 1`
   * “Employees in one dept not in another” → `WHERE dept_id NOT IN (...)`

5. **Be clear on JOINs** – most tricky part in interviews. Practice `INNER`, `LEFT`, `RIGHT`, `SELF JOIN`.


---

# 🟢 Example Table: `employees`

| emp\_id | name    | dept\_id | salary | city      | join\_date |
| ------- | ------- | -------- | ------ | --------- | ---------- |
| 1       | Alice   | 101      | 60000  | Delhi     | 2020-01-10 |
| 2       | Bob     | 101      | 75000  | Delhi     | 2019-03-12 |
| 3       | Charlie | 102      | 50000  | Mumbai    | 2021-07-19 |
| 4       | David   | 103      | 80000  | Bangalore | 2018-11-23 |
| 5       | Eva     | 101      | 45000  | Delhi     | 2022-05-15 |

---

## 1️⃣ `SELECT`

**Basic**:

```sql
SELECT name, salary FROM employees;
```

👉 Output: Just name and salary.

**Variations**:

* With function →

  ```sql
  SELECT UPPER(name), salary*12 AS annual_salary FROM employees;
  ```
* Distinct →

  ```sql
  SELECT DISTINCT city FROM employees;
  ```
* Aggregates →

  ```sql
  SELECT COUNT(*), AVG(salary) FROM employees;
  ```

---

## 2️⃣ `FROM`

**Basic**:

```sql
SELECT * FROM employees;
```

**Variations**:

* Alias →

  ```sql
  SELECT e.name FROM employees e;
  ```
* Join another table: (say `departments`)

  ```sql
  SELECT e.name, d.dept_name
  FROM employees e
  JOIN departments d ON e.dept_id = d.id;
  ```
* Subquery as table →

  ```sql
  SELECT * FROM (SELECT * FROM employees WHERE salary > 50000) AS high_paid;
  ```

---

## 3️⃣ `WHERE`

**Basic**:

```sql
SELECT * FROM employees WHERE city = 'Delhi';
```

**Variations**:

* Comparison → `WHERE salary > 60000`
* Logical → `WHERE city='Delhi' AND salary > 50000`
* Pattern → `WHERE name LIKE 'A%'`
* Range → `WHERE salary BETWEEN 45000 AND 70000`
* Set → `WHERE dept_id IN (101, 103)`
* Null check → `WHERE dept_id IS NULL`

---

## 4️⃣ `GROUP BY`

**Basic**:

```sql
SELECT dept_id, AVG(salary) 
FROM employees 
GROUP BY dept_id;
```

👉 Average salary per department.

**Variations**:

* Multiple columns →

  ```sql
  SELECT dept_id, city, COUNT(*)
  FROM employees
  GROUP BY dept_id, city;
  ```
* With expression →

  ```sql
  SELECT YEAR(join_date), COUNT(*) 
  FROM employees 
  GROUP BY YEAR(join_date);
  ```

---

## 5️⃣ `HAVING`

**Basic**:

```sql
SELECT dept_id, COUNT(*) 
FROM employees 
GROUP BY dept_id 
HAVING COUNT(*) > 2;
```

👉 Only departments with more than 2 employees.

**Variations**:

* On aggregate → `HAVING AVG(salary) > 60000`
* Multiple conditions →

  ```sql
  HAVING COUNT(*) > 2 AND MIN(salary) > 40000
  ```

---

## 6️⃣ `ORDER BY`

**Basic**:

```sql
SELECT * FROM employees ORDER BY salary DESC;
```

**Variations**:

* Multiple →

  ```sql
  ORDER BY dept_id ASC, salary DESC
  ```
* With expression →

  ```sql
  ORDER BY LENGTH(name)
  ```

---

## 7️⃣ `LIMIT`

**Basic**:

```sql
SELECT * FROM employees LIMIT 3;
```

**Variations**:

* With offset →

  ```sql
  LIMIT 2 OFFSET 2   -- skip first 2 rows, then give next 2
  ```
* Pagination example →
  Page 1 → `LIMIT 10 OFFSET 0`
  Page 2 → `LIMIT 10 OFFSET 10`

---

## 🔑 Extra Clauses

* **DISTINCT**: `SELECT DISTINCT dept_id FROM employees;`
* **UNION**:

  ```sql
  SELECT name FROM employees WHERE city='Delhi'
  UNION
  SELECT name FROM employees WHERE city='Mumbai';
  ```
* **CASE** (inside SELECT):

  ```sql
  SELECT name,
         CASE 
           WHEN salary > 70000 THEN 'High'
           WHEN salary BETWEEN 50000 AND 70000 THEN 'Medium'
           ELSE 'Low'
         END AS salary_band
  FROM employees;
  ```

---

# 📌 Mental Outline (Shortcut to Remember)

1. **SELECT** → What you want (columns, aggregates, distinct).
2. **FROM** → Where it comes from (table, joins, subquery).
3. **WHERE** → Filter rows before grouping.
4. **GROUP BY** → Aggregate rows into groups.
5. **HAVING** → Filter groups after aggregation.
6. **ORDER BY** → Sort results.
7. **LIMIT** → Restrict how many results.


---

# 🟢 Example Table: `employees`

| emp\_id | name    | dept\_id | salary | city      | join\_date |
| ------- | ------- | -------- | ------ | --------- | ---------- |
| 1       | Alice   | 101      | 60000  | Delhi     | 2020-01-10 |
| 2       | Bob     | 101      | 75000  | Delhi     | 2019-03-12 |
| 3       | Charlie | 102      | 50000  | Mumbai    | 2021-07-19 |
| 4       | David   | 103      | 80000  | Bangalore | 2018-11-23 |
| 5       | Eva     | 101      | 45000  | Delhi     | 2022-05-15 |

---

# 🟠 Big Story Query

👉 **Question**:
*“Find the top 2 departments (by average salary), but only consider employees in Delhi with salary > 50,000. Show department id, number of employees, and average salary.”*

---

### ✅ SQL Query

```sql
SELECT dept_id, 
       COUNT(*) AS emp_count, 
       AVG(salary) AS avg_salary
FROM employees
WHERE city = 'Delhi' AND salary > 50000       -- (Filter rows)
GROUP BY dept_id                               -- (Group by department)
HAVING COUNT(*) > 1                            -- (Only groups with >1 employee)
ORDER BY avg_salary DESC                       -- (Sort by avg salary)
LIMIT 2;                                       -- (Take top 2)
```

---

# 🔎 Step-by-step execution flow

1. **FROM employees** → start with the table.
2. **WHERE city='Delhi' AND salary > 50000** → keep only Alice (60000, Delhi) and Bob (75000, Delhi).
3. **GROUP BY dept\_id** → group Alice & Bob together (both dept 101).
4. **SELECT** → for each group: `dept_id=101, COUNT=2, AVG=67500`.
5. **HAVING COUNT(\*) > 1** → keep dept 101 (because 2 employees).
6. **ORDER BY avg\_salary DESC** → only 1 group, so order is trivial.
7. **LIMIT 2** → return top 2 rows (here just 1 row).

👉 **Output**

| dept\_id | emp\_count | avg\_salary |
| -------- | ---------- | ----------- |
| 101      | 2          | 67500       |

---

# 📝 Variations you could try

* Change filter:

  ```sql
  WHERE city IN ('Delhi','Mumbai')
  ```
* Add multiple group keys:

  ```sql
  GROUP BY dept_id, city
  ```
* Change HAVING condition:

  ```sql
  HAVING AVG(salary) > 60000
  ```
* Sort by count instead of salary:

  ```sql
  ORDER BY emp_count DESC
  ```
* Pagination (get 3rd page with 2 rows each):

  ```sql
  LIMIT 2 OFFSET 4
  ```

---

📌 This **one story query** covers:

* `SELECT` (columns, aggregates, alias)
* `FROM`
* `WHERE`
* `GROUP BY`
* `HAVING`
* `ORDER BY`
* `LIMIT`

---

---

# 🟢 Table: `employees`

| emp\_id | name    | dept\_id | salary | city      | join\_date |
| ------- | ------- | -------- | ------ | --------- | ---------- |
| 1       | Alice   | 101      | 60000  | Delhi     | 2020-01-10 |
| 2       | Bob     | 101      | 75000  | Delhi     | 2019-03-12 |
| 3       | Charlie | 102      | 50000  | Mumbai    | 2021-07-19 |
| 4       | David   | 103      | 80000  | Bangalore | 2018-11-23 |
| 5       | Eva     | 101      | 45000  | Delhi     | 2022-05-15 |

---

# 1️⃣ Question: Find the **second highest salary**

### Approach 1 → `LIMIT + OFFSET`

```sql
SELECT salary 
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

### Approach 2 → Subquery with `MAX`

```sql
SELECT MAX(salary) 
FROM employees 
WHERE salary < (SELECT MAX(salary) FROM employees);
```

### Approach 3 → Using `DISTINCT + LIMIT`

```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1,1;
```

---

# 2️⃣ Question: Find **employees with duplicate salaries**

### Approach 1 → `GROUP BY + HAVING`

```sql
SELECT salary
FROM employees
GROUP BY salary
HAVING COUNT(*) > 1;
```

### Approach 2 → `SELF JOIN`

```sql
SELECT e1.name, e1.salary
FROM employees e1
JOIN employees e2 
  ON e1.salary = e2.salary AND e1.emp_id <> e2.emp_id;
```

---

# 3️⃣ Question: Find **employees who earn more than average salary**

### Approach 1 → Subquery

```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### Approach 2 → Window Function

```sql
SELECT name, salary, AVG(salary) OVER() AS avg_salary
FROM employees
WHERE salary > AVG(salary) OVER();
```

---

# 4️⃣ Question: Count employees in each city

### Approach 1 → `GROUP BY`

```sql
SELECT city, COUNT(*) AS emp_count
FROM employees
GROUP BY city;
```

### Approach 2 → Window Function

```sql
SELECT name, city, COUNT(*) OVER(PARTITION BY city) AS emp_count
FROM employees;
```

---

# 5️⃣ Question: Find employees who joined in the most recent year

### Approach 1 → Subquery

```sql
SELECT *
FROM employees
WHERE YEAR(join_date) = (SELECT MAX(YEAR(join_date)) FROM employees);
```

### Approach 2 → Order + Limit

```sql
SELECT *
FROM employees
ORDER BY join_date DESC
LIMIT 1;
```

### Approach 3 → Window Function

```sql
SELECT *
FROM (
  SELECT *, RANK() OVER(ORDER BY join_date DESC) AS rnk
  FROM employees
) t
WHERE rnk = 1;
```

---

# 6️⃣ Question: Find employees in **dept 101 but not in Delhi**

### Approach 1 → Simple WHERE

```sql
SELECT * FROM employees
WHERE dept_id = 101 AND city <> 'Delhi';
```

### Approach 2 → `NOT IN`

```sql
SELECT * FROM employees
WHERE dept_id = 101
AND city NOT IN ('Delhi');
```

### Approach 3 → `EXCEPT` (if supported)

```sql
SELECT * FROM employees WHERE dept_id = 101
EXCEPT
SELECT * FROM employees WHERE dept_id = 101 AND city = 'Delhi';
```

---

**15 most common SQL interview questions**

* **Approach 1 (basic)** – beginner-friendly
* **Approach 2 (advanced)** – alternative (subquery, window function, join, etc.)

We’ll stick with our sample table `employees` (emp\_id, name, dept\_id, salary, city, join\_date).

---

# 🟢 15 Common SQL Interview Questions (with 2 Approaches Each)

---

### 1️⃣ Find the **second highest salary**

* **Approach 1 (LIMIT + OFFSET)**

```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

* **Approach 2 (Subquery)**

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

---

### 2️⃣ Find the **nth highest salary** (say 3rd)

* **Approach 1 (LIMIT)**

```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 2;
```

* **Approach 2 (Window Function)**

```sql
SELECT salary
FROM (
  SELECT salary, DENSE_RANK() OVER(ORDER BY salary DESC) AS rnk
  FROM employees
) t
WHERE rnk = 3;
```

---

### 3️⃣ Find employees with **duplicate salaries**

* **Approach 1 (GROUP BY)**

```sql
SELECT salary
FROM employees
GROUP BY salary
HAVING COUNT(*) > 1;
```

* **Approach 2 (SELF JOIN)**

```sql
SELECT e1.name, e1.salary
FROM employees e1
JOIN employees e2
  ON e1.salary = e2.salary AND e1.emp_id <> e2.emp_id;
```

---

### 4️⃣ Find employees who earn **more than average salary**

* **Approach 1 (Subquery)**

```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

* **Approach 2 (Window Function)**

```sql
SELECT name, salary
FROM (
  SELECT name, salary, AVG(salary) OVER() AS avg_sal
  FROM employees
) t
WHERE salary > avg_sal;
```

---

### 5️⃣ Find the **highest salary in each department**

* **Approach 1 (GROUP BY)**

```sql
SELECT dept_id, MAX(salary)
FROM employees
GROUP BY dept_id;
```

* **Approach 2 (Window Function)**

```sql
SELECT name, dept_id, salary
FROM (
  SELECT name, dept_id, salary,
         RANK() OVER(PARTITION BY dept_id ORDER BY salary DESC) AS rnk
  FROM employees
) t
WHERE rnk = 1;
```

---

### 6️⃣ Count employees in each city

* **Approach 1 (GROUP BY)**

```sql
SELECT city, COUNT(*) AS emp_count
FROM employees
GROUP BY city;
```

* **Approach 2 (Window Function)**

```sql
SELECT name, city, COUNT(*) OVER(PARTITION BY city) AS emp_count
FROM employees;
```

---

### 7️⃣ Find employees who joined in the most recent year

* **Approach 1 (Subquery)**

```sql
SELECT *
FROM employees
WHERE YEAR(join_date) = (SELECT MAX(YEAR(join_date)) FROM employees);
```

* **Approach 2 (Window Function)**

```sql
SELECT *
FROM (
  SELECT *, RANK() OVER(ORDER BY join_date DESC) AS rnk
  FROM employees
) t
WHERE rnk = 1;
```

---

### 8️⃣ Find employees in **dept 101 but not in Delhi**

* **Approach 1 (WHERE)**

```sql
SELECT * FROM employees
WHERE dept_id = 101 AND city <> 'Delhi';
```

* **Approach 2 (NOT IN)**

```sql
SELECT * FROM employees
WHERE dept_id = 101
AND city NOT IN ('Delhi');
```

---

### 9️⃣ Find the **3rd highest salary in each department**

* **Approach 1 (Subquery + LIMIT)**

```sql
SELECT dept_id, salary
FROM employees e1
WHERE 2 = (
  SELECT COUNT(DISTINCT salary)
  FROM employees e2
  WHERE e2.dept_id = e1.dept_id AND e2.salary > e1.salary
);
```

* **Approach 2 (Window Function)**

```sql
SELECT dept_id, name, salary
FROM (
  SELECT dept_id, name, salary,
         DENSE_RANK() OVER(PARTITION BY dept_id ORDER BY salary DESC) AS rnk
  FROM employees
) t
WHERE rnk = 3;
```

---

### 🔟 Find employees with **no manager** (NULL column)

* **Approach 1 (IS NULL)**

```sql
SELECT * FROM employees WHERE manager_id IS NULL;
```

* **Approach 2 (LEFT JOIN)**

```sql
SELECT e.name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id
WHERE m.emp_id IS NULL;
```

---

### 1️⃣1️⃣ Find all departments with **more than 3 employees**

* **Approach 1 (GROUP BY + HAVING)**

```sql
SELECT dept_id, COUNT(*) 
FROM employees
GROUP BY dept_id
HAVING COUNT(*) > 3;
```

* **Approach 2 (Window Function)**

```sql
SELECT DISTINCT dept_id
FROM (
  SELECT dept_id, COUNT(*) OVER(PARTITION BY dept_id) AS cnt
  FROM employees
) t
WHERE cnt > 3;
```

---

### 1️⃣2️⃣ Find **top 5 highest paid employees**

* **Approach 1 (ORDER + LIMIT)**

```sql
SELECT * FROM employees
ORDER BY salary DESC
LIMIT 5;
```

* **Approach 2 (Window Function)**

```sql
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER(ORDER BY salary DESC) AS rn
  FROM employees
) t
WHERE rn <= 5;
```

---

### 1️⃣3️⃣ Find employees whose salary is **above their department average**

* **Approach 1 (Subquery)**

```sql
SELECT name, dept_id, salary
FROM employees e1
WHERE salary > (
  SELECT AVG(salary)
  FROM employees e2
  WHERE e2.dept_id = e1.dept_id
);
```

* **Approach 2 (Window Function)**

```sql
SELECT name, dept_id, salary
FROM (
  SELECT name, dept_id, salary,
         AVG(salary) OVER(PARTITION BY dept_id) AS avg_sal
  FROM employees
) t
WHERE salary > avg_sal;
```

---

### 1️⃣4️⃣ Find duplicate employee names

* **Approach 1 (GROUP BY)**

```sql
SELECT name
FROM employees
GROUP BY name
HAVING COUNT(*) > 1;
```

* **Approach 2 (Window Function)**

```sql
SELECT *
FROM (
  SELECT name, COUNT(*) OVER(PARTITION BY name) AS cnt
  FROM employees
) t
WHERE cnt > 1;
```

---

### 1️⃣5️⃣ Show employees along with their **salary rank across company**

* **Approach 1 (Subquery + COUNT)**

```sql
SELECT e1.name, e1.salary,
       (SELECT COUNT(DISTINCT salary)
        FROM employees e2
        WHERE e2.salary >= e1.salary) AS rank_no
FROM employees e1;
```

* **Approach 2 (Window Function)**

```sql
SELECT name, salary,
       RANK() OVER(ORDER BY salary DESC) AS rank_no
FROM employees;
```

---

---

## 🔑 SQL Querying Tips

1. **Focus on Core 80%**

   * Most interview queries are from:

     * Aggregates (`COUNT, SUM, MAX, MIN, AVG`)
     * `GROUP BY + HAVING`
     * `JOIN` (inner, left, self, cross)
     * `Subquery` (single-row, multi-row, correlated)
     * `Window functions` (`RANK, DENSE_RANK, ROW_NUMBER, LEAD, LAG`)
     * Set operations (`UNION, INTERSECT, EXCEPT`)
   * If you master these, you can solve 80% of questions.

---

2. **Always Think in Steps**

   * When stuck:
     (a) Write a query to fetch base data.
     (b) Apply filters (`WHERE`).
     (c) Add grouping/aggregation.
     (d) Apply ordering/limits.
   * This “layer by layer” approach makes complex queries manageable.

---

3. **Execution Order vs Writing Order**

   * Remember:
     `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`
   * Interviewers often ask “Why can’t you use column alias in WHERE?” → Because `WHERE` executes before `SELECT`.

---

4. **Subquery vs JOIN**

   * Same problem can often be solved in **both ways**.
   * Interviews love this:

     * “Show me using a `JOIN`.”
     * “Now show me using a subquery.”
   * Be flexible.

---

5. **NULL Handling**

   * `= NULL` doesn’t work → always use `IS NULL` / `IS NOT NULL`.
   * Aggregates (`COUNT(*)`, `AVG`) **ignore NULLs** except `COUNT(*)`.
   * Interviewers often test this small detail.

---

6. **Practice on Small Tables**

   * Create 2–3 small sample tables (`employees`, `departments`, `projects`) with 5–10 rows.
   * Try writing queries on them instead of big datasets.
   * This forces you to visualize results and avoid blind coding.

---

7. **Optimization Mindset**

   * For MCQs/coding rounds: correctness > optimization.
   * For interviews: mention alternatives → “We could also index this column to speed up query.”
   * Shows you think like an engineer, not just a coder.

---

8. **Be Ready for Trick Questions**

   * Example: *“Can we use WHERE with aggregate functions?”*

     * No → must use `HAVING`.
   * Example: *“Difference between WHERE and HAVING?”*

     * WHERE filters **rows before grouping**, HAVING filters **groups after aggregation**.

---

9. **Write Neat Queries**

   * Even in tests, indent properly.
   * Interviewers notice if you structure queries clearly.

---

10. **Learn 1 Advanced Concept**

* Just knowing **window functions** or **CTEs** makes you stand out.
* Example:

  ```sql
  WITH avg_salary AS (
    SELECT dept_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY dept_id
  )
  SELECT e.name, e.salary, a.avg_sal
  FROM employees e
  JOIN avg_salary a ON e.dept_id = a.dept_id;
  ```
* Even if not required, you can mention: *“We could also solve this with a CTE for readability.”*
* Interviewer gets impressed.

---

⚡ **Strategy:**

* Daily: Pick **1 basic question** (like “second highest salary”) and solve it in **3 ways**.
* Weekly: Revise **execution order + key functions**.
* Before interviews: Go through **your outline of 15 questions with multiple approaches**.

---

