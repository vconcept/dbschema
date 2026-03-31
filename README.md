# University Information System Schema Design

This README documents the normalized database schema, SQL table creation, sample data inserts, and example queries for a University Information System.

## Step 1 — Schema Design (Normalized up to 3NF)

We model a University Information System with four main entities.

### 1️ Entities & Relationships

#### Students
Stores student information.

| Column      | Description                  |
|-------------|------------------------------|
| student_id  | Unique student identifier    |
| name        | Student full name            |
| email       | Unique email                 |
| age         | Student age                  |

#### Instructors
Stores lecturer information.

| Column         | Description               |
|----------------|---------------------------|
| instructor_id  | Unique instructor ID      |
| name           | Instructor name           |
| department     | Department name           |

#### Courses
Stores course details.

| Column         | Description                        |
|----------------|------------------------------------|
| course_id      | Unique course ID                   |
| title          | Course name                        |
| credits        | Credit units                       |
| instructor_id  | Instructor teaching course         |

#### Enrollments
Represents the many-to-many relationship between Students and Courses.

| Column      | Description                 |
|-------------|-----------------------------|
| student_id  | FK → Students               |
| course_id   | FK → Courses                |
| grade       | Student grade               |

### 2️ Normalization (Up to 3NF)

-  First Normal Form (1NF)
  - Atomic values ✔
  - No repeating groups ✔
-  Second Normal Form (2NF)
  - Enrollment table uses composite key `(student_id, course_id)`
  - Non-key attribute `grade` depends on full key ✔
-  Third Normal Form (3NF)
  - No transitive dependencies ✔
  - Instructor details stored separately from Courses ✔

### Relationships

- One Instructor → Many Courses
- Many Students ↔ Many Courses (via Enrollments)

## Step 2 — SQL Table Creation

```sql
-- STUDENTS TABLE
CREATE TABLE Students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(120) UNIQUE NOT NULL,
    age INT CHECK (age > 17)
);

-- INSTRUCTORS TABLE
CREATE TABLE Instructors (
    instructor_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(100) NOT NULL
);

-- COURSES TABLE
CREATE TABLE Courses (
    course_id INT PRIMARY KEY,
    title VARCHAR(150) NOT NULL,
    credits INT CHECK (credits BETWEEN 1 AND 6),
    instructor_id INT NOT NULL,
    FOREIGN KEY (instructor_id)
        REFERENCES Instructors(instructor_id)
        ON DELETE CASCADE
);

-- ENROLLMENTS TABLE (junction table)
CREATE TABLE Enrollments (
    student_id INT,
    course_id INT,
    grade CHAR(2),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id)
        REFERENCES Students(student_id)
        ON DELETE CASCADE,
    FOREIGN KEY (course_id)
        REFERENCES Courses(course_id)
        ON DELETE CASCADE
);
```

## Step 3 — Insert Sample Data

### Insert Students

```sql
INSERT INTO Students VALUES
(1, 'Alice Johnson', 'alice@email.com', 20),
(2, 'Bob Smith', 'bob@email.com', 22),
(3, 'Charlie Brown', 'charlie@email.com', 19);
```

### Insert Instructors

```sql
INSERT INTO Instructors VALUES
(101, 'Dr. Adams', 'Computer Science'),
(102, 'Dr. Bello', 'Mathematics'),
(103, 'Dr. Chen', 'Information Systems');
```

### Insert Courses

```sql
INSERT INTO Courses VALUES
(201, 'Database Systems', 3, 101),
(202, 'Calculus II', 4, 102),
(203, 'Web Development', 3, 103);
```

### Insert Enrollments

```sql
INSERT INTO Enrollments VALUES
(1, 201, 'A'),
(1, 203, 'B'),
(2, 201, 'B'),
(3, 202, 'A');
```

## Step 4 — Query Execution

### 1. Retrieve all students enrolled in "Database Systems"

```sql
SELECT s.student_id, s.name
FROM Students s
JOIN Enrollments e ON s.student_id = e.student_id
JOIN Courses c ON e.course_id = c.course_id
WHERE c.title = 'Database Systems';
```

### 2. List all courses with instructor names

```sql
SELECT c.title AS course,
       i.name AS instructor
FROM Courses c
JOIN Instructors i
ON c.instructor_id = i.instructor_id;
```

### 3. Find students NOT enrolled in any course

```sql
SELECT *
FROM Students s
WHERE NOT EXISTS (
    SELECT 1
    FROM Enrollments e
    WHERE e.student_id = s.student_id
);
```

Alternative using `LEFT JOIN`:

```sql
SELECT s.*
FROM Students s
LEFT JOIN Enrollments e
ON s.student_id = e.student_id
WHERE e.student_id IS NULL;
```

### 4. Update a student's email

```sql
UPDATE Students
SET email = 'alice_new@email.com'
WHERE student_id = 1;
```

### 5. Delete a course by ID

```sql
DELETE FROM Courses
WHERE course_id = 203;
```

> `ON DELETE CASCADE` automatically removes related enrollments.
