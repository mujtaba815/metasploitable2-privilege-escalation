# Task 05 — Privilege Escalation

**Zyroo Cybersecurity Internship Track — Week 5**
Prepared by: **Syed Mujtaba Hussain**

## Overview

This task picks up from the Week 4 exploitation exercise: a low-privilege (`www-data`) foothold on
**Metasploitable2** (`192.168.43.134`) was re-established, then systematically enumerated for a local
privilege escalation path. Two vectors were investigated — a kernel-level netlink race condition
(CVE-2009-1185), documented but ultimately not successful on this target, and a misconfigured SUID
binary (`/usr/bin/nmap`), which was successfully exploited to obtain full root access.

All exploitation was performed only against the local Metasploitable2 training VM on an isolated lab network.
No real, production, or third-party systems were targeted, in line with the internship's authorized-testing
policy.

## Escalation Chain

| | |
|---|---|
| **Initial Access** | CVE-2012-1823 (PHP-CGI Argument Injection) — re-established from Week 4 |
| **Access After Initial Exploit** | User-level (`www-data`) |
| **Vector Investigated (not used)** | CVE-2009-1185 — udev netlink local privilege escalation (kernel race condition) |
| **Vector Exploited** | Misconfigured SUID binary — `/usr/bin/nmap` 4.53, `--interactive` mode shell escape |
| **Commands Used** | `nmap --interactive` → `!sh` |
| **Access After Escalation** | Full root (`euid=0`) |

## Contents

| File | Description |
|---|---|
| `Privilege_Escalation_Report_Zyroo_Task05.pdf` | Full report: initial access recap, enumeration, vector investigated but ruled out, vector exploited with reasoning, before/after evidence, impact, mitigation |
| `Week4_Exploitation_Report_reference.pdf` | The Week 4 report this task builds on, included for traceability |
| `screenshots/` | Raw evidence screenshots referenced in the report |

## Screenshots

| # | File | Evidence of |
|---|---|---|
| 1 | `01-initial-access-module-search.png` | Locating the Week 4 exploit module via `search cgi_arg_injection` |
| 2 | `02-initial-access-module-configured.png` | Module configured (RHOSTS, TARGETURI, LHOST, LPORT) |
| 3 | `03-initial-access-session-established.png` | Meterpreter session established — `sysinfo` + `getuid` confirming `www-data` |
| 4 | `04-initial-access-www-data-confirmed.png` | `whoami`, `id`, `uname -a`, `pwd` confirming initial access level |
| 5 | `05-enum-crontab.png` | Enumeration — `/etc/crontab`, ruling out writable cron jobs as a vector |
| 6 | `06-enum-suid-binaries.png` | Enumeration — full SUID binary listing (`find / -perm -4000`), identifying `/usr/bin/nmap` |
| 7 | `07-nmap-version-suid-confirmed.png` | Vector identification — Nmap 4.53, SUID bit confirmed (`-rwsr-xr-x`, root-owned) |
| 8 | `08-nmap-interactive-root-shell.png` | Successful escalation — `nmap --interactive` → `!sh` → `whoami`/`id` confirming root |
| 9 | `09-udev-download.png` | Additional vector investigated — downloading the CVE-2009-1185 public PoC |
| 10 | `10-udev-source-code.png` | Confirming the retrieved source matches CVE-2009-1185 |
| 11 | `11-udev-upload-compile.png` | Uploading and compiling the PoC on target; locating the udevd netlink PID |
| 12 | `12-udev-pid-verification-attempt.png` | Cross-verifying the netlink PID via `/proc/net/netlink`, running the exploit attempt |

## Tools Used

- **Metasploit Framework** (`msfconsole`) — initial access, enumeration module
- **Manual Linux enumeration** — `find`, `cat`, `ls`, `uname`, `ps aux` for privilege escalation surface discovery
- **Nmap** — both as a target of the SUID vulnerability and a reference for the vulnerable version
- **Metasploitable2** — intentionally vulnerable lab target
- **Kali Linux (VMware)** — attacker machine, isolated local lab environment

## Rules of Engagement

No files were modified, deleted, or exfiltrated during this exercise beyond what was strictly necessary to
demonstrate the escalation path (creation of a required empty `/tmp/run` directory for the investigated-but-unused
udev vector). Enumeration and escalation were limited to what the task's own guidance explicitly permitted:
identifying and exploiting a privilege escalation vector, then confirming elevated access via `whoami`/`id`.

## Disclaimer

This repository documents authorized, controlled penetration testing performed entirely within a private lab
environment as part of a structured cybersecurity training exercise. None of the techniques described here were
used against any system without explicit authorization.
