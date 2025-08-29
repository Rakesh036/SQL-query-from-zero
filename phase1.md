# SQL Phase 1: Foundation Tutorial

## What is SQL?
SQL (Structured Query Language) is a language used to communicate with databases. Think of it as a way to ask questions to your data and get specific answers.

## Understanding Databases

### What is a Database?
A database is like a digital filing cabinet that stores information in an organized way.

### What is a Table?
A table is like a spreadsheet with rows and columns:
- **Columns (Fields):** Categories of information (like Name, Age, Salary)
- **Rows (Records):** Individual entries (like one person's complete information)

**Example Table: employees**
| id | name     | age | department | salary |
|----|----------|-----|------------|--------|
| 1  | John     | 25  | IT         | 50000  |
| 2  | Sarah    | 30  | HR         | 45000  |
| 3  | Mike     | 28  | IT         | 55000  |
| 4  | Lisa     | 26  | Finance    | 48000  |

---

## Lesson 1: Your First SQL Query - SELECT

### Basic Syntax:
```sql
SELECT column_name FROM table_name;
```

### Example 1: Get all information
```sql
SELECT * FROM employees;
```
**What this does:** Shows ALL columns (*) from the employees table
**Result:** The entire table above

### Example 2: Get specific columns
```sql
SELECT name, salary FROM employees;
```
**Result:**
| name  | salary |
|-------|--------|
| John  | 50000  |
| Sarah | 45000  |
| Mike  | 55000  |
| Lisa  | 48000  |

### Example 3: Get just one column
```sql
SELECT department FROM employees;
```
**Result:**
| department |
|------------|
| IT         |
| HR         |
| IT         |
| Finance    |

### Practice Time! 🎯
Try these queries (imagine you're writing them):
1. Get all employee names
2. Get all employee ages and departments
3. Get complete employee information

**Answers:**
1. `SELECT name FROM employees;`
2. `SELECT age, department FROM employees;`
3. `SELECT * FROM employees;`

---

## Lesson 2: Filtering Data with WHERE

### Why WHERE?
Sometimes you don't want ALL the data. You want specific information based on conditions.

### Basic WHERE Syntax:
```sql
SELECT column_name FROM table_name WHERE condition;
```

### Example 1: Find employees in IT department
```sql
SELECT * FROM employees WHERE department = 'IT';
```
**Result:**
| id | name | age | department | salary |
|----|------|-----|------------|--------|
| 1  | John | 25  | IT         | 50000  |
| 3  | Mike | 28  | IT         | 55000  |

### Example 2: Find employees with salary greater than 50000
```sql
SELECT name, salary FROM employees WHERE salary > 50000;
```
**Result:**
| name | salary |
|------|--------|
| Mike | 55000  |

### Common WHERE Operators:
- `=` Equal to
- `>` Greater than
- `<` Less than
- `>=` Greater than or equal to
- `<=` Less than or equal to
- `!=` or `<>` Not equal to

### Example 3: Multiple conditions with AND
```sql
SELECT name FROM employees WHERE age > 25 AND department = 'IT';
```
**Result:**
| name |
|------|
| Mike |

### Example 4: Multiple conditions with OR
```sql
SELECT name FROM employees WHERE department = 'IT' OR department = 'HR';
```
**Result:**
| name  |
|-------|
| John  |
| Sarah |
| Mike  |

### Practice Time! 🎯
1. Find all employees aged 26 or younger
2. Find employees in Finance department
3. Find employees with salary between 45000 and 50000 (use >= and <=)

**Answers:**
1. `SELECT * FROM employees WHERE age <= 26;`
2. `SELECT * FROM employees WHERE department = 'Finance';`
3. `SELECT * FROM employees WHERE salary >= 45000 AND salary <= 50000;`

---

## Lesson 3: Sorting Results with ORDER BY

### Why ORDER BY?
Data often comes in random order. ORDER BY helps you organize results in a meaningful way.

### Basic Syntax:
```sql
SELECT column_name FROM table_name ORDER BY column_name ASC/DESC;
```
- **ASC:** Ascending (low to high, A to Z) - this is default
- **DESC:** Descending (high to low, Z to A)

### Example 1: Sort by salary (lowest first)
```sql
SELECT name, salary FROM employees ORDER BY salary;
```
**Result:**
| name  | salary |
|-------|--------|
| Sarah | 45000  |
| Lisa  | 48000  |
| John  | 50000  |
| Mike  | 55000  |

### Example 2: Sort by salary (highest first)
```sql
SELECT name, salary FROM employees ORDER BY salary DESC;
```
**Result:**
| name  | salary |
|-------|--------|
| Mike  | 55000  |
| John  | 50000  |
| Lisa  | 48000  |
| Sarah | 45000  |

### Example 3: Sort by multiple columns
```sql
SELECT * FROM employees ORDER BY department, salary DESC;
```
**This sorts by department first, then within each department by salary (highest first)**

### Practice Time! 🎯
1. Sort employees by age (youngest first)
2. Sort employees by name (alphabetically)
3. Find employees in IT department, sorted by salary (highest first)

**Answers:**
1. `SELECT * FROM employees ORDER BY age;`
2. `SELECT * FROM employees ORDER BY name;`
3. `SELECT * FROM employees WHERE department = 'IT' ORDER BY salary DESC;`

---

## Lesson 4: Limiting Results with LIMIT

### Why LIMIT?
Sometimes you only want to see a few results, especially when dealing with large datasets.

### Basic Syntax:
```sql
SELECT column_name FROM table_name LIMIT number;
```

### Example 1: Get top 2 highest paid employees
```sql
SELECT name, salary FROM employees ORDER BY salary DESC LIMIT 2;
```
**Result:**
| name | salary |
|------|--------|
| Mike | 55000  |
| John | 50000  |

### Example 2: Get youngest employee
```sql
SELECT name, age FROM employees ORDER BY age LIMIT 1;
```
**Result:**
| name | age |
|------|-----|
| John | 25  |

---

## Common Data Types You Should Know

### 1. Numbers
- **INT:** Whole numbers (1, 2, 100)
- **DECIMAL/FLOAT:** Numbers with decimals (3.14, 99.99)

### 2. Text
- **VARCHAR:** Variable length text ('John', 'IT Department')
- **CHAR:** Fixed length text

### 3. Dates
- **DATE:** Just the date (2023-12-25)
- **DATETIME:** Date and time (2023-12-25 14:30:00)

---

## Phase 1 Practice Problems

### Easy Level:
1. Write a query to get all product names from a products table
2. Find all customers from 'Mumbai' city
3. Get the first 5 orders from an orders table

### Medium Level:
4. Find all products with price greater than 1000, sorted by price
5. Get customer names and cities, sorted alphabetically by name
6. Find the 3 most expensive products

### Solutions:
```sql
-- 1.
SELECT product_name FROM products;

-- 2.
SELECT * FROM customers WHERE city = 'Mumbai';

-- 3.
SELECT * FROM orders LIMIT 5;

-- 4.
SELECT * FROM products WHERE price > 1000 ORDER BY price;

-- 5.
SELECT customer_name, city FROM customers ORDER BY customer_name;

-- 6.
SELECT * FROM products ORDER BY price DESC LIMIT 3;
```

---

## Phase 1 Completion Checklist ✅

Before moving to Phase 2, make sure you can:
- [ ] Write basic SELECT statements
- [ ] Use WHERE with different operators (=, >, <, >=, <=)
- [ ] Combine conditions with AND/OR
- [ ] Sort results with ORDER BY (both ASC and DESC)
- [ ] Limit results with LIMIT
- [ ] Understand basic data types
- [ ] Solve 15-20 basic problems on HackerRank/LeetCode

## Next Steps
Once you're comfortable with these concepts, we'll move to Phase 2 where you'll learn advanced filtering, string functions, and date operations.

**Recommended Practice:**
- HackerRank: Complete "Basic Select" domain
- LeetCode: Problems 595, 584, 183, 1757
- GeeksforGeeks: SQL SELECT statement exercises

---

## Quick Reference Card

```sql
-- Basic SELECT
SELECT column1, column2 FROM table_name;
SELECT * FROM table_name;

-- WHERE clause
SELECT * FROM table_name WHERE condition;
SELECT * FROM table_name WHERE col1 = 'value' AND col2 > 100;

-- ORDER BY
SELECT * FROM table_name ORDER BY column_name;
SELECT * FROM table_name ORDER BY column_name DESC;

-- LIMIT
SELECT * FROM table_name LIMIT 5;
SELECT * FROM table_name ORDER BY column_name LIMIT 3;
```


