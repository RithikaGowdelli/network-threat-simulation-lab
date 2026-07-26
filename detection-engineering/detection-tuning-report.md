# Detection Tuning Report

## Purpose

This report documents how each detection rule in this lab was tuned: what the initial threshold was, why it changed, and what tradeoffs were made between catching real activity and generating noise.

## Scenario 2 — SSH Brute Force

| Iteration | Threshold | Result | Change made |
|---|---|---|---|
| 1 | 5 failures / 5-minute bin | Correctly flagged both attack windows (1,041 and 653 failures), well above threshold | Adopted as final threshold, no further iteration performed in this lab session |

**Rationale for final threshold:**
A threshold of more than 5 failures in a 5-minute window was chosen because it sits well above what a legitimate human user would generate by mistyping a password — typically 1 to 3 attempts before either succeeding or giving up and requesting a password reset. Setting the threshold at 5 leaves margin for the occasional distracted user without allowing meaningful brute-force volume to slip past undetected. For a service account rather than a human user, this threshold would likely need to be set lower (e.g., >2 or >3 in 5 minutes), since automated service accounts typically don't fail authentication at all under normal operation, making even a small number of failures unusual and worth flagging.

## Scenario 4 — Periodic Beacon

No fixed count-based threshold was used, since this detection is based on timing consistency rather than volume. The result — an average interval of 31.14 seconds with a standard deviation of only 1.95 seconds — was treated as sufficient evidence of periodicity without needing to define and test multiple threshold iterations in this lab session. A production deployment would need a defined standard-deviation ceiling to formally distinguish genuine beaconing from coincidentally regular legitimate traffic.

## Scenario 1 / 3

No Splunk-based detection rule was built for these two scenarios (see `spl-queries.md` for the reasoning), so there is no tuning history to report here. Detection and validation for both relied on Wireshark packet analysis and direct tool output (Nmap, Gobuster) rather than a Splunk-based threshold rule.
