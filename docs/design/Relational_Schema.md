# Relational Schema — UniSpace University Classroom Allocation System

**DBMS:** PostgreSQL
**Source:** `database/schema.sql`
**Related Documents:** [ER Diagram](ER_Diagram.png) · [Editable ER Diagram](ER_Diagram.drawio) · [Data Dictionary](Data_Dictionary.md)

## 1. Introduction

The UniSpace University Classroom Allocation System uses a relational database to manage university departments, users, faculty, classrooms, facilities, courses, course offerings, time slots and classroom allocations.

The database consists of 11 relations connected through primary keys and foreign keys.

### Notation

- **PK:** Primary Key
- **FK:** Foreign Key
- **UK:** Unique Key
- **NN:** NOT NULL
- **CK:** CHECK Constraint
- Composite primary keys are represented using `(A, B)`.

## 2. Relational Schema

### 2.1 Departments

`DEPARTMENTS(department_id, department_name)`

- PK: `department_id`
- UK: `department_name`

### 2.2 Users

`USERS(user_id, name, email, password_hash, role)`

- PK: `user_id`
- UK: `email`
- CK: `role IN ('ADMIN', 'FACULTY')`

### 2.3 Faculty

`FACULTY(faculty_id, user_id, department_id)`

- PK: `faculty_id`
- FK: `user_id` → `USERS(user_id)`
- FK: `department_id` → `DEPARTMENTS(department_id)`
- UK: `user_id`

### 2.4 Classrooms

`CLASSROOMS(room_id, room_number, building, floor, capacity)`

- PK: `room_id`
- UK: `(building, room_number)`
- CK: `floor >= 0`
- CK: `capacity > 0`

### 2.5 Facilities

`FACILITIES(facility_id, facility_name)`

- PK: `facility_id`
- UK: `facility_name`

### 2.6 Classroom Facilities

`CLASSROOM_FACILITIES(room_id, facility_id)`

- Composite PK: `(room_id, facility_id)`
- FK: `room_id` → `CLASSROOMS(room_id)`
- FK: `facility_id` → `FACILITIES(facility_id)`
- ON DELETE CASCADE applies to `room_id`.

### 2.7 Courses

`COURSES(course_id, course_name, course_code, department_id)`

- PK: `course_id`
- UK: `course_code`
- FK: `department_id` → `DEPARTMENTS(department_id)`

### 2.8 Course Offerings

`COURSE_OFFERINGS(offering_id, course_id, faculty_id, expected_strength, semester, academic_year)`

- PK: `offering_id`
- FK: `course_id` → `COURSES(course_id)`
- FK: `faculty_id` → `FACULTY(faculty_id)`
- CK: `expected_strength > 0`

### 2.9 Offering Required Facilities

`OFFERING_REQUIRED_FACILITIES(offering_id, facility_id)`

- Composite PK: `(offering_id, facility_id)`
- FK: `offering_id` → `COURSE_OFFERINGS(offering_id)`
- FK: `facility_id` → `FACILITIES(facility_id)`
- ON DELETE CASCADE applies to `offering_id`.

### 2.10 Time Slots

`TIME_SLOTS(slot_id, start_time, end_time)`

- PK: `slot_id`
- UK: `(start_time, end_time)`
- CK: `end_time > start_time`

### 2.11 Allocations

`ALLOCATIONS(allocation_id, offering_id, room_id, slot_id, day_of_week, allocation_date)`

- PK: `allocation_id`
- FK: `offering_id` → `COURSE_OFFERINGS(offering_id)`
- FK: `room_id` → `CLASSROOMS(room_id)`
- FK: `slot_id` → `TIME_SLOTS(slot_id)`
- UK: `(room_id, slot_id, day_of_week)`
- UK: `(offering_id, slot_id, day_of_week)`
- CK: `day_of_week` must be Monday, Tuesday, Wednesday, Thursday, Friday or Saturday.

## 3. Foreign Key Relationships

| Child Relation               | Foreign Key   | Parent Relation  | On Delete |
| ---------------------------- | ------------- | ---------------- | --------- |
| faculty                      | user_id       | users            | NO ACTION |
| faculty                      | department_id | departments      | NO ACTION |
| classroom_facilities         | room_id       | classrooms       | CASCADE   |
| classroom_facilities         | facility_id   | facilities       | NO ACTION |
| courses                      | department_id | departments      | NO ACTION |
| course_offerings             | course_id     | courses          | NO ACTION |
| course_offerings             | faculty_id    | faculty          | NO ACTION |
| offering_required_facilities | offering_id   | course_offerings | CASCADE   |
| offering_required_facilities | facility_id   | facilities       | NO ACTION |
| allocations                  | offering_id   | course_offerings | NO ACTION |
| allocations                  | room_id       | classrooms       | NO ACTION |
| allocations                  | slot_id       | time_slots       | NO ACTION |

`NO ACTION` is the default PostgreSQL foreign-key delete behaviour in the absence of an explicit delete action.

## 4. Relationship Cardinalities

| Relationship                   | Cardinality | Implementation                 |
| ------------------------------ | ----------- | ------------------------------ |
| Departments – Faculty          | 1 : N       | `faculty.department_id`        |
| Departments – Courses          | 1 : N       | `courses.department_id`        |
| Users – Faculty                | 1 : 0..1    | UNIQUE FK `faculty.user_id`    |
| Courses – Course Offerings     | 1 : N       | `course_offerings.course_id`   |
| Faculty – Course Offerings     | 1 : N       | `course_offerings.faculty_id`  |
| Classrooms – Facilities        | M : N       | `classroom_facilities`         |
| Course Offerings – Facilities  | M : N       | `offering_required_facilities` |
| Classrooms – Allocations       | 1 : N       | `allocations.room_id`          |
| Course Offerings – Allocations | 1 : N       | `allocations.offering_id`      |
| Time Slots – Allocations       | 1 : N       | `allocations.slot_id`          |

The two many-to-many relationships are implemented using associative relations with composite primary keys.

The `allocations` relation connects a course offering, classroom and time slot for a particular day of the week.

## 5. Integrity Constraints

### 5.1 Entity Integrity

Every relation has a primary key that uniquely identifies its records.

### 5.2 Referential Integrity

Foreign keys ensure that related records reference valid parent records.

### 5.3 Domain Integrity

CHECK constraints enforce the following rules:

| Relation         | Constraint                             |
| ---------------- | -------------------------------------- |
| users            | Role must be ADMIN or FACULTY          |
| classrooms       | Floor must be non-negative             |
| classrooms       | Capacity must be positive              |
| course_offerings | Expected strength must be positive     |
| time_slots       | End time must be later than start time |
| allocations      | Day must be Monday through Saturday    |

### 5.4 Uniqueness Constraints

Unique constraints prevent duplicate:

- Department names
- User email addresses
- Faculty user associations
- Classroom numbers within the same building
- Facility names
- Course codes
- Time-slot start/end combinations
- Classroom allocations for the same slot and day
- Course-offering allocations for the same slot and day

## 6. Normalization

The relational design uses normalization principles to reduce redundancy and improve data consistency.

- **First Normal Form (1NF):** Attributes contain atomic values. Multiple classroom facilities are represented using a separate associative relation rather than a multi-valued column.
- **Second Normal Form (2NF):** The associative relations use composite primary keys and contain no additional non-key attributes with partial dependencies.
- **Third Normal Form (3NF):** Department, faculty account, course and classroom details are maintained in their respective relations instead of being unnecessarily duplicated in allocation records.
- **BCNF:** Under the functional dependencies implied by the declared primary and unique keys, the relations are designed to satisfy BCNF.

## 7. Advanced Database Objects

The database includes the following additional objects:

| Object                            | Type             | Purpose                                                                   |
| --------------------------------- | ---------------- | ------------------------------------------------------------------------- |
| classroom_timetable               | View             | Combines allocation, course, faculty, classroom and time-slot information |
| classroom_utilization             | View             | Calculates classroom allocation counts and average utilization            |
| allocate_classroom                | Stored Procedure | Inserts classroom allocation records                                      |
| get_classroom_utilization         | Function         | Calculates average utilization for a specified classroom                  |
| validate_classroom_allocation     | Trigger Function | Validates classroom allocation business rules                             |
| trg_validate_classroom_allocation | Trigger          | Executes validation before allocation INSERT or UPDATE                    |

## 8. Additional Indexes

| Index                         | Table            | Indexed Columns               |
| ----------------------------- | ---------------- | ----------------------------- |
| idx_allocations_room_day_slot | allocations      | room_id, day_of_week, slot_id |
| idx_allocations_offering      | allocations      | offering_id                   |
| idx_allocations_slot          | allocations      | slot_id                       |
| idx_course_offerings_faculty  | course_offerings | faculty_id                    |
| idx_courses_department        | courses          | department_id                 |
| idx_classrooms_capacity       | classrooms       | capacity                      |

Primary-key and unique constraints also create their associated indexes automatically.

## 9. Database Integrity and Concurrency Control

The `trg_validate_classroom_allocation` trigger executes before INSERT or UPDATE operations on the allocations table.

It validates:

1. Existence of the course offering, classroom and time slot.
2. Classroom capacity against expected student strength.
3. Availability of all facilities required by the course offering.
4. Overlapping allocations in the same classroom.
5. Overlapping allocations for the same faculty member.

The trigger also uses PostgreSQL transaction-level advisory locking through `pg_advisory_xact_lock` to serialize allocation writes during validation.

## 10. Conclusion

The UniSpace relational schema provides a structured representation of the university classroom allocation database. Primary keys, foreign keys, unique constraints, check constraints, associative relations, views, stored procedures, functions and triggers support data integrity and scheduling operations.

The corresponding visual ER diagram is maintained separately as `ER_Diagram.png`, with its editable source in `ER_Diagram.drawio`.
