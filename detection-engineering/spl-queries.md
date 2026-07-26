# Detection Query Documentation

This file consolidates every SPL query used across the four scenarios, with plain-English explanation of what each line does.

## Scenario 2 — SSH Brute Force Detection

**Purpose:** Flag any source IP with more than 5 failed SSH authentication attempts within a 5-minute window.

```
index=main sourcetype=linux_secure "Failed password"
| rex field=_raw "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| bin _time span=5m
| stats count by src_ip, _time
| where count > 5
```

**Line-by-line explanation:**
1. `index=main sourcetype=linux_secure "Failed password"` — searches the ingested auth log data, filtering to only lines containing a failed login attempt
2. `rex field=_raw "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"` — extracts the source IP address out of the raw log text into a new field called `src_ip`, since Splunk did not auto-extract this field on ingestion
3. `bin _time span=5m` — groups events into fixed 5-minute time windows
4. `stats count by src_ip, _time` — counts how many failed attempts occurred per IP, per 5-minute window
5. `where count > 5` — keeps only the rows where a single IP exceeded 5 failures in one window, the actual detection threshold

**Result when run against real data:** 2 rows returned, both from `192.168.50.10` — 1,041 failures in one window, 653 in the next, totaling 1,694 (matching the original Hydra attack count exactly).

**Tuning notes:** A threshold of 5 failures per 5 minutes was chosen because it is well above what a legitimate user would generate by mistyping a password (typically 1 to 3 attempts), keeping the false-positive risk low while still catching automated brute-force volume, which in this test exceeded the threshold by more than 100x. This threshold was set directly based on that reasoning rather than iterated through multiple rounds of tuning in this lab session.

## Scenario 1 — Port Scan Detection

Not implemented in this lab. Detection and validation for this scenario relied on Wireshark packet-level analysis (isolating the SYN packet burst via `ip.addr==192.168.50.20 && tcp.flags.syn==1 && tcp.flags.ack==0`) and direct Nmap output, rather than a Splunk-ingested log source. Metasploitable2 does not log incoming connection attempts by default, and no additional log source (e.g. a firewall or IDS) was introduced to capture this. This is a deliberate scope decision for this scenario, not an oversight — the Splunk/SIEM detection skillset is demonstrated in Scenarios 2 and 4 instead.

## Scenario 3 — HTTP Path Enumeration Detection

Not implemented in this lab, for the same reason as Scenario 1: no Apache access log was ingested into Splunk for this scenario. Detection and validation relied on Gobuster's own output (4,613 paths tested) cross-validated against Wireshark's HTTP request count (4,615 displayed requests). If pursued in a future pass, the workflow would be: ingest the Apache combined log format access log, then count unique URI paths requested per source IP per minute, flagging any IP exceeding a defined threshold — a high rate of unique-path requests in a short window is the enumeration signature, similar in concept to the failure-count threshold used in Scenario 2.

## Scenario 4 — Periodic Beacon Detection

**Purpose:** Detect a host generating outbound connection attempts at suspiciously consistent time intervals, a signature of automated C2 beaconing rather than normal, irregular human-driven traffic.

```
index=main source="scenario4-beacon-connections.csv" Info="*[SYN]*" extracted_Source="192.168.50.20"
| table Time
| sort Time
| delta Time as gap_seconds
| stats avg(gap_seconds) as avg_gap_seconds, stdev(gap_seconds) as stdev_gap_seconds
```

**Line-by-line explanation:**
1. `index=main source="scenario4-beacon-connections.csv" Info="*[SYN]*" extracted_Source="192.168.50.20"` — searches the uploaded Wireshark CSV export, filtering to only the outbound connection-attempt packets (SYN) originating from the target machine
2. `table Time | sort Time` — isolates just the timestamp of each attempt, sorted in chronological order
3. `delta Time as gap_seconds` — calculates the time gap between each attempt and the one immediately before it
4. `stats avg(gap_seconds) as avg_gap_seconds, stdev(gap_seconds) as stdev_gap_seconds` — calculates the average gap and how much that gap varies (standard deviation) across all attempts

**Result when run against real data:** average gap of 31.14 seconds, standard deviation of 1.95 seconds across 8 connection attempts — independently recalculated directly from the raw CSV outside Splunk to confirm accuracy, and the numbers matched exactly.

**Tuning notes:** No formal threshold (like Scenario 2's ">5 in 5 minutes") was defined for this scenario, since the detection approach here is pattern-based rather than count-based. A production version of this detection would need a defined standard-deviation ceiling (e.g., flag any repeating destination where the timing standard deviation falls below some number of seconds across a minimum number of connections) to distinguish genuine beaconing from coincidentally regular legitimate traffic, such as a scheduled software update check.

## General Notes on Field Extraction

- Scenario 2's log data required manual field extraction via `rex`, since Splunk's automatic field extraction did not recognize the source IP within the raw `auth.log` line format.
- Scenario 4's data was uploaded as a structured CSV export from Wireshark, which Splunk automatically parsed into individual fields per column (`Time`, `extracted_Source`, `Destination`, `Info`, etc.), requiring no manual `rex` extraction.
- This is a genuine, worth-noting difference in approach depending on log source structure: unstructured text logs need explicit field extraction logic written by the analyst, while structured exports (CSV, JSON) can rely on Splunk's built-in parsing.
