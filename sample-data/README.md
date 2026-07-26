# Sample Data

This folder holds sanitized excerpts of real evidence from the lab (logs, pcap summaries, SPL output) — small enough to review in a browser, not full raw dumps.

## Sanitization Rules

Before adding anything here:

- Replace any real external IP with a documented placeholder (this lab is isolated, so this mainly matters if your Kali/Metasploitable2 IPs happen to overlap with a real routable range you don't want indexed)
- Strip any credentials, even lab test credentials, from log excerpts
- Keep excerpts short — 10 to 30 lines is enough to prove the finding, a full multi-thousand-line log dump is not necessary and is harder for a reviewer to read
- Note the source file and line range each excerpt was pulled from, so it's traceable

## Files

- `ssh-auth-log-excerpt.txt` — 6 real lines pulled from Scenario 2's ingested `attack_sample3.log`, showing the failed SSH login format (`Failed password for msfadmin from 192.168.50.10`). IPs left unchanged since this lab is fully internal/isolated (192.168.50.x is not a real routable range used elsewhere in this repo).
- `nmap-scan-output.txt` — full terminal output from Scenario 1's port scan (`nmap -sT -sV 192.168.50.20`), showing all 23 open ports and service versions identified.
- Scenario 3's full Gobuster output and Scenario 4's full beacon connection CSV (`scenario4-beacon-connections.csv`) are kept in their respective scenario folders rather than duplicated here, since they are already short enough to review directly (Gobuster output is in `scenarios/03-http-path-enumeration/README.md`; the beacon CSV is in `scenarios/04-outbound-connection/logs/`).
