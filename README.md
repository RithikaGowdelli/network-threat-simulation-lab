# Network Threat Simulation and Incident Validation Lab

## Summary

This project is a self-contained, isolated home lab used to generate controlled security events, investigate them using a SIEM (Splunk) and packet analysis (Wireshark), extract indicators of compromise, map findings to MITRE ATT&CK, and document the full incident response workflow.

All activity in this repository was generated in a private, isolated virtual network against systems I own and control. No public systems, real organizations, or third-party infrastructure were involved at any point.

## Why this project exists

Most SOC analyst portfolios show tools. This one shows workflow: generate an event, catch it in logs, prove it happened at the packet level, write the detection query, tune it, document the finding the way an analyst would write it up for a shift handoff.

## Lab Environment

| Component | Role | Details |
|---|---|---|
| Kali Linux | Attacker | [PLACEHOLDER — IP, VM specs] |
| Metasploitable2 | Target | [PLACEHOLDER — IP, VM specs] |
| Splunk | SIEM / log analysis | [PLACEHOLDER — version, indexes used] |
| Wireshark | Packet capture / validation | [PLACEHOLDER — version] |
| pfSense | Network segmentation / firewall | [PLACEHOLDER — if used] |
| Network mode | Isolation | VirtualBox internal network ("labnet"), no bridge to host or internet |

Full details: [architecture/lab-architecture.md](architecture/lab-architecture.md)
Full asset list: [asset-inventory.md](asset-inventory.md)

## Scenarios

| # | Scenario | Status | MITRE Technique(s) |
|---|---|---|---|
| 1 | Network Reconnaissance and Port Scanning | [PLACEHOLDER — Not started / In progress / Complete] | T1595, T1046 |
| 2 | Repeated SSH Authentication Failures | [PLACEHOLDER] | T1110, T1078, T1068 |
| 3 | Suspicious HTTP Path Enumeration | [PLACEHOLDER] | T1595.003, T1190 |
| 4 | Controlled Unexpected Outbound Connection | [PLACEHOLDER] | T1071, T1041 |

Each scenario has its own README under `/scenarios/` with the full investigation writeup.

## Detection Engineering

- [SPL query documentation](detection-engineering/spl-queries.md)
- [Detection tuning report](detection-engineering/detection-tuning-report.md)

## Incident Report

- [Full incident investigation report](incident-reports/incident-report-01.md)

## Lessons Learned

- [lessons-learned.md](lessons-learned.md)

## Repository Structure

```
network-threat-simulation-lab/
├── README.md
├── architecture/
│   └── lab-architecture.md
├── asset-inventory.md
├── scenarios/
│   ├── 01-network-reconnaissance/README.md
│   ├── 02-ssh-brute-force/README.md
│   ├── 03-http-path-enumeration/README.md
│   └── 04-outbound-connection/README.md
├── detection-engineering/
│   ├── spl-queries.md
│   └── detection-tuning-report.md
├── incident-reports/
│   └── incident-report-01.md
├── sample-data/
│   └── README.md
├── docs/
│   └── commit-message-guide.md
└── lessons-learned.md
```

## Disclaimer

All testing was performed exclusively against systems I own, in an isolated virtual lab with no internet-facing exposure. This repository is for educational and portfolio purposes only.
