# Feature: Suppliers and Supplier Orders

**Feature ID:** 5
**Branch pattern:** `feature/5-supplier-maintenance`
**Status:** Draft
**Created:** 2026-09-28
**Input:** Keep a list of the suppliers Eagle Wholesale buys from, order from them automatically when stock gets low (the min/max rule), email the orders, and check in the deliveries when they arrive.
**Depends on:** [Feature 1 — User Authorization](feature-1-user-authorization.md). Supplier orders (US-5.4 to US-5.9) also need [Feature 6 — Items](feature-6-item-maintenance.md) and [Feature 9 — Inventory Management](feature-9-inventory-management.md).

---

## User Stories



### US-5.1: Add a supplier

**As an** office staff member
**I want to** add a supplier with their contact details 
**So that** we can buy items from it and email orders to it

**Priority:** P1
**Independent test:** Add a supplier with good data; it shows up in the supplier list
**Acceptance scenarios:** see ### US-5.1 under Acceptance Criteria

### US-5.2: Edit a supplier

**As an** office staff member
**I want to** edit a supplier's details
**So that** orders go to the right address and email

**Priority:** P1
**Independent test:** Change a supplier's email; the new email is shown
**Acceptance scenarios:** see ### US-5.2 under Acceptance Criteria

### US-5.3: Find a supplier

**As an** office staff member
**I want to** search suppliers by number or name
**So that** I can quickly open the one I need

**Priority:** P2
**Independent test:** Search "fri"; "Frito Distributors" is listed
**Acceptance scenarios:** see ### US-5.3 under Acceptance Criteria

### US-5.4: Reorder automatically when stock gets low

**As the** system
**I want to** check an item every time its stock changes, and order more from its supplier when available drops below min
**So that** the warehouse does not run out of things stores want

**Priority:** P1
**Independent test:** An item with available 10, min 15, max 50, and 24 per case gets 2 cases added to its supplier's draft order
**Acceptance scenarios:** see ### US-5.4 under Acceptance Criteria

### US-5.5: Create or change a supplier order by hand

**As an** office staff member
**I want to** start a supplier order and add, change, or remove items (in cases) before it is sent
**So that** I can order extra when I know something big is coming

**Priority:** P1
**Independent test:** Add 4 cases of a 24-per-case item to a draft order; "on order" for that item goes up by 96
**Acceptance scenarios:** see ### US-5.5 under Acceptance Criteria

### US-5.6: Email supplier orders once a day

**As an** office staff member
**I want** all draft orders emailed to their suppliers at the warehouse's order send time
**So that** each supplier gets one order a day without us faxing it

**Priority:** P1
**Independent test:** At 4:00 PM, every draft order with at least one line is emailed and marked Sent
**Acceptance scenarios:** see ### US-5.6 under Acceptance Criteria

### US-5.7: Cancel a supplier order

**As an** office staff member
**I want to** cancel a supplier order that should not be filled
**So that** "on order" numbers stay correct

**Priority:** P2
**Independent test:** Cancel a Sent order; its status is Cancelled and "on order" goes down
**Acceptance scenarios:** see ### US-5.7 under Acceptance Criteria

### US-5.8: Receive a supplier delivery

**As a** warehouse staff member
**I want to** type in the PO number from the delivery paperwork, scan each item and enter how many cases came, see what is different from the order, and then finish receiving
**So that** stock on hand matches what actually arrived

**Priority:** P1
**Independent test:** Receive 4 of 4 cases on one line and 1 of 2 on another; on hand goes up by what came and the order is Received
**Acceptance scenarios:** see ### US-5.8 under Acceptance Criteria

### US-5.9: See which suppliers short us

**As a** manager
**I want** a report of ordered vs. received for each supplier
**So that** I can decide if we should change suppliers

**Priority:** P2
**Independent test:** A supplier that sent 120 of 156 units shows 76.9% filled
**Acceptance scenarios:** see ### US-5.9 under Acceptance Criteria

---

## Requirements



### Functional Requirements

**Supplier info**

- **FR-001**: A supplier MUST have: supply number (unique, up to 10 letters/numbers), name, street, city, state, zip code, phone, and email.
- **FR-002**: A supplier MAY have: contact name, fax, the supplier's own warehouse number (free text), and lead time in days.
- **FR-003**: Email is required because orders are emailed. It must look like a real email address.
- **FR-004**: A used supply number MUST give a duplicate error. Missing or bad data MUST give a validation error.
- **FR-005**: The supply number MUST NOT change after it is saved.

**Supplier orders**

- **FR-006**: A supplier order MUST have: a PO number made by the system (like `PO-000123`), warehouse, supplier, status, date created, and who created it (an employee or "System").
- **FR-007**: The statuses are: **Draft** → **Sent** → **Received**. An order can also be **Send Failed** (email did not go through) or **Cancelled**.
- **FR-008**: There is **only one Draft order per supplier per warehouse**. New lines for that supplier go onto that draft.
- **FR-009**: Each line has an item, a number of **whole cases** (1 or more), and the case cost at the time it was added. Units = cases × units per case.
- **FR-010**: An item can only be added if that supplier is the item's supplier and the item is stocked in that warehouse (Feature 9).
- **FR-011**: Lines can only be changed while the order is Draft.
- **FR-012**: "On order" for an item = the units on all Draft, Sent, and Send Failed orders for that warehouse.

**Automatic reorder**

- **FR-013**: Every time an item's stock changes (received, shipped, adjusted, ordered by a store, or min/max changed), the system MUST check the reorder rule:
  - If **available < min**, then units needed = **max − available**, and cases = units needed ÷ units per case, **rounded up**.
  - Those cases are added to the supplier's Draft order (a new Draft is made if there isn't one).
- **FR-014**: Because draft lines count as "on order", checking the same item again right away MUST NOT order it twice.

**Sending**

- **FR-015**: At each warehouse's order send time (Feature 3), the system MUST email every Draft order that has at least one line, plus every Send Failed order, to the supplier's email.
- **FR-016**: The email MUST include the supplier order form: company name and address, PO number and date, ship-to warehouse, supplier, and each line (supplier item #, SKU, description, cases, case cost, line cost) with the total. Replies go to the company email.
- **FR-017**: If the email goes through, the order becomes Sent. If not, it becomes Send Failed and is tried again at the next send time. An order is never emailed twice.

**Cancel**

- **FR-018**: Draft, Sent, and Send Failed orders MAY be cancelled with a reason. Received orders cannot. Cancelling removes the order from "on order".

**Receiving**

- **FR-019**: Only Sent orders for the worker's warehouse can be received. The worker finds the order by typing the PO number from the delivery paperwork.
- **FR-020**: The worker scans or types each item and enters the number of cases (default 1 each scan; scanning the same item again adds to it).
- **FR-021**: When the worker selects Done, the system MUST show each line with ordered, received, and difference. The worker checks each difference and can fix a count before finishing. Lines never scanned count as 0 received.
- **FR-022**: Finishing MUST: add the received units to on hand (Feature 9), take the whole order off "on order", and set status Received. It happens all at once or not at all.
- **FR-023**: An order is received **one time only**. Mistakes after that are fixed with an inventory adjustment (Feature 9).
- **FR-024**: Scanning an item that is not on the order MUST show "Not on this order" and not add it.

**Report**

- **FR-025**: The supplier report MUST show, for a warehouse and date range, every received order line where received ≠ ordered, plus a summary per supplier: orders, lines short, units ordered, units received, and **% filled = received ÷ ordered**.

**Who can do what**

- **FR-026**: Manager and Office Staff MAY manage suppliers and supplier orders. Manager and Warehouse Staff MAY receive. Warehouse Staff MAY view suppliers and orders.

---

## Assumptions

- Each item has one supplier (Feature 6).
- Suppliers deliver each order in one trip.
- We do not track where received goods are put on the shelves.
- Paying suppliers is not part of this system.
- Removing a supplier is not in this version.

---

## Edge Cases

- Available exactly equal to min → no reorder (must be **less than** min)
- Need fewer units than one case → order 1 case
- Adding an item from a different supplier → validation error
- 0 cases on a line → validation error
- Changing lines on a Sent order → wrong status error
- Receiving an order that is not Sent → wrong status error
- Receiving the same order twice → wrong status error
- Draft order with no lines at send time → not emailed, stays Draft

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: "On order" always equals the units on open orders
- **SC-003**: Each supplier gets at most one new order email per warehouse per day
- **SC-004**: After receiving, on hand went up by exactly what was received

---

## Data Ownership & Isolation

- Suppliers belong to the company (all warehouses share them).
- Supplier orders belong to one warehouse. Non-managers only see their own warehouse's orders.

---

## Key Entities

- **Supplier**: a company we buy from
- **Supplier Order**: an order to one supplier for one warehouse (also called a PO)
- **Supplier Order Line**: one item and number of cases on a supplier order
- **Supplier Order Form**: the printable/emailed copy of the order

---

## Acceptance Criteria (Gherkin)



### US-5.1 — Add a supplier



#### Scenario: Office staff adds a supplier with good data

- **Given** office staff is logged in
- **When** the employee adds supplier "S100" "Frito Distributors" with address, phone "405-555-0111", and email "[orders@frito.example](mailto:orders@frito.example)"
- **Then** the supplier is saved
- **And** "S100" shows up in the supplier list



#### Scenario: Supplier is not saved without an email

- **Given** office staff is adding a supplier
- **When** the employee leaves the email blank and saves
- **Then** the system shows "Email is required"
- **And** no supplier is saved



#### Scenario: Duplicate supply number is rejected

- **Given** supplier "S100" exists
- **When** the employee adds another supplier with number "S100"
- **Then** the system gives a duplicate error



#### Scenario: Warehouse staff cannot add suppliers

- **Given** warehouse staff is logged in
- **When** the employee tries to add a supplier
- **Then** the system gives a not allowed error



### US-5.2 — Edit a supplier



#### Scenario: Office staff changes a supplier's email

- **Given** supplier "S100" has email "[orders@frito.example](mailto:orders@frito.example)"
- **When** the employee changes it to "[po@frito.example](mailto:po@frito.example)" and saves
- **Then** "S100" shows email "[po@frito.example](mailto:po@frito.example)"



#### Scenario: Edit is not saved when the name is cleared

- **Given** supplier "S100" exists
- **When** the employee clears the name and saves
- **Then** the system gives a validation error
- **And** "S100" keeps its old name



### US-5.3 — Find a supplier



#### Scenario: Search suppliers by part of the name

- **Given** suppliers "Frito Distributors" and "Acme Auto Parts" exist
- **When** the employee searches for "fri"
- **Then** only "Frito Distributors" is listed



### US-5.4 — Reorder automatically when stock gets low



#### Scenario: Low item is added to the supplier's draft in whole cases

- **Given** item "CH-1001" (24 per case, supplier "S100") in "WH1" has on hand 30, on order 0, committed 20, min 15, max 50
- **When** its stock changes
- **Then** the Draft order for "S100" and "WH1" gets 2 cases of "CH-1001"
- **And** "on order" for "CH-1001" becomes 48



#### Scenario: Item at min is not reordered

- **Given** "CH-1001" in "WH1" has available 15 and min 15
- **When** its stock changes
- **Then** nothing is added to any supplier order



#### Scenario: Checking again does not order twice

- **Given** "CH-1001" was just reordered and has 48 units on the draft
- **When** its stock is checked again with no other change
- **Then** the draft still has 2 cases of "CH-1001"



#### Scenario: Two low items from the same supplier share one draft

- **Given** "CH-1001" and "CH-1002" both come from "S100" and both drop below min in "WH1"
- **When** each item is checked
- **Then** there is one Draft order for "S100" and "WH1" with both items on it



### US-5.5 — Create or change a supplier order by hand



#### Scenario: Office staff adds cases to a draft order

- **Given** "CH-1001" (24 per case, case cost 14.40) comes from "S100" and is stocked in "WH1"
- **And** "WH1" has a Draft order for "S100"
- **When** the employee adds 4 cases of "CH-1001"
- **Then** the line shows 4 cases, 96 units, and line cost 57.60
- **And** "on order" for "CH-1001" in "WH1" goes up by 96



#### Scenario: Item from another supplier cannot be added

- **Given** item "SH-2002" comes from supplier "S300"
- **When** the employee adds "SH-2002" to an order for "S100"
- **Then** the system gives a validation error



#### Scenario: Sent order cannot be changed

- **Given** order "PO-000118" is Sent
- **When** the employee changes a line
- **Then** the system gives a wrong status error



### US-5.6 — Email supplier orders once a day



#### Scenario: Draft orders are emailed at the send time

- **Given** "WH1" has order send time 4:00 PM
- **And** Draft orders for "S100" and "S300" each have at least one line
- **When** it is 4:00 PM
- **Then** each order is emailed to its supplier with the supplier order form attached
- **And** both orders become Sent



#### Scenario: Empty draft is not emailed

- **Given** the Draft order for "S200" in "WH1" has no lines
- **When** the 4:00 PM send runs
- **Then** no email is sent for it
- **And** it stays Draft



#### Scenario: Failed email is tried again the next day

- **Given** the email for "PO-000140" does not go through
- **When** the 4:00 PM send runs
- **Then** "PO-000140" becomes Send Failed
- **And** it is emailed again at the next send time



### US-5.7 — Cancel a supplier order



#### Scenario: Office staff cancels a sent order

- **Given** Sent order "PO-000118" has 96 units of "CH-1001" for "WH1"
- **When** the employee cancels it with reason "Supplier out of stock"
- **Then** the order becomes Cancelled
- **And** "on order" for "CH-1001" goes down by 96



#### Scenario: Received order cannot be cancelled

- **Given** order "PO-000110" is Received
- **When** the employee tries to cancel it
- **Then** the system gives a wrong status error



### US-5.8 — Receive a supplier delivery



#### Scenario: Worker receives a complete delivery

- **Given** Sent order "PO-000118" has 4 cases of "CH-1001" (24 per case)
- **And** "CH-1001" in "WH1" has 10 on hand and 96 on order
- **When** the worker enters "PO-000118", scans "CH-1001" and enters 4 cases, selects Done, and finishes
- **Then** "CH-1001" has 106 on hand and 0 on order
- **And** "PO-000118" is Received



#### Scenario: Worker sees differences before finishing

- **Given** "PO-000122" ordered 4 cases of "CH-1001" and 3 cases of "CH-1003"
- **When** the worker scans 4 cases of "CH-1001" and 5 cases of "CH-1003" and selects Done
- **Then** the system shows "CH-1001" difference 0 and "CH-1003" difference +2



#### Scenario: Short delivery can cause a new reorder

- **Given** Sent order "PO-000121" has 5 cases of "CH-1002" (12 per case)
- **And** "CH-1002" has min 50, max 100, on hand 0, and nothing committed
- **When** the worker receives only 2 cases and finishes
- **Then** "CH-1002" has 24 on hand
- **And** "CH-1002" is added to a new draft order for its supplier



#### Scenario: Item not on the order is refused

- **Given** the worker is receiving "PO-000118"
- **When** the worker scans "AU-3003", which is not on that order
- **Then** the system shows "Not on this order"



#### Scenario: An order cannot be received twice

- **Given** "PO-000118" is Received
- **When** a worker tries to receive it again
- **Then** the system gives a wrong status error



### US-5.9 — See which suppliers short us



#### Scenario: Supplier report shows short lines and % filled

- **Given** in "WH1" this month, "S100" orders were received: "PO-000118" (4 of 4 cases, 24 per case) and "PO-000121" (2 of 5 cases, 12 per case)
- **When** the manager runs the supplier report for "WH1" for this month
- **Then** one line is listed: "PO-000121" ordered 60, received 24, difference -36
- **And** "S100" shows units ordered 156, units received 120, 76.9% filled

