# Scenario 3: Suspicious HTTP Path Enumeration

## Objective

Simulate an attacker enumerating web application paths and directories on Metasploitable2's web service, detect the pattern in Splunk from web server logs, validate at the packet level in Wireshark, and map the finding to MITRE ATT&CK.

## Lab Prerequisites

- Kali Linux and Metasploitable2 on isolated labnet
- A directory/path enumeration tool installed on Kali (e.g. Gobuster, Dirb, or ffuf — pick one and confirm which)
- Metasploitable2 web service (Apache/DVWA/whatever is running on it) confirmed reachable
- Web server access logs available for ingestion into Splunk

## Controlled Test Procedure

**Note on network troubleshooting:** during this scenario, an IP conflict occurred after VM reboot — Kali was accidentally assigned 192.168.50.20 (the same IP as the target) instead of its correct 192.168.50.10. This caused ping to falsely appear successful (Kali was pinging itself) while curl/HTTP failed outright. Diagnosed via `ip addr show | grep 192.168.50` and fixed with `ip addr del`/`ip addr add`. Documenting this because IP conflicts on reboot are a realistic, recurring lab issue worth showing you can diagnose.

1. Wireshark capture started on eth0, no filter.
2. Gobuster directory enumeration run from Kali against the target's web server:
```
gobuster dir -u http://192.168.50.20 -w /usr/share/wordlists/dirb/common.txt
```
3. Capture stopped after scan completion, saved as `scenario3-httpenum-v2.pcapng`.

**Full command output:**

```
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)

[+] Url:                     http://192.168.50.20
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s

Starting gobuster in directory enumeration mode
.htaccess            (Status: 403) [Size: 295]
.htpasswd            (Status: 403) [Size: 295]
cgi-bin/             (Status: 403) [Size: 294]
.hta                 (Status: 403) [Size: 290]
dav                  (Status: 301) [Size: 317] [--> http://192.168.50.20/dav/]
index.php            (Status: 200) [Size: 891]
index                (Status: 200) [Size: 891]
phpinfo.php          (Status: 200) [Size: 48011]
phpinfo              (Status: 200) [Size: 47999]
phpMyAdmin           (Status: 301) [Size: 324] [--> http://192.168.50.20/phpMyAdmin/]
server-status         (Status: 403) [Size: 299]
test                 (Status: 301) [Size: 318] [--> http://192.168.50.20/test/]
twiki                (Status: 301) [Size: 319] [--> http://192.168.50.20/twiki/]
Progress: 4613 / 4613 (100.00%)

Finished
```

## Wireshark Validation

Filter applied: `http.request && ip.addr == 192.168.50.20`
Result: **4,615 displayed HTTP requests**, matching Gobuster's 4,613 wordlist entries (small overhead from initial handshake/setup traffic). Confirmed no response packets were counted by checking individual paths, e.g. `http.request && http.request.uri == "/test2"` returned exactly 1 request. Independent packet-level confirmation that the enumeration occurred as reported.

## phpinfo.php Investigation — Information Disclosure

No exploitation required — this page is publicly viewable by design and dumps the server's full PHP configuration.

```
curl http://192.168.50.20/phpinfo.php
```

**Key leaked information:**

| Detail | Value | Why it matters |
|---|---|---|
| PHP version | 5.2.4-2ubuntu5.10 | Over 15 years out of date; multiple known CVEs exist for this version line |
| Web server software | Apache/2.2.8 (Ubuntu) DAV/2 | Independently confirms the Nmap fingerprint from Scenario 1, this time straight from the server itself |
| Document root | `/var/www/` | Confirms exact filesystem location of the web application, useful if a later vulnerability allows file read/write |
| PHP config path | `/etc/php5/cgi/php.ini` | Exact configuration file location, useful for an attacker who gains any file access |
| Script path | `/var/www/phpinfo.php` | Full absolute path to this specific file |
| MySQL client library | 5.0.51a | Tells an attacker exactly which MySQL-side vulnerabilities might apply |
| Suhosin patch version | 0.9.6.2 | A hardening patch is present, but an outdated one — worth noting as partial mitigation, not full protection |

**Assessment:** this is a textbook information disclosure finding — no code execution, but a significant reconnaissance gift to an attacker, handing over exact software versions and file paths that would otherwise require guesswork or additional probing.

## phpMyAdmin Investigation — Default Credential Testing

Confirmed reachable via header check:
```
curl -I http://192.168.50.20/phpMyAdmin/
```
Returned HTTP 200, valid phpMyAdmin session cookies, and a `Last-Modified: Tue, 09 Dec 2008` header confirming this install is contemporaneous with the rest of the outdated software on this box.

**Credential testing** — attempted the following common Metasploitable2 default/misconfiguration combinations via the login form:
- `root` / *(blank password)*
- `root` / `root`
- `root` / `msfadmin`
- `msfadmin` / `msfadmin`

**Result:** all four attempts returned "Access denied." Confirmed negative finding — this specific misconfiguration (blank or trivially guessable MySQL root credentials) is not present on this build, despite the outdated software version. Note this does not rule out other credential pairs or brute-force feasibility; only the commonly documented defaults were tested here, consistent with the scope of this scenario (enumeration and identification, not exhaustive credential attacks — that's covered separately in Scenario 2's authentication testing).

## TWiki Investigation (CVE-2004-2649)

Confirmed the TWiki install is genuinely outdated: the WebHome page footer shows "Revision r1.20 - 02 Feb 2003," confirming a build from 2003.

**Command injection test** — attempted to break out of the `search` parameter in `bin/search` to execute an arbitrary shell command:
```
curl "http://192.168.50.20/twiki/bin/search/Main/?scope=text;regex=on;search=%27%3B\`echo%20+%20HELLO_FROM_TWIKI\`%3B%27"
```

**Result:** the payload was returned as literal, unexecuted search text:
```
Search: ';`echo   HELLO_FROM_TWIKI`;'
```
No command execution occurred. This is a confirmed negative result — the injection point tested is not exploitable on this installation, despite the underlying software being confirmed outdated (2003-era) via the revision metadata. This does not rule out other TWiki vulnerabilities from the same era, only this specific injection path.

## Wireshark Analysis Filters

- `http.request && ip.addr == [TARGET_IP]` — isolate HTTP requests to the target
- `http.response.code == 404` — isolate not-found responses, useful for spotting enumeration against nonexistent paths
- `tcp.stream eq [N]` — follow a specific request/response pair for detail

## Splunk Investigation Workflow

1. Ingest the Apache access log into Splunk.
2. Search for requests from the attacker IP within the test window.
3. Count unique URI paths requested per minute — a high rate of unique paths from one source in a short window is the enumeration signature.
4. Flag any source IP exceeding a defined unique-path-per-minute threshold.

## Working SPL Queries

Not applicable to this scenario. No Apache access log was ingested into Splunk — detection and validation relied on Gobuster's own output (4,613 paths tested) cross-validated against Wireshark's HTTP request count (4,615 displayed requests). This is a deliberate scope decision: the Splunk/SIEM detection skillset is demonstrated in Scenarios 2 and 4 instead.

## Investigation Questions

- What is the request rate (paths per minute/second) and does it exceed plausible human browsing speed?
- What proportion of requests returned 404 versus 200? A high 404 rate is a strong enumeration signal.
- Are the requested paths drawn from a known wordlist pattern (e.g. common.txt), which would confirm tool use over manual browsing?
- Did the enumeration surface any real hidden path that a legitimate user should not have been able to find?

## Indicators of Compromise

| Type | Value | Notes |
|---|---|---|
| Source IP | 192.168.50.10 | Kali attacker IP |
| Request rate | 4,613 requests in Gobuster's run, ~10 concurrent threads | Confirmed via 4,615 displayed HTTP requests in Wireshark |
| Paths discovered | phpinfo.php, phpMyAdmin, twiki | phpinfo.php confirmed a real information disclosure finding; phpMyAdmin and twiki tested with negative results (see below) |

## Timeline

| Event | Notes |
|---|---|
| IP conflict discovered and resolved | Kali was misconfigured to 192.168.50.20 (same as target) after a VM reboot; fixed before this scan could proceed |
| Gobuster enumeration run | 4,613 paths tested, completed to 100% |
| Wireshark validation | 4,615 requests confirmed via display filter |
| phpinfo.php reviewed | Confirmed information disclosure |
| phpMyAdmin credential testing | 4 attempts, all failed (negative finding) |
| TWiki command injection testing | 1 payload tested, failed (negative finding) |

*(Precise timestamps not captured this session — add exact times in a future pass if needed for the consolidated incident report.)*

## True Positive / False Positive Assessment

True positive by design — this was a deliberate, controlled enumeration run by the lab operator. In a real environment, this would be distinguished from a false positive (e.g. a legitimate search engine crawler like Googlebot, or a broken web application generating repeated 404s from normal navigation) by checking the User-Agent string and whether the request rate/pattern matches known crawler behavior versus a wordlist-driven tool.

## MITRE ATT&CK Mapping

| Technique ID | Name | Justification |
|---|---|---|
| T1595.003 | Active Scanning: Wordlist Scanning | Gobuster systematically requested 4,613 paths from a wordlist against the target web server |
| T1592 | Gather Victim Host Information | phpinfo.php disclosure provided detailed software version and file path information without any exploitation required |

## Remediation Recommendations

**Finding 1 — phpinfo.php publicly accessible, leaking server details:**
- Problem: phpinfo.php is publicly accessible with no authentication required, and it exposes the exact PHP version, Apache version, internal file paths (document root, config file location), and MySQL client library version.
- Why it matters: an attacker doesn't need to guess or scan for this information — it's handed to them directly, in one request. That shortens the time it takes to find a matching known vulnerability for this specific software stack, and the exposed file paths help if the attacker later finds any way to read or write files on the server.
- Fix: remove phpinfo.php entirely from the production web server — it should only ever exist temporarily during debugging, never left in a live environment. If similar diagnostic info is ever needed, restrict it to internal-only access via firewall rule or IP allowlist, never publicly reachable.

**Finding 2 — phpMyAdmin login exists, default credentials failed but remains internet-facing:**
- Problem: the phpMyAdmin login page is publicly reachable, even though the four default/common credential combinations tested didn't succeed.
- Why it matters: because it's internet-facing, it remains a viable target for a sustained brute-force attack using a larger password list, and successful login would grant full database access — including the ability to read, modify, or delete any data on the server.
- Fix: restrict phpMyAdmin to internal-only access via a firewall rule allowing only trusted/known IP addresses, and additionally enable account lockout after a small number of failed attempts to blunt brute-force attempts even from an allowed source.

**Finding 3 — TWiki is a confirmed 2003-era, unpatched installation:**
- Problem: TWiki is confirmed to be a 2003-era installation (revision r1.20), and while the specific command injection attempt tested (CVE-2004-2649) did not succeed, running software this outdated means other known vulnerabilities from that era likely still apply.
- Why it matters: an unpatched, decade-plus-old application is a standing risk even without one specific confirmed exploit — new CVEs against old software don't require a fresh attack to become dangerous, they just require someone to look them up.
- Fix: if TWiki isn't actually needed, remove it entirely. If it's required, upgrade to a current, supported version, and in the meantime restrict access via firewall rule to reduce exposure while the upgrade is scheduled.

## Screenshot Checklist

- [x] Enumeration tool terminal output with paths and response codes
- [x] Wireshark capture showing HTTP request pattern and count validation
- [x] phpinfo.php full output (PHP version, paths, server signature)
- [x] phpMyAdmin login attempts (all 4 credential combinations)
- [x] TWiki command injection test and literal-text result

## Status

Complete — evidence collected and validated, all three findings tested (1 confirmed positive, 2 confirmed negative), remediation drafted for all three.
