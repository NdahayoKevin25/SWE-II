# Feature: Location Management

**Feature ID:** 8
**Branch pattern:** `feature/8-location-management`
**Status:** Draft
**Created:** 2026-10-18
**Input:** The Owner keeps the list of fields where games are played. A team can have a home location, and a game's location can change.
**Depends on:** [Feature 1 — User Auth](feature-1-user-auth.md)

---

## User Stories

### US-8.1: Add a location

**As an** Owner
**I want to** add a field with its address
**So that** teams and games can use it

**Priority:** P1
**Independent test:** Add "Riverside Field 1" with an address; it shows in the list
**Acceptance scenarios:** see ### US-8.1 under Acceptance Criteria

### US-8.2: View locations

**As a** manager, referee, or public visitor
**I want to** see the fields and their addresses
**So that** I know where to go

**Priority:** P1
**Independent test:** Open the locations list without logging in; names and addresses show
**Acceptance scenarios:** see ### US-8.2 under Acceptance Criteria

### US-8.3: Edit a location

**As an** Owner
**I want to** fix a field's name, address, or notes
**So that** directions stay correct

**Priority:** P2
**Independent test:** Change an address; upcoming games at that field show the new address
**Acceptance scenarios:** see ### US-8.3 under Acceptance Criteria

### US-8.4: Retire or delete a location

**As an** Owner
**I want to** retire a field we no longer use, or delete one that was never used
**So that** the list stays current and old games keep their field

**Priority:** P2
**Independent test:** Delete a field with no games or teams; it is gone. Try a used one; it stays
**Acceptance scenarios:** see ### US-8.4 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: The system MUST let the Owner add, edit, retire, and delete locations.
- **FR-002**: Everyone, including visitors who are not logged in, MUST be able to view locations. Only the Owner can change them.
- **FR-003**: A location MUST have a name (1–100 characters, unique ignoring letter case) and an address (1–200 characters). It MAY have notes up to 500 characters.
- **FR-004**: A location MUST have a status of `ACTIVE` or `INACTIVE`. A new location MUST be `ACTIVE`.
- **FR-005**: An `INACTIVE` location MUST NOT be offered for new games or as a team's home location. Past games MUST still show it.
- **FR-006**: The system MUST NOT delete a location that a team or a game uses. The Owner can set it `INACTIVE` instead.
- **FR-007**: Changing a location's name or address MUST show everywhere the location is used, including upcoming games.

---

## Assumptions

- A location is one field or venue. It is not a city or a region.
- Teams ([Feature 6](feature-6-team-management.md)) use a location as their home. Games ([Feature 5](feature-5-game-management.md)) happen at a location.
- Changing the location of one game is done in Feature 5.

---

## Edge Cases

- Blank name or address → `ValidationError`
- Name already used (any letter case) → `ValidationError` with "Location name is already taken."
- Location id does not exist → `NotFoundError`
- Delete a location used by a team or a game → `ValidationError` with "Location is in use and cannot be deleted."
- Anyone other than the Owner changes a location → `NotAuthorizedError`

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A location that a team or game uses can never be deleted
- **SC-003**: A retired location still shows on past games

---

## Data Ownership & Isolation

- Locations are shared records. Nobody owns them.
- Anyone can read. Only the Owner can add, edit, retire, or delete.

---

## Key Entities

- **Location**: a field where games are played. Has a name, address, optional notes, and status
- **Team**: may have a location as its home (Feature 6)
- **Game**: happens at one location (Feature 5)

---

## Acceptance Criteria (Gherkin)

### US-8.1 — Add a location

#### Scenario: Owner adds a location

- **Given** the Owner is logged in
- **When** the Owner submits the name "Riverside Field 1" and the address "100 River Rd"
- **Then** the system creates the location as `ACTIVE`

#### Scenario: Owner adds a location with a name already used

- **Given** the location "Riverside Field 1" exists
- **And** the Owner is logged in
- **When** the Owner submits the name "riverside field 1"
- **Then** the system shows "Location name is already taken."

#### Scenario: Owner adds a location without an address

- **Given** the Owner is logged in
- **When** the Owner submits a blank address
- **Then** the system returns `ValidationError`
- **And** no location is created

#### Scenario: Manager tries to add a location

- **Given** a manager is logged in
- **When** the manager submits a new location
- **Then** the system returns `NotAuthorizedError`

### US-8.2 — View locations

#### Scenario: Visitor views locations

- **Given** the locations "Riverside Field 1" and "City Park Pitch" exist
- **And** nobody is logged in
- **When** the visitor opens the locations list
- **Then** both locations are listed with their addresses

### US-8.3 — Edit a location

#### Scenario: Owner fixes an address used by an upcoming game

- **Given** the Owner is logged in
- **And** an upcoming game is at "Riverside Field 1"
- **When** the Owner changes the address of "Riverside Field 1"
- **Then** the upcoming game shows the new address

### US-8.4 — Retire or delete a location

#### Scenario: Owner retires a location used by past games

- **Given** the Owner is logged in
- **And** a past game was at "Riverside Field 1"
- **When** the Owner sets "Riverside Field 1" to `INACTIVE`
- **Then** the past game still shows "Riverside Field 1"
- **And** the location is not offered for new games

#### Scenario: Owner deletes a location that a game uses

- **Given** the Owner is logged in
- **And** a game is at "Riverside Field 1"
- **When** the Owner deletes "Riverside Field 1"
- **Then** the system shows "Location is in use and cannot be deleted."
- **And** the location stays

#### Scenario: Owner deletes a location that nothing uses

- **Given** the Owner is logged in
- **And** no team or game uses "City Park Pitch"
- **When** the Owner deletes "City Park Pitch"
- **Then** the location is removed

---
