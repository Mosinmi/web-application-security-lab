# Web Application Security & Vulnerability Assessment Report

## OWASP Juice Shop — Local Security Assessment

**Assessment Type:** Web Application Security Assessment  
**Target:** OWASP Juice Shop  
**Target Address:** http://127.0.0.1:3000  
**Testing Environment:** Local / Authorized Lab  
**Tools:** Nmap, OWASP ZAP, PowerShell, Chrome DevTools  
**Assessment Date:** October 1, 2026  

---

# 1. Executive Summary

A security assessment was conducted against an intentionally vulnerable OWASP Juice Shop instance hosted locally.

The objective was to identify common web application security weaknesses, validate significant findings manually, document evidence, assess potential impact, and develop remediation recommendations.

The assessment identified three confirmed security findings:

| ID | Finding | Severity |
|---|---|---|
| F-01 | Unauthenticated Administrative Configuration Exposure | High |
| F-02 | SQL Injection in Product Search | High |
| F-03 | Missing Content Security Policy | Medium |

The assessment combined automated vulnerability scanning with manual validation to reduce reliance on scanner output alone.

The testing was restricted to the local Juice Shop instance at 127.0.0.1.

---

# 2. Scope

## In Scope

The following application was assessed:

http://127.0.0.1:3000

The assessment included:

- Network/service reconnaissance
- Web endpoint discovery
- Automated vulnerability scanning
- HTTP response/header analysis
- SQL injection validation
- Access-control validation
- Security-header analysis
- Evidence collection
- Remediation planning

## Out of Scope

The following were not assessed:

- External websites
- Production systems
- Third-party infrastructure
- Other users' systems
- Denial-of-service testing
- Destructive exploitation

All testing was performed against the intentionally vulnerable local laboratory environment.

---

# 3. Methodology

The assessment followed a structured workflow:

1. Reconnaissance
2. Application discovery
3. Automated vulnerability scanning
4. Manual validation
5. Evidence collection
6. Impact assessment
7. Remediation planning
8. Retesting methodology

Automated scanner results were treated as leads requiring manual validation where practical.

---

# 4. Reconnaissance

## 4.1 Port Discovery

Nmap was used to determine whether the Juice Shop service was accessible on TCP port 3000.

Command:

nmap -p 3000 127.0.0.1

Result:

3000/tcp open

Evidence:

E-01-nmap-port-3000.png

---

# 5. Application Endpoint Discovery

Browser and application traffic was inspected to identify API and application endpoints.

Observed endpoints included:

- /rest/admin/application-version
- /rest/admin/application-configuration
- /api/Challenges/
- /rest/languages
- /api/Quantitys/
- /rest/products/search
- /socket.io/

Additional namespaces identified during ZAP analysis included:

- /api/Challenges
- /api/Feedbacks
- /api/Quantitys
- /rest/admin
- /rest/languages
- /rest/products
- /rest/user

Evidence:

E-02-endpoint-discovery.png

---

# 6. Vulnerability Findings

## F-01 — Unauthenticated Administrative Configuration Exposure

### Severity

High

### Status

Confirmed

### Endpoint

GET /rest/admin/application-configuration

### Description

The administrative application-configuration endpoint returned a successful HTTP response without authentication.

A request without authentication returned:

HTTP/1.1 200 OK

with a JSON configuration object.

### Validation

The endpoint was requested without authentication and without a session cookie.

The response returned application configuration information.

### Impact

Unauthenticated access to configuration information can provide attackers with useful reconnaissance data about the application's architecture and security configuration.

The assessment did not demonstrate administrative privilege escalation or modification of server configuration.

### Evidence

E-03-admin-configuration.png

### Recommended Remediation

- Require authentication.
- Enforce role-based authorization.
- Restrict administrative endpoints to authorized users.
- Minimize configuration information returned to clients.
- Review related /rest/admin/* endpoints.

---

# F-02 — SQL Injection in Product Search

### Severity

High

### Status

Confirmed

### Endpoint

GET /rest/products/search?q=

### Parameter

q

### Description

The product search parameter was observed to process SQL metacharacters in a manner that produced a database syntax error.

### Baseline

A normal search request:

/rest/products/search?q=apple

returned HTTP 200 OK and product data.

### Manual Validation

The following test input was submitted:

/rest/products/search?q=%27%28

The application returned:

HTTP/1.1 500 Internal Server Error

with an SQLite database error indicating a syntax error.

### Impact

The behavior demonstrates unsafe handling of user-controlled input reaching database processing.

The assessment confirmed database error disclosure.

The assessment did not demonstrate:

- Data extraction
- Data modification
- Authentication bypass
- Remote code execution

### Evidence

E-04-sql-injection-zap.png

E-05-sql-injection-manual.png

### Recommended Remediation

- Use parameterized queries.
- Use prepared statements.
- Never concatenate user-controlled input into SQL statements.
- Validate input server-side.
- Suppress database implementation errors from client responses.

---

# F-03 — Missing Content Security Policy

### Severity

Medium

### Status

Confirmed

### Endpoint

/

### Description

The main application response did not include:

Content-Security-Policy

or:

Content-Security-Policy-Report-Only

### Validation

HTTP response headers were inspected manually.

The response included security headers such as:

X-Content-Type-Options: nosniff

X-Frame-Options: SAMEORIGIN

However, no CSP header was present.

### Impact

The absence of CSP removes an important browser-side defense-in-depth control.

This finding does not by itself demonstrate an exploitable XSS vulnerability.

### Evidence

E-06-missing-csp-zap.png

E-07-missing-csp-manual.png

### Recommended Remediation

Implement an application-specific Content Security Policy.

A report-only policy can initially be deployed to identify legitimate resource requirements before enforcement.

---

# 7. Security Controls Observed

The assessment also identified existing security controls.

Observed response headers included:

- X-Content-Type-Options: nosniff
- X-Frame-Options: SAMEORIGIN

The application also exposed a recruiting-related response header:

X-Recruiting: /#/jobs

These observations were recorded as contextual information rather than separate confirmed vulnerabilities.

---

# 8. Findings Not Promoted to Confirmed Vulnerabilities

Several scanner observations were reviewed but were not promoted to confirmed findings because their security impact could not be sufficiently demonstrated within the assessment.

These included observations relating to:

- Wildcard CORS on a public JavaScript resource
- Missing anti-clickjacking header on Socket.IO polling responses
- Missing X-Content-Type-Options on Socket.IO responses
- Session ID in URL rewrite
- Private IP disclosure
- Unix timestamp disclosure
- Suspicious comments

This distinction was made to avoid treating every automated scanner alert as a confirmed vulnerability.

---

# 9. Remediation

Detailed remediation guidance is documented in:

remediation/remediation-plan.md

and:

remediation/remediation-report.md

Primary remediation areas include:

### Access Control

Require authentication and authorization for administrative endpoints.

### Injection Prevention

Use parameterized database queries and prepared statements.

### Security Headers

Implement an appropriate Content Security Policy.

### Error Handling

Prevent database and application implementation details from being exposed to clients.

---

# 10. Evidence Repository

Evidence is stored under:

evidence/screenshots/

The evidence set contains:

- E-01-nmap-port-3000.png
- E-02-endpoint-discovery.png
- E-03-admin-configuration.png
- E-04-sql-injection-zap.png
- E-05-sql-injection-manual.png
- E-06-missing-csp-zap.png
- E-07-missing-csp-manual.png

---

# 11. Assessment Limitations

This assessment was performed against an intentionally vulnerable local application.

The assessment demonstrates vulnerability discovery and validation within the defined laboratory environment.

It should not be interpreted as an assessment of a production Juice Shop deployment or any unrelated external system.

The remediation section documents recommended controls and retesting procedures. The original vulnerable application was not modified as part of this assessment.

---

# 12. Conclusion

The assessment demonstrated a practical web application security workflow covering reconnaissance, automated scanning, manual validation, evidence collection, vulnerability analysis, and remediation planning.

The exercise provided hands-on experience with:

- Nmap reconnaissance
- OWASP ZAP
- HTTP request/response analysis
- SQL injection validation
- Access-control testing
- Security-header analysis
- Vulnerability documentation
- Remediation planning
- Security assessment reporting

The resulting artifacts are structured for use as a cybersecurity portfolio project and can be reviewed independently through the associated evidence and documentation.
