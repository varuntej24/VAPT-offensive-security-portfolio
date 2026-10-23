# Sample Finding: Broken Access Control

> **Portfolio note:** This is a fictional training-lab example created to demonstrate security reporting. It is not a real client assessment, and the results have not been independently tested.

## 1. Finding Details

| Field              | Details                                         |
| ------------------ | ----------------------------------------------- |
| Finding ID         | LAB-VAPT-001                                    |
| Vulnerability      | Broken Object-Level Authorization (BOLA / IDOR) |
| Severity           | Medium — illustrative only                      |
| Affected Component | Fictional API endpoint: `/api/v1/profile/{id}`  |
| Assessment Type    | Authorized API security lab                     |
| Status             | Simulated — not independently validated         |

## 2. Summary

Broken object-level authorization occurs when an application fails to verify whether the authenticated user is permitted to access a requested object. As a result, a user might be able to access another user's information by changing an object identifier.

This example describes a hypothetical API where authorization checks are assumed to be missing. The weakness must be reproduced in an authorized lab before it can be reported as a confirmed vulnerability.

## 3. Preconditions

* Two dedicated test accounts are available in an authorized lab.
* Each account owns a separate test profile.
* The tester is permitted to assess object-level authorization.
* No real user data is involved.

## 4. Test Scenario

1. Authenticate as Test User A.
2. Access the profile belonging to Test User A.
3. Record the expected response and the test profile identifier.
4. In the authorized lab, request a different test profile identifier belonging to Test User B.
5. Verify whether the server denies access or returns information that Test User A is not authorized to view.
6. Stop after collecting the minimum evidence required.

**Expected secure behavior:** The server should enforce object-level authorization and deny access to objects the authenticated user is not permitted to view.

**Hypothetical insecure behavior:** The server returns Test User B's profile data to Test User A without a valid authorization check.

## 5. Evidence

**Evidence status:** Illustrative only; no actual request or response was captured for this example.

Record the following when testing a real authorized lab:

* Sanitized request method and endpoint.
* Test account and role used.
* Redacted request and response.
* Expected versus observed behavior.
* Screenshot or log reference.

Do not include passwords, session tokens, personal information, or real user data.

## 6. Potential Impact

If confirmed, this weakness could allow unauthorized access to other users' profile information. The actual impact depends on the data exposed, the operations available, and the application's authorization model.

## 7. Remediation

* Enforce authorization checks on the server for every object-level request.
* Derive the authenticated user's identity from the validated session or token.
* Verify ownership or explicit permissions before returning or modifying an object.
* Apply authorization checks consistently across API endpoints and HTTP methods.
* Add automated tests covering access attempts between separate user accounts.
* Log suspicious authorization failures without recording sensitive data.

Changing object identifiers or relying on their unpredictability is not a substitute for authorization enforcement.

## 8. Retest Plan

After remediation:

1. Authenticate as Test User A.
2. Request Test User A's own test profile.
3. Request Test User B's test profile using the same account.
4. Confirm that the first request succeeds and the unauthorized request is denied.
5. Verify that no protected data is disclosed in the denial response.

**Retest status:** Not performed — fictional example.

## 9. References

* OWASP API Security Top 10 — Broken Object Level Authorization (API1).
* OWASP Authorization Cheat Sheet.

## 10. Lessons Learned

Object identifiers are not security controls. Authorization must be enforced server-side for every protected object and operation. Findings should distinguish verified evidence from hypothetical scenarios and should include clear remediation and retest criteria.
