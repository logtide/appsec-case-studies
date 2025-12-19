# appsec-case-studies

A small, practical AppSec portfolio: short case studies + a reusable vuln report template.

**Goal:** show how I think (risk → root cause → fix → verification) without publishing exploit write-ups, targets, or sensitive details.

## What’s inside

- `case-studies/`
  - Short, generic write-ups of common vulnerability classes (no real targets, no secrets).
- `templates/`
  - A clean vulnerability report template you can reuse for internal reports or bug bounty submissions.

## Principles

- **Defensive & educational only.**
- **No step-by-step exploitation** for real systems.
- **No target specifics** (domains, endpoints, tokens, org data).
- Focus on **root cause**, **impact**, **remediation**, and **verification**.

## How to use (fast)

1. Pick a case study in `case-studies/` and read the “Fix pattern” + “Verification” sections.
2. Use `templates/vuln-report-template.md` to write your own report for practice.
3. If you add new content: keep it short, generic, and safe.

## Suggested next additions (optional)

- `case-studies/authz-bypass.md`
- `case-studies/ssrf.md`
- `case-studies/stored-xss.md`
- `case-studies/csrf.md`

> If you’re reviewing this repo as a recruiter: open one case study and scan the “Fix pattern” and “Verification” sections — that’s the signal.
