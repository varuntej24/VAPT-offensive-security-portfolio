# Burp Suite: Proxy & Repeater Mechanics

## Overview
Burp Suite operates as a local intercepting HTTP/HTTPS proxy positioned between the client browser and the target web server. It provides deep visibility into application transactions, enabling manual parameter tampering, scope-restricted logging, and payload replay.

This module covers the execution workflow across two foundational components:
1. **Burp Suite: The Basics:** Architecture, project vs. user configurations, CA certificate trust negotiation, and target scope isolation.
2. **Burp Suite: Repeater:** Request manipulation, response view triage (Pretty/Raw/Hex/Render), Inspector tab modification, and manual injection verification.

---

## 1. Proxy Architecture & Local CA Trust Negotiation

```text
[ Browser (Firefox + FoxyProxy) ]
               │
      (HTTP / Plaintext TLS)
               ▼
   [ 127.0.0.1:8080 (Burp Proxy) ]  ◄── Dynamically signs TLS with PortSwigger CA
               │
      (Re-encrypted TLS)
               ▼
       [ Target Server ]
```

### The CA Certificate Installation Workflow
When intercepting encrypted HTTPS traffic, browsers throw severe TLS/SSL certificate authority invalidity errors because Burp behaves as an intentional Man-in-the-Middle (MITM).
1. Configure browser proxy (via FoxyProxy extension) to route HTTP/HTTPS to `127.0.0.1:8080`.
2. Navigate to `http://burpsuite` and download the `cacert.der` certificate.
3. Import the certificate into Firefox (`Settings -> Privacy & Security -> Certificates -> View Certificates -> Authorities -> Import`).
4. Enable **"Trust this CA to identify websites"**.

### Target Scope Management
By default, the Burp Proxy logs all network traffic passing through the port, including background OS updates, analytics, and browser extensions.
* **Setting Scope:** In `Target -> Site map`, right-click the root target domain and select **Add to scope**.
* **Display Filtering:** In `Proxy -> HTTP history`, click the filter bar and check **"Show only in-scope items"**.
* **Intercept Control:** In `Proxy -> Proxy settings -> Request interception rules`, toggle rules to ensure intercept triggers strictly on in-scope items.

---

## 2. Request Interception & Client-Side Bypass

Burp Proxy intercepts requests before they hit the network wire, proving that **client-side input validation and form restrictions offer zero security protection**.

### Operational Intercept Controls
* **Forward:** Releases the currently held request/response to its destination.
* **Drop:** Aborts the TCP connection; neither client nor server processes the rest of the payload.
* **Action Menu / Context Hotkey (`Ctrl + R`):** Dispatches the exact request state to other Burp modules (Repeater) for manual testing.

### Practical Lab Walkthrough: Bypassing Form Constraints
1. **Target:** Support contact form with client-side JavaScript input sanitization.
2. **Mechanism:** Entering special characters or exceeding character limits is blocked by local event listeners in the DOM.
3. **Execution:**
   * Entered valid alphanumeric input in the browser form.
   * Caught the outbound `POST /ticket/new` request in **Burp Proxy Intercept**.
   * Replaced the clean form fields with raw HTML/Script payloads directly in the intercept window:
     ```http
     POST /ticket/submit HTTP/1.1
     Host: target.thm
     Content-Type: application/x-www-form-urlencoded
     Content-Length: 42

     name=Test&message=<script>alert(1)</script>
     ```
   * Clicked **Forward**. The backend accepted the payload without error, demonstrating that client-side controls are bypassable upstream.

---

## 3. Burp Suite: Repeater Manual Request Crafting

Repeater allows capturing an existing HTTP request, manually modifying headers, parameters, or bodies, and resending it repeatedly while observing the direct raw response.

### Interface & Response Panels
* **Request List & History:** Tracks all iterations of a modified request, allowing navigation backward and forward through changes.
* **Inspector Tab:** Breaks raw text into manageable UI trees:
  * Request Attributes (Method, Protocol, Path).
  * Query Parameters & Body Parameters.
  * Request/Response Headers.
  * Cookie attributes.
* **Response Views:**
  * **Pretty:** Formatted and syntax-highlighted HTML/JSON/XML.
  * **Raw:** Exact byte stream returned by the server, critical for identifying unparsed headers and custom cookie flags.
  * **Hex:** Displays byte values in hexadecimal, used to check for binary data, null bytes (`%00`), and character encoding anomalies.
  * **Render:** Emulates a browser view to inspect visual layout changes or client-side redirects.

---

## 4. Exploitation Scenarios in Repeater

### Scenario A: Header Spoofing & Access Control Bypass
* **Endpoint:** `GET / HTTP/1.1`
* **Vulnerability:** Unauthenticated or unauthorized endpoint relying on custom header presence for access evaluation.
* **Methodology:**
  1. Captured the initial request to root `/`.
  2. Dispatched to Repeater (`Ctrl + R`).
  3. Appended authorization header in the request panel:
     ```http
     GET / HTTP/1.1
     Host: target.thm
     User-Agent: Mozilla/5.0
     FlagAuthorised: True
     Connection: close

     ```
  4. Dispatched payload (`Send`). Server returned an administrative response containing the access verification token.

### Scenario B: Triggering Server-Side Exceptions (Error Probing)
* **Endpoint:** `GET /products/1 HTTP/1.1`
* **Vulnerability:** Lack of type validation and unhandled exceptions on backend queries.
* **Methodology:**
  1. Sent `GET /products/1` to Repeater.
  2. Tested boundary values to verify input filtering:
     * Sent `/products/-1` and `/products/alpha`.
  3. The web application failed to sanitize the negative integer and broke internal query execution, triggering an unhandled `500 Internal Server Error`.
  4. Response leakage confirmed vulnerable error-handling pipelines and unparameterized backend logic.

### Scenario C: Union-Based SQL Injection via Repeater
* **Endpoint:** `GET /about/ID HTTP/1.1`
* **Target Table:** `people` | **Target Column:** `notes` (CEO ID = 1)
* **Methodology:**
  1. Captured `/about/2` and sent to Repeater (`Ctrl + R`).
  2. Verified injection vulnerability by breaking syntax with a single quote (`'`), returning a 500 error.
  3. Determined the number of columns in the original query using `ORDER BY`:
     * Balanced at 5 columns.
  4. Checked column data type reflection:
     * Discovered column 1 reflects string data.
  5. Injected UNION SELECT payload directly via Repeater:
     ```http
     GET /about/0%20UNION%20ALL%20SELECT%20notes,null,null,null,null%20FROM%20people%20WHERE%20id%20=%201 HTTP/1.1
     Host: target.thm
     Connection: close
     ```
  6. The server executed the multi-query and returned the contents of the `notes` column inside the HTTP response body.

---

## 5. Technical Interview Defense Points

* **"Why is installing a custom root CA required to intercept HTTPS traffic in Burp Suite?"**
  HTTPS is encrypted end-to-end via TLS. If a proxy attempts to read or modify traffic without the client's explicit consent, the browser detects an invalid certificate (name mismatch, untrusted signer) and terminates the connection. Installing the PortSwigger CA into the operating system or browser trust store designates Burp as a trusted certificate authority. This allows Burp to generate valid TLS certificates on the fly for any visited domain, establishing a secure TLS session with the client while maintaining a separate secure session with the upstream server.
* **"What is the operational difference between the Proxy tab and the Repeater tab?"**
  The **Proxy** tab is an inline interceptor designed for passive observation and real-time tampering of live browser traffic passing across the wire. The **Repeater** tab is an isolated laboratory tool used to store, edit, and replay a specific captured request multiple times independently of the browser. It allows rapid parameter fuzzing, header manipulation, and error testing without navigating through the user interface repeatedly.
* **"Why is setting Target Scope critical during an authorized penetration testing engagement?"**
  Scope definition ensures operational and legal compliance. Browsers routinely generate background traffic (telemetry, extension queries, OS updates, third-party CDNs). Restricting scope in Burp prevents third-party services outside the rules of engagement from being logged, intercepted, or tested, while also eliminating noise from the project's HTTP history and sitemap.
