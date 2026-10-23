# Web Application Penetration Testing Methodology

## 1. Objective

This document describes a structured methodology for assessing web application security. Testing must be performed only against applications owned by the tester or explicitly authorized for assessment.

## 2. Scope and Authorization

Before testing:

* Identify the approved target domains, IP addresses, applications, and environments.
* Confirm written authorization and the permitted testing window.
* Identify prohibited actions, rate limits, and points of contact.
* Use dedicated test accounts and test data where possible.
* Stop if testing affects availability or exposes unexpected sensitive data.

## 3. Testing Workflow

### Phase 1: Reconnaissance

Objective: Understand the application and its exposed functionality.

Checklist:

* [ ] Identify in-scope hostnames and application entry points.
* [ ] Review publicly available application documentation.
* [ ] Identify technologies and frameworks using non-invasive methods.
* [ ] Record discovered pages, endpoints, and authentication flows.
* [ ] Keep all discovery within the approved scope.

Deliverable: An inventory of in-scope assets and application entry points.

### Phase 2: Application Mapping

Objective: Understand how the application handles requests, identities, and data.

Checklist:

* [ ] Browse the application using a normal test account.
* [ ] Map important pages, forms, parameters, and API endpoints.
* [ ] Identify roles and permissions provided by the application.
* [ ] Observe session cookies, security headers, and authentication flows.
* [ ] Record expected behavior for important operations.

Deliverable: An application map and list of security-relevant functionality.

### Phase 3: Authentication and Session Management

Objective: Assess how users authenticate and maintain sessions.

Checklist:

* [ ] Review password and account recovery workflows.
* [ ] Check session cookie attributes such as Secure, HttpOnly, and SameSite.
* [ ] Observe session expiration and logout behavior.
* [ ] Review session renewal after authentication or privilege changes.
* [ ] Check for weaknesses using authorized test accounts only.

Deliverable: Documented authentication and session-management observations.

### Phase 4: Authorization and Access Control

Objective: Determine whether users can access only the resources and actions permitted to their roles.

Checklist:

* [ ] Compare access using separate authorized test accounts.
* [ ] Check object-level access controls with test-owned records.
* [ ] Verify role restrictions on sensitive operations.
* [ ] Check whether server-side authorization is enforced.
* [ ] Avoid accessing other users' real data.

Deliverable: Evidence of confirmed access-control weaknesses or a record of tests performed.

### Phase 5: Input Validation and Injection

Objective: Evaluate how the application processes untrusted input.

Checklist:

* [ ] Identify input points in forms, URLs, headers, and API requests.
* [ ] Review server-side validation and output encoding.
* [ ] Use harmless test inputs in an approved environment.
* [ ] Evaluate potential SQL injection, cross-site scripting, and command injection only within the approved scope.
* [ ] Record the affected input, observed behavior, and safe reproduction steps.

Deliverable: Validated findings with minimal-impact evidence.

### Phase 6: API Security

Objective: Evaluate API authentication, authorization, and data handling.

Checklist:

* [ ] Inventory documented and observed API endpoints.
* [ ] Review authentication requirements for protected endpoints.
* [ ] Check object-level and function-level authorization.
* [ ] Review excessive data exposure and input validation.
* [ ] Check rate limiting and error handling within approved limits.
* [ ] Verify that sensitive operations require appropriate authorization.

Deliverable: An API endpoint inventory and documented security findings.

### Phase 7: Security Configuration

Objective: Review common configuration and deployment weaknesses.

Checklist:

* [ ] Review HTTPS enforcement and certificate validity.
* [ ] Inspect relevant security headers.
* [ ] Check cookie security attributes.
* [ ] Review verbose error messages and unintended information disclosure.
* [ ] Check for exposed debug features or directory listings.
* [ ] Review cross-origin resource sharing (CORS) behavior.

Deliverable: A configuration review with recommended improvements.

### Phase 8: Evidence and Reporting

For every confirmed finding, record:

* Finding title and unique identifier.
* Affected endpoint or component.
* Severity and rationale.
* Preconditions and required privileges.
* Minimal, reproducible steps.
* Sanitized screenshots or request/response evidence.
* Security impact.
* Recommended remediation.
* Retest status, if applicable.

Never include passwords, session tokens, personal data, or unnecessary sensitive information in reports.

## 4. Finding Severity

Assess severity using business impact, exploitability, required privileges, exposure, and the potential confidentiality, integrity, and availability consequences.

Use CVSS where appropriate and explain any contextual adjustments. Do not assign a high severity solely because a tool reports an issue.

## 5. Remediation and Retesting

* Provide a specific, actionable remediation recommendation.
* Prefer fixes that address the underlying cause.
* Retest only within the authorized scope.
* Record whether the issue is fixed, partially fixed, or unresolved.
* Document residual risks and any limitations.

## 6. Tools

Tools may include:

* Burp Suite for HTTP inspection and authorized web testing.
* Browser developer tools for application behavior.
* Nmap for approved network discovery.
* Nuclei for carefully scoped template-based checks.
* Linux command-line utilities for evidence processing.

Tool output is an indication to investigate, not proof that a vulnerability exists.

## 7. Rules of Engagement

1. Obtain authorization before testing.
2. Test only explicitly in-scope assets.
3. Avoid destructive actions and unnecessary data access.
4. Respect rate limits and testing windows.
5. Protect collected evidence.
6. Stop and report unexpected impact.
7. Document limitations honestly.

## 8. Assessment Summary Template

**Assessment:** [Name of authorized lab or engagement]

**Scope:** [Approved target and boundaries]

**Testing dates:** [Start and end dates]

**Methods:** [Methods actually used]

**Findings:** [Confirmed issues and severity]

**Limitations:** [Areas not tested and reasons]

**Recommendations:** [Prioritized remediation actions]

**Retest status:** [Not tested / Fixed / Partially fixed / Unresolved]

---

*This methodology is a working checklist. Adapt it to the engagement scope, applicable standards, and rules of engagement.*
