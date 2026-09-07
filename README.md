# Agentic ABAP RAP Development Setup

GitHub Copilot + ADT MCP Server workflow for building SAP Fiori Elements apps on ABAP RAP — from requirements to smoke test with minimal manual steps.

---

## Prerequisites

| Tool | Purpose |
|------|---------|
| [VS Code](https://code.visualstudio.com/) | IDE |
| [GitHub Copilot](https://claude.ai/code) | AI agent (CLI or VS Code extension) |
| SAP BTP ABAP environment | Target system (cloud or on-premise) |
| ADT MCP Server | Connects Claude to your ABAP system |

---

## 1 — Get the `.agent` folder

**Option A — Download ZIP**
1. Open the repo on GitHub → **Code** → **Download ZIP**
2. Extract and open the folder in VS Code


**Option B — Clone this repo**
```bash
git clone https://github.com/satya01-sap/ABAP.git
cd ABAP
```


---

## 2 — Open in VS Code

```bash
code ABAP.code-workspace
```

Or: **File → Open Workspace from File…** → select `ABAP.code-workspace`

The `.agent` folder will be visible in the Explorer sidebar.

---

## 3 — Install GitHub Copilot

Install the **GitHub Copilot** extension from the VS Code Marketplace.

     
---

Then configure your ABAP system connection in ADT (Eclipse or VS Code SAP plugin) — the MCP server reads destinations from there.

Verify the connection is visible to Claude:
```
abap_list_destinations
```

---

## 5 — Configure test credentials

Copy and fill in `.agent/.env`:
```bash
cp .agent/.env.example .agent/.env   # if example exists, else edit directly
```

```bash
# .agent/.env
TEST_URL=https://<your-system>.abap-web.<region>.hana.ondemand.com/.../flp.html
TEST_USER=your_user@domain
TEST_PASSWORD=your_password
TEST_IDP=default
```

> `.agent/.env` is gitignored — credentials stay local.

---

## 6 — Run the agentic workflow

Open GitHub Copilot in VS Code (`` Ctrl+` `` → `claude`) and say:

```
@abap-rap-developer run workflow basic
```

The agent follows `.agent/workflows/basic-rap-workflow.md` through five phases:

| Phase | What happens |
|-------|-------------|
| **1 — Clarify** | Agent reads `.agent/requirements/requirements-clarification.md` and asks you to fill in any blanks |
| **2 — Generate** | ADT MCP creates table → CDS views → behavior definition → service definition → service binding |
| **3 — Activate** | All objects activated; optional ATC quality gate (P1 = must fix) |
| **4 — Test** | Playwright smoke test against your Fiori app URL using credentials from `.agent/.env` |
| **5 — Done** | Summary printed; next-step suggestions given |

---

## Folder structure

```
.agent/
├── .env                          # Test credentials (local only, gitignored)
├── state.md                      # Agent writes app URL + error log here
├── requirements/
│   └── requirements-clarification.md   # Fill this before starting
├── tests/
│   └── test-rules.md             # Playwright rules for Fiori Elements
└── workflows/
    └── basic-rap-workflow.md     # Main agentic workflow definition
```

---

## Customising requirements

Edit `.agent/requirements/requirements-clarification.md` before running the workflow to set:
- Target ABAP system / destination
- Package and transport request
- Entity names, key fields, and parent-child hierarchy
- RAP pattern (Managed / Unmanaged)
- Draft handling, actions, validations, value helps

The agent reads this file automatically — no need to retype answers in chat.

---

## Troubleshooting

**ADT MCP not connecting**
- Check the ABAP system is reachable and ADT is configured in Eclipse/VS Code
- Run `abap_list_destinations` in Claude to verify

**Activation fails**
- Check for naming conflicts or missing package
- Verify transport assignment if not using `$TMP`

**Playwright test fails**
- Confirm `TEST_URL` in `.agent/.env` is the published Fiori app URL
- Make sure the service binding is published in ADT before running Phase 4

---

## Extending the workflow

After completing the basic workflow, add these:

- `.agent/workflows/actions-workflow.md` — Approve/Reject actions
- `.agent/workflows/booking-child-workflow.md` — 1:N Travel → Booking
- `.agent/workflows/validation-workflow.md` — Custom business rules
