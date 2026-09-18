# Feature: Children's Ministry Check-In

**Feature ID:** 5
**Branch pattern:** `feature/5-childrens-ministry-checkin`
**Status:** Draft
**Created:** 2026-09-14
**Input:** Check children in and out of a service separately from their parent, releasing them only to their linked guardian.
**Depends on:** [Feature 3 — Children & Guardian Management](feature-3-children-guardian-management.md), [Feature 4 — Service & Adult Attendance Check-In](feature-4-service-adult-attendance.md)

---

## User Stories

### US-5.1: Check in a child

**As an** usher
**I want to** check in a child for a service
**So that** children's ministry attendance is recorded separately from the parent's

**Priority:** P1
**Independent test:** Check in a child for a service and confirm a child-attendance record exists
**Acceptance scenarios:** see ### US-5.1 under Acceptance Criteria

### US-5.2: Check out a child

**As an** usher
**I want to** check out a child only to their linked guardian
**So that** children are only released to an approved adult

**Priority:** P1
**Independent test:** Attempt checkout by the linked guardian (succeeds) and by someone else (fails)
**Acceptance scenarios:** see ### US-5.2 under Acceptance Criteria

### US-5.3: View today's child check-ins

**As an** usher or admin
**I want to** see which children are currently checked in and not yet checked out
**So that** I know who's still in children's ministry

**Priority:** P1
**Independent test:** View the list of currently-checked-in children for a service
**Acceptance scenarios:** see ### US-5.3 under Acceptance Criteria

### US-5.4: View children who missed a service

**As an** usher or admin
**I want to** see which active children did NOT check in for a given service
**So that** I know who to follow up with

**Priority:** P2
**Independent test:** View the missed list for a service and see active children with no attendance record
**Acceptance scenarios:** see ### US-5.4 under Acceptance Criteria

---



## Requirements



### Functional Requirements

- **FR-001**: System MUST record one child-attendance entry per child per service, with a `checkedInAt` timestamp and the staff member who checked them in.
- **FR-002**: System MUST NOT create a second child-attendance record for the same child and the same service.
- **FR-003**: Checking out a child MUST require identifying the pickup person and confirming they are that child's linked guardian.
- **FR-004**: System MUST reject a checkout attempt when the pickup person is not the child's linked guardian; the child remains checked in.
- **FR-005**: A successful checkout MUST record `checkedOutAt` and the staff member who performed the checkout.
- **FR-006**: Only `usher` or `admin` roles MUST be able to check children in or out.
- **FR-007**: System MUST compute the "missed" list for a service as active children with no attendance record for that service.
- **FR-008**: Deactivated children MUST NOT appear in attendance, currently-checked-in, or missed results.
- **FR-009**: A child's attendance record MUST be independent of their guardian's own attendance record for the same service (no automatic linking or mismatch flagging).

---



## Key Entities

- **ChildAttendance**: a record that a specific Child (Feature 3) checked in — and later checked out — of a specific Service (Feature 4). Checkout is restricted to the child's linked guardian.

---



## Acceptance Criteria



### US-5.1 — Check in a child



#### Scenario: Usher checks in a child 

- **Given** service `Sunday 9am` exists
- **And** child `Timmy Smith` is active
- **When** I check in `Timmy Smith` for `Sunday 9am`
- **Then** the API returns `201`
- **And** a child-attendance record links `Timmy Smith` to `Sunday 9am`



#### Scenario: Child is checked in twice for the same service (failure)

- **Given** `Timmy Smith` is already checked in for `Sunday 9am`
- **When** I attempt to check him in again for `Sunday 9am`
- **Then** the API returns `409` and no duplicate record is created



### US-5.2 — Check out a child



#### Scenario: Linked guardian picks up the child

- **Given** child `Timmy Smith` is checked in for `Sunday 9am`
- **And** his linked guardian is `Jane Smith`
- **When** I check out `Timmy Smith` to `Jane Smith`
- **Then** the API returns `200`
- **And** the record shows `checkedOutAt` set



#### Scenario: Non-guardian attempts to pick up the child

- **Given** child `Timmy Smith` is checked in for `Sunday 9am`
- **And** his linked guardian is `Jane Smith`
- **When** I attempt to check him out to `John Doe`, who is not his linked guardian
- **Then** the API returns `403` with `{ "message": "Pickup person is not this child's linked guardian." }`
- **And** `Timmy Smith` remains checked in



### US-5.3 — View today's child check-ins



#### Scenario: Currently checked-in children are listed

- **Given** child `Timmy Smith` is checked in for `Sunday 9am` and not yet checked out
- **When** I view currently-checked-in children for `Sunday 9am`
- **Then** the API returns `200` including `Timmy Smith`



#### Scenario: Checked-out child no longer appears 

- **Given** child `Timmy Smith` was checked in and then checked out for `Sunday 9am`
- **When** I view currently-checked-in children for `Sunday 9am`
- **Then** `Timmy Smith` does not appear in the results



### US-5.4 — View children who missed a service



#### Scenario: Admin views the missed list for a service

- **Given** child `Timmy Smith` is active and did not check in for `Sunday 9am`
- **When** I view the missed list for `Sunday 9am`
- **Then** the API returns `200` including `Timmy Smith`



#### Scenario: Deactivated children are excluded from the missed list

- **Given** child `Old Kid` is inactive and did not check in for `Sunday 9am`
- **When** I view the missed list for `Sunday 9am`
- **Then** `Old Kid` does not appear in the results

