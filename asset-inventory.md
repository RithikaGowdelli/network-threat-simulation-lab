# Asset Inventory

| Asset | Type | OS | IP Address | Role | Owner | Notes |
|---|---|---|---|---|---|---|
| Kali Linux | VM | [PLACEHOLDER — e.g. Kali 2024.x] | [PLACEHOLDER] | Attacker platform, Splunk host, Wireshark capture point | Rithika | Static IP not persisted — reset needed after reboot |
| Metasploitable2 | VM | Ubuntu 8.04 (intentionally vulnerable) | [PLACEHOLDER] | Target | Rithika | Known-vulnerable by design, isolated only |
| pfSense | VM/Appliance | [PLACEHOLDER] | [PLACEHOLDER] | Network segmentation, firewall logging | Rithika | [PLACEHOLDER — in use or planned] |
| VirtualBox Internal Network | Virtual switch | N/A | N/A ("labnet") | Isolation boundary | Rithika | No bridged adapter, no NAT to internet |

## Software Versions

| Tool | Version | Purpose |
|---|---|---|
| Splunk | [PLACEHOLDER] | SIEM / log search |
| Wireshark | [PLACEHOLDER] | Packet capture and analysis |
| Hydra | [PLACEHOLDER] | Brute-force simulation tool used in Scenario 2 |
| Nmap | [PLACEHOLDER] | Port scanning tool used in Scenario 1 |

## Scope Statement

All assets listed above are owned and controlled by the lab operator. No external, third-party, or production systems are included in this inventory or targeted by any scenario in this repository.
