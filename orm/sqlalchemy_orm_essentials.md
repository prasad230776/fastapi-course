# SQLAlchemy ORM Essentials

A concise reference guide explaining Object-Relational Mapping (ORM) using Python's **SQLAlchemy** library. It demonstrates how to map Python classes to database tables, execute CRUD operations, write queries, and model relationships.


## 1. What is an ORM?

An **ORM (Object-Relational Mapper)** is a library that acts as a translator between two worlds:
*   **Python World:** Deals with classes, objects, attributes, and lists.
*   **Database World:** Deals with tables, rows, columns, and SQL syntax.

| Database Concepts | Python (ORM) Concepts |
| :--- | :--- |
| Table | Class |
| Row / Record | Object Instance |
| Column / Attribute | Class Attribute |


## 2. SQLAlchemy Core Workflow & Building Blocks

Every SQLAlchemy application sets up the database connection and operations in a structured pipeline:

```
Database URL (Connection details)
      │
      ▼
Engine (Manages connections/pools) -> create_engine()
      │
      ▼
DeclarativeBase (Registry for models) -> class Base(DeclarativeBase)
      │
      ▼
Metadata (Stores schemas) & Create Tables -> Base.metadata.create_all()
      │
      ▼
Session Factory & Session (Workspace for CRUD) -> sessionmaker() -> Session
```

### Initial Configuration Example
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker

DATABASE_URL = "sqlite:///students.db"

# 1. Engine
engine = create_engine(DATABASE_URL)

# 2. Declarative Base
class Base(DeclarativeBase):
    pass

# 3. Session Factory
SessionLocal = sessionmaker(bind=engine)
```


## 3. Defining Models & Column Configuration

Models represent tables. We use type annotations with `Mapped` and configure columns using `mapped_column()`.

```python
from datetime import datetime
from decimal import Decimal
from sqlalchemy import String, Numeric, DateTime
from sqlalchemy.orm import Mapped, mapped_column

class Student(Base):
    __tablename__ = "students"  # Database table name

    # Columns
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100), nullable=False)
    age: Mapped[int] = mapped_column(default=18)
    email: Mapped[str] = mapped_column(String(100), unique=True, index=True)
    fee: Mapped[Decimal] = mapped_column(Numeric(10, 2))
    created_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.utcnow)
```

### Type Mapping & Constraints
*   `Mapped[Python_Type]`: Defines the Python type. SQLAlchemy infers standard DB types (e.g., `int` -> `Integer`, `str` -> `String`).
*   `mapped_column(SQLAlchemy_Type)`: Explicitly defines column configuration (e.g., `String(100)`, `Numeric(10,2)`).
*   **Constraints:** `primary_key=True`, `nullable=False`, `default=val`, `unique=True`, `index=True` (for search performance).


## 4. Basic CRUD Operations

Database transactions are managed inside a **Session**.

### Create (Insert)
```python
session = SessionLocal()

new_student = Student(name="John Doe", age=20, email="john@example.com")
session.add(new_student)  # Start tracking
session.commit()          # Save to database
```

### Read (Select)
*   **Get by Primary Key:**
    ```python
    student = session.get(Student, 1)  # Returns object or None
    ```
*   **Get All Matching Rows:**
    ```python
    from sqlalchemy import select

    stmt = select(Student).where(Student.age >= 21)
    result = session.execute(stmt)        # Execute query
    students = result.scalars().all()     # Convert result to Python list
    ```

### Update
*   **Object-Based Update (Recommended):**
    ```python
    student = session.get(Student, 1)
    student.age = 22       # Modify attribute directly
    session.commit()       # Saves changes
    ```
*   **Direct Query Update (Bulk):**
    ```python
    from sqlalchemy import update
    
    stmt = update(Student).where(Student.id == 1).values(age=22)
    session.execute(stmt)
    session.commit()
    ```

### Delete
```python
# Object-Based Delete
student = session.get(Student, 1)
session.delete(student)
session.commit()
```


## 5. Writing Select Queries (SQL vs. SQLAlchemy ORM)

Below is a syntax map of how common SQL queries translate into SQLAlchemy:

| Feature | SQL | SQLAlchemy ORM Query |
| :--- | :--- | :--- |
| **All Rows** | `SELECT * FROM students;` | `select(Student)` |
| **Where Filter** | `WHERE age >= 21` | `.where(Student.age >= 21)` |
| **Logical AND** | `WHERE city = 'A' AND marks >= 80` | `.where(Student.city == 'A', Student.marks >= 80)` |
| **Logical OR** | `WHERE city = 'A' OR city = 'B'` | `or_(Student.city == 'A', Student.city == 'B')` |
| **IN List** | `WHERE city IN ('Delhi', 'Mumbai')` | `.where(Student.city.in_(['Delhi', 'Mumbai']))` |
| **BETWEEN** | `WHERE marks BETWEEN 80 AND 90` | `.where(Student.marks.between(80, 90))` |
| **Pattern Match** | `WHERE name LIKE 'K%'` | `.where(Student.name.like('K%'))` |
| **Case-Insensitive**| `WHERE LOWER(name) LIKE '%ra%'` | `.where(Student.name.ilike('%ra%'))` |
| **Sorting** | `ORDER BY marks DESC` | `.order_by(Student.marks.desc())` |
| **Pagination** | `LIMIT 5 OFFSET 10` | `.limit(5).offset(10)` |

### Result Extraction Methods
*   `execute(stmt)`: Runs the statement. Returns a raw `Result` object.
*   `scalars()`: Extracts the main model objects (removes database wrapping).
*   `all()`: Fetches all matched records as a Python list.
*   `first()`: Returns the first record, or `None` if empty.
*   `one()`: Returns exactly one record (raises an error if 0 or 2+ matched).
*   `one_or_none()`: Returns one record or `None` (raises error if 2+ matched).


## 6. Table Relationships

Relationships link Python classes together, allowing easy navigation (e.g., `student.course`).

### One-to-Many / Many-to-One (1:N)
A course has many students; each student has one course.
```python
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

class Course(Base):
    __tablename__ = "courses"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    
    # Backref mapping (returns a Python list)
    students: Mapped[list["Student"]] = relationship(back_populates="course")

class Student(Base):
    __tablename__ = "students"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    course_id: Mapped[int] = mapped_column(ForeignKey("courses.id"))
    
    # Python object relationship (returns a single Course object)
    course: Mapped["Course"] = relationship(back_populates="students")
```

### One-to-One (1:1)
An employee has exactly one parking space.
```python
class Employee(Base):
    __tablename__ = "employees"
    id: Mapped[int] = mapped_column(primary_key=True)
    
    # uselist=False forces it to return a single object, not a list
    parking_space: Mapped["ParkingSpace"] = relationship(back_populates="employee", uselist=False)

class ParkingSpace(Base):
    __tablename__ = "parking_spaces"
    id: Mapped[int] = mapped_column(primary_key=True)
    employee_id: Mapped[int] = mapped_column(ForeignKey("employees.id"), unique=True) # unique=True prevents multiple links
    
    employee: Mapped["Employee"] = relationship(back_populates="parking_space")
```

### Many-to-Many (M:N)
Students enroll in many courses; courses have many students. Requires an **Association Table**.
```python
from sqlalchemy import Table, Column

# Association/Junction Table (Database-level only)
student_courses = Table(
    "student_courses",
    Base.metadata,
    Column("student_id", ForeignKey("students.id"), primary_key=True),
    Column("course_id", ForeignKey("courses.id"), primary_key=True),
)

class Student(Base):
    __tablename__ = "students"
    id: Mapped[int] = mapped_column(primary_key=True)
    
    # Map relationship via secondary association table
    courses: Mapped[list["Course"]] = relationship(secondary=student_courses, back_populates="students")

class Course(Base):
    __tablename__ = "courses"
    id: Mapped[int] = mapped_column(primary_key=True)
    
    students: Mapped[list["Student"]] = relationship(secondary=student_courses, back_populates="courses")
```
