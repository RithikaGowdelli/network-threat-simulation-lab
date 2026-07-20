# Sample Data

This folder holds sanitized excerpts of real evidence from the lab (logs, pcap summaries, SPL output) — small enough to review in a browser, not full raw dumps.

## Sanitization Rules

Before adding anything here:

- Replace any real external IP with a documented placeholder (this lab is isolated, so this mainly matters if your Kali/Metasploitable2 IPs happen to overlap with a real routable range you don't want indexed)
- Strip any credentials, even lab test credentials, from log excerpts
- Keep excerpts short — 10 to 30 lines is enough to prove the finding, a full multi-thousand-line log dump is not necessary and is harder for a reviewer to read
- Note the source file and line range each excerpt was pulled from, so it's traceable

## Files

[PLACEHOLDER — list each sanitized file added here with a one-line description, e.g.:
- `ssh-auth-log-excerpt.txt` — 25 lines from /var/log/auth.log covering the Hydra brute force window, IPs unchanged since lab-internal only
- `nmap-scan-output.txt` — full terminal output from Scenario 1 port scan
]
