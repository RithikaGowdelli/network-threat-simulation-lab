# Detection Tuning Report

## Purpose

This report documents how each detection rule in this lab was tuned: what the initial threshold was, why it changed, and what tradeoffs were made between catching real activity and generating noise.

## Scenario 2 — SSH Brute Force

| Iteration | Threshold | Result | Change made |
|---|---|---|---|
| 1 | [PLACEHOLDER] | [PLACEHOLDER — e.g. too sensitive, flagged normal retry behavior] | [PLACEHOLDER] |
| 2 | 5 failures / 5-minute bin | [PLACEHOLDER — final result] | Final threshold adopted |

**Rationale for final threshold:**
[PLACEHOLDER — explain why >5 in 5 minutes was chosen. What's the baseline for a legitimate user mistyping a password? Where would you set this differently for a service account vs. a human user?]

## Scenario 1 / 3 / 4

[PLACEHOLDER — fill in once these detections exist. If you didn't iterate on a threshold for a scenario, say so directly rather than padding this section.]

## General Lessons on Tuning

[PLACEHOLDER — 3 to 5 sentences on what you learned about the tradeoff between detection sensitivity and alert fatigue. This section is what actually demonstrates SOC analyst judgment to a hiring manager, so don't leave it thin or generic.]
