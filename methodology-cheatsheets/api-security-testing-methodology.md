# API Security Testing Methodology

## 1. Objective

This document outlines a structured approach to assessing API security. It covers endpoint discovery, authentication, authorization, input validation, configuration, and reporting.

All testing must be performed on systems owned by the tester or with explicit authorization.

## 2. Scope and Preparation

Before testing:

* Confirm written authorization and approved API endpoints.
* Identify the API base URL, supported versions, and testing environment.
* Review available OpenAPI/Swagger documentation.
* Obtain approved test accounts with different roles.
* Identify rate limits, prohibited actions, and testing windows.
* Use test data and avoid accessing real users' sensitive information.

## 3. API Testing Workflow

### Phase 1: Endpoint Discovery

Objective: Understand the API's exposed functionality.

Checklist:

* [ ] Review available API documentation.
* [ ] Identify HTTP methods used by each endpoint.
* [ ] Record required parameters, headers, and request bodies.
* [ ] Identify authentication requirements.
* [ ] Map endpoints to application functions and user roles.
* [ ] Record undocumented behavior observed during normal authorized use.

**Deliverable:** An API endpoint inventory.

### Phase 2: Authentication

Objective: Verify that protected endpoints require appropriate authentication.

Checklist:

* [ ] Identify endpoints that require authentication.
* [ ] Check how missing or invalid credentials are handled.
* [ ] Review token expiration and session invalidation.
* [ ] Check whether sensitive operations require reauthentication where appropriate.
* [ ] Review API key handling and storage practices.
* [ ] Ensure credentials and tokens are not exposed in URLs or logs.

**Deliverable:** Authentication test results with supporting evidence.

### Phase 3: Authorization

Objective: Verify that users can access only permitted objects and functions.

Checklist:

* [ ] Use separate, authorized test accounts with different roles.
* [ ] Check access to test-owned objects under different accounts.
* [ ] Verify that object-level permissions are enforced server-side.
* [ ] Check whether restricted functions are protected by role or permission checks.
* [ ] Review whether users can assign themselves unauthorized privileges.
* [ ] Confirm that denied requests do not disclose unnecessary information.

**Deliverable:** Documented authorization results, including any confirmed access-control weaknesses.

### Phase 4: Input Validation

Objective: Determine whether the API validates client-supplied data correctly.

Checklist:

* [ ] Identify parameters, JSON fields, headers, and query strings.
* [ ] Check required fields, data types, length limits, and allowed values.
* [ ] Review handling of unexpected or malformed input.
* [ ] Assess injection risks using harmless test cases in an approved environment.
* [ ] Check whether server-side validation is applied consistently.
* [ ] Confirm that error responses do not expose implementation details.

**Deliverable:** Validation observations and safely reproduced findings.

### Phase 5: Data Exposure

Objective: Check whether API responses disclose more information than necessary.

Checklist:

* [ ] Review response fields for sensitive or unnecessary data.
* [ ] Check that each user receives only authorized records.
* [ ] Review error messages for sensitive implementation details.
* [ ] Check whether sensitive information appears in URLs or responses.
* [ ] Use synthetic test records wherever possible.
* [ ] Redact tokens, credentials, and personal information from evidence.

**Deliverable:** Data exposure findings and recommended response minimization.

### Phase 6: Rate Limiting and Resource Controls

Objective: Assess protections against excessive requests and resource abuse.

Checklist:

* [ ] Review documented request limits.
* [ ] Check whether sensitive operations have appropriate rate limits.
* [ ] Assess throttling using low-volume tests within the approved limits.
* [ ] Review pagination and maximum request sizes.
* [ ] Check whether resource-intensive operations have reasonable safeguards.
* [ ] Stop testing if service stability is affected.

**Deliverable:** Rate-limit observations and resource-control recommendations.

### Phase 7: API Configuration

Objective: Review common API security configuration weaknesses.

Checklist:

* [ ] Verify HTTPS enforcement.
* [ ] Review relevant security headers.
* [ ] Inspect CORS behavior where browser-based access is relevant.
* [ ] Check supported API versions and deprecated endpoints.
* [ ] Review debug responses and error handling.
* [ ] Check that secrets are not embedded in public documentation or responses.

**Deliverable:** Configuration review notes.

### Phase 8: Business Logic

Objective: Verify that important workflows enforce their intended rules.

Checklist:

* [ ] Map sensitive workflows such as account changes or purchases in a test environment.
* [ ] Verify that users cannot skip required workflow steps.
* [ ] Check server-side enforcement of ownership and business rules.
* [ ] Review how duplicate or repeated requests are handled.
* [ ] Confirm that operations cannot be performed outside permitted states.
* [ ] Avoid real transactions or irreversible actions unless explicitly approved.

**Deliverable:** Business-logic observations supported by reproducible test evidence.

## 4. Finding Documentation

For every confirmed finding, record:

* Finding title and unique identifier.
* Affected endpoint and HTTP method.
* Required role or preconditions.
* Summary of the security weakness.
* Minimal reproduction steps.
* Sanitized request and response evidence.
* Impact and severity rationale.
* Recommended remediation.
* Retest result, if available.

Do not include live credentials, bearer tokens, API keys, or unnecessary sensitive data in reports.

## 5. Severity Assessment

Evaluate findings based on exploitability, required privileges, affected data, business impact, and potential confidentiality, integrity, and availability consequences.

Use CVSS when appropriate and explain any context-specific adjustments.

Distinguish confirmed vulnerabilities from unverified observations.

## 6. Remediation Principles

Recommended defensive controls include:

* Enforce authentication on protected endpoints.
* Apply object-level and function-level authorization server-side.
* Validate inputs against explicit schemas.
* Return only the data required by the client.
* Apply appropriate rate limits and resource constraints.
* Protect credentials and secrets throughout their lifecycle.
* Use consistent error handling and security logging.
* Retest fixes in an authorized environment.

## 7. Assessment Summary

**Assessment name:** [Authorized lab or engagement]

**Environment:** [Lab / Staging / Production, as authorized]

**Scope:** [Approved API base URL and endpoints]

**Testing dates:** [Start and end dates]

**Authentication methods:** [Methods actually tested]

**Findings:** [Confirmed issues and severity]

**Limitations:** [Untested endpoints or restrictions]

**Recommendations:** [Prioritized corrective actions]

**Retest status:** [Not tested / Fixed / Partially fixed / Unresolved]

## 8. Rules of Engagement

1. Obtain explicit authorization before testing.
2. Test only approved endpoints and environments.
3. Use designated test accounts and synthetic data.
4. Respect request limits and testing windows.
5. Avoid destructive operations and unnecessary data access.
6. Stop if unexpected service impact occurs.
7. Store evidence securely and redact sensitive values.
8. Report findings responsibly.

---

*This methodology is a working checklist. Adapt it to the approved scope, applicable standards, and engagement rules.*
