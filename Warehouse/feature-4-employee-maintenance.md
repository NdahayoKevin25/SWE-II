# Feature: Employee Maintenance

**Feature ID:** 4
**Branch pattern:** `feature/4-employee-maintenance`
**Status:** Draft
**Created:** 2026-09-28
**Input:** Add and edit employees: managers, office staff, warehouse staff, and truck driver including their log-in, role, and home warehouse. **Depends on:** [Feature 1 — User Authorization](feature-1-user-authorization.md), [Feature 3 — Warehouse Maintenance](feature-3-warehouse-maintenance.md)

---

## User Stories

### US-4.1: Add an employee

**As a** manager
**I want to** add an employee with a role, home warehouse, and log-in
**So that** they can use the system for their job

**Priority:** P1
**Independent test:** Add an employee with good data; that employee can log in
**Acceptance scenarios:** see ### US-4.1 under Acceptance Criteria

### US-4.2: Edit an employee

**As a** manager
**I want to** change an employee's details, role, or home warehouse, or reset their password
**So that** their access matches their current job

**Priority:** P1
**Independent test:** Change a role from Warehouse Staff to Office Staff; the next log in opens the Office menu
**Acceptance scenarios:** see ### US-4.2 under Acceptance Criteria

### US-4.3: Find an employee

**As a** manager
**I want to** list employees and filter by role or warehouse
**So that** I can quickly find someone, like all the drivers at one warehouse

**Priority:** P2
**Independent test:** Filter by role "Driver" and warehouse "WH1"; only those drivers are listed
**Acceptance scenarios:** see ### US-4.3 under Acceptance Criteria

---



## Requirements



### Functional Requirements

- **FR-001**: An employee MUST have: employee number (unique), first name, last name, phone, role, username (unique), and a starting password.
- **FR-002**: Home warehouse MUST be filled in for Office Staff, Warehouse Staff, and Drivers. It is optional for Managers.
- **FR-003**: Drivers MUST have a driver's license number. Other roles do not need one.
- **FR-004**: Email is optional. If it is filled in, it must look like a real email address.
- **FR-005**: Passwords MUST be at least 8 characters.
- **FR-006**: A used employee number or username MUST give a duplicate error. Usernames are compared without caring about capital letters ("ARuiz" = "aruiz").
- **FR-007**: Missing or bad data MUST give a validation error, and nothing is saved.
- **FR-008**: The employee number MUST NOT change after it is saved.
- **FR-009**: A manager MUST NOT be able to remove their own Manager role (so there is always a manager).
- **FR-010**: The starting menu comes from the role (Feature 1). Changing the role changes the menu at the next log in.
- **FR-011**: Only managers MAY see or change employees. Office staff MAY see the list of drivers for their warehouse (to pick a driver for a route).

---



## Assumptions

- One employee = one log-in.
- No payroll or time clock.
- Removing an employee is not in this version.

---



## Edge Cases

- Driver without a license number → validation error
- Office staff, warehouse staff, or driver without a home warehouse → validation error
- Username already used with different capital letters → duplicate error
- Manager changes their own role → not allowed error

---



## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A new employee can log in right away with the starting password
- **SC-003**: Passwords are never shown on any screen

---



## Data Ownership & Isolation

- Employees belong to the company. Only managers can list or change them.

---



## Key Entities

- **Employee**: a person who works for Eagle Wholesale and uses the system. Has a role and (except managers) a home warehouse.

---



## Acceptance Criteria (Gherkin)



### US-4.1 — Add an employee



#### Scenario: Manager adds a warehouse employee with good data

- **Given** warehouse "WH1" exists
- **When** the manager adds employee "E105" Ana Ruiz, role "Warehouse Staff", home warehouse "WH1", username "aruiz", password "Start#2026"
- **Then** the employee is saved
- **And** "aruiz" can log in and sees the Warehouse Floor menu



#### Scenario: Driver is not saved without a license number

- **Given** the manager is adding an employee with role "Driver"
- **When** the manager leaves the license number blank and saves
- **Then** the system gives a validation error
- **And** no employee is saved



#### Scenario: Office staff is not saved without a home warehouse

- **Given** the manager is adding an employee with role "Office Staff"
- **When** the manager leaves the home warehouse blank and saves
- **Then** the system gives a validation error



#### Scenario: Duplicate username is rejected

- **Given** an employee with username "aruiz" exists
- **When** the manager adds another employee with username "ARuiz"
- **Then** the system gives a duplicate error



### US-4.2 — Edit an employee



#### Scenario: Manager changes an employee's role

- **Given** "aruiz" has role "Warehouse Staff"
- **When** the manager changes the role to "Office Staff" and saves
- **Then** the next time "aruiz" logs in, the Office menu opens



#### Scenario: Manager resets a password

- **Given** employee "aruiz" exists
- **When** the manager resets the password to "NewPass#1"
- **Then** "aruiz" can log in with "NewPass#1"
- **And** the old password no longer works



#### Scenario: Manager cannot remove their own manager role

- **Given** manager "boss1" is logged in
- **When** "boss1" changes their own role to "Office Staff"
- **Then** the system gives a not allowed error
- **And** "boss1" is still a manager



### US-4.3 — Find an employee



#### Scenario: Filter employees by role and warehouse

- **Given** drivers "D7" and "D8" work at "WH1" and driver "D9" works at "WH2"
- **When** the manager filters by role "Driver" and warehouse "WH1"
- **Then** only "D7" and "D8" are listed

