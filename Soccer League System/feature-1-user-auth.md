# Feature: User Auth

**Feature ID:** 1  
**Branch pattern:** `feature/1-user-auth`  
**Status:** Draft  
**Created:** 2026-10-18  
**Input:** Control who can use the Soccer League system and what each role may do. Owners, managers, and referees log in and out. The public can look at league information without an account.  
**Depends on:** —

---

## User Stories

### US-1.1: Log in

**As an** owner, manager, or referee
**I want to** log in with my email and password
**So that** I can use the parts of the system my role is allowed to use

**Priority:** P1
**Independent test:** Log in with a good email and password; the session has that user's role
**Acceptance scenarios:** see ### US-1.1 under Acceptance Criteria

### US-1.2: Log out

**As a** user who is logged in
**I want to** log out
**So that** nobody else can keep using the system as me

**Priority:** P1
**Independent test:** Log in, log out, then try a protected action and get `NotAuthenticatedError`
**Acceptance scenarios:** see ### US-1.2 under Acceptance Criteria

### US-1.3: Role-based access

**As an** Owner
**I want** each role to reach only its own functions
**So that** people can only do the work they are responsible for

**Priority:** P1
**Independent test:** A `REFEREE` trying an Owner-only function gets `NotAuthorizedError`; an `OWNER` is allowed
**Acceptance scenarios:** see ### US-1.3 under Acceptance Criteria

### US-1.4: Managers stay with their own teams

**As an** Owner
**I want** managers limited to the teams they manage
**So that** one manager cannot change another manager's players or scores

**Priority:** P1
**Independent test:** A manager of team A adds a player to team B and gets `NotAuthorizedError`
**Acceptance scenarios:** see ### US-1.4 under Acceptance Criteria

### US-1.5: Browse without an account

**As a** public visitor
**I want to** look at published league information without logging in
**So that** family and friends can follow the games

**Priority:** P1
**Independent test:** With no login, open the leagues and games lists; they show. Try to change something; it is refused
**Acceptance scenarios:** see ### US-1.5 under Acceptance Criteria

---



## Requirements



### Functional Requirements

Who can do what:


| Role           | What they do                                                                                               |
| -------------- | ---------------------------------------------------------------------------------------------------------- |
| `OWNER`        | Runs the league. Maintains sports, leagues, seasons, games, teams, players, locations, referees, and users |
| `MANAGER`      | Looks after their own teams and players. Enters scores and statistics for their own team's games           |
| `REFEREE`      | Sees their own record and the games they are assigned to                                                   |
| Public visitor | Has no account. Can only view published information                                                        |


- **FR-001**: The system MUST let owners, managers, and referees log in with an email and a password.
- **FR-002**: The system MUST only let them in if the email and password match what is stored.
- **FR-003**: If log in fails, the system MUST show one generic error. It must not say whether the email or the password was wrong.
- **FR-004**: When someone logs out, the system MUST end the session. The system MUST reject log out if nobody is logged in.
- **FR-005**: Each user MUST have exactly one role: `OWNER`, `MANAGER`, or `REFEREE`.
- **FR-006**: The system MUST check the role on every protected action. If the role is not allowed, the system MUST report `NotAuthorizedError`.
- **FR-007**: A manager MUST be able to change only the teams, players, scores, and statistics of teams they manage.
- **FR-008**: A referee MUST be able to see only their own record and their own assigned games.
- **FR-009**: Visitors who are not logged in MUST be able to view published information only: sports, leagues, seasons, teams, rosters, games, scores, and statistics. They MUST NOT be able to add, edit, or delete anything.
- **FR-010**: There MUST be a first `OWNER` account so the league can log in before other users exist.
- **FR-011**: Passwords MUST be stored only as hashes. No screen or response may show a password or hash.
- **FR-012**: A deactivated user MUST NOT be able to log in. If a user is deactivated or gets a new password while logged in, the next protected action MUST return `NotAuthenticatedError`.

---



## Assumptions

- The first Owner is created by a setup script. There is no sign-up screen.
- Users never register themselves. Accounts are created in [Feature 10](feature-10-user-management.md).
- A session lasts 24 hours. The notes do not say, so this is a proposal.
- Password reset by email is not part of this system. The Owner sets a new password in Feature 10.
- Every other feature uses the roles and checks defined here.

---



## Edge Cases

- Empty email or password → `ValidationError`
- Unknown email, wrong password, or deactivated user → `NotAuthenticatedError` with the same generic message
- Log out with no session → `NotAuthenticatedError`
- Session ended by log out, password change, or deactivation → the next protected action returns `NotAuthenticatedError`
- Role not allowed for the action → `NotAuthorizedError`
- Manager works on a team they do not manage → `NotAuthorizedError`
- Referee asks for another referee's record or games → `NotAuthorizedError`
- Visitor tries to add, edit, or delete anything → `NotAuthenticatedError`

---



## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: Bad or deactivated credentials never create a session
- **SC-003**: After log out, every protected action returns `NotAuthenticatedError`
- **SC-004**: A visitor can read published data but can never change anything

---



## Data Ownership & Isolation

- A session belongs to one user. It never acts as someone else.
- Role is read from the user record at log in and checked again on each protected action.
- Managers act only on teams they manage. Referees see only their own data. Only the Owner sees or changes users.

---



## Key Entities

- **User**: a person who logs in. Has an email, password, role, and active status (kept in Feature 10)
- **Session**: the logged-in user's identity for one client (userId, role, login time, expiry)
- **Role**: `OWNER`, `MANAGER`, or `REFEREE`. Public visitors have no account

---



## Acceptance Criteria (Gherkin)



### US-1.1 — Log in



#### Scenario: Owner logs in with valid credentials

- **Given** an active user exists with email "[owner@league.org](mailto:owner@league.org)", a password, and the role `OWNER`
- **When** the user submits that email and password
- **Then** the system creates a session
- **And** the session shows the role `OWNER`



#### Scenario: Manager logs in with valid credentials

- **Given** an active user exists with the role `MANAGER`
- **When** the user submits the correct email and password
- **Then** the system creates a session with the role `MANAGER`



#### Scenario: Referee logs in with valid credentials

- **Given** an active user exists with the role `REFEREE`
- **When** the user submits the correct email and password
- **Then** the system creates a session with the role `REFEREE`



#### Scenario: User logs in with a wrong password

- **Given** an active manager exists
- **When** the manager submits an incorrect password
- **Then** the system shows "Invalid email or password."
- **And** no session is created



#### Scenario: Unknown user cannot log in

- **Given** no user exists with email "[ghost@league.org](mailto:ghost@league.org)"
- **When** someone submits that email and any password
- **Then** the system shows "Invalid email or password."
- **And** no session is created



#### Scenario: Deactivated user cannot log in

- **Given** a manager has been deactivated
- **When** the manager submits correct credentials
- **Then** the system shows "Invalid email or password."
- **And** no session is created



#### Scenario: User logs in with an empty password

- **Given** the log in screen is open
- **When** the user submits an email and no password
- **Then** the system returns `ValidationError`



#### Scenario: Passwords are never shown

- **Given** an active manager exists
- **When** the manager logs in
- **Then** no response contains the password or its hash



### US-1.2 — Log out



#### Scenario: Logged-in user logs out

- **Given** a user is logged in
- **When** the user selects Log out
- **Then** the session ends
- **And** any protected action returns `NotAuthenticatedError`



#### Scenario: Log out with nobody logged in

- **Given** nobody is logged in
- **When** the client tries to log out
- **Then** the system returns `NotAuthenticatedError`



#### Scenario: User deactivated while logged in

- **Given** a manager is logged in
- **And** the Owner deactivates the manager
- **When** the manager does any protected action
- **Then** the system returns `NotAuthenticatedError`



### US-1.3 — Role-based access



#### Scenario: Owner uses an Owner-only function

- **Given** a user is logged in with the role `OWNER`
- **When** the user opens User maintenance
- **Then** the system allows access



#### Scenario: Referee is denied an Owner-only function

- **Given** a user is logged in with the role `REFEREE`
- **When** the user creates a league
- **Then** the system returns `NotAuthorizedError`



#### Scenario: Menu shows only permitted functions for a manager

- **Given** a user is logged in with the role `MANAGER`
- **When** the home screen is shown
- **Then** the menu includes My Teams, My Players, and My Games
- **And** the menu does not include Users or Referees



#### Scenario: Menu shows only permitted functions for a referee

- **Given** a user is logged in with the role `REFEREE`
- **When** the home screen is shown
- **Then** the menu includes My Games
- **And** the menu does not include any maintenance function



### US-1.4 — Managers stay with their own teams



#### Scenario: Manager is limited to their own team

- **Given** a manager of "Eastside FC" is logged in
- **When** the manager adds a player to "Westside FC"
- **Then** the system returns `NotAuthorizedError`



#### Scenario: Owner can work on every team

- **Given** an Owner is logged in
- **When** the Owner adds a player to any team
- **Then** the system allows it



#### Scenario: Referee asks for another referee's games

- **Given** a referee is logged in
- **When** the referee asks for another referee's games
- **Then** the system returns `NotAuthorizedError`



### US-1.5 — Browse without an account



#### Scenario: Visitor views published information

- **Given** nobody is logged in
- **When** the visitor opens leagues, teams, games, or statistics
- **Then** the system shows them without asking for a password



#### Scenario: Visitor tries to change data

- **Given** nobody is logged in
- **When** the visitor tries to add or edit any record
- **Then** the system returns `NotAuthenticatedError`

---

