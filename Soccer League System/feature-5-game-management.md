# Feature: Game Management

**Feature ID:** 5
**Branch pattern:** `feature/5-game-management`
**Status:** Draft
**Created:** 2026-10-18
**Input:** The Owner schedules games for a season, one at a time or all at once. Games can change location or date, and finished games keep their score and player statistics as league history. Managers enter scores and statistics for their own team's games.
**Depends on:** [Feature 4 — Season Management](feature-4-season-management.md), [Feature 6 — Team Management](feature-6-team-management.md), [Feature 7 — Player Management](feature-7-player-management.md), [Feature 8 — Location Management](feature-8-location-management.md), [Feature 1 — User Auth](feature-1-user-auth.md)

---

## User Stories

### US-5.1: Add a game

**As an** Owner
**I want to** add one game with two teams, a date, a time, and a location
**So that** I can schedule a game by hand

**Priority:** P1
**Independent test:** Add a game between two teams of a season; it shows in the season's games as Scheduled
**Acceptance scenarios:** see ### US-5.1 under Acceptance Criteria

### US-5.2: Create all games for a season

**As an** Owner
**I want to** create every game of a season in one step
**So that** I do not schedule dozens of games by hand

**Priority:** P1
**Independent test:** Create games for a season with 6 teams and 10 games per team; every team has exactly 10 games
**Acceptance scenarios:** see ### US-5.2 under Acceptance Criteria

### US-5.3: View games

**As a** manager, referee, or public visitor
**I want to** see games with their date, time, location, and score
**So that** I know when and where to play and how games ended

**Priority:** P1
**Independent test:** Open a season's games without logging in; upcoming and finished games show
**Acceptance scenarios:** see ### US-5.3 under Acceptance Criteria

### US-5.4: Change a game's date or location

**As an** Owner
**I want to** change when or where a game is played
**So that** the schedule follows real conditions

**Priority:** P1
**Independent test:** Move a Scheduled game to another location; the game shows the new location
**Acceptance scenarios:** see ### US-5.4 under Acceptance Criteria

### US-5.5: Postpone or cancel a game

**As an** Owner
**I want to** postpone or cancel a game
**So that** everyone can see it will not be played as planned

**Priority:** P2
**Independent test:** Postpone a game; its status shows Postponed
**Acceptance scenarios:** see ### US-5.5 under Acceptance Criteria

### US-5.6: Enter a game score

**As a** manager
**I want to** enter the final score of my team's game
**So that** the result is recorded

**Priority:** P1
**Independent test:** Enter 3–1 for a game of my team; the game becomes Completed with that score
**Acceptance scenarios:** see ### US-5.6 under Acceptance Criteria

### US-5.7: Enter player statistics

**As a** manager
**I want to** enter goals, assists, and cards for my team's players in a game
**So that** each player has a record

**Priority:** P1
**Independent test:** Enter 2 goals for one of my players in a completed game; the player's stats show it
**Acceptance scenarios:** see ### US-5.7 under Acceptance Criteria

### US-5.8: Keep game history

**As an** Owner
**I want** finished games to stay in the system
**So that** the league has records of past games

**Priority:** P1
**Independent test:** Try to delete a Completed game; it stays. Delete an unused Scheduled game; it goes
**Acceptance scenarios:** see ### US-5.8 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: The system MUST let the Owner add, edit, and delete games. Managers and referees can only view them.
- **FR-002**: A game MUST have a season, a home team, an away team, a location, a date and time, and a status of `SCHEDULED`, `COMPLETED`, `POSTPONED`, or `CANCELLED`. A new game MUST start as `SCHEDULED`.
- **FR-003**: The home team and away team MUST be different teams of the season's league.
- **FR-004**: The game date MUST be inside the season's start and end dates.
- **FR-005**: A team MUST NOT have two games at the same date and time. A location MUST NOT have two games at the same date and time.
- **FR-006**: The Owner MUST be able to change the date, time, or location of a game that is not `COMPLETED`. The new location MUST be an active location.
- **FR-007**: The Owner MUST be able to postpone or cancel a game that is not `COMPLETED`. A postponed game MUST go back to `SCHEDULED` when it gets a new date and time.
- **FR-008**: To create all games of a season, the Owner MUST give games per team, the weekdays games may be played, a start time, and the minimum days between one team's games.
- **FR-009**: The system MUST show the proposed games first and save them only after the Owner confirms.
- **FR-010**: Created games MUST use only active teams of the league. Every team MUST get exactly the games-per-team number. Home and away games per team MUST differ by at most one.
- **FR-011**: Created games MUST be on the chosen weekdays, inside the season dates, and keep the minimum days between a team's games. No team or location may be double-booked.
- **FR-012**: Each created game MUST use the home team's home location. If a home team has none, the system MUST tell the Owner before saving.
- **FR-013**: If the rules cannot be met, the system MUST save no games and MUST say which rule failed. It MUST also refuse when fewer than two teams are active.
- **FR-014**: Created games MUST be normal games. Creating games again MUST replace only `SCHEDULED` games that have no score and no statistics.
- **FR-015**: The Owner MUST be able to enter a score for any game. A manager MUST be able to enter a score only for a game with a team they manage.
- **FR-016**: A score is two whole numbers from 0 to 99. A score MUST NOT be entered for a `CANCELLED` game or a game dated in the future. Saving a score MUST set the game to `COMPLETED`.
- **FR-017**: Player statistics MUST be kept per player per game: goals, assists, yellow cards, and red cards. Each is a whole number, 0 or more.
- **FR-018**: The player MUST be on one of the two teams of the game. A manager MUST enter statistics only for players of a team they manage. A team's goals MUST NOT add up to more than its score.
- **FR-019**: The Owner MUST be able to lock a game's score and statistics. A locked record can only be changed by the Owner.
- **FR-020**: The system MUST NOT delete a `COMPLETED` game. It MAY delete a `SCHEDULED` game that has no score and no statistics.
- **FR-021**: A manager MUST see all games of their teams. A referee MUST see their assigned games (see [Feature 9](feature-9-referee-management.md)). Visitors who are not logged in MUST see games, scores, and statistics of `ACTIVE` and `COMPLETED` seasons only.

---

## Assumptions

- Teams, players, and locations must exist before games can be created. They are built in Features 6, 7, and 8.
- The winner (home win, away win, or draw) is worked out from the score. It is not entered.
- Soccer statistics are goals, assists, yellow cards, and red cards. The notes only say each player has stats, so these four are a proposal.
- The settings for creating all games are given each time. They are not kept on the season.
- Referees are assigned in [Feature 9](feature-9-referee-management.md), after games exist.

---

## Edge Cases

- Same team as home and away → `ValidationError`
- Team that is not in the season's league → `ValidationError`
- Date outside the season → `ValidationError`
- Team or location already has a game at that date and time → `ValidationError`
- Inactive location chosen → `ValidationError`
- Change or delete a `COMPLETED` game → `ValidationError`
- Fewer than two active teams, or team count times games-per-team is odd → `ValidationError` and no games are saved
- Season dates too short for the rules → `ValidationError` naming the rule, and no games are saved
- Score below 0, above 99, or not a whole number → `ValidationError`
- Score for a cancelled game or a future game → `ValidationError`
- Player not on either team → `ValidationError`
- Team goals more than the score → `ValidationError`
- Manager enters data for a team they do not manage → `NotAuthorizedError`
- Manager edits a locked record → `NotAuthorizedError`
- Game id does not exist → `NotFoundError`

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: Creating all games either saves a full valid schedule or saves nothing
- **SC-003**: A `COMPLETED` game, with its score and statistics, can never be deleted
- **SC-004**: A manager can only change scores and statistics of their own team's games

---

## Data Ownership & Isolation

- Games are shared records that belong to a season.
- The Owner changes any game. A manager changes scores and statistics only for games with their own team, and only while the record is not locked.
- Referees and visitors can only read.

---

## Key Entities

- **Game**: a match between a home team and an away team. Has a season, location, date and time, and status
- **Result**: the final score of one game (home score, away score). A game has at most one
- **PlayerStat**: one player's statistics in one game (goals, assists, yellow cards, red cards)
- **Season, Team, Player, Location**: the records a game is built from (Features 4, 6, 7, and 8)

---

## Acceptance Criteria (Gherkin)

### US-5.1 — Add a game

#### Scenario: Owner adds a game

- **Given** the Owner is logged in
- **And** the season "Fall 2026" has the teams "Eastside FC" and "Westside FC"
- **When** the Owner submits a game with home "Eastside FC", away "Westside FC", a date inside the season, and the location "Riverside Field 1"
- **Then** the system creates the game with status `SCHEDULED`

#### Scenario: Owner adds a game with the same team on both sides

- **Given** the Owner is logged in
- **When** the Owner submits a game with "Eastside FC" as home and away
- **Then** the system returns `ValidationError`

#### Scenario: Owner adds a game outside the season dates

- **Given** the season ends on 2026-11-14
- **And** the Owner is logged in
- **When** the Owner submits a game on 2026-11-21
- **Then** the system returns `ValidationError`

#### Scenario: Owner double-books a team

- **Given** "Eastside FC" has a game on 2026-09-12 at 10:00
- **And** the Owner is logged in
- **When** the Owner submits another game with "Eastside FC" at that date and time
- **Then** the system returns `ValidationError`

#### Scenario: Owner double-books a location

- **Given** "Riverside Field 1" has a game on 2026-09-12 at 10:00
- **And** the Owner is logged in
- **When** the Owner submits another game at "Riverside Field 1" at that date and time
- **Then** the system returns `ValidationError`

#### Scenario: Manager tries to add a game

- **Given** a manager is logged in
- **When** the manager submits a new game
- **Then** the system returns `NotAuthorizedError`

### US-5.2 — Create all games for a season

#### Scenario: Owner creates all games for a season

- **Given** the Owner is logged in
- **And** the season has 6 active teams
- **When** the Owner asks for 10 games per team on Saturdays at 10:00 with 5 days between a team's games
- **Then** the system shows the proposed games
- **And** after the Owner confirms, every team has exactly 10 games
- **And** every game is on a Saturday inside the season dates

#### Scenario: Home and away games are balanced

- **Given** games were created for 6 active teams with 10 games per team
- **When** the Owner checks one team
- **Then** the team has 5 home games and 5 away games

#### Scenario: Created games use the home team's home location

- **Given** "Eastside FC" has the home location "Riverside Field 1"
- **When** games are created with "Eastside FC" at home
- **Then** those games are at "Riverside Field 1"

#### Scenario: A home team has no home location

- **Given** an active team has no home location
- **When** the Owner creates the games
- **Then** the system names the team and asks for a location before saving

#### Scenario: Owner cancels the proposed games

- **Given** the system shows proposed games
- **When** the Owner cancels
- **Then** no games are saved

#### Scenario: The season dates are too short

- **Given** the season is two weeks long
- **When** the Owner asks for 10 games per team
- **Then** the system returns `ValidationError` naming the rule
- **And** no games are saved

#### Scenario: Only one team is active

- **Given** the league has one active team
- **When** the Owner creates the games
- **Then** the system returns `ValidationError`

#### Scenario: Creating games again keeps finished games

- **Given** the season has 3 `COMPLETED` games and 20 `SCHEDULED` games
- **And** the Owner is logged in
- **When** the Owner creates the games again and confirms
- **Then** the 3 `COMPLETED` games stay as they are
- **And** the `SCHEDULED` games are replaced

### US-5.3 — View games

#### Scenario: Visitor views upcoming and finished games

- **Given** the season "Fall 2026" is `ACTIVE` with finished and upcoming games
- **And** nobody is logged in
- **When** the visitor opens the season's games
- **Then** upcoming games are listed by date, apart from finished games
- **And** finished games show their score

#### Scenario: Visitor cannot see games of a Planned season

- **Given** the season is `PLANNED`
- **When** a visitor asks for its games
- **Then** the system returns `NotFoundError`

#### Scenario: Manager sees their team's games

- **Given** a manager manages "Eastside FC"
- **When** the manager opens their games
- **Then** every game of "Eastside FC" is listed

### US-5.4 — Change a game's date or location

#### Scenario: Owner moves a game to another location

- **Given** the Owner is logged in
- **And** a `SCHEDULED` game is at "Riverside Field 1"
- **When** the Owner changes the location to "City Park Pitch"
- **Then** the game shows "City Park Pitch"

#### Scenario: Owner moves a game to an inactive location

- **Given** the location "Old Field" is inactive
- **And** the Owner is logged in
- **When** the Owner changes a game's location to "Old Field"
- **Then** the system returns `ValidationError`

#### Scenario: Owner reschedules a game

- **Given** the Owner is logged in
- **And** a `SCHEDULED` game exists
- **When** the Owner changes the date to another valid date
- **Then** the game shows the new date

#### Scenario: Owner changes a completed game

- **Given** the Owner is logged in
- **And** a game is `COMPLETED`
- **When** the Owner changes its location
- **Then** the system returns `ValidationError`

### US-5.5 — Postpone or cancel a game

#### Scenario: Owner postpones a game

- **Given** the Owner is logged in
- **And** a game is `SCHEDULED`
- **When** the Owner postpones it
- **Then** the game shows `POSTPONED`

#### Scenario: Postponed game gets a new date

- **Given** a game is `POSTPONED`
- **And** the Owner is logged in
- **When** the Owner sets a new valid date and time
- **Then** the game is `SCHEDULED` again

### US-5.6 — Enter a game score

#### Scenario: Manager enters the score of their team's game

- **Given** a manager manages "Eastside FC"
- **And** a `SCHEDULED` game of "Eastside FC" is dated in the past
- **When** the manager enters 3 for home and 1 for away
- **Then** the system saves the score
- **And** the game is `COMPLETED`

#### Scenario: Manager enters a score for another team's game

- **Given** a manager manages only "Eastside FC"
- **And** a game is between "Westside FC" and "Northside FC"
- **When** the manager enters a score
- **Then** the system returns `NotAuthorizedError`

#### Scenario: Manager enters a score for a future game

- **Given** a manager's team has a game dated in the future
- **When** the manager enters a score
- **Then** the system returns `ValidationError`

#### Scenario: Manager enters a score below zero

- **Given** a manager manages a team in the game
- **When** the manager enters -1 for home
- **Then** the system returns `ValidationError`

#### Scenario: Manager edits a locked score

- **Given** the Owner locked the game's record
- **And** a manager manages a team in the game
- **When** the manager changes the score
- **Then** the system returns `NotAuthorizedError`

#### Scenario: Owner corrects a score

- **Given** the Owner is logged in
- **And** a game's score is 3–1
- **When** the Owner changes it to 3–2
- **Then** the game shows 3–2

### US-5.7 — Enter player statistics

#### Scenario: Manager enters statistics for their own player

- **Given** a manager manages "Eastside FC"
- **And** "Jo Rivera" is a player of "Eastside FC" in a `COMPLETED` game
- **When** the manager enters 2 goals, 1 assist, 0 yellow cards, and 0 red cards for "Jo Rivera"
- **Then** the system saves the statistics for that game

#### Scenario: Manager enters statistics for a player not in the game

- **Given** a manager manages "Eastside FC"
- **When** the manager enters statistics for a player of a team not in the game
- **Then** the system returns `ValidationError`

#### Scenario: Player goals add up to more than the team's score

- **Given** "Eastside FC" scored 1 in the game
- **And** "Jo Rivera" already has 1 goal in it
- **When** the manager enters 1 goal for another "Eastside FC" player
- **Then** the system returns `ValidationError`

#### Scenario: Manager enters statistics for the other team's player

- **Given** a manager manages "Eastside FC"
- **When** the manager enters statistics for a "Westside FC" player
- **Then** the system returns `NotAuthorizedError`

### US-5.8 — Keep game history

#### Scenario: Owner deletes a completed game

- **Given** the Owner is logged in
- **And** a game is `COMPLETED`
- **When** the Owner deletes it
- **Then** the system returns `ValidationError`
- **And** the game and its score stay

#### Scenario: Owner deletes an unused scheduled game

- **Given** the Owner is logged in
- **And** a `SCHEDULED` game has no score and no statistics
- **When** the Owner deletes it
- **Then** the game is removed

---
