# Feature: User Authorization

**Feature ID:** 1
**Branch pattern:** `feature/1-user-authorization`
**Status:** Draft
**Created:** 2026-09-24
**Input:** Control who can use the warehouse system based on who they are and what role they have. Support log in and log out. **Depends on:** —

---

## User Stories

### US-1.1: Log in

**As an** employee (manager, warehouse staff, or driver) **I want to** log in with my username and password
**So that** I can use the parts of the system that my role is allowed to use

**Priority:** P1
**Independent test:** Call `login` with a good username and password; the session should have that employee’s role and warehouse **Acceptance scenarios:** see ### US-1.1 under Acceptance Criteria

### US-1.2: Log out

**As an** employee who is logged in
**I want to** log out
**So that** nobody else can keep using the system as me

**Priority:** P1
**Independent test:** Log in, log out, then try a protected action and get an unauthorization error **Acceptance scenarios:** see ### US-1.2 under Acceptance Criteria

### US-1.3: Role-based access control

**As a** manager
**I want** management and report functions to only be for managers
**So that** other employees can only do the work they are responsible for

**Priority:** P1
**Independent test:** A `DRIVER` session trying a manager-only action gets a Not Authorized Error; a `MANAGER` session is allowed
**Acceptance scenarios:** see ### US-1.3 under Acceptance Criteria

### US-1.4: Warehouse scope

**As a** manager
**I want** warehouse staff and drivers limited to their home warehouse **So that** people from one warehouse cannot change another warehouse’s stock or orders

**Priority:** P1
**Independent test:** A warehouse staff user from warehouse 1 asking for a warehouse 2 record gets an unauthorization error; a manager can access both
**Acceptance scenarios:** see ### US-1.4 under Acceptance Criteria

---



## Requirements



### Functional Requirements

- **FR-001**: The system MUST let employees log in with a username and password.
- **FR-002**: The system MUST only let them in if the username and password match what we have stored.
- **FR-003**: If login fails, the system MUST show a generic error. It should not say whether the username or the password was the problem.
- **FR-004**: When someone logs out, the system MUST end the session so they are not still authorized on that client.
- **FR-005**: The system MUST reject log out if nobody is logged in.
- **FR-006**: Each employee MUST have exactly one role: `MANAGER`, `OFFICE STAFF`, `WAREHOUSE STAFF`, or `DRIVER`.
- **FR-007**: Each office, warehouse, and driver employee MUST have a home warehouse. A manager MAY have one, but they are not limited by it.
- **FR-008**: For non-manager roles, actions on warehouse records (inventory, customers, routes, supplier orders, customer orders, deliveries) MUST stay in that employee’s home warehouse. If they try something outside that, the system MUST report an unauthorization error.
- **FR-009**: There MUST be an initial manager account so the company can log in before other employees exist.
- **FR-010**: If an employee is deactivated, they MUST not be able to log in anymore.

---



## Assumptions

- One password per employee.
- Username uniquely identifies a user.
- The first manager is created by a seed/script, not a sign-up screen.
- Employee accounts are created in Feature 4. This feature only logs people in and checks what they are allowed to do.

---



## Edge Cases

- Empty username or password → `ValidationError`
- Unknown username, wrong password, or inactive employee → `NotAuthenticatedError` with the same generic message
- Log out with no session → `NotAuthenticatedError`
- Role is not allowed by the access matrix → `NotAuthorizedError`
- Non-manager asking for another warehouse’s record → `NotFoundError`
- Employee gets deactivated while they are still logged in → the next protected action returns `NotAuthenticatedError`

---



## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: Bad or inactive credentials never create a session
- **SC-003**: After log out, every protected action returns `NotAuthenticatedError`

---



## Data Ownership & Isolation

- A session belongs to one employee. It never acts as someone else.
- Role and home warehouse are read from the employee record at log in and checked again on each protected action.
- Managers can see all warehouses. Everyone else only sees their home warehouse.

---



## Key Entities

- **Employee**: a person who uses the system. Has a username, password, role, and home warehouse
- **Session**: the logged-in employee’s identity for one client (employeeId, username, role, home warehouseId, login time)

---



## Acceptance Criteria (Gherkin)



### US-1.1 — Log in



#### Scenario: Employee logs in with valid credentials

- **Given** an active employee exists with username "office1", password, role, and home warehouse
- **When** the employee submits those credentials
- **Then** the system creates a session
- **And** the session shows role and home warehouse



#### Scenario: Employee logs in with a wrong password

- **Given** an active employee exists with username "office1"
- **When** the employee submits username "office1" and an incorrect password
- **Then** the system shows "Invalid username or password"
- **And** no session is created



#### Scenario: Unknown user cannot log in

- **Given** no employee exists with username "ghost"
- **When** someone submits username "ghost" and any password
- **Then** the system shows "Invalid username or password"
- **And** no session is created



#### Scenario: Inactive employee cannot log in

- **Given** employee "driver7" has been deactivated
- **When** "driver7" submits correct credentials
- **Then** the system shows "Invalid username or password"
- **And** no session is created



### US-1.2 — Log out



#### Scenario: Logged-in employee logs out

- **Given** an employee is logged in
- **When** the employee selects Log out
- **Then** the session ends
- **And** calling any protected operation returns `NotAuthenticatedError`



#### Scenario: Log out with no one logged in

- **Given** no employee is logged in
- **When** the client attempts to log out
- **Then** the system returns `NotAuthenticatedError`



### US-1.3 — Role-based access control



#### Scenario: Manager uses a manager-only function

- **Given** an employee is logged in with role "MANAGER"
- **When** the employee opens Warehouse maintenance
- **Then** the system allows access



#### Scenario: Driver is denied a manager-only function

- **Given** an employee is logged in with role "DRIVER"
- **When** the employee calls a warehouse maintenance operation
- **Then** the system returns `NotAuthorizedError`



#### Scenario: Menu shows only permitted functions

- **Given** an employee is logged in with role "WAREHOUSE"
- **When** the home screen is shown
- **Then** the menu includes Receive, Pick, Ship, and Inventory Adjustments
- **And** the menu does not include Employee Maintenance or Reports



### US-1.4 — Warehouse scope



#### Scenario: Office user is limited to their warehouse

- **Given** an employee with role "OFFICE" and home warehouse "WH1" is logged in
- **And** customer "C200" belongs to warehouse "WH2"
- **When** the employee requests customer "C200"
- **Then** the system returns `NotFoundError`



#### Scenario: Manager can access every warehouse

- **Given** an employee with role "MANAGER" is logged in
- **And** customers exist in warehouses "WH1" and "WH2"
- **When** the manager lists customers for all warehouses
- **Then** customers from both warehouses are returned

---

