# Requirements Clarification

Fill in each `[Answer]` before approving. Leave blank if unsure — the agent will ask.

---

## Q1 — Target ABAP System
Which system/destination should objects be generated in?

[Answer]: abap_cloud

---

## Q2 — Package
A) Use existing package (provide name):
B) Create new package (provide name + description):
C) Save locally in `$TMP` (no transport needed)

[Answer]: A) Use existing package: `ZCO_TRAVEL_001`

---

## Q3 — Transport Request
A) Use existing TR (provide number, e.g. `DEVK900123`):
B) Create new TR (provide short description):
C) Not needed — using `$TMP`

[Answer]: C) Not needed — local package, no transport

---

## Q4 — Group Suffix (`###`)
What is your group/participant suffix? (e.g. `001`, `042`, `XYZ`)

[Answer]: 001

---

## Q5 — Business Entities & Hierarchy
Describe each entity, its key fields, and parent-child relationships.

Example:
- `Travel` (root): travel_id, customer_id, begin_date, end_date, status
- `Booking` (child of Travel): booking_id, flight_date, carrier_id, price

[Answer]:
- `Travel` (root), based on `/DMO/TRAVEL_DATA`
- `Booking` (child of `Travel`), based on `/DMO/BOOKING_DATA`

---

## Q6 — Data Source
A) Use existing database tables (provide names):
B) Use DMO reference data (e.g. `/DMO/TRAVEL_DATA`):
C) Generate new tables from entity definitions above

[Answer]: B) Use DMO reference data: `/DMO/TRAVEL_DATA`, `/DMO/BOOKING_DATA`

---

## Q7 — RAP Pattern
A) Managed (recommended — framework handles CRUD)
B) Unmanaged (only if custom persistence logic is required)

[Answer]: A) Managed

---

## Q8 — Draft Handling
A) With draft (recommended for transactional apps — enables save/discard)
B) Without draft

[Answer]: A) With draft

---

## Q9 — Additional Features (select all that apply)
- [x] Action buttons (e.g. Approve, Reject)
- [x] Determinations (e.g. set status on save)
- [x] Validations (e.g. mandatory field checks)
- [x] Value helps (fixed domain values or CDS-based)
- [x] Authorization checks

[Answer]: All selected

---

## Q10 — Service Binding Type
A) `OData V4 - UI` (Fiori Elements, recommended)
B) `OData V2 - UI`
C) `Web API` (for external consumption)

[Answer]: A) OData V4 - UI