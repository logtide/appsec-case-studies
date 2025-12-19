# appsec-case-studies

Lean, recruiter-friendly AppSec case studies + a repeatable vuln report template.

## What this repo is
A small set of writeups that show how I think about:
- threat modeling and impact
- root cause analysis
- safe, responsible validation
- pragmatic remediation + verification

## Safety / scope
- Defensive / educational content only.
- No “how to break in” walkthroughs.
- No real targets, no customer/org data, no secrets.
- Examples are generic and meant for authorized testing or lab environments.

## Structure
- `case-studies/` — short case studies (one per file)
- `templates/` — copy/paste templates I use for writing clean reports

## Case studies
- `case-studies/bola-idor.md` — Broken Object Level Authorization (BOLA/IDOR)

## How to use
If you’re reviewing:
1. Open a case study.
2. Skim **Impact**, **Root causes**, **Fix**, **Verification**.

If you’re writing your own:
1. Copy `templates/vuln-report-template.md`
2. Fill it with a redacted, authorized example.
