# Case Study: JWT Misconfiguration — Algorithm Confusion (none / RS256 → HS256)

**OWASP Category:** API Security Top 10 — API2:2023 Broken Authentication
**Severity:** Critical
**CWE:** CWE-347 (Improper Verification of Cryptographic Signature)

---

## Summary

A web application using RS256-signed JWTs for authentication was vulnerable
to two distinct JWT attacks: accepting tokens signed with the `none` algorithm
(no signature), and accepting RS256 tokens re-signed with HS256 using the
public key as the HMAC secret — allowing full token forgery.

---

## Environment (redacted lab reproduction)

Application using `jsonwebtoken` (Node.js) with a public/private key pair for RS256.
Authentication endpoint: `POST /api/auth/login` → returns JWT.
Protected endpoint: `GET /api/admin/users` — admin only.

---

## Root cause

Two separate misconfigurations:

**1. `none` algorithm accepted:**
The JWT library was configured without explicitly rejecting the `none` algorithm.
An attacker could strip the signature and change `alg` to `none`.

**2. RS256 → HS256 confusion:**
The server's public key was discoverable (JWKS endpoint, `/api/.well-known/jwks.json`).
When the attacker re-signed the token using HS256 with the public key as the secret,
the library — if misconfigured — treated the public key as a symmetric secret and
accepted the forged token.

---

## Reproduction (lab/authorized environment only)

**Attack 1 — none algorithm:**
1. Obtain a valid JWT: `eyJhbGciOiJSUzI1NiJ9.<payload>.<signature>`
2. Decode payload, change `role` to `admin`
3. Re-encode with `alg: none`, remove signature
4. Send forged token: application accepts it

**Attack 2 — Algorithm confusion:**
1. Retrieve public key from JWKS endpoint
2. Re-sign modified payload with HS256 using the PEM public key as secret
3. Send forged token — vulnerable libraries accept it as valid HS256

---

## Impact

- Complete authentication bypass
- Privilege escalation to any role (admin, superuser)
- Full data access and account takeover

---

## Fix

```javascript
// Explicitly allowlist accepted algorithms — NEVER allow 'none'
const decoded = jwt.verify(token, publicKey, { algorithms: ['RS256'] });
```

- Disable JWKS public key exposure if not required by federation
- Pin the expected algorithm at verification time
- Use a well-maintained, audited JWT library and keep it updated

---

## Verification

1. Send a token with `alg: none` → expect `401 Unauthorized`
2. Send a HS256 token signed with the public key → expect `401 Unauthorized`
3. Send a valid RS256 token → expect `200 OK`
4. Check library version against known CVEs for JWT handling bugs

---

## Key takeaway

**Never trust the `alg` field in the token header.**
Always enforce the expected algorithm server-side, independent of what the
token claims to use.
