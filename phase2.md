# SQL Phase 2: Data Filtering & Functions Tutorial

## Introduction to Phase 2
In Phase 1, you learned basic queries. Now we'll make SQL work harder for you by manipulating and transforming data.

---

## Lesson 1: Advanced WHERE Conditions

### Sample Tables We'll Use:

**employees table:**
| id | name        | email              | age | department | salary | hire_date  | phone       |
|----|-------------|--------------------|----|------------|--------|------------|-------------|
| 1  | John Smith  | john@company.com   | 25 | IT         | 50000  | 2022-03-15 | 9876543210  |
| 2  | Sarah Jones | sarah@company.com  | 30 | HR         | 45000  | 2021-01-20 | NULL        |
| 3  | Mike Wilson | mike@company.com   | 28 | IT         | 55000  | 2020-05-10 | 9876543211  |
| 4  | Lisa Brown  | lisa@company.com   | 26 | Finance    | 48000  | 2023-02-14 | 9876543212  |

### 1.1 IN Operator - Multiple Values
Instead of writing multiple OR conditions, use IN:

```sql
-- Instead of this:
SELECT * FROM employees 
WHERE department = 'IT' OR department = 'HR' OR department = 'Finance';

-- Write this:
SELECT * FROM employees 
WHERE department IN ('IT', 'HR', 'Finance');
```

### 1.2 BETWEEN Operator - Range Values
```sql
-- Find employees aged between 25 and 28
SELECT name, age FROM employees 
WHERE age BETWEEN 25 AND 28;
```
**Result:**
| name        | age |
|-------------|-----|
| John Smith  | 25  |
| Mike Wilson | 28  |
| Lisa Brown  | 26  |

### 1.3 NOT Operator - Exclude Values
```sql
-- Find employees NOT in IT department
SELECT name, department FROM employees 
WHERE department NOT IN ('IT');
```

### 1.4 Complex Conditions
```sql
-- Find IT employees with salary > 50000 OR any employee aged < 27
SELECT name, department, salary, age FROM employees 
WHERE (department = 'IT' AND salary > 50000) OR age < 27;
```

### Practice Time! 🎯
1. Find employees with salary between 45000 and 50000
2. Find employees NOT in HR or Finance departments
3. Find employees aged 25, 26, or 30

**Answers:**
```sql
-- 1.
SELECT * FROM employees WHERE salary BETWEEN 45000 AND 50000;

-- 2. 
SELECT * FROM employees WHERE department NOT IN ('HR', 'Finance');

-- 3.
SELECT * FROM employees WHERE age IN (25, 26, 30);
```

---

## Lesson 2: String Functions

### Why String Functions?
Real data is messy. Names might have different cases, extra spaces, or you might need to extract parts of text.

### 2.1 UPPER() and LOWER()
```sql
-- Convert names to uppercase
SELECT UPPER(name) as upper_name FROM employees;
```
**Result:**
| upper_name   |
|--------------|
| JOHN SMITH   |
| SARAH JONES  |
| MIKE WILSON  |
| LISA BROWN   |

```sql
-- Convert to lowercase
SELECT LOWER(email) as lower_email FROM employees;
```

### 2.2 LENGTH() - String Length
```sql
-- Find length of employee names
SELECT name, LENGTH(name) as name_length FROM employees;
```
**Result:**
| name        | name_length |
|-------------|-------------|
| John Smith  | 10          |
| Sarah Jones | 11          |
| Mike Wilson | 11          |
| Lisa Brown  | 10          |

### 2.3 SUBSTRING() - Extract Part of String
```sql
-- Get first 4 characters of name
SELECT name, SUBSTRING(name, 1, 4) as first_four FROM employees;
```
**Result:**
| name        | first_four |
|-------------|------------|
| John Smith  | John       |
| Sarah Jones | Sara       |
| Mike Wilson | Mike       |
| Lisa Brown  | Lisa       |

### 2.4 LIKE - Pattern Matching
Very important for placements!

```sql
-- Find employees whose name starts with 'J'
SELECT name FROM employees WHERE name LIKE 'J%';
```
**Result:**
| name       |
|------------|
| John Smith |

**LIKE Wildcards:**
- `%` - Zero or more characters
- `_` - Exactly one character

**More Examples:**
```sql
-- Names ending with 'son'
SELECT name FROM employees WHERE name LIKE '%son';

-- Names with exactly 4 characters
SELECT name FROM employees WHERE name LIKE '____';

-- Emails from company.com domain
SELECT name, email FROM employees WHERE email LIKE '%@company.com';
```

### 2.5 CONCAT() - Joining Strings
```sql
-- Create full display name
SELECT CONCAT(name, ' (', department, ')') as display_name FROM employees;
```
**Result:**
| display_name      |
|-------------------|
| John Smith (IT)   |
| Sarah Jones (HR)  |
| Mike Wilson (IT)  |
| Lisa Brown (Finance) |

### Practice Time! 🎯
1. Find employees whose name contains 'i' (upper or lowercase)
2. Find employees with email ending in '.com'
3. Create a display showing "Name: John, Age: 25" format

**Answers:**
```sql
-- 1.
SELECT * FROM employees WHERE UPPER(name) LIKE '%I%';

-- 2.
SELECT * FROM employees WHERE email LIKE '%.com';

-- 3.
SELECT CONCAT('Name: ', name, ', Age: ', age) as display FROM employees;
```

---

## Lesson 3: Date Functions

### Why Date Functions?
Dates are everywhere in business data. You need to extract years, calculate differences, find recent records, etc.

### Sample date queries with our table:

### 3.1 YEAR(), MONTH(), DAY()
```sql
-- Extract year from hire date
SELECT name, hire_date, YEAR(hire_date) as hire_year FROM employees;
```
**Result:**
| name        | hire_date  | hire_year |
|-------------|------------|-----------|
| John Smith  | 2022-03-15 | 2022      |
| Sarah Jones | 2021-01-20 | 2021      |
| Mike Wilson | 2020-05-10 | 2020      |
| Lisa Brown  | 2023-02-14 | 2023      |

### 3.2 Current Date Functions
```sql
-- Current date and time
SELECT NOW() as current_datetime;
SELECT CURDATE() as current_date;

-- Find employees hired this year
SELECT name FROM employees WHERE YEAR(hire_date) = YEAR(NOW());
```

### 3.3 Date Arithmetic
```sql
-- Find employees hired in last 2 years
SELECT name, hire_date FROM employees 
WHERE hire_date >= DATE_SUB(NOW(), INTERVAL 2 YEAR);

-- Alternative using DATEDIFF
SELECT name, hire_date FROM employees 
WHERE DATEDIFF(NOW(), hire_date) <= 730; -- 730 days = ~2 years
```

### 3.4 Formatting Dates
```sql
-- Format hire date nicely
SELECT name, DATE_FORMAT(hire_date, '%M %d, %Y') as formatted_date 
FROM employees;
```
**Result:**
| name        | formatted_date  |
|-------------|-----------------|
| John Smith  | March 15, 2022  |
| Sarah Jones | January 20, 2021|
| Mike Wilson | May 10, 2020    |
| Lisa Brown  | February 14, 2023|

---

## Lesson 4: Mathematical Functions

### 4.1 Basic Math Operations
```sql
-- Calculate annual bonus (10% of salary)
SELECT name, salary, salary * 0.10 as bonus FROM employees;
```

### 4.2 ROUND(), CEIL(), FLOOR()
```sql
-- Round salary to nearest thousand
SELECT name, salary, ROUND(salary, -3) as rounded_salary FROM employees;
```

### 4.3 Mathematical Functions
```sql
-- Calculate salary statistics
SELECT 
    name,
    salary,
    ABS(salary - 50000) as diff_from_50k,
    ROUND(salary/12, 2) as monthly_salary
FROM employees;
```

---

## Lesson 5: Handling NULL Values

### What is NULL?
NULL means "no value" or "unknown". It's different from 0 or empty string.

### 5.1 Finding NULL Values
```sql
-- Find employees without phone numbers
SELECT name FROM employees WHERE phone IS NULL;
```
**Result:**
| name        |
|-------------|
| Sarah Jones |

### 5.2 Finding Non-NULL Values
```sql
-- Find employees WITH phone numbers
SELECT name, phone FROM employees WHERE phone IS NOT NULL;
```

### 5.3 COALESCE() - Replace NULL
```sql
-- Replace NULL phone with 'Not Provided'
SELECT name, COALESCE(phone, 'Not Provided') as contact_phone FROM employees;
```
**Result:**
| name        | contact_phone |
|-------------|---------------|
| John Smith  | 9876543210    |
| Sarah Jones | Not Provided  |
| Mike Wilson | 9876543211    |
| Lisa Brown  | 9876543212    |

### 5.4 IFNULL() - Alternative to COALESCE
```sql
-- Same as above, different function
SELECT name, IFNULL(phone, 'No Phone') as phone_display FROM employees;
```

---

## Lesson 6: Combining Everything

### Real-world Example Problems:

### Problem 1: Employee Directory
Create a clean employee directory showing full name in caps, department, and contact info:

```sql
SELECT 
    UPPER(name) as full_name,
    department,
    COALESCE(phone, 'Contact HR') as phone_number,
    YEAR(hire_date) as joined_year
FROM employees 
WHERE department IN ('IT', 'HR')
ORDER BY department, name;
```

### Problem 2: Recent Hires Analysis
Find employees hired in the last 3 years with complete contact info:

```sql
SELECT 
    name,
    email,
    phone,
    DATEDIFF(NOW(), hire_date) as days_employed
FROM employees 
WHERE hire_date >= DATE_SUB(NOW(), INTERVAL 3 YEAR)
    AND phone IS NOT NULL
ORDER BY hire_date DESC;
```

### Problem 3: Data Cleanup
Find potential data issues (missing phone or very short names):

```sql
SELECT 
    name,
    CASE 
        WHEN phone IS NULL THEN 'Missing Phone'
        WHEN LENGTH(name) < 5 THEN 'Short Name'
        ELSE 'OK'
    END as data_status
FROM employees
WHERE phone IS NULL OR LENGTH(name) < 5;
```

---

## Phase 2 Practice Problems

### Level 1: Basic Functions
1. Find all employees whose name is longer than 10 characters
2. Convert all department names to lowercase
3. Find employees hired in 2022

### Level 2: Pattern Matching
4. Find employees whose email starts with their first name (hint: use SUBSTRING and LIKE)
5. Find employees with phone numbers containing '543'
6. Find employees whose name contains exactly one space (first name + last name)

### Level 3: Complex Filtering
7. Find employees hired in the first quarter (Jan-Mar) of any year
8. Find employees whose name starts with a vowel (A, E, I, O, U)
9. Create a report showing "Name (Department) - Contact: Phone"

### Solutions:
```sql
-- 1.
SELECT name FROM employees WHERE LENGTH(name) > 10;

-- 2.
SELECT name, LOWER(department) as dept FROM employees;

-- 3.
SELECT name FROM employees WHERE YEAR(hire_date) = 2022;

-- 4.
SELECT name, email FROM employees 
WHERE email LIKE CONCAT(SUBSTRING(name, 1, POSITION(' ' IN name) - 1), '%');

-- 5.
SELECT name, phone FROM employees WHERE phone LIKE '%543%';

-- 6.
SELECT name FROM employees 
WHERE LENGTH(name) - LENGTH(REPLACE(name, ' ', '')) = 1;

-- 7.
SELECT name, hire_date FROM employees 
WHERE MONTH(hire_date) IN (1, 2, 3);

-- 8.
SELECT name FROM employees 
WHERE UPPER(SUBSTRING(name, 1, 1)) IN ('A', 'E', 'I', 'O', 'U');

-- 9.
SELECT CONCAT(name, ' (', department, ') - Contact: ', 
              COALESCE(phone, 'N/A')) as employee_info 
FROM employees;
```

---

## Common Interview Patterns in Phase 2

### Pattern 1: Data Validation
"Find records with potential data quality issues"
```sql
SELECT * FROM employees 
WHERE email NOT LIKE '%@%.%' 
   OR LENGTH(name) < 3 
   OR phone IS NULL;
```

### Pattern 2: Date-based Analysis
"Find employees hired in the same month as today"
```sql
SELECT name, hire_date FROM employees 
WHERE MONTH(hire_date) = MONTH(NOW());
```

### Pattern 3: String Manipulation
"Create username from email (part before @)"
```sql
SELECT name, email, 
       SUBSTRING(email, 1, POSITION('@' IN email) - 1) as username 
FROM employees;
```

---

## Phase 2 Completion Checklist ✅

Before moving to Phase 3, ensure you can:
- [ ] Use IN, BETWEEN, NOT operators confidently
- [ ] Apply string functions (UPPER, LOWER, LENGTH, SUBSTRING)
- [ ] Use LIKE with wildcards for pattern matching
- [ ] Work with date functions (YEAR, MONTH, DATE functions)
- [ ] Handle NULL values properly (IS NULL, IS NOT NULL, COALESCE)
- [ ] Combine multiple conditions with complex logic
- [ ] Solve 20-25 problems involving functions
- [ ] Create formatted output using CONCAT

---

## Key Functions Quick Reference

### String Functions:
```sql
UPPER(column)          -- Convert to uppercase
LOWER(column)          -- Convert to lowercase  
LENGTH(column)         -- Get string length
SUBSTRING(column, start, length)  -- Extract substring
CONCAT(str1, str2)     -- Join strings
LIKE 'pattern'         -- Pattern matching
```

### Date Functions:
```sql
NOW()                  -- Current date and time
CURDATE()             -- Current date only
YEAR(date)            -- Extract year
MONTH(date)           -- Extract month
DAY(date)             -- Extract day
DATEDIFF(date1, date2) -- Difference in days
DATE_ADD(date, INTERVAL n UNIT) -- Add time
DATE_SUB(date, INTERVAL n UNIT) -- Subtract time
```

### NULL Functions:
```sql
IS NULL               -- Check if NULL
IS NOT NULL          -- Check if not NULL
COALESCE(col, default) -- Replace NULL with default
IFNULL(col, default)   -- MySQL specific NULL replacement
```

### Math Functions:
```sql
ROUND(number, decimals) -- Round number
CEIL(number)           -- Round up
FLOOR(number)          -- Round down
ABS(number)            -- Absolute value
```

---

## Recommended Practice Schedule

### Week 3:
**Day 1-2:** Master IN, BETWEEN, NOT operators
**Day 3-4:** String functions (UPPER, LOWER, LENGTH, SUBSTRING)
**Day 5-7:** Pattern matching with LIKE

### Week 4:
**Day 8-10:** Date functions and date arithmetic
**Day 11-12:** NULL handling
**Day 13-14:** Complex combinations and review

### Platform Focus:
- **HackerRank:** Advanced Select challenges
- **LeetCode:** Problems 196 (Delete Duplicates), 175 (Combine Two Tables)
- **GeeksforGeeks:** SQL string functions section

---

## Common Mistakes to Avoid

1. **Case Sensitivity:** Remember that SQL keywords are not case-sensitive, but data values are
   ```sql
   -- This works
   WHERE department = 'IT'
   -- This might not find anything
   WHERE department = 'it'
   ```

2. **NULL Comparisons:** Never use = or != with NULL
   ```sql
   -- WRONG
   WHERE phone = NULL
   -- CORRECT  
   WHERE phone IS NULL
   ```

3. **LIKE without Wildcards:** Using LIKE without % or _ is same as =
   ```sql
   -- These are the same
   WHERE name LIKE 'John'
   WHERE name = 'John'
   ```

4. **Date Formats:** Be consistent with date formats
   ```sql
   -- Preferred format
   WHERE hire_date >= '2022-01-01'
   ```

---

## Real Placement Question Examples

### Question 1: Data Cleaning
"Write a query to find employees with invalid email addresses (not containing @ symbol)"
```sql
SELECT name, email FROM employees 
WHERE email NOT LIKE '%@%' OR email IS NULL;
```

### Question 2: Business Logic
"Find employees hired in Q1 (first quarter) with salary above department average"
```sql
-- We'll learn subqueries in Phase 5, but here's the date part:
SELECT name, hire_date, salary FROM employees 
WHERE MONTH(hire_date) IN (1, 2, 3);
```

### Question 3: Report Generation
"Create a summary showing 'Employee: NAME has worked for X years'"
```sql
SELECT CONCAT('Employee: ', name, ' has worked for ', 
              FLOOR(DATEDIFF(NOW(), hire_date)/365), ' years') as summary
FROM employees;
```

---

## Moving to Phase 3

Once you're confident with Phase 2, you'll be ready for **Phase 3: Aggregation & Grouping**. This is where SQL becomes really powerful for data analysis!

**You're ready for Phase 3 when you can:**
- Solve string manipulation problems quickly
- Handle date calculations confidently  
- Write complex WHERE conditions
- Deal with NULL values properly
