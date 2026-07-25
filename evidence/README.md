# Evidence Index

15 screenshots, curated down from 60 originally captured — kept only the images that actually prove a finding. Files marked **KEY** are the strongest, most important pieces of evidence in the whole project.

## /evidence/scenario-01/ (Network Reconnaissance)
1. Nmap scan output — part 1 (open ports and service versions)
2. Nmap scan output — part 2
3. Wireshark SYN packet filter, independently validating the scan actually happened
4. **KEY** — vsftpd backdoor exploit succeeded: `whoami` returns `root` with zero authentication
5. **KEY** — exploit confirmed: `hostname` and `id` prove this is the real target, not a local shell

## /evidence/scenario-03/ (HTTP Path Enumeration)
1. Gobuster scan output — part 1
2. Gobuster scan output — part 2
3. **KEY** — Wireshark confirms 4,615 requests, independently validating Gobuster's 4,613 paths
4. TWiki revision page, confirming the install is genuinely from 2003
5. **KEY** — TWiki command injection test result: payload returned as literal text, confirmed negative finding
6. phpMyAdmin response headers, confirming reachability before credential testing

## /evidence/troubleshooting/ (IP Conflict Diagnosis)
Real network troubleshooting encountered mid-project — kept as evidence of actual debugging skill, not just tool output.
1. **KEY** — root cause found: Kali was accidentally assigned the same IP as the target (192.168.50.20), explaining why ping worked but HTTP didn't
2. **KEY** — conflict fixed, Kali correctly restored to 192.168.50.10
3. **KEY** — connectivity confirmed restored, real HTML returned from the target

## /evidence/github-setup/
1. **KEY** — successful `git push`, confirming the repository is live on GitHub

---

**Note:** a Splunk setup folder existed in an earlier version of this evidence set but was removed — that work is still in progress (license renewal blocking Scenario 2) and one of those screenshots had a real password visible on screen. Clean screenshots will be added once Scenario 2 is actually completed successfully.
