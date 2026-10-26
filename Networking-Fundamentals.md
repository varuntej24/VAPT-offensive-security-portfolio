# Network Protocols & Architecture Cheat Sheet (Day 1)

## 1. The OSI vs. TCP/IP Models
- **Layer 7 - Application:** HTTP/S, DNS, SSH, FTP, SMB (Application interfaces).
- **Layer 4 - Transport:** TCP (reliable, stateful, connection-oriented) and UDP (stateless, connectionless).
- **Layer 3 - Network:** IP addressing, routing, ICMP packets.
- **Layer 2 - Data Link:** MAC addressing, Ethernet frames, ARP resolution.
- **Layer 1 - Physical:** Raw electrical/optical bitstream signals.

---

## 2. TCP 3-Way Handshake & Teardown
- **Establishment:**
  1. `Client` -> `SYN` (Sequence = X) -> `Server`
  2. `Server` -> `SYN-ACK` (Sequence = Y, Acknowledgment = X+1) -> `Client`
  3. `Client` -> `ACK` (Acknowledgment = Y+1) -> `Server`
- **Termination:**
  - `FIN` -> Graceful teardown initiated by either end.
  - `RST` -> Immediate connection reset (port closed, packet dropped, or abnormal termination).

---

## 3. Core Penetration Testing Port Directory
| Port | Protocol | Default Service | Core Security Risk / Vectors |
| :--- | :--- | :--- | :--- |
| **21** | TCP | FTP | Anonymous login, cleartext credentials, exploit vulnerable daemons (e.g., vsftpd 2.3.4). |
| **22** | TCP | SSH | Brute-force attacks, exposed private keys, outdated SSH versions. |
| **53** | TCP/UDP | DNS | DNS zone transfers (`AXFR`), cache poisoning, subdomain enumeration. |
| **80** | TCP | HTTP | Web app flaws (OWASP Top 10), cleartext traffic sniffing. |
| **443** | TCP | HTTPS | SSL/TLS misconfigurations, certificate bypasses, web exploits. |
| **445** | TCP | SMB | Null sessions, EternalBlue (MS17-010), unauthenticated shares, credential relaying. |

---

## 4. Key Diagnostic Commands
- **ICMP Echo Probe:** `ping -c 4 <target-ip>`
- **Route Tracing:** `traceroute <target-ip>` (or `tracert <target-ip>` on Windows)
- **DNS Records Query:** `dig <domain> ANY +noall +answer` (or `nslookup -type=any <domain>` on Windows)
- **DNS Zone Transfer Check:** `dig axfr @<nameserver> <domain>`
- **Domain WHOIS Lookup:** `whois <domain>`
