# Lab Architecture

## Network Diagram

```
                        ┌─────────────────────────────┐
                        │  VirtualBox Host (Windows)  │
                        │ Splunk Enterprise runs here │
                        └─────────────────────────────┘
                                     │
                        Host-only Adapter: 192.168.56.x
                                     │
        ┌────────────────────────────┴────────────────────────────┐
        │                                                         │
 ┌──────────────────┐                                    ┌──────────────────┐
 │   Kali Linux     │      Internal Network: "labnet"    │  Metasploitable2 │
 │   Attacker/      │◀──────────────────────────────▶   |Vulnerable Target │
 │   Wireshark      │      No internet, fully isolated   │  192.168.50.20   │
 │   192.168.50.10  │                                    └──────────────────┘
 └──────────────────┘
```

A rendered version of this diagram is also available in the main [README](../README.md#-lab-architecture).

## Network Details

| Item | Value |
|---|---|
| Virtualization platform | VirtualBox |
| Network mode | Internal Network ("labnet") for Kali ↔ Metasploitable2; separate Host-only Adapter (192.168.56.x) on Metasploitable2's second interface to reach Splunk on the Windows host |
| Internet access | None on labnet — fully isolated |
| Kali Linux IP | 192.168.50.10 (eth0, labnet) — resets on every VM reboot, manually re-added each session |
| Metasploitable2 IP | 192.168.50.20 (eth0, labnet); 192.168.56.20 (eth1, host-only, reaches Splunk) — also resets on reboot |
| Splunk deployment | Splunk Enterprise, running directly on the Windows host (not inside a VM), accessed via browser |
| Log ingestion method | Manual file upload via Splunk's Add Data → Upload feature — log/CSV files generated on Kali or Metasploitable2 were transferred to the Windows host (via VirtualBox shared clipboard or manual copy/paste, since no shared network path existed from Kali directly) and then uploaded into Splunk as static files, rather than a live forwarder or syslog stream |

## Data Flow

1. Attacker actions on Kali generate network traffic and target-side logs.
2. Wireshark captures raw packets on the labnet interface for ground-truth validation.
3. Relevant logs (auth logs, web server logs, Wireshark packet exports) are pulled from Kali/Metasploitable2 and manually transferred to the Windows host.
4. Logs are ingested into Splunk Enterprise via manual file upload.
5. Splunk searches (SPL) are used to detect and investigate the activity.
6. Findings are cross-validated against the Wireshark capture before being logged as true positive or false positive.

## Design Rationale

This lab uses a fully isolated VirtualBox internal network ("labnet") with no internet access or bridge to the host adapter, ensuring that every attack technique simulated here, including active exploitation and brute-force attacks, stays entirely contained and never touches a real or public system. Metasploitable2 was chosen as the target because it is intentionally built with known, well-documented vulnerabilities (an outdated FTP service, unpatched web applications, weak authentication), making it possible to generate realistic, reproducible findings without needing to discover novel vulnerabilities. Splunk Enterprise was used as the SIEM because it is one of the most widely deployed detection platforms in real SOC environments, and building working SPL queries against real log data demonstrates a directly transferable, industry-relevant skill.
