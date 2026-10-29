# Feature: User Management

**Feature ID:** 10
**Branch pattern:** `feature/10-user-management`
**Status:** Draft
**Created:** 2026-10-18
**Input:** The Owner creates and maintains the accounts that managers and referees use to log in, sets passwords, and deactivates people who leave.
**Depends on:** [Feature 1 — User Auth](feature-1-user-auth.md)

---

## User Stories

### US-10.1: Create a user

**As an** Owner
**I want to** create accounts for managers and referees
**So that** they can log in

**Priority:** P1
**Independent test:** Create a manager account; that person can log in as a manager
**Acceptance scenarios:** see ### US-10.1 under Acceptance Criteria

### US-10.2: View users

**As an** Owner
**I want to** see all users with their role and status
**So that** I know who has access

**Priority:** P1
**Independent test:** Open the users list; each user shows name, email, role, and status
**Acceptance scenarios:** see ### US-10.2 under Acceptance Criteria

### US-10.3: Edit a user and set a password

**As an** Owner
**I want to** change a user's name, email, or role, and set a new password
**So that** accounts stay correct and people who forgot a password can get back in

**Priority:** P2
**Independent test:** Set a new password for a user; the old password stops working and the new one works
**Acceptance scenarios:** see ### US-10.3 under Acceptance Criteria

### US-10.4: Deactivate a user

**As an** Owner
**I want to** deactivate a user
**So that** people who left lose access but their history stays

**Priority:** P2
**Independent test:** Deactivate a user; that user cannot log in
**Acceptance scenarios:** see ### US-10.4 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: Only an Owner MUST be able to create, view, edit, and deactivate users, and set passwords.
- **FR-002**: A user MUST have a name (1–100 characters), an email, and exactly one role: `OWNER`, `MANAGER`, or `REFEREE`.
- **FR-003**: An email MUST be a valid address, at most 100 characters, and unique, ignoring letter case.
- **FR-004**: A password MUST be at least 8 characters. It MUST be stored only as a hash.
- **FR-005**: Setting a new password or deactivating a user MUST end that user's sessions.
- **FR-006**: A deactivated user MUST NOT be able to log in. The user record stays for history. The Owner MUST be able to reactivate the user.
- **FR-007**: The system MUST NOT deactivate or change the role of the last active `OWNER`.
- **FR-008**: The system MUST NOT change the role of a manager who manages a team, or of a referee who is linked to a referee record.
- **FR-009**: Users MUST be listed alphabetically by name.

---

## Assumptions

- [Feature 1](feature-1-user-auth.md) handles log in, log out, and role checks. This feature only keeps the accounts.
- The first Owner is created by a setup script. Every other account is created here.
- A manager is tied to teams in [Feature 6](feature-6-team-management.md). A referee user is linked to a referee record in [Feature 9](feature-9-referee-management.md).
- Public visitors have no account.

---

## Edge Cases

- Blank name → `ValidationError`
- Email that is not valid, or already used (any letter case) → `ValidationError` with "Email is already in use." for duplicates
- Password shorter than 8 characters → `ValidationError`
- Role other than `OWNER`, `MANAGER`, or `REFEREE` → `ValidationError`
- Deactivate or demote the last active Owner → `ValidationError`
- Change the role of a manager who manages a team → `ValidationError`
- User id does not exist → `NotFoundError`
- Anyone other than the Owner uses user maintenance → `NotAuthorizedError`

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: Only the Owner can reach user maintenance
- **SC-003**: The system always has at least one active Owner

---

## Data Ownership & Isolation

- Users are system records kept only by the Owner.
- No screen or response ever shows a password or hash.
- Managers and referees cannot see or change any user, including themselves.

---

## Key Entities

- **User**: a person who logs in. Has a name, email, password hash, role, and active status
- **Team**: a manager user can manage teams (Feature 6)
- **Referee**: a referee user can be linked to one referee record (Feature 9)

---

## Acceptance Criteria (Gherkin)

### US-10.1 — Create a user

#### Scenario: Owner creates a manager account

- **Given** the Owner is logged in
- **When** the Owner submits a unique email, a name, a valid password, and the role `MANAGER`
- **Then** the system creates the user
- **And** the stored password is a hash

#### Scenario: Owner creates a user with an email already used

- **Given** a user with email "sam@league.org" exists
- **And** the Owner is logged in
- **When** the Owner submits a new user with email "SAM@league.org"
- **Then** the system shows "Email is already in use."
- **And** no user is created

#### Scenario: Owner creates a user with a short password

- **Given** the Owner is logged in
- **When** the Owner submits a password of 5 characters
- **Then** the system returns `ValidationError`

#### Scenario: Owner creates a user with an unknown role

- **Given** the Owner is logged in
- **When** the Owner submits the role "Scorekeeper"
- **Then** the system returns `ValidationError`
- **And** no user is created

#### Scenario: Manager tries to create a user

- **Given** a manager is logged in
- **When** the manager submits a new user
- **Then** the system returns `NotAuthorizedError`

### US-10.2 — View users

#### Scenario: Owner views users alphabetically

- **Given** the users "Zoe Adams" and "Ann Baker" exist
- **And** the Owner is logged in
- **When** the Owner opens the users list
- **Then** "Ann Baker" is listed before "Zoe Adams"
- **And** each row shows email, role, and status

#### Scenario: Referee tries to list users

- **Given** a referee is logged in
- **When** the referee asks for the users list
- **Then** the system returns `NotAuthorizedError`

### US-10.3 — Edit a user and set a password

#### Scenario: Owner changes a user's name

- **Given** the Owner is logged in
- **And** the user "Sam Lee" exists
- **When** the Owner changes the name to "Samuel Lee"
- **Then** the user shows the new name

#### Scenario: Owner sets a new password

- **Given** the Owner is logged in
- **And** a user is logged in elsewhere
- **When** the Owner sets the password "newpass123" for that user
- **Then** the user can log in with the new password
- **And** the old session ends

#### Scenario: Owner sets a short password

- **Given** the Owner is logged in
- **When** the Owner sets the password "abc" for a user
- **Then** the system returns `ValidationError`
- **And** the old password still works

#### Scenario: Owner changes the role of a manager who manages a team

- **Given** the Owner is logged in
- **And** the user manages a team
- **When** the Owner changes the role to `REFEREE`
- **Then** the system returns `ValidationError`
- **And** the role stays `MANAGER`

### US-10.4 — Deactivate a user

#### Scenario: Owner deactivates a referee user

- **Given** the Owner is logged in
- **And** a referee user has past game assignments
- **When** the Owner deactivates the user
- **Then** the user cannot log in
- **And** the past assignments still show

#### Scenario: Owner reactivates a user

- **Given** the Owner is logged in
- **And** a user is deactivated
- **When** the Owner reactivates the user
- **Then** the user can log in again

#### Scenario: Owner deactivates the last active Owner

- **Given** the Owner is the only active Owner
- **When** the Owner deactivates their own account
- **Then** the system returns `ValidationError`
- **And** the account stays active

---
