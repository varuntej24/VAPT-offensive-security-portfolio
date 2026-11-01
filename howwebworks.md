# How the Web Works: End-to-End Mechanics & Protocol Synthesis

## Overview
Web application security requires understanding the lifecycle of a web transaction from the moment a user enters a URL to the final rendering of the page. This module synthesizes four foundational layers:
1. **DNS in Detail:** Domain hierarchy, resolution flow, caching, and record architectures.
2. **HTTP in Detail:** Requests, response headers, verbs, status codes, and cookies.
3. **How Websites Work:** DOM trees, CSSOM, client-side script execution, and injection vectors.
4. **Putting It All Together:** The complete packet flow and handshake sequence.

---

## 1. DNS in Detail

The Domain Name System translates human-readable domain names into routable IP addresses via hierarchical, distributed queries across **UDP/TCP port 53**.

```text
User Input: [https://sub.example.com](https://sub.example.com)
                      │
                      ▼
[ Browser / OS Cache ] ──(Miss)──► [ Recursive Resolver (ISP / 1.1.1.1 / 8.8.8.8) ]
                                             │
      ┌──────────────────────────────────────┼──────────────────────────────────────┐
      ▼                                      ▼                                      ▼
[ 1. Root Server (.) ]           [ 2. TLD Server (.com) ]         [ 3. Authoritative Nameserver ]
  Returns TLD nameserver           Returns authoritative NS          Returns final A/AAAA record
```

### The 4-Step Resolution Sequence
1. **Recursive Resolver:** Receives the query from the client OS stub resolver. Checks local cache; if not found, initiates recursive resolution.
2. **Root Nameserver (`.`):** Directs the recursive resolver to the Top-Level Domain (TLD) server responsible for the extension (e.g., `.com`).
3. **TLD Nameserver:** Directs the resolver to the domain's Authoritative Nameserver.
4. **Authoritative Nameserver:** Holds the actual DNS zone records. Returns the destination IP address (`A` or `AAAA`) back to the resolver, which caches it and serves it to the client.

### Key Record Types & Attack Surface

| Record | Purpose | Security / Reconnaissance Relevance |
| :--- | :--- | :--- |
| **`A` / `AAAA`** | Hostname to IPv4 / IPv6 mapping | Identifies target public and origin IP addresses. |
| **`CNAME`** | Canonical name alias | Can lead to **Subdomain Takeover** if pointing to an unclaimed cloud resource (e.g., AWS S3, GitHub Pages). |
| **`MX`** | Mail Exchange server | Identifies third-party mail providers and anti-spam filters. |
| **`TXT`** | Arbitrary text payload | Leaks domain verification tokens, SPF (`v=spf1`), DKIM, and DMARC policies. |
| **`NS`** | Authoritative nameservers | Target for DNS Zone Transfers (`AXFR`) if misconfigured. |

---

## 2. HTTP in Detail

HTTP is a stateless, client-server protocol running on **TCP port 80** (or encapsulated via TLS on **TCP port 443**).

### Anatomy of an HTTP Request
```http
POST /api/login HTTP/1.1
Host: secure.example.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64)
Content-Type: application/x-www-form-urlencoded
Content-Length: 29
Cookie: session_hint=guest; tracking_id=89a7f3

username=admin&password=Password123!
```

* **Request Line:** Method (`POST`), Request-URI (`/api/login`), and Protocol Version (`HTTP/1.1`).
* **Headers:** Key-value pairs communicating metadata, content encoding, cache policies, and authentication state.
* **Blank Line (`\r\n\r\n`):** Terminates the header block; required by RFC specifications.
* **Message Body:** Payload sent to the server (form parameters, JSON, XML, or binary data).

### Core Methods & Security Implications
* **`GET`:** Safe and idempotent; parameters append directly to the URL (`/profile?id=10`). Susceptible to logging in server access logs and browser history.
* **`POST`:** Submits data in the request body; creates new resources or processes authentication.
* **`PUT` / `DELETE`:** Replaces resources or deletes targets. Dangerous if exposed without strict authorization checks.
* **`OPTIONS`:** Queries allowed HTTP verbs via the `Allow` header; critical during CORS (Cross-Origin Resource Sharing) preflight checks.

### HTTP Response Anatomy & Status Code Classes

```http
HTTP/1.1 200 OK
Date: Sun, 01 Nov 2026 03:15:00 GMT
Server: Apache/2.4.52 (Ubuntu)
Set-Cookie: sessionid=k72h9df83n; Secure; HttpOnly; SameSite=Strict
Content-Type: text/html; charset=UTF-8
Content-Length: 154

<html><body>Welcome to the Portal</body></html>
```

| Class | Category | Diagnostic Purpose |
| :--- | :--- | :--- |
| **`1xx`** | Informational | Request received, continuing process (e.g., `101 Switching Protocols`). |
| **`2xx`** | Success | Action completed successfully (`200 OK`, `201 Created`, `204 No Content`). |
| **`3xx`** | Redirection | Further action required (`301 Moved Permanently`, `302 Found`). |
| **`4xx`** | Client Error | Syntax or authorization issue (`400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`). |
| **`5xx`** | Server Error | Backend execution failure (`500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`). |

---

## 3. How Websites Work (Client-Side Rendering)

Browsers download raw text assets over HTTP and construct an interactive interface using three core technologies:

1. **HTML (Structure):** Parsed into the **Document Object Model (DOM)**, an in-memory tree representation of all nodes, forms, inputs, and attributes.
2. **CSS (Presentation):** Parsed into the **CSSOM**, applying styling rules and positioning layouts.
3. **JavaScript (Behavior):** Interpreted by the client browser engine (V8, SpiderMonkey) to handle dynamic logic, event hooks, and asynchronous API calls (`fetch`, `XMLHttpRequest`).

### Client-Side Security Realities
* **DOM Inspection:** All client-side code delivered to the browser can be viewed, paused, and manipulated using Browser Developer Tools (`F12`).
* **Hidden Forms & Variables:** Hiding form fields (`<input type="hidden">`) or disabling buttons via JavaScript (`disabled`) provides no security boundary. Attackers bypass client validation by intercepting requests in Burp Suite.
* **HTML & Script Injection:** Unsanitized user inputs reflected into the DOM trigger HTML Injection or escalate directly to Cross-Site Scripting (XSS).

---

## 4. Putting It All Together: The Complete Web Transaction

When a user submits `https://example.com/login` into a web browser, the system executes the following deterministic lifecycle:

```text
[ Step 1: DNS Resolution ]
Client ──► Resolver ──► Root/TLD/Authoritative ──► Returns Target IP (e.g., 93.184.216.34)

[ Step 2: Transport Layer Handshake (TCP) ]
Client ──► [SYN] ───────────────────────────────► Web Server (Port 443)
Client ◄── [SYN, ACK] ────────────────────────── Web Server
Client ──► [ACK] ───────────────────────────────► Web Server

[ Step 3: Cryptographic Handshake (TLS) ]
Client ──► ClientHello (Ciphers, TLS version) ──► Web Server
Client ◄── ServerHello + Digital Certificate ─── Web Server
Client ──► Key Exchange & Session Keys ────────► Web Server
[ Symmetrical Encryption Established ]

[ Step 4: Application Layer Exchange (HTTPS) ]
Client ──► Encrypted GET /login HTTP/1.1 ───────► Web Server
Client ◄── Encrypted 200 OK (HTML/CSS/JS) ────── Web Server

[ Step 5: Client-Side Execution ]
Browser engine parses DOM + executes JS ────────► Renders login portal
```

---

## 5. Technical Interview Defense Points

* **"What happens under the hood between pressing 'Enter' on a URL and viewing the page?"**
  1. The browser checks local caches (browser cache, OS cache, hosts file).
  2. If uncached, the OS requests an IP from the configured recursive DNS resolver, which traverses Root, TLD, and Authoritative servers.
  3. The client initiates a TCP 3-way handshake (`SYN`, `SYN-ACK`, `ACK`) with the resolved IP on port 80 (HTTP) or 443 (HTTPS).
  4. If HTTPS, a TLS handshake negotiates encryption algorithms, authenticates the server's certificate against trusted CAs, and establishes symmetric session keys.
  5. The client issues an HTTP `GET` request. The web server processes the request and responds with a status code (e.g., `200 OK`) and payload.
  6. The browser engine parses the HTML to construct the DOM, applies CSSOM styles, renders the layout, and executes client-side JavaScript.
* **"Why is client-side validation useless against a penetration tester?"**
  Client-side validation (HTML attributes like `required` or JS functions checking string length/format) executes solely within the user's browser environment. An attacker operates upstream of the server, completely controlling the client environment, and can easily bypass checks using DevTools or an intercepting proxy like Burp Suite before the payload reaches backend validation logic.
* **"What is the security difference between the `401 Unauthorized` and `403 Forbidden` status codes?"**
  `401 Unauthorized` indicates that authentication is missing or invalid; the client can authenticate and re-request the resource. `403 Forbidden` indicates that the server understands the client's identity, but authorization is explicitly denied; submitting credentials will not grant access.
