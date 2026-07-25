# Evidence Index

All screenshots taken during this project, organized by scenario. Filenames are numbered in chronological order within each folder. Files marked **KEY** are the most important pieces of evidence — the ones that prove the core finding.

**Note on accuracy:** these files were organized and labeled based on conversation context, not verified frame-by-frame against every image. If a label looks wrong when you open the file, trust the image over the label and correct the filename — flag it back to Claude if using AI help to fix, so the description stays accurate for anyone reviewing the repo later.

## /evidence/scenario-01/ (Network Reconnaissance)
1. Kali IP check
2. Nmap scan output (part 1)
3. Nmap scan output (part 2)
4. Wireshark SYN packet filter, validating the scan
5. vsftpd backdoor test — first attempt, run from wrong machine (Metasploitable2 instead of Kali), corrected afterward
6. vsftpd backdoor test — corrected, run from Kali
7-9. vsftpd backdoor — port 6200 connection attempts 1 through 3, all refused
10-12. Terminal troubleshooting while testing the backdoor (stuck sessions, clearing terminals)
13. **KEY** — vsftpd exploit succeeded: `whoami` returns `root`
14. **KEY** — vsftpd exploit confirmed: `hostname` and `id` confirm root access on the actual target

## /evidence/scenario-03/ (HTTP Path Enumeration)
1-2. Gobuster scan output (first run)
3. Wireshark sample of captured HTTP requests
4. Wireshark single-path verification (`/test2`)
5. Wireshark request/response filter troubleshooting
6. **KEY** — Wireshark confirms 4,615 requests, validating Gobuster's 4,613 paths
7. Gobuster rerun (after IP conflict was fixed) confirming identical results
8-9. TWiki `bin/view` pages, confirming 2003-era revision
10. TWiki search page, partial view
11. **KEY** — TWiki command injection test result: payload returned as literal text, confirmed negative
12. IP recheck at start of second session
13. phpMyAdmin header check, confirming reachability before credential testing

## /evidence/troubleshooting/ (IP Conflict Diagnosis — spans Scenario 1 to 3)
1-3. Confirming Metasploitable2's IP and Apache service status
4-11. Working through why curl failed despite ping succeeding (firewall checks, ARP flush attempt, netstat)
12-13. State after a clean VM reboot — Kali's IP missing, Metasploitable2's Apache confirmed still running
14. **KEY** — root cause found: Kali was accidentally assigned the same IP as the target (192.168.50.20)
15. Failed first attempt to remove the wrong IP (typo in command)
16. **KEY** — IP conflict fixed, Kali correctly shows 192.168.50.10
17. **KEY** — connectivity restored, real landing page HTML returned from the target
18. VirtualBox shared clipboard setting (unrelated side issue, resolved by typing manually)

## /evidence/github-setup/
1. GitHub "Create a new repository" page
2. Git push error (branch name mismatch, `main` vs `master`)
3. **KEY** — successful git push confirmed, repo live on GitHub

## /evidence/splunk-setup/ (Scenario 2 prerequisite — IN PROGRESS, not yet resolved)
1. Windows File Explorer showing Splunk installer location (D: drive — later confirmed to be a red herring, actual install is on C:)
2-3. Diagnosing the real Splunk install path (`C:\Program Files\Splunk`)
4-11. Password reset process: stopping Splunk, editing `user-seed.conf`, restarting
12. **UNRESOLVED** — Splunk trial license confirmed expired (Jul 5, 2026), blocking data upload for Scenario 2. Next step: full uninstall and clean reinstall.
