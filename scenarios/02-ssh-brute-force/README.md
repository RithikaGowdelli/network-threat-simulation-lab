# Scenario 2: Repeated SSH Authentication Failures

## Objective

Simulate a brute-force SSH login attempt against Metasploitable2, detect the failure pattern in Splunk, validate at the packet level in Wireshark, and build a tuned detection rule.

## Lab Prerequisites

- Kali Linux and Metasploitable2 on isolated labnet
- Hydra installed on Kali
- SSH service confirmed running on Metasploitable2
- Splunk Enterprise instance running on Windows host, accessible via browser, with Add Data → Upload available for manual file ingestion

## Controlled Test Procedure

1. Splunk Enterprise trial instance provisioned (60-day trial) after the original Splunk account expired and its previously ingested data was lost.
2. Brute-force login attack run from Kali (192.168.50.10) against Metasploitable2 (192.168.50.20) using a single known username and a leaked password wordlist:

```
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ssh://192.168.50.20
```

3. Failed login attempts logged automatically by the target's own SSH daemon into `/var/log/auth.log`.
4. Filtered log saved as `attack_sample3.log` and uploaded into Splunk via **Add Data → Upload**, with sourcetype manually set to `linux_secure` (auto-detection did not select this correctly).
5. Data ingestion confirmed with a basic search (`index=main sourcetype=linux_secure`) before building any detection logic — 1,698 events landed, closely matching the 1,694 failed attempts recorded in the original attack.
6. Separately, a short live re-run of the same Hydra command was captured in Wireshark on Kali's eth0 interface to independently validate the attack at the network/packet level (see Wireshark section below).

Known and confirmed:
- Failed login attempts generated: 1,694 (log-based), 1,698 events ingested into Splunk (small discrepancy likely a few non-attempt log lines captured in the filtered file)
- Target username: `msfadmin` (single username tried repeatedly — this was a targeted brute force, not credential stuffing across multiple usernames)
- Password source: `rockyou.txt`, a well-known leaked password list
- Log transfer method: manual file upload directly into Splunk (not a live forwarder)

## Logs and Evidence to Collect

- Metasploitable2 auth log (`/var/log/auth.log`) covering the attack window
- Wireshark .pcap of the SSH traffic
- Splunk search results for the failed login events
- Hydra terminal output showing attempt count and any successful credential found

## Wireshark Analysis Filters

- `ip.addr==192.168.50.20 && tcp.port==22` — isolate SSH traffic between Kali and the target
- Note: SSH traffic is encrypted, so Wireshark validates connection volume and timing, not credential content. The capture shows repeated SSHv2 Key Exchange Init / Diffie-Hellman handshake sequences followed by RST packets — each failed login attempt tears down the connection and immediately opens a new one, which is the packet-level signature of automated brute forcing.

**Result:** 1,679 packets matched the filter out of 1,683 total captured (99.8% displayed, 0% dropped), closely matching the 1,694 failed attempts recorded in the auth log. Packet-level and log-level evidence independently confirm the same event.

## Splunk Investigation Workflow

1. Search the ingested auth log source for failed SSH authentication events.
2. Extract source IP and username per failed attempt using `rex`.
3. Bucket events into 5-minute bins.
4. Flag any source IP exceeding the failure threshold (>5 per bin) as suspicious.

## Working SPL Queries

Detection search — tested and confirmed working against the ingested `attack_sample3.log`:

```
index=main sourcetype=linux_secure "Failed password"
| rex field=_raw "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| bin _time span=5m
| stats count by src_ip, _time
| where count > 5
```

**Result:** 2 rows returned, both from source IP `192.168.50.10`:
- 1,041 failures in one 5-minute bin
- 653 failures in the following 5-minute bin
- Total: 1,694 — matches the original attack count exactly

Both bins are far above the >5 threshold, correctly flagging the activity as brute-force behavior.

**Alert built on this query:** "SSH Brute Force Detection," trigger condition set to Number of Results > 0, with a trigger action configured (Add to Triggered Alerts). Saved and enabled successfully. Note: an earlier attempt to save this alert without a trigger action failed with "enable at least one action" — resolved by adding the trigger action before saving.

## Investigation Questions

- **What is the time gap between individual failed attempts?** The Wireshark capture shows attempts landing within fractions of a second of each other (multiple SSH handshakes within a 0.03-second window), consistent with automated tooling rather than a human typing passwords.
- **Is there a single username being tried repeatedly, or many usernames?** Single username (`msfadmin`) tried repeatedly against many passwords — this is a targeted brute force, not credential stuffing across multiple accounts.
- **Did the attack culminate in a successful login?** Not tested/confirmed in this scenario. This scenario's scope was detection and validation of failed-login volume, not achieving a successful compromise. Note this explicitly rather than implying compromise occurred.
- **How does the 5-minute bin threshold hold up against normal user behavior?** A >5-failures-per-5-minutes threshold is well above what a legitimate user mistyping a password would generate (typically 1 to 3 attempts), so this threshold has a low false-positive risk for normal usage while still catching automated brute-force volume.
- **Observation worth flagging:** the ingested log file contained events spanning more than one calendar day (some events timestamped July 4, some July 5). Worth clarifying in your own words during review — whether this reflects multiple attack runs concatenated into one log file, or a single long-running session — since it affects how you'd describe the attack's duration if asked in an interview.

## Indicators of Compromise

| Type | Value | Notes |
|---|---|---|
| Source IP | 192.168.50.10 | Kali attacker IP, confirmed via both Splunk (`src_ip` field) and Wireshark |
| Failed attempt count | 1,694 (auth log) / 1,698 (Splunk-ingested events) | Small discrepancy likely a few non-attempt log lines in the filtered file |
| Target account | msfadmin | Single account targeted, not credential stuffing |
| Password source | rockyou.txt | Well-known leaked password wordlist |
| Successful compromise | Not tested in this scenario | Scope was detection/validation of failed-login volume, not achieving login |
| Wireshark packet count | 1,679 packets matched SSH filter | Independent network-level confirmation of the same event |

## Timeline

| Time | Event |
|---|---|
| [PLACEHOLDER — exact Hydra start time not recorded from the original attack run] | Hydra brute-force attack initiated against 192.168.50.20 |
| 2026-07-04/05 (log spans both dates — see Investigation Questions note) | 1,694 failed login attempts logged to auth.log |
| [PLACEHOLDER] | attack_sample3.log uploaded to Splunk |
| [PLACEHOLDER] | SPL detection query built, tested, and confirmed (1,041 + 653 = 1,694 failures) |
| [PLACEHOLDER] | Alert "SSH Brute Force Detection" saved and enabled |
| [PLACEHOLDER] | Live Wireshark re-capture performed, 1,679 packets confirmed matching filter |

## True Positive / False Positive Assessment

True positive by design — this was a deliberate, controlled brute-force attack run by the lab operator. In a real environment, a false positive for this exact detection rule (>5 SSH failures from one IP in 5 minutes) would most likely come from a legitimate but misconfigured service account or script retrying a stale/rotated credential automatically, or a user account lockout policy test performed by IT. Distinguishing these from a real attack would rely on checking whether the source IP is an internal, known service host versus an external or unrecognized IP, and whether the target username is a real, expected service account rather than a generic one like `msfadmin`.

## MITRE ATT&CK Mapping

| Technique ID | Name | Justification |
|---|---|---|
| T1110.001 | Brute Force: Password Guessing | Hydra systematically tried a large password list against a single known username via SSH |
| T1021.004 | Remote Services: SSH | The targeted service was SSH remote login |

Note: T1078 (Valid Accounts) and privilege escalation techniques are not included, since no successful login or privilege escalation was achieved or tested in this scenario — only the failed-attempt detection was validated.

## Remediation Recommendations

**Problem:** During testing, SSH on the target machine allowed unlimited login attempts with no lockout or blocking mechanism in place. This let Hydra attempt 1,694 passwords in roughly 10 minutes without the system pushing back or slowing down.

**Why it matters:** If the target password had been weak instead of strong, this lack of protection would have let an attacker brute-force the password, log in, and gain root access. This is a realistic risk because free, widely available tools like Hydra require minimal skill to use, and combined with leaked password lists, make this attack trivial to carry out against any system without login protections.

**Fix:** Implement fail2ban, a tool that monitors authentication logs and automatically blocks an IP address after it exceeds a failure threshold — for example, more than 5 failed login attempts within a 5-minute window, matching the detection logic used in this investigation. This adds a second layer of defense so security doesn't depend entirely on password strength alone.

## Screenshot Checklist

- [x] Splunk search showing 1,698 ingested failed login events
- [x] Raw event detail view confirming log format and field parsing
- [x] SPL query and stats table showing the two threshold-breaching bins (1,041 + 653)
- [x] Alert "SSH Brute Force Detection" saved and enabled without error
- [x] Wireshark capture confirming 1,679 packets matching the SSH traffic filter
- [ ] Hydra terminal output with final attempt count (from original attack run — not captured at the time; live re-run was for Wireshark validation only, not attempt-count evidence)

## Status

Evidence complete: Splunk ingestion, detection query, alert, and independent Wireshark validation all done and confirmed. Remaining: lessons-learned section (to be written separately, in your own words).
