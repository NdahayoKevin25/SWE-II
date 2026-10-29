# Feature: League Management

**Feature ID:** 3  
**Branch pattern:** `feature/3-league-management`  
**Status:** Draft  
**Created:** 2026-10-18  
**Input:** The Owner creates and maintains leagues (for example a "Fall Adult League"). A league belongs to one sport and holds that league's teams and seasons.  
**Depends on:** [Feature 2 — Sport Management](feature-2-sport-management.md), [Feature 1 — User Auth](feature-1-user-auth.md)

---

## User Stories

### US-3.1: Create a league

**As an** Owner
**I want to** create a league under a sport
**So that** teams and seasons have a place to live

**Priority:** P1
**Independent test:** Create "Fall Adult League" under Soccer; it shows up in the leagues list
**Acceptance scenarios:** see ### US-3.1 under Acceptance Criteria

### US-3.2: View leagues

**As a** manager, referee, or public visitor
**I want to** see the leagues and their sport
**So that** I can find the league I care about

**Priority:** P1
**Independent test:** Open the leagues list without logging in and filter by sport
**Acceptance scenarios:** see ### US-3.2 under Acceptance Criteria

### US-3.3: Edit a league

**As an** Owner
**I want to** change a league's name or description
**So that** the league information stays correct

**Priority:** P2
**Independent test:** Rename a league; the list shows the new name
**Acceptance scenarios:** see ### US-3.3 under Acceptance Criteria

### US-3.4: Delete a league

**As an** Owner
**I want to** delete a league that has no teams or seasons
**So that** old records are never lost, but mistakes can be removed

**Priority:** P2
**Independent test:** Delete an empty league; it is gone. Try a league with a season; it stays
**Acceptance scenarios:** see ### US-3.4 under Acceptance Criteria

---



## Requirements



### Functional Requirements

- **FR-001**: The system MUST let the Owner create, edit, and delete leagues.
- **FR-002**: Everyone, including visitors who are not logged in, MUST be able to view leagues. Only the Owner can change them.
- **FR-003**: Each league MUST belong to exactly one existing sport.
- **FR-004**: A league MUST have a name. The name MUST be 1–100 characters and unique within its sport.
- **FR-005**: The system MUST NOT let the Owner change a league's sport once the league has a team or a season.
- **FR-006**: The system MUST NOT delete a league that has a team or a season.
- **FR-007**: Leagues MUST be listed alphabetically by name. The list MUST be filterable by sport.

---



## Assumptions

- A league can have many teams ([Feature 6](feature-6-team-management.md)) and many seasons ([Feature 4](feature-4-season-management.md)).
- Leagues are shared by everyone. A league does not belong to one user.

---



## Edge Cases

- Blank name → `ValidationError`
- Name longer than 100 characters → `ValidationError`
- Name already used in the same sport → `ValidationError` with "League name is already taken for this sport."
- Sport does not exist → `NotFoundError`
- League id does not exist → `NotFoundError`
- Change the sport of a league that has a team or a season → `ValidationError`
- Delete a league that has a team or a season → `ValidationError` with "League has teams or seasons and cannot be deleted."
- Anyone other than the Owner creates, edits, or deletes → `NotAuthorizedError`

---



## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A league with teams or seasons can never be deleted
- **SC-003**: A visitor can list leagues without logging in

---



## Data Ownership & Isolation

- Leagues are shared records. Nobody owns them.
- Anyone can read. Only the Owner can create, edit, or delete.

---



## Key Entities

- **League**: a group of teams that play together. Has a name, an optional description, and a sport
- **Sport**: the parent of the league (Feature 2)
- **Team** and **Season**: belong to one league (Features 6 and 4)

---



## Acceptance Criteria (Gherkin)



### US-3.1 — Create a league



#### Scenario: Owner creates a league

- **Given** the Owner is logged in
- **And** the sport "Soccer" exists
- **When** the Owner submits the name "Fall Adult League" for "Soccer"
- **Then** the system creates the league
- **And** the league belongs to "Soccer"



#### Scenario: Owner creates a league with a name already used in that sport

- **Given** the league "Fall Adult League" exists in "Soccer"
- **And** the Owner is logged in
- **When** the Owner submits the name "Fall Adult League" for "Soccer"
- **Then** the system shows "League name is already taken for this sport."



#### Scenario: Owner creates a league for a sport that does not exist

- **Given** the Owner is logged in
- **When** the Owner submits a league for an unknown sport
- **Then** the system returns `NotFoundError`
- **And** no league is created



#### Scenario: Manager tries to create a league

- **Given** a manager is logged in
- **When** the manager submits a new league
- **Then** the system returns `NotAuthorizedError`



### US-3.2 — View leagues



#### Scenario: Visitor views leagues

- **Given** the leagues "Fall Adult League" and "Spring League" exist
- **And** nobody is logged in
- **When** the visitor opens the leagues list
- **Then** both leagues are listed in alphabetical order with their sport



#### Scenario: Visitor filters leagues by sport

- **Given** leagues exist in "Soccer" and in "Futsal"
- **When** the visitor filters by "Soccer"
- **Then** only the "Soccer" leagues are listed



### US-3.3 — Edit a league



#### Scenario: Owner renames a league

- **Given** the Owner is logged in
- **And** the league "Fall Adult League" exists
- **When** the Owner renames it to "Fall Premier League"
- **Then** the league shows the new name



#### Scenario: Owner changes the sport of a league that has a season

- **Given** the Owner is logged in
- **And** the league has a season
- **When** the Owner changes the league's sport
- **Then** the system returns `ValidationError`
- **And** the league keeps its sport



### US-3.4 — Delete a league



#### Scenario: Owner deletes an empty league

- **Given** the Owner is logged in
- **And** the league has no teams and no seasons
- **When** the Owner deletes the league
- **Then** the league is removed



#### Scenario: Owner deletes a league that has a season

- **Given** the Owner is logged in
- **And** the league has a season
- **When** the Owner deletes the league
- **Then** the system shows "League has teams or seasons and cannot be deleted."
- **And** the league stays



#### Scenario: Owner deletes a league that has a team

- **Given** the Owner is logged in
- **And** the league has a team
- **When** the Owner deletes the league
- **Then** the system shows "League has teams or seasons and cannot be deleted."
- **And** the league stays

---

