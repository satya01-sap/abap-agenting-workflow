# Playwright Testing Rules

You are a Playwright test agent for a SAP Fiori Elements app built on ABAP RAP.

## Bootstrap

Before writing any test:
1. Read `app_url` from `.agent/state.md` — do not ask the user for it.
2. Read credentials from `.agent/.env`:
   ```
   TEST_USER=<email>
   TEST_PASSWORD=<password>
   TEST_IDP=<idp name, e.g. default | techrig>
   ```
   If `.env` is missing or incomplete → stop and tell the user to create it. Do not ask for credentials in chat.

## Test Scenario

1. **Login test** — always first. Navigates to `app_url`, selects IdP, enters credentials, asserts app shell loads.
2. **CRUD smoke tests** — Create, Read, Update, Delete one record per root entity.
3. **Scenario tests** — any business-specific flows from the requirements (e.g. approval workflow).

## Fiori Elements rules

These are non-negotiable — skipping them causes false failures.

**Wait for the busy indicator before every interaction.**
SAPUI5 overlays a busy indicator that intercepts clicks and causes 30s timeouts that look like missing elements.
```ts
await page.locator('[aria-valuetext="Busy"]').waitFor({ state: 'hidden' });
```

**Scope every field lookup to the dialog.**
The List Report filter bar and the Create dialog both contain inputs with matching labels. An unscoped selector fills the filter bar instead of the form — the test passes while creating nothing.
```ts
const dialog = page.getByRole('dialog');
await dialog.getByLabel('Booking Fee').fill('100');
```

**Prefer `getByLabel` scoped to the dialog over `aria-label` or index.**
If a field can't be found by label, fall back to the dialog's nth input — never to a page-wide index.

**Setting values via JS requires dispatching UI5's events.**
Only do this when Playwright's `fill()` fails. A raw `.value =` assignment does not update the UI5 model.
```ts
el.value = v;
['input', 'change', 'blur'].forEach(e =>
  el.dispatchEvent(new Event(e, { bubbles: true }))
);
```

**Draft apps need two steps.** Create opens a draft; the record only exists after Save. Assert on the saved record in the List Report, not on the dialog closing.

**Screenshot after every phase** (login, list loaded, dialog open, fields filled, saved). Failures are diagnosed from these.

## Execution rules

- Run steps one by one using the Playwright MCP tools — do not batch all steps into one call.
- On failure: capture the exact error message and failing selector. Write both to `.agent/state.md` error log before stopping.
- Do not retry a failing test yourself — the development loop handles retries.
- Save generated test files under `.agent/tests/` with name `<entity>-<scenario>.spec.ts`.
- A test is only "passing" when it exits with zero failures — not when it "seems to work".

## Test Date & format

TravelID : 1
AgencyID : 900
CustomerID : 400
Description : Test by playwrite,(Enter a text with a maximum of 10 characters) 
OverallStatus : one char only 
BookingFee : 100 USD
TotalFee : 101 USD
BeginDate: Sep 6, 2026
EndDate: Sep 16, 2026
CurrencyCode: USD

