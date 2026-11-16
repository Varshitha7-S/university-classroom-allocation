# UniSpace – Relational Database Schema

**Database:** PostgreSQL  
**Project:** University Classroom Allocation System

The following relations represent the relational structure of UniSpace. PK denotes Primary Key, FK denotes Foreign Key, and UQ denotes Unique.

## 1. DEPARTMENTS

`DEPARTMENTS(department_id, department_name)`

- Primary Key: `department_id`
- Unique: `department_name`

## 2. USERS

`USERS(user_id, name, email, password_hash, role)`

- Primary Key: `user_id`
- Unique: `email`
- Role constraint: `ADMIN` or `FACULTY`

## 3. FACULTY

`FACULTY(faculty_id, user_id, department_id)`

- Primary Key: `faculty_id`
- Foreign Key: `user_id` references `USERS(user_id)`
- Foreign Key: `department_id` references `DEPARTMENTS(department_id)`
- Unique: `user_id`

## 4. CLASSROOMS

`CLASSROOMS(room_id, room_number, building, floor, capacity)`

- Primary Key: `room_id`
- Unique: `(building, room_number)`
- Constraints: `floor >= 0`, `capacity > 0`

## 5. FACILITIES

`FACILITIES(facility_id, facility_name)`

- Primary Key: `facility_id`
- Unique: `facility_name`

## 6. CLASSROOM_FACILITIES

`CLASSROOM_FACILITIES(room_id, facility_id)`

- Composite Primary Key: `(room_id, facility_id)`
- Foreign Key: `room_id` references `CLASSROOMS(room_id)`
- Foreign Key: `facility_id` references `FACILITIES(facility_id)`

## 7. COURSES

`COURSES(course_id, course_name, course_code, department_id)`

- Primary Key: `course_id`
- Unique: `course_code`
- Foreign Key: `department_id` references `DEPARTMENTS(department_id)`

## 8. COURSE_OFFERINGS

`COURSE_OFFERINGS(offering_id, course_id, faculty_id, expected_strength, semester, academic_year)`

- Primary Key: `offering_id`
- Foreign Key: `course_id` references `COURSES(course_id)`
- Foreign Key: `faculty_id` references `FACULTY(faculty_id)`
- Constraint: `expected_strength > 0`

## 9. OFFERING_REQUIRED_FACILITIES

`OFFERING_REQUIRED_FACILITIES(offering_id, facility_id)`

- Composite Primary Key: `(offering_id, facility_id)`
- Foreign Key: `offering_id` references `COURSE_OFFERINGS(offering_id)`
- Foreign Key: `facility_id` references `FACILITIES(facility_id)`

## 10. TIME_SLOTS

`TIME_SLOTS(slot_id, start_time, end_time)`

- Primary Key: `slot_id`
- Unique: `(start_time, end_time)`
- Constraint: `end_time > start_time`

## 11. ALLOCATIONS

`ALLOCATIONS(allocation_id, offering_id, room_id, slot_id, day_of_week, allocation_date)`

- Primary Key: `allocation_id`
- Foreign Key: `offering_id` references `COURSE_OFFERINGS(offering_id)`
- Foreign Key: `room_id` references `CLASSROOMS(room_id)`
- Foreign Key: `slot_id` references `TIME_SLOTS(slot_id)`
- Unique: `(room_id, slot_id, day_of_week)`
- Unique: `(offering_id, slot_id, day_of_week)`
- Day constraint: Monday to Saturday

## Relationship Summary

| Relationship | Cardinality |
|---|---|
| Departments – Faculty | One-to-Many |
| Departments – Courses | One-to-Many |
| Users – Faculty | One-to-One (optional faculty record) |
| Courses – Course Offerings | One-to-Many |
| Faculty – Course Offerings | One-to-Many |
| Classrooms – Allocations | One-to-Many |
| Course Offerings – Allocations | One-to-Many |
| Time Slots – Allocations | One-to-Many |
| Classrooms – Facilities | Many-to-Many |
| Course Offerings – Facilities | Many-to-Many |

## Database Integrity

The relational schema enforces referential integrity through foreign keys, entity integrity through primary keys, and domain integrity through unique and check constraints.

The many-to-many relationships are implemented using junction tables with composite primary keys. Additional allocation validation is implemented through database triggers to check classroom capacity, required facilities, and scheduling conflicts.

**Implementation reference:** `database/schema.sql`, `database/advanced.sql`, `database/functions.sql`, `database/procedures.sql`, and `database/triggers.sql`.
