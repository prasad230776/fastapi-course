# SQL & Relational Database Essentials

A concise reference guide covering SQL fundamentals, database structure, and query operations up to JOINs. Examples are based on the **Employee Management System** database.

---

## 1. Introduction & Relational Database Concepts

*   **Database:** A structured collection of data.
*   **Relational Database:** Stores data in **tables** (rows/columns) connected through defined relationships.
*   **Primary Key (PK):** A unique, non-null column identifying each record (e.g., `employee_id`).
*   **Foreign Key (FK):** A column referencing a Primary Key in another table, creating a relationship (e.g., `department_id` in the `employee` table).

### Database Relationships
*   **One-to-Many (1:N):** One department has many employees. (Established by placing the department's PK as a FK in the employee table).
*   **Many-to-Many (M:N):** Many students enroll in many courses. (Requires a junction table linking the PKs of both).

---

## 2. SQLite Data Types & Schema Setup

SQLite is a lightweight database that stores everything in a single file and uses **Type Affinity**:

*   `INTEGER`: Whole numbers (IDs, counts).
*   `REAL`: Floating-point/decimal numbers (salaries, prices).
*   `TEXT`: Strings (names, emails).
*   `BLOB`: Binary data (images, documents).
*   `NULL`: Missing or unknown values.

### Sample Schema Setup Example (DDL)
```sql
-- Create Department Table
CREATE TABLE department (
    department_id INTEGER PRIMARY KEY,
    department_name TEXT NOT NULL
);

-- Create Employee Table
CREATE TABLE employee (
    employee_id INTEGER PRIMARY KEY,
    employee_name TEXT NOT NULL,
    salary REAL NOT NULL,
    department_id INTEGER,
    joining_date TEXT NOT NULL,
    FOREIGN KEY (department_id) REFERENCES department(department_id)
);
```

---

## 3. SQL Command Subsets

SQL commands are grouped into:
1.  **DDL (Data Definition Language):** Defines structure (`CREATE`, `ALTER`, `DROP`).
2.  **DML (Data Manipulation Language):** Manages data rows (`INSERT`, `UPDATE`, `DELETE`).
3.  **DQL (Data Query Language):** Retrieves data (`SELECT`).
4.  **TCL (Transaction Control Language):** Manages transaction units (`BEGIN`, `COMMIT`, `ROLLBACK`).

---

## 4. DDL & DML Commands (Modifying Structure & Data)

### Data Definition Language (DDL) Examples
```sql
-- Add a new column
ALTER TABLE employee ADD COLUMN email TEXT;

-- Rename a column
ALTER TABLE employee RENAME COLUMN joining_date TO start_date;

-- Drop table
DROP TABLE IF EXISTS old_projects;
```

### Data Manipulation Language (DML) Examples
```sql
-- Insert a row
INSERT INTO department (department_id, department_name) 
VALUES (1, 'Engineering');

-- Update matching records (Always use WHERE to avoid updating all rows!)
UPDATE employee 
SET salary = 75000.00 
WHERE employee_id = 101;

-- Delete matching records (Always use WHERE to avoid deleting all rows!)
DELETE FROM employee 
WHERE start_date < '2021-01-01';
```

---

## 5. DQL: Querying and Filtering Data (SELECT)

DQL is used to retrieve and filter records.

### Basic Retrieval
```sql
-- Select specific columns and apply alias
SELECT employee_name AS Name, salary * 12 AS annual_salary 
FROM employee;

-- Get unique values
SELECT DISTINCT department_id FROM employee;

-- Limit results and skip first 2 rows
SELECT employee_name FROM employee LIMIT 5 OFFSET 2;
```

### Filtering & Operators (`WHERE`)
```sql
-- Combine filters
SELECT * FROM employee 
WHERE salary >= 60000 AND city = 'Bengaluru';

-- Range, List and Wildcard patterns
SELECT * FROM employee 
WHERE salary BETWEEN 50000 AND 80000
  AND department_id IN (1, 2)
  AND employee_name LIKE 'A%'; -- Names starting with 'A'
```

### Sorting, Aggregation, and Grouping
```sql
-- Sort results
SELECT * FROM employee ORDER BY salary DESC, employee_name ASC;

-- Aggregate Functions: COUNT, SUM, AVG, MIN, MAX
SELECT COUNT(*) AS total_employees, AVG(salary) AS avg_salary 
FROM employee;

-- Grouping with filters (WHERE vs HAVING)
SELECT department_id, AVG(salary) AS avg_sal
FROM employee
WHERE city = 'Hyderabad'            -- Filter rows (before grouping)
GROUP BY department_id
HAVING AVG(salary) > 60000;         -- Filter groups (after grouping)
```

---

## 6. Joining Tables

Joins combine related records across tables using a matching key column.

### Explicit JOIN Examples (Recommended)
```sql
-- INNER JOIN: Matches rows in both tables
SELECT e.employee_name, d.department_name
FROM employee e
INNER JOIN department d ON e.department_id = d.department_id;

-- LEFT JOIN: All left records, and matches (or NULLs) from the right
SELECT e.employee_name, d.department_name
FROM employee e
LEFT JOIN department d ON e.department_id = d.department_id;
```

*(Note: SQLite does not support native `RIGHT JOIN` or `FULL OUTER JOIN`.)*
