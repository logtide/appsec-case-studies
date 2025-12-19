# BOLA / IDOR — Broken Object Level Authorization

**What this is:** the server allows access to an object (record/file/order/profile) without verifying the requester is authorized for *that specific object*.

This often shows up as **IDOR** when the client can reference objects by an ID and the backend doesn’t enforce ownership/tenant policy.

---

## Why it matters (impact)
Typical outcomes:
- Read other users’ data (PII, invoices, messages, internal notes)
- Modify or delete other users’ objects (orders, addresses, settings)
- Cross-tenant data exposure in multi-tenant SaaS (worst case)

---

## How to recognize the pattern (safe mental model)
If the system does:
- “Take `object_id` from the request”
- “Fetch object by ID”
- “Return it / update it”
…but does **not** also enforce:
- “Is this requester allowed to access this object?”

…it’s a BOLA risk.

---

## Minimal validation plan (authorized / lab-friendly)
Goal: prove the backend enforces object-level policy.

1) Create two distinct users: **User A** and **User B** (same tenant unless testing tenant isolation).  
2) Create one object per user: **Object A** and **Object B**.  
3) As **User A**, attempt to access/modify **Object B** via the normal product flow (no tricks, just object switching).  
4) Record results.

**Expected behavior**
- Server returns **403 Forbidden** (or **404 Not Found**) consistently for unauthorized objects.

**Risky behavior**
- Server returns data, accepts updates, or returns “success” for objects not owned by the requester.

> Notes: Using opaque IDs can reduce guessing, but **does not** replace authorization.

---

## Common root causes
- Authorization checks exist for “feature access” but not per-object access
- Backend trusts the frontend to only send “allowed” IDs
- Data access layer queries don’t scope by user/tenant
- Microservice boundaries: one service validates auth, another fetches objects without re-checking policy
- “Admin” or “support” roles accidentally over-broaden access

---

## Fix patterns that scale
### 1) Deny by default + enforce policy at the server
- Centralize authorization in middleware/service layer
- Require a policy decision for **every** object read/write

### 2) Scope data access by principal and tenant
Instead of:
- fetch by `object_id`

Do:
- fetch by (`tenant_id`, `object_id`) and validate ownership/relationship to requester

### 3) Make sensitive actions harder to abuse
- Re-authentication / step-up MFA for critical changes
- Strong audit logs for object reads/exports/role changes

### 4) Add tests so it can’t regress
Minimum test cases:
- User A can access Object A
- User A cannot access Object B
- Admin can access both (only if intended)
- Cross-tenant isolation (if multi-tenant)

---

## Detection & monitoring ideas (defensive)
- Alert on repeated access denials across many object IDs (enumeration signal)
- Alert when a user accesses many distinct objects quickly (export/scraping signal)
- Log object access with: principal, tenant, object type, object ID (hashed if needed), decision (allow/deny)


