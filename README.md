# appsec-case-studies

Lean, recruiter-friendly AppSec case studies + a repeatable vuln report template.
All examples use generic, redacted lab reproductions. No real targets, no customer data.

## What this repo is

A set of writeups showing how I think through:
- Threat modelling and business impact
- Root cause analysis
- Safe, responsible validation
- Pragmatic remediation and verification

## Safety / scope
Defensive and educational content only. No exploitation walkthroughs.
All examples are based on generic authorized lab environments.

---

## Case studies

| Vulnerability | OWASP Category | Severity | File |
|--------------|----------------|----------|------|
| BOLA / IDOR | API3:2023 | High | `case-studies/bola-idor.md` |
| Broken Function Level Authorization | API5:2023 | High | `case-studies/bfla-broken-function-level-auth.md` |
| JWT Algorithm Confusion | API2:2023 | Critical | `case-studies/jwt-misconfiguration.md` |
| Mass Assignment | API6:2023 | High | `case-studies/mass-assignment.md` |
| SSRF — Cloud Metadata | OWASP A10 | Critical | `case-studies/ssrf-internal-metadata.md` |

## Structure
```
case-studies/     # One case study per vulnerability
templates/        # Copy/paste vuln report template
```

## How to use

**If you're reviewing:** open a case study, skim Impact → Root Cause → Fix → Verification.

**If you're writing your own:** copy `templates/vuln-report-template.md`, fill with a redacted authorized example.

## Topics
`appsec` `owasp` `web-security` `burp-suite` `api-security` `vulnerability-research` `responsible-disclosure`
