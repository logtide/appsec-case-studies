# appsec-case-studies

Short, defensive AppSec write-ups focused on *root cause → impact → fixes → verification*.

**Scope**
- No real targets, no customer data, no exploit chains.
- Examples are **sanitized** and framed for **authorized testing only**.

## Contents

### Case studies
- [BOLA / IDOR (Broken Object Level Authorization)](case-studies/bola-idor.md)

### Templates
- [Vulnerability report template](templates/vuln-report-template.md)

## Why this exists (recruiter version)
I use these to demonstrate how I think:
- identify the security control that failed (usually **authorization**),
- articulate business impact,
- propose practical mitigations,
- define verification steps and regression tests.
