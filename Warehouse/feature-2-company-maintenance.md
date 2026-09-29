# Feature: Company Maintenance

**Feature ID:** 2
**Branch pattern:** `feature/2-company-maintenance`
**Status:** Ready
**Created:** 2026-09-28
**Input:** Keep up the one company profile (Eagle Wholesale). Its name, address, and contact info show up on supplier orders, bills of lading, and reports.
**Depends on:** [Feature 1 — User Authorization](feature-1-user-authorization.md)

---

## User Stories

### US-2.1: View company profile

**As a** manager
**I want to** see the company’s name, address, and contact details
**So that** I can check what will print on supplier orders and bills of lading

**Priority:** P1
**Independent test:** Call `getCompany` after bootstrap; the seeded company profile comes back
**Acceptance scenarios:** see ### US-2.1 under Acceptance Criteria

### US-2.2: Edit company profile

**As a** manager
**I want to** edit the company profile
**So that** documents and emails always show the current company information

**Priority:** P1
**Independent test:** Call `updateCompany` with valid data; `getCompany` returns the new values
**Acceptance scenarios:** see ### US-2.2 under Acceptance Criteria

---



## Requirements



### Functional Requirements

- **FR-001**: The system MUST have exactly **one** company record. It is created at bootstrap along with the first manager (Feature 1 FR-011).
- **FR-002**: Company MUST have: name, street address, city, state, postal code, phone, and email. All of these are required.
- **FR-003**: Company MAY also have a fax number and a logo reference.
- **FR-004**: Managers MUST be able to edit every company field. Other roles MUST only be able to see the company name (for headers).
- **FR-005**: The system MUST check that the email looks like an email, and that the phone has 10 digits (ignore things like dashes and parentheses).
- **FR-006**: The system MUST NOT let anyone add a second company or delete the company.
- **FR-007**: The company email MUST be used as the reply-to address on emailed supplier orders ([Feature 11](feature-11-automatic-reorder.md)).
- **FR-008**: The company name and address MUST show up on supplier orders ([Feature 10](feature-10-supplier-order-maintenance.md)) and bills of lading ([Feature 15](feature-15-ship-customer-orders.md)).

---



## Assumptions

- Eagle Wholesale is one legal company. Multiple warehouses belong to it.

---



## Edge Cases

- Required field left blank → `ValidationError`, nothing is saved
- Invalid email or phone → `ValidationError`
- Trying to create or delete a company → `NotAuthorizedError` / the action is not offered
- A non-manager calls `updateCompany` → `NotAuthorizedError`

---



## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: The company profile is there right after bootstrap
- **SC-003**: A saved change shows up on the next supplier order or bill of lading that gets generated

---



## Data Ownership & Isolation

- The company is global. Only managers can change it.

---



## Key Entities

- **Company**: the wholesaler itself; it owns all the warehouses

---



## Acceptance Criteria (Gherkin)



### US-2.1 — View company profile



#### Scenario: Manager views the company profile

- **Given** the system has been bootstrapped with company "Eagle Wholesale"
- **When** a manager opens Company Profile
- **Then** the company name, address, phone, and email are shown



### US-2.2 — Edit company profile



#### Scenario: Manager saves a valid company change

- **Given** a manager is on Company Profile
- **When** the manager changes the phone to "(405) 555-0100" and saves
- **Then** the company is saved
- **And** the profile shows phone "(405) 555-0100"



#### Scenario: Company is not saved when a required field is blank

- **Given** a manager is on Company Profile
- **When** the manager clears the company name and saves
- **Then** the system shows "Name is required"
- **And** the stored company is unchanged



#### Scenario: Company is not saved with an invalid email

- **Given** a manager is on Company Profile
- **When** the manager enters email "eagle-at-mail" and saves
- **Then** the system returns `ValidationError`
- **And** the stored company is unchanged



#### Scenario: Office user cannot edit the company

- **Given** an employee with role "OFFICE" is logged in
- **When** the employee calls the update company operation
- **Then** the system returns `NotAuthorizedError`

---

