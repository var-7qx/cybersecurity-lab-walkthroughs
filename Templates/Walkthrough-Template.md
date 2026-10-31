Lab Walkthrough Template: [Lab / Hostname / Challenge Title]
> [!NOTE]
> **Template Usage Instructions**
>
> This document provides a standardized, reusable structure for documenting laboratory exercises completed across authorized platforms (**Pentest Garage**, **INE Security**).
> - All commands, outputs, and findings in this template are **placeholders** (`[...]`) and must be replaced with your actual engagement data.
> - Ensure all evidence, screenshots, and terminal outputs are strictly **sanitized** (redact flags, tokens, proprietary material, and sensitive IPs) before committing.
---
1. Metadata & Engagement Profile
Attribute	Specification
Lab / Challenge Title	`[Insert Lab Name / Target Hostname]`
Platform / Provider	`[Pentest Garage / INE Security / Other]`
Course / Track	`[e.g., eJPT / eCPPT / Web Security / Network Exploitation]`
Difficulty Rating	`[Easy / Medium / Hard / Insane]`
Primary Category	`[e.g., Web Application / Network Service / Active Directory / PrivEsc]`
Date Completed	`[YYYY-MM-DD]`
Tools Utilized	`[e.g., Nmap, Burp Suite, Gobuster, Netcat, LinPEAS]`
---
2. Objective & Scenario Overview
2.1 Primary Objective
[State the authorized objective, e.g., identify unauthorized access vectors, evaluate configuration weaknesses, or demonstrate privilege escalation paths.]
2.2 Scenario & Architecture Summary
[Provide a high-level summary of the target environment, network architecture, or simulated organization.]
---
3. Authorization & Scope Verification
> [!IMPORTANT]
> **Scope Verification Checklist**
>
> Before executing any assessment tools, verify that testing activities conform strictly to platform terms:
> - [ ] Target confirmed within active, assigned lab environment boundaries.
> - [ ] Out-of-scope gateway routers, shared platform infrastructure, and external hosts excluded.
> - [ ] Platform rules of engagement confirmed (DoS and brute force limits adhered to).
> - [ ] Academic integrity check: No course exam solutions, proprietary manuals, or raw flags will be disclosed.
Scope Parameter	Specification
Authorized Target(s)	`[Target IP / CIDR / Domain]`
Excluded Assets / Ranges	`[Gateway IPs, platform hosting infrastructure, out-of-scope subnets]`
Testing Window	`[Designated lab session timeframe]`
Permissible Techniques	`[e.g., Service probing, banner grabbing, vulnerability validation]`
Prohibited Actions	`[e.g., Denial of Service (DoS), automated infrastructure brute-forcing]`
---
4. Reconnaissance
4.1 Host Discovery
Document active host discovery techniques used to locate target systems on the authorized subnet.
```bash

# [PLACEHOLDER - EXAMPLE SYNTAX: Discover active hosts on authorized subnet]

nmap -sn <target_subnet_placeholder>

```
Observed Active Hosts:
  - `[Target IP / Hostname]` - `[Host status and initial reachability notes]`
4.2 Port Scanning & Service Identification
Perform initial TCP/UDP port mapping and service interrogation.
```bash

# [PLACEHOLDER - EXAMPLE SYNTAX: Initial port scan with version detection and default safe scripts]

nmap -sC -sV -p- -oN nmap_initial.txt <target_ip_placeholder>

```
Discovered Port & Service Matrix:
Port / Transport	Service	Version / Banner	Initial Observations
`[Port/Protocol]`	`[Service Name]`	`[Version String]`	`[Initial observations and potential attack vectors]`
`[Port/Protocol]`	`[Service Name]`	`[Version String]`	`[Initial observations and potential attack vectors]`
---
5. Enumeration & Attack Surface Mapping
5.1 Service-Specific Enumeration
Perform detailed interrogation of discovered listening services.
Service Investigation: [e.g., Web Application - HTTP/HTTPS]
Technology Stack: `[Web server, backend language, framework, database]`
Content & Directory Discovery:
  ```bash
  # [PLACEHOLDER - EXAMPLE SYNTAX: Directory and endpoint discovery]
  gobuster dir -u http://<target_ip_placeholder>/ -w <wordlist_path_placeholder> -x php,html,txt
  ```
Identified Endpoints & Parameters:
  - `/[endpoint_path]` - `[Purpose, parameter notes, or access restrictions]`
  - `/[admin_endpoint]` - `[Authentication requirements and observations]`
Service Investigation: [e.g., SMB / File Sharing]
Share Enumeration:
  ```bash
  # [PLACEHOLDER - EXAMPLE SYNTAX: Enumerate shares via null / guest session]
  smbclient -L //<target_ip_placeholder>/ -N
  ```
Discovered Shares & Permissions:
  - `[ShareName]` - `[Read / Write / Anonymous access status]`
---
6. Vulnerability Findings
6.1 Vulnerability Overview Matrix
#	Vulnerability Finding	Severity	CWE / CVE Identifier	Affected Component
1	`[Finding Title / Vulnerability Name]`	`[Low / Medium / High / Critical]`	`[CWE-XXX / CVE-YYYY-XXXX]`	`[Service / Endpoint / Binary]`
2	`[Finding Title / Vulnerability Name]`	`[Low / Medium / High / Critical]`	`[CWE-XXX / CVE-YYYY-XXXX]`	`[Service / Endpoint / Binary]`
6.2 Detailed Finding Analysis
Document root causes and technical implications for each identified weakness.
Finding 1: [Finding Title / Vulnerability Name]
Severity Rating: `[Low / Medium / High / Critical]`
Vulnerability Classification: `[CWE / CVE identifier]`
Root Cause: [Explain why this weakness exists, e.g., missing parameter validation, outdated library, improper access controls.]
Technical Impact: [Describe what an attacker can achieve through exploitation.]
---
7. Exploitation & Verified Commands
Document the exact, verified sequence of commands and technical steps used to validate identified vulnerabilities.
7.1 Initial Foothold
Vulnerability Leveraged: `[Reference Finding from Section 6]`
Technical Rationale: [Explain the rationale for this approach based on enumeration results.]
Execution Sequence:
```bash

# [PLACEHOLDER - EXAMPLE SYNTAX: Step 1 - Exploit preparation or staging]

<command_or_script_preparation>

# [PLACEHOLDER - EXAMPLE SYNTAX: Step 2 - Set up local listener if applicable]

nc -lvnp <port_placeholder>

# [PLACEHOLDER - EXAMPLE SYNTAX: Step 3 - Trigger proof-of-concept execution]

curl -X POST http://<target_ip_placeholder>/<endpoint> -d "<payload_data>"

```
7.2 Privilege Escalation (Post-Exploitation)
Compromised Context:
  - User ID / Group ID: `[Initial user account / group context]`
  - Operating System / Kernel: `[OS version and architecture details]`
Local Weakness Identified: `[e.g., Misconfigured sudo rule, SUID binary, unquoted service path]`
Escalation Execution Sequence:
```bash

# [PLACEHOLDER - EXAMPLE SYNTAX: Step 1 - Verify local privileges]

sudo -l

# [PLACEHOLDER - EXAMPLE SYNTAX: Step 2 - Execute escalation command]

<verified_escalation_command>

# [PLACEHOLDER - EXAMPLE SYNTAX: Step 3 - Confirm elevated context]

id

```
---
8. Evidence & Proof of Access
> [!WARNING]
> **Data Sanitization & Content Warning**
>
> Prior to committing or publishing this writeup, perform thorough data sanitization:
> - **Redact Lab Secrets**: Obscure any platform CTF flags, tokens, or answer keys.
> - **Scrub Sensitive Data**: Redact actual passwords, private keys, hashes, internal usernames, and proprietary IP addresses.
> - **Protect Intellectual Property**: Do not include copyrighted course screenshots, exam questions, or instructor lab solutions.
8.1 Sanitized Command Output
```text

[PLACEHOLDER - EXAMPLE SANITIZED OUTPUT: User Context Verification]

$ id

uid=[UID]([user]) gid=[GID]([group]) groups=...

[PLACEHOLDER - EXAMPLE SANITIZED OUTPUT: System Information Verification]

$ uname -a

Linux [hostname] [kernel_version] ...

[PLACEHOLDER - EXAMPLE SANITIZED OUTPUT: Interface & Addressing Verification]

$ ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> ...

2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> inet [REDACTED_IP]/[CIDR] ...

```
---
9. Remediation & Defensive Hardening
Provide actionable, prioritized recommendations to eliminate root causes and harden the environment.
9.1 Immediate Countermeasures (Short-Term)
[Action 1: Immediate compensating control, e.g., restrict firewall rules, disable unnecessary listening service, revoke unnecessary SUID bits.]
[Action 2: Emergency configuration adjustment or access revocation.]
9.2 Architecture & System Hardening (Long-Term)
Input Validation & Sanitization: [Implement server-side input allowlists and context-aware output encoding.]
Principle of Least Privilege: [Reconfigure services to run under unprivileged, dedicated service accounts.]
Software Lifecycle & Patching: [Establish routine patching to ensure software packages remain up to date.]
Detection & Audit Logging: [Implement audit logging (e.g., auditd, Sysmon, central SIEM) to alert on anomalous execution.]
---
10. Lessons Learned & Methodology Takeaways
Reflect on assessment methodology, technical challenges, and improvements for future engagements.
Effective Techniques:
  - [Techniques, tools, or enumeration strategies that yielded high-fidelity results.]
Encountered Obstacles & Pitfalls:
  - [False leads, tool limitations, or troubleshooting hurdles experienced during testing.]
Methodology Adjustments:
  - [Concrete changes to make to your personal scanning, enumeration, or note-taking habits for future labs.]