# Vulnerability Report Template (Safe / Reusable)

> Use this template for internal write-ups or responsible disclosures.
> Keep evidence minimal, redact secrets, and never include real credentials.

---

## Title
**[Vuln Class]** – short description (e.g., “BOLA/IDOR via missing object-level authorization on resource read”)

## Summary (1–3 sentences)
What is vulnerable, at a high level, and why it matters.

## Severity
- **Proposed severity:** Low / Medium / High / Critical
- **Reasoning:** (impact + likelihood + reach)
- **Scoring (optional):** CVSS v3.1 vector (if you use it)

## Affected Component(s)
- Service / feature name:
- Environment: prod / staging / test
- Entry points (generic): API route type, UI page, workflow step  
  *(Avoid posting full real URLs publicly.)*

## Preconditions
What access is required?
- Anonymous / Authenticated user / Admin
- Specific role required? Y/N
- Any feature flags / settings?

## Root Cause
Describe the engineering cause, not the symptom.
Examples:
- Missing **object-level authorization** checks
- AuthZ check performed on the wrong identifier (user-controlled)
- Trusting client-supplied roles/claims
- Insecure direct reference to internal IDs

## Security Impact
What a malicious actor could do **in plain language**:
- Read other users’ data?
- Modify/delete resources?
- Escalate privileges?
- Pivot to other systems?

Also note **data types** potentially exposed (PII, tokens, billing, etc.).

## Evidence (Redacted)
Attach minimal proof:
- Screenshots with redactions
- Sanitized request/response snippets (placeholders)
- Logs (sanitized)

Example (generic):
```http
GET /api/resource/{RESOURCE_ID}  HTTP/1.1
Authorization: Bearer [REDACTED]
