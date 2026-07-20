# Scenario 4: Controlled Unexpected Outbound Connection

## Objective

Simulate a compromised host initiating an unexpected outbound connection (e.g. a reverse shell or beacon-style callback) from Metasploitable2 back to Kali, detect it in Splunk and Wireshark, and map it to MITRE ATT&CK as a command-and-control / exfiltration-style behavior.

## Lab Prerequisites

- Kali Linux and Metasploitable2 on isolated labnet
- A listener set up on Kali (e.g. netcat) to receive the outbound connection
- A method for triggering the outbound connection from Metasploitable2 (e.g. a reverse shell payload executed on the target, simulating post-exploitation behavior)
- Confirm and document exactly how you're triggering this — this needs to be a real, reproducible step, not assumed

## Controlled Test Procedure

[PLACEHOLDER — this scenario needs the most precision since "unexpected outbound connection" can mean several different things. Confirm with me which of these you're actually doing:
1. A reverse shell triggered manually on Metasploitable2 connecting back to a Kali listener
2. A scripted periodic "beacon" simulating C2 check-in traffic
3. Something else

Once confirmed, document:
```
[PLACEHOLDER — listener command on Kali, e.g. nc -lvnp [PORT]]
[PLACEHOLDER — trigger command on Metasploitable2]
```
]

## Logs and Evidence to Collect

- Wireshark .pcap showing the outbound connection establishment
- Any host-side log on Metasploitable2 showing the process that initiated the connection (if available)
- Splunk search results, assuming a log source is feeding this (confirm what, since this scenario may rely mainly on network evidence)
- Netcat listener output on Kali showing the connection was received

## Wireshark Analysis Filters

- `ip.src == [TARGET_IP] && ip.dst == [KALI_IP]` — isolate the outbound connection from target to attacker
- `tcp.flags.syn == 1 && tcp.flags.ack == 0 && ip.src == [TARGET_IP]` — isolate the initiating SYN from the target, which is the anomaly (targets don't normally initiate connections to attacker machines)
- Follow TCP stream on the relevant connection to see shell interaction if applicable

## Splunk Investigation Workflow

[PLACEHOLDER — same caveat as Scenario 1: Metasploitable2 doesn't have rich outbound connection logging by default. Confirm your actual log source for this scenario before I write SPL against it. Likely candidates: pfSense firewall log if in use, or a flow-log-style export of the pcap.]

## Working SPL Queries

```
[PLACEHOLDER — paste your actual tested query once the log source is confirmed]
```

## Investigation Questions

- Why is this connection anomalous — what makes "target initiates connection to attacker" different from normal expected traffic direction?
- What port was used, and is it a commonly allowed/overlooked port (e.g. 443, 53) versus an obviously suspicious one?
- How long did the connection stay open, and was any data actually transferred?
- In a real environment, what baseline of "normal outbound behavior" would this have needed to violate to trigger an alert?

## Indicators of Compromise

| Type | Value | Notes |
|---|---|---|
| Source IP (target) | [PLACEHOLDER] | |
| Destination IP (attacker) | [PLACEHOLDER] | |
| Destination port | [PLACEHOLDER] | |
| Connection duration | [PLACEHOLDER] | |

## Timeline

| Time (UTC) | Event |
|---|---|
| [PLACEHOLDER] | Listener started on Kali |
| [PLACEHOLDER] | Trigger executed on target |
| [PLACEHOLDER] | Connection established |
| [PLACEHOLDER] | Connection closed |

## True Positive / False Positive Assessment

[PLACEHOLDER — true positive by design. Note what would make this a false positive in production, e.g. legitimate outbound monitoring agents, scheduled software update checks.]

## MITRE ATT&CK Mapping

| Technique ID | Name | Justification |
|---|---|---|
| T1071 | Application Layer Protocol | If the outbound connection uses a standard protocol to blend in with normal traffic |
| T1041 | Exfiltration Over C2 Channel | Only include if you actually simulated data transfer over the connection — don't claim exfiltration if you only established the connection |

## Remediation Recommendations

- [PLACEHOLDER — draft yourself: egress filtering, outbound allowlisting, DNS/traffic anomaly detection, EDR process-to-network correlation]

## Screenshot Checklist

- [ ] Kali listener output showing connection received
- [ ] Wireshark capture showing the anomalous outbound SYN
- [ ] Splunk search results (if a log source is confirmed for this scenario)
- [ ] Any host-side evidence of the triggering process

## Status

[PLACEHOLDER — Not started / In progress / Complete]
