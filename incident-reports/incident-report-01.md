# Incident Investigation Report — INC-001: SSH Brute Force Against Metasploitable2

**Analyst:** Rithika
**Date of investigation:** 2026-07-24 to 2026-07-25 (Splunk ingestion, detection query, and alert build); Wireshark validation performed same session
**Report status:** Final

## 1. Executive Summary

A brute-force authentication attack was simulated against the SSH service on an isolated lab target (Metasploitable2, 192.168.50.20) using Hydra with a leaked password wordlist. The attack generated 1,694 failed login attempts from a single source IP (192.168.50.10). The activity was detected in Splunk using a custom SPL query that flagged any source IP exceeding 5 failed attempts within a 5-minute window, identifying two distinct attack windows (1,041 and 653 failures). The finding was independently confirmed at the network packet level via Wireshark, which captured 1,679 packets matching the same SSH traffic. This was a controlled, deliberate test; no real compromise occurred, and the activity is confirmed true positive by design.

## 2. Scope

- Systems involved: Kali Linux (192.168.50.10, attacker), Metasploitable2 (192.168.50.20, target)
- Time window investigated: Auth log data spanning July 4 to July 5, 2026 (log timestamps span both dates; the exact single continuous attack duration was not separately timestamped at the start of the original attack run — see Analyst Notes)
- Data sources reviewed: Splunk-ingested `attack_sample3.log` (sourcetype `linux_secure`), Wireshark packet capture filtered on `ip.addr==192.168.50.20 && tcp.port==22`

## 3. Detection

The activity was first identified by building and running a custom SPL query against the ingested auth log:
```
index=main sourcetype=linux_secure "Failed password"
| rex field=_raw "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| bin _time span=5m
| stats count by src_ip, _time
| where count > 5
```
This query flagged source IP 192.168.50.10 in two separate 5-minute windows, with failure counts of 1,041 and 653 — both far exceeding the >5 threshold used to distinguish automated brute-force behavior from normal human login error.

## 4. Investigation Narrative

1. A new Splunk Enterprise trial instance was provisioned after the prior Splunk account expired and its previously ingested data was lost.
2. The filtered log file `attack_sample3.log` was uploaded via Add Data → Upload. The sourcetype was manually set to `linux_secure`, since Splunk's auto-detection did not select the correct sourcetype on its own.
3. Ingestion was confirmed with a basic search (`index=main sourcetype=linux_secure`) before any detection logic was built — 1,698 events landed, closely matching the 1,694 failed attempts recorded in the original attack log (small discrepancy attributed to a few non-attempt lines captured in the filtered file).
4. The detection query (above) was built from scratch, since the original query text from a prior session had not been saved.
5. The query returned two rows, both from the same source IP, both far exceeding the threshold.
6. An alert ("SSH Brute Force Detection") was built on this query. The first save attempt failed with the error "enable at least one action" — resolved by adding a trigger action (Add to Triggered Alerts) before saving again.
7. To independently validate the finding outside of Splunk, a short live re-run of the same Hydra command was captured in Wireshark on Kali's eth0 interface, filtered on `ip.addr==192.168.50.20 && tcp.port==22`. This captured 1,679 packets matching the filter, closely matching the log-based failure count and confirming the finding at the network level as well.

## 5. Evidence

| Evidence type | Location in repo | Description |
|---|---|---|
| Splunk query and stats result | `evidence/scenario-02/01-KEY-spl-query-1041-653-results.png` | SPL query and the two threshold-breaching bins (1,041 + 653 failures) |
| Raw ingested events | `evidence/scenario-02/02-raw-events-list-1698-total.png` | Full event list confirming 1,698 events landed in Splunk |
| Expanded raw event | `evidence/scenario-02/03-raw-event-expanded-fields.png` | Single event showing log format and field parsing |
| Alert configuration | `evidence/scenario-02/04-KEY-alert-saved-enabled.png` | Confirmation the alert saved and enabled without error |
| Wireshark validation | `evidence/scenario-02/05-KEY-wireshark-1679-packets-confirmed.png` | 1,679 packets matching the SSH filter, independent network-level confirmation |
| Raw sample log excerpt | `sample-data/ssh-auth-log-excerpt.txt` | 6 real lines from the ingested auth log, sanitized |

## 6. Timeline (Consolidated)

| Time | Event | Source |
|---|---|---|
| 2026-07-04/05 (log spans both dates) | 1,694 failed SSH login attempts generated via Hydra and logged to auth.log | Original attack log |
| Session date | New Splunk trial provisioned, `attack_sample3.log` uploaded, sourcetype set to `linux_secure` | Splunk |
| Session date | 1,698 events confirmed ingested via basic search | Splunk |
| Session date | Detection SPL query built and tested, returning 1,041 + 653 failure counts from 192.168.50.10 | Splunk |
| Session date | Alert "SSH Brute Force Detection" built, initial save failed ("enable at least one action"), resolved by adding a trigger action | Splunk |
| Session date | Live Hydra re-run captured in Wireshark, 1,679 packets confirmed matching SSH filter | Wireshark |

## 7. Indicators of Compromise (Consolidated)

| Type | Value |
|---|---|
| Source IP | 192.168.50.10 |
| Target account | msfadmin |
| Failed attempt count | 1,694 (auth log) / 1,698 (Splunk-ingested) |
| Password source | rockyou.txt (known leaked password wordlist) |
| Wireshark packet count | 1,679 packets matching SSH filter |
| Successful compromise | Not tested/confirmed — scope was detection and validation of failed-login volume, not achieving a successful login |

## 8. Root Cause Analysis

Metasploitable2 is an intentionally vulnerable lab target with no rate-limiting, account lockout, or connection-throttling controls on its SSH service by design. This allowed Hydra to make 1,694 login attempts without any pushback or slowdown from the target. This finding demonstrates the analysis and detection method (identifying and validating brute-force activity via log volume and independent packet-level confirmation) rather than representing a real-world root cause discovery, since the underlying vulnerability (no lockout policy) was intentionally built into the target for this lab exercise.

## 9. MITRE ATT&CK Summary

| Technique ID | Name | Stage |
|---|---|---|
| T1110.001 | Brute Force: Password Guessing | Credential Access |
| T1021.004 | Remote Services: SSH | Lateral Movement (service targeted) |

## 10. Impact Assessment

In a real environment with a weak or common password on the targeted account, this volume of unrestricted login attempts would very likely have succeeded eventually, granting the attacker full account access and, depending on the account's privileges, potential root-level control of the system. From there, an attacker could establish persistence, move laterally to other systems on the network, or exfiltrate data. Even without a successful login, this volume of failed authentication traffic represents a real load on the target system and a clear signal that should trigger investigation in any monitored environment.

## 11. Remediation and Recommendations

**Problem:** SSH on the target machine allowed unlimited login attempts with no lockout or blocking mechanism in place, letting Hydra attempt 1,694 passwords in roughly 10 minutes without the system pushing back or slowing down.

**Why it matters:** If the target password had been weak instead of strong, this lack of protection would have let an attacker brute-force the password, log in, and gain root access. This is a realistic risk because free, widely available tools like Hydra require minimal skill to use, and combined with leaked password lists, make this attack trivial to carry out against any system without login protections.

**Fix:** Implement fail2ban, a tool that monitors authentication logs and automatically blocks an IP address after it exceeds a failure threshold — for example, more than 5 failed login attempts within a 5-minute window, matching the detection logic used in this investigation. This adds a second layer of defense so security doesn't depend entirely on password strength alone.

## 12. Analyst Notes

- The ingested log data spanned two calendar days (July 4 and July 5), rather than a single tight window. This was not fully resolved during this investigation — it's unclear whether this reflects multiple separate attack runs concatenated into one log file, or a single longer-running session than initially assumed.
- Splunk's automatic field extraction did not recognize the source IP within the raw log line, requiring a manual `rex` extraction — a good example of why understanding Splunk's field extraction mechanics matters beyond just running a search.
- The alert-save error ("enable at least one action") had occurred in a prior session as well and was resolved the same way both times, by adding a trigger action before saving. This appears to be a recurring Splunk UI behavior rather than a one-off issue.
