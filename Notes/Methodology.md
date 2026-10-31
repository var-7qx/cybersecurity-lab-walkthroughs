# Penetration Testing & Assessment Methodology

A structured assessment framework informed by the **Penetration Testing Execution Standard (PTES)**, **NIST SP 800-115**, and the **OWASP Web Security Testing Guide (WSTG)**.

---

## Assessment Lifecycle Overview

```text
[1. Scoping & Rules of Engagement]
                 |
                 v
[2. Reconnaissance & Intelligence Gathering]
                 |
                 v
[3. Enumeration & Attack Surface Mapping]
                 |
                 v
[4. Vulnerability Analysis & Validation]
                 |
                 v
[5. Post-Exploitation & Privilege Auditing]
                 |
                 v
[6. Reporting, Remediation & Defensive Analysis]
```

*The assessment phases may overlap or require iteration. Post-exploitation activities are performed only when explicitly authorized.*

---

## Phase 1: Pre-Engagement & Scoping

Establish authorization, boundaries, and communication procedures before starting assessment activities.

- **Rules of Engagement (RoE):** Define permitted testing windows, test accounts, excluded IP ranges, and prohibited activities such as denial-of-service testing.
- **Authorization Verification:** Confirm written permission and applicable platform rules.
- **Communication Channels:** Establish escalation procedures for unexpected service interruptions or other incidents.
- **Testing Infrastructure:** Verify the required VPN connection, routing, and connectivity to the authorized lab network.

## Phase 2: Reconnaissance & Intelligence Gathering

Collect contextual information about in-scope systems, domains, and exposed services.

### Passive Reconnaissance

- **Domain & DNS Intelligence:** Review WHOIS data, ASN information, and public DNS records such as A, MX, NS, and TXT.
- **Open-Source Intelligence (OSINT):** Review relevant public code repositories, technical documentation, forums, and archived pages.
- **Subdomain Discovery:** Review Certificate Transparency logs and other public sources for in-scope subdomains.

### Active Reconnaissance

- **Host Discovery:** Use appropriate ICMP, TCP, or local-network discovery techniques within the authorized scope.
- **Port Scanning:** Identify exposed TCP and UDP services using carefully scoped scans.
- **Service Fingerprinting:** Determine service versions, supported protocols, and possible operating-system indicators.

## Phase 3: Enumeration & Attack Surface Mapping

Investigate discovered services to understand their functionality, configuration, and potential security weaknesses.

### Service-Specific Enumeration

- **HTTP / HTTPS**
  - Identify web servers, CMS platforms, frameworks, and backend technologies.
  - Discover relevant directories, administrative endpoints, and API routes.
  - Map authentication workflows and authorization boundaries.

- **SMB / RPC**
  - Review accessible shares, domain information, group memberships, and password policies where permitted.
  - Check guest access, anonymous sessions, and SMB signing configuration.

- **Database Services**
  - Identify database versions, access controls, authentication settings, and remote-access policies.

- **LDAP / Active Directory**
  - Review accessible directory objects, user and group relationships, service principal names (SPNs), and domain configuration.

## Phase 4: Vulnerability Analysis & Validation

Assess discovered services and applications for security flaws, vulnerable software, and unsafe configurations.

### 1. Vulnerability Identification

- Compare confirmed software versions and configurations with trusted vulnerability references, including CVE records and the National Vulnerability Database (NVD).
- Check for insecure defaults, exposed management interfaces, and excessive permissions.
- Test relevant web application security categories, including SQL injection, cross-site scripting (XSS), server-side request forgery (SSRF), insecure direct object references (IDOR), and command injection.

### 2. Controlled Validation & Proof of Concept

- Use controlled tests designed to confirm a suspected vulnerability while minimizing impact.
- Record relevant parameters, commands, test conditions, and timestamps.
- Capture sanitized evidence sufficient to demonstrate the finding without exposing credentials, secrets, or unrelated data.
- Distinguish confirmed vulnerabilities from unverified suspicions.

## Phase 5: Post-Exploitation & Privilege Auditing

When explicitly permitted, assess the impact of initial access and review the host's privilege boundaries.

### Local Enumeration

- **Identity & Context:** Review the current user, group memberships, and effective privileges.
- **Linux Privilege Review:** Inspect SUID/SGID binaries, sudo permissions, capabilities, scheduled tasks, writable configuration files, and kernel versions.
- **Windows Privilege Review:** Inspect service permissions, service paths, relevant registry settings, assigned privileges, and credential-storage risks.
- **Sensitive Data Review:** Identify potential exposure of credentials, connection strings, keys, and sensitive configuration files without unnecessarily collecting or disclosing their contents.

### Network Pivoting & Lateral Movement

- Review network interfaces, routing tables, and neighbor information to understand reachable network segments.
- Assess forwarding, tunneling, and segmentation controls only within the approved scope.
- Avoid accessing unrelated systems or expanding the assessment beyond its authorization.

## Phase 6: Reporting, Remediation & Defensive Analysis

Translate technical observations into actionable security improvements.

### 1. Root-Cause Analysis

Explain why the weakness exists, such as missing input validation, excessive permissions, insecure defaults, or unpatched software.

### 2. Remediation Plan

- Recommend specific code fixes, configuration changes, access-control improvements, or architecture changes.
- Distinguish immediate mitigations from long-term corrective actions.
- Include verification steps to confirm that remediation is effective.

### 3. Documentation & Evidence Hygiene

- Remove credentials, tokens, personally identifiable information (PII), and other sensitive data from published evidence.
- Make findings reproducible without publishing unnecessary exploit details or secrets.
- Document impact, evidence, remediation, and lessons learned.
- Prioritize findings using risk, exploitability, and business impact.

---

## Assessment Principles

- **Authorization:** Test only systems explicitly included in the approved scope.
- **Least Impact:** Prefer the least disruptive method that provides sufficient evidence.
- **Evidence Quality:** Separate observed facts from assumptions and unverified hypotheses.
- **Confidentiality:** Store assessment data securely and sanitize public documentation.
- **Defensive Value:** Connect each finding to a clear remediation and verification plan.

## Reference Standards

- [Penetration Testing Execution Standard (PTES)](http://www.pentest-standard.org/)
- [NIST SP 800-115: Technical Guide to Information Security Testing and Assessment](https://csrc.nist.gov/pubs/sp/800/115/final)
- [OWASP Web Security Testing Guide (WSTG)](https://owasp.org/www-project-web-security-testing-guide/)
