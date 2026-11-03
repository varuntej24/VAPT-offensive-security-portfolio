# Manual Application Walking & Client-Side Reconnaissance

## Overview
Automated scanners often generate unnecessary noise or miss non-standard endpoints. Walking an application manually establishes the baseline architecture, input vectors, and hidden routes prior to automated fuzzing or vulnerability testing.

## Key Inspection Vectors

### 1. Source & DOM Analysis (`Ctrl + U` / Elements Tab)
* **HTML Comments:** Checking for forgotten developer notes, staging paths, internal credentials, or feature roadmaps.
* **Hidden Form Controls:** Identifying `<input type="hidden">` fields containing business logic data (roles, flags, discounts) intended to be immutable to client modification.
* **Inline Logic & Scripts:** Locating unlinked endpoints, backend routing patterns, or disabled client-side controls.

### 2. Debugger & Script Auditing (Sources Tab)
* **Route Discovery:** Parsing front-end JavaScript bundles and libraries for API paths (e.g., `/api/v1/...`) not linked in the UI navigation.
* **Hardcoded Credentials & Keys:** Reviewing source code for exposed tokens, test keys, or debug flags (`debug = true`).

### 3. Network Inspection (Network Tab)
* **XHR / Fetch Traffic:** Observing background API calls triggered by asynchronous user actions.
* **Response Header Analysis:** Identifying software fingerprints (`Server`, `X-Powered-By`) and verifying security headers (`Content-Security-Policy`, `Strict-Transport-Security`).

### 4. Storage & State Management (Application / Storage Tab)
* **Cookie Attributes:** Checking for `HttpOnly`, `Secure`, and `SameSite` flags on session tokens.
* **Web Storage:** Auditing `localStorage` and `sessionStorage` for sensitive cached data or authentication state.
