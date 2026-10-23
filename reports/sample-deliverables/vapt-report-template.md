# Vulnerability Assessment and Penetration Testing (VAPT) Report Template

> **Template notice:** This is a reusable reporting template. Replace the placeholders with verified findings from an authorized assessment. Do not present example data as a real client engagement.

## 1. Assessment Details

| Field                | Details                        |
| -------------------- | ------------------------------ |
| Assessment / Project | [Project or lab name]          |
| Assessed Application | [Authorized target or lab URL] |
| Assessment Type      | Web Application / API          |
| Assessment Period    | [Start date – End date]        |
| Tester               | [Your name or team]            |
| Report Version       | 1.0                            |
| Report Status        | Draft / Final                  |

## 2. Executive Summary

**Objective:**
[Briefly describe why the assessment was performed.]

**Overall Risk:**
[Critical / High / Medium / Low / Informational, based on validated findings.]

**Summary of Results:**
[Summarize the most important verified findings, their potential business impact, and the recommended priorities.]

## 3. Scope

### In Scope

* [Authorized application, hostname, API, or lab]
* [Approved test accounts and roles]

### Out of Scope

* [Excluded systems, endpoints, or activities]

### Limitations

* [Untested features, unavailable accounts, time restrictions, or other limitations]

## 4. Methodology

The assessment followed a structured process:

1. Confirm authorization, scope, and rules of engagement.
2. Discover and map in-scope application functionality.
3. Review authentication and session management.
4. Assess authorization and access controls.
5. Evaluate input validation and common vulnerability classes.
6. Review relevant API security controls and configuration.
7. Validate potential findings with minimal-impact tests.
8. Document evidence, assess risk, and recommend remediation.

Only activities actually performed should be reported as completed.

## 5. Findings Summary

| ID       | Finding                  | Severity   | Status |
| -------- | ------------------------ | ---------- | ------ |
| VAPT-001 | [Verified finding title] | [Severity] | Open   |
| VAPT-002 | [Verified finding title] | [Severity] | Open   |

Remove unused rows. Do not include unverified scanner alerts as confirmed vulnerabilities.

## 6. Detailed Finding Template

### VAPT-[ID]: [Finding Title]

**Severity:** [Critical / High / Medium / Low / Informational]

**Affected Component:** [Endpoint, page, API route, or component]

**CWE / Reference:** [Applicable CWE or reference, if verified]

**Description:**
[Explain the weakness, its root cause, and the conditions required for it to occur.]

**Evidence:**
[Insert sanitized screenshots, request/response excerpts, logs, or other relevant evidence. Remove secrets and personal data.]

**Reproduction Steps:**

1. [Describe the authorized test setup.]
2. [Provide the minimum steps needed to reproduce the issue.]
3. [Record the observed result.]
4. [Describe the expected secure behavior.]

**Impact:**
[Explain realistic consequences for confidentiality, integrity, availability, or business operations.]

**Risk Rationale:**
[Explain the severity rating using exploitability, required privileges, affected assets, and business impact. Include CVSS when appropriate.]

**Recommended Remediation:**

* [Specific corrective action]
* [Additional defense-in-depth measure, if applicable]

**References:**
[Relevant official documentation or security guidance]

**Retest Status:** [Not tested / Fixed / Partially fixed / Unresolved]

**Retest Notes:** [Evidence and date of retesting, if performed]

## 7. Recommendations and Priorities

| Priority  | Action                                                        |
| --------- | ------------------------------------------------------------- |
| Immediate | Address validated critical risks and contain active exposure. |
| High      | Remediate significant weaknesses within the agreed timeframe. |
| Medium    | Plan corrective work and verify controls.                     |
| Low       | Improve defense in depth and maintain secure configuration.   |

Adjust priorities to the actual risk and the organization's remediation policy.

## 8. Conclusion

[Summarize the application's security posture, the most important risks, and the recommended next steps. Clearly state assessment limitations.]

## 9. Retest Summary

| Finding ID | Original Severity | Retest Result                          | Evidence    |
| ---------- | ----------------- | -------------------------------------- | ----------- |
| VAPT-001   | [Severity]        | [Fixed / Partially fixed / Unresolved] | [Reference] |

## 10. Evidence Handling

* Store evidence securely and restrict access.
* Redact passwords, tokens, personal data, and unrelated sensitive information.
* Use only evidence necessary to demonstrate the finding.
* Follow the engagement's retention and deletion requirements.

---

**Responsible Testing Statement:** All testing must remain within explicitly authorized scope. This report template is intended for legitimate security assessments and training labs.
