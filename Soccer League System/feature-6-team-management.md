# Feature: Team Management

**Feature ID:** 6
**Branch pattern:** `feature/6-team-management`
**Status:** Draft
**Created:** 2026-10-18
**Input:** The Owner creates the teams of a league, gives each team a manager, and sets a home location. Managers look after their own teams.
**Depends on:** [Feature 3 — League Management](feature-3-league-management.md), [Feature 8 — Location Management](feature-8-location-management.md), [Feature 1 — User Auth](feature-1-user-auth.md), [Feature 10 — User Management](feature-10-user-management.md)

---

## User Stories

### US-6.1: Create a team

**As an** Owner
**I want to** create a team in a league and choose its manager
**So that** the team can play games

**Priority:** P1
**Independent test:** Create "Eastside FC" with a manager; it shows up in the league's teams
**Acceptance scenarios:** see ### US-6.1 under Acceptance Criteria

### US-6.2: Set a team's home location

**As an** Owner
**I want to** give a team a home location
**So that** games it hosts are placed at the right field

**Priority:** P2
**Independent test:** Set "Riverside Field 1" on a team; the team shows it
**Acceptance scenarios:** see ### US-6.2 under Acceptance Criteria

### US-6.3: View teams

**As a** manager, referee, or public visitor
**I want to** see the teams of a league
**So that** I know who plays

**Priority:** P1
**Independent test:** Open a league's teams without logging in; the teams show
**Acceptance scenarios:** see ### US-6.3 under Acceptance Criteria

### US-6.4: Edit my team

**As a** manager
**I want to** change my team's name and home location
**So that** my team's details stay right

**Priority:** P2
**Independent test:** As a manager, rename my team; the list shows the new name. Try another team; it is refused
**Acceptance scenarios:** see ### US-6.4 under Acceptance Criteria

### US-6.5: Retire or delete a team

**As an** Owner
**I want to** mark a team inactive, or delete a team that never played
**So that** old games keep their teams

**Priority:** P2
**Independent test:** Delete a team with no games; it is gone. Try one with games; it stays
**Acceptance scenarios:** see ### US-6.5 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: The system MUST let the Owner create, edit, and delete teams.
- **FR-002**: Each team MUST belong to exactly one league and have exactly one manager. The manager MUST be an active user with the role `MANAGER`.
- **FR-003**: One manager MAY manage more than one team.
- **FR-004**: A team MUST have a name (1–100 characters, unique within its league). A team MAY have a home location from the existing locations.
- **FR-005**: A manager MUST be able to change only the name and home location of a team they manage. Only the Owner can change a team's league or manager.
- **FR-006**: The system MUST NOT change a team's league if the team has games.
- **FR-007**: A team MUST have a status of `ACTIVE` or `INACTIVE`. An `INACTIVE` team MUST be left out when all games of a season are created. It keeps its history.
- **FR-008**: The system MUST NOT delete a team that has games. A team without games MAY be deleted, and its players are deleted with it.
- **FR-009**: Everyone, including visitors who are not logged in, MUST be able to view teams. Managers, referees, and visitors cannot create or delete teams.

---

## Assumptions

- Players are built in [Feature 7](feature-7-player-management.md), and games in [Feature 5](feature-5-game-management.md).
- Manager accounts are created in [Feature 10](feature-10-user-management.md).
- A team does not change leagues once it has played. That keeps the history simple.

---

## Edge Cases

- Blank name → `ValidationError`
- Name already used in the league → `ValidationError`
- Manager user is missing, inactive, or not a `MANAGER` → `ValidationError`
- Home location does not exist → `NotFoundError`
- League or team id does not exist → `NotFoundError`
- Change the league of a team that has games → `ValidationError`
- Delete a team that has games → `ValidationError` with "Team has games and cannot be deleted."
- Manager edits a team they do not manage → `NotAuthorizedError`
- Manager tries to change the team's league or manager → `NotAuthorizedError`

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A manager can only edit teams they manage
- **SC-003**: A team with games can never be deleted

---

## Data Ownership & Isolation

- A team belongs to one league and one manager.
- Managers change only their own teams. The Owner changes any team.
- Referees and visitors can only read.

---

## Key Entities

- **Team**: a group of players in one league. Has a name, status, a manager, and an optional home location
- **User (manager)**: the account that looks after the team (Feature 10)
- **League**: the parent of the team (Feature 3)
- **Location**: the home field (Feature 8)
- **Player** and **Game**: belong to or involve the team (Features 7 and 5)

---

## Acceptance Criteria (Gherkin)

### US-6.1 — Create a team

#### Scenario: Owner creates a team with a manager

- **Given** the Owner is logged in
- **And** the league "Fall Adult League" exists
- **And** an active manager "Dana Cole" exists
- **When** the Owner submits the team "Eastside FC" with manager "Dana Cole"
- **Then** the system creates the team in the league
- **And** "Dana Cole" is its manager

#### Scenario: Owner creates a team with a name already used in the league

- **Given** the team "Eastside FC" exists in the league
- **And** the Owner is logged in
- **When** the Owner submits another team named "Eastside FC" for the league
- **Then** the system returns `ValidationError`

#### Scenario: Owner picks a user who is not a manager

- **Given** the Owner is logged in
- **And** a user with the role `REFEREE` exists
- **When** the Owner submits a team with that user as manager
- **Then** the system returns `ValidationError`
- **And** no team is created

#### Scenario: One manager runs two teams

- **Given** "Dana Cole" manages "Eastside FC"
- **And** the Owner is logged in
- **When** the Owner creates "Westside FC" with manager "Dana Cole"
- **Then** "Dana Cole" manages both teams

#### Scenario: Manager tries to create a team

- **Given** a manager is logged in
- **When** the manager submits a new team
- **Then** the system returns `NotAuthorizedError`

### US-6.2 — Set a team's home location

#### Scenario: Owner sets a home location

- **Given** the Owner is logged in
- **And** the location "Riverside Field 1" exists
- **When** the Owner sets it as the home location of "Eastside FC"
- **Then** the team shows that home location

#### Scenario: Owner sets a home location that does not exist

- **Given** the Owner is logged in
- **When** the Owner sets an unknown location on a team
- **Then** the system returns `NotFoundError`

### US-6.3 — View teams

#### Scenario: Visitor views the teams of a league

- **Given** the league has the teams "Eastside FC" and "Westside FC"
- **And** nobody is logged in
- **When** the visitor opens the league's teams
- **Then** both teams are listed

### US-6.4 — Edit my team

#### Scenario: Manager renames their team

- **Given** a manager manages "Eastside FC"
- **When** the manager renames it to "Eastside United"
- **Then** the team shows the new name

#### Scenario: Manager edits another manager's team

- **Given** a manager manages only "Eastside FC"
- **When** the manager changes the name of "Westside FC"
- **Then** the system returns `NotAuthorizedError`

#### Scenario: Manager tries to change their team's manager

- **Given** a manager manages "Eastside FC"
- **When** the manager sets another user as the team's manager
- **Then** the system returns `NotAuthorizedError`

### US-6.5 — Retire or delete a team

#### Scenario: Owner marks a team inactive

- **Given** the Owner is logged in
- **When** the Owner sets a team to `INACTIVE`
- **Then** its games stay in the records
- **And** the team is left out when all games of a season are created

#### Scenario: Owner deletes a team that has games

- **Given** the Owner is logged in
- **And** the team has a game
- **When** the Owner deletes the team
- **Then** the system shows "Team has games and cannot be deleted."
- **And** the team stays

#### Scenario: Owner deletes a team without games

- **Given** the Owner is logged in
- **And** the team has no games
- **When** the Owner deletes the team
- **Then** the team and its players are removed

#### Scenario: Owner moves a team that has games to another league

- **Given** the Owner is logged in
- **And** the team has a game
- **When** the Owner changes the team's league
- **Then** the system returns `ValidationError`

---
