# Data Dictionary – UniSpace

## 1. Introduction

The data dictionary describes the attributes, data types, constraints, and purpose of each relation in the UniSpace University Classroom Allocation System.

**Notation:**
- PK – Primary Key
- FK – Foreign Key
- UQ – Unique
- NN – NOT NULL
- CK – Check Constraint

## 2. Table Definitions

### 2.1 DEPARTMENTS

| Attribute | Data type | Constraints | Description |
|---|---|---|---|
| department_id | INT | PK, GENERATED ALWAYS AS IDENTITY | Unique department identifier |
| department_name | VARCHAR(100) | NN, UQ | Name of the department |

### 2.2 USERS

| Attribute | Data type | Constraints | Description |
|---|---|---|---|
| user_id | INT | PK, GENERATED ALWAYS AS IDENTITY | Unique user identifier |
| name | VARCHAR(100) | NN | User's full name |
| email | VARCHAR(150) | NN, UQ | Unique email address |
| password_hash | TEXT | NN | Hashed password |
| role | VARCHAR(20) | NN, CK | User role: ADMIN or FACULTY |

### 2.3 FACULTY

| Attribute | Data type | Constraints | Description |
|---|---|---|---|
| faculty_id | INT | PK, GENERATED ALWAYS AS IDENTITY | Unique faculty identifier |
| user_id | INT | NN, FK, UQ | References the associated user account |
| department_id | INT | NN, FK | References the faculty member's department |

### 2.4 CLASSROOMS

| Attribute | Data type | Constraints | Description |
|---|---|---|---|
| room_id | INT | PK, GENERATED ALWAYS AS IDENTITY | Unique classroom identifier |
| room_number | VARCHAR(20) | NN | Classroom number |
| building | VARCHAR(100) | NN | Building name |
| floor | INT | NN, CK (floor >= 0) | Floor on which the classroom is located |
| capacity | INT | NN, CK (capacity > 0) | Maximum seating capacity |
| (building, room_number) | — | UQ | Prevents duplicate room numbers within a building |

### 2.5 FACILITIES

| Attribute | Data type | Constraints | Description |
|---|---|---|---|
| facility_id | INT | PK, GENERATED ALWAYS AS IDENTITY | Unique facility identifier |
| facility_name | VARCHAR(100) | NN, UQ | Name of the facility |

### 2.6 CLASSROOM_FACILITIES

| Attribute | Data type | Constraints | Description |
|---|---|---|---|
| room_id | INT | PK, FK, NN | References CLASSROOMS; cascades on classroom deletion |
| facility_id | INT | PK, FK, NN | References FACILITIES |

The composite primary key `(room_id, facility_id)` prevents the same facility from being assigned more than once to the same classroom.

### 2.7 COURSES

| Attribute | Data type | Constraints | Description |
|---|---|---|---|
| course_id | INT | PK, GENERATED ALWAYS AS IDENTITY | Unique course identifier |
| course_name | VARCHAR(150) | NN | Name of the course |
| course_code | VARCHAR(20) | NN, UQ | Unique course code |
| department_id | INT | NN, FK | Department offering the course |

### 2.8 COURSE_OFFERINGS

| Attribute | Data type | Constraints | Description |
|---|---|---|---|
| offering_id | INT | PK, GENERATED ALWAYS AS IDENTITY | Unique course offering identifier |
| course_id | INT | NN, FK | References the course |
| faculty_id | INT | NN, FK | References the assigned faculty member |
| expected_strength | INT | NN, CK (expected_strength > 0) | Expected number of students |
| semester | VARCHAR(30) | NN | Semester in which the course is offered |
| academic_year | VARCHAR(20) | NN | Academic year of the offering |

### 2.9 OFFERING_REQUIRED_FACILITIES

| Attribute | Data type | Constraints | Description |
|---|---|---|---|
| offering_id | INT | PK, FK, NN | References COURSE_OFFERINGS; cascades on offering deletion |
| facility_id | INT | PK, FK, NN | References FACILITIES |

The composite primary key `(offering_id, facility_id)` prevents duplicate facility requirements for the same offering.

### 2.10 TIME_SLOTS

| Attribute | Data type | Constraints | Description |
|---|---|---|---|
| slot_id | INT | PK, GENERATED ALWAYS AS IDENTITY | Unique time slot identifier |
| start_time | TIME | NN | Starting time of the slot |
| end_time | TIME | NN, CK (end_time > start_time) | Ending time of the slot |
| (start_time, end_time) | — | UQ | Prevents duplicate time-slot ranges |

### 2.11 ALLOCATIONS

| Attribute | Data type | Constraints | Description |
|---|---|---|---|
| allocation_id | INT | PK, GENERATED ALWAYS AS IDENTITY | Unique allocation identifier |
| offering_id | INT | NN, FK | References COURSE_OFFERINGS |
| room_id | INT | NN, FK | References CLASSROOMS |
| slot_id | INT | NN, FK | References TIME_SLOTS |
| day_of_week | VARCHAR(15) | NN, CK | Day of the week: Monday to Saturday |
| allocation_date | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP, nullable | Date and time when the allocation record is created |
| (room_id, slot_id, day_of_week) | — | UQ | Prevents the same classroom from being allocated twice for the same slot and day |
| (offering_id, slot_id, day_of_week) | — | UQ | Prevents the same offering from being allocated twice for the same slot and day |

## 3. Foreign Key Reference Summary

| Foreign key | Referenced table | Referenced attribute | Delete action |
|---|---|---|---|
| FACULTY.user_id | USERS | user_id | NO ACTION |
| FACULTY.department_id | DEPARTMENTS | department_id | NO ACTION |
| CLASSROOM_FACILITIES.room_id | CLASSROOMS | room_id | CASCADE |
| CLASSROOM_FACILITIES.facility_id | FACILITIES | facility_id | NO ACTION |
| COURSES.department_id | DEPARTMENTS | department_id | NO ACTION |
| COURSE_OFFERINGS.course_id | COURSES | course_id | NO ACTION |
| COURSE_OFFERINGS.faculty_id | FACULTY | faculty_id | NO ACTION |
| OFFERING_REQUIRED_FACILITIES.offering_id | COURSE_OFFERINGS | offering_id | CASCADE |
| OFFERING_REQUIRED_FACILITIES.facility_id | FACILITIES | facility_id | NO ACTION |
| ALLOCATIONS.offering_id | COURSE_OFFERINGS | offering_id | NO ACTION |
| ALLOCATIONS.room_id | CLASSROOMS | room_id | NO ACTION |
| ALLOCATIONS.slot_id | TIME_SLOTS | slot_id | NO ACTION |

## 4. Database Objects

| Object type | Object name | Purpose |
|---|---|---|
| View | classroom_timetable | Presents classroom timetable information |
| View | classroom_utilization | Provides classroom utilization information |
| Stored procedure | allocate_classroom | Performs classroom allocation operations |
| Function | get_classroom_utilization | Retrieves classroom utilization information |
| Trigger function | validate_classroom_allocation | Validates allocation rules |
| Trigger | trg_validate_classroom_allocation | Executes validation before INSERT or UPDATE on ALLOCATIONS |

## 5. Indexes

| Index name | Table | Purpose |
|---|---|---|
| idx_allocations_room_day_slot | ALLOCATIONS | Supports room, day, and slot lookups |
| idx_allocations_offering | ALLOCATIONS | Supports offering-based allocation lookups |
| idx_allocations_slot | ALLOCATIONS | Supports time-slot lookups |
| idx_course_offerings_faculty | COURSE_OFFERINGS | Supports faculty-based offering lookups |
| idx_courses_department | COURSES | Supports department-based course lookups |
| idx_classrooms_capacity | CLASSROOMS | Supports classroom capacity filtering |

## 6. Data Integrity Rules

The database enforces the following rules:

1. Every primary key uniquely identifies a record.
2. Foreign keys maintain relationships between tables.
3. Email addresses, department names, facility names, course codes, and specified combinations are unique.
4. User roles are restricted to ADMIN and FACULTY.
5. Classroom floor numbers cannot be negative.
6. Classroom capacity and expected student strength must be greater than zero.
7. A time slot must have an ending time later than its starting time.
8. Allocation days are restricted to Monday through Saturday.
9. A classroom cannot be assigned to two allocations for the same time slot and day.
10. A course offering cannot be assigned to two allocations for the same time slot and day.
11. The allocation trigger performs additional validation of capacity, required facilities, and scheduling conflicts.

## 7. Conclusion

The data dictionary documents all 11 relations in the UniSpace database, including their attributes, data types, keys, constraints, and relationships. It serves as a reference for database implementation, maintenance, and further development.
