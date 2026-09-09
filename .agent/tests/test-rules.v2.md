# Playwright Testing Rules for SAP RAP Fiori Elements

## Explicit Trigger

When the main agent reaches Step 5, or when told `run test` / `trigger test` / `@abap-developer-agent run test`:

1. Read the test credentials and application url from `.agent/.env` or `state.md` 
2. Call Playwright **MCP tools directly** in the sequence below — do NOT write a `.spec.ts` file, do NOT run `npx playwright test`.
3. Execute one tool call at a time — do not batch.
4. On failure: capture exact error + failing selector → append to `.agent/state.md` error log → stop. Do not retry.

---

## Tool Call Sequence

### 1. Navigate
```
browser_navigate
browser_snapshot()
```

### 2. Login
```
browser_click(element: IdP matching TEST_IDP)
browser_fill(element: username field, value: TEST_USER)
browser_fill(element: password field, value: TEST_PASSWORD)
browser_click(element: submit/login button)
browser_wait_for(selector: '[aria-valuetext="Busy"]', state: hidden)
browser_snapshot()
```

### 3. Confirm Home Screen
```
browser_wait_for(selector: '[aria-valuetext="Busy"]', state: hidden)
browser_snapshot()
# Assert: the home screen is the Fiori Elements list report with a visible Go button in the filter bar.
# Do not proceed until the list report home screen is visible.
```

### 4. Create New Travel Booking
```
browser_click(element: Create button)
browser_wait_for(selector: '[aria-valuetext="Busy"]', state: hidden)
browser_snapshot()

# The Create button opens a new Travel booking draft on the object page.
# This is not an edit flow. Fill values only on the object page fields, never the list report filter bar.
browser_fill(element: TravelID in dialog,      value: 1)
browser_fill(element: AgencyID in dialog,      value: 900)
browser_fill(element: CustomerID in dialog,    value: 400)
browser_fill(element: BeginDate in dialog,     value: Sep 6, 2026)
browser_fill(element: EndDate in dialog,       value: Sep 16, 2026)
browser_fill(element: BookingFee in dialog,    value: 100)
browser_fill(element: BookingFee currency field or paired currency value help, value: USD)
browser_fill(element: TotalPrice in dialog,    value: 101)
browser_fill(element: TotalPrice currency field or paired currency value help, value: USD)
browser_fill(element: CurrencyCode in dialog,  value: USD)
browser_fill(element: Description in dialog,   value: test1)
browser_fill(element: OverallStatus in dialog, value: 1)
browser_snapshot()

browser_click(element: Create button in footer)
browser_wait_for(selector: '[aria-valuetext="Busy"]', state: hidden)
browser_snapshot()
```

### 5. Return Home And Refresh
```
# Use the browser back action or navigate directly to the list report home screen if needed.
browser_wait_for(selector: '[aria-valuetext="Busy"]', state: hidden)
browser_snapshot()

browser_click(element: Go button on the home screen)
browser_wait_for(selector: '[aria-valuetext="Busy"]', state: hidden)
browser_snapshot()
# Assert: at least one row visible in the table, and the created Travel record is visible in the list report.
```

---

## Fiori Elements Rules (non-negotiable)

**Wait for busy indicator before every interaction.**
SAPUI5 overlays a busy indicator that intercepts clicks and causes 30s timeouts.
```
browser_wait_for(selector: '[aria-valuetext="Busy"]', state: hidden)
```

**Scope every field lookup to the dialog.**
The list report filter bar and the Travel object page share labels such as TravelID, AgencyID, CustomerID, BookingFee, TotalPrice, CurrencyCode, Description, and OverallStatus. Unscoped selectors can fill the filter bar or the wrong control.

**Use the footer Save action for this app.**
The Travel create flow opens a draft object page. Commit the draft with the footer Save button, then return to the list report home screen and press Go to refresh the table.

**Draft apps need two steps.** Create opens a draft; record exists only after footer Save. Assert on the saved record in the list report after returning home and pressing Go, not on the object page closing.

**Setting values via JS (last resort only).**
Only when `browser_fill` fails. A raw `.value =` assignment does not update the UI5 model — must dispatch events:
```
el.value = v;
['input', 'change', 'blur'].forEach(e =>
  el.dispatchEvent(new Event(e, { bubbles: true }))
);
```

**Screenshot after every phase** (login, list loaded, dialog open, fields filled, saved).

---

## Test Data

| Field         | Value              |
|---------------|--------------------|
| TravelID      | 1                  |
| AgencyID      | 900                |
| CustomerID    | 400                |
| Description   | Smoke Test         |
| OverallStatus | 1                  |
| BookingFee    | 100                |
| BookingFee currency | USD          |
| TotalPrice    | 101                |
| TotalPrice currency | USD          |
| BeginDate     | Sep 6, 2026        |
| EndDate       | Sep 16, 2026       |
| CurrencyCode  | USD                |
