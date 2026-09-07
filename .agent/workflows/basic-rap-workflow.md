# Basic ABAP RAP Fiori Elements Agentic Workflow

**Purpose:** Your job is follow and run the below steps one by one to get the job done.  


---

## Workflow Overview

```
Phase 1: Clarify     → Gather requirements
Phase 2: Generate    → Use ADT MCP to create RAP objects
Phase 3: Activate    → Activate all generated objects
Phase 4: Test        → Basic smoke test
Phase 5: Done        → Completion
```

**User Involvement:** Minimal (approvals at each phase)

---

## Phase 1: Clarify Requirements

**Goal:** Understand what app to build  
**Input Required:** 

Ask User to provide the business requirement. **Do not skip this step** 

**Agent Action:**
- Check for .agent/requirements/requirements-clarification.md for the reference template. 


**Success Criteria:** All fields answered, values validated (non-empty)

---

## Phase 2: Generate RAP Objects

**Goal:** Use ADT MCP Server to generate complete RAP layer  
**Scope:** Database table → CDS → Behavior → Service → Binding
**Success Criteria:** all the rap objects generated, no errors reported

---

## Phase 3: Activate Objects

**Goal:** Activate all generated RAP objects in the system

### Step 3.2: ATC Quality Gate (Optional)

**If quality gate required:**
```
abap_atc_run
├─ Input: Generated object list
├─ Output: Priority 1/2/3 findings
└─ Rule: P1 = MUST FIX, P2 = SHOULD FIX, P3 = NICE TO FIX
```

**Success Criteria:** 
- ✅ All  objects activated
- ✅ Zero P1 findings
- ✅ P2 findings reviewed (if any)

---

## Phase 4: Test (Smoke Test)

**Goal:** Verify app loads and responds to basic UI interaction

### Step 4.1: Publish Service Binding (Manual)

**Action:** In ADT, right-click service binding → Publish  
**Result:** Get public OData service URL

### Step 4.2: Smoke Test with Playwright

**Goal:** Test the applicating using playwrite MCP server

- Refer to `.agent/tests/test-rules.md`

---

## Phase 5: Done

**Completion Summary:**

```
✓ Phase 1: Requirements clarified
✓ Phase 2: All RAP objects generated
✓ Phase 3: All objects activated (ATC passed)
✓ Phase 4: Smoke test passed

🎉 RAP Fiori Elements App Ready

Next Steps:
1. Add custom business logic (actions, validations)
2. Customize UI (metadata extensions)
3. Deploy to production
4. Create end-to-end test scenarios
```


---

## Error Recovery

### If Generation Fails:
```
1. Check error message
2. Validate schema JSON syntax
3. Verify system connectivity (abap_list_destinations)
4. Retry with corrected schema
5. Max 3 attempts → Manual intervention required
```

### If Activation Fails:
```
1. Check individual object errors
2. Check for naming conflicts
3. Verify package exists
4. Check transport assignment
5. Retry activation
```

### If Test Fails:
```
1. Check TEST_URL is accessible
2. Verify credentials in .env
3. Check service binding is published
4. Review Playwright screenshots for UI state
5. Check browser console errors
```

---

## Commands for Agent

**Invoke workflow:**
```
@abap-rap-developer run workflow basic

@abap-rap-developer start phase 1

@abap-rap-developer show state

@abap-rap-developer retry phase 2
```

**Check progress:**
```
@abap-rap-developer list artifacts

@abap-rap-developer verify activation

@abap-rap-developer run test
```

---

## Configuration Files


**Test Environment:** `.agent/.env`
```bash
TEST_URL=https://...abap-web.eu10.hana.ondemand.com/.../flp.html
TEST_USER=your_user@techrig
TEST_PASSWORD=your_password
TEST_IDP=techrig
```


---


## Next: Extended Workflows

After completing basic workflow:
1. **Add Actions Workflow** - Approve, Reject operations
2. **Add Booking Child Workflow** - 1:N relationship (Travel → Booking)
3. **Validation Workflow** - Custom business rules
4. **Production Deployment Workflow** - Transport to prod system

---

**Created:** 2026-09-06  
**Status:** Ready for execution  
**Target System:** SAP BTP Cloud (abap_cloud)  
**Framework:** Managed RAP + Draft + OData V4 UI
