# PROJECT HANDOFF — Network Threat Simulation and Incident Validation Lab

**Purpose of this document:** paste this entire file into a new Claude chat if the current one gets overloaded. It contains everything needed to continue without re-explaining anything or losing progress.

**⚠️ SYNC STATUS — READ THIS FIRST:**
- **GitHub (live, pushed):** `https://github.com/RithikaGowdelli/network-threat-simulation-lab` — currently only has Scenario 1 complete. Everything else below is NOT yet on GitHub.
- **This zip / local backup file (`network-threat-simulation-lab-BACKUP-not-pushed-to-github.zip`):** ahead of GitHub — contains Scenario 1 AND Scenario 3 complete, PLUS a curated `/evidence/` folder with 15 essential screenshots, plus this handoff doc. This is the most current version of the work, but it is only saved locally/in this chat's downloads — NOT yet pushed to the live GitHub repo.
- **Decision made:** hold off pushing to GitHub until Scenario 2 and Scenario 4 are both finished, then do one single final push covering everything at once.
- **If resuming in a new chat:** the zip file is the source of truth for current progress, not the GitHub repo — GitHub is intentionally behind right now.
- **Evidence folder:** `/evidence/` contains 15 curated screenshots (scenario-01, scenario-03, troubleshooting, github-setup), each with clean filenames and an index at `evidence/README.md`. Cut down from 60 originally uploaded — only KEY proof points kept. Splunk screenshots deliberately excluded (unresolved work + one had a visible password).

---

## Lab Environment (confirmed working)

| Item | Value |
|---|---|
| Kali Linux IP | 192.168.50.10 (labnet, eth0) — **resets on every VM reboot**, must be re-added manually every time |
| Metasploitable2 IP | 192.168.50.20 (labnet, eth0) — **also resets on reboot**, same fix needed |
| Metasploitable2 second adapter | eth1, 192.168.56.20, host-only network, reaches Splunk on Windows host |
| Splunk | Runs on Windows host (not inside any VM), browser-based. Old account expired, all previously ingested data lost. New 60-day trial signed up — Rithika choosing between Splunk Cloud (recommended: faster, no install) vs Splunk Enterprise (already may have installed this — if so, stick with it rather than switching) |
| Network mode | VirtualBox Internal Network, "labnet" — fully isolated, no internet inside either VM |
| Kali OS | Kali Rolling 2026.1, username `kali` |
| Metasploitable2 OS | Ubuntu 8.04 (old — lacks modern commands like `service`; use `/etc/init.d/apache2 start` instead), username `msfadmin` |

**IP fix commands (run after EVERY VM reboot, on the respective machine):**
```
sudo ip addr add 192.168.50.10/24 dev eth0   # on Kali
sudo ip addr add 192.168.50.20/24 dev eth0   # on Metasploitable2
sudo ip link set eth0 up
```

**Known trap — IP CONFLICT:** Kali's IP has previously gotten accidentally set to .20 (same as target) during troubleshooting after a reboot. This causes `ping` to falsely succeed (Kali pinging itself) while `curl`/HTTP fails outright with "0 ms, could not connect." If this happens again: check with `ip addr show | grep 192.168.50` on Kali — if it shows .20, run:
```
sudo ip addr del 192.168.50.20/24 dev eth0
sudo ip addr add 192.168.50.10/24 dev eth0
```

**Clipboard between Windows host and Kali VM:** if copy-paste isn't working, check VirtualBox menu: `Devices → Shared Clipboard → Bidirectional`. If already set and still not working, just type commands manually rather than troubleshooting further — it's a known low-priority friction point.

---

## Scenario 1 — Network Reconnaissance and Port Scanning: ✅ DONE

- Command: `nmap -sT -sV 192.168.50.20`
- Result: 23 open ports out of 1000 scanned, 15.32 seconds, full output saved in repo
- Wireshark validated: SYN packet burst confirmed via filter `ip.addr==192.168.50.20 && tcp.flags.syn==1 && tcp.flags.ack==0`
- **Confirmed exploited: vsftpd 2.3.4 backdoor (CVE-2011-2523)** on port 21. Trigger: FTP login with username containing `:)`, then `nc 192.168.50.20 6200`. Took 4 attempts (3 failed, 1 succeeded) — inconsistent/timing-sensitive trigger. Confirmed root access via `whoami`→root, `hostname`→metasploitable, `id`→uid=0(root).
- Also noted but not tested: port 6667 UnrealIRCd backdoor (CVE-2010-2075), port 1524 pre-existing root bindshell (no auth needed at all)
- Full remediation drafted (3-part: problem/why it matters/fix) for port 21 fix and the undetected-scan finding
- Full evidence written into `scenarios/01-network-reconnaissance/README.md` in the repo — **STATUS: COMPLETE**

## Scenario 3 — Suspicious HTTP Path Enumeration: ✅ DONE

- Command: `gobuster dir -u http://192.168.50.20 -w /usr/share/wordlists/dirb/common.txt`
- Result: 4,613 paths tried, found `phpinfo.php`, `phpMyAdmin`, `twiki`, plus expected 403s on `.htaccess`/`.htpasswd`/etc.
- Wireshark validated: 4,615 displayed HTTP requests matched Gobuster's 4,613 (confirmed via filter `http.request && ip.addr == 192.168.50.20`)
- **phpinfo.php — CONFIRMED information disclosure finding.** Leaked: PHP 5.2.4-2ubuntu5.10, Apache 2.2.8, document root `/var/www/`, config path `/etc/php5/cgi/php.ini`, MySQL client 5.0.51a, Suhosin patch 0.9.6.2. This is the strongest/realest finding in this scenario.
- **phpMyAdmin — CONFIRMED negative.** Tested 4 credential combos (root/blank, root/root, root/msfadmin, msfadmin/msfadmin), all failed with "Access denied."
- **TWiki — CONFIRMED negative.** Verified genuinely 2003-era install (revision r1.20 in page footer). Tested CVE-2004-2649 command injection via `bin/search` with a `echo HELLO_FROM_TWIKI` payload — server returned it as literal text, no execution.
- Full remediation drafted (3-part structure) for all three findings
- Also documented mid-scenario: an IP conflict incident (Kali got assigned .20 by mistake) was diagnosed and fixed — worth keeping in the writeup as a real troubleshooting narrative
- Full evidence written into `scenarios/03-http-path-enumeration/README.md` in the repo — **STATUS: COMPLETE**

## Scenario 2 — Repeated SSH Authentication Failures: 🔶 IN PROGRESS (just resumed)

- Attack already run in a past session: `hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ssh://192.168.50.20` → 1,694 failed login entries in `/var/log/auth.log`
- Filtered log saved as `attack_sample3.log` — **saved locally on Rithika's laptop**, ready to upload
- Old Splunk account expired, all previously ingested data lost. **New 60-day trial just signed up and confirmed ready as of this handoff.**
- SPL query logic known (rex to extract IP, 5-minute bins, threshold >5 failures/IP) but exact query text was never saved — needs to be rebuilt from scratch in the new Splunk instance
- Previously also unresolved: Splunk alert wouldn't save, error "enable at least one action" — needs to be resolved fresh in the new trial
- **Immediate next steps when resuming:**
  1. Confirm both VM IPs are set (192.168.50.10 Kali, 192.168.50.20 Metasploitable2) after reboot
  2. Log into new Splunk instance (Cloud or Enterprise, browser-based on Windows host)
  3. Upload `attack_sample3.log` as a new data source
  4. Set sourcetype to `linux_secure`
  5. Confirm the data landed with a basic search before touching detection logic
  6. Rebuild the SPL query (rex extraction, 5-min bins, >5 threshold) from scratch, test it, screenshot everything this time — no screenshots exist from the original run
  7. Fix the alert-save error if it recurs
  8. Write full Scenario 2 README section (same structure as Scenario 1 and 3) using real evidence

## Scenario 4 — Controlled Unexpected Outbound Connection: ⬜ NOT STARTED

- Template exists in repo, no work done yet
- Needs a decision first: reverse shell trigger (netcat listener on Kali, triggered shell on Metasploitable2 connecting back) vs. simulated periodic beacon — Rithika needs to choose before starting
- Will follow the same evidence pattern: Wireshark capture, confirm via listener output, document, remediate

---

## GitHub Repo

**Status: pushed once already, one commit behind as of this handoff.** Live at:
`https://github.com/RithikaGowdelli/network-threat-simulation-lab`

Local repo on Rithika's Windows laptop: `D:\network-threat-simulation-lab`, connected to the above remote, branch `main`.

**Decision made:** hold off on pushing Scenario 3's update until Scenario 2 and 4 are both done — do ONE final push at the end covering everything, rather than pushing after every scenario. This avoids repeated file-sync friction between Claude's sandbox copy and Rithika's local Windows copy.

**When ready for the final push:** Claude will provide a fresh zip of the complete repo (with full git history) to download, extract over the existing local folder, then:
```
cd D:\network-threat-simulation-lab
git remote add origin https://github.com/RithikaGowdelli/network-threat-simulation-lab.git
git push -u origin main
```
Should be a clean fast-forward push since local commits build directly on what's already on GitHub.

---

## Resume Content (drafted, DO NOT send to employers until Scenario 2 & 4 are done)

Three portfolio project bullet sets were drafted together (in order): 1) SOC Alert Monitoring & Detection Engineering Lab (Splunk/Sysmon), 2) this project (Network Threat Simulation & Incident Validation Lab), 3) Microsoft Sentinel Cloud Threat Hunting Lab. Only project 2's bullets are reproduced here since it's the one being actively worked on:

> Built an isolated cybersecurity lab using Kali Linux, Metasploitable, Splunk, Wireshark, and Nmap to simulate and investigate 4 controlled attack scenarios spanning reconnaissance, authentication attacks, web enumeration, and network-based threat activity.
> Scanned 1,000 TCP ports against an isolated vulnerable target using Nmap, identifying 23 exposed services in 15.32 seconds and mapping attack surface exposure through service enumeration.
> Simulated 1,694 failed SSH authentication events using Hydra to emulate brute-force activity, generating telemetry for SIEM detection development and incident investigation.
> Performed web content enumeration using Gobuster across 4,613 directory paths, cross-validating 4,615 corresponding requests through Wireshark packet analysis to confirm application activity at the network level.
> Correlated TCP/IP, DNS, HTTP, and SSH traffic with Splunk security logs to validate alerts, reconstruct attack timelines, and extract 15+ Indicators of Compromise (IOCs). **(PLACEHOLDER — update after Scenario 2/4 complete, real IOC count not yet confirmed.)**
> Developed 5+ SPL detection searches for reconnaissance and authentication-based attacks, mapped validated activity to MITRE ATT&CK, and produced 4 incident investigation reports with remediation recommendations. **(PLACEHOLDER — update after Scenario 2/4 complete, SPL query count and incident report count not yet real.)**

**Do not send this resume version to any employer until the two placeholder bullets have real numbers.**

---

## Standing Rules for This Project (apply throughout)

- Never invent evidence, metrics, screenshots, or field names — always ask for real command output
- Every scenario needs: objective, prerequisites, procedure, logs/evidence, Wireshark filters, Splunk workflow, SPL queries, investigation questions, IOCs, timeline, true/false positive assessment, MITRE mapping, remediation, screenshot checklist, status
- Remediation recommendations are drafted by Rithika first using a 3-part structure (Problem / Why it matters / Fix), then reviewed/tightened by Claude — never written from scratch by Claude
- Lessons-learned section must be written by Rithika in her own words, not by Claude
- All testing confined to the isolated labnet — no real/public systems ever targeted
- Commit messages should be specific and incremental, matching real work done that session, not generic
- When something is tested and fails (negative result), that's documented as a legitimate finding, not treated as an error to hide or keep retrying indefinitely
- All commands run FROM Kali, aimed AT Metasploitable2 — Metasploitable2 is only logged into directly to check its own internal state (processes, firewall rules, IP config), never to launch attacks against itself

---

## What Happens Next (pick up here)

1. **Currently resuming Scenario 2** — Splunk trial confirmed ready. Do the 8 steps listed under Scenario 2 above.
2. After Scenario 2: do Scenario 4 (decide reverse shell vs. beacon approach first)
3. Do ONE final GitHub push covering Scenario 2 + 4 + any final touches
4. Build the final consolidated document with all screenshots embedded
5. Write lessons-learned section (Rithika's own words — Claude prompts with questions but does not write it)
6. Finalize architecture doc and detection-engineering docs once all scenarios are locked in
7. Update resume bullets with real final numbers once Scenario 2 & 4 are done — remove placeholder markers
