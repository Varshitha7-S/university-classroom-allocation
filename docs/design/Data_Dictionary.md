# Data Dictionary — UniSpace University Classroom Allocation System

**Database:** `classroom_allocation`  
**DBMS:** PostgreSQL  
**Source:** `database/schema.sql`  
**Related Documents:** [Relational Schema](Relational_Schema.md) · [ER Diagram](ER_Diagram.png)

## 1. Introduction

A data dictionary provides a detailed description of the tables, attributes, data types, constraints and relationships used in the UniSpace University Classroom Allocation System.

The database consists of 11 tables that store information about departments, users, faculty, classrooms, facilities, courses, course offerings, time slots and classroom allocations.

## 2. Table Definitions

### 2.1 DEPARTMENTS

**Purpose:** Stores the departments available in the university.

| Attribute | Data Type | Constraints | Description |
|---|---|---|---|
| department_id | INT | PK, Identity | Unique identifier for each department |
| department_name | VARCHAR(100) | UNIQUE, NOT NULL | Name of the department |

### 2.2 USERS

**Purpose:** Stores login and account information for administrators and faculty members.

| Attribute | Data Type | Constraints | Description |
|---|---|---|---|
| user_id | INT | PK, Identity | Unique identifier for each user |
| name | VARCHAR(100) | NOT NULL | Full name of the user |
| email | VARCHAR(150) | UNIQUE, NOT NULL | Email address used for login |
| password_hash | TEXT | NOT NULL | Hashed password of the user |
| role | VARCHAR(20) | NOT NULL, CHECK | User role: ADMIN or FACULTY |

### 2.3 FACULTY

**Purpose:** Stores faculty-specific information and connects faculty members to their user accounts and departments.

| Attribute | Data Type | Constraints | Description |
|---|---|---|---|
| faculty_id | INT | PK, Identity | Unique identifier for each faculty member |
| user_id | INT | FK, UNIQUE, NOT NULL | References USERS(user_id) |
| department_id | INT | FK, NOT NULL | References DEPARTMENTS(department_id) |

### 2.4 CLASSROOMS

**Purpose:** Stores information about classrooms available for allocation.

| Attribute | Data Type | Constraints | Description |
|---|---|---|---|
| room_id | INT | PK, Identity | Unique identifier for each classroom |
| room_number | VARCHAR(20) | NOT NULL | Room number |
| building | VARCHAR(100) | NOT NULL | Building in which the classroom is located |
| floor | INT | NOT NULL, CHECK | Floor number; must be non-negative |
| capacity | INT | NOT NULL, CHECK | Maximum seating capacity; must be positive |

**Additional Constraint:** `(building, room_number)` must be unique.

### 2.5 FACILITIES

**Purpose:** Stores the facilities and equipment that can be available in classrooms.

| Attribute | Data Type | Constraints | Description |
|---|---|---|---|
| facility_id | INT | PK, Identity | Unique identifier for each facility |
| facility_name | VARCHAR(100) | UNIQUE, NOT NULL | Name of the facility, such as Projector or Smart Board |

### 2.6 CLASSROOM_FACILITIES

**Purpose:** Implements the many-to-many relationship between classrooms and facilities.

| Attribute | Data Type | Constraints | Description |
|---|---|---|---|
| room_id | INT | Composite PK, FK | References CLASSROOMS(room_id) |
| facility_id | INT | Composite PK, FK | References FACILITIES(facility_id) |

**Additional Constraints:**
- Composite Primary Key: `(room_id, facility_id)`.
- Deleting a classroom automatically deletes its associated classroom-facility records through ON DELETE CASCADE.
- Both columns are implicitly NOT NULL because they form the composite primary key.

### 2.7 COURSES

**Purpose:** Stores the courses offered by different university departments.

| Attribute | Data Type | Constraints | Description |
|---|---|---|---|
| course_id | INT | PK, Identity | Unique identifier for each course |
| course_name | VARCHAR(150) | NOT NULL | Name of the course |
| course_code | VARCHAR(20) | UNIQUE, NOT NULL | Unique course code |
| department_id | INT | FK, NOT NULL | References DEPARTMENTS(department_id) |

### 2.8 COURSE_OFFERINGS

**Purpose:** Stores individual course offerings assigned to faculty members for a particular semester and academic year.

| Attribute | Data Type | Constraints | Description |
|---|---|---|---|
| offering_id | INT | PK, Identity | Unique identifier for each course offering |
| course_id | INT | FK, NOT NULL | References COURSES(course_id) |
| faculty_id | INT | FK, NOT NULL | References FACULTY(faculty_id) |
| expected_strength | INT | NOT NULL, CHECK | Expected number of students; must be positive |
| semester | VARCHAR(30) | NOT NULL | Semester in which the course is offered |
| academic_year | VARCHAR(20) | NOT NULL | Academic year of the offering |

### 2.9 OFFERING_REQUIRED_FACILITIES

**Purpose:** Implements the many-to-many relationship between course offerings and the facilities required for conducting them.

| Attribute | Data Type | Constraints | Description |
|---|---|---|---|
| offering_id | INT | Composite PK, FK | References COURSE_OFFERINGS(offering_id) |
| facility_id | INT | Composite PK, FK | References FACILITIES(facility_id) |

**Additional Constraints:**
- Composite Primary Key: `(offering_id, facility_id)`.
- Deleting a course offering automatically deletes its associated required-facility records through ON DELETE CASCADE.
- Both columns are implicitly NOT NULL because they form the composite primary key.

### 2.10 TIME_SLOTS

**Purpose:** Stores the available time slots used for scheduling classroom allocations.

| Attribute | Data Type | Constraints | Description |
|---|---|---|---|
| slot_id | INT | PK, Identity | Unique identifier for each time slot |
| start_time | TIME | NOT NULL | Starting time of the slot |
| end_time | TIME | NOT NULL, CHECK | Ending time of the slot; must be later than start_time |

**Additional Constraint:** `(start_time, end_time)` must be unique.

### 2.11 ALLOCATIONS

**Purpose:** Stores classroom allocation records that connect course offerings, classrooms and time slots for specific days.

| Attribute | Data Type | Constraints | Description |
|---|---|---|---|
| allocation_id | INT | PK, Identity | Unique identifier for each allocation |
| offering_id | INT | FK, NOT NULL | References COURSE_OFFERINGS(offering_id) |
| room_id | INT | FK, NOT NULL | References CLASSROOMS(room_id) |
| slot_id | INT | FK, NOT NULL | References TIME_SLOTS(slot_id) |
| day_of_week | VARCHAR(15) | NOT NULL, CHECK | Day of the allocation: Monday to Saturday |
| allocation_date | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Date and time when the allocation record is created; nullable |

**Additional Constraints:**
- `(room_id, slot_id, day_of_week)` must be unique to prevent duplicate classroom allocations for the same slot and day.
- `(offering_id, slot_id, day_of_week)` must be unique to prevent duplicate allocations of the same offering for the same slot and day.

## 3. Primary Key Summary

| Table | Primary Key |
|---|---|
| DEPARTMENTS | department_id |
| USERS | user_id |
| FACULTY | faculty_id |
| CLASSROOMS | room_id |
| FACILITIES | facility_id |
| CLASSROOM_FACILITIES | room_id, facility_id |
| COURSES | course_id |
| COURSE_OFFERINGS | offering_id |
| OFFERING_REQUIRED_FACILITIES | offering_id, facility_id |
| TIME_SLOTS | slot_id |
| ALLOCATIONS | allocation_id |

## 4. Foreign Key Summary

| Table | Foreign Key | References |
|---|---|---|
| FACULTY | user_id | USERS(user_id) |
| FACULTY | department_id | DEPARTMENTS(department_id) |
| CLASSROOM_FACILITIES | room_id | CLASSROOMS(room_id) |
| CLASSROOM_FACILITIES | facility_id | FACILITIES(facility_id) |
| COURSES | department_id | DEPARTMENTS(department_id) |
| COURSE_OFFERINGS | course_id | COURSES(course_id) |
| COURSE_OFFERINGS | faculty_id | FACULTY(faculty_id) |
| OFFERING_REQUIRED_FACILITIES | offering_id | COURSE_OFFERINGS(offering_id) |
| OFFERING_REQUIRED_FACILITIES | facility_id | FACILITIES(facility_id) |
| ALLOCATIONS | offering_id | COURSE_OFFERINGS(offering_id) |
| ALLOCATIONS | room_id | CLASSROOMS(room_id) |
| ALLOCATIONS | slot_id | TIME_SLOTS(slot_id) |

## 5. Relationship Summary

| Parent Table | Child Table | Relationship |
|---|---|---|
| DEPARTMENTS | FACULTY | One-to-Many |
| DEPARTMENTS | COURSES | One-to-Many |
| USERS | FACULTY | One-to-Zero-or-One |
| COURSES | COURSE_OFFERINGS | One-to-Many |
| FACULTY | COURSE_OFFERINGS | One-to-Many |
| CLASSROOMS | CLASSROOM_FACILITIES | One-to-Many |
| FACILITIES | CLASSROOM_FACILITIES | One-to-Many |
| COURSE_OFFERINGS | OFFERING_REQUIRED_FACILITIES | One-to-Many |
| FACILITIES | OFFERING_REQUIRED_FACILITIES | One-to-Many |
| CLASSROOMS | ALLOCATIONS | One-to-Many |
| COURSE_OFFERINGS | ALLOCATIONS | One-to-Many |
| TIME_SLOTS | ALLOCATIONS | One-to-Many |

`CLASSROOM_FACILITIES` and `OFFERING_REQUIRED_FACILITIES` are associative tables that implement many-to-many relationships.

## 6. Database Objects

In addition to the 11 base tables, the database uses the following advanced objects.

| Object Name | Object Type | Purpose |
|---|---|---|
| classroom_timetable | View | Provides combined timetable information |
| classroom_utilization | View | Provides classroom utilization information |
| allocate_classroom | Stored Procedure | Performs classroom allocation |
| get_classroom_utilization | Function | Calculates classroom utilization |
| validate_classroom_allocation | Trigger Function | Validates allocation rules |
| trg_validate_classroom_allocation | Trigger | Executes validation before allocation INSERT or UPDATE |

## 7. Indexes

The database includes additional indexes to improve query performance.

| Index Name | Table | Columns |
|---|---|---|
| idx_allocations_room_day_slot | allocations | room_id, day_of_week, slot_id |
| idx_allocations_offering | allocations | offering_id |
| idx_allocations_slot | allocations | slot_id |
| idx_course_offerings_faculty | course_offerings | faculty_id |
| idx_courses_department | courses | department_id |
| idx_classrooms_capacity | classrooms | capacity |

Primary-key and unique constraints also create their associated indexes automatically.

## 8. Conclusion

The UniSpace Data Dictionary documents the structure and purpose of all 11 database tables, including their attributes, data types, primary keys, foreign keys, uniqueness constraints and domain constraints.

It serves as a reference for understanding the database design and supports database development, maintenance and documentation.
