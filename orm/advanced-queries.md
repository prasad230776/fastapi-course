---
title: "Adv Queries"
description: "Advanced queries are used when working with multiple tables, nested queries, analytical reports, and custom SQL."
---

# Joins

A **join** combines data from two or more related tables based on a common column.

In SQLAlchemy, joins are performed using the `join()` and `outerjoin()` methods.

## Types of Joins

SQL supports two categories of joins:

- **Inner Join**
- **Outer Join**

The **Outer Join** category includes:

- Left Outer Join
- Right Outer Join
- Full Outer Join

```text
Join
│
├── Inner Join
└── Outer Join
      │
      ├── Left Outer Join
      ├── Right Outer Join
      └── Full Outer Join
```

> **Note:** In SQL, the `OUTER` keyword is optional.
>
> - `LEFT JOIN` = `LEFT OUTER JOIN`
> - `RIGHT JOIN` = `RIGHT OUTER JOIN`
> - `FULL JOIN` = `FULL OUTER JOIN`

## Sample Tables

### courses

| id | name | duration |
|---:|--------|---------:|
| 1 | Python | 6 |
| 2 | Java | 8 |
| 3 | React | 4 |
| 4 | AI | 10 |

### students

| id | name | age | course_id |
|---:|--------|----:|----------:|
| 1 | Arjun | 20 | 1 |
| 2 | Priya | 22 | 2 |
| 3 | Rahul | 19 | 1 |
| 4 | Sneha | 21 | 2 |
| 5 | Kiran | 20 | 3 |
| 6 | Nikhil | 20 | NULL |

## Inner Join

Returns only the rows that have matching records in both tables.

### Task

Retrieve each student's name along with the course they enrolled in.

### SQL

```sql
SELECT s.name, c.name
FROM students s
INNER JOIN courses c
ON s.course_id = c.id;
```

### SQLAlchemy

```python
stmt = (
    select(Student.name, Course.name)
    .join(Course)
)

result = session.execute(stmt).all()
```

### Expected Output

| Student | Course |
|----------|---------|
| Arjun | Python |
| Rahul | Python |
| Priya | Java |
| Sneha | Java |
| Kiran | React |

## Left Outer Join

Returns **all rows from the left table** and matching rows from the right table.

If no matching record exists, the right-side columns contain `NULL`.

### Task

Retrieve all students along with their course names, including students who are not enrolled in any course.

### SQL

```sql
SELECT s.name, c.name
FROM students s
LEFT OUTER JOIN courses c
ON s.course_id = c.id;
```

### SQLAlchemy

```python
stmt = (
    select(Student.name, Course.name)
    .outerjoin(Course)
)

result = session.execute(stmt).all()
```

### Expected Output

| Student | Course |
|----------|---------|
| Arjun | Python |
| Rahul | Python |
| Priya | Java |
| Sneha | Java |
| Kiran | React |
| Nikhil | NULL |

## Right Outer Join

Returns **all rows from the right table** and matching rows from the left table.

If no matching record exists, the left-side columns contain `NULL`.

> **Note:** SQLite does not support `RIGHT OUTER JOIN`.

### Task

Retrieve all courses along with their enrolled students, including courses that have no students.

### SQL

```sql
SELECT s.name, c.name
FROM students s
RIGHT OUTER JOIN courses c
ON s.course_id = c.id;
```

### SQLAlchemy

```python
# SQLite workaround

stmt = (
    select(Student.name, Course.name)
    .select_from(Course)
    .outerjoin(Student)
)

result = session.execute(stmt).all()
```

### Expected Output

| Student | Course |
|----------|---------|
| Arjun | Python |
| Rahul | Python |
| Priya | Java |
| Sneha | Java |
| Kiran | React |
| NULL | AI |

## Full Outer Join

Returns all rows from both tables.

Matching rows are combined, while non-matching rows contain `NULL`.

> **Note:** SQLite does not support `FULL OUTER JOIN`.

### Task

Retrieve every student and every course, even if they are not related.

### SQL

```sql
SELECT s.name, c.name
FROM students s
FULL OUTER JOIN courses c
ON s.course_id = c.id;
```

### SQLAlchemy

```python
# Supported by databases such as PostgreSQL

stmt = (
    select(Student.name, Course.name)
    .join(Course, full=True)
)

result = session.execute(stmt).all()
```

### Expected Output

| Student | Course |
|----------|---------|
| Arjun | Python |
| Rahul | Python |
| Priya | Java |
| Sneha | Java |
| Kiran | React |
| Nikhil | NULL |
| NULL | AI |

## Multiple Join

A multiple join combines more than two tables.

### Sample Tables

### departments

| id | name |
|---:|-------------------|
| 1 | Computer Science |
| 2 | Information Technology |

### courses

| id | name | department_id |
|---:|--------|--------------:|
| 1 | Python | 1 |
| 2 | Java | 1 |
| 3 | React | 2 |

### students

| id | name | course_id |
|---:|--------|----------:|
| 1 | Arjun | 1 |
| 2 | Priya | 2 |
| 3 | Rahul | 1 |

### Relationship

```text
Students
    │
    ▼
Courses
    │
    ▼
Departments
```

### Task

Retrieve each student's name, course name, and department name.

### SQL

```sql
SELECT s.name, c.name, d.name
FROM students s
INNER JOIN courses c
ON s.course_id = c.id
INNER JOIN departments d
ON c.department_id = d.id;
```

### SQLAlchemy

```python
stmt = (
    select(
        Student.name,
        Course.name,
        Department.name
    )
    .join(Course)
    .join(Department)
)

result = session.execute(stmt).all()
```

### Expected Output

| Student | Course | Department |
|----------|--------|-------------------|
| Arjun | Python | Computer Science |
| Priya | Java | Computer Science |
| Rahul | Python | Computer Science |

## Summary

| Join | Returns |
|------|----------|
| Inner Join | Only matching rows |
| Left Outer Join | All rows from the left table and matching rows from the right table |
| Right Outer Join | All rows from the right table and matching rows from the left table |
| Full Outer Join | All rows from both tables |
| Multiple Join | Combines data from more than two related tables |

## Aliases

An alias is a temporary name given to a table or a column within a query.

Aliases improve readability and are useful when:

- The same table is used multiple times.
- Table names are long.
- Column names become ambiguous after joins.

In SQLAlchemy:

- `aliased()` creates a table alias.
- `label()` creates a column alias.

### Table Alias

#### Task

Retrieve the student names using a table alias.

##### SQL

```sql
SELECT s.name
FROM students AS s;
```

##### SQLAlchemy

```python
from sqlalchemy.orm import aliased

S = aliased(Student)

stmt = select(S.name)

result = session.execute(stmt).all()
```

### Column Alias

#### Task

Display the student's name as **Student Name**.

##### SQL

```sql
SELECT name AS "Student Name"
FROM students;
```

##### SQLAlchemy

```python
stmt = (
    select(
        Student.name.label("Student Name")
    )
)

result = session.execute(stmt).all()
```

### Self Join

Aliases are required when the same table appears multiple times.

#### Task

Retrieve every employee along with their manager's name.

##### SQL

```sql
SELECT
    e.name,
    m.name
FROM employees e
JOIN employees m
ON e.manager_id = m.id;
```

##### SQLAlchemy

```python
Manager = aliased(Employee)

stmt = (
    select(
        Employee.name,
        Manager.name
    )
    .join(
        Manager,
        Employee.manager_id == Manager.id
    )
)

result = session.execute(stmt).all()
```

## Subqueries

A subquery is a query written inside another query.

The result of the inner query is used by the outer query.

### Task

Retrieve students enrolled in courses having a duration greater than 6 months.

##### SQL

```sql
SELECT *
FROM students
WHERE course_id IN (
    SELECT id
    FROM courses
    WHERE duration > 6
);
```

##### SQLAlchemy

```python
subquery = (
    select(Course.id)
    .where(Course.duration > 6)
)

stmt = (
    select(Student)
    .where(Student.course_id.in_(subquery))
)

students = session.execute(stmt).scalars().all()
```

### Scalar Subquery

A scalar subquery returns a single value.

#### Task

Retrieve students whose marks are greater than the average marks.

##### SQL

```sql
SELECT *
FROM students
WHERE marks >
(
    SELECT AVG(marks)
    FROM students
);
```

##### SQLAlchemy

```python
avg_marks = (
    select(func.avg(Student.marks))
    .scalar_subquery()
)

stmt = (
    select(Student)
    .where(Student.marks > avg_marks)
)

students = session.execute(stmt).scalars().all()
```

## EXISTS

`EXISTS` checks whether the inner query returns at least one matching row.

It is commonly used to determine whether related records exist.

### Task

Retrieve all courses having at least one enrolled student.

##### SQL

```sql
SELECT *
FROM courses c
WHERE EXISTS (
    SELECT *
    FROM students s
    WHERE s.course_id = c.id
);
```

##### SQLAlchemy

```python
from sqlalchemy import exists

stmt = (
    select(Course)
    .where(
        exists(
            select(Student.id)
            .where(Student.course_id == Course.id)
        )
    )
)

courses = session.execute(stmt).scalars().all()
```

## Raw SQL

Sometimes writing SQL directly is simpler than using ORM methods.

SQLAlchemy allows executing raw SQL using `text()`.

#### Task

Retrieve students from Hyderabad.

##### SQL

```sql
SELECT *
FROM students
WHERE city = 'Hyderabad';
```

##### SQLAlchemy

```python
from sqlalchemy import text

stmt = text("""
SELECT *
FROM students
WHERE city = 'Hyderabad'
""")

result = session.execute(stmt)
```

## Window Functions

Window functions perform calculations across a set of related rows while preserving every row in the result.

Unlike `GROUP BY`, which combines multiple rows into a single row, window functions keep every row and add calculated values.

They are called **Window Functions** because the calculation is performed over a **window (subset)** of rows.

```text
GROUP BY
────────
Multiple Rows
      │
      ▼
Single Row Per Group

Window Function
───────────────
Multiple Rows
      │
      ▼
Same Rows + Calculated Values
```

Window functions are commonly used for:

- Ranking
- Row numbering
- Running totals
- Moving averages
- Previous and next row comparison

### ROW_NUMBER()

Assign a unique row number based on descending marks.

##### SQL

```sql
SELECT
    name,
    marks,
    ROW_NUMBER() OVER (
        ORDER BY marks DESC
    ) AS row_no
FROM students;
```

##### SQLAlchemy

```python
stmt = (
    select(
        Student.name,
        Student.marks,
        func.row_number().over(
            order_by=Student.marks.desc()
        ).label("row_no")
    )
)

result = session.execute(stmt).all()
```

### RANK()

Students with equal marks receive the same rank, and the next rank is skipped.

```text
97 → Rank 1
95 → Rank 2
95 → Rank 2
92 → Rank 4
```

##### SQL

```sql
SELECT
    name,
    marks,
    RANK() OVER (
        ORDER BY marks DESC
    ) AS rank
FROM students;
```

##### SQLAlchemy

```python
stmt = (
    select(
        Student.name,
        Student.marks,
        func.rank().over(
            order_by=Student.marks.desc()
        ).label("rank")
    )
)

result = session.execute(stmt).all()
```

### DENSE_RANK()

Students with equal marks receive the same rank, and the next rank is not skipped.

```text
97 → Rank 1
95 → Rank 2
95 → Rank 2
92 → Rank 3
```

##### SQL

```sql
SELECT
    name,
    marks,
    DENSE_RANK() OVER (
        ORDER BY marks DESC
    ) AS rank
FROM students;
```

##### SQLAlchemy

```python
stmt = (
    select(
        Student.name,
        Student.marks,
        func.dense_rank().over(
            order_by=Student.marks.desc()
        ).label("rank")
    )
)

result = session.execute(stmt).all()
```

### Running Total

Calculate the cumulative marks of students.

##### SQL

```sql
SELECT
    name,
    marks,
    SUM(marks) OVER (
        ORDER BY id
    ) AS running_total
FROM students;
```

##### SQLAlchemy

```python
stmt = (
    select(
        Student.name,
        Student.marks,
        func.sum(Student.marks).over(
            order_by=Student.id
        ).label("running_total")
    )
)

result = session.execute(stmt).all()
```

## Summary

| Concept | SQLAlchemy |
|---------|------------|
| Table Alias | `aliased()` |
| Column Alias | `label()` |
| Subquery | `select()` inside another `select()` |
| Scalar Subquery | `.scalar_subquery()` |
| EXISTS | `exists()` |
| Raw SQL | `text()` |
| Window Functions | `.over()` |