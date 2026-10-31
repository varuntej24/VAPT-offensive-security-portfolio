# 03 - Nmap Post Port Scans: Service Detection, OS Fingerprinting, and NSE

## Overview
Discovering that a port is open is only the initial reconnaissance phase. Post-port scanning investigates the software running behind active sockets. This phase covers:
1. Identifying exact application service versions (`-sV`).
2. Fingerprinting the host operating system (`-O`).
3. Tracing packet routes (`--traceroute`).
4. Automating vulnerability and configuration auditing using the Nmap Scripting Engine (NSE).
5. Saving results across standard formats for assessment reporting.

---

## 1. Service Version Detection (`-sV`)

When Nmap identifies an open TCP/UDP port, it initially infers the service name from the well-known port mapping table (`/etc/services` or `nmap-services`). However, services frequently run on non-standard ports (e.g., HTTP on port 8080 or SSH on port 2222).

### Mechanics
Version scanning probes open ports with targeted application payloads (probes defined in `nmap-service-probes`) and compares the returned application banners and signatures against a database of thousands of known services.

```bash
# Basic service version detection
sudo nmap -sV <TARGET_IP>

# Control version detection intensity (0-9, default is 7)
# Higher intensity increases accuracy and probe count at the cost of speed
sudo nmap -sV --version-intensity 9 <TARGET_IP>

# Light detection (faster, less intrusive; intensity 2)
sudo nmap -sV --version-light <TARGET_IP>

# All probes (intensity 9, tests all signatures)
sudo nmap -sV --version-all <TARGET_IP>
```

---

## 2. OS Detection (`-O`) & Traceroute

### OS Fingerprinting (`-O`)
Nmap analyzes subtle implementation differences in how operating system network stacks conform to (or diverge from) RFC standards. It sends a sequence of TCP and UDP packets with specific flag combinations, options, and window sizes, comparing the response signatures to the database in `nmap-os-db`.

* **Metrics Evaluated:**
  * Initial Sequence Number (ISN) generation patterns.
  * TCP options support and ordering (Window Scale, SACK, Timestamps).
  * IP header ID sampling.
  * Explicit Congestion Notification (ECN) responses.
  * Reset (`RST`) packet bit patterns and quirks.

```bash
# Enable OS detection (requires root/raw socket privileges)
sudo nmap -O <TARGET_IP>

# Attempt aggressive OS guessing if matches are not definitive
sudo nmap -O --osscan-guess <TARGET_IP>

# Limit OS detection to promising targets (must have at least 1 open and 1 closed TCP port)
sudo nmap -O --osscan-limit <TARGET_IP>
```

### Traceroute (`--traceroute`)
Integrates network route discovery into the scan, determining the intermediate layer-3 hops to the target host using response data from the scan probes.

```bash
# Execute traceroute alongside version and OS detection
sudo nmap -sV -O --traceroute <TARGET_IP>
```

---

## 3. The Aggressive Scanning Preset (`-A`)

The `-A` switch is an aggregate option that combines multiple post-port scanning capabilities into a single flag:
* Enables OS detection (`-O`).
* Enables Service Version detection (`-sV`).
* Enables default Script Scanning (`-sC`).
* Enables Traceroute (`--traceroute`).

```bash
# Comprehensive aggressive scan
sudo nmap -A -p- <TARGET_IP>
```

---

## 4. Nmap Scripting Engine (NSE)

The Nmap Scripting Engine allows users to automate tasks ranging from advanced service enumeration to vulnerability auditing and configuration verification. Scripts are written in **Lua** and stored locally at `/usr/share/nmap/scripts/`.

### NSE Script Categories

| Category | Description | Safety Level |
| :--- | :--- | :--- |
| **`default`** | Standard essential scripts run with `-sC`; fast, reliable, and low network footprint. | Safe |
| **`safe`** | Non-intrusive scripts designed not to crash services or consume excessive bandwidth. | Safe |
| **`discovery`** | Queries active services and registries for additional network/directory information. | Safe |
| **`version`** | Extends `-sV` by sending specialized protocol probes to fingerprint versions. | Safe |
| **`vuln`** | Checks whether the target service matches signatures for known specific vulnerabilities (CVEs). | Low-to-Moderate Risk |
| **`auth`** | Audits authentication schemes and tests for default or bypass credentials. | Moderate Risk |
| **`intrusive`** | High risk of crashing unstable target services or triggering intrusion detection alarms. | High Risk |
| **`dos`** | Probes that may cause service denial of service; used for stress testing. | High Risk |

### Practical NSE Execution

```bash
# Run default script checks (equivalent to --script=default)
sudo nmap -sC -sV <TARGET_IP>

# Run all scripts within the 'safe' and 'discovery' categories
sudo nmap --script "safe or discovery" <TARGET_IP>

# Run vulnerability checks against identified services
sudo nmap -sV --script "vuln" -p 21,22,80,445 <TARGET_IP>

# Run a specific script by name (e.g., auditing banner/anonymous access)
sudo nmap -p 21 --script ftp-anon <TARGET_IP>
sudo nmap -p 445 --script smb-os-discovery <TARGET_IP>

# Inspect script details and help documentation
nmap --script-help "ftp-*"
```

---

## 5. Output Management and Formats

Pentesters and security analysts must preserve scan outputs for reporting, evidence tracking, and downstream pipeline processing.

| Flag | Format Type | Description & Primary Use Case |
| :--- | :--- | :--- |
| **`-oN`** | Normal | Standard human-readable output matching terminal display. |
| **`-oX`** | XML | Structured machine-readable output; parsed by automated tools and reporting frameworks. |
| **`-oG`** | Grepable | Single-line per host output; legacy format designed for quick parsing with `grep`, `awk`, and `cut`. |
| **`-oA`** | All Formats | Automatically generates `.nmap` (normal), `.xml` (XML), and `.gnmap` (grepable) files simultaneously. |

```bash
# Best Practice: Always record scans using -oA
sudo nmap -sV -sC -p- -oA scans/target_recon <TARGET_IP>
```

---

## 6. Practical Pentesting Workflow & Synthesis

```bash
# Standard Initial Triage (Fast Host & Common Service Scan)
sudo nmap -sS -sV -sC -F --min-rate 1000 -oA scans/initial_quick <TARGET_IP>

# Full Coverage Port Sweep (All 65,535 Ports)
sudo nmap -sS -p- --min-rate 1000 -oA scans/all_ports <TARGET_IP>

# Deep Analysis on Discovered Open Ports (Example: Ports 22, 80, 445)
sudo nmap -sV -sC -O --traceroute -p 22,80,445 -oA scans/deep_audit <TARGET_IP>
```

### Key Technical Takeaways for Interviews
* **How does version detection differ from port scanning?**
  Port scanning only evaluates whether a transport socket is open, closed, or filtered. Version detection (`-sV`) completes the handshake, issues application-specific probes from `nmap-service-probes`, and evaluates the returned banner and payload against regular expression signatures.
* **Why can OS detection fail?**
  OS fingerprinting requires at least one open port and one closed port to observe how the TCP/IP stack reacts to different probe conditions (e.g., comparing a `SYN-ACK` from an open port with an `RST` from a closed port). If all ports are filtered or only open ports exist, accuracy drops significantly.
* **Why use `-oA` consistently?**
  Assessment documentation must be reproducible. Saving all three formats ensures human readability (`.nmap`), grep compatibility for quick IP/port filtering (`.gnmap`), and XML ingestion into tools like Nessus, Metasploit, or custom reporting scripts (`.xml`).
