# API Security Testing Checklist

## 1. Objective

This checklist provides a structured approach to assessing API security in authorized penetration tests and controlled lab environments.

## 2. Scope and Authorization

* [ ] Confirm written authorization.
* [ ] Identify in-scope API base URLs and versions.
* [ ] Review API documentation, such as OpenAPI or Swagger.
* [ ] Identify permitted HTTP methods and testing limits.
* [ ] Create dedicated test accounts and test data.
* [ ] Confirm prohibited actions and rate limits.

## 3. API Endpoint Discovery

* [ ] Review available API documentation.
* [ ] Identify endpoints, HTTP methods, parameters, and content types.
* [ ] Record endpoints requiring authentication.
* [ ] Identify endpoints used by different user roles.
* [ ] Review error responses for unnecessary information disclosure.
* [ ] Keep discovery within the approved scope.

**Evidence to record:** Endpoint inventory, HTTP methods, authentication requirements, and observed response codes.

## 4. Authentication

* [ ] Verify that protected endpoints require authentication.
* [ ] Review token expiration and session invalidation.
* [ ] Check whether logout invalidates the relevant session or token.
* [ ] Review password recovery and account recovery workflows.
* [ ] Check how invalid, expired, or malformed tokens are handled.
* [ ] Confirm that credentials and tokens are not exposed in URLs or logs.

**Evidence to record:** Sanitized requests and responses demonstrating the authentication behavior.

## 5. Authorization and Access Control

### Object-Level Authorization (BOLA/IDOR)

* [ ] Create two authorized test accounts with separate test records.
* [ ] Verify that each account can access only permitted records.
* [ ] Check read, update, and delete permissions using test-owned data.
* [ ] Confirm that changing an object identifier does not bypass server-side authorization.

### Function-Level Authorization

* [ ] Review permissions for sensitive operations.
* [ ] Verify that ordinary users cannot perform administrator-only actions.
* [ ] Check authorization enforcement across relevant HTTP methods.
* [ ] Confirm that access restrictions are enforced server-side.

### Property-Level Authorization

* [ ] Check whether users can read restricted object properties.
* [ ] Review whether updates to sensitive properties are properly restricted.
* [ ] Check for excessive data exposure in API responses.
* [ ] Verify that the server accepts only permitted fields.

**Evidence to record:** Test account roles, expected access, observed access, and sanitized responses.

## 6. Input Validation and Data Handling

* [ ] Identify user-controlled parameters and request-body fields.
* [ ] Check required fields, data types, length limits, and accepted formats.
* [ ] Verify that unexpected values are rejected safely.
* [ ] Review error handling for information disclosure.
* [ ] Assess injection risks using harmless inputs in an authorized test environment.
* [ ] Verify that server-side validation is applied consistently.

**Evidence to record:** Input tested, expected behavior, actual behavior, and security impact.

## 7. Rate Limiting and Resource Consumption

* [ ] Review documented API request limits.
* [ ] Check whether rate limits apply to relevant sensitive endpoints.
* [ ] Verify that excessive requests are handled safely within approved limits.
* [ ] Review pagination and maximum request-size controls.
* [ ] Check whether expensive operations have appropriate resource limits.
* [ ] Stop testing if service performance degrades.

**Evidence to record:** Approved test conditions, observed rate-limit behavior, and response codes.

## 8. API Configuration and Transport Security

* [ ] Verify HTTPS enforcement.
* [ ] Review TLS certificate validity.
* [ ] Check CORS configuration against the application's intended origins.
* [ ] Review security-related response headers where applicable.
* [ ] Check for unnecessary debug information in responses.
* [ ] Verify that sensitive information is not unnecessarily cached.
* [ ] Review API versioning and deprecated endpoint exposure.

**Evidence to record:** Relevant response headers, configuration observations, and risk assessment.

## 9. Error Handling and Information Disclosure

* [ ] Check whether errors expose stack traces or internal paths.
* [ ] Review responses for secrets or unnecessary personal information.
* [ ] Verify that authentication and authorization failures are handled safely.
* [ ] Confirm that logs do not unnecessarily contain credentials or tokens.
* [ ] Check that error messages do not reveal sensitive implementation details.

## 10. Reporting Findings

For every confirmed finding, document:

* Finding ID and title.
* Affected endpoint and HTTP method.
* Severity and rationale.
* Required authentication and privileges.
* Preconditions and reproducible test steps.
* Sanitized request and response evidence.
* Security impact.
* Recommended remediation.
* Retest result, if available.

Do not include real credentials, session tokens, or unnecessary personal data in reports.

## 11. Remediation and Retesting

* [ ] Recommend server-side authorization checks.
* [ ] Recommend strict input validation and safe data handling.
* [ ] Recommend appropriate rate limits and resource controls.
* [ ] Recommend secure token and session management.
* [ ] Retest fixes only within the authorized scope.
* [ ] Record whether each finding is fixed, partially fixed, or unresolved.

## 12. Assessment Summary

**Assessment name:** [Authorized lab or engagement]

**Target:** [Approved API]

**Testing period:** [Start and end dates]

**API version:** [Version]

**Endpoints reviewed:** [Number]

**Confirmed findings:** [Number by severity]

**Limitations:** [Areas not tested]

**Retest status:** [Status]

## Responsible Testing

All checks must be performed only against systems owned by the tester or explicitly authorized for assessment. Use test accounts and test data, avoid destructive actions, and stop if unexpected impact occurs.

---

*This checklist is a working document. Adapt it to the engagement scope, applicable standards, and rules of engagement.*
