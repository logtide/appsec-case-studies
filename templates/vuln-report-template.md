# Vulnerability Report Template (Lean)

> Defensive use only. Do not include secrets, private URLs, customer data, or real credentials.

## 1) Summary
**Title:**  
**Category:** (AuthZ / AuthN / IDOR / XSS / SSRF / etc.)  
**Severity:** (Low/Med/High/Critical) + rationale  
**Status:** (Draft / Submitted / Fixed / Verified)

## 2) Impact (1–3 bullets)
- What an attacker could achieve
- What data/actions are exposed
- Who is affected (users, admins, orgs)

## 3) Affected component
- Product / service:
- Endpoint / feature area:
- Environments: (prod/stage) — **keep generic**
- Roles required: (unauth / user / admin)

## 4) Preconditions
- Required permissions (if any)
- Required feature flags / settings
- Any assumptions (e.g., “object IDs are guessable”)

## 5) Evidence
Attach or describe:
- Relevant logs / screenshots (redacted)
- Request/response snippets (redacted, non-operational)
- Timestamps (UTC)

## 6) Reproduction (authorized testing only)
> Keep steps minimal and non-exploitative. Use placeholders.

1. Authenticate as **User A**.
2. Perform **Action X** on **Object A**.
3. Attempt the same action on **Object B** that belongs to **User B**.
4. Observe whether the server enforces ownership / policy.

**Expected:** Server returns 403/404 or equivalent.  
**Actual:** Server returns data / performs action (describe).

## 7) Root cause (likely)
- Missing server-side authorization check on object access
- Over-trusting client-supplied identifiers
- Confused deputy / role scoping issue
- Incomplete policy checks across microservices

## 8) Recommended fix
- Enforce authorization on every object access (deny-by-default)
- Centralize policy checks (middleware/service layer)
- Scope queries by tenant/user (row-level authorization)
- Prefer opaque IDs (helps, but **not a substitute** for authz)
- Add audit logs for sensitive object access

## 9) Verification plan (how to prove it’s fixed)
- Unit tests: “User A cannot access User B’s object”
- Integration tests on key endpoints
- Regression checks for list/export endpoints
- Monitor logs for denied access attempts post-fix

## 10) Redaction checklist
- [ ] No secrets/tokens
- [ ] No real user identifiers
- [ ] No internal hostnames
- [ ] No customer/org data
- [ ] Generic endpoints / placeholders only
