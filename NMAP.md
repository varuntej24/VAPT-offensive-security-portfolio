# 01 - Nmap Live Host Discovery & Probe Mechanics

## Overview
Live host discovery (ping sweeping) identifies which target IP addresses are active before spending time port scanning. By supplying `-sn`, Nmap skips port scanning entirely and focuses exclusively on host liveness.

---

## 1. Host Discovery Probe Matrix

| Scan Type | Command Flag | Probe Sent | Host Is Online If Response Is: | Operational Context |
| :--- | :--- | :--- | :--- | :--- |
| **ARP Scan** | `sudo nmap -PR -sn <CIDR>` | Layer 2 ARP Request | ARP Reply (Target MAC) | **Local subnet only**. Extremely fast, reliable, cannot be blocked by OS firewalls. |
| **ICMP Echo** | `sudo nmap -PE -sn <CIDR>` | ICMP Type 8 (Echo Request) | ICMP Type 0 (Echo Reply) | Standard ping; often blocked by edge routers/firewalls. |
| **ICMP Timestamp** | `sudo nmap -PP -sn <CIDR>` | ICMP Type 13 (Timestamp) | ICMP Type 14 (Reply) | Useful when ICMP Echo (Type 8) is blocked, but timestamp queries are allowed. |
| **ICMP Address Mask** | `sudo nmap -PM -sn <CIDR>` | ICMP Type 17 (Address Mask) | ICMP Type 18 (Reply) | Rarely supported on modern OSs, but checks legacy network devices. |
| **TCP SYN Ping** | `sudo nmap -PS<ports> -sn <CIDR>` | TCP SYN to specific ports | `SYN/ACK` (Port Open) or `RST` (Port Closed) | Hits common ports (e.g., 22, 80, 443). As long as the host responds, it's alive. |
| **TCP ACK Ping** | `sudo nmap -PA<ports> -sn <CIDR>` | TCP ACK to specific ports | `RST` (Reset) | Effective against stateless firewalls that only filter incoming SYN packets. |
| **UDP Ping** | `sudo nmap -PU<ports> -sn <CIDR>` | UDP Datagram to closed ports | ICMP Port Unreachable (Type 3, Code 3) | If a closed port returns an ICMP unreachable error, the host IP is active. |

---

## 2. Essential Discovery Options

```bash
# Disable port scanning (Discovery only)
sudo nmap -sn 10.200.6.0/24

# Disable DNS resolution (Significantly speeds up scans)
sudo nmap -n -sn 10.200.6.0/24

# Force reverse-DNS resolution for all target IPs
sudo nmap -R -sn 10.200.6.0/24

# Treat all hosts as online (Skip host discovery entirely)
# Crucial when targets drop all ping/discovery probes behind a strict firewall
sudo nmap -Pn 10.200.6.50
```

---

## 3. Key Technical Takeaways for VAPT & Interviews

1. **Layer 2 vs. Layer 3 Discovery:**
   * When running Nmap as `root` against targets on the **same local subnet**, Nmap defaults to **ARP queries** (`-PR`) regardless of what ping options are set. ARP cannot be filtered by host-based firewalls.
   * When scanning across a **gateway/router** (remote subnet), Nmap must fall back to Layer 3/4 probes (ICMP, TCP SYN/ACK, UDP).
2. **Why `-PS` and `-PA` Work:**
   * For host discovery, **it does not matter if the port is open or closed**. 
   * If Nmap sends a SYN and gets a `SYN/ACK`, the host is alive.
   * If Nmap sends a SYN and gets an `RST`, the port is closed, **but the host is still alive** because a dead machine cannot send an `RST`.
3. **Firewall Bypasses with TCP ACK (`-PA`):**
   * Many simple packet-filtering firewalls block unsolicited inbound connection attempts (`SYN`), but allow inbound `ACK` packets thinking they belong to an already established outbound connection. When the host receives an unexpected `ACK`, its TCP stack responds with `RST`, confirming the IP is up.
