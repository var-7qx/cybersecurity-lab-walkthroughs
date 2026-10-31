# Linux Commands Reference & Auditing Guide

A practical command reference for Linux system enumeration, network inspection, process analysis, and privilege auditing during authorized security assessments.

---

## 1. System Architecture & Kernel Enumeration

Identify the operating system, distribution version, kernel architecture, and environment parameters.

```bash
# Display system architecture and kernel release
uname -a

# Display OS and distribution information
cat /etc/os-release
cat /etc/issue

# Display hostname and system details
hostnamectl

# Inspect CPU architecture and core information
lscpu

# Inspect environment variables
env
printenv

# View mounted filesystems and disk usage
df -h
lsblk
mount
```

---

## 2. User, Group & Identity Assessment

Determine the current user context, group memberships, sudo permissions, and logged-in sessions.

```bash
# Display user ID, group ID, and group memberships
id

# Display the effective username
whoami

# List usernames configured on the system
cut -d: -f1 /etc/passwd

# Inspect account records and login shells
cat /etc/passwd

# Inspect group definitions
cat /etc/group

# Review sudo permissions for the current user
sudo -l

# Display logged-in users
who
w

# Review recent login records
last -n 10
lastlog
```

---

## 3. File System Discovery & Permission Auditing

Inspect file permissions, SUID/SGID binaries, writable locations, recent modifications, and Linux capabilities.

```bash
# Find files with the SUID bit set
find / -type f -perm -4000 -exec ls -la {} + 2>/dev/null

# Find files with the SGID bit set
find / -type f -perm -2000 -exec ls -la {} + 2>/dev/null

# Find directories writable by everyone
find / -type d -perm -0002 2>/dev/null

# Find directories writable by the current user
find / -type d -writable 2>/dev/null

# Find files writable by everyone
find / -type f -perm -0002 2>/dev/null

# Find files modified within the last 24 hours
find / -type f -mtime -1 2>/dev/null

# Search /etc for configuration entries containing sensitive keywords
grep -rnwi -e 'password' -e 'secret' /etc/ 2>/dev/null

# Inspect Linux file capabilities
getcap -r / 2>/dev/null

# Locate a binary
command -v BINARY_NAME
whereis BINARY_NAME
```

**Note:** SUID/SGID permissions and capabilities are not automatically vulnerabilities. Review their context and whether they create an exploitable security weakness.

---

## 4. Network Configuration & Socket Auditing

Inspect network interfaces, routing tables, listening services, and local network information.

```bash
# Display network interfaces and IP addresses
ip addr show

# Alternative interface command, if installed
ifconfig -a

# Display routing table and default gateway
ip route show

# Alternative routing commands, if installed
route -n
netstat -rn

# Display listening TCP/UDP sockets and process information
ss -tulpn

# Legacy alternative, if installed
netstat -tulpn

# Inspect DNS resolver configuration
cat /etc/resolv.conf

# Inspect local hostname mappings
cat /etc/hosts

# Display neighbor cache
ip neigh

# Legacy alternative, if installed
arp -e
```

---

## 5. Process & Service Management

Inspect running processes, system services, and scheduled tasks.

```bash
# List processes with full command lines
ps aux
ps -ef

# Display the process hierarchy
pstree -p

# Show a snapshot of processes and resource usage
top -b -n 1 | head -n 25

# List running systemd services
systemctl list-units --type=service --state=running

# Inspect a service's effective configuration
systemctl cat SERVICE_NAME.service

# Review recent logs for a service
journalctl -u SERVICE_NAME.service -n 50 --no-pager

# List the current user's scheduled cron jobs
crontab -l

# Inspect system-wide cron configuration
cat /etc/crontab
ls -la /etc/cron.*

# Inspect a common Debian/Ubuntu cron spool directory
ls -la /var/spool/cron/crontabs/ 2>/dev/null
```

---

## 6. Network Interaction & Data Transfer

Utilities for connectivity checks, HTTP requests, and authorized file transfers.

```bash
# Test ICMP reachability
ping -c 4 TARGET_HOST

# Test TCP connectivity to a specified port
nc -zv TARGET_HOST PORT

# Start a local Netcat listener for an authorized lab
nc -lvnp PORT

# Download a file while preserving its filename
curl -O 'https://example.com/file'

# Send an HTTP POST request with JSON data
curl -X POST 'https://example.com/api' \
  -H 'Content-Type: application/json' \
  -d '{"param":"value"}'

# Download a resource and specify the output filename
wget 'https://example.com/file' -O OUTPUT_FILENAME

# Copy a local file to an authorized remote system
scp LOCAL_FILE USERNAME@REMOTE_HOST:/path/to/destination/

# Start a simple HTTP server in the current directory
python3 -m http.server PORT
```

**Security note:** A Python HTTP server exposes files from its serving directory to reachable clients. Use a dedicated test directory and stop the server when finished. Only transfer files between systems you are authorized to access.

---

## 7. Archive, Compression & Extraction

Create archives and inspect compressed files.

```bash
# Create a gzip-compressed tar archive
tar -czvf ARCHIVE_NAME.tar.gz /path/to/directory/

# Extract a tar.gz archive
tar -xzvf ARCHIVE_NAME.tar.gz

# Extract a ZIP archive to a destination directory
unzip ARCHIVE_NAME.zip -d /path/to/destination/

# Compress a file while retaining the original
gzip -k FILE_NAME

# Preview the first 20 lines of a gzip-compressed text file
zcat FILE_NAME.gz | head -n 20
```

---

## 8. Text Processing & Stream Editing

Filter command output, analyze logs, extract fields, and transform text.

```bash
# Match a pattern without case sensitivity
grep -i 'PATTERN' FILE_NAME

# Display lines that do not match a pattern
grep -v 'PATTERN' FILE_NAME

# Print the first and third whitespace-separated fields
awk '{print $1, $3}' FILE_NAME

# Extract the username and UID fields from /etc/passwd
cut -d: -f1,3 /etc/passwd

# Sort lines and remove adjacent duplicates
sort FILE_NAME | uniq

# Count occurrences of each line
sort FILE_NAME | uniq -c | sort -nr

# Replace text throughout each matching line
sed 's/TARGET_STRING/REPLACEMENT_STRING/g' FILE_NAME
```

---

## Usage Notes

- Commands may require elevated privileges to return complete results.
- Some utilities, such as `ifconfig`, `netstat`, `arp`, and `pstree`, may not be installed by default.
- Replace placeholders such as `TARGET_HOST`, `PORT`, and `FILE_NAME` with appropriate values.
- Review command output before drawing conclusions; a missing result does not necessarily prove that a system is secure.
- Run enumeration and network tests only on systems you own or are explicitly authorized to assess.
