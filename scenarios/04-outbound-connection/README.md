# Scenario 4: Controlled Unexpected Outbound Connection

## Objective

Simulate a compromised host generating periodic outbound "beacon" traffic back to an attacker-controlled listener, detect the underlying timing pattern using Wireshark packet capture and Splunk, and map it to MITRE ATT&CK as command-and-control behavior. Unlike Scenario 2 (which detects attacks by volume/count of failures), this scenario detects a threat by the *regularity of its timing*, a distinct detection technique used to catch malware beaconing in real environments.

## Lab Prerequisites

- Kali Linux (192.168.50.10) and Metasploitable2 (192.168.50.20) on isolated labnet
- Netcat listener on Kali to receive the outbound connection
- A scripted loop on Metasploitable2 to simulate periodic C2 check-in traffic
- Wireshark running on Kali, capturing on eth0, for the full duration of the test
- Splunk Enterprise instance on the Windows host, with Add Data → Upload available

## Controlled Test Procedure

This scenario uses a **simulated periodic beacon** (option 2), not a reverse shell, since it demonstrates a distinct detection method (timing-pattern analysis) not covered elsewhere in this lab.

**Listener on Kali:**
```
nc -lvkp 4444
```
Note: the `-k` flag is intended to keep the listener alive across multiple connections. In practice, this build of netcat did not fully honor `-k` and the listener dropped back to a fresh prompt after each connection closed, requiring manual restarts. This did not affect the underlying traffic pattern or the Wireshark capture, which ran continuously and independently of the listener's state.

**Beacon loop on Metasploitable2:**
```
while true; do echo beacon | nc -w 2 192.168.50.10 4444; sleep 30; done
```
Note: the original plan was to use `timeout 2` to force each connection closed. Metasploitable2's Ubuntu 8.04 base does not have `timeout` installed, so `nc -w 2` (netcat's own built-in connection timeout) was used instead, achieving the same effect.

Both IPs were confirmed set correctly before starting (`192.168.50.10` on Kali, `192.168.50.20` on Metasploitable2), consistent with the standard IP-reset-on-reboot issue documented across all scenarios in this lab.

## Logs and Evidence to Collect

- Wireshark .pcap showing all outbound connection attempts from Metasploitable2 to Kali on port 4444
- Kali listener terminal output showing received "beacon" messages
- Metasploitable2 terminal output showing the loop running and any connection errors
- A CSV export of the filtered Wireshark capture, uploaded into Splunk for timing analysis

## Wireshark Analysis Filters

- `ip.addr==192.168.50.20 && tcp.port==4444` — isolate all beacon-related traffic between target and attacker listener

**Result:** 46 total packets captured, 28 matched this filter. The capture shows two fully successful beacons (SYN → SYN-ACK → ACK → PSH-ACK carrying the "beacon" payload → ACK) at the start, followed by six later attempts that were refused (SYN sent, immediate RST-ACK returned) once the listener stopped accepting new connections. Despite the refusals, every single attempt — successful or refused — occurred on the same ~30-second schedule. This is itself a realistic and worth-noting finding: a compromised host will continue attempting to check in with its C2 server on schedule even if that server is temporarily unreachable, which is genuine malware behavior, not a flaw in this test.

## Splunk Investigation Workflow

The Wireshark capture was exported as CSV (`File → Export Packet Dissections → As CSV`) and uploaded into Splunk via Add Data → Upload, with sourcetype `csv` (auto-detected correctly, no manual override needed, unlike Scenario 2's log file).

Ingestion confirmed with:
```
index=main source="scenario4-beacon-connections.csv"
```
28 events returned, matching the Wireshark export exactly.

## Working SPL Queries

Isolate the outbound SYN attempts from the target and calculate the timing gap between each one:

```
index=main source="scenario4-beacon-connections.csv" Info="*[SYN]*" extracted_Source="192.168.50.20"
| table Time
| sort Time
| delta Time as gap_seconds
| stats avg(gap_seconds) as avg_gap_seconds, stdev(gap_seconds) as stdev_gap_seconds
```

**Result:**
- Average gap between beacon attempts: **31.14 seconds**
- Standard deviation: **1.95 seconds**

Independently recalculated directly from the raw CSV (outside Splunk) to confirm accuracy — the numbers matched exactly. A standard deviation this low relative to the average gap indicates highly consistent, machine-generated timing rather than random or human-driven traffic, which is the core signature used to detect beaconing behavior in real SOC environments.

## Investigation Questions

- **Why is this connection anomalous?** A host repeatedly initiating outbound connections to the same destination at near-identical time intervals is not typical of normal user-driven traffic, which tends to be irregular and bursty.
- **What port was used, and is it commonly allowed or overlooked?** Port 4444 was used in this lab for simplicity; in a real attack, this traffic would more likely use a commonly allowed port like 443 or 53 to blend in with legitimate traffic. This is worth noting as a limitation of the simulation.
- **How long did each connection stay open, and was data transferred?** The two successful connections completed a full handshake and delivered a small payload ("beacon") before closing normally. The six later attempts were refused immediately and transferred no data.
- **What baseline would "normal outbound behavior" need to violate to trigger an alert in production?** A real detection rule would need a defined threshold for timing consistency (e.g., standard deviation below some number of seconds across a minimum number of repeated connections to the same destination) to distinguish beaconing from coincidental, regularly-scheduled legitimate traffic (like an OS update checker).

## Indicators of Compromise

| Type | Value | Notes |
|---|---|---|
| Source IP (target/beaconing host) | 192.168.50.20 | Metasploitable2, simulating a compromised machine |
| Destination IP (C2 listener) | 192.168.50.10 | Kali, simulating the attacker's server |
| Destination port | 4444 | Chosen for lab simplicity; real C2 traffic often uses 443 or 53 to blend in |
| Beacon interval | ~31.14 seconds average, 1.95 second standard deviation | Calculated from 8 connection attempts across the capture |
| Successful connections | 2 of 8 | Remaining 6 refused due to listener restart behavior, not a change in beacon timing |

## Timeline

| Time (relative, seconds) | Event |
|---|---|
| 0.0 | First beacon: full connection established, "beacon" payload delivered |
| 33.99 | Second beacon: full connection established, "beacon" payload delivered |
| 67.99 | Third attempt: connection refused (listener not active) |
| 97.99 | Fourth attempt: connection refused |
| 127.99 | Fifth attempt: connection refused |
| 157.99 | Sixth attempt: connection refused |
| 187.99 | Seventh attempt: connection refused |
| 218.00 | Eighth attempt: connection refused |

## True Positive / False Positive Assessment

True positive by design — this was a deliberate, controlled simulation. In a real environment, a false positive for this detection approach would most likely come from legitimate scheduled software (update checkers, license validation pings, monitoring agents, or heartbeat health checks) that also connect out at fixed intervals. Distinguishing these from real beaconing would rely on checking whether the destination is a known, approved vendor endpoint, whether the traffic uses a documented legitimate protocol, and whether the interval matches a publicly known legitimate service's documented check-in schedule.

## MITRE ATT&CK Mapping

| Technique ID | Name | Justification |
|---|---|---|
| T1071 | Application Layer Protocol | The beacon used a simple TCP connection to communicate with the listener, simulating an application-layer C2 channel |
| T1571 | Non-Standard Port | Port 4444 is a well-known non-standard port commonly associated with reverse shells and C2 tooling in real-world attacks |

Note: T1041 (Exfiltration Over C2 Channel) is not included, since no meaningful data transfer occurred — only a small fixed "beacon" string was sent, which does not constitute exfiltration.

## Remediation Recommendations

**Problem:** During testing, the target machine was able to send repeated outbound connections to an external address with no restriction or monitoring in place. A script made connection attempts to that address every 31 seconds, and nothing on the network flagged this pattern.

**Why it matters:** If this had been real malware instead of a test script, this beaconing pattern would mean the machine is already compromised and is maintaining ongoing contact with an attacker's command server. Without detection, the attacker could continue sending it instructions, pull data out of the network, or trigger further attacks like ransomware at any time. This is a realistic risk because most networks don't monitor for repeating time-interval patterns in outbound traffic, so this kind of activity can quietly blend in and go unnoticed for a long time.

**Fix:** Implement monitoring for outbound traffic that connects to the same destination at consistent, repeating time intervals, using the same logic demonstrated in this investigation (calculating the average gap and variation between connection attempts). Once a low-variance, repeating pattern is identified, it should trigger an alert for investigation, or be blocked outright if the destination is unrecognized or unapproved.

## Status

Complete: beacon simulation executed, Wireshark validation done, Splunk timing-pattern detection query built and verified independently, remediation drafted.
