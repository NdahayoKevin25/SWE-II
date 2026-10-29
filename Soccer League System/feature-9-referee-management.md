# Feature: Referee Management

**Feature ID:** 9
**Branch pattern:** `feature/9-referee-management`
**Status:** Draft
**Created:** 2026-10-18
**Input:** The Owner keeps the list of referees and schedules them to games. Referees can log in and see the games they are assigned to.
**Depends on:** [Feature 5 — Game Management](feature-5-game-management.md), [Feature 1 — User Auth](feature-1-user-auth.md), [Feature 10 — User Management](feature-10-user-management.md)

---

## User Stories

### US-9.1: Add a referee

**As an** Owner
**I want to** add a referee to the list
**So that** the referee can be scheduled to games

**Priority:** P1
**Independent test:** Add "Pat Nguyen"; the referee list shows Pat
**Acceptance scenarios:** see ### US-9.1 under Acceptance Criteria

### US-9.2: View and edit referees

**As an** Owner
**I want to** see all referees and fix their details
**So that** I can pick who to schedule

**Priority:** P1
**Independent test:** Open the referee list; change a phone number; the list shows it
**Acceptance scenarios:** see ### US-9.2 under Acceptance Criteria

### US-9.3: Link a referee to a login

**As an** Owner
**I want to** link a referee to a user account with the role `REFEREE`
**So that** the referee can log in and see their games

**Priority:** P2
**Independent test:** Link a referee to a `REFEREE` user; that user sees only their own record
**Acceptance scenarios:** see ### US-9.3 under Acceptance Criteria

### US-9.4: Retire a referee

**As an** Owner
**I want to** mark a referee inactive
**So that** they are not offered for new games but their history stays

**Priority:** P2
**Independent test:** Deactivate a referee with past games; the games still show the referee
**Acceptance scenarios:** see ### US-9.4 under Acceptance Criteria

### US-9.5: Assign a referee to a game

**As an** Owner
**I want to** assign referees to a game with a role
**So that** every game has its officials

**Priority:** P1
**Independent test:** Assign "Pat Nguyen" as Center to a Scheduled game; the game shows Pat
**Acceptance scenarios:** see ### US-9.5 under Acceptance Criteria

### US-9.6: Change or remove an assignment

**As an** Owner
**I want to** replace or remove a referee on a game
**So that** I can cover for a referee who cannot come

**Priority:** P2
**Independent test:** Replace a referee on a Scheduled game; the new referee shows
**Acceptance scenarios:** see ### US-9.6 under Acceptance Criteria

### US-9.7: See my games

**As a** referee
**I want to** see the games I am assigned to
**So that** I know when and where to be

**Priority:** P1
**Independent test:** Log in as a referee with 2 assigned games; both show with date, time, and location
**Acceptance scenarios:** see ### US-9.7 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: The system MUST let the Owner add, view, edit, and deactivate referees. Only the Owner can see the full referee list and contact details.
- **FR-002**: A referee MUST have a first name and last name (1–50 characters each). A referee MAY have an email and a phone number (up to 20 characters).
- **FR-003**: A referee MAY be linked to one user account with the role `REFEREE`. A user MUST be linked to at most one referee.
- **FR-004**: A referee MUST have a status of `ACTIVE` or `INACTIVE`. An `INACTIVE` referee MUST NOT be offered for new assignments and keeps all past assignments.
- **FR-005**: The system MUST NOT delete a referee who has assignments.
- **FR-006**: Only the Owner MUST be able to assign referees. An assignment links one `ACTIVE` referee to one game that is `SCHEDULED` or `POSTPONED`. A game MAY have more than one referee.
- **FR-007**: Each assignment MUST have a role of `CENTER` or `ASSISTANT`. A game MUST have at most one `CENTER` and two `ASSISTANT` referees.
- **FR-008**: A referee MUST NOT be assigned to the same game twice, or to two games at the same date and time.
- **FR-009**: The Owner MUST be able to replace or remove an assignment only while the game is not `COMPLETED`.
- **FR-010**: A `CANCELLED` game MUST keep its assignments so the Owner can sort them out.
- **FR-011**: A referee who is logged in MUST see only their own record and their own assigned games.
- **FR-012**: Game views for visitors MUST show a referee's name only, never contact details.

---

## Assumptions

- The crew size (one center, up to two assistants) is a proposal. The notes only say referees are assigned to games.
- A referee can exist before they get a login.
- Paying referees is not part of this system.
- Games are built in [Feature 5](feature-5-game-management.md). Login accounts are in [Feature 10](feature-10-user-management.md).

---

## Edge Cases

- Blank first or last name → `ValidationError`
- Email that is not a valid address → `ValidationError`
- Linking a user who is not a `REFEREE`, or already linked → `ValidationError`
- Second `CENTER`, third `ASSISTANT`, same referee twice on a game, or `INACTIVE` referee → `ValidationError`
- Referee already has a game at that date and time → `ValidationError`
- Replace or remove an assignment on a `COMPLETED` game → `ValidationError`
- Delete a referee who has assignments → `ValidationError`
- Referee id or game id does not exist → `NotFoundError`
- Anyone other than the Owner changes referees or assignments → `NotAuthorizedError`
- Referee asks for another referee's record or games → `NotAuthorizedError`

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: No referee is ever booked for two games at the same time
- **SC-003**: A referee only sees their own record and games

---

## Data Ownership & Isolation

- Referees and assignments are kept by the Owner.
- A referee who is logged in reads only the record and assignments linked to their own account.
- Managers and visitors never see referee contact details.

---

## Key Entities

- **Referee**: a person who officiates games. Has a first name, last name, optional email and phone, and status
- **RefereeAssignment**: a referee scheduled to a game in a role (`CENTER` or `ASSISTANT`)
- **User**: the optional login of a referee (Feature 10)
- **Game**: the match a referee is assigned to (Feature 5)

---

## Acceptance Criteria (Gherkin)

### US-9.1 — Add a referee

#### Scenario: Owner adds a referee

- **Given** the Owner is logged in
- **When** the Owner submits the first name "Pat" and the last name "Nguyen"
- **Then** the system creates the referee as `ACTIVE`

#### Scenario: Owner adds a referee with a blank name

- **Given** the Owner is logged in
- **When** the Owner submits a blank last name
- **Then** the system returns `ValidationError`
- **And** no referee is created

#### Scenario: Manager tries to add a referee

- **Given** a manager is logged in
- **When** the manager submits a new referee
- **Then** the system returns `NotAuthorizedError`

### US-9.2 — View and edit referees

#### Scenario: Owner views the referee list

- **Given** the Owner is logged in
- **And** referees exist
- **When** the Owner opens the referee list
- **Then** each referee shows name, email, phone, and status

#### Scenario: Visitor cannot open the referee list

- **Given** nobody is logged in
- **When** a visitor asks for the referee list
- **Then** the system returns `NotAuthorizedError`

#### Scenario: Owner changes a phone number

- **Given** the Owner is logged in
- **And** the referee "Pat Nguyen" exists
- **When** the Owner changes the phone number
- **Then** the referee shows the new number

### US-9.3 — Link a referee to a login

#### Scenario: Owner links a referee to a referee user

- **Given** the Owner is logged in
- **And** a user with the role `REFEREE` is not linked yet
- **When** the Owner links the user to "Pat Nguyen"
- **Then** that user sees only the record of "Pat Nguyen"

#### Scenario: Owner links a user who is not a referee

- **Given** the Owner is logged in
- **And** a user with the role `MANAGER` exists
- **When** the Owner links that user to a referee
- **Then** the system returns `ValidationError`

#### Scenario: Owner links a user who is already linked

- **Given** a user is linked to "Pat Nguyen"
- **And** the Owner is logged in
- **When** the Owner links the same user to another referee
- **Then** the system returns `ValidationError`

### US-9.4 — Retire a referee

#### Scenario: Owner deactivates a referee with past assignments

- **Given** the Owner is logged in
- **And** the referee has past assignments
- **When** the Owner sets the referee to `INACTIVE`
- **Then** the past assignments still show
- **And** the referee is not offered for new assignments

#### Scenario: Owner deletes a referee with assignments

- **Given** the Owner is logged in
- **And** the referee has an assignment
- **When** the Owner deletes the referee
- **Then** the system returns `ValidationError`
- **And** the referee stays

### US-9.5 — Assign a referee to a game

#### Scenario: Owner assigns a center referee

- **Given** the Owner is logged in
- **And** "Pat Nguyen" is an active referee
- **And** a `SCHEDULED` game has no referees
- **When** the Owner assigns "Pat Nguyen" as `CENTER`
- **Then** the game shows "Pat Nguyen" as `CENTER`

#### Scenario: Owner assigns an assistant next to a center

- **Given** the Owner is logged in
- **And** a game has a `CENTER` referee
- **When** the Owner assigns another active referee as `ASSISTANT`
- **Then** the game has both referees

#### Scenario: Owner assigns a second center

- **Given** the Owner is logged in
- **And** a game already has a `CENTER` referee
- **When** the Owner assigns another referee as `CENTER`
- **Then** the system returns `ValidationError`

#### Scenario: Owner assigns an inactive referee

- **Given** the Owner is logged in
- **And** "Lee Park" is `INACTIVE`
- **When** the Owner assigns "Lee Park" to a game
- **Then** the system returns `ValidationError`

#### Scenario: Owner double-books a referee

- **Given** "Pat Nguyen" is assigned to a game on 2026-09-12 at 10:00
- **And** the Owner is logged in
- **When** the Owner assigns "Pat Nguyen" to another game at that date and time
- **Then** the system returns `ValidationError`

#### Scenario: Manager tries to assign a referee

- **Given** a manager is logged in
- **When** the manager assigns a referee to a game
- **Then** the system returns `NotAuthorizedError`

### US-9.6 — Change or remove an assignment

#### Scenario: Owner replaces a referee before the game

- **Given** the Owner is logged in
- **And** "Pat Nguyen" is assigned to a `SCHEDULED` game
- **When** the Owner replaces "Pat Nguyen" with "Sam Ortiz"
- **Then** "Sam Ortiz" is assigned
- **And** "Pat Nguyen" is not

#### Scenario: Owner removes an assignment from a completed game

- **Given** the Owner is logged in
- **And** the game is `COMPLETED`
- **When** the Owner removes a referee
- **Then** the system returns `ValidationError`

#### Scenario: A cancelled game keeps its assignments

- **Given** the Owner is logged in
- **And** a game with referees is `CANCELLED`
- **When** the Owner opens the game's referees
- **Then** the assignments are still listed

### US-9.7 — See my games

#### Scenario: Referee sees their assigned games

- **Given** a referee is logged in
- **And** the referee is assigned to two games
- **When** the referee opens their games
- **Then** both games show with date, time, and location

#### Scenario: Referee asks for another referee's games

- **Given** a referee is logged in
- **When** the referee asks for another referee's games
- **Then** the system returns `NotAuthorizedError`

---
