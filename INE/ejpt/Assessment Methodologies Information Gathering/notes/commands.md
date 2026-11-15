# Command Reference — Assessment Methodologies: Information Gathering CTF 1

A concise technical reference of all verified commands executed during the **INE Assessment Methodologies: Information Gathering CTF 1** laboratory engagement.

---

## 1. Network & Service Fingerprinting

### Full TCP Port Scan & Service Versioning
```bash
nmap -sV -sC -p- target.ine.local
```
- `-sV`: Probes open ports to determine service and version info.
- `-sC`: Executes default safe Nmap NSE scripts (discovering web server headers, robots.txt, generator tags).
- `-p-`: Scans all 65,535 TCP ports to ensure no non-standard listening services are missed.
- `target.ine.local`: Hostname resolved to `192.144.172.3`.

---

## 2. Web Content & Directory Enumeration

### Gobuster Directory Enumeration (Large Wordlist)
```bash
gobuster dir -u http://target.ine.local -w /usr/share/seclists/Discovery/Web-Content/big.txt
```
- `dir`: Selects directory and file enumeration mode.
- `-u http://target.ine.local`: Base URL of the target web application.
- `-w /usr/share/seclists/Discovery/Web-Content/big.txt`: Path to the SecLists `big.txt` dictionary (20,477 words).

### Dirb Recursive Directory Scanning
```bash
dirb http://target.ine.local
```
- Executes a recursive dictionary attack using the default `/usr/share/dirb/wordlists/common.txt` wordlist (4,612 words).
- Discovered that `http://target.ine.local/wp-content/uploads/` is listable (`(!) WARNING: Directory IS LISTABLE`).

---

## 3. Targeted Backup File Discovery

### Gobuster Extension Scanning for Backups
```bash
gobuster dir -u http://target.ine.local -w /usr/share/seclists/Discovery/Web-Content/common.txt -x bak
```
- `-x bak`: Appends the `.bak` file extension to each wordlist item to search for uncompiled backup files.
- Discovered `/wp-config.bak` returning `HTTP 200 OK` (Size: 3438 bytes).

---

## 4. Resource Inspection & Content Retrieval

### Retrieving Backup Configuration File via cURL
```bash
curl http://target.ine.local/wp-config.bak
```
- Performs a direct HTTP GET request to download the raw unexecuted PHP backup file.
- Disclosed authentication keys, salts, and `FLAG4`.

---

## 5. Website Mirroring & Offline Inspection

### Full Website Mirroring via HTTrack
```bash
httrack http://target.ine.local -O target.ine.local
```
- `http://target.ine.local`: Root URL of the site to crawl and mirror.
- `-O target.ine.local`: Output directory to store the mirrored filesystem hierarchy.

### Offline Inspection of Mirrored Files
```bash
cd ~/target.ine.local/target.ine.local
cat xmlrpc0db0.php
```
- Navigates into the mirrored directory structure.
- Displays the contents of the Really Simple Discovery (RSD) XML document, uncovering `FLAG5`.

