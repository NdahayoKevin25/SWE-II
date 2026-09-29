# Feature: Customers and Customer Orders

**Feature ID:** 7
**Branch pattern:** `feature/7-customer-maintenance`
**Status:** Draft
**Created:** 2026-09-28
**Input:** Keep a list of customers we sell to, their orders, pick their orders, package them, and ship them with a bill of lading.
**Depends on:** [Feature 1 — User Authorization](feature-1-user-authorization.md), [Feature 3 — Warehouse Maintenance](feature-3-warehouse-maintenance.md). Store orders (US-7.3 to US-7.8) also need [Feature 6 — Items](feature-6-item-maintenance.md), [Feature 8 — Routes](feature-8-route-maintenance.md), and [Feature 9 — Inventory Management](feature-9-inventory-management.md).

---

## User Stories

### US-7.1: Add a store

**As an** office staff member
**I want to** add a store with its delivery address and contact
**So that** we can take and deliver its orders

**Priority:** P1
**Independent test:** Add a store with good data; it shows up in the store list for its warehouse
**Acceptance scenarios:** see ### US-7.1 under Acceptance Criteria

### US-7.2: Edit a store

**As an** office staff member
**I want to** edit a store's details
**So that** deliveries and paperwork use current information

**Priority:** P1
**Independent test:** Change the delivery instructions; the new text is shown
**Acceptance scenarios:** see ### US-7.2 under Acceptance Criteria

### US-7.3: Type in a store order

**As an** office staff member
**I want to** start an order for a store and add items by SKU or UPC with a number of cases or units
**So that** the store's paper order sheet is in the system with the right prices

**Priority:** P1
**Independent test:** Add 3 cases and 6 units of an item; the totals use the case price and the unit price
**Acceptance scenarios:** see ### US-7.3 under Acceptance Criteria

### US-7.4: Change or cancel a store order

**As an** office staff member
**I want to** change lines before the order goes to the warehouse, and cancel an order any time before it ships
**So that** I can fix typing mistakes and store changes

**Priority:** P1
**Independent test:** Change a line from 3 cases to 2; the total and "committed" both go down
**Acceptance scenarios:** see ### US-7.4 under Acceptance Criteria

### US-7.5: Send an order to the warehouse

**As an** office staff member
**I want to** release a finished order to the warehouse floor
**So that** it shows up on the pick list

**Priority:** P1
**Independent test:** Release an order; it is Released and appears on the warehouse's Pick screen
**Acceptance scenarios:** see ### US-7.5 under Acceptance Criteria

### US-7.6: Pick an order

**As a** warehouse staff member
**I want to** take the next order from the pick list, walk the aisles, and enter how much of each line I actually picked
**So that** the order ships with what we really have, and anything short is cancelled

**Priority:** P1
**Independent test:** Pick 2 of 3 cases on one line; the order is Picked and the line shows 1 case short
**Acceptance scenarios:** see ### US-7.6 under Acceptance Criteria

### US-7.7: Check, package, and ship an order

**As a** warehouse staff member (shipping)
**I want to** check the picked order, enter the number of packages, pick the route, driver, and delivery date, and print the bill of lading
**So that** the order goes on the right truck and stock goes down

**Priority:** P1
**Independent test:** Ship an order; it is Shipped, on hand goes down, and a bill of lading prints
**Acceptance scenarios:** see ### US-7.7 under Acceptance Criteria

### US-7.8: See what we ran out of and what stores bought

**As a** manager
**I want** a report of every line we picked short, and a report of sales by store
**So that** I can fix min/max on items we keep running out of and see which stores buy the most

**Priority:** P2
**Independent test:** An item short on 3 orders shows "short 3 times"; a store with 2 shipped orders shows both in its total
**Acceptance scenarios:** see ### US-7.8 under Acceptance Criteria

---

## Requirements

### Functional Requirements

**Store info**

- **FR-001**: A store MUST have: customer number (unique, up to 10 letters/numbers), store name, delivery street, city, state, zip code, phone, and the warehouse that serves it.
- **FR-002**: A store MAY have: contact name, email, and delivery instructions (for example, "back door before 10am").
- **FR-003**: A used customer number MUST give a duplicate error. Missing or bad data MUST give a validation error.
- **FR-004**: The customer number MUST NOT change after it is saved. Only a manager MAY change which warehouse serves a store.

**Store orders**

- **FR-005**: A store order MUST have: an order number made by the system (like `SO-000001`), the store, the store's warehouse, order date, and who typed it in. It MAY have the store's own sheet number and a requested delivery date (today or later).
- **FR-006**: Order statuses go in this order: **Entered → Released → Picked → Shipped → Delivered**. An order can be **Cancelled** any time before Shipped. Delivered is set in Feature 8.
- **FR-007**: Each line has an item, **Case** or **Each**, and a quantity (whole number, 1 or more). Units = quantity × units per case for Case, or just the quantity for Each.
- **FR-008**: The line MUST save the price **at the time it is typed in**: case price for Case, unit price for Each. Line total = quantity × price. Order total = sum of line totals.
- **FR-009**: The item MUST be stocked in the order's warehouse (Feature 9).
- **FR-010**: Orders MUST be accepted even if we don't have enough stock. Shortages are handled when picking.
- **FR-011**: Lines can only be changed while the order is Entered. Release needs at least one line.
- **FR-012**: "Committed" for an item = units on Entered and Released orders + picked units on Picked orders. Any change to committed counts as a stock change (Feature 5 reorder check).
- **FR-013**: Cancelling needs a reason and removes the order from committed.
- **FR-014**: The customer order form (printable) MUST show the company, warehouse, order number, dates, store and delivery address, each line, and the total.

**Picking**

- **FR-015**: The pick list MUST show Released orders for the worker's warehouse, oldest requested date first.
- **FR-016**: Starting a pick marks the order "being picked by" that worker. Another worker cannot pick the same order at the same time.
- **FR-017**: The worker MUST enter a picked quantity for **every** line, from 0 up to the ordered quantity. **Short = ordered − picked**, and the short amount is cancelled (no backorder).
- **FR-018**: When finished, the order becomes Picked. If **nothing** was picked on any line, the order becomes Cancelled with reason "Nothing picked".

**Shipping**

- **FR-019**: Only Picked orders can be shipped. First the shipping worker checks each line and can fix the picked quantity if the cart does not match.
- **FR-020**: The worker MUST enter the number of packages (1 or more), a route, a driver (from that warehouse), and a delivery date (today or later). The store's route and that route's driver are filled in for them.
- **FR-021**: Shipping MUST, all at once: take the shipped units off on hand (Feature 9), remove the order from committed, set it to Shipped, and create the bill of lading.
- **FR-022**: If on hand is less than what is being shipped, the system MUST stop with a message naming the item. The worker fixes the count first (Feature 9).
- **FR-023**: The **bill of lading** MUST have its own number and show: company and warehouse, store name and delivery address, delivery instructions, order number, ship date, delivery date, route, driver, number of packages, each line (ordered, shipped, short), and signature lines for the driver and the store.

**Reports**

- **FR-024**: The **short picks report** MUST list, for a warehouse and date range, every order line picked short (order, store, item, ordered, picked, short), plus a summary per item: times short, units short, and the item's current min and max.
- **FR-025**: The **sales by store report** MUST show, for a warehouse and date range, each store's number of shipped orders, units shipped, and sales total (shipped quantity × line price).

**Who can do what**

- **FR-026**: Manager and Office Staff MAY manage stores and type in, change, release, and cancel orders. Manager and Warehouse Staff MAY pick and ship. Reports are for Manager and Office Staff. Non-managers only see their own warehouse.

---

## Assumptions

- Stores send paper order sheets. Stores do not log in.
- Every store pays the same price. No invoices or payments.
- We do not track aisle locations, so the pick list is sorted by SKU.
- Removing a store is not in this version.

---

## Edge Cases

- Item not stocked in the warehouse → validation error
- Quantity 0 or less → validation error
- Changing lines on a Released order → wrong status error
- Cancelling a Shipped order → wrong status error
- Picked more than ordered → validation error
- A second worker starts a pick that is already in progress → wrong status error naming the first worker
- Shipping with 0 packages → validation error
- Shipping with a driver from another warehouse → validation error

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A 30-line order sheet can be typed in using only the keyboard
- **SC-003**: Line prices never change after they are typed in
- **SC-004**: After shipping, on hand went down by exactly what shipped

---

## Data Ownership & Isolation

- Stores and their orders belong to one warehouse. Non-managers only see their own warehouse.

---

## Key Entities

- **Customer (Store)**: a convenience store that buys from us. Served by one warehouse.
- **Customer Order**: one store's order. Moves Entered → Released → Picked → Shipped → Delivered.
- **Customer Order Line**: an item, Case or Each, quantity, and saved price
- **Bill of Lading**: the paper that goes with the order on the truck

---

## Acceptance Criteria (Gherkin)

### US-7.1 — Add a store

#### Scenario: Office staff adds a store with good data

- **Given** office staff from warehouse "WH1" is logged in
- **When** the employee adds store "C001" "Quick Stop #4" with delivery address and phone
- **Then** the store is saved with warehouse "WH1"
- **And** "C001" shows up in the store list

#### Scenario: Store is not saved without a delivery address

- **Given** office staff is adding a store
- **When** the employee leaves the street blank and saves
- **Then** the system gives a validation error

#### Scenario: Duplicate customer number is rejected

- **Given** store "C001" exists
- **When** the employee adds another store with number "C001"
- **Then** the system gives a duplicate error

### US-7.2 — Edit a store

#### Scenario: Office staff edits delivery instructions

- **Given** store "C001" exists
- **When** the employee sets delivery instructions to "Back door before 10am" and saves
- **Then** "C001" shows the new instructions

#### Scenario: Office staff cannot move a store to another warehouse

- **Given** office staff from "WH1" is logged in
- **When** the employee changes the warehouse of "C001" to "WH2"
- **Then** the system gives a not allowed error

### US-7.3 — Type in a store order

#### Scenario: Office staff orders by case and by unit

- **Given** "CH-1001" has case price 19.99, unit price 0.89, 24 per case, and is stocked in "WH1"
- **And** store "C001" is served by "WH1"
- **When** the employee starts an order for "C001" and adds 3 cases and 6 each of "CH-1001"
- **Then** the order is Entered with a new order number
- **And** the lines are 59.97 and 5.34, and the order total is 65.31
- **And** committed for "CH-1001" goes up by 78 units

#### Scenario: Order is taken even when stock is low

- **Given** "CH-1001" in "WH1" has 10 on hand
- **When** the employee adds 2 cases of "CH-1001" to an order
- **Then** the line is added with no warning

#### Scenario: Item not stocked in the warehouse cannot be added

- **Given** item "AU-3003" is not stocked in "WH1"
- **When** the employee adds "AU-3003" to an order for "C001"
- **Then** the system gives a validation error

### US-7.4 — Change or cancel a store order

#### Scenario: Office staff lowers a line before release

- **Given** Entered order "SO-000010" has 3 cases of "CH-1001"
- **When** the employee changes it to 2 cases
- **Then** the line total is 39.98
- **And** committed for "CH-1001" goes down by 24

#### Scenario: Price change does not change typed-in lines

- **Given** Entered order "SO-000010" has 2 cases of "CH-1001" at 19.99
- **When** the case price of "CH-1001" changes to 21.00
- **Then** the line on "SO-000010" still shows 19.99

#### Scenario: Office staff cancels a released order

- **Given** Released order "SO-000011" commits 48 units of "CH-1001"
- **When** the employee cancels it with reason "Store closed"
- **Then** the order is Cancelled
- **And** committed for "CH-1001" goes down by 48

#### Scenario: Shipped order cannot be cancelled

- **Given** order "SO-000005" is Shipped
- **When** the employee tries to cancel it
- **Then** the system gives a wrong status error

### US-7.5 — Send an order to the warehouse

#### Scenario: Office staff releases an order

- **Given** Entered order "SO-000010" has at least one line
- **When** the employee releases it
- **Then** the order is Released
- **And** it shows on the Pick screen for "WH1"

#### Scenario: Empty order cannot be released

- **Given** Entered order "SO-000012" has no lines
- **When** the employee releases it
- **Then** the system gives a validation error

### US-7.6 — Pick an order

#### Scenario: Pick list shows the oldest order first

- **Given** "WH1" has Released orders "SO-000021" (wanted Oct 3) and "SO-000020" (wanted Oct 2)
- **When** warehouse staff opens Pick
- **Then** "SO-000020" is listed before "SO-000021"

#### Scenario: Worker picks an order with a short line

- **Given** "aruiz" is picking "SO-000020" with 3 cases of "CH-1001" and 6 each of "CH-1002"
- **When** "aruiz" enters 2 cases for "CH-1001" and 6 each for "CH-1002" and finishes
- **Then** "SO-000020" is Picked
- **And** the "CH-1001" line shows picked 2, short 1

#### Scenario: Two workers cannot pick the same order

- **Given** "aruiz" has started picking "SO-000020"
- **When** "jlee" tries to start picking "SO-000020"
- **Then** the system gives a wrong status error naming "aruiz"

#### Scenario: Order with nothing picked is cancelled

- **Given** "SO-000023" has one line of 2 cases of "CH-1005"
- **When** the worker enters 0 picked and finishes
- **Then** "SO-000023" is Cancelled with reason "Nothing picked"

### US-7.7 — Check, package, and ship an order

#### Scenario: Worker ships an order

- **Given** Picked order "SO-000020" has 48 units of "CH-1001" picked
- **And** "CH-1001" in "WH1" has 100 on hand
- **When** the worker checks the order, enters 2 packages, route "N1", driver "D7", delivery date Oct 2, and ships
- **Then** "SO-000020" is Shipped
- **And** "CH-1001" has 52 on hand
- **And** a bill of lading prints with its own number, the store's address, route "N1", driver "D7", 2 packages, and signature lines

#### Scenario: Worker fixes a picked quantity while checking

- **Given** Picked order "SO-000020" shows 2 cases of "CH-1001" picked
- **When** the worker finds only 1 case on the cart and changes it to 1
- **Then** the line shows picked 1, short 2

#### Scenario: Route and driver are filled in from the store

- **Given** store "C001" is on route "N1" whose driver is "D7"
- **When** the worker gets to the ship step for an order for "C001"
- **Then** route "N1" and driver "D7" are already filled in

#### Scenario: Order cannot ship when on hand is too low

- **Given** Picked order "SO-000025" has 30 units of "CH-1006" and "CH-1006" has 20 on hand
- **When** the worker ships it
- **Then** the system stops with a message naming "CH-1006"
- **And** on hand stays 20

### US-7.8 — See what we ran out of and what stores bought

#### Scenario: Short picks report groups by item

- **Given** this month "CH-1005" was picked short on 3 orders by 24, 12, and 24 units
- **And** "CH-1005" has min 20 and max 60
- **When** the manager runs the short picks report for "WH1"
- **Then** it lists 3 lines for "CH-1005"
- **And** the summary shows "CH-1005" short 3 times, 60 units, min 20, max 60

#### Scenario: Sales by store adds up shipped orders

- **Given** this month store "C001" had shipped orders of 19.99 and 45.32
- **When** the manager runs sales by store for "WH1"
- **Then** "C001" shows 2 orders and a total of 65.31
