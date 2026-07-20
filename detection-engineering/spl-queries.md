# Detection Query Documentation

This file consolidates every SPL query used across the four scenarios, with plain-English explanation of what each line does. Do not publish this file until every query below has been run against real data and confirmed working — a broken or untested query in a portfolio is worse than no query.

## Scenario 2 — SSH Brute Force Detection

**Purpose:** Flag any source IP with more than 5 failed SSH authentication attempts within a 5-minute window.

```
[PLACEHOLDER — paste your real, tested query]
```

**Line-by-line explanation:**
[PLACEHOLDER — once you paste the real query, I'll write the explanation against your actual syntax]

**Tuning notes:**
[PLACEHOLDER — e.g. why 5 attempts / 5 minutes was chosen as the threshold, what false-positive rate you observed, what you'd change for a real production environment]

## Scenario 1 — Port Scan Detection

```
[PLACEHOLDER — pending confirmation of log source]
```

## Scenario 3 — HTTP Path Enumeration Detection

```
[PLACEHOLDER — pending confirmation of log source and field extraction]
```

## Scenario 4 — Anomalous Outbound Connection Detection

```
[PLACEHOLDER — pending confirmation of log source]
```

## General Notes on Field Extraction

[PLACEHOLDER — document here how you're extracting fields, e.g. via `rex`, via a configured sourcetype, via Splunk's automatic field extraction. Being explicit about this shows a real understanding of Splunk internals rather than copy-pasted queries.]
