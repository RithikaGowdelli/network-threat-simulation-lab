# Commit Message Guide

Use these as templates, not verbatim text — write commits as you actually complete work, not all at once at the end. A commit history that's all one day with generic messages reads as fabricated.

## Format

```
<type>: <short summary, imperative mood, under 60 chars>

<optional body: what and why, not how>
```

Types: `feat`, `fix`, `docs`, `refactor`, `data`

## Examples

```
feat: add Nmap scan procedure for Scenario 1

docs: write detection tuning report for SSH brute force rule

data: add sanitized auth.log excerpt for Scenario 2

fix: correct SPL query field extraction for source IP

docs: add MITRE ATT&CK mapping to Scenario 3 README

feat: add Wireshark filters for HTTP enumeration detection

docs: write final incident investigation report

refactor: reorganize scenario folders for consistency

docs: complete lessons learned section
```

## What to Avoid

- "update files"
- "final version"
- "changes"
- Committing all four scenarios and every doc in a single commit — this is a red flag to anyone reviewing your repo, since it signals the work wasn't actually done incrementally
