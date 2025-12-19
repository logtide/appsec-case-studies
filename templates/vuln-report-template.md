# Vulnerability Report Template (AppSec)

> Use only for authorized testing (internal environments, your own apps, or programs with explicit permission).

## Title
[Short] [Vuln class] in [component] allows [impact]  
Example: `BOLA in Orders API allows unauthorized order access`

## Summary (1–3 sentences)
What the issue is, where it lives, and the outcome if exploited.

## Severity (and rationale)
- Proposed severity: [Low/Med/High/Critical]
- Rationale: data sensitivity, exploitability, affected users, required privileges

## Affected component(s)
- Service/API:
- Endpoint(s):
- Parameters/objects involved:
- Environments: [dev/stage/prod]
- Versions/commit:

## Preconditions
- Auth required? [yes/no]
- Role required? [user/admin/etc]
- Any special account state?

## Steps to reproduce (authorized environments only)
> Keep concise and deterministic.
1.
2.
3.

## Expected vs actual
- Expected:
- Actual:

## Impact
- What data/actions become possible?
- Which user groups are affected?
- Business/compliance risk (if relevant)

## Root cause (best guess)
Explain the missing/failed control (e.g., missing object-level authorization check).

## Evidence
- Request/response snippets (redacted)
- Screenshots (redacted)
- Logs / request IDs / timestamps (if available)

## Remediation guidance
Concrete fix suggestions:
- Authorization pattern to implement
- Scoping query recommendation
- Centralizing policy/middleware

## Verification / regression tests
What to test to ensure the fix sticks:
- positive test (authorized)
- negative test (unauthorized)
- edge cases (tenant boundaries, role changes, deleted objects)

## References
- OWASP: Broken Access Control
- OWASP API Top 10: BOLA
- Internal coding standards (if applicable)
