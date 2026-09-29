# Feature: Warehouse Maintenance

**Feature ID:** 3
**Branch pattern:** `feature/3-warehouse-maintenance`
**Status:** Draft
**Created:** 2026-09-28
**Input:** Add and edit the company's warehouses, including two settings: what time supplier orders get emailed each day, and how many days of stock the min and max should cover.
**Depends on:** [Feature 1 — User Authorization](feature-1-user-authorization.md), [Feature 2 — Company Maintenance](feature-2-company-maintenance.md)

---

## User Stories

### US-3.1: Add a warehouse

**As a** manager
**I want to** add a warehouse with its number, name, and address
**So that** stock, stores, and orders can be tracked per warehouse

**Priority:** P1
**Independent test:** Add warehouse "WH2" with good data; it shows up in the warehouse list
**Acceptance scenarios:** see ### US-3.1 under Acceptance Criteria

### US-3.2: Edit a warehouse

**As a** manager
**I want to** edit a warehouse's details and settings
**So that** its address and settings stay correct

**Priority:** P1
**Independent test:** Change the order send time; the new time is shown
**Acceptance scenarios:** see ### US-3.2 under Acceptance Criteria

### US-3.3: See the list of warehouses

**As a** manager
**I want to** see all warehouses in one list
**So that** I can find and open the one I need

**Priority:** P2
**Independent test:** With two warehouses saved, the list shows both sorted by number
**Acceptance scenarios:** see ### US-3.3 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: A warehouse MUST have a **warehouse number** (unique, up to 10 letters/numbers), name, street, city, state, zip code, and phone.
- **FR-002**: A warehouse MUST have three settings. If left blank they get these defaults:
  - **Order send time** — the time each day that supplier orders are emailed (default 4:00 PM)
  - **Min days of stock** — at least 1 (default 14)
  - **Max days of stock** — must be more than min days (default 60)
- **FR-003**: A warehouse number that is already used MUST give a duplicate error.
- **FR-004**: Missing or bad data MUST give a validation error, and nothing is saved.
- **FR-005**: The warehouse number MUST NOT change after it is saved. Everything else can be edited.
- **FR-006**: Only managers MAY add or edit warehouses. Other employees can only see the name and number of their home warehouse.

---

## Assumptions

- Eagle Wholesale starts with one warehouse, but the system allows more.
- We do not move stock between warehouses.
- Removing a warehouse is not in this version.

---

## Edge Cases

- Duplicate warehouse number → duplicate error
- Max days of stock not more than min days → validation error
- Order send time is not a real time → validation error
- Trying to change the warehouse number → validation error

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A new warehouse gets the default settings when none are typed in

---

## Data Ownership & Isolation

- Warehouses belong to the company. Only managers change them.

---

## Key Entities

- **Warehouse**: a building that holds stock. Its stores, routes, stock, and orders belong to it.
- **Warehouse Settings**: order send time, min days of stock, max days of stock

---

## Acceptance Criteria (Gherkin)



### US-3.1 — Add a warehouse



#### Scenario: Manager adds a warehouse with good data

- **Given** a manager is logged in
- **When** the manager adds warehouse "WH2" named "Eagle North" with a full address and phone
- **Then** the warehouse is saved
- **And** it shows order send time 4:00 PM, min days 14, and max days 60



#### Scenario: Warehouse is not saved without a name

- **Given** a manager is adding a warehouse
- **When** the manager leaves the name blank and saves
- **Then** the system gives a validation error
- **And** no warehouse is saved



#### Scenario: Warehouse is not saved when max days is not more than min days

- **Given** a manager is adding a warehouse
- **When** the manager enters min days 30 and max days 30
- **Then** the system shows "Max days must be more than min days"
- **And** no warehouse is saved



#### Scenario: Duplicate warehouse number is rejected

- **Given** warehouse "WH1" exists
- **When** the manager adds another warehouse with number "WH1"
- **Then** the system gives a duplicate error



### US-3.2 — Edit a warehouse



#### Scenario: Manager changes the order send time

- **Given** warehouse "WH1" has order send time 4:00 PM
- **When** the manager changes it to 2:30 PM and saves
- **Then** warehouse "WH1" shows order send time 2:30 PM



#### Scenario: Warehouse number cannot be changed

- **Given** warehouse "WH1" exists
- **When** the manager tries to change its number to "WH9"
- **Then** the system gives a validation error
- **And** the number stays "WH1"



### US-3.3 — See the list of warehouses



#### Scenario: Warehouse list is sorted by number

- **Given** warehouses "WH2" and "WH1" exist
- **When** the manager opens Warehouses
- **Then** "WH1" is listed before "WH2"

