# Feature: Member & Household Management

**Feature ID:** 2
**Branch pattern:** `feature/2-member-household-management`
**Status:** Draft
**Created:** 2026-09-14
**Input:** Track adult members' contact info and group them into households so the church knows who its members are.
**Depends on:** [Feature 1 — User Authentication & Role Management](feature-1-user-auth-roles.md)

---

## User Stories

### US-2.1: Add a member

**As an** admin or usher
**I want to** add a new adult member with contact info
**So that** they're in the system before attendance is ever taken

**Priority:** P1
**Independent test:** Create a member with a name and phone number and confirm they appear in the directory
**Acceptance scenarios:** see ### US-2.1 under Acceptance Criteria

### US-2.2: Edit a member

**As an** admin or usher
**I want to** update a member's contact info
**So that** records stay current

**Priority:** P1
**Independent test:** Update an existing member's email and confirm the change is saved
**Acceptance scenarios:** see ### US-2.2 under Acceptance Criteria

### US-2.3: Group members into a household

**As an** admin or usher
**I want to** assign members to a household
**So that** families are grouped together

**Priority:** P2
**Independent test:** Assign two members to the same household and confirm both appear under it
**Acceptance scenarios:** see ### US-2.3 under Acceptance Criteria

### US-2.4: View member directory

**As an** admin or usher
**I want to** view and search the list of members
**So that** I can find someone quickly

**Priority:** P1
**Independent test:** Search by name and receive matching members
**Acceptance scenarios:** see ### US-2.4 under Acceptance Criteria

### US-2.5: Deactivate a member

**As an** admin
**I want to** mark a member inactive rather than delete them
**So that** historical attendance and ride records stay intact

**Priority:** P2
**Independent test:** Deactivate a member and confirm their past attendance records are unaffected
**Acceptance scenarios:** see ### US-2.5 under Acceptance Criteria

---



## Requirements



### Functional Requirements

- **FR-001**: System MUST require full name and phone number to create a member.
- **FR-002**: Email and address fields MUST be optional on a member record.
- **FR-003**: A member MAY belong to at most one household.
- **FR-004**: System MUST record a `joinDate` for each member, defaulting to the date the record is created.
- **FR-005**: System MUST NOT create a member with the same full name and phone number as an existing member without explicit confirmation.
- **FR-006**: Only `admin` or `usher` roles MUST be able to create or edit member records.
- **FR-007**: Only `admin` MUST be able to deactivate a member.
- **FR-008**: Deactivating a member MUST NOT delete their historical attendance or route-assignment records.
- **FR-009**: Deactivated members MUST NOT appear in the default member directory or attendance check-in lists.

---



## Key Entities

- **Member**: an adult attendee; belongs to zero or one Household; managed by staff. Used by Feature 3 (as a guardian for children), Feature 4 (adult attendance check-in), and Feature 6 (route assignment).
- **Household**: a family grouping that a member — and later, that member's children — belong to.

---



## Acceptance Criteria



### US-2.1 — Add a member



#### Scenario: Usher adds a member with required fields

- **Given** I am signed in as a usher
- **When** I create a member with full name `Jane Smith` and phone `555-0100`
- **Then** the API returns `201`
- **And** `Jane Smith` appears in the member directory



#### Scenario: Member is created without a required field

- **Given** I am signed in as a usher
- **When** I attempt to create a member without a phone number
- **Then** the API returns a validation error and no member is created



### US-2.2 — Edit a member



#### Scenario: Usher updates a member's email

- **Given** a member `Jane Smith` exists
- **When** I update her email to `jane@example.com`
- **Then** the API returns `200`
- **And** her record reflects the new email



#### Scenario: Edit is attempted on a non-existent member

- **Given** no member with id `9999` exists
- **When** I attempt to edit member `9999`
- **Then** the API returns `404`



### US-2.3 — Group members into a household



#### Scenario: Two members are assigned to the same household

- **Given** members `Jane Smith` and `John Smith` exist
- **When** I assign both to household `Smith Family`
- **Then** the API returns `200`
- **And** viewing the `Smith Family` household lists both members



#### Scenario: Member is assigned to a non-existent household

- **Given** member `Jane Smith` exists
- **When** I attempt to assign her to a household id that does not exist
- **Then** the API returns `404` and her household is unchanged



### US-2.4 — View member directory



#### Scenario: Search returns matching members

- **Given** members `Jane Smith` and `John Doe` exist
- **When** I search the directory for `Smith`
- **Then** the API returns `200` with only `Jane Smith` in the results



#### Scenario: Search with no matches 

- **Given** the member directory has existing members
- **When** I search for a name that matches no one
- **Then** the API returns `200` with an empty list, not an error



### US-2.5 — Deactivate a member



#### Scenario: Admin deactivates a member 

- **Given** I am signed in as an admin
- **And** member `Jane Smith` has past attendance records
- **When** I deactivate `Jane Smith`
- **Then** the API returns `200`
- **And** `Jane Smith` no longer appears in the default member directory
- **And** her past attendance records remain unchanged



#### Scenario: Non-admin attempts to deactivate a member

- **Given** I am signed in as a usher
- **When** I attempt to deactivate a member
- **Then** the API returns `403` and the member remains active

