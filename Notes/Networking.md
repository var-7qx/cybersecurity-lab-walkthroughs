# Networking Fundamentals & Protocol Reference

A practical reference covering network architecture, core protocols, packet behavior, IP addressing, and traffic analysis for authorized cybersecurity lab assessments.

---

## 1. Network Models Comparison

### OSI Model

| Layer | Name | Function / Purpose | Protocols / Technologies | Data Unit |
|---:|---|---|---|---|
| 7 | Application | Network services for applications | HTTP, HTTPS, DNS, SMTP, FTP | Data |
| 6 | Presentation | Data representation, encryption, compression | TLS/SSL concepts, ASCII, JPEG | Data |
| 5 | Session | Establishes and manages communication sessions | RPC, NetBIOS | Data |
| 4 | Transport | End-to-end transport and segmentation | TCP, UDP, SCTP | Segment (TCP) / Datagram (UDP) |
| 3 | Network | Logical addressing and routing | IPv4, IPv6, ICMP, IPsec, OSPF, BGP | Packet |
| 2 | Data Link | Frames and local network addressing | Ethernet, Wi-Fi, ARP | Frame |
| 1 | Physical | Transmission of bits over a medium | Copper, fiber, radio signals | Bits |

**TCP/IP model mapping**

| TCP/IP Layer | Corresponding OSI Layers |
|---|---|
| Application | 5–7 |
| Transport | 4 |
| Internet | 3 |
| Network Access / Link | 1–2 |

*Note: These mappings are conceptual. Some protocols operate across multiple layers, and ARP's classification varies between networking models.*

---

## 2. Common Ports & Protocols Reference

| Port | Protocol / Service | Transport | Typical Usage | Security Assessment Relevance |
|---:|---|---|---|---|
| 21 | FTP | TCP | File transfer | Check encryption and anonymous access |
| 22 | SSH | TCP | Secure remote administration | Review authentication and configuration |
| 23 | Telnet | TCP | Remote terminal | Credentials and traffic may be transmitted in cleartext |
| 25 | SMTP | TCP | Email transfer | Review relay restrictions and supported commands |
| 53 | DNS | TCP/UDP | Name resolution | Review DNS configuration and authorized zone transfers |
| 67/68 | DHCP | UDP | Dynamic IP configuration | Review network trust and rogue-server protections |
| 80 | HTTP | TCP | Web traffic | Review application security and cleartext exposure |
| 88 | Kerberos | TCP/UDP | Network authentication | Review authentication configuration and AD security |
| 110 | POP3 | TCP | Email retrieval | Review transport encryption and authentication |
| 123 | NTP | UDP | Time synchronization | Review exposure and configuration |
| 135 | MS RPC | TCP | Windows RPC endpoint mapper | Review exposed RPC services |
| 139 | NetBIOS Session Service | TCP | Legacy network file sharing | Review legacy SMB exposure |
| 143 | IMAP | TCP | Email access | Review transport encryption and authentication |
| 389 | LDAP | TCP/UDP | Directory services | Review directory access controls and bind configuration |
| 443 | HTTPS | TCP, commonly | Encrypted web traffic over TLS | Review certificates, TLS configuration, and application security |
| 445 | SMB | TCP | Windows file sharing | Review shares, permissions, and SMB signing |
| 1433 | Microsoft SQL Server | TCP, commonly | Database access | Review authentication and network exposure |
| 3306 | MySQL | TCP | Database access | Review remote access and account permissions |
| 3389 | RDP | TCP; UDP also possible | Remote desktop | Review exposure and Network Level Authentication |
| 5985 | WinRM | TCP | Windows remote management over HTTP | Review access controls and transport protection |
| 5986 | WinRM | TCP | Windows remote management over HTTPS | Review certificates and access controls |
| 8080 | Alternate HTTP | TCP, commonly | Web applications, proxies, admin interfaces | Check exposed applications and administrative endpoints |

*Port numbers indicate common defaults, not guaranteed service identity. Services can use non-standard ports.*

---

## 3. IPv4 Addressing & Subnetting

### RFC 1918 Private IPv4 Ranges

| Private Block | Address Range | Total Addresses | Prefix |
|---|---|---:|---|
| 10.0.0.0/8 | 10.0.0.0 – 10.255.255.255 | 16,777,216 | /8 |
| 172.16.0.0/12 | 172.16.0.0 – 172.31.255.255 | 1,048,576 | /12 |
| 192.168.0.0/16 | 192.168.0.0 – 192.168.255.255 | 65,536 | /16 |

These are private address blocks. The older Class A/B/C terminology is classful addressing; CIDR prefixes are the modern way to describe network sizes.

### Common Subnet Masks

| CIDR | Subnet Mask | Total Addresses | Typical Usable Host Addresses |
|---|---|---:|---:|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 |
| /32 | 255.255.255.255 | 1 | 1 address |

*For conventional IPv4 subnets, the network and broadcast addresses are not assigned to hosts. /31 networks are an exception commonly used for point-to-point links. A /32 represents one specific IPv4 address.*

---

## 4. TCP Connection State & Flags

### TCP Control Flags

- **SYN (Synchronize):** Initiates a TCP connection and synchronizes sequence numbers.
- **ACK (Acknowledgment):** Acknowledges received TCP data or control information.
- **FIN (Finish):** Initiates an orderly TCP connection shutdown.
- **RST (Reset):** Resets a TCP connection or rejects a connection attempt.
- **PSH (Push):** Signals that received data should be pushed toward the receiving application without unnecessary buffering.
- **URG (Urgent):** Indicates that the urgent pointer field is significant.

### TCP Three-Way Handshake

```text
Client                         Server
  |                               |
  | -------- SYN (Seq=x) -------> |
  |                               |
  | <--- SYN-ACK (Seq=y, Ack=x+1) |
  |                               |
  | -------- ACK (Ack=y+1) ------>|
  |                               |
  |       Connection established  |
```

The handshake establishes TCP connection state before normal data transfer.

### Port Scan State Interpretation

- **Open:** The target accepts connections on the port. In a typical TCP SYN scan, a SYN-ACK indicates an open port.
- **Closed:** The target is reachable, but the port is not accepting connections; a TCP RST commonly indicates this.
- **Filtered:** A firewall or network filter prevents the scan from determining whether the port is open. Probes may be dropped or receive certain ICMP unreachable responses.
- **Open|Filtered:** Some scan methods cannot distinguish an open port from a filtered port based on the available response.

*Interpretation depends on the scan method, protocol, and network conditions.*

---

## 5. DNS Resolution & Record Types

| Record | Purpose | Example Command |
|---|---|---|
| A | Maps a hostname to an IPv4 address | `dig A example.com +noall +answer` |
| AAAA | Maps a hostname to an IPv6 address | `dig AAAA example.com +noall +answer` |
| CNAME | Provides an alias to another hostname | `dig CNAME www.example.com +noall +answer` |
| MX | Identifies mail exchange servers | `dig MX example.com +noall +answer` |
| NS | Identifies authoritative name servers | `dig NS example.com +noall +answer` |
| TXT | Stores text data, including SPF and verification records | `dig TXT example.com +noall +answer` |
| SOA | Provides zone authority and administrative information | `dig SOA example.com +noall +answer` |
| PTR | Supports reverse DNS lookup | `dig -x 192.0.2.10` |
| AXFR | Requests a DNS zone transfer | `dig AXFR example.com @192.0.2.53` |

*Use your lab's domain and authorized DNS server in place of the example values. A zone transfer should be tested only when authorized.*

---

## 6. Packet Capture & Traffic Analysis

### Common tcpdump Commands

```bash
# Capture packets on an interface and save them to a file
sudo tcpdump -i INTERFACE -w capture.pcap

# Read a capture without resolving addresses to names
tcpdump -r capture.pcap -n

# Capture traffic to or from a specific host
sudo tcpdump -i INTERFACE host TARGET_IP

# Capture traffic on TCP or UDP port 80 or 443
sudo tcpdump -i INTERFACE '(port 80 or port 443)'

# Capture initial TCP SYN packets without ACK
sudo tcpdump -i INTERFACE \
  'tcp[tcpflags] & (tcp-syn|tcp-ack) == tcp-syn'
```

Replace `INTERFACE` and `TARGET_IP` with values from your authorized lab. Use `ip addr show` to inspect available interfaces.

### Essential Wireshark Display Filters

```text
# Packets where the source or destination IP matches
ip.addr == 192.0.2.10

# TCP traffic using port 445
tcp.port == 445

# HTTP GET and POST requests
http.request.method == "GET" || http.request.method == "POST"

# DNS queries containing a domain string
dns.qry.name contains "example.com"

# TCP reset packets
tcp.flags.reset == 1

# TLS Server Hello messages
tls.handshake.type == 2
```

*The lines beginning with `#` are explanatory comments, not Wireshark filter expressions. HTTP filters apply only when Wireshark can dissect the traffic as HTTP; encrypted HTTPS traffic usually requires additional context or decryption to inspect application requests.*

---

## 7. Diagnostic Utilities Reference

```bash
# Trace the network path to a destination
traceroute TARGET_HOST

# Perform DNS lookup
nslookup example.com

# Display the ARP table on systems that support this command
arp -a

# Test TCP connectivity using curl's Telnet protocol handler
curl -v --connect-timeout 3 telnet://TARGET_HOST:PORT
```

Some tools may need to be installed separately. The available options and behavior can differ between Linux distributions.

---

## Practical Learning Checklist

- [ ] Explain the OSI and TCP/IP models.
- [ ] Recognize common ports and their associated services.
- [ ] Calculate network ranges for common CIDR prefixes.
- [ ] Explain a TCP three-way handshake.
- [ ] Interpret open, closed, and filtered scan results.
- [ ] Query common DNS record types.
- [ ] Capture packets using `tcpdump`.
- [ ] Apply Wireshark display filters.
- [ ] Document findings and observations from an authorized lab.

## Responsible Use

Perform scanning, packet capture, and service testing only on systems and networks you own or are explicitly authorized to assess. Sanitize credentials, sensitive traffic, and personal information before publishing captures or screenshots.
