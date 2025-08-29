Perfect ✅ This is the **right way to practice** — start with 50 structured problems (foundation), then move to the next 50 (intermediate + advanced).

I’ll first give you:

1. **Tables (schema + sample rows)**
2. **50 basic SQL problems** grouped by topic

Later, I’ll give you the **next 50** for joins, subqueries, window functions, and advanced cases.

---

# 📊 Sample Tables (you’ll use these for practice)

### 1. `Employees`

| emp\_id | name  | dept\_id | salary | hire\_date | manager\_id |
| ------- | ----- | -------- | ------ | ---------- | ----------- |
| 1       | Alice | 10       | 60000  | 2015-01-12 | NULL        |
| 2       | Bob   | 20       | 45000  | 2017-03-19 | 1           |
| 3       | Carol | 10       | 70000  | 2014-07-01 | 1           |
| 4       | David | 30       | 55000  | 2019-06-23 | 2           |
| 5       | Eve   | 20       | 50000  | 2020-11-02 | 2           |

---

### 2. `Departments`

| dept\_id | dept\_name  |
| -------- | ----------- |
| 10       | HR          |
| 20       | Engineering |
| 30       | Marketing   |

---

### 3. `Projects`

| proj\_id | proj\_name     | dept\_id | budget |
| -------- | -------------- | -------- | ------ |
| 101      | Recruitment    | 10       | 20000  |
| 102      | Website Revamp | 20       | 50000  |
| 103      | Ad Campaign    | 30       | 30000  |

---

### 4. `Attendance`

| emp\_id | work\_date | status |
| ------- | ---------- | ------ |
| 1       | 2023-07-01 | P      |
| 2       | 2023-07-01 | A      |
| 3       | 2023-07-01 | P      |
| 4       | 2023-07-01 | P      |
| 5       | 2023-07-01 | P      |

---

# 📝 50 Basic SQL Problems (Phase 1)

---

## A. **SELECT & WHERE (Filtering)**

1. Display all employees.
2. Show employee names and salaries only.
3. Find employees earning more than 50,000.
4. Get employees hired after 2016.
5. List employees from department 20.
6. Show employees whose name starts with “A”.
7. Find employees not in department 10.
8. Display employees with salary between 45,000 and 60,000.
9. Show employees whose name ends with “e”.
10. Get all distinct department IDs from employees.

---

## B. **ORDER BY & LIMIT**

11. Display employees sorted by salary (ascending).
12. Display employees sorted by salary (descending).
13. Show top 3 highest paid employees.
14. Show the employee with the lowest salary.
15. List employees ordered by hire date (earliest first).

---

## C. **AGGREGATE FUNCTIONS**

16. Find total number of employees.
17. Find average salary of employees.
18. Find maximum salary.
19. Find minimum salary.
20. Find sum of salaries of department 20 employees.

---

## D. **GROUP BY**

21. Find number of employees in each department.
22. Find average salary per department.
23. Find maximum salary per department.
24. Find minimum salary per department.
25. Show departments with total salary > 1,00,000.

---

## E. **HAVING**

26. Find departments having more than 1 employee.
27. Show departments where average salary > 55,000.
28. Find departments where max salary < 60,000.
29. Get department IDs where total salary > 80,000.
30. Show departments having exactly 2 employees.

---

## F. **JOINS (Basic)**

31. Display employee names with their department names.
32. List employees with project names from their department.
33. Show department name with number of employees in it.
34. Find employees working in “Engineering”.
35. List employees and their manager names (self-join).

---

## G. **STRING & DATE FUNCTIONS**

36. Show employee names in uppercase.
37. Show employee names in lowercase.
38. Find length of each employee name.
39. Show hire year of each employee.
40. Show employees hired in March (any year).

---

## H. **SET OPERATIONS**

41. Get all department IDs from `Departments` and `Employees` (UNION).
42. Get department IDs present in both `Departments` and `Employees` (INTERSECT).
43. Get department IDs in `Departments` but not in `Employees` (EXCEPT).

---

## I. **SUBQUERIES (Simple)**

44. Find employees with salary greater than average salary.
45. Find employees who earn the maximum salary.
46. Find employees who earn the minimum salary.
47. Find department with highest average salary.
48. Find employee(s) hired earliest.

---

## J. **MISC**

49. Count number of present employees from `Attendance` table.
50. Find employees who were absent on `2023-07-01`.

---

⚡ With these **50 questions**, you’ll cover:
✅ Filtering (`WHERE`)
✅ Sorting (`ORDER BY`, `LIMIT`)
✅ Aggregation + grouping (`GROUP BY`, `HAVING`)
✅ Joins
✅ Subqueries
✅ Set operations
✅ Functions

---

👉 Next phase (next 50) will cover: **nested subqueries, advanced joins, window functions (RANK, DENSE\_RANK, ROW\_NUMBER), CTEs, case statements, update/delete queries, schema design queries**.
