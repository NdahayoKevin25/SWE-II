# Feature: Children & Guardian Management

**Feature ID:** 3
**Branch pattern:** `feature/3-children-guardian-management`
**Status:** Draft
**Created:** 2026-09-14
**Input:** Track children and link each one to a single primary parent/guardian, so the church knows whose child they are.
**Depends on:** [Feature 2 — Member & Household Management](feature-2-member-household-management.md)

---

## User Stories

### US-3.1: Add a child

**As an** admin or usher
**I want to** add a child linked to a primary guardian
**So that** the system knows whose child they are before attendance is ever taken

**Priority:** P1
**Independent test:** Create a child record linked to an existing member and confirm the link is saved
**Acceptance scenarios:** see ### US-3.1 under Acceptance Criteria

### US-3.2: Edit a child's info

**As an** admin or usher
**I want to** update a child's info, including reassigning their guardian
**So that** records stay accurate as family situations change

**Priority:** P1
**Independent test:** Reassign an existing child to a different guardian and confirm the change is saved
**Acceptance scenarios:** see ### US-3.2 under Acceptance Criteria

### US-3.3: View children by guardian

**As an** admin or usher
**I want to** view all children linked to a guardian
**So that** I can see the whole family at a glance

**Priority:** P2
**Independent test:** View a guardian's profile and see all children linked to them
**Acceptance scenarios:** see ### US-3.3 under Acceptance Criteria

### US-3.4: Deactivate a child record

**As an** admin
**I want to** mark a child inactive rather than delete them
**So that** historical attendance stays intact

**Priority:** P2
**Independent test:** Deactivate a child and confirm their past attendance records are unaffected
**Acceptance scenarios:** see ### US-3.4 under Acceptance Criteria

---



## Requirements



### Functional Requirements

- **FR-001**: System MUST require a child's full name and a `guardianId` referencing an existing Member to create a child record.
- **FR-002**: A child MUST have exactly one primary guardian at any given time.
- **FR-003**: System MUST allow reassigning a child's guardian to a different existing Member.
- **FR-004**: Date of birth MUST be an optional field on a child record.
- **FR-005**: Only `admin` or `usher` roles MUST be able to create or edit child records.
- **FR-006**: Only `admin` MUST be able to deactivate a child record.
- **FR-007**: Deactivating a child MUST NOT delete their historical attendance records.
- **FR-008**: System MUST support listing all children linked to a given guardian.
- **FR-009**: Deactivated children MUST NOT appear in the default children directory or check-in lists.

---



## Key Entities

- **Child**: a minor attendee; belongs to exactly one guardian (a Member from Feature 2). Used by Feature 5 for children's-ministry check-in, where pickup is restricted to this linked guardian.

---



## Acceptance Criteria



### US-3.1 — Add a child



#### Scenario: Usher adds a child linked to an existing guardian 

- **Given** member `Jane Smith` exists
- **When** I create a child named `Timmy Smith` with guardian `Jane Smith`
- **Then** the API returns `201`
- **And** `Timmy Smith` appears linked to `Jane Smith`



#### Scenario: Child is created without a guardian 

- **Given** I am signed in as a usher
- **When** I attempt to create a child without a `guardianId`
- **Then** the API returns a validation error and no child is created



### US-3.2 — Edit a child's info



#### Scenario: Usher reassigns a child's guardian

- **Given** child `Timmy Smith` is linked to guardian `Jane Smith`
- **And** member `John Smith` exists
- **When** I reassign `Timmy Smith`'s guardian to `John Smith`
- **Then** the API returns `200`
- **And** `Timmy Smith` is now linked to `John Smith` and no longer to `Jane Smith`



#### Scenario: Child is reassigned to a non-existent guardian 

- **Given** child `Timmy Smith` exists
- **When** I attempt to reassign his guardian to a member id that does not exist
- **Then** the API returns `404` and his guardian is unchanged



### US-3.3 — View children by guardian



#### Scenario: Guardian's children are listed

- **Given** guardian `Jane Smith` has two linked children
- **When** I view children for `Jane Smith`
- **Then** the API returns `200` with both children listed



#### Scenario: Guardian with no children

- **Given** guardian `John Doe` has no linked children
- **When** I view children for `John Doe`
- **Then** the API returns `200` with an empty list



### US-3.4 — Deactivate a child record



#### Scenario: Admin deactivates a child 

- **Given** I am signed in as an admin
- **And** child `Timmy Smith` has past attendance records
- **When** I deactivate `Timmy Smith`
- **Then** the API returns `200`
- **And** `Timmy Smith` no longer appears in the default children directory
- **And** his past attendance records remain unchanged



#### Scenario: Non-admin attempts to deactivate a child

- **Given** I am signed in as a usher
- **When** I attempt to deactivate a child
- **Then** the API returns `403` and the child remains active

