# BOLA / IDOR (Broken Object Level Authorization)

**Category:** Access Control / Authorization  
**Also known as:** IDOR (Insecure Direct Object Reference), BOLA (OWASP API Top 10)  
**Goal of this note:** explain the *failure mode* and how to prevent it, not how to abuse real systems.

---

## 1) The pattern (what goes wrong)

A user is authenticated, but the application fails to verify they are **authorized** to access a specific object.

Common symptoms:
- Requests reference objects by predictable IDs (`orderId`, `documentId`, `userId`)
- The server checks “is logged in?” but not “does this user own/should see this object?”
- Authorization decisions happen in the UI, not on the server

---

## 2) Example scenario (sanitized)

Imagine an API that returns an order:

- The client requests an order by `orderId`
- The API returns the order if it exists
- **Bug:** the API does not verify the order belongs to the requester

**Root cause:** missing object-level authorization check on the server.

---

## 3) Why it matters (impact)

Typical impacts:
- **Data exposure** (PII, invoices, documents, internal notes)
- **Account takeover enablers** (reset tokens, session artifacts, profile data)
- **Privilege escalation** (view/modify other users’ resources)
- **Compliance issues** (GDPR/PCI depending on the data)

This class of bug is often **high severity** because it breaks trust boundaries.

---

## 4) How to fix it (the correct control)

### A. Enforce authorization at the object boundary
Authorization must happen **after identifying the requester** and **before returning/modifying the object**.

**Good rule of thumb**
> Every endpoint that accepts an object identifier must verify the requester’s permission for that specific object.

### B. Prefer “scoped queries” over “fetch then check”
Instead of:
- fetch order by `orderId`
- then check owner

Do:
- fetch order where `orderId = ? AND ownerId = currentUserId`
- if not found → return 404 (or 403 depending on policy)

### C. Centralize policy
Put authorization in:
- middleware / policy layer
- service methods
- or a policy engine (depending on stack)

Avoid scattered per-controller checks.

---

## 5) Verification (how you prove it’s fixed)

Minimum verification checklist:
- ✅ Authorized user can access their own object
- ✅ Unauthorized user cannot access someone else’s object
- ✅ “Not found” behavior does not leak existence (consider returning 404)
- ✅ Works for **read** and **write** operations
- ✅ Regression tests added at API and service layers

Suggested automated tests:
- unit test for policy function (owner/admin/role cases)
- integration test hitting endpoint with two users + two objects

---

## 6) Logging & detection (defensive monitoring ideas)

Even with prevention, monitor for signals of attempted abuse:

High-signal events:
- repeated 403/404 on object endpoints with changing IDs
- spikes in access to “view” endpoints across many object IDs
- access patterns inconsistent with normal UX flows

Log fields that help:
- `actor_user_id`, `actor_role`
- `target_object_type`, `target_object_id`
- `decision` (allow/deny), `reason`
- `source_ip`, `user_agent`, `request_id`

---

## 7) Quick reviewer checklist (what I look for)

- Do we ever authorize based on client-side state?
- Is there any endpoint that takes `userId`/`accountId` from the client?
- Are object IDs guessable or enumerable?
- Are we missing “tenantId” scoping in multi-tenant queries?
- Are batch endpoints (list/export/search) scoped correctly?

---

## 8) Takeaway principle

**AuthN (who you are) is not AuthZ (what you can access).**  
BOLA happens when “logged in” is treated as permission.
