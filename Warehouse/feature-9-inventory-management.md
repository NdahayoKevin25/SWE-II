# Feature: Inventory Management

**Feature ID:** 9
**Branch pattern:** `feature/9-inventory-management`
**Status:** Draft
**Created:** 2026-09-28
**Input:** Track how many units of each item are in each warehouse. Set each item's min and max (with suggestions based on how fast it sells), fix counts, keep a history of every change, and show stock reports.
**Depends on:** [Feature 3 — Warehouse Maintenance](feature-3-warehouse-maintenance.md), [Feature 6 — Items](feature-6-item-maintenance.md). Suggestions and sales numbers (US-9.6, US-9.7) need store orders from [Feature 7](feature-7-customer-maintenance.md).

---

## User Stories



### US-9.1: Stock an item in a warehouse

**As a** manager
**I want to** add an item to a warehouse with its starting quantity, min, and max
**So that** the system tracks it there and reorders it when it gets low

**Priority:** P1
**Independent test:** Stock "CH-1001" in "WH1" with 40 on hand, min 15, max 50; it shows 40 on hand
**Acceptance scenarios:** see ### US-9.1 under Acceptance Criteria

### US-9.2: Change min and max

**As a** manager
**I want to** change an item's min and max in a warehouse
**So that** reorder levels match how fast it sells

**Priority:** P1
**Independent test:** Change min and max; the new values are shown and the reorder check runs
**Acceptance scenarios:** see ### US-9.2 under Acceptance Criteria

### US-9.3: Adjust stock with a reason

**As a** warehouse staff member
**I want to** add or take away units with a reason (damaged, lost, found, count fix, other)
**So that** the system matches what is really on the shelf

**Priority:** P1
**Independent test:** Take away 3 as Damaged; on hand goes down by 3 and the history shows it
**Acceptance scenarios:** see ### US-9.3 under Acceptance Criteria

### US-9.4: Count an item

**As a** warehouse staff member
**I want to** type in how many I counted on the shelf and let the system work out the difference
**So that** I don't have to do the math myself

**Priority:** P2
**Independent test:** System says 40, I count 37; the system posts a -3 "count fix" adjustment
**Acceptance scenarios:** see ### US-9.4 under Acceptance Criteria

### US-9.5: See stock and its history

**As an** employee
**I want to** see on hand, on order, committed, available, min, and max, and every change that happened
**So that** I can answer "how many do we have, and what happened to them?"

**Priority:** P1
**Independent test:** After two adjustments, the history shows both with the running total
**Acceptance scenarios:** see ### US-9.5 under Acceptance Criteria

### US-9.6: Get suggested min and max

**As a** manager
**I want** the system to suggest min and max from how many units stores ordered in the last 60 days
**So that** I can keep reorder levels right without doing the math for 10,000 items

**Priority:** P2
**Independent test:** 900 units ordered in 60 days with settings 14/60 days suggests min 210 and max 900
**Acceptance scenarios:** see ### US-9.6 under Acceptance Criteria

### US-9.7: Stock and sales reports

**As a** manager
**I want** a stock status report (what we have vs. min/max, and what it is worth) and a sales by item report
**So that** I know where each warehouse stands and what sells

**Priority:** P2
**Independent test:** The stock report flags items below min; sales by item adds up shipped lines
**Acceptance scenarios:** see ### US-9.7 under Acceptance Criteria

---

## Requirements



### Functional Requirements

**Stock records**

- **FR-001**: There MUST be one stock record per item per warehouse, with **on hand**, **min**, and **max**, all counted in single units.
- **FR-002**: Stocking the same item twice in one warehouse MUST give a duplicate error.
- **FR-003**: Min and max MUST be whole numbers where 0 ≤ min ≤ max and max is at least 1.
- **FR-004**: Stocking an item MUST save the starting quantity (0 or more) as the first history entry.

**The numbers**

- **FR-005**: For each record the system MUST show:
  - **on hand** — units on the shelves
  - **on order** — units on open supplier orders (Feature 5)
  - **committed** — units promised to stores but not shipped yet (Feature 7)
  - **available = on hand + on order − committed**
  - **below min** — yes if available < min
- **FR-006**: Any change to on hand, on order, committed, min, or max MUST trigger the reorder check in Feature 5.

**Changing on hand**

- **FR-007**: On hand only changes through a history entry. Nobody types over the on hand number directly.
- **FR-008**: An adjustment MUST have a quantity (not 0, plus or minus), a reason (Count Fix, Damaged, Lost, Found, Other), and a note if the reason is Other.
- **FR-009**: An adjustment MUST NOT make on hand go below 0.
- **FR-010**: **Count** lets the worker enter the counted number. The system posts a Count Fix adjustment for the difference. If the count matches, nothing is posted.
- **FR-011**: Every history entry MUST save: type (Starting, Adjustment, Received, Shipped), quantity, new on hand, date and time, employee, reason or note, and the order number when there is one. History entries can never be changed or deleted.

**Suggestions**

- **FR-012**: **Demand** = units ordered by stores for that item and warehouse in the last 60 days. Orders cancelled by office staff do not count. Orders cancelled because "Nothing picked" DO count (that was real demand we could not fill).
- **FR-013**: **Per day** = demand ÷ 60. **Suggested min** = per day × min days of stock (Feature 3), rounded up. **Suggested max** = per day × max days of stock, rounded up (and always at least suggested min + 1).
- **FR-014**: Items with no demand show "No recent demand" and no suggestion.
- **FR-015**: The manager MAY accept one suggestion or a checked group. Accepting works like changing min/max by hand.

**Reports**

- **FR-016**: The **stock status report** MUST list each stocked item in a warehouse with on hand, on order, committed, available, min, max, below min, and **value = on hand × (case cost ÷ units per case)**. It can show only below-min items. Totals: item count, below-min count, total value.
- **FR-017**: The **sales by item report** MUST show, for a warehouse and date range, each item's units shipped, number of orders, and sales total (from shipped store orders, Feature 7).

**Who can do what**

- **FR-018**: Manager MAY stock items, change min/max, and accept suggestions. Manager and Warehouse Staff MAY adjust and count. Manager, Office Staff, and Warehouse Staff MAY view stock and history for their warehouse. Reports are for Manager and Office Staff.

---

## Assumptions

- We only track how many, not where on the shelves.
- Products do not expire, so no lot numbers or FIFO/LIFO.
- Until Features 5 and 7 are built, on order and committed are 0 (tests can fake them).
- The 60-day window matches "we keep about 2 months of inventory".
- Removing an item from a warehouse is not in this version.

---

## Edge Cases

- Adjustment of 0 → validation error
- Adjustment that would make on hand negative → validation error
- Reason Other with no note → validation error
- Min bigger than max → validation error
- Count that equals on hand → no adjustment is posted
- Office staff tries to adjust → not allowed error
- Warehouse staff opens another warehouse's stock → not found error

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: For every record, on hand equals the sum of its history entries
- **SC-003**: On hand is never negative
- **SC-004**: The stock status report for 10,000 items loads in under 5 seconds

---

## Data Ownership & Isolation

- Stock records and history belong to one warehouse. Non-managers only see their own warehouse.
- History records who made each change and cannot be edited.

---

## Key Entities

- **Stock Record**: one item in one warehouse, with on hand, min, and max
- **Stock History Entry**: one change to on hand (starting, adjustment, received, shipped)
- **Min/Max Suggestion**: a suggested min and max worked out from recent demand

---

## Acceptance Criteria (Gherkin)



### US-9.1 — Stock an item in a warehouse



#### Scenario: Manager stocks an item with good levels

- **Given** "CH-1001" is not stocked in "WH1"
- **When** the manager stocks it with starting quantity 40, min 15, and max 50
- **Then** the record is saved with 40 on hand
- **And** the history shows one Starting entry of 40



#### Scenario: Record is not saved when min is bigger than max

- **Given** "CH-1001" is not stocked in "WH1"
- **When** the manager stocks it with min 60 and max 50
- **Then** the system gives a validation error



#### Scenario: Stocking the same item twice is rejected

- **Given** "CH-1001" is already stocked in "WH1"
- **When** the manager stocks "CH-1001" in "WH1" again
- **Then** the system gives a duplicate error



### US-9.2 — Change min and max



#### Scenario: Manager changes min and max

- **Given** "CH-1001" in "WH1" has min 15 and max 50
- **When** the manager sets min 20 and max 80
- **Then** the record shows min 20 and max 80
- **And** the reorder check runs for "CH-1001"



### US-9.3 — Adjust stock with a reason



#### Scenario: Worker records damaged units

- **Given** "CH-1001" in "WH1" has 40 on hand
- **When** warehouse staff from "WH1" takes away 3 with reason "Damaged"
- **Then** on hand is 37
- **And** the history shows -3, "Damaged", and who did it



#### Scenario: Adjustment cannot make on hand negative

- **Given** "CH-1001" in "WH1" has 2 on hand
- **When** the worker takes away 5
- **Then** the system gives a validation error
- **And** on hand stays 2



#### Scenario: Reason Other needs a note

- **Given** a worker is adjusting "CH-1001"
- **When** the worker adds 4 with reason "Other" and no note
- **Then** the system gives a validation error



#### Scenario: Office staff cannot adjust stock

- **Given** office staff is logged in
- **When** the employee tries to adjust stock
- **Then** the system gives a not allowed error



### US-9.4 — Count an item



#### Scenario: Count lower than the system posts a count fix

- **Given** "CH-1001" in "WH1" has 40 on hand
- **When** the worker counts 37 and saves
- **Then** on hand is 37
- **And** the history shows -3 with reason "Count Fix"



#### Scenario: Count that matches posts nothing

- **Given** "CH-1001" in "WH1" has 40 on hand
- **When** the worker counts 40 and saves
- **Then** no history entry is added



### US-9.5 — See stock and its history



#### Scenario: Available is worked out

- **Given** "CH-1001" in "WH1" has 30 on hand, 24 on order, and 10 committed
- **When** an employee views it
- **Then** available is 44



#### Scenario: Below-min items are flagged

- **Given** "CH-1001" in "WH1" has available 12 and min 15
- **When** an employee lists stock with "Below min only"
- **Then** "CH-1001" is listed and marked below min



#### Scenario: History shows every change with a running total

- **Given** "CH-1001" in "WH1" started at 40, then -3, then +5
- **When** an employee views its history
- **Then** three entries are listed with totals 40, 37, and 42



### US-9.6 — Get suggested min and max



#### Scenario: Suggestion comes from recent demand

- **Given** "WH1" has min days 14 and max days 60
- **And** stores ordered 900 units of "CH-1001" in "WH1" in the last 60 days
- **When** the manager opens Min/Max Suggestions
- **Then** "CH-1001" shows 15.00 per day, suggested min 210, and suggested max 900



#### Scenario: Orders cancelled for nothing picked still count

- **Given** in the last 60 days "CH-1005" had 60 units on delivered orders, 60 units on an order cancelled as "Nothing picked", and 30 units on an order cancelled by office staff
- **When** the manager opens Min/Max Suggestions
- **Then** "CH-1005" shows 2.00 per day



#### Scenario: Manager accepts a suggestion

- **Given** "CH-1001" has min 15, max 50, and suggested min 210, max 900
- **When** the manager accepts the suggestion
- **Then** "CH-1001" has min 210 and max 900



#### Scenario: Item with no demand has no suggestion

- **Given** "AU-3003" has no store orders in the last 60 days
- **When** the manager opens Min/Max Suggestions
- **Then** "AU-3003" shows "No recent demand"



### US-9.7 — Stock and sales reports



#### Scenario: Stock status shows numbers, flag, and value

- **Given** "CH-1001" (case cost 14.40, 24 per case) in "WH1" has on hand 48, on order 0, committed 10, min 50, max 100
- **When** the manager runs the stock status report for "WH1"
- **Then** "CH-1001" shows available 38, below min, and value 28.80



#### Scenario: Sales by item adds up shipped lines

- **Given** this month "SO-000020" shipped 1 case of "CH-1001" at 19.99, and "SO-000031" shipped 2 cases at 19.99 and 6 each at 0.89
- **When** the manager runs sales by item for "WH1"
- **Then** "CH-1001" shows 78 units, 2 orders, and a total of 65.31

