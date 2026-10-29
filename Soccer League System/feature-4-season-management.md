# Feature: Season Management

**Feature ID:** 4
**Branch pattern:** `feature/4-season-management`
**Status:** Draft
**Created:** 2026-10-18
**Input:** The Owner creates the seasons of a league (for example "Fall 2026") with start and end dates, and closes them when they are over. A season holds the games for that period.
**Depends on:** [Feature 3 — League Management](feature-3-league-management.md), [Feature 1 — User Auth](feature-1-user-auth.md)

---

## User Stories

### US-4.1: Create a season

**As an** Owner
**I want to** create a season for a league with a start and end date
**So that** I can schedule games in that period

**Priority:** P1
**Independent test:** Create "Fall 2026" for a league; it shows up with status Planned
**Acceptance scenarios:** see ### US-4.1 under Acceptance Criteria

### US-4.2: View seasons

**As a** manager, referee, or public visitor
**I want to** see a league's seasons and their dates
**So that** I can find current and past games

**Priority:** P1
**Independent test:** Open a league's seasons without logging in; only Active and Completed seasons show
**Acceptance scenarios:** see ### US-4.2 under Acceptance Criteria

### US-4.3: Edit a season

**As an** Owner
**I want to** change a season's name, dates, or status
**So that** it matches the real calendar

**Priority:** P2
**Independent test:** Change a Planned season to Active; the list shows Active
**Acceptance scenarios:** see ### US-4.3 under Acceptance Criteria

### US-4.4: Complete a season

**As an** Owner
**I want to** mark a season Completed
**So that** its games and results stay as league history

**Priority:** P2
**Independent test:** Complete a season with results; the games and scores are still viewable
**Acceptance scenarios:** see ### US-4.4 under Acceptance Criteria

### US-4.5: Delete a season

**As an** Owner
**I want to** delete a season that has no games
**So that** mistakes can be removed without losing history

**Priority:** P2
**Independent test:** Delete an empty season; it is gone. Try one with games; it stays
**Acceptance scenarios:** see ### US-4.5 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: The system MUST let the Owner create, edit, and delete seasons.
- **FR-002**: Each season MUST belong to exactly one league.
- **FR-003**: A season MUST have a name (1–100 characters, unique within its league), a start date, and an end date. The end date MUST NOT be before the start date.
- **FR-004**: Seasons of the same league MUST NOT have overlapping dates.
- **FR-005**: A season MUST have a status of `PLANNED`, `ACTIVE`, or `COMPLETED`. A new season MUST start as `PLANNED`.
- **FR-006**: The Owner, managers, and referees MUST see all seasons. Visitors who are not logged in MUST see only `ACTIVE` and `COMPLETED` seasons.
- **FR-007**: The system MUST NOT let the Owner change season dates if a game of that season would fall outside the new dates.
- **FR-008**: Games, results, and statistics of a `COMPLETED` season MUST stay viewable.
- **FR-009**: The system MUST NOT delete a season that has games.

---

## Assumptions

- Games are created in [Feature 5](feature-5-game-management.md). That feature also creates all games of a season at once.
- Only the Owner changes the status. The order is Planned, then Active, then Completed.
- Seasons are shared. A season does not belong to one user.

---

## Edge Cases

- Blank name → `ValidationError`
- End date before start date → `ValidationError`
- Name already used in the same league → `ValidationError`
- Dates overlap another season of the league → `ValidationError`
- League does not exist → `NotFoundError`
- Season id does not exist → `NotFoundError`
- Shorten a season so a game falls outside → `ValidationError`
- Delete a season that has games → `ValidationError` with "Season has games and cannot be deleted."
- Visitor asks for a `PLANNED` season → `NotFoundError`
- Anyone other than the Owner creates, edits, or deletes → `NotAuthorizedError`

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A visitor never sees a `PLANNED` season
- **SC-003**: Completing a season never removes a game or result

---

## Data Ownership & Isolation

- Seasons are shared records. Nobody owns them.
- Anyone can read, with the visitor limit in FR-006. Only the Owner can create, edit, or delete.

---

## Key Entities

- **Season**: a dated period of play for one league. Has a name, start date, end date, and status
- **League**: the parent of the season (Feature 3)
- **Game**: belongs to one season (Feature 5)

---

## Acceptance Criteria (Gherkin)

### US-4.1 — Create a season

#### Scenario: Owner creates a season

- **Given** the Owner is logged in
- **And** the league "Fall Adult League" exists
- **When** the Owner submits "Fall 2026" from 2026-09-05 to 2026-11-14
- **Then** the system creates the season with status `PLANNED`

#### Scenario: Owner creates a season that ends before it starts

- **Given** the Owner is logged in
- **When** the Owner submits an end date before the start date
- **Then** the system returns `ValidationError`
- **And** no season is created

#### Scenario: Owner creates a season that overlaps another

- **Given** a season runs from 2026-09-05 to 2026-11-14 in the league
- **And** the Owner is logged in
- **When** the Owner submits a season from 2026-11-01 to 2026-12-20 for the same league
- **Then** the system returns `ValidationError`

#### Scenario: Owner creates a season with a name already used in the league

- **Given** the season "Fall 2026" exists in the league
- **And** the Owner is logged in
- **When** the Owner submits another season named "Fall 2026" for the league
- **Then** the system returns `ValidationError`

#### Scenario: Manager tries to create a season

- **Given** a manager is logged in
- **When** the manager submits a new season
- **Then** the system returns `NotAuthorizedError`

### US-4.2 — View seasons

#### Scenario: Visitor sees only Active and Completed seasons

- **Given** the league has seasons "Spring 2026" (`COMPLETED`), "Fall 2026" (`ACTIVE`), and "Winter 2027" (`PLANNED`)
- **And** nobody is logged in
- **When** the visitor opens the league's seasons
- **Then** "Spring 2026" and "Fall 2026" are listed
- **And** "Winter 2027" is not listed

#### Scenario: Owner sees every season

- **Given** the same three seasons exist
- **And** the Owner is logged in
- **When** the Owner opens the league's seasons
- **Then** all three seasons are listed

#### Scenario: Visitor asks for a Planned season directly

- **Given** the season "Winter 2027" is `PLANNED`
- **When** a visitor asks for that season
- **Then** the system returns `NotFoundError`

### US-4.3 — Edit a season

#### Scenario: Owner activates a season

- **Given** the Owner is logged in
- **And** a season is `PLANNED`
- **When** the Owner sets the status to `ACTIVE`
- **Then** the season shows `ACTIVE`

#### Scenario: Owner shortens a season so a game falls outside

- **Given** the Owner is logged in
- **And** a game of the season is on 2026-11-14
- **When** the Owner changes the end date to 2026-11-07
- **Then** the system returns `ValidationError`
- **And** the dates stay the same

### US-4.4 — Complete a season

#### Scenario: Owner completes a season and the games stay visible

- **Given** the Owner is logged in
- **And** the season has games with results
- **When** the Owner sets the status to `COMPLETED`
- **Then** the season shows `COMPLETED`
- **And** a visitor can still view its games and scores

### US-4.5 — Delete a season

#### Scenario: Owner deletes a season with no games

- **Given** the Owner is logged in
- **And** the season has no games
- **When** the Owner deletes the season
- **Then** the season is removed

#### Scenario: Owner deletes a season that has games

- **Given** the Owner is logged in
- **And** the season has a game
- **When** the Owner deletes the season
- **Then** the system shows "Season has games and cannot be deleted."
- **And** the season stays

---
