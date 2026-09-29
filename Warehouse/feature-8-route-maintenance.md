# Feature: Routes and Deliveries

**Feature ID:** 8
**Branch pattern:** `feature/8-route-maintenance`
**Status:** Draft
**Created:** 2026-09-28
**Input:** Set up delivery routes (a list of stores in driving order with a usual driver), then let drivers see their route for the day, load the truck, and confirm each delivery at the store.
**Depends on:** [Feature 3 — Warehouse Maintenance](feature-3-warehouse-maintenance.md), [Feature 4 — Employee Maintenance](feature-4-employee-maintenance.md), [Feature 7 — Customers](feature-7-customer-maintenance.md). Loading and delivering (US-8.4 to US-8.7) need shipped orders from Feature 7.

---



## User Stories



### US-8.1: Add a route

**As an** office staff member
**I want to** add a route with a code, name, delivery days, and usual driver
**So that** shipped orders can be put on it

**Priority:** P1
**Independent test:** Add route "N1" with good data; it shows up in the route list
**Acceptance scenarios:** see ### US-8.1 under Acceptance Criteria

### US-8.2: Set the stores on a route

**As an** office staff member
**I want to** add, remove, and reorder the stores on a route
**So that** the driver visits them in driving order

**Priority:** P1
**Independent test:** Add three stores, move the third to first; the stops show in the new order
**Acceptance scenarios:** see ### US-8.2 under Acceptance Criteria

### US-8.3: Edit a route

**As an** office staff member
**I want to** change a route's name, delivery days, or usual driver
**So that** the route matches how we actually deliver

**Priority:** P1
**Independent test:** Change the driver from "D7" to "D8"; the route shows "D8"
**Acceptance scenarios:** see ### US-8.3 under Acceptance Criteria

### US-8.4: See my route for the day

**As a** driver
**I want to** see today's orders for me, in stop order, with each store's address and number of packages
**So that** I know where to go and in what order

**Priority:** P1
**Independent test:** Driver "D7" opens My Route and sees only his shipped orders for today, in stop order
**Acceptance scenarios:** see ### US-8.4 under Acceptance Criteria

### US-8.5: Load the truck

**As a** driver
**I want to** check off each order as I load it onto the truck
**So that** I don't leave the warehouse missing an order

**Priority:** P2
**Independent test:** Load 1 of 2 orders; the system warns that 1 order is not loaded
**Acceptance scenarios:** see ### US-8.5 under Acceptance Criteria

### US-8.6: Confirm a delivery

**As a** driver
**I want to** enter how much of each line the store actually got, a reason for anything missing, and who signed for it
**So that** the office knows the store got its order

**Priority:** P1
**Independent test:** Confirm a delivery with every line in full; the order becomes Delivered
**Acceptance scenarios:** see ### US-8.6 under Acceptance Criteria

### US-8.7: See delivery problems by driver

**As a** manager
**I want** a report of every line delivered short, grouped by driver
**So that** I can follow up on damaged or missing goods

**Priority:** P2
**Independent test:** Driver "D7" with 2 short lines shows "2 lines short" and the reasons
**Acceptance scenarios:** see ### US-8.7 under Acceptance Criteria

---

## Requirements



### Functional Requirements

**Route setup**

- **FR-001**: A route MUST have: a route code (unique within its warehouse, up to 10 letters/numbers), a name, and a warehouse.
- **FR-002**: A route MAY have a usual driver (a Driver from the same warehouse) and delivery days (any of Monday–Sunday).
- **FR-003**: A route has a list of **stops**. Each stop is a store from the same warehouse, numbered 1, 2, 3, … with no gaps.
- **FR-004**: A store can be on **only one** route. Adding a store that is already on another route MUST give an error naming that route.
- **FR-005**: Removing or moving a stop MUST renumber the other stops so there are no gaps.
- **FR-006**: A used route code in the same warehouse MUST give a duplicate error. Missing or bad data MUST give a validation error.

**Driver's day**

- **FR-007**: **My Route** MUST show the logged-in driver's Shipped orders for the chosen day (default today), sorted by stop number. Each shows the order number, bill of lading number, store, address, delivery instructions, and packages.
- **FR-008**: **Load** MUST let the driver mark each order as loaded. If the driver marks the truck "ready to leave" while some orders are not loaded, the system MUST warn and list them.

**Delivery**

- **FR-009**: For each order, the driver MUST enter a delivered quantity for each line (filled in with the shipped quantity to start), from 0 up to the shipped quantity.
- **FR-010**: If delivered is less than shipped, the driver MUST pick a reason: **Damaged**, **Refused**, **Missing**, or **Other** (Other needs a note).
- **FR-011**: The driver MUST enter the name of the store person who signed. The system saves the date and time.
- **FR-012**: Confirming sets the order to **Delivered**. It happens once and cannot be changed after.
- **FR-013**: Delivery problems do NOT change stock. If goods come back to the warehouse, they are added back with an inventory adjustment (Feature 9).
- **FR-014**: If the store is closed, the driver does not confirm. Office staff moves the order to another day or driver.

**Report**

- **FR-015**: The **delivery problems report** MUST list, for a warehouse and date range, every line delivered short (order, store, driver, item, shipped, delivered, difference, reason), plus a summary per driver: lines short, units short, and a count for each reason.

**Who can do what**

- **FR-016**: Manager and Office Staff MAY set up routes and move orders between days/drivers. Drivers MAY only see, load, and confirm their own orders. Managers MAY confirm any delivery (for example, from a signed paper copy).

---

## Assumptions

- Routes are fixed. We do not use maps or GPS.
- We do not track which truck is used. The driver and the day are enough.
- The signed paper bill of lading is still kept.
- Removing a route is not in this version.

---

## Edge Cases

- Usual driver from another warehouse, or not a Driver → validation error
- Store from another warehouse as a stop → validation error
- Store already on another route → error naming that route
- Delivered more than shipped → validation error
- Short line with no reason → validation error
- Blank "signed by" → validation error
- Driver opening another driver's order → not found error
- Confirming an order that is already Delivered → wrong status error

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: Stop numbers are always 1, 2, 3, … with no gaps
- **SC-003**: A driver can confirm a delivery with no problems in three taps or fewer
- **SC-004**: The My Route screen works on a phone

---

## Data Ownership & Isolation

- Routes belong to one warehouse.
- Drivers only see their own orders.

---

## Key Entities

- **Route**: a delivery loop from one warehouse with a usual driver and delivery days
- **Route Stop**: a store's place in the route order
- **Delivery**: what the store got, why anything is missing, and who signed

---

## Acceptance Criteria (Gherkin)



### US-8.1 — Add a route



#### Scenario: Office staff adds a route with good data

- **Given** driver "D7" works at "WH1"
- **When** office staff from "WH1" adds route "N1" "North Loop" with driver "D7" and delivery days Tuesday and Friday
- **Then** the route is saved
- **And** it shows up in the route list for "WH1"



#### Scenario: Route is not saved without a name

- **Given** office staff is adding a route
- **When** the employee leaves the name blank and saves
- **Then** the system gives a validation error



#### Scenario: Route is not saved with a driver from another warehouse

- **Given** driver "D9" works at "WH2"
- **When** office staff from "WH1" adds a route with driver "D9"
- **Then** the system gives a validation error



#### Scenario: Duplicate route code is rejected

- **Given** route "N1" exists in "WH1"
- **When** the employee adds another route "N1" in "WH1"
- **Then** the system gives a duplicate error



### US-8.2 — Set the stores on a route



#### Scenario: Office staff adds and reorders stops

- **Given** route "N1" has stops "C001" (1) and "C002" (2)
- **When** the employee adds "C003" and moves it to stop 1
- **Then** the stops are "C003" (1), "C001" (2), "C002" (3)



#### Scenario: Store already on another route cannot be added

- **Given** store "C010" is on route "S1"
- **When** the employee adds "C010" to route "N1"
- **Then** the system gives an error naming route "S1"



#### Scenario: Removing a stop renumbers the rest

- **Given** route "N1" has stops "C003" (1), "C001" (2), "C002" (3)
- **When** the employee removes "C001"
- **Then** the stops are "C003" (1), "C002" (2)



### US-8.3 — Edit a route



#### Scenario: Office staff changes the usual driver

- **Given** route "N1" has driver "D7"
- **When** the employee changes the driver to "D8" and saves
- **Then** route "N1" shows driver "D8"



### US-8.4 — See my route for the day



#### Scenario: Driver sees only their orders in stop order

- **Given** driver "D7" has Shipped orders "SO-000026" (stop 1) and "SO-000020" (stop 2) for today
- **And** driver "D8" has Shipped order "SO-000027" for today
- **When** "D7" opens My Route
- **Then** "SO-000026" and "SO-000020" are listed in that order
- **And** "SO-000027" is not listed



### US-8.5 — Load the truck



#### Scenario: Driver is warned about an order not loaded

- **Given** "D7" has orders "SO-000026" and "SO-000020" for today
- **And** "D7" has marked only "SO-000026" as loaded
- **When** "D7" selects "Ready to leave"
- **Then** the system warns that "SO-000020" is not loaded



### US-8.6 — Confirm a delivery



#### Scenario: Driver confirms a full delivery

- **Given** "SO-000026" is Shipped to driver "D7"
- **When** "D7" keeps every delivered quantity the same as shipped and enters signed by "Maria Lopez"
- **Then** "SO-000026" is Delivered



#### Scenario: Driver records a damaged case

- **Given** "SO-000020" shipped 1 case of "CH-1001" and 6 each of "CH-1002"
- **When** "D7" enters "CH-1001" delivered 0 with reason "Damaged" and "CH-1002" delivered 6
- **Then** "SO-000020" is Delivered
- **And** the "CH-1001" line shows delivered 0, reason "Damaged"
- **And** on hand for "CH-1001" does not change



#### Scenario: Short line needs a reason

- **Given** "SO-000020" shipped 6 each of "CH-1002"
- **When** "D7" enters 4 delivered with no reason
- **Then** the system gives a validation error



#### Scenario: Delivery needs a signer

- **Given** "SO-000026" is Shipped to driver "D7"
- **When** "D7" confirms with "signed by" left blank
- **Then** the system gives a validation error
- **And** "SO-000026" stays Shipped



#### Scenario: Driver cannot confirm another driver's order

- **Given** "SO-000027" belongs to driver "D8"
- **When** driver "D7" tries to confirm it
- **Then** the system gives a not found error



### US-8.7 — See delivery problems by driver



#### Scenario: Delivery problems are grouped by driver

- **Given** this month "D7" delivered "SO-000020" with "CH-1001" 0 of 24 units (Damaged) and "SO-000030" with "CH-1002" 4 of 6 units (Missing)
- **When** the manager runs the delivery problems report for "WH1"
- **Then** both lines are listed with their reasons
- **And** "D7" shows 2 lines short, 26 units short, Damaged 1, Missing 1

