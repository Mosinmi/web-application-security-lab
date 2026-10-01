# Web Application Security & Vulnerability Assessment Lab

![Security Assessment](https://img.shields.io/badge/Focus-Web%20Application%20Security-red)
![OWASP Juice Shop](https://img.shields.io/badge/Target-OWASP%20Juice%20Shop-orange)
![Nmap](https://img.shields.io/badge/Tool-Nmap-blue)
![OWASP ZAP](https://img.shields.io/badge/Tool-OWASP%20ZAP-blue)
![Docker](https://img.shields.io/badge/Environment-Docker-blue)
![Status](https://img.shields.io/badge/Status-Completed-success)

## Overview

This project is a hands-on **Web Application Security and Vulnerability Assessment Lab** performed against an intentionally vulnerable **OWASP Juice Shop** application running in a controlled local environment.

The project demonstrates an end-to-end security assessment workflow covering:

**Reconnaissance → Enumeration → Automated Scanning → Manual Validation → Evidence Collection → Vulnerability Analysis → Remediation Planning → Retesting**

The assessment was performed exclusively against the local laboratory target:

`http://127.0.0.1:3000`

No production systems, third-party applications, or unauthorized infrastructure were targeted.

---

## Objectives

The objectives of this project were to:

- Perform network and service reconnaissance.
- Identify the application's attack surface.
- Discover API and application endpoints.
- Perform automated vulnerability scanning.
- Manually validate significant scanner findings.
- Analyze HTTP requests and responses.
- Identify and document confirmed vulnerabilities.
- Collect reproducible security evidence.
- Assess potential security impact.
- Develop technical remediation recommendations.
- Define appropriate retesting procedures.
- Produce professional security assessment documentation.

---

## Lab Environment

| Component | Details |
|---|---|
| **Target Application** | OWASP Juice Shop |
| **Target URL** | `http://127.0.0.1:3000` |
| **Operating System** | Windows 10 Pro |
| **Container Platform** | Docker Desktop |
| **Web Security Scanner** | OWASP ZAP 2.17.0 |
| **Network Scanner** | Nmap 7.991 |
| **Browser** | Google Chrome |
| **Command Shell** | PowerShell |
| **Testing Type** | Local / Authorized Laboratory |

### Scope

Testing was restricted to:

- `127.0.0.1`
- `localhost`
- The locally hosted OWASP Juice Shop application

---

# Assessment Methodology

The assessment followed a structured workflow:

1. **Reconnaissance**
2. **Service Discovery**
3. **Application Endpoint Enumeration**
4. **Automated Vulnerability Scanning**
5. **Manual Vulnerability Validation**
6. **Evidence Collection**
7. **Security Impact Assessment**
8. **Remediation Planning**
9. **Retesting Methodology**
10. **Professional Reporting**

Automated scanner alerts were treated as potential findings and were manually reviewed before being classified as confirmed vulnerabilities.

---

# 1. Reconnaissance

## Nmap Port Discovery

Nmap was used to determine whether the Juice Shop application was accessible on TCP port 3000.

**Command:**

    nmap -p 3000 127.0.0.1

**Result:**

    3000/tcp open

The scan confirmed that the local Juice Shop service was reachable through TCP port 3000.

**Evidence:** `evidence/screenshots/E-01-nmap-port-3000.png`

---

# 2. Application Endpoint Discovery

Browser traffic and application requests were inspected to identify API endpoints and application functionality.

### Key Endpoints Identified

- `/rest/admin/application-version`
- `/rest/admin/application-configuration`
- `/rest/products/search`
- `/rest/languages`
- `/api/Challenges/`
- `/api/Quantitys/`
- `/socket.io/`

Additional application namespaces identified during ZAP analysis included:

- `/api/Challenges`
- `/api/Feedbacks`
- `/api/Quantitys`
- `/rest/admin`
- `/rest/languages`
- `/rest/products`
- `/rest/user`

**Evidence:** `evidence/screenshots/E-02-endpoint-discovery.png`

---

# 3. Automated Vulnerability Scanning

**OWASP ZAP 2.17.0** was used to crawl and actively scan the local Juice Shop application.

The scanner identified multiple security alerts.

Scanner results were not automatically treated as confirmed vulnerabilities. Relevant findings were manually reviewed and validated before inclusion in the final assessment.

Three findings were confirmed:

| ID | Finding | Severity | Status |
|---|---|---|---|
| **F-01** | Unauthenticated Administrative Configuration Exposure | High | Confirmed |
| **F-02** | SQL Injection in Product Search | High | Confirmed |
| **F-03** | Missing Content Security Policy | Medium | Confirmed |

---

# Confirmed Vulnerability Findings

## F-01 — Unauthenticated Administrative Configuration Exposure

**Severity:** High  
**Status:** Confirmed

### Affected Endpoint

`GET /rest/admin/application-configuration`

### Description

The administrative application configuration endpoint returned a successful HTTP response without authentication.

The response contained application configuration information.

### Validation

The endpoint was accessed without providing authentication credentials or an authenticated session.

The server returned:

    HTTP/1.1 200 OK

and supplied configuration information in the response.

### Security Impact

Unauthenticated exposure of application configuration information can provide useful reconnaissance information about the application's architecture and security configuration.

The assessment did **not** demonstrate:

- Administrative privilege escalation
- Modification of server configuration
- Remote code execution

### Evidence

`evidence/screenshots/E-03-admin-configuration.png`

### Recommended Remediation

- Require authentication for administrative configuration endpoints.
- Enforce role-based authorization.
- Restrict administrative API functionality to authorized users.
- Minimize configuration information returned to clients.
- Prevent sensitive configuration information from being exposed through public APIs.
- Review related `/rest/admin/*` endpoints for similar access-control issues.

---

## F-02 — SQL Injection in Product Search

**Severity:** High  
**Status:** Confirmed

### Affected Endpoint

`GET /rest/products/search?q=`

### Affected Parameter

`q`

### Description

The product search functionality was observed to process SQL metacharacters in the `q` parameter in a manner that generated a database syntax error.

A normal search request was first established as a baseline.

**Baseline request:**

    GET /rest/products/search?q=apple

The baseline request returned:

    HTTP/1.1 200 OK

and product data.

A manually crafted SQL metacharacter input was then submitted.

### Manual Validation

**Validation request:**

    GET /rest/products/search?q=%27%28

The application returned:

    HTTP/1.1 500 Internal Server Error

with an SQLite database syntax error.

### Security Impact

The behavior demonstrates unsafe handling of user-controlled input reaching database processing.

The confirmed impact within this assessment was database error disclosure.

The assessment did **not** demonstrate:

- Data extraction
- Data modification
- Authentication bypass
- Remote code execution

### Evidence

- `evidence/screenshots/E-04-sql-injection-zap.png`
- `evidence/screenshots/E-05-sql-injection-manual.png`

### Recommended Remediation

- Use parameterized SQL queries.
- Use prepared statements.
- Never concatenate user-controlled input directly into SQL statements.
- Apply appropriate server-side input validation.
- Prevent database implementation details from being returned to clients.
- Log detailed database errors internally while returning generic client-facing errors.

---

## F-03 — Missing Content Security Policy

**Severity:** Medium  
**Status:** Confirmed

### Affected Resource

`GET /`

### Description

The main application response did not include either:

`Content-Security-Policy`

or:

`Content-Security-Policy-Report-Only`

Other security headers were observed, including:

- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: SAMEORIGIN`

### Validation

The application's HTTP response headers were inspected using both OWASP ZAP and manual HTTP header analysis.

No Content Security Policy header was present.

### Security Impact

The absence of CSP removes an important browser-side defense-in-depth control.

CSP can help restrict the sources from which scripts and other browser resources may be loaded.

The absence of CSP does **not**, by itself, demonstrate an exploitable cross-site scripting vulnerability.

### Evidence

- `evidence/screenshots/E-06-missing-csp-zap.png`
- `evidence/screenshots/E-07-missing-csp-manual.png`

### Recommended Remediation

Implement an application-specific Content Security Policy.

A staged implementation can begin with:

`Content-Security-Policy-Report-Only`

This allows policy violations to be observed and legitimate application dependencies to be identified before enforcement.

After refinement, the policy should be enforced using:

`Content-Security-Policy`

The policy should be as restrictive as practical for the application's legitimate functionality.

---

# Additional Scanner Observations

Several additional observations generated during automated scanning were reviewed but were **not promoted to confirmed vulnerabilities**.

These included:

- Wildcard CORS on a public JavaScript resource
- Missing anti-clickjacking header on a Socket.IO polling response
- Missing `X-Content-Type-Options` on a Socket.IO response
- Session ID in URL rewrite
- Private IP disclosure
- Unix timestamp disclosure
- Suspicious comments

These observations were not treated as confirmed vulnerabilities because sufficient security impact was not demonstrated within the scope of this assessment.

This demonstrates an important security assessment principle:

> Automated scanner output should be validated rather than blindly reported as exploitable vulnerabilities.

---

# Remediation Strategy

Detailed remediation documentation is available in:

- `remediation/remediation-plan.md`
- `remediation/remediation-report.md`

## Access Control

Administrative endpoints should require authentication and appropriate authorization.

## Injection Prevention

User-controlled input should be handled using parameterized database queries and prepared statements.

## Secure Error Handling

Database and implementation details should not be exposed through client-facing error messages.

## Browser Security

A properly configured Content Security Policy should be implemented to strengthen browser-side security controls.

---

# Retesting Strategy

Each confirmed finding should be retested after remediation.

| Finding | Retesting Objective |
|---|---|
| **F-01** | Unauthenticated requests should not return administrative configuration |
| **F-02** | SQL metacharacters should not generate database errors |
| **F-03** | A valid Content Security Policy header should be present |

The original vulnerable Juice Shop instance was not modified during this assessment.

Therefore, this project documents the remediation requirements and retesting methodology without falsely claiming that the vulnerabilities have already been fixed.

---

# Evidence

All assessment screenshots are stored under:

`evidence/screenshots/`

## Evidence Index

| Evidence ID | Description |
|---|---|
| **E-01** | Nmap port discovery |
| **E-02** | Application endpoint discovery |
| **E-03** | Administrative configuration exposure |
| **E-04** | SQL injection identified by ZAP |
| **E-05** | Manual SQL injection validation |
| **E-06** | Missing CSP identified by ZAP |
| **E-07** | Manual CSP header validation |

---

# Project Structure

    web-application-security-lab/
    │
    ├── README.md
    │
    ├── reconnaissance/
    │   ├── reconnaissance-report.md
    │   └── nmap-results.txt
    │
    ├── scans/
    │   └── zap-scan-summary.md
    │
    ├── findings/
    │   ├── F-01-admin-config-exposure.md
    │   ├── F-02-sql-injection.md
    │   └── F-03-missing-csp.md
    │
    ├── evidence/
    │   ├── evidence-log.md
    │   └── screenshots/
    │       ├── E-01-nmap-port-3000.png
    │       ├── E-02-endpoint-discovery.png
    │       ├── E-03-admin-configuration.png
    │       ├── E-04-sql-injection-zap.png
    │       ├── E-05-sql-injection-manual.png
    │       ├── E-06-missing-csp-zap.png
    │       └── E-07-missing-csp-manual.png
    │
    ├── remediation/
    │   ├── remediation-plan.md
    │   └── remediation-report.md
    │
    └── docs/
        └── final-security-assessment-report.md

---

# Skills Demonstrated

This project demonstrates practical experience with:

- Web application reconnaissance
- Vulnerability assessment
- OWASP ZAP
- Nmap
- HTTP request and response analysis
- API endpoint enumeration
- SQL injection identification
- Manual vulnerability validation
- Access-control assessment
- Security-header analysis
- Evidence collection
- Vulnerability documentation
- Security impact assessment
- Remediation planning
- Retesting methodology
- Technical security reporting
- PowerShell
- Docker-based security laboratories

---

# Tools Used

### OWASP ZAP

Used for web application crawling, active vulnerability scanning, and identification of potential security weaknesses.

### Nmap

Used for local service and port discovery.

### Chrome DevTools

Used to inspect application traffic and identify API endpoints and client-server communication.

### PowerShell

Used for HTTP request validation, reconnaissance commands, evidence generation, and project management.

### Docker Desktop

Used to host the intentionally vulnerable Juice Shop application in an isolated local environment.

### OWASP Juice Shop

Used as the intentionally vulnerable web application target for security testing.

---

# Security & Ethics

This project was performed against an intentionally vulnerable application in a controlled local laboratory environment.

Testing was restricted to systems under my control.

No unauthorized systems, production applications, or third-party infrastructure were targeted.

The project is intended for cybersecurity education, practical skill development, and professional portfolio demonstration.

---

# Documentation

Detailed project documentation is available in:

- `reconnaissance/reconnaissance-report.md`
- `scans/zap-scan-summary.md`
- `findings/F-01-admin-config-exposure.md`
- `findings/F-02-sql-injection.md`
- `findings/F-03-missing-csp.md`
- `evidence/evidence-log.md`
- `remediation/remediation-plan.md`
- `remediation/remediation-report.md`
- `docs/final-security-assessment-report.md`

---

# Project Status

**Completed — Assessment & Documentation**

This project demonstrates an end-to-end web application security assessment workflow covering reconnaissance, application enumeration, automated vulnerability scanning, manual validation, evidence collection, vulnerability analysis, remediation planning, and professional security reporting.