# Feature: Item Maintenance

**Feature ID:** 6
**Branch pattern:** `feature/6-item-maintenance`
**Status:** Draft
**Created:** 2026-09-28
**Input:** Add and edit the Eagle Wholesale Items
**Depends on:** [Feature 1 — User Authorization](feature-1-user-authorization.md), [Feature 5 — Suppliers](feature-5-supplier-maintenance.md)

---

## User Stories

### US-6.1: Add an item

**As an** office staff member
**I want to** add an item with its Stock Keeping Unit (SKU), Universal Product Code (UPC), description, prices, case size, and supplier
**So that** we can stock it, order it from the supplier, and sell it to stores

**Priority:** P1
**Independent test:** Add an item with good data; looking up its UPC finds it
**Acceptance scenarios:** see ### US-6.1 under Acceptance Criteria

### US-6.2: Edit an item

**As an** office staff member
**I want to** change an item's price, description, case size, or supplier
**So that** new orders use current information

**Priority:** P1
**Independent test:** Change the unit price; the item shows the new price
**Acceptance scenarios:** see ### US-6.2 under Acceptance Criteria

### US-6.3: Find an item

**As a** warehouse staff member or office staff member
**I want to** find an item by scanning or typing its UPC or SKU, or by searching its description
**So that** I can answer questions and do my work quickly

**Priority:** P1
**Independent test:** Scan UPC "012345678905"; item "CH-1001" is shown
**Acceptance scenarios:** see ### US-6.3 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: An item MUST have: **SKU** (our item number, unique, up to 20 letters/numbers), **UPC** (unique, 12 or 13 digits), description, **unit price**, **case price**, **units per case** (1 or more), **supplier**, **supplier item #**, and **case cost**.
- **FR-002**: An item MAY have a category: Grocery, Candy, Snacks, Beauty, Auto, or Other.
- **FR-003**: Unit price, case price, and case cost MUST be more than 0.
- **FR-004**: The UPC MUST be digits only, 12 or 13 digits long, with a correct check digit (the last digit that proves the barcode is valid).
- **FR-005**: A used SKU or UPC MUST give a duplicate error. Missing or bad data MUST give a validation error.
- **FR-006**: Each item has exactly **one** supplier, and it must be a saved supplier (Feature 5).
- **FR-007**: The SKU MUST NOT change after it is saved. The UPC can change (for example, new packaging).
- **FR-008**: Changing a price only affects orders typed in **after** the change. Old order lines keep their price (Feature 7).
- **FR-009**: Units per case MUST NOT be changed while the item is on an open supplier order or an unshipped store order. Doing so gives a wrong status error.
- **FR-010**: Find MUST work by exact UPC or SKU (scanned or typed) and by part of the description. The item list shows 50 items per page.
- **FR-011**: Manager and Office Staff MAY add and edit items. Warehouse Staff MAY find and view items.

---

## Assumptions

- The same case size is used for buying from the supplier and selling by the case.
- The case price is set by hand (it can include a small discount); it is not calculated from the unit price.
- There is no price history. The item has one current price.
- A barcode scanner types the UPC like a keyboard, so no special hardware code is needed.
- Removing an item is not in this version.

---

## Edge Cases

- UPC with a wrong check digit → validation error
- Units per case = 0 → validation error
- Supplier that does not exist → validation error
- Find with a UPC that is not in the system → not found error
- Search that matches more than 50 items → shown on more than one page

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: Finding an item by UPC or SKU takes under 1 second with 10,000 items

---

## Data Ownership & Isolation

- Items belong to the company. Every warehouse uses the same item list.

---

## Key Entities

- **Item**: a product we buy by the case and sell by the case or unit. Comes from one supplier.

---

## Acceptance Criteria (Gherkin)

### US-6.1 — Add an item

#### Scenario: Office staff adds an item with good data

- **Given** supplier "S100" exists
- **When** the employee adds SKU "CH-1001", UPC "012345678905", "Cheddar Chips 2oz", unit price 0.89, case price 19.99, 24 units per case, supplier "S100", supplier item "F-2231", case cost 14.40
- **Then** the item is saved
- **And** it shows up in the item list

#### Scenario: Item is not saved with a bad UPC

- **Given** office staff is adding an item
- **When** the employee enters UPC "012345678901"
- **Then** the system shows "UPC is not valid"
- **And** no item is saved

#### Scenario: Item is not saved with zero units per case

- **Given** office staff is adding an item
- **When** the employee enters 0 units per case
- **Then** the system gives a validation error

#### Scenario: Duplicate UPC is rejected

- **Given** an item with UPC "012345678905" exists
- **When** the employee adds another item with UPC "012345678905"
- **Then** the system gives a duplicate error

#### Scenario: Duplicate SKU is rejected

- **Given** an item with SKU "CH-1001" exists
- **When** the employee adds another item with SKU "CH-1001"
- **Then** the system gives a duplicate error

### US-6.2 — Edit an item

#### Scenario: Office staff changes an item's unit price

- **Given** "CH-1001" has unit price 0.89
- **When** the employee changes it to 0.95 and saves
- **Then** "CH-1001" shows unit price 0.95

#### Scenario: Units per case cannot change while the item is on order

- **Given** "CH-1001" is on a Sent supplier order
- **When** the employee changes units per case from 24 to 12
- **Then** the system gives a wrong status error
- **And** units per case stays 24

### US-6.3 — Find an item

#### Scenario: Warehouse staff scans a UPC

- **Given** "CH-1001" has UPC "012345678905"
- **When** warehouse staff scans "012345678905" on Find Item
- **Then** "CH-1001" "Cheddar Chips 2oz" is shown

#### Scenario: Item that is not in the system

- **Given** no item has SKU or UPC "999"
- **When** the employee looks up "999"
- **Then** the system gives a not found error

#### Scenario: Search items by description

- **Given** items "Cheddar Chips 2oz" and "Cherry Cola Candy" exist
- **When** the employee searches for "chips"
- **Then** only "Cheddar Chips 2oz" is listed
