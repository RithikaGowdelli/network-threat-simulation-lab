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

```
[PLACEHOLDER — build and test this against your real ingested access log, then paste the actual working query here. I'll review it once you have real field names from your log format — Apache combined log format field extraction in Splunk depends on your actual source type configuration]
```

## Investigation Questions

- What is the request rate (paths per minute/second) and does it exceed plausible human browsing speed?
- What proportion of requests returned 404 versus 200? A high 404 rate is a strong enumeration signal.
- Are the requested paths drawn from a known wordlist pattern (e.g. common.txt), which would confirm tool use over manual browsing?
- Did the enumeration surface any real hidden path that a legitimate user should not have been able to find?

## Indicators of Compromise

| Type | Value | Notes |
|---|---|---|
| Source IP | [PLACEHOLDER] | Kali attacker IP |
| User-Agent | [PLACEHOLDER] | Enumeration tools often use a distinctive or default User-Agent string |
| Request rate | [PLACEHOLDER] | Paths per minute during the attack window |
| Paths discovered | [PLACEHOLDER] | Any sensitive paths found |

## Timeline

| Time (UTC) | Event |
|---|---|
| [PLACEHOLDER] | Enumeration started |
| [PLACEHOLDER] | Threshold breach detected in Splunk |
| [PLACEHOLDER] | Enumeration ended |

## True Positive / False Positive Assessment

[PLACEHOLDER — true positive by design. Note what a false positive would look like here, e.g. a legitimate site crawler like Googlebot, or a broken web app generating repeated 404s from normal navigation.]

## MITRE ATT&CK Mapping

| Technique ID | Name | Justification |
|---|---|---|
| T1595.003 | Active Scanning: Wordlist Scanning | Enumeration tool systematically requests paths from a wordlist |
| T1190 | Exploit Public-Facing Application | Relevant only if enumeration led to exploitation of a discovered path — confirm before including |

## Remediation Recommendations

- [PLACEHOLDER — draft yourself: rate limiting, WAF rules, disabling directory listing, alerting on 404 spikes, custom error pages that don't leak information]

## Screenshot Checklist

- [ ] Enumeration tool terminal output with paths and response codes
- [ ] Splunk search showing the request rate spike
- [ ] Wireshark capture showing HTTP request pattern
- [ ] Access log excerpt (sanitized)

## Status

[PLACEHOLDER — Not started / In progress / Complete]
