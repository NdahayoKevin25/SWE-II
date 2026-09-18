# Feature: Driver Routes & Member Ride Assignments

**Feature ID:** 6
**Branch pattern:** `feature/6-driver-routes-assignments`
**Status:** Draft
**Created:** 2026-09-15
**Input:** Assign drivers to fixed pickup routes and assign members to a route so the church knows who needs a ride and who's driving.
**Depends on:** [Feature 1 — User Authentication & Role Management](feature-1-user-auth-roles.md), [Feature 2 — Member & Household Management](feature-2-member-household-management.md)

---

## User Stories

### US-6.1: Create a route

**As an** admin
**I want to** create a route with a name and a set of pickup stops
**So that** drivers know where to go

**Priority:** P1
**Independent test:** Create a route with at least one stop and confirm it's available to assign a driver and riders
**Acceptance scenarios:** see ### US-6.1 under Acceptance Criteria

### US-6.2: Assign a driver to a route

**As an** admin
**I want to** assign a staff member with the Driver role to a route
**So that** it's clear who's responsible for it

**Priority:** P1
**Independent test:** Assign a driver to a route and confirm the driver can view it
**Acceptance scenarios:** see ### US-6.2 under Acceptance Criteria

### US-6.3: Assign a member to a route

**As an** admin or usher
**I want to** assign a member to a route and a specific stop
**So that** the driver knows who to pick up

**Priority:** P1
**Independent test:** Assign a member to a route stop and confirm they appear on the driver's rider list
**Acceptance scenarios:** see ### US-6.3 under Acceptance Criteria

### US-6.4: View a driver's route

**As a** driver
**I want to** view my assigned route(s), stops, and riders
**So that** I know my pickups for the day

**Priority:** P1
**Independent test:** Sign in as a driver and view an assigned route with its riders
**Acceptance scenarios:** see ### US-6.4 under Acceptance Criteria

### US-6.5: Reassign a member to a different route

**As an** admin or usher
**I want to** move a member from one route/stop to another
**So that** ride assignments stay current

**Priority:** P2
**Independent test:** Move a member's assignment to a different route and confirm the old assignment is replaced
**Acceptance scenarios:** see ### US-6.5 under Acceptance Criteria

---



## Requirements



### Functional Requirements

- **FR-001**: Only `admin` MUST be able to create a Route, which requires a name and at least one stop (label + address).
- **FR-002**: System MUST allow assigning exactly one driver per route at a time; a driver MAY be assigned to more than one route.
- **FR-003**: Only staff with the `driver` role MUST be assignable as a route's driver.
- **FR-004**: `admin` or `usher` roles MUST be able to assign a member to a route and a specific stop on that route.
- **FR-005**: A member MUST be assigned to at most one route at a time.
- **FR-006**: The route stop chosen for a member MUST belong to the route they are being assigned to.
- **FR-007**: A driver MUST only be able to view routes assigned to them, not other drivers' routes.
- **FR-008**: Reassigning a member to a different route/stop MUST replace their prior assignment, not create a second one.
- **FR-009**: Routes run for every service; the system MUST NOT require a per-service route schedule.

---



## Key Entities

- **Route**: a named set of pickup stops, assigned to one driver (Staff from Feature 1); runs every service.
- **RouteStop**: an individual pickup stop (label + address) belonging to a route.
- **RouteAssignment**: links a Member (Feature 2) to a route and a specific stop on that route.

---



## Acceptance Criteria



### US-6.1 — Create a route



#### Scenario: Admin creates a route with stops 

- **Given** I am signed in as an admin
- **When** I create a route named `North Side Route` with stops `Corner of 5th & Elm` and `Oak & Main`
- **Then** the API returns `201`
- **And** `North Side Route` exists with both stops



#### Scenario: Non-admin attempts to create a route 

- **Given** I am signed in as a usher
- **When** I attempt to create a route
- **Then** the API returns `403` and no route is created



### US-6.2 — Assign a driver to a route



#### Scenario: Admin assigns a driver to a route

- **Given** route `North Side Route` exists
- **And** staff member `Mike` has role `driver`
- **When** I assign `Mike` as the driver for `North Side Route`
- **Then** the API returns `200`
- **And** `Mike` can view `North Side Route`



#### Scenario: Admin assigns a non-driver staff member

- **Given** route `North Side Route` exists
- **And** staff member `Sara` has role `usher`
- **When** I attempt to assign `Sara` as the driver for `North Side Route`
- **Then** the API returns a validation error and the route's driver is unchanged



### US-6.3 — Assign a member to a route



#### Scenario: Usher assigns a member to a route stop

- **Given** route `North Side Route` has stop `Corner of 5th & Elm`
- **And** member `Jane Smith` exists and has no current route
- **When** I assign `Jane Smith` to `North Side Route` at stop `Corner of 5th & Elm`
- **Then** the API returns `201`
- **And** `Jane Smith` appears on `North Side Route`'s rider list



#### Scenario: Member is assigned to a stop from a different route (failure)

- **Given** route `North Side Route` exists and route `East Side Route` has stop `Park & 3rd`
- **When** I attempt to assign a member to `North Side Route` using stop `Park & 3rd`
- **Then** the API returns a validation error and no assignment is created



### US-6.4 — View a driver's route



#### Scenario: Driver views their assigned route

- **Given** driver `Mike` is assigned to `North Side Route`
- **And** `North Side Route` has riders assigned to its stops
- **When** `Mike` requests his route
- **Then** the API returns `200` with the route's stops and riders



#### Scenario: Driver attempts to view a route not assigned to them

- **Given** driver `Mike` is not assigned to `East Side Route`
- **When** `Mike` requests `East Side Route`
- **Then** the API returns `403`



### US-6.5 — Reassign a member to a different route



#### Scenario: Member is moved to a different route/stop

- **Given** member `Jane Smith` is assigned to `North Side Route` at stop `Corner of 5th & Elm`
- **And** route `East Side Route` has stop `Park & 3rd`
- **When** I reassign `Jane Smith` to `East Side Route` at stop `Park & 3rd`
- **Then** the API returns `200`
- **And** `Jane Smith` no longer appears on `North Side Route`'s rider list
- **And** `Jane Smith` appears on `East Side Route`'s rider list



#### Scenario: Member is reassigned to the same route/stop

- **Given** member `Jane Smith` is already assigned to `North Side Route` at stop `Corner of 5th & Elm`
- **When** I reassign her to the same route and stop
- **Then** the API returns `200` with no duplicate assignment created

