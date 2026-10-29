# Feature: Sport Management

**Feature ID:** 2  
**Branch pattern:** `feature/2-sport-management`  
**Status:** Draft  
**Created:** 2026-10-18  
**Input:** The Owner keeps the list of sports the league runs. This system is set up for Soccer. Sports sit at the top of the league structure: a sport has leagues.  
**Depends on:** [Feature 1 — User Auth](feature-1-user-auth.md) (log in and roles)

---

## User Stories

### US-2.1: Add a sport

**As an** Owner
**I want to** add a sport
**So that** leagues can be created under it

**Priority:** P1
**Independent test:** Add the sport "Futsal"; it shows up in the sports list
**Acceptance scenarios:** see ### US-2.1 under Acceptance Criteria

### US-2.2: View sports

**As a** manager, referee, or public visitor
**I want to** see the list of sports
**So that** I can find the leagues of a sport

**Priority:** P1
**Independent test:** Open the sports list without logging in; sports show in alphabetical order
**Acceptance scenarios:** see ### US-2.2 under Acceptance Criteria

### US-2.3: Edit a sport

**As an** Owner
**I want to** change a sport's name or description
**So that** the list stays correct

**Priority:** P2
**Independent test:** Change the description of "Soccer"; the list shows the new text
**Acceptance scenarios:** see ### US-2.3 under Acceptance Criteria

### US-2.4: Delete a sport

**As an** Owner
**I want to** delete a sport that has no leagues
**So that** the list only has sports we really run

**Priority:** P2
**Independent test:** Delete a sport with no leagues; it is gone. Try a sport with leagues; it stays
**Acceptance scenarios:** see ### US-2.4 under Acceptance Criteria

---



## Requirements



### Functional Requirements

- **FR-001**: The system MUST let the Owner add, edit, and delete sports.
- **FR-002**: Everyone, including visitors who are not logged in, MUST be able to view sports. Only the Owner can change them.
- **FR-003**: A sport MUST have a name. The name MUST be 1–50 characters and unique, ignoring upper and lower case.
- **FR-004**: The sport "Soccer" MUST exist when the system is first set up.
- **FR-005**: The system MUST NOT delete a sport that has leagues.
- **FR-006**: Sports MUST be listed alphabetically by name.

---



## Assumptions

- This league runs Soccer. The sport list is kept because the class design puts Sports above Leagues.
- Leagues are built in [Feature 3](feature-3-league-management.md).
- Sports are shared by everyone. A sport does not belong to one user.

---



## Edge Cases

- Empty or blank name → `ValidationError`
- Name longer than 50 characters → `ValidationError`
- Name already used (any letter case) → `ValidationError` with "Sport name is already taken."
- Sport id does not exist → `NotFoundError`
- Delete a sport that has leagues → `ValidationError` with "Sport has leagues and cannot be deleted."
- Anyone other than the Owner adds, edits, or deletes → `NotAuthorizedError`

---



## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: "Soccer" is in the list on a fresh install
- **SC-003**: A sport that has leagues can never be deleted

---



## Data Ownership & Isolation

- Sports are shared records. Nobody owns them.
- Anyone can read. Only the Owner can add, edit, or delete.

---



## Key Entities

- **Sport**: a kind of game the league runs. Has a name and an optional description
- **League**: belongs to exactly one sport (see Feature 3)

---



## Acceptance Criteria (Gherkin)



### US-2.1 — Add a sport



#### Scenario: Owner adds a sport

- **Given** the Owner is logged in
- **When** the Owner submits the name "Futsal"
- **Then** the system creates the sport "Futsal"



#### Scenario: Owner adds a sport with a name that is already used

- **Given** the sport "Soccer" exists
- **And** the Owner is logged in
- **When** the Owner submits the name "soccer"
- **Then** the system shows "Sport name is already taken."
- **And** no sport is created



#### Scenario: Owner adds a sport with a blank name

- **Given** the Owner is logged in
- **When** the Owner submits a name with only spaces
- **Then** the system returns `ValidationError`
- **And** no sport is created



#### Scenario: Manager tries to add a sport

- **Given** a manager is logged in
- **When** the manager submits a new sport
- **Then** the system returns `NotAuthorizedError`



### US-2.2 — View sports



#### Scenario: Visitor views sports in alphabetical order

- **Given** the sports "Soccer" and "Futsal" exist
- **And** nobody is logged in
- **When** the visitor opens the sports list
- **Then** "Futsal" is listed before "Soccer"



#### Scenario: Soccer exists on a fresh install

- **Given** the system was just set up
- **When** anyone opens the sports list
- **Then** the list includes "Soccer"



### US-2.3 — Edit a sport



#### Scenario: Owner changes a sport's description

- **Given** the Owner is logged in
- **And** the sport "Soccer" exists
- **When** the Owner sets the description to "Outdoor 11-a-side"
- **Then** the sport shows the new description



#### Scenario: Owner renames a sport to a name that is already used

- **Given** the sports "Soccer" and "Futsal" exist
- **And** the Owner is logged in
- **When** the Owner renames "Futsal" to "Soccer"
- **Then** the system shows "Sport name is already taken."
- **And** "Futsal" keeps its name



### US-2.4 — Delete a sport



#### Scenario: Owner deletes a sport with no leagues

- **Given** the Owner is logged in
- **And** the sport "Futsal" has no leagues
- **When** the Owner deletes "Futsal"
- **Then** the sport is removed



#### Scenario: Owner deletes a sport that has leagues

- **Given** the Owner is logged in
- **And** the sport "Soccer" has a league
- **When** the Owner deletes "Soccer"
- **Then** the system shows "Sport has leagues and cannot be deleted."
- **And** the sport stays

---

