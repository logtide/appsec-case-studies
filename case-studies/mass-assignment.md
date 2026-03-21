# Case Study: Mass Assignment — Privilege Escalation via API Body Parameters

**OWASP Category:** API Security Top 10 — API6:2023 Unrestricted Access to Sensitive Business Flows
**Severity:** High
**CWE:** CWE-915 (Improperly Controlled Modification of Dynamically-Determined Object Attributes)

---

## Summary

An API endpoint that allowed users to update their own profile accepted and
processed additional undocumented fields including `role` and `isAdmin`.
By including these fields in the request body, a regular user could elevate
their own privileges to administrator level.

---

## Environment (redacted lab reproduction)

REST API. User profile update endpoint: `PUT /api/v1/users/me`
Intended body: `{ "name": "...", "email": "..." }`
Internal user model also contains: `role`, `isAdmin`, `creditBalance`

---

## Root cause

The API controller passed the entire request body directly into the ORM/model
update function without filtering or allowlisting permitted fields:

```javascript
// Vulnerable pseudo-code
app.put('/api/v1/users/me', authenticate, async (req, res) => {
    await User.update(req.user.id, req.body);  // entire body passed in
    res.json({ updated: true });
});
```

---

## Reproduction (lab/authorized environment only)

1. Log in as regular user, capture valid session token
2. Send profile update with extra fields injected:
   ```json
   PUT /api/v1/users/me
   {
     "name": "Elias",
     "email": "elias@example.com",
     "role": "admin",
     "isAdmin": true,
     "creditBalance": 99999
   }
   ```
3. Observe: `200 OK` — role and isAdmin updated server-side
4. Subsequent requests now reflect admin privileges

---

## Impact

- Horizontal and vertical privilege escalation
- Financial manipulation (e.g., credit balance inflation)
- Regulatory exposure (unauthorised role changes)
- Can be chained with BOLA/IDOR for account takeover at scale

---

## Fix

Use an explicit **allowlist** of fields permitted in each endpoint:

```javascript
// Fixed pseudo-code
const ALLOWED_UPDATE_FIELDS = ['name', 'email', 'bio'];

app.put('/api/v1/users/me', authenticate, async (req, res) => {
    const safeBody = pick(req.body, ALLOWED_UPDATE_FIELDS);  // whitelist only
    await User.update(req.user.id, safeBody);
    res.json({ updated: true });
});
```

- Use DTOs (Data Transfer Objects) or schema validation (Joi, Zod, Pydantic)
- Never pass raw request bodies into model update methods
- Apply the same principle to query parameters and form data

---

## Verification

1. Include `"role": "admin"` in update body → check role is unchanged server-side
2. Include `"isAdmin": true` → verify flag not set
3. Include `"creditBalance": 99999` → verify balance unchanged
4. Confirm only allowlisted fields are reflected in the response and DB

---

## Key takeaway

**Never trust client-supplied field names. Allowlist what can be modified —
never blocklist what cannot.**
