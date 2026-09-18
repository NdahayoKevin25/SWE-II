# Feature: Service & Adult Attendance Check-In

**Feature ID:** 4
**Branch pattern:** `feature/4-service-adult-attendance`
**Status:** Draft
**Created:** 2026-09-14
**Input:** Track which adult members attend each service via per-person check-in, so staff know who attended and who missed.
**Depends on:** [Feature 1 — User Authentication & Role Management](feature-1-user-auth-roles.md), [Feature 2 — Member & Household Management](feature-2-member-household-management.md)

---

## User Stories

### US-4.1: Define a service

**As an** admin
**I want to** create a service with a date and a label (e.g. "Sunday 9am")
**So that** attendance can be tracked against a specific gathering

**Priority:** P1
**Independent test:** Create a service and confirm it's available to check members into
**Acceptance scenarios:** see ### US-4.1 under Acceptance Criteria

### US-4.2: Check in a member

**As an** usher or admin
**I want to** check in a member for a specific service
**So that** their attendance is recorded as it happens

**Priority:** P1
**Independent test:** Check in a member for a service and confirm the attendance record exists
**Acceptance scenarios:** see ### US-4.2 under Acceptance Criteria

### US-4.3: Self check-in

**As a** member
**I want to** check myself in at a kiosk or app for today's service
**So that** I don't need staff to do it for me

**Priority:** P2
**Independent test:** Sign in as a member and check in for today's service without staff assistance
**Acceptance scenarios:** see ### US-4.3 under Acceptance Criteria

### US-4.4: View today's attendance

**As an** usher or admin
**I want to** see who has checked in for a given service
**So that** I know who's here

**Priority:** P1
**Independent test:** View the attendance list for a service and see all checked-in members
**Acceptance scenarios:** see ### US-4.4 under Acceptance Criteria

### US-4.5: View who missed a service

**As an** usher or admin
**I want to** see which active members did NOT check in for a given service
**So that** I know who to follow up with

**Priority:** P1
**Independent test:** View the missed list for a service and see active members with no attendance record
**Acceptance scenarios:** see ### US-4.5 under Acceptance Criteria

---



## Requirements



### Functional Requirements

- **FR-001**: Only `admin` MUST be able to create a Service, which requires a date and a label.
- **FR-002**: System MUST record one attendance entry per member per service, with a `checkedInAt` timestamp.
- **FR-003**: System MUST NOT create a second attendance record for the same member and the same service (duplicate check-ins are no-ops, not errors).
- **FR-004**: `usher` and `admin` roles MUST be able to check in any active member.
- **FR-005**: A member using self-check-in MUST only be able to check in themselves, not any other member.
- **FR-006**: System MUST compute the "missed" list for a service as active members with no attendance record for that service.
- **FR-007**: Deactivated members MUST NOT appear in attendance or missed-attendance results.
- **FR-008**: System MUST record which staff member performed a check-in; a `null` value indicates self-check-in.

---



## Key Entities

- **Service**: a specific gathering (date + label) that attendance is tracked against. Shared with Feature 5 (children's ministry attendance uses the same Service).
- **Attendance**: a record that a specific Member (Feature 2) checked in to a specific Service.

---



## Acceptance Criteria



### US-4.1 — Define a service



#### Scenario: Admin creates a service

- **Given** I am signed in as an admin
- **When** I create a service with label `Sunday 9am` and date `2026-09-20`
- **Then** the API returns `201`
- **And** `Sunday 9am` is available for check-ins



#### Scenario: Non-admin attempts to create a service

- **Given** I am signed in as a usher
- **When** I attempt to create a service
- **Then** the API returns `403` and no service is created



### US-4.2 — Check in a member



#### Scenario: Usher checks in a member

- **Given** service `Sunday 9am` exists
- **And** member `Jane Smith` is active
- **When** I check in `Jane Smith` for `Sunday 9am`
- **Then** the API returns `201`
- **And** an attendance record links `Jane Smith` to `Sunday 9am`



#### Scenario: Member is checked in twice for the same service 

- **Given** `Jane Smith` is already checked in for `Sunday 9am`
- **When** I attempt to check her in again for `Sunday 9am`
- **Then** the API returns `200` with no duplicate attendance record created



### US-4.3 — Self check-in



#### Scenario: Member checks themselves in

- **Given** I am signed in as member `Jane Smith`
- **And** service `Sunday 9am` exists for today
- **When** I check myself in for `Sunday 9am`
- **Then** the API returns `201`
- **And** the attendance record shows `checkedInBy` as `null`



#### Scenario: Member attempts to check in another member

- **Given** I am signed in as member `Jane Smith`
- **When** I attempt to check in member `John Doe`
- **Then** the API returns `403` and no attendance record is created



### US-4.4 — View today's attendance



#### Scenario: Admin views attendance for a service

- **Given** three members are checked in for `Sunday 9am`
- **When** I view the attendance list for `Sunday 9am`
- **Then** the API returns `200` listing all three members



#### Scenario: Service with zero check-ins

- **Given** service `Wednesday Night` has no check-ins yet
- **When** I view the attendance list for `Wednesday Night`
- **Then** the API returns `200` with an empty list



### US-4.5 — View who missed a service



#### Scenario: Admin views the missed list for a service

- **Given** member `Jane Smith` is active and checked in for `Sunday 9am`
- **And** member `John Doe` is active and did not check in for `Sunday 9am`
- **When** I view the missed list for `Sunday 9am`
- **Then** the API returns `200` listing `John Doe` and not `Jane Smith`



#### Scenario: Deactivated members are excluded from the missed list 

- **Given** member `Old Member` is inactive and did not check in for `Sunday 9am`
- **When** I view the missed list for `Sunday 9am`
- **Then** `Old Member` does not appear in the results

