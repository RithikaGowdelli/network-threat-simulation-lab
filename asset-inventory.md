# Asset Inventory

| Asset | Type | OS | IP Address | Role | Owner | Notes |
|---|---|---|---|---|---|---|
| Kali Linux | VM | Kali Rolling 2026.1 | 192.168.50.10 (eth0, labnet) | Attacker platform, Wireshark capture point | Rithika | Static IP not persisted — reset needed after every reboot |
| Metasploitable2 | VM | Ubuntu 8.04 (intentionally vulnerable) | 192.168.50.20 (eth0, labnet); 192.168.56.20 (eth1, host-only, reaches Splunk) | Target | Rithika | Known-vulnerable by design, isolated only |
| Windows Host | Physical/Host machine | Windows | Host-only network 192.168.56.x | Runs Splunk Enterprise, VirtualBox host | Rithika | Not part of the isolated labnet segment; only reachable via the host-only adapter |
| VirtualBox Internal Network | Virtual switch | N/A | N/A ("labnet") | Isolation boundary | Rithika | No bridged adapter, no NAT to internet |

## Software Versions

| Tool | Version | Purpose |
|---|---|---|
| Splunk | Splunk Enterprise (60-day trial) | SIEM / log search |
| Wireshark | Default version bundled with Kali Rolling 2026.1 (exact version not recorded) | Packet capture and analysis |
| Hydra | Default version bundled with Kali Rolling 2026.1 (exact version not recorded) | Brute-force simulation tool used in Scenario 2 |
| Nmap | Default version bundled with Kali Rolling 2026.1 (exact version not recorded) | Port scanning tool used in Scenario 1 |
| Gobuster | Default version bundled with Kali Rolling 2026.1 (exact version not recorded) | Directory/path enumeration tool used in Scenario 3 |
| Netcat (nc) | Default version bundled with Kali Rolling 2026.1 and Ubuntu 8.04 (exact version not recorded) | Listener and beacon simulation tool used in Scenario 4 |

## Scope Statement

All assets listed above are owned and controlled by the lab operator. No external, third-party, or production systems are included in this inventory or targeted by any scenario in this repository.
