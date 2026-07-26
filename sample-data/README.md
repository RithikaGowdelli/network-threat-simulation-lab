# Sample Data

This folder holds sanitized excerpts of real evidence from the lab (logs, pcap summaries, SPL output) — small enough to review directly in a browser, rather than full raw dumps.

Excerpts here are kept short (6 to 30 lines) since that's enough to demonstrate each finding, and every excerpt is traceable back to the full evidence and scenario README it was pulled from. All IP addresses are left unchanged, since this lab runs entirely on an isolated internal network (192.168.50.x / 192.168.56.x) with no connection to any real routable network.

## Files

- **`ssh-auth-log-excerpt.txt`** — 6 lines pulled from Scenario 2's ingested `attack_sample3.log`, showing the failed SSH login format (`Failed password for msfadmin from 192.168.50.10`).
- **`nmap-scan-output.txt`** — full terminal output from Scenario 1's port scan (`nmap -sT -sV 192.168.50.20`), showing all 23 open ports and service versions identified.

Scenario 3's full Gobuster output and Scenario 4's full beacon connection CSV (`scenario4-beacon-connections.csv`) are kept in their respective scenario folders rather than duplicated here, since they are already short enough to review directly:
- Gobuster output: [`scenarios/03-http-path-enumeration/README.md`](../scenarios/03-http-path-enumeration/README.md)
- Beacon CSV: [`scenarios/04-outbound-connection/logs/`](../scenarios/04-outbound-connection/logs/)
