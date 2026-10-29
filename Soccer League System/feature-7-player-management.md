# Feature: Player Management

**Feature ID:** 7
**Branch pattern:** `feature/7-player-management`
**Status:** Draft
**Created:** 2026-10-18
**Input:** Managers add and maintain the players on their own teams. Everyone can see rosters and each player's statistics. The Owner can fix any roster.
**Depends on:** [Feature 6 — Team Management](feature-6-team-management.md), [Feature 1 — User Auth](feature-1-user-auth.md)

---

## User Stories

### US-7.1: Add a player to my team

**As a** manager
**I want to** add a player to my team
**So that** the player is on the roster and can be in games

**Priority:** P1
**Independent test:** As a manager, add "Jo Rivera" to my team; the roster shows Jo
**Acceptance scenarios:** see ### US-7.1 under Acceptance Criteria

### US-7.2: View a roster

**As a** manager, referee, or public visitor
**I want to** see the players on a team
**So that** I know who plays for the team

**Priority:** P1
**Independent test:** Open a team's roster without logging in; active players show
**Acceptance scenarios:** see ### US-7.2 under Acceptance Criteria

### US-7.3: Edit a player

**As a** manager
**I want to** fix a player's name or jersey number
**So that** the roster stays correct

**Priority:** P2
**Independent test:** Change a player's jersey number; the roster shows the new number
**Acceptance scenarios:** see ### US-7.3 under Acceptance Criteria

### US-7.4: Remove a player

**As a** manager
**I want to** remove a player who left the team
**So that** the roster is current but the player's past stats stay

**Priority:** P2
**Independent test:** Remove a player who has stats; the player is hidden from the roster but the stats remain
**Acceptance scenarios:** see ### US-7.4 under Acceptance Criteria

### US-7.5: View a player's statistics

**As a** manager or public visitor
**I want to** see a player's statistics for each game and in total
**So that** I can see how each player does

**Priority:** P2
**Independent test:** Open a player with stats in 3 games; the per-game numbers and the totals show
**Acceptance scenarios:** see ### US-7.5 under Acceptance Criteria

### US-7.6: Fix any roster

**As an** Owner
**I want to** add, edit, remove, and move players on any team
**So that** I can fix problems for a manager

**Priority:** P2
**Independent test:** As the Owner, add a player to any team; the roster shows the player
**Acceptance scenarios:** see ### US-7.6 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: A manager MUST be able to add, edit, and remove players only on teams they manage.
- **FR-002**: The Owner MUST be able to add, edit, remove, and move players on any team. Referees and visitors cannot change players.
- **FR-003**: Each player MUST belong to exactly one team.
- **FR-004**: A player MUST have a first name and a last name, each 1–50 characters.
- **FR-005**: A player MAY have a jersey number from 0 to 99. Two active players of one team MUST NOT have the same number.
- **FR-006**: A player MUST have a status of `ACTIVE` or `INACTIVE`. Rosters MUST show only `ACTIVE` players, sorted by last name then first name.
- **FR-007**: Removing a player who has statistics MUST set the player to `INACTIVE`. Removing a player with no statistics MAY delete the player.
- **FR-008**: A player's statistics MUST be kept after the player is removed or moved. They stay with the games already played.
- **FR-009**: Everyone, including visitors who are not logged in, MUST be able to view rosters and player statistics. Visitors see name, jersey number, team, and statistics only.
- **FR-010**: Player statistics MUST show each game and totals for goals, assists, yellow cards, and red cards.

---

## Assumptions

- Statistics are entered in [Feature 5](feature-5-game-management.md). This feature only shows them.
- A player is on one team at a time. Only the Owner moves a player to another team.
- No private details like email or phone are kept for players.

---

## Edge Cases

- Blank first or last name → `ValidationError`
- Name longer than 50 characters → `ValidationError`
- Jersey number outside 0–99, or already used by an active player of the team → `ValidationError`
- Team does not exist → `NotFoundError`
- Player id does not exist → `NotFoundError`
- Manager changes a player of a team they do not manage → `NotAuthorizedError`
- Referee or visitor changes a player → `NotAuthorizedError`

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A manager can only change players on teams they manage
- **SC-003**: Removing or moving a player never deletes the player's statistics

---

## Data Ownership & Isolation

- A player belongs to one team. The team's manager looks after that player.
- Managers change only their own team's players. The Owner changes any player.
- Referees and visitors can only read.

---

## Key Entities

- **Player**: a person on a team's roster. Has a first name, last name, jersey number, and status
- **Team**: the roster the player belongs to (Feature 6)
- **PlayerStat**: the player's statistics for one game (Feature 5)

---

## Acceptance Criteria (Gherkin)

### US-7.1 — Add a player to my team

#### Scenario: Manager adds a player to their team

- **Given** the manager is logged in
- **And** the manager manages "Eastside FC"
- **When** the manager submits the first name "Jo", the last name "Rivera", and the jersey number 9
- **Then** the system creates the player
- **And** the player is on the roster of "Eastside FC"

#### Scenario: Manager adds a player with a blank last name

- **Given** a manager manages "Eastside FC"
- **When** the manager submits a player with a blank last name
- **Then** the system returns `ValidationError`
- **And** no player is created

#### Scenario: Manager adds a player with a jersey number already used

- **Given** an active player of "Eastside FC" wears number 9
- **And** the manager manages "Eastside FC"
- **When** the manager adds another player with number 9
- **Then** the system returns `ValidationError`

#### Scenario: Manager adds a player to another team

- **Given** a manager manages only "Eastside FC"
- **When** the manager adds a player to "Westside FC"
- **Then** the system returns `NotAuthorizedError`
- **And** no player is created

### US-7.2 — View a roster

#### Scenario: Visitor views a roster

- **Given** "Eastside FC" has the active players "Rivera, Jo" and "Adams, Sam" and one inactive player
- **And** nobody is logged in
- **When** the visitor opens the roster of "Eastside FC"
- **Then** "Adams, Sam" is listed before "Rivera, Jo"
- **And** the inactive player is not listed

### US-7.3 — Edit a player

#### Scenario: Manager changes a jersey number

- **Given** a manager manages "Eastside FC"
- **And** "Jo Rivera" is on "Eastside FC"
- **When** the manager changes the jersey number to 11
- **Then** the roster shows number 11 for "Jo Rivera"

#### Scenario: Referee tries to edit a player

- **Given** a referee is logged in
- **When** the referee edits a player
- **Then** the system returns `NotAuthorizedError`

### US-7.4 — Remove a player

#### Scenario: Manager removes a player who has statistics

- **Given** a manager manages "Eastside FC"
- **And** "Jo Rivera" has statistics from games
- **When** the manager removes "Jo Rivera"
- **Then** "Jo Rivera" is `INACTIVE` and hidden from the roster
- **And** the statistics stay

#### Scenario: Manager removes a player who has no statistics

- **Given** a manager manages "Eastside FC"
- **And** "Alex Kim" has no statistics
- **When** the manager removes "Alex Kim"
- **Then** the player is deleted

### US-7.5 — View a player's statistics

#### Scenario: Visitor views a player's statistics

- **Given** "Jo Rivera" has statistics in 3 games
- **And** nobody is logged in
- **When** the visitor opens the statistics of "Jo Rivera"
- **Then** the system shows the numbers for each game
- **And** the totals for goals, assists, yellow cards, and red cards

#### Scenario: Statistics of an inactive player stay visible

- **Given** "Jo Rivera" is `INACTIVE` and has statistics
- **When** a visitor opens the statistics of "Jo Rivera"
- **Then** the statistics are shown

### US-7.6 — Fix any roster

#### Scenario: Owner adds a player to any team

- **Given** the Owner is logged in
- **When** the Owner submits a valid player for "Westside FC"
- **Then** the player is on the roster of "Westside FC"

#### Scenario: Owner moves a player to another team

- **Given** the Owner is logged in
- **And** "Jo Rivera" has statistics from games for "Eastside FC"
- **When** the Owner moves "Jo Rivera" to "Westside FC"
- **Then** "Jo Rivera" is on the roster of "Westside FC"
- **And** the earlier statistics stay with the earlier games

---
