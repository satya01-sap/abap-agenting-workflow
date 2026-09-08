# Playwright Testing Rules for SAP RAP Fiori Elements (v2)

## Explicit Trigger

When the main agent reaches Step 5, or when told `run test` / `trigger test` / `@abap-developer-agent run test`:

1. `TEST_URL` is already loaded from `.agent/.env` by the Playwright MCP — do NOT read `.env`, do NOT read `state.md` for the URL.
2. Call Playwright **MCP tools directly** in the sequence below — do NOT write a `.spec.ts` file, do NOT run `npx playwright test`.
3. Execute one tool call at a time — do not batch.
4. On failure: capture exact error + failing selector → append to `.agent/state.md` error log → stop. Do not retry.

---

## Tool Call Sequence

### 1. Navigate
```
browser_navigate(url: TEST_URL)
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

### 3. List Report loads
```
browser_wait_for(selector: '[aria-valuetext="Busy"]', state: hidden)
browser_snapshot()
```

### 4. Create record
```
browser_click(element: Create button)
browser_wait_for(selector: '[aria-valuetext="Busy"]', state: hidden)
browser_snapshot()

# Fill fields — scope all lookups to the dialog, not the page
browser_fill(element: TravelID in dialog,      value: 1)
browser_fill(element: AgencyID in dialog,      value: 900)
browser_fill(element: CustomerID in dialog,    value: 400)
browser_fill(element: BeginDate in dialog,     value: Sep 6, 2026)
browser_fill(element: EndDate in dialog,       value: Sep 16, 2026)
browser_fill(element: BookingFee in dialog,    value: 100)
browser_fill(element: TotalPrice in dialog,    value: 101)
browser_fill(element: CurrencyCode in dialog,  value: USD)
browser_fill(element: Description in dialog,   value: Test by playwright)
browser_fill(element: OverallStatus in dialog, value: O)
browser_snapshot()

browser_click(element: Save button)
browser_wait_for(selector: '[aria-valuetext="Busy"]', state: hidden)
browser_snapshot()
```

### 5. Verify saved record in List Report
```
browser_click(element: Go button)
browser_wait_for(selector: '[aria-valuetext="Busy"]', state: hidden)
browser_snapshot()
# Assert: at least one row visible in the table
```

---

## Fiori Elements Rules (non-negotiable)

**Wait for busy indicator before every interaction.**
SAPUI5 overlays a busy indicator that intercepts clicks and causes 30s timeouts.
```
browser_wait_for(selector: '[aria-valuetext="Busy"]', state: hidden)
```

**Scope every field lookup to the dialog.**
The List Report filter bar and Create dialog share field labels. An unscoped selector fills the filter bar — the test passes while creating nothing.

**Draft apps need two steps.** Create opens a draft; record exists only after Save. Assert on the saved record in the List Report, not on the dialog closing.

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
| Description   | Test by playwright |
| OverallStatus | O                  |
| BookingFee    | 100                |
| TotalPrice    | 101                |
| BeginDate     | Sep 6, 2026        |
| EndDate       | Sep 16, 2026       |
| CurrencyCode  | USD                |
