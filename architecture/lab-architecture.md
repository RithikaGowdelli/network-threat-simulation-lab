# Lab Architecture

## Network Diagram

```
                        ┌─────────────────────────────┐
                        │   VirtualBox Host (Isolated) │
                        └─────────────────────────────┘
                                     │
                     Internal Network: "labnet"
                     No bridge to host adapter, no internet
                                     │
        ┌────────────────────────────┬────────────────────────────┐
        │                            │                             │
 ┌──────────────┐           ┌──────────────────┐          ┌───────────────┐
 │  Kali Linux    │           │  Metasploitable2  │          │  pfSense (opt) │
 │  Attacker/     │──────────▶  Vulnerable Target │◀─────────│  Segmentation  │
 │  Splunk/       │  attacks  │  [PLACEHOLDER IP]  │  logs    │  [PLACEHOLDER] │
 │  Wireshark     │           └──────────────────┘          └───────────────┘
 │  [PLACEHOLDER IP]│
 └──────────────┘
```

Note: replace the ASCII diagram above with an actual exported diagram (draw.io, Lucidchart, or similar) before publishing. Keep this text version as a fallback for viewers who don't load images.

## Network Details

| Item | Value |
|---|---|
| Virtualization platform | VirtualBox |
| Network mode | Internal Network ("labnet") |
| Internet access | None — fully isolated |
| Kali Linux IP | [PLACEHOLDER — e.g. 192.168.50.10, note: not persisted across reboot per your setup] |
| Metasploitable2 IP | [PLACEHOLDER] |
| Splunk deployment | [PLACEHOLDER — running on Kali? Separate VM? Version?] |
| Log ingestion method | [PLACEHOLDER — e.g. Python SimpleHTTPServer manual transfer, universal forwarder, syslog] |

## Data Flow

1. Attacker actions on Kali generate network traffic and target-side logs.
2. Wireshark captures raw packets on the labnet interface for ground-truth validation.
3. Relevant logs (auth logs, web server logs, etc.) are exported from Metasploitable2 and ingested into Splunk.
4. Splunk searches (SPL) are used to detect and investigate the activity.
5. Findings are cross-validated against the Wireshark capture before being logged as true positive or false positive.

## Design Rationale

[PLACEHOLDER — 2 to 3 sentences on why you built it this way, e.g. isolation as a safety control, choice of Metasploitable2 as an intentionally vulnerable target, why Splunk over other SIEMs]
