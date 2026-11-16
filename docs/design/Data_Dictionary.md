# Data Dictionary — UniSpace University Classroom Allocation System

**DBMS:** PostgreSQL  
**Source:** `database/schema.sql`  
**Related Documents:** [Relational Schema](Relational_Schema.md) · [ER Diagram](ER_Diagram.png)

**Key Legend:**
- PK – Primary Key
- FK – Foreign Key
- UK – Unique Key
- NN – NOT NULL
- CK – CHECK Constraint
- IDENTITY – Automatically generated identifier

## 1. Introduction

The UniSpace University Classroom Allocation System uses PostgreSQL to manage academic departments, users, faculty, classrooms, facilities, courses, course offerings, time slots and classroom allocations.

This data dictionary documents the database tables, their attributes, data types, constraints and purposes. It also describes the views, stored procedure, functions, triggers and indexes used by the system.

## 2. Table Definitions

### 2.1 Departments

**Purpose:** Stores academic departments within the university.

| Column | Data Type | Constraints | Description |
|---|---|---|---|
| department_id | INT | PK, IDENTITY | Unique department identifier |
| department_name | VARCHAR(100) | UK, NN | Name of the department |

### 2.2 Users

**Purpose:** Stores user accounts and authentication information.

| Column | Data Type | Constraints | Description |
|---|---|---|---|
| user_id | INT | PK, IDENTITY | Unique user identifier |
| name | VARCHAR(100) | NN | Full name of the user |
| email | VARCHAR(150) | UK, NN | Unique login email address |
| password_hash | TEXT | NN | Hashed password |
| role | VARCHAR(20) | NN, CK | User role: ADMIN or FACULTY |

### 2.3 Faculty

**Purpose:** Stores faculty profiles and their department associations.

| Column | Data Type | Constraints | Description |
|---|---|---|---|
| faculty_id | INT | PK, IDENTITY | Unique faculty identifier |
| user_id | INT | FK, UK, NN | References users(user_id) |
| department_id | INT | FK, NN | References departments(department_id) |

### 2.4 Classrooms

**Purpose:** Stores classroom information, location and seating capacity.

| Column | Data Type | Constraints | Description |
|---|---|---|---|
| room_id | INT | PK, IDENTITY | Unique classroom identifier |
| room_number | VARCHAR(20) | NN | Classroom number |
| building | VARCHAR(100) | NN | Building containing the classroom |
| floor | INT | NN, CK (>= 0) | Floor number |
| capacity | INT | NN, CK (> 0) | Maximum classroom seating capacity |

**Additional constraint:** `(building, room_number)` must be unique.

### 2.5 Facilities

**Purpose:** Maintains the list of available classroom facilities.

| Column | Data Type | Constraints | Description |
|---|---|---|---|
| facility_id | INT | PK, IDENTITY | Unique facility identifier |
| facility_name | VARCHAR(100) | UK, NN | Name of the facility |

### 2.6 Classroom_Facilities

**Purpose:** Implements the many-to-many relationship between classrooms and facilities.

| Column | Data Type | Constraints | Description |
|---|---|---|---|
| room_id | INT | PK (part), FK | References classrooms(room_id) |
| facility_id | INT | PK (part), FK | References facilities(facility_id) |

**Additional constraints:**
- Composite primary key: `(room_id, facility_id)`.
- Deleting a classroom cascades to its classroom-facility associations.
- The facility foreign key does not specify ON DELETE CASCADE.

### 2.7 Courses

**Purpose:** Stores course details and their associated departments.

| Column | Data Type | Constraints | Description |
|---|---|---|---|
| course_id | INT | PK, IDENTITY | Unique course identifier |
| course_name | VARCHAR(150) | NN | Name of the course |
| course_code | VARCHAR(20) | UK, NN | Unique course code |
| department_id | INT | FK, NN | References departments(department_id) |

### 2.8 Course_Offerings

**Purpose:** Stores courses offered by faculty members for specific semesters and academic years.

| Column | Data Type | Constraints | Description |
|---|---|---|---|
| offering_id | INT | PK, IDENTITY | Unique course offering identifier |
| course_id | INT | FK, NN | References courses(course_id) |
| faculty_id | INT | FK, NN | References faculty(faculty_id) |
| expected_strength | INT | NN, CK (> 0) | Expected number of students |
| semester | VARCHAR(30) | NN | Academic semester |
| academic_year | VARCHAR(20) | NN | Academic year |

### 2.9 Offering_Required_Facilities

**Purpose:** Stores the facilities required by individual course offerings.

| Column | Data Type | Constraints | Description |
|---|---|---|---|
| offering_id | INT | PK (part), FK | References course_offerings(offering_id) |
| facility_id | INT | PK (part), FK | References facilities(facility_id) |

**Additional constraints:**
- Composite primary key: `(offering_id, facility_id)`.
- Deleting a course offering cascades to its required-facility associations.
- The facility foreign key does not specify ON DELETE CASCADE.

### 2.10 Time_Slots

**Purpose:** Stores the available time intervals for classroom scheduling.

| Column | Data Type | Constraints | Description |
|---|---|---|---|
| slot_id | INT | PK, IDENTITY | Unique time-slot identifier |
| start_time | TIME | NN | Starting time of the slot |
| end_time | TIME | NN, CK | Ending time of the slot |

**Additional constraints:**
- `end_time > start_time`.
- `(start_time, end_time)` must be unique.

### 2.11 Allocations

**Purpose:** Stores classroom allocation records connecting course offerings, classrooms and time slots.

| Column | Data Type | Constraints | Description |
|---|---|---|---|
| allocation_id | INT | PK, IDENTITY | Unique allocation identifier |
| offering_id | INT | FK, NN | References course_offerings(offering_id) |
| room_id | INT | FK, NN | References classrooms(room_id) |
| slot_id | INT | FK, NN | References time_slots(slot_id) |
| day_of_week | VARCHAR(15) | NN, CK | Scheduled day from Monday to Saturday |
| allocation_date | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Date and time when allocation record is created |

**Additional constraints:**
- `(room_id, slot_id, day_of_week)` must be unique.
- `(offering_id, slot_id, day_of_week)` must be unique.
- The day must be Monday, Tuesday, Wednesday, Thursday, Friday or Saturday.

## 3. Relationship Summary

| Parent Table | Child Table | Relationship |
|---|---|---|
| Departments | Faculty | One-to-Many |
| Departments | Courses | One-to-Many |
| Users | Faculty | One-to-One (optional faculty record) |
| Courses | Course_Offerings | One-to-Many |
| Faculty | Course_Offerings | One-to-Many |
| Classrooms | Classroom_Facilities | One-to-Many |
| Facilities | Classroom_Facilities | One-to-Many |
| Course_Offerings | Offering_Required_Facilities | One-to-Many |
| Facilities | Offering_Required_Facilities | One-to-Many |
| Classrooms | Allocations | One-to-Many |
| Course_Offerings | Allocations | One-to-Many |
| Time_Slots | Allocations | One-to-Many |

The junction tables `classroom_facilities` and `offering_required_facilities` implement the many-to-many relationships.

## 4. Views

### 4.1 classroom_timetable

**Purpose:** Provides a readable timetable by joining allocation, course, faculty, classroom and time-slot information.

| Column | Source | Description |
|---|---|---|
| allocation_id | allocations | Allocation identifier |
| day_of_week | allocations | Scheduled weekday |
| course_code | courses | Course code |
| course_name | courses | Course title |
| faculty_name | users.name | Teaching faculty member |
| room_number | classrooms | Classroom number |
| building | classrooms | Building name |
| capacity | classrooms | Classroom capacity |
| expected_strength | course_offerings | Expected student strength |
| start_time | time_slots | Slot start time |
| end_time | time_slots | Slot end time |

### 4.2 classroom_utilization

**Purpose:** Calculates the number of allocations and average utilization percentage for every classroom, including classrooms without allocations.

| Column | Source | Description |
|---|---|---|
| room_id | classrooms | Classroom identifier |
| room_number | classrooms | Classroom number |
| building | classrooms | Building name |
| capacity | classrooms | Classroom capacity |
| total_allocations | COUNT(allocation_id) | Total allocations for the classroom |
| average_utilization_percentage | Calculated | Average student-strength-to-capacity percentage, rounded to two decimal places; zero when no allocations exist |

**Verification note:** The `advanced.sql` definition names the final view column `average_utilization_percentage`, while the live PostgreSQL query displayed `average_capacity_utilization`. Confirm the deployed view definition before treating these column names as identical.

## 5. Stored Procedure and Functions

| Name | Type | Parameters | Returns | Description |
|---|---|---|---|---|
| `allocate_classroom` | Stored Procedure | `p_offering_id INTEGER`, `p_room_id INTEGER`, `p_slot_id INTEGER`, `p_day_of_week VARCHAR(20)` | None | Inserts a classroom allocation; trigger validation applies |
| `get_classroom_utilization` | Function | `p_room_id INTEGER` | NUMERIC | Returns the average utilization percentage for a specified classroom |

### Trigger Function

| Name | Type | Returns | Description |
|---|---|---|---|
| `validate_classroom_allocation` | Trigger Function | TRIGGER | Validates allocation capacity, facilities and scheduling conflicts before allocation changes |

## 6. Trigger

### trg_validate_classroom_allocation

| Property | Definition |
|---|---|
| Trigger name | trg_validate_classroom_allocation |
| Table | allocations |
| Timing | BEFORE |
| Events | INSERT OR UPDATE |
| Execution | FOR EACH ROW |
| Function | validate_classroom_allocation() |

**Purpose:** Enforces business rules before an allocation is inserted or updated.

The trigger performs the following validations:

1. Checks whether the course offering exists.
2. Checks whether the classroom exists.
3. Checks whether the time slot exists.
4. Verifies that classroom capacity is sufficient for expected student strength.
5. Verifies that all facilities required by the offering are available in the classroom.
6. Prevents overlapping allocations in the same classroom on the same day.
7. Prevents overlapping allocations for the same faculty member on the same day.
8. Uses `pg_advisory_xact_lock(20261115, 1)` to serialize allocation writes during validation.

### Trigger Error Messages

| Condition | Error Message |
|---|---|
| Offering not found | Course offering does not exist |
| Classroom not found | Classroom does not exist |
| Time slot not found | Time slot does not exist |
| Insufficient capacity | Insufficient classroom capacity. Required: n, Available: m |
| Missing facilities | Classroom does not have all facilities required by this course |
| Room conflict | Classroom is already allocated during this time |
| Faculty conflict | Faculty member already has a class during this time |

## 7. Indexes

Primary-key and UNIQUE constraints create their associated indexes automatically.

The following additional indexes are defined in `database/advanced.sql`.

| Index | Table | Columns | Purpose |
|---|---|---|---|
| idx_allocations_room_day_slot | allocations | room_id, day_of_week, slot_id | Room availability and conflict checks |
| idx_allocations_offering | allocations | offering_id | Allocation lookups by offering |
| idx_allocations_slot | allocations | slot_id | Joins with time slots |
| idx_course_offerings_faculty | course_offerings | faculty_id | Faculty timetable and conflict checks |
| idx_courses_department | courses | department_id | Listing courses by department |
| idx_classrooms_capacity | classrooms | capacity | Finding classrooms by minimum capacity |

## 8. Sample Data and Database Verification

The initial seed script populates the database with the following records.

| Table | Initial Seed Records |
|---|---:|
| departments | 3 |
| users | 3 |
| faculty | 2 |
| classrooms | 5 |
| facilities | 4 |
| courses | 4 |
| course_offerings | 4 |
| time_slots | 6 |
| allocations | 0 |

The seed script does not insert any allocation records.

### Current Database State

A live PostgreSQL query was executed to verify the current allocation count and classroom utilization.

**Current allocation count: 1**

| Classroom | Capacity | Total Allocations | Utilization |
|---|---:|---:|---:|
| A101 | 40 | 0 | 0% |
| A102 | 70 | 1 | 64.29% |
| A103 | 100 | 0 | 0% |
| B201 | 80 | 0 | 0% |
| B202 | 60 | 0 | 0% |

The current database therefore contains one allocation associated with classroom A102. Its utilization is 64.29%.

## 9. Data Integrity and Validation

The database maintains consistency through:

1. **Entity Integrity:** Primary keys uniquely identify records.
2. **Referential Integrity:** Foreign keys maintain relationships between tables.
3. **Domain Integrity:** CHECK constraints restrict invalid values.
4. **Uniqueness:** UNIQUE constraints prevent duplicate values and allocation combinations.
5. **Automatic Identification:** Identity columns generate identifiers automatically.
6. **Default Values:** Allocation timestamps use `CURRENT_TIMESTAMP` when no value is supplied.
7. **Business Rule Validation:** Triggers validate capacity, facilities and scheduling conflicts.

## 10. Conclusion

The UniSpace data dictionary provides a comprehensive reference for all 11 relational tables and their associated database objects.

It documents the attributes, data types, constraints, relationships, views, stored procedures, functions, triggers and indexes used to implement the university classroom allocation system.

**Implementation references:**
- `database/schema.sql`
- `database/seed.sql`
- `database/advanced.sql`
- `database/functions.sql`
- `database/procedures.sql`
- `database/triggers.sql`
