
---

# 📘 Mock Database: `company_db`

We’ll create **5 core tables**:

## 1. `employees`

```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    gender CHAR(1),
    salary INT,
    hire_date DATE,
    dept_id INT,
    manager_id INT
);
```

---

## 2. `departments`

```sql
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50),
    location VARCHAR(50),
    manager_id INT
);
```

---

## 3. `projects`

```sql
CREATE TABLE projects (
    proj_id INT PRIMARY KEY,
    proj_name VARCHAR(50),
    location VARCHAR(50),
    dept_id INT
);
```

---

## 4. `employee_project` (Many-to-Many between employees & projects)

```sql
CREATE TABLE employee_project (
    emp_id INT,
    proj_id INT,
    role VARCHAR(50),
    PRIMARY KEY (emp_id, proj_id)
);
```

---

## 5. `salaries_history` (For tracking changes / advanced queries)

```sql
CREATE TABLE salaries_history (
    emp_id INT,
    salary INT,
    effective_date DATE
);
```

---

# 📊 Sample Data

We’ll insert a **small dataset** (so queries give meaningful results).

## `departments`

```sql
INSERT INTO departments VALUES
(10, 'Sales', 'New York', 101),
(20, 'Engineering', 'San Francisco', 102),
(30, 'HR', 'Chicago', 103),
(40, 'Finance', 'New York', 104);
```

---

## `employees`

```sql
INSERT INTO employees VALUES
(101, 'Alice', 'F', 90000, '2015-03-01', 10, NULL),   -- Manager Sales
(102, 'Bob', 'M', 120000, '2012-07-15', 20, NULL),    -- Manager Engineering
(103, 'Clara', 'F', 85000, '2016-01-10', 30, NULL),   -- Manager HR
(104, 'David', 'M', 110000, '2013-09-20', 40, NULL),  -- Manager Finance

(201, 'Eva', 'F', 70000, '2018-06-12', 10, 101),
(202, 'Frank', 'M', 60000, '2019-02-25', 10, 101),
(203, 'Grace', 'F', 75000, '2020-11-01', 20, 102),
(204, 'Henry', 'M', 50000, '2021-03-18', 20, 102),
(205, 'Ivy', 'F', 40000, '2022-08-22', 30, 103),
(206, 'Jack', 'M', 65000, '2019-12-05', 40, 104);
```

---

## `projects`

```sql
INSERT INTO projects VALUES
(1001, 'Alpha', 'New York', 10),
(1002, 'Beta', 'San Francisco', 20),
(1003, 'Gamma', 'Chicago', 30),
(1004, 'Delta', 'New York', 40),
(1005, 'Epsilon', 'San Francisco', 20);
```

---

## `employee_project`

```sql
INSERT INTO employee_project VALUES
(201, 1001, 'Sales Exec'),
(202, 1001, 'Sales Support'),
(203, 1002, 'Engineer'),
(204, 1002, 'Engineer'),
(203, 1005, 'Lead Engineer'),
(205, 1003, 'HR Associate'),
(206, 1004, 'Analyst'),
(201, 1004, 'Cross Team Support');
```

---

## `salaries_history`

```sql
INSERT INTO salaries_history VALUES
(201, 60000, '2019-01-01'),
(201, 65000, '2020-01-01'),
(201, 70000, '2021-01-01'),
(202, 50000, '2019-01-01'),
(202, 55000, '2020-01-01'),
(202, 60000, '2021-01-01'),
(203, 70000, '2020-01-01'),
(203, 75000, '2021-01-01');
```

---

✅ This dataset is small but **rich enough** for:

* **Basic SELECT / WHERE queries**
* **Aggregates (`SUM`, `AVG`, `MAX`, etc.)**
* **Joins (`INNER`, `LEFT`, `SELF`)**
* **Subqueries (`IN`, `EXISTS`)**
* **Group By / Having**
* **Window functions (later 150–200 problems)**
* **Advanced interview-style queries**

---
