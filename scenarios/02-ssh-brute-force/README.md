# Scenario 2: Repeated SSH Authentication Failures

## Objective

Simulate a brute-force SSH login attempt against Metasploitable2, detect the failure pattern in Splunk, validate at the packet level in Wireshark, and build a tuned detection rule.

## Lab Prerequisites

- Kali Linux and Metasploitable2 on isolated labnet
- Hydra installed on Kali
- SSH service confirmed running on Metasploitable2
- Log transfer path working (per your setup: manual transfer via Python SimpleHTTPServer into Splunk)

## Controlled Test Procedure

You've already run this attack. Fill in the actual command and confirm the numbers below match what you recorded:

```
[PLACEHOLDER — paste your actual hydra command, e.g. hydra -L userlist -P passlist ssh://[TARGET_IP]]
```

Known from your prior run (confirm these are still accurate before publishing):
- Failed login attempts generated: 1,694
- Log transfer method: Python SimpleHTTPServer, manual transfer to Splunk

## Logs and Evidence to Collect

- Metasploitable2 auth log (`/var/log/auth.log`) covering the attack window
- Wireshark .pcap of the SSH traffic
- Splunk search results for the failed login events
- Hydra terminal output showing attempt count and any successful credential found

## Wireshark Analysis Filters

- `tcp.port == 22 && ip.addr == [TARGET_IP]` — isolate SSH traffic to the target
- `tcp.flags.syn == 1` — count connection attempts, useful to cross-check against Hydra's attempt count
- Note: SSH traffic is encrypted, so Wireshark validates connection volume and timing, not credential content — be explicit about this limitation in your writeup, it's a realistic and correct caveat that shows you understand what packet capture can and can't tell you.

## Splunk Investigation Workflow

1. Search the ingested auth log source for failed SSH authentication events.
2. Extract source IP and username per failed attempt using `rex`.
3. Bucket events into 5-minute bins.
4. Flag any source IP exceeding the failure threshold (>5 per bin) as suspicious.

## Working SPL Queries

You told me you already have a working query using `rex`, 5-minute bins, and a >5-failures-per-IP threshold. Paste the actual query text here — I won't reconstruct it from memory since getting field names wrong would make this look fabricated to anyone who tests it:

```
[PLACEHOLDER — paste your real, tested SPL query]
```

Once you paste it, I'll review it for correctness, suggest tuning, and write the explanation section around it.

## Investigation Questions

- What is the time gap between individual failed attempts? Consistent gaps suggest automation, not a human.
- Is there a single username being tried repeatedly, or many usernames (credential stuffing vs. targeted brute force)?
- Did the attack culminate in a successful login? If yes, this changes the finding from "attempted" to "confirmed compromise" and changes the whole downstream writeup.
- How does the 5-minute bin threshold hold up against normal user behavior — would a legitimate user mistyping their password 3 times in 5 minutes trigger a false positive?

## Indicators of Compromise

| Type | Value | Notes |
|---|---|---|
| Source IP | [PLACEHOLDER] | Kali attacker IP |
| Failed attempt count | 1,694 | Confirm this is the final, accurate number |
| Target account(s) | [PLACEHOLDER] | Username(s) targeted |
| Successful compromise | [PLACEHOLDER — yes/no, and which credential if yes] | |

## Timeline

| Time (UTC) | Event |
|---|---|
| [PLACEHOLDER] | Hydra attack started |
| [PLACEHOLDER] | Threshold breach detected in Splunk |
| [PLACEHOLDER] | Attack ended |
| [PLACEHOLDER] | Correlated in Wireshark |

## True Positive / False Positive Assessment

[PLACEHOLDER — true positive by design since it's your controlled test. Write 2 to 3 sentences on what a false positive would look like for this exact detection rule, e.g. an internal script with hardcoded but rotated bad credentials, or a user account lockout policy test.]

## MITRE ATT&CK Mapping

| Technique ID | Name | Justification |
|---|---|---|
| T1110 | Brute Force | Repeated failed authentication attempts against SSH |
| T1078 | Valid Accounts | Relevant if the brute force succeeded and a valid account was compromised |
| T1068 | Exploitation for Privilege Escalation | Only relevant if you chained this into a privilege escalation step — confirm whether this actually happened in your lab run or remove this row if it didn't |

## Remediation Recommendations

- [PLACEHOLDER — draft these yourself: account lockout policy, fail2ban, key-based auth instead of password auth, rate limiting, alerting threshold tuning]

## Screenshot Checklist

- [ ] Hydra terminal output with final attempt count
- [ ] Splunk search showing the failed login events and the threshold-based alert
- [ ] Wireshark capture showing connection volume/timing
- [ ] auth.log excerpt (sanitized) showing raw failed login entries

## Status

[PLACEHOLDER — Not started / In progress / Complete]
