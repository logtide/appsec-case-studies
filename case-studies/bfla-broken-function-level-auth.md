# Case Study: Broken Function Level Authorization (BFLA)

**OWASP Category:** API Security Top 10 — API5:2023 Broken Function Level Authorization
**Severity:** High
**CWE:** CWE-285 (Improper Authorization)

---

## Summary

An API endpoint intended only for admin users was accessible to regular authenticated
users by directly calling it — no privilege escalation required. The application
controlled access via the UI (hiding the button) but applied no server-side
authorization check on the underlying API route.

---

## Environment (redacted lab reproduction)

Generic REST API application. Roles: `user`, `admin`.
The endpoint `DELETE /api/v1/users/{id}` was intended for admin use only.

---

## Root cause

The application relied on **UI-level access control only**. The frontend hid
the delete button for non-admin users, but the backend route handler checked
only that the request was authenticated (valid JWT), not that the caller held
the `admin` role.

```
// Vulnerable pseudo-code
app.delete('/api/v1/users/:id', authenticateToken, (req, res) => {
    // No role check here — only authentication
    db.deleteUser(req.params.id);
    res.json({ deleted: true });
});
```

---

## Reproduction (lab/authorized environment only)

1. Log in as a regular `user` and capture a valid JWT
2. Identify admin-only endpoints from API docs, JS source, or error messages
3. Send the request directly with the user's JWT:
   ```
   DELETE /api/v1/users/42
   Authorization: Bearer <user_jwt>
   ```
4. Observe: `200 OK { "deleted": true }` — user deleted without admin rights

---

## Impact

- Any authenticated user could delete, modify, or read other users' data
- Horizontal privilege escalation (user → admin-level actions)
- Data integrity and availability risks
- Compliance implications (GDPR — unauthorised data deletion)

---

## Fix

Apply **server-side role checks** on every sensitive endpoint. Never rely on
UI hiding alone:

```javascript
// Fixed pseudo-code
app.delete('/api/v1/users/:id', authenticateToken, requireRole('admin'), (req, res) => {
    db.deleteUser(req.params.id);
    res.json({ deleted: true });
});

function requireRole(role) {
    return (req, res, next) => {
        if (req.user.role !== role) return res.status(403).json({ error: 'Forbidden' });
        next();
    };
}
```

---

## Verification

1. As `user`: attempt `DELETE /api/v1/users/{id}` → expect `403 Forbidden`
2. As `admin`: same request → expect `200 OK`
3. Confirm the check is applied to all HTTP methods (GET, PUT, PATCH, DELETE)
4. Test with a manipulated JWT (downgraded role claim) — server must reject

---

## Key takeaway

**The server must enforce authorization independently of the client.**
UI-level controls are a UX feature, not a security control.
