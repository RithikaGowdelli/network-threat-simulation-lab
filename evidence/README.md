# Evidence Index

Screenshots and supporting evidence for all four scenarios, curated to show only the images that directly prove a finding. Files marked **KEY** are the strongest, most important pieces of evidence for that scenario.

## /evidence/scenario-01/ (Network Reconnaissance)
1. Nmap scan output — part 1 (open ports and service versions)
2. Nmap scan output — part 2
3. Wireshark SYN packet filter, independently validating the scan actually happened
4. **KEY** — vsftpd backdoor exploit succeeded: `whoami` returns `root` with zero authentication
5. **KEY** — exploit confirmed: `hostname` and `id` prove this is the real target, not a local shell

## /evidence/scenario-02/ (SSH Brute Force Detection)
1. **KEY** — SPL query and stats table showing the two threshold-breaching bins (1,041 + 653 failures, totaling 1,694)
2. Raw event list confirming 1,698 events ingested into Splunk
3. Expanded raw event showing log format and field parsing
4. **KEY** — Alert "SSH Brute Force Detection" saved and enabled without error
5. **KEY** — Wireshark capture confirming 1,679 packets matching the SSH traffic filter, independently validating the Splunk finding

## /evidence/scenario-03/ (HTTP Path Enumeration)
1. Gobuster scan output — part 1
2. Gobuster scan output — part 2
3. **KEY** — Wireshark confirms 4,615 requests, independently validating Gobuster's 4,613 paths
4. TWiki revision page, confirming the install is genuinely from 2003
5. **KEY** — TWiki command injection test result: payload returned as literal text, confirmed negative finding
6. phpMyAdmin response headers, confirming reachability before credential testing

## /evidence/scenario-04/ (Periodic Outbound Beacon)
1. Kali listener output showing beacon messages received
2. Metasploitable2 terminal showing the beacon loop running and connection-refused responses
3. **KEY** — Wireshark capture showing the first two successful beacons with full handshake and payload delivery, 33.99 seconds apart
4. **KEY** — Wireshark capture showing later connection attempts, still landing on the same ~30 second schedule despite being refused
5. **KEY** — Splunk query result showing a 31.14 second average beacon interval with only a 1.95 second standard deviation

## /evidence/troubleshooting/ (IP Conflict Diagnosis)
Real network troubleshooting encountered mid-project, kept as evidence of practical debugging skill rather than just tool output.
1. **KEY** — root cause found: Kali was accidentally assigned the same IP as the target (192.168.50.20), explaining why `ping` worked but HTTP did not
2. **KEY** — conflict fixed, Kali correctly restored to 192.168.50.10
3. **KEY** — connectivity confirmed restored, real HTML returned from the target

## /evidence/github-setup/
1. **KEY** — successful `git push`, confirming the repository is live on GitHub
